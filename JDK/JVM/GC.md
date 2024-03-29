# GC的一些常用问题

原文：https://www.zhihu.com/question/27339390



### 1. GC的两种判定方法

* 引用计数算法
* 可达性分析算法

### 2. 什么是分代回收？

不同对象生命周期不同，所以可以采取不同的回收方式，从而提高回收效率。可以分别为新生代、老年代进行垃圾回收。

### 3. GC原理是什么？JVM怎么回收内存？

从标记阶段到清除阶段。标记阶段进行相关存活对象的标记，紧接着在清除阶段将为被标记的对象进行回收清除。

### 4. 垃圾回收算法各自的优缺点是什么？

* 引用计数法：算法实现简单，效率高。但是不能解决循环引用的问题，导致内存泄漏
* 标记清楚法：算法实现简单，能够解决循环引用的问题。但是会产生内存碎片，因此需要维护一个空闲列表
* 标记清理法：该算法不会产生内存碎片，不会消耗2倍内存，但是效率低下
* 复制算法：效率高，不会产生内存碎片，但是需要消耗两倍内存，占用较大

### 5. G1的应用场景，是如何搭配使用垃圾回收器的？

面向服务端应用，针对具有大内存、多处理器的机器。

最主要的应用是，需要低GC延迟，并且具有大堆的应用程序提供解决方法。

### 6. 垃圾回收算法的实现原理

* **引用计数算法**：对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就+1；当引用失效时，引用计数器就-1。只要对象A的引用计数器的值为0，即表示对象A不可能再被使用，可以进行回收
* **可达性分析算法**：根据GC Roots 开始向下遍历整个对象树，如果一个对象没有任何引用链，则将会被回收
* **标记清除算法**：从引用跟节点开始遍历，标记所有被引用的对象，一般是在对象的Header中记录为可达对象；再对堆内存**从头到尾进行线性的遍历**，如果发现某个对象在其Header中没有标记为可达对象，则可以回收
* **复制算法**：将内存空间分为两块，每次只使用期中一块。在垃圾回收时将正在使用的内存中的存活对象复制到未使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收
* **标记压缩算法**：第一阶段和标记-清除算法一样，从根节点开始标记所有被引用的对象；第二阶段将所有的存活对象压缩到内存的一端，按顺序排放；最后，清理边界外所有的空间

### 7. GC Roots 有哪些？

* 虚拟机栈的栈帧的局部变量表中的引用
* 本地方法栈的JNI引用对象
* 方法区中类的静态属性
* 方法区中的常量
* 被同步锁 synchronized 持有的对象
* 对于分区的G1，可能会存在将另一个区域的对象也临时作为GC Roots

### 8. CMS解决了什么问题？说一下回收的过程？

CMS，全称 Concurrent Low Pause Collector，是jdk1.4后期版本开始引入的新gc算法，在jdk5和jdk6中得到了进一步改进。CMS主打低延迟，以快速响应用户为目标，可以和用户线程同时执行，提高用户体验。整个回收过程如下：

1. 初始标记：仅标记出 GC Roots 能直接关联到的对象
2. 并发标记：从 GC Roots 开始遍历整个对象引用链
3. 重新标记：修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录
4. 并发清除：清理删除标记阶段判断的已经死亡的对象

### 9. CMS 回收停顿了几次？为何要停顿？

停顿两次。因为CMS虽然是主打低延迟的，但是初始标记阶段和重新标记阶段不得不暂停用户线程，初始标记阶段的停顿以方便后续GC线程与用户线程并发的执行后续的间接标记存活对象，重新标记阶段的停顿是为了彻底标记之前被遗漏的部分。

### 10. JVM有哪些回收器？

* 串行回收器：Serial、Serial Old
* 并行回收器：ParNew、Parallel、Parallel Old
* 并发回收器：CMS、G1
* OpenJDK 12 的 Shennandoash，主打低延迟
* JDK11的 ZGC，可伸缩的低延迟垃圾回收器

按照对象所处的年代分，可以这样划分：

* 新生代收集器：串行Serial、并行ParNew、并行 Parallel
* 老年代收集器：串行 Serial Old、并发 CMS、并行Parallel Old

新生代收集器用于回收新生代，老年代收集器用于回收老年代

按JDK版本划分：

- jdk1.7 默认垃圾收集器Parallel Scavenge（新生代【标记-复制算法】）+Parallel Old（老年代【标记整理算法】）
- jdk1.8 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代）
- jdk1.9 默认垃圾收集器G1（Garbage First Garbage Collector）【从局部(两个Region之间)来看是基于"标记—复制"算法实现，从整体来看是基于"标记-整理"算法实现】