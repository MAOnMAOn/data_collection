## 14.2.2 安装 ElasticSearch 插件

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

在 http://locahost:9100/ 查看服务

![](/assets/jiqunweilianjie.png)

如果显示　“集群健康值: 未连接”，则修改elasticsearch目录下 conf/elasticsearch.yml，在 elasticsearch/conf 目录下的 elasticsearch.yml 中添加配置如下：

```
#避免出现跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: "X-Requested-With,Content-Type, Content-Length, Authorization"
```
当然，如果你要在另外的主机进行访问 head ,则需要修改源码:

4.修改head的源码

`vim elasticsearch-head/Gruntfile.js`

```
                connect: {
                        server: {
                                options: {
                                        port: 9100,
                                        hostname: '*',
                                        base: '.',
                                        keepalive: true
                                }
                        }
                }

增加hostname:'*'
```
修改目录中的　_site/app.js，并找到
`this.base_uri = this.config.base_uri ||this.prefs.get("app-base_uri") || "http://localhost:9200";`

把es的地址改为正确的地址。

最后，重启es 和head即可

![](/assets/headyilang.png)

### 2. elasticserch-bigdesk 安装
Bigdesk为Elastic集群提供动态的图表与统计数据。主要用于监控集群性能、CPU使用情况、内存使用率等，方便更好的配置集群节点等。

首先下载插件：

`git clone https://github.com/hlstudio/bigdesk.git`

然后，cd 到bigdesk/_site/目录下，运行下面命令

`python -m SimpleHTTPServer`

而Linux默认是安装Python的，所以只需要执行python -m SimpleHTTPServer。即可，而我们的HTTP服务在8000号端口上侦听。你会得到下面的信息：

`Serving HTTP on 0.0.0.0 port 8000 …`

然后打开浏览器（IE或Firefox），然后输入下面的URL：

`http://localhost:8000`

![](/assets/node88888.png)

如果你的目录下有一个叫 index.html 的文件名的文件，那么这个文件就会成为一个默认页，如果没有这个文件，那么，目录列表就会显示出来。

### 3. 实现主机访问虚拟机内 Web 服务

有时候, 我们会把一些站点部署在虚拟机之中，但是即便实体机与虚拟机可以互相 ping 通, 但就是不能在实体机里面访问虚拟机网络，那么此时就需要设置虚拟机防火墙。运行如下命令即可：

`sudo firewall-cmd --permanent --add-port=9100/tcp`

`sudo firewall-cmd --permanent --add-port=9200/tcp`

`sudo firewall-cmd --permanent --add-port=5601/tcp`

`sudo firewall-cmd --reload`
