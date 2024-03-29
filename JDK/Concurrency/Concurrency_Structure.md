# Java并发结构

原文：http://ifeve.com/java-concurrency-constructs/

## 线程

线程是一个独立执行的调用序列，同一个进程的线程在同一时刻共享一些系统资源（比如文件句柄等），也能访问同一个进程所创建的对象资源（如内存资源）。

java.lang.Thread对象负责统计和控制这种行为。

每个程序都至少拥有一个线程-即作为Java虚拟机启动参数运行在主类main方法的线程。在JVM初始化过程中，也可能启动其他的后台线程。这种线程的数目和种类，因JVM的实现而异。然而，所有用户级线程，都是显式被构造并在主线程或者其他用户线程中被启动。

这里对Thread类中的主要方法和属性，以及一些使用注意事项作出总结。这些内容在书《Java Concurrency Constructs》上进行进一步的探讨阐述。



### 构造方法

Thread类中不同的构造方法接受如下参数的不同组合：

* 一个Runnable对象：这种情况，Thread.start方法将会调用对应Runnable对象的run方法。如果没有提供Runnable对象，那么就会立即得到一个Thread.run的默认实现
* 一个作为线程标识名的String字符串：该标识在跟踪和调试过程中会非常有用，除此别无他用
* 线程组(ThreadGroup)：用来放置新创建的线程，如果提供的ThreadGroup不允许被访问，那么就会抛出一个SecurityException

Thread类本身就已经实现了Runnable接口，因此，除了提供一个用于执行的Runnable对象作为构造参数的办法之外，也可以创建一个Thread的子类，通过重写其run方法来达到同样的效果。然而，**比较好的实践方式确实分开定一个Runnable对象并用来作为构造方法的参数。**将代码分散在不同的类中，使得开发人员不需纠结于Runnable和Thread对象中使用的同步方法或同步块之间的内部交互。更普遍的是，这种分隔使得对操作的本身与其运行的上下文有着独立的控制。更好的是，同一个Runnable对象可以同时用来初始化其他的线程，也可以用于构造一些轻量化的执行框架（Executors）。另外需要提到的是通过继承Thread类实现线程的方式有一个缺点：使得该类无法再继承其他的类。

Thread对象拥有一个守护（Daemon）标识属性，这个属性无法在构造方法中被赋值，但是可以在线程启动之前设置该属性（通过setDaemon方法）。当程序中所有的非守护线程都已经终止，调用setDaemon方法可能会导致虚拟机粗暴的终止线程并退出。isDaemon方法能够返回该属性的值。守护状态的作用非常有限，即使是后台线程在程序退出的时候也经常需要做一些清理工作。



### 启动线程

调用start方法会触发Thread实例以一个新的线程启动其run方法。新线程不会持有调用线程的任何同步锁。

当一个线程正常地运行结束，或者抛出某种未检测的异常（比如：运行时异常RunningException、错误ERROR或者其子类），线程就会终止。当线程终止之后，是不能被重新启动的。在同一个Thread上调用多次start方法会抛出InvalidThreadStateException异常。

如果线程已经启动但是还没有终止，那么调用isAlive方法就会返回true，即使线程由于某些原因处于阻塞（Blocked）状态，该方法依然返回True。如果线程已经被取消（cancelled），那么该方法在什么时候返回false，就因各JVM的实现差异了。没有方法可以得知一个处于非活跃状态的线程是否已经被启动过（即：线程在开始运行前和结束运行后都会被返回false，无法得知处于false的线程具体的状态）。另外，虽然一个线程能够得知同一个线程组的其他线程的标识，但是却无法得知自己是由哪个线程调用启动的。



### 优先级

JVM为了实现跨平台的特性，Java语言在线程调用与调度公平性上未作出任何的承诺，甚至都不会严格保证线程会被执行。但是Java线程却支持优先级的方法，这些方法会影响线程的调度。

