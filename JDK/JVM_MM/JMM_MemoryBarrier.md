# Java内存模型-内存屏障

原文：http://ifeve.com/jmm-cookbook-mb/



**编译器和处理器必须同时遵守重排规则**。由于单核处理器能确保与“顺序执行”相同的一致性，所以在单核处理器上并不需要专门做什么处理，就可以保证正确的执行顺序。但在多核处理器上通常需要使用内存屏障指令来确保这种一致性。即使编译器优化掉了一个字段访问（例如，因为一个读入的值为被使用），这种情况下还是需要产生内存屏障，就好像这个访问仍然需要保护。

内存屏障仅仅与内存模型中“获取”、“释放”这些高层次的概念有间接的关系。内存屏障并不是“同步屏障”，内存屏障也与在一些垃圾回收机制中“写屏障（write barriers）”的概念无关。内存屏障指令仅仅直接控制CPU与其缓存之间，CPU与其准备将数据写入主存或者写入等待读取、预测指令执行的缓冲中的写缓冲之间的相互操作。这些操作可能导致缓冲、主内存和其他处理器做进一步交互。但在Java内存模型规范中，没有强制处理器之间的交互方式，只要数据最终变为全局可用，就是说在所有处理器中可见，并当这些数据可见时可以获取它们。



## 内存屏障种类 

几乎所有的处理器至少支持一种粗粒度的屏障指令，通常被称为“栅栏（Fence）”，它保证在栅栏前初始化的load和store指令，能够严格有序的在栅栏后的load和store质量之前执行。无论在何种处理器上，这几乎都是最耗时的操作之一（与原子指令差不多，甚至更消耗资源），所以大部分处理器支持更细粒度的屏障指令。

内存平展的一个特征是将它们运用于内存之间的访问。尽管在一些处理器上有一些名为屏障的指令，但是正确的/最好的屏障使用取决于内存访问的类型。下面是一些屏障指令的通常分类，正好它们可以对应上常用处理器上的特定指令（有时这些指令不会导致操作）。

#### LoadLoad屏障

序列：Load1，Loadload，Load2

确保Load1所有要读入的数据能够在被Load2和后续的load指令访问前读入。通常能执行预加载指令或/和支持乱序处理的处理器中需要显式声明Loadload屏障，因为在这些处理器中正在等待的加载指令能够绕过正在等待存储的指令。而对于总是能保证处理顺序的处理器上，设置该屏障相当于无操作。

#### StoreStore屏障

序列：Store1，StoreStore，Store2

确保Store1的数据在Store2以及后续的Store指令操作相关数据之前对其它处理器可见（例如向主存刷新数据）。通常情况下，如果处理器不能保证从写缓冲或/和缓存向其它处理器和主存中按顺序刷新数据，那么它需要使用StoreStore屏障。

#### LoadStore屏障

序列：Load1，LoadStore， Store2

确保Load1的数据在Store2和后续Store指令被刷新之前读取。在等待Store指令可以超过loads指令的乱序处理器上需要使用LoadStore屏障。

#### StoreLoad屏障

序列：Store1，StoreLoad，Load2

确保Store1的数据在被Load2和后续的Load指令读取之前对其他处理器可见。**StoreLoad屏障可以防止一个后续的load指令 不正确的使用了Store1的数据，而不是另一个处理器在相同内存位置写入一个新数据**。正因为如此，所以在下面所讨论的处理器为了在屏障前读取同样内存位置存过的数据，必须使用一个StoreLoad屏障将存储指令和后续的加载指令分开。Storeload屏障在几乎所有的现代多处理器中都需要使用，但通常它的开销也是最昂贵的。它们昂贵的部分原因是它们必须关闭通常的略过缓存直接从写缓冲区读取数据的机制。这可能通过让一个缓冲区进行充分刷新（flush）,以及其他延迟的方式来实现。

