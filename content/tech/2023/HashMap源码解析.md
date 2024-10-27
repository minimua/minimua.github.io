---
title: "HashMap源码解析"
date: 2023-03-01T22:00:00+08:00
tags: ["tech"]
categories: ["tech"]
---

## 简介

`HashMap` 是基于哈希表的 `Map` 接口实现。该实现提供了所有可选的 `Map` 操作，并允许 `null` 值和 `null` 键。（`HashMap` 类大致等同于 `Hashtable`，但它不是同步的，并且允许 `null` 值和键。）该类不保证 `Map` 中元素的顺序，尤其是不保证顺序会随着时间的推移保持不变。

## 基础知识
### 数组的优势和劣势
**优势**：

- **快速访问**：数组的最大优势在于它能够通过下标快速访问任意元素。访问时间复杂度为O(1)，这是由于数组是连续内存结构，计算下标即可快速定位元素。
**劣势**：
- **固定大小**：数组的大小在初始化时必须确定，无法动态扩展。这在需要存储未知数量数据时会带来不便。
- **插入和删除操作效率低**：插入或删除元素时，数组需要移动其他元素来保持连续性，时间复杂度为O(n)。  
### 链表的优势和劣势
**优势**：
- **动态大小**：链表可以根据需要动态增长或缩减，适合存储不确定数量的数据。
- **插入和删除操作高效**：在链表中插入或删除节点，只需改变指针方向，不需要移动其他节点，时间复杂度为O(1)（前提是已经找到位置）。
**劣势**：
- **访问效率低**：链表必须从头开始遍历才能找到指定节点，时间复杂度为O(n)。
### 散列表——整合了两种数据结构的优势

散列表（Hash Table）结合了数组和链表的优势：
- 使用数组来存储数据，以实现快速访问。
- 当出现冲突时，使用链表或其他结构来存储相同位置的多个元素。
### 散列表的特点
- **键值对存储**：散列表以键值对形式存储数据，通过键快速定位对应的值。
- **哈希函数**：散列表依赖于哈希函数来将键映射到数组的特定位置（索引）。
- **哈希冲突**：由于数组的固定长度，不同的键可能映射到相同的位置，称为“哈希冲突”。
### 什么是哈希
哈希是一种将任意长度的数据通过哈希函数转换为固定长度数据的过程。常用的哈希函数具有以下特点：
- **高效计算**：输入相同的数据总是产生相同的输出。
- **均匀分布**：理想情况下，不同的输入数据应尽可能均匀地分布到整个哈希空间。
## HashMap原理

### HashMap的继承体系

`HashMap`是Java集合框架的一部分，它继承了以下类和接口：
- **`AbstractMap`**：提供了Map接口的部分实现。
- **`Map`接口**：定义了键值对存储结构的基本行为。
- **`Cloneable`**：支持克隆操作。
- **`Serializable`**：支持序列化，方便在网络中传输。

### Node数据结构分析

在`HashMap`中，每个键值对是通过内部类`Node<K,V>`来存储的。其定义如下：

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

- **`hash`**：记录键的哈希值。
- **`key`**：存储键。
- **`value`**：存储值。
- **`next`**：指向下一个节点，形成链表，处理哈希冲突。

### 底层存储结构

`HashMap`的底层是一个**数组**，其中每个数组元素都是一个链表的头节点。在JDK 8中，当链表过长时，链表会转换为红黑树结构。

- **数组**：存储`Node`的引用，每个数组索引对应一个`bucket`。
- **链表**：解决哈希冲突的第一种方式，当多个键映射到相同的数组索引时，它们会形成一个链表。
- **红黑树**：为优化长链的查找效率，在链表长度超过8时，链表会转换为红黑树结构。

### put数据原理

