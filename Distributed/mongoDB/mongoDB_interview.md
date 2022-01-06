# MongoDB 面试题

原文：https://www.cnblogs.com/angle6-liu/p/10791875.html



## 1. 什么是MongoDB？

MongoDB是一个文档数据库，提供好的性能，领先的非关系型数据库。采用BSON存储文档数据。BSON 是一种类JSON的二进制形式的存储格式，简称 Binary JSON，相较于 json 多了 date 类型和二进制数组。

## 2. MongoDB的优势

* 面向文档的存储：以JSON格式的文档存储数据
* 任何属性都可以简历索引
* 复制以及高可扩展性
* 自动分片
* 丰富的查询功能
* 快速的即时更新

## 3. 什么是集合（表）

集合，就是一组 MongoDB 文档。它相当于 RDBMS 中的表的概念。集合位于单独的一个数据库中，一个集合内的多个文档**可以有不同的字段**。一般来说，集合中的文档都有着相同或相关的目的。



## 4. 什么是文档（记录）

文档，由一组 Key - Value 组成。文档是动态模式，这意味着同一个集合里的文档不需要有相同的字段和结构。在关系型数据库中，table中的每一条记录相当于MongoDB中的一个文档。

## 5 MongoDB与RMDB的关系

| MongoDB            | 关系型数据库 |
| ------------------ | ------------ |
| Database           | Database     |
| Collection         | Table        |
| Document           | Record / Row |
| Field              | Column       |
| Embedded Documents | Table Join   |

## 6. 使用MongoDB的理由

* 架构简单
* 没有复杂的连接
* 深度查询能力，MongoDB 支持动态查询
* 容易调试
* 容易扩展
* 不需要转换/映射应用对象到数据库的对象（表）
* 使用内存作为存储工作区，以便更快的存取数据

## 7. MongoDB中的命名空间是什么意思?

MongoDB 存储 BSON 对象在丛集（collection）中，**数据库名字**和**丛集名字**以据点连接起来，叫做名字空间（namescpace）

一个集合命名空间又有多个数据域（extent），集合命名空间里存储着集合的元数据，比如集合名称、集合的第一个数据域和最后一个数据域的位置等等。而一个数据域由若千条文档（document）组成，每个数据域都有一个头部，记录着第一条文档和最有一条文档的位置，以及该数据域的一些元数据。extent 之间，document 之间通过双向链表连接。

索引的存储数据结构就是B树，索引命名空间存储着对B树的根节点的指针。

## 8. MongoDB 支持主键外键关系吗？

默认的MongoDB是**不支持**主键和外键关系的。用 MongoDB 本身的API 需要硬编码才能实现外键关联，不够直观且难度较大。

## 9. 支持的数据类型

* String
* Integer
* Double
* Boolean
* Object
* Object ID：存储文档ID。分成4部分：时间戳、客户端ID、客户进程ID、三个字节的增量计数器
* Arrays
* Min/ Max Keys
* Datetime
* Code：存储JavaScript 代码
* Regular Expression：文档中存储正则表达式

## 10 MongoDB中如何实现聚合？

聚合操作能够处理数据记录并返回计算结果。聚合操作能够将多个文档中的值组合起来，对成祖数据执行各种操作，返回单一的结果。它相当于SQL中的count(*) 组合group by。对于 MongoDB 中的聚合操作，应用使用 aggregate() 方法：

db.collection_name.aggregate(AGGREGAT_OPERATION)



## 11 MongoDB支持存储过程吗？

MongoDB支持，使用 JavaScript 写，保存在 db.system.js表中



### 12 如何理解MongoDB的GridFS机制，MongoDB为何使用GridFS来存储文件？

GridFS 是一种将大型文件存储在 MongoDB 中的文件规范。使用 GridFS 可以将大文件分割成多个小文档存放，这样能够有效的保存大文档，而且解决BSON对象有限制的问题。

## 13 能否添加null？

对于对象而言，是可以的。

然而用户不能添加空值（null）到collection，因为空值不是对象。

然而，用户可以添加空对象{}

## 14 更新操作立刻fysnc到磁盘？

不会。磁盘写操作默认是延迟操作，写操作可能在两三秒（默认是60秒）后到达磁盘。例如：如果一秒内数据库收到一千个对一个对象递增的操作，仅刷新磁盘一次。

## 15  MongoDB是否支持事物？

MongoDB 4.0的新特性——事务（Transactions）：MongoDB 是不支持事务的，因此开发者在需要用到事务的时候，不得不借用其他工具，在业务代码层面去弥补数据库的不足。

事务和会话(Sessions)关联，一个会话同一时刻只能开启一个事务操作，当一个会话断开，这个会话中的事务也会结束。