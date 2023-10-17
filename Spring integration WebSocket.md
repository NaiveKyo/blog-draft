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

### JSR 356 Java API for WebSocket

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

> @OnOpen

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

> @OnMessage

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

> @OnClose

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

> @OnError

除了前面展示的几个生命周期注解，还有一种就是当收到了一个 error 时会触发，可以通过 `@OnError` 注解标注特定的方法。

#### Interface-Driven

基于注解驱动时，我们只需要在特定的类或者方法上标注 WebScoket 相关的生命周期注解即可。而在基于接口的方式中，开发者需要继承 `javax.websocket.Endpoint` 抽象类并 override 其中的 onOpen、onClose 和 onError 方法。

```java
public class MyEndpoint extends Endpoint {

    @Override
    public void onClose(Session session, CloseReason closeReason) {
        super.onClose(session, closeReason);
    }

    @Override
    public void onError(Session session, Throwable throwable) {
        super.onError(session, throwable);
    }

    @Override
    public void onOpen(Session session, EndpointConfig config) {
        // 需要注入一个 MessageHandler 来拦截消息
        // 可以查看 Tomcat 或者 Spring-Websocket 框架对该接口的实现
        session.addMessageHandler(new MessageHandler() {
        });
    }
}
```

同时注意，为了能拦截消息，需要在 onOpen 方法实现中为当前 Session 注入一个 `javax.websocket.MessageHandler` 实现，可以参考 Tomcat 或者 Spring-Websocket 中的实现。

查看 `MessageHandler` 接口源码，可以看到该接口内部还定义了两个接口 `javax.websocket.MessageHandler.Partial` 和 `javax.websocket.MessageHandler.Whole`，如果开发者需要 WebScoket 在收到部分消息时就得到通知，就需要实现 `Partial` 接口，反之，在收到完整的消息后通知就需要实现 `Whole` 接口。

下面的代码片段展示了如何在收到完整的文本消息将其转化为大写模式并返回给另一个 peer：

```java
@Override
public void onOpen(Session session, EndpointConfig config) {
    // 需要注入一个 MessageHandler 来拦截消息
    // 可以查看 Tomcat 或者 Spring-Websocket 框架对该接口的实现
    final RemoteEndpoint.Basic remote = session.getBasicRemote();
    session.addMessageHandler(new MessageHandler.Whole<String>() {
        @Override
        public void onMessage(String message) {
            try {
                remote.sendText(message.toUpperCase(Locale.ROOT));
            } catch (IOException e) {
                // handle send failure here
            }
        }
    });
}
```

#### Message Types, Encoders, and Decoders

Java 提供的 WebSocket API 非常强大，因为它允许以 Java POJO 的形式收发 WebSocket 消息。

Java API 默认提供三种基本消息类型：

- Text-bases messages；文本类型
- Binary messages；二级制消息；
- Pong messages；这种消息和 WebSocket connection 本身有关。

在 interface-driven 模式下，每个 session 对象都可以注入至少一个 `MessageHandler` 用于处理各种类型的消息。

而使用 annotation-driven 模式时，我们在标注了 `@OnMessage` 注解的方法中，可以提供一个关于消息的参数，此时消息的类型就取决于开发者声明的消息类型。

我们可以查看 Javadoc 中关于 `@OnMessage` 注解的描述，看看基于前面提到的三种基础类型的消息参数类型：

- https://javaee.github.io/javaee-spec/
- https://javaee.github.io/javaee-spec/javadocs/
- https://javaee.github.io/javaee-spec/javadocs/javax/websocket/OnMessage.html

- 如果方法要处理 text messages：
  - `String` 参数表示接收 whole message；
  - Java 的原始数据类型或者包装类也是接收 whole message；
  - 一个 `String`  类型参数外加一个 `boolean` 基础类型，表示一部分一部分的接收消息；
  - `Reader` 参数类型以 blocking stream 的形式接收 whole message；
  - 如果这个 Endpoint 有一个 text decoder（`Decoder.Text` 或者 `Decoder.TextStream`）时，也可以声明其他的 object parameter；
- 如果方法要处理 binary message：
  - `byte[]` 或者 `ByteBuffer` 参数用于接收 whole message；
  - `byte[]` 和一个 `boolean` 组合，或者 `ByteBuffer` 和一个 `boolean` 组合用于一部分一部分的接收消息；
  - `InputStream` 以 blocking stream 的形式接收 whole message；
  - 如果这个 Endpoint 有一个 binary decoder（`Decoder.Binary` 或者 `Decoder.BinaryStream`）时，也可以声明其他的 object parameter。
- 如果方法要处理 pong message：
  - `PongMessage` 类型参数用于处理 pong message。

使用 encoder 我们可以将任何 Java object 转化为 text 或者 binary 消息，进而将 text-base 或者 binary message 传给另一个 peer，收到消息后也可以将消息 decoded 为 Java object， XML 或者 JSON 也可以作为消息格式也是一样的流程。

