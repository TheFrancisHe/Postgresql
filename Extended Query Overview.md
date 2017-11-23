50.1.2. Extended Query Overview

In the extended-query protocol, execution of SQL commands is divided into multiple steps.

在extended-query协议里，sql命令的执行分为以下几步：

The state retained between steps is represented by two types of objects: prepared statements and portals.


A prepared statement represents the result of parsing and semantic analysis of a textual query string. 

prepared statement 代表文本query字符串经过解析和语义分析后的结果。

A prepared statement is not in itself ready to execute, because it might lack specific values for parameters.
prepared statement 并不是可以直接执行的，因为它缺少一些实际的参数。（毕竟预编译嘛）

A portal represents a ready-to-execute or already-partially-executed statement, with any missing parameter values filled in. 
portal代表可以 直接被执行 或者已经部分被执行的语句，只需要等待传递就可以完整执行。

(For SELECT statements, a portal is equivalent to an open cursor, but we choose to use a different term since cursors don't handle non-SELECT statements.)
The overall execution cycle consists of a parse step,which creates a prepared statement from a textual query string; 
a bind step,which creates a portal given a prepared statement and values for any needed parameters; 
and an execute step that runs a portal's query. In the case of a query that returns rows (SELECT, SHOW, etc), 
the execute step can be told to fetch only a limited number of rows, so that multiple execute steps might be needed to complete the operation.
The backend can keep track of multiple prepared statements and portals (but note that these exist only within a session,
and are never shared across sessions).Existing prepared statements and portals are referenced by names assigned when they were created.
In addition, an "unnamed" prepared statement and portal exist. 
Although these behave largely the same as named objects, operations on them are optimized for the case of executing a query only once and then discarding it,
whereas operations on named objects are optimized on the expectation of multiple uses.

对于Select 语句，一个portal相当于一个open cursor ，但是我们选择使用不同的术语，因为游标不处理非SELECT语句。）
整个执行周期由parse（用以对文本的query查询进行预编译）、bind（用以将给定的值同预编译后的sql绑定）；
execute（用以运行这个 portal的query，运行语句）。

当运行query是 select，show一类的，则仅仅返回有限的行，不返回全部的行，所以需要多次执行这一步，才能返回全部的数据。
当然，服务器backend会保持多个客户端的prepared statements和portals （但是注意，仅仅在当前会话里，其余会话的prepared statment 和 protals不会涉及）
