---
title: Linux 网络命令

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:00:53
---
#### 查询本机端口占用情况
lsof 用于查看某一端口的占用情况，比如查看 8000 端口使用情况：`lsof -i:8000`。

```bash
$ lsof -i:8000
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
lwfs    22065 root    6u  IPv4 4395053      0t0  TCP *:irdmi (LISTEN)

```

netstat 命令用于显示网络状态。

```bash
# 找出程序运行的端口（并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。）
$ netstat -ap | grep nginx
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN      24872/nginx: worker 
tcp        0      0 0.0.0.0:818             0.0.0.0:*               LISTEN      24872/nginx: worker 
tcp        0      0 0.0.0.0:https           0.0.0.0:*               LISTEN      24872/nginx: worker 
tcp        0      0 0.0.0.0:svn             0.0.0.0:*               LISTEN      24872/nginx: worker 
tcp        0      0 iZwz9cp52a5yqx9b8:https 119.123.69.251:65008    ESTABLISHED 24872/nginx: worker 
tcp        0      0 iZwz9cp52a5yqx9b8:https 119.123.69.251:65014    ESTABLISHED 24872/nginx: worker 
unix  3      [ ]         STREAM     CONNECTED     175354591 24872/nginx: worker  
unix  3      [ ]         STREAM     CONNECTED     175354590 30572/nginx: master  

# 找出运行在指定端口的进程
$ netstat -pan | grep 60819
tcp        0      0 0.0.0.0:60819           0.0.0.0:*               LISTEN      16586/java          
tcp        1      0 127.0.0.1:60819         127.0.0.1:47942         CLOSE_WAIT  16586/java 

# 查看是否有未知IP在进行发包
$ netstat -lntupa
```

#### 测试远程端口开放状态
我们可以使用 `nmap` 命令来查看目标服务器的端口开放状态。

```bash
$ nmap 120.77.246.50
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-05 15:10 CST
Nmap scan report for gsm.icebartch.com (120.77.246.50)
Host is up (0.0071s latency).
Not shown: 995 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
443/tcp  open   https
3306/tcp closed mysql
3690/tcp open   svn
```

#### 测试远程端口连通性
在确认了目标服务器的端口开放性之后，我们就可以使用 `telnet` 来测试端口的连通性了。

```bash
# 连通的情况，出现Escape character
$ telnet 120.77.246.50 22
Trying 120.77.246.50...
Connected to gsm.icebartch.com.
Escape character is '^]'.
SSH-2.0-OpenSSH_7.4

# 不通的情况，一直卡死
$ telnet 120.77.246.50 21
Trying 120.77.246.50...
```

#### 修改Linux系统实例默认远程端口
1. 修改配置
  将 `/etc/ssh/sshd_config` 下 `Port 22` 改为 `Port 1022`
1. 重启 sshd 服务。
  `systemctl restart sshd` 
1. 开发防火墙
  可选
1. 重新登录
  `ssh -p 1022 root@xxxx`


