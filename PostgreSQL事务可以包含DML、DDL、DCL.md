- 一个事务最大2^32条SQL(因为cmin,cmax的长度是4Bytes)
- PostgreSQL一个事务中可以包含DML, DDL, DCL.
  
  除了以下:
  
create tablespace

create database


使用concurrently并行创建索引

其他未尽情况略

(Oracle执行DDL前自动将前面的未提交的事务提交,所以Oracle不支持在事务中执行DDL语句)

- 这种情况和Oracle不同，oracle在执行ddl语句（如 drop table）会自动把事务提交掉。但是在Postgres里，ddl可以放到事务里进行。
- 废话不多说，举个例子：

```
postgres=# begin;
BEGIN
postgres=# drop table t ;
DROP TABLE
postgres=# rollback ;
ROLLBACK
postgres=# \d t
                         Table "public.t"
 Column |  Type   |                   Modifiers                    
--------+---------+------------------------------------------------
 id     | integer | not null default nextval('t_id_seq'::regclass)
```

个人觉得这是一个很好的特性，在进行关键操作，如 drop truancate 时，可以放到事务里进行。
