# 20道经典的Kafka面试题详解

来源：https://www.jianshu.com/p/511962462e58



## 1. 基础题目

### 1.1 什么是消费者组？

消费者组，是Kafka特有的概念。其官方的定义：

1. 定义：消费者组是Kafka提供的可扩展且具有容错性的消费者机制
2. 原理：在Kafka中，消费者组是一个有多个消费者实例构成的组。多个实例共同订阅若干个主题，实现共同消费。**同一个组下的每个实例都配置有相同的组ID**，被分配不同的订阅分区。当某个实例挂掉的时候，其他实例会自动地承担起它负责消费的分区。

此时，又有一个小技巧给到你：消费者组的题目，能够帮你在某种程度上掌控下面的面试方向。

1. 如果擅长位移值原理，就可以提一下消费者组的位移提交机制
2. 如果擅长Broker，可以提一下消费者组和Broker之间的交互
3. 如果擅长与消费者组完全不相关的Producer，就可以说：消费者组要消费的数据，完全来自于Producer端生产的消息。

### 1.2 在Kafka中，ZooKeeper的作用是什么？

目前，Kafka使用ZooKeeper存放集群元数据、成员管理、Controller选举，以及其它一些管理类任务。现在，Kafka也在去ZooKeeper。

* **存放集群元数据**：是指主题、分区等的所有数据都保存在ZooKeeper中，并且以它保存的数据为权威，其他节点的数据都要以ZooKeeper的保持对齐
* **成员管理**：指Broker节点的注册、注销以及属性变更等等
* **Controller选举**：选举集群Controller，而其他管理类任务包括但不限于主题删除、参数配置等

### 1.3 Kafka中位移(OffSet)的作用

在Kafka中，每个主题分区下的每条消息都被赋予一个唯一的ID数值，用于标识它在分区中的位置。**这个ID数值，就被称为位移，或者叫做偏移量**。一旦消息被写入到分区日志，它的位移值就不能被修改。

面试中，可以进一步介绍：

1. 如果深入了解 Broker底层日志写入的逻辑，可以强调消息在日志中的存放格式
2. 如果明白位移一旦写入就不能修改，可以强调“**Log Cleaner组件都不能影响位移值**”这件事
3. 如果对消费者的概念清晰，可以再详细说说**位移值和消费者位移值之间的区别** -- 估计也常常问

### 1.4 阐述Kafka中的领导者副本（Leader Replica）和追随者副本（Follower Replica）的区别

表面上这道题是考核对Leader和Follower的区别，但很容易引申到Kafka的同步机制。可以主动淡道它。大致如下：

Kafka副本，当前氛围领导者和追随者副本。**只有Leader副本才能对外提供<font color='red'>读写</font>服务**，相应Clients短的请求。Follower副本只是采用拉（Pull）的方式，**被动的**同步Leader副本中的数据，并且在Leader副本所在的Broker宕机后，随时准备参与应聘Leader副本。还要强调：

* **Follower副本也能对外提供读服务**。从Kafka2.4版本开始，社区通过引入新的Broker端的参数，允许Follower副本有限度的提供读服务
* **Leader和Follower的消息序列在实际场景中不一致**。很多原因可能造成Leader和Follower保存的消息序列不一致，比如程序的Bug、网络问题等等。这是很严重的错误，**必须要完全避免**。之前确保一致性的主要手段是**高水位机制**，但是高水位机制无法保证Leader连续变更场景下的数据一致性，因此，社区引入了**Leader Epoch机制**，用来修复高水位机制的弊端。

## 2. 实操题目

### 2.1 如何设置Kafka能接收的最大消息的大小？

这道题涉及到2个地方：消费者端和Broker端。

* Broker端参数：message.max.bytes、max.message.bytes(主题级别) 以及 replica.fetch.max.bytes
* Consumer端：fetch.message.max.bytes

Broker端的最后一个参数比较容易遗漏。我们必须要调整Follower副本能够接收的最大消息的大小，否则，副本同步就会说失败。

### 2.2 监控Kafka的框架有哪些？

下 面这些就是 Kafka 发展历程上比较有名气的监控系统。

