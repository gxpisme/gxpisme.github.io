# java 垃圾回收的背景
Java垃圾回收是通过（Tracing GC）来标记出使用的对象，剩下的就是垃圾，也就是未使用的对象。<br />然后将这些垃圾进行清理，从而腾挪出内存空间，供应用程序使用。<br />
<br />如图：通过GC Roots开始向下遍历，找出用到的对象（ObjectA、ObjectB、ObjectC、ObjectD、ObjectE），没有用到对象ObjectE，标记对象ObjectE是垃圾。<br />![image.png](/image/java-garbage-tri-color-one.png)<br />
<br />在（Tracing GC）追踪式垃圾收集的过程中，在原始阶段，会将所有的用户线程中断，只有GC线程在工作，这个过程又称为Stop The World 整个世界都停止了，简称为STW。可以想象下自己使用电脑的时候，电脑总是持续一分钟不能操作，是不是要崩溃了。<br />
<br />在回来看Java，事实上Java中的垃圾收集器并不是这样做的，而是引入了三色标记算法，将用户线程和垃圾回收线程一起工作，从而降低STW的时间。<br />

# 三色标记法
## 哪三种颜色，分别代表的含义
​

按照垃圾收集器是否访问过，来确定某个对象是什么颜色。

- 白色：垃圾收集器没有访问过。
- 黑色：垃圾收集器访问过，并且该对象所有引用都已经扫描过。
- 灰色：垃圾收集器访问过，但该对象至少有一个引用还没有扫描过。

​

三色标记法规定了：黑色不能直接指向白色。<br />
<br />看下图：<br />白色对象：B、C、F、G、H均没有被垃圾收集器访问过。<br />黑色对象：A、D被垃圾收集器访问过，并且他们的所有引用已经扫描过。

- A没有引用
- D引用E，E已经被扫描

灰色对象：E 被垃圾收集器访问过，但是E的引用F和G都没有被扫描过。<br />![image.png](/image/java-garbage-tri-color-two.png)<br />​<br />
## 标记图解
阶段一：选出所有的GC Root<br />阶段二：并发标记

1. 从GC Root开始选择对象，已到灰色集合中。
1. 从灰色集合中选择一个对象，并将其移动到黑色集合中。
1. 将它引用的每个白色对象移动到灰色集合中。
1. 重复上面2 3个步骤，直到灰色集合为空。


<br />![garbage-tri-color.gif](/image/java-garbage-tri-color-three.gif)<br />
<br />整个标记的过程中，是按照最理想的情况下进行的，也就是STW的情况下运行的。然而实际上用户线程一直存在，与垃圾收集线程共存。<br />​

这个时候会产生两种情况，一种是多标（把本来是垃圾，又标记为可使用对象），另一种是漏标（将可使用对象标记为了垃圾）。<br />​<br />
## 多标
把本来是垃圾，标记为可使用的对象。在下一次垃圾回收的时候，就会把这个多标的这个对象，垃圾回收掉，<br />这种多标的，下一次垃圾能够收集掉的垃圾，称为浮动垃圾。这种情况是可以容忍的。<br />​

举例一如下图：<br />若此时用户程序操作 D对象不再引用E对象，因此对象E、F、G应该被回收掉。<br />但是实际上，E已经是灰色了，所以它依然会遍历，最后把E、F、G对象放到黑色集合中。<br />这些E、F、G下次垃圾回收时，没有被引用到，自然会被垃圾回收掉。E、F、G就被称为浮动垃圾。<br />
<br />![image.png](/image/java-garbage-tri-color-four.png)<br />举例二如下图：<br />G对象已经扫描完了，G对象已经被标记为黑色了，若此时用户程序E对象不再引用G对象。但本次垃圾回收不掉，下次垃圾收集的时候，才会被回收。G对象也浮动垃圾。<br />![image.png](/image/java-garbage-tri-color-five.png)<br />

## 漏标
假设GC 线程执行遍历灰色对象E了，如下图所示。<br />
<br />此时用户线程执行了
```java
var G = objE.fieldG;
objE.fieldG = null;  // 灰色E 断开引用 白色G
objD.fieldG = G;  // 黑色D 引用 白色G
```
然后GC线程继续跑，因为E已经不对G引用了，所以不会遍历到对象G了。虽然D引用了G，但因为D已经是黑色了，不会在遍历了。最终垃圾回收的时候会把对象G当成垃圾回收掉，但是应用程序中其实还是用到了，这种情况是接受不了的。<br />![image.png](/image/java-garbage-tri-color-six.png)<br />​

在搜索引擎中，在《深入理解Java虚拟机：JVM高级特性与最佳实践》中均给出了出现上面漏标出现的条件。<br />条件如下：
```java
当且仅当以下两个条件同时满足时，会产生“对象消失”的问题，即原本应该是黑色的对象被误标为白色:
	①：赋值器插入了一条或多条从黑色对象到白色对象的新引用;
	②：赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。
```
_对这个结论有些质疑，见后面的疑问1。_<br />
<br />针对用户线程和GC线程并发过程中，产生漏标的这种情况，只要限制一个条件就能够避免漏标。<br />
<br />

### 漏标针对（条件①黑色引用白色）的解决方案
可以想象，当黑色引用白色时，可以将引用的白色对象置为灰色对象，然后在重新标记下。这个其实就是增量更新（Incremental Update）。<br />​

