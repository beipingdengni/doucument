下载驱动地址

phantomjs：http://phantomjs.org/download.html



pom.xml

```xml
<dependency>
  <groupId>org.seleniumhq.selenium</groupId>
  <artifactId>selenium-java</artifactId>
  <version>2.44.0</version>
</dependency>
<dependency>
  <groupId>com.codeborne</groupId>
  <artifactId>phantomjsdriver</artifactId>
  <version>1.2.1</version>
</dependency>
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>22.0</version>
</dependency>
<dependency>
  <groupId>net.java.dev.jna</groupId>
  <artifactId>jna</artifactId>
  <version>3.4.0</version>
</dependency>
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>1.3.1</version>
</dependency>
```



代码

```java
public class JsMain {
  
    public static void main(String[] args) {
        DesiredCapabilities dcaps = new DesiredCapabilities();
        //ssl证书支持
        dcaps.setCapability("acceptSslCerts", true);
        //截屏支持
        dcaps.setCapability("takesScreenshot", true);
        //css搜索支持
        dcaps.setCapability("cssSelectorsEnabled", true);
        //js支持
        dcaps.setJavascriptEnabled(true);
        // 超时
        dcaps.setCapability("phantomjs.page.settings.resource-timeout", 5000);
        //驱动支持
        dcaps.setCapability(PhantomJSDriverService.PHANTOMJS_EXECUTABLE_PATH_PROPERTY,"./phantomjs");
        //创建无界面浏览器对象
        PhantomJSDriver driver = new PhantomJSDriver(dcaps);
				// 让浏览器访问空间主页
//        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
        System.out.println("start=========>");
        for (int i = 0; i < 10; i++) {
            driver.get("https://www.baidu.com");
            System.out.println(driver.getWindowHandles().size());
            System.out.println("=======>" + System.currentTimeMillis());

        }

        driver.close();
        driver.quit();
    }

}

```

