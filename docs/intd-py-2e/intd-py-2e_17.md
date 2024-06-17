# 第十五章\. 时间上的数据：进程与并发

> 计算机可以做的一件事情是被密封在纸板箱中并坐在仓库里，这是大多数人类做不到的。
> 
> Jack Handey

这一章和接下来的两章比之前的内容稍微有些挑战。在这一章中，我们涵盖了时间上的数据（在单台计算机上的顺序访问和并发访问），接着我们将在第十六章中讨论盒子中的数据（特殊文件和数据库的存储和检索），然后在第十七章中讨论空间中的数据（网络）。

# 程序与进程

当您运行一个单独的程序时，操作系统会创建一个单独的*进程*。它使用系统资源（CPU、内存、磁盘空间）和操作系统内核中的数据结构（文件和网络连接、使用统计等）。进程与其他进程隔离——它不能看到其他进程在做什么或者干扰它们。

操作系统会跟踪所有正在运行的进程，为每个进程分配一点运行时间，然后切换到另一个进程，以实现公平地分配工作和对用户响应迅速的双重目标。您可以通过图形界面（如 macOS 的活动监视器、Windows 计算机上的任务管理器或 Linux 中的`top`命令）查看进程的状态。

您还可以从自己的程序中访问进程数据。标准库的`os`模块提供了一种常见的访问某些系统信息的方式。例如，以下函数获取运行中 Python 解释器的*进程 ID*和*当前工作目录*：

```py
>>> import os
>>> os.getpid()
76051
>>> os.getcwd()
'/Users/williamlubanovic'
```

这些获取*用户 ID*和*组 ID*：

```py
>>> os.getuid()
501
>>> os.getgid()
20
```

## 使用子进程创建进程

到目前为止，你所见过的所有程序都是单独的进程。您可以使用标准库的`subprocess`模块从 Python 启动和停止其他已经存在的程序。如果只想在 shell 中运行另一个程序并获取其生成的所有输出（标准输出和标准错误输出），请使用`getoutput()`函数。在这里，我们获取 Unix 的`date`程序的输出：

```py
>>> import subprocess
>>> ret = subprocess.getoutput('date')
>>> ret
'Sun Mar 30 22:54:37 CDT 2014'
```

在进程结束之前，您将得不到任何返回。如果需要调用可能需要很多时间的内容，请参阅“并发”中关于*并发*的讨论。由于`getoutput()`的参数是表示完整 shell 命令的字符串，因此可以包括参数、管道、`<` 和 `>` 的 I/O 重定向等：

```py
>>> ret = subprocess.getoutput('date -u')
>>> ret
'Mon Mar 31 03:55:01 UTC 2014'
```

将该输出字符串管道传递给`wc`命令计数一行、六个“单词”和 29 个字符：

```py
>>> ret = subprocess.getoutput('date -u | wc')
>>> ret
'       1       6      29'
```

变体方法称为`check_output()`接受命令和参数列表。默认情况下，它只返回标准输出作为字节类型而不是字符串，并且不使用 shell：

```py
>>> ret = subprocess.check_output(['date', '-u'])
>>> ret
b'Mon Mar 31 04:01:50 UTC 2014\n'
```

要显示其他程序的退出状态，`getstatusoutput()`返回一个包含状态代码和输出的元组：

```py
>>> ret = subprocess.getstatusoutput('date')
>>> ret
(0, 'Sat Jan 18 21:36:23 CST 2014')
```

如果您不想捕获输出但可能想知道其退出状态，请使用`call()`：

```py
>>> ret = subprocess.call('date')
Sat Jan 18 21:33:11 CST 2014
>>> ret
0
```

（在类 Unix 系统中，`0` 通常是成功的退出状态。）

那个日期和时间被打印到输出中，但没有在我们的程序中捕获。因此，我们将返回代码保存为 `ret`。

你可以以两种方式运行带参数的程序。第一种是在一个字符串中指定它们。我们的示例命令是 `date -u`，它会打印当前的日期和时间（协调世界时）：

```py
>>> ret = subprocess.call('date -u', shell=True)
Tue Jan 21 04:40:04 UTC 2014
```

你需要 `shell=True` 来识别命令行 `date -u`，将其拆分为单独的字符串，并可能扩展任何通配符字符，比如 `*`（在这个示例中我们没有使用任何通配符）。

第二种方法是将参数列表化，因此不需要调用 shell：

```py
>>> ret = subprocess.call(['date', '-u'])
Tue Jan 21 04:41:59 UTC 2014
```

## 使用 multiprocessing 创建一个进程

你可以将一个 Python 函数作为一个独立的进程运行，甚至使用 `multiprocessing` 模块创建多个独立的进程。示例 15-1 中的示例代码简短而简单；将其保存为 *mp.py*，然后通过输入 `python mp.py` 运行它：

##### 示例 15-1\. mp.py

```py
import multiprocessing
import os

def whoami(what):
    print("Process %s says: %s" % (os.getpid(), what))

if __name__ == "__main__":
    whoami("I'm the main program")
    for n in range(4):
        p = multiprocessing.Process(target=whoami,
          args=("I'm function %s" % n,))
        p.start()
```

当我运行这个时，我的输出看起来像这样：

