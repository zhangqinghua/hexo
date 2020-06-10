---
title: Chapter 2.3 数据类型

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:23
---

MySQL中定义数据字段的类型对你数据库的优化是非常重要的。

MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串（字符）类型。

## 数值类型
MySQL支持所有标准SQL数值数据类型。

这些类型包括严格数值数据类型（INTEGER、SMALLINT、DECIMAL和NUMERIC），以及近似数值数据类型（FLOAT、REAL和DOUBLE PRECISION）。

关键字INT是INTEGER的同义词，关键字DEC是DECIMAL的同义词。

BIT数据类型保存位字段值，并且支持MyISAM、MEMORY、InnoDB和BDB表。

作为SQL标准的扩展，MySQL也支持整数类型TINYINT、MEDIUMINT和BIGINT。下面的表显示了需要的每个整数类型的存储和范围。

|类型|大小|范围|用途|
| :- |
|TINYINT|1 byte|-128，127|小整数值|
|SMALLINT|2 bytes|±3万|大整数值|
|MEDIUMINT|3 bytes|±800万，7位数|大整数值|
|INT|4 bytes|±20亿，10位数|大整数值|
|BIGINT|8 bytes|±19位数|极大整数值|
|FLOAT|4 bytes||单精度浮点数值|
|DOUBLE|8 bytes||双精度浮点数值|
|DECIMAL|DECIMAL(M, D) M + 2 或 D + 2|依赖于M和D的值|小数值|

## 日期和时间类型
表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。

每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

TIMESTAMP类型有专有的自动更新特性，将在后面描述。

|类型|大小|范围|格式|用途|
| :- |
|YEAR|1 bytes|1901, 2155|YYYY|年份值|
|DATE|3 bytes|1000-01-01, 9999-12-31|YYYY-MM-DD|日期值|
|TIME|3 bytes|-838:59:59, 838:59:59|HH:MM:SS|时间值或持续时间|
|TIMESTAMP|4 bytes|1970-01-01 00:00:00, 2038|YYYYMMDD HHMMSS|混合日期和时间值，时间戳|
|DATETIME|8 bytes|1000-01-01 00:00:00, 9999-12-31 23:59:59|YYYY-MM-DD HH:MM:SS|混合日期和时间值|

## 字符串类型
字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

|类型|大小|用途|
| :- |
|CHAR|0-255 bytes|定长字符串|
|VARCHAR|0-65535 bytes|变长字符串|
|TINYTEXT|0-255 bytes|短文本字符串|
|TEXT|0-65 535 bytes|长文本数据|
|MEDIUMTEXT|0-16 777 215 bytes|中等长度文本数据|
|LONGTEXT|0-4 294 967 295 bytes|极大文本数据|
|TINYBLOB|0-255 bytes|不超过255个字符的二进制字符串|
|BLOB|0-65 535 bytes|二进制形式的长文本数据|
|LONGBLOB|0-4 294 967 295 bytes|二进制形式的极大文本数据|
|MEDIUMBLOB|0-16 777 215 bytes|极大文本数据|

char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。

CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。

有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。