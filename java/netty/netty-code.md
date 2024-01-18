

## 内置编码器

```java
// netty中基本的codec有base64、bytes、compression、json、marshalling、protobuf、serialization、string和xml这几种
netty-codec
netty-codec-http
netty-codec-http2
netty-codec-memcache
netty-codec-redis
netty-codec-socks
netty-codec-stomp
netty-codec-mqtt
netty-codec-haproxy
netty-codec-dns
```



## netty编码、解码结合（msgpack、protobuf）案例

### decoder：MessageToMessageDecoder\<ByteBuf>

### encoder：MessageToByteEncoder\<User>



## 使用msgpack处理案列

> MessagePack 是一个高效的二进制序列化格式。它让你像JSON一样可以在各种语言之间交换数据。但是它比JSON更快、更小。小的整数会被编码成一个字节，短的字符串仅仅只需要比它的长度多一字节的大小

```xml
<dependency>
    <groupId>org.msgpack</groupId>
    <artifactId>msgpack</artifactId>
    <version>0.6.12</version>
</dependency>
<!-- 原有json长度：27 byte  / msgPack长度：18 byte -->
```

### 解码器

```java
public class MsgPackDecoder extends MessageToMessageDecoder<ByteBuf> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out)
            throws Exception {
        final int length = msg.readableBytes();
        final byte[] array = new byte[length];
        msg.getBytes(msg.readerIndex(),array,0,length);
        MessagePack messagePack = new MessagePack();
        out.add(messagePack.read(array,User.class)); // 解码为user对象
    }
}
```

### 编码器

```java
public class MsgPackEncoder extends MessageToByteEncoder<User> {
    @Override
    protected void encode(ChannelHandlerContext ctx, User msg, ByteBuf out)
            throws Exception {
        MessagePack messagePack = new MessagePack();
        byte[] raw = messagePack.write(msg); // 编码user为byte
        out.writeBytes(raw);

    }
}
```

## protobuf

> 说明：不用proto文件
>
> 使用protobuf，但是protobuf需要维护大量的proto文件比较麻烦，现在一般可以使用protostuff。protostuff是一个基于protobuf实现的序列化方法，它较于protobuf最明显的好处是，在几乎不损耗性能的情况下做到了不用我们写.proto文件来实现序列化。

### xml 依赖

```xml
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-api</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.6.0</version>
</dependency>
```

### 代码

>  字段LinkedBuffer:   这个字段表示，申请一个内存空间用户缓存，LinkedBuffer.DEFAULT_BUFFER_SIZE表示申请了默认大小的空间512个字节，我们也可以使用MIN_BUFFER_SIZE，表示256个字节
>
> 字段schemaCache:  这个字段表示缓存的Schema。那这个Schema是什么呢？就是一个组织结构，就好比是数据库中的表、视图等等这样的组织机构，在这里表示的就是序列化对象的结构



```java
public class ProtostuffUtils {
    //避免每次序列化都重新申请Buffer空间
    private static LinkedBuffer buffer = LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE);
    //缓存Schema
    private static Map<Class<?>, Schema<?>> schemaCache = new ConcurrentHashMap<Class<?>, Schema<?>>();
    //序列化方法，把指定对象序列化成字节数组
    @SuppressWarnings("unchecked")
    public static <T> byte[] serialize(T obj) {
        Class<T> clazz = (Class<T>) obj.getClass();
        Schema<T> schema = getSchema(clazz);
        byte[] data;
        try {
            data = ProtostuffIOUtil.toByteArray(obj, schema, buffer);
        } finally {
            buffer.clear();
        }
        return data;
    }
    //反序列化方法，将字节数组反序列化成指定Class类型
    public static <T> T deserialize(byte[] data, Class<T> clazz) {
        Schema<T> schema = getSchema(clazz);
        T obj = schema.newMessage();
        ProtostuffIOUtil.mergeFrom(data, obj, schema);
        return obj;
    }
  	//注册Schema
    @SuppressWarnings("unchecked")
    private static <T> Schema<T> getSchema(Class<T> clazz) {
        Schema<T> schema = (Schema<T>) schemaCache.get(clazz);
        if (schema == null) {
            schema = RuntimeSchema.getSchema(clazz);
            if (schema == null) {
                schemaCache.put(clazz, schema);
            }
        }
        return schema;
    }
  
   public static void main(String[] args) {
        byte[] userBytes = ProtostuffUtil.serializer(new User(1, "zhangsan"));
        User user = ProtostuffUtil.deserializer(userBytes, User.class);
        System.out.println(user);
    }
}
```

