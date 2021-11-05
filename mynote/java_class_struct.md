# java Class类文件 分析

# java虚拟机 和 java语言
java虚拟机可以支持其他语言运行在java虚拟机之上，java虚拟机上执行的是字节码。
java语言翻译成字节码才能在java虚拟机上执行，因此只有任何语言能够翻译成字节码，同样可以运行在java虚拟机上。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1609920832931-bb967c9b-caec-4c3c-afe0-b2d3c5e3495d.png#height=233&id=hzOO0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=930&originWidth=1510&originalType=binary&ratio=1&size=384509&status=done&style=none&width=378)

Java技术能够一直保持着非常良好的向后兼容性，Class文件结构的稳定功不可没，

关于Class文件结构的内容，绝大部分都是在第一版的《Java虚拟机规范》中就已经定义好的，内容虽然古老，但时至今日，Java发展 经历了十余个大版本、无数小更新，那时定义的Class文件格式的各项细节几乎没有出现任何改变。

# class文件组成
Class文件是一组以8个字节为基础单位的二进制流。

> Class文件格式采用一种类似于C语言结构体的伪结构来存储数 据，这种伪结构中只有两种数据类型:“无符号数”和“表”。

## 无符号数
无符号数属于基本的数据类型，以**u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节**的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串 值。

## 表
表是由多个无符号数或者其他表作为数据项构成的复合数据类型，为了便于区分，所有表的命名 都习惯性地以"_info"结尾。


# 结构

| 类型 | 名称 | 备注 | 数量 |
| --- | --- | --- | --- |
| u4 | magic | 魔数 | 1 |
| u2 | minor_vesion | 次版本号 | 1 |
| u2 | major_version | 主版本号 | 1 |
| u2 | constant_pool_count | 常量池数量 | 1 |
| cp_info | constant_pool | 常量池 | constant_pool_count - 1 |
| u2 | access_flags | 访问标识 | 1 |
| u2 | this_class | 类索引 | 1 |
| u2 | super_class | 父类索引 | 1 |
| u2 | interfaces_count | 接口个数 | 1 |
| u2 | interfaces | 接口 | 1 |
| u2 | fields_count | 字段数 | 1 |
| field_info | fields | 字段表 | fields_count |
| u2 | methods_count | 方法数 | 1 |
| method_info | methods | 方法 | methods_count |
| u2 | attributes_count | 属性数 | 1 |
| attribute_info | attributes | 属性 | attributes_count |

## 魔数与class文件的版本

每个Class文件的**头4个字节**被称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。

Class文件的魔数值为`0xCAFEBABE`。

紧接着魔数的**4个字节**存储的是Class文件的版本号：第5和第6字节是**次版本号**（Minor Version），第7和第8字节是**主版本号**（Major Version）。


