## 代理抓包

#### burpsuite

Charles、mitmdump、Appium

### mitmdump

安装：brew install mitmproxy 、pip install mitmproxy

> `mitmweb -s proxy.py -p 端口`，-s和-p选项可以省略。
>
> mitmproxy  # 可以通过命令过滤请求；

可以提供一个web交互界面,  交互界面地址：localhost:8081；

> `mitmweb -p 8081`



手机先要连接代理IP，然后访问：http://mitm.it   下载安装ssl安全证书

安装证书,  手机端访问： http://mitm.it   iPhone下载后配置：**设置** -> **通用** -> **关于本机** -> **证书信任设置**，开启 mitmproxy 证书



script.py 是用来处理 mitmproxy 获取到的 request 和 response 的 .py 脚本

```python
import json
from urllib.parse import unquote
import re

def response(flow):
    # 提取请求的 url 地址
    request_url = flow.request.url
    # 通过 jd 字符串，过滤出 京东APP 的请求和返回数据
    if bool(re.search(r"jd", request_url)):     
        print("request_url >>> ", request_url)
        response_body = flow.response.text
        response_url = flow.request.url
        print("response_url >>> ", response_url)
        data = json.loads(response_body)
        ware_infos = data.get("wareInfo")
        goods_info = {}
        if ware_infos is not None:
            for ware_info in ware_infos:
                goods_info["wareId"]    = ware_info.get("wareId")
                goods_info["wname"]     = ware_info.get("wname")
                goods_info["jdPrice"]   = ware_info.get("jdPrice")
                goods_info["goodrate"]  = ware_info.get("good")
                goods_info["reviews"]   = ware_info.get("reviews")
                goods_info["shopId"]    = ware_info.get("shopId")
                goods_info["ShopName"]  = ware_info.get("goodShop").get("goodShopName")
                print(goods_info)
```

### mitmdump -s script.py