在下面讨论的所有处理器中，执行StoreLoad的指令也会同时获得其他三种屏障的效果。所以StoreLoad可以作为最通用的（但通常也是最耗性能）的一种Fence。(这是经验得出的结论，并不是必然)。反之不成立，为了达到StoreLoad的效果而组合使用其他屏障并不常见。
下表显示这些屏障如何符合JSR-133排序规则：

<table>
	<tr>
		<th>需要的屏障 </th>
		<th colspan="4">第二步</th>
	</tr>
	<tr>
		<td>第一步</td>
		<td>Normal Load</td>
		<td>Normal Store</td>
		<td>Volatile <br> Load <br> MonitorEnter</td>
		<td>Volatile <br> Store <br> MonitorExit</td>
	</tr>
	<tr>
		<td>Normal Load</td>
		<td></td>
		<td></td>
		<td></td>
		<td>LoadStore</td>
	</tr>
	<tr>
		<td>Normal Store</td>
		<td></td>
		<td></td>
		<td></td>
		<td>StoreStore</td>
	</tr>
	<tr>
		<td>Volatile <br> Load <br> MonitorEnter</td>
		<td>LoadLoad</td>
		<td>LoadStore</td>
		<td>LoadLoad</td>
		<td>LoadStore</td>
	</tr>
	<tr>
		<td>Volatile <br> Store <br> MonitorExit</td>
		<td></td>
		<td></td>
		<td>StoreLoad</td>
		<td>StoreStore</td>
	</tr>
</table>

另外，特殊的final字段规则在下列代码中需要一个StoreStore屏障：

```java
x.finalField = v; StoreStore; sharedRef = x;
```

如下例子解释如何放置屏障：

```java
class X {
	int a, b;
	volatile int v, u;

	void f() {
		int i, j;

		i = a;// load a
		j = b;// load b
		i = v;// load v
		// LoadLoad
		j = u;// load u
		// LoadStore
		a = i;// store a
		b = j;// store b
		// StoreStore
		v = i;// store v
		// StoreStore
		u = j;// store u
		// StoreLoad
		i = u;// load u
		// LoadLoad
		// LoadStore
		j = b;// load b
		a = i;// store a
	}
}
```

## 数据依赖和屏障

一些处理器为了保证依赖指令的交互次序需要使用LoadlLoad和LoadStore屏障。在一些（大部分）处理器中，一个load指令或者一个依赖于之前加载值的store指令被处理器排序，并不需要一个显式的屏障。这通常发生于两种情况：

1. 间接取值（indirection）：

```java
Load x；Load x.field
```

2. 条件控制（Control）

```java
Load x; if (predicate(x)) Load or Store y;
```

但特别的是不遵循间接排序的处理器，需要为final字段设置屏障，使它能通过共享引用访问最初的引用。

```java
x = sharedRef; … ; LoadLoad; i = x.finalField;
```

相反的，如下讨论，确定遵循数据依赖的处理器，提供了几个优化掉LoadLoad和LoadStore屏障指令的机会。（尽管如此，在任何处理器上，对于StoreLoad屏障不会自动清除依赖关系）。

### 与原子指令交互

屏障在不同处理器上还需要与MonitorEnter和MonitorExit实现交互。锁或者解锁，通常必须使用原子条件更新操作 CompareAndSwap(CAS) 指令或者 LoadLinked/StoreConditional (LL/SC) ，就如执行一个 volatile store 之后紧跟 volatile load 的语义一样。 CAS 或者 LL/SC 能够满足最小功能，一些处理器提供其他的原子操作（如，一个无条件交换），这在某些时候它可以替代或者与原子条件更新操作结合使用。

在所有处理器中，原子操作可以避免在正被读取/更新的内存位置进行写后读（read-after-write）（否则标准的循环直到成功的结构体（loop-until- success）没办法正常工作）。但处理器在是否为原子操作提供比隐式的 StoreLoad 更一般的屏障特性上表现不同。一些处理器上这些指令可以为 MornitorEnter/Exit 原生的生成屏障，其他的处理器中一部分或者全部屏障必须显式地指定。

为了分清这些影响，必须把 Volatiles 和 Monitors 分开：

