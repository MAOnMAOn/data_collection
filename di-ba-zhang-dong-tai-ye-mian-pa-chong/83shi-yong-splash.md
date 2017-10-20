## 7.2 使用Splash

Splash 是一个 JavaScript 渲染服务，是一个带有 HTTP API 的轻量级浏览器，同时它对接了 Python 中的 Twisted和 QT 库，利用它我们同样可以实现动态渲染页面的抓取。

利用 Splash 我们可以实现如下功能：

 1. 异步方式处理多个网页渲染过程
 2. 获取渲染后的页面的源代码或截图
 3. 通过关闭图片渲染或者使用 Adblock 规则来加快页面渲染速度
 4. 可执行特定的 JavaScript 脚本
 5. 可通过 Lua 脚本来控制页面渲染过程
 6. 获取渲染的详细过程并通过 HAR（HTTP Archive）格式呈现

### 1. 准备工作
请确保已经正确安装好了 Splash 并可以正常运行服务，如没有安装可以参考先前的安装说明。

### 2. 引入实例
运行如下命令，即可通过 Splash 提供的 Web 页面来测试 Splash的渲染过程：

`docker run -p 8050:8050 scrapinghub/splash`

然后打开：http://localhost:8050/ 即可看到其 Web 页面。

在网页右侧呈现的是一个渲染示例，我们可以看到在上方有一个输入框，默认是：http://google.com ，我们在这里换成百度测试一下，将内容更改为：https://www.baidu.com ，然后点击按钮，开始渲染。

然后我们能看到网页的返回结果呈现了渲染截图、HAR 加载统计数据、网页的源代码。
通过 HAR 的结果我们可以看到 Splash 执行了整个网页的渲染过程，包括 CSS、JavaScript 的加载等过程，呈现的页面和我们在浏览器得到的结果完全一致。
那么这个过程是由什么来控制的呢？我们重新返回首页可以看到实际上是有一段脚本，内容如下：

```
function main(splash, args)
  assert(splash:go(args.url))
  assert(splash:wait(0.5))
  return {
    html = splash:html(),
    png = splash:png(),
    har = splash:har(),
  }
end
```

这个脚本是实际是 Lua 语言写的脚本，Lua也是一门编程语言，简洁易用。
即使我们不懂这个语言的语法，但通过脚本表面意思我们也可以大致了解到它是首先调用 go() 方法去加载页面，然后调用 wait() 方法等待了一定的时间，最后返回了页面的源码、截图和 HAR 信息。
所以到这里我们可以大体了解到 Splash 是通过 Lua 脚本来控制了页面的加载过程，加载过程完全模拟浏览器，最后可返回各种格式的结果，如网页源码、截图等。

所以接下来我们要学会用 Splash 的话，一是需要了解其中 Lua 脚本的写法，二是需要了解相关 API 的用法。

### 3. Splash API调用
关于 Splash Lua 脚本的用法，本人没有相关经验，故不好意思班门弄斧了，现在考虑:如何才能利用Splash 来渲染页面呢？我们怎样才能和 Python 程序结合使用并抓取 JavaScript 渲染的页面呢？

其实 Splash 给我们提供了一些 HTTP API 接口，我们只需要请求这些接口并传递相应的参数即可获取页面渲染后的结果，下面我们对这些接口进行了解：

***render.html***

此接口用于获取 JavaScript 渲染的页面的 HTML 代码，接口地址就是 Splash 的运行地址加此接口名称，例如：http://localhost:8050/render.html ，可以用 curl 来测试一下：

`curl http://localhost:8050/render.html?url=https://www.baidu.com`

我们给此接口传递了一个 url 参数指定渲染的 URL，返回结果即页面渲染后的源代码。

如果用 Python 实现的话，代码如下：

```
import requests
url = 'http://localhost:8050/render.html?url=https://www.baidu.com'
response = requests.get(url)
print(response.text)
```

这样我们就可以成功输出百度页面渲染后的源代码了。

另外此接口还可以指定其他参数，如 wait 指定等待秒数，如果我们要确保页面完全加载出来可以增加等待时间，例如：

```
import requests
url = 'http://localhost:8050/render.html?url=https://www.taobao.com&wait=5'
response = requests.get(url)
print(response.text)
```

如果增加了此等待时间后，得到响应的时间就会相应变长，如在这里我们会等待大约 5 秒多钟即可获取 JavaScript 渲染后的淘宝页面源代码。
另外此接口还支持代理设置、图片加载设置、Headers设置、请求方法设置，具体的用法可以参见官方文档：https://splash.readthedocs.io/en/stable/api.html#render-html。

***render.png***

