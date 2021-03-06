Q:
>高工，有个问题请教下：
多列索引在 abc 三个字段上， 既然只是看 a字段来决定走不走索引，那么多列索引存在的意义是什么？
这是我的想法： 场景里面会有a=xx&&b>xx ，这种情况下，ab就可以做个多列索引，这样，直接在索引里通过b>xx进行过滤，要比 只在a上做索引这种情况快 ？  

A:
>多字段索引适合于查询条件比较固定的场景，不然则不建议使用

Q:
>嗯，查询条件固定后，多列索引快在哪里呢？相比于 只在最左边字段 a 建立索引。

A:
>如果只确定a，这个并不代表是你要的行，还要进行filter,可能返回的数据就要大很多，可选择性差

Q:
>比如说这种情况：
filter指的是还要回到 table里去过滤吧。
a=5&&b>6  和 a=5  , 意思是 后者过滤的行多么？
前者直接在索引里就得出结果了，而后者还得去表里再对ａ＝５的所有数据进行过滤，是这样么？

A:
>如果还有其他条件都要去表里filter，只是多列需要扫描的数据就更少了

Q:
>只是多列需要扫描的数据就更少了 ———— 也就说在多列索引ab里进行b>6的过滤， 比在表里更快是么？

A:
>那必须啊

Q:
>是因为 读取的行数不一样么？　如果索引只作用于ａ列的话，那么取出到内存的行数相对来说比较多。　　

A:
>优化有个目标，就是尽量返回少的数据,复合索引不就是这个目标嘛,如果还不理解，自己拿数据去体会吧.

----------------------------
```
有个问题：

where a=5&&b>5 ，索引在a列-  

贺冬_Postgres 2017/8/1 17:36:35
这种情况是 先把所有a=5的列取出到内存

贺冬_Postgres 2017/8/1 17:36:46
然后在进行 b>5的过滤么 ？

贺冬_Postgres 2017/8/1 17:37:14
就这么一个疑问，多列索引就没有问题了。

贺冬_Postgres 2017/8/1 17:37:21
 求解释啊-
 
 王硕：
 Index Scan using testhd_a_index on testhd  (cost=0.29..8.31 rows=1 width=8) (actual time=0.095..0.095 rows=0 loops=1)
   Index Cond: (a = 5)
   Filter: (b > 5)
   Rows Removed by Filter: 1

贺：
Index Cond: (a = 5)是不是代表把所有a=5的列取出到内存里呢？
Filter: (b > 5)是不是代表在内存里进行过滤呢？

王：
对  它先会根据索引找到对应的ctid  然后拿这个ctid集合找到所有tuple 然后filter
这个filter会根据情况或进行hash 或进行顺序

贺：
找到所有tuple____找到后所有满足条件a=5的tuple 都在内存里了吧。然后再在内存里对这些tuple进行过滤（b>5） 这个逻辑没错吧-   


wang:嗯  具体内部细节  还有很多

he:

所以如果业务场景里，经常出现  where a=xx&&b>xx ,最好在ab两列做多列索引- 这样的话，就不会有filter了吧。

贺冬_Postgres 2017/8/1 17:55:43
直接在 索引里进行删选了。 

贺冬_Postgres 2017/8/1 17:55:53
我没做实验，这是猜想。
2017/8/1 17:56:16
贺冬_Postgres 2017/8/1 17:56:16
没有filter就代表 返回的tuple更少更准确了。

贺冬_Postgres 2017/8/1 17:56:30
这就达到索引的目的了： 永远返回最少的行- 

贺冬_Postgres 2017/8/1 17:58:03
索引是这个样子么？

wang:
对。

```



```
postgres=# insert into abc select (random()*(10^5))::integer,(random()*(10^5))::integer,(random()*(10^5))::integer from generate_series(1,1000000) ;
INSERT 0 1000000


postgres=# analyze abc ;
ANALYZE
postgres=# \d abc
      Table "public.abc"
 Column |  Type   | Modifiers 
--------+---------+-----------
 a      | integer | 
 b      | integer | 
 c      | integer | 
Indexes:
    "idx_ab" btree (a, b)

postgres=# select * from abc limit 20 ;
   a   |   b   |   c   
-------+-------+-------
 89523 | 22512 | 83978
 99652 | 47678 |  7156
 36210 | 35428 | 27382
 19187 | 15047 | 60978
 57986 | 54399 |  4676
 91226 | 61242 | 78117
 67865 | 56228 | 23995
 90514 | 92676 | 41454
 85538 |  5706 | 47803
 34830 | 39928 | 61832
 46037 | 29451 | 84344
 30014 | 29103 | 32022
 37170 | 65314 | 67450
 64552 | 84501 | 82498
 25530 | 42487 | 36896
 30206 | 33713 | 98138
  8323 |  1578 | 54366
 32317 | 92092 | 47042
 73772 | 77630 | 52748
 21575 | 12459 | 92676
(20 rows)

postgres=# explain select * from abc where a=85533 and b=85538 ;
                            QUERY PLAN                             
-------------------------------------------------------------------
 Index Scan using idx_ab on abc  (cost=0.42..3.04 rows=1 width=12)
   Index Cond: ((a = 85533) AND (b = 85538)) // 不需要判断c，所以直接过滤了。
(2 rows)

postgres=# explain select * from abc where a=85533 and b=85538 and c<100;
                            QUERY PLAN                             
-------------------------------------------------------------------
 Index Scan using idx_ab on abc  (cost=0.42..3.05 rows=1 width=12)
   Index Cond: ((a = 85533) AND (b = 85538))
   Filter: (c < 100)// c无索引，所以需要取出所有数据，然后在内存里进行过滤。
(3 rows)

```
总结:  存在大量数据的情况下，如果c列没有索引，但是where 条件有c，这时候，先根据ab把相应的tuple取出到内存，再在内存里进行c条件的过滤，这是filter-

如果where 条件没有c，则直接在index里就直接得出结果了，不需要再取出所有行，进行c条件的过滤了，这就不需要filer了。

(数据较少甚至不走索引，走不走索引还是看 cost - 如果走索引的cost较小，那么就会走索引了。)：

-----------------------

上面就是我的猜想，结果也证实了猜想。

当然，我会在手册找到相应的描述的-- 

在手册的 调优部分，performance 应该有， 自己可以找找。


