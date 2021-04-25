---
title: 分布式事务 Seata 介绍

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:10
---
Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

## Seata 术语
**TC (Transaction Coordinator) - 事务协调者**
维护全局和分支事务的状态，驱动全局事务提交或回滚。

**TM (Transaction Manager) - 事务管理器**
定义全局事务的范围：开始全局事务、提交或回滚全局事务。

**RM (Resource Manager) - 资源管理器**
管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

## TCC 模式
回顾总览中的描述：一个分布式的全局事务，整体是 两阶段提交 的模型。全局事务是由若干分支事务组成的，分支事务要满足 两阶段提交 的模型要求，即需要每个分支事务都具备自己的：
1. 一阶段 prepare 行为。
1. 二阶段 commit 或 rollback 行为。

![](https://img.alicdn.com/tfs/TB14Kguw1H2gK0jSZJnXXaT1FXa-853-482.png)

根据两阶段行为模式的不同，我们将分支事务划分为 Automatic (Branch) Transaction Mode 和 Manual (Branch) Transaction Mode。

AT 模式（参考链接 TBD）基于支持本地 ACID 事务的关系型数据库：
1. 一阶段 prepare 行为：在本地事务中，一并提交业务数据更新和相应回滚日志记录。
1. 二阶段 commit 行为：马上成功结束，自动异步批量清理回滚日志。
1. 二阶段 rollback 行为：通过回滚日志，自动生成补偿操作，完成数据回滚。

相应的，TCC 模式，不依赖于底层数据资源的事务支持：
1. 一阶段 prepare 行为：调用 自定义 的 prepare 逻辑。
1. 二阶段 commit 行为：调用 自定义 的 commit 逻辑。
1. 二阶段 rollback 行为：调用 自定义 的 rollback 逻辑。

所谓 TCC 模式，是指支持把自定义的分支事务纳入到全局事务的管理中。

## Saga 模式
Saga 模式是 Seata 提供的长事务解决方案，在 Saga 模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210422164520.png)

理论基础：Hector & Kenneth 发表论⽂ Sagas （1987）。

适用场景：
1. 业务流程长、业务流程多
1. 参与者包含其它公司或遗留系统服务，无法提供 TCC 模式要求的三个接口

优缺点：
1. 一阶段提交本地事务，无锁，高性能
1. 事件驱动架构，参与者可异步执行，高吞吐
1. 补偿服务易于实现
1. 不保证隔离性