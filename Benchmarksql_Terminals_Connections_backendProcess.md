```
如果一个应用没有配置jdbc连接池-，是否意味着最多只能有一个backend process

贺冬_Postgres 2017/11/8 9:22:56
我现在不确定 backend process对应的是 应用的数量还是 应用发出请求的session的数量。
9:23:09
王亮 2017/11/8 9:23:09
这个不清楚，不了解jdbc池的实现
王亮 2017/11/8 9:23:20
代表的是连接数
王亮 2017/11/8 9:23:35
一个应用连接了100个，那么就会有100个backend

贺冬_Postgres 2017/11/8 9:23:47
连接了100个是什么意思。

贺冬_Postgres 2017/11/8 9:24:02
这100个是session 还是 数据库连接？
王亮 2017/11/8 9:24:04
做了100个连接
王亮 2017/11/8 9:24:15
数据库连接
9:25:25
贺冬_Postgres 2017/11/8 9:25:25
做了100个连接== um...  比如说java应用吧，怎么能做100个连接 ？ 

Connection conn= new Connection() ,这串代码如果多线程执行100次，可以看作100个连接吧。

贺冬_Postgres 2017/11/8 9:25:30
伪代码。

贺冬_Postgres 2017/11/8 9:25:38
Java的话。
王亮 2017/11/8 9:25:40
恩

贺冬_Postgres 2017/11/8 9:26:55
也就是意味着后台会有100个backend process 来与之对应。
王亮 2017/11/8 9:27:00
session和连接本来是同一件事情的不同层次的描述

贺冬_Postgres 2017/11/8 9:27:21
嗯，和oracle一样了。 session是依附于连接的吧。
9:27:35
王亮 2017/11/8 9:27:35
恩，客户端也应该有维护100个session的数据结构
王亮 2017/11/8 9:27:47
不是数据结构，是内存结构

贺冬_Postgres 2017/11/8 9:28:12
嗯，明白了。
王亮 2017/11/8 9:28:26
是一样，session的概念是数据库操作周期层面来讲的，连接是更底层，实用性更广

贺冬_Postgres 2017/11/8 9:29:24
以java为例，我感觉应该是 service 层没调用1次，就算作 一个session吧。

贺冬_Postgres 2017/11/8 9:29:34
不是很确定。
9:29:42
王亮 2017/11/8 9:29:42
这个不知道
王亮 2017/11/8 9:30:04
不熟悉java呢
王亮 2017/11/8 9:30:25
python还比较熟悉

贺冬_Postgres 2017/11/8 9:30:36
嗯，这就够了，这个软件每次跑的时候，后台会有100个jdbc连接-  应该是代码本身设计成这样了。

贺冬_Postgres 2017/11/8 9:30:50
哈哈，人生苦短用python-  

贺冬_Postgres 2017/11/8 9:30:54
谢了，亮哥。 
9:31:50
王亮 2017/11/8 9:31:50
ok，但是连接池的实现应该不是想象的那样，细节的话需要看代码了，好像也不会一开始就会建立100个连接等在那里，等着被别人用，这个要看具体实现了，但是概念就那么多
徐登峰 2017/11/8 9:31:53
JDBC的话 你起100个thread，光new Connection，还不行，
要调用一下connect函数，真去连
或者connect = DriverManager
                    .getConnection(“DB url”);
徐登峰 2017/11/8 9:32:45
每个thread connect后，PG就有一个新的postgres进程也就是backend process了
徐登峰 2017/11/8 9:33:31
人生苦短用python，同意
9:35:00
贺冬_Postgres 2017/11/8 9:35:00
哦，也是，是因为只有发送请求后， 
postmaster server 进程才会监听到这个请求，然后和client 握手-- 本质上也是tcp连接。


贺冬_Postgres 2017/11/8 9:35:38
应该是这么理解吧。
徐登峰 2017/11/8 9:36:43
TCP还是UDP我忘了，但是你的理解对
9:37:50
王亮 2017/11/8 9:37:50
维护连接的都是TCP的，UDP不用连接
徐登峰 2017/11/8 9:38:13
光new出来，那个connect对象还在client端，还没发送request，一调用那个函数就发送了
张元超 2017/11/8 9:39:21
getConnection的时候发起连接请求
9:40:33
张元超 2017/11/8 9:40:33

徐登峰 2017/11/8 9:42:04
对
9:46:18
贺冬_Postgres 2017/11/8 9:46:18
```

```
我:
我有个疑问，这几个backend process，
代表有postgres库上有5个session，而非5个数据库连接吧。
max_connection到底指的是不是它们的数量。
我记得Oracle里面，session是建立在connection之上的，是不一样的。
我:
纠结很久了，可是总找不到明白的解释。
我:
max_connection指的应该就是 数据库连接的数量，不是 backend process 的数量吧
桑栎:
一个 connect就会fork一个 backend process
我:
这没错。比如我用pgadmin客户端吧，我打开两个窗口-
我:
这算是在1个连接上的2个会话吧。
我:
还是算两个连接-
张文升:
两个窗口应该是两个连接啊。
张文升:
这个问题其实不用纠结的。session对客户端和server端来说是一个session，每个sesion和server需要创建连接，因为postgresql是一用户一进程的，所以每创建一个连接，postgres守护进程就会启动一个postgres服务进程。max_connection是允许创建的最大连接数的数量，其中包含几个部分，一部分是超级用户的，一部分是普通用户的。
我:
总感觉一些定义大家使用的很模糊。
张文升:
你可以到服务器上去查看你从pgadmin过来的连接，虽然client-addr一样，但是它们的端口是不同的。
张文升:
client_port不一样。
我:
嗯，是的，从pg_activity里就可以看出来。
张文升:
可能有的人的叫法你觉得不习惯。例如有人喜欢说新建一个session，有人喜欢说新建一个连接，但其实在pg中新建一个session就会新建一个连接，新建一个连接就会有一个新的进程。
张文升:
这个不矛盾的。
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
