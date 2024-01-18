## msgpack

### 使用案例

> org.msgpack:msgpack-core:(version)

#### 编码

```java
try {
  MessageBufferPacker packer = MessagePack.newDefaultBufferPacker();
  packer
    .packInt(1)
    .packString("leo");
  // pack arrays
  int[] arr = new int[] {3, 5, 1, 0, -1, 255};
  packer.packArrayHeader(arr.length);
  for (int v : arr) {
    packer.packInt(v);
  }
  // pack map (key -> value) elements
  // the number of (key, value) pairs
  packer.packMapHeader(2);
  // Put "apple" -> 1
  packer.packString("apple");
  packer.packInt(1);
  // Put "banana" -> 2
  packer.packString("banana");
  packer.packInt(2);

  // pack binary data
  byte[] ba = new byte[] {1, 2, 3, 4};
  packer.packBinaryHeader(ba.length);
  packer.writePayload(ba);

  packer.close();

} catch (IOException e) {
}


```

#### 解码

```java
MessageUnpacker unpacker = MessagePack.newDefaultUnpacker(packer.toMessageBuffer().array());
int id = unpacker.unpackInt();
String name = unpacker.unpackString();
System.out.print(id);
System.out.print(name);
int numPhones = unpacker.unpackArrayHeader();
String[] phones = new String[numPhones];
for (int i = 0; i < numPhones; ++i) {
  phones[i] = unpacker.unpackString();
}
int maplen = unpacker.unpackMapHeader();
for (int j = 0; j < maplen; j++) {
  unpacker.unpackString();
  unpacker.unpackInt();
}
unpacker.close();
```

文件流

```java
TabsJson tabsjson = new TabsJson();
tabsjson.type = 1; （int）
tabsjson.body = "abc"; （String）

// 编码
// 写入文件流的案例
File tempFile = File.createTempFile("target/tmp", ".txt");
tempFile.deleteOnExit();
// Write packed data to a file. No need exists to wrap the file stream with BufferedOutputStream, since MessagePacker has its own buffer
MessagePacker packer = MessagePack.newDefaultPacker(new FileOutputStream(tempFile));
// 以下是对自定义数据类型的打包
byte[] extData = tabsjson.body.getBytes(MessagePack.UTF8);
packer.packExtensionTypeHeader(tabsjson.type, extData.length());
packer.writePayload(extData);
packer.close();

// 解码
// 读取文件流
FileInputStream fileInputStream = new FileInputStream(new File(filepath));
MessageUnpacker unpacker = MessagePack.newDefaultUnpacker(fileInputStream);
//先将自定义数据的消息头读出
ExtensionTypeHeader et = unpacker.unpackExtensionTypeHeader();
//判断消息类型
if (et.getType() == 1) {
    int lenth = et.getLength();
    //按长度读取二进制数据
    byte[] bytes = new byte[lenth];
    unpacker.readPayload(bytes);
    //构造tabsjson对象
    TabsJson tab = new TabsJson();
    //构造unpacker将二进制数据解包到java对象中
    MessageUnpacker unpacker1 = MessagePack.newDefaultUnpacker(bytes);
    tab.type = unpacker1.unpackInt();
    tab.body = unpacker1.unpackString();
    unpacker1.close();
}
unpacker.close();
```



### msgpack 和 json 结合

> org.msgpack:jackson-dataformat-msgpack:0.8.20

#### 打包/解包

```java
ObjectMapper objectMapper = new ObjectMapper(new MessagePackFactory());
MyMessage myMessage = new MyMessage("remer", 1111);
byte[] bytes = objectMapper.writeValueAsBytes(myMessage);

MyMessage deserialized = objectMapper.readValue(bytes, MyMessage.class);
System.out.print(deserialized.name);
```

#### 实体类 继承（Serializable）

```java
public static class MyMessage implements Serializable {

  public String name;
  public double version;

  public MyMessage(String name, double version) {
    this.name = name;
    this.version = version;
  }

  public MyMessage(){}
}
```

