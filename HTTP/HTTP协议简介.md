[TOC]

# HTTP协议简介

## 01. 引言

<b>超文本传输协议</b>（ HTTP ）是一种用于分布式、协作式和超媒体信息系统的<u>应用层协议</u>。HTTP 是万维网的数据通信基础。

目前，HTTP/2 标准已经正式发表，取代 HTTP 1.1 成为 HTTP 的实现标准。

## 02. 主要特点

- 支持 B/S 和 C/S 模式
- 简单快速：客户向服务器请求服务时，只需传递请求方法和路径。请求方法常用的有 `GET`、`POST`、`HEAD`。每种方法规定了客户和服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因此通信速度很快
- 灵活：HTTP 允许传递任意类型的数据对象。正在传输的类型由 `Content-Type` 标记
- 无连接：每次连接只处理一次请求。服务器处理完客户的请求，并收到客户应答后，即断开连接。采用这种方式可以节省传输时间。
- 无状态：HTTP 协议是无状态协议。无状态协议是指协议对事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

## 03. URL（统一资源定位符）

统一资源定位符（英文缩写：URL ）是因特网上标准的资源地址，如同网络上的门牌。`URL` 标准格式如下：

```
[协议类型]://[服务器地址]:[端口号]/[资源层级UNIX文件路径][文件名]?[查询]#[片段ID]
```

`URL` 完整格式如下：

```
[协议类型]://[访问资源需要的凭证信息]@[服务器地址]:[端口号]/[资源层级UNIX文件路径][文件名]?[查询]#[片段ID]
```

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2019/12/20191231142452.jpg)

## 04. 	URL 和 URI

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2020/01/20200102100304.png)

- URL ：统一资源定位符
- URI  ：统一资源标识符

> URI 是一个紧凑的字符串用来标示抽象或物理资源。

> URI 可以被进一步被分为定位符、名字或两者都是。URL 是 URI 的子集，除了确定一个资源，还提供一种定位该资源的主要访问机制（如其网络“位置”）。

> URI 可以分为 URL 和 URN 或同时具备 locators 和 names 特性的一个东西。URN 作用就像一个人的名字，URL 就像人的一个地址。换句话说：URN 确定了东西的身份，URL 确定了找到它的方式。

例子：

```
ftp://ftp.is.co.za/rfc/rfc1808.txt (also a URL because of the protocol)
http://www.ietf.org/rfc/rfc2396.txt (also a URL because of the protocol)
ldap://[2001:db8::7]/c=GB?objectClass?one (also a URL because of the protocol)
mailto:John.Doe@example.com (also a URL because of the protocol)
news:comp.infosystems.www.servers.unix (also a URL because of the protocol)
tel:+1-816-555-1212
telnet://192.0.2.16:80/ (also a URL because of the protocol)
urn:oasis:names:specification:docbook:dtd:xml:4.1.2
```

这些全都是 URI，当然那些提供了访问机制的也是 URL

## 05. `HTTP` 请求消息

`HTTP` 请求消息有4部分组成：

- 请求行
- 请求头部
- 空行
- 请求数据

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2020/01/20200102104243.jpg)

### 请求行

请求行包括：请求方法字段、URL 字段、HTTP版本协议字段。它们用空格分隔。例如：

```
POST / HTTP 1.1
GET /background.png HTTP/1.0
HEAD /test.html?query=alibaba HTTP/1.1
OPTIONS /anypage.html HTTP/1.0
```

#### HTTP请求方法

- `GET`

  用于请求一个指定资源，使用 `GET ` 的请求应该只被用于获取数据。

- `HEAD`

  `HEAD` 方法和 `GET` 相同，但是服务器响应时不会返回消息体。`HEAD`常常用于：

  - 只是请求资源的头部
  - 检查链接的有效性
  - 检查网页是否被修改
  - 获取网页标志信息，获取 `rss` 种子信息，或传递安全认证信息

- `POST`

  发送数据给服务器，请求主体的类型由 `Content-Type` 指定。与 `PUT`方法的区别是：`PUT` 方法是幂等的。而连续调用同一个 `POST` 可能会带来额外的影响，比如多次提交订单。

  🚀 幂等：连续调用一次和多次的效果相同（无副作用）。

- `PUT`

  `PUT` 方法用于请求中的主体（数据）创建或者替换目标资源。

- `DELETE`

  `DELETE` 请求方法用于删除指定的资源

- `CONNECT`

- `OPTIONS`

- `TRACE`

- `PATCH`

### 请求数据

## 06. `HTTP` 响应消息

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2020/01/20200106103234.png)

`HTTP` 响应消息由四部分组成：

- 响应行
  - 协议版本：通常为 `HTTP/1.1`
  - 状态码
  - 状态文本：一个简短的，纯粹的信息，通过状态码的文本描述，帮助人们理解该 HTTP 信息。
- 响应头
- 空行
- 响应体

## 07. `HTTP` 头部

消息头部不区分大小写，根据不同背景，可以把消息头部分为几类：

