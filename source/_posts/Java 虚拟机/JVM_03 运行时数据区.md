---
title: JVM_03 运行时数据区

categories:
- Java 虚拟机

date: 2020-01-01 00:00:03
---

## 程序计数器
程序计数器也称 PC 寄存器，用来存储指向下一条指令的地址，也即将要执行的指令代码，由执行引擎读取下一条。它是一个很小的空间，几乎可以忽略不计，也是运行速度最快的存储区域。

![](https://img-blog.csdnimg.cn/20200526220317683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmNlbjk=,size_16,color_FFFFFF,t_70#pic_center)

> 声明：简要理解也就是因为在运行时 CPU 需要不停的切换个线程，这时候就需要 PC 寄存器来记录当前线程运行到哪个位置了，即将运行哪里给记录下来，等待下次切换回后进行调用。

## 虚拟机栈
虚拟机栈主管 Java 程序的运行，它保存方法的局部变量（8种基本数据类型、对象的引用地址）、部分结果，并参与方法的调用和返回。

毎个线程在创建吋都会创建一个虚拟机栈，其内部保存一个个的栈帧，对应着一次次的 Java 方法凋用。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA1MDMyMzUzMDQzNDUucG5n?x-oss-process=image/format,png)

#### 栈帧
毎个栈帧中存储着：
1. 局部变量表
1. 操作数栈（或表达式栈）
    主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间。
1. 动态链接（或指向运行吋常量池的方法引用）
    每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接。
    
    比如: `invokedynamic` 指令在 Java 源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用保存在 class 文件的常量池里。比如:描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用。

1. 方法返回地址（或方法正常退出或者异常退出的定义）的一些附加信息

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA1MDQxNjM2MDI0MzUucG5n?x-oss-process=image/format,png)

栈帧的大小取决于内部结构的大小。

#### 操作数栈


## 异常


## 堆