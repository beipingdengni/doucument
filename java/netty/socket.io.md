

## POM

服务端

```xml
<dependency>
  <groupId>com.corundumstudio.socketio</groupId>
  <artifactId>netty-socketio</artifactId>
  <version>1.7.18</version>
</dependency>

<dependency>
  <groupId>io.netty</groupId>
  <artifactId>netty-transport-native-kqueue</artifactId>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>io.netty</groupId>
  <artifactId>netty-transport-native-epoll</artifactId>
  <scope>provided</scope>
</dependency>
```

服务配置生效

```java
import com.corundumstudio.socketio.SocketConfig;
import com.corundumstudio.socketio.SocketIOServer;

@Configuration
@Slf4j
public class SocketIOServerConfig implements InitializingBean, DisposableBean {
 
    @Bean
    public SocketIOServer socketIOServer() {
        SocketConfig socketConfig = new SocketConfig();
        socketConfig.setTcpNoDelay(true);
        socketConfig.setAcceptBackLog(65535);
        socketConfig.setReuseAddress(true);
       // 配置启动
        Configuration config = new com.corundumstudio.socketio.Configuration();
        config.setSocketConfig(socketConfig);
        if (StringUtils.isEmpty(properties.getHost())) {
            IpUtil.ForgeInetAddress inetAddress = IpUtil.getIpAddress();
            config.setHostname(inetAddress == null ? "0.0.0.0" : inetAddress.getHostAddress());
            log.info("socket.io server IpUtil.getIpAddress.getHostAddress :{}", IpUtil.getIpAddress().getHostAddress());
        } else {
            config.setHostname('服务本地的IP');
            log.info("socket.io server ip:{}", config.getHostname());
        }
        config.setPort("8080");
        config.setBossThreads(1);
        config.setAllowCustomRequests(false);
        config.setTransports(Transport.POLLING, Transport.WEBSOCKET); // 轮训、或者 websocket
        config.setUpgradeTimeout(10); //协议升级超时时间(毫秒)，默认10秒，HTTP握手升级为ws协议超时时间
        config.setPingTimeout(60); // Ping消息超时时间(毫秒)，默认60秒，这个时间间隔内没有接收到心跳消息就会发送超时事件
        config.setPingInterval(25); //Ping消息间隔(毫秒)，默认25秒。客户端向服务器发送一条心跳消息间隔
        config.setAuthorizationListener('自定义实现参数获取鉴权');
        config.setUseLinuxNativeEpoll(''); //是否使用liunx 的epoll，需要引入对应的jar包
        config.setMaxHttpContentLength('1024*10'); //设置HTTP交互最大内容长度
      	config.setMaxFramePayloadLength('1024');//设置最大每帧处理数据的长度，防止他人利用大数据来攻击服务器
        return new SocketIOServer(config);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        socketIOServer().start();
    }

    @Override
    public void destroy() throws Exception {
        socketIOServer().stop();
    }
}
// 交互处理数据
socketIOServer.addEventListener('msg_up', String.class, (client, data, ackSender) -> {
	// 上行消息
  JSONObject updata = JSON.parseObject(data)
  
  // 发送业务数据下行
  client.sendEvent("msg_down", JsonUtil.write(obj)); // 下行
  
  // ack 数据下行
  if (ackSender.isAckRequested()) {
    ackSender.sendAckData("ack下行的消息");
  }
});
```

客户端

```xml
<dependency>
  <groupId>io.socket</groupId>
  <artifactId>socket.io-client</artifactId>
  <version>1.0.0</version>
</dependency>
```

配置启动

```java
// 服务端socket.io连接通信地址
String token = getToken(tenantId, productLine.name(), userId);
String url = "http://" + ip + ":" + port + "?token=" + token + "&clientType=" + clientType;

// 创建连接
IO.Options options = new IO.Options();
options.transports = new String[]{"websocket"};
options.reconnectionAttempts = 2;
// 失败重连的时间间隔
options.reconnectionDelay = 1000;
// 连接超时时间(ms)
options.timeout = 500;
// userId: 唯一标识 传给服务端存储
final Socket socket = IO.socket(url, options);
socket.connect();
socket.on(Socket.EVENT_CONNECT, args1 -> System.out.println("连接服务器成功"));
socket.on(Socket.EVENT_DISCONNECT, args1 -> System.out.println("在其他地点登录了"));
Thread.sleep(3000);
// 消息发送线程
new Thread(() -> {
  try {
    while (true) {
      Thread.sleep(5000);
      WsMessage message = buildMsg();
      socket.emit("msg_up", JSON.toJSONString(message));
    }
  } catch (Exception e) {}
}).start();

// 监听消息接收
socket.on("msg_down", new Emitter.Listener() {
  @Override
  public void call(Object... objects) {
    log.info("receive msg:{}", objects[0].toString());
    socket.emit("cmd_up", JSONObject.toJSONString(obj),(Ack) args -> {
      // 服务端口，ack 确定后下行的消息
    });
  }
});

socket.on("cmd_down", new Emitter.Listener() {
  @Override
  public void call(Object... objects) {
    log.info("receive cmd:{}", objects[0].toString());
  }
});

socket.on("msg_down", new Emitter.Listener() {
  @Override
  public void call(Object... objects) {
    log.info("receive msg:{}", objects[0].toString());
    socket.emit("cmd_up", JSONObject.toJSONString(obj));
  }
});
```

### 房间

```java
// 进入一个房间
socket.join('room'); 
// 离开一个房间
socket.leave('room');
//前端触发订阅/退订事件
socket.emit('subscribe',{"room" : "room_name"};
socket.emit('unsubscribe',{"room" : "room_name"};

//后台处理订阅/退订事件
socket.on('subscribe', function(data) { 
    socket.join(data.room);
})
socket.on('unsubscribe', function(data) { 
    socket.leave(data.room);
})
```



## 协议

服务端：

com.corundumstudio.socketio.handler.PacketListener#onPacket

PacketType

```java
public enum PacketType {
  	// 处理连接心跳
    OPEN(0), CLOSE(1), PING(2), PONG(3), MESSAGE(4), UPGRADE(5), NOOP(6),
  	// 传递参数内部，业务上 
    CONNECT(0, true), DISCONNECT(1, true), EVENT(2, true), ACK(3, true), ERROR(4, true), BINARY_EVENT(5, true), BINARY_ACK(6, true);
	
  	PacketType(int value) {
        this(value, false);
    }

    PacketType(int value, boolean inner) {
        this.value = value;
        this.inner = inner;
    }
}
```



### ACK

发送

```javascript
var userName = 'user' + Math.floor((Math.random()*1000)+1);  
function sendMessage() {
  var message = $('#msg').val();
  $('#msg').val('');
  var jsonObject = {userName: userName,message: message};
  //发送需要ack响应的消息
  socket.emit('ackevent1', jsonObject, function(arg1, arg2) {
    alert("从服务器发来的确认消息1:" + arg1 + ", 确认消息2:" + arg2);
  });
}
```

服务接收

```java
server.addEventListener("ackevent1", ChatObject.class, new DataListener<ChatObject>() {
  @Override
  public void onData(final SocketIOClient client, ChatObject data, final AckRequest ackRequest) {
    // 检查是否是客户端发来的ack 请求确认消息
    if (ackRequest.isAckRequested()) {
      // 发送ack响应消息给客户端
      ackRequest.sendAckData("服务器响应ack响应消息1", "服务器响应ack响应消息2");
    }
  }
});
```

