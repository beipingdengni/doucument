## Spring-AOP

Spring AOP（面向切面编程）是Spring框架的一个关键组件，它允许开发者定义横切关注点（cross-cutting concerns），比如事务管理、安全性、日志记录等，并将这些关注点模块化。

Spring AOP的实现基于代理模式，它在运行时为目标对象创建一个代理对象，代理会拦截对目标对象的所有方法调用，并根据配置执行相应的增强（advice）逻辑。

Spring AOP的源码比较复杂，涉及多个类和接口。下面是一些Spring AOP核心组件的简要介绍：

1. **ProxyFactoryBean**: 这是创建AOP代理的工厂Bean，它可以通过Spring配置文件或注解配置来使用。
2. **Advice**: 是一个标记接口，它是不同类型增强的顶级接口，如`BeforeAdvice`, `AfterReturningAdvice`, `ThrowsAdvice`等。
3. **Pointcut**: 定义了“何处”执行增强的逻辑，例如某些方法或者某个注解的存在。
4. **Advisor**: 它将`Advice`和`Pointcut`结合起来，告诉Spring AOP“何时”和“何处”执行增强。
5. **AopProxy**: 是创建AOP代理的核心接口，它有两个主要实现：`JdkDynamicAopProxy`（基于JDK动态代理）和`CglibAopProxy`（基于CGLIB的代理）。（通常是由 `ProxyFactory` 或者 `ProxyFactoryBean` 根据相应的配置自动处理的）
6. **MethodInterceptor**: 是`Advice`接口的子接口之一，它在目标方法执行前后允许增强逻辑的执行。

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

**@EnableAspectJAutoProxy** 

> 引入的关键包 , 使用了Aspectj

```
compile group: 'org.springframework', name: 'spring-aop', version: '5.2.1.RELEASE'
compile group: 'org.aspectj', name: 'aspectjrt', version: '1.9.5'
compile group: 'org.aspectj', name: 'aspectjweaver', version: '1.9.5'
```

**使用：ImportBeanDefinitionRegistrar 带入注册**

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

## ProxyFactory 创建代理工厂类

1. 无参构造函数
2. ProxyFactory(Object target) 指定target
3. ProxyFactory(Class[] proxyInterfaces) 指定代理类要实现的接口
4. ProxyFactory(Class proxyInterface, Interceptor interceptor) 指定代理类要实现的接口，以及一个切面
5. ProxyFactory(Class proxyInterface, TargetSource targetSource) 指定代理类要实现的接口，以及一个targetSource，targetSource类似于一个targetFactory，通过其，间接获取target

### 源代码实现

```java
// 源代码实现位置
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

### 生成一个代理对象

> 自定义能生成一个代理对象，而且我们还加了了一个环绕通知：

```java
@Test
public void createJdkDynamicProxyWithAdvisor() {
  ProxyFactory proxyFactory = new ProxyFactory();
  // 执行的service
  PerformerServiceImpl performer = new PerformerServiceImpl();
  proxyFactory.setTarget(performer);
	// 连接的接口类
  // setInterfaces和setProxyInterfaces的效果是相同的。设置需要被代理的接口，
  // 若没有实现接口，那就会采用cglib去代理
  // 需要说明的一点是：这里不设置也能正常被代理（若你没指定，Spring内部会去帮你找到所有的接口，然后全部代理上，设置的好处是只代理指定的接口
  proxyFactory.addInterface(PerformService.class);
  DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor();
  advisor.setAdvice(new MethodInterceptor() {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
      Object result = invocation.proceed();
      System.out.println("男孩唱完要行礼");
      return result;
    }
  });
	// 添加增强
  proxyFactory.addAdvisor(advisor);
  // 返回AOP代理类
  PerformerService proxy = (PerformerService) proxyFactory.getProxy();
  System.out.println("proxy class: "+proxy.getClass().getName());
  // 方法调用
  proxy.sing();
}

// 直接连接service，在执行上下拦截
ProxyFactory proxyFactory = new ProxyFactory(new HelloServiceImpl());
// 添加两个Advise，一个匿名内部类表示
proxyFactory.addAdvice((AfterReturningAdvice) (returnValue, method, args1, target) ->
                       System.out.println("AfterReturningAdvice method=" + method.getName()));
