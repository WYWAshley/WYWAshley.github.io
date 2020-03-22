---
layout: post
title: PySpark Cheatsheet
categories: [PySpark, cheatsheet]
description: PySpark Cheatsheet
keywords: PySpark, cheatsheet，RDD，SparkSQL
---

讲述了 RDD 与 DataFrame 的区别，并附上了速查表

## 一、PySpark-RDD-Basic

<img src="/images/posts/PySpark/PySpark-RDD.jpg" alt="PySpark-RDD" />

<br/>

## 二、PySpark-SQL

<img src="/images/posts/PySpark/PySpark-SQL.jpg" alt="PySpark-RDD" />

<br/>

## 三、Spark SQL概述

### 3.1． Spark SQL的前世今生

&emsp;&emsp;Shark是一个为Spark设计的大规模数据仓库系统，它与Hive兼容。Shark建立在Hive的代码基础上，并通过将Hive的部分物理执行计划交换出来。这个方法使得Shark的用户可以加速Hive的查询，但是Shark继承了Hive的大且复杂的代码使得Shark很难优化和维护，同时Shark依赖于Spark的版本。随着我们遇到了性能优化的上限，以及集成SQL的一些复杂的分析功能，我们发现Hive的MapReduce设计的框架限制了Shark的发展。在2014年7月1日的Spark Summit上，Databricks宣布终止对Shark的开发，将重点放到Spark SQL上。

### 3.2． 什么是Spark SQL

&emsp;&emsp;Spark SQL是Spark用来处理结构化数据的一个模块，它提供了一个编程抽象叫做DataFrame并且作为分布式SQL查询引擎的作用。

&emsp;&emsp;相比于Spark RDD API，Spark SQL包含了对结构化数据和在其上运算的更多信息，Spark SQL使用这些信息进行了额外的优化，使对结构化数据的操作更加高效和方便。

&emsp;&emsp;有多种方式去使用Spark SQL，包括SQL、DataFrames API和Datasets API。但无论是哪种API或者是编程语言，它们都是基于同样的执行引擎，因此你可以在不同的API之间随意切换，它们各有各的特点，看你喜欢那种风格。

### 3.3． 为什么要学习Spark SQL

&emsp;&emsp;我们已经学习了Hive，它是将Hive SQL转换成MapReduce然后提交到集群中去执行，大大简化了编写MapReduce程序的复杂性，由于MapReduce这种计算模型执行效率比较慢，所以Spark SQL应运而生，它是将Spark SQL转换成RDD，然后提交到集群中去运行，执行效率非常快！

**1.易整合** 

将sql查询与spark程序无缝混合，可以使用java、scala、python、R等语言的API操作。

**2.统一的数据访问** 

以相同的方式连接到任何数据源。

**3.兼容Hive** 

支持hiveSQL的语法。

**4.标准的数据连接** 

可以使用行业标准的JDBC或ODBC连接。

 <br/>

## 四、DataFrame

### 4.1． 什么是DataFrame

DataFrame的前身是SchemaRDD，从Spark 1.3.0开始SchemaRDD更名为DataFrame。与SchemaRDD的主要区别是：DataFrame不再直接继承自RDD，而是自己实现了RDD的绝大多数功能。你仍旧可以在DataFrame上调用rdd方法将其转换为一个RDD。

在Spark中，DataFrame是一种以RDD为基础的分布式数据集，类似于传统数据库的二维表格，DataFrame带有Schema元信息，即DataFrame所表示的二维表数据集的每一列都带有名称和类型，但底层做了更多的优化。DataFrame可以从很多数据源构建，比如：已经存在的RDD、结构化文件、外部数据库、Hive表。

<br/>

### 4.2． DataFrame与RDD的区别

&emsp;&emsp;RDD可看作是分布式的对象的集合，Spark并不知道对象的详细模式信息，DataFrame可看作是分布式的Row对象的集合，其提供了由列组成的详细模式信息（就是列的名称和类型），使得Spark SQL可以进行某些形式的执行优化。DataFrame和普通的RDD的逻辑框架区别如下所示：

<img src="/images/posts/PySpark/DataFrame.png" alt="RDD vs DataFrame"/>

&emsp;&emsp;上图直观地体现了DataFrame和RDD的区别。

&emsp;&emsp;左侧的RDD[Person]虽然以Person为类型参数，但Spark框架本身不了解 Person类的内部结构。

&emsp;&emsp;而右侧的DataFrame却提供了详细的结构信息，使得Spark SQL可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么，DataFrame多了数据的结构信息，即schema。这样看起来就像一张表了，DataFrame还配套了新的操作数据的方法，DataFrame API（如df.select())和SQL(select id, name from xx_table where ...)。

&emsp;&emsp;此外DataFrame还引入了off-heap,意味着JVM堆以外的内存, 这些内存直接受操作系统管理（而不是JVM）。Spark能够以二进制的形式序列化数据(不包括结构)到off-heap中, 当要操作数据时, 就直接操作off-heap内存. 由于Spark理解schema, 所以知道该如何操作。

&emsp;&emsp;RDD是分布式的Java对象的集合。DataFrame是分布式的Row对象的集合。DataFrame除了提供了比RDD更丰富的算子以外，更重要的特点是提升执行效率、减少数据读取以及执行计划的优化。

&emsp;&emsp;有了DataFrame这个高一层的抽象后，我们处理数据更加简单了，甚至可以用SQL来处理数据了，对开发者来说，易用性有了很大的提升。

&emsp;&emsp;不仅如此，通过DataFrame API或SQL处理数据，会自动经过Spark 优化器（Catalyst）的优化，即使你写的程序或SQL不高效，也可以运行的很快。

<br/>

### 4.3． DataFrame与RDD的优缺点

#### **4.3.1 RDD的优缺点：**

**优点:**

（1）编译时类型安全 
    编译时就能检查出类型错误

（2）面向对象的编程风格 
    直接通过对象调用方法的形式来操作数据

**缺点:**

（1）序列化和反序列化的性能开销 
    无论是集群间的通信, 还是IO操作都需要对对象的结构和数据进行序列化和反序列化。

（2）GC的性能开销 
    频繁的创建和销毁对象, 势必会增加GC

<br/>

#### **4.3.2 DataFrame**的优缺点：

**优点：**

&emsp;&emsp;DataFrame通过引入schema和off-heap（不在堆里面的内存，指的是除了不在堆的内存，使用操作系统上的内存），解决了RDD的缺点, Spark通过schame就能够读懂数据, 因此在通信和IO时就只需要序列化和反序列化数据, 而结构的部分就可以省略了；

**缺点：**

&emsp;&emsp;通过off-heap引入，可以快速的操作数据，避免大量的GC。但是却丢了RDD的优点，DataFrame不是类型安全的, API也不是面向对象风格的。



具体详情可以参考该[博客](https://www.cnblogs.com/Transkai/p/11360603.html)。