每个线程都有一个优先级，分布在Thread.MIN_PRIORITY和Thread.MAX_PRIORITY之间（分别是1和10）。默认情况下，新创建的线程都拥有和创建它的线程相同的优先级。main方法所关联的初始化线程拥有一个默认的优先级，这个优先级是Thread.NORM_PRIORITY(5)。 可以通过getPriority方法获得线程的优先级，也可以通过setPriority方法来动态修改。**一个线程的最高优先级有其所在的线程组限定**。

当可运行的线程数超过了可用的CPU数目时，线程调度器更偏向于去执行那些拥有更高优先级的线程。具体的策略因平台而异。比如有些JVM实现总是选择当前优先级最高的线程去执行；有些JVM将Java中的10个优先级映射到系统所支持的最小范围的优先级上，因此，拥有不同的优先级的线程可能最终被同等对待；还有些JVM会使用老化策略（随着时间的增长，线程的优先级逐渐升高）动态调整线程优先级；另一些JVM的调度策略会确保低优先级的线程最终还是能够有机会运行。设置线程优先级可以影响在同一台机器上运行的程序之间的调度结果，但是这不是必须的。

线程优先级对语义和正确性没有任何的影响。特别是，优先级管理不能用来代替锁机制。优先级仅仅是用来表明哪些线程是重要紧急的，当存在很多线程在激烈进行CPU资源竞争的情况下，线程的优先级标识将会显得非常有用。比如：在ParticleApplet中将particle animation线程的优先级设置的比创建它们的applet线程低，在某些系统上能够提高对鼠标点击的响应，而且不会对其他功能造成影响。但是即使setPriority方法被定义为空实现，程序在设计上也应该保证能够正确执行（尽管可能会没有响应）。

下面这个表格列出了不同类型任务在线程优先级设定上的通常约定。在很多并发应用中，在任一指定的时间点上，只有相对较少的线程处于可执行的状态（另外的线程可能由于各种原因处于阻塞状态），在这种情况下，没有什么理由需要去管理线程的优先级。另外一些情况下，在线程优先级上的调整可能会对并发系统的调优起到一些作用。

| 范围 | 用途                                                        |
| ---- | ----------------------------------------------------------- |
| 10   | Crisis management (应急处理)                                |
| 7-9  | Interactive, event-driven（交互相关，事件驱动）             |
| 4-6  | IO-bound （IO限制类）                                       |
| 2-3  | Background Computation （后台计算）                         |
| 1    | Run only if nothing else can （尽在没有任何线程运行时运行） |



### 控制方法

只有很少几个方法可以用于**跨线程交流**。

* 每个线程都有一个相关的Boolean类型的中断标识。在线程 t 上调用 t.interrupt() 会将该线程的中断标识设置为true，除非线程 t 正处于 Object.wait(), Thread.sleep()，或者Thread.join()。这些情况下，interrupt()调用会导致 t 上的这些操作抛出 InterruptedException 异常，但是 t 的中断标识会被设置为false
* 任何一个线程的中断状态都可以通过调用 isInterrupted() 方法来得到。如果线程已经通过 interrupt() 方法被中断，这个方法将会返回 true
* 但是，如果调用了Thread.interrupted() 方法且中断标识还没有被重置，或者是线程处于wait，sleep，join过程中，调用 isInterrupted() 方法将会抛出 InterruptedException 异常。调用 t.join() 方法将会暂停执行调用线程，直到线程t执行完毕：当 t.isAlive() 方法返回false的时候调用 t.join() 将会直接返回(return)。另一个带参数毫秒(millisecond)的join方法在被调用时，如果线程没能够在指定的时间内完成，调用线程将重新得到控制权。因为isAlive方法的实现原理，所以在一个还没有启动的线程上调用join方法是没有任何意义的。同样的，试图在一个还没有创建的线程上调用join方法也是不明智的

