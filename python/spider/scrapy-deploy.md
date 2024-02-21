# scrapyd

Scrapyd是用于部署和运行Scrapy爬虫的应用程序。它使您可以使用JSON API部署（上传）项目并控制其爬虫。

官方文档 https://scrapyd.readthedocs.io/

> pip install scrapyd

启动：scrapyd  ，访问地址：http://127.0.0.1:6800/

注意修改配置

> scrapyd配置*\Lib\site-packages\scrapyd中的default_scrapyd.conf：
>
> ​	将bind_address = 127.0.0.1改为bind_address = 0.0.0.0

学习博客文章：https://blog.csdn.net/weixin_38924500/article/details/110952553

## scrapy-client

scrapy-client它允许我们将本地的scrapy项目打包发送到scrapyd 这个服务端（**前提是服务器scrapyd正常运行**）

官方文档https://pypi.org/project/scrapyd-client/

> pip install scrapy-client

## ScrapydWeb

ScrapydWeb：用于Scrapyd集群管理的Web应用程序，支持Scrapy日志分析和可视化。

官方文档：https://pypi.org/project/scrapydweb/

# Gerapy（平台）

Gerapy 是一款分布式爬虫管理框架，支持 Python 3，基于 Scrapy、Scrapyd、Scrapyd-Client、Scrapy-Redis、Scrapyd-API、Scrapy-Splash、Jinjia2、Django、Vue.js 开发，Gerapy 可以帮助我们：

- 更方便地控制爬虫运行
- 更直观地查看爬虫状态
- 更实时地查看爬取结果
- 更简单地实现项目部署
- 更统一地实现主机管理
- 更轻松地编写爬虫代码

> GitHub：https://github.com/Gerapy/Gerapy

## 安装

> pip3 install gerapy

## 初始化启动

接下来我们来开始使用 Gerapy，首先利用如下命令进行一下初始化，在任意路径下均可执行如下命令：

> gerapy init

执行完毕之后，本地便会生成一个名字为 gerapy 的文件夹，接着进入该文件夹，可以看到有一个 projects 文件夹，我们后面会用到。

紧接着执行数据库初始化命令：

> cd gerapy ; gerapy migrate

这样它就会在 gerapy 目录下生成一个 SQLite 数据库，同时建立数据库表

> gerapy runserver  或 `gerapy runserver 0.0.0.0:8000` 