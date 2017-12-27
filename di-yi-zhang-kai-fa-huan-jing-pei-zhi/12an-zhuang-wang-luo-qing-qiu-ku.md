## 1.2 请求库的安装

爬虫可以简单分为几步：抓取页面、分析页面、存储数据。

在抓取页面的过程中，我们需要模拟浏览器向服务器发出请求，所以需要用到一些 Python 库来实现 HTTP 请求操作，在本书中我们用到的第三方库有 Requests、Selenium、Aiotttp 等。在本节我们介绍一下这些请求库的安装方法。

### 1、利用pip进行简单安装requests与selenium
无论是 Windows、Linux 还是 Mac，都可以通过 Pip 这个包管理工具来安装。
在命令行下运行如下命令即可完成 Requests、selenium等相关库的安装：

`pip3 install requests selenium`

这是最简单的安装方式，推荐此种方法安装。

为了验证库是否已经安装成功，可以在命令行下测试一下：

```
$ python3
>>> import requests
>>> import selenium
```

### 2、Aiohttp的安装
上文中的 Requests 库是一个阻塞式的 HTTP 请求库，当我们发出一个请求后，程序会一直等待服务器的响应，直到得到响应后程序才会进行下一步的处理，其实这个过程是比较耗费资源的。如果程序可以在这个等待过程中做一些其他的事情，如进行请求的调度、响应的处理等等，那么爬取效率一定会大大提高。

Aiohttp 就是这样一个提供异步 Web 服务的库，从 Python3.5 版本开始，Python 中加入了 async/await 关键字，使得回调的写法更加直观和人性化，Aiohttp的异步操作借助于 async/await 关键字写法变得更加简洁，架构更加清晰。使用异步请求库来进行数据抓取会大大提高效率，下面我们来看一下这个库的安装方法。

推荐使用 Pip 安装，命令如下：

`pip3 install aiohttp`

另外官方还推荐安装如下两个库，一个是字符编码检测库 cchardet，另一个是加速 DNS 解析库 aiodns，安装命令如下：

`pip3 install cchardet aiodns`

### 3、ChromeDriver的安装
鉴于本人没有在centos7.3之中进行 Chrome 浏览器的下载安装的经验，所以本节主要基于ubuntu16.04的环境进行说明

####（1）安装浏览器
Selenium 库作为一个自动化测试工具，需要浏览器来配合它使用，所以在运行selenium之前我们需要确保安装了相应的浏览器。

首先，配置 yum 源：

`sudo vim /etc/yum.repos.d/google-chrome.repo`

写入：

```
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```

安装浏览器，并加入 --nogpgcheck 参数,避免安装失败或无法更新：

`sudo yum -y install google-chrome-stable --nogpgcheck`

#### （2）查看浏览器版本
 点击 Chrome 的菜单，帮助->关于 Chrome，即可查看 Chrome 的版本号.
这里我的浏览器所显示的版本是 58.0.3029.81 unknown (64-bit) 也就是版本58.0
我们需要记住 Chrome 版本号，在后面选择 ChromeDriver 版本时需要用到。

#### （3）下载浏览器驱动
打开 ChromeDriver 的官方网站，链接为：https://sites.google.com/a/chromium.org/chromedriver/downloads 。 可以看到到目前为止最新版本为 2.32，但其支持的 Chrome 浏览器版本为 59-61，所以我们可以选择 2.30 或者2.31 版本。

找好对应的版本号后随后到 ChromeDriver 镜像站下载对应的安装包即可：https://chromedriver.storage.googleapis.com/index.html ，不同平台可以下载不同的安装包。

#### （4）环境变量配置
在 Linux、Mac 下，需要将可执行文件配置到环境变量或将文件移动到属于环境变量的目录里。

例如移动文件到 /usr/bin 目录，可命令行进入其所在路径，然后将其移动到 /usr/bin：

`sudo mv chromedriver /usr/bin`

当然也可以将 ChromeDriver 配置到 $PATH，首先可以将可执行文件放到某一目录，目录可以任意选择，例如将当前可执行文件放在 /usr/local/chromedriver 目录下，接下来可以修改 ~/.profile 文件，命令如下：

`export PATH="$PATH:/usr/local/chromedriver`

保存然后执行，即可完成环境变量的添加：

`source ~/.profile`

#### （5）验证安装
配置完成后，就可以在命令行下直接执行 chromedriver 命令，如命令行下输入：

`chromedriver`

也可以在程序中测试，执行如下 Python 代码：

