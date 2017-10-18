## 8.3x concurrent.futures进程池

之前我们使用多线程(threading)和多进程(multiprocessing)完成常规的需求，在启动的时候start、jon等步骤不能省，复杂的需要还要用1-2个队列。随着需求越来越复杂，如果没有良好的设计和抽象这部分的功能层次，代码量越多调试的难度就越大。有没有什么好的方法把这些步骤抽象一下呢，让我们不关注这些细节，轻装上阵呢？

答案是：有的。

### （1）案列引入
从Python3.2开始一个叫做concurrent.futures被纳入了标准库，该模块中有2个类：ThreadPoolExecutor 和 ProcessPoolExecutor，也就是对 threading 和 multiprocessing 进行了高级别的抽象，暴露出统一的接口，帮助开发者非常方便的实现异步调用。下面来感受一下：

```
import time
from concurrent.futures import ProcessPoolExecutor


def fib(n):
    if n <= 2:
        return 1
    return fib(n - 1) + fib(n - 2)

def pool_factory(func, data, max_proc=3):
    with ProcessPoolExecutor(max_workers=max_proc) as excutor:
        res = {
            num: result
            for num, result in zip(data, excutor.map(func, data))
        }
        return res

start = time.time()
NUMBERS = range(15, 32)
res = pool_factory(fib, NUMBERS)

for el in res:
    print("fib({}) = {}".format(el, res[el]))
print('COST: {}'.format(time.time() - start))
```

代码运行结果：

```
fib(15) = 610
fib(16) = 987
fib(17) = 1597
fib(18) = 2584
fib(19) = 4181
fib(20) = 6765
fib(21) = 10946
fib(22) = 17711
fib(23) = 28657
fib(24) = 46368
fib(25) = 75025
fib(26) = 121393
fib(27) = 196418
fib(28) = 317811
fib(29) = 514229
fib(30) = 832040
fib(31) = 1346269
COST: 0.804440975189209
```
是不是有一种轻便的感觉呢？除了我们上面用到的 map，另外一个常用的方法是 submit 。如果你要提交的任务的函数是一样的，就可以简化成 map。但是假如提交的任务函数是不一样的，或者执行的过程之可能出现异常（使用 map执行过程中发现问题会直接抛出错误）就要用到 submit：

```
from concurrent.futures import ProcessPoolExecutor, as_completed

def fib(n):
    if n == 31:
        raise Exception("fib must less than 31")
    if n <= 2:
        return 1
    return fib(n - 1) + fib(n - 2)

def pool_factory(func, data, max_proc=3):
    with ProcessPoolExecutor(max_workers=max_proc) as excutor:
        excutor_dict = {excutor.submit(func, num): num for num in data}
        res_dict = {}
        for excutor_run in as_completed(excutor_dict):
            num = excutor_dict[excutor_run]

            try:
                result = excutor_run.result()
            except Exception as e:
                print("Raise error: {}".format(e))
            else:
                res_dict[num] = result
        return res_dict

start = time.time()
NUMBERS = range(15, 32)
res = pool_factory(fib, NUMBERS)

for el in res:
    print("fib({}) = {}".format(el, res[el]))
print('COST: {}'.format(time.time() - start))


def pool_factory(func, data, max_proc=3):
    with ProcessPoolExecutor(max_workers=max_proc) as excutor:
        res = {
            num: result
            for num, result in zip(data, excutor.map(func, data))
        }
        return res

pool_factory(fib, NUMBERS)
```

运行结果：

```
Raise error: fib must less than 31
fib(16) = 987
fib(15) = 610
fib(17) = 1597
fib(18) = 2584
fib(19) = 4181
fib(20) = 6765
fib(21) = 10946
fib(22) = 17711
fib(23) = 28657
fib(24) = 46368
fib(25) = 75025
fib(26) = 121393
fib(27) = 196418
fib(28) = 317811
fib(29) = 514229
fib(30) = 832040
COST: 0.5278811454772949
---------------------------------------------------------------------------
_RemoteTraceback                          Traceback (most recent call last)
_RemoteTraceback:
"""
Traceback (most recent call last):
  File "/home/lv/softwares/anaconda3/lib/python3.6/concurrent/futures/process.py", line 175, in _process_worker
    r = call_item.fn(*call_item.args, **call_item.kwargs)
  File "/home/lv/softwares/anaconda3/lib/python3.6/concurrent/futures/process.py", line 153, in _process_chunk
    return [fn(*args) for args in chunk]
  File "/home/lv/softwares/anaconda3/lib/python3.6/concurrent/futures/process.py", line 153, in <listcomp>
    return [fn(*args) for args in chunk]
  File "<ipython-input-13-c7b473b85dd7>", line 5, in fib
    raise Exception("fib must less than 31")
Exception: fib must less than 31
"""
```
可以看到，第一次捕捉到了异常，但是第二次执行的时候错误直接抛出来了。

