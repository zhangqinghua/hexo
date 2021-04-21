---
title: Docker 镜像

categories:
- 部署运维
- Docker 教程

date: 2020-07-08
---

## 附：镜像安装 MySQL
#### 搜索镜像
```bash
zhangqinghua$ docker search mysql
```

#### 拉取 MySQL 8.0
```bash
zhangqinghua$ docker pull mysql:8.0
8.0: Pulling from library/mysql
80369df48736: Pull complete 
e8f52315cb10: Pull complete 
cf2189b391fc: Pull complete 
cc98f645c682: Pull complete 
27a27ac83f74: Pull complete 
fa1f04453414: Pull complete 
d45bf7d22d33: Pull complete 
3dbac26e409c: Pull complete 
9017140fb8c1: Pull complete 
b76dda2673ae: Pull complete 
bea9eb46d12a: Pull complete 
e1f050a38d0f: Pull complete 
Digest: sha256:7345ce4ce6f0c1771d01fa333b8edb2c606ca59d385f69575f8e3e2ec6695eee
Status: Downloaded newer image for mysql:8.0
docker.io/library/mysql:8.0
```

#### 查看镜像
```bash
zhangqinghua$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               8.0                 c8ee894bd2bd        3 weeks ago         456MB
```

#### 运行镜像
```bash
# --name 			指定容器名称
# -p 3306:3306			将宿主机的3306端口映射到容器的3306端口上
# -e MYSQL_ROOT_PASSWORD 	重置MYSQL ROOT的密码
# -d  			    	指定镜像
zhangqinghua$ docker run --name=beesgo_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0
c82e078ba80ce959cec46f3f615339b618ccdadc5f3623bff0abaf7f03b6cf6e
```

#### 查看容器
```bash
zhangqinghua$ docker ps -a
CONTAINER ID    IMAGE        CREATED          PORTS                               NAMES
5555f130af86    mysql:8.0    5 seconds ago    0.0.0.0:3306->3306/tcp, 33060/tcp   beesgo_mysql
```

#### 进入容器
```bash
zhangqinghua$ docker exec -it 5555f130af86 /bin/bash
root@5555f130af86:/$ 
```

#### 登陆 MySQL
```bash
root@5555f130af86:/$ mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.18 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

#### 创建数据库和用户
```bash
mysql> set global validate_password.policy=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password.length=4;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE DATABASE beesgo CHARACTER SET utf8mb4;
Query OK, 1 row affected (0.01 sec)

mysql> CREATE USER 'beesgo'@'%' IDENTIFIED BY 'beesgo';
Query OK, 0 rows affected (0.01 sec)

mysql> grant all privileges on beesgo.* to 'beesgo'@'%' with grant option;
Query OK, 0 rows affected (0.00 sec)

mysql> quit;
Bye
```

#### 退出容器
```bash
CTRL + P + Q
root@5555f130af86:/# read escape sequence
zhangqinghua$ 
```

#### 停掉容器
```bash
zhangqinghua$ docker stop beesgo_mysql
beesgo_mysql
```

#### 删除容器
```bash
zhangqinghua$ docker rm  beesgo_mysql
beesgo_mysql
```

#### 数据库不适合运行在 Docker 中，为什么？
1. 数据不安全
2. 数据库的网络的要求高
3. 运行数据库的时候需要一些硬件环境的支持
1. Docker 中打包的应用应该是无状态的应用

## 附：镜像安装 Redis
#### 拉取镜像
```bash
[root@izwz9hxps1f8ad2fzfttnbz ~]$ docker pull redis
Using default tag: latest
latest: Pulling from library/redis
8d691f585fa8: Pull complete 
8ccd02d17190: Pull complete 
4719eb1815f2: Pull complete 
200531706a7d: Pull complete 
eed7c26916cf: Pull complete 
e1285fcc6a46: Pull complete 
Digest: sha256:fe80393a67c7058590ca6b6903f64e35b50fa411b0496f604a85c526fb5bd2d2
Status: Downloaded newer image for redis:latest
```

#### 运行镜像
```bash
# --name	设置容器的名称
# -p		映射6379端口
# -d		指定镜像
# redis-server	创建容器成功后，要运行的命令，这里是启动redis-server

[root@izwz9hxps1f8ad2fzfttnbz ~]$ docker run --name=beesgo-redis -p 6379:6379 -d redis:latest redis-server
8c26678baf89ab9f1b1cee94cc47411f20dbab6b23a8854558711f917399f4db
```

## 附：镜像安装 Nginx
## 附：镜像安装 Tomcat