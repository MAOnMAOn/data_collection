## scrapy 分布式原理

scrapy 默认并不支持分布式，在过去，使用 scrapy 进行大规模爬取工作，一般需要提取做好网页地址的分割与分发，实现一种近似分布式。

现在，基于scrapy 的分布式框架越来越多，比较著名的有 scrapy-redis 以及 scrapy-cluster。在平时的工作之中，也是使用较为广的分布式框架。现在，我们以scrapy-redis 为例，进行 scrapy 分布式的介绍。

### 1. scrapy 单机架构
在了解 scrapy-redis 之前我们需要简单地回顾一下 scrapy 原生的架构。我们知道，scrapy 原生的任务调度基于文件系统，通过 JOBDIR 指定任务信息存储的路径。同时 scrapy 还会在本机维护一个爬取队列，用来调度爬取中的相关任务。

![](/assets/scrapyshecle.png)

以上是 scrapy 的单主机架构图，其各个组件主要有以下几个功能：

#### Scrapy Engine
引擎负责控制数据流在系统中各组件之间的流动，并在相应动作发生时触发事件。

#### Scheduler
调度器负责从引擎接收 request，并将他们入队，以便之后引擎请求的时候，把 request 返还回去。

#### Spiders
Spider 是 scrapy用户编写的用于分析 response 并提取 item，并获取额外跟进 url 的类。每个 spider 负责处理一个特定的网站。

#### Item Pipeline
Item Pipeline 负责处理被 spider 提取出来的 item。典型的处理有清理、验证以及持久化

#### Data flow
scrapy 之中的数据流由执行引擎控制，其过程如下：

* 引擎打开一个网站，找到该网站的 spider，并向该 spider 请求第一个要爬取的链接。
* 引擎从 spider 中获取第一个要爬取的 url 并在调度器以 request　方式调度。
* 引擎向调度器请求下一个要爬取的 url。
* 调度器返回下一个要爬取的 url 给引擎，引擎将 url 通过下载器中间件(请求 request 方向) 
* 
* 
* 