`HashMap`的`put()`方法是向集合中插入键值对的核心操作。其基本流程如下：
1. **计算哈希值**：通过`key`的`hashCode()`方法计算哈希值，然后对数组长度取模，找到存储位置。
2. **查找位置**：通过计算出的数组索引找到对应的`bucket`，如果该位置为空，直接创建新节点并存入。
3. **处理哈希冲突**：如果该位置已有节点，遍历链表（或树）查找是否已存在相同键。如果找到相同键，更新对应的值；否则，将新节点插入链表末尾或红黑树中。
4. **检查容量**：插入后，判断`HashMap`是否需要扩容。如果当前元素个数超过阈值（即`capacity * loadFactor`），则进行扩容。

### 什么是Hash碰撞

当两个不同的键通过哈希函数计算得到相同的哈希值时，就会发生哈希冲突（Hash Collision）。由于数组索引有限，而可能的键几乎无限，哈希冲突是不可避免的。`HashMap`通过链表（或树）来解决冲突。

### 什么是链化

链化是`HashMap`解决哈希冲突的经典方法之一。当多个键的哈希值相同且映射到同一个`bucket`时，它们会形成一个链表。插入新元素时，遍历链表查找是否存在相同键，若无，则将新节点插入链表末尾。

### JDK 8为什么引入红黑树

在JDK 8之前，`HashMap`仅使用链表处理哈希冲突。当链表过长时，查找、插入和删除的时间复杂度会退化为O(n)，降低性能。

为了解决这一问题，JDK 8引入了红黑树。当链表长度超过8时，链表会转换为红黑树，查找效率从O(n)提升到O(log n)。这样，在极端情况下也能保证较高的性能。

## 源码

### 核心属性

``` java
public class HashMap<K,V> extends AbstractMap<K,V>  
    implements Map<K,V>, Cloneable, Serializable {
		/**  
		 * The default initial capacity - MUST be a power of two. * mjy：默认table大小  
		 */  
		static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  
		  
		/**  
		 * The maximum capacity, used if a higher value is implicitly specified 
		 * by either of the constructors with arguments. *
		 * MUST be a power of two <= 1<<30. 
		 * mjy：table最大长度  
		 */  
		static final int MAXIMUM_CAPACITY = 1 << 30;  
		  
		/**  
		 * The load factor used when none specified in constructor. 
		 * mjy：默认负载因子大小  
		 */  
		static final float DEFAULT_LOAD_FACTOR = 0.75f;  
		  
		/**  
		 * The bin count threshold for using a tree rather than list for a 
		 * bin.  Bins are converted to trees when adding an element to a 
		 * bin with at least this many nodes. The value must be greater 
		 * than 2 and should be at least 8 to mesh with assumptions in 
		 * tree removal about conversion back to plain bins upon * shrinkage. 
		 * mjy：链表树化阈值  
		 */  
		static final int TREEIFY_THRESHOLD = 8;  
		  
		/**  
		 * The bin count threshold for untreeifying a (split) bin during a 
		 * resize operation. Should be less than TREEIFY_THRESHOLD, and at 
		 * most 6 to mesh with shrinkage detection under removal. 
		 * mjy：树降级为链表的阈值  
		 */  
		static final int UNTREEIFY_THRESHOLD = 6;  
		  
		/**  
		 * The smallest table capacity for which bins may be treeified. 
		 * (Otherwise the table is resized if too many nodes in a bin.) 
		 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts 
		 * between resizing and treeification thresholds. 
		 * mjy：树化的另一个参数，当哈希表中的所有参数超过64，才会允许树化  
		 */  
		static final int MIN_TREEIFY_CAPACITY = 64;  
		  
		/**  
		 * Basic hash bin node, used for most entries.  (See below for 
		 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.) 
		 * mjy：每个键值对通过这个Node存储  
		 */  
		static class Node<K,V> implements Entry<K,V> {  
		  
		    // mjy：记录键的哈希值  
		    final int hash;  
		    // mjy：存储键  
		    final K key;  
		    // mjy：存储值  
		    V value;  
		    // mjy：指向下一个节点，形成链表，处理哈希冲突  
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
		            Entry<?,?> e = (Entry<?,?>)o;  
		            if (Objects.equals(key, e.getKey()) &&  
		                Objects.equals(value, e.getValue()))  
		                return true;  
		        }  
		        return false;  
		    }  
		}

		/* ---------------- Fields -------------- */  
  
		/**  
		 * The table, initialized on first use, and resized as 
		 * necessary. When allocated, length is always a power of two. 
		 * (We also tolerate length zero in some operations to allow 
		 * bootstrapping mechanics that are currently not needed.) 
		 * mjy：哈希表，初始化时机？  
		 */  
		transient Node<K,V>[] table;  
  
		/**  
		 * Holds cached entrySet(). Note that AbstractMap fields are used 
		 * for keySet() and values(). 
		 */
		 transient Set<Entry<K,V>> entrySet;  
  
		/**  
		 * The number of key-value mappings contained in this map. 
		 * mjy：当前哈希表中元素个数  
		 */  
		transient int size;  
  
		/**  
		 * The number of times this HashMap has been structurally modified 
		 * Structural modifications are those that change the number of mappings in 
		 * the HashMap or otherwise modify its internal structure (e.g., 
		 * rehash).  This field is used to make iterators on Collection-views of 
		 * the HashMap fail-fast.  (See ConcurrentModificationException). 
		 * mjy：当前哈希表结构修改次数  
		 */  
		transient int modCount;  
		  
		/**  
		 * The next size value at which to resize (capacity * load factor). 
		 * mjy：扩容阈值，当哈希表中的元素超过阈值，出发扩容  
		 * @serial  
		 */  
		// (The javadoc description is true upon serialization.  
		// Additionally, if the table array has not been allocated, this  
		// field holds the initial array capacity, or zero signifying  
		// DEFAULT_INITIAL_CAPACITY.)  
		int threshold;  
		  
		/**  
		 * The load factor for the hash table. 
		 * mjy：负载因子，通过负载因子计算阈值， threshold = capacity * loadFactor  
		 * @serial  
		 */  
		final float loadFactor;
    }

```

