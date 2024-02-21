### filesystem cache

在Java中，文件系统缓存(FileSystem Cache)通常是指操作系统级别的缓存，它用于提高文件I/O操作的性能。当你在Java程序中读取或写入文件时，操作系统会将数据缓存到内存中，这样后续对同一文件的访问就可以直接从内存中获取数据，而不是每次都从磁盘读取，从而大幅提高效率。

Java本身并没有直接提供控制文件系统缓存的API，因为这是由底层操作系统管理的。不过，Java NIO（非阻塞I/O）提供了`MappedByteBuffer`类，可以让开发者利用内存映射文件(memory-mapped files)的功能，这在某种程度上与文件系统缓存的概念相似。



## 文本写入（MappedByteBuffer）

> try-with-resources 语句自动关闭资源

```java
public class ZeroCopyLogWriter {
    public static void main(String[] args) {
        try {
            RandomAccessFile file = new RandomAccessFile("logfile.txt", "rw");
            FileChannel channel = file.getChannel();

            // 将文件映射到内存中
            MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1024); // channel.size()
            // 在内存映射文件中进行写入操作
            buffer.put("Log entry 1".getBytes());
            buffer.put("Log entry 2".getBytes());
            // 刷新缓冲区并释放资源
            buffer.force();
          
            // 读取或写入数据
            while (buffer.hasRemaining()) {
                byte b = buffer.get(); // 读取数据
            }
            // 修改数据
            buffer.put(0, (byte) 97); // 在索引0的位置写入值 'a'
            // 强制将缓冲区的更改写入文件
            buffer.force();
						
          	// 关闭
            channel.close();
            file.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 文件复制（transferFrom）

```java
public class ZeroCopyExample {
    public static void main(String[] args) throws Exception {
        RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
        FileChannel fromChannel = fromFile.getChannel();

        RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
        FileChannel toChannel = toFile.getChannel();

        long position = 0;
        long count = fromChannel.size();

        // 使用 ByteBuffer 进行零拷贝操作
        toChannel.transferFrom(fromChannel, position, count);

        fromChannel.close();
        toChannel.close();
        fromFile.close();
        toFile.close();
    }
}
```



## 时间环（circular buffer）

```java
public class CircularBuffer {
    private int[] buffer;
    private int size;
    private int writePointer = 0;
    private int readPointer = 0;

    public CircularBuffer(int size) {
        this.size = size;
        buffer = new int[size];
    }

    public void write(int data) {
        buffer[writePointer] = data;
        writePointer = (writePointer + 1) % size;
        if (writePointer == readPointer) {
            readPointer = (readPointer + 1) % size; // 溢出时覆盖最旧的数据
        }
    }

    public int read() {
        if (readPointer != writePointer) {
            int data = buffer[readPointer];
            readPointer = (readPointer + 1) % size;
            return data;
        }
        return -1; // 环为空
    }
}

```

