# Java volatile 学习笔记

> volatile 在java中有两个作用，一是内存可见，二是禁止指令重排序。

## 内存可见

![](/image/java-volatile-memory-visible.jpg)

### 不加volatile 内存不可见
```
public class See {
    public static volatile boolean flag = true;

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (flag) {
                    System.out.println("running");
                }
                System.out.println("done");
            }
        });

        thread.start();

        flag = false;
    }
}
```


### 加volatile内存可见

## 指令重排序


## 参考资料
[Java内存访问重排序的研究](https://tech.meituan.com/2014/09/23/java-memory-reordering.html)
