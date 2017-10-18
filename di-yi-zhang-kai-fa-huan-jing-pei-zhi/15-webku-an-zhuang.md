## 1.5 Web库的安装

Web 想必我们都不陌生，当下我们日常访问的网站都是 Web 服务程序搭建而成的，Python 同样也有一些这样的 Web 服务程序，比如 Flask、Django 等，我们可以拿它来开发网站，开发接口等等。

在这里，我们主要要用到这些 Web 服务程序来搭建一些 API 接口，供我们的爬虫使用。例如，维护一个代理池，代理保存在 Redis 数据库中，我们要将代理池作为一个公共的组件使用，那么如何构建一个方便的平台来供我们取用这些代理呢？最合适不过的就是通过 Web 服务提供一个 API 接口了。

在这篇文档之中中用到的一些 Web 服务程序主要有 Flask、Tornado 以及 Django, Django 安装配置我们会在另外提及。

### 1、Flask安装
Flask 是一个轻量级的 Web 服务程序，简单、易用、灵活，在这里中我们主要用它来做一些 API 服务，本节我们来了解下它的安装方式。

#### （1） Pip安装
推荐使用 Pip 安装，命令如下：

`pip3 install flask`

运行完毕之后就可以完成安装。

#### （2） 验证安装
安装成功之后可以运行如下实例代码进行测试：

```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

运行代码，可以发现系统会默认在本地 5000 端口开启 Web 服务，控制台输出如下：

` * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

直接访问：http://127.0.0.1:5000/， 可以观察到网页中呈现了 Hello World!

### 2、Tornado的安装
Tornado 是 Python 中一个支持异步的Web框架，通过使用非阻塞 I/O 流，它可以支撑成千上万的开放连接，效率非常高，本节介绍一下它的安装方式。

####（1）Pip安装
推荐使用 Pip 安装，命令如下：

｀pip3 install tornado｀

执行完毕之后即可完成安装。

#### （2） 验证安装
这里我们也可以用一个 hello world 程序进行安装验证

```
In [1]: from tornado.web import RequestHandler,Application

In [2]: from tornado import ioloop

In [3]: class MainHandler(RequestHandler):
   ...:     def get(self):
   ...:         self.write("Hello world")

In [4]: def make_app():
   ...:     return Application([
   ...:         (r"/", MainHandler),
   ...:     ])

In [5]: if __name__ == "__main__":
   ...:     app = make_app()
   ...:     app.listen(address='0.0.0.0',port=8889)
   ...:     ioloop.IOLoop.current().start()
```
直接运行程序，可以发现系统在 8889 端口运行了 Web 服务，控制台没有输出内容，此时访问：http://127.0.0.1:8888/， 可以看到网页中的 Hello, world ,这就说明 tornado 已经安装成功

### 3、 相关链接
#### （1）Flask
GitHub：https://github.com/pallets/flask
官方文档：http://flask.pocoo.org
中文文档：http://docs.jinkan.org/docs/flask
PyPi：https://pypi.python.org/pypi/Flask

#### （2）Tornado
GitHub：https://github.com/tornadoweb/tornado
PyPi：https://pypi.python.org/pypi/tornado
官方文档：http://www.tornadoweb.org