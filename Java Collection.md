# The Collection Framework

集合框架是表示和操作集合的统一体系结构，使它们能够独立于其表示的细节进行操作。它减少了编程工作，同时提高了性能。它实现了不相关 api 之间的互操作性，减少了设计和学习新 api 的工作量，并促进了软件重用。该框架基于十多个 collection 接口。它包括这些接口的实现和操作它们的算法。

# Overview

## Introduction

Java 平台包含集合框架。一个 collection 对象表示一组对象的集合。它为存储和操作 collection 提供了统一的结构，使操作集合得以独立于其实现细节。集合框架主要有以下优点：

- 减少代码量；
  - 提供了一些数据结构和算法，使得开发者可以复用它们；
- 提高性能；
  - 数据结构和算法提供了高性能实现。因为每个接口的不同实现都是可互换的，因此可以通过切换不同实现来提高性能；
- 为不相关的 API 提供可互操作性；
  - 通过建立一种公共语言来来回传递集合；
- 减少 API 的学习成本；
  - 只需要学习特定的几个集合 API 就可以了；
- 减少设计和实现 API 的成本；
  - 不要求您生成特定的集合 api；
- 代码复用；
  - 为 collection 提供了一个公共的接口，并且提供一系列算法来操作它们；

集合框架由以下几部分组成：

- `java.util.Collection` 接口：表示不同类型的集合，比如 set、list、map，这些接口构成了框架的基础；
- `General-purpost implementations`：集合接口有主要的实现；
- `Legacy implementations`：Java 早期发行版本提供的集合类：Vector 和 Hashtable，新的集合接口实现改进了它们；
- `Special-purpost implementations`：设计用于特殊情况的实现。这些实现展示非标准的性能特征、使用限制或行为；
- `Concurrent implementations`：为高并发情况提供的集合实现；
- `Wrapper implementations`：向其他实现添加功能，例如同步；
- `Convenience implementations`：集合接口的高性能 "mini-implementations"（迷你实现）；
- `Abstract implementations`：对部分接口的实现，用于开发者自定义实现；
- `Algorithms`：提供一些静态方法用于执行常用的操作，比如对 List 进行排序；
- `Infrastructure`：这些接口为集合接口提供一些必不可少的支持；
- `Array Utilities`：为原始数据类型和引用类型的数组提供工具方法。严格来说，它不是集合框架的一部分，该特性与集合框架同时添加到 Java 平台，并依赖于一些相同的基础设施。

## Collection Interfaces

collection interfaces 被划分为两种：

一类是以 `java.util.Collection` 作为最基础接口，它有一些扩展接口：

- `java.util.Set`；
- `java.util.SortedSet`；
- `java.util.NavigableSet`；
- `java.util.Queue`；
- `java.util.concurrent.BlockingQueue`；
- `java.util.concurrent.TransferQueue`；
- `java.util.Deque`；
- `java.util.concurrent.BlockingDeque`；

另一类是基于 `java.util.Map`  的，它们并不是真正的集合。但是这些接口包含 `collection-view` 相关操作，使得开发者可以像集合那样去操作它们。Map 有以下扩展：

- `java.util.SortedMap`；
- `java.util.NavigableMap`；
- `java.util.concurrent.ConcurrentMap`；
- `java.util.concurrent.ConcurrentNavigableMap`；

集合接口中定义的很多修改相关的方法都被标记为可选的（optional），它的实现可以选择不去执行相关操作，而是抛出一个运行时异常：`UnsupportedOperationException`。下面引进几个术语：

- 不支持修改操作（比如：add、remove、clear）的 Collections 被标记为 `unmodifiable`，与之对应的就是 `modifiable`；
- 能够保证在 Collections 中的 Object 不被修改但是可以查看的被标记为 `immutable`，与之对应的就是 `mutable`；
- 保证集合大小不变但是元素可以改变的 List 被称为 `fixed-size`，与之对应的就是 `variable-size`；
- 能够通过索引快速访问元素（通常是常数级时间复杂度）的 List 叫做 `random access` List，不支持随机访问的 List 叫做 `sequential access` List。`java.util.RandomAccess` 标记接口声明 List 具有随机访问的能力。这使得泛型算法能够在应用于随机或顺序访问列表时改变其行为以提供良好的性能。

