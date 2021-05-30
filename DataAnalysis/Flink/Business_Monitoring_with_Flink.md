# Flink在业务监控中的应用

原文：https://engineering.zalando.com/posts/2017/07/complex-event-generation-for-business-process-monitoring-using-apache-flink.html



这是一篇个人翻译的文章。如果晦涩难懂，请参考原文链接。

## 业务流程

首先，先来熟悉一下问题领域。看看我们对业务流程监控的解决方案。

> *To start off, we would like to offer more context on the problem domain. Let’s begin by having a look at the business processes monitored by our solution.*



**业务流程**，简单的说，就是相关事件链。它具有一个“开始”和一个“完成”事件。如下图所示：

> *A **business process** is, in its simplest form, a chain of correlated events. It has a start and a completion event. See the example depicted below:*

![1](./images/Business_Monitoring_with_Flink/1.webp)



示例中的“**开始事件**“是”ORDER_CREATED"。每当客户下单时，Zalando平台（内部）就会生成一个"ORDER_CREATED"事件。这个事件的大致是如下的JSON结构：

> *The **start event** of the example business process is ORDER_CREATED. This event is generated inside Zalando’s platform whenever a customer places an order. It could have the following simplified JSON representation:*

```json
{
 "event_type":          "ORDER_CREATED",
 "event_id":            1,
 "occurred_at":         "2017-04-18T20:00:00.000Z",
 "order_number":        123
}
```



“**完成事件**”命名为"ALL_PARCELS_SHIPPED"，这个事件的意思是，一个订单的包含的所有包裹都已经打包好，可以提供给物流公司运输了。这个事件的JSON结构如下：

> *The **completion event** is ALL_PARCELS_SHIPPED. It means that all parcels pertaining to an order have been handed over for shipment to the logistic provider. The JSON representation is therefore:*

```json
{
 "event_type":      "ALL_PARCELS_SHIPPED",
 "event_id":        11,
 "occurred_at":     "2017-04-19T08:00:00.000Z",
 "order_number":    123
}
```



注意到，这两个事件，都与**订单号**（order_number）相关联，而且它们都根据**发生时间**（occurred_at）排序。

> *Notice that the events are correlated on **order_number**, and also that they occur in order according to their **occurred_at** values.*



所以，可以监控这两个事件的间隔时间。假设，设定一个特定的阈值，比如7天，那么就可以判单哪些订单已经过期，然后就可以采取一些措施，保证相关的包裹立即发货。这样一来，就可以保证客户的满意度。

> *So we can monitor the time interval between these two events, ORDER_CREATED and ALL_PARCELS_SHIPPED. If we specify a threshold, e.g. 7 days, we can tell for which orders the threshold has been exceeded and then can take action to ensure that the parcels are shipped immediately, thus keeping our customers satisfied.*



## 问题陈述

**复杂事件**，就是从其它事件“**推测**”出来的事件。

> *A **complex event** is an event which is inferred from a pattern of other events.*



还是用业务流程的例子，比如，从一系列 "PARCEL_SHIPPED" 事件，通过一定的规则（pattern）可以“推测”出 ALL_PARCELS_SHIPPED 事件。例如：在7天内，系统收到某一个订单的所有 "PAECEL_SHIPPED"事件，那么就可以生成一个 “ALL_PARCELS_SHIPPED”事件；相反的，如果过了7天，只收到部分 “PARCEL_SHIPPED” 事件，那么就可以生成一个警告事件 “THRESHOLD_EXCEEDED”。

> *For our example business process, we want to infer the event ALL_PARCELS_SHIPPED from a pattern of PARCEL_SHIPPED events, i.e. generate ALL_PARCELS_SHIPPED when all distinct PARCEL_SHIPPED events pertaining to an order have been received within 7 days. If the received set of PARCEL_SHIPPED events is incomplete after 7 days, we generate the alert event THRESHOLD_EXCEEDED.*



假设，如果提前知道某个订单需要发货多少个包裹，就可以知道接收到多少个 “PARCEL_SHIPPED” 事件表示发货已经结束了。这个信息，实际上包含在 “ORDER_CREATED” 事件的某个属性里，比如属性 "parcels_to_ship":3* 。

> *We assume that we know beforehand how many parcels we will ship for a specific order, thus allowing us to determine if a set of PARCEL_SHIPPED events is complete. This information is contained in the ORDER_CREATED event in the form of an additional attribute, e.g. "parcels_to_ship":  3.*



此外，假设这些事件按顺序依次发出去后，例如， 事件"ORDER_CREATED" 的 “occurred_at” 时间戳应该 **小于** 事件“PARCEL_SHIPPED”事件的时间戳。

