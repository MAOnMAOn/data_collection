## 5.1.2 异常处理

之前我们了解了请求的发送过程，但出现了异常怎么办呢？这时如果我们不处理这些异常，程序很可能报错而终止运行，所以异常处理还是十分有必要的。Urllib 的 error 模块定义了由 request 模块产生的异常。如果出现了问题，request 模块便会抛出 error 模块中定义的异常。

### 1. URLError
URLError 类来自 Urllib 库的 error 模块，它继承自 OSError 类，是 error 异常模块的基类，由 request 模块生的异常都可以通过捕获这个类来处理。
它具有一个属性 reason，即返回错误的原因。下面用一个实例来感受一下：

```
from urllib import request, error
try:
    response = request.urlopen('http://www.abcffde.com')
except error.URLError as e:
    print(e.reason)
```

打开不存在的页面，照理来说应该会报错，但是这时我们捕获了 URLError 这个异常，运行结果如下：

`Not Found`

程序没有直接报错，而是输出了如上内容，这样通过如上操作，我们就可以避免程序异常终止，同时异常得到了有效处理。

### 2. HTTPError
它是 URLError 的子类，专门用来处理 HTTP 请求错误，比如认证请求失败等等。
它有三个属性。

 - code，返回 HTTP Status Code，即状态码，比如 404 网页不存在，500 服务器内部错误等等。
 - reason，同父类一样，返回错误的原因。
 - headers，返回 Request Headers。

下面我们来用几个实例感受一下：

```
from urllib import request,error
try:
    response = request.urlopen('http://192.168.1.1')
except error.HTTPError as e:
    print(e.reason, e.code, seq='\n')
```

运行结果：

```
Not Found
Unauthorized 401
```

依然是同样的网址，在这里我们捕获了 HTTPError 异常，输出了 reason、code 属性。因为 URLError 是 HTTPError 的父类，所以我们可以先选择捕获子类的错误，再去捕获父类的错误，所以上述代码更好的写法如下：

```
from urllib import request, error

try:
    response = request.urlopen('http://192.168.1.1')
except error.HTTPError as e:
    print(e.reason, e.code, sep='\n')
except error.URLError as e:
    print(e.reason)
else:
    print('Request Successfully')
```

这样我们就可以做到先捕获 HTTPError，获取它的错误状态码、原因、Headers 等详细信息。如果非 HTTPError，再捕获 URLError 异常，输出错误原因。最后用 else 来处理正常的逻辑，这是一个较好的异常处理写法。

有时候 reason 属性返回的不一定是字符串，也可能是一个对象，再看下面的实例：

```
import socket
import urllib.request
import urllib.error

try:
    response = urllib.request.urlopen('https://www.baidu.com', timeout=0.01)
except urllib.error.URLError as e:
    print(type(e.reason))
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

在这里我们直接设置了超时时间来强制抛出 timeout 异常。运行结果如下：

```
<class 'socket.timeout'>
TIME OUT
```

可以发现 reason 属性的结果是 socket.timeout 类。所以在这里我们可以用 isinstance() 方法来判断它的类型，做出更详细的异常判断。