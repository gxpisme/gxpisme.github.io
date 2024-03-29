# 执行`echo someData > a.txt`的命令
> 业务上有个需求，需要用shell命令，在java代码中执行。 

## 第一版java代码
使用`Runtime.getRuntime().exec(cmd);`
```html
String cmd = "echo someData > a.txt";
Process process = Runtime.getRuntime().exec(cmd);
process.waitFor(); //waitFor代表一直等到cmd命令执行结束，才往下执行
```
发现不成功，也没有相关的报错日志。<br />换其他方法吧<br />​<br />
## 第二版java代码
使用ProcessBuilder来实现<br />`ProcessBuilder pb = new ProcessBuilder(cmd); `<br />`Process process = pb.start();`
```html
String cmd = "echo someData > a.txt";
ProcessBuilder pb=new ProcessBuilder(cmd);
Process process = pb.start();
process.waitFor(); //waitFor代表一直等到cmd命令执行结束，才往下执行
```
发现不成功，但是有相关的报错日志。
```html
Exception in thread "main" java.io.IOException: Cannot run program "echo someData > a.txt": error=2, No such file or directory
```
就是没有找到文件，是不是相对路径的问题，然后又弄成了绝对路径a.txt 换成 /tmp/a.txt，发现还是同样的错误。<br />
<br />然后百度、google，发现[https://blog.csdn.net/xzw_123/article/details/46983507](https://blog.csdn.net/xzw_123/article/details/46983507)文章，
> 执行诸如:ps -ef | grep -v grep 带有管道或重定向的命令就会出错。我们都知道使用以上两种方法执行命令时，如果带有参数就要把命令分割成数组或者List传入，不然会被当成一个整体执行（会出错，比如执行"ps -e"）。对于|,<,>号来说，这样做也不行。对于Linux系统，解决方法就是把整个命令都当成sh的参数传入，用sh来执行命令。

## 第三版java代码
### ProcessBuiler方式
```java
//  new ProcessBuiler().command()方式
List<String> commands = Lists.newArrayList("sh", "-c", "echo someData > a.txt");
ProcessBuilder commandAdd = new ProcessBuilder().command(commands);
Process process = commandAdd.start();
process.waitFor(); //waitFor代表一直等到cmd命令执行结束，才往下执行
```
### Runtime方式
```html
// Runtime.getRuntime().exec()方式
String[] commands = new String[3];
commands[0] = "sh";
commands[1] = "-c";
commands[2] = "echo someData > a.txt";
Runtime.getRuntime().exec(commands).waitFor();
```

<br />发现可行，也没有报错，结果也符合预期。
# sh -c
![image.png](/image/mynote/java_exec_shell_1.png)<br />从 ` -c ` 代表从字符串中读取命令<br />​

如下图：<br />![image.png](/image/mynote/java_exec_shell_2.png)
## 其实本质上要解决一个awk的问题
就是计算出在a.txt中有，在b.txt中没有的数据。<br />​

`awk 'NR==FNR {a[$0];next} {if (!($0 in a)) print $0}' a.txt b.txt > result.txt`<br />​

awk下NR FNR的详解参考 [awk NR FNR的使用](https://mp.weixin.qq.com/s/MqZf80E6my3TUhYN6YgPIg)<br />​

​

​

​<br />
