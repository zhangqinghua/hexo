---
title: Chapter 4.1 数据库

categories:
- Interview

date: 2020-04-28 00:00:41
---

## 索引
#### 什么是索引
索引就是将表中某几个字段提取出来，开辟新的存储空间并进行排序，并且把所有值和rowid存储其中，并用一个指针指向表中原来对应行的记录。

#### 索引的实现
MySQL 目前主要有以下几种索引实现：HASH、BTREE，RTREE、FULLTEXT。
- HASH
    由于 HASH 的唯一（几乎 100% 的唯一）及类似键值对的形式，很适合作为索引。
    HASH 索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。但是，这种高效是有条件的，即只在“=”和“in”条件下高效，对于范围查询、排序及组合索引仍然效率不高。
- BTREE
    BTREE 索引就是一种将索引值按一定的算法，存入一个树形的数据结构中（二叉树），每次查询都是从树的入口 root 开始，依次遍历 node，获取 leaf。这是 MySQL 里默认和最常用的索引类型。
- RTREE
    RTREE 在 MySQL 很少使用，仅支持 geometry 数据类型，支持该类型的存储引擎只有 MyISAM、BDb、InnoDb、NDb、Archive 几种。
- FULLTEXT


#### 索引的种类
- 普通索引
    在 MySQL 中执行查询时，只能使用一个索引。如果我们在多个字段上分别建索引，执行查询时，MySQL 会选择一个最严格（获得结果集记录数最少）的索引。
- 唯一索引
    类似普通索引，索引列的值必须唯一（可以为空，这点和主键索引不同）
- 主键索引
    特殊的唯一索引，不允许为空，只能有一个，一般是在建表时指定
- 组合索引
    在多个字段上创建索引，遵循最左前缀原则，就是最左优先，先要看第一列，在第一列满足的条件下再看左边第二列，以此类推。
    ```sql
    create table test(
        id1 int ,
        id2 int,
        id3 int,
        id4 int,
        key index_id12(id1,id2));

    # 用到索引
    explain select * from test where id1 < 10;
    # 用到索引
    explain select * from test where id1 < 10 and id2 > 1; 
    # 用到索引
    explain select * from test where id2 > 1 and id1 < 2; 
    # 未用到索引
    explain select * from test where id2 > 1;
    # 未用到索引
    explain select * from test order by id1 desc ;
    # 用到索引
    explain select id1 from test order by id1 desc ;
    explain select id1,id2 from test order by id1 desc ;
    # 未用到索引
    explain select id1,id2,id3 from test order by id1 desc ;
    ```
- 全文索引
    即为全文索引，目前只有 MyISAM 引擎支持。其可以在 CREATE TABLE、ALTER TABLE、CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR、TEXT 列上可以创建全文索引。

    全文索引并不是和 MyISAM 一起诞生的，它的出现是为了解决 WHERE name LIKE “%word%" 这类针对文本的模糊查询效率较低的问题。
    
#### 索引的优缺点
- 
- 建立索引会占用磁盘空间

#### 何时使用索引
- 主键、unique字段
- 和其他表做连接的字段（或者说外建）需要加索引
- 在 where 里使用 ＞，≥，＝，＜，≤，is null和 between 等字段
- 使用不以通配符开始的 like，例如 where A like 'China%'
- 聚集函数 min()，max() 中的字段
- order by 和 group by 字段

#### 何时不使用索引
- 表中记录较少
- 不作为 where 查询条件时
- 数据重复且分布平均的字段（只有很少数据值的列）
- 经常插入、删除、修改的表要减少索引
- text，image 等类型不应该建立索引，这些列的数据量大

#### 索引何时失效
- 组合索引未使用最左前缀，例如组合索引（A，B），where B = b 不会使用索引
- like 未使用最左前缀，where A like '%China'
- 搜索一个索引而在另一个索引上做 order by，where A = a order by B，只使用 A 上的索引，因为查询只使用一个索引
- or 会使索引失效。如果查询字段相同，也可以使用索引。例如where A = a1 or A = a2（生效），where A = a or B = b（失效）
- 如果列类型是字符串，要使用引号。例如 where A='China'，否则索引失效（会进行类型转换）
- 在索引列上的操作，函数（upper() 等）、or、!=、<>、not in 等

#### 索引会不会使插入、删除效率变低？怎么解决

#### 什么是 B+ 树

## 事务
#### 什么是事务

#### 事务的特点是什么

#### 事务的隔离级别有哪些

#### 什么是脏读

#### 什么是幻读

#### 什么是不可重复读

## 分布式事务
#### 什么是分布式事务

#### CAS 是什么

#### 什么是 Compare and Swap

#### Compare and Swap 为什么能保证原子性

#### 分布式锁的实现有哪些

