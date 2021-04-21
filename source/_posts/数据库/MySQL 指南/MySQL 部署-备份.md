---
title: MySQL 部署-备份

categories:
- MySQL 指南

date: 2020-04-28 00:00:02
---


#### 测试

## 全量备份与增量备份
### 全量备份
全量数据就是数据库中所有的数据，全量备份就是把数据库中所有的数据进行备份。
```bash
# 备份一个库
mysqldump 
mysqldump -uroot -poldbody -S /data/33-6/mysql.sock -F -B oldbody|gzip > /server/backup/mysqlbak_$(data +%F).sql.gz

# 备份所有库
mysqldump -uroot -poldbody -S /data/33-6/mysql.sock -F -B -A|gzip > /server/backup/mysqlbak_$(data +%F).sql.gz
```

### 增量备份
增量数据就是从上次全量备份之后，更新的新数据。对于MySQL来说，binlog日志就是MySQL的增量数据。

#### 按天备份情况
优点：
1. 恢复时间短
1. 维护成本低

缺点：
1. 占用空间多
1. 占用资源多，经常锁表操作

|周一全量备份|周一增量数据|周一全量备份|周二增量数据|...|
|:--|
|000000001.sql.gz|mysql-bin.000036|000000002.sql.gz|mysql-bin.000056|
||mysql-bin.000037||mysql-bin.000057|
||mysql-bin.000038||mysql-bin.000058|
||mysql-bin.000039||mysql-bin.000059|




#### 按周备份情况
优点：
1. 占用空间小
1. 占用资源少，无需经常锁表，用户体验会好点

缺点：
1. 维护成本大，恢复麻烦

|周六全量备份|周一增量备份|周二增量备份|周三增量备份|...|
|:--|
|000000001.sql.gz|mysql-bin.000037|mysql-bin.000037|mysql-bin.000037||

#### 全量和增量的频率是怎么做的呢？
1. 中小公司，全量一般是每天一次，业务流量低谷执行全备，执行前要锁表
1. 单台数据库，如何增量。用rsync（配合定时任务频率大点，或者inotify，主从复制）把所有binlog备份到远程服务器，尽量做主从复制
1. 大公司周备，每周六00点一次全量，下周日-下周六00点前都是增量
1. 一主多从，会有一个从库做备份，延迟同步

#### mysql的mysqldump备份什么时候能派上用场？ 
1. 迁移或者升级数据库时
1. 增加从库的时候
1. 因为硬件或特殊异常情况，主库或者从库挂机，主从可以相互切换，无需备份
1. 人为的DDL，DML语句，主从库没办法了，所有库都会执行。此时需要备份
1. 跨机房灾备，需要备份拷贝走

增量备份例子：
```bash
rsync -avz /data/3306/mysql-bin.000* rsync_backup@10.0.0.18::backup --password-file=/etc/rsync.password
```

#### 什么情况下需要增量恢复？
我们在生产工作中一般常用一主多从的数据库架构，常见的备份方案是在某一不对外服务的从库上开启binlog，然后实施定时全备份和实时增量备份

#### 什么是增量恢复
利用二进制日志和全备进行的恢复国产，被称为增量恢复

#### 主或从库宕机（硬件损坏）是否需要增量恢复？
不要增量恢复，主库宕机，只需要把其中一个同步最快的从库（master.info，或5.5半同步机制）切换为主库即可。从库宕机，直接不用就好了（一般会配LVS负载均衡）， 或者正常修复。

#### 人为操作数据库SQL破坏主库是否需要增量恢复？
在数据库主库内部命令行误操作，会导致所有的数据库（包括从库）数据丢失，例如：在主库里执行了drop database test;这样的删除语句，这是所有的从库也会执行这个drop database test;语句，从而导致所有的数据库的test库丢死后。这样的场景是需要增量恢复的。

#### 只有一个主库是否需要增量恢复？
如果公司里只有一个主库的情况，首先应该做定时全量备份（每天一次）及增量备份（每个1-10分钟对binlog日志做切割然后备份到其它的服务器上，或者本地其它的硬盘里）或者些到网络文件系统（备份服务器）里。

