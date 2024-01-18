## maven 2

```xml
<dependency>
    <groupId>io.undertow</groupId>
    <artifactId>undertow-core</artifactId>
    <version>2.3.10.Final</version>
</dependency>
<dependency>
    <groupId>io.undertow</groupId>
    <artifactId>undertow-servlet</artifactId>
    <version>2.3.10.Final</version>
</dependency>
```

## java代码演示

#### 基础使用

```java
public class HelloWorldServer {
    public static void main(final String[] args) {
        Undertow server = Undertow.builder()
                .addHttpListener(8080, "localhost")
                .setHandler(new HttpHandler() {
                    @Override
                    public void handleRequest(final HttpServerExchange exchange) throws Exception {
                        exchange.getResponseHeaders().put(Headers.CONTENT_TYPE, "text/plain");
                        exchange.getResponseSender().send("Hello World");
                    }
                }).build();
        server.start();
    }
}
// java -cp target/dependency/*:target/undertow-quickstart.jar org.wildfly.undertow.quickstart.HelloWorldServer
// 访问返回：http://localhost:8080
```

### 集成到severlet

```java
public class ServletServer {  
      
  public static final String MYAPP = "/myapp";
  
  public static void main(String[] args) throws ServletException {  
    
    ClassLoader classLoader = ServletServer.class.getClassLoader()
    DeploymentInfo servletBuilder = Servlets.deployment().setClassLoader(classLoader)  
      .setContextPath(MYAPP)  
      .setDeploymentName("myapp.war")  
      .addServlets(Servlets.servlet("MessageServlet", MessageServlet.class).addMappings("/messageServlet")  
                   , Servlets.servlet("MyServlet", MyServlet.class).addMappings("/myServlet"));

    DeploymentManager manager = Servlets.defaultContainer().addDeployment(servletBuilder);  
    manager.deploy();  

    HttpHandler servletHandler = manager.start();  
    PathHandler path = Handlers.path(Handlers.redirect(MYAPP)).addPrefixPath(MYAPP, servletHandler);  

    Undertow server = Undertow.builder().addHttpListener(8080, "localhost").setHandler(path).build();  
    server.start();  
  }
  // http://localhost:8080/myapp/messageServlet
  // http://localhost:8080/myapp/myServlet
}
```

