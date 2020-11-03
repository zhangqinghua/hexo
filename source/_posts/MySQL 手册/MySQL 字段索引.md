---
title: MySQL 字段索引

categories:
- MySQL 手册

date: 2020-07-07 00:00:12
---
索引的建立对于 MySQL 的高效运行是很重要的，索引可以大大提高 MySQL 的检索速度。打个比方，如果合理的设计且使用索引的 MySQL 是一辆兰博基尼的话，那么没有设计和使用索引的 MySQL 就是一个人力三轮车。

## 索引种类
从原理上，索引分为聚集索引和非聚集索：
1. 聚集索引
   索引中键值的逻辑顺序决定了表中相应行的物理顺序（索引中的数据物理存放地址和索引的顺序是一致的），可以这么理解：只要是索引是连续的，那么数据在存储介质上的存储位置也是连续的。

   InnoDB 引擎会为每张表都加一个聚集索引，而聚集索引指向的的数据又是以物理磁盘顺序来存储的，自增的主键会把数据自动向后插入，避免了插入过程中的聚集索引排序问题。如果对聚集索引进行排序，这会带来磁盘IO性能损耗是非常大的。

   如果一个主键被定义了，那么这个主键就是作为聚集索引。

   如果没有主键被定义，那么该表的第一个唯一非空索引被作为聚集索引。

   如果没有主键也没有合适的唯一索引，那么 InnoDB 内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个 6 个字节的列，改列的值会随着数据的插入自增。

1. 非聚集索引
   索引的逻辑顺序与磁盘上的物理存储顺序不同。非聚集索引的键值在逻辑上也是连续的，但是表中的数据在存储介质上的物理顺序是不一致的，即记录的逻辑顺序和实际存储的物理顺序没有任何联系。索引的记录节点有一个数据指针指向真正的数据存储位置。



从功能上，索引可以分成以下 6 种：
1. 普通索引
   最基本的索引，它没有任何限制。
1. 唯一索引
   它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。
1. 主键索引
   是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引。
1. 组合索引
   指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合。
1. 全文索引
   主要用来查找文本中的关键字，而不是直接与索引中的值相比较。全文索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的 `WHERE` 语句的参数匹配。

## 优点缺点
优点：
1. 索引可以大大提高MySQL的检索速度。

缺点：
1. 建立索引会占用磁盘空间的索引文件。
1. 降低更新表的速度，如对表进行 `INSERT`、`UPDATE` 和 `DELETE`。因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件。

## 注意事项
1. 使用短索引
   对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个 `CHAR(255)` 的列，如果在前 10 个或 20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和 I/O 操作。
1. 谨慎使用 `LIKE` 查询
   一般情况下不推荐使用 `LIKE` 操作，如果非使用不可，如何使用也是一个问题。`LIKE "%aaa%"` 不会使用索引而 `LIKE "aaa%"` 可以使用索引。
1. 列不能包含 `null` 值
   只要列中包含有 `null` 值都将不会被包含在索引中，复合索引中只要有一列含有 `null` 值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为 `null`。
1. 不要在列上进行运算
   这将导致索引失效而进行全表扫描，例如：`SELECT * FROM table_name WHERE YEAR(column_name)<2017`。
