
# Java类加载&PHP类加载

# 类加载
> 本质上都是用到某个类，然后找到某个类，最后加载某个类。
> 用某个类 -> 找某个类 -> 加载某个类



## PHP 类加载
一般都是采用魔术函数，php里面有__autoload()方法。<br />

```php
<?php
class a {
    public function __construct() {
        echo "from a class";
    }
}
```

```php
<?php
// 这里就是魔术方法，当找类找不到时，就会调用该方法。
// include_once 代表将该类加载，然后就能使用该类了。
function __autoload($name) {
    echo "----------- class >> ". $name . " << class --------- " . PHP_EOL;
    include_once($name . ".php");
}

new a();
```

```php

[xpisme@aliyun /home/xpisme] ls
a.php b.php
// 命令行直接执行b.php
[xpisme@aliyun /home/xpisme] php b.php
----------- class >> a << class ---------
from a class

```

<br />虽然上面这些都是demo级别的，但是原理确实如此，通过魔术方法来实现的。<br />

> 在php大项目中，很多目录，很多文件，在加载的时候，如何能找到正确的位置呢？其实是使用命名空间的，这样就能够准确定位到某个类的位置，然后加载进来。



> 除了__autoload魔术方法，自己也可以进行定义特定的函数，然后用spl_autoload_register进行注册，自己定义的就可以生效了。



1. 用某个类  `new a()`
2. 找某个类  `__autoload($className)`
3. 加载某个类 `require_once($class)`


## Java类加载

每个类都有自己的ClassLoader，当用到某个类的时候，就会启用该类的ClassLoader进行类加载。

```java
Object obj = new Object();
// 对应的类加载 obj.getClass().getClassLoader();

Integer integer = new Integer(1);
// 对应的类加载 integer.getClass().getClassLoader();        
```

`xx.getClass().getClassLoader()` 得到的是一个ClassLoader类，这个ClassLoader类用来寻找某个类，并且加载某个类。

所以，ClassLoader类中有两个重要的函数：一个是寻找某个类`findClass`，另一个是加载某个类`loadClass`。
伪代码应该是

```java
findClass(); // 找到某个类
loadClass(); // 加载某个类
```

但是java中有很多自己的类，比如：`java.lang.Object` `java.lang.Integer` `java.lang.Exception` 等等。

如果用户自己也定义一个`java.lang.Object` 类，那系统中就会出现多个不同的Object类，到底以哪个为准呢？Java类型体系中最基础的无从保证，所以就需要一套机制来保证java程序稳定运行。


- 将这些基础类，用一种类来加载，这个类加载只能java使用，而不能是某个用户或使用者，一般用户或使用者基本都编写java程序，这个基础的类加载用C++来编写。**（离用户最远）**

- java中有一些通用的扩展等，用扩展类加载，将类进行区分开，不同用处的类用不同的类加载，保证不会冲突。**（离用户较远）**

- java中剩下的就是我们编写的代码，用类加载，这个类加载只用于加载我们编写的代码，这个比较通用，一般也是系统给写好的。**（离用户较近）**

- java中我们自己去定义加载某些类呢？用自定义的类加载，自定义加载哪些类。**（离用户最近）**

整体看下来，分了这四层，但是这四层如何进行协作呢？毕竟都在一个项目里运行。

如果要用到某个类，就加载该类到程序中。先判断是否已经加载了，如果没有加载则加载类，这好像是废话，具体看下伪代码吧。假设要用到类A。

```java
if (离用户最近.hasLoad) {
    // 最近已经加载了，太好了。
    return;
} else if (离用户较近.hasLoad) {
    // 较近已经加载了，太好了。
    return;
} else if (离用户较远.hasLoad) {
    // 较远已经加载了，太好了。
    return;
} else if (离用户最远.hasLoad) {
	// 最远已经加载了，太好了。
    return;
}

// 这个查看类是否加载的过程顺序是 【最近 -> 较近 -> 较远 -> 最远】
// 走到这里，发现各个层次的类加载器都没有加载。 
// 那么接下来就开始加载类

if (离用户最远.loadClass) {
    // 最远加载成功
} else if (离用户较远.loadClass) {
    // 较远加载成功
} else if (离用户较近.loadClass) {
    // 较近加载成功
} else if (离用户最近.loadClass) {
    // 最近加载成功
}

// 这里加载类的顺序是 【最远 -> 较远 -> 较近 -> 最近】
```

