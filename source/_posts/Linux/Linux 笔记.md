---
title: Linux 笔记

tags:
- Linux

categories:
- Linux

date: 2018-01-25
---

Linux is a name that broadly denotes a family of free and open-source software operating systems (OS) built around the Linux kernel. Typically, Linux is packaged in a form known as a Linux distribution (or distro for short) for both desktop and server use. The defining component of a Linux distribution is the Linux kernel, an operating system kernel first released on September 17, 1991, by Linus Torvalds. Many Linux distributions use the word "Linux" in their name. The Free Software Foundation uses the name GNU/Linux to refer to the operating system family, as well as specific distributions, to emphasize that most Linux distributions are not just the Linux kernel, and that they have in common not only the kernel, but also numerous utilities and libraries, a large proportion of which are from the GNU project. This has led to some controversy. 001

## Ubuntu安装Mysql并开放远程端口
1. 安装Mysql
	```shell
	sudo apt-get install mysql-server
	```
1. 进入mysql设置远程访问账户
	```sql
	grant all on *.* to root@'%' identified by 'root' with grant option;
	flush privileges;
	```
1. 编辑mysql配置文件，把其中`bind-address = 127.0.0.1`注释了
	```shell
	sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf 
	```

1. 重启MySql
	```shell
	service mysql restart
	```

## 程序前台后台切换
1. 在Linux终端运行命令的时候，在命令末尾加上 & 符号，就可以让程序在后台运行
	```shell
	root@Ubuntu$ ./tcpserv01 &
	```

2. 如果程序正在前台运行，可以使用 `ctrl + z` 选项把程序暂停，然后用 `bg %[number]` 命令把这个程序放到后台运行
	```shell
	cat@Ubuntu:~/unp/unpv13e/tcpcliserv$ ./tcpserv01
	^Z
	[1]+  Stopped                 ./tcpserv01
	cat@Ubuntu:~/unp/unpv13e/tcpcliserv$ bg %1
	[1]+ ./tcpserv01 &
	cat@Ubuntu:~/unp/unpv13e/tcpcliserv$
	```

3. 对于所有运行的程序，我们可以用`jobs –l`指令查看
	```shell
	cat@Ubuntu:~/unp/unpv13e/tcpcliserv$ jobs -l
	[1]+  4524 Running                 ./tcpserv01 &
	```

4. 也可以用 `fg %[number]` 指令把一个程序掉到前台运行
	```shell
	cat@Ubuntu:~/unp/unpv13e/tcpcliserv$ fg %1
	./tcpserv01
	```

5. 也可以直接终止后台运行的程序，使用 `kill` 命令
	```shell
	cat@Ubuntu:~/unp/unpv13e/tcpcliserv$ kill %1
	```

6. `ps -ef`列出所有正在进行的进程


## ssh scp等消除每次问yes/no方法 
Are you sure you want to continue connecting (yes/no)? 
- 这个是ssh安全认证是的一个RSA认证。此处必须选择yes才能连接，第一次yes后，他会询问你是否永久把这个RSA认证加入本地，选择yes后，以后不会再出现提醒。	每次登陆只需要输入密码即可。
- 也可以不用输入1中的yes，但是需要修改本机配置。
	```shell
	sudo vim /etc/ssh/ssh_config
	# StrictHostKeyChecking ask 改成
	StrictHostKeyChecking no 
	```

## 安装Java
```shell
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update

JDK6 ：
sudo apt-get install oracle-java6-installer

JDK 7：
sudo apt-get install oracle-java7-installer

JDK 8:
sudo apt-get install oracle-java8-installer
```