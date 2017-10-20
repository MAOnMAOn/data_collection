## 7.3.2 Redis数据库数据存储

Redis 是一个基于内存的高效的键值型非关系型数据库，存取效率极高，而且支持多种存储数据结构，使用也非常简单，这里我们简单介绍一下 Python 的 Redis 操作。

### 1、准备工作
在本节开始之前请确保已经安装好了 Redis 及 RedisPy库，如果要做数据导入导出操作的话还需要安装 RedisDump。

### 2、Redis与StrictRedis
RedisPy 库提供两个类 Redis 和 StrictRedis 用于实现Redis 的命令操作。
StrictRedis 实现了绝大部分官方的命令，参数也一一对应，比如 set() 方法就对应 Redis 命令的 set 方法。而Redis 是 StrictRedis 的子类，它的主要功能是用于向后兼容旧版本库里的几个方法，为了做兼容，将方法做了改写，比如 lrem() 方法就将 value 和 num 参数的位置互换，和Redis 命令行的命令参数不一致。

官方推荐使用 StrictRedis，所以我们也用 StrictRedis类相关方法作演示。

3. 连接Redis
当前在本地我已经安装了 Redis 并运行在 6379 端口，密码设置为 foobared。
那么可以用如下示例连接 Redis 并测试：

```
from redis import StrictRedis
redis = StrictRedis(host='localhost', port=6379, db=0, password='foobared')
redis.set('name', 'Bob')
```

在这里我们传入了 Redis 的地址，运行端口，使用的数据库，密码信息。在默认不传的情况下，这四个参数分别为 localhost、6379、0、None。现在我们声明了一个StrictRedis 对象，然后接下来调用了 set() 方法，设置一个键值对。运行没有报错就说明我们连接成功，并可以执行 set()、get() 操作了。

当然我们还可以使用 ConnectionPool 来连接，示例如下：
```
from redis import StrictRedis, ConnectionPool

pool = ConnectionPool(host='localhost', port=6379, db=0,password='foobared')
redis = StrictRedis(connection_pool=pool)
```

这样的连接效果是一样的，观察源码可以发现 StrictRedis内其实就是用 host、port 等参数又构造了一个 ConnectionPool，所以我们直接将 ConnectionPool 当参数传给 StrictRedis 也是一样的。

另外 ConnectionPool 还支持通过 URL 来构建，URL 的格式支持如下三种：

 1. redis://[:password]@host:port/db
 2. rediss://[:password]@host:port/db
 3. unix://[:password]@/path/to/socket.sock?db=db

这三种 URL 分别表示创建 Redis TCP 连接、Redis TCP+SSL 连接、Redis Unix Socket 连接，我们只需要构造上面任意一种连接 URL 即可，其中 password 部分如果有则可以写，没有可以省略，下面我们再用URL连接进行演示：

```
url = 'redis://:foobared@localhost:6379/0'
pool = ConnectionPool.from_url(url)
redis = StrictRedis(connection_pool=pool)
```

在这里我们使用了第一种连接字符串进行连接，我们首先声明了一个 Redis 连接字符串，然后调用 from_url() 方法创建一个 ConnectionPool，然后将其传给 StrictRedis 即可完成连接，所以使用 URL 的连接方式还是比较方便的。

### 4、 Key 操作
在这里主要将 Key 的一些判断和操作方法做下总结：

方法    |	作用  |	  参数说明  |	示例   |	示例说明 |	示例结果
--------|---------|-------------|----------|-------------|----------
exists(name)| 判断一个key是否存在 |	name: key名	|redis.exists('name')|	是否存在name这个key	|True
delete(name) |删除一个key|	name: key名 |redis.delete('name')|	删除name这个key	| 1
type(name)|判断key类型|name: key名 |redis.type('name')|	判断name这个key类型	| b'string'
keys(pattern)|获取所有符合规则的key	| pattern: 匹配规则|redis.keys('n*')|	获取所有以n开头的key |[b'name']
randomkey()|获取随机的一个key|	|	randomkey() |获取随机的一个key|	b'name'
rename(src, dst) | 将key重命名 | src: 原key名 dst: 新key名 |	redis.rename('name', 'nickname')| 将name重命名为nickname | True
dbsize() | 获取当前数据库中key的数目 |	|	dbsize() |	获取当前数据库中key的数目  |100
expire(name, time) |设定key过期时间，单位秒	|name: key名 time: 秒数|	redis.expire('name', 2)	| 将name这key的过期时间设置2秒 |True
ttl(name) |	获取key过期时间，单位秒，-1为永久不过期	| name: key名 |	redis.ttl('name') |	获取name这key的过期时间	| -1
move(name, db) | 将key移动到其他数据库 |name: key名 db: 数据库代号|	move('name', 2)	| 将name移动到2号数据库	| True
flushdb() |	删除当前选择数据库中的所有key |	| flushdb() |	删除当前选择数据库中的所有key |	True
flushall() |删除所有数据库中的所有key | | flushall() |	删除所有数据库中的所有key |	True

