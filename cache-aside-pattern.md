## 缓存更新 Cache Aside Pattern





### 问题1 更新操作  先删缓存，在更新数据库


### 问题2 更新操作 不是删除缓存，而是设置缓存


### 问题3 更新操作 设置数据库成功，设置缓存失败


### 问题4 读取操作 主从复制延迟，读到的旧数据


参考资料

- [缓存更新的套路](https://coolshell.cn/articles/17416.html)

- [究竟先操作缓存，还是数据库？](https://mp.weixin.qq.com/s/CuwTRC8HrMHxWZe3_OX98g)

- [Cache Aside Pattern](https://mp.weixin.qq.com/s/7IgtwzGC0i7Qh9iTk99Bww)
