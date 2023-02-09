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



# API Specification

包含集合框架的类和接口的带注释的大纲，以及到JavaDoc的链接。

参考：https://docs.oracle.com/javase/8/docs/technotes/guides/collections/reference.html



# Tutorial

本小节参见：https://docs.oracle.com/javase/tutorial/collections/index.html

主要介绍 Java 的集合框架，以及利用它们编写更简洁高效的程序；

## Introduction

集合 Collection 有时也叫做容器 Container，它将多个元素组合为一个元素。Collection 常用来存储、检索、操作甚至在聚集的元素中通信。

### 什么是集合框架？

collections framework 代表着一种统一的基础架构，主要用来表示和操作 collection。所有的集合框架都应该包括以下几部分：

- Interfaces：表示 collection 的抽象数据类型。接口允许集合的不同实现有彼此独立的操作元素的行为。在面向对象语言中，接口通常呈层级结构；
- Implementations：它们是 Interfaces 的具体实现。本质上它们就是可复用的数据结构；
- Algorithms：能够执行有效计算的方法，比如在实现了 collection interfaces 的对象上进行元素的查询、排序操作。一般此种方法都是具有多态性的。本质上，算法是可以复用的功能函数。

除了 Java 的集合框架外，最有名的莫过于 C++ 的 STL（Standard Template Library）和 Smalltalk 的 collection hierarchy。

## Interfaces

Java 集合框架中最核心的几个接口封装了集合的不同类型，如下表所示：

（1）Collection 体系

| 接口       | 子接口 | 子接口    |
| ---------- | ------ | --------- |
| Collection | Set    | SortedSet |
|            | List   |           |
|            | Queue  |           |
|            | Deque  |           |

（2）Map 体系

| 接口 | 子接口    |
| ---- | --------- |
| Map  | SortedMap |

- 注意 Map 并不是 Collection；
- 所有的接口都支持泛型；

具体描述：

- `Collection`：collection 体系的根接口。一个 collection 表示一组 element 的集合。它给出了集合的通用定义；在 Java 平台中，没有该接口的直接实现，而是提供了继承它的一些子接口；
- `Set`：不包含重复元素的集合；
- `List`：有序的集合（有时也叫做 sequence），它可以存储相同的元素，并提供对 List 中的元素随机访问的能力（通过索引下标）；
- `Queue`：一般存储将要被处理的多个元素。除了 Collection 默认的行为外，Queue 还提供了一些附加的插入、抽取和检视操作；

通常队列（但不一定）以 FIFO（first-in，first-out）的方式对元素进行排序。优先级队列就是一种例外的情况，它根据提供的比较器或元素的自然顺序对元素进行排序。但是无论使用何种顺序，队列的头都是将调用 `remove` 或 `poll` 方法删除的元素。在 FIFO 队列中，所有新加入的元素都会被插入到队列的尾部。其他类型的队列可能使用不同的放置规则。但是每个 Queue 的实现都必须指定它的排序属性。

- `Deque`：一般存储将要被处理的多个元素。除了 Collection 默认的行为外，Deque 还提供了一些附加的插入、抽取和检视操作；

Deque 可以使用 FIFO（first-in，first-out）或 LIFO（last-in，first-out）在 Deque 中，所有新元素都可以在队列的两端插入、检索和删除。

- `Map`：存储一组 key 和 value 映射关系的对象。Map 不能存储相同的 key，每个 key 可以关联至少一个对象；

最后两个核心集合接口仅仅是 Set 和 Map 的排序版本：

- `SortedSet`：按某种顺序保存元素的 Set。提供了几个额外的方法，这些方法利用了其有序的特性；
- `SortedMap`：映射关系有序的 Map。它是 SortedSet 在 Map 中的体现。排序映射用于键/值对的自然排序集合，例如字典和电话簿；

### The Collection Interface

> （1）简介

在集合框架中有这样一种构造形式，叫做 `conversion constructor`，例如下面这样，在不清楚 c 是什么具体类型的时候，可以通过此种构造方式将 c 中的所有元素存储到 List 中。

```java
Collection<String> c = ...;
List<String> list = new ArrayList<>(c);
```

Collection 提供了集合的一些通用的操作。

- 常规方法：`size()` 、`isEmpty()` 、`contains(Object element)` 、`add(E e)`、`remove(Object e)` 以及 `Iterator<E> iterator()`；
- 操作整个集合的方法：`containsAll(Collection<?> c)`、`addAll(Collection<? extends E> c)`、`removeAll(Collection<?> c)`、`retainAll(Collection<?> c)` 以及 `clear()`；
- 数组操作：`toArray()` 以及 `toArray(T[] a)`；

