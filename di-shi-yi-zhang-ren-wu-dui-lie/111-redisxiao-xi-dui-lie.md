## 11.1 Redis 消息队列

Redis 作为内存数据库，基本上是存内存操作，这意味着内存并不是 redis 的瓶颈。同时，其异步非阻塞的 I/O 特性使得 redis 可以胜任许多需要应用消息队列的场景。下面就让我们体验感受一下吧。

### 1. Redis 消息队列解决方案
Redis 在消息队列中主要有两种解决方案：
1. 自带的 PUB/SUB 机制
此时，模式的生产者发布一条消息会被多个消费者消费，主要使用于广播的问题，但不适用于队列问题。主要适用于对于数据可靠性要求不高的应用场景。
2. PUSH/POP 机制
利用列表数据结构，生产者 lpush 消息，消费者 brpop 消息，并设置超时时间。此时数据的可靠性会有较大的提升，而且可以通过多个 client 来提升消费的速度。但消息状态相对关于简单，如果消费失败需要重新 push 到队列之中。下面，我们使用了redis 提供的 blpop 获取队列数据，如果队列没有数据则阻塞等待，也就是监听:

```
import redis

class Task(object):
    def __init__(self):
        self.con = redis.StrictRedis(host='localhost',db=5,password='foobared')
        self.queue = 'task:prodcons:queue'
        
    def listen_task(self):
        while True:
            task = self.con.blpop(self.queue, 0)[1]
            print('Task get', task)

if __name__ == '__main__':
    print('listen task queue')
    Task().listen_task()
```

这里我们基于 Redis 实现了简单的队列监听，单启动脚本以后，还需要生成者向队列之中推送数据。

### 2. 应用实例
首先，我们需要启动上文之中的队列监听脚本，然后运行如下脚本：

```
import redis
import random
import logging
from flask import Flask, redirect

app = Flask(__name__)

rcon = redis.StrictRedis(host='localhost', db=5 ,password='foobared')
prodcons_queue = 'task:prodcons:queue'
 
@app.route('/')
def index():
 
    html = """
        <br>
        <center><h3>Redis Message Queue</h3>
        <br>
        <a href="/prodcons">生产消费者模式</a>
        <br>
        </center>
        """
    return html
 
@app.route('/prodcons')
def prodcons():
    elem = random.randrange(10)
    rcon.lpush(prodcons_queue, elem)
    logging.info("lpush {} -- {}".format(prodcons_queue, elem))
    return redirect('/')
 
if __name__ == '__main__':
    app.run(port=5001,debug=True)
```

启动脚本，并在浏览器进行访问。很快就可以通过监听任务，看到相关打印信息。当然我们也可以通过 web 压力测试工具进行试验。在异步的任务中，可以执行一些耗时间的操作，当然目前这些做法并不知道异步的执行结果，如果需要知道异步的执行结果，可以考虑设计协程任务或者使用一些工具如RQ或者celery等。
