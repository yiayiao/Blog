---
title: Python学习总结05——threading
date: 2019-08-01 08:08:32
mathjax: true
tags:
 - Code
categories:
 - Python
 - Basic
---

### 了解进程与线程

想要了解线程，就绕不开进程。

#### 什么是进程

>An executing instance of a program is called a process.
>Each process provides the resources needed to execute a program. A process has a virtual address space, executable code, open handles to system objects, a security context, a unique process identifier, environment variables, a priority class, minimum and maximum working set sizes, and at least one thread of execution. Each process is started with a single thread, often called the primary thread, but can create additional threads from any of its threads.

翻译：一个正在执行的程序的实例被称之为进程。每一个进程都提供了程序允许所需要的资源，一个进程包含该一个虚拟地址空间，可执行的代码，连接系统其他对象的handle，一个安全上下文，一个独立的进程编号，环境变量，优先级（priority class），最大和最小的工作空间，以及至少一个正在执行的线程。每一个进程启动时都伴随了一个线程，通常称之为主线程，进程中的每一个线程都可以再创建其他的线程。

#### 什么是线程

>A thread is an execution context, which is all the information a CPU needs to execute a stream of instructions.

翻译：线程是一个执行的上下文，包含了CPU需要执行的一系列指令的所有信息。

*个人翻译未必准确，建议参考英文*

以上是我能够查到的，对进程和线程的比较官方比较权威比较可信的解释，这句话我琢磨了比较久，用了三个比较，因为计算机到底是一门人造学科，很多概念在大家定义和实践它时，在不同人之间有所差别。以上是一个比较通用，大家比较接收的定义，写Python时用到的进程和线程完全符号以上定义，而对Linux系统来说有人就有了其他的看法，比如“Linux没有线程”，在Windows系统上讨论以上的进程和线程的定义就又不一样了。

