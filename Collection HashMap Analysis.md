# 前言

研究 Java HashMap 源码，了解相关特性，以及在 JDK 7 和 JDK 8 中的不同实现方式，切换方式的原因。



后续工作：

（1）研究引用对象；

（2）研究 Java 的并发相关工具；



# Map

`java.util.Map` 接口是早期 `java.util.Dictionary` 抽象类的替代品，它是一种存储 keys 和 values 映射关系的对象，且不允许包含重复的 key，每个 key 可以关联至少一种对象（对应单个 Object 或 Collection）。

Map 接口提供了一些 `collection-view`（集合视图），包括 keys 的 set，values 的 collection，以及 key-value pair 的 set。当我们在 map 的集合视图上调用迭代器方法进行遍历时，相关的顺序是迭代器返回的元素顺序，而有一种特殊的 Map 实现 `TreeMap` 则自己维持了一套顺序，而 `HashMap` 则没有相关的顺序保证。

一些特殊的 Map 实现对 key 和 value 可能有一些限制。比如说禁止有 null key 和 value 存在，有的对 key 的类型有限制，试图插入不符合条件的键或值会抛出未检查的异常，通常是 NullPointerException 或 ClassCastException。

Map 接口有几个需要注意的地方：

- 定义了一些通用的查询方法；
- 定义了一些通用的修改方法；
- 定义了一些通用的大数据量操作；
- 定义了 Map 中键值对的抽象：Entry 接口；
  - Entry 接口中定义了关于从 Entry 中 get/set key/value 的方法；
  - 定义了如何比较两个 entry 对象是否相等：



# AbstractMap

`java.util.AbstractMap` 抽象类实现了 Map 接口，为 Map 实现提供了一个骨架，减少子类实现的工作量。

开发者可以通过继承该类，实现或重写某些方法，就可以实现自己的定制化 Map。

AbstractMap 除了实现了 Map 接口的部分方法外，还提供了两个继承自 Map.Entry 的静态类：

- `java.util.AbstractMap.SimpleEntry`：简单的键值对实现，其中 value 是可以通过 setValue 改变的；
- `java.util.AbstractMap.SimpleImmutableEntry`：维护不可变的键值对的 Entry。该实现不支持 setValue 方法。（在并发编程中可以返回线程安全的键值对集合）；



# HashMap（JDK 8）

## 1、简介

`java.util.HashMap` 是 Map 接口采用 hash 表的一种实现。该类提供所有 Map 相关的可选操作（之前的文章中提到过 optional operations），它允许插入 null key 和 null value；（可以简单的将 HashMap 看作没有同步保证和允许 null 的 Hashtable）

```java
@Test
public void test3() {
    Map<String, String> hashMap = new HashMap<>();
    
    hashMap.put(null, null);
    hashMap.put(null, null);
    hashMap.put(null, null);

    System.out.println(hashMap.size());
    Iterator<Map.Entry<String, String>> itr = hashMap.entrySet().iterator();
    while (itr.hasNext()) {
        Map.Entry<String, String> next = itr.next();
        System.out.printf("%s -> %s%n", next.getKey(), next.getValue());
    }
}
```

输出：

```
1
null -> null
```

哈希表的 Map 实现可以保证常量时间复杂度的 get 和 put 操作（前提是假设使用的 hash 映射函数可以很好的将元素分散到不同的 bucket 中，注：HashMap 中使用 bucket 来形容存储元素的容器。）如果想要遍历 HashMap 中的所有元素，则需要先获取相应的 collection-view，且遍历花费的时间和 HashMap 的容量即 bucket 的数量加上 HashMap 的 size 即 key-value-mapping 的数量成正比。

因此，在构建 HashMap 实例时，最好不要把初始容量（initial capacity）设置的太大，或者负载因子（load factor）设置的太小。这对迭代性能影响很大。

## 2、两个参数

在构建 HashMap 实例时有两个参数很重要，它们会影响 HashMap 的性能：

（1）initial capacity；

对应 HashMap 持有的 hash table 的 buckets 的数量；它只决定 HashMap 创建之初的 buckets 数量，后续会动态伸缩；

（2）load factor；

负载因子衡量的是在哈希表的容量自动增加之前允许增加到什么样的程度；当 hash table 中的 entry 数量超过负载因子和当前容量的乘积时，hash table 就会进行 `rehashed`（再哈希，实际上就是内容数据结构发生了改变），rehashed 后，hash table 的 buckets 数量大约是原来的两倍；

作为一个通用的规则，load factor 的默认值是 0.75，此时在时间和空间上会有很好的权衡。load factor 较高则空间消耗会降低，但是会增加查找成本（对应 HashMap 类中大多数操作都会变慢，包括 get 和 put）。

在设置 Map 的初始容量时，应考虑  Map 中的预期条目数及其负载系数，从而尽量减少发生 rehashed 的次数。**如果初始容量大于预期存储最大条目数除以负载因子，则不会发生重新散列操作**。

如果我们要使用 HashMap 存储很多的键值对，给与 HashMap 一个足够大的初始容量可以避免再哈希的发生，性能上会提高很多。

## 3、注意事项

（1）注意：在任何和 hash table 中，一旦出现了很多具有相同 `hashcode()`（哈希冲突），哈希表的性能都会下降。为了改善这种情况，当 key 是可以比较的时候（实现了 Comparable 接口，或者类似 String 那种有自然顺序），HashMap 遇到哈希冲突时会根据 key 比较的顺序来减少冲突的发生（解决 hash 冲突的一些方法，可以上网搜搜，比如链地址法、再 hash 法等等）。

（2）注意：HashMap 实现不是线程安全的，不能保证操作是同步的。如果多个线程并发的访问同一个 HashMap 实例，且对其 map 结构进行了修改，就必须从外部进行同步（修改 map 结构的操作包括增删一个或多个映射，如果仅仅是改变了一个 map 本来就有的 key 关联的 value，这就不属于修改 map 的结构）。

如果不能保证同步，则建议不要在多线程环境下使用 HashMap，可以使用其包装实现或者 Map 的并发实现。（`Collections.synchronizedMap` 或 `ConcurrentHashMap`）

（3）注意：之前也提到过 Map 的 collection-view 通过 Iterator 迭代时，采用的是 `fail-fast` 模式，一旦遇到问题就会干净利落的失败，不会造成某些恶劣影响。比如在迭代过程中使用 remove 方法移除了某个键值对，则立即抛出 `ConcurrentModificationException`。

因此，在面对并发修改时，迭代器会快速而干净地失败，而不会在未来不确定的时间出现任意的、不确定的行为。

请注意，迭代器的快速失败行为不能得到保证，一般来说，在存在不同步的并发修改时，不可能做出任何硬保证。快速失败迭代器会尽可能地抛出 ConcurrentModificationException。因此，编写依赖于这个异常的程序是错误的。`the fail-fast behavior of iterators should be used only to detect bugs.`

​	







进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/api/java/lang/ref/package-summary.html