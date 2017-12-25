## 14.2.3 ElasticSearch 集群搭建

使用 es 进行数据存储与分析，往往还需要部署 es 集群，今天我们就基于 elasticsearch5.5.0 版本进行 es 集群的搭建。

### 1. 配置文件
es 有2个配置文件，分别是：

* elasticsearch.yml：ES 的具体配置。
* log4j2.properties：ES 日志文件。

今天，我主要了解一下 elasticsearch.yml 的配置。

#### (1) 网络
es 在网络方面主要提供2种协议(http,tcp):
>http: 用于 REST API 的使用
>tcp: 用于节点之间的通信

对应模块有 http 与 transport:

* http.host：设置 HTTP 的服务地址；
* http.port：设置 HTTP 端口，默认端口 9200-9300；
* http.cors.enabled: 是否支持跨域；
* http.cors.allow-origin: 跨域支持
* transport.host：设置 TCP 的服务地址；
* transport.tcp.port：设置 TCP 端口，默认端口 9300-9400；

同时，es 还提供了一个 network.host 设置，http和tcp的host默认是绑定在它上面的，所以以上参数可以不用进行配置。

* network.host：默认为本地回环地址。当定义该属性时，程序环境默认切换至生产环境（即开启 Bootstrap Checks）

#### (2) 集群
es 中的集群是根据集群名称来关联的，它会将集群名称相同的节点自动关联成一个集群，并通过节点名称来做区分。节点常用的分为 集群管理节点 master 与数据操作节点 data，默认情况下是两者都是。

* cluster.name：集群名称，默认名称 elasticsearch；
* node.name：节点名称，同一集群下，节点名称不能重复；
* node.master: 指定节点是否可以被选为 node
* node.data: 指定节点是否存储索引数据，默认为 true

#### (3) 发现机制
* discovery.zen.ping.unicast.hosts：需发现的节点地址，支持 IP 和 domain，在不指定端口时，会默认扫描端口 9300 - 9305。

```
discovery.zen.ping.unicast.hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
```

* discovery.zen.minimum_master_nodes：master 节点最少可用数，当节点数小于该值，服务将不可用。这也是为了避免集群中出现多个中心，导致数据不一致；

```
# 计算公式：(master 节点数 / 2) + 1
# 例子：(3 / 2) + 1
discovery.zen.minimum_master_nodes: 2
```

#### (4) Bootstrap Checks（生产环境）
上面讲到了 Bootstrap Checks，这也是它区别之前老版本的一个地方，在以前的版本中，也有这些警告，但有时会被人忽视，造成了服务的不稳定性。在版本 5.0 之后，对这些在启动时做了强校验，来保证在生产环境下的稳定性。这些校验主要涉及有内存、线程数、文件句柄等。

** JVM heap **
建议将最小堆与最大堆设置为一样，当设置bootstrap.memory_lock时，在程序启动就会对内存进行锁定。

`ES_JAVA_OPTS=-Xms512 -Xmx512m`

** 内存锁定，禁止内存与磁盘的置换 **

`bootstrap.memory_lock: true`

不过因为Centos6不支持SecComp，而ES5默认`bootstrap.system_call_filter`为 true 进行检测，所以可能会导致检测失败，失败后直接导致ES不能启动

```
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```

最后，还可以通过 ulimit 取消文件数与线程数限制。

### 2. 容器化配置
容器化部署，主要通过 docker 来实现，为简单起见，以 docker 方式进行部署。

* es1.yml

```
cluster.name: my-application
node.name: node-1
# 可以选择宿主机地址，网络模式选择 --net=host
network.host: 172.17.0.2
bootstrap.memory_lock: true
discovery.zen.ping.unicast.hosts: ["172.17.0.3"]
discovery.zen.minimum_master_nodes: 2
```

* es2.yml

```
cluster.name: my-application
node.name: node-2
network.host: 172.17.0.3
bootstrap.memory_lock: true
discovery.zen.ping.unicast.hosts: ["172.17.0.2"]
discovery.zen.minimum_master_nodes: 2
```

现在，启动 elasticsearch 容器：

```
# 启动 es1
docker run -d \
--name=es1 \
-p 9200:9200 \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
--ulimit memlock=-1:-1 \
--ulimit nofile=65536:65536 \
-v ${YOUR_PATH}/es1.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v ${YOUR_PATH}/data:/usr/share/elasticsearch/data \
-v ${YOUR_PATH}/logs:/usr/share/elasticsearch/logs \
elasticsearch:5.5.0
```

```
# 启动 es2
docker run -d \
--name=es2 \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
--ulimit memlock=-1:-1 \
--ulimit nofile=65536:65536 \
-v ${YOUR_PATH}/es2.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v ${YOUR_PATH}/data:/usr/share/elasticsearch/data \
-v ${YOUR_PATH}/logs:/usr/share/elasticsearch/logs \
elasticsearch:5.5.0
```










