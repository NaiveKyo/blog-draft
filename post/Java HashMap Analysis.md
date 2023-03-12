---
title: Java Collection HashMap Analysis
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425111113.jpg'
coverImg: /img/20220425111113.jpg
cover: false
toc: true
mathjax: false
date: 2023-03-12 17:33:46
summary: "Java HashMap 源码分析"
categories: "Java"
keywords: "Java"
tags: "Java"
---

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

哈希表的 Map 实现可以保证常量时间复杂度的 get 和 put 操作（前提是假设使用的 hash 映射函数可以很好的将元素分散到不同的 bucket 中，注：HashMap 中使用 bucket，也叫 bin 来形容存储元素的容器。）如果想要遍历 HashMap 中的所有元素，则需要先获取相应的 collection-view，且遍历花费的时间和 HashMap 的容量即 bucket 的数量加上 HashMap 的 size 即 key-value-mapping 的数量成正比。

因此，在构建 HashMap 实例时，最好不要把初始容量（initial capacity）设置的太大，或者负载因子（load factor）设置的太小。这对迭代性能影响很大。

## 2、两个参数

在构建 HashMap 实例时有两个参数很重要，它们会影响 HashMap 的性能：

（1）initial capacity；

对应 HashMap 持有的 hash table 的 buckets 的数量；它只决定 HashMap 创建之初的 buckets 数量，后续会动态伸缩；

（2）load factor；

负载因子衡量的是在哈希表的容量自动增加之前允许增加到什么样的程度；当 hash table 中的 entry 数量超过负载因子和当前容量的乘积时，hash table 就会进行 `rehashed`（再哈希，实际上就是内容数据结构发生了改变），rehashed 后，hash table 的 buckets 数量大约是原来的两倍；

作为一个通用的规则，load factor 的默认值是 0.75，此时在时间和空间上会有很好的权衡。load factor 较高则空间消耗会降低，但是会增加查找成本（对应 HashMap 类中大多数操作都会变慢，包括 get 和 put）。

在设置 Map 的初始容量时，应考虑  Map 中的预期条目数及其负载系数，从而尽量减少发生 rehashed 的次数。**如果初始容量大于预期存储最大条目数除以负载因子，则不会发生重新散列操作**。（`initialCapacity = (expectCapacity / loadFactor) + 1F`）

如果我们要使用 HashMap 存储很多的键值对，给与 HashMap 一个足够大的初始容量可以避免再哈希的发生，性能上会提高很多。

## 3、注意事项

（1）注意：在任何和 hash table 中，一旦出现了很多具有相同 `hashcode()`（哈希冲突），哈希表的性能都会下降。为了改善这种情况，当 key 是可以比较的时候（实现了 Comparable 接口，或者类似 String 那种有自然顺序），HashMap 遇到哈希冲突时会根据 key 比较的顺序来减少冲突的发生（解决 hash 冲突的一些方法，可以上网搜搜，比如链地址法、再 hash 法等等）。

（2）注意：HashMap 实现不是线程安全的，不能保证操作是同步的。如果多个线程并发的访问同一个 HashMap 实例，且对其 map 结构进行了修改，就必须从外部进行同步（修改 map 结构的操作包括增删一个或多个映射，如果仅仅是改变了一个 map 本来就有的 key 关联的 value，这就不属于修改 map 的结构）。

如果不能保证同步，则建议不要在多线程环境下使用 HashMap，可以使用其包装实现或者 Map 的并发实现。（`Collections.synchronizedMap` 或 `ConcurrentHashMap`）

（3）注意：之前也提到过 Map 的 collection-view 通过 Iterator 迭代时，采用的是 `fail-fast` 模式，一旦遇到问题就会干净利落的失败，不会造成某些恶劣影响。比如在迭代过程中使用 remove 方法移除了某个键值对，则立即抛出 `ConcurrentModificationException`。

因此，在面对并发修改时，迭代器会快速而干净地失败，而不会在未来不确定的时间出现任意的、不确定的行为。