<table>
  <tr>
    <th>需要的屏障</th>
  	<th colspan=6>第二步</th>
  </tr>
  <tr>
    <td>第一步</td>
    <td>Normal Load</td>
    <td>Normal Store</td>
    <td>Volatile Load</td>
    <td>Volatile Store</td>
    <td>MonitorEnter</td>
    <td>MonitorExit</td>
  </tr>
  <tr>
    <td>Normal Load</td>
    <td></td>
    <td></td>
    <td></td>
    <td>LoadStore</td>
    <td></td>
    <td>LoadStore</td>
  </tr>
  <tr>
    <td>Normal Store</td>
    <td></td>
    <td></td>
    <td></td>
    <td>StoreStore</td>
    <td></td>
    <td>StoreExit</td>
  </tr>
  <tr>
    <td>Volatile Load</td>
    <td>LoadLoad</td>
    <td>LoadStore</td>
    <td>LoadLoad</td>
    <td>LoadStore</td>
    <td>LoadEnter</td>
    <td>LoadExit</td>
  </tr>
  <tr>
    <td>Volatile Store</td>
    <td></td>
    <td></td>
    <td>StoreLoad</td>
    <td>StoreStore</td>
    <td>StoreEnter</td>
    <td>StoreExit</td>
  </tr>
  <tr>
    <td>MonitorEnter</td>
    <td>EnterLoad</td>
    <td>EnterStore</td>
    <td>EnterLoad</td>
    <td>EnterStore</td>
    <td>EnterEnter</td>
    <td>EnterExit</td>
  </tr>
  <tr>
    <td>MonitorExit</td>
    <td></td>
    <td></td>
    <td>ExitLoad</td>
    <td>ExitStore</td>
    <td>ExitEnter</td>
    <td>ExitExit</td>
  </tr>
</table>

另外，特殊的 final 字段规则需要一个 StoreLoad 屏障。

```
x.finalField = v; StoreStore; sharedRef = x;
```

在这张表里， “Enter”与“Load”相同，“Exit”与“Store”相同，除非被子原子指令的使用和特性覆盖。特别是：

* EnterLoad 在进入任何需要执行 Load 指令的同步块/方法时都需要。这与 LoadLoad 相同，除非在 MonitorEnter 时候使用了原子指令并且它本身提供一个至少有 LoadLoad 属性的屏障，如果是这种情况，相当于没有操作
* StoreExit 在退出任何执行 Store 指定的同步方法块时候都需要。这与 StoreStore 一致，除非 MonitorExit 使用原子操作，并且提供了一个至少有 StoreStore 属性的屏障，如果是这种情况，相当于没有操作
* ExitEnter 和 StoreLoad 一样，除非 MonitorExit 使用了原子指令，并且/或者 MonitorEnter 至少提供一种屏障，该屏障具有 StoreLoad 的属性，如果是这种情况，相当于没有操作

​        在编译时不起作用或者导致处理器上不产生操作的指令比较特殊。例如，当没有交替的 Load 和 Store 指令时， EnterEnter 用于分离嵌套的 MonitorEnter。 下面例子说明如何使用这些指令类型：

```java
class X {
	int a;
	volatile int v;

	void f() {
		int i;
		synchronized (this) { // enter EnterLoad EnterStore
			i = a;// load a
			a = i;// store a
		}// LoadExit StoreExit exit ExitEnter

		synchronized (this) {// enter ExitEnter
			synchronized (this) {// enter
			}// EnterExit exit
		}// ExitExit exit ExitEnter ExitLoad

		i = v;// load v

		synchronized (this) {// LoadEnter enter
		} // exit ExitEnter ExitStore

		v = i; // store v
		synchronized (this) { // StoreEnter enter
		} // EnterExit exit
	}

}
```

Java层次的对原子条件更新的操作将在JDK1.5中发布（JSR-166），因此编译器需要发布相应的代码，综合使用上表中使用MonitorEnter和MonitorExit的方式，——从语义上说，有时在实践中，这些Java中的原子更新操作，就如同他们都被锁所包围一样。