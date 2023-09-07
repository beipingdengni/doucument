Spring test

#### 单元测试：

> spring
>
> @TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)这里的事务关联到配置文件中的事务控制器（transactionManager = "transactionManager"），同时指定自动回滚（defaultRollback = true）。这样做操作的数据才不会污染数据库！



#### 需要的依赖包：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>RELEASE</version>
</dependency>
<dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId> 
    <scope>test</scope>  
</dependency>

```

#### 创建基础类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations={"classpath:applicationContext.xml"})
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true))
public abstract class SpringBaseTest {

    public SpringBaseTest(){
    }
  
    @Before
    public void setUp() {
        
    }
}
```

#### 单元测试类

```java

public class SendMsgServiceImplTest extends SpringBaseTest {

    @InjectMocks
    private SendMsgService sendMsgService;

    @Autowired
    private NotifyConfig notifyConfig;
  
    @Autowired
    private ShortMsgFeignClient shortMsgFeignClient;
    
      @Test
    public void asyncSend() {
      	// 直接真是调用
        SmsMsgRequestDto requestDto = new SmsMsgRequestDto();
        requestDto.setNationalTelCode("86");
        requestDto.setCommercialId(810111734L);
        sendMsgService.asyncSend(requestDto);
    }
 }
```

