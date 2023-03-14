# 任务取消

原文：http://ifeve.com/cancellation/



当某个线程中的活动执行失败或想改变运行意图，也许就有必要或想要在其它线程中取消这个线程的活动，而不管这个线程正在做什么。取消会给运行中的线程带来一些无法预料的失败情况。取消操作异步特性相关的设计技巧，让人想起了因系统崩溃和连接断开任何时候都有可能失败的分布式系统的那些技巧。并发程序还要确保多线程共享的对象的状态一致性。

在大多数多线程程序中，取消任务（Cancellation）是普遍存在的，常见于：

* 几乎所有与GUI中取消按钮相关的活动
* 多媒体演示（如动画循环）中的正常终止活动
* 线程中生成的结果不再需要。例如使用多个线程搜索数据库，只要某个线程返回了结果，其他的线程都可以取消
* 由于一组活动中的一个或多个遇到意外错误或异常导致整租活动无法继续



## 中断 (Interruption)

**实现取消任务的最佳技术是使用线程的中断状态**，这个状态由Thread.interrupt设置，可被Thread.isInterrupted检测到，通过Thread.interrupted清除，有时候抛出InterruptedException异常来响应。

**线程中断起着请求取消活动的作用**。但无法阻止有人将其用作它途，而用来作取消操作是预期的用途。基于中断的任务取消依赖于取消者和被取消者间的一个协议以确保跨多线程使用的对象在被取消线程终止的时候不被损坏。大部分（理想情况下是所有的）java.*包中的类都遵守这个协议。

几乎在所有的情况下，取消一个与线程有关系的活动都应当终止对应的线程。但中断机制不会强制线程立马终止。这就给任何被中断的线程一个在终止前做些清理操作的机会，但也给代码增加了及时检查中断状态以及采取合适操作的职责。

延迟甚至忽略任务取消的请求给写出良好响应性且非常健壮的代码提供了途径。因为不会直接将线程中断掉，所以很难或不可能撤销的动作的前面可以作为一个安全点，然后在此安全点检查中断状态。响应中断大部分可选的方式在$3.1.1中有讨论。

继续执行（忽略或者清除了中断）可能适用于那些不打算终止的线程。例如，那些对于程序基本功能不可或缺的数据库管理服务。一旦遇到中断，可中止这些特殊任务，然后允许线程继续执行其他任务。然而，即使在这里，将中断的线程替换成一个处于初始状态的新启动的线程会更易于管理。

突然终止（比如抛出错误）一般适用于提供独立服务、除了run方法中finally子句外无需其它清理操作的线程。但是，当线程执行的服务被其他线程依赖时（见$4.3），就应当以某种形式通知这些依赖的线程或设置状态指示。（异常本身不会自动在线程间传播）

线程中使用的对象被其它线程依赖时必须使用回滚或前滚技术。

在某种程度上，可通过决定多久用Thread.currentThread().isInterrupted()来检查中断状态以控制代码对中断的响应灵敏性。中断状态检查不需要太频繁以免影响程序效率。例如，如果需要取消的活动包含大约10000条指令，每10000条指令做一次取消检查，那么从取消请求到关闭平均会消耗15000条指令。只要活动继续运行没有什么实际的危害，这个数量级可以满足大部分应用的需求。通常，这个理由可以让你将中断检测代码仅放到既方便检测又是重要的程序点。在性能关键型应用中，也许值得构建一个分析模型或收集经验值来准确地决定响应性与吞吐量间的最佳权衡。

Object.wait()，Thread.join()，Thread.sleep()以及他们衍生出的方法都会自动检测中断。这些方法一旦中断就会抛出InterruptedException来中止，然后让线程苏醒并执行与活动取消相关的代码。

按照惯例，应当在抛出InterruptedException时清除中断状态。有时候有必要这样做来支持一些清理工作，但这也可能是错误与混乱之源。当处理完InterruptedException后想要传播中断状态，必须要么重新抛出捕获的InterruptedException，要么通过Thread.currentThread().interrupt()重新设置中断状态。如果你的代码调用了其它未正确维持中断状态的代码（例如，忽略InterruptedException又不重设状态），可以能通过维持一个字段来规避问题，这个字段用于保存活动取消的标识，在调用interrupte的时候设置该字段，从有问题的调用中返回时检查该字段。

有两种情况线程会保持休眠而无法检测中断状态或接收InterruptedException：在同步块中和在I/O中阻塞时。线程在等待同步方法或者同步块的锁时不会对中断有响应。但是，如$2.5中讨论的，当需要大幅降低在活动取消间被卡在锁等待中的几率，可以使用lock工具类。使用lock类的代码阻塞仅是为了访问锁对象本身，而不是这些锁所保护的代码。这些阻塞的耗时天生就很短（尽管时间不能严格保证）。



