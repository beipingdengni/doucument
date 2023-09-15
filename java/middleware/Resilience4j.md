# Resilience4j

>  断路器、重试、限流器、隔离、线程池隔离和限时器



JAR 引入

```groovy
repositories {
    jCenter()
}

dependencies {
  	// spring boot2
  	compile "io.github.resilience4j:resilience4j-spring-boot2:${resilience4jVersion}"
    // spring cloud2
    compile "io.github.resilience4j:resilience4j-spring-cloud2:${resilience4jVersion}"
    compile('org.springframework.boot:spring-boot-starter-actuator')
    compile('org.springframework.boot:spring-boot-starter-aop')
    compile('org.springframework.cloud:spring-cloud-starter-config')  
}
```



```xml
<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```



```properties
management.endpoints.web.exposure.include=*
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
management.endpoint.metrics.enabled=true
```

