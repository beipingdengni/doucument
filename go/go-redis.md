## go redis

### 安装redis

```bash
go get -u github.com/go-redis/redis
```

### 操作扫描数据

```go
package main

import (
	"fmt"
	"strings"
	"time"
	"github.com/go-redis/redis"
	// _ "github.com/go-sql-driver/mysql"
)

// 初始化
func init() {
}

func main() {
	redis_scan()
}

func redis_scan() {

	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "123456", //  no password set
		DB:       0,        // use default DB
	})

	var cursor uint64
	var n int
	var del_num int = 0
	for {
		var keys []string
		var err error
		//*扫描所有key，每次1000条
		keys, cursor, err = client.Scan(cursor, "*my_cs_key*", 1000).Result()
		if err != nil {
			panic(err) // 错误终止
		}
    // 单条处理查询
		n += len(keys)
		for _, key := range keys {
			// 返回的key字符串中是否包含
      if strings.Contains(key, "im_app:my_cs_key:dialog:111111") {
				continue // 不处理
			}
      // 查询key，过期时间
			ttl, err := client.TTL(key).Result()
			if err != nil {
				fmt.Printf("nill")
			}
      // 过期时间为 -1 永远不过期
			is_time := ttl == -1*time.Second
			if is_time {
				fmt.Printf("key: %v ====> %v\n", key, ttl)
			}
			// 删除不设置过期数据
			if strings.Contains(key, "im_app:my_cs_key:dialog:123123") && is_time {
				_, err := client.Del(key).Result()
				if err != nil {
					fmt.Printf("del nill %v\n", err.Error())
				}
				del_num += 1
			}
		}
    // 返回数据处理完成，但此时，指针游标为0，说明已经扫描完成
		if cursor == 0 {
			break // 不处理后续的数据
		}
	}
	fmt.Printf("del error num %v \n", del_num)
}
```

