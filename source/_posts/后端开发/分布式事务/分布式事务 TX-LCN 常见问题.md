---
title: 分布式事务 TX-LCN 常见问题

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:32
---
**ClassNotFoundException: org.springframework.data.jdbc.repository.config.JdbcRepositoryConfigExtension**
场景：参考官网上的demo，直接启动 tm-demo 失败。
原因：启动类放错地方，且也提示了报错 ** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.
解决：建个包，把启动类放进去，再启动就 OK 了。

