---
title: 分布式事务 TX-LCN 改进优化

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:32
---
**关闭 TxManager 后，TxClient 一直处于连接不上的状态。即使重启 TxManager 也没有相应。**
1. 将 TxClient 的重试次数改成无限。

```java
/**
* TxLCN 断开无限重试
* 1. 默认配置下TxManager下线后，TxClient会重试6次重连。如果6次都没有连上TxManager，则不再进行尝试。
*    这时即使重启了TxManager，也需要重启TxClient。
* 2. 所以我希望TxClient能够无限尝试重连TxManager。
* 3. 设置次数，14400 * 30 = 一个月
*/
@Bean
@Primary
public RpcConfig rpcConfig() {
   RpcConfig rpcConfig = new RpcConfig();
   rpcConfig.setReconnectCount(14400 * 30);
   return rpcConfig;
}
```