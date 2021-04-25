---
title: 分布式事务 TX-LCN 快速入门

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:31
---
## Txlcn Client 配置
```yml
# springcloud feign 下开启负载均衡时的配置。开启后同一个事务组下相同的模块会重复调用。
# 对应dubbo框架下需要设置的是 @Reference的loadbalance，有下面四种，作用都是开启后同一个事务组下相同的模块会重复调用。
#txlcn_random=com.codingapi.txlcn.tracing.dubbo.TxlcnRandomLoadBalance
#txlcn_roundrobin=com.codingapi.txlcn.tracing.dubbo.TxlcnRoundRobinLoadBalance
#txlcn_leastactive=com.codingapi.txlcn.tracing.dubbo.TxlcnLeastActiveLoadBalance
#txlcn_consistenthash=com.codingapi.txlcn.tracing.dubbo.TxlcnConsistentHashLoadBalance

tx-lcn.ribbon.loadbalancer.dtx.enabled=true
# tx-manager 的配置地址，多个用,分割。注意设置上的地址在启动的时候会检查并连接，连接不成功会启动失败。
# tx-manager 下集群策略，当增加一个新的tx-manager后，tx-manager也会通知到其他的在线模块，然后tx-client会在连接上后面加入的模块。
tx-lcn.client.manager-address=127.0.0.1:8070,127.0.0.1:8071
# 该参数是分布式事务框架存储的业务切面信息。采用的是h2数据库。绝对路径。该参数默认的值为{user.dir}/.txlcn/{application.name}-{application.port}
tx-lcn.aspect.log.file-path=logs/.txlcn/demo-8080
# 调用链长度等级，默认值为3.标识调用连长度为3，该参数是用于识别分布式事务的最大通讯时间。
tx-lcn.client.chain-level=3
# 该参数为tc与tm通讯时的最大超时时间，单位毫米。该参数不需要配置会在连接初始化时由tm返回。
tx-lcn.client.tm-rpc-timeout=2000
# 该参数为分布式事务的最大时间，单位毫米。该参数不需要配置会在连接初始化时由tm返回。
tx-lcn.client.dtx-time=50000
# 该参数为雪花算法的机器编号。该参数不需要配置会在连接初始化时由tm返回。
tx-lcn.client.machine-id=1
#该参数为事务方法注解切面的orderNumber，默认值为0.
tx-lcn.client.dtx-aspect-order=0
#该参数为事务连接资源方法切面的orderNumber，默认值为0.
tx-lcn.client.resource-order=0
#是否开启日志记录。当开启以后需要配置对应logger的数据库连接配置信息。
tx-lcn.logger.enabled=false

#该参数为tm下的配置，tc下忽略
tx-lcn.client.tx-manager-delay=2000
#该参数为tm下的配置，tc下忽略
tx-lcn.client.tx-manager-heart=2000
```

#### 特别注意
微服务集群且用到 LCN 事务模式时，为保证性能请开启TX-LCN重写的负载策略：

```yml
# SpringCloud 开启 (application.properties)
tx-lcn.springcloud.loadbalance.enabled=true
```
关闭业务RPC重试：

```yml
# 关闭Ribbon的重试机制
ribbon.MaxAutoRetriesNextServer=0
```

为什么要关闭服务调用的重试。远程业务调用失败有两种可能： （1），远程业务执行失败 （2）、远程业务执行成功，网络失败。对于第2种，事务场景下重试会发生，某个业务执行两次的问题。 如果业务上控制某个事务接口的幂等，则不用关闭重试。

## Txlcn Manager 配置
```yml
spring.application.name=tx-manager
server.port=7970

#mysql 配置
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/tx-manager?characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=root
        
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.use-generated-keys=true


# TxManager Host Ip 默认为 127.0.0.1
tx-lcn.manager.host=127.0.0.1
# TxClient连接请求端口 默认为 8070
tx-lcn.manager.port=8070
# 心跳检测时间(ms) 默认为 300000
tx-lcn.manager.heart-time=300000
# 分布式事务执行总时间(ms) 默认为36000
tx-lcn.manager.dtx-time=36000
#参数延迟删除时间单位ms  默认为dtx-time值
tx-lcn.message.netty.attr-delay-time=36000
#事务处理并发等级 默认为128
tx-lcn.manager.concurrent-level=128

#后台登陆密码，默认值为codingapi
tx-lcn.manager.admin-key=codingapi
#分布式事务锁超时时间 默认为-1，当-1时会用tx-lcn.manager.dtx-time的时间
tx-lcn.manager.dtx-lock-time=-1
#雪花算法的sequence位长度，默认为12位.
tx-lcn.manager.seq-len=12
#异常回调开关
tx-lcn.manager.ex-url-enabled=false
# 事务异常通知（任何http协议地址。未指定协议时，为TxManager提供的接口）
tx-lcn.manager.ex-url=/provider/email-to/***@**.com



# 开启日志,默认为false
tx-lcn.logger.enabled=true
logging.level.com.codingapi=debug
#redis 的设置信息
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
```

#### 特别注意
1. TxManager 所有配置均有默认配置，请按需覆盖默认配置。
1. 特别注意 TxManager 进程会监听两个端口号，一个为 TxManager 端口，另一个是事务消息端口。TxClient 默认连接事务消息端口是 8070，所以，为保证 TX-LCN 基于默认配置运行良好，请设置 TxManager 端口号为8069 或者指定事务消息端口为 8070。
1. 分布式事务执行总时间 a 与 TxClient 通讯最大等待时间 b、TxManager 通讯最大等待时间 c、微服务间通讯时间 d、微服务调用链长度 e 几个时间存在着依赖关系。 a >= 2c + (b + c + d) * (e - 1), 特别地，b、c、d 一致时，a >= (3e-1)b。你也可以在此理论上适当在减小 a 的值，发生异常时能更快得到自动补偿，即 a >= (3e-1)b - Δ（原因）。 最后，调用链小于等于 3 时，将基于默认配置运行良好。
1. 若用 `tx-lcn.manager.ex-url=/provider/email-to/xxx@xx.xxx` 这个配置，配置管理员邮箱信息(如QQ邮箱)。

配置管理员邮箱信息：

```yml
spring.mail.host=smtp.qq.com
spring.mail.port=587
spring.mail.username=xxxxx@**.com
spring.mail.password=*********
```