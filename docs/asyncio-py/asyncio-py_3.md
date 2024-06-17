# 第三章 Asyncio 演练

> Asyncio 为 Python 提供了另一种轻量级的并发编程工具，比线程或多进程更轻量级。简单来说，它通过一个事件循环执行一系列任务来实现这一点，其中的一个关键区别是每个任务可以选择何时将控制权返回给事件循环。
> 
> Philip Jones，[“理解 Asyncio”](http://bit.ly/2EPys9Q)

Python 中的`asyncio` API 之所以复杂，是因为它旨在解决不同群体的不同问题。不幸的是，很少有指导可以帮助你找出`asyncio`的哪些部分对*你*而言是重要的。

我的目标是帮助你理清楚这一点。Python 中 async 功能的两个主要目标受众如下：

最终用户开发者

这些人想要使用`asyncio`来开发应用程序。我假设你是这一群体中的一员。

框架开发者

这些人想要创建最终用户开发者可以在其应用程序中使用的框架和库。

当今社区对`asyncio`存在很多困惑，主要是因为缺乏对这种差异的理解。例如，官方 Python 文档中关于`asyncio`的内容更适合框架开发者而不是最终用户。这意味着阅读这些文档的最终用户开发者很快就会被表面上的复杂性震惊到。在能够实际操作之前，你被迫先完全理解这些内容。

我希望这本书能够帮助你区分出对最终用户开发者重要的 Asyncio 特性和对框架开发者重要的特性。

###### 提示

如果你对像 Asyncio 这样的并发框架在内部是如何构建的低级细节感兴趣，我强烈推荐 Dave Beazley 的一场精彩演讲，[“从头开始的 Python 并发：LIVE！”](https://oreil.ly/_68Rm)，在这场演讲中，他演示了如何组装一个类似 Asyncio 的简化版本的异步框架。

我的目标只是让你对 Asyncio 的基本构建块有最基本的理解，足以让你能够编写简单的程序，当然也足以让你深入参考更完整的资料。¹

首先，我们有一个“快速入门”部分，介绍了 Asyncio 应用程序中最重要的构建块。

# 快速入门

> 你只需要了解七个函数就可以使用 Asyncio 进行[日常使用]。
> 
> Yury Selivanov，PEP 492 的作者，该提案为 Python 添加了`async`和`await`关键字

深入阅读 Asyncio 的[官方文档](https://oreil.ly/4Y_Pd)可能会让人望而却步。有许多章节涉及新的、神秘的术语和概念，即使是经验丰富的 Python 程序员也会感到陌生，因为 Asyncio 在 Python 中是一个非常新的东西。稍后我将详细解释如何解读`asyncio`模块文档，但现在您需要知道，您需要关注的实际表面领域要比看上去的`asyncio`库小得多。

Yury Selivanov，[PEP 492](https://oreil.ly/I3K7H)的作者及异步 Python 的全面主要贡献者，在他的 PyCon 2016 演讲“async/await in Python 3.5 and Why It Is Awesome”中解释了`asyncio`模块中的许多 API 实际上是为框架设计者而不是终端用户开发者设计的。在那次演讲中，他强调了终端用户应该关注的主要特性。这些特性是`asyncio`API 的一个小子集，可以总结如下：

+   启动`asyncio`事件循环

+   调用`async`/`await`函数

+   创建一个*任务*以在循环中运行

+   等待多个任务完成

+   在所有并发任务完成后关闭循环

在本节中，我们将看看这些核心特性，并了解如何在 Python 中使用基于事件的编程迅速入门。

在 Python 中，Asyncio 的“Hello World”看起来像示例 3-1。

##### 示例 3-1\. Asyncio 的“Hello World”

```py
# quickstart.py
import asyncio, time

async def main():
    print(f'{time.ctime()} Hello!')
    await asyncio.sleep(1.0)
    print(f'{time.ctime()} Goodbye!')

asyncio.run(main())  ![1](img/1.png)
```

![1](img/#co_asyncio_walk_through_CO1-1)

`asyncio`提供了一个`run()`函数来执行`async def`函数以及从那里调用的所有其他协程，比如在`main()`函数中的`sleep()`。

这是运行示例 3-1 的输出：

```py
$ python quickstart.py
Sun Aug 18 02:14:34 2019 Hello!
Sun Aug 18 02:14:35 2019 Goodbye!

```

在实践中，基于 Asyncio 的大部分代码将使用这里显示的`run()`函数，但理解更多关于该函数为您做了什么是很重要的。这种理解很重要，因为它将影响您如何设计更大的应用程序。

示例 3-2 是我将称之为“Hello-ish World”的示例。它与`run()`的功能不完全相同，但足以介绍我们在本书其余部分将构建的思想。您需要基本的协程知识（本章后面会深入讨论），但现在试着跟上并专注于高级概念。

##### 示例 3-2\. Asyncio 的“Hello-ish World”

```py
# quickstart.py
import asyncio
import time

async def main():
    print(f"{time.ctime()} Hello!")
    await asyncio.sleep(1.0)
    print(f"{time.ctime()} Goodbye!")

loop = asyncio.get_event_loop()  ![1](img/1.png)
task = loop.create_task(main())  ![2](img/2.png)
loop.run_until_complete(task)  ![3](img/3.png)
pending = asyncio.all_tasks(loop=loop)
for task in pending:
    task.cancel()
group = asyncio.gather(*pending, return_exceptions=True)  ![4](img/4.png)
loop.run_until_complete(group)  ![3](img/3.png)
loop.close()  ![5](img/5.png)
```

![1](img/#co_asyncio_walk_through_CO2-1)

`loop` = *`asyncio.get_event_loop()`*

在运行任何协程之前，你需要一个循环实例，这是你获取它的方式。事实上，无论你在哪里调用它，只要你只使用单个线程，`get_event_loop()`每次都会给你相同的`loop`实例。² 如果你在`async def`函数内部，应该使用`asyncio.get_running_loop()`，它总是给你预期的内容。这些内容在本书的后面部分将详细介绍。

![2](img/#co_asyncio_walk_through_CO2-2)

`task` = *`loop.create_task(coro)`*

在这种情况下，具体的调用是`loop.create_task(main())`。你的协程函数在你执行此操作之前不会被执行。我们说`create_task()` *调度* 你的协程在循环上运行。³ 返回的`task`对象可以用于监视任务的状态（例如，它是仍在运行还是已经完成），并且还可以用于从已完成的协程中获取结果值。你可以用`task.cancel()`取消任务。

![3](img/#co_asyncio_walk_through_CO2-3)

*`loop.run_until_complete(coro)`*

这个调用会*阻塞*当前线程，通常是主线程。注意，`run_until_complete()`只会在给定的*`coro`*完成之前保持循环运行，但是在循环运行时，所有*其他*安排在循环上的任务也会运行。在内部，`asyncio.run()`为你调用`run_until_complete()`，因此以相同的方式阻塞主线程。

![4](img/#co_asyncio_walk_through_CO2-4)

`group` = *`asyncio.gather(task1, task2, task3)`*

当程序的“主”部分因接收到[进程信号](https://oreil.ly/KfOmB)或某些代码调用`loop.stop()`而解除阻塞时，`run_until_complete()`之后的代码将运行。如本例所示的标准习语是收集仍未完成的任务，取消它们，然后再次使用`loop.run_until_complete()`直到这些任务完成。`gather()`是执行收集的方法。注意，`asyncio.run()`将执行取消、收集和等待挂起任务完成的所有操作。

![5](img/#co_asyncio_walk_through_CO2-6)

*`loop.close()`*

`loop.close()`通常是最后的操作：它必须在停止的循环上调用，并且它将清除所有队列并关闭执行器。*停止*的循环可以重新启动，但是*关闭*的循环则永远消失。在返回之前，`asyncio.run()`会关闭循环。这是可以接受的，因为每次调用`run()`时都会创建一个新的事件循环。

示例 3-1 表明，如果使用`asyncio.run()`，这些步骤都是不必要的：它们都为你完成了。然而，理解这些步骤很重要，因为在实践中会遇到更复杂的情况，你需要额外的知识来处理它们。这本书的后面部分将详细涵盖其中的几个。

###### 注意

前面的示例仍然过于简单，不足以在实际环境中使用。需要更多关于正确关闭处理的信息。示例的目标仅是介绍 `asyncio` 中最重要的函数和方法。有关正确关闭处理的更实用信息在 “启动和关闭（优雅地！）” 中呈现。

`asyncio` 在 Python 中公开了与事件循环相关的大量底层机制，并且需要你意识到像生命周期管理这样的方面。例如，这与 Node.js 不同，Node.js 也包含一个事件循环，但它将其隐藏得相对较深。然而，一旦你使用了 `asyncio` 一段时间，你会开始注意到启动和关闭事件循环的模式与此处呈现的代码并没有太大的不同。我们将在本书后面更详细地探讨管理循环生命周期的一些细微差别。

我在上一个示例中遗漏了一些内容。你需要了解的最后一个基本功能是如何运行 *阻塞* 函数。关于协作式多任务的事情是，你需要所有 I/O 绑定函数……嗯，协作，并且这意味着允许使用关键字 `await` 切换回循环。今天大部分的 Python 代码并没有这样做，而是依赖于你在线程中运行这些函数。在广泛支持 `async def` 函数之前，你会发现使用这些阻塞库是不可避免的。

为此，`asyncio` 提供了一个 API，该 API 与 `concurrent.futures` 包中的 API 非常相似。该包提供了一个 `ThreadPoolExecutor` 和一个 `ProcessPoolExecutor`。默认情况下是基于线程的，但可以使用基于线程或基于池的执行器。我在前面的示例中省略了执行器的考虑，因为它们可能会使基本部分的描述变得模糊。现在这些已经涵盖了，我们可以直接看看执行器了。

有一些需要注意的怪癖。让我们看看 示例 3-3 中的代码样本。

##### 示例 3-3\. 基本执行器接口

```py
# quickstart_exe.py
import time
import asyncio

async def main():
    print(f'{time.ctime()} Hello!')
    await asyncio.sleep(1.0)
    print(f'{time.ctime()} Goodbye!')

def blocking():  ![1](img/1.png)
    time.sleep(0.5)  ![2](img/2.png)
    print(f"{time.ctime()} Hello from a thread!")

loop = asyncio.get_event_loop()
task = loop.create_task(main())

loop.run_in_executor(None, blocking)  ![3](img/3.png)
loop.run_until_complete(task)

pending = asyncio.all_tasks(loop=loop)  ![4](img/4.png)
for task in pending:
    task.cancel()
group = asyncio.gather(*pending, return_exceptions=True)
loop.run_until_complete(group)
loop.close()
```

![1](img/#co_asyncio_walk_through_CO3-1)

`blocking()` 在内部调用传统的 `time.sleep()`，这本质上会阻塞主线程并阻止你的事件循环运行。这意味着你不能将此函数作为协程—事实上，你甚至不能从主线程中的 *任何地方* 调用此函数，因为 `asyncio` 循环正是在那里运行的。我们通过在 *执行器* 中运行此函数来解决这个问题。

![2](img/#co_asyncio_walk_through_CO3-2)

与本节无关，但稍后在本书中需要牢记的一点是，阻塞的睡眠时间（0.5 秒）比非阻塞的睡眠时间（1 秒）短。这使得代码示例看起来整洁而整齐。在 “等待执行器关闭期间” 中，我们将探讨在关闭序列期间执行器函数超过其异步对应物时会发生什么情况。

![3](img/#co_asyncio_walk_through_CO3-3)

`await loop.run_in_executor(None, func)`

这是我们 `asyncio` 必须掌握的关键功能列表的最后一项。有时您需要在单独的线程甚至单独的进程中运行事物：这种方法正是用于此目的。在此，我们将我们的阻塞函数传递到默认执行器中运行。⁴ 注意，`run_in_executor()` 不会阻塞主线程：它只是调度执行器任务以运行（它返回一个 `Future`，这意味着如果该方法在另一个协程函数中调用，您可以 `await` 它）。执行器任务将仅在调用 `run_until_complete()` 后开始执行，这允许事件循环开始处理事件。

![4](img/#co_asyncio_walk_through_CO3-4)

进一步参考第 2 处提示：`pending` 中的任务集合不包括在 `run_in_executor()` 中调用的 `blocking()`。对于返回 `Future` 而不是 `Task` 的任何调用，情况都是如此。文档在指定返回类型方面做得非常好，因此你会在那里看到返回类型；只需记住 `all_tasks()` 确实只返回 `Task`，而不是 `Future`。

这是运行此脚本的输出：

```py
$ python quickstart_exe.py
Sun Aug 18 01:20:42 2019 Hello!
Sun Aug 18 01:20:43 2019 Hello from a thread!
Sun Aug 18 01:20:43 2019 Goodbye!

```

现在您已经看到了 `asyncio` 对于最终用户开发者需求的最重要部分，是时候扩展我们的范围并将 `asyncio` API 排列成一种层次结构了。这将使得从文档中获取所需内容并了解如何获取更加容易。

# asyncio 之塔

正如您在前面的部分中看到的，要使用 `asyncio` 作为最终用户开发者，您只需了解少数几个命令即可。不幸的是，`asyncio` 的文档提供了大量的 API，并且以非常“扁平”的格式呈现，这使得很难分辨哪些是用于常规使用的东西，哪些是提供给框架设计者的设施。

当框架设计者查看同样的文档时，他们寻找*钩点*，以便将其新框架或第三方库连接起来。在本节中，我们将通过框架设计者的眼光来看 `asyncio`，以了解他们可能如何构建一个新的异步兼容库。希望这将有助于进一步界定您在自己工作中需要关注的功能。

从这个角度来看，把`asyncio`模块视为一个层次结构会更有用（而不是一个平面列表），每个级别都建立在前一个级别的规范之上。不幸的是，情况并非如此简洁，我在 表格 3-1 中对排列进行了一些改动，但希望这能给你提供`asyncio` API 的另一种视角。

###### 警告

表格 3-1，以及这里给出的“级别”的名称和编号，完全是我自己创造的，旨在增加一些结构以帮助解释`asyncio` API。专业读者可能会以不同的顺序排列这些内容，这也没关系！

表 3-1\. `asyncio` 的特性按层次结构排列；对于最终用户开发者而言，最重要的级别已经用粗体标出。

| 级别 | 概念 | 实现 |
| --- | --- | --- |
| **第 9 级** | **网络：流** | `StreamReader`, `StreamWriter`, `asyncio.open_connection()`, `asyncio.start_server()` |
| 第 8 级 | 网络：TCP & UDP | `Protocol` |
| 第 7 级 | 网络：传输 | `BaseTransport` |
| **第 6 级** | **工具** | `asyncio.Queue` |
| **第 5 级** | **子进程和线程** | `run_in_executor()`, `asyncio.subprocess` |
| 第 4 级 | 任务 | `asyncio.Task`, `asyncio.create_task()` |
| 第 3 级 | Futures | `asyncio.Future` |
| **第 2 级** | **事件循环** | `asyncio.run()`, `BaseEventLoop` |
| **第 1 级 (基础)** | **协程** | `async def`, `async with`, `async for`, `await` |

在最基础的第 1 级，我们有你在本书早些时候已经看到的协程。这是可以开始考虑设计第三方框架的最低级别，令人惊讶的是，这种方法在当前有两个异步框架中相当受欢迎：[Curio](https://oreil.ly/Zu0lP) 和 [Trio](https://oreil.ly/z2lZY)。这两者仅依赖于 Python 中的原生协程，完全不依赖于`asyncio`库模块中的任何东西。

下一个级别是事件循环。协程本身并不实用：如果没有一个循环来运行它们，它们什么也做不了（因此，必然地，Curio 和 Trio 实现了它们自己的事件循环）。`asyncio` 提供了循环的*规范*，`AbstractEventLoop`，以及*实现*，`BaseEventLoop`。

规范与实现之间的明确分离使得第三方开发者能够创建事件循环的替代实现，这在 [uvloop](https://oreil.ly/2itn_) 项目中已经实现，该项目提供比`asyncio`标准库模块中更快的循环实现。重要的是，uvloop 只是简单地“插入”到层次结构中，并且仅仅替换堆栈中的循环部分。能够做出这些选择的能力正是为什么`asyncio` API 被设计成这样，各个部分之间有着明确的分离。

层级 3 和 4 带来了未来和任务，它们非常密切相关；它们之间的分离只是因为`Task`是`Future`的子类，但可以轻易地被视为同一层级。`Future`实例代表着某种正在进行的操作，将通过事件循环上的通知返回结果，而`Task`则代表在事件循环上运行的协程。简而言之：未来是“循环感知”的，而任务既是“循环感知”又是“协程感知”的。作为最终用户开发者，你将更多地使用任务而不是未来，但对于框架设计者而言，比例可能相反，这取决于具体细节。

Tier 5 代表着启动和等待必须在单独线程或甚至单独进程中运行的工作的设施。

Tier 6 代表额外的异步感知工具，例如`asyncio.Queue`。我本可以将这个层级放在网络层级之后，但我认为在我们查看 I/O 层之前，先把所有的协程感知 API 处理掉更整洁。`asyncio`提供的`Queue`与`queue`模块中的线程安全`Queue`有非常相似的 API，只是`asyncio`版本在`get()`和`put()`上需要使用`await`关键字。你不能直接在协程内使用`queue.Queue`，因为它的`get()`会阻塞主线程。

最后，我们有网络 I/O 层级 7 到 9。作为最终用户开发者，最方便的 API 是位于层级 9 的流 API。我将流 API 放置在塔的最高抽象层级。直接在流层级下面的是协议 API（层级 8），这是一个更细粒度的 API；你*可以*在所有可以使用流层级的情况下使用协议层级，但使用流会更简单。最终的网络 I/O 层级是传输层级（层级 7）。除非你正在创建一个供他人使用并需要定制传输设置的框架，否则你不太可能直接使用这个层级。

在“快速入门”中，我们仅仅涉及了使用`asyncio`库所需的绝对最低限度。现在我们已经全面了解了整个`asyncio`库的 API 是如何组织的，我想重新审视那个功能列表，并强调你可能需要学习哪些部分。

当学习如何使用`asyncio`库模块来编写网络应用程序时，这些层级是最重要关注的：

Tier 1

理解如何编写`async def`函数并使用`await`调用和执行其他协程是至关重要的。

Tier 2

理解如何启动、关闭和与事件循环交互是至关重要的。

Tier 5

执行器在您的异步应用程序中使用阻塞代码是必要的，而现实是大多数第三方库还不兼容`asyncio`。一个很好的例子是 SQLAlchemy 数据库 ORM 库，目前还没有功能相当的`asyncio`兼容替代品。

等级 6

如果您需要向一个或多个长时间运行的协程提供数据，最好的方法是使用 `asyncio.Queue`。这与使用 `queue.Queue` 在线程之间分发数据的策略完全相同。Asyncio 版本的 `Queue` 使用与标准库队列模块相同的 API，但使用协程而不是像 `get()` 这样的阻塞方法。

等级 9

流 API 为您提供了处理网络套接字通信的最简单方法，这是您应该开始原型化网络应用程序想法的地方。您可能会发现需要更细粒度的控制，那么您可以切换到协议 API，但在大多数项目中，保持简单通常是最好的，直到您完全了解您要解决的问题。

当然，如果您使用的是像 `aiohttp` 这样直接处理所有套接字通信的`asyncio`兼容的第三方库，您将不需要直接使用`asyncio`网络层。在这种情况下，您必须大量依赖提供的库文档。

`asyncio` 库试图为终端用户开发人员和框架设计者提供足够的功能。不幸的是，这意味着`asyncio` API 可能显得有些冗长。我希望本节已经提供了足够的路线图，帮助您找出您需要的部分。

在接下来的几节中，我们将更详细地查看前面列表中的组成部分。

###### 提示

[pysheeet](http://bit.ly/2toWDL1) 网站提供了`asyncio` API 的深入摘要（或“速查表”）；每个概念都附带一个简短的代码片段。内容非常密集，所以我不建议初学者使用，但如果您有 Python 经验，并且只有在以代码形式呈现新的编程信息时才“get 到”，这肯定是一个有用的资源。

# 协程

让我们从非常基础的开始：什么是协程？

我在这一节的目标是帮助您理解诸如*协程对象*和*异步函数*之类术语背后的具体含义。接下来的示例将展示通常情况下大多数程序不需要的低级交互；然而，这些示例将帮助您更清晰地理解 Asyncio 的基本部分，并且会使后续章节更容易理解。

下面的示例可以在 Python 3.8 交互模式中重现，并且我建议您通过自己输入它们、观察输出并可能尝试不同的与`async`和`await`交互方式来自己完成它们。

###### 注意

`asyncio` 首次添加到 Python 3.4 中，但使用 `async def` 和 `await` 的新协程语法仅在 Python 3.5 中添加。人们在 3.4 中如何使用 `asyncio`？他们以非常特殊的方式使用 *生成器* 来模拟协程。在一些旧代码库中，您会看到带有 `@asyncio.coroutine` 装饰器并包含 `yield from` 语句的生成器函数。使用新的 `async def` 创建的协程现在被称为 *本地协程*，因为它们作为协程内置到语言中。本书完全忽略了基于生成器的旧协程。

## 新的 async def 关键字

让我们从最简单的事情开始，如 示例 3-4 所示。

##### 示例 3-4\. 异步函数是函数，而不是协程

```py
>>> async def f():  ![1](img/1.png)
...   return 123
...
>>> type(f)  ![2](img/2.png)
<class 'function'>
>>> import inspect  ![3](img/3.png)
>>> inspect.iscoroutinefunction(f)  ![4](img/4.png)
True
```

![1](img/#co_asyncio_walk_through_CO4-1)

这是最简单的协程声明：它看起来像一个普通函数，但是以关键字 `async def` 开始。

![2](img/#co_asyncio_walk_through_CO4-2)

惊喜！`f` 的确切类型并不是“协程”，它只是一个普通函数。虽然通常将 `async def` 函数称为协程，严格来说它们被 Python 视为 *协程函数*。这种行为与 Python 中生成器函数的工作方式相同：

```py
>>> def g():
...     yield 123
...
>>> type(g)
<class 'function'>
>>> gen = g()
>>> type(gen)
<class 'generator'>
```

即使 `g` 有时被错误地称为“生成器”，它仍然是一个函数，只有在 *评估* 此函数时才会返回生成器。协程函数的工作方式完全相同：您需要 *调用* `async def` 函数才能获取协程对象。

![3](img/#co_asyncio_walk_through_CO4-3)

标准库中的 `inspect` 模块可以提供比 `type()` 内置函数更好的反射能力。

![4](img/#co_asyncio_walk_through_CO4-4)

有一个 `iscoroutinefunction()` 函数可以让你区分普通函数和协程函数。

回到我们的 `async def f()`，示例 3-5 展示了我们调用它时会发生什么。

##### 示例 3-5\. async def 函数返回一个协程对象

```py
>>> coro = f()
>>> type(coro)
<class 'coroutine'>
>>> inspect.iscoroutine(coro)
True
```

这使我们回到了最初的问题：协程究竟是什么？*协程* 是一个 *对象*，它封装了在完成之前已被挂起的底层函数的能力。如果这听起来耳熟，那是因为协程与生成器非常相似。事实上，在 Python 3.5 引入 *本地* 协程之前，使用 Python 3.4 的 `asyncio` 库已经可以通过使用带有特殊装饰器的普通生成器来使用。⁵ 新的 `async def` 函数（及其返回的协程）行为类似于生成器是一件并不奇怪的事。

我们可以进一步玩弄协程对象，看看 Python 如何利用它们。最重要的是，我们想看看 Python 如何能够在协程之间“切换”执行。首先让我们看看如何获取返回值。

当协程*返回*时，实际上是引发了一个`StopIteration`异常。示例 3-6 在同一会话中延续了前面示例的内容，阐明了这一点。

##### 示例 3-6\. 协程内部：使用 send() 和 StopIteration

```py
>>> async def f():
...    return 123
>>> coro = f()
>>> try:
...   coro.send(None)  ![1](img/1.png)
... except StopIteration as e:
...   print('The answer was:', e.value)  ![2](img/2.png)
...
The answer was: 123
```

![1](img/#co_asyncio_walk_through_CO5-1)

协程通过“发送”`None`来*启动*。在内部，这就是*事件循环*将要对你的宝贵协程做的事情；你永远不需要手动执行这个过程。你创建的所有协程都会通过`loop.create_task(*coro*)`或`await *coro*`来执行。是`loop`在幕后执行`.send(None)`。

![2](img/#co_asyncio_walk_through_CO5-2)

当协程*返回*时，会引发一种特殊的异常，称为`StopIteration`。注意，我们可以通过异常本身的`value`属性访问协程的返回值。再次强调，你无需知道它是如何工作的：从你的角度来看，`async def`函数将简单地使用`return`语句返回一个值，就像普通函数一样。

这两点，`send()`和`StopIteration`，分别定义了执行协程的开始和结束。到目前为止，这似乎只是运行函数的一种非常复杂的方式，但没关系：*事件循环*将负责使用这些低级别的内部驱动协程。从你的角度来看，你只需将协程安排到循环中执行，它们将按顺序自上而下执行，几乎像普通函数一样。

下一步是看看如何暂停协程的执行。

## 新的 await 关键字

这个新关键字[`await`](https://oreil.ly/uk4H3)始终接受一个参数，并且仅接受称为*可等待对象*的东西，其定义如下（仅此而已！）：

+   一个协程（即调用了`async def`函数的*结果*）。⁶

+   任何实现了`__await__()`特殊方法的对象。这个特殊方法*必须*返回一个迭代器。

第二种可等待对象超出了本书的范围（你在日常`asyncio`编程中永远不会需要它），但第一个用例相当简单，如示例 3-7 所示。

##### 示例 3-7\. 在协程上使用 await

```py
async def f():
    await asyncio.sleep(1.0)
    return 123

async def main():
    result = await f()  ![1](img/1.png)
    return result
```

![1](img/#co_asyncio_walk_through_CO6-1)

调用`f()`会产生一个协程；这意味着我们可以对其使用`await`。当`f()`完成时，`result`变量的值将为`123`。

在结束本节并继续进行事件循环之前，查看一下如何将异常传递给协程会很有用。 这通常用于取消：当您调用 `task.cancel()` 时，事件循环将内部使用 `coro.throw()` 来在您的协程内部引发 `asyncio.CancelledError`（示例 3-8）。

##### 示例 3-8\. 使用 coro.throw() 将异常注入到协程中

```py
>>> coro = f()  ![1](img/1.png)
>>> coro.send(None)
>>> coro.throw(Exception, 'blah')  ![2](img/2.png)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in f
Exception: blah
blah
```

![1](img/#co_asyncio_walk_through_CO7-1)

与以前一样，从协程函数 `f()` 创建了一个新的协程。

![2](img/#co_asyncio_walk_through_CO7-2)

我们不再执行另一个 `send()`，而是调用 `throw()` 并提供一个异常类和一个值。 这会在我们的协程内部，在 `await` 点引发异常。

`throw()` 方法（在 `asyncio` 内部）用于 *任务取消*，我们也可以非常容易地演示。 我们甚至将继续在 示例 3-9 中处理取消，这次在一个新的协程中。

##### 示例 3-9\. 使用 CancelledError 进行协程取消

```py
>>> import asyncio
>>> async def f():
...     try:
...         while True: await asyncio.sleep(0)
...     except asyncio.CancelledError:  ![1](img/1.png)
...         print('I was cancelled!')  ![2](img/2.png)
...     else:
...         return 111
>>> coro = f()
>>> coro.send(None)
>>> coro.send(None)
>>> coro.throw(asyncio.CancelledError) ![3](img/3.png)
I was cancelled!  ![4](img/4.png)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration  ![5](img/5.png)
```

![1](img/#co_asyncio_walk_through_CO8-1)

现在我们的协程函数处理了一个异常。 事实上，它处理了在整个 `asyncio` 库中用于任务取消的 *特定* 异常类型：`asyncio.CancelledError`。 请注意，异常是从外部注入到协程中的； 即，由事件循环，我们仍然使用手动的 `send()` 和 `throw()` 命令进行模拟。 在真实代码中，您将在稍后看到，当任务被取消时，`CancelledError` 会在任务封装的协程内部引发。

![2](img/#co_asyncio_walk_through_CO8-2)

一个简单的消息，表明任务被取消了。 请注意，通过处理异常，我们确保它不再传播，并且我们的协程将会 `return`。

![3](img/#co_asyncio_walk_through_CO8-3)

这里我们抛出 `CancelledError` 异常。

![4](img/#co_asyncio_walk_through_CO8-4)

不出所料，我们看到取消消息被打印出来了。

![5](img/#co_asyncio_walk_through_CO8-5)

我们的协程正常退出。（回想一下，`StopIteration` 异常是协程正常退出的方式。）

为了强调任务取消只是常规异常抛出（和处理）的要点，让我们看看 示例 3-10，在那里我们吸收取消并转移到另一个协程。

##### 示例 3-10\. 仅供教育目的使用——不要这样做！

```py
>>> async def f():
...     try:
...         while True: await asyncio.sleep(0)
...     except asyncio.CancelledError:
...         print('Nope!')
...         while True: await asyncio.sleep(0) ![1](img/1.png)
...     else:
...         return 111
>>> coro = f()
>>> coro.send(None)
>>> coro.throw(asyncio.CancelledError)  ![2](img/2.png)
Nope!
>>> coro.send(None)  ![3](img/3.png)
```

![1](img/#co_asyncio_walk_through_CO9-1)

如果在取消之后，我们只是立即回到等待另一个可等待对象，会发生什么？

![2](img/#co_asyncio_walk_through_CO9-2)

不足为奇的是，我们的外部协程继续存在，并且立即在 *新* 协程内部再次暂停。

![3](img/#co_asyncio_walk_through_CO9-3)

一切都按预期进行，我们的协程继续按预期暂停和恢复。

当然，不言而喻的是，您绝不能真的这样做！如果您的协程收到取消信号，那就是明确的指令只做必要的清理工作并退出。不要简单地忽略它。

到了这一步，通过手动执行所有`.send(None)`调用来假装成事件循环已经相当疲劳了，因此在示例 3-11 中，我们将引入`asyncio`提供的循环，并相应地清理前面的示例。

##### 示例 3-11\. 使用事件循环执行协程

```py
>>> async def f():
...     await asyncio.sleep(0)
...     return 111
>>> loop = asyncio.get_event_loop()  ![1](img/1.png)
>>> coro = f()
>>> loop.run_until_complete(coro)  ![2](img/2.png)
111
```

![1](img/#co_asyncio_walk_through_CO10-1)

获取一个循环。

![2](img/#co_asyncio_walk_through_CO10-2)

将协程运行到完成。在内部，这为我们做了所有那些`.send(None)`方法调用，并通过`StopIteration`异常检测我们协程的完成，该异常还包含了我们的返回值。

# 事件循环

前一节展示了`send()`和`throw()`方法如何与协程交互，但这仅是为了帮助您理解协程本身的结构。`asyncio`中的事件循环处理所有协程之间的切换，以及捕获`StopIteration`异常——以及更多内容，如监听套接字和文件描述符的事件。

您可以完全不需要直接使用事件循环来编写您的`asyncio`代码，可以完全使用`await`调用，通过`asyncio.run(*coro*)`调用来启动。然而，有时可能需要与事件循环进行某种程度的交互，在这里我们将讨论如何获取它。

有两种方法：

*推荐*

`asyncio.get_running_loop()`，可在协程上下文内部调用

*不建议使用*

`asyncio.get_event_loop()`，可在任何地方调用

您会在许多现有代码中看到不建议使用的函数，因为较新的函数`get_running_loop()`是在 Python 3.8 中引入的。因此，了解旧方法的基本工作原理在实践中将非常有用，因此我们将同时查看两者。让我们从示例 3-12 开始。

##### 示例 3-12\. 始终获取相同的事件循环

```py
>>> loop = asyncio.get_event_loop()
>>> loop2 = asyncio.get_event_loop()
>>> loop is loop2  ![1](img/1.png)
True
```

![1](img/#co_asyncio_walk_through_CO11-1)

标识符`loop`和`loop2`均指向同一实例。

这意味着，如果您在协程函数内部需要访问循环实例，调用`get_event_loop()`或`get_running_loop()`来获取它是可以的。您*不需要*通过所有函数显式传递`loop`参数。

如果您是框架设计者，情况就不同了：最好设计您的函数接受一个`loop`参数，以防您的用户正在与[事件循环策略](https://oreil.ly/oMe9w)进行一些不寻常的操作。本书不涉及策略，我们将不再多说。

因此，如果 `get_event_loop()` 和 `get_running_loop()` 的功能相同，为什么它们都存在呢？`get_event_loop()` 方法仅在*同一线程*内工作。事实上，如果在新线程中调用 `get_event_loop()`，除非您使用 `new_event_loop()` 明确创建一个新的循环，并通过调用 `set_event_loop()` 将该新实例设置为该线程的*唯一*循环，否则它将失败。大多数情况下，我们只需要（并希望！）在单线程中运行一个单独的循环实例。这几乎是异步编程的整个初衷。

相比之下，`get_running_loop()`（推荐的方法）将始终按预期运行：因为它只能在协程、任务或从这些中调用的函数的上下文中调用，它总是提供*当前*正在运行的事件循环，这几乎总是您想要的。

引入 `get_running_loop()` 还简化了后台任务的生成。考虑 示例 3-13，在其中创建了额外的任务且*未*等待。

##### 示例 3-13\. 创建任务

```py
async def f():
    # Create some tasks!
    loop = asyncio.get_event_loop()
    for i in range():
        loop.create_task(<some other coro>)
```

在这个例子中，意图是在协程内部启动全新的任务。通过不等待它们，我们确保它们将独立于协程函数 `f()` 内部的执行上下文而运行。实际上，在由它启动的任务完成之前，`f()` 将会退出。

在 Python 3.7 之前，必须首先获取 `loop` 实例以安排任务，但引入 `get_running_loop()` 后，出现了其他使用它的 `asyncio` 函数，比如 `asyncio.create_task()`。从 Python 3.7 开始，生成异步任务的代码现在看起来像是 示例 3-14。

##### 示例 3-14\. 以现代方式创建任务

```py
import asyncio

async def f():
    # Create some tasks!
    for i in range():
        asyncio.create_task(<some other coro>)
```

还可以使用另一个名为 `asyncio.ensure_future()` 的低级函数以与 `create_task()` 相同的方式生成任务，您可能仍然会在旧的 `asyncio` 代码中看到对 `ensure_future()` 的调用。我考虑避免讨论 `ensure_future()`，但它是一个关于仅供框架设计者使用的 `asyncio` API 的完美案例，但这使得应用程序开发人员最初理解 `asyncio` 变得更加困难。`asyncio.create_task()` 与 `asyncio.ensure_future()` 之间的区别对于许多新手来说是微妙且令人困惑的。我们将在下一节探讨这些区别。

# 任务和未来

之前我们讨论了协程及其在循环中运行以便有用的方式。现在我想简要讨论 `Task` 和 `Future` API。您将与之最多打交道的是 `Task`，因为大部分工作将涉及使用 `create_task()` 函数运行协程，就像在“快速入门”中描述的那样。`Future` 类实际上是 `Task` 的超类，并为与循环的交互提供所有功能。

一个简单的思路是这样的：`Future` 表示某个活动的未来完成状态，并由循环管理。`Task` 完全相同，但特定的“活动”是一个协程——可能是您使用 `async def` 函数和 `create_task()` 创建的协程之一。

`Future` 类代表与循环交互的*状态*。这种描述太模糊了，因此您可以将 `Future` 实例视为完成状态的切换器，而不是一个有用的东西。当创建 `Future` 实例时，切换器设置为“尚未完成”，但稍后将“完成”。实际上，`Future` 实例有一个称为 `done()` 的方法，允许您检查状态，如 Example 3-15 中所示。

##### Example 3-15\. 使用 `done()` 检查完成状态

```py
>>> from asyncio import Future
>>> f = Future()
>>> f.done()
False
```

`Future` 实例还可以执行以下操作：

+   设置“结果”值（使用 `.set_result(*value*)` 设置，使用 `.result()` 获取）

+   使用 `.cancel()` 取消（并使用 `.cancelled()` 检查取消）

+   添加额外的回调函数，在 future 完成时运行

尽管 `Task` 更常见，但无法完全避免 `Future`：例如，在执行器上运行函数将返回一个 `Future` 实例，*而不是* `Task`。让我们快速浏览一下 Example 3-16，感受直接使用 `Future` 实例的情况。

##### Example 3-16\. 与 `Future` 实例交互

```py
>>> import asyncio
>>>
>>> async def main(f: asyncio.Future):  ![1](img/1.png)
...     await asyncio.sleep(1)
...     f.set_result('I have finished.')  ![2](img/2.png)
...
>>> loop = asyncio.get_event_loop()
>>> fut = asyncio.Future()  ![3](img/3.png)
>>> print(fut.done())  ![4](img/4.png)
False
>>> loop.create_task(main(fut))  ![5](img/5.png)
<Task pending name='Task-1' coro=<main() running at <console>:1>>
>>> loop.run_until_complete(fut)  ![6](img/6.png)
'I have finished.'
>>> print(fut.done())
True
>>> print(fut.result())  ![7](img/7.png)
I have finished.
```

![1](img/#co_asyncio_walk_through_CO12-1)

创建一个简单的 `main` 函数。我们可以运行它，等待一会儿，然后在这个 `Future` `f` 上设置一个结果。

![2](img/#co_asyncio_walk_through_CO12-2)

设置结果。

![3](img/#co_asyncio_walk_through_CO12-3)

手动创建一个 `Future` 实例。请注意，此实例（默认情况下）与我们的 `loop` 绑定，但不会附加到任何协程上（这就是 `Task` 的用途）。

![4](img/#co_asyncio_walk_through_CO12-4)

在执行任何操作之前，请验证 future 是否尚未完成。

![5](img/#co_asyncio_walk_through_CO12-5)

*Schedule* `main()` 协程，传递 future。记住，`main()` 协程只是睡眠然后切换 `Future` 实例的状态。（注意，`main()` 协程还未开始运行：协程只在循环运行时运行。）

![6](img/#co_asyncio_walk_through_CO12-6)

这里我们在 `Future` 实例上使用 `run_until_complete()`，而不是 `Task` 实例。⁷ 这与您之前看到的不同。现在循环正在运行，`main()` 协程将开始执行。

![7](img/#co_asyncio_walk_through_CO12-7)

最终，当 future 的结果设置时，它会完成。完成后，可以访问结果。

当然，你不太可能直接按照这里所示的方式与`Future`直接交互；代码示例仅供教育目的。你与`asyncio`的大多数接触将通过`Task`实例完成。

你可能会想知道，如果在`Task`实例上调用`set_result()`会发生什么。在 Python 3.8 之前可以这样做，但现在不再允许。`Task`实例是协程对象的包装器，它们的结果值只能在底层协程函数的内部设置，如示例 3-17 中所示。

##### 示例 3-17。在 Task 上调用 set_result()

```py
>>> import asyncio
>>> from contextlib import suppress
>>>
>>> async def main(f: asyncio.Future):
...     await asyncio.sleep(1)
...     try:
...         f.set_result('I have finished.')  ![2](img/2.png)
...     except RuntimeError as e:
...         print(f'No longer allowed: {e}')
...         f.cancel()  ![3](img/3.png)
...
>>> loop = asyncio.get_event_loop()
>>> fut = asyncio.Task(asyncio.sleep(1_000_000))  ![1](img/1.png)
>>> print(fut.done())
False
>>> loop.create_task(main(fut))
<Task pending name='Task-2' coro=<main() running at <console>:1>>
>>> with suppress(asyncio.CancelledError):
...     loop.run_until_complete(fut)
...
No longer allowed: Task does not support set_result operation
>>> print(fut.done())
True
>>> print(fut.cancelled())  ![3](img/3.png)
True
```

![1](img/#co_asyncio_walk_through_CO13-3)

唯一的区别在于我们创建了一个`Task`实例而不是`Future`。当然，`Task` API 要求我们提供一个协程；我们只是使用`sleep()`因为它很方便。

![2](img/#co_asyncio_walk_through_CO13-1)

此处引用的一个臭名昭著的解释指出，正在传递一个`Task`实例。它满足函数的类型签名（因为`Task`是`Future`的子类），但自 Python 3.8 以来，我们不再允许在`Task`上调用`set_result()`：尝试这样做将引发`RuntimeError`。其思想是，`Task`代表正在运行的协程，因此结果应始终仅来自于那里。

![3](img/#co_asyncio_walk_through_CO13-2)

然而，我们仍然可以`cancel()`一个任务，这将在底层协程中引发`CancelledError`。

## 创建一个 Task？确保一个 Future？下定决心吧！

在“快速入门”中，我说过运行协程的方法是使用`asyncio.create_task()`。在引入该函数之前，需要获取一个`loop`实例，并使用`loop.create_task()`来完成相同的操作。实际上，还可以使用另一个模块级函数来实现这一点：`asyncio.ensure_future()`。一些开发者推荐使用`create_task()`，而另一些则推荐使用`ensure_future()`。

在撰写本书期间的研究中，我深信 API 方法`asyncio.ensure_future()`对于广泛误解`asyncio`库负有很大责任。大多数 API 确实非常清晰，但学习中存在一些障碍，这就是其中之一。当你遇到`ensure_future()`时，你的大脑会非常努力地将其整合到你对`asyncio`如何使用的心理模型中，并可能失败！

对`ensure_future()`的问题在[Python 3.6 `asyncio`文档](https://oreil.ly/fnjCs)中有详细说明。

`asyncio.ensure_future`(*`coro_or_future`*, *, _`loop=None`)

> 调度执行一个*协程对象*：将其包装在一个 future 中。返回一个*Task*对象。
> 
> 如果参数是一个*Future*，它将直接返回。

什么！？当我第一次读到这个时，它非常令人困惑。这里有一个（希望）更清晰的`ensure_future()`描述：

+   如果传入一个协程，它将产生一个`Task`实例（并且您的协程将被安排在事件循环上运行）。这与调用`asyncio.create_task()`（或`loop.create_task()`）并返回新的`Task`实例完全相同。

+   如果传入一个`Future`实例（或`Task`实例，因为`Task`是`Future`的子类），您将获得相同的东西，*没有改变*。是的，真的！

这个函数是`asyncio` API 的一个很好的例子，它展示了面向*终端用户开发者*（高级 API）和面向*框架设计者*（低级 API）之间的差异。让我们更详细地看看它是如何工作的，在示例 3-18 中。

##### 示例 3-18\. 详细了解 ensure_future()的操作

```py
import asyncio

async def f():  ![1](img/1.png)
    pass

coro = f()  ![2](img/2.png)
loop = asyncio.get_event_loop()  ![3](img/3.png)

task = loop.create_task(coro)  ![4](img/4.png)
assert isinstance(task, asyncio.Task)  ![5](img/5.png)

new_task = asyncio.ensure_future(coro)  ![6](img/6.png)
assert isinstance(new_task, asyncio.Task)

mystery_meat = asyncio.ensure_future(task)  ![7](img/7.png)
assert mystery_meat is task  ![8](img/8.png)
```

![1](img/#co_asyncio_walk_through_CO14-1)

一个简单的无操作协程函数。我们只需要一个能生成协程的东西。

![2](img/#co_asyncio_walk_through_CO14-2)

我们通过直接调用函数来创建协程对象。您的代码很少会这样做，但我想在这里明确指出（几行以下）我们将协程对象传递给`create_task()`和`ensure_future()`。

![3](img/#co_asyncio_walk_through_CO14-3)

获取循环。

![4](img/#co_asyncio_walk_through_CO14-4)

首先，我们使用`loop.create_task()`在循环上安排我们的协程，并获得一个新的`Task`实例。

![5](img/#co_asyncio_walk_through_CO14-5)

我们验证类型。到目前为止，没有什么有趣的。

![6](img/#co_asyncio_walk_through_CO14-6)

我们展示了`asyncio.ensure_future()`可以用来执行与`create_task()`相同的操作：我们传入了一个协程，然后得到了一个`Task`实例（并且协程已被安排在循环上运行）！如果您传入一个协程，`loop.create_task()`和`asyncio.ensure_future()`之间没有任何区别。

![7](img/#co_asyncio_walk_through_CO14-7)

但是，如果我们将一个`Task`实例传递给`ensure_future()`会发生什么？请注意，在步骤 4 中已经通过`loop.create_task()`创建了一个`Task`实例。

![8](img/#co_asyncio_walk_through_CO14-8)

我们确切地得到了*与传入相同*的`Task`实例：它没有任何变化地传递了下来。

直接传递`Future`实例有什么意义？为什么用同一个函数做两件不同的事情？答案是`ensure_future()`旨在被*框架作者*使用，为*终端用户开发者*提供可以处理两种类型参数的 API。不相信？这是来自前 BDFL 本人的解释：

> `ensure_future()`的用途在于，如果你有一个可能是协程或`Future`（后者包括`Task`，因为它是`Future`的子类）的东西，并且你想要能够调用一个仅在`Future`上定义的方法（可能唯一有用的例子是`cancel()`）。当它已经是`Future`（或`Task`）时，这不会做任何操作；当它是一个协程时，它会将其包装在一个`Task`中。
> 
> 如果你知道你有一个协程并且想要调度它，正确的 API 是`create_task()`。唯一需要调用`ensure_future()`的时候是在你提供一个 API（像大多数 asyncio 的 API 一样）接受协程或`Future`，并且你需要对它做一些需要你有一个`Future`的操作的时候。
> 
> Guido van Rossum，在[issue #477](https://oreil.ly/ydRpR)上[评论](https://oreil.ly/cSOFB)

总之，`asyncio.ensure_future()`是一个专为框架设计者设计的辅助函数。这最容易通过类似函数的类比来解释，所以让我们这样做。如果你有几年的编程经验，你可能见过类似于`listify()`函数的函数，在 Example 3-19 中有所提及。

##### Example 3-19\. 一个将输入强制转换为列表的实用函数

```py
def listify(x: Any) -> List:
    """ Try hard to convert x into a list """
    if isinstance(x, (str, bytes)):
        return [x]

    try:
        return [_ for _ in x]
    except TypeError:
        return [x]
```

这个函数尝试将参数无论如何转换为一个列表。这些类型的函数经常在 API 和框架中用于将输入强制转换为已知类型，这简化了后续的代码—在这种情况下，你知道参数（从`listify()`输出）将始终是一个列表。

如果我将`listify()`函数重命名为`ensure_list()`，那么你应该开始看到与`asyncio.ensure_future()`的类比：它总是试图强制将参数转换为`Future`（或其子类）类型。这是一个实用函数，旨在使*框架开发者*的生活更轻松，而不是像你和我这样的最终用户开发者。

实际上，`asyncio`标准库模块本身正是出于这个原因使用`ensure_future()`。当你下次查看 API 时，无论你在哪里看到一个被描述为“可等待对象”的函数参数，很可能内部正在使用`ensure_future()`来强制转换参数。例如，`asyncio.gather()`函数有以下签名：

```py
asyncio.gather(*`*``aws`*, *`loop``=``None`*, ...)

```

*`aws`*参数意味着“可等待对象”，其中包括协程、任务和 future。在内部，`gather()`使用`ensure_future()`进行类型强制转换：任务和 future 保持不变，而为协程创建任务。

这里的关键点是，作为最终用户应用开发者，你永远不应该需要使用`asyncio.ensure_future()`。这更多是框架设计者的工具。如果你需要在事件循环中安排一个协程，直接使用`asyncio.create_task()`即可。

在接下来的几节中，我们将回到语言级别的特性，从异步上下文管理器开始。

# 异步上下文管理器：async with

在上下文管理器中支持协程的能力非常方便。这是有道理的，因为许多情况需要在定义良好的范围内打开和关闭网络资源，比如连接。

理解`async with`的关键在于意识到上下文管理器的操作是通过*方法调用*驱动的，然后考虑：如果这些方法是协程函数会怎样？事实上，这正是它的工作原理，就像在示例 3-20 中展示的那样。

##### 示例 3-20\. 异步上下文管理器

```py
class Connection:
    def __init__(self, host, port):
        self.host = host
        self.port = port
    async def __aenter__(self):  ![1](img/1.png)
        self.conn = await get_conn(self.host, self.port)
        return conn
    async def __aexit__(self, exc_type, exc, tb):  ![2](img/2.png)
        await self.conn.close()

async with Connection('localhost', 9001) as conn:
    <do stuff with conn>
```

![1](img/#co_asyncio_walk_through_CO15-1)

对于同步上下文管理器，不再使用`__enter__()`特殊方法，而是使用新的`__aenter__()`特殊方法。这个特殊方法必须是`async def`方法。

![2](img/#co_asyncio_walk_through_CO15-2)

同样地，使用`__aexit__()`代替`__exit__()`。参数与`__exit__()`相同，并在上下文管理器主体中引发异常时被填充。

###### 注意

仅仅因为你在程序中使用了`asyncio`并不意味着你所有的上下文管理器都必须像这些那样是异步的。只有在*enter*和*exit*方法中需要`await`某些内容时才有用。如果没有阻塞 I/O 代码，只需使用常规的上下文管理器即可。

现在——就你我之间而言——我不太喜欢使用这种显式的上下文管理器风格，因为标准库的`contextlib`模块中存在着令人惊叹的`@contextmanager`装饰器。正如你所料，还有一个异步版本`@asynccontextmanager`，使得创建简单的异步上下文管理器变得更加容易。

## contextlib 的方式

这种方法类似于标准库中的`@contextmanager`装饰器。简而言之，示例 3-21 首先看了阻塞方式。

##### 示例 3-21\. 阻塞方式

```py
from contextlib import contextmanager

@contextmanager  ![1](img/1.png)
def web_page(url):
    data = download_webpage(url)  ![2](img/2.png)
    yield data
    update_stats(url)  ![3](img/3.png)

with web_page('google.com') as data:  ![4](img/4.png)
    process(data)  ![5](img/5.png)
```

![1](img/#co_asyncio_walk_through_CO16-1)

`@contextmanager`装饰器将生成器函数转换为上下文管理器。

![2](img/#co_asyncio_walk_through_CO16-2)

这个函数调用（我为这个示例编造的）看起来非常像需要使用网络接口的东西，这比“正常”的 CPU 密集型代码慢了几个数量级。这个上下文管理器*必须*在专用线程中使用；否则，整个程序将在等待数据时暂停。

![3](img/#co_asyncio_walk_through_CO16-3)

想象一下，每次从 URL 处理数据时我们更新一些统计信息，比如 URL 被下载的次数。从并发的角度来看，我们需要知道这个函数是否涉及内部 I/O，比如通过网络写入数据库。如果是这样，`update_stats()`也是一个阻塞调用。

![4](img/#co_asyncio_walk_through_CO16-4)

我们正在使用我们的上下文管理器。特别注意网络调用（`download_webpage()`）如何隐藏在上下文管理器的构建中。

![5](img/#co_asyncio_walk_through_CO16-5)

此函数调用，`process()`，也可能是阻塞的。我们需要查看函数的操作，因为阻塞或非阻塞的区别不太明显。它可能是：

+   无害且非阻塞（快速且 CPU 绑定）

+   轻微阻塞（快速且 I/O 绑定，可能是像快速磁盘访问这样的东西而不是网络 I/O）

+   阻塞的（慢且 I/O 绑定）

+   邪恶的（慢且 CPU 绑定）

为了简单起见，在这个例子中，假设调用`process()`是一个快速的、CPU 绑定的操作，因此是非阻塞的。

Example 3-22 正是相同的例子，但使用了在 Python 3.7 中引入的新的 async-aware helper。

##### 示例 3-22\. 非阻塞方式

```py
from contextlib import asynccontextmanager

@asynccontextmanager  ![1](img/1.png)
async def web_page(url):  ![2](img/2.png)
    data = await download_webpage(url)  ![3](img/3.png)
    yield data  ![4](img/4.png)
    await update_stats(url)  ![5](img/5.png)

async with web_page('google.com') as data:  ![6](img/6.png)
    process(data)
```

![1](img/#co_asyncio_walk_through_CO17-1)

新的`@asynccontextmanager`装饰器的使用方式完全相同。

![2](img/#co_asyncio_walk_through_CO17-2)

但是，它确实要求修饰的生成器函数使用`async def`声明。

![3](img/#co_asyncio_walk_through_CO17-3)

与以前一样，在将数据提供给上下文管理器的主体之前，我们从 URL 中获取数据。我添加了`await`关键字，它告诉我们，这个协程将允许事件循环在我们等待网络调用完成时运行其他任务。

请注意，我们*不能*简单地将`await`关键字附加到任何内容上。这一变化预设了我们还能够*修改*`download_webpage()`函数本身，并将其转换为与`await`关键字兼容的协程。对于无法修改函数的情况，需要采取不同的方法；我们将在下一个示例中讨论这一点。

![4](img/#co_asyncio_walk_through_CO17-4)

与以前一样，数据被提供给上下文管理器的主体。我试图保持代码简洁，因此省略了您通常应该编写的处理调用者主体中引发的异常的常规`try/finally`处理程序。

请注意，`yield`的存在改变了函数成为*生成器函数*；在第 1 点中的额外存在`async def`关键字使其成为*异步生成器函数*。当调用时，它将返回一个*异步生成器*。`inspect`模块有两个函数可以测试这些：`isasyncgenfunction()`和`isasyncgen()`。

![5](img/#co_asyncio_walk_through_CO17-5)

在这里，假设我们已经将`update_stats()`函数内的代码转换为允许其生成协程。然后我们可以使用`await`关键字，在等待 I/O 绑定工作完成时进行上下文切换到事件循环。

![6](img/#co_asyncio_walk_through_CO17-6)

在使用上下文管理器本身时，还需要进行另一个更改：我们需要使用`async with`而不是普通的`with`。

希望这个例子显示了新的`@asynccontextmanager`与`@contextmanager`装饰器非常类似。

在调用 3 和 5 中，我说需要修改一些函数以返回协程；这些函数是`download_webpage()`和`update_stats()`。通常情况下，这并不容易做到，因为需要在套接字级别添加异步支持。前面的示例的重点仅是展示新的`@asynccontextmanager`装饰器，而不是展示如何将阻塞函数转换为非阻塞函数。更常见的情况是，当您希望在程序中使用阻塞函数时，但无法修改该函数中的代码。

这种情况通常会发生在第三方库中，一个很好的例子是`requests`库，它在整个过程中使用阻塞调用。⁸ 如果无法更改被调用的代码，还有另一种方法。这是一个便利的地方，可以展示如何使用*执行器*来做到这一点，如示例 3-23 所示。

##### 示例 3-23\. 非阻塞的“有点帮助于我的朋友”方法

```py
from contextlib import asynccontextmanager

@asynccontextmanager
async def web_page(url):  ![1](img/1.png)
    loop = asyncio.get_event_loop()
    data = await loop.run_in_executor(
        None, download_webpage, url)  ![2](img/2.png)
    yield data
    await loop.run_in_executor(None, update_stats, url)  ![3](img/3.png)

async with web_page('google.com') as data:
    process(data)
```

![1](img/#co_asyncio_walk_through_CO18-1)

对于本例，假设我们*无法*修改我们两个阻塞调用`download_webpage()`和`update_stats()`的代码；即我们不能将它们修改为协程函数。这很糟糕，因为基于事件的编程最严重的罪行之一是违反了绝对不能阻止事件循环处理事件的规则。

为了解决问题，我们将使用一个*执行器*在单独的线程中运行阻塞调用。执行器作为事件循环本身的属性向我们提供。

![2](img/#co_asyncio_walk_through_CO18-2)

我们调用执行器。签名是`AbstractEventLoop.run_in_executor`（*`executor`*，*`func`*，*`*args`*）。如果要使用默认执行器（即`ThreadPoolExecutor`），必须将*`executor`*参数的值传递为`None`。⁹

![3](img/#co_asyncio_walk_through_CO18-3)

与调用`download_webpage()`一样，我们还在执行器中运行另一个阻塞调用`update_stats()`。请注意，你*必须*在前面使用`await`关键字。如果忘记了，异步生成器（即您的异步上下文管理器）在继续执行之前不会等待调用完成。

可能会在许多基于`asyncio`的代码库中广泛使用异步上下文管理器，因此对它们有一个很好的理解是非常重要的。您可以在[Python 3.7 文档](http://bit.ly/2FoWl9f)中阅读有关新的`@asynccontextmanager`装饰器的更多信息。

# 异步迭代器：async for

接下来是 `for` 循环的异步版本。如果您首先意识到普通迭代——就像许多其他语言特性一样——是通过使用*特殊方法*来实现的，这将最容易理解它是如何工作的。

作为参考，示例 3-24 展示了通过使用 `__iter__()` 和 `__next__()` 方法定义标准（非异步）迭代器的方式。

##### 示例 3-24\. 传统的非异步迭代器

```py
>>> class A:
...     def __iter__(self):   ![1](img/1.png)
...         self.x = 0  ![2](img/2.png)
...         return self  ![3](img/3.png)
...     def __next__(self):  ![4](img/4.png)
...         if self.x > 2:
...             raise StopIteration  ![5](img/5.png)
...         else:
...             self.x += 1
...             return self.x  ![6](img/6.png)
>>> for i in A():
...     print(i)
1
2
3
```

![1](img/#co_asyncio_walk_through_CO19-1)

*迭代器*必须实现 `__iter__()` 特殊方法。

![2](img/#co_asyncio_walk_through_CO19-2)

将一些状态初始化为“起始”状态。

![3](img/#co_asyncio_walk_through_CO19-3)

`__iter__()` 特殊方法必须返回一个*可迭代*对象；即，一个实现了 `__next__()` 特殊方法的对象。在这种情况下，它是同一个实例，因为 `A` 本身也实现了 `__next__()` 特殊方法。

![4](img/#co_asyncio_walk_through_CO19-4)

定义了 `__next__()` 方法。这将在迭代序列的每一步中调用，直到……

![5](img/#co_asyncio_walk_through_CO19-5)

…抛出 `StopIteration`。

![6](img/#co_asyncio_walk_through_CO19-6)

每次迭代的*返回值*都是生成的。

现在你可能会问：如果将 `__next__()` 特殊方法声明为 `async def` 协程函数会发生什么？那将允许它`await`某种 I/O 绑定操作——这几乎正是 `async for` 的工作方式，除了一些围绕命名的小细节。规范（在 PEP 492 中）显示，要在异步迭代器上使用 `async for`，异步迭代器本身需要满足几个条件：

1.  你必须实现 `def __aiter__()`。（注意：*不是* 使用 `async def`！）

1.  `__aiter__()` 必须返回一个实现了 `async def __anext__()` 的对象。

1.  `__anext__()` 必须为每次迭代返回一个值，并在完成时引发 `Stop​AsyncIteration`。

让我们快速看一下它可能如何工作。想象我们在一个 [Redis](https://redis.io/) 数据库中有一堆键，我们想迭代它们的数据，但我们仅在需要时获取数据。这样的异步迭代器可能看起来像 示例 3-25。

##### 示例 3-25\. 从 Redis 获取数据的异步迭代器

```py
import asyncio
from aioredis import create_redis

async def main():  ![1](img/1.png)
    redis = await create_redis(('localhost', 6379))  ![2](img/2.png)
    keys = ['Americas', 'Africa', 'Europe', 'Asia']  ![3](img/3.png)

    async for value in OneAtATime(redis, keys):  ![4](img/4.png)
        await do_something_with(value)  ![5](img/5.png)

class OneAtATime:
    def __init__(self, redis, keys):  ![6](img/6.png)
        self.redis = redis
        self.keys = keys
    def __aiter__(self):  ![7](img/7.png)
        self.ikeys = iter(self.keys)
        return self
    async def __anext__(self):  ![8](img/8.png)
        try:
            k = next(self.ikeys)  ![9](img/9.png)
        except StopIteration:  ![10](img/10.png)
            raise StopAsyncIteration

        value = await redis.get(k)  ![11](img/11.png)
        return value

asyncio.run(main())
```

![1](img/#co_asyncio_walk_through_CO20-1)

`main()` 函数：我们在代码示例底部使用 `asyncio.run()` 运行它。

![2](img/#co_asyncio_walk_through_CO20-2)

我们使用 `aioredis` 中的高级接口来获取连接。

![3](img/#co_asyncio_walk_through_CO20-3)

想象一下，与这些键关联的每个值都相当大，并存储在 Redis 实例中。

![4](img/#co_asyncio_walk_through_CO20-4)

我们正在使用 `async for`：关键是*迭代* *能够暂停自身*，同时等待下一个数据到达。

![5](img/#co_asyncio_walk_through_CO20-5)

为了完整起见，假设我们还对提取的值执行了一些 I/O 绑定活动——也许是一个简单的数据转换——然后将其发送到另一个目标。

![6](img/#co_asyncio_walk_through_CO20-6)

此类的初始化器非常普通：我们存储 Redis 连接实例和要迭代的键列表。

![7](img/#co_asyncio_walk_through_CO20-7)

就像前面的代码示例中使用`__iter__()`一样，我们使用`__aiter__()`来为迭代设置事物。我们在键`self.ikeys`上创建一个普通迭代器，并`return self`，因为`OneAtATime`还实现了`__anext__()`协程方法。

![8](img/#co_asyncio_walk_through_CO20-8)

请注意，`__anext__()`方法使用`async def`声明，而`__aiter__()`方法仅使用`def`声明。

![9](img/#co_asyncio_walk_through_CO20-9)

对于每个键，我们从 Redis 获取值：`self.ikeys`是键的常规迭代器，因此我们使用`next()`来移动它们。

![10](img/#co_asyncio_walk_through_CO20-10)

当`self.ikeys`耗尽时，我们处理`StopIteration`并简单地将其转换为`StopAsyncIteration`！这就是您如何在异步迭代器内部发出停止信号。

![11](img/#co_asyncio_walk_through_CO20-11)

最后——这个示例的整个重点——我们可以获取与此键关联的 Redis 数据。我们可以`await`数据，这意味着在等待网络 I/O 时可以运行其他代码。

希望这个示例很清楚：`async for`提供了在迭代本身执行 I/O 的数据时保留简单`for`循环便利性的能力。好处是您可以通过单个循环处理大量数据，因为您只需处理每个数据块的微小批次。

# 使用异步生成器简化代码

*异步生成器*是`async def`函数，其中包含`yield`关键字。异步生成器使代码更简单。

但是，如果您有使用生成器 *仿佛* 它们是协程的经验，比如使用 Twisted 框架、Tornado 框架或者 Python 3.4 的`asyncio`中的`yield from`，那么对它们的理解可能会有些困惑。因此，在继续之前，最好您能确信

+   协程和生成器是完全不同的概念。

+   异步生成器的行为与普通生成器非常相似。

+   对于迭代，您使用`async for`来处理异步生成器，而不是用于普通生成器的普通`for`。

在前一节中用于演示与 Redis 交互的异步迭代器的示例，如果我们将其设置为异步生成器，将会变得简单得多，如示例 3-26 所示。

##### 示例 3-26\. 使用异步生成器更容易

```py
import asyncio
from aioredis import create_redis

async def main():  ![1](img/1.png)
    redis = await create_redis(('localhost', 6379))
    keys = ['Americas', 'Africa', 'Europe', 'Asia']

    async for value in one_at_a_time(redis, keys):  ![2](img/2.png)
        await do_something_with(value)

async def one_at_a_time(redis, keys):  ![3](img/3.png)
    for k in keys:
        value = await redis.get(k)  ![4](img/4.png)
        yield value  ![5](img/5.png)

asyncio.run(main())
```

![1](img/#co_asyncio_walk_through_CO21-1)

`main()`函数与 Example 3-25 中的版本相同。

![2](img/#co_asyncio_walk_through_CO21-2)

嗯，几乎一样：我将名称从*`CamelCase`*改为*`snake_case`*。

![3](img/#co_asyncio_walk_through_CO21-3)

我们的函数现在声明为`async def`，使其成为*协程函数*，由于这个函数也包含`yield`关键字，我们将其称为*异步生成器函数*。

![4](img/#co_asyncio_walk_through_CO21-4)

我们不必像之前的示例中那样做繁琐的事情，比如使用`self.ikeys`：在这里，我们直接循环遍历键并获取值…

![5](img/#co_asyncio_walk_through_CO21-5)

…然后将其 yield 给调用者，就像一个普通的生成器一样。

如果这对你来说是新的，它可能看起来很复杂，但我建议你在几个玩具示例上自己试试。它很快就会感觉自然起来。异步生成器很可能会在基于`asyncio`的代码库中变得流行，因为它们带来了与普通生成器相同的所有好处：使代码更短，更简单。

# 异步推导式

现在我们已经看到 Python 如何支持异步迭代，下一个自然的问题是它是否也适用于列表推导式——答案是*是*。这种支持是在[PEP 530](https://oreil.ly/4qNoH)中引入的，我建议你自己阅读一下这个 PEP；它很简短，易读。Example 3-27 展示了典型异步推导式的布局。

##### 示例 3-27。异步列表、字典和集合推导式

```py
>>> import asyncio
>>>
>>> async def doubler(n):
...     for i in range(n):
...         yield i, i * 2  ![1](img/1.png)
...         await asyncio.sleep(0.1)  ![2](img/2.png)
...
>>> async def main():
...     result = [x async for x in doubler(3)]  ![3](img/3.png)
...     print(result)
...     result = {x: y async for x, y in doubler(3)}  ![4](img/4.png)
...     print(result)
...     result = {x async for x in doubler(3)}  ![5](img/5.png)
...     print(result)
...
>>> asyncio.run(main())
[(0, 0), (1, 2), (2, 4)]
{0: 0, 1: 2, 2: 4}
{(2, 4), (1, 2), (0, 0)}
```

![1](img/#co_asyncio_walk_through_CO22-1)

`doubler()`是一个非常简单的异步生成器：给定一个上限值，它将迭代一个简单的范围，产生值及其两倍的元组。

![2](img/#co_asyncio_walk_through_CO22-2)

睡一会儿，只是为了强调这确实是一个异步函数。

![3](img/#co_asyncio_walk_through_CO22-3)

异步列表推导式：请注意如何使用`async for`而不是通常的`for`。这个差异与“异步迭代器：async for”中的示例所示的相同。

![4](img/#co_asyncio_walk_through_CO22-4)

异步字典推导式；所有通常的技巧都有效，比如将元组解包成`x`和`y`，以便它们可以提供给字典推导式语法。

![5](img/#co_asyncio_walk_through_CO22-5)

异步集合推导式的工作方式与你期望的完全相同。

你也可以在推导式中使用`await`，如 PEP 530 所述。这不应该让人感到意外；`await``coro`是一个正常的表达式，可以在大多数你期望的地方使用。

是`async for`使理解成为*异步理解*，而不是`await`的存在。`await`要在理解内部合法（或者说，在协程函数体内使用）只需其在异步函数`async def`中使用即可。在同一列表理解中结合使用`await`和`async for`实际上是结合了两个独立的概念，但我们会在示例 3-28 中这样做，以确保你对异步语言语法感到舒适。

##### 示例 3-28\. 将所有内容整合在一起

```py
>>> import asyncio
>>>
>>> async def f(x):  ![1](img/1.png)
...   await asyncio.sleep(0.1)
...   return x + 100
...
>>> async def factory(n):  ![2](img/2.png)
...   for x in range(n):
...     await asyncio.sleep(0.1)
...     yield f, x  ![3](img/3.png)
...
>>> async def main():
...   results = [await f(x) async for f, x in factory(3)]  ![4](img/4.png)
...   print('results = ', results)
...
>>> asyncio.run(main())
results =  [100, 101, 102]
```

![1](img/#co_asyncio_walk_through_CO23-1)

一个简单的协程函数：暂停一会儿；然后返回参数加上 100。

![2](img/#co_asyncio_walk_through_CO23-2)

这是一个*异步生成器*，我们稍后将在异步列表理解中调用它，使用`async for`来驱动迭代。

![3](img/#co_asyncio_walk_through_CO23-3)

异步生成器将产生一个`f`和迭代变量`x`的元组。`f`的返回值是一个*协程函数*，而不是一个协程。

![4](img/#co_asyncio_walk_through_CO23-4)

最后是异步理解。这个示例被构造出来演示同时包含`async for`和`await`的理解。让我们分解一下理解内部发生了什么。首先，`factory(3)`调用返回一个异步生成器，必须通过迭代来驱动它。因为它是一个*异步*生成器，你不能简单地使用`for`；你必须使用`async for`。

异步生成器生成的值是一个由协程函数`f`和一个`int`组成的元组。调用协程函数`f()`会产生一个协程，必须使用`await`来评估它。

注意在理解中，使用`await`与使用`async for`毫不相关：它们完全执行不同的任务，并作用于完全不同的对象。

# 启动和关闭（优雅地！）

大多数基于异步的程序将会是长时间运行的、基于网络的应用程序。这个领域在正确处理启动和关闭的复杂性方面包含了惊人的多样性。

在这两者中，启动更为简单。启动`asyncio`应用程序的标准方式是拥有一个`main()`协程函数，并使用`asyncio.run()`调用它，就像在本章开头的示例 3-2 中展示的那样。

通常，启动过程会相对直接；对于前面描述的服务器情况，你可以在[文档](http://bit.ly/2FrKaIV)中进一步了解。接下来，我们还会简要看一下服务器启动的示例代码。

关闭过程要复杂得多。对于关闭，我之前讲过发生在`asyncio.run()`内部的流程。当`async def main()`函数退出时，将执行以下操作：

1.  收集所有仍未完成的任务对象（如果有）。

1.  取消这些任务（这会在每个正在运行的协程内部引发`CancelledError`，您可以选择在协程函数的`try/except`中处理它）。

1.  将所有这些任务聚合成一个*组*任务。

1.  在组任务上使用`run_until_complete()`等待它们完成——也就是说，让`CancelledError`异常被引发并处理。

`asyncio.run()`为您执行这些操作，但是尽管有这些帮助，构建您的前几个非平凡`asyncio`应用程序的过程中，您可能会试图消除诸如“任务已被销毁但仍处于挂起状态！”之类的错误消息。这是因为您的应用程序未预期某个或多个前面的步骤。示例 3-29 是一个引发此烦人错误的脚本的示例。

##### 示例 3-29\. 待处理任务的销毁者

```py
# taskwarning.py
import asyncio

async def f(delay):
    await asyncio.sleep(delay)

loop = asyncio.get_event_loop()
t1 = loop.create_task(f(1))  ![1](img/1.png)
t2 = loop.create_task(f(2))  ![2](img/2.png)
loop.run_until_complete(t1) ![3](img/3.png)
loop.close()
```

![1](img/#co_asyncio_walk_through_CO24-1)

任务 1 将运行 1 秒钟。

![2](img/#co_asyncio_walk_through_CO24-2)

任务 2 将运行 2 秒钟。

![3](img/#co_asyncio_walk_through_CO24-3)

仅运行直到任务 1 完成。

运行它会产生以下输出：

```py
$ python taskwarning.py
Task was destroyed but it is pending!
task: <Task pending coro=<f() done, defined at [...snip...]>

```

此错误告诉您，在关闭循环时，一些任务尚未完成。我们希望避免这种情况，这就是为什么习惯性的关闭过程是收集所有未完成的任务，取消它们，然后让它们全部完成*之后*再关闭循环。`asyncio.run()`为您执行所有这些步骤，但是重要的是详细了解这个过程，这样您就能处理更复杂的情况。

让我们看一个更详细的代码示例，展示所有这些阶段。示例 3-30 是一个基于 Telnet 的回显服务器的迷你案例研究。

##### 示例 3-30\. Asyncio 应用生命周期（基于 Python 文档中的 TCP 回显服务器）

```py
# telnetdemo.py
import asyncio
from asyncio import StreamReader, StreamWriter

async def echo(reader: StreamReader, writer: StreamWriter): ![1](img/1.png)
    print('New connection.')
    try:
        while data := await reader.readline():  ![2](img/2.png)
            writer.write(data.upper())  ![3](img/3.png)
            await writer.drain()
        print('Leaving Connection.')
    except asyncio.CancelledError:  ![4](img/4.png)
        print('Connection dropped!')

async def main(host='127.0.0.1', port=8888):
    server = await asyncio.start_server(echo, host, port) ![5](img/5.png)
    async with server:
        await server.serve_forever()

try:
    asyncio.run(main())
except KeyboardInterrupt:
    print('Bye!')
```

![1](img/#co_asyncio_walk_through_CO25-1)

此`echo()`协程函数将被（服务器）用于为每个建立的连接创建一个协程。该函数正在使用`asyncio`的流 API 进行网络操作。

![2](img/#co_asyncio_walk_through_CO25-2)

为了保持连接活跃，我们将使用一个无限循环来等待消息。

![3](img/#co_asyncio_walk_through_CO25-3)

将数据返回给发送者，但使用全部大写字母。

![4](img/#co_asyncio_walk_through_CO25-4)

如果此任务被*取消*，我们将打印一条消息。

![5](img/#co_asyncio_walk_through_CO25-5)

此 TCP 服务器启动代码直接来自于 Python 3.8 文档。

启动回显服务器后，可以使用 telnet 进行交互：

```py
$ telnet 127.0.0.1 8888
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi!
HI!
stop shouting
STOP SHOUTING
^]
telnet> q/
Connection closed.

```

该会话的服务器输出如下（服务器会一直运行，直到我们按下 Ctrl-C）：

```py
$ python telnetdemo.py
New connection.
Leaving Connection.
^CBye!

```

在刚才展示的 Telnet 会话中，客户端（即 Telnet）在服务器停止之前关闭了连接，但是让我们看看如果在连接活动时关闭服务器会发生什么。我们将从服务器进程看到以下输出：

```py
$ python telnetdemo.py
New connection.
^CConnection dropped!
Bye!

```

在这里，你可以看到 `CancelledError` 的异常处理程序被触发。 现在让我们假设这是一个真实的生产应用程序，并且我们希望将所有关于断开连接的事件发送到监控服务。 代码示例可能会修改为如下形式 示例 3-31。

##### 示例 3-31\. 在取消处理程序内创建任务

```py
# telnetdemo.py
import asyncio
from asyncio import StreamReader, StreamWriter

async def send_event(msg: str):  ![1](img/1.png)
    await asyncio.sleep(1)

async def echo(reader: StreamReader, writer: StreamWriter):
    print('New connection.')
    try:
        while (data := await reader.readline()):
            writer.write(data.upper())
            await writer.drain()
        print('Leaving Connection.')
    except asyncio.CancelledError:
        msg = 'Connection dropped!'
        print(msg)
        asyncio.create_task(send_event(msg))  ![2](img/2.png)

async def main(host='127.0.0.1', port=8888):
    server = await asyncio.start_server(echo, host, port)
    async with server:
        await server.serve_forever()

try:
    asyncio.run(main())
except KeyboardInterrupt:
    print('Bye!')
```

![1](img/#co_asyncio_walk_through_CO26-1)

假装这个协程实际上联系外部服务器以提交事件通知。

![2](img/#co_asyncio_walk_through_CO26-2)

因为事件通知器涉及网络访问，因此通常会在单独的异步任务中进行这类调用； 这就是我们在这里使用 `create_task()` 函数的原因。

然而，这段代码有一个 bug。 如果我们重新运行示例，并确保在连接活动时停止服务器（使用 Ctrl-C）：

```py
$ python telnetdemo.py
New connection.
^CConnection dropped!
Bye!
Task was destroyed but it is pending!
task: <Task pending name='Task-6' coro=<send_event() done, ...>

```

要理解为什么会发生这种情况，我们必须回到 `asyncio.run()` 在关闭阶段执行的清理事件序列； 特别是重要的部分是，当我们按下 Ctrl-C 时，所有当前活动的任务都被收集并取消。 此时，*仅这些任务* 然后被等待，而 `asyncio.run()` 在此之后立即返回。 我们修改后的代码中的 bug 是，在现有“echo”任务的取消处理程序内创建了一个*新*任务。 这个新任务是在 `asyncio.run()` 收集和取消了过程中所有任务之后才创建的。

这就是为什么了解 `asyncio.run()` 如何工作非常重要。

###### 提示

作为一个基本的经验法则，尽量避免在 `CancelledError` 异常处理程序内创建新任务。 如果必须这样做，请确保在同一函数范围内也要 `await` 新任务或 future。

最后：如果您正在使用库或框架，请确保遵循其关于如何执行启动和关闭的文档。 第三方框架通常提供其自己的启动和关闭功能，并提供事件钩子进行定制。 您可以在 Sanic 框架中查看这些钩子的示例 “案例研究：缓存失效”。

## `gather()` 中的 `return_exceptions=True` 是什么意思？

您可能已经注意到在关闭序列期间的 `gather()` 调用中的关键字参数 `return_exceptions=True`，但在当时我非常狡猾地没有提到它。 `asyncio.run()` 也在内部使用 `gather()` 和 `return_exceptions=True`，现在是进一步讨论的时候了。

不幸的是，默认设置为`gather(..., return_exceptions=False)`。 这个默认设置在大多数情况下都有问题，包括关闭过程，这就是为什么`asyncio.run()`将参数设置为`True`。 直接解释起来有点复杂； 相反，让我们逐步观察一系列观察，这将使理解变得更加容易：

1.  `run_until_complete()`作用于一个 future；在关闭期间，它是由`gather()`返回的 future。

1.  如果该 future 引发异常，该异常也将从`run_until_complete()`中抛出，这意味着循环将停止。

1.  如果`run_until_complete()`用于一个组合 future，任何子任务中引发的异常也会在“组”future 中引发，如果在子任务中没有处理的话。注意，这包括`CancelledError`。

1.  如果只有一些任务处理`CancelledError`而其他任务没有处理，那么没有处理的任务将导致循环停止。这意味着循环会在所有任务完成*之前*停止。

1.  对于关闭，我们确实不希望出现这种行为。我们希望`run_until_complete()`仅在组中的所有任务都完成时才结束，无论某些任务是否引发异常。

1.  因此我们有`gather(*, return_exceptions=True)`：该设置使得“组”future 将子任务中的异常视为*返回值*，因此它们不会冒出来干扰`run_until_complete()`。

就是这样：`return_exceptions=True`和`run_until_complete()`之间的关系。以这种方式捕获异常的一个不良后果是，一些错误可能会逃脱您的注意，因为它们现在（实际上）正在组任务内部处理。如果这是一个问题，您可以从`run_until_complete()`获取输出列表，并扫描其中任何`Exception`的子类，然后根据您的情况编写适当的日志消息。示例 3-32 演示了这种方法。

##### 示例 3-32\. 所有任务都将完成

```py
# alltaskscomplete.py
import asyncio

async def f(delay):
    await asyncio.sleep(1 / delay)  ![1](img/1.png)
    return delay

loop = asyncio.get_event_loop()
for i in range(10):
    loop.create_task(f(i))
pending = asyncio.all_tasks()
group = asyncio.gather(*pending, return_exceptions=True)
results = loop.run_until_complete(group)
print(f'Results: {results}')
loop.close()
```

![1](img/#co_asyncio_walk_through_CO27-1)

如果有人传入零的话，那就糟糕了...

这里是输出：

```py
$ python alltaskscomplete.py
Results: [6, 9, 3, 7, ...
          ZeroDivisionError('division by zero',), 4, ...
          8, 1, 5, 2]

```

没有`return_exceptions=True`，在`run_until_complete()`中会引发`ZeroDivisionError`，导致循环停止，从而阻止其他任务完成。

在下一节中，我们将讨论处理信号（除了 KeyboardInterrupt 之外），但在此之前，值得注意的是，优雅关闭是网络编程中更为困难的方面之一，对于`asyncio`也是如此。本节中的信息仅为一个起点。我建议您在自己的自动化测试套件中为干净的关闭设置具体的测试。不同的应用通常需要不同的策略。

###### 提示

我在 Python 包索引（PyPI）上发布了一个小包[`aiorun`](https://oreil.ly/kQDt8)，主要用于我在处理`asyncio`关闭时的实验和教育，其中包含了本节许多想法。也许对您来说，调试代码并在自己的`asyncio`关闭场景中尝试新想法也很有用。

## 信号

前面的示例展示了如何通过按下 Ctrl-C 停止事件循环。在`asyncio.run()`内部，引发的`KeyboardInterrupt`有效地解除了`loop.run_until_complete()`调用，并允许随后的关闭序列发生。

`KeyboardInterrupt`对应于`SIGINT`信号。在网络服务中，更常见的进程终止信号实际上是`SIGTERM`，这也是在 Unix shell 中使用`kill`命令时的默认信号。

###### 提示

Unix 系统上的`kill`命令具有迷惑性的名称：它只是向进程发送信号。如果没有参数，`kill` *`<PID>`*将发送一个`TERM`信号：您的进程可以接收该信号并以优雅的方式关闭或者简单地忽略它！但这是个坏主意，因为如果您的进程最终不停止，那么想要杀死进程的下一步通常是执行`kill -s KILL` *`<PID>`*，这将发送`KILL`信号。这会强制关闭您的程序，而您的程序无能为力。接收`TERM`（或`INT`）信号是您控制方式关闭的机会。

`asyncio`内置支持处理进程信号，但是信号处理通常存在一定的复杂性（与`asyncio`无关）。我们无法在这里覆盖所有内容，但我们可以看一些需要考虑的基本情况。示例 3-33 将产生以下输出：

```py
$ python shell_signal01.py
<Your app is running>
<Your app is running>
<Your app is running>
<Your app is running>
^CGot signal: SIGINT, shutting down.

```

我按下 Ctrl-C 来停止程序，如最后一行所示。示例 3-33 有意避免使用便捷的`asyncio.run()`函数，因为我想警告您在处理两个最常见的信号`SIGTERM`和`SIGINT`时要注意特定的陷阱。在讨论这些之后，我将展示一个使用更方便的`asyncio.run()`函数的最终示例。

##### 示例 3-33\. 使用`KeyboardInterrupt`作为 SIGINT 处理程序的复习

```py
# shell_signal01.py
import asyncio

async def main():  ![1](img/1.png)
    while True:
        print('<Your app is running>')
        await asyncio.sleep(1)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    task = loop.create_task(main())  ![2](img/2.png)
    try:
        loop.run_until_complete(task)
    except KeyboardInterrupt:  ![3](img/3.png)
        print('Got signal: SIGINT, shutting down.')
    tasks = asyncio.all_tasks(loop=loop)
    for t in tasks:
        t.cancel()
    group = asyncio.gather(*tasks, return_exceptions=True)
    loop.run_until_complete(group)
    loop.close()
```

![1](img/#co_asyncio_walk_through_CO28-1)

这是我们应用程序的主要部分。为了保持简单，我们将在一个无限循环中休眠。

![2](img/#co_asyncio_walk_through_CO28-2)

此启动和关闭序列将与您在前一节中熟悉的内容相同。我们安排`main()`，调用`run_forever()`，并等待某些事件停止循环。

![3](img/#co_asyncio_walk_through_CO28-3)

在这种情况下，只有 Ctrl-C 才能停止循环。然后我们处理`KeyboardInterrupt`并执行所有必要的清理工作，正如前几节所述。

到目前为止，这相当简单明了。现在我要复杂化事情了。假设：

+   您的一位同事要求您除了处理`SIGINT`信号外，还请处理`SIGTERM`信号作为关闭信号。

+   在您的实际应用程序中，您需要在 `main()` 协程内部进行清理；您将需要处理 `CancelledError`，并且异常处理程序内部的清理代码将需要几秒钟来完成（想象一下，您必须与网络对等方通信并关闭一堆套接字连接）。

+   如果您的应用程序接收到多次信号（例如重新运行任何关闭步骤），则不应该做奇怪的事情；在接收到第一个关闭信号后，您希望简单地忽略任何新信号直到退出。

`asyncio` 提供了足够的 API 粒度来处理所有这些情况。示例 3-34 修改了之前的简单代码示例，包含了这些新特性。

##### 示例 3-34\. 处理 SIGINT 和 SIGTERM，但只停止循环一次

```py
# shell_signal02.py
import asyncio
from signal import SIGINT, SIGTERM  ![1](img/1.png)

async def main():
    try:
        while True:
            print('<Your app is running>')
            await asyncio.sleep(1)
    except asyncio.CancelledError:  ![2](img/2.png)
        for i in range(3):
            print('<Your app is shutting down...>')
            await asyncio.sleep(1)

def handler(sig):  ![3](img/3.png)
    loop.stop()  ![4](img/4.png)
    print(f'Got signal: {sig!s}, shutting down.')
    loop.remove_signal_handler(SIGTERM)  ![5](img/5.png)
    loop.add_signal_handler(SIGINT, lambda: None)  ![6](img/6.png)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    for sig in (SIGTERM, SIGINT):  ![7](img/7.png)
        loop.add_signal_handler(sig, handler, sig)
    loop.create_task(main())
    loop.run_forever()  ![8](img/8.png)
    tasks = asyncio.all_tasks(loop=loop)
    for t in tasks:
        t.cancel()
    group = asyncio.gather(*tasks, return_exceptions=True)
    loop.run_until_complete(group)
    loop.close()
```

![1](img/#co_asyncio_walk_through_CO29-1)

从标准库 `signal` 模块导入信号值。

![2](img/#co_asyncio_walk_through_CO29-2)

这次，我们的 `main()` 协程将在内部进行一些清理。当接收到取消信号（通过取消每个任务来启动）时，将在关闭过程的 `run_until_complete()` 阶段继续运行 3 秒钟。它会打印，“Your app is shutting down…”。

![3](img/#co_asyncio_walk_through_CO29-3)

这是当我们接收信号时的回调处理程序。它通过稍后的 `add_signal_handler()` 调用配置在循环上。

![4](img/#co_asyncio_walk_through_CO29-4)

处理程序的主要目的是停止循环：这将解除 `loop.run_forever()` 调用的阻塞，并允许挂起任务的收集和取消，并且 `run_complete()` 用于关闭。

![5](img/#co_asyncio_walk_through_CO29-5)

由于我们现在处于关闭模式，我们*不希望*再次触发 `SIGINT` 或 `SIGTERM` 这个处理程序：在 `run_until_complete()` 阶段调用 `loop.stop()` 将干扰我们的关闭过程。因此，我们从循环中*移除*了 `SIGTERM` 的信号处理程序。

![6](img/#co_asyncio_walk_through_CO29-6)

这是一个“坑”：我们不能简单地删除 `SIGINT` 的处理程序，因为如果这样做，`KeyboardInterrupt` 将再次成为 `SIGINT` 的处理程序，就像我们在添加自己的处理程序之前一样。相反，我们将空的 `lambda` 函数设置为处理程序。这意味着 `KeyboardInterrupt` 保持远离，而 `SIGINT`（和 Ctrl-C）则没有效果。¹⁰

![7](img/#co_asyncio_walk_through_CO29-7)

这里将信号处理程序附加到循环中。请注意，正如之前讨论的那样，设置在 `SIGINT` 上的处理程序意味着 `KeyboardInterrupt` 将不再在 `SIGINT` 上引发。在 Python 中，引发 `KeyboardInterrupt` 是 `SIGINT` 的“默认”处理程序，直到您做某些更改来改变处理程序，正如我们在这里所做的。

![8](img/#co_asyncio_walk_through_CO29-8)

通常情况下，执行会在`run_forever()`上阻塞，直到某些东西停止循环。在这种情况下，如果将`SIGINT`或`SIGTERM`发送到我们的进程，循环将在`handler()`内部停止。其余代码与以前完全相同。

下面是输出：

```py
$ python shell_signal02.py
<Your app is running>
<Your app is running>
<Your app is running>
<Your app is running>
<Your app is running>
^CGot signal: Signals.SIGINT, shutting down.
<Your app is shutting down...>
^C<Your app is shutting down...>  ![1](img/1.png)
^C<Your app is shutting down...>

```

![1](img/#comarker1)

在关闭阶段，我按了很多次 Ctrl-C，但正如预期的那样，在`main()`协程最终完成之前什么也没发生。

在这些示例中，我以较困难的方式控制了事件循环的生命周期，但这是必要的，以解释关闭过程的各个组成部分。在实践中，我们更愿意使用更方便的`asyncio.run()`函数。示例 3-35 保留了前述信号处理设计的特性，同时还利用了`asyncio.run()`的便利性。

##### 示例 3-35\. 使用 asyncio.run()时的信号处理

```py
# shell_signal02b.py
import asyncio
from signal import SIGINT, SIGTERM

async def main():
    loop = asyncio.get_running_loop()
    for sig in (SIGTERM, SIGINT):
        loop.add_signal_handler(sig, handler, sig)  ![1](img/1.png)

    try:
        while True:
            print('<Your app is running>')
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        for i in range(3):
            print('<Your app is shutting down...>')
            await asyncio.sleep(1)

def handler(sig):
    loop = asyncio.get_running_loop()
    for task in asyncio.all_tasks(loop=loop):  ![2](img/2.png)
        task.cancel()
    print(f'Got signal: {sig!s}, shutting down.')
    loop.remove_signal_handler(SIGTERM)
    loop.add_signal_handler(SIGINT, lambda: None)

if __name__ == '__main__':
    asyncio.run(main())
```

![1](img/#co_asyncio_walk_through_CO30-1)

因为`asyncio.run()`控制了事件循环的启动，我们在`main()`函数中第一次更改信号处理行为的机会就来了。

![2](img/#co_asyncio_walk_through_CO30-2)

在信号处理程序中，我们不能像前面的示例那样停止循环，因为我们会收到有关在`main()`任务完成之前停止循环的警告。相反，我们可以在这里启动任务取消，这最终会导致`main()`任务退出；当这发生时，`asyncio.run()`内部的清理处理将接管。

## 等待执行者在关闭期间

“快速入门”介绍了基本的执行者接口，使用了示例 3-3，在那里我指出阻塞的`time.sleep()`调用比`asyncio.sleep()`调用方便，幸运的是对我们来说，这意味着执行者任务比`main()`协程更早完成，因此程序可以正确关闭。

本节将探讨在执行者作业需要较长时间完成而所有待处理的`Task`实例都完成时发生的关闭情况。简短的答案是：没有干预的话，你会得到类似于示例 3-36 代码产生的错误。

##### 示例 3-36\. 执行者需要太长时间才能完成

```py
# quickstart.py
import time
import asyncio

async def main():
    loop = asyncio.get_running_loop()
    loop.run_in_executor(None, blocking)
    print(f'{time.ctime()} Hello!')
    await asyncio.sleep(1.0)
    print(f'{time.ctime()} Goodbye!')

def blocking():
    time.sleep(1.5)  ![1](img/1.png)
    print(f"{time.ctime()} Hello from a thread!")

asyncio.run(main())
```

![1](img/#co_asyncio_walk_through_CO31-1)

此代码示例与示例 3-3 完全相同，*唯一*的区别是阻塞函数中的休眠时间现在比异步函数长。

运行此代码会产生以下输出：

```py
$ python quickstart.py
Fri Jan 24 16:25:08 2020 Hello!
Fri Jan 24 16:25:09 2020 Goodbye!
exception calling callback for <Future at [...snip...]>
Traceback (most recent call last):

<big nasty traceback>

RuntimeError: Event loop is closed
Fri Jan 24 16:25:09 2020 Hello from a thread!
```

这里发生的是，在幕后，`run_in_executor()`并*不*创建一个`Task`实例：它返回一个`Future`。这意味着它不包括在“活动任务”集合中，这些任务会在`asyncio.run()`内部被取消，因此`run_until_complete()`（在`asyncio.run()`内部调用）不会等待执行器任务完成。`RuntimeError`是由`asyncio.run()`内部的`loop.close()`调用引发的。

在编写时，Python 3.8 中的`loop.close()`不会等待所有执行器作业完成，这就是为什么`run_in_executor()`返回的`Future`抱怨的原因：在它解析完成时，循环已经关闭了。关于如何改进这一点，Python 核心开发团队正在讨论，但在找到解决方案之前，你需要一种处理这些错误的策略。

###### 提示

在 Python 3.9 中，`asyncio.run()`函数已经[得到改进](https://oreil.ly/ZrpRb)，正确等待执行器关闭，但在编写时，这个改进尚未回溯到 Python 3.8。

有几个修复这个问题的想法，每个都有不同的权衡，我们将看一些。我在这个练习中真正的目标是帮助你从不同的角度考虑事件循环的生命周期，考虑所有可能在非平凡程序中相互操作的协程、线程和子进程的生命周期管理。

第一个想法——也是最容易实现的，如示例 3-37 所示——是总是在协程内部`await`执行器任务。

##### 示例 3-37\. 选项 A：将执行器调用包装在协程内部

```py
# quickstart.py
import time
import asyncio
from concurrent.futures import ThreadPoolExecutor as Executor

async def main():
    loop = asyncio.get_running_loop()
    future = loop.run_in_executor(None, blocking)  ![1](img/1.png)
    try:
        print(f'{time.ctime()} Hello!')
        await asyncio.sleep(1.0)
        print(f'{time.ctime()} Goodbye!')
    finally:
        await future  ![2](img/2.png)

def blocking():
    time.sleep(2.0)
    print(f"{time.ctime()} Hello from a thread!")

try:
    asyncio.run(main())
except KeyboardInterrupt:
    print('Bye!')
```

![1](img/#co_asyncio_walk_through_CO32-1)

这个想法旨在解决`run_in_executor()`仅返回`Future`实例而不是任务的缺陷。我们无法在`all_tasks()`中捕获这个作业（在`asyncio.run()`内使用），但我们可以在 future 上使用`await`。计划的第一部分是在`main()`函数内创建一个 future。

![2](img/#co_asyncio_walk_through_CO32-2)

我们可以使用`try/finally`结构确保在`main()`函数返回之前等待 future 完成。

代码能够正常运行，但它对执行器函数的生命周期管理施加了严重限制：这意味着你必须在每一个创建执行器作业的作用域内使用`try/finally`。我们更希望像创建异步任务一样生成执行器作业，并且仍然可以在`asyncio.run()`内部进行优雅的退出处理。

下一个想法，如示例 3-38 所示，更加狡猾一些。由于我们的问题在于执行器创建了一个 future 而不是任务，并且`asyncio.run()`内部的关闭处理处理的是任务，我们的下一个计划是将执行器产生的 future 包装在一个新的任务对象中。

##### 示例 3-38\. 选项 B：将执行器 future 添加到收集的任务中

```py
# quickstart.py
import time
import asyncio
from concurrent.futures import ThreadPoolExecutor as Executor

async def make_coro(future):  ![2](img/2.png)
    try:
        return await future
    except asyncio.CancelledError:
        return await future

async def main():
    loop = asyncio.get_running_loop()
    future = loop.run_in_executor(None, blocking)
    asyncio.create_task(make_coro(future))  ![1](img/1.png)
    print(f'{time.ctime()} Hello!')
    await asyncio.sleep(1.0)
    print(f'{time.ctime()} Goodbye!')

def blocking():
    time.sleep(2.0)
    print(f"{time.ctime()} Hello from a thread!")

try:
    asyncio.run(main())
except KeyboardInterrupt:
    print('Bye!')
```

![1](img/#co_asyncio_walk_through_CO33-2)

我们获取从`run_in_executor()`调用返回的未来，并将其传递给一个新的实用函数`make_coro()`。这里的重要点在于我们使用了`create_task()`，这意味着该任务将出现在`asyncio.run()`的关闭处理中的`all_tasks()`列表中，并且在关闭过程中将收到取消。

![2](img/#co_asyncio_walk_through_CO33-1)

这个实用函数`make_coro()`简单地等待未来完成——但关键是，它*继续等待*未来，即使在处理`CancelledError`的异常处理程序中也是如此。

这种解决方案在关闭过程中表现更加良好，我鼓励您运行示例并在打印“Hello！”后立即按 Ctrl-C。关闭过程仍将等待`make_coro()`退出，这意味着它也等待我们的执行器作业退出。然而，这段代码非常笨拙，因为您必须在`make_coro()`调用中将每个执行器`Future`实例包装起来。

如果我们愿意放弃`asyncio.run()`函数的便利性（直到 Python 3.9 可用），我们可以通过自定义循环处理做得更好，如示例 3-39 所示。

##### 示例 3-39\. 选项 C：就像露营一样，带上自己的循环和执行器

```py
# quickstart.py
import time
import asyncio
from concurrent.futures import ThreadPoolExecutor as Executor

async def main():
    print(f'{time.ctime()} Hello!')
    await asyncio.sleep(1.0)
    print(f'{time.ctime()} Goodbye!')
    loop.stop()

def blocking():
    time.sleep(2.0)
    print(f"{time.ctime()} Hello from a thread!")

loop = asyncio.get_event_loop()
executor = Executor()  ![1](img/1.png)
loop.set_default_executor(executor)  ![2](img/2.png)
loop.create_task(main())
future = loop.run_in_executor(None, blocking)  ![3](img/3.png)
try:
    loop.run_forever()
except KeyboardInterrupt:
    print('Cancelled')
tasks = asyncio.all_tasks(loop=loop)
for t in tasks:
    t.cancel()
group = asyncio.gather(*tasks, return_exceptions=True)
loop.run_until_complete(group)
executor.shutdown(wait=True)  ![4](img/4.png)
loop.close()
```

![1](img/#co_asyncio_walk_through_CO34-1)

这一次，我们创建了我们自己的执行器实例。

![2](img/#co_asyncio_walk_through_CO34-2)

我们必须将我们的自定义执行器设置为循环的默认执行器。这意味着无论代码在何处调用`run_in_executor()`，都将使用我们的自定义实例。

![3](img/#co_asyncio_walk_through_CO34-3)

就像以前一样，我们运行阻塞函数。

![4](img/#co_asyncio_walk_through_CO34-4)

最后，在关闭循环之前，我们可以显式等待所有执行器作业完成。这样可以避免之前看到的“事件循环已关闭”消息。我们之所以能够这样做，是因为我们可以访问执行器对象；默认执行器在`asyncio` API 中未公开，这就是为什么我们无法对其调用`shutdown()`并被迫创建自己的执行器实例的原因。

最后，我们有一种普遍适用的策略：无论何时调用`run_in_executor()`，您的程序仍将进行干净关闭，即使在所有异步任务完成后仍有执行器作业在运行。

我强烈建议您尝试这里显示的代码示例，并尝试使用不同的策略创建任务和执行器作业，分时执行并尝试清洁关闭。我期待 Python 的未来版本将允许`asyncio.run()`函数内部等待执行器作业完成，但我希望本节中的讨论仍对您开发围绕清洁关闭处理的思路有所帮助。

¹ 当它们可用时！撰写本文时，Asyncio 的唯一可用参考资料是官方 Python 文档中的 API 规范和一系列博客文章，其中本书中链接了几篇。

² `asyncio` API 允许你用多个循环实例和线程做很多疯狂的事情，但这不是一个讨论这些内容的合适的书籍。99%的情况下，你只会为你的应用程序使用一个单一的主线程，就像这里展示的那样。

³ 在 API 文档中，使用参数名*`coro`*是一种常见的约定。它指的是一个*协程*；也就是说，严格来说，是调用`async def`函数的*结果*，而*不是*函数本身。

⁴ 不幸的是，`run_in_executor()`的第一个参数是要使用的`Executor`实例，而你*必须*传递`None`才能使用默认值。每次我使用这个时，感觉“executor”参数都在呼唤着要成为一个带有默认值为`None`的关键字参数。

⁵ 而且，其他开源库（如 Twisted 和 Tornado）过去是这样暴露异步支持的。

⁶ 还可以接受的是基于生成器的旧版协程，它是一个被`@types.coroutine`装饰并在内部使用`yield from`关键字挂起的生成器函数。在本书中，我们将完全忽略旧版协程。请把它们从你的脑海中抹去！

⁷ 这里的文档不一致：签名给出的是 `AbstractEventLoop.run_until_complete`(*`future`*)，但实际上应该是 `AbstractEventLoop.run_until_complete`(*`coro_or_future`*)，因为相同的规则适用。

⁸ 在事后给现有框架添加异步支持可能会非常困难，因为可能需要对代码库进行大规模的结构性更改。这在一个[用于 `requests` 的 GitHub 问题](https://oreil.ly/we5cZ)中有所讨论。

⁹ 是的，这真的很烦人。每次我使用这个调用时，我都不禁想知道为什么更常见的习惯用法，即将`executor=None`作为关键字参数，不被优先选择。

¹⁰ `add_signal_handler()`可能应该被命名为`set_signal_handler()`，因为你每个信号类型只能有一个处理程序；对于同一个信号再次调用`add_signal_handler()`将替换该信号的现有处理程序。
