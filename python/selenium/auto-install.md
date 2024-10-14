

## 自动安装驱动

参考博客： https://zhuanlan.zhihu.com/p/648948497

```python
# pip install webdriver-manager
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
#
service = ChromeService(executable_path=ChromeDriverManager().install())
driver = webdriver.Chrome(service=service)
# 业务逻辑
driver.quit()
```

#### 本地驱动

```python
# 本地驱动
#options = webdriver.ChromeOptions()
#options.binary_location = chrome_bin
#driver = webdriver.Chrome(options=options)

#或
from selenium.webdriver.chrome.service import Service
from selenium.webdriver import Chrome,ChromeOptions
service = Service('./chromedriver')
options = ChromeOptions()
# 伪造浏览器
options.add_experimental_option('excludeSwitches', ['enable-automation','enable-logging']) # 正常浏览器
options.add_experimental_option('useAutomationExtension', False) # # 取消chrome受自动控制提示
# 加载
driver = webdriver.Chrome(options = options)
# 绕过检测
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
  "source": """
                Object.defineProperty(navigator, 'webdriver', {
                get: () => false
                })
            """
})

# 远程驱动
# options = webdriver.ChromeOptions()
# options.add_experimental_option("debuggerAddress", "127.0.0.1:5003")
# driver = webdriver.Chrome(options = options)
```



### 远程驱动

```shell
# cd /Applications/Google Chrome.app/Contents/MacOS/
chrome --remote-debugging-port=9222 --user-data-dir=<some directory>
#--user-data-dir 必须得加，然后打开 http://loacalhost:9222 ，就能看到你开启的chrome实例中所有打开的标签页面


```

Firefox

```shell
firefox.exe -start-debugger-server=4444
# open -n /Applications/Firefox.app -start-debugger-server=4444

# docker run -d --name firefox -e TZ=Asia/Hong_Kong  -e DISPLAY_WIDTH=1920 -e DISPLAY_HEIGHT=1080 -e KEEP_APP_RUNNING=1 -e ENABLE_CJK_FONT=1  -e VNC_PASSWORD=admin  -p 5800:5800 -p 5900:5900 -v /data/firefox/config:/config:rw --shm-size 2g jlesage/firefox
#参数介绍
-e TZ=Asia/Hong_Kong       # 设置时区
-e DISPLAY_WIDTH=1920
-e DISPLAY_HEIGHT=1080     #设置显示的高宽
-e KEEP_APP_RUNNING=1      # 保持启动状态
-e ENABLE_CJK_FONT=1       # 防止显示页面时中文乱码
-e SECURE_CONNECTION=1     # 启用HTTPS功能
-e VNC_PASSWORD=admin  #设置VNC的访问密码,自定义即可
-p 5800:5800               #访问firefox的web端口
-p 5900:5900               #VNC端口
-v /data/irefox/config:/config:rw         # 容器挂载目录，存放firefox数据
--shm-size 2g               # 设置容器的内存资源为2g 
```



#### Mac Chrome添加启动参数

参数临时

```shell
open -n /Applications/Google\ Chrome.app --args --enable-speech-input --disable-web-security --remote-debugging-port=9222
# -n 打开新的进程
# --enable-speech-input的含义是启用语音输入功能
# --remote-debugging-port=9222 远程debug端口
# --user-data-dir= 临时数据存储目录
```

文件配置

```shell
#打开Terminal cd "/Applications/Google Chrome.app/Contents/MacOS/"
#重命名的启动脚本 sudo mv "Google Chrome" Google.real
#将下面的代码复制到文本, 保存为Google Chrome.
#并且复制到 /Applications/Google Chrome.app/Contents/MacOS/
#添加脚步 sudo printf '#! /bin/bash\ncd "/Applications/Google Chrome.app/Contents/MacOS"\n"/Applications/Google Chrome.app/Contents/MacOS/Google.real" --disable-web-security --user-data-dir "$@"\n'
#給你新的脚本添加执行权限: sudo chmod u+x "Google Chrome"

# ”Google Chrome“文件内容如下：
# #!/bin/bash
# cd "/Applications/Google Chrome.app/Contents/MacOS"
#"/Applications/Google Chrome.app/Contents/MacOS/Google.real" --args --disable-web-security --user-data-dir
```

### 传统模式加载

```python
#传统模式加载
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
# 创建Chrome的选项实例
options = webdriver.ChromeOptions()
# 如果需要，可以设置其他选项，例如无头模式、禁用GPU等
# options.add_argument('--headless')
# options.add_argument('--disable-gpu')
options.add_experimental_option('useAutomationExtension', False) # # 取消chrome受自动控制提示
option.add_experimental_option("excludeSwitches", ['enable-automation'])  # 正常浏览器
service = ChromeService(executable_path=CHROMEDRIVER_PATH)
driver = webdriver.Chrome(service=service, options=options)
```

###  设置隐式等待时间为

```python
driver = webdriver.Chrome()  
driver.implicitly_wait(10)  # 设置隐式等待时间为10秒  
driver.get("http://www.example.com")  
element = driver.find_element_by_id("some_id")
```

### 等待元素出现

