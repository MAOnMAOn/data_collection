## 1.7 Scrapy及部分插件的安装

Scrapy 是一个十分强大的爬虫框架，依赖的库比较多，至少需要依赖库有 Twisted，lxml，pyOpenSSL。而在不同平台环境又各不相同，所以在安装之前最好确保把一些基本库安装好。

### 1、Scrapy 安装

#### （1）Anaconda安装

这种方法是一种比较简单的安装 Scrapy 的方法（尤其是对 Windows 来说），如果你的 Python 是使用 Anaconda 安装的，或者还没有安装 Python 的话，可以使用该方法安装，简单省力，但通过 Anaconda 安装 Scrapy 可能会给后续 Scrapyd 模块的使用带来困扰，所以推荐使用下文中的直接安装方法。

如果已经安装好了 Anaconda，那么可以通过 conda 命令安装 Scrapy，安装命令如下：

`conda install Scrapy`

运行之后便可以完成 Scrapy 的安装。

#### （2）通过 Pip 安装

在 CentOS、RedHat 环境下，我们首先需要确保一些依赖库已经安装，运行如下命令：

```
sudo yum groupinstall -y development tools
sudo yum install -y epel-release libxslt-devel libxml2-devel openssl-devel
```

最后利用 Pip 安装 Scrapy 即可，运行如下命令：

`pip3 install Scrapy`

运行完毕之后即可完成 Scrapy 的安装。

#### （3）常见错误

_**pkg\_resources.VersionConflict: \(six 1.5.2 \(/usr/lib/python3/dist-packages\), Requirement.parse\('six&gt;=1.6.0'\)\)**_

six 包版本过低，six包是一个提供兼容 Python2 和 Python3 的库，升级 six 包即可：

`sudo pip3 install -U six`

_**c/\_cffi\_backend.c:15:17: fatal error: ffi.h: No such file or directory**_

缺少 Libffi 这个库。什么是 libffi？“FFI” 的全名是 Foreign Function Interface，通常指的是允许以一种语言编写的代码调用另一种语言的代码。而 Libffi 库只提供了最底层的、与架构相关的、完整的”FFI”。这里执行下面命令安装相应的库即可：

`sudo yum install gcc libffi-devel python-devel openssl-devel`

_**Command "python setup.py egg\_info" failed with error code 1 in /tmp/pip-build/cryptography/**_

报错，显示缺少加密的相关组件，利用Pip 安装即可。

`pip3 install cryptography`

_**ImportError: No module named '\_cffi\_backend'**_

缺少 cffi 包，使用 Pip 安装即可：

`pip3 install cffi`

### 2、ScrapySplash 的安装部署

ScrapySplash 是一个 Scrapy 中支持 JavaScript 渲染的工具，其安装分为两部分，一个是是 Splash 服务的安装，安装方式是通过 Docker，安装之后会启动一个 Splash 服务，我们可以通过它的接口来实现 JavaScript 页面的加载。另外一个是 ScrapySplash 的 Python 库的安装，安装之后即可在 Scrapy 中使用 Splash 服务。

#### （1）安装Splash

ScrapySplash 会使用 Splash 的 HTTP API 进行页面渲染，所以我们需要安装 Splash 来提供渲染服务，安装是通过 Docker 安装，在这之前请确保已经正确安装好了 Docker。安装命令如下：

`docker run -p 8050:8050 scrapinghub/splash`

这时我们打开：[http://localhost:8050](http://localhost:8050) 即可访问 Splash 的页面。

当然 Splash 也可以直接安装在远程服务器上，我们在服务器上运行以守护态运行 Splash 即可，命令如下：

`docker run -d -p 8050:8050 scrapinghub/splash`

在这里多了一个 -d 参数，它代表将 Docker 容器以守护态运行，这样在中断远程服务器连接后不会终止 Splash 服务的运行。

#### （2） ScrapySplash 安装

成功安装了 Splash 之后，我们接下来再来安装一下其 Python 库，安装命令如下：

`pip3 install scrapy-splash`

命令运行完毕后就会成功安装好此库。

### 3、ScrapyRedis的安装

ScrapyRedis 是 Scrapy 分布式的扩展模块，有了它我们可以方便地实现 Scrapy 分布式爬虫的搭建，本节来介绍一下 ScrapyRedis 的安装方式。

#### （1）Pip安装

推荐使用 Pip 安装，命令如下：

`pip3 install scrapy-redis`

#### （2）Wheel安装

也可以到 PyPi 下载 Wheel 文件安装：[https://pypi.python.org/pypi/scrapy-redis\#downloads](https://pypi.python.org/pypi/scrapy-redis#downloads) ，比如我们选取版本 0.6.8，则可以下载 scrapy\_redis-0.6.8-py2.py3-none-any.whl，然后 Pip 安装即可。

`pip3 install scrapy_redis-0.6.8-py2.py3-none-any.whl`

#### （3）测试安装

安装完成之后，可以在 Python 命令行下测试。

```
$ python3
>>> import scrapy_redis
```

如果没有错误报出，则证明库已经安装好了。

### 4、相关链接

#### （1）Scrapy 文档

官方网站：[https://scrapy.org](https://scrapy.org)  
官方文档：[https://docs.scrapy.org](https://docs.scrapy.org)  
PyPi：   [https://pypi.python.org/pypi/Scrapy](https://pypi.python.org/pypi/Scrapy)  
GitHub： [https://github.com/scrapy/scrapy](https://github.com/scrapy/scrapy)  
中文文档：[http://scrapy-chs.readthedocs.io](http://scrapy-chs.readthedocs.io)

#### （2）ScrapySplash 文档

GitHub：[https://github.com/scrapy-plugins/scrapy-splash](https://github.com/scrapy-plugins/scrapy-splash)  
Splash 官方文档：[http://splash.readthedocs.io](http://splash.readthedocs.io)

#### （3）ScrapyRedis 文档

GitHub：[https://github.com/rmax/scrapy-redis](https://github.com/rmax/scrapy-redis)  
PyPi：[https://pypi.python.org/pypi/scrapy-redis](https://pypi.python.org/pypi/scrapy-redis)  
官方文档：[http://scrapy-redis.readthedocs.io](http://scrapy-redis.readthedocs.io)

