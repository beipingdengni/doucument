## Kafka 基本配置

#### 启动

```
// 创建topic
./kafka-topics.sh --create --zookeeper 192.168.119.131:2181 --replication-factor 1 --partitions 4 --topic yqtopic1

// 查看topic
 ./kafka-topics.sh --describe --zookeeper 192.168.119.131:2181 --topic yqtopic1
 
// 更新topic
./kafka-topics.sh --alter --zookeeper 192.168.119.131:2181 --topic yqtopic1 --partitions 12

// 查看日志文件，更新partion
ls -al /tmp/kafka-logs/

```

创建partition建议

```
选择合适的Partitions数量：
1、越多的分区可以提供更高的吞吐量
2、越多的分区需要打开更多地文件句柄
3、更多地分区会导致更高的不可用性
4、越多的分区可能增加端对端的延迟
5、越多的partition意味着需要客户端需要更多的内存

如何为Kafka集群选择合适的Partitions数量：https://www.cnblogs.com/fanguangdexiaoyuer/p/6066820.html

```



### 为什么不支持减少分区？

```
按照Kafka现有的代码逻辑而言，此功能完全可以实现，不过也会使得代码的复杂度急剧增大。
1、实现此功能需要考虑的因素很多，比如删除掉的分区中的消息该作何处理？
2、如果随着分区一起消失则消息的可靠性得不到保障；
3、如果需要保留则又需要考虑如何保留。直接存储到现有分区的尾部，消息的时间戳就不会递增，如此对于Spark、Flink这类需要消息时间戳（事件时间）的组件将会受到影响；
4、如果分散插入到现有的分区中，那么在消息量很大的时候，内部的数据复制会占用很大的资源，而且在复制期间，此主题的可用性又如何得到保障？
5、与此同时，顺序性问题、事务性问题、以及分区和副本的状态机切换问题都是不得不面对的。

反观这个功能的收益点却是很低，如果真的需要实现此类的功能，完全可以重新创建一个分区数较小的主题，然后将现有主题中的消息按照既定的逻辑复制过去即可。

虽然分区数不可以减少，但是分区对应的副本数是可以减少的，这个其实很好理解，你关闭一个副本时就相当于副本数减少了。不过正规的做法是使用kafka-reassign-partition.sh脚本来实现，具体用法可以自行搜索。

```

#### 修改副本

```
创建文件
partitions-topic.json 

{
  "partitions":[{
    "topic": "test1",
    "partition": 0,
    "replicas": [1,2]
  },{
    "topic": "test1",
    "partition": 1,
    "replicas": [0,3]
  },{
    "topic": "test1",
    "partition": 2,
    "replicas": [4,5]
  }],
  "version":1
}

// 执行副本搬迁
 ../bin/kafka-reassign-partitions.sh --zookeeper 127.0.0.1:2181 --reassignment-json-file partitions-topic.json --execute  

// 查看迁移情况
../bin/kafka-reassign-partitions.sh --zookeeper 127.0.0.1:2181 --reassignment-json-file partitions-topic.json --verify

kafka-reassign-partitions.sh工具来重新分布分区。该工具有三种使用模式：
    generate模式，给定需要重新分配的Topic，自动生成reassign plan（并不执行）
    execute模式，根据指定的reassign plan重新分配Partition
    verify模式，验证重新分配Partition是否成功

```





生产

```xml
 <entry key="bootstrap.servers" value="${bootstrap.servers}"/>
 <entry key="group.id" value="${web.env}-loyalty-producer"/>
<!-- 重试 -->
 <entry key="retries" value="10"/>
 <!-- 批量发送字节数。参数值越大，吞吐率越高，缺点是增大了响应延时。-->
 <entry key="batch.size" value="16384"/>
  <!-- 批量发送最大等待时间。如果待发送数据不足batch.size参数指定的大小，那么最多等待多长时间。 -->
 <entry key="linger.ms" value="1"/>
 <!-- 64M -->
 <entry key="buffer.memory" value="67108864"/>
 <!-- 
 			当buffer满了或者metadata获取不到(比如leader挂了)等情况下的最大阻塞时间
 			现在http调用是6秒，阻塞4秒，在http调用超时前抛异常 
 			send接口最大阻塞时间
-->
 <entry key="max.block.ms" value="4000"/>
 
 <entry key="key.serializer" value="org.apache.kafka.common.serialization.StringSerializer"/>
 <entry key="value.serializer" value="org.apache.kafka.common.serialization.StringSerializer"/>
```

#### 消费

```xml
<entry key="bootstrap.servers" value="${bootstrap.servers}"/>
<entry key="group.id" value="${web.env}-loyalty-new-Member-consumer"/>
<entry key="auto.commit.interval.ms" value="10000"/>
<entry key="session.timeout.ms" value="15000"/>
<entry key="auto.offset.reset" value="earliest"/>
<entry key="max.poll.records" value="50"/>

<!-- 是否自动提交offset。在当前kafka的实现中，是在poll中完成offset的自动提交处理。默认：true-->
<entry key="enable.auto.commit " value="false"/>

<entry key="key.deserializer" value="org.apache.kafka.common.serialization.StringDeserializer"/>
<entry key="value.deserializer" value="org.apache.kafka.common.serialization.StringDeserializer"/>

<!-- 
"max.poll.interval.ms" 默认：300s ，大于session.timeout.ms  两次poll之间的最大间隔。必须确认在该时间间隔内，能够处理完成所有的业务
"max.poll.records" 默认：Integer.MAX_VALUE， 如果单条消息超过 1 MB，建议设置为 1
"fetch.max.bytes" 默认：50M, 建议设置成公网带宽的一半（注意这里的单位是bytes，公网带宽的单位是bits）
"max.partition.fetch.bytes", 建议设置成fetch.max.bytes的三分之一或者四分之一
--> 
```



