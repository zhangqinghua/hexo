---
title: MySQL 性能调优

categories:
- MySQL 手册

date: 2020-07-07 00:00:00
---
## 性能指标

## 性能分析
#### show status
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

#### Query Profiler
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

#### explain
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

## 数据对比

## test
https://www.cnblogs.com/claireyuancy/p/7258314.html


## 问题
1. 优化数据库的方法
1. 实践中如何优化 MySQL
1. MySQL 如何优化 DISTINCT
1. MySQL 数据库作发布系统的存储，一天五万条以上的增量，预计运维三年,怎么优化
1. 锁的优化策略
1. 什么情况下设置了索引但无法使用
1. SQL注入漏洞产生的原因和如何防止
1. 说说对SQL语句优化有哪些方法？（选择几条）
1. 查询缓慢和解决方式（explain、慢查询日志、show profile等）
1. SQL优化，常用的索引？
1. 数据库性能调优如何做
1. 如何做的 MySQL 优化
1. MySQL 数据库优化会涉及到哪些？
1. 从SQL、JVM、架构、数据库四个方面讲讲优化思路，以及如何优先排序？
1. 四个表 记录成绩，每个大约十万条记录，如何找到成绩最好的同学
1. MySQL的慢 SQL 优化一般如何来做？除此外还有什么方法优化？
1. MySQL如何获取慢SQL，以及慢查询的解决方式
1. 数据库崩溃时事务的恢复机制
1. 在工作中，SQL语句的优化和注意的事项
1. 数据库万级变成亿级，你如何来解决。
1. MySQL数据库引擎？应用场景？查询优化？NoSQL有用或了解吗？
1. Redis 和数据库如何保证数据一致性
1. 谈谈MySQL的查询优化方法，重点谈谈优化步骤。
1. 如何防止sql注入，了解哪些加密算法，rsa过程说下
1. MySQL的常见优化方式、定为慢查询
1. 优化（explain，慢查询，show profile）
1. myisamchk 是用来做什么的