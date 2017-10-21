## 1.9.2 安装 ElasticSearch 插件

之前我们简单说完 ELK 组件的安装和配置，但是看着这枯燥无味的命令行启动的ELK，可能会让大家觉得很抽象，只知道安装成功了，不知道怎么使用，不知道该从何下手；那么我们就可以给ES安装几个可视化控件，可以更方便、更直观的了解ES的结构，以及之后为ES集群扩展做准备。

### 1. elasticserch-head 安装
ElasticSearch-Head 是一个与 Elastic 集群（Cluster）相交互的Web前台。其可以展现ES集群的拓扑结构，并且可以通过它来进行索引（Index）和节点（Node）级别的操作，并且提供了一组针对集群的查询API，并将结果以json和表格形式返回结果，接下来我们就来说明一下安装过程。

#### (1) 安装nodejs
打开终端输入

`sudo curl -sL -o /etc/yum.repos.d/khara-nodejs.repo https://copr.fedoraproject.org/coprs/khara/nodejs/repo/epel-7/khara-nodejs-epel-7.repo`

然后执行：

`sudo yum install -y nodejs nodejs-npm`

使用淘宝镜像源：

` sudo npm install -g cnpm --registry=https://registry.npm.taobao.org`

#### (2) 安装git
我们可以执行以下命令，完成安装 git
```
sudo yum remove git #移除之前老的git
git --version #查看是否安装git
sudo yum install git #安装git
```
#### (3) 安装head
首先执行命令克隆插件：

`git clone git://github.com/mobz/elasticsearch-head.git`

然后进入 elasticsearch-head，执行

`sudo cnpm install && npm run start`

http://locahost:9100/查看服务

![](/assets/jiqunweilianjie.png)

如果显示　“集群健康值: 未连接”

则修改elasticsearch目录下 conf/elasticsearch.yml

在 elasticsearch/conf 目录下的 elasticsearch.yml 中添加配置如下：

```
#避免出现跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"
```

重启es 和head即可

ps:这里不着重介绍如何使用插件，大家可以自行去了解或者看我下一篇集群介绍。

elasticserch-kopf 安装
简介

Kopf是一个ElasticSearch的管理工具，它也提供了对ES集群操作的API。是一个我非常喜欢，非常强大的工具。


安装

使用git下载kopf

git clone git://github.com/lmenezes/elasticsearch-kopf.git
下载elasticserch-kopf 完成后会在用户下创建elasticserch-kopf 文件夹

cd到elasticserch-kopf 目录下

npm install 安装kopf npm会自动检索没有安装的插件

可能会遇到的问题

Local Npm module "grunt-contrib-copy" not found.  Is it installed?

Local Npm module "grunt-contrib-uglify" not found.  Is it installed?

Local Npm module "grunt-contrib-jshint" not found.  Is it installed?

Local Npm module "grunt-contrib-less" not found.  Is it installed?

Local Npm module "grunt-contrib-clean" not found.  Is it installed?

Local Npm module "grunt-contrib-watch" not found.  Is it installed?

Local Npm module "grunt-contrib-concurrent" not found.  Is it installed?

Local Npm module "grunt-contrib-nodemon" not found.  Is it installed?

Local Npm module "grunt-contrib-newer" not found.  Is it installed?

Warning: Task "copy:vendor" not found. Use --force to continue.

Aborted due to warnings.
这是因为grunt缺少对应的module

通过命令

npm install module-名字 --save-dev 
把所有缺少的module安装上即可

安装完成运行

grunt server &

http://locahost:9000/查看服务


配置kopf 添加elasticserch服务端口即可。

elasticserch-bigdesk 安装
简介

Bigdesk为Elastic集群提供动态的图表与统计数据。主要用于监控集群性能、CPU使用情况、内存使用率等，方便更好的配置集群节点等。


安装

打开终端下载

git clone https://github.com/hlstudio/bigdesk.git
cd 到bigdesk/_site/目录下

运行

python -m SimpleHTTPServer
而Linux默认是安装Python的，所以只需要执行python -m SimpleHTTPServer。

这就行了，而我们的HTTP服务在8000号端口上侦听。你会得到下面的信息：

Serving HTTP on 0.0.0.0 port 8000 …

然后打开浏览器（IE或Firefox），然后输入下面的URL：

http://localhost:8000

如果你的目录下有一个叫 index.html 的文件名的文件，那么这个文件就会成为一个默认页，如果没有这个文件，那么，目录列表就会显示出来。

### 4. 实现主机访问虚拟机内 Web 服务

有时候, 我们会把一些站点部署在虚拟机之中，但是即便实体机与虚拟机可以互相 ping 通, 但就是不能在实体机里面访问虚拟机网络，那么此时就需要设置虚拟机防火墙。运行如下命令即可：

`sudo firewall-cmd --permanent --add-port=9100/tcp`
`sudo firewall-cmd --permanent --add-port=9200/tcp`
`sudo firewall-cmd --permanent --add-port=5601/tcp`

`sudo firewall-cmd --reload`
