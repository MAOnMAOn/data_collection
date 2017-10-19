## 3.1.1 发送请求

使用 Urllib 的 request 模块我们可以方便地实现 Request 的发送并得到 Response，我们本节来看下它的具体用法

### 1. urlopen()
urllib.request 模块提供了最基本的构造 HTTP 请求的方法，利用它可以模拟浏览器的一个请求发起过程，同时它还带有处理authenticaton（授权验证），redirections（重定向)，cookies（浏览器Cookies）以及其它内容。

我们来感受一下它的强大之处，以 Python 官网为例，我们来把这个网页抓下来：

```
from urllib import request

response = request.urlopen('https://www.python.org')
print(response.read().decode('utf-8'))
```

鉴于打印内容过长，这里就不予显示了。

接下来我们看下它返回的到底是什么，利用 type() 方法输出 Response 的类型。

```
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(type(response))
```

输出结果如下：

`<class 'http.client.HTTPResponse'>`

通过输出结果可以发现它是一个 HTTPResposne 类型的对象，它主要包含的方法有 read()、readinto()、getheader(name)、getheaders()、fileno() 等方法和 msg、version、status、reason、debuglevel、closed 等属性。

得到这个对象之后，我们把它赋值为 response 变量，然后就可以调用这些方法和属性，得到返回结果的一系列信息了。
例如调用 read() 方法可以得到返回的网页内容，调用 status 属性就可以得到返回结果的状态码，如 200 代表请求成功，404 代表网页未找到等。
下面再来一个实例感受一下：

```
from urllib import request

response = request.urlopen('https://www.python.org')
print(response.status)
print(response.getheaders())
print(response.getheader('Server'))
```

运行结果如下：

```
200
[('Server', 'nginx'), ('Content-Type', 'text/html; charset=utf-8'), ('X-Frame-Options', 'SAMEORIGIN'), ('X-Clacks-Overhead', 'GNU Terry Pratchett'), ('Content-Length', '47397'), ('Accept-Ranges', 'bytes'), ('Date', 'Mon, 01 Aug 2016 09:57:31 GMT'), ('Via', '1.1 varnish'), ('Age', '2473'), ('Connection', 'close'), ('X-Served-By', 'cache-lcy1125-LCY'), ('X-Cache', 'HIT'), ('X-Cache-Hits', '23'), ('Vary', 'Cookie'), ('Strict-Transport-Security', 'max-age=63072000; includeSubDomains')]
nginx
```

可见，三个输出分别输出了响应的状态码，响应的头信息，以及通过调用 getheader() 方法并传递一个参数 Server 获取了 headers 中的 Server 值，结果是 nginx，意思就是服务器是 nginx 搭建的。利用最基本的 urlopen() 方法，可以完成最基本的简单网页的 GET 请求抓取。

如果我们想给链接传递一些参数该怎么实现呢？我们首先看一下 urlopen() 函数的API：

```
urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
```

可以发现除了第一个参数可以传递 URL 之外，还可以传递其它的内容，比如 data（附加数据）、timeout（超时时间）等等。下面我们看一下这几个参数的用法。

**data参数**

data 参数是可选的，如果要添加 data，它要是字节流编码格式的内容，即 bytes 类型，通过 bytes() 方法可以进行转化，另外如果传递了这个 data 参数，它的请求方式就不再是 GET 方式请求，而是 POST。下面用一个实例来感受一下：

```
from urllib import parse       
from urllib import request

data = bytes(parse.urlencode({'word': 'hello'}), encoding='utf8')
response = request.urlopen('http://httpbin.org/post', data=data)
print(response.read())
```

在这里我们传递了一个参数 word，值是 hello。它需要被转码成bytes（字节流）类型。其中转字节流采用了 bytes() 方法，第一个参数需要是 str（字符串）类型，需要用 urllib.parse 模块里的 urlencode() 方法来将参数字典转化为字符串。第二个参数指定编码格式，在这里指定为 utf8。

在这里请求的站点是 httpbin.org，它可以提供 HTTP 请求测试，本次我们请求的 URL 为：http://httpbin.org/post ，这个链接可以用来测试 POST 请求，它可以输出 Request 的一些信息，其中就包含我们传递的 data 参数。运行结果如下：