开发者可以通过实现 `javax.websocket.Encoder` 接口来构造自己的 encoder，同样 decoder 需要实现 `javax.websocket.Decoder` 接口。此外，一个 endpoint 实例需要知道可用的 encoders 和 decoders，在 annotation-driven 模式下，我们可以为 `@ClientEndpoint` 或者 `@ServerEndpoint` 注解指定 encoder 和 decoder 集合。

下面的例子展示了如何注册一个 `MessageEncoder`，它可以将 `MyJavaObject` 实例转化为 text message，而 `MessageDecoder` 的功能正好相反：

```java
@ServerEndpoint(value="/endpoint", encoders = MessageEncoder.class, decoders= MessageDecoder.class)
public class MyEndpoint {
...
}

class MessageEncoder implements Encoder.Text<MyJavaObject> {
   @override
   public String encode(MyJavaObject obj) throws EncodingException {
      ...
   }
}

class MessageDecoder implements Decoder.Text<MyJavaObject> {
   @override 
   public MyJavaObject decode (String src) throws DecodeException {
      ...
   }

   @override 
   public boolean willDecode (String src) {
      // return true if we want to decode this String into a MyJavaObject instance
   }
}
```

注意，`Encoder` 接口中还定义了很多子接口：

- `Encoder.Text`：将 Java objects 转化为 text messages；
- `Encoder.TextStream`：将 Java objects 添加到 character stream 中；
- `Encoder.Binary`：将 Java objects 转化为 binary messages；
- `Encoder.BinaryStream`：将 Java objects 添加到 binary stream 中；

类似的，`Decoder` 接口中也定义了四种子接口：

- `Decoder.Text`：将 text message 转化为 Java object；
- `Decoder.TextStream`：从 character stream 中读取 Java object；
- `Decoder.Binary`：将 binary message 转化为 Java object；
- `Decoder.BinaryStream`：从 binary stream 中读取 Java object；

#### Conclusion

Java 集成了 IETF WebSocket 规范，并提供 WebSocket 相关的 Java API，开发者可以通过实现这些 API 来构造自己的 WebSocket 实现。这样不管是 Web Clients 还是 Native Clients，只要同样实现了 WebSocket 规范，它们就可以很轻松的使用 WebSocket 协议同 Java 后端通信。

同时 Java API 也很灵活，配置也很简单，Java 开发者可以选择自己喜欢的方式去使用它（基于注解，或者基于接口）。

## Practice

### Tomcat Server

由于 WebSocket 的 server 桌面端实现不太好做，因此选取 SpringBoot 内嵌 Tomcat Servlet 容器来做 WebSocket 的服务端，新建一个 SpringBoot 项目即可。

找到 Apache Tomcat 官方文档中关于 WebSocket 的部分，比如 springboot 2.7.15 版本内嵌的 tomcat 是 9.0.79，官方文档地址：

- https://tomcat.apache.org/tomcat-9.0-doc/web-socket-howto.html

可以看到 Tomcat 实现的是 JSR-356 提出的 WebSocket 1.1 API。

Tomcat 官方给出了一些例子，包括 client 端的和 server 端的，可以参考 Github 案例：

