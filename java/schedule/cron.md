## 工具类

```xml
<dependency>
  <groupId>com.cronutils</groupId>
  <artifactId>cron-utils</artifactId>
  <version>9.1.5</version>
</dependency>
```

### spring cron

```java
// 可以使用redis或数据库来实现
SimpleTriggerContext simpleTriggerContext = new SimpleTriggerContext(); // TriggerContext

// 创建cron表达式计算
CronTrigger cronTrigger = new CronTrigger("*/5 * * * * ?");

// 获取下一次执行的时间
Date date = cronTrigger.nextExecutionTime(simpleTriggerContext);

// 注解处理
// org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor#processScheduled
// org.springframework.scheduling.config.ScheduledTaskRegistrar#scheduleCronTask // 增加cron

// 这个注册循环调用
// Runnable task
// new ReschedulingRunnable(task, trigger, this.clock, this.scheduledExecutor, errorHandler).schedule();
```

### quartz cron

```java
org.quartz.CronExpression
org.quartz.CronTrigger
  org.quartz.impl.triggers.CronTriggerImpl

// 间隔五分钟就处理
CronTriggerImpl cronTriggerImpl = new CronTriggerImpl("cron的名字","group任务所在分组","0 */ * * * ?");
// 下次执行时间
Date nextDate = cronTriggerImpl.getNextFireTime(); 

// job任务：jobClass 需要继承接口：org.quartz.Job
JobDetailImpl jobDetailImpl =new JobDetailImpl("job的名字","job的group任务所在分组",jobClass)

// org.quartz.JobBuilder
// org.quartz.TriggerBuilder
  // org.quartz.CronScheduleBuilder
```



### Cron表达式举例

```properties
"30 * * * * ?" 每半分钟触发任务
"30 10 * * * ?" 每小时的10分30秒触发任务
"30 10 1 * * ?" 每天1点10分30秒触发任务
"30 10 1 20 * ?" 每月20号1点10分30秒触发任务
"30 10 1 20 10 ? *" 每年10月20号1点10分30秒触发任务
"30 10 1 20 10 ? 2011" 2011年10月20号1点10分30秒触发任务
"30 10 1 ? 10 * 2011" 2011年10月每天1点10分30秒触发任务
"30 10 1 ? 10 SUN 2011" 2011年10月每周日1点10分30秒触发任务
"15,30,45 * * * * ?" 每15秒，30秒，45秒时触发任务
"15-45 * * * * ?" 15到45秒内，每秒都触发任务
"15/5 * * * * ?" 每分钟的每15秒开始触发，每隔5秒触发一次
"15-30/5 * * * * ?" 每分钟的15秒到30秒之间开始触发，每隔5秒触发一次
"0 0/3 * * * ?" 每小时的第0分0秒开始，每三分钟触发一次
"0 15 10 ? * MON-FRI" 星期一到星期五的10点15分0秒触发任务
"0 15 10 L * ?" 每个月最后一天的10点15分0秒触发任务
"0 15 10 LW * ?" 每个月最后一个工作日的10点15分0秒触发任务
"0 15 10 ? * 5L" 每个月最后一个星期四的10点15分0秒触发任务
"0 15 10 ? * 5#3" 每个月第三周的星期四的10点15分0秒触发任务
```

