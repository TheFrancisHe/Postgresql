- Combining Multiple Indexes

condition（查询条件）里有 and 或者 or ，并且 and 和 or 相关的列都创建了索引，那么就可以走 bitmap index的 合并。

相当于，先对索引取出数据， 这些数据有tuple id （ctid），对这些ctid做 bitmap，最后传导给

- 代码所在路径：
src/backend/executor

- 哦，对了，这篇博客有利于学习 bitmap index （oracle数据库的bitmap index ）

```
有个问题需要问楼主：
如果我的语句是 select * from table where Gender=‘男’ 
性别建立了位图索引，当我需要搜索性别为男的记录时，我需要把‘男’对应的位图向量取出来，那么取出位图向量时，我仍然需要遍历整个位图向量来获取到底哪一位是１，位图向量的长度跟表中记录的长度是一样的，此时效率岂不是还是很低吗？　这种情况下，建立位图向量和不建立位图向量的区别只是在于磁盘遍历和内存遍历的速度问题吗？

简单说来，位图是用字节8个bit中的每个bit来代表0和1的，这样占用存储就非常小。以这张表为例，假设有100万的数据，那么代表“男性”的位图的大小就是100万Bit。注意，这只是Bit，换算成字节的话，就是12.5万Byte，也就是125KB的大小，这个大小是绝对足够全部装入内存的。位图的关键是缩小了存储空间，以使得内存遍历成为可能。

即便我有足够大的机器，能够把所有记录放入内存里，对bit位的遍历(判断0和1)和遍历一条记录(比较两个值)的速度也有非常大的差距。前者有直接的机器指令支持，后者则需要多条指令才能完成，在机器指令集方面就拉开了差距。同时在进行多条件判断时，位运算也有直接的的机器指令，还支持指令集并行优化，因此综合起来，位图索引的效率就非常高了。
```


http://www.cnblogs.com/LBSer/p/3322630.html

通读完毕后，可以对 bitmap 串 ，bitmap index 应用场景有基本的认识。

>这个时候有人会说使用位图索引，因为busy只有两个值。好，我们使用位图索引索引busy字段！假设用户A使用update更新某个机器的busy值，
>比如update table set table.busy=1 where rowid=100;，但还没有commit，而用户B也使用update更新另一个机器的busy值，update table set table.busy=1 where rowid=12; 这个时候用户B怎么也更新不了，需要等待用户A commit。
>原因：用户A更新了某个机器的busy值为1，会导致所有busy为1的机器的位图向量发生改变，因此数据库会将busy＝1的所有行锁定，只有commit之后才解锁。

最后一段话，给我造成了些困扰，现在明白了，通过德哥的这篇文章：

https://github.com/digoal/blog/blob/master/201705/20170512_01.md

>不适合频繁的更新，因为更新可能带来行迁移，以及VALUE的变化。

>如果是行迁移，需要更新整个bitmap串。

>如果是VALUE变化，则需要修改整个与变化相关的VALUE的BIT串。(疑问： update的话，必定发生行迁移，德哥的意思难道是也有不发生行迁移的update么？)

>再参考这篇blog：http://www.itpub.net/thread-1003097-1-1.html  

>如果被索引的列经常被更新的话，则不适合使用位图索引。因为当更新位图所在的列时，由于要在不同的索引条目之间修改bit位，比如将第一条记录从01变为02，则必须将01所在的索引条目的第一个bit位改为0，再将02所在的索引条目的第一个bit位改为1。因此，在更新索引条目的过程中，会锁定位图索引里多个索引条目。也就是同时只能有一个用户能够更新表T，从而降低了并发性。

大概了解了：每一个bitmap 叶子节点相当于一个索引条目，（索引逻辑最小单位），当更新索引的时候，当然需要将对应的索引条目lock住，然后再更新了。


        我描述的不详细，回头用图完善下这里。
        

        
- 留个问题： 关于 heap table 和 行迁移， 我的了解仅仅趋于表面，之后会深入的,

>堆表（heap table）数据插入时时存储位置是随机的，主要是数据库内部块的空闲情况决定，获取数据是按照命中率计算，全表扫表时不见得先插入的数据先查到。

