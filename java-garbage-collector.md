# JAVA-垃圾回收-学习笔记

## 为什么要进行垃圾回收
> 对C、C++来讲，开发人员不仅要创建对象，而且要销毁对象。
> 
> 但是对于JAVA来说，开发人员只要创建对象就行，不用专门去销毁对象，释放内存。
> 
> JAVA虚拟机会把垃圾对象进行清理，保障内存有足够的空间可用。

## JAVA垃圾存在什么位置？

java的**内存**按照两个维度进行区分如下

1. Pre Thead （每个线程中）
	- Program Counter Register（程序计数器）
	- JVM Stacks（虚拟机栈）
	- Native Method Stacks（本地方法栈）
2. Common Area （公共区域）
	- **Heap（堆）**
	- Method Area（方法区）
	- Runtime Constant Pool（运行时常量）

Java垃圾存在`Common Area`公共区域中的`Heap`区。

这个Heap的大小，通过`-Xms`指定Heap初始值大小  `-Xmx`指定Heap的最大值

JAVA垃圾回收就是要回收Heap中垃圾。

## 找出垃圾？
> 找出垃圾，通用的有两种算法。1. Reference Count 引用计数法。2. Root Searching 根搜索算法

### Reference Count 引用计数法
每个对象都有其引用，如果其引用为0，那么就认为没有被引用了，就可以被回收了。

```
a = new Object();  //该创建出来的对象，其引用计数为1
b = a; //b引用该创建对象，则该对象其引用计数为2
b = null; //b不再引用对象，则该对象其引用计数为1
a = null; //a不再引用对象，则该对象其引用计数为0，此时就认为该对象没有被引用，就是垃圾，可以被回收了。
```

![](java-garbage-collector-refer-to-each-other.jpg)

该引用计数的会产生内存泄漏，因为有的垃圾回收不了，什么垃圾这么牛逼，就是循环引用的垃圾。A对象引用B对象，B对象引用A对象，A、B对象都没有被使用到，使用该算法回收不了垃圾。

详细可以看下PHP是如何使用引用计数进行垃圾回收的，很有意思。
[php垃圾回收](https://gxpisme.github.io/php7-garbage-collection)

### Root Searching 根搜索算法
因为在java中入口 `public static void main(String[] args)`内，都是使用的对象，这个其实就是根Root，所以从这个根Root往下找，找到的都是使用的对象，在内存中将使用到的对象排除掉后，就是垃圾对象了。


## 垃圾清理？
> 找到垃圾了，如何对垃圾进行清理，主要看下三种算法。

### 垃圾清理算法
#### 标记清除(Mark-Sweep)

![](/image/java-garbage-collector-mark-sweep.jpg)

将可回收的对象进行标记清除，下次使用这块内存的时候就可以直接使用。

这里带来的问题是：其一是执行效率不稳定，会随着对象数量的增长而效率降低。其二是内存空间碎片化。

#### 标记复制(Mark-Copy)

![](/image/java-garbage-collector-mark-copy.jpg)

半区复制：将可用内存按照容量划分为大小相等的两块，每次只十四用其中一块，当这一块用完了，就将存活的对象复制到另一块上，然后将该内存清理。

优点：存活的对象较少，实现简单，运行高效

缺点：空间浪费


#### 标记整理(Mark-Compact)

![](/image/java-garbage-collector-mark-compact.jpg)

标记-整理”(Mark-Compact)算法，其中的标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存。

在对象存活率较高的情况下，效率会降低。这种对象移动操作必须全程暂停用户应用程序才能运行，Stop The World，简称为STW。


## 分代收集理论

弱分代假说 Weak Generational Hypothesis 绝大多数对象都是朝生夕灭的。

强分代假说 Strong Generational Hypothesis 熬过越多次垃圾收集过程的对象就越难消亡

跨代引用假说 Intergenerational Reference Hypothesis 跨代引用相对于同代引用来说仅占极少数。

根据分代收集理论将Heap区分为新生代（Young）和老年代（Old）。

新生代（Young）这里就是将绝大多数对象存在了新生代，因为绝大多数对象都是朝生夕灭的。
老年代（Old）这存了经过多次垃圾收集过程，依然存活的对象。

![](/image/java-garbage-collector-ratio.jpg)


设置新生代和老年代的比例 参数是NewRatio

设置Survivor在新生代的比例 参数是SurvivorRatio


```
java -XX:+PrintFlagsFinal -version | grep NewRatio      查看新生代和老年代  默认内存比例

java -XX:+PrintFlagsFinal -version | grep SurvivorRatio  查看Survivor在新生代 默认内存比例

jstat -gc pid  可以查看真实的大小，然后自己可以计算下比例。

S0C：第一个幸存区的大小
S1C：第二个幸存区的大小
EC：伊甸园区的大小
OC：老年代大小
```

![](/image/java-garbage-collector-young-gc.jpg)

对象年龄最大为15，因为对象头信息只有4bit来记录年龄。
设置进入老年代的对象年龄通过该参数进行指定`+XX:MaxTenuringThreshold`

### 新生代
> 新生代 采用标记复制算法

新生代产生垃圾回收又称为 YoungGC  也称为Mirror gc

### 老年代
> 老年代 采用标记整理算法

老年代产生垃圾回收被称为 OldGC 也称为 Major GC

Full GC = YoungGC + OldGC


### 跨代引用
存在互相引用关系的两个对象，是应该倾向于同时生存或者同时消亡的。

举个例子，如果某个新生代对象存在跨代引用，由于老年代对象难以消亡，该引用会使得新生代对象在收集时同样可以存活，
进而在年龄增长之后晋升到老年代中，这时跨代引用也随即被消除了。

我们就不应再为了少量的跨代引用而扫描整个老年代，也不必浪费空间专门记录每一个对象是否存在及存储哪些跨代引用
只需要在新生代上建立一个全局的数据结构（Remembered Set），这个结构把老年代划分成若干个小块，标识出老年代的哪一块内存会存在跨代引用。

## 垃圾收集器
### 新生代
- Serial 单线程工作的收集器 （标记复制算法）
- ParNew 多线程工作的收集器（标记复制算法）
- Parallel Scavenge 多线程工作收集器（标记复制算法）
### 老年代
- Serial Old 单线程工作收集器（标记整理算法）
- Parallel Old 多线程工作收集器（标记整理算法）
- CMS 多线程工作收集器（标记清除算法）

Serial 和 Serial Old 配合
ParNew 和 CMS 配合
Parallel Scavenge 和 Parallel Old 配合

CMS收集器是一种以获取最短回收停顿时间为目标的收集器。

CMS使用的是标记清除算法，所以会产生碎片，实际上。ParNew和CMS进行搭配的时候，特殊的时候需要内存整理时，偶尔还会用到Serial Old收集器。

查看使用的哪个收集器 `java -XX:+PrintCommandLineFlags -version`
- `-XX:+UseParallelGC` 代表Parallel Scavenge 和 Parallel Old 配合
- `-XX:+UseConcMarkSweepGC -XX:+UseParNewGC` 代表的是ParNew 和 CMS 配合

