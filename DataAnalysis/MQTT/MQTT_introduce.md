## MQTT 介绍 ##

原文：https://zhuanlan.zhihu.com/p/421109780

这是一篇MQTT入门的文章。内容如下：

* mqtt协议
* mqtt协议特点
  * 发布订阅
  * QoS（Quality of Service Levels）
* mqtt数据包结构
  * MQTT固定头
  * MQTT可变头/Variable header
  * Payload消息体
* 环境搭建
  * MQTT服务器搭建
  * MQTT Client
* 总结

### MQTT 协议 ###

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于**发布/订阅**（publish/subscribe）模式的“轻量级”通讯协议，该协议构建于TCP/IP协议之上，由IBM在1999年发布。

MQTT最大的优点在于，**用极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务**。

作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动设备等方面有较广泛的应用。

### MQTT协议特点 ###

MQTT是一个基于**客户端-服务器**的消息发布/订阅传输协议。

MQTT协议是轻量、简单、开放和易于实现的，这些特点使它适用范围非常广泛。在很多情况下，包括受限的环境中，如：机器与机器（M2M）通信和物联网（IoT）。

其在，通过微信链路通讯传感器、偶尔拨号的医疗设备、只能加剧、及一些小型化设备中已广泛使用。

MQTT协议当前版本是2014年发布的MQTT v3.1.1。除标准版外，还有一个简化版MQTT-SN，该协议只要针对嵌入式设备，这些设备一般工作在TCP/IP网络，如：ZigBee。

MQTT和HTTP一样，运行在传输协议/互联网协议（TCP/IP）堆栈之上。

#### 发布和订阅 ####

MQTT使用的发布/订阅消息模式，它提供了**一对多**的消息分发机制，从而实现与应用程序的解耦。

这是一种消息传递模式，消息不是直接从发送器发送到接收器（点对点），而是由MQTT Server（或者叫MQTT Broker）分发的。

#### QoS（Quality of Service Levels