```java
-XX:MetaspaceSize=512m
-XX:MaxMetaspaceSize=512m Metaspace 总空间的最大允许使用内存，默认是不限制。
-XX:CompressedClassSpaceSize=1G Metaspace 中的 Compressed Class Space 的最大允许内存，默认值是 1G，这部分会在 JVM 启动的时候向操作系统申请 1G 的虚拟地址映射，但不是真的就用了操作系统的 1G 内存。
```

<br />

### 分配metaspace空间
![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296255350-b6beb0f7-3f3d-4802-b6bb-246a6ff17144.png#clientId=ucef1c819-9448-4&from=paste&height=364&id=u432f6b59&margin=%5Bobject%20Object%5D&name=image.png&originHeight=728&originWidth=1768&originalType=binary&ratio=1&size=174469&status=done&style=none&taskId=ua3659a7e-9640-44d6-89cc-8e723f0c6db&width=884)
### 回收metaspace空间
![](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296298039-2cd49dca-d902-41f4-a6cb-87322ebebb96.png#clientId=ucef1c819-9448-4&from=paste&height=465&id=u2dfab26f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=930&originWidth=1882&originalType=binary&ratio=1&size=351427&status=done&style=none&taskId=u28e984dc-5e9b-4764-943b-13b85eabb3a&width=941)<br />

### metaspace与GC

- 分配空间时
   - 虚拟机维护了一个阈值，如果 Metaspace 的空间大小超过了这个阈值，那么在新的空间分配申请时，虚拟机首先会通过收集可以卸载的类加载器来达到复用空间的目的，而不是扩大 Metaspace 的空间，这个时候会触发 GC。这个阈值会上下调整，和 Metaspace 已经占用的操作系统内存保持一个距离。
- 碰到 Metaspace OOM
   - Metaspace 的总使用空间达到了 MaxMetaspaceSize 设置的阈值，或者 Compressed Class Space 被使用光了，如果这次 GC 真的通过卸载类加载器腾出了很多的空间，这很好，否则的话，我们会进入一个糟糕的 GC 周期，即使我们有足够的堆内存。​



## 内存分配
### VirtualSpaceList
一个 Node 是 2MB 的空间，这里的 2MB 不是真的就消耗了主存的 2MB，只有之后在使用的时候才会真的消耗内存。这里是虚拟内存映射。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296792574-5d1d31c9-bb82-42e4-9186-0e292969ec8d.png#clientId=u6825509a-c1b7-4&from=paste&height=577&id=u038cc23b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1154&originWidth=2080&originalType=binary&ratio=1&size=129896&status=done&style=none&taskId=u4c9ea597-e7cb-40e3-9eeb-43c5aef1504&width=1040)
### Chunk
从一个 Node 中分配内存，每一块称为 MetaChunk，chunk 有三种规格，在 64 位系统中分别为 1K、4K、64K。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296913208-61720658-786d-4824-bfce-5c074d37ca9e.png#clientId=u6825509a-c1b7-4&from=paste&height=247&id=ub57aaae1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=494&originWidth=2186&originalType=binary&ratio=1&size=50460&status=done&style=none&taskId=uf82936d1-627e-430e-8a5f-c6302927d5d&width=1093)<br />

- 通常，一个标准的类加载器在第一次申请空间时，会得到一个 4K 的 chunk，直到它达到了一个随意设置的阈值，此时分配器失去了耐心，之后会一次性给它一个 64K 的大 chunk。
- bootstrap classloader 是一个公认的会加载大量的类的加载器，所以分配器会给它一个巨大的 chunk，一开始就会给它 4M。可以通过 InitialBootClassLoaderMetaspaceSize 进行调优。
- 反射类类加载器 (jdk.internal.reflect.DelegatingClassLoader) 和匿名类类加载器只会加载一个类，所以一开始只会给它们一个非常小的 chunk（1K），因为给它们太多就是一种浪费。
### Block
在 Metachunk 上，我们有一个二级分配器（class-loader-local allocator），它将一个 Metachunk 分割成一个个小的单元，这些小的单元称为 Metablock，它们是实际分配给每个调用者的。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632297016413-69e5d847-86b4-4487-bd12-e6e6d4e838c8.png#clientId=u6825509a-c1b7-4&from=paste&height=467&id=u22ba32fd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=934&originWidth=1972&originalType=binary&ratio=1&size=100160&status=done&style=none&taskId=u47d40f58-c69b-46ec-95ae-5d38ed5b1be&width=986)<br />

# 参考资料
[https://www.javadoop.com/post/metaspace](https://www.javadoop.com/post/metaspace)
