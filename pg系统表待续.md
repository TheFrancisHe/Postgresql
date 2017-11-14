系统表, 系统表之间基本上都是以oid关联. 例如pg_attrdef.adrelid 关联 pg_class.oid


```
查看系统表：
postgres=# \dS+
```


看看pg_class：
```
postgres=# \d pg_class
         Table "pg_catalog.pg_class"
       Column        |   Type    | Modifiers 
---------------------+-----------+-----------
 relname             | name      | not null
 relnamespace        | oid       | not null
 reltype             | oid       | not null
 reloftype           | oid       | not null
 relowner            | oid       | not null
 relam               | oid       | not null
 relfilenode         | oid       | not null
 reltablespace       | oid       | not null
 relpages            | integer   | not null
 reltuples           | real      | not null
 relallvisible       | integer   | not null

查看oid：
postgres=# select oid from pg_class limit 1 ;
  oid  
-------
 74719
(1 row)

这个oid也被其他字段用来做关联。
pg_class 存储所有的 object-
```
查看pg_catalog下的系统relation：
```
select relkind,relname from pg_class where relnamespace = (select oid from pg_namespace where nspname='pg_catalog') and
relkind='r' order by 1,2;
```

查看pg_catalog下系统sequence：
```
select relkind,relname from pg_class where relnamespace = (select oid from pg_namespace where nspname='pg_catalog') and
relkind='S' order by 1,2;
```
查看pg_catalog下系统view：
```
select relkind,relname from pg_class where relnamespace = (select oid from pg_namespace where nspname='pg_catalog') and
relkind='v' order by 1,2;
```





```
查看 access method （支持的索引）

postgres=# select amname from pg_am;
 amname 
--------
 btree
 hash
 gist
 gin
 spgist
 brin
(6 rows)

 r | pg_aggregate -- 聚合函数信息, 包括聚合函数的中间函数, 中间函数的初始值, 最终函数等.
 r | pg_am -- 系统支持的索引访问方法. (如btree, hash, gist, gin , spgist)
 r | pg_amop -- 存储每个索引访问方法操作符家族(pg_opfamily)中的详细操作符信息
 r | pg_amproc -- 存储每个索引访问方法操作符家族(pg_opfamily)支持的函数信息.
 r | pg_attrdef -- 存储数据表列的默认值(例如创建表时指定了列的default值).
 r | pg_attribute -- 存储数据表列的详细信息. 包括隐含的列(ctid, cmin, cmax, xmin, xmax)
 r | pg_auth_members -- 数据库用户的成员关系信息.

postgres=# select attname from pg_attribute where attrelid ='abc'::regclass ;
 attname  
----------
 a
 b
 c
 cmax
 cmin
 ctid
 tableoid
 xmax
 xmin
(9 rows)
```
