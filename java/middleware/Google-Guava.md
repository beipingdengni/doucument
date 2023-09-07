### 谷歌实用工具包 google guava 

**通常来说，Guava Cache适用于：**
1. 你愿意消耗一些内存空间来提升速度。
2. 你预料到某些键会被查询一次以上。
3. 缓存中存放的数据总量不会超出内存容量。（Guava Cache是单个应用运行时的本地缓存。它不把数据存放到文件或外部服务器。如果这不符合你的需求，请尝试Memcached这类工具）
4. 如果你的场景符合上述的每一条，Guava Cache就适合你。
5. 如同范例代码展示的一样，Cache实例通过CacheBuilder生成器模式获取，但是自定义你的缓存才是最有趣的部分。
6. 注：如果你不需要Cache中的特性，使用ConcurrentHashMap有更好的内存效率——但Cache的大多数特性都很难基于旧有的ConcurrentMap复制，甚至根本不可能做到

**参考网站**

#### [Google Guava官方教程（中文版）](http://ifeve.com/google-guava/)

#### [githup 地址](https://github.com/google/guava)：https://github.com/google/guava

## [spring mvc 使用 guava 自定义缓存](http://www.cnblogs.com/gongxijun/p/5781108.html)
##### 项目中引入包

maven 引入包
----

    ``` xml
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.0</version>
            <!-- or, for Android: -->
            <version>23.0-android</version>
        </dependency>
    ```

gradle 映入
----

    ``` java
        dependencies {
            compile 'com.google.guava:guava:23.0'
            // or, for Android:
            compile 'com.google.guava:guava:23.0-android'
            }
    ```

1. **介绍guava缓存**

    > 简单的demo

    ``` java
        LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
                .maximumSize(1000)
                .expireAfterWrite(10, TimeUnit.MINUTES)  // 在添加十分钟后删除缓存
                .removalListener(MY_LISTENER) // 自行添加删除的监听器的实现
                .build(
                    new CacheLoader<Key, Graph>() {
                        public Graph load(Key key) throws AnyException {
                            return createExpensiveGraph(key);
                        }
                });
    ```

2. 介绍一下cache 参数的相关设置