如果不允许数据丢失，最好的办法就是做从库，通过drbd（基于磁盘块的）同步。

正常情况：
1. 主从同步：除了分担读写分离压力外，还可以防止物理设置损坏数据丢失的恢复
1. 从库备份：在从库进行全量和增量方式的备份，可以防止人为对主库的误操作导致数据丢失，确保备份的从库实时和主库是同步状态的

> 小结
> 一般由人为（或程序）逻辑的方式在数据库执行的SQL语句等误操作，需要增量恢复，因为此时，所有的从库也执行了误操作语句。

## MySQL增量恢复必备条件
### 开启log-bin日志功能
MySQL数据库开启log-bin参数记录binlog日志功能如下：
```bash
grep log-bin /data/3306/my.cnf
log-bin=/data/3306/mysql-bin
```

提示：主库和备份的从库都要开启binlog记录功能

> 小结
> 存在一份全备份加上全备之后的时刻到问题时刻的所有增量binlog文件备份



## 使用cp进行备份
#### 向数据表施加读锁（只能读不能写）
```bash
mysql> flush tables with read lock;
```

#### 备份数据文件
```bash
zhangqinghua$ mkdir /backup                     # 创建文件夹存放备份数据库文件
zhangqinghua$ cp -a /var/lib/mysql/* /backup    # 保留权限的拷贝源数据文件
```

#### 释放锁
```bash
mysql> unlock tables;
```

#### 模拟数据丢失并恢复
```bash
# 随便删除一条数据
zhangqinghua$ systemctl stop mysqld             # 先停止MySQL服务
zhangqinghua$ rm -rf /var/lib/mysql/*           # 清空数据，这一步可以不做
zhangqinghua$ cp -a /backup/* /var/lib/mysql/   # 将备份的数据文件拷贝回去
zhangqinghua$ systemctl start mysqld            # 重启MySQL服务
# 可以看到，数据又回来了
```

## 使用mysqldump备份数据
mysqldump是MySQL自带的逻辑备份工具。它的原理是，通过协议连接到MySQL数据库，将需要备份的数据查询出来转化成对应的insert语句，当我们需要还原这些数据时，只要执行这些insert语句即可。

```bash
zhangqinghua$ mysql -uroot -p -e 'show master status'
Enter password: 
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000002 |      584 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
```


mysqldump是MySQL自带的逻辑备份工具。它的原理是，通过协议连接到MySQL数据库，将需要备份的数据查询出来转化成对应的insert语句，当我们需要还原这些数据时，只要执行这些insert语句即可。

mysqldump是一个客户端工具，所有当它连接数据库时，也会读取MySQL数据库的配置文件，加载跟客户端相关的配置。

#### 尝试

