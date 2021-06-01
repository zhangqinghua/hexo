---
title: 分布式事务 Seata Server 部署

categories:
- 后端开发
- 分布式事务

date: 2021-03-16 00:00:11
---
## 直接部署
#### 下载
https://github.com/seata/seata/releases

#### 配置

#### 启动
```bash
zhangqinghua$ sh ./bin/seata-server.sh
```

## 容器部署

## Nacos 启动
#### 上传配置至 Nacos 配置中心
**1. 下载配置文件**

分别下载 [nacos-config.sh](https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh) 和 [config.txt](https://github.com/seata/seata/blob/develop/script/config-center/config.txt)。

`config.txt` 就是 Seata 各种详细的配置，执行 `nacos-config.sh` 即可将这些配置导入到 Nacos，这样就不需要将 `file.conf` 和 `registry.conf` 放到我们的项目中了，需要什么配置就直接从 Nacos 中读取。

**2. 同步参数到配置中心**

```bash
# -h: Nacos host，默认值 localhost
# -p: Nacos port，默认值 8848
# -g: Nacos 配置分组，默认值为 'SEATA_GROUP'
# -t: Nacos 租户信息，对应 Nacos 的命名空间ID字段, 默认值为空 ''
zhangqinghua$ sh nacos-config.sh -h 120.78.223.168 -p 31848 -g SEATA_GROUP -t 150dfa3b-2681-4190-9f03-d0a09e255539
set nacosAddr=120.78.223.168:31848
set group=SEATA_GROUP
Set transport.type=TCP successfully 
...
Set metrics.exporterList=prometheus successfully 
Set metrics.exporterPrometheusPort=9898 successfully 
=========================================================================
 Complete initialization parameters,  total-count:93 ,  failure-count:4 
=========================================================================
 init nacos config fail. 
```

**3. 效果查看**

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210531170943.png)

#### 数据库配置
**1. Seata Server 建表**
给 Seata Server 的数据库建表。

表文件：https://github.com/seata/seata/blob/develop/script/server/db/mysql.sql

**2. Seata Client 建表**

在每个 Seata Client 数据库上建表。

```
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'increment id',
  `branch_id` bigint(20) NOT NULL COMMENT 'branch transaction id',
  `xid` varchar(100) NOT NULL COMMENT 'global transaction id',
  `context` varchar(128) NOT NULL COMMENT 'undo_log context,such as serialization',
  `rollback_info` longblob NOT NULL COMMENT 'rollback info',
  `log_status` int(11) NOT NULL COMMENT '0:normal status,1:defense status',
  `log_created` datetime NOT NULL COMMENT 'create datetime',
  `log_modified` datetime NOT NULL COMMENT 'modify datetime',
  `ext` varchar(100) DEFAULT NULL COMMENT 'reserved field',
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='AT transaction mode undo table';
```

#### Seata Client 配置
**1. SpringCloud 服务引入 Seata Client 依赖**

```xml
<dependency>
   <groupId>com.alibaba.cloud</groupId>
   <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
   <exclusions>
         <!--跟 BeetSQL 集合，排除冲突的依赖-->
         <exclusion>
            <groupId>org.ow2.asm</groupId>
            <artifactId>asm</artifactId>
         </exclusion>
         <!--Seata Client 版本需要和 Seata Server 一致-->
         <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-spring-boot-starter</artifactId>
         </exclusion>
   </exclusions>
</dependency>
<!--Seata Client 版本需要和 Seata Server 一致-->
<dependency>
   <groupId>io.seata</groupId>
   <artifactId>seata-spring-boot-starter</artifactId>
   <version>1.4.2</version>
</dependency>
```

**2. 配置 yml 信息**