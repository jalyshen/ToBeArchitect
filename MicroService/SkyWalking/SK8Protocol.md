# 详解 SkyWalking 跨进程传播协议

原文：https://www.toutiao.com/a7002874064033530404/?log_from=08c8eb8ae53a8_1630907360616



​        SkyWalking 跨进程传播协议用于上下文的传播，当前介绍3.0版本，称为 sw8 协议。



### Header项

​        Header 应该是上下文传播的最低要求

* Header 名称：sw8
* Header值：由 - 分隔的8个字段组成。Header值的**长度应该小于 2KB**

### Header值

​        Header值中，包含以下8个字段：

* Sample（采样）：0 或者 1。
  * 0 表示上下文存在，但是可以（也很可能）被忽略
  * 1 表示这个追踪需要采样并发送到后端
* TraceId (追踪ID)：是 Base64 编码的字符串，其内容是：分隔的三个long 类型值，表示此追踪的唯一标识
* Parent trace segment Id（父追踪片段ID）：是 Base64 编码的字符串，其内容是字符串且全局唯一
* Parent Span Id（父跨度ID）：是一个从 0 开始的整数，这个跨度ID指向父追踪片段（segment）中的父跨度（span）
* Parent service（父服务名称）：是 Base64 编码的字符串，其内容是一个长度小于或等于 50  个 UTF-8 编码的字符串
* Parent Service Instance（父服务实例标识）：是 Base64 编码的字符串，其内容是一个长度小于或等于 50 个UTF-8编码的字符串
* Parent Endpoint（父服务的端点）：是 Base64 编码的字符串，其内容是父追踪片段（segment）中第一个入口跨度（span）的操作名，其长度小于或等于 50 个 UTF-8 编码的字符组成
* Peer（本地请求的目标地址）：是 Base64 编码的字符串，其内容是客户端用于访问目标服务的网络地址（不一定是 *IP + 端口*）

### Header值示例

​        上面很抽象，现在举例说明，能够更好的理解。

​        有两个服务，分别叫做 **onemore-a** 和 **onemore-b**，用户通过 HTTP 调用 onemore-a 的 /onemore-a/get，然后 onemore-a 的 /onemore-a/get 又通过 HTTP 调用 onemore-b 的 /onemore-b/get，流程图如下：

![1](./images/SK8Protocol/1.jpg)

​        那么，在 onemore-b 的 /onemore-b/get 的 Header 中就可以发现一个叫做 **sw8** 的key，其值是：

```shell
1-YTRlYzZmYzhjY2FiNGJiNGI2ODIwNjQ2OThjYzk3ZTYuNzQuMTYyMTgzODExMDQ1NTAwMDk=-YTRlYzZmYzhjY2FiNGJiNGI2ODIwNjQ2OThjYzk3ZTYuNzQuMTYyMTgzODExMDQ1NTAwMDg=-2-b25lbW9yZS1h-ZTFkMmZiYjYzYmJhNDMwNDk5YWY4OTVjMDQwZTMyZmVAMTkyLjE2OC4xLjEwMQ==-L29uZW1vcmUtYS9nZXQ=-MTkyLjE2OC4xLjEwMjo4MA==
```

​        以字符 - 进行分割，可以得到：

* 1：采样，标识这个追踪需要采样并发送到后端
* YTRlYzZmYzhjY2FiNGJiNGI2ODIwNjQ2OThjYzk3ZTYuNzQuMTYyMTgzODExMDQ1NTAwMDk=：追踪ID，解码后是：a4ec6fc8ccab4bb4b682064698cc97e6.74.16218381104550009
* YTRlYzZmYzhjY2FiNGJiNGI2ODIwNjQ2OThjYzk3ZTYuNzQuMTYyMTgzODExMDQ1NTAwMDg=：父追踪片段ID，解码后是：a4ec6fc8ccab4bb4b682064698cc97e6.74.16218381104550009
* 2：父跨度ID
* b25lbW9yZS1h：父服务名，解码后为：onemore-a
* ZTFkMmZiYjYzYmJhNDMwNDk5YWY4OTVjMDQwZTMyZmVAMTkyLjE2OC4xLjEwMQ==：父服务实例标识，解码后为：e1d2fbb63bba430499af895c040e32fe@192.168.1.101
* L29uZW1vcmUtYS9nZXQ=：父服务的端点，解码后：/onemore-a/get
* MTkyLjE2OC4xLjEwMjo4MA==：本请求的目标地址，解码后为：192.168.1.102:80



### 扩展Header项

​        扩展Header项是为了高级特性设计的，它提供了部署在上游和下游服务中的探针之间的交互功能。

* Header名称：sw8-x
* Header值：由 -  分隔，字段可扩展

### 扩展Header值

​         当前值包括的字段：

* Tracing Mode（追踪模式），空、0，或者 1。默认是“空”或者“0”。
  * 这个值表示在这个上下文中生成的所有跨度（span）应该跳过分析
  * 默认情况下，这个应该在上下文中传播到服务端，除非它在跟踪过程中被更改