---
title: Kafka Deployment And Quick Start
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425110625.jpg'
coverImg: /img/20220425110625.jpg
cover: false
toc: true
mathjax: false
date: 2022-12-17 23:40:12
summary: "Linux 部署 Kafka 环境以及 Quick Start"
categories:
 - Kafka
 - [Linux, Kafka]
keywords: [Linux, Kafka]
tags: [Linux, Kafka]
---

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

## Use Cases

下面是 Kafka 的一些经典使用场景：

### （1）Messaging

Kafka 可以用来替代传统的消息代理，比如 ActiveMQ、RabbitMQ。

与大多数消息传递系统相比，Kafka 具有更好的吞吐量、内置分区、结点复制、高容错，这些特性使得 Kafka 称为大规模消息处理程序的一个很好的解决方案。

### （2）Website Activity Tracking

Kafka 最初的一个应用方向就是跟踪网站用户的活动，在该活动捕捉流的基础上构建一组实时的发布/订阅机制。

这意味着网站的活动（page views、searches or other actions user may take）可以发布到中央主题中，并且每一种活动对应一个 topic。然后订阅这些 topic 的系统就可以做各种处理。

### （3）Metrics

Kafka 也可以操作监控数据，用来生成统计数据。

### （4）Log Aggregation

许多开发者将 Kafka 作为日志聚合解决方案的替代品。Log Aggregation 操作通常指从服务器上收集物理日志文件，将它们放置某个中心位置，比如文件服务器或者 HDFS，从而做进一步处理。

### （5）Stream Processing

Kafka 的很多用户将数据处理管道（processing pipeline）中的数据分为多个阶段进行处理。首先从 Kafka Topic 中获取原始输入数据，然后做进一步操作，比如 aggregated、enriched 或者转换为新的 Topic 供后续的处理。

比如说存在这样一种 processing pipeline：

- 处理推荐新闻的处理管道首先可能会从 RSS 中抓取文章内容，并将其发布到 "articles" topic；
- 进一步的处理可能是规范化获取的文章信息，并将处理后的文章内容发布到新的主题中；
- 最终阶段可能就算尝试向用户推荐这些内容；

上面的这种处理管道会基于单个 topic 创建实时的数据流图。

