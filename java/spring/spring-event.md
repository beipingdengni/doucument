



## spring event



### ApplicationContextEvent 抽象类，可以自定义实现

Spring框架本身提供的一些与`ApplicationContextEvent`相关的事件包括：

1. `ContextRefreshedEvent`：当`ApplicationContext`被初始化或刷新时触发。这通常发生在容器实例化（通过调用`refresh()`方法）时。
2. `ContextStartedEvent`：当Spring容器调用`ConfigurableApplicationContext`的`start()`方法开始或重新开始容器时触发。
3. `ContextStoppedEvent`：当Spring容器调用`ConfigurableApplicationContext`的`stop()`方法停止容器时触发。
4. `ContextClosedEvent`：当`ApplicationContext`被关闭时触发。容器被关闭时，其管理的所有单例Bean都会被销毁。



### ApplicationListener

```java
import org.springframework.context.ApplicationListener;  
import org.springframework.context.event.ContextRefreshedEvent;  
import org.springframework.stereotype.Component;  
  
@Component  
public class MyContextRefreshedEventListener implements ApplicationListener<ContextRefreshedEvent> {  
  
    @Override  
    public void onApplicationEvent(ContextRefreshedEvent event) {  
        // 这里写你的初始化逻辑  
    }  
}
```

### 增加异步多线程处理【默认单现场】

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(10);
        threadPoolTaskExecutor.setMaxPoolSize(50);
        threadPoolTaskExecutor.setQueueCapacity(5);
        threadPoolTaskExecutor.setKeepAliveSeconds(1);
        threadPoolTaskExecutor.initialize();
        return threadPoolTaskExecutor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new AsyncUncaughtExceptionHandler() {
            @Override
            public void handleUncaughtException(Throwable throwable, Method method, Object... objects) {
                System.out.println("出现异常啦~~~~~~");
            }
        };
    }
}

// 或者

	/**
    * 核心线程数10：线程池创建时初始化的线程数
    * 最大线程数20：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
    * 缓冲队列200：用来缓冲执行任务的队列
    * 允许线程的空闲时间60秒：超过了核心线程数之外的线程，在空闲时间到达之后会被销毁
    * 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
    * 线程池对拒绝任务的处理策略：此处采用了CallerRunsPolicy策略，当线程池没有处理能力的时候，该策略会直接在execute方法的调用线程中运行被拒绝的任务；如果执行程序已被关闭，则会丢弃该任务
    * 设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
    * 设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住
    */
    @EnableAsync
    @Configuration
    class TaskPoolConfig{
      @Bean("taskExecutor") // 名称一定要对
      public Executor taskExecutor(){
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("TaskExecutor-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        return executor;
      }
    }

```



### 原理

> 发布服务执行：AbstractApplicationContext#publishEvent
>
> 调用处理广播：ApplicationEventMulticaster#multicastEvent

默认实现：SimpleApplicationEventMulticaster

