---
title: Linux 磁盘命令

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:00:59
---

## 磁盘状态
#### 查询磁盘剩余空间
`df` 用于查看磁盘剩余空间。

```bash
[root@izwz98nlvwu8fv5c0warnfz ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G   30G  7.5G  80% /
devtmpfs        3.7G     0  3.7G   0% /dev
tmpfs           3.7G     0  3.7G   0% /dev/shm
tmpfs           3.7G  8.6M  3.7G   1% /run
tmpfs           3.7G     0  3.7G   0% /sys/fs/cgroup
tmpfs           756M     0  756M   0% /run/user/0
```

#### 查询某个目录所占空间大小
`du` 用来查看某个目录所占空间大小，语法：du [-abckmsh] [文件或者目录名]。

```bash
# 不加任何选项和参数只列出目录（包含子目录）大小。
$ du folder
16	folder/SubFolder2
240592	folder/SubFolder1
240640	folder

# 直查指定目录大小
$ du --max-depth=1 /data/ -h
1.9G	/data/deploy
4.0K	/data/wwwlogs
216M	/data/mysql
2.1G	/data/

# -a 全部文件与目录大小都列出来
$ du -a folder
16	folder/.DS_Store
8	folder/File2
8	folder/SubFolder2/File2
8	folder/SubFolder2/File1
16	folder/SubFolder2
240576	folder/SubFolder1/File2.ttc
16	folder/SubFolder1/.DS_Store
0	folder/SubFolder1/File1
240592	folder/SubFolder1
8	folder/File1
240640	folder

# -c 最后加总
$ du  -c folder
16	folder/SubFolder2
240592	folder/SubFolder1
240640	folder
240640	total 

# -s：只列出总和
$ du -s folder
240640	folder

# -h：系统自动调节单位  -m：以MB为单位输出 -k：以KB为单位输出 
# -b：列出的值以bytes为单位输出，默认是以Kbytes
$ du -h folder
8.0K	folder/SubFolder2
117M	folder/SubFolder1
118M	folder
```

## 清理磁盘
#### 释放磁盘已删除文件
已删除的文件可能由于进程未结算，还占用空间。这时候需要杀掉进程或长期服务器才能把空间释放出来。

```bash
# 查询已删除但未释放空间的文件（-h 以合适的单位显示）
$ lsof | grep deleted -h

# 对待这种进程不停对文件写日志的操作，要释放文件占用的磁盘空间，最好在线清空这个文件
$ echo "" > myfile.iso
```

#### 分区和格式化磁盘的
后续添加：https://www.cnblogs.com/zhang-jun-jie/p/9266810.html

## 磁盘分析
#### 评估磁盘 I/O 性能
`iostat` 用于评估磁盘 I/O 性能。

```bash
# 2      每隔2秒采样一次
# 3      共采样3次
# rkB/s  每秒读取数据量 kB
# wkB/s  每秒写入数据量 kB
# svctm  I/O请求的平均服务时间，单位毫秒。svctm和await的值相近，表示没有I/O等待，磁盘性能很好
# await  I/O请求的平均等待时间，单位毫秒，值越小，性能越好。如果svctm和await的值差距大，需要优化程序或更换磁盘
# util   一秒钟有百分几的时间用于I/O操作。接近100%时，表示磁盘带宽跑满，需要优化程序或增加磁盘

[root@izwz98nlvwu8fv5c0warnfz ~]# iostat -xdk 2 3
Linux 3.10.0-693.2.2.el7.x86_64 (izwz98nlvwu8fv5c0warnfz)       01/22/2021      _x86_64_        (4 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.04     2.55    0.05    2.16     2.39    36.06    34.81     0.01    2.96   12.86    2.75   0.13   0.03
vdb               0.00     0.00    0.00    0.00     0.00     0.00    45.71     0.00    0.60    0.60    0.00   0.53   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    2.00     0.00    14.00    14.00     0.00    0.00    0.00    0.00   0.00   0.00
vdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     5.00    0.00    1.50     0.00    26.00    34.67     0.00    0.00    0.00    0.00   0.00   0.00
vdb               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
```

#### 查询进程的磁盘状态
还可以使用 `pidstat` 用于查看每个进程使用内存的用量分解信息，参考上面。

#### 查询文件状态
```bash
$ stat /data
  文件："/data"
  大小：4096      	块：8          IO 块：4096   目录
设备：fd01h/64769d	Inode：1572865     硬链接：5
权限：(0755/drwxr-xr-x)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2020-05-15 18:00:35.958200010 +0800
最近更改：2019-11-14 20:04:27.523461820 +0800
最近改动：2019-11-14 20:04:27.523461820 +0800
创建时间：-
```