1. 多索引查询只有一个生效
   查询只使用一个索引，因此如果 `WHERE` 子句中已经使用了索引的话，那么 `OEDER BY` 中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。参考：[数据库中查询记录时是否每次只能使用一个索引](https://www.jianshu.com/p/34194ea5a4a3)。


## 索引操作
查看索引：

```sql
-- 使用 SHOW KEYS 查看索引
SHOW KEYS FROM biz_user;
-- 使用 SHOW INDEX 语句查看索引
SHOW INDEX FROM biz_user;

Table	      Non_unique	Key_name	Seq_in_index	Column_name       ...
biz_goods     0	                PRIMARY	        1	        id                ...
biz_goods     1	                cate_id_key	1	        cate_id           ...

-- 使用 SHOW CREATE TABLE 语句查看索引
SHOW CREATE TABLE biz_goods;

CREATE TABLE `biz_goods` (
  `id` bigint(20) NOT NULL,
  `cate_id` bigint(20) NOT NULL COMMENT '分类Id',
  `detail` mediumtext NOT NULL COMMENT '商品详情',
  PRIMARY KEY (`id`),
  KEY `cate_id_key` (`cate_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品表';
```

创建索引：

```sql
-- 普通索引：直接创建索引
CREATE INDEX cate_id_key ON `biz_goods`(cate_id);

-- 普通索引：创建表的时候同时创建索引
CREATE TABLE `biz_goods` (
  `id` bigint(20) NOT NULL,
  `cate_id` bigint(20) NOT NULL COMMENT '分类Id',
  `detail` mediumtext NOT NULL COMMENT '商品详情',
  PRIMARY KEY (`id`),
  KEY `cate_id_key` (`cate_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品表';

-- 唯一索引：直接创建索引
CREATE UNIQUE INDEX cate_id_key ON `biz_goods`(cate_id);

-- 唯一索引：创建表的时候同时创建索引
CREATE TABLE `biz_goods` (
  `id` bigint(20) NOT NULL,
  `cate_id` bigint(20) NOT NULL COMMENT '分类Id',
  `detail` mediumtext NOT NULL COMMENT '商品详情',
  PRIMARY KEY (`id`),
  UNIQUE `cate_id_key` (`cate_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品表';

-- 主键索引：一般是在建表的时候同时创建
CREATE TABLE `biz_goods` (
  `id` bigint(20) NOT NULL,
  PRIMARY KEY (`id`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品表';

-- 全文索引：直接创建索引
CREATE FULLTEXT INDEX detail_key ON `biz_goods`(detail);

-- 全文索引：创建表的时候同时创建索引
CREATE TABLE `biz_goods` (
  `id` bigint(20) NOT NULL,
  `detail` mediumtext NOT NULL COMMENT '商品详情',
  PRIMARY KEY (`id`),
  FULLTEXT (detail)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='商品表';
```

修改索引：

```sql
-- 普通索引：修改表结构的方式添加索引
ALTER TABLE `biz_goods` ADD INDEX cate_id_key ON (cate_id);

-- 组合索引：直接创建索引
ALTER TABLE `biz_goods` ADD INDEX cate_id_name_key (cate_id, name);

-- 唯一索引：修改表结构的方式添加索引
ALTER TABLE `biz_goods` ADD UNIQUE cate_id_key ON (cate_id);

-- 全文索引：修改表结构的方式添加索引
ALTER TABLE `biz_goods` ADD FULLTEXT detail_key(detail); 
```

删除索引：

```sql
-- 使用 DROP INDEX 命令删除索引
DROP INDEX cate_id_key on `biz_goods`;

-- 使用 ALTER TABLE 命令删除索引
ALTER TABLE `biz_goods` DROP INDEX cate_id_key;
```


## 最左前缀
在 MySQL 建立联合索引时会遵守最左前缀匹配原则，即最左优先，在检索数据时从联合索引的最左边开始匹配。



## 索引原理
实际上，索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。







## 附：超键、主键、外键、候选键
**超键**是指在关系中能唯一标识元组的属性集。在下面的数据中，「学号」、「学号，性别」、「学号，年龄」等可以组成一个超键。

**候选键**是不含多余属性的超键为候选键。在下面的数据中，「学号」是一个候选键，而「学号，性别」不是候选键，因为它的性别属性是多余的。

**主键**是用户选择的候选键作为该元组的唯一标识。

**外键**是在一张表里持有另一张表的主键。

|学号       |姓名       |性别   |年龄   |系别   |专业
|:-         |
|20020612   |李辉       |男     |20     |计算机 |软件开发
|20060613   |张明       |男     |18     |计算机 |软件开发 
|20060614   |王小玉     |女     |19     |物理   |力学
|20060615   |李淑华     |女     |17     |生物   |动物学
|20060616   |赵静       |男     |21     |化学   |食品化学
|20060617   |赵静       |女     |20     |生物   |植物学











## 问题
1. 主键、超键和候选键有什么区别
1. 你怎么看到为表格定义的所有索引
1. 可以使用多少列创建索引
1. 索引的底层实现原理和优化
1. 简单描述MySQL中，索引，主键，唯一索引，联合索引的区别，对数据库的性能有什么影响（从读写两方面）
1. 索引的目的是什么
1. 索引对数据库系统的负面影响是什么
1. 为数据表建立索引的原则有哪些
1. 什么情况下不宜建立索引
1. 主键、外键和索引的区别？
1. 索引分类，最左前缀原则，哪些条件无法使用索引
1. B 树、B+ 树区别，索引为何使用 B+ 树
1. 聚集索引与非聚集索引（使用非聚集索引的查询过程）
1. SQL优化，常用的索引？
1. 查询中哪些情况不会使用索引？
1. 数据库索引，底层是怎样实现的，为什么要用B树索引？
1. MySQL 是怎么用B+树？
1. MySQL 索引类别有哪些，什么是覆盖索引
1. 数据库索引有哪些？底层怎么实现的？数据库怎么优化？
1. 聚簇索引&非聚簇索引
1. 索引什么时候会失效变成全表扫描
1. 索引的类型，索引的底层实现原理
1. MySQL索引设计，联合索引，SQL 语句优化，abc 索引，搜索b，会使用索引吗（走索引要回表）
1. MySQL索引实现，如何解决慢查询
1. 用过MySQL吗？为啥加索引会变快？聚簇型索引和非聚簇型索引的区别？
1. 说一聚簇索引和非聚簇索引的有什么不同
1. 数据库（最多的还是MySQL，Nosql有redis）索引（包括分类及优化方式，失效条件，底层结构）
