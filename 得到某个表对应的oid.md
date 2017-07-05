### 两种方式得到某张表的oid，这里以表 foo 为例：
```
第一种：传统方式
postgres=# select oid from pg_class where relname='foo';
  oid  
-------
 49542
(1 row)
第二种：利用pg oid别名特性
postgres=# select 'foo'::regclass::oid;
  oid  
-------
 49542
(1 row)

```

**根据个人喜好选择吧！个人觉得，后者更加专业一些**
