---
title: JVM_08 性能监控

categories:
- Java 虚拟机

date: 2020-01-01 00:00:08
---

## jps
jps （JVM Process Status Tool）显示指定系统内所有的 HotSpot 虚拟机进程。

命令格式：

```bash
$ jps [options] [hostid]
```

option 参数：
1. -l : 输出主类全名或jar路径
1. -q : 只输出LVMID
1. -m : 输出JVM启动时传递给main()的参数
1. -v : 输出JVM启动时显示指定的JVM参数（默认）

示例：

```bash
$ jps -l -m
  28920 org.apache.catalina.startup.Bootstrap start
  11589 org.apache.catalina.startup.Bootstrap start
  25816 sun.tools.jps.Jps -l -m
```

## jinfo
jinfo（JVM Configuration info）这个命令作用是实时查看和调整虚拟机运行参数。

之前的 `jps -v` 口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用 jinfo 口令。

命令格式：
```
$ jinfo [option] [args] LVMID
```

option 参数：
1. -flag : 输出指定args参数的值
1. -flags : 不需要args参数，输出所有JVM参数的值
1. -sysprops : 输出系统属性，等同于System.getProperties()

示例：
```bash
$ jinfo -flag 11494
-XX:CMSInitiatingOccupancyFraction=80
```

## jstat
jstat（JVM statistics Monitoring）是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

#### -class
监视类装载、卸载数量、总空间以及耗费的时间。

```bash
# Loaded :      加载 Class 的数量
# Bytes :       Class 字节大小
# Unloaded :    未加载 Class 的数量
# Bytes :       未加载 Class 的字节大小
# Time :        加载时间
zhangqinghua$ jstat -class 11589
Loaded  Bytes  Unloaded  Bytes     Time   
7035  14506.3     0     0.0       3.67
```

#### -compiler
输出JIT编译过的方法数量耗时等。

```bash
# Compiled :      编译数量
# Failed :        编译失败数量
# Invalid :       无效数量
# Time :          编译耗时
# FailedType :    失败类型
# FailedMethod :  失败方法的全限定名
zhangqinghua$ jstat -compiler 1262
Compiled Failed   Invalid  Time     FailedType  FailedMethod
2573      1       0        47.60    1           org/apache/catalina/loader/WebappClassLoader findResourceInternal  
```

#### -gc
垃圾回收堆的行为统计：

```bash
# C 即 Capacity 总容量，U 即 Used 已使用的容量
# S0C :  survivor0区的总容量
# S1C :  survivor1区的总容量
# S0U :  survivor0区已使用的容量
# S1C :  survivor1区已使用的容量
# EC :   Eden区的总容量
# EU :   Eden区已使用的容量
# OC :   Old区的总容量
# OU :   Old区已使用的容量
# PC :   当前perm的容量 (KB)
# PU :   perm的使用 (KB)
# YGC :  新生代垃圾回收次数
# YGCT : 新生代垃圾回收时间
# FGC :  老年代垃圾回收次数
# FGCT : 老年代垃圾回收时间
# GCT :  垃圾回收总消耗时间
zhangqinghua$ jstat -gc 1262
S0C    S1C     S0U     S1U   EC       EU        OC          OU        PC       PU         YGC    YGCT    FGC    FGCT     GCT   
26112.0 24064.0 6562.5  0.0   564224.0 76274.5   434176.0   388518.3  524288.0 42724.7    320    6.417   1      0.398    6.815

# 这个命令意思就是每隔 2000ms 输出 1262 的 gc 情况，一共输出 20 次
zhangqinghua$ jstat -gc 1262 2000 20
...
```

#### -gccapacity
同 `-gc`，不过还会输出Java堆各区域使用到的最大、最小空间：

```bash
# NGCMN :   新生代占用的最小空间
# NGCMX :   新生代占用的最大空间
# OGCMN :   老年代占用的最小空间
# OGCMX :   老年代占用的最大空间
# OGC：     当前年老代的容量 (KB)
# OC：      当前年老代的空间 (KB)
# PGCMN :   perm占用的最小空间
# PGCMX :   perm占用的最大空间
zhangqinghua$ jstat -gccapacity 1262
 NGCMN    NGCMX     NGC    S0C   S1C       EC         OGCMN      OGCMX      OGC        OC
614400.0 614400.0 614400.0 26112.0 24064.0 564224.0   434176.0   434176.0   434176.0   434176.0 

PGCMN    PGCMX     PGC      PC         YGC    FGC 
524288.0 1048576.0 524288.0 524288.0    320     1  
```

