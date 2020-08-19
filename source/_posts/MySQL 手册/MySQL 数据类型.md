---
title: MySQL 数据类型

categories:
- MySQL 手册

date: 2020-07-07 00:00:12
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
|FLOAT|4 bytes|8精度|单精度浮点数值|
|DOUBLE|8 bytes|18精度|双精度浮点数值|
|DECIMAL|DECIMAL(M, D) M + 2 或 D + 2|依赖于M和D的值|小数值|

## 日期和时间类型
表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。

每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

TIMESTAMP类型有专有的自动更新特性，如下所示：

```sql
-- 在创建新记录的时候把这个字段设置为给定值，以后修改时不刷新
TIMESTAMP DEFAULT CURRENT_TIMESTAMP  

-- 在创建新记录的时候把这个字段设置为给定值，以后修改时刷新它
TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  

-- 在创建新记录的时候把这个字段设置为给定值，以后修改时刷新它
TIMESTAMP DEFAULT 'yyyy-mm-dd hh:mm:ss' ON UPDATE CURRENT_TIMESTAMP  

-- 在创建新记录的时候把这个字段设置为0，以后修改时刷新它 
TIMESTAMP ON UPDATE CURRENT_TIMESTAMP 
```

|类型|大小|范围|格式|用途|
| :- |
|YEAR|1 bytes|1901, 2155|YYYY|年份值|
|DATE|3 bytes|1000-01-01, 9999-12-31|YYYY-MM-DD|日期值|
|TIME|3 bytes|-838:59:59, 838:59:59|HH:MM:SS|时间值或持续时间|
|DATETIME|8 bytes|1000-01-01 00:00:00, 9999-12-31 23:59:59|YYYY-MM-DD HH:MM:SS|混合日期和时间值|
|TIMESTAMP|4 bytes|1970-01-01 00:00:00, 2038|YYYYMMDD HHMMSS|混合日期和时间值，时间戳|
## 字符串类型
字符串类型指 `CHAR`、`VARCHAR`、`BINARY`、`VARBINARY`、`BLOB`、`TEXT`、`ENUM` 和 `SET`。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

字符串类型可以分为存储文本的和存储二进制数据的类型，其中二进制数据可以是图片，视频等，很少用到。

|类型|大小|用途|
| :- |
|SET|0-255 bytes|多选字符串数据类型|
|ENUM|0-255 bytes|定长字符串|
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

`CHAR(n)` 和 `VARCHAR(n)` 中括号中 n 代表字符的个数，并不代表字节个数，比如 `CHAR(30)` 就可以存储 30 个字符。

`CHAR` 和 `VARCHAR` 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

`BINARY` 和 `VARBINARY` 类似于 `CHAR` 和 `VARCHAR`，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

`BLOB` 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 `BLOB` 类型：`TINYBLOB`、`BLOB`、`MEDIUMBLOB` 和 `LONGBLOB`。它们区别在于可容纳存储范围不同。

有 4 种 `TEXT` 类型：`TINYTEXT`、`TEXT`、`MEDIUMTEXT` 和 `LONGTEXT`。对应的这 4 种 `BLOB` 类型，可存储的最大长度不同，可根据实际情况选择。

> `BLOB` 和 `TEXT` 类型之间的唯一区别在于对 `BLOB` 值进行排序和比较时区分大小写，对 `TEXT` 值不区分大小写。

## 字符集
字符集是用来定义存储字符串的方式。MySQL 支持 30 多种字符集。

常见字符集：

|字符集|使用场景|最大长度|字符长度|
| :- |
|ascii|英文|1字节|英文1字节|
|utf8|各种语言|3字节|英文1字节、汉字3字节|
|utf8mb4|各种语言，表情|4字节|英文1字节、汉字4字节|

#### 查看默认字符集
```sql
mysql> SHOW VARIABLES LIKE 'character_set_server';
+----------------------+--------+
| Variable_name        | Value  |
+----------------------+--------+
| character_set_server | gbk    |
+----------------------+--------+
1 row in set, 1 warning (0.01 sec)

mysql> SHOW VARIABLES LIKE 'collation_server';
+------------------+-------------------+
| Variable_name    | Value             |
+------------------+-------------------+
| collation_server | gbk_chinese_ci    |
+------------------+-------------------+
1 row in set, 1 warning (0.01 sec)
```

#### 配置默认字符集
如果没有指定服务器字符集，MySQL 会默认使用 latin1 作为服务器字符集。

在配置文件配置默认字符集：

```cnf
[mysqld]
character-set-server=字符集名称
```

连接 MySQL 服务器时指定字符集：

```bash
mysql -uroot -p123 --default-character-set=utf8mb4
```

使用命令修改字符集：
```sql
-- use到当前的数据库中,修改字符集
mysql> use mydb

mysql> alter database mydb character set utf-8;
```

创建数据库时指定字符集：

```sql
CREATE DATABASE `tb_students_info` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

修改数据库时指定字符集：

```sql
ALTER DATABASE `tb_students_info` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

新增表时指定字符集：

```sql
CREATE TABLE `tb_students_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(10) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `height` float DEFAULT NULL,
  `course_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8
```

修改表时指定字符集：
```sql
ALTER TABLE `tb_students_info` TO CHARACTER SET utf8mb4;
```

## 校对规则
校对规则定义了比较字符串的方式。字符集和校对规则是一对多的关系, MySQL 支持 70 多种校对规则。

如果只指定了字符集，没有指定校对规则，MySQL 会使用该字符集对应的默认校对规则。如果要使用字符集的非默认校对规则，需要在指定字符集的同时指定校对规则。

#### 配置校对规则
创建数据库时指定字符集：

```sql
CREATE DATABASE `sptest` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

修改数据库时指定字符集：

```sql
ALTER DATABASE `sptest` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

## 字段、类型及其长度
|字段CN|字段EN|类型|长度|
| :- |
|**用户信息 userInfo**|
|昵称     |nickname   |varchar    |20     |
|姓名     |username   |varchar    |20     |    
|性别     |sex        |char       |1      |
|生日     |birthday   |date       |       |
|**钱包 wallet**|
|用户Id   |userId     |bigint     |20     |
|余额     |balance    |decimal    |10,2   |
|**钱包流水**|
|操作类型 |optype     |int        |1      |
|操作数额 |opAmount   |decimal    |10,2   |
|之前数额 |bfAmount   |decimal    |10,2   |
|之后数额 |afAmount   |decimal    |10,2   |
|业务Key  |busKey     |varchar    |256    |
|业务类型 |busType    |int        |1      |

1. 把IP地址存成 UNSIGNED INT
1. 