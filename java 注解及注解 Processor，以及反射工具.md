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

更多信息可以参考：[java.lang.reflect.AnnotatedElement](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedElement.html)，该接口定义了一些方法用于读取注解信息。

## 5、Design Considerations

当我们设计一个注解的时候，一定要考虑到此种注解的 `cardinality of annotations`（注解的基数）。

- 在实际环境中使用这个注解，可能不会用它标注元素，或者只标注一次，甚至重复标注；
- 通过 `@Target` 来限制该注解保留的范围，是源码级别、还是 class 级别，甚至是运行时；

比如说像创建一个只能用在方法和字段上的可以重复标注的注解。

合理的设计注解，我们在程序中编码获取该注解及其定义的属性时也会更加灵活和高效。



# 三、反射

## 1、概念

Java 的 Reflection API 允许在代码中获取被加载类的字段、方法、构造器的信息，在安全范围内操作这些信息，甚至获得完全控制权。

反射包：`java.lang.reflect`

- 描述：此包提供了一些类和接口，用于获取类和对象的反射信息。

其中有几个特殊的类和接口：

> ReflectPermission 和 AccessibleObject

在 `ReflectPermission` 可用的前提下， `AccessibleObject`  可以控制对目标对象的访问权限，比如默认只能获取目标对象的共有的属性或方法，通过访问控制就可以抑制这些限制，从而获取所有的信息。

注：这里涉及到 Java Security 相关知识，且 ReflectPermission 在 AccessibleObject 有使用：

![](https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20221126103439.png)

![](https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20221126104611.png)

> Array

`Array` final 类提供了一些静态方法用于创建和访问数组。

## 2、说明

### 使用反射

当程序在 JVM 中运行时，如果需要改变或者检测运行时行为，就可以使用反射。反射是一种高级的特性，如果想要使用它，最好对 Java 这门的基础掌握的很牢固。

要记住：反射是一种强大的技术，它可以使应用程序执行原本不可能执行的操作。

### 可扩展

程序可能会使用一些外部的、用户自定义的类，此时可以通过使用它们的全限定名结合反射创建实例；

### Class Browsers 和 Visual Development Environment

查看类的信息需要枚举类中的所有成员；

可视化开发中使用反射获取的信息能够帮助开发者编写正确的代码。

### Debuggers 和 Test Tools

Debug 需要检测类的私有成员；

测试工具可以使用反射调用定义在类上的方法，确保测试代码的高覆盖率。

### 反射的缺点

反射非常高效，但是也要注意不能随意使用，如果某些操作不需要使用反射就能够完成，那就尽量不使用它。

通过反射访问代码时，需要注意以下事项：

（1）Performance 和 Overhead（性能和开销）

因为反射涉及到动态解析类型，所以不能够适配某些 JVM 的优化策略（比如说 JIT）；

因此反射操作的性能要比不使用它的操作更慢，在对性能敏感的应用当中经常调用的代码应该避免使用反射操作。

（2）Security Restrictions（安全限制）

反射需要运行时的权限，但是在安全管理器（Java Security Manager）下程序运行时可能没有这些权限。对于必须运行在 Restricted Security Context 中的程序，此时是否使用反射就需要认真考虑一下了。 

（2）Exposure of Internals（暴露内部信息）

因为反射允许执行某些在非反射环境下的非法操作，比如说访问 private 属性或者方法，反射就可能导致一些意想不到的副作用：可能破坏代码的可移植性，反射打破了抽象，因此可能会随着平台的升级而改变行为。

`Since reflection allows code to perform operations that would be illegal in non-reflective code, such as accessing `private` fields and methods, the use of reflection can result in unexpected side-effects, which may render code dysfunctional and may destroy portability. Reflective code breaks abstractions and therefore may change behavior with upgrades of the platform.`



## 3、Classes

任何类型要么是引用类型、要么是基础数据类型。

