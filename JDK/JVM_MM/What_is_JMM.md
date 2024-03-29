## Java 内存模型是什么？解决什么问题？如何解决？

原文：https://blog.csdn.net/weixin_42706227/article/details/88636277



### Java MM是什么？

很简单，Java内存模型不是实际存在的与硬件相关的东西，它是一种共享内存系统中多线程程序读写操作行为的规范、一种标准。通过这些规则来规范对内存的读写操作，从而保证指令执行的正确性。它与处理器有关、与缓存有关、与并发有关、与编译器有关。

### Java MM解决了什么问题？

它解决了CPU多级缓存、处理器优化、指令重排等导致的内存访问问题，保证了并发场景下的**一致性、原子性和有序性**。

Java MM规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中使用到的变量的主内存副本拷贝，**线程对变量的所有操作都必须在工作内存中进行**，而不能直接读写主内存。不同的线程之间无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。

而**JMM就作用于工作内存和主存之间数据同步过程**。它规定了如何做数据同步以及什么时候做数据同步。

![1](.\images\What_is_JMM\1.png)

### Java MM 是如何解决的？

在Java中提供了一系列和并发处理相关的关键字，比如 volatile、synchronized、final、concurrent 包等。这些就是 Java 内存模型封装了底层的实现后提供给程序员使用的一些关键字。

在开发多线程的代码时，可以直接使用 synchronized 等关键字来控制并发，从来就不需要关心底层的编译器优化、缓存一致性等问题。所以，Java MM除了定义一套规范，还提供了一系列原语，封装了底层实现后，供开发者直接使用。

#### 原子性

在Java中，为了保证原子性，提供了两个高级的字节码指令 monitorenter 和 monitorexit。在 synchronized 的实现原理文章中，介绍过这两个字节码，在 Java 中对应的关键字就是 synchronized。

因此，在 Java 中可以使用 Synchronized 来保证方法和代码块内的操作时原子性的。

#### 可见性

Java内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值的这种依赖主内存作为传递媒介的方式来实现的。

Java中的 volatile 关键字提供了一个功能，那就是**被其修饰的变量在被修改后可以立即同步到主内存，被其修饰的变量在每次使用之前都从主内存刷新**。因此，可以使用 volatile 来保证多线程操作时变量的可见性。

除了 volatile，Java 中的 synchronized 和 final 两个关键字也可以实现可见性。只不过实现的方式不同。

#### 有序性

在 Java 中，可以使用 synchronized 和 volatile 来保证多线程之间操作的有序性。实现方式有所区别：

* volatile 会禁止指令重排
* synchronized 保证同一时刻只允许一条线程操作