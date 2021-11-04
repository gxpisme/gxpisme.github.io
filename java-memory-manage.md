# 从一段代码来看Java的内存管理

代码比较简单，就是new一个小狗，给它取个名字，叫一下。

```java
public class test {

    public static void main(String[] args) {
        Dog yellow = new Dog("大黄");
        yellow.say();
        Dog white = new Dog("小白");
        white.say();
    }
}

public class Dog {
    private String name;

    public Dog(String name) {
        this.name = name;
    }

    public void say() {
        System.out.println(name + "叫 汪汪");
    }
}
```


`Dog yellow = new Dog("大黄");`

`Dog white = new Dog("小白");`

不难理解，这里new出来的实例对象是需要存储起来的，哪这个实例存在哪了呢？new出来的对象都存放到了一块内存上，这块内存叫堆（Heap）。

![image.png](/image/java-memory-manage-one.png)


至于Dog这个类，Dog这个类有自己的属性name，有自己的方法say。这些信息，同样有地方存储。存储的地方称为方法区，在JDK1.8又称为元数据空间。

![image.png](/image/java-memory-manage-two.png)

（yellow）大黄和（white）小白，这两个对象怎么知道自己属于哪个类呢？其实对象中会存在一个类型指针指向它的类型。用来知道（yellow）大黄属于Dog类的，知道（white）小白属于Dog类的。
​

![image.png](/image/java-memory-manage-three.png)


在java中会有很多类，也会有很多对象，类信息都存在了方法区/元数据空间中，new出来的对象都存在了堆Heap中。
​

​

在java中，会有很多线程一起工作。所有线程使用到的类，new出来的对象，都分别存放在了方法区/元数据空间、堆Heap中。所以堆Heap和元数据空间是所有线程共享的，共同拥有的。
​

​

刚才讲了，java中会有很多线程，每个线程都会执行函数，函数的执行都是在栈Stack中进行的。还是最上面的例子`yellow.say();` 这些函数间的相互调用，都存放到栈中了，栈也是需要内存来存储的。
​

![image.png](/image/java-memory-manage-four.png)


栈帧中就存储了变量、方法出口等信息。

![image.png](/image/java-memory-manage-five.png)


栈帧中存储了对象指针，也就是Heap堆中的对象大黄的地址。

![image.png](/image/java-memory-manage-six.png)


​

在从内存的情况来看
​

栈内存：每个线程一块内存，单独的。
堆内存：多个线程共享内存，共享的，共用的。
元数据空间内存：多个线程共享内存，共享的，共用的。

![image.png](/image/java-memory-manage-seven.png)


**先来说说栈内存**
如何设置栈内存大小呢？

- -Xss <size> set java thread stack size 设置java每个栈的栈内存大小。

如果java程序运行期间超出了设置的栈内存大小呢？ 代码如下。
```java
public class Test {

    public static void main(String[] args) {
        leak();
    }

    public static void leak(){
        while (true) {
            leak();
        }
    }
}

// 报错如下
Exception in thread "main" java.lang.StackOverflowError
	......
```
**再来说下堆Heap内存**
如何设置堆内存大小呢？

- -Xms<size>        set initial Java heap size		设置java Heap初始大小
- -Xmx<size>        set maximum Java heap size	设置java Heap最大值

一般情况下，设置-Xms的值等于-Xmx的值
​

**最后来说下方法区、元数据空间内存**
如何设置元数据空间大小呢？
在JDK1.8中使用

- -XX:MetaspaceSize   设置元数据空间大小
- -XX:MaxMetaspaceSize 设置元数据空间最大内存

一般情况下，设置-XX:MetaspaceSize的值等于-XX:MaxMetaspaceSize的值
​

​

**留个思考题**
多个线程都会操作堆Heap区，多个线程在同一个Heap中申请内存，是如何保障多个线程相互不影响的呢？多个线程同时操作Heap，效率是如何保证的呢？

[在这里评论回复即可](https://github.com/gxpisme/gxpisme.github.io/issues/new)
