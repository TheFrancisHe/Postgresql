
- psql相关的事务模式变量
```
ON_ERROR_ROLLBACK, ON_ERROR_STOP
postgres=# \set ON_ERROR_ROLLBACK on
```
如果开启ON_ERROR_ROLLBACK, 会在每一句SQL前设置隐形的savepoint, 可以继续下面的SQL, 而不用全部回滚

- 先举例说明该参数的效果：

```
postgres=# \set ON_ERROR_ROLLBACK on // 在当前会话设置on_error_rollback为on
postgres=# begin;
BEGIN
postgres=# sele;//故意模拟某一条sql执行错误。
ERROR:  syntax error at or near "sele"
LINE 1: sele;
        ^
postgres=# insert into t values (1); //发现，哪怕上一条sql执行错误，该条sql也能执行成功。
INSERT 0 1
postgres=# insert into t values (1);
INSERT 0 1

postgres=# sele;//再次模拟某一条sql执行错误。
ERROR:  syntax error at or near "sele"
LINE 1: sele;
        ^
postgres=# insert into t values (1);//继续执行从成功。
INSERT 0 1
postgres=# commit;//成功提交，ON_ERROR_ROLLBACK的启用说明在一个事务里，错误sql自动回滚，不影响其余sql的执行。
COMMIT
postgres=# 
```
- 下面我们再看看，如果将ON_ERROR_ROLLBACK参数关闭，会有什么后果。
```
postgres=# \set ON_ERROR_ROLLBACK off //在当前会话设置on_error_rollback为on
postgres=# begin; //开启事务
BEGIN
postgres=# sele; //故意模拟某一条sql执行错误。
ERROR:  syntax error at or near "sele"
LINE 1: sele;
        ^
postgres=# insert into t values (1); //事务里，在上一条语句执行错误后，尝试执行插入语句，却发现执行错误。
ERROR:  current transaction is aborted, commands ignored until end of transaction block//当前事务被aborted，接下来所有的sql将会执行失败----
postgres=# insert into t values (1);
ERROR:  current transaction is aborted, commands ignored until end of transaction block
postgres=# rollback ;
ROLLBACK
```

- 那么该参数的到底是处于什么需求呢？？

请看看这个需求

https://stackoverflow.com/questions/14908451/is-there-a-way-to-set-an-option-that-will-cause-a-postgresql-script-to-continue/14910166#14910166

大致是提问者要在一个事务里，插入很多条数据，这些数据数据量很大，但不是很重要。

由于该事务所运行的大量sql语句，是特定程序模拟仿真出来的，提问者不想因为几条sql而使得这个导入数据的事务执行失败。

这些失败数据也不是很重要。作者提问有没有什么方法可以忽略执行错误的sql？？

** 这不就是该参数一个很典型的场景么？ **

- 官网的介绍。

> ON_ERROR_ROLLBACK When on, if a statement in a transaction block generates an error,
the error is ignored and the transaction continues. When interactive, such errors are only ignored in interactive sessions,
and not when reading script files. 
When off (the default), a statement in a transaction block that generates an error aborts the entire transaction


- 以上










