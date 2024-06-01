## 官方网站：https://neo4j.com/docs/

在图计算中，基本的数据结构表达就是：

```
G=(V, E) 
V=vertex（节点） 
E=edge（边）
```

###  创建关系和处理

Cypher语言，便携查询语句

```shell
# 创建节点
CREATE (n:Person { name: 'Andres', title: 'Developer' }) return n;
CREATE (n:Person { name: 'Vic', title: 'Developer' }) return n;
# 创建关系
match(n:Person{name:"Vic"}),(m:Person{name:"Andres"}) create (n)-[r:Friend]-///#>(m) return r;
match(n:Person{name:"Vic"}),(m:Person{name:"Andres"}) create (n)<-[r:Friend]-(m) return r;

查询
sql = (p:Person)-[rel:LIKES {type: 'Graphs'}]->(t:Technology)
```



参考博客使用：

https://zhuanlan.zhihu.com/p/589336779