网关端口：9090

```
#服务名称
spring:
  application:
    name: gateway-config
  cloud:
    gateway:
      routes:
        - id: route-socket
          uri: ws://127.0.0.1:8880
          predicates:
            - Path=/nio/info
```



后端websocket是：8080