* **Kafka Manager**：应该是最有名的专属Kafka监控框架了，是独立的监控系统
* **Kafka Monitor**：LinkedIn开源的免费框架，支持对集群进行系统监测，并实时监控测试结果
* **CruiseControl**：也是 LinkedIn 公司开源的监控框架，用于实时监测资源使用率，以及 提供常用运维操作等。无 UI 界面，只提供 REST API
* **JMX监控**：由于 Kafka 提供的监控指标都是基于 JMX 的，因此，市面上任何能够集成 JMX 的框架都可以使用，比如 Zabbix 和 Prometheus
* **JMXTool**：社区提供的命令行工具，能够实时监控 JMX 指标。答上这一条，属于绝对 的加分项，因为知道的人很少，而且会给人一种你对 Kafka 工具非常熟悉的感觉。如果 你暂时不了解它的用法，可以在命令行以无参数方式执行一下kafka-run-class.sh kafka.tools.JmxTool，学习下它的用法
* 其他大数据平台的监控体系：像 Cloudera 提供的 CDH 这类大数据平台，天然就提 供 Kafka 监控方案

### 2.3 Broker的Heap Size的设置

它是一类非常通用的面试题目。一旦你应对不当，面试方向很有可能被引到 JVM 和 GC 上去，那样的话，你被问住的几率就会增大。因此，建议简单地介绍一下 Heap Size 的设置方法，并把重点放在 Kafka Broker 堆大小设置的最佳实践上。

这是一种回复：**任何 Java 进程 JVM 堆大小的设置都需要仔细地进行考量和测试。一个常见的做法是，以默认的初始 JVM 堆大小运行程序，当系统达到稳定状态后，手动触发一次 Full GC，然后通过 JVM 工具查看 GC 后的存活对象大小。之后，将堆大小设置成存活对象总大小的 1.5~2 倍。对于 Kafka 而言，这个方法也是适用的。不过，业界有个最佳实践，那就是将 Broker 的 Heap Size 固定为 6GB。经过很多公司的验证，这个大 小是足够且良好的。**

### 2.4 如何估算Kafka集群的机器数量？

这道题目考查的是**机器数量和所用资源之间的关联关系**。所谓***资源，也就是 CPU、内存、磁盘和带宽***。

通常来说，CPU和内存资源的充足是比较容易保证的，因此，需要从磁盘空间和带宽占用两个维度去评估机器数量。

在预估磁盘的占用时，**一定不要忘记计算副本同步的开销**。如果一条消息占用1KB的磁盘空间，那么，在3个副本的中，就需要3K的总空间来保存这条消息。显式地，将这些考虑因素都罗列出来，能够彰显对问题思考的全面性。

对于评估带宽来说，常见的带宽有 *1Gbps* 和 *10Gbps*，但要切记，**这两个数字仅仅是最大值**。因此，最好要明确给定的带宽是多少。然后，明确阐述**当带宽占用接近总带宽的90%时**，丢包就会发生。

### 2.5 Leader总是-1，怎么破？

在生产环境中一定碰到过“某个主题分区不能工作了”的情形。使用命令行查看状态的话，会发现 Leader 是 -1，于是，你使用各种命令都无济于事，最后只能用“重启大法”。那么，有没有什么办法，可以不重启集群，就能解决此事呢?这就是此题的由来。

答案：**删除ZooKeeper节点/Controller，出发Controller重新选举。Controller重选能够为所有主题分区重刷分区状态，可以有效解决因不一致导致的Leader不可用问题。** 

## 3. 炫技式题目 

### 3.1 LEO、LSO、AR、ISR、HW都是什么含义？

* **LEO**：Log End Offset。**日志末端位移值**或**末端偏移量**。表示日志下一条待插入消息的位移值。举个例子：如果日志有10条信息，位移值从0开始，那么，第10条消息的位置值就是9，此时，LEO = 10
* **LSO**：Log Stable Offset。这是Kafka事务的概念。如果没有使用到事务，那么这个值不存在(其实也不是不存在，只是设置成一个无意义的值)。**该值控制了事务型消费者能够看到的消息范围**。它经常与 Log **Start** Offset，即日志起始位移相混淆，因为有些人将后者也缩写成LSO。在Kafka中，LSO专指 Log Stable Offset
* **AR**：Assigned Replicas。AR，是主题被创建后，分区创建时被分配的副本集合，副本个数由副本因子决定
* **ISR**：In-Sync Replicas。Kafka中特别重要的概念，**指代的是AR中那些与Leader保持同步的副本集合**。在AR中的副本可能不在ISR中，但**Leader副本天然就包含在ISR中**。
  * 关于ISR，还有一个常见的面试题是：如何判断副本是否应该属于ISR。目前的判断依据是：Follower副本的LEO落后 Leader LEO的时间，是否超过了Broker端参数replica.lag.time.max.ms值。如果超过了，副本就从ISR中移除。
