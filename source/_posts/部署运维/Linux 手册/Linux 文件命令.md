---
title: Linux 文件命令

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:00:55
---

## 文件搜索
#### 按文件名搜索
模糊搜索指定文件（目录、文件名）：

```bash
$ find /data -cmin -100 | egrep  "icebartech"
/data/wwwlogs/sportcenter.icebartech.com_nginx.log
/data/deploy/Java/icebartech-vastscene/logs/start.log
```

#### 按后缀名搜索
将目前目录及其子目录下所有延伸档名是 c 的文件列出来：

```bash
$ find  -name "*.jar"
./file/icebartech-cutterbar.jar
./file/icebartech-cutterbar-1.jar
```

#### 按文件类型搜索
将目前目录其其下子目录中所有一般文件列出：

```bash
$ find -type f
./deploy/logs/enweis/info.enweis.log
./deploy/logs/enweis/error.enweis.log
```

#### 按体积大小搜索
将指定大小的文件搜索处理：

```bash
find . -type f -size +800M
```

#### 按时间搜索
```bash
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

#### 排除指定文件
```bash
$ find /data -ctime -1 | egrep -v "enjoyshop"
/data/wwwlogs/80_nginx.log
/data/wwwlogs/sportcenter.icebartech.com_nginx.log

$ find /data -ctime -1 | egrep -v "(enjoyshop|sportcenter)"
/data/wwwlogs/80_nginx.log
```

## 文件操作
#### 创建文件
#### 复制文件
#### 删除文件

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
从远程复制到本地，只要将从本地复制到远程的命令的后 2 个参数调换顺序即可。
```bash
scp root@www.runoob.com:/home/root/others/music /home/space/music/1.mp3 

scp -r www.runoob.com:/home/root/others/ /home/space/music/

# 如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号，命令格式如下：
scp -P 4588 remote@www.runoob.com:/usr/local/sin.sh /home/administrator
```

## 修改权限
#### 查看权限
```bash
$ ls -l
total 65640
drwxr-xr-x   3 zhangqinghua  staff        96  4 18 22:42 config
drwxr-xr-x   4 zhangqinghua  staff       128  4 18 23:10 data
drwxr-xr-x@ 12 zhangqinghua  staff       384  4 18 22:58 elasticsearch-5.5.1
-rw-r--r--@  1 zhangqinghua  staff  33511694  4 18 22:58 elasticsearch-5.5.1.zip
drwxr-xr-x   2 zhangqinghua  staff        64  4 18 22:49 plugins
```

#### 修改权限
将 `/data/deploy` 目录下所有文件改为可读可写。

```bash
$ chmod -R 777 /data/deploy/;
```

https://blog.csdn.net/lv8549510/article/details/85406215

## 常见问题
#### 删除文件提示无法删除
当使用 `rm` 删除文件和文件夹的时候提示：rm: 无法删除"bash": 不允许的操作

删除属性：

```bash
chattr -i authorized_keys2
chattr -a authorized_keys2
chattr -u authorized_keys2
```

再次删除该文件，即可正常删除了