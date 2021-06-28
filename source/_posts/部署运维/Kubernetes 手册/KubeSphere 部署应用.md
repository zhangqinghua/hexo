---
title: KubeSphere 部署应用

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13 07:00:07
---

## KubeSphere 部署 Nacos

## KubeSphere 部署 Nacos

## KubeSphere 部署 MySQL 集群
```conf
# MySQL Master 基本配置
[client]
default-character-set=utf8mb4
[mysql]
default-character=set=utf8mb4
[mysql]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collaction-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

# MySQL Master 主从复制
server_id=1
log-bin=mysql-bin
read-only=0

binlog-do-db=easybyte_order

replicate-ignore-db=sys
replicate-ignore-db=mysql
replicate-ignore-db=infomation-schema
replicate-ignore-db=performance-schema


# MySQL 配置
[client]
default-character-set=utf8mb4
[mysql]
default-character=set=utf8mb4
[mysql]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collaction-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

# MySQL Slave 主从复制
server_id=2
log-bin=mysql-bin
read-only=1

biglog-do-db=easybyte-order

replicate-ignore-db=sys
replicate-ignore-db=mysql
replicate-ignore-db=infomation-schema
replicate-ignore-db=performance-schema
```