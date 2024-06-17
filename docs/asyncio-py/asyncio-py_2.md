# 第二章。关于线程的真相

> 让我们坦率一点——你真的不想使用 Curio。所有事情都一样，你可能应该使用线程进行编程。是的，线程。那些线程。真的，我不是在开玩笑。
> 
> 戴夫·比兹利，《使用 Curio 进行开发》（https://oreil.ly/oXJaC）

如果你以前从未听说过线程，这里有一个基本的描述：线程是操作系统（OS）提供给软件开发者的一种特性，使他们能够指示操作系统哪些部分的程序可以并行运行。操作系统决定如何与各部分共享 CPU 资源，就像操作系统决定如何与同时运行的所有其他不同程序（进程）共享 CPU 资源一样。

既然你在读一本 Asyncio 的书，那么这一定是我告诉你，“线程很糟糕，你永远不应该使用它们”的部分，对吧？不幸的是，情况并不那么简单。我们需要权衡使用线程的利与弊，就像对待任何技术选择一样。

这本书根本不应该涉及到线程。但是这里有两个问题：Asyncio 被提供作为线程的一种替代，因此没有进行一些比较很难理解其价值；即使使用 Asyncio，你仍然可能需要处理线程和进程，所以你需要了解一些关于线程的知识。

###### 警告

这次讨论的背景仅限于网络编程应用程序中的并发性。抢占式多线程也用于其他领域，在那里权衡是完全不同的。

# 线程的好处

这些是线程的主要优点：

代码的易读性

你的代码可以并发运行，但仍然可以以非常简单、自顶向下的线性指令序列进行设置，以至于在函数体内，你可以假装没有发生并发——这一点非常关键。

具有共享内存的并行性

你的代码可以利用多个 CPU，同时线程可以共享内存。在许多工作负载中，这一点非常重要，因为在不同进程的独立内存空间之间移动大量数据可能会成本过高。

技术和现有代码

有大量的知识和最佳实践适用于编写多线程应用程序。还有大量依赖多线程并发操作的现有“阻塞”代码。

现在，关于*Python*，关于并行性的观点是值得质疑的，因为 Python 解释器使用全局锁，称为*全局解释器锁*（GIL），来保护解释器本身的内部状态。也就是说，它提供了对多个线程之间潜在灾难性竞态条件的保护。锁的一个副作用是它最终将所有线程固定在程序的单个 CPU 上。你可以想象，这会消除任何并行性能优势（除非你使用像 Cython 或 Numba 这样的工具来规避这个限制）。

然而，关于感知上的简单性的第一个观点是很重要的：在 Python 中，线程 *感觉* 特别简单，如果你以前没有因为不可能的竞争条件而受到过伤害，线程提供了一个非常吸引人的并发模型。即使你以前曾经受过伤害，线程仍然是一个令人信服的选择，因为你很可能已经学会了（通过艰难的方式）如何保持代码既简单又安全。

我这里没有空间深入讨论更安全的线程编程，但一般来说，最佳实践是使用 `concurrent.futures` 模块中的 `ThreadPoolExecutor` 类，通过 `submit()` 方法传递所有必需的数据。Example 2-1 展示了一个基本的例子。

##### Example 2-1\. 线程的最佳实践

```py
from concurrent.futures import ThreadPoolExecutor as Executor

def worker(data):
    <process the data>
with Executor(max_workers=10) as exe:
    future = exe.submit(worker, data)
```

`ThreadPoolExecutor` 提供了一个非常简单的接口来在线程中运行函数 —— 最棒的是，如果需要，你可以简单地通过使用 `ProcessPoolExecutor` 将线程池转换为子进程池。它与 `ThreadPoolExecutor` 拥有相同的 API，这意味着你的代码在改变时受影响很小。执行器 API 也被用于 `asyncio` 中，并在下一章节中进行了描述（参见 Example 3-3）。

通常情况下，你会更喜欢你的任务生命周期较短，这样当程序需要关闭时，你可以简单地调用 `Executor.shutdown(wait=True)` 并等待一两秒钟，以允许执行器完成。

最重要的是：如果可能的话，你应该尽量防止你的线程代码（在上述示例中的 `worker()` 函数）访问或写入任何全局变量！

###### 提示

