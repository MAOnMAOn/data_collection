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


安装结束之后我们可以创建 Python3 软链接，命令如下：
`sudo ln -s /usr/local/python3/bin/python3 /usr/bin/python3`

随后下载 Pip 安装包并安装，命令如下：
```
wget --no-check-certificate https://github.com/pypa/pip/archive/9.0.1.tar.gz
tar -xzvf 9.0.1.tar.gz
cd pip-9.0.1
python3 setup.py install
```

安装完成后再创建 Pip3 链接，这样就成功安装了Python3以及pip3，命令如下：
`sudo ln -s /usr/local/python3/bin/pip /usr/bin/pip3`

### 3. Anaconda安装
Anaconda 同样支持 Linux，其官方下载链接为：https://www.continuum.io/downloads，选择 Python3 版本的安装包下载即可。

如果下载速度过慢可以选择使用清华大学镜像，
下载列表链接为：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/，
使用说明链接为：https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/，
我们可以选择需要的版本进行下载，速度相比官网会快很多。

### 4. 相关链接

官方网站：http://python.org
下载地址：https://www.python.org/downloads
第三方库：https://pypi.python.org/pypi
官方文档：https://docs.python.org/3
中文教程：http://www.runoob.com/python3/python3-tutorial.html
Awesome Python：https://github.com/vinta/awesome-python
Awesome Python 中文版：https://github.com/jobbole/awesome-python-cn







