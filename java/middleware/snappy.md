## snappy压缩

```xml
<dependency>
 <groupId>org.xerial.snappy</groupId>
 <artifactId>snappy-java</artifactId>
 <version>1.1.4</version>
</dependency>
```

### 字符压缩

```java
import org.xerial.snappy.Snappy;
String input = "Hello snappy-java! Snappy-java is a JNI-based wrapper of Snappy, a fast compresser/decompresser.";
{ 
    byte[] compressed = Snappy.compress(input.getBytes("UTF-8"));
    byte[] uncompressed = Snappy.uncompress(compressed);
    String result = new String(uncompressed, "UTF-8");
    System.out.println(result);
}

{
    byte[] compressed = Snappy.compress(input);
    System.out.println(Snappy.uncompressString(compressed));
}

{
    double [] arr = new double[]{123.456,234.567,345.678};
    byte[] compressed = Snappy.compress(arr);
    double [] unarr = Snappy.uncompressDoubleArray(compressed);
    System.out.println(Arrays.toString(unarr));
}
```

### 低级别API

```java
/*
*inputAddr:待压缩数据的内存地址
*inputSize:待压缩数据的byte size
*destAddr:压缩结果的保存地址
*return：返回压缩后数据的大小
*/
public static long rawCompress(long inputAddr,long inputSize, long destAddr)throws java.io.IOException;
/*
*inputAddr:待解压缩数据的内存地址
*inputSize:待解压缩数据的byte size
*destAddr:解压缩结果的保存地址
*return：返回解压缩后数据的大小
*/
public static long rawUncompress(long inputAddr,long inputSize,long destAddr)throws java.io.IOException;
```

### 文件压缩

> SnappyOutputStream和SnappyInputStream分别用于流数据的压缩/解压缩。
>
> 此外，从snappy v1.1.0开始提供了Framing-format(帧格式)输入输出流的压缩/解压缩的方法：SnappyFramedOutputStream和SnappyFramedInputStream。
>
> 需要注意:
>
> ​	以SnappyOutputStream压缩的数据不能以SnappyFramedInputStream方法解开，反之亦然。下面以SnappyOutputStream和SnappyInputStream为例介绍。
>
> ```java
> // 内存中字节压缩
> ByteArrayInputStream is = new ByteArrayInputStream(data);
> ByteArrayOutputStream os = new ByteArrayOutputStream();
> ```

#### 压缩

```java
File file = new File("..."); //待压缩文件
File out = new File("./", file.getName() + ".snappy"); //压缩结果文件

byte[] buffer = new byte[1024 * 1024 * 8];
FileInputStream  fi = null;
FileOutputStream fo = null;
SnappyOutputStream sout = null;
try
{
    fi = new FileInputStream(file); 
    fo = new FileOutputStream(out);
    sout = new SnappyOutputStream(fo);
    while(true)
    {
        int count = fi.read(buffer, 0, buffer.length);
        if(count == -1) { break; }
        sout.write(buffer, 0, count);
    }
    sout.flush();
}
catch(Throwable ex)
{
    ex.printStackTrace();
}
finally 
{
    if(sout != null) {try { sout.close();} catch (Exception e) {}}
    if(fi != null) { try { fi.close(); } catch(Exception x) {} }
    if(fo != null) { try { fo.close(); } catch(Exception x) {} }
}
```

#### 解压

```java
File file = new File("xxx"); //待解压文件
File out = new File("xxx");  //解压后文件

byte[] buffer = new byte[1024 * 1024 * 8];
FileInputStream  fi = null;
FileOutputStream fo = null;
SnappyInputStream sin = null;
try
{
    fo = new FileOutputStream(out);
    fi = new FileInputStream(file.getPath());
    sin = new SnappyInputStream(fi);        
    while(true)
    {
        int count = sin.read(buffer, 0, buffer.length);
        if(count == -1) { break; }
        fo.write(buffer, 0, count);
    }
    fo.flush();
}
catch(Throwable ex)
{
    ex.printStackTrace();
}
finally 
{
    if(sin != null) { try { sin.close(); } catch(Exception x) {} }
    if(fi != null) { try { fi.close(); } catch(Exception x) {} }
    if(fo != null) { try { fo.close(); } catch(Exception x) {} }
}
```

