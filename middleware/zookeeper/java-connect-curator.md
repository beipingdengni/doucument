

# 官方网站

https://curator.apache.org/getting-started.html

## Stat

ZooKeeper命名空间中的每个znode都有一个与之关联的stat结构，类似于Unix/Linux文件系统中文件的stat结构，znode的stat结构的字段和各自的含义如下：

1. cZxid: 创建znode的事务ID

2. mZxid: 最后修改znode的事务ID

3. pZxid: 最后修改添加或删除子节点的事务ID

4. ctime: 表示从1970-01-01T00:00:00Z开始以毫秒为单位的znode创建时间

5. mtime: 表示从1970-01-01T00:00:00Z开始以毫秒为单位的znode最后修改时间

6. dataVersion: 表示对该znode数据所做的更改次数

7. cVersion: 表示对改znode的子节点进行的更改次数

8. aclVersion: 表示对znode的ACL（访问权限列表）进行更改的次数

9. emphemeralOwner: 如果znode是临时节点，则表示znode所有者的session ID; 如果改节点不是临时节点，则该值为0

10. dataLength: znode数据的长度

11. numChildren: 表示znode子节点的数量

12. Zxid：类似于RDBMS中的事务ID，用于标识一次更新操作的Proposal ID（提议ID）。

    > 为了保证顺序性，该zxid必须单调递增。因此ZooKeeper使用一个64位的数来标识，高32位是leader的epoch（Leader对应的年号）, 从1开始，每次选出新的Leader, epoch加1， 低32位为该epoch的序号， 每次epoch变化都将低32位的序号重置，这样就保证了Zxid的全局递增性



## Java 连接

> Zookeeper典型应用场景的实现，这些实现是基于Curator Framework。

```xml
<dependency>
  <groupId>org.apache.curator</groupId>
  <artifactId>curator-recipes</artifactId>
  <version>5.1.0</version>
</dependency>
```

### SDK常用功能

1. create()：创建一个ZooKeeper节点。
2. createContainers()：创建一个或多个ZooKeeper节点，同时创建父节点。
3. delete()：删除一个ZooKeeper节点。
4. checkExists()：检查一个ZooKeeper节点是否存在。
5. setData()：设置一个ZooKeeper节点的数据。
6. getData()：获取一个ZooKeeper节点的数据。
7. getChildren()：获取一个ZooKeeper节点的子节点。
8. getACL()：获取一个ZooKeeper节点的ACL权限列表。
9. setACL()：设置一个ZooKeeper节点的ACL权限列表。
10. inTransaction()：启动一个事务，用于执行多个操作。
11. watched()：监听一个ZooKeeper节点的变化。
12. blockUntilConnected()：阻塞直到与ZooKeeper服务器建立连接。
13. start()：启动CuratorFramework实例。
14. close()：关闭CuratorFramework实例。

### ZooKeeper 的监听器原理如下

1. 客户端注册监听器：客户端调用注册监听器的方法（比如 exists、getData、getChildren、addWatch），传递节点路径和监听器，然后组装成一个 Packet 对象发送给服务端。
2. 服务端存储监听器：服务端根据客户端的请求判断是否需要注册监听器，需要的话将节点路径和 ServerCnxn（表示客户端与服务端的连接）存储到服务端本地的 WatchManager 中，然后向客户端发送响应。
3. 客户端接收服务端的响应：客户端接收到服务端的响应后，将监听器注册到客户端本地的 ZKWatcherManager。
4. 服务端触发事件通知：对于客户端的 create、delete、setData 方法的调用会触发服务端向客户端发送事件通知。
5. 客户端接收服务端的事件通知：客户端接收到服务端的事件通知后执行监听器的回调方法。

#### 特点

1. exists 方法监听节点创建、删除、数据内容变化。
2. getData 方法监听节点删除、数据内容变化。
3. getChildren 方法监听节点的删除、子节点的创建或者删除。

##### 这三个方法注册的监听器具有如下三个特点：

1. 一次性 ： （对于 Standard 类型的监听器，即默认类型）无论是服务端还是客户端，一旦监听器被触发，都会从存储中删除该监听器。因此需要反复注册，可以有效减轻服务器的压力。
2. 客户端串行执行监听器的回调 ： 因为是从阻塞队列中取出，保证了顺序性。
3. 轻量 ： 客户端向服务端注册监听器的时候，并不会把客户端的监听器对象传递给服务端，仅仅是在客户端请求中使用boolean类型的属性进行标记。服务端的监听器事件通知非常简单，只会通知客户端发生了事件，不会说明具体的事件内容。

