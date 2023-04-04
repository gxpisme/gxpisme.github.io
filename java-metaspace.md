# MetaSpace是什么？
MetaSpace 是JDK1.8之后才有的，称为元数据空间，存放类的元数据 class metadata。

<br />

我是这样理解的，比如有一个狗的模具（类），通过狗模具（类）印出来的狗（实例）会有很多，这些狗（实例）存在了Heap里，也就是堆中。而这个狗模具（类）存放到那了呢？存放到了MetaSpace空间中。

- 狗模具(类) 存放在 MetaSpace中
- 印出来的狗（实例） 存放在 Heap中

![](/image/java-metaspace-one.png)

<br />

在Java中 everthing is object 所有东西都是对象。对应到咱们写的代码上，一个类的相关信息都存放在了MetaSpace空间中，类的相关信息：属性字段、方法参数、常量池、接口等等。

# MetaSpace如何管理空间
## MetaSpace 空间大小
Metaspace 区域位于堆外，所以它的最大内存大小取决于系统内存，而不是堆大小，我们可以指定 MaxMetaspaceSize 参数来限定它的最大内存，若不指定MaxMetaspaceSize参数最大内存就是系统内存。
<br />

> -XX:MetaspaceSize=512m<br />-XX:MaxMetaspaceSize=512m Metaspace总空间的最大允许使用内存，默认是不限制。<br />-XX:CompressedClassSpaceSize=1G Metaspace 中的 Compressed Class Space 的最大允许内存，默认值是 1G，这部分会在 JVM 启动的时候向操作系统申请 1G 的虚拟地址映射，但不是真的就用了操作系统的 1G 内存。

## 分配Metaspace空间
通俗来讲：就是加载类时，需要将类的元数据往_Metaspace_空间存一份。
当加载类的时候，类加载器会负责将类的元数据信息存放到Metaspace空间中，将类实例存放到Heap中。

如下图：类加载器Id，加载了类X，类X生成了两个实例，但是类X的元数据只有一份，因此Metaspace空间只有一份类X的元数据。<br />类加载器Id，加载了类Y，类Y生成了一个实例，Metaspace空间有一份类Y的元数据。<br />![](/image/java-metaspace-two.png)

## 回收metaspace空间
分配给类的空间，这个空间是归属于这个该类的类加载器，只有当这个类加载器卸载的时候，这个空间才会被释放。

通俗来讲：一个类加载器的所有类，均不再引用时，类加载器在卸载的时候，metaspace空间才会释放。

如下图

阶段一：类Y的实例c，不再引用了。实例c还在Heap 堆中。<br />阶段二：类X的实例a b，也不再引用了。实例a b还在Heap堆中。<br />阶段三：GC（垃圾回收）开始执行，实例a b c都被回收。<br />阶段四：类加载器Id加载的所有类都被回收，类加载器对应的Metaspace空间也被回收掉。![image.png](/image/java-metaspace-three.png)

## 内存分配
### VirtualSpaceList
一个 Node 是 2MB 的空间，这里的 2MB 不是真的就消耗了内存的 2MB，只有在使用的时候才会真的消耗内存。这里是虚拟内存映射。

![image.png](/image/java-metaspace-four.png)

### Chunk
从一个 Node 中分配内存，每一块称为 MetaChunk，chunk 有三种规格，在 64 位系统中分别为 1K、4K、64K。

![image.png](/image/java-metaspace-five.png)<br />

- 通常，一个标准的类加载器在第一次申请空间时，会得到一个 4K 的 chunk，直到它达到了一个随意设置的阈值，此时分配器失去了耐心，之后会一次性给它一个 64K 的大 chunk。
- bootstrap classloader 是一个公认的会加载大量的类的加载器，所以分配器会给它一个巨大的 chunk，一开始就会给它 4M。可以通过 InitialBootClassLoaderMetaspaceSize 进行调优。
- 反射类类加载器 (jdk.internal.reflect.DelegatingClassLoader) 和匿名类类加载器只会加载一个类，所以一开始只会给它们一个非常小的 chunk（1K），因为给它们太多就是一种浪费。

### Block
在 Metachunk 上，我们有一个二级分配器（class-loader-local allocator），它将一个 Metachunk 分割成一个个小的单元，这些小的单元称为 Metablock，它们是实际分配给每个调用者的。<br />![image.png](/image/java-metaspace-six.png)<br />


# MetaSpace 架构
> 深入到MetaSpace架构中，看看他们多层，多组件是如何组织协作的。


