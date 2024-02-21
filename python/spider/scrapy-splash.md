# Scrapy-Splash

github: https://github.com/scrapy-plugins/scrapy-splash

## 动态渲染JS

`scrapy-splash` 是一个Scrapy的插件，用于与Splash服务集成，Splash是一个JavaScript渲染服务，可以用于处理动态网页和JavaScript渲染。`scrapy-splash` 可以帮助Scrapy爬虫处理那些需要JavaScript渲染的网页，通过将请求发送到Splash服务，获取JavaScript渲染后的页面内容，从而使Scrapy可以处理动态网页和JavaScript生成的内容。

**原因**

像selenium、phantomjs都是常用的渲染网页的工具。

就拿selenium来说，需要通过加载一个浏览器内核来进行渲染，效率有点低。而且与Scrapy集成需要实现一个downloder middleware，操作起来有些许的复杂，对我这种懒人来说简直折磨。scrapy-deltafetch的出现，仅仅几行配置就解决了这些问题。

## 安装依赖

> pip3 install scrapy-splash

## 启动容器

> ```shell
> # 拉取
> docker pull scrapinghub/splash
> 
> docker run -d -p 8050:8050 scrapinghub/splash --disable-private-mode
> # --disable-private-mode 是Docker引擎的一个选项，它用于禁用Docker的私有模式。
> 	# 在私有模式下，Docker引擎会将容器的网络命名空间隔离开来，使得容器间无法相互访问。
> 	# 禁用私有模式后，容器将能够相互访问，这在某些场景下可能是有用的，比如需要在容器之间进行通信或者共享网络资源。
> 
> # 运行后就删除
> docker run -d --rm -p8050:8050 scrapinghub/splash
> ```

## 爬虫程序配置

> 在settings.py中添加splash服务的参数。

```python
SPLASH_URL = 'http://localhost:8050'
DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
}
SPIDER_MIDDLEWARES = {
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
}
DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```

### 请求

> 原本使用Request来请求网站，这里要修改成SplashRequst

```python
from scrapy_splash import SplashRequest
# 原本是yield Request()
yield SplashRequest()
```

