## 安装：viper

```shell
go get -u github.com/spf13/viper

## 
github.com/fsnotify/fsnotify # 文件夹或文件监控变化
github.com/gin-gonic/gin # web服务
```



### 用例参考：config.yaml

```yaml
app:
  app1:
    timeout: 120
    rpc: true
    compatible: true
 
mysql:
  host: localhost
  port: 3306
  user: root
  password: root
  database: test
  
redis:
  host: localhost
  port: 6379
  dbname: 0
```

### 配置读取: config.go

```go
package config
 
import (
	"fmt"
	"github.com/fsnotify/fsnotify"
	"github.com/spf13/viper"
)
 
type Config struct {
	App   App
	MySQL MySQL
	Redis Redis
}
 
type App struct {
	App1 App1
}
 
type App1 struct {
	Timeout    int
	Rpc        bool
	Compatible bool
}
 
type MySQL struct {
	Host     string
	User     string
	Password string
	Port     int
	Database string
}
 
type Redis struct {
	Host   string
	Port   int
	Dbname int
}
 
var AppConfig Config
 
func init()  {
	viperCfg := viper.New()
 
	viperCfg.SetConfigName("config")
	viperCfg.SetConfigType("yaml")
	viperCfg.AddConfigPath("D:\\demo1\\src\\demo\\demo06\\go-viper-http-watch\\config\\")
 
	viperCfg.ReadInConfig()
 
	err := viperCfg.Unmarshal(&AppConfig)
	if err != nil {
		fmt.Println(err)
	}
 
	viperCfg.WatchConfig()
	viperCfg.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("Config file changed:", e.Name)
		if err = viperCfg.Unmarshal(&AppConfig); err != nil {
			fmt.Println(err)
		}
	})
}
 
func GetConfig() Config {
	return AppConfig
}
```

### Web 服务获取：main.go

```go
package main
 
import (
	"github.com/gin-gonic/gin"
	. "go-viper-http-watch/config"
	"net/http"
)
 
func main()  {
	router := gin.Default()
 
	router.GET("/info", func(ctx *gin.Context) {
		ctx.JSON(http.StatusOK, gin.H{
			"code": 0,
			"msg": AppConfig.MySQL.Host,
		})
	})
 
	router.Run(":8080")
}
```

