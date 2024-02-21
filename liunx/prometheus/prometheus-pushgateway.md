## pushgateway

https://github.com/prometheus/pushgateway

博客参考：https://blog.51cto.com/u_13236892/5968944

### 安装

> 注意参数：`-persistence.file `，`-persistence.interval`

```shell
docker run -d -p 9091:9091 prom/pushgateway

wget https://github.com/prometheus/pushgateway/releases/download/v1.7.0/pushgateway-1.7.0.darwin-arm64.tar.gz
tar zxvf pushgateway-1.7.0.darwin-arm64.tar.gz
# 启动
nohup pushgateway \
--web.enable-admin-api  \
--persistence.file="pushfile.txt" \
--persistence.interval=10m  &
# 注意:为了防止 pushgateway 重启或意外挂掉，导致数据丢失，可以通过 -persistence.file 和 -persistence.interval 参数将数据持久化下来。
# 访问地址: http://127.0.0.1:9091/metrics
```

### prometheus配置

> prometheus.yml，注意参数`honor_labels: true`
>
> 因为Prometheus配置pushgateway的时候，指定job和instance,但是它只表示pushgateway实例，不能真正表达收集数据的含义。所以在prometheus中配置pushgateway的时候，需要添加honor_labels: true参数，从而避免收集数据被push本身的job和instance被覆盖。不加honor_labels: true，会取gateway的job和instance，设置了的话会取push过来的数据，job必填，instance没有就为""空字符串

```yaml
  - job_name: pushgetway  
    honor_labels: true # 很重要
    honor_timestamps: true
    scrape_interval: 15s # scrape_interval: 拉取 targets 的默认时间间隔，即拉取业务监控数据的间隔时间。
    scrape_timeout: 10s # 拉取一个 target 的超时时间，即拉取业务监控数据接口的超时时间。
    metrics_path: /metrics #默认值/metrics
    scheme: http
    static_configs:
    - targets:
      - 127.0.0.1:9091
      labels:
          instance: pushgateway
```

### 推送消息操作

push_memory.sh （推送消息到网关）

> 推送消息: echo "test_metric 2333" | curl --data-binary @- http://127.0.0.1:9091/metrics/job/test_job
>
> 删除指标：curl -X DELETE http://127.0.0.1:9091/metrics/job/test_job 也可以在web界面进行删除

```shell
#!/bin/bash 
# desc push memory info 

total_memory=$(free  |awk '/Mem/{print $2}')
used_memory=$(free  |awk '/Mem/{print $3}')

job_name="custom_memory"
instance_name="10.0.0.11"
cat <<EOF | curl --data-binary @- http://127.0.0.1:9091/metrics/job/$job_name/instance/$instance_name
#TYPE custom_memory_total  gauge
custom_memory_total $total_memory
#TYPE custom_memory_used  gauge
custom_memory_used $used_memory
EOF
# 文件导出
curl -XPOST --data-binary @data.txt http://127.0.0.1:9091/metrics/job/nginx/instance/114.67.116.119
# data.txt 内容如下：
# http_request_total{code="200",domain="abc.cn"} 276683
# http_request_total{schema="https",domain="abc.cn"} 0
# http_request_time{code="total",domain="abc.cn"} 76335.809000
# http_request_maxip{clientip="114.67.116.119",domain="abc.cn"} 81
```

## java 使用

### mavan xml

```xml
<dependencies>
    <!-- Prometheus Java客户端 -->
    <dependency>
        <groupId>io.prometheus</groupId>
        <artifactId>simpleclient</artifactId>
        <version>0.16.0</version>
    </dependency>
    <!-- Prometheus Pushgateway客户端 -->
    <dependency>
        <groupId>io.prometheus</groupId>
        <artifactId>simpleclient_pushgateway</artifactId>
        <version>0.16.0</version>
    </dependency>
</dependencies>
```

### 使用案例

```java
import io.prometheus.client.CollectorRegistry;
import io.prometheus.client.Counter;
import io.prometheus.client.exporter.PushGateway;

public class PushgatewayExample {
    public static void main(String[] args) throws Exception {
       // 创建一个注册表
        CollectorRegistry registry = new CollectorRegistry();
      
        // 创建并注册一个指标（例如，一个Gauge）
        Gauge duration = Gauge.build()
                .name("my_spring_job_duration_seconds")
                .help("Duration of my batch job in seconds.")
                .register(registry);// 可以配置，registry，也可以不用
				
      	// 模拟作业运行时间
        duration.startTimer();
      	// 操作业务【算计算时间】
      	// 停止计时器并记录最终时间
        duration.setDuration();

        // 创建PushGateway
        PushGateway pushGateway = new PushGateway("http://localhost:9091");

        // 将指标推送到PushGateway
        pushGateway.push(CollectorRegistry.defaultRegistry, "job_name"); // 服务名称最好

        // 关闭PushGateway
        pushGateway.shutdown();
    }
}
```

