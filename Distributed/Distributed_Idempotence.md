# 深入理解幂等性

https://www.cnblogs.com/javalyy/p/8882144.html

## 什么是幂等性

HTTP/1.1 中对幂等性的**定义**是：一次和多次请求某**一个**资源**对于资源本身**应该具有相同的结果（网络超时等问题除外）。换句话说，**其任意多次执行对资源本身所产生的影响均与一次执行的影响相同**。

*英文原文：Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.*

仔细研读，需要注意如下几点：

* 幂等不仅仅只是一次（或多次）请求对资源没有副作用（比如查询数据库操作，没有增删改，因此没有对数据有任何影响）。
* 幂等还包括第一次请求的时候对资源产生了副作用，但是以后的多次请求都不会再对资源产生副作用。
* 幂等关注的是以后的多次请求是否对资源产生的副作用，而不关注结果。
* 网络超时等问题，不是幂等的讨论范围。

幂等性是系统服务对外一种承诺（而不是实现），承诺只要调用接口成功，外部多次调用对系统的影响是一致的。声明为幂等的服务会认为外部调用失败是常态，并且失败之后必然会有重试。

## 什么情况下需要幂等

业务开发中，经常会遇到重复提交的情况，无论是由于网络问题无法收到请求结果而重新发起请求，或是前端的操作抖动而造成重复提交情况。**在交易系统、支付系统这种重复提交造成问题尤其明显**，比如：

* 用户在App上连续点击了多次提交订单，后台应该只产生一个订单；
* 向支付系统发起支付请求，由于网络问题或者系统问题，出现了重发，支付系统只扣一次款。

**很显然，声明幂等的服务认为：外部调用者会存在多次调用的情况，为了防止外部多次调用对系统数据状态的发生多次改变，将服务设计成幂等。**

## 幂等与防重

重复提交也有类似问题。实际上，**重复提交与服务幂等的初衷是不一样的**。

重复提交是在第一次请求已经成功的情况下，**人为的进行多次操作**，导致不满足幂等要求的服务多次改变状态。

而幂等更多使用的情况是**第一次请求不知道结果**（比如超时）或者失败的异常情况下，发起多次请求，目的是多次确认第一次请求成功。

## 什么情况下需要保证幂等

以SQL为例子，存在以下三种场景，只有第三种场景需要幂等：

* SELECT col1 FROM table1 WHERE col2 = 2, 无论执行多少次都不会改变状态，不需要额外考虑幂等
* UPDATE table1 SET col1 = 1 WHERE col2 = 2, 无所执行多少次（成功），结果都是一致的，不需要额外考虑幂等
* UPDATE table1SET col=col+1 WHERE col2 = 2, 每次执行，结果都会变化，就需要考虑幂等

## 为什么要设计幂等的服务

幂等可以使得客户端逻辑处理变得简单，却额外给服务器端逻辑增加的复杂代价。满足幂等服务的需要在逻辑中至少包含两点：

* 首先去查询上一次的执行状态，如果没有任何记录，则认为是第一次请求
* 在服务改变状态的业务逻辑前，保证防重复提交的逻辑

## 幂等的不足

幂等是为了简化客户端逻辑处理，却增加了服务提供者的逻辑和成本，是否必要，需要根据具体场景具体分析。因此除了业务上的特殊要求外，尽量不提供幂等的接口。

* 增加了额外控制幂等业务逻辑，复杂化了业务功能
* 把并行执行的功能改成串行化，降低了执行效率

## 保证幂等策略

幂等需要通过**唯一标识**（*在分布式系统中，通常采用TraceId、SessionId或者Token来标识请求的唯一性*）来保证。也就是说，相同的标识号，认为是同一个业务操作。使用一个唯一标识号，可以确保后面多次的相同的业务处理逻辑和执行效果是一致的。

以支付为例，在不考虑并发的情况下，实现幂等很简单：

1. 先查询一下订单是否已经支付
2. 如果已经支付过，返回成功；如果没有，进入支付流程，修改订单状态

## 防重提交策略

“幂等策略”是分两步实施的，而且第二步严重依赖第一步，无法保证操作的**原子性**。在高并发的情况下容易出现问题：第二次请求在第一次请求的第二步还未执行（即还未修改订单状态）前到来。

针对这种情况，有以下的策略来避免：

### 乐观锁

如果只是更新**已有**的数据，没有必要对业务进行加锁，设计表结构时使用乐观锁，一般通过Version字段来做乐观锁，这样既能保证执行效率，又能保证幂等。