此接口可以获取网页截图，参数相比 render.html 又多了几个，如 width、height 来控制宽高，返回的是 PNG 格式的图片二进制数据。
示例如下：

`curl http://localhost:8050/render.png?url=https://www.taobao.com&wait=5&width=1000&height=700`

在这里我们还传入了 width 和 height 来放缩页面大小为 1000x700 像素。
如果用 Python 实现，我们可以将返回的二进制数据保存为PNG 格式的图片，实现如下：

```
import requests

url = 'http://localhost:8050/render.png?url=https://www.jd.com&wait=5&width=1000&height=700'
response = requests.get(url)
with open('taobao.png', 'wb') as f:
    f.write(response.content)
```

这样我们就成功获取了京东首页渲染完成后的页面截图，详细的参数设置可以参考官网文档：https://splash.readthedocs.io/en/stable/api.html#render-png。

***render.jpeg***

此接口和 render.png 类似，不过它返回的是 JPEG 格式的图片二进制数据。
另外此接口相比 render.png 还多了一个参数 quality，可以用来设置图片质量。

***render.har***

此接口用于获取页面加载的 HAR 数据，示例如下：

`curl http://localhost:8050/render.har?url=https://www.jd.com&wait=5`

返回结果非常多，是一个 Json 格式的数据，里面包含了页面加载过程中的 HAR 数据。

***render.json***

此接口包含了前面接口的所有功能，返回结果是 Json 格式，示例如下：

`curl http://localhost:8050/render.json?url=https://httpbin.org`

结果如下：

`{"title": "httpbin(1): HTTP Client Testing Service", "url": "https://httpbin.org/", "requestedUrl": "https://httpbin.org/", "geometry": [0, 0, 1024, 768]}`

我们可以通过传入不同的参数控制其返回的结果，如传入html=1，返回结果即会增加源代码数据，传入 png=1，返回结果机会增加页面 PNG 截图数据，传入har=1则会获得页面 HAR 数据，例如：

`curl http://localhost:8050/render.json?url=https://httpbin.org&html=1&har=1`

这样返回的 Json 结果便会包含网页源代码和 HAR 数据。更多参数设置可以参考官方文档：https://splash.readthedocs.io/en/stable/api.html#render-json。

***execute***

此接口才是最为强大的接口，用此接口便可以实现和 Lua 脚本的对接。
前面的 render.html、render.png 等接口对于一般的 JavaScript 渲染页面是足够了，但是如果要实现一些交互操作的话还是无能为力的，所以这里就需要使用此 execute 接口来对接 Lua 脚本和网页进行交互了。

不过，在这里我们更加关心的肯定是如何用 Python 来实现，用 Python 实现如下：

```
import requests
from urllib.parse import quote

lua = '''
function main(splash)
    return 'hello'
end
'''

url = 'http://localhost:8050/execute?lua_source=' + quote(lua)
response = requests.get(url)
print(response.text)
```

运行结果：

`hello`

在这里我们用 Python 中的三引号来将 Lua 脚本包括起来，然后用 urllib.parse 模块里的 quote()方法将脚本进行 URL 转码，随后构造了 Splash 请求 URL，将其作为 lua_source 参数传递，这样运行结果就会显示 Lua 脚本执行后的结果。我们再来一个实例看一下：

```
import requests
from urllib.parse import quote

lua = '''
function main(splash, args)
  local treat = require("treat")
  local response = splash:http_get("http://httpbin.org/get")
    return {
    html=treat.as_string(response.body),
    url=response.url,
    status=response.status
    }
end
'''

url = 'http://localhost:8050/execute?lua_source=' + quote(lua)
response = requests.get(url)
print(response.text)
```

运行结果：

```{"url": "http://httpbin.org/get", "status": 200, "html": "{\n  \"args\": {}, \n  \"headers\": {\n    \"Accept-Encoding\": \"gzip, deflate\", \n    \"Accept-Language\": \"en,*\", \n    \"Connection\": \"close\", \n    \"Host\": \"httpbin.org\", \n    \"User-Agent\": \"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/602.1 (KHTML, like Gecko) splash Version/9.0 Safari/602.1\"\n  }, \n  \"origin\": \"60.207.237.85\", \n  \"url\": \"http://httpbin.org/get\"\n}\n"}```

返回结果是 Json 形式，我们成功获取了请求的 URL、状态码和网页源代码。
如此一来，我们之前所说的 Lua 脚本均可以用此方式与 Python 进行对接，这样的话，所有网页的动态渲染、模拟点击、表单提交、页面滑动、延时等待后的一些结果均可以自由控制，获取页面源码和截图都不在话下。