```
JDK 1.1能支持版本号为45.0~45.65535的Class文件，无法执行版本号为46.0以上的Class文件，
JDK 1.2则能支持45.0~46.65535的Class文件。
JDK 1.3则能支持45.0~47.65535的Class文件。
JDK 1.4则能支持45.0~48.65535的Class文件。
JDK 1.5则能支持45.0~49.65535的Class文件。
JDK 1.6则能支持45.0~50.65535的Class文件。
JDK 1.7则能支持45.0~51.65535的Class文件。
JDK 1.8则能支持45.0~52.65535的Class文件。
JDK 1.9则能支持45.0~53.65535的Class文件。
......
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1609925022094-c6164cb2-1b22-480c-8d55-1d971f18d22e.png#height=78&id=LHyTM&margin=%5Bobject%20Object%5D&name=image.png&originHeight=156&originWidth=1368&originalType=binary&ratio=1&size=131325&status=done&style=none&width=684)
0xcafebabe 就是 魔数
0x0034 就是 3*16 + 4 = 52，也就是JDK1.8。
关于次版本号，曾经在现代Java(即Java 2)出现前被短暂使用过，JDK 1.0.2支持的版本45.0~45.3(包括45.0~45.3)。JDK 1.1支持版本45.0~45.65535，从JDK 1.2以后，直到JDK 12之前次版本 号均未使用，全部固定为零。


## 常量池
### 常量池数量
> 常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值（constant_pool_count）

constant_pool_count 常量池数量是从1开始的，不是从0开始的。将第0项常量空出来是有特殊考虑的，这样做的目的在于：如果**后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”**的含义。可以把索引值设置为0来表示。

- [ ] 这里还是没能理解0究竟作何用处。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1609987757150-8aa49243-c63c-4825-a23b-c4adfc831cde.png#height=118&id=KN6dE&margin=%5Bobject%20Object%5D&name=image.png&originHeight=236&originWidth=1280&originalType=binary&ratio=1&size=247429&status=done&style=none&width=640)
constant_pool_count 是 用u2来表示的，也就是两个字节。这里0x001d转为10进制就是29，也就是有29-1=28个常量。
### 常量池（字面量和符号引用）
常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。
字面量：接近于java语言的常量概念，例如：文本字符串、被声明为final的常量值。
符号引用：编译原理方面的概念（被模块导出或者开放的包、类和接口的全限定名、字段的名称和描述符。。。。）


常量池中每一项常量都是一个表，截止JDK13，常量表中有17种不同类型的常量。这17种表都有一个共同的特点，表结构起始的第一位是个u1类型（1个字节）的标志位，代表着当前常量属于哪种常量类型。


**常量池的项目类型**

| 类型 | 标志 | 描述 | 项目 | 类型 | 详细描述 |
| --- | --- | --- | --- | --- | --- |
| CONSTANT_Utf8_info | 1 | UTF-8编码的字符串 | tag | u1 | 值为1 |
|  |  |  | length | u2 | UTF-8 编码的字符串占的字节数 |
|  |  |  | bytes | u1 | 长度为length的UTF-8编码的字符串 |
| CONSTANT_Integer_info | 3 | 整型字面量 | tag | u1 | 值为3 |
|  |  |  | bytes | u4 | 按照高位在前存储的int值 |
| CONSTANT_Float_info | 4 | 浮点型字面量 | tag | u1 | 值为4 |
|  |  |  | bytes | u4 | 按照高位在前存储的float值 |
| CONSTANT_Long_info | 5 | 长整型字面量 | tag | u1 | 值为5 |
|  |  |  | bytes | u8 | 按照高位在前存储的long值 |
| CONSTANT_Double_info | 6 | 双精度浮点型字面量 | tag | u1 | 值为6 |
|  |  |  | bytes | u8 | 按照高位在前存储的double值 |
| CONSTANT_Class_info | 7 | 类或接口的符号引用 | tag | u1 | 值为7 |
|  |  |  | index | u2 | 指向全限定名常量项的索引 |
| CONSTANT_String_info | 8 | 字符串类型字面量 | tag | u1 | 值为8 |
|  |  |  | index | u2 | 指向字段串字面量的索引 |
| CONSTANT_Fieldref_info | 9 | 字段的符号引用 | tag | u1 | 值为9 |
|  |  |  | index | u2 | 指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项 |
|  |  |  | index | u2 | 指向字段描述符CONSTANT_NameAndType的索引项 |
| CONSTANT_Methodref_info | 10 | 类中方法的符号引用 | tag | u1 | 值为10 |
|  |  |  | index | u2 | 指向声明方法的类描述符CONSTANT_Class_info的索引项 |
|  |  |  | index | u2 | 指向名称及类型描述符CONSTANT_NameAndType的索引项 |
| CONSTANT_InterfaceMethodref_info | 11 | 接口中方法的符号引用 |  |  |  |
| CONSTANT_NameAndType_info | 12 | 字段或方法的部分符号引用 | tag | u1 | 值为12 |
|  |  |  | index | u2 | 指向该字段或方法名称常量项的索引 |
|  |  |  | index | u2 | 指向该字段或方法描述符常量项的索引 |
| CONSTANT_MethodHandle_info | 15 | 表示方法句柄 |  |  |  |
| CONSTANT_MethodType_info | 16 | 表示方法类型 |  |  |  |
| CONSTANT_Dynamic_info | 17 | 表示一个动态计算常量 |  |  |  |
| CONSTANT_InvokeDynamic_info | 18 | 表示一个动态方法调用点 |  |  |  |
| CONSTANT_Module_info | 19 | 表示一个模块 |  |  |  |
| CONSTANT_Package_info | 20 | 表示一个模块中开发或导出的包 |  |  |  |



上面说了表结构起始的第一位是个u1类型就是0x0a=10，对应到上面的那就是CONSTANT_Methodref_info 类中方法的符号引用。
​

```java
mac@macbook-pro ~/xpstudy $ cat Test.java
public class Test {
    public static void main(String [] args) {
        System.out.println("hello Test");
    }
}

