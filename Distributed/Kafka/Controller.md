# Kafka的控制器controller详解

原文：https://www.cnblogs.com/qinchaofeng/p/13706965.html



## 一 控制器简介

​        控制器组件（Controller），是 Apache Kafka 的核心组件。它的主要作用是在 Apache ZooKeeper 的帮助下，管理和协调整个 Kafka 集群。集群中任意一台 Broker 都能充当控制器的角色，但是，在运行过程中，只能有一个 Broker 称为控制器，行使其管理和协调的职责。换句话说，每个正常运行的 Kafka 集群，在任意时刻都有且只有一个控制器。

​        官网上有个名为 activeController 的JMX 指标，可以帮助实时监控控制器的存活状态。这个 JMX 指标非常关键，在实际运维操作过程中，一定要实时查看这个指标的值。

​        下面介绍控制器（Controller）的内部原理和运行机制。

## 二 控制器的原理和内部运行机制

### 2.1 ZooKeeper 介绍

​        开始之前，需要先介绍 Apache ZooKeeper，因为控制器是重度依赖ZooKeeper的。因此，有必要花一些时间学习下 ZooKeeper。

​        ZooKeeper 是**一个提供高可靠性的分布式协调服务框架**。它使用的数据模型类似于文件系统的树形结构，根目录也是以 “/” 开始。该结构上的每个节点被称为 ***znode***，用来保存一些元数据协调信息。如果以 znode 持久化来划分，znode 可以分为“***持久性 znode***” 和“***临时性 znode***”。持久性 znode 不会因为 ZooKeeper 集群重启而消失，而临时 znode 则与创建该 znode 的ZooKeeper 会话绑定，一定会话结束，该节点会被自动删除。

​        ZooKeeper 赋予客户端监控 znode 变更的能力，即所谓的 watch 通知功能。一旦 znode 节点被创建、删除、子节点数量发生变化，亦或是 znode 所存的数据本身变更，ZooKeeper 会通过节点变更监听器（ChangeHandler）的方式显式通知客户端。

​        依托于这些功能，ZooKeeper 常被用来实现集群成员管理、分布式锁、领导者选举等功能。Kafka 控制器大量使用 watch 功能实现对集群的协调管理。下图，展示的是 Kafka 在 ZooKeeper 中创建的 znode 分布。暂时不用去理解每个 znode 的作用，但大致能体会到 Kafka 对 ZooKeeper 的依赖。

![1](./images/Controller/1.webp)

### 2.2 控制器是如何被选出来的

​        那么控制器是如何被选出来的呢？前面提过，每台 Broker 都能充当控制器，那么，当集群启动后，Kafka 会怎么确认控制器位于哪台 Broker 呢？

​        实际上，Broker 启动时，会尝试去 ZooKeeper 中创建 /controller 节点。Kafka 当前选举控制器的原则是：第一个成功创建 /controller 节点的 Broker 会被指定为控制器。

### 2.2  控制器的作用

​        控制器是起协调作用的组件。那么，协调作用到底指的是什么呢？大致可以分为以下5种协调作用。

####  1 主题管理

​        **创建、删除、增加分区**

​        这里的主题管理，就是指控制器完成对 kafka 主题的创建、删除以及分区增加的操作。换句话说，当执行kafka-topics脚本时，大部分的后台工作都是控制器来完成的。*详细了解kafka-topics脚本*

#### 2 分区重分配

### 2.3 控制器保存了什么数据

### 2.4 控制器内部设计原理

## 三 社区工作