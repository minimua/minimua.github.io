---
title: "Java基础-集合"
date: 2023-07-09T22:00:00+08:00
tags: ["tech"]
categories: ["tech"]
---

 #### 1.  说说 List,Set,Map 三者的区别？三者底层的数据结构？

-   `List`(对付顺序的好帮手): 存储的元素是有序的、可重复的。
-   `Set`(注重独一无二的性质): 存储的元素是无序的、不可重复的。
-   `Queue`(实现排队功能的叫号机): 按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的。
-   `Map`(用 key 来搜索的专家): 使用键值对（key-value）存储，类似于数学上的函数 y=f(x)，"x" 代表 key，"y" 代表 value，key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。

#### List
-   `Arraylist`： `Object[]` 数组
-   `Vector`：`Object[]` 数组
-   `LinkedList`： 双向链表(JDK1.6 之前为循环链表，JDK1.7 取消了循环)

#### Set
-   `HashSet`(无序，唯一): 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素
-   `LinkedHashSet`: `LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的。有点类似于我们之前说的 `LinkedHashMap` 其内部是基于 `HashMap` 实现一样，不过还是有一点点区别的
-   `TreeSet`(有序，唯一): 红黑树(自平衡的排序二叉树)

#### Queue
-   `PriorityQueue`: `Object[]` 数组来实现二叉堆
-   `ArrayQueue`: `Object[]` 数组 + 双指针


#### Map
-   `HashMap`： JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
-   `LinkedHashMap`： `LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
-   `Hashtable`： 数组+链表组成的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的
-   `TreeMap`： 红黑树（自平衡的排序二叉树）

```md
List是有序，可重复的，ArrayList底层是数组，查询修改快；LinkedList底层是链表，新增删除快；
Set是无序，不可重复的，HashSet是基于HashMap实现的，是唯一的，无序的；
Map是键值对的形式，是无需且不可重复的，HashMap在jdk1.8之前底层是数组加链表，jdk1.8之后是数组加链表加红黑树

```

#### 2.  有哪些集合是线程不安全的？怎么解决呢？
**ArrayLiat线程不安全**
使用Vector  
使用Collections.synchronizeList()  
使用CopyOnWriteArrayList()
**HashSet线程不安全**
Collections.synchronizeSet
CopyOnWriteHashSet
HashMap线程不安全
**HashTable**
Collections.synchronizedMap
ConcurrentHashMap
#### 3.  比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同

HashSet： 无序  
LinkedHashSet： 按照插入顺序  
TreeSet： 从小到大排序

#### 4.  HashMap 和 Hashtable 的区别？HashMap 和 HashSet 区别？HashMap 和 TreeMap 区别？
**HashMap 和 Hashtable 的区别**
1.  **线程是否安全：** `HashMap` 是非线程安全的，`Hashtable` 是线程安全的,因为 `Hashtable` 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
2.  **效率：** 因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰，不要在代码中使用它；
3.  **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
4.  **初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
5.  **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。
```md
HashMap允许键和值为null，HashTable不可以；HashMap不是线程安全的，HashTable是，但HashTable是个基本被淘汰的类，如果需要线程安全，应该用ConcurrentHashMap；
```

#### 5.  HashMap 的底层实现
JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。**HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。**

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。**

相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

```md
JDK1.8之前HashMap底层是数组加链表的结构，主要是通过对key的hashcode经过一个扰动函数处理得到一个hash值，再通过一个算法就是(table.length - 1)&hash得到这个元素在数组中应该存放的位置，也就是下标。当然因为这个数组的长度是有限的，我们如果数据很多的话，肯定会有重复需要放在一个位置的情况，这就是hash碰撞。如果重复了，就会把这个元素插在原来元素的前面，然后原来的元素向下移，形成一个链表的结构，这就是JDK1.8之前的头插法，从性能的角度来看，因为链表的查询是比较满的，如果要插在尾部的话，就需要一步一步找到这个链表的最后一个位置，肯定是不太合适的，所以JDK1.7之前采用的是这种头插法，但是这种方法在多线程的时候会出现链表死循环的问题，所以在JDK1.8之后就采用尾插法了。

在JDK1.8，引入了红黑树的数据结构，当数组的长度大于等于64并且链表上的元素个数大于8个的时候就会将链表转为红黑树。如果数组的长度没到64，会先进行扩容，对于新new出来一个HashMap的数组，如果没有指定长度，会在第一次put数据的时候初始化一个长度，默认是16，当长度超过一个阈值，默认是长度乘以负载因子0.75，也就是12的时候就会进行扩容的操作，会创建一个新的数组长度是原来的两倍，然后rehash，把原来的数据迁移到新的数组。
```
#### 6.  HashMap 的长度为什么是 2 的幂次方
为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。这也就解释了 HashMap 的长度为什么是 2 的幂次方。
#### 7.  ConcurrentHashMap 和 Hashtable 的区别？
`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同。

-   **底层数据结构：** JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
-   **实现线程安全的方式（重要）：** ① **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(`Segment`)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 对 `synchronized` 锁做了很多优化）** 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② **`Hashtable`(同一把锁)** :使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。
- 
#### 8.  ConcurrentHashMap 线程安全的具体实现方式/底层具体实现