```
b'{\n  "args": {}, \n  "data": "", \n  "files": {}, \n  "form": {\n    "word": "hello"\n  }, \n  "headers": {\n    "Accept-Encoding": "identity", \n    "Connection": "close", \n    "Content-Length": "10", \n    "Content-Type": "application/x-www-form-urlencoded", \n    "Host": "httpbin.org", \n    "User-Agent": "Python-urllib/3.6"\n  }, \n  "json": null, \n  "origin": "115.216.127.56", \n  "url": "http://httpbin.org/post"\n}\n'
```

我们传递的参数出现在了 form 字段中，这表明是模拟了表单提交的方式，以 POST 方式传输数据。

**timeout参数**

timeout 参数可以设置超时时间，单位为秒，意思就是如果请求超出了设置的这个时间还没有得到响应，就会抛出异常，如果不指定，就会使用全局默认时间。它支持 HTTP、HTTPS、FTP 请求。下面来用一个实例感受一下：

```
from urllib import request
response = request.urlopen('http://httpbin.org/get', timeout=1)
print(response.read())
```

运行结果如下：

```
During handling of the above exception, another exception occurred:

Traceback (most recent call last): File "/var/py/python/urllibtest.py", line 4, in <module> response = urllib.request.urlopen('http://httpbin.org/get', timeout=1)
...
urllib.error.URLError: <urlopen error timed out>
```

在这里我们设置了超时时间是 1 秒，程序 1 秒过后服务器依然没有响应，于是抛出了 URLError 异常，它属于 urllib.error 模块，错误原因是超时。
因此我们可以通过设置这个超时时间来控制一个网页如果长时间未响应就跳过它的抓取，利用 try except 语句就可以实现这样的操作，代码如下：

```
import socket
from urllib import request
from urllib import error

try:
    response = request.urlopen('http://httpbin.org/get', timeout=0.1)
except error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

在这里我们请求了 http://httpbin.org/get 这个测试链接，设置了超时为 0.1 秒，捕获了 URLError 这个异常，然后判断异常原因是 socket.timeout 类型，就得出它确实是因为超时而报错，打印输出了 TIME OUT。运行结果如下：

`TIME OUT`

常理来说，0.1 秒内基本不可能得到服务器响应，因此输出了 TIME OUT 的提示。
这样，我们可以通过设置 timeout 这个参数来实现超时处理，有时还是很有用的。

**其他参数**

还有 context 参数，它必须是 ssl.SSLContext 类型，用来指定 SSL 设置。
cafile 和 capath 两个参数是指定 CA 证书和它的路径，这个在请求 HTTPS 链接时会有用。cadefault 参数现在已经弃用了，默认为 False。

以上讲解了 urlopen() 方法的用法，通过这个最基本的函数可以完成简单的请求和网页抓取。

### 2. Request
由上我们知道利用 urlopen() 方法可以实现最基本请求的发起，但这几个简单的参数并不足以构建一个完整的请求，如果请求中需要加入 Headers 等信息，我们就可以利用更强大的 Request 类来构建一个请求。首先我们用一个实例来感受一下 Request 的用法：

```
from urllib import request

