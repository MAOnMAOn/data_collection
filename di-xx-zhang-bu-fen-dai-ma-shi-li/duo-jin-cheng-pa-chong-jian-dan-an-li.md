## 6.7 PyQurey 多进程爬虫   ---\(目标站点：智联招聘\)

之前我们介绍了 Python3 下的多进程包，multiprocessing，现在我们就以智联招聘为目标站点，结合先前所介绍的相关知识\(requests，pyquery\)，进行简单的数据爬取。代码非常简单，所以没有做过多的说明\(毕竟知乎上一大堆\)。

### 1. 请求网页，获取 html 文本

```
import time
from urllib.parse import quote
from multiprocessing import Pool, cpu_count

import requests
import pandas as pd
from pyquery import PyQuery as pq
from sqlalchemy import create_engine

def request_url(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36',
        'Connection': 'keep-alive',
    }
    wb_data = requests.get(url,headers=headers).text
    return wb_data
```

首先，导入模块，其中 sqlalchemy 是 Python 中一个操作数据库的驱动。我们在这里简单添加了用户代理，以获取网页的 html 文本。

### 2. 通过 PyQuery 进行解析与数据选取

在 parse\_html 函数中，我们通过 CSS 选择器进行数据选取，生成了一个嵌套列表。

```
def parse_html(wb_data):
    doc = pq(wb_data)
    company = doc('.gsmc > a:nth-child(1)').text().split(' ')
    address = doc('tr:nth-child(1) > td.gzdd').text().split(' ')
    salary = doc('tr:nth-child(1) > td.zwyx').text().split(' ')
    return {"company":company,"address":address,"salary":salary,}
```

### 3. 通过 Pandas 进行简单的数据合并

首先，通过 pandas 把列表变成对应的数据框，然后将数据框依据索引合并成大表。

```
def data_processing(parse_dict):
    company1 = pd.DataFrame(parse_dict["company"], columns=['公司'])
    address1 = pd.DataFrame(parse_dict["address"], columns=['地址'])
    salary1 = pd.DataFrame(parse_dict["salary"], columns=['收入'])
    df = company1.join(address1).join(salary1)
    return df
```

### 4. 数据存储

这里我们使用 sqlalchemy 进行 ORM 操作, 在 Dataframe 的 to\_sql 方法之中的 if\_exists 选项填入追加选项，自动创建并添加数据。同时一次性存入1000条数据。

```
def save_into_mysql(dataframe,table_name):
    MYSQL_URL = 'mysql+pymysql://test2:123456@192.168.1.159:3306/spider?charset=utf8'
    #MYSQL_URL = 'mysql+pymysql://%s:%s@%s:3306/%s?charset=%s' % (USER, PASSWD, HOST, DB, CHARSET)
    con = create_engine(MYSQL_URL)
    dataframe.to_sql(name = table_name,con=con,if_exists='append',index=False,chunksize=1000)
```

### 5. 采用进程池

通过 multiprocessing 中的 cpu\_count 方法，获取本地主机 cpu 核心数，这里的 run 函数是前面各个过程的集成，我们通过 pool.map 把网址列表中的链接进行爬取。

```
    pool = Pool(processes = cpu_count())
    pool.map(run, url_list)
    pool.close()
    pool.join()
```

### 6. 代码合并

```
import time
from urllib.parse import quote
from multiprocessing import Pool,cpu_count

import requests
import pandas as pd
from pyquery import PyQuery as pq
from sqlalchemy import create_engine


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
    url_pattern = 'http://sou.zhaopin.com/jobs/searchresult.ashx?kw={key}&sm=0&p={num}'
    url_list = [url_pattern.format(key=quote('产品经理'),num=num) for num in range(1,51)]
    pool = Pool(processes = cpu_count())
    pool.map(run, url_list)
    pool.close()
    pool.join()    
    b = time.time()
    print('总共花了:',b-a)
```

### 7. 引入 ProcessPoolExecutor

这里我们引入 ProcessPoolExecutor ，对 multiprocessing.Pool 进行进一步封装，首先，我们需要导入相应模块：

`from concurrent.futures import ProcessPoolExecutor`

然后，把 `if __name__ == '__main__':` 下面的代码进行修改，代码如下：

```
if __name__ == '__main__':
    a = time.time()
    url_pattern = 'http://sou.zhaopin.com/jobs/searchresult.ashx?kw={key}&sm=0&p={num}'
    url_list = [url_pattern.format(key=quote('产品经理'),num=num) for num in range(1,51)]
    with ProcessPoolExecutor(max_workers=4) as executor:
        executor.map(run, url_list)

    b = time.time()
    print('总共花了:',b-a)
```

在代码量减少的同时，运行效率变化不大，总共花了: 11.856663465499878 爬取了3019个条目。当然，你也可以尝试一下 concurrent.futures 中的 ThreadPoolExecutor 模块，用法基本相同，总共花了: 9.865242958068848！我们将会在下一节说明该模块的简单使用。

