# Kafka 何时、如何删除Topic

原文：https://www.toutiao.com/a7048240867353657869/



Kafka 有很多状态机和管理器，如 Controller 通道管理器 ControllerChannelManager、处理 Controller 事件的 ControllerEventManager 等。这些管理器和状态机，大多与各自“宿主”联系密切。就如 Controller 这两个管理器，必须与 Controller 组合紧密耦合，才能实现各自功能。

Kafka 还有一些状态机和管理器，具有相对独立的功能框架，不严重依赖使用方，如：

* TopicDeletionManager（主题删除管理器）负责对指定 Kafka 主题执行删除操作，清楚待删除主题在集群上的各类“痕迹”
* ReplicaStateMachie（副本状态机）负责定义 Kafka 副本状态、合法的状态转换，以及管理状态之间的转换
* PartitionStateMachie（分区状态机）负责定义 Kafka 分区状态、合法的状态转换，以及管理状态之间的切换

这里讨论一下Kafka 是如何删除一个主题的。

## 前言

以为成功执行了命令 ***kafka-topic.sh --delete*** 后，主题就会被删除。这种不正确的认知会导致经常发生主题没有被删除干净。于是，网传终极“武林秘籍”：手动删除磁盘上的日志文件，手动删除 Zookeeper 下关于主题的各个节点。但不推荐这样做：

* 这样做并不完整，除非重启Broker，否则，这套“秘籍”无法清楚 Controller 端各个Broker上元数据缓存中的待删除主题的相关条目
* 这个“秘籍”并没有被官方认证

与其琢磨删除主题失败之后怎么自救，还不如研究 Kafka 到底是如何执行删除主题操作的。

TopicDeletionManager.scala 包括：

* **DeletionClient 接口**：负责实现删除主题以及后续的动作如何更新元数据
* **ControllerDeletionClient 类**：实现DeletionClient 接口的类，分别实现了刚刚说到的那4个方法
* **TopicDeletionManager类**：主题删除管理器类定义方法维护主题删除前后集群状态的正确性。如：何时删除主题、何时主题不能被删除、主题删除过程中要规避哪些操作等

## DeletionClient 接口及实现

删除主题，并将删除主题的事件同步给其它Broker。

DeletionClient 接口目前只有一个实现类 ControllerDeletionClient，构造器的两个字段：

* KafkaController 实例： Controller 组件对象
* KafkaZkClient 实例：Kafka 与 Zookeeper 交互的客户端对象

### API

#### deleteTopic

删除主题在 ZK 上的所有”痕迹“。分别调用了 KafkaZkClient 的 3 个方法，删除 ZooKeeper 下： 

* /brokers/topics/节点
* /config/topics/节点
* /admin/delete_topics/节点

#### deleteTopicDeletions

批量删除ZK下待删除主题的标记节点。调用 KafkaZkClient#deleteTopicDeletions，批量删除一组主题，位置为：/admin/delete_topics 下的字节点。

注意：deleteTopicDeletions 这个方法名结尾的 Deletions，表示 /admin/delete_topics 下的字节点，所以：

*  deleteTopic 是删除主题
* deleteTopicDeletions 是删除 /admin/delete_topics 下的对应字节点

这两个方法里都有 epochZkVersion 字段，代表期望的Controller Epoch 版本号。若使用一个旧 Epoch 版本号执行这些方法，zk会拒绝，因为和她自己保存的版本号不匹配。

若一个 Controller 的 Epoch < ZK中保存的，则该 Controller 很可能是已过期的Controller。这就是 Zombie Controller，epochZkVersion 字段的作用，就是隔离 Zombie Controller发送的操作。

#### mutePartitionModifications

屏蔽主题分区数据变更监听：取消/brokers/topics/节点数据变更的监听。

当该主题的分区数据发生变更后，由于对应ZK监听器已被取消，因此不会触发Controller相应处理逻辑。

为何取消该监听器？为了避免操作相互干扰：假设用户A发起主题删除，同时用户B为这个主题新增分区。此时，这两个操作就会冲突，若允许Controller同时处理这两个操作，势必会造成逻辑混乱及状态不一致。为应对这种情况，在移除主题副本和分区对象前，代码要先执行这个方法，确保不再响应用户对该主题的其他操作。

#### sendMetadataUpdate



## TopicDeletionManager定义及初始化

