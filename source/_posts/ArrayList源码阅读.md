---
title: ArrayList源码阅读
date: 2019-04-02 21:22:51
categories: "JavaSE"
tags:
    - JavaSE
    - 源码阅读
copyright:
---

![](/images/JavaSE_ArrayList.png)

### 简介
- **数据结构**：基于数组，能够动态到地增加或减小其大小，当也可以调用ensureCapacity方法来进行人工地增加ArrayList的容量，从而避免再分配的消耗时间，前提是事先知道需要存储很多内容。
- **继承关系**：继承自AbstractList，实现了List、RandomAccess、Cloneable、Serializable接口。其类图如下：
![](/images/JavaSE_ArrayList_UML.png)

- **空值**：允许 null 的存在。
- **线程安全性**：线程不安全。
---

### 相关域
```java
private static final long serialVersionUID = 8683452581122892189L;

private static final int DEFAULT_CAPACITY = 10;

private static final Object[] EMPTY_ELEMENTDATA = {};

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

transient Object[] elementData; 

private int size;
```
- **serialVersionUID**: 序列号
- **DEFAULT_CAPACITY**: 默认初始化容量，默认为10
- **EMPTY_ELEMENTDATA**: 空数组，当用户指定ArrayList的容量为0时，返回该空数组
- **DEFAULTCAPACITY_EMPTY_ELEMENTDATA**: 空数组，当用户没有指定ArrayList的容量时，elementData将会引用此数组
- **elementData**: 用户存储数据的数组
- **size**: 当前ArrayList实际存储的数量数量

---

### 构造器
- **ArrayList(int initialCapacity)**：构造一个指定初始容量的ArrayList

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;                  
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}
```
当初始化容量等于0时，返回EMPTY_ELEMENTDATA

- **ArrayList()**：构造一个默认容量的ArrayList

```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```
创建一个空的 ArrayList，此时其内数组elementData = {}, 长度为 0, 当元素第一次被加入时，扩容至默认容量 10

- **ArrayList(Collection<? extends E> c)**：构造一个包含collection的ArrayList，传入一个collection，其内元素将会全部添加到新建的ArrayList实例中


```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)        //1 
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);       //2
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;                                     //3
    }
}
```
1. c.toArray可能不会返回 Object[]，可以查看 java 官方编号为 6260652 的 bug
2. 若 c.toArray() 返回的数组类型不是 Object[]，则利用 Arrays.copyOf(); 来构造一个大小为 size 的 Object[] 数组
3. 若collection长度为0，则替换为空数组

---

### 自动扩容机制
- 因为扩容发生在添加元素时，首先看一下add()方法

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  
    elementData[size++] = e;
    return true;
}
```
这里调用了ensureCapacityInternal(size + 1),size + 1保证资源空间不被浪费，按当前情况，保证要存多少个元素，就只分配多少空间资源，这里后续会修改modCount的值，

- 看下ensureCapacityInternal方法

```java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```
这里计算了ArrayList当前合理的容量（即size+1)，

- 我们看下计算方式：calculateCapacity

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {   
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {             //1
        return Math.max(DEFAULT_CAPACITY, minCapacity);                 //2
    }
    return minCapacity;
}
```
1. 一个私有静态方法，当用户没有指定ArrayList的容量时，构造器会将DEFAULTCAPACITY_EMPTY_ELEMENTDATA赋给elementData；
2. 当前合理的容量计算方式如下：
如果当前的数组缓冲区是DEFAULTCAPACITY_EMPTY_ELEMENTDATA，说明当前容量为默认初始容量，ArrayList在以默认初始化容量构造之后还没有进行扩容过，
那么如果当前所需的容量小于默认初始容量时，当前所需的最小容量依然不变，为默认初始容量
如果当前所需的容量大于默认初始容量时，当前所需的容量变为minCapacity（size+1）。

- 接着看下ensureExplicitCapacity：

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;                                             //1
    if (minCapacity - elementData.length > 0)               //2
        grow(minCapacity);                                  //3
}

```
1. 这里修改了modCount的值，至于modCount的作用，详见[深入ArrayList看fast-fail机制](./JavaSE/深入ArrayList看fast-fail机制.md)
2. 防止溢出代码：确保指定的最小容量大于数组缓冲区当前的长度
3. 当当前合理的容量大于数组缓冲区的长度时，才真正调用grow(minCapacity)进行扩容

