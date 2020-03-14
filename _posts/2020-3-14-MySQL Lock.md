---
layout: post
title: MySQL Lock
categories: [SQL, Database]
description: 关于MySQL两种存储引擎MyIsam和InnoDB的数据库锁介绍
keywords: MyIsam, InnoDB, Lock
---

锁是计算机协调多个进程或线程并发访问某一资源的机制

在数据库中，除传统的计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。

## 表锁（以MyIsam为例）

### 1. 加读锁

* ```show open tables;```   查看表上加过的锁

* ```lock table 表名 read(write), 表名2 read(write);```   手动增加锁

* ```unlock tables;```   释放表

  | session 1                | session 2                  |
  | ------------------------ | -------------------------- |
  | 给表加读锁               |                            |
  | 可以读取该表             | 也可以读取该表             |
  | 不可以查询其他未锁定的表 | 也可以查询或更新其他表     |
  | 不可以增删改该表         | 增删改该表会一直等待获得锁 |
  | 释放表锁                 | 获得锁，增删改成功         |

### 2. 加写锁

​		写锁会影响读锁和写锁，读锁不会影响读锁，会影响写锁

### 3. 分析表锁定

​		可以通过检查 table_locks_waited 和 table_locks_immediate 状态变量来分析系统上的表锁定 ```show status like 'table%';```  其中 Table_locks_immediate 表示可以立即获取表级锁定的次数，即每次获取次数加一，==Table_locks_waited== 出现表级锁定争用而发生等待的次数，即此值高代表存在着严重的表级锁争用情况。

### 4. 总结

​		此外，Myisam的读写锁调度时写优先，这也是 **myisam 不适合做写为主的引擎**，因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远堵塞。可以优化，将大query拆分成小query，然后建立高效的索引加快检索。

<br/>

## 行锁（以InnoDB为例）

innoDB和Myisam 区别在于支持事务，采用行级锁

* 在不可重复读情况下，session 可以读到自己已修改但未提交的数据，**读己之所写**

* session1在修改的时候session2是不能修改同一行的，读取也是不可以的会一直等待，所以可以避免不可重复读的问题，只有session1提交之后session2才能修改这一行

* 但是不是同一行的可以同时修改，两边提交了的话都是会修改的

* **没有使用索引，行锁变表锁**

* 当我们用范围条件而不是想等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项（**该索引和主键聚集索引都要加**）加锁（record-key），对于键值在条件范围内但并不存在的记录叫做间隙**（gap-key）这是针对非唯一索引或者唯一索引的范围查询的**，合起来叫做next-key。间隙锁会是某些不存在的键值被无辜的锁定，而造成在锁定的时候无法插入锁定简直范围内的任何数据，在某些场景下会造成很大的危害。

* ==当前读== ```select * from a where a.b=400 for update```  在扫描到的任何索引记录上加**排它的next-key lock**。如果事务对数据加上排他锁之后，则其他事务不能对该数据加任何的锁。获取排他锁的事务既能读取数据，也能修改数据。

  ==当前读==```select * from a where a.b=400 lock in share mode;```  在扫描到的任何索引记录上加共享的**（shared）next-key lock**，其他事务可以读取数据，但不能对该数据进行修改，直到所有的共享锁被释放。如果事务对某行数据加上共享锁之后，可进行读操作；其他事务可以对该数据加共享锁，但不能加排他锁，且只能读数据，不能修改数据。

  ==快照读==```select ...from...```  查询数据不加任何类型的锁，**所以并不是加了排他锁这行数据就不能被读取了。**

  ```update..where  delete from..where```  在扫描到的任何索引记录上加next-key lock

  ```insert into..```  简单的insert会在insert的行对应的索引记录上加一个排它锁，这是一个record lock，并没有gap，所以并不会阻塞其他session在gap间隙里插入记录。假设发生了一个唯一键冲突错误，那么将会在重复的索引记录上加读锁。当有多个session同时插入相同的行记录时，如果另外一个session已经获得该行的排它锁，那么将会导致死锁。这是改用```INSERT ... ON DUPLICATE KEY UPDATE ```这种 sql 和 insert 加锁的不同的是，如果检测到键冲突，它直接申请加排它锁，而不是共享锁，不会发生上述死锁。

* ==意向锁==的主要作用是处理行锁和表锁之间的矛盾，能够显示“某个事务正在某一行上持有了锁，或者准备去持有锁”，**为了实现多粒度锁机制**（白话：为了表锁和行锁都能用）。因为当表中的几行被锁定了，另一个命令想要获得表锁时需要遍历整表确定是否有行级锁，那么在行级锁申请之前先申请一个表的意向锁就可以了。

* ```show status like 'innodb_row_lock%'``` 中的 ==InnoDB_row_lock_waits== 代表行锁冲突次数， ==Innodb_row_lock_current_waits== 正在等待锁定的数量。可以用 ```show profile;```查看为什么有那么多锁

![image-20200313215132343](C:\Users\yuwen\AppData\Roaming\Typora\typora-user-images\image-20200313215132343.png)