mac@macbook-pro ~/xpstudy $ hexdump -C Test.class
00000000  ca fe    ba be    00 00    00 34     00 1d    0a 00    06 00    0f 09  |.......4........|
         {    魔 数     }  {次版本号} {主版本号} {常量池数}{标志十}{指向..}{指向...} {标志九}
00000010  00 10    00 11    08 00    12 0a     00 13    00 14    07 00    15 07  |................|
          {指向...}{指向.} {标志8}{指向..}{标志十} {指向..} {指向..}{标志七}{指向..} {标志七}
00000020  00 16    01 00    06 3c    69 6e     69 74    3e 01    00 03    28 29  |.....<init>...()|
         {指向..}{标志一}{utf8长}{长度为6的UTF-8编码的字符串}   {标志一}{utf8长}{长度为3的UTF-8编码的...
00000030  56 01    00 04    43 6f    64 65     01 00    0f 4c    69 6e    65 4e  |V...Code...LineN|
  ...字符串}{标志一}{utf8长}{长度为4的UTF-8字符串}{标志一}{utf8长}{长   度   为15   的  UTF-8
00000040  75 6d    62 65    72 54    61 62     6c 65    01 00    04 6d    61 69  |umberTable...mai|
          的  字   符    串                          }{标志一}{utf8长}{长度为4的UTF-8的
00000050  6e 01    00 16    28 5b    4c 6a     61 76    61 2f    6c 61    6e 67  |n...([Ljava/lang|
	  字符串}{标志一}{utf8长} {         长         度       为       22    的   UTF-8 的
00000060  2f 53    74 72    69 6e    67 3b     29 56    01 00    0a 53    6f 75  |/String;)V...Sou|
          字         符            串                }{标志一}{utf8长}{ 长   度  为
00000070  72 63    65 46    69 6c    65 01     00 09    54 65    73 74    2e 6a  |rceFile...Test.j|
	        为10 的  UTF-8 字  符    串 }{标志一}{utf8长}{ 长 度 为 9 的 UTF-8 编 码
00000080  61 76    61 0c    00 07    00 08     07 00    17 0c    00 18    00 19  |ava.............|
		的 字 符 串   }{标志12}{指向} {指向...} {标志7}{指向全..}{标志12}{指向} {指向...}
00000090  01 00    0a 68    65 6c    6c 6f     20 54    65 73    74 07    00 1a  |...hello Test...|
       {标志一}{utf8长}{  长   度   为  10   的     字       符     串} {标志7}{指向全限定名常量项的索引}
000000a0  0c 00    1b 00    1c 01    00 04     54 65    73 74    01 00    10 6a  |........Test...j|
      {标志12}{指向..} {指向...}{标志1}{utf8长}{ 长 度 为 4的字符串} {标志1}{utf8长}{
000000b0  61 76    61 2f    6c 61    6e 67     2f 4f    62 6a    65 63    74 01  |ava/lang/Object.|
          长      度          为          16       的       字     符     串  }{标志1}
000000c0  00 10    6a 61    76 61    2f 6c     61 6e    67 2f    53 79    73 74  |..java/lang/Syst|
		{utf8长}{          长      度          为          16       的       字
000000d0  65 6d    01 00    03 6f    75 74     01 00    15 4c    6a 61    76 61  |em...out...Ljava|
        符     串}{标志一}{utf8长}{长度为3的字符串}{标志一}{utf8长}{           长
000000e0  2f 69    6f 2f    50 72    69 6e     74 53    74 72    65 61    6d 3b  |/io/PrintStream;|
           度   为    21    的      字       符          串                      }
000000f0  01 00    13 6a    61 76    61 2f     69 6f    2f 50    72 69    6e 74  |...java/io/Print|
	{标志一}{utf8长}{          长      度          为          19       的       字
00000100  53 74    72 65    61 6d    01 00     07 70    72 69    6e 74    6c 6e  |Stream...println|
                 符      串      }  {标志一}{utf8长}{  长   度   为  7的  字  符 串}
00000110  01 00    15 28    4c 6a    61 76     61 2f    6c 61    6e 67    2f 53  |...(Ljava/lang/S|
      {标志一}{utf8长}{       长          度           为           21
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         }
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c





mac@bogon $ ~/xpstudy $ javap -v Test
Classfile /Users/mac/xpstudy/Test.class
  Last modified Jan 7, 2021; size 412 bytes
  MD5 checksum 3b29ea81bf55ce01d8fce32dfa0a7868
  Compiled from "Test.java"
public class Test
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // hello Test
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // Test
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Test.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Class              #23            // java/lang/System
  #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #18 = Utf8               hello Test
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               Test
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
{
  public Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String hello Test
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 3: 0
        line 4: 8
}
```


## 访问标志 u2
`**u2**`
在常量池结束之后，紧接着的2个字节代表访问标志access_flags.

| 标志名称 | 标志值 | 含义 |
| --- | --- | --- |
| ACC_PUBLIC | 0x0001 | 是否为public类型 |
| ACC_FINAL | 0x0010 | 是否设置为final，只有类可设置 |
| ACC_SUPER | 0x0020 | JDK1.0.2之后编译的都是为真 |
| ACC_INTERFACE | 0x0200 | 标识这是一个接口 |
| ACC_ABSTART | 0x0400 | 是否为abstract类型 |
| ACC_SYNTHETIC | 0x1000 | 标识这个类，并非用户代码产生 |
| ACC_ANNOTATION | 0x2000 | 标识这是一个注解 |
| ACC_ENUM | 0x4000 | 标识这是一个注解 |
| ACC_MOUDLE | 0x8000 | 标识这是一个模块 |

```java
mac@macbook-pro ~/xpstudy $ cat Test.java
public class Test {
    public static void main(String [] args) {
        System.out.println("hello Test");
    }
}
// ACC_PUBLIC

mac@bogon $ ~/xpstudy $ javap -v Test
Classfile /Users/mac/xpstudy/Test.class
......
  public Test();
    descriptor: ()V
    flags: ACC_PUBLIC
......
```
ACC_PUBLIC、ACC_SUPER标志应当为真，而ACC_FINAL、ACC_INTERFACE、ACC_ABSTRACT、ACC_SYNTHETIC、ACC_ANNOTATION、ACC_ENUM、ACC_MODULE这七 个标志应当为假，因此它的access_flags的值应为:0x0001|0x0020=0x0021。


```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c

```
## 类索引、父类索引
在访问标识结束之后，紧接着的2个字节代表类索引this_class，在类索引后是父类索引super_class，同样只占两个字节。
​

Java语言不允许多重继承，因此父类索引只有一个，除了java.lang.Object之外，所有的java类都有父类。
​

```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志} {类索引} {父类索引}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c
```


类索引 0x0005 = 5  是 Test，看16进制也是，第5个常量是代表着CONSTANT_class_info，这个CONSTANT_class_info指向了第21个常量CONSTANT_utf8_info，也就是Test。
​

父类索引 0x0006 = 6 是 Java/lang/Object
```java
mac@bogon $ ~/xpstudy $ javap -v Test
......
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // hello Test
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // Test
   #6 = Class              #22            // java/lang/Object
......
```


## 接口索引数量和接口索引
用来描述这个类实现了哪些接口，这些被实现的接口将按implements关键字后的接口顺序从左到右排列在接口索引集合中。
```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志} {类索引} {父类索引} {接口索引数}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c
```


## 字段表
字段表(field_info)用于描述接口或者类中声明的变量。
Java语言中的字段“field”包括类级别变量和实例级别变量。但不包括包括方法内部的局部变量。



| 类型 | 名称 | 数量 |  |  |  |
| --- | --- | --- | --- | --- | --- |
| u2 | access_flags | 1 | 标志名称 | 标志值 | 含义 |
|  |  |  | ACC_PUBLIC | 0x0001 | 字段是否是public |
|  |  |  | ACC_PRIVATE | 0x0002 | 字段是否是private |
|  |  |  | ACC_PROTECTED | 0x0004 | 字段是否是protected |
|  |  |  | ACC_STATIC | 0x0008 | 字段是否是static |
|  |  |  | ACC_FINAL | 0x0010 | 字段是否是final |
|  |  |  | ACC_VOLATILE | 0x0040 | 字段是否是volatile |
|  |  |  | ACC_TRANSIENT | 0x0080 | 字段是否是transient |
|  |  |  | ACC_SYNTHETIC | 0x0100 | 字段是否有编译器自动生成 |
|  |  |  | ACC_ENUM | 0x0400 | 字段是否为enum |
| u2 | name_index | 1 |  |  |  |
| u2 | descriptor_index | 1 |  |  |  |
| u2 | attributes_count | 1 |  |  |  |
| attribute_info | attributes | attributes_count |  |  |  |



```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志} {类索引} {父类索引} {接口索引数}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
        {字段数}
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c
```


## 方法表
Class文件存储格式中对方法的描述与对字段的描述采用了几乎完全一致的方式。
仅在访问标志和属性表集合的可选项中有所差别。
>

> 因为volatile关键字和transient 关键字不能修饰方法，所以方法表的访问标志中没有了ACC_VOLATILE标志和ACC_TRANSIENT标志。与之相对，synchronized、native、strictfp和abstract关键字可以修饰方法，方法表的访问标志中也相应地增加了ACC_SYNCHRONIZED、ACC_NATIVE、ACC_STRICTFP和ACC_ABSTRACT标志。



| 类型 | 名称 | 数量 |  |  |  |
| --- | --- | --- | --- | --- | --- |
| u2 | access_flags | 1 | 标志名称 | 标志值 | 含义 |
|  |  |  | ACC_PUBLIC | 0x0001 | 方法是否是public |
|  |  |  | ACC_PRIVATE | 0x0002 | 方法是否是private |
|  |  |  | ACC_PROTECTED | 0x0004 | 方法是否是protected |
|  |  |  | ACC_STATIC | 0x0008 | 方法是否是static |
|  |  |  | ACC_FINAL | 0x0010 | 方法是否是final |
|  |  |  | **ACC_SYNCHRONIZED** | **0x0020** | 方法是否为synchronized |
|  |  |  | ACC_BRIDGE | 0x0040 | 方法是否由编译器生成的桥接方法 |
|  |  |  | ACC_VARARGS | 0x0080 | 方法是否接受不定参数 |
|  |  |  | ACC_NATIVE | 0x0100 | 方法是否是native |
|  |  |  | ACC_ABSTRACT | 0x0400 | 方法是否是abstract |
|  |  |  | ACC_STRICT | 0x0800 | 方法是否是strict |
|  |  |  | ACC_SYNTHETIC | 0x1000 | 方法是否有编译器自动生成 |
| u2 | name_index | 1 |  |  |  |
| u2 | descriptor_index | 1 |  |  |  |
| u2 | attributes_count | 1 |  |  |  |
| attribute_info | attributes | attributes_count |  |  |  |

```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志} {类索引} {父类索引} {接口索引数}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
        {字段数}   {方法数} {访问标志} {名称索引} {描述符索引} {属性数}  {属性}
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/546024/1610728084682-abf9d1d4-b522-40f3-8067-0727c89a3333.png#height=173&id=bZ92D&margin=%5Bobject%20Object%5D&name=image.png&originHeight=346&originWidth=1468&originalType=binary&ratio=1&size=440458&status=done&style=none&width=734)


