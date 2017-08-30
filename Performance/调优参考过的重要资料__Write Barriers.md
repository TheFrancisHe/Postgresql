Linux OS: Write Barriers  :

http://blog.163.com/digoal@126/blog/static/163877040201132692318242/      //备注，德哥这篇讲的不详细，具体参考自己的word文档。


//这篇是官方文档，有大致介绍，但细节部分，参考自己总结的word文档。

https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Storage_Administration_Guide/writebarrieronoff.html


什么是 metadata （元数据）

http://blog.csdn.net/yangzhongxuan/article/details/5205085

Linux日志文件系统及性能分析

http://www.cnblogs.com/bodhitree/p/6047598.html

Barriers and journaling filesystems （显示了和 日志文件系统的关系）

https://lwn.net/Articles/283161/

Cache写机制：Write-through与Write-back：

http://witmax.cn/cache-writing-policies.html

硬盘缓存有什么用？硬盘缓存的作用 

http://www.gezila.com/tutorials/17460.html

日志文件系统是怎样工作的

http://linuxperf.com/?p=153

浅谈数据库系统中的cache
http://www.hellodb.net/2010/08/db-storage.html

Direct-access devices //设备直接读

http://www.staff.uni-mainz.de/tacke/scsi/SCSI2-09.html

细说IDE硬盘的容量限制

http://article.pchome.net/content-7882html

解析 Linux 中的 VFS 文件系统机制

https://www.ibm.com/developerworks/cn/linux/l-vfs/

Windows存储管理之磁盘类型简介

https://community.emc.com/docs/DOC-18438?decorator=print

postgresql 预写式日志(Write Ahead Long)

http://www.cnblogs.com/daduxiong/archive/2010/08/20/1804301.html

ext3，ext4，xfs和btrfs文件系统性能对比

http://www.cnblogs.com/tommyli/p/3201047.html

XFS

https://zh.wikipedia.org/wiki/XFS

由异常掉电问题---谈xfs文件系统

http://blog.csdn.net/huyangg/article/details/48524955


Linux 内核的文件 Cache 管理机制介绍

https://www.ibm.com/developerworks/cn/linux/l-cache/

在PostgreSQL上插入性能最好的文件系统是什么？

https://gxnotes.com/article/138576.html

四大Linux文件系统在2.6.34内核下的基准测试

http://os.51cto.com/art/201004/196827_all.htm


Linux内核的文件预读(readahead) 

http://blog.chinaunix.net/uid-667478-id-2384354.html

Linux文件预读分析以及评估对系统的影响

http://blog.yufeng.info/archives/1347

Linux内核的文件预读readahead

http://www.cnblogs.com/kerrycode/p/4743015.html


readahead

https://zh.wikipedia.org/wiki/Readahead


部分内容比官方手册描述的详细些。

https://www.postgresql.org/docs/9.3/static/kernel-resources.html

Managing Kernel Resources

了解你的磁盘之使用bonnie++测试磁盘性能 

http://blog.chinaunix.net/uid-24774106-id-3728780.html


swap

https://wiki.archlinux.org/index.php/Swap_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)


Linux命令-自动挂载文件/etc/fstab功能详解[转]

http://www.cnblogs.com/qiyebao/p/4484047.html


nodiratime:不修改目录的atime

http://blog.csdn.net/hunanchenxingyu/article/details/38308217?locationNum=1&fps=1

Linux 系统中一些针对文件系统的节能技巧

https://www.ibm.com/developerworks/cn/linux/l-cn-fsgreen/

调整linux内核尽量用内存，而不用swap

http://www.myjishu.com/?p=80

Tuning Red Hat Enterprise Linux for Sybase Databases

https://access.redhat.com/solutions/69988


原]性能优化：Swap调优

http://itindex.net/detail/54785-%E6%80%A7%E8%83%BD-%E4%BC%98%E5%8C%96-swap

Linux的内存回收和交换

http://liwei.life/2016/06/27/linux%E7%9A%84%E5%86%85%E5%AD%98%E5%9B%9E%E6%94%B6%E5%92%8C%E4%BA%A4%E6%8D%A2/

swappiness

参数值可为 0-100，控制系统 swap 的程序。高数值可优先系统性能，在进程不活跃时主动将其转换出物理内存。低数值可优先互动性并尽量避免将进程转换处物理内存，并降低反应延迟。默认值为 60


理解LINUX的MEMORY OVERCOMMIT

http://linuxperf.com/?p=102



vm overcommit参数

http://mdba.cn/2016/01/11/vm-overcommit%E5%8F%82%E6%95%B0/

swap与释放内存。

http://9237101.blog.51cto.com/9227101/1921268

目前这个这个时代，swap space还有什么意义？ 

https://www.zhihu.com/question/26573247