其中用到了写屏障。<br />写屏障，其实就是在赋值前后加两个函数，赋值前一个函数，赋值后一个函数。赋值前的函数称为写前屏障，赋值后的函数称为写后屏障。
```java

void oop_field_store(oop* field, oop new_value) {
    *field = new_value; // 赋值操作
}

void oop_field_store(oop* field, oop new_value) {
    pre_write_barrier(field); // 写屏障-写前操作
    *field = new_value;
    post_write_barrier(field, value);  // 写屏障-写后操作
}
```
​

主要针对这个操作`objD.fieldG = G;  // 黑色D 引用 白色G`，进行写屏障处理。<br />![image.png](/image/java-garbage-tri-color-seven.png)<br />进行写屏障处理，将新引用的G对象放到需要重新标记的集合里，`remark_set.add(new_value);`后面再扫描一下就行了，保障了不会漏标。
```java
void post_write_barrier(oop* field, oop new_value) {
   remark_set.add(new_value); // 记录新引用的对象
}
```

<br />**CMS 收集器采用该方案（增量更新）。**
### 漏标针对（条件②灰色断开引用白色）的解决方案
可以把断开引用的白色，记录下来，保持最开始时候的样子，将这个白色对象置为灰色对象，最后重新标记下。这个其实就是原始快照（Snapshot At The Beginning）。<br />​

主要针对这个操作`objE.fieldG = null;  // 灰色E 断开引用 白色G `，进行写屏障处理。上面已经介绍了写屏障，可以上翻再回顾下。<br />![image.png](/image/java-garbage-tri-color-eight.png)<br />
<br />这里是如何处理的呢？其实也就是把G记录了下来，`remark_set.add(old_value);`后面再重新扫描下。
```java
void pre_write_barrier(oop* field) {
      oop old_value = *field; // 获取旧值
      remark_set.add(old_value); // 记录  原来的引用对象
}
```
​

**G1 收集器采用该方案（SATB）。**
# GC线程与用户线程协作
上面说了两个阶段分别是

- 阶段一：选出所有的GC Root
- 阶段二：并发标记


<br />又因为GC和用户线程并发时会产生漏标的情况，需要重新扫描下，所以一共是三个阶段。

- 阶段一：选出所有的GC Root （STW，只有GC线程工作）耗时短
- 阶段二：并发标记（GC线程和用户线程工作）耗时长
- 阶段三：重新标记（STW，只有GC线程工作）耗时短



# 疑问
## 疑问1
在用户线程与GC线程并发时，出现漏标的条件如下：
```java
当且仅当以下两个条件同时满足时，会产生“对象消失”的问题，即原本应该是黑色的对象被误标为白色:
	①：赋值器插入了一条或多条从黑色对象到白色对象的新引用;
	②：赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。
```
​

但是我想指出的是，若用户线程将黑色对象直接指向了白色对象呢？如下图：<br />![image.png](/image/java-garbage-tri-color-nine.png)<br />
<br />那岂不是出现漏标就不是当且仅当那两个条件了，在论文中[https://www.cs.cmu.edu/~fp/courses/15411-f14/misc/wilson94-gc.pdf](https://www.cs.cmu.edu/~fp/courses/15411-f14/misc/wilson94-gc.pdf)  作者是这样写
```java
If the mutator creates a pointer from a black object to a white one it must somehow notify the collector that its assumption has
been violated This ensures that the collectors book keeping is brought up to date

如果用户线程创建了一个从黑色对象指向白色对象的指针，它必须以某种方式通知收集器它的规则被违背了。这确保了收集器记录是最新的。
```
我猜测，`objG.field = H` 这种情况下，用增量更新来解决肯定是没有问题的。但是用SATB来解决，肯定是有问题的，你知道哪里出错了吗？<br />​<br />
## 疑问2
重新标记，为什么也要STW。<br />​

重新标记是将少量的需要重新扫描的对象重新在扫描一次，这些对象数量少。这些对象重新标记后，就可以进行垃圾回收了。如果不去STW的话，重新标记的对象可能还是会存在漏标的情况。<br />​<br />
## 疑问3
哪些垃圾收集器整个阶段都是STW的。<br />新生代的垃圾收集器：Serial、ParNew、Parallel Scavenge都是STW的。<br />老年代的垃圾收集器：Serial Old、Parallel Old都是STW的。<br />​

哪些垃圾收集器部分阶段是STW的。<br />老年代垃圾收集器：CMS<br />面对全区垃圾收集器：G1、ZGC<br />
<br />这个说部分阶段是STW，实际上指的是标记过程不是STW，GC线程和用户线程同时工作的。<br />在选取GC Roots阶段，是STW的。<br />在重新标记阶段，是STW的。<br />

# 参考资料
[https://www.cs.cmu.edu/~fp/courses/15411-f14/misc/wilson94-gc.pdf](https://www.cs.cmu.edu/~fp/courses/15411-f14/misc/wilson94-gc.pdf)<br />[https://www.cnblogs.com/wjxzs/p/14233656.html](https://www.cnblogs.com/wjxzs/p/14233656.html)<br />[https://www.cnblogs.com/hongdada/p/14578950.html](https://www.cnblogs.com/hongdada/p/14578950.html)

