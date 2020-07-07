---
title: 常用MySQL

categories:
- MySQL 指南

date: 2020-04-28 00:00:91
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

#### 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
这里只用根据学生 ID 把成绩分组，对分组中的 score 求平均值，最后在选取结果中 AVG 大于 60 的即可. 注意，这里必须要给计算得到的 AVG 结果一个 alias。（AS ss）得到学生信息的时候既可以用 join 也可以用一般的联合搜索

```sql
-- having
select st.sname name, sc.score score from
    student st,
    (select sid, avg(score) score from sc group by sid having score > 60)sc
where st.sid = sc.sid

-- right join
select Student.SId, Student.Sname, r.ss from Student right join(
      select SId, AVG(score) AS ss from sc
      GROUP BY SId
      HAVING AVG(score)> 60
)r on Student.SId = r.SId;

-- left join
select s.sid,ss,sname
from (select sid, AVG(score) as ss from sc GROUP BY sid HAVING AVG(score)> 60)r
left join (select Student.SId, Student.Sname from Student)s on s.SId = r.SId;

name	score
赵雷	89.66667
钱电	70.00000
孙风	80.00000
周梅	81.50000
郑竹	93.50000
```

#### 查询在 SC 表存在成绩的学生信息

```sql
select distinct(sname) from student, sc where student.sid = sc.sid

sname
赵雷
钱电
孙风
李云
周梅
吴兰
郑竹
```

#### 查询所有同学的学生编号、学生姓名、选课总数、所有课程的成绩总和
联合查询不会显示没选课的学生。

```sql
select student.sid, student.sname,r.coursenumber,r.scoresum
from student,
(select sc.sid, sum(sc.score) as scoresum, count(sc.cid) as coursenumber from sc
group by sc.sid)r
where student.sid = r.sid;

sid	sname	coursenumber	scoresum
01	赵雷	3	269.0
02	钱电	3	210.0
03	孙风	3	240.0
04	李云	3	100.0
05	周梅	2	163.0
06	吴兰	2	65.0
07	郑竹	2	187.0
```

如要显示没选课的学生(显示为NULL)，需要使用join。

```sql
select s.sid, s.sname,r.coursenumber,r.scoresum
from (
    (select student.sid,student.sname
    from student
    )s
    left join
    (select
        sc.sid, sum(sc.score) as scoresum, count(sc.cid) as coursenumber
        from sc
        group by sc.sid
    )r
   on s.sid = r.sid
);

sid	sname	coursenumber	scoresum
01	赵雷	3	269.0
02	钱电	3	210.0
03	孙风	3	240.0
04	李云	3	100.0
05	周梅	2	163.0
06	吴兰	2	65.0
07	郑竹	2	187.0
09	张三	null    null
10	李四	null    null
```

#### 查有成绩的学生信息
这一题涉及到in和exists的用法，在这种小表中，两种方法的效率都差不多。

当表2的记录数量非常大的时候，选用exists比in要高效很多

EXISTS用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False.

结论：IN()适合B表比A表数据小的情况

结论：EXISTS()适合B表比A表数据大的情况

```sql
-- in
select sid, sname from student where sid in (select sid from sc)

-- exists
select sid, sname from student where exists (select 1 from sc where sc.sid = student.sid)

sid	sname
01	赵雷
02	钱电
03	孙风
04	李云
05	周梅
06	吴兰
07	郑竹
```

#### 查询「李」姓老师的数量
```sql
select count(1) from teacher where tname like '李%';

count
1
```

#### 查询学过「张三」老师授课的同学的信息
多表联合查询

```sql
select s.sid, s.sname, t.tname from student s, sc, course c, teacher t
where s.sid = sc.sid
and   sc.cid = c.cid
and c.tid = t.tid
and t.tname = '张三';

sid	sname	tname
01	赵雷	张三
02	钱电	张三
03	孙风	张三
04	李云	张三
05	周梅	张三
07	郑竹	张三
```

#### 查询没有学全所有课程的同学的信息
因为有学生什么课都没有选，反向思考，先查询选了所有课的学生，再选择这些人之外的学生