请注意，迭代器的快速失败行为不能得到保证，一般来说，在存在不同步的并发修改时，不可能做出任何硬保证。快速失败迭代器会尽可能地抛出 ConcurrentModificationException。因此，编写依赖于这个异常的程序是错误的。`the fail-fast behavior of iterators should be used only to detect bugs.`

## 4、实现分析

HashMap 这种实现通常被看作 binned（bucketed）的 hash table，一旦 bins 过大，它们就会转化为 TreeNodes 组成的 bins，这种结构和 TreeMap 类似（红黑树）。HashMap 的大多数方法通常会使用正常的 bins，只有在适当的时候才会调用 TreeNode 方法（通过查看 node 的实例类型）。遍历和使用由 TreeNodes 组成的 bins 和常规的 HashMap 操作没什么区别，只不过在数据量大的时候，TreeNode 查询键值对会更快一些。

Tree bins（bin 的元素由 TreeNode 组成）中元素的顺序是由 hashCode 决定的。但是当 hashCode 相同，且 HashMap 的键实现了 Comparable 接口，就会调用它们的 compareTo 方法来决定顺序（例如 `class C implements Comparable<C>` ，会通过反射来检测泛型的实际类型）。使用 tree bins 虽然会让 Map 实现变得更复杂，但是有一点却很值得，因为当两个 key 有不同的 hashCode 或者可以比较，它就能提供最坏情况下 `O(log n)` 级别的时间复杂度。

因为 TreeNodes 的数量通常是普通 nodes 的两倍，所以说只有当 bins 包含了足够多的 node 后才会使用它（参考 `TREEIFY_THRESHOLD` 常量）。同时当 node 数量变得足够小时，TreeNodes 又会被转变为正常的 nodes（通过 `removal` 或者 `resizing`）。

