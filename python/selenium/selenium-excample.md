# selenium案列

```python
# 设置浏览器参数
option = ChromeOptions()
# options.add_extension(extension_file_path) #插件扩展
# options.add_experimental_option("detach", True) # 如果将参数设置detach为 true，只要不向驱动程序发送退出命令，浏览器在进程结束后仍保持打开状态。
# options.add_experimental_option('excludeSwitches', ['disable-popup-blocking']) #阻止打开弹出窗口

# 静音播放
option.add_argument('-mute-audio')
# 忽略证书错误
option.add_argument('--ignore-certificate-errors')
option.add_argument('--ignore-ssl-errors')
option.add_argument('--ignore-ssl-error')
# 忽略证书错误，即使访问的网站使用的是不安全的 HTTPS 协议，也可以不做证书验证直接访问
option.add_experimental_option('excludeSwitches', ['ignore-certificate-errors'])
# 添加一个参数，该参数的键为 --ignore-certificate-errors-spki-list，值为空字符串。该参数用于忽略证书错误，其中 spki-list 是一种可选参数，表示公钥 pinning 列表，用于验证服务器证书。在这里，空字符串表示不对 pinning 列表进行任何验证，直接忽略所有证书错误。
option.add_argument('---ignore-certificate-errors-spki-list')
# 禁用扩展
option.add_argument('--disable-extensions')
# 将 Chrome 浏览器的日志级别设置为 "info" 级别
option.add_argument('log-level=2')
service = webdriver.ChromeService(
  service_args=[
    '--log-level=DEBUG', # 日志级别
    '--disable-build-check' #Chromedriver 和 Chrome 浏览器版本应匹配，如果不匹配，驱动程序将出错。如果禁用构建检查，则可以强制驱动程序与任何版本的 Chrome 一起使用。请注意，这是一项不受支持的功能，并且不会调查错误。
  ], 
  log_output=subprocess.STDOUT, # 输出日志
  log_output='./chrome_log'
                                  ）
                                 
options = webdriver.ChromeOptions()
options.add_argument('lang=zh_CN.utf-8')  # 设置默认编码为utf-8
options.add_argument('--no-sandbox') # # 以最高权限运行
# options.add_argument('User-Agent=Mozilla/5.0 (Linux; U; Android 8.1.0; zh-cn; BLA-AL00 Build/HUAWEIBLA-AL00) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/57.0.2987.132 MQQBrowser/8.9 Mobile Safari/537.36')
#options.add_argument('--user-agent=""')  # 设置请求头的User-Agent
#options.add_argument('--window-size=1280x1024')  # 设置浏览器分辨率（窗口大小）
options.add_argument('--start-maximized')  # 最大化运行（全屏窗口）,不设置，取元素会报错
options.add_argument('--disable-infobars')  # 禁用浏览器正在被自动化程序控制的提示
#options.add_argument('--incognito')  # 隐身模式（无痕模式）
#options.add_argument('--hide-scrollbars')  # 隐藏滚动条, 应对一些特殊页面
#options.add_argument('--disable-javascript')  # 禁用javascript
options.add_argument('--blink-settings=imagesEnabled=false')  # 不加载图片, 提升速度
options.add_argument('--headless')  # 浏览器不提供可视化页面
#options.add_argument('--ignore-certificate-errors')  # 禁用扩展插件并实现窗口最大化
options.add_argument('--disable-gpu')  # 禁用谷歌浏览器GPU加速-配置1
options.add_argument('–disable-software-rasterizer')  # 禁用谷歌浏览器GPU加速-配置2
#options.add_argument('--disable-extensions')  #禁用扩展插件
#options.add_argument("--proxy-server=http://" + ip_port) # HTTP代理
options.add_experimental_option('useAutomationExtension', False) # # 取消chrome受自动控制提示
option.add_experimental_option("excludeSwitches", ['enable-automation'])  # 正常浏览器window.navigator.webdriver的值为undefined,而使用selenium访问则该值为true,该方法规避这种风险。
option.add_argument('--kiosk-printing')  #默认打印机进行打印

options.binary_location = r"C:\Program Files (x86)\Google\Application\chrome.exe"  # 手动指定使用的浏览器位置
option.add_experimental_option("debuggerAddress", "127.0.0.1:9222")  #调用原来的浏览器，不用再次登录即可重启

prefs = {"":""}
prefs["credentials_enable_service"] = False
prefs["profile.password_manager_enabled"] = False
option.add_experimental_option("prefs", prefs) # 屏蔽'保存密码'提示框
```

## 爬虫知乎

```python
from time import sleep
from selenium.webdriver.chrome.service import Service
from selenium.webdriver import Chrome,ChromeOptions
from selenium.webdriver.common.by import By
import warnings
 
def main():
    #忽略警告
    warnings.filterwarnings("ignore")
    # 创建一个驱动
    service = Service('chromedriver.exe')
    options = ChromeOptions()
    # 伪造浏览器
    options.add_experimental_option('excludeSwitches', ['enable-automation','enable-logging'])
    options.add_experimental_option('useAutomationExtension', False)
    # 创建一个浏览器
    driver = Chrome(service=service,options=options)
    # 绕过检测
    driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
        "source": """
               Object.defineProperty(navigator, 'webdriver', {
               get: () => false
               })
           """
    })
    # 打开知乎登录页面
    driver.get('https://www.zhihu.com/')
    sleep(30)
    # 点击搜索框
    driver.find_element(By.ID,'Popover1-toggle').click()
    # 输入内容
    driver.find_element(By.ID,'Popover1-toggle').send_keys('汉江大学')
    sleep(2)
    # 点击搜索图标
    driver.find_element(By.XPATH,'//*[@id="root"]/div/div[2]/header/div[2]/div[1]/div/form/div/div/label/button').click()
    # 等待页面加载完
    driver.implicitly_wait(20)
    # 获取标题
    title = driver.find_element(By.XPATH,'//*[@id="SearchMain"]/div/div/div/div/div[2]/div/div/div/h2/div/a/span').text
    # 点击阅读全文
    driver.find_element(By.XPATH,'//*[@id="SearchMain"]/div/div/div/div/div[2]/div/div/div/div/span/div/button').click()
    sleep(2)
    # 获取帖子内容
    content = driver.find_element(By.XPATH,'//*[@id="SearchMain"]/div/div/div/div/div[2]/div/div/div/div/span[1]/div/span/p').text
    # 点击评论
    driver.find_element(By.XPATH,'//*[@id="SearchMain"]/div/div/div/div/div[2]/div/div/div/div/div[3]/div/div/button[1]').click()
    sleep(2)
    # 点击获取更多评论
    driver.find_element(By.XPATH,'//*[@id="SearchMain"]/div/div/div/div/div[2]/div/div/div/div[2]/div/div/div[2]/div[2]/div/div[3]/button').click()
    sleep(2)
    # 获取评论数据的节点
    divs = driver.find_elements(By.XPATH,'/html/body/div[6]/div/div/div[2]/div/div/div/div[2]/div[3]/div')
    try:
        for div in divs:
            # 评论内容
            comment = div.find_element(By.XPATH,'./div/div/div[2]').text
            f.write(comment)  # 写入文件
            f.write('\n')
            print(comment)
    except:
        driver.close()
 
if __name__ == '__main__':
    # 创建文件存储数据
    with open('05.txt','a',encoding='utf-8')as f:
        main()
```

