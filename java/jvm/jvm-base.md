# 常用配置JVM参考

## 参考配置

```shell
# 2C*G
java -Xbootclasspath/a:/use/local/server1/config/ \
-Dfile.encoding=UTF-8 -Duser.home=/export/Logs/ \
-Duser.timezone=GMT+08 \
-Duser.language=zh \
-Duser.country=CN \
-Xms5g -Xmx5g -Xss256k -Xmn2g \
-XX:SurvivorRatio=4 -XX:MetaspaceSize=300m -XX:MaxMetaspaceSize=300m \
-XX:+UseCompressedOops \
-XX:+TieredCompilation \
-XX:+AggressiveOpts \
-XX:+UseBiasedLocking \
-XX:+DisableExplicitGC \
-XX:+UseConcMarkSweepGC \
-XX:+UseParNewGC \
-XX:+CMSParallelRemarkEnabled \
-Xnoclassgc \
-XX:MaxTenuringThreshold=15 \
-XX:CMSInitiatingOccupancyFraction=60 \
-XX:+UseCMSInitiatingOccupancyOnly \
-XX:LargePageSizeInBytes=128m \
-XX:+UseFastAccessorMethods \
-XX:+HeapDumpOnOutOfMemoryError \
-XX:HeapDumpPath=/use/local/Logs/heapdump.hprof \
-jar /use/local/server1/my-customer-server.jar

# jdk8 and previous versions
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps
# jdk9 and later versions
-verbose:gc -Xlog:gc*=info,gc+heap=debug,gc+phases=debug:file=xxx/gc.log:t,tags
```

JVM 的启动参数, 从形式上可以简单分为：

- 以`-`开头为标准参数，所有的 JVM 都要实现这些参数，并且向后兼容。
- 以`-X`开头为非标准参数， 基本都是传给 JVM 的，默认 JVM 实现这些参数的功能，但是并不保证所有 JVM 实现都满足，且不保证向后兼容。
- 以`-XX:`开头为非稳定参数, 专门用于控制 JVM 的行为，跟具体的 JVM 实现有关，随时可能会在下个版本取消。
- `-XX:+-Flags` 形式, `+-` 是对布尔值进行开关。
- `-XX:key=value` 形式, 指定某个选项的值。



#### 4核8GB内存推荐配置chatGPT

```shell
-Xms4g -Xmx4g
-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/
-XX:+PrintGCDetails -XX:+PrintGCDateStamps
-Xloggc:/var/log/gc.log
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M
-Djava.awt.headless=true
# -XX:+UseGCLogFileRotation、-XX:NumberOfGCLogFiles 和 -XX:GCLogFileSize：启用GC日志文件的轮换，并设置轮换文件的数量和大小
# -Djava.awt.headless=true：在没有图形界面的服务器环境中运行Java程序，不需要图形硬件支持。
```



### VM 总内存=堆+栈+非堆+堆外内存

> 打印配置参数，常使用：   java -X  