```sql
select * from student
where student.sid not in (
  select sc.sid from sc
  group by sc.sid
  having count(sc.cid)= (select count(cid) from course)
);

sid	sname	ssex
05	周梅	女
06	吴兰	女
07	郑竹	女
09	张三	女
10	李四	女
11	李四	女
12	赵六	女
13	孙七	女
```

#### 查询至少有一门课与学号为"01"的同学所学相同的同学的信息
这个用联合查询也可以，但是逻辑不清楚，我觉得较为清楚的逻辑是这样的：从sc表查询01同学的所有选课cid--从sc表查询所有同学的sid如果其cid在前面的结果中--从student表查询所有学生信息如果sid在前面的结果中

```sql
select sid, sname from student
where student.sid in (
    select sc.sid from sc
    where sc.cid in(
        select sc.cid from sc
        where sc.sid = '01'
    )
);

sid	sname
01	赵雷
02	钱电
03	孙风
04	李云
05	周梅
06	吴兰
07	郑竹
```

#### 查询和"01"号的同学学习的课程完全相同的其他同学的信息
```sql
select sid from sc
where sid<>'01'
group by sid
having group_concat(cid order by cid ) =
    (select group_concat(cid order by cid ) from SC where sid = '01')
```

#### 查询没学过"张三"老师讲授的任一门课程的学生姓名
仍然还是嵌套，三层嵌套， 或者多表联合查询

```sql
select sid, sname from student
    where student.sid not in(
        select sc.sid from sc where sc.cid in(
            select course.cid from course where course.tid in(
                select teacher.tid from teacher where tname = "张三"
            )
        )
    );

select sid, sname from student
where student.sid not in(
    select sc.sid from sc,course,teacher
    where
        sc.cid = course.cid
        and course.tid = teacher.tid
        and teacher.tname= "张三"
);

sid	sname
06	吴兰
09	张三
10	李四
11	李四
12	赵六
13	孙七
```

#### 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
从SC表中选取score小于60的，并group by sid，having count 大于1

```sql
select student.sid, student.sname, AVG(sc.score) from student,sc
where
    student.sid = sc.sid and sc.score<60
group by sc.sid
having count(*)>1;

sid	sname	AVG(sc.score)
04	李云	33.33333
06	吴兰	32.50000
```

#### 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
双表联合查询，在查询最后可以设置排序方式，语法为ORDER BY ***** DESC\ASC;

```sql
select student.sid, student.sname, sc.score from student, sc
where student.sid = sc.sid
and sc.score < 60
and cid = "01"
ORDER BY sc.score DESC;

sid	sname	score
04	李云	50.0
06	吴兰	31.0
```

#### 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
```sql
select *  from sc
left join (
    select sid,avg(score) as avscore from sc
    group by sid
    )r
on sc.sid = r.sid
order by avscore desc;

sid	cid	score	sid(1)	avscore
07	02	89.0	07	93.50000
07	03	98.0	07	93.50000
01	03	99.0	01	89.66667
01	02	90.0	01	89.66667
01	01	80.0	01	89.66667
05	01	76.0	05	81.50000
05	02	87.0	05	81.50000
03	02	80.0	03	80.00000
```

#### 查询各科成绩最高分、最低分和平均分
以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率

及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90

要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```sql
select
sc.CId ,
max(sc.score)as 最高分,
min(sc.score)as 最低分,
AVG(sc.score)as 平均分,
count(*)as 选修人数,
sum(case when sc.score>=60 then 1 else 0 end )/count(*)as 及格率,
sum(case when sc.score>=70 and sc.score<80 then 1 else 0 end )/count(*)as 中等率,
sum(case when sc.score>=80 and sc.score<90 then 1 else 0 end )/count(*)as 优良率,
sum(case when sc.score>=90 then 1 else 0 end )/count(*)as 优秀率
from sc
GROUP BY sc.CId
ORDER BY count(*)DESC, sc.CId ASC

CId	最高分	最低分	平均分	选修人数	及格率	中等率	优良率	优秀率
01	80.0	31.0	64.50000	6	0.6667	0.3333	0.3333	0.0000
02	90.0	30.0	72.66667	6	0.8333	0.0000	0.5000	0.1667
03	99.0	20.0	68.50000	6	0.6667	0.0000	0.3333	0.3333
```

