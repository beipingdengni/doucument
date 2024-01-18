## 多线程异步支持

```xml
<!-- 任务执行器 --> 
<task:executor id="executor" pool-size="10" />
<!-- 名字必须是applicationEventMulticaster，因为AbstractApplicationContext默认找个 --> 
<bean id="applicationEventMulticaster" class="org.springframework.context.event.SimpleApplicationEventMulticaster"> 
　　<!-- 注入任务执行器 这样就实现了异步调用 -->
　　<property name="taskExecutor" ref="executor"/> 
</bean>
```

### 可以直接在监听类上面增加@Aync

> Spring提供了@Aync注解来完成异步调用，我们在监听事件处理的onApplicationEvent方法上加上@Aync注解即可
> 这个注解用于标注某个方法或某个类里面的所有方法都是需要异步处理的。被注解的方法被调用的时候，会在新线程中执行，而调用它的方法会在原来的线程中执行。这样可以避免阻塞、以及保证任务的实时性
>
> @Async(value = "taskExecutor")

```xml
<!-- 开启@AspectJ AOP代理  -->  
<aop:aspectj-autoproxy proxy-target-class="true"/> 

<!-- @EnableAsync开启异步执行配置  -->  
<task:executor id="executor" pool-size="10" />
<task:annotation-driven executor="executor" /> 
```

```java
@Configuration
public class TaskPoolConfig {
    @Bean(name = "asyncExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("asyncExecutor-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        return executor;
    }
   	@Bean
		public SimpleApplicationEventMulticaster applicationEventMulticaster() {
			SimpleApplicationEventMulticaster simpleAppEventMulti = new SimpleApplicationEventMulticaster();
			simpleAppEventMulti.setTaskExecutor(taskExecutor());
			return simpleAppEventMulti;
		}
}

@EventListener
@Async("asyncExecutor")
public void listener(TestEvent event) throws InterruptedException {
  Thread.sleep(5000);
  log.info("监听到数据：{}", event.getMessage());
}
```



## 定义事件

### 普通类

```java
public class OrderCreatedEvent extends ApplicationEvent {
    private Order order;
 
    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }
 
    public Order getOrder() {
        return order;
    }
}
```

### 范型类

```java
@Data
class BaseEvent<T> implements ResolvableTypeProvider {
    private T data;
    private String addOrUpdate;
  
    public BaseEvent(T data, String addOrUpdate) {
        this.data = data;
        this.addOrUpdate = addOrUpdate;
    }
  
    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getData()));
    }
}

@Component
public class EventListenerService{
	@EventListener
	public void handlePersonEvent(BaseEvent<Person>personEvent){
		Log.info("监听到PersonEvent:{)",personEvent);
	}
  
	@EventListener
	public void handleorderEvent(BaseEvent<Order>orderEvent){
		log.info("监听到orderEvent:{)",orderEvent);
	}
}
// 日志打印
//监听到PersonEvent:BaseEvent(data=Person(name=why),addorUpdate=add)
//监听到orderEvent:BaseEvent(data=Order(orderName=why),addorUpdate=update)
```



## 发布事件

```java
@Service
public class OrderService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;
 
    public void createOrder(Order order) {
        // 保存订单
        // ...
 
        // 触发订单创建事件
        OrderCreatedEvent event = new OrderCreatedEvent(this, order);
        eventPublisher.publishEvent(event);
    }
}
```



## 消费事件

### 第一种

```java
@Component
public class MyListener implements ApplicationListener<OrderCreatedEvent> {
    @Override
    public void onApplicationEvent(OrderCreatedEvent event) {
        System.out.println(event.getOrder());
    }
}
```

### 第二种

```java
@Component
@Slf4j
public class DemoEventListener {

    @EventListener
    public void demoEventListener(OrderCreatedEvent event){
        log.info("OrderCreatedEvent listener,eventValue:{},order=2",event.getOrder());
    }
}
```

### 第三种有顺序监听

1. @Order注解
2. SmartApplicationListener

> 以上两个注解是一样的

```java
@Component
@Slf4j
public class DemoEventSmartApplicationListener implements SmartApplicationListener {
    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> aClass) {
        return aClass == AppEvent.class;
    }

    @Override
    public boolean supportsSourceType(Class<?> aClass) {
        return true;
    }

    @Override
    public int getOrder() {
        return 1;
    }

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        AppEvent event = (AppEvent)applicationEvent;
        log.info("smartapplicationListener,value:{},order=1",event.getEventData());
    }
}
```

