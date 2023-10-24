### 这个文档中放置一些关于spring mvc 文档





### spring AntPathMatcher

> 参考博客  https://blog.csdn.net/JokerLJG/article/details/127751174

### 接口方法

| 方法                                                         | 描述                                               |
| ------------------------------------------------------------ | -------------------------------------------------- |
| boolean isPattern(String path)                               | 判断路径是否是模式                                 |
| boolean match(String pattern, String  path)                  | 判断路径是否完全匹配                               |
| boolean matchStart(String pattern, String  path)             | 判断路径是否前缀匹配                               |
| 前缀匹配的意思：路径能与模式的前面部分匹配，但模式可能还有后面多余部分 |                                                    |
| 例如：/test能前缀匹配/test/{id}（但模式还有多余的/{id}部分未匹配） |                                                    |
| String extractPathWithinPattern(String  pattern, String path) | 得到模式匹配的部分值                               |
| 该方法只返回路径的实际模式匹配部分                           |                                                    |
| 例如：myroot/*.html  匹配 myroot/myfile.html 路径，结果为 myfile.html |                                                    |
| Map<String, String>  extractUriTemplateVariables(String pattern, String path) | 提取路径中的路径参数值                             |
| Comparator<String>  getPatternComparator(String path)        | 得到一个排序比较器，用于对匹配到的所有路径进行排序 |
| String combine(String pattern1, String  pattern2)            | 合并两个模式                                       |



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
