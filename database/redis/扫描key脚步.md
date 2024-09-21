

## 扫描key（查询大对象）

```sh
#!/bin/bash  

THRESHOLD=1000  
REDIS_HOST=localhost  
REDIS_PORT=6379  
REDIS_PASSWORD=  # 如果设置了密码，请填写  

redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD --scan --pattern '*' | while read -r key; do  
    type=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD TYPE "$key")  
    case "$type" in  
        list)  
            size=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD LLEN "$key")  
            if [ "$size" -gt "$THRESHOLD" ]; then  
                echo "Big list found: $key with $size elements"  
            fi  
            ;;  
        set)  
            # 对于集合，可以使用 SCARD 命令  
            size=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD SCARD "$key")  
            if [ "$size" -gt "$THRESHOLD" ]; then  
                echo "Big set found: $key with $size elements"  
            fi  
            ;;  
        zset)  
            # 对于有序集合，可以使用 ZCOUNT 命令计算范围大小，但这里我们简化为只计算所有元素  
            size=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT -a $REDIS_PASSWORD ZCOUNT "$key" "-inf" "+inf")  
            if [ "$size" -gt "$THRESHOLD" ]; then  
                echo "Big sorted set found: $key with $size elements"  
            fi  
            ;;  
        *)  
            # 其他类型（如字符串、哈希）可以根据需要添加检查  
            ;;  
    esac  
done
```

