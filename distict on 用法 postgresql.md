- SELECT DISTINCT select_list ... (NULL在DISTINCT [ON] 中视为相等)

show code 
```
postgres=# create table t3 (id int );
CREATE TABLE
postgres=# insert into t3 values (1);
INSERT 0 1
postgres=# insert into t3 values (2);
INSERT 0 1
postgres=# insert into t3 values (null);
INSERT 0 1
postgres=# insert into t3 values (null);
INSERT 0 1
postgres=# insert into t3 values (null);
INSERT 0 1
postgres=# select * from t3 ;
 id 
----
  1
  2
   
   
   
(5 rows)

postgres=# select distinct id from t3 ; // 这个结果证明了 null 在postgresql里被视为相等
 id 
----
   
  1
  2
(3 rows)
```

- DISTINCT ON 的用法
>Here expression is an arbitrary value expression that is evaluated for all rows. A
set of rows for which all the expressions are equal are considered duplicates, and
only the first row of the set is kept in the output. Note that the "first row" of a set is
unpredictable unless the query is sorted on enough columns to guarantee a unique
ordering of the rows arriving at the DISTINCT filter. (DISTINCT ON processing
occurs after ORDER BY sorting.)

>意思是DISTINCT ON ( expression [, …] )把记录根据[, …]的值进行分组，分组之后仅返回每一组的第一行。
需要注意的是，如果你不指定ORDER BY子句，返回的第一条的不确定的。如果你使用了ORDER BY 子句，
那么[, …]里面的值必须靠近ORDER BY子句的最左边。

废话不多说， 举个例子吧：

```
postgres=# CREATE TABLE score_ranking (id int, name text, subject text, score numeric);
CREATE TABLE

postgres=# INSERT INTO score_ranking VALUES (1,'killerbee','数学',99.5), (2,'killerbee','语文',89.5),(3,'killerbee','英语',79.5), (4,'killerbee','物理',99.5), (5,'killerbee','化学',98.5),(6,'刘德华','数学',89.5), (7,'刘德华','语文',99.5), (8,'刘德华','英语',79.5),(9,'刘德华','物理',89.5), (10,'刘德华','化学',69.5),(11,'张学友','数学',89.5), (12,'张学友','语文',91.5), (13,'张学友','英语',92.5),(14,'张学友','物理',93.5), (15,'张学友','化学',94.5);
INSERT 0 15
//先查看下当前表的内容。
postgres=# 
postgres=# select * from score_ranking ;
 id |   name    | subject | score 
----+-----------+---------+-------
  1 | killerbee | 数学    |  99.5
  2 | killerbee | 语文    |  89.5
  3 | killerbee | 英语    |  79.5
  4 | killerbee | 物理    |  99.5
  5 | killerbee | 化学    |  98.5
  6 | 刘德华    | 数学    |  89.5
  7 | 刘德华    | 语文    |  99.5
  8 | 刘德华    | 英语    |  79.5
  9 | 刘德华    | 物理    |  89.5
 10 | 刘德华    | 化学    |  69.5
 11 | 张学友    | 数学    |  89.5
 12 | 张学友    | 语文    |  91.5
 13 | 张学友    | 英语    |  92.5
 14 | 张学友    | 物理    |  93.5
 15 | 张学友    | 化学    |  94.5
(15 rows)



//以下就是 distict on 的用法：

//取出每门课程的第一名.
postgres=# select distinct on (subject) id,name,subject,score from score_ranking order by subject,score desc;

备注：逻辑是： 先依据subject进行分组，比如如果subject有五类，那就分成5组。其次，score desc决定每一组内部排序，最终决定每组取谁。

 id |   name    | subject | score 
----+-----------+---------+-------
  5 | killerbee | 化学    |  98.5
  1 | killerbee | 数学    |  99.5
  4 | killerbee | 物理    |  99.5
 13 | 张学友    | 英语    |  92.5
  7 | 刘德华    | 语文    |  99.5
(5 rows)

//当没用指定ORDER BY子句的时候返回的记录是不确定的。
postgres=# select distinct on (subject) id,name,subject,score from score_ranking ;
 id |   name    | subject | score 
----+-----------+---------+-------
  5 | killerbee | 化学    |  98.5
  1 | killerbee | 数学    |  99.5
  4 | killerbee | 物理    |  99.5
 13 | 张学友    | 英语    |  92.5
  2 | killerbee | 语文    |  89.5
(5 rows)

//by score,subject desc; 这里有固定格式，具体可以自己体会。 否则报错：
postgres=# select distinct on (subject) id,name,subject,score from score_ranking order by score,subject desc;
ERROR:  SELECT DISTINCT ON expressions must match initial ORDER BY expressions
LINE 1: select distinct on (subject) id,name,subject,score from scor...


// distinct on （括号里也能跟两个参数）
postgres=# select distinct on (id ,subject) id,name,subject,score from score_ranking order by id,subject,score desc;
 id |   name    | subject | score 
----+-----------+---------+-------
  1 | killerbee | 数学    |  99.5
  2 | killerbee | 语文    |  89.5
  3 | killerbee | 英语    |  79.5
  4 | killerbee | 物理    |  99.5
  5 | killerbee | 化学    |  98.5
  6 | 刘德华    | 数学    |  89.5
  7 | 刘德华    | 语文    |  99.5
  8 | 刘德华    | 英语    |  79.5
  9 | 刘德华    | 物理    |  89.5
 10 | 刘德华    | 化学    |  69.5
 11 | 张学友    | 数学    |  89.5
 12 | 张学友    | 语文    |  91.5
 13 | 张学友    | 英语    |  92.5
 14 | 张学友    | 物理    |  93.5
 15 | 张学友    | 化学    |  94.5
(15 rows)

```

完毕























