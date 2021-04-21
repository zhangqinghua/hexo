---
title: 新时间与日期 API

categories:
- Java8 新特性

tag:
- Java 8
- 日期
- 时间

date: 2019-07-03
---

Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。

## Date

## Instant
Instant 时间戳是以 Unix 元年（1970年1月1月 00:00:00）到某一时间之间的毫秒值。Instant 默认获取 UTC 时区。

1. 创建 Instant
```java
// 获取当前时间的Instant 
Instant inst = Instant.now();
System.out.prinlnt(inst1); // 2019-07-02T17:26:59.629Z

// 获取指定时间戳的Instant
Instant inst2 = Instant.ofEpochSecond(1023445);
System.out.prinlnt(inst2); // 1970-01-01T02:46:40Z
```

1. 偏移时间
```java
// 向前偏移8个小时获取中国时间
Instant inst = Instant.now();
OffsetDateTime odt = ins1.atOffset(ZoneOffset.ofHours(8));
System.out.println(odt); // 2019-07-03T01:24:37.642+08:00
```

1. 获取时间戳
```java
// 获取时间戳
Instant inst = Instant.now();
System.out.println(ins1.toEpochMilli());
```