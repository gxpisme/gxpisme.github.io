<a name="neivc"></a>
# Mysql-Innodb-架构图
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679899660187-3bd9a629-ec16-4a63-8b94-5c46a8ef6b7f.png#averageHue=%23f0f0ef&clientId=ucd3092d5-0c1e-4&from=paste&height=734&id=u0d33c021&name=image.png&originHeight=1468&originWidth=2530&originalType=binary&ratio=2&rotation=0&showTitle=false&size=760384&status=done&style=none&taskId=u58aedfc1-20e0-482f-837f-720d3e1b288&title=&width=1265)
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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679665356155-366b4e53-ce0a-4d0d-a537-45ed684da6da.png#averageHue=%23e6e6e6&clientId=u6068c053-791c-4&from=paste&height=130&id=u1d8074d9&name=image.png&originHeight=260&originWidth=624&originalType=binary&ratio=2&rotation=0&showTitle=false&size=41124&status=done&style=none&taskId=u01f95316-20d6-4325-ba3c-a00e10d56dc&title=&width=312)<br />计算机局部性原理：是指CPU访问存储器时，无论是存取指令还是存取数据，所访问的存储单元都趋于聚集在一个较小的连续区域中。<br />从磁盘加载数据时，会把相邻的数据也一同加载进来，下次用到就不用去磁盘拿数据了。

磁盘加载数据到Buffer Pool中。
<a name="MCHAI"></a>
#### 自适应哈希索引
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679665398195-eb231731-7812-47c0-9d4a-60433555261b.png#averageHue=%237da851&clientId=u6068c053-791c-4&from=paste&height=223&id=ua7d3917c&name=image.png&originHeight=446&originWidth=594&originalType=binary&ratio=2&rotation=0&showTitle=false&size=66175&status=done&style=none&taskId=u40fab5a9-22da-47a4-8ff3-2b952c81249&title=&width=297)

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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679663994293-520a223b-136b-4ed6-a7e4-c5fc39eb014e.png#averageHue=%23ebeae9&clientId=u6068c053-791c-4&from=paste&height=199&id=u45253922&name=image.png&originHeight=398&originWidth=760&originalType=binary&ratio=2&rotation=0&showTitle=false&size=166458&status=done&style=none&taskId=u0d4548c3-d90c-4b2d-9e70-2258e9bf4a0&title=&width=380)<br />第①步先写入Undo Buffer中，然后Undo Buffer刷新到磁盘中（默认系统表空间ibdata1，也可指定Undo表空间）。<br />用途：事务回滚（rollback）、MVCC。