### 构造方法

``` java
/* ---------------- Public operations -------------- */  
  
/**  
 * Constructs an empty <tt>HashMap</tt> with the specified initial  
 * capacity and load factor. 
 * 
 * @param  initialCapacity the initial capacity  
 * @param  loadFactor      the load factor  
 * @throws IllegalArgumentException if the initial capacity is negative  
 *         or the load factor is nonpositive */
 public HashMap(int initialCapacity, float loadFactor) {  
    // mjy：一些校验，capacity必须大于0，最大值max  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    // mjy：loadFactor必须大于0  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);  
}  
  
/**  
 * Constructs an empty <tt>HashMap</tt> with the specified initial  
 * capacity and the default load factor (0.75). 
 * 
 * @param  initialCapacity the initial capacity.  
 * @throws IllegalArgumentException if the initial capacity is negative.  
 */
 public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}  
  
/**  
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity  
 * (16) and the default load factor (0.75). 
 */
 public HashMap() {  
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted  
}  
  
/**  
 * Constructs a new <tt>HashMap</tt> with the same mappings as the  
 * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with  
 * default load factor (0.75) and an initial capacity sufficient to 
 * hold the mappings in the specified <tt>Map</tt>.  
 * 
 * @param   m the map whose mappings are to be placed in this map  
 * @throws  NullPointerException if the specified map is null  
 */
 public HashMap(Map<? extends K, ? extends V> m) {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;  
    putMapEntries(m, false);  
}
```

#### tableSizeFor()