从网上看到了一篇讲Linux的进程与线程的通俗易懂的好文，链接请点{% link 这里 https://my.oschina.net/cnyinlinux/blog/422207 link %}，我认为Linux是有线程的，部分观点认为Linux没有线程，因为创建进程和线程都调用系统底层的clone接口，只是参数不同。但是我看到这一点参数的不同给创建的进程与线程带来了实质的差别——是否与父进程共享空间。另外链接中的博文里面也提到创建线程时并不采用clone系统调用，而是采用线程库函数。常用线程库有Linux-Native线程库和POSIX线程库。其中应用最为广泛的是POSIX线程库。因此读者在多线程程序中看到的是pthread_create而非clone。

*我对Linux操作系统实现知之甚少，很多浅见都是网上浏览所得，期待以后能够系统性的学习*

脱离开具体的场景来讨论问题的经常是没有意义的，创造一个概念在生搬硬套现实的场景同样容易产生谬误。关于进程和线程，还有一个一句话定义：线程是最小的执行单位，进程是最小的分配资源单位。这句话固然精辟，高度概括，但我认为它还是一个总结，是加深理解的一个维度，而不是不可反驳真理。


### Python线程

先划一下重点，Python真的很简单，线程相关的知识点不外乎下面几个，如果都能想起来，下文就不用继续看了。

1. Python创建线程有两种方式，一种是创建Thread对象，传入要运行的方法；另一种是继承Thread类，重写其中的run方法。
2. 通过将子线程设置为守护线程，setDeaon(True)，可以让主线程结束时子线程也同时结束。
3. 通过join方法让父线程等待子线程执行结束，注意如果同时创建了多个子线程，这些子线程需要并行的执行，需要在这些子线程的start方法都已经被调用了之后，再调用每个子线程的join函数，否者这些子线程间的执行将变为串行。
4. 线程锁，Lock和Rlock，Lock为指令锁，不可重入，RLock为可重入锁
5. 线程间交互，Condition和Event
6. 队列queue

以下分别讲述给个知识点，以代码为主：

#### 线程创建：

1. 通过创建Thread对象创建子线程

```python 
import threading
import time

def run(n):
    print("task", n)
    time.sleep(2)

t1 = threading.Thread(target=run, args=('t1',)) # 传参为元组，逗号为必须
t1.start()
t2 = threading.Thread(target=run, args=('t2',))
t2.start()
print(threading.active_count()) # 结果为3，1个主线程，2个子线程
```
1. 通过继承Thread类并且重写run方法创建子线程

```python 
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, n, name):
        super(MyThread, self).__init__()
        self.n = n
        self.name = name

    def run(self) -> None:
        print("running task ", self.n)
        time.sleep(2)

print(threading.current_thread()) # 打印当前线程名称， MainThread
t1 = MyThread(1, '线程1')
t2 = MyThread(2, '线程2')
print(threading.active_count()) # 打印结果为1，1个主线程
t1.start()
t2.start()
print(threading.active_count()) # 打印结果为3，1个主线程，2个子线程
```

#### setDaemon方法和join方法

以下的一段程序来自网络（{% link 链接 https://www.cnblogs.com/alex3714/articles/5230609.html link %}）：

```python
import time
import threading

def run(n):
    print('--running--', n)
    time.sleep(2)   # 尝试修改此处的sleep时长，看程序执行结果有什么影响
    print('--done--')

def main():
    for i in range(5):
        t = threading.Thread(target=run, args=[i, ])
        print('starting thread', t.getName())
        t.start()
        t.join(timeout=3)  # 尝试修改此处的join时长，看程序执行结果有什么影响

m = threading.Thread(target=main, args=[])
m.setDaemon(True)  # 将此处的setDaemon(True)去掉，程序执行结果有什么影响
m.start()
m.join(timeout=5)
print("---main thread done----")
```

程序执行结果如下：
{% asset_img screenshot_01.png [执行结果] %}

可以看到，1.因为第14行join的存在，Thread-2、Thread-3与Thread-4这些子线程之间变成了串行执行；2.因为17行的m.setDaemon(True),当主线程退出后，变量m引用的线程立即退出，线程Thread-4也立即退出。Thread-4的立即退出，说明对于Thread-4而言，Deamon也是True。

修改第6行的sleep时长为3，修改第14行的timeout时长为2，执行程序结果如下：
{% asset_img screenshot_02.png [执行结果] %}

可以看到，因为join等待的时长小于函数执行的时长，所以没有等Thread-2执行结束，Thread-3就开始执行了。尝试在13行插入t.setDaemon(False)，又有不同的执行结果，可以看到当Thread-2、Thread-3与Thread-4这些子线程设置了Daemon为False，它们的父线程m需要等到它们都执行结束了才会退出，结果不在这里贴出来了，请自行尝试。

#### Lock和Rlock

Python线程锁，网上的示例很多，这里只给出一个示例：

```python
import threading

def run():
    global num
    # lock.acquire() # 注释掉锁之后，最末行print的结果小于100
    tmp = num + 1
    print(tmp)       # 这里需要做些什么，否则很难复现
    num = tmp
    # lock.release()

num = 0
lock = threading.Lock()

threads = []
for i in range(100):
    t1 = threading.Thread(target=run, args=())
    t1.start()
    threads.append(t1)

for t in threads:
    t.join()

print(num)
```

#### Condition和Event

线程间同步，对应场景类似两个人同时在干活，B需要在A完成特定的工序之后，才能继续后面的工作，B怎么知道A已经完成呢？也无外乎两种方式，第一种是A做完了通知B，期间B一直在休息区坐着；另一种是B隔一会就过来看一下，这种方式下A已完成工作这个信号需要时确切无疑的，当B过来看的时候，A不能表示我马上就好，或者还有5分钟就好，而应该明确告诉B我已经好了或者我还没好。对应的，Condition的方式类似A结束了通知B，期间B在休息室里；Event的方式就类似B一直在边上看着。

Condition的示例：

```python
import threading
import time

con = threading.Condition()
num = 0

class Producer(threading.Thread): # 生产者
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        global num
        con.acquire()         # 锁定线程
        while True:
            print("开始添加！！！")
            num += 1
            print("火锅里面鱼丸个数：%s" % str(num))
            time.sleep(1)
            if num >= 5:
                print("火锅里面里面鱼丸数量已经到达5个，无法添加了！")
                con.notify()  # 告诉休息区的小伙伴开吃
                con.wait()    # 自己进去休息区
        con.release()         # 释放锁

class Consumers(threading.Thread): # 消费者
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        con.acquire()
        global num
        while True:
            print("开始吃啦！！！")
            num -= 1
            print("火锅里面剩余鱼丸数量：%s" %str(num))
            time.sleep(2)
            if num <= 0:
                print("锅底没货了，赶紧加鱼丸吧！")
                con.notify()  # 告诉休息区的小伙伴开始生产
                con.wait()    # 自己进入休息区等待
        con.release()

p = Producer()
c = Consumers()
p.start()
c.start()
```

代码示例转载自网上，链接：{% link 上海-悠悠 https://www.cnblogs.com/alex3714/articles/5230609.html link %}。

以上是一个简单的生产者消费者示例，首先我们注意到13行和31行con.acquire()，可以动手试一下把它注释掉会怎么样，con.acquire()和con.release()的组合可以采用with语句进行替换，代码会更加简洁。然后，notify和wait的顺序不能换，不能干完活不通知对方一声自己就进入等待区了，这样会形成阻塞；但是如果将以上代码21行和22行的notify和wait的顺序交换，程序还是能正常执行，这是因为锁的原因，可以自己尝试一下，思考为什么会这样。

然后试想一下我们能否通过线程锁实现该模型，让两个线程竞争一把锁，谁拿到锁谁工作，工作完了释放锁？这样是可行的，但是需要加一些手段，为此我写了下面一段代码。关键就在于代码的第22行和第40行，当前获取了锁的进程工作完毕之后，需要sleep一下再竞争锁，否则很可能自己刚刚释放掉锁，又被自己拿到了，那整个流程就自己一个人在干活了，没别人什么事了。显然每次都sleep一下显得很low，所以设计出condition是有道理的。

```python
import threading
import time

lock = threading.Lock()
num = 0

class Producer(threading.Thread): # 生产者
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        global num
        lock.acquire()  # 锁定线程
        while True:
            print("开始添加！！！")
            num += 1
            print("火锅里面鱼丸个数：%s" % str(num))
            time.sleep(0.5)
            if num >= 5:
                print("火锅里面里面鱼丸数量已经到达5个，无法添加了！")
                lock.release()         # 释放锁
                time.sleep(2)
                lock.acquire()  # 锁定线程

class Consumers(threading.Thread): # 消费者
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        global num
        lock.acquire()  # 锁定线程
        while True:
            print("开始吃啦！！！")
            num -= 1
            print("火锅里面剩余鱼丸数量：%s" %str(num))
            time.sleep(0.5)
            if num <= 0:
                print("锅底没货了，赶紧加鱼丸吧！")
                lock.release()
                time.sleep(2)
                lock.acquire()  # 锁定线程

p = Producer()
c = Consumers()
p.start()
c.start()
```

至于Event，不得不再提一下python很简单，还是直接看示例吧，来自网络：{% link 金角大王Alex-Python之路,Day9, 进程、线程、协程篇 https://www.cnblogs.com/alex3714/articles/5230609.html link %}。

```python
import threading,time
import random
def light():
    if not event.isSet():
        event.set() #wait就不阻塞 #绿灯状态
    count = 0
    while True:
        if count < 10:
            print('\033[42;1m--green light on---\033[0m')
        elif count <13:
            print('\033[43;1m--yellow light on---\033[0m')
        elif count <20:
            if event.isSet():
                event.clear()
            print('\033[41;1m--red light on---\033[0m')
        else:
            count = 0
            event.set() #打开绿灯
        time.sleep(1)
        count +=1
def car(n):
    while 1:
        time.sleep(random.randrange(10))
        if  event.isSet(): #绿灯
            print("car [%s] is running.." % n)
        else:
            print("car [%s] is waiting for the red light.." %n)
if __name__ == '__main__':
    event = threading.Event()
    Light = threading.Thread(target=light)
    Light.start()
    for i in range(3):
        t = threading.Thread(target=car,args=(i,))
        t.start()
```

#### 队列queue

python的queue，与c++标准库中的queue类似，但是它的get和put两个函数，都有一个block的形参，而且默认为True，这使得它非常适用与线程间的交互，比如只需要简单改写一下上面的例子，它就可以代替event。网上相关的资料很多，不在这里赘述了。
