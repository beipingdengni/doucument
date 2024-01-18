## 添加cookie

设置header 添加 cookie

```
document.cookie = 'user=tianbeiping1;expire=36000;path=/;domain=.jd.com'

# java response
Set-Cookie = 'user=tianbeiping1;expire=36000;path=/;domain=.jd.com'
```

## 请求地址中出现特殊的字符

### URL使用encode后请求

### tomcat 增加配置

> 路径：relaxedPathChars
>
> 路径下参数：relaxedQueryChars

#### 第一种（tomcat 服务）

​	server.xml 增加配置

```xml
 <Connector port="8080" 
 						protocol="HTTP/1.1" 
 						connectionTimeout="20000"
 						relaxedPathChars="&lt;&gt;[\]^`{|}" 
 						relaxedQueryChars="&lt;&gt;[\]^`{|}" 
 						redirectPort="8443" />

或者 catalina.properties 修改如下：
tomcat.util.http.parser.HttpParser.requestTargetAllow=&lt;&gt;[\]^`{|}
```

#### 第二种（spring boot服务）

```yaml
server:
  port: 80
  tomcat:
    relaxedPathChars: <,>, [,],^,`,{,|,}
    relaxedQueryChars: <,>,[,\,],^,`,{,|,}
```

## tomcat自动启动

### Maven pom配置

```xml
<properties>
  <servlet-api.version>3.1.0</servlet-api.version>
  <tomcat.version>9.0.39</tomcat.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-core</artifactId>
    <version>${tomcat.version}</version>
    <scope>provided</scope>
  </dependency>

  <dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <version>${tomcat.version}</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
```

### java servlet 文件

```java
@WebServlet(urlPatterns = "/")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        String name = req.getParameter("name");
        if (name == null) {
            name = "world";
        }
        PrintWriter pw = resp.getWriter();
        pw.write("<h1>Hello, " + name + "!</h1>");
        pw.flush();
    }
}
```

以上注解可以替换以下xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
 
   <!-- start 使用来注解可以删除以下配置 -->
    <servlet>
        <servlet-name>helloServlet</servlet-name>
        <servlet-class>coderead.servlet.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>helloServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
   <!-- end 使用来注解可以删除以上配置 -->
  
</web-app>
```



### 启动tomcat

> 可以参考：https://www.jianshu.com/p/6bff04337753

```java
public class Main {

    public static void main(String[] args) throws LifecycleException {
        // 启动 tomcat
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.getInteger("port", 8080));
        tomcat.getConnector();
        // 创建 WebApp
        Context context = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());
        WebResourceRoot resources = new StandardRoot(context);
        resources.addPreResources(
                new DirResourceSet(resources, "/WEB-INF/classes",
                        new File("target/classes").getAbsolutePath(), "/"));
        context.setResources(resources);

        tomcat.start();
        tomcat.getServer().await();
    }
}
```

或者使用

```java
public class TomcatStart {

    public static int TOMCAT_PORT = 8080;
    public static String TOMCAT_HOSTNAME = "127.0.0.1";
    public static String WEBAPP_PATH = "src/main";
    public static String WEBINF_CLASSES = "/WEB-INF/classes";
    public static String CLASS_PATH = "target/classes";
    public static String INTERNAL_PATH = "/";
    
    public static void main(String[] args) throws ServletException, LifecycleException {
        TomcatStart.run();
    }
    
    public static void run() throws ServletException, LifecycleException {
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(TomcatStart.TOMCAT_PORT);
        tomcat.setHostname(TomcatStart.TOMCAT_HOSTNAME);
        tomcat.setBaseDir("."); // tomcat 信息保存在项目下
       	// https://www.cnblogs.com/ChenD/p/10061008.html
        StandardContext myCtx = (StandardContext) tomcat
            .addWebapp("/access", System.getProperty("user.dir") + File.separator + TomcatStart.WEBAPP_PATH);
        /*
         * true时：相关classes | jar 修改时，会重新加载资源，不过资源消耗很大
         * autoDeploy 与这个很相似，tomcat自带的热部署不是特别可靠，效率也不高。生产环境不建议开启。
         * 相关文档：http://www.blogjava.net/wangxinsh55/archive/2011/05/31/351449.html
         */
        myCtx.setReloadable(false);
        // 上下文监听器
        myCtx.addLifecycleListener(new AprLifecycleListener());
        
        /*String webAppMount = System.getProperty("user.dir") + File.separator + TomcatStart.CLASS_PATH;
        WebResourceRoot root = new StandardRoot(myCtx);
        root.addPreResources(
            new DirResourceSet(root, TomcatStart.WEBINF_CLASSES, webAppMount, TomcatStart.INTERNAL_PATH));*/
        
        // 注册servlet
        tomcat.addServlet("/access", "demoServlet", new DemoServlet());
        // servlet mapping
        myCtx.addServletMappingDecoded("/demo.do", "demoServlet");
        tomcat.start();
        tomcat.getServer().await();
      	// 访问路径为：localhost:8080/access/demo.do  映射到DemoServlet
    }
    
}
```