起初，Thread类还支持一些控制方法：suspend,resume,stop以及destroy。这几个方法已经被声明为过期了。其中，destroy() 方法从来没有被实现，估计以后也不会。而通过使用等待/唤醒机制增加suspend和resume方法在安全性和可靠性的效果有所欠缺。后续会讨论这些问题。

### 静态方法

Thread类中的部分方法被设计为**只适用于当前正在运行的线程**（即调用Thread方法的线程）。为了强调这点，这些方法被声明为静态的。

* Thread.currentThread()：返回当前线程的引用，得到这个引用可以用来调用其他的非静态方法。比如Thread.currentThread().getPriority()来得到当前线程的优先级

* Thread.interrupted()：该方法会清除当前线程的中断状态并返回前一个状态。（一个线程中断状态是不允许被其他线程清楚的）

* Thread.sleep(long msecs)：该方法会使得当前线程暂停执行**至少**msecs毫秒

* Thread.yield()：该方法纯粹只是建议JVM对其他已经处于就绪状态的线程（如果有的话）调用执行，而不是当前线程。最终JVM如何去实现这种行为就要看具体JVM的实现了。

  * 尽管缺乏保障，但在不支持分时间片/可抢占式的线程调用的单CPU的JVM实现上，yield()方法依然能够起切实的作用的。在这种情况下，线程只在被阻塞的情况下（比如等待I/O，或是调用了sleep（）等）才会进行重新调度。在这些系统上，那些执行非阻塞的耗时的计算任务的线程就会占用CPU很长时间，最终导致应用的响应能力降低。如果一个非阻塞的耗时计算线程会导致时间处理线程或者其他交互线程超出可容忍的限度的话，就可以在其中插入yield操作(或者是sleep)，使得具有较低线程优先级的线程也可以执行。为了避免不必要的影响，你可以只在偶然间调用yield方法，比如，可以在一个循环中插入如下代码：

    ```java
    if (Math.random() < 0.01) Thread.yield()
    ```

  * 在支持可抢占式调度的Java虚拟机实现上，线程调度器忽略yield操作，特别是在多核处理器上



### 线程组

每一个线程都是一个线程组的成员。默认情况下，新建线程和创建它的线程属于同一个线程组。**线程组是以树状分布的**。当创建一个新的线程组，这个线程组成为当前线程组的子组。*getThreadGroup()* 方法会返回当前线程所属的线程组，对应地，ThreadGroup类也有方法可以得到哪些线程目前属于这个线程组，比如 *enumerate()* 方法。

ThreadGroup类存在的一个目的，是支持安全策略来动态的限制对该组的线程操作。比如，对不属于同一组的线程调用 *interrupt()* 是不合法的。这是为了避免某些问题（比如，一个applet线程尝试杀掉主屏幕的刷新线程）所采取的措施。ThreadGroup也可以为该组所有线程设置一个最大的线程优先级。

**线程组往往不会直接在程序中被使用**。在大多数的应用中，如果仅仅是为在程序中跟踪线程对象的分组，那么普通的集合类（比如java.util.Vector）应是更好的选择。

在ThreadGroup类为数不多的几个方法中，*uncaughtException()* 方法却是非常有用的，当线程组中的某个线程因抛出未检测的异常（比如空指针异常NullPointerException）而中断的时候，调用这个方法可以打印出线程的调用栈信息。

## 同步

### 对象与锁

**每一个Object类及其子类的实例都拥有一个锁**。其中，标量类型int，float等不是对象类型，但是标量类型可以通过其包装类来作为锁。单独的成员变量是不能被标明为同步的。**锁，只能用在使用了这些变量的方法上**。成员变量可以被声明为*volatile*，这种方式会影响该变量的**原子性，可见性以及排序性**。

类似的，持有标量变量元素的数组对象拥有锁，但是其中的标量元素却不拥有锁。（也就是说，没有办法将数组成员声明为volatile类型的）。如果锁住了一个数组并不代表其数组成员都可以被原子的锁定。也没有能在一个原子操作中锁住多个对象的方法。

