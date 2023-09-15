# ribbon

## pom xml

```
<dependency>
    <groupId>com.netflix.ribbon</groupId>
    <artifactId>ribbon</artifactId>
    <version>2.2.2</version>
</dependency>
```



## 自定义策略

```
# IRule接口实现类
CustomRule extends AbstractLoadBalancerRule
```



## 基础使用

```java
HttpResourceGroup httpResourceGroup = Ribbon.createHttpResourceGroup("movieServiceClient",
            ClientOptions.create()
                    .withMaxAutoRetriesNextServer(3)
                    .withConfigurationBasedServerList("localhost:8080,localhost:8088"));
HttpRequestTemplate<ByteBuf> recommendationsByUserIdTemplate = httpResourceGroup.newTemplateBuilder("recommendationsByUserId", ByteBuf.class)
            .withMethod("GET")
            .withUriTemplate("/users/{userId}/recommendations")
            .withFallbackProvider(new RecommendationServiceFallbackHandler())
            .withResponseValidator(new RecommendationServiceResponseValidator())
            .build();
Observable<ByteBuf> result = recommendationsByUserIdTemplate.requestBuilder()
                        .withRequestProperty("userId", "user1")
                        .build()
                        .observe();

// 自定义调用
CommandBuilder.<String>newBuilder()
        .withLoadBalancer(LoadBalancerBuilder.newBuilder().buildFixedServerListLoadBalancer(serverList))
        .build(new LoadBalancerExecutable<String>() {
            @Override
            public String run(Server server) throws Exception {
                URL url = new URL("http://" + server.getHost() + ":" + server.getPort() + path);
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                return conn.getResponseMessage();
            }
        }).execute();
```



## spring 配置使用

```yaml
#全局配置
ribbon:
  OkToRetryOnAllOperations: true #对所有操作请求都进行重试,默认false
  ReadTimeout: 1000   #负载均衡超时时间，默认值5000
  ConnectTimeout: 3000 #请求连接的超时时间，默认值2000
  # ribbon 中，请求最多会被执行—— 1 + maxAutoRetries + (maxAutoRetries + 1) * MaxAutoRetriesNextServer
  MaxAutoRetries: 1    #对当前实例的重试次数，默认0
  MaxAutoRetriesNextServer: 0 #重试切换实例的次数，默认1
  http:
    client: #apache http client
      enabled: true
  # 这里采用随机访问，要修改其它的策略，把全类名换掉即可
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
  
```

### bean配置

```
@Bean
public IRule ribbonRule(){
	return new RandomRule();
}

```



## 负载策略

```
负载均衡策略的父接口，里边的核心方法就是choose方法，用来选择一个服务实例。

AbstractLoadBalancerRule AbstractLoadBalancerRule是一个抽象类，里边主要定义了一个ILoadBalancer，这里定义它的目的主要是辅助负责均衡策略选取合适的服务端实例。

RandomRule 看名字就知道，这种负载均衡策略就是随机选择一个服务实例，看源码我们知道，在RandomRule的无参构造方法中初始化了一个Random对象，然后在它重写的choose方法又调用了choose(ILoadBalancer lb, Object key)这个重载的choose方法，在这个重载的choose方法中，每次利用 random对象生成一个不大于服务实例总数的随机数，并将该数作为下标所以获取一个服务实例。

RoundRobinRule RoundRobinRule这种负载均衡策略叫做线性轮询负载均衡策略。这个类的choose(ILoadBalancer lb, Object key)函数整体逻辑是这样的：开启一个计数器count，在while循环中遍历服务清单，获取清单之前先通过incrementAndGetModulo方法获取一个下标，这个下标是一个不断自增长的数先加1然后和服务清单总数取模之后获取到的（所以这个下标从来不会越界），拿着下标再去服务清单列表中取服务，每次循环计数器都会加1，如果连续10次都没有取到服务，则会报一个警告No available alive servers after 10 tries from load balancer: XXXX。

RetryRule（在轮询的基础上进行重试） 看名字就知道这种负载均衡策略带有重试功能。首先RetryRule中又定义了一个subRule，它的实现类是RoundRobinRule，然后在RetryRule的choose(ILoadBalancer lb, Object key)方法中，每次还是采用RoundRobinRule中的choose规则来选择一个服务实例，如果选到的实例正常就返回，如果选择的服务实例为null或者已经失效，则在失效时间deadline之前不断的进行重试（重试时获取服务的策略还是RoundRobinRule中定义的策略），如果超过了deadline还是没取到则会返回一个null。

WeightedResponseTimeRule（权重 —nacos的NacosRule ，Nacos还扩展了一个自己的基于配置的权重扩展） WeightedResponseTimeRule是RoundRobinRule的一个子类，在WeightedResponseTimeRule中对RoundRobinRule的功能进行了扩展， WeightedResponseTimeRule中会根据每一个实例的运行情况来给计算出该实例的一个权重，然后在挑选实例的时候则根据权重进行挑选，这样能够实现更优的实例调用。 WeightedResponseTimeRule中有一个名叫DynamicServerWeightTask的定时任务，默认情况下每隔30秒会计算一次各个服务实例的权重，权重的计算规则也很简单，如果一个服务的平均响应时间越短则权重越大，那么该服务实例被选中执行任务的概率也就越大。

ClientConfigEnabledRoundRobinRule ClientConfigEnabledRoundRobinRule选择策略的实现很简单，内部定义了RoundRobinRule，choose方法还是采用了RoundRobinRule的 choose方法，所以它的选择策略和RoundRobinRule的选择策略一致，不赘述。

BestAvailableRule BestAvailableRule继承自ClientConfigEnabledRoundRobinRule，它在ClientConfigEnabledRoundRobinRule的基础上主要增加了根据 loadBalancerStats中保存的服务实例的状态信息来过滤掉失效的服务实例的功能，然后顺便找出并发请求最小的服务实例来使用。然而loadBalancerStats有可能为null，如果loadBalancerStats为null，则BestAvailableRule将采用它的父类即 ClientConfigEnabledRoundRobinRule的服务选取策略（线性轮询）。

ZoneAvoidanceRule （默认规则，复合判断server所在区域的性能和server的可用性选择服务器。） ZoneAvoidanceRule是PredicateBasedRule的一个实现类，只不过这里多一个过滤条件，ZoneAvoidanceRule中的过滤条件是以 ZoneAvoidancePredicate为主过滤条件和以 AvailabilityPredicate为次过滤条件组成的一个叫做CompositePredicate的组合过滤条件，过滤成功之后，继续采用线性轮询 (RoundRobinRule)的方式从过滤结果中选择一个出来。

AvailabilityFilteringRule（先过滤掉故障实例，再选择并发较小的实例） 过滤掉一直连接失败的被标记为circuit tripped的后端Server，并过滤掉那些高并发的后端Server或者使用一个AvailabilityPredicate来包含过滤server的逻辑，其实就是检查status里记录的各个Server的运行状态
```

