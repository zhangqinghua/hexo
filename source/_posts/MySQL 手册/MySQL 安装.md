---
title: MySQL 安装

categories:
- MySQL 手册

date: 2020-07-07 00:00:13
---

## Centos 本地安装
安装后，配置文件位于 `/etc/my.cnf`，数据文件位于 `/var/lib/mysql`。

#### 卸载旧 MySQL
```bash
[root@vultrguest ~]# yum list installed mysql*
Failed to set locale, defaulting to C.UTF-8
Installed Packages
mysql.x86_64                                         8.0.17-3.module_el8.0.0+181+899d6349                                  @AppStream
mysql-common.x86_64                                  8.0.17-3.module_el8.0.0+181+899d6349                                  @AppStream
[root@vultrguest ~]# yum remove mysql-common.x86_64
...
[root@vultrguest ~]# yum list installed mysql*
Failed to set locale, defaulting to C.UTF-8
Error: No matching Packages to list
```

#### 下载 rpm 源文件
```bash
[root@vultrguest ~]# sudo mkdir /home/downloads
[root@vultrguest ~]# sudo cd  /home/downloads
# 这里是MySQL 5.7的源文件
[root@vultrguest ~]# sudo wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'
# 这里是MySQL 8.0的源文件
[root@vultrguest ~]# sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

#### 添加 rpm 到仓库
```bash
[root@vultrguest ~]# sudo rpm -ivh mysql80-community-release-el7-3.noarch.rpm
[root@vultrguest ~]# sudo yum update
```

#### 使用 yum 安装
```bash
[root@vultrguest ~]# sudo yum install mysql-community-server
...
```

#### 启动 MySQL 服务
```bash
[root@vultrguest ~]# sudo systemctl start mysqld
...
```

#### 查看初始密码
```bash
[root@vultrguest ~]# sudo grep 'temporary password' /var/log/mysqld.log
0sIbtR0(Wkge
```

#### 登录MySQL
使用到初始密码。

```bash
[root@vultrguest ~]# mysql -uroot -p  # 输入查看到的密码
...
```

## Centos 在线安装
安装后，配置文件位于 `/etc/my.cnf`，数据文件位于 `/var/lib/mysql`。

#### 搜索 MySQL
```bash
[root@vultrguest ~]# yum list | grep mysql
Failed to set locale, defaulting to C.UTF-8
apr-util-mysql.x86_64                                1.6.1-6.el8                                       AppStream 
dovecot-mysql.x86_64                                 1:2.2.36-10.el8                                   AppStream 
freeradius-mysql.x86_64                              3.0.17-6.module_el8.1.0+198+858eb655              AppStream 
```

#### 安装MySQL客户端
```bash
[root@vultrguest ~]# yum install -y mysql
...
```

#### 安装 MySQL 服务端
没有指定版本默认 8.0。

```bash
[root@vultrguest ~]# yum install -y mysql-server
...
```

#### 启动 MySQL 服务
```bash
[root@vultrguest ~]# systemctl start mysqld
[root@vultrguest ~]# systemctl status mysqld
● mysqld.service - MySQL 8.0 database server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-03-08 17:58:00 UTC; 15s ago
...
```

#### 登录 MySQL
刚安装密码为空。
```bash
[root@vultrguest ~]# mysql -uroot -p
Enter password:    
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.17 Source distribution

mysql> 
```


## Docker 安装
安装后，配置文件位于容器 `/etc/my.cnf`，数据文件位于 `/var/lib/mysql`。

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

# 登录 MySQL
...
```

## 常见问题
1. MySQL 启用失败
如果启动失败，可以查看 MySQL 日志文件 `/var/log/mysql/mysqld.log`，一般是内存不足。MySQL 刚启动就占用了接近 500MB 的内存，所以机器配置最低也需要 1G。


1. 登录报错 `error 2059: Authentication plugin 'caching_sha2_password' cannot be loaded`
这是因为 MySQL8.0 版本默认的认证方式是 caching_sha2_password。若想 在MySQL8.0 版本中继续使用旧版本中的认证方式需要在 my.cnf 文件中配置并重启，因为此参数不可动态修改。

```mysql
mysql> set global default_authentication_plugin='mysql_native_password';
ERROR 1238 (HY000): Variable 'default_authentication_plugin' is a read only variable
```

写入my.cnf文件后重启MySQL：
```bash
vim my.cnf
[mysqld]
default_authentication_plugin=mysql_native_password
```

另一种解决方法：兼容新老版本的认证方式。
```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root' PASSWORD EXPIRE NEVER; #修改加密规则 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root'; #更新一下用户的密码 
FLUSH PRIVILEGES; #刷新权限
--创建新的用户：
create user root@'%' identified WITH mysql_native_password BY 'root';
grant all privileges on *.* to root@'%' with grant option;
flush privileges;
--在MySQL8.0创建用户并授权的语句则不被支持：
mysql> grant all privileges on *.* to root@'%' identified by 'root' with grant option;
   ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'identified by 'root' with grant option' at line 1
   mysql> 
```