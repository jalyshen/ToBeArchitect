# gRPC详解

原文：https://www.jianshu.com/p/9c947d98e192



### 1. gRPC是什么？

gRPC是什么可以使用官网的一句话来概括：

> A high performance, open-source universal RPC framework

所谓RPC框架实际是提供了一套机制，使得应用程序之间可以进行通信，而且也遵从 server / client 模型。使用的时候，客户端调用 server 端提供的接口就像是调用本地的函数一样。如下图所示，就是一个典型的 RPC 结构：

![1](./images/gRPC/1.png)

### 2. gRPC 有什么好处，以及在什么场景下需要用 gRPC

既然是 server / client 模型，那么直接用 restful API 不是也可以满足吗？为什么需要RPC呢？下面就来看看 RPC 到底有什么优势。

#### 2.1 gRCP vs RESTful API

gRPC 和 RESTful API 都是提供了一套通信机制，用于 server / client 模型通信，而且它们都是用 **HTTP** 作为底层的传输协议（严格的说，gRCP使用的是 http2.0， 而RESTful API 则不一定）。不过 gRPC 还是有些特有的优势：

* gRPC 可以通过 protobuf 来定义接口，从而可以有更加严格的接口约束条件
* 通过 protobuf 可以将数据序列化为二进制编码，可以大幅减少传输的数据，提供性能
* gRPC 可以方便地支持流式通信（理论上通过HTT2.0就可以使用 Streaming 模式，但是通常web服务器的 RESTful API 很少这么用，通常的流式数据应用，如视频流，一般都会使用专门的协议，如 HLS，RTMP等，这些就不是我们通常 web 服务了，而是专门的服务器应用）

#### 2.2 使用场景

* **需要对接口进行严格约束的情况**。比如提供一个公共服务，如果访问的客户有公司外部的客户，这时对于接口就要严格的约束，不希望客户端传入任意的数据，尤其是考虑到安全性的因素。这时使用 gRPC iu可以通过 protobuf 来提供严格的接口约束
* **对性能有更高要求时**。