## 属性
对于每一个属性，它的**名称**都要从常量池中引用一个CONSTANT_Utf8_info类型的常量来表示，而属性值的定义则是完全自定义的。只需要通过一个u4的长度属性去说明属性值所占用的位数即可。

| 类型 | 名称 | 数量 | 说明 |
| --- | --- | --- | --- |
| u2 | attribute_name_index | 1 | 属性的**名称**，引用常量池 |
| u4 | attribute_length | 1 | 属性长度 |
| u1 | info | attribute_length | 自定义的属性值 |



0x0001 代表有1个属性
0x0009 代表9 是属性的索引，指向的CONSTANT_Utf8_info常量，`#9 = Utf8    Code`
0x0000001d  代表`29` 就是属性长度。
```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志} {类索引} {父类索引} {接口索引数}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
        {字段数}   {方法数} {访问标志} {名称索引} {描述符索引} {属性数} {属性索引}  { 属 性
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
           长 度}
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
0000019c
```


### 属性种类很多
| 属性名称 | 使用位置 | 含义 |
| --- | --- | --- |
| Code | 方法表 | Java代码编译成的字节码指令 |
| ConstantValue | 字段表 | 由final关键字定义的常量值 |
| Deprecated | 类、方法表、字段表 | 被声明为Deprecated的方法和字段 |
| Exceptions | 方法表 | 方法抛出的异常列表 |
| EnclosingMethod | 类文件 | 仅当一个类为局部类或者匿名类时才能拥有这个属性，
这个属性用于标志这个类所在的外围方法。 |
| InnerClasses | 类文件 | 内部类列表 |
| LineNumberTable | Code属性 | Java源码的行号与字节码指令的对应关系 |
| LocalVariableTable | Code属性 | 方法的局部变量描述 |
| StackMapTable | Code属性 | 供新的类型检查验证器检查和处理目标方法的局部变量和操作数栈所需要的类型是否匹配 |
| ....... |  |  |



