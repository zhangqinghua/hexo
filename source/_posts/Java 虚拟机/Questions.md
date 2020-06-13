---
title: Questions

categories:
- Java Virtual Machine Specification

tags:
- JVM
- questions

date: 2020-01-21
---

#### 如果对象的引用被置为`null`，垃圾收集器是否会立即释放对象占用的内存？

#### `finalize`方法的工作原理

#### Java 8的内存分代改进

#### 分别写出堆内存溢出与栈内存溢出的程序
```java
// 栈溢出
public void f() {
    f();
}

// 堆溢出
public void testd() {
    List<String> list = new ArrayList<>();
    int i = 0;

    while (true) {
        list.add(new String(i + ""));
        i++;
    }
}
```

#### JDK是什么？JRE又是什么？它们是什么关系

![a conceptual diagram of Oracle's Java SE products](001.png)

#### Java中的对象访问是如何进行的

## 运行时数据区
#### 运行时数据区包含哪些

### 堆
#### 为什么要将堆内存分区

#### 堆内存分为哪几块

### 桢

#### 反射中，`Class.forName()`和`ClassLoader.loadClass()`区别是什么

#### 说说强引用、软引用、弱引用、虚引用以及他们之间和GC的关系

#### JVM 垃圾回收机制，何时触发 MinorGC 等操作

#### 对象如何晋升到老年代

#### Eden和Survivor的比例分配等

#### 什么是类加载器的双亲委派模型

#### 什么是指令重排序、内存屏障与先行发生原则

#### `volatile`的语义，它修饰的变量一定线程安全吗

#### 堆和栈有什么区别

#### 堆内存中到底存在着什么东西

#### 类变量是否在JVM启动时就初始化好的

#### Java 的方法（函数）到底是传值还是传址

#### `OutOfMemory`错误分几种

#### 为什么会产生`StackOverflowError`

#### 一个机器上可以有多个JVM吗

#### JVM之间可以互访吗

#### JVM中到底哪些区域是共享的

#### JVM有哪些调整参数


## GC
#### 如何判断对象是否死去
#### 有哪些垃圾收集算法
#### Minor GC和Full CG有什么区分

## 优化
####  一个线程默认占多少内存

#### 一台机器可以创建多少个线程

#### 线程切换消耗一般是多少