从 3.6.0 版本开始，增加了 addWatch 方法，支持注册持久监听器（监听节点的创建、删除、数据内容变化）、持久递归监听器（监听节点/子节点的创建、删除、数据内容变化）



## 初始化

```java
@Configuration
public class CuratorLockConfig {

  @Bean(initMethod = "start")
  public CuratorFramework curatorFramework(){
    //重试策略
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
    //connectString使用逗号分隔如: localhost:2181,localhost:2182
    CuratorFramework client = CuratorFrameworkFactory.newClient("localhost:2181", retryPolicy);
    // client.aclProvider(aclProvider)
    // .authorization("digest", (id0 + ":" + id1).getBytes()) //使用用户名/密码进行连接
    return client;
  } 
// CuratorFramework curatorFramework = CuratorFrameworkFactory.builder()
//   .aclProvider(aclProvider) // 权限的策略
//   .connectString(url)
//   .authorization("digest", (user + ":" + password).getBytes()) //使用用户名/密码进行连接
//   .retryPolicy(new ExponentialBackoffRetry(1000, 3)) //重试策略  初始休眠时间为 1000ms, 最大重试次数为 3
//   .build();
// curatorFramework.start();
}
```



### 基础使用

```java
//创建一个节点
String string = curatorClient.create().forPath("/my");
System.out.println(string);
//获取节点列表
List<String> strings = curatorClient.getChildren().forPath("/");
//建立一个节点并储存数据
String string = curatorClient.create().forPath("/my/fun2", "asd".getBytes());
//设置数据
Stat stat = curatorClient.setData().forPath("/my/fun2", "123".getBytes());
//判断是否存在，stat==null即不存在
Stat stat = curatorFramework.checkExists().forPath("/a");
//删除节点
curatorClient.delete().guaranteed().deletingChildrenIfNeeded().forPath("/a");

// 创建持久化节点
curatorFramework.create()
  .creatingParentContainersIfNeeded() // 如果指定节点的父节点不存在，则会自动级联创建父节点
  .withMode(CreateMode.PERSISTENT) // CreateMode.EPHEMERAL 临时节点
  .forPath(path, "=== CONTENT ===".getBytes());

// 创建持久化顺序节点
curatorFramework.create()
  .creatingParentContainersIfNeeded() // 如果指定节点的父节点不存在，则会自动级联创建父节点
	.withMode(CreateMode.PERSISTENT_SEQUENTIAL) // CreateMode.EPHEMERAL_SEQUENTIAL 临时顺序节点
  .forPath(path + "/i-", "persistent seq data".getBytes());

// 删除节点
curatorFramework.delete().forPath(path);

// 级联删除子节点
curatorFramework.delete().guaranteed().deletingChildrenIfNeeded().forPath(path);
```

### 加锁

InterProcessMutex：分布式可重入排它锁;    此锁可以重入，但是重入几次需要释放几次;

​	原理: InterProcessMutex通过在zookeeper的某路径节点下创建临时序列节点来实现分布式锁，即每个线程（跨进程的线程）获取同一把锁前，都需要在同样的路径下创建一个节点，节点名字由uuid + 递增序列组成。而通过对比自身的序列数是否在所有子节点的第一位，来判断是否成功获取到了锁。当获取锁失败时，它会添加watcher来监听前一个节点的变动情况，然后进行等待状态。直到watcher的事件生效将自己唤醒，或者超时时间异常返回

InterProcessSemaphoreMutex：分布式排它锁;  不可重入的互斥锁，也就意味着即使是同一个线程也无法在持有锁的情况下再次获得锁，所以需要注意，不可重入的锁很容易在一些情况导致死锁;

InterProcessReadWriteLock：分布式读写锁;   读锁和读锁不互斥，只要有写锁就互斥;

InterProcessMultiLock：将多个锁作为单个实体管理的容器;     多重共享锁;

InterProcessSemaphoreV2:   共享信号量;  当然可以一次获取多个信号量;  



