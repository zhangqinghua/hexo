---
title: Chapter 4.1 MySQL 常用存储引擎之 MyISAM

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:41
---
MyISAM 存储引擎是 MySQL 中常见的存储引擎，曾是 MySQL 的默认存储引擎。MyISAM 存储引擎是基于 ISAM 存储引擎发展起来的，它解决了 ISAM 的很多不足。MyISAM 增加了很多有用的扩展。

## MyISAM 的文件类型
MyISAM 存储引擎的表存储成 3 个文件。文件的名字与表名相同，扩展名包括 **.frm**、**.MYD** 和 **.MYI**。

1. **.frm**：存储表的结构

1. **.MYD**：存储数据，是 MYData 的缩写

1. **.MYI**：存储索引，是 MYIndex 的缩写

## MyISAM 的存储格式
基于 MyISAM 存储引擎的表支持 3 种不同的存储格式，包括静态型、动态型和压缩型。