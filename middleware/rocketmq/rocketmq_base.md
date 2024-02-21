

## Java应该关闭日志打印

方法一

```bash
-Drocketmq.client.logLevel=\"ERROR\" \
```

方法二

```xml
<loggers>
  <root level="INFO">
    <appender-ref ref="RollingFile"/>
    <appender-ref ref="ErrorRollingFile"/>
  </root>
	<!-- 关键是配置 RocketmqClient 为warn -->
	<logger name="RocketmqClient" level="ERROR" additivity="false">
		<appender-ref ref="ErrorRollingFile"/>
	</logger>
</loggers>
```

## 使用RocketMQ时，需要注意以下几个关键点：

1. **集群设计**：根据业务需求设计合理的集群规模。考虑使用主从同步或异步复制，以及是否需要跨数据中心的复制。

2. **消息大小**：默认情况下，**RocketMQ单个消息的大小限制为4MB**。如果需要发送更大的消息，可以调整最大消息大小的配置，但这可能会影响性能。

3. **消息积压**：监控消息的积压情况。如果消费者处理速度慢于生产者的发送速度，可能会导致消息堆积，需要及时调整消费者的数量或性能。

4. **消息丢失**：了解不同消息发送和消费模式下的消息可靠性保证，比如可靠同步发送、可靠异步发送和单向发送。

5. **消息重试**：默认情况下，消费失败的消息会被重试，需要合理配置重试策略，如重试次数和延迟级别。

6. **顺序消息**：如果业务需要顺序消费，需要使用顺序消息特性，并确保生产者按照顺序发送消息。

7. **事务消息**：如果使用了事务消息，需要确保实现了正确的事务监听器，并处理好事务的回查逻辑。

8. **资源管理**：合理分配Topic和队列数量，避免过多的Topic和队列导致资源浪费。

9. **监控告警**：部署监控系统，对集群状态、性能指标和业务数据进行监控，并设置告警阈值。

10. **日志记录**：配置适当的日志级别，记录关键的业务和系统信息，以便故障排查和性能优化。

11. **版本兼容性**：升级RocketMQ时要注意版本兼容性，避免因升级引入不兼容的问题。

12. **持久化策略**：根据业务需求配置消息的持久化策略，例如是否需要刷盘策略。

13. **消费者幂等**：确保消费者处理消息的逻辑是幂等的，避免因为消息重复消费导致的问题。

14. **安全性**：配置网络和访问控制，确保消息通信的安全性。

15. **文档和社区**：阅读官方文档，关注社区动态，理解最佳实践和常见问题解决方案。

在部署和使用RocketMQ时，这些注意事项能帮助你更好地规划资源，保证系统的稳定性和可靠性。

## 生产者（Producer）相关配置：

1. `sendMsgTimeout`：发送消息时的超时时间，默认为3000毫秒。
2. `retryTimesWhenSendFailed`：同步模式下，发送消息失败后的重试次数，默认为2次。
3. `retryTimesWhenSendAsyncFailed`：异步模式下，发送消息失败后的重试次数，默认为2次。
4. `compressMsgBodyOverHowmuch`：指定消息体的压缩阈值，默认为4KB，超过该值将进行压缩。
5. `maxMessageSize`：允许的最大消息大小，默认为4MB。
6. `retryAnotherBrokerWhenNotStoreOK`：是否在消息发送失败时，自动切换到其他Broker进行重试，默认为false。

## 消费者（Consumer）相关配置：

1. `consumeThreadMin`：消费者的最小线程数，默认为20。
2. `consumeThreadMax`：消费者的最大线程数，默认为64。
3. `consumeMessageBatchMaxSize`：每次拉取的最大消息数量，默认为1。
4. `pullInterval`：拉消息的间隔时间，单位为毫秒，默认为0（即不间断拉取）。
5. `pullBatchSize`：每次拉取的消息数量，默认为32条。
6. `consumeTimeout`：消息消费超时时间，默认为15分钟。
7. `maxReconsumeTimes`：消息最大重试消费次数，默认为16次。
8. `messageModel`：消息模型，支持集群消费（`CLUSTERING`）和广播消费（`BROADCASTING`）。

## NameServer和Broker相关配置：

1. `brokerIP1`：Broker的IP地址，如果有多网卡，可以指定用于监听的网卡IP。
2. `brokerIP2`：Broker的备用IP地址，用于客户端连接。
3. `namesrvAddr`：NameServer的地址列表，多个NameServer地址用分号隔开。
4. `brokerClusterName`：Broker所属的集群名称。
5. `brokerName`：Broker的名称。
6. `defaultTopicQueueNums`：默认创建Topic时的队列数量。
7. `autoCreateTopicEnable`：是否允许Broker自动创建Topic，默认为true。
8. `autoCreateSubscriptionGroup`：是否允许Broker自动创建订阅组，默认为true。