### 5、String操作
Redis 中存在最基本的键值对形式存储，用法总结如下：

方法   |	作用   |	参数说明  |	示例  |  示例说明	 |  示例结果
-------|-----------|--------------|-------|--------------|-------------
set(name, value) |给数据库中key为name的string赋予值value|name: key名 value: 值|redis.set('name', 'Bob')|给name这个key的value赋值为Bob |	True
get(name) |	返回数据库中key为name的string的value |	name: key名 |	redis.get('name') |	返回name这个key的value | b'Bob'
getset(name, value) |	给数据库中key为name的string赋予值value并返回上次的value	|name: key名 value: 新值 |redis.getset('name', 'Mike') |	赋值name为Mike并得到上次的value	| b'Bob'
mget(keys, *args) |	返回多个key对应的value | keys: key的列表 |	redis.mget(['name', 'nickname']) | 返回name和nickname的value | [b'Mike', b'Miker']
setnx(name, value) | 若key不存在才设置value	| name: key名 |	redis.setnx('newname', 'James')	|如果newname这key不存在则设置值为James |	第一次运行True，第二次False
setex(name, time, value) |	设置可以对应的值为string类型的value，并指定此键值对应的有效期 |	name: key名 time: 有效期 value: 值 | redis.setex('name', 1, 'James') |	将name这key的值设为James，有效期1秒	| True
setrange(name, offset, value)| 设置指定key的value值的子字符串 |	name: key名 offset: 偏移量 value: 值	| redis.set('name', 'Hello') redis.setrange('name', 6, 'World') |	设置name为Hello字符串，并在index为6的位置补World |	11，修改后的字符串长度
mset(mapping) |	批量赋值 |	mapping: 字典 |	redis.mset({'name1': 'Durant', 'name2': 'James'}) | 将name1设为Durant，name2设为James |True
msetnx(mapping)	| key均不存在时才批量赋值 |	mapping: 字典 |	redis.msetnx({'name3': 'Smith', 'name4': 'Curry'}) |	在name3和name4均不存在的情况下才设置二者值	 | True
incr(name, amount=1) |	key为name的value增值操作，默认1，key不存在则被创建并设为amount | name: key名 amount:增长的值 |	redis.incr('age', 1) |	age对应的值增1，若不存在则会创建并设置为1 |	1，即修改后的值
decr(name, amount=1) |	key为name的value减值操作，默认1，key不存在则被创建并设置为-amount |	name: key名 amount:减少的值 |	redis.decr('age', 1) |	age对应的值减1，若不存在则会创建并设置为-1 | -1，即修改后的值
append(key, value) | key为name的string的值附加value	| key: key名 |	redis.append('nickname', 'OK') | 向key为nickname的值后追加OK |	13，即修改后的字符串长度
substr(name, start, end=-1) | 返回key为name的string的value的子串 | name: key名 start: 起始索引 end: 终止索引，默认-1截取到末尾 |	redis.substr('name', 1, 4) |	返回key为name的值的字符串，截取索引为1-4的字符 | b'ello'
getrange(key, start, end) |	获取key的value值从start到end的子字符串 |	key: key名 start: 起始索引 end: 终止索引 | redis.getrange('name', 1, 4) |	返回key为name的值的字符串，截取索引为1-4的字符 | b'ello'

### 6、List操作
List，即列表。Redis 还提供了列表存储，列表内的元素可以重复，而且可以从两端存储，用法总结如下：

