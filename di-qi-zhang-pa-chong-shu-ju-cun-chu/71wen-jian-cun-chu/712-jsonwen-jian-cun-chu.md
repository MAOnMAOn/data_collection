## 7.1.2 Json文件存储

Json，全称为 JavaScript Object Notation, 也就是 JavaScript 对象标记，通过对象和数组的组合来表示数据，构造简洁但是结构化程度非常高，它是一种轻量级的数据交换格式。

### 1、读取Json
Python 内置的 json 库来供我们实现 Json 文件的读写操作，可以调用 json 库的 loads() 方法将 Json 文本字符串转为 Json 对象，可以通过 dumps()方法将 Json 对象转为文本字符串。

例如在这里有一段 Json 形式的字符串，它是 str 类型，我们用 Python 将可其转换为可操作的数据结构。

```
import json

str = '''
[{
    "name": "Bob",
    "gender": "male",
    "birthday": "1992-10-18"
}, {
    "name": "Selina",
    "gender": "female",
    "birthday": "1995-10-18"
}]
'''
print(type(str))
data = json.loads(str)
print(data)
print(type(data))
```

运行结果：

```
<class 'str'>
[{'name': 'Bob', 'gender': 'male', 'birthday': '1992-10-18'}, {'name': 'Selina', 'gender': 'female', 'birthday': '1995-10-18'}]
<class 'list'>
```

在这里我们使用了 loads() 方法将字符串转为 Json 对象，由于最外层是中括号，所以最终的类型是列表类型。

如果我们是从 Json 文本中读取内容，例如在这里有一个data.json 文本文件，其内容是刚才我们所定义的 Json 字符串。
我们可以先将文本文件内容读出，然后再利用 loads() 方法转化。

```
import json
with open('data.json', 'r') as file:
    str = file.read()
    data = json.loads(str)
    print(data)
```

运行结果：

```
[{'name': 'Bob', 'gender': 'male', 'birthday': '1992-10-18'}, {'name': 'Selina', 'gender': 'female', 'birthday': '1995-10-18'}]
```

以上是读取 Json 文件的方法。

### 2、输出Json
另外我们还可以调用 dumps() 方法来将 Json 对象转化为字符串,然后再调用文件的 write() 方法即可写入到文本。
例如我们将刚上例中的列表重新写入到文本。

```
import json

data = [{
    'name': 'Bob',
    'gender': 'male',
    'birthday': '1992-10-18'
}]
with open('data.json', 'w') as file:
    file.write(json.dumps(data))
```

另外如果我们想保存 Json 的格式，可以再加一个参数 indent，代表缩进字符个数。
with open('data.json', 'w') as file:
    file.write(json.dumps(data, indent=2))
这样得到的内容会自动带有缩进，格式会更加清晰。

另外如果 Json 中包含中文字符，为了输出中文，我们还需要指定一个参数 ensure_ascii 为 False，另外规定文件输出的编码。例如我们将之前的 Json 的部分值改为中文，再用之前的方法写入到文本。
```
import json

data = [{
    'name': '王伟',
    'gender': '男',
    'birthday': '1992-10-18'
}]
with open('data.json', 'w', encoding='utf-8') as file:
    file.write(json.dumps(data, indent=2, ensure_ascii=False))
```

这样我们就可以输出 Json 为中文了，所以如果字典中带有中文的内容我们需要设置 ensure_ascii 参数为 False 才可正常写入中文。