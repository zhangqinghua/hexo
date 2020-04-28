---
title: Chapter 6.2 如何进行基准测试

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:62
---
基准测试有两种主要的策略：一是针对整个系统的整体测试，另外是单独测试（如单独对 MySQL 进行基准测试）。这两种策略被称为集成式（full-stack）和单组件试（single-component）基准测试。

## 针对整个系统进行基准测试
对整个系统进行基准测试，即从系统入口进行测试，如网站 Web 前端，手机 APP 前端等。它能够测试整个系统的性能，包括 WEB 服务器缓存、数据库等，能反映出系统中各个组件接口间的性能问题，体现真实性能情况。缺点就是测试设计复杂，消耗时间长。

## 单独对 MySQL 进行基准测试
单独对 MySQL 进行基准测试和整体测试相反，测试设计简单，所需耗费时间短，但是无法全面了解整个系统的性能基线。

## 基准测试的常见指标
开始基准测试甚至是设计基准测试之前，需要先明确测试的目标。测试决定选择什么样的测试工具和技术，以获取精确而有意义的测试结果。可以将测试目标细化为一些列问题，比如“增加缓存是否能够带来性能提升？“。

有时候需要用不同的方法测试不同的指标，比如针对延迟（latency）和吞吐量（throughput）就需要采用不同的测试方法。

基准测试通常有以下这几种测试指标：
1. TPS：单位时间内所处理的事务数
1. QPS：单位时间内所处理的查询数
1. 并发量：同时处理的查询请求数
1. 响应时间：平均响应时间、最小响应时间、最大响应时间、各时间所占百分比

## 基准测试的步骤
1. 设计和规划基准测试
    在开始实施基准测试之前要规划基准测试。选择合适的测试方案。设计专用的基准测试是很复杂的，往往需要一个迭代的过程。首先需要获取生产数据集的快照，并且该快照很容易还原，以便进行后续的测试。这里数据选择上也需要注意，要选择一个有代表性的时间段，最好选取的时间段数据可以覆盖整个系统的活动状态。
1. 基准测试应该运行多长时间
    基准测试应该运行足够长的时间，这里足够长的时间指的是，让测试一直运行到确认系统已经稳定为止。详细解释参照《MySQL高性能》。
1. 准备基准测试及数据收集脚本
    一般需要收集CPU使用率、IO、网络流量、状态与计数器信息等。
    ```bash
    # 下面是一个简单的收集MySQL测试数据的shell脚本
    INTERVAL=5  #运行间隔，每隔多少时间收集一下状态信息
    PREFIX=/home/imooc/benchmarks/$INTERVAL-sec-status#定义了状态信息的存储位置
    RUNFILE=/home/imooc/benchmarks/running#指定了运行标识，如果存在标识，证明脚本在运行，想停止脚本，就删除标识文件的方式来停止脚本
    echo "1">$RUNFILE #标识文件
    MYSQL=/usr/local/mysql/bin/mysql #mysql命令所在的位置
    $MYSQL -e "show global variables" >>mysql-variables#记录了进行测试的当前mysql的一些设置信息
    while test -e $RUNFILE; #循环体开始
    do
        file=$(date +%F_%I) #定义了脚本运行时间
        sleep=$(date +%s.$N | awk '{print 5 - ($1 % 5)}') #每隔多久运行一次脚本
        sleep $sleep
        ts="$(date +"TS %s.$N $F %T")" 
        loadavg="$(uptime)" #系统的负载情况
        echo "$ts $loadavg" >> $PREFIX-${file}-status #记录到文件中
        $MYSQL -e "show global status" >> $PREFIX-${file}-status & #mysql的全局的状态信息
        echo "$ts $loadavg" >> $PREFIX-${file}-innodbstatus  #记录在文件里
        $MYSQL -e "show engine innodb status" >> $PREFIX-${file}-innodbstatus &        #收集innodb的状态信息
        echo "$ts $loadavg" >> $PREFIX-${file}-processlist
        $MYSQL -e "show full processlist\G" >> $PREFIX-${file}-processlist &        #收集mysql线程的情况
        echo $ts
    done    
    echo Exiting because $RUNFILE does not exists
    ```
    在执行基准测试时，要尽可能多的收集被测试系统的信息。这里可以通过一些自动化脚本进行收集系统状态的性能指标，如CPU使用率、磁盘I/O、网络流量统计、SHOW GLOBAL STATUS计数器等。
1. 运行记基准测试
1. 保存及分析基准测试结果
    最好以图形的方法来展示测试结果。
    ```bash
    # 下面是保存分析结果的脚本示例
    #!/bin/bash
    awk '
        BEGIN {
            printf "#ts date time load QPS";
            fmt=" %.2f";
        }
        /^TS/ {
            ts=substr($2,1,index($2,".")-1);
            load=NF-2;
            diff=ts-prev_ts;
            printf "\n%s %s %s %s %s",ts,$3,$4,substr($load,1,length($load)-1);
            prev_ts=ts;
        }
        /Queries/{
            print fmt,($2-Queries)/diff;
            Queries=$2
        }
        ' "$@"
    ```

## 基准测试中容易忽略的问题
1. 使用生产环境数据时只使用了部分数据
    如果要使用产环境数据测试推荐使用全部生产环境数据，因为人为的选择一部分数据测试则一数据量不足，二不能很好的反映数据发布，测试的结果也是不准确的。
1. 在多用户场景中，只做单用户的测试
    在Web应用中通常存在多用户高并发的场景，因此基准测试一定要考虑到多线程的场景，推荐使用多线程并发测试。因为MySQL在多线程的性能表现可能和单线程完成不一样。
1. 反复执行同一查询
    同一查询可能会命中缓存，不能真实反映查询性能。