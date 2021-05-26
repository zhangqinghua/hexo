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