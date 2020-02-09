---
title: Python学习总结04——socket
date: 2019-07-14 15:13:44
mathjax: true
tags:
 - Code
categories:
 - Python
 - Basic
---


### 写在前面的

忘记了是在大学的Java课上，还是在计算机网路课上，第一次接触Socket，当时似懂非懂，有一种不明觉厉的感觉，后来毕业找工作笔试的时候还想拿出来秀一下，Copy网上的实现写了一个最为简单的Socket，然而并没有什么卵用。我们学习的时候，还是要务实一些，尽量搞清楚一个东西的原理以及应用场景，不然头脑里只是一些范范的知识，只会瞎扯和空谈，是没有任何用处的。

那么Socket到底是什么呢，它是编程接口，是对TCP/IP的封装，{% link Python官网Socket一节 https://docs.python.org/3/library/socket.html link %}有一个副标题“Low-level networking interface”，直译过来就是：底层的网络接口。Unix、Windows和MacOS系统均提供了基于BSD规范的Socket接口，Python的Socket类其实是对系统Socket接口的进一步封装。通过Socket，我们可以实现应用层的一些协议，例如ftp和http，或者定义一套自己通信标准，从而实现本机进程间乃至不同主机间的通信。

不得不说，Python将Socket封装的很好，以下是一个简单的Socket实现。

### 从实例开始

服务端：

```Python
import socket

server = socket.socket()
server.bind(('0.0.0.0', 6971))
server.listen(5)

while True:
    conn, addr = server.accept()
    print(conn, addr)

    while True:
        data = conn.recv(1024)
        if len(data) == 0: #在Windows平台上，客户端断连会抛出异常
            print('client is closed')
            break
        print('received data', data.decode('utf-8'))
        conn.send(data.decode('utf-8').upper().encode('utf-8'))

    conn.close()
server.close()
```

客户端：

```Python
import socket

client = socket.socket()
client.connect(('localhost', 6971))

while True:
    data = input("please input >>")
    if len(data) == 0:
        continue
    client.send(data.encode('utf-8'))
    data = client.recv(1024)
    print(data.decode('utf-8'))

client.close()
```

以上代码实现了一组Socket应用，服务端接收客户端请求，将客户端传过来的字符串转换为大写之后，再传回给客户端。先从整体看一下Socket服务端程序，首先创建一个socket对象，然后绑定IP地址和端口号，启动监听，接受客户端连接，然后开始接收数据。然后按照顺序看局部：

**1. 创建socket对象 - socket.socket**

socket.socket()函数，在参数为空时，使用默认的参数，该函数声明如下：

```Python
socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

第一个参数为Socket Families(地址簇)，常量名称以“AF_”打头，从官网和源码中可以看到很多的类型，我目前学习接触到的有“socket.AF_INET”和“socket.AF_INET6”，分别代表了IPV4和IPV6，其他的类型以后有用到再补充；第二个参数为Socket Types，当地址簇为“socket.AF_INET”和“socket.AF_INET6”时，常用的Socket Type有以下的几种：

|Socket Type|说明|
|:--|--|
|socket.SOCK_STREAM    |for tcp|
|socket.SOCK_DGRAM     |for udp| 
|socket.SOCK_RAW       |原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而SOCK_RAW可以；其次，SOCK_RAW也可以处理特殊的IPv4报文；此外，利用原始套接字，可以通过IP_HDRINCL套接字选项由用户构造IP头。|
|socket.SOCK_RDM       |是一种可靠的UDP形式，即保证交付数据报但不保证顺序。SOCK_RAM用来提供对原始协议的低级访问，在需要执行某些特殊操作时使用，如发送ICMP报文。SOCK_RAM通常仅限于高级用户或管理员运行的程序使用。|

**2. 绑定地址和端口 - socket.bind**

注意传入 socket.bind 函数的参数为一个元组，原因我推测是 socket.socket() 创建socket对象时，如果传入其他的Family和Type，socket.bind 函数的入参会有很大的差别，所以索性将 socket.bind 函数的入参定义为元组的形式。

server.bind(('0.0.0.0', 6971)) 表示绑定本机**所有IP地址**的6871端口，如果你的服务器配置又多个IP地址，而你想要限定对其中某个IP地址开启Socket服务，可以将 '0.0.0.0' 替换为你需要开启Socket服务的IP地址。

**3. 启动端口监听 - socket.listen**

这个函数在不同的环境下有不同的效果，在Ubuntu 18.04 + Python3.6的环境执行以上程序，启动多个客户端，会有1个客户端与服务端建立起了连接并且可以交互，此外会有6个客户端阻塞在第10行 client.send 前，其他启动的客户端则阻塞在第4行 client.connect。如果将服务端 server.listen(5) 改为 server.listen(0)，则会有1个客户端阻塞在第10行 client.send 前。

对此我是这样理解的，server.listen 函数定义了一个等待交互的客户端队列长度，队列的长度为参数 n + 1，已经与服务端建立了连接并且正在通信的client不在队列中。所以当 server.listen(5) 时，有6个客户端被Python解释器阻塞在了connect之后，send之前；其余的则被系统或者Python解释器阻塞在了connect一步。

文字表述比较难懂，建议在自己的环境上执行一下试试。

**4. 接收客户端数据 - conn.recv**

socket.accept 函数接收了一个客户端连接，返回一个 conn 连接对象，以及客户端地址。conn.recv 接收客户端发送过来的数据，conn.recv(1024) 的参数1024表示最多接收1024字节，可以将参数写大一些用于一次性接收更多的数据，但是一次性接收的数据不可能无限多，所以将这个参数配置得很大是没有意义的。一次最多接收多少数据，不同的环境有不同的限制。

**5. 判断客户端是否断连**

服务端程序第13行，我们通过 len(data) == 0 判断客户端是否断连，这个判断在Ubuntu系统上有效，在Windows系统中，如果客户端断连，服务端会抛出异常，需要对这个异常进行捕获。

**6. 最后说说客户端**

客户端的实现比较简单，首先建立连接，然后循环发送数据。注意第8行的判断，如果用户输入为空，continue 进入下一轮循环，原因是这样的：用户输入的数据为空时，客户端的确会通过 socket 发出一个空的数据，但是服务端收不到这个空的数据。

这样一来，客户端认为自己已经发送了，阻塞在11行 data = client.recv(1024) 等待服务端的响应，而服务端没有接收到任何数据，自然也不会发送响应，最后的结果就是程序卡死。为了规避该问题，我们在Socket发送数据前，先坚持即将发送的数据是否为空，如果将要发送的数据为空，则不再发送。


### 处理粘包

“粘包”通俗一点说，就是服务端发送了多个包，但是客户接收的时候，多个包粘在了一起，变成了一个包。粘包并不复杂，要明确首先粘包并没有丢包，所有的数据包都在缓冲区区里面；其次粘包并没有重包，没有一份数据在缓冲区里出现了多次，还需要你自己去重。

为什么会出现粘包，有下面两点原因: （转载自他人网页，{% link 链接 https://blog.csdn.net/gengbaolong/article/details/75450208 link %} ）
>1、发送端需要等缓冲区满才发送出去，造成粘包 (发送端出现粘包)
>2、接收端没有及时接收缓冲区包数据，造成一次性接收多个包，出现粘包 (接收端出现粘包)

理解了粘包的原理之后，处理起来并不困难，总结了一下处理Python的粘包有三种方法：

1. 发送第一个包后，第二个包前sleep一段时间，导致缓冲区超时，这种处理方法的问题就是造成了程序的延时。
2. 发送第一个包后，等到另一端确认接收完毕，再发送第二个数据包。
3. 发送一个结构体，让另一端明确需要接收的数据包长度，因而只接收固定长度的数据包。

### SocketServer

SocketServer是Python对简单socket进行的一个服务端的封装，本文最开始的示例中的Socket服务端程序，只能与单个客户端进行交互，可以使用 socketserver.ThreadingTCPServer 进行改写，用很少的代码，就能实现一个支持多线程，可以与多个客户端同时进行交互的Socket服务端程序。

示例如下：

``` Python
import socketserver

class SimpleHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            data = self.request.recv(1024)
            if len(data) == 0:
                print('client is closed')
                break
            print('received data', data.decode())
            self.request.send(data.decode().upper().encode())

if __name__ == '__main__':
    server = socketserver.ThreadingTCPServer(('localhost', 6971), SimpleHandler)
    server.serve_forever()
```

更为详细的SocketServer使用方法，在网上可以搜到很多，更建议的{% link 参考官网 https://docs.python.org/3/library/socketserver.html#socketserver.ThreadingMixIn link %}，带着具体的问题去寻找答案。

个人基于Python Socket写的一个Ftp程序：{% link 链接 https://github.com/yiayiao/approach-to-python/tree/master/socket-learning link %}
