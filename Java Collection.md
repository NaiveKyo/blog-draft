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

Map 是一种保存 keys 和 values 映射关系的对象。map 对象不能保存相同的 key：但是一个 key 却可以映射至少一个对象。它是数学模型中 "函数" 的抽象。Map 接口定义了 map 的基础操作（比如：put、get、remove、containsKey，containsValue，size 以及 empty），大量数据操作（比如 putAll、clear），以及集合视图（比如 keySet、entrySet and values）。

Java 平台提供了 Map 的三种通用实现：HashMap、TreeMap 以及 LinkedHashMap。它们的行为和表现类似于 HashSet、TreeSet 以及 LinkedHashSet。

在学习 Map 的更多细节之前，先来了解一下 JDK 8 新增的集合聚合操作，将集合元素转换为 Map：

```java
// Group employees by department
Map<Department, List<Employee>> byDept = employees.stream()
.collect(Collectors.groupingBy(Employee::getDepartment));
```

```java
// Compute sum of salaries by department
Map<Department, Integer> totalByDept = employees.stream()
.collect(Collectors.groupingBy(Employee::getDepartment,
Collectors.summingInt(Employee::getSalary)));
```

```java
// Partition students into passing and failing
Map<Boolean, List<Student>> passingFailing = students.stream()
.collect(Collectors.partitioningBy(s -> s.getGrade()>= PASS_THRESHOLD));
```

```java
// Classify Person objects by city
Map<String, List<Person>> peopleByCity
         = personStream.collect(Collectors.groupingBy(Person::getCity));
```

```java
// Cascade Collectors 
Map<String, Map<String, List<Person>>> peopleByStateAndCity
  = personStream.collect(Collectors.groupingBy(Person::getState,
  Collectors.groupingBy(Person::getCity)))
```

