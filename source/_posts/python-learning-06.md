---
title: Python学习总结06——multiprocessing
date: 2019-08-05 21:55:32
mathjax: true
tags:
 - Code
categories:
 - Python
 - Basic
---

Python的多进程，官网上multiprocessing的文档，比我在网络上看到的其他一些博客和教程写得都要好，给出链接：{% link 点击这里 https://docs.python.org/3/library/multiprocessing.html#multiprocessing-programming link %}，建议浏览英文文档，或者中英文对照看。本文以下的内容，我基本上按照官方文档的结构进行展开，对其中部分内容给出自己的理解与总结。

### 概述

> multiprocessing is a package that **supports** spawning processes using an API similar to the threading module. The multiprocessing package **offers** both local and remote concurrency, effectively side-stepping the Global Interpreter Lock by using subprocesses instead of threads. Due to this, the multiprocessing module allows the programmer to fully leverage multiple processors on a given machine. It runs on both Unix and Windows.

这里我直接将官网原文引用了过来，部分单词加粗强调当然是为了引起重视，multiprocessing 模块是支持创建进程，注意是**支持**，而不是**用于**，它包含了创建进程的能力，同时也**提供**了本地和远端的 python 进程同步的能力。如果说 multiprocessing 就是用来生成多进程的，显然不全面；同时注意一下文档有写它支持本地与远端进程间的同步，并不意味着它可以在远端服务器上创建进程。

### 进程的启动

在 multiprocessing 中，通过创建一个 Process 对象然后调用它的 start() 方法来生成进程，根据不同的平台， multiprocessing 支持三种启动进程的方法：spawn、fork 和 forkserver。spawn 直接新启动一个 python 解释器进程；fork 和 forkserver 都是调用了 unix 底层的 fork 函数，区别是 fork 会对当前进程的资源进行拷贝，而 forserver 先创建一个所谓 server 进程，再对 server 进程进行 fork，这样forserver 创建的子进程就不包含当前进程资源的拷贝。

尝试执行以下的代码，将第8行 get_context 方法中的参数分别修改成 forkserver 和 spawn，能看到执行结果的差别。还可以进一步尝试将第6行的“if __name__ == '__main__':” 去掉，看程序能否正常执行，分别采用 spawn、fork 和 forkserver 启动进程时有什么不同的执行结果。

```python
import multiprocessing

def f():
    print(q)

if __name__ == '__main__':
    q = 1
    ctx = multiprocessing.get_context('fork')
    p = ctx.Process(target=f)
    p.start()
```

当线程的启动方式为 fork 时，以上的程序能够正常执行并且打印结果为1，原因就是 fork 创建的子进程对父进程进行了拷贝，其中就包含 q，其值为1。

以上的程序通过 get_context() 来获取 context，使用 context 来设置进程的启动方式，这也可以通过 multiprocessing.set_start_method() 来设置，但是 set_start_method() 函数的效果是全局性的，当你的程序需要打包成 package 提供给别人使用时，建议使用 context 。还需要注意文档中使用 context 创建的对象在别的 context 创建的进程中可能不兼容，理解这句话可以参考以下的代码：

```python
import multiprocessing

def producer(q):
    for i in range(10):
        q.put("包子 %s" % i)
    print("开始等待顾客买包子...")
    q.join()
    print("所有的包子被取完了...")

def consumer(n):
    while q.qsize() == 0:
        pass
    while q.qsize() > 0:
        print("%s 买到" % n, q.get())
        q.task_done()

if __name__ == '__main__':
    ctx = multiprocessing.get_context('spawn')
    q = ctx.JoinableQueue() # 将这一行修改成 multiprocessing.JoinableQueue()试试
    p = ctx.Process(target=producer, args=(q,))
    p.start()
    c1 = consumer("小王")
```

### 队列，管道和共享内存

python进程的队列，是一个被封装的很重的东西，官网上有下面一段note：

> **Note:** When an object is put on a queue, the object is pickled and a background thread later flushes the pickled data to an underlying pipe. This has some consequences which are a little surprising, but should not cause any practical difficulties – if they really bother you then you can instead use a queue created with a manager.

进程 queue 基于底层的 pipe 事项，当数据放入队列和从队列取出时，它经历了一个序列化和反序列化的过程。当一个东西经过了过多的封装，我就比较倾向于认为它不太可靠，或者性能上存在问题，或者功能上尤其是并发上可能存在缺陷。不过在注重功能实现，不需要考虑工程压力的时，进程 queue 还是非常简单方便的，它的用法基本和线程 queue 相同，就不再赘述了，可以参考官网。

进程间通信，基本上适用以下的几个套路：

1.socket： 比较底层，想用肯定能用上，几乎所有的语言都支持，实现比较复杂
2.共享内存： 依赖了语言和框架，比如 python 可以在父子进程设置共享内存
3.pipe： 以来系统，语言和框架

*个人总结，仅供参考*

接下来着重看一下 pipe 和共享内存：

#### 管道 Pipe

multiprocessing.Pipe([duplex]) 返回一对 Connection 对象 (conn1, conn2) 分别代表管道的两端，如果 duplex 传 True，那么这个管道是双向的，如果 duplex 传 False，那么该管道为单向的，单向的管道 conn1 只能被用于接收信息，conn2 只能被用作发送信息。想要用会 Pipe，需要熟悉 Connection 类，此外正如官网所介绍的，multiprocessing.connection 还有一些更加灵活高级的用法，了解它们可以帮助你应对更加复杂的场景。

```python
from multiprocessing import Pipe
a, b = Pipe()
a.send([1, 'hello', None])
print(b.recv())
b.send_bytes(b'thank you')
print(a.recv_bytes())
```

通过上面的例子，可以看到 Pipe 的使用并不是和多进程绑定在一起的，上例中使用了 Pipe 但是没有创建多进程，python 其他很多模块设计也体现着这一特点。

#### 共享内存

>It is possible to create shared objects using shared memory which can be inherited by child processes.

可以通过使用共享内存创建共享对象，在线程与其创建的子线程之间进行共享。内存共享通过创建 ctypes 对象来达成，在 multiprocessing 模块下，有 Value 和 Array 两个函数返回用于共享内存的 ctypes 对象；在 multiprocessing.sharedctypes 模块下，有 RawArray、RawValue、Array、Value 四个函数返回共享内存的 ctypes 对象。

*我没太搞懂 multiprocessing.Value 和 multiprocessing.sharedctypes.Value 之间的区别*

具体的实例请参加：{% link 官网 https://docs.python.org/3.7/library/multiprocessing.html#module-multiprocessing.sharedctypes link %}

#### Manager

> Managers provide a way to create data which can be shared between different processes, including sharing over a network between processes running on different machines. A manager object controls a server process which manages shared objects. Other processes can access the shared objects by using proxies.

谈到 Manager 模块，这可以说是 Python 在进程共享数据上大招，它提供的进程间的数据共享，不仅仅限于 Python 的父子进程间，还包含本机不用的 Python 进程间的数据共享，以及两个不同机器的进程间通过网络的数据共享。概述中提到的 multiprocessing 提供了远程进程间的数据共享能力，指的就是它。

目前我知识将官网上的 Manager 一节粗浅的浏览了一下，深入的学习和总结留待以后用到再说了，因为确实 Manager 相对复杂一些，知识点更加细节。如果仅仅是需要创建子进程，实现数据同步，或者进程间同步，不依赖 Manager 也能实现；如果有场景，需要在多个不相干的 Python 进程间进行通信，可以考虑它。

#### Programming guidelines

最后建议看一下官网的 Programming guidelines，里面有一些非常好的编程建议以及最佳实践，唯一的缺憾是目前还没有中文翻译。

*有时间的话我来翻译一下*