#### -gcutil
同 `-gc`，不过输出的是已使用空间占总空间的百分比：

```bash
# LGCC： 最近垃圾回收的原因
# GCC：  当前垃圾回收的原因
zhangqinghua$ jstat -gcutil 28920
S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
12.45   0.00  33.85   0.00   4.44  4       0.242     0    0.000    0.242
```

#### -gcnew
统计新生代的行为：

```bash
# NGC:   当前年轻代的容量 (KB)
# S0CMX: 最大的S0空间 (KB)
# S0C:   当前S0空间 (KB)
# ECMX:  最大eden空间 (KB)
# EC:    当前eden空间 (KB)
zhangqinghua$ jstat -gcnew 28920
S0C      S1C      S0U        S1U  TT  MTT  DSS      EC        EU         YGC     YGCT  
419392.0 419392.0 52231.8    0.0  6   6    209696.0 3355520.0 1172246.0  4       0.242
```

#### -gcold
统计旧生代的行为：

```bash
zhangqinghua$ jstat -gcold 28920
PC       PU        OC           OU       YGC    FGC    FGCT     GCT   
1048576.0  46561.7   6291456.0     0.0      4      0      0.000    0.242
```

#### -gcoldcapacity
统计旧生代的大小和空间：

```bash
zhangqinghua$ jstat -gcoldcapacity 28920
OGCMN       OGCMX        OGC         OC         YGC   FGC    FGCT     GCT   
6291456.0   6291456.0   6291456.0   6291456.0     4     0    0.000    0.242
```

#### -gcpermcapacity
永生代行为统计：

```bash
zhangqinghua$ jstat -gcpermcapacity 28920
PGCMN      PGCMX       PGC         PC      YGC   FGC    FGCT     GCT   
1048576.0  2097152.0  1048576.0  1048576.0     4     0    0.000    0.242
```

#### -printcompilation
hotspot 编译方法统计：

```bash
# Compiled：被执行的编译任务的数量
# Size：    方法字节码的字节数
# Type：    编译类型
# Method：  编译方法的类名和方法名。类名使用"/" 代替 "." 作为空间分隔符. 方法名是给出类的方法名。
#           格式是一致于 HotSpot - XX:+PrintComplation 选项
jstat -printcompilation 28920
Compiled  Size  Type Method
1291      78     1    java/util/ArrayList indexOf
```

## jmap
jmap（JVM Memory Map）命令用于生成 heap dump 文件，如果不使用这个命令，还阔以使用 `-XX:+HeapDumpOnOutOfMemoryError` 参数来让虚拟机出现 OOM 的时候自动生成 dump 文件。

jmap 不仅能生成 dump 文件，还阔以查询 `finalize` 执行队列、Java 堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

命令格式：

```bash
jmap [option] LVMID
```

option 参数：
1. dump : 生成堆转储快照
1. finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
1. heap : 显示Java堆详细信息
1. histo : 显示堆中对象的统计信息
1. permstat : to print permanent generation statistics
1. F : 当-dump没有响应时，强制生成dump快照

#### -dump
dump 堆到文件，format 指定输出格式，live 指明是活着的对象,file 指定文件名：

```sql
$ jmap -dump:live,format=b,file=dump.hprof 28920
  Dumping heap to /home/xxx/dump.hprof ...
  Heap dump file created
```

dump.hprof 这个后缀是为了后续可以直接用MAT（Memory Anlysis Tool）打开。

#### -finalizerinfo
打印等待回收对象的信息：

```bash
$ jmap -finalizerinfo 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  Number of objects pending for finalization: 0
```

可以看到当前 F-QUEUE 队列中并没有等待 Finalizer 线程执行 `finalizer` 方法的对象。

#### -heap
打印 heap 的概要信息，GC 使用的算法，heap 的配置及 wise heap 的使用情况,可以用此来判断内存目前的使用情况以及垃圾回收情况。

