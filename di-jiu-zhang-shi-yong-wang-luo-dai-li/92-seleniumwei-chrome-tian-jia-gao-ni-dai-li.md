## 9.2 Selenium 中为 Chrome 添加高匿代理

之前我们简单地实现了路由器的模拟登录与重启，但在实际的项目之中，频繁的地重启路由器，不见见对路由器正常工作产生影响，可能还会影响其他人的生活与工作，所以我们还需要采取其他方式来获取可用的代理。比如使用高匿代理，关于如何使用付费高匿代理，在相关的商家都会站点给出接入指南，但细心的你可能会发现，在相关的代码实例当中，你可能根本找到 Chromedriver 的接入示例，所以今天我们就简单介绍一下在 Selenium 中为 Chromedriver 添加高匿代理。

### 1. 案例引入
默认情况下，Chrome 的 --proxy-server="http://ip:port" 参数不支持设置用户名和密码认证。这样就使得"Selenium + Chrome Driver"无法使用 HTTP Basic Authentication 的HTTP代理。一种变通的方式就是采用IP地址认证，但在国内网络环境下，大多数用户都采用ADSL形式网络接入，IP是变化的，也无法采用IP地址绑定认证。因此迫切需要找到一种让Chrome自动实现HTTP代理用户名密码认证的方案。

Stackoverflow上有人分享了一种利用Chrome插件实现自动代理用户密码认证的方案非常不错，我们可用查看一下[详细内容](http://stackoverflow.com/questions/9888323/how-to-override-basic-authentication-in-selenium2-with-java-using-chrome-driver)。

在这个思路上，我们选择了阿布云代理进行实验，实现了自动化的 Chrome　插件插件，即根据指定的代理 “username:password@ip:port” 自动创建一个 Chrome 代理插件。

### 2. 代码示例
首先从 github 上下载相应的模板文件：

`git clone https://github.com/RobinDev/Selenium-Chrome-HTTP-Private-Proxy`

然后，在 settings.py 文件之中进行如下配置：
```
import random

__all__ = ["CHROME_PROXY_HELPER_DIR", 
           "CUSTOM_PROXY_EXTENSIONS_DIR",
           "proxyHost", 
           "proxyPort", 
           "proxyUser", 
           "proxyPass", 
           "mobileEmulation"]

#''Chrome-proxy-helper' Chrome代理模板插件目录
CHROME_PROXY_HELPER_DIR = '~/EBusiness/MobileTMall/Chrome-Proxy'
# 存储自定义 Chrome 代理拓展文件的目录
CUSTOM_PROXY_EXTENSIONS_DIR = '~/EBusiness/MobileTMall/chrome-proxy-extensions'

# 代理的相关设置
proxyHost = "proxy.abuyun.com"
proxyPort = 9020
proxyUser = "H0S0SJE671YU29WD"
proxyPass = "10AB134444DF23D4"

# 设置手机请求头
phone_ua = [
    'Mozilla/5.0 (Linux; U; Android 5.1; zh-cn; m1 metal Build/LMY47I) AppleWebKit/537.36 (KHTML, like Gecko)Version/4.0 Chrome/37.0.0.0 MQQBrowser/7.6 Mobile Safari/537.36',
    ......
]

# 设置浏览器相关参数  
width = 320
height = 640
pixel_ratio = 3.0

mobileEmulation = {
    "deviceMetrics":{
        "width":width,
        "height":height,
        "pixelRatio":pixel_ratio
    },
    "userAgent":random.choice(phone_ua)
}
```

最后编写爬虫脚本代码：

```
import os
import time
import sys
import zipfile

from pyvirtualdisplay import Display
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

from EBusiness.settings import *

# 不设置环境变量可能会报错
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__))) 
# python3环境变量设置在pycharm里面as root,在终端运行用sys来进行更改设置
sys.path.insert(0, BASE_DIR)  

def get_chrome_proxy_extension(proxy):
    """获取一个Chrome代理扩展,里面配置有指定的代理(带用户名密码认证) 
    proxy - 指定的代理,格式: username:password@ip:port 
    """ 
    extension_file_path = os.path.join(CUSTOM_PROXY_EXTENSIONS_DIR, '{}.zip'.format(proxy.replace(':', '_')))
    if not os.path.exists(extension_file_path):
        with zipfile.ZipFile(extension_file_path, 'w') as zf:
            zf.write(os.path.join(CHROME_PROXY_HELPER_DIR, 'manifest.json'), 'manifest.json')
            background_content = open(os.path.join(CHROME_PROXY_HELPER_DIR, 'background.js')).read()
            background_content = background_content.replace('%proxy_host', proxyHost)
            background_content = background_content.replace('%proxy_port', proxyPort)
            background_content = background_content.replace('%username', proxyUser)
            background_content = background_content.replace('%password', proxyPass)
            zf.writestr('background.js', background_content)
    return extension_file_path


class TMallSpider(object):
    
    def __init__(self):
        self.display = Display(visible=0)
        self.display.start()
        chrome_proxy = get_chrome_proxy_extension(proxy=proxyUser)
        options = webdriver.ChromeOptions()
        options.add_experimental_option('mobileEmulation', mobileEmulation)  # 包含随机请求头
        options.add_extension(chrome_proxy)
        self.driver = webdriver.Chrome(chrome_options=options)
        self.driver.delete_all_cookies()

    def get_proxy(self):
        self.driver.implicitly_wait(20)
        locator = (By.XPATH, '//*[@id="rightinfo"]/dl/dd[1]')
        self.driver.get('http://ip.chinaz.com/')
        WebDriverWait(self.driver, 20, 0.5).until(EC.presence_of_all_elements_located(locator))
        driver_ip = self.driver.find_element_by_xpath('//*[@id="rightinfo"]/dl/dd[1]').text
        self.driver.find_element_by_css_selector('#rightinfo > div > a').click()
        # time.sleep(0)
        # UA = self.driver.find_element_by_xpath('//*[@id="rightinfo"]/dl/dd[6]').text
        print({'User-Agent': UA, 'proxy': driver_ip})
        self.driver.quit()
        
if __name__ == "__main__":
    spider = TMallSpider()
    spider.get_proxy()
```
