---
title: Chapter 5.3 索引的操作

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:53

mathjax: true
---
#### 创建索引
```sql
-- 单列索引
CREATE INDEX index_name ON table_name (column_name);

-- 唯一索引
CREATE UNIQUE INDEX index_name on table_name (column_name);

-- 联合索引
CREATE INDEX index_name on table_name (column1, column2);

-- 全文索引
CREATE TABLE table_name (
    id INT(11) NOT NULL AUTO_INCREMENT,
    column1 VARCHAR(255),
    column2 TEXT NOT NULL,

    PRIMARY KEY (id),
    FULLTEXT KEY index_name(column1, column2)  
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

CREATE FULLTEXT INDEX index_name on table_name(column1, column2);

ALTER TABLE table_name ADD FULLTEXT INDEX index_name(column1, column2);
```

#### 修改索引
修改个 O，直接删掉重建。

#### 删除索引
删除索引时应当特别小心，数据库的性能可能会因此而降低或者提高。

```sql
-- 使用 DROP INDEX 命令删除索引
DROP INDEX table_name.index_name;

-- 使用 ALTER 命令删除主键
ALTER TABLE table_name DROP PRIMARY KEY;

-- 使用 ALTER 命令删除索引
ALTER TABLE table_name DROP INDEX index_name;
```

#### 显示索引信息
你可以使用 SHOW INDEX 命令来列出表中的相关的索引信息。可以通过添加 **\G** 来格式化输出信息。
```sql
SHOW INDEX FROM table_name;
```