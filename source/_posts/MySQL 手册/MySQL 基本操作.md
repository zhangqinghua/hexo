---
title: MySQL 基本操作

categories:
- MySQL 手册

date: 2020-07-07 00:00:12
---

## binlog 日志
binlog 记录了所有的 DDL 和 DML （除了数据查询语句）语句，以事件形式记录，还包含语句所执行的消耗的时间，MySQL 的二进制日志是事务安全型的。

#### 常用操作
查看所有binlog日志列表。

```sql
show master logs;
```

查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值。

```sql
show master status;
```

```
刷新log日志，自此刻开始产生一个新编号的binlog日志文件。

```sql
flush logs;
```

> 注：每当 MySQL 服务重启时，会自动执行此命令，刷新binlog日志；在mysqldump备份数据时加 -F 选项也会刷新binlog日志；

重置(清空)所有binlog日志。

```sql
reset master;
```

查询 binlog 设置：
```sql
show global variables like 'log_bin%';

log_bin	                            ON
log_bin_basename	            /var/lib/mysql/binlog
log_bin_index	                    /var/lib/mysql/binlog.index
log_bin_trust_function_creators	    OFF
log_bin_use_v1_row_events	    OFF
```

#### 开启 binlog
MySQL8.0 默认开始 binlog。

#### 关闭 binlog
MySQL8.0 需要在 my.cnf 配置上：

```
[mysqld]
skip-log-bin
```

然后重启。

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
