## uiautomator2

github地址： https://github.com/openatx/uiautomator2

> api的元素使用：
>
> https://zhuanlan.zhihu.com/p/660221899
>
> 脚本安装：
>
> ```python
> import os
> # https://pypi.douban.com/simple  # 豆瓣镜像
> # https://pypi.tuna.tsinghua.edu.cn/simple  # 清华镜像
> mirror = " -i https://pypi.douban.com/simple"
> os.system("python -m pip install --upgrade pip" + mirror)  # 更新 pip
> os.system("pip install --pre -U uiautomator2" + mirror)  # 安装 uiautomator2
> os.system("pip install --pre weditor" + mirror)  # 安装 weditor
> os.system("python -m uiautomator2 init")  #安装 atx-agent 至手机
> 
> # 参考博客：https://zhuanlan.zhihu.com/p/596385083?utm_id=0
> ```

##  先查看设备`adb devices`

```python
# 安装
# pip install -U uiautomator2

# 打印设备
import uiautomator2 as u2
d = u2.connect() # connect to device
# 通过WiFi连接（确保手机与电脑处于同一个局域网并且能ping同手机）
# d = u2.connect('192.168.0.100')
print(d.info)

#第一种方式：通过手机wifi进行连接，参数为ip
d1 = u2.connect_wifi("xx.xx.xx.xx")
print(d1.info)

#第二种方式：通过手机序列号连接
d = u2.connect_usb("xxxx")
print(d.info)
print(d.device_info) #可以获取详细的设备信息

#第三种方式：通过adb-wifi连接，也就是adb tcpip模式，注意不药丢掉端口号
# #使用tcpip命令 , 开启远端adb,这一步需要手机通过USB连接到电脑
# adb tcpip 5555
# #其中192.168.3.2是手机的局域网IP地址,断开电脑
# adb connect 192.168.3.2:5555 
 d = u2.connect_adb_wifi("192.168.x.xx:5555")
 print(d.info)

# 启动app
d.app_stop("com.jingdong.app.mall")
	# 停止所有APP，excludes参数可以指定哪些APP不停止
	# d.app_stop_all(excludes=["com.android.browser", "com.android.bluetooth"])
d.app_start('com.jingdong.app.mall')
	# 直接启动APP，若wait设置为True则一直等到启动结束
		d.app_start("com.android.settings", wait=True)
	# 等待APP启动结束，默认20秒超时，启动后 返回pid，如果启动失败则pid为0
		pid = d.app_wait("com.android.settings", timeout=20)
    #pid2 = d.app_wait("com.android.settings", front=True) # 等待应用前台运行
		print(pid)

#文件传到
 # 外部往手机端推送
  d.push("foo.txt", "/sdcard/")
  d.push("foo.txt", "/sdcard/bar.txt") # push and rename
  # 从手机内部pull出来
  d.pull("/sdcard/foo.txt", "tmp.txt")

# 安装APP，参数可以是本地文件也可以是url
# d.app_install(r"C:\Users\admin\Desktop\IcyFtpServer_v1.0.apk")
# 卸载APP
# d.app_uninstall("com.ice.icyftpserver")
# 卸载所有APP，excludes参数指定要保留的APP
# d.app_uninstall_all(excludes=["app_uninstall_all"])

# d.app_clear('指定的包名')
d.xpath('//android.widget.ViewFlipper/android.widget.LinearLayout[1]').click()
# 记住开启，虚拟键盘的快速输入 【⚠️注意】
d.send_keys("茅台酒53度飞天500ml", clear=True)
	#	或者直接使用文本替换方式
	# d(resourceId="com.jd.lib.search.feature:id/adn").set_text('茅台酒53度飞天500ml')
  # clear_text() , get_text()
# 输入文本
d(resourceId="com.jd.lib.search.feature:id/at6", text="茅台酒53度飞天500ml").click()
# d(description="元素定位表达式").click() #选择文本点击
# d(text="元素定位表达式")
d.xpath('//*[@resource-id="com.jd.lib.search.feature:id/a68"]/android.widget.RelativeLayout[1]').click()
d.click(0.728, 0.905)
d.press("back")
#press: enter,home,search,delete,left,right,up,down,center....等

# 滑动
# 从sx，sy坐标滑动至ex，ey坐标
d.swipe(sx, sy, ex, ey)
# 扩展版的滑动操作： d.swipe_ext('left',scale=0.9)
# 先定位元素，再使用元素对象滑动： e = d(text = '活动')   e.swipe('down',steps=100)

# 上翻、下翻、左翻、右翻 还可以使用Direction作为参数
# from uiautomator2 import Direction
# # d.swipe_ext(Direction.FORWARD) # 页面下翻, 等价于 d.swipe_ext("up"), 只是更好理解
# # d.swipe_ext(Direction.BACKWARD) # 页面上翻
# # d.swipe_ext(Direction.HORIZ_FORWARD) # 页面水平右翻
# # d.swipe_ext(Direction.HORIZ_BACKWARD) # 页面水平左翻

# 检查元素是否存在
# d(resourceId="com.jd.lib.productdetail.feature:id/nd").exists()
# d.exists(description="需要检查的元素")
# 滚动查找某元素
# d.exists(scrollable=True, descriptionContains="需要检查的元素")
# deal_fetch = d(text='立即抢购') // if deal_fetch.exists() and deal_fetch.info['clickable'] // deal_fetch.click()

# d(text='支付宝').click(timeout=5)     # 5秒(超时时间)内等待元素出现后点击，超过后报错
# d(text='支付宝').click_exists(timeout=10.0)   # 超时时间内等待元素出现后点击，如果查找到元素点击返回布尔值true，否则返回false
# 点击并轮询对象直到消失（每隔interval时间点击一次，直到最大点击次数maxretry后返回一个布尔值），其中maxretry为最多点击次数，默认10；interval为轮询时间间隔，默认为1
# a1 = d(text="支付宝").click_gone(maxretry=10, interval=1.0) # 默认10；interval为轮询时间间隔，默认为1。

# init 所有的已经连接到电脑的设备，# 并安装ATX
python -m uiautomator2 init
# 高阶用法
# init and set atx-agent listen in all address
python -m uiautomator2 init --addr :7912
```

