---
title: MySQL 性能优化

categories:
- 数据存储
- MySQL 手册

date: 2020-07-07 00:00:00
---
## 优化的目标
1. QPS
1. TPS
1. 吞吐量
1. 响应时间
1. 减少 IO 次数 
    IO 永远是数据库最容易瓶颈的地方，大部分数据库操作中超过90%的时间都是 IO 操作所占用的，减少 IO 次数是SQL 优化中需要第一优先考虑，当然，也是收效最明显的优化手段。
1. 降低 CPU 计算 
    除了 IO 瓶颈之外，SQL优化中需要考虑的就是 CPU 运算量的优化了。order by, group by,distinct … 都是消耗 CPU 的大户（这些操作基本上都是 CPU 处理内存中的数据比较运算）。当我们的 IO 优化做到一定阶段之后，降低 CPU 计算也就成为了我们 SQL 优化的重要目标

## 优化的原则
数据库优化可以分成 4 个方面来讲：
1. SQL 语句和索引的优化
1. 数据表的优化
1. 存储引擎的优化
1. 系统配置的优化
1. 硬件和架构的优化

其中，数据结构、SQL、索引是成本最低，且效果最好的优化手段。而硬件的优化则成本比较高。

![](https://p1htmlkernalweb.mybluemix.net/image/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzg0OTAxNzMvYXJ0aWNsZS9kZXRhaWxzLzEwNDk5NzUwMQ==_aHR0cHM6Ly9pbWctYmxvZy5jc2RuaW1nLmNuLzIwMjAwMzIwMjAwNDM2NjY3LnBuZz94LW9zcy1wcm9jZXNzPWltYWdlL3dhdGVybWFyayx0eXBlX1ptRnVaM3BvWlc1bmFHVnBkR2ssc2hhZG93XzEwLHRleHRfYUhSMGNITTZMeTlpYkc5bkxtTnpaRzR1Ym1WMEwzRnhYek00TkRrd01UY3osc2l6ZV8xNixjb2xvcl9GRkZGRkYsdF83MA==)

漏斗法则优化法可以归纳为5个层次：
1. 减少数据访问（减少磁盘访问）
2. 返回更少数据（减少网络传输或磁盘访问）
3. 减少交互次数（减少网络传输）
4. 减少服务器 CPU 开销（减少 CPU 及内存开销）
5. 利用更多资源（增加资源）

由于每一层优化法则都是解决其对应硬件的性能问题，所以带来的性能提升比例也不一样。传统数据库系统设计是也是尽可能对低速设备提供优化方法，因此针对低速设备问题的可优化手段也更多，优化成本也更低。

我们任何一个SQL的性能优化都应该按这个规则由上到下来诊断问题并提出解决方案，而不应该首先想到的是增加资源解决问题。

以下是每个优化法则层级对应优化效果及成本经验参考：

|优化法则|性能提升效果|优化成本|
| :- |
|减少数据访问|1~1000|低|
|返回更少数据|1~100|低|
|减少交互次数|1~20|低|
|减少服务器 CPU 开销|1~5|低|
|利用更多资源|@~10|高|

![](http://img.shangdixinxi.com/up/info/202006/20200607132814970915.png)

![](https://ucc.alicdn.com/pic/developer-ecology/76f2a3150468463ea197396326949fb0.png)

## SQL 优化
#### 优化 SQL 的一般步骤
1. 定位慢 SQL
    通过 `show status` 命令了解各种SQL的执行效率。

1. 定位 SQL 瓶颈
    通过 `explain` 或 `desc` 分析低效SQL的执行计划。

    通过 `show profile` 分析 SQL。

    通过 `trace` 分析优化器如何选择执行计划。

1. 优化 SQL
    运用 SQL 优化原则来优化 SQL。

1. 优先优化高并发的 SQL，而不是执行频率低的某些大 SQL
    对于破坏性来说，高并发的 SQL 总是会比低频率的来得大，因为高并发的 SQL 一旦出现问题，甚至不会给我们任何喘息的机会就会将系统压跨。而对于一些虽然需要消耗大量 IO 而且响应很慢的 SQL，由于频率低，即使遇到，最多就是让整个系统响应慢一点，但至少可能撑一会儿，让我们有缓冲的机会。

    尽可能对每一条运行在数据库中的 SQL 进行 `explain`。

#### SQL 优化原则
锁优化

索引优化

select 优化

join 优化

where 优化


1. 避免查询全部字段
    只查必要的字段。如果需要全部字段，避免使用 `select *`，把所有字段名列出来。

1. 避免全盘扫描
    在 `where`，`group by`，`order by`，`on` 等从句涉及到的列上建立索引

    避免在 `where` 子句中使用 `!=` 或 `<>` 操作符，否则将引擎放弃使用索引而进行全表扫描。

    避免在 `where` 子句中对字段进行 `null` 值判断，否则将导致引擎放弃使用索引而进行全表扫描。如有必要可以用 0 代替 `null`。

    避免在 `where` 子句中使用 `or` 来连接条件。如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描。

    避免使用前导模糊查询，否则将导致全表扫描。例如 `name like '%c%'` 改成 `name like '%c%'`。

    慎用 `not in`，否则会导致全表扫描。对于连续的数值，能用 `between` 就不要用 `in` 了，尽量使用 `exists` 代替 `in`。

    避免在 `where` 子句中对字段进行表达式与函数或其他表达式运算操作，这将导致引擎放弃使用索引而进行全表扫描。例如 `where num / 2 = 100` 改成 `where num = 100 * 2`。

    使用 `limit` 对查询结果的记录进行限定，单条查询最后添加 `limit 1`，停止全表扫描。

1. 锁优化
    MyISAM：MyISAM 使用表锁，可以考虑转成
1. 优化子查询
    使用子查询需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作。

    用 `join` 代替子查询。虽然 `join` 性能并不佳，但是和 MySQL 的子查询比起来还是有非常大的性能优势。

1. 优化 `join` 查询
    尽量少使用 `join`。

    确认两个表中 `join` 的字段是被建过索引的。这样，MySQ L内部会启动为你优化 `join` 的 SQL 语句的机制。

    确认两个表中 `join` 的字段是相同类型的。例如：如果你要把 `decimal` 字段和一个 `int` 字段 `join` 在一起，MySQL就无法使用它们的索引。对于那些字符类型的字段，还需要有相同的字符集才行（两个表的字符集有可能不一样）。

1. 优化 `rand` 查询
    实现随机选挑一行数据的功能，不建议使用 `rand` 函数，性能很低。可以先统计总行数，然后计算出一个随机数，再使用 `limit` 查询。

    例如 `select id from biz_user order by rand() limit 1` 可以改成 `select id from biz_user limit 100,1`。

1. 优化 `count` 查询
    注意  `count(1)` 和 `count(name)` 的区别。前者统计所有行，后者只统计字段值不为 `null` 的行。

1. 优化 `limit` 查询
    结果只会有一条结果，加上 `limit 1` 可以增加性能。这样一样，MySQL 数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据。

1. 优化 `union` 查询
    `union` 需要将两个（或者多个）结果集合并后再进行唯一性过滤操作，这就会涉及到排序，增加大量的 CPU 运算，加大资源消耗及延迟。
    
    所以当我们可以确认不可能出现重复结果集或者不在乎重复结果集的时候，尽量使用 `union all` 而不是 `union`。

1. 优化 `distinct` 查询
    用途：查询某个字段不重复的记录。

    原因：对最终结果集完成一次排序，这是成本最昂贵的排序之一。

    功能优化：因为 `distinct` 只能返回他的目标字段，而无法返回其他字段。如果我们希望返回其目标字段所在的记录，可以使用 `group by` 代替，例如：`select distinct(name) from biz_user` 替换为 `select id, name from biz_user group by name`。 
    
    性能优化：`SELECT DISTINCT e.empno, e.lastname FROM emp e, empproject ep WHERE e.empno=ep.empno` 可以替换成为 `SELECT e.empno, e.lastname from emp e, empproject ep WHERE e.empno=ep.empno GROUP BY e.empno,e.lastname`。

    还可以使用子查询代替：`SELECT e.empno, e.lastname from emp e WHERE EXISTS (select 1 from empproject where e.empno=empno)`。

    还可以使用 `in` 字句查询：`SELECT e.empno, e.lastname from emp e where e.empno in (select empno from empproject)`。

    提升效率比：??

    https://blog.csdn.net/u010745238/article/details/42846897 ??

1. 优化查询缓存
    像 `now()`、 `rand()`、`curdate()` 或是其它的诸如此类的 SQL 函数都不会开启查询缓存，因为这些函数的返回是会不定的易变的。所以，你所需要的就是用一个变量来代替 MySQL 的函数，从而开启缓存。

    例如 `where gmtmodify >= now()` 可以改成 `where gmtmodify >= '2020-08-10'`。

1. 尽量少排序
    排序操作会消耗较多的 CPU 资源，所以减少排序可以在缓存命中率高等 IO 能力足够的场景下会较大影响 SQL 的响应时间。
    
    对于 MySQL 来说，减少排序有多种办法，比如：减少参与排序的记录条数，非必要不对数据进行排序。

1. 尽量早过滤
    将过滤性更好的字段放得更靠前。在 SQL 编写中同样可以使用这一原则来优化一些 `join` 的 SQL。比如我们在多个表进行分页数据查询的时候，我们最好是能够在一个表上先过滤好数据分好页，然后再用分好页的结果集与另外的表 `join`，这样可以尽可能多的减少不必要的 IO 操作，大大节省 IO 操作所消耗的时间。

1. 避免类型转换
    这里所说的“类型转换”是指 `where` 子句中出现字段的类型和传入的参数类型不一致的时候发生的类型转换。

1. 避免复杂 SQL
    提升可阅读性；避免慢查询的概率；可以转换成多个短查询，用业务端处理。


查询分析方法
观察，至少跑1天，看看生产的慢SQL情况。

开启慢查询日志，设置阈值，比如超过5秒钟的就是慢SQL，并将它抓取出来。

explain+慢SQL分析

show profile

运维经理 or DBA，进行SQL数据库服务器的参数调优。


in 和 exists
小表驱动大表，即小的数据集驱动大的数据集。

当B表的数据集必须小于A表的数据集时，用in优于exists。

当A表的数据集系小于表的数据集时，用exists优于in。

提高Order By的速度
Order by时select *是一个大忌，只Query需要的字段，这点非常重要。 在这里的影响是:

1.1 当Query的字段大小总和小于max_length_for_sort_data 而且排序字段不是TEXT|BLOB类型时，会用改进后的算法——单路排序，否则用老算法——多路排序。

1.2 两种算法的数据都有可能超出sort_buffer的容量， 超出之后，会创建tmp文件进行合并排序，导致多次I/O，但是用单路排序算法的风险会更大一些，所以要提高sort_buffer_size 。

尝试提高sort_buffer_size
不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的

尝试提高max_length_for_sort_data
提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率。

Group By
group by实质是先排序后进行分组，遵照索引建的最佳左前缀

当无法使用索引列，增大max_ length_ for_ sort_data参数的设置+增大sort_buffer_size参数的设置

where高于having，能写在where限定的条件就不要去having限定了。


## ??
1. 固定长度的表会更快 
    “static” 或 “fixed-length”



## 表结构优化
#### 选择适当数据类型
1. 数字类型
    能确定不会使用负数的字段，建议添加 `unsigned` 定义。

    合理使用 `tinyint`、`int`、`bigint`。因为三者所占用的存储空间也有很大的差别。

    不要使用 `double`，不仅仅只是存储长度的问题，同时还会存在精确性的问题。
    
    固定精度的小数，也不建议使用 `decimal`，建议乘以固定倍数转换成整数存储，可以大大节省存储空间，且不会带来任何附加维护成本。

1. 字符类型
    非万不得已不要使用 `text` 数据类型，其处理方式决定了他的性能要低于 `char` 或者是 `varchar` 类型的处理。

    定长字段，建议使用 `char` 类型，不定长字段尽量使用 `varchar`，且仅仅设定适当的最大长度，而不是非常随意的给一个很大的最大长度限定，因为不同的长度范围，MySQL 也会有不一样的存储处理。

    对于状态字段，以尝试使用 `enum` 来存放，因为可以极大的降低存储空间。
    
    如果是存放可预先定义的属性数据呢？可以尝试使用 `set` 类型，即使存在多种属性，同样可以游刃有余，同时还可以节省不小的存储空间。
    
    尽量避免在数据库中存储二进制数据。可以采用第三方的对象存储服务。

1. 时间类型
    不建议通过 `int` 类型类存储一个 Unix 时间戳的值，因为这太不直观，会给维护带来不必要的麻烦，同时还不会带来任何好处。
    
    尽量使用 `timestamp` 类型，因为其存储空间只需要 `datetime` 类型的一半。
    
    对于只需要精确到某一天的数据类型，建议使用 `date` 类型，因为他的存储空间只需要 3个字节，比 `timestamp` 还少。

1. 字符编码
    建议统一使用 utf8mb4 字符编码，兼容绝大多数文字和表情。

#### 精简字段
对于大多数的数据库引擎来说，硬盘操作可能是最重大的瓶颈。所以，把你的数据变得紧凑会对这种情况非常有帮助，因为这减少了对硬盘的访问。

如果一个表只会有几列罢了（比如说字典表，配置表），那么，我们就没有理由使用 INT 来做主键，使用 MEDIUMINT, SMALLINT 或是更小的 TINYINT 会更经济一些。如果你不需要记录时间，使用 DATE 要比 DATETIME 好得多。

当然，你也需要留够足够的扩展空间，不然，你日后来干这个事，你会死的很难看，参看Slashdot的例子（2009年11月06日），一个简单的ALTER TABLE语句花了3个多小时，因为里面有一千六百万条数据。

#### 适当拆分
将表中大部分访问都不需要用到的字段，和类似于 `text` 或者是很大的 `varchar` 类型的大字段，拆分到另外的独立表中。以减少常用数据所占用的存储空间，每个数据块中可以存储的数据条数可以大大增加，既减少物理 IO 次数，也能大大提高内存中的缓存命中率。

#### 垂直分割
“垂直分割”是一种把数据库中的表按列变成几张表的方法，这样可以降低表的复杂度和字段的数目，从而达到优化的目的。（以前，在银行做过项目，见过一张表有100多个字段，很恐怖）

示例一：在Users表中有一个字段是家庭地址，这个字段是可选字段，相比起，而且你在数据库操作的时候除了个人信息外，你并不需要经常读取或是改写这个字段。那么，为什么不把他放到另外一张表中呢？ 这样会让你的表有更好的性能，大家想想是不是，大量的时候，我对于用户表来说，只有用户ID，用户名，口令，用户角色等会被经常使用。小一点的表总是会有好的性能。

示例二： 你有一个叫 “last_login” 的字段，它会在每次用户登录时被更新。但是，每次更新时会导致该表的查询缓存被清空。所以，你可以把这个字段放到另一个表中，这样就不会影响你对用户ID，用户名，用户角色的不停地读取了，因为查询缓存会帮你增加很多性能。

另外，你需要注意的是，这些被分出去的字段所形成的表，你不会经常性地去Join他们，不然的话，这样的性能会比不分割时还要差，而且，会是极数级的下降。

#### 适度冗余
适当冗余被频繁引用的字段，避免减少联表查询。

#### 避免 null 字段
当字段中存在 `null` 值时会影响索引查询的效率。可以设置为空字符串或 0 代替。

#### 分区？？
这里的分区和上面的适当拆分、垂直分割不同，是对应用层透明的。

分区是根据一定的规则，数据库把一个表分解成多个更小的、更容易管理的部分，是一种水平划分。对应用来说是完全透明的，不影响应用的业务逻辑，即不用修改代码。因此能存更多的数据，查询，删除也支持按分区来操作，从而达到优化的目的。

分区可按以下四种类型分区：
1. RANGE表分区：范围表分区，按照一定的范围值来确定每个分区包含的数据;
1. LIST表分区：列表表分区，按照一个一个确定的值来确定每个分区包含的数据;
1. HASH表分区：哈希表分区，按照一个自定义的函数返回值来确定每个分区包含的数据;
1. KEY表分区 ：key表分区，与哈希表分区类似，只是用MySQL自己的HASH函数来确定每个分区包含的数据。

如果有考虑分区，可以提前做准备，避免下列一些限制：
1. 如果分区字段中有主键或者唯一索引列，那么所有主键列和唯一索引列都必须包含进来，如果表中有主键或唯一索引，那么分区键必须是主键或唯一索引
1. 分区表中无法使用外键约束
1. NULL值会使分区过滤无效
1. 目前mysql不支持空间类型和临时表类型进行分区。不支持全文索引
1. 所有分区必须使用相同的存储引擎

#### 分表？？
这里的分表和上面的适当拆分、垂直分割不同，是对应用层透明的。

分表分水平分表和垂直分表。

水平分表即拆分成数据结构相同的各个小表，如拆分成 table1, table2...，从而缓解数据库读写压力。

垂直分表即将一些字段分出去形成一个新表，各个表数据结构不相同，可以优化高并发下锁表的情况。

可想而知，分表的话，程序的逻辑是需要做修改的，所以，一般是在项目初期时，预见到大数据量的情况，才会考虑分表。后期阶段不建议分表，成本很大。

#### 分库？？
分库一般是主从模式，一个数据库服务器主节点复制到一个或多个从节点多个数据库，主库负责写操作，从库负责读操作，从而达到主从分离，高可用，数据备份等优化目的。

分库分表最佳实践：https://cloud.tencent.com/developer/article/1448626

## 锁的优化
http://c.biancheng.net/cpp/html/1481.html

https://cloud.tencent.com/developer/news/456119

#### 获取锁等待情况
可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺：
mysql> show status like 'Table%';
+----------------------------+----------+
| Variable_name       | Value |
+----------------------------+----------+
| Table_locks_immediate   | 105       |
| Table_locks_waited  | 3     |
+----------------------------+----------+
2 rows in set (0.00 sec)
 
可以通过检查Innodb_row_lock状态变量来分析系统上的行锁的争夺情况：
mysql> show status like 'innodb_row_lock%';
+----------------------------------------+----------+
| Variable_name               | Value |
+----------------------------------------+----------+
| Innodb_row_lock_current_waits   | 0     |
| Innodb_row_lock_time            | 2001  |
| Innodb_row_lock_time_avg        | 667       |
| Innodb_row_lock_time_max    | 845       |
| Innodb_row_lock_waits       | 3     |
+----------------------------------------+----------+
5 rows in set (0.00 sec)
 
另外，针对Innodb类型的表，如果需要察看当前的锁等待情况，可以设置InnoDB Monitors，然后通过Show innodb status察看，设置的方式是：
    CREATE TABLE innodb_monitor(a INT) ENGINE=INNODB;
监视器可以通过发出下列语句来被停止：
    DROP TABLE innodb_monitor;
设置监视器后，在show innodb status的显示内容中，会有详细的当前锁等待的信息，包括表名、锁类型、锁定记录的情况等等，便于进行进一步的分析和问题的确定。打开监视器以后，默认情况下每15秒会向日志中记录监控的内容，如果长时间打开会导致.err文件变得非常的巨大，所以我们在确认问题原因之后，要记得删除监控表以关闭监视器。或者通过使用--console选项来启动服务器以关闭写日志文件。

#### 对 MySIAM 类型的表
1) Myisam类型的表可以考虑通过改成Innodb类型的表来减少锁冲突。

2) 根据应用的情况，尝试横向拆分成多个表或者改成Myisam分区对减少锁冲突也会有一定的帮助。

#### 对 Innodb 类型的表
1) 首先要确认，在对表获取行锁的时候，要尽量的使用索引检索纪录，如果没有使用索引访问，那么即便你只是要更新其中的一行纪录，也是全表锁定的。要确保sql是使用索引来访问纪录的，必要的时候，请使用explain检查sql的执行计划，判断是否按照预期使用了索引。

