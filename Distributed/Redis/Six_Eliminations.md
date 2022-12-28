# Redis 6种淘汰机制

原文：https://www.toutiao.com/article/7130210248756920864/?log_from=de34dd61aee878_1660180029851



Redis 是一个基于内存的缓存数据库，既然是基于内存的，那就肯定会有存满的时候。如果真的填满了，无法添加新的数据。此时Redis就需要执行既定的一些淘汰策略。Redis提供了六种淘汰策略。

### 1. 具体的六种策略

1. **noeviction**（默认策略）：对于写请求不再提供服务，直接返回错误（DEL 请求和部分特殊请求除外）
2. **alleys-lru** ：从所有 key 中使用 LRU 算法进行淘汰（LRU算法：即最近最少使用算法）
3. **volatile-lru**：从设置过期时间的 key 中使用 LRU 算法进行淘汰
4. **allleys-random**：从所有 key 中随机淘汰数据
5. **volatile-random**：从设置了过期时间的 Key 中随机淘汰
6. **volatile-ttl**：在设置了过期时间的 Key 中，淘汰过期时间剩余最短的

当使用 volatile-lru、volatile-random、volatile-ttl 这三种策略时，如果没有 key 可以被淘汰，则和 noeviction 一样返回错误。

### 2. 如何获取及设置内存淘汰策略

1. 获取当前内粗淘汰策略：

   ``` shell
   127.0.0.1:6379> config get maxmemory-policy
   ```

   可以看到当前使用的 noeviction 策略

2. 获取Redis能使用的最大内存大小

     ``` shell
     127.0.0.1:6379> config get maxmemory
     ```

   如果不设置最大内存大小或者设置最大内存大小为0，在 64 位OS下不限制内存你大小；在32位OS下最多使用 3GB 内存。 32位的机器最大只支持4GB的内存，而系统本身需要一定的内存资源来支持运行。

3. 设置淘汰策略

   通过配置文件设置淘汰策略（修改 redis.conf 文件）：

   ``` shell
   maxmemory-policy allkeys-lru
   ```

   通过命令修改淘汰策略：

   ``` shell
   127.0.0.1:6379> config set maxmemory-policy allkeys-lru
   ```

4. 设置Redis最大占用内存大小

   ```shell
   # 设置 Redis 最大占用内存大小为100M
   127.0.0.1:6379> config set maxmemory 100mb
   ```

   