* **HW**：高水位值(High watermark)。**这是控制消费者可读取消息范围的重要字段**。一个普通消费者只能“看到”Leader 副本上介于 Log Start Offset 和 HW(不含)之间的 所有消息。水位以上的消息是对消费者不可见的。
  * 关于HW，问法有很多，能想到的 最高级的问法，就是让你完整地梳理下 Follower 副本拉取 Leader 副本、执行同步机制 的详细步骤。这就是最后一道题的题目，一会儿会给出答案和解析。

### 3.2 __consumer_offsets是最什么用的？

这是一个**内部主题，不需要动手干预**，由Kafka自行管理。当然，也可以手动创建该主题。

它的主要作用是：**负责注册消费者以及保存位移值。**可能大家对于保存位移值的功能很熟悉，但是**其实该主题也是保存消费者元数据的地方。**另外，这里的消费者泛指消费者组和独立消费者，不仅仅是消费者组。

Kafka的GroupCoordinator组件提供对该主题的完整管理功能，包括该主题的创建、写入、读取和Leader维护等。

### 3.3 分区Leader选举策略有几种？

分区的 Leader 副本选举对用户是完全透明的，它是由 Controller 独立完成的。需要回答的是，在哪些场景下，需要执行分区 Leader 选举。每一种场景对应于一种选举策略。当前，Kafka 有 4 种分区 Leader 选举策略。

* **Offline Partition Leader**：每当有分区上线时，就需要执行Leader选举。所谓的分区上线，可能是创建了新分区，也可能是之前的下线分区重新上线。**这是最常见的分区Leader选举场景**

* **Reassign Partition Leader**：当手动运行 *kafka-reassign-partition* 命令时，或者是调用Admin的 alterPartitionReassignments 方法执行分区副本重分配时，可能触发此类选举。

  假设原来的AR是 *[1, 2, 3]* , Leader是 1 ，当执行副本重分配后，副本集合 AR 被设成了 *[4, 5, 6]* ，显然Leader必须要变更，此时会发生 Reassign Partition Leader 选举

* **Rreferred Replica Partiton Leader**：当手动运行 *kafka-preferred-replica-election* 命令，或自动触发了 Preferred Leader选举时，该类策略被激活。所谓的 Preferred Leader，指的是 AR 中的第一个副本。比如 AR 是 *[3, 2, 1]* ，那么Preferred Leader就是 3

* **Controlled Shutdown Partition Leader**：当Broker正常关闭时，该Broker上的所有Leader副本都会下线，因此，需要为受影响的分区执行相应的Leader选举

这4类选举策略的大致思想都是类似的，即从AR中挑选首个在ISR中的副本，作为新Leader。当然，个别策略有些微小差异。

### 3.4 Kafka 的哪些场景使用了零拷贝（Zero Copy）？

Zero Copy是特别容易被问到的高阶题目。在Kafka中，体现Zero Copy使用场景的地方**有两处：基于mmap的索引和日志文件读写所用的TransportLayer**

* **基于mmap的索引**：索引都是基于 ***MappedByteBuffer*** 的，也就是让用户态和内核态共享内核态的数据缓冲区，此时，数据不需要复制到用户空间。不过，mmap虽然避免了不必要的拷贝，但不一定就能保证很高的性能。在不同的操作系统下，mmap的创建和销毁成本可能是不一样的。很高的创建和销毁开销会抵消Zero Copy带来的性能优势。由于这种不确定性，在Kafka中，**只有索引应用了mmap**，最核心的日志并未使用mmap机制
* **TransportLayer**：TransportLayer 是 Kafka 传输层的接口。它的某个实现类使用了 ***FileChannel*** 的 **transferTo** 方法。该方法底层使用 **sendfile** 实现了Zero Copy。对Kafka而言，如果 *I/O* 通道使用普通的 PLAINTEXT，那么kafka 就可利用Zero Copy 特性，**直接将页缓存中的数据发送到网卡的Buffer中**，避免中间的多次拷贝。相反，如果 *I/O* 通道**启用了SSL，那么Kafka便无法利用Zero Copy特性了**

