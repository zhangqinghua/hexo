---
title: JVM_06 垃圾回收器

categories:
- Java 虚拟机

date: 2020-01-01 00:00:06
---
垃圾回收算法是垃圾回收器的理论基础，而垃圾回收器是其具体实现。下面介绍 HotSpot 虚拟机提供的几种垃圾回收器。

## 吞吐量、停顿时间
吞吐量描述的是用户线程执行的时间比上全部运行时间（全部运行时间 = 用户线程时间 + 垃圾回收线程执行时间执行），吞吐量越高，表明系统的资源利用率越高。

停顿时间表示的是 GC 线程在执行过程中，导致用户线程停顿的时间，如果停顿时间越长，那么用户线程卡顿的时间越长，用户体验越差，因此我们希望的是停顿时间越短越好。

吞吐量和低延时（停顿时间）这两个指标是对立的，无法同时兼顾两者，如果想追求低延时，那么吞吐量就会下降；如果追求高吞吐量，那么停顿时间就会变长。不过随着目前垃圾回收器的不断发展，越来越多的垃圾回收器都是以「在保证高吞吐量的情况下，尽可能的去追求低延时」为原则，来进行垃圾回收器的实现。

> 另外还有一个指标就是「内存占用率」，因为垃圾回收器在执行过程中，它也需要占用一定的内存空间，当然我们期望的是内存占用率越小越好，尤其是在服务器内存配置较低的情况下。如果服务器的资源配置很高，内存很大，内存占用率高一点也可以接受。

## 串行、并行、并发
串行回收指的是单线程回收，全程 STW。停顿次数少，单次停顿时间长，吞吐量高。

并行回收指的是多线程回收，全程 STW。停顿次数少，单次停顿时间长，吞吐量高。

![](https://oscimg.oschina.net/oscnet/up-0704e3acfd751cb0c7fed5901cf73c9cccf.png)

并发回收指的是多线程分阶段回收，只有某阶段会 STW。停顿次数多，单次停顿时间短，吞吐量低。

![](https://oscimg.oschina.net/oscnet/up-2e760881402ee830c9cf6f415d3b9f9ccf8.png)

> 单核机器使用串行回收可以避免多线程切换，提高回收效率。

## Minor GC、Major GC、Full GC
Minor GC 是发生在年轻代的 GC。

Major GC 是发生在老年代的 GC。

Full GC 是全堆 GC，比如元数据区引起年轻代和老年代的回收。

## Serial/Serial Old
最古老的回收器，是一个单线程回收器，用它进行垃圾回收时，必须暂停所有用户线程。Serial 是针对新生代的回收器，采用复制算法；而 Serial Old 是针对老生代的回收器，采用 Mark-Compact 算法。优点是简单高效，缺点是需要暂停用户线程。

## ParNew
Seral/Serial Old 的多线程版本，使用多个线程进行垃圾回收。

## Parallel
Parallel Scavenge 是新生代的并行回收器，回收期间不需要暂停其他线程，采用 Copying 算法。该回收器与前两个回收器不同，主要为了达到一个可控的吞吐量。

## Parallel Old
Parallel Scavenge 的老生代版本，采用 Mark-Compact 算法和多线程。

## CMS
CMS 的全称是 Concurrent-Mark-Sweep 的缩写，翻译过来就是并发标记清除，它是一款「以低停顿时间为目标」的垃圾回收器，特点是低延时。它解决了老年代 GC 时出现长时间卡顿的问题。

CMS 的工作原理大致分为四个步骤：初始标记、并发标记、重新标记、并发清除。使用参数 `-XX:+UseConcMarkSweepGC` 即可开启使用 CMS 垃圾回收器。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9LNWNxaWEwdVY4R3pHNU41YWhlN0ZwSWlhYlhhaWNvc3NNSFYzUnlHQVdxd0MyVk1nQ3ZiaDdKNjBraWNpYXNkS0ZhZHFQSGhablRiZnBNeThxaWNoVk1xMG1WQS82NDA?x-oss-process=image/format,png)

在 JDK9 中，取消了 ParNew 与 Serial Old、Serial 与 CMS 的搭配组合，并且 CMS 被标记 `Deprecated`，在 JDK14 中 CMS 被彻底移除。

#### 特点
1. 只回收老年代和元数据区。
1. 使用预处理机制，使用内存空间达到一个阈值（默认 92%）就开始回收工作。CMS 的回收机制使得它需要在内存用尽前开始回收操作，否则会导致并发回收失败。

#### 步骤
1. 初始标记
    会导致 STW。标记老年代中所有的GC Roots对象，标记年轻代中活着的对象引用到的老年代的对象。
1. 并发标记 
    与用户线程同时工作。从“初始标记”阶段标记的对象开始找出所有存活的对象
1. 预清理阶段
    与用户线程同时工作。用来处理前一个阶段因为引用关系改变导致没有标记到的存活对象的，它会扫描所有标记为Direty的Card。
1. 可终止的预处理
    与用户线程同时工作。这个阶段尝试着去承担下一个阶段「重新标记」足够多的工作。这个阶段持续的时间依赖好多的因素，由于这个阶段是重复的做相同的事情直到发生aboart的条件（比如：重复的次数、多少量的工作、持续的时间等等）之一才会停止。
1. 重新标记
    会导致 STW。标记整个年老代的所有的存活对象。
1. 并发清除
    与用户线程同时工作。清除那些没有标记的对象并且回收空间。
1. 并发重置
    与用户线程同时工作。重新设置 CMS 算法内部的数据结构，准备下一个 CMS 生命周期的使用。
## G1
G1（Garbage First）回收器技术的前沿成果，是面向服务端的回收器，能充分利用 CPU 和多核环境。是一款并行与并发回收器，它能够建立可预测的停顿时间模型。



## ZCG

## 垃圾回收器组合
|Young|Tenured|JVM options|
| :-- |
|Serial|Serial Old|`-XX:+UseSerialGC`|
|Parallel Scavenge|Serial|`-XX:+UseParallelGC -XX:-UseParallelOldGC`|
|Parallel Scavenge|Parallel Old|`-XX:+UseParallelGC -XX:+UseParallelOldGC`|
|Parallel New或Serial|CMS|`-XX:+UseParNewGC` `-XX:+UseConcMarkSweepGC`|
|G1|G1|`-XX:+UseG1GC`|

## 垃圾回收器对比
待补充。。。

|回收器|范围|模式|算法|特点|吞吐量|停顿时间|备注|
| :- |
|Serial|Young|Client|复制|串行、STW||
|Serial Old|Old|Client、Server|标记-整理|串行、STW||
|ParNew|Young|Server|复制|并行、STW||
|Parallel|Young||复制|并行、STW|99%|
|Parallel Old|Old||标记-整理|并行、STW|99%|200ms|
|CMS|Old||标记-清除|并行、少量 STW||
|G1|Young、Old|Server|标记-整理|并行、少量 STW|97%|210ms|Java9 默认|
|ZGC||||并行、少量 STW|98%|2ms|实验性|

## 问题
GC 回收器有哪些？
经典 7 种。GMS

讲一下各种 GC 回收器的出现时间、原理、优缺点。

#### GMS
CMS 解决什么问题，说一下回收的过程。

CMS 回收停顿了几次，为什么要停顿两次。

CMS哪个阶段是并发的，哪个阶段是串行的？

#### G1
G1 的内存模型讲一下。

G1 回收器讲下回收过程。

G1 对 CMS 的有哪些改进？

G1 和 GMS 最大的区别在哪里？

GC、G1 和 ZGC 的区别。