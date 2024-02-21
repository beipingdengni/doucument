###  文件配置

spring boot 相关配置

```properties
eureka.instance.prefer-ip-address为true或者false
# 注册eureka 显示名称
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}:@project.version@		

#默认如下：${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}

# hystix配置
hystrix.threadpool.default.coreSize: 50
hystrix.threadpool.default.maxQueueSize: 100
hystrix.threadpool.default.queueSizeRejectionThreshold: 100
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
```




>
> @ImportResource('classpath:spring-bean.xml')
> 
> 添加使用导入 spring.xml  等xml的配置入口文件
>
> @ConfigurationProperties 读取配置主文件中的文件

 ``` yaml
# @ConfigurationProperties(prefix = "web.config")
web:
  config:
      webTitle: "欢迎使用SpringBoot"
      authorName: "菩提树下的杨过"
      authorBlogUrl: "http://yjmyzz.cnblogs.com/" 

 ```

>
>  导入文件 自定义配置文件
>
>  @PropertySource（‘mysql.propertis’）
>

``` text
把所有配置全都打在一个jar包里，显然不是最好的做法，更常见的做法是把配置文件放在jar包外面，可以在需要时，不动java代码的前提下修改配置，spring-boot会按以下顺序加载配置文件application.properties或application.yml：

1 先查找jar文件同级目录下的 ./config 子目录 有无配置文件 （外置)
2 再查找jar同级目录 有无配置文件（外置)
3 再查找config这个package下有无配置文件（内置)
4 最后才是查找classpath 下有无配置文件（内置)
```

### 配置不同的环境

> 不同环境的配置文件以"application-环境名.yml"命名
>
> 如果有二个环境dev、prod，项目工程中有上述二个文件即可

``` yaml
#application.properties(yml)
#环境配置
application-dev.yml
application-prod.yml
#使用prod 
#在  application.yml 中 添加
spring.profile.active=prod
#启动时候:  启动shell脚本中，动态指定，例如 java -jar spring-boot-SNAPSHOT.jar --spring.profiles.active=prod
```

###  bean中应用application.yml 中的值

> 配置文件中，应用配置自身部分配置

``` yaml
#在application.properties
	com.didispace.blog.name=程序猿DD
  com.didispace.blog.title=Spring Boot教程
# 连续的应用
	com.didispace.blog.desc=${com.didispace.blog.name}正在努力写《${com.didispace.blog.title}》

```

配置bean

``` java
@Data
@Component
public class BlogProperties {
  @Value("${com.didispace.blog.name}")
  private String name;
  @Value("${com.didispace.blog.title}")
  private String title;
}
```

## @PropertySource分析

`@PropertySource` 是 Spring 框架中用于声明属性源的注解，它可以用来指定配置文件的位置。`@PropertySource` 注解通常与 `@Configuration` 注解一起使用，将配置文件中的属性加载到 Spring 的 `Environment` 中。Spring 的 `Environment` 抽象提供了一个配置属性的查找功能，包括从属性文件、JVM 系统属性、系统环境变量和命令行参数中查找。

`@PropertySource` 注解的底层实现涉及以下几个关键步骤：

1. **解析注解**：
   在应用的配置阶段，Spring 容器会解析带有 `@PropertySource` 注解的配置类。

2. **加载属性资源**：
   容器使用 `PropertySourceFactory` 来创建 `PropertySource` 对象，该对象封装了属性资源的内容。

3. **注册属性源**：
   创建的 `PropertySource` 对象被添加到 `Environment` 的 `PropertySources` 集合中。如果有多个 `PropertySource`，它们将按照声明的顺序进行注册，可以有优先级。

4. **属性解析**：
   当需要解析属性值时，Spring 会通过 `Environment` 检索所有的 `PropertySources` 集合，并按顺序查找第一个匹配的属性值。

下面是 `@PropertySource` 注解的一个简单用法示例：

```java
@Configuration
@PropertySource("classpath:myapp.properties")
public class AppConfig {
    // ...
}
```

在这个例子中，`myapp.properties` 文件中的属性会被加载到 Spring `Environment` 中。

底层实现的关键类和接口包括：

- `PropertySource`：封装了一个属性源的抽象，例如属性文件。
- `PropertySources`：是 `PropertySource` 对象的集合，可以包含多个属性源。
- `Environment`：提供了对属性值的搜索和解析，它聚合了所有的 `PropertySources`。
- `PropertySourcesPropertyResolver`：是用于在 `PropertySources` 中解析属性的类。

以下是 Spring 处理 `@PropertySource` 注解的简化伪代码：

