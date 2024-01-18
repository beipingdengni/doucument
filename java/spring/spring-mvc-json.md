

## 请求和返回时间格式控制

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```

### spring自带jackson注解

| 注解                  | 作用                         |
| --------------------- | ---------------------------- |
| @JsonIgnoreProperties | 批量设置转 JSON 时忽略的属性 |
| @JsonIgnore           | 转 JSON 时忽略当前属性       |
| @JsonProperty         | 修改转换后的 JSON 的属性名   |
| @JsonFormat           | 转 JSON 时格式化属性的值     |

```java
@Data
public class User {
  // 默认走，yaml全局配置文件格式
  private Date birthday;
  // 格式化
	@JsonFormat(pattern = "yyyy年MM月dd日")
	private LocalDateTime localDateTimeFormat;
  
  @JsonSerialize(using = CustomDateSerializer.class) 
  @JsonDeserialize(using=CustomDateSerializer.class)
	private Date createTime;
}
```

自定义时间格式的序列化和反序列化【JsonSerializer、JsonDeserializer】

> @JsonSerialize
>
> @JsonDeserialize

```java
public class CustomDateSerializer extends JsonSerializer<Date> {
    @Override
    public void serialize(Date value, JsonGenerator jsonGenerator, SerializerProvider provider) throws IOException,
            JsonProcessingException {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        jsonGenerator.writeString(sdf.format(value));
    }
}
```



## Spring mvc

```xml
<mvc:annotation-driven>  
  <mvc:message-converters>  
    <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">  
      <property name="objectMapper" ref="customObjectMapper"></property>  
    </bean>  
  </mvc:message-converters>  
</mvc:annotation-driven>  
<bean id="customObjectMapper" class="com.tbp.common.custom.CustomObjectMapper"></bean>  
```



## spring boot 自定义转化器json

#### WebMvcConfigurationSupport

```java
@Configuration
@Slf4j
public class WebMvcConfig extends WebMvcConfigurationSupport {
    /**
     * 扩展消息转换器
     */
    @Override
    protected void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        // 创建消息转换器对象
        MappingJackson2HttpMessageConverter messageConverter = new MappingJackson2HttpMessageConverter();
        // 设置对象转换器
        messageConverter.setObjectMapper(new CustomObjectMapper());
        // 添加到mvc框架消息转换器中，优先使用自定义转换器
        converters.add(0, messageConverter);
    }
}
```

#### JacksonObjectMapper

```java
/**
 * 对象映射器:基于jackson将Java对象转为json，或者将json转为Java对象
 * 将JSON解析为Java对象的过程称为 [从JSON反序列化Java对象]
 * 从Java对象生成JSON的过程称为 [序列化Java对象到JSON]
 */
public class CustomObjectMapper extends ObjectMapper {
    private static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    private static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    private static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";
    public JacksonObjectMapper() {
        super();
        // 收到未知属性时不报异常
        this.configure(FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 统一返回数据的输出风格 转为蛇形命名法
        this.setPropertyNamingStrategy(new PropertyNamingStrategies.SnakeCaseStrategy());
        // 反序列化时，属性不存在的兼容处理
        this.getDeserializationConfig()
                .withoutFeatures(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

        // 格式化时间
        JavaTimeModule module = new JavaTimeModule();

        module.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)))
                .addDeserializer(Date.class, new DateDeserializers.DateDeserializer())

                // .addSerializer(BigInteger.class, ToStringSerializer.instance)
                // .addSerializer(Long.class, ToStringSerializer.instance)
                .addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                .addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                .addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)))
                .addSerializer(Date.class, new DateSerializer(false, new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT)))
                ;

        // 注册功能模块 添加自定义序列化器和反序列化器
        this.registerModule(module);
    }
}
```

