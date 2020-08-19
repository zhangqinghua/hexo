---
title: MySQL 性能分析

categories:
- MySQL 手册

date: 2020-07-07 00:00:01
---
MySQL 数据库性能遇到瓶颈，如何快速定位问题的原因，是每个 DBA 或系统运维人员应该思考的问题。正确的借助一些性能分析工具，能够帮助 DBA 或系统运维人员进行问题快速的定位。

## 性能指标
下面是 MySQL 数据库，或者说所有数据库的三个关键性能指标：
1. QPS 每秒处理的查询数
1. TPS 每秒处理的事务数
1. IOPS 每秒磁盘进行的I/O操作次数

#### QPS

#### TPS
Transactions Per Second，即服务器每秒处理的事务数，适用 InnoDB 存储引擎。

TPS 是软件测试结果的测量单位。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数。

一般的，评价系统性能均以每秒钟完成的技术交易的数量来衡量。系统整体处理能力取决于处理能力最低模块的 TPS 值。


???

## show status
`show status` 命令可以查看 MySQL 服务器的状态信息。

```sql
- 查询当前MySQL本次启动后的运行统计时间
show status like 'uptime';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Uptime        | 5667  |
+---------------+-------+
1 row in set (0.00 sec)

- 本次MySQL启动后执行的SELECT语句的次数
show status like 'com_select';
- 本次MySQL启动后执行的INSERT语句的次数
show status like 'com_insert';
- 本次MySQL启动后执行的UPDATE语句的次数
show status like 'com_update';
- 本次MySQL启动后执行的DELETE语句的次数
show status like 'com_delete';

+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Com_select    | 1     |
+---------------+-------+
1 row in set (0.00 sec)

- 查看MySQL服务器的线程信息
show status like 'Thread_%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 0     |
| Threads_connected | 1     |
| Threads_created   | 1     |
| Threads_running   | 1     |
+-------------------+-------+
4 rows in set (0.00 sec)

- 查看试图连接到MySQL(不管是否连接成功)的连接数
show status like 'connections';

- 查看当前打开的连接的数量。
show status like 'threads_connected';

- 查看当前打开的连接的数量。
show status like 'threads_connected';

- 查看创建用来处理连接的线程数。如果Threads_created较大，你可能要增加thread_cache_size值。
show status like 'threads_created';

- 查看激活的(非睡眠状态)线程数。
show status like 'threads_running';

- 查看立即获得的表的锁的次数。
show status like 'table_locks_immediate';

- 查看不能立即获得的表的锁的次数。如果该值较高，并且有性能问题，你应首先优化查询，然后拆分表或使用复制。
show status like 'table_locks_waited';

- 查看创建时间超过slow_launch_time秒的线程数。
show status like 'slow_launch_threads';

- 查看查询时间超过long_query_time秒的查询的个数。
show status like 'slow_queries';
```

## Query Profiler
Query Profiler 是 MySQL 自带的一种查询诊断分析工具，通过它可以分析出一条SQL语句的性能瓶颈在什么地方。

查询类型包括：
1. ALL 显示所有性能信息
1. BLOCK IO 显示块IO操作的次数
1. CONTEXT SWITCHES 显示上下文切换次数，不管是主动还是被动
1. CPU 显示用户CPU时间、系统CPU时间
1. IPC 显示发送和接收的消息数量
1. MEMORY [暂未实现]
1. PAGE FAULTS 显示页错误数量
1. SOURCE 显示源码中的函数名称与位置
1. SWAPS 显示SWAP的次数

结果参数说明：
1. System lock
    确认是由于哪个锁引起的，通常是因为MySQL或InnoDB内核级的锁引起的建议：如果耗时较大再关注即可，一般情况下都还好
1. Sending data
    从server端发送数据到客户端，也有可能是接收存储引擎层返回的数据，再发送给客户端，数据量很大时尤其经常能看见
    备注：Sending Data不是网络发送，是从硬盘读取，发送到网络是Writing to net
    建议：通过索引或加上LIMIT，减少需要扫描并且发送给客户端的数据量
1. Sorting result
    正在对结果进行排序，类似Creating sort index，不过是正常表，而不是在内存表中进行排序
    建议：创建适当的索引
1. Table lock
    表级锁，没什么好说的，要么是因为MyISAM引擎表级锁，要么是其他情况显式锁表
1. create sort index
    当前的SELECT中需要用到临时表在进行ORDER BY排序
    建议：创建适当的索引