#### 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
这一道题有点tricky，可以用变量，但也有更为简单的方法，即自交（左交）

用sc中的score和自己进行对比，来计算“比当前分数高的分数有几个”。

```sql
select a.cid, a.sid, a.score, count(b.score)+1 as rank
from sc as a
left join sc as b
on a.score<b.score and a.cid = b.cid
group by a.cid, a.sid,a.score
order by a.cid, rank ASC;

cid	sid	score	rank
01	01	80.0	1
01	03	80.0	1
01	05	76.0	3
01	02	70.0	4
01	04	50.0	5
01	06	31.0	6
02	01	90.0	1
02	07	89.0	2
02	05	87.0	3
02	03	80.0	4
```

#### 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
这里主要学习一下使用变量。在SQL里面变量用@来标识。

```sql
set @crank=0;
select q.sid, total, @crank := @crank +1 as rank from(
select sc.sid, sum(sc.score) as total from sc
group by sc.sid
order by total desc)q;

sid	total	rank
01	269.0	1
03	240.0	2
02	210.0	3
07	187.0	4
05	163.0	5
04	100.0	6
06	65.0	7
```

#### 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
有时候觉得自己真是死脑筋。group by以后的查询结果无法使用别名，所以不要想着先单表group by计算出结果再从第二张表里添上课程信息，而应该先将两张表join在一起得到所有想要的属性再对这张总表进行统计计算。这里就不算百分比了，道理相同。
注意一下，用case when 返回1 以后的统计不是用count而是sum

```sql
select course.cname, course.cid,
sum(case when sc.score<=100 and sc.score>85 then 1 else 0 end) as "[100-85]",
sum(case when sc.score<=85 and sc.score>70 then 1 else 0 end) as "[85-70]",
sum(case when sc.score<=70 and sc.score>60 then 1 else 0 end) as "[70-60]",
sum(case when sc.score<=60 and sc.score>0 then 1 else 0 end) as "[60-0]"
from sc left join course
on sc.cid = course.cid
group by sc.cid;

cname	cid	[100-85]	[85-70]	[70-60]	[60-0]
语文	01	0	        3	1	2
数学	02	3	        1	0	2
英语	03	2	        2	0	2
```

#### 查询各科成绩前三名的记录
大坑比。mysql不能group by 了以后取limit，所以不要想着讨巧了，我快被这一题气死了。思路有两种，第一种比较暴力，计算比自己分数大的记录有几条，如果小于3 就select，因为对前三名来说不会有3个及以上的分数比自己大了，最后再对所有select到的结果按照分数和课程编号排名即可。

```sql
select * from sc
where (
select count(*) from sc as a
where sc.cid = a.cid and sc.score<a.score
)< 3
order by cid asc, sc.score desc;

sid	cid	score
01	01	80.0
03	01	80.0
05	01	76.0
01	02	90.0
07	02	89.0
05	02	87.0
01	03	99.0
07	03	98.0
02	03	80.0
03	03	80.0
```

order by cid asc, sc.score desc;
第二种比较灵巧一些，用自身左交，但是有点难以理解。
先用自己交自己，条件为a.cid = b.cid and a.score<b.score，其实就是列出同一门课内所有分数比较的情况。
想要查看完整的表可以

```sql
select * from sc a
left join sc b on a.cid = b.cid and a.score<b.score
order by a.cid,a.score;
```

查看，发现结果是47行的一个表，列出了类似 01号课里“30分小于50，也小于70，也小于80，也小于90”“50分小于70，小于80，小于90”.....
所以理论上，对任何一门课来说，分数最高的那三个记录，在这张大表里，通过a.sid和a.cid可以联合确定这个同学的这门课的这个分数究竟比多少个其他记录高/低，
如果这个特定的a.sid和a.cid组合出现在这张表里的次数少于3个，那就意味着这个组合（学号+课号+分数）是这门课里排名前三的。
所以下面这个计算中having count 部分其实count()或者任意其他列都可以，这里制定了一个列只是因为比count()运行速度上更快。

