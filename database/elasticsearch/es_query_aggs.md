## 查询

Elasticsearch中的聚合，包含多种类型，最常用的两种，一个叫`桶`，一个叫`度量`

### **桶（bucket）**

- Date Histogram Aggregation：根据日期阶梯分组，例如给定阶梯为周，会自动每周分为一组
- Histogram Aggregation：根据数值阶梯分组，与日期类似 (你需要指定一个阶梯值（interval）来划分阶梯大小)
  - 计算公式如下：bucket_key = Math.floor((value - offset) / interval) * interval + offset
  - 案例：以一件价格450的商品为例，`value：当前数据的值450，offset：起始偏移量，默认为0，interval：阶梯间隔，比如200，`因此你得到的：key = Math.floor((450 - 0) / 200) * 200 + 0 = 400
- Terms Aggregation：根据词条内容分组，词条内容完全匹配的为一组
- Range Aggregation：数值和日期的范围分组，指定开始和结束，然后按段分组

bucket aggregations 只负责对数据进行分组，并不进行计算，因此往往bucket中往往会嵌套另一种聚合：metrics aggregations即度量

### **度量（metrics）**

分组完成以后，我们一般会对组中的数据进行聚合运算，例如求平均值、最大、最小、求和等，这些在ES中称为`度量`

比较常用的一些度量聚合方式：

- Avg Aggregation：求平均值
- Max Aggregation：求最大值
- Min Aggregation：求最小值
- Percentiles Aggregation：求百分比
- Stats Aggregation：同时返回avg、max、min、sum、count等
- Sum Aggregation：求和
- Top hits Aggregation：求前几
- Value Count Aggregation：求总数



> **注意**：在ES中，需要进行聚合、排序、过滤的字段其处理方式比较特殊，因此不能被分词。这里我们将color和make这两个文字类型的字段设置为keyword类型，这个类型不会被分词，将来就可以参与聚合





## 使用案例

### DSL(阶梯分桶Histogram Aggregation)

```json
GET /cars/_search
{
  "size":0,
  "aggs":{
    "price":{
      "histogram": {
        "field": "price",
        "interval": 5000,
        "min_doc_count": 1  // 增加一个参数min_doc_count为1，过滤文档数量为0的桶
      }
    }
  }
}
```

### DSL(范围分桶Range Aggregation)

```json
GET /cars/_search
{
  "size": 0, 
  "aggs": {
    "price_range": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 0,
            "to": 20000
          }
        ]
      },
      "aggs":{
        "maker":{
            "terms":{
                "field":"make"
            },
            "aggs":{
            "avg_price": { 
               "avg": {
                  "field": "price" 
               }
            }
		  }
        }
      }
    }
  }
}
```

### DSL(日期分桶date_histogram)

```json
GET /cars/_search
{
  "size":0,
  "aggs" : {
      "date" : {
          "date_histogram" : {
              "field" : "sold",
              "interval" : "1M",
              "format" : "yyyy-MM",
              "time_zone": "-01:00",
              "min_doc_count": 1
          }
      }
  }
}
```



### DSL

```json
POST /es-customer/_search
{
  "size" : 0,
  "aggs" : {
      "days" : {
        "date_histogram": {
          "field": "createTime",
          "interval": "day"
        },
        "aggs": {
          "distinct_name" : {
              "cardinality" : {
                "field" : "firstName"
              }
          }
        }
      }
  }
}
```

### java

> 使用spring es data

```java
@Test
public void testAggAndDistinct(){
  //获取注解，通过注解可以得到 indexName 和 type
  Document document = Customer.class.getAnnotation(Document.class);
  // dateHistogram  Aggregation 是时间柱状图聚合，按照天来聚合 ， dataAgg 为聚合结果的名称，createTime 为字段名称
  // cardinality 用来去重
  SearchQuery searchQuery  = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withSearchType(SearchType.QUERY_THEN_FETCH)
    .withIndices(document.indexName()).withTypes(document.type())
    .addAggregation(AggregationBuilders.dateHistogram("dataAgg").field("createTime").dateHistogramInterval(DateHistogramInterval.DAY)
                    .subAggregation(AggregationBuilders.cardinality("nameAgg").field("firstName")))
    .build();

  // 聚合的结果
  Aggregations aggregations = elasticsearchTemplate.query(searchQuery, response -> response.getAggregations());
  Map<String, Aggregation> results = aggregations.asMap();
  Histogram histogram = (Histogram) results.get("dataAgg");
  // 将bucket list 转换成 map ， key -> 名字   value-> 出现次数
  histogram.getBuckets().stream().forEach(t->{
    Histogram.Bucket histogram1 = t;
    System.out.println(histogram1.getKeyAsString());
    Cardinality cardinality = histogram1.getAggregations().get("nameAgg");
    System.out.println(cardinality.getValue());
  });
}
```