``` java
/**  
 * Returns a power of two size for the given target capacity. 
 * mjy：返回一个大于等于当前cap的数字，并且这个数字一定是2的次方  
 *  cap = 10  
 *  n = 10 - 1 = 9  0b1001 
 *  0b1001 | 0b0100 = 0b1101 
 *  0b1101 | 0b0011 = 0b1111 
 *  0b1111 | 0b0000 = 0b1111 
 *  ... 
 *  继续执行右移8位和 16位的操作，也不会再变化，最终 n = 15  
 *  返回n+1=16，所以大于等于10的最小2的幂是 16  
 */
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

### put方法分析

**核心是putVal()**
1. 如果定位到的数组位置没有元素 就直接插入。
2. 如果定位到的数组位置有元素就和要插入的 key 比较，如果 key 相同就直接覆盖，如果 key 不相同，就判断 p 是否是一个树节点，如果是就调用`e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

``` java
/**  
 * Associates the specified value with the specified key in this map. 
 * If the map previously contained a mapping for the key, the old 
 * value is replaced. 
 * 
 * @param key key with which the specified value is to be associated  
 * @param value value to be associated with the specified key  
 * @return the previous value associated with <tt>key</tt>, or  
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.  
 *         (A <tt>null</tt> return can also indicate that the map  
 *         previously associated <tt>null</tt> with <tt>key</tt>.)  
 */public V put(K key, V value) {  
    return putVal(hash(key), key, value, false, true);  
}  
  
/**  
 * Implements Map.put and related methods 
 * 
 * @param hash hash for key  
 * @param key the key  
 * @param value the value to put  
 * @param onlyIfAbsent if true, don't change existing value  
 * @param evict if false, the table is in creation mode.  
 * @return previous value, or null if none  
 * mjy：  
 * hash：键的哈希值，通常通过hash()方法计算得来。  
 * key：要插入的键  
 * value：要插入的值  
 * onlyIfAbsent：如果为 true，表示当键存在时不覆盖原有值（即只在没有映射的情况下插入）  
 * evict：是否在某些情况下触发后续处理（一般用于LinkedHashMap的操作）  
 */  
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,  
               boolean evict) {  
    /**  
     * mjy：  
     * tab：存储键值对的哈希表（数组）  
     * p：当前哈希表的某个索引位置的第一个节点（即链表或红黑树的头节点）  
     * n：哈希表（数组）长度  
     * i：当前键应该插入的数组位置  
     */  
    Node<K,V>[] tab; Node<K,V> p; int n, i;  
    // mjy：哈希表未初始化或者长度为0，调用resize()进行初始化（延迟初始化）  
    if ((tab = table) == null || (n = tab.length) == 0)  
        n = (tab = resize()).length;  
    // mjy：p为当前节点，i = (n - 1) & hash 为当前键应该插入的数组位置（即索引i）  
    if ((p = tab[i = (n - 1) & hash]) == null)  
        // mjy：当前索引处元素为空，创建新节点并插入  
        tab[i] = newNode(hash, key, value, null);  
    else {  
        /**  
         * mjy：当前索引处不为空  
         * e：临时存储当前元素  
         */  
        Node<K,V> e; K k;  
        // mjy：现有的节点p和当前插入的键key完全相同（hash和equals方法都相同）  
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k))))  
            // mjy：e赋值为p，之后会更新  
            e = p;  
        else if (p instanceof TreeNode)  
            // 当前节点是TreeNode（红黑树节点）  
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
        else {  
            // mjy：否则就是链表，通过链表遍历处理冲突  
            for (int binCount = 0; ; ++binCount) {  
                // mjy：（e = p.next为获取当前节点的下一个节点）  
                // mjy：条件成立即为遍历到了链表的最后一个元素，没找到key一致的node，newNode新节点加入链表末尾，跳出循环  
                if ((e = p.next) == null) {  
                    p.next = newNode(hash, key, value, null);  
                    // 链表长度超过TREEIFY_THRESHOLD - 1，树化  
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                        treeifyBin(tab, hash);  
                    break;                }  
                // mjy：e != null  
                // mjy：条件成立即为遍历中，找到了和当前插入的键key完全相同（hash和equals方法都相同）的数据，跳出循环  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    break;  
                /**  
                 * mjy：将p移动到下个节点  
                 * e = p.next  
                 * e != null                 * p = e ==》p.next  
                 */                p = e;  
            }  
        }  
        // mjy：找到已经存在的节点，需要替换  
        if (e != null) { // existing mapping for key  
            V oldValue = e.value;  
            if (!onlyIfAbsent || oldValue == null)  
                e.value = value;  
            afterNodeAccess(e);  
            return oldValue;  
        }  
    }  
    // mjy：哈希表（散列表）结构被修改的次数，替换不算  
    ++modCount;  
    // mjy：插入新元素后size自增，如果自增后的值大于阈值，触发扩容  
    if (++size > threshold)  
        resize();  
    afterNodeInsertion(evict);  
    return null;}
```
### resize方法分析