这只是一小部分例子，更多详情参考 [Aggregation Operation](https://docs.oracle.com/javase/tutorial/collections/streams/index.html)

### Object Ordering

假设现在有一个 List （只有一个元素：1），可以这样对其排序：

```java
Collections.sort(l);
```

如果 List 包含 String 类型的元素，调用上述方法则所有元素按照字母顺序升序排列。如果 List 中包含 Date 元素，则会按照时间升序，为什么会这样？因为 String 和 Date 都实现了 `Comparable` 接口，实现了该接口的类都必须为自己提供一个 `natural ordering`，这样这个类的实例就可以按照该顺序自动排序了。

下表展示了 Java 平台中实现了 Comparable 接口的一些重要的类：

| Class        | Natural Ordering                           |
| ------------ | ------------------------------------------ |
| Byte         | 有符号数值；                               |
| Character    | 无符号数值；                               |
| Long         | 有符号数值；                               |
| Integer      | 有符号数值；                               |
| Short        | 有符号数值；                               |
| Double       | 有符号数值；                               |
| Float        | 有符号数值；                               |
| BigInteger   | 有符号数值                                 |
| BigDecimal   | 有符号数值                                 |
| Boolean      | Boolean.FALSE < Boolean.True               |
| File         | 依据文件系统中该 File 的 Path 按字典顺序； |
| String       | 字典序；                                   |
| Date         | 按发生时间顺序；                           |
| CollationKey | 特定于语言环境的词典序；                   |

看看 Comparable 接口的定义：

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

`compareTo` 方法将接口的对象参数和特定的对象进行比较：

- 当接收对象参数 less than 特定对象时，返回负数；
- 当接收对象参数 equal to 特定对象时，返回 0；
- 当接收对象参数 greater than 特定对象时，返回正数；

前面提到了 `java.util.Comparable` 接口，下面是一个例子：

```java
import java.util.*;

public class Name implements Comparable<Name> {
    private final String firstName, lastName;

    public Name(String firstName, String lastName) {
        if (firstName == null || lastName == null)
            throw new NullPointerException();
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String firstName() { return firstName; }
    public String lastName()  { return lastName;  }

    public boolean equals(Object o) {
        if (!(o instanceof Name))
            return false;
        Name n = (Name) o;
        return n.firstName.equals(firstName) && n.lastName.equals(lastName);
    }

    public int hashCode() {
        return 31 * firstName.hashCode() + lastName.hashCode();
    }

    public String toString() {
		return firstName + " " + lastName;
    }

    public int compareTo(Name n) {
        int lastCmp = lastName.compareTo(n.lastName);
        return (lastCmp != 0 ? lastCmp : firstName.compareTo(n.firstName));
    }
}
```

这个类保存人的名字，但是有一些小小的限制：仅支持 firstName 和 lastName，而且没有考虑国际化，但是该例展示了以下几个重要的地方：

- `Name` 的对象是不可变的。在所有其他条件相同的情况下，不可变类型是最好的选择，特别是对于将用作 set 中的元素或 map 中的键的对象。对于此类集合，如果在遍历它们的同时修改了元素，就会抛出异常；
- 构造器中对 null 进行了判断，这样可以确保所有的对象都是同样的格式，不会抛出 NPE；
- 重写了 `hashcode` 方法。对于任何重写了 `equals` 方法的类，重写 hashcode 是必不可少的。（因为 `Equal objects must have equal hash codes`，相同的对象必定拥有相同的 hashcode）；
- 对于 `equals` 方法，如果接收的参数对象是 null 或者和当前类实例类型匹配，就会返回 false。此时调用 `compareTo` 方法也会抛出运行时异常；
- 重写 `toString` 方法是为了提供更好的可阅读性。Collection 的 toString 方法其实是调用集合存储的元素的 toString 方法。

前面的 Comparable 接口为类提供了自然排序方式，如果我们要为某个类提供除此之外的排序方式，可以通过 `java.util.Comparator` 接口（是一种封装了 ordering 的对象）。

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

compare 方法会返回：负数、0、整数，分别对应 o1 less than o2，o1 equal to o2，o1 greater than o2。如果接收的参数不合适，该方法会抛出 ClassCastException。

其他行为和 compareTo 方法几乎一致，但是需要注意的是比较的两个元素，compareTo 是接收参数和自身比较，而 compare 是参数 1 和参数 2 比较。

`Comparator` 适合为 List 排序，但是它有一个缺点：它不能用于有序集合，比如 TreeSet。因为它提供的排序方式不能和 equals 方法一致。这就意味着，Comparator 判断相同的对象，调用 equals 方法不一定相同。

要解决这个问题也很简单，只需要调整一下 Comparator 比较的具体逻辑，让其兼容 equals 方法就可以了。

### The SortedSet Interface

SortedSet 是一种元素按照某种顺序的 Set。在构造的时候可以通过传递相关参数控制是使用元素自身的自然顺序还是通过 Comparator 提供的顺序。

除了 Set 提供的基本操作外，SortedSet 还定义了以下几种操作：

- Range view（范围操作）：允许对有序 set 做任何范围操作；
- Endpoints：返回有序集合的头部或尾部元素；
- Comparator access：如果存在有序集合采用的比较器，则返回该 Comparator；

```java
public interface SortedSet<E> extends Set<E> {
    // Range-view
    SortedSet<E> subSet(E fromElement, E toElement);
    SortedSet<E> headSet(E toElement);
    SortedSet<E> tailSet(E fromElement);

    // Endpoints
    E first();
    E last();

    // Comparator access
    Comparator<? super E> comparator();
}
```

### The SortedMap Interface

SortedMap 是一种 entries 有序的 Map。在构建实例时可以通过传递参数选择是使用 key 的自然顺序，还是提供一个 Comparator。

SortedMap 为基础的 Map 附加了以下操作：

- Range view；任意范围操作；
- Endpoints：返回 SortedMap 的第一个或最后一个 key；
- Comparator access：返回可能存在的 Comparator；

```java
public interface SortedMap<K, V> extends Map<K, V>{
    Comparator<? super K> comparator();
    SortedMap<K, V> subMap(K fromKey, K toKey);
    SortedMap<K, V> headMap(K toKey);
    SortedMap<K, V> tailMap(K fromKey);
    K firstKey();
    K lastKey();
}
```

### Summary of Interfaces

Java Collections Framework 有以下核心接口，可以分为两种体系：

（1）第一种体系以 Collection 接口为根，衍生出 Set、List 以及 Queue；

- Set 不允许重复元素，且 null 元素只能存一个；
- List 允许存储重复元素，可以准确控制每个元素；
- Queue 中的元素通常是 FIFO 的；
- Deque 中的元素通常允许 FIFO 或 LIFO；

（2）第二种体系以 Map 接口为根，存储 key 和 value 的映射关系；

- SortedMap 将 key-value pairs 以特定顺序存储；

这些接口允许集合对元素的操作细节独立于规范。

## Aggregate Operations

可以先回顾一下 Java 的 [Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) 和 [Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

考虑这样的场景，假设构建一个社交网络 app，对于用户信息可能是这样抽象的：

```java
public class Person {

    public enum Sex {
        MALE, FEMALE
    }

    String name;
    LocalDate birthday;
    Sex gender;
    String emailAddress;
    
    // ...

    public int getAge() {
        // ...
    }

    public String getName() {
        // ...
    }
}
```

下面是使用增强 for 循环打印用户的姓名信息：

```java
for (Person p : roster) {
    System.out.println(p.getName());
}
```

使用聚合操作打印：

```java
roster
    .stream()
    .forEach(e -> System.out.println(e.getName());
```

下面介绍两类主题：

- Pipelines and Streams（管道和流）；
- 迭代器和聚合操作的不同；

### Pipelines and Streams

pipeline 是一系列聚合操作的集合。

```java
roster
    .stream()
    .filter(e -> e.getGender() == Person.Sex.MALE)
    .forEach(e -> System.out.println(e.getName()));
```

等价于 for-each：

```java
for (Person p : roster) {
    if (p.getGender() == Person.Sex.MALE) {
        System.out.println(p.getName());
    }
}
```

pipeline 包含以下几个组件：

- A source（来源）：可以是集合、数组、生成器函数或者 I/O channel；
- Zero or more `intermediate operations`（可能存在的多个中间操作）：一个中间操作，比如 filter，会产生新的流；
  - 一个 stream 是元素的序列，和集合不一样，它不是存储元素的数据结构，stream 通过 pipeline 运输来自 source 的 values。
- A `terminal operation`：终结操作，比如 forEach，会产生 non-stream 结果，比如一个原始数据类型（like a double value），或者什么值都没有（like forEach）。在上面的例子中，传给 forEach 的参数是一个 lambda expression，它会在每一个对象上调用 getName 方法，而 Java runtime 和 compiler 会自动推断该对象是 Person 类型的实例。

下面的例子计算年龄平均值：

```java
double average = roster
    .stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .mapToInt(Person::getAge)
    .average()
    .getAsDouble();
```

### Differences Between Aggregate Operations and Iterators

一个聚合操作，比如 forEach，看起来和迭代器类似，但是有以下不同之处：

- They use internal iteration：聚合操作中没有类似 next 的方法指示要处理集合的下一个元素。
  - 通过 `internal delegation` 应用程序会确定要处理的集合是什么类型，而 JDK 则会决定要如何去迭代这个集合；
  - 通过 `external iteration` 应用程序会决定要如何迭代指定类型的集合；
  - 但是，外部迭代只能按顺序遍历集合的元素，内部迭代则没有这个限制。它可以更容易地利用并行计算的优势，这包括将一个问题划分为子问题，同时解决这些问题，然后将子问题的解决结果结合起来。
- They process elements from a stream：聚合操作在流中处理元素，而不是直接处理集合本身，因此，它们也被称为 `stream operations`；
- They support behavior as parameters：聚合操作可以接收 lambda expression 作为参数。这样我们就可以为该聚合操作定制具体的行为。

### Reduction

前面有个计算平均值的例子：

```java
double average = roster
    .stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .mapToInt(Person::getAge)
    .average()
    .getAsDouble();
```

在 JDK 中流操作有这样一类终结操作（比如 average、sum、min、max 以及 count），此类操作会合并流中所有内容返回一个 value。这种操作也叫做 `reduction operations`。

除了返回单个值的，JDK 也提供返回一个集合的 reduction operations。

下面介绍：

- The Stream.reduce Method；
- The Stream.collect Method；

#### The Stream.reduce Method

[Stream.reduce](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#reduce-T-java.util.function.BinaryOperator-) 是一种通用的归约操作。参考下面的例子用于计算年龄和：

```java
Integer totalAge = roster
    .stream()
    .mapToInt(Person::getAge)
    .sum();
```

然后是等价的归约操作：

```java
Integer totalAgeReduce = roster
   .stream()
   .map(Person::getAge)
   .reduce(
       0,
       (a, b) -> a + b);
```

reduce 方法接收两个参数：

- identity：如果 stream 是空的，该参数就会作为默认的结果返回，否则作为归约操作的初始值；
- accumulator：该函数接收两个参数，它们是归约操作中要归约的内容，同时也会作为流的下一个元素，继续和后面的元素进行归约，最终会归约为一个结果。

reduce 操作总会返回一个新的值。但是，累加器函数也会在每次处理流的元素时返回一个新值。假设您希望将流的元素减少为更复杂的对象，例如集合。这可能会影响应用程序的性能。如果 reduce 操作涉及到向集合中添加元素，那么每次累加器函数处理一个元素时，它都会创建一个包含该元素的新集合，这是低效的。如果使用 `Stream.collect` 会更高效。

#### The Stream.collect Method

reduce 方法在处理元素的同时也会创建一个新的元素，和它不同的是 collect 方法可以修改、改变一个已经存在的 value；

考虑这样一种场景，要计算流中所有元素的平均值，此时我们需要两种数据：所有元素的数量以及它们的和。但是和 reduce 以及其他归约函数一样，collect 也只会返回一个 value，但是它可以创建一种新的包含所有成员变量的数据类型，并且同时保持跟踪成员的数量和它们的和。参考下面的例子：

```java
class Averager implements IntConsumer
{
    private int total = 0;
    private int count = 0;
        
    public double average() {
        return count > 0 ? ((double) total)/count : 0;
    }
        
    public void accept(int i) { total += i; count++; }
    public void combine(Averager other) {
        total += other.total;
        count += other.count;
    }
}
```

下面的 pipeline 使用 Averager 类和 collect 方法来计算平均值：

```java
Averager averageCollect = roster.stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .map(Person::getAge)
    .collect(Averager::new, Averager::accept, Averager::combine);
                   
System.out.println("Average age of male members: " +
    averageCollect.average());
```

例子中的 collect 操作接收三个参数：

- supplier：supplier 是一个工厂方法用来创建一个新的实例。在 collect 操作中，它创建归约结果的容器，也就是 Averager 实例；
- accumulator：累加器函数将流中的元素合并到结果容器中。在本例中在 Averager 上的合并操作是这样的：递增 count 值，同时将新增累加到 total 上；
- combiner：组合器函数接收两个结果容器并合并它们的内容。在本例中，它将新的 Averager 中的值累加到已有的 Averager 结果容器中。

有以下注意点：

- supplier 是一个 lambda expression（或者方法引用）而不是一个值；
- accumulator 和 combiner 函数不会返回值；
- 可以在并行流中使用 collect 操作。（在 parallel stream 中调用 collect 方法，JDK 会自动创建新的现在调用 combiner function 创建新对象，因此不必担心同步问题）；

除了例子中这种计算平均值的方式，JDK 也提供了 average 操作，只不过例子中定制化程度比较高，可以根据实际情况选择使用。

下面展示 collect 操作：

```java
List<String> namesOfMaleMembersCollect = roster
    .stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .map(p -> p.getName())
    .collect(Collectors.toList());
```

该例子中 collect 接收一个 Collector 参数，该参数封装了一个函数，和前面的 collect 接收的参数一样：supplier、accumulator 和 combiner。

`java.util.stream.Collectors` 工具类提供了很多拥有的 reduction operations，例如，将元素积累到集合中，并根据各种标准汇总元素。这些归约操作返回类 Collector 的实例，因此您可以将它们用作收集操作的参数。

以下示例按性别对成员进行分组：

```java
Map<Person.Sex, List<String>> namesByGender =
    roster
        .stream()
        .collect(
            Collectors.groupingBy(
                Person::getGender,                      
                Collectors.mapping(
                    Person::getName,
                    Collectors.toList())));
```

groupingBy 操作在例子中接收两个参数，一个分类器函数和一个 Collector 实例（叫做 `downstream collector`），这是Java运行时应用于另一个收集器的结果的收集器。因此，生成的流只包含成员的名称。

如果一个 pipeline 包含至少一个 downstream collector，就叫做 `multilevel reduction`；（多级归约）。

```java
Map<Person.Sex, Integer> totalAgeByGender =
    roster
        .stream()
        .collect(
            Collectors.groupingBy(
                Person::getGender,                      
                Collectors.reducing(
                    0,
                    Person::getAge,
                    Integer::sum)));
```

该例中的 reduce 操作接收三个参数：

- identity：和 Stream.reduce 操作一样，identity 参数既是初始值也是默认值；
- mapper：reducing 操作对流中所有元素使用该映射函数；
- operation：这个 operation function 用来归约上面的 mapped values。

### Parallelism

并行计算包括将一个问题划分为多个子问题，同时对子问题求解（在并行处理中，每个子问题都在单独的线程中处理），然后合并所有子问题的解，最终求得原始问题的解。Java SE 提供了 [fork/join framework](https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html) 用于更方便的实现并行计算。但是，使用该框架时必须指定划分子问题的方式，通过聚合操作，Java runtime 可以处理 partitioning 并且 combining 所有结果。

使用并行流处理集合的一个难点是集合并不是 thread-safe 的，这就意味着如果没有引入 [thread interference](https://docs.oracle.com/javase/tutorial/essential/concurrency/interfere.html) 和 [memory consistency errors](https://docs.oracle.com/javase/tutorial/essential/concurrency/memconsist.html) 时，多个线程是无法同时操作同一个集合的。Java 的集合框架提供了集合的 [synchronization wrappers](https://docs.oracle.com/javase/tutorial/collections/implementations/wrapper.html)，它为集合添加了自动同步处理使其线程安全。但是同步也会导致 [thread contention](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html#thread_contention)。开发者应当注意避免线程竞争，因为它会阻止并行计算。聚合操作和并行流允许开发者在 non-thread-safe 的集合上实现并行性，当然前提是不会涉及到修改集合。

要注意有时候并行计算并不比串行计算更快，在拥有大数据量以及多核时采用并行计算效率更高，可以根据实际情况选择。

本小节介绍以下内容：

- Executing Streams in Parallel；（并行流）
- Concurrent Reduction；（并发归约）
- Ordering；（排序）
- Side Effects：（副作用）
  - Laziness；（惰性）
  - Interference；（干扰）
  - Stateful Lambda Expressions；（具有状态的 Lambda 表达式）；

#### Executing Streams in Parallel

可以通过串行或并行的方式执行 streams。当并行执行流时，Java Runtime 将流划分为多个区域形成多个子流。聚合操作并行遍历和处理这些子流，然后组合结果。

在创建流时，如果没有特别声明就会创建串行流，要创建并行流，只需要调用 `Collection.parallelStream` 方法即可，或者调用 `BaseStream.parallel` 方法。

参考下面的例子：

```java
double average = roster
    .parallelStream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .mapToInt(Person::getAge)
    .average()
    .getAsDouble();
```

#### Concurrent Reduction

再看下面的例子，根据姓名进行分组，这个例子使用 collect 方法：

```java
Map<Person.Sex, List<Person>> byGender =
    roster
        .stream()
        .collect(
            Collectors.groupingBy(Person::getGender));
```

下面是等价的并行处理：

```java
ConcurrentMap<Person.Sex, List<Person>> byGender =
    roster
        .parallelStream()
        .collect(
            Collectors.groupingByConcurrent(Person::getGender));
```

这就叫做 `concurrent reduction`，对于包含 collect 操作的特定管道，如果以下所有条件都为真，则 Java 运行时执行并发归约：

- 流是并行流；
- collect 操作接收的参数：Collector 具有这样的属性，[Collector.Characteristics.CONCURRENT](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.Characteristics.html#CONCURRENT)，可以使用 `Collector.characteristics` 方法判断一个收集器的相关特性；
- 要么 stream 是无需的，要么收集器具有 `Collector.Characteristics.UNORDERED` 特性。可以调用 `BaseStream.unordered` 操作判断流是否是无需的。

#### Ordering

管道处理流元素的顺序取决于流是串行执行还是并行执行、流的来源以及中间操作。例如，考虑下面的例子，它使用 forEach 操作多次打印 ArrayList 中存储的元素：

```java
Integer[] intArray = {1, 2, 3, 4, 5, 6, 7, 8 };
List<Integer> listOfIntegers =
    new ArrayList<>(Arrays.asList(intArray));

System.out.println("listOfIntegers:");
listOfIntegers
    .stream()
    .forEach(e -> System.out.print(e + " "));
System.out.println("");

System.out.println("listOfIntegers sorted in reverse order:");
Comparator<Integer> normal = Integer::compare;
Comparator<Integer> reversed = normal.reversed(); 
Collections.sort(listOfIntegers, reversed);  
listOfIntegers
    .stream()
    .forEach(e -> System.out.print(e + " "));
System.out.println("");
     
System.out.println("Parallel stream");
listOfIntegers
    .parallelStream()
    .forEach(e -> System.out.print(e + " "));
System.out.println("");
    
System.out.println("Another parallel stream:");
listOfIntegers
    .parallelStream()
    .forEach(e -> System.out.print(e + " "));
System.out.println("");
     
System.out.println("With forEachOrdered:");
listOfIntegers
    .parallelStream()
    .forEachOrdered(e -> System.out.print(e + " "));
System.out.println("");
```

这个例子中有 5 个管道，输出的结果类似下面这样：

```java
listOfIntegers:
1 2 3 4 5 6 7 8
listOfIntegers sorted in reverse order:
8 7 6 5 4 3 2 1
Parallel stream:
3 4 1 6 2 5 7 8
Another parallel stream:
6 3 1 5 7 8 4 2
With forEachOrdered:
8 7 6 5 4 3 2 1
```

解释：

- 第一个管道按照元素加入集合的顺序打印；
- 第二个管道通过 Collections.sort 方法逆序后打印所有元素；
- 第三个和第四个管道显然是随机打印所有元素。记得前面提到过流操作使用 internal iteration 处理流中的元素。因此，当您并行执行流时，Java 编译器和运行时将确定处理流元素的顺序，以最大化并行计算的好处，除非流操作另有指定；
- 第五个管道使用方法 `forEachOrdered`，它按照源指定的顺序处理流的元素，而不管以串行还是并行的方式执行流。注意，如果对并行流使用类似 forEachOrdered 的操作，可能会失去并行的好处。

### Side Effects

如果一个方法或表达式除了返回或产生一个值之外，还修改了计算机的状态，那么它就会产生副作用。

例如，mutable reductions（前面提到过）或者调用 `System.out.println()` 方法用来 debugging。JDK 很好地处理了管道中的某些副作用。特别是，collect 方法被设计为以并行安全的方式执行最常见的有副作用的流操作。forEach 和 peek 等操作是针对副作用设计的；返回 void 的 lambda 表达式，在表达式中调用 System.out.println 打印，除了副作用什么都做不了。即便如此，你也应该小心使用 forEach 和 peek 操作；如果您对并行流使用这些操作之一，那么 Java 运行时可能会从多个线程并发调用您指定为参数的 lambda 表达式。此外，永远不要将在 filter 和 map 等操作中将有副作用的 lambda 表达式作为参数传递。后面会介绍 interference 和 stateful lambda expressions，这两者都可能是副作用的来源，并可能返回不一致或不可预测的结果，特别是在并行流中。下面会先介绍 laziness，因为它会直接影响到 interference。

#### Laziness

所有的中间操作（intermediate operation）都是 lazy 的。如果表达式、方法或算法仅在需要时才计算其值，则该表达式、方法或算法为 lazy（如果一个算法被立即评估或处理，那么它就是 eager 的）。中间操作是懒惰的，因为它们直到终端操作（terminal operations）开始才开始处理流的内容。延迟处理流使 Java 编译器和运行时能够优化它们处理流的方式。

比如说，有一个 pipeline 以这样的顺序：filter-mapToInt-average 进行处理。mapToInt 从 filter 操作中获取元素，而 average 则获取从 mapToInt 产生的几个整数。average 操作会重复进行多次直到从流中获取所需的所有元素，然后计算平均值。

#### Interference

流操作中的 lambda expressions 应该是 `not interfere` 的（不会干扰）。在 pipeline 处理流的时候，流的 source 如果被修改了就会产生干扰（interference）。比如说，下面的代码试图将 List 中的所有 String 元素连接起来。但是，它会抛出一个 `ConcurrentModificationException`：

```java
try {
    List<String> listOfStrings =
        new ArrayList<>(Arrays.asList("one", "two"));
         
    // This will fail as the peek operation will attempt to add the
    // string "three" to the source after the terminal operation has
    // commenced. 
             
    String concatenatedString = listOfStrings
        .stream()
        
        // Don't do this! Interference occurs here.
        .peek(s -> listOfStrings.add("three"))
        
        .reduce((a, b) -> a + " " + b)
        .get();
                 
    System.out.println("Concatenated string: " + concatenatedString);
         
} catch (Exception e) {
    System.out.println("Exception caught: " + e.toString());
}
```

例子中字符串 List 通过 reduce 终结操作连接所有的元素并获得一个 `Optional<String>` 对象。但是 pipeline 调用了一个中间操作 peek，该操作试图向 stream 的 source 也即 listOfStrings 添加一个元素。记住，所有的中间操作都是 lazy 的。这就意味着例子中调用 get 方法后，pipeline 才开始处理，当 get 方法执行完成后，pipeline 也就结束了。peek 操作试图在 pipeline 执行时修改 stream source，这会导致 Java runtime 抛出 ConcurrentModificationException。

#### Stateful Lambda Expressions

在 stream 操作中要避免使用 `stateful lambda expressions` 作为参数。有状态 lambda 表达式的结果依赖于在管道执行期间可能改变的任何状态。下面的例子尝试通过 map 中间操作将 List listOfIntegers 中的元素添加到一个新的 List 实例中。它会执行两次，分别使用穿行流和并行流：

```java
List<Integer> serialStorage = new ArrayList<>();
     
System.out.println("Serial stream:");
listOfIntegers
    .stream()
    
    // Don't do this! It uses a stateful lambda expression.
    .map(e -> { serialStorage.add(e); return e; })
    
    .forEachOrdered(e -> System.out.print(e + " "));
System.out.println("");
     
serialStorage
    .stream()
    .forEachOrdered(e -> System.out.print(e + " "));
System.out.println("");

System.out.println("Parallel stream:");
List<Integer> parallelStorage = Collections.synchronizedList(
    new ArrayList<>());
listOfIntegers
    .parallelStream()
    
    // Don't do this! It uses a stateful lambda expression.
    .map(e -> { parallelStorage.add(e); return e; })
    
    .forEachOrdered(e -> System.out.print(e + " "));
System.out.println("");
     
parallelStorage
    .stream()
    .forEachOrdered(e -> System.out.print(e + " "));
System.out.println("");
```

这里面用到的 lambda 表达式：

```java
e -> {
    parallelStorage.add(e);
    return e;
}
```

就是一个 stateful lambda expression。每次运行代码时，其结果都可能不同。输出可能是这样的：

```java
Serial stream:
8 7 6 5 4 3 2 1
8 7 6 5 4 3 2 1
Parallel stream:
8 7 6 5 4 3 2 1
1 3 6 2 4 5 8 7
```

forEachOrdered 操作按照流指定的顺序处理元素，而不管流是串行执行还是并行执行。但是，当流并行执行时，map 操作处理 Java 运行时和编译器指定的流元素。因此，lambda表达式 `e -> { parallelStorage.add(e); return e;` 向 parallelStorage 中添加元素就会在每次代码运行时发生变化。对于确定性和可预测的结果，请确保流操作中的 lambda 表达式参数不是有状态的。

注意：例子中使用了 `synchronizedList` 所以 `List parallelStorage` 是线程安全的。要记住集合不是线程安全的。这意味着多个线程不应该同时访问特定的集合。假设我们创建了常规的 not-thred-safe 的集合：

```java
List<Integer> parallelStorage = new ArrayList<>();
```

再执行前面的例子，程序的结果就是不可预测的，因为多个线程并发修改了 parallelStorage 而没有像同步这样的机制来安排特定线程何时可以访问 List 实例。因此，该示例可以打印类似以下的输出：

```java
Parallel stream:
8 7 6 5 4 3 2 1
null 3 5 4 7 8 1 2
```



## Implementations

本章介绍 Java Collection Framework core interfaces 的部分实现：

- 通用实现：使用最广泛的集合；
- 特殊实现：适用特殊的场景，拥有 nonstandard performance、特征、使用限制或者行为；
- 并发实现：为支持高并发设计的集合，这些实现是 `java.util.concurrent` 包的一部分；
- 包装实现：结合其他实现（通常是通用实现提供的几种集合）一起使用。提供附加的或受限制的功能；
- 便利实现：是一些 mini-implementations，通常通过静态工厂方法提供，这些方法为特殊集合(例如，只有一个元素的 set)的通用目的实现提供了方便、高效的替代方案；
- 抽象实现：定义了一种集合的通用骨架，方便开发者扩展和自定义集合。这是一个高级的话题，不是特别难，但是相对来说很少有人需要做。

通用实现如下表所示：

| Interfaces | Hash table Implementations | Resizable array Implementations | Tree Implementations | Linked list Implementations | Hash table + Linked list Implementations |
| ---------- | -------------------------- | ------------------------------- | -------------------- | --------------------------- | ---------------------------------------- |
| `Set`      | `HashSet`                  |                                 | `TreeSet`            |                             | `LinkedHashSet`                          |
| `List`     |                            | `ArrayList`                     |                      | `LinkedList`                |                                          |
| `Queue`    |                            |                                 |                      |                             |                                          |
| `Deque`    |                            | `ArrayDeque`                    |                      | `LinkedList`                |                                          |
| `Map`      | `HashMap`                  |                                 | `TreeMap`            |                             | `LinkedHashMap`                          |

如上表所示，Java Collections Framework 为 Set、List 和 Map 提供了一些通用的实现。比如 HashSet、ArrayList 和 HashMap 在大多数情况下都非常合适去使用它们。

注意 `SortedSet` 和 `SortedMap` 没有出现在上表中。但是它们的实现 TreeSet 和 TreeMap 归属到了 Set 和 Map 的所属行，这里还有 Queue 的两种通用实现：`LinkedList`（它也是 List 的实现），以及 `PriorityQueue`（没有出现在表中）。这两种实现提供了不同的语义：LinkedList 提供 FIFO 语义，而 PriorityQueue 则根据元素进行排序。

- 每个通用实现都提供其接口中包含的所有可选操作；
- 都允许空元素、键和值；
- 没有同步保证（即线程不安全）；
- 都拥有 fail-fast 的迭代器，在迭代过程中一旦发现非法并发修改操作就会立即失败，而不会做其他操作；
- 都实现了 `Serializabel` 接口并且都支持 `clone` 方法；

在过去版本的 JDK 集合中，Vector 和 Hashtable 都是同步的，而新的集合框架都是不同步的。因为在某些时候采取同步没有任何好处，比如在单线程中使用集合且只进行读操作。此外，在某些情况下，不必要的同步可能导致死锁。

如果你需要使用线程安全的集合，可以使用 synchronization wrappers，允许将任何集合转换为同步集合。因此，同步对于通用实现是可选的，而对于遗留实现是必须的。

此外，`java.util.concurrent` 包提供了扩展 Queue 的 BlockingQueue 接口和扩展 Map 的 ConcurrentMap 接口的并发实现。这些实现比单纯的同步实现提供了更高的并发性。

通常，您应该考虑接口，而不是实现。在大多数情况下，实现的选择只影响性能。

### Set Implementations

Set 实现分为两类：通用实现和特殊实现；

#### General-Purpost Set Implementation

Set 有三种通用实现：

- HashSet：比 TreeSet 要快得多（对大多数操作提供对数时间甚至常数时间），但是不提供有序元素集。
- TreeSet：如果需要使用 SortedSet 接口，或者需要按照元素顺序迭代集合，可以使用 TreeSet。
- LinkedHashSet：LinkedHashSet 在某种意义上介于 HashSet 和 TreeSet 之间。它使用一个链表链接 hash table，并且可以根据元素插入时的顺序去迭代它，运行速度几乎和 HashSet 一样快。LinkedHashSet 实现将其客户端从 HashSet 提供的未指定的、通常是混乱的排序中解脱出来，而不会导致与 TreeSet 相关的成本增加。

关于 HashSet 有一点需要注意：线性迭代是 entries 的总数量和 buckets（the capacity）的总数量之和。因此，选择过高的初始容量会浪费空间和时间。另一方面，选择一个太低的初始容量会浪费时间，因为每次被迫增加容量时都会复制数据结构。如果没有声明初始容量，则会使用默认值 16。在过去，选择质数作为初始容量有一定的优势。这已经不是事实了。在内部，容量总是四舍五入到 2 的幂。初始容量由int构造函数指定。下面这行代码分配了一个初始容量为 64 的 HashSet。

```java
Set<String> s = new HashSet<String>(64);
```

HashSet 类有一个优化参数叫做 `load factor`。如果很在意 HashSet 的空间消耗，可以阅读 HashSet 的文档获取更多信息。否则使用默认值就可以了。

LinkedHashSet 具有与 HashSet 相同的调优参数，但迭代时间不受容量的影响。TreeSet 没有调优参数。

#### Special-Purpost Set Implementations

Set 有两种特殊实现：

- EnumSet：是 Set 为枚举类型提供的高性能实现。存储的所有元素必须是相同类型的枚举。在内部，由一个 bit-vector 表示，通常是一个 long。枚举集支持枚举类型范围内的迭代。

比如，定义一周的天数的枚举，可以迭代所有工作日，EnumSet 提供了一个静态工厂方法：

```java
 for (Day d : EnumSet.range(Day.MONDAY, Day.FRIDAY))
        System.out.println(d);
```

枚举集还为传统的位标志提供了丰富的、类型安全的替代：

```java
   EnumSet.of(Style.BOLD, Style.ITALIC)
```

- CopyOnWriteArraySet 是 Set 的一种实现，可以存储 `copy-on-write`（写入时复制）数组，所有的可变操作，比如 add、set 以及 remove，都会生成原数组的复制而不需要任何锁定操作。甚至迭代也可以安全地与元素插入和删除同时进行。与大多数 Set 实现不同，添加、删除和包含方法所需的时间与 Set 的大小成正比。这种实现只适用于很少修改但经常迭代的集合。它非常适合于维护必须防止重复的事件处理程序列表。

### List Implementations

List 实现可以分为通用实现和特殊实现；

#### General-Purpost List Implementations

有两种 List 的通用实现：

- ArrayList：
  - 支持常量级查询时间复杂度，可以使用 `System.arraycopy` 复制数组中的多个元素到另一个数组；
- LinkedList：
  - 如果需要频繁向 List 中插入和删除元素，可以使用 LinkedList，这些操作在 LinkedList 是常量时间复杂度的，而在 ArrayList 中是线性时间复杂度。

增删操作在 ArrayList 中需要付出很大的代价，但是在查找上是常量时间的（随机访问），在 LinkedList 中查找元素是线性时间的，而增删元素确是常量时间。

ArrayList 有一个优化参数：`initial capacity`。它指的是数组列表在增加之前可以容纳的元素数量。

LinkedList 没有优化参数，但是提供了七个附加操作，比如 clone，其他操作则是在 Queue 接口中定义的。

#### Special-Purpost List Implementations

CopyOnWriteArrayList 是 List 的一种特殊实现，它通过 `copy-on-write` 的原则存储一个数组。该实现和 CopyOnWriteArraySet 类似。它不需要同步包装，即使在迭代的时候也不会抛出 ConcurrentModificationException。

这种实现非常适合维护事件处理程序列表（event-handler lists），因为在事件处理程序列表中更改不频繁，遍历很频繁，而且可能很耗时。

如果需要同步支持，可以考虑使用 Vector，它比 Collections.synchronizedList 封装的 ArrayList 要快一些，但是有一些遗留的问题需要注意。

如果想要大小固定的 List，可以考虑使用 Arrays.asList 方法。



### Map Implementations

Map 的实现分为：通用实现、特殊实现以及并发实现；

#### General-Purpost Map Implementations

Map 有三个通用实现：

- HashMap：
  - 如果需要非常快的速度且不考虑迭代的顺序，可以使用 HashMap；
- TreeMap：
  - 如果想使用 SortedMap 的行为或者 key 有序的集合视图，可以使用 TreeMap；
- LinkedHashMap：
  - 如果既想要 HashMap 的性能也想有顺序的迭代，可以使用 LinkedHashMap。

从上面几个实现来看，Map 和 Set 的通用实现有些类似。

LinkedHashMap 提供了两个 LinkedHashSet 没有的功能。

- 创建 LinkedHashMap 时，可以指定根据 key 的访问顺序或者插入顺序，换句话说，当我们查询某个 value 时，会将关联的 key 带到 map 的末尾；
- 同样 LinkedHashMap 提供了 `removeEldestEntry` 方法，它支持在插入新的 entry 到 map 中时，自动覆盖掉最旧的 entry；

比如下面的例子，最多允许 map 增长到 100 个 entries，然后当插入新的 entry 时自动移除最旧的 entry，保持100个条目的稳定状态。

```java
private static final int MAX_ENTRIES = 100;

protected boolean removeEldestEntry(Map.Entry eldest) {
    return size() > MAX_ENTRIES;
}
```

#### Special-Purpost Map Implementations

Map 有三种特殊实现：

- EnumMap：
  - 内部使用了一个数组，使用 enum keys 实现高性能的 map。这种实现将 Map 接口的丰富性和安全性与接近数组的速度相结合。如果你想要将一个枚举映射到一个值，你应该总是优先使用 EnumMap 而不是数组。
- WeakHashMap：
  - WeakHashMap 是 Map 的一种实现，它的 keys 是弱引用，当在 WeakHashMap 之外没有其他地方引用某个 key 时，垃圾回收机制就会回收该 key 对应的键值对。该类提供了利用弱引用功能的最简单方法。它对于实现“类似注册表”的数据结构很有用，在这种结构中，当任何线程都无法访问条目的键时，条目的实用功能就会消失。
- IdentityHashMap：
  - 它是基于 hash table 的 identity-based 的 Map 实现。该类对于保持拓扑的对象图转换 (如序列化或深度复制) 非常有用。要执行这样的转换，您需要维护一个基于身份的“节点表”，以跟踪已经看到的对象。在动态调试器和类似的系统中，基于身份的映射也用于维护对象到元信息的映射。最后，基于身份的映射在抵御 “欺骗攻击” 方面很有用，这种攻击是由故意错误的 equals 方法造成的，因为 IdentityHashMap 从不在其键上调用 equals 方法。这种实现的另一个好处是速度快。



#### Concurrent Map Implementations

`java.util.concurrent` 包下面包含 ConcurrentMap 接口，它扩展了 Map 体系并且添加了 putIfAbsent、remove 和 replace 方法。ConcurrentHashMap 是该接口的实现。

ConcurrentHashMap 是一个由 hash 表实现的高并发、高性能的 Map 实现。此实现在执行检索时不会阻塞，并允许客户端选择 update 的并发级别。它的目的是作为 Hashtable 的直接替代品:除了实现 ConcurrentMap 之外，它还支持 Hashtable 特有的所有遗留方法。同样，如果您不需要遗留操作，请小心使用 ConcurrentMap 接口操作它。



### Queue Implementations

Queue 的实现分为两类：普通实现和并发实现。

#### General-Purpost Queue Implementations

前面提到过，LinkedList 实现了 Queue 接口，提供了 add、poll 的 FIFO 操作，等等。

PriorityQueue 类是基于堆数据结构的优先级队列，在构建 PriorityQueue 时可以指定是使用元素的自然顺序还是提供一个 Comparator。

队列检索操作(轮询、删除、查看和元素)访问队列头部的元素。队列的头是相对于指定顺序的最小元素。如果多个元素绑定最小值，则头部是其中一个元素; 

PriorityQueue 及其迭代器实现了 Collection 和 Iterator 接口的所有可选方法。方法 iterator 中提供的迭代器不能保证以任何特定顺序遍历 PriorityQueue 中的元素。对于有序遍历，可以考虑使用 `Arrays.sort(pq.toArray())`。

#### Concurrent Queue Implementations

`java.util.concurrent` 包下面包含了 Queue 接口的同步实现相关的接口和类。

BlockingQueue 用一些操作扩展了 Queue，这些操作在检索元素时等待队列变为非空，在存储元素时等待队列中的空间变为可用。该接口由以下类实现：

- `LinkedBlockingQueue`：由链接节点支持的可选有界 FIFO 阻塞队列；
- `ArrayBlockingQueue`：由数组支持的有界 FIFO 阻塞队列；
- `PriorityBlockingQueue`：由堆支持的无界阻塞优先队列；
- `DelayQueue`：由堆支持的基于时间的调度队列；
- `SynchronousQueue`： a simple rendezvous mechanism that uses the `BlockingQueue` interface. （基于 BlockingQueue 接口实现的简单会合机制）；

在JDK 7中，`TransferQueue` 是一个专门化的 `BlockingQueue`，其中向队列中添加元素的代码可以选择等待(阻塞)另一个线程中的代码来检索元素。`TransferQueue `有一个单独的实现：

- `LinkedTransferQueue`：基于链接节点的无界`TransferQueue`；

### Deque Implementations

Deque 接口表示双端队列。Deque 接口的实现可以提供替代某些 Collection 实现的做用。它的实现分为两类：通用实现和并发实现。

#### General-Purpose Deque Implementations

通用实现包括 LinkedList 和 ArrayDeque 类。Deque 接口支持在两端插入、删除和检索元素。

ArrayDeque 类是  Deque 接口的可变大小的数组实现，而LinkedList 类是 List 的实现。

Deque 接口中的基本插入、删除和检索操作 addFirst、addLast、removeFirst、removeLast、getFirst 和 getLast。方法 addFirst 在 Deque 实例的头部添加一个元素，而 addLast 在尾部添加一个元素。

LinkedList 实现比 ArrayDeque 实现更灵活。LinkedList 实现了所有可选的 List 操作。在 LinkedList 实现中允许空元素，但在 ArrayDeque 实现中不允许。

在效率方面，ArrayDeque 在两端的添加和删除操作上比 LinkedList 更高效。LinkedList 实现中的最佳操作是在迭代期间删除当前元素。LinkedList 实现并不是迭代的理想结构。

LinkedList 实现比 ArrayDeque 实现消耗更多的内存。对于 ArrayDeque 实例遍历，可以使用以下任意方法:

（1）foreach

foreach 是快速的，可以用于各种类型的列表。

```java
ArrayDeque<String> aDeque = new ArrayDeque<String>();

. . .
for (String str : aDeque) {
    System.out.println(str);
}
```

（2）Iterator

迭代器可用于在各种类型的列表上对各种类型的数据进行前向遍历。

```java
ArrayDeque<String> aDeque = new ArrayDeque<String>();
. . .
for (Iterator<String> iter = aDeque.iterator(); iter.hasNext();  ) {
    System.out.println(iter.next());
}
```



#### Concurrent Deque Implementations

LinkedBlockingDeque 是 Deque 的并发实现。如果 deque 为空，则 takeFirst 和 takeLast 等方法会等待该元素变得可用，然后检索并删除相同的元素。

### Wrapper Implementations

包装器实现将其所有实际工作委托给指定的集合，但在该集合提供的功能之上添加额外的功能。对于喜欢设计模式的人来说，这是一个装饰器模式的例子。

这些实现是匿名的；标准库提供的不是公共类，而是静态工厂方法。所有这些实现都可以在 Collections 类中找到，该类仅由静态方法组成。

#### Synchronization Wrappers

同步包装器向任意集合添加自动同步(线程安全)。六个核心集合接口 (collection、Set、List、Map、SortedSet 和 SortedMap) 中的每一个都有一个静态工厂方法。

```java
public static <T> Collection<T> synchronizedCollection(Collection<T> c);
public static <T> Set<T> synchronizedSet(Set<T> s);
public static <T> List<T> synchronizedList(List<T> list);
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m);
public static <T> SortedSet<T> synchronizedSortedSet(SortedSet<T> s);
public static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> m);
```

这些方法都返回一个同步的(线程安全的)集合，该集合由指定的集合备份。为了保证串行访问，对备份集合的所有访问都必须通过返回的集合来完成。保证这一点的简单方法是不保留对备份集合的引用。使用以下技巧创建同步集合。

```java
List<Type> list = Collections.synchronizedList(new ArrayList<Type>());
```

以这种方式创建的集合与正常同步的集合 (如 Vector ) 一样是线程安全的。

面对并发访问，当迭代返回的集合时，用户必须手动同步它。原因是迭代是通过对集合的多次调用来完成的，而集合必须组合成单个原子操作。下面是对包装同步集合进行迭代的习惯用法。

```java
Collection<Type> c = Collections.synchronizedCollection(myCollection);
synchronized(c) {
    for (Type e : c)
        foo(e);
}
```

如果使用了显式迭代器，则必须在同步块内调用迭代器方法。不遵循此建议可能导致不确定性行为。在同步映射的 Collection 视图上迭代的习惯用法与此类似。当迭代任何 Collection 视图时，用户必须在 synchronized Map 上同步，而不是在 Collection 视图本身上同步，如下面的示例所示。

```java
Map<KeyType, ValType> m = Collections.synchronizedMap(new HashMap<KeyType, ValType>());
    ...
Set<KeyType> s = m.keySet();
    ...
// Synchronizing on m, not s!
synchronized(m) {
    while (KeyType k : s)
        foo(k);
}
```

使用包装器实现的一个小缺点是您不能执行包装实现的任何非接口操作。因此，例如，在前面的 List 示例中，您不能在包装的ArrayList 上调用 ArrayList 的 ensureCapacity 操作。

#### Unmodifiable Wrappers

同步包装器将同步功能添加到被包装的集合中，而不可修改的包装器则移除了某些原有的功能。特别是，它们通过拦截将修改集合的所有操作并抛出UnsupportedOperationException来剥夺修改集合的能力。不可修改的包装器有以下两个主要用途：

- 使集合在构建后不可变。在这种情况下，最好不要维护对支持集合的引用。这绝对保证了不变；
- 允许某些客户端只读访问您的数据结构。您保留了对支持集合的引用，但分发了对包装器的引用。通过这种方式，客户端可以查看但不能修改，而您可以保持完全访问。

与同步包装器一样，六个核心Collection接口中的每一个都有一个静态工厂方法。

```java
public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c);
public static <T> Set<T> unmodifiableSet(Set<? extends T> s);
public static <T> List<T> unmodifiableList(List<? extends T> list);
public static <K,V> Map<K, V> unmodifiableMap(Map<? extends K, ? extends V> m);
public static <T> SortedSet<T> unmodifiableSortedSet(SortedSet<? extends T> s);
public static <K,V> SortedMap<K, V> unmodifiableSortedMap(SortedMap<K, ? extends V> m);
```

#### Checked Interface Wrappers

`Collections.checked `接口包装器提供给泛型集合使用。这些实现返回指定集合的动态类型安全视图，如果客户端试图添加错误类型的元素，该视图将抛出 ClassCastException。该语言中的泛型机制提供了编译时(静态)类型检查，但也有可能破坏这种机制。动态类型安全视图完全消除了这种可能性。

### Convenience Implementations

本节将介绍几个小型实现，当您不需要它们的全部功能时，它们比通用实现更方便、更高效。本节中的所有实现都是通过静态工厂方法而不是公共类提供的。

#### List View of an Array

`Arrays.asList` 方法返回接收的数组参数的集合视图。对 List 的更改将写入数组，反之亦然。集合的大小就是数组的大小，不能更改。如果在 List 上调用了 add 或 remove 方法，则会导致UnsupportedOperationException。

这个实现的正常用途是作为基于数组和基于集合的 api 之间的桥梁。它允许您将数组传递给需要 Collection 或 List 的方法。然而，这个实现还有另一个用途。如果您需要一个固定大小的List，那么它比任何通用的 List 实现都更有效。

```java
List<String> list = Arrays.asList(new String[size]);
```

注意，对原始数组的引用不会被保留。

#### Immutable Multiple-Copy List

偶尔你会需要一个由同一元素的多个副本组成的不可变 List。`Collections.nCopies` 返回这样的 List。这个实现有两个主要用途。第一个是初始化一个新创建的 List；例如，假设你想要一个初始包含 1000 个空元素的数组列表。像下面这样：

```java
List<Type> list = new ArrayList<Type>(Collections.nCopies(1000, (Type)null));
```

当然，每个元素的初始值不必为空。第二个主要用途是增长一个现有的 List。例如，假设您想在 `List< string >` 的末尾添加 69 个字符串 “fruit bat” 的副本。不清楚你为什么要做这样的事情，但让我们假设你做了。下面是你该怎么做。

```java
lovablePets.addAll(Collections.nCopies(69, "fruit bat"));
```

通过使用接受索引参数和 Collection 参数的 addAll 方法，可以将新元素添加到 List 的中间，而不是到列表的末尾。

#### Immutable Singleton Set

有时您需要一个不可变的单例集，它由单个指定的元素组成。`Collections.singleton` 方法返回这样的 Set。此实现的一个用途是从集合中删除所有指定元素的出现。

```java
c.removeAll(Collections.singleton(e));
```

相关的习惯用法从 map 中删除映射到指定值的所有元素。例如，假设您有一个Map - job，它将人们映射到他们的工作领域，并假设您希望消除所有的律师。下面的一行代码就可以做到这一点。

```java
job.values().removeAll(Collections.singleton(LAWYER));
```

此实现的另一个用途是向编写为接受值集合的方法提供单个输入值。

#### Empty Set，List，and Map Constants

Collection s类提供了返回空 Set、List 和 Map 的方法：emptySet、emptyList 以及 emptyMap。当您根本不想提供任何值时，这些常量的主要用途是作为方法的输入，这些方法接受一个值集合，如本例所示。

```java
tourist.declarePurchases(Collections.emptySet());
```

### Summary of Implementations

Java 集合框架提供核心接口的一些通用实现：

- 对于 Set 接口，HashSet 用的最广泛；
- 对于 List 接口，ArrayList 使用的最广泛；
- 对于 Map 接口，HashMap 使用的最广泛；
- 对于 Queue 接口，LinkedList 使用的最广泛；
- 对于 Deque 接口，ArrayDeque 使用的最广泛；

每个通用实现都提供其接口中包含的所有可选操作。

Java Collections Framework 还为需要非标准性能、使用限制或其他不寻常行为的情况提供了一些特殊用途的实现。

java.util.concurrent 包包含几个集合实现，它们是线程安全的，但不受单个排除锁的控制。

Collections 类 (与 Collection 接口相反) 提供了操作或返回集合的静态方法，这些方法被称为 Wrapper 实现。

最后，还有几种便捷性实现，当您不需要它们的全部功能时，它们可能比通用目的的实现更高效。便利实现是通过静态工厂方法提供的。



进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html
- https://docs.oracle.com/javase/tutorial/collections/index.html
- https://docs.oracle.com/javase/tutorial/collections/algorithms/index.html