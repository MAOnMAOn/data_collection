## 10.1 模拟登录豆瓣

今天我们就借鉴 github 上的一个项目 [xchaoinfo/fuck-login](https://github.com/xchaoinfo/fuck-login/blob/master/020douban/douban.py) 进行豆瓣模拟登录的讲解，下面就让我们一起感受一下吧。

### 1. 调试分析

首先，打开火狐浏览器的开发者工具，并观察页面，如果你的浏览器在近期进行过相关的登录操作，网站可能就不需要你进行验证码的输入，所以切记打开一个近期未进行登录操作的浏览器进行调试:

![](/assets/doubandenlu2.png)

进入经过跳转的页面以后，我们可以开始分析我们发出请求的请求头与请求参数：

![](/assets/cookises参数.png)

这里我们找到了我们准备请求的网址，请求头所需要的内容以及请求参数，可以准备代码了！

### 2. 引入代码

```
from os import remove
import http.cookiejar as cookielib
from urllib.request import urlretrieve

import requests
from bs4 import BeautifulSoup
from PIL import Image


url = 'https://accounts.douban.com/login'  # 302 重定向跳转

datas = {'source': 'index_nav'}

headers = {'Host':'www.douban.com',
           'Referer': 'https://www.douban.com/',
           'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0',
           'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
           'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
           'Accept-Encoding':'gzip, deflate, br'}

# 尝试使用cookie信息
session = requests.session()
session.cookies = cookielib.LWPCookieJar(filename='cookies')
try:
    session.cookies.load(ignore_discard=True)
except:
    print("Cookies未能加载")
    #cookies加载不成功，则输入账号密码信息
    datas['form_email'] = input('Please input your account:')
    datas['form_password'] = input('Please input your password:')
```

这里我们设置了 session 连接，并添加了相应的请求头参数，下面开始获取验证码：

```
def get_captcha():
    '''
    获取验证码及其ID
    '''
    r = requests.post(url, data=datas, headers=headers)
    page = r.text
    soup = BeautifulSoup(page, "html.parser")
    # 利用bs4获得验证码图片地址
    img_src = soup.find('img', {'id': 'captcha_image'}).get('src')
    urlretrieve(img_src, 'captcha.jpg')
    try:
        im = Image.open('captcha.jpg')
        im.show()
        im.close()
    except:
        print('到本地目录打开captcha.jpg获取验证码')
    finally:
        captcha = input('please input the captcha:')
        remove('captcha.jpg')
    captcha_id = soup.find(
        'input', {'type': 'hidden', 'name': 'captcha-id'}).get('value')
    return captcha, captcha_id
```

![](/assets/capusdfdsfv.png)

编写登录逻辑：

```
def login():
    try:
        captcha, captcha_id = get_captcha()
        # 增加表数据
        datas['captcha-solution'] = captcha
        datas['captcha-id'] = captcha_id
    except:
        pass
    login_page = session.post(url, data=datas, headers=headers)
    page = login_page.text
    soup = BeautifulSoup(page, "html.parser")
    result = soup.findAll('div', attrs={'class': 'title'})
    #进入豆瓣登陆后页面，打印热门内容
    for item in result:
        print(item.find('a').get_text())
    # 保存 cookies 到文件，
    # 下次可以使用 cookie 直接登录，不需要输入账号和密码
    session.cookies.save()
```

最后代码如下：

```
from os import remove
import http.cookiejar as cookielib
from urllib.request import urlretrieve

import requests
from bs4 import BeautifulSoup
from PIL import Image


url = 'https://accounts.douban.com/login'  # 302 重定向跳转

datas = {'source': 'index_nav'}

headers = {'Host':'www.douban.com',
           'Referer': 'https://www.douban.com/',
           'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0',
           'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
           'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
           'Accept-Encoding':'gzip, deflate, br'}

# 尝试使用cookie信息
session = requests.session()
session.cookies = cookielib.LWPCookieJar(filename='cookies')
try:
    session.cookies.load(ignore_discard=True)
except:
    print("Cookies未能加载")
    #cookies加载不成功，则输入账号密码信息
    datas['form_email'] = input('Please input your account:')
    datas['form_password'] = input('Please input your password:')


def get_captcha():
    '''
    获取验证码及其ID
    '''
    r = requests.post(url, data=datas, headers=headers)
    page = r.text
    soup = BeautifulSoup(page, "html.parser")
    # 利用bs4获得验证码图片地址
    img_src = soup.find('img', {'id': 'captcha_image'}).get('src')
    urlretrieve(img_src, 'captcha.jpg')
    try:
        im = Image.open('captcha.jpg')
        im.show()
        im.close()
    except:
        print('到本地目录打开captcha.jpg获取验证码')
    finally:
        captcha = input('please input the captcha:')
        remove('captcha.jpg')
    captcha_id = soup.find(
        'input', {'type': 'hidden', 'name': 'captcha-id'}).get('value')
    return captcha, captcha_id


def isLogin():
    '''
    通过查看用户个人账户信息来判断是否已经登录
    '''
    url = "https://www.douban.com/accounts/"
    login_code = session.get(url, headers=headers,
                             allow_redirects=False).status_code
    if login_code == 200:
        return True
    else:
        return False


def login():
    # 如果网页没有验证码则跳过
    try:
        captcha, captcha_id = get_captcha()
        # 增加表数据
        datas['captcha-solution'] = captcha
        datas['captcha-id'] = captcha_id
    except:
        pass
    login_page = session.post(url, data=datas, headers=headers)
    page = login_page.text
    soup = BeautifulSoup(page, "html.parser")
    result = soup.findAll('div', attrs={'class': 'title'})
    #进入豆瓣登陆后页面，打印热门内容
    for item in result:
        print(item.find('a').get_text())
    # 保存 cookies 到文件，
    # 下次可以使用 cookie 直接登录，不需要输入账号和密码
    session.cookies.save()

if __name__ == '__main__':
    if isLogin():
        print('Login successfully')
    else:
        login()
```

运行结果如下：

```
Cookies未能加载
Please input your account:youraccount
Please input your password:yourpassword
please input the captcha:desire

毒舌又自恋的王国维（修改版）
还记得那个横扫围棋界的AI“阿法狗”吗？现在，它输了……
一些建筑（和光）
看电影学穿搭 | 与你共赴一堂两小时的着装补习班
中年串串之歌
对不起，我们最终还是没有变成超级英雄
【我的宝贝1】我的马那面孔&王尔德是什么味道&如何用三个物件具象化麦克白
村官
第ℵ₀课：创新的维度
在冷门季节去北海道
来碗鸡蛋炒饭
九州散策|熊本看城与熊记
从人工智能到后人类：再度更新的AlphaGo没有改变什么？
帮朋友装书架
一栋幸福的房子要长什么样？
关于《老友记》是否为烂剧的争论
柳林风声 插图对比
一年听歌心得（二）
编剧有话说，在下认错，求高抬贵手
大家好，我是N017号童模
```



