---
title: 并发编程-volatile 关键字

categories:
- Java 并发编程

date: 2020-01-01 00:00:04
---
从是什么、有什么、为什么、如何用、底层原理等几个维度分析 `volatile` 关键字。

#### volatile 是什么
`volatile` 关键字是 Java 虚拟机提供的一个轻量级（乞丐版 `synchronized`）同步机制。

#### volatile 出现背景
1. 多线程出现的可见性、有序性问题。
1. 和 `synchronized` 的比较
   `volatile` 是一种非锁机制，这种机制可以避免锁机制引起的上下文切换。

`volatile` 和 `synchronized` 的区别：
1. `volatile` 本质是在告诉 JVM 当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取； `synchronized` 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住；
1. `volatile` 仅能使用在变量级别；`synchronized` 则可以使用在变量、方法、和类级别的；
1. `volatile` 仅能实现变量的修改可见性，不能保证原子性；而 `synchronized` 则可以保证变量的修改可见性和原子性；
1. `volatile` 不会造成线程的阻塞；`synchronized` 可能会造成线程的阻塞。
1. `volatile` 标记的变量不会被编译器优化；`synchronized` 标记的变量可以被编译器优化；

#### volatile 基本使用
直接修饰变量，参考保证可见性和禁止指令重排。

#### volatile 特点特性
`volatile` 基本上准守了 JMM 的规范，有以下 3 个特性：

1. 保证可见性

1. 保证有序性

1. 不保证原子性（没有准守 JMM 规范）

保证可见性：先来看这么一段程序，线程 T1 调用了 useData 方法，该方法判断 `initFlag` 是否为 true，不是则一直循环等待。线程 T2 修改了 `initFlag` 为 true，但是线程 T1 没有获取，程序一直在死循环中。

```java
public class Test6 {

    public static void main(String[] args) throws InterruptedException {
        SharaData sharaData = new SharaData();
        // 1. 一个线程使用数据
        new Thread(() -> sharaData.useData(), "T1").start();
        Thread.sleep(2000);
        // 2. 一个线程准备使用
        new Thread(() -> sharaData.prepareData(), "T2").start();
    }
}


class SharaData {

    private boolean initFlag = false;

    public void prepareData() {
        System.out.println("prepareing data start...");
        initFlag = true;
        System.out.println("prepareing data finish...");
    }

    public void useData() {
        System.out.println("waiting data...");
        while (!initFlag) {
        }
        System.out.println("===============success");
    }
}

---------------------------------------------------------打印信息----------------------------------------------------------
waiting data...
prepareing data start...
prepareing data finish...
```
 
这时候我们只需要对 `initFlag` 字段加上 `volatile` 修饰，另外一个线程即可获取 `initFlag`  字段的变更：
```java
public class Test6 {

    public static void main(String[] args) throws InterruptedException {
        SharaData sharaData = new SharaData();
        // 1. 一个线程使用数据
        new Thread(() -> sharaData.useData(), "T1").start();
        Thread.sleep(2000);
        // 2. 一个线程准备使用
        new Thread(() -> sharaData.prepareData(), "T2").start();
    }
}


class SharaData {

    private volatile boolean initFlag = false;

    public void prepareData() {
        System.out.println("prepareing data start...");
        initFlag = true;
        System.out.println("prepareing data finish...");
    }

    public void useData() {
        System.out.println("waiting data...");
        while (!initFlag) {
        }
        System.out.println("===============success");
    }
}

---------------------------------------------------------打印信息----------------------------------------------------------
waiting data...
prepareing data start...
prepareing data finish...
===============success

Process finished with exit code 0
```

保证有序性：`volatile` 实现了禁止指令重拍优化，从而避免多线程环境下程序出现乱序执行的现象。

请看下面一段代码，在多线程环境下，语句 1 和 语句 2 可能会发生重排。语句 3 和 语句 4 也可能会发生重排。结果都和预想的不一致。

