

## 自动安装驱动

参考博客： https://zhuanlan.zhihu.com/p/648948497

```python
# pip install webdriver-manager
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
service = ChromeService(executable_path=ChromeDriverManager().install())
driver = webdriver.Chrome(service=service)
# 业务逻辑
driver.quit()

#传统模式加载
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
options = webdriver.ChromeOptions()
options.add_experimental_option("excludeSwitches", ["enable-automation"])
options.add_experimental_option("useAutomationExtension", False)
service = ChromeService(executable_path=CHROMEDRIVER_PATH)
driver = webdriver.Chrome(service=service, options=options)



#查找元素
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


#等待元素出现
from selenium.webdriver.support.ui import WebDriverWait
el = WebDriverWait(driver, timeout=3).until(lambda d: d.find_element_by_tag_name("p"))

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