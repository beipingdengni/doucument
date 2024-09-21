## redis 详细解释

https://blog.csdn.net/weixin_43746433/article/details/118729449



SET 操作

```shell
# 更新一个简单的键值对
SET mykey "new_value"
# 更新哈希表中的字段
HSET myhash field1 "new_value"
# 添加或更新集合中的成员
SADD myset member1 member2
# 添加或更新有序集合中的成员
ZADD myzset 1 member1 2 member2
```

监控和运维

```shell
redis-cli -h 10.28.1.1 -p 6379 -a 1212112

# 查询大key
redis-cli -h 10.28.1.1 -p 6379 -a 1212112 --bigkeys

# 监控详情
redis-cli -h 10.28.1.1 -p 6379 -a 1212112 info
```



清除数据

```shell
# 清除当前数据库中的所有键
FLUSHDB
# 清除所有数据库中的所有键
FLUSHALL
```

