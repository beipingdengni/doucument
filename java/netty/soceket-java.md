# 服务端

## 阻塞

```java
/**
 * 1.打开一个服务端通道
 * 2.绑定对应的端口号
 * 3.通道默认是阻塞的，需要设置为非阻塞
 * 4.检查是否有客户端连接 有客户端连接会返回对应的通道
 * 5.获取客户端传递过来的数据,并把数据放在byteBuffer这个缓冲区中
 * 6.给客户端回写数据
 * 7.释放资源
 */
public static void main(String[] args) throws IOException, InterruptedException {
  //        1.打开一个服务端通道
  ServerSocketChannel serverSocketChannel=ServerSocketChannel.open();
  //        2.绑定对应的端口号
  serverSocketChannel.bind(new InetSocketAddress(7777));
  //        3.通道默认是阻塞的，需要设置为非阻塞
  serverSocketChannel.configureBlocking(false);
  System.out.println("服务端启动+++");
  while (true){
    //        4.检查是否有客户端连接 有客户端连接会返回对应的通道
    SocketChannel accept = serverSocketChannel.accept();
    if(accept==null){
      System.out.println("没有客户端连接");
      Thread.sleep(1000);
      continue;
    }
    //        5.获取客户端传递过来的数据,并把数据放在byteBuffer这个缓冲区中
    ByteBuffer allocate = ByteBuffer.allocate(1024);
    //返回值结果，返回是整数，代表本次读取到的有效字节数，0代表没有读到数据，-1代表读到末位
    int read = accept.read(allocate);
    System.out.println("客户端读取到的数据："+new String(allocate.array(),0,read, StandardCharsets.UTF_8));
    //        6.给客户端回写数据
    accept.write(ByteBuffer.wrap("嘻嘻".getBytes(StandardCharsets.UTF_8)));
    //        7.释放资源
    accept.close();
  }
}
```

## 异步访问

```java
public static void main(String[] args) throws IOException {
  //        1.打开一个服务端通道
  ServerSocketChannel channel = ServerSocketChannel.open();
  //        2.绑定对应的端口号
  channel.bind(new InetSocketAddress(7777));
  //        3.通道默认是阻塞的，需要设置为非阻塞
  channel.configureBlocking(false);
  //        4.创建选择器
  Selector selector = Selector.open();
  //        5.将服务端通道注册到选择器上,并指定注册监听的事件为OP_ACCEPT
  channel.register(selector, SelectionKey.OP_ACCEPT);
  System.out.println("服务端启动成功");
  while (true){
    //        6.检查选择器是否有事件,返回值为时间个数
    int select = selector.select(2000);
    if(select==0){
      System.out.println("无事发生");
      continue;
    }
    //        7.获取事件集合
    Set<SelectionKey> selectionKeys = selector.selectedKeys();
    //        8.判断事件是否是客户端连接事件SelectionKey.isAcceptable()
    Iterator<SelectionKey> iterator = selectionKeys.iterator();
    while (iterator.hasNext()){
      SelectionKey next = iterator.next();
      if(next.isAcceptable()){
        //        9.得到客户端通道,并将通道注册到选择器上, 并指定监听事件为OP_READ
        SocketChannel accept = channel.accept();
        accept.configureBlocking(false);
        System.out.println("有客户端连接");
        //将通道设置为非阻塞状态，因为selector需要轮询监听每个通道
        accept.register(selector,SelectionKey.OP_READ);
      }
      //        10.判断是否是客户端读就绪事件SelectionKey.isReadable()
      if (next.isReadable()) {
        //        11.得到客户端通道,读取数据到缓冲区
        SocketChannel socketChannel = (SocketChannel) next.channel();
        ByteBuffer allocate = ByteBuffer.allocate(1024);
        int read = socketChannel.read(allocate);
        if(read>0){
          System.out.println("客户端消息："+new String(allocate.array(),0,read, StandardCharsets.UTF_8));
          //        12.给客户端回写数据
          socketChannel.write(ByteBuffer.wrap("嘻嘻".getBytes(StandardCharsets.UTF_8)));
          socketChannel.close();
        }
      }
      //        13.从集合中删除对应的事件, 因为防止二次处理.
      iterator.remove();
    }
  }
}
```



# 客户端连接访问

```java
public static void main(String[] args) throws IOException {
  //        1.打开通道
  SocketChannel channel=SocketChannel.open();
  //        2.设置连接IP和端口号
  channel.connect(new InetSocketAddress("localhost",7777));
  //        3.写出数据
  channel.write(ByteBuffer.wrap("嘿嘿".getBytes(StandardCharsets.UTF_8)));
  //        4.读取服务器写回的数据
  ByteBuffer allocate = ByteBuffer.allocate(1024);
  int read = channel.read(allocate);
  System.out.println("服务端消息："+new String(allocate.array(),0,read,StandardCharsets.UTF_8));
  //        5.释放资源
  channel.close();
}

```

