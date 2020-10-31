---
title: MySQL 语言分类

categories:
- MySQL 手册

date: 2020-07-07 00:00:10
---
SQL 语言分为数据控制语言、数据定义语言、数据操纵语言和数据查询语言四大类。

## 数据控制语言 DCL
DCL 用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。

#### 创建用户

#### 删除用户

#### 授予用户权限
对于新建的 MySQL 用户，必须给它授权，可以用 `GRANT` 语句来实现对新建用户的授权。

```sql
-- 授予数据库db1的所有权限给指定账户
GRANT ALL ON db1.* TO 'user1'@'localhost';
-- 授予角色给指定的账户
GRANT 'role1', 'role2' TO 'user1'@'localhost', 'user2'@'localhost';
```

若要权限生效，需要执行以下语句：

```sql
FLUSH PRIVILEGES;
```

关于授予权限的明细请参考[这里](https://www.cnblogs.com/Rohn/p/11722515.html)

#### 查看用户权限
当成功创建用户账户后，还不能执行任何操作，需要为该用户分配适当的访问权限。可以使用 `SHOW GRANTS FOR` 语句来查询用户的权限。例如：

```sql
mysql> SHOW GRANTS FOR test;
+-------------------------------------------+
| Grants for test@%                         |
+-------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' |
+-------------------------------------------+
1 row in set (0.00 sec)
```

#### 撤销用户权限
`REVOKE` 语句主要用于撤销权限，语法跟 `GRANT` 相似，但是效果相反。

> 若要使用REVOKE语句，必须拥有 MySQL 数据库的全局 CREATE USER 权限或 UPDATE 权限。

## 数据定义语言 DDL
数据定义语言有 `CREATE`、`DROP`、`ALTER`等，对逻辑结构等有操作的，其中包括表结构、视图和索引等。

## 数据操纵语言 DML
如 `INSERT`、`UPDATE`、`DELETE` 等，即对数据进行操作。

## 数据查询语言 DQL
即查询操作，以 `SELECT` 关键字开头，各种简单查询，连接查询等都属于 DQL。

