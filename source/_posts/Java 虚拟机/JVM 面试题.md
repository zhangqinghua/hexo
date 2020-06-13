---
title: JVM 面试题

categories:
- Java Virtual Machine Specification

tags:
- JVM
- questions

date: 2020-01-21
---

1. 内存模型以及分区，需要详细到每个区放什么
1. 对象创建方法，对象的内存分配，对象的访问定位
1. GC的两种判定方法：引用计数器和引用链
1. GC的三种三种收集方法：标记清除，标记整理，复制算法的原理与特点，分别用于什么地方，如果让你优化收集算法，有什么思路
1. GC收集器有哪些，GMS收集器与G1收集器的特点
1. Minor GC与Full GC分别发生在什么时候
1. 几种常用的内存调试工具：jmap、jstack、jconsole
1. 类加载的五个过程：加载、验证、准备、解析、初始化
1. 双亲委派模型：BootstrapClassLoader、ExtensionClassLoader、ApplicationClassLoader
1. 分派：静态分派与动态分派