引用类型：Classes、enums、arrays（它们都继承自 `java.lang.Object`）和接口，以及 `java.lnag.String`，基础数据类型的包装类比如 `java.lang.Double`，接口 `java.io.Serializable`，枚举 `javax.swing.SortOrder`；

基础数据类型：`boolean`, `byte`, `short`, `int`, `long`, `char`, `float`, 和`double`。

对于各种类型的实例对象，JVM 都会实例化一个不可变的 `java.lang.Class` 类型的对象，这个对象提供了一系列方法用于在运行时获取目标对象的各种成员属性及类型信息，Class 还提供了创建新类和新对象的功能 ，更重要的是：它是所有反射 API 的入口。下面就来了解一下常用的关于 Class 的反射 API。

### 获取 Class 对象

`java.lang.Class` 是调用所有反射 API 的入口，首先我们要做的就是获取 Class 的实例；

此外还需注意的是在 `java.lang.reflect` 包下面，除了 `java.lang.reflect.ReflectPermission` 有公开的构造器外，其他所有类都没有 public 类型的构造器，只能通过 Class 对象的方法去获取这些类的 Class 对象。

根据代码拥有的对象的访问权限，有几种方式可以获取 Class 对象，包括类的全限定名、类型、已存在的 Class。

#### （1）Object.getClass()

当我们拥有一个对象的实例，调用实例的 `getClass()` 方法就可以很容易的获得其对应的 Class 对象，当然此种方式只适用于继承自 Object 类的子类的实例的引用。比如说：

获取 String 类的 Class 对象：

```java
@Test
public void test1() {
    String txt = "abc";
    Class<? extends String> StrClazz = txt.getClass();
}
```

通过 `System.console()` 方法和 JVM 关联的唯一的 console，`getClass()` 方法返回的值是 `java.io.Console` 对应的 Class 对象：

```java
@Test
public void test2() {
    Class<? extends Console> clz = System.console().getClass();
}

public static Console console() {
    if (cons == null) {
        synchronized (System.class) {
            cons = sun.misc.SharedSecrets.getJavaIOAccess().console();
        }
    }
    return cons;
}
```

下面的例子中 A 是枚举 E 的一个实例，调用它的 `getClass()` 方法返回的 Class 对象对应的是枚举 E：

```java
enum E {
    A, B;
}

@Test
public void test3() {
    Class<? extends E> clz = E.A.getClass();
}
```

数组是一系列元素的集合，数组实例也可以调用 `getClass()` 方法，下面的例子返回的 Class 对象就是 `int[]` 对应的 Class 实例：

```java
@Test
public void test4() {
    int[] arr = { 1, 2, 3, 4, 5, 6, 7 };
    Class<? extends int[]> arrClz = arr.getClass();
}
```

下面的例子展示了类型为 `java.util.Set` 的 `java.util.HashSet` 实例调用 `getClass()` 方法获得的 Class 对象，这个对象对应 `java.util.HashSet`：

```java
@Test
public void test5() {
    Set<String> set = new HashSet<>();
    Class<? extends Set> setClz = set.getClass();
}
```

#### （2）.class Syntax

如果我们知道具体的类型但是没有可用的实例，此时可以利用 `类型.class` 这种语法直接获取对应的 Class 对象，对于原始数据类型也是一样的：

```java
@Test
public void test6() {
    Class<TestGetClassInstance> testClz = TestGetClassInstance.class;
    boolean b = true;
    // compile-time error
    // b.getClass();
    
    // correct
    Class<Boolean> booleanClass = boolean.class;
}
```

注意直接使用 `b.getClass()` 时，编译器会生成 error 信息，因为 `boolean` 是原始数据类型不能够 `dereferenced`。

上面的例子中 `boolean.class` 返回的是 `boolean` 原始类型的 Class 对象；

再看下面两个例子：

```java
@Test
public void test7() {
    // 可以使用全限定名获取 Class 对象
    Class<PrintStream> printStreamClass = java.io.PrintStream.class;
    
    // 下面获取的是多维数组的 Class 对象
    Class<int[][][]> multiDimensional = int[][][].class;
}
```

