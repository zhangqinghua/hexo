---
title: Java 正则表达式

categories:
- Regular

date: 2018-06-27 20:42:03
---

Java 正则表达式主要包含 3 个类 String、Pattern、Matcher。

## String 类

`str.matches(regex)` 表示字符串是否在给定的正则表达式匹配，跟 `Pattern.matches(regex, str)` 结果一样。

```java
String str = new String("Welcome to Tutorialspoint.com");

// 匹配多个任意字符 + Tutorials + 多个任意字符
System.out.println(str.matches("(.*)Tutorials(.*)"));

// 匹配 Tutorials
System.out.println(str.matches("Tutorials"));

// 匹配 Welcome + 多个任意字符
System.out.println(str.matches("Welcome(.*)"));

// true
// false
// true
```

`str.replaceAll` 则是基于正则表达式的替换。

```java
// 替换所有数字
System.out.println("a8729a".replaceAll("\\d", "-")); 

// a----a
 ```

## Pattern 类

Pattern 用于创建一个正则表达式，也可以说创建一个匹配模式。

Pattern 的构造方法是私有的，不可以直接创建，但可以通过 `Pattern.complie(String regex)` 简单工厂方法创建。

Pattern 相较 `str.matches(regex)` 方式而言效率要高，且 Matches 提供了各种各样的功能可以实现多样操作。

```java
// 编译匹配 3 位字符的模式。
Pattern p = Pattern.compile("[a-z]{3}");

// 匹配结果存放在 m 里面
Matcher m = p.matche("fgh");

// 判断是否匹配
System.out.println(m.matches());

// 效果一样
System.out.println("fgh".matches("[a-z]{3}"));

// true
// true
```

## Matcher 类

一个 Matcher 对象是一个状态机器，它依据 Pattern 对象做为匹配模式对字符串展开匹配检查。

`boolean matches()` 是最常用方法，尝试对整个目标字符展开匹配检测，也就是只有整个目标字符串完全匹配时才返回真值。
```java
Pattern pattern = Pattern.compile("\\?{2}");

Matcher matcher = pattern.matcher("??");
System.out.println(matcher.matches());

matcher = pattern.matcher("?");
System.out.println(matcher.matches());

// true
// false
```

`boolean find()` 对字符串进行匹配，匹配到的字符串可以在任何位置。
```java
Pattern p = Pattern.compile("\\d+");

System.out.println(p.matcher("22bb23").find());
System.out.println(p.matcher("aa2223").find());
System.out.println(p.matcher("aa2223bb").find());
System.out.println(p.matcher("aabb").find());

// true
// true
// true
// false
```

`int start()` 返回当前匹配到的字符串在原目标字符串中的位置。
`int end()` 返回当前匹配的字符串的最后一个字符在原目标字符串中的索引位置。
`String group()` 返回匹配到的子字符串。
```java
Pattern p = Pattern.compile("\\d+");
Matcher m = p.matcher("aa22bb23");

m.find();
int start = m.start();
String group = m.group();
int end = m.end();

System.out.println(start);
System.out.println(group);
System.out.println(end);

// 2
// 22
// 4
```

`matches()` 、 `find()`都会影响标记，而 `reset()` 可以重置这些标记。
```java
Pattern p = Pattern.compile("\\d{3,5}");
String s = "123-34345-234-00";
Matcher m = p.matcher(s);

System.out.println(m.matches());
// true
System.out.println(m.start());
// 0

System.out.println(m.find());
// true

System.out.println(m.start());
// 4

m.reset();

System.out.println(m.find());
// true

System.out.println(m.start());
// 0
```

`boolean lookingAt()` 对前面的字符串进行匹配，只有匹配到的字符串在最前面才会返回 `true` 。
```java
Pattern p = Pattern.compile("\\d+");
Matcher m = p.matcher("22bb23");
System.out.println(m.lookingAt());

m = p.matcher("bb2233");
System.out.println(m.lookingAt());

// true
// false
```
