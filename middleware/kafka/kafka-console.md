## 启动服务

> ./kafka-server-start.sh -daemon ../config/server.properties

server.properties

```properties
zookeeper.connect=localhost:2181/k1 
```

## 创建topic

> ./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test-topic --partitions 1 --replication-factor 1 

## 生产

> ./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic 

## 消费

> ./kafka-console-consumer.sh --bootstrap-server localhost:9093 --topic test-topic --from-beginning

## 镜像复制同步

kafka2.4版本，推出一个新的mirror maker2（以下mm2即代表mirror maker2）。mirror maker2基于kafka connect工具，解决了上面说的大部分问题。

mirror maker2是基于kafka connect框架进行开发的，可以简单地将mirror maker2视作几个source connector和sink connector的组合。包括：

- MirrorSourceConnector, MirrorSourceTask：用来进行同步数据的connector
- MirrorCheckpointConnector, MirrorCheckpointTask：用来同步辅助信息的connector，这里的辅助信息主要是consumer的offset
- MirrorHeartbeatConnector, MirrorHeartbeatTask：维持心跳的connector

不过虽然mirror maker2岁基于kafka connect框架，但它却做了一定的改造，可以单独部署一个mirror maker2集群，当然也可以部署在kafka connect单机或kafka connect集群环境上。这部分后面介绍部署的时候再介绍。

和mm1一样，在最简单的主从备份场景中，mm2建议部署在目标（target）集群，即从远端消费然后本地写入。如果部署在源集群端，那么出错的时候可能会出现丢数据的情况

### shell脚本

```sh
./bin/kafka-mirror-maker.sh \
	--consumer.config ./config/consumer.properties \
	--producer.config ./config/producer.properties \
	--num.streams 4 \
	--whitelist test-topic
# --whitelist=".*"
```

### consumer.config

```properties
bootstrap.servers=localhost:9092
client.id=test.k1Andk2
group.id=test.k1Andk2
partition.assignment.strategy=org.apache.kafka.clients.consumer.RoundRobinAssignor
```

### producer.config

```properties
bootstrap.servers=localhost:9093
client.id=test.k1Andk2
```

### 基本配置

| 参数名                      | 参数含义                                                     |
| --------------------------- | ------------------------------------------------------------ |
| whitelist                   | 指定一个正则表达式，指定拷贝源集群中的哪些topic。比如 a\|b  表示拷贝源集群上连个topic的数据a和b。注意，当使用新版本consumer时必须指定该参数。为了方便使用，可以将\|替换为, |
| blacklist                   | 指定一个正则表达式，屏蔽指定topic的拷贝。注意，该参数只使用于老版本consumer |
| abort.on.send.failure       | 若设置为true，当发送失败时则关闭MirrorMaker                  |
| consumer.config             | 指定MirrorMaker下consumer的属性文件。至少指定bootstrap.server和group.id |
| producer.config             | 指定MirrorMaker下producer的属性文件                          |
| consumer.rebalance.listener | 指定MirrorMaker使用的consumer rebalance监听器                |
| rebalance.listener.args     | 指定MirrorMaker使用的consumer rebalance监听器的参数，与consumer.rebalance.listener一同使用 |
| message.handler             | 指定消息处理器类。消息处理器在consumer获取消息与producer发送消息之间调用 |
| message.handler.args        | 指定消息处理器类的参数，与message.handler一同使用            |
| num.streams                 | 指定MirrorMaker线程数。默认是1                               |
| offset.conmmit.interval.ms  | 指定MirrorMaker位移提交间隔，默认值为1分钟                   |
| help                        | 打印帮助信息                                                 |

### java 

#### maven xml

```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>connect-mirror</artifactId>
  <version>2.4.0</version>
</dependency>
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>connect-mirror-client</artifactId>
  <version>2.4.0</version>
</dependency>

<!-- 
	./connect-mirror \
		--source-cluster <source_cluster> 
		--destination-cluster <destination_cluster> 
		--source-group <source_group> 
		--destination-group <destination_group>
-->
```

### Java代码使用

```java
import io.confluent.connect.mirror.MirrorClientConfig;
import io.confluent.connect.mirror.MirrorClient;
import org.apache.kafka.clients.consumer.ConsumerRecord;

public class MirrorClientExample {
    public static void main(String[] args) {

        MirrorClientConfig config = new MirrorClientConfig();
        config.put("sourceClusterBootstrapServers", "source.kafka.server1:9092");
        config.put("destinationClusterBootstrapServers","destination.kafka.server1:9092");

        MirrorClient client = new MirrorClient(config);
        // 使用 MirrorClient 执行相关任务
        // 例如：复制数据、监控连接镜像等
        // 从源集群消费数据
        client.consumeFromSourceCluster("source-topic", record -> {
            // 处理从源集群接收到的记录
            // 将记录发送到目标集群
            client.sendToDestinationCluster("destination-topic", record);
        });
    }
}

```

### connect-mirror-client 常见的配置

- `sourceClusterBootstrapServers`：源集群的引导服务器地址
- `destinationClusterBootstrapServers`：目标集群的引导服务器地址
- `sourceClusterGroupId`：源集群的消费者组ID
- `destinationClusterGroupId`：目标集群的消费者组ID
- `sourceClusterSecurityProtocol`：源集群的安全协议
- `destinationClusterSecurityProtocol`：目标集群的安全协议
- 其他配置选项可以根据需要进行设置，如连接超时、序列化器等。
