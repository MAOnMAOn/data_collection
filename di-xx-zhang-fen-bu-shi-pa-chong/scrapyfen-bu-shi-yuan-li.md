## scrapy 分布式原理

scrapy 默认并不支持分布式，在过去，使用 scrapy 进行大规模爬取工作，一般需要提取做好网页地址的分割与分发，实现一种近似分布式。

现在，基于scrapy 的分布式框架越来越多，比较著名的有 scrapy-redis 以及 scrapy-cluster