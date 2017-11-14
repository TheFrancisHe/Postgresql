- Multicolumn Index

>多列索引, 使用任何列作为条件, 只要条件中的操作符或函数能满足opclass的匹配,
都可以使用索引, 索引被扫描的部分还是全部基本取决于条件中是否有索引的第一
列作为条件之一.

```
// c2虽然不是驱动列（最左边的列）但是同样走了索引。
postgres=# explain select * from test where c2=200;
                                  QUERY PLAN                                   
-------------------------------------------------------------------------------
 Index Only Scan using idx_test_1 on test  (cost=0.29..1110.40 rows=1 width=8)
   Index Cond: (c2 = 200)
(2 rows)

postgres=# explain select * from test where c1=200;
                                 QUERY PLAN                                 
----------------------------------------------------------------------------
 Index Only Scan using idx_test_1 on test  (cost=0.29..2.18 rows=1 width=8)
   Index Cond: (c1 = 200) // index condition ，过滤条件。
(2 rows)
```

一般规律： c1是驱动咧， c2是非驱动列
 
 在 mysql 里，如果判断条件是 c2，则为 seqscan- 
 
 但是，pg里却不是这样

```
postgres=# explain select * from test where c2=200;
                                  QUERY PLAN                                   
-------------------------------------------------------------------------------
 Index Only Scan using idx_test_1 on test  (cost=0.29..1110.40 rows=1 width=8)
   Index Cond: (c2 = 200) // index condition ，过滤条件。
(2 rows)
```
**关于为何为这样，这是群友的探讨结果**
```
pg按cost选计划，所以走还是不走第二列的索引不是绝对的

This index could in principle be used for queries that have constraints on b and/or c with no constraint on a — but the entire index would have to be scanned, so in most cases the planner would prefer a sequential table scan over using the index.

看cost

an index on (a, b, c)
```
接下来探究下手册：

**11.3. Multicolumn Indexes**

先说下重要的段落：

```
A multicolumn B-tree index can be used with query conditions that involve any subset of the index's columns, but the index is most efficient when there are constraints on the leading (leftmost) columns.

典型的使用案例： leading index 的约束条件为 等号（=），外加 非leading index 的约束条件为不等号（用来限定具体的范围）：

The exact rule is that equality constraints on leading columns, plus any inequality constraints on the first column that does not have an equality constraint, will be used to limit the portion of the index that is scanned. 

非leading index的约束，用来check符合范围的tuples，这些操作直接在索引进行，而不是在table，这样节省访问table的cost了，但是大多数情况下呢，并没有减少scan的范围，不过扫描对象换成索引罢了。 
Constraints on columns to the right of these columns are checked in the index, so they save visits to the table proper, but they do not reduce the portion of the index that has to be scanned. 

For example, given an index on (a, b, c) and a query condition WHERE a = 5 AND b >= 42 AND c < 77, the index would have to be scanned from the first entry with a = 5 and b = 42 up through the last entry with a = 5. （这就是scan的范围没有减少，只不过这次在索引里搞定罢了。）
Index entries with c >= 77 would be skipped, but they'd still have to be scanned through. 

```
----

关于多列索引的结构，大致如下描述吧，这也是说明，为啥有时候多列索引起作用，有时候和seq scan是一个小效果了。


多列B-tree索引查询，可以被 查询条件 包含索引列的子集 所触发，但是想要获取最高的查询性能，限制条件必须包含驱动列。

如驱动列操作符为等于号，加上其余的索引列为不等号，这会限制索引的scan数量。

备注：根据知乎答案mysql的多列， b不一定能使用到索引。

```

原则上， 如果条件里没有 a，只有 b 或者c ，这时候，不得不scan整个索引了， 所以大多数情况下，查询计划更偏向走 seq scan 而不是 再使用索引了。 
疑问：走索引扫描的话，读进去的索引不是更少么？ 这得看源码了。

This index could in principle be used for queries that have constraints on b and/or c with no constraint on a — but the entire index would have to be scanned, so in most cases the planner would prefer a sequential table scan over using the index.
```

最主要的是参考德哥的博客：

https://github.com/digoal/blog/blob/master/201702/20170205_01.md

https://www.zhihu.com/question/36996520 //同样具有参考意义。 不过是mysql的。


务必将本博客内容研究清楚 。

遗留问题：

以下示例中：

For example, given an index on (a, b, c) and a query condition WHERE a = 5 AND b >= 42 AND c < 77, the index would have to be scanned from the first entry with a = 5 and b = 42 up through the last entry with a = 5.

（WHERE a = 5 AND b >= 42 AND c < 77），从a=5, b=42开始的所有索引条目，都会被扫描。 // 这里一定是这样么 ？ 第二列和第三列何时走索引呢？

Index entries with c >= 77 would be skipped, but they'd still have to be scanned through.

其他例子

（WHERE b >= 42 AND c < 77），所有索引条目，都会被扫描。只要不包含驱动列，则扫描所有索引条目。　// 

（WHERE a = 5 AND c < 77），a=5的所有索引条目，都会被扫描。 // 为什么这里不应该是 a=5, b=42  // 

（WHERE a >= 5 AND b=1 and c < 77），从a=5开始的所有索引条目，都会被扫描。　　因为　

// 我的疑问：

b列和c列，大多数情况下是不走索引的。

但是 当b列和c列也是按照 b-tree的结构排列的呢？ (情况就是知乎答案那种情况，a是固定值的时候，b才有资格谈排列。)

----------------

我自己的总结： 看 驱动列就可以了。 其余可以略过了。只不过充当一个过滤的作用。



----------------------------------------------------------------------------------------------------------------------
看看oracle的多列索引:

```
去看下索引的原理
先按a数据排序  然后同一个a列值里面再按b数据排序  然后依次
所以除去a列的话  其实bcd列的数据可以看做是无序的 
```
![oid别名](https://github.com/TheFrancisHe/Postgresql/blob/master/image/duoliesuoyin.png)
































