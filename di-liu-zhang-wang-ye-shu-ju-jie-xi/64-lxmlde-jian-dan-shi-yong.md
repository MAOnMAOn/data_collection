## 6.4 LXML的简单使用

### 1. 环境准备
在通过 lxml 进行网页解析之前，请确保 lxml 库及其依赖已经正确安装。

### 2. 实例引入
我们现用一个实例来感受一下使用 XPath 来对网页进行解析的过程，代码如下：

```
from lxml import etree
text = '''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </ul>
 </div>
'''
html = etree.HTML(text)
result = etree.tostring(html)
print(result.decode('utf-8'))
```

在这里我们首先导入了 LXML 库的 etree 模块，然后声明了一段 HTML 文本，调用 HTML 类进行初始化，这样我们就成功构造了一个 XPath 解析对象，在这里注意到 HTML 文本中的最后一个 li 节点是没有闭合的，但是 etree 模块可以对 HTML 文本进行自动修正。

调用 tostring() 方法即可输出修正后的 HTML 代码，但是结果是 bytes 类型，在这里我们利用 decode() 方法转成 str 类型，结果如下：

```
<html><body><div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </li></ul>
 </div>
</body></html>
```

可以看到经过处理之后 li 节点标签被补全，并且还自动添加了 body、html 节点。

另外我们也可以直接读取文本文件进行解析，示例如下：
```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = etree.tostring(html)
print(result.decode('utf-8'))
```

其中 test.html 的内容就是上面例子中的 HTML 代码，内容如下：

```
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </ul>
 </div>
```

### 3. 所有节点
我们一般会用 // 开头的 XPath 规则来选取所有符合要求的节点，以上文的 HTML 文本为例，如果我们要选取所有节点，可以这样实现：

```
from lxml import etree
html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//*')
print(result)
```

我们在这里使用 * 代表匹配所有节点，也就是整个 HTML 文本中的所有节点都会被获取，可以看到返回形式是一个列表，每个元素是 Element 类型，其后跟了节点的名称，如 html、body、div、ul、li、a 等等，所有的节点都包含在列表中了。

当然此处匹配也可以指定节点名称，如果我们想获取所有 li 节点，示例如下：

```
from lxml import etree
html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li')
print(result)
print(result[0])
```

在这里我们要选取所有 li 节点可以使用 //，然后直接加上节点的名称即可，调用时直接调用 xpath() 方法即可提取。
运行结果：

```
[<Element li at 0x105849208>, <Element li at 0x105849248>, <Element li at 0x105849288>, <Element li at 0x1058492c8>, <Element li at 0x105849308>]
<Element li at 0x105849208>
```

在这里我们可以看到提取结果是一个列表形式，其每一个元素都是一个 Element 对象，如果要取出其中一个对象可以直接用中括号加索引即可取出，如 [0]。

### 4. 子节点
我们通过 / 或 // 即可查找元素的子节点或子孙节点，加入我们现在想选择 li 节点所有直接 a 子节点，可以这样来实现：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li/a')
print(result)
```

在这里我们通过追加一个 /a 即选择了所有 li 节点的所有直接 a 子节点，因为 //li 是选中所有li节点， /a 是选中li节点的所有直接子节点 a，二者组合在一起即获取了所有li节点的所有直接 a 子节点。

但是此处的 / 是选取直接子节点，如果我们要获取所有子孙节点就该使用 // 了，例如我们要获取 ul 节点下的所有子孙 a 节点，可以这样来实现：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//ul//a')
print(result)
```

运行结果是相同的。
但是这里如果我们用 //ul/a 就无法获取任何结果了，因为 / 是获取直接子节点，而在 ul 节点下没有直接的 a 子节点，只有 li 节点，所以无法获取任何匹配结果，代码如下：
```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//ul/a')
print(result)
```

运行结果：
`[]`

因此在这里我们要注意 / 和 // 的区别，/ 是获取直接子节点，// 是获取子孙节点。

### 5. 父节点
我们知道通过连续的 / 或 // 可以查找子节点或子孙节点，那假如我们知道了子节点怎样来查找父节点呢？在这里我们可以用 .. 来获取父节点。

比如我们现在首先选中 href 是 link4.html 的 a 节点，然后再获取其父节点，然后再获取其 class 属性，代码如下：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//a[@href="link4.html"]/../@class')
print(result)
```

运行结果：

`['item-1']`

检查一下结果，正是我们获取的目标 li 节点的 class，获取父节点成功。
同时我们也可以通过 parent:: 来获取父节点，代码如下：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//a[@href="link4.html"]/parent::*/@class')
print(result)
```

### 6. 属性匹配
在选取的时候我们还可以用 @ 符号进行属性过滤，比如在这里如果我们要选取 class 为 item-1 的 li 节点，可以这样实现:

