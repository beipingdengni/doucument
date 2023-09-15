# Feign 配置使用

## spring boot 配置

```yaml
feign:
  client:
    config:
      default:
      	# 指定Feign客户端连接提供者的超时时限
        connectTimeout: 5000
        # 指定Feign客户端连接上提供者后，向提供者进行提交请求，从提交时刻开始，到接收到响应，这个时段的超时时限
        readTimeout: 5000
  # 开启Feign对Hystrix的支持
  hystrix:
    enabled: true
```

## openfeign 会使用到的包

> 版本：10.7.0
>
> io.github.openfeign, feign-core
> io.github.openfeign, feign-hystrix
> io.github.openfeign, feign-ribbon
> io.github.openfeign, feign-jaxrs
> io.github.openfeign, feign-okhttp
> io.github.openfeign, feign-jackson
> io.github.openfeign, feign-slf4j

#### spring cloud 配合使用

> 使用一下版本分析
>
> org.springframework.cloud,spring-cloud-netflix-core:1.4.5

```
启动注册配置类：FeignClientsRegistrar    关注方法：registerFeignClients
		使用了启动导入：ImportBeanDefinitionRegistrar
		使用类扫描：ClassPathScanningCandidateComponentProvider

启动创建代理类：FeignClientFactoryBean 继承：FactoryBean<Object> 获取实例：getObject 

ribbon关联引用：LoadBalancerFeignClient

Hystrix使用实现类：HystrixTargeter
		此方法：HystrixTargeter#target 最后还是使用了：ReflectiveFeign#newInstance 创建代理

所有代理方法拦截器接口都实现：InvocationHandler    默认使用：FeignInvocationHandler
		此处还使用了方法拦截器器：MethodHandler   默认使用：SynchronousMethodHandler

ribbon服务重试使用：RequestSpecificRetryHandler   
			默认实现：DefaultLoadBalancerRetryHandler
```

# 基础配置（feigh、ribbon、hysteria）

### 使用Ribbon或者Feigin的时候，是可以开启超时重试功能的

```
# 开启熔断
feign.hystrix.enabled=true
#Hystrix超时时间（默认1000ms，单位：ms）
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=95000
// 配置hystrix默认线程池
hystrix:
  threadpool:
    default:
      coreSize: 50
      maxQueueSize: 100
      queueSizeRejectionThreshold: 100

开启的配置如下（另外ribbon超时时间和断路器超时时间也需要配置）
spring.cloud.loadbalancer.retry.enabled=true

ribbon.ReadTimeout=90000
ribbon.ConnectTimeout=10000
# 同一实例最大重试次数，不包括首次调用
ribbon.MaxAutoRetries=1
# 重试其他实例的最大重试次数，不包括首次所选的server
ribbon.MaxAutoRetriesNextServer= 2
# 是否所有操作都进行重试
ribbon.OkToRetryOnAllOperations=true


当我们需要关闭重试功能的时候，是不是spring.cloud.loadbalancer.retry.enabled=false就可以了呢，并不是。
需要把ribbon.OkToRetryOnAllOperations=false才行

```

#### 代码分解

