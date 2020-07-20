---
title: MySQL 基本操作

categories:
- MySQL 手册

date: 2020-07-07 00:00:12
---
## 修改用户密码
#### MySQL 5.7 修改用户密码
```sql
-- 调整MySQL密码验证规则
set global validate_password_policy=0;
set global validate_password_length=4;
SET global sql_mode = '';
-- 修改用户密码
alter user 'root'@'localhost' identified by 'bg360123456';
```

#### MySQL 8.0 修改用户密码
```sql
-- 调整MySQL密码验证规则
set global validate_password.policy=0;
set global validate_password.length=4;
-- 修改用户密码
alter user 'root'@'localhost' identified with mysql_native_password by '123456';
```

## 创建数据库和用户
5.7 和 8.0 均适用。

```sql
create database aonitask character set utf8mb4;
create user 'username'@'host' identified by 'password';
grant all privileges on aonitask.* to 'aonitask'@'%' with grant option;
```
