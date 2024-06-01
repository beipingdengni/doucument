# selenium

# [Selenium网址](https://www.cnblogs.com/xmlbw/p/4402859.html)

Selenium官网：http://www.seleniumhq.org/

Selenium火狐插件地址：http://release.seleniumhq.org/selenium-ide/

浏览器驱动地址： 

firefoxdriver: https://github.com/mozilla/geckodriver/releases

chromedriver：http://chromedriver.storage.googleapis.com/index.html

| **Chrome**:  | https://chromedriver.chromium.org/downloads  <br/>https://registry.npmmirror.com/binary.html?path=chromedriver/ |
| ------------ | ------------------------------------------------------------ |
| **Edge**:    | https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/ |
| **Firefox**: | https://github.com/mozilla/geckodriver/releases              |
| **Safari**:  | https://webkit.org/blog/6900/webdriver-support-in-safari-10/ |
| phantomjs    | http://phantomjs.org/download.html                           |
| IEdriver     | http://selenium-release.storage.googleapis.com/index.html    |



> 官方网站：https://www.selenium.dev/zh-cn/documentation/

### 本地html加载

```python
# 获取本地HTML文件的路径
local_html_path = 'file:///' + os.path.abspath('path/to/your/local.html')
# 使用Selenium打开本地HTML文件
driver.get(local_html_path)
```

## chrome

驱动下载： https://chromedriver.chromium.org/downloads

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
 
# 初始化无头 Chrome 浏览器
chrome_options = Options()
chrome_options.add_argument("--headless")  # 确保无界面模式
driver_path = '/path/to/your/chromedriver'  # 指定 chromedriver 路径
 
# 获取网页内容的函数
def get_page_content(url):
    driver = webdriver.Chrome(executable_path=driver_path, options=chrome_options)
    driver.get(url)  # 打开网页
    return driver.page_source  # 获取网页源代码
 
# 示例 URL
url = 'http://example.com'
 
# 获取网页内容
content = get_page_content(url)
print(content)  # 打印网页内容
 
# 关闭无头浏览器
driver.quit()
```

## PhantomJS

http://wenku.kuryun.com/docs/phantomjs/download.html

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
```

java

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
        driver.get("https://www.baidu.com");
//        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);

        String pageSource = driver.getPageSource();
        System.out.println(pageSource);

        driver.close();
        driver.quit();
    }

}

```

