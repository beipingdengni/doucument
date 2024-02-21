# micrometer

参考学习博客地址

[深入分析基于Micrometer和Prometheus实现度量和监控的方案](https://www.cnblogs.com/throwable/p/13257557.html)

## maven

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-core</artifactId>
    <version>1.12.2</version>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.12.2</version>
</dependency>
```

## Micrometer提供的度量类库

> `Meter`是指一组用于收集应用中的度量数据的接口，Meter单词可以翻译为"米"或者"千分尺"，但是显然听起来都不是很合理，因此下文直接叫`Meter`，直接当成一个专有名词，理解它为度量接口即可。`Meter`是由`MeterRegistry`创建和保存的，可以理解`MeterRegistry`是`Meter`的工厂和缓存中心，一般而言每个JVM应用在使用Micrometer的时候必须创建一个`MeterRegistry`的具体实现。Micrometer中，`Meter`的具体类型包括：`Timer`，`Counter`，`Gauge`，`DistributionSummary`，`LongTaskTimer`，`FunctionCounter`，`FunctionTimer`和`TimeGauge`。下面分节详细介绍这些类型的使用方法和实战使用场景。而一个`Meter`具体类型需要通过名字和`Tag`(这里指的是Micrometer提供的Tag接口)作为它的唯一标识，这样做的好处是可以使用名字进行标记，通过不同的`Tag`去区分多种维度进行数据统计

## MeterRegistry(注册)

`MeterRegistry`在`Micrometer`是一个抽象类，主要实现包括：

- 1、`SimpleMeterRegistry`：每个`Meter`的最新数据可以收集到`SimpleMeterRegistry`实例中，但是这些数据不会发布到其他系统，也就是数据是位于应用的内存中的。
- 2、`CompositeMeterRegistry`：多个`MeterRegistry`聚合，内部维护了一个`MeterRegistry`的列表。
- 3、全局的`MeterRegistry`：工厂类`io.micrometer.core.instrument.Metrics`中持有一个静态`final`的`CompositeMeterRegistry`实例`globalRegistry`

### SimpleMeterRegistry

> 最简单注册

```java
MeterRegistry registry = new SimpleMeterRegistry();
Counter counter = registry.counter("counter");
counter.increment();
```

### CompositeMeterRegistry

> 复合注入（多个）

```java
CompositeMeterRegistry composite = new CompositeMeterRegistry();

Counter compositeCounter = composite.counter("counter");
compositeCounter.increment(); // <- 实际上这一步操作是无效的,但是不会报错

SimpleMeterRegistry simple = new SimpleMeterRegistry();
composite.add(simple);  // <- 向CompositeMeterRegistry实例中添加SimpleMeterRegistry实例

compositeCounter.increment();  // <-计数成功
```

### MeterRegistry

> 自定义注册

```java
Metrics.addRegistry(new SimpleMeterRegistry());
Counter counter = Metrics.counter("counter", "tag-1", "tag-2");
counter.increment();
```

## Tag与Meter的命名

> `Micrometer`中，`Meter`的命名约定使用英文逗号(`dot`，也就是".")分隔单词。但是对于不同的监控系统，对命名的规约可能并不相同，如果命名规约不一致，在做监控系统迁移或者切换的时候，可能会对新的系统造成破坏。`Micrometer`中使用英文逗号分隔单词的命名规则，再通过底层的命名转换接口`NamingConvention`进行转换，最终可以适配不同的监控系统，同时可以消除监控系统不允许的特殊字符的名称和标记等。开发者也可以覆盖`NamingConvention`实现自定义的命名转换规则：`registry.config().namingConvention(myCustomNamingConvention);`。在`Micrometer`中，对一些主流的监控系统或者存储系统的命名规则提供了默认的转换方式，例如当我们使用下面的命名时候

```java
MeterRegistry registry = ...
registry.timer("http.server.requests");
```

对于不同的监控系统或者存储系统，命名会自动转换如下：

- 1、Prometheus - http_server_requests_duration_seconds。
- 2、Atlas - httpServerRequests。
- 3、Graphite - http.server.requests。
- 4、InfluxDB - http_server_requests。

其实`NamingConvention`已经提供了5种默认的转换规则：dot、snakeCase、camelCase、upperCamelCase和slashes。

### tag标签规范和含义

> 另外，`Tag`（标签）是`Micrometer`的一个重要的功能，严格来说，一个度量框架只有实现了标签的功能，才能真正地多维度进行度量数据收集。Tag的命名一般需要是有意义的，所谓有意义就是可以根据`Tag`的命名可以推断出它指向的数据到底代表什么维度或者什么类型的度量指标。假设我们需要监控数据库的调用和Http请求调用统计，一般推荐的做法是

```java
MeterRegistry registry = ...
registry.counter("database.calls", "db", "users");
registry.counter("http.requests", "uri", "/api/users");

// 这样，当我们选择命名为"database.calls"的计数器，我们可以进一步选择分组"db"或者"users"分别统计不同分组对总调用数的贡献或者组成。一个反例如下：
registry.counter("calls", "class", "database", "db", "users");
registry.counter("calls", "class", "http", "uri", "/api/users");

// 通过命名"calls"得到的计数器，由于标签混乱，数据是基本无法分组统计分析，这个时候可以认为得到的时间序列的统计数据是没有意义的。可以定义全局的Tag，也就是全局的Tag定义之后，会附加到所有的使用到的Meter上(只要是使用同一个MeterRegistry)，全局的Tag可以这样定义
registry.config().commonTags("stack", "prod", "region", "us-east-1");
// 和上面的意义是一样的
registry.config().commonTags(Arrays.asList(Tag.of("stack", "prod"), Tag.of("region", "us-east-1"))); 

// 我们需要过滤一些必要的标签或者名称进行统计，或者为Meter的名称添加白名单，这个时候可以使用MeterFilter
registry.config()
    .meterFilter(MeterFilter.ignoreTags("http"))
    .meterFilter(MeterFilter.denyNameStartsWith("jvm"));
// 示忽略"http"标签，拒绝名称以"jvm"字符串开头的Meter。更多用法可以参详一下MeterFilter这个类

```

## 指标 Meter

### Counter(计数器)

> 计数器记录单一计数指标，该Counter接口允许按固定数量递增，该数量必须为正数，可以用来统计无上限的数据。
>
> > 适用于一些增长类型的统计，例如下单、支付次数、`HTTP`请求总量记录等等
>
> ```java
> Counter counter = Counter.builder("name")  //名称
> 				.baseUnit("unit") //基础单位
> 				.description("desc") //描述
> 				.tag("tagKey", "tagValue")  //标签
> 				.register(new SimpleMeterRegistry());//绑定的MeterRegistry
> 		counter.increment();
> ```

### Timer(计时器)

> 用于测量短时延迟和此类事件的频率， 适用于记录耗时比较短的事件的执行时间，通过时间分布展示事件的序列和发生频率
>
> > 1、记录指定方法的执行时间用于展示。
> >
> > 2、记录一些任务的执行时间，从而确定某些数据来源的速率，例如消息队列消息的消费速率等
>
> ```java
> // 静态方法处理
> Timer timer = ...
> timer.record(() -> dontCareAboutReturnValue());
> timer.recordCallable(() -> returnValue());
> 
> // 包装处理
> Runnable r = timer.wrap(() -> dontCareAboutReturnValue());
> Callable c = timer.wrap(() -> returnValue());
> 
> // Timer timer = Metrics.timer("timer", "createOrder", "cost");
> // timer.record(() -> createOrder(order1));
> // 或是
> Timer timer = Timer
>     .builder("my.timer")
>     .description("a description of what this timer does") // 可选
>     .tags("region", "test") // 可选
>     .register(registry);
> 
> // 计算耗时
> Timer.Sample sample = Timer.start(registry);
> // 这里做业务逻辑
> sample.stop(registry.timer("my.timer", "response", response.status()));
> 
> //获取结果
> //获取总耗时
> System.out.println(timer.totalTime(TimeUnit.MILLISECONDS));
> //获取计数
> System.out.println(timer.count());
> //获取最大耗时
> System.out.println(timer.max(TimeUnit.MILLISECONDS));
> ```
>
> 

### LongTaskTimer(长任务计时器)

> 长任务计时器是一种特殊类型的计时器，可让您在正在测量的事件仍在运行时测量时间。一个普通的 Timer 只记录任务完成后的持续时间。
>
> > `@Scheduled`和`@Timed`注解，基于`spring-aop`完成定时调度任务的总耗时记录
>
> ```java
> @Timed(value = "aws.scrape", longTask = true)
> @Scheduled(fixedDelay = 360000)
> void scrapeResources() {
>     //这里做相对耗时的业务逻辑
> }
> 
> MeterRegistry meterRegistry = new SimpleMeterRegistry();
> LongTaskTimer longTaskTimer = meterRegistry.more().longTaskTimer("longTaskTimer");
> longTaskTimer.record(() -> {
>   //这里编写Task的逻辑
> });
> //或者这样
> Metrics.more().longTaskTimer("longTaskTimer").record(()-> {
>   //这里编写Task的逻辑
> });
> ```

### Gauge (仪表盘)

> 一般用来统计有上限可增可减的数据，仪表是获取当前值的句柄。仪表的典型示例是集合或映射的大小或处于运行状态的线程数。
>
> > 1、有自然(物理)上界的浮动值的监测，例如物理内存、集合、映射、数值等。
> >
> > 2、有逻辑上界的浮动值的监测，例如积压的消息、（线程池中）积压的任务等，其实本质也是集合或者映射的监测。
>
> ```java
> List<String> list = registry.gauge("listGauge", Collections.emptyList(), new ArrayList<>(), List::size); 
> List<String> list2 = registry.gaugeCollectionSize("listSize2", Tags.empty(), new ArrayList<>()); 
> Map<String, Integer> map = registry.gaugeMapSize("mapGauge", Tags.empty(), new HashMap<>());
> 
> //计数
> AtomicInteger n = registry.gauge("numberGauge", new AtomicInteger(0));
> n.set(1);
> 
> //一般我们不需要操作Gauge实例
> Gauge gauge = Gauge
>     .builder("gauge", myObj, myObj::gaugeValue)
>     .description("a description of what this gauge does") // 可选
>     .tags("region", "test") // 可选
>     .register(registry);
> 
> BlockingQueue<String> QUEUE = new ArrayBlockingQueue<>(500);
> BlockingQueue<String> REAL_QUEUE MR.gauge("messageGauge", QUEUE, Collection::size);
> REAL_QUEUE.put("发送的消息体");
> new Thread(()->{
>   while (true) {
>     // 消费获取
>     Message message = REAL_QUEUE.take();
>   }
> }).start();
> 
> ```

### TimeGauge(跟踪时间值的专用量规)

> TimeGauge是一个跟踪时间值的专用量规，可缩放到每个注册表实现所期望的基本时间单位。
>
> ```java
> AtomicInteger count = new AtomicInteger();
> TimeGauge.Builder<AtomicInteger> timeGauge = TimeGauge.builder("timeGauge", count,TimeUnit.SECONDS, 
>                                                                AtomicInteger::get);
> timeGauge.register(new SimpleMeterRegistry());
> count.addAndGet(10086);
> count.set(1);
> //输出
> // name:timeGauge,tags:[],type:GAUGE,value:[Measurement{statistic='VALUE', value=10086.0}]
> // name:timeGauge,tags:[],type:GAUGE,value:[Measurement{statistic='VALUE', value=1.0}]
> ```
>
> 

### DistributionSummary（分布摘要跟踪事件的分布）

> 它在结构上类似于定时器，但记录的是不代表时间单位的值。例如，您可以使用分布摘要来衡量到达服务器的请求的负载大小。
>
> > 不依赖于时间单位的记录值的测量，例如服务器有效负载值，缓存的命中率等。
>
> ```java
> DistributionSummary summary = registry.summary("response.size");
> 
> DistributionSummary summary = DistributionSummary
>     .builder("response.size")
>     .description("a description of what this summary does") // 可选
>     .baseUnit("bytes") // 可选
>     .tags("region", "test") // 可选
>     .scale(100) // 可选
>     .register(registry);
> 
> // 使用案例
> public class DistributionSummaryMain {
>     private static final DistributionSummary DS = DistributionSummary.builder("cacheHitPercent")
>             .register(new SimpleMeterRegistry());
> 
>     private static final LoadingCache<String, String> CACHE = CacheBuilder.newBuilder()
>             .maximumSize(1000)
>             .recordStats()
>             .expireAfterWrite(60, TimeUnit.SECONDS)
>             .build(new CacheLoader<String, String>() {
>                 @Override
>                 public String load(String s) throws Exception {
>                      return selectFromDatabase();
>                 }
>             });
> 
>     public static void main(String[] args) throws Exception {
>         String key = "doge";
>         String value = CACHE.get(key);
>         record();
>     }
> 
>     private static void record() throws Exception {
>         CacheStats stats = CACHE.stats();
>         BigDecimal hitCount = new BigDecimal(stats.hitCount());
>         BigDecimal requestCount = new BigDecimal(stats.requestCount());
>       	// 计算缓存，命中率
>         DS.record(hitCount.divide(requestCount, 2, BigDecimal.ROUND_HALF_DOWN).doubleValue());
>     }
> }
> ```
>
> 

### FunctionCounter(函数计数器)

> 在函数编程中可以传递一个函数，在需要时调用函数进行获取数据。

### FunctionTimer(函数计时器)

> 在函数编程中可以传递一个函数，在需要时调用函数进行获取数据。

### print输出

```java
private static final SimpleMeterRegistry R = new SimpleMeterRegistry();
// 处理逻辑
Search.in(R).meters().forEach(each -> {
            StringBuilder builder = new StringBuilder();
            builder.append("name:")
                    .append(each.getId().getName())
                    .append(",tags:")
                    .append(each.getId().getTags())
                    .append(",type:").append(each.getId().getType())
                    .append(",value:").append(each.measure());
            System.out.println(builder.toString());
        });
```



# 演示

## 入门代码

访问地址可以查看到监控：http://localhost:8080/prometheus

```java
import com.sun.net.httpserver.HttpServer;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.composite.CompositeMeterRegistry;
import io.micrometer.core.instrument.simple.SimpleMeterRegistry;
import io.micrometer.prometheus.PrometheusConfig;
import io.micrometer.prometheus.PrometheusMeterRegistry;


import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;

public class MicrometerTest
{
    public static void main( String[] args )
    {
        //组合注册表
        CompositeMeterRegistry composite = new CompositeMeterRegistry();
        //内存注册表
        MeterRegistry registry = new SimpleMeterRegistry();
        composite.add(registry);
        //普罗米修斯注册表
        PrometheusMeterRegistry prometheusRegistry = new PrometheusMeterRegistry(PrometheusConfig.DEFAULT);
        composite.add(prometheusRegistry);
        //计数器
        Counter compositeCounter = composite.counter("counter");
        //计数
        compositeCounter.increment();
      	// 打印查看
      	System.out.println(compositeCounter.measure()); // // [Measurement{statistic='counter', value=1.0}]
        try {
            //暴漏8080端口来对外提供指标数据
            HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
            server.createContext("/prometheus", httpExchange -> {
                //获取普罗米修斯指标数据文本内容
                String response = prometheusRegistry.scrape();
                //指标数据发送给客户端
                httpExchange.sendResponseHeaders(200, response.getBytes().length);
                try (OutputStream os = httpExchange.getResponseBody()) {
                    os.write(response.getBytes());
                }
            });
          	// 启动服务
            new Thread(server::start).start();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## spring boot 接入演示

配置

>  management.endpoint.${端点ID}.enabled=true/false
>
> http://{host}:{management.port}/{management.endpoints.web.base-path}/{endpointId}
>
> 以下配置：http://localhost:10091/management/actuator/prometheus

```yaml
server:
  port: 9091
management:
  server:
    port: 10091
  endpoints:
    web:
      exposure:
        include: '*' # info,health,prometheus
      base-path: /management		
```

### prometheus.yml

> 启动`prometheus`
>
> 可选参数 --storage.tsdb.path=存储数据的路径，默认路径为./data 
>
> ./prometheus --config.file=prometheus.yml
>
> 访问地址：http://localhost:9090/targets

```yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
	# 这里配置需要拉取度量信息的URL路径，这里选择应用程序的prometheus端点
    metrics_path: /management/prometheus
    static_configs:
	# 这里配置host和port
      - targets: ['localhost:10091']
```

