--------------------------------------------------------------------------------------------------
                                                        2017.7.31

alter system 修改 enable_indexscan 参数 无论如何都改不了，只能在会话级别改，为什么？？？？？

答案：db级别的参数覆盖了alter system（实例级别）的参数。

以下是相关素材：

**查看db级别设置的参数：**

pg_db_role_setting 系统relation需要搞清楚。

```
postgres=# select * from pg_db_role_setting ;
 setdatabase | setrole |       setconfig        
-------------+---------+------------------------
       13241 |       0 | {enable_indexscan=off}
(1 row)

```
**查看该参数是否被覆盖：**
```
postgres=# select name,setting,reset_val,pending_restart from pg_settings where name='enable_indexscan';
       name       | setting | reset_val | pending_restart 
------------------+---------+-----------+-----------------
 enable_indexscan | off      | off        | f
(1 row)

reset_val的值如果和默认值不同，那么该参数被覆盖了。
```

**遗留问题写博客**

重新推演一下你的这个场景，用什么方式进行排查参数是在cluster/数据库/用户级别的哪个级别做的修改？

---------------------
