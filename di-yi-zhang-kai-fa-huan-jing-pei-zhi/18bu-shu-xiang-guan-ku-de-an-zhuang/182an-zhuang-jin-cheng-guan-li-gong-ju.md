## 1.8.2 安装进程管理工具

在项目开发之中，进程管理是十分重要的一个环节。进行服务器进程的管理除了使用 `ps -ef | grep "process"` 或者　`kill -9 PID` 还可以使用一些开源管理工具，比如 supervisor。

supervisor 是用 Python 开发的一套通用的进程管理程序，可以将一个普通的命令行进程变为后台 daemon，并监控进程状态，异常退出时可以自动重启。

今天我来介绍一下 centos7 下，supervisor 的安装配置，下面让我们一起来感受一下吧。

### 1. 安装

\(1\) pip 安装  
当前 supervisor 对于 python3 的支持并不友好，如果你的开发环境还是 python2，那你可以直接用 pip 进行安装：

`pip install supervisor`

\(2\) 使用 easy\_install 进行安装  
先前，我们使用 Anaconda 安装 python3 时，仅仅设置了当前普通用户的 python3 开发环境，这里我们可以通过这一点来安装 easy\_install：

```
sudo su  # 切换到 root 用户下
yum install -y python-setuptools.noarch
easy_install-2.7 supervisor
```

### 2. 基础配置

\(1\) 生成默认配置文件  
在 root 用户下，生成默认配置文件

`# echo_supervisord_conf > /etc/supervisord.conf`

我们可以根据需要修改里面的配置。我这里，每个不同的项目，使用了一个单独的配置的文件，我们选择放置在 /etc/supervisor/ 下面，于是修改 /etc/supervisor/supervisord.conf ，内容如下：

```
[unix_http_server]
;file=/tmp/supervisor.sock  ; the path to the socket file
;修改为 /home/supervisor 目录，避免被系统删除
file=/home/supervisor/supervisor.sock   ; the path to the socket file

...

[supervisord]
;logfile=/tmp/supervisord.log ; main log file; default $CWD/supervisord.log
;修改为 /home/supervisor 目录，避免被系统删除
logfile=/home/supervisor/supervisord.log ; main log file; default $CWD/supervisord.log
logfile_maxbytes=50MB        ; max main logfile bytes b4 rotation; default 50MB
logfile_backups=10           ; # of main logfile backups; 0 means none, default 10
loglevel=info                ; log level; default info; others: debug,warn,trace
;pidfile=/tmp/supervisord.pid ; supervisord pidfile; default supervisord.pid
;修改为 /home/supervisor 目录，避免被系统删除
pidfile=/home/supervisor/supervisord.pid ; supervisord pidfile; default supervisord.pid
```

** 启用浏览器管理 **

supervisor 同时提供了通过浏览器来管理进程的方法，只需要注释掉如下几行就可以了。

```
;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for ;all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))
[supervisorctl]
...
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
```

** 使用 include **

在配置文件的最后，有一个 [include] 的配置项，跟 Nginx 一样，可以 include 某个文件夹下的所有配置文件，这样我们就可以为每个进程或相关的几个进程的配置单独写成一个文件。

```
[include]
files = /etc/supervisor/*.conf
```

** 设置权限 **

`sudo chown user /home/supervisor`


现在，我们可以启动 supervisor 了：

`# /usr/bin/supervisord -c /etc/supervisord.conf`

> tips：如果修改了配置文件，可以用kill -HUP重新加载配置文件

`# cat /tmp/supervisord.pid | xargs sudo kill -HUP`

\(2\) 设置开机自启动  
为了能够在机器启动之后自动启动supervisor，需要把supervisor进程配置进systemd。

首先，进入目录 /usr/lib/systemd/system/，增加文件 supervisord.service，来使得机器启动的时候启动supervisor，文件内容：

```
# supervisord service for systemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

激活开机启动命令：

`systemctl enable supervisord.service`

启动supervisor进程:  
`systemctl start supervisord.service`

如果修改了supervisor.service文件，可以通过reload命令来重新加载配置文件：

`systemctl reload supervisord.service`

### 3. 运行supervisor

启动supervisor输入如下命令，使用具体的配置文件执行：

`supervisord -c supervisord.conf`

关闭supervisord需要通过supervisor的控制器：

`supervisorctl -c supervisord.conf shutdown`

重启supervisord也是通过supervisor的控制器：

`supervisorctl -c supervisord.conf reload`

supervisor运行后本身是守护进程，通过自身来管理相应的子进程，通过观察相应的进程状态就很明了：

```
ps -ef | grep supervisord
root     16157     1  0 20:25 ?        00:00:00 /usr/bin/python /usr/bin/supervisord -c /etc/supervisord.conf
```

### 4. 简单示例

使用echo_supervisord_conf命令得到supervisor配置模板,

`echo_supervisord_conf > celery_supervisord.conf`

新行输入如下配置信息（以celery worker为例，具体含义看注释）：

```
[program:celery.worker] 
;指定运行目录 
directory=/home/xxx/webapps/yshblog_app/yshblog
;运行目录下执行命令
command=celery -A yshblog worker --loglevel info --logfile celery_worker.log
 
;启动设置 
numprocs=1          ;进程数
autostart=true      ;当supervisor启动时,程序将会自动启动 
autorestart=true    ;自动重启
 
;停止信号,默认TERM 
;中断:INT (类似于Ctrl+C)(kill -INT pid)，退出后会将写文件或日志(推荐) 
;终止:TERM (kill -TERM pid) 
;挂起:HUP (kill -HUP pid),注意与Ctrl+Z/kill -stop pid不同 
;从容停止:QUIT (kill -QUIT pid) 
stopsignal=INT
```
其中第一行是必须的，设置该程序的名称（可自行修改，不要和其他program重复）。

这里没提到的参数配置不是必须的，可以参考Supervisor的官网。

还需说明日志的问题。原本我设置了日志配置，而不是通过celery命令设置--logfile参数：

```
;输出日志 
stdout_logfile=celery_worker.log 
stdout_logfile_maxbytes=10MB  ;默认最大50M 
stdout_logfile_backups=10     ;日志文件备份数，默认为10 
 
;错误日志 
redirect_stderr=false         ;为true表示禁止监听错误 
stderr_logfile=celery_worker_err.log 
stderr_logfile_maxbytes=10MB 
stderr_logfile_backups=10
```

