## maven xml

```xml
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
  <version>2.3.0</version>
</dependency>
```

## quartz.properties

```properties
org.quartz.scheduler.instanceName = MyScheduler
org.quartz.threadPool.threadCount = 3
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
#激活失败容忍度，只有超过这个容忍度才会判定位misfire
org.quartz.jobStore.misfireThreshold=30000
```

### java 代码实现

#### 构建job

```java
public class HelloWord implements Job{  
    //实现自己的定时方法  
    public void execute(JobExecutionContext context) throws JobExecutionException {  
        System.out.println("hello world " + new Date());  
    }  
} 
或者
//继承QuartzJobBean,并重写executeInternal方法  
public class QuartzTask extends QuartzJobBean{  

    private static int i = 0;  
    
    @Override  
    protected void executeInternal(JobExecutionContext context)  throws JobExecutionException {  
        System.out.println("task running..."+ ++i + "进行中...");  
    }    
} 
```

#### 启动任务

```java
//获取scheduler实例  
Scheduler scheduler=StdSchedulerFactory.getDefaultScheduler();  
scheduler.start();  

//当前时间  
Date runTime=new Date();  

//定义一个 job 对象并绑定我们写的  HelloWord 类     
// 真正执行的任务并不是Job接口的实例，而是用反射的方式实例化的一个JobDetail实例   
JobDetail job=JobBuilder.newJob(HelloWord.class).withIdentity("job1","group1").build();  

// 定义一个触发器，startAt方法定义了任务应当开始的时间 .即下一个整数分钟执行  
Trigger trigger=TriggerBuilder.newTrigger().withIdentity("trigger1","group1").startAt(runTime).build();  

// CronTrigger的指令常量失火
CronTrigger cronTrigger = TriggerBuilder.newTrigger()
                .withIdentity("trigger1","group1")
                .startNow()
  							.withSchedule(CronScheduleBuilder.cronSchedule("0 0/2 8-17 * * ?"))
                //.withSchedule(CronScheduleBuilder.cronSchedule("0 0/2 8-17 * * ?").withMisfireHandlingInstructionFireAndProceed())
  							//.forJob("myJob", "group1")
	  						//.inTimeZone(TimeZone.getTimeZone("Asia/Shanghai"))
                .build();
//withMisfireHandlingInstructionFireNow（失效之后再恢复并马上执行）
//withMisfireHandlingInstructionNextWithRemainingCount（失效之后不处理） 

  
// 将job和Trigger放入scheduler  
scheduler.scheduleJob(job, trigger);  

//启动  
scheduler.start();  
```

失效策略，重启任务执行配置

​	https://blog.csdn.net/like_java_/article/details/108128022



> Scheduler scheduler = schedulerFactoryBean.getScheduler();   // spring中获取 SchedulerFactoryBean
>
> Scheduler scheduler=StdSchedulerFactory.getDefaultScheduler();  // 自定义 quartz

### 添加

任务实现执行逻辑类`clusteredJobClass `为job 接口的实现类

传递到上线文中参数`data`是一个`hashmap`

```java
String jobGroupName = jobName + ".g";
String triggerName = jobName + ".t";
String triggerGroupName = jobName + ".tg";

TriggerBuilder builder = TriggerBuilder.newTrigger().withIdentity(triggerName, triggerGroupName);
if (StringUtils.isBlank(cron)) {
  builder.startNow().forJob(jobName, jobGroupName);
} else {
  builder.withSchedule(CronScheduleBuilder.cronSchedule(cron).withMisfireHandlingInstructionDoNothing());
}

scheduler.scheduleJob(JobBuilder.newJob().withIdentity(jobName, jobGroupName)
                      .storeDurably(true).ofType(clusteredJobClass) // job任务
                      .setJobData(new JobDataMap(data)) // 上下文参数
                      .build(), 
                      builder.build());
```

### 删除

```java
String jobGroupName = jobName + ".g";
String triggerName = jobName + ".t";
String triggerGroupName = jobName + ".tg";

TriggerKey triggerKey = new TriggerKey(triggerName, triggerGroupName);
scheduler.pauseTrigger(triggerKey);
scheduler.unscheduleJob(triggerKey);
scheduler.deleteJob(new JobKey(jobName, jobGroupName));
```

### 暂停

```java
 String jobGroupName = jobName + ".g";
 String triggerName = jobName + ".t";
 String triggerGroupName = jobName + ".tg";

scheduler.pauseJob(new JobKey(jobName, jobGroupName));
scheduler.pauseTrigger(new TriggerKey(triggerName, triggerGroupName));
```

### 重启

```
String jobGroupName = jobName + ".g";
String triggerName = jobName + ".t";
String triggerGroupName = jobName + ".tg";

scheduler.resumeJob(new JobKey(jobName, jobGroupName));
scheduler.resumeTrigger(new TriggerKey(triggerName, triggerGroupName));
```

