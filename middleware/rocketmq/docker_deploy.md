

### Docker部署开发环境

但实例机器

```bash
docker pull apache/rocketmq:latest

# NameServer
docker run -d \
      --name rmqnamesrv \
      -e "JAVA_OPT_EXT=-Xms512M -Xmx512M -Xmn128m" \
      -p 9876:9876 \
      apache/rocketmq:latest \
      sh mqnamesrv

# Broker
docker run -d \
      --name rmqbroker \
      -p 10911:10911 \
      -p 10909:10909 \
      -p 10912:10912 \
      --link rmqnamesrv \
      -e "JAVA_OPT_EXT=-Xms512M -Xmx512M -Xmn128m" \
      -e "NAMESRV_ADDR=rmqnamesrv:9876" \
      apache/rocketmq:latest \
      sh mqbroker -c /home/rocketmq/rocketmq-4.9.2/conf/broker.conf
docker pull styletang/rocketmq-console-ng:latest

docker run -d \
    --name rmqconsole \
    -p 9800:8080 \
    --link rmqnamesrv \
    -e "JAVA_OPTS=-Drocketmq.namesrv.addr=rmqnamesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
    -t styletang/rocketmq-console-ng:latest
```

