---
title: Spring Boot Integate Redis Pub/Sub Feature
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425111059.jpg'
coverImg: /img/20220425111059.jpg
cover: false
toc: true
mathjax: false
date: 2023-03-01 23:23:18
summary: "Spring Boot 集成 Redis 发布订阅功能"
categories: "Redis"
keywords: ["Redis", "Spring Boot"]
tags: ["Redis", "Spring Boot"]
---

# Prerequirement

- Docker 环境；

参考网址：

Redis 官网关于容器化的 get started：https://redis.io/docs/stack/get-started/install/docker/

注意，这里使用的是 Redis-stack（含有 redis-server 和 redisinsight 服务）；

Linux 启用 Docker 服务后，执行以下命令：

```bash
# 查看可用的 redis-stack 镜像
docker search --no-trunc redis/redis-stack
# 拉取最新的镜像
docker pull redis/redis-stack:latest
# 查看本地镜像
docker images
# 先尝试启动镜像
docker -d -p 6379:6379 --name redis redis/redis-stack:latest
# 查看镜像日志, 以便确定当前 redis 版本
docker logs -f redis
```

确定了版本之后前往  Redis 官网找到对应的配置文件模板：

Redis 各个版本的配置文件 example：https://redis.io/docs/management/config/

下载后，修改配置文件：

```bash
daemonize no; # 无需修改, 因为 docker 启动容器时已经附加了 -d
# bind 127.0.0.1 -::1 # 注释掉, 允许远程连接
# protected-mode no # 改为 no, 允许远程连接
requirepass 123456 # 设置密码访问 redis-server
dir /data/redis # 注意 redis-stack 镜像中 Redis 数据存放的位置是 /data/redis
appendonly yes # 允许数据持久化
```

将配置文件上传到服务器，存放到自己喜欢的位置，同时再某个地方创建一个目录用于存放 Redis 持久化的数据；

启动 redis-stack：

```bash
docker run -d -p 6379:6379 -p 8001:8001 -v /usr/local/docker_data/redis/local_data/:/data -v /usr/local/docker_data/redis/redis-stack.conf:/redis-stack.conf --name redis redis/redis-stack:latest
```

参数说明：

（1）-d 以后台进程运行；

（2）-p 容器和宿主机端口映射；

（3）-v 是 volume 的缩写，docker 中可以为容器绑定 volume，这样容器结束运行后和 volume 绑定的目录或文件会持久化到物理机中；

- `/usr/local/docker_data/redis/local_data/:/data`：Redis 数据持久化的目录，由于 Redis-stack 容器创建了某些环境变量，这里的 /data 指代容器中 redis 数据持久化的目录，将其和物理机的某个路径绑定；
- `/usr/local/docker_data/redis/redis-stack.conf:/redis-stack.conf`：指定容器中 Redis-server 启动时使用的配置文件，/redis-stack.conf 也是 Redis-stack 容器指定的环境变量；

（4）--name redis，为运行的容器指定一个标识符，这里如果不使用 --name，则会默认生成一个随机字符串，使用 docker ps 可以查看相关信息；

注意：使用 RedisDesktopManager 工具连接容器中的 docker 时，本机不要使用代理工具，会提示连接错误。



# Spring Boot Integrate Redis

## 依赖

官网文档：

