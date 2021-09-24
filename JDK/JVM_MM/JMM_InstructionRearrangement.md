# JMM 指令重排

原文：http://ifeve.com/jmm-cookbook-reorderings/



​        对于编译器的编写者来说，Java内存模型（JMM）主要是禁止指令重排的规则所组成的，其中包括了字段（包括数组中的元素）的存取指令和监视器（锁）的控制指令。

## Volatile与监视器

​        JMM中关于 **volatile** 和 **监视器** 主要的规则可以被看作一个矩阵。这个矩阵的单元格表示在一些特定的后续关联指令的情况下，指令不能被重排。下面的表格并不是JMM规范包含的，而是一个用来观察JMM模型对编译器和运行系统造成的主要影响的工具。

| 是否重排                      | 第二个操作                 |                               |                               |
| ----------------------------- | -------------------------- | ----------------------------- | ----------------------------- |
| 第一个操作                    | Normal Load / Normal Store | Volatile Load  / MonitorEnter | Volatile Store  / MonitorExit |
| Normal Load / Normal Store    |                            |                               | No                            |
| Volatile Load /  MonitorEnter | No                         | No                            | No                            |
| Volatile Store / MonitorExit  |                            | No                            | No                            |

关于上面这个表格一些术语的说明：

* Normal Load 指令包括：对非volatile字段的读取，getfield、getstatic和array load
* Normal Store指令包括：对非volatile字段的存储，putfield、putstatic和array store
* Volatile Load指令包括：对多线程环境的volatile变量的读取，getfield，getstatic
* Volatile Store指令包括：对多线程环境的volatile变量的存储，putfield，putstatic
* Monitor Enters指令（包括进入同步块synchronized方法）是用于多线程环境的所对象
* MonitorExits指令（包括离开同步块synchronized方法）是用于多线程环境的锁对象



​        在JMM中，Normal Load指令与Normal Store指令的规则是一致的，类似的还会有Volatile load指令与MonitorEnter指令，以及Volatile Store指令与MonitorExit指令，因此这几对指令的单元格在上面表格里都合并在一起（但是在后面部分的表格中，会在有需要的时候展开）。在这个小节中，我们仅仅考虑那些被当作原子单元的可读可写的变量，也就是说那些没有位域（bit fields），非对齐访问（unaligned accesses）或者超过平台最大字长（word size）的访问。

​        **任意数量的指令操作都可以被表示成这个表格中的第一个操作或者第二个操作**。例如，在单元格（Normal Store，Volatile Store）中，有一个No, 就表示任何非volatile字段的store指令操作不能与后面任意一个volatile store指令重排，如果出现任何这样的重排，会使多线程程序的运行发生变化。

​        JSR-133规范规定上述关于volatile和监视器的规则仅仅适用于可能会被多线程访问的变量或对象。因此，一个编译器可以最终证明（往往是需要很大的努力）一个锁只被单线程访问，那么这个锁就可以被去除。与之类似，一个volatile变量只被单线程访问也可以被当作是普通的变量。还有进一步更细粒度的分析与优化，例如：那些被证明在一段时间内对多线程不可访问的字段。

​        在上表中，空白的单元格代表在不违反Java的基本语义下的重排是允许的（详细参考JLS中的说明）。例如，即使上表中没有说明，但是也不能对同一个内存地址上的load指令和之后紧跟着的store指令进行重排。但是你可以对两个不同的内存地址上的load和store指令进行重排，而且往往在很多编译器转换和优化中会这么做。这其中就包括了一些往往不认为是指令重排的例子。例如，重用一个基于已经加载的字段的计算后的值而不是像一次指令重排那样去重新加载并重新计算。然而，JMM规范允许编译器经过一些转换后消除这些可以避免的依赖，使其可以支持指令重排。

​        在任何的情况下，即使是程序员错误的使用了同步读取，指令重排的结果也必须达到最基本的Java安全要求。所有的显示字段都必须不是被设定成0或null这样的预构造值，就是被其他线程设值。这通常必须把所有存储在堆内存里的对象在其被构造函数使用前进行归零操作，并且从来不对归零store指令进行重排。一种比较好的方式是在垃圾回收中对回收的内存进行归零操作。可以参考JSR-133规范中其他情况下的一些关于安全保证的规则。

​        这里描述的规则和属性都是适用于读取Java环境中的字段。在实际的应用中，这些都可能会另外与读取内部的一些记账字段和数据交互，例如对象头，GC表和动态生成的代码。

## Final字段

​        Final字段的load和store指令相对于有锁的或者volatile字段来说，就跟Normal load和Normal store的存取是一样的，但是需要加入两条附加的指令重排规则：

1. 如果在构造函数中有一条final字段的store指令，同时这个字段是一个引用，那么它将不能与构造函数外后续可以让持有这个final字段的对象被其他线程访问的指令重排。例如：不能重排下面的语句：

   ```java
   x.finalField = v;
   ... ;
   sharedRef = x;
   ```

   这条规则会在下列情况下生效：例如，当你内联一个构造函数时，正如“..."的部分表示这个构造函数的逻辑边界那样。你不能把这个构造函数中的对于这个final字段的store指令移动到构造函数外的一条store指令后面，因为这可能会使这个对像对其他线程可见。（正如你将在下面看到的，这样的操作还需要声明一个内存屏障）。类似的，也不能把下面的前两条指令与第三条指令进行重排：

   ```java
   x.afield = 1; x.finalField = v; ... ; sharedRef = x;
   ```

2. 一个final字段的初始化load指令不能与包含该字段的对象的初始化load指令进行重排。在下面这种情况下，这条规则就会生效：

   ```java
   x = shareRef; … ; i = x.finalField;
   ```

   由于这两条指令是依赖的，编译器将不会对这样的指令进行重排。但是，这条规则会对某些处理器有影响。

​        上述规则，要求对于带有final字段的对象的load本身时synchronized，volatile，final或者来自类似的load指令，从而确保Java程序员对final字段的正确使用，并最终使构造函数中初始化的store指令和构造函数外的store指令排序。