``` java

/**  
 * Initializes or doubles table size.  If null, allocates in 
 * accord with initial capacity target held in field threshold. 
 * Otherwise, because we are using power-of-two expansion, the 
 * elements from each bin must either stay at same index, or move 
 * with a power of two offset in the new table. 
 * mjy：扩容：解决哈希冲突导致的链化影响查询效率的问题，扩容会缓解该问题  
 * @return the table  
 */final Node<K,V>[] resize() {  
    // mjy：扩容前的哈希表  
    Node<K,V>[] oldTab = table;  
    // mjy：扩容前的哈希表（数组）长度，旧数组为空则为0  
    int oldCap = (oldTab == null) ? 0 : oldTab.length;  
    // 扩容前的阈值，触发本次扩容的阈值  
    int oldThr = threshold;  
    // 存放新数组大小，新数组的扩容的阈值  
    int newCap, newThr = 0;  
    // 根据当前数组的大小（oldCap）和阈值（oldThr），选择不同的扩容策略  
    // 大于0，即hashMap中的散列表已经初始化过了，正常扩容  
    if (oldCap > 0) {  
        // 已经最大了，不再扩容  
        if (oldCap >= MAXIMUM_CAPACITY) {  
            threshold = Integer.MAX_VALUE;  
            return oldTab;  
        }  
        // newCap（新数组大小）：oldCap（旧数组大小）左移1位数值翻倍，newCap小于数组最大限制且oldCap大于16  
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&  
                 oldCap >= DEFAULT_INITIAL_CAPACITY)  
            // newThr（新扩容阈值）：oldThr（旧扩容阈值）左移1位数值翻倍  
            newThr = oldThr << 1; // double threshold  
    }  
    /**  
     * mjy：  
     * oldCap == 0 oldThr > 0 hashMap中的散列表是null  
     * 1.new HashMap(initCap, loadFactor);     
     * 2.new HashMap(initCap);    
     * 3.new HashMap(map); 并且这个map有数据  
     */  
    else if (oldThr > 0) // initial capacity was placed in threshold  
        newCap = oldThr;  
    /**  
     * mjy：  
     * oldCap == 0 oldThr == 0  
     * 1.new HashMap();     
     * */    
     else {               // zero initial threshold signifies using defaults  
        newCap = DEFAULT_INITIAL_CAPACITY;  
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  
    }  
    // mjy：newThr为0时，通过newCap和loadFactor计算出新的newThr  
    if (newThr == 0) {  
        float ft = (float)newCap * loadFactor;  
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?  
                  (int)ft : Integer.MAX_VALUE);  
    }  
    threshold = newThr;  
    /** mjy：开始扩容 */  
    // mjy：创建一个更长更大的数组  
    @SuppressWarnings({"rawtypes","unchecked"})  
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  
    table = newTab;  
    // mjy：旧数组不为空  
    if (oldTab != null) {  
        // mjy遍历旧数组  
        for (int j = 0; j < oldCap; ++j) {  
            // mjy：临时变量，（旧数组中的）当前node节点  
            Node<K,V> e;  
            // mjy：当前节点（桶位）有数据，（可能为：单个数据/链表/红黑树），该数据赋值给e  
            if ((e = oldTab[j]) != null) {  
                // mjy：旧数组的这个节点置空，方便JVM GC时回收  
                oldTab[j] = null;  
                // mjy：1.下个节点为空，即当前节点为单个数据，从未发生过碰撞  
                if (e.next == null)  
                    // mjy：直接计算出 该元素在新数组中的位置 e.hash & (newCap - 1) ，放进去  
                    newTab[e.hash & (newCap - 1)] = e;  
                else if (e instanceof TreeNode) {  
                    // mjy：3.红黑树的情况  
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
                }  
                else { // preserve order  
                    // mjy：2.链表的情况  
                    // 低位链表：在新数组中的下标位置，和在旧数组的一致  
                    Node<K,V> loHead = null, loTail = null;  
                    // 高位链表：在新数组中的下标位置，为旧数组的下标位置 + 旧数组的长度  
                    Node<K,V> hiHead = null, hiTail = null;  
                    Node<K,V> next;  
                    do {  
                        // mjy：下个节点  
                        next = e.next;  
                        /**  
                         * mjy：每个节点的哈希值与旧数组的容量oldCap进行按位与操作，决定其放在低位或高位  
                         * 低位节点  
                         */  
                        if ((e.hash & oldCap) == 0) {  
                            // 如果loTail为空，则说明当前是链表的第一个节点  
                            if (loTail == null)  
                                loHead = e;  
                            else                                
	                            // 否则赋值给next  
                                loTail.next = e;  
                            loTail = e;  
                        }  
                        else {  
                            // 高位节点  
                            if (hiTail == null)  
                                hiHead = e;  
                            else                                
	                            hiTail.next = e;  
                            hiTail = e;  
                        }  
  
                    } while ((e = next) != null);  
                    // mjy：低位链表不为空  
                    if (loTail != null) {  
                        // 尾节点下个节点置空  
                        loTail.next = null;  
                        // 头节点赋值新数组  
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
### get方法分析
核心是getNode

``` java

