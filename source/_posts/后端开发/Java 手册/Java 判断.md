---
title: Java 判断

categories:
- 后端开发
- Java 手册

date: 2021-05-07
---
## 数字判断
参考：[Java判断字符串是否为数字的多种方式，你用对了吗](https://blog.csdn.net/mryang125/article/details/113146057)

#### 异常方式
```java
public static boolean isNumeric1(String str) {
   try {
      Double.parseDouble(str);
      return true;
   } catch(Exception e){
      return false;
   }
}
```

#### 数字字符
字符串的底层实现其实就是字符数组，如果这个字符数组中每个字符都是数字，那么这个字符串不就是数字字符串了吗？利用 `java.lang.Character#isDigit(int)` 判断所有字符是否为数字字符从而达到判断数字字符串的目的：

```java
public static boolean isNumeric4(String str) {
if (str == null) return false;
   for (char c : str.toCharArray ()) {
      if (!Character.isDigit(c)) return false;
   }
   return true;
}
```

如果你的 Java 版本是 8 以上，以上的写法可以替换成如下 `Stream` 流的方式，从而看起来更优雅。所以，茴香豆的‘茴’又多了一种写法！

```java
public static boolean isNumeric4(String str) {
   return str != null && str.chars().allMatch(Character::isDigit);
}
```

#### 正则表达式
```java
private static final Pattern NUMBER_PATTERN = Pattern.compile("-?\\d+(\\.\\d+)?");

public static boolean isNumeric2(String str) {
   return str != null && NUMBER_PATTERN.matcher(str).matches();
}
```

#### NumberFormat
通常使用 `NumberFormat` 类的 format 方法将一个数值格式化为符合某个国家地区习惯的数值字符串，例如我们输入 18，希望输出 18￥，使用这个类再好不过了。我们也可以用该类的 `parse` 方法来判断输入字符串是否为数字。

```java
public static boolean isNumeric3(String str) {
   if (str == null) return false;
   NumberFormat formatter = NumberFormat.getInstance();
   ParsePosition pos = new ParsePosition(0);
   formatter.parse(str, pos);
   return str.length() == pos.getIndex();
}
```

#### 外部工具类
使用外部工具类通常需要引入外部 jar 文件，一般的依赖是 Apache 的 `comons-lang`：

```xml
<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-lang3</artifactId>
</dependency>
```

```java
// isParsable只接受十进制数字字符串
public static boolean isNumeric5(String str) {
   return NumberUtils.isParsable(str);
}

// isCreatable不仅可以接受十进制数字字符串，还可以接受八进制，十六进制以及科学计数法表示的数字字符串 
public static boolean isNumeric6(String str) {
   return NumberUtils.isCreatable(str);
}

// isNumeric会认为空字符串为非法数字    >>> 只能判断整数
public static boolean isNumeric7(String str) {
   return StringUtils.isNumeric(str);
}

// isNumericSpace则认为空字符串也是数字 >>> 只能判断整数
public static boolean isNumeric8(String str) {
   return StringUtils.isNumericSpace(str);
}
```