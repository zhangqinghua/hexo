---
title: MySQL 基准测试

categories:
- MySQL 手册

date: 2020-07-07 00:00:01
---
基准测试是指通过设计科学的测试方法、测试工具和测试系统，实现对一类测试对象的某项性能指标进行定量的和可对比的测试。

性能指标
1. RT：响应时间。包括平均响应时间、最小响应时间、最大响应时间、每个响应时间的查询占比。比较需要重点关注的是，前 95-99% 的最大响应时间。因为它决定了大多数情况下的短板。
1. TPS：Transactions Per Second ，即数据库每秒执行的事务数，以 commit 成功次数为准。
1. QPS：Queries Per Second ，即数据库每秒执行的 SQL 数（含 insert、select、update、delete 等）。
1. Concurrency Threads：并发量，每秒可处理的查询请求的数量。

总结来说，实际就是 2 个维度：
1. 延迟
1. 吞吐量

## sysbench 介绍
sysbench 是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。

它主要包括以下几种方式的测试：
1. CPU 性能
1. 磁盘 IO 性能
1. 调度程序性能
1. 内存分配及传输速度
1. POSIX 线程性能
1. 数据库性能(OLTP 基准测试)

目前 sysbench 主要支持 MySQL、PgSQL、Oracle 这 3 种数据库。

sysbench 也是目前 DBA 最喜欢用来做 MySQL 性能的测试工具。

## 安装
centos 安装，需要到 /usr/share/sysbench/ 目录使用：

```bash
sudo yum install -y sysbench
```

mac 安装，需要到 /usr/local/Cellar/sysbench/1.0.17_1/share/sysbench 目录使用

```bash
sudo brew install sysbench
```

## 准备数据
执行以下命令，sysbench 会自动在数据库常见相应的测试数据。`oltp_common.lua` 是要执行的测试脚本。这里使用自带的 lua 测试脚本。
```bash
sysbench oltp_common.lua \
    --mysql-host=216.155.135.22 \
    --mysql-port=3306 \
    --mysql-user=sptest \
    --mysql-password=sptest \
    --mysql-db=sptest \
    --table-size=100000 \
    --tables=9 \
    prepare
```

MySQL 相关参数：
1. mysql-host       
    MySQL server IP  。
1. mysql-port       
    MySQL server port 。
1. mysql-db         
    MySQL Server 数据库名。
1. mysql-user       
    MySQL server 账号。
1. mysql-password   
    MySQL server 密码。


sysbench 相关参数：
1. time
  最大的总执行时间，以秒为单位，默认为 10 秒。
1. tables
    表数。
1. table-size
    表记录条数。
1. threads
    要使用的线程数，默认 1 个。
1. report-interval
    以秒为单位定期报告具有指定间隔的中间统计信息，默认为 0 ，表示禁用中间报告。
1. prepare
    执行准备数据

## 执行测试
执行下面命令开始对数据库进行测试。`oltp_read_write.lua` 表示混合读写。在一个事务中，默认比例是：select:update_key:update_non_key:delete:insert = 14:1:1:1:1 。这也是为什么，我们测试出来的 TPS 和 QPS 的比例，大概在 1:18~20 左右。相当于说，一个事务中，有 18 个读写操作。`run` 表示执行测试命令。

```bash
sysbench oltp_read_write.lua \
	--mysql-host=216.155.135.22 \
	--mysql-port=3306 \
	--mysql-user=sptest \
	--mysql-password=sptest \
	--mysql-db=sptest  \
	--threads=32 \
	--time=60 \
	--report-interval=10  \
	run
```

执行结果说明：

