## maven包

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <!-- <version>4.1.82.Final</version>-->
    <version>5.0.0.Alpha2</version>
</dependency>
```



## 以下案例使用：netty创建websocket

## 创建server服务端

```java
public class WebSocketNettyServer {
 
    private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
 
    public static void main(String[] args) {
 
        // 创建两个线程池
        // 主线程池
        NioEventLoopGroup mainGrp = new NioEventLoopGroup();
        // 从线程池
        NioEventLoopGroup subGrp = new NioEventLoopGroup();
 
        try {
            // 创建Netty服务器启动对象
            ServerBootstrap serverBootstrap = new ServerBootstrap();
 
            // 初始化服务器启动对象
            serverBootstrap
                    // 指定使用上面创建的两个线程池
                    .group(mainGrp, subGrp)
                    // 指定Netty通道类型
                    .channel(NioServerSocketChannel.class)
                    // 指定通道初始化器用来加载当Channel收到事件消息后，
                    // 如何进行业务处理
                    .childHandler(new WebSocketChannelInitializer());
 
            // 绑定服务器端口，以同步的方式启动服务器
            ChannelFuture future = serverBootstrap.bind(8880).sync();
            // 等待服务器关闭
            future.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // 优雅关闭服务器
            mainGrp.shutdownGracefully();
            subGrp.shutdownGracefully();
        }
    }
}
```

### 创建ChannelInitializer

```java
public class WebSocketChannelInitializer extends ChannelInitializer<SocketChannel>{
    // 初始化通道
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        // 获取管道，将一个一个的ChannelHandler添加到管道中
        ChannelPipeline pipeline = ch.pipeline();
 
        // 添加一个http的编解码器
        pipeline.addLast(new HttpServerCodec());
        // 添加一个用于支持大数据流的支持
        pipeline.addLast(new ChunkedWriteHandler());
        // 添加一个聚合器，这个聚合器主要是将HttpMessage聚合成FullHttpRequest/Response
        pipeline.addLast(new HttpObjectAggregator(1024 * 1024));
 
        // 需要指定接收请求的路由
        // 必须使用以 /nio/info 后缀结尾的url才能访问
        pipeline.addLast(new WebSocketServerProtocolHandler("/nio/info"));
        // 添加自定义的Handler
        pipeline.addLast(new MyScocketHandler());
    }
}
```

### 自定义Handler

```java
public class MyScocketHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
 
    /**用来保存所有的客户端连接*/
    public static ChannelGroup clients = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    /**时间格式化*/
    private SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
 
    /**
     * 当有新的客户端连接服务器之后，会自动调用这个方法
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        // 将新的通道加入到clients
        clients.add(ctx.channel());
    }
 
    @Override
    protected void messageReceived(ChannelHandlerContext channelHandlerContext, TextWebSocketFrame msg) throws Exception {
        StringBuilder resMsg = new StringBuilder();
        // 获取客户端发送过来的文本消息
        String text = msg.text();
        System.out.println("获取客户端发送内容:" + text);
        //发送所有客户端
        /**
        for (Channel client : clients) {
            System.out.println("获取客户端:" + client.id());
            // 将消息发送到所有的客户端
         resMsg.append(text).append(",").append(channelHandlerContext.channel().id()).append(",").append(sdf.format(new Date()));
            client.writeAndFlush(new TextWebSocketFrame(resMsg.toString()));
        }
         */
        //发送指定客户端
        resMsg.append(text).append(",").append(channelHandlerContext.channel().id()).append(",").append(sdf.format(new Date()));
        channelHandlerContext.channel().writeAndFlush(new TextWebSocketFrame(resMsg.toString()));
 
    }
}
```

## 创建client客户端

### 创建连接

```java
public class WebSocketNettyClient {
 
    public static void main(String[] args)  {
 
        EventLoopGroup group = new NioEventLoopGroup();
        final ClientHandler clientHandler =new ClientHandler();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group).channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .option(ChannelOption.SO_KEEPALIVE,true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            // 添加一个http的编解码器
                            pipeline.addLast(new HttpClientCodec());
                            // 添加一个用于支持大数据流的支持
                            pipeline.addLast(new ChunkedWriteHandler());
                            // 添加一个聚合器，这个聚合器主要是将HttpMessage聚合成FullHttpRequest/Response
                            pipeline.addLast(new HttpObjectAggregator(1024 * 1024));
 
                            pipeline.addLast(clientHandler);
 
                        }
                    });
 
            URI websocketURI = new URI("ws://localhost:8880/nio/info");
            HttpHeaders httpHeaders = new DefaultHttpHeaders();
            //进行握手
            WebSocketClientHandshaker handshaker = WebSocketClientHandshakerFactory.newHandshaker(websocketURI, WebSocketVersion.V13, (String)null, true,httpHeaders);
 
            final Channel channel=bootstrap.connect(websocketURI.getHost(),websocketURI.getPort()).sync().channel();
            clientHandler.setHandshaker(handshaker);
            handshaker.handshake(channel);
            //阻塞等待是否握手成功
            clientHandler.handshakeFuture().sync();
 
            //发送消息
            JSONObject userInfo = new JSONObject();
            userInfo.put("userId","u1001");
            userInfo.put("userName","月夜烛峰");
            channel.writeAndFlush(new TextWebSocketFrame(userInfo.toString()));
 
            // 等待连接被关闭
            channel.closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
 
            group.shutdownGracefully();
        }
 
    }
}
```

### 客户端逻辑handler

```java
public class ClientHandler extends SimpleChannelInboundHandler<Object> {
 
    private WebSocketClientHandshaker handshaker;
    ChannelPromise handshakeFuture;
 
    /**
     * 当客户端主动链接服务端的链接后，调用此方法
     *
     * @param channelHandlerContext ChannelHandlerContext
     */
    @Override
    public void channelActive(ChannelHandlerContext channelHandlerContext) {
        System.out.println("客户端Active .....");
        handlerAdded(channelHandlerContext);
    }
 
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        System.err.println("exceptionCaught:异常信息:" + cause.getMessage());
        ctx.close();
    }
 
    public void setHandshaker(WebSocketClientHandshaker handshaker) {
        this.handshaker = handshaker;
    }
 
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        this.handshakeFuture = ctx.newPromise();
    }
 
    public ChannelFuture handshakeFuture() {
        return this.handshakeFuture;
    }
 
    @Override
    protected void messageReceived(ChannelHandlerContext ctx, Object o) throws Exception {
        // 握手协议返回，设置结束握手
        if (!this.handshaker.isHandshakeComplete()) {
            FullHttpResponse response = (FullHttpResponse) o;
            this.handshaker.finishHandshake(ctx.channel(), response);
            this.handshakeFuture.setSuccess();
            System.out.println("握手成功::messageReceived HandshakeComplete...");
            return;
        } else if (o instanceof TextWebSocketFrame) {
            TextWebSocketFrame textFrame = (TextWebSocketFrame) o;
            System.out.println("接收消息::messageReceived textFrame: " + textFrame.text());
        } else if (o instanceof CloseWebSocketFrame) {
            System.out.println("关闭链接::messageReceived CloseWebSocketFrame");
        }
    }
}
```

