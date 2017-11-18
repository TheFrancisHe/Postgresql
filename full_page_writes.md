full_page_writes (boolean)

防止页折断，OS crash导致页折断，导致页混乱，导致无法从wal日志里恢复。he wal机制连起来看。

>full_page_writes: 是否开启全页写入,此参数是为了防止块折断的一种策略,关于块折断，每种数据库都会遇到这样的问题，起因是这样的：linux操作系统文件系统一个块一般是4k,而数据库则一般是一个块8k,当数据库的脏块刷新到磁盘上时,由于底层是两个块组成的,比如刷第一个操作系统块到磁盘上了,而当刷第二个操作系统块的时候发生了停电等突然停机事故,则就发生了块折断（数据块是否折断是根据块的checksum值来检查的）,为了避免这种事故,pg采用了这样的机制:
当checkpoint后的一个块第一次变脏后就要整块写入到wal日志中,后续继续修改此块则只把修改的信息写入wal日志中,如果在此过程中发生了停电,则实例启动后会从checkpoint检查点，之后开始进行实例恢复,如果有块折断,则在全页写入的块为基础进行恢复,最后覆盖磁盘上的折断块,所以当每次checkpoint后如果数据有修改都会进行全页写入,因此控制checkpoint的
间隔是否重要,如果checkpoint_segments设置太小就会造成频繁的checkpoint,进而导致写入了过多的全页.mysql为了防止块折断采用了double write,oracle采用了redo+undo机制,其中undo记录了前镜像,而redo则既记录了修改数据又记录了undo块。

http://ju.outofmemory.cn/entry/88245

>开启的时候，在checkpoint之后的第一次对page的更改，postgres会将每 个disk page写入WAL。这样可以防止系统当机（断电）的时候，page刚好只有被写一半。打开这个选项可以保证page image的完整性。
关 闭的时候会有一定的性能增加。尤其使用带电池的RAID卡的时候，危险更低。这个选项属于底风险换取性能的选项，可以关闭

https://ruimemo.wordpress.com/2010/03/31/postgresql-performance-and-maintenance-%EF%BC%88postgres-%E4%BC%98%E5%8C%96%E4%B8%8E%E7%BB%B4%E6%8A%A4/


When this parameter is on, the PostgreSQL server writes the entire content of each disk page to WAL during the first modification of that page after a checkpoint. 

This is needed because a page write that is in process during an operating system crash might be only partially completed,

leading to an on-disk page that contains a mix of old and new data.

The row-level change data normally stored in WAL will not be enough to completely restore such a page during post-crash recovery.

Storing the full page image guarantees that the page can be correctly restored, but at the price of increasing the amount of data that must be written to WAL. 

(Because WAL replay always starts from a checkpoint, it is sufficient to do this during the first change of each page after a checkpoint. 

Therefore, one way to reduce the cost of full-page writes is to increase the checkpoint interval parameters.)

Turning this parameter off speeds normal operation, but might lead to either unrecoverable data corruption, or silent data corruption, after a system failure. 

The risks are similar to turning off fsync, though smaller, and it should be turned off only based on the same circumstances recommended for that parameter.

Turning off this parameter does not affect use of WAL archiving for point-in-time recovery (PITR) (see Section 25.3).

This parameter can only be set in the postgresql.conf file or on the server command line. The default is on.





http://mysql.taobao.org/monthly/2015/11/05/

http://blog.chinaunix.net/uid-20726500-id-5105905.html


结合wal日志的介绍：

>
Reducing checkpoint_timeout and/or max_wal_size causes checkpoints to occur more often. This allows faster after-crash recovery, since less work will need to be redone. However, one must balance this against the increased cost of flushing dirty data pages more often. If full_page_writes is set (as is the default), there is another factor to consider. To ensure data page consistency, the first modification of a data page after each checkpoint results in logging the entire page content. 

疑问：

>
In that case, a smaller checkpoint interval increases the volume of output to the WAL log, partially negating the goal of using a smaller interval, and in any case causing more disk I/O.部分程度上与“使用smaller interval ckpt 会导致更多的磁盘IO相悖”--