```bash
sysbench 1.0.14 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 10
Report intermediate results every 10 second(s)
Initializing random number generator from current time


Initializing worker threads...

--线程启动
Threads started!

-- 每10秒钟报告一次测试结果，tps、每秒读、每秒写、95%以上的响应时长统计
[ 10s ] thds: 10 tps: 956.45 qps: 19139.38 (r/w/o: 13399.56/3825.92/1913.91) lat (ms,95%): 29.72 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 10 tps: 989.71 qps: 19804.24 (r/w/o: 13862.17/3962.55/1979.52) lat (ms,95%): 28.67 err/s: 0.10 reconn/s: 0.00
[ 30s ] thds: 10 tps: 995.44 qps: 19909.03 (r/w/o: 13936.11/3982.05/1990.87) lat (ms,95%): 27.66 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 10 tps: 983.30 qps: 19660.90 (r/w/o: 13764.23/3930.08/1966.59) lat (ms,95%): 27.66 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 10 tps: 994.20 qps: 19882.58 (r/w/o: 13918.26/3975.92/1988.41) lat (ms,95%): 29.19 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            1663326 --读总数
        write:                           475233 -- 写总数  
        other:                           237617 -- 其他操作总数(SELECT、INSERT、UPDATE、DELETE之外的操作，例如COMMIT等)  
        total:                           2376176 -- 全部总数
    transactions:                        118808 (989.85 per sec.) -- 总事务数(每秒事务数)
    queries:                             2376176 (19797.13 per sec.)
    ignored errors:                      1      (0.01 per sec.) --总忽略错误总数(每秒忽略错误次数)
    reconnects:                          0      (0.00 per sec.) --重连总数(每秒重连次数)

General statistics: --常规统计
    total time:                          120.0244s --总耗时
    total number of events:              118808 --共发生多少事务数

Latency (ms):
         min:                                    6.08 --最小耗时
         avg:                                   10.10 --平均耗时
         max:                                   87.65 --最长耗时
         95th percentile:                       28.16 --超过95%平均耗时
         sum:                              1199522.76

Threads fairness: --并发统计
    events (avg/stddev):           11880.8000/273.15 --总处理事件数/标准偏差
    execution time (avg/stddev):   119.9523/0.00 --总执行时间/标准偏差
```

## 测试结果
下面是针对 MySQL 进行不同维度的测试结果。

#### 请求延迟
测试延迟对 MySQL 并发的影响，结论：每 20 ms 的延迟都使得 MySQL 并发降低一半。

MySQL 数据：
1. 机器配置：1核 1024MB
1. CPU 使用率：0%
1. Mem 使用率：50%/360MB
1. 测试线程数：32

韩国测试机器：180ms ping、CPU 10% 使用率、Mem 65%/500MB 使用率。
```bash
SQL statistics:
    queries performed:
        read:                            34678
        write:                           9883
        other:                           4945
        total:                           49506
    transactions:                        2468   (8.11 per sec.)
    queries:                             49506  (162.73 per sec.)
    ignored errors:                      9      (0.03 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          304.2212s
    total number of events:              2468

Latency (ms):
         min:                                 3646.35
         avg:                                 3917.42
         max:                                 7517.01
         95th percentile:                     4437.27
         sum:                              9668200.40

Threads fairness:
    events (avg/stddev):           77.1250/1.60
    execution time (avg/stddev):   302.1313/1.22

tps: 6.30 qps: 156.27 (r/w/o: 115.08/25.29/15.90) lat (ms,95%): 5409.26 err/s: 0.10 reconn/s: 0.00
```

纽约测试机器：1ms ping、CPU 85% 使用率、Mem 65%/500MB 使用率。

```bash
SQL statistics:
    queries performed:
        read:                            891128
        write:                           254245
        other:                           127161
        total:                           1272534
    transactions:                        63509  (211.55 per sec.)
    queries:                             1272534 (4238.85 per sec.)
    ignored errors:                      143    (0.48 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          300.2053s
    total number of events:              63509

Latency (ms):
         min:                                   22.98
         avg:                                  151.22
         max:                                  647.85
         95th percentile:                      186.54
         sum:                              9603770.62

Threads fairness:
    events (avg/stddev):           1984.6562/12.64
    execution time (avg/stddev):   300.1178/0.05

tps: 197.49 qps: 3996.19 (r/w/o: 2804.28/793.04/398.87) lat (ms,95%): 223.34 err/s: 0.70 reconn/s: 0.00
```

