# tomcat配置一览

## tomcat目录

- bin —— 存放关闭和启动tomcat脚本文件。在Unix系统中是`*.sh`文件，在Windows系统中是`*.bat`文件。
- conf  —— 存放配置文件。最重要的文件是`server.xml`
- lib —— toncat服务器所需jar包
- logs —— 默认的日志文件
- temp —— 存放tomcat运行时产生的临时目录
- webapps —— 存放web应用程序
- work —— 已部署web应用程序的临时工作目录

## `server.xml`

![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/2019/09/20190926174537.jpg)

### `<Server>`

`<Server>`表示整个Servlet容器（Catalina），必须是`server.xml`文件中的最外层的元素，并且有且仅有一个。

### `<Service>`

`<Service>`中有一或多个`<Connector>`集合。这些`<Connector>`共享一个`<Engine>`来处理请求。一个`<Server>`下可以有多个`<Service>`元素。

### `<Connector>`

#### HTTP/1.1

**HTTP Connector**表示`<Connector>`支持HTTP/1.1协议。能够执行servlet和JSP页面，也能够让Catalina充当独立的WEB服务器。一个`<Connector>`监听一个特定的TCP端口号。一个`<Service>`下可以有多个`<Connector>`。每个`<Connector>`都会转发到对应的`<Engine>`来处理请求并创建响应。

每个请求在请求期间都需要一个线程。如果接受到的并发请求大于线程可以处理的数量，则将创建其他线程，直到`maxThreads`最大值。如果接收到更多并发请求，它们将堆积在`<Connector>`创建的server socket内，直到`acceptCount`最大值。如果还有并发请求，将会收到“连接被拒绝”的错误，直到有足够的资源处理它们。

|    属性    |                             描述                             |
| :--------: | :----------------------------------------------------------: |
|    port    |                          TCP端口号                           |
|  protocol  | 设置传输协议，默认为HTTP/1.1；可以使用：`Http11NioProtocol`、`Http11Nio2Protocol`、`Http11AprProtocol` |
|   scheme   |      协议名称。默认为`http`；可以使用：`https`、`http`       |
| SSLEnabled | 是否启用SSL通信，默认为`false`；可以使用：`true`、`false`；如果您设置了`true`，那么还需要设置`scheme`和`secure`属性 |

#### HTTP/2

**HTTP Upgrade Protocol**表示支持HTTP/2协议的升级协议组件。

#### AJP

省略

### `<Engine>`

`<Engine>`表示与特定`<Service>`关联的整个请求处理机制。`<Engine>`接受并处理来自多个`<Connector>`的请求，并返回给`<Connector>`完成的响应，以便最终将响应传输给客户端。

在`<Service>`下的所有相应`<Connector>`之后，必须内嵌套一个`<Engine>`。

## 参照

- [tomcat9官方指南](https://tomcat.apache.org/tomcat-9.0-doc)
- [tomcat外传](https://zhuanlan.zhihu.com/p/54121733)