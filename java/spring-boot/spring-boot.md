### 微服务 和 集群

#### 相关文档

##### [中文文档](https://springcloud.cc/)

#### [官方文档](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#boot-features-spring-application)

####  [官方网站](https://spring.io/)



```java
判断服务：SERVLET/REACTIVE
org.springframework.boot.WebApplicationType#deduceFromClasspath
加载环境
org.springframework.boot.SpringApplication#getSpringFactoriesInstances(java.lang.Class<T>, java.lang.Class<?>[], java.lang.Object...)
		加载包下的文件：/spring-boot-2.1.7.RELEASE.jar!/META-INF/spring.factories
 		org.springframework.core.io.support.SpringFactoriesLoader#loadFactoryNames
  		定位缓存
  		org.springframework.core.io.support.SpringFactoriesLoader#loadSpringFactories

准备环境
org.springframework.boot.SpringApplication#prepareEnvironment
环境监听【 Listeners 】
org.springframework.boot.SpringApplicationRunListeners#environmentPrepared
发送通知：
	org.springframework.cloud.bootstrap.BootstrapApplicationListener
  
      接受通知【初始化核心的方法，准备环境】：
      org.springframework.cloud.bootstrap.BootstrapApplicationListener#bootstrapServiceContext
         调用方法
         SpringApplicationBuilder builder = (new SpringApplicationBuilder(new Class[0]))
       	.profiles(environment.getActiveProfiles())
        .bannerMode(Mode.OFF)
        .environment(bootstrapEnvironment)
        .registerShutdownHook(false)
        .logStartupInfo(false)
        .web(WebApplicationType.NONE);
        方法调用：【调用到了 org.springframework.boot.SpringApplication#run(java.lang.String...) 】
          ConfigurableApplicationContext context = builder.run(new String[0]);
				处理：ApplicationContextInitializer
				 org.springframework.cloud.bootstrap.BootstrapApplicationListener#apply

      本地配置文件加载：
      发送通知事件：
        org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent
      接受通知：
        本地配置文件，application.yaml, bootstrap.yaml
        org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEvent
        日志：
  org.springframework.boot.context.logging.LoggingApplicationListener#onApplicationEvent

创建容器
org.springframework.boot.SpringApplication#createApplicationContext
	  类型为servlet：org.springframework.boot.WebApplicationType#SERVLET
  	org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext

开始准备
org.springframework.boot.SpringApplication#prepareContext
  调用处理【 🐶 ApplicationContextInitializer】
  org.springframework.boot.SpringApplication#applyInitializers
  监听处理【准备】
  org.springframework.boot.SpringApplicationRunListeners#contextPrepared
	加载自动配置类  org.springframework.boot.SpringApplication#loadorg.springframework.boot.SpringApplication#loadorg.springframework.boot.SpringApplication#load
          
核心❤️---刷新【refresh】
org.springframework.boot.SpringApplication#refreshContext
  最后调用：
  org.springframework.context.support.AbstractApplicationContext#refresh
  
调用 
org.springframework.boot.SpringApplication#callRunners
  org.springframework.boot.ApplicationRunner
  org.springframework.boot.CommandLineRunner
  
监听开始运行【start】
org.springframework.boot.SpringApplicationRunListeners#started
调用【running】
org.springframework.boot.SpringApplicationRunListeners#running
  
```



#### 加载配置文件核心：

  //   org.springframework.boot.env.EnvironmentPostProcessor

【接受监听事件】org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEnvironmentPreparedEvent

【系统】org.springframework.boot.context.config.ConfigFileApplicationListener.Loader#load()

【cloud】org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration

【参数绑定】org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor#bind

```

// 开始调用 BeanFactoryPostProcessors
org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors

注册到BeanDefinitionRegistry中
org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanDefinitionRegistryPostProcessors

环境参数适配器
org.springframework.core.env.PropertySourcesPropertyResolver
获取属性
org.springframework.core.env.AbstractEnvironment#propertyResolver

绑定参数：
Binder.get(this.environment).bind("spring.cloud.refresh",
      Bindable.ofInstance(this));
```



#### ❤️ 处理【spring.factories 文件配置】

```
获取注解类【配置全名称】
SpringFactoriesLoader.loadFactoryNames(this.annotationClass, this.beanClassLoader))

环境参数处理：
org.springframework.context.annotation.ConfigurationClassPostProcessor#postProcessBeanFactory
	
	ConfigurationClassParser parser = new ConfigurationClassParser(
          this.metadataReaderFactory, 							
          this.problemReporter, 
          this.environment,
					this.resourceLoader, 
					this.componentScanBeanNameGenerator,
          registry)
   处理：bootstrapImportSelectorConfiguration、启动Application

关注一下，可以自定注解启动类【配合/spring.factories 】：
org.springframework.cloud.commons.util.SpringFactoryImportSelector#selectImports


```



##### 配置注解

```
@ConditionalOnBean（仅仅在当前上下文中存在某个对象时，才会实例化一个Bean）
@ConditionalOnClass（某个class位于类路径上，才会实例化一个Bean）
@ConditionalOnExpression（当表达式为true的时候，才会实例化一个Bean）
@ConditionalOnMissingBean（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
@ConditionalOnMissingClass（某个class类路径上不存在的时候，才会实例化一个Bean）
@ConditionalOnNotWebApplication（不是web应用）
```



##### 组件

```
获取本地IP
org.springframework.cloud.client.HostInfoEnvironmentPostProcessor#getFirstNonLoopbackHostInfo
// 获取ip
org.springframework.cloud.commons.util.InetUtils

初始化广播方法：
org.springframework.context.event.SimpleApplicationEventMulticaster

注册监听事件：
	org.springframework.context.support.AbstractApplicationContext#registerListeners

refresh方法中：
org.springframework.context.support.AbstractApplicationContext#refresh
BeanFactoryPostProcessor 处理器工作
org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors
处理：RefreshScope	
org.springframework.cloud.context.scope.refresh.RefreshScope#postProcessBeanDefinitionRegistry
mapper:
org.mybatis.spring.mapper.MapperScannerConfigurer#postProcessBeanDefinitionRegistry
处理 RefreshScope
org.springframework.cloud.autoconfigure.RefreshAutoConfiguration.RefreshScopeBeanDefinitionEnhancer#postProcessBeanDefinitionRegistry

servlet处理结束发送事件
org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#finishRefresh
初始化完成
org.springframework.boot.web.servlet.context.ServletWebServerInitializedEvent

注册：EurekaAutoServiceRegistration
org.springframework.cloud.netflix.eureka.serviceregistry.EurekaAutoServiceRegistration#onApplicationEvent(org.springframework.context.ApplicationEvent)
		WebServerInitializedEvent 处理注册

注解context
org.springframework.context.annotation.AnnotationConfigApplicationContext
配置文件
org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext
```



