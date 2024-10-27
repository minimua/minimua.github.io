---
title: 'ArrayList源码解析'
date: 2023-02-01T22:00:00+08:00
tags: ["tech"]
categories: ["tech"]
---

# 简介

`ArrayList`继承于 **`AbstractList`** 类，实现了 **`List`**, **`RandomAccess`**, **`Cloneable`**, **`java.io.Serializable`** 这些接口。

- `List`：表明是一个列表，支持添加、删除、查找等操作，并且可以通过下标进行访问。
- `RandomAccess`：是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 `ArrayList` 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
- `Cloneable`  ：即覆盖了函数`clone()`，能被克隆。
- `Serializable`：这意味着`ArrayList`支持序列化，能通过序列化去传输。


``` java
public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
{}
```

## 源码

### 核心属性

``` java
public class ArrayListTest<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {  
    private static final long serialVersionUID = 8683452581122892189L;  
  
    /**  
     * Default initial capacity.     
     * mjy：默认初始容量大小  
     */  
    private static final int DEFAULT_CAPACITY = 10;  
  
    /**  
     * Shared empty array instance used for empty instances.     
     * mjy：空数组（用于空实例）  
     */  
    private static final Object[] EMPTY_ELEMENTDATA = {};  
  
    /**  
     * Shared empty array instance used for default sized empty instances. We     
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when     
     * first element is added.     
     * mjy：用于默认大小空实例的共享空数组实例。和EMPTY_ELEMENTDATA数组区分，以知道在添加第一个元素时容量需要增加多少。  
     */  
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};  
  
    /**  
     * The array buffer into which the elements of the ArrayList are stored.     
     * The capacity of the ArrayList is the length of this array buffer. Any     
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA     
     * will be expanded to DEFAULT_CAPACITY when the first element is added.    
     * mjy：存储ArrayList元素的数组缓冲区（就是保存ArrayList数据的数组）。  
     * ArrayList的容量是此数组缓冲区的长度。  
     * 添加第一个元素时，任何具有elementData==DEFAULTCAPACITY_EMMENTDATA的空ArrayList都将扩展为DEFAULT_CAPACITY。  
     */  
    transient Object[] elementData; // non-private to simplify nested class access  
  
    /**  
     * The size of the ArrayList (the number of elements it contains).     
     * mjy：数组长度大小（元素个数）  
     *  
     * @serial  
     */  
    private int size;

}   
```


### 构造方法

``` java
/**  
 * Constructs an empty list with the specified initial capacity. 
 * 
 * @param initialCapacity the initial capacity of the list  
 * @throws IllegalArgumentException if the specified initial capacity  
 *                                  is negative 
 * mjy：带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）  
 */  
public ArrayList(int initialCapacity) {  
    if (initialCapacity > 0) {  
        // 传入的参数大于0，创建initialCapacity大小的数组  
        this.elementData = new Object[initialCapacity];  
    } else if (initialCapacity == 0) {  
        // 传入的参数等于0，创建空数组  
        this.elementData = EMPTY_ELEMENTDATA;  
    } else {  
        // 其他情况异常  
        throw new IllegalArgumentException("Illegal Capacity: " +  
                initialCapacity);  
    }  
}  
  
/**  
 * Constructs an empty list with an initial capacity of ten. 
 * mjy：默认无参构造函数 DEFAULTCAPACITY_EMPTY_ELEMENTDATA为0。初始其实是空数组，当添加第一个元素的时候数组容量才变成10  
 */
 public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}  
  
/**  
 * Constructs a list containing the elements of the specified 
 * collection, in the order they are returned by the collection's 
 * iterator. 
 * 
 * @param c the collection whose elements are to be placed into this list  
 * @throws NullPointerException if the specified collection is null  
 * mjy：构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。 new ArrayList<>(paramList);  
 */public ArrayList(Collection<? extends E> c) {  
    // mjy：将参数转换为数组  
    elementData = c.toArray();  
    // elementData数组的长度不为0  
    if ((size = elementData.length) != 0) {  
        // c.toArray might (incorrectly) not return Object[] (see 6260652)  
        // mjy：elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组）  
        if (elementData.getClass() != Object[].class)  
            // mjy：将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组  
            elementData = Arrays.copyOf(elementData, size, Object[].class);  
    } else {  
        // replace with empty array. mjy： elementData数组的长度为0，即为空数组  
        this.elementData = EMPTY_ELEMENTDATA;  
    }  
}
```


