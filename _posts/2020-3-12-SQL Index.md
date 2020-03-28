---
layout: post
title: MySQL查询优化
categories: [SQL]
description: SQL查询优化，包括explain，日志，索引的使用
keywords: SQL Index
---

包括SQL查询优化的各种内容

MySQL 中利用索引、日志查询、explain等工具进行优化查询

### MySQL性能下降的原因

* 查询语句不行，没建立索引
* 索引失效，建立了没用上（单值索引、复合索引）
* 关联查询太多join
* 服务器调优及各个参数设置

<br/>

### 优化过程

1. 观察，至少跑一天，看看生产的慢SQL情况
2. 开启慢查询日志，设置阈值，比如超过5秒的就是慢SQL，并将其抓取出来
3. explain+慢SQL分析
4. show profile
5. 运维经理或者DBA进行SQL数据库服务器的参数调优

<br/>

### SQL执行加载顺序

1. FROM\<left table\> 
2. ON\<join_condition\>
3. \<join_type\> join \<right_table\>
4. WHERE\<where_condition\>
5. GROUP BY\<group_by_list\>
6. having
7. SELECT
8. DISTINCT\<distinct_list\>
9. order by\<order_by_list\>
10. LIMIT\<limit_number\>

<br/>

### SQL七种 join

* 左连接：select * from A left join B on A.key=B.key 保留A的全部key
* 左半连接：select * from A left join B on A.key=B.key where B.key is null 保留A的除了与B相同的key，也就是B的key为null
* 右连接：select * from A right join B on A.key=B.key
* 右半连接：select * from A right join B on A.key=B.key where A.key is null
* 内连接：select * from A inner join B on A.key=B.key 只保留A和B共有的值
* 全外连接：select * from A full outer join B on A.key=B.key A和B全部拥有的值，当然因为 mysql 不知道 full join，所以我们要用 left join right join union 配合使用：select * from (select * from A left join B on A.key=B.key union select * from A right join B on A.key=B.key) c
* 外连接减去内连接：select * from A full outer join B on A.key=B.key where A.key is null or B.key is null A和B各自的独有 select * from (select * from A left join B on A.key=B.key  where B.key is null union select * from A right join B on A.key=B.key where B.key is null)

<br/>

## 索引

### 1. 索引简介

定义：帮助SQL高效获取数据的数据结构。提高查询效率，类似于字典。影响到where查询和order by排序部分。除了数据本身，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据。索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。

优势：提高数据检索的效率，降低了数据库的IO成本；通过索引对数据进行排序，降低数据排序的成本，降低了CPU的消耗。

劣势：索引包含了主键和索引字段，并指向实体表的记录，所以索引也要占空间的；索引会降低更新表的速度，因为不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段；索引需要根据业务不断优化。

分类：单值索引、唯一索引、复合索引

```mysql
create [unique] index idx_name on mytable(columnname(length));

alter table add [unique|primary key|fulltext] index idx_name on (columnname(length));

drop index idx_name on table;

show index from table;
```

mysql索引结构：

