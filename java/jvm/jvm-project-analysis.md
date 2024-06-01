

## jstat

```
jstat -gcutil 49030 1000 20
jstat -gcutil pid 时间秒 次数
```

## jstack

```
jstack [-option] <pid> // 打印某个进程的堆栈信息

-F： 当正常输出的请求不被响应时，强制输出线程堆栈
-m： 如果调用到本地方法的话，可以显示C/C++的堆栈
-l： 除堆栈外，显示关于锁的附加信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-h： 打印帮助信息
```

### 排查方法

```
查看CUP靠前：top -Hp pid
	转化CPU高的：printf "%x\n" 3440  
工作台查看：jstack pid 
	jstack pid |grep "print的16进制的值"
输出到文件： jstack <pid> > <pid>_core.dum
```

## jmap

```
生成内存快照
jmap -dump:format=b,file=heapdump.bin <pid>

查看对象统计信息
jmap -histo <pid>

查看 ClassLoader 统计信息
jmap -clstats <pid>
```

## jcmd

```
列出当前所有可用的 Java 进程
jcmd 

输出 Java 进程的 GC 信息
jcmd <pid> GC.info

输出 Java 进程的线程堆栈信息
jcmd <pid> Thread.print

执行 Java 进程的垃圾回收操作
jcmd <pid> GC.run
```