```java
spring-cloud-netflix-core 实现了大部分的feigin和ribbon接口，整合到spring里
// feign
io.github.openfeign:feign-core:9.7.0
包含如下
拦截器方法，实现接口：RequestInterceptor
请求方法模版构建：RequestTemplate
请求实现：Client 【使用，okhttp 等】 配合ribbon使用实现类：LoadBalancerFeignClient

实现了方法反射调用
SynchronousMethodHandler#invoke 
	调用：Request request = targetRequest(template);构建一个请求头
			1、添加拦截器
			2、target.apply(new RequestTemplate(template))
	调用：response = client.execute(request, options);

ribbon 
调用类请求url：AbstractLoadBalancerAwareClient#executeWithLoadBalancer
// 负载均衡选择器
jar：spring-cloud-commons :  服务选择器 ServiceInstanceChooser


执行逻辑： feignh-core.jar [9.7.0]
核心启动
Feign 
	newInstance(Target<T> target)		
	// 构建
	Feign build()
			// 方法拦截初始化【MethodHandler】
			SynchronousMethodHandler.Factory synchronousMethodHandlerFactory =
          new SynchronousMethodHandler.Factory(client, retryer, requestInterceptors, logger,
                                               logLevel, decode404, closeAfterDecode);
			// 方法解析类 【apply 方法中，把method和MethodHandler】
      ParseHandlersByName handlersByName =
          new ParseHandlersByName(contract, options, encoder, decoder, queryMapEncoder,
                                  errorDecoder, synchronousMethodHandlerFactory);
      // 构建好代理反射类
      return new ReflectiveFeign(handlersByName, invocationHandlerFactory, queryMapEncoder);
	
// 实现feign 创建对象
ReflectiveFeign 继承 Feign
	// 获取拦截器工厂
	InvocationHandlerFactory factory
	// 创建代理类，返回代理对象
	newInstance(Target<T> target)
		/**
		* 这个地方，初始化准备工作
		*/
		// 拦截工厂获取，存放对于方法拦截器MethodHandler
		InvocationHandler handler = factory.create(target, methodToHandler);
		// 开启代理
		T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), new Class<?>[]{target.type()}, handler);
		
// 拦截器 invoke
FeignInvocationHandler 实现接口 InvocationHandler
	// 存放方法关联的，方法处理器
	private final Map<Method, MethodHandler> dispatch;
	// 调用方法
	invoke(Object proxy, Method method, Object[] args)
			// 获取方法处理逻辑
			dispatch.get(method).invoke(args)

// 方法拦截器实现类
SynchronousMethodHandler implements MethodHandler
	// 分布
	invoke(Object[] argv)
		/**
		* 获取来自：		
		*		ParseHandlersByName 类
		*			 Map<String, MethodHandler> apply(Target key) 方法			 		
		*					// 这个方案获取【方法上面的注解（如（@RequestLine ））】
		*					//  feign包下面：Contract 类就是使用解析方法上面注解
		*					List<MethodMetadata> metadata = contract.parseAndValidatateMetadata(key.type());
		*			 		//创建类 
		*					new BuildFormEncodedTemplateFromArgs(md, encoder, queryMapEncoder);		
		*/
		RequestTemplate template = buildTemplateFromArgs.create(argv);
		以下两个方法放在：while(true)里面
		//调用-执行
		executeAndDecode(template)
    // 执行调用异常，进行重试【这个是默认的】
    // 初始化参数：间隔100，延时 1毫秒 尝试五次
    retryer.continueOrPropagate(e);
      

	// 执行方法，调用
	executeAndDecode(RequestTemplate template)
		// 获取请求对象
		Request request = targetRequest(template);
			// 拦截器使用
			interceptor.apply(template);
		// 执行请求
		response = client.execute(request, options);
		// 返回值封装
		response.toBuilder().request(request).build();
		// 返回反序列化的对象
		Object result = decode(response);

feign中定义的client请求实现处理
接口：client 
实现类：默认Client.Default
jar包：feign-okhttp  						 实现类：OkHttpClient
jar包：spring-cloud-netflix-core 实现类：LoadBalancerFeignClient


```

#  以下一套结合ribbon和spring一起使用

```java
jar包：spring-cloud-netflix-core 配置使用spring
	DefaultFeignLoadBalancedConfiguration 默认
	OkHttpFeignLoadBalancedConfiguration  使用OkHttp
	HttpClientFeignLoadBalancedConfiguration 使用httpclient

以上几个类配置：
	// ServiceInstanceChooser 选择服务器接口类 
	首先要实现类：AbstractLoadBalancingClient extends AbstractLoadBalancerAwareClient implements ServiceInstanceChooser

// 负载平衡
LoadBalancerFeignClient
	// 缓存和重试
	private CachingSpringLoadBalancerFactory lbClientFactory;
	execute
		// 构建创建ribbon 请求头
		FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);
		// 获取配置信息
		IClientConfig requestConfig = getClientConfig(options, clientName);
		// 返回结果
		lbClient(clientName).executeWithLoadBalancer(ribbonRequest,requestConfig).toResponse()
		// 创建ClientFactory
		lbClient(String clientName)
			lbClientFactory.create(clientName)
			
// 承接上面类方法：executeWithLoadBalancer 实现
AbstractLoadBalancerAwareClient 类
	// 选择服务执行调用
	executeWithLoadBalancer
		// 使用构建，配置--环境变量
		LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);

LoadBalancerCommand类
	Observable<Server> selectServer()
  	// 获取loadBalancerContext 来自： AbstractLoadBalancerAwareClient 继承的类 
  	// 最终使用：·  
  	// 调用方法使用：LoadBalancerContext类
  	Server server = loadBalancerContext.getServerFromLoadBalancer(loadBalancerURI, loadBalancerKey)

LoadBalancerContext 
		getServerFromLoadBalancer
				// 获取负载平衡器 ，如下继承关系 
				//  AbstractLoadBalancer implements ILoadBalancer
				//  BaseLoadBalancer extends AbstractLoadBalancer
				// 这个是用到eureka，实现接口：ZoneAwareLoadBalancer extends DynamicServerListLoadBalancer extends BaseLoadBalancer
				ILoadBalancer lb = getLoadBalancer();
				//执行选择服务
				Server svc = lb.chooseServer(loadBalancerKey);
						// 方法中调用
						// 获取注册服务名，随机选取一个可用的
						 String zone = ZoneAvoidanceRule.randomChooseZone(zoneSnapshot, availableZones);
						// 获取均衡器
						BaseLoadBalancer zoneLoadBalancer = getLoadBalancer(zone);
							 // 方法中调用 
							 IRule rule = cloneRule(this.getRule())
               // 构建均衡器返回
               loadBalancer = new BaseLoadBalancer(this.getName() + "_" + zone, rule, this.getLoadBalancerStats());
						server = zoneLoadBalancer.chooseServer(key);

// 执行方法调用  
AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig)  
// 父类是：FeignLoadBalancer extends AbstractLoadBalancerAwareClient
类：RetryableFeignLoadBalancer
			// 执行请求
			execute(final RibbonRequest request, IClientConfig configOverride)
  		// 选择服务
  		choose(String serviceId)
  				new RibbonLoadBalancerClient.RibbonServer(serviceId,                                                 							this.getLoadBalancer().chooseServer(serviceId));
  		

java.net.NoRouteToHostException: No route to host (Host unreachable)
Request processing failed; nested exception is feign.RetryableException: No route to host (Host unreachable) 
  
  

重试类：DefaultLoadBalancerRetryHandler
重试配置参数：
// 相同服务尝试次数，默认0
this.retrySameServer = clientConfig.get(CommonClientConfigKey.MaxAutoRetries, DefaultClientConfigImpl.DEFAULT_MAX_AUTO_RETRIES);
// 尝试重试服务，默认1
this.retryNextServer = clientConfig.get(CommonClientConfigKey.MaxAutoRetriesNextServer, DefaultClientConfigImpl.DEFAULT_MAX_AUTO_RETRIES_NEXT_SERVER);
// 是否开启重试
this.retryEnabled = clientConfig.get(CommonClientConfigKey.OkToRetryOnAllOperations, false);
```