>对了bitmap index 在greenplum里支持，在PG里不支持，但是有其他形式。

```
select * from table where col = a and col = b and col2=xxx;        
-- a,b的bit串进行BITAND的操作，然后再和col2=xxx的BIT进行BITAND操作，返回BIT位为1的，使用bitmap function返回行号，取记录。    

关于这里的bitand 操作，参见百度百科

https://baike.baidu.com/item/bitand/10610292?fr=aladdin
      
select count(*) from table where col = a and col = b and col2=xxx;        
-- a,b的bit串进行BITAND的操作，然后再和col2=xxx的BIT进行BITAND操作，返回BIT位为1的，使用bitmap function返回行号，取记录，算count(*)。
```

-关于PG里的bitmap scan，可以参考德哥这篇博客：

https://github.com/digoal/blog/blob/master/201702/20170221_02.md

>对于每个查询条件，在对应索引中找到符合条件的堆表PAGE，每个索引构造一个bitmap串。

在这个bitmap串中，每一个BIT位对应一个HEAP PAGE，代表这个HEAP PAGE中有符合该条件的行。

备注：关键点在于1个bit对应1个heap page-  以及本文最后的bitmap index scan 的触发情况。

-关于 联合多列索引（字面翻译，直接看术语就行）

### 11.5. Combining Multiple Indexes

1,先介绍下，大背景：

```
A single index scan can only use query clauses that use the index's columns with operators of its operator class and are joined with 

AND. For example, given an index on (a, b) a query condition like WHERE a = 5 AND b = 6 could use the index, but a query like WHERE a = 

5 OR b = 6 could not directly use the index.
如果在 a,b 字段建立一个 联合索引 idx_ab 后，如果查询条件是 a = 5 AND b = 6 ，则不会直接使用 这个联合索引idx_ab。

Fortunately, PostgreSQL has the ability to combine multiple indexes (including multiple uses of the same index) to handle cases that 

cannot be implemented by single index scans. 联合索引用来解决单个索引无法导致索引无法使用的矛盾， 如上一段所描述的问题。

postgres=# \d abc ;
      Table "public.abc"
 Column |  Type   | Modifiers 
--------+---------+-----------
 a      | integer | 
 b      | integer | 
 c      | integer | 
Indexes:
    "idx_ab" btree (a, b)

\d: extra argument ";" ignored
postgres=# explain analyze select * from abc where a=34843 and b=62333;
                                                  QUERY PLAN                                                   
---------------------------------------------------------------------------------------------------------------
 Index Scan using idx_ab on abc  (cost=0.42..3.04 rows=1 width=12) (actual time=43.147..43.147 rows=0 loops=1)
   Index Cond: ((a = 34843) AND (b = 62333))
 Planning time: 0.087 ms
 Execution time: 58.988 ms
(4 rows)

postgres=# explain analyze select * from abc where a=34843 or b=62333;
                                                            QUERY PLAN                                           
                  
-----------------------------------------------------------------------------------------------------------------
------------------
 Bitmap Heap Scan on abc  (cost=12596.39..12624.90 rows=22 width=12) (actual time=11304.795..11347.780 rows=26 lo
ops=1)
   Recheck Cond: ((a = 34843) OR (b = 62333))
   Heap Blocks: exact=26
   ->  BitmapOr  (cost=12596.39..12596.39 rows=22 width=0) (actual time=11304.783..11304.783 rows=0 loops=1)
         ->  Bitmap Index Scan on idx_ab  (cost=0.00..1.81 rows=11 width=0) (actual time=0.057..0.057 rows=9 loop
s=1)
               Index Cond: (a = 34843)
         ->  Bitmap Index Scan on idx_ab  (cost=0.00..12594.58 rows=11 width=0) (actual time=11304.726..11304.726
 rows=17 loops=1)
               Index Cond: (b = 62333)
 Planning time: 0.089 ms
 Execution time: 11347.837 ms
(10 rows)


这段话是说， 之前多个索引不能组合使用，现在可以组合使用了，不管是 多个不同索引还是相同索引多次使用。
The system can form AND and OR conditions across several index scans.
For example, a query like WHERE x = 42 OR x = 47 OR x = 53 OR x = 99 could be broken down into four separate scans of an index on x,
each scan using one of the query clauses. The results of these scans are then ORed together to produce the result.
Another example is that if we have separate indexes on x and y, 
one possible implementation of a query like WHERE x = 5 AND y = 6 is to use each index with the appropriate query clause and then AND together the index results to identify the result rows.



```

