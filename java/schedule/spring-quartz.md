## maven xml

```xml
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
  <version>2.3.0</version>
</dependency>
```

## 基础xml配置任务

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  
     <!--注入任务处理类bean  -->   
     <bean id="quartzTask" class="com.tbp.schedule.MyQuartzTask"></bean> 
   
     <!-- 1.简单触发器任务详细信息bean -->   
     <bean id="myJobDetail1" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
     	<!-- 设置任务执行对象 -->
     	<property name="targetObject" ref="quartzTask"></property>
     	<!-- 设置任务执行对象中对应的执行方法 -->
     	<property name="targetMethod" value="doSimpleTask"></property>
     	<!-- 设置任务是否可并发执行，默认为不并发 -->
     	<property name="concurrent" value="false"></property>
     </bean>

<!--	&lt;!&ndash; 2.制定简单触发器 &ndash;&gt;-->
<!--	<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">-->
<!--		&lt;!&ndash; 设置任务详细信息 &ndash;&gt;-->
<!--		<property name="jobDetail" ref="myJobDetail1"></property>-->
<!--		&lt;!&ndash; 设置延迟1秒开始执行,自己看情况 &ndash;&gt;-->
<!--		<property name="startDelay" value="1000"></property>-->
<!--		&lt;!&ndash; 设置任务执行间隔，此处设置的是每5秒执行一次  &ndash;&gt;-->
<!--		<property name="repeatInterval" value="5000"></property>-->
<!--	</bean>-->
  
  	 <!-- 2.任务触发器 -->
     <bean id="cronTrigger2" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
     	<!-- 设置任务详细信息 -->
     	<property name="jobDetail" ref="myJobDetail2"></property>
     	<!-- 设置quartz任务执行表达式 ,每隔三秒执行一次任务-->
     	<property name="cronExpression" value="0/3 * * * * ?"></property>
     </bean>
   
     <!-- 设置触发器调度工厂 -->   
     <bean id="schedulerFactory" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
       <!-- <property name="schedulerContextAsMap">  //②以Map类型设置SchedulerContext数据
　　　　			<map>
　　　　　　			<entry key="timeout" value="30" />
　　　　			</map>
　　				</property> --> 
      <!-- <property name="configLocation" value="classpath:com/smart/quartz/quartz.properties" /> -->
     	<property name="triggers">
     		<list>
     			<!-- 触发器调度工厂调度简单触发器 -->
     			<ref bean="simpleTrigger"/>
     			<!--<ref bean=""> 多个触发器可以这样配置-->
     		</list>
     	</property>
     </bean> 
</beans>
```

### SchedulerFactoryBean常见的属性

```groovy
SchedulerFactoryBean 还拥有一些常见的属性。
（1）calendars：类型为 Map，通过该属性向 Scheduler 注册 Calendar。
（2）jobDetails：类型为 JobDetail[]，通过该属性向 Scheduler 注册 JobDetail。
（3）autoStartup：SchedulerFactoryBean 在初始化后是否马上启动 Scheduler，默认为 true。如果设置为  false，则需要手工启动 Scheduler。
（4）startupDelay：在 SchedulerFactoryBean 初始化完成后，延迟多少秒启动 Scheduler，默认值为0，表示马上启动。除非拥有需要立即执行的任务，一般情况下，可以通过属性让 Scheduler 延迟一小段时间后启动，以便让 Spring 能够更快初始化容器中剩余的 Bean。

SchedulerFactoryBean 的一个重要功能是允许用户将 Quartz 配置文件中的信息转移到 Spring 配置文件中，由此带来的好处是配置信息的集中化管理，同时我们不必熟悉多种框架有差异的配置文件结构。
（1）datasource：当需要使用数据库来持久化任务调度数据时，用户可以在 Quartz 中配置数据源，也可以直接在 Spring 中通过 datasource 指定一个 Spring 管理的数据源。如果指定了该属性，那么，即使在 quartz.properties 中已经定义了数据源，也会被 dataSource 覆盖。
（2）transactionManager：可以通过该属性设置一个 Spring 事务管理器。在设置 datasource 时，Spring 强烈推荐用户使用一个事务管理器，否则数据表锁定可能无法正常工作。
（3）nonTransactionalDataSource：在全局事务的情况下，如果用户不希望 Scheduler 执行的相关数据操作参与到全局事务中，则可以通过该属性指定数据源。在 Spring 本地事务的情况下，使用 datasource 属性就足够了。
（4）quartzProperties：类型为 Properties，允许用户在 Spring 中定义 Quartz 的属性，其值将覆盖  quartz.properties配置文件中的设置。这些属性必须是Quartz能够识别的合法属性，在配置时，需要查看 Quartz 的相关文档。

<bean id="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
　　...
　　<property name="quartzProperties">  //①Quartz属性项1
　　　　<props>
　　　　　　<prop key="org.quartz.threadPool.class">
　　　　　　　　org.quartz.simpl.SimpleThreadPool
　　　　　　</prop>
　　　　　　<prop key="org.quartz.threadPool.threadCount">10</prop>//②Quartz属性项2
　　　　</props>
　　</property>
</bean>
```