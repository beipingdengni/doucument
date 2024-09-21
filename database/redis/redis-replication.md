参考博客： https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/redis-master-slave.md



阿里巴巴同步工具：https://tair-opensource.github.io/RedisShake/

java工具包：https://github.com/leonchen83/redis-replicator

shell工具包：https://github.com/leonchen83/redis-rdb-cli

> 故从 Redis 2.8 版本开始，其引入了 PSYNC 命令用于替代此前的 SYNC 命令，实现对**同步**操作进行优化。PSYNC 命令具有完整重同步、部分重同步两种模式。

# 整体架构复制同步

## 一、Redis replication复制的完整流程

1 slave node启动之后，仅仅只会保存master node的信息，包括master node的host 和ip，但是复制流程还没有开始。
2 salve node 内部有个定时任务，每秒检查是否有master node要连接和复制，如果发现， slave node 就跟 master node 建立socket网络连接。
3 slave node 发送ping命令给master node。
4 口令认证，如果master 设置了 requiress，那么 salve node必须发送masterauth的口令过去进行认证。
5 slave node第一次连接到master，会执行全量复制，将所有数据发送给slave node
6 master node后续持续写命令，异步复制到slave node。

![image-20240221105646235](imgs/redis-replication/image-20240221105646235.png)

## 二、Redis的断点续传和无磁盘化复制，全量复制

### 1.断点续传

其实redis从2.8开始就支持主从复制的断点续传，即如果主从复制过程中，由于网络等原因断掉，那么可以接着上次复制的地方，继续复制下去。
而底层实现的机制就是通过在master 和slave中都保存的replica offset，offset是保存在backlog中的，如果master和slave网络连接断掉，slave会让master从上次的replica offset开始继续复制，但是如果没有找到offset，就会执行一次全量复制

#### 1.1 offset

1）master和slave都会维护一个offset，两者都会在自身不断累加offset，
2）slave每秒都会上传offset给master，同时master也会保存每个slave的offset

#### 1.2 backlog

master有一个backlog文件，默认是1mb，offset就是保存在backlog中的，backlog文件主要是用来做全量复制断开的增量复制

#### 1.3 Master run idheartbeat

如果根据ip+host来定位master是不靠谱的，尤其当master重启或者数据发生变化的时候，那么slave应根据不同的run id区分，run id 不同就做全量复制

#### 1.4 heartbeat

1. 主从节点互相都会发送heartbeat信息
2. master 默认每隔10秒发送一次heartbeat，slave node每秒发送一个heartbeat。

### 2.无磁盘化复制

master在内存中直接创建rdb，然后发送给slave，不会再自己本地落地磁盘

### 3.全量复制

1. master 执行 bgsave 命令，在本地生成一份rdb快照文件。
2. master node 将rdb快照文件发送到 salve node，如果rdb复制事件超过60s(repl-timeout)，那么 slave node 就会认为复制失败，可以适当的调大这个参数。
3. 对于千兆网卡的机器，一般一秒传输100MB文件，6G文件很可能超过60s秒，导致复制失败
4. master node 生成rdb时，会将所有新增的命令存储到内存中，在slave node 保存了rdb之后，再讲新的写命令复制给 slave node。
5. client-output-buffer-limit slave 256MB 64MB 60,如果在复制期间，新增的数据超过内存缓存区大小64MB，或者一次性到达256MB，那么停止复制，复制失败
6. 如果slave node 开启了AOF，那么会立即执行BGRWRITEAOF，重新AOF。

### 4.增量复制

1. 如果复制过程中，master-slave网络连接中断，那么salve重新连接到master，是进行增量复制
2. master 直接从自己的backlog中获取部分丢失的数据，发送给slave node，默认backlog的大小是1MB
3. master 会根据salve发送的psync中的offset来从backlog中获取数据。



# PSYNC 命令的基本原理

## Replication Offset 复制偏移量

