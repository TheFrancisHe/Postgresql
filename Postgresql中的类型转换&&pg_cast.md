- Postgres 关于 pg_cast 中的介绍：
https://www.postgresql.org/docs/9.5/static/catalog-pg-cast.html

先从几个示例入手：
note:这些示例所涉及知识点： 1.类型转换 2.字符串转义

大致总结下此文： 
pg里的类型转换： 显式、隐式、assignment(有人叫赋值转换，我觉得不大妥，所以就不翻译了。) 这三种转换。

显示： 不同类型间，称为显示转换。 
隐式：同一类型间，低字节到高字节为隐式转换，比如 int 到 bigint 
assignment: 同一类型间，高字节到低字节成为发assignment ，比如 smallint到int。

```
//
implicitly-typed literal or constant
隐式转换示例：

//An escape string constant is specified by writing the letter E (upper or lower case) just before the opening single quote, e.g., E'foo'.
postgres=# create table  typecast (name varchar(32) ,id int );
CREATE TABLE
//name为varchar类型，id为int。 

postgres=# insert into typecast values ('foo',1);
INSERT 0 1
postgres=# insert into typecast values ('killBee',1);
INSERT 0 1
postgres=# insert into typecast values ('killer\nBee',1);
INSERT 0 1
postgres=# insert into typecast values (E'killer\nBee',1);
INSERT 0 1

postgres=# select * from typecast ;
    name     | id 
-------------+----
 foo         |  1
 killBee     |  1
 killer\nBee |  1
 killer     +|  1
 Bee         | 
(4 rows)
//以上插入的数据也都是同类型的，这属于隐式转换。
//同时运行sql
postgres=# select castsource::regtype,casttarget::regtype,castcontext from pg_cast where castsource='varchar'::regtype;
    castsource     |    casttarget     | castcontext 
-------------------+-------------------+-------------
 character varying | regclass          | i  // 那么为什么这里也是 隐式呢 ？？
 character varying | text              | i  // 那么为什么这里也是 隐式呢 ？？
 character varying | character         | i  // 那么为什么这里也是 隐式呢 ？？
 character varying | "char"            | a  // 那么为什么这里 是assignment 模式呢？　
 character varying | name              | i
 character varying | xml               | e
 character varying | character varying | i  //从这里可以看出来，从 character varying 到character varying 
 
 我搜寻了很多博客，从　百度百科到wiki ，终于明白，pg里定义的 显式转换、隐式转换、赋值转换（assignment）和 开发里不一样。
 区别就是在开发里，不同类型间做转换，肯定是显示转换。
 同一类型，高字节到低字节是隐式转换，低字节到高字节是显示转换（但是这种情况在pg里这种情况就叫做assignment！！！）
 
 下面开始我的验证：
 postgres=# select castsource::regtype,casttarget::regtype,castcontext from pg_cast where castsource='int'::regtype;
 castsource |    casttarget    | castcontext 
------------+------------------+-------------
 integer    | bigint           | i
 integer    | smallint         | a  //同类型高到低
 integer    | real             | i
 integer    | double precision | i
 integer    | numeric          | i
 integer    | money            | a  //同类型 money 8 字节，int 4字节，低到高。  在这里 money 可以算作 数值类型。   
 integer    | boolean          | e
 integer    | oid              | i
 integer    | regproc          | i
 integer    | regprocedure     | i
 integer    | regoper          | i
 integer    | regoperator      | i
 integer    | regclass         | i
 integer    | regtype          | i
 integer    | regconfig        | i
 integer    | regdictionary    | i
 integer    | regrole          | i
 integer    | regnamespace     | i
 integer    | "char"           | e
 integer    | abstime          | e
 integer    | reltime          | e
 integer    | bit              | e
(22 rows)

其余情况各位逗比可以自行验证看看我说的是否正确。
我比较容易较真，这是我总结出来的，我不知道官方文档有没有相应解释。
最后的最后，举几个 显示转换类型的例子来结束吧！ 

postgres=# create table  naruto (flag boolean,boobar bit);
CREATE TABLE
从上面的对比可以看出来， int 到 boolean 和 bit 都是 显式转换，所以会出现如下错误：
postgres=# insert into naruto values (1,1010101);
//提示类型不匹配
ERROR:  column "flag" is of type boolean but expression is of type integer
LINE 1: insert into naruto values (1,1010101);
                               ^
//同时给出了提示：要么重写一个值，要么进行类型转换。
HINT:  You will need to rewrite or cast the expression.
postgres=# insert into naruto values (1::boolean,1010101);
//提示第二个字段需要显示类型转换。
ERROR:  column "boobar" is of type bit but expression is of type integer
LINE 1: insert into naruto values (1::boolean,1010101);
HINT:  You will need to rewrite or cast the expression.

//成功插入数据！！ 两个值都进行了显示转换！
postgres=# insert into naruto values (1::boolean,1010101::bit);
INSERT 0 1

```
这就是PG里的 隐式转换、显式转换的例子。
下面看看assignment！

```
postgres=# create table sasuke (id smallint);
CREATE TABLE

//插入默认是int型，int 4个字节， smallint 2个字节，所以是从高字节到低字节。
//这就是 assignment ，插入成功，不需要cast或 ::进行强制转换，但是在开发里却需要标注，这是不同之处。
postgres=# insert into sasuke values (5);
INSERT 0 1
//插入失败，超出smallint的范畴。
postgres=# insert into sasuke values (55555555);
ERROR:  smallint out of range
postgres=# 
```



