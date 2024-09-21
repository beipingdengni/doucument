

```xml
<dependency>
  <groupId>com.caucho</groupId>
  <artifactId>hessian</artifactId>
  <version>4.0.38</version>
</dependency>
```

Hessian 是由 caucho 提供的一个基于 binary-RPC 实现的远程通讯 library 。它是**高性能二进制协议**，支持很**多种语言**，众所周知大名鼎鼎的开源rpc的框架中都有使用。

建议使用场景

> dubbo 协议基于 hessian 作为序列化协议。使用的场景是：传输数据量小（每次请求在 100kb 以内），但是并发量很高。



Java demo代码

```java
public static <T> byte[] serialize(T input) {
  if (input == null) throw new NullPointerException("input is requird.");
  byte[] result = null;
  ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
  HessianSerializerOutput hessianSerializerOutput = null;
  try {
    hessianSerializerOutput = new HessianSerializerOutput(outputStream);
    //write本身是线程安全的
    hessianSerializerOutput.writeObject(input);
    hessianSerializerOutput.close();
    result = outputStream.toByteArray();
  } catch (Exception ex) {
    throw new RuntimeException(ex);
  } finally {
    IOUtils.closeQuietly(outputStream);
  }
  return result;
}

public static <T> T deserialize(Class<T> retCls, byte[] input) {
  if (input == null) throw new NullPointerException();
  T result = null;
  ByteArrayInputStream byteArrayInputStream = null;
  try {
    byteArrayInputStream = new ByteArrayInputStream(input);
    HessianSerializerInput hessianInput = new HessianSerializerInput(byteArrayInputStream);
    result = (T) hessianInput.readObject();
  } catch (Exception e) {
    throw new RuntimeException(e);
  } finally {
    IOUtils.closeQuietly(byteArrayInputStream);
  }
  return result;
}
```

