## Spring-AOP

### 使用

```java
// 第一种
@Pointcut("execution(* com.tbp.base.web.controller.*Controller.*(..))")
public void interceptController() {}

@Around("interceptController()")
public Object handle(ProceedingJoinPoint joinPoint) throws Throwable {
 Object[] args = joinPoint.getArgs();
 return joinPoint.proceed(args);
}

// 第二种
@Around("execution (* com.tbp.boot.facade..*(..))")
public Object recordTime(ProceedingJoinPoint joinPoint) {
  Object[] args = joinPoint.getArgs();
  return joinPoint.proceed(args);
}
```







### 开启aop 

参考博客：https://www.cnblogs.com/liuyk-code/p/9886033.html

@EnableAspectJAutoProxy 

> 引入的关键包 , 使用了Aspectj

```
compile group: 'org.springframework', name: 'spring-aop', version: '5.2.1.RELEASE'
compile group: 'org.aspectj', name: 'aspectjrt', version: '1.9.5'
compile group: 'org.aspectj', name: 'aspectjweaver', version: '1.9.5'
```

使用：ImportBeanDefinitionRegistrar 带入注册

```
// 配置类 
// 使用：SmartInstantiationAwareBeanPostProcessor 继承 InstantiationAwareBeanPostProcessor
自定义注入类：org.springframework.aop.config.internalAutoProxyCreator
// 创建前后置处理器
AnnotationAwareAspectJAutoProxyCreator 
		extends AspectJAwareAdvisorAutoProxyCreator  
			extends AbstractAdvisorAutoProxyCreator
				extends AbstractAutoProxyCreator
					 extends ProxyProcessorSupport
							implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware 
// 处理器
AbstractAutoProxyCreator
【1】#postProcessBeforeInstantiation 处理带有【@aspect】注解的类【实例前，前置】
 		#shouldSkip 是否跳过
 			#findCandidateAdvisors 查找所有bean含有需要处理
 				#AspectJAdvisorsBuilder.buildAspectJAdvisors() 【处理所有的增加器】
 					将获取到增加器放入：类：BeanFactoryAspectJAdvisorsBuilder 
 															属性：Map<String, List<Advisor>> advisorsCache
 		
【2】#postProcessBeforeInstantiation 处理创建代理类 【实例后，前置】
		#wrapIfNecessary
 			#getAdvicesAndAdvisorsForBean 获取拦截器
 				#AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass); 获取能用的增强器
 				#sortAdvisors 排序增强器
	 		#createProxy  创建拦截器
	 			# 创建类：ProxyFactory 做代理
【3】

DefaultAopProxyFactory 默认aop 创建工厂

ReflectiveMethodInvocation
	# proceed
	开始调用方法

ReflectiveAspectJAdvisorFactory
	getAdvisors 获取增加器

```

#### ProxyFactory 创建代理工厂类

```
		ProxyFactory proxyFactory = new ProxyFactory();
		proxyFactory.copyFrom(this);

		if (!proxyFactory.isProxyTargetClass()) {
			if (shouldProxyTargetClass(beanClass, beanName)) {
				proxyFactory.setProxyTargetClass(true);
			}
			else {
				evaluateProxyInterfaces(beanClass, proxyFactory);
			}
		}

		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		proxyFactory.addAdvisors(advisors);
		proxyFactory.setTargetSource(targetSource);
		customizeProxyFactory(proxyFactory);

		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}
```

