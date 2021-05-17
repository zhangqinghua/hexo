---
title: 分布式事务 TX-LCN 常见问题

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:32
---
## TxLCN TM 监听IP 坑。
**1. 指定监听 IP**
TM 的默认监听 IP 是 127.0.0.1。这意味着 TC 只能使用 127.0.0.1 去访问，无法远程调用。
```yml
# TM 配置
tx-lcn.manager.host: 127.0.0.1
tx-lcn.manager.port: 8070

# TC 配置
tx-lcn.client.manager-address: 127.0.0.1:31080
```

**2. 模糊监听 IP**
根据官方文档描述，如果将 IP 设置为 0.0.0.0，既可以远程访问。

```yml
# TM 配置
tx-lcn.manager.host: 0.0.0.0
tx-lcn.manager.port: 8070

# TC 配置
tx-lcn.client.manager-address: 47.119.139.41:31080
```

但是这会出现一个 Bug，TC 先请求一遍 47.119.139.41:31080，再请求一遍 0.0.0.0:31080。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210517172712.png)

**3. 去掉监听 IP**
最后的方案是去掉监听 IP，这样子既可以远程访问，又不会去请求 0.0.0.0:31080。

```yml
# TM 配置
tx-lcn.manager.host: 
tx-lcn.manager.port: 8070

# TC 配置
tx-lcn.client.manager-address: 47.119.139.41:31080
```

#### 常见问题
**ClassNotFoundException: org.springframework.data.jdbc.repository.config.JdbcRepositoryConfigExtension**
场景：参考官网上的demo，直接启动 tm-demo 失败。
原因：启动类放错地方，且也提示了报错 ** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.
解决：建个包，把启动类放进去，再启动就 OK 了。

## TxLCN 
**TxLCN Client 一直无法连上 TxLCN manager**
