---
title: Chapter 1.4 三种方案对比

categories:
- 分布式

date: 2020-05-18 00:00:14
---
上面几种方式，哪种方式都无法做到完美。就像 CAP 一样，在复杂性、可靠性、性能等方面无法同时满足，所以，根据不同的应用场景选择最适合自己的才是王道。

从理解的难易程度角度（从低到高）：数据库 > Redis > Zookeeper。

从实现的复杂性角度（从低到高）：Zookeeper >= Redis > 数据库。

从性能角度（从高到低）：Redis > Zookeeper >= 数据库。

从可靠性角度（从高到低）：Zookeeper > Redis > 数据库。