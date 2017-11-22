故障切换：

Connection Fail-over

To support simple connection fail-over it is possible to define multiple endpoints (host and port pairs) in the connection url separated by commas. The driver will try to once connect to each of them in order until the connection succeeds. If none succeed, a normal connection exception is thrown.
The syntax for the connection url is:

jdbc:postgresql://host1:port1,host2:port2/database

The simple connection fail-over is useful when running against a high availability postgres installation that has identical data on each node. For example streaming replication postgres or postgres-xc cluster.
For example an application can create two connection pools. One data source is for writes, another for reads. 


负载均衡：

The write pool limits connections only to master node:
jdbc:postgresql://node1,node2,node3/accounting?targetServerType=master . 

And read pool balances connections between slaves nodes, but allows connections also to master if no slaves are available:
jdbc:postgresql://node1,node2,node3/accounting?targetServerType=preferSlave&loadBalanceHosts=true

targetServerType
Allows opening connections to only servers with required state, the allowed values are any, master, slave and preferSlave. 

The master/slave distinction is currently done by observing if the server allows writes.

The value preferSlave tries to connect to slaves if any are available, otherwise allows falls back to connecting also to master.

loadBalanceHosts = boolean
In default mode (disabled) hosts are connected in the given order. If enabled hosts are chosen randomly from the set of suitable candidates.



refer:https://jdbc.postgresql.org/documentation/94/connect.html#connection-parameters

      
      