- [Spring Boot Working with Redis](https://docs.spring.io/spring-boot/docs/2.7.9/reference/html/data.html#data.nosql.redis)
- [Spring Redis Support](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis)

本文使用的 Spring Boot 版本为 2.7.9，Spring-Redis 默认使用 Lettuce 创建 connection，Lettuce 需依赖 apache-common-pool2 依赖：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

直接引入，Spring Boot 已经定义好了对应的版本。

## 定制化

可以根据需要注入 `RedisConnectionFactory` 和 `RedisTemplate`，Spring Boot 默认的 Redis 自动装配类是：`org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration`

```java
@AutoConfiguration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

   @Bean
   @ConditionalOnMissingBean(name = "redisTemplate")
   @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
   public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
      RedisTemplate<Object, Object> template = new RedisTemplate<>();
      template.setConnectionFactory(redisConnectionFactory);
      return template;
   }

   @Bean
   @ConditionalOnMissingBean
   @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
   public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
      return new StringRedisTemplate(redisConnectionFactory);
   }

}
```

可以看到它向容器中注入了两个 RedisTemplate（注：`StringRedisTemplate extends RedisTemplate<String, String>`），并且在类上方的 `@Import` 注解中引入了 Lettuce 或 Jedis 相关依赖（这两个依赖都可以提供连接 Redis 服务的能力）；

一般情况下我们会对 RedisTemplate 进行某个定制化处理，比如序列化操作。

### RedisTemplate

查看 RedisTemplate 源码：

```java
public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
    // 。。。。。。
    private @Nullable RedisSerializer<?> defaultSerializer;
	private @Nullable ClassLoader classLoader;

	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer keySerializer = null;
	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer valueSerializer = null;
	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashKeySerializer = null;
	@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashValueSerializer = null;
	private RedisSerializer<String> stringSerializer = RedisSerializer.string();

	private @Nullable ScriptExecutor<K> scriptExecutor;
    // 。。。。。。
    
    public RedisTemplate() {}
    
    @Override
	public void afterPropertiesSet() {

		super.afterPropertiesSet();

		boolean defaultUsed = false;

		if (defaultSerializer == null) {

			defaultSerializer = new JdkSerializationRedisSerializer(
					classLoader != null ? classLoader : this.getClass().getClassLoader());
		}

		if (enableDefaultSerializer) {

			if (keySerializer == null) {
				keySerializer = defaultSerializer;
				defaultUsed = true;
			}
			if (valueSerializer == null) {
				valueSerializer = defaultSerializer;
				defaultUsed = true;
			}
			if (hashKeySerializer == null) {
				hashKeySerializer = defaultSerializer;
				defaultUsed = true;
			}
			if (hashValueSerializer == null) {
				hashValueSerializer = defaultSerializer;
				defaultUsed = true;
			}
		}

		if (enableDefaultSerializer && defaultUsed) {
			Assert.notNull(defaultSerializer, "default serializer null and not all serializers initialized");
		}

		if (scriptExecutor == null) {
			this.scriptExecutor = new DefaultScriptExecutor<>(this);
		}

		initialized = true;
	}
    // 。。。。。。
}
```

看 `afterPropertiesSet` 方法，可以看到默认的序列化器是使用 JDK 提供的序列化机制（基于两个接口 ObjectInput 和 ObjectOutput），有时候 JDK 序列化并不是一个很好的选择，尤其是在分布式微服务系统中，主要有以下几点原因：

（1）无法跨语言，微服务系统中，不同服务节点可能采用不同的语言开发，而网络数据传输是基于序列化二进制流的；

（2）容易被攻击，对于不信任数据的反序列化，往往是非常危险的，应该尽量避免。Java 序列化机制中并没有提供很好的安全机制；

（3）序列化数据字节流太大，和其他序列化机制相比（比如使用 ByteBuffer、Jackson、Protobuf 等等），JDK 序列化后的字节数量太多；

（4）性能差，和其他序列化机制相比，JDK 序列化速度要慢很多；

因此我们可以针对序列化这一块对 RedisTemplate 进行定制；

### StringRedisTemplate

看自动装配类注入的另一个 Template：

```java
public class StringRedisTemplate extends RedisTemplate<String, String> {

   public StringRedisTemplate() {
      setKeySerializer(RedisSerializer.string());
      setValueSerializer(RedisSerializer.string());
      setHashKeySerializer(RedisSerializer.string());
      setHashValueSerializer(RedisSerializer.string());
   }

   public StringRedisTemplate(RedisConnectionFactory connectionFactory) {
      this();
      setConnectionFactory(connectionFactory);
      afterPropertiesSet();
   }

   protected RedisConnection preProcessConnection(RedisConnection connection, boolean existingConnection) {
      return new DefaultStringRedisConnection(connection);
   }
}
```

这个类就更简单了，继承了 RedisTemplate，然后使用了一些预先定义好的序列化器，可以看到 key-value、hashkey-hashvalue 用的都是 `RedisSerializer.string()`，点进去看看会发现实际用的是 `java.nio.Charset` 对 String 类型的数据进行编码和解码操作；

### 使用 Jackson 序列化机制

下面定制我们自己的使用 Jackson 序列化的 RedisTemplate：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

/**
 * @author NaiveKyo
 * @version 1.0
 * @since 2023/2/26 23:37
 */
@Configuration(proxyBeanMethods = false)
public class RedisConfiguration {
    
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        
        template.setKeySerializer(RedisSerializer.json());
        template.setValueSerializer(RedisSerializer.json());
        template.setHashKeySerializer(RedisSerializer.json());
        template.setHashValueSerializer(RedisSerializer.json());
        
        return template;
    }
    
}
```

由于 Jackson 的配置较为繁琐，还没仔细研究，这里就暂时使用预定义好的利用 Jackson 的序列化器，**RedisSerializer 接口中预先定义的各种序列化器都支持处理 null objects/empty arrays 数据，也建议如果要自定义序列化器一定要考虑处理这种情况，因为 Redis 不会接收 null key 或 value，但是当 key 不存在时却会返回 null。**

> 补充：JSR 107

从 Spring 3.1 开始提供缓存支持，到 Spring 4.1 提供对 JSR 107 中定义的缓存注解提供了支持以及更多的定制化操作：

- https://docs.spring.io/spring-framework/docs/5.2.22.RELEASE/spring-framework-reference/integration.html#cache
- https://jcp.org/en/jsr/detail?id=107



## Spring Redis Pub/Sub

### Introduction

官网：https://spring.io/projects/spring-data-redis

文档：https://docs.spring.io/spring-data/redis/docs/2.7.7/reference/html/#reference

pub/sub：https://docs.spring.io/spring-data/redis/docs/2.7.7/reference/html/#pubsub

Spring Data 项目为 Redis 提供了专门的 Spring 消息集成，和之前学习的 Spring JMS 集成类似。

先简单了解一下 Redis 的 Pub/Sub：https://redis.io/docs/manual/pubsub/

注意 Redis pub/sub 和 key space（database & scoping）无关，它不会干扰任何数据库，这就意味着在 db0 中 publish，在 db10 中也可以 subscribe。并且这些数据也不会被持久化。发布的消息只会消费一次。

> 发布消息

发布消息很简单，两种调用形式：

- `org.springframework.data.redis.core.RedisTemplate#convertAndSend` 方法；
- `org.springframework.data.redis.connection.RedisConnection#publish`  方法；

