# MySql 主从复制原理
可以先看下[MySQL主从服务搭建](mysql-build-master-slave)

自己动手搭建下，印象比较深刻。

# 直观的展示
## Master processlist
登录`master`MySql服务器，执行命令`show processlist`，展示线程列表。

```
mysql> show processlist;
+----+------+--------------------+------+-------------+-------+-----------------------------------------------------------------------+------------------+
| Id | User | Host               | db   | Command     | Time  | State                                                                 | Info             |
+----+------+--------------------+------+-------------+-------+-----------------------------------------------------------------------+------------------+
|  6 | xp   | master1:52865      | NULL | Binlog Dump | 28604 | Master has sent all binlog to slave; waiting for binlog to be updated | NULL             |
|  8 | root | localhost:61860    | NULL | Query       |     0 | init                                                                  | show processlist |
+----+------+--------------------+------+-------------+-------+-----------------------------------------------------------------------+------------------+
```

这里展示的有两个线程

- `Id`为6的线程，是把二进制日志`dump`出去，所以对应的`Command`是`Binlog Dump`。

- `Id`为8的线程，就是执行当前命令的这个线程，可忽略，`Info`信息可以看出。

咱们再来看看Slave的线程列表
## Slave processlist

登录`slave MySql`服务器，执行命令`show processlist`，展示线程列表。

```
mysql> show processlist;
+----+-------------+-----------------+------+---------+-------+-----------------------------------------------------------------------------+------------------+
| Id | User        | Host            | db   | Command | Time  | State                                                                       | Info             |
+----+-------------+-----------------+------+---------+-------+-----------------------------------------------------------------------------+------------------+
|  7 | system user |                 | NULL | Connect | 28875 | Waiting for master to send event                                            | NULL             |
|  8 | system user |                 | NULL | Connect |  6283 | Slave has read all relay log; waiting for the slave I/O thread to update it | NULL             |
| 11 | root        | localhost:27662 | NULL | Query   |     0 | init                                                                        | show processlist |
+----+-------------+-----------------+------+---------+-------+-----------------------------------------------------------------------------+------------------+
3 rows in set (0.00 sec)
```

这里展示了三个线程，`Id`分别为7，8，11。


- `Id`为`7`的线程，是用来接收master的数据的，其实是存放成`relay log`，`State`信息可以看出来。

- `Id`为`8`的线程，是用来读取`relay log`的，然后将这些重放到数据库表中，这样就完成了复制。

- `Id`为`11`的线程，就是执行当前命令的这个线程，可忽略，`Info`信息可以看出。

## 整体看
Master 只有一个关于复制的线程

Slave  有两个关于复制的线程

# Master和Slave的线程是如何配合一起工作的


