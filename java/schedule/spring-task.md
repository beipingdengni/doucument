

## 

## 注解方式

```xml
<context:annotation-config />
<!-- 自动调度需要扫描的包 --> 
<context:component-scan base-package="com.tbp.worker"/>
<!-- 定时器开关 -->
<task:executor id="executor" pool-size="5"/>
<task:scheduler id="scheduler" pool-size="10"/>
<task:annotation-driven executor="executor" scheduler="scheduler"/>

以上配置如同下
@EnableScheduling
```

### 代码中添加注解

```java
@Scheduled(cron = "*/5 * * * * ? ")
public void doSomething() {
// something that should execute periodically
}
```

## xml配置方式

```xml
<context:annotation-config />
<!-- 自动调度需要扫描的包 --> 
<context:component-scan base-package="com.tbp.worker"/>

<!-- 定时器开关 -->
<task:executor id="executor" pool-size="5"/>
<task:scheduler id="scheduler" pool-size="10"/>
<task:annotation-driven executor="executor" scheduler="scheduler"/>

<!-- 配置调度 需要在类名前添加 @Service -->  
<task:scheduled-tasks>
	<task:scheduled ref="demoTask" method="myTestWork" cron="0/10 * * * * ?"/>
</task:scheduled-tasks>
<!-- 不通过配置调度,需要在类名前 @Component/@Service,在方法名 前添加@Scheduled(cron="0/5 * * * * ? ")、即用注解的方式-->  
```

## 自定义实现(提交定时任务)

```java
// org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler
//配置定时任务线程池-自定义名称避免冲突
@Bean(name = "myThreadPoolTaskScheduler")
public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
  ThreadPoolTaskScheduler executor = new ThreadPoolTaskScheduler();
  executor.setPoolSize(2);
  executor.setThreadNamePrefix("task-");
  executor.setWaitForTasksToCompleteOnShutdown(true);
  executor.setAwaitTerminationSeconds(60);
  return executor;
}

// org.springframework.scheduling.TaskScheduler
myThreadPoolTaskScheduler.schedule(task, new CronTrigger("0/10 * * * * ?"));
```

### Spring 线程池

```java
@Bean
public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
  ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
  int i = Runtime.getRuntime().availableProcessors();
  //核心线程数目
  executor.setCorePoolSize(i * 2);
  //指定最大线程数
  executor.setMaxPoolSize(i * 2);
  //队列中最大的数目
  executor.setQueueCapacity(i * 2 * 10);
  //线程名称前缀
  executor.setThreadNamePrefix("ThreadPoolTaskExecutor-");
  //rejection-policy：当pool已经达到max size的时候，如何处理新任务
  //CALLER_RUNS：不在新线程中执行任务，而是由调用者所在的线程来执行
  //对拒绝task的处理策略
  executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
  //当调度器shutdown被调用时等待当前被调度的任务完成
  executor.setWaitForTasksToCompleteOnShutdown(true);
  //线程空闲后的最大存活时间
  executor.setKeepAliveSeconds(60);
  //加载
  executor.initialize();
  log.info("初始化线程池成功");
  return executor;
}

```

