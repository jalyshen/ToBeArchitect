# HTTP 基本原理

原文：https://www.toutiao.com/i7053301582976451080/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1642231403&app=news_article&utm_source=weixin&utm_medium=toutiao_ios&use_new_style=1&req_id=2022011515232301013819015601A19DCC&share_token=A22DB1BD-8F81-49A2-92FB-9F9694E030C2&group_id=7053301582976451080



HTTP 协议是 Hyper Text Transfer Protocol（超文本传输协议）的缩写，是用于万维网（WWW，World Wide Web）服务器传输超文本到本地浏览器的传送协议。HTTP 是一个基于 TCP/IP 通信协议传递数据（HTML文本，图片文件，查询结果等）。

HTTP 是基于客户/服务器模式，且面向连接的。典型的HTTP事务处理有如下的过程：

1. 客户与服务器建立连接
2. 客户向服务器提出请求
3. 服务器接受请求没，并根据请求返回相应的文件作为应答
4. 客户与服务器关闭连接

HTTP 三点注意事项：

1. **HTTP 是无连接的**：无连接的含义，是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间
2. **HTTP 是媒体独立的**：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的 MIME-Type 内容类型
3. **HTTP 是无状态的**：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快

## HTTP请求

HTTP请求可以分成四个部分：

* 请求的网址（Request URL）
* 请求的方式（Request Method）
* 请求头（Request Headers）
* 请求体（Request Body）

### 请求的网址

请求的网址即URL，可以唯一确定请求的资源

### 请求的方式

常用的请求方式有：GET、POST。 还有 PUT，DELETE等

* **GET**：求指定网页信息，并返回网页内容，提交的数据最多只有 1024 字节
* **POST**：向指定资源提交数据并进行请求处理，数据都包含在请求体中，提交的数据没有字节限制

### 请求头

* Accept：指定客户端可识别的内容类型
* Accept-Encoding：指定客户端可识别的内容编码
* Accept- Language：指定客户端可识别的语言类型
* Cookie：王占位了辨别用户身份进行会话跟踪而存储在用户本地的数据，主要功能是维持当前会话
* Host：指定请求的服务器域名和端口号
* User-Agent：使服务器识别客户端使用的操作系统即版本、浏览器以及版本信息，实现爬虫时加上可伪装成浏览器
* Content- Type：请求的媒体类型信息（确定了POST请求提交数据的方式）
* Referrer：包含一个URL，用户以该URL代表的页面出发访问当前请求页面

#### Content-Type 和 POST 提交数据方式的关系

| Content-Type                      | 提交数据方式   |
| --------------------------------- | -------------- |
| application/x-www-form-urlencoded | 表单数据       |
| multipart/form-data               | 表单文件       |
| application/json                  | 序列化JSON数据 |
| text/xml                          | XML数据        |

### 请求体

请求体中的内容一般是POST请求的表单数据。而 GET 请求为空。

## HTTP 响应

http 响应分为三部分内容：

* 响应状态码（Response Status Code）
* 响应头（Response Headers）
* 响应体（Response Body）

### 响应状态码

常用的响应状态码有如下这些：

| **状态码** | **英文名称**          | **说明**                                                     |
| ---------- | --------------------- | ------------------------------------------------------------ |
| 100        | Continue              | 服务器已经收到请求的一部分，正在等待其余部分，应继续提出请求 |
| 200        | OK                    | 服务器已经成功处理了请求                                     |
| 302        | Move Temporarily      | 服务器要求客户端重新发送一个请求                             |
| 304        | Not Modified          | 此请求返回的网页未修改，继续使用上次的资源                   |
| 404        | Not Found             | 服务器找不到请求的网页                                       |
| 500        | Internal Server Error | 服务器遇见错误，无法完成请求                                 |

### 响应头

| **响应头**       | **说明**                  |
| ---------------- | ------------------------- |
| Content-Encoding | WEB服务器支持的编码类型   |
| Content-Language | 相应体的语言              |
| Content-Length   | 响应体的长度              |
| Content-Type     | 返回内容的媒体类型        |
| Date             | 原始服务器消息发出的时间  |
| Expires          | 响应过期的日期和时间      |
| Last-Modified    | 请求资源的最后修改时间    |
| Set-Cookie       | 设置HTTP Cookie           |
| Location         | 重定向接收到请求的URL位置 |

### 响应体

响应体包含响应的正文数据。