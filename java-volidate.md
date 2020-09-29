# Java Volidate 学习笔记

> volidate 在java中有两个作用，一是内存可见，二是禁止指令重排序。

## 内存可见
```
public class Demo {
    public static int n = 0;

    public static void main(String[] args) throws InterruptedException {

        // 线程one 对n++ 1亿次
        Thread one = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 100000000; i++) {
                    n++;
                }
            }
        });

        // 线程two 对n++ 1亿次
        Thread two = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 100000000; i++) {
                    n++;
                }
            }
        });

        // 线程one two开始执行
        one.start();
        two.start();

        // 线程one two等待线程结束
        one.join();
        two.join();

        // 输出结果 2亿？
        System.out.println(n);
    }
}
```

线程one和two分别对n加1亿次，那最终结果执行之后，发现结果小于2亿。

所以线程one或线程two取的值，一定是有旧的值，拿些旧的值进行操作，所以就会出现最终值小于2亿的情况。




### 不加volidate 内存不可见

### 加volidate内存可见

## 指令重排序


## 参考资料
[Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)
