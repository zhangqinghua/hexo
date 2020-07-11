---
title: Stream 示例

categories:
- Java8 新特性

tags:
- Java 8

date: 2019-08-27
---

Stream 常用示例

1. 收集起始时间到结束时间之间所有的时间并以字符串集合方式返回
```java
/**
 * 收集起始时间到结束时间之间所有的时间并以字符串集合方式返回
 * @param  start 2018-10-11
 * @param  end   2018-10-15
 * @return 2018-10-11,2018-10-12,2018-10-13
 */
public static List<String> collectLocalDates(LocalDate start, LocalDate end){
	// 用起始时间作为流的源头，按照每次加一天的方式创建一个无限流
	return Stream.iterate(start, localDate -> localDate.plusDays(1))
	     // 截断无限流，长度为起始时间和结束时间的差+1个
	     .limit(ChronoUnit.DAYS.between(start, end) + 1)
	     // 由于最后要的是字符串，所以map转换一下
	     .map(LocalDate::toString)
	     // 把流收集为List
	     .collect(Collectors.toList());
}
```

1. List<Res> 数组，将 value 属性逗号拼接
```java
String str = List.stream().map(Res::getValue).collect(Collectors.joining(","));
```

1. `String` 类型的 `List` 集合转大写
```java
List<String> alpha = Arrays.asList("a", "b", "c", "d");

// Before Java8
List<String> alphaUpper = new ArrayList<>();
for (String s : alpha) {
    alphaUpper.add(s.toUpperCase());
}
System.out.println(alpha); // [a, b, c, d]
System.out.println(alphaUpper); // [A, B, C, D]

// Java 8
List<String> collect = alpha.stream().map(String::toUpperCase).collect(Collectors.toList());
System.out.println(collect); // [A, B, C, D]

// Extra, streams apply to any data type.
List<Integer> num = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> collect1 = num.stream().map(n -> n * 2).collect(Collectors.toList());
System.out.println(collect1); // [2, 4, 6, 8, 10]
```

1. `Object` 类型的 List 转 `String` 类型的 `List`
```java
List<Staff> staff = Arrays.asList(
                new Staff("mkyong", 30, new BigDecimal(10000)),
                new Staff("jack", 27, new BigDecimal(20000)),
                new Staff("lawrence", 33, new BigDecimal(30000)));
// Before Java 8
List<String> result = new ArrayList<>();
for (Staff x : staff) {
    result.add(x.getName());
}
System.out.println(result); // [mkyong, jack, lawrence]

// Java 8
List<String> collect = staff.stream().map(x -> x.getName()).collect(Collectors.toList());
System.out.println(collect); // [mkyong, jack, lawrence]

```

1. `Object` 类型的 List 转其他 `Object` 类型的 `List`
```java
// Before Java 8
List<Staff> staff = Arrays.asList(
        new Staff("mkyong", 30, new BigDecimal(10000)),
        new Staff("jack", 27, new BigDecimal(20000)),
        new Staff("lawrence", 33, new BigDecimal(30000))
);
List<StaffPublic> result = new ArrayList<>();
for (Staff temp : staff) {    
    StaffPublic obj = new StaffPublic();
    obj.setName(temp.getName());
    obj.setAge(temp.getAge());
    if ("mkyong".equals(temp.getName())) {
        obj.setExtra("this field is for mkyong only!");
    }
    result.add(obj);
}
System.out.println(result);

// Java 8
List<StaffPublic> result = staff.stream().map(temp -> {
    StaffPublic obj = new StaffPublic();
    obj.setName(temp.getName());
    obj.setAge(temp.getAge());
    if ("mkyong".equals(temp.getName())) {
        obj.setExtra("this field is for mkyong only!");
    }
    return obj;
}).collect(Collectors.toList());
```