MVCC用于提高读写并发，很好的提升了性能，阿里集团Tair中间件Tair进行参考了这个将字符串做了版本号，有效提高了性能。
<a name="ofs7X"></a>
#### 第②步 Redo日志缓冲：Redo Log Buffer
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679663448708-164a0cb9-fbee-4efa-bc81-ff8e64d7766d.png#averageHue=%23ecebe6&clientId=u6068c053-791c-4&from=paste&height=130&id=u997a43d8&name=image.png&originHeight=260&originWidth=864&originalType=binary&ratio=2&rotation=0&showTitle=false&size=103490&status=done&style=none&taskId=u8556bc14-f4f1-4237-8f57-1073360f42c&title=&width=432)<br />第②步写入Redo Log Buffer，然后刷新到磁盘（ib_logfile0 ib_logfile1）中。<br />用途：当数据库意外宕机后，可根据redo日志进行数据恢复，提高数据可用性。

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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679664557967-5d008ac4-cc59-413c-b933-b5b8928b83ee.png#averageHue=%23536755&clientId=u6068c053-791c-4&from=paste&height=270&id=u4b8162fc&name=image.png&originHeight=540&originWidth=778&originalType=binary&ratio=2&rotation=0&showTitle=false&size=253837&status=done&style=none&taskId=ub8156b7b-174f-420d-8826-150ea9242a2&title=&width=389)<br />第③步写入Change Buffer，然后Merge到索引上。InnoDB的插入缓冲也是包含在这里的。<br />适用范围：仅用于非唯一索引。因为如果是唯一索引的话还要去Buffer Pool中进行查看是否唯一，就失去了意义。<br />用途：将多次写，合并为一次，提升插入效率，
<a name="BFDnw"></a>
#### 第④步 双写：Double Write Buffer
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679664807906-2b377322-1513-42fb-8f88-51e540bbb7e8.png#averageHue=%23e4e4e4&clientId=u6068c053-791c-4&from=paste&height=295&id=u417a6819&name=image.png&originHeight=590&originWidth=612&originalType=binary&ratio=2&rotation=0&showTitle=false&size=90960&status=done&style=none&taskId=u5b79ebf1-75ae-42e5-8062-75a03f053ff&title=&width=306)<br />第④步双写<br />原因：缓冲区Buffer Pool与磁盘交互最小单元为4KB，但是InnoDB中最小的单位是16KB，直接往磁盘独立表空间上刷数据时，可能存在刷一部分就意外宕机了，这部分数据是不完整的，无法被识别的。<br />工作机制：先把Buffer Pool中的数据刷到系统表空间ibdata1中，然后再往独立表空间上刷数据，若刷的过程中意外宕机了，那么可以从系统表空间ibdata1中进行数据恢复，提高了数据可靠性。
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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679662904081-3f4124ee-4f92-4309-a012-94453102de69.png#averageHue=%23b3b7c1&clientId=u6068c053-791c-4&from=paste&height=174&id=ue94e5661&name=image.png&originHeight=348&originWidth=258&originalType=binary&ratio=2&rotation=0&showTitle=false&size=34368&status=done&style=none&taskId=u1c7bedf9-2107-4f26-984d-f70138abe99&title=&width=129)<br />通过参数innodb_data_file_path来进行控制的。
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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679662894721-3891d00f-b3a3-4287-ae20-c6db2f053aa0.png#averageHue=%23cfcfcf&clientId=u6068c053-791c-4&from=paste&height=136&id=u0f5607bc&name=image.png&originHeight=272&originWidth=230&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22319&status=done&style=none&taskId=u74754b9f-3a56-4a89-8e99-121b57fbf60&title=&width=115)<br />通过参数innodb_file_per_table来进行控制的。
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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679662883765-34b070b6-0fa4-437e-8999-8682088606d3.png#averageHue=%23d6d6d6&clientId=u6068c053-791c-4&from=paste&height=162&id=uf7fe3eda&name=image.png&originHeight=324&originWidth=262&originalType=binary&ratio=2&rotation=0&showTitle=false&size=28354&status=done&style=none&taskId=ua707978f-348e-4286-8361-0e7241ede44&title=&width=131)<br />通过 `CREATE TABLESPACE tablespace_name ADD DATAFILE 'file_name'`来添加。
```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;
```
存放的位置在MySQL数据目录下：`/path/to/mysql/data/ts1.ibd`

<a name="gk0Rr"></a>
#### undo 表空间：Undo Tablespaces
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679662873175-c927d0e1-8a8b-4ac7-97b4-ce4299e22c57.png#averageHue=%23d8d8d8&clientId=u6068c053-791c-4&from=paste&height=283&id=ud7dbb28a&name=image.png&originHeight=566&originWidth=310&originalType=binary&ratio=2&rotation=0&showTitle=false&size=55139&status=done&style=none&taskId=u2d605da0-ae87-40d6-8881-c4d52c4942f&title=&width=155)<br />默认存放在系统表空间ibdata1中。

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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679662860379-2240080c-0772-46fa-bf03-2d1e62652c1d.png#averageHue=%23cfcfcf&clientId=u6068c053-791c-4&from=paste&height=145&id=u606d2195&name=image.png&originHeight=290&originWidth=216&originalType=binary&ratio=2&rotation=0&showTitle=false&size=21110&status=done&style=none&taskId=u82b1248a-8623-4f2f-83dc-3d7e508815d&title=&width=108)
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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/546024/1679661726299-15461305-ce7e-4a20-8bfb-f6f5ca619bb9.png#averageHue=%23e6e5e4&clientId=u6068c053-791c-4&from=paste&height=140&id=u440f91b3&name=image.png&originHeight=280&originWidth=340&originalType=binary&ratio=2&rotation=0&showTitle=false&size=47093&status=done&style=none&taskId=u95f8da90-954c-45d8-be71-ea317794791&title=&width=170)<br />默认情况下会有两个文件，存放在MySQL数据目录下。
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

