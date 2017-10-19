## 6.5 BeautifulSoup的使用

虽然正则表达式十分强大，但是一旦正则写的有问题，可能得到的就不是我们想要的结果了，而且对于一个网页来说，都有一定的特殊的结构和层级关系，而且很多节点都有id或class来对作区分，因此借助于它们的结构和属性来提取也是可以的。

BeautiSoup，它就是借助网页的结构和属性等特性来解析网页的工具，有了它我们不用再去写一些复杂的正则，只需要简单的几条语句就可以完成网页中某个元素的提取。

### 1. BeautifulSoup简介
简单来说，BeautifulSoup 就是 Python 的一个 HTML 或 XML 的解析库，我们可以用它来方便地从网页中提取数据，官方的解释如下：

>BeautifulSoup提供一些简单的、Python式的函数用来处理导航、搜索、修改分析树等>功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，不需要多少代码就可以写出一个完整的应用程序。 BeautifulSoup 自动将输入文档转换为 Unicode 编码，输出文档转换为 utf-8 编码。你不需要考虑编码方式，除非文档没有指定一个编码方式，这时你仅仅需要说明一下原始编码方式就可以了。 BeautifulSoup 已成为和 lxml、html6lib 一样出色的 Python 解释器，为用户灵活地提供不同的解析策略或强劲的速度。
所以说，利用它我们可以省去很多繁琐的提取工作，提高解析效率。

### 2. 解析器
BeautifulSoup 在解析的时候实际上是依赖于解析器的，它除了支持 Python 标准库中的 HTML 解析器，还支持一些第三方的解析器比如 LXML，下面我们对 BeautifulSoup 支持的解析器及它们的一些优缺点做一个简单的对比:

解析器  |	使用方法  | 	优势      |      	劣势
--------|-------------|---------------|--------------------------------
Python标准库 |	BeautifulSoup(markup, "html.parser") |	内置标准库、执行速度适中、文档容错能力强  |	Python 2.7.3 or 3.2.2)前的版本中文容错能力差
LXML HTML 解析器 |	BeautifulSoup(markup, "lxml") |	速度快、文档容错能力强|	需要安装C语言库
LXML XML 解析器	| BeautifulSoup(markup, "xml") |	速度快、唯一支持XML的解析器  |	需要安装C语言库
html5lib  |	BeautifulSoup(markup, "html5lib") |	最好的容错性、以浏览器的方式解析文档、生成 HTML5 格式的文档 |	解析速度较慢

所以通过以上对比可以看出，LXML 这个解析器有解析 HTML 和 XML 的功能，而且速度快，容错能力强，所以推荐使用这个解析器来进行解析。后面 BeautifulSoup 的用法实例也统一用这个解析器来演示。

### 3. 基本使用
下面我们首先用一个实例来感受一下 BeautifulSoup 的基本使用：

```
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.prettify())
print(soup.title.string)
```

运行结果：

```
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
 <body>
  <p class="title" name="dromouse">
   <b>
    The Dormouse's story
   </b>
  </p>
  <p class="story">
   Once upon a time there were three little sisters; and their names were
   <a class="sister" href="http://example.com/elsie" id="link1">
    <!-- Elsie -->
   </a>
   ,
   <a class="sister" href="http://example.com/lacie" id="link2">
    Lacie
   </a>
   and
   <a class="sister" href="http://example.com/tillie" id="link3">
    Tillie
   </a>
   ;
and they lived at the bottom of a well.
  </p>
  <p class="story">
   ...
  </p>
 </body>
</html>
The Dormouse's story
```

首先声明了一个变量 html，它是一个 HTML 字符串，但它并不是一个完整的 HTML 字符串，body 和 html 节点都没有闭合，我们将它当作第一个参数传给 BeautifulSoup 对象，第二个参数传入解析器类型，在这里使用 lxml，这样就完成了 BeaufulSoup 对象的初始化，将它赋值给 soup 这个变量。那么接下来就可以通过调用 soup 的各个方法和属性对这串 HTM L代码解析了。

我们首先调用了 prettify() 方法，这个方法可以把要解析的字符串以标准的缩进格式输出，在这里注意到输出结果里面包含了 body 和 html 节点，也就是说对于不标准的 HTML 字符串 BeautifulSoup 可以自动更正格式，这一步实际上不是由 prettify() 方法做的，这个更正实际上在初始化 BeautifulSoup 时就完成了。

