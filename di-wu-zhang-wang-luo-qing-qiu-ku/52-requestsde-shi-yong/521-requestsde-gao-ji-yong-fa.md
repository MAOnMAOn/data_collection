## 5.2.2 Requests 的高级用法

之前我们了解了 Requests 的基本用法，如基本的 GET、POST 请求以及 Response 对象的用法，本节我们再来了解下 Requests 的一些高级用法，如文件上传，代理设置，Cookies 设置等等。

### 1. 文件上传
我们知道 Reqeuests 可以模拟提交一些数据，假如有的网站需要我们上传文件，我们同样可以利用它来上传，实现非常简单，实例如下：

```
import requests

files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
```

之前我们下载保存了一个文件叫做 favicon.ico，这次我们用它为例来模拟文件上传的过程。需要注意的是，favicon.ico 这个文件需要和当前脚本在同一目录下。如果有其它文件，当然也可以使用其它文件来上传，更改下名称即可。运行结果如下：

```
{
  "args": {},
  "data": "",
  "files": {
    "file": "data:application/octet-stream;base64,AAAAAA...="
  },
  "form": {},
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Content-Length": "6665",
    "Content-Type": "multipart/form-data; boundary=809f80b1a2974132b133ade1a8e8e058",
    "Host": "httpbin.org",
    "User-Agent": "python-requests/2.10.0"
  },
  "json": null,
  "origin": "60.207.237.16",
  "url": "http://httpbin.org/post"
}
```

以上部分内容省略，这个网站会返回一个 Response，里面包含 files 这个字段，而 form 是空的，这证明文件上传部分会单独有一个 files 字段来标识。

### 2. Cookies
如果使用了 Urllib 处理过 Cookies，写法会比较复杂，而有了 Requests，获取和设置 Cookies 只需要一步即可完成。先用一个实例感受一下获取 Cookies 的过程：

```
import requests

r = requests.get('https://www.baidu.com')
print(r.cookies)
for key, value in r.cookies.items():
    print(key + '=' + value)
```

运行结果如下：

```
<RequestsCookieJar[<Cookie BDORZ=27315 for .baidu.com/>, <Cookie __bsi=13533594356813414194_00_14_N_N_2_0303_C02F_N_N_N_0 for .www.baidu.com/>]>
BDORZ=27315
__bsi=13533594356813414194_00_14_N_N_2_0303_C02F_N_N_N_0
```

首先我们调用了 cookies 属性即可成功得到了 Cookies，可以发现它是一个 RequestCookieJar 类型，然后我们用 items() 方法将其转化为元组组成的列表，遍历输出每一个 Cookie 的名和值，实现 Cookies 的遍历解析。
当然，我们也可以直接用 Cookies 来维持登录状态。
比如我们以知乎为例，直接利用 Cookies 来维持登录状态。
首先登录知乎，将 Headers 中的 Cookies 复制下来，
这里可以替换成你自己的 Cookies，将其设置到 Headers 里面，发送 Request，示例如下：

```
import requests

headers = {
    'Cookie': 'd_c0="AJCC-UdB7AuPThXiS2xJ_9ppCSriyH_tNqc=|1497613365"; _zap=fce3b97f-10a5-4328-8020-6ec0ed75623b; q_c1=743451b141214da3b0b8219fa157b778|1504059314000|1497415542000; q_c1=743451b141214da3b0b8219fa157b778|1504059314000|1497415542000; r_cap_id="MzEwNjZiZGI0Y2M4NDYzYzlmNDg0NGEzMjIzZThlZDI=|1506054080|ed8bf4c5e79d27da188ee919efaf41561b2d826f"; cap_id="ZTZjOGI3MmEyY2NjNDlkZDllY2NmNjMyODhhNWI3Yzk=|1506054080|d7311bdb8630e831ef378b385201c53658b1ef8a"; z_c0=Mi4xVUcwb0FnQUFBQUFBa0lMNVIwSHNDeGNBQUFCaEFsVk54eHpzV1FCcU96MWpucTloaDNTU1RTRFZhbGZKcUgxeXZn|1506054087|1febbeba83a9c07a8caa32d4dda2493ea36e769c; __utma=51854390.551812567.1505796284.1506156285.1506403464.15; __utmz=51854390.1506403464.15.11.utmcsr=zhihu.com|utmccn=(referral)|utmcmd=referral|utmcct=/question/24590883; __utmv=51854390.100-1|2=registration_date=20151005=1^3=entry_date=20151005=1; aliyungf_tc=AQAAAOGIZybTqQcAsnvYc6vrlPqVGGC1; _xsrf=c45e174d-6515-440b-a138-00d60cb6eff2',
    'Host': 'www.zhihu.com',
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36',
}
r = requests.get('https://www.zhihu.com', headers=headers)
print(r.text)
```
发现结果中包含了登录后的结果。
当然也可以通过 cookies 参数来设置，不过这样就需要构造 RequestsCookieJar 对象，而且需要分割一下 Cookies ，相对繁琐，不过效果是相同的。

