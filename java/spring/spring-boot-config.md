###  文件配置

spring boot 相关配置

```

eureka.instance.prefer-ip-address为true或者false
// 注册eureka 显示名称
eureka.instance.instance-id=
		${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
		:@project.version@		

默认如下：${spring.cloud.client.hostname}:${spring.application.name}
		:${spring.application.instance_id:${server.port}}


hystix配置
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

 ``` java
 @ConfigurationProperties(prefix = "web.config")

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

``` js

把所有配置全都打在一个jar包里，显然不是最好的做法，更常见的做法是把配置文件放在jar包外面，可以在需要时，不动java代码的前提下修改配置，spring-boot会按以下顺序加载配置文件application.properties或application.yml：

4.1 先查找jar文件同级目录下的 ./config 子目录 有无配置文件 （外置)

4.2 再查找jar同级目录 有无配置文件（外置)

4.3 再查找config这个package下有无配置文件（内置)

4.4 最后才是查找classpath 下有无配置文件（内置)

```

### 配置不同的环境

> 不同环境的配置文件以"application-环境名.yml"命名
>
> 如果有二个环境dev、prod，项目工程中有上述二个文件即可

``` js
 application.properties(yml)

 application-dev.yml
 application-prod.yml

 使用prod 

 在  application.yml 中 添加
 spring.profile.active=prod

 启动时候:  启动shell脚本中，动态指定，例如 java -jar spring-boot-SNAPSHOT.jar --spring.profiles.active=prod

```

###  bean中应用application.yml 中的值

``` js
  在application.properties

  com.didispace.blog.name=程序猿DD
  com.didispace.blog.title=Spring Boot教程
  // 连续的应用
  com.didispace.blog.desc=${com.didispace.blog.name}正在努力写《${com.didispace.blog.title}》

```

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