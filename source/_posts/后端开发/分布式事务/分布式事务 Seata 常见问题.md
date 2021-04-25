---
title: 分布式事务 Seata 常见问题

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:12
---

## 常见问题
**can not register RM,err:can not connect to services-server.**
场景：SpringCloud 引入 seata client 以来后，启动警告。
原因：没有启动 seata server。
解决：启动 seata server。

**Could not found property service.disableGlobalTransaction, try to use default value instead.**
场景：SpringCloud 引入 seata client 以来后，启动警告。
原因：没有配置或者seata 无法识别 `.yml` 的 `service.disableGlobalTransaction` 属性。
解决：使用 `.properties` 的配置文件或者另外创建 file.conf 文件写入 "service {disableGlobalTransaction: false}"。

**will connect to 127.0.0.1:8091**
场景：SpringCloud 引入 seata client 以来后，第一次启动能连接上本地 seata，第二次启动没法连接上本地 seata。
原因：？？
解决：关闭 seata server 命令行，重新打开运行 seata server。


正常情况，Seata 日志：

```
16:48:50.705  INFO --- [ettyServerNIOWorker_1_4_8] i.s.c.r.processor.server.RegTmProcessor  : TM register success,message:RegisterTMRequest{applicationId='easybyte-log', transactionServiceGroup='easybyte-log'},channel:[id: 0xfe8578f6, L:/127.0.0.1:8091 - R:/127.0.0.1:49393],client version:1.0.0
16:48:50.714  INFO --- [rverHandlerThread_1_6_500] i.s.c.r.processor.server.RegRmProcessor  : RM register success,message:RegisterRMRequest{resourceIds='jdbc:mysql://47.119.139.41:3306/easybyte', applicationId='easybyte-log', transactionServiceGroup='easybyte-log'},channel:[id: 0x0ab655bf, L:/127.0.0.1:8091 - R:/127.0.0.1:49392],client version:1.0.0
```

异常情况，Seata 日志：

```
16:47:19.896  INFO --- [ettyServerNIOWorker_1_1_8] i.s.c.r.n.AbstractNettyRemotingServer    : channel:[id: 0x5b47e626, L:/127.0.0.1:8091 - R:/127.0.0.1:49374] read idle.
16:47:19.896  INFO --- [ettyServerNIOWorker_1_2_8] i.s.c.r.n.AbstractNettyRemotingServer    : channel:[id: 0xd2688132, L:/127.0.0.1:8091 - R:/127.0.0.1:49375] read idle.
16:47:19.899  INFO --- [ettyServerNIOWorker_1_1_8] i.s.c.r.n.AbstractNettyRemotingServer    : 127.0.0.1:49374 to server channel inactive.
16:47:19.899  INFO --- [ettyServerNIOWorker_1_2_8] i.s.c.r.n.AbstractNettyRemotingServer    : 127.0.0.1:49375 to server channel inactive.
16:47:19.899  INFO --- [ettyServerNIOWorker_1_1_8] i.s.c.r.n.AbstractNettyRemotingServer    : remove unused channel:[id: 0x5b47e626, L:/127.0.0.1:8091 - R:/127.0.0.1:49374]
16:47:19.899  INFO --- [ettyServerNIOWorker_1_2_8] i.s.c.r.n.AbstractNettyRemotingServer    : remove unused channel:[id: 0xd2688132, L:/127.0.0.1:8091 - R:/127.0.0.1:49375]
16:47:19.914  INFO --- [ettyServerNIOWorker_1_2_8] i.s.c.r.n.AbstractNettyRemotingServer    : closeChannelHandlerContext channel:[id: 0xd2688132, L:/127.0.0.1:8091 - R:/127.0.0.1:49375]
16:47:19.914  INFO --- [ettyServerNIOWorker_1_1_8] i.s.c.r.n.AbstractNettyRemotingServer    : closeChannelHandlerContext channel:[id: 0x5b47e626, L:/127.0.0.1:8091 - R:/127.0.0.1:49374]
16:47:19.916  INFO --- [ettyServerNIOWorker_1_2_8] i.s.c.r.n.AbstractNettyRemotingServer    : 127.0.0.1:49375 to server channel inactive.
16:47:19.916  INFO --- [ettyServerNIOWorker_1_1_8] i.s.c.r.n.AbstractNettyRemotingServer    : 127.0.0.1:49374 to server channel inactive.
16:47:19.916  INFO --- [ettyServerNIOWorker_1_2_8] i.s.c.r.n.AbstractNettyRemotingServer    : remove unused channel:[id: 0xd2688132, L:/127.0.0.1:8091 ! R:/127.0.0.1:49375]
16:47:19.916  INFO --- [ettyServerNIOWorker_1_1_8] i.s.c.r.n.AbstractNettyRemotingServer    : remove unused channel:[id: 0x5b47e626, L:/127.0.0.1:8091 ! R:/127.0.0.1:49374]
```

2021-04-22: 发现是没有加上 `@GlobalTransactional` 注解引起的。第一个服务加了 `@GlobalTransactional` 注解，第二个服务没有加，结果第二个服务一直启动不了。
2021-04-22: 又发现不是这个问题。