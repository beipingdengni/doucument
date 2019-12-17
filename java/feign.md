## Feign 配置使用



#### 使用spring feign 配置使用

```java
public interface FeignProxyFactory {
    <T> T create(Class<T> proxyType, String baseUrl);
}

/**
* 测试
*/
public class KryFeignJaxrsProxyFactory implements FeignProxyFactory {

    @Autowired(required = false)
    private ObjectMapper objectMapper;


    @Override
    public <T> T create(Class<T> proxyType, String baseUrl) {

        T instance = Feign.builder()
                .encoder(null == objectMapper ? new JacksonEncoder() : new JacksonEncoder(objectMapper))
                .decoder(null == objectMapper ? new JacksonDecoder() : new JacksonDecoder(objectMapper))
                .logger(new Slf4jLogger())
                .logLevel(Logger.Level.FULL)
                .contract(new JAXRSContract())
                .requestInterceptor(new KryHeaderRequestInterceptor())
                .target(proxyType, baseUrl);
        return instance;
    }
}


```

#### spring-bean xml 使用

```xml
<bean id="kryFeignJaxrsProxyFactory" class="com.calm.b.common.KryFeignJaxrsProxyFactory"/>

    <bean id="goodServiceClient" factory-bean="kryFeignJaxrsProxyFactory" factory-method="create">
        <constructor-arg value="com.calm.thirdapi.good.service.GoodServiceClient"/>
        <constructor-arg value="${order.good.server.url}"/>
    </bean>
```

