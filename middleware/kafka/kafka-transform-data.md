# kafka如何扩容

https://blog.51cto.com/u_16213646/9666178



```

```

最近我们生产环境的kafka集群有增加节点的需求，然而kafka在新增节点后并不会像elasticsearch那样感知到新节点加入后自动将数据reblance到新集群中，因此这个过程需要我们手动分配。一番折腾之后，实现了增加kafka集群节点并将原有数据均匀分配到扩容后的集群。下面结合一个例子谈一下整个过程。

## 一. 环境说明

1.假定当前的cluster中只有（0，1，2）三个kafka节点，有一个名为20191009的topic，该topic有2个replica，均匀分布在三个节点上.

2.我们要做的是在cluster中新增两个节点（记为3）后，将的数据均匀分到新集群中的0,1,2,3 四个节点上。

## 二、kafka集群扩容

```shell
1. scp -r kafka_2.11-0.11.0.3  root@172.18.186.29:/SERVICES01/  ##拷贝kafka文件到要扩容的机器上
2. cd /SERVERS01/kafka_2.11-0.11.0.3/config  ##进入172.18.186.29机器的kafka配置文件夹
3. vi server.properties   ##打开配置文件 修改kafka配置以下参数:
      broker.id=3
      listeners=PLAINTEXT://172.18.186.29:9092
4. wq    #保存文件
5. bin/kafka-server-start.sh -daemon config/server.properties  ##启动kafka
```

## 三 .kafka数据迁移

```shell
1. bin/kafka-topics.sh --list --zookeeper 172.18.163.203:2181,172.18.163.204:2181,172.18.163.205:2181  
##查看要topic列表和选择要迁移的topic名称

2. cat << EOF > topic-to-move.json
{"topics": [{"topic": "201909ack"},{"topic": "20191009"}],
"version":1
}EOF
##确定要重启分配分区的主题，新建topics-to-move.json   迁移json文件,迁移topic名称为”201909ack”,”20191009” 两个

3. bin/kafka-reassign-partitions.sh --zookeeper 172.18.163.203:2181,172.18.163.204:2181,172.18.163.205:2181 --topics-to-move-json-file topics-to-move.json --broker-list "0,1,2,4" –generate

##通过kafka-reassign-partitions.sh 获取重新分配方案,--broker-lsit 的参数 "0,1,2,4"是指集群中每个broker的id，由于我们是需要将所有topic均匀分配到扩完结点的4台机器上，所以要指定。同理，当业务改变为将原来的所有数据从旧节点（0,1,2）迁移到新节点（4）实现数据平滑迁移，这时的参数应"4",执行后会出现以下内容:
Current partition replica assignment       ##当前分区副本分配
{"version":1,"partitions":[{"topic":"20191009","partition":0,"replicas":[1,0]},{"topic":"201909ack","partition":0,"replicas":[0,2]}]}

Proposed partition reassignment configuration ##建议的分区重新分配配置
{"version":1,"partitions":[{"topic":"20191009","partition":0,"replicas":[2,0]},{"topic":"201909ack","partition":0,"replicas":[2,0]}]}

4. cat << EOF > expand-cluster-reassignment.json
{"version":1,"partitions":[{"topic":"20191009","partition":0,"replicas":[2,0]},{"topic":"201909ack","partition":0,"replicas":[2,0]}]}
EOF

##改变后的分布配置保存为expand-cluster-reassignment.json

5.bin/kafka-reassign-partitions.sh --zookeeper 172.18.163.203:2181,172.18.163.204:2181,172.18.163.205:2181 --reassignment-json-file expand-cluster-reassignment.json --execute

##使用-execute执行迁移计划

6. bin/kafka-reassign-partitions.sh --zookeeper 172.18.163.203:2181,172.18.163.204:2181,172.18.163.205:2181  --reassignment-json-file expand-cluster-reassignment.json --verify

##使用--verify进行迁移结果的验证,出现以下则迁移成功:
  Status of partition reassignment:
Reassignment of partition [20191009,0] completed successfully
Reassignment of partition [201909ack,0] completed successfully
```

## 四. kafka Topic迁移后验证

```shell
1.bin/kafka-console-producer.sh --broker-list 172.18.163.203:9092,172.18.163.204:9092,172.18.163.205:9092 --topic 20191009

##在172.18.160.203机器执行迁移的TOPIC生产者

2. bin/kafka-console-consumer.sh --zookeeper 172.18.163.203:2181,172.18.163.204:2181,172.18.163.205:2181 --topic 20191009 --from-beginning

##在新扩容的机器上执行迁移的TOPIC消费者,正常消费所有数据,则再次验证TOPIC迁移成功
```



## 五 .总结说明

操作中三个小技巧：

1.主题量很多的是就不要一个一个复制粘贴了，用excel的拼接函数，还是很方便

2. 最后一步验证中，主题很多的时候，会有很多在为未完成的输出语句夹杂其中。在语句后面加上  | grep -c "progress"就知道有多少分区还没完成，输出为0的时候就是完成了。

总结和知识点理解

1、kafka新建主题时的分区分配策略：随机选取第一个分区节点，然后往后依次增加。例如第一个分区选取为1，第二个分区就是2，第三个分区就是3.  1，2，3是brokerid。不会负载均衡，所以要手动重新分配分区操作，尽量均衡。

2、在生产的同时进行数据迁移会出现重复数据。所以迁移的时候避免重复生产数据，应该停止迁移主题的生产。同时消费不会，同时消费之后出现短暂的leader报错，会自动恢复。

3、新增了broker节点，如果有主题的分区在新增加的节点上，生产和消费的客户端都应该在hosts配置文件中增加新增的broker节点，否则无法生产消费，但是也不报错。