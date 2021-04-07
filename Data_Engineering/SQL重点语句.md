# SQL比较复杂的语句
![image](https://user-images.githubusercontent.com/65893273/113826151-6a676300-97b4-11eb-88b1-c192290dbf8f.png)  
以题代练，图来自知乎猴子数据分析  

## 练习1  
%表示任意字符串    
count计数  
avg求平均  
sum求和  

查询姓孟的老师个数：  
select count(教姓名)    
from teacher    
where 教师姓名 like '孟%';    

查询名字里带孟的老师个数：  
select count(教姓名)    
from teacher    
where 教师姓名 like '%孟%';    

查询名字里最后一个字为孟的老师个数：    
select count(教姓名)      
from teacher      
where 教师姓名 like '%孟';  

查询课程编号为0002的平均成绩：  
select avg(成绩) 
from 成绩表
where 课程号=‘0002’；

查询选了课程的学生人数：    
select count（distinct（学号）） as 学生人数  
from 成绩表；  

## 练习2 分组

sql执行顺序:     
1 from   
2 join   
3 on   
4 where   
5 group by(开始使用select中的别名，后面的语句中都可以使用)  
6 avg,sum....   
7 having   
8 select   
9 distinct   
10 order by  
11 limit   

注意点：一般，WHERE在前，GROUP BY在后，即先进行筛选，然后进行分组；  
HAVING只能跟在GROUP BY之后，对分组后的聚合结果进行筛选；HAVING的前提是分组；WHERE在最前，最先对原始数据进行一遍筛选；  
WHERE的条件里只能用已有的列进行条件判断，不允许使用聚合函数。HAVING之后可以允许使用聚合函数；  
聚合函数包括count(),sum(),avg(),max(),min()  

查询各科成绩最高分和最低分：  
select 课程号，max（成绩），min（成绩）  
from 成绩表  
group by 课程号；  

查询上每门课的学生人数：  
select count（distinct（学号））  
from 成绩表  
group by 课程号；  
如果数据录入正确，distinct没必要加。    
 
查询男生、女生人数：    
select count（学号）  
from 学生表  
group by 性别；  

## 练习3 分组结果
查询平均成绩>60的学生学号和平均成绩:  
select 学号，avg（成绩）  
from 成绩表  
group by 学号  
having avg（成绩）>60;  

至少选修两门课的学生学号
select 学号  
from 成绩表  
group by 学号  
having count(课程号)>=2;  

查询同姓名的学生名单及人数:  
select 姓名,count(姓名)
from 学生
group by 姓名
having count(姓名)>1;

查询不及格的课程并按课程号从大到小排列:    
select 课程号    
from 成绩表    
where 成绩<60    
order by 课程号 desc;    
升序指定为asc, 降序指定为 desc    

查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列:    
select avg(成绩) as 平均成绩,课程号  
from 成绩表  
group by 课程号  
order by avg(成绩) asc, 课程号 desc;  

检索课程编号为“0004”且分数小于60的学生学号，结果按按分数降序排列:    
select 学号,分数    
from 成绩表    
where 课程号 = '0004' && 成绩<60    
order by 分数 desc;  

统计每门课程的学生选修人数(超过2人的课程才统计)  
要求输出课程号和选修人数，查询结果按人数降序排序，若人数相同，按课程号升序排序  
select 课程号,count(学号) as 选修人数  
from 成绩表  
group by 课程号  
having count(学号)>2  
order by count(学号) desc, 课程号 asc;  

查询两门以上不及格课程的同学的学号及其平均成绩:    
select 学号,avg(成绩) as 平均成绩  
from 成绩表  
group by 学号  
having 学号 in (select 学号   
from 成绩表  
where 成绩<60  
group by 学号  
having count(课程号)>2);  

查询学生的总成绩并进行排名:    
select 学号,sum(成绩)  
from 成绩表  
group by 学号  
order by sum(成绩) desc;  

查询平均成绩大于60分的学生的学号和平均成绩:  
select 学号,avg(成绩) as 平均成绩  
from 成绩表  
group by 学号  
having avg(成绩)>60;  

## 练习4 嵌套查询
查询所有课程成绩小于60分学生的学号、姓名:  
select 学号,姓名    
from student    
where 学号 in (select 学号   
from score   
group by 学号   
having max(成绩)<60);  

## 练习5 exist和not exist  
EXISTS 运算符    
EXISTS 运算符用于判断查询子句是否有记录，如果有一条或多条记录存在返回 True，否则返回 False。    

SQL EXISTS 语法    
SELECT column_name(s)    
FROM table_name    
WHERE EXISTS    
(SELECT column_name FROM table_name WHERE condition);    

![image](https://user-images.githubusercontent.com/65893273/113837917-df409a00-97c0-11eb-9332-83aae6efaf52.png)    
符合条件的是google,Facebook和菜鸟,不符合的是淘宝和微博.      
使用exists 输出google,Facebook和菜鸟;     
使用not exists 输出淘宝和微博.     

## 练习6 TOP N 问题 关联子查询
按课程号分组取成绩最大值所在行的数据:  
我们可以使用分组（group by）和汇总函数得到每个组里的一个值（最大值，最小值，平均值等）。但是无法得到成绩最大值所在行的数据。  
select 课程号,max(成绩) as 最大成绩  
from score   
group by 课程号; 

此时用关联子查询实现:  
select *
from 成绩表 as a
where 成绩 = (select max(成绩)
from 成绩表 as b  
where b.课程号 = a.课程号  
group by 课程号);  

where b.课程号 = a.课程号 这句话指明是关联子查询.  
运行逻辑是,先从主查询里,取一个课程号N,进入到子查询,子查询返回该课程的最大成绩.   
指明课程号N后,子查询里只有一个课程号了,group by 多余.  
主查询此时where 变成 where 成绩 = 子查询返回的最大成绩 AND 课程号=课程号N.  

## 练习7 联结
![image](https://user-images.githubusercontent.com/65893273/113845408-11a1c580-97c8-11eb-8acb-371151f950c8.png)  
inner join 只返回匹配的  
left join 返回左表所有的和右表匹配的值,左表有右表无的用NULL填充  
right join类似  
full outer join 则所有不匹配的值用NULL填充  

来自runoob:在使用join时,数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回给用户。  
在使用 left join 时，on 和 where 条件的区别如下：  
1、on 条件是在生成临时表时使用的条件，它不管 on 中的条件是否为真，都会返回左边表中的记录。  
2、where 条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有 left join 的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。    
假设有两张表：  
表1：tab1  
id size  
1  10  
2  20  
3  30  
表2：tab2    
size name  
10   AAA  
20   BBB  
20   CCC  
两条SQL:  
1、select * from tab1 left join tab2 on tab1.size = tab2.size where tab2.name='AAA'  
2、select * from tab1 left join tab2 on tab1.size = tab2.size and tab2.name='AAA'  
第一条SQL的过程：  
1、中间表    
on 条件:    
tab1.size = tab2.size  
结果:   
tab1.id |tab1.size |tab2.size |tab2.name  
1 |10 |10 |AAA  
2 |20 |20 |BBB  
2 |20 |20 |CCC  
3 |30 |(null) |(null)
2、再对中间表过滤    
where 条件：    
tab2.name='AAA'   
结果:  
tab1.id tab1.size tab2.size tab2.name      
1 10 10 AAA  

第二条SQL的过程：  
1、中间表  
on 条件:    
tab1.size = tab2.size and tab2.name='AAA'    
(条件不为真也会返回左表中的记录)   
tab1.id tab1.size tab2.size tab2.name  
1 10 10 AAA  
2 20 (null) (null)  
3 30 (null) (null)  

其实以上结果的关键原因就是 left join,right join,full join 的特殊性。  
不管 on 上的条件是否为真都会返回 left 或 right 表中的记录，full 则具有 left 和 right 的特性的并集。  
而 inner jion 没这个特殊性，则条件放在 on 中和 where 中，返回的结果集是相同的。  

查询所有学生的学号、姓名、选课数、总成绩:  
select a.学号, a.姓名,count(b.课程号) as 选课数, sum(b.成绩) as 总成绩  
from 学生表 as a  
left join 成绩表 as b   
on a.学号 = b.学号  
group by a.学号;  

查询学生的选课情况：学号，姓名，课程号，课程名称:     
数据分布在三张表中    
select a.学号, a.姓名, c.课程号, c.课程名称    
from 学生表 as a inner join 成绩表 as b on a.学号 = b.学号  
inner join 课程 on b.课程号=c.课程号;    

## 练习8 行列互换
基于case语句:  
SQL CASE 表达式是一种通用的条件表达式，类似于其它语言中的 if/else 语句。   

CASE WHEN condition THEN result   
[WHEN ...]   
[ELSE result]   
END   

CASE 子句可以用于任何表达式可以有效存在的地方。   
condition 是一个返回boolean 的表达式。   
如果结果为真，那么 CASE 表达式的结果就是符合条件的 result。   
如果结果为假，那么以相同方式搜寻任何随后的 WHEN 子句。   
如果没有 WHEN condition 为真，那么 case 表达式的结果就是在 ELSE 子句里的值。   
如果省略了 ELSE 子句而且没有匹配的条件， 结果为 NULL。  

![image](https://user-images.githubusercontent.com/65893273/113859495-8af4e480-97d7-11eb-9b79-098622dea23b.png)  
select date,sum(case when result = '胜' then 1 else 0) as win, sum(case when result = '负' then 1 else 0) as lost  
from database  
group by date;  

![image](https://user-images.githubusercontent.com/65893273/113860054-2ede9000-97d8-11eb-8bfe-067b89b8e93e.png)    
select   
    (case when 语文>=80 then '优秀'    
          when 语文>=60 then '及格'   
          else '不及格' end) as 语文,   
    (case when 数学>=80 then '优秀'   
          when 数学>=60 then '及格'   
          else '不及格' end) as 数学,   
    (case when 英语>=80 then '优秀'   
          when 英语>=60 then '及格'   
          else '不及格' end) as 英语,   
from table   

![image](https://user-images.githubusercontent.com/65893273/113860284-706f3b00-97d8-11eb-88d3-5c1b1bb50ae3.png)  
![image](https://user-images.githubusercontent.com/65893273/113860318-782edf80-97d8-11eb-837e-97aa820d4dd5.png)  
![image](https://user-images.githubusercontent.com/65893273/113860353-81b84780-97d8-11eb-9e8d-4fc06ff683cc.png)  
![image](https://user-images.githubusercontent.com/65893273/113860484-add3c880-97d8-11eb-8dfb-3dba21dd7929.png)  
![image](https://user-images.githubusercontent.com/65893273/113860573-c217c580-97d8-11eb-866a-5023d2a1d12e.png)  
此题中,用sum取代max也可.  

case具有两种格式。简单case函数和case搜索函数:  
--简单case函数  
case sex  
  when '1' then '男'  
  when '2' then '女’  
  else '其他' end  
  
--case搜索函数  
case when sex = '1' then '男'  
     when sex = '2' then '女'  
     else '其他' end    



