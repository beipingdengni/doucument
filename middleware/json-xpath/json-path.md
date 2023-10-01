# jsonpath

### 语法

下表将列举所有支持的语法，并对XPath进行比较：

| XPath | JsonPath           | 说明                                                         |
| ----- | ------------------ | ------------------------------------------------------------ |
| `/`   | `$`                | 文档根元素                                                   |
| `.`   | `@`                | 当前元素                                                     |
| `/`   | `.`或`[]`          | 匹配下级元素                                                 |
| `..`  | `N/A`              | 匹配上级元素，JsonPath不支持此操作符                         |
| `//`  | `..`               | 递归匹配所有子元素                                           |
| `*`   | `*`                | 通配符，匹配下级元素                                         |
| `@`   | `N/A`              | 匹配属性，JsonPath不支持此操作符                             |
| `[]`  | `[]`               | 下标运算符，根据索引获取元素，**XPath索引从1开始，JsonPath索引从0开始** |
| `|`   | `[,]`              | 连接操作符，将多个结果拼接成数组返回，可以使用索引或别名     |
| `N/A` | `[start:end:step]` | 数据切片操作，XPath不支持                                    |
| `[]`  | `?()`              | 过滤表达式                                                   |
| `N/A` | `()`               | 脚本表达式，使用底层脚本引擎，XPath不支持                    |
| `()`  | `N/A`              | 分组，JsonPath不支持                                         |

注意：

- JsonPath的索引从0开始计数
- JsonPath中字符串使用单引号表示，例如:`$.store.book[?(@.category=='reference')]`中的`'reference'`



#### 案例使用

接下来我们看一下如何对这个文档进行解析：

| XPath                  | JsonPath                                   | Result                                   |
| ---------------------- | ------------------------------------------ | ---------------------------------------- |
| `/store/book/author`   | `$.store.book[*].author`                   | 所有book的author节点                     |
| `//author`             | `$..author`                                | 所有author节点                           |
| `/store/*`             | `$.store.*`                                | store下的所有节点，book数组和bicycle节点 |
| `/store//price`        | `$.store..price`                           | store下的所有price节点                   |
| `//book[3]`            | `$..book[2]`                               | 匹配第3个book节点                        |
| `//book[last()]`       | `$..book[(@.length-1)]`，或 `$..book[-1:]` | 匹配倒数第1个book节点                    |
| `//book[position()<3]` | `$..book[0,1]`，或 `$..book[:2]`           | 匹配前两个book节点                       |
| `//book[isbn]`         | `$..book[?(@.isbn)]`                       | 过滤含isbn字段的节点                     |
| `//book[price<10]`     | `$..book[?(@.price<10)]`                   | 过滤`price<10`的节点                     |
| `//*`                  | `$..*`                                     | 递归匹配所有子节点                       |

### 函数

| 函数       | 描述                     | 输出     |
| ---------- | ------------------------ | -------- |
| `min()`    | 提供数字数组的最小值     | `Double` |
| `max()`    | 提供数字数组的最大值     | `Double` |
| `avg()`    | 提供数字数组的平均值     | `Double` |
| `stddev()` | 提供数字数组的标准偏差值 | `Double` |
| `length()` | 提供数组的长度           | Integer  |

### 过滤器运算符

| 操作符  | 描述                                     |
| ------- | ---------------------------------------- |
| `==`    | left等于right（注意1不等于'1'）          |
| `!=`    | 不等于                                   |
| `<`     | 小于                                     |
| `<=`    | 小于等于                                 |
| `>`     | 大于                                     |
| `>=`    | 大于等于                                 |
| `=~`    | 匹配正则表达式[?(@.name =~ /foo.*?/i)]   |
| `in`    | 左边存在于右边 [?(@.size in ['S', 'M'])] |
| `nin`   | 左边不存在于右边                         |
| `size`  | （数组或字符串）长度                     |
| `empty` | （数组或字符串）为空                     |



## java 使用

pom xml

```xml
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.4.0</version>
</dependency>
```

基本sdk

```java
// 读取json文件
String path =System.getProperty("user.dir")+File.separator+"testdata"+File.separator+"test.json";
String jsonString = FileUtils.readFileToString(new File(path),"utf-8");
// json上下文构建
ReadContext context = JsonPath.parse(json);

// 获取数据
JsonPath.read(json,"$.store.book[1].author");
// 或
context.read("$.store.book[1].author");

// 数组获取
List<Object> prices = context.read("$.store.book[?(@.price>10)]");

// 对象获取
Configuration conf = Configuration.defaultConfiguration();
List<Map<String, Object>> expensiveBooks = JsonPath
  .using(conf)
  .parse(json)
  .read("$.store.book[?(@.price > 10)]", List.class)
  
// 泛型类型
TypeRef<List<String>> typeRef = new TypeRef<List<String>>(){};

TypeRef<List<String>> typeRef = new TypeRef<List<String>>(){};
List<String> titles = JsonPath.parse("json 内容").read("$.store.book[*].title", typeRef);

// 自定义过滤
// Filter filter = Filter.filter(Criteria.where("isbn").exists(true).and("category").in("fiction", "JavaWeb"));
Filter cheapFictionFilter = filter(
   where("category").is("fiction").and("price").lte(10D)
  // where("foo").exists(true)).or(where("bar").exists(true)
);
List<Map<String, Object>> books = parse(json).read("$.store.book[?]", cheapFictionFilter);


// 读取JSON得到某个具体值（推荐使用这种方法，一次解析多次调用）
Object document = Configuration.defaultConfiguration().jsonProvider().parse(json);
String author0 = JsonPath.read(document, "$.store.book[0].author");
String author1 = JsonPath.read(document, "$.store.book[1].author");

```

