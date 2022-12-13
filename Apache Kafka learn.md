# Apache Kafka

website：https://kafka.apache.org/

笔者在接触 kafka 之前已经使用过其他的消息队列，比如 activemq，它是标准的 JMS 实现，至于 kafka，看网上经常将它和其他消息队列放在一起讲，但是 kafka 究竟和 JMS 是什么关系呢？可以参考这位大佬的文章：

- https://www.kai-waehner.de/blog/2022/05/12/comparison-jms-api-message-broker-mq-vs-apache-kafka/

该文从十个方面对消息队列和 kafka 进行了比较：

1. Message broker vs. data streaming platform（消息代理者 vs 数据流平台）
2. API Specification vs. open-source protocol implementation（JMS API 实现 vs 开源数据流协议实现）
3. Transactional vs. analytical workloads（事务 vs 工作负载分析）
4. Push vs. pull message consumption
5. Simple vs. powerful and complex API
6. Storage for durability vs. true decoupling
7. Server-side data-processing vs. decoupled continuous stream processing（服务端数据处理 vs 解耦的持续数据流处理）
8. Complex operations vs. serverless cloud（复杂的操作 vs 无服务云函数）
9. Java/JVM vs. any programming language
10. Single deployment vs. multi-region (including hybrid and multi-cloud) replication

# Introduction

## What is event streaming?

Event streaming 类似于人体的中央神经处理系统。

从技术角度看，Event streaming 就是以事件流的形式实时地从数据源中捕获数据的一种实践。

- 这些数据源有数据库、传感器、移动设备、云服务或者应用程序。

- 捕获到的事件流将会被持久化以供后续的检索；

- 实时或者回顾性的去操作、处理甚至是相应数据流；
- 根据需要将事件流路由到不同的目的地。

数据在事件流中不断地流动，并根据需要对数据进行解析，最后，在任何时间、任何地点，我们都可以实时地获取到正确的信息。

## What can I use event streaming for?

事件流的应用范围非常广泛。比如下列情形：

- 实时处理订单或者金融事务。比如证券交易所、银行和保险公司；
- 实时跟踪和监控汽车、卡车、车队和货物，比如物流和汽车制造业；
- 持续捕捉和分析从物联网设备或其他设备传感器上传的数据，比如说工厂或者风电场；
- 收集客户信息或者订单信息并迅速做出响应，比如在零售业、酒店、旅游行业以及移动应用程序；
- 检测医院中病人的情况，预测病情变化，以确保能够及时处理突发状况；
- 存储并协同处理公司各个部门产生的数据；
- 作为数据平台、事件驱动架构和微服务的基础。

## Apache kafka is an event streaming platform

Kafka 结合了三个关键的功能，基于它们就可以利用事件流实现端到端的一站式解决方案：

1. To **publish** (write) and **subscribe to** (read) streams of events, including continuous import/export of your data from other systems.（发布/订阅事件流）
2. To **store** streams of events durably and reliably for as long as you want.（持久、可靠的存储事件流）
3. To **process** streams of events as they occur or retrospectively.（在发生或者回顾事件时处理事件流）

所有这些功能都以分布式、高度可伸缩、弹性、容错和安全的方式提供。Kafka 可以部署在裸机、虚拟机和容器上，也可以部署在本地和云中。可以选择自己管理 Kafka 环境，也可以选择使用各种供应商提供的托管服务。



## How does Kafka work in a nutshell?

Kafka 是如何工作的？

