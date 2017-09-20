### ctuator 通过暴露一系列的endpoints可以让开发者快速了解spring boot的各项运行指标

#### 引入包

``` java
compile('org.springframework.boot:spring-boot-starter-actuator')

// yml 文件中配置
management:
  security:
    enabled: false #关掉安全认证
  port: 1101 #管理端口调整成1101
  context-path: /admin #actuator的访问路径

// 访问 ： http://localhost:1101/admin/metrics 可查看相关内容关于jvm
```

1. actuator  
    1. Provides a hypermedia-based “discovery page” for the other endpoints.Requires Spring HATEOAS to be on the classpath
1. auditevents 
    1. Exposes audit events information for the current application.
1. autoconfig  
    1. Displays an auto-configuration report showing all auto-configuration candidates and the reason why they ‘were’ or ‘were not’ applied.
1. beans 
    1. Displays a complete list of all the Spring beans in your application.
1. configprops  
    1.   Displays a collated list of all @ConfigurationProperties.
1. dump 
    1. Performs a thread dump.
1. env
    1. Exposes properties from Spring’s ConfigurableEnvironment.
1. flyway
    1. Shows any Flyway database migrations that have been applied.
1. health
    1. Shows application health information (when the application is secure,a simple ‘status’ when accessed over an unauthenticated connection orfull message details when authenticated).
1. info 
    1. Displays arbitrary application info.
1. loggers 
    1. Shows and modifies the configuration of loggers in the application.
1. liquibase 
    1. Shows any Liquibase database migrations that have been applied.
1. metrics  
    1. hows ‘metrics’ information for the current application.
1. mappings  
    1. Displays a collated list of all @RequestMapping paths.
1. shutdown 
    1. Allows the application to be gracefully shutdown (not enabled by default).
1. trace 
    1. Displays trace information (by default the last 100 HTTP requests).