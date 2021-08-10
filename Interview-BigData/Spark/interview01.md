# Spark面试题一

原文：https://www.cnblogs.com/think90/p/11461367.html



## 1. Spark中的RDD是什么，有何特性？

​        RDD（Resilient Distributed Dataset）叫做**分布式数据集**，是Spark中最基本的数据抽象，它代表一个**不可变，可分区**，里面**元素可以并行计算**的集合。

​        Dataset：一个集合，用于存放数据

​        Distributed：分布式，可以并行在集群计算

​        Resilient：表示弹性的。这里的“**弹性**”表示：

* RDD中的数据可以存储在内存或者磁盘中
* RDD中的分区是可以改变的

​        RDD具有5大特性：

1. A list of partitions: 一个分区列表，RDD中的数据都存储在一个分区列表中
2. A function for computing each split: 作用在每一个分区中的函数
3. A list of dependencies on other RDDs: 一个RDD依赖于其他多个RDD，这点很重要，RDD的容错机制就是依据这个特性而来的
4. Optionally, a Partitioner for key-value RDDs (eg: to say that the RDD is hash-partitioned)：可选的，针对于K-V 类型的RDD才有这个特性，作用时决定了数据的来源以及数据处理后的去向
5. 可选项，数据本地性，数据位置最优

## 2. Spark 中常用的算子区别

​        主要是涉及这几个：map， mapPartitions，foreach， foreachPatition

* map：用于遍历RDD，将函数应用于每一个元素，返回新的RDD（transformation算子）
* mapPartitions：用于遍历操作RDD中的每一个分区，返回生成一个新的RDD（transformation算子）
* foreach：用于遍历RDD，将函数应用于每一个元素，无返回值（action算子）
* foreachPatition：用于遍历操作RDD中的每一个分区，无返回值（action算子）

​        总结：一般使用mapPartitions和foreachPatition算子比map和foreach更加高效，推荐使用。

## 3. 谈谈Spark中的宽窄依赖

​        RDD和它的父RDD的关系，存在两种类型：**窄依赖** 和 **宽依赖**

### 3.1 窄依赖

​        每一个父RDD的Partition最多被子RDD的一个Partition使用，是**一对一**的关系，也就是父RDD的一分区去到了子RDD的一个分区中，这个过程没有shuffle产生。

* 输入输出一对一的算子，且结果RDD的分区结构不变，主要是map/flatmap

* 输入输出一对一的算子，但结果RDD的分区结构发生了变化，如union/coalesce

​        从输入中选择部分元素的算子，如filter，distinc，substrack，sample

### 3.2 宽依赖

​        多个子RDD的Partition会依赖同一个父RDD的Partition，关系是**一对多**，父RDD的一个分区的数据去到了RDD的不同分区里，会有shuffle的产生。

* 对单个RDD基于Key进行重组和Reduce，如groupByKey，reduceByKey
* 对两个RDD基于Key进行join和重组，如join

​        经过大量shuffle生成的RDD，建议进行缓存，这样避免失败后重新计算带来的开销。

*注：reduce是一个action，和reduceByKey完全不同*



​        区分的标准就是看父RDD的一个分区的数据的流向，要是流向一个Partition的话，就是窄依赖；否则就是宽依赖。如下图所示：

![1](./images/interview01/1.png)

## 4. Spark如何划分stage

​        先说说概念，Spark任务会根据RDD之间的依赖关系，形成一个DAG有向无环图，DAG会提交给DAGScheduler，DAGScheduler会把DAG划分相互依赖的多个stage，划分依据就是宽窄依赖。**遇到宽依赖就划分stage**，每个stage包含一个或多个task，然后将这些task以taskSet的形式提交给TaskScheduler运行，stage是由一组并行的task组成。

1. Spark程序中可以因为不同的action触发众多的job，一个程序中可以有很多的job，每个job是由一个或者多个stage构成，后面的stage依赖于前面的stage。也就是说，只有前面依赖的stage计算完毕后，后面的stage才会运行

2. stage的划分标准就是宽窄依赖：何时产生宽窄依赖就会产生一个新的stage。例如：reuduceByKey，groupByKey，join的算子，会导致宽依赖的产生

3. 切割规则：从后往前，遇到宽依赖就切割stage

4. 图解

   ![2](./images/interview01/2.png)

5. 计算格式：pipeline管道计算模式，pipeline只是一种计算思想，一种模式。如图：

   ![3](./images/interview01/3.png)

6. Spark的Pipeline管道计算模式相当于执行了一个高阶函数，也就是说，来一条数据，然后计算一条数据，会把所有的逻辑走完，然后落地；而MapReduce是1+1=2，2+1=3这样的计算模式，也就是计算完落地，然后再计算，然后在落地到磁盘或者内存，最后数据是落在计算节点上，按reduce的hash分区落地。*（类Pipeline似于RMDB的批量提交；而MR则是执行一条就立即提交）*

   管道计算模式完全计划于内存的计算，所以比MapReduce 快

