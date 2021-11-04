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

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636014406451-5c0b2b12-c5c0-48f6-9168-ca91d762f838.png#clientId=u06b34608-3eab-4&from=paste&height=272&id=uda29942b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=544&originWidth=414&originalType=binary&ratio=1&size=70833&status=done&style=none&taskId=u5669994c-9206-4b69-94cc-837b9a3316c&width=207)


至于Dog这个类，Dog这个类有自己的属性name，有自己的方法say。这些信息，同样有地方存储。存储的地方称为方法区，在JDK1.8又称为元数据空间。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636014326425-42f0fb77-b10d-42e1-b54b-3e482bfb04fa.png#clientId=u06b34608-3eab-4&from=paste&height=284&id=u3f266d30&margin=%5Bobject%20Object%5D&name=image.png&originHeight=568&originWidth=480&originalType=binary&ratio=1&size=52319&status=done&style=none&taskId=ubd0529f3-a05a-41f5-83d2-2f4ad83818f&width=240)

（yellow）大黄和（white）小白，这两个对象怎么知道自己属于哪个类呢？其实对象中会存在一个类型指针指向它的类型。用来知道（yellow）大黄属于Dog类的，知道（white）小白属于Dog类的。
​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636016366603-f86ad849-7d71-42fb-8690-0a6e4abde198.png#clientId=u06b34608-3eab-4&from=paste&height=296&id=u830c22d1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=592&originWidth=1008&originalType=binary&ratio=1&size=148574&status=done&style=none&taskId=u8edce17b-90bf-48bb-bd51-458692ad285&width=504)


在java中会有很多类，也会有很多对象，类信息都存在了方法区/元数据空间中，new出来的对象都存在了堆Heap中。
​

​

在java中，会有很多线程一起工作。所有线程使用到的类，new出来的对象，都分别存放在了方法区/元数据空间、堆Heap中。所以堆Heap和元数据空间是所有线程共享的，共同拥有的。
​

​

刚才讲了，java中会有很多线程，每个线程都会执行函数，函数的执行都是在栈Stack中进行的。还是最上面的例子`yellow.say();` 这些函数间的相互调用，都存放到栈中了，栈也是需要内存来存储的。
​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636015796588-bfd2e22c-3e31-40da-bcb4-bca6c3fc2804.png#clientId=u06b34608-3eab-4&from=paste&height=243&id=uc0a902f6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=486&originWidth=648&originalType=binary&ratio=1&size=117853&status=done&style=none&taskId=u19b38de6-e86b-404b-8fdb-5b8c8b9881c&width=324)


栈帧中就存储了变量、方法出口等信息。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636016187270-b23e4b93-8aef-47d1-93b5-97672eea2690.png#clientId=u06b34608-3eab-4&from=paste&height=239&id=u09c65188&margin=%5Bobject%20Object%5D&name=image.png&originHeight=478&originWidth=760&originalType=binary&ratio=1&size=140264&status=done&style=none&taskId=ube2b8ef9-d5ce-4a5d-be8e-4378d07d9bb&width=380)


栈帧中存储了对象指针，也就是Heap堆中的对象大黄的地址。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636016306597-68bbe26c-4f25-42a9-aac1-6b303c88d17e.png#clientId=u06b34608-3eab-4&from=paste&height=297&id=u87a7dd74&margin=%5Bobject%20Object%5D&name=image.png&originHeight=594&originWidth=1810&originalType=binary&ratio=1&size=370743&status=done&style=none&taskId=u60329245-92a6-4bc8-9874-1a84f1381ed&width=905)


​

在从内存的情况来看
​

栈内存：每个线程一块内存，单独的。
堆内存：多个线程共享内存，共享的，共用的。
元数据空间内存：多个线程共享内存，共享的，共用的。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1636016651700-6423da23-baa0-4e17-bb9d-18c0fe5661ea.png#clientId=u06b34608-3eab-4&from=paste&height=411&id=ub0c01d9f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=822&originWidth=1828&originalType=binary&ratio=1&size=543697&status=done&style=none&taskId=ue2ea179f-de15-470e-986c-0187656ef87&width=914)


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