2) 由于mysql的行锁是针对索引加的锁，不是针对纪录加的锁，所以虽然是访问不同行的纪录，但是如果是相同的索引键，是会被加锁的。应用设计的时候也要注意，这里和Oracle有比较大的不同。

3) 当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，当表有主键或者唯一索引的时候，不是必须使用主键或者唯一索引锁定纪录，其他普通索引同样可以用来检索纪录，并只锁定符合条件的行。

4) 用SHOW INNODB STATUS来确定最后一个死锁的原因。查询的结果中，包括死锁的事务的详细信息，包括执行的SQL语句的内容，每个线程已经获得了什么锁，在等待什么锁，以及最后是哪个线程被回滚。详细的分析死锁产生的原因，可以通过改进程序有效的避免死锁的产生。

5) 如果应用并不介意死锁的出现，那么可以在应用中对发现的死锁进行处理。

6) 确定更合理的事务大小，小事务更少地倾向于冲突。

7) 如果你正使用锁定读，（SELECT ... FOR UPDATE或 ... LOCK IN SHARE MODE），试着用更低的隔离级别，比如READ COMMITTED。

8) 以固定的顺序访问你的表和行。则事务形成良好定义的查询并且没有死锁。


## 配置优化
#### 存储引擎
不同存储引擎性能对比。