在 PSYNC 命令中引入了 **Replication Offset 复制偏移量**的概念，包括 Master 的复制偏移量、Slave 的复制偏移量。每次当 Master 向 Slave 传播 N 个字节的数据命令时，Master 就会将自身的复制偏移量加上 N；每次当 Slave 从 Master 接收到 N 个字节的数据后，Slave 就会将自身的复制偏移量加上 N。这样只需对比 Master、Slave 的复制偏移量，即可知道二者之间是否处于一致的状态。具体地来说，当二者之间的复制偏移量完全一样，则说明二者的数据状态是一致的；反之则说明二者的数据状态不一致

> 例如，Master 的复制偏移量为 255，Slave A 的复制偏移量为 255，Slave B 的复制偏移量为 200。此时说明：Master、Slave A 之间的数据状态是一致的；Master、Slave B 之间的数据状态则不一致。那么要让 Slave B 与 Master 保持一致，Master 只需再次向 Slave B 补传后者所缺失的 55 (255-200) 个字节的数据即可

## Replication Backlog 复制积压缓冲区

为了能够实现 Master 对 Slave B 补传数据，就需要在 Master 中引入一个 **Replication Backlog 复制积压缓冲区**的概念。其是 Master 中的一个固定长度的 FIFO 队列。每次 Master 向所有 Slave 传播、发送命令的同时，也会将该命令写入复制积压缓冲区当中。同时也会在复制积压缓冲区记录其中每个字节数据所对应的偏移量。

这样当 Slave 断线重连后，Slave 会通过 PSYNC 命令向 Master 告知 Slave 自身的复制偏移量。Master 即可根据 Slave 的复制偏移量 offset 确定是采用部分重同步、还是完整重同步。具体地

- 如果 offset 偏移量之后的数据 (即从偏移量 offset+1 开始的数据) 依然存在于 Master 的复制积压缓冲区当中，则 Master 就可以进行部分重同步，只需将保存在复制积压缓冲区中 offset 偏移量之后的所有数据再次发送给 Slave 即可
- 如果 offset 偏移量之后的数据 (即从偏移量 offset+1 开始的数据) 不存在于 Master 的复制积压缓冲区当中，则 Master 就只能 Slave 进行完整重同步，以让二者达到一致。因为复制积压缓冲区是固定长度的，其只能保存 Master 在最近一段时间内的所有数据、命令。并不能保存无限期限的数据、命令

## Redis 服务端的运行 ID

事实上对于 Redis 服务端而言，无论是 Master 还是 Slave。其在启动运行后，都会有一个唯一标识 ——run ID (运行 ID)。当 Slave 第一次与 Master 建立同步时，Master 会将自身的运行 ID 发送给 Slave。Slave 会把这个 Master 的 run ID 保存、记录下来。这样当 Slave 断线、重连后，会将之前保存的 Master 的 run ID 再次发送给目前连接到的 Master

- 如果 Master 发现从 Slave 处接收到的 run ID 就是自己本身，那说明这个 Slave 在断开之前连的就是自己。这样 Master 就会继续尝试执行部分重同步操作
- 反之则说明，Slave 在断开之前连接的 Master 与 当前正在连接的 Master 并不是同一个。此时，Master 就必须对 Slave 进行完整重同步

## PSYNC 命令执行流程

对于 Slave 而言：

- 如果 Slave 此前未进行过任何复制操作 或 已经执行过了 SLAVEOF NO ONE 命令，那么 Slave 在开始一次新的复制时将会向 Master 发送 PSYNC ? -1 命令，主动请求 Master 进行完整重同步
- 反之，Slave 此前已经复制过了某个 Master。那么 Slave 在开始复制时会向 Master 发送 PSYNC 命令。其中，runid 是 Slave 上次复制的 Master 的 run ID；offset 则是 Slave 的复制偏移量。Master 会根据上述两个参数的情况来决定具体执行何种同步。即是完整重同步、还是部分重同步

对于 Master 而言，其在接收到 Slave 的 PSYNC 命令，作出以下三种回复中的一种

1. **FULLRESYNC** ：意为 Master 将会与 Slave 进行完整重同步。其中，runid 是 Master 的运行 ID。Slave 会将其保存、记录下来。以便下一次发送 PSYNC 命令时使用；offset 则是 Master 当前的复制偏移量，Slave 会将该值用于初始化自身的复制偏移量
2. **+CONTINUE** ：意为 Master 将会与 Slave 进行部分重同步。此时 Slave 只需等待 Master 将自身所缺失的部分数据发送过来即可
3. **-ERR**：意为 Master 版本低于 2.8，无法识别 PSYNC 命令。此时 Slave 会发送 SYNC 命令，并与 Master 进行完整重同步

