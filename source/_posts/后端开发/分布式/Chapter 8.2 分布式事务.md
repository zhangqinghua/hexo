---
title: Chapter 8.2 分布式事务

categories:
- 后端开发
- MySQL 性能调优

date: 2020-04-28 00:00:82
---

## CAP 理论
CAP 原则是指在一个分布式系统中，一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）三者不可兼得。
1. 一致性：在分布式系统的所有数据备份中，在同一时刻是否有同样的值（等同于所有节点都访问同一份最新的数据副本）。
1. 可用性：在集群中一部分节点发生故障后，集群整体能否响应客户端的读写请求（对数据更新具备高可用性）。
1. 分区容错性：系统如果不能在时限内达成数据的一致性，就意味着发生了分区，必须就当前操作的 C 和 A 之间做出选择。以实际效果而言，分区相当于对通信的时限要求。

## BASE 理论
BASE 理论是 CAP 理论的验证，包括基本可用性（Basically Available）、柔性状态（Soft State）、最终一致性（Eventual Consistency）三个原则。

## 两阶段提交协议
分布式事务指涉及操作多个数据库的事务，在分布式系统中，各个节点之间在物理上相互独立，通过网络进行沟通和协调。

二阶段提交（Two-Phase Commit）指在计算机网络及数据库领域内，为了使分布式数据库的所有节点在进行事务提交时都保持一致性而设计的一种算法。在分布式系统中，每个节点虽然都可以知道自己的操作是否成功，却无法知道其它节点的操作是否成功。

在一个事务跨越多个节点时，为了保持事务的 ACID 特性，需要引入一个作为协调者的组件来统一掌控所有节点（称作参与者）的操作结果，并最终确认这些节点是否真正提交操作结果（比如将更新后的数据写入磁盘等）。因此，二阶段提交的算法思路可以概括为：参与者将操作成败通知协调者，再由协调者根据所有参与者的反馈决定各参与者是提交操作还是终止操作。

#### Prepare（准备阶段）
事务协调者（事务管理器）给每个参与者（源管理器）都发送 Prepare 消息，每个参与者要么直接返回失败（如权限验证失败），要么在本地执行事务，写本地的 redo 和 undo 日志但不提交，是一种“万事俱备，只欠东风”的状态。

#### Commit（提交阶段）
如果协调者接收到了参与者的失败消息或者等待超时，则直接给每个参与者都发送回滚消息，否则发送提交消息，参与者根据协调者的指令执行提交或者回滚操作，释放在所有事务处理过程中使用的锁资源。

![](http://www.pbteach.com/post/java_distribut/subject_dtx-02/1563273133485.png)

#### 两阶段提交的缺点
两阶段提交的缺点如下。
1. 同步阻塞问题：在执行过程中，所有参与者的任务都是阻塞执行的。
1. 单点故障问题：所有请求都需要经过协调者，在协调者发生故障时，所有参与者都会被阻塞。
1. 数据不一致：在二阶段提交的第 2 阶段，在协调者想参与者发送 Commmit 请求后发生了局部网络异常，或者在发送 Commit 请求过程中协调者发生了故障，导致只有一部分参与者接收到 Commit 请求，于是整个分布式系统出现了数据不一致的现象，这也被成为脑裂。
1. 协调者宕机后事务状态丢失：协调者在发出 Commit 消息之后宕机，唯一接收到这条消息的参与者也宕机，即使协调者通过选择产生了新的协调者，这条事务的状态也是不确定的，没有人知道事务是否已被提交。

## 三阶段提交协议
三阶段提交（Tree-Phase Commit）是二阶段提交的改进版本，具体改进如下：
1. 引入超时机制：在协调者和参与者中引入了超时机制，如果协调者长时间接收不到参与者的反馈，则认为参与者执行失败。
1. 在第 1 阶段和第 2 阶段都加入了一个预准备阶段以保证在最后的任务提交之前各参与者的状态是一致的。也就是说，除了引入超时机制，三阶段提交协议把两阶段提交协议的准备阶段再次一分为二，这样三阶段提交就有了 CanCommmit、PreCommit、DoCommit 三个阶段。

#### CanCommit 阶段
协调者向参与者发送 Commit 请求，参与者如果可提交就返回 Yes 响应，否则返回 No 响应。

#### PreCommit 阶段
协调者根据参与者的反应来决定是否继续进行，有以下两种可能。
1. 假如协调者从所有参与者那里获得的反馈都是 Yes 响应，就预执行事务。
1. 假如有任意参与者向协调者发送了 No 响应，或者在等待超时之后协调者都没有接收到参与者的响应，则执行事务的中断。

#### DoCommit 阶段
该阶段进行真正的事务提交，主要包括：协调者发送提交请求，参与者提交事务，参与者响应反馈（在事务提交完之后向协调者发送 Ack 响应），协调者确定完成事务。

![](https://tse3-mm.cn.bing.net/th/id/OIP.f9Y1UcT76nyk3o04kJ_ndQAAAA?pid=Api&rs=1)

## 柔性事务
在分布式数据库领域，基于 CAP 理论及 BASE 理论，阿里巴巴提出了柔性事务的概念。

我们通常所说的柔性事务分为：两阶段型、补偿型、异步确保型、最大努力通知型。

两阶段事务指分布式事务的两阶段提交，对应技术上的 XA 和 JTA/JTS，是分布式环境下事务处理的典型规范模式。

TCC 型事务（Try、Confirm、Cancel）为补偿型事务，是一种基于补偿的事务处理模型。如下图所示，服务器 A 发起事务，服务器 B 参与事务，如果服务器 A 的事务和服务器 B 的事务都顺利执行完成并提交，则整个事务执行完成。但是，如果事务 B 执行失败，事务 B 本身就回滚，这时事务 A 已被提交，所以需要执行一个补偿操作，将已经提交的事务 A 执行的操作进行回滚操作，恢复到未执行前事务 A 的状态。需要注意的是，发起提交的一般是主业务服务，而状态补偿的一般是业务活动管理者，因为活动日志被存储在业务活动管理中，补偿需要依靠日志进行恢复。TCC 事务模型牺牲了一定的隔离性和一致性，但是提高了事务的可用性。

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=453289732,866507005&fm=26&gp=0.jpg)

异步确保型事务指将一系列同步的事务操作修改为基于消息队列异步执行的操作，来避免分布式事务中同步阻塞带来的数据操作性能下降。如下图所示，在写业务数据 A 触发后将执行以下流程：
1. 业务 A 的模块在数据库 A 上执行数据更新操作。
1. 业务 A 调用写消息数据模块。
1. 写消息日志模块将数据库的写操作写入数据库 A 中。
1. 读消息日志模块接收操作日志。
1. 读消息数据调用写业务 B 的模块。
1. 写业务 B 更新数据到数据库 B。
1. 写业务数据 B 的模块发送异步消息更新到数据库 A 中的写消息日志状态，说明自己已经完成了异步数据更新操作。

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1591804156048&di=3595c8ccd553796b0f88856d17459242&imgtype=0&src=http%3A%2F%2Fimg1.imgtn.bdimg.com%2Fit%2Fu%3D2000960586%2C3622433777%26fm%3D214%26gp%3D0.jpg)

最大努力通知型事务也是通过消息中间