一些实现限制了可以存储哪些元素(或者在 map 的情况下，键和值)。可能的限制包括要求要素：

- 存储特定的类型；
- 不能为 null；
- Obey some arbitrary predicate；

试图添加违反实现限制的元素会导致运行时异常，通常是ClassCastException、IllegalArgumentException 或 NullPointerException。试图删除或测试违反实现限制的元素可能导致异常。一些受限制的集合允许这种用法。

## Collection Implementations

实现集合接口的类通常遵循这样的命名格式：`<implementation-style><Interface>`。下表展示一些通用实现：

| Interface | Hash Table | Resizable Array | Balanced Tree | Linked List | Hash Table + Linked List |
| --------- | ---------- | --------------- | ------------- | ----------- | ------------------------ |
| Set       | HashSet    |                 | TreeSet       |             | LinkedHashSet            |
| List      |            | ArrayList       |               | LinkedList  |                          |
| Deque     |            | ArrayDeque      |               | LinkedList  |                          |
| Map       | HashMap    |                 | TreeMap       |             | LinkedHashMap            |

上述通用实现集合接口中定义的所有可选操作并且对存储的元素没有任何限制。它们都是非同步的，同时要注意 `java.util.Collections` 中包含了一些叫做 `synchronization wrappers` 的静态工厂方法，可以将这些非同步化的集合同步化。所有包装后的实现都是 `fail-fast` 迭代的（一旦遇到错误立即返回），一旦遇到并发修改就会失败，而不是出现各种问题。

对集合框架核心接口的抽象实现是：`AbstractCollection`、`AbstractSet`、`AbstractList`、`AbstractSequentialList`、`AbstractMap`，它们已经尽可能的减少了后续需要实现的方法。在 API 文档中详细描述了实现的具体方法、基础操作的性能指标以及如果要覆盖的话应该重写的方法。

## Concurrent Collections

下面是一些并发安全的集合实现：

- `LinkedBlockingQueue`；
- `ArrayBlockingQueue`；
- `PriorityBlockingQueue`；
- `DelayQueue`；
- `SynchronousQueue`；
- `LinkedBlockingDeque`；
- `LinkedTransferQueue`；
- `CopyOnWriteArrayList`；
- `CopyOnWriteArraySet`；
- `ConcurrentSkipListSet`；
- `ConcurrentHashMap`；
- `ConcurrentSkipListMap`；

## Design Goals

主要的设计目的是构造一个小的、重要的  API，采用 `"conceptual weight"`（权重理念）。至关重要的是，新功能对当前的 Java 程序员来说没有太大的不同；它必须扩大现有的设施，而不是取代它们。 同时，新的 API 必须足够强大，以提供前面描述的所有优点。

为了保持核心接口的数量较少，接口不会试图捕获诸如可变性、可修改性和可调整性等细微差别。相反，核心接口中的某些调用是可选的，使实现能够抛出 UnsupportedOperationException，以表明它们不支持指定的可选操作。集合实现必须清楚地记录实现支持哪些可选操作。

为了保持每个核心接口中的方法数量较小，一个接口只在以下情况下包含一个方法：

- 是一个真正的基础操作 `fundamental operation`：这是一种基本的运算，其他的运算可以被合理地定义；
- There is a compelling performance reason why an important implementation would want to override it.

所有合理的集合表示都能很好地互操作，这很关键。这包括数组，在不改变语言的情况下，数组不能直接实现 Collection 接口。因此，该框架包含了将集合移动到数组的方法、将数组视为集合的方法以及将映射视为集合的方法。





进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html