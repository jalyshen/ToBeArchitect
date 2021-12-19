# Java内存模型-指南 (Recipes)

原文：http://ifeve.com/cookbook-recipes/



### 单处理器(Uniprocessors)

如果能保证正在生成的代码只会运行在单个处理器上，那就可以跳过本节的其余部分。因为单处理器保持着明显的顺序一致性，除非对象内存以某种方式与可异步访问的I/O内存共享，否则永远都不需要插入屏障指令。采用了特殊映射的java.nio buffers 可能会出现这种情况，但也许只会影响内部的JVM支持代码，而不会影响Java代码。而且，可以想象，如果上下文切换时不要求充分的同步，那就需要使用一些特殊的屏障了。



### 插入屏障(Inserting Barriers)

当程序执行时碰到了不同类型的存取，就需要屏障指令。几乎无法找到一个“最理想”位置，能将屏障执行总次数降到最小。编译器不知道指定的 ***load*** 或 ***store*** 指令是先于还是后于需要一个屏障操作的另一个 ***load*** 或 ***store*** 指令；如，当volatile store后面是一个return时。最简单保守的策略是为任一给定的 ***load，store，lock*** 或 ***unlock*** 生成代码时，都假设该类型的存取需要“最重量级”的屏障：

1. 在每条 ***volatile store*** 指令之前插入一个 ***StoreStore*** 屏障。(在ia64平台上，必须将该屏障及大多数屏障合并成相应的load或store指令。)
2. 如果一个类包含 **final 字段**，在该类每个构造器的全部store指令之后，return指令之前插入一个StoreStore屏障。
3. 在每条 ***volatile store*** 指令之后插入一条 ***StoreLoad*** 屏障。注意，虽然也可以在每条volatile load指令之前插入一个StoreLoad屏障，但对于使用volatile的典型程序来说则会更慢，因为读操作会大大超过写操作。或者，如果可以的话，将volatile store实现成一条原子指令（例如x86平台上的XCHG），就可以省略这个屏障操作。如果原子指令比StoreLoad屏障成本低，这种方式就更高效。
4. 在每条volatile load指令之后插入LoadLoad和LoadStore屏障。在持有数据依赖顺序的处理器上，如果下一条存取指令依赖于volatile load出来的值，就不需要插入屏障。特别是，在load一个volatile引用之后，如果后续指令是null检查或load此引用所指对象中的某个字段，此时就无需屏障。
5. 在每条MonitorEnter指令之前或在每条MonitorExit指令之后插入一个ExitEnter屏障。(根据上面的讨论，如果MonitorExit或MonitorEnter使用了相当于StoreLoad屏障的原子指令，ExitEnter可以是个空操作(no-op)。其余步骤中，其它涉及Enter和Exit的屏障也是如此。)
6. 在每条MonitorEnter指令之后插入EnterLoad和EnterStore屏障。
7. 在每条MonitorExit指令之前插入StoreExit和LoadExit屏障。
8. 如果在未内置支持间接load顺序的处理器上，可在final字段的每条load指令之前插入一个LoadLoad屏障。（[此邮件列表](http://www.cs.umd.edu/~pugh/java/memoryModel/archive/0180.html)和[linux数据依赖屏障的描述](http://lse.sourceforge.net/locking/wmbdd.html)中讨论了一些替代策略。）

这些屏障中的有一些通常会简化成空操作。实际上，大部分都会简化成空操作，只不过在不同的处理器和锁模式下使用了不同的方式。最简单的例子，在x86或sparc-TSO平台上使用CAS实现锁，仅相当于在volatile store后面放了一个StoreLoad屏障。

### 移除屏障(Removing Barriers)

上面的保守策略对有些程序来说也许还能接受。volatile的主要性能问题出在与store指令相关的StoreLoad屏障上。这些应当是相对罕见的 —— 将volatile主要用于避免并发程序里读操作中锁的使用，仅当读操作大大超过写操作才会有问题。但是至少能在以下几个方面改进这种策略：

- 移除冗余的屏障。可以根据前面章节的表格内容来消除屏障：

<table border="1" cellspacing="2" cellpadding="2">
<tbody>
<tr>
<td rowspan="1" colspan="3" align="center">Original</td>
<td>=&gt;</td>
<td rowspan="1" colspan="3" align="center">Transformed</td>
</tr>
<tr>
<td>1st</td>
<td>ops</td>
<td>2nd</td>
<td>=&gt;</td>
<td>1st</td>
<td>ops</td>
<td>2nd</td>
</tr>
<tr>
<td>LoadLoad</td>
<td>[no loads]</td>
<td>LoadLoad</td>
<td>=&gt;</td>
<td></td>
<td>[no loads]</td>
<td>LoadLoad</td>
</tr>
<tr>
<td>LoadLoad</td>
<td>[no loads]</td>
<td>StoreLoad</td>
<td>=&gt;</td>
<td></td>
<td>[no loads]</td>
<td>StoreLoad</td>
</tr>
<tr>
<td>StoreStore</td>
<td>[no stores]</td>
<td>StoreStore</td>
<td>=&gt;</td>
<td></td>
<td>[no stores]</td>
<td>StoreStore</td>
</tr>
<tr>
<td>StoreStore</td>
<td>[no stores]</td>
<td>StoreLoad</td>
<td>=&gt;</td>
<td></td>
<td>[no stores]</td>
<td>StoreLoad</td>
</tr>
<tr>
<td>StoreLoad</td>
<td>[no loads]</td>
<td>LoadLoad</td>
<td>=&gt;</td>
<td>StoreLoad</td>
<td>[no loads]</td>
<td></td>
</tr>
<tr>
<td>StoreLoad</td>
<td>[no stores]</td>
<td>StoreStore</td>
<td>=&gt;</td>
<td>StoreLoad</td>
<td>[no stores]</td>
<td></td>
</tr>
<tr>
<td>StoreLoad</td>
<td>[no volatile loads]</td>
<td>StoreLoad</td>
<td>=&gt;</td>
<td></td>
<td>[no volatile loads]</td>
<td>StoreLoad</td>
</tr>
</tbody>
</table>

类似的屏障消除也可用于锁的交互，但要依赖于锁的实现方式。 使用循环，调用以及分支来实现这一切就留给读者作为练习。:-)

