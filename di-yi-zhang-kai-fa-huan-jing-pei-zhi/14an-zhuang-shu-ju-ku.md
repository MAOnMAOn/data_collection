## 1.4 数据库的安装

作为数据存储的重要部分，数据库同样是必不可少的，数据库可以分为关系型数据库和非关系型数据库。

关系型数据库如 SQLite、MySQL、Oracle、SQL Server、DB2 等，其数据库是以表的形式存储，非关系型数据库如MongoDB、Redis，其存储形式是键值对，存储形式更加灵活。这里用到的数据库主要有关系型数据库 MySQL 及非关系型数据库 MongoDB、Redis。

### 1、MySQL的安装
#### (1) 安装mysql
以 MySQL 5.7 的 Yum 源为例，如果需要其他版本可以另寻，安装命令如下：

```
sudo  rpm -qa | grep maria
sudo  yum remove mariadb-libs -y
sudo  yum localinstall http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo  yum install mysql-community-server -y
```

#### (2) 启动mysql
接下来需要启动 MySQL 服务，启动 MySQL 服务命令：

```
sudo systemctl enable mysqld.service
sudo systemctl start mysqld.service
```

停止、重启命令：

```
sudo systemctl stop mysqld.service
sudo systemctl restart mysqld.service
```

#### (3) 修改mysql管理员密码
以上我们就完成了 Linux 下 MySQL 的安装，现在我们查找初始密码：

`grep 'temporary password' /var/log/mysqld.log`

安装完成之后可以修改密码，可以执行如下命令：

`mysql -uroot -p`

输入密码后进入 MySQL 命令行模式。

```
mysql> use mysql;
mysql> UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
mysql> FLUSH PRIVILEGES;
```

如果出现 ** You must reset your password using ALTER USER statement before executing this statement. ** 的错误提示，可以首先设置如下：

`set password = password("MkcCuY2fA=q/");`

如果出现 ** Your password does not satisfy the current policy requirements **的错误提示，则表示需要进行密码策略设置，在内部的测试服务器上可以简单测试。

>首先，把判断密码的标准修改为基于密码的长度：

>`set global validate_password_policy=0;`

>然后，把默认最小密码长度设置为4:

>`set global validate_password_length=1;`