- grow(int minCapacity)：真正进行扩容的操作

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;            //1

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);                     //2
    if (newCapacity - minCapacity < 0)                                      //3
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)                                   //4
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
1. 数组缓冲区最大存储容量，在一些 VM 会在一个数组中存储某些数据，所以需要减去8；
2. 计算新的容量newCapacity：扩充当前容量的1.5倍；
3. 如果newCapacity 依旧小于 minCapacity ，那么新的容量就为minCapacity；
4. 若 newCapacity 大于最大存储容量，则进行大容量分配。

- 大容量分配方法：hugeCapacity(int minCapacity),最大分配 Integer.MAX_VALUE

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```

综上，ArrayList的整个扩容流程如下：
1. 调用add方法时，在添加元素之前，需要确保当前容量足够，按照当前所需最小容量（size+1，下面用minCapacity代替）调用ensureCapacityInternal确保容量足够，在这个过程中可能会触发扩容。
2. ensureCapacityInternal先计算当前合理的容量，计算方法如下：如果当前的数组缓冲区是DEFAULTCAPACITY_EMPTY_ELEMENTDATA，说明当前容量为默认初始容量，ArrayList在以默认初始化容量构造之后还没有进行扩容过，那么如果当前所需的容量小于默认初始容量时，当前合理的容量依然不变，为默认初始容量，如果当前合理的容量大于默认初始容量时，当前合理的容量变为minCapacity（size+1）。
3. 得到合理的所需容量之后，就调用ensureExplicitCapacity判断是否需要扩容，判断方式如下：当当前合理的容量大于数组缓冲区的长度时，才真正调用grow(minCapacity)进行扩容
4. 真正扩容的过程是grow方法，流程如下：计算新的容量newCapacity：当前容量的1.5倍；如果newCapacity 依旧小于 minCapacity ，那么新的容量就为minCapacity；如果 newCapacity 大于最大存储容量，则进行大容量分配。确定好新数组的容量之后，调用Arrays.copyOf(elementData, newCapacity)复制数组；
5. 到了这里才算是确保容量足够，可以添加元素了，执行添加元素的操作

### 常见操作
#### 添加
- **add(int index, E element)**：在指定位置插入新元素，原先在 index 位置的值往后移动一位

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);            //1

    ensureCapacityInternal(size + 1);   //2

    System.arraycopy(elementData, index, elementData, index + 1,
                        size - index);  //3
    elementData[index] = element;       
    size++;
}
```

1. 检查下标是否越界:

```java
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
2. 确保空间足够，又不浪费资源  
3. 参数说明：第一个是要复制的数组，第二个是从要复制的数组的第几个开始，第三个是复制到那，四个是复制到的数组第几个开始，最后一个是复制长度。



- **addAll(Collection<? extends E> c)**：将一个集合的所有元素顺序添加（追加）到 lits 末尾

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;                  //1
    ensureCapacityInternal(size + numNew);  //2 
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

1. 要添加的个数
2. 扩容

- **addAll(int index, Collection<? extends E> c)**:在指定下标之后插入集合中的元素

```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();               //1
    int numNew = a.length;              
    ensureCapacityInternal(size + numNew);  

    int numMoved = size - index;            //2
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                            numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

1. 将集合中的元素转为数组  
2. 要移动的数量

#### 删除

- **remove(int index)**: 删除指定下标的元素

```java
 public E remove(int index) {
    rangeCheck(index);                      //1

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;        //2
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
    elementData[--size] = null;             //3

    return oldValue;                        //4
}
```
1. 检查下标是否越界
2. 要移动的长度
3. 将最后一个元素置空
4. 返回删除的元素

- **remove(Object o)**: 删除指定第一个的元素，即符合条件的索引最低值。如果list中不包含这个元素，这个list不会改变，如果包含，index之后的元素都左移一位。

```java
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);          //1
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

1. 快删除，跳过检查，不返回被删除的值

```java
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

- **removeAll(Collection<?> c)**: 移除list中指定集合包含的所有元素

```java
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);      //1
    return batchRemove(c, false);   //2
}
```

1. 判断集合是否为空，为空报NPE；
2. 批量删除c集合的元素,第二个参数是否采补集,如果true：移除list中除了c集合中的所有元素,如果false：移除list中c集合中的元素

```java
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)               //1
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {                                                //2
            System.arraycopy(elementData, r,
                                elementData, w,
                                size - r);
            w += size - r;
        }
        if (w != size) {                                                //3
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
```

