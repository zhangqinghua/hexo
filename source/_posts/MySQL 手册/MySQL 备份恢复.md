---
title: MySQL 备份恢复

categories:
- MySQL 手册

date: 2020-07-07 00:00:04
---
数据库备份的主要目的是灾难恢复，备份还可以测试应用、回滚数据修改、查询历史数据、审计等。

## 备份类型
根据是否需要数据库离线有：
1. 冷备：需要关 MySQL 服务，读写请求均不允许状态下进行。
1. 温备：服务在线，但仅支持读请求，不允许写请求。
1. 热备：备份的同时，业务不受影响。

> 注1：这些类型的备份取决于业务的需求而不是备份工具。
> 注2：MyISAM 不支持热备，InnoDB 支持热备，但是需要专门的工具。

根据要备份的数据集合的范围有：
1. 全量备份：即备份所有数据。
1. 增量备份：自上次全量备份以来更改了的数据，不能单独使用，要借助全量备份，备份的频率取决于数据的更新频率。

根据备份数据或文件有：
1. 物理备份：即直接备份数据文件
1. 逻辑备份：即备份表中的数据或代码

## 备份频率
中小公司，全量一般是每天一次，业务流量低谷执行全备，执行前要锁表。

大公司周备，每周六00点一次全量，下周日-下周六00点前都是增量。

备份脚本：

```bash
# 文件位于：/usr/local/script/backup_database.sh，确保mysqldump命令可用。
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

删除过时备份脚本：

```bash
# 文件位于：/usr/local/script/clean_database.sh
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

定时执行脚本：

```bash
# 文件位于：/usr/local/script/crontab.sh
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

## binlog 详解
binlog 记录了所有的 DDL 和 DML （除了数据查询语句）语句，以事件形式记录，还包含语句所执行的消耗的时间，MySQL 的二进制日志是事务安全型的。

#### 常用操作
查看所有binlog日志列表。

```sql
show master logs;
```

查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值。

```sql
show master status;
```

```
刷新log日志，自此刻开始产生一个新编号的binlog日志文件。

```sql
flush logs;
```

> 注：每当 MySQL 服务重启时，会自动执行此命令，刷新binlog日志；在mysqldump备份数据时加 -F 选项也会刷新binlog日志；

重置(清空)所有binlog日志。

```sql
reset master;
```

查询 binlog 设置：
```sql
show global variables like 'log_bin%';

log_bin	                            ON
log_bin_basename	            /var/lib/mysql/binlog
log_bin_index	                    /var/lib/mysql/binlog.index
log_bin_trust_function_creators	    OFF
log_bin_use_v1_row_events	    OFF
```

#### 开启 binlog
MySQL8.0 默认开始 binlog。

#### 关闭 binlog
MySQL8.0 需要在 my.cnf 配置上：

```
[mysqld]
skip-log-bin
```

然后重启。

## cp 备份
我们可以直接将数据库文件复制到另外一个目录达到备份的目的。

#### 向数据表施加读锁（只能读不能写）
```sql
mysql> flush tables with read lock;
```

#### 备份数据文件
```bash
zhangqinghua$ mkdir /backup                     # 创建文件夹存放备份数据库文件
zhangqinghua$ cp -a /var/lib/mysql/* /backup    # 保留权限的拷贝源数据文件
```

#### 释放锁
```sql
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
## mysqldump 备份
mysqldump 是 MySQL 自带的逻辑备份工具。它的原理是，通过协议连接到 MySQL 数据库，将需要备份的数据查询出来转化成对应的 `insert` 语句，当我们需要还原这些数据时，只要执行这些 `insert` 语句即可。

mysqldump 是一个客户端工具，所有当它连接数据库时，也会读取 MySQL 数据库的配置文件，加载跟客户端相关的配置。

#### 导出数据
```bash
zhangqinghua$ mysqldump -uusername -ppassword --all-databases > all.sql

# 设置字符集
zhangqinghua$ mysqldump -uusername -ppassword --default-character-set=gb2312 db1 table1 > tb1.sql
```

#### 导入数据
```bash
zhangqinghua$ mysql -uusername -ppassword db1 < tb1tb2.sql
```

或者 MySQL 命令执行：
```sql
mysql> user db1;
mysql> source tb1tb2.sql;
```

#### 实战
在对数据库进行一次全量备份和 N 次增量备份后，删库删表，然后尝试恢复。

原始数据：
```sql
mysql> select id, image_key, image_link, title from po_banner;
+----+-----------+------------+-------+
| id | image_key | image_link | title |
+----+-----------+------------+-------+
|  1 | 1         | 1          | 123   |
|  2 | 111       | 222        | 333   |
+----+-----------+------------+-------+
2 rows in set (0.00 sec)
```

全量备份：

```bash
[root@vultrguest mysql]# mysqldump -uroot -p --single-transaction --master-data=2 --routines --flush-logs --all-databases > /backup/alldb.sql;
```

增量备份：

```sql
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

