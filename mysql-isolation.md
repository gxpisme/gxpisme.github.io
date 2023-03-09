# MySQL 隔离级别（Isolation）

> 之前只是模糊的知道，事务隔离级别有四个，默认的是REPEATABLE READ.
>
> 今天在观察线上DB配置的时候，发现配置的是READ COMMITTED。于是找DBA确认，其他线上都是这么配置的吗？DBA确认线上是这个READ COMMITTED隔离级别，并表示这种情况已经足够用。
>
> 脏读 dirty read、幻读 phantom read、不可重复读 Non-Repeatable Read 三者如何定义、什么区别？
>
> 之前面试别人的时候，也经常问这个知识点，但是自己也不太清晰，趁此机会，补下这个知识点。


`ACID`中的`I`就是`Isolation`，多个事务独立发生，不受干扰。

隔离级别由高到低如下

- `SERIALIZABLE`     序列化
- `REPEATABLE READ`  可重复读 简称RR  (`default` 默认隔离级别)
- `READ COMMITTED`   读已提交 简称RC
- `READ UNCOMMITTED` 读未提交


### REPEATABLE READ
`REPEATABLE READ` 可重复读，在一个事务中，执行同样的select，得到的结果是一致的。

其中按照查询条件，分为两个情况，一种是没有用到唯一索引，使用的gap lock，间隙锁；另一种是用到唯一索引，使用的lock 只是单行，具体下面来分析下。

#### 没用到唯一索引 gap lock
> 没有用到唯一索引，就会使用gap lock，这样其他事务进行操作的话，就会处于阻塞状态，必须等到该事务commit之后，释放gap lock。其他事务就能继续操作了。


```
表 和 数据

CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
```

时间线         | 事务A           | 事务B     |
--------------|----------------|-----------|
1 |  start transaction   |    |
2 |     | start transaction   |
3 |  update t set b = 0 where a = 1;   |    |
4 |     |  update t set b = 0 where a = 2;  |
5 |     |  这个一直处于wait中，超时就会失败 (ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction)  |
6 |  commit   |     |


#### 用唯一索引 lock 单行

```
表 和 数据

CREATE TABLE `t` (
  `a` int(11) primary key auto_increment NOT NULL,
  `b` int(11) DEFAULT NULL
) ENGINE=InnoDB;

INSERT INTO t VALUES (1,2),(2,2);

```

时间线         | 事务A           | 事务B     |
--------------|----------------|-----------|
1 |  start transaction   |    |
2 |     | start transaction   |
3 |  UPDATE t SET b = 3 where a = 1;   |    |
4 |     |  update t set b = 4 where a = 2;  无阻塞，直接成功 |
5 |  select b from t where a = 2; **这里查出的b值还是 2**   |     |
6 |  commit   |     |
7 |    |  commit   |


在隔离级别是`REPEATABLE READ`下，

对于具有唯一搜索条件的唯一索引，InnoDB只锁定找到的索引记录，而不锁定它之前的间隙。

对于其他搜索条件，InnoDB使用间隙锁或next-key锁锁定被扫描的索引范围，以阻止其他会话插入到该范围所覆盖的间隙中。

### READ COMMITTED & READ UNCOMMITTED
`READ COMMITTED` 读已提交

在这两个隔离级别下，锁的都是自己行，而不会像`REPEATABLE READ`在没有用到唯一索引使用gap Lock。


```
表 和 数据

CREATE TABLE `t` (
  `a` int(11) NOT NULL,
  `b` int(11) DEFAULT NULL
) ENGINE=InnoDB;

INSERT INTO t VALUES (1,2),(2,2);

```

#### READ COMMITTED 级别

时间线         | 事务A           | 事务B     |
--------------|----------------|-----------|
1 |  start transaction   |    |
2 |     | start transaction   |
3 |  UPDATE t SET b = 3 where a = 1;   |    |
4 |     |  update t set b = 4 where a = 2;  无阻塞，直接成功 |
5 |  select b from t where a = 2; **这里查出的b值是 2**   |
6 |     |  commit   |
7 |  select b from t where a = 2; **这里查出的b值是 4**   |     |
8 |  commit   |     |


#### READ UNCOMMITTED 级别
`READ UNCOMMITTED` 读未提交

时间线         | 事务A           | 事务B     |
--------------|----------------|-----------|
1 |  start transaction   |    |
2 |     | start transaction   |
3 |  UPDATE t SET b = 3 where a = 1;   |    |
4 |     |  update t set b = 4 where a = 2;  无阻塞，直接成功 |
5 |  select b from t where a = 2; **事务B未提交，这里查出的b值是 4**   |
6 |     |  commit   |
7 |  commit   |     |


### 总结下
- MySQL隔离级别分为四个 隔离级别由高到低
	- `SERIALIZABLE`	序列化
	- `REPEATABLE READ` 可重复读 简称RR  (`default` 默认隔离级别)
	- `READ COMMITTED`   读已提交 简称RC
	- `READ UNCOMMITTED` 读未提交
- `REPEATABLE READ` 可重复读，两个事务是完全隔离的，操作的同一张表的时候，按照是否使用了唯一索引进行区分。
	- 如果使用了唯一索引，那么锁定的是固定行，只要另一个事务不是操作锁定行，都可以继续操作，没有冲突；
	- 如果没有使用唯一索引，那就是使用的Gap Lock，间隙锁，那么另一个事务操作的时候就会阻塞。
- `READ COMMITTED` 读已提交
	- 这种情况下，不会使用Gap Lock，只要两个事务操作的不是同一行数据，就不会进行阻塞。
	- **其中事务A读取事务B操作完的数据，但是事务B还没有commit。事务A是读取不到的，必须等到事务B commit之后，事务A才能读到。**

- `READ UNCOMMITTED` 读已提交
	- 这种情况下，不会使用Gap Lock，只要两个事务操作的不是同一行数据，就不会进行阻塞。
	- **其中事务A读取事务B操作完的数据，但是事务B还没有commit，事务A是读取到的。**


## 脏读 & 幻读 & 不可重复读

### 脏读 dirty read
隔离级别是`READ UNCOMMITTED`，就可能会发生脏读现象。

事务A能够读到事务B未提交的内容就是脏读 dirty read。

### 幻读 phantom read
同一个事务内，同样的查询，数据返回不一致，**重点是行数变了。侧重`INSERT`和`DELETE`**

会发生的隔离级别`READ UNCOMMITTED`和`READ COMMITTED`

`REPEATABLE READ` 会通过Next-Key Locking机制解决幻读问题

![](/image/mysql-isolation-2.png)

### 不可重复读 Non-Repeatable Read
同一个事务内，两次取同行数据，发现数据不一致，**重点是同样的行，返回的行数不变。侧重`UPDATE`**

会发生的隔离级别`READ UNCOMMITTED`和`READ COMMITTED`。

![](/image/mysql-isolation-1.png)


参考资料

-  [innodb-transaction-isolation-levels](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html)
-  [what-is-the-difference-between-non-repeatable-read-and-phantom-read](https://stackoverflow.com/questions/11043712/what-is-the-difference-between-non-repeatable-read-and-phantom-read])
-  [Isolation-database-systems](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29)
