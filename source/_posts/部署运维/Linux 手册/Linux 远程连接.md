---
title: Linux 远程连接

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:00:52
---


## 常见问题

#### 重装后无法登录
出现这个问题的原因是你本地保存的主机密钥与服务器的密钥不一致，通常因为你重装了服务器的系统，这样它的密钥就变了。

```bash
zhangqinghua$ ssh root@120.76.98.47
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:O+8bUuRS7w2VG1Jk4cqFcfqxBBDLjma35OsXJ6ymznw.
Please contact your system administrator.
Add correct host key in /Users/zhangqinghua/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/zhangqinghua/.ssh/known_hosts:33
ECDSA host key for 120.76.98.47 has changed and you have requested strict checking.
Host key verification failed.
```

解决方法有 2 种：
1. 删除 `.ssh/known_hosts` 对应的前面记录。
1. 清除对应的机器的缓存 `ssh-keygen -R 你的远程服务器ip地址`。

#### SSH 远程连接报错：ssh_exchange_identification: read: Connection reset by peer
发现是公司网络问题，使用手机热点网络能连。被阿里云的云盾拦截了：https://help.aliyun.com/knowledge_detail/37914.html