---
title: JVM_07 参数配置

categories:
- Java 虚拟机

date: 2020-01-01 00:00:07
---
在虚拟机运行的过程中，如果可以跟踪系统的运行状态，那么对于问题的故障排查会有一定的帮助，为此，在虚拟机提供了一些跟踪系统状态的参数，使用给定的参数执行Java虚拟机，就可以在系统运行时打印相关日志，用于分析实际问题。我们进行虚拟机参数配置，其实就是围绕着堆、栈、方法区、进行配置。

|参数|描述|示例|
| :- |
|`-Xverify`|关闭大部分的类验证措施|`java -jar -Xverify:none xx.jar`|
||
|栈配置|
|`-Xss`|指定线程最大的栈空间大小，默认1m|`java -jar -Xss1m xx.jar`|
||
|堆配置|
|`-Xms`|启动时初始堆大小，一般跟下面相等|`java -jar -Xms256m xx.jar`|
|`-Xmx`|获得的最大堆大小，一般跟上面相等|`java -jar -Xmx1024m xx.jar`|
|`-Xmn`|新生代大小，默认为堆的25%|`java -jar -Xmn20m xx.jar`|
|`-XX:MaxPermSize`|设置老年代大小|`java -jar -XX:MaxPermSize=8 xx.jar`|
|`-XX:NewRatio`|老年代和新生代的比例|`java -jar -XX:NewRatio=2 xx.jar`|
|`-XX:SurvivorRatio`|新生代中Eden空间和From/To空间的比例|`java -jar -XX:SurvivorRatio=8 xx.jar`|
||
|收集器配置|
|`-XX:+UseSerialGC`|使用Serial收集器|`java -jar -XX:+UseSerialGC xx.jar`|
|`-XX:+UseParallelGC`|使用Parallel收集器|`java -jar -XX:+UseParallelGC xx.jar`|
|`-XX:+UseParalledlOldGC`|使用Parallel Old收集器|`java -jar -XX:+UseParalledlOldGC xx.jar`|
|`-XX:+UseConcMarkSweepGC`|使用并发收集器|`java -jar -XX:+UseConcMarkSweepGC xx.jar`|
||
|并行收集器设置|
|`-XX:ParallelGCThreads`|设置并行收集器收集时使用的CPU数。并行收集/线程数|`java -jar -XX:ParallelGCThreads=3 xx.jar`|
|`-XX:MaxGCPauseMillis`|设置并行收集最大暂停时间|`java -jar -XX:MaxGCPauseMillis=100 xx.jar`|
|`-XX:GCTimeRatio`|设置垃圾回收时间占程序运行时间的百分比.公式为1/(1+n)|`java -jar -XX:GCTimeRatio=18 xx.jar`|
||
|垃圾回收统计信息|
|`-XX:+PrintGC`|每次触发GC的时候打印相关日志|`java -jar -XX:+PrintGC xx.jar`|
|`-XX:+PrintGCDetails`|启动时控制台打印各个区的详细情况|`java -jar -XX:+PrintGCDetails xx.jar`|
|`-XX:+PrintGCTimeStamps`|||
|`-Xloggc:filename`|
||
|并发收集器设置|
|`-XX:+CMSIncrementalMode`|设置为增量模式，适用于单CPU情况|`java -jar -XX:+CMSIncrementalMode xx.jar`|
|`-XX:ParallelGCThreads`|Parallel回收并行处理的线程数，默认CPU核数|`java -jar -XX:+ParallelGCThreads=3 xx.jar`|
||
|溢出处理|
|`-XX:+HeapDumpOnOutOfMemoryError`|在内存溢出时导出整个堆信息|`java -jar -XX:HeapDumpOnOutOfMemoryError xx.jar`|
|`-XX:HeapDumpPath`|设置导出堆的存放路径，跟上面一起用|`java -jar -XX:HeapDumpPath=d:/Test03.dump xx.jar`|

默认情况下，`-Xms` 为物理电脑内存的的 1/64 大小，`-Xmx` 为物理电脑内存的的 1/4 大小。

通常情况下，将 `-Xms` 与 `-Xmx` 设置为同样的值，防止系统不断的扩容和释放内存，造成不必要的压力。

通常情况下，JVM 可用的堆内存要少于设定的容量，例如将 `-Xmx` 设置为 600m，实际可用的是 575m。这是因为 survivor0 和 survivor1 区同时只能用一个。

## 问题
JVM性能调优的方法和步骤，JVM的关键性核心参数配置

你熟悉的JVM调优参数，使用过哪些调优工具？

用过什么JVM调优命令？

JVM最常用的参数配置讲讲