- 重排代码（在允许的范围内）以更进一步移除LoadLoad和LoadStore屏障，这些屏障因处理器维持着数据依赖顺序而不再需要。
- 移动指令流中屏障的位置以提高调度(scheduling)效率，只要在该屏障被需要的时间内最终仍会在某处执行即可。
- 移除那些没有多线程依赖而不需要的屏障；例如，某个volatile变量被证实只会对单个线程可见。而且，如果能证明线程仅能对某些特定字段执行store指令或仅能执行load指令，则可以移除这里面使用的屏障。但是所有这些通常都需要作大量的分析。

### 杂记(Miscellany)

JSR-133也讨论了在更为特殊的情况下可能需要屏障的其它几个问题：

- Thread.start()需要屏障来确保该已启动的线程能看到在调用的时刻对调用者可见的所有store的内容。相反，Thread.join()需要屏障来确保调用者能看到正在终止的线程所store的内容。实现Thread.start()和Thread.join()时需要同步，这些屏障通常是通过这些同步来产生的。
- static final初始化需要StoreStore屏障，遵守Java类加载和初始化规则的那些机制需要这些屏障。
- 确保默认的0/null初始字段值时通常需要屏障、同步和/或垃圾收集器里的底层缓存控制。
- 在构造器之外或静态初始化器之外神秘设置System.in, System.out和System.err的JVM私有例程需要特别注意，因为它们是JMM final字段规则的遗留的例外情况。
- 类似地，JVM内部反序列化设置final字段的代码通常需要一个StoreStore屏障。
- 终结方法可能需要屏障（垃圾收集器里）来确保Object.finalize中的代码能看到某个对象不再被引用之前store到该对象所有字段的值。这通常是通过同步来确保的，这些同步用于在reference队列中添加和删除reference。
- 调用JNI例程以及从JNI例程中返回可能需要屏障，尽管这看起来是实现方面的一些问题。
- 大多数处理器都设计有其它专用于IO和OS操作的同步指令。它们不会直接影响JMM的这些问题，但有可能与IO,类加载以及动态代码的生成紧密相关。