```
>>> from selenium import webdriver
>>> browser = webdriver.Chrome()
```

运行之后会弹出一个空白的 Chrome 浏览器，证明所有的配置都没有问题，如果没有弹出，请检查之前的每一步的配置。

若弹出之后闪退，则可能是 ChromeDriver 版本和 Chrome 版本不简容，请更换 ChromeDriver 版本。

#### （6）启动 chromedriver 无界面模式
对于版本高于 60 的 chrome　浏览器，我们可以启用浏览器的无界面模式。

##### 第一种方式
在linux环境下 pip 安装 PyVirtualDisplay,让后 执行如下python代码：

```
In [78]: from pyvirtualdisplay import Display

In [79]: display = Display(visible=0)

In [80]: display.start()
Out[80]: <Display cmd_param=['Xvfb', '-br', '-nolisten', 'tcp', '-screen', '0', '1024x768x24', ':1195'] cmd=['Xvfb', '-br', '-nolisten', 'tcp', '-screen', '0', '1024x768x24', ':1195'] oserror=None return_code=None stdout="None" stderr="None" timeout_happened=False>

In [81]: from selenium import webdriver

In [82]: driver = webdriver.Chrome()

In [83]: driver.get('https:www.baidu.com')

In [84]: display.stop()
Out[84]: <Display cmd_param=['Xvfb', '-br', '-nolisten', 'tcp', '-screen', '0', '1024x768x24', ':1195'] cmd=['Xvfb', '-br', '-nolisten', 'tcp', '-screen', '0', '1024x768x24', ':1195'] oserror=None return_code=0 stdout="" stderr="" timeout_happened=False>
```

如此，便不会Chrome浏览器便会不会弹出。

##### 第二种方式
启用Chrome的headless模式：

```
>>> from selenium import webdriver
>>> options = webdriver.ChromeOptions()
>>> options.add_argument('--headless')
>>> browser = webdriver.Chrome(chrome_options=options)
```

### 3、PhantomJS的安装
#### （1）下载安装
如果我们使用 Chrome 或 Firefox 进行网页抓取的话，每次抓取的时候，都会弹出一个浏览器，比较影响使用。所以在这里再引入无界面浏览器，即 PhantomJS。

Selenium 支持 PhantomJS，而且在简单任务下，其运行效率也是较高的，还支持各种参数配置，使用非常方便，下面我们就来了解一下 PhantomJS 的安装过程。

我们需要在官方网站下载对应的安装包。http://phantomjs.org/download.html， 它支持多种操作系统，Windows、Linux、Mac、FreeBSD等，我们可以选择对应的平台将安装包下载下来。

Linux 下环境变量配置可以参见 ChromeDriver 安装一节，在此不再赘述，关键在于将 PhantomJS 的可执行文件所在路径配置到环境变量里。

#### （2）验证安装

在 Selenium 中使用的话，我们只需要将 Chrome 切换为 PhantomJS 即可。
```
from selenium import webdriver
browser = webdriver.PhantomJS()
browser.get('https://www.baidu.com')
print(browser.current_url)
```
在这里我们访问了百度，然后将当前的 URL 打印出来。控制台输出如下：

`https://www.baidu.com/`

如此一来我们便完成了 PhantomJS 的配置，在后面我们可以利用它来完成一些页面的抓取。

#### （3）PhantomJS相对于其他浏览器(驱动)的不足

1. 不更改请求头的前提下，被识别为爬虫的可能性较大。
2. phantomJS本身在多线程方面还有很多bug，不建议使用多线程提升并发。
3. 主程序退出后，selenium不保证phantomJS也成功退出，最好手动关闭phantomJS。
4. 官方API更新相对较慢。
5. 其他......

### 4、补充
这里我们简要列出部分文档链接作为本节补充。
#### （1）selenium
官方网站：http://www.seleniumhq.org
GitHub：https://github.com/SeleniumHQ/selenium/tree/master/py
PyPi：https://pypi.python.org/pypi/selenium
官方文档：http://selenium-python.readthedocs.io
中文文档：http://selenium-python-zh.readthedocs.io

#### （2）PhantomJS
官方网站：http://phantomjs.org
官方文档：http://phantomjs.org/quick-start.html
下载地址：http://phantomjs.org/download.html
API接口说明：http://phantomjs.org/api/command-line.html

#### （3）Aiohttp
官方文档：http://aiohttp.readthedocs.io/en/stable
GitHub：https://github.com/aio-libs/aiohttp
PyPi：https://pypi.python.org/pypi/aiohttp