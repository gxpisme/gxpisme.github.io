# JAVA-垃圾回收-学习笔记

## 为什么要进行垃圾回收
> 对C、C++来讲，开发人员不仅要创建对象，而且要销毁对象。
> 
> 但是对于JAVA来说，开发人员只要创建对象就行，不用专门去销毁对象，释放内存。
> JAVA虚拟机会把垃圾对象进行清理，保障垃圾对象不占用内存空间。


## JAVA垃圾存在什么位置？

java的**内存**按照两个维度进行区分如下

1. Pre Thead （每个线程中）
	- Program Counter Register（程序计数器）
	- JVM Stacks（虚拟机栈）
	- Native Method Stacks（本地方法栈）
2. Common Area （公共区域）
	- Heap（堆）
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

该引用计数的会产生内存泄漏，因为有的垃圾回收不了，什么垃圾这么牛逼，就是循环引用的垃圾。A对象引用B对象，B对象引用A对象，A、B对象都没有被使用到，使用该算法回收不了垃圾。

详细可以看下PHP是如何使用引用计数进行垃圾回收的，很有意思。
[php垃圾回收](https://gxpisme.github.io/php7-garbage-collection)

### Root Searching 根搜索算法
因为在java中入口 `public static void main(String[] args)`内，都是使用的对象，这个其实就是根Root，所以从这个根Root往下找，找到的都是使用的对象，在内存中将使用到的对象排除掉后，就是垃圾对象了。


## 垃圾清理？
### 垃圾清理算法
#### 标记清除(Mark-Sweep)

#### 标记复制(Mark-Copy)
#### 标记整理(Mark-Compact)
