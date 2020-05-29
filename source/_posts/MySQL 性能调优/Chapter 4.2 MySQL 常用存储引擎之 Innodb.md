---
title: Chapter 4.2 MySQL 常用存储引擎之 Innodb

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:42
---

## 什么是 InnoDB？

## InnoDB 有什么特点

## MyISAM 和 InnoDB 的区别
1. 锁的区别
    MyISAM 常用的是表级锁，而 InnoDB 采用的是行级锁（当索引失败的时候切换回表级锁）。

1. 对事务的支持
    MyISAM 是不支持事务的，而 InnoDB 支持事务操作。如果您的数据安全性要求高，那么最好选择 InnoDB。

1. 存储结构的区别
    MyISAM 在磁盘上会存储 3 个文件，其中 **.frm** 文件存储表定义，**.myd** 存储数据文件，**.myi** 存储索引文件。
    InnoDB 在磁盘上会存储 2 个文件，其中 **.frm** 存储表结构文件，**.ibd** 文件存储数据和索引。

    由此可见，MyISAM 的叶子节点存储的是数据所在的地址，而不是数据。InnoDB 叶子节点存储的是整个数据行的所有数据。

    ![](001.png)