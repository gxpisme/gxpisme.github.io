<a name="YSEcG"></a>
# Java-锁
## 背景
在操作资源的时候，尽量不去涉及到锁，因为涉及到锁的时候，就会有竞争，有竞争就会降低性能。

其实资源分为两类，一种是线程内资源，一种是共享资源（只读，读写）；再来看java，java可单线程，可多线程。

我分了几个层次来看这件事情。<br />**第一层：java单线程操作线程内资源。 因为是单线程，没有线程与它竞争，操作的资源又仅仅是线程内资源。所以不会有竞争的，性能也是可以保证的。**<br />**第二层：java单线程操作共享资源。 虽然是共享资源，但只有一个线程在操作，也是不会有竞争的，性能也是可以保证的。**<br />**第三层：java多线程操作共享资源（只读）。虽然是java多个线程在操作共享资源，但是共享资源是只读的，多个线程也不会竞争，无论这个只读的共享资源获取多少次，它都是固定的。**<br />**第四层：java多线程操作共享资源（读写）。这个时候就会产生竞争，A线程对共享资源写，B线程对共享资源读就会产生竞争。**

所以我们尽可能的避免多线程操作读写共享资源，因为这会降低性能。但是在真实的场景中，因为数据一般都会存在数据库，服务器又是很多台。服务器之间的竞争用数据库的锁来解决，服务器内的竞争用java锁来解决。
<a name="NLB7g"></a>
# Java-锁
在java中可以按照不同的维度来把锁归类，接下来介绍下归类后不同的锁。
<a name="lxnHk"></a>
## 乐观锁与悲观锁
乐观锁：就是比较乐观，没有那么多的竞争者，先去获取数据，在最终修改数据的时候才会加锁。适用于读多写少的场景。<br />悲观锁：就是比较悲观，有很多的竞争者，先去加锁，然后再去操作。适用于写多的场景。

乐观锁代表的就是CAS（Compare And Swap），这个会带来ABA问题，通过版本号来进行解决。<br />悲观锁代表的就是ReentrantLock，ReentrantLock分为公平锁和非公平锁。 ReentrantLock分析文章。

- 公平锁：从队尾进。
- 非公平锁：进队头进，默认的事非公平锁。
<a name="OPeOS"></a>
### CAS
Compare And Swap，比较交换。CAS 的操作包括三个参数：内存地址 V、旧的预期值 A、新的值 B。<br />它的原理是，先比较内存地址 V 的当前值是否与预期值 A 相等，如果相等，就将内存地址 V 的值修改为新的值 B，否则什么都不做。

下面就可以简单来理解CAS，比较（money=100 and name=小明），交换（money=money+50）
```sql
-- 小明有100，过了新年，收了50块钱压岁钱。
update user set money=money+50 where money=100 and name="小明";
```
旧的预期值**A** money是100，name是小明。新的值**B** money是money+50。

<a name="Zrkfw"></a>
### ABA
CAS会带来ABA问题，什么是ABA问题呢？A变为了B，然后从B又变回了A。从而看起来是一样的，其实它是不一样的。

| name值 | 线程1 | 线程2 |
| --- | --- | --- |
| A | 获取到A |  |
| A |  | 获取到A |
| B |  | 改为B |
| A |  | 改为A |
| C | 改为C |  |

其实线程1在将name值改为C时，并不知道A被修改成B，又被修改A的，以为name值并没有变化，其实已经变化了。<br />如何识别到这种变化呢？增加一个版本号即可。

| name值 | version | 线程1 | 线程2 |
| --- | --- | --- | --- |
| A | 5 | 获取到A version=5 |  |
| A | 5 |  | 获取到A version=5 |
| B | 6 |  | 改为B version+1 |
| A | 7 |  | 改为A version+1 |
| A |  | version!=5，改C失败 |  |

<a name="lKjH2"></a>
## 锁升级（偏向锁 -> 轻量级锁 -> 重量级锁）
synchronized会有锁升级的过程：偏向锁 -> 轻量级锁 -> 重量级锁。<br />偏向锁：就是乐观锁，会存储锁信息线程，通过CAS来进行判定。<br />轻量级锁：也被称为自旋锁，不断地自旋来获取锁。<br />重量级锁：会用操作系统的锁。

synchronized的分析文章
<a name="uBxQp"></a>
## 可重入锁
可重入锁：指的是同一个线程能够多次获取同一个锁。synchronized 和 ReentrantLock，都是可重入锁。