但是，乐观锁存在ABA问题。**解决ABA问题，就要保证version字段一直增长**。

### 防重表

新建一张表，存储各个要保证幂定业务的请求唯一标识，并把这个**请求唯一标识**作为防重表的**唯一索引**。

每接收一个请求，先查询防重表中是否存储当前的唯一标识，如果没有，就把唯一标识插入到防重表，然后执行相应的业务，无论操作成功还是失败，都要在**业务表**中更新此次之行的状态为”成功“或者”失败“，然后，再删除防重表中的这条记录。

### 分布式锁

上面的**防重表**也可以使用Redis，于是就成了一个真正意义上的分布式锁。具体的实施逻辑与**防重表**一致。不再赘述。

### Token令牌

使用“支付”业务举例。

这种方式分成2个阶段：申请Token令牌阶段和支付阶段。

第一阶段，**申请Token令牌阶段**。在进入到提交订单页面前，需要订单系统根据用户信息向支付系统发起一次申请Token的请求，支付系统将Token保存到Redis中，为第二阶段支付使用。

第二阶段，**支付阶段**。订单系统拿着申请到的Token发起支付请求，支付系统会检查Redis中是否存在此Token。如果**存在**，表明第一次发起支付请求，删除缓存中的Token，然后开启支付处理流程；如果**不存在**，标识非法请求。

Token令牌就是**分布式锁**的一种具体实现方式。

### 缓冲区

依旧使用支付系统为例子。

把订单的支付请求都快速的接下来，一个快速接单的缓冲管道。后续使用异步任务处理管道中的支付请求，过滤掉重复的待支付订单。优点是同步转异步，实现高吞吐；不足时不能及时返回给客户支付结果，需要通过监听来获取异步处理的结果。

## RESTful API 幂等性

https://www.jianshu.com/p/9d46a730284e

大家最经常写的代码就是restful API了，承载API的是HTTP协议。下面分析一下HTTP协议的各个方法的幂等性。

### HTTP协议各个方法的幂等分析

##### HTTP GET

Get方法，用于获取资源，不管调用多少次接口，结果都不会改变，所以是**幂等的**。

只是查询数据，不会影响到资源（状态）的变化。

需要注意的是，幂等性指的是**作用于结果，而非资源本身**。如何理解这句话？就是说，Get方法多次调用同一个接口（每次传入的参数一致），每次返回的内容可能会不一样，但是**对资源本身没有影响**。

举个例子：获取服务器当前的时间，每次调用这个Get方法时，传入的参数是一样的（比如无参数），但是每次调用返回的结果（当前服务器的时间）却是不一样的。但是调用时，Get方法并不会对“时间”这个资源产生影响。因此，也是幂等的。

##### HTTP POST

POST方法通常用来修改资源的，因此，该方法一定是一个**非幂等的**。

##### HTTP PUT

PUT方法通常用于**全量修改一个资源的信息**，如果多次调用同一个请求（要修改的内容不会变）成功的情况下，实际上对资源变化只会影响一次，而且结果也是相同的。所以，PUT方式是**幂等的**。（*这个结果貌似超出了预想 ;)*）

##### HTTP PATCH

PATCH方法通常用于更新资源的**部分**信息，这个方法与PUT相似。但是，PATCH 提供的实体则需要根据程序或其它协议的定义，解析后在服务器上执行，以此来修改服务器上的资源。换句话说，PATCH 请求是会执行某个程序的，如果重复提交，程序可能执行多次，对服务器上的资源就可能造成额外的影响，这就可以解释它为什么是非幂等的了。

举个例子：使用Patch方法更新一个对象的部分字段，系统就会记录这次操作。如果用户多次提交同一个请求，那么其他相关资源可能会多次之行，同时系统中也会有多次操作记录。

##### HTTP DELETE

DELETE通常用来删除资源。同一次请求调用一次或者多次，对资源的影响是一样的，此方法是一个**幂等的**。

### 如何设计符合幂等的高质量的Restful API

通过上面对HTTP每个方法的分析，有助于我们写出高质量的幂等的API。

##### GET vs POST

通常，GET是幂等的，POST是非幂等的。因此，GET适合查询，POST适合**新增**资源。

##### POST vs PUT

POST通常用于新增资源，PUT用于**全量更新**资源。

##### PUT vs PATCH

PUT通常是全量更新资源，而PATCH是部分更新资源。

