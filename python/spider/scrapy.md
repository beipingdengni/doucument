## Scrapy

> 官方网站（https://scrapy.org/）
>
> 中文文档：https://www.osgeo.cn/scrapy/



机构图

![tu](https://pic2.zhimg.com/v2-8c591d54457bb033812a2b0364011e9c_1200x500.jpg)

```markdown
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

```markdown
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

## 以下简单demo

### 安装

```shell
pip3 install scrapy -i https://pypi.douban.com/simple/

pip install scrapy
pip install selenium == 3.0.0
pip install pymysql
pip install bs4
```

## 创建项目 

### 创建project、spider

```shell
python3 -m scrapy startproject scrapyspider
python3 -m scrapy genspider douban https://book.douban.com

scrapy startproject scrapyspider
scrapy genspider douban https://book.douban.com
```

### 目录结构介绍

```python
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
            doubanSpider.py
```

#### 配置基础参数

>  settings.py   #// 设置文件

```python
ROBOTSTXT_OBEY = False #reboot机器人
COOKIES_ENABLED = False #cookie 缓存
DOWNLOAD_DELAY = 3  # 随机延迟
RANDOMIZE_DOWNLOAD_DELAY=True #随机延迟
```

#### Middleware自定义

```python
DOWNLOADER_MIDDLEWARES = {
		# 项目目录名.模块名.类名:优先级(1-1000不等)
    "scrapyspider.middlewares.SeleniumDownloaderMiddleware": 543,
}
```

##### selenium

###### 安装依赖

```python
pip install scrapy-selenium

# settings.py
DOWNLOADER_MIDDLEWARES = {
  'scrapy_selenium.SeleniumMiddleware': 800
}

SELENIUM_DRIVER_NAME = 'chrome'  #浏览器名称
SELENIUM_DRIVER_EXECUTABLE_PATH = 'chromedriver.exe'  #驱动路径
SELENIUM_DRIVER_ARGUMENTS = ['-headless'] #参数，不打开浏览器

# 封装使用
from scrapy_selenium import SeleniumRequest
class AccupassSpider(scrapy.Spider):
    name = 'douban'
    allowed_domains = ['book.douban.com']
    start_urls = ['http://book.douban.com']
 
    def start_requests(self):
        yield SeleniumRequest(url='http://book.douban.com', callback=self.parse)

```

京东商城手机页面爬取

```python
import scrapy
from scrapy import Item, Field
from scrapy_selenium import SeleniumRequest
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# 定义一个Item类
class HuaweiPhoneItem(Item):
    phone_name = Field()
    phone_price = Field()
    phone_image = Field()
    phone_description = Field()

# 爬虫
class JDPhoneSpider(scrapy.Spider):
    name = 'jd_phone'
    start_urls = ['https://list.jd.com/list.html?cat=9987,653,655']

    def start_requests(self):
        for url in self.start_urls:
            yield SeleniumRequest(url=url, callback=self.parse)

    def parse(self, response):
        phone_items = response.css('.gl-item')
        for item in phone_items:
            shop_name = item.css('.curr-shop::text').get()
            if '京东自营' in shop_name:  # 只获取京东自营店
                phone_name = item.css('.p-name a::text').get()
                if '华为' in phone_name or 'HUAWEI' in phone_name:  # 只获取华为手机
                    phone_image = item.css('.p-img img::attr(data-lazy-img)').get()
                    phone_description = item.css('.p-name a::attr(title)').get()
                    # 等待价格加载完成
                    price_element = WebDriverWait(response, 10).until(
                        EC.presence_of_element_located((By.CSS_SELECTOR, '.p-price strong i'))
                    )
                    phone_price = price_element.text
                    # 使用Item存储数据
                    yield HuaweiPhoneItem(
                        phone_name=phone_name.strip(),
                        phone_price=phone_price,
                        phone_image=response.urljoin(phone_image),  # 补全图片URL
                        phone_description=phone_description
                    )

        # 处理分页
        next_page_element = response.css('.pn-next')
        next_page = next_page_element.attrib['href'] if next_page_element else None
        if next_page:
            next_page_url = response.urljoin(next_page)
            yield SeleniumRequest(url=next_page_url, callback=self.parse)

# 输出 到execl中
# scrapy crawl jd_phone -o huawei_phones.csv

# pip install pandas openpyxl
# pipelines.py
import pandas as pd

class ExcelExportPipeline:
    def open_spider(self, spider):
        # 在爬虫开始时初始化pandas DataFrame
        self.items = []

    def close_spider(self, spider):
        # 在爬虫结束时将DataFrame保存到Excel文件
        df = pd.DataFrame(self.items)
        df.to_excel('huawei_phones.xlsx', index=False)

    def process_item(self, item, spider):
        # 将每个item添加到列表中
        self.items.append(item)
        return item

# settings.py
ITEM_PIPELINES = {
    'myproject.pipelines.ExcelExportPipeline': 300,
}

```



###### 自定义组件

```python
from scrapy import signals
import scrapy
from selenium import webdriver
import time
from selenium.webdriver.support.ui import WebDriverWait
from scrapy.http import HtmlResponse
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By 

class SeleniumDownloaderMiddleware:
    
    def __init__(self):
      	option = webdriver.ChromeOptions()  # 实例化一个浏览器对象
        option.add_argument('--headless')       # 无界面运行
        option.add_argument('--disable-gpu')    # 禁止gpu加速
        option.add_argument("no-sandbox")       # 取消沙盒模式
        option.add_argument("disable-blink-features=AutomationControlled")  # 禁用启用Blink运行时的功能
        option.add_experimental_option('excludeSwitches', ['enable-automation'])    # 开发者模式
        
    		self.driver = webdriver.Chrome(options=option)  # 创建一个无头浏览器
        self.driver.maximize_window()
        self.wait = WebDriverWait(self.driver, 10)

    def __del__(self):
        self.driver.close()

    def process_request(self, request, spider):
        offset = request.meta.get('offset', 1)
        self.driver.get(request.url)
        time.sleep(1)
        if offset > 1:
            self.driver.find_element_by_xpath('.//*[@class="anticon"]').click()
        #html = self.driver.page_source
        #self.wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '.m-itemlist .items .item')))
        return scrapy.http.HtmlResponse(url = request.url, 
                                        body = self.driver.page_source.encode('utf-8'), 
                                        encoding = 'utf-8', 
                                        request = request, 
                                        status = 200)

```



#### Pipeline-自定义

```python
ITEM_PIPELINES = {
		# 项目目录名.模块名.类名:优先级(1-1000不等)
   "scrapyspider.pipelines.CarMysqlPipeline": 300,
}
```

##### mysql

```python
import pymysql
from .settings import *

# create database cardb charset utf8;
# use cardb;
# create table cattab(name varchar(200),price varchar(100),link varchar(300) )charset=utf8;
class CarMysqlPipeline(object):
    def __init__(self):
        self.db = None  # 初始化表
        self.cur = None  # 初始化游标对象

    def open_spider(self, spider):
        """
        爬虫程序开始时，只执行一次，一般用于数据库的连接
        """
        self.db = pymysql.connect(host=MYSQL_HOST, user=MYSQL_USER, password=MYSQL_PWD, database=MYSQL_DB,
                                  charset=CHARSET)  # 数据库连接
        self.cur = self.db.cursor()  # 创建游标对象

    def process_item(self, item, spider):
        ins = 'insert into cartab values(%s,%s,%s)'  # 写sql语句
        li = [
            item["name"].strip(),
            item["price"].strip(),
            item["link"].strip()
        ]
        self.cur.execute(ins, li)
        self.db.commit()  # 提交到数据库执行
        return item

    def close_spider(self, spider):
        """
        爬虫程序结束时，只执行一次，一般用于数据库的断开
        """
        self.cur.close()  # 关闭游标
        self.db.close()  # 关闭表
```

##### Mongo

```python
from pymongo import MongoClient
from middle.settings import HOST
from middle.settings import PORT
from middle.settings import DB_NAME
from middle.settings import SHEET_NAME


class MiddlePipeline(object):
    def __init__(self):
        client = MongoClient(host=HOST, port=PORT)
        my_db = client[DB_NAME]
        self.sheet = my_db[SHEET_NAME]

    def process_item(self, item, spider):
        self.sheet.insert(dict(item))
        return item
# 构造注入配置
# def __init__(self, mongo_uri, mongo_db, mongo_port):
#     self.mongo_uri = mongo_uri
#     self.mongo_db = mongo_db
#     self.mongo_port = mongo_port
# @classmethod
# def from_crawler(cls, crawler):
#     return cls(mongo_uri=crawler.settings.get('MONGO_URI'),
#                mongo_db=crawler.settings.get('MONGO_DB'),
#                mongo_port=crawler.settings.get('MONGO_PORT')
#                )
```

##### 图片下载

> settings.py中配置
>
> IMAGES_STORE = './images'	# 需要设置存储图片的路径

```python
from scrapy import Request
from scrapy.exceptions import DropItem
from scrapy.pipelines.images import ImagesPipeline
# 需要设置存储图片的路径

class ImagePipeline(ImagesPipeline):
   def file_path(self, request, response=None, info=None):
       url = request.url
       file_name = url.split('/')[-1]
       return file_name

   def item_completed(self, results, item, info):
    	 # 逻辑处理
       image_paths = [x['path'] for ok, x in results if ok]
       if not image_paths:
           raise DropItem('Image Downloaded Failed')
       return item

   def get_media_requests(self, item, info):
       yield Request(item['image_url'])
       # 下载图片，如果传过来的是集合需要循环下载
       # meta里面的数据是从spider获取，然后通过meta传递给下面方法：file_path
       # yield scrapy.Request(url=item['url'],meta={'name':item['name']})
```



### 第一个爬虫

```python
from scrapy.spiders import Spider

headers={
  "User-Agent": "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
}

class BlogSpider(Spider):
		# 用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字
    name = 'douban'
    # 包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 
    # 后续的URL则从初始的URL获取到的数据中提取
    #start_urls = ['https://book.douban.com']
    
    def start_requests(self):
		  print("==============start_requests==============")
      #yield scrapy.Request(url="https://book.douban.com", callback=self.parse)
      yield scrapy.Request(url="https://book.douban.com", headers=headers, callback=self.parse)
      #增加代理
      #scapy.Request(url=url, callback=self.parse,headers={"User-Agent": "scrape web"},meta={"proxy": "http:/154.112.82.262:8050"})

		# spider的一个方法。 被调用时，每个初始URL完成下载后生成的 Response 对象将会作为唯一的参数传递给该函数。 
    # 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 Request 对象
    def parse(self, response):
        # 下载文件（第一个是缩略图地址，第二个参数是文件保存地址+文件名称+后缀名）
        #urllib.request.urlretrieve('文件url地址',d_path+"/" + name + ".jpg")
        titles = response.xpath('//a[@class="post-title-link"]/text()').extract()
        articlItem = DouanItem()
        for title in titles:
          	articlItem['title'] = title.strip()
            print title.strip()
        titles = response.xpath('//*[@class="slide-list"]//li/div/a')
        for item in titles:
            print item.xpath('./@title').extract_first()
        yield articlItem
        
        
```

#### Items.py

```python
class DouanItem(scrapy.Item):
    title = scrapy.Field()
```

#### pipelines.py

```python
class DouanPipeline: 
    def process_item(self, item, spider):
        return item
```

##### Item管道主要有4个方法，分别是

```
1）open_spider(spider)
2）close_spider(spider)
3）from_crawler(cls,crawler)
4）process_item(item,spider)
```

### xpath 基础使用

#### 参考如下

```

```

博客网站：https://zhuanlan.zhihu.com/p/342903085



### 启动爬虫

#### 部署启动

```python
#打开终端进入项目所在路径(即:scrapyspider路径下)运行下列命令:
python -m  scrapy crawl douban

scrapy crawl douban
# 爬虫数据数据以json格式输出到文件 JSON，JSON lines，CSV，XML
scrapy crawl dmoz -o douban.json -t json 
```

#### 本地启动

##### 本地project目录下创建，以下py文件填加如下内容

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from scrapy.cmdline import execute
import os
import sys


# 第一种模式
if __name__ == '__main__':
    sys.path.append(os.path.dirname(os.path.abspath(__file__)))
    execute(['scrapy', 'crawl', 'douban'])

# 第二种方式
from scrapy import cmdline
cmdline.execute('scrapy crawl douban'.split())
```

### 自定义代理池

#### 配置组件

```python
PROXY_URL = 'http://127.0.0.1:5000/proxy'

DOWNLOADER_MIDDLEWARES = {
  'XXXXXX.middlewares.ProxyMiddleware': 543, #启动
  'scrapy.downloadermiddleware.httpproxy.HttpProxyMiddleware': None #关闭
}
```

#### 代码

```python
import json
import requests
import logging

class ProxyMiddleware(object):
    def __init__(self, proxy_url):
        self.logger = logging.getLogger(__name__)
        self.proxy_url = proxy_url

    def get_random_proxy(self):
        try:
            response = requests.get(self.proxy_url)
            if response.status_code == 200:
                p = json.loads(response.text)
                proxy = '{}:{}'.format(p.get('ip'), p.get('port'))
                print('get proxy ...')
                ip = {"http": "http://" + proxy, "https": "https://" + proxy}
                # 验证一下，代理池能否访问
                r = requests.get("http://www.baidu.com", proxies=ip, timeout=4)
                if r.status_code == 200:
                    return proxy
        except:
            print('get proxy again ...')
            return self.get_random_proxy()

    def process_request(self, request, spider):
      			# 随机获取代理地址
            proxy = self.get_random_proxy()
            if proxy:
                self.logger.debug('======' + '使用代理 ' + str(proxy) + "======")
                # 增加代理
                request.meta['proxy'] = 'https://{proxy}'.format(proxy=proxy)

    def process_response(self, request, response, spider):
        if response.status != 200:
            print("again response ip:")
            request.meta['proxy'] = 'https://{proxy}'.format(proxy=self.get_random_proxy())
            return request
        return response

    @classmethod
    def from_crawler(cls, crawler):
        settings = crawler.settings
        return cls(
            proxy_url=settings.get('PROXY_URL')
        )
```

### 参考agent 基础信息

```
[
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Acoo Browser; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)",
    "Mozilla/4.0 (compatible; MSIE 7.0; AOL 9.5; AOLBuild 4337.35; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
    "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
    "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
    "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
    "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
    "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
]
```

