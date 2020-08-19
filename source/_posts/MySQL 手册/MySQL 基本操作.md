---
title: MySQL 基本操作

categories:
- MySQL 手册

date: 2020-07-07 00:00:13
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

## 常见问题
1. 登录报错 `error 2059: Authentication plugin 'caching_sha2_password' cannot be loaded`
这是因为 MySQL8.0 版本默认的认证方式是 caching_sha2_password。若想 在MySQL8.0 版本中继续使用旧版本中的认证方式需要在 my.cnf 文件中配置并重启，因为此参数不可动态修改。

```mysql
mysql> set global default_authentication_plugin='mysql_native_password';
ERROR 1238 (HY000): Variable 'default_authentication_plugin' is a read only variable
```

写入my.cnf文件后重启MySQL（测试无效？？？）：
```bash
vim my.cnf
[mysqld]
default_authentication_plugin=mysql_native_password
```

另一种解决方法：兼容新老版本的认证方式。
```mysql
ALTER USER 'sptest'@'%' IDENTIFIED WITH mysql_native_password BY 'sptest';

FLUSH PRIVILEGES;
```