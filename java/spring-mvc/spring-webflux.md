# Spring Flux



```java
// 增加时候请求
public Mono<Integer> add(@RequestBody Publisher<UserAddDTO> addDTO) {
  
// 自定义路由
@Bean
public RouterFunction<ServerResponse> route(GreetingHandler greetingHandler) {
	 return RouterFunctions.route(RequestPredicates.GET("/hello").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), greetingHandler::hello);
    }

// 自定义路由
@Bean
public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
         return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
             .andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
             .andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
}

// 请求三方接口
public class GreetingWebClient { 
	private WebClient client = WebClient.create("http://localhost:8080"); 
     private Mono<ClientResponse> result = client.get()
             .uri("/hello")             
             .accept(MediaType.TEXT_PLAIN)
             .exchange();
 
     public String getResult() {
         return ">> result = " + result.flatMap(res -> res.bodyToMono(String.class)).block();
     }
 }
 
 // 启动服务 spring boot
 SpringApplication.run(CjsReactiveRestServiceApplication.class, args);
 GreetingWebClient gwc = new GreetingWebClient();
```



# 自定义handler处理数据

```java
import org.springframework.http.MediaType;
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.http.server.reactive.ReactorHttpHandlerAdapter;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.server.*;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.netty.http.server.HttpServer;

public class UserHandler {

    private final UserService userService;

    public UserHandler(UserService userService) {
        this.userService = userService;
    }

    // 根据id查询
    public Mono<ServerResponse> getById(ServerRequest request){
        // 获取id值
        String id = request.pathVariable("id");
        // 空值处理
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();

        // 调用Service方法得到数据
        Mono<User> userMono = userService.getById(Integer.parseInt(id));
        // 把userMono进行转换返回
        return userMono.flatMap(user ->
                ServerResponse
                        .ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(BodyInserters.fromValue(userMono))
                        .switchIfEmpty(notFound)
        );
    }

    // 查询多个
    public Mono<ServerResponse> getAll(ServerRequest request){
        // 调用Service得到结果
        Flux<User> users = userService.getAll();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(users, User.class);

    }

    // 保存
    public Mono<ServerResponse> save(ServerRequest request){
        // 获取User对象
        Mono<User> userMono = request.bodyToMono(User.class);
        return ServerResponse.ok().build(userService.save(userMono));
    }


    public static void main(String[] args) {
        // 创建对象
        UserService userService = new UserService();
        UserHandler userHandler = new UserHandler(userService);
        // 创建路由
        RouterFunction<ServerResponse> route = RouterFunctions
                .route(RequestPredicates.GET("/user/{id}").and(RequestPredicates.accept(MediaType.APPLICATION_JSON)), userHandler::getById)
                .andRoute(RequestPredicates.GET("/users").and(RequestPredicates.accept(MediaType.APPLICATION_JSON)), userHandler::getAll);
        // 路由和handler适配
        HttpHandler httpHandler = RouterFunctions.toHttpHandler(route);
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);
        // 创建服务器
        HttpServer httpServer = HttpServer.create();
        httpServer.handle(adapter).bindNow();
    }
}
```

## 服务层处理

```java
@Service
public class UserService {

    // 模拟数据库存储
    private Map<Integer, User> map = new HashMap<>();

    public UserService() {
        map.put(1, new User("zhangsan"));
        map.put(2, new User("lisi"));
        map.put(3, new User("wangwu"));
    }

    // 根据id查询
    public Mono<User> getById(Integer id){
        // 返回数据或空值
        return Mono.justOrEmpty(map.get(id));
    }

    // 查询多个
    public Flux<User> getAll(){
        return Flux.fromIterable(map.values());
    }

    // 保存
    public Mono<Void> save(Mono<User> userMono){
        return userMono.doOnNext(user -> {
            int id = map.size() + 1;
            map.put(id, user);
        }).thenEmpty(Mono.empty()); // 最后置空
    }
}
```

# 数据库处理

- @ReadOnlyProperty 的作用是防止代码修改创建时间和更新时间，这个会由 MySQL 自动完成
- 目前 R2DBC 尚不支持 Audit 功能，所以 createdBy 和 updatedBy 还不能自动设置

```
public interface StudentRepository extends ReactiveCrudRepository<Student, Long> {
}

@Configuration
public class R2dbcConfig {
    @Bean
    public NamingStrategy namingStrategy() {
        return new NamingStrategy() {
            @Override
            public String getColumnName(RelationalPersistentProperty property) {
                return property.getName();
            }
        };
    }
}
```

