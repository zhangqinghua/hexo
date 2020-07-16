---
title: 安装 MySQL

categories:
- Docker 笔记

date: 2020-07-08
---

```bash
[root@vultrguest ~]# docker pull mysql
...
[root@vultrguest ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              be0dbf01a0f3        4 weeks ago         541MB

# 这里顺便设置了密码
[root@vultrguest ~]# docker run --name mysql01 -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

# 进入MySQL容器
[root@vultrguest ~]# docker exec -it mysql01 bash
```

或者：

```bash
# 如果要挂载 MySQL 配置文件的话，我们必须在物理机上存在着该配置文件。
docker run --name mysql01\
-p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 \
-v /data/docker/mysql/conf:/etc/mysql/conf.d \
-v /data/docker/mysql/data:/var/lib/mysql \
-v /data/docker/mysql/files:/var/lib/mysql-files \
mysql
```

## 常见问题
1. Docker 启动 MySQL 闪退
通常是内存不足。MySQL 最低需要 500MB 的内存启动。

1. MySQL8.0 启动失败
报错：Error on realpath() on '/var/lib/mysql-files' No such file or directory。

这是因为当指定了外部配置文件与外部存储路径时，也需要指定 /var/lib/mysql-files的外部目录。

```bash
docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /data/docker/mysql/conf/mysql/conf:/etc/mysql -v /data/docker/mysql/data/mysql/files:/var/lib/mysql-files  mysql
```

1. 无法挂载配置文件
宿主机没有配置文件，然后容器也没有自动生成配置文件。

根据官网说明：如果要挂载 MySQL 配置文件的话，我们必须在物理机上存在着该配置文件。

```bash

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
symbolic-links=0
!includedir /etc/mysql/conf.d/
```