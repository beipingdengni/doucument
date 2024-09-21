

## 查看进程占用的线程梳理

```shell
ps -eLf | awk '{print $2}' | sort | uniq -c | sort -nr|head -n10
#ps -eLf：显示所有线程的详细列表。-e选项表示显示所有进程，-L选项表示显示线程，-f选项表示全格式显示。
#awk '{print $2}'：从ps的输出中提取第二列（PID），由于ps -eLf也包含了TID，这个命令会输出所有的PID和TID，但我们主要是用来统计的。
#sort：将输出排序，方便后续处理。
#uniq -c：对排序后的输出进行去重并计数，显示每个唯一项的出现次数，这里表示每个PID（包括其线程）的数量。
#sort -nr：最后按数字排序，-n表示按数值排序，-r表示逆序排列，使得线程数最多的进程排在最前面。
#head -n10 获取排名的前10个
```



## jstat

```shell
jstat -gcutil 49030 1000 20
jstat -gcutil pid 时间秒 次数
```

## jstack

```shell
jstack [-option] <pid> // 打印某个进程的堆栈信息

-F： 当正常输出的请求不被响应时，强制输出线程堆栈
-m： 如果调用到本地方法的话，可以显示C/C++的堆栈
-l： 除堆栈外，显示关于锁的附加信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-h： 打印帮助信息
```

### 排查方法

```shell
查看CUP靠前：top -Hp pid
	转化CPU高的：printf "%x\n" 3440  
工作台查看：jstack pid 
	jstack pid |grep "print的16进制的值"
输出到文件： jstack <pid> > <pid>_core.dum
```

## jmap

```shell
#生成内存快照
jmap -dump:format=b,file=heapdump.bin <pid>

#查看对象统计信息
jmap -histo <pid>

#查看 ClassLoader 统计信息
jmap -clstats <pid>
```

## jcmd(执行jvm命令，查看相关信息)

```shell
#列出当前所有可用的 Java 进程
jcmd 

#输出 Java 进程的 GC 信息
jcmd <pid> GC.info

#输出 Java 进程的线程堆栈信息
jcmd <pid> Thread.print

#执行 Java 进程的垃圾回收操作
jcmd <pid> GC.run
jcmd 1 VM.native_memory #添加JVM参数-XX:NativeMemoryTracking=detail开启，使用jcmd查看，如下：
```

## 排查案例

一次Java内存占用高的排查案例，解释了我对内存问题的所有疑问：https://zhuanlan.zhihu.com/p/662499118



### pmap

> pmap liunx 查看内存工具 `pmap [options] pid [...]`
>
> 循环显示进程 3066 的设备格式的最后 1 行，间隔 3 秒：`while true; do pmap -d 3066 | tail -1; sleep 2; done`
>
> ```shell
> -x, --extended
> 	显示扩展格式。
> -d, --device
> 	显示设备格式。
> -q, --quiet
> 	不显示某些页眉或页脚行。
> -A, --range <low>,<high>
> 	将结果限制在给定范围内的低地址和高地址。
> -X
> 	显示比 -x 选项更多的细节。 警告：根据 /proc/PID/smaps 改变格式。
> -XX
> 	显示内核提供的所有内容。
> -p, --show-path
> 	在映射列中显示文件的完整路径。
> -c, --read-rc
> 	读取默认配置。
> -C, --read-rc-from <file>
> 	从指定文件读取配置。
> -n, --create-rc
> 	创建新的默认配置。
> -N, --create-rc-to file
> 	创建新配置到指定文件。
> -h, --help
> 	现实帮助信息并退出。
> -V, --version
> 	显示版本信息并退出。
> 	
> 查询显示扩展格式：pmap -x 1
> 检查那些占用内存较大的内存段: pmap -x 1 | sort -nrk3 | less 
> 查看指定位置内存信息：tail -c +$((0x00007f8845000000+1)) /proc/1/mem|head -c $((11616*1024))|strings|less -S
> 857705
> tail -c +$((0x00007f8845000000+1)) /proc/857705/mem|head -c $((11616*1024))|strings|less -S
> 
> Address：表示此内存段的起始地址
> Kbytes：表示此内存段的大小(ps：这是虚拟内存)
> RSS：表示此内存段实际分配的物理内存，这是由于Linux是延迟分配内存的，进程调用malloc时Linux只是分配了一段虚拟内存块，直到进程实际读写此内存块中部分时，Linux会通过缺页中断真正分配物理内存。
> Dirty：此内存段中被修改过的内存大小，使用mmap系统调用申请虚拟内存时，可以关联到某个文件，也可不关联，当关联了文件的内存段被访问时，会自动读取此文件的数据到内存中，若此段某一页内存数据后被更改，即为Dirty，而对于非文件映射的匿名内存段(anon)，此列与RSS相等。
> Mode：内存段是否可读(r)可写(w)可执行(x)
> Mapping：内存段映射的文件，匿名内存段显示为anon，非匿名内存段显示文件名(加-p可显示全路径)。
> ```

