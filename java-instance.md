# 单例
> 单例简单来讲就算是在内存中一个对象只存在一个实例。
>
> 如何保证一个对象在内存中只存在一个实例呢？有很多的知识点，一起来学习下。


## 有饿汉(Hungry)和懒汉(Lazy)
> 不止一次听说有单例有饿汉和懒汉模式，之前没有太在意。

### 饿汉(Hungry)
- static的成员是在类加载的时候就生成了，而不是生成对象的时候。所以是安全的，因为类加载之前是不可能生成对象和多线程的

```
  public class Hunger {
  		// 静态初始化器由JVM在类初始化时运行，在类装入之后，但在类被任何线程使用之前。
        private static Hunger instance = new Hunger();

        private Hunger() {}

        public static Hunger getInstance() {
            return instance;
        }
    }
```

问题：Hungry是提前创建，不管后面用不用。如果不用的话，就会浪费空间、资源。

### 懒汉（Lazy）
- Lazy是用的时候创建。

#### 最基础的
```
// 最基础的，但是这种不是线程安全的，并发情况下，会出现两个及以上实例。
public class FirstDemo {

    private static FirstDemo firstDemo = null;

    private FirstDemo() {
    }

    public static FirstDemo getInstance() throws InterruptedException {
        // 这里并发的情况下，就会出现两个实例
        if (firstDemo == null) {
            firstDemo = new FirstDemo();
        }
        return firstDemo;
    }
}
```

#### synchronized

```
public class FirstDemo {

    private static FirstDemo firstDemo = null;

    private FirstDemo() {
    }

    public static FirstDemo getInstance() throws InterruptedException {
        if (firstDemo == null) {
            // 这里并发的情况下，会有多个线程在这里执行，先获取锁然后创建对象。
            synchronized (FirstDemo.class) {
                firstDemo = new FirstDemo();
            }
        }
        return firstDemo;
    }
}
```
假设有两个线程，线程A和线程B同时到了，线程A获取到了锁，线程B在此等待，线程A创建对象操作完后，线程A释放锁，此时线程B获取到锁，又进到了里面，还是会重新创建对象。**所以就引出了Double Check Lock**

#### Double Check Lock

```
public class FirstDemo {

    private static FirstDemo firstDemo = null;

    private FirstDemo() {
    }

    public static FirstDemo getInstance() throws InterruptedException {
        if (firstDemo == null) {
            // 这里并发的情况下，会有多个线程在这里执行，先获取锁然后创建对象。
            synchronized (FirstDemo.class) {
                // 防止等待锁的线程B（此时已经有线程A创建好并释放锁），线程B在判断下就能保证只有一个对象了。
            	if (firstDemo == null) {
                    firstDemo = new FirstDemo();
                }
            }
        }
        return firstDemo;
    }
}
```

#### volidate的使用
`new FirstDemo()` 在java中创建一个对象不是一个原子操作，可以被分解为三个步骤。

```
//1：分配对象FirstDemo的内存空间
memory = allocate();
//2：初始化对象(给相应的属性进行赋值)
ctorInstance(memory);
//3：设置instance指向刚分配的内存地址
instance = memory;
```

在上面的伪代码中2和3可能会被指令重排。（为什么会指令重排，因为指令重排能够压榨CPU的性能）。

重排之后的伪代码如下

```
//1：分配对象的内存空间
memory = allocate();
//3：设置instance指向刚分配的内存地址
instance = memory;
//2：初始化对象
ctorInstance(memory);
```
在单线程创建单例的场景，肯定是没有问题的。但是在多线程并发的情况下，可能会导致某些线程访问到未初始化的变量，然后进行后面操作，肯定是有问题的。


| 时间  | 线程A  | 线程B |
|------------- |--------------- | ------------- |
| t1      | A1: 分配对象的内存空间 |           |
| t2      | A3: 设置instance指向内存空间        |           |
| t3      |         | B1: 判断instance是否为空           |
| t4      |         | B2: 由于instance不为null，线程B将访问instance引用的对象           ，但是此时对象时一个空值，然后业务逻辑就拿着这个空值用了，肯定是有问题的。|
| t5      | A2: 初始化对象        |            |
| t6      | A4: 访问instance引用的对象，A线程是正确的，也拿到值了，肯定是没有问题的。| |

所以总结来看由于指令重排序导致了线程B拿到一个空值实例，肯定在业务中有问题的。

在上面的代码中看，线程A创建对象中的初始化对象还未完成，锁还未释放，线程B在第最外层的判断为false，直接返回了一个空值的对象。

如何防止指令重排序呢？ 采用`volidate`即可。


```
public class FirstDemo {

    // volidate 就是防止指令重排序
    private static volidate FirstDemo firstDemo = null;

    private FirstDemo() {
    }

    public static FirstDemo getInstance() throws InterruptedException {
        if (firstDemo == null) {
			// 这里并发的情况下，会有多个线程在这里执行，先获取锁然后创建对象。
            synchronized (FirstDemo.class) {
                // 防止等待锁的线程B（此时已经有线程A创建好并释放锁），线程B在判断下就能保证只有一个对象了。
                if (firstDemo == null) {
                    firstDemo = new FirstDemo();
                }
            }
        }
        return firstDemo;
    }
}
```

## 静态内部类

这种方式跟饿汉方式采用的机制类似，但又有不同，两者都是采用了类装载的机制来保证初始化实例时只有一个线程。

不同的地方在饿汉方式是只要类被装载就会实例化，没有Lazy-Loading的作用，

静态内部类方式在类被装载时并不会立即实例化，而是调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton实例化。

类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。

```
public class Singleton {

    private Singleton() {}

    private static class SingletonInstance {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```

这种情况下就不用考虑指令重排的问题了，因为只有单个线程。


## 参考资料
- [why-is-hungry-singleton-pattern-thread-safe](https://stackoverflow.com/questions/49504574/why-is-hungry-singleton-pattern-thread-safe)
- [单例陷阱——双重检查锁中的指令重排问题](https://www.cnblogs.com/lkxsnow/p/12293791.html)
- [Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)
