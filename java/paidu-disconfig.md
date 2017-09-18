

### 百度开源的配置文件。。

#### [githup地址](https://github.com/knightliao/disconf)

#### [中文文档说明](http://disconf.readthedocs.io/zh_CN/latest/)

#### 先观看官方文档 ，建立服务器 客服端

1.  简单入门：
    
    导入包
    ``` xml

    <dependency>
        <groupId>com.baidu.disconf</groupId>
        <artifactId>disconf-client</artifactId>
        <version>2.6.36</version>
    </dependency

    ```
2.  xml 文件中配置

    > spring-context.xm 内容

    ``` xml
        <?xml version="1.0" encoding="UTF-8"?>

        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

            <import resource="classpath:spring-disconf.xml"/>
            <import resource="classpath:spring-bean.xml"/>

        </beans>
    ```

> disconf配置文件spring-disconf.xml

``` xml

    <?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:aspectj-autoproxy proxy-target-class="true"/>

    <bean id="disconfMgrBean" class="com.baidu.disconf.client.DisconfMgrBean"
          destroy-method="destroy">
        <property name="scanPackage" value="com.example"/>
    </bean>
    <bean id="disconfMgrBean2" class="com.baidu.disconf.client.DisconfMgrBeanSecond"
          init-method="init" destroy-method="destroy">
    </bean>

    <bean id="configproperties_disconf"
          class="com.baidu.disconf.client.addons.properties.ReloadablePropertiesFactoryBean">
        <property name="locations">
            <list>
                <value>app.properties</value>
            </list>
        </property>
    </bean>

    <bean id="propertyConfigurer"
          class="com.baidu.disconf.client.addons.properties.ReloadingPropertyPlaceholderConfigurer">
        <property name="ignoreResourceNotFound" value="true"/>
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="propertiesArray">
            <list>
                <ref bean="configproperties_disconf"/>
            </list>
        </property>
    </bean>
</beans>

 ```

> spring-bean.xml中使用disconf注入的属性

```  xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

        <bean id="appConfig" class="com.example.config.AppConfig">
            <property name="checkSign" value="${checkSign}"/>
            <property name="sendEmailWhenStart" value="${sendEmailWhenStart}"/>
            <property name="env" value="${app.env}"/>
            <property name="sendEmailWhenError" value="${sendEmailWhenError}"/>
        </bean>

    </beans>

```
> AppConfig是一个演示用的配置类

``` java

    package com.example.config;
    import lombok.Data;

    @Data
    public class AppConfig {
        private String env;
        private boolean sendEmailWhenStart;
        private boolean sendEmailWhenError;
        private boolean checkSign;
    }

```