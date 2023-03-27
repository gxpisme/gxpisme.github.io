<a name="neivc"></a>
# Mysql-Innodb-架构图
![image.png](/image/mysql-innodb-architecuture-0.png)
<a name="az3kl"></a>
## MySQL
:::info
下面介绍时，强烈建议回过来看图进行理解，回过来看图进行理解，回过来看图进行理解。
:::
<a name="k951s"></a>
### 连接器
Mysql作为服务器，一个客户端的Sql连接过来就需要分配一个线程进行处理，这个线程会专门负责监听请求并读取数据。这部分的线程和连接管理都是有一个连接器，专门负责跟客户端建立连接、权限认证、维持和管理连接。

可以通过`show processlist`来查看有哪些连接。
```sql
mysql> show processlist;
+----+------+-----------+-------+---------+------+----------+------------------+
| Id | User | Host      | db    | Command | Time | State    | Info             |
+----+------+-----------+-------+---------+------+----------+------------------+
| 11 | root | localhost | study | Query   |    0 | starting | show processlist |
+----+------+-----------+-------+---------+------+----------+------------------+
```
<a name="Wrp4S"></a>
### 查询缓存
就是根据SELECT语句来进行缓存，这个查询缓存的功能在MySQL8.0被移除了。

经典的设计，应该利用memcached或redis来缓存。
<a name="Tu2yE"></a>
### SQL解析器
先进行词法分析，将输入的SQL语句，分解为一个个token。<br />再进行语法分析，根据语法规则，最终转化为解析树。
<a name="OBgyh"></a>
### SQL优化器
然后对SQL解析器解析出来的解析树进行各种优化，包括重写查询、决定表的读取顺序，以及选择合适的索引等。
<a name="Vii5M"></a>
### 执行器
就是去存储引擎中去执行了。
<a name="qzNwE"></a>
## InnoDB
<a name="hBo08"></a>
### 读
<a name="YoNGn"></a>
#### 磁盘加载数据
![image.png](/image/mysql-innodb-architecuture-1.png)
<br />计算机局部性原理：是指CPU访问存储器时，无论是存取指令还是存取数据，所访问的存储单元都趋于聚集在一个较小的连续区域中。<br />从磁盘加载数据时，会把相邻的数据也一同加载进来，下次用到就不用去磁盘拿数据了。

磁盘加载数据到Buffer Pool中。
<a name="MCHAI"></a>
#### 自适应哈希索引
![image.png](/image/mysql-innodb-architecuture-2.png)

自适应哈希索引，是InnoDB引擎的内部的，工作原理就是将热点数据建立为哈希表，时间复杂度为O(1)，能够更快速的找到数据。

参数`innodb_adaptive_hash_index`来控制是否开启，默认是开启的。
```sql
mysql> show variables like 'innodb_adaptive_hash_index';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_adaptive_hash_index | ON    |
+----------------------------+-------+
1 row in set (0.01 sec)
```

<a name="XKfgO"></a>
### 写
<a name="L2G4E"></a>
#### 第①步 Undo日志缓冲：Undo Buffer
![image.png](/image/mysql-innodb-architecuture-3.png)<br />第①步先写入Undo Buffer中，然后Undo Buffer刷新到磁盘中（默认系统表空间ibdata1，也可指定Undo表空间）。<br />用途：事务回滚（rollback）、MVCC。

MVCC用于提高读写并发，很好的提升了性能，阿里集团Tair中间件Tair进行参考了这个将字符串做了版本号，有效提高了性能。
<a name="ofs7X"></a>
#### 第②步 Redo日志缓冲：Redo Log Buffer
![image.png](/image/mysql-innodb-architecuture-4.png)<br />第②步写入Redo Log Buffer，然后刷新到磁盘（ib_logfile0 ib_logfile1）中。<br />用途：当数据库意外宕机后，可根据redo日志进行数据恢复，提高数据可用性。

由于redo log文件打开并没有使用O_DIRECT，因此重做日志缓冲先写入文件系统缓存，为了确保redo log写入磁盘，必须进行一次fsync操作。<br />innodb_flush_log_at_trx_commit 参数用来控制重做日志刷新到磁盘的策略。

参数默认为1，意思是每次写都会刷盘操作。
```sql
mysql> show variables like 'innodb_flush_log_at_trx_commit';
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| innodb_flush_log_at_trx_commit | 1     |
+--------------------------------+-------+
1 row in set (0.00 sec)
```

