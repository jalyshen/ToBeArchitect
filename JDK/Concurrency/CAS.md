## Java并发编程 - CAS

原文：https://www.cnblogs.com/iou123lg/p/9314826.html



### 1. 并发编程三要素：原子性、可见性、有序性

在讨论CAS之前，先说说并发编程三要素。理解这三要素可以帮助理解CAS的作用。

* 原子性：

    指的是一个操作不能再继续拆分，要么一次操作完成，要么就不执行。在Java中，为了保证原子性，提供了两个高级的字节码指令（monitorenter和monitorexit），这个就是关键字 synchronized

* 可见性：

    指的是一个变量在一个线程更改后，其他的线程能立刻看到最新的值

* 有序性：

    指的是程序的执行按照代码的先后顺序执行，对于可见性和有序性，Java提供了关键字 volatile。volatile禁止指令重排，保证了有序性，同时volatile可以保证变量的读写及时从缓存中刷新到主存，也就保证了可见性。除此之外，synchronized是可见性和有序性另外一种实现，同步方法和同步代码块一个变量在同一个时间只能有一个线程访问，这就是一种先后顺序，而对于可见性保证，只能有一个线程操作变量，那么其他线程只能在前一个线程操作完成后才可以看到变量最新的值

做个总结，synchronized 一次性满足了3个特性，所以可以大胆假设 CAS + volatile 组合可以满足3个特性。

### 2. CAS介绍

CAS全称 Compare-and-Swap，是计算机科学中一种实现多线程原子操作的指令，它比较内存中当前存在的值和外部给定的期望值，只有两者相等时，才将这个内存值修改为新的给定值。CAS操作包含三个操作数：需要读写的内存位置（V）、拟比较的预期原值（A）和拟写入的新值（B）。如果V的值和A的值匹配，则将V的值更新为B，否则不做任何操作。

多线程尝试使用CAS更新同一变量时，只有一个线程可以操作成功，其他的线程都会失败，失败的线程不会被挂起，只是在此次竞争中被告知失败，可以继续尝试CAS操作。

### 3. CAS背后实现

JUC下的 atomic 类都是通过CAS来实现的。下面就以 AtomicInteger 类为例来阐述 CAS 的实现。直接看方法 compareAndSet，调用了 unsafe 类的 compareAndSwapInt方法：

```java
public final boolean compareAndSet(int expect, int uddate) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

其中四个参数分别表示对象、对象的地址（定位到V）、预期值（A）、修改值（B）。

再看 unsafe 类的方法，这是一个 native 方法，所有只能继续看看OpenJDK的代码：

```java
pulbic final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

在 unsafe.cpp 找到方法 CompareAndSwapInt， 可以依次看到变量 obj， offset，e和x，其中addr就是当前内存位置指针，最终再调用 Atomic 类的 cmpxchg 方法：

```c++
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```

找到 类atomic.hpp，从变量命名上基本可以知道其含义：

```C++
static jint     cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value);
```

和volatile 类型， CAS也是依赖不同的CPU会有不同的实现，在 src/os_cpu目录下可以看到不同的实现，以 atomic_linux_x86.inline.hpp为例， 是这么实现的：

```c++
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```

底层通过指令 cmpxchgl 来实现。如果程序是在多核环境下，还会先在 cmpxchgl前生成 lock 指令前缀，反之，如果是在单核环境下就不需要生成 lock 指令前缀。为什么多核要生成 lock 指令前缀？因为 CAS 是一个原子操作，原子操作映射到计算机实现，多核CPU的时候，如果这个操作给到了多个CPU，就会破坏原子性，所以多核环境肯定要先加一个 lock 指令，不管这个它是以**总线锁**还是以**缓存锁**来实现的，单核就不存在这个问题了。

### 4.  CAS存在的问题

#### 4.1 ABA问题

因为CAS需要在操作值得时候，检查值有没有发生变化。如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了，ABA问题的解决思路就是使用版本号，每次变更的时候更新版本号，那么 A->B->A就变成了 A1->B->A2。

再举例说明一下，线程1希望A替换为B，执行操作CAS(A,B)，此时线程2做了个操作，将 A->B变成了 A ->C，A的版本已经发生了变化，再执行线程1时会被认为还是那个 A，链表变成 B -> C。如果有 B.next = null，C这个节点就会丢失了。

从JDK1.5开始，JDK 的 Atomic 包里提供了一个类 AtomicStampedReference来解决ABA问题。这个类的 compareAndSet 方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果跟全都相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

#### 4.2 循环时间长开销大

一般 CAS 操作都是在不停的自旋。这个操作本身就有可能会失败的。如果一直不停的失败，会给CPU带来非常大的开销。

#### 4.3 只能保证一个共享变量的原子操作

看了CAS的实现就知道这个只能针对一个共享变量。如果有多个共享变量就只能使用 sychronized。除此之外，可以考虑使用AtomicReference来包装多个变量，通过这个方式来处理多个共享变量的情况。