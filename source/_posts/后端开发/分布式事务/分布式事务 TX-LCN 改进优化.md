---
title: 分布式事务 TX-LCN 改进优化

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:32
---
**关闭 TxManager 后，TxClient 一直处于连接不上的状态。即使重启 TxManager 也没有相应。**
1. 将 TxClient 的重试次数改成无限。