在 JDK 8 和以后的版本，Collection 接口还引入了 `Stream<E> stream()` 和 `Stream<E> parallelStream()` 用于从集合上获取顺序流或并行流。

> （2）遍历集合

三种方式：

- 使用聚合操作；

在 JDK 8 和之后的版本中，迭代集合最好的方式就是获取流然后结合 Lambda 语法执行聚合操作，这样编写的代码更少，更具表现力。比如下面这样遍历一个形状集合，然后打印红色的对象：

```java
myShapesCollection.stream()
.filter(e -> e.getColor() == Color.RED)
.forEach(e -> System.out.println(e.getName()));
```

除此之外也可以使用并行流，在集合足够大且机器具有多个核心时，使用并行流的效率更高（底层使用 fork-join 框架并行切分任务，执行完毕后合并子任务结果）：

```java
myShapesCollection.parallelStream()
.filter(e -> e.getColor() == Color.RED)
.forEach(e -> System.out.println(e.getName()));
```

在 Stream API 中有很多方法，可以高效的操作集合，比如将某个元素集合转换为 String 集合然后拼接为一个由 "," 分隔的字符串：

```java
String joined = elements.stream()
    .map(Object::toString)
    .collect(Collectors.joining(", "));
```

甚至求和：

```java
int total = employees.stream()
.collect(Collectors.summingInt(Employee::getSalary)));
```

这些只是您可以使用流和聚合操作所做的一些示例。

- 使用增强 for 循环；

```java
for (Object o : collection)
    System.out.println(o);
```

- 使用迭代器；

