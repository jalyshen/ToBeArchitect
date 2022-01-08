# Executor：ThreadPoolExecutor

原文：https://blog.csdn.net/tongdanping/article/details/79625109



ThreadPoolExecutor 是 Executor 接口的一个重要的实现类，是线程池的具体实现，用来执行被提交的任务。

## 一. ThreadPoolExecutor 的创建

1. 直接创建 ThreadPoolExceutor 的实例对象，这样需要自己配置 ThreadPoolExecutor 中的每一个参数：

   ```java
   ThreadPoolExecutor tpe = 
       new ThreadPoolExecutor(int corePoolSize, 
                              int maximumPoolSize, 
                              long keepAliveTime, 
                              TimeUnit unit, 
                              BlockingQueue<Runnable> workQueue);
   ```

   

2. ThreadPoolExecutor 通过 Executors 工具类来创建 ThreadPoolExecutor 的子类

   FixedThreadPool、SingleThreadExecutor、CachedThreadPool，这些子类继承ThreadPoolExecutor，并且其中的一些参数已经被配置好了：

   ```java
   // FixedThreadPool
   ExecutorService ftp = Executors.newFixedThreadPool(int threadNums);
   ExecutorService ftp = Executors.newFixedThreadPool(int threadNums
                                                     ThreadFactor threadFactory);
   
   // SingleThreadExecutor
   ExecutorService ste = Executors.newSingleThreadExecutor();
   ExecutorService ste = Executors.newSingleThreadExecutor(ThreadFactory threadFactory);
   
   // CachedThreadPool
   ExecutorService cte = Executors.newCachedThreadPool();
   ExecutorService cte = Executors.newCachedThreadPool(ThreadFactory threadFactory);
   ```

   

## 二. ThreadPoolExecutor的子类

1. ### FixedThreadPool

   ```java
   public static ExecutorService newFixedThreadPool(int nThreads) {
       return 
           new ThreadPoolExecutor(nThreads, 0L,
                                  TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>())
   }
   
   public static ExecutorService newFixedThreadPool(int nThreads, 
                                       ThreadFactory threadFactory) {
       return 
           new ThreadPoolExecutor(nThreads, nThreads,
                                 0L, TimeUnit.MILLISECONDS,
                                 new LinkedBlockingQueue<Runnable>,
                                 threadFactory);
   }
   ```

   **FixedThreadPool 的应用场景**：这是一个固定线程数量的固定线程池，适用于为了满足资源管理的需求，而需要适当限制当前线程数量的情景，适用于**负载比较重**的服务器。

   

   可以看出，它的实现就是把线程池最大线程数量 maxmumPoolSize 和核心线程池的数量 corePoolSize 设置为相等，并且使用了 LinkedBlockingQueue 作为阻塞队列，那么首先可以知道线程池的线程数量最多就是 nThread，只会在核心线程池阶段创建。此外，因为 LinkedBlockingQueue 是无限的双向队列，因此当任务不能立刻执行时，都会被添加到阻塞队列中。

   因此可以得到 FixedThreadPool 的工作流大致如下：

   * 当前核心线程池总线程数量小于 corePoolSize，那么创建线程并执行任务
   * 如果当前线程数量等于 corePoolSize，那么把任务添加到阻塞队列中
   * 如果线程池中的线程执行完任务，那么获取阻塞队列中的任务并执行

   > <font color='red'>**注意**</font>：因为阻塞队列是无限的双向队列，因此如果没有调用 shutDownNow() 或者 shutDown() 方法，线程池是不会拒绝任务的。如果线程池中的线程一直被占用，FixedThreadPool 是不会拒绝任务的。
   >
   > 因为，使用的是 LinkedBlockingQueue，因此 maximunPoolSize， keepAliveTime 都是无效的，因为阻塞队列是无限的，因此线程数量肯定小于 corePoolSize，因此 keepAliveTime 是无效的。

   

2. ### SingleThreadExecutor

   ```java
   public static ExecutorService newSingleThreadExecutor() {
       return new FinalizableDelegatedExecutorService (
       	new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                          new LinkedBlockingQueue<Runnable>()));
   }
   
   public staic ExecutorService newSingleThreadExecutor(
       ThreadFactory threadFactory) {
       return new FinalizableDelegatedExecutorService (
       	new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                          new LinkedBlockingQueue<Runnable>(),
                          threadFactory));
   }
   ```

   **SingleThreadExecutor 应用场景**：只有一个线程的线程池，常用于需要让线程**顺序执行**，并且在任意时间，只能有一个任务被执行，而不能有多个线程同时执行的场景。

   因为使用了 LinkedBlockingQueue，因此和 FixedThreadPool 一样，maximumPoolSize 、keepAliveTime 都是无效的。 corePoolSize 为1，因此最多只能创建一个线程。

   SingleThreadPool 的工作流程大致如下：

   * 当前核心线程池总数量小于 corePoolSize(1)，那么创建线程并执行任务
   * 如果当前线程数量等于 corePoolSize，那么把任务添加到阻塞队列中
   * 如果线程池中的线程执行完任务，那么获取阻塞队列中的任务并执行

   

3. ### CachedThreadPool

   ```java
   public static ExecutorService newCachedThreadPool() {
       return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                            new SynchronousQueue<Runnable>());
   }
   
   public static ExecutorService new newCachedThreadPool(
       ThreadFactory threadFactory) {
         return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                            new SynchronousQueue<Runnable>(),
                                      threadFactory);
   }
   ```

   **CachedThreadPool 应用场景**：适用于执行很多**短期异步**任务的小程序，或者是负**载较轻的**服务器。

   CachedThreadPool 使用 SynchronousQueue 作为阻塞队列，SynchronousQueue 是不存储元素的阻塞队列，实现“一对一的交付”。也就是说，每次向队列中 put 一个任务必须等有线程来 take 这个任务，否则就会一直阻塞该任务。如果一个线程要 take 一个任务，就要一直阻塞直到有任务被 put 进阻塞队列。

   因为 CachedThreadPool 的 maximumPoolSize 为 Integer.MAX_VALUE，因此 CachedThreadPool 是无界的线程池，也就是说可以一直不断的创建线程。 corePoolSzie 为 0，因此在CachedThread Pool 中直接通过阻塞队列来进行任务的提交。

   CachedThreadPool 的工作流程大致如下：

   * 首先执行 SynchronousQueue.offer() 把任务提交给阻塞队列。如果这时候正好在线程池中有空闲的线程执行SynchronousQueue.pull()，那么 offer 操作和 poll 操作配对，线程执行任务
   * 如果执行 SynchronousQueue.offer() 把任务提交给阻塞队列时 maximumPoolSize =0，或者没有空闲线程来执行 SynchronousQueue.poll() ，那么步骤一失败，那么创建一个新线程来执行任务
   * 如果当前线程执行完任务则循环从阻塞队列中获取任务。如果当前队列中没有提交（offer）任务，那么线程等待 keepAliveTime 的时间，在 CachedThreadPool 中为 60 秒，在 keepAliveTime 时间内如果有任务提交则获取并执行任务，如果没有则销毁线程。因此最后如果一直没有任务提交了，线程池中的线程数量最终为0

   > 注意：因为 maximumPoolSize = Integer.MAX_VALUE，因此可以不断的创建新线程，这样可能会导致CPU和内存资源耗尽

