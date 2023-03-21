# java动态代理 AOP原理


# 静态代理
## 基础的类

### 定义接口
```
public interface IService {
    public void sayHello();
}
```
### 定义类（一定要实现IService接口）
```
/**
 * 正常的情况下，只要直接调用这个就可以了。
 */
public class RealService implements IService {
    @Override
    public void sayHello() {
        System.out.println("say Hello");
    }
}
```

### 定义代理类(一定要实现IService接口)
```

/**
 * 代理类必须实现Iservice接口
 */
public class TraceProxy implements IService {
    /**
     * 存储真正类的对象(现实的情况是 直接new该对象，然后进行调用)
     * 
     * 但是咱们想通过代理来实现
     */
    private IService realService;

    /**
     * 创建的时候，就把该真正的对象放到该代理类的一个属性里。
     * @param realService
     */
    public TraceProxy(IService realService) {
        this.realService = realService;
    }

    /**
     * 由于实现了IService接口，所以一定会有和真正类相同的接口，因为他们都实现了IService接口。
     * 在代理类调用的时候也非常方便
     */
    @Override
    public void sayHello() {
        // 加一些操作：比如加个log
        System.out.println("entering sayHello");
        // 这里才是 真正类 调用的地方
        this.realService.sayHello();
        // 加一些操作：比如加个log
        System.out.println("leaving sayHello");
    }
}

```

## 调用的地方
```
public class SimpleStaticProxyDemo {

    /**
     * 调用的地方
     * @param args
     */
    public static void main(String[] args) {
        // 创建需要代理的对象
        IService realService = new RealService();
        // 创建代理
        TraceProxy traceProxy = new TraceProxy(realService);
        // 代理进行访问
        traceProxy.sayHello();
    }
}
```

## 输出
```
entering sayHello
say Hello
leaving sayHello
```

## 总结
- 因为上述都是静态编码的，每次针对一个接口有**额外操作**的时候，都需要编码。上面这种说的情况就是静态代理。
- 如果能够针对某一些特殊的类或者接口能够动态代理增加**额外操作**，对业务代码无侵入，并且还能够**额外操作**，岂不完美。

> 看下接下来的动态代理的原理，以及实现。

# 动态代理
> 动态代理分为两种，分别是JDK动态代理和CGLIB动态代理

## JDK动态代理
### 基础的类
```
// 这是定义的接口
public interface IService {
    public void sayHello();
}
```
```
/**
 * 正常的情况下，只要直接调用这个就可以了。
 */
public class RealService implements IService {
    @Override
    public void sayHello() {
        System.out.println("hello");
    }
}
```
```
/**
 * 静态代理：之前是固定的一个代理类，需要实现统一interface的
 *
 * 需要实现InvocationHandler，其实里面只有一个invoke方法。
 *
 */
public class SimpleInvocationHandler implements InvocationHandler {
    /**
     * 静态代理这里是指定的Interface
     *
     * 但这里动态代理是Object用来代表
     * 该类属性 存储真正类的对象(现实的情况是 直接new该对象，然后进行调用)
     */
    private Object realObject;

    /**
     * 该对象创建的时候就把对象放进来
     *
     * @param realObject
     */
    public SimpleInvocationHandler(Object realObject) {
        this.realObject = realObject;
    }

    /**
     * 统一的调用方法
     * @param proxy 代理，代码里没有用到
     * @param method 需要调用的方法
     * @param args 调用方法传的参数
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("entering " + method.getName());
        Object result = method.invoke(realObject, args);
        System.out.println("leaving " + method.getName());
        return result;
    }
}
```

### 调用的地方
```
public class SimpleJDKDynamicProxyDemo {

    public static void main(String[] args) {
        // 创建一个真正的对象
        RealService realService = new RealService();

        /**
         * 创建代理对象
         *
         * 强制转化为IService RealService实现了这个IService
         */
        IService proxyInstance = (IService)Proxy.newProxyInstance(
                // loader 表示类加载器, 使用和IService一样的类加载器
                IService.class.getClassLoader(),
                // interfaces 表示代理类要实现的接口列表，是一个数组，元素的类型只能是接口，不能是普通的类，IService是一个接口
                new Class<?>[]{IService.class},
                // 第三个参数类型是InvocationHandler，它是一个接口，它只包含了一个方法invoke，对代理接口所有方法的调用都会转给该方法
                // SimpleInvocationHandler对标的是静态代理类的TraceProxy
                new SimpleInvocationHandler(realService)
        );

        // 代理类调用
        proxyInstance.sayHello();
    }
}
```

#### Proxy.newProxyInstance
这个其实就是创建一个代理对象，这个代理对象时动态创建的。
这个代理类其实就是获取构造方法，创建代理对象。

### 动态代理的使用
使用它，可以编写通用的代理逻辑，用于各种类型的被代理对象，而不需要为每个被代理的类型都创建一个静态代理类。

```
// 定义两个接口
public interface IServiceA {
    public void sayHello();
}

public interface IServiceB {
    public void fly();
}

```

