

```xml
<dependency>
  <groupId>io.dropwizard.metrics</groupId>
  <artifactId>metrics-core</artifactId>
  <version>4.2.0</version>
</dependency

<!-- spring metrics -->
<dependency>
    <groupId>com.ryantenney.metrics</groupId>
    <artifactId>metrics-spring</artifactId>
    <version>3.1.3</version>
</dependency>
```



> Metrics是一个提供服务性能检测工具的Java类库，它提供了功能强大的性能指标工具库用于度量生产环境中的各关键组件性能。

度量类型[#](#度量类型)
--------------

Metrics提供了以下几种基本的度量类型：

*   `Gauge`：用于提供自定义度量。
*   `Counter`：计数器，本质是一个`java.util.concurrent.atomic.LongAdder`。
*   `Histogram`：直方图数据。
*   `Meter`：统计系统中某一事件的响应速率，如TPS、QPS。该项指标值直接反应系统当前的处理能力
*   `Timer`：计时器，是`Meter`和`Histogram`的结合，可以统计接口请求速率和响应时长。

### Gauge[#](#gauge)

`Gauge`是对一项值的瞬时度量。我们可以通过实现`Gauge`接口来根据业务场景自定义度量。

例如，想要度量队列中处于等待状态的作业数量：

```Java
public class QueueManager {
    private final Queue queue;

    public QueueManager(MetricRegistry metrics, String name) {
        this.queue = new Queue();
        // 通过MetricRegistry 的register方法注册Gauge度量
        metrics.register(MetricRegistry.name(QueueManager.class, name, "size"),
                         new Gauge<Integer>() {
                             @Override
                             public Integer getValue() {
                                 return queue.size();
                             }
                         });
    }
}
```

官方目前提供了以下几种`Gauge`实现：

![](https://files.mdnice.com/user/23626/982b6e6a-5cf5-4c77-aa18-2d83a5e87ed2.png)

### Counter[#](#counter)

`Counter`是一个常规计数器，用于对某项指标值进行累加或者递减操作。

`Counter`本质是一个`java.util.concurrent.atomic.LongAdder`，在多线程同时更新计数器的场景下，当并发量较大时，`LongAdder`比`AtomicLong`具有更高的吞吐量，当然空间资源消耗也更大一些。

```Java
final Counter evictions = registry.counter(name(SessionStore.class, "cache-evictions"));
evictions.inc();
evictions.inc(3);
evictions.dec();
evictions.dec(2);
```

### Histograms[#](#histograms)

`Histogram`反应的是数据流中的值的分布情况。包含最小值、最大值、平均值、中位数、p75、p90、p95、p98、p99以及p999数据分布情况。

```Java
private final Histogram responseSizes = metrics.histogram(name(RequestHandler.class, "response-sizes"));

public void handleRequest(Request request, Response response) {
    // etc
    responseSizes.update(response.getContent().length);
}
```

`Histogram`计算分位数的方法是先对整个数据集进行排序，然后取排序后的数据集中特定位置的值（比如p99就是取倒序1%位置的值）。这种方式适合于小数据集或者批处理系统，不适用于要求高吞吐量、低延时的服务。

对于数据量较大，系统对吞吐量、时延要求较大的场景，我们可以采用抽样的方式获取数据。通过动态地抽取程序运行过程中的能够代表系统真实运行情况的一小部分数据来实现对整个系统运行指标的近似度量，这种方法叫做蓄水池算法（**reservoir sampling**）。

Metrics中提供了各式各样的`Reservoir`实现：

![](https://files.mdnice.com/user/23626/31d81c0f-f58f-4dd1-b8aa-0f890130d5f0.png)

### Meter[#](#meter)

`Meter`用于度量事件响应的平均速率，它表示的是应用程序整个运行生命周期内的总速率（总请求响应量/处理请求的总毫秒数，即每秒请求数）。

除此之外，`Meter`还提供了1分钟、5分钟以及15分钟的动态平均响应速率。

```Java
final Meter getRequests = registry.meter(name(WebProxy.class, "get-requests", "requests"));
getRequests.mark();
getRequests.mark(requests.size());
```

### Timer[#](#timer)

`Timer`会度量服务的响应速率，同时也会统计服务响应时长的分布情况。

```Java
final Timer timer = registry.timer(name(WebProxy.class, "get-requests"));

final Timer.Context context = timer.time();
try {
    // handle request
} finally {
    context.stop();
}
```

* * *

Reporters[#](#reporters)
------------------------

通过上述各项度量监测服务指标后，我们可以通过Reporters报表导出度量结果。`metrics-core`模块中实现了以下几种导出指标的Report：

![](https://files.mdnice.com/user/23626/085acc5e-17ab-428e-9d97-a2ada7f0003d.png)

### Console Reporters[#](#console-reporters)

定时向控制台发送服务的各项指标数据。

```Java
final ConsoleReporter reporter = ConsoleReporter.forRegistry(registry)
                                                .convertRatesTo(TimeUnit.SECONDS)
                                                .convertDurationsTo(TimeUnit.MILLISECONDS)
                                                .build();
reporter.start(1, TimeUnit.MINUTES);
```

### CsvReporter[#](#csvreporter)

定时向给定目录下的`.csv`文件追加服务各项指标数据。对于每一项指标都会在指定目录下创建一个`.csv`文件，然后定时（本例中是1s）向每个文件中追加指标最新数据。

```Java
final CsvReporter reporter = CsvReporter.forRegistry(registry)
                                        .formatFor(Locale.US)
                                        .convertRatesTo(TimeUnit.SECONDS)
                                        .convertDurationsTo(TimeUnit.MILLISECONDS)
                                        .build(new File("~/projects/data/"));
reporter.start(1, TimeUnit.SECONDS);
```

### JmxReporter[#](#jmxreporter)

将服务的各项度量指标通过JMX MBeans暴露出来，之后可以使用VisualVM查看指标数据。**生产环境不建议使用。**

```Java
final JmxReporter reporter = JmxReporter.forRegistry(registry).build();
reporter.start();
```

### Slf4jReporter[#](#slf4jreporter)

`Slf4jReporter`允许我们将服务的指标数据作为日志记录到日志文件中。

```Java
final Slf4jReporter reporter = Slf4jReporter.forRegistry(registry)
                                            .outputTo(LoggerFactory.getLogger("com.example.metrics"))
                                            .convertRatesTo(TimeUnit.SECONDS)
                                            .convertDurationsTo(TimeUnit.MILLISECONDS)
                                            .build();
reporter.start(1, TimeUnit.MINUTES);
```

* * *

如何使用[#](#如何使用)
--------------

### 直接引用[#](#直接引用)

直接依赖Metrics的核心库，通过其提供的各类API完成服务指标数据度量。

1.  引入Maven依赖

```XML
<dependency>
  <groupId>io.dropwizard.metrics</groupId>
  <artifactId>metrics-core</artifactId>
  <version>${metrics.version}</version>
</dependency>
```

1.  创建一个`MetricRegistry`对象，它是Metrics类库的核心类，所有的应用指标都需要注册到`MetricRegistry`。

```Java
// 实例化MetricsRegistry
final MetricRegistry metrics = new MetricRegistry();

// 开启Console Reporter
startConsoleReporter();

Meter requests = metrics.meter(name(MetricsConfig.class, "requests", "size"));
requests.mark();

void startReport() {
    ConsoleReporter reporter = ConsoleReporter.forRegistry(metrics)
            .convertRatesTo(TimeUnit.SECONDS)
            .convertDurationsTo(TimeUnit.MILLISECONDS)
            .build();
    reporter.start(1, TimeUnit.SECONDS);
}


```

### 整合Spring[#](#整合spring)

1.  引入Maven依赖

```XML
<dependency>
    <groupId>com.ryantenney.metrics</groupId>
    <artifactId>metrics-spring</artifactId>
    <version>3.1.3</version>
</dependency>
```

1.  通过Java注解配置Metrics。

```Java
import java.util.concurrent.TimeUnit;
import org.springframework.context.annotation.Configuration;
import com.codahale.metrics.ConsoleReporter;
import com.codahale.metrics.MetricRegistry;
import com.codahale.metrics.SharedMetricRegistries;
import com.ryantenney.metrics.spring.config.annotation.EnableMetrics;
import com.ryantenney.metrics.spring.config.annotation.MetricsConfigurerAdapter;

@Configuration
@EnableMetrics
public class SpringConfiguringClass extends MetricsConfigurerAdapter {

    @Override
    public void configureReporters(MetricRegistry metricRegistry) {
        // registerReporter allows the MetricsConfigurerAdapter to
        // shut down the reporter when the Spring context is closed
        registerReporter(ConsoleReporter
            .forRegistry(metricRegistry)
            .build())
            .start(1, TimeUnit.MINUTES);
    }

}


@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({DelegatingMetricsConfiguration.class})
public @interface EnableMetrics {
    // 默认false表示使用JDK动态代理，设置为true时表示使用CGLIB动态代理（当使用基于类的服务暴露方式时）
    boolean exposeProxy() default false;
    // 设置为true时，目标对象可以通过AopContext.currentProxy()访问封装它的代理
    boolean proxyTargetClass() default false;
}

```

#### 使用限制[#](#使用限制)

因为Spring AOP中，只有声明为`public`的方法可以被代理，所以`@Timed`, `@Metered`, `@ExceptionMetered`以及 `@Counted`在`non-public`无法生效。

因为`@Gauge`注解不涉及代理，所以它可以被应用在`non-public`属性和方法上。

```Java
public class MetricsBeanPostProcessorFactory {
    private MetricsBeanPostProcessorFactory() {
    }

    public static AdvisingBeanPostProcessor exceptionMetered(MetricRegistry metricRegistry, ProxyConfig proxyConfig) {
        return new AdvisingBeanPostProcessor(ExceptionMeteredMethodInterceptor.POINTCUT, ExceptionMeteredMethodInterceptor.adviceFactory(metricRegistry), proxyConfig);
    }

    public static AdvisingBeanPostProcessor metered(MetricRegistry metricRegistry, ProxyConfig proxyConfig) {
        return new AdvisingBeanPostProcessor(MeteredMethodInterceptor.POINTCUT, MeteredMethodInterceptor.adviceFactory(metricRegistry), proxyConfig);
    }

    public static AdvisingBeanPostProcessor timed(MetricRegistry metricRegistry, ProxyConfig proxyConfig) {
        return new AdvisingBeanPostProcessor(TimedMethodInterceptor.POINTCUT, TimedMethodInterceptor.adviceFactory(metricRegistry), proxyConfig);
    }

    public static AdvisingBeanPostProcessor counted(MetricRegistry metricRegistry, ProxyConfig proxyConfig) {
        return new AdvisingBeanPostProcessor(CountedMethodInterceptor.POINTCUT, CountedMethodInterceptor.adviceFactory(metricRegistry), proxyConfig);
    }

    public static GaugeFieldAnnotationBeanPostProcessor gaugeField(MetricRegistry metricRegistry) {
        return new GaugeFieldAnnotationBeanPostProcessor(metricRegistry);
    }

    public static GaugeMethodAnnotationBeanPostProcessor gaugeMethod(MetricRegistry metricRegistry) {
        return new GaugeMethodAnnotationBeanPostProcessor(metricRegistry);
    }

    public static CachedGaugeAnnotationBeanPostProcessor cachedGauge(MetricRegistry metricRegistry) {
        return new CachedGaugeAnnotationBeanPostProcessor(metricRegistry);
    }

    public static MetricAnnotationBeanPostProcessor metric(MetricRegistry metricRegistry) {
        return new MetricAnnotationBeanPostProcessor(metricRegistry);
    }

    public static HealthCheckBeanPostProcessor healthCheck(HealthCheckRegistry healthRegistry) {
        return new HealthCheckBeanPostProcessor(healthRegistry);
    }
}
```

除此此外，在一个方法中调用处于同一个类中的另一个带有Metrics注解的方法时，方法执行流程不会经过代理。