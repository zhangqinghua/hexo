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

## 常见问题
1. Docker 启动 MySQL 闪退
内存不足。