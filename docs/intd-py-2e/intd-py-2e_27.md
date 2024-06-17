# 附录 C. 完全不同的东西：异步

我们的前两个附录是为初学者编写的，但这个是为那些有些进阶的人准备的。

和大多数编程语言一样，Python 一直以来都是*同步*的。它按顺序线性地运行代码，逐行从头到尾。当你调用一个函数时，Python 跳到其代码中，并且调用者会等待函数返回后才能恢复其原来的工作。

你的 CPU 一次只能做一件事情，因此同步执行是完全合理的。但事实证明，一个程序实际上并不总是在运行代码，而是在等待某些东西，比如来自文件或网络服务的数据。这就像我们盯着浏览器屏幕等待网站加载一样。如果我们能避免这种“忙等待”，我们可能会缩短程序的总运行时间。这也被称为提高*吞吐量*。

在第十五章中，你看到如果需要并发处理，你可以选择线程、进程或者像`gevent`或`twisted`这样的第三方解决方案。但现在有越来越多的*异步*答案，既内置于 Python 中，也有第三方解决方案。这些与通常的同步 Python 代码并存，但借用一句《捉鬼敢死队》中的警告，不能混用。我会向你展示如何避免任何幽灵般的副作用。

# 协程和事件循环

在 Python 3.4 中，Python 加入了一个标准的*异步*模块，名为`asyncio`。Python 3.5 随后添加了关键字`async`和`await`。这些实现了一些新概念：

+   *协程*是在各种点暂停的函数

+   一个*事件循环*，用于调度和运行协程

这些让我们编写看起来像我们习惯的普通同步代码的异步代码。否则，我们需要使用第十五章和第十七章中提到的方法，并在后面总结为“异步与…”。

正常的多任务处理是你的操作系统对你的进程所做的事情。它决定什么是公平的，谁是 CPU 食量巨大者，何时打开 I/O 水龙头等等。然而，事件循环提供了*协作式多任务处理*，其中协程指示何时能够开始和停止。它们在单个线程中运行，因此你不会遇到我在“线程”中提到的潜在问题。

通过在初始的`def`前加上`async`来*定义*一个协程。你通过以下方式*调用*一个协程：

+   将`await`放在前面，悄悄地将协程添加到现有的事件循环中。你只能在另一个协程内部这样做。

+   或者通过`asyncio.run()`来使用，它显式地启动一个事件循环。

+   或者使用`asyncio.create_task()`或`asyncio.ensure_future()`。

这个例子使用了前两种调用方法：

```py
>>> import asyncio
>>>
>>> async def wicked():
...     print("Surrender,")
...     await asyncio.sleep(2)
...     print("Dorothy!")
...
>>> asyncio.run(wicked())
Surrender,
Dorothy!
```

这里有一个戏剧性的两秒等待，在打印页上你看不到。为了证明我们没有作弊（详见第十九章中的`timeit`详细信息）：

```py
>>> from timeit import timeit
>>> timeit("asyncio.run(wicked())", globals=globals(), number=1)
Surrender,
Dorothy!
2.005701574998966
```

`asyncio.sleep(2)` 的调用本身就是一个协程，在这里只是一个示例，模拟像 API 调用这样的耗时操作。

`asyncio.run(wicked())` 这一行是从同步 Python 代码中运行协程的一种方式（这里是程序的顶层）。

与标准同步对应的区别（使用`time.sleep()`）是，在运行`wicked()`时，调用者不会被阻塞两秒钟。

运行协程的第三种方式是创建一个*任务*并`await`它。此示例展示了任务方法以及前两种方法：

```py
>>> import asyncio
>>>
>>> async def say(phrase, seconds):
...     print(phrase)
...     await asyncio.sleep(seconds)
...
>>> async def wicked():
...     task_1 = asyncio.create_task(say("Surrender,", 2))
...     task_2 = asyncio.create_task(say("Dorothy!", 0))
...     await task_1
...     await task_2
...
>>> asyncio.run(wicked())
Surrender,
Dorothy!
```

如果你运行这个代码，你会发现这次两行打印之间没有延迟。这是因为它们是独立的任务。`task_1`在打印`Surrender`后暂停了两秒，但这并不影响`task_2`。

`await`类似于生成器中的`yield`，但是它不返回值，而是标记一个在需要时事件循环可以暂停它的位置。

