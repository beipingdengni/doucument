# 浏览器滑动验证码



### QQ空间验证

参考博客网址

> https://www.aneasystone.com/archives/2018/03/python-selenium-geetest-crack.html

```python
import cv2
import numpy as np
import os


import time
from selenium import webdriver
 
 # 1、创建一个driver对象，访问qq登录页面
browser = webdriver.Chrome()
browser.get("https://qzone.qq.com/")

# 2、输入账号密码
# 2.0 点击切换到登录的iframe
browser.switch_to.frame('login_frame')
# 2.1 点击账号密码登录
browser.find_element_by_id('switcher_plogin').click()
# 2.2定位账号输入框，输入账号
browser.find_element_by_id("u").send_keys("123292678")
# 2.3定位密码输入输入密码
browser.find_element_by_id("p").send_keys("PYTHON01")
# 3、点击登录
browser.find_element_by_id('login_button').click()
time.sleep(3)

# 4、模拟滑动验证
# 4.1切换到滑动验证码的iframe中
tcaptcha = browser.find_element_by_id("tcaptcha_iframe")
browser.switch_to.frame(tcaptcha)
# 4.2 获取滑动相关的元素
# 选择拖动滑块的节点
slide_element = browser.find_element_by_id('tcaptcha_drag_thumb')
# 获取滑块图片的节点
slideBlock_ele = browser.find_element_by_id('slideBlock')
# 获取缺口背景图片节点
slideBg = browser.find_element_by_id('slideBg')
# 4.3计算滑动距离
sc = SlideVerificationCode(save_image=True)
distance = sc.get_element_slide_distance(slideBlock_ele,slideBg)
# 滑动距离误差校正，滑动距离*图片在网页上显示的缩放比-滑块相对的初始位置
distance = distance*(280/680) - 22
print("校正后的滑动距离",distance)
# 4.4、进行滑动
sc.slide_verification(browser,slide_element,distance=100)




def get_element_slide_distance(self, slider_ele, background_ele, correct=0):
    """
    根据传入滑块，和背景的节点，计算滑块的距离
    
    该方法只能计算 滑块和背景图都是一张完整图片的场景，
    如果背景图是通过多张小图拼接起来的背景图，
    该方法不适用，请使用get_image_slide_distance这个方法
    :param slider_ele: 滑块图片的节点
    :type slider_ele: WebElement
    :param background_ele: 背景图的节点
    :type background_ele:WebElement
    :param correct:滑块缺口截图的修正值，默认为0,调试截图是否正确的情况下才会用
    :type: int
    :return: 背景图缺口位置的X轴坐标位置（缺口图片左边界位置）
    """
    # 获取验证码的图片
    slider_url = slider_ele.get_attribute("src")
    background_url = background_ele.get_attribute("src")
    # 下载验证码背景图,滑动图片
    slider = "slider.jpg"
    background = "background.jpg"
    self.onload_save_img(slider_url, slider)
    self.onload_save_img(background_url, background)
    # 读取进行色度图片，转换为numpy中的数组类型数据，
    slider_pic = cv2.imread(slider, 0)
    background_pic = cv2.imread(background, 0)
    # 获取缺口图数组的形状 -- 缺口图的宽和高
    width, height = slider_pic.shape[::-1]
    print(slider_pic.shape[::-1])
    print("==============")
    # 将处理之后的图片另存
    slider01 = "slider01.jpg"
    background_01 = "background01.jpg"
    cv2.imwrite(background_01, background_pic)
    cv2.imwrite(slider01, slider_pic)
    # 读取另存的滑块图
    slider_pic = cv2.imread(slider01)
    # 进行色彩转换
    slider_pic = cv2.cvtColor(slider_pic, cv2.COLOR_BGR2GRAY)
    # 获取色差的绝对值
    slider_pic = abs(255 - slider_pic)
    # 保存图片
    cv2.imwrite(slider01, slider_pic)
    # 读取滑块
    slider_pic = cv2.imread(slider01)
    # 读取背景图
    background_pic = cv2.imread(background_01)
    # 比较两张图的重叠区域
    result = cv2.matchTemplate(slider_pic, background_pic, cv2.TM_CCOEFF_NORMED)
    # 获取图片的缺口位置
    top, left = np.unravel_index(result.argmax(), result.shape)
    # 背景图中的图片缺口坐标位置
    print("当前滑块的缺口位置：", (left, top, left + width, top + height))
    return left

def slide_verification(self, driver, slide_element, distance):
    """
    滑动滑块进行验证
    :param driver: driver对象
    :type driver:webdriver.Chrome
    :param slide_element: 滑块的元组
    :type slider_ele: WebElement
    :param distance: 滑动的距离
    :type: int
    :return:
    """
    # 获取滑动前页面的url地址
    start_url = driver.current_url
    print("需要滑动的距离为：", distance)
    # 根据滑动距离生成滑动轨迹
    locus = self.get_slide_locus(distance)
    print("生成的滑动轨迹为:{}，轨迹的距离之和为{}".format(locus, distance))
    # 按下鼠标左键
    ActionChains(driver).click_and_hold(slide_element).perform()
    time.sleep(0.5)
    # 遍历轨迹进行滑动
    for loc in locus:
    time.sleep(0.01)
    ActionChains(driver).move_by_offset(loc, random.randint(-5, 5)).perform()
    ActionChains(driver).context_click(slide_element)
    # 释放鼠标
    ActionChains(driver).release(on_element=slide_element).perform()
```

