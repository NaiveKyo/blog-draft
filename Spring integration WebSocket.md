## WebSocket

### Reference

- [JSR 356 Java<sup>TM</sup> API for WebSocket](https://jcp.org/en/jsr/detail?id=356)

- https://www.oracle.com/technical-resources/articles/java/jsr356.html

- https://stackoverflow.com/questions/26452903/javax-websocket-client-simple-example

- http://www.programmingforliving.com/2013/08/jsr-356-java-api-for-websocket-client-api.html

对于大多数 web 拥有程序而言，基于 HTTP 的请求-响应模型有一定的局限性，信息只能通过一次次请求进行传递，无法持续的传输信息。

过去采用了多种方式来解决这个问题，比如长轮询和 Comet 技术（HTTP 长连接的 "服务器推"），但是它们还是有一定的缺陷。

在 2011 年，IETF 提出了一份 RFC 6455 将客户端和服务端的全双工双向通信定义为 WebSocket 协议，从此以后大部分的浏览器都会提供支持 WebSocket 协议的客户端 API。此外，也有一批基于 Java 语言的支持 WebSocket 协议的库被开发。

WebSocket 协议通过 HTTP upgrade 技术可以将一个 HTTP 连接升级为 WebSocket 长连接。一旦进行了升级，客户端和服务端就可以彼此独立的向对方传输数据（全双工双向通信），且不需要请求头和 cookie，这就大大降低了网络传输所需的带宽。通常来讲 WebSocket 会用来周期性的传输一些体积较小的信息（比如几个字节的数据）。

### JSR 356

RFC 6455 提出了标准化的 WebSocket 协议，而在 Java 中 JSR 356 则定义了 WebSocket 的 Java API 规范，如果开发者想要在应用程序中集成 WebSocket （不管是在 Java 客户端还是在服务端），只需要实现对应的 api 就可以了。

> Programming Model

定义 JSR 356 的专家组希望能够结合 Java EE 开发者们常用的模式和技术，因此 JSR 356 支持注解和注入（Annotations and Injection），通常来讲，它支持两种开发模式：

- Annotation-driven：只需要对应的注解标注在 POJO 上，就可以使用 WebSocket 的生命周期事件；
- Interface-driven：开发者可以实现 `Endpoint` 接口，实现该接口中定义的方法，就可以和 WebSocket 的 lifecycle events 交互了。

> Lifecycle events

WebSocket 中常用的一些生命周期事件：

- 客户端可以通过发起 HTTP handshake request 来建立 connection；
- 服务端对 handshake request 做出 response；
- 双方建立 connection，从现在开始连接是完全对称的；
- 客户端和服务端可以同时发送和接受消息；
- 直到其中一方关闭 connection。

不管是基于注解的还是基于接口的编程模式，大多数 WebSocket lifecycle events 都会和一个 Java method 关联。