```
// 两个接口对应两个类的实现
public class ServiceAImpl implements IServiceA {
    @Override
    public void sayHello() {
        System.out.println("hello");
    }
}

public class ServiceBImpl implements IServiceB {
    @Override
    public void fly() {
        System.out.println("fly");
    }
}

```

```
// 针对类/接口的附加操作
public class SimpleInvocationHandler implements InvocationHandler {
    private Object obj;

    public SimpleInvocationHandler(Object obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("entering " + obj.getClass().getSimpleName() + " :: " + method.getName());
        Object res = method.invoke(obj, args);
        System.out.println("leaving " + obj.getClass().getSimpleName() + " :: " + method.getName());
        return res;
    }
}

```


```
// 统一调用，动态创建代理类，代理类调用
public class Demo {
    public static void main(String[] args) {
    
    	// new该对象，然后将该对象放到代理类中，然后用代理类调用
        ServiceAImpl serviceA = new ServiceAImpl();
        IServiceA proxyInstanceA = (IServiceA)Proxy.newProxyInstance(
                IServiceA.class.getClassLoader(),
                new Class<?>[]{IServiceA.class},
                new SimpleInvocationHandler(serviceA)
        );
        proxyInstanceA.sayHello();

        ServiceBImpl serviceB = new ServiceBImpl();
        IServiceB proxyInstanceB = (IServiceB) Proxy.newProxyInstance(
                IServiceB.class.getClassLoader(),
                new Class<?>[]{IServiceB.class},
                new SimpleInvocationHandler(serviceB)
        );
        proxyInstanceB.fly();
    }
}
```

### JDK动态代理的使用范围
看上面的各种情况，能够看出来只是针对接口做的代理。
实现的原理就是动态创建一个个代理类。

### 问答
- 问题1：能直接对类进行代理吗？
- 答案1：不能。只能作用于接口类的实现。
- 问题2：JDK动态代理是基于什么实现的？
- 答案2：基于动态创建一个个代理类来实现的。

## CGLIB动态代理
### 看简单代码实现
```
// 真正的类，只含有一个函数
public class RealService {
    public void sayHello() {
        System.out.println("hello");
    }
}
```

```
// 简单的方法拦截类，
public class SimpleInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("1");
        
        // 通过该函数，调用原始方法
        Object result = methodProxy.invokeSuper(o, args);
        
        System.out.println("2");
        return result;
    }
}
```

```
public class Demo {

    public static void main(String[] args) {
    	// 其实目的就是创建一个子类对象实例
        Enhancer enhancer = new Enhancer();
        // 设置父类是RealService
        enhancer.setSuperclass(RealService.class);
        // 设置回调是拦截类（拦截类中有调用原始方法的实现）
        enhancer.setCallback(new SimpleInterceptor());
        // 创建子对象实例
        RealService realService = (RealService) enhancer.create();
        // 开始调用
        realService.sayHello();
    }
}
```

### 总结
- Java SDK动态代理的局限在于，它只能为接口创建代理，返回的代理对象也只能转换到某个接口类型
- cglib的实现机制与Java SDK不同，它是通过继承实现的，它也是动态创建了一个类，但这个类的父类是被代理的类，代理类重写了父类的所有public非final方法，改为调用Callback中的相关方法
- 从代理的角度看，Java SDK代理的是对象，需要先有一个实际对象，自定义的InvocationHandler引用该对象，然后创建一个代理类和代理对象，客户端访问的是代理对象，代理对象最后再调用实际对象的方法，cglib代理的是类，创建的对象只有一个。


#### 如何理解：SDK代理的是对象，cglib代理的是类，创建的对象只有一个？
SDK代理时，会有个对象就创建个代理对象。
cglib代理时，有个类就会创建个子类。

假设有一个类Apple。

SDK代理时，A = SDKProxy(Apple), B = SDKProxy(Apple) 会创建两个**Apple实例**，两个代理实例
（如果创建N个，那么就会存在**N个Apple**实例，并且会存在N个代理实例）

cglib代理时，A = cglibProxy(Apple), B = cglibProxy(Apple) 会创建两个**Apple子实例**。
（如果创建N个，就会存在**N个子Apple**实例，其实**不会有Apple实例**的）


# AOP 基于 CGLIB 的简单实现

> **强烈建议自己手写一遍**
> 
> **强烈建议自己手写一遍**
> 
> **强烈建议自己手写一遍**

```
public class A {
    private String singer = "果冻";

    public Integer sing(String name) {
        System.out.println(name + " " + singer + " sing");
        return 123;
    }
}
```

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAspect {
    Class<?>[] value();
}
```

```
@MyAspect({A.class})
public class ServiceLogAspect {

    public static void before(Object object, Method method, Object[] args) {
        System.out.println("entering " + method.getDeclaringClass().getSimpleName() + "::" + method.getName() + " , args: " + Arrays.toString(args));
    }