然后我们调用了 soup.title.string ，这个实际上是输出了 HTML 中 title 节点的文本内容。所以 soup.title 就可以选择出 HTML 中的 title 节点，再调用 string 属性就可以得到里面的文本了，所以我们就可以通过简单地调用几个属性就可以完成文本的提取了。

### 4. 节点选择器
选择元素的时候直接通过调用节点的名称就可以选择节点元素，然后再调用 string 属性就可以得到节点内的文本了，这种选择方式速度非常快，如果单个节点结构话层次非常清晰，可以选用这种方式来解析。

***选择元素***

下面我们再用一个例子详细说明一下它的选择方法：

```
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.title)
print(type(soup.title))
print(soup.title.string)
print(soup.head)
print(soup.p)
```

运行结果：

```
<title>The Dormouse's story</title>
<class 'bs4.element.Tag'>
The Dormouse's story
<head><title>The Dormouse's story</title></head>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
```

在这里我们依然选用了刚才的 HTML 代码，我们首先打印输出了 title 节点的选择结果，输出结果正是 title 节点加里面的文字内容。接下来输出了它的类型，是 bs4.element.Tag 类型，这是 BeautifulSoup 中的一个重要的数据结构，经过选择器选择之后，选择结果都是这种 Tag 类型，它具有一些属性比如 string 属性，调用 Tag 的 string 属性，就可以得到节点的文本内容了。

接下来我们又尝试选择了 head 节点，结果也是节点加其内部的所有内容，再接下来选择了 p 节点，不过这次情况比较特殊，我们发现结果是第一个 p 节点的内容，后面的几个 p 节点并没有选择到，也就是说，当有多个节点时，这种选择方式只会选择到第一个匹配的节点，其他的后面的节点都会忽略。

***提取信息***

调用 string 属性来获取文本的值，那我们要获取节点属性值怎么办呢？获取节点名怎么办呢？下面我们来统一梳理一下信息的提取方式

**获取名称**

可以利用 name 属性来获取节点的名称。还是以上面的文本为例，我们选取 title 节点，然后调用 name 属性就可以得到节点名称。

`print(soup.title.name)`

运行结果:

`title`

**获取属性**

每个节点可能有多个属性，比如 id，class 等等，我们选择到这个节点元素之后，可以调用 attrs 获取所有属性。

```
print(soup.p.attrs)
print(soup.p.attrs['name'])
```

运行结果：

```
{'class': ['title'], 'name': 'dromouse'}
dromouse
```

可以看到 attrs 的返回结果是字典形式，把选择的节点的所有属性和属性值组合成一个字典，接下来如果要获取 name 属性，就相当于从字典中获取某个键值，只需要用中括号加属性名称就可以得到结果了，比如获取 name 属性就可以通过 attrs['name'] 得到相应的属性值。

其实这样的写法还有点繁琐，还有一种更简单的获取方式，我们可以不用写 attrs，直接节点元素后面加中括号，传入属性名就可以达到属性值了，样例如下：

```
print(soup.p['name'])
print(soup.p['class'])
```

运行结果：

```
dromouse
['title']
```

在这里注意到有的返回结果是字符串，有的返回结果是字符串组成的列表。比如 name 属性的值是唯一的，返回的结果就是单个字符串，而对于 class，一个节点元素可能由多个 class，所以返回的是列表，所以在实际处理过程中要注意判断类型。

**获取内容**

可以利用 string 属性获取节点元素包含的文本内容，比如上面的文本我们获取第一个 p 节点的文本：

`print(soup.p.string)`

运行结果：

`The Dormouse's story`

再次注意一下这里选择到的 p 节点是第一个 p 节点，获取的文本也就是第一个 p 节点里面的文本。

***嵌套选择***

在上面的例子中我们知道每一个返回结果都是 bs4.element.Tag 类型，它同样可以继续调用节点进行下一步的选择，比如我们获取了 head 节点元素，我们可以继续调用 head 来选取其内部的 head 节点元素。

```
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.head.title)
print(type(soup.head.title))
print(soup.head.title.string)
```

运行结果：

```
<title>The Dormouse's story</title>
<class 'bs4.element.Tag'>
The Dormouse's story
```

