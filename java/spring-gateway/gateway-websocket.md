## gateway网关端口：9090

### 配置如下：

```yaml
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

### 后端websock提供服务为：8880

### 前端连接如下：

```js
var socket;

//判断当前浏览器是否支持websocket
if (window.WebSocket) {
  //go on
  socket = new WebSocket("ws://localhost:9990/nio/info");
  //相当于channelReado, ev 收到服务器端回送的消息
  socket.onmessage = function (msg) {
    appendInfo(msg.data, "socket_keep_alive_table");
  }
  //相当于连接开启(感知到连接开启)
  socket.onopen = function (ev) {
    console.log("websocket 已打开");
  }
  //相当于连接关闭(感知到连接关闭)
  socket.onclose = function (ev) {
    console.log("websocket 已关闭");
  }
} else {
  alert("当前浏览器不支持websocket")
}
 
//发送消息到服务器
function send(message) {
  if (!window.socket) { //先判断socket是否创建
    return;
  }
  if (socket.readyState == WebSocket.OPEN) {
    //通过socket 发送消息
    socket.send(message)
  } else {
    alert("连接没有开启");
  }
}
```

