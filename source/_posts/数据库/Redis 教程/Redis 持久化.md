---
title: Redis 持久化

categories:
- Redis 教程

tags:
- redis

date: 2020-05-28
---
是什么、有什么、怎么用、为什么。

## 持久化简介

## RDB 持久化
RDB 是在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行说讲的 Snapshot 快照，它恢复时是将快照文件直接读到内存中。

Redis 会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程不进行任何 IO 操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那 RDB 方式要比 AOF 方式更加高效。RDB 的缺点是最后一次持久化后的数据可能丢失。

Fork 的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。

RDB 保存的是 dump.db 文件，存储于 Redis 的根目录。

#### RDB 的策略
Redis RDB 持久化的触发条件有以下：
1. Redis shutdown；
1. Redis 执行 `save` 或者 `bgsave` 命令；
1. 符合配置文件的其中一个条件；

save 命令只管保存，其它不管，全部阻塞。bgsave 命令 Redis 会在后台异步进行快照操作，同时还可以响应客户端请求。可以通过 lastsave 命令获取最后一次成功执行快照的时间。

默认持久化配置：

```conf
save 900          # 900 秒内改动一次，就触发持久化。
save 300 10       # 300 秒内改动一次，就触发持久化
save 60  10000    # 60  秒内改动一次，就触发持久化
```

如果需要禁用持久化，只需要加上以下配置：

```conf
save ""
```

#### RDB 的恢复
在配置文件加上以下的配置后，Redis 重启会从安装目录寻找 dump.rdb 文件进行恢复。

```conf


```

Redis 重启启动后，会从安装目录寻找 dump.rdb 文件进行恢复。

> 注意，Redis shutdown 之后会立即备份数据，把原来备份的 dump.rdb 给覆盖掉。

#### RDB 的其它配置
`stop-writes-on-bgsave-error` 当备份出错时是否停止写入。如果设置为 yes 那么当持久化是出现异常，比如磁盘满了，那么 Redis 就会抛出以下异常并拒绝新的写入：

```
Redis is configured to save RDB snapshots, but is currently not able to persist on disk. 
```

也就是说，它认为，你当下，持久化数据出现了问题，你就不要再 set 啦。

如果设置为 no，表示你不在乎数据不一致或者有其它的手段发现和控制。

```conf
stop-writes-on-bgsave-error yes
```

`rdbcompression` 对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的化，Redis 会采用 LZF 算法进行压缩。如果你不想消耗 CPU 来进行压缩的化，可以设置关闭此功能。

```conf
rdbcompression yes
```

`rdbchecksum` 在存储快照后，还可以让 Redis 使用 CRC64 算法来进行数据校验，但是这样做会增加大约 10% 的性能消耗。如果希望获取最大的性能提升，可以关闭此功能。  

## 两种备份方式的对比


## Snapshot 快照
只要符合其中一个条件，就触发持久化。
