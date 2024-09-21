# 默认web请求(默认)

### 1. 创建HTTP服务器

使用Go语言的`net/http`包可以轻松地创建一个HTTP服务器。下面是一个简单的示例：

```go
package main

import (
 "fmt"
 "net/http"
)

func main() {
 http.HandleFunc("/", hello)
 http.ListenAndServe(":8080", nil)
}

func hello(w http.ResponseWriter, r *http.Request) {
 fmt.Fprintf(w, "Hello, World!")
}

```

在这个示例中，我们定义了一个`hello`函数作为请求处理函数，并使用`http.HandleFunc`将其绑定到根URL（`/`）。然后，通过`http.ListenAndServe`启动HTTP服务器，监听8080端口。

### 2. 实现路由

要实现路由功能，我们需要根据请求的URL匹配相应的处理函数。以下是一个简单的路由实现：

```go
package main

import (
 "fmt"
 "net/http"
)

var routes = map[string]http.HandlerFunc{
 "/":      hello,
 "/about": about,
}

func main() {
 http.HandleFunc("/", root)
 http.ListenAndServe(":8080", nil)
}

func root(w http.ResponseWriter, r *http.Request) {
 handler := routes[r.URL.Path]
 if handler != nil {
 handler(w, r)
 } else {
 http.NotFound(w, r)
 }
}

func hello(w http.ResponseWriter, r *http.Request) {
 fmt.Fprintf(w, "Hello, World!")
}

func about(w http.ResponseWriter, r *http.Request) {
 fmt.Fprintf(w, "About Us")
}
```

在这个示例中，我们定义了一个`routes`映射，将URL路径映射到相应的处理函数。然后，在`root`函数中，我们根据请求的URL路径查找对应的处理函数并执行它。如果找不到对应的处理函数，则返回一个404错误。

### 3. 添加中间件

中间件可以在请求处理过程中执行一些通用操作。以下是一个简单的中间件示例，用于记录每个请求的IP地址：

```go
package main

import (
 "fmt"
 "net/http"
)

var routes = map[string]http.HandlerFunc{
 "/":      hello,
 "/about": about,
}

func main() {
 http.HandleFunc("/", middleware(root))
 http.ListenAndServe(":8080", nil)
}

func middleware(next http.HandlerFunc) http.HandlerFunc {
 return func(w http.ResponseWriter, r *http.Request) {
 fmt.Printf("IP Address: %s\n", r.RemoteAddr)
 next(w, r)
 }
}

func root(w http.ResponseWriter, r *http.Request) {
 handler := routes[r.URL.Path]
 if handler != nil {
 handler(w, r)
 } else {
 http.NotFound(w, r)
 }
}

func hello(w http.ResponseWriter, r *http.Request) {
 fmt.Fprintf(w, "Hello, World!")
}

func about(w http.ResponseWriter, r *http.Request) {
 fmt.Fprintf(w, "About Us")
}
```

在这个示例中，我们定义了一个`middleware`函数，它接受一个处理函数作为参数，并返回一个新的处理函数。新的处理函数在执行原始处理函数之前，会先记录请求的IP地址。然后，在`main`函数中，我们将`root`函数作为中间件应用到所有路由上。

### 4. 使用模板引擎

为了渲染动态 网页，我们可以使用模板引擎。Go语言有许多优秀的模板引擎可供选择，如`html/template`、`text/template`和`gohtml/template`等