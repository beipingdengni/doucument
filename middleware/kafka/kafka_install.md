# UI界面

可以使用安装

https://github.com/provectus/kafka-ui?utm_source=gold_browser_extension



## 部署使用注意发送消息内容大小

message.max.bytes=5242880

在生产者端配置max.request.size，这是单个消息最大字节数，根据实际调整，max.request.size 必须小于 message.max.bytes 以及消费者的 max.partition.fetch.bytes。这样消息就能不断发送。
