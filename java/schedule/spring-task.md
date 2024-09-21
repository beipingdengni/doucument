

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

# 自定义动态添加任务

## 核心类处理

> config.ScheduledTaskRegistrar#scheduleTasks

```java
@Configuration      //1.主要用于标记配置类，兼备Component的效果。
@EnableScheduling   // 2.开启定时任务
public class DynamicScheduleTask implements SchedulingConfigurer {
 
    @Mapper
    public interface CronMapper {
      @Select("select cron from cron limit 1")
      public String getCron();
   }

    @Autowired //注入mapper // 数据库中获取
    CronMapper cronMapper;
 		
  	// 执行定时任务
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
			 //设置20个线程,默认单线程,如果不设置的话，不能同时并发执行任务
        registrar.setScheduler(Executors.newScheduledThreadPool(10));
        this.registrar = registrar;
     
        taskRegistrar.addTriggerTask(
            //1.添加任务内容(Runnable)
            // 这里的代码可以变，
            () -> System.out.println("执行动态定时任务: " + LocalDateTime.now().toLocalTime()),
            //2.设置执行周期(Trigger)
            // 基本不变
            triggerContext -> {
                //2.1 从数据库获取执行周期
                String cron = cronMapper.getCron();
                //2.2 合法性校验.
                if (StringUtils.isEmpty(cron)) {
                    // Omitted Code ..
                }
                //2.3 返回执行周期(Date)
                return new CronTrigger(cron).nextExecutionTime(triggerContext);
            }
        );
    }
  
  	// 任务添加
  	public void	addJob(String jobId,String beanName,String cron){
      	// ScheduledFuture<?> future = registrar.getScheduler().schedule(task.getRunnable(), task.getTrigger());
      	// registrar.addScheduledTask() //需要调用执行
        registrar.scheduleTriggerTask(new CronTask(new Runnable(){ /*.....*/ },cron))
    } 
  	
  	// 任务删除
	  public void	stopAll(){
      	 registrar.getScheduledTasks().forEach(future->{
           // mayInterruptIfRunning设成false话，不允许在线程运行时中断，设成true的话就允许。
           future.cancel(false);
        });
    } 
  
  	@PreDestroy
    public void destroy() {
        this.registrar.destroy();
    }
}
```

### 其他的实现方式（自定义）

```java
@Configuration
public class SchedulingConfig {
		// 需要的任务现场池
    @Bean
    public TaskScheduler taskScheduler() {
        // 获取系统处理器个数, 作为线程池数量
        int corePoolSize = Runtime.getRuntime().availableProcessors();
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        // 定时任务执行线程池核心线程数
        taskScheduler.setPoolSize(corePoolSize);
        taskScheduler.setRemoveOnCancelPolicy(true);
        taskScheduler.setThreadNamePrefix("AntiFraudSchedulerThreadPool-");
        return taskScheduler;
    }
}
// 创建ScheduledTask包装类 ScheduledFuture是ScheduledFuture<?> schedule(Runnable task, Trigger trigger)方法的返回值。

//ScheduledFuture继承了Future接口，Future接口提供一组辅助方法，比如：
//	cancel()：取消任务
//	isCancelled()：任务是不是取消了
//	isDone()：任务是不是已经完成了
//	get()：用来获取执行结果

// 调度类里面可以取消
// 调用cancel方法时，会将我们的任务从workQueue中移除
public final class ScheduledTask {
		// 远程调用
    public volatile ScheduledFuture<?> future;
		// 取消定时任务
    public void cancel() {
        ScheduledFuture<?> scheduledFuture = this.future;
        if (Objects.nonNull(scheduledFuture)) {
            scheduledFuture.cancel(true);
        }
    }
}

//  自动增加定时任务
@Slf4j
@Component
public class CronTaskRegistrar implements DisposableBean {

    @Resource
    private TaskScheduler taskScheduler;

    // 保存任务Id和定时任务
    private final Map<String, ScheduledTask> scheduledTaskMap = new ConcurrentHashMap<>(64);

    // 添加任务
    public void addTask(Runnable task, String cronExpression,String jobId) {
        addTask(new CronTask(task, cronExpression),jobId);
    }

    public void addTask(CronTask cronTask,String jobId) {
        if (Objects.nonNull(cronTask)) {
            Runnable task = cronTask.getRunnable();
            if (this.scheduledTaskMap.containsKey(task)) {
                removeTask(jobId);
            }
            // 保存任务Id和定时任务
            this.scheduledTaskMap.put(jobId, scheduleTask(cronTask));
        }
    }

    // 通过任务Id，取消定时任务
    public void removeTask(String jobId) {
        ScheduledTask scheduledTask = this.scheduledTaskMap.remove(jobId);
        if (Objects.nonNull(scheduledTask)) {
            scheduledTask.cancel();
        }
    }

    public ScheduledTask scheduleTask(CronTask cronTask) {
        ScheduledTask scheduledTask = new ScheduledTask();
        scheduledTask.future = this.taskScheduler.schedule(cronTask.getRunnable(), cronTask.getTrigger());
        return scheduledTask;
    }

    // 销毁
    @Override
    public void destroy() {
        this.scheduledTaskMap.values().forEach(ScheduledTask::cancel);
        this.scheduledTaskMap.clear();
    }
}

// 添加顺序
// cron--》runable--》CronTask--》ScheduledTask--》taskScheduler
```

