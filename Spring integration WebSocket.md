## WebSocket

### Reference

- [JSR 356 Java<sup>TM</sup> API for WebSocket](https://jcp.org/en/jsr/detail?id=356)
- https://www.oracle.com/technical-resources/articles/java/jsr356.html
- https://stackoverflow.com/questions/26452903/javax-websocket-client-simple-example
- http://www.programmingforliving.com/2013/08/jsr-356-java-api-for-websocket-client-api.html
- https://github.com/javaee （the legacy Java EE organization）
- https://javaee.github.io/javaee-spec/ （这里可以看企业级 Java 开发相关的 api 和文档，比如 javax.websocket 下的类的 api 文档）
- https://github.com/jakartaee （the current, active Enterprise Java development organization）

对于大多数 web 应用程序而言，基于 HTTP 的请求-响应模型有一定的局限性，信息只能通过一次次请求进行传递，无法持续的传输信息。

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

##### Endpoint and Paramters

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

##### @OnOpen

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

##### @OnMessage

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

##### @OnClose

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

##### @OnError

除了前面展示的几个生命周期注解，还有一种就是当收到了一个 error 时会触发，可以通过 `@OnError` 注解标注特定的方法。

该注解标注的方法可以接收以下可选参数（更多信息参考 [@OnError API doc](https://javaee.github.io/javaee-spec/javadocs/)）：

- 一个可选的 Session 参数；
- 一个 Throwable 参数，可以是以下几种类型：
  - connection problems：比如说在 socket 正常关闭前抛出的异常，一般被包装为 `javax.websocket.SessionException`；
  - runtime errors：一般是开发者自定义的 message handlers 中抛出的；
  - conversion errors：对 income messages 进行 encoding 时抛出的异常，如果正常编码才会调用 message handler。一般被包装为 `javax.websocket.DecodeException`；
- 使用  @PathParam 注解标注的可变参数；

#### Interface-Driven

参考：[Endpoint api doc](https://javaee.github.io/javaee-spec/javadocs/javax/websocket/Endpoint.html#onError-javax.websocket.Session-java.lang.Throwable-)

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

#### Configuration （Important）

<font style="color:red">注意：后文中提到的通过 user properties 来配置 Tomcat 定义的一些属性，这里是 user properties 是通过 `Map<String, Object> userProperties = session.getUserProperties()` 获取的一个 Map，将对应的属性设置进去就可以生效了。</font> 

如果是使用了 Spring 对 WebSocket 的实现，可以参考：https://stackoverflow.com/questions/66482114/how-can-you-set-org-apache-tomcat-websocket-blocking-send-timeout

下面介绍一些 Tomcat 针对 WebSocket 的实现所采用了一些配置属性：

##### Timeout

（1）`org.apache.tomcat.websocket.BLOCKING_SEND_TIMEOUT` 属性用于配置采用 blocking mode 的 WebSocket session 在发送消息时的写入超时时间，默认是 20000 毫秒（i.e. 20s），可以通过用户的配置文件去更改它；

（2）除了使用 Java WebSocket API 的 `Session.setMaxIdleTimeout(long)` 方法外，Tomcat 还提供了更强大的控制由于 session 失活导致的超时问题。

-  `org.apache.tomcat.websocket.READ_IDLE_TIMEOUT_MS` ：该属性决定了经过多少毫秒之后如果一直没有接收到 WebSocket message 就会触发 session timeout；（通常是 server 端等待接收 client 发送过来的消息）；

- `org.apache.tomcat.websocket.WRITE_IDLE_TIMEOUT_MS`：该属性决定了经过多少毫秒之后如果一直没有发送 WebSocket message 就会触发 session timeout；（通常是 server 端发送给 client 消息）；

这两个属性和 `Session.setMaxIdleTimeout(long)` 方法有这样的关系：如果没有使用该方法，就会应用上面两个 read/write 超时属性，而且这两个属性可以同时设置也可以只设置一个。

##### Message Handler

（3）如果应用程序没有为 WebSocket 接收的 binary message 定义 `MessageHandler.Partial` 处理器实现，则一旦接收到 binary message，它们会先进入一个缓冲区（must be buffered），直到接收完整的 binary 消息后调用一次程序中注入的 `MessageHandler.Whole` 实现。

针对 binary message，默认的 buffer size 是 8192 bytes（i.e. 8 MB）。web application 中可以通过设置 servlet context initialization parameter：`org.apache.tomcat.websocket.binaryBufferSize` 来改变缓冲区的大小。

（4）同样的，如果应用程序没有为 WebSocket 接收的 text message 定义 `MessageHandler.Partial` 实现，那么接收到的消息也是先进入一个缓冲区，收到完整的消息后再传给 `MessageHandler.Whole` 实现。默认的缓冲区大小也是 8192 bytes。

web application 中可以通过设置 servlet context initialization parameter：`org.apache.tomcat.websocket.textBufferSize` 来改变缓冲区大小。

（5）Java WebSocket specification 1.0 中规定 the first endpoint 在完成 WebSocket handshake 后就不允许在进行任何编程式的 deployment 了。而默认情况下 Tomcat 则允许进行额外的编程式 deployment。可以通过 `org.apache.tomcat.websocket.noAddAfterHandshake` servlet content initialization parameter 去控制相关行为。如果不想这样，可以通过设置系统属性 `org.apache.tomcat.websocket.STRICT_SPEC_COMPLIANCE` 为 true 即可，但是此时如果显式的配置了 servlet context 参数，则这些参数的优先级会更高。

##### I/O Timeout

（6）当 WebSocket client 同 server endpoint 建立 WebSocket 连接，且通过连接进行 I/O 操作，I/O connection 的超时机制由 `javax.websocket.ClientEndpointConfig` 提供的 `userProperties` 来控制。

属性 `org.apache.tomcat.websocket.IO_TIMEOUT_MS` 可以控制 I/O 操作的超时时间，是一个字符串且以毫秒作为单位，默认是 5000（i.e. 5s）。

##### Secure WebSocket connection

（7）当 WebSocket client 想要连接一个具有安全机制的 secure server endpoint 时，`javax.websocket.ClientEndpointConfig` 提供的 `userProperties` 中可以配置 SSL，支持一下属性：

- `org.apache.tomcat.websocket.SSL_CONTEXT`
- `org.apache.tomcat.websocket.SSL_PROTOCOLS`
- `org.apache.tomcat.websocket.SSL_TRUSTSTORE`
- `org.apache.tomcat.websocket.SSL_TRUSTSTORE_PWD`

默认的 TRUSTSTORE passpord 是 `changeit`；

但是注意如果设置了 `org.apache.tomcat.websocket.SSL_CONTEXT` 属性，此时上面的 `org.apache.tomcat.websocket.SSL_TRUSTSTORE` 和 `org.apache.tomcat.websocket.SSL_TRUSTSTORE_PWD` 会失效。

对于 secure server endpoint，默认会启用主机名校验（host name verification），如果想要绕过这层安全限制（当然是不推荐这样做的，开发者可以提供自己的 SSLContext 实现，并通过设置 `org.apache.tomcat.websocket.SSL_CONTEXT` 用户属性指向该实现。需要注意的是我们定制的 `SSLContext`使用的 `TrustManager` 必须是继承自 `javax.net.ssl.X509ExtendedTrustManager` 的实现，通过覆盖其中的特定方法来决定是否采用特殊的验证方式或者不进行验证。

（8）当 WebSocket Client 和 server endpoint 建立 connection 时，client 进行 HTTP 重定向的次数由 `javax.websocket.ClientEndpointConfig` 提供的 `userProperties` 中的 `org.apache.tomcat.websocket.MAX_REDIRECTIONS` 属性决定的，默认值是 20，如果该值为 0 则表示禁用 HTTP 重定向。

（9）如果同 client 建立 connection 的 server endpoint 要求 BASIC 或者 DIGEST 认证，则可以采用下面的配置属性：

- `org.apache.tomcat.websocket.WS_AUTHENTICATION_USER_NAME`
- `org.apache.tomcat.websocket.WS_AUTHENTICATION_PASSWORD`

此外，目标 server 如果指定了特定的 realm，那么 WebSocket client 也可以配置为针对特定的 realm 发送 credentials（凭据），相关属性如下：

- `org.apache.tomcat.websocket.WS_AUTHENTICATION_REALM`

（10）如果 WebSocket client 是通过 proxy forward 和 server endpoint 建立连接的（也可以理解为 server endpoint 外面有一层 gateway），并且这个 proxy 也需要 BASIC 或者 DIGEST 认证，则需要设置以下 user properties：

- `org.apache.tomcat.websocket.WS_PROXY_AUTHENTICATION_USER_NAME`
- `org.apache.tomcat.websocket.WS_PROXY_AUTHENTICATION_PASSWORD`

此外，在此种情况下如果也指定了特定了 realm 才发送 credentials 则可以通过这个属性来配置 realm：

- `org.apache.tomcat.websocket.WS_PROXY_AUTHENTICATION_REALM`



#### Ping/Pong Implementation

RFC 中也定义了关于 server 和 client 的 ping/pong 交互模式，往往用于 client 和 server 的 connection 心跳机制。

可以参考：https://dzone.com/articles/ping-pong-implementation-jsr-356

参考：

- https://dzone.com/articles/ping-pong-implementation-jsr-356
- https://github.com/abhijeetashri/websocket-ping-pong-java/blob/main/src/com/websocket/pingpong/endpoint/WebsocketEventsEndpoint.java
- https://github.com/morgwai/servlet-utils
- https://github.com/morgwai/servlet-scopes
- https://github.com/morgwai/servlet-scopes



## Spring Boot WebSocket

参考：https://docs.spring.io/spring-framework/docs/5.3.31/reference/html/web.html#websocket-server

### Spring WebSocket Support

Spring 体系对 WebSocket 的支持：

- Spring Framework 提供了 WebSocket API（包括 client 和 server）；
  - Spring 的 WebSocket Support 并不强制依赖于 Spring MVC，其他框架也可以实现 Spring 提出的 WebSocket 相关接口。
  - 包：spring-websocket-${latest}.jar

### Spring MVC Integrate WebSocket

- Spring MVC 可以很容易集成 Spring WebSocket API：
  - DispatchServlet 既可以处理常规的 HTTP 请求，也可以处理 WebSocket 的 handshake 请求；
  - 在其他的 http 处理场景中，也可以借助 WebSocketHttpRequestHandler 实现对 WebScoket 的支持；
  - 但是需要注意如果是和 JSR 356 一块工作就需要注意一些东西；
  - Java WebSocket API（JSR-356）提供两种部署机制：
    - 第一种基于注解的方式涉及到在应用程序启动时 Servlet Container 的 classpath scan，这也是 servlet 3 的一个特性；
    - 第二种基于接口的方式则是借助了 SPI 功能，在 Servlet 容器启动的时候使用相关的 registration API 注入开发者提供的实现；
    - 但是这两种机制都没法使用一个 "front controller" （比如 Spring MVC 的 DispatchServlet）去同时处理 websocket 的 handshake 和其他常规 HTTP 请求；
  - 这是一个 JSR 356 的一个重要的限制，Spring MVC 为了解决这个问题，利用了特定 servlet server （Tomcat 等等）对 `RequestUpgradeStrategy` API 的实现。
  - 这样的策略在 Tomcat、Jetty、GlassFish、WebLogic，WebSphere、Undertow 中都存在。

```
针对取消 JSR 356 的这个限制的请求可以参考：https://github.com/jakartaee/websocket/issues/211
Tomcat、Undertow、WebSphere 都提供了自己的 API 替代方案。
```

第二个需要考虑的是支持 JSR-356 的 Servlet Containers 会去扫描 `ServletContainerInitializer`（SCI），这可能会拖慢应用启动速度。在某些场景下，比如升级到具有JSR-356支持的Servlet容器版本后观察到重大影响，则应该可以通过使用web.xml中的 &lt;absolute-ordering /&gt;元素来选择性地启用或禁用 web fragments (和 SCI 扫描)，如下面的示例所示：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering/>

</web-app>
```

也可以有选择性的开启某些 web fragments，通过名字指定就行，比如 Spring 的 `SpringServletContainerInitializer`，该类提供了 Servlet 3 Java Initialization API：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering>
        <name>spring_web</name>
    </absolute-ordering>

</web-app>
```



### WebSocket Server Configuration

每个底层的 WebSocket Engine 都会暴露一些配置给开发人员调整，比如前面提到的 Tomcat 允许配置的几个属性，如消息 buffer size、idle timeout 等等。

> Tomcat、WildFly、GlassFish

在 Spring 应用中，针对 Tomcat、WildFly 和 GlassFish，可以在配置类中注入定制的 `ServletServerContainerFactoryBean` 即可，比如下面这样：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }

}
```

如果是 client-size 需要改动 WebSocket 的配置，则使用 `WebSocketContainerFactoryBean`。

> Jetty

如果用的是 Jetty 实现，则需要提供一个预先定制好的 `WebSocketServerFactory` 示例，并把它注入 Spring 的 `DefaultHandshakeHandler` 中：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(),
            "/echo").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }

}
```



### Allowed Origins

Spring Framework 4.1.5 中，WebSocket 和 SockJS 默认的行为是接收同源请求，也可以允许某个特地的来源集合。这个检测主要是为 browser clients 设计的，不影响其他 clients。

The three possible behaviors are:

- Allow only same-origin requests (default): In this mode, when SockJS is enabled, the Iframe HTTP response header `X-Frame-Options` is set to `SAMEORIGIN`, and JSONP transport is disabled, since it does not allow checking the origin of a request. As a consequence, IE6 and IE7 are not supported when this mode is enabled.
- Allow a specified list of origins: Each allowed origin must start with `http://` or `https://`. In this mode, when SockJS is enabled, IFrame transport is disabled. As a consequence, IE6 through IE9 are not supported when this mode is enabled.
- Allow all origins: To enable this mode, you should provide `*` as the allowed origin value. In this mode, all transports are available.

You can configure WebSocket and SockJS allowed origins, as the following example shows:

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("https://mydomain.com");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```



## Example

前面已经了解过 WebSocket 的相关规范以及在 Java 中的应用，主要有以下要点：

- [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455)
- [JSR 356](https://jcp.org/en/jsr/detail?id=356)
- Tomcat 对 JSR 356 的实现
- Spring 对 WebSocket 的支持；
  - 以及是如何解决 Spring WebSocket 结合 JSR 356 运行时环境的

创建一个 Spring Boot 工程，选择的版本是 2.7.17，由于是内嵌了 Tomcat，如果需要针对 WebSocket Server 端做一些配置就需要通过  Java Config 的方式。

Spring Boot websocket starter 中注入了 Spring websocket 依赖，真正负责处理 WebSocket 协议的其实还是底层的 servlet container 实现，这里默认使用 Tomcat。

- Servlet Container 实现负责提供 JSR 356 runtime；

- Spring MVC 的 DispatchServlet 可以处理 websocket 的 upgrade 请求以及其他正常的 http 请求；
- Spring 的 websocket 模块负责提供更多强大的功能：
  - 建立 websocket connection 时可以做一些操作；
  - 不同消息类型的 handler 实现；
  - 等等

<font style="color:blue">注意：可以参考多个版本的 Spring doc 的 websocket 章节。Spring 是从 4.0 版本开始支持 websocket 的。</font>

### Spring 4.0 WebSocket Support

文档地址：https://docs.spring.io/spring-framework/docs/4.0.0.RELEASE/spring-framework-reference/html/websocket.html

从 Spring 4.0 开始新增了两个模块：

- `spring-websocket` module：为 web application 提供双向的基于 WebSocket 的通信机制。主要是为了适配  JSR-356。同时考虑到有些浏览器（比如 IE < 10 的版本）不支持 websocket，Spring 也提供了一种可选措施：SockJS-based（i.e. WebSocket emulation）；
- `spring-messaging` module：
  - 增加了对 STOMP 的支持，STOMP 是 WebSocket 的子协议。在基于注解的编程模式中，一个 @Controller 可以同时使用 @RequestMapping 和 @MessageMapping，前者可以处理 HTTP 请求，后者可以处理 WebSocket-clients 发送过来的 message；
  - 该模块同时也负责提供 Spring-Integration 工程中的某些核心抽象，比如：Message、MessageChannel、MessageHandler，以及其他基础设施，主要服务于 messaging applications。

关于 WebSocket 的更多细节应该参考 [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455)，在本文中至少应该明白 HTTP 是如何 initial handshake，这依赖于 HTTP 的一项机制：protocol upgrade（i.e. protocol switch），server 通过返回响应码 101 表示同意客户端的协议升级请求，假设 handshake 成功，HTTP upgrad request 底层的 TCP socket 就会保持 open 状态，此时 server 和 client 就可以进行双向通信了。

- Spring Framework 4 的 spring-websocket 模块为 WebSocket 提供了全面的支持，同时兼容了 Java WebSocket API（JSR-356），同时提供其他功能；
- 但是有些时候客户端是不支持 WebSocket 协议的，比如某些浏览器不支持 WebSocket 协议，或者在一些特殊的场景中使用了严格的代理策略也可能会阻止 HTTP upgrade 请求，毕竟该请求要维持很长时间的连接（可以参考这篇文章：[How HTML5 Web Sockets Interact With Proxy Servers](https://www.infoq.com/articles/Web-Sockets-Proxy-Servers/)）
  - 因此如果你想要构建一个基于 WebSocket 的应用，有时候能够模拟 WebSocket API 的备选方案是不可缺少的。在 Spring framework 中，基于 [SockJS protocol](https://github.com/sockjs/sockjs-protocol) 提供了一种透明的备选方案，通过开启相关配置就可以使用它。

#### Messaging Architecture

在使用 WebSocket 的时候有些设计理念应该尽早考虑到，特别是此类应用和常规 web application 的差异。

当今 REST 是一种广泛使用的 web application 架构，它依赖于 URLs、HTTP methods，以及其他某些设计原则，比如 hypermedia（links），stateless 等等。

和它们相比，WebSocket 仅仅使用一个 URL，而且该 URL 就是为了初始化 HTTP handshake。此后，所有的消息都是在这一条 connection 中 share and flow。这是一种完全不同的、异步的、基于事件驱动的消息架构。它更接近于传统的 messaging applications（e.g. JMS, AMQP）。

Spring Framework 包含了一个新的 `spring-messaging` 模块，这个模块为 `spring-integration` 工程提供了一些核心的抽象，比如 Message、MessageChannel、MessageHandler 以及其他服务于 messaging architecture 的基础设施。这个模块还包含了一些注解，可以将 message mapping 到某个 method 上，类似于 Spring MVC 的注解开发模式。

#### Sub-Protocol Support

WebSocket 提供了一种 `messaging architecture` ，但是没有强制要求使用特定的 `messaging protocol`。它是 TCP 上的一个轻量的一个 layer，主要功能就是将字节流转换为消息流（包括 text 和 binary），至于消息具体的含义则是由应用程序决定的。

和 HTTP 这种 application-level protocol 不同，对于框架而言，WebSocket protocol 传输过来的消息携带的信息非常少，没法基于这些信息去 route and process 数据。可以看出来，WebSocket 所在的层次确实比较低，除了某些特殊的应用需要针对底层做很多处理，其他的应用要使用它一般会构造一个基于 WebSocket 的更上层的框架。这就和现在的 web 应用会使用 web 框架而不是单独的 Servlet API 是类似的道理。

出于这个原因，WebSocket RFC 定义了 [sub protocols](https://datatracker.ietf.org/doc/html/rfc6455#section-1.9)。在进行 handshake 的时候，客户端和服务端可以使用 `Sec-WebSocket-Protocol` 头来采用某个 sub-protocol（也就是一个更高级的 application-level 的协议）。使用 sub-protocol 并不是必须的，但是如果不用，应用程序依然需要选择一个 client 和 server 都能理解的消息格式，这种格式可以是定制的，也可以是某些框架提供的，也可以是一种标准的消息协议。

Spring Framework 提供了对 [STOMP](https://stomp.github.io/stomp-specification-1.2.html#Abstract) 的支持 —— 一种简单的消息协议，最早用于脚本语言，inspired by HTTP。

#### WebSocket Server

Spring Framework 提供了一套可以适配多种 WebSocket engines 的 WebSocket API。比如说，它可以运行在 Tomcat（7.0.47+）提供的 JSR-356 runtimes 中，或者 GlassFish（4.0+），也可以适配其他 WebSocket runtimes，比如说 Jetty（9.0+）提供的 native WebSocket Support。

注意：

正如前面了解到的，直接使用 WebSocket API 对于应用程序而言是非常底层的 —— 在消息的格式被确定之前，框架没法通过注解去解释并路由消息。这就是为啥程序需要考虑使用一种 sub-protocol。

当我们使用一个高层次的 protocol 时，WebSocket API 的细节就没那么重要了，这就和使用 HTTP 时，TCP communication 也不会暴露给应用程序。

> 一、WebSocketHandler

想要创建一个 WebSocket Server 是很简单的，可以去实现 `WebSocketHandler` 接口，也可以扩展它的实现类 `TextWebSocketHandler` 或者 `BinaryWebSocketHandler`。（接口 - 抽象类 - 具体实现 - 应用程序扩展）

```java
public class MyHandler extends TextWebSocketHandler {

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        // TODO 针对文本消息的特定处理逻辑
    }
    
}
```

下面通过 java-config 为某个特定的 URL 注入我们的文本消息处理器：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfiguration implements WebSocketConfigurer {
    
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler");
    }
    
    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }
    
}
```

如果是在 Spring MVC 应用程序中且存在 DispatchServlet 的配置（因为要借助 DispatchServlet 去处理 Upgrade 请求），就可以使用上面的配置，但是注意 Spring Websocket support 并不强依赖 Spring MVC 框架，也可以集成到其他提供 HTTP 处理机制的 Web 框架，此时可以用 `WebSocketHttpRequestHandler` 实现。





TODO：

注意在 Spring Boot 中内嵌的 Tomcat 如果同时要使用 WebSocket 功能，需要这样：

- https://stackoverflow.com/questions/52185059/use-java-websocket-api-in-spring-boot-application
- https://thegeekyasian.com/websocket-in-spring-boot/



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