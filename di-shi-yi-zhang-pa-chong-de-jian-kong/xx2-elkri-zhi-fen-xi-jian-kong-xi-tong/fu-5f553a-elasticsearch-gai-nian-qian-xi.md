## 附录:ElasticSearch概念浅析

ElasticSearch是一个基于[Lucene](https://baike.baidu.com/item/Lucene)的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，可以进行实时搜索，实时分析，实时存储引擎。ES不是个关系型数据库，是文档数据库，也可称NoSQL数据库。著名的 github 网站，就是用 es 来搜索 TB 级别的数据，包括13亿文件与1300亿行的代码\(2016年\)。

### 1. 基本概念

* 集群：多台 ES 服务器结合的统称叫 ES 集群，一个集群包括多台服务器，多个节点。

* 节点：也叫 Node,一个节点是集群中的一个服务器，作为集群的一部分，其参与集群的索引与搜索功能。

* 索引：一个索引就是一个拥有几分相似特征的文档集合。下面命令可以查看当前节点的所有 Index:

  > `curl -X GET 'http://localhost:9200/_cat/indices?v'`

* 类型：在一个索引之中，可以定义一种或多种类型。一个类型是索引的一个逻辑上的分类/分区，其语义用于自定义。根据规划，Elastic 6.x 版只允许每个 Index 包含一个 Type，7.x 版将会彻底移除 Type。下面的命令可以列出每个 Index 所包含的 Type:

  > `curl 'localhost:9200/_mapping?pretty=true'`

* 文档:是可被索引的基础信息单元。文档以 JSON 格式来表示。文档必须被索引/赋予一个索引的type。

![](/assets/文档索引图.png)

* 分片：一个索引保存了文档数据以后，会分布放在各个分片之中，分片又被放到集群各个机器上。各个分片都是独立的"索引"\(可增加、删除、修改、查询\)
  > 优点：横向拓展，水平分割数据容量；可在分片上并行操作。

![](/assets/esdpian存储.png)

副本：也叫复制分片，一个分片可有多个复制分片，其主要用于防止分片故障，加速索引查询\(一个节点失败，其他节点顶上\)。

### 2. 与关系型数据库的异同

es 与关系型数据库都可以进行数据的存储与检索外，还有一些近似的特征值：

| 关系型数据库 | ElasticSearch |
| --- | --- |
| 数据库 Database | 索引 Index |
| 表 Table | 类型 Type |
| 数据行 Row | 文档 Document |
| 数据列 Column | 字段 field |

当然，他们也有各自的使用场景：

|  | 关系型数据库 | ElasticSearch |
| --- | --- | --- |
| 存储方式 | 行存储，适用于OLTP业务 | 索引存储，适用于检索业务 |
| 拓展性 | 多单机，拓展性不佳 | 水平拓展 |
| 事务 | 支持 | 不支持 |
| 一致性 | strong consistency | 可配置 |
| 二级索引 | 支持 | 支持 |
| 全文检索 | 支持，较鸡肋 | 支持 |

可以看到，MySQL更为成熟，而且支持事务，支持二级索引，容灾备份方案也最为成熟，所以线上核心业务Mysql是不二之选。

而ES现在不仅提供全文检索，还提供统计功能，并且提供的Restful接口非常好用，配上Kibana还可以进行图形化展示，第三方插件也很丰富。虽然ES可以水平扩展，但是考虑到ES的大部分检索都会检索该index的所有shard，如果单个index数据过大，性能多少也会受到影响，所以单个index的大小最好控制在一定的范围。而且ES也可以作为MySQL索引来使用，虽然Mysql也有索引功能，但是过多的索引往往会拖累MySQL的性能，并且线上MySQL数据库大多也不允许执行统计类的sql，这时可以用ES辅助实现统计。

### 3. 倒排索引

当我们在阅读一本书的时候，通常使用页数来查找相关内容。每一页上具有一定数量的文本，这些文本记录的信息。当使用倒排方式后，不再有整页整页的信息，信息被分割成一个个的关键字，并辅以关键字所在的原书中的页数，而构成一个倒排基本单位。

例如，一本旅游类书籍的第250页讲诉了“麻婆豆腐”的来历，在经过倒排后，“麻婆豆腐”被作为一个单独的关键字切分出来，保存在索引中，同时还带有其页码250，作为索引的内容。这样当信息系统检索“麻婆豆腐”这个关键字的时候，系统可以迅速的给出其页码，然后再到原书中取出相关页面文本内容。这种查找方式比从第一页开始线性匹配所有文本，找出包含有“麻婆豆腐”的页面内容快得多。

![](/assets/倒排索引相关.png)

从理论上说，倒排是一种面向单词的索引机制。通常，由词和出现情况两部分组成。对于索引中的每个词，都跟随一个列表（位置表），用来记录单词在所有文档中出现的位置。由于不是由记录来确定属性值，而是由属性值确定记录位置，故名倒排索引。

| 关键字 | 倒排列表 |
| --- | --- |
| java\(TF-IDF\) | \(文章1,&lt2, 10&gt;,2\) |
| 聊天 | \(文章2,&lt12,25,100&gt;,3\) |
| 系统 | \(文章3,&lt10&gt;,1\) |
| 屏蔽脏话 | \(文章5,&lt50,60&gt;,2\) |
| 功能原理 | \(文章6,&lt56,57,58&gt;,3\) |

在倒排索引中，关键字的数量并非随着文本内容的增长也线性增长。这因为不论多大数量的文本数据库，总能规范出一个关键词表搜到实际语言因素的限制，它的增长率在文本数据库达到一定规模后可以忽略不计。有人做过统计，对于1GB的文本信息来说，词汇表的大小在5MB左右。

### 4. 基本命令操作

#### \(1\) 新建和删除索引

新建索引，可以直接发出 PUT 请求：

`curl -X PUT 'localhost:9200/goods'`

服务器会返回一个 json 对象，里面的 acknowledged 自动表示操作成功。现在，我们可以发出一个 DELETE 请求，去删除该索引。

`curl -X DELETE 'localhost:9200/weather'`

#### \(2\) 使用中文分词设置

首先，安装中文分词插件，这里使用 ik：

`$ ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.5.1/elasticsearch-analysis-ik-5.5.1.zip`

上面代码安装的是5.5.1版的插件，与 Elastic 5.5.1 配合使用。  
接着，重新启动 Elastic，就会自动加载这个新安装的插件。然后，新建一个 Index，指定需要分词的字段。这一步根据数据结构而异，下面的命令只针对本文。基本上，凡是需要搜索的中文字段，都要单独设置一下。

```
curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}'
```

这里，我们首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个中文字段\(user、title、desc\)，需要指定中文分词器。分词器也叫 analyzer。

#### \(4\) 数据操作

##### a. 新增记录

向指定的 /Index/Type 发送 PUT 请求，就可以在 Index 里面新增一条记录。

```
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}'
```

服务器会返回一个 json 对象，并给出 Index、Type、Id、Version等相关信息。注意到，路劲之中最后的1是记录的id,其也可以是字母。

如果新增记录时候，不去指定id，需要改成 POST 请求。此时，服务器返回的\_id字段会是一个随机字符串：

```
curl -X POST 'localhost:9200/accounts/person' -d '
{
  "user": "李四",
  "title": "工程师",
  "desc": "系统管理"
}'
```

##### b. 查看记录

向/Index/Type/Id发出 GET 请求，就可以查看这条记录，如果 Id 不正确，就查不到数据：

`curl 'localhost:9200/accounts/person/1?pretty=true'`

##### c. 删除记录

删除记录就是发出 DELETE 请求:

`curl -X DELETE 'localhost:9200/accounts/person/1'`

##### d. 更新记录

更新，就是使用 PUT 请求，重新发送一次数据：

```
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理，软件开发"
}'
```

#### \(5\) 数据查询

查询大概分为三类：基本查询，组合查询，过滤\(查询同时，通过filter条件在不影响打分的情况下筛选数据\)，今天我们来看一下 match 查询：

##### a. 返回所有记录

使用 GET 方法，直接请求 /Index/Type/\_search 即可：

`curl 'localhost:9200/accounts/person/_search'`

返回：

```
{"took":2,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":2,"max_score":1.0,"hits":[{"_index":"accounts","_type":"person","_id":"AWBO3zpg0PC03vhf0GoB","_score":1.0,"_source":
{
  "user": "李四",
  "title": "工程师",
  "desc": "系统管理"
}},{"_index":"accounts","_type":"person","_id":"1","_score":1.0,"_source":
{
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理，软件开发"
}}]}}
```

返回结果的took字段表示该操作的耗时（单位为毫秒），timed\_out字段表示是否超时，hits字段表示命中的记录，里面子字段之中：total: 表示返回记录数，这里为2, max\_score: 最高匹配长度，这里为 1.0, hits表示返回记录组成的数组。另外返回记录之中的 \_score 字段，表示匹配的长度，默认按照该字段进行降序排序。

##### b. 全文检索

在 ES 之中，使用自己的查询语法，需要在 GET 请求带有数据体。

```
curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件" }}
}'
```

这里使用 Match 查询，指定了匹配条件是 desc 字段包含'软件'这个词语。

#### \(6\) 关于 mapping 映射

创建索引时，可以预先定义字段的类型以及相关属性，每个字段定义一种类型，属性比mysql里面丰富，前面没有传入，因为elasticsearch会根据json源数据来猜测是什么基础类型。M挨批评就是我们自己定义的字段的数据类型，同时告诉elasticsearch如何索引数据以及是否可以被搜索。

配置相关属性：

* String类型： 两种text keyword。text会对内部的内容进行分析，索引，进行倒排索引等，为设置为keyword则会当成字符串，不会被分析，只能完全匹配才能找到String。在es5已经被废弃了。

* 日期类型：date 以及datetime等。

* 数据类型:integer long double等等。

* bool类型

* binary类型

* 复杂类型：object nested。

* geo类型：geo-point地理位置。

* 专业类型：ip competition。

* object ：json里面内置的还有下层{}的对象。

* nested：数组形式的数据。



