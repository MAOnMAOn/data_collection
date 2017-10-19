## 3.3 Aiohttp 的基本使用

虽然 requests 是一个非常优秀的请求库，但其并不支持异步编程，为了进一步提升爬虫工作的效率，这里就有了一个支持异步请求的 aiohttp 库，请安装 记得安装 Python3.5+ 版本。有了 aiohttp，爬虫运行效率会让你两眼一亮。

由于 aiohttp 库在许多方面的用法与 requests 类似，所以对于重复内容，这里只进行简单介绍。

### 1. 准备工作
开始之前请确保已正确安装好 aiohttp 库，如果没有安装可以参考先前的安装说明。

### 2. 实例引入
首先是导入aiohttp模块,然后我们试着获取一个web源码，这里以GitHub的公共Time-line页面为例:

```
import asyncio
import aiohttp

async def aiohttp_test01(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            print(resp.status)
            print(await resp.text())

loop = asyncio.get_event_loop()
tasks = [aiohttp_test01("https://api.github.com/events")]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

上面的代码中，我们创建了一个 ClientSession 对象命名为session，然后通过session的get方法得到一个 ClientResponse 对象，命名为resp，get方法中传入了一个必须的参数url，就是要获得源码的http url。至此便通过协程完成了一个异步IO的get请求。

有get请求当然有post请求，并且post请求也是一个协程：

`session.post('http://httpbin.org/post', data=b'data')`

用法和get是一样，区别是post需要一个额外的参数data，即是需要post的数据。
除了get和post请求外，其他http的操作方法也是一样的：

```
session.put('http://httpbin.org/put', data=b'data')
session.delete('http://httpbin.org/delete')
session.head('http://httpbin.org/get')
session.options('http://httpbin.org/get')
session.patch('http://httpbin.org/patch', data=b'data')
```

### 3. 在URL中传递参数
我们经常需要通过 get 在url中传递一些参数，参数将会作为url问号后面的一部分发给服务器。在aiohttp的请求中，允许以dict的形式来表示问号后的参数。举个例子，如果你想传递 key1=value1 与 key2=value2 到 httpbin.org/get 你可以使用下面的代码：

```
import aiohttp

params = {'key1': 'value1', 'key2': 'value2'}
async with aiohttp.ClientSession() as session:
    async with session.get('http://httpbin.org/get',
                           params=params) as resp:
                           assert resp.url == 'http://httpbin.org/get?key2=value2&key1=value1'
```

可以看到，代码正确的执行了，说明参数被正确的传递了进去。不管是一个参数两个参数，还是更多的参数，都可以通过这种方式来传递。除了这种方式之外，还有另外一个，使用一个 list 来传递（这种方式可以传递一些特殊的参数，例如下面两个key是相等的也可以正确传递）：

```
import aiohttp

params = [('key', 'value1'), ('key', 'value2')]
async with aiohttp.ClientSession() as session:
    async with session.get('http://httpbin.org/get',
                       params=params) as r:
    assert r.url == 'http://httpbin.org/get?key=value2&key=value1'
```

### 4. 响应的内容
还是以GitHub的公共Time-line页面为例，我们可以获得页面响应的内容：

```
import aiohttp

async with aiohttp.ClientSession() as session:
    async with session.get('https://api.github.com/events') as resp:
    print(await resp.text())
    print(await resp.text(encoding='gbk'))
    print(await resp.read())
    print(await resp.json())
```

text(),read()方法是把整个响应体读入内存，如果你是获取大量的数据，请考虑使用”字节流“（streaming response）

```
with open(filename, 'wb') as fd:
    while True:
        chunk = await resp.content.read(chunk_size)
        if not chunk:
            break
        fd.write(chunk)
```

### 5. 自定义请求头
如果你想添加请求头，可以像get添加参数那样以dict的形式，作为get或者post的参数进行请求：

```
import json
import aiohttp

url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}
headers = {'content-type': 'application/json'}

async with aiohttp.ClientSession() as session:
    await session.post(url,
                       data=json.dumps(payload),
                       headers=headers)
```

### 6. 自定义Cookie
给服务器发送cookie，可以通过给 ClientSession 传递一个cookie参数：

```
import aiohttp

url = 'http://httpbin.org/cookies'
cookies = {'cookies_are': 'working'}
async with aiohttp.ClientSession(cookies=cookies) as session:
    async with session.get(url) as resp:
        assert await resp.json() == {
           "cookies": {"cookies_are": "working"}}
```

### 7. post 数据
前面我们简单介绍了通过 post 方法进行简单的表单提交，这里我们再了解一下其他数据提交方式。

***post json***

```
import json
import aiohttp

url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}

async with aiohttp.ClientSession(cookies=cookies) as session:
    async with session.post(url, data=json.dumps(payload)) as resp:
    ...
```

其实json.dumps(payload)返回的也是一个字符串，只不过这个字符串可以被识别为json格式

***post 小文件***

```
import aiohttp

url = 'http://httpbin.org/post'
data = FormData()
data.add_field('file',
               open('report.xls', 'rb'),
               filename='report.xls',
               content_type='application/vnd.ms-excel')

async with aiohttp.ClientSession(cookies=cookies) as session:
    await session.post(url, data=data)
```

把文件对象设置为数据参数，aiohttp将自动以字节流的形式发送给服务器。

### 8. 控制同时连接的数量（连接池）
连接池也可以理解为同时请求的数量，为了限制同时打开的连接数量，我们可以将限制参数传递给连接器：

`conn = aiohttp.TCPConnector(limit=30)`

同时最大进行连接的连接数为30，默认是100，limit=0的时候是无限制。

限制同时打开限制同时打开连接到同一端点的数量（(host, port, is_ssl) 三的倍数），可以通过设置 limit_per_host 参数：

`conn = aiohttp.TCPConnector(limit_per_host=30)`

### 9. 设置代理
aiohttp支持使用代理来访问网页：

```
import aiohttp

async with aiohttp.ClientSession() as session:
    async with session.get("http://python.org",
                           proxy="http://some.proxy.com") as resp:
        print(resp.status)
```

当然也支持需要授权的页面：

```
import aiohttp

async with aiohttp.ClientSession() as session:
    proxy_auth = aiohttp.BasicAuth('user', 'pass')
    async with session.get("http://python.org",
                           proxy="http://some.proxy.com",
                           proxy_auth=proxy_auth) as resp:
        print(resp.status)
```

或者通过这种方式来验证授权：

`session.get("http://python.org", proxy="http://user:pass@some.proxy.com")`


### 10. 超时处理
 默认的IO操作都有5分钟的响应时间 我们可以通过 timeout 进行重写：

```
async with session.get('https://github.com', timeout=60) as r:
    ...
```

如果 timeout=None 或者 timeout=0 将不进行超时检查，也就是不限时长。