### add

``` java
/**  
 * Appends the specified element to the end of this list. 
 * 
 * @param e element to be appended to this list  
 * @return <tt>true</tt> (as specified by {@link Collection#add})  
 * mjy： 添加元素，将指定的元素追加到此列表的末尾  
 */  
public boolean add(E e) {  
    // mjy： 扩容校验  
    ensureCapacityInternal(size + 1);  // Increments modCount!!  
    // mjy： 添加元素  
    elementData[size++] = e;  
    return true;
}


// mjy： 确保内部容量  
private void ensureCapacityInternal(int minCapacity) {  
    // mjy：如果元素是空  
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        // mjy： 所需要的最小容量为 默认容量和最小容量相比 较大的一个  
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);  
    }  
    // 确保 容量  
    ensureExplicitCapacity(minCapacity);  
}  
  
private void ensureExplicitCapacity(int minCapacity) {  
    modCount++;  
  
    // overflow-conscious code  
    // mjy：所需要的最小容量减去现在的数组元素长度 大于0  
    if (minCapacity - elementData.length > 0)  
        // mjy：调用grow方法进行扩容，调用此方法代表已经开始扩容了  
        grow(minCapacity);  
}




```

### grow

``` java
/**  
 * The maximum size of array to allocate. 
 * Some VMs reserve some header words in an array. 
 * Attempts to allocate larger arrays may result in 
 * OutOfMemoryError: Requested array size exceeds VM limit 
 * mjy： 是JVM能分配的数组的最大 大小，一般为Integer.MAX_VALUE - 8（这和JVM内部实现有关，留出了一些用于对象头信息的空间）。  
 */  
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;  
  
/**  
 * Increases the capacity to ensure that it can hold at least the 
 * number of elements specified by the minimum capacity argument. 
 * mjy：ArrayList扩容的核心方法。  
 *  
 * @param minCapacity the desired minimum capacity  
 */
 private void grow(int minCapacity) {  
    // overflow-conscious code  
    // mjy： 表示当前数组的容量（即当前存储元素的数组 elementData 的长度）  
    int oldCapacity = elementData.length;  
    // mjy：是新的数组容量，计算方式是旧容量的 1.5 倍 (oldCapacity + oldCapacity >> 1，即右移一位表示除以2)。通过这种方式，ArrayList 的容量会在需要时以指数方式增加，避免频繁扩容。  
    int newCapacity = oldCapacity + (oldCapacity >> 1);  
    // mjy：minCapacity是调用grow方法时传入的参数，表示所需的最小容量。如果计算出的newCapacity小于minCapacity，那么直接将newCapacity设置为minCapacity，确保新容量至少能满足当前所需的空间。  
    if (newCapacity - minCapacity < 0)  
        newCapacity = minCapacity;  
    // mjy：如果newCapacity超过了MAX_ARRAY_SIZE，调用 hugeCapacity(minCapacity) 方法来决定实际的容量。  
    if (newCapacity - MAX_ARRAY_SIZE > 0)  
        newCapacity = hugeCapacity(minCapacity);  
    // minCapacity is usually close to size, so this is a win:  
    // mjy：调用Arrays.copyOf方法将现有的数组elementData复制到一个新的数组中，新数组的长度为 newCapacity，实现了数组的扩容。这个过程会保留原数组中的所有元素，但底层的存储空间已经扩展。  
    elementData = Arrays.copyOf(elementData, newCapacity);  
}  
  
// mjy： 处理可能超过最大数组大小的情况  
private static int hugeCapacity(int minCapacity) {  
    // mjy： 如果minCapacity为负数，意味着发生了容量溢出（例如，容量计算溢出了Integer的范围），此时抛出 OutOfMemoryError    
    if (minCapacity < 0) // overflow  
        throw new OutOfMemoryError();  
    // mjy：如果minCapacity超过了MAX_ARRAY_SIZE，返回Integer.MAX_VALUE，即数组会扩展到 JVM 可以支持的最大值。否则即minCapacity小于MAX_ARRAY_SIZE，但仍然超出了合理范围，则返回 MAX_ARRAY_SIZE。  
    return (minCapacity > MAX_ARRAY_SIZE) ?  
            Integer.MAX_VALUE :  
            MAX_ARRAY_SIZE;  
}
```

# 扩容分析


![](https://r2.voidmu.com/20241006210735.png)