```sql
select a.sid,a.cid,a.score from sc a
left join sc b on a.cid = b.cid and a.score<b.score
group by a.cid, a.sid
having count(b.cid)<3
order by a.cid;
```

#### 查询每门课程被选修的学生数
```sql
select cid, count(sid) from sc
group by cid;

cid	count(sid)
01	6
02	6
03	6
```

#### 查询出只选修两门课程的学生学号和姓名

```sql
-- 嵌套查询
select student.sid, student.sname from student
where student.sid in
(select sc.sid from sc
group by sc.sid
having count(sc.cid)=2
);

-- 联合查询
select student.SId,student.Sname
from sc,student
where student.SId=sc.SId
GROUP BY sc.SId
HAVING count(*)=2；

SId	Sname
05	周梅
06	吴兰
07	郑竹
```

#### 查询男生、女生人数

```sql
select ssex, count(*) from student
group by ssex;

ssex	count(*)
女	8
男	4
```

#### 查询名字中含有「风」字的学生信息

```sql
select *
from student
where student.Sname like '%风%'
```

#### 查询同名学生名单，并统计同名人数

```sql
-- 找到同名的名字并统计个数
select sname, count(*) from student
group by sname
having count(*)>1;

sname	count(*)
李四	2

-- 嵌套查询列出同名的全部学生的信息
select * from student
where sname in (
select sname from student
group by sname
having count(*)>1
);

sid	sname	ssex
10	李四	女
11	李四	女
```

#### 查询 1990 年出生的学生名单

```sql
select *
from student
where YEAR(student.Sage)=1990;
```

#### 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
```sql
select sc.cid, course.cname, AVG(SC.SCORE) as average from sc, course
where sc.cid = course.cid
group by sc.cid
order by average desc,cid asc;

cid	cname	average
02	数学	72.66667
03	英语	68.50000
01	语文	64.50000
```

#### 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
having也可以用来截取结果表，在这里就先得到平均成绩总表，再截取AVG大于85的即可.

```sql
sid	sname	aver
01	赵雷	89.66667
07	郑竹	93.50000
```

#### 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数

```sql
select student.sname, sc.score from student, sc, course
where student.sid = sc.sid
and course.cid = sc.cid
and course.cname = "数学"
and sc.score < 60;

sname	score
李云	30.0
```

#### 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
```sql
select student.sname, cid, score from student
left join sc
on student.sid = sc.sid;

sname	cid	score
赵雷	01	80.0
赵雷	02	90.0
赵雷	03	99.0
钱电	01	70.0
钱电	02	60.0
钱电	03	80.0
孙风	01	80.0
孙风	02	80.0
```

#### 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数

```sql
select student.sname, course.cname,sc.score from student,course,sc
where sc.score>70
and student.sid = sc.sid
and sc.cid = course.cid;

sname	cname	score
赵雷	语文	80.0
赵雷	数学	90.0
赵雷	英语	99.0
钱电	英语	80.0
孙风	语文	80.0
孙风	数学	80.0
孙风	英语	80.0
```

#### 查询存在不及格的课程
可以用group by 来取唯一，也可以用distinct

```sql
-- group by
select cid from sc
where score< 60
group by cid;

-- distinct
select DISTINCT sc.CId
from sc
where sc.score <60;

cid
01
02
03
```

#### 查询课程编号为 01 且课程成绩在 80 分及以上的学生的学号和姓名

```sql
select student.sid,student.sname
from student,sc
where cid="01"
and score>=80
and student.sid = sc.sid;

sid	sname
01	赵雷
03	孙风
```

#### 求每门课程的学生人数

```sql
select sc.CId,count(*) as 学生人数
from sc
GROUP BY sc.CId;

CId	学生人数
01	6
02	6
03	6
```

#### 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
用having max()理论上也是对的，但是下面那种按分数排序然后取limit 1的更直观可靠

```sql
select student.*, sc.score, sc.cid from student, teacher, course,sc
where teacher.tid = course.tid
and sc.sid = student.sid
and sc.cid = course.cid
and teacher.tname = "张三"
having max(sc.score);

select student.*, sc.score, sc.cid from student, teacher, course,sc
where teacher.tid = course.tid
and sc.sid = student.sid
and sc.cid = course.cid
and teacher.tname = "张三"
order by score desc
limit 1;

sid	sname	sage	            ssex	score	cid
01	赵雷	1990-01-01 00:00:00	男	90.0	02
```

