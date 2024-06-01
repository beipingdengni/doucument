







### 客户端凭证

请求token

`curl -XPOST -H 'Authorization: Basic client_id和client_secret组合base64' -H 'Content-Type: application/x-www-form-urlencoded' -d 'grant_type=client_credentials' https://my.local/oauth2/token`

下面是利用token

`curl -XPOST -H 'Authorization: Bearer token值' -H 'Content-Type: application/json' -d '{"id":1}' https://demo.local/api/queryUserInfo`

或

`curl -XPOST -H 'Content-Type: application/json' https://demo.local/api/queryUserInfo?access_token=token值`

##### 利用basic 请求模式去请求token

```java
// 创建凭证提供者并设置用户名和密码  
CredentialsProvider provider = new BasicCredentialsProvider();  
// username 可以使用client_id, password可以使用client_secret
UsernamePasswordCredentials credentials = new UsernamePasswordCredentials("username", "password");  
provider.setCredentials(AuthScope.ANY, credentials);  
// 上面方法等于 credentials = "username:password"; 
// 使用base64 编码 credentials 就可以传递 encodedCredentials =Base64.encodeString(credentials);
// header中就可以传递> Authorization: Basic ${encodedCredentials}

// 使用凭证提供者创建 HttpClient 实例  
CloseableHttpClient httpClient = HttpClients.custom()  
  .setDefaultCredentialsProvider(provider)  
  .build();  
// 创建 POST 请求  
HttpPost httpPost = new HttpPost(tokenEndpoint);  
// 设置请求头，指定内容类型为 application/x-www-form-urlencoded  
httpPost.setHeader("Content-Type", "application/x-www-form-urlencoded");  
httpPost.setHeader("Accept", "application/json");  
// 设置请求体  
StringEntity params = new StringEntity(requestBody, StandardCharsets.UTF_8);  
httpPost.setEntity(params);
// 执行请求并获取响应  
HttpResponse response = httpClient.execute(httpPost);  
// 获取响应实体  
HttpEntity entity = response.getEntity();  
// 输出响应内容  
if (entity != null) {  
  System.out.println(EntityUtils.toString(entity));  
}
```



## 授权码

> 1、客户端访问（需要输入用户名和密码）：
>     http://localhost:8089/oauth/authorize?response_type=code&client_id=client&redirect_uri=http://www.baidu.com&scope=all
> 2、获取授权码
>     2.1、自动认证则返回重定向地址：https://www.baidu.com/?code=xLPYQo
>     2.2、手动认证则需要点击授权，返回重定向地址：https://www.baidu.com/?code=mVjzsD
> 3、后台根据授权码获取token（需要客户端ID和客户端密码）
>     http://localhost:8089/oauth/token?grant_type=authorization_code&redirect_uri=http://www.baidu.com&code=oVuabA
> 4、输入客户端ID和密码后，返回JSON字符串
>     {"access_token":"19bd7961-2dab-49a2-ac34-cc94453d8768","token_type":"bearer","refresh_token":"535e4e97-d217-4c55-8769-3b8d7eec412a","expires_in":3599,"scope":"all"}
> 5、验证token信息
>         5.1、http://localhost:8089/user/selectUserDetail/1?access_token=535e4e97-d217-4c55-8769-3b8d7eec412a
>     或者
>         5.2、header中加入类似以下值对：
>             Authorization：Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MTExMjY1MTUsInVzZXJfbmFtZSI6InVzZXIiLCJqdGkiOiJmNzliNjA3MS0xYjk5LTRjZmEtODYxYy03MjExZTBhYzBhN2MiLCJjbGllbnRfaWQiOiJjbGllbnQiLCJzY29wZSI6WyJhbGwiXX0.5gCmEZM4jtv2JT-7akJ1spe3WhRjTKld8wuMVp7Qio0

##  隐式授权

> 1、浏览器访问（需要输入用户名和密码）：
>     http://localhost:8089/oauth/authorize?response_type=token&client_id=client&redirect_uri=http://www.baidu.com&scope=all
> 2、获取token
>     2.1、自动认证则返回重定向地址：https://www.baidu.com/#access_token=b4672f27-7339-4b5d-b746-d50e07eccf21&token_type=bearer&expires_in=3599
>     2.2、手动认证则需要点击授权，返回重定向地址：https://www.baidu.com/#access_token=ea1b53e8-0ecb-4065-b50d-6e1bd715a94e&token_type=bearer&expires_in=3599
> 3、验证token
>     http://localhost:8089/user/selectUserDetail/1?access_token=b4672f27-7339-4b5d-b746-d50e07eccf21

注意：授权码与隐式授权码的response_type不同，一个为code，另一个为token 

## 客户端凭证

> 1、访问地址（需要指定客户端ID和客户端密码）：
>     http://localhost:8089/oauth/token?grant_type=client_credentials&client_id=client&client_secret=123
> 2、获取返回结果：
>     {"access_token":"101f312e-69cd-4130-a50e-c4f74086dc83","token_type":"bearer","expires_in":3599,"scope":"all"}
> 3、验证token：
>     http://localhost:8089/user/selectUserDetail/1?access_token=101f312e-69cd-4130-a50e-c4f74086dc83

## 密码模式

> 1、访问址（需要指定用户名和密码）：
>
> ​	 http://localhost:8089/oauth/token?grant_type=password&username=user&password=123
> 2、获取返回值信息：
>
> ​	{"access_token":"713546a6-28c7-4e9e-9276-938ad1e0f7f9","token_type":"bearer","refresh_token":"c2387108-5ecd-4d28-af46-d3eafaeb3004","expires_in":3599,"scope":"all"}
> 3、验证token：
> ​    http://localhost:8089/user/selectUserDetail/1?access_token=713546a6-28c7-4e9e-9276-938ad1e0f7f9