Kafka 是一个分布式系统，主要包括客户端和服务端，它们彼此通过高性能的 [TCP network protocol](https://kafka.apache.org/protocol.html) 进行通信。并且 Kafka 也可以部署在各种环境中，比如裸机、虚拟机、本地、云。

### Servers

**Servers**：Kafka 以集群形式运行，包括一个或者多个服务器。这些服务器可以跨多个数据中心或者云。

- 其中有一些服务器用于存储数据，构成了 storage layer，可以称它们为 brokers（代理者）；
- 还有一些服务器运行 [Kafka Connect](https://kafka.apache.org/documentation/#connect)，这些 Connect 是 Kafka 和其他系统沟通的桥梁，可以将数据以事件流的形式持续导入和导出。这样 Kafka 就可以和现有系统（比如关系型数据库和其他 Kafka 集群）继承；

最后，Kafka 集群是高度可伸缩和容错的：如果集群中任何一个服务出现了故障，其他服务器将接管它们的工作，以确保集群在没有任何数据丢失的情况下持续运行下去。

### Clients

**Clients**：它们允许开发者编写分布式应用程序和微服务，这些应用程序和微服务可以并行的、大规模地读取、写入和处理事件流，即使出现了网络问题或者机器故障的情况下也能以容错的方式进行。

Kafka 社区提供了一些增强型的 [clients](https://cwiki.apache.org/confluence/display/KAFKA/Clients)：这些客户端能够用于 Java 和 Scala，包括更高级别的 Kafka Streams 库，还适用于 Go、Python、C/C++ 和其他很多编程语言，甚至是 REST API。

## Main Concepts and Terminology

### Events

**Event** 记录的是世界上或者应用程序中 "something happened" 的事物。在文档中也可以叫做记录或者消息。我们以事件的形式读取或者写入数据到 Kafka。从概念上讲，事件具有键、值、时间戳和可选的元数据头（metadata header）下面是一个例子：

- Event key："Alice"
- Event value："Made a payment of $200 to Bob"
- Event timestamp："Jun.25, 2020 at 2:06 p.m."

### Producers

**Producers** 就是客户端应用程序：publish (write) events to Kafka, and **consumers** are those that subscribe to (read and process) these events.

在 Kafka 中，生产者和消费者是高度解耦的，并且彼此不可见，这种特性也是 Kafka 实现高可伸缩性的关键点。比如说，生产者无需等待消费者。Kafka 提供了各种 [guarantees](https://kafka.apache.org/documentation/#semantics)，比如确保事件只会被处理一次。

### Topics

**Topics** 中结构化并持久地存储事件。topic 就像是文件系统中的目录，Event 就是目录下的文件。Kafka 中的 Topic 总是关联多个生产者和多个消费者：比如 0 个、1 个、或者多个生产者，消费者也是一样。

（这里提到 Topic，也许我们会想到传统消息系统中的主题，不过 Kafka 中的主题有些不太一样，因为我们可以根据需要一次或者多次阅读 Topic 中的 Event，传统消息系统中消息被消费后就删除了。）

我们可以为 Kafka 中每个 Topic 做配置，定义 Topic 保存 Event 的最长时间，超过了这个时间，Event 就会被丢弃。Kafka 的性能和数据的大小有关，因此长时间存储大量数据是完全没问题的。

### Partitions

**Partitioned**：Topics 是分区的（**partitioned**），这就意味着一个 Topic 将被分成几块区域，每个区域有可能分散到不同的 Kafka broker 中。数据的分布式布局对可伸缩性非常重要，因为它允许客户端程序同时从/向多个代理读取和写入数据。

当一个事件发布到 Topic 中，它实际上是被分配到 Topic 的某个 partition 中。如果是具有相同 key 的事件（比如说 key 是某个汽车制造商的 ID）将会被写入到相同的 partition 中。并且 Kafka 保证给定 topic-patition 的消费者将始终以和写入事件完全相同的数据读取分区的事件 [guarantees](https://kafka.apache.org/documentation/#semantics)。

### Replications

**Replicated**：为了确保数据具备容错性和高可用性，每个 Topic 都可以被复制（即使是跨区域、跨多个机房），这样就总会有多个 brokers 拥有相同的数据副本，以防出现错误、维护 broker 或者其他情况。

在生产环境中常规的配置是将复制体设置为 3 个，这就意味者有 3 份备份的数据 + 一个原始数据。同时复制是发生在 Topic 的 Partition 级别上的。

到此为止，我们就了解了 Kafka 中通用的概念和术语，如果想更深入的了解 Kakfa 的设计，可以阅读文档中关于 Design 的部分：

- https://kafka.apache.org/documentation/#design

## Kafka APIs

In addition to command line tooling for management and administration tasks, Kafka has five core APIs for Java and Scala:

- The [Admin API](https://kafka.apache.org/documentation.html#adminapi) to manage and inspect topics, brokers, and other Kafka objects.
- The [Producer API](https://kafka.apache.org/documentation.html#producerapi) to publish (write) a stream of events to one or more Kafka topics.
- The [Consumer API](https://kafka.apache.org/documentation.html#consumerapi) to subscribe to (read) one or more topics and to process the stream of events produced to them.
- The [Kafka Streams API](https://kafka.apache.org/documentation/streams) to implement stream processing applications and microservices. It provides higher-level functions to process event streams, including transformations, stateful operations like aggregations and joins, windowing, processing based on event-time, and more. Input is read from one or more topics in order to generate output to one or more topics, effectively transforming the input streams to output streams.
- The [Kafka Connect API](https://kafka.apache.org/documentation.html#connect) to build and run reusable data import/export connectors that consume (read) or produce (write) streams of events from and to external systems and applications so they can integrate with Kafka. For example, a connector to a relational database like PostgreSQL might capture every change to a set of tables. However, in practice, you typically don't need to implement your own connectors because the Kafka community already provides hundreds of ready-to-use connectors.



