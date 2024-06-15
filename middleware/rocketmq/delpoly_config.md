# 部署配置文件调整

https://blog.csdn.net/usagoole/article/details/126355207

### 操作系统优化

操作系统选择：选择Linux内核选择3.10版及以上。最好选择Centos7及以上

### 最大文件数

vim /etc/security/limits.conf

```
soft nofile 655360
hard nofile 655360
```

### 系统参数调整

vim /etc/sysctl.conf

```

vm.overcommit_memory=1
vm.drop_caches=1
vm.zone_reclaim_mode=0
vm.max_map_count=655360
vm.dirty_background_ratio=50
vm.dirty_ratio=50
vm.dirty_writeback_centisecs=360000
vm.page-cluster=3
vm.swappiness=1
```

### 参数说明

| 参数                               | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| overcommit_memory                  | 是否允许内存的过量分配<br/>overcommit_memory=0 当用户申请内存的时候，内核会去检查是否有这么大的内存空间<br/>overcommit_memory=1 内核始终认为，有足够大的内存空间，直到它用完了为止<br/>overcommit_memory=2 内核禁止任何形式的过量分配内存 |
| drop_caches                        | 写入的时候，内核会清空缓存，腾出内存来，相当于 sync 。drop_caches=1 会清空页缓存<br/>drop_caches=2 会清空<br/>inode 和目录树 drop_caches=3 都清空 |
| zone_reclaim_mode                  | zone_reclaim_mode=0系统会倾向于从其他节点分配内存<br/>zone_reclaim_mode=1系统会倾向于从本地节点回收 Cache 内存 |
| max_map_count                      | 定义了一个进程能拥有的最多的内存区域，默认为 65536           |
| dirty_background_ratio/dirty_ratio | 当 dirty cache 到了多少的时候，就启动 pdflush 进程，将 dirty cache 写回磁盘<br/>当有 dirty_background_bytes/dirty_bytes 存在的时候，dirty_background_ratio/dirty_ratio 是被自动计算的 |
| dirty_writeback_centisecs          | pdflush 每隔多久，自动运行一次（单位是百分之一秒）           |
| page-cluster                       | 每次 swap in 或者 swap out 操作多少内存页为 2 的指数 page-cluster=0 表示 1 页 page-cluster=1 表示 2 页 page-cluster=2 表示 4 页 page-cluster=3 表示 8 页 |
| swappiness                         | swappiness=0 仅在内存不足的情况下，当剩余空闲内存低于 vm.min_free_kbytes limit 时，使用交换空间<br/>swappiness=1 内核版本 3.5 及以上、Red Hat 内核版本 2.6.32-303 及以上，进行最少量的交换，而不禁用交换<br/>swappiness=10 当系统存在足够内存时，推荐设置为该值以提高性能<br/>swappiness=60 默认值<br/>swappiness=100 内核将积极的使用交换空间 |

### Master 部署配置文件【broker-a】

```properties
#Broker所属集群
brokerClusterName=MqCluster
namesrvAddr=10.28.1.32:9876;10.28.1.33:9876
brokerIP1=10.28.1.32
listenPort=10911
brokerName=broker-a
brokerId=0
#删除文件时间点,默认凌晨4点
deleteWhen=04
#文件保留时间,默认48小时
fileReservedTime=48
#Broker的角色
#-ASYNC_MASTER异步复制Master
#-SYNC_MASTER同步双写Master
#-SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#-ASYNC_FLUSH异步刷盘
#-SYNC_FLUSH同步刷盘
flushDiskType=ASYNC_FLUSH
autoCreateTopicEnable=false
autoCreateSubscriptionGroup=false
defaultTopicQueueNums=4
#自动创建订阅组,建议线下开启,线上关闭
#CommitLog日志文件大小，默认1G
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=6000000
#强制删除文件间隔时间（单位毫秒）
destroyMapedFileIntervalForcibly=120000
#定期检查Hanged文件间隔时间（单位毫秒）
redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=80
#限制的消息大小4M 65536
maxMessageSize=4194304
#刷CommitLog，至少刷几个PAGE
flushCommitLogLeastPages=4
#刷ConsumeQueue，至少刷几个PAGE
flushConsumeQueueLeastPages=2
#刷CommitLog，彻底刷盘间隔时间
flushCommitLogThoroughInterval=10000
#刷ConsumeQueue，彻底刷盘间隔时间
flushConsumeQueueThoroughInterval=60000
#并发send线程数，多线程来发送消息可能会出现broker busy
#sendMessageThreadPoolNums=1
sendMessageThreadPoolNums=32
#使用锁
useReentrantLockWhenPutMessage=true
#拉消息线程池数量
pullMessageThreadPoolNums=128
#发送消息线程等待时间，默认200ms
waitTimeMillsInSendQueue=5000
#系统页面缓存繁忙超时时间(翻译),默认值 1000
osPageCacheBusyTimeOutMills=5000

#存储路径
storePathRootDir=/export/Data/mq-data/store
storePathCommitLog=/export/Data/mq-data/store/commitlog
```

### Slave部署配置文件【broker-b】

```properties

```