#### 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
为了验证这一题，先修改原始数据

```sql
UPDATE sc SET score=90
where sid = "07"
and cid ="02";
```

and cid ="02";
这样张三老师教的02号课就有两个学生同时获得90的最高分了。

这道题的思路继续上一题，我们已经查询到了符合限定条件的最高分了，这个时候只用比较这张表，找到全部score等于这个最高分的记录就可，看起来有点繁复。

```sql
select student.*, sc.score, sc.cid from student, teacher, course,sc
where teacher.tid = course.tid
and sc.sid = student.sid
and sc.cid = course.cid
and teacher.tname = "张三"
and sc.score = (
    select Max(sc.score)
    from sc,student, teacher, course
    where teacher.tid = course.tid
    and sc.sid = student.sid
    and sc.cid = course.cid
    and teacher.tname = "张三"
);

sid	sname	sage	ssex	score	cid
01	赵雷	1990-01-01 00:00:00	男	90.0	02
07	郑竹	1989-07-01 00:00:00	女	90.0	02
```

#### 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
同上，在这里用了inner join后会有概念是重复的记录：“01 课与 03课”=“03 课与 01 课”，所以这里取唯一可以直接用group by

```sql
select  a.cid, a.sid,  a.score from sc as a
inner join
sc as b
on a.sid = b.sid
and a.cid != b.cid
and a.score = b.score
group by cid, sid;

cid	sid	score
01	03	80.0
02	03	80.0
03	03	80.0
```

#### 查询每门功成绩最好的前两名
```sql
select a.sid,a.cid,a.score from sc as a
left join sc as b
on a.cid = b.cid and a.score<b.score
group by a.cid, a.sid
having count(b.cid)<2
order by a.cid;

sid	cid	score
01	01	80.0
03	01	80.0
01	02	90.0
07	02	90.0
01	03	99.0
07	03	98.0
```

#### 统计每门课程的学生选修人数（超过 5 人的课程才统计）

```sql
select sc.cid, count(sid) as cc from sc
group by cid
having cc >5;

cid	cc
01	6
02	6
03	6
```

#### 检索至少选修两门课程的学生学号

```sql
select sid, count(cid) as cc from sc
group by sid
having cc>=2;

sid	cc
01	3
02	3
03	3
04	3
05	2
06	2
07	2
```

#### 查询选修了全部课程的学生信息

```sql
select student.*
from sc ,student
where sc.SId=student.SId
GROUP BY sc.SId
HAVING count(*) = (select DISTINCT count(*) from course )

sid	sname	sage	ssex
01	赵雷	1990-01-01 00:00:00	男
02	钱电	1990-12-21 00:00:00	男
03	孙风	1990-05-20 00:00:00	男
04	李云	1990-08-06 00:00:00	男
```

#### 查询各学生的年龄，只按年份来算

#### 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一

```sql
select student.SId as 学生编号,student.Sname  as  学生姓名,
TIMESTAMPDIFF(YEAR,student.Sage,CURDATE()) as 学生年龄
from student

学生编号	学生姓名	学生年龄
01	赵雷	30
02	钱电	29
03	孙风	30
04	李云	29
05	周梅	28
06	吴兰	28
```

#### 查询本周过生日的学生

```sql
select *
from student
where WEEKOFYEAR(student.Sage)=WEEKOFYEAR(CURDATE());
```

#### 查询下周过生日的学生
```sql
select *
from student
where WEEKOFYEAR(student.Sage)=WEEKOFYEAR(CURDATE())+1;
```

#### 查询本月过生日的学生
```sql
select *
from student
where MONTH(student.Sage)=MONTH(CURDATE());
```

#### 查询下月过生日的学生
```sql
select *
from student
where MONTH(student.Sage)=MONTH(CURDATE())+1;
```

[50道SQL练习题及答案与详细分析](https://www.jianshu.com/p/476b52ee4f1b)