***Class实例***本质上是个对象。而在*静态*同步方法中用的就是**类对象的锁**。

### 同步方法 和 同步块

使用synchronized 关键字，有两种语法结构：*同步代码块*和*同步方法*。同步代码块需要提供一个作为锁的对象参数。这就允许了任意方法可以去锁任意一个对象。但在同步代码块中使用的最普通的参数却是*this*。

同步代码块被认为比同步方法更加的基础。如下两种声明是等同的：

```java
synchronized void f() { 
  /* body */ 
}

void f() { 
  synchronized(this) { 
    /* body */ 
  } 
}
```

**synchronized 关键字并不是方法签名的一部分。**所以当子类覆写父类中的同步方法或是接口中声明的同步方法的时候，**synchronized修饰符是不会被自动继承的**，另外，构造方法不可能是真正同步的（尽管可以在构造方法中使用同步块）。

同步实例方法在其子类和父类中使用同样的锁，但是内部类方法的同步却独立于其外部类。然而一个非静态的内部类方法可以通过下面这种方式锁住其外部类：

```java
synchronized(OuterClass.this) { 
  /* body */ 
}
```

### 等待锁 与 释放锁

使用synchronized关键字须遵循一套内置的锁等待-释放机制。**所有的锁都是块结构的**。当进入一个同步方法或同步块的时候必须获得该锁，而退出的时候（即使是异常退出）必须释放这个锁。不能忘记释放锁。

**锁操作是建立在独立的线程上的**而不是独立的调用基础上。一个线程能够进入一个同步代码的条件是当前锁未被占用或者是当前线程已经占用了这个锁，否则线程就会阻塞住。（这种可重入锁或是递归锁不同于POSIX线程）。这就允许一个同步方法可以去直接调用同一个锁管理的另一个同步方法，而不需要被冻结（注：即不需要再经历释放锁-阻塞-申请锁的过程）。

同步方法或同步块遵循这种锁获取/锁释放的机制有一个前提，那就是所有的同步方法或同步块都是在**同一个锁对象上**。如果一个同步方法正在执行中，其他的非同步方法也可以在任何时候执行。也就是说，同步不等于原子性，但是同步机制可以用来实现原子性。

当一个线程释放锁的时候，另一个线程可能正等待这个锁（也可能是同一个线程，因为这个线程可能需要进入另一个同步方法）。但是关于哪一个线程能够紧接着获得这个锁以及什么时候，这是没有任何保证的。（也就是，没有任何的公平性保证）另外，没有什么办法能够得到一个给定的锁正被哪个线程拥有着。

除了锁控制之外，同步也会对底层的内存系统带来副作用。

### 静态变量/方法 

锁住一个对象并不会原子性的保护该对象类或其父类的静态成员变量，而应该**通过同步的静态方法或代码块来保证访问一个静态的成员变量**。静态同步使用的是静态方法锁声明的类对象所拥有的锁。类C的静态锁可以通过内置的实例方法获取到：

```java
synchronized(C.class) { 
    /* body */ 
}
```

每个类所对应的静态锁和其他的类（包括其父类）没有任何的关系。通过在子类中增加一个静态同步方法来试图保护父类中的静态成员变量是无效的。应使用显式的代码块来代替。如下这种方式也是一种<font color='red'>**不好**</font>的实践：

```java
synchronized(getClass()) {
    /* body */ 
} // Do not use
```

这种方式，可能锁住的实际中的类，并不是需要保护的静态成员变量所对应的类（有可能是其子类）。

JVM在类加载和类初始化阶段，内部获得并释放类锁。除非你要去写一个特殊的类加载器或者需要使用多个锁来控制静态初始顺序，这些内部机制不应该干扰普通类对象的同步方法和同步块的使用。JVM没有什么内部操作可以独立的获取你创建和使用的类对象的锁。然而当你继承java.*的类的时候，你需要特别小心这些类中使用的锁机制。

