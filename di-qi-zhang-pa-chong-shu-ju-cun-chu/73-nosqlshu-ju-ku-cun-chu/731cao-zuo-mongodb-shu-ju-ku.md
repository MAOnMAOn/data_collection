## 7.3.1 操作　Mongodb数据库

MongoDB 是由 C++ 语言编写的非关系型数据库，是一个基于分布式文件存储的开源数据库系统，其内容存储形式类似 Json 对象，它的字段值可以包含其他文档，数组及文档数组，非常灵活，在这一节我们来看一下 Python3 下 MongoDB 的存储操作。

### 1、准备工作

在开始之前请确保已经安装好了 MongoDB 并启动了其服务，另外安装好了 Python 的 PyMongo库。

### 2、 连接MongoDB

连接 MongoDB 我们需要使用 PyMongo 库里面的 MongoClient，一般来说传入 MongoDB 的 IP 及端口即可，第一个参数为地址 host，第二个参数为端口 port，端口如果不传默认是 27017。

```
import pymongo
client = pymongo.MongoClient(host='localhost', port=27017)
```

这样我们就可以创建一个 MongoDB 的连接对象了。
另外 MongoClient 的第一个参数 host 还可以直接传MongoDB 的连接字符串，以 mongodb 开头，例如：

`client = MongoClient('mongodb://localhost:27017/')`

可以达到同样的连接效果。

### 3、 指定数据库

MongoDB 中还分为一个个数据库，接下来的一步就是指定要操作哪个数据库，在这里以 test 数据库为例进行说明，所以下一步我们需要在程序中指定要使用的数据库。

```
db = client.test
db = client['test']
```

两种方式是等价的。

### 4、 指定集合

MongoDB 的每个数据库又包含了许多集合 Collection，也就类似与关系型数据库中的表，下一步我们需要指定要操作的集合，在这里我们指定一个集合名称为 students，学生集合，和指定数据库类似，指定集合也有两种方式：

```
collection = db.students
collection = db['students']
```

### 5、 插入数据

接下来我们便可以进行数据插入了，对于 students 这个Collection，我们新建一条学生数据，以字典的形式表示：
```
student = {
    'id': '20170101',
    'name': 'Jordan',
    'age': 20,
    'gender': 'male'
}
```

在这里我们指定了学生的学号、姓名、年龄和性别，然后接下来直接调用 collection 的 insert() 方法即可插入数据，代码如下：

`result = collection.insert(student)`

在 MongoDB 中，每条数据其实都有一个 _id 属性来唯一标识，如果没有显式指明 _id，MongoDB 会自动产生一个 ObjectId 类型的 _id 属性。insert() 方法会在执行后返回的 _id 值。

当然我们也可以同时插入多条数据，只需要以列表形式传递即可，示例如下：
```
student1 = {
    'id': '20170101',
    'name': 'Jordan',
    'age': 20,
    'gender': 'male'
}

student2 = {
    'id': '20170202',
    'name': 'Mike',
    'age': 21,
    'gender': 'male'
}

result = collection.insert([student1, student2])
print(result)
```

返回的结果是对应的 _id 的集合。

实际上在 PyMongo 3.X 版本中，insert() 方法官方已经不推荐使用了，当然继续使用也没有什么问题，官方推荐使用 insert_one() 和 insert_many() 方法将插入单条和多条记录分开。

```
student = {
    'id': '20170101',
    'name': 'Jordan',
    'age': 20,
    'gender': 'male'
}

result = collection.insert_one(student)
print(result)
print(result.inserted_id)
```

运行结果：

```
<pymongo.results.InsertOneResult object at 0x10d68b558>
5932ab0f15c2606f0c1cf6c5
```

返回结果和 insert() 方法不同，这次返回的是InsertOneResult 对象，我们可以调用其 inserted_id 属性获取 _id。

对于 insert_many() 方法，我们可以将数据以列表形式传递即可，示例如下：

```
student1 = {
    'id': '20170101',
    'name': 'Jordan',
    'age': 20,
    'gender': 'male'
}

student2 = {
    'id': '20170202',
    'name': 'Mike',
    'age': 21,
    'gender': 'male'
}

result = collection.insert_many([student1, student2])
print(result)
print(result.inserted_ids)
```

insert_many() 方法返回的类型是 InsertManyResult，调用inserted_ids 属性可以获取插入数据的 _id 列表，运行结果：

```
<pymongo.results.InsertManyResult object at 0x101dea558>
[ObjectId('5932abf415c2607083d3b2ac'), ObjectId('5932abf415c2607083d3b2ad')]
```

### 6、 查询
插入数据后我们可以利用 find_one() 或 find() 方法进行查询，find_one() 查询得到是单个结果，find() 则返回一个生成器对象。

