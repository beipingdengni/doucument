

## spring mybatis



## 

```java
// 扫描包
String packageSearchPath = "com.tbp.dao"
this.mapperLocations = new PathMatchingResourcePatternResolver().getResources(packageSearchPath);

// 构建session
SqlSessionFactoryBean
// session 模版
SqlSessionTemplate
// 扫描mapping 构建bean
MapperFactoryBean
// 自动扫描
MapperScannerConfigurer
```

