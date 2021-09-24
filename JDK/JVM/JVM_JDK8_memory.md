# JDK8 JVM的内存分配

从JDK8开始，JVM废弃了永久代（PermGen），使用了元空间（metaspace）

## 背景

### JDK8之前的JVM永久代（PermGen）在哪里

JDK8之前的HotSpot JVM的结构（*虚拟机栈与本地方法栈合在一起*）如下：

![](./images/images_JDK8_memory/Hotspot_JVM_Before8.png)

​                                                            HotSpot JVM结构（JDK8以前）

从上图可以看出，方法区（Method Area）与Heap堆都是线程共享的内存区域。

在HotSpot JVM中，就来讨论**永久代**，就是上图的方法区（JVM规范中称为方法区，左上角的绿色方块）。《Java虚拟机规范》只是规定了有方法区这个概念和它的作用，并没有规定如何去实现它。所以，在其他的JVM（比如JRockit VM，IBM J9 VM）不存在永久代。

### JDK8 JVM的内存模型

JDK8的内存模型的变化：

![](./images/images_JDK8_memory/HotSpot_JVM_JDK8.jpg)

​                                                            JDK8 JVM的内存结构

1. 新生代：Eden + From Servivor + To Servivor
2. 老年代：OldGen
3. 永久代（方法区实现）： PermGen 被MetaSpace（本地内存）替代

## 为什么废弃永久代

### 官方动机

移除永久代是为了融合HotSpot JVM与JRockit VM而做的努力，因为JRockit没有永久代，不需要配置。

原文如下：

> This is part of the JRockit and Hotspot convergence effort. JRockit customers do not need to configure the permanent generation (since JRockit does not have a permanent generation) and are accustomed to not configuring the permanent generation.

### 现实原因

我们以前经常遇到的OOM（java.lang.OutOfMemoryErr:PermGen）就是因为永久代不够用，或者有内存泄露造成的。

## 理解元空间

### 元空间大小

方法区主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于**堆**的一部分，但是为了与堆进行区分，通常又称为“非堆”。

**元空间的本质**和永久代类似，**都是对JVM规范中方法区的实现**。不过，元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用了本地内存**。所以，元空间可以使用的内存大小，理论上，取决于32位/64位系统可虚拟的内存大小。

#### 常用的配置参数

##### MetaspaceSize

**初始Metaspace大小** 。此参数用于设置初始化的metaspace大小，控制元空间发生GC的阈值。GC后，动态增加或降低Metaspace Size。在默认的情况下，这个值大小根据不同的平台在12M到20M浮动。使用 

```shell
java -XX:+PrintFlagsInitial
```

 命令，可以查看机器上初始化的参数值。

##### MaxMetaspaceSize

**最大Metaspace大小** 。 此参数用于设置Metaspace增长的上限，防止因为某些情况导致Metaspace无限地使用本地内存，影响到其他程序。默认的值为4294967295B（大约4096MB）。

##### MinMetaspaceFreeRatio

**最小空闲比** 。 当进行过**Metaspace GC后**，会计算当前Metaspace的**空闲空间比**。如果空闲比 **小于\(\<\)** 这个参数值（即实际非空闲占比过大，内存不够用），那么JVM将增长Metaspace大小。**默认值是40，即40%。**

设置该参数可以控制Metaspace的增长速度，太小的值会导致Metaspace增长缓慢，Metaspace的使用逐渐趋于饱和，可能会影响之后类的加载。而太大又会导致Metaspace增长过快，浪费内存。

##### MaxMetaspaceFreeRatio

**最大空闲比** ，同MixMetaspaceFreeRatio作用类似。GC之后计算了空闲比，如果比值 **大于\(\>\)** 这个参数值，那么JMV将会释放Metaspace的部分空间。**默认值是70，即70%。**

##### MaxMetaspaceExpansion

Metaspace增长时的最大幅度，即每次增长的内存大小不能 **超过\(\>\)** 这个参数设置的值。默认值是5452592B（大约为5MB）。

##### MinMetaspaceExpansion

Metaspace增长时的最小幅度。即每次增长的内存大小不能 **小于\(\<\)** 这个参数设置的值。默认值为340784B（大约330KB为）。