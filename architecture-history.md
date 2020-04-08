# 架构的演变历程

## 单机服务器
> 背景: 用户量很少，基本上没有人访问，单机服务器显然够用。
>
> 目的: 就是让用户够用就行。
>
> 操作：搭建一台服务器，上面部署了应用程序，数据库，文件。

![](/image/architecture-history-1.png)

## 三台服务器
> 背景: 随着业务的发展，单机服务器扛不住了，性能越来越差，越来越多的文件占磁盘越来越大。
>
> 目的: 就是让用户可用，能够满足用户的需求。
>
> 操作：搭建三台服务器，分别是应用服务器，数据库服务器，文件服务器。

- 应用服务器占用CPU较多，应该选择CPU偏好的机器。
- 数据库服务器占用内存磁盘较多，应该选用内存磁盘偏好的机器。
- 文件服务器占用磁盘较多，应该选用磁盘较大的机器。

![](/image/architecture-history-2.png)


## 引入缓存
> 背景: 网站访问的特点，大部分流量都会落到部分数据上，这些数据称为热点数据。用户访问量的增加，使得性能也越来越差，数据库扛不住了。
>
> 目的: 就是让用户可用，能够满足用户的需求。
>
> 操作：搭建缓存服务器，将热点数据放入缓存服务器中，从而降级数据库服务器的负载。 缓存服务器可以是分布式的，这样就不用受到容量的限制了。

- 也可以有些本地缓存的情况，本地缓存更快。但其容量严格会受到应用服务器的限制。
- 最佳实践就是能用分布式缓存，尽量用分布式缓存。本地缓存除非有特别的场景才用。缓存越多，数据不一致的情况越严重。

![](/image/architecture-history-3.png)
![](/image/architecture-history-4.png)
![](/image/architecture-history-5.png)
![](/image/architecture-history-6.png)
![](/image/architecture-history-7.png)
![](/image/architecture-history-8.png)
![](/image/architecture-history-9.png)
![](/image/architecture-history-10.png)
