## 1.1 Python3的安装

标签（空格分隔）： 章节说明

既然要用 Python3 开发爬虫，那么第一步一定是安装 Python3。

centos7 下安装方式有多种，比如命令安装、源码安装以及 anaconda 安装，推荐使用anaconda 或者命令安装，毕竟源码安装需要自行编译，时间较长。

### 1. 命令行安装
#### python3.5版本
```
sudo yum install -y https://centos7.iuscommunity.org/ius-release.rpm
sudo yum update
sudo yum install -y python35u python35u-libs python35u-devel python35u-pip
```
执行完毕之后便可成功安装 Python3.5 及 Pip3。

### 2. 源码安装
如果命令行安装方式有问题，还可以下载 Python3 源码进行安装。

源码下载地址为：https://www.python.org/ftp/python， 可以自行选用想要的版本进行安装，在此以 Python3.6.2 为例进行说明，安装路径设置为 /usr/local/python3。

首先创建安装目录，命令如下：

`sudo mkdir /usr/local/python3`

接下来编译安装，所需时间可能较长，请耐心等待，命令如下：

```
sudo ./configure --prefix=/usr/local/python3
sudo make
sudo make install
```

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
Anaconda 同样支持 Linux，其官方下载链接为：https://www.continuum.io/downloads， 选择 Python3 版本的安装包下载即可。

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