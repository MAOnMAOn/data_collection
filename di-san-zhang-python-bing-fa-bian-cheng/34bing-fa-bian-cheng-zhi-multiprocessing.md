## 3.5 并发编程之multiprocessing

之前我们介绍了 Python3 下的 _thread 多线程库的使用。并说明了一点：python中的多线程其实并不是真正的多线程，并不能做到充分利用多核CPU资源。如果想要充分利用，在python中大部分情况需要使用多进程，那么这个包就叫做 multiprocessing。

借助它，可以轻松完成从单进程到并发执行的转换。multiprocessing支持子进程、通信和共享数据、执行不同形式的同步，提供了Process、Queue、Pipe、Lock等组件。

### 1. Process
这里我首先定义5个worker函数(其中一个函数有明显错误),以便下文使用:

```
import time

def worker_0(interval):
    print("错误的工作")
    time.sleep(interval)
    0/0
    print("end工作失误")
def worker_1(interval):
    print("工作1")
    time.sleep(interval)
    print("end工作1")
def worker_2(interval):
    print("工作212")
    time.sleep(interval)
    print("end工作212")
def worker_3(interval):
    print("工作three")
    time.sleep(interval)
    print("end工作three")
def worker_4(interval):
    print("工作four")
    time.sleep(interval)
    print("end工作four")
```

***基本使用***

在multiprocessing中，每一个进程都用一个Process类来表示。首先看下它的API：

`Process([group [, target [, name [, args [, kwargs]]]]])`

 - target表示调用对象，可以传入方法的名字
 - args表示被调用对象的位置参数元组，比如target是函数a，他有两个参数m，n，那么args就传入(m, n)即可
 - kwargs表示调用对象的字典
 - name是别名，相当于给这个进程取一个名字
 - group分组，实际上不使用

Process类的相关方法有：

 - is_alive():判断进程是否存活
 - join([timeout]):子进程结束再执行下一步,让父进程等待子进程结束,可以通过设置 timeout 处理进程阻塞
 - run():如果在创建Process对象的时候不指定target，那么就会默认执行Process的run方法
 - start():启动进程,区分run()
 - terminate():终止进程,关于终止进程没有这么简单,貌似用psutil包会更好

Process类的相关属性：

 - authkey: process设置过程的授权密钥
 - daemon:父进程终止后自动终止，且自己不能产生新进程，必须在start()之前设置
 - exitcode:进程在运行时为None、如果为–N，表示被信号N结束
 - name:进程的名字,自定义
 - pid:每个进程有唯一的PID编号。

***Process(),start(),join()***

```
from multiprocessing import Process
import time

if __name__ == '__main__':
    a=time.time()
    p1=Process(target=worker_1,args=(4,))
    p2 = Process(target=worker_2, args=(6,))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
    b=time.time()
    print('finish',b-a)
```

这里一共开了两个进程,p1和p2,arg=(4,)中的4是fun1函数的参数,这里要用tulpe类型,如果两个参数或更多就是arg=(参数1,参数2...),之后用start()启动进程,我们设置等待p1和p2进程结束再执行下一步.来看下面的运行结果,2个函数基本在同一时间开始运行,当运行完毕(fun1睡眠4秒,同时fun2睡眠6秒),才执行print语句。

运行结果如下：

```
工作1
工作212
end工作1
end工作212
finish 6.02905011177063
```

现在再来看下start()与join()处于不同位置会发生什么：

```
from multiprocessing import Process
import time

if __name__ == '__main__':
    a=time.time()
    p1=Process(target=worker_1,args=(4,))
    p2 = Process(target=worker_2, args=(6,))
    p1.start()
    p1.join()
    p2.start()
    p2.join()
    b=time.time()
    print('finish',b-a)
```

运行结果：

```
工作1
end工作1
工作212
end工作212
finish 10.03864574432373
```

现在是先运行 worker_1 函数,运行完毕再运行 worker_2 接着再是 print 'finish',即先运行进程p1再运行进程p2,在每个子进程都调用了join()方法，这样父进程（主进程）就会等待子进程执行完毕。感受到 join() 的魅力了吧。

***name,daemon,is_alive()***

```
from multiprocessing import Process
import time

if __name__ == '__main__':
    a=time.time()
    p1=Process(target=worker_1,args=(4,))
    p2 = Process(target=worker_2, args=(6,))
    p1.daemon=True
    p2.daemon = True
    p1.start()
    p2.start()
    p1.join()
    print('进程1:',p1,'\n','进程2:',p2)
    print('进程1:',p1.is_alive(),'\n','进程2:',p2.is_alive())
    b=time.time()
    print('finish',b-a)
```