如果说 Map 中元素的 hashCode 分布很均匀，一般就不会使用 tree bins。理想情况下，使用随机 hashCode，bins 中的 nodes 一般遵循[泊松分布](https://en.wikipedia.org/wiki/Poisson_distribution)。

tree bins 的第一个结点通常就是它的根结点。但是有些时候（当前只出现在 Iterator.remove 方法中），root 可能不是第一个 node，此时可以通过 `TreeNode.root()` 获取根结点。

所有适当的内部方法会接收一个 hash code 作为一个参数（通常由一个公共方法计算获得），这样后续的某些操作就可以不用重新计算 hash code 了。大部分内部方法也会接收一个 "tab" 参数，它就是当前的 table，但是在 resizing 和 converting 时，tab 可能是新的或旧的 table。

当 bin lists 进行 treeify、split 或者 untreeified，我们会保持它们的相对访问顺序（也就是 Node.next）。

## 静态变量

下面看一下 HashMap 类中拥有的静态变量：

### （1）DEFAULT_INITIAL_CAPACITY

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

初始化容量，最好是 2 的幂。

### （2）MAXIMUM_CAPACITY

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

最大容量，如果通过构造参数指定该参数，则必须小于等于 1 << 30（整数 4 字节）；

### （3）DEFAULT_LOAD_FACTOR

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

默认的负载因子，通常情况下使用 0.75 可以在时间和空间成本上较为均衡。

### （4）TREEIFY_THRESHOLD

```java
static final int TREEIFY_THRESHOLD = 8;
```

bin 树化的阈值，超过了该阈值，则 bins 由原来的 list of node 转变为 tree node。当某个 bin 拥有了足够多数量的 node 时，此时加入了一个新的 node，bin 就会树化。

（就是 HashMap 的一个 bucket 树化的阈值）

该参数应该大于 2 且至少为 8。

### （5）UNTREEIFY_THRESHOLD

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

bin untreeifying 的阈值。该参数必须小于 TREEIFY_THRESHOLD，且至少为 6。

### （6）MIN_TREEIFY_CAPACITY

```java
static final int MIN_TREEIFY_CAPACITY = 64;
```

向 map 中 put 新的元素，一旦对应的 bin 触发了树化操作（即单个 bin 内 node 数量 > TREEIFY_THRESHOLD），就会将当前 hash 表的长度（即 `table[].length`） 和该值进行比较；

- tab.len < MIN_TREEIFY_CAPACITY：就会触发整个 hash 表的 resize 操作，扩容并迁移旧表数据；
- tab.len > MIN_TREEIFY_CAPACITY：就会触发当前 bin 的树化操作。

该值至少为 4 * TREEIFY_THRESHOLD 避免 resizing 和 treeification thresholds 之间的冲突。

## 结点定义

### Basic hash bin node

看一下普通的 hash bin node：

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

可以看到这是一种链状结构，每个结点中有当前节点的键值对、hash 值、下一个结点的链接。重写了 hashCode 和 equals 方法。

### entry for tree bins

先了解一个继承关系：

`HashMap.TreeNode` extend `LinkedHashMap.Entry` entend `HashMap.Entry`

先看 LinkedHashMap 中的 Entry：

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

可以看到这个 Entry 扩展了前面的 Node，增加了 before 和 after 两个链接，并提供了新的构造方法，这样 HashMap 中的 Entry 中一共有三个链接了：before、next、after（二叉查找树的 2-3 结点树）；

然后看看 HashMap.Entry（红黑树结点）：

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
	
    // 。。。。。。
}
```

可以看到这是一种红黑树的实现；

## 静态工具方法

### （1）hash

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

该方法计算 HashMap 的 key 的 hash 值，主要通过 key.hashCode 高位分散到低位来减少 hash 冲突，更多信息参考源码注释。

### （2）comparableClassFor

```java
static Class<?> comparableClassFor(Object x) {
    if (x instanceof Comparable) {
        Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
        if ((c = x.getClass()) == String.class) // bypass checks
            return c;
        if ((ts = c.getGenericInterfaces()) != null) {
            for (int i = 0; i < ts.length; ++i) {
                if (((t = ts[i]) instanceof ParameterizedType) &&
                    ((p = (ParameterizedType)t).getRawType() ==
                     Comparable.class) &&
                    (as = p.getActualTypeArguments()) != null &&
                    as.length == 1 && as[0] == c) // type arg is c
                    return c;
            }
        }
    }
    return null;
}
```

可以看到这这个方法的主要目的是检测 x 对象的 Class 类型，如果它是 `class C implements Comparable` 的形式就返回它的 Class 对象，否则返回 null；

如果 c 是 String.class 直接就绕过检查了，因为 String 本身的 hashcode 是被缓存的，且继承了 Comparable 接口，重写了 compare 方法；

如果不是 String，就先获取该 Class 实现的所有接口泛型，因为 `Comparable<T>` 就是泛型接口，然后对其实现的每个泛型接口进行判断是不是 Comparable.class，已经 Comparable 的那个 T 的泛型参数是不是有效的。

### （3）进行 compare 的方法；

```java
@SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
static int compareComparables(Class<?> kc, Object k, Object x) {
    return (x == null || x.getClass() != kc ? 0 :
            ((Comparable)k).compareTo(x));
}
```

### （4）tableSizeFor

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

如果构造参数传递的初始容量并不是 2 的幂的话，则进行调整。

## 常规属性

### （1）table；

```java
transient Node<K,V>[] table;
```

table 其实就表示 list of bins 或者 tree bins，它是懒初始化的，一般只有第一次使用 HashMap 或者 resize 的时候会进行调整。在实例化时，它的 length 是 2 的幂。

### （2）entrySet；

```java
transient Set<Map.Entry<K,V>> entrySet;
```

Map 的 collection-view 中保存 entry 的 set；

### （3）size；

```java
transient int size;
```

Map 持有的键值对的数量；（也就是所有 bin 中的所有 entry 的总数量）

### （4）modCount；

```java
transient int modCount;
```

记录 HashMap 发生结构修改的次数，什么是结构修改呢？就是 HashMap 的 key-value mapping 的数量发生了变化或者内部的机构发生改变（比如：增删 entry，或 rehash）。这个属性的作用就是在遍历 HashMap 的集合视图的时候，判断是否需要 fail-fast。（比如发生了 ConcurrentModificationException）

### （5）threshold；

```java
int threshold;
```

当发生 resize 的时候，使用该值作为新的 size，通常是 `capacity * load factor`。

除此之外，如果 table 数组没有实例化，这个字段存的就是初始数组容量，或者为 0 表示 DEFAULT_INITIAL_CAPACITY。

### （6）loadFactor

```java
final float loadFactor;
```

hash table 的负载因子。

## 公共方法

### （1）构造方法

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

可以看到包含完整参数的构造方法，首先判断初始容量和负载因子是否合法，不合法就抛出异常或者调整为特定值，前面提到 `threshold` 通常存的是 hash table 数组的初始长度，`tableSizeFor` 则是规范化容量的方法（确保 threshold 是 2 的幂）。

```java
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 这里有个特殊的构造方法是构造了一个空的 HashMap，之前提到过 HashMap 是懒初始化的，在 put 元素时会检查是否需要初始化 table。
public HashMap() {
	this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

```java
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

这个构造方法是根据已有的一个 Map 构造新的 HashMap，可以参考一下 putMapEntries 方法中处理逻辑：

- HashMap 是空的，且已知要插入元素的数量，通过 `float ft = ((float)s / loadFactor) + 1.0F;` 可以为新的 HashMap 指定一个初始容量，减少 HashMap 的结构变化次数，提高一定的性能；
- 如果 HashMap 不是空的，则进行比较 `s > threshold` 来判断是否需要 `resize()`，resize 会将 table 的 size 增长 2 倍；

### （2）size

```java
public int size() {
    return size;
}
```

很简单，获取 HashMap 中所有 key-value mapping 的数量；

### （3）get

根据 key 获取对应的 value，如果没有存对应的 mapping 就返回 null，否则返回 value；

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

`hash(key)` 先计算 hash 值，作为参数传递，后面就不用重复计算了。重点在 getNode 方法：

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; 
    Node<K,V> first
    Node<K,V> e; 
    int n; 
    K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

这个方法的判断逻辑是这样的：

（1）首先我们的 HashMap 必须是有 bin 的，也就是 table 不能为空且 bin 的数量大于 0；

（2）然后我们根据 hash 函数映射，找到该 key 对应的 bin：`first = tab[(n - 1) & hash]`，first 就是这个 bin 的头部，也就是前面提到的 `java.util.HashMap.Node`；

（3）如果这个 first 确实存在，它有三种情况，要么是一个只有一个结点的链表，要么是多个结点的链表，要么是多结点的红黑树，首先比较 first 结点是否和 key 对应，比较顺序为：

- 先比较 Node 的 hash 值，这个 hash 是用 hash() 方法计算出来的，和 key 的 hashcode 有关，后面再讲，hash 相同的同时需要保证 key 的 Node.key 是相同的引用，或者 equals 返回 true；如果最终结果为 true，则说明目标 key 在 Map 中确实存在，且为某个 bin 的首结点，立即返回这个 first；
- 如果首结点没有成功匹配，则判断首结点的 next 是不是为 null，为 null 表明该 key 没有在 Map 中存在，如果不为 null，则该 bin 可能是一个多结点的链表或红黑树：
  - 如果是链表，则继续向下找，匹配方式和 first 结点逻辑一致；
  - 如果是树，则将 first 转换为 TreeNode，再去树中查找，`((TreeNode<K,V>)first).getTreeNode(hash, key)`

```java
final TreeNode<K,V> getTreeNode(int h, Object k) {
    return ((parent != null) ? root() : this).find(h, k, null);
}
```

`root()` 方法是从某个 TreeNode 结点出发找到整棵树的根结点，然后调用根结点的 `find` 方法去找

```java
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    TreeNode<K,V> p = this;
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h)
            p = pl;
        else if (ph < h)
            p = pr;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if (pl == null)
            p = pr;
        else if (pr == null)
            p = pl;
        else if ((kc != null ||
                  (kc = comparableClassFor(k)) != null) &&
                 (dir = compareComparables(kc, k, pk)) != 0)
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null)
            return q;
        else
            p = pl;
    } while (p != null);
    return null;
}
```

这个树结点的查找算法就暂时不仔细研究，注意其 key 的匹配方式：

```java
else if ((pk = p.key) == k || (k != null && k.equals(pk)))
    return p;
