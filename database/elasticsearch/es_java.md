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

## 检查索引是否存在

```java
try {    
  IndexTemplatesExistRequest request = new IndexTemplatesExistRequest("test-logs");    
  boolean exists = client.indices().existsTemplate(request, RequestOptions.DEFAULT);    
  System.out.println("模板是否存在:" + exists );
} catch (IOException e) {    
  e.printStackTrace();
}
```



## 创建索引、删除索引

```java

CreateIndexRequest request = new CreateIndexRequest("twitter");

// 请求配置setting
// request.settings(Settings.builder() 
//     .put("index.number_of_shards", 3)
//     .put("index.number_of_replicas", 2)
// );

// ===============================
// 请求配置mapping
{
  "properties": {
    "birthday": {
      "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
      "type": "date"
    },
    "address": {
      "type": "text"
    },
    "hight": {
      "type": "text"
    },
    "name": {
      "type": "keyword"
    },
    "age": {
      "type": "long"
    }
  }
}
//request.mapping("json",XContentType.JSON)
// 或者使用如下
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.startObject("properties");
    {
        builder.startObject("message");
        {
            builder.field("type", "text");
        }
        builder.endObject();
    }
    builder.endObject();
}
builder.endObject();
request.mapping(builder);

// ===============================
// 别名请求过滤 twitter_alias 别名，关联索引user，kimchy
// request.alias(new Alias("twitter_alias"));
// request.alias(new Alias("twitter_alias").filter(QueryBuilders.termQuery("user", "kimchy")));

// ===============================
// 请求配置 setting、mapping
String json = {
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "message": {
        "type": "text"
      }
    }
  },
  "aliases": {
    "twitter_alias": {}
  }
}
request.source(json,XContentType.JSON) // json 请求处理

// ===============================
request.setTimeout(TimeValue.timeValueMinutes(2));  // 超时配置
request.setMasterTimeout(TimeValue.timeValueMinutes(1)); // 连接主节点超时

// 同步
CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
// 异步 监听着（listener）
client.indices().createAsync(request, RequestOptions.DEFAULT, listener);

```

## 滚动索引

```java
try {    
	RolloverRequest request = new RolloverRequest("test-logs-write", null);    
  request.addMaxIndexAgeCondition(new TimeValue(7, TimeUnit.DAYS));    
  request.addMaxIndexDocsCondition(2);    
  request.addMaxIndexSizeCondition(new ByteSizeValue(5, ByteSizeUnit.GB));    
  RolloverResponse rolloverResponse = client.indices().rollover(request, RequestOptions.DEFAULT);
  System.out.println("设置滚动索引成功!");    
  System.out.println("isRolledOver:" + rolloverResponse.isRolledOver());
} catch (IOException e) {    
  log.error("IOException:{}", e);
}
```

## 写入数据

```java
try {    
  // 1、创建索引请求    
  IndexRequest request = new IndexRequest("test-logs-write", "_doc");    
  // 2、准备文档数据    
  Map<String, Object> jsonMap = new HashMap<>();    
  jsonMap.put("file_name", "财政部二月份文件");    
  jsonMap.put("table", "103_table-111");    
  jsonMap.put("update_time", System.currentTimeMillis());    
  request.source(jsonMap, XContentType.JSON).setRefreshPolicy(WriteRequest.RefreshPolicy.IMMEDIATE);    
  //3、发送请求    同步方式    
  IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);    
  System.out.println("添加数据成功!");    
  System.out.println("indexResponse:" + indexResponse);    
  System.out.println(indexResponse.status());
} catch (IOException e) {    
  log.error("出现异常:{}", e);
}
```



## 查询数据

```java
try {            
  // 1、创建search请求            
  SearchRequest searchRequest = new SearchRequest("test-logs-read");            
  searchRequest.types("_doc");            
  // 2、用SearchSourceBuilder来构造查询请求体 ,请仔细查看它的方法，构造各种查询的方法都在这。            
  SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();            
  sourceBuilder.from(0);            
  sourceBuilder.size(10);            
  sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));            
  sourceBuilder.sort(new FieldSortBuilder("update_time").order(SortOrder.DESC));            
  MultiMatchQueryBuilder matchQueryBuilder = QueryBuilders.multiMatchQuery("103", "table").fuzziness(Fuzziness.AUTO);            
  sourceBuilder.query(matchQueryBuilder);            
  //3. 高亮设置            
  HighlightBuilder highlightBuilder = new HighlightBuilder();            
  highlightBuilder.requireFieldMatch(false).field("table").preTags("<font color=red>").postTags("</font>");       sourceBuilder.highlighter(highlightBuilder);            
  //4.将请求体加入到请求中            
  searchRequest.source(sourceBuilder);            
  //5、发送请求            
  SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);            
  SearchHits hits = searchResponse.getHits();            
  SearchHit[] searchHits = hits.getHits();            
  System.out.println("检索数据成功!");            
  System.out.println(searchResponse);            
  System.out.println(hits);            
  System.out.println("num:" + searchHits.length);            
  for (SearchHit hit : searchHits) {
    //                String source = hit.getSourceAsString();
    //                System.out.println(source);                
    System.out.println(hit);            
  }        
} catch (IOException e) {            
  log.error("出现异常:{}", e);        
}
```

