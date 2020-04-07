---
layout: post
title: MySQL 窗口函数
categories: [SQL]
description: MySQL OLAP
keywords: MySQL
---

MySQL 窗口函数介绍

## 一、窗口函数的必要性

​	窗口函数，又叫 OLAP(Online Analytical Processing) 函数，可对数据库数据进行实时分析处理。功能是可以同时分组和排序，并且**不减少表的行数**（这就是与聚合函数最大的差别），使用场景比较多的是排名 rank，topN 问题。基本语法：

```mysql
<窗口函数> over (partition BY <用于分组的列名> order by <用于排序的列名>);
-- over 关键字用于指定函数的窗口范围，
-- partition by 用于对表分组，
-- order by 子句用于对分组后的结果进行排序。
```

<br>

## 二、专用窗口函数

<img src="/images/posts/MySQL OLAP/1.jpg"/>

### 1. 序号函数

应用场景：希望查询每个用户订单金额最高的前三个订单

* **rank()** 如果有并列名次的行，占用下一个名次的位置。1，1，3...；

* **dense_rank()** 如果有并列名词的行，顺次排列，不占用下一个名次的位置。1，1，2...；

* **row_number()** 顺次排序，不考虑并列名次的问题。1，2，3...。

  <img src="/images/posts/MySQL OLAP/2.jpg"/>

<br>

### 2. 头尾函数

​	应用场景：查询截止到当前订单，按照日期排序第一个订单和最后一个订单的订单金额。

* **first_value()** 获得同组中的第一个列值
* **last_value()**  获得同组中的最后一个列值。这里要注意一下：last_value()默认统计范围是 rows between unbounded preceding and current row，也就是取当前行数据与当前行之前的数据的比较。如果需要显示同组的最后一个值，要加上 **rows between unbounded preceding and unbounded following**。

```mysql
select *, last_value(grade) over(partition by clsno order by grade rows between unbounded preceding and unbounded following) from stu;
```

<br>

### 3. 前后函数

​	Lag() 和 Lead() 函数可以在同一次查询中取出同一字段的前N行的数据(Lag)和后N行的数据(Lead)作为独立的列。

​	应用场景：查询上一个订单距离当前订单的时间间隔。

* **lag(exp_str,offset,defval) over(partition by ..order by …)**
* **lead(exp_str,offset,defval) over(partition by ..order by …)**

​	其中 exp_str 是字段名，offset 是偏移量，defval 默认值可以不定义。

```mysql
select id,order_time
#上一次订单时间
,lag(order_time,1) over(partition by id order by order_time) last_order_time
 
#下一次订单时间
,lead(order_time,1) over(partition by id order by order_time) next_order_time
 
from order_main_table
```

<br>

### 4. 分布函数

应用场景：小于等于当前订单金额的订单比例有多少，或者，小于当前订单金额的订单比例有多少。

* **percent_rank()** 获得该行排名的分位数，如果名次一样，则分位数也一样，占用下一个名词的位置；

* **cume_dist()** 计算某个值在一组有序的数据中累计的分布，和percent_rank()的区别如下图：

  <img src="/images/posts/MySQL OLAP/4.png"/>

  也就是，cume_dist() 不是从0开始的，percent_rank() 是从0开始的；有重复值的时候，cume_dist() 是按照重复值的最后一行计算的，percent_rank() 是按照重复值的第一行计算的。

<br>

### 5. 其他函数

* **nth_value(expr, n)** 返回窗口中第N个expr的值，expr可以是表达式，也可以是列名。

  应用场景：每个用户订单中显示本用户金额排名第二和第三的订单金额。

* **nfile(n)** 将分区中的有序数据分为n个桶，记录桶号。

  应用场景：将每个用户的订单按照订单金额分成3组，或者由于数据量较大，将数据平均分配到 **N 个并行的进程**分别计算，此时就可以用NFILE(N)对数据进行分组，由于记录数不一定被N整除，所以数据不一定完全平均，然后将不同桶号的数据再分配。。

<br>

## 三、聚合函数

​	可以明确、直观地看到**截止到某行数据**，统计数据是多少，同时可以看出每行数据对整体统计数据的影响。用法与专用窗口函数相同，但括号中需要指定聚合的列名。

```mysql
SELECT *,sum(成绩) OVER (ORDER BY 学号) AS current_sum,
avg(成绩) OVER (ORDER BY 学号) AS current_avg,
count(成绩) OVER (ORDER BY 学号) AS current_count,
max(成绩) OVER (ORDER BY 学号) AS current_max,
min(成绩) OVER (ORDER BY 学号) AS current_min FROM 班级表;
```

<img src="/images/posts/MySQL OLAP/3.jpg"/>

<br>

## 四、移动选择窗口行数

多用于公司业绩名单排名中，可以通过移动平均直观地看到与相邻名次业绩的平均、求和等统计数据。

```mysql
CURRENT ROW 边界是当前行，一般和其他范围关键字一起使用

UNBOUNDED PRECEDING 边界是分区中的第一行

UNBOUNDED FOLLOWING 边界是分区中的最后一行

expr PRECEDING  边界是当前行减去expr的值

expr FOLLOWING  边界是当前行加上expr的值

INTERVAL 7 DAY PRECEDING  边界不是基于行而是基于范围的
```

<br>

①计算当前行与前n行（共n+1行）的聚合窗口函数

```text
SELECT *,avg(成绩) OVER (ORDER BY 学号 ROWS n PRECEDING) AS current_avg FROM 班级表; 
```

②计算当前行与之后n行的聚合窗口函数

```text
SELECT *,avg(成绩) OVER (ORDER BY 学号 ROWS n FOLLOWING) AS current_avg FROM 班级表; 
```

③计算当前行与前n1行、后n2行的聚合窗口函数

```text
SELECT *,avg(成绩) OVER (ORDER BY 学号 
ROWS BETWEEN n1 PRECEDING AND n2 FOLLOWING) AS current_avg FROM 班级表; 
```

<br>

## 五、注意事项：

①partition子句可以省略，省略时默认不指定分组（开窗列），但会因此失去窗口函数的功能，所有一般不这样使用；

②因为窗口函数是对 where 和 group by 子句处理后的结果进行操作，所以原则上只能写在 

selec t子句中；

③窗口函数中不能嵌套使用窗口函数和聚合函数；

④专用窗口函数（）为空，聚合窗口函数（）中会写对应聚合列。