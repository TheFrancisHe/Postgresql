>Checkpoints are points in the sequence of transactions at which it is guaranteed that the heap and index data files have been updated with all information written before that checkpoint. 
At checkpoint time,all dirty data pages are flushed to disk and a special checkpoint record is written to the log file. 
(The change records were previously flushed to the WAL files.) 
In the event of a crash, the crash recovery procedure looks at the latest checkpoint record to determine the point in the log
(known as the redo record) from which it should start the REDO operation.
Any changes made to data files before that point are guaranteed to be already on disk. Hence, after a checkpoint,
log segments preceding the one containing the redo record are no longer needed and can be recycled or removed.
(When WAL archiving is being done, the log segments must be archived before being recycled or removed.)

CKPT 是事务序列里一些点，这个点保证在检查点之前的，属于heap和index数据文件的相关信息已经被正确更新了。
备注：我个人理解的是，ckpt进程干完活后，会在xlog里记录一个点，叫做checkpoint-

ckpt过程里，所有的脏数据页面被刷新到磁盘，并且一个特殊的记录的ckpt点被记录到xlog-

- 这些刷新到磁盘的change recored会被先刷新到wal文件里）

当发生system crash时，recovery 步骤先查找xlog里最近一次的ckpt记录（也叫做 redo record），用以决定从何处开始进行故障恢复，即从redo record开始进行
xlog的redo操作。

在redo record点之前的所有关于data file的所有change都可以保证已经被刷入磁盘。因此ckpt后，包含redo record的log segment 之前的log segment已经不需要，
并且可以被recycle或者被remove了。 

- 当wal 归档完成后，log segments在被移除或者重用前必须被归档。

