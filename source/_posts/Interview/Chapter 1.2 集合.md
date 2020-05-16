---
title: Chapter 1.2 集合

categories:
- Interview

date: 2020-04-28 00:00:12
---
#### Java 集合是什么
Java 集合是 Java 提供的工具包，包含了常用的数据结构：集合、链表、队列、栈、数组、映射等。

Java 集合主要可以划分为 4 个部分：**List** 列表、**Set** 集合、**Map** 映射、工具类（**Iterator** 迭代器、**Enumeration** 枚举类、**Arrays** 和 **Collections**）。

#### Java 集合工具包框架图
![](https://en.proft.me/media/java/collectionsTable.png)

- Map 
    **Map** 是一个映射接口，即键值对。**Map** 中的每一个元素包含一个 key 和它对应的值。
    **AbstractMap** 是个抽象类，它实现了 **Map** 接口中的大部分 API。而 **HashMap**，**TreeMap**，**WeakHashMap** 都是继承于 **AbstractMap**。
   **Hashtable** 虽然继承于 **Dictionary**，但它实现了 **Map** 接口。

- Collection
    **Collection** 是一个接口，是高度抽象的集合，它包含了集合的基本操作和属性。
    **Collection** 包含了 **Set** 和 **List** 两大分支：
    - **Set** 是一个不允许有重复元素的集合，**Set** 的实现类有 **HashSet** 和 **TreeSet**。**HashSet** 依赖于 **HashMap**，它实际是通过 **HashMap** 实现的；**TreeSet** 依赖于 **TreeMap**。
    - **List** 是一个有序的队列，每一个元素都有它的索引。第一个元素的索引值是 0。**List** 的实现类有 **LinkedList**、**ArrayList**、**Vector**、**Stack**。



#### HashMap 和 Hashtable 的区别
参考[文章](https://blog.csdn.net/wangxing233/article/details/79452946)。

1. 产生时间不同：**Hashtable** 是 Java 一发布时就已经提供的键值映射的数据结构，基本已经被弃用。**HashMap** 产生与 JDK 1.2，是目前应用最为广泛的一种数据类型。
1. 对外提供的接口不同：**Hashtable** 比 **HashMap** 多提供了 **elments()** 和 *contains()* 两个方法。
1. 对 Null key、value 支持不同：**Hashtable** 既不支持 Null key 也不支持 Null value。**HashMap** 中，null 可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为 null。
1. 线程安全性不同：Hashtable是线程安全的，它的每个方法中都加入了Synchronize方法。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步。HashMap不是线程安全的，在多线程并发的环境下，可能会产生死锁等问题。
1. 遍历方式的内部实现上不同
    **HashMap**、**Hashtable** 都使用了 **Iterator**。而由于历史原因，**Hashtable** 还使用了 **Enumeration** 的方式。
1. 初始容量不同
    H**ashtable** 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。
    **HashMap** 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。
    创建时，如果给定了容量初始值，那么 **Hashtable** 会直接使用你给定的大小，而 **HashMap** 会将其扩充为 2 的幂次方大小。也就是说 **Hashtable** 会尽量使用素数、奇数。而 **HashMap** 则总是使用 2 的幂作为哈希表的大小。

    之所以会有这样的不同，是因为 **Hashtable** 和 **HashMap** 设计时的侧重点不同。**Hashtable** 的侧重点是哈希的结果更加均匀，使得哈希冲突减少。当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀。而 **HashMap** 则更加关注 hash 的计算效率问题。在取模计算时，如果模数是 2 的幂，那么我们可以直接使用位运算来得到结果，效率要大大高于做除法。**HashMap** 为了加快 hash 的速度，将哈希表的大小固定为了 2 的幂。当然这引入了哈希分布不均匀的问题，所以 **HashMap** 为解决这问题，又对 hash 算法做了一些改动。这从而导致了 **Hashtable** 和 **HashMap** 的计算 hash 值的方法不同。

1. 计算 hash 值的方法不同
    为了得到元素的位置，首先需要根据元素的 key 计算出一个 hash 值，然后再用这个 hash 值来计算得到最终的位置。
    **Hashtable** 直接使用对象的 **hashCode**，这是 JDK 根据对象的地址或者字符串或者数字算出来的 **int** 类型的数值。然后再使用除留余数发来获得最终的位置。
    ```java
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    ```
    **Hashtable** 在计算元素的位置时需要进行一次除法运算，而除法运算是比较耗时的。
    **HashMap** 为了提高计算效率，将哈希表的大小固定为了 2 的幂，这样在取模预算时，不需要做除法，只需要做位运算。位运算比除法的效率要高很多。

1. 父类不同：**HashMap** 继承自 **AbstractMap**，**Hashtable** 继承自 **Dictionary**（以废弃），不过它们都实现了 **Map**、**Cloneable**、**Serializable** 这 3 个接口。 
    ![](https://img-blog.csdn.net/20180306020714182?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    ![](https://img-blog.csdn.net/20180306020658482?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ3hpbmcyMzM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 代码题：两个有序数组，数组中存在重复数字，合并成一个有序数组，去除重复数字。

## HashMap
#### 什么是 HashMap
**HashMap** 是 JDK 1.2 提供的键值映射、线程不安全的数据结构。

#### 什么是 Hashtable
**Hashtable** 是 Java 一开始就提供的键值映射、线程安全的数据结构。

#### HashMap 的实现原理


#### HashMap 中的 get() 方法是如何实现的
- 对输入的 key 的值计算 hash 值。
- 首先判断 HashMap 中的数组是否为空和数组的长度是否为 0,如果为空和为 0,则直接放回 null。
- 如果不为空和 0，计算 key 对应的数组下标，判断对应位置上的第一个 node 是否满足条件。如果满足条件，直接返回。
- 如果不满足条件，判断当前 node 是否是最后一个。如果是，说明不存在 key，则返回 null。
- 如果不是最后一个，判断是否是红黑树。如果是红黑树，则使用红黑树的方式获取对应的key。
- 如果不是红黑树，遍历链表是否有满足条件的。如果有，直接放回，否则返回null。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

/**
    * Implements Map.get and related methods
    *
    * @param hash hash for key
    * @param key the key
    * @return the node, or null if none
    */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
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

#### HashMap 可以用在哪些场景

#### HashMap及线程安全的ConcurrentHashMap，以及各自优劣势

#### 什么是 CocurrentHashMap
**ConcurrentHashMap** 是 Java 并发包中提供的一个线程安全且高效的 **HashMap** 实现。

## ArrayList
#### 什么是 ArrayList
**ArrayList** 是一个动态数组，从数据结构上来讲，是数组实现的线性表（即顺序表）。

#### ArrayList 的优点
- **ArrayList** 实现了 **Cloneable** 接口，能被克隆。
- **ArrayList** 实现了 **Serializable** 接口，支持序列化。
- **ArrayList** 实现了 **RandomAccess** 接口，支持快速随机访问。

#### ArrayList 的缺点
- 当容量不够时，每次增加元素，都要将原来的元素拷贝到一个新的数组中，非常之耗时，也因此建议在事先能确定元素数量的情况下，才使用 **ArrayList**，否则建议使用 **LinkedList**。
- **ArrayList** 不是线程安全的，只能用在单线程环境下，多线程环境下可以考虑用 **Collections.synchronizedList(List l)** 函数返回一个线程安全的 **ArrayList** 类，也可以使用 concurrent 并发包下的 **CopyOnWriteArrayList** 类。

#### ArrayList 底层原理
**ArrayList** 是基于数组实现的，是一个动态数组，其容量能自动增长。



#### ArrayList 的 3 个构造函数
- 无参构造方法构造的 ArrayList 的容量默认为 10，实现数组为空数组。
- 指定容量的构造方法创建一个指定容量的数组。
- 带有 **Collection** 参数的构造方法，将 **Collection** 转化为数组赋给 **ArrayList** 的实现数组 **elementData**。

#### ArrayList 的默认容量
默认容量为 10。

#### ArrayList 的最大容量
**ArrayList** 最大容量是 **Integer.MAX_VALUE - 8**。

因为数组对象有一个额外的元数据用于表示数组的大小。而 **Integer.MAX_VALUE** 的值为 2^31 = 2,147,483,648 = 8 bytes。

#### ArrayList 的扩容算法
- 每次在增加元素时，ArrayList 都判断容量是否足以容纳新元素。
- 当现有容量不足以容纳新元素时，就设置新的容量为就的容量的 1.5 倍 + 1，如果还是不够，则直接设置新容量为插入的参数（也就是所需的容量）。例如现有容量为 10，所需容量为 11，则新容量为 10 + 10 / 2 + 1 = 16。现在容量为 10，所需容量为 17，则新容量设置为 17。
- 根据新容量创建一个新数组，使用 **Arrays.copyof()** 方法将原有数组元素复制到新的数组。

```java
private void grow(int miniCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - miniCapacity < 0)
        newCapacity = miniCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(miniCapacity);

    // miniCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

#### ArrayList 的优化要点
- 如果我们大概知道元素的个数，可以指定个数的方式去构造 **ArrayList**，这样可以避免底层数组的多次拷贝，进而提高程序性能。
- 

#### 讲一下 Arrays.copyof() 方法
该方法实际上是在其内部又创建了一个长度为 **newlength** 的数组，调用 **System.arraycopy()** 方法，将原来数组中的元素复制到了新的数组中。

```java
public static <T> T[] copyOf(T[] original, int newLength) {  
    return (T[]) copyOf(original, newLength, original.getClass());  
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    T[] copy = ((Object)newType == (Object)Object[].class)
                ? (T[]) new Object[newLength]
                : (T[]) Array.newInstance(newType.getComponentType(), newLength);

    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```
#### 讲一下 System.arraycopy() 方法
该方法被标记了 **native**，调用了系统的 C/C++ 代码。该函数实际上最终调用了 C 语言的 **memmove()** 函数，因此它可以保证同一个数组内元素的正确复制和移动，比一般的复制方法的实现效率要高很多，很适合用来批量处理数组。Java 强烈推荐在复制大量数组元素时用该方法，以取得更高的效率。

```java
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

#### 什么是 fail-fast 机制

## LinkedList
#### LinkedList 简介
**LinkedList** 是一个继承于 **AbstractSequentialList** 的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
**LinkedList** 实现了 **List** 接口，能对它进行队列操作。
**LinkedList** 实现了 **Deque** 接口，即能将 **LinkedList** 当作双端队列使用。
**LinkedList** 实现了 **Cloneable** 接口，即覆盖了函数 **clone()**，能克隆。
**LinkedList** 实现了 **Serializable** 接口，这意味着 **LinkedList** 支持序列化，能通过序列化去传输。
**LinkedList** 是非同步的。

#### LinkedList 底层原理

#### LinkedList 的数据结构
**LinkedList** 的本质是双向链表。

- **LinkedList** 继承于 **AbstractSequentialList**，并且实现了 **Dequeue** 接口。
- **LinkedList** 包含两个重要的成员：**header** 和 **size**。
    - **header** 是双向链表的表头，它是双向链表节点所对应的类 **Entry** 的实例。**Entry** 中包含成员变量： **previous**, **next**, **element**。其中，**previous** 是该节点的上一个节点，**next** 是该节点的下一个节点，**element** 是该节点所包含的值。
    - **size** 是双向链表中节点的个数。

![](https://images0.cnblogs.com/blog/497634/201401/272345393446232.jpg)

#### LinkedList 的源码解析

#### LinkedList 的遍历方式

## 迭代器
#### Iterable 是什么
**Iterable** 是一个接口，有一个抽象方法叫做 **iterator()**，返回结果是一个 **Iterator** 对象（一般是在集合中定义了 **Itr** 内部类，，该类实现了 **Iterator** 接口）

#### Iterator 是什么
**Iterator** 是一个接口，用于对集合（**Collection**）进行遍历。

#### 可以使用 Iterator 遍历的本质是什么？
实现 **Iterable** 接口。

#### 哪些集合可以用迭代器进行遍历？
实现了 **Collection** 接口的集合都可以使用迭代器进行遍历，包括 **Set**、**List**、**Queue**。

**Map** 因为没有实现 **Iterable** 接口，所以不能使用迭代器进行遍历，但是 **Map** 可以使用 **entrySet()** 方法将 key 转成 **Set** 集合从而使用迭代器。

#### foreach 增强循环为什么 JDK 1.5 后才能使用？
**foreach** 循环底层使用了迭代器的方式，迭代器 1.5 版本才提供。

#### foreach 和迭代器 Iterator 的区别？
**foreach** 可以遍历数组和集合，迭代器只能遍历集合，不能遍历数组。

#### foreach 有什么缺陷？
- 不能方便访问数组的下标
- 不能在 **foreach** 循环中尝试对变量进行赋值，只是一个临时变量
- 与使用 **Iterator** 相比，不能方便的删除元素

#### 迭代器提供了哪些方法？
- **hasNext** 用于判断是否还有下一个元素，同一个迭代器只能使用一次，使用一次之后，指针已经指向末尾，再次调用 **hasNext** 方法时，返回 **false**。
- **next** 用于获取下一个元素，并将指针下移一位。
- **remove** 用于删除元素，每次调用 **next** 后只能调用一次 **remove** 方法。

#### 什么是 ListIterator？
在迭代器迭代集合的过程中，不允许对集合进行修改，否则会抛出异常（并发修改）。

**ListIterator** 允许程序员按照任一方向遍历集合，并且可以在迭代期间修改集合并获取迭代器在集合中的位置。

通过 **List.listIterator()** 方法获取一个 **ListIterator**。

#### Iterator 和 ListIterator 的联系和区别？
**Iterator** 是 **ListIterator** 的父接口。

- **Iterator** 给所有的 **Collection** 使用，而 **ListIterator** 指给 **List** 及其实现类使用。
- **Iterator** 只提供了 3 个方法：**hasNext**、**next** 和 **remove**。**ListIterator** 提供了其它很多方法，功能较多。
- **Iterator** 只能对集合进行正序遍历，而 **ListIterator** 提供了正序和逆序遍历。
- **Iterator** 只能实现删除元素的功能，在修改元素时可能会触发并发修改异常。**ListIterator** 支持在遍历时进行增删改的操作。
- **Iterator** 不能获取元素的索引，而 **ListIterator** 可以获取元素的前后索引。

## Collections
#### Collections 是什么？
**Collections** 是 Java 提供的一个工具类，用于程序员对 **Collection** 进行简化操作。

#### Collections 有什么方法？
- **void addAll(Collection c, T... elements)**
    一次性向集中添加若干元素。
    ```java
    List<String> list = new ArrayList<>();
    Collections.addAll(list, "a", "b", "c");
    ```
- **void sort(List list)**
    自然排序，默认使用内部比较器进行排序。
    ```java
    List<String> list = new ArrayList<>();
    Collections.addAll(list, "c", "b", "a");

    Collections.sort(list);

    // 自定义排序
    Collections.sort(list, (a, b) -> a.lenght > b.lenght);
    ```