- [Client side example](https://github.com/apache/tomcat/tree/9.0.x/webapps/examples/websocket)
- [Server side example](https://github.com/apache/tomcat/tree/9.0.x/webapps/examples/WEB-INF/classes/websocket)

Tomcat 提供了 WebSocket 规范提出了不少建议（和 Tomcat 自身有关的），随着时间的推移，这些规范也会被 WebScoket 采纳。

#### Configuration

Tomcat 的 WebSocket 实现采用了一些配置属性，比如下面这些：

（1）`org.apache.tomcat.websocket.BLOCKING_SEND_TIMEOUT` 属性用于配置采用 blocking mode 的 WebSocket session 在发送消息时的写入超时时间，默认是 20000 毫秒（i.e. 20s），可以通过用户的配置文件去更改它；

（2）除了使用 Java WebSocket API 的 `Session.setMaxIdleTimeout(long)` 方法外，Tomcat 还提供了更强大的控制由于 session 失活导致的超时问题。

-  `org.apache.tomcat.websocket.READ_IDLE_TIMEOUT_MS` ：该属性决定了经过多少毫秒之后如果一直没有接收到 WebSocket message 就会触发 session timeout；（通常是 server 端等待接收 client 发送过来的消息）；

- `org.apache.tomcat.websocket.WRITE_IDLE_TIMEOUT_MS`：该属性决定了经过多少毫秒之后如果一直没有发送 WebSocket message 就会触发 session timeout；（通常是 server 端发送给 client 消息）；

这两个属性和 `Session.setMaxIdleTimeout(long)` 方法有这样的关系：如果没有使用该方法，就会应用上面两个 read/write 超时属性，而且这两个属性可以同时设置也可以只设置一个。

（3）如果应用程序没有为 WebSocket 接收的 binary message 定义 `MessageHandler.Partial` 处理器实现，则一旦接收到 binary message，它们会先进入一个缓冲区（must be buffered），直到接收完整的 binary 消息后调用一次程序中注入的 `MessageHandler.Whole` 实现。

针对 binary message，默认的 buffer size 是 8192 bytes（i.e. 8 MB）。web application 中可以通过设置 servlet context initialization parameter：`org.apache.tomcat.websocket.binaryBufferSize` 来改变缓冲区的大小。

（4）同样的，如果应用程序没有为 WebSocket 接收的 text message 定义 `MessageHandler.Partial` 实现，那么接收到的消息也是先进入一个缓冲区，收到完整的消息后再传给 `MessageHandler.Whole` 实现。默认的缓冲区大小也是 8192 bytes。

web application 中可以通过设置 servlet context initialization parameter：`org.apache.tomcat.websocket.textBufferSize` 来改变缓冲区大小。

（5）Java WebSocket specification 1.0 中规定 the first endpoint 在完成 WebSocket handshake 后就不允许在进行任何编程式的 deployment 了。而默认情况下 Tomcat 则允许进行额外的编程式 deployment。可以通过 `org.apache.tomcat.websocket.noAddAfterHandshake` servlet content initialization parameter 去控制相关行为。如果不想这样，可以通过设置系统属性 `org.apache.tomcat.websocket.STRICT_SPEC_COMPLIANCE` 为 true 即可，但是此时如果显式的配置了 servlet context 参数，则这些参数的优先级会更高。

（6）当 WebSocket client 同 server endpoint 建立 WebSocket 连接，且通过连接进行 I/O 操作，I/O connection 的超时机制由 `javax.websocket.ClientEndpointConfig` 提供的 `userProperties` 来控制。

属性 `org.apache.tomcat.websocket.IO_TIMEOUT_MS` 可以控制 I/O 操作的超时时间，是一个字符串且以毫秒作为单位，默认是 5000（i.e. 5s）。

（7）当 WebSocket client 想要连接一个具有安全机制的 secure server endpoint 时，`javax.websocket.ClientEndpointConfig` 提供的 `userProperties` 中可以配置 SSL，支持一下属性：

- `org.apache.tomcat.websocket.SSL_CONTEXT`
- `org.apache.tomcat.websocket.SSL_PROTOCOLS`
- `org.apache.tomcat.websocket.SSL_TRUSTSTORE`
- `org.apache.tomcat.websocket.SSL_TRUSTSTORE_PWD`

默认的 TRUSTSTORE passpord 是 `changeit`；

但是注意如果设置了 `org.apache.tomcat.websocket.SSL_CONTEXT` 属性，此时上面的 `org.apache.tomcat.websocket.SSL_TRUSTSTORE` 和 `org.apache.tomcat.websocket.SSL_TRUSTSTORE_PWD` 会失效。

对于 secure server endpoint，默认会启用主机名校验（host name verification）



TODO：

For secure server end points, host name verification is enabled by default. To bypass this verification (not recommended), it is necessary to provide a custom `SSLContext` via the `org.apache.tomcat.websocket.SSL_CONTEXT` user property. The custom `SSLContext` must be configured with a custom `TrustManager` that extends `javax.net.ssl.X509ExtendedTrustManager`. The desired verification (or lack of verification) can then be controlled by appropriate implementations of the individual abstract methods.

When using the WebSocket client to connect to server endpoints, the number of HTTP redirects that the client will follow is controlled by the `userProperties` of the provided `javax.websocket.ClientEndpointConfig`. The property is org.apache.tomcat.websocket.MAX_REDIRECTIONS. The default value is 20. Redirection support can be disabled by configuring a value of zero.

When using the WebSocket client to connect to a server endpoint that requires BASIC or DIGEST authentication, the following user properties must be set:

- `org.apache.tomcat.websocket.WS_AUTHENTICATION_USER_NAME`
- `org.apache.tomcat.websocket.WS_AUTHENTICATION_PASSWORD`

Optionally, the WebSocket client can be configured only to send credentials if the server authentication challenge includes a specific realm by defining that realm in the optional user property:

- `org.apache.tomcat.websocket.WS_AUTHENTICATION_REALM`

When using the WebSocket client to connect to a server endpoint via a forward proxy (also known as a gateway) that requires BASIC or DIGEST authentication, the following user properties must be set:

- `org.apache.tomcat.websocket.WS_PROXY_AUTHENTICATION_USER_NAME`
- `org.apache.tomcat.websocket.WS_PROXY_AUTHENTICATION_PASSWORD`

Optionally, the WebSocket client can be configured only to send credentials if the server authentication challenge includes a specific realm by defining that realm in the optional user property:

- `org.apache.tomcat.websocket.WS_PROXY_AUTHENTICATION_REALM`



TODO：后续步骤

- 编写 Tomcat + WebSocket 案例；
- Spring MVC 对 WebSocket 的支持：https://docs.spring.io/spring-framework/docs/5.3.30/reference/html/web.html#websocket
- SpringBoot 内嵌 Tomcat 且自动装配 WebSocket：
  - https://spring.io/projects/spring-boot#learn
  - https://docs.spring.io/spring-boot/docs/2.7.16/reference/html/
  - https://docs.spring.io/spring-boot/docs/2.7.16/reference/html/messaging.html#messaging.websockets
  - 参考配置类：
  - org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
  - org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration
  - org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration

