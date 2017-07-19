单条sql插入多行，这种方式与开启事务，插入多条语句这种方式不相同，

这多条数据是在同一条sql被插入的。


- 话不多说，举个例子：


```
1.同一条语句插入多条sql
postgres=#  create table user_info(id int, info text);
CREATE TABLE
postgres=#  insert into user_info(id,info) values(1,'test'),(1,'test'),(1,'test'),(1,'test'),(1,'test');
INSERT 0 5
postgres=#  select ctid,cmin,cmax,xmin,xmax,* from user_info ;

//可以看出来这5条tuple（记录）的cmin/cmax值都是0，说明这5条记录是通过执行同一条sql产生的。
 ctid  | cmin | cmax | xmin  | xmax | id | info 
-------+------+------+-------+------+----+------
 (0,1) |    0 |    0 | 36039 |    0 |  1 | test
 (0,2) |    0 |    0 | 36039 |    0 |  1 | test
 (0,3) |    0 |    0 | 36039 |    0 |  1 | test
 (0,4) |    0 |    0 | 36039 |    0 |  1 | test
 (0,5) |    0 |    0 | 36039 |    0 |  1 | test
(5 rows)
 ctid  
 cmin 
 cmax 
 xmin   
 xmax
 
 
// 以上这几个字段均为隐藏字段（均与mvcc机制关联，这里不介绍)
// cmin和cmax 标识在同一个事务中多个语句命令的序列值，从0开始，用于同一个事务中实现版本可见性判断，其实这两个字段是相同的。
// 或者可以理解成 多条语句的执行顺序。
// 关于cmin和cmax字段，完后我会单独写一篇博客来讲述。（其实cmin与cmax是一样的）
```
这种单条sql插入多行，性能是单条数据插入单行 这种方式的好多倍。

如果是批量提交，可以考虑使用这种方式。
