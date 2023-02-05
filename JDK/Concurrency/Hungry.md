# 多线程饥饿现象，饥饿与死锁区别

原文：https://blog.csdn.net/qq_42411214/article/details/107110547



## 饥饿和饿死的概念、饥饿与死锁的区别



### 饥饿和饿死的概念

1. 饥饿是指系统不能保证某个进程的等待时间上界，从而使该进程长时间等待，当等待时间给进程推进和响应带来明显影响时，称发生了“**进程饥饿**”。
2. 当饥饿到一定程度的进程赋予所赋予的任务即使完成也不再具有实际意义时称该进程被“**饿死**”。
3. "**[死锁](./DeadLock.md)**"是指在多道程序系统中，一组进程中的每一个进程都无限期待等待被该组进程中的另一个进程所占有且永远不会释放的资源。

### 饥饿与死锁的区别

* 相同点

  二者都是因为竞争资源引起的

* 不同的

  死锁是同步的，饥饿是异步的。即：

  * 死锁可以认为是**两个及以上**线程或进程**同时**在请求对方占有的资源。
  * 饥饿可以认为是**一个或以上**线程或进程在无限等待另外两个或多个线程或进程占有的但是**不会往外释放**的资源。

  就是，死锁里资源的占有方和资源的拥有方相互请求对方的资源，但是饥饿是一方请求不知道哪一方的资源，就是饿了需要吃饭，但是谁给都行。

1. 从进程状态考虑，死锁进程都处于等待状态，忙等待（处于运行或者就绪状态）的进程并非处于等待状态，但却可以被饿死；
2. 死锁进程等待永远不会被释放的资源，饿死进程等待会被释放但却不会分配给自己的资源，表现为等待时限没有上界（排队等待或忙式等待）；
3. 死锁一定发生了循环等待，而饿死则不然。这也表明通过资源分配图可以检测死锁存在与否，但却不能检测是否有进程饿死；
4. 死锁一定涉及多个进程，而饿死或被饿死的进程可能只有一个；
5. 在饥饿的情形下，系统中有至少一个进程能正常运行，只是饥饿进程得不到执行机会。而死锁则可能会最终使整个系统陷入死锁并崩溃

### 例子

死锁例一：

> 如果线程A锁住了记录R1并等待记录R2，而线程B锁住了记录R2并等待记录R1，这样两个线程A和线程B就发生了死锁现象

死锁例二：

> 两个山羊过一个独木桥，两只羊同时走到桥中间，一个山羊等待另一个山羊过去了然后再过桥，另一个山羊等这一个山羊过去，结果两只山羊都堵在中间动弹不得。

饥饿例子：

> 资源在其中两个或以上线程或进程相互使用，第三方线程或进程始终得不到。想象一下三个人传球，其中两个传来传去，第三个人始终得不到。



## 案例导引

让有限的工作线程来轮流异步处理无限多的工作任务，也可以将其归类为分工模式，典型实现就是线程池，也体现经典设计模式中的享元模式。

例如：饭店给每个客人都配备一个新的服务员，那么开销资源消耗太大

*注意：不同的类型应该使用不同的线程池，这样能避免饥饿，提升效率。*

例如：饭店切菜就是切菜的员工，服务员就是服务员



### 产生饥饿

下面代码会产生饥饿问题，线程池中两个线程都去招待客人，没有

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService pool = Executors.newFixedThreadPool(2);
    pool.execute(() -> {
       System.out.println(Thread.currentThread().getName()+"招待顾客");
        Future<String> submit = pool.submit(() -> {
            System.out.println(Thread.currentThread().getName()+"做宫保鸡丁");
            return "去做宫保鸡丁";
        });
    });
    pool.execute(()-> {
        System.out.println(Thread.currentThread().getName() + "招待顾客");
        Future<String> submit = pool.submit(() -> {
            System.out.println(Thread.currentThread().getName() + "做水煮鱼");
            return "去做水煮鱼";
        })
    })
}
```

执行结果如下图：

![1](.\images\Hungry\1.png)



### 饥饿解决

将线程池中的线程数改为3，这样可以**暂时**解决饥饿问题。通过扩大线程池中的线程数方法来解决：

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService pool = Executors.newFixedThreadPool(3);
    pool.execute(() -> {
       System.out.println(Thread.currentThread().getName()+"招待顾客");
        Future<String> submit = pool.submit(() -> {
            System.out.println(Thread.currentThread().getName()+"做宫保鸡丁");
            return "去做宫保鸡丁";
        });
    });
    pool.execute(()-> {
        System.out.println(Thread.currentThread().getName() + "招待顾客");
        Future<String> submit = pool.submit(() -> {
            System.out.println(Thread.currentThread().getName() + "做水煮鱼");
            return "去做水煮鱼";
        })
    })
}
```

执行结果：

![2](.\images\Hungry\2.png)

这里说说饥饿问题，要把饥饿和死锁问题区分开。

饥饿，它是两个或者多个线程都需要继续向下执行但是需要别的资源来保证自己向下执行；死锁需要最少两个线程他们之间的运行需要对方的数据资源来帮助自己向下执行，饥饿是不能通过JConsole 检测出来是因为他们之间不竞争资源，但是死锁可以通过工具检测出来，因为线程间存在资源的竞争。



### 解决饥饿问题

多创建一个线程池解决问题，做菜线程就是做菜线程，服务员线程池就是服务员线程池，各个线程池各司其职：

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService waiterpool = Executors.newFixedThreadPool(1);
    ExecutorService cookpool = Executors.newFixedThreadPool(1);
    waiterpool.execute(() -> {
       System.out.println(Thread.currentThread().getName()+"招待顾客");
        Future<String> submit = cookpool.submit(() -> {
            System.out.println(Thread.currentThread().getName()+"做宫保鸡丁");
            return "去做宫保鸡丁";
        });
    });
    waiterpool.execute(()-> {
        System.out.println(Thread.currentThread().getName() + "招待顾客");
        Future<String> submit = cookpool.submit(() -> {
            System.out.println(Thread.currentThread().getName() + "做水煮鱼");
            return "去做水煮鱼";
        })
    })
}
```

执行结果：

![3](.\images\Hungry\3.png)

