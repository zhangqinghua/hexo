---
title: MySQL 部署-安装

categories:
- MySQL 指南

date: 2020-04-28 00:00:01
---

## 使用yum安装MySQL
### 卸载旧的MySQL
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

### 使用yum源安装 

#### 下载rpm文件
```bash
[root@vultrguest ~]# sudo mkdir /home/downloads
[root@vultrguest ~]# sudo cd  /home/downloads
# 这里是MySQL 5.7的源文件
[root@vultrguest ~]# sudo wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'
# 这里是MySQL 8.0的源文件
[root@vultrguest ~]# sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

#### 添加源到仓库中
```bash
[root@vultrguest ~]# sudo rpm -ivh mysql80-community-release-el7-3.noarch.rpm
[root@vultrguest ~]# sudo yum update
```

#### 安装MySQL
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

#### 登录MySQL（使用初始密码）
```bash
[root@vultrguest ~]# mysql -uroot -p  # 输入查看到的密码
...
```

### 使用yum库安装
配置文件：/etc/my.cnf
数据文件：/var/lib/mysql

#### 搜索仓库是否有mysql-server
```bash
[root@vultrguest ~]# yum list | grep mysql
Failed to set locale, defaulting to C.UTF-8
apr-util-mysql.x86_64                                1.6.1-6.el8                                       AppStream 
dovecot-mysql.x86_64                                 1:2.2.36-10.el8                                   AppStream 
freeradius-mysql.x86_64                              3.0.17-6.module_el8.1.0+198+858eb655              AppStream 
grafana-mysql.x86_64                                 6.2.2-2.el8                                       AppStream 
mysql.x86_64                                         8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
mysql-common.x86_64                                  8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
mysql-devel.x86_64                                   8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
mysql-errmsg.x86_64                                  8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
mysql-libs.x86_64                                    8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
mysql-server.x86_64                                  8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
mysql-test.x86_64                                    8.0.17-3.module_el8.0.0+181+899d6349              AppStream 
pcp-pmda-mysql.x86_64                                4.3.2-2.el8                                       AppStream 
php-mysqlnd.x86_64                                   7.2.11-2.module_el8.1.0+209+03b9a8ff              AppStream 
postfix-mysql.x86_64                                 2:3.3.1-9.el8                                     AppStream 
qt5-qtbase-mysql.i686                                5.11.1-7.el8                                      AppStream 
qt5-qtbase-mysql.x86_64                              5.11.1-7.el8                                      AppStream 
rsyslog-mysql.x86_64                                 8.37.0-13.el8                                     AppStream 
rubygem-mysql2.x86_64                                0.4.10-4.module_el8.1.0+214+9be47fd7              AppStream 
rubygem-mysql2-doc.noarch                            0.4.10-4.module_el8.1.0+214+9be47fd7              AppStream 
```

#### 安装MySQL客户端
```bash
[root@vultrguest ~]# yum install -y mysql
...
```

#### 安装MySQL服务端（没有指定版本默认8.0）
```bash
[root@vultrguest ~]# yum install -y mysql-server
...
```

#### 启动MySQL服务
```bash
[root@vultrguest ~]# systemctl start mysqld
[root@vultrguest ~]# systemctl status mysqld
● mysqld.service - MySQL 8.0 database server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-03-08 17:58:00 UTC; 15s ago
  Process: 6095 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 5968 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mysqld.service (code=exited, status=0/SUCCESS)
  Process: 5944 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 6052 (mysqld)
   Status: "Server is operational"
    Tasks: 39 (limit: 5066)
   Memory: 461.7M
   CGroup: /system.slice/mysqld.service
           └─6052 /usr/libexec/mysqld --basedir=/usr

Mar 08 17:57:53 vultrguest systemd[1]: Starting MySQL 8.0 database server...
Mar 08 17:57:53 vultrguest mysql-prepare-db-dir[5968]: Initializing MySQL database
Mar 08 17:58:00 vultrguest systemd[1]: Started MySQL 8.0 database server.
```

#### 登录MySQL（刚安装密码为空）
```bash
[root@vultrguest ~]# mysql -uroot -p
Enter password:    
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.17 Source distribution

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

## 修改用户密码
#### MySQL 5.7修改用户密码
```sql
-- 调整MySQL密码验证规则
set global validate_password_policy=0;
set global validate_password_length=4;
SET global sql_mode = '';
-- 修改用户密码
alter user 'root'@'localhost' identified by 'bg360123456';
```

#### MySQL 8.0修改用户密码
```sql
-- 调整MySQL密码验证规则
set global validate_password.policy=0;
set global validate_password.length=4;
-- 修改用户密码
alter user 'root'@'localhost' identified with mysql_native_password by '123456';
```

## 创建数据库和用户
```sql
-- 5.7和8.0均适用
create database aonitask character set utf8mb4;
create user 'username'@'host' identified by 'password';
grant all privileges on aonitask.* to 'aonitask'@'%' with grant option;
```
