- 先说下操作符号，就以 + 和 * 举个例子：

```
//求 3+4 的和
postgres=# select 3+4 ;
 ?column? 
----------
        7
(1 row)
//通过调用专属操作符求和 OPERATOR是函数,pg_catalog.+ 可以看作传入参数
postgres=# select 3 OPERATOR(pg_catalog.+) 4 ;
 ?column? 
----------
        7
(1 row)
//用这种开发思维来求 3和4的积
postgres=# select 3 OPERATOR(pg_catalog.*) 4 ;
 ?column? 
----------
       12
(1 row)


```

- 关于特殊符号的介绍：
![特殊符号介绍](https://github.com/TheFrancisHe/Postgresql/blob/master/image/teshufuhao.png)

这里以$符号在prepare statement中的使用为例吧！

```
postgres=# prepare p1 (smallint) as select * from sasuke where id=$1;
PREPARE
//这里面的$1表示第一个变量。  即 p1
postgres=# execute p1(5);
 id 
----
  5
(1 row)
```

