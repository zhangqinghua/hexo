---
title: Linux 常用操作.md

categories:
- Linux 系列

tag:
- Linux

date: 2020-05-24
---

## 磁盘操作
#### 查看磁盘或目录的容量
df 查看已挂载磁盘的总容量、使用容量、剩余容量等，可以不加任何参数，默认是按k为单位显示的。

Filesystem 表示扇区，也就是你划分磁盘时所分的区；1K-blocks/1M-blocks 表示以 1K/1M 为单位；Used 和 Available 分别是已使用和剩余；Use% 就是已经使用的百分比，如果这个值大于 90% 那么你就应该注意了，磁盘很有可能马上就会变满的；Mounted on 则表示该分区（扇区）所挂载的地方。

```bash
$ df
Filesystem    512-blocks     Used Available Capacity iused               ifree %iused  Mounted on
/dev/disk1s1    64692872 48488488  10759648    82%  945226 9223372036853830581    0%   /
devfs                371      371         0   100%     642                   0  100%   /dev
/dev/disk1s4    64692872  4194352  10759648    29%       2 9223372036854775805    0%   /private/var/vm
/dev/disk0s3   184580088 60742528 123837560    33%  282381            62236207    0%   /Volumes/Local Disk
map -hosts             0        0         0   100%       0                   0  100%   /net
map auto_home          0        0         0   100%       0                   0  100%   /home

# -h 使用合适的单位显示，例如G
$ df -h
Filesystem      Size   Used  Avail Capacity iused               ifree %iused  Mounted on
/dev/disk1s1    31Gi   23Gi  5.1Gi    82%  945233 9223372036853830574    0%   /
devfs          186Ki  186Ki    0Bi   100%     642                   0  100%   /dev

# -k -m 分别为使用K，M为单位显示
$ df -m
Filesystem    1M-blocks  Used Available Capacity iused               ifree %iused  Mounted on
/dev/disk1s1      31588 23680      5249    82%  945234 9223372036853830573    0%   /
```

#### 查看某个目录所占空间大小
du 用来查看某个目录所占空间大小，语法：du [-abckmsh] [文件或者目录名]。

```bash
# 不加任何选项和参数只列出目录（包含子目录）大小。
$ du folder
16	folder/SubFolder2
240592	folder/SubFolder1
240640	folder

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

#### 磁盘的分区和格式化
后续添加：https://www.cnblogs.com/zhang-jun-jie/p/9266810.html

## 端口操作
#### 查看端口占用情况
```bash
# lsof -i:端口号 用于查看某一端口的占用情况，比如查看8000端口使用情况，lsof -i:8000
$ lsof -i:8000
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
lwfs    22065 root    6u  IPv4 4395053      0t0  TCP *:irdmi (LISTEN)

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
```

#### 查看端口开放情况
略过。。。

#### 远程测试端口情况
```bash
# 使用 telnet 方式测试远程主机端口是否打开
$ telnet 120.77.246.50 22
Trying 120.77.246.50...
Connected to 120.77.246.50.
```

## 文件操作
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

#### 搜索文件
```bash
# 将目前目录及其子目录下所有延伸档名是 c 的文件列出来 
$ find  -name "*.jar"
./file/icebartech-cutterbar.jar
./file/icebartech-cutterbar-1.jar

# 模糊搜索指定文件（目录、文件名）
$ find /data -cmin -100 | egrep  "icebartech"
/data/wwwlogs/sportcenter.icebartech.com_nginx.log
/data/deploy/Java/icebartech-vastscene/logs/start.log

# 将目前目录其其下子目录中所有一般文件列出
$ find -type f
./deploy/logs/enweis/info.enweis.log
./deploy/logs/enweis/error.enweis.log

# 排除指定文件
$ find /data -ctime -1 | egrep -v "enjoyshop"
/data/wwwlogs/80_nginx.log
/data/wwwlogs/sportcenter.icebartech.com_nginx.log

$ find /data -ctime -1 | egrep -v "(enjoyshop|sportcenter)"
/data/wwwlogs/80_nginx.log

# 按时间查找
# +n：>n -n: <n n: =n
# mtime 修改实际（天） mmin 修改实际（分）
# atime 访问实际（天） amin 访问实际（分）
# ctime 创建实际（天） cmin 创建实际（分）
# /data 目录下一天内创建的文件
$ find /data -ctime -1
/data/wwwlogs/80_nginx.log
/data/wwwlogs/enjoyshop.icebartech.com_nginx.log
/data/wwwlogs/sportcenter.icebartech.com_nginx.log
```

## 历史操作
```bash
$ history
1093  clear
1094  find /data | egrep ".jar" - ls
1095  find /data | egrep "icebartech-pianoroom" - ls
1096  find /data/deploy/Java/icebartech- | egrep ".jar" - ls
1097  find /data/deploy/Java/ | egrep ".jar" - ls
1098  find /data/deploy/Java/ | egrep "icebartech-gsm" - ls
1099  find /data/deploy/Java/ | egrep "icebartech-gsm" 
1100  history
```