#### Code属性
> JAVA 程序方法体里面的代码经过JAVA编译器处理之后，最终变为字节码指令存储在Code属性内。
> JAVA 程序方法体里面的代码经过JAVA编译器处理之后，最终变为字节码指令存储在Code属性内。
> JAVA 程序方法体里面的代码经过JAVA编译器处理之后，最终变为字节码指令存储在Code属性内。

Code 属性表结构

| 类型 | 名称 | 数量 | 说明 |
| --- | --- | --- | --- |
| u2 | attribute_name_index | 1 | 属性的**名称**，引用常量池 |
| u4 | attribute_length | 1 | 属性长度 |
| u2 | max_stack | 1 | 操作数栈深度的最大值，再方法操作的任意时刻，操作数栈都不会超过这个深度。 |
| u2 | max_locals | 1 | 代表了局部变量表所需的存储空间。 |
| u4 | code_length | 1 | 用于存储Java源程序编译后生成的字节码指令。
code_length 字节码长度，code是用于存储字节码指令的一系列字节流。 |
| u1 | code | code_length |  |
| u2 | exception_table_length | 1 |
异常表 |
| exception_info | exception_table | exception_table_length |  |
| u2 | attributes_count | 1 |
属性表 |
| attribute_info | attributes | attributes_count |  |