方法  | 	作用  |	参数说明  |	示例  | 	示例说明  |	示例结果
------|-----------|-----------|-------|---------------|-----------------
rpush(name, *values)| 在key为name的list尾添加值为value的元素，可以传多个 |	name: key名 values: 值 | redis.rpush('list', 1, 2, 3) |	给list这个key的list尾添加1、2、3  |	3，list大小
lpush(name, *values) | 在key为name的list头添加值为value的元素，可以传多个|	name: key名 values: 值 | redis.lpush('list', 0) |	给list这个key的list头添加0  |	4，list大小
llen(name) | 返回key为name的list的长度 | name: key名 | redis.llen('list')|	返回key为list的列表的长度 |	4
lrange(name, start, end) | 返回key为name的list中start至end之间的元素 |	name: key名 start: 起始索引 end: 终止索引 |	redis.lrange('list', 1, 3) |	返回起始为1终止为3的索引范围对应的list | [b'3', b'2', b'1']
ltrim(name, start,end)|	截取key为name的list，保留索引为start到end的内容 |	name:key名 start: 起始索引 end: 终止索引 |ltrim('list', 1, 3) |	保留key为list的索引为1到3的元素	| True
lindex(name, index)	| 返回key为name的list中index位置的元素 | name: key名 index: 索引 | redis.lindex('list', 1) | 返回key为list的列表index为1的元素|	b'2'
lset(name, index, value) |	给key为name的list中index位置的元素赋值，越界则报错 | name: key名 index: 索引位置 value: 值 | redis.lset('list', 1, 5) |	将key为list的list索引1位置赋值为5 |	True
lrem(name, count, value) | 删除count个key的list中值为value的元素|name: key名 count: 删除个数 value: 值	| redis.lrem('list', 2, 3) |	将key为list的列表删除2个3 |	1，即删除的个数
lpop(name) | 返回并删除key为name的list中的首元素	| name: key名	| redis.lpop('list') | 返回并删除名为list的list第一个元素 | b'5'
rpop(name) | 返回并删除key为name的list中的尾元素 | name: key名 |	redis.rpop('list') |	返回并删除名为list的list最后一个元素 | b'2'
blpop(keys, timeout=0) |	返回并删除名称为在keys中的list中的首元素，如果list为空，则会一直阻塞等待|	keys: key列表 timeout: 超时等待时间，0为一直等待 |	redis.blpop('list')|	返回并删除名为list的list的第一个元素 |	[b'5']
brpop(keys, timeout=0) |	返回并删除key为name的list中的尾元素，如果list为空，则会一直阻塞等待	| keys: key列表 timeout: 超时等待时间，0为一直等待 |	redis.brpop('list')|	返回并删除名为list的list的最后一个元素 | [b'2']
rpoplpush(src, dst) |	返回并删除名称为src的list的尾元素，并将该元素添加到名称为dst的list的头部|	src: 源list的key dst: 目标list的key	| redis.rpoplpush('list', 'list2') |	将key为list的list尾元素删除并返回并将其添加到key为list2的list头部|	b'2'

### 7、Set操作
Set，即集合。Redis 还提供了集合存储，集合中的元素都是不重复的，用法总结如下：