> *Furthermore, we assume that the events are emitted in order, i.e. the **occurred_at** timestamp of ORDER_CREATED is smaller than all of the PARCEL_SHIPPED’s timestamps.*



另外，要求完成事件 "ALL_PARCELS_SHIPPED" 也有一个时间戳，这个时间戳与最后一个 “PARCEL_SHIPPED” 事件的时间戳相等。

> *Additionally we require the complex event ALL_PARCELS_SHIPPED to have the timestamp of the last PARCEL_SHIPPED event.*



下面流程图展示了这个原始的思路：

> *The raw specification can be represented through the following flowchart:*

![2](./images/Business_Monitoring_with_Flink/2.webp)



现在，使用Flink替代Kafka来处理这些事件。

> *We process all events from separate Apache Kafka topics using Apache Flink. For a more detailed look of our architecture for business process monitoring, [please have a look here](https://www.slideshare.net/ZalandoTech/stream-processing-using-apache-flink-in-zalandos-world-of-microservices-reactive-summit/33).*



## 生成复合事件 (Generating Complex Events)

现在，已经知道了问题是什么了，可以着手解决问题了：生成复合事件 “ALL_PARCELS_SHIPPED” 和 “THRESHOLD_EXCEEDED”

> *We now have all the required prerequisites to solve the problem at hand, which is to generate the complex events ALL_PARCELS_SHIPPED and THRESHOLD_EXCEEDED.*



首先，看看Flink的Job的实现方式：

> *First, let’s have an overview on our Flink job’s implementation:*

![3](./images/Business_Monitoring_with_Flink/3.webp)

1. 从 Kafka 读取事件 "ORDER_CREATED" 和 "PARCEL_SHIPPED"
2. 为了事件的时间处理，给事件添加“水印”（watermark）信息
3. 通过关键属性，如 *order_number*，把属于同一个订单的事件归并到一起
4. 给每个 *oder_number* 的客户事件触发器分配一个 *TumblingEventTimeWindows*
5. 在事件触发前，在触发窗口时间内给事件排序。触发器会检查在触发窗口内，水印是否被赋予了一个更大的时间戳。这是为了保证当前触发窗口内**按顺序**收集到足够多的元素
6. 分配第二个 *TumblingEventTimeWindows*，同时设定它的触发点是7天，并添加自定义累加器和时间触发器
7. 通过计数器结果触发，并生成 *ALL_PARCELS_SHIPPED* 事件；或者根据时间触发，同时生成 *THRESHOLD_EXCEEDED* 事件。计数器的结果由事件 *ORDER_CREATED* 的属性 “*parcels_to_ship*” 决定
8. 把数据流按照事件  *ALL_PARCELS_SHIPPED* 和 *THRESHOLD_EXCEEDED* 拆分成两组，然后把数据流写入到Kafka里

> 1. *Read the Kafka topics ORDER_CREATED and PARCEL_SHIPPED.*
> 2. *Assign watermarks for event time processing.*
> 3. *Group together all events belonging to the same order, by keying by the correlation attribute, i.e. order_number.*
> 4. *Assign TumblingEventTimeWindows to each unique order_number key with a custom time trigger.*
> 5. *Order the events inside the window upon trigger firing. The trigger checks whether the watermark has passed the biggest timestamp in the window. This ensures that the window has collected enough elements to order.*
> 6. *Assign a second TumblingEventTimeWindow of 7 days with a custom count and time trigger.*
> 7. *Fire by count and generate ALL_PARCELS_SHIPPED or fire by time and generate THRESHOLD_EXCEEDED. The count is determined by the "parcels_to_ship" attribute of the ORDER_CREATED event present in the same window.*
> 8. *Split the stream containing events ALL_PARCELS_SHIPPED and THRESHOLD_EXCEEDED into two separate streams and write those into distinct Kafka topics.*



The simplified code snippet is as follows:

```java
// 1
List topicList = new ArrayList<>();
topicList.add("ORDER_CREATED");
topicList.add("PARCEL_SHIPPED");
DataStream streams = env.addSource(
      new FlinkKafkaConsumer09<>(topicList, new SimpleStringSchema(), properties))
      .flatMap(new JSONMap()) // parse Strings to JSON

// 2-5
DataStream orderingWindowStreamsByKey = streams
      .assignTimestampsAndWatermarks(new EventsWatermark(topicList.size()))
      .keyBy(new JSONKey("order_number"))
      .window(TumblingEventTimeWindows.of(Time.days(7)))
      .trigger(new OrderingTrigger<>())
      .apply(new CEGWindowFunction<>());

// 6-7
DataStream enrichedCEGStreams = orderingWindowStreamsByKey
     .keyBy(new JSONKey("order_number"))
     .window(TumblingEventTimeWindows.of(Time.days(7)))
     .trigger(new CountEventTimeTrigger<>())
     .reduce((ReduceFunction) (v1, v2) -> v2); // always return last element

// 8
enrichedCEGStreams
      .flatMap(new FilterAllParcelsShipped<>())
      .addSink(new FlinkKafkaProducer09<>(Config.allParcelsShippedType,
         new SimpleStringSchema(), properties)).name("sink_all_parcels_shipped");
enrichedCEGStreams
      .flatMap(new FilterThresholdExceeded<>())
      .addSink(new FlinkKafkaProducer09<>(Config.thresholdExceededType,
         newSimpleStringSchema(), properties)).name("sink_threshold_exceeded");
```



## 挑战与学习 （Challenges and Learnings）

### CEG触发的条件需要有序的事件

> **The firing condition for CEG requires ordered events**



根据之前问题的陈述，需要让事件 *ALL_PARCELS_SHIPPED* 拥有最后一个 *PARCEL_SHIPPED* 事件里的“事件时间”信息。触发器 *CountEventTimeTrigger* 触发的条件是，需要在窗口内的事件按顺序排序，所以可以获得最后一个 *PARCEL_SHIPPED* 

> As per our problem statement, we need the ALL_PARCELS_SHIPPED event to have the event time of the last PARCEL_SHIPPED event. The firing condition of the CountEventTimeTrigger thus requires the events in the window to be in order, so we know which PARCEL_SHIPPED event is last.



在上述代码的2-5步，实现了对事件的排序。接收到每个元素（事件）后，会把**最大的时间戳**作为关键状态存储起来。在注册时间时，触发器会检查水印的值是否大于最大的时间戳。如果是，说明当前窗口内已经收集到了足够的元素（事件）并已经排好了序。我们通过让水印只在所有事件中最早的时间戳中进行来保证这一点。请注意，按窗口状态的大小对事件进行排序代价很高，因为窗口状态是存储在内存中的。

> We implement the ordering in steps 2-5. When each element comes, the keyed state stores the biggest timestamp of those elements. At the registered time, the trigger checks whether the watermark is greater than the biggest timestamp. If so, the window has collected enough elements for ordering. We assure this by letting the watermark only progress at the earliest timestamp among all events. Note that ordering events is expensive in terms of the size of the window state, which keeps them in-memory.



### 事件以不同的速率到达窗口

> **Events arrive in windows at different rates**



从Kafka的2个不同的Topic读取事件流：ORDER_CREATED 和 PARCEL_SHIPPED。 前者的规模比后者的大得多，所以消费前者就会比后者慢。

> We read our event streams from two distinct Kafka topics: ORDER_CREATED and PARCEL_SHIPPED. The former is much bigger than the latter in terms of size. Thus, the former is read at a slower rate than the latter.



事件到达窗口的速率是不同的。这会影响业务逻辑的实现，特别是OrderingTrigger的触发条件。它通过保持最小的可见时间戳作为水印来等待两种事件类型到达相同的时间戳。事件在窗口的状态中堆积，直到触发器触发并清除它们。具体来说，如果主题ORDER_CREATED中的事件从1月3日开始，PARCEL_SHIPPED中的事件从1月1日开始，后者将堆积起来，只有在Flink在1月3日处理完前者之后才会被清除。这将消耗大量内存。

> Events arrive in the window at different speeds. This impacts the implementation of the business logic, particularly the firing condition of the OrderingTrigger. It waits for both event types to reach the same timestamps by keeping the smallest seen timestamp as the watermark. The events pile up in the windows’ state until the trigger fires and purges them. Specifically, if events in the topic ORDER_CREATED start from January 3rd and and the ones in PARCEL_SHIPPED start from January 1st, the latter will be piling up and only purged after Flink has processed the former at January 3rd. This consumes a lot of memory.



### 一些事件在计算开始时就不正确

> **Some generated events will be incorrect at the beginning of the computation**



事件要设置过期，因为Kafka队列的资源是有限的，不能设置无限保留。当启动Flink的Job，那些过期的事件不会进行计算。因为缺失了数据，一些复杂的事件要么不会生成，要么会出错。例如，缺失 PARCEL_SHIPPED事件的结果就是生成THRESHOLD_EXCEEDED事件，而不是ALL_PARCELS_SHIPPED。

> We cannot have an unlimited retention time in our Kafka queue due to finite resources, so events expire. When we start our Flink jobs, the computation will not take into account those expired events. Some complex events will either not be generated or will be incorrect because of the missing data. For instance, missing PARCEL_SHIPPED events will result in the generation of a THRESHOLD_EXCEEDED event, instead of an ALL_PARCELS_SHIPPED event.



真实数据大且凌乱。先用简单的数据开始测试

> **Real data is big and messy. Test with sample data first**



起初，使用真实数据测试Flink Job并对其业务逻辑进行推理。结果发现，对于调试触发器的逻辑并不方便，而且效率低下。一些事件会丢失，或者事件的属性值是错误的。这使得第一次迭代的推理变得不必要的困难。不久之后，我们实现了一个定制源函数，模拟真实事件的行为，并研究生成的复杂事件。

> At the beginning, we used real data to test our Flink job and reason about its logic. We found its use inconvenient and inefficient for debugging the logic of our triggers. Some events were missing or their properties were incorrect. This made reasoning unnecessarily difficult for the first iterations. Soon after, we implemented a custom source function, simulated the behaviour of real events, and investigated the generated complex events.



### 有时，数据太大反而不好处理

> **Data is sometimes too big for reprocessing**



复杂事件的丢失促使我们需要通过重新处理整个Kafka输入主题来再次生成它们，这对于我们来说是30天的事件。**事实证明，这种再处理对我们来说是不可行的。** 因为CEG的触发条件需要有序的事件，而且因为事件是以不同的速率读取的，所以我们的内存消耗随着我们想要处理的事件的时间间隔而增长。事件在窗口的状态中堆积，等待水印进程，以便触发器触发并清除它们。

> The loss of the complex events prompts the need to generate them again by reprocessing the whole Kafka input topics, which for us hold 30 days of events. This reprocessing proved to be unfeasible for us. Because the firing condition for CEG needs ordered events, and because events are read at different rates, our memory consumption grows with the time interval of events we want to process. Events pile up in the windows’ state and await the watermark progression so that the trigger fires and purges them.



我们使用AWS EC2 t2。中等实例在我们的测试集群中，分配1GB的RAM。我们观察到，我们最多可以在2天内重新处理，而不会因为OutOfMemory异常而导致TaskManager崩溃。因此，我们对早期事件实现了额外的过滤。

> We used AWS EC2 t2.medium instances in our test cluster with 1GB of allocated RAM. We observed that we can reprocess, at most, 2 days worth without having TaskManager crashes due to OutOfMemory exceptions. Therefore, we implemented additional filtering on earlier events.





### 结论

> ### Conclusion



上面我们向您展示了如何设计和实现复杂事件ALL_PARCELS_SHIPPED和THRESHOLD_EXCEEDED。我们已经展示了如何使用Flink的事件时间处理能力实时生成这些数据。我们还介绍了我们在此过程中遇到的挑战，并描述了我们如何使用Flink强大的事件时间处理功能(如水印、事件时间窗口和自定义触发器)来应对这些挑战。

> Above we have shown you how we designed and implemented the complex events ALL_PARCELS_SHIPPED and THRESHOLD_EXCEEDED. We have shown how we generate these in real-time using Flink’s event time processing capabilities. We have also presented the challenges we’ve encountered along the way and have described how we met those using Flink’s powerful event time processing features, i.e. watermark, event time windows and custom triggers.



高级读者将知道[CEP库](https://ci.apache.org/projects/flink/flink-docs-release-1.3/dev/libs/cep.html) Flink提供。当我们开始使用我们的用例(Flink 1.1)时，我们发现这些用例不容易实现。我们相信，在迭代地改进我们的模式时，对触发器的完全控制给了我们更多的灵活性。同时，CEP库已经成熟，在即将到来的Flink 1.4中，它还将支持[CEP模式中的动态状态更改](https://issues.apache.org/jira/browse/FLINK-6418)。这将使类似于我们的用例的实现更加方便。

> Advanced readers will be aware of the [CEP library](https://ci.apache.org/projects/flink/flink-docs-release-1.3/dev/libs/cep.html) Flink offers. When we started with our use cases (Flink 1.1) we determined that these cannot be easily implemented with it. We believed that full control of the triggers gave us more flexibility when refining our patterns iteratively. In the meantime, the CEP library has matured and in the upcoming Flink 1.4 it will also support [dynamic state changes in CEP patterns](https://issues.apache.org/jira/browse/FLINK-6418). This will make implementations of use cases similar to ours more convenient.