# 缓存更新 Cache Aside Pattern





## 问题
### 问题1 更新操作  先删缓存，在更新数据库

假设有两个并发操作，一个是更新`update`操作，一个读`read`操作，
此时 缓存中的数据为`num=1`，数据库中的数据为`num=1`。
更新操作设置`num=2`。

下面看具体看并发操作顺序

1. `update` 先删除了缓存。  此时情况为：`缓存 empty ; 数据库 num=1`
2. `read` 读进来，缓存未命中。 此时情况为：`缓存 empty ; 数据库 num=1`
3. `read` 还是这个读操作，读到了数据库 `num=1`。此时情况为：`缓存 empty ; 数据库 num=1`。
4. `update` 更新数据库，操作结束。 此时情况为：`缓存 empty ; 数据库 num=2`
5. `read` 还是读操作，由于刚才读到的数据库`num=1`，所以设置缓存为`num=2`，此时情况`缓存 empty ; 数据库 num=2`。


### 问题2 更新操作 不是删除缓存，而是设置缓存


### 问题3 更新操作 设置数据库成功，设置缓存失败


### 问题4 读取操作 主从复制延迟，读到的旧数据


参考资料

- [缓存更新的套路](https://coolshell.cn/articles/17416.html)

- [究竟先操作缓存，还是数据库？](https://mp.weixin.qq.com/s/CuwTRC8HrMHxWZe3_OX98g)

- [Cache Aside Pattern](https://mp.weixin.qq.com/s/7IgtwzGC0i7Qh9iTk99Bww)
