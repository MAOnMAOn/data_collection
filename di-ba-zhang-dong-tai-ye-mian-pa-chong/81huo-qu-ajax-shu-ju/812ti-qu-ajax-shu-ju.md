## 8.1.2 Ajax结果提取

仍然以天猫移动端页面为例，用 Python 来模拟这些 Ajax 请求，把部分商品内容爬取下来。

### 1. 分析请求
我们打开 Ajax 的 XHR 过滤器，然后一直滑动页面加载新的商品内容，可以看到会不断有Ajax请求发出。

我们选定其中一个请求来分析一下它的参数信息，点击该请求进入详情页面可以发现这是一个 GET 类型的请求，请求链接为：`https://list.tmall.com/m/search_items.htm?page_size=20&page_no=1&q=sql` ，请求的参数有三个：page_size、q、page_no。

随后我们再看一下其他的请求，观察一下这些请求，发现它们的 page_size、q 始终如一。所以改变的值就是 page_no，很明显这个参数就是用来控制分页的，page=1 代表第一页，page=2 代表第二页，以此类推。

以上的推断过程可以实际观察参数的规律即可得出。

### 2. 分析响应
随后我们观察一下这个请求的响应内容，它是一个 Json 格式，浏览器开发者工具自动为做了解析方便我们查看，可以看到其中有我们想要的数据。这样我们可以请求一个接口就得到 20 个商品数据，而且请求的时候只需要改变 page 参数即可。这样我们只需要简单做一个循环就可以获取到想要的数据了。

### 3. 实战演练
在这里我们就开始用程序来模拟这些 Ajax 请求，将开始爬取天猫数据。

首先我们定义一个方法，来获取每次请求的结果，在请求时page 是一个可变参数，所以我们将它作为方法的参数传递进来，代码如下：

```
from urllib.parse import urlencode
import requests
base_url = 'https://list.tmall.com/m/search_items.htm?'

headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36',
}

def get_page(page):
    params = {
        'page_size': 20,
        'q': 'sql',
        'page_no': page
    }
    url = base_url + urlencode(params)
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
    except requests.ConnectionError as e:
        print('Error', e.args)
```

首先在这里我们定义了一个 base_url 来表示请求的 URL 的前半部分，接下来构造了一个参数字典，其中只有 page_no 是可变参数，接下来我们调用了 urlencode() 方法将参数转化为 URL 的 GET请求参数，即类似于 page_size=20&page_no=2&q=sql 这样的形式，随后 base_url 与参数拼合形成一个新的 URL，然后我们用 Requests 请求这个链接，加入 headers 参数，然后判断响应的状态码，如果是200，则直接调用 json() 方法将内容解析为 Json 返回，否则不返回任何信息，如果出现异常则捕获并输出其异常信息。

随后我们需要定义一个解析方法，用来从结果中提取我们想要的信息,赋值为一个新的字典返回,最终把数据进行存储即可。
