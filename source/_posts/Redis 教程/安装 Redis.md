---
title: 安装 Redis

categories:
- Redis 教程

tags:
- redis

date: 2019-07-14
---

## redis 安装
1. 下载最新版 Redis-5.0.5 源码
```shell
sudo mkdir -p /home/downloads
sudo cd /home/downloads
sudo wget http://download.redis.io/releases/redis-5.0.5.tar.gz
```
1. 解压缩
```shell
sudo tar -zxvf redis-5.0.5.tar.gz
```
1. 编译
```shell
sudo cd redis-5.0.5
sudo make MALLOC=libc
```
1. 修改 redis.conf 配置文件
```shell
vim redis.conf

# 1. 取消绑定本地 IP
# bind 127.0.0.1

# 2. 取消保护模式
protected-mode no

# 3. 启用守护进程
daemonize yes

# 4. 设置密码
requirepass 123456
```
1. 启动 Redis
```shell
sudo src/redis-server redis.conf 
```
1. 检测是否运行成功
```shell
sudo yum install net-tools
sudo netstat -ntlp

# Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
# tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      5346/src/redis-serv 
# tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2750/sshd           
# tcp6       0      0 :::6379                 :::*                    LISTEN      5346/src/redis-serv 
# tcp6       0      0 :::22                   :::*                    LISTEN      2750/sshd  
```

## 主从同步配置
现有 Redis1（202.182.108.206），Redis2（202.182.118.220），如果