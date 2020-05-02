---
title: Chapter 1.2 异常体系

categories:
- Interview

date: 2020-04-28 00:00:12
---
#### Java 的异常体系
![](https://images2015.cnblogs.com/blog/936870/201707/936870-20170709120346087-1351539391.png)
- Throwable 类是 Java 语言中所有错误或异常的超类。它包含了其线程创建时线程执行堆栈的快照。它还包含了给出有关错误更多信息的消息字符串
- Error 是 Throwable 的子类，用于指示合理的应用程序不应该试图捕获的严重问题
- Exception 异常主要分为两类
    - 一类是 IOException（I/O 输入输出异常），其中 IOException 及其子类异常又被称作「受查异常」
    - 另一类是 RuntimeException（运行时异常），RuntimeException 被称作「非受查异常」

