# ArrayList

`ArrayList` 的底层是数组队列，相当于动态数组。与 $Java$ 中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用 `ensureCapacity` 操作来增加 `ArrayList` 实例的容量。这可以减少递增式再分配的数量。

```java
public void ensureCapacity(int minCapacity) {
    if (minCapacity > elementData.length
        && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
            && minCapacity <= DEFAULT_CAPACITY)) {
        modCount++;
        grow(minCapacity);
    }
}
```

`ArrayList` 继承于 `AbstractList` ， 实现了 `List` ， `RandomAccess` ， `Cloneable` ， `java.io.Serializable` 这些接口。

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable 
{

}
```

- `List` ：列表，支持添加、删除、查找等操作，并且可以通过下标进行访问。
- `RandomAccess` ：这是一个标识接口，表明这个接口的 `List` 集合支持**快速随机访问**。在 `ArrayList` 中，我们即可以通过元素的序号快速获取元素对象。
- `Cloneable` ：表明它具有拷贝能力，可以进行深拷贝或浅拷贝操作。
- `Serializable` ：表明它可以进行序列化操作，也就是可以将对象转换为字节流进行持久化存储或网络传输。

![alt text](image.png)

## 构造函数

`ArrayList` 有三种方式来初始化，源码如下：
```java
/**
 * 默认初始容量大小
 */
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 默认构造函数，使用初始容量10构造一个空列表(无参数构造)
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 带初始容量参数的构造函数。（用户自己指定容量）
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {//初始容量大于0
        //创建initialCapacity大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {//初始容量等于0
        //创建空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {//初始容量小于0，抛出异常
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}


/**
 *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
 *如果指定的集合为null，throws NullPointerException。
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        // toArray方法可能返回非 Object[] 类型的数组
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

## ArrayList 扩容

以下以使用无参构造函数创建的 `ArrayList` 为例。

```java

// 实际元素个数
private int size;

/**
* 将指定的元素追加到此列表的末尾。
*/
public boolean add(E e) { 
    // 加元素之前，先调用ensureCapacityInternal方法 
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 这里看到ArrayList添加元素的实质就相当于为数组赋值
    elementData[size++] = e;
    return true;
}
```

> $JDK11$ 移除了 `ensureCapacityInternal()` 和 `ensureExplicitCapacity()` 方法

```java
// 根据给定的最小容量和当前数组元素来计算所需容量。
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果当前数组元素为空数组（初始情况），返回默认容量和最小容量中的较大值作为所需容量
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    // 否则直接返回最小容量
    return minCapacity;
}

//判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    //判断当前数组容量是否足以存储minCapacity个元素
    if (minCapacity - elementData.length > 0)
        //调用grow方法进行扩容
        grow(minCapacity);
}

// 确保内部容量达到指定的最小容量。
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

- 当我们要 `add` 进第 $1$ 个元素到 `ArrayList` 时， `elementData.length` 为 $0$ （因为还是一个空的 $list$ ），因为执行了 `ensureCapacityInternal()` 方法， 所以 `minCapacity` 此时为 $10$ 。此时， `minCapacity - elementData.length > 0` 成立，所以会进入 `grow(minCapacity)` 方法。
- 当 `add` 第 $2$ 个元素时， `minCapacity` 为 $2$ ，此时 `elementData.length` （容量）在添加第一个元素后扩容到 $10$ 了。此时 `minCapacity - elementData.length > 0` 不成立，所以不会进入（执行 `grow(minCapacity)` ）方法。
- 添加 $3$、$4$ ... 到第 $10$ 个元素时，依然不会执行 `grow` 方法， 数组容量都为 $10$ 。

```java
public static final int SOFT_MAX_ARRAY_LENGTH = Integer.MAX_VALUE - 8;

public static int newLength(int oldLength, int minGrowth, int prefGrowth) {
    // preconditions not checked because of inlining
    // assert oldLength >= 0
    // assert minGrowth > 0

    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // 新容量
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {
        return prefLength;
    } else {
        // put code cold in a separate method
        return hugeLength(oldLength, minGrowth);
    }
}

private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
            minCapacity - oldCapacity, /* 至少增加容量到minCapacity */
            oldCapacity >> 1           /* 默认增长原容量的 1.5 倍 */);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
```
`int newCapacity = oldCapacity + (oldCapacity >> 1)` ，所以 $ArrayList$ 每次扩容之后容量都会变为原来的 $1.5$ 倍左右。