在[文档](https://oreil.ly/Cf_hd)中可以找到更多相关信息。同步和异步代码可以在同一个程序中共存。只需记住在函数定义前加上`async`，在调用异步函数前加上`await`即可。

一些更多信息：

+   一个[`asyncio`链接列表](https://oreil.ly/Vj0yD)。

+   用于`asyncio`的[网络爬虫](https://oreil.ly/n4FVx)的代码。

# Asyncio 替代方案

虽然`asyncio`是标准的 Python 包，但是你可以在没有它的情况下使用`async`和`await`。协程和事件循环是独立的。`asyncio`的设计有时会受到[批评](https://oreil.ly/n4FVx)，并且出现了第三方替代品：

+   [`curio`](https://curio.readthedocs.io)

+   [`trio`](https://trio.readthedocs.io)

让我们展示一个使用`trio`和[`asks`](https://asks.readthedocs.io)(一个类似于`requests`API 的异步 Web 框架)的真实例子。示例 C-1 展示了一个使用`trio`和`asks`进行并发 Web 爬虫的例子，这个例子改编自 stackoverflow 的一个[答案](https://oreil.ly/CbINS)。要运行这个例子，首先要安装`trio`和`asks`。

##### 示例 C-1\. trio_asks_sites.py

```py
import time

import asks
import trio

asks.init("trio")

urls = [
    'https://boredomtherapy.com/bad-taxidermy/',
    'http://www.badtaxidermy.com/',
    'https://crappytaxidermy.com/',
    'https://www.ranker.com/list/bad-taxidermy-pictures/ashley-reign',
]

async def get_one(url, t1):
    r = await asks.get(url)
    t2 = time.time()
    print(f"{(t2-t1):.04}\t{len(r.content)}\t{url}")

async def get_sites(sites):
    t1 = time.time()
    async with trio.open_nursery() as nursery:
        for url in sites:
            nursery.start_soon(get_one, url, t1)

if __name__ == "__main__":
    print("seconds\tbytes\turl")
    trio.run(get_sites, urls)
```

下面是我得到的：

```py
$ python trio_asks_sites.py
seconds bytes   url
0.1287  5735    https://boredomtherapy.com/bad-taxidermy/
0.2134  146082  https://www.ranker.com/list/bad-taxidermy-pictures/ashley-reign
0.215   11029   http://www.badtaxidermy.com/
0.3813  52385   https://crappytaxidermy.com/
```

注意到`trio`没有使用`asyncio.run()`，而是使用了自己的`trio.open_nursery()`。如果你感兴趣，可以阅读关于`trio`背后设计决策的[文章](https://oreil.ly/yp1-r)和[讨论](https://oreil.ly/P21Ra)。

一个名为[`AnyIO`](https://anyio.readthedocs.io/en/latest)的新包提供了对`asyncio`、`curio`和`trio`的单一接口。

未来，你可以期待更多的异步方法，无论是在标准 Python 中还是来自第三方开发者。

# 异步对比…

如你在本书中许多地方看到的那样，有许多并发技术。异步处理与它们相比如何？

进程

如果你想要使用机器上的所有 CPU 核心或多台机器，这是一个很好的解决方案。但是进程比较重，启动时间长，并且需要序列化进行进程间通信。

线程

虽然线程被设计为进程的“轻量级”替代方案，但每个线程使用大量内存。协程比线程轻得多；在仅支持几千个线程的机器上，可以创建数十万个协程。

绿色线程

绿色线程像`gevent`一样表现良好，并且看起来像是同步代码，但是需要*猴子补丁*标准的 Python 函数，例如套接字库。

回调

像`twisted`这样的库依赖于*回调*：当某些事件发生时调用的函数。这对 GUI 和 JavaScript 程序员来说很熟悉。

队列—当你的数据或进程确实需要多台机器时，这些通常是一个大规模的解决方案。

# 异步框架和服务器

Python 的异步新增功能是最近才有的，开发者们需要时间来创建像 Flask 这样的异步框架版本。

[ASGI](https://asgi.readthedocs.io)标准是 WSGI 的异步版本，详细讨论请参见[这里](https://oreil.ly/BnEXT)。

这里有一些 ASGI Web 服务器：

+   [`hypercorn`](https://pgjones.gitlab.io/hypercorn)

+   [`sanic`](https://sanic.readthedocs.io)

+   [`uvicorn`](https://www.uvicorn.org)

还有一些异步 Web 框架：

+   [`aiohttp`](https://aiohttp.readthedocs.io)—客户端 *和* 服务器

+   [`api_hour`](https://pythonhosted.org/api_hour)

+   [`asks`](https://asks.readthedocs.io)—类似于`requests`

+   [`blacksheep`](https://github.com/RobertoPrevato/BlackSheep)

+   [`bocadillo`](https://github.com/bocadilloproject/bocadillo)

+   [`channels`](https://channels.readthedocs.io)

+   [`fastapi`](https://fastapi.tiangolo.com)—使用类型注解

+   [`muffin`](https://muffin.readthedocs.io)

+   [`quart`](https://gitlab.com/pgjones/quart)

+   [`responder`](https://python-responder.org)

+   [`sanic`](https://sanic.readthedocs.io)

+   [`starlette`](https://www.starlette.io)

+   [`tornado`](https://www.tornadoweb.org)

+   [`vibora`](https://vibora.io)

最后，一些异步数据库接口：

+   [`aiomysql`](https://aiomysql.readthedocs.io)

+   [`aioredis`](https://aioredis.readthedocs.io)

+   [`asyncpg`](https://github.com/magicstack/asyncpg)
