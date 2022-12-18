---
title: Spring Boot Integrate Kafka
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425110658.jpg'
coverImg: /img/20220425110658.jpg
cover: false
toc: true
mathjax: false
date: 2022-12-18 23:04:20
summary: "Spring Boot 集成 Kafka"
categories: "Spring Boot"
keywords: ["Spring Boot", "Kafka"]
tags: ["Spring Boot", "Kafka"]
---

# Spring Boot Integration Kafka

之前已经了解过如何在 [Linux 中部署 Kafka 环境](https://naivekyo.github.io/2022/12/17/kafka-deployment-and-quick-start/)了，现在看一下如何编写 Java 客户端集成 Kafka。

项目环境：

- JDK 8+；
- Kafka_2.13-3.3.1（虚拟机）
- Spring Boot 2.7.6

这里需要注意版本依赖关系：

参考：

- https://docs.spring.io/spring-boot/docs/2.7.6/reference/html/getting-started.html#getting-started
- https://spring.io/projects/spring-kafka
- https://docs.spring.io/spring-kafka/docs/2.8.11/reference/html/

`Spring Boot 2.7.x  -> Spring 5.5.x -> Spring-Kafka 2.8.x -> Kafka Clients 3.0.x`

参考文章：

- https://docs.spring.io/spring-boot/docs/2.7.6/reference/html/messaging.html#messaging.kafka
  - 里面包含 Apache Kafka 在 Spring 项目中的常用操作：充当消息代理工具、Kafka Streams 等等
- https://spring.io/projects/spring-kafka
  - 将 Spring 的核心概念结合 Apache Kafka 的消息代理机制提供的一种消息传递解决方案；
  - 核心实现类似 [Spring 对 JMS 的支持](https://docs.spring.io/spring-framework/docs/5.2.22.RELEASE/spring-framework-reference/integration.html#jms)

# Preparation

首先在虚拟机中[搭建 Kafka 环境](https://naivekyo.github.io/2022/12/17/kafka-deployment-and-quick-start/)，注意开放 Kafka 端口 9092，且修改 Kafka 配置文件中关于 Socket 的配置，比如 listeners 属性，因为我们要从外部连接服务器上的 Kafka broker，最后本机创建 Spring Boot 项目，引入 Kafka 相关依赖：

```xml
<!-- spring integration kafka -->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.8.11</version>
</dependency>
```

当然这里的版本号也可以不写，因为是 Spring Boot 项目，父工程已经定义好了对应的版本号。

# Quick Start

由于我们使用了 Spring Boot，默认会提供两个 Kafka 的自动装配类

- `org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration`
- `org.springframework.boot.autoconfigure.kafka.KafkaAnnotationDrivenConfiguration`

只需在配置文件中定义好属性，就可以自动向容器中注入相关的 Bean，当然也可以选择使用自定义的配置类。

下面看几个概念：

- `org.springframework.kafka.core.KafkaAdmin`：Spring 代理了 `org.apache.kafka.clients.admin.AdminClient` 用于操作 Topic 并将其注入 Spring 容器上下文；
- `org.springframework.kafka.core.ProducerFactory`：生产 `org.apache.kafka.clients.producer.Producer` 实例的策略工厂，Producer 用于发送消息；
- `org.springframework.kafka.core.ConsumerFactory`：生产 `org.apache.kafka.clients.consumer.Consumer` 实例的策略工厂，Consumer 用于接收消息；
- `org.springframework.kafka.core.KafkaTemplate`：Spring 特有的 Template 设计理念在集成 Kafka 时的应用，封装了一些便捷的方法。

- `org.springframework.kafka.listener.MessageListenerContainer`：Spring 专门为 Kafka 提供的 ListenerContainer 抽象接口，仅供内部使用，有两个实现：
  - `KafkaMessageListenerContainer`：单线程接收所有 Topic 或所有 Partition 传递过来的消息； 
  - `ConcurrentMessageListenerContainer`：代理至少一个 KafkaMessageListenerContainer 实例来提供多线程的消息接收服务。
- `@KafkaListener`：Spring 提供的注解，用于将指定的方法声明为某个 ListenerContainer 的 Listener，消息的具体消费逻辑就是在这里。同时 Spring 还提供了 `MessagingMessageListenerAdapter` 适配器类为 Listener 提供了一系列增强服务，比如消息转换机制等等。

如果我们不做任何定制操作，Spring Boot 会根据配置属性自动装配一些 Bean，比如 KafkaTemplate、DefaultKafkaConsumerFactory、DefaultKafkaProducerFactory、KafkaAdmin、ConcurrentKafkaListenerContainerFactory。

## Config Properties

配置文件信息：

```yml
server:
  port: 9002

spring:
  jackson:
    date-format: "yyyy-MM-dd HH:mm:ss"
    time-zone: GMT+8
    locale: zh_CN
    
  kafka:
    bootstrap-servers: 192.168.154.3:9092
    consumer:
      group-id: testGroup
```

更多配置信息可以参考 `org.springframework.boot.autoconfigure.kafka.KafkaProperties` 以及 Kafka 官方文档。

这里的 Kafka 服务已经以 standalone mode 在虚拟机中运行了。

## Example

无需额外配置，直接编写测试类，如果要进行定制，请参考 Spring 以及 Kafka 官网相关信息。

```java
@RestController
public class TestController {

    @Resource
    private KafkaTemplate<String, String> kafkaTemplate;
    
    private static final String TEST_TOPIC = "spring-boot-test-topic";
    
    @GetMapping("/send")
    public String hello() {
        int random = ThreadLocalRandom.current().nextInt(1, 100) + 1;
        this.kafkaTemplate.send(TEST_TOPIC, "Random number: " + random);
        
        return random + "";
    }
    
    @KafkaListener(topics = TEST_TOPIC)
    public void processMessage(String content) {
        System.out.printf("receive message from topic: %s;\tcontent: %s\n", TEST_TOPIC, content);
    }
    
}
```

这里没有自己构建 Topic Bean，而是使用 `@KafkaListener` 注解，注解属性中指定 Topic 名称，如果 Kafka 服务中没有该 Topic 则会自动创建，且使用默认的 Topic 配置。

查看服务器 Topic 信息：

```bash
[root@localhost bin]# ./kafka-topics.sh --bootstrap-server 192.168.154.3:9092 --list
__consumer_offsets
connect-test
quickstart-events
spring-boot-test-topic
[root@localhost bin]#
```

可以看到已经有我们创建的测试用的 Topic 了，且注意这里的 bootstrap-server 要和我们在配置文件 server.properties 中监听的一致。

看一下消费者的 group 信息：

```bash
[root@localhost bin]# ./kafka-consumer-groups.sh --list --bootstrap-server 192.168.154.3:9092
testGroup
console-consumer-93312
connect-local-file-sink
[root@localhost bin]#
```

也有 testGroup 了。

最后调用几次请求，看看 Topic 中的数据：

```bash
[root@localhost bin]# ./kafka-console-consumer.sh --bootstrap-server 192.168.154.3:9092 --topic spring-boot-test-topic --from-beginning
Random number: 8
Random number: 7
Random number: 19
Random number: 84
Random number: 45
Random number: 92
Random number: 20
Random number: 59
Random number: 19
Random number: 21
Random number: 74
Random number: 70
Random number: 12
Random number: 89
Random number: 66
Random number: 45
Random number: 71
```

这次仅仅演示了用 Kafka 替代原有项目中的消息队列的例子，如果要使用 Kafka Stream 或其他特性，可以参考相关文档。