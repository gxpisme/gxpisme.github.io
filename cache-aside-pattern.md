# 缓存更新 Cache Aside Pattern

## 读`read` 两种情况
![](image/Cache-Aside-Design-Pattern-Flow-Diagram-e1470471723210.png)
###  第一种 `cache hit` 缓存命中
应用程序从cache中取数据，取到后返回。
### 第二种 `cache miss` 缓存失效
应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。

## 更新`update`
![](image/Updating-Data-using-the-Cache-Aside-Pattern-Flow-Diagram-1-e1470471761402.png)

先把数据存到数据库中，成功后，再让缓存失效。


## 问题
### 问题1 为什么是 先更新数据库再删缓存？

假设有两个并发操作，一个是更新`update`操作，一个读`read`操作。

此时 缓存和数据库中的数据均为`num=1`，更新操作将设置`num=2`。

下面具体看并发操作顺序

1. `update` 先删除了缓存。  此时情况为：`缓存 empty ; 数据库 num=1`
2. `read` 读进来，缓存未命中。 此时情况为：`缓存 empty ; 数据库 num=1`
3. `read` 还是这个读操作，读到了数据库 `num=1`。此时情况为：`缓存 empty ; 数据库 num=1`。
4. `update` 更新数据库，操作结束。 此时情况为：`缓存 empty ; 数据库 num=2`
5. `read` 还是读操作，由于刚才读到的数据库`num=1`，所以设置缓存为`num=2`，此时情况为：`缓存 empty ; 数据库 num=2`。

上面这几个顺序不是固定的，只要保证两点
1. `read`在`update`之后，也就保证了，`read`读不到缓存，必须经过数据库。
2. `read`操作读到的数据库数据是还未更新的（可能是update还没更新，也可能是主从延迟），就会出现数据不一致。

### 问题2 为什么是 是删除`del`缓存而不是设置`set`缓存？

假设有两个并发更新`update`操作，先假设为`A update` 和 `B update`

此时 缓存和数据库中的数据均为`num=1`，`A update`更新操作将设置`num=2`，`B update`更新操作将设置`num=3`。

下面具体看并发操作顺序

1. `A update` 更新数据库 `num=2`。 此时情况为：`缓存 num=1 ; 数据库 num=2`。
2. `B update` 另一个并发写请求，更新数据库 `num=3`。此时情况为：`缓存 num=1 ; 数据库 num=3`。
3. `B update` 这个请求写缓存 `num=3`。 此时情况为：`缓存 num=3 ; 数据库 num=3`。
4. `A update` 最开始的请求写缓存 `num=2`。此时情况为：`缓存 num=2 ; 数据库 num=3`。

缓存和数据库数据不一致。如果是`del`缓存，就不会出现这样的问题。

### 问题3 如果`update`更新操作 发生了设置数据库成功，删除`del`缓存失败。
这种肯定会出现的，可能是网络抖动，应用服务器连接缓存服务器失败、超时等等。

其实如果连接数据库服务器失败的话，是没有问题的，因为数据库和缓存都没有变化，还是一致的。

为了保证**删除缓存**这个信息传递到缓存服务器，我们可以通过中间件`databus`等，监听`MySQL bin log`的方式，一旦`MySQL bin log`数据有变动，则发出消息，推送到缓存服务器，让缓存服务器删除对应的数据。使用了消息队列，可以保证消息必达。

最佳的方式
- 给缓存设置有效期。
- 删除缓存失败 重试一次，换机器重试等。

### 问题4 读取操作 主从复制延迟，读到的旧数据

这个就是发生的概率比较大，因为数据库一般都是主从同步的架构，并且存在一定的延迟，还有就是`read`操作一般都是落在`从库`上。

解决方法
- 读主库。
- 给缓存设置有效期。
- 通过中间件，列如上面的`databus`监听从库的`bin log`消息，再去触发删除缓存操作。


### 问题5 Cache Aside pattern 并发问题，我觉得是不太容易理解。

假设有两个并发操作，一个是更新`update`操作，一个读`read`操作。

此时 缓存中没有数据，数据库中的数据为`num=1`，更新操作将设置`num=2`。

下面具体看并发操作顺序
1. `read`读操作，`cache miss`没有命中缓存。此时情况为：`缓存 empty ; 数据库 num=1`。
2. 还是这个`read`读操作，读到数据库`num=1`。此时情况为：`缓存 empty ; 数据库 num=1`。
3. `update`更新操作，更新数据库`num=2`。   此时情况为：`缓存 empty ; 数据库 num=2`。
4. 还是这个`update`更新操作，让缓存失效了。  此时情况为：`缓存 empty ; 数据库 num=2`。
5. 还是这个`read`读操作，设置缓存为`num=1`，因为上面步骤2从数据库中读到的是1。此时情况为：`缓存 empty ; 数据库 num=2`。

此时，数据情况不一致，但是发生的概率较小。

这几个步骤，第1步和最后一步都是`read`操作，中间是`update`操作。

一般来讲，`update`操作要比`read`操作耗时长。


## 参考资料

- [缓存更新的套路](https://coolshell.cn/articles/17416.html)

- [究竟先操作缓存，还是数据库？](https://mp.weixin.qq.com/s/CuwTRC8HrMHxWZe3_OX98g)

- [Cache Aside Pattern](https://mp.weixin.qq.com/s/7IgtwzGC0i7Qh9iTk99Bww)