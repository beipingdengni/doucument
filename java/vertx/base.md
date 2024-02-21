# Vert.X

> 官方网站： https://vertx.io/

博客网站

> https://zhuanlan.zhihu.com/p/407323966?utm_id=0
>
> 2020年Vertx快速入门参考: https://blog.csdn.net/qq_18870127/article/details/110943712

## maven

```xml
<dependency>
 <groupId>io.vertx</groupId>
 <artifactId>vertx-web</artifactId>
 <version>4.5.1</version>
</dependency>
```



```java
Vertx vertx = Vertx.vertx();
Router router = Router.router(vertx);

// 跨域处理
// router.route().handler(CorsHandler.create().addRelativeOrigin("vertx\\.io").allowedMethod(HttpMethod.GET));

// 中间拦截处理，阻塞
router.route().handler("/static/*").handler();

// 中间拦截处理，阻塞
router.route().blockingHandler(ctx -> {
  // Do something that might take some time synchronously
  service.doSomethingThatBlocks();
  // Now call the next handler
  ctx.next();
});

// Route route = router.route().method(HttpMethod.POST);
router.route().handler(ctx -> {
  // 获取到response对象
  HttpServerResponse response = ctx.response();
  // 设置响应头
	response.putHeader("Content-type", "text/html;charset=utf-8");
  // 响应数据
  response.end("Hello World from Vert.x-Web!");
});

// 处理post json 请求
If you know the request body is JSON, then you can use 
body
 and use the right getter (e.g.: .asJsonObject()). if you know it’s a string you can use .asString(), or to retrieve it as a buffer use .buffer()

//   返回json，响应头content typ设置为application/json
// router.get("/some/path").respond(ctx -> Future.succeededFuture(new JsonObject().put("hello", "world")));
// 确保e Pojo is serialized to json，响应头content typ设置为application/json
// router.get("/some/path").respond(ctx -> Future.succeededFuture(new Pojo()));

router.get("/some/path").respond(ctx -> ctx
      .response()
      .putHeader("Content-Type", "text/plain")
      .end("hello world!"));

router.get("/some/path").respond(ctx -> ctx
      .response()
      .setChunked(true)
      .write("Write some text..."));

HttpServer server = vertx.createHttpServer();
server.requestHandler(router).listen(8080);
```