## producer

### 同步发送消息、异步发送消息、单向发送消息

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class SyncProducer {
    public static void main(String[] args) throws Exception {
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        // 设置NameServer的地址
        producer.setNamesrvAddr("localhost:9876");
        // 启动Producer实例
        producer.start();
        
        for (int i = 0; i < 10; i++) {
            // 创建消息，并指定Topic，Tag和消息体
            Message msg = new Message("TopicTest", "TagA", ("Hello RocketMQ " + i).getBytes());
          
          	// 同步发送
            // 发送消息到一个Broker
            SendResult sendResult = producer.send(msg);
            // 通过sendResult返回消息是否成功送达
            System.out.printf("%s%n", sendResult);
          
          	// 异步发送数据
            producer.send(msg, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.printf("%-10d OK %s %n", index, sendResult.getMsgId());
                }
                
                @Override
                public void onException(Throwable e) {
                    System.out.printf("%-10d Exception %s %n", index, e);
                    e.printStackTrace();
                }
            });
          
          	// 单向发送消息，没有任何返回结果
            producer.sendOneway(msg);
        }
      	// 异步发送需要时间，等待一段时间后关闭Producer实例
        // producer.shutdown();
      
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

### 批量发送（分割消息）

```java
public class BatchProducer {
    public static void main(String[] args) throws Exception {
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        // 设置NameServer的地址
        producer.setNamesrvAddr("localhost:9876");
        // 启动Producer实例
        producer.start();

        // 创建消息集合
        List<Message> messages = new ArrayList<>();

        // 批量创建消息，这里假设所有消息都有相同的Topic和Tag
        for (int i = 0; i < 10; i++) {
            Message msg = new Message("TopicTest", "TagA", ("Hello RocketMQ " + i).getBytes());
            messages.add(msg);
        }

        // 批量发送消息
        // 注意：不同的消息Broker可能对批量消息的大小有限制，默认是不超过4MB
        SendResult sendResult = producer.send(messages);
      
      	// 消息大小限制：单个批次的消息大小不应超过4MB的默认限制。
      	// 如果消息总大小超过这个限制，需要将大批量消息拆分成多个小批量进行发送。
        final int SIZE_LIMIT = 1024 * 1024;  // 限制消息最大大小，例如：1MB
        ListSplitter splitter = new ListSplitter(messages, SIZE_LIMIT);
        while (splitter.hasNext()) {
            List<Message> listItem = splitter.next();
            producer.send(listItem);
        }


        // 打印发送结果
        System.out.printf("%s%n", sendResult);

        // 如果不再发送消息，关闭Producer实例
        producer.shutdown();
    }
}
```





## Consumer

RocketMQ 支持两种主要的消息消费模式：集群消费（Clustering）和广播消费（Broadcasting）。

1. **集群消费（Clustering）**：

   - 在这种模式下，同一个消费者组（Consumer Group）中的所有消费者共同消费某个主题（Topic）的消息。
   - 每条消息只会被消费者组中的一个消费者消费。
   - 如果一个主题有多个消息队列（Queue），那么这些队列会均匀地分配给消费者组中的每个消费者。
   - 这种模式适合于负载均衡和故障转移的场景。

2. **广播消费（Broadcasting）**：

   - 在广播消费模式下，消息会发送给消费者组中的所有消费者，每个消费者都会收到一份完整的消息副本。
   - 每条消息都会被消费者组中的每个消费者独立消费。
   - 这种模式适合于需要每个消费者都能接收到所有消息的场景，例如日志收集。