```java
//使用
@Resource
private CuratorFramework curatorFramework;

// curator框架实现分布式锁
@Override
public boolean deduckStock(Long goodsId, Integer num) {
  //  mutex.acquire(); 不支持可重入 == 会造成死锁
  // InterProcessSemaphoreMutex mutex = new InterProcessSemaphoreMutex(curatorClient, "/lock_02/fun6");
  
  InterProcessMutex lock = new InterProcessMutex(curatorFramework,"/mylock");
  
  // InterProcessReadWriteLock lock = new InterProcessReadWriteLock(curatorFramework,"/mylock"); 
  // InterProcessLock readLock()  //读锁
  // InterProcessLock writeLock()  //写锁
  
  // 获取锁
  if(lock.acquire(10, TimeUnit.SECONDS)){
    try {
      //获取锁成功后开始执行业务
      return true;
    } finally {
      //释放锁资源
      lock.release();
    }
  }
  return false;
}
```

### 监听

CuratorCache   节点事件包括创建、设值、删除等，对应的curator方法如下：

1. forCreates() ：创建
2. forChanges() ：设值
3. forCreatesAndChanges() ：创建和设值
4. forDeletes() ：删除
5. forAll() ：所有事件

**默认情况下，listener监听整个子树（指定节点及其子节点）的事件，如下表所示：**

| forCreates()             | forChanges() | forCreatesAndChanges() | forDeletes() | forAll() |      |
| ------------------------ | ------------ | ---------------------- | ------------ | -------- | ---- |
| 创建                     | Y            |                        | Y            |          | Y    |
| 设值                     |              | Y                      | Y            |          | Y    |
| 创建子节点               | Y            |                        | Y            |          | Y    |
| 为子节点设值             |              | Y                      | Y            |          | Y    |
| 创建子节点并为子节点设值 |              | Y                      | Y            |          | Y    |
| 删除子节点               |              |                        |              | Y        | Y    |
| 删除节点                 |              |                        |              | Y        | Y    |

**如果指定了  SINGLE_NODE_CACHE 选项，则只监听单个节点的事件，如下表所示：**

| forCreates()             | forChanges() | forCreatesAndChanges() | forDeletes() | forAll() |      |
| ------------------------ | ------------ | ---------------------- | ------------ | -------- | ---- |
| 创建                     | Y            |                        | Y            |          | Y    |
| 设值                     |              | Y                      | Y            |          | Y    |
| 创建子节点               |              |                        |              |          |      |
| 为子节点设值             |              |                        |              |          |      |
| 创建子节点并为子节点设值 |              |                        |              |          |      |
| 删除子节点               |              |                        |              |          |      |
| 删除节点                 |              |                        |              | Y        | Y    |

注：如果删除多级节点，会触发多次。



```java
//使用的是默认选项
CuratorCache build = CuratorCache.build(curatorClient, "/my");
//注释行使用的是 SINGLE_NODE_CACHE 选项
CuratorCache build2 = CuratorCache.build(curatorClient, "/my", CuratorCache.Options.SINGLE_NODE_CACHE);

CuratorCacheListener listener = CuratorCacheListener.builder().forInitialized(() -> {
	System.out.println("Initialized!");
})
  .forCreates(childData -> System.out.println("创建! " + childData))
  .forChanges((childData, childData1) -> System.out.println("变更! " + childData + ", " + childData1))
  .forCreatesAndChanges((childData, childData1) -> System.out.println("创建+变更! " + childData + ", " + childData1)).forDeletes(childData -> System.out.println("删除! " + childData))
  .forAll((type, childData, childData1) -> System.out.println("All-所有操作! " + type + ", " + childData + ", " + childData1))
  .build();

build.listenable().addListener(listener);
build.start();
```

### 选主（Leader Election）

```java
LeaderSelectorListener listener = new LeaderSelectorListenerAdapter()
{
    public void takeLeadership(CuratorFramework client) throws Exception
    {
     	// this callback will get called when you are the leader do whatever 
      // leader work you need to and only exit this method when you want to relinquish leadership
      // 当您是领导者并执行您需要的任何领导者工作时，将调用此回调，并且只有当您想要放弃领导权时才退出此方法
    }
}

LeaderSelector selector = new LeaderSelector(client, path, listener);
selector.autoRequeue();  // 不是必需的，但这是您可能期望的行为
selector.start();
```

