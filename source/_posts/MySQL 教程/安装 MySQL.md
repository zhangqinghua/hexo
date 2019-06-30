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