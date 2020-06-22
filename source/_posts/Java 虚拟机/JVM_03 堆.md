---
title: JVM_03 堆

categories:
- Java 虚拟机

date: 2020-01-01 00:00:03
---
一个进程对应一个 JVM 实例，一个运行时数据区。一个进程中的多个线程共享同一个方法区、堆空间，各自拥有程序计数器、本地方法栈、虚拟机栈。

在 Java 中所有的对象实例以及数组都应当在运行时分配在堆上。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA1MjMyMTQ5MTc0NTEucG5n?x-oss-process=image/format,png)

## Eden区

TABLE
## 存活区

## 养老区

## 对象分配的一般过程
为新对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收后是否会在内存空间中产生内存碎片。

![](https://blog.csdn.net/qq_29310729/article/details/106405018)