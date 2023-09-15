# hystrix

##  熔断、容错、隔离

Hystrix 旨在执行以下操作：

- 保护和控制通过第三方客户端库访问（通常通过网络）依赖项造成的延迟和故障。
- 阻止复杂分布式系统中的级联故障。
- 快速失败并快速恢复。
- 如果可能的话，回退并优雅地降级。
- 实现近乎实时的监控、警报和操作控制。

## 工作流程

<img src="https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/circuit-breaker-1280.png" alt="img" style="zoom:50%;" />

## pom xml

```xml
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>x.y.z</version>
</dependency>
```

## 配置

#### 官方网站

> https://github.com/Netflix/Hystrix/wiki/Configuration#CommandProperties

```yaml
hystrix:
  command:
    default:  #default全局有效，service id指定应用有效
      execution:
        timeout:
          #如果enabled设置为false，则请求超时交给ribbon控制,为true,则超时作为熔断根据
					#设置的超时时间应该满足：(1 + MaxAutoRetries) * （1 + MaxAutoRetriesNextServer） ReadTimeOut < hystrix 的 timeoutInMilliseconds*
          enabled: true
        isolation:
          thread:
            timeoutInMilliseconds: 10000 #断路器超时时间，默认1000ms
```

![image-20230915212209163](/Users/tbp/Documents/practice/doucument/java/middleware/img/hystrix/image-20230915212209163.png)

## 缓存（WEB）

```java
HystrixRequestContext context = HystrixRequestContext.initializeContext();
context.shutdown();

// 查看是否位缓存
CommandHelloWorld helloWord = new CommandHelloWorld("Bob")
String s =helloWord.execute();
// 判断，返回值是否位缓存
boolen  isCache = helloWord.isResponseFromCache()


public class HystrixRequestContextServletFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
     throws IOException, ServletException {
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        try {
            chain.doFilter(request, response);
        } finally {
            context.shutdown();
        }
    }
}
```



## 值检测

```
// 是否，失败执行
command.isFailedExecution()
// 是否响应，从返回值
command.isResponseFromFallback()
```

# 图像监控（Hystrix Dashboard）

```yaml
# 开启actuator的hystrix.stream监控终端
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

暴露的健康地址： http://localhost:8080/actuator/hystrix.stream

<img src="/Users/tbp/Documents/practice/doucument/java/middleware/img/hystrix/image-20230915221705797.png" alt="image-20230915221705797" style="zoom:50%;" />

## 监控图介绍

![在这里插入图片描述](/Users/tbp/Documents/practice/doucument/java/middleware/img/hystrix/hystrix-dashboard.png)

## Turbine （聚合监控）

### pom xml

```xml
<!-- 单个 -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<!-- 集群 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-turbine</artifactId>
 </dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-netflix-turbine</artifactId>
 </dependency>
<!-- 页面 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
<!-- 监控 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!--eureka客户端依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 基础配置

```properties
spring.application.name=hystrix-dashboard-turbine
server.port=8001
#配置Eureka中的serviceId列表，表明监控哪些服务
turbine.appConfig=node01,node02 # 指定集群组名称
#指定聚合哪些集群，多个使用”,”分割，默认为default。可使用http://.../turbine.stream?cluster={clusterConfig之一}访问
turbine.aggregator.clusterConfig= default 
turbine.clusterNameExpression= new String("default")
# 1. clusterNameExpression指定集群名称，默认表达式appName；
# 	此时：turbine.aggregator.clusterConfig需要配置想要监控的应用名称；
# 2. 当clusterNameExpression: default时，turbine.aggregator.clusterConfig可以不写，因为默认就是default；
# 3. 当clusterNameExpression: metadata[‘cluster’]时，假设想要监控的应用配置了
#			eureka.instance.metadata-map.cluster: ABC，则需要配置，同时turbine.aggregator.clusterConfig: ABC

# eureka server 集群地址
eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
# 区分集群，使用案列
# 指定要监控的微服务组
turbine.aggregator.clusterConfig= group1,group2
# 指定要监控微服务组名称来自于Eureka元数据cluster
turbine.clusterNameExpression= metadata['cluster'] 
# http://turbine-hostname:port/turbine.stream?cluster=group1
```

每个微服务启动配置

```properties
# eureka server 集群地址
eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
# 指定集群
eureka.instance.metadata-map.cluster =group1   # 自定义Eureka元数据
# 每个服务,开启Feign对Hystrix的支持
feign.hystrix.enabled=true
# 指定Feign客户端连接提供者的超时时限
feign.client.config.default.connectTimeout=5000
# 指定Feign客户端连接上提供者后，向提供者进行提交请求，从提交时刻开始，到接收到响应，这个时段的超时时限
feign.client.config.default.readTimeout=5000
# 隔离开启
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=3000
# 开启actuator的hystrix.stream监控终端
management.endpoints.web.exposure.include=hystrix.stream

# @EnableFeignClients  // 开启Feign客户端

```



### java 启动类注解

#### 访问地址

> Cluster via Turbine (default cluster): http://turbine-hostname:port/turbine.stream
> Cluster via Turbine (custom cluster): http://turbine-hostname:port/turbine.stream?cluster=[clusterName]
> Single Hystrix App: http://hystrix-app:port/hystrix.stream

```java
@SpringBootApplication
@EnableHystrixDashboard  // 单个监控，只需求启动
@EnableTurbine
public class DashboardApplication {
	public static void main(String[] args) {
		SpringApplication.run(DashboardApplication.class, args);
	}
}
```



# demo 使用

### Java 代码

#### 构造线程（自定义参数设置）

```java
public class CommandHelloWorld extends HystrixCommand<String> {

  // 自定义配置
  private static final Setter cachedSetter = 
        Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
            .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld")
						//.andCommandPropertiesDefaults(   // 自定义
            //            HystrixCommandProperties.Setter()
            //                	 .withExecutionTimeoutInMilliseconds(600)	 // 隔离级别时间
            //                   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)) //
						.andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("HelloWorldPool")));    
  
 
    private final String name;

    public CommandHelloWorld(String name) {
        super(cachedSetter);
        // super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }
		// 执行
    @Override
    protected String run() {
        return "Hello " + name + "!";
    }
  	// 熔断， 返回结果
    @Override
    protected String getFallback() {
        return "Hello Failure " + name + "!";
    }
   // 失败， 会退
  	@Override
    protected Observable<Boolean> resumeWithFallback() {
        return Observable.just( true );
    }
  	// 缓存
    @Override
    protected String getCacheKey() {
        return String.valueOf(value);
    }
}
```

#### 构造

```java
public class CommandHelloWorld extends HystrixObservableCommand<String> {

    private final String name;

    public CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected Observable<String> construct() {
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> observer) {
                try {
                    if (!observer.isUnsubscribed()) {
                        // a real example would do work like a network call here
                        observer.onNext("Hello");
                        observer.onNext(name + "!");
                        observer.onCompleted();
                    }
                } catch (Exception e) {
                    observer.onError(e);
                }
            }
         } ).subscribeOn(Schedulers.io());
    }
}
```



## 返回结果处理

```java
String s = new CommandHelloWorld("Bob").execute();
Future<String> s = new CommandHelloWorld("Bob").queue();
Observable<String> s = new CommandHelloWorld("Bob").observe(); //hot observable
Observable<String> s = new CommandHelloWorld("Bob").toObservable(); //cold observable
```

![image-20230915213100393](/Users/tbp/Documents/practice/doucument/java/middleware/img/hystrix/image-20230915213100393.png)