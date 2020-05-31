---
title: Chapter 4.1 数据库

categories:
- Interview

date: 2020-04-28 00:00:41
---
## 最全 MySQL 面试 60 题和答案
#### MySQL 中有哪几种锁？
1. 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
2. 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
3. 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

#### MySQL中有哪些不同的表格？
共有5种类型的表格：
1. MyISAM
1. InnoDB
1. MEMORY
1. MRG_MYISAM
1. CSV
1. ARCHIVE
1. BLACKHOLE
1. PERFORMANCE_SCHEMA

#### 简述在 MySQL 数据库中 MyISAM 和 InnoDB 的区别
MyISAM：
1. 不支持事务，但是每次查询都是原子的；

1. 支持表级锁，即每次操作是对整个表加锁；

1. 存储表的总行数；

1. 一个MYISAM表有三个文件：索引文件、表结构文件、数据文件；

1. 采用菲聚集索引，索引文件的数据域存储指向数据文件的指针。辅索引与主索引基本一致，但是辅索引不用保证唯一性。

InnoDB：
1. 支持ACID的事务，支持事务的四种隔离级别；

1. 支持行级锁及外键约束：因此可以支持写并发；

1. 不存储总行数；

1. 一个InnoDb引擎存储在一个文件空间（共享表空间，表大小不受操作系统控制，一个表可能分布在多个文件里），也有可能为多个（设置为独立表空，表大小受操作系统文件大小限制，一般为2G），受操作系统文件大小的限制；

1. 主键索引采用聚集索引（索引的数据域存储数据文件本身），辅索引的数据域存储主键的值；因此从辅索引查找数据，需要先通过辅索引找到主键值，再访问辅索引；最好使用自增主键，防止插入数据时，为维持B+树结构，文件的大调整。

#### MySQL 中 InnoDB 支持的四种事务隔离级别名称，以及逐级之间的区别？
SQL标准定义的四个隔离级别为：
1. read uncommited ：读到未提交数据

1. read committed：脏读，不可重复读

1. repeatable read：可重读

1. serializable ：串行事物

#### CHAR 和 VARCHAR 的区别？
1. CHAR 和 VARCHAR 类型在存储和检索方面有所不同

2. CHAR 列长度固定为创建表时声明的长度，长度值范围是 1 到 255，当 CHAR 值被存储时，它们被用空格填充到特定长度，检索 CHAR 值时需删除尾随空格。

#### 主键和候选键有什么区别？
1. 表格的每一行都由主键唯一标识,一个表只有一个主键。

1. 主键也是候选键。按照惯例，候选键可以被指定为主键，并且可以用于任何外键引用。

#### myisamchk 是用来做什么的？
它用来压缩 MyISAM 表，这减少了磁盘或内存使用。

#### MyISAM Static 和 MyISAM Dynamic 有什么区别？
在 MyISAM Static 上的所有字段有固定宽度。动态 MyISAM 表将具有像 TEXT，BLOB 等字段，以适应不同长度的数据类型。

MyISAM Static 在受损情况下更容易恢复。

#### 如果一个表有一列定义为TIMESTAMP，将发生什么？
每当行被更改时，时间戳字段将获取当前时间戳。

#### 列设置为AUTO INCREMENT时，如果在表中达到最大值，会发生什么情况？
它会停止递增，任何进一步的插入都将产生错误，因为密钥已被使用。

#### 怎样才能找出最后一次插入时分配了哪个自动增量？
LAST_INSERT_ID 将返回由 auto_increment 分配的最后一个值，并且不需要指定表名称。

#### 你怎么看到为表格定义的所有索引？
```sql
show index from table_name;
```

#### liek 声明中的 ％ 和 _ 是什么意思？
％ 对应于 0 个或更多字符，_ 只是 like 语句中的一个字符。

#### 如何在 Unix 和 MySQL 时间戳之间进行转换？
UNIX_TIMESTAMP 是从 MySQL 时间戳转换为 Unix 时间戳的命令
FROM_UNIXTIME 是从 Unix 时间戳转换为 MySQL 时间戳的命令

#### 列对比运算符是什么？
在SELECT语句的列比较中使用=，<>，<=，<，> =，>，<<，>>，<=>，AND，OR或LIKE运算符。

#### BLOB 和 TEXT 有什么区别？
1. BLOB 是一个二进制对象，可以容纳可变数量的数据。TEXT 是一个不区分大小写的 BLOB。

1. BLOB 和 TEXT 类型之间的唯一区别在于对BLOB值进行排序和比较时区分大小写，对 TEXT 值不区分大小写。

#### mysql_fetch_array 和 mysql_fetch_object 的区别是什么？
1. mysql_fetch_array – 将结果行作为关联数组或来自数据库的常规数组返回。

1. mysql_fetch_object – 从数据库返回结果行作为对象。

#### MyISAM 表格将在哪里存储，并且还提供其存储格式？
每个MyISAM表格以三种格式存储在磁盘上：
1. “.frm”文件存储表定义
1. 数据文件具有“.MYD”（MYData）扩展名
1. 索引文件具有“.MYI”（MYIndex）扩展名

#### MySQL 如何优化 DISTINCT？
DISTINCT 在所有列上转换为 GROUP BY，并与 ORDER BY 子句结合使用。

```sql
select distinct t1.a from t1, t2 where t1.a = t2.a;
```

