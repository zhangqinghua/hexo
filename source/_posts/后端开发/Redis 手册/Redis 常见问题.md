---
title: Redis 常见问题

categories:
- 后端开发
- Redis 手册

date: 2021-06-09
---

#### MISCONF Redis is configured to save RDB snapshots, but it is currently not able to...
场景：Docker 部署 Redis，隔几周就报这个异常。
原因：Redis 快照关闭了导致不能持久化。
解决：设置 `stop-writes-on-bgsave-error: no`。

```conf
# redis.conf
stop-writes-on-bgsave-error: no
```

参考：[笔记：解决redis连接错误：MISCONF Redis is configured to save RDB snapshots, but it is currently not able to...](https://blog.csdn.net/qq_31766907/article/details/78715935)