## 心跳检测机制

与此同时在 Redis 2.8 版本中也引入了心跳机制。即在命令传播阶段，Slave 默认会以每秒一次的频率向 Master 发送如下命令。其中 offset 是 Slave 的复制偏移量

```
REPLCONF ACK <offset>
```

即所谓的心跳机制，其意义有以下几个方面：

**1. 检查主从之间的网络状态**

Master 如果长时间没有接收到某个 Slave 发送的 REPLCONF ACK 命令，那么 Master 就知道其与该 Slave 的网络连接存在异常

**2. 用于实现 min-slaves 配置项**

Redis 的配置文件提供了 min-slaves 配置项以防止 Master 在不安全的情况下执行写命令，示例配置如下所示。如果期望禁用该功能，只需将 min-slaves-to-write、min-slaves-max-lag 任意一项设为 0 即可

```
# Master 在接受客户端写入命令时，所必需的可用Slave的最小数目要求
# 一旦可用Slave的数量低于该配置项值，则Master会拒绝执行写命令
min-slaves-to-write 3

# Master与Slave 之间网络延迟(lag)的上限，Unit：秒
# 一旦某个Slave与Master的延迟超过该值，则Master会该Slave将视为不可用状态
min-slaves-max-lag 10
```

**3. 检测命令丢失**

Master 传播给 Slave 的命令可能会由于网络原因导致 Slave 并未收到。而 Slave 通过 REPLCONF ACK 命令主动向 Master 上报自身当前的复制偏移量。这样 Master 即可主动发现传播的命令是否发生了丢失，并利用复制积压缓冲区对丢失的命令进行补传。由于 REPLCONF ACK 命令、复制积压缓冲区都是 2.8 版本开始引入的。故在之前的版本中。即使命令在传播过程中丢失了，Master、Slave 均无法注意到。更无法进行命令补传

# PSYNC同步机制操作（操作）

## 全量复制

1、slave第一次启动时，连接Master，发送PSYNC命令，格式为 `psync {runId} {offset}`

> `{runId}` 为master的运行id；`{offset}`为slave自己的复制偏移量。
> slave第一次连接master时，slave并不知道master的`runId`，也不知道自己偏移量，这时候slave会传一个问号和`-1`，告诉master节点是第一次同步。格式为`psync ? -1`

> 每个Redis节点启动后，都会动态分配一个40位的十六进制字符串作为运行ID，即{runId}。运行ID的主要作用是用来唯一识别Redis节点。
> 参与复制的主从节点都会维护自身复制偏移量，即 `{offset}`。

2、当master接收到`psync ? -1`时，知道slave是要全量复制，就会将自己的runId和offset告知slave，回复命令`fullresync {runId} {offset}`。同时，master会执行bgsave命令来生成rdb文件，期间的所有写命令将被写入缓冲区。

> slave接受到master的回复命令后，会保存master的runId和offset，slave此时处于同步状态。

> 缓冲区本质上是先进先出（FIFO）的定长队列，能实现保存最近已复制数据的功能，用于部分复制和复制命令丢失的数据补救。

3、master bgsave执行完毕，向slave发送rdb文件。rdb文件发送完毕后，开始向slave发送缓冲区中的写命令。
4、slave收到rdb文件，丢弃所有旧数据，开始载入rdb文件。rdb文件同步结束之后，slave执行从master缓冲区发送过来的所有写命令。
5、此后 master 每执行一个写命令，就向slave发送相同的写命令。

<img src="imgs/redis-replication/image-20240221113850558.png" alt="image-20240221113850558" style="zoom:75%;" />

## 增量复制

1、如果出现网络闪断或者命令丢失等异常情况时，当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当作psync参数发送给主节点，要求进行部分复制操作，格式为`psync {runId} {offset}`。
2、主节点接到psync命令后首先核对参数runId是否与自身一致，如果一致，说明之前复制的是当前主节点；之后根据参数offset在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送+continue响应，表示可以进行部分复制；否则进行全量复制。
3、主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。