第一行结果是我们调用了 head 之后再次调用了 title 来选择的 title 节点元素，然后我们紧接着打印输出了它的类型，可以看到它仍然是 bs4.element.Tag 类型，也就是说我们在 Tag 类型的基础上再次选择得到的依然还是 Tag 类型，每次返回的结果都相同，所以这样我们就可以这样做嵌套的选择了。

***关联选择***

我们在做选择的时候有时候不能做到一步就可以选择到想要的节点元素，有时候在选择的时候需要先选中某一个节点元素，然后以它为基准再选择它的子节点、父节点、兄弟节点等等。所以在这里我们就介绍下如何来选择这些节点元素。

**子节点和子孙节点**

选取到了一个节点元素之后，如果想要获取它的直接子节点可以调用 contents 属性，我们用一个实例来感受一下：

```
html = """
<html>
    <head>
        <title>The Dormouse's story</title>
    </head>
    <body>
        <p class="story">
            Once upon a time there were three little sisters; and their names were
            <a href="http://example.com/elsie" class="sister" id="link1">
                <span>Elsie</span>
            </a>
            <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
            and
            <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>
            and they lived at the bottom of a well.
        </p>
        <p class="story">...</p>
"""
```

运行结果：

```
['\n            Once upon a time there were three little sisters; and their names were\n            ', <a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>, '\n', <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, ' \n            and\n            ', <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>, '\n            and they lived at the bottom of a well.\n        ']
```

返回的结果是列表形式，p 节点里面既包含文本，又包含节点，返回的结果会将他们以列表形式都统一返回。注意得到的列表的每一个元素都是 p 节点的直接子节点，比如第一个 a 节点里面包含了一层 span 节点，这个就相当于孙子节点了，但是返回结果中并没有单独把 span 节点选出来作为结果的一部分，所以说 contents 属性得到的结果是直接子节点的列表。

同样地我们可以调用 children 属性，得到相应的结果：

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.p.children)
for i, child in enumerate(soup.p.children):
    print(i, child)
```

运行结果：

```
<list_iterator object at 0x1064f7dd8>
0
            Once upon a time there were three little sisters; and their names were

1 <a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
2

3 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
4  
            and

5 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
6
            and they lived at the bottom of a well.
```

还是同样的 HTML 文本，在这里我们调用了 children 属性来进行选择，返回结果可以看到是生成器类型，所以接下来我们用 for 循环输出了一下相应的内容，内容其实是一样的，只不过 children 返回的是生成器类型，而 contents 返回的是列表类型。
如果我们要得到所有的子孙节点的话可以调用 descendants 属性：

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.p.descendants)
for i, child in enumerate(soup.p.descendants):
    print(i, child)
```

运行结果：

```
<generator object descendants at 0x10650e678>
0
            Once upon a time there were three little sisters; and their names were

1 <a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
2

3 <span>Elsie</span>
4 Elsie
5

6

7 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
8 Lacie
9  
            and

10 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
11 Tillie
12
            and they lived at the bottom of a well.
```

返回结果还是生成器，遍历输出一下可以看到这次的输出结果就包含了 span 节点，descendants 会递归地查询所有子节点，得到的是所有的子孙节点。

**父节点和祖先节点**

如果要获取某个节点元素的父节点，可以调用 parent 属性：

```
html = """
<html>
    <head>
        <title>The Dormouse's story</title>
    </head>
    <body>
        <p class="story">
            Once upon a time there were three little sisters; and their names were
            <a href="http://example.com/elsie" class="sister" id="link1">
                <span>Elsie</span>
            </a>
        </p>
        <p class="story">...</p>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.a.parent)
```

运行结果：

```
<p class="story">
            Once upon a time there were three little sisters; and their names were
            <a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
</p>
```

在这里我们选择的是第一个 a 节点的父节点元素，很明显它的父节点是 p 节点，输出结果便是 p 节点及其内部的内容。
注意到这里输出的仅仅是 a 节点的直接父节点，而没有再向外寻找父节点的祖先节点，如果我们要想获取所有的祖先节点，可以调用 parents 属性：

```
html = """
<html>
    <body>
        <p class="story">
            <a href="http://example.com/elsie" class="sister" id="link1">
                <span>Elsie</span>
            </a>
        </p>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(type(soup.a.parents))
print(list(enumerate(soup.a.parents)))
```

