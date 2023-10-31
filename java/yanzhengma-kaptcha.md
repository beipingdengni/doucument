## pom xml

```xml
<dependency>
  <groupId>pro.fessional</groupId>
  <artifactId>kaptcha</artifactId>
  <version>2.3.3</version>
</dependency>
```



```xml
<!-- 配置Kaptcha -->
<bean id="kaptchaProducer" class="com.google.code.kaptcha.impl.DefaultKaptcha">
  <property name="config">
    <bean class="com.google.code.kaptcha.util.Config">
      <constructor-arg>
        <props>
          <!--验证码图片不生成边框-->
          <prop key="kaptcha.border">no</prop>
          <!-- 验证码图片宽度为120像素 -->
          <prop key="kaptcha.image.width">120</prop>
          <!-- 验证码图片字体颜色为蓝色 -->
          <prop key="kaptcha.textproducer.font.color">blue</prop>
          <!-- 每个字符最大占用40像素 -->
          <prop key="kaptcha.textproducer.font.size">40</prop>
          <!-- 验证码包含4个字符 -->
          <prop key="kaptcha.textproducer.char.length">4</prop>
        </props>
      </constructor-arg>
    </bean>
  </property>
</bean>
```

