---
title: Python 模块 subprocess
date: 2019-07-25 20:21:48
mathjax: true
tags:
 - Code
categories:
 - Python
 - module
---

谈一下自己学习subprocess模块和自己实践的过程中，遇到的一些问题与思考体会。先给出：{% link 官网链接 https://docs.python.org/3/library/subprocess.html#subprocess.Popen.communicate link %}

首先从名字说起，模块名称中包含process，就说明了其与线程的联系，官网中对于这个模型的介绍可谓相当的精辟，建议在学习这个模块的用法前多阅读几遍加深理解：

>The subprocess module allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes. This module intends to replace several older modules and functions.

然后，关注到这个模块最主要的两个函数是 subprocess.run() 和 subprocess.Popen()，它们有什么差别？我从官网粗略看了一下，没太理解为什么要同时设计 run 和 Popen；但是从官网一些实现的描述，我注意到 run 内部封装了一个 Popen。其实再仔细看一下，就可以发现最直接的一点差别，run 是一个函数，Popen 是一个类。再进一步，run 函数返回的对象类型是 subprocess.CompletedProcess，直白一点，run 函数直接获取到了子进程执行的结果；相比之下，Popen 类允许通过 stdin， 与子进程进行交互。

#### Part 1.

关于 subprocess.run() 函数，我想用一个简单示例结束战斗，python 用起来本来就很简单，更为丰富的使用方法可以参考官网。

```python
import subprocess
ret = subprocess.run(["ls", "-lrt"], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
print(ret.returncode)
print(ret.stdout.decode())
print(ret.stderr.decode())
```
#### Part 2.

对于 subprocess.Popen 类，它面向的整个与子进程的交互过程，问题来了，交互过程何时开始，何时结束？测试如下的代码的执行：

```python
import subprocess
import time
proc = subprocess.Popen(["pwd"], stdout=subprocess.PIPE)
time.sleep(10)
```

代码第4行 time.sleep(10) 让程序在 Popen 执行之后 sleep 10秒，在程序启动的10秒内和程序执行结束后，两次在Ubuntu终端下执行 ”ps -ef | grep pwd” 命令，可以看到如下执行结果：

{% asset_img screenshot-01.png [pwd命令的执行结果] %}

从上文的执行结果分析，程序第3行 subprocess.Popen 启动了pwd子进程，pwd子进程很快执行结束，而因为主进程没有执行 wait 调用，子进程变成了 defunct 僵尸进程，直到整个测试程序执行结束。为了避免子进程变成 defunct 僵尸进程，在 Popen 执行之后，需要调用 proc.wait(或者proc.communicate)。

我认为 proc.communicate 比 proc.wait 更值得推荐，原因我关注到了这么两点：首先在子进程输出到PIPE，并且输出太多超过了系统 buffer 空间大小时，wait 函数将会阻塞，而 communicate 函数没有类似的问题（具体的原因可能需要借助源码的学习了）；其次 proc.communicate 直接返回子进程的 stdout 和 stderror，在这个过程中 proc.communicate 主动对 stdout 和 stderror 的 PIPE 进行了 close，而 proc.wait 函数需要你主动进行 close 。

#### Part 3.

上面一节我们还是没有提到与子进程进行交互，在使用 Popen 时，我们可以通过子进程的标准输入输出，与子进程进行交互，示例如下：

```python test.py
while True:
    data = input().upper()
    print(data)
```

```python main.py
import subprocess
proc = subprocess.Popen(["python3", "test.py"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
try:
    while True:
        data = input('please input >')
        proc.stdin.write(data.encode())   # 写入字节
        proc.stdin.write(b'\n')           # 写入换行符，子进程每次读取一行
        proc.stdin.flush()                # 需要flush，子进程才能立即接收
        line = proc.stdout.readline()
        print(line)
except KeyboardInterrupt:
    print("user closed...")
finally:
    proc.kill()
    proc.wait()
    proc.stdin.close()
    proc.stdout.close()
    proc.stderr.close()
```

以上的 main.py，创建子进程调用 test.py，实现了将输入的字符转换为大写之后再输出的功能，留意代码中的注释部分，与子进程进行交互时，需要写入换行符，并且刷新PIPE。

以上的进程间交互有一个问题，如果子进程的输出不止一行怎么办，注意千万不要将 proc.stdout.readline() 放到一个循环里面，当子进程已经完成本轮输出之后，主进程还在读，便会形成阻塞。这个问题怎么解决呢？老实说我也不知道，查找了一些材料也没有找到破解的方法，或许主进程知道子进程每次交互的输出大小，或者子进程能正常退出是唯一的解决方法。因为这个问题不知道怎么解，所以应用的场景也就受到了限制，应用在子进程的执行结果较为确定的场景应该还是不错的，比如对子进程进行工具测试。
