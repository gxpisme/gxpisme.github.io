# MySql 主从复制加速

- 可以先看下[MySQL主从服务搭建](mysql-build-master-slave)
- 也可以再看下[MySQL主从复制原理](mysql-replication-theory)


> 先看图回顾下MySQL主从复制原理。

![](image/mysql-replication.jpg)


# 如何加速复制呢？
> master 一个 binlog dump线程
> slave 一个 存放relay log线程
> slave 另一个 relay log 重放到数据库表线程

从这三个线程着手，最好实现的是重放线程。有很多库，那就可以每个库都可以开一个线程，这就加快速度了。