/**  
 * Returns the value to which the specified key is mapped, 
 * or {@code null} if this map contains no mapping for the key.  
 * 
 * <p>More formally, if this map contains a mapping from a key  
 * {@code k} to a value {@code v} such that {@code (key==null ? k==null :  
 * key.equals(k))}, then this method returns {@code v}; otherwise  
 * it returns {@code null}.  (There can be at most one such mapping.)  
 * 
 * <p>A return value of {@code null} does not <i>necessarily</i>  
 * indicate that the map contains no mapping for the key; it's also  
 * possible that the map explicitly maps the key to {@code null}.  
 * The {@link #containsKey containsKey} operation may be used to  
 * distinguish these two cases. 
 * 
 * @see #put(Object, Object)  
 */
 public V get(Object key) {  
    Node<K,V> e;  
    return (e = getNode(hash(key), key)) == null ? null : e.value;  
}  
  
/**  
 * Implements Map.get and related methods * * @param hash hash for key  
 * @param key the key  
 * @return the node, or null if none  
 */
 final Node<K,V> getNode(int hash, Object key) {  
    /**  
     * tab：当前哈希表（数组）  
     * first：桶位的头元素  
     * e：临时node元素  
     * n：数组长度  
     */  
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;  
    // mjy：当前数组不为空且长度大于0，且桶位中的头元素不为空  
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (first = tab[(n - 1) & hash]) != null) {  
        // mjy： 头元素和要取的元素完全一致，直接返回  
        if (first.hash == hash && // always check first node  
            ((k = first.key) == key || (key != null && key.equals(k))))  
            return first;  
        // mjy：不一致，判断（链表中）头元素的下个节点不为空  
        if ((e = first.next) != null) {  
            // mjy：红黑树  
            if (first instanceof TreeNode)  
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);  
            do {  
                // mjy：链表，循环判断  
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    return e;  
            } while ((e = e.next) != null);  
        }  
    }  
    return null;  
}

