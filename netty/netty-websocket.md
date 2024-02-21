# spring gateway网关配置socket转发

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

# socket io

官方网站： https://socket.io/zh-CN/

## python（python-socketio）

 https://python-socketio.readthedocs.io/en/stable/intro.html

## netty socket.io

```xml
<dependency>
    <groupId>com.corundumstudio.socketio</groupId>
    <artifactId>netty-socketio</artifactId>
    <version>2.0.6</version>
</dependency>
```

