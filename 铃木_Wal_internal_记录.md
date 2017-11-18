>XLOG records are written into the in-memory WAL buffer by change operations such as insertion, deletion, or commit action. 
They are immediately written into a WAL segment file on the storage when a transaction commits/aborts. 
(To be precise, the writing of XLOG records may occur in other cases. The details will be described in Section 9.5.) 
LSN (Log Sequence Number) of XLOG record represents the location where its record is written on the transaction log.
LSN of record is used as the unique id of XLOG record.

1.wal如何记录change，请看9.5

2.引入了LSN的概念，xlog的lsn代表了记录这些变化的record在xlog的位置，同时作为xlog record的唯一id。

Mysql关于LSN的描述：
>LSN(Log Sequence Number，日志序列号，ib_uint64_t) 保存在 log_sys.lsn 中，在 log_init() 中初始化，初始值为 LOG_START_LSN(8192) 。改值实际上对应日志文件的偏移量，新的 LSN＝旧的LSN + 写入的日志大小，在调用日志写入函数时，LSN 就一直随着写入的日志长度增加。