#### 线程数

## 硬件层面
1. CPU
    什么地方用到CPU：查询、比较、排序等

    配置线程数：`show variables like '%_io_threads'`

    不同CPU对吞吐量的影响：vultr 1cpus = 4000 2cpus = 10098 4cpus = 16800 6cpus = 19416 8cpus = 20164 

    https://blog.csdn.net/sinat_36246371/article/details/53425183
1. 内存
    什么地方用到内存：索引、缓存

    不同内存对吞吐量的影响：vultr 500MB = 没能启动 1GB/2GB = 4000 4GB = 10098 8GB = 16800 16GB = 19416 32GB = 20164 

1. 硬盘
    机械硬盘、固态硬盘、内存硬盘

1. ping值
    将ping值控制在1ms以内。

    使用 IP 而不是域名做数据库路径，避免 DNS 解析。

    不同ping对TPS的影响：每20ms的延迟，TPS降1倍。

#### 读写分离
一台数据库支持最大连接数是有限的，如果用户的并发访问很多，一台服务器无法满足需求，可以集群处理。MySQL 集群处理技术最常用的就是读写分离。

## 外部优化
1. 代码、业务优化，减少数据查询
1. 缓存如 Redis 等，将热点数据缓存起来
1. 搜索引擎如 solr，elasticsearch