### 3. 会话维持
在 Requests 中，我们如果直接利用 get() 或 post() 等方法的确可以做到模拟网页的请求。但是这实际上是相当于不同的会话，即不同的 Session，也就是说相当于你用了两个浏览器打开了不同的页面。

设想这样一个场景，我们第一个请求利用了 post() 方法登录了某个网站，第二次想获取成功登录后的自己的个人信息，你又用了一次 get() 方法去请求个人信息页面。实际上，这相当于打开了两个浏览器，是两个完全不相关的会话，所以就不能成功获取个人信息了。

我们当然可以在两次请求的时候都设置好一样的 Cookies ，但这样做起来还是显得很繁琐，我们还有更简单的解决方法。

其实解决这个问题的主要方法就是维持同一个会话，也就是相当于打开一个新的浏览器选项卡而不是新开一个浏览器。但是我又不想每次设置 Cookies，那该怎么办？这时候就可以利用 Session对象维护一个会话，而且不用担心 Cookies 的问题，它会帮我们自动处理好。下面用一个实例来感受一下：

```
import requests

requests.get('http://httpbin.org/cookies/set/number/123456789')
r = requests.get('http://httpbin.org/cookies')
print(r.text)
```

在实例中我们请求了一个测试网址：http://httpbin.org/cookies/set/number/123456789，请求这个网址我们可以设置一个 Cookie，名称叫做 number，内容是 123456789，随后又请求了http://httpbin.org/cookies，此网址可以获取当前的 Cookies。运行结果如下：

```
{
  "cookies": {}
}
```

并不行。我们再用 Session 试试看：

```
import requests

s = requests.Session()
s.get('http://httpbin.org/cookies/set/number/123456789')
r = s.get('http://httpbin.org/cookies')
print(r.text)
```

看下运行结果：

```
{
  "cookies": {
    "number": "123456789"
  }
}
```

此时可以成功获取，所以，利用 Session 我们可以做到模拟同一个会话，而且不用担心 Cookies 的问题，通常用于模拟登录成功之后再进行下一步的操作。
Session 在平常用到的非常广泛，可以用于模拟在一个浏览器中打开同一站点的不同页面。

### 4. SSL证书验证
Requests 提供了证书验证的功能，当发送 HTTP 请求的时候，它会检查 SSL 证书，我们可以使用 verify 参数来控制是否检查此证书，其实如果不加的话默认是 True，会自动验证。

现在我们用 Requests 来测试 12306 的证书一下：

```
import requests

response = requests.get('https://www.12306.cn')
print(response.status_code)
```

运行结果如下：

```
requests.exceptions.SSLError: ("bad handshake: Error([('SSL routines', 'tls_process_server_certificate', 'certificate verify failed')],)",)
```

提示一个错误，叫做 SSLError，证书验证错误。所以如果我们请求一个 HTTPS 站点，但是证书验证错误的页面时，就会报这样的错误，那么如何避免这个错误呢？很简单，把 verify 这个参数设置为 False 即可。改成如下代码：

```
import requests

response = requests.get('https://www.12306.cn', verify=False)
print(response.status_code)
```

这样，就会打印出请求成功的状态码。

```
/home/lv/softwares/anaconda3/lib/python3.6/site-packages/requests/packages/urllib3/connectionpool.py:852: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
<Response [200]>
```

不过发现报了一个警告，它提示建议让我们给它指定证书。
我们可以通过设置忽略警告的方式来屏蔽这个警告：

```
import requests
from requests.packages import urllib3

urllib3.disable_warnings()
response = requests.get('https://www.12306.cn', verify=False)
print(response.status_code)
```

或者通过捕获警告到日志的方式忽略警告：

```
import logging
import requests
logging.captureWarnings(True)
response = requests.get('https://www.12306.cn', verify=False)
print(response.status_code)
```

当然我们也可以指定一个本地证书用作客户端证书，可以是单个文件（包含密钥和证书）或一个包含两个文件路径的元组。
```
import requests

response = requests.get('https://www.12306.cn', cert=('/path/server.crt', '/path/key'))
print(response.status_code)
```
当然上面代码是实例，我们需要有 crt 和 key 文件，指定它们的路径。注意本地私有证书的 key 必须要是解密状态，加密状态的 key 是不支持的。

### 5. 代理设置
对于某些网站，在测试时请求几次，能正常获取内容。但是一旦开始大规模爬取，对于大规模且频繁的请求，网站可能会直接登录验证，验证码，甚至直接封禁IP。

