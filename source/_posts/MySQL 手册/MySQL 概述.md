---
title: MySQL 概述

categories:
- MySQL 手册

date: 2020-07-07 00:00:15
---
MySQL 概述
MySQL 安装
MySQL 数据类型
MySQL 数据索引
MySQL 常用函数
MySQL 操作语句
MySQL 查询语句
MySQL 存储引擎
MySQL 锁定机制
MySQL 事务机制
MySQL 备份恢复
MySQL 读写分离
MySQL 性能调优

## 概要
1. 怎样才能找出最后一次插入时分配了哪个自动增量
1. MySQL 有关权限的表都有哪几个

1. 如何通俗地理解三个范式
1. 数据库三范式及判断、E-R 图
1. MySQL范式和反范式的区别以及彼此的优缺点
1. 数据库的范式








## 表与视图
1. 什么是基本表
1. 什么是视图
1. 什么是游标
1. 试述视图的优点
1. 什么叫视图，游标是什么
1. 完整性约束包括哪些
1. drop、truncate、delete 区别









## 常见问题
1. 抛出异常 java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for column 'name' at row 1

数据库字段、表、数据库、MySQL 的编码需要设置成utf8mb4、数据库连接设置编码

```sql
show variables like "%char%";
```

我的问题是，阿里云数据的chareacer_set_server=utf8，修改即可。