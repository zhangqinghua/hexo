---
title: Linux 整机状态

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:00:60
---
#### uptime 查询平均负载
`uptime` 命令能够打印系统总共运行了多长时间和系统的平均负载。`uptime` 命令可以显示的信息显示依次为：现在时间、系统已经运行了多长时间、目前有多少登陆用户、系统在过去的 1 分钟、5 分钟和 15 分钟内的平均负载。

系统平均负载是指在特定时间间隔内运行队列中的平均进程数。如果每个 CPU 内核的当前活动进程数不大于 3 的话，那么系统的性能是良好的。如果每个 CPU 内核的任务数大于 5，那么这台机器的性能有严重问题。如果你的 Linux 主机是 1 个双核 CPU 的话，当 load Average 为 6 的时候说明机器已经被充分使用了。

```bash
# 10:10                          系统当前时间
# up 3 days, 59 mins             主机已运行时间,时间越大，说明你的机器越稳定。
# 2 users                        用户连接数，是总连接数而不是用户数
# load averages: 2.87 2.50 2.29  系统平均负载，统计最近1，5，15分钟的系统平均负载
zhangqinghua$ uptime
10:10  up 3 days, 59 mins, 2 users, load averages: 2.87 2.50 2.29
```

#### top 实时查询系统运行情况
`top` 命令可以实时动态地查看系统的整体运行情况，是一个综合了多方信息监测系统性能和运行信息的实用工具。通过 `top` 命令所提供的互动式界面，用热键可以管理。

```bash
# top - 11:07:27                 当前系统时间
# up 31 days                     系统已经运行了31天
# 2 users                        2个用户当前登录
# load average                   系统负载，统计最近1，5，15分钟的系统平均负载
# Tasks:  82 total               总进程数
# 1 running                      正在运行的进程数
# 81 sleeping                    睡眠的进程数
# 0 stopped                      停止的进程数
# 0 zombie                       冻结进程数
# %Cpu(s)   0.3 us               用户空间占用CPU百分比
# %Cpu(s)   0.5 us               内核空间占用CPU百分比
# %Cpu(s)   0.0 ni               用户进程空间内改变过优先级的进程占用CPU百分比
# %Cpu(s)   99  id               空闲CPU百分比
# %Cpu(s)   0.0 wa               等待输入输出的CPU时间百分比
# %Cpu(s)   0.0 hi               硬中断（Hardware IRQ）占用CPU的百分比
# %Cpu(s)   0.0 si               软中断（Software Interrupts）占用CPU的百分比
# %Cpu(s)   0.0 st               ?
# KiB Mem :  3881692 total       物理内存总量
# KiB Mem :  2273716 used        使用的物理内存总量
# KiB Mem :  157092  free        空闲内存总量
# KiB Mem :  1450884 buff/cache  用作内核缓存的内存量
# KiB Swap:  0 total             交换区总量
# KiB Swap:  0 free              空闲交换区总量
# KiB Swap:  0 used              使用的交换区总量
# KiB Swap:  1331200 avail Mem   代表可用于进程下一次分配的物理内存数量

top - 11:07:27 up 31 days,  2:16,  2 users,  load average: 0.01, 0.30, 0.33
Tasks:  82 total,   1 running,  81 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.5 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3881692 total,   157092 free,  2273716 used,  1450884 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1331200 avail Mem 

# PID 进程Id
# USER   	进程所有者的用户名
# PR        优先级
# NI        nice值。负值表示高优先级，正值表示低优先级
# VIRT      进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
# RES       进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
# SHR       共享内存大小，单位kb
# S         进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
# %CPU      上次更新到现在的CPU时间占用百分比
# %MEM      进程使用的物理内存百分比
# TIME+     进程使用的CPU时间总计，单位1/100秒
# COMMAND   命令名/命令行
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
29336 root      20   0 3570612 621528  13748 S   3.0 16.0  16:02.36 java
23436 root      20   0 1294092  12612   6860 S   1.0  0.3  12:18.56 staragent-core
 1052 root      20   0 1005228  16268   3256 S   0.3  0.4  86:54.00 /usr/local/clou
32750 root      20   0 3921148 414736  13172 S   0.3 10.7   1:17.68 java                                                                                                                                    
```

在 `top` 命令执行过程中可以使用的一些交互命令：
1. `P`   根据CPU使用百分比大小进行排序；
1. `M`   根据驻留内存大小进行排序；
1. `T`   根据时间/累计时间进行排序；
1. `c`   切换显示命令名称和完整命令行；
1. `k`   终止一个进程；
1. `q`   退出程序；

#### 查询进程的线程消耗时间排行
```bash
# -m 显示所有的线程
# -p pid 进程使用CPU的时间
# -o 该参数后是用户自定义格式
[root@iZwz9cp52a5yqx9b86ehecZ icebartech-mlen]# ps -mp 29336 -o THREAD,tid,time
USER     %CPU PRI SCNT WCHAN  USER SYSTEM   TID     TIME
root      1.1   -    - -         -      -     - 00:18:56
root      0.0  19    - futex_    -      - 29336 00:00:00
root      0.0  19    - futex_    -      - 29337 00:00:24
root      0.0  19    - futex_    -      - 29338 00:00:20
```

## 常见问题
#### 假如生产环境出现 CPU 占用过高，请谈谈你的分析思路和定位？
需要结合 Linux 和 JDK 命令一块分析。

1. top 命令找出 CPU 占用最高的进程；
1. ps -ef 或 jps 进一步定位，得知是一个怎样的后台程序；
1. 定位到具体线程或者代码；
1. 将需要的线程 Id 转换为 16 进制格式（英文小写）；
1. jstack