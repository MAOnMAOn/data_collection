## 4.4 对抗反爬虫

很多爬虫，在一开始都能正常运行，不过有时候，你可能会很不幸地发现爬虫报错了，当我们确认网页结构没有发生变动，网络连接正常时，程序发生错误就很可能是因为我们被 ban 了，那怎么发现自己可能被网站识别了？

当你在页面看到如下关键字时，爬虫基本上就已经暴露了：

```
CAPTCHA pages
Unusual content delivery delay
Frequent response with HTTP 404, 301 or 50x errors
301 Moved Temporarily
401 Unauthorized
403 Forbidden
404 Not Found
408 Request Timeout
429 Too Many Requests
503 Service Unavailable
```
所以，如何构建一个健壮的爬虫，就显得尤为重要了。

### 1. IP/网络层 
大多数网站都是针对单一ip的异常行为，来采取反爬虫措施，所以我们可以通过改变ip来规避反爬，一般而言，利用代理服务器(HTTP/SOCKET)进行源IP更改，这是最简单易行的方式。

当然，除了使用网络代理，我们还可以自己实现动态ip切换：

1. 模拟路由器登录
2. 通过postman 查找路由器的断线、重连接口
3. 当发现ip被主机识别以后，强制路由器断线、重连


***路由器登录***

1. 普通表单：直接通过表单发送 FORM 请求

2. 401 Unauthorized：
>没有登录的情况下，服务器返回401，并在Header 之中设置 WWW-Authenticate: Basic realm=“User Visible Realm”，客户端收到 401 同时检查到。
Header里包含了以上信息的情况下，会自动弹出登录框要求用户登录

![](/assets/路由器登录验证.png)

***2种实现自动登录的方法***
* url：usename:password@ip
   >如：admin：rootadmin@192.168.1.1
  
* set Authorization field in Header:
>Key: Authorization
Value: ‘Basic ‘ + base64.b64encode(‘username:password’)
Authorization: Basic YWRtaW46cm9vdGFkbWlu

***路由器重连***

![](/assets/路由器重连.png)

<br />

### 2. 合理设置 HTTP 请求头内容
#### b. CSS 相关属性检测

***nofollow 属性***

```
"Nofollow" provides a way for webmasters to tell search engines "Don't
follow links on this page" or "Don't follow this specific link."

Originally, the nofollow attribute appeared in the page-level meta tag,
and instructed search engines not to follow (i.e., crawl) any outgoing
links on the page. For example:
```

`<meta name="robots" content="nofollow" />`

***css 的 display 属性***

当 CSS 之中有 display:none 的属性设置时，表示隐藏当前块。表名网站在编写前端页面时有意识地进行了反爬处理：

```
The display CSS property specifies the type of rendering box used for an
element. In HTML, default display property values are taken from behaviors
described in the HTML specifications or from the browser/user default
stylesheet. The default value in XML is inline
```
通过下面的js代码，我们可以检测display属性：

```
document.getElementById('id').style.display
document.getElementsByClassName(’classname')[0].style.display
```

***好的规避反爬虫检查的方法***

1. 多主机策略(多ip)

2. 爬的慢一点，避免攻击主机，如果发现被阻止，立即降低访问频次，设计得smart一点，来找到访问频次限制的临界点

3. 通过变换ip 或者 代理服务器 来隐蔽

4. 把爬虫放到频繁访问的主站ip的子网下，比如教育网

5. 频繁改变自己的 UserAgent

6. 探测陷阱， 比如nofollow 的tag,或者 display：none 的 CSS

7. 如果使用了规则来批量爬取，需要对规则进行组合

8. 如果有可能， 按照 Robots.txt 定义的行为进行抓取