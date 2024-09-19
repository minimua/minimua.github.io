---
title: "ArrayList源码解析 building"
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


# 源码分析


``` java
package java.util;  
  
import java.util.function.Consumer;  
import java.util.function.Predicate;  
import java.util.function.UnaryOperator;  
  
  
public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {  
    private static final long serialVersionUID = 8683452581122892189L;  
  
    /**  
     * Default initial capacity.  mjy：默认初始容量大小  
     */  
    private static final int DEFAULT_CAPACITY = 10;  
  
    /**  
     * Shared empty array instance used for empty instances. mjy：空数组（用于空实例）。  
     */  
    private static final Object[] EMPTY_ELEMENTDATA = {};  
  
    /**  
     * Shared empty array instance used for default sized empty instances. We     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when     * first element is added.     */    // mjy：用于默认大小空实例的共享空数组实例。和EMPTY_ELEMENTDATA数组区分，以知道在添加第一个元素时容量需要增加多少。  
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};  
  
    /**  
     * The array buffer into which the elements of the ArrayList are stored.     * The capacity of the ArrayList is the length of this array buffer. Any     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA     * will be expanded to DEFAULT_CAPACITY when the first element is added.     */    // mjy：存储ArrayList元素的数组缓冲区（就是保存ArrayList数据的数组）。ArrayList的容量是此数组缓冲区的长度。添加第一个元素时，任何具有elementData==DEFAULTCAPACITY_EMMENTDATA的空ArrayList都将扩展为DEFAULT_CAPACITY。  
    transient Object[] elementData; // non-private to simplify nested class access  
  
    /**  
     * The size of the ArrayList (the number of elements it contains). mjy：数组长度大小（元素个数）  
     *  
     * @serial  
     */  
    private int size;  
  
    /**  
     * Constructs an empty list with the specified initial capacity.     *     * @param initialCapacity the initial capacity of the list  
     * @throws IllegalArgumentException if the specified initial capacity  
     *                                  is negative     */    // mjy：带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）  
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
     * Constructs an empty list with an initial capacity of ten.     */    // mjy：默认无参构造函数 DEFAULTCAPACITY_EMPTY_ELEMENTDATA为0。初始其实是空数组，当添加第一个元素的时候数组容量才变成10  
    public ArrayList() {  
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
    }  
  
    /**  
     * Constructs a list containing the elements of the specified     * collection, in the order they are returned by the collection's     * iterator.     *     * @param c the collection whose elements are to be placed into this list  
     * @throws NullPointerException if the specified collection is null  
     */    // mjy：构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。 new ArrayList<>(paramList);    public ArrayList(Collection<? extends E> c) {  
        // 将参数转换为数组  
        elementData = c.toArray();  
        // elementData数组的长度不为0  
        if ((size = elementData.length) != 0) {  
            // c.toArray might (incorrectly) not return Object[] (see 6260652)  
            // mjy： elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组）  
            if (elementData.getClass() != Object[].class)  
                // mjy： 将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组  
                elementData = Arrays.copyOf(elementData, size, Object[].class);  
        } else {  
            // replace with empty array. mjy： elementData数组的长度为0，即为空数组  
            this.elementData = EMPTY_ELEMENTDATA;  
        }  
    }  
  
    /**  
     * Trims the capacity of this <tt>ArrayList</tt> instance to be the  
     * list's current size.  An application can use this operation to minimize     * the storage of an <tt>ArrayList</tt> instance.  
     */    // mjy： 修改这个ArrayList实例的容量是列表的当前大小。 可以使用此操作来最小化ArrayList实例的存储。  
    public void trimToSize() {  
        modCount++;  
        if (size < elementData.length) {  
            elementData = (size == 0)  
                    ? EMPTY_ELEMENTDATA  
                    : Arrays.copyOf(elementData, size);  
        }  
    }  
  
    /**  
     * Increases the capacity of this <tt>ArrayList</tt> instance, if  
     * necessary, to ensure that it can hold at least the number of elements     * specified by the minimum capacity argument.     * mjy： 如有必要，增加此ArrayList实例的容量，以确保它至少可以容纳最小容量参数指定的元素数量。  
     *  
     * @param minCapacity the desired minimum capacity   mjy：所需的最小容量  
     */  
    public void ensureCapacity(int minCapacity) {  
        // mjy： (数组元素不为空) 如果是true，minExpand的值为0，如果是false,minExpand的值为10  
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)  
                // any size if not default element table  
                ? 0  
                // larger than default for default empty table. It's already  
                // supposed to be at default size.                : DEFAULT_CAPACITY;  
        // 如果最小容量大于已有的最大容量  
        if (minCapacity > minExpand) {  
            ensureExplicitCapacity(minCapacity);  
        }  
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
  
    /**  
     * The maximum size of array to allocate.     * Some VMs reserve some header words in an array.     * Attempts to allocate larger arrays may result in     * OutOfMemoryError: Requested array size exceeds VM limit     * mjy： 是JVM能分配的数组的最大 大小，一般为Integer.MAX_VALUE - 8（这和JVM内部实现有关，留出了一些用于对象头信息的空间）。  
     */  
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;  
  
    /**  
     * Increases the capacity to ensure that it can hold at least the     * number of elements specified by the minimum capacity argument.     * mjy：ArrayList扩容的核心方法。  
     *  
     * @param minCapacity the desired minimum capacity  
     */    private void grow(int minCapacity) {  
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
        // mjy： 如果minCapacity为负数，意味着发生了容量溢出（例如，容量计算溢出了Integer的范围），此时抛出 OutOfMemoryError        if (minCapacity < 0) // overflow  
            throw new OutOfMemoryError();  
        // mjy：如果minCapacity超过了MAX_ARRAY_SIZE，返回Integer.MAX_VALUE，即数组会扩展到 JVM 可以支持的最大值。否则即minCapacity小于MAX_ARRAY_SIZE，但仍然超出了合理范围，则返回 MAX_ARRAY_SIZE。  
        return (minCapacity > MAX_ARRAY_SIZE) ?  
                Integer.MAX_VALUE :  
                MAX_ARRAY_SIZE;  
    }  
  
    /**  
     * Returns the number of elements in this list.     * mjy： 返回此列表中的元素数  
     *  
     * @return the number of elements in this list  
     */    public int size() {  
        return size;  
    }  
  
    /**  
     * Returns <tt>true</tt> if this list contains no elements.  
     * mjy： 判断是否为空 如果此列表不包含元素，则返回 true  
     *     * @return <tt>true</tt> if this list contains no elements  
     */    public boolean isEmpty() {  
        return size == 0;  
    }  
  
    /**  
     * Returns <tt>true</tt> if this list contains the specified element.  
     * More formally, returns <tt>true</tt> if and only if this list contains  
     * at least one element <tt>e</tt> such that  
     * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>.  
     *     * @param o element whose presence in this list is to be tested  
     * @return <tt>true</tt> if this list contains the specified element  
     */    // mjy： 判断是否包含指定元素，包含返回true  
    public boolean contains(Object o) {  
        // mjy： indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1  
        return indexOf(o) >= 0;  
    }  
  
    /**  
     * Returns the index of the first occurrence of the specified element     * in this list, or -1 if this list does not contain the element.     * More formally, returns the lowest index <tt>i</tt> such that  
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,  
     * or -1 if there is no such index.     */    // mjy： 返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1  
    public int indexOf(Object o) {  
        // mjy： 入参元素为空  
        if (o == null) {  
            for (int i = 0; i < size; i++)  
                // mjy： 判断是否存在空元素  
                if (elementData[i] == null)  
                    return i;  
        } else {  
            // 不为空  
            for (int i = 0; i < size; i++)  
                // mjy： equals()方法比较  
                if (o.equals(elementData[i]))  
                    return i;  
        }  
        return -1;  
    }  
  
    /**  
     * Returns the index of the last occurrence of the specified element     * in this list, or -1 if this list does not contain the element.     * More formally, returns the highest index <tt>i</tt> such that  
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,  
     * or -1 if there is no such index.     */    // mjy： 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1  
    public int lastIndexOf(Object o) {  
        // 倒序循环  
        if (o == null) {  
            for (int i = size - 1; i >= 0; i--)  
                if (elementData[i] == null)  
                    return i;  
        } else {  
            for (int i = size - 1; i >= 0; i--)  
                if (o.equals(elementData[i]))  
                    return i;  
        }  
        return -1;  
    }  
  
    /**  
     * Returns a shallow copy of this <tt>ArrayList</tt> instance.  (The  
     * elements themselves are not copied.)     *     * @return a clone of this <tt>ArrayList</tt> instance  
     */    // mjy： 克隆方法，返回此ArrayList实例的浅拷 （元素本身不被复制。）  
    public Object clone() {  
        try {  
            ArrayList<?> v = (ArrayList<?>) super.clone();  
            // mjy： Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度  
            v.elementData = Arrays.copyOf(elementData, size);  
            v.modCount = 0;  
            return v;  
        } catch (CloneNotSupportedException e) {  
            // this shouldn't happen, since we are Cloneable  
            // mjy： 这不应该发生，因为我们是可以克隆的  
            throw new InternalError(e);  
        }  
    }  
  
    /**  
     * Returns an array containing all of the elements in this list     * in proper sequence (from first to last element).     * mjy： 以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。  
     * <p>The returned array will be "safe" in that no references to it are  
     * maintained by this list.  (In other words, this method must allocate     * a new array).  The caller is thus free to modify the returned array.     * mjy： 返回的数组将是安全的，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。  
     * <p>This method acts as bridge between array-based and collection-based  
     * APIs.     * mjy： 因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。  
     *  
     * @return an array containing all of the elements in this list in  
     * proper sequence     */    public Object[] toArray() {  
        return Arrays.copyOf(elementData, size);  
    }  
  
    /**  
     * Returns an array containing all of the elements in this list in proper     * sequence (from first to last element); the runtime type of the returned     * array is that of the specified array.  If the list fits in the     * specified array, it is returned therein.  Otherwise, a new array is     * allocated with the runtime type of the specified array and the size of     * this list.     *     * <p>If the list fits in the specified array with room to spare  
     * (i.e., the array has more elements than the list), the element in     * the array immediately following the end of the collection is set to     * <tt>null</tt>.  (This is useful in determining the length of the  
     * list <i>only</i> if the caller knows that the list does not contain  
     * any null elements.)     *     * @param a the array into which the elements of the list are to  
     *          be stored, if it is big enough; otherwise, a new array of the     *          same runtime type is allocated for this purpose.     * @return an array containing the elements of the list  
     * @throws ArrayStoreException  if the runtime type of the specified array  
     *                              is not a supertype of the runtime type of every element in     *                              this list     * @throws NullPointerException if the specified array is null  
     */    @SuppressWarnings("unchecked")  
    public <T> T[] toArray(T[] a) {  
        if (a.length < size)  
            // Make a new array of a's runtime type, but my contents:  
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());  
        System.arraycopy(elementData, 0, a, 0, size);  
        if (a.length > size)  
            a[size] = null;  
        return a;  
    }  
  
    // Positional Access Operations  
  
    @SuppressWarnings("unchecked")  
    E elementData(int index) {  
        return (E) elementData[index];  
    }  
  
    /**  
     * Returns the element at the specified position in this list.     * mjy： 返回此列表中指定位置的元素  
     *  
     * @param index index of the element to return  
     * @return the element at the specified position in this list  
     * @throws IndexOutOfBoundsException {@inheritDoc}  
     */    public E get(int index) {  
        // mjy： 校验数组下标越界  
        rangeCheck(index);  
  
        return elementData(index);  
    }  
  
    /**  
     * Replaces the element at the specified position in this list with     * the specified element.     * mjy： 用指定的元素替换此列表中指定位置的元素  
     *  
     * @param index   index of the element to replace  
     * @param element element to be stored at the specified position  
     * @return the element previously at the specified position  
     * @throws IndexOutOfBoundsException {@inheritDoc}  
     */    public E set(int index, E element) {  
        // mjy： 校验数组下标越界  
        rangeCheck(index);  
        // mjy： 旧的值  
        E oldValue = elementData(index);  
        // mjy： 新的赋值  
        elementData[index] = element;  
        // mjy： 返回旧的值  
        return oldValue;  
    }  
  
    /**  
     * Appends the specified element to the end of this list.     * mjy： 添加元素，将指定的元素追加到此列表的末尾  
     *  
     * @param e element to be appended to this list  
     * @return <tt>true</tt> (as specified by {@link Collection#add})  
     */    public boolean add(E e) {  
        // mjy： 扩容校验  
        ensureCapacityInternal(size + 1);  // Increments modCount!!  
        // mjy： 添加元素  
        elementData[size++] = e;  
        return true;    }  
  
    /**  
     * Inserts the specified element at the specified position in this     * list. Shifts the element currently at that position (if any) and     * any subsequent elements to the right (adds one to their indices).     * mjy： 添加元素，在此列表中的指定位置插入指定的元素  
     * mjy：先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。  
     *  
     * @param index   index at which the specified element is to be inserted  
     * @param element element to be inserted  
     * @throws IndexOutOfBoundsException {@inheritDoc}  
     */    public void add(int index, E element) {  
        // mjy： 校验  
        rangeCheckForAdd(index);  
  
        ensureCapacityInternal(size + 1);  // Increments modCount!!  
        // mjy： System.arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己  
        System.arraycopy(elementData, index, elementData, index + 1,  
                size - index);  
        elementData[index] = element;  
        size++;  
    }  
  
    /**  
     * Removes the element at the specified position in this list.     * Shifts any subsequent elements to the left (subtracts one from their     * indices).     * mjy：删除元素，删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）  
     *  
     * @param index the index of the element to be removed  
     * @return the element that was removed from the list  
     * @throws IndexOutOfBoundsException {@inheritDoc}  
     */    public E remove(int index) {  
        rangeCheck(index);  
  
        modCount++;  
        E oldValue = elementData(index);  
  
        int numMoved = size - index - 1;  
        if (numMoved > 0)  
            System.arraycopy(elementData, index + 1, elementData, index,  
                    numMoved);  
        elementData[--size] = null; // clear to let GC do its work  
  
        return oldValue;  
    }  
  
    /**  
     * Removes the first occurrence of the specified element from this list,     * if it is present.  If the list does not contain the element, it is     * unchanged.  More formally, removes the element with the lowest index     * <tt>i</tt> such that  
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>  
     * (if such an element exists).  Returns <tt>true</tt> if this list  
     * contained the specified element (or equivalently, if this list     * changed as a result of the call).     * mjy： 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。  
     *  
     * @param o element to be removed from this list, if present  
     * @return <tt>true</tt> if this list contained the specified element  
     */    public boolean remove(Object o) {  
        if (o == null) {  
            for (int index = 0; index < size; index++)  
                if (elementData[index] == null) {  
                    fastRemove(index);  
                    return true;                }  
        } else {  
            for (int index = 0; index < size; index++)  
                if (o.equals(elementData[index])) {  
                    fastRemove(index);  
                    return true;                }  
        }  
        return false;  
    }  
  
    /*  
     * Private remove method that skimjy bounds checking and does not     * return the value removed.     */    private void fastRemove(int index) {  
        modCount++;  
        int numMoved = size - index - 1;  
        if (numMoved > 0)  
            System.arraycopy(elementData, index + 1, elementData, index,  
                    numMoved);  
        elementData[--size] = null; // clear to let GC do its work  
    }  
  
    /**  
     * Removes all of the elements from this list.  The list will     * be empty after this call returns.     * mjy： 从列表中清空（删除）所有元素。  
     */  
    public void clear() {  
        modCount++;  
  
        // clear to let GC do its work  
        for (int i = 0; i < size; i++)  
            elementData[i] = null;  
  
        size = 0;  
    }  
  
    /**  
     * Appends all of the elements in the specified collection to the end of     * this list, in the order that they are returned by the     * specified collection's Iterator.  The behavior of this operation is     * undefined if the specified collection is modified while the operation     * is in progress.  (This implies that the behavior of this call is     * undefined if the specified collection is this list, and this     * list is nonempty.)     *     * @param c collection containing elements to be added to this list  
     * @return <tt>true</tt> if this list changed as a result of the call  
     * @throws NullPointerException if the specified collection is null  
     */    public boolean addAll(Collection<? extends E> c) {  
        Object[] a = c.toArray();  
        int numNew = a.length;  
        ensureCapacityInternal(size + numNew);  // Increments modCount  
        System.arraycopy(a, 0, elementData, size, numNew);  
        size += numNew;  
        return numNew != 0;  
    }  
  
    /**  
     * Inserts all of the elements in the specified collection into this     * list, starting at the specified position.  Shifts the element     * currently at that position (if any) and any subsequent elements to     * the right (increases their indices).  The new elements will appear     * in the list in the order that they are returned by the     * specified collection's iterator.     *     * @param index index at which to insert the first element from the  
     *              specified collection     * @param c     collection containing elements to be added to this list  
     * @return <tt>true</tt> if this list changed as a result of the call  
     * @throws IndexOutOfBoundsException {@inheritDoc}  
     * @throws NullPointerException      if the specified collection is null  
     */    public boolean addAll(int index, Collection<? extends E> c) {  
        rangeCheckForAdd(index);  
  
        Object[] a = c.toArray();  
        int numNew = a.length;  
        ensureCapacityInternal(size + numNew);  // Increments modCount  
  
        int numMoved = size - index;  
        if (numMoved > 0)  
            System.arraycopy(elementData, index, elementData, index + numNew,  
                    numMoved);  
  
        System.arraycopy(a, 0, elementData, index, numNew);  
        size += numNew;  
        return numNew != 0;  
    }  
  
    /**  
     * Removes from this list all of the elements whose index is between     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.  
     * Shifts any succeeding elements to the left (reduces their index).     * This call shortens the list by {@code (toIndex - fromIndex)} elements.  
     * (If {@code toIndex==fromIndex}, this operation has no effect.)  
     *     * @throws IndexOutOfBoundsException if {@code fromIndex} or  
     *                                   {@code toIndex} is out of range  
     *                                   ({@code fromIndex < 0 ||  
     *                                   fromIndex >= size() ||     *                                   toIndex > size() ||     *                                   toIndex < fromIndex})     */    protected void removeRange(int fromIndex, int toIndex) {  
        modCount++;  
        int numMoved = size - toIndex;  
        System.arraycopy(elementData, toIndex, elementData, fromIndex,  
                numMoved);  
  
        // clear to let GC do its work  
        int newSize = size - (toIndex - fromIndex);  
        for (int i = newSize; i < size; i++) {  
            elementData[i] = null;  
        }  
        size = newSize;  
    }  
  
    /**  
     * Checks if the given index is in range.  If not, throws an appropriate     * runtime exception.  This method does *not* check if the index is     * negative: It is always used immediately prior to an array access,     * which throws an ArrayIndexOutOfBoundsException if index is negative.     */    private void rangeCheck(int index) {  
        if (index >= size)  
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
    }  
  
    /**  
     * A version of rangeCheck used by add and addAll.     */    private void rangeCheckForAdd(int index) {  
        if (index > size || index < 0)  
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
    }  
  
    /**  
     * Constructs an IndexOutOfBoundsException detail message.     * Of the many possible refactorings of the error handling code,     * this "outlining" performs best with both server and client VMs.     */    private String outOfBoundsMsg(int index) {  
        return "Index: " + index + ", Size: " + size;  
    }  
  
    /**  
     * Removes from this list all of its elements that are contained in the     * specified collection.     *     * @param c collection containing elements to be removed from this list  
     * @return {@code true} if this list changed as a result of the call  
     * @throws ClassCastException   if the class of an element of this list  
     *                              is incompatible with the specified collection     *                              (<a href="Collection.html#optional-restrictions">optional</a>)  
     * @throws NullPointerException if this list contains a null element and the  
     *                              specified collection does not permit null elements     *                              (<a href="Collection.html#optional-restrictions">optional</a>),  
     *                              or if the specified collection is null     * @see Collection#contains(Object)  
     */    public boolean removeAll(Collection<?> c) {  
        Objects.requireNonNull(c);  
        return batchRemove(c, false);  
    }  
  
    /**  
     * Retains only the elements in this list that are contained in the     * specified collection.  In other words, removes from this list all     * of its elements that are not contained in the specified collection.     *     * @param c collection containing elements to be retained in this list  
     * @return {@code true} if this list changed as a result of the call  
     * @throws ClassCastException   if the class of an element of this list  
     *                              is incompatible with the specified collection     *                              (<a href="Collection.html#optional-restrictions">optional</a>)  
     * @throws NullPointerException if this list contains a null element and the  
     *                              specified collection does not permit null elements     *                              (<a href="Collection.html#optional-restrictions">optional</a>),  
     *                              or if the specified collection is null     * @see Collection#contains(Object)  
     */    public boolean retainAll(Collection<?> c) {  
        Objects.requireNonNull(c);  
        return batchRemove(c, true);  
    }  
  
    private boolean batchRemove(Collection<?> c, boolean complement) {  
        final Object[] elementData = this.elementData;  
        int r = 0, w = 0;  
        boolean modified = false;  
        try {  
            for (; r < size; r++)  
                if (c.contains(elementData[r]) == complement)  
                    elementData[w++] = elementData[r];  
        } finally {  
            // Preserve behavioral compatibility with AbstractCollection,  
            // even if c.contains() throws.            if (r != size) {  
                System.arraycopy(elementData, r,  
                        elementData, w,  
                        size - r);  
                w += size - r;  
            }  
            if (w != size) {  
                // clear to let GC do its work  
                for (int i = w; i < size; i++)  
                    elementData[i] = null;  
                modCount += size - w;  
                size = w;  
                modified = true;  
            }  
        }  
        return modified;  
    }  
  
    /**  
     * Save the state of the <tt>ArrayList</tt> instance to a stream (that  
     * is, serialize it).     *     * @serialData The length of the array backing the <tt>ArrayList</tt>  
     * instance is emitted (int), followed by all of its elements  
     * (each an <tt>Object</tt>) in the proper order.  
     */    private void writeObject(java.io.ObjectOutputStream s)  
            throws java.io.IOException {  
        // Write out element count, and any hidden stuff  
        int expectedModCount = modCount;  
        s.defaultWriteObject();  
  
        // Write out size as capacity for behavioural compatibility with clone()  
        s.writeInt(size);  
  
        // Write out all elements in the proper order.  
        for (int i = 0; i < size; i++) {  
            s.writeObject(elementData[i]);  
        }  
  
        if (modCount != expectedModCount) {  
            throw new ConcurrentModificationException();  
        }  
    }  
  
    /**  
     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,  
     * deserialize it).     */    private void readObject(java.io.ObjectInputStream s)  
            throws java.io.IOException, ClassNotFoundException {  
        elementData = EMPTY_ELEMENTDATA;  
  
        // Read in size, and any hidden stuff  
        s.defaultReadObject();  
  
        // Read in capacity  
        s.readInt(); // ignored  
  
        if (size > 0) {  
            // be like clone(), allocate array based upon size not capacity  
            ensureCapacityInternal(size);  
  
            Object[] a = elementData;  
            // Read in all elements in the proper order.  
            for (int i = 0; i < size; i++) {  
                a[i] = s.readObject();  
            }  
        }  
    }  
  
    /**  
     * Returns a list iterator over the elements in this list (in proper     * sequence), starting at the specified position in the list.     * The specified index indicates the first element that would be     * returned by an initial call to {@link ListIterator#next next}.  
     * An initial call to {@link ListIterator#previous previous} would  
     * return the element with the specified index minus one.     *     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.  
     *     * @throws IndexOutOfBoundsException {@inheritDoc}  
     */    public ListIterator<E> listIterator(int index) {  
        if (index < 0 || index > size)  
            throw new IndexOutOfBoundsException("Index: " + index);  
        return new ListItr(index);  
    }  
  
    /**  
     * Returns a list iterator over the elements in this list (in proper     * sequence).     *     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.  
     *     * @see #listIterator(int)  
     */    public ListIterator<E> listIterator() {  
        return new ListItr(0);  
    }  
  
    /**  
     * Returns an iterator over the elements in this list in proper sequence.     *     * <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.  
     *     * @return an iterator over the elements in this list in proper sequence  
     */    public Iterator<E> iterator() {  
        return new Itr();  
    }  
  
    /**  
     * An optimized version of AbstractList.Itr     */    private class Itr implements Iterator<E> {  
        int cursor;       // index of next element to return  
        int lastRet = -1; // index of last element returned; -1 if no such  
        int expectedModCount = modCount;  
  
        public boolean hasNext() {  
            return cursor != size;  
        }  
  
        @SuppressWarnings("unchecked")  
        public E next() {  
            checkForComodification();  
            int i = cursor;  
            if (i >= size)  
                throw new NoSuchElementException();  
            Object[] elementData = ArrayList.this.elementData;  
            if (i >= elementData.length)  
                throw new ConcurrentModificationException();  
            cursor = i + 1;  
            return (E) elementData[lastRet = i];  
        }  
  
        public void remove() {  
            if (lastRet < 0)  
                throw new IllegalStateException();  
            checkForComodification();  
  
            try {  
                ArrayList.this.remove(lastRet);  
                cursor = lastRet;  
                lastRet = -1;  
                expectedModCount = modCount;  
            } catch (IndexOutOfBoundsException ex) {  
                throw new ConcurrentModificationException();  
            }  
        }  
  
        @Override  
        @SuppressWarnings("unchecked")  
        public void forEachRemaining(Consumer<? super E> consumer) {  
            Objects.requireNonNull(consumer);  
            final int size = ArrayList.this.size;  
            int i = cursor;  
            if (i >= size) {  
                return;  
            }  
            final Object[] elementData = ArrayList.this.elementData;  
            if (i >= elementData.length) {  
                throw new ConcurrentModificationException();  
            }  
            while (i != size && modCount == expectedModCount) {  
                consumer.accept((E) elementData[i++]);  
            }  
            // update once at end of iteration to reduce heap write traffic  
            cursor = i;  
            lastRet = i - 1;  
            checkForComodification();  
        }  
  
        final void checkForComodification() {  
            if (modCount != expectedModCount)  
                throw new ConcurrentModificationException();  
        }  
    }  
  
    /**  
     * An optimized version of AbstractList.ListItr     */    private class ListItr extends Itr implements ListIterator<E> {  
        ListItr(int index) {  
            super();  
            cursor = index;  
        }  
  
        public boolean hasPrevious() {  
            return cursor != 0;  
        }  
  
        public int nextIndex() {  
            return cursor;  
        }  
  
        public int previousIndex() {  
            return cursor - 1;  
        }  
  
        @SuppressWarnings("unchecked")  
        public E previous() {  
            checkForComodification();  
            int i = cursor - 1;  
            if (i < 0)  
                throw new NoSuchElementException();  
            Object[] elementData = ArrayList.this.elementData;  
            if (i >= elementData.length)  
                throw new ConcurrentModificationException();  
            cursor = i;  
            return (E) elementData[lastRet = i];  
        }  
  
        public void set(E e) {  
            if (lastRet < 0)  
                throw new IllegalStateException();  
            checkForComodification();  
  
            try {  
                ArrayList.this.set(lastRet, e);  
            } catch (IndexOutOfBoundsException ex) {  
                throw new ConcurrentModificationException();  
            }  
        }  
  
        public void add(E e) {  
            checkForComodification();  
  
            try {  
                int i = cursor;  
                ArrayList.this.add(i, e);  
                cursor = i + 1;  
                lastRet = -1;  
                expectedModCount = modCount;  
            } catch (IndexOutOfBoundsException ex) {  
                throw new ConcurrentModificationException();  
            }  
        }  
    }  
  
    /**  
     * Returns a view of the portion of this list between the specified     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.  (If  
     * {@code fromIndex} and {@code toIndex} are equal, the returned list is  
     * empty.)  The returned list is backed by this list, so non-structural     * changes in the returned list are reflected in this list, and vice-versa.     * The returned list supports all of the optional list operations.     *     * <p>This method eliminates the need for explicit range operations (of  
     * the sort that commonly exist for arrays).  Any operation that expects     * a list can be used as a range operation by passing a subList view     * instead of a whole list.  For example, the following idiom     * removes a range of elements from a list:     * <pre>  
     *      list.subList(from, to).clear();  
     * </pre>  
     * Similar idioms may be constructed for {@link #indexOf(Object)} and  
     * {@link #lastIndexOf(Object)}, and all of the algorithms in the  
     * {@link Collections} class can be applied to a subList.  
     *     * <p>The semantics of the list returned by this method become undefined if  
     * the backing list (i.e., this list) is <i>structurally modified</i> in  
     * any way other than via the returned list.  (Structural modifications are     * those that change the size of this list, or otherwise perturb it in such     * a fashion that iterations in progress may yield incorrect results.)     *     * @throws IndexOutOfBoundsException {@inheritDoc}  
     * @throws IllegalArgumentException  {@inheritDoc}  
     */    public List<E> subList(int fromIndex, int toIndex) {  
        subListRangeCheck(fromIndex, toIndex, size);  
        return new SubList(this, 0, fromIndex, toIndex);  
    }  
  
    static void subListRangeCheck(int fromIndex, int toIndex, int size) {  
        if (fromIndex < 0)  
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);  
        if (toIndex > size)  
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);  
        if (fromIndex > toIndex)  
            throw new IllegalArgumentException("fromIndex(" + fromIndex +  
                    ") > toIndex(" + toIndex + ")");  
    }  
  
    private class SubList extends AbstractList<E> implements RandomAccess {  
        private final AbstractList<E> parent;  
        private final int parentOffset;  
        private final int offset;  
        int size;  
  
        SubList(AbstractList<E> parent,  
                int offset, int fromIndex, int toIndex) {  
            this.parent = parent;  
            this.parentOffset = fromIndex;  
            this.offset = offset + fromIndex;  
            this.size = toIndex - fromIndex;  
            this.modCount = ArrayList.this.modCount;  
        }  
  
        public E set(int index, E e) {  
            rangeCheck(index);  
            checkForComodification();  
            E oldValue = ArrayList.this.elementData(offset + index);  
            ArrayList.this.elementData[offset + index] = e;  
            return oldValue;  
        }  
  
        public E get(int index) {  
            rangeCheck(index);  
            checkForComodification();  
            return ArrayList.this.elementData(offset + index);  
        }  
  
        public int size() {  
            checkForComodification();  
            return this.size;  
        }  
  
        public void add(int index, E e) {  
            rangeCheckForAdd(index);  
            checkForComodification();  
            parent.add(parentOffset + index, e);  
            this.modCount = parent.modCount;  
            this.size++;  
        }  
  
        public E remove(int index) {  
            rangeCheck(index);  
            checkForComodification();  
            E result = parent.remove(parentOffset + index);  
            this.modCount = parent.modCount;  
            this.size--;  
            return result;  
        }  
  
        protected void removeRange(int fromIndex, int toIndex) {  
            checkForComodification();  
            parent.removeRange(parentOffset + fromIndex,  
                    parentOffset + toIndex);  
            this.modCount = parent.modCount;  
            this.size -= toIndex - fromIndex;  
        }  
  
        public boolean addAll(Collection<? extends E> c) {  
            return addAll(this.size, c);  
        }  
  
        public boolean addAll(int index, Collection<? extends E> c) {  
            rangeCheckForAdd(index);  
            int cSize = c.size();  
            if (cSize == 0)  
                return false;  
  
            checkForComodification();  
            parent.addAll(parentOffset + index, c);  
            this.modCount = parent.modCount;  
            this.size += cSize;  
            return true;        }  
  
        public Iterator<E> iterator() {  
            return listIterator();  
        }  
  
        public ListIterator<E> listIterator(final int index) {  
            checkForComodification();  
            rangeCheckForAdd(index);  
            final int offset = this.offset;  
  
            return new ListIterator<E>() {  
                int cursor = index;  
                int lastRet = -1;  
                int expectedModCount = ArrayList.this.modCount;  
  
                public boolean hasNext() {  
                    return cursor != SubList.this.size;  
                }  
  
                @SuppressWarnings("unchecked")  
                public E next() {  
                    checkForComodification();  
                    int i = cursor;  
                    if (i >= SubList.this.size)  
                        throw new NoSuchElementException();  
                    Object[] elementData = ArrayList.this.elementData;  
                    if (offset + i >= elementData.length)  
                        throw new ConcurrentModificationException();  
                    cursor = i + 1;  
                    return (E) elementData[offset + (lastRet = i)];  
                }  
  
                public boolean hasPrevious() {  
                    return cursor != 0;  
                }  
  
                @SuppressWarnings("unchecked")  
                public E previous() {  
                    checkForComodification();  
                    int i = cursor - 1;  
                    if (i < 0)  
                        throw new NoSuchElementException();  
                    Object[] elementData = ArrayList.this.elementData;  
                    if (offset + i >= elementData.length)  
                        throw new ConcurrentModificationException();  
                    cursor = i;  
                    return (E) elementData[offset + (lastRet = i)];  
                }  
  
                @SuppressWarnings("unchecked")  
                public void forEachRemaining(Consumer<? super E> consumer) {  
                    Objects.requireNonNull(consumer);  
                    final int size = SubList.this.size;  
                    int i = cursor;  
                    if (i >= size) {  
                        return;  
                    }  
                    final Object[] elementData = ArrayList.this.elementData;  
                    if (offset + i >= elementData.length) {  
                        throw new ConcurrentModificationException();  
                    }  
                    while (i != size && modCount == expectedModCount) {  
                        consumer.accept((E) elementData[offset + (i++)]);  
                    }  
                    // update once at end of iteration to reduce heap write traffic  
                    lastRet = cursor = i;  
                    checkForComodification();  
                }  
  
                public int nextIndex() {  
                    return cursor;  
                }  
  
                public int previousIndex() {  
                    return cursor - 1;  
                }  
  
                public void remove() {  
                    if (lastRet < 0)  
                        throw new IllegalStateException();  
                    checkForComodification();  
  
                    try {  
                        SubList.this.remove(lastRet);  
                        cursor = lastRet;  
                        lastRet = -1;  
                        expectedModCount = ArrayList.this.modCount;  
                    } catch (IndexOutOfBoundsException ex) {  
                        throw new ConcurrentModificationException();  
                    }  
                }  
  
                public void set(E e) {  
                    if (lastRet < 0)  
                        throw new IllegalStateException();  
                    checkForComodification();  
  
                    try {  
                        ArrayList.this.set(offset + lastRet, e);  
                    } catch (IndexOutOfBoundsException ex) {  
                        throw new ConcurrentModificationException();  
                    }  
                }  
  
                public void add(E e) {  
                    checkForComodification();  
  
                    try {  
                        int i = cursor;  
                        SubList.this.add(i, e);  
                        cursor = i + 1;  
                        lastRet = -1;  
                        expectedModCount = ArrayList.this.modCount;  
                    } catch (IndexOutOfBoundsException ex) {  
                        throw new ConcurrentModificationException();  
                    }  
                }  
  
                final void checkForComodification() {  
                    if (expectedModCount != ArrayList.this.modCount)  
                        throw new ConcurrentModificationException();  
                }  
            };  
        }  
  
        public List<E> subList(int fromIndex, int toIndex) {  
            subListRangeCheck(fromIndex, toIndex, size);  
            return new SubList(this, offset, fromIndex, toIndex);  
        }  
  
        private void rangeCheck(int index) {  
            if (index < 0 || index >= this.size)  
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
        }  
  
        private void rangeCheckForAdd(int index) {  
            if (index < 0 || index > this.size)  
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
        }  
  
        private String outOfBoundsMsg(int index) {  
            return "Index: " + index + ", Size: " + this.size;  
        }  
  
        private void checkForComodification() {  
            if (ArrayList.this.modCount != this.modCount)  
                throw new ConcurrentModificationException();  
        }  
  
        public Spliterator<E> spliterator() {  
            checkForComodification();  
            return new ArrayListSpliterator<E>(ArrayList.this, offset,  
                    offset + this.size, this.modCount);  
        }  
    }  
  
    @Override  
    public void forEach(Consumer<? super E> action) {  
        Objects.requireNonNull(action);  
        final int expectedModCount = modCount;  
        @SuppressWarnings("unchecked") final E[] elementData = (E[]) this.elementData;  
        final int size = this.size;  
        for (int i = 0; modCount == expectedModCount && i < size; i++) {  
            action.accept(elementData[i]);  
        }  
        if (modCount != expectedModCount) {  
            throw new ConcurrentModificationException();  
        }  
    }  
  
    /**  
     * Creates a <em><a href="Spliterator.html#binding">late-binding</a></em>  
     * and <em>fail-fast</em> {@link Spliterator} over the elements in this  
     * list.     *     * <p>The {@code Spliterator} reports {@link Spliterator#SIZED},  
     * {@link Spliterator#SUBSIZED}, and {@link Spliterator#ORDERED}.  
     * Overriding implementations should document the reporting of additional     * characteristic values.     *     * @return a {@code Spliterator} over the elements in this list  
     * @since 1.8  
     */    @Override  
    public Spliterator<E> spliterator() {  
        return new ArrayListSpliterator<>(this, 0, -1, 0);  
    }  
  
    /**  
     * Index-based split-by-two, lazily initialized Spliterator     */    static final class ArrayListSpliterator<E> implements Spliterator<E> {  
  
        /*  
         * If ArrayLists were immutable, or structurally immutable (no         * adds, removes, etc), we could implement their spliterators         * with Arrays.spliterator. Instead we detect as much         * interference during traversal as practical without         * sacrificing much performance. We rely primarily on         * modCounts. These are not guaranteed to detect concurrency         * violations, and are sometimes overly conservative about         * within-thread interference, but detect enough problems to         * be worthwhile in practice. To carry this out, we (1) lazily         * initialize fence and expectedModCount until the latest         * point that we need to commit to the state we are checking         * against; thus improving precision.  (This doesn't apply to         * SubLists, that create spliterators with current non-lazy         * values).  (2) We perform only a single         * ConcurrentModificationException check at the end of forEach         * (the most performance-sensitive method). When using forEach         * (as opposed to iterators), we can normally only detect         * interference after actions, not before. Further         * CME-triggering checks apply to all other possible         * violations of assumptions for example null or too-small         * elementData array given its size(), that could only have         * occurred due to interference.  This allows the inner loop         * of forEach to run without any further checks, and         * simplifies lambda-resolution. While this does entail a         * number of checks, note that in the common case of         * list.stream().forEach(a), no checks or other computation         * occur anywhere other than inside forEach itself.  The other         * less-often-used methods cannot take advantage of most of         * these streamlinings.         */  
        private final ArrayList<E> list;  
        private int index; // current index, modified on advance/split  
        private int fence; // -1 until used; then one past last index  
        private int expectedModCount; // initialized when fence set  
  
        /**  
         * Create new spliterator covering the given  range         */        ArrayListSpliterator(ArrayList<E> list, int origin, int fence,  
                             int expectedModCount) {  
            this.list = list; // OK if null unless traversed  
            this.index = origin;  
            this.fence = fence;  
            this.expectedModCount = expectedModCount;  
        }  
  
        private int getFence() { // initialize fence to size on first use  
            int hi; // (a specialized variant appears in method forEach)  
            ArrayList<E> lst;  
            if ((hi = fence) < 0) {  
                if ((lst = list) == null)  
                    hi = fence = 0;  
                else {  
                    expectedModCount = lst.modCount;  
                    hi = fence = lst.size;  
                }  
            }  
            return hi;  
        }  
  
        public ArrayListSpliterator<E> trySplit() {  
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;  
            return (lo >= mid) ? null : // divide range in half unless too small  
                    new ArrayListSpliterator<E>(list, lo, index = mid,  
                            expectedModCount);  
        }  
  
        public boolean tryAdvance(Consumer<? super E> action) {  
            if (action == null)  
                throw new NullPointerException();  
            int hi = getFence(), i = index;  
            if (i < hi) {  
                index = i + 1;  
                @SuppressWarnings("unchecked") E e = (E) list.elementData[i];  
                action.accept(e);  
                if (list.modCount != expectedModCount)  
                    throw new ConcurrentModificationException();  
                return true;            }  
            return false;  
        }  
  
        public void forEachRemaining(Consumer<? super E> action) {  
            int i, hi, mc; // hoist accesses and checks from loop  
            ArrayList<E> lst;  
            Object[] a;  
            if (action == null)  
                throw new NullPointerException();  
            if ((lst = list) != null && (a = lst.elementData) != null) {  
                if ((hi = fence) < 0) {  
                    mc = lst.modCount;  
                    hi = lst.size;  
                } else  
                    mc = expectedModCount;  
                if ((i = index) >= 0 && (index = hi) <= a.length) {  
                    for (; i < hi; ++i) {  
                        @SuppressWarnings("unchecked") E e = (E) a[i];  
                        action.accept(e);  
                    }  
                    if (lst.modCount == mc)  
                        return;  
                }  
            }  
            throw new ConcurrentModificationException();  
        }  
  
        public long estimateSize() {  
            return (long) (getFence() - index);  
        }  
  
        public int characteristics() {  
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;  
        }  
    }  
  
    @Override  
    public boolean removeIf(Predicate<? super E> filter) {  
        Objects.requireNonNull(filter);  
        // figure out which elements are to be removed  
        // any exception thrown from the filter predicate at this stage        // will leave the collection unmodified        int removeCount = 0;  
        final BitSet removeSet = new BitSet(size);  
        final int expectedModCount = modCount;  
        final int size = this.size;  
        for (int i = 0; modCount == expectedModCount && i < size; i++) {  
            @SuppressWarnings("unchecked") final E element = (E) elementData[i];  
            if (filter.test(element)) {  
                removeSet.set(i);  
                removeCount++;  
            }  
        }  
        if (modCount != expectedModCount) {  
            throw new ConcurrentModificationException();  
        }  
  
        // shift surviving elements left over the spaces left by removed elements  
        final boolean anyToRemove = removeCount > 0;  
        if (anyToRemove) {  
            final int newSize = size - removeCount;  
            for (int i = 0, j = 0; (i < size) && (j < newSize); i++, j++) {  
                i = removeSet.nextClearBit(i);  
                elementData[j] = elementData[i];  
            }  
            for (int k = newSize; k < size; k++) {  
                elementData[k] = null;  // Let gc do its work  
            }  
            this.size = newSize;  
            if (modCount != expectedModCount) {  
                throw new ConcurrentModificationException();  
            }  
            modCount++;  
        }  
  
        return anyToRemove;  
    }  
  
    @Override  
    @SuppressWarnings("unchecked")  
    public void replaceAll(UnaryOperator<E> operator) {  
        Objects.requireNonNull(operator);  
        final int expectedModCount = modCount;  
        final int size = this.size;  
        for (int i = 0; modCount == expectedModCount && i < size; i++) {  
            elementData[i] = operator.apply((E) elementData[i]);  
        }  
        if (modCount != expectedModCount) {  
            throw new ConcurrentModificationException();  
        }  
        modCount++;  
    }  
  
    @Override  
    @SuppressWarnings("unchecked")  
    public void sort(Comparator<? super E> c) {  
        final int expectedModCount = modCount;  
        Arrays.sort((E[]) elementData, 0, size, c);  
        if (modCount != expectedModCount) {  
            throw new ConcurrentModificationException();  
        }  
        modCount++;  
    }  
}
```

# 扩容分析
