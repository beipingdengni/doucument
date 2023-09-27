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