## 监视器

正如每个对象都有一个锁一样，每一个对象同时拥有一个由这些方法(wait,notify,notifyAll,Thread,interrupt)管理的一个等待集合。拥有锁和等待集合的实体通常被称为监视器（虽然每种语言定义的细节略有不同），任何一个对象都可以作为一个监视器。

对象的等待集合，是由Java虚拟机来管理的。每个等待集合上都持有在当前对象上等待尚未被唤醒或是释放的阻塞线程。

因为与等待集合交互的方法（wait，notify，notifyAll）只在拥有目标对象的锁的情况下才被调用，因此无法在编译阶段验证其正确性，但在运行阶段错误的操作会导致抛出IllegalMonitorStateException异常。

这些方法的操作描述如下：

**Wait**

调用wait方法会产生如下操作：

* 如当前线程已经终止，则这个方法会立即退出并抛出一个InterruptedExeption异常；否则当前线程就**进入阻塞状态**
* Java虚拟机将该线程放置在目标对象的等待集合中
* **释放目标对象的同步锁**，除此之外的其他锁依然由该线程所有。即使是在目标对象上多次嵌套的同步调用，所持有的可重入锁也会完整的释放。这样，后面恢复的时候，当前的锁状态能够完全的恢复

**Notify**

调用notify会产生如下操作：

* JVM从目标对象的等待集合中**随机选择**一个线程（称为T，前提是等待集合中还存在一个或多个线程）并从等待集合中移出T。当等待集合中存在多个线程时，并没有机制保证哪个线程会被选择到
* 线程T必须重新获得目标对象的锁，直到有线程调用notify释放该锁，否则线程会一直阻塞下去。如果其他线程先一步获得了该锁，那么线程T将继续进入阻塞状态
* 线程T从之前wait的点开始继续执行

**NotifyAll**

notifyAll方法与notify方法的运行机制是一样的，只是这些过程是在对象等待集合中的所有线程上发生（事实上，是同时发生）的。但是因为这些线程都需要获得同一个锁，最终也只能有一个线程继续执行下去

**Interrupt**

如果在一个因wait而中断的线程上调用Thread.interrupt方法，之后的处理机制和notify机制相同，只是在重新获取这个锁之后，该方法将会抛出一个InterruptedException异常并且线程的中断标识将被设为false。如果interrupt操作和一个notify操作在同一时间发生，那么不能保证哪个操作先被执行，因此任何一个结果都是可能。（JLS的未来版本可能会对这些操作结果提供确定性保证）

**Timed Wait**（定时等待）

定时版本的wait方法，wait(long mesecs)和wait(long msecs, int nanosecs)，参数指定了需要在等待集合中等待的最大时间值。如果在时间限制之内没有被唤醒，它将自动释放，除此之外，其他的操作都和无参数的wait方法一样。并没有状态能够表明线程正常唤醒与超时唤醒之间的不同。需要注意的是，wait(0)与wait(0,0)方法其实都具有特殊的意义，其相当于**不限时**的wait()方法，这可能与你的直觉相反。

由于线程竞争，调整策略以及定时器粒度等方面的原因，定时等待方法可能会消耗任意的时间。（注：关于定时器粒度并没有任何的保证，目前大多数JVM实现当参数设置小于1毫秒的时候，观察的结果基本上在1～20毫米之间）

Thead.sleep(long msecs)方法使用了**定时等待的wait方法**，但是使用的并不是当前对象的同步锁。它的效果如下描述：

```java
if (msecs != 0) {
    Object s = new Object(); // 使用了新对象来锁
    synchronized(s){
        s.wait(msecs);
    }
}
```

当然，系统不需要使用这种方法实现sleep方法。需要注意的，sleep(0)方法的含义是中断线程至少零(0)时间，随便怎么解释都行。（该方法有着特殊的作用，从原理上它可以保证系统重新进行一次CPU竞争）