Raymond Hettinger 在 [PyCon Russia 2016](https://oreil.ly/ZZVps) 和 [PyBay 2017](https://oreil.ly/JDplJ) 上提出了一些更安全的线程代码指南。我强烈建议你把这些视频加入你的观看列表中。

# 线程的缺点

> 非平凡的多线程程序对人类来说是难以理解的。通过设计模式、更好的原子性粒度（例如事务）、改进的语言和形式化方法，可以改善编程模型。然而，这些技术仅仅是在减少线程模型中不必要的巨大非确定性。这个模型依然是根本无法解决的。
> 
> Edward A. Lee 的 [“The Problem with Threads”](http://bit.ly/2CFOv8a)

线程的缺点已经在其他几个地方提到过，但为了完整起见，让我们把它们收集在这里：

线程是困难的。

线程程序中的线程错误和竞争条件是修复最困难的错误。通过经验，设计新软件以减少这些问题是可能的，但在非平凡、天真设计的软件中，它们几乎是无法修复的，即使是专家也是如此。真的！

线程是资源密集型的。

线程需要额外的操作系统资源来创建，比如预先分配的每个线程的栈空间，这会消耗进程虚拟内存。这在 32 位操作系统中是一个大问题，因为每个进程的地址空间限制为 3 GB。¹ 如今，随着 64 位操作系统的广泛普及，虚拟内存不再像以前那样宝贵（虚拟内存的可寻址空间通常为 48 位，即 256 TiB）。在现代桌面操作系统上，包括每个线程的栈空间，操作系统甚至不会在需要之前分配物理内存。例如，在拥有 8 GB 内存的现代 64 位 Fedora 29 Linux 上，使用以下简短的代码创建 10,000 个空线程：

```py
# threadmem.py
import os
from time import sleep
from threading import Thread
threads = [
  Thread(target=lambda: sleep(60)) for i in range(10000)
]
[t.start() for t in threads]
print(f'PID = {os.getpid()}')
[t.join() for t in threads]
```

导致`top`中的以下信息：

```py
MiB Mem : 7858.199 total, 1063.844 free, 4900.477 used
MiB Swap: 7935.996 total, 4780.934 free, 3155.062 used

  PID USER      PR  NI    VIRT    RES    SHR COMMAND
15166 caleb     20   0 80.291g 131.1m   4.8m python3
```

预分配的虚拟内存令人震惊，约为 80 GB（由于每个线程 8 MB 的栈空间！），但驻留内存仅约为 130 MB。在 32 位 Linux 系统上，由于 3 GB 用户空间地址空间限制，我将无法创建这么多线程，*不论*实际内存消耗如何。为了解决在 32 位系统上的这个问题，有时必须减少预配置的栈大小，您仍然可以在 Python 中使用`threading.stack_size([size])`来做到这一点。显然，减少栈大小对运行时安全性的影响取决于函数调用可能嵌套的程度，包括递归。单线程协程没有这些问题，并且对于并发 I/O 来说是一个更优越的选择。

线程可能会影响吞吐量。

在非常高的并发水平（例如，>5,000 个线程），由于上下文切换的成本，也可能会对吞吐量产生影响，假设您能够配置操作系统以允许创建那么多线程！例如，最近的 macOS 版本变得如此乏味，以至于测试前述的 10,000 个空线程示例时，我放弃了尝试提升所有限制。

线程是不灵活的。

无论线程是否准备好工作，操作系统都会持续共享 CPU 时间给所有线程。例如，一个线程可能在等待套接字上的数据，但操作系统调度程序仍可能在任何实际工作需要之前的数千次切换该线程和其他线程之间。（在异步世界中，`select()`系统调用用于检查是否需要转换套接字等待的协程；如果不需要，该协程甚至不会被唤醒，完全避免了任何切换成本。）

这些信息都不是新的，线程作为编程模型的问题也不是特定于平台的。例如，[Microsoft Visual C++文档](http://bit.ly/2Fr3eXK)对线程的看法如下：

> Windows API 中的核心并发机制是线程。通常使用 CreateThread 函数创建线程。尽管线程相对容易创建和使用，但操作系统分配了大量时间和其他资源来管理它们。此外，尽管每个线程保证接收与同一优先级级别的任何其他线程相同的执行时间，但相关开销要求您创建足够大的任务。对于更小或更细粒度的任务，与并发相关的开销可能会超过并行运行任务的好处。

但——我听到你反对——这是*Windows*，对吧？Unix 系统肯定不会有这些问题？以下是来自 Mac 开发者文库的[线程编程指南](https://oreil.ly/W3mBM)中类似的建议：

> 在内存使用和性能方面，线程对您的程序（以及系统）有实际成本。每个线程需要在内核内存空间和程序内存空间中分配内存。管理线程并协调其调度所需的核心结构存储在内核中，使用固定内存。您线程的堆栈空间和每个线程数据存储在程序的内存空间。大多数这些结构在您首次创建线程时创建和初始化——这个过程可能会比较昂贵，因为需要与内核进行必要的交互。

他们在[并发编程指南](https://oreil.ly/fcGNL)中更进一步（重点是我的）：

> 在过去，将并发引入应用程序需要创建一个或多个额外的线程。不幸的是，编写线程化的代码是具有挑战性的。线程是必须手动管理的低级工具。考虑到应用程序的最佳线程数量可以根据当前系统负载和底层硬件动态变化，实现正确的线程解决方案变得*极为困难*，甚至不可能实现。此外，通常与线程一起使用的同步机制增加了软件设计的复杂性和风险，而没有任何改善性能的保证。

这些主题反复出现：

+   线程使代码难以理解。

+   对于大规模并发（数千个并发任务），线程是一种效率低下的模型。

接下来，让我们看一个涉及线程的案例研究，突出第一点也是最重要的一点。

# 案例研究：机器人和餐具

> 第二，更重要的是，我们没有（至今仍没有）相信标准的多线程模型，即带有共享内存的抢占式并发：我们仍然认为，在“a = a + 1”不确定的语言中，没有人能编写正确的程序。
> 
> Roberto Ierusalimschy 等人的[“Lua 的演变”](http://bit.ly/2Fq9M8P)

在本书的开头，我讲述了一个餐馆的故事，其中类人机器人——ThreadBots——完成了所有的工作。在那个比喻中，每个工人都是一个线程。在例 2-2 中的案例研究中，我们将探讨*为什么*线程被认为是不安全的。

##### 例 2-2\. 用于桌面服务的 ThreadBot 编程

```py
import threading
from queue import Queue

class ThreadBot(threading.Thread):  ![1](img/1.png)
  def __init__(self):
    super().__init__(target=self.manage_table)  ![2](img/2.png)
    self.cutlery = Cutlery(knives=0, forks=0)  ![3](img/3.png)
    self.tasks = Queue()  ![4](img/4.png)

  def manage_table(self):
    while True:  ![5](img/5.png)
      task = self.tasks.get()
      if task == 'prepare table':
        kitchen.give(to=self.cutlery, knives=4, forks=4) ![6](img/6.png)
      elif task == 'clear table':
        self.cutlery.give(to=kitchen, knives=4, forks=4)
      elif task == 'shutdown':
        return
```

![1](img/#co_the_truth_about_threads_CO1-1)

`ThreadBot`是线程的子类。

![2](img/#co_the_truth_about_threads_CO1-2)

线程的目标函数是后面在文件中定义的`manage_table()`方法。

![3](img/#co_the_truth_about_threads_CO1-3)

该机器人将负责等待服务桌子，并需要负责一些餐具。每个机器人在这里都会跟踪从厨房拿到的餐具。（`Cutlery`类将稍后定义。）

![4](img/#co_the_truth_about_threads_CO1-4)

机器人也将被分配任务。它们将被添加到任务队列中，并在其主处理循环中执行这些任务。

![5](img/#co_the_truth_about_threads_CO1-5)

该机器人的主要例程是这个无限循环。如果你需要关闭一个机器人，你必须给它`shutdown`任务。

![6](img/#co_the_truth_about_threads_CO1-6)

对于这个机器人，只定义了三个任务。其中`prepare table`是机器人必须执行的任务，用于准备新的服务桌。在我们的测试中，唯一的要求是从厨房获取一套餐具并将其放在桌子上。`clear table`用于清理桌子：机器人必须将使用过的餐具退回到厨房。`shutdown`只是关闭机器人。

例 2-3 展示了`Cutlery`对象的定义。

##### 例 2-3\. `Cutlery`对象的定义

```py
from attr import attrs, attrib

@attrs  ![1](img/1.png)
class Cutlery:
    knives = attrib(default=0)  ![2](img/2.png)
    forks = attrib(default=0)

    def give(self, to: 'Cutlery', knives=0, forks=0):  ![3](img/3.png)
        self.change(-knives, -forks)
        to.change(knives, forks)

    def change(self, knives, forks):  ![4](img/4.png)
            self.knives += knives
            self.forks += forks

kitchen = Cutlery(knives=100, forks=100)  ![5](img/5.png)
bots = [ThreadBot() for i in range(10)]  ![6](img/6.png)

import sys
for bot in bots:
    for i in range(int(sys.argv[1])):  ![7](img/7.png)
        bot.tasks.put('prepare table')
        bot.tasks.put('clear table')
    bot.tasks.put('shutdown')  ![8](img/8.png)

print('Kitchen inventory before service:', kitchen)
for bot in bots:
    bot.start()

for bot in bots:
    bot.join()
print('Kitchen inventory after service:', kitchen)
```

![1](img/#co_the_truth_about_threads_CO2-1)

`attrs`是一个开源的 Python 库，与线程或`asyncio`无关，是一个非常棒的库，可以轻松创建类。在这里，`@attrs`装饰器将确保这个`Cutlery`类将自动设置所有通常的样板代码（比如`__init__()`）。

![2](img/#co_the_truth_about_threads_CO2-2)

`attrib()`函数提供了一个创建属性的简便方法，包括默认值，你可能会在`__init__()`方法中作为关键字参数处理。

![3](img/#co_the_truth_about_threads_CO2-3)

这种方法用于将刀叉从一个`Cutlery`对象转移到另一个对象。通常，它将被机器人用于从厨房获取餐具供新桌子使用，并在清空桌子后将餐具退回到厨房。

![4](img/#co_the_truth_about_threads_CO2-4)

这是一个非常简单的实用程序函数，用于修改对象实例中的库存数据。

![5](img/#co_the_truth_about_threads_CO2-5)

我们定义`kitchen`作为餐具库存的标识符。通常，每个机器人将从这个位置获取餐具。还要求在清理桌子时，他们必须将餐具归还到这个库存位置。

![6](img/#co_the_truth_about_threads_CO2-6)

当我们进行测试时，执行此脚本。我们将使用 10 个 ThreadBots 进行测试。

![7](img/#co_the_truth_about_threads_CO2-7)

我们从命令行参数中获取桌子的数量，然后为每个机器人分配这些桌子的任务，负责餐厅里的摆放和清理工作。

![8](img/#co_the_truth_about_threads_CO2-8)

`shutdown`任务将使机器人停止运行（这样`bot.join()`稍后就会返回）。脚本的其余部分会打印诊断信息并启动机器人。

你测试代码的策略基本上是让一组 ThreadBots 在一系列的桌子服务中运行。每个 ThreadBot 必须执行以下操作：

+   *准备*一个“四人桌”，意味着从厨房获取四套刀子和叉子。

+   *清理*一个桌子，意味着将四套刀子和叉子从桌子上归还到厨房。

如果你让一群 ThreadBots 在一些桌子上运行特定次数，那么当所有工作都完成后，所有的刀子和叉子应该都回到厨房，并且都会被记录。

明智地决定，你要测试一下，每个 ThreadBot 同时处理一百张桌子，以确保它们可以协同工作，不会出错。这是测试的输出结果：

```py
$ python cutlery_test.py 100
Kitchen inventory before service: Cutlery(knives=100, forks=100)
Kitchen inventory after service: Cutlery(knives=100, forks=100)

```

所有的刀子和叉子最终都回到了厨房！因此，你为自己写的出色代码而感到自豪，并部署了这些机器人。不幸的是，在*实践中*，每当餐厅关门时，你都会发现并*没有*所有餐具都被收回。当你增加更多机器人或餐厅更加繁忙时，问题变得更加严重。沮丧之下，你又进行了测试，除了测试的规模（10000 张桌子！）之外，什么都没改变：

```py
$ python cutlery_test.py 10000
Kitchen inventory before service: Cutlery(knives=100, forks=100)
Kitchen inventory after service: Cutlery(knives=96, forks=108)

```

糟糕。现在你发现确实存在问题。在服务了 10000 张桌子后，你发现厨房里剩下的刀子和叉子数量不对。为了重现问题，你检查错误是否一致：

```py
$ python cutlery_test.py 10000
Kitchen inventory before service: Cutlery(knives=100, forks=100)
Kitchen inventory after service: Cutlery(knives=112, forks=96)

```

尽管与上次运行相比，错误仍然存在，但出现的错误*数量和种类*不同。这真是荒谬！记住，这些机器人都是异常精良构造的，不会出错。问题到底出在哪里？

让我们总结一下当前的情况：

+   你的 ThreadBot 代码非常简单易读。逻辑没有问题。

+   你有一个工作正常的测试（100 张桌子），可以可靠地通过。

+   你有一个更长的测试（10000 张桌子），可以可靠地失败。

+   较长的测试以*不同且不可重现的方式*失败了。

这些是竞态条件错误的几个典型迹象。有经验的读者已经看到了原因，所以现在让我们来调查这个问题。这一切都归结为我们 `Cutlery` 类内部的这个方法：

```py
def change(self, knives, forks):
    self.knives += knives
    self.forks += forks
```

内联求和运算 `+=` 在内部实现（在 Python 解释器的 C 代码内部）作为几个单独的步骤：

1.  将当前值 `self.knives` 读入临时位置。

1.  将新值 `knives` 添加到该临时位置中的值。

1.  将新总数从临时位置复制回原始位置。

抢占式多任务的问题在于，任何忙于这些步骤的线程都可以在*任何时候*被中断，而不同的线程可以有机会通过相同的步骤工作。

在这种情况下，假设 ThreadBot *A* 执行步骤 1，然后操作系统调度器暂停 *A* 并切换到 ThreadBot *B*。 *B* *也* 读取 `self.knives` 的当前值；然后执行返回到 *A*。 *A* 增加其总数并写回，但接着 *B* 继续从它被暂停的地方（步骤 1 后）执行，并增加并写回 *它* 的新总数，从而*擦除*了 *A* 所做的更改！

###### 警告

尽管这听起来复杂，但这个竞态条件的例子几乎是可能的最简单情况。我们能够检查*所有*代码，甚至还有测试可以按需重现问题。在真实世界中，对于大型项目，试想一下它可能变得多么困难！

可以通过在修改共享状态的地方（假设我们在 `Cutlery` 类中添加了 `threading.Lock`）周围放置*锁*来解决此问题：

```py
def change(self, knives, forks):
    with self.lock:
      self.knives += knives
      self.forks += forks
```

但这要求您知道所有状态将在多个线程之间共享的地方。当您控制所有源代码时，这种方法是可行的，但当使用许多第三方库时（在 Python 中由于美妙的开源生态系统而可能），它变得非常困难。

请注意，仅通过查看源代码本身是不可能看到竞态条件的。这是因为源代码并不提供任何关于执行将在何处在线程之间切换的提示。无论如何，这也不会有用，因为操作系统可以在几乎任何地方在线程之间切换。

另一个，更好的解决方案——异步编程的要点是修改我们的代码，以便仅使用一个 ThreadBot 并配置它根据需要在*所有*表之间移动。对于我们的案例研究来说，这意味着厨房里的刀具和叉子将仅由单个线程修改。

而且更好的是，在我们的异步程序中，我们可以清楚地看到上下文在多个并发协程之间切换的地方，因为`await`关键字明确指示了这样的位置。我决定不在这里展示案例研究的异步版本，因为第三章详细解释了如何深入使用`asyncio`。但如果你的好奇心难以满足，附录 B-1 中有一个带注释的示例；在你阅读下一章之后可能才会有所理解！

¹ 一个 32 位进程的理论地址空间为 4 GB，但操作系统通常会保留其中的一部分。通常，作为可寻址的虚拟内存，进程只剩下 3 GB，但在某些操作系统上，这个值可能低至 2 GB。请把本节提到的数字理解为概括，而非绝对值。这里有太多平台特定的（和历史上敏感的）细节不便深入讨论。