- 通用首部消息：可以用于请求和响应中，但是与消息主体中的数据无关

  - `Cache-Control`

    被用于请求头和响应头中，通过指定指令来实现缓存机制。缓存机制是单向的，这意味着在请求中设置的指令，不一定被包含在响应中。

    示例：

    ```
    Cache-Control: no-cache, no-store, must-revalidate
    Cache-Control:public, max-age=31536000
    ```

  - `Connection`

    决定当前的事务完成后，是否关闭网络连接。如果该值是`keep-alive`，网络连接都是持久的，不会关闭，使得对同一服务器的请求可以继续在该连接上完成。

    示例：

    ```
    Connection: keep-alive
    Connection: close
    ```

  - `Transfor-Encoding`

    指明了将<u>实体首部消息</u>传递给用户所采用的编码形式。例如：

    ```
    Transfer-Encoding: chunked
    Transfer-Encoding: compress
    Transfer-Encoding: deflate
    Transfer-Encoding: gzip
    Transfer-Encoding: identity
    ```

- 请求首部消息：客户端向服务器发送请求的报文时所使用的头部

  - `Accept`

    用来告知服务器，客户端可以处理的内容类型。服务器可以从诸多被选项中选择一项进行应用，并使用 `Content-Type` 响应头通知客户端它的选择。

    示例：

    ```
    Accept: text/html
    Accept: image/*
    Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
    ```

  - `Range`

    `Range`  告知服务器返回文件的哪一个部分。在一个 `Range` 中，可以一次性请求多个部分，服务器会以 `multipart`  文件的形式将其返回。若服务器返回的是范围相应，需要使用 206 状态码。如果请求的范围不合法，返回 416 状态码，表示客户端错误。服务器允许忽略 `Range`，从而返回整个文件，状态码用 200。

    示例：

    ```
    Range: bytes=200-1000, 2000-6576, 19000- 
    ```

  - `Authorization`

    包含用户代理身份的凭证，用于服务器的验证。语法：`Authorization: <type> <credentials>`

    示例：

    ```
    Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l
    ```

  - `Host`

    指定了服务器的域名，以及TCP端口号（可选），如果没有给定，则请求服务的默认端口（例如HTTP会使用默认的80端口）

    示例：

    ```
    Host: developer.cdn.mozilla.net
    ```

  - `User-Agent`

    用户浏览器、操作系统相关信息

    示例：

    ```
    User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0
    ```

- 响应首部消息：从服务器向客户端响应时使用的字段

  - `Accept-Ranges`

    如果服务器支持 `Range`，首先需要告知客户端，服务器会在响应头中添加 `Accept-Ranges:bytes` 表示支持 `Range` 请求，之后客户端才能发起带 `Range` 的请求。不支持的话，使用 `Accept-Ranges` 表示。

  - `Content-Type`

    用于指定资源的类型。例如：

    ```
    Content-Type: text/html; charset=utf-8
    Content-Type: multipart/form-data; boundary=something
    ```

  - `Location`

    表示需要将页面重新定向到指定的地址。一般在响应码为 3XX 的响应中才会有意义。例如：

    ```
    Location: /index.html
    ```

  - `ETag`

    表示资源特定版本的标识符。如果给定 URL 中的资源更改，则一定要生成新的 `ETag` 值。因此类似于指纹。例如：

    ```
    ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
    ETag: W/"0815"
    ```

  - `Server`

    表示处理请求的服务器所用到的软件信息：示例：

    ```
    Server: Apache/2.4.1 (Unix)
    ```

- 实体首部消息：含有和消息主体相关的附加信息，例如主体的长度或者 MIME 类型。

  - `Allow`
  
    用于枚举资源所支持的 HTTP 方法的集合。
  
    若服务器端返回状态码 `405`，则 `Allow` 字段也需要同时返回给客户端。如果 `Allow` 字段值为空，表示资源不接受任何 HTTP 方法的请求。
  
    示例：
  
    ```
    Allow: GET, POST, HEAD
    ```
  
  - `Last-Modified`
  
    包含服务器认定的资源做出的修改的日期和时间。通常被用作一个验证器，来判断接收到的或者存储的资源。由于精确度比 `Etag` 低，所以这是一个备用机制。例如：
  
    ```
    Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT 
    ```
  
  - `Expires`
  
    包含日期/时间，在这个时间之后，响应过期。如果在 `Cache-Control` 的响应头设置了 `max-age` 或 `s-max-age`，那么 `Expires` 头会被忽略。
  
    示例：
  
    ```
    Expires: Wed, 21 Oct 2015 07:28:00 GMT
    ```

## 08. `HTTP` 状态码

### 1XX 信息型响应

### 2XX 成功响应

- 200 OK

  请求成功

- 201 Created

  请求成功，并创建了一个新的资源。通常是在 POST 请求，或某些 PUT 请求之后返回的响应。

- 202 Accepted

  表示服务器端已经收到请求消息，但是尚未进行处理。无法确保完成处理，即稍后无法通过 HTTP 协议给客户端发送一个异步请求来告知其请求的处理结果。该请求码经常用于其他进程或服务器处理请求的情况，或者用于批处理。

