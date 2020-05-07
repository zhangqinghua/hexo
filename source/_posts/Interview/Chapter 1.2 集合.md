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
#### ArrayList 底层原理

#### LinkedList 底层原理

