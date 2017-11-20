Oracle 实例崩溃恢复：

断电后，dirty buffer 丢失了：

哪些buffer 可以丢失？  

未提交事务对应的事务可以丢失

已提交事务对应的dirty_buffer 可以丢失。


看看这种情况：

dirty buffer 所对应的事务已经提交 ：这种脏块丢失无法接受。

Oracle崩溃之后，有些脏块应该找回来。

通过redo log 找回来。

一部分日志在log buffer里，一部分在redo log 里。

如果一个事务所对应的log记录还在log buffer里，那么，说明这个事务还未提交，那么事务对应的脏块可以丢失。


------------------------------

实例崩溃恢复，使用redo log 将数据库崩溃的瞬间所对应的脏块构造出来。

从redo log 里的哪条记录恢复呢？

恢复起点：redo log 

恢复终点：redo log（current 日志最后一条） 最后一条。

LRBA：检查点队列最早脏块对应的日志。 这个点代表此前所有的日志对应的脏块已经写到磁盘了。

之后的日志都是最引的

CKPT运行的时候，把起点记录到control file 里，oracle启动起来后，找到lrba地址。

日志是按照顺序跑的。跑日志，未提交事务也有部分构造出来。

对所有未提交事务都会回滚。

-----------------

1. 首先Oracle 发现db 非正常关闭，需要做实例恢复。

2. 在control file中找到lrba地址，从而找到日志起点，然后找到终点（最后一条日志）。

3. 这时候Oracle把已提交事务对应的脏块都构造出来，但是部分未提交事务对应赃块也会构造从出来，这时候需要用到undo-

4. 在此后的操作中，自动将未提交事务自动进行回滚。


UNDO表空间有undo段，自动使用undo表空间的undo 段是如何使用的？

开始一个事务的时候，使用到undo表空间，

比如delete一个事务，会把修改前的数据，放到undo表空间的undo段里。

回滚

CR块

实例崩溃恢复：


前滚日志，回滚脏块。

未提交事务日志所对应的脏块，需要回滚。

回滚也记录日志。





