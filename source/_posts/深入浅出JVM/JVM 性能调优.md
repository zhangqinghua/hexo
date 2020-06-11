---
title: JVM 性能调优

categories:
- Java Virtual Machine Specification

tags:
- JVM

date: 2020-01-21
---
性能调优的目的就是使用较小的内存占用来获得较高的吞吐量或者较低的延迟。

程序在上线前的测试或运行中有时会出现一些大大小小的 JVM 问题，比如 CPU Load 过高、请求延迟、TPS 降低等，甚至出现内存泄漏（每次垃圾收集使用的时间越来越长，垃圾收集频率越来越高，每次垃圾收集清理掉的垃圾数据越来越少）、内存溢出导致系统崩溃，因此需要对JVM进行调优，使得程序在正常运行的前提下，获得更高的用户体验和运行效率。

这里有几个比较重要的指标：
1. 延迟：由于垃圾收集而引起的程序停顿时间。
1. 吞吐量：用户程序运行时间占用户程序和垃圾收集占用总时间的比值。
1. 内存占用：程序正常运行需要的内存大小。

当然，和 CAP 原则一样，同时满足一个程序内存占用小、延迟低、高吞吐量是不可能的，程序的目标不同，调优时所考虑的方向也不同，在调优之前，必须要结合实际场景，有明确的的优化目标，找到性能瓶颈，对瓶颈有针对性的优化，最后进行测试，通过各种监控工具确认调优后的结果是否符合目标。

## 调优工具

## 第三方调优工具
#### jps
jps 是 Java 自带的查看进程的命令，通过这个命令可以查看当前系统所有运行中的 Java 进程、Java 包名、jar 包名及 JVM 参数等。

```bash
# 查看 Java 进程概述
zhangqinghua$ jps
54592 Launcher
54593 IcebartechApplication
54340 
54597 Jps
54350 RemoteMavenServer36

# -q 只显示VM 标示，不显示jar,class, main参数等信息
zhangqinghua$ jps -q
55667
55653
55673

# -m 输出传递给main 方法的参数，在嵌入式jvm上可能是null
zhangqinghua$ jps -m
28715 Jps -m
23789 BossMain
23651 Resin -socketwait 32768 -stdout /data/aoxj/resin/log/stdout.log -stderr /data/aoxj/resin/log/stderr.log

# -l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
zhangqinghua$ jps -l
28729 sun.tools.jps.Jps
23789 com.asiainfo.aimc.bossbi.BossMain
23651 com.caucho.server.resin.Resin

# -v 输出传递给JVM的参数
zhangqinghua$ jps -v
23789 BossMain
28802 Jps -Denv.class.path=/data/aoxj/bossbi/twsecurity/java/trustwork140.jar
```

#### jmap
jmap（JVM Memory Map）命令用于生成 heap dump 文件，如果不使用这个命令，还可以使用 `-XX:+HeapDumpOnOutOfMemoryError` 参数来让虚拟机出现 OOM 的时候自动生成 dump 文件。 jmap 不仅能生成 dump 文件，还可以查询 finalize 执行队列、Java 堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

`-dump` 会将堆 dump 到文件，format 指定输出格式，live 指明是活着的对象，file 指定文件名：

```bash
zhangqinghua$ jmap -dump:live,format=b,file=dump.hprof 24971
Dumping heap to /usr/local/java/jdk1.7.0_79/dump.hprof ...
Heap dump file created
```

`-finalizerinfo` 打印等待回收的对象信息：

```bash
# Number of objects pending for finalization: 0 
# 说明当前 F-QUEUE 队列中并没有等待 Fializer 线程执行 finalizer 方法的对象
zhangqinghua$ jmap -finalizerinfo 24971
Attaching to process ID 24971, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.79-b02
Number of objects pending for finalization: 0
```

`-histo` 打印堆的对象统计，包括对象数、内存大小等等。`jmap -histo:live` 这个命令执行，JVM 会先触发 gc，然后再统计信息：

