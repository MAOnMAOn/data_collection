## 1.9.1 安装配置 ELK

在进行 ELK 系统的搭建之前，我们首先需要在服务器搭建 java 运行环境，为了安装时不出错，建议选统一 ElasticSearch 版本，全部选择5.５版本。

>System:Centos release 7.3

>Java:Oracle-jdk version "1.8.0_131"

>ElasticSearch : 5.5.0

>Kibana : 5.5.0

### 1. 安装配置 Java 环境
Elasticsearch 官方建议使用 Oracle 的 JDK8，在安装之前首先要确定下机器有没有安装 JDK。

***解压安装***

首先在终端中输入命令，进行验证:

`java -version`

由于我的系统是最小化安装的，终端显示了报错信息。

解压 jdk-8u131-linux-x64.tar.gz 文件，并重命名为 jdk/。

使用如下命令,把　jdk/ 文件放入 /usr/local 目录下:

`mv jdk/ /usr/local`

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





