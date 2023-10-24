

## 滚动查询ES（查询、使用scrollId滚动、删除快照）

```shell
### 查询, 返回值中包含：_scroll_id (返回最外层)
POST http://localhost:9200/cs_event_ticket_v1/_search?scroll=5m
Content-Type: application/json;charset=UTF-8

{"query":{"match_all":{}}}

### 滚动
POST http://localhost:9200/_search/scroll
Content-Type: application/json;charset=UTF-8

{"scroll":"1m","scroll_id":"DnF1ZXJ5V......"}

### 手动删除scroll_id的方法
DELETE http://localhost:9200/_search/scroll
Content-Type: application/json;charset=UTF-8

{"scroll_id": "DnF1ZXJ5V......"}


### DELETE /_search/scroll/_all 全部删除
```

## Java rest level 查询

### client连接

```java
@Bean
    //当前es相关的配置存在则实例化RestHighLevelClient,如果不存在则不实例化RestHighLevelClient
    @ConditionalOnBean(value = ElasticsearchRuntimeEnvironment.class)
    public RestHighLevelClient restHighLevelClient(){

        //es地址，以逗号分隔
        String nodes = esRuntimeEnvironment.getAddress();
        nodes = nodes.contains("http://") ? nodes.replace("http://","") : nodes;
        //es密码
        String password = esRuntimeEnvironment.getPassword();
        String scheme = esRuntimeEnvironment.getScheme();
        List<HttpHost> httpHostList = new ArrayList();
        //拆分es地址
        for(String address : nodes.split(",")){
            int index = address.lastIndexOf(":");
            httpHostList.add(new HttpHost(address.substring(0, index),Integer.parseInt(address.substring(index + 1)),scheme));
        }
        //转换成 HttpHost 数组
        HttpHost[] httpHosts = httpHostList.toArray(new HttpHost[httpHostList.size()]);

        //构建连接对象
        RestClientBuilder builder = RestClient.builder(httpHosts);

        //使用账号密码连接
        if ( StringUtils.isNotEmpty(password)){
            String username = esRuntimeEnvironment.getUsername();
            CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
            credentialsProvider .setCredentials(AuthScope.ANY,new UsernamePasswordCredentials(username,password));
            builder.setHttpClientConfigCallback(f->f.setDefaultCredentialsProvider(credentialsProvider));
        }

        // 异步连接延时配置
        builder.setRequestConfigCallback(requestConfigBuilder -> {
            requestConfigBuilder.setConnectTimeout(esRuntimeEnvironment.getConnectTimeout());
            requestConfigBuilder.setSocketTimeout(esRuntimeEnvironment.getSocketTimeout());
            requestConfigBuilder.setConnectionRequestTimeout(esRuntimeEnvironment.getConnectionRequestTimeout());
            return requestConfigBuilder;
        });

        // 异步连接数配置
        builder.setHttpClientConfigCallback(httpClientBuilder -> {
            httpClientBuilder.setMaxConnTotal(esRuntimeEnvironment.getMaxConnectNum());
            httpClientBuilder.setMaxConnPerRoute(esRuntimeEnvironment.getMaxConnectPerRoute());
            return httpClientBuilder;
        });

        return new RestHighLevelClient(builder);
    }
```



### 滚动查询

```java
// 创建Client连接对象
String[] ips = {"localhost:9200"};
HttpHost[] httpHosts = new HttpHost[ips.length];
for (int i = 0; i < ips.length; i++) {
  httpHosts[i] = HttpHost.create(ips[i]);
}
RestHighLevelClient  client = new RestHighLevelClient(RestClient.builder(httpHosts));

/**
 * 滚动查询数据
 * @param indexName
 * @param utime
 */
public List<String> scrollSearchAll(String indexName, String utime) throws IOException{
 
    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
	  //946656000为2000-01-01 00:00:00
    boolQueryBuilder.must(QueryBuilders.rangeQuery("utime").lt(utime).gt("946656000"));
 
    //builder
    SearchSourceBuilder builder = new SearchSourceBuilder()
            .query(boolQueryBuilder)
            .size(500);
 
    // 构建SearchRequest
    SearchRequest searchRequest = new SearchRequest();
    searchRequest.indices(indexName);
    searchRequest.source(builder);
 		// 快照存储时间
    Scroll scroll = new Scroll(new TimeValue(600000));
    searchRequest.scroll(scroll);
 		// 查询搜索
    SearchResponse searchResponse = restHighLevelClient.search(searchRequest);
 		// 获取scrollId 值
    String scrollId = searchResponse.getScrollId();
    SearchHit[] hits = searchResponse.getHits().getHits();
 		// 存储返回值
    List<String> resultSearchHit = new ArrayList<>();
 
    while (ArrayUtils.isNotEmpty(hits)) {
        for (SearchHit hit : hits) {
            log.info("准备删除的数据hit:{}", hit);
            resultSearchHit.add(hit.getId());
        }
        // 再次发送请求,并使用上次搜索结果的ScrollId
        SearchScrollRequest searchScrollRequest = new SearchScrollRequest(scrollId);
        searchScrollRequest.scroll(scroll); //时间加上
        SearchResponse searchScrollResponse = restHighLevelClient.searchScroll(searchScrollRequest);
 				// 查询后滚动返回的新scrollId
        scrollId = searchScrollResponse.getScrollId();
        hits = searchScrollResponse.getHits().getHits();
    }
    // 及时清除es快照，释放资源
    ClearScrollRequest clearScrollRequest = new ClearScrollRequest();
    clearScrollRequest.addScrollId(scrollId);
    restHighLevelClient.clearScroll(clearScrollRequest);
 
    return resultSearchHit;
}
```

