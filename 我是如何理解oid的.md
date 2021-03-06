- 先介绍下oid的使用：
以系统表 pg_class为例，查看下postgres里各个对象（表、序列、索引 等）的oid

> pg_class 存储的都是这些对象的信息

```
postgres=# \d pg_class // 列出pg_class表的所有字段。
         Table "pg_catalog.pg_class"
       Column        |   Type    | Modifiers 
---------------------+-----------+-----------
 relname             | name      | not null
 relnamespace        | oid       | not null
 reltype             | oid       | not null
 reloftype           | oid       | not null
 relowner            | oid       | not null
 relam               | oid       | not null
 relfilenode         | oid       | not null
 reltablespace       | oid       | not null
 relpages            | integer   | not null
 reltuples           | real      | not null
 relallvisible       | integer   | not null
 reltoastrelid       | oid       | not null
 relhasindex         | boolean   | not null
 relisshared         | boolean   | not null
 relpersistence      | "char"    | not null
 relkind             | "char"    | not null
 relnatts            | smallint  | not null
 relchecks           | smallint  | not null
 relhasoids          | boolean   | not null
 relhaspkey          | boolean   | not null
 relhasrules         | boolean   | not null
 relhastriggers      | boolean   | not null
 relhassubclass      | boolean   | not null
 relrowsecurity      | boolean   | not null
 relforcerowsecurity | boolean   | not null
 relispopulated      | boolean   | not null
 relreplident        | "char"    | not null
 relfrozenxid        | xid       | not null
 relminmxid          | xid       | not null
 relacl              | aclitem[] | 
 reloptions          | text[]    | 
Indexes:
    "pg_class_oid_index" UNIQUE, btree (oid)
    "pg_class_relname_nsp_index" UNIQUE, btree (relname, relnamespace)
    "pg_class_tblspc_relfilenode_index" btree (reltablespace, relfilenode)
//有没有觉得奇怪？ 明明没有oid这个字段，但是你执行下面语句却有结果，这是为什么呢 ？
postgres=# select oid from pg_class limit 5 ;
  oid  
-------
  2671
  2672
 16399
 16405
 16407
(5 rows)

同时在 \d pg_class 命令下还有这个描述 
** "pg_class_oid_index" UNIQUE, btree (oid)**
oid 上有 btree索引，并且有唯一约束 pg_class_oid_index 。
也证明了存在oid
```
**那oid在哪儿？到底为什么会出现这种情况 ？**

来看看postgres官网对 oid的介绍：
>1.Object identifiers (OIDs) are used internally by PostgreSQL as primary keys for various system tables. 
这里表明了 oid 是内部使用，并作为系统表的主键。

>2.OIDs are not added to user-created tables, unless WITH OIDS is specified when the table is created, or the default_with_oids configuration variable is enabled.
oid不会添加到 用户自己创建的表里，除非明确指定 WITH OIDS 或者 default_with_oids 打开。

>3.Type oid represents an object identifier. There are also several alias types for oid: regproc, regprocedure, regoper, regoperator, regclass, regtype, regrole, regnamespace, regconfig, and regdictionary. Table 8-24 shows an overview.
oid代表着object identifier (对象标识号). oid 还有一些别名： regproc, regprocedure, regoper, regoperator, regclass, regtype, regrole, regnamespace, regconfig, and regdictionary

>4.The oid type is currently implemented as an unsigned four-byte integer. Therefore, it is not large enough to provide database-wide uniqueness in large databases, or even in large individual tables. So, using a user-created table's OID column as a primary key is discouraged. OIDs are best used only for references to system tables.
oid类型是unsigned four-byte integer，因此还不是足够大，不建议用作用户表的主键，最佳用处是作为系统表的主键。

根据stackoverflow的高票用户的回答：

*OIDs basically give you a built-in, globally unique id for every row, contained in a system column (as opposed to a user-space column). That's handy for tables where you don't have a primary key, have duplicate rows, etc. 
For example, if you have a table with two identical rows, and you want to delete the oldest of the two, 
you could do that using the oid column.
In my experience, the feature is generally unused in most postgres-backed applications (probably in part because they're non-standard), and [their use is essentially deprecated](http://www.postgresql.org/docs/8.4/interactive/runtime-config-compatible.html#GUC-DEFAULT-WITH-OIDS):
In PostgreSQL 8.1 default_with_oids is off by default; in prior versions of PostgreSQL, it was on by default.
The use of OIDs in user tables is considered deprecated, so most installations should leave this variable disabled. Applications that require OIDs for a particular table should specify WITH OIDS when creating the table. This variable can be enabled for compatibility with old applications that do not follow this behavior.*

大意是你要是有个表没有用主键，这时候可以把oid充当为主键使用，当然这是没办法的办法。

总结： oid是给内部表做标识用的，不推荐使用。 建议将 default_with_oids 设置为off。 
建表的时候，如果想使用主键，请自行建立。oid本身大小固定的，万一 行数超过了oid 的最大限制数（4 byte int），那就无法插入新行了。
