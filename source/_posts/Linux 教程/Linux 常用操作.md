---
title: Linux 常用操作

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

#### 磁盘释放已删除文件
已删除的文件可能由于进程未结算，还占用空间。这时候需要杀掉进程或长期服务器才能把空间释放出来。

```bash
# 查询已删除但未释放空间的文件（-h 以合适的单位显示）
$ lsof | grep deleted -h

# 对待这种进程不停对文件写日志的操作，要释放文件占用的磁盘空间，最好在线清空这个文件
$ echo "" > myfile.iso
```

#### 磁盘的分区和格式化
后续添加：https://www.cnblogs.com/zhang-jun-jie/p/9266810.html

## 端口操作
#### 查看本机端口占用情况
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

# 查看是否有未知IP在进行发包
$ netstat -lntupa
```

#### 测试远程端口开放状态
我们可以使用 `nmap` 命令来查看目标服务器的端口开放状态。

```bash
zhangqinghua$ nmap 120.77.246.50
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
zhangqinghua$ telnet 120.77.246.50 22
Trying 120.77.246.50...
Connected to gsm.icebartch.com.
Escape character is '^]'.
SSH-2.0-OpenSSH_7.4

# 不通的情况，一直卡死
zhangqinghua$ telnet 120.77.246.50 21
Trying 120.77.246.50...
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

# 将指定大小的文件搜索处理
find . -type f -size +800M


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

## 复制文件
#### 上传文件到服务器
```bash
# 上传文件到服务站指定目录
scp users/test.txt root@127.0.0.1:/usr/local/users

# 上传文件到服务器指定目录并修改文件名
scp users/test.txt root@127.0.0.1:/usr/local/users/tests2.txt

# （test 目录存在）上传 users 文件夹到服务器 /usr/local/test 目录
scp -r users root@127.0.0.1:/usr/local/test

# （test 目录不存在）上传 users 文件夹到服务器指 /usr/local/ 目录并修改文件夹名称为 test
scp -r users root@127.0.0.1:/usr/local/test
```

#### 从服务器下载文件
从远程复制到本地，只要将从本地复制到远程的命令的后2个参数调换顺序即可。
```bash
scp root@www.runoob.com:/home/root/others/music /home/space/music/1.mp3 

scp -r www.runoob.com:/home/root/others/ /home/space/music/

# 如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号，命令格式如下：
scp -P 4588 remote@www.runoob.com:/usr/local/sin.sh /home/administrator
```

## 其他
脚本贴心的帮我们释放了一些内存资源，以便获取更多的资源进行挖矿。

众所周知，Linux 系统会随着长时间的运行，会产生很多缓存，清理方式就是写一个数字到 drop_caches 文件里，这个数字通常为 3。

sync 命令将所有未写的系统缓冲区写到磁盘中，执行之后就可以放心的释放缓存了。

```bash
sync && echo 3 >/proc/sys/vm/drop_caches 
```

## 修改Linux系统实例默认远程端口
1. 修改配置
  将 `/etc/ssh/sshd_config` 下 `Port 22` 改为 `Port 1022`
1. 重启 sshd 服务。
  `systemctl restart sshd` 
1. 开发防火墙
  可选
1. 重新登录
  `ssh -p 1022 root@xxxx`

## 删除文件
当使用rm删除文件和文件夹的时候提示：rm: 无法删除"bash": 不允许的操作

删除属性：
chattr -i authorized_keys2
chattr -a authorized_keys2
chattr -u authorized_keys2

再次删除该文件，即可正常删除了

## 重装后无法登录
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

## 高延迟SSH部分解决方案
VPS 在国外，延迟总有那么 200～300ms，一来一回，500ms 是免不了了。可是默认情况下，你每输入一个字符，SSH 客户端（openssh/putty/securecrt）都会发送给服务器，然后服务器将响应返回。

典型 ssh 情况下是执行命令，比如 ls，网络交互是：发送 l 给 svr, svr 返回 l ，显示 l ，发送 s 给 svr，svr 返回 s ，显示 s ，发送回车给 svr，svr执行 ls ，返回 ls 的输出。也就是说，光输入一个 ls 命令就至少需要 1s+ 的时间。但如果是要输入一个很复杂的命令，也许还没输入完，你就崩溃了。

采用 putty 内建的 Local Echo 和 Local Line Editing 支持，可以部分地解决这个问题：默认配置下，登录以后点击左上角的Putty图标，选择change settings=>Terminal，将Local Echo和Local line editing改成force on，就可以允许你在本地编辑一行命令，按下回车，然后命令才被发送到服务器。结果是服务器接收一整条命令，然后显示一整条命令，然后再输出这条命令的执行结果。

首页 登入 RSS 注册 留言 链接 归档 关于
boblog评论系统回归Linux：非特权用户使用crontab实现开机任务
FEB
27
高延迟SSH部分解决方案  不指定
felix021 @ 2012-2-27 21:27 [IT » 网络] 评论(1) , 引用(0) , 阅读(12878) | Via 本站原创 大 | 中 | 小  
vps在国外，延迟总有那么200～300ms，一来一回，500ms是免不了了。可是默认情况下，你每输入一个字符，ssh客户端（openssh/putty/securecrt）都会发送给服务器，然后服务器将响应返回。

典型ssh情况下是执行命令，比如ls，网络交互是：发送 l 给svr, svr返回 l ，显示 l ，发送 s 给svr，svr返回 s ，显示 s ，发送回车给svr，svr执行 ls ，返回 ls 的输出。也就是说，光输入一个ls命令就至少需要1s+的时间。但如果是要输入一个很复杂的命令，也许还没输入完，你就崩溃了。

采用putty(windows版ok，linux版未测试）内建的Local Echo和Local Line Editing支持，可以部分地解决这个问题：默认配置下，登录以后点击左上角的Putty图标，选择change settings=>Terminal，将Local Echo和Local line editing改成force on，就可以允许你在本地编辑一行命令，按下回车，然后命令才被发送到服务器。结果是服务器接收一整条命令，然后显示一整条命令，然后再输出这条命令的执行结果。

相应的代价就是：
1. 没法使用自动补全和其他bash/readline的快捷键了；
2. 使用vi这类程序的时候，就没法正常编辑了，这时需要再把这两个选项关闭。。。（为什么没有快捷键………………）