模拟误删数据：

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

恢复全量数据：

```bash
# 恢复数据前先关闭数据写入。
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

恢复增量数据：

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
## 主从复制
主从复制使得数据可以从一个数据库服务器复制到其他服务器上，在复制数据时，一个服务器充当主服务器，其余的服务器充当从服务器。

MySQL 服务器之间的主从同步是基于二进制日志机制，主服务器使用二进制日志来记录数据库的变动情况，从服务器通过读取和执行该日志文件来保持和主服务器的数据一致。

#### 用途
主从复制可以用于以下场景：
1. 备份，避免影响业务
1. 读写分离，提供查询服务
1. 实时灾备，用于故障切换

#### 原理

#### 步骤
1.主库：开启二进制日志和配置一个独立的ID，重启

```conf
[mysqld]
server-id=1
```

2.主库：创建一个用来专门复制主服务器数据的账号，记录二进制文件的位置信息

```sql
mysql> create user 'repl'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> grant replication slave on *.* to 'repl'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000003 |      940 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.01 sec)
```

3.主库：暂停写入，备份数据

```bash
# 主库加上一把锁，阻止对数据库进行任何的写操作
mysql> flush tables with read lock;
Query OK, 0 rows affected (0.00 sec)

# 主库备份数据
[root@vultrguest ~]# docker exec mysql01 mysqldump -uroot -p123456 --all-databases > /data/all.sql;
mysqldump: [Warning] Using a password on the command line interface can be insecure.
```

4.从库导入数据

```bash
root@b142eb1f220c:/# mysql -uroot -p123456 < /var/lib/mysql/all.sql
mysql: [Warning] Using a password on the command line interface can be insecure.
```

5.从库：配置一个唯一的ID，重启

```conf
[mysqld]
server-id=2
```

6.配置复制账号，二进制文件的位置信息

```sql
mysql> change master to master_host='139.180.197.68', master_port=3306, master_user='repl', master_password='123456', master_log_file='binlog.000003', master_log_pos=940;
Query OK, 0 rows affected, 2 warnings (0.02 sec)
```


7.从库：开启同步线程

```sql
mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 139.180.197.68
...
```

![](https://images2015.cnblogs.com/blog/771870/201603/771870-20160309163148225-1200721404.png)

上面的两个进程都显示YES则表示配置成功。

8.主库：恢复写入操作

```sql
mysql> unlock tables;
```

> 对于主库已有数据，只能手工导入从库。

## 备份方法对比
|备份方法|备份速度|恢复速度|便捷性|功能|一般用于|
|:--|
|cp|快|快|一般、灵活性低|很弱|少量数据|
|mysqldump|慢|慢|一般、可无视存储引擎的差异|一般|中小型数据量的备份|
|lvm2快照|快|快|一般、支持几乎热备、速度快|一般|中小型数据量的备份|
|xtrabackup|较快|较快|实现innodb热备、对存储引擎有要求|强大|较大规模的备份|

## 常见问题
1. Last_IO_Error: error connecting to master
  主从复制，从库提示：Last_IO_Error: error connecting to master。

  原因：防火墙未关闭。

1. Slave failed to initialize relay log info structure from the repository
  MySQL主从复制，启动slave时出现报错。

  原因：保留之前relay_log的信息，所以导致启动slave报错。

  解决：reset salve; change master ...; start slave;

## 面试题
1. 全量备份和增量备份的区别？

1. 什么情况下需要增量恢复？

1. 只有一个主库是否需要增量恢复？
每天全量备份 + 每小时增量备份

1. 人为操作数据库SQL破坏主库是否需要增量恢复？

1. 主或从库宕机（硬件损坏）是否需要增量恢复？
不需要。主库宕机，切换从库。从库宕机，正常修复。

1. mysqldump 备份什么时候能派上用场？
跨机房灾备、迁移数据、增加从库

1. 什么是主从复制？

1. 数据库主从同步数据一致性如何解决？技术方案的优劣势比较？

1. MySQL 主从同步的实现原理？

1. 数据库崩溃时事务的恢复机制

1. myisamchk 是用来做什么的