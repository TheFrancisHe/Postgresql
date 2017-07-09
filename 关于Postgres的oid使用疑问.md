- 之前，我已经介绍了oid是什么，oid应该如何使用。
但是在使用oid的过程中，我自己碰到一个问题：

```
//1.运行完如下语句后，'boolean'::regtype的值是 boolean。
postgres=# select 16::regtype ;
 regtype 
---------
 boolean
(1 row)

//2.于是我就推断：如果运行select 'boolean'::regtype应该返回的是16吧！ （16是其对应的 oid 的值）
//于是运行如下语句：
postgres=# select 'boolean'::regtype ;
 regtype 
---------
 boolean // 为什么不是16？　为什么　select 16::regtype 和 select 'boolean'::regtype 结果是一样的？

//再看一条语句：
 
postgres=# SELECT attname FROM pg_attribute WHERE attrelid = 'foo'::regclass;//在这里可以看出来'foo'::regclass返回是oid类型。
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
```
总结下问题：  当使用 oid 别名的时候（也就是形式大概是xxxx::regxxxx  ，比如上面的 16::regtype 、'boolean'::regtype 、'foo'::regclass ），
什么时候返回的是oid，什么时候返回的是oid对应的name呢  ?


- 在这里，官方文档我自己没有找到相关论述，如果你找到了请告我下
- 再上stackoverflow 看看高票是如何描述的：
链接：https://stackoverflow.com/questions/13289107/what-does-regclass-signify-in-postgresql
里面描述了：  

>Casting to regclass is a shortcut way of saying "this the name of a relation, 
please convert it to the oid of that relation".
转换成 regclass相当于以一种简单的方式说：这是该relation（表）的名字，请将其转换成oid。

显然，高票答案也米有解决问题。

- 最后只能翻阅源码了。
通过同事的帮助，定位到了源码文件：
src/backend/utils/adt/regproc.c

最终确定以下两个函数决定::regclass/regtype等别名的转换和输出：
regrolein 和 regroleout 这两个函数决定的。

大概逻辑是：
16::regtype 和 'boolean'::regtype 他们都会被转换成oid，但是只要输出到psql命令行，就会都转换成oid对应的name
如果不输出到命令行，则不进行转换，比如： WHERE attrelid = 'foo'::regclass;就没有转换其对应的name即'foo',因为只是在内部进行判断，
而没有进行输出。