Metaspace分了三个层，最底层，中间层，最上层。<br />最底层：直接从操作系统中申请大块内存。<br />中间层：将内存块划分为小块Chunk给类加载器使用。<br />最上层：类加载器分开的Chunk，给调用代码使用。
## 最底层 VirtualSpaceList
在最底层，JVM 通过 mmap(3) 接口向操作系统申请内存映射，在 64 位平台上，每次申请 2MB 空间。<br />​<br />
> 当然，这里的 2MB 不是真的就消耗了主存的 2MB，只有之后在使用的时候才会真的消耗内存。这里是虚拟内存映射。

每次申请过来的内存区域，放到一个链表中 [VirtualSpaceList](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/memory/metaspace/virtualSpaceList.hpp#l39)，作为其中的一个 Node。看下图。<br />​

一个 Node 是 2MB 的空间，前面说了在使用的时候再向操作系统申请实际的内存，但是频繁的系统调用会降低性能，所以 Node 内部需要维护一个水位线，当 Node 内已使用内存快达到水位线的时候，向操作系统要新的内存页。并且相应地提高水位线。

直到一个 Node 被完全用完，会分配一个新的 Node，并且将新的Node加入到链表中，老的 Node 就 “退休” 了。下图中，前面的三个 Node 就是退休状态了。退休节点的剩余空间不会丢失，而是被分割成块并添加到全局自由列表中。
<br />
<br />

从一个 Node 中分配内存，每一块称为 MetaChunk，chunk 有三种规格，在 64 位系统中分别为 1K、4K、64K。

![image.png](/image/java-metaspace-seven.png)

VirtualSpaceList 和它的Node节点是全局的结构，Metachunk归属于类加载器。因此在VirtualSpaceList中的Node节点经常会有不同的类加载器。如下图：一个Node中就有a、b、c、d四个类加载器。

![image.png](/image/java-metaspace-eight.png)

当一个类加载器它所有的类都被卸载后，被卸载的类加载器所对应的元数据信息被释放。所有空闲的chunk被添加到一个全局空闲列表中([ChunkManager](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/memory/metaspace/chunkManager.hpp#l44))<br />​

![image.png](/image/java-metaspace-nine.png)

如果其他类加载器开始加载类，在申请Metaspace空间的时候，会用之前的空闲空间。上图类加载器b对应的空间被释放，下图类加载器e和类加载器f就会使用类加载器b所释放的空间。

![image.png](/image/java-metaspace-ten.png)

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

![image.png](/image/java-metaspace-eleven.png)


这个 chunk 诞生的时候，它只包含header，之后的分配都只要在顶部进行分配就行。<br />​

由于这个 chunk 是归属于一个类加载器的，所以如果它不再加载新的类，那么 unused 空间就将真的浪费掉。

# ClassloaderData and ClassLoaderMetaspace
在 JVM 内部，类加载器是以 [ClassLoaderData](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/classfile/classLoaderData.hpp#l176) 结构标识的，这个结构引用了一个 [ClassLoaderMetaspace](http://hg.openjdk.java.net/jdk/jdk11/file/1ddf9a99e4ad/src/hotspot/share/memory/metaspace.hpp#l230)结构，它维护了该加载器使用的所有的 Metachunk。<br />​

当这个类加载器被卸载的时候，这个 ClassLoaderData 和 ClassLoaderMetaspace 会被删除。并且会将所有的这个加载器用到的 chunks 归还到空闲列表中。如果恰好把整个 Node 都清空了，那么这个 Node 的内存直接还给操作系统。

# 内存什么时候会还给操作系统
当一个 VirtualSpaceListNode 中的所有 chunk 都是空闲的时候，这个 Node 就会从链表 VirtualSpaceList 中移除，它的 chunks 也会从空闲列表中移除，这个 Node 就没有被使用了，会将其内存归还给操作系统。

对于一个空闲的 Node 来说，拥有其上面的 chunks 的所有的类加载器必然都是被卸载了的。
至于这个情况是否可能发生，主要就是取决于碎片化：
一个 Node 是 2M，chunks 的大小为 1K, 4K 或 64K，所以通常一个 Node 上有约 150-200 个 chunks，如果这些 chunks 全部由同一个类加载器拥有，回收这个类加载器就可以一次性回收这个 Node，并且把它的空间还给操作系统。

但是，如果这些 chunks 分配给不同的类加载器，每个类加载器都有不同的生命周期，那么什么都不会被释放。这也许就是在告诉我们，要小心对待大量的小的类加载器，如那些负责加载匿名类或反射类的加载器。

![image.png](/image/java-metaspace-twelve.png)

- 每次向操作系统申请 2M 的虚拟空间映射，放置到全局链表中，待需要使用的时候申请内存。
- 一个 Node 会分割为一个个的 chunks，分配给类加载器，一个 chunk 属于一个类加载器。
- chunk 再细分为一个个 Metablock，这是分配给调用者的最小单元。
- 当一个类加载器被卸载，它占有的 chunks 会进入到空闲列表，以便复用，如果运气好的话，有可能会直接把内存归还给操作系统。

​<br />

# 对象类型指针 Class Metadata address
大家都知道，Java中实例对象都在Heap堆中存储，这些对象对应的类结构都存储在Metaspace空间中。<br />一个实例对象有一个指针指向Metaspace空间所对应的类结构。<br />​<br />
> 实例对象包含三部分：对象头、实例变量、填充数据。
> 对象头存储了到对象类型数据的指针。

![image.png](/image/java-metaspace-fourteen.png)

在64位系统上对象类型指针占用64bit，2^64bit能够代表2^34G内存的地址，在实际的机器上，并不需要那么多对象。采用对象压缩指针技术，就使用32bit，2^32bit能够代表4G内存的地址，在应用过程中，能够处理正常情况。这样对象能够占用更少的内存。

![image.png](/image/java-metaspace-fifteen.png)

使用对象压缩指针技术，在Metaspace空间中存储的Klass结构，该内存需要是一块连续的内存，否则在内存大于4G的机器上，无法准备找到。

若内存16G，指针能够表示的内存只有4G，肯定是满足不了的，为了解决这种情况，就分配一块连续小于4G的内存用来存储Klass结构。

存储Klass结构的内存是连续的，从某个内存地址开始到某个内存地址结束。找到真正的Klass结构地址，需要加上一个base值。

# Class Space 和 Non-Class Space
由于对象类型指针压缩技术，Metaspace需要一块连续的内存存储Klass结构。但是对应非Klass结构的元数据，则不需要一块联系的内存进行存储。​

联想到之前提到的全局VirtualSpaceList结构和全局空闲链表ChunkManager结构。

![image.png](/image/java-metaspace-sixteen.png)

![image.png](/image/java-metaspace-seventeen.png)

class space
- 只有一个node节点，并且节点内存很大，是连续的，内存小于4G。

non-class space
- 有N个node节点，每个节点大概为2MB。

![image.png](/image/java-metaspace-eighteen.png)
​

全局空闲链表ChunkManager能够将non-class空间和class空间中的空闲的内存利用起来，提高内存的利用率。
# 真实案例
Class Space Count：只有一个Node节点

NonClass Space Count：有205个Node节点

![image.png](/image/java-metaspace-nineteen.png)


# Metaspace的大小
```java
这下面两个配置项都能控制Metaspace的大小，两个配置项有着微妙的联系。

-XX:MaxMetaspaceSize
-XX:CompressedClassSpaceSize
```
## Metaspace 空间图
Metaspace主要分为两个空间，一个是non-class空间，另一个是class空间。non-class空间有许多Node节点，class空间只有一个Node节点，class空间是一块连续的内存。<br />![image.png](/image/java-metaspace-twenty.png)<br />

## -XX:MaxMetaspaceSize
-XX:MaxMetaspaceSize 代表的是non-class space + class space，也就是这两个空间之和。默认不限制大小，但是还会受到机器本身内存空间的限制。<br />​<br />
## -XX:CompressedClassSpaceSize
`-XX:CompressedClassSpaceSize` 代表的是metaspace空间中的class space，它的大小默认是1G。<br />在启动的时候限制Class space的大小，默认值为1G，启动后就不能修改了。<br />​

在使用压缩类指针技术的前提下，也就是启动参数加上`-XX:UseCompressedClassPointers`，这个Class Space空间最大可以是多少内存呢？2^32 = 4G，所以class space空间最大时4G内存，即`-XX:CompressedClassSpaceSize=4G`。<br />​

​<br />
## 来个demo，练下手
问：若class space空间默认为1G，non-class space ：class space = 5：1。那么MaxMetaspaceSize 应该是多少呢？<br />答案是：6G。Metaspace空间=non-class space + class space<br />​<br />
# 一个类大概需要多少Metaspace空间
一个类所需要的Metaspace空间，分别用到了Metaspace中的non-class space和class space。<br />​

![image.png](/image/java-metaspace-twenty-one.png)<br />

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
- [https://www.javadoop.com/post/metaspace](https://www.javadoop.com/post/metaspace)
- [https://stuefe.de/posts/metaspace/what-is-metaspace/](https://stuefe.de/posts/metaspace/what-is-metaspace/)
