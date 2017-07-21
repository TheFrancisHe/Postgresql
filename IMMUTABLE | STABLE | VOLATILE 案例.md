1.在postgres里，函数有三个状态，这三个状态跟函数的本身有关。

2.这些状态被Postgres执行计划的优化器拿来用，如果优化器看到你这个函数是超级稳定的状态，比如说 immutable，那么优化器对于同样的参数传入（在一个sql里面），

即使你扫描了多行，我也只会处理一遍，也就是说 immutable 类似于常量，对于同样的参数传入，只执行一遍，大大简化了运算量。

IMMUTABLE : 最稳定

STABLE  ：第二稳定

VOLATILE  ：最不稳定 ：即使你传入参数是一样的，多次调用输出的结果也不一样。比如 select now();


- 对于状态为VOLATILE的函数来说，函数多次调用，输出的结果是不一样的。

```
postgres=# select now();
              now              
-------------------------------
 2017-06-07 05:47:08.937961+08
(1 row)

postgres=# select now();
              now              
-------------------------------
 2017-06-07 05:47:09.703899+08
(1 row)

postgres=# select now();
             now              
------------------------------
 2017-06-07 05:47:10.44367+08
(1 row)

```
```
postgres=# create sequence seq; //创建一个序列
CREATE SEQUENCE

postgres=#  select provolatile from pg_proc where proname='nextval'; //查询nextval函数当前所属的状态
 provolatile 
-------------
 v                              //v 表示 VOLATILE ，即最不稳定的状态。
(1 row)


VOLATILE状态意味着：
无论在一个事务里，还是在一个sql里，netval的值一直是不同的。
举例：

//在一个事务里：

postgres=# begin;
BEGIN
postgres=# select nextval('seq');
 nextval 
---------
       1
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
       2
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
       3
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
       4
(1 row)

postgres=# end; 

// 在同一条语句里：
postgres=# select nextval('seq') from t; 
 nextval 
---------
      40
      41
      42
      43
(4 rows)

postgres=# select nextval('seq') from t; 
 nextval 
---------
      44
      45
      46
      47
(4 rows)


postgres=# select nextval('seq'::regclass) from generate_series(1,3);
 nextval 
---------
      54
      55
      56
(3 rows)

可以看出来，结果都是不一样的，也就是说每次调用nextval，nextval都会执行一次。

```
－　如果把状态改掉，改为immutable呢？

又会是什么结果？
```
postgres=#  alter function nextval(regclass) immutable;
ALTER FUNCTION
//首先，在事务里，因为以下sql每次单独执行，都会被解析一次，所以每次结果也是。
postgres=# begin;
BEGIN
postgres=# select nextval('seq');
 nextval 
---------
      80
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
      81
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
      82
(1 row)

postgres=# commit;
COMMIT

那么，如果是在同一条sql里呢？
postgres=# select nextval('seq'::regclass) from generate_series(1,3);
 nextval 
---------
      83
      83
      83
(3 rows)
可见，结果都是相同的
执行计划的计划器在解析sql并执行的时候，不管你这里有多少条记录，只会执行一次。


```



－　如果把状态改掉，改为STABLE呢？
```
//事务里： 每次结果均不同，因为sql被解析了很多次。
postgres=# select nextval('seq');
 nextval 
---------
      87
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
      88
(1 row)

postgres=# select nextval('seq');
 nextval 
---------
      89
(1 row)

postgres=# commit

//如果是在同一条sql中呢？
//可以发现，依旧是不同的，但是请注意，这里涉及到一个 nextval的位置问题：
postgres=# select nextval('seq'::regclass) from generate_series(1,3);
 nextval 
---------
      90
      91
      92
(3 rows)

// 如果将nextval放在where子句里面呢？？
//以下示例参考了德哥的blog
PostgreSQL Function volatile,stable,immutable difference between QUERY select and where. 
http://blog.163.com/digoal@126/blog/static/1638770402013111851116709/

//当前是stable状态
postgres=# select currval('seq');
 currval 
---------
     113
(1 row)

// 可以看出这里返回了三行，也就是说 执行了三次nextval函数的结果都是一样的 

postgres=# select * from t where nextval('seq')=114;
 id 
----
  2
  3
  4
(3 rows)

//切换状态为 volatile; 
postgres=# alter function nextval(regclass) volatile;
ALTER FUNCTION
postgres=# select currval('seq');
 currval 
---------
     114
(1 row)
// 这里返回一行，也就是说执行的三次nextval结果均不同
postgres=# select * from t where nextval('seq')=115;
 id 
----
  2
(1 row)
```

- 如果状态是IMMUTABLE,where子句是什么情况呢？

```
postgres=# alter function nextval(regclass) immutable;
ALTER FUNCTION
postgres=#  select currval('seq');
 currval 
---------
     117
(1 row)
／／返回三行，说明三次执行返回值相同

postgres=# select * from t where nextval('seq')=118;
 id 
----
  2
  3
  4
(3 rows)
／／　这里恰好验证了这个道理，三次执行返回值相同。
postgres=# select nextval('seq'::regclass) from t ;
 nextval 
---------
     119
     119
     119
(3 rows)

postgres=# 
```

```

综上所述：VOLATILE
Stable函数在execute时执行.
Immutable函数在plan时执行.
所以stable和immutable仅仅当使用planed query时才有区别
Planed query属于一次plan, 多次execute. Immutable在planed query中也只会执行一次. 而stable函数在这
种场景中是会执行多次的
Immutable稳定性> stable> volatile