### （2）Future
Future是常见的一种并发设计模式，在多个其他语言中都可以见到这种解决方案。

一个Future对象代表了一些尚未就绪（完成）的结果，在「将来」的某个时间就绪了之后就可以获取到这个结果。比如上面的例子，我们期望并发的执行一些参数不同的fib函数，获取全部的结果。传统模式就是在等待queue.get返回结果，这个是同步模式，而在Future模式下，调用方式改为异步，而原先等待返回的时间段，由于「Local worker thread」的存在，这个时候可以完成其他工作。

### （3）<small>multiprocessing.Pool vs concurrent.futures.PoolExecutor</small>

concurrent.futures 的架构明显要复杂一些，不过更利于写出高效、异步、非阻塞的并行代码，而且 concurrent.futures 的接口更简单一些。其参数就一个max_workers。
由于concurrent.futures底层还是用着threading和multiprocessing，相当于在其上又封装了一层，并且重新设计了架构，所以会慢一点。

但是如果要处理的是一个很大的可迭代对象，就会有非常大的性能差别。这是因为multiprocessing.Pool是批量提交任务的，可以节省IPC(进程间通信)开销。而ProcessPoolExecutor每次都只提交一个任务。不过在Python3.5的时候已经通过给map方法添加chunksize参数解决了。

你可能比较迷惑，我们看一段代码就好了：

```
import time
from multiprocessing.pool import Pool
from concurrent.futures import as_completed, ProcessPoolExecutor

NUMBERS = range(1, 100000)
K = 50


def f(x):
    r = 0
    for k in range(1, K+2):
        r += x ** (1 / k**1.5)
    return r
print('multiprocessing.pool.Pool:\n')

#part1
start = time.time()
l = []
pool = Pool(3)
for num, result in zip(NUMBERS, pool.map(f, NUMBERS)):
    l.append(result)
print(len(l))
print('COST: {}'.format(time.time() - start))
print('ProcessPoolExecutor without chunksize:\n')

#part2
start = time.time()
l = []
with ProcessPoolExecutor(max_workers=3) as executor:
    for num, result in zip(NUMBERS, executor.map(f, NUMBERS)):
        l.append(result)
print(len(l))
print('COST: {}'.format(time.time() - start))
print('ProcessPoolExecutor with chunksize:\n')

#part3
start = time.time()
l = []
with ProcessPoolExecutor(max_workers=3) as executor:
    chunksize, extra = divmod(len(NUMBERS), executor._max_workers * 4)
    for num, result in zip(NUMBERS, executor.map(f, NUMBERS, chunksize=chunksize)):
        l.append(result)

print(len(l))
print('COST: {}'.format(time.time() - start))
```

运行结果：

```
multiprocessing.pool.Pool:

99999
COST: 1.0184590816497803
ProcessPoolExecutor without chunksize:

99999
COST: 20.88278365135193
ProcessPoolExecutor with chunksize:

99999
COST: 1.0761966705322266
```

这里第一段代码使用multiprocessing.pool，最快。第二段使用没有加chunksize， 这个速度不忍直视，大家不要再这样犯错了。第三段加上了chunksize，分块的原则和第一段标准库实现的一样，速度又回到同一个水平了。


### （4）小结
显然用futures的写法上更简洁一些，concurrent.futures的性能并没有更好，只是让编码变得更简单。考虑并发编程的时候，任何简化都是好事。从长远来看，concurrent.futures编写的代码更容易维护。

使用map时，future是逐个迭代提交，multiprocessing.Pool是批量提交jobs，因此对于大批量jobs的处理，multiprocessing.Pool效率会更高一些。对于需要长时间运行的作业，用future更佳，future提供了更多的功能（callback, check status, cancel）。

concurrent.futures.ProcessPoolExecutor是对multiprocessing的封装，在运行时需导入\__main__，不能直接在交互窗口工作。

concurrent.futures 之中还有一个 ThreadPoolExecutor 模块，用法ProcessPoolExcutor 类似，大家有兴趣可以进行进一步了解。