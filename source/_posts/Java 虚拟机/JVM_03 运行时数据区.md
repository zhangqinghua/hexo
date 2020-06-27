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
虚拟机栈主管 Java 程序的运行，它保存方法的局部变量（8 种基本数据类型、对象的引用地址）、部分结果，并参与方法的调用和返回。

#### 存放内容

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

## 堆
堆是 Java 虚拟机所管理的内存中最大的一块。堆的生命周期跟 Java 虚拟机一致，随着 Java 虚拟机的创建而创建，随着 Java 虚拟机的销毁而销毁。

#### 存放内容
堆只存放对象。

#### 内存划分
堆在内存中被划分成两个不同的区域：新生代、老年代。其中新生代又可以分成三个区域：Eden、Survivor From、Survivor To。

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=1681675119,796495208&fm=26&gp=0.jpg)

新生代用来存放绝大多数新建的对象，这些对象绝大多数朝生夕灭。所以，一般采用标记-复制算法来进行处理。

新生代又可以分成三个区域：Eden、Survivor From、Survivor To。

Eden 位于堆的新生代，是新对象分配内存的地方。由于堆是所有线程共享的，因此在堆上分配内存需要加锁，非常耗时间。SUN SDK 为了提升效率，会为每个新建的线程在 Eden 上分配一块独立的空间由该线程独享，称为 TLAB（Thread Local Allocation Buffer），在 TLAB 分配内存不需要加锁。如果对象过大或 TLAB 用完，则仍然在堆上进行分配。如果 Eden 内存也用完了，则会进行一次Minor GC。

Survivor From、Survivor To 这两个区是相对的，每次进行 Minor GC 后就互换。在发生 Minor GC 时，Eden 和 Survival From 会把一些仍然存活的对象复制进 Survival To 里，并清除内存。Survival To 会把一些存活得足够旧的对象移至年老代。

默认情况下，Eden 与 Survivor From 和 Survivor To 的大小比例是 8:1:1。另外在统计堆内存时只统计了一个 Survivor 的大小，因为 Survivor 采用了复制算法，同一时间内只能使用一个 Survivor 空间。

老年代里存放的都是存活时间较久的，大小较大的对象，因此年老代使用标记整理算法。当年老代容量满的时候，会触发一次 Major GC，回收年老代和年轻代中不再被使用的对象资源。

#### 参数设置

## 方法区
方法区是所有线程共享。主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。

方法区在不同的虚拟机中有不同的实现。例如在 Hotspot 中方法区有永久代和元空间两种实现。可以把方法区当成接口，永久代或者元空间当作实现。

#### 特点
方法区与堆一样，是各个线程共享的内存区域，当一个类没有加载的话，只能有一个线程去调用 ClassLoader，其他线程想要使用这个类的话就必须得等待，我们只需要加载一次。

方法区在 JVM 启动的时候被创建，并且它的实际的物理内存空间中和堆区一样都可以是不连续的。

方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误。

关闭 JVM 就会释放这个区域的内存。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MDQxMzU3MjQ4OTUucG5n?x-oss-process=image/format,png)

#### 内部结构
方法区存储有已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MTAxNTA1MDQ3NjcucG5n?x-oss-process=image/format,png)

对每个加载的类型（类、接口、枚举、注解），JVM必须在方法区中存储以下类型信息:
1. 这个类型的完整有效名
    例如 `com.icebartech.Child`
1. 这个类型直接父类的完整有效名。
    例如 `com.icebartech.Parent`, 对于 `interface` 或是 `java.lang.Object` 则没有父类。
1. 这个类型的修饰符
    例如 `public`、`abstract`、`final` 的某个子集。
1. 这个类型的接接口
    直接接口一个有序列表。

域的相关信息包括: 域名称、域类型、域修饰符（`public`、`private`、`protected`、`static`、`final`、`volatile`、`transient` 的某个子集）。JVM 必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。

JVM 必须保存所有方法的以下信息，同域信息一样包括声明顺序:
1. 方法名称
1. 方法的返回类型（或 `Void.class`）
1. 方法参数的数量和类型（按顺序）
1. 方法的修饰符（`public`、`private`、`protected`、`static`、`final`、`volatile`、`transient` 的某个子集）
1. 方法的字节码、操作数栈、局部变量表及大小（`abstract` 和 `native` 方法除外）
1. 异常表（`abstract` 和 `native` 方法除外）
1. 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

常量池，可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MTExMzA2MzY0MTkucG5n?x-oss-process=image/format,png)

常量池表是 .class 文件的一部分，用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

运行时常量池是方法区的一部分。在加载类和接口到虚拟机后，就会创建对应的运行时常量池。

数量值、字符串值、类引用、方法引用、字段引用。

#### 为什么需要提供一个常量池呢？
一个 Java 源文件中的类、接口，编译后产生一个字节码文件。而 Java 中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池。

#### Hotspot 中方法区的演进
在JDK7 及以前，习惯上把方法区，称为永久代。JDK8 开始，使用元空间取代了永久代。

本质上，方法区和永久代并不等，仅是对 Hotspot 而言的。《Java 虚拟规范》对如何实现方法区，不做统一要求，例如: BEA JRockit/ IBM J9中不存在永久代的概念。而且从现在看来，当年使用永久代，不是好的 idea。永久代使用的是 Java 虚拟机内存，它会导致 Java 程序更容易 OOM（超过 `-XX:MaxPermSize` 上限）。

到了JDK8，Hotspot 终于完全废弃了永久代的概念，改用与 JRockit、J9 一样的元空间来代替。元空间的本质和永久代类似，都是对JVM规范中方法区的实现，不过元空间与永代最大的区别在于元空间不在虚拟机设置的内存中，而是使用本地内存。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doLzEzOTI1MTcxMzgvaW1nUmVwb3NpdG9yeUBtYXN0ZXIvaW1hZ2UtMjAyMDA2MDQxNTQ5Mzc3NDAucG5n?x-oss-process=image/format,png)


#### 大小设置
方法区的大小不必是固定的，JVM 可以根据应用的需要动态调整。

JDK7 及以前是通过 `-xx:PermSize` 来设置永久代初始分配空间，默认值是 20.75M。通过 `-XX:MaxPermSize` 来设定永久代最大可分配空间，32 位机器默认是 64M，64 位机器模式是 82M。当 JVM 加载的类信息容量超过了这个值，会报异常 `OutOfMemoryError : PermGenspace`。

JDK8 及以后是通过 `-XX: MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 来指定元空间的大小。
默认情况下，`-XX: MetaspaceSize` 的值是 21M，而 `-XX:MaxMetaspaceSize` 的值是 -1，即没有限制，直到虚拟机耗尽所有的可用系统内存发生溢出，抛出 `OutOfMemoryError: Metaspace` 异常。

另外要说明的是，当 JVM 加载的类信息超过初始元空间大小（高水位），Full GC 将会被触发并卸载没用的类（即这些类对应的类加载器不再存活）。然后这个高水位线将会重置，新的高水位线的值取决于 GC 后释放了多少元空间。如果释放的空间不足，那么在不超过 `MaxMetaspaceSize` 时，适当提高该值。如果释放空间过多，则适当降低该值。所以为了避免频繁的 CG，建议将 `-XX:MetaspaceSize` 设置为一个相对较高的值。