3. **RocketMQ 支持两种消息消费方式：推（Push）和拉（Pull）。**

   1. **推模式（Push）**：
      - 推模式下，消费者订阅主题后，Broker 会主动将消息推送给消费者。
      - 消费者只需要实现一个消息监听器来处理接收到的消息。
      - 推模式相对简单，不需要消费者主动去请求消息，适合大多数使用场景。
      - RocketMQ 的 `DefaultMQPushConsumer` 是使用推模式的消费者实现。
   2. **拉模式（Pull）**：
      - 拉模式下，消费者需要主动向Broker请求消息。
      - 消费者可以更精细地控制何时以及如何拉取消息，例如可以根据消费能力来控制拉取速率。
      - 拉模式给了消费者更多的控制权，但同时也增加了实现的复杂性。
      - RocketMQ 的 `DefaultMQPullConsumer` 是使用拉模式的消费者实现。

   大多数情况下，推荐使用推模式因为它简化了消费者的实现并能够很好地平衡负载。如果需要更精细的控制，例如在某些特定条件下暂停拉取或根据消费者的处理能力动态调整拉取速率，可以使用拉模式。

   1. **顺序消费消息监听器（MessageListenerOrderly）**：
      - 该监听器保证同一个消息队列（Message Queue）的消息按照顺序逐一消费。
      - 消费者会阻塞直到消息处理完成，适用于对顺序性有要求的业务场景。
   2. **并发消费消息监听器（MessageListenerConcurrently）**：
      - 该监听器允许消息并发消费，不保证消息的顺序。
      - 消费者可以同时处理多个消息，适用于对顺序性要求不高，但对吞吐量要求较高的场景。

下面代码是push推模式

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class Consumer {
    public static void main(String[] args) throws Exception {
        // 创建消费者，并设置消费者组名
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_group_name");

        // 设置NameServer地址
        consumer.setNamesrvAddr("nameserver1:9876;nameserver2:9876");

        // 订阅一个或多个Topic，以及Tag来过滤需要消费的消息
        consumer.subscribe("topic_name", "*");

        // 设置为集群消费模式
        // consumer.setMessageModel(MessageModel.CLUSTERING);

        // 设置为广播消费模式
        // consumer.setMessageModel(MessageModel.BROADCASTING);

        // 注册消息监听器
      	// 顺序处理消息
        consumer.registerMessageListener((MessageListenerOrderly) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                // 处理消息
                System.out.printf("%s Receive New Messages: %s %n", 
                                  Thread.currentThread().getName(), new String(msg.getBody()));
            }
            return ConsumeOrderlyStatus.SUCCESS;
        });
      
      	// 注册回调实现类来处理从broker拉取回来的消息
      	// 并发处理消息
        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                // 消费者获取消息逻辑
                System.out.printf("%s Receive New Messages: %s %n", 
                                  Thread.currentThread().getName(), new String(msg.getBody()));
            }
            // 标记该消息已经被成功消费
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 启动消费者
        consumer.start();
    }
}
```

下面代码是pull拉模式

```java
import org.apache.rocketmq.client.consumer.DefaultMQPullConsumer;
import org.apache.rocketmq.client.consumer.PullResult;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;
import java.util.Set;

public class PullConsumer {
    public static void main(String[] args) throws Exception {
        // 实例化Pull消费者
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("pull_consumer_group");
        // 设置NameServer地址
        consumer.setNamesrvAddr("localhost:9876");
        // 启动消费者
        consumer.start();

        // 获取指定Topic的消息队列集合
        Set<MessageQueue> mqs = consumer.fetchSubscribeMessageQueues("TopicTest");
        for (MessageQueue mq : mqs) {
            System.out.printf("Consume from the queue: %s%n", mq);
            // 无限循环拉取消息
            while (true) {
                try {
                  // 拉取消息，最后一个参数表示拉取的最大消息数量
                  PullResult pullResult = consumer.pullBlockIfNotFound(mq, null, getMessageQueueOffset(mq), 32);
                    // 获取消息集合
                    List<MessageExt> messageExtList = pullResult.getMsgFoundList();
                    if (messageExtList != null) {
                        for (MessageExt message : messageExtList) {
                            // 处理业务逻辑
                            System.out.println(new String(message.getBody()));
                        }
                    }
                    // 将新的消费偏移量更新到存储系统中
                    putMessageQueueOffset(mq, pullResult.getNextBeginOffset());
                  
                    // 处理不同的拉取结果
                    switch (pullResult.getPullStatus()) {
                        case FOUND:
                            // 找到消息，业务逻辑处理
                            break;
                        case NO_MATCHED_MSG:
                            // 没有匹配的消息
                            break;
                        case NO_NEW_MSG:
                            // 没有新消息，可以休息一会儿再拉
                            break;
                        case OFFSET_ILLEGAL:
                            // 偏移量非法，可能是消息被删除了或者其他原因
                            break;
                        default:
                            break;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        // 关闭消费者
        consumer.shutdown();
    }

    private static long getMessageQueueOffset(MessageQueue mq) {
        // 从存储系统中获取消息队列的消费偏移量
        // 这里返回0表示从头开始消费
        return 0;
    }

    private static void putMessageQueueOffset(MessageQueue mq, long offset) {
        // 将消息队列的消费偏移量存储到系统中
    }
}
```

