转载自 ：http://www.cnblogs.com/gaojian/p/3277406.html

转载自德哥 ：http://blog.163.com/digoal@126/blog/static/1638770402015226114157379/

转载自权叔 ：https://my.oschina.net/quanzl/blog/686932

关键部分：

```
何时发生Checkpoint呢？

 

下列条件任意之一会导致Checkpoint发生：

　shared_buffers中，产生了 checkpoint_segments*16MB 以上的数据。

　距离上次Checkpoint发生，经过了 checkpoint_timeout *checkpoint_completion_target 秒

　用户执行Checkpoint 命令。

PostgreSQL 9.5 以前的版本, 通过checkpoint_segments 来触发基于XLOG个数的检查点, 
通过 " (2 + checkpoint_completion_target) * checkpoint_segments + 1 " 这个公式来技术保留的XLOG个数.

PostgreSQL 9.5 将废弃checkpoint_segments 参数, 并引入max_wal_size 和 min_wal_size 参数, 通过max_wal_size和checkpoint_completion_target 参数来控制产生多少个XLOG后触发检查点, 通过min_wal_size和max_wal_size参数来控制哪些XLOG可以循环使用.

那么PostgreSQL 9.5如何计算什么时候触发检查点呢?
通过CalculateCheckpointSegments函数来计算, 依赖max_wal_size和CheckPointCompletionTarget
target = (double ) max_wal_size / (2.0 + CheckPointCompletionTarget);

5、每次 checkpoint 涉及的最大段数，原来是由我们通过参数来直接指定的，而现在

	target = (double) max_wal_size / (2.0 + CheckPointCompletionTarget);

	/* round down */
	CheckPointSegments = (int) target;

	if (CheckPointSegments < 1)
		CheckPointSegments = 1;
我们知道 CheckPointCompletionTarget 取值范围是 (0.0, 1.0)，所以最终CheckPointSegments得到的值范围是 max_wal_size 的 1/3 ～ 1/2，最小为1。

```
