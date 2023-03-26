## 关于JDK集合框架中的List

原文：[面渣逆袭！Java集合夺命连环三十问 - 长文多图全干货，建议收藏-今日头条 (toutiao.com)](https://www.toutiao.com/article/7213171743022776835/?app=news_article&timestamp=1679466476&use_new_style=1&req_id=20230322142755026AFBFEDBCE88B540A5&group_id=7213171743022776835&share_token=1F8B066D-CD51-411F-A65E-41EBB62470C7&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect)

这里摘录这篇文章中关于List部分的内容。

### 1. ArrayList 与 LinkedList 有什么区别？

#### 1.1 数据结构不同

* ArrayList 基于 数组 实现

* LinkedList 基于双向链表实现

    ![1](.\images\List\1.png)

#### 1.2 多数情况下，ArrayList更便于查找，LinkedList 更利于增删

* ArrayList 基于数组实现，get(int index) 可以直接通过数组下标获取，时间复杂度O(1)； LinkedList 基于双向链表实现， get(int index) 需要遍历链表，时间复杂度 O (n)；当然，get(E element) 这种查找，两种集合都需要遍历，时间复杂度都是  O (n)；
* ArrayList 增删操作如果是在数组末尾的位置，直接插入或者删除就可以了；但是，如果插入中间的位置，就需要把插入位置后的元素都向前或者向后移动，甚至还有可能触发扩容；双向链表的插入和删除只需要改变前驱节点、后继节点和插入节点的指向即可，不需要移动元素；

![2](.\images\List\2.png)

![3](.\images\List\3.png)

***注意：这里可能会出陷阱，LinkedLit 更便于增删是体现在平均步长上，不是体现在时间复杂度上，二者增删的时间复杂度都是 O(n)***

#### 1.3 是否支持随机访问

* ArrayList 基于数组，所以它可以根据下标查找，支持随机访问。当然，它也实现了 RandomAccess 接口，这个接口用来标识是否支持随机访问；
* LinkedList 基于链表，所以它没法根据序号直接获取元素，它没有实现 RandomAccess 接口，不支持随机访问

#### 1.4 内存占用

*  ArrayList 基于数组，是一块连续的内存空间；

* LinkedList 是基于链表，内存空间不连续。

    它们在空间占用上都有一些额外的消耗：

    * ArrayList 是预先定义好的数组，可能会有空的内存空间，存在一定的浪费
    * LinkedList 每个节点都需要存储前驱和后继，每个节点会占用更多的空间

### 2. ArrayList 的扩容机制

ArrayList 是基于数组的集合，数组的容量是在定义的时候确定的，如果数据满了，再插入，就会数据溢出。所以在插入的时候，需要先检查是否需要扩容，如果当前容量+1超过数据长度，就会进行扩容。

ArrayList 的扩容是创建一个 1.5 倍的新数组，然后把原来数组的值拷贝过去。

![4](.\images\List\4.png)

### 3. ArrayList 怎么序列化？为什么用 transient 修饰数组？

ArrayList 的序列化不太一样，它使用 transient 修饰存储元素的 elementData 的数组，transient 关键字的作用是让被修饰的成员属性不被序列化。

那为什么 ArrayList 不直接序列化元素数组呢？

**出于效率的考虑。** 假设数组的长度是100，但实际上只有50有元素，剩下的50是用不到的。因此后面的50个可以不用序列化，这样可以提高序列化和反序列化的效率，也可以节省内存/存储空间。

那么 ArrayList 如何序列化呢？

ArrayList 通过两个方法 readObject、writeObject 自定义序列化和反序列化策略，实际直接使用两个流 ObjectOutputStream 和 ObjectInputStream 来进行序列化和反序列化。

![5](.\images\List\5.png)

### 4. 快速失败（fail-fast）和安全失败（fail-safe）是什么？

#### 4.1 快速失败（fail-fast），是java集合的一种错误检测机制

* 在用迭代器遍历一个集合对象时，如果线程A遍历过程中，线程B对集合对象的内容进行了修改（增加、删除、修改），则会抛出 ConcurrentModificationException
* 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用了一个 modCount 变量。集合在被遍历期间如果内容发生了变化，就会改变 modCount 的值。每当迭代器使用 hasNext() / next() 遍历下一个元素之前，都会检测 modCount 变量是否为 expectedmodCount 值，是的话就返回遍历；否则抛出异常，终止遍历；
* 注意：这里异常的抛出条件时检测到 modCount != expectedmodCount 这个条件。如果集合发生变化时修改 modCount 值刚好又设置为了 exceptedmodCount 值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只是建议用于检查并发修改的bug。
* 场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改），比如ArrayList类。

#### 4.2 安全失败（fail-safe）

* 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历的
* 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException
* 缺点：基于拷贝内容的优点是避免了ConcurrentModificationException，但同样的，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的
* 场景：java.uitl.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改，比如CopyOnWriteArrayList 类



### 有几种实现 ArrayList 线程安全的方法

fail-fast 是一种可能触发的机制，实际上，ArrayList 的线程安全仍然没有保证，一般，保证 ArrayList 的线程安全可以通过下面这些方案：

* 用 Vector 替代 ArrayList （不推荐，Vector是一个历史遗留类）
* 使用 Collections.synchronizedList 包装 ArrayList，然后操作包装后的 List
* 使用 CopyOnWriteArrayList 替代 ArrayList
* 在使用 ArrayList 时，应用程序通过同步机制去控制 ArrayList 的读写



### CopyOnWriteArrayList

CopyOnWriteArrayList 就是线程安全版本的 ArrayList。

它的名字叫做 CopyOnWrite -- 写时复制，已经表明了它的原理。

CopyOnWriteArrayList 采用了一种读写分离的并发策略。CopyOnWriteArrayList 容器允许并发读，读操作是无锁的，性能较高。至于写操作，比如向容器添加一个元素，则首先将当前容器复制一份，然后在新副本上执行写操作，结束之后再将原容器的引用指向新容器。

![6](.\images\List\6.png)