```
result = collection.find_one({'name': 'Mike'})
print(type(result))
print(result)
```

在这里我们查询 name 为 Mike 的数据，它的返回结果是字典类型，运行结果：

```
<class 'dict'>
{'_id': ObjectId('5932a80115c2606a59e8a049'), 'id': '20170202', 'name': 'Mike', 'age': 21, 'gender': 'male'}
```

可以发现它多了一个 _id 属性，这就是 MongoDB 在插入的过程中自动添加的。

对于多条数据的查询，我们可以使用 find() 方法，例如在这里查找年龄为 20 的数据，示例如下：
```
results = collection.find({'age': 20})
print(results)
for result in results:
    print(result)
```

运行结果：

```
<pymongo.cursor.Cursor object at 0x1032d5128>
{'_id': ObjectId('593278c115c2602667ec6bae'), 'id': '20170101', 'name': 'Jordan', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('593278c815c2602678bb2b8d'), 'id': '20170102', 'name': 'Kevin', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('593278d815c260269d7645a8'), 'id': '20170103', 'name': 'Harden', 'age': 20, 'gender': 'male'}
```

返回结果是 Cursor 类型，相当于一个生成器，我们需要遍历取到所有的结果，每一个结果都是字典类型。

如果要查询年龄大于 20 的数据，则写法如下：

`results = collection.find({'age': {'$gt': 20}})`

在这里查询的条件键值已经不是单纯的数字了，而是一个字典，其键名为比较符号 $gt，意思是大于，键值为 20，这样便可以查询出所有年龄大于 20 的数据。

在这里将比较符号归纳如下表：

 符号  |  含义    |     示例
------|---------|--------------------
|\$lt |   小于  | {'age': {'\$lt': 20}}
|\$gt |   大于  | {'age': {'\$gt': 20}}
|\$lte| 小于等于| {'age': {'\$lte': 20}}
|\$ne |  不等于 | {'age': {'\$ne': 20}}
|\$in |	在范围内| {'age': {'\$in': [20, 23]}}
|\$nin|不在范围内|{'age': {'$nin': [20, 23]}}

另外还可以进行正则匹配查询，例如查询名字以 M 开头的学生数据，示例如下：

`results = collection.find({'name': {'$regex': '^M.*'}})`

在这里使用了 $regex 来指定正则匹配，^M.* 代表以 M 开头的正则表达式，这样就可以查询所有符合该正则的结果。

在这里将一些功能符号再归类如下：

 符号    |	含义  |	      示例 |	示例含义
--------|---------|----------------------------|-------------
|\$regex |匹配正则|{'name': {'\$regex': '^M.*'}}| name 以 M开头
|\$exists|属性是否存在|{'name':{'\$exists': True}} name| name 属性存在
|\$type	|类型判断|	{'age': {'\$type': 'int'}}| age 的类型为 int
|\$mod  |数字模操作 | {'age': {'\$mod': [5, 0]}} |年龄模 5 余 0
|\$text	|文本查询|{'\$text': {'\$search': 'Mike'}} |text 类型属性中含 Mike 字符串
|\$where|高级条件查询|{'\$where': 'obj.fans_count == obj.follows_count'}	|自身粉丝数等于关注数

以上操作的更详细用法在可以在 MongoDB 官方文档找到： https://docs.mongodb.com/manual/reference/operator/query/。

### 7、 计数
要统计查询结果有多少条数据，可以调用 count() 方法，如统计所有数据条数：
```
count = collection.find().count()
print(count)
```

或者统计符合某个条件的数据，其结果是一个数值，即符合条件的数据条数：

```
count = collection.find({'age': 20}).count()
print(count)
```

### 8、 排序
可以调用 sort() 方法，传入排序的字段及升降序标志即可，示例如下：

```
results = collection.find().sort('name', pymongo.ASCENDING)
print([result['name'] for result in results])
```

运行结果：

`['Harden', 'Jordan', 'Kevin', 'Mark', 'Mike']`

在这里我们调用了 pymongo.ASCENDING 指定升序，如果要降序排列可以传入 pymongo.DESCENDING。

### 9、 偏移
在某些情况下我们可能想只取某几个元素，在这里可以利用skip() 方法偏移几个位置，比如偏移 2，就忽略前 2 个元素，得到第三个及以后的元素。

```
results = collection.find().sort('name', pymongo.ASCENDING).skip(2)
print([result['name'] for result in results])
```

运行结果：

`['Kevin', 'Mark', 'Mike']`

另外还可以用 limit() 方法指定要取的结果个数，示例如下：

```
results = collection.find().sort('name', pymongo.ASCENDING).skip(2).limit(2)
print([result['name'] for result in results])
```

