---
title: MySQL 查询语句

categories:
- MySQL 手册

date: 2020-07-07 00:00:07
---

MySQL 查询不区分大小写，下面的查询均有效：

```sql
SELECT VERSION(), CURRENT_DATE;
SeLect version(), current_date;
seleCt vErSiOn(), current_DATE;
```

#### LIKE 子句
`LIKE` 操作符通常用于基于模式查询选择数据。以正确的方式使用 `LIKE` 运算符对于增加/减少查询性能至关重要。

MySQL 中提供两个通配符，用于与 `LIKE` 运算符一起使用，它们分别是：百分比符号 `%` 和下划线 `_`。其中 `%` 用于通配符允许匹配任何字符串的零个或多个字符，类似正则表达式中的 `*`。`_` 通配符允许匹配任何单个字符。

示例：

```sql
-- 搜索名字以字符a开头的员工信息
SELECT * FROM employees WHERE firstName LIKE 'a%';

-- 搜索员工以on字符结尾的姓氏，例如，Patterson，Thompson
SELECT * FROM employees WHERE lastName LIKE '%on';

-- 查找 lastname 字段值中包含on字符串的所有员工
SELECT * FROM employees WHERE lastname LIKE '%on%';

-- 查找名字以T开头的员工，以m结尾，并且包含例如Tom，Tim之间的任何单个字符
SELECT * FROM employees WHERE firstname LIKE 'T_m';

-- 要搜索姓氏(lastname)不以字符B开头的员工。请注意，使用 LIKE 运算符，该模式不区分大小写，因此，b% 和 B% 模式产生相同的结果。
SELECT * FROM employees WHERE lastName NOT LIKE 'B%';
```

有时想要匹配的模式包含通配符，例如10%，_20等这样的字符串时。在这种情况下，您可以使用ESCAPE子句指定转义字符，以便 MySQL 将通配符解释为文字字符。如果未明确指定转义字符，则反斜杠字符 `\` 是默认转义字符。

```sql
-- 查询productCode字段中包含_20字符串的值
SELECT * FROM products WHERE productCode LIKE '%\_20%';

-- 指定一个不同的转义字符
SELECT * FROM products WHERE productCode LIKE '%$_20%' ESCAPE '$';
```

>  LIKE 操作符强制 MySQL 扫描整个表以找到匹配的行记录，因此，它不允许数据库引擎使用索引进行快速搜索。因此，当要从具有大量行的表查询数据时，使用LIKE运算符来查询数据的性能会大幅降低。

#### REGEXP 子句
MySQL 中使用 **REGEXP** 操作符来进行正则表达式匹配，跟 **LIKE** 的功能类似。

示例：

```sql
-- 查找name字段中以'st'为开头的所有数据 
SELECT name FROM person_tbl WHERE name REGEXP '^st';

-- 查找name字段中以'ok'为结尾的所有数据
SELECT name FROM person_tbl WHERE name REGEXP 'ok$';

-- 查找name字段中包含'mar'字符串的所有数据
SELECT name FROM person_tbl WHERE name REGEXP 'mar';

-- 查找name字段中以元音字符开头或以'ok'字符串结尾的所有数据
SELECT name FROM person_tbl WHERE name REGEXP '^[aeiou]|ok$';
```

操作符说明：

1. 位置操作符
    `^` 表示开始，`$` 表示结尾
1. 字符匹配操作符
    `.` 表示除 `\n` 之外的任何单个字符。
    `*` 任意数量的任意字符。 
    `+` 一个或多个任意字符。
1. 选择操作符
    `[...]` 表示字符集合。匹配所包含的任意一个字符。 
    `[^...]` 表示负值字符集合。匹配未包含的任意字符。 
    `p1|p2|p3`  匹配 p1 或 p2 或 p3。 
1. 数量操作符
    `a{5}` 表示 5 个 a，n 是一个非负整数。 
    在 `{n, m}`  中，m 和 n 均为非负整数，其中 n <= m。最少匹配 n 次且最多匹配 m 次。

## 问题
1. SQL 语言包括哪几部分，每部分都有哪些操作关键字
1. like 声明中的 ％ 和 _ 是什么意思
1. 列对比运算符是什么
1. 如何显示前 50 行
1. 解释 MySQL 外连接、内连接与自连接的区别
1. NULL 是什么意思
1. 查询语句不同元素（where、jion、limit、group by、having等等）执行先后顺序
1. MySQL 有哪几种 join 方式，底层原理是什么
1. MySQL limit 分页如何保证可靠性
1. sql语法（join，union，子查询，having，group by）