示例：

```sql
# 查看是否开启了Profile功能，0未开启 1开启
> select @@profiling;

# 开启Profile功能
> set profiling=1;

# 测试查询语句
> select * from biz_sku

# 查询此语句的queryId
> show profiles;
351	0.00053725	select * from biz_sku

# 查询此queryId的性能消耗
> show  profile for query 351;

# 查询此queryId指定资源类型的性能消耗
> show  profile cpu ,swaps for query 351;

Status	                Duration	CPU_user	CPU_system	Swaps
starting	        0.000021	0.000000	0.000000	0    
checking permissions	0.000007	0.000000	0.000000	0
Opening tables	        0.000015	0.000000	0.000000	0
init	                0.000026	0.000000	0.000000	0
System lock	        0.000007	0.000000	0.000000	0
optimizing	        0.000003	0.000000	0.000000	0
statistics	        0.000009	0.000000	0.000000	0
preparing	        0.000008	0.000000	0.000000	0
executing	        0.000002	0.000000	0.000000	0
Sending data	        0.000481	0.000000	0.000000	0
end	                0.000005	0.000000	0.000000	0
query end	        0.000006	0.000000	0.000000	0
closing tables	        0.000012	0.000000	0.000000	0
freeing items	        0.000011	0.000000	0.000000	0
cleaning up	        0.000003	0.000000	0.000000	0
```

## explain
MySQL 提供了一个 explain 命令用来对 select 语句进行分析，并输出执行的详细信息，以供开发人员针对性优化。

使用：

```sql
mysql> explain select * from servers;
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | servers | ALL  | NULL          | NULL | NULL    | NULL |    1 | NULL  |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
1 row in set (0.03 sec)
```

各列的含义如下：
1. **id** 
    SQL执行的顺序的标识，SQL从大到小的执行。如果id相同，则从上到下。
1. **select_type** 
    表示查询中每个select子句的类型。
1. **table** 
    显示这一行的数据是关于哪张表的，有时不是真实的表名字，看到的是derivedx（x是个数字，我的理解是第几步执行的结果）
1. **type** 
    表示MySQL在表中找到所需行的方式，又称“访问类型”。
1. **possible_keys**  
    指出 MySQL 能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用。
1. **Key** 
    显示 MySQL 实际决定使用的键（索引）
1. **key_len** 
    表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）
1. **ref** 
    表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
1. **rows** 
    表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数
1. **extra** 
    该列包含MySQL解决查询的详细信息

**select_type** 可选值:
1. SIMPLE
    简单SELECT,不使用UNION或子查询等
1. PRIMARY
    查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY
1. UNION
    UNION中的第二个或后面的SELECT语句
1. DEPENDENT
    UNION UNION中的第二个或后面的SELECT语句，取决于外面的查询