那么为了防止这种情况的发生，我们就需要设置代理来解决这个问题，在 Requests 中需要用到 proxies 这个参数。可以用这样的方式设置：

```
import requests

proxies = {
  'http': 'http://10.10.1.10:3128',
  'https': 'http://10.10.1.10:1080',
}
requests.get('https://www.taobao.com', proxies=proxies)
```

当然直接运行这个实例可能不行，因为这个代理可能是无效的，请换成有效代理试验。

若代理需要使用 HTTP Basic Auth，可以使用类似 http://user:password@host:port 这样的语法来设置代理。实例如下：

```
import requests

proxies = {
    'https': 'http://user:password@10.10.1.10:3128/',
}
requests.get('https://www.taobao.com', proxies=proxies)
```

除了基本的 HTTP 代理，Requests 还支持 SOCKS 协议的代理。首先需要安装 Socks 这个库，命令如下：

`pip3 install "requests[socks]"`

然后就可以使用 SOCKS 协议代理了，实例如下：

```
import requests

proxies = {
    'http': 'socks5://user:password@host:port',
    'https': 'socks5://user:password@host:port'
}
requests.get('https://www.taobao.com', proxies=proxies)
```

### 6. 超时设置
在本机或服务器网络不佳时，我们可能会等待很久才可能会收到响应，甚至因为收不到响应而报错。为了防止服务器不能及时响应，应该设置一个超时时间，即超过了这个时间还没有得到响应，那就报错。设置超时时间需要用到 timeout 参数。这个时间的计算是发出 Request 到服务器返回 Response 的时间。下面用一个实例来感受一下：

```
import requests

r = requests.get('https://www.taobao.com', timeout=1)
print(r.status_code)
```

通过这样的方式，我们可以将超时时间设置为 1 秒，如果 1 秒内没有响应，那就抛出异常。

实际上请求分为两个阶段，即 connect（连接）和 read（读取）。设置的 timeout 值将会用作 connect 和 read 二者的 timeout 总和。如果要分别指定，就可以传入一个元组：

`r = requests.get('https://www.taobao.com', timeout=(5, 11))`

### 7. 身份认证
在访问网站时，我们可能会遇到浏览器弹出认证页面窗口的情况，此时可以使用 Requests 自带的身份认证功能，实例如下：

```
import requests
from requests.auth import HTTPBasicAuth

r = requests.get('http://localhost:5000', auth=HTTPBasicAuth('username', 'password'))
print(r.status_code)
```

如果用户名和密码正确的话，请求时就会自动认证成功，会返回 200 状态码，如果认证失败，则会返回 401 状态码

当然如果参数都传一个 HTTPBasicAuth 类，就显得有点繁琐了，所以 Requests 提供了一个更简单的写法，可以直接传一个元组，它会默认使用 HTTPBasicAuth 这个类来认证。所以上面的代码可以直接简写如下：

```
import requests

r = requests.get('http://localhost:5000', auth=('username', 'password'))
print(r.status_code)
```

Requests 还提供了其他的认证方式，如 OAuth 认证，不过需要安装 oauth 包，命令如下：

`pip3 install requests_oauthlib`

使用 OAuth1 认证的方法如下：

```
import requests
from requests_oauthlib import OAuth1

url = 'https://api.twitter.com/1.1/account/verify_credentials.json'
auth = OAuth1('YOUR_APP_KEY', 'YOUR_APP_SECRET',
              'USER_OAUTH_TOKEN', 'USER_OAUTH_TOKEN_SECRET')
requests.get(url, auth=auth)
```

### 8. Prepared Request
我们可以将请求表示为一个数据结构，其中的各个参数都可以通过一个 request 对象来表示，在 Requests 中，这个数据结构就叫 Prepared Request。我们用一个实例感受一下：

```
from requests import Request, Session

url = 'http://httpbin.org/post'
data = {
    'name': 'MAOn'
}
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36'
}
s = Session()
req = Request('POST', url, data=data, headers=headers)
prepped = s.prepare_request(req)
r = s.send(prepped)
print(r.text)
```

在这里我们引入了 Request，然后用 url、data、headers 参数构造了一个 Request 对象，这时我们需要再调用 Session 的 prepare_request() 方法将其转换为一个 Prepared Request 对象，然后调用 send() 方法发送即可，运行结果如下：

```
{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "name": "MAOn"
  },
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Connection": "close",
    "Content-Length": "9",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36"
  },
  "json": null,
  "origin": "115.216.123.178",
  "url": "http://httpbin.org/post"
}
```

可以看到我们达到了同样的 POST 请求效果。

有了 Request 这个对象，我们就可以将一个个请求当做一个独立的对象来看待，这样在进行队列调度的时候会非常方便。
