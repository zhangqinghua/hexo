---
title: Java 集合操作

categories:
- 后端开发
- Java 手册

date: 2021-05-07
---
## 排序
#### 简单类型排序
#### 对象类型排序，未实现 Comparable

```java
Collections.sort(ds, (a, b) -> a.getFinishTime().isAfter(b.getFinishTime()) ? 1 : -1);

ds.sort((a, b) -> a.getFinishTime().isAfter(b.getFinishTime()) ? 1 : -1);
```

#### 对象类型排序，已实现 Comparable
```java
ds.sort(Comparator.comparing(Store::getDistance));
```

## 转换
#### 嵌套 List 转 List
```java
List<Integer> collect = IntStream.range(1, 10).boxed().collect(Collectors.toList());
List<Integer> collect1 = IntStream.range(10, 20).boxed().collect(Collectors.toList());

List<List<Integer>> lists = new ArrayList<>();
lists.add(collect);
lists.add(collect1);
ArrayList<Integer> collect2 = lists.stream().collect(ArrayList::new, ArrayList::addAll, ArrayList::addAll);
System.out.println(collect2);
```

## 去重
#### 基本数据类型
**1. 使用两个 for 循环实现 List 去重（有序）**

```java
public static List removeDuplicationBy2For(List<Integer> list) {
   for (int i = 0; i < list.size(); i++) {
      for (int j = i + 1; j < list.size(); j++) {
            if (list.get(i).equals(list.get(j))) {
               list.remove(j);
            }
      }
   }
}
```

**2. 使用 List 集合 contains 方法循环遍历（有序）**

```java
public static List removeDuplicationByContains(List<Integer> list) {
   List<Integer> newList = new ArrayList<>();
   for (int i = 0; i < list.size(); i++) {
      boolean isContains = newList.contains(list.get(i));
      if (!isContains) {
            newList.add(list.get(i));
      }
   }
   list.clear();
   list.addAll(newList);
   return list;
}
```

**3. 使用 HashSet 实现 List 去重（无序）**

```java
public static List removeDuplicationByHashSet(List<Integer> list) {
    HashSet set = new HashSet(list);
    //把List集合所有元素清空
    list.clear();
    //把HashSet对象添加至List集合
    list.addAll(set);
    return list;
}
```

**4.使用 TreeSet 实现 List 去重（有序）**

```java
public static List removeDuplicationByTreeSet(List<Integer> list) {
    TreeSet set = new TreeSet(list);
    //把List集合所有元素清空
    list.clear();
    //把HashSet对象添加至List集合
    list.addAll(set);
    return list;
}
```

**5. 使用 Java 8 新特性 stream 实现 List 去重（有序）**

```java
public static List removeDuplicationByStream(List<Integer> list) {
    List newList = list.stream().distinct().collect(Collectors.toList());
    return newList;
}
```

**6. 效率对比**

随机数在100范围内：
1. 使用 `HashSet` 实现 `List` 去重时间：32 ms
1. 使用 `TreeSet` 实现 `List` 去重时间：40 ms
1. 使用 Java8 新特性stream实现 `List` 去重：128 ms
1. 使用两个 `for` 循环实现 `List` 去重：693 ms
1. 使用 `List` 集合 `contains` 方法循环遍历：30 ms

随机数在1000范围内：
1. 使用 `HashSet` 实现 `List` 去重时间：34 ms
1. 使用 `TreeSet` 实现 `List` 去重时间：72 ms
1. 使用 Java8 新特性 `stream` 实现 `List` 去重：125 ms
1. 使用两个 `for` 循环实现 `List` 去重：1063 ms
1. 使用 `List` 集合 `contains` 方法循环遍历：85 ms

随机数在10000范围内：
1. 使用 `HashSet` 实现 `List` 去重时间：51 ms
1. 使用 `TreeSet` 实现 `List` 去重时间：103 ms
1. 使用 Java8 新特性 `stream` 实现 `List` 去重：201 ms
1. 使用两个 `for` 循环实现 `List` 去重：5448 ms
1. 使用 `List` 集合 `contains` 方法循环遍历：791 ms


**6. 结论**

无序 `HashSet`，有序 `TreeSet`。

#### 复杂数据类型

## Map 专题
#### 创建并初始化 Map
**1. static 块初始化**

```java
private static final Map<String, String> myMap;
static {
   myMap = new HashMap<String, String>();
   myMap.put("a", "b");
   myMap.put("c", "d");
}
```

**2. 双括号初始化 （匿名内部类）**

```java
HashMap<String, String > h = new HashMap<String, String>(){{
      put("a","b");    
}};
```

**3. Guava**

```java
Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
```

> ImmutableMap 的值不能为null，否则会报 null value in entry: b=null。

#### 动态指定 Map 泛型
Map 如果没有指定泛型，则取第一个输入的类型作为泛型。

```java
public static <T> T parseObject(TypeReference<T> type) {
   Map map = new TreeMap<>();
   map.put("123", BigDecimal.valueOf(10));
   map.put(LocalDate.now(), BigDecimal.valueOf(10));

   return (T) map;
}

public static void main(String[] args) {
   TreeMap<LocalDate, BigDecimal> map = parseObject(new TypeReference<TreeMap<LocalDate, BigDecimal>>() {});

   System.out.println(map.get(LocalDate.now()));
}

===============
Exception in thread "main" java.lang.ClassCastException: java.lang.String cannot be cast to java.time.chrono.ChronoLocalDate
	at java.time.LocalDate.compareTo(LocalDate.java:137)
	at java.util.TreeMap.put(TreeMap.java:568)
	at com.easybyte.api.sys.feign.RequestFeignProxy.parseObject(RequestFeignProxy.java:279)
	at com.easybyte.api.sys.feign.RequestFeignProxy.main(RequestFeignProxy.java:285)
```

#### JSON 转成指定类型的泛型
```java
TreeMap<LocalDate, BigDecimal> map = <JSONObject.parseObject(String.valueOf(resultParam.getValue()), new TypeReference<TreeMap<LocalDate, BigDecimal>>() {});
```