```java
public void processPropertySourceAnnotations(AnnotationMetadata metadata) {
    // 解析 @PropertySource 注解
    MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(PropertySource.class.getName());
    
    // 遍历所有的 @PropertySource 注解
    for (Map<String, Object> attribute : attributes) {
        String name = (String) attribute.get("name");
        String[] locations = (String[]) attribute.get("value");
        boolean ignoreResourceNotFound = (Boolean) attribute.get("ignoreResourceNotFound");

        for (String location : locations) {
            try {
                // 加载属性资源
                Resource resource = new ClassPathResource(location);
                PropertySource<?> propertySource = new PropertiesPropertySource(name, PropertiesLoaderUtils.loadProperties(resource));
                // 注册到 Environment 中
                this.environment.getPropertySources().addLast(propertySource);

            } catch (IOException e) {
                if (!ignoreResourceNotFound) {
                    throw new IllegalStateException(e);
                }
            }
        }
    }
}
```

实际的实现会更复杂，并且会涉及到 Spring 的 Bean 生命周期、Bean 工厂后处理器以及资源加载的细节。要理解整个处理流程，可以查看 `PropertySourcesLoader`、`PropertySourcesPlaceholderConfigurer` 和相关类的源码。

## PropertySource、PropertySourcesPropertyResolver

```java
// @Autowired
// private Environment environment;
//  接口实现 EnvironmentAware

public class MyEnvironmentPostProcessor implements EnvironmentPostProcessor {
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) { 
       // 配置使用
			 PropertySource<String> customPropertySource = new MapPropertySource("myPropertySource", myPropertiesMap);
       environment.getPropertySources().addFirst(customPropertySource);
      
      // spring boot 
      // PropertySourcesLoader loader = new PropertySourcesLoader();
    }
}
```

## PropertySourcesLoader

`PropertySourcesLoader` 是 Spring 框架中用于加载属性到 `PropertySources` 中的一个工具类。这个类不像 `PropertySourcesPlaceholderConfigurer` 那样常见，因为它通常在框架内部使用，尤其是在处理 `@PropertySource` 注解时。然而，如果你需要在编程的方式中加载和操作属性源，`PropertySourcesLoader` 可以为你提供帮助。

### 使用方式

`PropertySourcesLoader` 可以用来以编程的方式加载属性文件，并将其作为 `PropertySource` 添加到 `Environment` 的 `PropertySources` 集合中。以下是使用 `PropertySourcesLoader` 的一个示例：

```java
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.MutablePropertySources;
import org.springframework.core.env.PropertySource;
import org.springframework.core.env.PropertySources;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.core.io.support.PropertySourceFactory;
import org.springframework.core.io.support.ResourcePropertySource;

public class SomeClass {
    public void loadProperties(ConfigurableEnvironment environment) throws IOException {
      	// spring 高版本 
        PropertySourcesLoader loader = new PropertySourcesLoader();
        Resource resource = new ClassPathResource("config/myapp.properties");
        PropertySource<?> propertySource = loader.load(resource, "myPropertySource", null);
        MutablePropertySources propertySources = environment.getPropertySources();
        propertySources.addLast(propertySource);
    }
}
```

在这个例子中，我们创建了一个 `PropertySourcesLoader` 的实例，并使用它来加载一个位于类路径中的 `myapp.properties` 文件。加载后的属性源被命名为 `"myPropertySource"` 并添加到 `Environment` 的 `PropertySources` 集合的末尾。

### 底层原理

`PropertySourcesLoader` 的主要职责是加载属性资源并将其包装为 `PropertySource` 对象。它通常通过以下步骤工作：

1. **加载资源**：使用传入的 `Resource` 对象（例如，来自文件、类路径、URL等）来加载属性。
2. **创建 PropertySource**：将加载的属性封装到 `PropertySource` 对象中。如果指定了 `name` 参数，则使用该名称；否则，通常使用资源的描述（例如文件名）作为 `PropertySource` 的名称。
3. **添加到 PropertySources**：最后，将创建的 `PropertySource` 对象添加到 `PropertySources` 集合中，可以指定添加到集合的位置（如 `addFirst`、`addLast`、`addBefore` 或 `addAfter`）。

注意，`PropertySourcesLoader` 类在 Spring Framework 5.1 中已被标记为 `@Deprecated`，这意味着在未来的版本中可能会被移除或替换。在实际开发中，你更可能使用 `@PropertySource` 注解或直接操作 `ConfigurableEnvironment` 来管理属性源。

## PropertySourcesPlaceholderConfigurer

`PropertySourcesPlaceholderConfigurer` 的主要功能如下：

- **解析占位符**：它会解析 bean 定义中的 `${...}` 占位符，并用 `PropertySources` 中的属性值替换它们。
- **使用 `Environment`**：它能够利用当前的 `Environment` 中的 `PropertySources` 来解析属性。
- **自定义属性源**：你可以通过编程方式添加自定义的 `PropertySource` 对象，以供解析占位符时使用。