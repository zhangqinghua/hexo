---
title: MySQL 存储引擎

categories:
- MySQL 手册

date: 2020-07-07 00:00:07
---
数据库存储引擎是数据库底层软件组件，数据库管理系统使用数据引擎进行创建、查询、更新和删除数据操作。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎还可以获得特定的功能。

## MyISAM
MyISAM 是基于 ISAM 的存储引擎，并对其进行扩展，是在 Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM 拥有较高的插入、查询速度，但不支持事务。

## InnoDB
InnoDB 事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键。MySQL 5.5.5 之后，InnoDB 作为默认存储引擎。

## Memory
Memory 存储引擎将表中的数据存储到内存中，为查询和引用其他数据提供快速访问。

## 存储引擎的操作
显示可用的数据库引擎和默认引擎：
```sql
SHOW ENGINES;

Engine      Support  Transactions   XA       Savepoints  Comment
MyISAM      YES      NO             NO       NO          MyISAM storage engine
InnoDB      DEFAULT  YES            YES      YES         Supports transactions, row-level locking, and foreign keys
MEMORY      YES      NO             NO       NO          Hash based, stored in memory, useful for temporary tables
```

临时修改默认存储引擎：

```sql
SET default_storage_engine = MyISAM;
```

全局修改默认存储引擎：
1. 打开 MySQL 配置文件：/etc/my.cnf。
1. 添加或新增行 `default-storage-engine=INNODB`。

查看某个存储引擎的具体信息：

```sql
SHOW ENGINE InnoDB status\G;
```

![](http://img.mp.sohu.com/upload/20170713/e972e47b2c554f789e02e90b26a8b543_th.png)

创建表时指定存储引擎的类型：
```sql
CREATE TABLE mytable (id int, titlechar(20)) ENGINE = INNODB
```

修改表的存储引擎：

```sql
ALTER TABLE engineTest ENGINE = INNODB；
```

## 存储引擎的选择
可以根据以下的原则来选择 MySQL 存储引擎：
1. 如果要提供提交、回滚和恢复的事务安全（ACID 兼容）能力，并要求实现并发控制，InnoDB 是一个很好的选择。
1. 如果数据表主要用来插入和查询记录，则 MyISAM 引擎提供较高的处理效率。
1. 如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存的 Memory 引擎中，MySQL 中使用该引擎作为临时表，存放查询的中间结果。
1. 如果只有 `INSERT` 和 `SELECT` 操作，可以选择 Archive 引擎，Archive 存储引擎支持高并发的插入操作，但是本身并不是事务安全的。Archive 存储引擎非常适合存储归档数据，如记录日志信息可以使用 Archive 引擎。

|功能|MySIAM|InnoDB|Memory|Archive|
| :- |
|存储限制|256TB|64TB|RAM|None|
|事务支持|No|Yes|No|No|
|数据缓存|No|Yes|N/A|No|
|外健支持|No|Yes|No|No|
|数值索引|Yes|Yes|Yes|No|
|哈希索引|No|No|Yes|No|
|全文索引|Yes|Yes|No|No|
|行锁支持|表锁|表锁、行锁|

## 问题
1. 有哪些存储引擎
1. 简述 MyISAM 和 InnoDB 的区别
1. MyISAM Static 和 MyISAM Dynamic 有什么区别
1. MyISAM 表格将在哪里存储，并且还提供其存储格式
1. MySQL 默认存储引擎？MyISAM、InnoDB、MEMORY的区别
1. MySQL 数据库默认存储引擎，有什么优点
1. MySQL引擎及区别，项目用的哪个，为什么
1. MySQL采用了什么存储引擎，为什么？
1. MySQL数据库引擎？应用场景？查询优化？NoSQL有用或了解吗？
1. MySQL用的什么存储引擎，这个存储引擎用的什么数据结构 ，有哪些优缺点，怎么使用
1. 引擎对比（InnoDB，MyISAM）