> 接收消息

接收消息也有两种方式：

（1）在低层次上，调用 RedisConnection 的 subscribe 和 pSubscribe 方法，它们是 Redis 相关命令在 Spring 中的实现，可以通过 channel 或 pattern 订阅；

**但是需要注意**：在 Spring Data Redis 项目中调用这些方法会阻塞当前线程，一直等待消息的到来，明显不太适合实际环境，好在 Spring Data Redis 为我们提供了另一种方式；

（2）RedisMessageListenerContainer 和 RedisMessageListenerAdapter

**RedisMessageListenerContainer ：**

和集成 JMS 时类似，Spring 也提供了 listener container 和 listener adapter，前者维护了一组接收消息的线程，并将接收到的消息分派给 listener 处理消息。message listener container 是 MDP 和 messaging provider 之间的中间层，负责注册与接收消息、资源获取和释放、异常转换等。这样开发者就只需要关注消息接收和处理逻辑了。

MessageListener 这个接口也比较特殊，它的实现不仅可以监听常规消息，来可以监听一些特殊的消息，比如取消订阅的消息。

此外为了提供性能，RedisMessageListenerContainer 可以让一个 connection 或一个 thread 在多个 listeners 之间共享，这样就不用频繁创建 connection 或 thread 了（资源池的应用）。此外 container 还允许在程序运行时修改其配置，这样就不用重启应用了。

