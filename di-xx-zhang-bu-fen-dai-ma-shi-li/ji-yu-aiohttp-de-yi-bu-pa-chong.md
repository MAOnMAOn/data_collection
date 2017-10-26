## x.x 基于 aiohttp 的异步爬虫(智联招聘)

先前，我们已经了解了了异步编程相比与同步编程的巨大优势，这里我们可以基于实现 asyncio/aiohttp 模块，实现一个简单的异步爬虫。

### 1. 案例来源
可能你已经听过[开源程序架构](http://aosabook.org/en/index.html)系列书籍。今天介绍的爬虫就是改造自[500 Lines or Less中的爬虫项目](https://github.com/aosabook/500lines/tree/master/crawler)。

由于使用了异步的aiohttp与asyncio，性能还是相当不错的。运行时发现一小时可以爬下五万个网页，虽然链家对爬虫很宽容，但是也不能火力全开，最终尝试后发现将每个协程爬取的睡眠时间设置为0.1s比较适中。至于线程池 + requests + bs4 的结构，没有尝试过，不好对比它们的性能差距。
以下是官网的介绍，Python 版本要求>3.4。500行项目里的爬虫并不爬取网页数据，只是通过搜集每一页的url链接展示了如何结合 asyncio 库使用 aiohttp 这个异步 HTTP 客户端。

```
Authors: A. Jesse Jiryu Davis and Guido van Rossum
Project: Web crawler
Requirements:
  * Python 3.4+
  * aiohttp

This is a web crawler. You give it a URL and it will crawl that
website by following href links in the HTML pages.

It doesn't do anything with the crawled pages, and the algorithm for
finding links is intentionally naive -- those parts are easily
modified, and not of particular interest (just use your favorite HTML
parser instead of a regular expression).

The point of the example is to show off how to write a reasonably complex HTTP
client application using the asyncio module. This module, originally nicknamed
Tulip, is new in the Python 3.4 standard library, based on PEP 3156. The
example uses an HTTP client implementation for asyncio called "aiohttp", by
Andrew Svetlov, Nikolay Kim, and others.
```

### 2. 爬虫依赖库与全局变量
导入模块

```
import time
from datetime import datetime
from asyncio import Queue
from urllib.parse import quote

import asyncio
import aiohttp
import pandas as pd
from pyquery import PyQuery as pq
from sqlalchemy import create_engine
```

配置全局变量

```
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
    'Connection': 'keep-alive',
}

table_name = 'zhilian'
MYSQL_URL = 'mysql+pymysql://%s:%s@%s:3306/%s?charset=%s' % (USER, PASSWD, HOST, DB, CHARSET)
con = create_engine(MYSQL_URL)
max_page = 90
sleep_interval = 0  # 睡眠时间
```

init构造方法

```
def __init__(self, roots, max_tries=4, max_tasks=10, _loop=None):
    self.loop = _loop or asyncio.get_event_loop()
    self.roots = roots
    self.max_tries = max_tries
    self.max_tasks = max_tasks
    self.urls_queue = Queue(loop=self.loop)
    self.seen_urls = set()
    self.session = aiohttp.ClientSession(loop=self.loop, headers=headers)
    for root in roots:
        self.urls_queue.put_nowait(root)

    self.started_at = datetime.now()
    self.end_at = None
```

 - self.loop:是事件循环;
 - self.roots:传入的url列表，也就是90个目录页的url，通过迭代器实现;
 - self.max_tries:每个url的最大重试次数，如果几次打开都失败，就忽略掉此页;
 - self.max_tasks:任务数，也就是协程数量;
 - self.urls_queue:是待爬取的url链接队列;
 - self.seen_urls:存储着已经爬取了的页面的集合，用于计算总爬取数量 ;
 - self.session:aiohttp的异步会话，也是需要传入一个事件循环作为参数;
 - self.urls_queue.putnowait():往队列里放入目录页;
 - self.started_at, self.end_at：存放着开始时间与结束时间，用于计算总耗时;

### 3. 生产者、消费者模式
一般情况下，多数异步问题都可以套用生产者消费者模型。在这里，创建一个生产者去生产url地址。

run函数比较简单，根据任务数量创建多个work协程用来消费url地址，都传入同一个事件循环;并且赋值初始时间，等待队列完成，再取得结束时间，退出每一个协程。
每一个work协程都是无限的去从urls_queue中取url地址，网页请求是一个耗时的IO任务，所以被async def定义，一旦进入阻塞立刻就切换另一个协程运行，耗时主要体现在session会话得到response上，而网页解析，处理数据是个CPU密集性计算，但运算量不高，耗时相对更少。

```
    async def work(self):
        try:
            while True:
                url = await self.urls_queue.get()
                await self.handle(url)
                time.sleep(sleep_interval)
                self.urls_queue.task_done()
        except asyncio.CancelledError:
            pass

    async def run(self):
        workers = [asyncio.Task(self.work(), loop=self.loop)
                   for _ in range(self.max_tasks)]
        self.started_at = datetime.now()
        await self.urls_queue.join()
        self.end_at = datetime.now()
        for w in workers:
            w.cancel()
```

这里我们还可以把数据存储也改成异步存储，比如使用aiomysql 或 twisted。

### 4. 时间开销
相比于同步模式，异步代码在在时间开销上，表现较佳。耗时为 6.876924 secs 而相同情况下的同步代码耗时则为 68.32336187 secs。

相对于其他并发模式(asyncio + aiohttp + ThreadPoolExecutor 或者 asyncio + aiohttp + ProcessPoolExecutor)，经过本地的简单测试，
可以感受到原生的 asyncio + aiohttp 是最快的。所以在实际工作中推荐使用 asyncio + aiohttp 。

### 5. 完整代码
异步代码

```
import time
import asyncio
import aiohttp
from datetime import datetime
from asyncio import Queue
from urllib.parse import quote

import pandas as pd
from pyquery import PyQuery as pq
from sqlalchemy import create_engine

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
    'Connection': 'keep-alive',
}

table_name = 'zhilian'
MYSQL_URL = 'mysql+pymysql://test2:123456@192.168.1.159:3306/spider?charset=utf8'
#MYSQL_URL = 'mysql+pymysql://%s:%s@%s:3306/%s?charset=%s' % (USER, PASSWD, HOST, DB, CHARSET)
con = create_engine(MYSQL_URL)
max_page = 90

sleep_interval = 0


def get_page_url(_max_page, start=1):
    page = start
    url_pattern = 'http://sou.zhaopin.com/jobs/searchresult.ashx?kw={key}&sm=0&p={p}'
    while page < _max_page:
        yield url_pattern.format(key=quote('产品经理') ,p=page)
        page += 1


class Crawler:
    def __init__(self, roots, max_tries=4, max_tasks=10, _loop=None):
        self.loop = _loop or asyncio.get_event_loop()
        self.roots = roots
        self.max_tries = max_tries
        self.max_tasks = max_tasks
        self.urls_queue = Queue(loop=self.loop)
        self.seen_urls = set()
        self.session = aiohttp.ClientSession(loop=self.loop, headers=headers)
        for root in roots:
            self.urls_queue.put_nowait(root)

        self.started_at = datetime.now()
        self.end_at = None

    def close(self):
        self.session.close()

    @staticmethod
    async def fetch_pqquery(response):  # 进行网页的请求，返回 html
        if response.status == 200:
            text = await response.text()
            doc = pq(text)
            return doc

    def parse_root_pqquery(self, doc):
        company = doc('.gsmc > a:nth-child(1)').text().split(' ')
        address = doc('tr:nth-child(1) > td.gzdd').text().split(' ')
        salary = doc('tr:nth-child(1) > td.zwyx').text().split(' ')

        company1 = pd.DataFrame(company, columns=['公司'])
        address1 = pd.DataFrame(address, columns=['地址'])
        salary1 = pd.DataFrame(salary, columns=['收入'])

        df = company1.join(address1).join(salary1)
        df.to_sql(name = table_name,con=con,if_exists='append',index=False,chunksize=1000)

    async def handle(self, url):  # 进行请求，并执行 retry
        tries = 0
        while tries < self.max_tries:
            try:
                response = await self.session.get(url, allow_redirects=False)
                break
            except aiohttp.ClientError:
                pass
            tries += 1
        try:
            doc = await self.fetch_pqquery(response)
            print('root:{}'.format(url))
            self.parse_root_pqquery(doc)
        finally:
            await response.release()

    async def work(self):
        try:
            while True:
                url = await self.urls_queue.get()
                await self.handle(url)
                time.sleep(sleep_interval)
                self.urls_queue.task_done()
        except asyncio.CancelledError:
            pass

    async def run(self):
        workers = [asyncio.Task(self.work(), loop=self.loop)
                   for _ in range(self.max_tasks)]
        self.started_at = datetime.now()
        await self.urls_queue.join()
        self.end_at = datetime.now()
        for w in workers:
            w.cancel()

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    crawler = Crawler(get_page_url(max_page), max_tasks=100)
    loop.run_until_complete(crawler.run())
    print('Finished {0} urls in {1} secs'.format(
        len(crawler.seen_urls),
        (crawler.end_at - crawler.started_at).total_seconds()))
    crawler.close()
    loop.close()
```

同步执行的代码：

```
import time
from urllib.parse import quote

import requests
import pandas as pd
from pyquery import PyQuery as pq
from sqlalchemy import create_engine

url_pattern = 'http://sou.zhaopin.com/jobs/searchresult.ashx?kw={key}&sm=0&p={num}'
url_list = [url_pattern.format(key=quote('产品经理'),num=num) for num in range(1,90)]


def request_url(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
        'Connection': 'keep-alive',
    }
    wb_data = requests.get(url,headers=headers).text
    return wb_data

def parse_html(wb_data):
    doc = pq(wb_data)
    company = doc('.gsmc > a:nth-child(1)').text().split(' ')
    address = doc('tr:nth-child(1) > td.gzdd').text().split(' ')
    salary = doc('tr:nth-child(1) > td.zwyx').text().split(' ')
    return {"company":company,"address":address,"salary":salary,}

def data_processing(parse_dict):
    company1 = pd.DataFrame(parse_dict["company"], columns=['公司'])
    address1 = pd.DataFrame(parse_dict["address"], columns=['地址'])
    salary1 = pd.DataFrame(parse_dict["salary"], columns=['收入'])
    df = company1.join(address1).join(salary1)
    return df

def save_into_mysql(dataframe,table_name):
    MYSQL_URL = 'mysql+pymysql://test2:123456@192.168.1.159:3306/spider?charset=utf8'
    #MYSQL_URL = 'mysql+pymysql://%s:%s@%s:3306/%s?charset=%s' % (USER, PASSWD, HOST, DB, CHARSET)
    con = create_engine(MYSQL_URL)
    dataframe.to_sql(name = table_name,con=con,if_exists='append',index=False,chunksize=1000)

def run(url):
    wb_data = request_url(url)
    parse_dict = parse_html(wb_data)
    dataframe = data_processing(parse_dict)
    save_into_mysql(dataframe=dataframe, table_name='zhilian')

if __name__ == '__main__':
    a = time.time()
    for url in url_list:
        num = 1
        print('This is NO.{}!'.format(num),'\n',url)
        num += 1
        run(url)
    b = time.time()
    print('总共花了:',b-a)
```