```py
Process 6224 says: I'm the main program
Process 6225 says: I'm function 0
Process 6226 says: I'm function 1
Process 6227 says: I'm function 2
Process 6228 says: I'm function 3
```

`Process()` 函数生成了一个新进程，并在其中运行 `do_this()` 函数。因为我们在一个有四次循环的循环中执行了这个操作，所以我们生成了四个执行 `do_this()` 然后退出的新进程。

`multiprocessing` 模块比一个喜剧团的小丑还要多。它真的是为那些需要将某些任务分配给多个进程以节省总体时间的时候而设计的；例如，下载网页进行爬取，调整图像大小等。它包括了排队任务、启用进程间通信以及等待所有进程完成的方法。“并发性” 探讨了其中的一些细节。

## 使用 terminate() 杀死一个进程

如果你创建了一个或多个进程，并且想出于某种原因终止其中一个（也许它陷入了循环，或者你感到无聊，或者你想成为一个邪恶的霸主），使用 `terminate()`。在 示例 15-2 中，我们的进程会计数到一百万，每一步都会睡眠一秒，并打印一个恼人的消息。然而，我们的主程序在五秒内失去耐心，然后将其从轨道上摧毁。

##### 示例 15-2\. mp2.py

```py
import multiprocessing
import time
import os

def whoami(name):
    print("I'm %s, in process %s" % (name, os.getpid()))

def loopy(name):
    whoami(name)
    start = 1
    stop = 1000000
    for num in range(start, stop):
        print("\tNumber %s of %s. Honk!" % (num, stop))
        time.sleep(1)

if __name__ == "__main__":
    whoami("main")
    p = multiprocessing.Process(target=loopy, args=("loopy",))
    p.start()
    time.sleep(5)
    p.terminate()
```

当我运行这个程序时，我得到了以下输出：

```py
I'm main, in process 97080
I'm loopy, in process 97081
    Number 1 of 1000000\. Honk!
    Number 2 of 1000000\. Honk!
    Number 3 of 1000000\. Honk!
    Number 4 of 1000000\. Honk!
    Number 5 of 1000000\. Honk!
```

## 使用 os 获取系统信息

标准的 `os` 包提供了关于你的系统的许多详细信息，并且如果以特权用户（root 或管理员）身份运行你的 Python 脚本，还可以控制其中的一些内容。除了在 第十四章 中介绍的文件和目录函数外，它还有像这样的信息函数（在 iMac 上运行）：

```py
>>> import os
>>> os.uname()
posix.uname_result(sysname='Darwin',
nodename='iMac.local',
release='18.5.0',
version='Darwin Kernel Version 18.5.0: Mon Mar 11 20:40:32 PDT 2019;
 root:xnu-4903.251.3~3/RELEASE_X86_64',
machine='x86_64')
>>> os.getloadavg()
(1.794921875, 1.93115234375, 2.2587890625)
>>> os.cpu_count()
4
```

一个有用的函数是 `system()`，它会执行一个命令字符串，就像你在终端上输入一样：

```py
>>> import os
>>> os.system('date -u')
Tue Apr 30 13:10:09 UTC 2019
0
```

