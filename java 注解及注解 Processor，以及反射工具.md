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



## 3、Predefined Annotations Type

Java SE 提供了一些预先定义好的注解，其中有的适用于编译器，有的适用于其他注解。

比如说在 `java.long` 包下定义的 `@Deprecated`、`@Override` 和 `@SuppressWarnings` 在 Java 中经常使用。

> Java Language 中经常使用的注解

（1）@Deprecated

该注解标注的元素不推荐使用，甚至在后续版本中会废弃，在程序中使用此类元素时，编译器会提示警告信息。同时 Javadoc 中也会添加 @deprecated 标签。

值得一提的是注解和 Javadoc 的 tag 标签都是以 @ 符号开头，这并非巧合，它们在概念上是相关的，注意注解 @ 后面跟的第一个字母是大写的，注释 @ 后面跟的都是小写字母。

```java
// Javadoc comment follows
/**
 * @deprecated
 * explanation of why it was deprecated
 */
@Deprecated
static void deprecatedMethod() { }
```

（2）@Override

被该注解标注的方法会通知编译器此方法在父类中声明，在子类中被重写

（3）@SuppressWarnings

该注解用来阻止编译器生成警告信息，比如下面的例子，调用一个标注了 @Deprecated 的方法，同时在调用者上标注 @SuppressWarnings 注解，本来编译器应该警告开发者该元素不推荐使用，但是 @SuppressWarnings 又阻止了编译器这么做：

```java
// use a deprecated method and tell 
// compiler not to generate a warning
@SuppressWarnings("deprecation")
void useDeprecatedMethod() {
    // deprecation warning
    // - suppressed
    objectOne.deprecatedMethod();
}
```

Java 中不同的警告信息都会归属于一种警告类别，Java 语言规范列出了两个类别：deprecation 和 unchecked。当于泛型出现之前的遗留代码进行交互时，可能会出现 unchecked 警告，我们可以使用下面的方式去抑制多种警告：

`@SuppressWarnings({ "unchecked", "deprecation" })`

（4）@SafeVarargs

在构造器或方法上标注该注解，就可以断言代码不会对可变参数执行潜在的不安全的操作，同时对可变参数的 unchecked 警告也会被抑制。

（5）@FuncationalInterface

该注解在 JDK 1.8 版本引入，标注某个接口为函数式接口；

> 为注解服务的注解

可以作用与注解的注解叫做元注解（meta-annotations），此类注解一般在 `java.lang.annotation` 包下：

（1）@Retention

此注解表明注解保留的作用域

- RetentionPolicy.SOURCE：表明注解可以在源代码级别保留，编译器会忽略它们；（.java 文件）
- RetentionPolicy.CLASS：表明注解可以在编译时期保留，JVM 会忽略它们；（.class 文件）
- RetentionPolicy.RUNTIME：表明注解可以被 JVM 保留，意味着此类注解能够在运行时使用；

（2）@Document

此注解用于提醒 Java doc 工具再生成文档的时候将注解信息也一并带上，默认情况下一个注解是不会生成 Java doc 的。

（3）@Target

此注解用于声明目标注解可以作用的 Java 元素范围，具体作用范围如下：

- ElementType#TYPE：可以标注在类上
- ElementType#FIELD：可以标注在字段或属性上
- ElementType#METHOD：可以标注在方法上
- ElementType#PARAMETER：可以标注在方法参数上
- ElementType#CONSTRUCTOR：可以标注在构造器上
- ElementType#LOCAL_VARIABLE：可以标注在 local variable 上
- ElementType#ANNOTATION_TYPE：可以标注在注解上面
- ElementType#PACKAGE：可以标注在包上
- ElementType#TYPE_PARAMETER
- ElementType#TYPE_USE

（4）@Inherited

此注解用于声明子类可以继承父类上标注的注解（父类的注解被 @Inherited 修饰）。

通常情况下在继承关系中，父类上标注的注解和子类上标注的注解是分开的。

（5）@Repeatable

此注解从 Java SE 8 引入，此注解修饰的注解可以在同一个 Java 元素上多次修饰。在前面也提到过，更多信息参考：https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html

## 4、Retrieving Annotations

在 Java 的 Reflection API 中定义了很多方法用于检索注解的，比如说这个方法：`java.lang.reflect.AnnotatedElement#getAnnotation` 利用这个方法我们可以从特定的元素上获取想要的的注解，比如这样：

```java
@TestAnnotation1(value = "测试 1")
public class RetrievingAnnotations {
    
    @Test
    public void test1() {
        RetrievingAnnotations target = new RetrievingAnnotations();

        TestAnnotation1 a1 = target.getClass().getAnnotation(TestAnnotation1.class);

        System.out.println(a1.value()); // Expect output: 测试 1
    }
}
```

为什么需要先获取 Class 呢，可以看下面的继承图：

![](https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20221120205221.png)

如果存在前面所说的 Container Annotation（Java SE 8），比如一个类上标注了多个相同的注解，此时可以利用其他方法获取，比如：`java.lang.reflect.AnnotatedElement#getAnnotationsByType` 此方法。

```java
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(Annotation3Container.class)
public @interface TestAnnotation3 {
    String value();
}

@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Annotation3Container {
    TestAnnotation3[] value();
}
```

```java
@Test
public void test2() {
    TestAnnotation3[] annoArray = TestContainerAnnotation.class.getAnnotationsByType(TestAnnotation3.class);

    for (TestAnnotation3 anno : annoArray) {
        System.out.println(anno.value());
        // Expect output:
        // t1
        // t2
        // t3
    }
}

@TestAnnotation3(value = "t1")
@TestAnnotation3(value = "t2")
@TestAnnotation3(value = "t3")
static class TestContainerAnnotation {
    
}
```

更多信息可以参考：[java.lang.reflect.AnnotatedElement](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedElement.html)，该接口定于了一些方法用于读取注解信息。

## 5、Design Considerations

当我们设计一个自定义注解的时候，一定要考虑到此种注解的 `cardinality of annotations`。

- 在实际环境中使用这个注解，可能不会用它标注元素，或者只标注一次，甚至重复标注；
- 通过 `@Target` 来限制该注解保留的范围，是源码级别、还是 class 级别，甚至是运行时；

比如说像创建一个只能用在方法和字段上的可以重复标注的注解。

合理的设计注解，我们在程序中编码获取该注解及其定义的属性时也会更加灵活和高效。

