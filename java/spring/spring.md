## Spring



```java

启动拦截：如：EnableFeignClients 等
接口： ImportBeanDefinitionRegistrar

扫描文件，创建BeanDefinition：
	ClassPathScanningCandidateComponentProvider
	ClassPathBeanDefinitionScanner
	// bean前置处理注入bean
	BeanDefinitionRegistryPostProcessor
```



### BeanDefinitionRegistry 和 BeanDefinitionHolder

```java
两种方式创建类

// 自定义创建类
BeanDefinitionBuilder definition = BeanDefinitionBuilder
				.genericBeanDefinition(FeignClientFactoryBean.class);
BeanDefinitionHolder definitionHolder=new BeanDefinitionHolder(
  																					beanDefinition,
   																					beanName,
  																					aliases)
BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, registry);

// 自定义获取Bean
BeanDefinitionBuilder builder = BeanDefinitionBuilder
																	.genericBeanDefinition(FeignClientSpecification.class);
BeanDefinition beanDefinition =builder.getBeanDefinition()
registry.registerBeanDefinition(beanName,beanDefinition)
```



