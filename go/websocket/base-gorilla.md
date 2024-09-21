

## 引入包

go  get -u github.com/gorilla/websocket

go  get -u github.com/gin-gonic/gin

## 简单的demo

```go
package main

import (
    "fmt"
    "github.com/gin-gonic/gin"
    "github.com/gorilla/websocket"
    "net"
    "net/http"
)

// 注册路由
func RegistRoute(engine *gin.Engine) {
    group := engine.Group("/index")
    {
        group.GET("/regist", controller.Index.Regist)
        group.GET("/index", controller.Index.Index)
    }
    // http 渲染处理
  	engine.GET("/home", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello world!",
        })
    })
    engine.LoadHTMLGlob("templates/*")
}

// 创建控制器
var Index = new(index)

type index struct {
}

// 检查是否为socket
var upGrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool {
        return true
    },
}

// 首页渲染 - 放在目录： /templates 下
func (i *index) Index(ctx *gin.Context) {
    ctx.HTML(http.StatusOK, "index.html", map[string]interface{}{
        "title": "首页",
    })
}

// websocket 接收处理
func (i *index) Regist(ctx *gin.Context) {
    conn, err := upGrader.Upgrade(ctx.Writer, ctx.Request, nil)
    if err != nil {
        fmt.Println("websocket error:", err)
        return
    }
    //得到客户的连接ip和端口
    fmt.Println("client connect:", conn.RemoteAddr())
    go i.Do(conn)
}

// 处理socket 消息交互
func (i *index) Do(conn *websocket.Conn) {
    for {

        //获取到前端发送过来的websocket消息
        contentType, message, err := conn.ReadMessage()
        if err != nil {
            //判断是不是超时
            if netErr, ok := err.(net.Error); ok {
                if netErr.Timeout() {
                    fmt.Printf("ReadMessage timeout remote:%v\n", conn.RemoteAddr())

                }
            }
            // 其他错误，如果是 1001 和  1000，就不打印日志
            if websocket.IsUnexpectedCloseError(err, websocket.CloseGoingAway, websocket.CloseNormalClosure) {

            }

            conn.Close()
            //这边必须return，因为前端的链接可能某个时间段就断开链接了，要是没有return，会导致产生 repeated read on failed websocket connection报错,而且也可以防止内存泄露
            return
        }

        fmt.Println(contentType)
        fmt.Println(string(message))
        //写入ws
        msg := []byte("我是server")
        err = conn.WriteMessage(1, msg)
        if err != nil {
            fmt.Println("发生错误了")
            fmt.Println(err.Error())
        }
    }
}


func main() {
    r := gin.Default() //创建gin
    RegistRoute(r) //绑定路由
    r.Run(":8001")     //运行绑定端口
}
```

### 带有前端的处理

template/index.html

```html
hello
<button onclick="sendMsg()">发送消息给服务端</button>
<script>
    var ws = new WebSocket("http://localhost:8001/index/regist")
    ws.onopen = function (event){
        console.log(event)
        console.log("打开成功")
    }
    ws.onmessage = function (event){
        console.log(event)
        console.log("接收到消息："+event.data)
    }
    ws.onclose = function (event){
        console.log(event)
        console.log("关闭")
    }
   function sendMsg(){
        ws.send("hello server,I am client")
    }

</script>
```