## 常用命令
#### 将数据库aonitask备份信息输出到屏幕上（不带数据库）
```bash
[root@vultrguest mysql]# mysqldump -uroot aonitask -p
Enter password: 
-- MySQL dump 10.13  Distrib 8.0.17, for Linux (x86_64)
--
-- Host: localhost    Database: aonitask
-- ------------------------------------------------------
-- Server version	8.0.17

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!50503 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `po_banner`
--

DROP TABLE IF EXISTS `po_banner`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `po_banner` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `creator` varchar(64) NOT NULL DEFAULT '[SYS]' COMMENT '创建者id',
  `gmt_created` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `is_deleted` char(1) NOT NULL DEFAULT 'n' COMMENT '是否已删除 y:已删除 n:未删除',
  `modifier` varchar(64) NOT NULL DEFAULT '[SYS]' COMMENT '修改者id',
  `image_key` varchar(64) NOT NULL COMMENT '图片Key',
  `image_link` varchar(512) NOT NULL COMMENT '链接地址',
  `release_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '发布时间',
  `status` char(1) DEFAULT 'n' COMMENT '状态 n-下线 y-上线',
  `title` varchar(64) NOT NULL COMMENT '标题',
  PRIMARY KEY (`id`),
  KEY `isDeleted` (`is_deleted`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='广告表';
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `po_banner`
--

LOCK TABLES `po_banner` WRITE;
/*!40000 ALTER TABLE `po_banner` DISABLE KEYS */;
INSERT INTO `po_banner` VALUES (1,'[SYS]','2020-03-08 18:39:10','2020-03-08 18:49:11','n','[SYS]','1','1','2020-03-08 18:39:10','n','123');
/*!40000 ALTER TABLE `po_banner` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2020-03-08 19:15:37
```

#### 将数据库aonitask备份信息写入aonitask.sql脚本（不带数据库）
```bash
[root@vultrguest /]# mysqldump -uroot aonitask > /backup/aonitask.sql -p
Enter password: 
[root@vultrguest /]# ll /backup
total 4
-rw-r--r--. 1 root root 2827 Mar  8 19:21 aonitask.sql
```

#### 将数据库aonitask备份信息写入aonitask.sql脚本（带数据库）
使用--databases选项指定数据库时，即可在备份时生成创建数据库的语句。
```bash
[root@vultrguest /]# mysqldump -uroot --databases aonitask > /backup/aonitask.sql -p
Enter password: 
[root@vultrguest /]# cat /backup/aonitask.sql 
-- MySQL dump 10.13  Distrib 8.0.17, for Linux (x86_64)
...

--
-- Current Database: `aonitask`
--

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `aonitask` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;

USE `aonitask`;

--
-- Table structure for table `po_banner`
--
...
```

#### 备份aonitask中的所有表，但不生成创建aonitask的语句
```bash
[root@vultrguest /]# mysqldump -uroot aonitask -p
Enter password: 
```

#### 备份aonitask中的所有表，同时生成创建aonitask的语句
```bash
[root@vultrguest /]# mysqldump -uroot --databases aonitask -p
Enter password: 
```

#### 备份aonitask中的t1、t2表，同时生成创建aonitask的语句
```bash
[root@vultrguest /]# mysqldump -uroot --databases aonitask t1 t2 -p
Enter password: 
```

#### 备份aonitask1，aonitask2中的所有表，同时生成创建aonitask1、aonitask2的语句
```bash
[root@vultrguest /]# mysqldump -uroot --databases aonitask1 aonitask2 -p
Enter password: 
```

#### 只备份表结构，不备份数据
```bash
[root@vultrguest /]# mysqldump -uroot --databases -d aonitask -p
Enter password: 
```



#### 备份所有的数据库
```bash
[root@vultrguest /]# mysqldump -uroot --all-databases -p
Enter password: 
```

#### 导出所有库
```bash
mysqldump -uusername -ppassword --all-databases > all.sql
```

#### 导入所有库
```bash
mysql>source all.sql;
```

#### 导出某些库
```bash
mysqldump -uusername -ppassword --databases db1 db2 > db1db2.sql
```

#### 导入某些库
```bash
mysql>source db1db2.sql;
```

#### 导入某个库
```bash
mysql>ource db1.sql;
```

#### 导出某些数据表
```bash
mysqldump -uusername -ppassword db1 table1 table2 > tb1tb2.sql
```

#### 导入某些数据表
```bash
mysql -uusername -ppassword db1 < tb1tb2.sql
```

或者MySQL命令行：
```sql
user db1;
source tb1tb2.sql;
```

#### mysqldump字符集设置
```bash
mysqldump -uusername -ppassword --default-character-set=gb2312 db1 table1 > tb1.sql
```

## 常用选项
#### --master-data
--master-data可以记录备份日志的还原点，有3个选值：
1. 0: 表示在备份时，不记录对应二进制日志文件的位置，和不使用此选项一样
1. 1: 表示在备份时，在备份文件中生成对应的`CHANGE MASTER TO`语句，标明此次备份开始时二进制日志的前缀名以及其所处的position（位置）
1. 3: 跟选项2一样，只不过此条语句是注释状态的

```bash
[root@vultrguest ~]# mysqldump -uroot --master-data=2 aonitask -p
Enter password: 
-- MySQL dump 10.13  Distrib 8.0.17, for Linux (x86_64)
...

--
-- Position to start replication or point-in-time recovery from
--

-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.000002', MASTER_LOG_POS=2946;

--
-- Table structure for table `po_banner`
--
...
```

#### --fulsh-logs
即生成新日志，例如当前二进制日志是binlog.000002，调用`mysqladmin flush-logs`后，即生成binlog.000003，后续的操作会写入binlog.000003中。所以我们可以每小时调用此命令以达到增量备份的效果。

```bash
# 生成备份，刷新二进制日志，并清空旧日志
[root@vultrguest mysql]# mysqldump -uroot -p --flush-logs --delete-master-logs --all-databases
```


#### 其它常用选项
在数据库中，还存在一些存储过程和存储函数，存在一些触发器、事件表，这些东西也需要备份以免最终的备份“不全”：
- --events选项：表示备份时，事件表会被备份
- --routines选项：表示备份时，存储过程和存储函数也会被备份
- --triggers选项：表示备份时，触发器会被备份

## 自动化脚本
#### 备份脚本
文件放置于/usr/local/script/backup_database.sh，确保mysqldump命令可用。

```bash
#!/bin/bash  
#Shell Command For Backup MySQL Database Everyday Automatically By Crontab  
#time 2015-5-20 
    #name huxianglin 
USER=root 
PASSWORD=1 
DATABASE1=aonitask 
BACKUP_DIR=/data/backup/database/                       # 备份数据库文件的路径 
LOGFILE=/data/backup/database/data_backup.log           # 备份数据库脚本的日志文件 
DATE=`date +%Y%m%d-%H%M -d -3minute`                    # 获取当前系统时间-3分钟 
DUMPFILE1=$DATE-zblog.sql                               # 需要备份的数据库名称 
ARCHIVE1=$DUMPFILE1-tar.gz                              # 备份的数据库压缩后的名称 

if [ ! -d $BACKUP_DIR ];                                # 判断备份路径是否存在，若不存在则创建该路径 
then  
mkdir -p "$BACKUP_DIR" 
fi  

echo -e "\n" >> $LOGFILE   
echo "------------------------------------" >> $LOGFILE  
echo "BACKUP DATE:$DATE">> $LOGFILE  
echo "------------------------------------" >> $LOGFILE  

cd $BACKUP_DIR                                          #跳到备份路径下 
mysqldump -u$USER -p$PASSWORD $DATABASE1 > $DUMPFILE1   #使用mysqldump备份数据库 
if [[ $? == 0 ]]; then 
tar czvf $ARCHIVE1 $DUMPFILE1 >> $LOGFILE 2>&1          #判断是否备份成功，若备份成功，则压缩备份数据库，否则将错误日志写入日志文件中去。 
echo "$ARCHIVE1 BACKUP SUCCESSFUL!" >> $LOGFILE  
rm -f $DUMPFILE1 
else  
echo “$ARCHIVE1 Backup Fail!” >> $LOGFILE  
fi 
```

#### 删除过时备份脚本
/usr/local/script/clean_database.sh

```bash
#!/bin/bash 
#time 2015-05-21 
#name huxianglin 

BACKUPDIR="/data/backup/database/"                                     #定义备份文件路径 
KEEPTIME=7                                                             #定义需要删除的文件距离当前的天数 
DELFILE=`find $BACKUPDIR -type f -mtime +$KEEPTIME -exec ls {} \;`     #找到天数大于7天的文件 
for delfile in ${DELFILE}                                              #循环删除满足天数大于七天的文件 
do 
rm -f $delfile 
done 
```

#### 定时执行脚本
/usr/local/script/crontab.sh

```bash
SHELL=/bin/bash 
PATH=/sbin:/bin:/usr/sbin:/usr/bin 
MAILTO=root 
# For details see man 4 crontabs 
# Example of job definition: 
# .---------------- minute (0 - 59) 
# |  .------------- hour (0 - 23) 
# |  |  .---------- day of month (1 - 31) 
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ... 
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat 
# |  |  |  |  | 
# *  *  *  *  * user-name  command to be executed 
  01 00 * * * root /usr/local/script/backup_database.sh              #定义每天凌晨0点01分执行备份数据库脚本 
  02 00 * * 0 root /usr/local/script/clean_database.sh               #定义每周日凌晨0点02分执行删除数据库备份文件 
```

## 实战
在对数据库进行一次全量备份和N次增量备份后，删库删表，然后尝试恢复。

#### 原始数据
```bash
mysql> select id, image_key, image_link, title from po_banner;
+----+-----------+------------+-------+
| id | image_key | image_link | title |
+----+-----------+------------+-------+
|  1 | 1         | 1          | 123   |
|  2 | 111       | 222        | 333   |
+----+-----------+------------+-------+
2 rows in set (0.00 sec)
```
#### 全量备份
```bash
[root@vultrguest mysql]# mysqldump -uroot -p --single-transaction --master-data=2 --routines --flush-logs --all-databases > /backup/alldb.sql;
```

#### 增量备份
```bash
# change id = 1, image_key = 1_change > binlog.000002
[root@vultrguest mysql]# mysqladmin -uroot -p1 flush-logs

# change id = 1, image_link = 1_change > binlog.000003
[root@vultrguest mysql]# mysqladmin -uroot -p1 flush-logs

# change id = 1, title = 1_change > binlog.000004
[root@vultrguest mysql]# mysqladmin -uroot -p1 flush-logs

mysql> select id, image_key, image_link, title from po_banner;
+----+-----------+------------+----------+
| id | image_key | image_link | title    |
+----+-----------+------------+----------+
|  1 | 1_change  | 1_change   | 1_change |
|  2 | 111       | 222        | 333      |
+----+-----------+------------+----------+
2 rows in set (0.00 sec)
```

#### 模拟误删数据
```bash
# 将日志切换到binlog.000005
[root@vultrguest mysql]# mysqladmin -uroot -p1 flush-logs

# 模拟误删数据...

# 查看数据
mysql> select id, image_key, image_link, title from po_banner;
+----+-----------+------------+-------+
| id | image_key | image_link | title |
+----+-----------+------------+-------+
|  2 | 111       | 222        | 333   |
+----+-----------+------------+-------+
1 row in set (0.00 sec)
```

#### 恢复全量数据
恢复数据前先关闭数据写入。

```bash
[root@vultrguest mysql]# mysql -uroot -p1 -e 'source /backup/alldb.sql;'
mysql: [Warning] Using a password on the command line interface can be insecure.

mysql> select id, image_key, image_link, title from po_banner;
+----+-----------+------------+-------+
| id | image_key | image_link | title |
+----+-----------+------------+-------+
|  1 | 1         | 1          | 123   |
|  2 | 111       | 222        | 333   |
+----+-----------+------------+-------+
2 rows in set (0.00 sec)
```

#### 恢复增量数据
```bash
[root@vultrguest mysql]# mysqlbinlog binlog.000002 binlog.000003 binlog.000004 | mysql -uroot -p1
mysql: [Warning] Using a password on the command line interface can be insecure.

mysql> select id, image_key, image_link, title from po_banner;
+----+-----------+------------+----------+
| id | image_key | image_link | title    |
+----+-----------+------------+----------+
|  1 | 1_change  | 1_change   | 1_change |
|  2 | 111       | 222        | 333      |
+----+-----------+------------+----------+
2 rows in set (0.00 sec)
```

## 对比
|备份方法|备份速度|恢复速度|便捷性|功能|一般用于|
|:--|
|cp|快|快|一般、灵活性低|很弱|少量数据|
|mysqldump|慢|慢|一般、可无视存储引擎的差异|一般|中小型数据量的备份|
|lvm2快照|快|快|一般、支持几乎热备、速度快|一般|中小型数据量的备份|
|xtrabackup|较快|较快|实现innodb热备、对存储引擎有要求|强大|较大规模的备份|