## 4.4 对抗反爬虫

爬虫与反爬虫之间的对抗，是所有爬虫工程师以及网站管理人员不得不面对的一个课题，作为爬虫的编写者，了解常见的反爬机制，是十分有必要的。现在，我们就从Web 服务器，网站反爬虫技术，爬取策略等几个方面展开，进行本节的介绍。

### 1. 服务器对 web 请求的处理
一般来说，基本的Web服务器请求有以下几个步骤：

1. 建立连接————接受客户端连接，若不希望与客户端进行连接，可以将其关闭。
2. 接收请求————从网络中读取一条 HTTP 请求报文。
3. 处理请求————解释请求报文，并采取行动。
4. 访问资源————访问报文中指定的资源。
5. 构建响应————创建带有正确首部的 HTTP响应报文。
6. 发送响应————将响应送给客户端
7. 记录事务处理过程————将已经完成事务有关的内容记录在一个日志文件中。

现在，我们以 Apache 网络服务器为例进行讲解：

![](/assets/服务器处理web请求.png)服务器处理 Web 请求流程

从上图我们可以看到，当客户端发起访问时，
 
1. 网络流量首先会到达防火墙，防火墙会对相关请求的对访问频次进行检查。

2. 通过网关进行端口映射后，请求相关数据到达对应的服务，例如 Apache

3. 在 Apache 之中，Apache 会通过 virtual host 查找根目录，并查找 .htaccess 伪静态设置，映射实际目录及文件

4. 执行脚本或提取文件

5. 服务器端确认 cookie 信息，查找用户，确认成功以后，进行用户权限检查

6. 执行命令，服务器返回数据(response)

### 2. 网站如何进行反爬
网站发现爬虫的手段很多，主要有以下几种：

1. 后台统计，查找非常规访问频次单一ip

2. 在后台统计中，查询单ip非常规的数据流量

3. 管理员对大量重复简单的网站浏览行为进行统计分析

4. 如果用户只下载网页，没有后续的js、css请求，那么很被判定为爬虫

5. 在前端页面设置陷阱来发现爬虫，例如通过一些css对用户隐藏的链接，只有爬虫可以访问

6. 其他
当网站(管理人员)发现异常访问，并判断为爬虫时，也有许多的反制措施，比如：

***通过 userAgent***

假设我们已经确定了2类爬虫的UserAgent,分别是 Nutch 和 Scrapy

```
UserAgent:Nutch
UserAgent:Scrapy
```

对应反爬虫的 .htaccess 文件定义：
```
BrowserMatchNoCase Nutch bad_bot
BrowserMatchNoCase Scrapy bad_bot
Order Deny,Allow
Deny from env=bad_bot
```
这里，Nutch 和 Scrapy 为禁止的key,bad_bot为定义的环境变量。

***启用 JavaScript***
大量使用动态页面，使得爬取难度增加，常规手段只能拿到一个基本的html页面，获取不到重要信息。

同时，即使爬虫采用了Web环境来渲染网页，也会大大增加爬虫的负担与爬取时间
同时，采用动态加载技术，对服务器的负担也会减轻。

***基于流量的拒绝***
在 Apache 之中开启带宽限制模块 LoadModule bw_module ，设置访问的最大带宽，比如每个ip最多3个连接，最大1MB/s，在 mod_bw.so 之中设置：

```
BandWidthModule On
ForceBandWidthModule On
BandWidth all 1024000
MinBandwidth all -l
MaxConnection all 3
```
```
#<Location/modbw>
#  SetHandler modbw-handler
#</Location>
```

***在iptables中进行控制***

```
syntax
/sbin/iptables -A INPUT -p tcp --syn --dport $port -m connlimit
--connlimit-above N -j REJECT --reject-with tcp-reset

Example: Limit HTTP Connections Per IP / Host
/sbin/iptables -A INPUT -p tcp --syn --dport 80 -m connlimit --
connlimit-above 20 -j REJECT --reject-with tcp-reset

Example: Limit HTTP Connections Per IP / Host Per Second
iptables -A INPUT -m state --state RELATED,ESTABLISHED -m
limit --limit 10/second --limit-burst 20 -j ACCEPT
```

当收到20个数据包后，触发访问频次限制，此时每秒最多10次连接请求，即单位时间为 100ms；

如果100ms内没有收到请求，系统触发的条件就+1，也就是说如果1s钟内停止发送数据，则再次建立10次连接之内都不会激活单位计数限制。

***投毒***
在不影响正常业务开展的前提下，网站可能会输出假数据，让爬虫知难而退。

1. 应对反爬虫
怎么发现自己可能被网站识别了？
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

动态ip切换技术
1. 模拟路由器登录
2. 通过postman 查找路由器的断线、重连接口
3. 当发现ip被主机识别以后，强制路由器断线、重连

路由器登录
1. 普通表单：直接通过表单发送 FORM 请求
2. 401 Unauthorized：
 没有登录的情况下，服务器返回401，并在Header 之中设置 WWW-Authenticate: Basic realm=“User Visible Realm”，客户端收到 401 同时检查到
Header里包含了以上信息的情况下，会自动弹出登录框要求用户登录

图片

2种实现自动登录的方法
1. url：usename:password@ip
  如：admin：rootadmin@192.168.1.1
2. set Authorization field in Header:
Key: Authorization
Value: ‘Basic ‘ + base64.b64encode(‘username:password’)
Authorization: Basic YWRtaW46cm9vdGFkbWlu

路由器重连
图片

nofollow 属性
"Nofollow" provides a way for webmasters to tell search engines "Don't
follow links on this page" or "Don't follow this specific link."

Originally, the nofollow attribute appeared in the page-level meta tag,
and instructed search engines not to follow (i.e., crawl) any outgoing
links on the page. For example:

<meta name="robots" content="nofollow" />

css 的 display 属性
The display CSS property specifies the type of rendering box used for an
element. In HTML, default display property values are taken from behaviors
described in the HTML specifications or from the browser/user default
stylesheet. The default value in XML is inline

display:none 隐藏当前块
检测display属性
document.getElementById('id').style.display
document.getElementsByClassName(’classname')[0].style.display

好的规避反爬虫检查的方法
1. 多主机策略
2. 爬的慢一点，避免攻击主机，如果发现被阻止，立即降低访问频次，设计得smart一点，来找到访问频次限制的临界点
3. 通过变换ip 或者 代理服务器 来隐蔽
4. 把爬虫放到频繁访问的主站ip的子网下，比如教育网
5. 频繁改变自己的 UserAgent
6. 探测陷阱， 比如nofollow 的tag,或者 display：none 的 CSS
7. 如果使用了规则来批量爬取，需要对规则进行组合
8. 如果有可能， 按照 Robots.txt 定义的行为进行抓取