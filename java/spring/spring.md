## Spring 启动顺序

<img src="https://pic2.zhimg.com/80/v2-f7402c5f14604c9ce0d5b0bb3a1fba35_1440w.webp" alt="img" style="zoom:50%;" />



## Spring BeanFactory

```java
启动拦截：如：EnableFeignClients 等
接口： ImportBeanDefinitionRegistrar

扫描文件，创建BeanDefinition：
	ClassPathScanningCandidateComponentProvider
	ClassPathBeanDefinitionScanner
	// bean前置处理注入bean
	BeanDefinitionRegistryPostProcessor
```

## spring bean 生命周期

> AbstractAutowireCapableBeanFactory#doCreateBean

1. Spring Bean 的生命周期涉及多个步骤，从创建到销毁。以下是主要的生命周期阶段：

   1. **实例化（Instantiation）**:
      Spring容器通过反射机制创建Bean实例。

   2. **设置属性（Populate Properties）**:
      Spring容器注入配置在Bean定义中的属性值。

   3. **BeanNameAware**:
      如果Bean实现了`BeanNameAware`接口，Spring将调用`setBeanName()`方法，传入Bean的ID。

   4. **BeanFactoryAware**:
      如果Bean实现了`BeanFactoryAware`接口，Spring将调用`setBeanFactory()`方法，传入`BeanFactory`。

   5. **ApplicationContextAware**:
      如果Bean实现了`ApplicationContextAware`接口，Spring将调用`setApplicationContext()`方法，传入当前的`ApplicationContext`。

   6. **BeanPostProcessor（前置处理）**:
      `BeanPostProcessor`的`postProcessBeforeInitialization()`方法将被调用，可以对Bean进行额外处理。

   7. **InitializingBean**:
      如果Bean实现了`InitializingBean`接口，Spring将调用其`afterPropertiesSet()`方法。

   8. **自定义初始化方法**:
      如果Bean定义了自定义的初始化方法（通过`init-method`属性指定），该方法将被调用。

   9. **BeanPostProcessor（后置处理）**:
      `BeanPostProcessor`的`postProcessAfterInitialization()`方法将被调用。

   10. **Bean准备就绪**:
       此时，Bean已经准备好可以被应用程序使用了。

   11. **DisposableBean**:
       当容器关闭时，如果Bean实现了`DisposableBean`接口，Spring将调用其`destroy()`方法。

   12. **自定义销毁方法**:
       如果Bean定义了自定义的销毁方法（通过`destroy-method`属性指定），该方法将被调用。

   13. **销毁**:
       Bean被销毁，不再可用。

   以下是一个示例，演示了如何在Bean中使用部分生命周期方法：

   ```java
   import org.springframework.beans.factory.DisposableBean;
   import org.springframework.beans.factory.InitializingBean;
   
   public class MyBean implements InitializingBean, DisposableBean {
       private String property;
   
       public void setProperty(String property) {
           this.property = property;
       }
   
       @Override
       public void afterPropertiesSet() throws Exception {
           // 自定义初始化操作
           System.out.println("InitializingBean.afterPropertiesSet()");
       }
   
       public void customInit() {
           // 自定义初始化方法
           System.out.println("customInit()");
       }
   
       @Override
       public void destroy() throws Exception {
           // 自定义销毁操作
           System.out.println("DisposableBean.destroy()");
       }
   
       public void customDestroy() {
           // 自定义销毁方法
           System.out.println("customDestroy()");
       }
   }
   ```

   在Spring配置文件中配置init-method和destroy-method：

   ```xml
   <bean id="myBean" class="com.example.MyBean" init-method="customInit" destroy-method="customDestroy">
       <property name="property" value="Some Value"/>
   </bean>
   ```

   在Spring容器创建和销毁`myBean`时，会按照生命周期顺序调用相应的方法。



