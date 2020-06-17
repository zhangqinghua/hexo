---
title: JVM 内存模型

categories:
- Java 虚拟机

tags:
- JVM

date: 2020-01-21
---

![](https://img-blog.csdn.net/20180730200648583?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![]()

## Java 内存模型
Java 内存模型简称 JMM，定义了 Java 虚拟机（JVM）在计算机内存中的工作方式。

JMM 中分为主内存和工作内存，主内存可粗略认为是堆，工作内存认为是栈。主内存里面存储着所有变量，主内存是共享内存区域，所有线程都可以访问。每一个线程都私有一个工作内存，工作内存里面保存着主内存里面变量值的副本，线程对变量的操作都是在工作内存中完成，操作结束后再放回主内存。

![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTYwOTIxMTgyMzM3OTA0?x-oss-process=image/format,png)