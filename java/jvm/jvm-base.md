



## 常用配置JVM参考

```
-Xms2048M, -Xmx2048M, -XX:MetaspaceSize=384M, -XX:MaxMetaspaceSize=384M, -Xss256k, -XX:MaxTenuringThreshold=15, -Xnoclassgc, -XX:+UseParNewGC, -XX:+UseConcMarkSweepGC, -XX:+CMSParallelRemarkEnabled, -XX:+UseCMSCompactAtFullCollection, -XX:CMSInitiatingOccupancyFraction=65, -XX:+UseCMSInitiatingOccupancyOnly, -XX:+ExplicitGCInvokesConcurrent, -XX:CMSFullGCsBeforeCompaction=0, -XX:+PrintGCDateStamps, -XX:+PrintGCDetails, -XX:+PrintGCTimeStamps, -XX:+PrintHeapAtGC, -Xloggc:logs/JVMGC.log, -XX:+CMSClassUnloadingEnabled, -Djava.awt.headless=true, -Duser.timezone=GMT+08, -XX:+UnlockExperimentalVMOptions, -XX:+UseCGroupMemoryLimitForHeap, -javaagent:dest/agent-bootstrap.jar

```



```
mat 内存分析工具：

MemoryAnalyzer -data ./workspace


```

