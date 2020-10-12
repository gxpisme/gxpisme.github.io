# Java volatile 学习笔记

> volatile 在java中有两个作用，一是内存可见，二是禁止指令重排序。

## 内存可见

### JMM (Java Memory Model) java内存模型

![](/image/java-volatile-memory-visible.jpg)

### 不加volatile 内存不可见
```
public class Demo {
    public static boolean flag = true;

    public static void main(String[] args) throws InterruptedException {

        // 启动一个线程
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                // 从主内存中读取flag值 若flag为true 则一直执行
                while (flag) {
                }
                // 若flag为true 那么done能打印出来了
                System.out.println("done");
            }
        });

        thread.start();
        
        // 当前线程设置flag为false，那么主内存中flag也就为false了。
        flag = false;
    }
}
```

该代码永远也不会输出done，因为工作线程`thread`不会从主内存中再去加载flag值，因此工作线程中flag值一直为true。

这种就是工作线程对主内存不可见，主内存修改值，工作线程不知道，不可见。


### 加volatile内存可见

```
public class Demo {
    public static volatile boolean flag = true;

    public static void main(String[] args) throws InterruptedException {

        // 启动一个线程
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                // 从主内存中读取flag值 若flag为true 则一直执行
                while (flag) {
                }
                // 若flag为true 那么done能打印出来了
                System.out.println("done");
            }
        });

        thread.start();
        
        // 当前线程设置flag为false，那么主内存中flag也就为false了。
        flag = false;
    }
}
```

该代码会输出done，因为工作线程`thread`会从主内存中再去加载flag值，因此当`main`线修改flag为false时，

由于flag是用volatile来修饰的，所以工作线程`thread`中flag会通知失效，重新会从主内存中拉取该flag值。





## 指令重排序


## 参考资料
[Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)
