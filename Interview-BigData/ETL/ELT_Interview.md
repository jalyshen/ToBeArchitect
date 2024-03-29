# ETL 架构师面试题

原文：https://www.cnblogs.com/littlewu/p/12513451.html



## 1. ETL 四个阶段分别是什么？

​        ETL整个过程分为4个阶段：抽取（Extract）、清洗（Clean）、一致性处理（Comform）和交付（Delivery），简称为 ***ECCD***。

* 抽取（Extract）阶段的主要任务
  * 读取源系统的数据模型
  * 连接并访问源系统的数据
  * 变化数据捕获
  * 抽取数据到数据准备区
* 清洗（Clean）阶段的主要任务
  * 清洗并增补列的属性
  * 清洗并增补数据接哦股
  * 清洗并增补数据规则
  * 增补复杂的业务规则
  * 建立元数据库描述数据质量
  * 将清洗后的数据保存到数据准备区
* 一致性处理（Comform）阶段的主要任务
  * 一致性处理业务标签，即维度表中的描述属性
  * 一致性处理业务度量及性能指标，通常是事实表中的事实
  * 去除重复数据
  * 国际化处理
  * 将一致性处理后的数据保存到数据准备区
* 交付（Delivery）阶段的主要任务
  * 加载星型的和经过雪花处理的维度表数据
  * 产生日期维度
  * 加载退化维度
  * 加载子维度 
  * 加载1、2、3型的缓慢变化维度
  * 处理迟到的维度和迟到的事实 
  * 加载多值维度
  * 加载有复杂层级结构的维度
  * 加载文本事实到维度表
  * 处理事实表的代理键
  * 加载三个基本类型的事实表数据
  * 加载和更新聚集
  * 将处理好的数据加载到数据仓库