request = request.Request('https://python.org')
response = request.urlopen(request)
print(response.read().decode('utf-8'))
```

可以发现，我们依然是用 urlopen() 方法来发送这个请求，只不过这次 urlopen() 方法的参数不再是一个 URL，而是一个 Request 类型的对象，通过构造这个这个数据结构，一方面我们可以将请求独立成一个对象，另一方面可配置参数更加丰富和灵活。

下面我们看一下 Request 都可以通过怎样的参数来构造，它的构造方法如下：

`class urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)`

第一个 url 参数是请求 URL，这个是必传参数，其他的都是可选参数。

第二个 data 参数如果要传必须传 bytes（字节流）类型的，如果是一个字典，可以先用 urllib.parse 模块里的 urlencode() 编码。

第三个 headers 参数是一个字典，这个就是 Request Headers 了，你可以在构造 Request 时通过 headers 参数直接构造，也可以通过调用 Request 实例的 add_header() 方法来添加。

添加 Request Headers 最常用的用法就是通过修改 User-Agent 来伪装浏览器，默认的 User-Agent 是 Python-urllib，我们可以通过修改它来伪装浏览器，比如要伪装火狐浏览器，你可以把它设置为：

`Mozilla/5.0 (X11; U; Linux i686) Gecko/20071127 Firefox/2.0.0.11`

第四个 origin_req_host 参数指的是请求方的 host 名称或者 IP 地址。

第五个 unverifiable 参数指的是这个请求是否是无法验证的，默认是False。意思就是说用户没有足够权限来选择接收这个请求的结果。例如我们请求一个 HTML 文档中的图片，但是我们没有自动抓取图像的权限，这时 unverifiable 的值就是 True。

第六个 method 参数是一个字符串，它用来指示请求使用的方法，比如GET，POST，PUT等等。

下面我们传入多个参数构建一个 Request 来感受一下：

```
from urllib import request, parse

url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host': 'httpbin.org'
}
dict = {'name': 'MAOn'}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

在这里我们通过四个参数构造了一个 Request，url 即请求 URL，在headers 中指定了 User-Agent 和 Host，传递的参数 data 用了 urlencode() 和 bytes() 方法来转成字节流，另外指定了请求方式为 POST。运行结果如下：

```
{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "name": "MAOn"
  },
  "headers": {
    "Accept-Encoding": "identity",
    "Connection": "close",
    "Content-Length": "9",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)"
  },
  "json": null,
  "origin": "115.196.7.254",
  "url": "http://httpbin.org/post"
}
```

通过观察结果可以发现，我们成功设置了 data，headers 以及 method。
另外 headers 也可以用 add_header() 方法来添加。

```
req = request.Request(url=url, data=data, method='POST')
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
```

如此一来，我们就可以更加方便地构造一个 Request，实现请求的发送啦。

### 3. 高级用法
在上面的过程中，虽然可以构造 Request，但是一些更高级的操作，比如 Cookies 处理，代理设置等操作却无法进行，所以更强大的工具 Handler 便登场了。

可以把它理解为各种处理器，有专门处理登录验证的，有处理 Cookies 的，有处理代理设置的，利用它们我们几乎可以做到任何 HTTP 请求中所有的事情。
首先介绍下 urllib.request 模块里的 BaseHandler类，它是所有其他 Handler 的父类，它提供了最基本的 Handler 的方法，例如 default_open()、protocol_request() 方法等。

接下来就有各种 Handler 子类继承这个 BaseHandler 类，举例几个如下：
HTTPDefaultErrorHandler 用于处理 HTTP 响应错误，错误都会抛出 HTTPError 类型的异常。

 - HTTPRedirectHandler 用于处理重定向。
 - HTTPCookieProcessor 用于处理 Cookies。
 - ProxyHandler 用于设置代理，默认代理为空。
 - HTTPPasswordMgr 用于管理密码，它维护了用户名密码的表。
 - HTTPBasicAuthHandler 用于管理认证，如果一个链接打开时需要认证，那么可以用它来解决认证问题。

另外一个比较重要的类就是 OpenerDirector，也称 Opener，我们之前用过 urlopen() 这个方法，实际上它就是 Urllib为我们提供的一个 Opener。Opener 可以使用 open() 方法，返回的类型和 urlopen() 如出一辙。那么它和 Handler 有什么关系？简而言之，就是利用 Handler 来构建 Opener。
下面我们用几个实例来感受一下他们的用法：

***认证***

有些网站在打开时它就弹出了一个框，直接提示你输入用户名和密码，认证成功之后才能查看页面，如图所示：

![](/assets/身份验证图.png)

要请求这样的页面，借助于 HTTPBasicAuthHandler 就可以完成，代码如下：

```
from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener
from urllib.error import URLError

username = 'username'
password = 'password'
url = 'http://192.168.1.1'

p = HTTPPasswordMgrWithDefaultRealm()
p.add_password(None, url, username, password)
auth_handler = HTTPBasicAuthHandler(p)
opener = build_opener(auth_handler)

try:
    result = opener.open(url)
    html = result.read().decode('utf-8')
    print(html)
except URLError as e:
    print(e.reason)
```

