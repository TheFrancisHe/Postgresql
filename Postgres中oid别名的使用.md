*文章http://blog.csdn.net/shiyibodec/article/details/52447755 给了本人很大的启示*

### 上一节介绍了oid，这节简单 oid alias 怎么使用

让我们从两个示例入手。

例1：查询表foo的所有字段。
```
postgres=# create table foo (id int,name varchar(12));
CREATE TABLE
传统方法：
note：这里面pg_class、pg_attribute均为system relation（系统表)，
pg_class存储着数据库所有对象（表、视图、索引、序列、等对象）的记录。
pg_attribute存储着所有数据库里存在的所有的字段，比如所有表的所有字段都可以在pg_attribute查询到。
postgres=# SELECT attname FROM pg_attribute WHERE attrelid = (SELECT oid FROM pg_class WHERE relname = 'foo'); 
 attname  
----------
 id
 name
 ctid
 xmin
 cmin
 xmax
 cmax
 tableoid
(8 rows)
现在，我们使用oid 的alia ：
postgres=# SELECT attname FROM pg_attribute WHERE attrelid = 'foo'::regclass;
 attname  
----------
 id
 name
 ctid
 xmin
 cmin
 xmax
 cmax
 tableoid
(8 rows)
可以看出来，'foo'::regclass 相当于语句 (SELECT oid FROM pg_class WHERE relname = 'foo')
使用 alia（别名）大大简化了操作。
```
例2：查询foo表的一些信息
```
postgres=# select oid,relname,reltuples from pg_class where oid='foo'::regclass; 
  oid  | relname | reltuples 
-------+---------+-----------
 49542 | foo     |         0
(1 row)

这里使用oid作为查询条件，可以看出来：
'foo'::regclass : 这个表达式将 表名转换成了该表的oid
```


根据官网文档：

![image](https://github.com/TheFrancisHe/Postgresql/image/20170705141459.png)

可以推导出来，其余用法类似。

比如，查看那些表的数据类型为test：

```
postgres=# select relname  from pg_class where reltype = 'test'::regtype;
 relname 
---------
 test
(1 row)

test同时也是一张表，创建的test表的同时，也创建了名为test 的数据类型，这属于自定义数据类型的知识。
```

用法大致是这样，内部原理是怎么样的，请追溯源码。
