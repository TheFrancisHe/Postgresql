>Minor releases never change the internal storage format and are always compatible with earlier and later minor releases of the same major version number, e.g., 8.4.2 is compatible with 8.4, 8.4.1 and 8.4.6. To update between compatible versions, you simply replace the executables while the server is down and restart the server. The data directory remains unchanged — minor upgrades are that simple.

总结： 保持 pgdata不变，替换掉PGHOME里的所有可执行文件。 

      其实就是覆盖安装。
      
 
注意： 安装的时候注意权限问题。


主要步骤如下：

```
tar -jxvf postgresql-9.5.7.tar.bz2  
cd postgresql-9.5.7  
./configure --prefix=$PGHOME        // 应该是指定安装路径，具体 我忘了，回头查查手册。
make world -j 8  
make install-world  
```

安装时候注意权限问题。
