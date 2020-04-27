---
title: MySQL主从服务搭建
date: 2015-05-26
tags:
    - MySQL
categories: 数据库
---


## 配置一主一从
> 主(Master)：192.168.56.102
>
> 从(Slave)：192.168.56.101


## 主(Master)：配置文件 `/etc/my.cnf`
```
[mysqld]
# 声明二进制日志文件为mysql-bin.XXXXXX

log-bin=mysql-bin

#二进制格式有三个row/statement/mixed
#row       记录磁盘改变  适合 执行语句长，改变小
#statement 记录执行语句 适合 执行语句短 磁盘改变多
#mixed     row + statement混合

binlog_format=mixed

#给mysql服务一个独特的id，通常为局域网的ip最后一段值

server_id   = 102
```


## 从(Slave)：配置文件 `/etc/my.cnf`
```
[mysqld]
log-bin=mysql-bin
binlog_format=mixed
server_id   = 101
#得到主的bin-log分析后成relay-log才能被自己所用
relay-log=mysql-relay
```

## 账号设置
> 主(Master)服务器设置账号
>
> 从(Slave)服务器凭借账号去读主(Master)服务器的bin-log日志
>
> 从(Slave)服务器上注意：master_log_file和master_log_pos这两个选项
>
> 这两个选项应该和show master status;出来的结果一样(因为从哪个文件，哪个位置开始读)

主(Master)：
```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000015 |      547 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

在 主(Master)设置账号
```
mysql> grant replication client,replication slave on *.* to xp@'192.168.56.%' identified by 'xpisme';
mysql> flush privileges;
```

在从(Slave)上操作 从(Slave)关联到主(Master)服务器
```
mysql> change master to
master_host='192.168.56.102',
master_user='xp',
master_password='xpisme',
master_log_file='mysql-bin.000015',
master_log_pos=547;
```

在从(Slave)上操作
```
mysql> start slave;
```

**搞定**

[linux-installation-yum](https://dev.mysql.com/doc/refman/5.6/en/linux-installation-yum-repo.html)
[change-master-to](https://dev.mysql.com/doc/refman/5.6/en/change-master-to.html)
