## JVM 回顾

参考url：https://www.cnblogs.com/yjmyzz/p/jvm-memory-structure-and-gc.html

#### 一次 JVM FullGC 的排查过程及解决方案【SoftReference】

> 说明博客  https://blog.csdn.net/mifffy_java/article/details/91491377

```
虚拟机在抛出 OutOfMemoryError 之前会保证所有的软引用对象已被清除。此外，没有任何约束保证软引用将在某个特定的时间点被清除，或者确定一组不同的软引用对象被清除的顺序。不过，虚拟机的具体实现会倾向于不清除最近创建或最近使用过的软引用。

简单总结如下：
1. 在你看到 OutOfMemoryError 前，Java 虚拟机一定会回收 SoftReference 对象；
2. Java 不保证 SoftReference 对象何时被清除，相关的机制是 JVM 实现相关的；

实际上也就是可能在OutOfMemoryError 前，JVM未必会清理SoftReference 。软引用对象占用内存都可能进入到 Old-Gen。这样你的进程会频繁触发 Full GC，但即使这样，JVM 也不一定会清理 SoftReference 占用的内存。这就出现了websocket服务出现的长期fullGc，慢响应但系统不挂。

发生 GC 的时候，是否清理 SoftReference 取决于两个因素：
1. 引用的时间戳；
2. 有多少可用内存。
 
计算公式非常简单，首先定义：
free_heap    - 堆里的空闲内存数量，单位是 MB
interval       - 上一次 GC 时间与与引用记录的时间戳之间的时间间隔
ms_per_mb - 是一个毫秒数常量，表示每 MB 空闲堆中保留的 SoftReference 数量。
 
判定公式是：
interval <= free_heap * ms_per_mb“
 
其中 ms_per_mb 是一个可以设置的 JVM 参数：-XX:SoftRefLRUPolicyMSPerMB，结合公式很容易看明白，这个参数决定 FullGC 保留的 SoftReference 数量，参数值越大，GC 后保留的软引用对象就越多。
 
有些博客在推荐 JVM 参数时，建议 -XX:SoftRefLRUPolicyMSPerMB 配置成 0 ，这样可以避免在 GC 后保留 SoftReference。是否这样就可以完全避免软引用回收的问题？

配置
-XX:SoftRefLRUPolicyMSPerMB=0
理论上这样配置，会使得数据在GC时瞬间收回，我们通过以下代码实测发现在YGC的情况下，对象不会回收，直接FGC才会进行回收。如果不设置JVM参数，则FGC仍然可能不会回收。
```

## CMS（Concurrent Mark-Sweep）

CMS（Concurrent Mark-Sweep）垃圾收集器是Java虚拟机（JVM）中的一种以获取最短回收停顿时间为目标的收集器。CMS适用于注重服务的响应速度，希望系统停顿时间最短的应用场景。但请注意，从Java 9开始，CMS垃圾收集器已经被标记为废弃，并在后续版本中被移除。

在配置CMS垃圾收集器时，以下是一些重要的JVM参数：

1. `-XX:+UseConcMarkSweepGC`:
   - 启用CMS垃圾收集器。

2. `-XX:+UseCMSInitiatingOccupancyOnly`:
   - 只在到达阈值时才触发CMS回收，而不是基于运行时的自适应策略。

3. `-XX:CMSInitiatingOccupancyFraction=<value>`:
   - 设置堆使用率的阈值，超过这个阈值时，将触发CMS回收。
   - `<value>`是一个介于0到100之间的整数，表示百分比。

4. `-XX:+CMSScavengeBeforeRemark`:
   - 在CMS的remark阶段前进行一次young generation的垃圾收集，减少remark的时间并提高效率。

5. `-XX:+UseCMSCompactAtFullCollection`:
   - 在CMS收集器进行完整垃圾收集后，进行一次内存压缩。

6. `-XX:ParallelCMSThreads=<value>`:
   - 设置并行执行CMS阶段的线程数。
   - `<value>`应该根据你的CPU核心数进行设置。

7. `-XX:+UseParNewGC`:
   - 使用ParNew（并行）垃圾收集器处理新生代，这通常与CMS一起使用。

8. `-XX:SurvivorRatio=<value>`:
   - 设置新生代中Eden区与Survivor区的比例。
   - `<value>`是一个整数，表示Eden区与一个Survivor区的大小比例。（默认值：8）

9. `-XX:NewRatio=<value>`:
   - 设置年轻代（包括Eden和Survivor区）与老年代的比例。
   - `<value>`是一个整数，表示老年代与年轻代的大小比例。（默认值：2）

10. `-XX:MaxTenuringThreshold=<value>`:
    - 设置对象在Survivor区中复制的次数，超过这个次数后，对象会被晋升到老年代。