在这里，首先实例化了一个 HTTPBasicAuthHandler 对象，参数是 HTTPPasswordMgrWithDefaultRealm 对象，它利用 add_password() 添加进去用户名和密码，这样我们就建立了一个处理认证的 Handler。

接下来利用 build_opener() 方法来利用这个 Handler 构建一个 Opener，那么这个 Opener 在发送请求的时候就相当于已经认证成功了。最后利用 Opener 的 open() 方法打开链接，就可以完成认证了，在这里获取到的结果就是认证后的页面源码内容。

***代理***

在做爬虫的时候免不了要使用代理，如果要添加代理，可以这样做：

```
from urllib.error import URLError
from urllib.request import ProxyHandler, build_opener

proxy_handler = ProxyHandler({
    'http': 'http://127.0.0.1:9743',
    'https': 'https://127.0.0.1:9743'
})
opener = build_opener(proxy_handler)
try:
    response = opener.open('https://www.baidu.com')
    print(response.read().decode('utf-8'))
except URLError as e:
    print(e.reason)
```

在此本地搭建了一个代理，运行在 9743 端口上。
在这里使用了 ProxyHandler，ProxyHandler 的参数是一个字典，键名是协议类型，比如 HTTP 还是 HTTPS 等，键值是代理链接，可以添加多个代理。然后利用 build_opener() 方法利用这个 Handler 构造一个 Opener，然后发送请求即可。

***Cookies***

Cookies 的处理就需要 Cookies 相关的 Handler 了。先用一个实例来感受一下怎样将网站的 Cookies 获取下来，代码如下：

```
import http.cookiejar, urllib.request

cookie = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('https://www.baidu.com')
for item in cookie:
    print(item.name+"="+item.value)
```

首先我们必须声明一个 CookieJar 对象，接下来我们就需要利用 HTTPCookieProcessor 来构建一个 Handler，最后利用 build_opener() 方法构建出 Opener，执行 open() 函数即可。运行结果如下：

```
BIDUPSID=B61BCB3EEE5CC0FB4349ADDFD982E37B
PSTM=1507528930
BD_NOT_HTTPS=1
```

可以看到输出了每一条 Cookie 的名称还有值。我们知道 Cookies 实际也是以文本形式保存的，也可以把 cookies 存入文本，我们用下面的实例来感受一下：

```
filename = 'cookies.txt'
cookie = http.cookiejar.MozillaCookieJar(filename)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
cookie.save(ignore_discard=True, ignore_expires=True)
```

这时的 CookieJar就需要换成 MozillaCookieJar，生成文件时需要用到它，它是 CookieJar 的子类，可以用来处理 Cookies 和文件相关的事件，读取和保存 Cookies，它可以将 Cookies 保存成 Mozilla 型浏览器的 Cookies 的格式。
运行之后可以发现生成了一个 cookies.txt 文件。

另外还有一个 LWPCookieJar，同样可以读取和保存 Cookies，但是保存的格式和 MozillaCookieJar 的不一样，它会保存成与 libwww-perl(LWP) 的 Cookies 文件格式。要保存成 LWP 格式的 Cookies 文件，可以在声明时就改为：

`cookie = http.cookiejar.LWPCookieJar(filename)`

由此看来生成的格式还是有比较大的差异的。

那么生成了 Cookies 文件，怎样从文件读取并利用呢？下面我们以 LWPCookieJar 格式为例来感受一下：

```
cookie = http.cookiejar.LWPCookieJar()
cookie.load('cookies.txt', ignore_discard=True, ignore_expires=True)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
print(response.read().decode('utf-8'))
```

可以看到我们这里调用了 load() 方法来读取本地的 Coookis 文件，获取到了 Cookies 的内容。不过前提是我们首先利用生成了 LWPCookieJar 格式的 Cookies，获取到 Cookies 之后，后面同样的方法构建 Handler 和 Opener 即可。

运行结果正常输出百度网页的源代码。