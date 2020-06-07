---
title: Chapter 2.2 SQL 练习题

categories:
- MySQL 性能调优

date: 2020-04-28 00:00:22
---
网上流传较广的50道SQL训练，奋斗了不知道多久终于写完了。前18道题的难度依次递增，从19题开始的后半部分算是循环练习和额外function的附加练习，难度恢复到普通状态。

第9题非常难，我反正没有写出来，如果有写出来了的朋友还请赐教。

这50道里面自认为应该没有太多错误，而且尽可能使用了最简单或是最直接的查询，有多种不相上下解法的题目我也都列出了，但也欢迎一起学习的朋友进行讨论和解法优化啊~

## 数据表
学生表 Student

```sql
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');
```

科目表 Course

```sql
create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```

教师表 Teacher
```sql
create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```

成绩表 SC

```sql
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

## 练习题目
#### 查询"01"课程比"02"课程成绩高的学生的信息及课程分数
因为需要全部的学生信息，则需要在 sc 表中得到符合条件的 sid 后与 student 表进行 join，可以 `left join` 也可以 `right join`。

```sql
-- left join
SELECT s.sname, sc.* FROM(
        select  t1.sid sid, t1.cid t1_cid, t1.score t1_socre, t2.cid t2_cid, t2.score t2_socre from 
                (select * from sc where sc.cid = '01') as t1,
                (select * from sc where sc.cid = '02') as t2
        where t1.sid = t2.sid and t1.score > t2.score
	) sc LEFT JOIN student s ON s.sid = sc.sid

-- right join
select s.sname, sc.* from student s right join( 
        select  t1.sid sid, t1.cid t1_cid, t1.score t1_socre, t2.cid t2_cid, t2.score t2_socre from 
                (select * from sc where sc.cid = '01') as t1,
                (select * from sc where sc.cid = '02') as t2
        where t1.sid = t2.sid and t1.score > t2.score
) sc on s.sid = sc.sid

sname	sid	t1_cid	t1_socre	t2_cid	t2_socre
钱电	02	01	70.0		02	60.0
李云	04	01	50.0		02	30.0
```

#### 查询同时存在"01"课程和"02"课程的情况
```sql
-- 以Student作为主表
select * from 
    (select * from sc where sc.CId = '01') as t1, 
    (select * from sc where sc.CId = '02') as t2
where t1.SId = t2.SId;

SId	CId	score	SId(1)	CId(1)	score(1)
01	01	80.0	01	02	90.0
02	01	70.0	02	02	60.0
03	01	80.0	03	02	80.0
04	01	50.0	04	02	30.0
05	01	76.0	05	02	87.0

-- 以SC作为主表
???
```

#### 查询存在"01"课程但可能不存在"02"课程的情况(不存在时显示为 null)
这一道就是明显需要使用 join 的情况了，02可能不存在，即为 `left join` 的右侧或 `right join` 的左侧即可。

```sql
-- left join 
select * from 
(select * from sc where sc.CId = '01') as t1
left join 
(select * from sc where sc.CId = '02') as t2
on t1.SId = t2.SId;

-- right join 
select * from 
(select * from sc where sc.CId = '02') as t2
right join 
(select * from sc where sc.CId = '01') as t1
on t1.SId = t2.SId;

SId	CId	score	SId(1)	CId(1)	score(1)
01	01	80.0	01	02	90.0
02	01	70.0	02	02	60.0
03	01	80.0	03	02	80.0
04	01	50.0	04	02	30.0
05	01	76.0	05	02	87.0
06	01	31.0	null    null    null		
```

[50道SQL练习题及答案与详细分析](https://www.jianshu.com/p/476b52ee4f1b)
