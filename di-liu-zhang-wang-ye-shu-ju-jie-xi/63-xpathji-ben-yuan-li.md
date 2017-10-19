## 6.3 Xpath的基本原理

XPath，全称 XML Path Language，即 XML 路径语言，它是一门在XML文档中查找信息的语言。XPath 最初设计是用来搜寻XML文档的，但是它同样适用于 HTML 文档的搜索。所以在做爬虫时，我们完全可以使用 XPath 来做相应的信息抽取。

### 1. XPath概览
XPath 的选择功能十分强大，它提供了非常简洁明了的路径选择表达式，另外它还提供了超过 100 个内建函数用于字符串、数值、时间的匹配以及节点、序列的处理等等，几乎所有我们想要定位的节点都可以用XPath来选择。
XPath语法参考文档：http://www.w3school.com.cn/xpath/index.asp

### 2. XPath开发工具

 1. 开源的XPath表达式编辑工具:XMLQuire(XML格式文件可用)
 2. chrome插件 XPath Helper
 3. firefox插件 XPath Checker

### 3. XPath语法
XPath 是一门在 XML 文档中查找信息的语言，可用来在 XML 文档中对元素和属性进行遍历。

```
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="eng">Harry Potter</title>
  <price>29.99</price>
</book>

<book>
  <title lang="eng">Learning XML</title>
  <price>39.95</price>
</book>

</bookstore>
```

选取节点 XPath 使用路径表达式在 XML 文档中选取节点。节点是通过沿着路径或者 step 来选取的。下面列出了最有用的路径表达式：

表达式  |	描述
--------|-----------------------------
/	    | 从根节点选取。
nodename| 选取此节点的所有子节点。
//	    | 从当前节点 选择 所有匹配文档中的节点
.	    | 选取当前节点。
..	    |选取当前节点的父节点。
@	    | 选取属性。

***部分实例***

在下面的表格中，我们已列出了一些路径表达式以及表达式的结果：

路径表达式	| 结果
------------|------------------------------------------
/bookstore  | 选取根元素 bookstore。假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！
bookstore	|选取 bookstore 元素的所有子节点。默认从根节点选取
bookstore/book|	选取属于 bookstore 的子元素的所有 book 元素。
//book	    |选取所有 book 子元素，而不管它们在文档中的位置。
//book/./title|	选取所有 book 子元素，从当前节点查找title节点
//price/..	|选取所有 book 子元素，从当前节点查找父节点
bookstore//book	|选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。
//@lang	|选取名为 lang 的所有属性。

***谓语条件（Predicates）***

 1. 谓语用来查找某个特定的信息或者包含某个指定的值的节点。
 2. 所谓"谓语条件"，就是对路径表达式的附加条件。
 3. 谓语是被嵌在方括号中，都写在方括号"[]"中，表示对节点进行进一步的筛选。

***相关实例***

在下面的表格中，我们列出了带有谓语的一些路径表达式，以及表达式的结果：

路径表达式	       |    结果
-------------------|--------------------------------------------------
/bookstore/book[1] |	选取属于 bookstore 子元素的第一个 book 元素。
/bookstore/book[last()] | 选取属于 bookstore 子元素的最后一个 book 元素。
/bookstore/book[last()-1] |	选取属于 bookstore 子元素的倒数第二个 book 元素。
/bookstore/book[position()<3] |	选取最前面的两个属于 bookstore 元素的子元素的 book 元素。
//title[@lang] | 选取所有拥有名为 lang 的属性的 title 元素。
//title[@lang=’eng’] | 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。
//book[price] |	选取所有 book 元素，且被选中的book元素必须带有price子元素
/bookstore/book[price>35.00] | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。
/bookstore/book[price>35.00]/title | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。

***选取未知节点***

 XPath 通配符可用来选取未知的 XML 元素。

通配符 | 描述
-------|------------
*      |	匹配任何元素节点。
@*     |	匹配任何属性节点。

***实例***

在下面的表格中，我们列出了一些路径表达式，以及这些表达式的结果：

路径表达式  |	结果
------------|---------------------------------------
/bookstore/*| 选取 bookstore 元素的所有子元素。
//*	        | 选取文档中的所有元素。
//title[@*]	| 选取所有带有属性的 title 元素。

***选取若干路径***

通过在路径表达式中使用“|”运算符，您可以选取若干个路径。

***例子***

在下面的表格中，我们列出了一些路径表达式，以及这些表达式的结果：

路径表达式  |	结果
------------|-------------------------
//book/title \| //book/price| 选取 book 元素的所有 title 和 price 元素。
//title \| //price |	选取文档中的所有 title 和 price 元素。
/bookstore/book/title \| //price  |	选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。


### 4. XPath 高级用法

***模糊查询 contains***

目前许多web框架，都是动态生成界面的元素id，因此在每次操作相同界面时，ID都是变化的，这样为自动化测试造成了一定的影响。

```
<div class="eleWrapper" title="请输入用户名">
<input type="text" class="textfield" name="ID9sLJQnkQyLGLhYShhlJ6gPzHLgvhpKpLzp2Tyh4hyb1b4pnvzxFR!-166749344!1357374592067" id="nt1357374592068"  />
</div>
```

解决方法 使用xpath的匹配功能，//input[contains(@id,'nt')]

***测试使用的XML***

```
<Root>

<Person ID="1001" >

<Name lang="zh-cn" >张城斌</Name>

<Email xmlns="www.quicklearn.cn" > cbcye@live.com </Email>

<Blog>http://cbcye.cnblogs.com</Blog>

</Person>

<Person ID="1002" >

<Name lang="en" >Gary Zhang</Name>

<Email xmlns="www.quicklearn.cn" > GaryZhang@cbcye.com</Email>

<Blog>http://www.quicklearn.cn</Blog>

</Person>

</Root>
```

查询所有Blog节点值中带有 cn 字符串的Person节点
1.Xpath表达式：
`/Root//Person[contains(Blog,'cn')]`
2.查询所有Blog节点值中带有 cn 字符串并且属性ID值中有01的Person节点
Xpath表达式：
`/Root//Person[contains(Blog,'cn') and contains(@ID,'01')]`