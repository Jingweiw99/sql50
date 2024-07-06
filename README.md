> 50题经典的sql练习

## 创建数据库和表

```sql
dorp database if exists mysql_test cascade;
create database mysql_test;
use mysql_test;
```

学生表
创建学生表student

```sql
create table student (
  s_id int,
  s_name varchar(8),
  s_birth date,
  s_sex varchar(4)
);
```

插入学生数据

```sql
insert into student values
(1,'赵雷','1990-01-01','男'),
(2,'钱电','1990-12-21','男'),
(3,'孙风','1990-05-20','男'),
(4,'李云','1990-08-06','男'),
(5,'周梅','1991-12-01','女'),
(6,'吴兰','1992-03-01','女'),
(7,'郑竹','1989-07-01','女'),
(8,'王菊','1990-01-20','女');
```

课程表
创建课程表course

```sql
create table course (
  c_id int,
  c_name varchar(8),
  t_id int
);
```

插入课程数据

```sql
insert into course values
(1,'语文',2),
(2,'数学',1),
(3,'英语',3);
```

教师表teacher
创建教师表teacher

```sql
create table teacher (
	t_id int,
	t_name varchar(8)
);

insert into teacher values
(1,'张三'),
(2,'李四'),
(3,'王五');
```

成绩表
成绩表score

```sql
create table score (
	s_id int,
	c_id int,
	s_score int
);
insert into score values
(1,1,80),
(1,2,90),
(1,3,99),
(2,1,70),
(2,2,60),
(2,3,65),
(3,1,80),
(3,2,80),
(3,3,80),
(4,1,50),
(4,2,30),
(4,3,40),
(5,1,76),
(5,2,87),
(6,1,31),
(6,3,34),
(7,2,89),
(7,3,98);
```