##### AbstractLoadBalancerAwareClient#executeWithLoadBalancer

> command.submit  重点关注问题

```java
// 摘取一段代码
public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) {
  // 构建请求
  LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);
  // 使用javarx 流处理【重点关注】
   return command.submit(
                new ServerOperation<T>() {
                    @Override
                    public Observable<T> call(Server server) {
                        URI finalUri = reconstructURIWithServer(server, request.getUri());
                        S requestForServer = (S) request.replaceUri(finalUri);
                        try {
                          // 使用请求
                            return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
                        } 
                        catch (Exception e) {
                            return Observable.error(e);
                        }
                    }
                })
                .toBlocking()
                .single();
}

protected LoadBalancerCommand<T> buildLoadBalancerCommand(final S request, final IClientConfig config) {
  	// 获取重试类【控制重试】
		RequestSpecificRetryHandler handler = getRequestSpecificRetryHandler(request, config);
		LoadBalancerCommand.Builder<T> builder = LoadBalancerCommand.<T>builder()
      	// this 是继承 AbstractLoadBalancerAwareClient
				.withLoadBalancerContext(this)
      	// 添加重试类
				.withRetryHandler(handler)
      // 最终使用到 selectServer() 方法中，loadBalancerURI
				.withLoadBalancerURI(request.getUri());
  	// 类AbstractLoadBalancingClient 中实现了方法 ，通过key定位服务
  	// 最终使用到 selectServer() 方法中，loadBalancerKey
		customizeLoadBalancerCommandBuilder(request, config, builder);
		return builder.build();
	}

```



#### 使用spring feign 配置使用

```java
public interface FeignProxyFactory {
    <T> T create(Class<T> proxyType, String baseUrl);
}

/**
* 测试
*/
public class KryFeignJaxrsProxyFactory implements FeignProxyFactory {

    @Autowired(required = false)
    private ObjectMapper objectMapper;


    @Override
    public <T> T create(Class<T> proxyType, String baseUrl) {

        T instance = Feign.builder()
                .encoder(null == objectMapper ? new JacksonEncoder() : new JacksonEncoder(objectMapper))
                .decoder(null == objectMapper ? new JacksonDecoder() : new JacksonDecoder(objectMapper))
                .logger(new Slf4jLogger())
                .logLevel(Logger.Level.FULL)
                .contract(new JAXRSContract())
                .requestInterceptor(new KryHeaderRequestInterceptor())
                .target(proxyType, baseUrl);
        return instance;
    }
}

```

#### spring-bean xml 使用

```xml
<bean id="kryFeignJaxrsProxyFactory" class="com.calm.b.common.KryFeignJaxrsProxyFactory"/>

    <bean id="goodServiceClient" factory-bean="kryFeignJaxrsProxyFactory" factory-method="create">
        <constructor-arg value="com.calm.thirdapi.good.service.GoodServiceClient"/>
        <constructor-arg value="${order.good.server.url}"/>
    </bean>
```

