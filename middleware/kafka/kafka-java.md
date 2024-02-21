



## 发送producer

在使用Apache Kafka进行消息发送时，生产者（Producer）的配置参数对性能和可靠性都有显著影响。以下是一些关键的生产者配置参数：

1. `bootstrap.servers`:
   - Kafka集群的地址列表，格式为 `host1:port1,host2:port2,...`。
   - 生产者用这些服务器来建立初始连接到Kafka集群。

2. `key.serializer` 和 `value.serializer`:
   - 指定消息的key和value的序列化处理类。
   - 常用的序列化类有 `StringSerializer` 和 `ByteArraySerializer`。

3. `acks`:
   - 指定必须有多少个分区副本收到消息，生产者才会认为消息是写入成功的。
   - 常用的值有 `0`（不等待确认），`1`（只等待leader确认），`all` 或 `-1`（等待所有副本确认）。

4. `buffer.memory`:
   - 生产者可用来缓冲等待被发送到服务器的记录的总字节数。
   - 如果记录发送速度超过发送到服务器的速度，生产者会阻塞或抛出异常，取决于`max.block.ms`的设置。

5. `compression.type`:
   - 指定消息的压缩类型，如`none`、`gzip`、`snappy`、`lz4`或`zstd`。
   - 压缩可以减少网络传输和存储的数据量，但会增加CPU的使用。

6. `retries`:
   - 设置生产者从发送失败中恢复的次数。
   - 如果设置为大于0的值，生产者会在发送失败后重新发送。

7. `batch.size`:
   - 当多个消息被发送到同一个分区时，生产者会尝试将消息批量处理成一个batch，以减少请求次数。
   - 这个设置控制一个batch的最大大小（以字节为单位）。

8. `linger.ms`:
   - 控制生产者发送消息的频率。
   - 如果设置为0，则消息会立即发送，否则会在这个时间内等待更多的消息加入到同一个batch。

9. `max.block.ms`:
   - 在抛出超时异常前，生产者在发送消息时将阻塞的最大时间。
   - 这个设置影响`send()`和`partitionsFor()`方法。

10. `max.request.size`:
    - 控制生产者发送请求的大小。
    - 它可以避免发送过大的消息，超过broker端配置的上限。

11. `client.id`:
    - 任意字符串，用于追踪请求来源。

12. `security.protocol`:
    - 用于和Kafka集群通信的协议，如`PLAINTEXT`、`SSL`、`SASL_PLAINTEXT`、`SASL_SSL`。

这些是一些常见的配置参数，但Kafka提供了更多的配置选项来控制生产者的行为。具体配置应根据使用场景和需求进行调整。配置完成后，可以使用以下方式创建一个生产者实例：

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "all");
props.put("linger.ms", "0");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
// props.put("partitioner.class", "com.tbp.kafka.CustomPartitioner"); // 指定自定义分区器

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
// 批量发送
Properties props = new Properties();
// props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker:9092");
// props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384); // 设置批次大小为16KB
props.put(ProducerConfig.LINGER_MS_CONFIG, 10); // 设置linger.ms为10ms
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432); // 设置buffer.memory为32MB
// 批量发送开始，利用内存缓冲区
try (KafkaProducer<String, String> producer = new KafkaProducer<>(props)) {
  for (int i = 0; i < 100; i++) {
    // 创建ProducerRecord，可以指定key或者不指定
    ProducerRecord<String, String> record = new ProducerRecord<>("your_topic", "key-" + i, "value-" + i);
    // 发送消息，消息会被暂存在内存中，直到达到批次大小或linger.ms，然后批量发送
    Future<RecordMetadata> future = producer.send(record);
  }
  // 强制发送所有剩余的消息
  producer.flush();
}