11. `-XX:+PrintGCDetails` 和 `-XX:+PrintGCDateStamps`:
    - 输出详细的GC日志信息以及GC发生的时间戳。

举例，启用CMS收集器的JVM参数配置可能如下：

```shell
java -XX:+UseConcMarkSweepGC -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=75 -XX:+CMSScavengeBeforeRemark -XX:+UseCMSCompactAtFullCollection -XX:ParallelCMSThreads=4 -XX:+UseParNewGC -XX:SurvivorRatio=8 -XX:NewRatio=2 -XX:MaxTenuringThreshold=15 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -jar your-application.jar


-Xms4g -Xmx4g -Xmn2g
-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m
-XX:+UseConcMarkSweepGC
-XX:+UseCMSCompactAtFullCollection
-XX:+CMSParallelRemarkEnabled
-XX:+DisableExplicitGC
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-XX:+PrintHeapAtGC
-Xloggc:/dev/shm/mq_gc_%p.log
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/dev/shm/mq_heapdump.hprof
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=5
-XX:GCLogFileSize=30M
# -XX:+UseGCLogFileRotation、-XX:NumberOfGCLogFiles 和 -XX:GCLogFileSize 配置了GC日志文件的轮转。
# -XX:+CMSParallelRemarkEnabled 启用CMS的并行备注过程。
# -XX:+DisableExplicitGC 禁止系统调用System.gc()。
# -XX:+UseCMSCompactAtFullCollection 在进行完全垃圾回收后，对老年代进行压缩。


```

配置这些参数时，应考虑到应用程序的特点以及硬件资源的限制，可能需要通过实验调整参数来达到最佳效果。另外，由于CMS已经在较新的Java版本中被废弃，建议考虑使用G1或其他现代垃圾收集器。

## G1（Garbage-First）

G1（Garbage-First）垃圾收集器是一种服务器端垃圾收集器，适用于具有大内存需求的多处理器机器。G1旨在提供高吞吐量的同时，还能保持较低的延迟。启用G1收集器通常需要以下JVM参数：

1. `-XX:+UseG1GC`:
   - 启用G1垃圾收集器。

2. `-XX:G1NewSizePercent=<value>` 和 `-XX:G1MaxNewSizePercent=<value>`:
   - 设置新生代大小的初始百分比和最大百分比。

3. `-XX:MaxGCPauseMillis=<value>`:
   - 设置期望达到的最大GC停顿时间目标（以毫秒为单位）。

4. `-XX:G1HeapRegionSize=<value>`:
   - 设置G1的区域的大小，单位为MB。合理的区域大小可以提高G1的效率。

5. `-XX:ConcGCThreads=<value>`:
   - 设置并行标记阶段的线程数。

6. `-XX:InitiatingHeapOccupancyPercent=<value>`:
   - 触发并发GC周期的Java堆占用率阈值。

7. `-XX:G1HeapWastePercent=<value>`:
   - 设置重新回收的空间大小的百分比。

8. `-XX:+PrintGCDetails` 和 `-XX:+PrintGCDateStamps`:
   - 输出详细的GC日志信息以及GC发生的时间戳。

9. `-XX:+PrintGCApplicationStoppedTime`:
   - 打印应用程序因GC而停顿的时间。

10. `-XX:+PrintAdaptiveSizePolicy`:
    - 打印自适应大小调整策略的信息。

11. `-XX:G1ReservePercent=<value>`:
    - 设置作为保留空间的百分比，以减少晋升失败的风险。

示例配置G1垃圾收集器的JVM参数：

```shell
java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16M -XX:InitiatingHeapOccupancyPercent=45 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -jar your-application.jar
```

在配置这些参数时，通常需要根据应用程序的需求和特性进行测试和调整，以找到最佳的性能平衡点。例如，如果应用程序经常出现长时间的GC停顿，可以尝试调整`-XX:MaxGCPauseMillis`来降低停顿时间。同样，`-XX:G1HeapRegionSize`的调整也可以影响GC的效率和细粒度。

请注意，G1垃圾收集器在Java 7 Update 4及以后的版本中可用，并且从Java 9开始成为默认的垃圾收集器。在Java 8及以前的版本中，默认使用的是Parallel GC。

## ZGC（Z Garbage Collector）

ZGC（Z Garbage Collector）是Java虚拟机中的一种低延迟垃圾收集器，它在JDK 11中引入，并且针对多处理器机器上的大型堆设计，其主要目标是支持高吞吐量同时保持低延迟，即使是在几个TB的堆内存中也能维持毫秒级的停顿时间。以下是配置ZGC时可能会使用的一些重要JVM参数：

1. `-XX:+UseZGC`:
   - 启用ZGC垃圾收集器。

2. `-XX:ZHeapSize=<value>`:
   - 设置ZGC管理的堆内存的大小。

