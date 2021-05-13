---
title: Linux 内存命令

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:00:57
---
## 内存状态
#### free 查看内存状态
`free` 命令用于查看应用程序可用内存数。

经验值：
1. 应用程序可用内存/系统物理内存 < 20% 内存不足；
1. 应用程序可用内存/系统物理内存 > 70% 内存充足；

```bash
# 按照KB单位显示
[root@izwz98nlvwu8fv5c0warnfz ~]# free
              total        used        free      shared  buff/cache   available
Mem:        7734544     3901428     1668240        8744     2164876     3535576
Swap:             0           0           0

# 按照MB单位显示
[root@izwz98nlvwu8fv5c0warnfz ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           7553        3810        1629           8        2114        3452
Swap:             0           0           0

# 按照GB单位显示
[root@izwz98nlvwu8fv5c0warnfz ~]# free -g
              total        used        free      shared  buff/cache   available
Mem:              7           3           1           0           2           3
Swap:             0           0           0

# 自动选取合适单位显示
[root@izwz98nlvwu8fv5c0warnfz ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           7.4G        3.7G        1.6G        8.5M        2.1G        3.4G
Swap:            0B          0B          0B
```

## 内存进程
#### pidstat 查看进程内存占用
`pidstat` 用于查看每个进程使用 CPU 和内存的用量分解信息。

```bash
# -u 2         每个2秒采样一次
# -p 1282167   进程Id

[root@iZj6ci1r3ycwolsv7h3gbiZ ~]# pidstat -u 2 -p 1282167 
Linux 4.18.0-193.28.1.el8_2.x86_64 (iZj6ci1r3ycwolsv7h3gbiZ)    01/22/2021      _x86_64_        (4 CPU)

12:01:53 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
12:01:55 PM     0   1282167    0.50    0.00    0.00    0.00    0.50     3  java
12:01:57 PM     0   1282167    0.50    0.00    0.00    0.00    0.50     3  java
12:01:59 PM     0   1282167    4.50    0.50    0.00    0.00    5.00     3  java
```

## 释放内存
#### sync 释放内存
众所周知，Linux 系统会随着长时间的运行，会产生很多缓存，清理方式就是写一个数字到 `drop_caches` 文件里，这个数字通常为 3。

`sync` 命令可以将所有未写的系统缓冲区写到磁盘中，执行之后就可以放心的释放缓存了。

```bash
sync && echo 3 >/proc/sys/vm/drop_caches 
```