#### 如何显示前 50 行？
```sql
select * from table_name limit 0, 50;
```

#### 可以使用多少列创建索引？
任何标准表最多可以创建 16 个索引列。

#### NOW() 和 CURRENT_DATE() 有什么区别？
1. NOW() 命令用于显示当前年份，月份，日期，小时，分钟和秒。
1. CURRENT_DATE() 仅显示当前年份，月份和日期。

#### 什么是非标准字符串类型？
1. TINYTEXT
1. TEXT
1. MEDIUMTEXT
1. LONGTEXT

#### 什么是通用 SQL 函数？
1. CONCAT(A, B) – 连接两个字符串值以创建单个字符串输出。通常用于将两个或多个字段合并为一个字段。
1. FORMAT(X, D)- 格式化数字X到D有效数字。
1. CURRDATE(), CURRTIME()- 返回当前日期或时间。
1. NOW（） – 将当前日期和时间作为一个值返回。
1. MONTH（），DAY（），YEAR（），WEEK（），WEEKDAY（） – 从日期值中提取给定数据。
1. HOUR（），MINUTE（），SECOND（） – 从时间值中提取给定数据。
1. DATEDIFF（A，B） – 确定两个日期之间的差异，通常用于计算年龄
1. SUBTIMES（A，B） – 确定两次之间的差异。
1. FROMDAYS（INT） – 将整数天数转换为日期值。

#### MySQL 支持事务吗？
在缺省模式下，MySQL 是 autocommit 模式的，所有的数据库更新操作都会即时提交，所以在缺省情况下，MySQL 是不支持事务的。

但是如果你的 MySQL 表类型是使用 InnoDB Tables 或 BDB tables的话，你的 MySQL 就可以使用事务处理,使用 SET
AUTOCOMMIT=0 就可以使 MySQL 允许在非 autocommit 模式，在非 autocommit 模式下，你必须使用 COMMIT 来提交你的更改，或者用 ROLLBACK 来回滚你的更改。

#### MySQL 里记录货币用什么字段类型好
NUMERIC 和 DECIMAL 类型被 MySQL 实现为同样的类型，这在 SQL92 标准允许。他们被用于保存值，该值的准确精度是极其重要的值，例如与金钱有关的数据。当声明一个类是这些类型之一时，精度和规模的能被(并且通常是)指定。

例如：
```sql
salary DECIMAL(9,2)
```

在这个例子中，9 代表将被用于存储值的总的小数位数，而 2 代表将被用于存储小数点后的位数。

因此，在这种情况下，能被存储在salary列中的值的范围是从 -9999999.99 到 9999999.99。

#### MySQL 有关权限的表都有哪几个？
MySQL 服务器通过权限表来控制用户对数据库的访问，权限表存放在 MySQL 数据库里，由 mysql_install_db 脚本初始化。这些权限表分别 user，db，table_priv，columns_priv 和 host。

#### 列的字符串类型可以是什么？
字符串类型是：
1. SET
1. BLOB
1. ENUM
1. CHAR
1. TEXT

#### MySQL 数据库作发布系统的存储，一天五万条以上的增量，预计运维三年,怎么优化？
1. 选择合适的表字段数据类型和存储引擎，适当的添加索引。
1. MySQL 库主从读写分离。
1. 找规律分表，减少单表中的数据量提高查询速度。
1. 添加缓存机制，比如 memcached，apc 等。
1. 不经常改动的页面，生成静态页面。
1. 书写高效率的SQL。比如 SELECT * FROM TABEL 改为 SELECT field_1, field_2, field_3 FROM TABLE.

## 108道Java面试题:工作10年，面试超过500人想进阿里总结出的!

#### 数据库三范式及判断、E-R 图

#### InnoDB 和 MyISAM 存储引擎的区别

#### 索引分类，最左前缀原则，哪些条件无法使用索引

#### B 树、B+ 树区别，索引为何使用 B+ 树

#### 聚集索引与非聚集索引（使用非聚集索引的查询过程）

#### 事务的ACID（原子性、一致性、隔离性、持久性）

#### 事务隔离级别和各自存在的问题（脏读、不可重复读、幻读）和解决方式（间隙锁及MVCC）

#### 乐观锁和悲观锁、行锁与表锁、共享锁与排他锁

#### MVCC（增加两个版本号）及delete、update、select时的具体控制

#### 死锁判定原理和具体场景

#### 查询缓慢和解决方式（explain、慢查询日志、show profile等）

#### drop、truncate、delete 区别

#### 查询语句不同元素（where、jion、limit、group by、having等等）执行先后顺序

#### mysql优化，读写分离、主从复制

#### 数据库崩溃时事务的恢复机制（REDO 日志和 UNDO 日志）

## 2019 最全支付宝高级Java现场面试37题
#### SQL优化，常用的索引？

#### 数据库性能调优如何做

#### 数据库事务属性

## 2019 最全阿里天猫Java 3面真题，含面试题答案！
无

## 2019 最新阿里中间件Java 4轮面试题！60万年薪起步~
#### 数据库索引，表锁；乐观锁；悲观锁

## 2019蚂蚁金服 Java面试题目！涵盖现场3面真题
#### 如何做的 MySQL 优化

## 阿里蚂蚁金服中间件(Java 4轮面试题含答案)：Redis缓存+线程锁+微服务等
#### MySQL 事务隔离级别以及 MVCC 机制