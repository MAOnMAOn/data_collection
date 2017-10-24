## 2.10.1 Socket 编程实例

Socket 起源于 Unix ，Unix/Linux 基本哲学之一就是“一切皆文件”，都可以用“打开(open) –> 读写(write/read) –> 关闭(close)”模式来进行操作。因此 Socket 也被处理为一种特殊的文件。本节我们就通过相关实例来对这种文件进行'处理'，让我们一起体验一下吧。

### 1. Socket 类型
socket 类型在 Linux 和 Python 之中是一样的，只是 Python 中的类型都定义在 socket 模块中，调用方式为 socket.SOCK_XXX，类型主要为如下三种：

* 流式 socket (SOCK_STREAM)
>流式套接字提供可靠的、面向连接的通信流；其使用 tcp 协议，从而保证了数据传输的准确性与顺序性。

* 数据报 socket (SOCK_DGRAM)
>数据报套接字定义了一种无连接的服务，数据通过相互独立的报文进行传输，是无序的，并且不保证是可靠的、无差错的。其使用数据报协议 UDP。

* 原始 socket (SOCK_RAW)
原始套接字，普通的套接字无法无法处理 ICMP、IGMP 等网络报文，而 SOCK_RAW 可以；其次，SOCK_RAW 也可以处理特色的 IPV4 报文；此外，利用原始套接字，可以通过 IP_HDRINCL 套接字选项由用户构造 ip 头。

### 2. TCP 通信
一般而言，socket的 TCP 通信主要有如下流程：

![](/assets/tcpscoket.png)

针对以上流程的几个步骤，python 之中也给出了相应的方法进行处理：

1. socket 函数
使用给定的地址族、套接字类型、协议编号(默认为0)来创建套接字。

>socket.socket([family[, type[, proto]]])
>family: AF_INET (默认为ipv4)，AF_INET6 (ipv6)，AF_UNIX(Unix系统进程间通信)
>type : SOCK_STREAM(TCP), SOCK_DGRAM(UDP)
>protocol: 一般为0或者默认
>如果 socket 创建失败会抛出一个 socket.error 异常

#### (1) 服务器端函数
1. bind 函数
将套接字绑定到地址，python 中，以元祖(host, port)的形式表示地址，Linux下使用 sockaddr_in 结构体指针。

>socket.socket().bind(address)
>address为元组 (host,port)
>host: ip 地址，为一个字符串
>port: 自定义主机号，为整数类型

2. listen 函数
是服务器的这个端口与ip处于监听状态，等待网络中的某一客户机的连接请求。若客户端有连接请求，端口就会接收这个连接。

>socket.socket().listen(backlog)
>backlog: 操作系统可以挂起的最大连接数量。该值至少为1，大部分应用程序设为5就可以了

3. accept 函数
接收远程计算机的连接请求，建立起与客户机之间的通信连接。服务求处于监听状态时，如果某手机壳获得客户机的连接请求，此时并非立刻处理该请求，而是把请求放在等待队列之中，当系统空闲时再处理客户机的连接请求。

>socket.socket().accept()
>返回 (conn, address),其中 conn 为新的套接字对象，可以用来接收和发送数据。address 是连接客户端的地址的地址。

#### (2) 客户端函数
1. connect 函数
用来请求连接远程服务器。

>socket.socket(address)
>address: 格式为元组(hostname,port)，如果连接出错，返回socket.error 错误

#### (3) 通用函数
1. recv 函数
用于接收远程主机传来的数据。

>socket.socket().recv(bufsize[,flag])
>bufsize: 指定要接收的数据的大小
>flag: 提供有关消息的其他信息，通常可以忽略

2. send 函数
发送数据给指定的远程主机

>socket.socket().send(string[,flag])
>string: 要发送的字符串数据
>flag: 提供有关消息的其他信息，通常可以忽略
>返回值为要发送的字节数量，该数量可能小于 string 的字节大小

>socket.socket().sendall(string[,flag])
>完整发送 TCP 数据，将字符串中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据
>返回值：成功返回 None，失败则抛出异常。

3. close 函数
关闭套接字

>socket.socket().close()

#### (4) 简单的客户端服务器 TCP 连接
这里编写一个简单的回显服务器和客户端模型，客户端发出的数据，服务器会回显到客户端的终端上(只是一个简单的模型，没考虑错误处理等问题),新建２个 python 文件

```
# 服务端文件
import socket
import subprocess

BUF_SIZE = 1024  # 设置缓冲区大小
server_addr = ('127.0.0.1', 8887)  # ip与端口构成地址
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # 生成一个新的　socket 对象
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) # 设置地址复用
server.bind(server_addr) # 绑定地址
server.listen(5)  # 监听，最大监听数为５
print('ok')
while True:
    client, client_addr = server.accept()  
    print('Connected by', client_addr)
    while True:
        data = client.recv(BUF_SIZE)
        print(data)
        client.sendall(data)
server.close()
```

```
# 客户端文件
import socket

BUF_SIZE = 1024
server_addr = ('127.0.0.1', 8887)
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 返回新的 socket 对象
client.connect(server_addr) # 要连接的服务器地址
while True:
    data = input('Please input some string >')
    client.sendall(data.encode('utf8')) 
    data = client.recv(BUF_SIZE)
    print(data)
client.close()
```

#### (5) 带错误处理的客户端服务器TCP连接
在进行网络编程时, 最好使用大量的错误处理, 能够尽量的发现错误, 也能够使代码显得更加严谨。

```
# 服务端
import socket
import sys

BUF_SIZE = 1024
server_addr = ('127.0.0.1', 8882)  #IP和端口构成表示地址
try:
    print('ok')
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # 生成新的 socket 对象
except socket.error:
    print("Creating Socket Failure")
    sys.exit()

print('Socket Created!')
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) # 设置地址复用
try:
    server.bind(server_addr) # 绑定地址
except socket.error:
    print("Creating Socket Failure")
    sys.exit()
    
print('Socket Bind!')
server.listen(5) # 监听，最大监听数为５
print('Socket listening')
while True:
    client, client_addr = server.accept()  # 接收 TCP 连接，并返回新的套接字与地址，阻塞函数
    print('Connected by', client_addr)
    while True:
        data = client.recv(BUF_SIZE)  # 从客户端接收数据
        print(data.decode('utf8'))
        client.sendall(data)
server.close()  
```

```
# 客户端文件
import sys
import socket

BUF_SIZE = 1024
server_addr = ('127.0.0.1', 8883)  #IP和端口构成表示地址
try:
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # 返回新的 socket 对象

except socket.error:
    print("Creating Socket Faile")
    sys.exit()
client.connect(server_addr)  # 要连接的服务器地址
while True:
    data = input("Please input some string > ")
    if not data:
        print("input can't empty, Please input again..")
        continue
    client.sendall(data.encode('utf8'))   #发送数据到服务器
    data = client.recv(BUF_SIZE)
    print(data.decode('utf8'))
client.close()
```
