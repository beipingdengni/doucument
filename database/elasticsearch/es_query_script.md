## Script update

```

```





## Script Sort

### 查询DSL

```json
{
    "from":0,
    "size":10,
    "query":{
        "match_all": {}
    },
    "sort":[
        {
            "_script":{
                "script":{
                    "source":"params.sortMap[doc['gender'].value]",
                    "lang":"painless",
                    "params":{
                        "sortMap":{
                            "Male":1,
                            "Female":2
                        }
                    }
                },
                "type":"number",
                "order":"asc"
            }
        },
        {
            "_id":{
                "order":"desc"
            }
        }
    ]
}
```

### java sort

```java
// 定义一个Map, key为gender字段值，value为分数
Map<String, Integer> sortMap = new HashMap<>(2);
int sort = 1;
sortMap.put("Male", sort++);
sortMap.put("Female", sort++);

// 编写painless脚本
String scriptText ="params.sortMap[doc['gender'].value]";
Map<String, Object> params = new HashMap<>(1);
params.put("sortMap", sortMap);
Script script = new Script(ScriptType.INLINE, "painless", scriptText,params);
ScriptSortBuilder genderSortBuilder = SortBuilders.scriptSort(script,ScriptSortBuilder.ScriptSortType.NUMBER).order(SortOrder.ASC);

// 构造查询器
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
// 先按性别排
searchSourceBuilder.sort(genderSortBuilder);
// 再按id降序排
searchSourceBuilder.sort("_id", SortOrder.DESC);
```