1. 判断元素是否保留，complement是否采补集,如果true：移除list中除了c集合中的所有元素,如果false：移除list中c集合中的元素
2. 最后当r!=size表示可能出错了，将r后面的元素都复制到w之后
3. 如果w == size，表示全部元素都保留了，所以也就没有删除操作发生，所以会返回false；反之，返回true，并更改数组； w!=size; 即使try抛出异常，也能正常处理异常抛出前的操作，因为w始终为要保留的前半部分，数组也不会因此乱序。

- **removeIf(Predicate<? super E> filter)**：根据Predicate条件来移除元素

```java
public boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    // figure out which elements are to be removed
    // any exception thrown from the filter predicate at this stage
    // will leave the collection unmodified
    int removeCount = 0;
    final BitSet removeSet = new BitSet(size);
    final int expectedModCount = modCount;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        @SuppressWarnings("unchecked")
        final E element = (E) elementData[i];
        if (filter.test(element)) {                         //1
            removeSet.set(i);
            removeCount++;
        }
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();        //2
    }

    // shift surviving elements left over the spaces left by removed elements
    final boolean anyToRemove = removeCount > 0;            //3
    if (anyToRemove) {
        final int newSize = size - removeCount;
        for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {     
            i = removeSet.nextClearBit(i);
            elementData[j] = elementData[i];
        }
        for (int k=newSize; k < size; k++) {                            //4
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
```
1. 判断是否满足条件， 如果满足就将对应的下标存放到bitset中，并增加应移除元素的数量
2. 检查遍历过程中是否被外部改变
3. 如果有要移除的元素，则获取bitset中置位0的下标，并将元素移到新位置
4. 将剩余空间置空

#### 查询
- **get(int index)**: 获取指定位置上的元素，下标从0开始

```java
public E get(int index) {
    rangeCheck(index);              //1
    return elementData(index);      //2
}
```
1. 检查数组是否越界，代码如下：

```java
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

2. 返回数组中改位置的元素：

```java
E elementData(int index) {
    return (E) elementData[index];
}
```

- **contains(Object o)**：判断ArrayList是否包含某元素

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
```

- **size()**: 返回ArrayList实际存储的元素数量

```java
public int size() {
    return size;
}
```

- **isEmpty()**：判断ArrayList是否为空

```java
public boolean isEmpty() {
    return size == 0;
}
```


- **indexOf(Object o)**：获取元素的下标

```java
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

- **lastIndexOf(Object o)**: 逆序查找，返回元素的最低索引值(最首先出现的索引位置)，其实就是最后一个出现的位置

```java
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

#### 修改

- **set(int index, E element)**: 将index位置的元素设置为element

```java
public E set(int index, E element) {
    rangeCheck(index);                  //1

    E oldValue = elementData(index);    //2
    elementData[index] = element;
    return oldValue;                    //3
}
```

1. 检查下标是否越界
2. 获取元素旧值
3. 返回旧值


- **clear()**：将ArrayList清空

```java
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```

#### 其他
- **trimToSize()**: 将数组缓冲区大小调整到实际 ArrayList 存储元素的大小，该方法由用户手动调用，以减少空间资源浪费的目的。

```java
public void trimToSize() {
    modCount++;                         //1
    if (size < elementData.length) {
        elementData = (size == 0)       //2
                ? EMPTY_ELEMENTDATA
                : Arrays.copyOf(elementData, size);
    }
}
```

当实际大小 < 数组缓冲区大小时,比如调用默认构造函数后，刚添加一个元素，此时 elementData.length = 10，而 size = 1，通过这一步，可以使得空间得到有效利用，而不会出现资源浪费的情况。



- **clone()**: 深拷贝，克隆一个ArrayList实例,对拷贝出来的ArrayList对象的操作不会对原来的ArrayList造成影响

```java
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();                  //1
        v.elementData = Arrays.copyOf(elementData, size);            //2
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```
    
1. Object 的克隆方法：会复制本对象及其内所有基本类型成员和 String 类型成员，但不会复制对象成员、引用对象;
2. 对需要进行复制的引用变量，进行独立的拷贝：将存储的元素移入新的 ArrayList 中

- **toArray()**: 返回 ArrayList 的 Object 数组

```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

- **toArray(T[] a)**:返回ArrayList 元素组成的数组,存储早a数组中

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());                    //1
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

1. 如果数组a的大小 < ArrayList的元素个数,则新建一个T[]数组。