container 为异步消息处理提供了支持，但是需要提供一个 `java.util.concurrent.Executor`（或者 Spring 的 `TaskExecutor`）来分派消息。根据负载、listener 的数量或运行时环境，您应该更改或调整执行程序以更好地满足您的需求。

**RedisMessageListenerAdapter：**

RedisMessageListenerAdapter 是 Spring 异步消息处理的一个重要的组件，简而言之，它允许开发者将任意的 class 作为 MDP（事件驱动的 POJO）使用（尽管有一些限制）。

### Example

下面看例子：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;
import org.springframework.data.redis.serializer.RedisSerializer;

@Slf4j
@Configuration(proxyBeanMethods = false)
public class RedisConfiguration {

    public static final String SPECIAL_CHANNEL_NAME = "my_test_channel";
    
    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        
        template.setKeySerializer(RedisSerializer.json());
        template.setValueSerializer(RedisSerializer.json());
        template.setHashKeySerializer(RedisSerializer.json());
        template.setHashValueSerializer(RedisSerializer.json());
        
        return template;
    }


    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(RedisConnectionFactory redisConnectionFactory, MessageListenerAdapter messageListenerAdapter) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        // 设置 RedisConnectionFactory
        container.setConnectionFactory(redisConnectionFactory);
        // 注册一个 map: MessageListener -> Channel/Pattern
        container.addMessageListener(messageListenerAdapter, ChannelTopic.of(SPECIAL_CHANNEL_NAME));
        return container;
    }

    @Bean
    public MessageListenerAdapter messageListenerAdapter() {
        MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageProcessor(), "handleMessage");
        adapter.setSerializer(RedisSerializer.json());
        return adapter;
    }

    static class MessageProcessor {
        public void handleMessage(String message) {
            // do something ...
            log.info("handle received message...");
        }
    }
}
```

这一次我们向容器中注入了两个 Bean：

（1）`RedisMessageListenerContainer`；

从 RedisConnection 处接收 subscribe channel 的 message 并分发给内部注册的 MessageListener，这里我们向 container 中注入了 `messageListenerAdapter` 和 `ChannelTopic.of(SPECIAL_CHANNEL_NAME)` 这一种处理关系；

（2）`MessageListenerAdapter`；

实现了 `MessageListener` 接口，同时其内部持有 delegate 对象，通过反射调用 delegate 的特定方法来处理，这个 delegate 可以是其他对象也可以是 adapter 本身；

本例中指定该 adapter 的序列化工具是 `RedisSerializer.json()`，和定制 RedisTemplate 时一致，都是 Jackson，这样消息的发送和接收都能顺利执行；

本例中我们定义了 `MessageProcessor` 作为消息的处理器，adapter 对 container 传递过来的消息做反序列化处理，然后反射调用该处理器的特定方法来处理消息；

更多信息请参见相关类的源码注释；

## 测试

Spring 配置：application.yml

```yml
server:
  port: 10000
  
spring:
  mvc:
    servlet:
      load-on-startup: 1

  redis:
    password: 123456
    port: 6379
    host: 192.168.154.3
    database: 0
```

TestController：

```java
@RestController
public class TestController {
    
    @Resource
    private RedisTemplate<String, String> redisTemplate;
    
    @GetMapping("/hello")
    public String sayHello() {
        return "hello world";
    }
    
    @GetMapping("/publish-message")
    public String publishMessage() {
        this.redisTemplate.convertAndSend(RedisConfiguration.SPECIAL_CHANNEL_NAME, "hello world!");
        return "ok";
    }
    
}
```

注意：这里用的是 `@Resource` 注解，先根据 bean 名称注入，找不到再根据类型，最终注入的就是我们定制的 `RedisTemplate`；

如果使用 `@Autowire` 注解，导入的就可能是 `StringRedisTemplate` 实例了，因为这里做了编译期泛型擦除，运行时类型和 `StringRedisTemplate` 一致，会影响消息的序列化和反序列化。

运行 SpringBoot 服务，浏览器访问：`http://localhost:10000/publish-message`

程序打印：

```
2023-03-01 23:02:26.953  INFO 8004 --- [enerContainer-1] i.n.r.config.RedisConfiguration          : handle received message...
```