- 204 No Content

  表示执行成功，但是没有数据，浏览器不需要刷新页面，也不用导向新的页面。

- 205 Reset Content

  表示执行成功，重置页面。

- 206 Partial Content 🎏

  表示请求成功，并且主体包含所请求的数据区间，该数据区间由请求的 `Range` 头部指定。
  
  如果只包含一个数据区间，那么整个响应的 `Content-Type` 首部的值为所请求的文件类型，同时包含 `Content-Range` 头部。
  
  如果包含多个数据区间，那么整个相应的 `Content-Type`  首部的值为 `multipart/byteranges`，其中一个片段对应一个数据区间，并提供 `Content-Range` 和 `Content-Type` 描述信息。
  
  ① 只包含一个数据区间的响应：
  
  ```
  HTTP/1.1 206 Partial Content
  Date: Wed, 15 Nov 2015 06:25:24 GMT
  Last-Modified: Wed, 15 Nov 2015 04:58:08 GMT
  Content-Range: bytes 21010-47021/47022
  Content-Length: 26012
  Content-Type: image/gif
  
  ... 26012 bytes of partial image data ...
  ```
  
  ② 包含多个数据区间的响应：
  
  ```
  HTTP/1.1 206 Partial Content
  Date: Wed, 15 Nov 2015 06:25:24 GMT
  Last-Modified: Wed, 15 Nov 2015 04:58:08 GMT
  Content-Length: 1741
  Content-Type: multipart/byteranges; boundary=String_separator
  
  --String_separator
  Content-Type: application/pdf
  Content-Range: bytes 234-639/8000
  
  ...the first range...
  --String_separator
  Content-Type: application/pdf
  Content-Range: bytes 4590-7999/8000
  
  ...the second range
  --String_separator--
  ```

### 3XX 重定向

- 301 moved permanently

  说明请求的资源已经被移动到了由 `Location` 头部指定的 URL 上，是固定的不会再改变。搜索引擎会根据该响应修正。

- 302 Found

  表明请求的资源暂时移动到了由 `Location` 头部指定的 URL 上。浏览器会重定向到这个 URL，但是搜索引擎不会对该资源的链接进行更新。

  注意：由于历史原因，浏览器可能会在重定向后的请求中把 POST 改为 GET 方法，如果不想这样，请使用 307 状态码。

- 303 See Other

  请求重定向，重定向页面的方法要总是使用 `GET`。

- 304 Not Modified

  说明无需再次传输请求内容，也就是说可以使用缓存的内容。

  当客户端缓存了目标资源但不确定该缓存资源是否是最新版本的时候, 就会发送一个条件请求，这样就可以辨别出一个请求是否是条件请求，在进行条件请求时,客户端会提供给服务器一个 `If-Modified-Since` 请求头,其值为服务器上次返回的 `Last-Modified` 响应头中的 `Date` 日期值,还会提供一个 `If-None-Match` 请求头,值为服务器上次返回的 `ETag` 响应头的值。

  服务器会读取这两个请求头中的值，判断出客户端缓存资源是否是最新的，如果是的话，服务器就会返回 304 响应头，但是没有响应体。客户端收到 304 响应后，就会从本地缓存中读取对应的资源。

- 307 Temporary Redirect

  请求资源被临时的移动到了 `Location` 头部所指向的 URL 上。会在请求中重用原始请求方法和消息主体。

  如果需要将请求的方法转换为 `GET`，可以考虑 303 状态码。

### 4XX 客户端错误

- 400 Bad Request

  表示由于语法无效，服务器无法理解该请求。

- 401 Unauthorized

  由于缺乏目标资源要求的身份验证凭证，发送的请求未得到满足。与 403 不同，客户端仍然可以进行身份验证。

- 403 forbidden

  服务器端有能力处理该请求，但是拒绝授权访问。类似于 401，但是进入该状态后不能再继续验证。该访问是长期禁止的，并且与应用逻辑有密切关系（例如不正确的密码）

- 404 not found

  请求所希望得到的资源未被在服务器上发现。404 不能说明请求的资源是临时还是永久丢失。

- 408 Request timeout

  表示服务器想要将没有在使用的连接关闭。一些服务器会在空闲连接上发送此信息，**即便是客户端没有发送任何请求的情况下。**

  服务器应该在此类响应中将 `Connection` 首部的值设置为 `close`，因为 408 意味着服务器已经决定将连接关闭，而不是继续等待。

- 409 Conflict

  表示请求与服务器端目标资源的当前状态相冲突

### 5XX 服务端错误

- 500 internal sever error

  服务器端错误的响应状态码，意味着所请求的服务器遇到意外的情况并阻止其执行请求

- 501 Not Implemented

  表示请求的方法不被服务器支持，因此无法被处理

- 503 service unavailable

  服务器没有准备好处理请求。常见原因是服务器因维护或重载而停机。

- 505 http version not supported

  表示服务器不支持请求所使用的 HTTP 版本。

## 参考

- [面试官（9）：可能是全网最全的http面试答案](https://juejin.im/post/5d032b77e51d45777a126183#heading-4)

- [MDN-HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

