## 1.4 数据库的安装

作为数据存储的重要部分，数据库同样是必不可少的，数据库可以分为关系型数据库和非关系型数据库。

关系型数据库如 SQLite、MySQL、Oracle、SQL Server、DB2 等，其数据库是以表的形式存储，非关系型数据库如MongoDB、Redis，其存储形式是键值对，存储形式更加灵活。这里用到的数据库主要有关系型数据库 MySQL 及非关系型数据库 MongoDB、Redis。

### 1、MySQL的安装
#### （1）安装mysql
以 MySQL 5.7 的 Yum 源为例，如果需要其他版本可以另寻，安装命令如下：

```
sudo  rpm -qa | grep maria
sudo  yum remove mariadb-libs -y
sudo  yum localinstall http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo  yum install mysql-community-server -y
```

#### （2）启动mysql
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

#### （3）修改mysql管理员密码
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

命令中 newpass 即为修改的新的 MySQL 密码，请自行替换,当然我们也可以修改mysql的默认密码安全策略，此处不再赘述。

#### （4）mysql远程访问配置
由于 Linux 一般会作为服务器使用，为了使得 MySQL 可以被远程访问，我们需要修改 MySQL 的配置文件，配置文件路径一般为 /etc/mysql/my.cnf。
如使用 vi 进行修改的命令如下：

`vi /etc/mysql/my.cnf`

取消此行的注释：

`bind-address = 127.0.0.1`

此行限制了 MySQL 只能本地访问而不能远程访问，取消注释即可解除此限制。

同时，mysql默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，我们添加一个新的帐户：

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'djangotest'@'%' IDENTIFIED BY 'admin123!' WITH GRANT OPTION;
```

修改完成之后重启 MySQL 服务，这样 MySQL 就可以被远程访问了。

#### （5）修改mysql默认字符编码
最后我们还需要配置默认编码为utf8，修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：

```
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
```

重新启动mysql服务即可。

到此为止，Linux 下安装 MySQL 的过程结束。

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