#### 线程数量
测试不同的线程数对 MySQL 并发的影响。结论：线程多的话容易死机。

MySQL 数据：
1. 延迟：1ms ping
1. 机器配置：1核 1024MB
1. CPU 使用率：0%
1. Mem 使用率：50%/360MB

纽约测试机器：1 threads、CPU 35% 使用率、Mem 51% 使用率。

```bash
SQL statistics:
    queries performed:
        read:                            206374
        write:                           58964
        other:                           29482
        total:                           294820
    transactions:                        14741  (49.13 per sec.)
    queries:                             294820 (982.67 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          300.0153s
    total number of events:              14741

Latency (ms):
         min:                                   12.93
         avg:                                   20.34
         max:                                  142.58
         95th percentile:                       29.19
         sum:                               299784.93

Threads fairness:
    events (avg/stddev):           14741.0000/0.00
    execution time (avg/stddev):   299.7849/0.00
```

纽约测试机器：16 threads、CPU 81% 使用率、Mem 59% 使用率。

```bash
SQL statistics:
    queries performed:
        read:                            204008
        write:                           58243
        other:                           29126
        total:                           291377
    transactions:                        14554  (145.23 per sec.)
    queries:                             291377 (2907.52 per sec.)
    ignored errors:                      18     (0.18 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          100.2105s
    total number of events:              14554

Latency (ms):
         min:                                   22.33
         avg:                                  110.06
         max:                                  359.90
         95th percentile:                      147.61
         sum:                              1601767.80

Threads fairness:
    events (avg/stddev):           909.6250/11.37
    execution time (avg/stddev):   100.1105/0.04

tps: 148.87 qps: 2998.68 (r/w/o: 2103.43/596.00/299.25) lat (ms,95%): 134.90 err/s: 0.00 reconn/s: 0.00
```

纽约测试机器：32 threads、CPU 83% 使用率、Mem 62% 使用率。

```bash
SQL statistics:
    queries performed:
        read:                            216020
        write:                           61610
        other:                           30816
        total:                           308446
    transactions:                        15386  (153.49 per sec.)
    queries:                             308446 (3077.06 per sec.)
    ignored errors:                      44     (0.44 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          100.2361s
    total number of events:              15386

Latency (ms):
         min:                                   28.93
         avg:                                  208.24
         max:                                  692.07
         95th percentile:                      262.64
         sum:                              3204016.01

Threads fairness:
    events (avg/stddev):           480.8125/3.92
    execution time (avg/stddev):   100.1255/0.06

tps: 151.34 qps: 3048.14 (r/w/o: 2138.99/606.07/303.08) lat (ms,95%): 267.41 err/s: 0.30 reconn/s: 0.00
```

纽约测试机器：64 threads、CPU 85% 使用率、Mem 66% 使用率。

```bash
# 测试到一半死机
tps: 160.19 qps: 3345.46 (r/w/o: 2363.85/652.14/329.47) lat (ms,95%): 707.07 err/s: 2.70 reconn/s: 0.00
```

#### 不同版本

#### 记录大小
测试不同的表的数据量对并发的影响。

MySQL 数据：
1. 机器配置：1核 1024MB
1. CPU 使用率：0%
1. Mem 使用率：50%/360MB
1. 测试线程数：32

1w 数据：

