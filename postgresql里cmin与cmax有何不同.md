- 在这之前，做个与cmin和cmax相关的实验：

之前已经说过， cmin与cmax代表同一个事务里，该行记录所对应的sql执行的顺序，下面验证下

```
// 当前 user_info 表的信息，当前有5条记录。 cmin 与 cmax都是0
//原先已经存在的记录：
postgres=#  select ctid,cmin,cmax,xmin,xmax,* from user_info ;
 ctid  | cmin | cmax | xmin  | xmax | id | info 
-------+------+------+-------+------+----+------
 (0,1) |    0 |    0 | 36039 |    0 |  1 | test
 (0,2) |    0 |    0 | 36039 |    0 |  1 | test
 (0,3) |    0 |    0 | 36039 |    0 |  1 | test
 (0,4) |    0 |    0 | 36039 |    0 |  1 | test
 (0,5) |    0 |    0 | 36039 |    0 |  1 | test
(5 rows)

// 同一个事务里，用4条sql插入4条语句：
postgres=# begin;
BEGIN
postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=# commit;
COMMIT

//查询结果：
postgres=#  select ctid,cmin,cmax,xmin,xmax,* from user_info ;
  ctid  | cmin | cmax | xmin  | xmax | id | info 
--------+------+------+-------+------+----+------
 (0,1)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,2)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,3)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,4)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,5)  |    0 |    0 | 36039 |    0 |  1 | test
 
 //以下4条数据为刚刚同一个事务里所插入的tuple（记录），可以看出来 cmin与cmax是递增的，因为是同一个事务里的 四条sql的结果。
 
 (0,7)  |    0 |    0 | 36041 |    0 |  1 | test
 (0,8)  |    1 |    1 | 36041 |    0 |  1 | test
 (0,9)  |    2 |    2 | 36041 |    0 |  1 | test
 (0,10) |    3 |    3 | 36041 |    0 |  1 | test
(9 rows)

不显式开启事务，直接在psql执行3条语句：

postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=# insert into user_info(id,info) values(1,'test');
INSERT 0 1
postgres=#  select ctid,cmin,cmax,xmin,xmax,* from user_info ;
  ctid  | cmin | cmax | xmin  | xmax | id | info 
--------+------+------+-------+------+----+------
 (0,1)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,2)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,3)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,4)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,5)  |    0 |    0 | 36039 |    0 |  1 | test
 (0,7)  |    0 |    0 | 36041 |    0 |  1 | test
 (0,8)  |    1 |    1 | 36041 |    0 |  1 | test
 (0,9)  |    2 |    2 | 36041 |    0 |  1 | test
 (0,10) |    3 |    3 | 36041 |    0 |  1 | test
 
 //以下是执行结果：可以发现cmin与cmax均为0， 因为它们不在同以事务里，各自属于各自的事务。只有在同一事务执行，才会排序。
 (0,11) |    0 |    0 | 36042 |    0 |  1 | test
 (0,12) |    0 |    0 | 36043 |    0 |  1 | test
 (0,13) |    0 |    0 | 36044 |    0 |  1 | test
(12 rows)
```


- 问题来了： cmax与cmin有何不同？？？

下面是pg社区的官方回复：



- What is the difference between cmin and cmax

I looked into the source code, and I think I now understand it:
cmin and cmax are same! The documentation is too old now.

I made another test:
In terminal A:

```
pgsql=# begin;
BEGIN
pgsql=# select * from tab01;
 id | cd
----+----
(0 rows)

pgsql=# select xmin,xmax,cmin,cmax,* from tab01;
 xmin | xmax | cmin | cmax | id | cd
------+------+------+------+----+----
(0 rows)

pgsql=# insert into tab01 values(1,'1'),(2,'2'),(3,'3');
INSERT 0 3
pgsql=# insert into tab01 values(4,'4'),(5,'5'),(6,'6');
INSERT 0 3
pgsql=# select xmin,xmax,cmin,cmax,* from tab01;
 xmin | xmax | cmin | cmax | id | cd
------+------+------+------+----+----
 1897 |    0 |    0 |    0 |  1 | 1
 1897 |    0 |    0 |    0 |  2 | 2
 1897 |    0 |    0 |    0 |  3 | 3
 1897 |    0 |    1 |    1 |  4 | 4
 1897 |    0 |    1 |    1 |  5 | 5
 1897 |    0 |    1 |    1 |  6 | 6
(6 rows)

pgsql=# commit;

Then I begin to delete record:
pgsql=# begin;
BEGIN
pgsql=# delete from tab01 where id=1 or id=2;
DELETE 2
pgsql=# delete from tab01 where id=3;
DELETE 1
pgsql=# delete from tab01 where id=4;
DELETE 1
pgsql=# delete from tab01 where id=5;
DELETE 1
pgsql=#

But I have not commit my deleting action.
Then in terminal B, I can see:
pgsql=# select xmin,xmax,cmin,cmax,* from tab01;
 xmin | xmax | cmin | cmax | id | cd
------+------+------+------+----+----
 1897 | 1898 |    0 |    0 |  1 | 1
 1897 | 1898 |    0 |    0 |  2 | 2
 1897 | 1898 |    1 |    1 |  3 | 3
 1897 | 1898 |    2 |    2 |  4 | 4
 1897 | 1898 |    3 |    3 |  5 | 5
 1897 |    0 |    1 |    1 |  6 | 6
(6 rows)

pgsql=#
```

---------------
In fact , in the source code of PG,I can find it---heap_getsysattr function

in heaptuple.c,here is it:

--------code begin----
```
case MinCommandIdAttributeNumber:
case MaxCommandIdAttributeNumber:

/*
 * cmin and cmax are now both aliases for the same field, which
 * can in fact also be a combo command id. XXX perhaps we should
 * return the "real" cmin or cmax if possible, that is if we are
 * inside the originating transaction?
 */
result = CommandIdGetDatum(HeapTupleHeaderGetRawCommandId(tup->t_data));
break;
```
--------code end-------