## 4. 深度思考题

### 4.1 Kafka 为什么不支持读写分离？

这道题考察的目标是对Leader/Follower模型的思考。

Leader/Follower 模型**并没有**规定 Follower 副本不可以对外提供读服务。很多框架都是允许这么做的，只是Kafka最初为了避免不一致性的问题，而采用了让 Leader 统一提供服务的方式。

不过，**在Kafka 2.4 后**，Kafka 提供了有限度的读写分离，也就是说，**Follower 副本能够对外提供读服务**。

下面再给出 2.4 版本之前不支持读写分离的理由：

* 场景不适用：读写分离适用于那种读负载很大，而写操作相对不频繁的场景。显然 Kafka 不属于这个场景
* 同步机制：Kafka采用 **PULL** 方式实现 Follower 的同步。因此，**Follower与Leader存在不一致性窗口**。如果允许读Follower副本，就势必要处理消息滞后（Lagging）的问题 - *【貌似读写分离都存在这样的问题】*

### 4.2 如何调优Kafka？

任何调优的问题，第一步，都**要明确优化目标，并且定量给出目标！** 这点特别重要。对于kafka而言，***常见的优化目标是：吞吐量、延时、持久化和可用性***。这些目标的优化思路都是不同的，甚至是相反的。

确定了目标后，还要明确优化的维度。有些调优属于通用的优化思路，比如对操作系统、JVM等的优化；有些则是针对性的，比如要优化Kafka的TPS等。从3个方向去考虑：

* **Producer端**：增加 batch.size、linger.ms， 启用压缩，关闭重试等
* **Broker端**：增加 num.replica.fetchers，提升 Follower 同步TPS，避免  Broker Full GC等
* **Consumer端**：增加 fetch.min.bytes 等

### 4.3 Contoller发生网络分区(Network Partitioning)时，Kafka会怎么样？

这道题目能够诱发对分布式系统设计、CAP 理论、一致性等多方面的思考。不过，针对故障定位和分析的这类问题。建议你首先言明“实用至上”的观点，即：不论怎么进行理论分析，永远都要以实际结果为准。一旦发生 Controller 网络分区，那么，**第一要务就是 查看集群是否出现“脑裂”**，即同时出现两个甚至是多个 Controller 组件。这可以根据 Broker 端监控指标 ActiveControllerCount 来判断。

现在分析下，一旦出现这种情况，Kafka 会怎么样。

由于Controller会给 Broker 发送3类请求：LeaderAndIsrRequest、StopReplicaRequest和 UpdateMetadataRequest，因此，一旦出现网络分区，这些请求将不能顺利到达Broker端。这将影响主题的创建、修改、删除操作的信息同步，表现为集群方佛僵住了一样，无法感知到后面的所有操作。因此，网络分区通常都是非常严重的问题。

### 4.4 Java Consumer为什么采用单线程来获取消息？

实际上，**Java Consumer是双线程的设计。一个线程是用户主线程，负责获取消息；另一个线程时心跳线程，负责向Kafka汇报消费者存活情况。** 给心跳单独一个专属线程，能够有效的规避因消息处理速度慢而被视为下线的“假死”情况。

**单线程获取消息的设计能够避免阻塞式的消息获取方式**。单线程轮询方式容易实现异步非阻塞式，这样便于将消费者扩展成支持实时流处理的操作算子。因为很多实时流处理操作算子都不能是阻塞式的。另一个可能的好处是，可以简化代码的开发。多线程交互的代码是很容易出错的。

### 4.5 简述Follower副本消息同步的完整流程

首先，Follower发送FETCH请求给Leader；接着，Leader会读取底层日志文件中的消息数据，再更新它内存中的 Follower 副本的LEO值，更新为 FETCH 请求中的 fetchOffset 值。最后，尝试更新分区高水位值。Follower 接收到FETCH 相应之后，会把消息写入底层日志，接着更新 LEO 和 HW 值。

Leader 和 Follower 的 HW 值更新时机是不同的，Follower 的 HW 更新永远落后于 Leader 的 HW。这种时间上的错配是造成各种不一致的原因。

