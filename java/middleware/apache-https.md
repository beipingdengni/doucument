## apache http 代理

```java
// 代理地址
HttpHost proxy = new HttpHost("191.168.1.303", 7443, "https");
RequestConfig config = RequestConfig.custom()
  .setProxy(proxy)
  .build();
HttpGet request = new HttpGet("/");
request.setConfig(config);
// 访问地址
HttpHost target= new HttpHost("https://jd.com", 443, "https");
CloseableHttpResponse response = httpclient.execute(target, request);
```

## https信任访问(apache)

```java
// 自签证书信任
TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
// 加载策略 null可以是：trustStore
SSLContext sslContext = SSLContexts.custom().loadTrustMaterial(null, acceptingTrustStrategy).build();
// ssl协议上下文请求
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE);
// 注册协议
Registry<ConnectionSocketFactory> socketFactoryRegistry =RegistryBuilder<ConnectionSocketFactory>
  .create()
  .register("https", sslsf)
  .register("http", new PlainConnectionSocketFactory())
  .build();
// 基本信息请求
BasicHttpClientConnectionManager connectionManager = 
  new BasicHttpClientConnectionManager(socketFactoryRegistry);
// 组装请求
CloseableHttpClient httpClient = HttpClients
  .custom()
  .setSSLSocketFactory(sslsf)
  .setConnectionManager(connectionManager)
  .build();
```



## java https

```java
URL localURL= new URL(urlPath);
URLConnection connection = localURL.openConnection();
HttpURLConnection httpURLConnection = (HttpURLConnection) connection;
// http 请求
if (connection instanceof HttpsURLConnection) {
  TrustManager[] tm = {ignoreCertificationTrustManger};
  try {
    // 证书
    SSLContext sslContext = SSLContext.getInstance("SSL", "SunJSSE");
    sslContext.init(null, tm, new java.security.SecureRandom());
    // 工厂连接
    SSLSocketFactory ssf = sslContext.getSocketFactory();
    ((HttpsURLConnection) httpURLConnection).setSSLSocketFactory(ssf);
    // 增加信任
    ((HttpsURLConnection) httpURLConnection).setHostnameVerifier(ignoreHostnameVerifier);
  } catch (Exception e1) {
    logger.logError(e1.getMessage(), e1);
  }
}
httpURLConnection.setDoOutput(true);
httpURLConnection.setRequestMethod("POST");
httpURLConnection.setRequestProperty("Content-Type", "application/octet-stream");
httpURLConnection.setRequestProperty("Accept-Encoding", "chunck");
httpURLConnection.setConnectTimeout(3000);
outputStream = httpURLConnection.getOutputStream();

```

## https自定义协议信任访问

```java
//  KeyStore trustStore = KeyStore.getInstance("PKCS12");
KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());  
FileInputStream instream = new FileInputStream(new File("d:\\tomcat.keystore"));  
// 加载keyStore d:\\tomcat.keystore    
trustStore.load(instream, "123456".toCharArray());  
instream.close(); 

// 相信自己的CA和所有自签名的证书  
SSLContext sslcontext = SSLContexts.custom()
  .loadTrustMaterial(trustStore, new TrustSelfSignedStrategy()).build();  
// 只允许使用TLSv1协议  
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslcontext, new String[] { "TLSv1" }, null,SSLConnectionSocketFactory.BROWSER_COMPATIBLE_HOSTNAME_VERIFIER);

httpclient = HttpClients.custom().setSSLSocketFactory(sslsf).build();  
// 创建http请求(get方式)  
HttpGet httpget = new HttpGet("https://localhost:8443/myDemo/Ajax/serivceJ.action");  
System.out.println("executing request" + httpget.getRequestLine());  
CloseableHttpResponse response = httpclient.execute(httpget);
HttpEntity entity = response.getEntity();  
EntityUtils.consume(entity);  

// 简介方式
HttpHost proxy = new HttpHost("100.67.76.9",10003);
CloseableHttpClient httpClient = HttpClients.custom()
        .setProxy(proxy)
        .setHostnameVerifier(new AllowAllHostnameVerifier())
        .setSslcontext(new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy()
            {
                public boolean isTrusted(X509Certificate[] arg0, String arg1) throws CertificateException
                {
                    return true;
                }
            }).build()).build();
```



## https访问（及密码携带 basic）

```java
CloseableHttpClient client = HttpClients.custom().setRetryHandler(new MyRequestRetryHandler()).build();
HttpHost targetHost = new HttpHost("127.0.0.1", 9200, "http");
// 基础信任
CredentialsProvider credsProvider = new BasicCredentialsProvider();
// 指定域名和端口
credsProvider.setCredentials(new AuthScope(targetHost.getHostName(), targetHost.getPort()),
                             new UsernamePasswordCredentials("elastic", "password"))
// 全部信任
credsProvider.setCredentials(AuthScope.ANY,new UsernamePasswordCredentials("username","password"));
// Add AuthCache to the execution context
HttpClientContext context = HttpClientContext.create();
// 上下文证书信任
context.setCredentialsProvider(credsProvider);
// 上下文
CloseableHttpResponse response = client.execute(method, context);
HttpEntity entity = response.getEntity();
String responseBody = EntityUtils.toString(entity);
System.out.println(responseBody);
```