```bash
# jmap -histo:live 24971 | grep com.yuhuo 查询类名包含 com.yuhuo 的信息
# jmap -histo:live 24971 | grep com.yuhuo > histo.txt 保存信息到 histo.txt 文件
zhangqinghua$  jmap -histo:live 24971 | more
 #num     #instances         #bytes  class name
----------------------------------------------
   1:        100134       14622728  <constMethodKlass>
   2:        100134       12830128  <methodKlass>
   3:         88438       12708392  [C
   4:          8271       10163584  <constantPoolKlass>
   5:         27806        9115784  [B
   6:          8271        6225312  <instanceKlassKlass>
   7:          6830        5632192  <constantPoolCacheKlass>
   8:         86717        2081208  java.lang.String
   9:          2264        1311720  <methodDataKlass>
  10:         10880         870400  java.lang.reflect.Method
  11:          8987         869888  java.lang.Class
  12:         13330         747264  [[I
  13:         11808         733872  [S
  14:         20110         643520  java.util.concurrent.ConcurrentHashMap$HashEntry
  15:         18574         594368  java.util.HashMap$Entry
  16:          3668         504592  [Ljava.util.HashMap$Entry;
  17:         30698         491168  java.lang.Integer
  18:          2247         486864  [I
  19:          7486         479104  java.net.URL
  20:          8032         453616  [Ljava.lang.Object;
  21:         10259         410360  java.util.LinkedHashMap$Entry
  22:           699         380256  <objArrayKlassKlass>
  23:          5782         277536  org.apache.catalina.loader.ResourceEntry
  24:          8327         266464  java.lang.ref.WeakReference
  25:          2374         207928  [Ljava.util.concurrent.ConcurrentHashMap$HashEntry;
  26:          3440         192640  java.util.LinkedHashMap
  27:          4779         191160  java.lang.ref.SoftReference
  28:          3576         171648  java.util.HashMap
  29:         10080         161280  java.lang.Object
```

`-permstat` 打印 Java 堆内存的永久区的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。

```bash
zhangqinghua$ jmap -permstat 24971
Attaching to process ID 24971, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.79-b02
finding class loader instances ..done.
computing per loader stat ..done.
please wait.. computing liveness.......................liveness analysis may be inaccurate ...
class_loader    classes bytes   parent_loader   alive?  type
 
<bootstrap>   3034    18149440      null      live    <internal>
0x000000070a88fbb8  1   3048      null      dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070a914860  1   3064    0x0000000709035198  dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070a9fc320  1   3056    0x0000000709035198  dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070adcb4c8  1   3064    0x0000000709035198  dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070a913760  1   1888    0x0000000709035198  dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x0000000709f3fd40  1   3032      null      dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070923ba78  1   3088    0x0000000709035260  dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070a88fff8  1   3048      null      dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
0x000000070adcbc58  1   1888    0x0000000709035198  dead    sun/reflect/DelegatingClassLoader@0x0000000703c50b58
```

`-heap` 打印 heap 的概要信息，GC 使用的算法，heap 的配置及使用情况，可以用此来判断内存目前的使用情况以及垃圾回收情况：

```bash
zhangqinghua$ jmap -heap 24971
Attaching to process ID 24971, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.79-b02
 
using thread-local object allocation.
Parallel GC with 4 thread(s)
 
Heap Configuration:
   MinHeapFreeRatio = 0
   MaxHeapFreeRatio = 100
   MaxHeapSize      = 4146069504 (3954.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 21757952 (20.75MB)
   MaxPermSize      = 85983232 (82.0MB)
   G1HeapRegionSize = 0 (0.0MB)
 
Heap Usage:
PS Young Generation
Eden Space:
   capacity = 517996544 (494.0MB)
   used     = 151567520 (144.54605102539062MB)
   free     = 366429024 (349.4539489746094MB)
   29.26033421566612% used
From Space:
   capacity = 41943040 (40.0MB)
   used     = 0 (0.0MB)
   free     = 41943040 (40.0MB)
   0.0% used
To Space:
   capacity = 40370176 (38.5MB)
   used     = 0 (0.0MB)
   free     = 40370176 (38.5MB)
   0.0% used
PS Old Generation
   capacity = 115343360 (110.0MB)
   used     = 32927184 (31.401809692382812MB)
   free     = 82416176 (78.59819030761719MB)
   28.54709972034801% used
PS Perm Generation
   capacity = 85983232 (82.0MB)
   used     = 54701200 (52.16712951660156MB)
   free     = 31282032 (29.832870483398438MB)
   63.6184506300019% used
 
20822 interned Strings occupying 2441752 bytes.
```

`-F` 强制模式。如果指定的 pid 没有响应，请使用 `jmap -dump` 或 `jmap -histo` 选项。此模式下，不支持 live 子选项。



#### jconsole

#### Visual VM