```
from lxml import etree
html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]')
print(result)
```

在这里我们通过加入 [@class="item-0"] 就限制了节点的 class 属性为 item-0，而 HTML 文本中符合条件的 li 节点有两个，所以返回结果应该返回两个匹配到的元素，结果如下：

`[<Element li at 0x10a399288>, <Element li at 0x10a3992c8>]`

### 7. 文本获取
我们用 XPath 中的 text() 方法可以获取节点中的文本，我们接下来尝试获取一下上文 li 节点中的文本，代码如下：
```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]/text()')
print(result)
```
运行结果如下：
['\n     ']
很奇怪的是我们并没有获取到任何文本，而是只获取到了一个换行符，这是为什么呢？因为 XPath 中 text() 前面是 /，而此 / 的含义是选取直接子节点，而此处很明显 li 的直接子节点都是 a 节点，文本都是在 a 节点内部的，所以这里匹配到的结果就是被修正的 li 节点内部的换行符，因为自动修正的li节点的尾标签换行了。

其中一个节点因为自动修正，li 节点的尾标签添加的时候换行了，所以提取文本得到的唯一结果就是 li 节点的尾标签和 a 节点的尾标签之间的换行符。
因此，如果我们想获取 li 节点内部的文本就有两种方式，一种是选取到 a 节点再获取文本，另一种就是使用 //，我们来看下二者的区别是什么。
首先我们选取到 a 节点再获取文本，代码如下：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]/a/text()')
print(result)
```

运行结果：

`['first item', 'fifth item']`

可以看到这里返回值是两个，内容都是属性为 item-0 的 li 节点的文本，这也印证了我们上文中属性匹配的结果是正确的。

在这里我们是逐层选取的，先选取了 li 节点，又利用 / 选取了其直接子节点 a，然后再选取其文本，得到的结果恰好是符合我们预期的两个结果。
我们再来看下用另一种方式 // 选取的结果，代码如下：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]//text()')
print(result)
```

运行结果：

`['first item', 'fifth item', '\n     ']`

不出所料，这里返回结果是三个，可想而知这里是选取所有子孙节点的文本，其中前两个就是 li 的子节点 a 节点内部的文本，另外一个就是最后一个 li 节点内部的文本，即换行符。

所以说，如果我们要想获取子孙节点内部的所有文本，可以直接用 // 加 text() 的方式获取，这样可以保证获取到最全面的文本信息，但是可能会夹杂一些换行符等特殊字符。如果我们想获取某些特定子孙节点下的所有文本，可以先选取到特定的子孙节点，然后再调用 text() 方法获取其内部文本，这样可以保证获取的结果是整洁的。

### 8. 属性获取
我们知道了用 text() 可以获取节点内部文本，那么节点属性该怎样获取呢？其实还是用 @ 符号就可以，例如我们想获取所有 li 节点下所有 a 节点的 href 属性，代码如下：

```
from lxml import etree

html = etree.parse('./test.html', etree.HTMLParser())
result = html.xpath('//li/a/@href')
print(result)
```

在这里我们通过 @href 即可获取节点的 href 属性，注意此处和属性匹配的方法不同，属性匹配是中括号加属性名和值来限定某个属性，如 [@href="link1.html"]，而此处的 @href 指的是获取节点的某个属性，二者需要做好区分。
运行结果：

`['link1.html', 'link2.html', 'link3.html', 'link4.html', 'link5.html']`

可以看到我们成功获取了所有 li 节点下的 a 节点的 href 属性，以列表形式返回。

### 9. 属性多值匹配
有时候某些节点的某个属性可能有多个值，例如下面例子：

```
from lxml import etree
text = '''
<li class="li li-first"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[@class="li"]/a/text()')
print(result)
```

在这里 HTML 文本中的 li 节点的 class 属性有两个值 li 和 li-first，但是此时若还想用之前的属性匹配获取就无法匹配了，代码运行结果：

`[]`

这时如果属性有多个值就需要用 contains() 函数了，代码可以改写如下：

```
from lxml import etree
text = '''
<li class="li li-first"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "li")]/a/text()')
print(result)
```

这样我们通过 contains() 方法，第一个参数传入属性名称，第二个参数传入属性值，这样只要此属性包含所传入的属性值就可以完成匹配了。运行结果：

`['first item']`

此种选择方式在某个节点的某个属性有多个值的时候经常会用到，如某个节点的 class 属性通常有多个。

### 10. 多属性匹配
另外我们可能还遇到一种情况，我们可能需要根据多个属性才能确定一个节点，这是就需要同时匹配多个属性才可以，那么这里可以使用运算符 and 来连接，示例如下：

```
from lxml import etree
text = '''
<li class="li li-first" name="item"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "li") and @name="item"]/a/text()')
print(result)
```

