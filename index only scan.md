**ndex only scan ,是我们用select选择字段的时候，所选的字段全部都有索引，那么只需在索引中取数据，就不必访问数据块了，从而提高效率。**

中文：   http://blog.csdn.net/luojinbai/article/details/44021523
>index only scan ,是我们用select选择字段的时候，所选的字段全部都有索引，那么只需在索引中取数据，就不必访问数据块了，从而提高效率。


详细解释：https://wiki.postgresql.org/wiki/Index-only_scans

>Index-only scans are a major performance feature added to Postgres 9.2. They allow certain types of queries to be satisfied just by retrieving data from indexes, and not from tables. This can result in a significant reduction in the amount of I/O necessary to satisfy queries.

这段表达的意思，和该文的部分内容相似，可以参考下：

https://zhuanlan.zhihu.com/p/23624390

>每次给字段建一个新索引， 字段中的数据就会被复制一份出来， 用于生成索引。 因此， 给表添加索引，会增加表的体积， 占用磁盘存储空间。

所以可以直接从索引中提取数据的。



>During a regular index scan, indexes are traversed, in a manner similar to any other tree structure, by comparing a constant against Datums that are stored in the index. Btree-indexed types must satisfy the trichotomy property; that is, the type must follow the reflexive, symmetric and transitive law. Those laws accord with our intuitive understanding of how a type ought to behave anyway, but the fact that an index's physical structure reflects the relative values of Datums actually mandates that these rules be followed by types. Btree indexes contain what are technically redundant copies of the column data that is indexed.


反面例子：

```
postgres=# create index idx_test_c2 on test(c2);
CREATE INDEX
postgres=# analyze testonlyscan;
ANALYZE
postgres=# explain select * from test where c2=100;
                               QUERY PLAN                               
------------------------------------------------------------------------
 Index Scan using idx_test_c2 on test  (cost=0.29..2.91 rows=1 width=8)
   Index Cond: (c2 = 100)
(2 rows)
```
