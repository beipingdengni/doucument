

## 定义动态路由刷新（RouteDefinitionWriter）

```java
//动态路由服务
@Service
public class GoRouteServiceImpl implements ApplicationEventPublisherAware {
    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;
  
    private ApplicationEventPublisher publisher;
  
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher=applicationEventPublisher;
    }
    //增加路由
    public String add(RouteDefinition definition){
        routeDefinitionWriter.save(Mono.just(definition)).subscribe();
        this.publisher.publishEvent(new RefreshRoutesEvent(this));
        return "success";
    }
    //更新路由
    public String update(RouteDefinition definition){
        try {
            delete(definition.getId());
        }catch (Exception e){
            return "update fail, not find route routeId:"+definition.getId();
        }
        try {
            routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            this.publisher.publishEvent(new RefreshRoutesEvent(this));
            return "success";
        }catch (Exception e){
            return "update route fail";
        }
    }
    //删除路由
    public Mono<ResponseEntity<Object>> delete(String id){
        return this.routeDefinitionWriter.delete(Mono.just(id)).then(Mono.defer(()->{
           // 发送刷新RefreshRoutesEvent
	          this.publisher.publishEvent(new RefreshRoutesEvent(this));
            return Mono.just(ResponseEntity.ok().build());
        })).onErrorResume((t)->{
            return t instanceof NotFoundException;
        }, (t) ->{
            return Mono.just(ResponseEntity.notFound().build());
        });
    }
}
```

## 实现动态加载刷新路由策略

```java
// 继承实现以下接口
RouteDefinitionRepository
  InMemoryRouteDefinitionRepository //内存
  RedisRouteDefinitionRepository 		//redis 实现
```

## 暴露接口

```java
// 实现了controller接口
org.springframework.cloud.gateway.actuate.AbstractGatewayControllerEndpoint
```

## 动态刷新,发送RefreshRoutesEvent刷新事件

```
@Service
public class GatewayDynamicRouteService implements ApplicationEventPublisherAware {

    @Resource
    private RedisRouteDefinitionRepository redisRouteDefinitionRepository;

    private ApplicationEventPublisher applicationEventPublisher;
		// 增加路由
    public void add(RouteDefinition routeDefinition) {
        redisRouteDefinitionRepository.save(Mono.just(routeDefinition)).subscribe();
        applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this));
    }
  	// 更新
    public int update(RouteDefinition routeDefinition) {
        redisRouteDefinitionRepository.delete(Mono.just(routeDefinition.getId()));
        redisRouteDefinitionRepository.save(Mono.just(routeDefinition)).subscribe();
        applicationEventPublisher.publishEvent(new RefreshRoutesEvent(this));
        return 1;
    }
		//删除
    public Mono<ResponseEntity<Object>> delete(String id) {
        return redisRouteDefinitionRepository.delete(Mono.just(id)).then(Mono.defer(() -> Mono.just(ResponseEntity.ok().build())))
                .onErrorResume(t -> t instanceof NotFoundException, t -> Mono.just(ResponseEntity.notFound().build()));
    }
  
   @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```