synchronized的demo，testA和testB方法均被synchronized关键字修饰。同一个线程即主线程能够进入testA方法，也能够进入testB方法，这就意味着，synchronized是可重复入的。

```java
public class T {

    synchronized static String testA() {
        String res = testB();
        return "A" + res;
    }

    synchronized static String testB() {
        return "B";
    }

    public static void main(String[] args) {
        String s = testA();
        System.out.println(s); // 输出AB
    }
}
```

reentrantLock的demo，同一个线程能够持有同一把锁既能进入testA方法，也能进入testB方法。
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 */
public class T {

    private static Lock lock = new ReentrantLock();

    static String testA() {
        String res;
        try {
            lock.lock();
            res = testB();
        } finally {
            lock.unlock();
        }
        return "A" + res;
    }

    static String testB() {
        String res;
        try {
            lock.lock();
            res = "B"; // 这里打个断点，lock.sync.state=2 (这里涉及到可重复入的实现原理)
        } finally {
            lock.unlock();
        }
        return res;
    }

    public static void main(String[] args) {
        String s = testA();
        System.out.println(s); // AB
    }
}
```
<a name="I7ph5"></a>
## 死锁
> 多个线程同时被阻塞，它们一个或者全部都在等待某个资源被释放，导致线程被无限期地阻塞。

<a name="rElPe"></a>
### 举例
线程1，获取A锁，再获取B锁，然后进行操作，释放B锁，释放A锁。<br />线程2，获取B锁，在获取A锁，然后进行操作，释放A锁，释放B锁。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/177252/1679560975852-469cc92c-dcbf-4b86-924d-ed3f738376f2.png#clientId=u30dd6475-37b0-4&from=paste&height=249&id=u4b8fed43&name=image.png&originHeight=498&originWidth=876&originalType=binary&ratio=2&rotation=0&showTitle=false&size=100167&status=done&style=none&taskId=uc1eb1e39-43b2-4c07-989f-c83291cabdb&title=&width=438)<br />如图所示此时线程1获取A锁成功，线程2获取B锁成功；线程1再去获取B锁失败，线程2再去获取A锁失败。<br />这时情况是线程1持有A锁要获取B锁，线程2持有B锁要获取A锁，造成死锁。

<a name="SKElb"></a>
### 分析死锁的条件

1. 锁只能互斥使用，例如锁A不能同时被线程1和线程2使用。
2. 不可抢占，例如线程1不能强制占用线程2持有的锁B，只能等线程2释放了锁B，才能被线程1占用。
3. 请求和保持，例如线程1在获取锁B的时候，保持着对锁A的持有。
4. 等待环路，例如线程1占有线程2的资源，线程2占有线程1的资源，有一个等待环路。
<a name="uKoQL"></a>
### 解决

1. 依次获取锁，都是按照先获取锁A，再获取锁B。就会破坏等待环路的这个条件，从而不会造成死锁。
2. 一次性获取所有需要的锁，把锁A和锁B包装为一个AB锁，只能同时获取，同时释放，也会破坏等待环路的这个条件。
3. 获取锁，获取不到的话，不去等待锁的释放，而是重试或者放弃自己所持有的锁。这样破坏的是请求和保持的条件，也不会造成死锁。
   1. 线程1已经持有了锁A，再获取锁B时失败，那么不等待锁B的释放，等一段时间重试。
   2. 线程1已经持有了锁A，再获取锁B时失败，那么不等待锁B的释放，直接释放自己持有的A锁。
<a name="x2yCa"></a>
### 活锁
还是上面的例子。<br />线程1持有锁A，然后获取锁B；线程2持有锁B，然后获取锁A。为了避免死锁，当获取不到锁时，隔一段时间再重试。

| 时间 | 线程1  | 线程2 |
| --- | --- | --- |
| 1 | 获取锁B失败，等1秒再获取 | 获取锁A失败，等1秒再获取 |
| 2 |  |  |
| 3 | 获取锁B失败，等1秒再获取 | 获取锁A失败，等1秒再获取 |
| 4 |  |  |
| 5 | 获取锁B失败，等1秒再获取 | 获取锁A失败，等1秒再获取 |
| ...... |  |  |

这就是活锁，怎么解决呢？随机一下等待时间就可以了。**不要在相同的时间都去获取对应持有的锁。**


<a name="ms6ix"></a>
# 参考资料
[https://tech.meituan.com/2018/11/15/java-lock.html](https://tech.meituan.com/2018/11/15/java-lock.html)
