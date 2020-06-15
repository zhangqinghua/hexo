---
title: MySQL 存储引擎之MyISAM

categories:
- MySQL 指南

date: 2020-04-28 00:00:41
---
MyISAM 是 MySQL 的默认数据库引擎（5.5 版本之前），由早期的 ISAM（有索引的顺序访问方式）所改良。虽然性能极佳，但不支持事务处理。

## MyISAM 的文件类型
MyISAM 存储引擎的表存储成 3 个文件。文件的名字与表名相同，扩展名包括 **.frm**、**.MYD** 和 **.MYI**。

1. **.frm**：存储表的结构

1. **.MYD**：存储数据，是 MYData 的缩写

1. **.MYI**：存储索引，是 MYIndex 的缩写

## MyISAM 的存储格式
基于 MyISAM 存储引擎的表支持 3 种不同的存储格式，包括静态型、动态型和压缩型。

## 如何使用
创建表时指定表的存储引擎为 MyISAM。

```sql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `mobile` varchar(32) DEFAULT NULL COMMENT '注册手机号',
  `nickname` varchar(64) DEFAULT NULL COMMENT '发表人昵称',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

然后就可以在 **./data/phonerepairer/** 目录里看到 MyISAM 创建的 2 个文件。

```
user.frm
user.MYD
user.MYI
```

## MYI 索引文件
MyISAM 索引文件和数据文件是分离的，索引文件仅保存记录所在页的指针（物理位置），通过这些地址来读取页，进而读取被索引的行。

树中叶子保存的是对应行的物理位置。通过该值，存储引擎能顺利地进行回表查询，得到一行完整记录。同时，每个叶子页也保存了指向下一个叶子页的指针。从而方便叶子节点的范围遍历。

![](https://pic4.zhimg.com/80/v2-5094f94cd876c866b7b50481956ced6f_720w.jpg)

以内容存储。

如果username重复了。