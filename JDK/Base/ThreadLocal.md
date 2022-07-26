# ThreadLocal 介绍

原文：https://zhuanlan.zhihu.com/p/102744180



通过一下几个角度来分析：

1. ThreadLocal 是什么
2. ThreadLocal 怎么用
3. ThreadLocal 源码分析
4. ThreadLocal 内存泄漏问题

说明：源码基于 JDK 1.8

### 1. ThreadLocal 是什么

ThreadLocal 叫做**本地线程变量**，意思是，ThreadLocal 中填充的是当前线程的变量。该变量对其他线程而言是封闭且隔离的，ThreadLocal 为变量在每个线程中创建了一个副本，这样每个线程都可以访问自己内部的副本变量。

从字面意思很容易理解，但是实际角度就没那么容易了。ThreadLocal 的使用场景也很丰富：

* 在进行对象跨层传递的时候，使用 ThreadLocal 可以避免多次传递，打破层次间的约束

* 线程间数据隔离

* 进行事务操作，用于存储线程事务信息

* 数据库连接，Session 会话管理

  

### 2. ThreadLocal 怎么用

先来看个使用的例子：

```java
public class ThreadLocalTest02 {

    public static void main(String[] args) {
        ThreadLocal<String> local = new ThreadLocal<>();
        IntStream.range(0, 10).forEach(i -> new Thread(() -> {
            local.set(Thread.currentThread().getName() + ":" + i);
            System.out.println("线程：" + 
                               Thread.currentThread().getName() + 
                               ",local:" + local.get());
        }).start());
    }
}
```

结果输出：

```shell
线程：Thread-0,local:Thread-0:0
线程：Thread-1,local:Thread-1:1
线程：Thread-2,local:Thread-2:2
线程：Thread-3,local:Thread-3:3
线程：Thread-4,local:Thread-4:4
线程：Thread-5,local:Thread-5:5
线程：Thread-6,local:Thread-6:6
线程：Thread-7,local:Thread-7:7
线程：Thread-8,local:Thread-8:8
线程：Thread-9,local:Thread-9:9
```

从结果看，每个线程都有自己的 local 值，这就是 ThreadLocal 的基本使用。

下面从 JDK 源码角度分析一下 ThreadLocal 的工作原理。

### 3. ThreadLocal 源码分析

#### 3.1 set 方法

```java
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the
     * {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's 
     * copy of this thread-local.
     */
    public void set(T value) {
        //首先获取当前线程对象
        Thread t = Thread.currentThread();
        //获取线程中变量 ThreadLocal.ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        //如果不为空，
        if (map != null)
            map.set(this, value);
        else
            // 如果为空，初始化该线程对象的map变量，
            // 其中 key 为当前的threadlocal 变量
            createMap(t, value);
    }

    /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     */
    //初始化线程内部变量 threadLocals ，key 为当前 threadlocal
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

       /**
         * Construct a new map initially 
         *      containing (firstKey, firstValue).
         * ThreadLocalMaps are constructed lazily, so we only create
         * one when we have at least one entry to put in it.
         */
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & 
                         (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }


        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

ThreadLocalMap 作为 ThreadLoacl 的一个静态内部类，里面定义了 Entry 来保存数据。而且是继承的弱引用。在 Entry 内部使用 ThreadLocal 作为key，使用设置的 value 作为 value。

对于那个线程内部有个 ThreadLocal.ThreadLocalMap 变量，存取值的时候，也是从这个容器中来获取。

#### 3.2 get 方法

```java
    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

看看 **Thread** 类中定义的 threadLocals：

```java
    /* ThreadLocal values pertaining to this thread. 
     * This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
```

通过 上面的分析，应该对 ThreadLocal 有所了解了。首先获取当前线程，然后通过 key threadlocal 获取设置的 value。

问题：为啥key 是 threadlocal，而不是 当前的 thread ？

### 4. ThreadLocal 内存泄漏问题

首先来看这个类：

```java
        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

注释说的清洗，*Note that null keys (i.e.  entry.get() == null)*， 如果 *key threadlocal* 为 null了，这个 entry 就可以清除了。 ThreadLocal 是一个弱引用，当为 null 时， 会被当成垃圾回收。

 ![1](.\images\ThreadLocal\1.jpg)

重点来了，突然 ThreadLocal 是 null 了，也就是要被垃圾回收器回收了，但是此时 ThreadLocalMap（thread 的内部属性）生命周期和 Thread 的一样，它不会回收，这个时候就出现了一个现象，那就是 ThreadLocalMap 的 Key 没了，但是对应的 value 还在，从而造成了内存泄漏。

**解决办法**：使用完 TreadLocal 后，执行 remove 操作，避免出现内存溢出情况。

所以，如同 lock 的操作，最后要执行解锁操作一样， ThreadLocal 使用完毕一定要记得执行 remove 方法，清除当前线程的数值。如果不 remove 当前线程对应的 Value， 就会一直存在这个值。

使用了线程池，可以达到“线程复用”的效果。但是归还线程之前记得清除 ThreadLocalMap ，要不然再取出该线程的时候， ThreadLocal 变量还会存在。这就不仅仅是内存泄漏的问题了，整个业务逻辑都可能出现错误。

### 5. 为什么 Key 使用弱引用？

如果使用强引用，当 ThreadLocal 对象的引用（强引用）被回收了，ThreadLocalMap 本身依然还持有 ThreadLocal 的强引用，如果没有手动删除这个 Key，则 ThreadLocal 不会被回收，所以只要当前线程不消亡， ThreadLocalMap 引用的那些对象就不会被回收，可以认为导致 Entry 内存泄漏。

* 强引用：普通的引用，强引用指向的对象不会被回收
* 软引用：仅有软引用指向的对象，只有发生GC且内存不足，才会被回收
* 弱引用：仅有弱引用指向的对象，只要发生GC就会被回收