# selenium案列



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

