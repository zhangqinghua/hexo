---
title: JVM 监控工具

categories:
- Java Virtual Machine Specification

tags:
- JVM

date: 2020-01-21
---

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
S0C    S1C     S0U     S1U   EC       EU        OC         
26112.0 24064.0 6562.5  0.0   564224.0 76274.5   434176.0   

OU        PC       PU         YGC    YGCT    FGC    FGCT     GCT   
388518.3  524288.0 42724.7    320    6.417   1      0.398    6.815

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

[参考](https://www.cnblogs.com/ityouknow/p/5714703.html)