```python
from selenium import webdriver  
from selenium.webdriver.common.by import By  
from selenium.webdriver.support.ui import WebDriverWait  
from selenium.webdriver.support import expected_conditions as EC  
  
driver = webdriver.Chrome()  
driver.get("http://www.example.com")  
try:  
    element = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, "some_id"))
 		element = WebDriverWait(driver, 10).until(EC.visibility_of_element_located((By.XPATH, '//*[@class="wrap"]/')))
finally:  
    driver.quit()
```

### 查找元素

```python
from selenium.webdriver.common.keys import Keys

driver.findElement(By.className("className"));
driver.findElement(By.cssSelector(".className"));

#  如果你查找的是多个元素，只需要将其中的find_element替换成find_elements即可。
driver.find_element(By.XPATH,'XPATH')
driver.find_element(By.CLASS_NAME,'CLASS_NAME')
driver.find_element(By.CSS_SELECTOR,'CSS_SELECTOR')
driver.find_element(By.ID,'ID')
driver.find_element(By.LINK_TEXT,'LINK_TEXT')
driver.find_element(By.PARTIAL_LINK_TEXT,'PARTIAL_LINK_TEXT')
driver.find_element(By.TAG_NAME,'TAG_NAME')

all_memeber_ele = driver.find_elements(By.XPATH, '//div[@role="group"]/div[@role="listitem"]')
    for member in all_memeber_ele:
        member.click()
      	# 已经有回复的消息，不发生
        reciv_num_ele=member.find_element(By.XPATH,'//*[@id="_39539886-0"]/div[1]/span/span')
        if reciv_num_ele is None:
          	# 
            getDriver().find_element(By.XPATH, '//*[@id="boss-chat-editor-input"]').send_keys("你好")
            # 输入会车键
            getDriver().find_element(By.XPATH, '//*[@id="boss-chat-editor-input"]').send_keys(Keys.ENTER)
```

### iframe切换

```python
# 切换至iframe，以便于操作该iframe中的元素
driver.switch_to.frame(driver.find_element('xpath', '//div[@class="QQMailSdkTool_login_loginBox_qq"]/iframe'))
# 切换至第二层iframe
driver.switch_to.frame(driver.find_element('id', 'ptlogin_iframe'))
driver.find_element('link text', '注册账号').click()
driver.switch_to.default_content()  # 切换回默认窗体，也就是最原始的html之中
```

### 发送鼠标动作

```shell
常用操作：
1. 删除键（BackSpace）：send_keys(Keys.BACK_SPACE)  
2. 空格键（Space）：send_keys(Keys.SPACE)       
3. 制表键（Tab）：send_keys(Keys.TAB)         
4. 回退键（Esc）：send_keys(keys.ESCAPE)      
5. 回车键（Enter）：send_keys(Keys.ENTER)       
6. Ctrl+A：send_keys(Keys.CONTROL,'a') 
7. Alt+C：send_keys(Keys.ALT,'c')     
8. Alt+V：send_keys(Keys.ALT,'v')           
9. 按下某个键盘上的键：key_down(value, element=None)
10.松开某个键：key_up(value, element=None) 
11.键盘右键：send_keys(Keys.RIGHT)

# 按下某个键盘上的键：key_down(value, element=None）
# 松开某个键：key_up(value, element=None)
# value：要发送的修饰键。在Keys类中定义。element：发送密钥的元素。如果没有，则向当前聚焦的元素发送一个键
# 只能与修饰键（Control、Alt 和 Shift）一起使用
# 构造ActionChains对象：ActionChains(driver) perform()执行所有存储的操作
ActionChains(driver).key_down(Keys.CONTROL).send_keys('a').key_up(Keys.CONTROL).perform()

# 滚动元素页面内可见  
# element.scrollIntoView（）; // 等同于element.scrollIntoView(true) ,滚动到顶部
driver.execute_script("arguments[0].scrollIntoView(true);", single_elemet_item)
#element.scrollIntoView（scrollIntoViewOptions）; //对象参数
# true 元素的顶部将对齐到可滚动祖先的可见区域的顶部。
#     	对应于scrollIntoViewOptions: {block: “start”, inline: “nearest”}。这是默认值
# false 元素的底部将与可滚动祖先的可见区域的底部对齐。
# 			对应于scrollIntoViewOptions: {block: “end”, inline: “nearest”}
# scrollIntoViewOptions	[可选]，目前这个参数浏览器对它的支持并不好，可以查看下文兼容性详情
# behavior	[可选]定义过渡动画。“auto”,“instant"或"smooth”。默认为"auto"。
# block	[可选] “start”，“center”，“end"或"nearest”。默认为"center"。
# inline	[可选] “start”，“center”，“end"或"nearest”。默认为"nearest"。
```



### 动作API(如滑动、键盘、鼠标)