迭代其接口：

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove(); //optional
}
```

迭代：

```java
static void filter(Collection<?> c) {
    for (Iterator<?> it = c.iterator(); it.hasNext(); )
        if (!cond(it.next()))
            it.remove();
}
```

### The Set Interface

Set 就是不包含重复元素的 Collection。Set 接口中的方法都是从 Collection 继承的。只不过追加了禁止添加重复元素的限制。此外，Set 对 `equals` 和 `hashcode` 方法添加了更强的约束。如果两个 Set 实例包含相同的元素，则它们相同。

Java 平台提供了三种 Set 的不同实现：HashSet、TreeSet 以及 LinkedHashSet。

（1）HashSet 利用一个 hash table 来存储元素，性能最好，但是它不能保证插入元素的顺序；

（2）TreeSet 使用红黑树来存储元素，根据存储的元素来进行排序，它比 HashSet 要慢很多；

（3）LinkedHashSet 使用链表版本的 hash table 存储元素，根据元素插入时的顺序排序。和 HashSet 相比，只需要花费略高的代价就可以保证顺序。



### The List Interface

List 就是有序的 Collection（有时也叫序列）。List 中可以包含重复的元素除了继承自 Collection 接口的方法，List 接口包含以下特有的方法：

- Positional access：List 通过元素的数值下标操作元素，提供 get、set、add、addAll 以及 remove 方法；
- Search：在 List 中搜索某个元素，然后返回元素下标，搜索方法包括 indexOf、lastIndexOf；
- Iteration：继承了 Iterator 接口，结合 List 序列的特性。通过 listIterator 方法提供此特性；
- Range-view：sublist 方法为 List 提供 `range operation`；

Java 平台提供了 List 的两个通用实现：

（1）ArrayList，它的性能最好；

（2）LinkedList 在特定情况下提供更好的性能；

可以从元素的访问以及元素的插入删除来比较两者；

在 Collections 工具类中提供了一些 List 的算法，它们都支持多态：

- `sort`：使用归并排序算法排序 List，它提供快速的稳定的排序（稳定排序是指排序前后不改变相等元素的前后位置）；

- `shuffle`：随机排列 List 中的元素；
- `reverse`：使 List 中的元素逆序；
- `rotate`：将 List 中的所有元素旋转指定的距离；
- `swap`：交换 List 中两个元素的位置；
- `replaceAll`：用另一个指定值替换所有出现的指定值；
- `fill`：用指定的值覆盖 List 中的每个元素；
- `copy`：将源列表复制到目标列表中；
- `binarySearch`：使用二分查找算法在有序列表中搜索元素；
- `indexOfSubList`：返回一个 List 中与另一个 List 相等的第一个子列表的索引；
- `lastIndexOfSubList`：返回一个 List 中与另一个 List 相等的最后一个子列表的索引。

### The Queue Interface

Queue 就是存储待处理元素的集合。除了 Collectino 定义的方法，Queue 还提供了一些特有的 insertion、removal and inspection 的操作。

```java
public interface Queue<E> extends Collection<E> {
    E element();
    boolean offer(E e);
    E peek();
    E poll();
    E remove();
}
```

每个 Queue 方法有两种格式：

（1）当操作执行失败时抛出异常；

（2）当操作执行失败时排除特定指；（该值根据方法不同，返回 null 或者 false）；

如下表所示：

| 操作类型 | 抛出异常  | 返回特定值 |
| -------- | --------- | ---------- |
| Insert   | add(e)    | offer(e)   |
| Remove   | remove()  | poll()     |
| Examine  | element() | peek()     |

一般来说 Queue 采取 FIFO 模式，特殊的情况就是优先级队列，它根据元素值进行排序。但是无论顺序是怎么样的，调用 `remove` 或者 `poll` 方法都会返回队列的头结点。FIFO 模式下的 Queue，新的元素将会被追加到队列的末尾；而其他类型的 Queue 也许会采用不同的存储规则。每个 Queue 的实现都必须指定其排序的规则；

Queue 的实现通常不允许插入 null 元素，而 LinkedList 是一个例外，它也是 Queue 的一种实现，但是处于历史原因，它可以存储 null 元素，但是开发者应该注意避免这种情况，因为 poll 和 peek 方法返回值中 null 具有特殊意义。

Queue 的实现通常没有基于存储的元素来重写 `equals` 和 `hashcode` 方法，而是选择保留继承自 Object 的这两个方法。

Queue 接口没有定义阻塞队列方法，而阻塞队列方法在并发编程中很常见。这些方法在接口  `java.util.concurrent.BlockingQueue` 中定义，用于等待元素出现或等待空间可用，它扩展了 Queue。

下面这个例子展示了利用优先级队列对集合进行排序，仅作演示优先级队列的特殊行为，集合排序有更好的方法：

```java
static <E> List<E> heapSort(Collection<E> c) {
    // 这里的优先级队列基于 二叉平衡堆 实现
    Queue<E> queue = new PriorityQueue<E>(c);
    List<E> result = new ArrayList<E>();

    while (!queue.isEmpty())
        result.add(queue.remove());

    return result;
}
```



### The Deque Interface

Deque 是双端队列。双端队列是元素的线性集合，支持在两个端点插入和删除元素。Deque 接口是一种比 Stack 和 Queue 更丰富的抽象数据类型，因为它同时实现了堆栈和队列。Deque 接口定义了访问 Deque 实例两端元素的方法，包括插入、删除以及评估元素。像 ArrayDeque 和 LinkedList 这样的预定义类实现了 Deque 接口。

注意 Deque 既可以采用 LIFO 模式的 stack，也可以采用 FIFO 的 queue，Deque 接口提供的方法可以分为三部分：

（1）Insert

`addFirst` 和 `offerFirst` 在 Deque 的首部插入元素，`addLast` 和 `offerLast` 则在 Deque 的末端插入元素，当 Deque 元素数量收到限制的时候，使用 `offerFirst` 和 `offerLast` 方法更好，因为 `addFirst` 和 `addLast` 方法在双端队列容量满了的情况下插入元素会失败并抛出异常；

（2）Remove

`removeFirst` 和 `pollFirst` 从 Deque 的首部移除元素，`removeLast` 和 `removeLast` 在 Deque 的尾部移除元素，当双端队列是空的时候，`pollXxx` 方法会返回 null，而 `removeXxx` 方法则会抛出异常；

（3）Retrieve

`getFirst` 和 `peekFirst` 查看 Deque 的第一个元素，它们不会从 Deque 中移除对应的元素。类似的 `getLast` 和 `peekLast` 查看 Deque 的最后一个元素。`getFirst` 和 `getLast` 在 Deque 是空的时候会抛出异常，而另外两个方法则返回 null；

这 12 个用于插入、移除以及检索 Deque 元素的方法如下表所示：

| 操作类型 | Deque 的首部           | Deque 的尾部         |
| -------- | ---------------------- | -------------------- |
| Insert   | addFirst、offerFirst   | addLast、offerLast   |
| Remove   | removeFirst、pollFirst | removeLast、pollLast |
| Examine  | getFirst、peekFirst    | getLast、peekLast    |

除了这些插入、删除和检查 Deque 实例的基本方法之外，Deque 接口也有一些预定义的方法。其中有一个 `removeFirstOccurrence` 方法，如果指定元素存在于 Deque 实例中，此方法将删除第一个检索到的该元素，如果该元素不存在，Deque 实例不受影响。另一个类似的方法是 `removeLastOccurrence`；

这两个方法的返回值是 boolean，如果双端队列中存在指定的元素则返回 true，反之则是 false。

### The Map Interface





进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html
- https://docs.oracle.com/javase/tutorial/collections/interfaces/map.html