```bash
SQL statistics:
    queries performed:
        read:                            914830
        write:                           260914
        other:                           130505
        total:                           1306249
    transactions:                        65160  (217.05 per sec.)
    queries:                             1306249 (4351.21 per sec.)
    ignored errors:                      185    (0.62 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          300.2017s
    total number of events:              65160

Latency (ms):
         min:                                   20.28
         avg:                                  147.38
         max:                                 2627.49
         95th percentile:                      186.54
         sum:                              9603145.85

Threads fairness:
    events (avg/stddev):           2036.2500/14.71
    execution time (avg/stddev):   300.0983/0.05

tps: 207.40 qps: 4156.87 (r/w/o: 2912.25/829.21/415.41) lat (ms,95%): 200.47 err/s: 0.40 reconn/s: 0.00
```

10w 条数据
```bash
SQL statistics:
    queries performed:
        read:                            194726
        write:                           55406
        other:                           27728
        total:                           277860
    transactions:                        13819  (229.45 per sec.)
    queries:                             277860 (4613.49 per sec.)
    ignored errors:                      90     (1.49 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.2256s
    total number of events:              13819

Latency (ms):
         min:                                   14.97
         avg:                                  139.16
         max:                                 2190.64
         95th percentile:                      179.94
         sum:                              1923116.42

Threads fairness:
    events (avg/stddev):           431.8438/3.48
    execution time (avg/stddev):   60.0974/0.04

tps: 235.40 qps: 4748.13 (r/w/o: 3332.62/943.01/472.50) lat (ms,95%): 183.21 err/s: 1.70 reconn/s: 0.00
```

100w 条数据

```bash
SQL statistics:
    queries performed:
        read:                            210938
        write:                           60047
        other:                           30047
        total:                           301032
    transactions:                        14980  (249.26 per sec.)
    queries:                             301032 (5009.10 per sec.)
    ignored errors:                      87     (1.45 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0949s
    total number of events:              14980

Latency (ms):
         min:                                   13.19
         avg:                                  128.30
         max:                                  444.92
         95th percentile:                      164.45
         sum:                              1921953.92

Threads fairness:
    events (avg/stddev):           468.1250/3.76
    execution time (avg/stddev):   60.0611/0.02

tps: 236.21 qps: 4765.82 (r/w/o: 3347.28/944.22/474.31) lat (ms,95%): 170.48 err/s: 1.80 reconn/s: 0.00
```

500w 条数据

```bash
SQL statistics:
    queries performed:
        read:                            187712
        write:                           53519
        other:                           26770
        total:                           268001
    transactions:                        13362  (222.05 per sec.)
    queries:                             268001 (4453.65 per sec.)
    ignored errors:                      46     (0.76 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.1732s
    total number of events:              13362

Latency (ms):
         min:                                   19.09
         avg:                                  143.94
         max:                                  520.59
         95th percentile:                      186.54
         sum:                              1923285.64

Threads fairness:
    events (avg/stddev):           417.5625/5.21
    execution time (avg/stddev):   60.1027/0.04

tps: 229.90 qps: 4597.61 (r/w/o: 3217.74/919.78/460.09) lat (ms,95%): 179.94 err/s: 0.30 reconn/s: 0.00
```

1000w 条数据（只测试了一张表，因为准备数据时机器卡死了）

```bash
SQL statistics:
    queries performed:
        read:                            178374
        write:                           50785
        other:                           25413
        total:                           254572
    transactions:                        12672  (210.55 per sec.)
    queries:                             254572 (4229.71 per sec.)
    ignored errors:                      69     (1.15 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.1844s
    total number of events:              12672

Latency (ms):
         min:                                   17.12
         avg:                                  151.73
         max:                                  617.00
         95th percentile:                      196.89
         sum:                              1922761.62

Threads fairness:
    events (avg/stddev):           396.0000/4.53
    execution time (avg/stddev):   60.0863/0.04

tps: 207.90 qps: 4184.34 (r/w/o: 2935.26/832.49/416.59) lat (ms,95%): 196.89 err/s: 0.80 reconn/s: 0.00
```

5000w 条数据

#### 机器配置

#### 读写分离




