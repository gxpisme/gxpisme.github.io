## 博客

> 体系化的整理自己的知识

## 大图

```

整体的思考是这样的。

首先从架构入手，通过了解架构的演进，就对整体知识框架有了认知；

然后以请求经过的先后顺序来看，分别经过了接入层、应用程序、缓存、消息队列、检索、数据库。虽然并不是每个请求都经过，但是大部分请求都会经过这些节点。

最后针对整个系统而言会有监控、日志。这些所有的，都是建立在基础（数据结构&算法&计算机网络）之上的。



对于这些各个领域（接入层、应用程序、消息队列、检索、数据库）来讲，以一种什么样的思路进行剖开呢。

我们大部分时候都是知道这些知识点，但是也需要将这些知识点，组织起来，形成知识体系。

就按照背景、目的、操作、核心来解构这个领域的知识。

先来看 背景、目的、操作 这三个， 思路就是：针对什么样的背景，要达到什么目的，采取了什么操作。

再来看 背景、目的、核心 这三个， 思路就是：针对什么样的背景，要达到什么目的，核心的知识体系是什么？

```

<img style=" width: 60%; " src="https://gxpisme.github.io/image/knowledge.png" />


## 架构
- [架构的演变历程](/architecture-history)
- [架构的模式](/architecture-pattern)

## 接入层
- [网关（接入层）介绍](/gateway-introduce)
- [（Domain Name System）DNS](/gateway-dns)

## 应用程序-php
 - [PHP 从CGI到PHP-FPM](php-cgi)
 - [PHP current、end函数遇到的坑](php-current-end)
 - [PHP的生命周期](php-life-cycle)
 - [PHP的编译](php-token)
 - [PHP5.6的变量 zval 结构](php-zval)
 - [PHP5.6环境搭建 ](php-build-enviorment)
 - [PHP HashTable](php-hashtable)
 - [PHP7.2之HashTable](php7-hashtable)
 - [PHP7.2的变量 zval 结构](php7-2-zval)
 - [PHP7 扩展开发 hello world](php7-ext)
 - [php7 垃圾回收](php7-garbage-collection)
 - [PHP7 内存管理 （一）](php7-memory-manage-1)
 - [PHP7 内存管理 （二）](php7-memory-manage-2)

## 应用程序-Java
 - [JAVA-内存管理](java-memory-manage)
 - [JAVA-内存模型-JMM](java-memory-model)
 - [JAVA-垃圾回收](java-garbage-collector)
 - [JAVA-垃圾回收-三色标记](java-garbage-tri-color)
 - [JAVA-类加载](java-classloader)
 - [JAVA-class类文件结构](/mynote/java_class_struct)
 - [JAVA-切面 AOP](java-aop)
 - [JAVA-动态代理 AOP原理](java-dynamic-proxy)
 - [JAVA-单例](java-instance)
 - [JAVA-锁](java-locks)
 - [JAVA-锁-Synchronized](java-synchronized)
 - [JAVA-锁-ReentrantLock](java-reentrantlock)
 - [JAVA-Metaspace](java-metaspace)
 
### 应用程序-Java之Spring
 - [Spring-注册Bean](spring-register-bean)

## 缓存-redis
 - [关于Redis的思考](redis-think)
 - [缓存更新 Cache Aside Pattern](cache-aside-pattern)

## 数据库-MySql
 - [MySQL-InnoDB-架构图](mysql-innodb-architecuture)
 - [MySQL主从服务搭建](mysql-build-master-slave)
 - [MySQL主从复制](mysql-replication-theory)
 - [MySQL-InnoDB插入缓冲](mysql-innodb-insert-buffer)
 - [MySQL-InnoDB 行锁](mysql-innodb-row-lock)
 - [MySQL 慢SQL](mysql-slow-log)
 - [MySQL-Isolation 隔离级别](mysql-isolation)

## 计算机基础
 - [ip](ip)
 - [cpu高速缓存](cpu-cache)

## Linux
- [awk](https://mp.weixin.qq.com/s/MqZf80E6my3TUhYN6YgPIg)

## 学习笔记
 - [技术学习笔记](note)