## I/O和资源撤销 (I/O and Resource Revocation)

一些I/O支持类（尤其是java.net.Socket及相关类）提供了在读操作阻塞的时候能够超时的可选途径，在这种情况下就可以在超时后检测中断。java.io中的其它类采用了另一种方式 -- 一种特殊形式的资源撤销。如果某个线程在一个I/O对象s（如InputStream）上执行s.close()，那么任何其它尝试使用s的线程将收到一个IOException。I/O关闭会影响所有使用关闭了的I/O对象的线程，会导致I/O对象不可用。如有必要，可以创建一个新的I/O对象来替代关闭的I/O对象。

这与其它资源撤销的用途密切相关（如为了安全目的）。该策略也会保护应用免让共享的I/O对象因其它使用了此I/O对象的线程被取消而自动变得不可用。大部分java.io中的类不会也不能在出现I/O异常时清除失败状态。例如，如果在StreamTokenizer或ObjectInputStream操作中间出现了一个底层I/O异常，没有一个使用的回复动作能继续保持预期的保障。所以，作为一种策略，JVM不会自动中断I/O操作。

**这给代码强加了额外的职责来处理取消事件。**若一个线程正在执行I/O操作，如果在此I/O操作期间试图取消该I/O操作，必须意识到I/O对象正在使用且关闭该I/O对象是你想要的行为。如果能接受这种情况，就可以通过关闭I/O对象和中断线程来完成活动取消。例如：

```java
class CancellableReader {                        // Incomplete
	private Thread readerThread; // only one at a time supported
	private FileInputStream dataFile;

	public synchronized void startReaderThread() 
		throws IllegalStateException, FileNotFoundException {
		if (readerThread != null) throw new IllegalStateException();
			dataFile = new FileInputStream("data");
			readerThread = new Thread(new Runnable() {
				public void run() { doRead(); }
			});
		readerThread.start();
	}

	protected synchronized void closeFile() { // utility method
		if (dataFile != null) {
			try { dataFile.close(); } 
			catch (IOException ignore) {}
			dataFile = null;
		}
	}

	protected void doRead() {
		try {
			while (!Thread.interrupted()) {
				try {
					int c = dataFile.read();
					if (c == -1) break;
					else process(c);
				} catch (IOException ex) {
					break; // perhaps first do other cleanup
				}
			}
		} finally {
			closeFile();
			synchronized(this) { readerThread = null; }
		}
	}

	public synchronized void cancelReaderThread() {
		if (readerThread != null) readerThread.interrupt();
			closeFile();
	}
}
```

很多其它取消I/O的场景源于需要中断那些等待输入而输入却不会或不能及时到来的线程。大部分基于套接字的流，可以通过设置套接字的超时参数来处理。其它的，可以依赖InputStream.available，然后手写自己的带时间限制的轮询循环来避免超时之后还阻塞在I/O中。这种设计可以使用一种类似$3.1.1.5中描述的有时间限制的退避重试协议。例如：

```java
class ReaderWithTimeout {                // Generic code sketch
	// ...
	void attemptRead(InputStream stream, long timeout) throws... {
		long startTime = System.currentTimeMillis();
		try {
			for (;;) {
				if (stream.available() > 0) {
					int c = stream.read();
					if (c != -1) process(c);
					else break; // eof
				} else {
					try {
						Thread.sleep(100); // arbitrary fixed back-off time
					} catch (InterruptedException ie) {
						/* ... quietly wrap up and return ... */ 
					}
					long now = System.currentTimeMillis();
					if (now - startTime >= timeout) {
						/* ... fail ...*/
					}
				}
			}
		} catch (IOException ex) { /* ... fail ... */ }
	}
}
```

> *脚注：有些JDK发布版本也支持InterruptedIOException，但只是部分实现了且仅限于某些平台。在本文撰写之时，未来版本打算停止对其支持，部分原因是由于I/O对象不可用会引起不良后果。但既然InterruptedIOException定义为IOException的一个子类，这种设计的工作方式与包含InterruptedIOException支持的版本上描述的相似，尽管存在额外的不确定性：中断可能抛出InterruptedIOException或InterruptedException。捕获InterruptedIOException然后将其作为一个InterruptedException重新抛出能部分解决该问题。*

## 异步终止(Asynchronous Termination)

stop方法起初包含在Thread类中，但是已经不推荐使用了。**Thread.stop()会导致不管线程正在做什么就突然抛出一个ThreadDeath异常**（*不推荐使用的原因*）。（与interrupt()类似，stop()不会中止锁等待或I/O等待。但与Interrupt不同的是，它不严格保证中止wait()，sleep()或join()）。

这会是个非常危险的操作。因为Thread.stop()产生异步信号，某些操作由于程序安全和对象一致性必须回滚或前滚，而活动正在执行这些操作或代码段时可能被终止掉。例如：