3. `-XX:ConcGCThreads=<value>`:
   - 设置并发GC线程的数量，这些线程用于并行处理GC操作。

4. `-XX:ZCollectionInterval=<value>`:
   - 设置强制GC周期的时间间隔（以秒为单位）。

5. `-XX:ZFragmentationLimit=<value>`:
   - 设置允许的堆碎片的百分比。

6. `-XX:+ZUncommit`:
   - 允许ZGC在系统空闲时释放未使用的堆内存给操作系统。

7. `-XX:ZUncommitDelay=<value>`:
   - 设置在释放内存之前，未使用的内存区域可以保留多长时间（以秒为单位）。

8. `-XX:ZProactive`:
   - 启用主动GC模式，ZGC会根据堆内存使用情况主动触发GC。

9. `-XX:+ZStatistics`:
   - 启用详细的GC统计信息。

10. `-XX:SoftMaxHeapSize=<value>`:
    - 设置软最大堆大小，ZGC将尝试在该限制下收回内存。

11. `-XX:ZMarkStackSpaceLimit=<value>`:
    - 设置标记堆栈空间的大小限制。

12. `-XX:ZProactiveGCLimit=<value>`:
    - 设置主动GC的触发阈值。

13. `-XX:ZAllocationSpikeTolerance=<value>`:
    - 设置分配高峰的容忍度。

14. `-XX:+ZVerifyViews`:
    - 启用ZGC的视图验证（主要用于调试和测试）。

15. `-XX:+PrintGC` 和 `-XX:+PrintGCDetails`:
    - 打印GC日志和详细信息。

ZGC的配置示例：

```shell
java -XX:+UseZGC -XX:ConcGCThreads=4 -XX:SoftMaxHeapSize=8G -XX:+PrintGC -XX:+PrintGCDetails -jar your-application.jar
```

在配置ZGC时，需要注意的是，与其他垃圾收集器相比，ZGC的默认配置已经能够提供良好的性能和低延迟。通常情况下，大多数应用无需过多调整参数即可获得满意的效果。如果需要进一步调优，可以根据应用程序的具体情况和性能测试结果，逐步调整上述参数。同时，ZGC在JDK 11及以后的版本中可用，在使用时请确保你的应用运行在支持ZGC的Java版本上。

## docker配置GC

当运行Java应用程序在Docker容器中时，配置垃圾收集器（GC）需要考虑容器的资源限制，确保JVM能够正确识别并适应容器环境。以下是一些与GC和Docker配置相关的关键建议：

1. **容器资源限制**:
   - 使用`-m`或`--memory`标志来限制Docker容器可以使用的内存量。
   - 使用`--cpus`标志限制容器可以使用的CPU核心数。

2. **JVM版本**:
   - 使用Java 8u131及以上版本，或者Java 9及以上版本，因为这些版本开始支持容器环境并能够识别容器提供的资源限制。

3. **JVM启动参数**:
   - 对于较新版本的JVM（Java 8u191、Java 10+），默认能够识别Docker容器的资源限制。如果使用较老的版本，请添加`-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`参数来使JVM识别容器的内存限制。
   - 对于CPU限制，较新版本的JVM也能够自动识别。如果需要手动指定，请使用`-XX:ParallelGCThreads`、`-XX:ConcGCThreads`等参数根据CPU限制进行调整。

4. **GC配置**:
   - 根据所选的垃圾收集器（如G1、ZGC、Shenandoah等），适当配置GC相关参数，如`-XX:MaxGCPauseMillis`、`-XX:InitiatingHeapOccupancyPercent`等，以优化GC性能。

5. **内存和堆大小**:
   - 使用`-Xms`和`-Xmx`来设置JVM的初始和最大堆大小。这些值应该考虑容器的内存限制。

6. **健康检查和应用监控**:
   - 使用Docker的健康检查和应用性能监控工具来监控Java应用的健康状态和性能指标。

7. **日志和诊断**:
   - 配置JVM参数以输出GC日志，如`-Xlog:gc`（Java 9+）或`-XX:+PrintGCDetails`等，以便于问题诊断。

8. **Java工具接入**:
   - 如果需要，可以配置JVM以允许远程调试和JMX监控。

一个Docker运行Java应用程序的示例命令可能如下：

```shell
docker run -m 2g --cpus 2 my-java-app:latest java -XX:+UseG1GC -Xms1g -Xmx1g -XX:MaxGCPauseMillis=200 -XX:+PrintGCDetails -jar /path/to/app.jar
```

在这个例子中，Docker容器被限制使用最多2GB的内存和2个CPU核心。JVM配置为使用G1垃圾收集器，并设置了初始和最大堆大小为1GB，同时设置了最大GC停顿时间为200ms，并打开了GC详细日志。