    public static void after(Object object, Method method, Object[] args, Object result) {
        System.out.println("leaving " + method.getDeclaringClass().getSimpleName() + "::" + method.getName() + " , result: " + result);
    }
}
```

```
public class CGLibContainer {

    /**
     * 定义拦截点
     */
    public static enum InterceptPoint {
        BEFORE, AFTER
    }

    /**
     * aspects 记录了拦截类
     */
    private static Class<?>[] aspects = new Class<?>[] {
            ServiceLogAspect.class
    };

    /**
     * interceptMethodsMap 记录了拦截的方法
     */
    private static Map<Class<?>, Map<InterceptPoint, List<Method>>> interceptMethodsMap = new HashMap<>();

    static {
        // 这里在类加载的时候就会执行  aspects是所有的拦截类实现
        for (Class<?> cls : aspects) {

            // 查询拦截类是否有对类进行绑定
            MyAspect aspect = cls.getAnnotation(MyAspect.class);
            if (aspect != null) {
                // 不为空 就是有绑定

                // 从拦截类中获取方法
                Method before = null;
                try {
                    before = cls.getMethod("before", new Class<?>[]{Object.class, Method.class, Object[].class});
                } catch (NoSuchMethodException e) {
                }
                Method after = null;
                try {
                    after = cls.getMethod("after", new Class<?>[]{Object.class, Method.class, Object[].class, Object.class});
                } catch (NoSuchMethodException e) {
                }

                // 拦截类绑定的一些类
                Class<?>[] intercepttedArr = aspect.value();

                // 遍历类，将类中的方法 需要拦截的 存储下来
                // 若类中有个方法，这个方法都会被BEFORE、AFTER拦截
                for (Class<?> interceptted : intercepttedArr) {
                    addInterceptMethod(interceptted, InterceptPoint.BEFORE, before);
                    addInterceptMethod(interceptted, InterceptPoint.AFTER, after);
                }
            }
        }
    }

    /**
     * 添加拦截方法
     *
     * @param cls
     * @param point
     * @param method
     */
    private static void addInterceptMethod(Class<?> cls, InterceptPoint point, Method method) {
        if (method == null) {
            return;
        }

        Map<InterceptPoint, List<Method>> map = interceptMethodsMap.get(cls);
        if (map == null) {
            map = new HashMap<>();
            interceptMethodsMap.put(cls, map);
        }
        List<Method> methods = map.get(point);
        if (methods == null) {
            methods = new ArrayList<>();
            map.put(point, methods);
        }

        methods.add(method);
    }

    /**
     * 获取拦截方法
     * @param cls
     * @param point
     * @return
     */
    public static List<Method> getInterceptMethods(Class<?> cls, InterceptPoint point) {
        Map<InterceptPoint, List<Method>> map = interceptMethodsMap.get(cls);
        if (map == null) {
            return Collections.emptyList();
        }

        List<Method> methods = map.get(point);
        if (methods == null) {
            return Collections.emptyList();
        }
        return methods;
    }

    static class AspectInterceptor implements MethodInterceptor {
        @Override
        public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
            // 执行before方法  从拦截点获取拦截的方法
            List<Method> beforeMethods = getInterceptMethods(o.getClass().getSuperclass(), InterceptPoint.BEFORE);
            for (Method m : beforeMethods) {
                m.invoke(null, new Object[]{o, method, args});
            }

            try {
                // 调用原始方法
                Object result = methodProxy.invokeSuper(o, args);

                // 执行after方法 从拦截点获取拦截的方法
                List<Method> afterMethods = getInterceptMethods(o.getClass().getSuperclass(), InterceptPoint.AFTER);
                for (Method m : afterMethods) {
                    m.invoke(null, new Object[]{o, method, args, result});
                }

                return result;
            } catch (Throwable e) {
                throw e;
            }
        }
    }

    @SneakyThrows
    public static <T> T getInstance(Class<T> cls) {
        T obj;
        if (!interceptMethodsMap.containsKey(cls)) {
            // 如果不包含，证明没有拦截到，则正常创建类，跟平常一样
            obj = cls.newInstance();
        } else {
            // 如果包含，证明拦截到了，就如下创建（原理还是创建了子类）
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(cls);
            enhancer.setCallback(new AspectInterceptor());
            obj = (T) enhancer.create();
        }

        return obj;
    }

}
```

# 参考资料
- [Java编程的逻辑 (86) - 动态代理](https://www.cnblogs.com/swiftma/p/6869790.html)
- [Java帝国之动态代理](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513926&idx=1&sn=1c43c5557ba18fed34f3d68bfed6b8bd&chksm=80d67b85b7a1f2930ede2803d6b08925474090f4127eefbb267e647dff11793d380e09f222a8&scene=21#wechat_redirect)
- [从兄弟到父子：动态代理在民间是怎么玩的？](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513980&idx=1&sn=a7d6145b13270d1768dc416dbc3b3cbd&chksm=80d67bbfb7a1f2a9c01e7fe1eb2b3319ecc0d210a88a1decd1c4d4e1d32e50327c60fa5b45c8&scene=21#wechat_redirect)
