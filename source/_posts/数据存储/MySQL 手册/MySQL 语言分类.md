---
title: MySQL 语言分类

categories:
- 数据存储
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

`CREATE` 语句可以用来创建数据库、表和索引。


```sql
-- 创建一个名为 "my_db" 的数据库
CREATE DATABASE my_db;

-- 创建一个名为 "Persons" 的表，包含五列：PersonID、LastName、FirstName、Address 和 City。
CREATE TABLE Persons (
   PersonID int,
   LastName varchar(255),
   FirstName varchar(255),
   Address varchar(255),
   City varchar(255)
);

-- 在 "Persons" 表的 "LastName" 列上创建一个名为 "PIndex" 的索引
CREATE INDEX PIndex ON Persons (LastName)

-- 如果索引不止一个列，您可以在括号中列出这些列的名称，用逗号隔开
CREATE INDEX PIndex ON Persons (LastName, FirstName)
```

`ALTER` 语句用于在已有的表中添加、删除或修改列。

```sql
-- 在 "Persons" 表中添加一个名为 "DateOfBirth" 的列
ALTER TABLE Persons ADD DateOfBirth date

-- 改变 "Persons" 表中 "DateOfBirth" 列的数据类型
ALTER TABLE Persons ALTER COLUMN DateOfBirth year

-- 删除 "Person" 表中的 "DateOfBirth" 列
ALTER TABLE Persons DROP COLUMN DateOfBirth

-- 删除 "Persons" 表中 "DateOfBirth" 列的索引
ALTER TABLE Persons DROP INDEX DateOfBirthIndex
```

`DROP` 语句可以轻松地删除索引、表和数据库。

```sql
-- 删除数据库
DROP DATABASE database_name

-- 删除表
DROP TABLE table_name

-- 删除列
ALTER TABLE table_name DROP COLUMN column_name

-- 删除索引
ALTER TABLE table_name DROP INDEX index_name
```

## 数据操纵语言 DML
如 `INSERT`、`UPDATE`、`DELETE`、`TRUNCATE` 等，即对数据进行操作。

`DELETE` 语句用于删除表中的记录。

```sql
-- 删除所有数据
DELETE FROM table_name;

-- 删除指定数据
DELETE FROM table_name WHERE some_column=some_value;
```

`TRUNCATE` 跟没有 `WHERE` 子句的 `DELETE` 类似，但是速度更快，系统资源和事务日志资源消耗更少。

```sql
-- 删除所有数据
TRUNCATE FROM table_name;
```

## 数据查询语言 DQL
即查询操作，以 `SELECT` 关键字开头，各种简单查询，连接查询等都属于 DQL。


#### 执行顺序
一般 SQL 执行顺序：
1. `FROM` 需要从哪个数据表检索数据。
1. `WHERE` 过滤表中数据的条件。
1. `GROUP BY` 如何将上面过滤出的数据分组。
1. `HAVING` 对上面已经分组的数据进行过滤的条件。
1. `SELECT` 查看结果集中的哪个列，或列的计算结果。
1. `ORDER` 按照什么样的顺序来查看返回的数据。

当一个查询语句同时出现了 `WHERE`、`JOIN`、`LIMIT`、`GROUP BY`、`HAVING` 的时候，执行顺序和编写顺序是：
1. 执行 `JOIN` 连接条件。
1. 执行 `WHERE xxx` 对全表数据做筛选，返回第1个结果集。
1. 针对第 1 个结果集使用 `GROUP BY` 分组，返回第 2 个结果集。
1. 针对第 2 个结果集中的每1组数据执行 `SELECT xx`，有几组就执行几次，返回第 3 个结果集。
1. 针对第 3 个结集执行 `HAVING xxx` 进行筛选，返回第 4 个结果集。
1. 针对第 4 个结果集排序（`ORDER BY`、`LIMIT`）。

## 面试题
`DROP`、`DELETE`、`TRUNCATE` 的区别
1. 定义：`DROP` 是 DDL 语句，`DELETE`、`TRUNCATE` 是 DML语句。
1. 区别：`TRUNCATE` 跟没有 `WHERE` 子句的 `DELETE` 类似，但是速度更快，系统资源和事务日志资源消耗更少。
1. 效率：`DROP` > `TRUNCATE` > `DELETE`。
1. 用途：不需要表时使用 `DROP`，保留表不需要数据时使用 `TRUNCATE`，仅删除部分数据使用 `DELETE`。