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
        // 启动线程A
        Thread A = new Thread(new Runnable() {
            @Override
            public void run() {
                // 从主内存中读取flag值 读取到工作内存中 若flag为true 则一直执行
                while (flag) {
                }
                // 若flag为false 那么done能打印出来了
                System.out.println("done");
            }
        }, "A");

        A.start();

        // 当前线程设置flag为false，那么主内存中flag也就为false了。
        flag = false;
    }
}
```

该代码永远也不会输出done，因为工作线程`A`不会从主内存中再去加载flag值，因此工作线程中flag值一直为true。

这种就是工作线程A对主内存不可见，主内存修改值，工作线程不知道，不可见。


### 加volatile内存可见

```
public class Demo {

    public static volatile boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        // 启动线程A
        Thread A = new Thread(new Runnable() {
            @Override
            public void run() {
                // 从主内存中读取flag值 读取到工作内存中 若flag为true 则一直执行
                while (flag) {
                }
                // 若flag为false 那么done能打印出来了
                System.out.println("done");
            }
        }, "A");

        A.start();

        // 当前线程设置flag为false，那么主内存中flag也就为false了。
        flag = false;
    }
}
```

该代码会输出done，因为工作线程`A`会从主内存中再去加载flag值，因此当`main`线程修改flag为false时，那么主内存flag就为false了。

由于flag是用volatile来修饰的，所以工作线程`A`中flag会被通知失效，重新会从主内存中拉取该flag值。

此时就实现了内存可见

## 指令重排序
```
public class ReOrder {
    private static int x = 0, y = 0;
    private static int a = 0, b = 0;

    public static void main(String[] args) throws InterruptedException {
        int i = 0;
        while (true){
            i++;
            x = 0;
            y = 0;
            a = 0;
            b = 0;
            Thread one = new Thread(new Runnable() {
                public void run() {
                    a = 1;
                    x = b;
                }
            });

            Thread other = new Thread(new Runnable() {
                public void run() {
                    b = 1;
                    y = a;
                }
            });
            one.start();
            other.start();
            one.join();
            other.join();
            String result = "第" + i + "次 (" + x + "," + y + "）";
            if (x == 0 && y == 0) {
                System.err.println(result);
                break;
            }
        }
    }
}
```

分析下代码

每次循环时，默认x=0,y=0,a=0,b=0。

线程one 设置a=1,x=b;  线程other 设置b=1,y=a

然后线程one和线程other开始执行。

如果线程one先执行，线程other再执行。顺序为`a=1,x=b,b=1,y=a`，那么结果为x=0,y=1

如果线程other先执行，线程one再执行。顺序为`b=1,y=a,a=1,x=b`，那么结果为x=1,y=0

如果线程one和线程other同时执行，顺序为`a=1,b=1,x=b,y=a`（`a=1,b=1`可互换，`x=b,y=a`可互换）,那么结果为x=1，y=1。

但是无论上述哪种情况，结果都不会出现x=0,y=0的情况。

如果出现这种情况，执行顺序应该为`x=b,y=a,b=1,a=1`, 但是线程one设置顺序a=1,x=b; 线程other设置顺序b=1,y=a，

那么肯定发生了线程one x=b先于a=1执行 或者 线程other y=a先于b=1执行。

但是实际运行中确实出现了x=0,y=0的情况，就证明了重排序的情况。（我自己操作中出现了 第1363751次 (0,0））

若用volatile来修饰x、y、a、b，就永远不会出现x=0,y=0的情况。


### 为什么有指令重排序？
> 为了压榨CPU，提高CPU利用率。

现在的CPU一般采用流水线来执行指令。一个指令的执行被分成：取指、译码、访存、执行、写回、等若干个阶段。然后，多条指令可以同时存在于流水线中，同时被执行。

**指令流水线并不是串行的**，并不会因为一个耗时很长的指令在“执行”阶段呆很长时间，而导致后续的指令都卡在“执行”之前的阶段上。

相反，**流水线是并行的**，多个指令可以同时处于同一个阶段，只要CPU内部相应的处理部件未被占满即可。

### 为什么在单例中会用到volatile



## 参考资料
[Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)