7. 管道中的RDD何时落地呢？shuffle write的时候；或者对RDD进行持久化的时候

8. stage的task的并行度是有stage的最后一个RDD的分区数来决定的。一般的说，一个partition对应一个task，但最后reduce的时候可以手动改变reduce的个数，也就是改变最后一个RDD的分区数，也就是改变了并行度。例如：reduceByKey(\_+_,3)

9. 优化：提高stage的并行度： reduceByKey(\_+_, partiton的个数)，join(\_+_, patition的个数)

## 5. DAGScheduler分析

​        DAGScheduler是一个**面向Stage的调度器**，主要入参有：

```java
dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, allowLocal, resultHandler, localProperties.git)
```

其中：

* rdd：final RDD
* cleanedFunc：计算每个分区的函数
* resultHander：结果侦听器

主要功能：

1. 接受用户提交的Job
2. 将Job根据类型划分为不同的Stage，记录那些RDD，Stage被物化，并在每一个Stage内产生一系列的task，并封装成taskset
3. 决定每个task的最佳位置，任务在数据所在节点上运行，并结合当前的缓存情况，将taskset提交给TaskScheduler
4. 重新提交Suffle输出丢失的stage给taskScheduler

注：一个 Stage 内部的错误不是由 shuffle 输出丢失造成的，DAGScheduler是不管的，由TaskScheduler负责尝试重新提交task执行。

## 6. Job的生成

​        一旦 driver 程序中出现action，就会生成一个job，比如count等，向 DAGScheduler 提交job，如果 driver 程序后面还有 action，那么其他 action 也会对应生成响应的 job，所以，driver 端有多少action，就会提交多少 job，这可能就是为什么 spark 将 driver 程序称为 application 而不是 job 的原因。

​        每一个job 可能会包含一个或者多个 stage，最后一个stage生成result，在提交job的过程中，DAGScheduler 会首先从后往前划分stage，划分的标准就是宽依赖，一旦遇到宽依赖就划分，然后提交没有父阶段的stage们，并在提交过程中，计算该stage的task数目以及类型，并提交具体的task，这些无父阶段的stage提交完成之后，依赖该Stage的stage才会提交。

## 7. 有向无环图

​        DAG，有向无环图，简单的说，就是一个由顶点和有方向性的便构成的图中，从任意一个顶点出发，没有任意一条路径会将其带回到出发点的顶点位置。

​        为每个spark job 计算具有依赖关系的多个stage任务阶段，通常根据shuffle来划分stage，如reduceByKey，groupByKey等涉及到shuffle的transformation就会产生新的stage，然后将每个stage划分为具体的一组任务，以TaskSets的形式提交给底层的任务调度模块来执行，其中不同stage之前的RDD为宽依赖关系，TaskScheduler任务调度模块负责具体启动任务，监控和汇报任务运行情况。

## 8. RDD的操作

​        RDD支持两大类的操作：

* **转换**（Transformation）：现有的RDD通关转换生成一个新的RDD，转换是延时执行的
* **动作**（Actions）：在RDD上运行计算后，返回结果给驱动程序或写入文件系统

​        例如，map就是一种transformation，它将数据集每一个元素都传递给函数，并返回一个新的分布数据集表示结果；reduce则是一种action，通过一些函数将所有的元素叠加起来，并将最终结果返回给Driver程序。

### 8.1 Transformation

1. **map(func)**：

   Return a new distributed dataset formed by passing each element of the source through a function *func*

   返回一个新分布式数据集，由每一个输入元素经过func函数转换后组成

2. **filter(func)**：

   Return a new dataset formed by selecting those elements of the source on which func returns true.

   返回一个新的数据集，由经过func函数计算后返回值为true的输入元素组成

3. **flatMap( func )**：

   Similar to map, but each input item can be mapped to 0 or  more output items ( so func should return a Seq rather than a single item)

   类似于map，但是每一个输入元素可以被映射为0个或者多个输出元素（因此，func函数应该返回一个序列，而不是单一元素）

4. **mapPartitions( func )**：

   Similarto map, but runs separately on each petition (block) of the RDD, so func must be of type Iterator<T> => Iterator<U> when running on an RDD of Type T.

   类似于map，但独立地在RDD的每一个分块上运行，因此在类型为T的RDD上运行时，func的函数类型必须是Iterator<T> => Iterator<U>，mapPartitions将会被每一个数据集分区调用一次，各个数据集分区的全部内容将作为顺序的数据流川入函数func的参数中，func必须返回另一个Iterator<T>。被合并的结果自动转换成为新的RDD。