// 单条立即发送
// 将linger.ms设置为0以确保消息立即发送
props.put(ProducerConfig.LINGER_MS_CONFIG, 0);
// 不需要设置batch.size，因为linger.ms为0意味着不会等待其他消息
producer.send(record); //不需要调用producer.flush()，因为linger.ms为0，消息会立即发送
```

#### 自定义分区发送CustomPartitioner

自定义分区策略需要实现`org.apache.kafka.clients.producer.Partitioner`接口

```java
ublic class CustomPartitioner implements Partitioner {

    @Override
    public void configure(Map<String, ?> configs) {
        // 这里可以获取配置信息
    }
  
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        int partitionCount = cluster.partitionCountForTopic(topic);
        if (keyBytes == null) {
            // 如果key为空，则随机分配一个分区
            return (int) (Math.random() * partitionCount);
        } else {
            // 如果key不为空，则根据key的hash值分配分区
            return Math.abs(Arrays.hashCode(keyBytes)) % partitionCount;
        }
    }

    @Override
    public void close() {
        // 清理资源
    }
}
```

在设置这些参数时，需要平衡吞吐量、延迟和可靠性的需求。例如，设置`acks=all`可以提高数据不丢失的机会，但可能会降低吞吐量，因为生产者需要等待所有副本的确认。同样地，使用压缩可以提高网络效率，但会增加CPU的负担。因此，正确的配置应该根据你的具体需求和资源限制来确定。



## 消费者consumer

在使用Apache Kafka进行消息消费时，消费者（Consumer）的配置参数同样对性能和可靠性有重要影响。以下是一些关键的消费者配置参数：

1. `bootstrap.servers`:
   - Kafka集群的地址列表，格式为 `host1:port1,host2:port2,...`。
   - 消费者用这些服务器来建立初始连接到Kafka集群。

2. `group.id`:
   - 消费者所属的消费组ID。
   - Kafka依靠消费组来进行消费者管理以及消息的负载均衡。

3. `key.deserializer` 和 `value.deserializer`:
   - 指定消息的key和value的反序列化处理类。
   - 常用的反序列化类有 `StringDeserializer` 和 `ByteArrayDeserializer`。

4. `enable.auto.commit`:
   - 指定是否启用自动提交消费位移。
   - 如果设置为`true`，消费者会在后台周期性地提交位移。

5. `auto.commit.interval.ms`:
   - 如果启用了自动提交位移，这个设置指定提交位移的频率。

6. `auto.offset.reset`:
   - 当没有初始位移或位移无效时，消费者应该从哪里开始消费。
   - 常用的值有`latest`（从最新的记录开始）、`earliest`（从最早的记录开始）。

7. `fetch.min.bytes`:
   - 服务器为了响应fetch请求应返回的最小数据量。
   - 如果不满足这个最小量，请求会等待直到有足够的可用数据。

8. `fetch.max.bytes`:
   - 服务器为了响应fetch请求应返回的最大数据量。
   - 这个值必须比broker端配置的`message.max.bytes`更小。

9. `max.poll.records`:
   - 单次调用`poll()`方法能返回的最大记录数。

10. `session.timeout.ms`:
    - 消费者在被认为死亡之前可以与服务器断开连接的时间。
    - 如果消费者在这个时间内没有发送心跳到broker，它就会被认为已经死亡，消费组将会触发重新平衡。

11. `heartbeat.interval.ms`:
    - 消费者在两次心跳之间的预期时间。
    - 通常设置比`session.timeout.ms`小。

12. `max.partition.fetch.bytes`:
    - 消费者从每个分区里返回的最大数据量。
    - 这个值必须比broker端配置的`max.message.size`更大。

13. `security.protocol`:
    - 用于和Kafka集群通信的协议，如`PLAINTEXT`、`SSL`、`SASL_PLAINTEXT`、`SASL_SSL`。

14. `partition.assignment.strategy`:
    - 分区分配策略，决定了如何在消费者之间分配分区。
    - 常用的值有`Range`和`RoundRobin`。
    - `Range`：这是默认的分区分配策略。它将每个主题的分区范围分配给消费者组中的消费者。例如，如果有10个分区和2个消费者，那么每个消费者会分配到5个分区。
    - `RoundRobin`：此策略将分区以循环方式一个个分配给消费者组中的消费者。
    - `StickyAssignor`：这个策略会尽量保持现有的分区分配，即使有消费者加入或者离开消费者组。它可以最小化因为分区重新分配而导致的消费者重平衡

消费者的配置示例：

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "test-group");
props.put("enable.auto.commit", "true");
// 自动提交的时间间隔 在spring boot 2.X 版本中这里采用的是值的类型为Duration 需要符合特定的格式，如1S,1M,2H,5D
props.put("auto.commit.interval.ms", "1000");
props.put("max.poll.records", "500"); // 一次拉取的最大消息数
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
// 指定自定义分配策略
// props.put("partition.assignment.strategy", "com.tbp.kafka.CustomPartitionAssignor"); 

// 随机分配策略
// props.put("partition.assignment.strategy", "org.apache.kafka.clients.consumer.RoundRobinAssignor");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
// 批量消费
try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
  consumer.subscribe(Arrays.asList("your_topic")); // 订阅的主题
  while (true) {
    // poll方法用于批量拉取消息，这里的参数是超时时间
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
      System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
      // 处理消息
    }
    // 手动提交偏移量
    consumer.commitAsync();
  }
}
```

#### 自定义的消费者分区分配策略

实现`org.apache.kafka.clients.consumer.ConsumerPartitionAssignor`接口

```java
public class CustomPartitionAssignor implements ConsumerPartitionAssignor {

    @Override
    public GroupAssignment assign(Cluster metadata, GroupSubscription groupSubscription) {
        // 实现自定义的分区分配逻辑
        // 返回每个消费者应该消费的分区的映射
        return null;
    }

    @Override
    public ByteBuffer subscriptionUserData(Set<String> topics) {
        // 如果需要，可以向订阅数据中添加自定义的用户数据
        return null;
    }

    @Override
    public String name() {
        // 返回分配策略的名称
        return "CustomPartitionAssignor";
    }
    
    // 其它必要的方法实现...
}
```

在设置这些参数时，同样需要根据具体的业务需求和资源限制进行调整。例如，如果需要更快的消费速度，可以增加`max.poll.records`的值，但这可能会增加客户端的内存压力。另外，`enable.auto.commit`设置为`true`可以简化消费位移的管理，但在某些情况下可能会导致消息的重复消费或丢失，因此某些场景下可能需要手动控制位移提交。

