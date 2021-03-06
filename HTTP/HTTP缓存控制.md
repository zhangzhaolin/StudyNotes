# HTTP 缓存控制

## 01. 缓存过程

1. 浏览器发送请求前，根据请求头的 `expires` 和 `cache-control` 判断是否命中（包含是否过期） <u>强缓存策略</u>，如果命中，则直接从缓存获取资源，并不会发送请求，如果没有命中，则进入下一步。
2. 如果没有命中 <u>强缓存策略</u>，浏览器会发送请求，根据请求头的 `last-modified` 和 `etag` 判断是否命中 <u>协商缓存</u>，如果命中，直接从缓存获取资源。如果没有命中，则进入下一步。
3. 如果前两步都没有命中，则直接从服务器获取资源。

## 02. 强制缓存

强制缓存：不会向服务器发送请求，直接从缓存中读取资源。

在谷歌控制台中可以看到该请求返回 200 状态码，并且显示 `from disk cache` 或 `from memory cache`。强缓存可以通过设置两种 HTTP 头部实现：`Expires`、`Cache-Control`。

### `Expires`

`Expires` 是 HTTP/1.0 控制网页缓存的字段，指定缓存到期的 GMT（格林尼治时间）的绝对时间。

但是：响应报文中的 `Expires` 所定义的时间是相对于服务器上的时间而言的，如果客户端的时间与服务器上的时间不一致，缓存时间就没有意义了。为了解决这一问题，HTTP/1.1 新增了 `Cache-Control` 来定义缓存过期的时间。

### `Cache-Control`

在 HTTP/1.1 中，`Cache-Control` 是最重要的规则，包含了 **缓存策略** 和 **过期策略**。

语法为 ：

```
Cache-Control：cache-directive
```

`cache-directive` 主要有五种常见的取值：

1. `public`

   所有内容都将被缓存（客户端和代理服务器都可以缓存）

2. `private`

   所有内容只有客户端可以缓存，代理服务器不缓存，`Cache-Control` 的默认取值

3. `no-store`

   所有内容都不会缓存，既不是用强制缓存，也不使用协商缓存

4. `no-cache`

   客户端缓存内容，但是是否使用缓存需要通过 **协商缓存** 来验证决定。表示不使用 `Cache-Control` 的缓存控制方式做前置验证，而是使用 `ETag` 或者 `Last-Modified` 字段来控制缓存。

5. `max-age=xxx`

   缓存内容将在 xxx 秒后失效

## 协商缓存

协商缓存就是强制缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存的过程。主要由以下两种情况：

- 协商缓存生效，返回 304 和 `Not Modified`

  ![协商缓存生效](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2020/01/20200119120107.png)

- 协商缓存失效，返回 200 和请求结果

  ![协商缓存失效](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2020/01/20200119141938.png)

协商缓存可以有两种 HTTP 请求头实现 ： `Last-Modified` 和 `ETag`

### `Last-Modified` 和 `If-Modified-Since`

浏览器在第一次访问资源时，服务器返回资源的同时，在响应头部中添加 `Last-Modified` 头部，值表示：这个资源在服务器上的最后修改时间，浏览器接收后缓存文件和头部。

```http
Last-Modified: Fri, 22 Jul 2016 01:47:00 GMT
```

浏览器下一次请求这个资源，浏览器检测到有 `Last-Modified` 这个头部，于是添加 `If-Modified-Since` 这个头部，值就是 `Last-Modified` 值；服务器再次收到这个请求，会根据 `If-Modified-Since` 中的值和服务器中该资源最后修改时间做对比，如果没有变化，则返回 304 和空的响应体，直接从缓存读取；如果服务器该资源修改时间超过了 `If-Modified-Since` 的时间，说明文件有更新，于是返回新的资源文件和 200。

### `Last-Modified` 弊端

- 如果本地打开缓存文件，即使没有对文件进行修改，但是还会造成 `Last-Modified` 被修改，服务器不能命中缓存，导致发送相同的资源
- 因为 `Last-Modified` 只能以秒计时，如果在不可感知的时间内修改完文件，那么服务端会认为资源还是命中了，不会返回正确的（更新后的）资源。

## 参考

- [深入理解浏览器的缓存机制](https://www.jianshu.com/p/54cc04190252)