```
### remove方法分析
核心是removeNode

``` java

/**  
 * Removes the mapping for the specified key from this map if present. 
 * 
 * @param  key key whose mapping is to be removed from the map  
 * @return the previous value associated with <tt>key</tt>, or  
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.  
 *         (A <tt>null</tt> return can also indicate that the map  
 *         previously associated <tt>null</tt> with <tt>key</tt>.)  
 */public V remove(Object key) {  
    Node<K,V> e;  
    return (e = removeNode(hash(key), key, null, false, true)) == null ?  
        null : e.value;  
}  
  
/**  
 * Implements Map.remove and related methods * * @param hash hash for key  
 * @param key the key  
 * @param value the value to match if matchValue, else ignored  
 * @param matchValue if true only remove if value is equal  
 * @param movable if false do not move other nodes while removing  
 * @return the node, or null if none  
 */final Node<K,V> removeNode(int hash, Object key, Object value,  
                           boolean matchValue, boolean movable) {  
    /**  
     * mjy：  
     * tab：当前哈希表（数组）  
     * p：当前node元素  
     * n：数组长度  
     * index：删除的元素数组下标  
     */  
    Node<K,V>[] tab; Node<K,V> p; int n, index;  
    // 都不为空，有元素，需要进行查找并删除  
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (p = tab[index = (n - 1) & hash]) != null) {  
        /**  
         * node：查找到的结果  
         * e：临时，链表的下个元素，循环查找  
         */  
        Node<K,V> node = null, e; K k; V v;  
        // 当前桶位的头元素，即为要删除的元素（完全相等）赋值给node  
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k)))) {  
            node = p;  
        }  
        // mjy：头元素的下个元素不为空  
        else if ((e = p.next) != null) {  
            // mjy：当前桶位是红黑树  
            if (p instanceof TreeNode)  
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);  
            else {  
                // mjy：链表，循环查找，赋值给node  
                do {  
                    if (e.hash == hash &&  
                        ((k = e.key) == key ||  
                         (key != null && key.equals(k)))) {  
                        node = e;  
                        break;                    }  
                    p = e;  
                } while ((e = e.next) != null);  
            }  
        }  
        // mjy：node不为空，查找到了需要删除的元素，对比value  
        if (node != null && (!matchValue || (v = node.value) == value ||  
                             (value != null && value.equals(v)))) {  
            // 红黑树的删除逻辑  
            if (node instanceof TreeNode)  
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);  
            // mjy：桶位的头节点即为要删除的元素  
            else if (node == p)  
                // 当前桶位置指向 node的下个节点  
                tab[index] = node.next;  
            else                
	            // 当前p的下个节点，指向要删除的node的下个节点（即移除了node）  
                p.next = node.next;  
            ++modCount;  
            --size;  
            afterNodeRemoval(node);  
            return node;  
        }  
    }  
    return null;  
}
```



## 总结
HashMap1.8的底层是数组+链表+红黑树，每个数组元素其实是链表的头节点，链表过长就会转为红黑树。
其中每个节点底层是用Node对象实现的，Node中有hash，key，value，next节点几个属性。
HashMap在put的时候，会先判断数组是否为空，如果为空会先初始化为16，这是一个延时初始化。然后回通过寻址算法和key的hash值，找到它的下标位置，如果这个位置为空就直接插入，不为空会判断这个节点是否为红黑树，如果是红黑树就调用红黑树的插入方法，否则就是链表，就会插入链表的尾部。这个过程中都会判断存在的key和新的要插入的key是否完全相等，完全相等就会直接覆盖。插入完成后，会判断HashMap中的元素数量是否超过阈值，如果超过了就会触发扩容。
扩容每次都会扩容为原来的2倍，会创建一个新数组长度是原来的2倍，把原来数组中的数据挪动到新的数组中，扩容会进行一个rehash的操作，元素可能会停在原来的位置，可能会移动到原始位置+扩容的大小的位置。