针对此种场景，Kafka 提供了一种轻量级的强大的流处理库，叫做 [Kafka Streams](https://kafka.apache.org/documentation/streams)，它处理数据的方式和上面的例子流程差不多。除了 Kafka Streams 之外，其他的开源的流处理工具有 [Apache Storm](https://storm.apache.org/)、和 [Apache Samza](https://samza.apache.org/)。

### （6）Event Sourcing

[Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) 是一种应用程序设计风格，其中状态更改被记录为按照时间排序的记录序列。Kafka 支持大量日志数据存储的特性使得它非常适合作为这个设计风格的应用程序的后端。

### （7）Commit Log

Kafka 可以作为分布式系统的外部 commit-log 工具。这种日志有助于分布式系统不同节点之间的数据同步，并且在节点发生故障时也可以用来恢复节点。

Kafka 的 [日志压缩](https://kafka.apache.org/documentation.html#compaction) 特性非常适合这种应用场景。这种用法也类似于 [Apache BookKeeper](https://bookkeeper.apache.org/) 项目。

# Quick Start

首先需要下载最新的稳定版本的 Kafka。

Kafka 下载地址：https://kafka.apache.org/downloads

其次 Kafka 需要 Java 8+ 的环境，启动 Kafka 则需要结合 ZooKeeper 或者 KRaft（Apache Kafka Raft）。

- Kafka 将元数据托管给 ZooKeeper；
- 当 Kafka 3.3.1 版本公布后，Apache 为了解耦 Kafka 和 ZooKeeper，提供了一种一致性协议 Kafka Raft，当然还是预览版本，Kafka 结合 KRaft 就可以自己管理元数据了。
  - https://www.infoq.com/news/2022/10/apache-kafka-kraft/
  - https://developer.confluent.io/learn/kraft/

本文我们以 Kafka 结合 ZooKeeper 为例，所以还需要下载 LST 版本的 ZooKeeper。

ZK 下载地址：https://zookeeper.apache.org/releases.html

ZK 学习文章：https://naivekyo.github.io/2022/04/28/centos7-deploy-zookeeper-server

## （1）下载 ZooKeeper 和 Kafka

```bash
# 解压
tar -zxf apache-zookeeper-3.7.1-bin.tar.gz
tar -zxf kafka_2.13-3.3.1.tgz

# 修改 zk 权限
chown root:root -R apache-zookeeper-3.7.1-bin/
```

## （2）ZK 启动 Standalone 模式

默认的配置文件例子在：apache-zookeeper-3.7.1-bin/conf/zoo_sample.cfg

先把默认的配置文件名字改了，然后创建一个新的简单的配置文件：

```bash
cd ./conf
# 先备份默认的样例配置文件
mv ./zoo_sample.cfg ./zoo_sample.cfg_bak
# 然后创建新的，注意 ZK 对配置文件只有后缀要求, 名字可以随意，这里为了方便所以叫 zoo.cfg
vim zoo.cfg

# 文件内容
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

属性说明：

- tickTime：ZooKeeper 将毫秒作为基本的时间单位。该属性决定默认的心跳时间周期，而会话超时的最小时间是 tickTime 的两倍；
- dataDir：存储内存中数据库快照的目录，如果没有特别说明，这个数据库将会记录数据更新的事务日志；
- clientPort：监听客户端连接的端口号。

注意配置信息中的 dataDir 属性，配置样例中说这个目录是存放 ZK 快照数据的，根据 FHS，我们可以将其放到 /var/lib 下面，并且新建一个 ZK 专有的目录（需要手动创建这个目录）。

tip：/var 是记录 Linux 程序运行时产生的数据的，而 Linux 硬件即文件，即使是内存也是可以直接以文件方式访问的。/var 目录可以使用内存模拟磁盘，速度快一些。

配置文件创建好了，就可以以单机模式启动 ZK 了：

```bash
# 启动 Server, 默认 standalone mode
bin/zkServer.sh start
# 启动 Client 连接 Server
bin/zkCli.sh -server 127.0.0.1:2181
```

连接成功后，就可以进入 ZK 的 shell 环境了。之前也了解过，ZK 存储结点的数据结构类似于文件树，树的根结点就叫做 zookeeper。

更多操作参考官方文档。

## （3）启动 Kafka

Kafka 压缩文件解压后，可以自行了解具体的目录含义。

下面首先关掉前面启动的 ZK 服务：

```bash
./apache-zookeeper-3.7.1-bin/bin/zkServer.sh stop
```

因为 Kafka 的启动脚本中已经包含了启动 ZK 的指令，并且也提供有默认的 ZK 配置文件：

```bash
# 为了方便演示，这里仅仅改动 dataDir 属性
vim /usr/local/software/kafka_2.13-3.3.1/config/zookeeper.properties

# 修改一行就可以了 
dataDir=/var/lib/zookeeper
```

启动 ZK：

```bash
# 启动
/kafka_2.13-3.3.1/bin/zookeeper-server-start.sh ../config/zookeeper.properties

# 挂后台
ctrl + z
bg %1
jobs
```

启动 Kafka：

```bash
# 启动
/kafka_2.13-3.3.1/kafka-server-start.sh ../config/server.properties

# 挂后台
ctrl + z
bg %2
jobs
```

如果这两个服务都正常启动了，那么我们的 Kafka 单机运行环境算是准备成功了。

## （4）创建存储 Event 的 Topic

Kafka 是分布式的事件流处理平台，它允许你跨越多台机器进行读、写、存储以及处理 [Event](https://kafka.apache.org/documentation/#messages)（也可以叫做 records 或者 messages）。

比如说这些 Event：交易订单、移动设备的地理位置、运输订单、物联网设备或者医疗设备的传感器测量数据等等。这些事件将被组织起来发送到不同的 [Topics](https://kafka.apache.org/documentation/#intro_concepts_and_terms) 中。一个非常形象的比喻：Topic 就类似于文件系统中的目录，Events 就是目录中的文件。

所以如果要写入 Event 就得先创建 Topic。

进入 kafka 的 bin 目录，输入以下指令：

```bash
# 创建一个 topic
[root@localhost bin]# ./kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
Created topic quickstart-events.
```

Kafka 提供的的 bin 目录下的所有命令行工具都有一些附加参数：直接输入脚本的名称比如 kafka-topics.sh，就可以看到提示信息，包含所有的参数。

比如说使用以下参数可以查看指定的 Topic 的信息：

```bash
[root@localhost bin]# ./kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
Topic: quickstart-events        TopicId: LreFQyR3SNKB3hlfdQR1Tw PartitionCount: 1       ReplicationFactor: 1    Configs:
        Topic: quickstart-events        Partition: 0    Leader: 0       Replicas: 0     Isr: 0
```

## （5）向 Topic 中写入 Event

Kafka 的 clients 通过网络向 Kafka brokers 中读/写 events，brokers 一旦接收到 event，就会以持久且容错的方式存储事件，直到我们需要处理这些事件为止，甚至可以永远存储下去。

下面运行 kafka 的 producer client 去写一些事件到 broker 中：

```bash
[root@localhost bin]# ./kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
>This is my first event
>This is my second event
>^C
[root@localhost bin]#
```

输入命令后，每行数据都是一个 event，按下 Ctrl + C 退出。



## （6）读取 events

使用 kafka 的 consumer client 去读取 events：

```bash
[root@localhost bin]# ./kafka-console-consumer.sh --topic quickstart-events --from-beginning  --bootstrap-server localhost:9092
This is my first event
This is my second event
^CProcessed a total of 2 messages
[root@localhost bin]#
```

这个例子中我们使用 --from-beginning 读取指定 topic 中的所有 event，当然也可以指定 offset，更多信息可以直接输入脚本 kafka-console-consumer.sh 查看可用参数。

一样也是按下 Ctrl + C 退出。

跟多尝试：开两个 terminal，一个 kafka producer、一个 kafka consumer，然后实时写 event，就可以看到 consumer 端实时的显示出刚写入的 event。

因为 Events 在 Kafka 中是持久存储的，所以 Events 可以被任意多个消费端在任意时间点读取。

## （7）使用 Kafka Connect 以事件流方式导入/导出数据

你也许会有大量的数据在关系型数据库或者消息系统中，同时还有很多应用程序在使用这些数据。[Kafka Connect](https://kafka.apache.org/documentation/#connect) 允许你持续不断地从外部系统中抽取数据传递给 Kafka，反之亦然。

Kakfa Connect 是可以扩展的，这就意味着你可以自定义 Connect 的实现用于连接自己的数据应用，从而将数据传递个给 Kafka。同时 Kakfa 社区也提供了很多已经实现的 Connect。

本文将展示最简单的一种用法：从文件中读取数据然后通过 Connect 将数据传递给 Kafka，或者从 Kafka 中导出数据到文件。

首先我们需要修改 connect 的配置文件，为它添加一个 jar 包，更多信息可以参考 [Kafka Configuration](https://kafka.apache.org/documentation/#configuration) 中关于 Connection 的配置说明，其中有个 [plugin.path](https://kafka.apache.org/documentation/#connectconfigs_plugin.path) 属性用来指定 connect 的插件。

虽然官方说这里可以用相对路径，但是也是建议我们在生产环境下使用绝对路径会更好，为了养成良好习惯，直接使用绝对路径：

```bash
# 编辑配置文件, 在 kafka 的 config 目录下面
vim /.../kafka_2.13-3.3.1/config/connect-standalone.properties

# 添加或者修改 plugin.path 属性
plugin.path=/usr/local/software/kafka_2.13-3.3.1/libs/connect-file-3.3.1.jar
```

创建测试文件（这个测试文件的名称是有意义的，具体可以查看 /config/connect-file-source.properties 配置文件）：

`echo -e "foo\nbar" > test.txt`

然后我们将以 standalone 模式开启两个 connector，这意味着它俩是作为本机的两个单独的进程启动的。

然后提供三个配置文件作为参数，第一个配置文件是 Kakfa Connect 处理相关的配置，包括一些公共信息比如 Kafka brokers 的连接信息，数据的序列化格式等等。

剩下的配置文件用于创建不同的 connector，这些文件中必须包含一个唯一的连接器名称以及其他配置信息，这样就可以实例化相关的 Connector。

```bash
# 开启两个 Connector 进程
./bin/connect-standalone.sh ./config/connect-standalone.properties ./config/connect-file-source.properties ./config/connect-file-sink.properties
```

然后新开一个 terminal 或者将这个作为后台任务运行。

Kafka 附带的这些样例配置文件使用了前面我们启动 Kakfa 的默认本地集群配置，并创建了两个 Connector 实例。

第一个连接器实例是 source connector，用于从输入文件中按行读取数据然后一条一条的发布到 Kafka Topic 中；

第二个连接器实例是 sink connector，用于从 Kafka Topic 中读取数据然后将它们一行一行的输出到目标文件中。

上面的命令启动后控制台会打印一些日志信息，一旦 Kafka Connect 开始处理数据，source connector 就会从 `test.txt` 文件中读取数据发布到名为 `connect-test` 的 Topic 中（在配置文件中定义的 Topic 名字），同时 sink connector 就会开始从 `connect-test` Topic 中读取数据并将其写入到 `test.sink.txt` 文件中。

我们可以检查以下文件的内容：

```bash
[root@localhost kafka_2.13-3.3.1]# ll
总用量 76
drwxr-xr-x. 3 root root  4096 9月  30 03:06 bin
drwxr-xr-x. 3 root root  4096 12月 17 23:13 config
drwxr-xr-x. 2 root root  8192 12月 17 20:26 libs
-rw-rw-r--. 1 root root 14842 9月  30 03:03 LICENSE
drwxr-xr-x. 2 root root   284 9月  30 03:06 licenses
drwxr-xr-x. 2 root root  4096 12月 17 23:13 logs
-rw-rw-r--. 1 root root 28184 9月  30 03:03 NOTICE
drwxr-xr-x. 2 root root    44 9月  30 03:06 site-docs
-rw-r--r--. 1 root root     8 12月 17 23:13 test.sink.txt
-rw-r--r--. 1 root root     8 12月 17 22:54 test.txt
[root@localhost kafka_2.13-3.3.1]# more test.sink.txt
foo
bar
[root@localhost kafka_2.13-3.3.1]#
```

此时数据已经存储到 Kafka 的对应的 Topic 中了，可以看一下：

```bash
[root@localhost kafka_2.13-3.3.1]# ./bin/kafka-console-consumer.sh --topic connect-test --from-beginning --bootstrap-server localhost:9092
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```

现在刚刚创建的 connectors 还在正常工作，我们可以向源文件写入数据看看有什么效果：

tip：新开一个 terminal 向文件追加写入数据

```bash
[root@localhost kafka_2.13-3.3.1]# echo Anther line >> test.txt
```

然后就可以实时的看到刚刚的消费者终端打印了新写入的数据：

```bash
{"schema":{"type":"string","optional":false},"payload":"Anther line"}
```

## （8）使用 Kafka Streams 处理 Events

一旦数据作为 events 存储到 Kafka 中，我们就可以通过 [Kafka Streams](https://kafka.apache.org/documentation/streams) 结合 Java/Scala 客户端来处理这些数据

The library supports exactly-once processing, stateful operations and aggregations, windowing, joins, processing based on event-time, and much more.

The [Kafka Streams demo](https://kafka.apache.org/documentation/streams/quickstart) and the [app development tutorial](https://kafka.apache.org/documentation/streams/tutorial) demonstrate how to code and run such a streaming application from start to finish.



## （9）关闭 Kafka 环境

最后我们看一下如何关闭 Kafka：

- 关闭正在运行的 producer 和 consumer，只需要输入 Ctrl + C 就可以了（如果是后台任务就先切到前台，然后关闭）；
- 关闭 broker 也是和上面一样，Ctrl + C；
- 最后，如果第一步开启的 Zookeeper 和 Kafak 正在前台运行，输入 Ctrl + C 就可以了。

如果 Kafka 相关的进程在后台运行且不方便转到前台进行关闭，可以使用：

ZK 和 Kafka 可以使用下面两个命令：

```bash
[root@localhost kafka_2.13-3.3.1]# ll ./bin/ | grep stop
-rwxrwxr-x. 1 root root  1361 9月  30 03:03 kafka-server-stop.sh
-rwxrwxr-x. 1 root root  1366 9月  30 03:03 zookeeper-server-stop.sh
```

其他的可以根据 PID kill 掉。

## 总结

关于 Kafka 的更多信息可以参考文档或者查阅资料、书籍。