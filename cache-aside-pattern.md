# 缓存更新 Cache Aside Pattern





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

### 问题3 更新操作 设置数据库成功，设置缓存失败


### 问题4 读取操作 主从复制延迟，读到的旧数据


参考资料

- [缓存更新的套路](https://coolshell.cn/articles/17416.html)

- [究竟先操作缓存，还是数据库？](https://mp.weixin.qq.com/s/CuwTRC8HrMHxWZe3_OX98g)

- [Cache Aside Pattern](https://mp.weixin.qq.com/s/7IgtwzGC0i7Qh9iTk99Bww)
