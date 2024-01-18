LRU

> LRU（The Least Recently Used）是最经典的一款缓存淘汰算法，其原理是 ：如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很低，当数据所占据的空间达到一定阈值时，这个最少被访问的数据将被淘汰掉

LFU

> LFU（Least frequently used）即**最不频繁访问**，其原理是：如果一个数据在近期被高频率地访问，那么在将来它被再访问的概率也会很高，而访问频率较低的数据将来很大概率不会再使用

[【深入解析Redis的LRU与LFU算法实现】](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247560043&idx=2&sn=48494925dbd6f19712c346156c4c3559&chksm=fbb06295ccc7eb838638af3646d6aa9918d0fcd5f4a0c95bacb71380f6a3cbe2101596250ba2&scene=27)



guava cache参考：https://blog.51cto.com/u_5650011/5394827

### CacheLoader的方式：

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
...
try {
  return graphs.get(key);
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}

...
//或者使用下面方法，不抛出异常
return graphs.getUnchecked(key);
```



```java
public LoadingCache<String, String> caches = CacheBuilder
  .newBuilder().maximumSize(100)
  .expireAfterWrite(100, TimeUnit.SECONDS) // 根据写入时间过期
  .build(new CacheLoader<String, String>() {
    @Override
    public String load(final String key) {
      return getSchema(key);
    }
        
    @Override
    public Map<String,String> loadAll(final Iterable<? extends String> keys) throws Exception {
      //com.google.common.collect.Lists
      ArrayList<String> keysList = Lists.newArrayList(keys);
      return getSchemas(keysList);
    }
  });

    private static Map<String,String> getSchemas(List<String> keys) {
        Map<String,String> map = new HashMap<>();

        //...
        System.out.println("loadall...");
        return map;
    }

    List<String> keys = new ArrayList<>();
    keys.add("key2");
    keys.add("key3");

    try {
      caches.getAll(keys);
    } catch (ExecutionException e1) {
      e1.printStackTrace();
    }
```

### callable方式：

```java
LoadingCache<String, String> cache = CacheBuilder
    .newBuilder().maximumSize(100)
    .expireAfterWrite(100, TimeUnit.SECONDS) // 根据写入时间过期
    .build(new CacheLoader<String, String>() {
        @Override
        public String load(String key) {
            return getSchema(key);
        }
});

private static String getSchema(String key) {
    System.out.println("load...");
    return key+"schema";
}

try {
    String value = cache.get("key4", new Callable<String>() {
            @Override
            public String call() throws Exception {
                System.out.println("i am callable...");
                return "i am callable...";
            }
      });
    System.out.println(value);
} catch (ExecutionException e1) {
    e1.printStackTrace();
}
//输出值：i am callable...
```

### 缓存删除

主动删除，见操作方法，删除单个、批量删除、删除所有

被动删除

- 超过最大个数删除：LRU+FIFO => 访问次数一样少的情况下使用FIFO
- 过期删除：访问时间过期、写入时间过期
- 引用删除：通过使用弱引用的键、或弱引用的值、或软引用的值，Guava Cache可以垃圾回收