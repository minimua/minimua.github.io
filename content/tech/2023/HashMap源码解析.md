---
title: "HashMap源码解析 building"
date: 2023-03-01T22:00:00+08:00
tags: ["tech"]
categories: ["tech"]
---

# 简介

以下是对 `HashMap` 类说明的中文翻译：

`HashMap` 是基于哈希表的 `Map` 接口实现。该实现提供了所有可选的 `Map` 操作，并允许 `null` 值和 `null` 键。（`HashMap` 类大致等同于 `Hashtable`，但它不是同步的，并且允许 `null` 值和键。）该类不保证 `Map` 中元素的顺序，尤其是不保证顺序会随着时间的推移保持不变。

对于基本操作（如 `get` 和 `put`），该实现提供了常量时间的性能，前提是哈希函数能够将元素适当地分散到各个桶中。遍历集合视图所需的时间与 `HashMap` 实例的“容量”（即桶的数量）和其大小（即键值映射的数量）成正比。因此，如果迭代性能很重要，不应将初始容量设置得太高（或者负载因子设置得太低）。

`HashMap` 实例有两个参数会影响其性能：初始容量和负载因子。容量是哈希表中桶的数量，初始容量即哈希表创建时的容量。负载因子是一个衡量哈希表在容量自动增加之前允许填满程度的指标。当哈希表中的条目数量超过负载因子与当前容量的乘积时，哈希表将进行重新哈希（即重建其内部数据结构），使得哈希表的桶数大约翻倍。

通常情况下，默认的负载因子（0.75）在时间和空间开销之间提供了良好的权衡。较高的负载因子会减少空间开销，但会增加查找成本（这会影响 `HashMap` 类的操作，如 `get` 和 `put`）。在设置初始容量时，应考虑映射中预计的条目数量和负载因子，以尽量减少重新哈希的次数。如果初始容量大于最大条目数除以负载因子，则不会发生重新哈希操作。

如果要在 `HashMap` 实例中存储大量映射，创建一个具有足够大容量的实例将比让其根据需要自动进行重新哈希更有效率地存储映射。需要注意的是，使用具有相同 `hashCode()` 的多个键会显著降低任何哈希表的性能。为了减轻这一影响，当键是 `Comparable` 时，该类可能会使用键之间的比较顺序来帮助打破冲突。

需要注意的是，该实现不是同步的。如果多个线程同时访问 `HashMap`，并且至少有一个线程对其结构进行修改，则必须进行外部同步。（结构修改是指添加或删除一个或多个映射的操作；仅仅修改已经包含的键的关联值并不属于结构修改。）通常可以通过同步封装 `Map` 的某个对象来实现同步。如果不存在这样的对象，应该使用 `Collections.synchronizedMap` 方法对 `Map` 进行“包装”。最好在创建时完成此操作，以防止意外的非同步访问：

```java
Map m = Collections.synchronizedMap(new HashMap(...));
```

该类所有“集合视图方法”返回的迭代器是快速失败的：如果在创建迭代器后，`Map` 的结构以任何方式被修改，除了通过迭代器自身的 `remove` 方法外，迭代器将抛出 `ConcurrentModificationException`。因此，在面对并发修改时，迭代器会快速且干净地失败，而不是在未来某个不确定的时间冒着发生任意非确定性行为的风险。

需要注意的是，迭代器的快速失败行为不能被保证，因为在未同步的并发修改情况下，通常不可能做出任何严格的保证。快速失败迭代器是在尽力而为的基础上抛出 `ConcurrentModificationException`。因此，依赖于该异常来确保程序正确性的做法是不对的：迭代器的快速失败行为应仅用于检测程序中的错误。

该类是 Java 集合框架的成员。

自：1.2 版本

参见：`Object.hashCode()`、`Collection`、`Map`、`TreeMap`、`Hashtable`

作者：Doug Lea, Josh Bloch, Arthur van Hoff, Neal Gafter

类型参数：

- `<K>` – `Map` 中维护的键的类型
- `<V>` – 映射值的类型

**源码注释的原文**

> Hash table based implementation of the Map interface. This implementation provides all of the optional map operations, and permits null values and the null key. (The HashMap class is roughly equivalent to Hashtable, except that it is unsynchronized and permits nulls.) This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.
> This implementation provides constant-time performance for the basic operations (get and put), assuming the hash function disperses the elements properly among the buckets. Iteration over collection views requires time proportional to the "capacity" of the HashMap instance (the number of buckets) plus its size (the number of key-value mappings). Thus, it's very important not to set the initial capacity too high (or the load factor too low) if iteration performance is important.
> An instance of HashMap has two parameters that affect its performance: initial capacity and load factor. The capacity is the number of buckets in the hash table, and the initial capacity is simply the capacity at the time the hash table is created. The load factor is a measure of how full the hash table is allowed to get before its capacity is automatically increased. When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is rehashed (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.
> As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs. Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put). The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.
> If many mappings are to be stored in a HashMap instance, creating it with a sufficiently large capacity will allow the mappings to be stored more efficiently than letting it perform automatic rehashing as needed to grow the table. Note that using many keys with the same hashCode() is a sure way to slow down performance of any hash table. To ameliorate impact, when keys are Comparable, this class may use comparison order among keys to help break ties.
> Note that this implementation is not synchronized. If multiple threads access a hash map concurrently, and at least one of the threads modifies the map structurally, it must be synchronized externally. (A structural modification is any operation that adds or deletes one or more mappings; merely changing the value associated with a key that an instance already contains is not a structural modification.) This is typically accomplished by synchronizing on some object that naturally encapsulates the map. If no such object exists, the map should be "wrapped" using the Collections.synchronizedMap method. This is best done at creation time, to prevent accidental unsynchronized access to the map:
> Map m = Collections.synchronizedMap(new HashMap(...));
> The iterators returned by all of this class's "collection view methods" are fail-fast: if the map is structurally modified at any time after the iterator is created, in any way except through the iterator's own remove method, the iterator will throw a ConcurrentModificationException. Thus, in the face of concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future.
> Note that the fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification. Fail-fast iterators throw ConcurrentModificationException on a best-effort basis. Therefore, it would be wrong to write a program that depended on this exception for its correctness: the fail-fast behavior of iterators should be used only to detect bugs.
> This class is a member of the Java Collections Framework.
> Since:
> 1.2
> See Also:
> Object.hashCode(), Collection, Map, TreeMap, Hashtable
> Author:
> Doug Lea, Josh Bloch, Arthur van Hoff, Neal Gafter
> Type parameters:
> <K> – the type of keys maintained by this map
> <V> – the type of mapped values
