---
title: Chapter 7.1 SQL 大全

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:71
---
#### test

## 创建索引
#### 单列索引
单列索引基于单一的字段创建，其基本语法如下所示：

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

#### 唯一索引
唯一索引不止用于提升查询性能，还用于保证数据完整性。唯一索引不允许向表中插入任何重复值。其基本语法如下所示：

```sql
CREATE UNIQUE INDEX index_name
on table_name (column_name);
```

#### 聚簇索引
聚簇索引在表中两个或更多的列的基础上建立。其基本语法如下所示：

```sql
CREATE INDEX index_name
on table_name (column1, column2);
```

#### 隐式索引
隐式索引由数据库服务器在创建某些对象的时候自动生成。例如，对于主键约束和唯一约束，数据库服务器就会自动创建索引。

## 删除索引
引可以用 SQL **DROP** 命令删除。删除索引时应当特别小心，数据库的性能可能会因此而降低或者提高。

```sql
DROP INDEX table_name.index_name;
```