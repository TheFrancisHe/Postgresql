```


实际测试现象：

```
当我在PGAdmin连接到100.222的postgres实例上的Postgres库之后，
从server上 ps -ef|grep postgres可以看出来：
已经有一个backend process来与之对应了。

此时是CRT开了两个窗口，PGAdmin连接到postgres的postgres库。

postgres   2885   1917  0 13:13 ?        00:00:00 postgres: postgres postgres [local] idle  
postgres   2971   1917  0 13:13 ?        00:00:00 postgres: postgres postgres [local] idle                         
postgres   3000   1917  0 13:14 ?        00:00:00 postgres: postgres postgres 192.168.100.1(6548) idle             
当在PGAdmin上开了两个Query command 窗口后但是未发送任意query后，
ps -ef|grep postgres的结果与上面一致。

现在分别在两个窗口发送：
select  count(*) from abc ;
后，ps -ef|grep postgres

postgres   2885   1917  0 13:13 ?        00:00:00 postgres: postgres postgres [local] idle
postgres   2971   1917  0 13:13 ?        00:00:00 postgres: postgres postgres [local] idle                         
postgres   3000   1917  0 13:14 ?        00:00:01 postgres: postgres postgres 192.168.100.1(6548) SELECT           
postgres   3498   1917  2 13:21 ?        00:00:00 postgres: postgres postgres 192.168.100.1(4416) idle             
postgres   3520   1917  1 13:21 ?        00:00:00 postgres: postgres postgres 192.168.100.1(4486) idle  

postgres=# select application_name ,client_port from pg_stat_activity where datname = 'postgres';
     application_name     | client_port 
--------------------------+-------------
 psql                     |          -1
 psql                     |          -1
 pgAdmin 4 - DB:postgres  |        6548
 pgAdmin 4 - CONN:5638817 |        4416
 pgAdmin 4 - CONN:652151  |        4486
(5 rows)5条记录，代表Postgres数据库有5个数据库连接。

在这里其实准确来讲，一个后台进程是对应一个session的。
当把terminal设置成100后，出来的100个数据库backend process ，其实对应100个session-（或者理解max_connection的那个数据连接）
再结合

由此回到Benchmarksql测试那部分，结合自带的word，也就不难理解说的 terminal被仿真成了一个数据结构，几只“猴子”不断跳来跳去，足够维护几千个连接了。

```
