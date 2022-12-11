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