```java
...............
00000120  74 72    69 6e    67 3b    29 56     00 21    00 05    00 06    00 00  |tring;)V.!......|
          的      字       符     串         } {访问标志} {类索引} {父类索引} {接口索引数}
00000130  00 00    00 02    00 01    00 07     00 08    00 01    00 09    00 00  |................|
        {字段数}   {方法数} {访问标志} {名称索引} {描述符索引} {属性数} {属性索引}  { 属 性
00000140  00 1d    00 01    00 01    00 00     00 05    2a b7    00 01    b1 00  |..........*.....|
           长 度}{max_stack}{max_l...}{code_length }    {         code      } {exception_table_
00000150  00 00    01 00    0a 00    00 00     06 00    01 00    00 00    01 00  |................|
     length} {属性数}  {属性索引}{  属  性  长  度  }{ ----------------------- }{访问
00000160  09 00    0b 00    0c 00    01 00     09 00    00 00    25 00    02 00  |............%...|
        标志}{名称索引}{描述符索引}{属性数} {属性索引 } { 属  性  长   度 }{..stack} {max
00000170  01 00    00 00    09 b2    00 02     12 03    b6 00    04 b1    00 00  |................|
    _locals} {  code_length  } {             code                    } {exception_table_length}
00000180  00 01    00 0a    00 00    00 0a     00 02    00 00    00 03    00 08  |................|
          {属性数} {属性索引} { 属 性 长 度  }     {-------------------------------
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
          -------}
0000019c
```


### 最后一个属性 SourceFile
| 类型 | 名称 | 数量 |
| --- | --- | --- |
| u2 | attribute_name_index | 1 |
| u4 | attribute_length | 1 |
| u2 | sourecefile_index | 1 |

```java
00000190  00 04    00 01    00 0d    00 00     00 02    00 0e                    |............|
          -------}{  name_index  }  {attribute_length} {sourcefile_index}
0000019c
```


# 终于分析完了！好酸爽
# 终于分析完了！好酸爽
# 终于分析完了！好酸爽

