# http连接

xml

```xml
<dependency>
  <groupId>org.elasticsearch.client</groupId>
  <artifactId>elasticsearch-rest-high-level-client</artifactId>
  <version>7.8.1</version>
</dependency>
```

映入代码，密码

```java
// 没有密码
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
RestHighLevelClient client = new RestHighLevelClient(builder);

//  设置用户名以及密码
CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
UsernamePasswordCredentials usernamePasswordCredentials = new UsernamePasswordCredentials("elastic", "elastic");
credentialsProvider.setCredentials(AuthScope.ANY, usernamePasswordCredentials);

RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"))
									.setHttpClientConfigCallback(httpClientBuilder -> httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider));
RestHighLevelClient client = new RestHighLevelClient(builder);
```

