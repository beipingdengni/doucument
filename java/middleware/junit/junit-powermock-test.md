#### 单元测试：

> 在单元测试中，我们往往想去独立地去测一个类中的某个方法，但是这个类可不是独立的，它会去调用一些其它类的方法和service，这也就导致了以下两个问题：
>   
>    - 外部服务可能无法在单元测试的环境中正常工作，因为它们可能需要**访问数据库或者使用一些其它的外部系统**。
>   - 我们的测试关注点在于这个类的实现上，外部类的一些行为可能会影响到我们对本类的测试，那也就失去了我们进行单测的意义。
>   
>    为了解决这种问题，**Mockito**和**PowerMock**诞生了。这两种工具都可以用一个虚拟的对象来替换那些外部的、不容易构造的对象，便于我们进行单元测试。<b style="color:red">两者的不同点在于Mockito对于大多数的标准单测case都很适用，而PowerMock可以去解决一些更难的问题（比如静态方法、私有方法等）</b>



#### 需要的依赖包：

```xml
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <artifactId>hamcrest-core</artifactId>
            <groupId>org.hamcrest</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>RELEASE</version>
</dependency>
```

#### 创建基础类

```java
@Slf4j
@RunWith(PowerMockRunner.class)
@PowerMockRunnerDelegate(SpringJUnit4ClassRunner.class)
@PowerMockIgnore("javax.management.*")
public abstract class MockBaseTest {

    public MockBaseTest(){
    }
  
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }
}
```

#### 单元测试类

```java
@RunWith(MockitoJUnitRunner.class)
public class SendMsgServiceImplTest extends MockBaseTest {

    @InjectMocks
    private SendMsgServiceImpl sendMsgService;

    @Mock
    private NotifyConfig notifyConfig;
  
    @Mock
    private ShortMsgFeignClient shortMsgFeignClient;
    
      @Test
    public void asyncSend() {
      	// given 参数
        when(notifyConfig.getAppkey()).thenReturn("123");
      	// given 参数
        SendSmsResponse response = new SendSmsResponse();
        response.setCode(0);
        when(shortMsgFeignClient.sendSms(Matchers.anyString(), Matchers.anyString(), Matchers.anyLong(), Matchers.anyString(), Matchers.any(SendSmsRequest.class)))
                .thenReturn(response);
      	// mock 方法
        SmsMsgRequestDto requestDto = new SmsMsgRequestDto();
        requestDto.setNationalTelCode("86");
        requestDto.setCommercialId(810111734L);
        sendMsgService.asyncSend(requestDto);
    }
 }
```