方法   |	作用	| 参数说明  |	示例 |	示例说明  | 	示例结果
-------|------------|-----------|--------|------------|---------------
sadd(name, *values)	|向key为name的set中添加元素 | name: key名 values: 值，可为多个 |	redis.sadd('tags', 'Book', 'Tea', 'Coffee') |	向key为tags的set中添加Book、Tea、Coffee三个内容 | 3，即插入的数据个数
srem(name, *values) | 从key为name的set中删除元素 | name: key名 values: 值，可为多个 |	redis.srem('tags', 'Book')|	从key为tags的set中删除Book|	1，即删除的数据个数
spop(name) | 随机返回并删除key为name的set中一个元素 | name: key名 |	redis.spop('tags')|	从key为tags的set中随机删除并返回该元素 | b'Tea'
smove(src, dst, value) | 从src对应的set中移除元素并添加到dst对应的set中|	src: 源set dst: 目标set value: 元素值 | redis.smove('tags', 'tags2', 'Coffee') |	从key为tags的set中删除元素Coffee并添加到key为tags2的set|	True
scard(name)| 返回key为name的set的元素个数 |	name: key名 |	redis.scard('tags')	| 获取key为tags的set中元素个数	| 3
sismember(name, value) | 测试member是否是key为name的set的元素|name:key值|	redis.sismember('tags', 'Book') | 判断Book是否为key为tags的set元素|	True
sinter(keys, *args)	|返回所有给定key的set的交集 | keys: key列表	redis.sinter(['tags', 'tags2']) |	返回key为tags的set和key为tags2的set的交集 |	{b'Coffee'}
sinterstore(dest, keys, *args) | 求交集并将交集保存到dest的集合 |	dest:结果集合 keys:key列表 | redis.sinterstore('inttag', ['tags', 'tags2']) |	求key为tags的set和key为tags2的set的交集并保存为inttag |	1
sunion(keys, *args) | 返回所有给定key的set的并集 |	keys: key列表 |	redis.sunion(['tags', 'tags2'])	| 返回key为tags的set和key为tags2的set的并集 |	{b'Coffee', b'Book', b'Pen'}
sunionstore(dest, keys, *args) | 求并集并将并集保存到dest的集合 |	dest:结果集合 keys:key列表 | redis.sunionstore('inttag', ['tags', 'tags2']) |	求key为tags的set和key为tags2的set的并集并保存为inttag |	3
sdiff(keys, *args) | 返回所有给定key的set的差集 | keys: key列表 |	redis.sdiff(['tags', 'tags2']) |	返回key为tags的set和key为tags2的set的差集 |	{b'Book', b'Pen'}
sdiffstore(dest, keys, *args) |	求差集并将差集保存到dest的集合 |	dest:结果集合 keys:key列表| redis.sdiffstore('inttag', ['tags', 'tags2'])|	求key为tags的set和key为tags2的set的差集并保存为inttag |	3
smembers(name) | 返回key为name的set的所有元素 |	name: key名 |	redis.smembers('tags') | 返回key为tags的set的所有元素 |	{b'Pen', b'Book', b'Coffee'}
srandmember(name) |	随机返回key为name的set的一个元素，但不删除元素 |	name: key值	| redis.srandmember('tags') |	随机返回key为tags的set的一个元素 |

### 8、Sorted Set操作
Sorted Set，即有序集合，它相比集合多了一个分数字段，利用它我们可以对集合中的数据进行排序，其用法总结如下：

方法 |	作用  | 	参数说明  | 	示例 |	示例说明  |	示例结果
-----|--------|---------------|----------|------------|-------------
zadd(name, args, *kwargs) |	向key为name的zset中添加元素member，score用于排序。如果该元素存在，则更新其顺序 | name: key名 args: 可变参数 |	redis.zadd('grade', 100, 'Bob', 98, 'Mike')	| 向key为grade的zset中添加Bob，score为100，添加Mike，score为98 |	2，即添加的元素个数
zrem(name, *values) | 删除key为name的zset中的元素 |	name: key名 values: 元素 |	redis.zrem('grade', 'Mike') | 从key为grade的zset中删除Mike |	1，即删除的元素个数
zincrby(name, value, amount=1) |	如果在key为name的zset中已经存在元素value，则该元素的score增加amount，否则向该集合中添加该元素，其score的值为amount |	name: key名 value: 元素 amount: 增长的score值 |	redis.zincrby('grade', 'Bob', -2) |	key为grade的zset中Bob的score减2 | 98.0，即修改后的值
zrank(name, value) |	返回key为name的zset中元素的排名（按score从小到大排序）即下标 |	name: key名 value: 元素值	| redis.zrank('grade', 'Amy') |	得到key为grade的zset中Amy的排名	 | 1
zrevrank(name, value) |	返回key为name的zset中元素的倒数排名（按score从大到小排序）即下标 |	name: key名 value: 元素值 | redis.zrevrank('grade', 'Amy') |	得到key为grade的zset中Amy的倒数排名 | 2
zrevrange(name, start, end, withscores=False) |	返回key为name的zset（按score从大到小排序）中的index从start到end的所有元素|	name: key值 start: 开始索引 end: 结束索引 withscores: 是否带score |	redis.zrevrange('grade', 0, 3) | 返回key为grade的zset前四名元素	|  [b'Bob', b'Mike', b'Amy', b'James']
zrangebyscore(name, min, max, start=None, num=None, withscores=False)|	返回key为name的zset中score在给定区间的元素 | name:key名 min: 最低score max:最高score start: 起始索引 num: 个数 withscores: 是否带score |	redis.zrangebyscore('grade', 80, 95) |	返回key为grade的zset中score在80和95之间的元素 |	[b'Amy', b'James']
zcount(name, min, max) | 返回key为name的zset中score在给定区间的数量 |	name:key名 min: 最低score max: 最高score | redis.zcount('grade', 80, 95)|	返回key为grade的zset中score在80到95的元素个数 |	2
zcard(name)	| 返回key为name的zset的元素个数 | name: key名 |	redis.zcard('grade') |	获取key为grade的zset中元素个数 | 3
zremrangebyrank(name, min, max) |	删除key为name的zset中排名在给定区间的元素 |	name:key名 min: 最低位次 max: 最高位次 |	redis.zremrangebyrank('grade', 0, 0) |	删除key为grade的zset中排名第一的元素 |	1，即删除的元素个数
zremrangebyscore(name, min, max) |	删除key为name的zset中score在给定区间的元素 | name:key名 min: 最低score max:最高score |	redis.zremrangebyscore('grade', 80, 90) |	删除score在80到90之间的元素 | 1，即删除的元素个数

