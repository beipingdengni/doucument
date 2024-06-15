## go rocketmq

### 安装

```bash
go get -U github.com/apache/rocketmq-client-go
```

执行消费

```go
package main

import (
	"context"
	"flag"
	"fmt"
	"os"

	"github.com/apache/rocketmq-client-go/v2"
	"github.com/apache/rocketmq-client-go/v2/primitive"
	"github.com/apache/rocketmq-client-go/v2/producer"
)

func main() {
	// 1. 创建主题，这一步可以省略，在send的时候如果没有topic，也会进行创建。
	//    CreateTopic("testTopic01")
	// 2.生产者向主题中发送消息
	var msg string
	defaultSendMsg := `json字符`
	flag.StringVar(&msg, "D", defaultSendMsg, "发现消息: msg")
	flag.Parse()
	fmt.Println("============================")
	fmt.Println(msg)
	SendSyncMessage(msg)
	fmt.Println("============================")
	// 3.消费者订阅主题并消费
	//    SubcribeMessage()
}

// 创建主题
// func CreateTopic(topicName string) {
// 	endPoint := []string{"10.233.129.21:9876"}
// 	// 创建主题
// 	testAdmin, err := admin.NewAdmin(admin.WithResolver(primitive.NewPassthroughResolver(endPoint)))
// 	if err != nil {
// 		fmt.Printf("connection error: %s\n", err.Error())
// 	}
// 	err = testAdmin.CreateTopic(context.Background(), admin.WithTopicCreate(topicName))
// 	if err != nil {
// 		fmt.Printf("createTopic error: %s\n", err.Error())
// 	}
// }

// 同步发送消息
func SendSyncMessage(message string) {
	// 发送消息
	endPoint := []string{"10.234.4.87:9876"}
	// 创建一个producer实例
	p, _ := rocketmq.NewProducer(
		producer.WithNameServer(endPoint),
		producer.WithRetry(2),
		producer.WithGroupName("GO_DEMO_APP"), // 集群中发应用名称
	)
	// 启动
	err := p.Start()
	if err != nil {
		fmt.Printf("start producer error: %s", err.Error())
		os.Exit(1)
	}
	// mss := primitive.NewMessage("需要发送的topic", []byte(message))
	mss := &primitive.Message{
		Topic: "需要发送的topic",
		Body:  []byte(message),
	  // Queue: &primitive.MessageQueue{
		// 	Topic:      "需要发送的topic",
		// 	BrokerName: "发送到对应broker名称服务器上",
		// 	QueueId:    2,
		// },
	}
	// 发送消息
	// mss.WithDelayTimeLevel(3)  // 给message设置延迟级别
	result, err := p.SendSync(context.Background(), mss)
	if err != nil {
		fmt.Printf("send message error: %s\n", err.Error())
	} else {
		fmt.Printf("send message seccess: result=%s\n", result.String())
	}
}

// func SubcribeMessage() {  // 消费者
// 	// 订阅主题、消费
// 	endPoint := []string{"10.233.129.21:9876"}
// 	// 创建一个consumer实例
// 	c, err := rocketmq.NewPushConsumer(consumer.WithNameServer(endPoint),
// 		consumer.WithConsumerModel(consumer.Clustering),
// 		consumer.WithGroupName("ConsumerGroupName"),
// 	)
// 	// 订阅topic
// 	err = c.Subscribe("testTopic01", consumer.MessageSelector{}, func(ctx context.Context, msgs ...*primitive.MessageExt) (consumer.ConsumeResult, error) {
// 		for i := range msgs {
// 			fmt.Printf("subscribe callback : %v \n", msgs[i])
// 		}
// 		return consumer.ConsumeSuccess, nil
// 	})
// 	if err != nil {
// 		fmt.Printf("subscribe message error: %s\n", err.Error())
// 	}
// 	// 启动consumer
// 	err = c.Start()
// 	if err != nil {
// 		fmt.Printf("consumer start error: %s\n", err.Error())
// 		os.Exit(-1)
// 	}
//	time.Sleep(24*time.Hour) //不能让主goroutine退出
// 	err = c.Shutdown()
// 	if err != nil {
// 		fmt.Printf("shutdown Consumer error: %s\n", err.Error())
// 	}
```

### 事物发送

```go
func main() {
	p, _ := rocketmq.NewTransactionProducer(
		&DemoListener{},
		producer.WithNsResolver(primitive.NewPassthroughResolver([]string{"127.0.0.1:9876"})),
		producer.WithRetry(1),
	)
  // 开启事物
	err := p.Start()
	if err != nil {
		fmt.Printf("start producer error: %s\n", err.Error())
		os.Exit(1)
	}
  // 发送消息
	res, err := p.SendMessageInTransaction(context.Background(),
		primitive.NewMessage("TopicTest5", []byte("Hello RocketMQ again ")))

	if err != nil {
		fmt.Printf("send message error: %s\n", err)
	} else {
		fmt.Printf("send message success: result=%s\n", res.String())
	}
	time.Sleep(5 * time.Minute) // 暂定5分钟
  // 关闭
	err = p.Shutdown()
	if err != nil {
		fmt.Printf("shutdown producer error: %s", err.Error())
	}
}
```







```
涉及数据表: job,customer_batch,job_exec_result,task_exec_result

```