```java
int     a    = 0;
boolean flag = false;

public void method1() {
   a    = 1;      // 语句 1
   flag = true;   // 语句 2
}

public void method2() {
   if (flag) {
      a = a + 5;                                // 语句 3
      System.out.println("****retValue: " + a); // 语句 4
   }
}
```

采用 `volatile` 修饰后，结果即可正确：

```java
volatile int     a    = 0;
volatile boolean flag = false;

public void method1() {
   a    = 1;      // 语句 1
   flag = true;   // 语句 2
}

public void method2() {
   if (flag) {
      a = a + 5;                                // 语句 3
      System.out.println("****retValue: " + a); // 语句 4
   }
}
```

不保证原子性：原子性即不可分割，完整性，也即某个线程正在做某个具体业务时，中间不可加塞或者被分割。需要整体完整要么同时成功，要么同时失败。

`volatile` 不保证原子性，参考下面例子：

```java
public class Test6 {
    public static void main(String[] args) throws InterruptedException {
        SharaData sharaData = new SharaData();

        // 1. 启动20个线程去添加数据
        for (int i = 0; i < 10; i++) {
            new Thread(() -> sharaData.plus1000()).start();
        }

        // 2. 需要等待上面线程都计算完毕，再使用main线程取得最终结果值
        Thread.sleep(2000);

        // 3. 查看最终结果
        System.out.println("Count: " + sharaData.count);
    }
}


class SharaData {

    public volatile int count = 0;

    public void plus1000() {
        for (int i = 0; i < 1000; i++) {
            count++;
        }
    }
}

---------------------------------------------------------打印信息----------------------------------------------------------
Count: 9146
```

#### volatile 底层原理
保证可见性：`volatile` 关键字解决的问题就是：当一个线程写入该值后，另一个线程读取的必定是新值。

`volatile` 保证了修饰的共享变量在转换为汇编语言时，会加上一个以 `lock` 为前缀的指令，当 CPU 发现这个指令时，立即会做两件事情：

1. 将当前内核中线程工作内存中该共享变量刷新到主存；

1. 通知其他内核里缓存的该共享变量内存地址无效；

参考：MESI 协议

保证有序性：`volatile` 可以禁止指令重排，这就保证了代码的程序会严格按照代码的先后顺序执行，保证了有序性。被 `volatile` 修饰的变量的操作，会严格按照代码顺序执行。

接下来我们就说一下为了实现 `volatile` 内存语义 JMM 是怎样限制重排序（包括编译器重排序和处理器重排序）的。

为了实现 `volatile` 的内存语义，JMM 会限制特定类型的编译器和处理器重排序，JMM 会针对编译器制定 `volatile` 重排序规则表：

1. 第二个操作是volatile写，不管第一个操作是什么都不会重排序
1. 第一个操作是volatile读，不管第二个操作是什么都不会重排序
1. 第一个操作是volatile写，第二个操作是volatile读，也不会发生重排序

