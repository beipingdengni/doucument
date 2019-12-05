## Scrapy

> 官方网站（https://scrapy.org/）
>
> 中文文档：https://www.osgeo.cn/scrapy/



机构图

![tu](https://pic2.zhimg.com/v2-8c591d54457bb033812a2b0364011e9c_1200x500.jpg)

```
Scrapy Engine
	引擎负责控制数据流在系统中所有组件中流动，并在相应动作发生时触发事件。 详细内容查看下面的数据流(Data Flow)部分。
	
调度器(Scheduler)
	调度器从引擎接受request并将他们入队，以便之后引擎请求他们时提供给引擎。
	
下载器(Downloader)
	下载器负责获取页面数据并提供给引擎，而后提供给spider。
	
Spiders
	Spider是Scrapy用户编写用于分析response并提取item(即获取到的item)或额外跟进的URL的类。 每个spider负责处理一个特定(或一些)网站。
	
Item Pipeline
	Item Pipeline负责处理被spider提取出来的item。典型的处理有清理、 验证及持久化(例如存取到数据库中)。
	
下载器中间件(Downloader middlewares)
	下载器中间件是在引擎及下载器之间的特定钩子(specific hook)，处理Downloader传递给引擎的response。 其提供了一个简便的机制，通过插入自定义代码来扩展Scrapy功能。
	
Spider中间件(Spider middlewares)
	Spider中间件是在引擎及Spider之间的特定钩子(specific hook)，处理spider的输入(response)和输出(items及requests)。 其提供了一个简便的机制，通过插入自定义代码来扩展Scrapy功能。

```

#### 数据流(Data flow)

```
1、引擎打开一个网站(open a domain)，找到处理该网站的Spider并向该spider请求第一个要爬取的URL(s)。
2、引擎从Spider中获取到第一个要爬取的URL并在调度器(Scheduler)以Request调度。
3、引擎向调度器请求下一个要爬取的URL。
4、调度器返回下一个要爬取的URL给引擎，引擎将URL通过下载中间件(请求(request)方向)转发给下载器(Downloader)。
5、一旦页面下载完毕，下载器生成一个该页面的Response，并将其通过下载中间件(返回(response)方向)发送给引擎。
6、引擎从下载器中接收到Response并通过Spider中间件(输入方向)发送给Spider处理。
7、Spider处理Response并返回爬取到的Item及(跟进的)新的Request给引擎。
8、引擎将(Spider返回的)爬取到的Item给Item Pipeline，将(Spider返回的)Request给调度器。
9、(从第二步)重复直到调度器中没有更多地request，引擎关闭该网站。

```



#### 以下简单demo

##### 创建项目

```python
scrapy startproject scrapyspider

目录结构
scrapyspider/
    scrapy.cfg
    scrapyspider/
        __init__.py   #// 设置文件
        items.py  		#// 设置文件
        pipelines.py  #// pipelines文件
        settings.py   #// 设置文件
        spiders/      #// 放置spider代码的目录
            __init__.py
            BlogSpider.py
```

##### 第一个爬虫

```python
from scrapy.spiders import Spider

class BlogSpider(Spider):
		# 用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字
    name = 'woodenrobot'
    # 包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 后续的URL则从初始的URL获取到的数据中提取
    start_urls = ['http://woodenrobot.me']
		
		# spider的一个方法。 被调用时，每个初始URL完成下载后生成的 Response 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 Request 对象
    def parse(self, response):
        titles = response.xpath('//a[@class="post-title-link"]/text()').extract()
        for title in titles:
            print title.strip()
```

##### 启动爬虫

```
打开终端进入项目所在路径(即:scrapyspider路径下)运行下列命令:

scrapy crawl woodenrobot
```