详情见 [mysql 官方说明](http://dev.mysql.com/doc/refman/5.7/en/validate-password-options-variables.html#sysvar_validate_password_policy)

#### (4) mysql远程访问配置
由于 Linux 一般会作为服务器使用，为了使得 MySQL 可以被远程访问，我们需要修改 MySQL 的配置文件，配置文件路径一般为 /etc/my.cnf。
如使用 vi 进行修改的命令如下：

`vi /etc/my.cnf`

取消此行的注释：

`bind-address = 127.0.0.1`

此行限制了 MySQL 只能本地访问而不能远程访问，取消注释即可解除此限制。

同时，mysql默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，我们添加一个新的帐户：

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'djangotest'@'%' IDENTIFIED BY 'admin123!' WITH GRANT OPTION;
```

#### (5) 修改mysql默认字符编码
完成远程访问设置以后，还需要配置默认编码为utf8，修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：

```
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
```

#### (6) 修改 mysql 默认存储路径
首先停止mysql服务，编辑/etc/my.cnf配置文件，并在[mysqld]下添加设置,修改 datadir，把 datadir=/var/lib/mysql 改为自定义存储路径，如：

`datadir=/data/mysqldb/mysql/`

拷贝文件到 datadir, 通过 cp -a 保持原有权限：

`sudo cp -a /var/lib/mysql /data/mysqldb`

关闭系统校验(当然也可以关闭 SeLinux):

`sudo setenforce 0`

最后，启动mysql服务，并进入数据库验证：

![](/assets/mysql验证.png)

如果在错误日志之中出现如下警示：

`[Warning] Can't create test file /data/mysqldb/mysql/docker-52.lower-test`

则需要为 mysql 用户配置相应的文件目录权限，比如：

`cd /data/mysqldb/mysql && sudo chown -R mysql:mysql *`

这里，我们简单改变了整个目录的权限(这是一个不好的习惯)，一般可以修改指定文件目录的权限比如 **ibdata1 ** 。


#### (7) 其他设置
如果 mysql 是安装在服务器上，还需要修改/etc/my.cnf配置文件，并在[mysqld]下添加设置：

**设置独立表空间**

`innodb_file_per_table=1`

**设置查询缓存**

```
# query_cache
query_cache_size=0
query_cache_type=0
```

**配置 sql_mode**

`sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES`

** 重置root密码 **

1. 停止数据库服务

2. 修改 /etc/my.cnf 在 [mysqld] 下加入 `skip-grant-tables`

3. 启动数据库

4. 修改 root 密码
```
UPDATE mysql.user SET authentication_string=PASSWORD("yourpasswd") WHERE user='root' and Host = 'localhost';
flush privileges;
```

5. 修改 /etc/my.cnf 并注释掉 skip-grant-tables

最后，重新启动mysql服务即可。

### 2、MongoDB安装
MongoDB 是由 C++ 语言编写的非关系型数据库，是一个基于分布式文件存储的开源数据库系统，其内容存储形式类似 Json 对象，它的字段值可以包含其他文档，数组及文档数组，非常灵活。

#### （1）mongodb安装
首先添加 MongoDB 源：

`sudo vi /etc/yum.repos.d/mongodb-org.repo`

修改为如下内容并保存：

```
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

然后执行 yum 命令安装：

`sudo yum install mongodb-org`

启动 MongoDB 服务：

`sudo systemctl start mongod`

停止和重新加载 MongoDB 服务：

```
sudo systemctl stop mongod
sudo systemctl reload mongod
```

#### （2）mongodb的相关配置
一般我们在 Linux 上配置 MongoDB 都是为了远程连接使用的，所以在这里还需要配置一下 MongoDB 的远程连接和用户名密码，现在进入 MongoDB 命令行：

`mongo --port 27017`

然后进入到 MongoDB 的命令行交互模式下，在此模式下运行如下命令：

```
> use admin
switched to db admin
> db.createUser({user: 'admin', pwd: '123456', roles: [{role: 'root', db: 'admin'}]})
Successfully added user: {
        "user" : "admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
```

这样我们就创建了一个用户名为 admin，密码为 123456 的用户，赋予最高权限。
随后需要修改 MongoDB 的配置文件，
执行如下命令：

`sudo vi /etc/mongod.conf`

修改 net 部分为：

```
net:
  port: 27017
  bindIp: 0.0.0.0
```

这样配置后 MongoDB 可被远程访问。
另外还需要添加如下权限认证配置，直接添加如下内容到配置文件：

```
security:
 authorization: enabled
```

配置完成之后我们需要重新启动 MongoDB 服务，命令如下：

`sudo service mongod restart`

客户端连接后，我们可以进行验证，输入命令进入数据库：

`mongo --port 27017`

```
> use admin
switched to db admin
> db.auth("admin", "admin")
1 // 输出1表示验证成功
> show dbs
admin  0.000GB
local  0.000GB
```

这样远程连接和权限认证就配置完成了。

#### （3）mongodb的可视化工具

在这里推荐一个可视化工具 RoboMongo/Robo 3T，使用简单，功能强大，官方网站：https://robomongo.org/， 三大平台都有支持，下载链接：https://robomongo.org/download。

另外还有一个简单易用的可视化工具，Studio 3T，同样具有方便的图形化管理，官方网站：https://studio3t.com， 同样支持三大平台，下载链接：https://studio3t.com/download/。

### 3、Redis数据库安装
#### （1）Redis安装配置
Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

首先添加 EPEL 仓库，然后更新 Yum 源：

```
sudo yum install epel-release
sudo yum update
```

然后安装 Redis 数据库：

`sudo yum -y install redis`

安装好之后启动 Redis 服务：

`sudo systemctl start redis`

同样可以使用 redis-cli 进入 Redis 命令行模式操作。
另外为了可以使 Redis 能被远程连接，需要修改配置文件，路径为

`/etc/redis.conf`

注释这一行：

`bind 127.0.0.1`

另外推荐给 Redis 设置密码，取消注释这一行：

`requirepass foobared`

foobared 即当前密码，可以自行修改。
之后保存重启 Redis 服务：

`sudo systemctl restart redis`

这样就可以远程连接 Redis 了。

另外，在 Mac 以及Windows 下也可以安装 Redis Desktop Manager 可视化管理工具来管理 Redis。

#### （2）RedisDump的安装
RedisDump 是一个用于 Redis 数据导入导出的工具，是基于 Ruby 实现的，所以要安装 RedisDump 需要先安装Ruby。

##### a. 安装 Ruby 依赖

```
sudo yum install -y ruby ruby-devel rubygems
# 添加淘宝源
gem source --add https://gems.ruby-china.org/ 
查看并删除国外相关源
gem source -l && gem source --remove https://rubygems.org/ 
```

##### b. 安装redis-dump
Centos7.3 默认支持ruby到2.0.0，可gem 安装redis需要最低是2.2.2
解决办法是先安装rvm，再把ruby版本提升至2.3.3。

```
# 添加公钥
gpg2 --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
# 安装RVM
sudo curl -L get.rvm.io | bash -s stable
source /home/user/.rvm/scripts/rvm  # root 用户下为source /usr/local/rvm/scripts/rvm
# 安装一个 ruby 版本
rvm install 2.3.4
# 验证安装
ruby --version
# 最后安装redis-dump
gem install redis-dump
```

##### c. 导入导出数据

```
# 导出数据
redis-dump -u :password@192.168.1.159:6379 > test.json
导入数据
cat test.json | redis-load -u :password@192.168.1.159:6379
```

### 4. 安装Berkeley DB
Berkeley DB是一个嵌入式数据库，为应用程序提供可伸缩的、高性能的、有事务保护功能的数据管理服务。

其主要特点有：
* 嵌入式
>直接链接到应用程序中，与应用程序运行于同样的地址空间中，因此，无论是在网络上不同计算机之间还是在同一台计算机的不同进程之间，数据库操作并不要求进程间通讯。 Berkeley DB为多种编程语言提供了API接口，其中包括C、C++、Java、Perl、Tcl、Python和PHP，所有的数据库操作都在程序库内部发生。多个进程，或者同一进程的多个线程可同时使用数据库，有如各自单独使用，底层的服务如加锁、事务日志、共享缓冲区管理、内存管理等等都由程序库透明地执行。

* 轻便灵活
>可以运行于几乎所有的UNIX和Linux系统及其变种系统、Windows操作系统以及多种嵌入式实时操作系统之下，已经被好多高端的因特网服务器、台式机、掌上电脑、机顶盒、网络交换机以及其他一些应用领域所采用。一旦Berkeley DB被链接到应用程序中，终端用户一般根本感觉不到有一个数据库系统存在。

* 可伸缩
>Database library本身是很精简的（少于300KB的文本空间），但它能够管理规模高达256TB的数据库。它支持高并发度，成千上万个用户可同时操纵同一个数据库

#### 编译安装

```
$ sudo su    #　切换用户
# wget http://download.oracle.com/berkeley-db/db-6.2.32.NC.tar.gz
# tar zxvf db-6.2.32.NC.tar.gz
# mv db-6.2.32.NC /usr/local/berkeleydb
# cd build_unix
# ../dist/configure
# make && make install
# echo '/usr/local/berkeleydb/lib/' >> /etc/ld.so.conf
# ldconfig  　# 这2句的作用就是通知系统Berkeley DB的动态链接库在/usr/local/berkeleydb/lib/目录。
```