# 单列安装7.0以上

## 增加es账户

```shell
sudo useradd es
sudo passwd es

# es 安装的目录
chown -R es:es /opt/module/es-7.8.0
```

## 给新创建的普通用户设置sudo权限

```shell
vim /etc/sudoers
最下面增加
es ALL(ALL) ALL

或使用一下
visudo # 使用root用户执行
es ALL(ALL) ALL  # 在 root ALL(ALL) ALL下面新增
```

## 调整，每个进程可以打开的文件数的限制 

```shell
vim /etc/security/limits.conf

# 增加内容 * 带表 Linux 所有用户名称 
es soft nofile 65536 #  * soft nofile 65536 
es hard nofile 65536 #  * hard nofile 65536

# 在启动 Elasticsearch 之前，将 ulimit -l unlimited 设置为 root。 
# 或者，在 /etc/security/limits.conf 中将 memlock 设置为无限制：
es soft memlock unlimited
es hard memlock unlimited

# 确保Elasticsearch用户可以创建的线程数至少为4096, ulimit -u 4096,指令或者本地目录
 * hard nproc 4096     # * 带表 Linux 所有用户名称 
```



## 调整，一个进程可以拥有的 VMA(虚拟内存区域)的数量,默认值为 65536 

```sh
sudo vim /etc/sysctl.conf

# 配置内容
vm.max_map_count=655360

# 执行生效
sudo sysctl -p
```

##### 将 TCP 重传的最大次数减少到 5, 五次重传对应于大约六秒的超时。

```shell
/etc/sysctl.conf

sysctl -w net.ipv4.tcp_retries2=5
```

## 制定临时目录(jna)

```shell
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djna.tmpdir=<path>"
```

## 交换内存对性能影响

##### On Linux systems, you can disable swap temporarily by running

交换对于性能和节点稳定性来说非常糟糕，应该不惜一切代价避免。 它可能导致垃圾收集持续几分钟而不是几毫秒，并且可能导致节点响应缓慢甚至与集群断开连接。 在弹性分布式系统中，让操作系统杀死节点更为有效。

```sh
第一种
# 禁用所有交换文件
sudo swapoff -a
第二种
# /etc/fstab   文件增加
第三种
#Linux 系统上可用的另一个选项是确保 sysctl 值 vm.swappiness 设置为 1。这会减少内核交换的倾向，并且在正常情况下不应导致交换，同时仍允许整个系统在紧急情况下进行交换。
```

##### 要启用内存锁，请在elasticsearch.yml中将bootstrap.memory_lock设置为true

```
bootstrap.memory_lock: true
```



## 6.0配置

```yaml
cluster.name:		配置elasticsearch的集群名称，默认是elasticsearch。建议修改成一个有意义的名称。   
node.name:		节点名，通常一台物理服务器就是一个节点，es会默认随机指定一个名字，建议指定一个有意义的名称，
方便管理一个或多个节点组成一个cluster集群，集群是一个逻辑的概念，节点是物理概念
path.data:	设置索引数据的存储路径，默认是es_home下的data文件夹，可以设置多个存储路径，用逗号隔开。      
path.logs: 		设置日志文件的存储路径，默认是es_home下的logs文件夹         
network.host:   设置绑定主机的ip地址，设置为0.0.0.0表示绑定任何ip，允许外网访问，生产环境建议设置为具体的ip。   
http.port: 9200		设置对外服务的http端口，默认为9200。      
transport.tcp.port: 9300 	集群结点之间通信端口      
discovery.zen.ping.unicast.hosts:[“host1:port”, “host2:port”, “…”]  设置集群中master节点的初始列表。
discovery.zen.ping.timeout: 3s  	设置ES自动发现节点连接超时的时间，默认为3秒，如果网络延迟高可设置大些。
discovery.zen.minimum_master_nodes: (nodes/2)+1 ,选主节点策略，防止脑裂
http.cors.enabled：	是否支持跨域，默认为false
http.cors.allow-origin：	当设置允许跨域，默认为*,表示支持所有域名
```



## 执行启动

```shell
./elasticsearch -d
```



