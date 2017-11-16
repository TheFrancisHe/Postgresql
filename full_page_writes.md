full_page_writes (boolean)

防止页折断，OS crash导致页折断，导致页混乱，导致无法从wal日志里恢复。he wal机制连起来看。

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





http://mysql.taobao.org/monthly/2

http://blog.chinaunix.net/uid-20726500-id-5105905.html