```sh
#最大堆内存，默认为物理内存的1/4或者1G，最小为2M；单位与-Xms一致
-Xmx2048M, 
# #堆内存空间的初始大小 ， 默认为物理内存的1/64，最小为1M；可以指定单位，比如k、m，若不指定，则默认为字节
-Xms2048M
# 使用 G1 垃圾收集器 不应该 设置该选项，在其他的某些业务场景下可以设置。官方建议设置为 -Xmx 的 1/2 ~ 1/4。
# -Xmn, 等价于 -XX:NewSize  #新生代大小
-Xss256k,  # 每个线程栈的字节数   与-XX:ThreadStackSize=1m等价

# Meta空间
-XX:MetaspaceSize=384M,
-XX:MaxMetaspaceSize=384M, #Java8默认不限制Meta空间,一般不允许设置该选项;-XX:MaxPermSize=size,这是 JDK1.7 之前使用的
-XX:MaxTenuringThreshold=15, # 新生代存活年纪
# -XX:MaxDirectMemorySize=size，系统可以使用的最大堆外内存，这个参数跟-Dsun.nio.MaxDirectMemorySize效果相同。

-XX:+UseAdaptiveSizePolicy #设置此选项后，并行收集器会自动调整年轻代Eden区大小和Survivor区大小的比例，以达成目标系统规定的最低响应时间或者收集频率等指标。此参数建议在使用并行收集器时，一直打开。

-XX:+UseCompressedOops # 开启指针压缩
# Object o = new Object()
# 如果不开启普通对象指针压缩，-UseCompressedOops，会在内存中消耗24个字节，o 指针占8个字节，Object对象占16个字节。
# 如果开启普通对象指针压缩，+UseCompressedOops，会在内存中消耗20个字节，o指针占4个字节，Object对象占16个字节。
-XX:+TieredCompilation # DK8之后默认是开启，
# 关闭针对class的gc功能，表示不对方法区进行垃圾回收。请谨慎使用，因为其阻止内存回收，所以可能会导致OutOfMemoryError错误，慎用；
-Xnoclassgc,
-XX:+TraceClassLoading, # 显示类加载以及卸载的信息，否开启JVM的分层编译
# -XX:+UnlockExperimentalVMOptions -XX:+EnableJVMCI -XX:+UseJVMCICompiler #JDK10后，启动graal JIT 替换 C2
-XX:+AggressiveOpts # 打开预期在即将发布的版本中默认的点性能编译器优化
-XX:+UseBiasedLocking # JDK1.6后默认启用

# 配置GC
#  G1 配置 -XX:+UseG1GC -XX:MaxGCPauseMillis=200
# CMS
-XX:+UseParNewGC, #设置年轻代为并发收集。JDK5.0以上JVM会自行设置，无需再设
-XX:+UseConcMarkSweepGC,  # 即CMS收集，设置年老代为并发收集
-XX:+CMSParallelRemarkEnabled, 
 # 打开内存空间的压缩和整理，在Full GC后执行
-XX:+UseCMSCompactAtFullCollection,
# -XX:+CMSIncrementalMode：设置为增量收集模式。一般适用于单CPU情况。
#该参数启用后，参数CMSInitiatingOccupancyFraction才会生效。默认关闭
-XX:+UseCMSInitiatingOccupancyOnly
 # 该值代表老年代堆空间的使用率，默认值为98，以确保年老代有足够的空间接纳来自年轻代的对象，避免Full GC的发生
-XX:CMSInitiatingOccupancyFraction=65
#该参数启用后JVM无论什么时候调用系统GC，都执行CMS GC，而不是Full GC
#每次Full GC后立刻开始压缩和整理内存
-XX:+ExplicitGCInvokesConcurrent, 
# 设置在执行多少次Full GC后对内存空间进行压缩整理，默认值0
-XX:CMSFullGCsBeforeCompaction=0,
#相对于并行收集器，CMS收集器默认不会对永久代进行垃圾回收。如果希望对永久代进行垃圾回收，可用设置-XX:+CMSClassUnloadingEnabled。默认关闭
-XX:+CMSClassUnloadingEnabled,
# 在cms gc remark之前做一次ygc，减少gc roots扫描的对象数，从而提高remark的效率，默认关闭。
-XX:+CMSScavengeBeforeRemark

#其它垃圾回收参数
-XX:+ScavengeBeforeFullGC  # 年轻代GC优于Full GC执行
-XX:+DisableExplicitGC  # 不响应 System.gc() 代码
#-XX:+UseThreadPriorities：启用本地线程优先级API。即使 java.lang.Thread.setPriority() 生效，不启用则无效。
#-XX:SoftRefLRUPolicyMSPerMB=0：软引用对象在最后一次被访问后能存活0毫秒（JVM默认为1000毫秒）。
#-XX:TargetSurvivorRatio=90：允许90%的Survivor区被占用（JVM默认为50%）。提高对于Survivor区的使用率。


# 内容OOM打印
-XX:+HeapDumpOnOutOfMemoryError   # 错误dump oom
# -XX:HeapDumpPath=目录参数表示生成DUMP文件的路径，也可以指定文件名称，
#  例如：−XX:HeapDumpPath={目录}参数表示生成DUMP文件的路径，也可以指定文件名称，
#  例如：-XX:HeapDumpPath=目录参数表示生成DUMP文件的路径，也可以指定文件名称，
#  例如：−XX:HeapDumpPath={目录}/java_heapdump.hprof。如果不指定文件名，默认为：java__heapDump.hprof。
-XX:HeapDumpPath=/usr/local/ # oom保持快照位置
# -XX:OnError 选项, 发生致命错误时(fatal error)执行的脚本
	# 例如, 写一个脚本来记录出错时间, 执行一些命令, 或者 curl 一下某个在线报警的url. 示例用法: java -XX:OnError="gdb - %p" MyApp 可以发现有一个 %p 的格式化字符串，表示进程 PID。
# -XX:OnOutOfMemoryError
# -XX:ErrorFile=filename 选项, 致命错误的日志文件名，绝对路径或者相对路径

# 打印日志
-XX:-CITime  #打印消耗在JIT编译的时间。
-XX:+PrintGCDetails,  # 显示比-verbose:gc更多更准确的垃圾回收信息
-XX:+PrintGCDateStamps,  
-XX:+PrintGCTimeStamps, 
-XX:+PrintHeapAtGC,   # 进行GC的前后打印出堆的信息
-Xloggc:logs/JVMGC.log  # 报错GC日志

# 容器配置docker、k8s
# 在jdk8版本中加入了启动参数 UseCGroupMemoryLimitForHeap，从8u191开始引入了java10+上的UseContainerSupport选项，而且是默认启用的，不用设置。同时 UseCGroupMemoryLimitForHeap这个就弃用了，不建议继续使用，以下参数更加细腻的控制 JVM 使用的内存比率
# -XX:InitialRAMPercentage、-XX:MaxRAMPercentage、-XX:MinRAMPercentage
-XX:+UseCGroupMemoryLimitForHeap
# UseCGroupMemoryLimitForHeap 可以让JVM自动检测容器的可用内存，MaxRAMFraction为容器内存和堆内存的比例，比如容器内存为2G，MaxRAMFraction为2，则最大堆内存为2G/2=1G（设置为1的话就是CGroupMemoryLimit的全部，设置为2的话一半，3的话就是1/3以此类推），JVM就能通过检测容器的内存来自动调整堆内存大小，不用再显示设置堆内存了。
#参考配置 -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1
# docker配置堆大小
# docker run --rm -m 2GB openjdk:8u191-alpine java -XX:MaxRAMFraction=1 -XshowSettings:vm -version
# docker run --rm -m 2GB openjdk:8u191-alpine java -XX:InitialRAMPercentage=40.0 -XX:MaxRAMPercentage=90.0 -XX:MinRAMPercentage=50.0 -XshowSettings:vm -version

# 环境参数   查看默认的所有系统属性
# java -XshowSettings:properties -version
-Djava.awt.headless=true, 
-Duser.timezone=GMT+08  # 设置用户的时区为东八区
-Dfile.encoding=UTF-8   # 设置默认的文件编码为UTF-8
-Duser.language=zh
-Duser.country=CN
-Duser.home=/export/Logs/
-Djava.security.egd=file:/dev/./urandom #定随机数熵源(Entropy Source)

# 配置agent
-javaagent:dest/agent-bootstrap.jar
```