## 同步故障处理

### 拷贝超时

对于数据量较大的主节点，比如生成的rdb文件超过6GB以上时要格外小心。传输文件这一步操作非常耗时，速度取决于主从节点之间网络带宽，通过细致分析full resync和masterslave这两行日志的时间差，可以算出rdb文件从创建到传输完毕消耗的总时间。如果总时间超过repl-timeout所配置的值（默认60秒），从节点将放弃接受rdb文件并清理已经下载的临时文件，导致全量复制失败。

> 针对数据量较大的节点，建议调大repl-timeout参数防止出现全量同步数据超时。
> 例如对于千兆网卡的机器，网卡带宽理论峰值大约每秒传输100MB，在不考虑其他进程消耗带宽的情况下，6GB的rdb文件至少需要60秒传输时间，默认配置下，极易出现主从数据同步超时。

### 积压缓冲区拷贝溢出

如果主节点创建和传输RDB的时间过长，对于高流量写入场景非常容易造成主节点复制客户端缓冲区溢出。默认配置为`client-output-buffer-limit slave 256MB 64MB 60`，如果60秒内缓冲区消耗持续大于64MB或者直接超过256MB时，主节点将直接关闭复制客户端连接，造成全量同步失败。

因此，运维人员需要根据主节点数据量和写命令并发量调整`client-output-buffer-limit slave`配置，避免全量复制期间客户端缓冲区溢出。

### slave全量同步的响应问题

slave节点接收完主节点传送来的全部数据后会清空自身旧数据，执行flash old data，然后加载rdb文件。对于较大的rdb文件，这一步操作依然比较耗时。

对于线上做读写分离的场景，从节点也负责响应读命令，如果slave节点正出于全量复制阶段，那么slave节点在响应读命令可能拿到过期或错误的数据。对于这种场景，redis复制提供了slave-server-stale-data yes参数（默认开启），如果开启则slave节点依然响应所有命令。对于无法容忍不一致的应用场景可以设置no来关闭命令执行，此时从节点除了info和slaveof命令之外所有的命令只返回sync with master in progress信息

> 输入以下指令: set key 001、del key 001、set key 001 ex 5  # 5秒过期

```shell
redis-cli PSYNC ? -1 # 输入同步指令后，一下是返回值
Entering replica output mode...  (press Ctrl-C to quit)
# 应该是返回对应：{runId} {offset}
PSYNC replied +FULLRESYNC 686aa2be99b1e47cdcb16394e49c58b300f46331 186
Full resync with master, discarding 468284 bytes of bulk transfer...
Full resync done. Logging commands from master.
"ping"
"SELECT","0"
"set","key","001"
"ping"
"del","key"
"ping"
"ping"
"ping"
"SET","key","001","PXAT","1708487423126"
"ping"
"DEL","key"
```

## java 代码实现

### 原生代码处理

```java
import java.io.*;
import java.net.Socket;

public class RedisReplica {
    private static final String REDIS_MASTER_HOST = "master_host";
    private static final int REDIS_MASTER_PORT = 6379;

    public static void main(String[] args) {
        try (Socket socket = new Socket(REDIS_MASTER_HOST, REDIS_MASTER_PORT);
             OutputStream outputStream = socket.getOutputStream();
             InputStream inputStream = socket.getInputStream();
             BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
             BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(outputStream))) {

            // 发送 PSYNC 命令，这里使用 ? -1 表示初次同步
            writer.write("PSYNC ? -1\r\n");
            writer.flush();

            // 读取来自 Redis 主服务器的响应
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println("Received: " + line);
                // 这里需要根据实际响应处理数据同步逻辑
                // 如果是 RDB 文件，需要解析并加载数据
                // 如果是实时命令，需要应用到本地数据集
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**解析RDB文件**：在初次同步时，Redis主服务器会发送一个RDB文件。你需要能够解析这个文件并加载数据到你的本地存储。
**处理命令流**：在RDB文件之后，主服务器会发送实时的写命令。你需要解析这些命令并应用到你的本地数据集。
**断线重连和部分同步**：如果连接中断，你需要能够重新连接到主服务器，并根据最后接收的偏移量请求部分同步。
**错误处理和日志记录**：为了稳定运行，你需要处理网络错误、协议解析错误，并记录必要的日志信息。

### redis-replicator实现

使用文档：https://github.com/leonchen83/redis-replicator/blob/master/README.zh_CN.md#1-redis-replicator

在Java中实现Redis的`PSYNC`命令通常是为了参与Redis的复制协议，这涉及到与Redis主服务器建立连接，并遵循特定的协议来进行数据同步。`PSYNC`命令是复制协议的一部分，用于实现部分同步或全同步。

以下是使用`redis-replicator`库实现`PSYNC`命令的一个简化示例。这个库封装了与Redis复制相关的细节，包括处理`PSYNC`命令。

首先，添加`redis-replicator`依赖到你的项目中。如果你使用Maven，可以在`pom.xml`文件中添加以下依赖：

```xml
<dependency>
    <groupId>com.moilioncircle</groupId>
    <artifactId>redis-replicator</artifactId>
    <version>版本号</version>