运行结果：

```
工作1
工作212
end工作1
进程1: <Process(Process-78, stopped daemon)>
 进程2: <Process(Process-79, started daemon)>
进程1: False
 进程2: True
finish 4.023045301437378
end工作212
```

可以看到,name是给进程赋予名字, 运行到 print('进程1:',p1.is_alive(),'\n','进程2:',p2.is_alive()) 这句的时候,p1进程已经结束(返回False),p2进程仍然在运行(返回True),但p2没有用join(),所以直接接着执行主进程,由于用了daemon=Ture,父进程终止后自动终止,p2进程没有结束就强行结束整个程序了。

***run()***

run()在Process没有指定target函数时,默认用run()函数运行程序。实例如下：

```
from multiprocessing import Process
import time

def fun1():
    print('this is fun1',time.ctime())
    time.sleep(2)
    print('fun1 finish',time.ctime())

if __name__ == '__main__':
    a=time.time()
    p=Process()
    p.run=fun1
    p.start()
    p.join()
    b=time.time()
    print('finish',b-a)
```

运行结果:

```
this is fun1 Mon Oct  9 17:30:53 2017
fun1 finish Mon Oct  9 17:30:55 2017
finish 2.0163445472717285
```

### 2. Lock
当线程的输出操作同时进行时，可能会出现资源不足，导致出现输出错位的情况。这时候我们需要相关输出'互斥',即在某一时间，只能一个进程输出，其他进程等待。等刚才那个进程输出完毕之后，另一个进程再进行输出。

我们可以通过 Lock 来实现，在一个进程输出时，加锁，其他进程等待。等此进程执行结束后，释放锁，其他进程可以进行输出。下面就用一个实例来感受一下：

```
from multiprocessing import Process, Lock
import time

class MyProcess(Process):
    def __init__(self, loop, lock):
        Process.__init__(self)
        self.loop = loop
        self.lock = lock

    def run(self):
        for count in range(self.loop):
            time.sleep(0.1)
            self.lock.acquire()
            print('Pid: ' + str(self.pid) + ' LoopCount: ' + str(count))
            self.lock.release()

if __name__ == '__main__':
    lock = Lock()
    for i in range(1, 5):
        p = MyProcess(i, lock)
        p.start()
```

我们在print方法的前后分别添加了获得锁和释放锁的操作。这样就能保证在同一时间只有一个print操作。所以在访问临界资源时，使用Lock就可以避免进程同时占用资源而导致的一些问题。

### 3. Queue
Process 之间有时需要通信，可以使用 multiprocessing 的 Queue 实现多进程之间的数据传递，Queue 本身是一个消息列队程序，与Queue.Queue 不同，Queue.Queue 是进程内非阻塞队列，multiprocess.Queue 是跨进程通信队列。多进程前者是各自私有，后者是各子进程共有。

我们以 multiprocess.Queue 为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：
```
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据
def read(q):
    while True:
        if not q.empty():
            value = q.get(True)
            print('Get %s from queue.' % value)
            time.sleep(random.random())
        else:
            break

if __name__=='__main__':
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    pw.start()    
    pw.join()
    pr.start()
    pr.join()
    print('所有数据都写入并且读完')
```

结果如下:

```
Put A to queue...
Put B to queue...
Put C to queue...
Get A from queue.
Get B from queue.
Get C from queue.
所有数据都写入并且读完
```

另外，另外队列中常用的方法有：

 - Queue.qsize() 返回队列的大小
 - Queue.empty() 如果队列为空，返回True, 反之False
 - Queue.full() 如果队列满了，返回True,反之False
 - Queue.get([block[, timeout]]) 获取队列，timeout等待时间
 - Queue.get_nowait() 相当Queue.get(False)
 - Queue.put(item) 阻塞式写入队列，timeout等待时间
 - Queue.put_nowait(item) 相当Queue.put(item, False)

### 4. Pipe
管道，顾名思义，一端发一端收。之前我们介绍过，由于多进程缺乏同步机制，因此不得不采取消息传递机制，在进程间传递复制的数据。在这里，Pipe可以是单向(half-duplex)，也可以是双向(duplex)。我们通过mutiprocessing.Pipe(duplex=False)创建单向管道 (默认为双向)。

