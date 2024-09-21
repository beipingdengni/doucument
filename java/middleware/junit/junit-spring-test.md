Spring boot test

#### 单元测试：

> spring
>
> @TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)这里的事务关联到配置文件中的事务控制器（transactionManager = "transactionManager"），同时指定自动回滚（defaultRollback = true）。这样做操作的数据才不会污染数据库！
>
> 



#### 需要的依赖包：

```xml
<!--spring-test测试=-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.3.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-test</artifactId>
    <version>1.5.9.RELEASE</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>

```

#### 创建基础类

```java
@RunWith(SpringRunner.class) //14.版本之前用的是SpringJUnit4ClassRunner.class
@SpringBootTest(classes = Application.class) //1.4版本之前用的是//@SpringApplicationConfiguration(classes = Application.class)
@EnableAutoConfiguration
public class SystemInfoServiceImplTest {

        @Autowired
        private ISystemInfoService systemInfoservice;

        @Test
        public void add() throws Exception {
        }

        @Test
         public void findAll() throws Exception {
         }
}
```



