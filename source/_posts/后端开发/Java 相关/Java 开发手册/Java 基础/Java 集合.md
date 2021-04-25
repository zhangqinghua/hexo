---
title: Java 集合

categories:
- 后端开发
- Java 基础

data: 2020-11-05 00:00:59
---
集合框架 `Collection` 存放于 `java.util` 包中，是一个用来存放对象的容器。

集合框架被设计成要满足以下几个目标：
1. 该框架必须是高性能的。基本集合（动态数组，链表，树，哈希表）的实现也必须是高效的。
1. 该框架允许不同类型的集合，以类似的方式工作，具有高度的互操作性。
1. 对一个集合的扩展和适应必须是简单的。

为此，整个集合框架就围绕一组标准接口而设计。你可以直接使用这些接口的标准实现，诸如： `LinkedList`, `HashSet`, 和 `TreeSet` 等,除此之外你也可以通过这些接口实现自己的集合。

Java 集合框架主要包括两种类型的容器，一种是集合（`Collection`），存储一个元素集合，另一种是图（`Map`），存储键/值对映射。`Collection` 接口又有 3 种子类型，`List`、`Set` 和 `Queue`，再下面是一些抽象类，最后是具体实现类，常用的有 `ArrayList`、`LinkedList`、`HashSet`、`LinkedHashSet`、`HashMap`、`LinkedHashMap` 等等。