```java
class C {                                         // Fragments
	private int v;  // invariant: v >= 0

	synchronized void f() {
		v = -1  ;   // temporarily set to illegal value as flag
		compute();  // possible stop point (*)
		v = 1;      // set to legal value
	}

	synchronized void g() { 
		while (v != 0) { 
			--v; 
			something(); 
		} 
	}
}
```

如果Thread.stop()碰巧导致(*)行终止，对象就被破坏了：线程一旦终止，对象将保持在不一致状态，因为变量v被设了一个非法的值。其它线程在该对象上的任何调用会执行不想要的或危险的操作。例如，这里g()方法中的循环将自旋 2 * Integer.MAX_VALUE次。stop()让回滚或前滚恢复技术的使用变得极其困难。乍一看，这个问题看起来不太严重-- 毕竟，调用compute()抛出的任何未捕获异常都会破坏状态。但是，Thread.stop()的后果更隐蔽，因为在可能忽略了ThreadDeath异常（由Thread.stop()抛出）而仍传播取消请求的方法中你什么也做不了。而且，除非在每行代码后面都放一个catch(ThreadDeath)，否则就没有办法准确恢复当前对象的状态，所以可能碰到未检测到的破坏。相比之下，通常可以将代码写得健壮些，不用大费周章就能消除或处理其它类型的运行时异常。

换言之，**禁用Thread.stop()不是为了修复它有缺陷的逻辑，而是纠正对其功能的错误认识**。不可能允许所有方法的每条字节码都能出现取消操作导致的异常（底层操作系统代码开发者非常熟悉这个事实。即使程序非常短，很小的异步取消安全的例程也会是个艰巨的任务。）

注意，任意正在执行的方法可以捕获并忽略由stop()导致的ThreadDeath异常。这样的话，stop()就和interrupte()一样不能保证线程会被终止，这更危险。任何stop()的使用都暗含着开发者评估过试图突然终止某个活动带来的潜在危害比不这样做的潜在危害更大。

## 资源控制(Resource Control)

活动取消可能出现在可装载和执行外部代码的任一系统的设计中。试图取消未遵守标准约定的代码面临着难题。外部代码也许完全忽略了中断，甚至是捕获ThreadDeath异常后将其丢弃，在这种情况下调用 Thread.interrupt()和Thread.stop()将不会有什么效果。

你无法精准控制外来代码的行为及其耗时。但能够且应当使用标准的安全措施来限制不良后果。一种方式是创建和使用一个SecurityManager，当某个线程运行的时间太长，就拒绝所有对受检资源的请求。这种形式的资源拒绝同$3.1.1.2中讨论的资源撤销策略一起，能够阻止外部代码执行任一与其它应当继续执行的线程竞争资源。副作用就是，这些措施经常最终会导致线程因异常而挂掉。

此外，可以调用某个线程的setPriority(Thread.MIN_PRIORITY)将CPU资源的竞争降到最小。可以用一个SecurityManager来阻止该线程将优先级提高。

## 多步取消(Multiphase Cancellation)

有时候，即使取消的是普通的代码，损害也比通常的更大。为应付这种可能性，可以建立一个通用的多步取消功能，尽可能尝试以破坏性最小的方式来取消任务，如果稍时候还没有终止，再使用一种破坏性较大的方式。

在大多数操作系统进程级，多步取消是一种常见的模式。例如，它用在Unix关闭期间，先尝试使用kill -1 终止任务，若有必要随后再用kill -9，大多数win系统中的任务管理器也使用了类似的策略。

这里有个简单版本的示例：（Thread.join()使用方面的更多细节参见$4.3.2）

```java
class Terminator {
	// Try to kill; return true if known to be dead

	static boolean terminate(Thread t, long maxWaitToDie) { 
		
		if (!t.isAlive()) return true;  // already dead
		// phase 1 -- graceful cancellation
		
		t.interrupt();       
		try { t.join(maxWaitToDie); } 
		catch(InterruptedException e){} //  ignore 

		if (!t.isAlive()) return true;  // success

		// phase 2 -- trap all security checks

		theSecurityMgr.denyAllChecksFor(t); // a made-up method
		try { t.join(maxWaitToDie); } 
		catch(InterruptedException ex) {} 

		if (!t.isAlive()) return true; 

		// phase 3 -- minimize damage

		t.setPriority(Thread.MIN_PRIORITY);
		return false;
	}
}
```

注意，这里的terminate()方法本身忽略了中断。这表明取消操作所做的这种策略选择一旦开始就必须继续。取消正在执行的取消操作，会给处理已经开始的终止相关的清理带来另一些问题。

因不同JVM实现中Thread.isAlive()的行为不尽相同，当join因线程结束返回后，在线程完全死掉之前isAlive还有可能返回true。