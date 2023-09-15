# eureka

## 基础版本使用

```
 <dependency>
  <groupId>com.netflix.eureka</groupId>
  <artifactId>eureka-client</artifactId>
  <version>1.1.16</version>
 </dependency>
```

## 自动构建war部署

```
git clone https://github.com/Netflix/eureka.git
cd eureka && ./gradlew clean build

Eureka Server WAR archive (./eureka-server/build/libs/eureka-server-XXX.war ) // http://localhost:8080/eureka 
Eureka Client (./eureka-client/build/libs/eureka-client-XXX.jar )
```

## 服务端-注册中心

接口API文档： https://github.com/Netflix/eureka/wiki/Eureka-REST-operations

```

```



## 客户端使用

### 配置：eureka-client.properties

#### jvm参数配置： *-Deureka.environment* = prod

#多环境 eureka-clien-{dev,prod}t.properties

```properties
eureka.numberRegistrySyncRetries=0
eureka.enableSelfPreservation=false #关闭自我保护机制，保证不可用服务及时踢出
eureka.renewalPercentThreshold=[0.0, 1.0] # 默认，15%

以下是必须配置
eureka.name=app-user-client #应用名称
eureka.port=8080 # 端口
eureka.serviceUrls=http://localhost:8080/eureka  #服务地址
eureka.vipAddress=  #虚拟IP

```

#### 其他配置，可以参考一下的两个类属性

> 实例类：https://github.com/Netflix/eureka/blob/master/eureka-client/src/main/java/com/netflix/appinfo/EurekaInstanceConfig.java
>
> 客户端：https://github.com/Netflix/eureka/blob/master/eureka-client/src/main/java/com/netflix/discovery/EurekaClientConfig.java
>
> 客户端：https://github.com/Netflix/eureka/blob/master/eureka-core/src/main/java/com/netflix/eureka/DefaultEurekaServerConfig.java#L221

#### 基础说明

```
Register：注册发生在第一次心跳时（30 秒后），查看可用实例：http://169.254.169.254/latest/metadata
Renew： Eureka客户端需要通过每30秒发送一次心跳来更新租约。更新通知 Eureka 服务器该实例仍然存活。如果服务器在 90 秒内没有看到更新，则会从其注册表中删除该实例。建议不要更改更新间隔，因为服务器使用该信息来确定客户端到服务器的通信是否存在广泛的问题
Fetch Registry：eureka客户端从服务器获取注册表信息并缓存在本地。之后，客户使用该信息来寻找其他服务。通过获取上一个提取周期与当前提取周期之间的增量更新，定期（每 30 秒）更新此信息。增量信息在服务器中保存的时间更长（大约 3 分钟），因此增量获取可能会再次返回相同的实例。Eureka客户端自动处理重复信息。获取增量后，Eureka 客户端通过比较服务器返回的实例计数来与服务器协调信息，如果由于某种原因信息不匹配，则重新获取整个注册表信息。Eureka 服务器缓存增量、整个注册表和每个应用程序的压缩有效负载以及相同的未压缩信息。有效负载还支持 JSON/XML 格式。Eureka 客户端使用 jersey apache 客户端获取压缩 JSON 格式的信息
Cancel：Eureka 客户端在关闭时向 Eureka 服务器发送取消请求。这将从服务器的实例注册表中删除该实例，从而有效地使该实例脱离流量。
Time Lag： Eureka 客户端的所有操作可能需要一些时间才能反映在 Eureka 服务器中，然后反映在其他 Eureka 客户端中。这是因为eureka服务器上的有效负载缓存会定期刷新以反映新信息。Eureka 客户端还会定期获取增量。因此，更改可能需要最多2分钟才能传播到所有 Eureka 客户端
```



### java 代码

```java
// Pre release 1.1.153, 如下初始化：Eureka Client
DiscoveryManager.getInstance().initComponent(new CloudInstanceConfig(),new DefaultEurekaClientConfig());
// 其他数据中心，使用以下方式
DiscoveryManager.getInstance().initComponent(new MyDataCenterInstanceConfig(),new DefaultEurekaClientConfig());

// 当前应用状态
ApplicationInfoManager.getInstance().setInstanceStatus(InstanceStatus.UP)

// 取消，Eureka 客户端在关闭时向 Eureka 服务器发送取消请求。这将从服务器的实例注册表中删除该实例，从而有效地使该实例脱离流量。
DiscoveryManager.getInstance().shutdownComponent()
  
// 客户端的配置， virtualHostname Eureka 服务端主页中 application 选项下的服务名称，表示是否为 https 方式
// InstanceInfo EurekaClient.getNextServerFromEureka(String virtualHostname, boolean secure)
InstanceInfo nextServerInfo = DiscoveryManager.getInstance().getDiscoveryClient().getNextServerFromEureka(virtualHostname, false);


//自定义配置
String myValue = nextServerInfo.getMetadata().get("myKey");



Socket s = new Socket();
int serverPort = nextServerInfo.getPort();
try {
  // 发起HTTP使用
  s.connect(new InetSocketAddress(nextServerInfo.getHostName(),serverPort));
} catch (IOException e) {
  System.err.println("Could not connect to the server :"+ nextServerInfo.getHostName() + " at port " + serverPort);
}

```

# spring cloud eureka 使用如下 

## 注册服务中心

### properties

```properties
# 默认端口
eureka.client.serviceUrl.defaultZone=http://localhost:8001/eureka/
# 配置说明
eureka.client.register-with-eureka= #表示是否将自己注册到Eureka Server，默认是true。
eureka.client.fetch-registry= #表示是否从Eureka Server获取注册信息，默认为true。
eureka.client.serviceUrl.defaultZone= #这个是设置与Eureka Server交互的地址，客户端的查询服务和注册服务都需要依赖这个地址

#注册中心
#@EnableEurekaServer
```

### Pom xml

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-openfeign</artifactId>
	</dependency>
</dependencies>
```



## 消费注册中心

### properties

```properties
eureka.client.serviceUrl.defaultZone= #这个是设置与Eureka Server交互的地址，客户端的查询服务和注册服务都需要依赖这个地址
# 内外IP：${spring.cloud.client.ip-address}${server.port}
eureka.instance.prefer-ip-address=true #注册到注册中心时将决定是否采用ip来注册, 如果为true
#配置instanceId值如下：
#${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port} # IdUtils 获取唯一值
eureka.instance.hostname= ${spring.cloud.client.ipAddress} # host 也使用
#关闭自我保护机制，保证不可用服务及时踢出
eureka.server.enable-self-preservation=false

# @EnableDiscoveryClient
# @EnableFeignClients
```

### pom xml

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