```

走到这个分支此时 hash 已经相等，再比较 key 的引用或者 equals 结果，除了数据结构不一样，具体涉及到 key-value mapping 的比较和链表形式是一致的。

> 总结

HashMap 允许最多存在一个 null 和 null 的映射，允许 value 为 null，get 的时候先比较 Node.hash（这个 hash 是使用静态工具方法 hash() 根据 key 的 hashcode 计算得来），如果 Node.hash 相同，则进一步比较 Node.key 和 key 的引用是否相同或者 equals 方法是否返回 ture，最终决定目标结点和目标 key 具有相同的 key。



### （4）put

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

向 HashMap 中添加键值对，如果 map 中已经有了相同的 key 映射，则覆盖该 key 关联的值，注意方法是有返回值的，如果返回不是 null 的值，则表明 key 之前已经存在，返回的值就是被覆盖的 value；

如果返回 null 分为两种情况：

- 要 put 的 key-value pair 是一个新的映射关系，Map 之前没有包含，put 执行成功；
- 可能之前存在这样的映射，但是之前的 key 映射的 value 是 null；

再看内部的 putVal 方法：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 判断 HashMap 是否已经初始化了, 如果没有就调用 resize 方法初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 根据新 key 的 hash 值, 经过散列映射到 table 的某个 index 上, 检测这个位置的 nude 是不是 null, 是 null 就要进行分配, 否则就说明出现了哈希冲突，沿着这个 node 链或树向下找到合适的位置插入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 如果 Map 中已经存在了新 key 对应的 mapping, 则用新 value 覆盖掉旧的 mapping
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果是树的话, 就利用红黑树的插入方式插入
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 如果是链的话, 就找到链的末尾插入新结点
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 如果 Map 中已经存在了新 key 对应的 mapping, 则根据选项采取策略
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 旧的 value 不存在或者为 null, 直接覆盖
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // LinkedHashMap 中实现该方法, 用于后处理
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 前面提到 modCount 记录 HashMap 结构变化的次数, 主要用于在迭代集合视图时保证 fail-fast
    ++modCount;
    // 新加入一个 mapping, size 加 1, 同时 threshold 记录 HashMap 下一次扩容时的容量, 如果新加入一个 mapping, size > threshold 就需要扩容了
    if (++size > threshold)
        resize();
    // LinkedHashMap 中实现该方法
    afterNodeInsertion(evict);
    return null;
}
```



### （5）resize

resize 是 HashMap 的初始化和扩容方法：

- 初始化的时候，分配前面提到的 threshold 字段存储的值的容量的 table；

- 扩容的时候 table 自动扩大两倍。

除此之外，由于遵循 2 的幂扩张机制，原来 bin 中的所有元素再扩容后，要么和原来保持相同的 index，要么移动 2 的幂的偏移量。

源码：

```java
final Node<K,V>[] resize() {
    // 备份旧的 hash 表
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 扩容策略: 2 的幂机制, 直接在旧的基础上增大两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 为扩容后的 hash 表分配内存空间
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //  执行扩容操作: 将旧表中的元素迁移到新表中
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```



### （6）treeifyBin

此方法用于树化 hash 表中某个链表 bin 结点：

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

注意这里的单个 bin 的树化阈值是 TREEIFY_THRESHOLD，默认是 8，而 bin 的树化逻辑中必须要求整个 hash table 的长度大于等于 MIN_TREEIFY_CAPACITY，才能执行真正的树化操作，否则触发 resize；

