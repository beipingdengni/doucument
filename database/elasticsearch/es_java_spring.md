

### 查询接口

#### 构建请求连接

```java
public static RestHighLevelClient getInstance(String hosts, String userName, String password) {
        String[] localHosts = hosts.split(",");
        HttpHost[] httpHosts = new HttpHost[localHosts.length];
  			// 分布式
        for (int i = 0; i < localHosts.length; i++) {
            URL hostUrl = new URL(localHosts[i]);
            httpHosts[i] = new HttpHost(hostUrl.getHost(), hostUrl.getPort(), hostUrl.getProtocol());
        }
				// 配置用户和密码
        CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        Credentials credentials = new UsernamePasswordCredentials(userName, password);
        credentialsProvider.setCredentials(AuthScope.ANY, credentials);

        RestClientBuilder builder = RestClient.builder(httpHosts).setRequestConfigCallback(requestConfigBuilder -> {
            requestConfigBuilder.setConnectTimeout(-1);
            requestConfigBuilder.setSocketTimeout(-1);
            requestConfigBuilder.setConnectionRequestTimeout(-1);
            return requestConfigBuilder;
        }).setHttpClientConfigCallback(httpClientBuilder -> {
            httpClientBuilder.disableAuthCaching();
            return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
        }).setMaxRetryTimeoutMillis(60 * 1000);

        client = new RestHighLevelClient(builder);
        return client;
    }
```



#### 配置xml

```xml
org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate




<bean name="elasticsearchTemplate" class="org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate">
  <!-- 数据库连接 -->
  <constructor-arg name="client" ref="client" /> 
  <!-- 自定义返回类 -->
  <constructor-arg name="resultsMapper" ref="esCustomizeResultMapper" /> 
</bean>

```



### 返回结果处理

```java
org.springframework.data.elasticsearch.core.DefaultResultMapper
	//方法 mapResults // 对返回，结果处理
```