3.这段描述 How to combine multiple indexes ~~
```
To combine multiple indexes, the system scans each needed index and prepares a bitmap in memory giving the locations of table rows that are reported as matching that index's conditions. //bitmap串在内存里，串的每个bit除了代表1，0外，还代表location。
The bitmaps are then ANDed and ORed together as needed by the query.//生成bitmap串后，进行 与  或 运算。
Finally, the actual table rows are visited and returned.
The table rows are visited in physical order, because that is how the bitmap is laid out;//返回的rows顺序是物理顺序，因为： bitmap串本身代表row的location。

参考德哥的这段话：
对于每个查询条件，在对应索引中找到符合条件的堆表PAGE，每个索引构造一个bitmap串。
在这个bitmap串中，每一个BIT位对应一个HEAP PAGE，代表这个HEAP PAGE中有符合该条件的行。//  所以，这里是物理顺序。
根据条件的多少，组成了多个bitmap。
例如 a=1 or a=2 是两个bitmap。

this means that any ordering of the original indexes is lost, and so a separate sort step will be needed if the query has an ORDER BY clause.
//
For this reason, and because each additional index scan adds extra time, the planner will sometimes choose to use a simple index scan even though additional indexes are available that could have been used as well.//如果由排序的话，查询计划器有时选择走一个简单的索引，即使有多个索引可用也如此，即使可以走索引联合。 

```

4.这段描述 使用场景，第一段说明了 多列索引和联合索引的权衡。

```
In all but the simplest applications, there are various combinations of indexes that might be useful,
and the database developer must make trade-offs to decide which indexes to provide. 
Sometimes multicolumn indexes are best, but sometimes it's better to create separate indexes and rely on the index-combination feature. //有时multicolumn索引是最好的，但是有的时候， 分开创建索引并依赖 index-combination特性更适合。
For example, if your workload includes a mix of queries that sometimes involve only column x, sometimes only column y, and sometimes both columns, you might choose to create two separate indexes on x and y, relying on index combination to process the queries that use both columns.如，有时查询条件只有x，有时只有y，有时候全部包括，这个时候，分开创建两个索引更加合适。  
（创建多列索引的话，如果只包含y，那就不适合了。）

You could also create a multicolumn index on (x, y). This index would typically be more efficient than index combination for queries  involving both columns（这句话是重点）,  but as discussed in Section 11.3, it would be almost useless for queries involving only y, so it should not be the only index. 
A combination of the multicolumn index and a separate index on y （建立了两个索引，一个只是y的，一个是 多列索引）would serve reasonably well. For queries involving only x, the multicolumn index could be used, though it would be larger and hence slower than an index on x alone. （对于只包含x的，多列索引也可以用，不过有点大并且可能有点慢。）The last alternative is to create all three indexes（最后一个可选是创建三个索引）, but this is probably only reasonable if the table is searched much more often than it is updated and all three types of query are common. If one of the types of query is much less common than the others, you'd probably settle for creating just the two indexes that best match the common types.最后这句话，看起来像是概率的问题，如果三种查询都很多，切概率一样，那么三种索引都要，如果另外两种大，那么只要另外两种就行，按概率来吧。
```
还是看场景，需要前期的压力测试。

参考德哥blog：https://github.com/digoal/blog/blob/master/201702/20170221_02.md

stackoverflow 解答

https://stackoverflow.com/questions/6592626/what-is-a-bitmap-heap-scan-in-a-query-plan

https://www.postgresql.org/message-id/12553.1135634231@sss.pgh.pa.us


  
  