#### （3）Class.forName()

如果知道目标类的全限定名，就可以通过 `Class.forName()` 这个 Class 类的静态方法获取相应的 Class 实例，但是这种方式无法获取原始数据类型的 Class 实例。

研究该方法的源码时可以看看 JLS（Java Language Specification）中关于类加载的描述：[12.4 Initialization Of Classes and Interfaces](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html#jls-12.4)，就可以知道 JVM 是如何确保类只会被初始化一次的（毕竟 Java 运行环境是多线程的）。

下面看例子，这里有特殊的地方，比如 `[D` 表示原始数据类型 `double[]` 的 Class 对象，`[[Ljava.lang.String;` 是 `String[][]` 二维数组的 Class 对象（tip：`;` 不能丢）。

```java
@Test
public void test8() throws ClassNotFoundException {
    // 创建 TestClassForName 的 Class 对象
    Class<?> clz = Class.forName("io.naivekyo.reflection.TestClassForName");

    // 创建 double[] 数组的 Class 对象, 注意是原始数据类型, 等同于 double[].class
    Class<?> doubleArray = Class.forName("[D");

    // 创建 String[][] 的 Class 对象, 等同于 String[][].class
    Class<?> stringArray = Class.forName("[[Ljava.lang.String;");
}
```

至于这些类型、数组或者原始类型的名称是如何获得的，可以使用 Class 的静态方法 `getName()` 获取：

```java
@Test
public void test9() {
    System.out.println(String.class.getName());
    System.out.println(int.class.getName());
    System.out.println(long[].class.getName());
    System.out.println(String[].class.getName());
    System.out.println(Integer.class.getName());
    System.out.println(TestClassForName.class.getName());
}
```

Expect Output：

```
java.lang.String
int
[J
[Ljava.lang.String;
java.lang.Integer
io.naivekyo.reflection.TestClassForName
```

#### （4）原始数据类型包装类的 TYPE 静态字段

使用 `.class` 语法可以很方便的获取原始数据类型的 Class 对象，除了这种方式以外还有一种方式可以做到。

对于每一个原始数据类型和 void 类型，在 `java.lang` 包下都有一个包装类，这些包装类将原始数据类型转换为引用类型，包装类有一个名为 `TYPE` 的静态 final 变量，该变量存储的是对应的原始类型的 Class 对象：

```java
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
```

```java
@Test
public void test10() {
    System.out.println(Integer.TYPE);
    System.out.println(Void.TYPE);
}
```

#### （5）返回 Classes 的 Reflect API

当我们已经获得了 Class 对象时，可以调用几个反射 API，这些方法能够返回 Classes：

（1）`getSuperClass()`

返回指定 Class 对象的父类 Class 对象：

```java
@Test
public void test11() {
    Class<? super JButton> superclass = javax.swing.JButton.class.getSuperclass();
    System.out.println(superclass); // Expect Output: class javax.swing.AbstractButton
}
```

（2）`getClasses()`

返回指定 Class 中定义的 public 类型的 class、interface、enum（也包括继承的）

```java
@Test
public void test12() {
    Class<?>[] classes = Character.class.getClasses();
    for (Class<?> clz : classes) {
        System.out.println(clz);
    }
}
```

```
class java.lang.Character$Subset
class java.lang.Character$UnicodeBlock
class java.lang.Character$UnicodeScript
```

Character 中定义了两个 public 成员类：`java.lang.Character.Subset` 和 `java.lang.Character.UnicodeBlock`，一个 public 枚举 `java.lang.Character.UnicodeScript`

（3）`getDeclaredClasses()`

该方法返回目标 Class 中声明的所有类型的 class、interface、enum 成员：

```java
@Test
public void test13() {
    Class<?>[] allDeclaredClasses = Character.class.getDeclaredClasses();
    for (Class<?> clz : allDeclaredClasses) {
        System.out.println(clz);
    }
}
```

```
class java.lang.Character$CharacterCache
class java.lang.Character$Subset
class java.lang.Character$UnicodeBlock
class java.lang.Character$UnicodeScript
```

出了上面提到的 3 个 public 类型的，还获得了 private 的 `java.lang.Character.CharacterCache`。

（4）e.g：

- `java.lang.Class.getDeclaringClass`；
- `java.lang.reflect.Field.getDeclaredClass()`；
- `java.lang.reflect.Method.getDeclaredClass()`；
- `java.lang.reflect.Constructor.getDeclaredClass()`；

 上面的四个方法都会获得声明该对象的类的 Class 对象实例。

比如下面的测试类：

```
public class TestGetDeclaredMethod {
    
    private int a;

    public TestGetDeclaredMethod() {
    }

    public TestGetDeclaredMethod(int a) {
        this.a = a;
    }

    public int getA() {
        return a;
    }

    public void setA(int a) {
        this.a = a;
    }
}
```

有一个私有属性，getter/setter 方法、两个构造器：

```
@Test
public void test14() {
    Class<TestGetDeclaredMethod> clz = TestGetDeclaredMethod.class;
    
    Class<?> declaringClass = clz.getDeclaringClass();
    System.out.println(declaringClass); // expect output: null

    System.out.println();
    
    for (Field field : clz.getDeclaredFields()) {
        System.out.println(field.getDeclaringClass());
        // expect output: class io.naivekyo.reflection.TestGetDeclaredMethod
    }
    
    System.out.println();
    
    for (Method method : clz.getDeclaredMethods()) {
        System.out.println(method.getDeclaringClass());
        // expect output: 
        // class io.naivekyo.reflection.TestGetDeclaredMethod
        // class io.naivekyo.reflection.TestGetDeclaredMethod
    }

    System.out.println();
    
    for (Constructor<?> constructor : clz.getDeclaredConstructors()) {
        System.out.println(constructor.getDeclaringClass());
        // expect output: 
        // class io.naivekyo.reflection.TestGetDeclaredMethod
        // class io.naivekyo.reflection.TestGetDeclaredMethod
    }
}
```

该测试方法中会打印声明该成员的类的 Class 对象，在这里就是 TestGetDeclaredMethod 对应的 Class 对象，第一次打印为 null 是因为 TestGetDeclaredMethod 没有作为其他类的成员。

此外有一点需要注意的是匿名内部类没有 declaring class，但是有 enclosing class，可以获得声明该 enclosing class 的 Class 对象：

```java
public class AnonymousEnclosingTest {
    
    static Object o = new Object() {
        public void m() {}
    };

    static Class<?> c = o.getClass().getEnclosingClass();

    public static void main(String[] args) {
        System.out.println(c);
        // Expect output: class io.naivekyo.reflection.AnonymousEnclosingTest
        System.out.println(o.getClass().getDeclaringClass());
        // Expect output: null
    }
}
```

（5）`Class.getEnclosingClass()`

该方法返回目标 Class 对象的附着 Class 对象（Enclosing class）

比如：

```java
@Test
public void test15() {
    Class<?> enclosingClass = Thread.State.class.getEnclosingClass();
    System.out.println(enclosingClass);
    // Expect output: class java.lang.Thread

    System.out.println(Thread.class.getEnclosingClass());
    // Expect output: null
}
```

`Thread.State` 枚举是作为 `Thread` 的附着类声明的，因此调用 getEnclosingClass 获得是 Thread 的 Class 对象。

而如果直接调用 `Thread.class.getEnclosingClass()`，Thread 作为 [top level class](https://docs.oracle.com/javase/specs/jls/se7/html/jls-7.html#jls-7.6) 声明，因此返回 null。



### 检查类修饰符和类型

通常会通过一个或多个修饰符声明一个类，这些修饰符将会影响类的运行时行为。

- 访问控制：public、protected、private；
- 需要重写的：abstract；
- 限定只有一个实例：static；
- 禁止修改：final；
- 强制执行严格浮点行为：strictfp；
- 注解

这些修饰符并不一定能作用于所有的类，比如说 interface 就不能被 final 修饰，enum 不能被 abstract 修饰。

`java.lang.reflect.Modifier` 中包含了所有可用的修饰符信息，并提供了一系列方法用于解码 `java.lang.Class#getModifiers` 、`java.lang.reflect.Member#getModifiers` 方法返回的数据。

下面的例子展示了如何获得类声明的各种组件，包括修饰符、泛型参数、实现的接口和继承树，`java.lang.Class` 实现了 `java.lang.reflect.AnnotatedElement` 接口，因此我们也可以在运行时获得注解信息。

```java
public class ClassDeclarationSpy {

    public static void show(String... args) {
        try {
            Class<?> c = Class.forName(args[0]);
            System.out.printf("Class:%n %s%n%n", c.getCanonicalName());
            System.out.printf("Modifiers: %n %s%n%n", Modifier.toString(c.getModifiers()));

            System.out.printf("Type Parameters:%n");
            TypeVariable[] tv = c.getTypeParameters();
            if (tv.length != 0) {
                System.out.printf(" ");
                for (TypeVariable t : tv) {
                    System.out.printf("%s ", t.getName());
                }
                System.out.printf("%n%n");
            } else {
                System.out.printf(" -- No Type Parameters --%n%n");
            }

            System.out.printf("Implemented Interfaces: %n");
            Type[] intfs = c.getGenericInterfaces();
            if (intfs.length != 0) {
                for (Type intf : intfs) {
                    System.out.printf(" %s%n", intf.toString());
                }
                System.out.printf("%n");
            } else {
                System.out.printf(" -- No Implemented Interfaces --%n%n");
            }

            System.out.printf("Inheritance Path:%n");
            List<Class> l = new ArrayList<>();
            printAncestor(c, l);
            if (l.size() != 0) {
                for (Class cl : l) {
                    System.out.printf(" %s%n", cl.getCanonicalName());
                }
                System.out.printf("%n");
            } else {
                System.out.printf(" -- No Super Classes --%n%n");
            }

            System.out.printf("Annotations:%n");
            Annotation[] ann = c.getAnnotations();
            if (ann.length != 0) {
                for (Annotation a : ann) {
                    System.out.printf(" %s%n", a.toString());
                }
                System.out.printf("%n");
            } else {
                System.out.printf(" -- No Annotations --%n%n");
            }

        } catch (ClassNotFoundException e) {
            // production code should handle this exception more gracefully
            e.printStackTrace();
        }
    }
    
    private static void printAncestor(Class<?> c, List<Class> l) {
        Class<?> ancestor = c.getSuperclass();
        if (ancestor != null) {
            l.add(ancestor);
            printAncestor(ancestor, l);
        }
    }
    
    public static void main(String[] args) {
        show("java.util.concurrent.ConcurrentNavigableMap");
    }
}
```

输出：

```
Class:
 java.util.concurrent.ConcurrentNavigableMap

Modifiers: 
 public abstract interface

Type Parameters:
 K V 

Implemented Interfaces: 
 java.util.concurrent.ConcurrentMap<K, V>
 java.util.NavigableMap<K, V>

Inheritance Path:
 -- No Super Classes --

Annotations:
 -- No Annotations --
```

该接口实际声明如下：

```java
public interface ConcurrentNavigableMap<K,V>
    extends ConcurrentMap<K,V>, NavigableMap<K,V> {}
```

注意这是一个接口，它被隐式地声明为 abstract，Java 编译器对所有接口都是这样的处理，上面的 map 包含两个泛型参数 K 和 V，示例代码只是简单打印了它们的名字，通过 `java.lang.reflect.TypeVariable` 接口提供的方法可以查看更多的关于泛型的附加信息。

当然，接口也可以实现其他接口。

下面看看 String 数组的信息：

```java
public static void main(String[] args) {
    show("[Ljava.lang.String;");
}
```

```
Class:
 java.lang.String[]

Modifiers: 
 public abstract final

Type Parameters:
 -- No Type Parameters --

Implemented Interfaces: 
 interface java.lang.Cloneable
 interface java.io.Serializable

Inheritance Path:
 java.lang.Object

Annotations:
 -- No Annotations --
```

因为 arrays 也是运行时对象，JVM 定义了所有的类型信息。注意 arrays 实现了 Cloneable 和 Serializable 接口并且继承自 Object。

下面看一个过失的类：`show("java.security.Identity");`

```
Class:
 java.security.Identity

Modifiers: 
 public abstract

Type Parameters:
 -- No Type Parameters --

Implemented Interfaces: 
 interface java.security.Principal
 interface java.io.Serializable

Inheritance Path:
 java.lang.Object

Annotations:
 @java.lang.Deprecated()
```

获得的注解信息显示该类被 `@Deprecated` 注解标注，是不推荐使用的 API。

### Discover Class Members

Class 类中提供了两种方法用于检索可访问的 fields、methods 和 constructors：

- 枚举所有成员的方法；
- 搜索特定成员的方法；

此外，访问在类上声明的成员的方法和访问父类或者父接口的成员方法是不一样的。

| Class API             | 检索所有成员 | 继承的成员 | 私有成员 |
| --------------------- | ------------ | ---------- | -------- |
| `getDeclaredField()`  | no           | no         | yes      |
| `getField()`          | no           | yes        | no       |
| `getDeclaredFields()` | yes          | no         | yes      |
| `getFields()`         | yes          | yes        | no       |

Method 也和 Field 一样。

| Class API                   | 检索所有成员 | 继承的成员      | 私有成员 |
| --------------------------- | ------------ | --------------- | -------- |
| `getDeclaredConstructor()`  | no           | N/A<sup>1</sup> | yes      |
| `getConstructor()`          | no           | N/A<sup>1</sup> | no       |
| `getDeclaredConstructors()` | yes          | N/A<sup>1</sup> | yes      |
| `getConstructors()`         | yes          | N/A<sup>1</sup> | no       |

<sup>1</sup> ：构造器是不能继承的。

看下面的示例：

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Member;
import static java.lang.System.out;

enum ClassMember { CONSTRUCTOR, FIELD, METHOD, CLASS, ALL }

public class ClassSpy {
    public static void show(String... args) {
        try {
            Class<?> c = Class.forName(args[0]);
            out.format("Class:%n  %s%n%n", c.getCanonicalName());

            Package p = c.getPackage();
            out.format("Package:%n  %s%n%n",
                    (p != null ? p.getName() : "-- No Package --"));

            for (int i = 1; i < args.length; i++) {
                switch (ClassMember.valueOf(args[i])) {
                    case CONSTRUCTOR:
                        printMembers(c.getConstructors(), "Constructor");
                        break;
                    case FIELD:
                        printMembers(c.getFields(), "Fields");
                        break;
                    case METHOD:
                        printMembers(c.getMethods(), "Methods");
                        break;
                    case CLASS:
                        printClasses(c);
                        break;
                    case ALL:
                        printMembers(c.getConstructors(), "Constuctors");
                        printMembers(c.getFields(), "Fields");
                        printMembers(c.getMethods(), "Methods");
                        printClasses(c);
                        break;
                    default:
                        assert false;
                }
            }

            // production code should handle these exceptions more gracefully
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        }
    }

    private static void printMembers(Member[] mbrs, String s) {
        out.format("%s:%n", s);
        for (Member mbr : mbrs) {
            if (mbr instanceof Field)
                out.format("  %s%n", ((Field)mbr).toGenericString());
            else if (mbr instanceof Constructor)
                out.format("  %s%n", ((Constructor)mbr).toGenericString());
            else if (mbr instanceof Method)
                out.format("  %s%n", ((Method)mbr).toGenericString());
        }
        if (mbrs.length == 0)
            out.format("  -- No %s --%n", s);
        out.format("%n");
    }

    private static void printClasses(Class<?> c) {
        out.format("Classes:%n");
        Class<?>[] clss = c.getClasses();
        for (Class<?> cls : clss)
            out.format("  %s%n", cls.getCanonicalName());
        if (clss.length == 0)
            out.format("  -- No member interfaces, classes, or enums --%n");
        out.format("%n");
    }

    public static void main(String[] args) {
        show("java.lang.ClassCastException", "CONSTRUCTOR");
    }
}
```

此处打印出 `java.lang.ClassCastException` 的基本信息和构造器的信息：

```
Class:
  java.lang.ClassCastException

Package:
  java.lang

Constructor:
  public java.lang.ClassCastException()
  public java.lang.ClassCastException(java.lang.String)
```

因为构造器无法继承，所有这里没有打印出用于构建异常链的构造器（通常这种方法带有 `Throwable` 参数），因为它是在父类 `RuntimeException` 中定义的。

### TroubleShooting

下面展示了使用反射 API 可能出现的异常或者提示信息。

> 编译器警告：... uses unchecked or unsafe operations

调用方法的时候，会检查参数的类型和值，甚至可能发生类型转换。

比如下面的例子：

```java
import java.lang.reflect.Method;
 
public class ClassWarning {
    void m() {
    try {
        Class c = ClassWarning.class;
        Method m = c.getMethod("m");  // warning
 
        // production code should handle this exception more gracefully
    } catch (NoSuchMethodException x) {
            x.printStackTrace();
        }
    }
}
```

很多库提供的方法都使用了泛型进行优化，包括 Class 中的一些方法，例子中 `c` 被声明为原始类型（没有写泛型参数 `Class<?>`），但是它调用 `getMethod()` 方法的参数是泛型，这时就会发生未经检查的类型转换，编译器会生成警告信息。

参考 JSL：

- [Java Language Specification，Java SE 7 Edition](https://docs.oracle.com/javase/specs/jls/se7/html/index.html)
- [Unchecked Conversion](https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.9)
- [Method Invocation Conversion](https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.3)

有两种解决方案，推荐的做法是为 c 声明一个合适的泛型，比如：

`Class<?> c = warn.getClass();`

或者，使用注解抑制编译器警告行为：

```java
Class c = ClassWarning.class;

@SuppressWarnings("unchecked")
Method m = c.getMethod("m");
```

注意：通常来说，不应该忽略警告信息，因为它可能导致一些 bug。比如上面的例子中使用合适的泛型声明会更好，除非实在没有办法，这时就使用注解抑制编译器。

> 实例化时，没有构造器访问权限

如果使用 `Class.newInstance()` 方法调用无参构造器创建实例，可能会抛出 `InstantiationException`。因为此时无参构造器对调用者是不可访问的（比如 private 修饰，或其他情况）。

`Class.newInstance()` 的行为非常类似直接使用 `new` 关键字，生成失败的原因也是一样的。只不过在反射中可以利用 `java.lang.reflect.AccessibleObject` 类来越过访问限制。

比较遗憾的是 `java.lang.Class` 没有扩展 AccessibleObject，而 `java.lang.reflect.Member` 体系则引入了 AccessibleObject，因此我们可以使用 `Constructor.newInstance()` 因为它继承自 AccessibleObject。

类似这样的操作：

```java
Class<ClassDeclarationSpy> clz = ClassDeclarationSpy.class;

Constructor<ClassDeclarationSpy> noArgsConstructor = clz.getDeclaredConstructor(null);

noArgsConstructor.setAccessible(true);

ClassDeclarationSpy instance = noArgsConstructor.newInstance();
out.println(instance);
```

获取无参构器，然后使用 AccessibleObject 提供的方法获取访问权，最后生成实例。