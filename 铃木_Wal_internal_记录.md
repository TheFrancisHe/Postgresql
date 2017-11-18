>XLOG records are written into the in-memory WAL buffer by change operations such as insertion, deletion, or commit action. 
They are immediately written into a WAL segment file on the storage when a transaction commits/aborts. 
(To be precise, the writing of XLOG records may occur in other cases. The details will be described in Section 9.5.) 
LSN (Log Sequence Number) of XLOG record represents the location where its record is written on the transaction log.
LSN of record is used as the unique id of XLOG record.

1.wal如何记录change，请看9.5

2.引入了LSN的概念，xlog的lsn代表了记录这些变化的record在xlog的位置，同时作为xlog record的唯一id。
