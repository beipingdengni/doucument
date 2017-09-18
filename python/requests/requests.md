### pythons中，用作网络请求。。 <br/> 网络爬虫

官方文档
===========

[中文文档](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html)
----------
[英文文档](http://www.python-requests.org/en/master/user/quickstart/#custom-headers)
----------

> 安装 requests
>
>  基本用法
``` Python
    pip install requests

    r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
    head = r.headers['content-type']
    coding = r.encoding
    text = r.text
    json = r.json()

    print(head)
    print(coding)
    print(text)
    print(json)

```