### 查看参数( java -X )

```shell
# 查看 VM 设置
java -XshowSettings:vm -version

#当前 JDK/JRE 的默认显示语言设置
java -XshowSettings:locale -version
```

#### 辅助信息参数设置

-XX:-CITime：打印消耗在JIT编译的时间。
-XX:ErrorFile=./hs_err_pid.log：保存错误日志或数据到指定文件中。
-XX:HeapDumpPath=./java_pid.hprof：指定Dump堆内存时的路径。
-XX:-HeapDumpOnOutOfMemoryError：当首次遭遇内存溢出时Dump出此时的堆内存。
-XX:OnError=";"：出现致命ERROR后运行自定义命令。
-XX:OnOutOfMemoryError=";"：当首次遭遇内存溢出时执行自定义命令。
-XX:-PrintClassHistogram：按下 Ctrl+Break 后打印堆内存中类实例的柱状信息，同JDK的 jmap -histo 命令。
-XX:-PrintConcurrentLocks：按下 Ctrl+Break 后打印线程栈中并发锁的相关信息，同JDK的 jstack -l 命令。
-XX:-PrintCompilation：当一个方法被编译时打印相关信息。
-XX:-PrintGC：每次GC时打印相关信息。
-XX:-PrintGCDetails：每次GC时打印详细信息。
-XX:-PrintGCTimeStamps：打印每次GC的时间戳。
-XX:-TraceClassLoading：跟踪类的加载信息。
-XX:-TraceClassLoadingPreorder：跟踪被引用到的所有类的加载信息。
-XX:-TraceClassResolution：跟踪常量池。

### 垃圾回收器

- `-XX:+UseG1GC`：使用 G1 垃圾回收器
- `-XX:+UseConcMarkSweepGC`：使用 CMS 垃圾回收器
- `-XX:+UseSerialGC`：使用串行垃圾回收器
- `-XX:+UseParallelGC`：使用并行垃圾回收器