</dependency>
```

请确保替换`版本号`为最新的可用版本。

```java
Replicator replicator = new RedisReplicator("redis://127.0.0.1:6379?authPassword=foobared&readTimeout=10000&ssl=yes");
// 解析RDB文件
Replicator replicator = new RedisReplicator("redis:///path/to/dump.rdb?rateLimit=1000000");
// 解析AOF文件
Replicator replicator = new RedisReplicator("redis:///path/to/dump.aof?rateLimit=1000000");
// 密码
Replicator replicator = new RedisReplicator("rediss://user:pass@127.0.0.1:6379?rateLimit=1000000");
```

然后，你可以使用以下代码来实现`PSYNC`：

```java
import com.moilioncircle.redis.replicator.RedisReplicator;
import com.moilioncircle.redis.replicator.Replicator;
import com.moilioncircle.redis.replicator.event.Event;
import com.moilioncircle.redis.replicator.event.EventListener;

public class RedisPSyncExample {

    public static void main(String[] args) {
        // 创建RedisReplicator实例连接到Redis主服务器
        try (Replicator replicator = new RedisReplicator("redis://主服务器IP:6379")) {
            // 添加事件监听器来处理接收到的事件
            replicator.addEventListener(new EventListener() {
                @Override
                public void onEvent(Replicator replicator, Event event) {
                    // 处理事件，例如打印到控制台  KeyStringValueString是string类型
                    if (event instanceof KeyValuePair) {
                        // RDB解析出来都是对于的值，key value
                        KeyValuePair kv = (KeyValuePair) event;
                        System.out.println(kv.getKey());
                        System.out.println(kv.getValue());
                    } else if (event instanceof DefaultCommand) {
                        // AOF解析出来都是指令
                        // 如果连接的是redis服务地址，这里就是监听所有执行的指令
                        DefaultCommand defaultCommand = (DefaultCommand) event;
                        System.out.println(CommandName.name(new String(defaultCommand.getCommand())));
                        System.out.println(new String(defaultCommand.getArgs()[0]));
                    }else {
                        System.out.println(event);
                    }
                }
            });
            // 开始监听命令流
            replicator.open();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**注意在某些云服务中, PSYNC 是被禁止使用的, 因此我们使用 Scan 命令来替换PSYNC命令扫描全库, 如下所示**

> ```java
> Replicator r = new RedisReplicator("redis://127.0.0.1:6379?enableScan=yes&scanStep=256");
> r.addEventListener(new EventListener() {
>   @Override
>   public void onEvent(Replicator replicator, Event event) {
>   	System.out.println(event);
>   }
> });
> r.open();
> ```

## 执行java指令

```xml
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>5.1.0</version>
</dependency>
```

指令现象

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Protocol;

public class RedisCacheMain {
    public static void main(String[] args) {
        // 创建Jedis实例连接到Redis服务器
        try (Jedis jedis = new Jedis("localhost", 6379)) {
            // 执行Redis命令
            String command = "SET";
            String key = "mykey";
            String value = "myvalue";
            Object result = jedis.sendCommand(Protocol.Command.valueOf(command), key, value);
            System.out.println("Command result: " + result);
            // 执行其他Redis命令...
        }
    }
}
```

