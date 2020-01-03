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
```

