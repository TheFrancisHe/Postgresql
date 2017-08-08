drop database 会立即释放磁盘空间么 ？

>DROP DATABASE drops a database. It removes the catalog entries for the database and deletes the directory containing the data. It can only be executed by the database owner. 


由此可见，可以立即 释放磁盘空间。  与 DML等 语句不同，后者需要 清理进程。

```
[postgres@postgres ~] $df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       1.1T  609G  401G  61% /
devtmpfs         63G     0   63G   0% /dev
tmpfs            63G  4.0K   63G   1% /dev/shm
tmpfs            63G   49M   63G   1% /run
tmpfs            63G     0   63G   0% /sys/fs/cgroup
/dev/sda1       4.7G   97M  4.4G   3% /boot
tmpfs            13G     0   13G   0% /run/user/0

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 benchmark | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 tmp       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(5 rows)

postgres=# drop database benchmark 
postgres-# ;
DROP DATABASE

[postgres@postgres ~] $df -h 
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       1.1T  491G  519G  49% /
devtmpfs         63G     0   63G   0% /dev
tmpfs            63G  4.0K   63G   1% /dev/shm
tmpfs            63G   49M   63G   1% /run
tmpfs            63G     0   63G   0% /sys/fs/cgroup
/dev/sda1       4.7G   97M  4.4G   3% /boot
tmpfs            13G     0   13G   0% /run/user/0
```

/dev/sda2  由 61%变为49%
