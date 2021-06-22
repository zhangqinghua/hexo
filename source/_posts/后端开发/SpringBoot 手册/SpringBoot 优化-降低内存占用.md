---
title: SpringBoot 优化-降低内存占用
categories:
- 后端开发
- SpringBoot 手册

date: 2021-05-23
---
参考：[微服务中使用 OpenJ9 JVM 内存占用降60%(相对HotSpot)](https://cloud.tencent.com/developer/article/1489112)

参考：[没想到Spring Boot居然这么耗内存，有点惊讶](https://www.cnblogs.com/hulianwangjiagoushi/p/11672839.html)

参考：[Java微服务内存占用分析](https://blog.csdn.net/lgxzzz/article/details/103220202)

我这边测试是降低20%左右，一个 314MB 的微服务，可以降到 250MB 左右。