#### **uiautomator2 wifi连接手机**

```shell
【实施方法】
手机和电脑同时连接到同一个wifi上
1、开启远程adb
#开启远端adb,这一步需要手机通过USB连接到电脑
adb tcpip 5555
#结果如下：restarting in TCP mode port: 5555
#然后断开USB
adb connect 192.168.3.2:5555 
#其中192.168.3.2是手机的局域网IP地址
adb devices
#确认可以看到设备信息

2、通过adb命令启动uiautomator2的agent

db shell /data/local/tmp/atx-agent -d
3、通过uiautomator2脚本连接手机执行用例

import uiautomator2 as u2
d = u2.connect_wifi('192.168.3.2')
print(d.info)
发现可以成功执行

PS：因为有些操作系统上uiautomator2的agent无法自动拉起，所以需要手动通过adb命令拉起
```

### atx-agent(安装：`python -m uiautomator2 init`)

手动安装如下:

> https://github.com/openatx/atx-agent
>
> ```shell
> # 推送包
> adb push ~/Documents/project/soft/atx-agent/atx-agent /data/local/tmp
> # 修改权限
> adb shell chmod 755 /data/local/tmp/atx-agent
> # start 服务
> adb shell /data/local/tmp/atx-agent server -d
> # stop 服务
> adb shell /data/local/tmp/atx-agent server -d --stop
> 
> adb shell rm -rf /data/local/tmp/atx-agent
> 
> ```

### weditor 获取元素工具

```shell
# 安装
pip install -U weditor
# 启动 访问web
weditor # 启动web
# Windows系统可以使用命令在桌面创建一个快捷方式 weditor --shortcut
```

## adb(推荐使用android studio安装)

> brew install android-sdk
>
> ```shell
> export ANDROID_HOME=/Users/tianbeiping1/Library/Android/sdk
> export platform_tools=$ANDROID_HOME/platform-tools
> export PATH=$PATH:$platform_tools
> # 或
> echo 'export ANDROID_HOME=/usr/local/share/android-sdk' >> ~/.zshrc
> echo 'export PATH=$PATH:$ANDROID_HOME/tools' >> ~/.zshrc
> echo 'export PATH=$PATH:$ANDROID_HOME/platform-tools' >> ~/.zshrc
> # 生效
> source ~/.zshrc
> ```
>
> 手动下载安装：https://developer.android.com/tools/releases/platform-tools?hl=zh-cn
>
> 配置： ~/.zshrc

```shell
# 连接设备
adb connect localhost:5000
adb kill-server # 关闭服务
adb start-server # 重启服务

# 查看设备
adb device
# 安装
adb install app-release.apk 
# 设备：可以是IP 或者 设置名称
adb -s 指定设备 -r app.apk

adb shell pm list packages -3 #(===>查看外部安装包)
# 查看某个包的具体信息
adb shell dumpsys package com.jingdong.app.mall
# 查看当前正在运行的Activity
adb logcat | grep Displayed

# 将本机中的文件写入到设备
	# 在 Terminal 中输入下命令，其实这一步和git中的 push 有些相似之处
	adb push <本机文件路径/文件名> <设备路径>
# 将设备中的文件读取到本机中
	# 在 Terminal 中输入以下命令，其实这一步也和git中的 pull 比较相似
	adb pull <设备中文件路径/文件名> <本机路径>

# -s 可以在多个模拟器中指定一个进行操作
# -r 覆盖安装
adb -s emulator-5554 install -r ./JioChat_Poc_V1.0.7.1026_1912.apk
adb -s emulator-5554 uninstall com.jiochat.jiochatapp
```

## 模拟器

### mumu

> https://mumu.163.com/mac/

## Android Emulator(M1)

> https://github.com/google/android-emulator-m1-preview/releases
>
> 需要配置：[adb到模拟器中参考](https://blog.csdn.net/iYNing/article/details/129314021)
>
> https://www.jianshu.com/p/830b3ab00196

## 代理抓包

#### burpsuite

Charles、mitmdump、Appium

### mitmdump

安装：brew install mitmproxy

> mitmweb    # mitmproxy有三种启动方式，此处使用的命令可以提供一个web交互界面,  交互界面地址：localhost:8081；
>
> > `mitmweb -s proxy.py -p 端口`，-s和-p选项可以省略。
>
> mitmproxy  # 可以通过命令过滤请求；

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

> mitmdump -s script.py

