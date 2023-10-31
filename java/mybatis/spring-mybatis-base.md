





## spring mvc 配置

```xml
<context:component-scan base-package="com.tbp"/>

<mvc:default-servlet-handler/>

<!--Mybatis与Spring的整合-->
<!--1、配置数据源-->
<bean id="dataSource"  class="com.alibaba.druid.pool.DruidDataSource">
  <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
  <property name="url" value="jdbc:mysql://localhost:3306/demo?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai&amp;allowPublicKeyRetrieval=true"></property>
  <property name="username" value="root"></property>
  <property name="password" value="123456"></property>
  <property name="initialSize"  value="6"></property>
  <property name="maxActive"  value="20"></property>
</bean>

<!--2、配置SessionFactoryBean  mybatis-plus  -->
<!--<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">-->
<bean id="sessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource"></property>
  <property name="mapperLocations" value="classpath:mappers/*.xml"></property>
  <property name="configLocation"  value="classpath:mybatis-config.xml"></property>
  <!--分页插件,MP 3.4以后的版本需要在SqlSessionFactory中进行设置-->
  <property name="plugins">
    <array>
      <bean class="com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor">
        <property name="interceptors">
          <list>
            <bean class="com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor"></bean>
          </list>
        </property>
      </bean>
    </array>
  </property>
</bean>

<!--3、配置 Mapper扫描器-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="com.tbp.dao.mapper"></property>
</bean>

<!--4、配置 mybatis-config.xml-->
<!--定义事务管理-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"></property>
</bean>

<!--开启注解模式-->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>

<!-- 启用注解形式的任务功能 -->
<task:annotation-driven></task:annotation-driven>
```

