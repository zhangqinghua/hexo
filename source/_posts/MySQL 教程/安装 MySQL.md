---
title: 安装 MySQL

categories:
- MySQL 教程

tag:
- Linux
- MySQL
- 安装

date: 2019-06-30
---


## MySQL 5.7安装
1. 添加MySQL yum源
```bash
wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'
sudo rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum repolist all | grep mysql
```

1. 选择安装版本
如果想安装最新版本的，直接使用 yum 命令即可
```bash
sudo yum install mysql-community-server
```

1. 启动 MySQL 服务
```bash
sudo systemctl start mysqld
sudo systemctl status mysqld
```

1. 查看初始密码
```bash
sudo grep 'temporary password' /var/log/mysqld.log
```

1. 登陆
```bash
mysql -uroot -p  #输入查看到的密码
```

1. 安全设置
```sql
set global validate_password_policy=0;
set global validate_password_length=4;
```

1. 设置root密码
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'bg360123456';
```

1. 创建数据库和用户
```sql
CREATE DATABASE <datebasename> CHARACTER SET utf8mb4;
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
grant all privileges on aonitask.* to 'aonitask'@'%' with grant option;
```

## MySQL 8安装
1. 下载rpm文件
```bash
sudo mkdir /home/downloads
sudo cd  /home/downloads
sudo wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

1. 添加源到仓库中
```bash
sudo rpm -ivh mysql80-community-release-el7-3.noarch.rpm
sudo yum update
```

1. 安装mysql
```bash
sudo yum install mysql mysql-server
```

1. 启动mysql
```bash
sudo systemctl start mysqld
sudo systemctl status mysqld
```
1. 登陆mysql
```bash
# 获取临时密码
grep "temporary password" /var/log/mysqld.log
# 登陆mysql
mysql -uroot -p0sIbtR0(Wkge
```
1. 修改临时密码
```sql
-- 调整MySQL密码验证规则
set global validate_password.policy=0;
set global validate_password.length=4;
-- 修改用户密码
alter user 'root'@'localhost' identified with mysql_native_password by '123456';
```
1. 创建数据库
```sql
create database gcsgame;
```
1. 创建用户
```sql
create user 'gcsgame'@'%' identified by 'gcsgame';
```
1. 分配权限
```sql
grant all privileges on gcsgame.* to 'gcsgame'@'%' with grant option;
```