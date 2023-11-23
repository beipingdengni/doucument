



```
#根据id删除
POST /indexname/_delete_by_query
{
  "query": { 
    "match": {
      "id": "100000"
      
    }
  }
}
```

