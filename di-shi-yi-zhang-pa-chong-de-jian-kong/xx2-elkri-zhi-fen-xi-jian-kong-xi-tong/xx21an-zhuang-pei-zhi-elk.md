## 14.2.1 安装配置 ELK

在进行 ELK 系统的搭建之前，我们首先需要在服务器搭建 java 运行环境，为了安装时不出错，建议选统一 ElasticSearch 版本，全部选择5.５版本。

>System:Centos release 7.3

>Java:Oracle-jdk version "1.8.0_131"

>ElasticSearch : 5.5.0

>Kibana : 5.5.0

ELK 各个组件我们在 Centos 中可以采用 yum, rpm 方式安装，当然也可以采用源码安装，今天我们来介绍一下源码安装方式。

### 1. 安装配置 Java 环境
Elasticsearch 官方建议使用 Oracle 的 JDK8，在安装之前首先要确定下机器有没有安装 JDK。

***解压安装***

首先在终端中输入命令，进行验证:

`java -version`

由于我的系统是最小化安装的，终端显示了报错信息。

解压已经下载好的 jdk-8u131-linux-x64.tar.gz 文件，并重命名为 jdk/。

使用如下命令,把　jdk/ 文件放入 /usr/local 目录下:

`sudo mv jdk/ /usr/local`

***环境变量配置***

我们虽然可以在 /etc/profile 文件之中进行环境配置，但为了便于然后的管理，推荐大家在 /etc/profile.d 目录下新建 shell 文件配置环境:

`sudo vim /etc/profile.d/jdk.sh`

写入如下内容：

```
export JAVA_HOME=/usr/local/jdk
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

保存后运行 `source /etc/profile` 使环境变量生效。

退出当前终端，并重新登录，输入 `java -version` 确认安装:

![](/assets/javaversion.png)

### 2. ElasticSearch 安装

第一步，解压下载的 zip 文件，重命名为 elasticsearch/ , 并移动至 /usr/local 目录下。

执行如下命令:

`sudo vi /usr/local/elasticsearch/config/elasticsearch.yml`

第二步，修改配置文件如下:

```
#这里指定的是集群名称，需要修改为对应的，开启了自发现功能后，ES会按照此集群名称进行集群发现

cluster.name: elk

#数据目录
path.data:/data/elk/data

# log目录
path.logs:/data/elk/logs

#修改ES的监听地址，这样别的机器也可以访问
network.host:0.0.0.0

#默认的端口号
http.port:9200
```

第三步：为普通用户添加权限

`sudo chown -R user /data/elk`

把 /data/elk 目录及其下的所有文件和子目录的属主改成 user。

第四步，我们还需要修改系统默认参数，以确保有足够的资源启动 ES：

***设置内核参数***

执行如下命令:

`sudo vi /etc/sysctl.conf`

>//在文件之中，添加如下参数:
>vm.max_map_count=655360

再执行如下命令生效：

`sudo sysctl -p`

***设置资源参数***

`sudo vi /etc/security/limits.conf`

在底部添加如下内容：

> \* soft nofile 65536

> \* hard nofile 131072

> \* soft nproc 65536

> \* hard nproc 131072

***设置用户参数***

`sudo vi /etc/security/limits.d/20-nproc.conf`

> 修改
>user soft nproc 65536

接着 **重启** 机器，并输入 `ulimit -n` 进行验证，如果显示并非预期的 65536，比如 1024 则可能需要修改 ssh 配置：

`sudo vim /etc/ssh/sshd_config`

把其中的 UseLogin 的值改为 yes,　然后重启服务：

`sudo service sshd restart`

退出当前用户，并查看 `ulimit -n`，发现已改为 65536.

第五步　我们需要启动 es 并验证安装，**重启** 机器以后，进入 elasticsearch 的 bin 目录，以非 root 用户
在后台启动:

`./elasticsearch -d`

使用 curl 查看信息：

`curl -X GET http://localhost:9200`

![](/assets/qidonges.png)

这样，我们便完成了 es 的安装。

### 3. 安装 logstash

参考 es 的安装，解压文件，重命名并移动至 /usr/local 目录下。然后进入 logstash/bin 文件夹，输入：

`./logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'`

然后你会发现终端在等待你的输入。没问题，敲入 Hello World，回车，然后查看会返回结果:

![](/assets/fhuijkiegelk.png)

***生成并编辑配置文件***

这里，我们新建一个 logstash 配置文件，我们首先在输入项中设置了日志(input.file.path)的路径，在 filter 下面，进行正则匹配过滤。最后指定输出为本地 elasticsearch。

```
input {
  file {
    path => "/data/elk/logs/*.log"
    start_position => beginning
    ignore_older => 0
    sincedb_path => "/dev/null"
  } 
}
filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}
output {
  elasticsearch { hosts => ["xxx.xxx.x.xxx:9200"] }
  stdout { codec => rubydebug }
}
```

可能会有报错，但这个配置文件仅仅为了演示，如何改进会在下文演示。

在 logstash 目录之中执行如下命令，即可启动 logstash：

`./bin/logstash -f ./config/simple.conf `

***提升 logstash 启动速度***

logstash 在安装完成以后，启动时间可能会越来越长，甚至是5到10多分钟。以至于怀疑程序错误，这时候可能是系统的“熵”低了。

如果出现logstash启动慢，使用如下命令查询，。

`cat /proc/sys/kernel/random/entropy_avail`

如果返回值小于1000，那么就需要安装 haveged 包，执行如下命令即可:

`yum install haveged -y && systemctl start haveged && systemctl enable haveged && systemctl status haveged`

### 4. 安装 kibana

参考 es 的安装，解压文件，重命名并移动至 /usr/local 目录下。然后进入 kibana/config 文件夹，编辑 kibana.yml 配置文件，修改以下参数：

```
#开启默认端口5601如果5601被占用可用5602或其他
server.port:5601

#站点地址
server.host:“localhost”　

#指向elasticsearch服务的ip地址
elasticsearch.url:http://localhost:9200

kibana.index:“.kibana”
```

***关闭 selinux ***

执行如下命令：

`sudo vim /etc/sysconfig/selinux `

>设置 SELINUX=disabled

然后执行

`sudo setenforce 0`


进入相应目录，cd kibana/bin 运行　./kibana

通过kibana窗口观察你的结果：

http://localhost:5601

![](/assets/kibana1.png)