```python
from selenium.webdriver.common.action_chains import ActionChains,ScrollOrigin

#动作API
	#暂停（pause）
  clickable = driver.find_element(By.ID, "clickable")
	ActionChains(driver)\
        .move_to_element(clickable)\
        .pause(1)\
        .click_and_hold()\
        .pause(1)\
        .send_keys("abc")\
        .perform()
  #释放所有动作
  ActionBuilder(driver).clear_actions()

#键盘
	#按下某键，以输入shift+abc为例
  ActionChains(driver)\
    .key_down(Keys.SHIFT)\
    .send_keys("abc")\
    .perform()
  #弹起某键，以输入shift+a和shift+b为例
  ActionChains(driver)\
    .key_down(Keys.SHIFT)\
    .send_keys("a")\
    .key_up(Keys.SHIFT)\
    .send_keys("b")\
    .perform()
  #复制和粘贴
  cmd_ctrl = Keys.COMMAND if sys.platform == 'darwin' else Keys.CONTROL
	ActionChains(driver)\
        .send_keys("Selenium!")\
        .send_keys(Keys.ARROW_LEFT)\
        .key_down(Keys.SHIFT)\
        .send_keys(Keys.ARROW_UP)\
        .key_up(Keys.SHIFT)\
        .key_down(cmd_ctrl)\
        .send_keys("xvv")\
        .key_up(cmd_ctrl)\
        .perform()
#鼠标（鼠标点击保持，该方法将鼠标移动到元素中心与按下鼠标左键相结合）
	#助于聚焦特定元素
  clickable = driver.find_element(By.ID, "clickable")
  ActionChains(driver)\
    .click_and_hold(clickable)\
    .perform()
  #鼠标点击释放
  clickable = driver.find_element(By.ID, "click")
  ActionChains(driver)\
  	.click(clickable)\
  	.perform()
# 鼠标定义的5种按键
# 0——鼠标左键
# 1——鼠标中键
# 2——鼠标右键
# 3——X1（后退键）
# 4——X2（前进键）

#鼠标右击
   clickable = driver.find_element(By.ID, "clickable")
    ActionChains(driver)\
        .context_click(clickable)\
        .perform()
#按下鼠标3键
 action = ActionBuilder(driver)
    action.pointer_action.pointer_down(MouseButton.BACK)
    action.pointer_action.pointer_up(MouseButton.BACK)
    action.perform()
#按下鼠标4键
    action = ActionBuilder(driver)
    action.pointer_action.pointer_down(MouseButton.FORWARD)
    action.pointer_action.pointer_up(MouseButton.FORWARD)
    action.perform()
#鼠标双击
    clickable = driver.find_element(By.ID, "clickable")
    ActionChains(driver)\
        .double_click(clickable)\
        .perform()
#鼠标移动到元素上
    hoverable = driver.find_element(By.ID, "hover")
    ActionChains(driver)\
        .move_to_element(hoverable)\
        .perform()
#鼠标位移
	#从元素左顶边进行位移
    mouse_tracker = driver.find_element(By.ID, "mouse-tracker")
    ActionChains(driver)\
        .move_to_element_with_offset(mouse_tracker, 8, 11)\
        .perform()
  #从当前窗口左上角位移
    action = ActionBuilder(driver)
    action.pointer_action.move_to_location(8, 12)
    action.perform()
  #从当前鼠标位置位移
   ActionChains(driver)\
        .move_by_offset( 13, 15)\
        .perform()

#拖拽元素
	#该方法首先单击并按住源元素，移动到目标元素的位置，然后释放鼠标。
    draggable = driver.find_element(By.ID, "draggable")
    droppable = driver.find_element(By.ID, "droppable")
    ActionChains(driver)\
        .drag_and_drop(draggable, droppable)\
        .perform()
	#通过位移拖拽
    draggable = driver.find_element(By.ID, "draggable")
    start = draggable.location
    finish = driver.find_element(By.ID, "droppable").location
    ActionChains(driver)\
        .drag_and_drop_by_offset(draggable, finish['x'] - start['x'], finish['y'] - start['y'])\
        .perform()
		#笔（部分浏览器生效）由于笔只在部分浏览器生效，这里就不写了，如果你感兴趣，或者有需求可以去官方文档查看，这里贴出官方文档地址。

	#滚轮（只有谷歌内核浏览器生效）
  #滚动到某元素位置
		iframe = driver.find_element(By.TAG_NAME, "iframe")
		ActionChains(driver)\
    	.scroll_to_element(iframe)\
	    .perform()
	#定量滚动
    footer = driver.find_element(By.TAG_NAME, "footer")
    delta_y = footer.rect['y']
    ActionChains(driver)\
        .scroll_by_amount(0, delta_y)\
        .perform()
	#从一个元素滚动指定量
    iframe = driver.find_element(By.TAG_NAME, "iframe")
    scroll_origin = ScrollOrigin.from_element(iframe)
    ActionChains(driver)\
        .scroll_from_origin(scroll_origin, 0, 200)\
        .perform()
	#从一个元素滚动，并指定位移
    footer = driver.find_element(By.TAG_NAME, "footer")
    scroll_origin = ScrollOrigin.from_element(footer, 0, -50)
    ActionChains(driver)\
        .scroll_from_origin(scroll_origin, 0, 200)\
        .perform()
	#从一个元素的原点位移
    ActionChains(driver)\
        .scroll_from_origin(scroll_origin, 0, 200)\
        .perform()
```