一个进程从PIPE一端输入对象，然后被PIPE另一端的进程接收，单向管道只允许管道一端的进程输入，而双向管道则允许从两端输入。下面用一个实例来感受一下：

```
from multiprocessing import Process,Pipe

class Consumer(Process):

    def __init__(self,pipe):
        Process.__init__(self)
        self.pipe = pipe
    def run(self):
        self.pipe.send('Consumer Words')
        print('Consumer Received:',self.pipe.recv())

class Producer(Process):
    def __init__(self,pipe):
        Process.__init__(self)
        self.pipe = pipe
    def run(self):
        print('Producer Received:',self.pipe.recv())
        self.pipe.send('Producer Words')

pipe = Pipe()
p = Producer(pipe[0])
c = Consumer(pipe[1])
p.daemon = c.daemon = True
p.start()
c.start()
p.join()
c.join()
print('Ended!')
```

在这里声明了一个默认为双向的管道，然后将管道的两端分别传给两个进程。两个进程互相收发。观察一下结果：

```
Producer Received: Consumer Words
Consumer Received: Producer Words
Ended!
```

以上就是对管道的简单介绍。

### 5. Pool
在利用Python进行系统管理的时候，特别是同时操作多个文件目录，或者远程控制多台主机，并行操作可以节约大量的时间。当被操作对象数目不大时，可以直接利用multiprocessing中的Process动态成生多个进程，十几个还好，但如果是上百个，上千个目标，手动的去限制进程数量却又太过繁琐，此时可以发挥进程池的功效。

Pool可以提供指定数量的进程，供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来它。

在这里需要了解阻塞和非阻塞的概念。阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态。阻塞即要等到回调结果出来，在有结果之前，当前进程会被挂起。

Pool的用法有阻塞和非阻塞两种方式。非阻塞即为添加进程后，不一定非要等到改进程执行完就添加其他进程运行，阻塞则相反。

```
from multiprocessing import Process, Pool

function_list = [worker_1,worker_2,worker_3,worker_4,worker_0]
a = time.time()
pool = multiprocessing.Pool()
for func in function_list:
    pool.apply_async(func,(1,))
pool.close()
pool.join()
b = time.time()
print('finsh:', b-a)
```
在这里利用了apply_async方法，即非阻塞,可以发现在这里添加三个进程进去后，立马就开始执行，不用非要等到某个进程结束后再添加新的进程进去。

结果如下:

```
工作212
工作four
工作three
工作1
end工作four
错误的工作
end工作212
end工作three
end工作1
finsh: 2.141101837158203
```

如果要使用阻塞，在这里只需要把apply_async改成apply即可。

下面对相关函数进行解释：

 - apply_async(func[, args[, kwds[, callback]]]) 它是非阻塞。
 - apply(func[, args[, kwds]])是阻塞的。
 - close() 关闭pool，使其不在接受新的任务。
 - terminate() 结束工作进程，不在处理未完成的任务。
 - join() 主进程阻塞，等待子进程的退出，join方法要在close或terminate后使用。

当然每个进程可以在各自的方法返回一个结果。apply或apply_async方法可以拿到这个结果并进一步进行处理。

如果现在有一堆数据要处理，每一项都需要经过一个方法来处理，那么map非常适合。

比如现在你有一个数组，包含了所有的URL，而现在已经有了一个方法用来抓取每个URL内容并解析，那么可以直接在map的第一个参数传入方法名，第二个参数传入URL数组。现在我们用一个实例来感受一下：

```
from multiprocessing import Process, Pool
import requests
from requests.exceptions import ConnectionError

def scrape(url):
    try:
        print(requests.get(url))
    except ConnectionError:
        print('Error Occured',url)
    finally:
        print('URL',url,'Scraped')
pool = Pool()
urls = [
    'https://www.baidu.com',
    'http://www.meituan.com/',
    'http://blog.csdn.net/',
    'http://xxxyxxx.net'
]
pool.map(scrape,urls)
```

在这里初始化一个Pool，指定进程数为3，如果不指定，那么会自动根据CPU内核来分配进程数。然后有一个链接列表，map函数可以遍历每个URL，然后对其分别执行scrape方法。

运行结果如下：

```
<Response [200]>
URL https://www.baidu.com Scraped
Error Occured http://xxxyxxx.net
URL http://xxxyxxx.net Scraped
<Response [200]>
URL http://blog.csdn.net/ Scraped
<Response [200]>
URL http://www.meituan.com/ Scraped
Out[10]:
[None, None, None, None]
```

以上便是 multiprocessing 模块的简单介绍。