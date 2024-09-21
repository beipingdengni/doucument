## 官方地址

官方网站：https://grpc.io/

java github： https://github.com/grpc/grpc-java



## 实现GRPC通讯

src/main/proto/helloworld.proto

```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_package = "com.tbp.grpc.hello";
option java_outer_classname = "HelloServiceProto";

// The greeting service definition.
service HelloService {
    // Sends a greeting
    rpc SayHello (HelloRequest) returns (HelloReply) {
    }
}

// The request message containing the user's name.
message HelloRequest {
    string name = 1;
}

// The response message containing the greetings
message HelloReply {
    string message = 1;
}
```

### 服务端

```java
import com.tbp.grpc.hello.HelloReply;
import com.tbp.grpc.hello.HelloRequest;
import com.tbp.grpc.hello.HelloServiceGrpc;
import io.grpc.Server;
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;
import java.io.IOException;

public class HelloServer {
    // 暴露的接口
    private int port = 50051;
    private Server server;
    // 启动
    private void start() throws IOException {
        // 启动服务端口
        server = ServerBuilder.forPort(port).addService(new HelloServiceImpl()).build().start();
        System.out.println("Server started, listening on " + port);
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.err.println("*** shutting down gRPC server since JVM is shutting down");
            HelloServer.this.stop();
            System.err.println("*** server shut down");
        }));
    }
    // 停止服务
    private void stop() {
        if (server != null) {
            server.shutdown();
        }
    }
    // block 一直到退出程序
    private void blockUntilShutdown() throws InterruptedException {
        if (server != null) {
            server.awaitTermination();
        }
    }
    // 实现 定义一个实现服务接口的类
    private class HelloServiceImpl extends HelloServiceGrpc.HelloServiceImplBase {
        @Override
        public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
            // 接收消息
            System.out.println("接收Message from gRPC-Client:" + req.getName());
            // 回复消息
            HelloReply reply = HelloReply.newBuilder().setMessage(("服务端返回的数据 ，Hello " + req.getName())).build();
            responseObserver.onNext(reply);
            responseObserver.onCompleted();
            System.out.println("Message Response:" + reply.getMessage());
        }
    }
    // 执行启动服务
    public static void main(String[] args) throws IOException, InterruptedException {
        final HelloServer server = new HelloServer();
        server.start();
        server.blockUntilShutdown();
    }
}
```

### 客户端

```java
import com.tbp.grpc.hello.HelloReply;
import com.tbp.grpc.hello.HelloRequest;
import com.tbp.grpc.hello.HelloServiceGrpc;
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.StatusRuntimeException;
import java.util.concurrent.TimeUnit;

public class HelloClient {
    private final ManagedChannel channel;
    private final HelloServiceGrpc.HelloServiceBlockingStub blockingStub;
    // 构建服务
    public HelloClient(String host, int port) {
        channel = ManagedChannelBuilder.forAddress(host, port).usePlaintext(true).build();
        blockingStub = HelloServiceGrpc.newBlockingStub(channel);
    }
    // 关闭服务
    public void shutdown() throws InterruptedException {
        channel.shutdown().awaitTermination(5, TimeUnit.SECONDS);
    }
    // 调用服务端的接口
    public void greet(String name) {
        HelloRequest request = HelloRequest.newBuilder().setName(name).build();
        HelloReply response;
        try {
            response = blockingStub.sayHello(request);
            String message = response.getMessage();
            System.out.println("返回结果：===》"+message);
        } catch (StatusRuntimeException e) {
            System.out.println("RPC failed: "+ e.getStatus());
            return;
        }
        System.out.println("Message from gRPC-Server: " + response.getMessage());
    }
    // 执行测试验证
    public static void main(String[] args) throws InterruptedException {
        HelloClient client = new HelloClient("127.0.0.1", 50051);
        try {
            String user = "world";
            if (args.length > 0) {
                user = args[0];
            }
            client.greet(user);
        } finally {
            client.shutdown();
        }
    }
}
```



## 注册中心etcd

依赖

```xml
<dependency>
    <groupId>io.etcd</groupId>
    <artifactId>etcd4j</artifactId>
    <version>3.0.14</version>
</dependency>
```

注册：

```java
client.getKVClient().put(ByteSequence.from("service_name"), ByteSequence.from("service_address"));
```

获取：

```java
String address = client.getKVClient().get(ByteSequence.from("service_name")).get().getKvs().get(0).getValue().toStringUtf8();
```

## spring boot 注册

参考：https://github.com/grpc-ecosystem/grpc-spring

### server

```
<dependency>
  <groupId>net.devh</groupId>
  <artifactId>grpc-spring-boot-starter</artifactId>
  <version>2.15.0.RELEASE</version>
</dependency>
```

注册

```yaml
spring:
  application:
    name: grpc_demo_server
  # 不启动 Tomcat 容器: server 这边只需要提供gRPC远程调用服务即可，Tomcat 这种 Web 服务用不上启动反而浪费额外资源和端口
  main:
    web-application-type: none
grpc:
  server:
    port: 9000 # gRPC 启动端口
```



```
@GrpcService
public class GrpcServerService extends GreeterGrpc.GreeterImplBase {

    @Override
    public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
        HelloReply reply = HelloReply.newBuilder().setMessage("Hello ==> " + req.getName()).build();
        responseObserver.onNext(reply);
        responseObserver.onCompleted();
    }

}
```

### client

```xml
<dependency>
  <groupId>net.devh</groupId>
  <artifactId>grpc-client-spring-boot-starter</artifactId>
  <version>2.15.0.RELEASE</version>
</dependency>
```

调用

```yaml
spring:
  application:
    name: grpc_demo_client
grpc:
  client:
    grpc-server: # 自定义服务名
      address: 'static://127.0.0.1:9000' # 调用 gRPC 的地址
      negotiation-type: plaintext # 明文传输
```

```java
@GrpcClient("grpc-server")
private GreeterGrpc.GreeterBlockingStub greeterStub;
```

