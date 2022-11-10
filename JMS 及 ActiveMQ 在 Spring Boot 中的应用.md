# JMS

参考文章：

- [Oracle 关于 JMS 的介绍](https://www.oracle.com/technical-resources/articles/java/intro-java-message-service.html)
- [JSR914](https://www.jcp.org/en/jsr/detail?id=914)
- 旧的 JMS 官网：https://javaee.github.io/jms-spec/
- 新的 JMS（Jakarta Messaging）：https://projects.eclipse.org/projects/ee4j.messaging
- wiki：https://github.com/jakartaee/messaging/wiki



# 简介

远程方法调用叫做 RPC（Remote procedure call），一般包含 Java RMI，它是同步的：调用者调用远程方法时必须阻塞直到方法完成并返回执行结果，此种方式过于依赖远程方法，耦合度过高。换句话说，RPC 系统要求客户端和服务端在同一时间必须都是正常的，而在某些应用当中这种使用场景就不太合适了。

此时 Message-Oriented Middleware（MOM）系统提供了一种解决方案，它们基于异步交互模型，并提供可以跨网络访问的消息队列，注意这里的消息指的是不同系统之间异步请求或者事件，这些消息包含了特定业务所需要的数据。

Java Message Service（JMS），它是由 Sun 公司和其他公司在建立 Java 社区时提出的概念，具体信息在 [JSR914](https://www.jcp.org/en/jsr/detail?id=914) 中，是第一个获得行业内广泛支持的企业消息传递 API。

JMS 旨在开发异步发送和接收异步数据和事件的业务应用程序。JMS 支持两种模式：

（1）point-point（queuing）；

（2）publish-subscribe；

只需遵循 JMS 规范便可开发出企业级消息系统。JMS 可以归属到中间件中的 MOM 类别，这是一个层次较低的抽象层，在数据库、应用适配器、事件处理、业务流程自动化之下。

## 1、架构

一个 JMS 应用由以下几部分组成：

- A JMS Provider：实现 JMS 规范的一个消息系统；
- JMS clients：可以发送和接收消息的 Java 应用；
- Messages：承载 JMS clients 之间交流信息的对象；
- Administered objects：由管理员创建的用于使用 JMS clients 的预配置 JMS 对象。

## 2、消息传递模型

JMS 支持两种不同的消息传递模型：

（1）点对点（destination：队列）：在这种模式下，消息由生产者制造发送给消费者，消息首先会传递到一个队列中，然后它会继续传递给这个队列中注册的消费者。生产者和消费者就通过一个队列联系起来，任意数量的生产者都可以将消息传给该队列，但是一个消息只能被一个消费者所消费。如果没有消费者注册该队列，那么队列就会持久这些消息，直到某个消费者注册并进行消费。

（2）发布/订阅（destination：主题）：在这种模式下，生产者生产的一个消息将会被很多消费者消费。首先消息传递给一个主题（Topic），接下来所有订阅该 Topic 的消费者都会收到这条消息。除此之外，任意数量的生产者都可以发送消息给该 Topic，任意一个消息都可以传递给多个订阅者。如果没有消费者订阅 Topic，Topic 不会保存信息，除非它为不活动的使用者提供订阅。持久订阅就是某个消费者订阅了该 Topic，但是该消费者处理非活动状态。

## 3、JMS 编程模型

一个 JMS 程序由一组程序定义消息（application-define message）和一组 exchange（交换机）message 的 clients 组成。JMS clients 彼此通过 JMS API 发送和接收消息从而进行交互，消息由三部分组成：header、properties 和 a body：

- header：每条消息都必须有的，它包含用于路由消息的以及标识消息的数据，这些数据有一部分在 JMS Provider 制造消息和传递消息的就设置了，然后剩下的由客户端在处理消息时根据消息信息来设置；
- properties：它是可选的，一般提供客户端用于筛选消息的数据，它提供一些附加信息，比如谁创建了这条消息、什么时候创建的。Properties 可以看作是 Header 的一个扩展，它由键值对组成，client 可以利用 properties 来微调消息；
- The body：这个也是可选的，它包含要被 exchanged 的消息的实际数据，JMS 定义了六种 JMS Provider 需要支持的消息种类：

（1）Message：没有消息体的消息；

（2）StreamMessage：body 包含 Java 原始数据流的一种消息，它是按照顺序读取和写入的；

（3）MapMessage：body 包含一组间值对，不用遵循特定的顺序；

（4）TextMessage：body 包含一条 Java String，必须说 XML message；

（5）ObjectMessage：body 包含一个 Java 序列化对象；

（6）BytesMessage：body 包含字节流；

## 4、生产和消费消息

下面从较高的层次展示了消息从制造到消费的一个过程，开发者可以按照步骤开发 clients 从而生产和消费消息，需要注意的是，如果 client 同时制造和消费消息，有些公共的步骤就不需要重复了：

![](https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20221110225004.png)

## 5、生产消息的客户端

按照如下步骤操作：

（1）可以使用 JNDI 找到 `ConnectionFactory` 实例，或者直接实例化并设置某些属性。

一个 client 使用一个 connection factory，它可以是 queue 也可以是 topic，主要用于为 provider 创建 Connection，such as：

```java
Context ctx = new InitialContext();
ConnectionFactory cf1 = (ConnectionFactory) ctx.lookup("jms/QueueConnectionFactory");
ConnectionFactory cf2 = (ConnectionFactory) ctx.lookup("/jms/TopicConnectionFactory");
```

当然，也可以直接实例化：

```java
ConnectionFactory connFactory = new com.sun.messaging.ConnectionFactory();
```

（2）使用 `ConnectionFactory` 对象创建一个 `Connection` 对象，比如：

```java
Connection connection = connFactory.createConnection();
```

注意最终创建的这些 Connection 是要被关闭的，调用这样的方法 `Connection.close()`。

（3）使用 Connection 对象创建一个或者多个 `Session` 对象，Session 提供事务上下文，用于管理一组接收-消费消息的原子工作单元，创建方式如下：

```java
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE)
```

`createSession()` 方法有两个参数：

- 第一个参数（例子中是 false）用于管理是否开启事务；
- 第二个参数管理消息的签收方式，例子中这种自动签收意味着消息被顺利接收之后就自动签收。

（4）通过 JNDI 找到 `Destination` 对象，或者直接实例化；

客户端使用 destination 对象来指定消息的来源和消费消息的对象信息，在点对点模式下，destination 就是 queue，在发布/订阅模型下，destination 就是 topic，比如说：

```java
Destination dest = (Queue) ctx.lookup("jms/SomeQueue");
Queue q = new com.sun.messaging.Queue("world");
```

（5）使用 Session 和 Destination 对象创建 `MessageProvider` 用于将消息发给 Destination；

需要注意的是我们也可以在没有 Destination 的情况下创建 MessageProvider，但是在下面的例子中就必须为生产的每个消息指定 Destination：

```java
MessageProducer producer = session.createProducer(SomeQueue OR SomeTopic);

// 发送消息
producer.send(message);
```

## 6、消费消息的客户端

和生产消息的客户端前四个步骤一样，就不再赘述了；

（5）使用 Session 对象和 Destination 对象创建 `MessageConsumer` 对象用于接收消息，比如下面这样：

```java
MessageConsumer consumer = session.createConsumer(SomeQueue or SomeTopic);
```

注意，如果使用了 Topic，此处也可也创建持久订阅，``Session.createDurableSubscriber()` 。

一旦创建了消费者，它就可以接收消息。但是，消息传递功能直到开启前面创建的连接时才会生效，使用 start 方法：

```java
connection.start();
Message msg = consumer.receive();
```

`receive()` 方法接收一个 long 类型的参数，表示 time-out（比如说 3000L 表示 3 秒 ），这个参数在同步消费时非常重要，当然如果使用异步消费机制，就需要一个消息监听器。

（6）如果想要使用异步消费，就需要为 MessageConsumer 对象注册一个 `MessageListener` 对象；

一个 MessageListener 对象充当消息的异步事件处理器，`MessageListener` 接口只有一个方法：`onMessage()`，这个方法用于接收和处理消息，

在下面的代码中，MyListener 是一个实现了 MessageListener 接口的类，使用消息消费者的方法注册 listener：

```java
MessageListener listener = new MyListener();
consumer.setMessageListener(listener);
```

为了避免消息丢失，Connection 对象的 `start()` 方法要在 Consumer 注册 Listener 之后调用，当消息开始投递的时候，当消息被投递后，JMS providers 就会自动调用消息监听器的 `onMessage()` 方法。

（7）最后一步就是开启 Connection 的 start 方法，表示消息可以开始投递了。

## 7、同步和异步消息消费

JMS Client 可以同步或者异步两种方式消费消息：

- 同步：在这种模式下，client 通过调用 `MessageConsumer` 对象的  `receive()` 方法接收消息，线程阻塞直到该方法返回，如果消息不可用，这个方法就会阻塞调用者直到超时，注意这个模式下，client 依次只能使用一条消息；
- 异步：在这种模式下，client 为 MessageConsumer 注册 MessageListener，有点类似回调，消费者客户端消费一条消息后， session 会调用 `onMessage()` 方法通知。换句话说，线程不会阻塞。



## 8、消息的可靠性

JMS 定义了两种投递模式：

（1）持久化消息：可以确保消息被成功消费且仅仅消费一次，也就是消息不会丢失；

（2）非持久化消息：只保证消息只会投递一次，不在意消息是否丢失。

其实这个问题，就是关于性能的一个权衡问题，消息的传递越可靠，实现这种可靠性所需的带宽和开销就越大。可以通过产生非持久消息来最大化性能，也可以通过产生持久消息来最大化可靠性。

更多信息和高级特性可以自行查阅。

## 9、消息驱动的 Bean

JMS 在 J2EE 平台上使用的很广泛，其中一个很棒的例子就是 message-dirven beans，它是 EJBs 在 EJB 2.0/2.1 中声明的一个种类，另外两个是 session beans 和 entity beans，但是后两者只能同步调用。

MDB（JMS Message-Driven Bean）是实现 JMS MessageListener 接口的 JMS 消息消费者。当 MDB 容器接收到消息时调用 `onMessage()` 方法。注意，我们不需要调用 MDBs 的远程方法，实际上也根本没有相关定义