运行结果：

```
<class 'generator'>
[(0, <p class="story">
<a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
</p>), (1, <body>
<p class="story">
<a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
</p>
</body>), (2, <html>
<body>
<p class="story">
<a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
</p>
</body></html>), (3, <html>
<body>
<p class="story">
<a class="sister" href="http://example.com/elsie" id="link1">
<span>Elsie</span>
</a>
</p>
</body></html>)]
```

返回结果是一个生成器类型，我们在这里用列表输出了它的索引和内容，可以发现列表中的元素就是 a 节点的祖先节点。

**兄弟节点**

上面说明了子节点和父节点的获取方式，如果要获取同级的节点也就是兄弟节点应该怎么办？我们先用一个实例来感受一下：

```
html = """
<html>
    <body>
        <p class="story">
            Once upon a time there were three little sisters; and their names were
            <a href="http://example.com/elsie" class="sister" id="link1">
                <span>Elsie</span>
            </a>
            Hello
            <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
            and
            <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>
            and they lived at the bottom of a well.
        </p>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print('Next Sibling', soup.a.next_sibling)
print('Prev Sibling', soup.a.previous_sibling)
print('Next Siblings', list(enumerate(soup.a.next_siblings)))
print('Prev Siblings', list(enumerate(soup.a.previous_siblings)))
```

运行结果：

```
Next Sibling
            Hello

Prev Sibling
            Once upon a time there were three little sisters; and their names were

Next Siblings [(0, '\n            Hello\n            '), (1, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>), (2, ' \n            and\n            '), (3, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>), (4, '\n            and they lived at the bottom of a well.\n        ')]
Prev Siblings [(0, '\n            Once upon a time there were three little sisters; and their names were\n            ')]
```

可以看到在这里我们调用了四个不同的属性，next_sibling 和 previous_sibling 分别可以获取节点的下一个和上一个兄弟元素，next_siblings 和 previous_siblings 则分别返回所有前面和后面的兄弟节点的生成器。

**提取信息**

在上面我们讲解了关联元素节点的选择方法，如果我们想要获取它们的一些信息，比如文本、属性等等也是同样的方法。

```
html = """
<html>
    <body>
        <p class="story">
            Once upon a time there were three little sisters; and their names were
            <a href="http://example.com/elsie" class="sister" id="link1">Bob</a><a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
        </p>
"""
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print('Next Sibling:')
print(type(soup.a.next_sibling))
print(soup.a.next_sibling)
print(soup.a.next_sibling.string)
print('Parent:')
print(type(soup.a.parents))
print(list(soup.a.parents)[0])
print(list(soup.a.parents)[0].attrs['class'])
```

运行结果：

```
Next Sibling:
<class 'bs4.element.Tag'>
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
Lacie
Parent:
<class 'generator'>
<p class="story">
            Once upon a time there were three little sisters; and their names were
            <a class="sister" href="http://example.com/elsie" id="link1">Bob</a><a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
</p>
['story']
```

如果返回结果是单个节点，那么可以直接调用 string、attrs 等属性来获得其文本和属性，如果返回结果是多个节点的生成器，则可以转为列表后取出某个元素，然后再调用 string、attrs 等属性来获取其对应节点等文本和属性。

### 5. 方法选择器
上文的选择方法都是通过属性来选择元素的，这种选择方法非常快，但是如果要进行比较复杂的选择的话则会比较繁琐，不够灵活。所以 BeautifulSoup 还为我们提供了一些查询的方法，比如 find_all()、find() 等方法，我们可以调用方法然后传入相应等参数就可以灵活地进行查询了。

**find_all()**

find_all，顾名思义，就是查询所有符合条件的元素，可以给它传入一些属性或文本来得到符合条件的元素，功能十分强大。其API如下：

`find_all(name , attrs , recursive , text , **kwargs)`

**name**

我们可以根据节点名来查询元素，下面我们用一个实例来感受一下：

```
html='''
<div class="panel">
    <div class="panel-heading">
        <h4>Hello</h4>
    </div>
    <div class="panel-body">
        <ul class="list" id="list-1">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
            <li class="element">Jay</li>
        </ul>
        <ul class="list list-small" id="list-2">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
        </ul>
    </div>
</div>
'''
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.find_all(name='ul'))
print(type(soup.find_all(name='ul')[0]))
```

