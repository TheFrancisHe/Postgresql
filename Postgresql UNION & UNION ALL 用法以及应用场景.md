想必都接触过联合查询 Union 吧，那么 Postgresql里，union 如何使用呢 ？

Union又有哪些应用场景呢？？

废话少说，show me your demo 

```
//其实Union可以适应各种花式查询，比如：

1. 我想要一张表的前三条记录和名字为"张三"的记录。

//查询所有记录：
postgres=# select * from t_union  ;
   name    
-----------
 killerbee
 naruto
 frank
 dancer
 张三
(5 rows)

//？？想想这里为嘛会错误？？  提示语法错误。答案就是：少了 括号() ,请注意sql语法。
postgres=# select * from t_union limit 3 union select * from t_union where name=' 张三' ;
ERROR:  syntax error at or near "union"
LINE 1: select * from t_union limit 3 union select * from t_union wh...
                                      ^
//正确答案：
postgres=# (select * from t_union limit 3) union select * from t_union where name='张三' ;
   name    
-----------
 张三
 killerbee
 frank
 naruto
(4 rows)
```
// union 在生产中还有各种各种样的花式查询，以下需求来自互联网：

- 应用场景：

1.最常见的是过程表与历史表UNION 

2.相同数据表，来至不同数据源的UNION数据统计。

3.有时候利用union可以解决一些奇怪的判断语句.比如将报表的合计一起返回

4.之前使用的例子，有多个信息模块的数据，需要展示，每个模块表都有一个title,id,picture字段。为减少多次的查询SQL，使用union将这些表的数据合为一个结果集返回。






