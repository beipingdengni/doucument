## 开启异步

参考博客：源码解析https://www.cnblogs.com/linuxtop/p/16969358.html

```java
@EnableAsync // 开启异步注解的支持
```

### 配置线程池大小

默认配置线程池大小

```properties
spring.task.execution.pool.core-size=8
spring.task.execution.pool.max-size=Integer.MAX_VALUE
spring.task.execution.pool.queue-capacity=Integer.MAX_VALUE
spring.task.execution.pool.keep-alive=60 //单位是：秒
```

参考修改

```properties
## 配置线程池个数
#如果是 CPU 密集型任务，那么线程池的线程个数应该尽量少一些，一般为 CPU 的个数+1条线程。
#如果是 IO 密集型任务，那么线程池的线程可以放的很大，如 2*CPU 的个数。
#对于混合型任务，如果可以拆分的话，通过拆分成 CPU 密集型和 IO 密集型两种来提高执行效率；如果不能拆分的的话就可以根据实际情况来调整线程池中线程的个数

# 线程池创建时的初始化线程数，默认为8
spring.task.execution.pool.core-size=5
# 线程池的最大线程数，默认为int最大值
spring.task.execution.pool.max-size=10
# 用来缓冲执行任务的队列，默认为int最大值
spring.task.execution.pool.queue-capacity=5
# 非核心线程的存活时间，线程终止前允许保持空闲的时间
spring.task.execution.pool.keep-alive=60
# 线程池的前缀名称，设置好了之后可以方便我们在日志中查看处理任务所在的线程池
spring.task.execution.thread-name-prefix=my-self-task-

# 是否允许核心线程超时
spring.task.execution.pool.allow-core-thread-timeout= 
# 是否等待剩余任务完成后才关闭应用
spring.task.execution.shutdown.await-termination= 
# 等待剩余任务完成的最大时间
spring.task.execution.shutdown.await-termination-period=
```

### 自定义线程池

```java
@Slf4j
@EnableAsync //对应的@Enable注解，最好写在属于自己的配置文件上，保持内聚性
@Configuration
// 第一种方式自定义实现类
public class AsyncConfig implements AsyncConfigurer {
  
// 	  //自定义，线程池（第二种方式）
//    @Bean(name = "taskExecutor")
//    public Executor taskExecutor() {
//        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
//        executor.setCorePoolSize(10);
//        executor.setMaxPoolSize(200);
//        executor.setQueueCapacity(50);
//        executor.setKeepAliveSeconds(60);
//        executor.setThreadNamePrefix("taskExecutor-ws-");
//        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
//        return executor;
//    }

   @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10); //核心线程数
        executor.setMaxPoolSize(20);  //最大线程数
        executor.setQueueCapacity(1000); //队列大小
        executor.setKeepAliveSeconds(300); //线程最大空闲时间
        executor.setThreadNamePrefix("async-Executor-"); 指定用于新创建的线程名称的前缀。
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy()); // 拒绝策略（一共四种）
       /**
         * 拒绝处理策略
         * CallerRunsPolicy()：交由调用方线程运行，比如 main 线程。
         * AbortPolicy()：直接抛出异常。
         * DiscardPolicy()：直接丢弃。
         * DiscardOldestPolicy()：丢弃队列中最老的任务。
         */
        /**
         * 特殊说明：
         * 1. 这里演示环境，拒绝策略咱们采用抛出异常
         * 2.真实业务场景会把缓存队列的大小会设置大一些，
         * 如果，提交的任务数量超过最大线程数量或将任务环缓存到本地、redis、mysql中,保证消息不丢失
         * 3.如果项目比较大的话，异步通知种类很多的话，建议采用MQ做异步通知方案
         */
        // 这一步千万不能忘了，否则报错： java.lang.IllegalStateException: ThreadPoolTaskExecutor not initialized
        // 线程初始化
      	executor.initialize();
        return executor;
    }
    
    // 异常处理器：当然你也可以自定义的，这里我就这么简单写了~~~
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return return new AsyncUncaughtExceptionHandler() {

            @Override
            public void handleUncaughtException(Throwable arg0, Method arg1, Object... arg2) {
                log.error("=========================="+arg0.getMessage()+"=======================", arg0);
                log.error("exception method:"+arg1.getName());
            }
        };
    }

}
```



### 使用线程池

```java
@Component 
public class AsyncMyTask {    
    protected final Logger logger = LoggerFactory.getLogger(this.getClass());     
    // 直接使用该注解即可
    @Async    
    public void doTask(int i) throws InterruptedException{        
        logger.info("Task-Native" + i + " started.");    
    } 
}

// 注入ThreadPoolTaskExecutor
@Autowired
private ThreadPoolTaskExecutor threadPoolTaskExecutor;

threadPoolTaskExecutor.execute(() -> {
    System.out.println("lambda 1 当前线程 "+Thread.currentThread().getName());
});
```