![](https://coder-wang-1304346453.cos.ap-beijing.myqcloud.com/blog/20210116222259.png)

如何保证这些操作不会发送重排序呢？就是通过插入内存屏障保证的，JMM层面的内存屏障分为读（load）屏障和写（Store）屏障，排列组合就有了四种屏障。对于volatile操作，JMM内存屏障插入策略：
1. 在每个 `volatile` 写操作的前面插入一个 `StoreStore` 屏障
1. 在每个 `volatile` 写操作的后面插入一个 `StoreLoad` 屏障
1. 在每个 `volatile` 读操作的后面插入一个 `LoadLoad` 屏障
1. 在每个 `volatile` 读操作的后面插入一个 `LoadStore` 屏障

![](https://coder-wang-1304346453.cos.ap-beijing.myqcloud.com/blog/20210116222227.png)

上面的屏障都是 JMM 规范级别的，意思是，按照这个规范写 JDK 能保证 `volatile` 修饰的内存区域的操作不会发送重排序。

在硬件层面上，也提供了一系列的内存屏障来提供一致性的能力。拿X86平台来说，主要提供了这几种内存屏障指令：
1. `lfence` 指令：在 `lfence` 指令前的读操作当必须在 `lfence` 指令后的读操作前完成，类似于读屏障
1. `sfence` 指令：在 `sfence` 指令前的写操作当必须在 `sfence` 指令后的写操作前完成，类似于写屏障
1. `mfence` 指令： 在 `mfence` 指令前的读写操作当必须在 `mfence` 指令后的读写操作前完成，类似读写屏障。

不保证原子性：先来看这么一段代码：

```java
public class Test8 {

    volatile int n = 0;

    public void add() {
        n++;
    }
}
```

我们通过 `javap -c` 命令将上面一段代码编译成为字节码：

```
public class test5.Test8 {
  volatile int n;

  public test5.Test8();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iconst_0
       6: putfield      #2                  // Field n:I
       9: return

  public void add();
    Code:
       0: aload_0
       1: dup                               // 复制栈顶一个字长内存
       2: getfield      #2                  // 获取初始值
       5: iconst_1                          
       6: iadd                              // 加1操作
       7: putfield      #2                  // 写回主内存
      10: return
}
```

从字节码我们可以看到，`n++` 被拆分成为 3 个指令：
1. 执行 `getfield` 拿到原始 n；
1. 执行 `iadd` 进行加 1 操作；
1. 执行 `putfield` 把累加后的值写回主内存；

可以看出，`n++` 操作在多线程环境下是线程不安全的，即使加了 volatile 修饰，也不能保证原子性。

#### 附：volatile 如何保证原子性
1. 加 `synchronized`；
1. 使用原子类如：`ActomInteger` 等；

#### 附：volatile 单例模式分析
DCL（双检索）机制不一定线程安全，原因是有指令重排的存在，原因在于某一个线程执行到第一次检测，读取到的 `instance` 不为 `null` 时，`instance` 的引用对象可能没有完成初始化。

```java
public class Singleton {

   private static Singleton instance = null;

   /**
    * instance = new Singleton() 可以访问以下3步完成（伪代码）
    * 1. memory = allocate()     分配对象内存空间
    * 2. instance(memory)        初始化对象
    * 3. instance = memory       设置 instance 指向刚刚分配的内存地址，此时 instace != null
    *
    * 步骤2和3不存在数据依赖关系，而且无论重排前还是重排后程序的执行结果在单线程中并没有变化，因此这种重排优化是被允许的
    */
   public static getInstance() {
      if (instance == null) {
         sychronized (instance) {
            if (instance == null) {
               instance = new Singleton();
            }
         }
      }
   }
}
```

指令重排只会保证串行语义执行的一致性（单线程），但并不关心多线程之间的语义一致性。所以当一条线程访问 `instace` 不为 `null` 时，由于 `instance` 实例未必已初始化完成，也就造成了线程安全问题。

加入 `volatile` 可以禁止指令重排。

```java
private volatile static Singleton instance = null;
```

#### 附：MESI 协议
在早期的 CPU 中，是通过在总线加 LOCK# 锁的方式实现的，但是这种方式开销太大，所以 Intel 开发了缓存一致性协议，也就是 MESI 协议。

该缓存一致性思路：当 CPU 写数据时，如果发现操作的变量时共享变量，即其他线程的工作内存也存在该变量，于是会发信号通知其他CPU该变量的内存地址无效。当其他线程需要使用这个变量时，如内存地址失效，那么它们会在主存中重新读取该值。

#### 附：指令重排
计算机在执行程序时候，为了提高性能，编译器和处理器常常会对指令做重排，一般分为以下 3 种：
1. 编译器优化的重排
1. 指令并行的重排
1. 内存系统的重排

处理器在进行重排序会确保单线程环境中最终执行结果和代码顺序执行的结果一致。

![](https://dengshuoimg.oss-cn-beijing.aliyuncs.com/hexo%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/java%E6%8C%87%E4%BB%A4%E9%87%8D%E6%8E%92.png)

重排案例一：

```java
int x = 11; // 语句 1
int y = 12; // 语句 2
x = x + 5;  // 语句 3
y = x * x;  // 语句 4
```

1. 正常顺序
   语句 1、语句 2、语句 3、语句 4。

1. 可能重排的顺序
   语句 2、语句 1、语句 3、语句 4。

1. 可能重排的顺序
   语句 1、语句 3、语句 2、语句 4。

1. 不可能重排的顺序
   语句 4、语句 1、语句 2、语句 3。

重排案例二：

```java
int a;
int b;
int x;
int y = 0;

// 线程一
x = a;
b = 1;

// 线程二
y = b;
a = 2;
```

正常情况下，x 的值应该为 0，y 的值为 0。

但是如果编译器对这段代码执行重排优化后，可能会出现以下情况：

```java
// 线程一
b = 1;
x = a;

// 线程二
a = 2;
y = b;
```

这时候 x 的值可能为 2，y 的值为 1。

因此，在多线程环境中，线程交替运行，由于编译器优化重排的存在，两个线程中使用的变量可能会无法保持一致性。

下面的例子中，在多线程环境下，语句 1 和 语句 2 可能会发生重排。语句 3 和 语句 4 也可能会发生重排。结果都和预想的不一致。

```java
int     a    = 0;
boolean flag = false;

public void method1() {
   a    = 1;      // 语句 1
   flag = true;   // 语句 2
}

public void method2() {
   if (flag) {
      a = a + 5;                                // 语句 3
      System.out.println("****retValue: " + a); // 语句 4
   }
}
```

参考：[为什么要做指令重排](https://segmentfault.com/a/1190000015032700)

> 在多线程环境中，需要禁止指令重排。volatile 实现了禁止指令重排优化，从而避免多线程环境下程序出现乱序执行的现象。

#### 附：内存屏障
内存屏障是一种 CPU 指令，用于控制特定条件下的重排序和内存可见性问题。Java 编译器会根据内存屏障的规则禁止重排序。

内存屏障的作用有两个：
1. 保证特定操作的执行顺序；
1. 保证某些变量的内存可见性（利用该特性实现 `volatile` 的内存可见性）；

由于编译器和处理器都能执行指令重排优化，如果在指令间插入一条内存屏障的指令则会告诉编译器和CPU，不管什么指令都不能对这后面的指令重排序。也就是说通过插入内存屏障禁止在内存屏障前后的执行执行重排序优化。

内存屏障的另外一个作用是强制刷出各种 CPU 的缓存数据，因此任何 CPU 上的线程都能读取到这些数据的最新版本。

对 `volatile` 进行读操作时，会在读操作前面加入一条 load 屏障指令，从主内存中读取共享变量。对 `volatile` 进行写操作时，会在写操作后面加入一条 store 屏障指令，将工作内存中的共享变量值刷回到主内存中。

![](http://concurrent.redspider.group/article/02/imgs/%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C.png)

内存屏障可以被分为以下几种类型：
1. `LoadLoad` 屏障
   对于这样「`Load1; LoadLoad; Load2`」的语句 ，在 `Load2` 及后续读取操作要读取的数据被访问前，保证 `Load1` 要读取的数据被读取完毕。
1. `LoadStore` 屏障
   对于这样「`Load1; LoadStore; Store2`」的语句，在 `Store2` 及后续写入操作被刷出前，保证 `Load1` 要读取的数据被读取完毕。
1. `StoreLoad` 屏障
   对于这样「`Store1; StoreLoad; Load2`」的语句，在 `Load2` 及后续所有读取操作执行前，保证 `Store1` 的写入对所有处理器可见。它的开销是四种屏障中最大的。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能。
1. `StoreStore` 屏障
   对于这样「`Store1; StoreStore; Store2`」的语句，在 `Store2` 及后续写入操作执行前，保证 `Store1` 的写入操作对其它处理器可见。