## 1.8.1 Scrapyd及其拓展的安装使用

### 1、Scrapyd安装配置
Scrapyd 是一个用于部署和运行 Scrapy 项目的工具。有了它，你可以将写好的 Scrapy 项目上传到云主机并通过 API 来控制它的运行。

#### (1) Pip安装
推荐使用 Pip 安装，命令如下：

`pip3 install scrapyd`

#### (2) 配置
安装完毕之后需要新建一个配置文件 /etc/scrapyd/scrapyd.conf，Scrapyd 在运行的时候会读取此配置文件。
在 Scrapyd 1.2 版本后不会自动创建该文件，需要自行添加。执行命令新建文件：

```
sudo mkdir /etc/scrapyd
sudo vi /etc/scrapyd/scrapyd.conf
```

写入如下内容：

```
[scrapyd]
eggs_dir    = eggs
logs_dir    = logs
items_dir   =
jobs_to_keep = 5
dbs_dir     = dbs
max_proc    = 0
max_proc_per_cpu = 10
finished_to_keep = 100
poll_interval = 5.0
bind_address = 0.0.0.0
http_port   = 6800
debug       = off
runner      = scrapyd.runner
application = scrapyd.app.application
launcher    = scrapyd.launcher.Launcher
webroot     = scrapyd.website.Root

[services]
schedule.json     = scrapyd.webservice.Schedule
cancel.json       = scrapyd.webservice.Cancel
addversion.json   = scrapyd.webservice.AddVersion
listprojects.json = scrapyd.webservice.ListProjects
listversions.json = scrapyd.webservice.ListVersions
listspiders.json  = scrapyd.webservice.ListSpiders
delproject.json   = scrapyd.webservice.DeleteProject
delversion.json   = scrapyd.webservice.DeleteVersion
listjobs.json     = scrapyd.webservice.ListJobs
daemonstatus.json = scrapyd.webservice.DaemonStatus
```

配置文件的内容可以参见官方文档：https://scrapyd.readthedocs.io/en/stable/config.html#example-configuration-file ，在这里的配置文件有所修改，其中之一是 max_proc_per_cpu 官方默认为 4，即一台主机每个 CPU 最多运行 4 个Scrapy Job，在此提高为 10，另外一个是 bind_address，默认为本地 127.0.0.1，在此修改为 0.0.0.0，以使外网可以访问。

#### (3) 后台运行
由于 Scrapyd 是一个纯 Python 项目，在这里可以直接调用 scrapyd 来运行，为了使程序一直在后台运行，可以使用如下命令：

`(scrapyd > /dev/null &)`

这样 Scrapyd 就会在后台持续运行了，控制台输出直接忽略，当然如果想记录输出日志可以修改输出目标，如：

`(scrapyd > ~/scrapyd.log &)`

则会输出 Scrapyd 运行输出到 ~/scrapyd.log 文件中。运行之后便可以在浏览器的 6800 访问 WebUI 了，可以简略看到当前 Scrapyd 的运行 Job、Log 等内容

当然运行 Scrapyd 更佳的方式是使用 Supervisor ，另外 Scrapyd 也支持 Docker。

### 2、ScrapydClient安装
在将 Scrapy 代码部署到远程 Scrapyd 的时候，其第一步就是要将代码打包为 Egg 文件，其次需要将 Egg 文件上传到远程主机，这个过程若用程序来实现是完全可以的，但是我们并不需要做这些工作，因为 ScrapydClient 已经为我们实现了这些功能。

#### (1) Pip安装
推荐使用 Pip 安装，命令如下：

`pip3 install scrapyd-client`

#### (2) 验证安装
安装成功后会有一个可用命令，叫做 scrapyd-deploy，即部署命令。
我们可以输入如下测试命令测试 ScrapydClient 是否安装成功：

`scrapyd-deploy -h`

如果出现如下类似输出则证明 ScrapydClient 已经成功安装:

```
Usage: scrapyd-deploy [options] [ [target] | -l | -L <target> ]
......
```
### 3、ScrapydAPI的安装
安装好 Scrapyd 之后，我们可以直接请求它提供的 API 即可获取当前主机的 Scrapy 任务运行状况。如某台主机的 IP 为 192.168.1.1，则可以直接运行如下命令获取当前主机的所有 Scrapy 项目：

`curl http://localhost:6800/listprojects.json`

运行结果：

`{"status": "ok", "projects": ["myproject", "otherproject"]}`

返回结果是 Json 字符串，通过解析这个字符串我们便可以得到当前主机所有项目。
但是用这种方式来获取任务状态还是有点繁琐，所以 ScrapydAPI 就为它做了一层封装，下面我们来看下它的安装方式。

#### (1) Pip安装
推荐使用 Pip 安装，命令如下：

`pip install python-scrapyd-api`

#### (2) 验证安装
安装完成之后便可以使用 Python 来获取主机状态了，所以如上的操作便可以用代码实现：

```
from scrapyd_api import ScrapydAPI
scrapyd = ScrapydAPI('http://localhost:6800')
print(scrapyd.list_projects())
```

运行结果：

`["myproject", "otherproject"]`

这样我们便可以用 Python 直接来获取各个主机上 Scrapy 任务的运行状态了。

### 4、相关链接
#### （1）Scrapyd
GitHub：https://github.com/scrapy/scrapyd
PyPi：https://pypi.python.org/pypi/scrapyd
官方文档：https://scrapyd.readthedocs.io

#### （2）ScrapydClient
GitHub：https://github.com/scrapy/scrapyd-client
PyPi：https://pypi.python.org/pypi/scrapyd-client
使用说明：https://github.com/scrapy/scrapyd-client#scrapyd-deploy

#### （3）ScrapydAPI
官方文档：http://python-scrapyd-api.readthedocs.io/en/latest/usage.html