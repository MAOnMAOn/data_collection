## 8.1.1 什么是Ajax

Ajax，全称为 Asynchronous JavaScript and XML，即异步的 JavaScript 和 XML。
Ajax 不是一门编程语言，而是利用 JavaScript 在保证页面不被刷新、页面链接不改变的情况下与服务器交换数据并更新部分网页的技术。

对于传统的网页，如果想更新其内容，那么必须要刷新整个页面，但有了 Ajax，我们便可以实现在页面不被全部刷新的情况下更新其内容。在这个过程中，页面实际是在后台与服务器进行了数据交互，获取到数据之后，再利用 JavaScript 改变网页，这样网页内容就会更新了。

### 1. 相关实例
我们在浏览网页的时候会发现很多网页都有上滑查看更多的选项，比如我们以淘宝首页为例，切换到淘宝页面，快速下滑，可以发现下滑之后，再向下就没有了，转而会出现一个加载的动画，不一会儿下方就继续出现了新内容，那么这个过程其实就是 Ajax 加载的过程。

我们注意到页面其实并没有整个刷新，也就意味着页面的链接是没有变化的，但是这个过程网页中却又多了新的内容，也就是后面刷出来的新的内容。这就是通过 Ajax 获取新的数据并呈现而实现的过程。

***基本原理***

初步了解了 Ajax 之后我们再来详细了解一下它的基本原理，发送 Ajax 请求到网页更新的这个过程可以简单分为三步：
 - 发送请求
 - 渲染网页
 - 解析内容
下面我们分别来详细介绍一下这几个过程。

***发送请求***

我们知道 JavaScript 可以实现页面的各种交互功能，那么 Ajax 也不例外，它也是由 JavaScript 来实现的，实际它就是执行了类似如下的代码：

```
var xmlhttp;
if (window.XMLHttpRequest) {
    // code for IE7+, Firefox, Chrome, Opera, Safari
    xmlhttp=new XMLHttpRequest();
} else {// code for IE6, IE5
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
xmlhttp.onreadystatechange=function() {
    if (xmlhttp.readyState==4 && xmlhttp.status==200) {
        document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
}
xmlhttp.open("POST","/ajax/",true);
xmlhttp.send();
```

这是 JavaScript 对 Ajax 最底层的实现，实际上就是新建了 XMLHttpRequest 对象，然后调用了onreadystatechange 属性设置了监听，然后调用 open() 和 send() 方法向某个链接也就是服务器发送了一个请求，我们在前面用 Python 实现请求发送之后是可以得到响应结果的，只不过在这里请求的发送变成了 JavaScript 来完成，由于设置了监听，所以当服务器返回响应时，onreadystatechange 对应的方法便会被触发，然后在这个方法里面解析响应内容即可。

***解析内容***

得到响应之后，onreadystatechange 属性对应的方法便会被触发，此时利用 xmlhttp 的 responseText 属性便可以取到响应的内容。这也就是类似于 Python 中利用 Requests 向服务器发起了一个请求，然后得到响应的过程。那么返回内容可能是 HTML，可能是 Json，接下来只需要在方法中用 JavaScript 进一步处理即可。比如如果是 Json 的话，可以进行解析和转化。

***渲染网页***

JavaScript 有改变网页内容的能力，解析完响应内容之后，就可以调用 JavaScript 来针对解析完的内容对网页进行下一步的处理了。比如通过 document.getElementById().innerHTML 这样的操作便可以对某个元素内的源代码进行更改，这样网页显示的内容就改变了，这样的操作也被称作 DOM 操作，即对 Document网页文档进行操作，如更改、删除等。

如上例中，document.getElementById("myDiv").innerHTML=xmlhttp.responseText 便将 ID 为 myDiv 的节点内部的 HTML 代码更改为服务器返回的内容，这样 myDiv 元素内部便会呈现出服务器返回的新数据，网页的部分内容看上去就更新了。
以上就是Ajax的三个步骤。

我们观察到，以上的步骤其实都是由 JavaScript 来完成的，它完成了整个请求、解析、渲染的过程。所以再回想淘宝的下拉刷新，这其实就是 JavaScript 向服务器发送了一个 Ajax 请求，然后获取新的淘宝页面数据，将其解析，并将其渲染在网页中。

### 2. Ajax分析方法
在这里我们还是需要借助于浏览器的开发者工具，我们以 Chrome 浏览器为例来看一下怎样操作。我们在天猫中输入关键字 sql 打开链接：https://list.tmall.com/search_product.htm?q=sql ，随后在页面中按下F12 按键，开始使用开发者工具，这里我们切换到 Network 选项卡，随后重新刷新页面，可以发现在这里出现了非常多的条目。

当我们向下滑动页面，其实就是在页面加载过程中浏览器与服务器之间发送 Request 和接收 Response 的所有记录。

Ajax其实有其特殊的请求类型，它叫做 xhr，network中我们可以发现一个名称为 getIndex 开头的请求，其 Type 为 xhr，这就是一个 Ajax 请求，鼠标点击这个请求，可以查看这个请求的详细信息，我们在右侧可以观察到其 Request Headers、URL 和 Response Headers 等信息，其中 Request Headers 中有一个信息为 X-Requested-With:XMLHttpRequest，这就标记了此请求是 Ajax 请求。

随后我们点击一下 Preview，即可看到响应的内容，响应内容是 Json 格式，在这里 Chrome 为我们自动做了解析，我们可以点击箭头来展开和收起相应内容。

接下来我们再利用 Chrome 开发者工具的筛选功能筛选出所有的 Ajax 请求，在请求的上方有一层筛选栏，我们可以点击 XHR，这样在下方显示的所有请求便都是 Ajax 请求了。

再接下来我们我们不断滑动页面，可以看到在页面底部有一条条新的商品条目被刷出，而开发者工具下方也一个个地出现 Ajax 请求，这样我们就可以捕获到所有的 Ajax 请求了。
随意点开一个条目都可以清楚地看到其 Request URL、Request Headers、Response Headers、Response Body等内容，想要模拟请求和提取就非常简单了。