1. UNION
    RESULT (UNION的结果
1. SUBQUERY
    子查询中的第一个SELECT
1. DEPENDENT
    SUBQUERY 子查询中的第一个SELECT，取决于外面的查询
1. DERIVED
    派生表的SELECT, FROM子句的子查询
1. UNCACHEABLE SUBQUERY
    一个子查询的结果不能被缓存，必须重新评估外链接的第一行

**type** 可选值：
1. all
    Full Table Scan， MySQL将遍历全表以找到匹配的行
1. index 
    Full Index Scan，index与ALL区别为index类型只遍历索引树
1. range
    只检索给定范围的行，使用一个索引来选择行
1. ref
    表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
1. eq_ref
    类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
1. const、system
    当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，使用system
1. null
    MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

**extra** 可选值：
1. Using where
    列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤。
1. Using temporary
    表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询。
1. Using filesort
    MySQL中无法利用索引完成的排序操作称为“文件排序”。
1. Using join buffer
    改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。
1. Impossible where
    这个值强调了where语句会导致没有符合条件的行。
1. Select tables optimized away
    这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行。

详细介绍：https://segmentfault.com/a/1190000008131735

## 慢日志查询
慢查询日志用来记录在 MySQL 中执行时间超过指定时间的查询语句。通过慢查询日志，可以查找出哪些查询语句的执行效率低，以便进行优化。

通俗的说，MySQL 慢查询日志是排查问题的 SQL 语句，以及检查当前 MySQL 性能的一个重要功能。如果不是调优需要，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。

#### 查询慢日志配置
```sql
-- 查看是否开启慢查询日志功能
show variables LIKE 'slow_query%';
+---------------------+---------------------------------------------------------------------+
| Variable_name       | Value                                                               |
+---------------------+---------------------------------------------------------------------+
| slow_query_log      | OFF                                                                 |
| slow_query_log_file | C:\ProgramData\MySQL\MySQL Server 5.7\Data\LAPTOP-UHQ6V8KP-slow.log |
+---------------------+---------------------------------------------------------------------+
2 rows in set, 1 warning (0.02 sec)

-- 查询超过多少秒才记录
show variables LIKE 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set, 1 warning (0.01 sec)
```

#### 开启慢日志查询
使用命令开启慢日志查询，只对当前数据库生效，重启MySQL失效。

```sql
-- 开启慢日志查询
set global slow_query_log = 1;

set global slow_query_log = ON;

-- 慢查询界定时间（秒）
set global long_query_time = 3;
```

如果需要永久生效，修改 MySQL 配置文件后重启。

```cnf
[mysqld]

# 开启慢日志查询
slow_query_log = 1

# 慢日志文件存放位置
log-slow-queries=dir\filename

# 慢查询界定时间（秒）
long_query_time=n
```
#### 关闭慢日志查询
使用命令关闭慢日志查询，只对当前数据库生效，重启MySQL失效。
```sql
-- 关闭慢日志查询
set global slow_query_log = 0;
```

如果需要永久生效，修改 MySQL 配置文件后重启。
```cnf
[mysqld]

# 关闭慢日志查询
slow_query_log = 0
```

#### 其他操作
执行一条 SQL 语句。

```sql
USE test;
Database changed

SELECT * FROM tb_student;
+----+--------+
| id | name   |
+----+--------+
|  1 | Java   |
|  2 | MySQL  |
|  3 | Python |
+----+--------+
3 rows in set (0.08 sec)
```

打开慢日志文件，可以看到：

```
# Time: 2020-06-01T01:59:18.368780Z
# User@Host: root[root] @ localhost [::1]  Id:     3
# Query_time: 0.006281  Lock_time: 0.000755 Rows_sent: 2  Rows_examined: 1034
use test;
SET timestamp=1590976758;
SHOW VARIABLES LIKE 'slow_query%';
```

清空慢日志记录：

```bash
mysqladmin -uroot -p flush-logs
```

执行该命令后，命令行会提示输入密码。输入正确密码后，将执行删除操作。新的慢查询日志会直接覆盖旧的查询日志，不需要再手动删除。

数据库管理员也可以手工删除慢查询日志，删除之后需要重新启动 MySQL 服务。

> 注意：通用查询日志和慢查询日志都是使用这个命令，使用时一定要注意，一旦执行这个命令，通用查询日志和慢查询日志都只存在新的日志文件中。如果需要备份旧的慢查询日志文件，必须先将旧的日志改名，然后重启 MySQL 服务或执行 `mysqladmin` 命令。

## explain 优化案例
#### 表结构
```sql
CREATE TABLE `a`
(
    `id`          int(11) NOT NULLAUTO_INCREMENT,
    `seller_id`   bigint(20)                                       DEFAULT NULL,
    `seller_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL,
    `gmt_create`  varchar(30)                                      DEFAULT NULL,
    PRIMARY KEY (`id`)
);
CREATE TABLE `b`
(
    `id`          int(11) NOT NULLAUTO_INCREMENT,
    `seller_name` varchar(100) DEFAULT NULL,
    `user_id`     varchar(50)  DEFAULT NULL,
    `user_name`   varchar(100) DEFAULT NULL,
    `sales`       bigint(20)   DEFAULT NULL,
    `gmt_create`  varchar(30)  DEFAULT NULL,
    PRIMARY KEY (`id`)
);
CREATE TABLE `c`
(
    `id`         int(11) NOT NULLAUTO_INCREMENT,
    `user_id`    varchar(50)  DEFAULT NULL,
    `order_id`   varchar(100) DEFAULT NULL,
    `state`      bigint(20)   DEFAULT NULL,
    `gmt_create` varchar(30)  DEFAULT NULL,
    PRIMARY KEY (`id`)
);
```

#### 待优化 SQL
三张表关联，查询当前用户在当前时间前后10个小时的订单情况，并根据订单创建时间升序排列，具体SQL如下：

```sql
select a.seller_id,
       a.seller_name,
       b.user_name,
       c.state
