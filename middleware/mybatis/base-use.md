

## jar使用

```
  <!-- MyBatis -->
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.4</version>
  </dependency>
```



```
String resource = "mybatis-config.xml";
// 加载MyBatis的主配置文件
InputStream inputStream = Resources.getResourceAsStream(resource);
// 通过构建器（SqlSessionFactoryBuilder）构建一个SqlSessionFactory工厂对象
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream)


SqlSession  sqlSession= factory.openSession()

```