## 附一：SQL 注入
SQL 注入产生的原因：程序开发过程中不注意规范书写 SQL 语句和对特殊字符进行过滤，导致客户端可以通过全局变量 POST 和 GET 提交一些 sql 语句正常执行。

防止 SQL 注入的方式：
1. 开启配置文件中的 `magic_quotes_gpc` 和 `magic_quotes_runtime` 设置
1. 执行 SQL 语句时使用 addslashes 进行 SQL 语句转换
1. SQL 语句书写尽量不要省略双引号和单引号
1. 过滤掉 SQL 语句中的一些关键词：`update`、`insert`、`delete`、`select`、 `*` 
1. 提高数据库表和字段的命名技巧，对一些重要的字段根据程序的特点命名，取不被猜到的

## 附二：不同数据库量级解决方案
https://blog.souche.com/mysql_optimize/

#### 百万级数据量
这个数据量基本上大家都经历过，也能感知一些性能问题显露出来了，这个阶段的优化几乎是最重要的，因为到后期千万级，甚至亿级别的阶段，数据库几乎无法动弹，可调整性很低。
1. 单数据库
1. 索引优化
1. 字段优化
1. 查询优化

#### 千万级数据量
到了这个阶段的数据量，数据本身已经有很大的价值了，数据除了满足常规业务需求外，还会有一些数据分析的需求。而这个时候数据可变动性不高，基本上不会考虑修改原有结构，一般会考虑从分区，分表，分库三方面做优化：
1. 分区
    分区是根据一定的规则，数据库把一个表分解成多个更小的、更容易管理的部分，是一种水平划分。对应用来说是完全透明的，不影响应用的业务逻辑，即不用修改代码。因此能存更多的数据，查询，删除也支持按分区来操作，从而达到优化的目的。

1. 分表
    分表分水平分表和垂直分表。

    水平分表即拆分成数据结构相同的各个小表，如拆分成 table1, table2...，从而缓解数据库读写压力。

    垂直分表即将一些字段分出去形成一个新表，各个表数据结构不相同，可以优化高并发下锁表的情况。

    可想而知，分表的话，程序的逻辑是需要做修改的，所以，一般是在项目初期时，预见到大数据量的情况，才会考虑分表。后期阶段不建议分表，成本很大。

1. 分库
    分库一般是主从模式，一个数据库服务器主节点复制到一个或多个从节点多个数据库，主库负责写操作，从库负责读操作，从而达到主从分离，高可用，数据备份等优化目的。

    当然，主从模式也会有一些缺陷，主从同步延迟，binlog 文件太大导致的问题等等，这里不细讲（笔者也学不动了）。

#### 亿级数据量
没碰到过。。。。