运行结果：

`['Kevin', 'Mark']`

如果不加 limit() 原本会返回三个结果，加了限制之后，会截取 2 个结果返回。

值得注意的是，在数据库数量非常庞大的时候，如千万、亿级别，最好不要使用大的偏移量来查询数据，很可能会导致内存溢出，可以使用类似如下操作来进行查询：

```
from bson.objectid import ObjectId
collection.find({'_id': {'$gt': ObjectId('593278c815c2602678bb2b8d')}})
```

这时记录好上次查询的 _id。

### 10、 更新
对于数据更新可以使用 update() 方法，指定更新的条件和更新后的数据即可，例如：
```
condition = {'name': 'Kevin'}
student = collection.find_one(condition)
student['age'] = 25
result = collection.update(condition, student)
print(result)
```
在这里我们将 name 为 Kevin 的数据的年龄进行更新，首先指定查询条件，然后将数据查询出来，修改年龄，之后调用 update() 方法将原条件和修改后的数据传入，即可完成数据的更新。
运行结果：

`{'ok': 1, 'nModified': 1, 'n': 1, 'updatedExisting': True}`

返回结果是字典形式，ok 即代表执行成功，nModified 代表影响的数据条数。

另外我们也可以使用 $set 操作符对数据进行更新，代码改写如下：

`result = collection.update(condition, {'$set': student})`

这样可以只更新 student 字典内存在的字段，如果其原先还有其他字段则不会更新，也不会删除。而如果不用 $set 的话则会把之前的数据全部用 student 字典替换，如果原本存在其他的字段则会被删除。

另外 update() 方法其实也是官方不推荐使用的方法，在这里也分了 update_one() 方法和 update_many() 方法，用法更加严格，第二个参数需要使用 $ 类型操作符作为字典的键名，我们用示例感受一下。

```
condition = {'name': 'Kevin'}
student = collection.find_one(condition)
student['age'] = 26
result = collection.update_one(condition, {'$set': student})
print(result)
print(result.matched_count, result.modified_count)
```

在这里调用了 update_one() 方法，第二个参数不能再直接传入修改后的字典，而是需要使用 {'$set': student} 这样的形式，其返回结果是 UpdateResult 类型，然后调用 matched_count 和 modified_count 属性分别可以获得匹配的数据条数和影响的数据条数。

我们再看一个例子：

```
condition = {'age': {'$gt': 20}}
result = collection.update_one(condition, {'$inc': {'age': 1}})
print(result)
print(result.matched_count, result.modified_count)
```

在这里我们指定查询条件为年龄大于 20，然后更新条件为 {'$inc': {'age': 1}}，也就是年龄加 1，执行之后会将第一条符合条件的数据年龄加 1。
运行结果：

```
<pymongo.results.UpdateResult object at 0x10b8874c8>
1 1
```

可以看到匹配条数为 1 条，影响条数也为 1 条。

如果调用 update_many() 方法，则会将所有符合条件的数据都更新，示例如下：

```
condition = {'age': {'$gt': 20}}
result = collection.update_many(condition, {'$inc': {'age': 1}})
print(result)
print(result.matched_count, result.modified_count)
```

这时候匹配条数就不再为 1 条了，运行结果如下：

```
<pymongo.results.UpdateResult object at 0x10c6384c8>
3 3
```

可以看到这时所有匹配到的数据都会被更新。

### 11、 删除
删除操作比较简单，直接调用 remove() 方法指定删除的条件即可，符合条件的所有数据均会被删除，示例如下：

```
result = collection.remove({'name': 'Kevin'})
print(result)
```

运行结果：

`{'ok': 1, 'n': 1}`

另外依然存在两个新的推荐方法，delete_one() 和 delete_many() 方法，示例如下：

```
result = collection.delete_one({'name': 'Kevin'})
print(result)
print(result.deleted_count)
result = collection.delete_many({'age': {'$lt': 25}})
print(result.deleted_count)
```

运行结果：

```
<pymongo.results.DeleteResult object at 0x10e6ba4c8>
1
4
```

delete_one() 即删除第一条符合条件的数据，delete_many() 即删除所有符合条件的数据，返回结果是 DeleteResult 类型，可以调用 deleted_count 属性获取删除的数据条数。

### 12、 更多

这里 PyMongo 还提供了一些组合方法，如find_one_and_delete()、find_one_and_replace()、find_one_and_update()，就是查找后删除、替换、更新操作，用法与上述方法基本一致。

同时，我们还可以对索引进行操作，如 create_index()、create_indexes()、drop_index() 等。

另外还有对数据库、集合本身以及其他的一些操作，在这不再一一讲解，可以参见官方文档：http://api.mongodb.com/python/current/api/pymongo/。