```java
private static int hugeLength(int oldLength, int minGrowth) {
    int minLength = oldLength + minGrowth;
    if (minLength < 0) { // overflow
        throw new OutOfMemoryError(
            "Required array length " + oldLength + " + " + minGrowth + " is too large");
    } else if (minLength <= SOFT_MAX_ARRAY_LENGTH) { 
        // 无法增长到1.5倍 但是可以扩容到minCapacity
        return SOFT_MAX_ARRAY_LENGTH;
    } else { // SOFT_MAX_ARRAY_LENGTH 只是一个软限制
        return minLength;
    }
}
```

从上面的 `grow()`方法源码我们知道：如果新容量 `prefLength` 大于 `SOFT_MAX_ARRAY_LENGTH` ，进入（执行） `hugeCapacity()` 方法来比较 `minCapacity` 和 `SOFT_MAX_ARRAY_LENGTH` ，如果 `minCapacity` 大于最大容量，则新容量则为 `minCapacity`, 否则新容量为 `SOFT_MAX_ARRAY_LENGTH`。

```java
@IntrinsicCandidate
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) 
{
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                        Math.min(original.length, newLength));
    return copy;
}

public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}
```

`Arrays.copyOf()` 方法主要是为了给原有数组扩容。

```java
@IntrinsicCandidate
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

可以看出，`copyOf()` 方法内部实际调用了 `System.arraycopy()` 方法。这二者的区别在于， `arraycopy()` 需要目标数组，将原数组拷贝到你所定义的数组里， 或原数组，而且可以选择拷贝的起点 `destPos` 和长度 `length` ，以及放入新数组中的位置`srcPos` 。 `copyOf()` 是系统自动在内部新建一个数组，并返回该数组。实际作用就是扩容原数组（或者说是返回扩容后的原数组）。

```java
public void ensureCapacity(int minCapacity) {
    if (minCapacity > elementData.length
        && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
                && minCapacity <= DEFAULT_CAPACITY)) {
        modCount++;
        grow(minCapacity);
    }
}
```

该方法留给用户自己调用，目的是在可能会加入大量元素之前使用该方法，来申请足够大的空间，减少自身扩容函数的调用，减少增量重新分配的次数，节约时间。



## ArrayList / Vector

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]` 存储，适用于频繁的查找工作，线程不安全。
- `Vector` 是 `List` 的古老实现类，底层使用 `Obejct[]` 存储，线程安全。

## ArrayList / LinkedList

- 是否保证线程安全： `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
- 底层数据结构： `ArrayList` 底层使用的是 `Object` 数组； `LinkedList`底层使用的是**双向链表**数据结构（ $JDK1.6$ 之前为循环链表， $JDK1.7$ 取消了循环。）
- 插入和删除是否受元素位置的影响：
    - `ArrayList`采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。比如：执行 `add(E e)` 方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 $O(1)$ ，但是如果要在指定位置 $i$ 插入和删除元素的话（ `add(int index, E element)` ），时间复杂度就为 $O(n)$ ，因为在进行上述操作的时候集合中第 $i$ 和第 $i$ 个元素之后的 $(n-i)$ 个元素都要执行向后位/向前移一位的操作。
    - `LinkedList`采用链表存储，所以在头尾插入或者删除元素不受元素位置的影响（ `add(E e)` 、 `addFirst(E e)` 、 `addLast(E e)` 、 `removeFirst()` 、 `removeLast()` ），时间复杂度为 $O(1)$ ，如果是要在指定位置 $i$ 插入和删除元素的话（ `add(int index, E element)` ， `remove(Object o)` ， `remove(int index)` ），时间复杂度为 $O(n)$ ，因为需要先移动到指定位置再插入和删除。
- 是否支持快速随机访问： `LinkedList`不支持高效的随机元素访问，而 `ArrayList` （实现了 `RandomAccess` 接口）支持。
- 内存空间占用： `ArrayList` 的空间浪费主要体现在 $list$ 列表的结尾会预留一定的容量空间，而 $LinkedList$ 的空间花费则体现在它的每一个元素都需要消耗比 $ArrayList$ 更多的空间（因为要存放的直接后继和直接前驱以及数据）。
