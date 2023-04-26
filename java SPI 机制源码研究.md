# Java SPI And Proxy

主要学习 JDK 提供的服务发现机制以及代理机制；

# Java SPI

## 参考

官方：

- Java api doc：https://docs.oracle.com/javase/8/docs/api/

- 类：`java.util.ServiceLoader` 源码

 博客：

- https://pedrorijo.com/blog/java-service-loader/

## 简介

JDK 在 `java.util` 包中提供了一个简单的 service-provider 工具类：ServiceLoader。

它的一个经典使用场景是 JBDC 在运行时自动查找并注册可以使用的驱动。

查阅源码注释：`service` 可以称之为一组 interfaces 或者 abstract classes，而一个 `service provider` 则通常是 `service` 的一种实现。service provider 中的 classes 通常实现 service 本身定义的 interfaces 或者继承了 abstract classes。

同时 service providers 可以以 extensions 的形式安装到 Java 平台的实现中，也就是将对应的 jar 包放到特定的扩展目录中就可以了。

出于类加载的目的，一个 service 通常表示一种特定的类型，比如一个 interface 或者一个 abstract class（也可以使用一个具体的类，但是不建议这么做）。而 service 的 provider 通常包含一个或多个具体的类（concrete classes），这些类使用特定的数据和代码扩展这种 `service type`。	

provider 类通常是一个 proxy，它包含足够的信息来决定提供者是否能够满足特定的请求，同时包含了按需创建实际提供者的代码。provider classes 的实现细节与特定的 service 高度相关，因此没有一个类或者接口可以统一的表示它们，所以 Java 没有定义这样的类型，Java 对它们唯一的要求就是要有一个没有参数的构造器，这样在类加载阶段就可以实例化这个 service provider class。

每一个 service provider 有一个特定的标识符，它定义在 `provider-configuration` 文件中，Java 平台要求这个文件在 `META-INFO/services` 资源目录下，文件的名字必须是 `service's type` 接口或类的全限定名（jsl 将其定义为 `binary name`）。该文件中包含了一系列具体 provider classes 的 binary name，一个类占一行，注释用 `'#'` 开头，整个文件的编码方式是 `UTF-8`；

如果一个特定的 service provider class 的 binary name 存在多个 configuration 文件中，或者在一个 configuration 文件中出现了多次，重复的名字会被忽略。

Providers 的 located 和 instantiated 是 lazily 的，是按需加载和实例化的。service loader 程序维护了一个缓存用来存储到目前为止已经加载的 providers。每一次调用 `java.util.ServiceLoader#iterator`  方法就会返回一个迭代器对象，这个迭代器中包含了 provider cache 中的所有元素，且按照实例化的顺序。然后，它就会惰性加载并实例化任何剩余的 providers 同时按照顺序将它们添加到缓存中。

如果想要清空 provider cache，只需要调用 `java.util.ServiceLoader#reload` 方法即可，它会清空 cache，如果此时再次调用 iterator 方法，就会立即惰性加载和实例化可以找到的 provider，reload 方法的使用场景就是在 Java Runtime 环境中，如果添加了其他的 provider 到类路径下，此时需要将其加载进内存，只需要调用 reload 方法，然后如果再次触发了 iterator 方法的调用就能够将我们新提供的 provider 加载进缓存中。

注意由于涉及到类加载，所以 service loader 程序的执行上下文一定是在 security context 中。且受信任的系统代码应该在 privileged security context 中调用此类中的方法；

由于 ServiceLoader 不是线程安全的，所以不推荐在多线程环境下使用。

除非特别声明，否则在调用 ServiceLoader 类的方法时传递 null 参数，一般都会报空指针异常。

Usage Note：如果用于加载服务的 classloader 所处的类路径下包含网络 URL，那么在搜索 provider-configuration file 时 URL 会被忽略。

## 测试用例及源码分析

看如下测试用例代码：

service interface：

```java
public interface MyServiceType {
    void greeting();
}
```

service provider 1：

```java
public class MyServiceProvider implements MyServiceType {
    
    static {
        // do something...
        System.out.println("MyServiceProvider has been instantiated.");
    }
    
    @Override
    public void greeting() {
        System.out.println("hello, i'm customize service provider.");
    }
}
```

service provider 2：

```java
public class MyServiceProvider2 implements MyServiceType {

    static {
        // do something...
        System.out.println("MyServiceProvider2 has been instantiated.");
    }
    
    @Override
    public void greeting() {
        System.out.println("hello, i'm customize service type provider 2.");
    }
}
```

`/META-INF/services/io.naivekyo.spi.type.MyServiceType`

```
io.naivekyo.spi.provider.MyServiceProvider
io.naivekyo.spi.provider.MyServiceProvider2
```

test code：

（这里演示的 provider 比较简单，只提供了简单的 static 代码块和具体实现方法，现实中的场景是一个 service provider 类是一系列类的主入口，我们希望在类的实例化的时候做很多操作，所以下面的测试代码仅仅是在 security context 中加载类并进行类的实例化，并没有将具体的对象收集起来，关于类的实例化，参见 [Section 12.4 of The Java Language Specification.](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html)）

```java
@Test
void test() {
    // 从两个地方获取 service provider 的特定实现类 binary name
    // (1) System property 'my.service.impl'
    // (2) 使用 ServiceLoader mechanism
    String systemProperty = null;
    try {
        systemProperty = AccessController.doPrivileged(new PrivilegedAction<String>() {
            @Override
            public String run() {
                return System.getProperty("my.service.impl");
            }
        });
    } catch (Exception e) {
        systemProperty = null;
    }

    if (systemProperty != null && !"".equals(systemProperty)) {
        String[] split = systemProperty.split(",");
        for (String s : split) {
            try {
                Class.forName(s, true, ClassLoader.getSystemClassLoader());
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
    }

    AccessController.doPrivileged(new PrivilegedAction<Void>() {
        @Override
        public Void run() {
            ServiceLoader<MyServiceType> loadedProviders = ServiceLoader.load(MyServiceType.class);
            Iterator<MyServiceType> providersIterator = loadedProviders.iterator();
            try {
                while (providersIterator.hasNext()) {
                    providersIterator.next();
                }
            } catch (Exception e) {
                // Do nothing
            }
            return null;
        }
    });
}
```

控制台输出：

```
MyServiceProvider has been instantiated.
MyServiceProvider2 has been instantiated.
```

使用 ServiceLoader 的核心代码是：

```java
Iterator<MyServiceType> providersIterator = loadedProviders.iterator();
try {
    while (providersIterator.hasNext()) {
        providersIterator.next();
    }
} catch (Exception e) {
    // Do nothing
}
```

debug 一下会发现核心点在于 `java.util.ServiceLoader.LazyIterator`，而不是 `java.util.ServiceLoader#providers`，对于 service provider 是惰性加载的，调用了 `load` 方法只是创建了一些对象，正在有用的是调用 `hasNext` 和 `next` 方法，这两个方法主要是通过 Java 的资源服务读取 `/META-INF/services/io.naivekyo.spi.type.MyServiceType` 文件，按行读取其中的服务实现类的 binary name，做相关处理后最终执行类加载：

```java
// java.util.ServiceLoader.LazyIterator#nextService
c = Class.forName(cn, false, loader);

// 实例化并缓存起来
S p = service.cast(c.newInstance());
providers.put(cn, p);
```

注意上面我们的两种方式，如果是根据系统属性实例化的对应这行代码：

```java
Class.forName(s, true, ClassLoader.getSystemClassLoader());
```

根据实际需要做不同处理；

更多信息参考 `ServiceLoader` 源码；