```bash
$ jmap -heap 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01  

  using thread-local object allocation.
  Parallel GC with 4 thread(s)//GC 方式  

  Heap Configuration: //堆内存初始化配置
     MinHeapFreeRatio = 0 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
     MaxHeapFreeRatio = 100 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
     MaxHeapSize      = 2082471936 (1986.0MB) //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
     NewSize          = 1310720 (1.25MB)//对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
     MaxNewSize       = 17592186044415 MB//对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
     OldSize          = 5439488 (5.1875MB)//对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
     NewRatio         = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
     SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
     PermSize         = 21757952 (20.75MB)  //对应jvm启动参数-XX:PermSize=<value>:设置JVM堆的‘永生代’的初始大小
     MaxPermSize      = 85983232 (82.0MB)//对应jvm启动参数-XX:MaxPermSize=<value>:设置JVM堆的‘永生代’的最大大小
     G1HeapRegionSize = 0 (0.0MB)  

  Heap Usage://堆内存使用情况
  PS Young Generation
  Eden Space://Eden区内存分布
     capacity = 33030144 (31.5MB)//Eden区总容量
     used     = 1524040 (1.4534378051757812MB)  //Eden区已使用
     free     = 31506104 (30.04656219482422MB)  //Eden区剩余容量
     4.614088270399305% used //Eden区使用比率
  From Space:  //其中一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  To Space:  //另一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  PS Old Generation //当前的Old区内存分布
     capacity = 86507520 (82.5MB)
     used     = 0 (0.0MB)
     free     = 86507520 (82.5MB)
     0.0% used
  PS Perm Generation//当前的 “永生代” 内存分布
     capacity = 22020096 (21.0MB)
     used     = 2496528 (2.3808746337890625MB)
     free     = 19523568 (18.619125366210938MB)
     11.337498256138392% used  

  670 interned Strings occupying 43720 bytes.
```

可以很清楚的看到 Java 堆中各个区域目前的情况。

#### -histo
打印堆的对象统计，包括对象数、内存大小等等 （因为在 dump:live 前会进行 full gc，如果带上 live 则只统计活对象，因此不加 live 的堆大小要大于加 live 堆的大小）。

```bash
$ jmap -histo:live 28920 | more
 num     #instances         #bytes  class name
----------------------------------------------
   1:         83613       12012248  <constMethodKlass>
   2:         23868       11450280  [B
   3:         83613       10716064  <methodKlass>
   4:         76287       10412128  [C
   5:          8227        9021176  <constantPoolKlass>
   6:          8227        5830256  <instanceKlassKlass>
   7:          7031        5156480  <constantPoolCacheKlass>
   8:         73627        1767048  java.lang.String
   9:          2260        1348848  <methodDataKlass>
  10:          8856         849296  java.lang.Class
  ....
```

`class name` 是对象类型，说明如下：

```
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象
```

#### -permstat
打印Java堆内存的永久保存区域的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

```bash
$ jmap -permstat 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  finding class loader instances ..done.
  computing per loader stat ..done.
  please wait.. computing liveness.liveness analysis may be inaccurate ...
  
  class_loader            classes bytes   parent_loader           alive?  type  
  <bootstrap>             3111    18154296          null          live    <internal>
  0x0000000600905cf8      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006008fcb48      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006016db798      0       0       0x00000006008d3fc0      dead    java/util/ResourceBundle$RBClassLoader@0x0000000780626ec0
  0x00000006008d6810      1       3056      null          dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
```

#### -F
强制模式。如果指定的 pid 没有响应，请使用 `jmap -dump` 或 `jmap -histo` 选项。此模式下，不支持 live 子选项。

## jstack
jstack用于生成java虚拟机当前时刻的线程快照。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

#### 命令格式
```bash
$ jstack [option] LVMID
```

#### option 参数
1. -F : 当正常输出请求不被响应时，强制输出线程堆栈
1. -l : 除堆栈外，显示关于锁的附加信息
1. -m : 如果调用到本地方法的话，可以显示C/C++的堆栈

#### 示例
```bash
$ jstack -l 11494|more
2016-07-28 13:40:04
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.71-b01 mixed mode):

"Attach Listener" daemon prio=10 tid=0x00007febb0002000 nid=0x6b6f waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"http-bio-8005-exec-2" daemon prio=10 tid=0x00007feb94028000 nid=0x7b8c waiting on condition [0x00007fea8f56e000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000cae09b80> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
      .....
```

[参考](https://www.cnblogs.com/ityouknow/p/5714703.html)

## 问题
JVM相关的分析工具有使用过哪些？具体的性能调优步骤吗？

讲一下 OOM 以及遇到这种情况怎么处理的，是否使用过日志分析工具

你熟悉的JVM调优参数，使用过哪些调优工具？

VisualVM:JDK自带JVM可视化工具，能过对内存、gc、cpu、thread、class、变量等等信息进行可视化。

几种常用的内存调试工具：jmap、jstack、jconsole