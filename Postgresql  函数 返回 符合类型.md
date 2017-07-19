- 返回一个 复合数据类型（可以是某张表同名的类型）

```
//创建表 
postgres=# create table t1 (id int,name text,crt_time timestamp(0));
CREATE TABLE

//创建函数f_t1  可以注意到 返回类型是t1
postgres=# create or replace function f_t1 (i_id int) returns setof t1 as $$
postgres$# declare
postgres$# begin
postgres$# return query select * from t1 where id=i_id;
postgres$# return;
postgres$# end;
postgres$# $$ language plpgsql;
CREATE FUNCTION

//插入测试数据
postgres=# insert into t1 values(1,'killerbee',now());
INSERT 0 1
postgres=# insert into t1 values(1,'KILLERBEE',now());
INSERT 0 1

//执行函数f_t1
postgres=# select * from f_t1(1);
 id |   name    |      crt_time       
----+-----------+---------------------
  1 | killerbee | 2017-06-06 18:52:28
  1 | KILLERBEE | 2017-06-06 18:52:35
(2 rows)

postgres=# select  f_t1(1);
                f_t1                 
-------------------------------------
 (1,killerbee,"2017-06-06 18:52:28")
 (1,KILLERBEE,"2017-06-06 18:52:35")
(2 rows)
```
- 返回一个record 