<br />其中每个类加载器都有parent（除了离用户最远的类加载器），用于实现这种上下关系。大概意思就是，看类是否加载，找该类加载是否已经加载，没有找到则找**加载的父类**加载，加载的父类加载没有的话，就找**父类加载**的**父类**加载，以此类推。假设都找完了，还是没有，那么从根加载器，看能否加载，如果不能，则一直找到其子类进行加载。<br />
<br />上面的demo对应到真实的java中

- 离用户最近 - CustomClassLoader（自定义类加载）（Java编写）（parent是AppClassLoader）
- 离用户较近 - AppClassLoader（应用类加载）（Java编写）(parent是ExtClassLoader)
- 离用户较远 - ExtClassLoader（扩展类加载）（Java编写）(parent是BootstrapClassLoader)
- 离用户最远 - BootstrapClassLoader（启动类加载）（C++编写）（parent是null）


上面其实就是java中的双亲委派模型（Parents Delegation Model）
工作流程就是一个类加载器收到类加载请求，它首先不会自己尝试加载这个类，而是会把请求委托给父类的加载器来完成。因此所有的加载请求最终都会传送到顶部的启动类加载器。


<br />接下来看下精简核心的源代码ClassLoader类。

```java
    protected Class<?> loadClass(String name, boolean resolve) {
            // 首先，检查类是被加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                // 看下该父类是否为null，不为null，则用父类进行加载
                if (parent != null) {
                    // 这里是递归的。
                    // 顺序 Bootstrap -> Ext -> App -> Custom
		    c = parent.loadClass(name, false);
                } else {
                    // 父类为null，就是C++编写的类加载了。
                    c = findBootstrapClassOrNull(name);
                }

                if (c == null) {
                    // 如果仍然没有找到，则调用findClass去查找类并加载
                    // 因为loadClass整体是递归的，这个findClass的顺序和之前正好相反
                    // Custom -> App -> Ext -> Bootstrap
                    c = findClass(name);
                }
            }
    }
```

<br />简单理解就可以按照上面所描述的进行理解。<br />

### 代码验证
```java
public class One {
    public static void main(String[] args) {
        // 获取该类的ClassLoader
        ClassLoader classLoader = One.class.getClassLoader();
        // 如果classLoader有值，则就一直找classLoader的parent
        while (classLoader != null) {
            classLoader = classLoader.getParent();
            System.out.println(classLoader.getClass().getName());
        }
        System.out.println("-----");
        System.out.println(classLoader);
    }
}
```
输出结果

```java
[xpisme@aliyun /home/xpisme] java One
sun.misc.Launcher$AppClassLoader
sun.misc.Launcher$ExtClassLoader
-----
null
```
可以看到先从 AppClassLoader（应用类加载），然后到ExtClassLoader（扩展类加载），最后到BootstrapClassLoader（启动类加载）。


接下来，若将One.class 打包为One.jar 放到jre/lib/ext 下，然后再执行

`jar cvf One.jar One.class`


```java
[xpisme@aliyun /home/xpisme] java One
sun.misc.Launcher$ExtClassLoader
-----
null
```
可以看到可以看到先从到ExtClassLoader（扩展类加载），然后到BootstrapClassLoader（启动类加载）。这是因为把One.jar放到ExtClassLoader加载的目录下了。



## Java类的唯一性
对于任意一个类，都必须由加载它的类加载器和类本身一起共同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。
比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个Java虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。

若一个类，被A类加载器加载了一遍，B类加载器也加载了一遍。这个类在java虚拟机中其实是两个，并不相等。只有类加载器+类才能确定唯一性。
