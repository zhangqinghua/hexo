---
title: SQL 训练题

cateogries:
- 数据库

date: 2018-05-25
---

很全的sql语句练习题。

![数据库表](001.png)

1. 查询课程1的成绩比课程2的成绩高的所有学生的学号
    ```sql
    SELECT
        a.* 
    FROM
        ( SELECT * FROM sc WHERE cno = 1 ) a,
        ( SELECT * FROM sc WHERE cno = 2 ) b 
    WHERE
        a.score > b.score 
        AND a.sno = b.sno
    -- sno  cno score
    -- 2	1	50
    -- 4	1	90
    -- 6	1	70
    -- 7	1	80
    ```

1. 查询平均成绩大于60分的同学的学号和平均成绩
    ```sql
    SELECT
        sno,
        avg( score ) AS sscore 
    FROM
        sc 
    GROUP BY
        sno 
    HAVING
        avg( score ) > 60
    -- sno  sscore
    -- 1	84.5000
    -- 3	72.0000
    -- 4	78.2500
    -- 5	95.7500
    -- 6	68.2500
    -- 7	74.4000
    ```