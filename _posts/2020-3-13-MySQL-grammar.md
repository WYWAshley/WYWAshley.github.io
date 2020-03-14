## 一、查看表结构

​	表结构就是定义数据表文件名，确定数据表包含哪些字段，各字段的字段名、字段类型、及宽度，并将这些数据输入到计算机当中。

### 1. describe/desc 表名

```mysql
desc employees;
describe employees;
```

### 2. show columns from 表名

```mysql
show columns from employees;
```

上述两个方法结果是一样的，字段解释如下：

* Field: 字段表示的是列名
* Type: 字段表示的是列的数据类型
* Null: 字段表示这个列是否能取空值
* Key: 在mysql中key 和index 是一样的意思，这个Key列可能会看到有如下的值：PRI(主键)、MUL(普通的b-tree索引)、UNI(唯一索引)
* Default: 列的默认值
* Extra :其它信息

![这里写图片描述](https://img-blog.csdn.net/20171228124938336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUzODk0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 3. show create table 表名

```mysql
show create table employees;
```

![这里写图片描述](https://img-blog.csdn.net/20171228125012524?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUzODk0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4.use information_schema; select * from columns where table_name='表名';

​	information_schema 数据库时MySQL自带的，它提供了访问数据库元数据的方式。

```mysql
use information_schema;
select * from columns where table_name='employees';
```

![这里写图片描述](https://img-blog.csdn.net/20171228125019200?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUzODk0MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>

## 二、添加约束

### 1. 建表时添加约束

```mysql
CREATE TABLE goodtest  (
	gid int(10) NOT NULL auto_increment,
    gname varchar(20) NOT NULL,
    sex tinyint(1) DEFAULT 1,
    ...
    PRIMARY KEY (gid),
    UNIQUE KEY (gname),
    FOREIGN KEY (gid) REFERENCES goodtest2(gid)
);
```

### 2. 通过alter语句添加约束

```mysql
# 主键约束
-- 直接修改表添加约束
alter table goodtest add PRIMARY KEY (gid);
-- 通过修改列定义添加或者添加主键，修改约束一般是先删掉原有的后重新添加
alter table goodtest modify gid int(10) PRIMARY KEY;
-- 通过change修改，和modify区别在于当不需要重命名时两个相同的列名是必要的
alter table goodtest change gid gid int(10) PRIMARY KEY;
-- 删除主键约束
alter table goodtest DROP PRIMARY KEY;

# 唯一性约束
-- [constraint gname_uni]是可以省略的
alter table goodtest add [constraint gname_uni] UNIQUE KEY(gname);
alter table goodtest modify gname varchar(20) UNIQUE KEY(gname);
alter table goodtest change gname gname varchar(20) UNIQUE KEY(gname);
-- 删除唯一性约束（类似索引删除方式，可以通过show index from
-- goodtest;查看）
alter table goodtest DROP {INDEX|KEY} key_name;

# 外键约束
alter table gsales add constraint fk_goods_gid FOREIGN KEY(gid) REFERENCES goods(gid);
-- 删除外键约束（建议在创建外键时加上约束的名字）
alter table gsales DROP FOREIGN KEY fk_goods_gid; 

# 默认约束
alter table goodtest alter [column] sex SET DEFAULT 1;
-- 删除默认约束 DROP DEFAULT
alter table goodtest alter [column] sex DROP DEFAULT;
alter table goodtest modify sex tinyint(1) DEFAULT 1;
-- 删除默认约束，不写即可
alter table goodtest modify sex tinyint(1);
alter table goodtest change sex sex tinyint(1) DEFAULT 1;

# 非空约束
alter table goodtest modify gname varchar(20) NOT NULL;
alter table goodtest change gname gname varchar(20) NOT NULL;
-- 删除非空约束 NULL
alter table goodtest modify gname varchar(20) NULL;
alter table goodtest change gname gname varchar(20) NULL;

# 自增约束
-- 一张表只能有一个自增长列，并且该列必须为键(主键、唯一键、外键)
alter table goodtest modify gid int(10) AUTO_INCREMENT;
alter table goodtest change gid gid int(10) AUTO_INCREMENT;
-- 删除
alter table goodtest modify gid int(10);
alter table goodtest change gid gid int(10);
```

<br/>

## 三、添加删除索引

1. 加索引

   ```mysql
   alter table t1 add index 索引名 (字段名1, 字段名2...)
   ```

2. 加主关键字索引

   ```mysql
   alter table t1 add primary key(id);
   ```

3. 加唯一条件索引

   ```mysql
   alter table add unique id_cname(cardname);
   create unique index index_name on table_name (column_list) ;
   ```

4. 删除某个索引

   ```mysql
   alter table t1 drop index id_cname;
   ```

<br/>

## 四、修改列

1. 修改列名

   ```mysql
   -- change方法可以修改当前列名和字段类型，所以修改列名时要保证字段类型保持不变的填写
   alter table t1 change a b int;
   ```

2. 修改列类型

   ```mysql
   -- modify方法可以改变列的类型，此时不需要重命名
   alter table t1 modify b bigint not null;
   -- change方法可以修改列类型
   alter table t1 change b b bigint not null; 
   ```

3. 增加新列

   ```mysql
   alter table t1 add d timestamp;
   ```

4. 删除列

   ```mysql
   alter table t1 drop column d;
   ```

5. 重命名表

   ```mysql
   alter table t1 rename t2;
   ```

<br/>

## ALTER Table

```sql
alter_specification:
    table_options
  | ADD [COLUMN] col_name column_definition
        [FIRST | AFTER col_name]
  | ADD [COLUMN] (col_name column_definition,...)
  | ADD {INDEX|KEY} [index_name]
        [index_type] (key_part,...) [index_option] ...
  | ADD {FULLTEXT|SPATIAL} [INDEX|KEY] [index_name]
        (key_part,...) [index_option] ...
  | ADD [CONSTRAINT [symbol]] PRIMARY KEY
        [index_type] (key_part,...)
        [index_option] ...
  | ADD [CONSTRAINT [symbol]] UNIQUE [INDEX|KEY]
        [index_name] [index_type] (key_part,...)
        [index_option] ...
  | ADD [CONSTRAINT [symbol]] FOREIGN KEY
        [index_name] (col_name,...)
        reference_definition
  | ADD CHECK (expr)
  | ALGORITHM [=] {DEFAULT|INPLACE|COPY}
  | ALTER [COLUMN] col_name {SET DEFAULT literal | DROP DEFAULT}
  | CHANGE [COLUMN] old_col_name new_col_name column_definition
        [FIRST|AFTER col_name]
  | [DEFAULT] CHARACTER SET [=] charset_name [COLLATE [=] collation_name]
  | CONVERT TO CHARACTER SET charset_name [COLLATE collation_name]
  | {DISABLE|ENABLE} KEYS
  | {DISCARD|IMPORT} TABLESPACE
  | DROP [COLUMN] col_name
  | DROP {INDEX|KEY} index_name
  | DROP PRIMARY KEY
  | DROP FOREIGN KEY fk_symbol
  | FORCE
  | LOCK [=] {DEFAULT|NONE|SHARED|EXCLUSIVE}
  | MODIFY [COLUMN] col_name column_definition
        [FIRST | AFTER col_name]
  | ORDER BY col_name [, col_name] ...
  | RENAME [TO|AS] new_tbl_name
```