### java classpath(cp)

> 注意：windows系统里的分隔符是`; ` Unix系统的分隔符是`:`

```sh
java -classpath .;fastjson.jar com.tbp.MyMain
 # java -cp .;fastjson.jar com.tbp.MyMain
 # java -cp ./lib/* com.tbp.MyMain
 # 多级目录
 # -classpath "./libs/*;./libs/"
 # java -classpath $(echo libs/*.jar | tr ' ' ';') Test
```

### java -jar (Xbootclasspath)

Java 命令行提供了如何扩展bootStrap 级别class的简单方法.

1、-Xbootclasspath: 完全取代基本核心的Java class 搜索路径.不常用,否则要重新写所有Java 核心class

2、-Xbootclasspath/a: 后缀在核心class搜索路径后面.常用

3、-Xbootclasspath/p: 前缀在核心class搜索路径前面.不常用,避免

```sh
java -Xbootclasspath/a:/usrhome/thirdlib.jar: -jar yourJarExe.jar
```

### 设置 agent 的语法

- `-agentlib:libname[=options]` 启用native方式的agent, 参考 `LD_LIBRARY_PATH` 路径。

- `-agentpath:pathname[=options]` 启用native方式的agent。

- `-javaagent:jarpath[=options]` 启用外部的agent库, 比如 `pinpoint.jar` 等等。

- `-Xnoagent` 则是禁用所有 agent

  ```shell
  # 开启 CPU 使用时间抽样分析
  JAVA_OPTS="-agentlib:hprof=cpu=samples,file=cpu.samples.log"
  # hprof 是 JDK 内置的一个性能分析器。cpu=samples 会抽样在各个方法消耗的时间占比, Java 进程退出后会将分析结果输出到文件。
  ```

### 运行模式(server、client)

有 64 位能力的 jdk 环境下将默认启用server, JDK1.7 之前在 32 位的 x86 机器上的默认值是 `-client` 选项

目前采用都是: `java -sever` 模式

#### 配置 JVM 对字节码的处理模式：

- `-Xint`：在解释模式（interpreted mode）下，-Xint 标记会强制 JVM 解释执行所有的字节码，这当然会降低运行速度，通常低 10 倍或更多。
- `-Xcomp`：-Xcomp 参数与 -Xint 正好相反，JVM 在第一次使用时会把所有的字节码编译成本地代码，从而带来最大程度的优化。
- `-Xmixed`：-Xmixed 是混合模式，将解释模式和变异模式进行混合使用，有 JVM 自己决定，这是 JVM 的默认模式，也是推荐模式。 我们使用 `java -version` 可以看到 `mixed mode` 等信息。



## Docker 配置JVM

### UseCGroupMemoryLimitForHeap

> 在jdk8版本中加入了启动参数 UseCGroupMemoryLimitForHeap，从8u191开始引入了java10+上的UseContainerSupport选项，而且是默认启用的，不用设置。同时 UseCGroupMemoryLimitForHeap这个就弃用了，不建议继续使用，以下参数更加细腻的控制 JVM 使用的内存比率 `-XX:InitialRAMPercentage、-XX:MaxRAMPercentage、-XX:MinRAMPercentage`
>
> UseCGroupMemoryLimitForHeap 可以让JVM自动检测容器的可用内存，MaxRAMFraction为容器内存和堆内存的比例，比如容器内存为2G，MaxRAMFraction为2，则最大堆内存为2G/2=1G（设置为1的话就是CGroupMemoryLimit的全部，设置为2的话一半，3的话就是1/3以此类推），JVM就能通过检测容器的内存来自动调整堆内存大小，不用再显示设置堆内存了。
>
> `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1`

### InitialRAMPercentage、MaxRAMPercentage、MinRAMPercentage

演示

> `docker run --rm -m 2GB openjdk:8u191-alpine java -XX:MaxRAMFraction=1 -XshowSettings:vm -version`

```
VM settings:
    Max. Heap Size (Estimated): 6.81G
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM
```

> `docker run --rm -m 2GB openjdk:8u191-alpine java -XX:InitialRAMPercentage=40.0 -XX:MaxRAMPercentage=90.0 -XX:MinRAMPercentage=50.0 -XshowSettings:vm -version`

```
VM settings:
    Max. Heap Size (Estimated): 6.13G
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM
```



## mat 内存分析工具

>  MemoryAnalyzer -data ./workspace

