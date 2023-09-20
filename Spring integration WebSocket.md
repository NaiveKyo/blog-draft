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

#### Programming Model

定义 JSR 356 的专家组希望能够结合 Java EE 开发者们常用的模式和技术，因此 JSR 356 支持注解和注入（Annotations and Injection），通常来讲，它支持两种开发模式：

- Annotation-driven：只需要对应的注解标注在 POJO 上，就可以使用 WebSocket 的生命周期事件；
- Interface-driven：开发者可以实现 `Endpoint` 接口，实现该接口中定义的方法，就可以和 WebSocket 的 lifecycle events 交互了。

#### Lifecycle events

WebSocket 中常用的一些生命周期事件：

- 客户端可以通过发起 HTTP handshake request 来建立 connection；
- 服务端对 handshake request 做出 response；
- 双方建立 connection，从现在开始连接是完全对称的；
- 客户端和服务端可以同时发送和接受消息；
- 直到其中一方关闭 connection。

不管是基于注解的还是基于接口的编程模式，大多数 WebSocket lifecycle events 都会和一个 Java method 关联。

#### Annotation-Driven

Java 中将能够接收 WebSocket Request 的东西定义为 Endpoint，在 Annotation-Driven 模式下可以将 `@ServerEndpoint` 注解标注在某个 POJO 上，它就被声明为一个 Endpoint。该注解的 value 属性定义了 endpoint 的 path，参考如下代码：

```java
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint("/hello")
public class MyEndpoint {
}
```

上面代码的含义就是向 container 在 relative path 'hello' 上发布了一个 endpoint。这个 path 也可以包含 path parameters，比如 `/hello/{userid}`，如果在 lifecycle method 的某个参数上标注了 `@PathParam` 注解就可以获取到 userid 参数值。

假如 web 项目部署在本地机器上，且根路径是 mycontentroot，同时监听 8080 端口，那么 WebSocket 就可以通过 `ws://localhost:8080/mycontentroot/hello` 来访问。

如果一个 POJO 类上标注了 `@ClientEndpoint`，那么它也被声明为一个 endpoint，负责发起 (initiate) 一个 WebSocket connection，`@ClientEndpoint` 和 `@ServerEndpoint` 最大的不同之处在于 ClientEndpoint 无法接收 path value element，因为它不会监听接收的请求。

```java
import javax.websocket.ClientEndpoint;

@ClientEndpoint
public class MyClientEndpoint {
}
```

在 Annotation-Driven 模式下如果想要在 POJO 中初始化一个 WebSocket connection，可以这样做：

```java
javax.websocket.WebSocketContainer container = 
javax.websocket.ContainerProvider.getWebSocketContainer();

container.conntectToServer(MyClientEndpoint.class, 
new URI("ws://localhost:8080/tictactoeserver/endpoint"));
```

此后，标注了 `@ServerEndpoint` 和 `@ClientEndpoint` 的类被称为 annotated endpoints。

一旦建立了 WebSocket connection，就会创建一个 `Session` 并且标注了 `@OnOpen` 注解的方法也会同时被调用。这个方法可以包含一下几个参数：

- 一个 `javax.websocket.Session` 参数，指向了被创建的 Session；
- 一个 `EndpointConfig` 实例，它包含了关于 endpoint configuration 的信息；
- 剩下的可变参数就是标注了 `@PathParam` 注解的参数了，指向 endpoint path 中包含的 path parameters。

下面的代码演示了当 WebSocket "opened" 时建立的 session 的标识符。

```java
@ServerEndpoint("/hello")
public class MyEndpoint {
    
    @OnOpen
    public void myOnOpen(Session session) {
        System.out.println("WebSocket opened: " + session.getId());
    }
    
}
```

