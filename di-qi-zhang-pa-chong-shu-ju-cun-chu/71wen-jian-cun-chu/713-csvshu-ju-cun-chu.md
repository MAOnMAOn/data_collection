## 5.1.3 CSV文件存储

CSV 文件是一个字符序列，可以由任意数目的记录组成，记录间以某种换行符分隔，每条记录由字段组成，字段间的分隔符是其它字符或字符串，最常见的是逗号或制表符，不过所有记录都有完全相同的字段序列，相当于一个结构化表的纯文本形式，结构简单清晰，所以有时候我们用 CSV 来保存数据相对比较方便。

### 1、写入
在这里我们先看一个最简单的例子：

```
import csv
with open('data.csv', 'w') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(['id', 'name', 'age'])
    writer.writerow(['10001', 'Mike', 20])
    writer.writerow(['10002', 'Bob', 22])
    writer.writerow(['10003', 'Jordan', 21])
```

首先打开了一个 data.csv 文件，然后指定了打开的模式为 w，即写入，获得文件句柄，随后调用 csv 库的 writer() 方法初始化一个写入对象，传入该句柄，然后调用 writerow() 方法传入每行的数据即可完成写入。
运行结束后会生成一个名为 data.csv 的文件，数据就成功写入了。

如果我们想修改列与列间的默认分隔符可传入 delimiter 参数，代码如下：

```
import csv

with open('data.csv', 'w') as csvfile:
    writer = csv.writer(csvfile, delimiter=' ')
    writer.writerow(['id', 'name', 'age'])
    writer.writerow(['10001', 'Mike', 20])
    writer.writerow(['10002', 'Bob', 22])
    writer.writerow(['10003', 'Jordan', 21])
```

例如这里在初始化写入对象的时候传入 delimiter 为空格，这样输出的结果的每一列就是以空格分隔的了。

当然我们也可以调用 writerows() 方法同时写入多行，此时参数就需要为二维列表，例如：

```
import csv
with open('data.csv', 'w') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(['id', 'name', 'age'])
    writer.writerows([['10001', 'Mike', 20], ['10002', 'Bob', 22], ['10003', 'Jordan', 21]])
```

输出效果是相同的。

不过一般情况下，爬虫爬取的都是结构化数据，一般用字典表示，在 csv 库中也提供了字典的写入方式，实例如下：

```
import csv
with open('data.csv', 'w') as csvfile:
    fieldnames = ['id', 'name', 'age']
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerow({'id': '10001', 'name': 'Mike', 'age': 20})
    writer.writerow({'id': '10002', 'name': 'Bob', 'age': 22})
    writer.writerow({'id': '10003', 'name': 'Jordan', 'age': 21})
```

在这里我们先定义了三个字段，用 fieldnames 表示，然后传给 DictWriter 初始化一个字典写入对象，然后可以先调用 writeheader() 方法先写入头信息，然后再调用 writerow() 方法传入相应字典即可，最终写入的结果是完全相同的 。

另外如果我们想追加写入的话可以修改文件的打开模式，如将 open() 函数的第二个参数改成 a 就可以变成追加写入，代码如下：

```
import csv
with open('data.csv', 'a') as csvfile:
    filenames = ['id', 'name', 'age']
    writer = csv.DictWriter(csvfile, fieldnames=filenames)
    writer.writerow({'id': '10004', 'name': 'Durant', 'age': 22})
```

如果我们要写入中文内容的话可能会遇到字符编码的问题，此时我们需要给 open() 参数指定一个编码格式，比如这里再写入一行包含中文的数据，代码需要改写如下：

```
import csv
with open('data.csv', 'a', encoding='utf-8') as csvfile:
    fieldnames = ['id', 'name', 'age']
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writerow({'id': '10005', 'name': '王伟', 'age': 22})
```

在这里需要给 open() 函数指定编码，否则可能会发生编码错误。
以上便是 CSV 文件的写入方法。

另外如果我们接触过 Pandas 等库的话，可以调用 DataFrame 对象的 to_csv() 方法也可以非常方便地将数据写入到 CSV 文件中, 同时我们还可以通过设置 to_csv() 方法的mode参数为 a 实现追加写入。

### 2、写入
我们同样可以使用 csv 库来读取 CSV 文件，例如我们现在将刚才写入的文件内容读取出来，代码如下：

```
import csv
with open('data.csv', 'r', encoding='utf-8') as csvfile:
    reader = csv.reader(csvfile)
    for row in reader:
        print(row)
```

运行结果：

```
['id', 'name', 'age']
['10001', 'Mike', '20']
['10002', 'Bob', '22']
['10003', 'Jordan', '21']
['10004', 'Durant', '22']
['10005', '王伟', '22']
```

在这里我们构造的是 Reader 对象，通过遍历输出了每行的内容，每一行都是一个列表形式，注意在这里如果 CSV 文件中包含中文的话需要指定文件编码。

另外如果我们接触过 Pandas 的话，可以利用 read_csv() 方法将数据从 CSV 中读取出来，例如：

```
import pandas  as pd
df = pd.read_csv('data.csv')
print(df)
```

运行结果：

```
      id    name  age
0  10001    Mike   20
1  10002     Bob   22
2  10003  Jordan   21
3  10004  Durant   22
4  10005    王伟   22
```

在做数据分析的时候此种方法用的比较多，也是一种比较方便的读取 CSV 文件的方法。