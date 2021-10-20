# Java基本类型和包装类
> 先了解下基础，有个大概印象

## 包装类


- java 有八种基本类型，每种基本类型都有对应的包装类。
- 那包装类又是什么呢？简单来讲，包装类就是针对每种基本类型进行包装了下，提供更好的一些方法进行使用。
- java中很多代码只能操作对象，操作对象就操作包装类吧，包装类对基本类型都包装了下，所以包装类还是很重要的吧。



> **下面是java中基本类型和对应的包装类**

| 基本类型 | 包装类 |
| --- | --- |
| boolean | Boolean |
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |



## 基本类型和包装类的转换
**Integer**
```java
// 基本类型
int i1 = 12345;
// 基本类型转化为包装类 将基本类型通过 Integer.valueOf方法转换为 包装类
Integer iObj = Integer.valueOf(i1);
// 包装类转化为基本类型 包装类的intValue方法装换为基本类型
int i2 = iObj.intValue();
```

<br />其他类型也是如此。<br />

### 装箱
基本类型转换为包装类

- 包装类 = new 包装类(值);


<br />**从基本类型到包装类。 把基本类型装到包装类中，就是装箱。**<br />

### 拆箱
包装类转换为基本类型

- 基本类型 = 包装类.xxxValue();


<br />**从包装类到基本类型。把包装类中搞出基本类型，就是拆箱。**<br />

# 自动装箱/拆箱
## 自动装箱
在JAVA SE5之前若要生成一个数值为10的Integer对象，必须new Integer才可。<br />例如下面代码
```java
Integer i = new Integer(10);
```

<br />但是在JAVA SE5之后，直接执行如下代码就可生成数值为10的Integer对象<br />

```java
Integer i = 10;
```


## 自动拆箱
同样也是JAVA SE5进行的自动拆箱操作。<br />JAVA SE5之前
```java
Integer i = new Integer(10); // 装箱
int j = i.intValue();		 // 拆箱
```

<br />JAVA SE5之后
```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```


# 源码剖析 自动装箱/拆箱实现


## 源码
```java
public class Main {
    public static void main(String[] args) {
        Integer i = 10; //装箱操作
        int n = i;		//拆箱操作
    }
}
```


## 编译
```java
javac Main.java
```


## 反编译
> 可以看到第13行，实际上调用了Integer.valueOf方法   编译实现了装箱操作
> 可以看到第16行，实际上调用了Integer.intValue方法  编译实现了拆箱操作



```java
 mac@MacBook-Pro  ~/study/java  javap -c Main
Compiled from "Main.java"
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: bipush        10
       2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1
       6: aload_1
       7: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
      10: istore_2
      11: return
}
```


# 包装类.valueOf() 和 new 包装类()
可以看下Integer的实现<br />

```java


	public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }


    public Integer(int value) {
        this.value = value;
    }
```


- 其实Integer.valueOf实际上是调用了new Integer，本质上都调用了 new Integer
- 但是尽量调用valueOf方法，因为有些判断，Integer如果在 -128到127之间，可以直接使用内存中的数据，这些数据在内存中已经有了，就不用再new了。这种使用方式在redis中也有用到。


<br />

```java
看代码
public class Main {
    public static void main(String[] args) {
        System.out.println(new Integer(1000) == 1000);
    }
}

看反编译的代码
mac@MacBook-Pro ~/study/java javap -c Main
Compiled from "Main.java"
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: new           #3                  // class java/lang/Integer
       6: dup
       7: sipush        1000
      10: invokespecial #4                  // Method java/lang/Integer."<init>":(I)V
      13: invokevirtual #5                  // Method java/lang/Integer.intValue:()I  这里是自动拆箱
      16: sipush        1000
      19: if_icmpne     26
      22: iconst_1
      23: goto          27
      26: iconst_0
      27: invokevirtual #6                  // Method java/io/PrintStream.println:(Z)V
      30: return
}
```


## 考察IntegerCache


```java
public class Main {
    public static void main(String[] args) {
        Integer a = 100;
        Integer b = 100;
        Integer c = 200;
        Integer d = 200;
        System.out.println(a == b);
        System.out.println(c == d);
    }
}
```

<br />![image.png](/image/mynote/java_auto_package_image.png)<br />Integer如果在 -128到127之间，可以直接使用内存中的数据。<br />变量 a b 指向的都是同一个地址。<br />变量 c d 指向不同的地址。