5. **groupByKey( [numTasks] )**：

   When called on a dataset of (K,V) pairs, returns a dataset of (K, Iterable)  pairs.

   Note: If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using reduceByKey or combineByKy will yield much better performance.

   Note: By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional numTasks argument to set a different number of tasks.

   在一个（K，V）对的数据集上调用，返回一个（K，Seq[V]）对的数据集

   注意：默认情况下，只有8个并行任务来做操作，但是可以传入一个可选的numTasks参数来改变它。如果分组时用来计算聚合操作（如sum或者average），那么应该使用reduceByKey或者combineByKey来提供更好的性能。

6. **reduceByKey( func, [numTasks] )**：

   When called on a dataset of (K, V) pairs, returns a dataset of (K,V) pairs where the values for each key are aggregated using the given reduce function func, which must be of type (V,V) => V. Like ingroupByKey, the number of reduce tasks is configurable through an optional second argument.

   在一个（K，V）对的数据集上调用时，返回一个（K，V）对的数据集，使用指定的reduce函数，将相同Key的值聚合到一起，类似groupByKey，reduce任务个数是可以通过第二个可选参数来配置的。

7. **sortByKey( [ascending], [numTasks] )**

   When called on a dataset of (K,V) pairs where K implements Ordered, returns a dataset of (K,V) pairs sorted by keys in ascending or descending order, as specified in the boolean ascending argument.

   在一个（K，V）对的数据集上调用，K必须实现Ordered接口，返回一个按照Key进行排序的（K，V）对数据集，圣墟或者降序由ascending布尔参数决定

8. **join( otherDataset, [numTasks] )**

   When called on dataset of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are also supported through leftOuterJoin and rightOuterJoin.

   类行为（K，V）和（K，W）类型的数据集上调用时，返回一个相同Key对应的所有元素对在一起的（K， （V，W））数据集

9. **collect()**

   Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data.

   在驱动（应用）程序中，以数组的形式，返回数据集的所有元素。这通常会在使用filter或者其他操作并返回一个足够小的数据子集后再使用会比较有用。

10. **count()**

    Return the number of elements in the dataset.

    返回数据集的元素的个数。

11. **foreach( func )**

    Run a function func on each element of the dataset. This is usually done for side elffects such as updating an accumulator variable (see below) or interacting with external storage systems.

    在数据集的每一个元素上，运行函数func进行更新，这通常用于边缘效果。例如更新一个累加器，或者和外部存储系统进行交互，如HBase

12. **saveAsTextFile( path )**
    Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supporte file system. Spark will call toString on each element to convert it to a line of text in the file.

    将数据集的元素，以textfile的形式，保存到本地文件系统，HDFS或者任务其他Hadoop支持的文件系统，对于每个元素，Spark将会调用toString方法，将它转换为文件中的文本行。

## 9 RDD缓存

​        Spark 可以使用 persist 和 cache 方法将任务RDD缓存到内存、磁盘文件系统中。缓存是容错的，如果一个RDD分片丢失，可以通过构建它的 transformation 自动重构。被缓存的 RDD 被使用时，存取速度会被大大加速。一般的，executor内存60%做cache，剩下40%做task。

​        Spark中，RDD 类可以使用 cache() 和 persist() 方法来缓存。 cache() 是 persist() 的特例。将该 RDD 缓存到内存中。而 persist() 可以指定一个 StorageLevel。 StorageLevel的列表可以再StorageLevel 伴生单例对象中找到。

​       Spark的不同StorageLevel，目的满足内存使用和CPU效率权衡上的不同需求。建议通过一下步骤来进行选择：

* 如果RDDs可以很好的与默认的存储级别（MEMORY_ONLY）契合，就不需要做任何修改了。这已经是CPU使用率最高的选贤，它使得RDDs的操作尽可能的块
* 如果不行，试着使用 MEMORY_ONLY_SER 并且选择一个快速序列化的库，使得对象在有比较高的空间使用率的情况下，依然可以较快被访问
* 尽可能不要存储到硬盘上，除非计算数据集的函数，计算量特别大，或者它们过滤了大量的数据。否则，重新计算一个分区的速度，和与从硬盘中读取基本差不多快
* 如果想有快速故障恢复能力，使用复制存储级别（例如：用Spark来响应web应用的请求）。所有的存储级别都有通过重新计算丢失数据恢复错误的容错机制，但是复制存储级别可以在RDD上持续的运行任务，而不需要等待丢失的分区被重新计算
* 如果想要自定义存储级别（比如复制因子为3而不是2），可以使用 StorageLevel 单例对象的 apply() 方法。

在不使用 cached RDD的时候，及时使用unpersist方法来释放它。

## 10 Spark中的cache和persist的区别

* **cache**：缓存数据，默认是缓存在内存中，其本质还是调用persist
* **persist**：缓存数据，有丰富的数据缓存策略。数据可以保存在内存，也可以保存在磁盘中，使用的时候指定对应的缓存级别就可以了。

## 11 RDD共享变量



