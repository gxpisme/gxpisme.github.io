# JMM (Java Memory Model)

## 回顾Java 内存管理
先来回顾下Java的内存管理，看如下图所示。

每个线程都有一块栈内存，多个线程共享堆内存和元数据空间内存。

每个线程中会有栈帧，栈帧对应的就是函数操作。

![image.png](/image/java-memory-model-1.png)


# JMM
将共享的内存抽象一下，看下图所示：

![image.png](/image/java-memory-model-2.png)

- 共享变量在主内存中，在工作内存中会有共享变量的**副本**。

- Java线程对变量的所有操作都在工作内存中进行。

## 三大特性：内存可见性、原子性、有序性

Java内存模型是围绕在并发过程中如何处理三个特性来建立的。

哪三个特性呢？它们分别是 内存可见性、原子性、有序性。


### 先来说下内存可见性。

由于java线程对变量的操作都在工作内存中，现在有两个java线程A、B，变量flag初始为true，线程A、B都有变量flag的操作，若线程A修改了变量flag的值，那线程B的工作内存变量flag还是旧值。这就是内存可见性。

```java
flag = true;

// 线程A 虽然这里修改了
flag = false;

// 线程B 但这里会一直执行，因为线程B中的工作内存flag变量的值还是为true
while(flag) {
}
```

### 再来说下原子性。

原子性就是一个事件要么发生，要么不发生，不存在中间状态。

`i = 0;`  这个就是原子操作

`i++;`   这个就是非原子操作，先获取到i的值，然后在对这个值+1。


### 最后说下有序性。
程序最终都会转为指令一步步运行，这些指令的执行顺序会被优化，所以最终执行的时候，并不是按照程序中所写的那样，从前往后，一步步执行。美团一篇文章有证明[https://tech.meituan.com/2014/09/23/java-memory-reordering.html](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)。


可见性：可以通过volatile、synchronized、final关键字来解决

原子性：可以通过synchronized关键字来解决

有序性：可以通过volatile、synchronized关键字来解决


## Happens-Before原则

除了这三个特性，还有线程Happens-Before原则，规则定了多线程语义下，操作符合哪些规范和要求。

> Happens-Before 原则：Java内存模型中定义的两项操作之间的顺序关系，比如：操作A先与操作B执行。

> 这个原则是判断数据是否存在竞争，线程是否安全的重要手段。

```java
// 在线程A执行
i = 1;
// 在线程B执行
j = i
// 在线程C执行
i = 2;
```

程序次序规则：一个线程内，按照代码顺序，书写在前面的代码happens-before（先发生于）书写在后面的代码；

管程锁定规则：对同一个资源，unlock操作happens-before（先发生于）后面的lock操作；

volatile变量规则：对一个变量的写操作happens-before（先发生于）后面对这个变量的读操作；

线程启动规则：Thead对象的start()方法happens-before（先发生于）该线程任意操作；

线程终止规则：线程中所有操作都happens-before（先发生于）该线程的终止检测操作。

线程中断规则：对线程interrupt()方法的调用happens-before（先发生于）被中断线程的代码检测到中断事件的发生；

对象终结规则：一个对象的构造函数happens-before（先发生于）它的finalize()方法。

传递性：如果操作A happens-before（先发生于）操作B，操作B happens-before（先发生于）操作C，那就可以得出操作A happens-before（先发生于）操作C的结论。


还有一个原则是：as-if-serial原则，看起来是串行的。

在单线程内，不管怎么重排序，最终的执行结果都是正确的。

```java
a = 1;
b = 3;
c = a + b;
```

第一行代码和第二行代码可能会有重排序，但是无论怎么重排序，第三行代码一定是在第一和第二行代码执行完成后才去执行。

上面的代码最终可能执行的顺序如下，第一和第二行交换了执行顺序，但是结果是正确的，看起来是串行的，as-if-serial。

```java
b = 3;
a = 1;
c = a + b;
```

