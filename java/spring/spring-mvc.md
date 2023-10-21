### 这个文档中放置一些关于spring mvc 文档



## 跨域

```xml
<mvc:cors>
  <mvc:mapping path="/api/goldenEye/*"
               allowed-methods="GET,POST,OPTIONS"
               allowed-headers="Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range"
               max-age="3600"
               allowed-origins="http://ge-work.jd.com,https://ge-work.jd.com,http://jdbxsaas-test.jd.com,http://demo.jr.jd.com"
               allow-credentials="true"/>
</mvc:cors>
```

### 源码分析

```java
// 初始化处理，配置缓存
	RequestMappingHandlerMapping#initCorsConfiguration

// http请求执行
  AbstractHandlerMapping#getHandler
    // 获取配置
    UrlBasedCorsConfigurationSource#getCorsConfiguration
    // 增加拦截处理跨域
    AbstractHandlerMapping#getCorsHandlerExecutionChain
    // 添加 options 请求处理
    	PreFlightHandler
    		CorsProcessor#processRequest //（默认：DefaultCorsProcessor）
    // 添加 拦截器
    	AbstractHandlerMapping.CorsInterceptor
```



## 过滤器

```java
@Bean
public CorsFilter corsFilter() {
    CorsConfiguration config = new CorsConfiguration();
    // 使用默认CORS参数
    // config.applyPermitDefaultValues()
    config.setAllowCredentials(true);
    config.addAllowedOrigin("*");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    config.setMaxAge(1800L);
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return new CorsFilter(source);
}
```

**我们需要在web.xml中配置`DelegatingFilterProxy`来代理配置的`CorsFilter`的调用，不能直接在web.xml中配置Spring MVC的CorsFilter！**

```xml
<filter>
    <filter-name>corsFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <!--设置是否调用代理目标bean上的 Filter.init 和 Filter.destroy的生命周期方法。-->
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
    <!--DelegatingFilterProxy的filter-name应该和Spring管理的Filter的beanName一致-->
    <!--如果不一致，需要配置targetBeanName参数指向Spring管理的Filter的beanName-->
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>corsFilter</param-value>
    </init-param>
```



### 全局异常

@ControllerAdvice



@ResponseBody
@ExceptionHandler(value = Exception.class)



### 使用RequestBodyAdvice 和 ResponseBodyAdvice增强器

RequestBodyAdvice 和 ResponseBodyAdvice都需要搭配`@RestControllerAdvice`或`@ControllerAdvice`使