在这里 HTML 文本的 li 节点又增加了一个属性 name，这时候我们需要同时根据 class 和 name 属性来选择，就可以 and 运算符连接两个条件，两个条件都被中括号包围，运行结果如下：

`['first item']`

### 11. 按序选择
有时候我们在选择的时候可能某些属性同时匹配了多个节点，但是我们只想要其中的某个节点，如第二个节点，或者最后一个节点，这时可以利用中括号传入索引的方法获取特定次序的节点，示例如下：

```
from lxml import etree

text = '''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </ul>
 </div>
'''
html = etree.HTML(text)
result = html.xpath('//li[1]/a/text()')
print(result)
result = html.xpath('//li[last()]/a/text()')
print(result)
result = html.xpath('//li[position()<3]/a/text()')
print(result)
result = html.xpath('//li[last()-2]/a/text()')
print(result)
```

第一次选择我们选取了第一个 li 节点，中括号中传入数字1即可，注意这里和代码中不同，序号是以 1 开头的，不是 0 开头的。

第二次选择我们选取了最后一个 li 节点，中括号中传入 last() 即可，返回的便是最后一个 li 节点。

第三次选择我们选取了位置小于 3 的 li 节点，也就是位置序号为 1 和 2 的节点，得到的结果就是前 2 个 li 节点。

第四次选择我们选取了倒数第三个 li 节点，中括号中传入 last()-2即可，因为 last() 是最后一个，所以 last()-2 就是倒数第三个。
运行结果如下：

```
['first item']
['fifth item']
['first item', 'second item']
['third item']
```

在这里我们使用了 last()、position() 等函数，XPath 中提供了 100 多个函数，包括存取、数值、字符串、逻辑、节点、序列等处理功能，具体所有的函数作用可以参考：http://www.w3school.com.cn/xpath/xpath_functions.asp。

### 12. 节点轴选择
XPath 提供了很多节点轴选择方法，英文叫做 XPath Axes，包括获取子元素、兄弟元素、父元素、祖先元素等等，在一定情况下使用它可以方便地完成节点的选择，我们用一个实例来感受一下：

```
from lxml import etree

text = '''
<div>
    <ul>
         <li class="item-0"><a href="link1.html"><span>first item</span></a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </ul>
 </div>
'''
html = etree.HTML(text)
result = html.xpath('//li[1]/ancestor::*')
print(result)
result = html.xpath('//li[1]/ancestor::div')
print(result)
result = html.xpath('//li[1]/attribute::*')
print(result)
result = html.xpath('//li[1]/child::a[@href="link1.html"]')
print(result)
result = html.xpath('//li[1]/descendant::span')
print(result)
result = html.xpath('//li[1]/following::*[2]')
print(result)
result = html.xpath('//li[1]/following-sibling::*')
print(result)
```

运行结果：
```
[<Element html at 0x107941808>, <Element body at 0x1079418c8>, <Element div at 0x107941908>, <Element ul at 0x107941948>]
[<Element div at 0x107941908>]
['item-0']
[<Element a at 0x1079418c8>]
[<Element span at 0x107941948>]
[<Element a at 0x1079418c8>]
[<Element li at 0x107941948>, <Element li at 0x107941988>, <Element li at 0x1079419c8>, <Element li at 0x107941a08>]
```

第一次选择我们调用了 ancestor 轴，可以获取所有祖先节点，其后需要跟两个冒号，然后是节点的选择器，这里我们直接使用了 *，表示匹配所有节点，因此返回结果是第一个 li 节点的所有祖先节点，包括 html，body，div，ul。

第二次选择我们又加了限定条件，这次在冒号后面加了 div，这样得到的结果就只有 div 这个祖先节点了。

第三次选择我们调用了 attribute 轴，可以获取所有属性值，其后跟的选择器还是 *，这代表获取节点的所有属性，返回值就是 li 节点的所有属性值。

第四次选择我们调用了 child 轴，可以获取所有直接子节点，在这里我们又加了限定条件选取 href 属性为 link1.html 的 a 节点。

第五次选择调用了 descendant 轴，可以获取所有子孙节点，这里我们又加了限定条件获取 span 节点，所以返回的就是只包含 span 节点而没有 a 节点。

第六次选择我们调用了 following 轴，可以获取当前节点之后的所有节点，这里我们虽然使用的是 * 匹配，但又加了索引选择，所以只获取了第二个后续节点。

第七次选择我们调用了 following-sibling 轴，可以获取当前节点之后的所有同级节点，这里我们使用的是 * 匹配，所以获取了所有后续同级节点。
以上是XPath轴的简单用法，更多的轴的使用可以参考：http://www.w3school.com.cn/xpath/xpath_axes.asp