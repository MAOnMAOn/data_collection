## 9.3.2 多 WAN 口路由器拨号

之前我们简单地讲解了内网路由器断线重启的方法，但大家也许会注意到一点：粗暴地重启路由器会引发内网网络通信的中断，不仅仅影响网络程序的运行，也容易对其他用户的工作学习造成不便。

因此，我们必须避免在单 WAN 口路由器，以及正常工作日频繁进行路由器重启。这里我们以多 WAN 口路由器为例，进行 ADSL 拨号。

### 1. 准备工作
在开始之前，请务必阅读上一节内容，对路由器抓包相关内容有所了解。

### 2. 拨号问题记录
进行路由器宽带拨号，需要进行反复的试验。目前出现的问题主要有一下几点：

问题      　　　　　    | 解决方案
----------------------|--------------
ip 更换过快，引发 ip 更换失败|降低 ip 切换频率
部分应用连接中断报错   | 1、降低切换频率；<br/> 2、如果是浏览器，进行刷新、等待与重试;<br/> 3、如果有可能，把数据库应用放在本地，设置 host 为 localhost

### 3. 实例演示
配置文件：

```
# settings.py
import base64


ip = '192.168.1.1'
username = 'your_name'
pwd = 'your_password'

auth = username + ':' + pwd
auth = auth.encode('utf8')
key = base64.b64encode(auth).decode('utf8')

headers = {
    'Host': ip,
    'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:56.0) Gecko/20100101 Firefox/56.0',
    'Accept': '*/*',
    'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding': 'gzip, deflate',
    'Content-Type': 'application/x-www-form-urlencode',
    'X-Requested-With': 'XMLHttpRequest',
    'Referer': 'http://' + ip,
    'Content-Length': '18',
    'Authorization': 'Basic ' + key,
    'Connection': 'keep-alive',
}
```

运行文件：

```
import time
import logging
from urllib import request

import requests

from settings import *

__all__ = ['run']

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
    datefmt='%a, %d %b %Y %H:%M:%S',
    filename='/tmp/adsl.log',
    filemode='w'
)


class Dial(object):

    def __init__(self):
        self.s = requests.session()
        # 请求的拨号接口
        self.url = 'http://192.168.1.1/router/wan_status_set.cgi'

    @staticmethod
    def pause_ip():
        outer_ip = request.urlopen("http://ip.6655.com/ip.aspx").read().decode()
        logging.debug(outer_ip)
        time.sleep(120)

    def wan_operator(self, wan, status_code):
        params = {
            'wanid': wan,
            'statu': status_code,
        }
        post_request = self.s.post(self.url, data=params, headers=headers)
        logging.info(post_request.text)

    def run(self):
        number = 1
        try:
            Dial.pause_ip()
            self.wan_operator(wan='WAN1', status_code=0)
            self.wan_operator(wan='WAN1', status_code=1)
            Dial.pause_ip()
            self.wan_operator(wan='WAN2', status_code=0)
            self.wan_operator(wan='WAN2', status_code=1)
        except Exception as e:
            number = number - 1
            logging.warning(e.args)
            time.sleep(60)
            if number > 0:
                self.run()


if __name__ == '__main__':
    adsl = Dial()
    adsl.run()
```

这里简单演示了双 WAN 口版本，可以通过 crontab 设置夜间运行。当代码执行时间为工作日之时，可以切换为单 WAN 口拨号，并通过执行 shell 脚本进行 ip release。以下为相应的执行脚本。

```
#!/bin/bash
PUBLIC_IP=`wget http://ipecho.net/plain -O - -q ; echo`
echo $PUBLIC_IP
sudo dhclient -r
sudo dhclient
sudo dhclient

ifconfig enp3s0 | grep "inet"
```