运行结果：

```
[<ul class="list" id="list-1">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>, <ul class="list list-small" id="list-2">
<li class="element">Foo</li>
<li class="element">Bar</li>
</ul>]
<class 'bs4.element.Tag'>
```

在这里我们调用了 find_all() 方法，传入了一个 name 参数，参数值为 ul，也就是说我们想要查询所有 ul 节点，返回结果是列表类型，长度为 2，每个元素依然都是 bs4.element.Tag 类型。

因为都是 Tag 类型，所以我们依然可以进行嵌套查询，还是同样的文本，在这里我们查询出所有 ul 节点后再继续查询其内部的 li 节点。

```
for ul in soup.find_all(name='ul'):
    print(ul.find_all(name='li'))
```

运行结果：

```
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>]
[<li class="element">Foo</li>, <li class="element">Bar</li>]
```

返回结果是列表类型，列表中的每个元素依然还是 Tag 类型。接下来我们就可以遍历每个 li 获取它的文本了。

```
for ul in soup.find_all(name='ul'):
    print(ul.find_all(name='li'))
    for li in ul.find_all(name='li'):
        print(li.string)
```

运行结果：
```
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>]
Foo
Bar
Jay
[<li class="element">Foo</li>, <li class="element">Bar</li>]
Foo
Bar
```

*attrs*

除了根据节点名查询，也可以传入一些属性来进行查询，我们用一个实例感受一下：

```
html='''
<div class="panel">
    <div class="panel-heading">
        <h4>Hello</h4>
    </div>
    <div class="panel-body">
        <ul class="list" id="list-1" name="elements">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
            <li class="element">Jay</li>
        </ul>
        <ul class="list list-small" id="list-2">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
        </ul>
    </div>
</div>
'''
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.find_all(attrs={'id': 'list-1'}))
print(soup.find_all(attrs={'name': 'elements'}))
```

运行结果：

```
[<ul class="list" id="list-1" name="elements">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>]
[<ul class="list" id="list-1" name="elements">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>]
```

在这里我们查询的时候传入的是 attrs 参数，参数的类型是字典类型，比如我们要查询 id 为 list-1 的节点，那就可以传入attrs={'id': 'list-1'} 的查询条件，得到的结果是列表形式，包含的内容就是符合 id 为 list-1 的所有节点，上面的例子中符合条件的元素个数是 1，所以结果是长度为 1 的列表。

