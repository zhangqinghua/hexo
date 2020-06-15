---
title: MySQL 存储引擎之InnoDB

categories:
- MySQL 指南

date: 2020-04-28 00:00:42
---
InnoDB是 MySQL 的数据库引擎之一，与传统的 MyISAM 相比，InnoDB 最大的特点是支持了 ACID 兼容的事务功能。

## InnoDB 有什么特点

## MyISAM 和 InnoDB 的区别
1. 锁的区别
    MyISAM 常用的是表级锁，而 InnoDB 采用的是行级锁（当索引失败的时候切换回表级锁）。

1. 对事务的支持c
    MyISAM 是不支持事务的，而 InnoDB 支持事务操作。如果您的数据安全性要求高，那么最好选择 InnoDB。

1. 存储结构的区别
    MyISAM 在磁盘上会存储 3 个文件，其中 **.frm** 文件存储表定义，**.myd** 存储数据文件，**.myi** 存储索引文件。
    InnoDB 在磁盘上会存储 2 个文件，其中 **.frm** 存储表结构文件，**.ibd** 文件存储数据和索引。

    由此可见，MyISAM 的叶子节点存储的是数据所在的地址，而不是数据。InnoDB 叶子节点存储的是整个数据行的所有数据。

## 适用范围
InnoDB 表是如下情况理想的存储引擎。

1. 更新密集的表：InnoDB 存储引擎特别适合处理多重并发的更新要求。

1. 事务：InnoDB 是唯一支持事务的标准 MySQL 存储引擎，这是管理敏感数据（如金融信息和用户注册信息）的刚需。

1. 自动灾难恢复：与其他存储引擎不同，InnoDB 表能够自动从灾难中恢复。虽然 MyISAM 表能在灾难后修复，但其过程要长得多。

## 如何使用
创建表时指定表的存储引擎为 InnoDB。

```sql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `mobile` varchar(32) DEFAULT NULL COMMENT '注册手机号',
  `nickname` varchar(64) DEFAULT NULL COMMENT '发表人昵称',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

然后就可以在 **./data/phonerepairer/** 目录里看到 InnoDB 创建的 2 个文件。

```
user.frm
user.ibd
```