proxyFactory.addAdvice(new LogMethodBeforeAdvice()); // LogMethodBeforeAdvice implements MethodBeforeAdvice
// 获取代理类
HelloService proxy = (HelloService) proxyFactory.getProxy();
proxy.hello();

// 增加自定义pointcut： AspectJExpressionPointcut
//当然你也可以使用和容器相关的ProxyFactoryBean
ProxyFactoryBean factory = new ProxyFactoryBean();
factory.setTarget(new Person());
//声明一个aspectj切点,一张切面
String pointcutExpression = "execution(* com.example.controller.*.*(..))";
AspectJExpressionPointcut cut = new AspectJExpressionPointcut();
cut.setExpression(pointcutExpression); // 设置切点表达式
//声明一个通知（此处使用环绕通知 MethodInterceptor ）
Advice advice = (MethodInterceptor) invocation -> {
  System.out.println("============>放行前拦截...");
  Object obj = invocation.proceed();
  System.out.println("============>放行后拦截...");
  return obj;
};
// 切面=切点+通知
// 它还有个构造函数：DefaultPointcutAdvisor(Advice advice); 用的切面就是Pointcut.TRUE，所以如果你要指定切面，请使用自己指定的构造函数
// Pointcut.TRUE：表示啥都返回true，也就是说这个切面作用于所有的方法上/所有的方法
// addAdvice();方法最终内部都是被包装成一个 `DefaultPointcutAdvisor`，且使用的是Pointcut.TRUE切面，因此需要注意这些区别
Advisor advisor = new DefaultPointcutAdvisor(cut, advice);
factory.addAdvisor(advisor);
//拿到代理对象
Person p = (Person) factory.getObject();
```

## ProxyFactoryBean

在 Spring 配置文件中配置 `ProxyFactoryBean` 的基本步骤如下：

1. 定义目标对象（target）。
2. 定义增强（advice），即在执行目标方法时需要应用的横切逻辑。
3. 定义切入点（pointcut），指明增强应用于目标对象的哪些方法。
4. 使用 `ProxyFactoryBean` 将目标对象、增强和切入点组合起来。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 目标对象的定义 -->
    <bean id="myTarget" class="com.example.MyTargetClass"/>

    <!-- 增强（advice）的定义 -->
    <bean id="myAdvice" class="com.example.MyAdvice"/>

    <!-- ProxyFactoryBean 的定义 -->
    <bean id="myProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="myTarget"/>
        <property name="interceptorNames">
            <list>
                <value>myAdvice</value>
            </list>
        </property>
    </bean>
</beans>
```

> 在上述配置中，`myTarget` 是我们想要代理的目标对象，`myAdvice` 是一个增强，它包含了横切逻辑。`ProxyFactoryBean` 的 `target` 属性设置为目标对象，`interceptorNames` 属性设置为一个列表，包含了所有应用到该代理对象上的增强。
>
> 当然，Spring 也支持通过注解配置代理，使用 `@EnableAspectJAutoProxy` 注解可以开启对 AspectJ 自动代理的支持，这样就不需要显式地定义 `ProxyFactoryBean`，Spring 会自动为带有 `@Aspect` 注解的类生成代理。
>
> ```java
> @Configuration
> @EnableAspectJAutoProxy
> // spring bean xml 需要配置: <aop:aspectj-autoproxy />
> public class AppConfig {
>  // 配置类内容
> }
> ```

### aspectj 类匹配

> spring-aop：AOP核心功能，例如代理工厂等
>
> aspectjweaver：支持切入点表达式等
>
> aspectjrt：支持aop相关注解等
>
> 注：aspectjweaver包含aspectjrt，所以我们只需要引入aspectjweaver依赖包就可以了

```xml
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
  <version>1.8.0</version>
</dependency>
```

接口实现匹配类和方法

```java
AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut("execution(* service.HelloService.*(..))");
Class<HelloService> clazz = HelloService.class;
Method method = clazz.getDeclaredMethod("hello");

System.out.println("切点表达式匹配结果-类匹配："+pointcut.matches(clazz));
System.out.println("切点表达式匹配结果-方法匹配："+pointcut.matches(method, clazz));
```