![](https://www.runoob.com/wp-content/uploads/2014/01/2243690-9cd9c896e0d512ed.gif)

集合框架是一个用来代表和操纵集合的统一架构。所有的集合框架都包含如下内容：
1. 接口：是代表集合的抽象数据类型。例如 Collection、List、Set、Map 等。之所以定义多个接口，是为了以不同的方式操作集合对象。
1. 实现：是集合接口的具体实现。从本质上讲，它们是可重复使用的数据结构，例如：`ArrayList`、`LinkedList`、`HashSet`、`HashMap`。
1. 算法：是实现集合接口的对象里的方法执行的一些有用的计算，例如：搜索和排序。这些算法被称为多态，那是因为相同的方法可以在相似的接口上有着不同的实现。

![](https://www.runoob.com/wp-content/uploads/2014/01/java-coll.png)

## Map
`HashMap.get()` 实现过程：

#### HashMap
1. 实现原理
1. get 实现
1. 如何保证线程安全
1. 链表，红黑树
1. Hash 冲突

#### ConcurrentHashMap
1. HashMap vs ConcurrentHashMap
1. 如何解决并发问题
1. 如何保证线程安全、并发度大小
1. jdk1.8变化
1. 为什么底层需要红黑树

## List
1. ArrayList 原理
1. LinkedList 原理
1. ArrayList vs LinkedList

#### Set 和 List 的区别
1. `Set` 接口实例存储的是无序的，不重复的数据。`List` 接口实例存储的是有序的，可以重复的元素。

2. `Set` 检索效率低下，删除和插入效率高，插入和删除不会引起元素位置改变 （实现类有 `HashSet`、`TreeSet`）。

3. `List` 和数组类似，可以动态增长，根据实际存储的数据的长度自动增长 `List` 的长度。查找元素效率高，插入删除效率低，因为会引起其他元素位置改变（实现类有 `ArrayList`、`LinkedList`、`Vector`）。

## Collections
`java.util.Collections`，是不属于 Java 的集合框架的，它是集合类的一个工具类/帮助类。此类不能被实例化，服务于 Java 的 `Collection` 框架。

它包含有关集合操作的静态多态方法，实现对各种集合的搜索、排序、线程安全等操作。

#### 排序
使用 `sort` 方法可以根据元素的自然顺序对指定列表按升序进行排序。列表中的所有元素都必须实现 `Comparable` 接口， 而且必须是使用指定比较器可相互比较的。

```java
public class Test {
    public static void main(String[] args) {
        List list = Arrays.asList("one two three four five".split(" "));
        Collections.sort(list);
        System.out.println(list);
    }
}

// [five, four, one, three, two]   //（按字母排序）
```

`Collections.sort()` 使用原理：
1. 底层调用了 `Arrays.sort()`。


`Timsort` 算法的过程包括：？？？？？？？？？？？
1. 

```java
// 1. Collections.sort 调用了 Arrays.sort
public default void sort(Comparator<? super E> c) {
   Object[] a = this.toArray();
   Arrays.sort(a, (Comparator) c);
   ListIterator<E> i = this.listIterator();
   for (Object e : a) {
      i.next();
      i.set((E) e);
   }
}

// 2. Arrays.sort
public static <T> void sort(T[] a, Comparator<? super T> c) {
   if (c == null) {
      sort(a);
   } else {
      if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
      else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
   }
}
```

#### 混排
使用 `shuffle` 可以混排集合中元素的顺序。这个算法对实现一个碰运气的游戏非常有用，在生成测试案例时也十分有用。

```java
public class Test {
    public static void main(String[] args) {
        List list = Arrays.asList("one two three four five".split(" "));
        Collections.shuffle(list);
        System.out.println(list);
    }
}

// [three, five, four, one, two]
```

#### 反转
使用 `reverse` 可以反转集合中元素的顺序。

```java
public class Test {
    public static void main(String[] args) {
        List list = Arrays.asList("one two three four five".split(" "));
        Collections.reverse(list);
        System.out.println(list);
    }
}

// [five, four, three, two, one]
```

#### 交换
`swap(List list, int m, int n)` 用于交换集合中指定元素索引 `m`, `n` 的位置。

```java
public class Test {
    public static void main(String[] args) {
        List list = Arrays.asList("one two three four five".split(" "));
        Collections.swap(list, 2, 3);
        System.out.println(list);
    }
}

// [one, two, four, three, five]
```

#### 替换
使用 `fill` 可以替换集合中的所有元素。

```java
public class Test {
    public static void main(String[] args) {
        List list = Arrays.asList("one two three four five".split(" "));
        Collections.fill(list, "zero");
        System.out.println(list);
    }
}

// [zero, zero, zero, zero, zero]
```

#### 拷贝 
使用 `copy` 将源集合中的元素全部复制到目标中，并且覆盖相应索引的元素。目标集合至少与源集合一样长。

```java
public class Test {
    public static void main(String[] args) {
        List list1 = Arrays.asList("one two three four five".split(" "));
        List list2 = Arrays.asList("一 二 三 四 五".split(" "));
        Collections.copy(list1, list2);
        System.out.println(list1);
    }
}
// [一, 二, 三, 四, 五]
```

#### 位移
使用 `rotate(List list, int m)` 根据指定的距离 `m` 循环移动列表中的元素。集合中的元素向后移 `m` 个位置，在后面被遮盖的元素循环到前面来。

```java
public class Test {
    public static void main(String[] args) {
        List list1 = Arrays.asList("one two three four five".split(" "));
        Collections.rotate(list1, 2);
        System.out.println(list1);
    }
}

// [four, five, one, two, three]
```

#### 最大最小
`min(Collection)`、` min(Collection, Comparator)`、`max(Collection)` 和 `max(Collection, Comparator)` 可以根据指定比较器产生的顺序，返回给定 `Collection` 的最小（大）元素。

```java
public class Test {
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add(10);
        list.add(40);
        list.add(20);
        list.add(50);
        System.out.println(Collections.min(list));
        System.out.println(Collections.max(list));
    }
}

// 10     
// 50
```

## 索引位置
`indexOfSublist(List list, List sublist)` 用于查找 `sublist` 在 `list` 中首次出现位置的索引。返回指定源列表中第一次出现指定目标列表的起始位置。

`lastIndexOfSublist(List list, List sublist)` 用于返回指定列表中最后一次出现指定目标列表的起始位置。

```java
public class Test {
    public static void main(String[] args) {
        List list = Arrays.asList("one two three four five".split(" "));
        List subList=Arrays.asList("three four five".split(" "));
        System.out.println(Collections.indexOfSubList(list,subList));

    }
}

// 2
```