### 9、Hash操作
Hash，即哈希。Redis 还提供了哈希表的数据结构，我们可以用name指定一个哈希表的名称，然后表内存储了各个键值对，用法总结如下：

方法  | 	作用 |	参数说明  | 	示例  |	 示例说明  | 	示例结果
------|----------|------------|-----------|------------|-------------
hset(name, key, value) | 向key为name的hash中添加映射 | name: key名 key: 映射键名 value: 映射键值 |	hset('price', 'cake', 5) |	向key为price的hash中添加映射关系，cake的值为5 |	1，即添加的映射个数
hsetnx(name, key, value)|	向key为name的hash中添加映射，如果映射键名不存在| name: key名 key: 映射键名 value: 映射键值 | hsetnx('price', 'book', 6) |	向key为price的hash中添加映射关系，book的值为6 | 1，即添加的映射个数
hget(name, key)	| 返回key为name的hash中field对应的value	| name: key名 key: 映射键名  |	redis.hget('price', 'cake') |	获取key为price的hash中键名为cake的value	 | 5
hmget(name, keys, *args) | 返回key为name的hash中各个键对应的value |	name: key名 keys: 映射键名列表 | redis.hmget('price', ['apple', 'orange']) |	获取key为price的hash中apple和orange的值	| [b'3', b'7']
hmset(name, mapping) | 向key为name的hash中批量添加映射 | name: key名 mapping: 映射字典 |	redis.hmset('price', {'banana': 2, 'pear': 6}) |	向key为price的hash中批量添加映射  |	True
hincrby(name, key, amount=1) | 将key为name的hash中映射的value增加amount|	name: key名 key: 映射键名 amount: 增长量 |	redis.hincrby('price', 'apple', 3) | key为price的hash中apple的值增加3 |  6，修改后的值
hexists(name, key) | key为namehash中是否存在键名为key的映射 | name: key名 key: 映射键名 |	redis.hexists('price', 'banana') |	key为price的hash中banana的值是否存在 |	True
hdel(name, *keys) |	key为namehash中删除键名为key的映射 | name: key名 key: 映射键名 |	redis.hdel('price', 'banana') |	从key为price的hash中删除键名为banana的映射|	True
hlen(name) | 从key为name的hash中获取映射个数 |	name: key名 |	redis.hlen('price')	| 从key为price的hash中获取映射个数	| 6
hkeys(name)	| 从key为name的hash中获取所有映射键名 |	name: key名 |	redis.hkeys('price') | 从key为price的hash中获取所有映射键名	|   [b'cake', b'book', b'banana', b'pear']
hvals(name)	| 从key为name的hash中获取所有映射键值 |	name: key名 |	redis.hvals('price') |	从key为price的hash中获取所有映射键值  |	[b'5', b'6', b'2', b'6']
hgetall(name) |	从key为name的hash中获取所有映射键值对 |	name: key名 |	redis.hgetall('price') | 从key为price的hash中获取所有映射键值对    |{b'cake': b'5', b'book': b'6', b'orange': b'7', b'pear': b'6'}