相关知识点：redo log都是以512字节进行存储的，称为一个块。由于redo log块的大小和磁盘扇区一样大小，都是512字节。因此redo log从内存刷到磁盘，可以保证原子性。
<a name="IDQuC"></a>
#### 第③步 变更缓冲：Change Buffer
![image.png](/image/mysql-innodb-architecuture-5.png)<br />第③步写入Change Buffer，然后Merge到索引上。InnoDB的插入缓冲也是包含在这里的。<br />适用范围：仅用于非唯一索引。因为如果是唯一索引的话还要去Buffer Pool中进行查看是否唯一，就失去了意义。<br />用途：将多次写，合并为一次，提升插入效率，
<a name="BFDnw"></a>
#### 第④步 双写：Double Write Buffer
![image.png](/image/mysql-innodb-architecuture-6.png)<br />第④步双写<br />原因：缓冲区Buffer Pool与磁盘交互最小单元为4KB，但是InnoDB中最小的单位是16KB，直接往磁盘独立表空间上刷数据时，可能存在刷一部分就意外宕机了，这部分数据是不完整的，无法被识别的。<br />工作机制：先把Buffer Pool中的数据刷到系统表空间ibdata1中，然后再往独立表空间上刷数据，若刷的过程中意外宕机了，那么可以从系统表空间ibdata1中进行数据恢复，提高了数据可靠性。
<a name="pIHsO"></a>
### 磁盘
这里的MySQL数据目录是：`/path/to/mysql/data`
```sql
mysql> show variables like 'datadir';
+---------------+---------------------+
| Variable_name | Value               |
+---------------+---------------------+
| datadir       | /path/to/mysql/data |
+---------------+---------------------+
1 row in set (0.01 sec)
```
<a name="Ctmoc"></a>
#### 系统表空间：System Tablespace
![image.png](/image/mysql-innodb-architecuture-7.png)<br />通过参数innodb_data_file_path来进行控制的。
```sql
mysql> show variables like 'innodb_data_file_path';
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:12M:autoextend |
+-----------------------+------------------------+
1 row in set (0.00 sec)
```
这里说下这个value的含义，系统表空间名字为ibdata1，初始值12M，大小自动增长，每次增长幅度是`innodb_autoextend_increment`控制的。<br />对应文件存在MySQL数据目录下，即：`/path/to/mysql/data/ibdata1`。

系统表空间包含Double Write缓冲、Undo 日志、Change 缓冲、InnoDB 数据字典。
<a name="raSnX"></a>
#### 独立表空间：Table Tablespaces
![image.png](/image/mysql-innodb-architecuture-8.png)<br />通过参数innodb_file_per_table来进行控制的。
```sql

mysql> show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.00 sec)
```

既然表空间确定是独立的了，那么它会存在MySQL的数据目录下，然后是库目录，就能找到对应表名开头，ibd结尾的文件了。<br />这里的MySQL数据目录是：`/path/to/mysql/data`<br />这里的库目录是：`/path/to/mysql/data/test`<br />这里的独立表空间文件是：`/path/to/mysql/data/test/t1.ibd`
```sql
mysql> USE test; -- 用test库

mysql> CREATE TABLE t1 (
   id INT PRIMARY KEY AUTO_INCREMENT,
   name VARCHAR(100)
 ) ENGINE = InnoDB;

$> cd /path/to/mysql/data/test
$> ls
t1.ibd
```
<a name="DjiJm"></a>
#### 通用表空间：General Tablespaces
![image.png](/image/mysql-innodb-architecuture-9.png)<br />通过 `CREATE TABLESPACE tablespace_name ADD DATAFILE 'file_name'`来添加。
```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;
```
存放的位置在MySQL数据目录下：`/path/to/mysql/data/ts1.ibd`

<a name="gk0Rr"></a>
#### undo 表空间：Undo Tablespaces
![image.png](/image/mysql-innodb-architecuture-10.png)<br />默认存放在系统表空间ibdata1中。

也可以通过参数进行开启，默认未开启。后面这个功能参数会被废弃掉。
```sql
mysql> show variables like 'innodb_undo_tablespaces';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_undo_tablespaces | 0     |
+-------------------------+-------+
1 row in set (0.00 sec)
```

<a name="v1KGX"></a>
#### 临时表空间：The Temporary Tablespace
![image.png](/image/mysql-innodb-architecuture-11.png)
> 存储的是临时表的相关信息，例如大数据量排序时会生成临时表。

通过这个参数innodb_temp_data_file_path来控制。
```sql
mysql> show variables like 'innodb_temp_data_file_path';
+----------------------------+-----------------------+
| Variable_name              | Value                 |
+----------------------------+-----------------------+
| innodb_temp_data_file_path | ibtmp1:12M:autoextend |
+----------------------------+-----------------------+
1 row in set (0.00 sec)
```
这里说下这个value的含义，临时表空间名字为ibtmp1，初始值12M，大小自动增长。

对应文件存在MySQL数据目录下，即：`/path/to/mysql/data/ibtmp1`。
<a name="QV8cz"></a>
#### redo 日志
![image.png](/image/mysql-innodb-architecuture-12.png)<br />默认情况下会有两个文件，存放在MySQL数据目录下。
```sql
bash# ls -lh /path/to/mysql/data/ib_logfile*
-rw-r----- 1 mysql mysql 48M Mar 24 09:32 /path/to/mysql/data/ib_logfile0
-rw-r----- 1 mysql mysql 48M Mar 10 07:34 /path/to/mysql/data/ib_logfile1
```
这两个文件ib_logfile0和ib_logfile1构成一个redo 日志文件组。

工作机制是：InnoDB存储引擎先写ib_logfile0文件，当写满后，会切换至ib_logfile1文件，ib_logfile1文件写满后，会再切换至ib_logfile0中写。
<a name="EfuWj"></a>
# 参考资料
[https://www.geeksforgeeks.org/architecture-of-mysql/](https://www.geeksforgeeks.org/architecture-of-mysql/)<br />[https://dev.mysql.com/doc/refman/5.7/en/innodb-architecture.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-architecture.html)<br />[https://b23.tv/4SJlfQs](https://b23.tv/4SJlfQs)<br />《高性能MySQL》第四版

