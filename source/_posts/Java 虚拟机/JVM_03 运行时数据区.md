---
title: JVM_03 运行时数据区

categories:
- Java 虚拟机

date: 2020-01-01 00:00:03
---
虚拟机的内存区域又称为运行时数据区，分为线程私有区域（程序计数器、虚拟机栈、本地方法区）、线程共享区域（堆，方法区）和直接内存。

线程私有区域的生命周期与线程相同，随线程的启动而创建，随线程的结束而销毁。线程共享区域随虚拟机的启动而创建，随虚拟机的关闭而销毁。

直接内存也称为堆外内存，它并不是运行时数据区的一部分，但在并发编程中被频繁使用。NIO 模块提供的基于 `Channel` 与 `Buffer` 的 I/O 操作方式就是基于堆外内存实现的。

## 程序计数器
> 私有、行号、空、极小、无溢出

程序计数器是一块很小的内存空间，用与存储当前运行的线程所执行的字节码的行号指示器。每个运行中的线程都有一个独立的程序计数器，在方法正在执行时，该方法的程序计数器记录的是实时虚拟机字节码指令的地址；如果该方法执行的是 Native 方法，则程序计数器的值为空。

程序计数器属于“线程私有”的内存区域，它是唯一没有内存溢出（OOM）的区域。

![](https://img-blog.csdnimg.cn/20200526220317683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmNlbjk=,size_16,color_FFFFFF,t_70#pic_center)

## 虚拟机栈
> 私有、方法执行过程、栈帧、局部变量表、操作数据栈、动态链接、方法出口

虚拟机栈是描述 Java 方法执行过程的内存模型，它在当前栈帧中存储了局部变量表、操作数据栈、动态链接、方法出口等信息。

栈帧用来记录方法的执行过程，在方法被执行时虚拟机会为其创建一个与之对应的栈帧，方法的执行和返回对应栈帧在虚拟机中的入栈与出栈。无论方法是正常完成还是异常完成，都视为方法运行结束。

![](https://img2020.cnblogs.com/blog/1889810/202006/1889810-20200613173051867-127265622.png)

## 本地方法区
> Native

本地方法区和虚拟机栈的作用相似，区别是虚拟机栈为执行 Java 方法服务，本地方法区（栈）为执行 Native 方法服务。

## 堆
> 共享 新生代 老年代

在虚拟机运行过程中创建的对象和产生的数据都被存储在堆中，堆是被线程共享的内存区域，也是垃圾收集器进行垃圾回收的最主要的内存区域。

由于现代虚拟机采用分代收集算法，因此堆从 GC 的角度还可以细分为：新生代、老年代和永久代。

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1681675119,796495208&fm=26&gp=0.jpg)

#### 新生代
新创建的对象都会放在新生代（除了大对象外），新生代默认占用 1/3 的堆内存空间。由于虚拟机会频繁创建对象，所以新生代会频繁触发 MinorGC 进行垃圾回收。

因为 98% 的对象都是“朝生夕死”，为了高效的利用内存，新生代又分为 Eden 区、SurvivorFrom 区和 SurvivorTo 区，如下所述：
1. Eden 区
    Java 新创建的对象会首先被存放在 Eden 区，如果新创建的对象属于大对象，则直接将其分配到老年代。在 Eden 区的内存空间不足时会触发 MinorGC，对新生代进行垃圾回收。

    大对象一般为 2kb ~ 128kb，可以通过 `-XX:PretenusrSizeThreshold` 进行设置。
1. SurvivorTo 区
    保留上一次 MinorGC 时的幸存者。
1. SurvivorFrom 区
    将上一次 MinorGC 时的幸存者作为这一次 MinorGC 的被扫描者。

新生代的 GC 过程称为 MinorGC，采用复制算法实现，具体如下：
1. 把在 Eden 和 SurvivorFrom 区存活的对象复制到 SurvivorTo 区中，同时把这些对象的年龄加 1；如果对象的年龄达到老年代的标准或 SirvivorTo 区的内存空间不够，则直接将其复制到老年代。
1. 清空 Eden 和 SurvivorFrom 区中的对象。
1. 将 SurvivorFrom 和 SurvivorTo 区互换，原来的 SurvivorTo 区成为下一次 GC 时的 SurvivorFrom 区。

> 达到老年代的标准的次数默认为 15，可以通过 `-XX:MaxTenuringThreshold` 设置。

#### 老年代
老年代主要存放长生命周期或比较大的对象。老年代的 GC 过程叫作 MajorGC。在老年代，对象比较稳定，MajorGC 不会被频繁触发。在进行 MajorGC 是，虚拟机会进行一次 MinorGC，在 MinorGC 过后仍然出现老年代且当老年代空间不足或无法找到足够大的连续内存空间分配给新创建的大对象时，出触发 MajorGC 进行垃圾回收，释放虚拟机的内存空间。

MajorGC 采用标记清除算法，该算法首先会扫描所有对象并标记存活的对象，然后回收未被标记的对象，并释放内存空间。

因为要先扫描老年代的对象再回收，所以 MajorGC 的耗时较长。MajorGC 的标记清除算法容易产生内存碎片。在老年代没有内存空间可分配时，会抛出 OOM 异常。

## 方法区
方法区是所有线程共享，主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。

方法区在不同的虚拟机中有不同的实现。例如在 Hotspot 中方法区有永久代和元空间两种实现。可以把方法区当成接口，永久代或者元空间当作实现。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MDQxMzU3MjQ4OTUucG5n?x-oss-process=image/format,png)

