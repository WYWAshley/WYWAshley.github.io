---
layout: post
title: 没有GROUP BY子句的MySQL聚合函数
categories: [SQL]
description: some word here
keywords: MySQL
---

没有GROUP BY子句的MySQL聚合函数

* 问题提出：在 MySQL 中,我观察到尽管没有 GROUP BY 子句，但在 SELECT 列表中使用 AGGREGATE FUNCTION 的语句被执行了。如果这样做，其他RDBMS产品(例如SQL Server)将引发错误。

* 举例，从 tbl1 SELECT col1,col2,sum(col3); 得到执行而没有任何错误,并返回 col1,col2 的第一行值和 col3 的所有值之和。以上查询的结果是一行。

* 原因：这是设计使然-它是MySQL允许的标准的许多扩展之一.

  对于诸如 SELECT 名称之类的查询，MAX(age)FROM t; 参考文档说：

> Without GROUP BY, there is a single group and it is indeterminate
> which name value to choose for the group

​	有关更多信息,请参见关于分组处理的[the documentation](http://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html).

​	设置ONLY_FULL_GROUP_BY可以控制此行为,请参见[5.1.7 Server SQL Modes](http://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by)启用此功能将不允许查询缺少group by语句的聚合函数,并且默认情况下从MySQL版本5.7.5启用该查询.