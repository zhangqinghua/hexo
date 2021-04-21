---
title: SpringCloud Nacos

categories:
- SpringCloud
date: 2021-02-17 00:00:01
---

## 附：Docker 安装 Nacos
#### 创建数据库
在 MySQL 创建 Nocas 数据库 nacos_cofig。

然后执行 SQL 文件：https://github.com/alibaba/nacos/blob/master/config/src/main/resources/META-INF/nacos-db.sql

> Nacos1 只支持 MySQL8 以下的版本。

#### 拉取镜像
```bash
zhangqinghua$ docker pull nacos/nacos-server
...

zhangqinghua$ docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
nacos/nacos-server           latest              9c0b55a5ab2c        4 weeks ago         935MB
```

#### 启动容器
```bash
zhangqinghua$ docker run -d -p 8848:8848  \
                         -e MODE=standalone \
                         -e PREFER_HOST_MODE=hostname \
                         -e SPRING_DATASOURCE_PLATFORM=mysql \
                         -e MYSQL_SERVICE_HOST=127.0.0.1 \
                         -e MYSQL_SERVICE_PORT=3306 \
                         -e MYSQL_SERVICE_DB_NAME=nacos_config \
                         -e MYSQL_SERVICE_USER=root \
                         -e MYSQL_SERVICE_PASSWORD=123456 \
                         -e MYSQL_DATABASE_NUM=1 \
                         --restart always --name mynacos nacos/nacos-server
```

|配置项|描述|可选参数|默认值|
| :- |
|MODE|模式（集群、单机）|cluster / standalone|cluster|
|NACOS_SERVERS|nacos cluster地址|eg. ip1, ip2, ip3||
|NACOS_SERVER_IP|多网卡下的自定义nacos服务器IP|||
|PREFER_HOST_MODE|是否支持 hostname|hostname / ip|ip|
|NACOS_SERVER_PORT|服务端口号||8848|
|SPRING_DATASOURCE_PLATFORM|单机模式支持 MySQL|mysql / empty|empty|
|MYSQL_DATABASE_NUM|数据库数量||2|
|MYSQL_MASTER_SERVICE_HOST|MySQL 主节点 host|||
|MYSQL_MASTER_SERVICE_PORT|MySQL 主节点 port||3306|
|MYSQL_MASTER_SERVICE_DB_NAME|MySQL 主节点数据库名|||
|MYSQL_MASTER_SERVICE_USER|MySQL 主节点用户名|||
|MYSQL_MASTER_SERVICE_PASSWORD|MySQL 主节点密码|||
|MYSQL_SLAVE_SERVICE_HOST|MySQL 从节点 host|||
|MYSQL_SLAVE_SERVICE_PORT|MySQL 从节点 port||3306|
|JVM_XMS|-Xms||2G|
|JVM_XMX|-Xmx||2G|
|JVM_XMN|-Xmx||2G|
|JVM_MS|-XX:MetaspaceSize||128M|
|JVM_MMS|-XX:MaxMetaspaceSize||320M|
|NACOS_DEBUG|开启远程调试|y / n|n|
|TOMCAT_ACCESSLOG_ENABLED|server.tomcat.accesslog||false|
#### 查看启动日志
```bash
zhangqinghua$ docker logs -f mynacos
```

#### 访问 Nacos
访问地址：http://localhost:8848/nacos
账号密码：nacos / nacos

## 附：常见问题
**Docker 部署 Nacos 报错：java.lang.IllegalStateException: No DataSource set**
原因：Nacos1 不支持 MySQL8。
解决：1.切换 5.7 版本 2.修改 Nacos1 源码 3.使用Nacos2（目前没有 Dcoker 版本）

**启动报错：Caused by: java.net.UnknownHostException: jmenv.tbsite.net**
原因：启动模式没设置导致的。
解决：`sh startup.sh -m standalone`。

启动报错：Referenced from: /private/var/folders/wz/7_0z_zvj7_j8dsrm7cg6yz2w0000gn/T/librocksdbjni6447145289520783579.jnilib (which was built for Mac OS X 10.15)
原因：MacOS 版本过低。
解决：升级 MacOS 或使用 Docker。