#### 特点
方法区与堆一样，是各个线程共享的内存区域，当一个类没有加载的话，只能有一个线程去调用 ClassLoader，其他线程想要使用这个类的话就必须得等待。类只需要加载一次。

方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误。

#### 内部结构
方法区存储有已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MTAxNTA1MDQ3NjcucG5n?x-oss-process=image/format,png)

对每个加载的类型（类、接口、枚举、注解），虚拟机必须在方法区中存储以下类型信息:
1. 这个类型的完整有效名
    例如 `com.icebartech.Child`

1. 这个类型直接父类的完整有效名。
    例如 `com.icebartech.Parent`, 对于 `interface` 或是 `java.lang.Object` 则没有父类。

1. 这个类型的修饰符
    例如 `public`、`abstract`、`final` 的某个子集。

1. 这个类型的接口
    直接接口一个有序列表。

域的相关信息包括: 域名称、域类型、域修饰符（`public`、`private`、`protected`、`static`、`final`、`volatile`、`transient` 的某个子集）。xuniji必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。

JVM 必须保存所有方法的以下信息，同域信息一样包括声明顺序:
1. 方法名称
1. 方法的返回类型（或 `Void.class`）
1. 方法参数的数量和类型（按顺序）
1. 方法的修饰符（`public`、`private`、`protected`、`static`、`final`、`volatile`、`transient` 的某个子集）
1. 方法的字节码、操作数栈、局部变量表及大小（`abstract` 和 `native` 方法除外）
1. 异常表（`abstract` 和 `native` 方法除外）
1. 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

常量池，可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型。

常量池表是 .class 文件的一部分，用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

运行时常量池是方法区的一部分。在加载类和接口到虚拟机后，就会创建对应的运行时常量池。

数量值、字符串值、类引用、方法引用、字段引用。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MTExMzA2MzY0MTkucG5n?x-oss-process=image/format,png)

#### 为什么需要提供一个常量池呢？
一个 Java 源文件中的类、接口，编译后产生一个字节码文件。而 Java 中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池。

#### Hotspot 中方法区的演进
在 JDK7 及以前，习惯上把方法区称为永久代。JDK8 开始，使用元空间取代了永久代。

本质上，方法区和永久代并不等，仅是对 Hotspot 而言的。《Java 虚拟规范》对如何实现方法区，不做统一要求，例如: BEA JRockit/ IBM J9中不存在永久代的概念。而且从现在看来，当年使用永久代，不是好的 idea。永久代使用的是 Java 虚拟机内存，它会导致 Java 程序更容易 OOM（超过 `-XX:MaxPermSize` 上限）。

到了JDK8，Hotspot 终于完全废弃了永久代的概念，改用与 JRockit、J9 一样的元空间来代替。元空间的本质和永久代类似，都是对JVM规范中方法区的实现，不过元空间与永代最大的区别在于元空间不在虚拟机设置的内存中，而是使用本地内存。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MDQxNTQ5Mzc3NDAucG5n?x-oss-process=image/format,png)


#### 大小设置
方法区的大小不必是固定的，JVM 可以根据应用的需要动态调整。

JDK7 及以前是通过 `-xx:PermSize` 来设置永久代初始分配空间，默认值是 20.75M。通过 `-XX:MaxPermSize` 来设定永久代最大可分配空间，32 位机器默认是 64M，64 位机器模式是 82M。当 JVM 加载的类信息容量超过了这个值，会报异常 `OutOfMemoryError : PermGenspace`。

JDK8 及以后是通过 `-XX: MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 来指定元空间的大小。
默认情况下，`-XX: MetaspaceSize` 的值是 21M，而 `-XX:MaxMetaspaceSize` 的值是 -1，即没有限制，直到虚拟机耗尽所有的可用系统内存发生溢出，抛出 `OutOfMemoryError: Metaspace` 异常。

另外要说明的是，当 JVM 加载的类信息超过初始元空间大小（高水位），Full GC 将会被触发并卸载没用的类（即这些类对应的类加载器不再存活）。然后这个高水位线将会重置，新的高水位线的值取决于 GC 后释放了多少元空间。如果释放的空间不足，那么在不超过 `MaxMetaspaceSize` 时，适当提高该值。如果释放空间过多，则适当降低该值。所以为了避免频繁的 CG，建议将 `-XX:MetaspaceSize` 设置为一个相对较高的值。

## 问题
1. 讲一下 JVM 的内存结构。
程序计数器、虚拟机栈、本地方法区、堆、方法区
私有共享、溢出

1. 堆分为哪几部分，默认年龄多大进入老年代？
新生代，Eden，ServivorFrom、ServivorTo，老年代。15

1. JVM 的内存结构，Java8 做了什么修改？
永久区、元数据

#### Java 堆
1. 新生代分为几个区？
Eden、ServivorFrom、ServivorTo

1. 新生代使用什么算法进行垃圾回收？为什么使用这个算法？
复制算法。过程
因为复制算法的实现，导致复制算法适用于对象较少的情况下，当Eden区的内存被填满，会触发minorGC ，Eden区对象会从Eden转移到Survivor区，随着年龄的增加再到老年代。

1. JVM 内存结构详细分配，各比例是多少
堆？1:2 8:1:1

1. JVM 数据存储模型，新生代、年老代的构造？

1. JVM 内存模型，新生代和老年的回收机制？
什么时候回收、复制算法、标志整理、碎片、各种虚拟机的实现

#### 内存溢出
1. 有哪些内存溢出的异常？触发条件是什么
栈溢出：默认栈大小、1w个栈帧、方法调用太多，或者方法内的局部变量太多、、
堆溢出：老年代空间不足、对象太多，
方法区和运行时常量池溢出
