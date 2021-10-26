```java
-XX:MetaspaceSize=512m
-XX:MaxMetaspaceSize=512m Metaspace 总空间的最大允许使用内存，默认是不限制。
-XX:CompressedClassSpaceSize=1G Metaspace 中的 Compressed Class Space 的最大允许内存，默认值是 1G，这部分会在 JVM 启动的时候向操作系统申请 1G 的虚拟地址映射，但不是真的就用了操作系统的 1G 内存。
```

<br />

### 分配metaspace空间
![](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296255350-b6beb0f7-3f3d-4802-b6bb-246a6ff17144.png#clientId=ucef1c819-9448-4&from=paste&height=364&id=u432f6b59&margin=%5Bobject%20Object%5D&name=image.png&originHeight=728&originWidth=1768&originalType=binary&ratio=1&size=174469&status=done&style=none&taskId=ua3659a7e-9640-44d6-89cc-8e723f0c6db&width=884)
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


# MetaSpace 架构
> 深入到MetaSpace架构中，看看他们多层，多组件是如何组织协作的。

​

Metaspace分了三个层，最底层，中间层，最上层。<br />最底层：直接从操作系统中申请大块内存。<br />中间层：将内存块划分为小块Chunk给类加载器使用。<br />最上层：类加载器分开的Chunk，给调用代码使用。
## 最底层 VirtualSpaceList
在最底层，JVM 通过 mmap(3) 接口向操作系统申请内存映射，在 64 位平台上，每次申请 2MB 空间。<br />​<br />
> 当然，这里的 2MB 不是真的就消耗了主存的 2MB，只有之后在使用的时候才会真的消耗内存。这里是虚拟内存映射。

每次申请过来的内存区域，放到一个链表中 [VirtualSpaceList](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/memory/metaspace/virtualSpaceList.hpp#l39)，作为其中的一个 Node。看下图。<br />​

一个 Node 是 2MB 的空间，前面说了在使用的时候再向操作系统申请实际的内存，但是频繁的系统调用会降低性能，所以 Node 内部需要维护一个水位线，当 Node 内已使用内存快达到水位线的时候，向操作系统要新的内存页。并且相应地提高水位线。

直到一个 Node 被完全用完，会分配一个新的 Node，并且将新的Node加入到链表中，老的 Node 就 “退休” 了。下图中，前面的三个 Node 就是退休状态了。退休节点的剩余空间不会丢失，而是被分割成块并添加到全局自由列表中。<br />
<br />从一个 Node 中分配内存，每一块称为 MetaChunk，chunk 有三种规格，在 64 位系统中分别为 1K、4K、64K。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296792574-5d1d31c9-bb82-42e4-9186-0e292969ec8d.png#clientId=u6825509a-c1b7-4&from=paste&height=577&id=u038cc23b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1154&originWidth=2080&originalType=binary&ratio=1&size=129896&status=done&style=none&taskId=u4c9ea597-e7cb-40e3-9eeb-43c5aef1504&width=1040)<br />VirtualSpaceList 和它的Node节点是全局的结构，Metachunk归属于类加载器。因此在VirtualSpaceList中的Node节点经常会有不同的类加载器。如下图：一个Node中就有a、b、c、d四个类加载器。![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632296913208-61720658-786d-4824-bfce-5c074d37ca9e.png#clientId=u6825509a-c1b7-4&from=paste&height=247&id=ub57aaae1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=494&originWidth=2186&originalType=binary&ratio=1&size=50460&status=done&style=none&taskId=uf82936d1-627e-430e-8a5f-c6302927d5d&width=1093)<br />
<br />当一个类加载器它所有的类都被卸载后，被卸载的类加载器所对应的元数据信息被释放。所有空闲的chunk被添加到一个全局空闲列表中([ChunkManager](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/memory/metaspace/chunkManager.hpp#l44))<br />​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634808493712-1f846986-5c7f-4ad2-8617-a2a0e0e77990.png#clientId=u0b635656-98b2-4&from=paste&height=307&id=u24df8780&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=667&originalType=binary&ratio=1&size=23092&status=done&style=none&taskId=u06cac263-cdff-4637-ad71-7dea52fa139&width=500)<br />如果其他类加载器开始加载类，在申请Metaspace空间的时候，会用之前的空闲空间。<br />上图类加载器b对应的空间被释放，下图类加载器e和类加载器f就会使用类加载器b所释放的空间。<br />​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634808688137-9494b6ab-e79f-4d25-a728-f8f2a2156819.png#clientId=u0b635656-98b2-4&from=paste&height=295&id=uc67c5330&margin=%5Bobject%20Object%5D&name=image.png&originHeight=394&originWidth=667&originalType=binary&ratio=1&size=20968&status=done&style=none&taskId=ube5e5ff6-b995-4049-9714-f970ace74fb&width=500)

## 中间层 MetaChunk
类加载器从Metaspace空间申请内存（通常比较小，几十，几百bytes），按照200bytes来说吧。会得到一个Metachunk，比要申请内存大小大的多的一小块内存。

从VirtualSpaceList里申请内存需要加锁，所以非常昂贵。我们不想频繁这样申请，因此通常会申请到一个比较大的内存Metachunk ，能够更快满足未来内存的分配。并且可以不用加锁，并发的给其他类加载器分配内存。只有当chunk用完了，才会再次从VirtualSpaceList申请。<br />​

Metaspace内存分配给类加载器多大的内存呢？

- 通常，一个标准的类加载器在第一次申请空间时，会得到一个 4K 的 chunk，直到它达到了一个随意设置的阈值，此时分配器失去了耐心，之后会一次性给它一个 64K 的大 chunk。
- bootstrap classloader 是一个公认的会加载大量的类的加载器，所以分配器会给它一个巨大的 chunk，一开始就会给它 4M。可以通过 InitialBootClassLoaderMetaspaceSize 进行调优。
- 反射类类加载器 (jdk.internal.reflect.DelegatingClassLoader) 和匿名类类加载器只会加载一个类，所以一开始只会给它们一个非常小的 chunk（1K），因为给它们太多就是一种浪费。


<br />

## 最上层 MetaBlock
在 Metachunk 上，我们有一个分配器（class-loader-local allocator），它将一个 Metachunk 分割成一个个小的单元，这些小的单元称为 Metablock，它们是分配给每个调用者的。<br />(例如：一个Metablock包含一个InstanceKlass)<br />​

这个分配器（class-loader-local allocator）非常原始，它的速度快。<br />class metadata的生命周期是和类加载器绑定的，所以在类加载器卸载的时候，才去释放它占用的这些空间。<br />​

​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1632297016413-69e5d847-86b4-4487-bd12-e6e6d4e838c8.png#clientId=u6825509a-c1b7-4&from=paste&height=467&id=u22ba32fd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=934&originWidth=1972&originalType=binary&ratio=1&size=100160&status=done&style=none&taskId=u47d40f58-c69b-46ec-95ae-5d38ed5b1be&width=986)<br />
<br />这个 chunk 诞生的时候，它只包含header，之后的分配都只要在顶部进行分配就行。<br />​

由于这个 chunk 是归属于一个类加载器的，所以如果它不再加载新的类，那么 unused 空间就将真的浪费掉。

# ClassloaderData and ClassLoaderMetaspace
在 JVM 内部，类加载器是以 [ClassLoaderData](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/classfile/classLoaderData.hpp#l176) 结构标识的，这个结构引用了一个 [ClassLoaderMetaspace](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/memory/metaspace.hpp#l230)结构，它维护了该加载器使用的所有的 Metachunk。<br />​

当这个类加载器被卸载的时候，这个 ClassLoaderData 和 ClassLoaderMetaspace 会被删除。并且会将所有的这个加载器用到的 chunks 归还到空闲列表中。如果恰好把整个 Node 都清空了，那么这个 Node 的内存直接还给操作系统。
# 内存什么时候会还给操作系统
当一个 VirtualSpaceListNode 中的所有 chunk 都是空闲的时候，这个 Node 就会从链表 VirtualSpaceList 中移除，它的 chunks 也会从空闲列表中移除，这个 Node 就没有被使用了，会将其内存归还给操作系统。<br />
<br />对于一个空闲的 Node 来说，拥有其上面的 chunks 的所有的类加载器必然都是被卸载了的。<br />至于这个情况是否可能发生，主要就是取决于碎片化：<br />一个 Node 是 2M，chunks 的大小为 1K, 4K 或 64K，所以通常一个 Node 上有约 150-200 个 chunks，如果这些 chunks 全部由同一个类加载器拥有，回收这个类加载器就可以一次性回收这个 Node，并且把它的空间还给操作系统。<br />​

但是，如果这些 chunks 分配给不同的类加载器，每个类加载器都有不同的生命周期，那么什么都不会被释放。这也许就是在告诉我们，要小心对待大量的小的类加载器，如那些负责加载匿名类或反射类的加载器。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634818686700-0f1f2ae2-a4b0-46e6-9e27-111880471b25.png#clientId=u27838527-79a4-4&from=paste&id=u75cf1502&margin=%5Bobject%20Object%5D&name=image.png&originHeight=403&originWidth=638&originalType=binary&ratio=1&size=21243&status=done&style=none&taskId=u640967d7-888a-47ae-8050-aede5bffa6d)

- 每次向操作系统申请 2M 的虚拟空间映射，放置到全局链表中，待需要使用的时候申请内存。
- 一个 Node 会分割为一个个的 chunks，分配给类加载器，一个 chunk 属于一个类加载器。
- chunk 再细分为一个个 Metablock，这是分配给调用者的最小单元。
- 当一个类加载器被卸载，它占有的 chunks 会进入到空闲列表，以便复用，如果运气好的话，有可能会直接把内存归还给操作系统。

​<br />


# 对象类型指针 Class Metadata address
大家都知道，Java中实例对象都在Heap堆中存储，这些对象对应的类结构都存储在Metaspace空间中。<br />一个实例对象有一个指针指向Metaspace空间所对应的类结构。<br />​<br />
> 实例对象包含三部分：对象头、实例变量、填充数据。
> 对象头存储了到对象类型数据的指针。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634898917826-09f8102e-c61a-4175-9138-ea1a9cef79d5.png#clientId=ue8d1e6c6-b076-4&from=paste&height=266&id=u8714a9e3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=532&originWidth=1476&originalType=binary&ratio=1&size=217362&status=done&style=none&taskId=u6cf734ad-72f2-41e9-aa7d-606982b9f38&width=738)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634893906268-ce82e9d6-19d8-4907-83e3-38b2b0509012.png#clientId=u3b77b092-32d8-4&from=paste&id=oy3M4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=288&originWidth=322&originalType=binary&ratio=1&size=10778&status=done&style=none&taskId=u2d498f28-d56c-4d3d-bde4-1645f9f2365)<br />在64位系统上对象类型指针占用64bit，2^64bit能够代表2^34G内存的地址，在实际的机器上，并不需要那么多对象。采用对象压缩指针技术，就使用32bit，2^32bit能够代表4G内存的地址，在应用过程中，能够处理正常情况。这样对象能够占用更少的内存。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634894037330-cb8c4290-0b79-41aa-98c3-29ec08a2f250.png#clientId=u3b77b092-32d8-4&from=paste&id=AP9ow&margin=%5Bobject%20Object%5D&name=image.png&originHeight=293&originWidth=389&originalType=binary&ratio=1&size=14376&status=done&style=none&taskId=u9f405d51-9e0d-47cd-b190-8ccd6d275fb)<br />使用对象压缩指针技术，在Metaspace空间中存储的Klass结构，该内存需要是一块连续的内存，否则在内存大于4G的机器上，无法准备找到。若内存16G，指针能够表示的内存只有4G，肯定是满足不了的，为了解决这种情况，就分配一块连续小于4G的内存用来存储Klass结构。存储Klass结构的内存是连续的，从某个内存地址开始到某个内存地址结束。找到真正的Klass结构地址，需要加上一个base值。<br />​<br />
# Class Space 和 Non-Class Space
由于对象类型指针压缩技术，Metaspace需要一块连续的内存存储Klass结构。但是对应非Klass结构的元数据，则不需要一块联系的内存进行存储。<br />​

联想到之前提到的全局VirtualSpaceList结构和全局空闲链表ChunkManager结构。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634901187984-80624674-706e-4e72-a669-c4773a0ff49f.png#clientId=ue8d1e6c6-b076-4&from=paste&height=209&id=u9c9a6357&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1154&originWidth=2080&originalType=binary&ratio=1&size=129896&status=done&style=none&taskId=u3596dce0-b174-4856-abbb-11e4f46f907&width=377)![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634901197436-67fa513e-981e-4e6d-b3ed-a9a076015797.png#clientId=ue8d1e6c6-b076-4&from=paste&height=186&id=ub76aefec&margin=%5Bobject%20Object%5D&name=image.png&originHeight=898&originWidth=2202&originalType=binary&ratio=1&size=85670&status=done&style=none&taskId=u849ac9b2-693a-4426-8d2f-bd421a46351&width=457)<br />class space

- 只有一个node节点，并且节点内存很大，是连续的，内存小于4G。

non-class space

- 有N个node节点，每个节点大概为2MB。


<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634895460756-8dfce97f-14da-4709-baf7-324547b3b4d6.png#clientId=u3b77b092-32d8-4&from=paste&id=gO0tE&margin=%5Bobject%20Object%5D&name=image.png&originHeight=678&originWidth=764&originalType=binary&ratio=1&size=64871&status=done&style=none&taskId=ua55fa1e9-e133-4efc-8468-9ab12029f9c)<br />​

全局空闲链表ChunkManager能够将non-class空间和class空间中的空闲的内存利用起来，提高内存的利用率。
# 真实案例
Class Space Count：只有一个Node节点<br />NonClass Space Count：有205个Node节点<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1634901313437-25b7da26-1c15-4847-ac29-fc1fa65c9f96.png#clientId=ue8d1e6c6-b076-4&from=paste&height=261&id=ue0452417&margin=%5Bobject%20Object%5D&name=image.png&originHeight=522&originWidth=1810&originalType=binary&ratio=1&size=345665&status=done&style=none&taskId=u53f47e7e-fd62-4776-b391-c04b2ee0499&width=905)


# Metaspace的大小
```java
这下面两个配置项都能控制Metaspace的大小，两个配置项有着微妙的联系。

-XX:MaxMetaspaceSize
-XX:CompressedClassSpaceSize
```
## Metaspace 空间图
Metaspace主要分为两个空间，一个是non-class空间，另一个是class空间。non-class空间有许多Node节点，class空间只有一个Node节点，class空间是一块连续的内存。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1635127890363-6ad67897-a840-4bc2-b2ae-d1889e376053.png#clientId=u46982cdd-7412-4&from=paste&id=u2d5d6c69&margin=%5Bobject%20Object%5D&name=image.png&originHeight=279&originWidth=653&originalType=binary&ratio=1&size=19146&status=done&style=none&taskId=u4b3fb29d-9c8c-4fd0-81db-79e6ec57e57)<br />

## -XX:MaxMetaspaceSize
-XX:MaxMetaspaceSize 代表的是non-class space + class space，也就是这两个空间之和。默认不限制大小，但是还会受到机器本身内存空间的限制。<br />​<br />
## -XX:CompressedClassSpaceSize
`-XX:CompressedClassSpaceSize` 代表的是metaspace空间中的class space，它的大小默认是1G。<br />在启动的时候限制Class space的大小，默认值为1G，启动后就不能修改了。<br />​

在使用压缩类指针技术的前提下，也就是启动参数加上`-XX:UseCompressedClassPointers`，这个Class Space空间最大可以是多少内存呢？2^32 = 4G，所以class space空间最大时4G内存，即`-XX:CompressedClassSpaceSize=4G`。<br />​

​<br />
## 来个demo，练下手
问：若class space空间默认为1G，non-class space ：class space = 5：1。那么MaxMetaspaceSize 应该是多少呢？<br />答案是：6G。<br />​<br />
# 一个类大概需要多少Metaspace空间
一个类所需要的Metaspace空间，分别用到了Metaspace中的non-class space和class space。<br />​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1635129233919-a09f00a4-87d0-4e26-94e0-f1ed83ba2def.png#clientId=u46982cdd-7412-4&from=paste&height=479&id=u89944215&margin=%5Bobject%20Object%5D&name=image.png&originHeight=638&originWidth=677&originalType=binary&ratio=1&size=46527&status=done&style=none&taskId=u695d2709-351b-4bf5-8f6d-3f2ef262d47&width=508)<br />

## 深入 class space
Klass：最大的一部分是 Klass 结构，它是固定大小的。<br />vtable：可变大小，由类中的方法数量决定。<br />itable：可变大小，由这个类所实现接口数量决定。<br />nonstatic oop map：Java类中引用对象的成员的位置，即非静态Oopmap。
## non-class 空间

- ConstantPool：常量池，可变大小。
- 每个成员方法的 metadata：ConstMethod 结构，包含了好几个可变大小的内部结构，如方法字节码、局部变量表、异常表、参数信息、方法签名等；
- 运行时数据，用来控制 JIT 的行为。
- 注解

​<br />
# non-class space 与 class space的大概比例
一个JVM开发者提供的数据，共有4个classloader。<br />我们可以使用 `jcmd pid VM.metaspace` 进行度量。

| loader | #classes | non-class space _(avg per class)_ | class space _(/avg per class)_ | ratio non-class/class |
| --- | --- | --- | --- | --- |
| all | 11503 | 60381k _(5.25k)_ | 9957k _(0.86k)_ | 6.0 : 1 |
| bootstrap | 2819 | 16720k _(5.93k)_ | 1768k _(0.62k)_ | 9.5 : 1 |
| app | 185 | 1320k _(7.13k)_ | 136k _(0.74k)_ | 9.7 : 1 |
| anonymous | 869 | 1013k _(1.16k)_ | 475k _(0.55k)_ | 2.1 : 1 |

- 对于正常的类（我们假设通过 bootstrap 和 app 加载的类是正常的），我可以得到平均每个类需要约 5-7k 的 Non-Class Space 和 600-900 bytes 的 Class Space。

​<br />
# Metaspace与GC的关系
-XX:MetaspaceSize  是初始化元数据空间大小。<br />-XX:MaxMetaspaceSize 是最大的元数据空间大小。<br />-XX:CompressedClassSpaceSize 是元数据空间中class space大小。<br />​

若配置如下
```java
-XX:MetaspaceSize=256M
-XX:MaxMetaspaceSize=512G
-XX:CompressedClassSpaceSize=1G
```
​

当Metaspace空间使用达到了MetaspaceSize，但是还没有达到MaxMetaspaceSize，则会触发FULL GC。<br />用来尝试减少Metaspace的大小。解决方法：设置MetaspaceSize=MaxMetaspaceSize。<br />​

当Metaspace空间使用达到了MaxMetaspaceSize，则会触发FULL GC，GC之后，Metaspace大小依然不减，那么则会继续FULL GC。解决方法：扩大MaxMetaspaceSize。


# 参考资料
[https://www.javadoop.com/post/metaspace](https://www.javadoop.com/post/metaspace)
