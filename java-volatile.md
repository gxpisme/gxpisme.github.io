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
        
        // 设置主内存的flag为true
        flag = false;
    }
}
```
该代码中有两个线程，一个是主线程，一个是子线程。在主线程中设置`flag=false`，在主内存flag为false。

但是在子线程中，flag值一直未true，因为子线程不会主内存中再去加载flag值。

这种就是工作子线程对主内存不可见，主内存修改值，工作线程不知道，不可见。

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
        
        // 设置主内存的flag为true
        flag = false;
    }
}
```




## 指令重排序


## 参考资料
[Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)
