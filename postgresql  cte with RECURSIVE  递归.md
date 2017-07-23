今天说说 Postgresql 的 CTE (Common Table Expressions)

今天讲讲其应用场景和应用方式吧！

- 说说with（CTE）是为了满足什么需求而存在的：

以下这两段话的链接：http://www.craigkerstiens.com/2013/11/18/best-postgres-feature-youre-not-using/
>SQL by default isn’t typically friendly to dive into, and especially so if you’re reading someone else’s already created queries. For some reason most people throw out principles we follow in other languages such as commenting and composability just for SQL. I was recently reminded of a key feature in Postgres that most don’t use by @timonk highlighting it in his AWS Re:Invent Redshift talk. The simple feature actually makes SQL both readable and composable, and even for my own queries capable of coming back to them months later and understanding them, where previously they would not be.



>The feature itself is known as CTEs or common table expressions, you may also here it referred to as WITH clauses. The general idea is that it allows you to create something somewhat equivilant to a view that only exists during that transaction. You can create multiple of these which then allow for clear building blocks and make it simple to follow what you’re doing.

大致总结下这段话的含义：

随着业务的复杂，所编写的sql变得越来越复杂，尤其是别人写的复杂sql，需要很长时间读懂。

由于种种原因，大家现在写sql也不按照一定的原则来，而且sql的可读性和重用性大大降低。

CTEs这个特性，就是为了简化sql，使得sql变得更加可读的~~



- 再看看官方文档的介绍：

>WITH provides a way to write auxiliary statements for use in a larger query. 
These statements, which are often referred to as Common Table Expressions or CTEs,
can be thought of as defining temporary tables that exist just for one query.
Each auxiliary statement in a WITH clause can be a SELECT, INSERT, UPDATE, or DELETE; 
and the WITH clause itself is attached to a primary statement that can also be a SELECT, INSERT, UPDATE, or DELETE.

大致总结下这段话：
1.with 提供辅助性的语句，可以用来简化查询。

2.这些辅助性的语句就是CTEs（Common Table Expressions）

- 可以看出，以上是一个意思。CTEs作为辅助性语句，可以大大简化sql。
- 问题来了，CTEs是如何简化sql 的？？

话不多说，举个例子：

- 非递归形式的cte用法：

示例来源:

```
testdb# select * from COMPANY;
 id | name  | age | address   | salary
----+-------+-----+-----------+--------
  1 | Paul  |  32 | California|  20000
  2 | Allen |  25 | Texas     |  15000
  3 | Teddy |  23 | Norway    |  20000
  4 | Mark  |  25 | Rich-Mond |  65000
  5 | David |  27 | Texas     |  85000
  6 | Kim   |  22 | South-Hall|  45000
  7 | James |  24 | Houston   |  10000
(7 rows)
  
// 现在，让我们写一个查询使用WITH子句来选择从上面的表中，记录如下：
With CTE AS
(Select
 ID
, NAME
, AGE
, ADDRESS
, SALARY
FROM COMPANY )
Select * From CTE; 

// 以上PostgreSQL的表会产生以下结果： 
id | name  | age | address   | salary
----+-------+-----+-----------+--------
  1 | Paul  |  32 | California|  20000
  2 | Allen |  25 | Texas     |  15000
  3 | Teddy |  23 | Norway    |  20000
  4 | Mark  |  25 | Rich-Mond |  65000
  5 | David |  27 | Texas     |  85000
  6 | Kim   |  22 | South-Hall|  45000
  7 | James |  24 | Houston   |  10000
(7 rows)

```
可以看出来，CTE的一个重要功能，就是大大简化子查询，

出自之外，在执行计划上也做了一部分优化。

具体可以自行搜索资料学习，查看官方文档。

- 递归查询的一种顺序：

http://francs3.blog.163.com/blog/static/40576727201011815721822/

https://segmentfault.com/a/1190000007402657


- 另外一种顺序的递归查询

示例来源：

http://francs3.blog.163.com/blog/static/40576727201011815721822/

- 递归查询的一些原理：

https://technet.microsoft.com/zh-cn/library/ms186243(v=sql.105).aspx#示例

https://my.oschina.net/Kenyon/blog/55137

- 递归的执行步骤：

http://blog.csdn.net/habren/article/details/51137094

- 递归的一个优化案例：
http://www.alonely.com.cn/PostgreSQL/20160923/42959.html

- 递归查询的其他案例：

http://blog.163.com/digoal@126/blog/static/163877040201132843255911/

其余请参考德哥的blog---