from a,
     b,
     c
where a.seller_name = b.seller_name
  and b.user_id = c.user_id
  and c.user_id = 17
  and a.gmt_create
    BETWEEN DATE_ADD(NOW(), INTERVAL – 600 MINUTE)
    AND DATE_ADD(NOW(), INTERVAL 600 MINUTE)
```
#### 数据量
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2R2lhY21zTjgySE1JZFh5aWJWWjFOZmhmeUxJOGlhWUl4SU54UFdxa1Nsalh5UFlxSHJOMVU2MWljUS82NDA_d3hfZm10PXBuZw?x-oss-process=image/format,png)

#### 原执行时间
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2M0pydlYyMlNWbENuRU1HZm1jSXBxaWNOWGliV2I4elBVbjdLMUVURnIzNWNXdmliQWYzQUlDRjV3LzY0MD93eF9mbXQ9cG5n?x-oss-process=image/format,png)

#### 原实行计划
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2YnB2ZVlBUGxqaGlhWVpIaWNWeTh1cW1ycWc1T1l1V0FGM2FURDR0Q3hZaWFUQkVpYVpoUGRKcFIxQS82NDA_d3hfZm10PXBuZw?x-oss-process=image/format,png)

#### 初步优化思路
1. SQL 中 `where` 条件字段类型要跟表结构一致，表中 `user_id` 为 `varchar(50)` 类型，实际 SQL 用的 `int` 类型，存在隐式转换，也未添加索引。将 b 和 c 表 `user_id` 字段改成 `int` 类型。
1. 因存在 `b` 表和 `c` 表关联，将 `b` 和 `c` 表 `user_id` 创建索引
1. 因存在 `a` 表和 `b` 表关联，将 `a` 和 `b` 表 `seller_name` 字段创建索引
1. 利用复合索引消除临时表和排序

#### 初步优化SQL
```sql
alter table b modify `user_id` int(10) DEFAULT NULL;
alter table c modify `user_id` int(10) DEFAULT NULL;
alter table c add index `idx_user_id`(`user_id`);
alter table b add index `idx_user_id_sell_name`(`user_id`,`seller_name`);
alter table a add index `idx_sellname_gmt_sellid`(`gmt_create`,`seller_name`,`seller_id`);
```

#### 优化后执行时间
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2empYMm04ZG4yUDZvbXNTZkwxSFg2UFZKVlVjQUlsR1ZzSTB5cWw1ZWJNdTZIaGROV292aWJSQS82NDA_d3hfZm10PXBuZw?x-oss-process=image/format,png)

#### 优化后执行计划
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2dHRsV3U3cENLQlBmZWtPdXdzbnZPOEZpY3gxRmRFbFRSRU9NWlVzUDY2MmV6eWhudXRiNHltZy82NDA_d3hfZm10PXBuZw?x-oss-process=image/format,png)

#### 查看 warnings 信息
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2OUk5V29JM1RKRkdrSzJ5cG1YWDdYc2FpY2NzVjhqVDdCUjU5d0JyeTV3a0htRzJ2ZmhMU2Nrdy82NDA_d3hfZm10PXBuZw?x-oss-process=image/format,png)

#### 继续优化 gmt_create
```sql
alter table a modify "gmt_create" datetime DEFAULT NULL;
```

#### 查看执行时间
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2UUFNcHh4TzI5R0huOEwxbmhYbmJTc2JBOFRHa2ljY1dpYVBsS3ptNWhLb1k3Q1RpYVRWU1o3U0ZnLzY0MD93eF9mbXQ9cG5n?x-oss-process=image/format,png)

#### 查看执行计划
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9FbnpBd3ZwVFFKNzY5TTZaaWNvbjRWVGdpY2FMdTNyN0s2enNxUFlXQWlhc0RySThYZXMzT2xGVmV1eXlpY0hmZ0ZhYXVPNlNpYTBCRVhqekhCTjB6RGQzNmliQS82NDA_d3hfZm10PXBuZw?x-oss-process=image/format,png)

#### 总结
1. 查看执行计划 `explain`
1. 如果有告警信息，查看告警信息 `show warnings`
1. 查看SQL涉及的表结构和索引信息
1. 根据执行计划，思考可能的优化点
1. 按照可能的优化点执行表结构变更、增加索引、SQL 改写等操作
1. 查看优化后的执行时间和执行计划
1. 如果优化效果不明显，重复第四步操作