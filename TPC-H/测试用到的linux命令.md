- 建立软链接：

```
ln -s /home/postgres/2.17.2/dbgen /tmp/dss-data

不需要先建立 dss—data文件夹
```

- 后台执行脚本：


```
直接在crt端执行，可以随便按下回车-- 此时进程还在后台执行。

在此期间可以用ps -ef|grep tpch 命令，看看进程是否执行完毕。

nohup ./tpch.sh ./results tpch postgres &

```
