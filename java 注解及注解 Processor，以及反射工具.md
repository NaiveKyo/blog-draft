# 一、简介

参考：

- https://docs.oracle.com/javase/tutorial/java/index.html
- https://docs.oracle.com/javase/tutorial/java/annotations/index.html

在 Java 中，注解（Annotation）是元数据的一种，主要用来给编译器提供一些信息，使用注解可以让编程更加高效。

注解主要有以下几个用途：

- 为编译器提供一些信息：编译器利用注解信息可以检测错误和抑制编译警告；
- 在编译时期或部署时期进行某些操作：编写某些工具软件可以利用注解中的信息生成代码、xml 文件等等；
- 运行时处理：在运行时可以根据注解上的信息做一些操作。

# 二、注解基础知识

## 1、Annotation's Format

先看一个简单的例子：

```java
@Documented
@Target(value = ElementType.TYPE)
@Retention(value = RetentionPolicy.RUNTIME)
public @interface TestAnnotation1 {
    
    String value();

    String name() default "";
    
    String desc() default "";
    
}
```

这是一个自定义注解，可以看到它由以下几部分组成：

- 使用 `@interface` 声明这是一个注解；
- 注解上面可以标注其他注解，此类注解被称为元注解；
  - 元注解描述了当前注解的某些特征，比如例子中的元注解告诉程序，TestAnnotation1 注解可以用来生成 java-doc，能够标注在类上面，作用范围为运行时。
- 注解可以拥有属性，比如 `String value()`，表示该注解有一个 String 类型的名为 value 的属性，而且 value 这个命名非常特殊；
- 属性可以有默认值，使用 `default` 设置默认值；

通常来讲每个注解都必须有一个属性叫做 `value()`；

下面看看如何使用该注解：

```java
// 不指定其他属性, 默认赋值给 value 属性, 可以看出 value 这个名称是有特殊含义的
@TestAnnotation1("test value")
public class TestClass1 {
}

// 也可以显式指定 value 的值
@TestAnnotation1(value = "test value")
public class TestClass1 {
}

// 显式指定多个属性的值时, 必须编写完整
@TestAnnotation1(value = "test value", name = "test")
public class TestClass1 {
}

@TestAnnotation1(value = "test value", name = "test", desc = "xxx")
public class TestClass1 {
}
```

除了上面的信息外，还可以看出，拥有默认值的属性是可以省略的。

再看看两个特殊的注解：

```java
// 该注解没有任何属性
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation2 {
}

// 类上可以标注多个注解, 且没有属性的注解可以简写
@TestAnnotation2
@TestAnnotation1("test value")
public class TestClass1 {
}
```

下面看看从 JDK 8 新增的可重复注解：

```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(TestAnnotation3s.class)
public @interface TestAnnotation3 {
    
    String value();
}

// 此种注解被称为 Container Annotation Type, 它的 value 值为注解数组
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation3s {
    TestAnnotation3[] value();
}

// 现在就可以在一个类上标注多个相同的注解, 但是它们的 value 不一样
@TestAnnotation3("v1")
@TestAnnotation3("v2")
public class TestClass2 {
}
```

Java SE 自定义的注解在 `java.lang` 和 `java.lang.annotation` 包下面，比如常见的 `@Override`、`SuppressWarnings`，它们都是预先定义好的注解，当然我们也可以像上面例子一样自定义自己的注解。



## 2、Where Annotations Can Be used

Annotation 可以在声明上使用：类声明、方法声明、字段声明以及其他编程元素上。

截止到 Java 8，注解也可以作用于下列类型，下面是一些例子：

- 创建类实例时：`new @Interned MyObject();`
- 类型转换：`myString = (@NonNull String) str;`
- implement 关键字：`class UnmodifiableList<T> implements @Readonly List<@Readonly T> { ... }`
- throws 关键字：`void monitorTemperature() throws @Critical TemperatureException { ... }`

这种类型的注解叫做 `type annotation`，可以参考 [Type Annotations and Pluggable Type Systems](https://docs.oracle.com/javase/tutorial/java/annotations/type_annotations.html)

type annotation 是为了增强 Java 程序的分析能力，以确保更强的类型检测，虽然说 Java 8 没有提供任何的类型检测框架，但是它允许开发者自己编写或者下载任何实现了一个或多个可拔插模块的类型检测框架，结合 Java 编译器实现更强大功能。

比如说你项让程序中的某个引用不能赋为 null 值，从而避免 NullPointerException，我们就可以编写一个插件来实现这个功能，可以使用注解，比如这样：`@NonNull String str;`。这样在编译代码时，编译器如果发现 str 为 null，就会打印警告信息。

当然大部分情况下，我们不会自己编写检测模块，而是使用第三方的检测框架，可以参考这个：[Checker Framework](https://checkerframework.org/)。

进度：

- https://docs.oracle.com/javase/tutorial/java/annotations/declaring.html