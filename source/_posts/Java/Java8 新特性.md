---
title: Java8 新特性

categories:
- Java

date: 2017-10-11
---

Java8 笔记

## 集合删除

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
list.add(4);
list.removeIf(s -> s%2==0);               // 过滤掉模2等于0的数
list.forEach(s -> System.out.println(s)); // 输出 1 3

List<String> strings = new ArrayList<>();
strings.add("ab");
strings.add("ac");
strings.add("bc");
strings.add("cd");
Predicate<String> predicate = (s) -> s.startsWith("a"); // 这里单独定义了过滤器
strings.removeIf(predicate);                            // 过滤掉以"a"开头的元素
strings.forEach(s -> System.out.println(s));            // 输出 bc cd
```

## `@toString` 指定属性
为了防止相互引用造成死循环问题，在`toString`时候需要屏蔽一些属性
```java
@ToString(of = {"id", "name"})
public class Type {
	private Long id;
	private String name;
	private List<Gallery> galleries;
}
```