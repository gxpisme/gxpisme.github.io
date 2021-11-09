# java常用工具
# 随身携带的命令行工具

>  与java同一个目录下会有好多工具，接下来学习下常用的工具。

![image.png](/image/mynote/java_common_tools_1.png)


大部分命令，都可以 `命令 -h` 来查看帮助信息。
​

## `java` 查看一些默认值
`java -XX:+PrintFlagsFinal -version` 会列出所有的参数。


## `jps` 查看正在运行的java进程

`jps`  输出java进程ID

![image.png](/image/mynote/java_common_tools_2.png)
​

`jps -v`  输出java进程启动时的参数，这里的参数能够看出来堆大小、元空间大小、使用什么收集器。


![image.png](/image/mynote/java_common_tools_3.png)


## `jinfo` 查看java配置信息
`jinfo pid` 输出该pid进程下的所有配置信息
​

## `jstat` 统计信息

可以统计很多信息

`jstat -options` 可以查看可以统计哪些信息

```java
 ~/study  jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```
​

最常用的就是看gc的统计信息。

### ​`jstat -gc 59559 2000 5` 统计pid为59559的gc信息，每2000毫秒（2s）统计一次，统计5次。

```java
 ~/study  jstat -gc 59559 2000 5
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
10752.0 10752.0  0.0    0.0   65536.0   4604.6   70144.0     3996.8   21248.0 20678.2 2560.0 2340.4      2    0.027   1      0.078    0.105
10752.0 10752.0  0.0    0.0   65536.0   4604.6   70144.0     3996.8   21248.0 20678.2 2560.0 2340.4      2    0.027   1      0.078    0.105
10752.0 10752.0  0.0    0.0   65536.0   4604.6   70144.0     3996.8   21248.0 20678.2 2560.0 2340.4      2    0.027   1      0.078    0.105
10752.0 10752.0  0.0    0.0   65536.0   4604.6   70144.0     3996.8   21248.0 20678.2 2560.0 2340.4      2    0.027   1      0.078    0.105
10752.0 10752.0  0.0    0.0   65536.0   4604.6   70144.0     3996.8   21248.0 20678.2 2560.0 2340.4      2    0.027   1      0.078    0.105

比较常用：
 S0C   survivor0区的容量
 S1C   survivor1区的容量
 S0U   survivor0区已使用容量
 S1U   survivor0区已使用容量
 EC    eden区容量
 EU    eden区已使用容量
 OC    old区容量
 OU    old区已使用容量
 MC    metspace容量
 MU    metspace已使用容量
 CCSC  压缩类容量
 CCSU  压缩类已使用容量
 YGC   YoungGC次数
 YGCT  YoungGC总耗时
 FGC   FullGC次数
 FGCT  FullGC总耗时
 GCT   GC总耗时
```
### `jstat -gcutil 59559 2000 5` 统计pid为59559的gc信息，每2000毫秒（2s）统计一次，统计5次。
**-gcutil gc常用指标信息**
​

```java
~/study  jstat -gcutil 59559 2000 5
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00   7.03   5.70  97.32  91.42      2    0.027     1    0.078    0.105
  0.00   0.00   7.03   5.70  97.32  91.42      2    0.027     1    0.078    0.105
  0.00   0.00   7.03   5.70  97.32  91.42      2    0.027     1    0.078    0.105
  0.00   0.00   7.03   5.70  97.32  91.42      2    0.027     1    0.078    0.105
  0.00   0.00   7.03   5.70  97.32  91.42      2    0.027     1    0.078    0.105

S0     survivor0区使用容量百分比
S1     survivor1区使用容量百分比
E      eden区使用容量百分比
O      old区使用容量百分比
M      metaspace使用容量百分比
CCS    压缩类使用容量百分比
YGC    YoungGC次数
YGCT   YoungGC总耗时
FGC    FullGC次数
FGCT   FullGC总耗时
GCT    GC总耗时
```
## `jstack` 查看栈信息
`jstack pid` 输出该pid进程下的栈信息
​

## `jmap` 查看Heap堆信息
第一步生成Heap文件
`jmap -dump:live,format=b,file=heap.bin pid`
```java
 ~/study  jmap -dump:live,format=b,file=heap.bin 59559
Dumping heap to /Users/gxp/study/heap.bin ...
Heap dump file created
```
第二步分析Heap文件
`jhat heap.bin`
```java
 ~/study  jhat heap.bin
Reading from heap.bin...
Dump file created Tue Nov 09 17:26:33 CST 2021
Snapshot read, resolving...
Resolving 79950 objects...
Chasing references, expect 15 dots...............
Eliminating duplicate references...............
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```
上面日志显示`Server is ready.`  [http://localhost:7000/](http://localhost:7000/) 就能查看相关信息。
​

# 可视化工具
VisualVM 这个工具，可以将上面导出的堆文件`heap.bin`导进去进行分析。

![image.png](/image/mynote/java_common_tools_4.png)


# Arthas
> 阿里巴巴开源的一个java诊断工具 [https://github.com/alibaba/arthas](https://github.com/alibaba/arthas)

> 用户手册：[https://arthas.aliyun.com/doc/](https://arthas.aliyun.com/doc/)



# 常见问题排查
## Java应用 CPU 过高如何排查

1. `top` 确定最耗cpu的进程。
2. `top -Hp PID（进程ID）`，显示一个进程ID的线程运行信息列表 (按键`P` CPU按占有资源排序)。
3. `printf "%x\n" 线程ID`，线程ID从10进制转16进制。
4. `jstack 进程ID | grep "刚才转的16进制"` 查看相应的代码。

## Full GC如何排查

1. 确定一定要有GC日志。java启动参数要包括

`-Xloggc:/home/work/logs/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps`

2. 查看GC日志

![image.png](/image/mynote/java_common_tools_5.png)

3. 导出Heap堆信息，分析Heap堆。

用上面的jmap命令即可。