只要 WebSocket 没有 closed，那么建立的 Session 实例就是有效的。Session 实例中包含了很多有趣的方法，开发者可以通过调用这些方法来获取关于 connection 的更多信息。此外 Session 中还包含了一个和 application-specific data 的 hook，就是 `getUserProperties()` 方法，它返回一个 `Map<String, Object>` ，通过这个 hook 开发者就可以向其中填充特定的数据，比如和 session 或者 application-specific 相关的数据，这样就可以在 WebSocket lifecycle 对应的多个 method 之间共享同一份数据。

（<font style='color:green'>Tips：注意使用 getUserProperties hook 来保存 session 相关的或者 application-specific 的数据，可以在 lifecycle method 之间共享数据。</font>）

当一个 WebSocket endpoint 收到 messages 时，标注了 `@OnMessage` 的方法就会被调用，这个方法可以包含一下参数：

- `javax.websocket.Session` 参数；
- 标注了 `@PathParam` 注解的一个或多个参数，指向 endpoint path 中的 path parameters；
- 收到的 message，参考下文列出的各种消息类型。

假如对方发过来一个文本消息，下面的代码片段展示了如何打印这个消息：

```java
@ServerEndpoint("/hello")
public class MyEndpoint {
    
    @OnOpen
    public void myOnOpen(Session session) {
        System.out.println("WebSocket opened: " + session.getId());
    }
    
    @OnMessage
    public void myOnMessage(String text) {
        System.out.println("WebSocket received message: " + text);
    }
    
}
```

此外需要注意的是：如果标注了 `@OnMessage` 注解的方法返回类型不是 void，而是其他类型，此时 WebSocket 的 implementation 就会把返回的数据发送给 the other peer（另一个对等方，注意 ws 是由两个对等方建立的），比如下面的代码就把收到的消息返回给消息的发送者：

```java
@OnMessage
public String myOnMessage (String txt) {
   return txt.toUpperCase();
}
```

通过 WebSocket connection 发送消息的另一个方式是这样的：

```java
RemoteEndpoint.Basic other = session.getBasicRemote();
other.sendText ("Hello, world");
```

在这种方式中，我们借助了 Session 实例，注意 WebSocket Lifecycle Method 方法都可以轻易的获取到它。而 `getBasicRemote()` 方法则可以返回 WebSocket 两个连接者中的另一个。`RemoteEndpoint` 实例可以用来发送文本消息，还有下文描述的其他类型的消息。

当 WebSocket connection 即将要关闭时，标注了 `@OnClose` 注解的方法会被调用，这个方法可以包含如下参数：

- `java.websocket.Session` 参数，需要注意的是在 `@OnClose` 标注的方法执行完成并且 return 之后，Session 对象就不可以再使用了，因为此时 connection 真的关闭了；
- 一个 `javax.websocket.CloseReason` 参数描述了关闭 WebSocket 的原因，比如说：正常关闭（normal closure）、协议错误（protocol error）、服务负载非常高导致超载（overloaded service）等等；
- 标注了 `@PathParam` 的可变参数，指向 endpoint path 中的 path parameters。

下面的代码展示如何打印 WebSocket 关闭的原因：

```java
@ServerEndpoint("/hello")
public class MyEndpoint {
    
    @OnOpen
    public void myOnOpen(Session session) {
        System.out.println("WebSocket opened: " + session.getId());
    }
    
    // @OnMessage
    // public void myOnMessage(Session session, String text) throws IOException {
    //     System.out.println("WebSocket received message: " + text);
    // }
    
    // send message through return keyword.
    // @OnMessage
    // public String myOnMessage(String text) throws IOException {
    //     return text;
    // }
    
    // anther way
    @OnMessage
    public void myOnMessage(Session session, String text) throws IOException {
        RemoteEndpoint.Basic remote = session.getBasicRemote();
        remote.sendText("hello another peer.");
    }
    
    
    @OnClose
    public void myOnClose(CloseReason reason) {
        System.out.println("Closing a WebSocket due to " + reason.getReasonPhrase());
    }
    
}
```

除了前面展示的几个生命周期注解，还有一种就是当收到了一个 error 时会触发，可以通过 `@OnError` 注解标注特定的方法。

#### Interface-Driven