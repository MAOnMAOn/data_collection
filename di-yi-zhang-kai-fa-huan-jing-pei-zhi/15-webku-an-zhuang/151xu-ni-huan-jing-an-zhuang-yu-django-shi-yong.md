## 1.5.1 安装 Django 

我们先前简单介绍了 Python 2大常用 Web 库的安装，在这里，我们再简单介绍一下 Django 框架以及 python 虚拟环境的搭建。

Django 是用 Python 写的一个自由和开放源码 web 应用程序框架。可以帮助我们更快、更容易地开发web站点。

Django 也拥有许多强大的开发组件，比如：处理用户认证（注册、登录、登出）的方式、一个管理站点的面板、表单、上传文件的方式，等等。可以让我们无需重新发明轮子就能快速建立新的站点。在后面我们会针对 Scrapy 与 Django 开发爬虫监控的简单框架。

### 1、安装虚拟环境
在安装 Django 之前，我们通常会安装一个虚拟环境 （也称为 virtualenv），相关内容我们会在下文进行讲解。

### 2、Django安装
#### （1） Pip安装
推荐使用 Pip 安装，命令如下：

`pip3 install django`

运行完毕之后就可以完成安装。

#### （2） 验证安装
安装成功之后可以通过运行 Python 解释器进行测试：

`>>> import django`

没有报错，这样就成功安装了 django。

### 3、创建 Django 项目
第一步是创建一个新的 Django 项目。 首先，我们需要运行一些由 Django 提供的脚本，为我们即将开始的项目建立主要骨架。 它会生成一系列的文件夹和文件，在后面的项目中我们会需要修改和使用到它们。

####（1）激活虚拟环境

`source activate mydjangoapp`

####（2）创建工程
这里使用 django 自带的django-admin来创建工程，选择当前目录，在 Linux 系统下，一定要记得输入 最后的那个点

`django-admin startproject mysite .`

####（3）启动工程项目
在控制台中，可以通过运行 python manage.py runserver 开启 web 服务器：

`python manage.py runserver`

现在，我们需要做的就是检测你的站点的服务器是否已经在运行了。打开浏览器（火狐，Chrome，Safari，IE 等）输入这个网址：

`http://127.0.0.1:8000`

当你看到如下内容就证明项目启动成功了：

![](/assets/it_worked2.png)

### 3、 相关链接
GitHub：https://github.com/django/django
官方文档：https://www.djangoproject.com/
中文文档：http://python.usyiyi.cn/translate/django_182/index.html
学习站点：http://code.ziqiangxuetang.com/django/django-tutorial.html