对于一些常用的属性比如 id、class 等，我们可以不用 attrs 来传递，比如我们要查询 id 为 list-1 的节点，我们可以直接传入 id 这个参数，还是上面的文本，我们换一种方式来查询。

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.find_all(id='list-1'))
print(soup.find_all(class_='element'))
```

运行结果：

```
[<ul class="list" id="list-1">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>]
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>, <li class="element">Foo</li>, <li class="element">Bar</li>]
```

在这里我们直接传入 id='list-1' 就可以查询 id 为 list-1 的节点元素了。而对于 class 来说，由于 class 在 python 里是一个关键字，所以在这里后面需要加一个下划线，class_='element'，返回的结果依然还是 Tag 组成的列表。

***find()***

除了 find_all() 方法，还有 find() 方法，只不过 find() 方法返回的是单个元素，也就是第一个匹配的元素，而 find_all() 返回的是所有匹配的元素组成的列表。

```
html='''
<div class="panel">
    <div class="panel-heading">
        <h4>Hello</h4>
    </div>
    <div class="panel-body">
        <ul class="list" id="list-1">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
            <li class="element">Jay</li>
        </ul>
        <ul class="list list-small" id="list-2">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
        </ul>
    </div>
</div>
'''
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.find(name='ul'))
print(type(soup.find(name='ul')))
print(soup.find(class_='list'))
```

运行结果：

```
<ul class="list" id="list-1">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>
<class 'bs4.element.Tag'>
<ul class="list" id="list-1">
<li class="element">Foo</li>
<li class="element">Bar</li>
<li class="element">Jay</li>
</ul>
```

返回结果不再是列表形式，而是第一个匹配的节点元素，类型依然是 Tag 类型。
另外还有许多的查询方法，用法与前面介绍的 find_all()、find() 方法完全相同，只不过查询范围不同，在此做一下简单的说明。

***find_parents()*** ***find_parent()***

find_parents() 返回所有祖先节点，find_parent() 返回直接父节点。

***find_next_siblings()*** ***find_next_sibling()***

find_next_siblings() 返回后面所有兄弟节点，find_next_sibling() 返回后面第一个兄弟节点。

***find_previous_siblings()*** ***find_previous_sibling()***

find_previous_siblings() 返回前面所有兄弟节点，find_previous_sibling() 返回前面第一个兄弟节点。

***find_all_next()***  ***find_next()***

find_all_next() 返回节点后所有符合条件的节点, find_next() 返回第一个符合条件的节点。

***find_all_previous()*** ***find_previous()***

find_all_previous() 返回节点后所有符合条件的节点, find_previous() 返回第一个符合条件的节点。

### 6. CSS选择器
BeautifulSoup 还提供了另外一种选择器，那就是 CSS 选择器，如果对 Web 开发熟悉对话，CSS 选择器肯定也不陌生。使用 CSS 选择器，只需要调用 select() 方法，传入相应的 CSS 选择器即可，我们用一个实例来感受一下：

```
html='''
<div class="panel">
    <div class="panel-heading">
        <h4>Hello</h4>
    </div>
    <div class="panel-body">
        <ul class="list" id="list-1">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
            <li class="element">Jay</li>
        </ul>
        <ul class="list list-small" id="list-2">
            <li class="element">Foo</li>
            <li class="element">Bar</li>
        </ul>
    </div>
</div>
'''
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.select('.panel .panel-heading'))
print(soup.select('ul li'))
print(soup.select('#list-2 .element'))
print(type(soup.select('ul')[0]))
```

运行结果：

```
[<div class="panel-heading">
<h4>Hello</h4>
</div>]
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>, <li class="element">Foo</li>, <li class="element">Bar</li>]
[<li class="element">Foo</li>, <li class="element">Bar</li>]
<class 'bs4.element.Tag'>
```

在这里我们用了三次 CSS 选择器，返回的结果均是符合 CSS 选择器的节点组成的列表。例如 select('ul li') 则是选择所有 ul 节点下面的所有 li 节点，结果便是所有的 li 节点组成的列表。最后一句我们打印输出了列表中元素的类型，可以看到类型依然是 Tag 类型。

***嵌套选择***

select() 方法同样支持嵌套选择，例如我们先选择所有 ul 节点，再遍历每个 ul 节点选择其 li 节点，样例如下：
```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
for ul in soup.select('ul'):
    print(ul.select('li'))
```
运行结果：

```
[<li class="element">Foo</li>, <li class="element">Bar</li>, <li class="element">Jay</li>]
[<li class="element">Foo</li>, <li class="element">Bar</li>]
```

可以看到正常输出了遍历每个 ul 节点之后，其下的所有 li 节点组成的列表。

***获取属性***

我们知道节点类型是 Tag 类型，所以获取属性还是可以用原来的方法获取，仍然是上面的 HTML 文本，我们在这里尝试获取每个 ul 节点的 id 属性。

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
for ul in soup.select('ul'):
    print(ul['id'])
    print(ul.attrs['id'])
```

运行结果：

```
list-1
list-1
list-2
list-2
```

可以看到直接传入中括号和属性名和通过 attrs 属性获取属性值都是可以成功的。

***获取文本***

那么获取文本当然也可以用前面所讲的 string 属性，还有一个方法那就是 get_text()，同样可以获取文本值。

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
for li in soup.select('li'):
    print('Get Text:', li.get_text())
    print('String:', li.string)
```

运行结果：

```
Get Text: Foo
String: Foo
Get Text: Bar
String: Bar
Get Text: Jay
String: Jay
Get Text: Foo
String: Foo
Get Text: Bar
String: Bar
```

二者的效果完全一致，都可以获取到节点的文本值。

### 7. 小结
最后做一下简单的总结：

 1. 推荐使用 LXML 解析库，必要时使用 html.parser。
 2. 节点选择筛选功能弱但是速度快。
 3. 建议使用 find()、find_all() 查询匹配单个结果或者多个结果。
 4. 如果对 CSS 选择器熟悉的话可以使用 select() 选择法。