* BTree索引，不删除节点，而是更新失效
* Hash索引
* Full-Text索引：查看[链接](https://blog.csdn.net/mrzhouxiaofei/article/details/79940958)
* R-Tree索引

哪些情况需要创建索引：

&emsp;&emsp;主键自动建立唯一索引，频繁作为查询条件的字段应该创建索引，查询中与其他表关联的字段外键关系建立索引，频繁更新的字段不适合创建索引，where条件用不到的字段不创建索引，高并发条件下倾向创建组合索引，排序中的查询字段（顺序），查询中统计或者分组字段。

哪些情况不建议建立索引：

&emsp;&emsp;表内记录太少，经常增删改的表，数据重复且分布平均的表字段。

<br/>

### 2. 性能分析

* MySQL Query Optimizer： 

  &emsp;&emsp;有专门负责优化 SELETC 语句的优化器模块，主要功能是通过计算分析系统中收集到的统计信息，为客户端请求的 Query 提供他认为最优的执行计划，但不一定是DBA认为的最优。当客户端向MySQL请求一条 query，命令解释器模块完成请求分类，区别是 select 并转发给 mysql query optimizer 时，mysql query optimizer 首先会对整条 query 进行优化，处理掉一些常量表达式的预算，直接换算成常量值，并对 query 中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件、结构调整等，然后分析 query 中的 hint 信息（如果有），看显示 hint 信息是否可以完全确定该 query 的执行计划，如无或者不可以，则会读取所涉及对象的统计信息，根据 query 进行写相应的计算分析，然后再得出最后的执行计划。

* MySQL 常见瓶颈：CPU饱和，I/O，服务器硬件

* Explain：

  使用 explain 关键字可以模拟优化器执行 SQL 查询语句，从而知道是如何处理你的SQL语句的，分析你的查询语句或是表结构的性能瓶颈。explain + SQL语句

  执行计划包含的信息字段解释：

  * id：select 查询的序列号，包含一组数字，表示查询中执行 select 字句或操作。相同的id意味着执行顺序由上至下；id 不同，id 值越大的分组越先被执行（如子查询，从内到外），组内顺序执行。   ==》**表的读取顺序**
  * select_type：①SIMPLE  不包含子查询或者 union 查询 ②PRIMARY 查询中若包含任意复杂的部分，最外层被标记为主查询  ③SUBQUERY 子查询  ④DERIVED 临时表\<derived+id\>  ⑤UNION    ⑥UNION RESULT 两种 union 结果的合并。    ==》**数据读取操作的操作类型**
  * table：显示这一行的数据是关于哪张表的
  * type：访问类型，从最好到最差依次是：system 系统变量 > **const** 用于匹配主键或唯一索引，在 where 中会变成常量 > **eq_ref** 唯一性索引扫描，对于每个索引键表中有且仅有一个记录匹配 > **ref** 非唯一性索引扫描，返回匹配某个单独值的所有行，本质也是索引匹配，只是有多行 > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > **range** 使用索引划定了一个范围，使用between、and、><、in （range类型查询字段之后的索引会失效）\> **index** 全盘扫描索引 > all.       range级别就好了。
  * possible_keys：显示可能应用到这张表的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被实际使用。  ==》**那些索引可以被使用**
  * keys：实际使用的索引，如果为NULL，则没有使用索引。查询中若使用了覆盖索引，则该索引仅出现在key列表中。  ==》**那些索引被实际使用了**
  * key_len：表示索引中使用的字节数，可通过该列计算查询中使用的索引长度，在不损失精度的情况下，越短越好（如组合索引就是两列字段）。key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得到的。
  * ref：显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值。  ==》**表之间的引用**
  * rows：根据表统计信息及索引选用情况，大致估算出找到所需的记录需要读取的行数。==》**每张表有多少行被优化器优化查询**
  * extra：①Using filesort 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，mysql中无法使用索引完成的排序操作称为“文件排序”，如果可以尽快优化，不能让他另起炉灶；②Using temporary 新建了内部的临时表保存中间结果，然后折腾，然后删除，特别注意group by，按照index顺序来；③Using index i表示相应的select操作中使用了覆盖索引，避免访问了表的数据行，效率不错，ii如果同时出现using where，表明索引被用来执行索引键值的查找，iii如果没有同时出现using where，表明索引用来读取数据而非执行查找动作 ==》注意如果要使用覆盖索引一定要注意select列表中只取出需要的列不可select* ④Using where 使用where过滤 ⑤Using join buffer 使用了连接缓存 ⑥Impossible where where子句的值总是false，不能用于过滤 ⑦select tables optimized away ⑧distinct

<br/>

### 3. 索引优化

* join 语句的优化

  尽可能减少 join 语句中的 NestedLoop 的循环总次数，把大表放在前面，因为后面的表会进行全表扫描，所以左连接在右表加 index，右连接在左表加index，注意不要和in、exists混乱了；

  优先优化 NestedLoop 的内层循环；

  保证 Join 语句中被驱动表上 Join 条件字段已经被索引；

  当无法保证被驱动表的 Join 条件字段被索引且内存资源充足的条件下，不要太吝啬 joinbuffer；

* 索引失效的原因

  1. 全值匹配我最爱：索引的所有列都用上，列的顺序无所谓，因为 mysql 会自动优化 sql 语句
  2. 最佳左前缀法则：如果索引了多列，查询从索引的最做前列（否则不能使用索引）开始并且不跳过索引中的列（否则就只使用了一部分）
  3. 不要在索引列上做任何操作（计算、函数、自动或手动类型转换，会导致索引失效而转向全表扫描）
  4. 存储引擎不能使用索引中范围条件（between，<，>，in）右边的列，访问类型前面的用到ref，中间的范围是range，后面的是all。
  5. 尽量使用**覆盖索引**，使得索引列和查询列一致或范围内，减少使用 select *，在 extra 内容中会有 Using where 和 Using index 即直接从 index 中获取数据，这个时候就算没有满足第2点或者第4点，也是很快的因为不是全表扫描。另外注意 select 中添加本身就已经有索引但是不在覆盖索引中的列也是可以的，比如主键索引。注意 like 开头不是%的后面的索引依旧可以用没有失效，但是开头%，like字段开始索引就失效了
  6. 使用 不等于 != <> 会导致全表扫描，但该写的时候还是得写
  7. is null， is not null 也会无法使用索引
  8. like 以**通配符开头**如 '%abc'，mysql索引失效会变成全表扫描，但是‘abc%’不会。但一定要开头怎么办？用覆盖索引解决，可以使得访问类型 type 变成 index
  9. 字符串如**varchar不加单引号索引失效**，因为会自动进行类型转换，索引失效
  10. 少用 or，也会索引失效

* 分组

  1. group by 实质是先排序后进行分组，遵照索引键的最佳做前缀原则，不能用index可能会有临时表产生
  2. 无法使用索引列的时候增大max_length_for_sort_data 和 sort_buffer_size
  3. where 高于 having，能在where中写的不要用having

* 排序

  1. 排序和查找使用相同的索引
  2. 排序顺序要按照索引列中的顺序来，除非之前在 where 语句中已经让列值等于常量了 where a=const order by b, c     where a=const and b>const order by b, c
  3. 索引列用于排序后，之后的索引列不用于查找了，相当于是范围，大哥被范围了，后面就群龙无首了。
  4. order by语句使用索引最左前列，索引最左前列原则
  5. 使用where子句与order by 子句条件列组合满足索引最左前列原则
  6. 尽可能在索引列上完成排序，但如果一定不在索引列，如一个asc一个desc（要么同升要么同降），要用到filesort，mysql就要启动双路排序和单路排序。①双路是两次扫描磁盘，读取行指针和orderby列在buffer中进行排序，然后再按照列中的值重新从列表中读取其他字段；②一次读取所有需要的列，在buffer排序，少了IO但是空间需要更大所以可能要抓好几次多次排序合并。可以增大 sort_buffer_size 和 max_length_for_sort_data，一定不要用select *！！！！太占用空间了

* 永远小表驱动大表

   1. 因为数据库连接次数减少

   2. 小数据集驱动大数据集：in or exits ???

      ```mysql
      select * from A where id in (select id from B)
      等价于：
      for select id from B
      for select * from A where A.id=B.id
      
      select * from A where exists(select 1 from B where B.id=A.id)
      等价于：
      for select * from A
      for select * from B where B.id=A.id
      
      ***EXISTS用于检查子查询是否至少会返回一行数据，将著查询的数据放到子查询中做条件验证，根据验证结果（True or False）来决定主查询的结果是否得以保留。所以exists后面select 什么都可以，数据不是重点，重点是where筛选，甚至是null
      当A表大于B表时，用in，当B表大于A表时用exists
      ```

<br/>

## 慢查询日志

* MYSQL 的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阈值的语句，具体指运行时间超过long_query_time的SQL，则会被记录到慢查询日志中。long_query_time默认为10。```show variables like '%long_query_log%';```      开启```set global slow_query_log=1;```    查询当前系统中有多少慢查询记录```show variables like '%slow_queries%';```
* 日志分析工具 mysqldumpslow
* 另外```show processlist;```   可以查看进程，关闭死锁 ```kill id;```    
* ```show profiles;```可以查看每条sql语句执行时间    
* ```show profile cpu, block io for query 3;```可以查看某一条 sql 生命周期的详细信息 converting HEAP to MyISAM查询结果太大，内存都不够用了往磁盘上搬，Creating tmp table创建临时表，Copying to tmp table on disk把内存中临时表复制到磁盘，危险！ Locked 锁住了。
* ```select * from mysql.general_log;```可以查看全局日志

<br/>