这是一个大杂烩。查看 [文档](https://oreil.ly/3r6xN) 以获取有趣的小知识。

## 使用 psutil 获取进程信息

第三方包 [psutil](https://oreil.ly/pHpJD) 还为 Linux、Unix、macOS 和 Windows 系统提供了系统和进程信息。

你可以猜测如何安装它：

```py
$ pip install psutil
```

覆盖范围包括以下内容：

系统

CPU、内存、磁盘、网络、传感器

进程

ID、父 ID、CPU、内存、打开的文件、线程

我们已经在前面的`os`讨论中看到，我的计算机有四个 CPU。它们已经使用了多少时间（以秒为单位）？

```py
>>> import psutil
>>> psutil.cpu_times(True)
[scputimes(user=62306.49, nice=0.0, system=19872.71, idle=256097.64),
scputimes(user=19928.3, nice=0.0, system=6934.29, idle=311407.28),
scputimes(user=57311.41, nice=0.0, system=15472.99, idle=265485.56),
scputimes(user=14399.49, nice=0.0, system=4848.84, idle=319017.87)]
```

它们现在有多忙？

```py
>>> import psutil
>>> psutil.cpu_percent(True)
26.1
>>> psutil.cpu_percent(percpu=True)
[39.7, 16.2, 50.5, 6.0]
```

也许你永远不需要这种类型的数据，但知道在哪里查找是很好的。

# 命令自动化

你经常从 shell 中运行命令（要么手动输入命令，要么使用 shell 脚本），但 Python 有多个良好的第三方管理工具。

一个相关的主题，*任务队列*，在“队列”中讨论。

## Invoke

`fabric`工具的第一个版本允许您使用 Python 代码定义本地和远程（网络）任务。开发人员将此原始包拆分为`fabric2`（远程）和`invoke`（本地）。

通过运行以下命令安装`invoke`：

```py
$ pip install invoke
```

`invoke`的一个用途是将函数作为命令行参数提供。让我们创建一个*tasks.py*文件，其中包含示例 15-3 中显示的行。

##### 示例 15-3\. tasks.py

```py
from invoke import task

@task
def mytime(ctx):
    import time
    now = time.time()
    time_str = time.asctime(time.localtime(now))
    print("Local time is", timestr)
```

（那个`ctx`参数是每个任务函数的第一个参数，但它仅在`invoke`内部使用。你可以随意命名它，但必须有一个参数在那里。）

```py
$ invoke mytime
Local time is Thu May  2 13:16:23 2019
```

使用参数`-l`或`--list`来查看可用的任务：

```py
$ invoke -l
Available tasks:

  mytime
```

任务可以有参数，你可以从命令行同时调用多个任务（类似于 shell 脚本中的`&&`使用）。

其他用途包括：

+   使用`run()`函数运行本地 shell 命令

+   响应程序的字符串输出模式

这只是一个简短的一瞥。详细信息请参阅[文档](http://docs.pyinvoke.org)。

## 其他命令助手

这些 Python 包在某种程度上类似于`invoke`，但在需要时可能有一个或多个更适合：

+   [`click`](https://click.palletsprojects.com)

+   [`doit`](http://pydoit.org)

+   [`sh`](http://amoffat.github.io/sh)

+   [`delegator`](https://github.com/kennethreitz/delegator.py)

+   [`pypeln`](https://cgarciae.github.io/pypeln)

# 并发

官方 Python 网站总结了一般的并发概念以及标准库中的[并发](http://bit.ly/concur-lib)。这些页面包含许多链接到各种包和技术；在本章中，我们展示了最有用的一些链接。

在计算机中，如果你在等待什么东西，通常是有两个原因：

I/O 绑定

这是目前最常见的情况。计算机 CPU 速度非常快 - 比计算机内存快数百倍，比磁盘或网络快数千倍。

CPU 绑定

CPU 保持繁忙。这发生在像科学或图形计算这样的*数字计算*任务中。

另外两个与并发相关的术语是：

同步

事物紧随其后，就像一行幼鹅跟随它们的父母。

异步

任务是独立的，就像随机的鹅在池塘里溅水一样。

随着您从简单系统和任务逐渐过渡到现实生活中的问题，您在某个时候将需要处理并发性。以网站为例。您通常可以相当快地为 web 客户端提供静态和动态页面。一秒钟的时间被认为是交互式的，但如果显示或交互需要更长时间，人们会变得不耐烦。像 Google 和 Amazon 这样的公司进行的测试表明，如果页面加载速度稍慢，流量会迅速下降。

但是，如果有些事情花费很长时间，比如上传文件、调整图像大小或查询数据库，你又无能为力怎么办？你不能再在同步的 web 服务器代码中做了，因为有人在等待。

在单台计算机上，如果要尽可能快地执行多个任务，就需要使它们相互独立。慢任务不应该阻塞其他所有任务。

本章前面展示了如何利用多进程在单台计算机上重叠工作。如果您需要调整图像大小，您的 web 服务器代码可以调用一个单独的、专用的图像调整进程来异步和并发地运行。它可以通过调用多个调整大小的进程来扩展您的应用程序。

诀窍在于让它们彼此协同工作。任何共享的控制或状态意味着会有瓶颈。更大的诀窍是处理故障，因为并发计算比常规计算更难。许多事情可能会出错，你成功的几率会更低。

好的。什么方法可以帮助您应对这些复杂性？让我们从管理多个任务的好方法开始：*队列*。

## 队列

队列类似于列表：东西从一端添加，从另一端取走。最常见的是所谓的*FIFO*（先进先出）。

假设你正在洗盘子。如果你被困在整个工作中，你需要洗每个盘子，擦干它，并把它收起来。你可以用多种方式做到这一点。你可能先洗第一只盘子，然后擦干，然后把它收起来。然后你重复第二只盘子，依此类推。或者，您可以批量操作，洗所有的盘子，擦干它们，然后把它们收起来；这意味着您在水槽和沥干架上有足够的空间来存放每一步积累的所有盘子。这些都是同步方法——一个工人，一次做一件事。

作为替代方案，您可以找一个或两个帮手。如果您是洗碗工，您可以把每个洗净的盘子交给擦干工，擦干工再把每个擦干的盘子交给收拾工。只要每个人的工作速度一样，你们应该比一个人做快得多。

然而，如果你洗碗比烘干快怎么办？湿碟子要么掉在地上，要么堆在你和烘干机之间，或者你只是走音哼着歌等待烘干机准备好。如果最后一个人比烘干机慢，干燥好的碟子最终可能会掉在地上，或者堆在一起，或者烘干机开始哼歌。你有多个工人，但整体任务仍然是同步的，只能按照最慢的工人的速度进行。

*众人拾柴火焰高*，古语如是说（我一直以为这是阿米什人的，因为它让我想到了建造谷仓）。增加工人可以建造谷仓或者更快地洗碗。这涉及到*队列*。

通常，队列传输*消息*，可以是任何类型的信息。在这种情况下，我们对分布式任务管理的队列感兴趣，也称为*工作队列*、*作业队列*或*任务队列*。水池中的每个碟子都交给一个可用的洗碗机，洗碗机洗完后交给第一个可用的烘干机，烘干机烘干后交给一个放置者。这可以是同步的（工人等待处理一个碟子和另一个工人来接收它），也可以是异步的（碟子在不同速度的工人之间堆积）。只要你有足够的工人，并且他们跟得上碟子的速度，事情就会快得多。

## 进程

你可以用许多方法实现队列。对于单台机器，标准库的 `multiprocessing` 模块（前面你见过）包含一个 `Queue` 函数。让我们模拟只有一个洗碗机和多个烘干进程（稍后有人会把碟子放好），以及一个中间的 `dish_queue`。将这个程序称为 *dishes.py*（Example 15-4）。

##### Example 15-4\. dishes.py

```py
import multiprocessing as mp

def washer(dishes, output):
    for dish in dishes:
        print('Washing', dish, 'dish')
        output.put(dish)

def dryer(input):
    while True:
        dish = input.get()
        print('Drying', dish, 'dish')
        input.task_done()

dish_queue = mp.JoinableQueue()
dryer_proc = mp.Process(target=dryer, args=(dish_queue,))
dryer_proc.daemon = True
dryer_proc.start()

dishes = ['salad', 'bread', 'entree', 'dessert']
washer(dishes, dish_queue)
dish_queue.join()
```

运行你的新程序，像这样：

```py
$ python dishes.py
Washing salad dish
Washing bread dish
Washing entree dish
Washing dessert dish
Drying salad dish
Drying bread dish
Drying entree dish
Drying dessert dish
```

这个队列看起来很像一个简单的 Python 迭代器，产生一系列的碟子。实际上，它启动了独立的进程以及洗碗机和烘干机之间的通信。我使用了 `JoinableQueue` 和最终的 `join()` 方法来告诉洗碗机所有的碟子已经干燥好了。在 `multiprocessing` 模块中还有其他的队列类型，你可以阅读 [文档](http://bit.ly/multi-docs) 获取更多例子。

## 线程

*线程*在一个进程中运行，并可以访问进程中的所有内容，类似于多重人格。`multiprocessing` 模块有一个名为 `threading` 的表兄弟，它使用线程而不是进程（实际上，`multiprocessing` 是它基于进程的对应物）。让我们用线程重新做我们的进程示例，如 Example 15-5 所示。

##### Example 15-5\. thread1.py

```py
import threading

def do_this(what):
    whoami(what)

def whoami(what):
    print("Thread %s says: %s" % (threading.current_thread(), what))

if __name__ == "__main__":
    whoami("I'm the main program")
    for n in range(4):
        p = threading.Thread(target=do_this,
          args=("I'm function %s" % n,))
        p.start()
```

这是我的打印输出：

```py
Thread <_MainThread(MainThread, started 140735207346960)> says: I'm the main
program
Thread <Thread(Thread-1, started 4326629376)> says: I'm function 0
Thread <Thread(Thread-2, started 4342157312)> says: I'm function 1
Thread <Thread(Thread-3, started 4347412480)> says: I'm function 2
Thread <Thread(Thread-4, started 4342157312)> says: I'm function 3
```

我们可以通过线程重新复制我们基于进程的洗碟子示例，如 Example 15-6 所示。

##### Example 15-6\. thread_dishes.py

```py
import threading, queue
import time

def washer(dishes, dish_queue):
    for dish in dishes:
        print ("Washing", dish)
        time.sleep(5)
        dish_queue.put(dish)

def dryer(dish_queue):
    while True:
        dish = dish_queue.get()
        print ("Drying", dish)
        time.sleep(10)
        dish_queue.task_done()

dish_queue = queue.Queue()
for n in range(2):
    dryer_thread = threading.Thread(target=dryer, args=(dish_queue,))
    dryer_thread.start()

dishes = ['salad', 'bread', 'entree', 'dessert']
washer(dishes, dish_queue)
dish_queue.join()
```

`multiprocessing`和`threading`之间的一个区别是，`threading`没有`terminate()`函数。没有简单的方法来终止运行中的线程，因为它可能会在您的代码中引发各种问题，甚至可能影响时空连续体本身。

线程可能是危险的。就像 C 和 C++等语言中的手动内存管理一样，它们可能会导致极难发现，更不用说修复的错误。要使用线程，程序中的所有代码（以及它使用的外部库中的代码）都必须是*线程安全*的。在前面的示例代码中，线程没有共享任何全局变量，因此它们可以独立运行而不会出错。

假设你是一个在闹鬼的房子里进行超自然调查的调查员。鬼魂在走廊里游荡，但彼此并不知道对方的存在，随时都可以查看、添加、删除或移动房子里的任何物品。

你戒备地穿过房子，用你那令人印象深刻的仪器进行测量。突然间，你注意到你刚刚走过的烛台不见了。

房子里的内容就像程序中的变量一样。鬼魂是进程（房子）中的线程。如果鬼魂只是偶尔瞥一眼房子的内容，那就没有问题。就像一个线程读取常量或变量的值而不试图改变它一样。

然而，某些看不见的实体可能会拿走你的手电筒，往你的脖子上吹冷风，把弹珠放在楼梯上，或点燃壁炉。*真正*微妙的鬼魂会改变你可能永远不会注意到的其他房间里的东西。

尽管你有花哨的仪器，但你要弄清楚谁做了什么，怎么做的，什么时候做的，以及在哪里做的，是非常困难的。

如果您使用多个进程而不是线程，那就像每个房子只有一个（活着的）人一样。如果您把白兰地放在壁炉前，一个小时后它仍会在那里——有些会因蒸发而丢失，但位置不变。

当不涉及全局数据时，线程可能是有用且安全的。特别是，在等待某些 I/O 操作完成时，线程可节省时间。在这些情况下，它们不必争夺数据，因为每个线程都有完全独立的变量。

但线程有时确实有充分理由更改全局数据。事实上，启动多个线程的一个常见原因是让它们分配某些数据的工作，因此预期对数据进行一定程度的更改。

安全共享数据的通常方法是在修改线程中的变量之前应用软件*锁定*。这样在进行更改时可以阻止其他线程进入。这就像让一个捉鬼者守卫你想保持清静的房间一样。不过，诀窍在于你需要记得解锁它。而且，锁定可以嵌套：如果另一个捉鬼者也在监视同一个房间，或者是房子本身呢？锁的使用是传统的但难以做到完全正确。

###### 注：

在 Python 中，由于标准 Python 系统中的一个实现细节，线程不会加速 CPU 密集型任务，这称为*全局解释器锁*（GIL）。这存在是为了避免 Python 解释器中的线程问题，但实际上可能使多线程程序比其单线程版本或甚至多进程版本更慢。

因此，对于 Python，建议如下：

+   对于 I/O 密集型问题，请使用线程

+   对于 CPU 密集型问题，请使用进程、网络或事件（在下一节中讨论）

## concurrent.futures

正如您刚刚看到的，使用线程或多进程涉及许多细节。`concurrent.futures` 模块已添加到 Python 3.2 标准库中，以简化这些操作。它允许您调度异步工人池，使用线程（当 I/O 密集型时）或进程（当 CPU 密集型时）。您将得到一个 *future* 来跟踪它们的状态并收集结果。

示例 15-7 包含一个测试程序，您可以将其保存为 *cf.py*。任务函数 `calc()` 睡眠一秒钟（我们模拟忙于某事），计算其参数的平方根，并返回它。程序可以接受一个可选的命令行参数，表示要使用的工人数，默认为 3。它在线程池中启动此数量的工人，然后在进程池中启动，然后打印经过的时间。`values` 列表包含五个数字，逐个发送给 `calc()` 在工人线程或进程中。

##### 示例 15-7\. cf.py

```py
from concurrent import futures
import math
import time
import sys

def calc(val):
    time.sleep(1)
    result = math.sqrt(float(val))
    return result

def use_threads(num, values):
    t1 = time.time()
    with futures.ThreadPoolExecutor(num) as tex:
        results = tex.map(calc, values)
    t2 = time.time()
    return t2 - t1

def use_processes(num, values):
    t1 = time.time()
    with futures.ProcessPoolExecutor(num) as pex:
        results = pex.map(calc, values)
    t2 = time.time()
    return t2 - t1

def main(workers, values):
    print(f"Using {workers} workers for {len(values)} values")
    t_sec = use_threads(workers, values)
    print(f"Threads took {t_sec:.4f} seconds")
    p_sec = use_processes(workers, values)
    print(f"Processes took {p_sec:.4f} seconds")

if __name__ == '__main__':
    workers = int(sys.argv[1])
    values = list(range(1, 6)) # 1 .. 5
    main(workers, values)
```

这里是我得到的一些结果：

```py
$ python cf.py 1
Using 1 workers for 5 values
Threads took 5.0736 seconds
Processes took 5.5395 seconds
$ python cf.py 3
Using 3 workers for 5 values
Threads took 2.0040 seconds
Processes took 2.0351 seconds
$ python cf.py 5
Using 5 workers for 5 values
Threads took 1.0052 seconds
Processes took 1.0444 seconds
```

那一秒钟的 `sleep()` 强制每个工人对每个计算都花费一秒钟：

+   只有一个工人同时工作，一切都是串行的，总时间超过五秒。

+   五个工人与被测试值的大小匹配，所以经过的时间略多于一秒。

+   使用三个工人，我们需要两次运行来处理所有五个值，所以经过了两秒。

在程序中，我忽略了实际的 `results`（我们计算的平方根），以突出显示经过的时间。此外，使用 `map()` 来定义池会导致我们在返回 `results` 之前等待所有工人完成。如果您希望在每次完成时获取每个结果，让我们尝试另一个测试（称为 *cf2.py*），在该测试中，每个工人在计算完值及其平方根后立即返回该值（示例 15-8）。

##### 示例 15-8\. cf2.py

```py
from concurrent import futures
import math
import sys

def calc(val):
    result = math.sqrt(float(val))
    return val, result

def use_threads(num, values):
    with futures.ThreadPoolExecutor(num) as tex:
        tasks = [tex.submit(calc, value) for value in values]
        for f in futures.as_completed(tasks):
             yield f.result()

def use_processes(num, values):
    with futures.ProcessPoolExecutor(num) as pex:
        tasks = [pex.submit(calc, value) for value in values]
        for f in futures.as_completed(tasks):
             yield f.result()

def main(workers, values):
    print(f"Using {workers} workers for {len(values)} values")
    print("Using threads:")
    for val, result in use_threads(workers, values):
        print(f'{val} {result:.4f}')
    print("Using processes:")
    for val, result in use_processes(workers, values):
        print(f'{val} {result:.4f}')

if __name__ == '__main__':
    workers = 3
    if len(sys.argv) > 1:
        workers = int(sys.argv[1])
    values = list(range(1, 6)) # 1 .. 5
    main(workers, values)
```

我们的 `use_threads()` 和 `use_processes()` 函数现在是生成器函数，每次迭代调用 `yield` 返回。在我的机器上运行一次，您可以看到工人不总是按顺序完成 `1` 到 `5`：

```py
$ python cf2.py 5
Using 5 workers for 5 values
Using threads:
3 1.7321
1 1.0000
2 1.4142
4 2.0000
5 2.2361
Using processes:
1 1.0000
2 1.4142
3 1.7321
4 2.0000
5 2.2361
```

您可以在任何时候使用 `concurrent.futures` 启动一堆并发任务，例如以下内容：

+   爬取网页上的 URL

+   处理文件，如调整图像大小

+   调用服务 API

如往常一样，[文档](https://oreil.ly/dDdF-) 提供了额外的详细信息，但更加技术性。

## 绿色线程和 gevent

正如你所见，开发者传统上通过将程序中的慢点运行在单独的线程或进程中来避免慢点。Apache 网络服务器就是这种设计的一个例子。

一个替代方案是*基于事件*的编程。一个基于事件的程序运行一个中央*事件循环*，分发任何任务，并重复该循环。NGINX 网络服务器遵循这种设计，并且通常比 Apache 更快。

`gevent`库是基于事件的，并完成了一个巧妙的技巧：你编写普通的命令式代码，它会神奇地将部分代码转换为*协程*。这些协程类似于可以相互通信并跟踪其位置的生成器。`gevent`修改了 Python 许多标准对象如`socket`，以使用其机制而不是阻塞。这不能与 Python 中用 C 编写的插件代码一起工作，比如一些数据库驱动。

你可以使用`pip`安装`gevent`：

```py
$ pip install gevent
```

这里是[在`gevent`网站的示例代码的变体](http://www.gevent.org)。在即将到来的 DNS 部分中，你会看到`socket`模块的`gethostbyname()`函数。这个函数是同步的，所以你要等待（可能很多秒），而它在世界各地的名称服务器中查找地址。但你可以使用`gevent`版本来独立查找多个站点。将其保存为*gevent_test.py*（示例 15-9）。

##### 示例 15-9\. gevent_test.py

```py
import gevent
from gevent import socket
hosts = ['www.crappytaxidermy.com', 'www.walterpottertaxidermy.com',
    'www.antique-taxidermy.com']
jobs = [gevent.spawn(gevent.socket.gethostbyname, host) for host in hosts]
gevent.joinall(jobs, timeout=5)
for job in jobs:
    print(job.value)
```

在前面的示例中有一个单行的 for 循环。每个主机名依次提交给`gethostbyname()`调用，但它们可以异步运行，因为这是`gevent`版本的`gethostbyname()`。

运行*gevent_test.py*：

```py
$ python gevent_test.py 
66.6.44.4
74.125.142.121
78.136.12.50
```

`gevent.spawn()`创建一个*greenlet*（有时也称为*绿色线程*或*微线程*）来执行每个`gevent.socket.gethostbyname(url)`。

与普通线程的区别在于它不会阻塞。如果发生了本应该阻塞普通线程的事件，`gevent`会切换控制到其他的 greenlet。

`gevent.joinall()`方法等待所有生成的作业完成。最后，我们会输出这些主机名对应的 IP 地址。

你可以使用它的富有表现力的名为*monkey-patching*的函数，而不是`gevent`版本的`socket`。这些函数修改标准模块如`socket`，以使用绿色线程而不是调用模块的`gevent`版本。当你希望`gevent`被应用到所有代码，甚至是无法访问的代码时，这是非常有用的。

在你的程序顶部添加以下调用：

```py
from gevent import monkey
monkey.patch_socket()
```

这里将`gevent`套接字插入到任何地方普通的`socket`被调用的地方，即使是在你的程序中的标准库中。再次强调，这仅适用于 Python 代码，而不适用于用 C 编写的库。

另一个函数 monkey-patches 更多的标准库模块：

```py
from gevent import monkey
monkey.patch_all()
```

在你的程序顶部使用这个来获取尽可能多的`gevent`加速。

将此程序保存为*gevent_monkey.py*（示例 15-9）。

##### 示例 15-10\. gevent_monkey.py

```py
import gevent
from gevent import monkey; monkey.patch_all()
import socket
hosts = ['www.crappytaxidermy.com', 'www.walterpottertaxidermy.com',
    'www.antique-taxidermy.com']
jobs = [gevent.spawn(socket.gethostbyname, host) for host in hosts]
gevent.joinall(jobs, timeout=5)
for job in jobs:
    print(job.value)
```

再次运行程序：

```py
$ python gevent_monkey.py
66.6.44.4
74.125.192.121
78.136.12.50
```

使用`gevent`存在潜在风险。与任何基于事件的系统一样，每段执行的代码应该相对迅速。虽然它是非阻塞的，但执行大量工作的代码仍然慢。

monkey-patching 的概念使一些人感到不安。然而，像 Pinterest 这样的大型网站使用`gevent`显著加速他们的网站。就像药瓶上的小字一样，请按照指示使用`gevent`。

欲知更多示例，请参阅这个详尽的`gevent`[教程](https://oreil.ly/BWR_q)。

###### 注意

你可能也考虑使用[`tornado`](http://www.tornadoweb.org)或者[`gunicorn`](http://gunicorn.org)，这两个流行的事件驱动框架提供了低级事件处理和快速的 Web 服务器。如果你想构建一个快速的网站而不想麻烦传统的 Web 服务器如 Apache，它们值得一试。

## twisted

[`twisted`](http://twistedmatrix.com/trac)是一个异步的、事件驱动的网络框架。你可以将函数连接到诸如数据接收或连接关闭等事件，当这些事件发生时，这些函数就会被调用。这是一种*回调*设计，如果你之前写过 JavaScript 代码，这种方式可能很熟悉。如果你还不熟悉，它可能看起来有些反直觉。对于一些开发者来说，基于回调的代码在应用程序增长时变得更难管理。

通过以下命令安装它：

```py
$ pip install twisted
```

`twisted`是一个庞大的包，支持多种基于 TCP 和 UDP 的互联网协议。简单说，我们展示了一个从[twisted 示例](http://bit.ly/twisted-ex)改编的小小的“敲门”服务器和客户端。首先，让我们看看服务器，*knock_server.py*：(示例 15-11)。

##### 示例 15-11\. knock_server.py

```py
from twisted.internet import protocol, reactor

class Knock(protocol.Protocol):
    def dataReceived(self, data):
        print('Client:', data)
        if data.startswith("Knock knock"):
            response = "Who's there?"
        else:
            response = data + " who?"
        print('Server:', response)
        self.transport.write(response)

class KnockFactory(protocol.Factory):
    def buildProtocol(self, addr):
        return Knock()

reactor.listenTCP(8000, KnockFactory())
reactor.run()
```

现在让我们快速浏览它的可靠伴侣，*knock_client.py*（示例 15-12）。

##### 示例 15-12\. knock_client.py

```py
from twisted.internet import reactor, protocol

class KnockClient(protocol.Protocol):
    def connectionMade(self):
        self.transport.write("Knock knock")

    def dataReceived(self, data):
        if data.startswith("Who's there?"):
            response = "Disappearing client"
            self.transport.write(response)
        else:
            self.transport.loseConnection()
            reactor.stop()

class KnockFactory(protocol.ClientFactory):
    protocol = KnockClient

def main():
    f = KnockFactory()
    reactor.connectTCP("localhost", 8000, f)
    reactor.run()

if __name__ == '__main__':
    main()
```

首先启动服务器：

```py
$ python knock_server.py
```

然后，启动客户端：

```py
$ python knock_client.py
```

服务器和客户端交换消息，服务器打印对话：

```py
Client: Knock knock
Server: Who's there?
Client: Disappearing client
Server: Disappearing client who?
```

我们的恶作剧客户端然后结束，让服务器等待笑话的结尾。

如果你想进入`twisted`的世界，请尝试一些它文档中的其他示例。

## asyncio

Python 在 3.4 版本中加入了`asyncio`库。它是使用新的`async`和`await`功能定义并发代码的一种方式。这是一个涉及许多细节的大课题。为了避免在本章节中过多涉及，我已将有关`asyncio`和相关主题的讨论移至附录 C。

## Redis

我们之前关于洗碗的代码示例，使用进程或线程，在单台机器上运行。让我们再来看一种可以在单台机器或跨网络运行的队列方法。即使有多个歌唱进程和跳舞线程，有时一台机器还不够，你可以把这一节当作单台（一台机器）和多台并发之间的桥梁。

要尝试本节中的示例，您需要一个 Redis 服务器及其 Python 模块。您可以在“Redis”中找到获取它们的位置。在那一章中，Redis 的角色是数据库。在这里，我们展示它的并发性格。

使用 Redis 列表是快速创建队列的方法。Redis 服务器运行在一台机器上；这可以是与其客户端相同的机器，或者是客户端通过网络访问的另一台机器。无论哪种情况，客户端通过 TCP 与服务器通信，因此它们是网络化的。一个或多个提供者客户端将消息推送到列表的一端。一个或多个客户端工作进程使用*阻塞弹出*操作监视此列表。如果列表为空，它们就会坐在那里打牌。一旦有消息到达，第一个渴望的工作进程就会获取到它。

像我们早期基于进程和线程的示例一样，*redis_washer.py*生成一系列的菜品（示例 15-13）。

##### 示例 15-13\. redis_washer.py

```py
import redis
conn = redis.Redis()
print('Washer is starting')
dishes = ['salad', 'bread', 'entree', 'dessert']
for dish in dishes:
    msg = dish.encode('utf-8')
    conn.rpush('dishes', msg)
    print('Washed', dish)
conn.rpush('dishes', 'quit')
print('Washer is done')
```

循环生成四条包含菜品名称的消息，然后是一条说“quit”的最终消息。它将每条消息追加到 Redis 服务器中的名为`dishes`的列表中，类似于追加到 Python 列表中。

一旦第一道菜准备好，*redis_dryer.py*就开始工作（示例 15-14）。

##### 示例 15-14\. redis_dryer.py

```py
import redis
conn = redis.Redis()
print('Dryer is starting')
while True:
    msg = conn.blpop('dishes')
    if not msg:
        break
    val = msg[1].decode('utf-8')
    if val == 'quit':
        break
    print('Dried', val)
print('Dishes are dried')
```

该代码等待第一个令牌为“dishes”的消息，并打印每一个干燥。它通过结束循环遵循*quit*消息。

先启动烘干机，然后启动洗碗机。在命令末尾使用`&`将第一个程序置于*后台*；它会继续运行，但不再接受键盘输入。这适用于 Linux、macOS 和 Windows，尽管您可能在下一行看到不同的输出。在这种情况下（macOS），它是关于后台烘干机进程的一些信息。然后，我们正常启动洗碗机进程（*前台*）。您将看到两个进程的混合输出：

```py
$ python redis_dryer.py & 
[2] 81691
Dryer is starting
$ python redis_washer.py
Washer is starting
Washed salad
Dried salad
Washed bread
Dried bread
Washed entree
Dried entree
Washed dessert
Washer is done
Dried dessert
Dishes are dried
[2]+  Done                    python redis_dryer.py
```

一旦从洗碗机进程到达 Redis 的菜品 ID 开始，我们勤劳的烘干机进程就开始将它们取回。每个菜品 ID 都是一个数字，除了最后的*sentinel*值，即字符串`'quit'`。当烘干机进程读取到`quit`菜品 ID 时，它就会退出，并且一些更多的后台进程信息会打印到终端（同样依赖系统）。您可以使用一个标志（一个否则无效的值）来指示数据流本身的某些特殊情况，例如，我们已经完成了。否则，我们需要添加更多的程序逻辑，比如以下内容：

+   预先同意一些最大的菜品编号，这实际上会成为一个标志。

+   进行一些特殊的*out-of-band*（不在数据流中的）进程间通信。

+   在一段时间内没有新数据时，设置超时。

让我们做一些最后的更改：

+   创建多个`dryer`进程。

+   给每个烘干机添加超时，而不是寻找一个标志。

新的*redis_dryer2.py*显示在示例 15-15 中。

##### 示例 15-15\. redis_dryer2.py

```py
def dryer():
    import redis
    import os
    import time
    conn = redis.Redis()
    pid = os.getpid()
    timeout = 20
    print('Dryer process %s is starting' % pid)
    while True:
        msg = conn.blpop('dishes', timeout)
        if not msg:
            break
        val = msg[1].decode('utf-8')
        if val == 'quit':
            break
        print('%s: dried %s' % (pid, val))
        time.sleep(0.1)
    print('Dryer process %s is done' % pid)

import multiprocessing
DRYERS=3
for num in range(DRYERS):
    p = multiprocessing.Process(target=dryer)
    p.start()
```

在后台启动烘干进程，然后在前台启动洗碗机进程：

```py
$ python redis_dryer2.py &
Dryer process 44447 is starting
Dryer process 44448 is starting
Dryer process 44446 is starting
$ python redis_washer.py
Washer is starting
Washed salad
44447: dried salad
Washed bread
44448: dried bread
Washed entree
44446: dried entree
Washed dessert
Washer is done
44447: dried dessert

```

一个更干燥的过程读取`quit` ID 并退出：

```py
Dryer process 44448 is done
```

20 秒后，其他烘干程序从它们的`blpop`调用中获取到`None`的返回值，表示它们已超时。它们说出它们的最后一句话并退出：

```py
Dryer process 44447 is done
Dryer process 44446 is done
```

在最后一个烘干子进程退出后，主要的烘干程序就结束了：

```py
[1]+  Done                    python redis_dryer2.py
```

## 超越队列

随着更多的零部件移动，我们可爱的装配线被打断的可能性也更多。如果我们需要洗一顿宴会的盘子，我们有足够的工人吗？如果烘干机喝醉了怎么办？如果水槽堵塞了怎么办？担心，担心！

如何应对这一切？常见的技术包括以下几种：

火而忘

只需传递事物，不要担心后果，即使没有人在那里。这就是盘子掉在地上的方法。

请求-响应

洗碗机收到烘干机的确认，烘干机收到收拾碗盘者的确认，每个管道中的盘子都会这样。

回压或节流

如果下游的某个人跟不上，这种技术会让快速工人放松点。

在实际系统中，您需要确保工人们能够跟上需求；否则，您会听到盘子掉在地上的声音。您可以将新任务添加到*待处理*列表中，而某些工作进程则会弹出最新消息并将其添加到*正在处理*列表中。当消息完成时，它将从正在处理列表中删除，并添加到*已完成*列表中。这让您知道哪些任务失败或花费太长时间。您可以自己使用 Redis 来完成这一过程，或者使用已经编写和测试过的系统。一些基于 Python 的队列包可以增加这种额外的管理水平，包括：

+   [`celery`](http://www.celeryproject.org)可以使用我们讨论过的方法（`multiprocessing`、`gevent`等）同步或异步地执行分布式任务。

+   [rq](http://python-rq.org)是一个基于 Redis 的 Python 作业队列库。

[队列](http://queues.io)讨论了队列软件，基于 Python 和其他语言。

# 即将到来

在本章中，我们将数据流经过程。在下一章中，您将看到如何在各种文件格式和数据库中存储和检索数据。

# 要做的事情

15.1 使用`multiprocessing`创建三个单独的进程。使每个进程在零到一秒之间等待一个随机数，打印当前时间，然后退出。
