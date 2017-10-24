## 7.2.2 SqlAlchemy 简单使用

SQLAlchemy 是 Python 中最广泛使用的 ORM 工具之一，它采用了类似于 Java 里 Hibernate 的数据映射模型， 而不是其他 ORM 框架采用的 Active Record 模型。

SQLAlchemy分为两个部分，一个是最常用的 ORM 对象映射，另一个是核心的 SQL expression。 第一个比较容易理解，纯粹的 ORM；而第二个则是是 DBAPI 的封装，通过一些sql表达式来避免了直接写sql。 所以通过使用 SQLAlchemy 与数据库进行交互时我们可以：

 - 使用ORM避免直接书写sql
 - 使用raw sql直接书写sql
 - 使用sql expression，通过SQLAlchemy的方法写sql表达式
 
好了，下面就让我们一起感受一下吧！

### 1、连接数据库
在本节开始之前请确保已经安装好了 MySQL 数据库并正常运行，而且需要安装好 PyMysql 与 SQLAlchemy 库，如果没有安装，可以参考第一章的安装说明。

当准备工作完成，我们就可以开始连接数据库了，通过 sqlalchemy.create_engine() 可以连接数据库，同时确保已经安装相关驱动，这里我们使用PyMySQL 作为驱动，另外先要提前创建 test 这个测试数据库：

```
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://root:mysql@127.0.0.1:3306/test', echo=True)
```

create_engine 之中的字符串语句一般使用 MySQLdb/MySQL-Python 的默认写法，如：

`engine = create_engine('mysql://user:password@host:port/database', echo=True)`

这里使用了 PyMySQL，并设置 echo=True 开启调试，这样当我们执行文件的时候会提示相应的文字。

### 2、使用SQL语句直接操作数据库
假如在 MySQL 中的 test 数据库已有表student，列为 id,name,score，我们可以直接运行如下代码：

```
sql = 'SELECT * from student where id = 0'
ret = engine.execute(sql)
for item in ret:
    print(item) ## 输出id为0的行
```

当然 sqlalchemy 也支持 SQL 语句的事务操作，代码如下：

```
with engine.connect() as conn:
    trans = conn.begin()
    # 执行数据库事务
    try:
        trans.commit('select * from zhilian limit 2') # 
    except: 
        trans.rollback()
    trans.close()
```

### 3、ORM 操作
#### 1、定义映射
数据库连接完成之后，我们就可以通过  sqlalchemy 建立映射。这里使用两个表来说明，一个用户表users，一个电子邮件表 addresses，两者一对多的关系。我们先定义这两个映射：

```
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

Base = declarative_base()


class Address(Base):
    """电子邮件表"""
    __tablename__ = 'addresses'
    id = Column(Integer, primary_key=True)
    email_address = Column(String(30), nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship("User", back_populates="addresses")
    def __repr__(self):
        return "<Address(email_address='{}')>".format(self.email_address)
        
        
class User(Base):
    """用户表"""
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(10))
    fullname = Column(String(20))
    password = Column(String(20))
    addresses = relationship("Address", order_by=Address.id, back_populates="user")
    def __repr__(self):
        return "<User(name='{}', fullname='{}', password='{}')>".format(
            self.name, self.fullname, self.password)
            
Base.metadata.create_all(engine)  # 把数据存入数据库之中！
```

现在数据库里面已经有两个表了。下面我们就可以对这两个表进行常规的操作了。

#### 2、增删改查
对数据库的操作必须先创建一个session，增删改查操作都有这个session负责，首先我们先创建一个session工厂类，由它来负责后续的session创建

```
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
```

***添加用户***
```
session = Session()  # 先使用工程类来创建一个session
ed_user = User(name='ed', fullname='Ed Jones', password='edspassword')
session.add(ed_user)
# 同时创建多个
session.add_all([
    User(name='wendy', fullname='Wendy Williams', password='foobar'),
    User(name='mary', fullname='Mary Contrary', password='xxg527'),
    User(name='fred', fullname='Fred Flinstone', password='blah')])
# 提交事务
session.commit()
```

***数据查询***
在 session 上面调用 query( )方法会创建一个Query对象:

```
for user in session.query(User).order_by(User.id):
    print(user.name, user.fullname)
# 使用filter_by过滤
for name in session.query(User.name).filter_by(fullname='Ed Jones'):
    print(name)
# 使用sqlalchemy的SQL表达式语法过滤，可以使用python语句
for name in session.query(User.name).filter(User.fullname=='Ed Jones'):
    print(name)
```

***删除***

```
session.delete(ed_user)
session.query(User).filter_by(name='ed').count()
```

#### 3、一对多的关系映射
sqlalchemy 使用 ForeignKey 来指明一对多的关系，比如一个用户可有多个邮件地址，而一个邮件地址只属于一个用户。那么就是典型的一对多或多对一关系。在 Address 类中，我们定义外键，还有对应所属的 user 对象

```
user_id = Column(Integer, ForeignKey('users.id'))
user = relationship("User", back_populates="addresses")
```

而在User类中，我们定义addresses属性

`addresses = relationship("Address", order_by=Address.id, back_populates="user")`

可以注意到，这两个类中都通过 relationship() 方法指明相互关系。

下面，我们就通过几个例子来操作一对多的关系映射：

```
# 先添加一个用户，并且给这个用户增加两个邮件地址
jack = User(name='jack', fullname='Jack Bean', password='gjffdd')
jack.addresses = [Address(email_address='jack@google.com'),
                  Address(email_address='j25@yahoo.com')]
session.add(jack)
session.commit()
# 查询
jack = session.query(User).filter_by(name='jack').one()
# 只有在调用jack.addresses时才会调用查询邮件地址的SQL，这个是典型的懒加载模式
jack.addresses

***join查询***
session.query(User).join(Address).filter(Address.email_address=='jack@google.com').all()
```

有时候我们不想使用懒加载，而是要强制一次性加载某个关联数据，那么可以使用 joinedload 方法进行操作：

```
from sqlalchemy.orm import joinedload
jack = session.query(User).options(joinedload(User.addresses)).filter_by(name='jack').one()
```

