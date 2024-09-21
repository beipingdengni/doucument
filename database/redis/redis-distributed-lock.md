

## Redis set 指令

```
SET key value [NX | XX] [GET] [EX seconds | PX milliseconds |
```

## Options（选择）

The `SET` command supports a set of options that modify its behavior:

- `EX` *seconds* -- 设置指定的过期时间，以秒为单位
- `PX` *milliseconds* --  设置指定的过期时间，以毫秒为单位
- `EXAT` *timestamp-seconds* -- 设置密钥过期的指定 Unix 时间（以秒为单位）
- `PXAT` *timestamp-milliseconds* -- 设置密钥过期的指定 Unix 时间（以毫秒为单位）
- `NX` -- 仅当key不存在设置key
- `XX` --  仅当key存在设置key
- `KEEPTTL` -- Retain the time to live associated with the key.
- `GET` -- 返回存储在 key 处的旧字符串，如果 key 不存在则返回 nil。`SET`如果存储在 key 中的值不是字符串，则会返回错误并中止。

注意：由于`SET`命令选项可以替换`SETNX`， `SETEX`， `PSETEX`，`GETSET`，因此在 Redis 的未来版本中这些命令可能会被弃用并最终删除

## Return（返回）

Simple string reply: `OK`如果`SET`执行正确

Null reply: `(nil)`如果`SET`由于用户指定了`NX`或`XX`选项但不满足条件而未执行操作。

如果命令与`GET`选项一起发出，则上述内容不适用。`SET`无论是否实际执行，它都会回复如下：

​	Bulk string reply:  存储在 key 处的旧字符串值

​	Null reply: `(nil)`如果key不存在

## 分布式锁

### redisson

#### Pom

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson</artifactId>
  <version>3.11.5</version>
</dependency>
```

#### 配置类

```java
@Configuration
public class RedissonConfig {
    @Resource
    private RedisProperties redisProperties;
    @Bean
    public RedissonClient redissonClient(){
       RedissonClient redissonClient;
        Config config = new Config();
        String url = "redis://"+redisProperties.getHost()+":"+redisProperties.getPort();
        //单机redis，如果是redis集群使用config.useClusterServers().addNodeAddress(...)
        config.useSingleServer().setAddress(url).setPassword(null).setDatabase(redisProperties.getDatabase());
        try{
            redissonClient = Redisson.create(config);
            return redissonClient;
        }catch(Exception e){
            e.printStackTrace();
            return null;
        }
    }
}
```

#### 分布式锁实现

```java
@Resource
private RedissonClient redissonClient;

// redisson框架实现分布式锁
@Override
public boolean deduckStock(Long goodsId, Integer num) {
  try {
    //获取锁
    RLock lock = redissonClient.getLock("/myLock");
    //设置超时时间
    lock.lock(10L,TimeUnit.SECONDS);
    //获取锁成功后开始执行业务
    return true
  } finally {
    //释放锁资源
    lock.unlock();
  }
  return false;
}
```

### spring

#### pom

```xml
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 配置类

```
@Configuration
public class RedisLockConfig {

    @Bean
    public RedisLockRegistry redisLockRegistry(RedisConnectionFactory redisConnectionFactory) {
        //registryKey全局分布式锁key前缀
        return new RedisLockRegistry(redisConnectionFactory, "mylock");
    }
}
```

#### 分布式锁实现



```java
/**
 * RedisStringCommands.SetOption.SET_IF_ABSENT：当key不存在时才会set
 * RedisStringCommands.SetOption.SET_IF_PRESENT：当key存在时才会set
 * RedisStringCommands.SetOption.UPSERT：当key存在时更新value，key不存在时set
 * 参考文档：https://docs.spring.io/spring-data/redis/docs/3.1.4/api/org/springframework/data/redis/connection/RedisStringCommands.SetOption.html#SET_IF_ABSENT
 */

@Resource
private RedisLockRegistry redisLockRegistry;

//spring integration redis分布式锁实现
@Override
public boolean deduckStock(Long goodsId, Integer num) {
  Lock lock = redisLockRegistry.obtain("lock:" + goodsId);
  //尝试获取锁
  if (lock.tryLock(10, TimeUnit.SECONDS))
  try {
    //获取锁成功后开始执行业务
    return true;
  } finally {
    //释放锁
    lock.unlock();
  }
  return false;
}
```