表关系
![image.png](https://cdn.nlark.com/yuque/0/2024/png/26264991/1719934770985-f4ab446b-6013-4670-8b16-cf29a690f1f2.png#averageHue=%232d2d2c&clientId=u0c474d85-159f-4&from=paste&height=232&id=ufda01373&originHeight=284&originWidth=1078&originalType=binary&ratio=1.2260417938232422&rotation=0&showTitle=false&size=18349&status=done&style=none&taskId=u49f7115c-f997-4c86-b972-957573ed761&title=&width=879.2522452586268)

## 1. 查询"1"课程比"2"课程成绩高的学生的信息及课程分数

```sql
select s.*, sc1.s_score as score_01, sc2.s_score as score_02
from student s
         inner join (
    select *
    from score
    where c_id = 1
) sc1
                    on s.s_id = sc1.s_id
         inner join (
    select *
    from score
    where c_id = 2
) sc2
                    on s.s_id = sc2.s_id
where sc1.s_score > sc2.s_score;
```

## 2. 查询01课程比02课程 成绩低的学生信息及课程分数

```sql
# 上述1方法稍微修改
select stu.*, s.score01, s.score02
from student stu
         inner join
     (select s1.`s_id` s_id, s1.s_score score01, s2.s_score score02
      from score s1
               inner join (select * from score where c_id = 2) s2 on s1.s_id = s2.s_id
      where s1.c_id = 1) s
     on stu.s_id = s.s_id
where s.score01 < s.score02;
```

## 3. 查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

```sql
# 3. 查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
select s_id, avg(s_score)
from score
group by s_id
having avg(s_score) >= 60;
select stu.*, ifnull(round(s.`avg(s_score)`, 2), 0) 平均成绩
from student stu
         inner join (select s_id, avg(s_score)
                     from score
                     group by s_id
                     having avg(s_score) >= 60) s on stu.s_id = s.s_id
```

## 4. 查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩

```sql
# 4. 查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
select stu.s_id s_id, stu.s_name s_name, ifnull(round(avg_score, 2), 0) avg_score
from student stu
         left join (select sc.s_id, avg(s_score) avg_score
                    from score sc
                    group by s_id
                    having avg(sc.s_score) < 60) s on stu.s_id = s.s_id;

# !!!注意了： 这里忽略avg 运算忽略null值，导致s_id为8 的数据没算上。 
# 题目要求包括无成绩的。因此上面方法有误
# 将分数判断为null也输出为平均成绩为0 先判断null然后ifnull为0
# 正确sql语句如下
select s.s_id, s.s_name, ifnull(round(avg_score, 2), 0) as avg_score
from student s
         left join (
    select s_id, avg(s_score) as avg_score
    from score
    group by s_id
) t1
                   on s.s_id = t1.s_id
where avg_score is null
   or avg_score < 60;
```

## 5. 查询所有同学的学生编号、学生姓名，选课总数、所有课程的总成绩

```sql
# 5.  查询所有同学的学生编号、学生姓名，选课总数、所有课程的总成绩。
# s.id s_name (student) 选课总数（score 通过s_id聚合计算）所有课程的总成绩(score表 聚合)
# 注意执行顺序

select stu.s_id s_id, s_name, ifnull(count, 0) count, ifnull(sum_score, 0) sum_score
from student stu
         left join (select sc.s_id, count(*) count, sum(s_score) sum_score
                    from score sc
                    group by s_id) ss on stu.s_id = ss.s_id
```

## 6. 查询李姓老师的数量

```sql
# 6. 查询 李 姓老师的数量
# teacher表: count数量 条件：t_name like '李%'
select count(*) as count
from teacher
where t_name like '李%'
```

## 7. 查询学过“张三”老师授课的同学的信息

```sql
# 7. 查询学过“张三”老师授课的同学的信息
# student: student.*
# 张三老师找到teacher表的t_id，关联course表的t_id，找到c_id，关联score标的c_id，找到s_id，关联student表的s_id。 连接之后可以查询了。
# 两种
select *
from student
where s_id in (select s_id
               from score
               where c_id = (select c_id
                             from course
                             where t_id = (select t_id
                                           from teacher
                                           where t_name = '张三')));

select *
from student
where s_id in (select s_id
               from score sc
                        inner join (select c_id
                                    from course
                                             inner join (select t_id
                                                         from teacher
                                                         where t_name = '张三') t on course.t_id = t.t_id) s
                                   on sc.c_id = s.c_id);
```

## 8. 查询没学过"张三"老师授课的同学的信息

```sql
# 8. 查询没学过"张三"老师授课的同学的信息
# student: *
# 没学过张三老师授课的学生 不等于老师不是张三的课程下对应的学生 最后再进行过滤
select *
from student
where s_id not in (select distinct s_id
                   from score sc
                            inner join (select c_id
                                        from course
                                                 inner join (select t_id
                                                             from teacher
                                                             where t_name = '张三') t on course.t_id = t.t_id) s
                                       on sc.c_id = s.c_id);
```

## 9. 查询学过编号为1也学过编号为2的课程的同学的信息

```sql
# 9. 查询学过编号为1也学过编号为2的课程的同学的信息
# student: *
# score表中找出兼有c_id为1，也含有c_id为2的s_id， student在这些s_id里面查询

# !!! 注意这个思路 选择1,2之间的数据，正好是两条 说明既含有1也含有2
select *
from student
where s_id in (
    select s_id
    from score
    where c_id in (1, 2)
    group by s_id
    having count(*) = 2
)
```

## 10. 查询学过编号为01，但是没有学过编号为02课程的同学的信息

```sql
# 10. 查询学过编号为01，但是没有学过编号为02课程的同学的信息
# 注意： 这里思路不能通过c_id=2来连接，转换为找课程不为2的学生s_id。
select *
from student
where s_id in (
    select s_id
    from score
    where c_id = 1
      and s_id not in (
        select s_id
        from score
        where c_id = 2
    )
)
```

## 11. 查询没有学全所有课程的同学的信息

```sql
# 11. 查询没有学全所有课程的同学的信息
# 分组之后学生课程的数量和 课程表的中数量对比
select *
from student
where s_id not in (
    select s_id
    from score
    group by s_id
    having count(*) = (
        select count(*)
        from course
    )
)
```

## 12. 查询至少有一门课与学号1的同学所学相同的同学的信息

```sql
# 12. 查询至少有一门课与学号1的同学所学相同的同学的信息
# 连接s_id等于1的c_id条件 满足条件
select stu.*
from student stu
where s_id in (select distinct s_id
               from score sc
                        inner join(select c_id from score where s_id = 1) sc2
                                  on sc.c_id = sc2.c_id);
```

## 13. 询和01号同学学习的课程完全相同的其他同学的信息

```sql
# 13. 查询和01号同学学习的课程完全相同的其他同学的信息
# student表 : *
# 01同学学习的课程（score表 s_id 查询c_id） 注意这里还需要排除 自己。
# 完全相同
# 两种方式
select stu.*
from student stu
where stu.s_id in (select s_id
                   from score sc
                   where sc.c_id in (
                       select c_id
                       from score
                       where s_id = 1
                   )
                     and s_id != 1
                   group by s_id
                   having count(*) = (select count(*)
                                      from score
                                      where s_id = 1));

select *
from student
where s_id in (
    select s_id
    from score s
             inner join (
        select c_id
        from score
        where s_id = 1
    ) t1
                        on s.c_id = t1.c_id
    where s_id != 1
    group by s_id
    having count(*) = (
        select count(*)
        from score
        where s_id = 1
    )
);
```

## 14. 查询没学过"张三"老师讲授任意门课程的学生姓名

```sql
# 14. 查询没学过"张三"老师讲授任意门课程的学生姓名
# 找学过"张三老师"讲授任意门课程的学生姓名 然后最后取反
# student表的s_name

# 这里是单列的没有使用关联查询 单列或者单列单行
select stu.s_name
from student stu
where stu.s_id not in (select s_id
                       from score
                       where c_id in (select c_id
                                      from course
                                      where t_id = (
                                          select t_id
                                          from teacher
                                          where t_name = "张三"
                                      )));
```

## 15. 查询两门及其以上不及格课程的同学的 学号，姓名及其平均成绩

```sql
# 15. 查询两门及其以上不及格课程的同学的 学号，姓名及其平均成绩
# student表的s_id,s_name,course表的 计算出来的avg(score) group by s_id
# 两门以上不及格 (先通过表中找到所有不及格的，然后分组，找到这里面行数大于等于2的s_id)
select stu.s_id, stu.s_name, sc.`avg(s_score)` avg_score
from student stu
         inner join (select s_id, avg(s_score)
                     from score
                     where s_score < 60
                     group by s_id
                     having (count(*) >= 2)) sc on stu.s_id = sc.s_id
```

## 16. 检索01课程分数小于60，按分数降序排列的学生信息

```
# 16. 检索01课程分数小于60，按分数降序排列的学生信息
# student表: *
select stu.*, s_score
from student stu
         inner join (select s_id, s_score
                     from score
                     where c_id = 1
                       and s_score < 60) sc on stu.s_id = sc.s_id
order by s_score desc;

```

## 17. 按平均成绩从高到底显示所有学生的所有课程的成绩以及平均成绩

```sql
# 17. 按平均成绩从高到底显示所有学生的所有课程的成绩以及平均成绩
# score表: 所有课程(course表)，课程成绩，平均成绩
# 平均成绩从高到低
# 这个没有将数据合并
select sc.s_id, c_name, s_score, `avg(s_score)` avg_score
from score sc
         inner join course c on sc.c_id = c.c_id
         inner join (select s_id, avg(s_score)
                     from score
                     group by s_id) scc on sc.s_id = scc.s_id
order by avg_score desc;

# 优化
select s.s_id                                        s_id,
       sum(case c_id when 1 then s_score else 0 end) 语文,
       sum(case c_id when 2 then s_score else 0 end) 数学,
       sum(case c_id when 3 then s_score else 0 end) 英语,
       ifnull(round(avg(s_score), 2), 0)             avg_score
from student stu
         inner join score s on stu.s_id = s.s_id
group by s.s_id
```

## 18. 查询各科成绩的做高分，最低分和平均分

```sql
# 18. 查询各科成绩的做高分，最低分和平均分
# 课程id，课程name，最高分，最低分，及格率，中等率，优良率，优秀率，
# 及格:>=60 中等：70-80 优良80-90，优秀>=90 各科成绩
# score表:最高分c，最低分c，平均分c，及格率c，中等率c，优良率c，优秀率c course表: 课程id，课程name
select co.c_id,
       max(s_score)                                                                                     最高分,
       min(s_score)                                                                                     最低分,
       ifnull(round(avg(s_score), 2), 0)                                                                平均分,
       concat(sum(case when s_score >= 60 then 1 else 0 end) / count(*) * 100, '%')                     及格率,
       concat(sum(case when s_score >= 70 and sc.s_score < 80 then 1 else 0 end) / count(*) * 100, '%') 中等率,
       concat(sum(case when s_score >= 80 and sc.s_score < 90 then 1 else 0 end) / count(*) * 100, '%') 优良率,
       concat(sum(case when s_score >= 90 then 1 else 0 end) / count(*) * 100, '%')                     优秀率
from score sc
         left join course co on sc.c_id = co.c_id
group by co.c_id

```

## 19. 按各科成绩进行排序，并显示排名

```sql
# 19. 按各科成绩进行排序，并显示排名
# score表: c_id， s_score, course表: c_name   rankc
select stu.s_id, stu.s_name, t1.c_id, t1.s_score, rank() over (partition by c_id order by s_score desc) rank_no
from student stu
         inner join(select s_id, c_id, s_score from score) t1 on t1.s_id = stu.s_id
order by c_id, s_score desc;


```

## 20. 查询学生总成绩并进行排名

```sql
# 20. 查询学生总成绩并进行排名
# @i := @i + 1代表一个变量，表示每次加1 自增长序列从0开始
select (@i := @i + 1) rank_no, t2.*
from (select @i := 0) var
         inner join (
    select s.s_id, s.s_name, sum_score
    from student s
             inner join (
        select s_id, sum(s_score) as sum_score
        from score
        group by s_id
    ) t1
                        on s.s_id = t1.s_id
    order by sum_score desc
) t2;

```

## 21. 查询不同老师所教的不同课程平均分从高到低显示

```sql
# 21. 查询不同老师所教的不同课程平均分从高到低显示
# course表: c_id，c_name课程  | teacher表: t_name老师名字 | score表: s_score平均分c

# select t3.*, t4.*
select t3.t_id t_id, c_id, c_name, round(avg_score, 2) avg_score, t_name
from (select t2.t_id t_id, t2.c_id c_id, c_name, avg(s_score) avg_score
      from score t1
               inner join(select t_id, c_id, c_name
                          from course
      ) t2 on t1.c_id = t2.c_id
      group by t_id, c_id, c_name
      order by avg_score desc) t3
         inner join teacher t4 on t3.t_id = t4.t_id;
```

## 22. 查询所有课程成绩第2名到第3名的学生信息及该课程成绩

```sql
# mysql8支持
select c.c_id,
       c.c_name,
       s.s_id,
       s.s_name,
       s.s_birth,
       s.s_sex,
       sc.s_score
from (
         select c_id,
                s_id,
                s_score,
                row_number() over (partition by c_id order by s_score desc ) rank_no
         from score
     ) sc
         inner join student s on sc.s_id = s.s_id
         inner join course c on sc.c_id = c.c_id
where sc.rank_no IN (2, 3)
order by c.c_id, sc.rank_no;
```

## 23. 统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

```sql
# course表: c_id课程编号,c_name课程名称   score表: 分数段人数 s_id
# 注意这里case when的用法 等值或者条件判断
select s.*, c_name
from course c
         inner join
     (select t1.c_id                                                           c_id,
             sum(case when s_score >= 85 and s_score <= 100 then 1 else 0 end) `[100-85]`,
             sum(case when s_score >= 70 and s_score < 85 then 1 else 0 end)   `[85-70]`,
             sum(case when s_score >= 60 and s_score < 70 then 1 else 0 end)   `[70-60]`,
             sum(case when s_score >= 0 and s_score < 60 then 1 else 0 end)    '[0-60]'
      from course t1
               inner join score t2 on t1.c_id = t2.c_id
      group by c_id) s on c.c_id = s.c_id
```

## 24. 查询学生的平均成绩及其名次

```sql
# score: s_id, s_score平均成绩c， 平均成绩的名次
select s_id, avg(s_score) avg_score
from score t1
group by s_id
order by avg_score


# 上面这种写法是错误的，因为先执行了select，然后order by
# 正确做法一 内部排序后内联出排序完整的排名
# select *
select t3.s_id s_id, t3.avg_score avg_score, rank_no
from (select s_id, avg(s_score) avg_score
      from score t1
      group by s_id) t3
         inner join (
    select t2.*, (@i := @i + 1) rank_no
    from (select s_id, avg(s_score) avg_score
          from score t1
          group by s_id
          order by avg_score desc) t2
             inner join (select @i := 0) var
) t4 on t3.s_id = t4.s_id
order by s_id;
# 正确做法二 排序后排名
select (@i := @i + 1) rank_no, t2.*
from (select @i := 0) var
         inner join (
    select s.s_id, s.s_name, avg_score
    from student s
             inner join (
        select s_id, round(avg(s_score), 2) as avg_score
        from score
        group by s_id
    ) t1
                        on s.s_id = t1.s_id
    order by avg_score desc
) t2;
```

## 25. 查询各科成绩前三名的记录

```sql
# 25. 查询各科成绩前三名的记录
# score表: c_id,s_score course表: c_name
# 内外关联取前三条
select c.c_id, c.c_name, s.s_id, s.s_name, s_score
from (
         select *
         from score sc
         where (
                   select count(*)
                   from score sc1
                   where sc.c_id = sc1.c_id
                     and sc.s_score < sc1.s_score
               ) < 3
     ) t1
         inner join student s on t1.s_id = s.s_id
         inner join course c on t1.c_id = c.c_id
order by c.c_id, s_score desc;
```

## 26. 查询每门课程被选修的学生数

```sql
# 26. 查询每门课程被选修的学生数
# course表: c_id, c_name课程名字, score表: c_id计算学生数
# 先连接后查询
select t3.c_id c_id, course_count, c_name
from (select t1.c_id, count(*) course_count
      from score t1
               inner join course t2 on t1.c_id = t2.c_id
      group by t1.c_id) t3
         inner join course t4 on t3.c_id = t4.c_id;
# 先查询后连接
select t1.c_id c_id, course_count, c_name
from (select count(*) course_count, c_id
      from score
      group by c_id) t1
         inner join course t2 on t1.c_id = t2.c_id

```

## 27. 查询只有两门课程的全部学生的学号和姓名

```sql
# 27. 查询只有两门课程的全部学生的学号和姓名
# student表: s_id,s_name
# 条件 c_id的数量为2
select t1.s_id s_id, s_name
from student t1
         inner join (select s_id, count(*) course_count
                     from score
                     group by s_id
                     having count(*) = 2
) t2 on t1.s_id = t2.s_id;

select t1.s_id s_id, s_name
from student t1
where s_id in (select s_id
               from score
               group by s_id
               having count(*) = 2);
```

## 28. 查询男生、女生人数

```sql
# 28. 查询男生、女生人数
select s_sex, count(*) 人数
from student
group by s_sex
```

## 29. 查询名字中带有"风"字的学生信息

```sql
select *
from student
where s_name like '%风%'
```

## 30. 查询同名同性别学生名单，并统计同名人数

```sql
# 30. 查询同名同性别学生名单，并统计同名人数
select s_name, s_sex, count(*) count_stu
from student
group by s_name, s_sex
having count_stu > 1;
```

## 31. 查询1990年出生的学生名单

```sql
select *
from student
where '1990' = year(s_birth)
```

## 32. 查询每门课程平均成绩，结果按平均成绩降序排列，平均成绩相同时，按照课程编号升序排列

```sql
# 32. 查询每门课程平均成绩，结果按平均成绩降序排列，平均成绩相同时，按照课程编号升序排列。
select t1.c_id c_id, c_name, avg_score
from course t1
         inner join (select c_id, round(avg(s_score), 2) avg_score
                     from score
                     group by c_id
) t2 on t1.c_id = t2.c_id
order by avg_score desc, c_id
```

## 33. 查询平均成绩大于等于85的所有学生的学号，姓名和平均成绩

```sql
# 33. 查询平均成绩大于等于85的所有学生的学号，姓名和平均成绩
select t1.s_id s_id, s_name, avg_score
from student t1
         inner join (select s_id, round(avg(s_score), 2) avg_score from score group by s_id having avg_score >= 85) t2
                    on t1.s_id = t2.s_id
```

## 34. 查询课程名称为数学，且分数低于60的学生姓名和分数

```sql
# 全连接后查询
select s_name, s_score
from student t1
         inner join score t2 on t1.s_id = t2.s_id
         inner join course t3 on t2.c_id = t3.c_id
where c_name = '数学'
  and s_score < 60
# 先查询条件
select s_name, s_score
from student s
         inner join (
    select s_id, s_score
    from score sc
             inner join course c on sc.c_id = c.c_id
    where c_name = '数学'
      and s_score < 60
) t1
                    on s.s_id = t1.s_id;
```

## 35. 获取所有学生的课程及分数情况

```sql
# 35. 获取所有学生的课程及分数情况
# 学生名字，课程，分数
select s_name, c_name, s_score
from student t1
         inner join score t2 on t1.s_id = t2.s_id
         inner join course t3 on t2.c_id = t3.c_id
# 现在 是分开的数据，将其合并为一行
select t1.s_id s_id, s_name, 语文, 数学, 英语
from student t1
         inner join (
    select t2.s_id                                            s_id,
           sum(case c_name when '语文' then s_score else 0 end) '语文',
           sum(case c_name when '数学' then s_score else 0 end) '数学',
           sum(case c_name when '英语' then s_score else 0 end) '英语'

    from student t1
             inner join score t2 on t1.s_id = t2.s_id
             inner join course t3 on t2.c_id = t3.c_id
    group by s_id
) t2 on t1.s_id = t2.s_id

```

## 36. 查询课程成绩在70分以上的学生姓名，课程名称和分数

```sql
# 36. 查询课程成绩在70分以上的学生姓名，课程名称和分数
select s_name, s_score, c_name
from student t1
         inner join (select *
                     from score
                     where s_score > 70) t2 on t1.s_id = t2.s_id
         inner join course t3 on t2.c_id = t3.c_id
```

## 37. 查询课程不及格的学生

```sql
# 37. 查询课程不及格的学生
select s_name, s_score, c_name
from student t1
         inner join score t2 on t1.s_id = t2.s_id
         inner join course t3 on t2.c_id = t3.c_id
where s_score < 60
```

## 38. 查询课程编号为1且课程成绩在80分及以上的学生的学号和姓名

```sql
# 38. 查询课程编号为1且课程成绩在80分及以上的学生的学号和姓名
select t1.s_id s_id, s_name
from student t1
         inner join score t2 on t1.s_id = t2.s_id
where t2.c_id = '1'
  and t2.s_score >= 80
```

## 39. 求每门课程的学生人数

```sql
select t1.c_id c_id, c_name, course_count
from course t1
         inner join (select c_id, count(*) course_count from score group by c_id) t2 on t1.c_id = t2.c_id;
```

## 40. 查询选修"张三"老师授课的学生中，成绩最高的学生信息及其成绩

```sql
# 40. 查询选修"张三"老师授课的学生中，成绩最高的学生信息及其成绩
select t3.*, max_score
from student t3
         inner join (select s_id, s_score max_score
                     from score t1
                              inner join (select c_id
                                          from course
                                                   inner join teacher on course.t_id = teacher.t_id
                                          where t_name = '张三'
                     ) t2 on t1.c_id = t2.c_id
                     order by s_score desc
                     limit 1) t4
                    on t3.s_id = t4.s_id
```

## 41. 查询不同课程成绩相同的学生的学生编号，课程编号，学生成绩

```sql
# 41. 查询不同课程成绩相同的学生的学生编号，课程编号，学生成绩
select *
from score
where s_score in (select s_score from score group by s_score having count(*) > 1)
```

## 42. 查询每门成绩最好的前三名

```sql
# 42. 查询每门成绩最好的前三名
select c.c_id, c.c_name, s.s_id, s.s_name, s_score
from (
         select *
         from score sc
         where (
                   select count(*)
                   from score sc1
                   where sc.c_id = sc1.c_id
                     and sc.s_score < sc1.s_score
               ) < 3
     ) t1
         inner join student s on t1.s_id = s.s_id
         inner join course c on t1.c_id = c.c_id
order by c.c_id, s_score desc;

```

## 43. 统计每门课程的学生选修人数

```sql
# 43. 统计每门课程的学生选修人数（超过5人的课程才统计）
# 要求输出课程号和选修人数，查询结果按人数降序排列， 若人数相同，按照课程号关联式升序排列。
select c_id, count(*) stu_count
from score
group by c_id
having stu_count > 5
order by stu_count desc, c_id;
```

## 44. 检索至少选修两门课程的学生学号

```sql
# 44. 检索至少选修两门课程的学生学号
select s_id
from score
group by s_id
having count(*) >= 2
;
```

## 45. 查询选修了全部课程的学生信息

```sql
# 45. 查询选修了全部课程的学生信息
select stu.*
from student stu
where s_id in (
    select s_id
    from score
    where c_id in (select c_id from course)
    group by s_id
    having count(*) = (select count(*) from course)
)
```

## 46. 查询个学生的年龄

```sql
# 46. 查询个学生的年龄
# 现在月份小于出生月份 减去1
# 现在月份等于出生月份
# - 判断日 相等 不变
# - 小于 减去1
# 现在月份大于出生月份 不变
# 将减去1的合并
select s_id,
       s_name,
       s_birth,
       case
           when month(now()) < month(s_birth) or (month(now()) = month(s_birth) and day(now()) < day(s_birth))
               then year(now()) - year(s_birth) - 1
           else year(now()) - year(s_birth) end as age
from student;
# 使用if
select s_id,
       s_name,
       s_birth,
       if(
                   month(current_date()) < month(s_birth) or
                   (month(current_date()) = month(s_birth) and day(current_date()) < day(s_birth)),
                   year(current_date()) - year(s_birth) - 1,
                   year(current_date()) - year(s_birth)
           )
           as s_age
from student;

```

## 47. 查询本周过生日的学生

```sql
# 47. 查询本周过生日的学生
select *
from student
where datediff(s_birth, current_date()) between 0 and 7
```

## 48. 查询下周过生日的学生

```sql
# 48. 查询下周过生日的学生
select *
from student
where datediff(s_birth, current_date()) between 7 and 14	
```

## 49. 查询本月过生日的学生

```sql
# 49. 查询本月过生日的学生
select *
from student
where month(s_birth) = month(current_date());
```

## 50. 查询12月过生日的学生

```sql
# 50. 查询12月过生日的学生
select *
from student
where month(s_birth) = 12;
```

