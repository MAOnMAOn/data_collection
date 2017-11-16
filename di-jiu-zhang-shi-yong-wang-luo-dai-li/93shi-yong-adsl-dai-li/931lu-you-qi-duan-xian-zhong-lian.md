## 9.3.1 路由器断线重启

我们在之前的反爬虫策略之中，曾经讲到通过重启路由器的方式，来生成动态 ip ，以规避网站的反爬虫行为，今天我们就通过模拟路由器登录与重启来实现动态 ip 生成的功能，好了，现在让我们一起来感受一下吧。

### 1. 登录路由器
首先，我们需要打开火狐浏览器浏览器，输入路由器默认地址 192.168.1.1 并输入设置好的路由器用户名与密码(这里建议大家可以学习一下密码原则),切记不要使用默认的用户名、密码，切记！不要使用简单的用户名密码，最好关闭远程访问，特别是没有防火墙功能的路由器更需要对黑客的暴力破解心存警惕！！

现在我们进入路由器首页，路由器版本有点老，不过一切运行还算正常：

![](/assets/lyqmnidl1.png)

现在我们运行如下代码以查看当前主机的公网 ip：

```
from urllib import request

ip = request.urlopen("http://ip.6655.com/ip.aspx").read().decode()
print(ip)
```

结果如下：

`125.119.123.0`

打开火狐浏览器开发者工具，并开启调试模式，切记一定要勾选-----开启持续日志的选项，否则，登录以后链接可能就会跳转消失了：

![](/assets/调试模式356.png)

接着，选择网络之中的 HTML 与 XHR 选项，如此，页面内容会少很多，而且关键的东西也不会缺失。

![](/assets/路由器登录需要修改图片28.png)

这里我们经过分析选中了第一个有效的 POST 请求链接，划去的地方就是我们用户名密码的 base64 编码，这里是一个非常简单的登录，虽然有些不通用，但思路还是类似的。

这里我们查看一下表单数据，发现一条表单数据，把它记下！

![](/assets/biaodanshuju.png)

### 2. 实例演示

```
import base64

import requests


ip = '192.168.1.1'
username = 'account-name'
pwd = 'account-password'

auth = username + ':' + pwd
auth = auth.encode('utf8')
key = base64.b64encode(auth).decode('utf8')

headers = {
    'Host': ip,
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36',
    'Accept': '*/*',
    'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding': 'gzip, deflate',
    'Content-Type': 'application/x-www-form-urlencode',
    'Referer': 'http://' + ip,
    'Content-Length': '20',
    'Authorization': 'Basic ' + key,
    'Connection': 'keep-alive',
    'Cache-Control': 'max-page=0'
}
# login = {
#     'REPORT_METHOD':'xml',
#     'ACTION':'login_plaintext',
#     'USER':username,
#     'PASSWD':pwd,
#     'CAPTCHA':''
# }

login ={
    'noneed':'1508932198395'
}

s = requests.session()
login = s.post('http://'+ip+'/router/get_user_type.cgi', data=login, headers=headers)
print(login.text)
```

这里我们直接复制了请求头，作为 requests 中的 header ，注意到请求头之中有一个 'Authorization'：里面正是我们需要传入的 base64　编码内容，通过请求头与表单数据来提交请求，我们轻松地实现了路由器的访问。结果如下：

`{"user_type":"1"}`

以此类推，模拟路由器重启也是类似的道理：

```
# 这里省去部分内容

headers = {
    'Host': ip,
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36',
    'Accept': '*/*',
    'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding': 'gzip, deflate',
    'Content-Type': 'application/x-www-form-urlencode',
    'X-Requested-With': 'XMLHttpRequest',
    'Referer':'http://' + ip,
    'Content-Length': '13',
    'Authorization': 'Basic ' + key,
    'Connection': 'keep-alive',
}

reboot = {
    'noneed':'noneed'
}

reboot=s.post('http://'+ip+'/router/reboot.cgi',data=reboot,headers=headers)
print(reboot.text)
```

运行结果如下：

`["SUCCESS"]`

最后我们重新测试一下公网 ip 进行验证：

```
from urllib import request

ip = request.urlopen("http://ip.6655.com/ip.aspx").read().decode()
print(ip)
```

结果如下：

`183.159.208.241`

如此，我们就获得了新的公网 ip！



