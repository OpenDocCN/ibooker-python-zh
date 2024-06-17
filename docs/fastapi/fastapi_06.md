# 第四章。异步、并发和 Starlette 之旅

> Starlette 是一个轻量级的 ASGI 框架/工具包，非常适合在 Python 中构建异步 Web 服务。
> 
> Starlette 的创建者 Tom Christie

# 预览

上一章简要介绍了开发者在编写新 FastAPI 应用时会遇到的第一件事情。本章强调了 FastAPI 的底层 Starlette 库，特别是其对*async*处理的支持。在概述 Python 中“同时执行多个任务”的多种方式之后，您将看到其最新的`async`和`await`关键字如何被整合到 Starlette 和 FastAPI 中。

# Starlette

FastAPI 的许多 Web 代码基于[Tom Christie 创建的 Starlette 包](https://www.starlette.io)。它可以作为自己的 Web 框架使用，也可以作为其他框架（如 FastAPI）的库使用。与任何其他 Web 框架一样，Starlette 处理所有常规的 HTTP 请求解析和响应生成。它类似于[Flask 底层的 Werkzeug](https://werkzeug.palletsprojects.com)。

但是它最重要的特性是支持现代 Python 异步 Web 标准：[ASGI](https://asgi.readthedocs.io)。直到现在，大多数 Python Web 框架（如 Flask 和 Django）都是基于传统的同步[WSGI 标准](https://wsgi.readthedocs.io)。因为 Web 应用程序经常连接到较慢的代码（例如数据库、文件和网络访问），ASGI 避免了基于 WSGI 的应用程序的阻塞和忙等待。

因此，使用 Starlette 及其相关框架的 Python Web 包是最快的，甚至与 Go 和 Node.js 应用程序一较高下。

# 并行计算类型

在深入了解 Starlette 和 FastAPI 提供的*async*支持的详细信息之前，了解一下我们可以实现*concurrency*的多种方式是很有用的。

在*parallel*计算中，任务同时分布在多个专用 CPU 上。这在像图形和机器学习这样的“数值计算”应用程序中很常见。

在*concurrent*计算中，每个 CPU 在多个任务之间切换。某些任务比其他任务花费的时间更长，我们希望减少所需的总时间。读取文件或访问远程网络服务比在 CPU 中运行计算慢得多，字面上慢了成千上百万倍。

Web 应用程序完成了大量缓慢的工作。我们如何使 Web 服务器或任何服务器运行得更快？本节讨论了一些可能性，从系统范围内到本章重点：FastAPI 对 Python 的`async`和`await`的实现。

## 分布和并行计算

如果您有一个真正大型的应用程序——一个在单个 CPU 上会显得吃力的应用程序——您可以将其分解成片段，并使这些片段在单个机器的多个 CPU 上运行或在多台机器上运行。您可以以许多种方式做到这一点，如果您拥有这样的应用程序，您已经了解了其中的一些方式。管理所有这些片段比管理单个服务器更复杂和昂贵。

本书关注的是可以放在单个盒子上的小到中型应用程序。这些应用程序可以有同步和异步代码的混合，由 FastAPI 很好地管理。

## 操作系统进程

操作系统（或*OS*，因为打字疼）调度资源：内存、CPU、设备、网络等等。它运行的每个程序都在一个或多个*进程*中执行其代码。操作系统为每个进程提供受管理的、受保护的资源访问，包括它们何时可以使用 CPU。

大多数系统使用*抢占式*进程调度，不允许任何进程独占 CPU、内存或任何其他资源。操作系统根据其设计和设置不断挂起和恢复进程。

对于开发者来说，好消息是：这不是你的问题！但通常伴随着好消息而来的坏消息是：即使你想改变，你也不能做太多事情。

对于 CPU 密集型 Python 应用程序，通常的解决方案是使用多个进程并让操作系统管理它们。Python 有一个[multiprocessing module](https://oreil.ly/YO4YE)用于此目的。

## 操作系统线程

您也可以在单个进程内运行*控制线程*。Python 的[threading package](https://oreil.ly/xwVB1)管理这些线程。

当程序受 I/O 限制时，通常建议使用线程，而当程序受 CPU 限制时，建议使用多个进程。但线程编程很棘手，可能导致难以找到的错误。在*介绍 Python*中，我把线程比作在闹鬼的房子中飘荡的幽灵：独立而看不见，只能通过它们的效果来检测。嘿，谁移动了那个烛台？

传统上，Python 保持基于进程和基于线程的库分开。开发者必须学习其中的奥秘细节才能使用它们。一个更近期的包叫做[concurrent.futures](https://oreil.ly/dT150)，它是一个更高级别的接口，使它们更易于使用。

如您将看到的，通过新的 async 函数，您可以更轻松地获得线程的好处。FastAPI 还通过线程池管理普通同步函数（`def`而不是`async def`）的线程。

## 绿色线程

更神秘的机制由*绿色线程*，例如[greenlet](https://greenlet.readthedocs.io)，[gevent](http://www.gevent.org)和[Eventlet](https://eventlet.net)呈现出来。这些线程是*协作式*的（而不是抢占式的）。它们类似于操作系统线程，但在用户空间（即您的程序）而不是操作系统内核中运行。它们通过*猴子补丁*标准 Python 函数（在运行时修改标准 Python 函数）使并发代码看起来像正常的顺序代码：当它们阻塞等待 I/O 时，它们放弃控制权。

操作系统线程比操作系统进程“轻”（使用更少内存），而绿色线程比操作系统线程更轻。在一些[基准测试](https://oreil.ly/1NFYb)中，所有异步方法通常比它们的同步对应方法更快。

###### 注意

在你读完本章之后，你可能会想知道哪个更好：gevent 还是 asyncio？我认为并没有一个适用于所有情况的单一偏好。绿色线程早在之前就已实现（使用了来自多人在线游戏*Eve Online*的思想）。本书采用了 Python 的标准 asyncio，这与线程相比更简单，性能也更好。

## 回调

互动应用程序的开发者，如游戏和图形用户界面的开发者，可能已经熟悉*回调*。您编写函数并将其与事件（如鼠标点击、按键或时间）关联起来。这个类别中最显著的 Python 包是[Twisted](https://twisted.org)。它的名字反映了基于回调的程序有点“内外颠倒”和难以理解的现实。

## Python 生成器

像大多数语言一样，Python 通常是顺序执行代码。当您调用一个函数时，Python 会从它的第一行运行到结束或`return`。

但是在 Python 的*生成器函数*中，您可以在任何点停止并返回，并在以后返回到该点。这个技巧就是`yield`关键字。

在一集*Simpsons*中，Homer 将他的车撞到了一尊鹿的雕像上，接着有三行对话。 示例 4-1 定义了一个普通的 Python 函数来`return`这些行作为列表，并让调用者对其进行迭代。

##### 示例 4-1\. 使用 `return`

```py
>>> def doh():
...     return ["Homer: D'oh!", "Marge: A deer!", "Lisa: A female deer!"]
...
>>> for line in doh():
...     print(line)
...
Homer: D'oh!
Marge: A deer!
Lisa: A female deer!
```

当列表相对较小时，这种方法非常有效。但是如果我们要获取所有*Simpsons*剧集的对话呢？列表会占用内存。

示例 4-2 展示了一个生成器函数如何分配这些行。

##### 示例 4-2\. 使用 `yield`

```py
>>> def doh2():
...     yield "Homer: D'oh!"
...     yield "Marge: A deer!"
...     yield "Lisa: A female deer!"
...
>>> for line in doh2():
...     print(line)
...
Homer: D'oh!
Marge: A deer!
Lisa: A female deer!
```

我们不是在普通函数`doh()`返回的列表上进行迭代，而是在生成器函数`doh2()`返回的*生成器对象*上进行迭代。实际的迭代（`for...in`）看起来一样。Python 从`doh2()`返回第一个字符串，但会追踪它的位置以便于下一次迭代，直到函数耗尽对话。

任何包含`yield`的函数都是生成器函数。鉴于这种能力可以返回到函数中间并恢复执行，下一节看起来像是一个合理的适应。

## Python 的 async、await 和 asyncio

Python 的[asyncio](https://oreil.ly/cBMAc)特性已经在不同的版本中引入。您至少要运行 Python 3.7，这是`async`和`await`成为保留关键字的版本。

以下示例展示了一个只有在异步运行时才会有趣的笑话。自己运行这两个示例，因为时机很重要。

首先，运行不好笑的 示例 4-3。

##### 示例 4-3\. 枯燥

```py
>>> import time
>>>
>>> def q():
...     print("Why can't programmers tell jokes?")
...     time.sleep(3)
...
>>> def a():
...     print("Timing!")
...
>>> def main():
...     q()
...     a()
...
>>> main()
Why can't programmers tell jokes?
Timing!
```

在问题和答案之间会有三秒钟的间隙。哈欠。

但是异步的 示例 4-4 有些不同。

##### 示例 4-4\. 滑稽

```py
>>> import asyncio
>>>
>>> async def q():
...     print("Why can't programmers tell jokes?")
...     await asyncio.sleep(3)
...
>>> async def a():
...     print("Timing!")
...
>>> async def main():
...     await asyncio.gather(q(), a())
...
>>> asyncio.run(main())
Why can't programmers tell jokes?
Timing!
```

这次，答案应该在问题之后立即出现，然后是三秒的沉默——就像一个程序员在讲述一样。哈哈！咳咳。

###### 注

在 示例 4-4 中，我使用了 `asyncio.gather()` 和 `asyncio.run()`，但调用异步函数有多种方法。在使用 FastAPI 时，你不需要使用这些。

运行 示例 4-4 时，Python 认为是这样的：

1.  执行 `q()`。现在只是第一行。

1.  好吧，你懒惰的异步 `q()`，我已经设置好秒表，三秒钟后我会回来找你。

1.  与此同时，我将运行 `a()`，并立即打印答案。

1.  没有其他的 `await`，所以回到 `q()`。

1.  无聊的事件循环！我将坐在这里，等待剩下的三秒钟。

1.  好了，现在我完成了。

此示例使用 `asyncio.sleep()` 用于需要一些时间的函数，就像读取文件或访问网站的函数一样。在那些可能大部分时间都在等待的函数前面加上 `await`。该函数在其 `def` 前需要有 `async`。

###### 注意

如果你定义了一个带有 `async def` 的函数，它的调用者必须在调用它之前放置一个 `await`。而且调用者本身必须声明为 `async def`，并且 *其* 调用者必须 `await` 它，以此类推。

顺便说一句，即使函数中没有调用其他异步函数的 `await` 调用，你也可以将函数声明为 `async`。这不会有害处。

# FastAPI 和异步

经过那段漫长的田野之旅，让我们回到 FastAPI，看看为什么任何这些都很重要。

因为 Web 服务器花费大量时间在等待上，通过避免部分等待，即并发，可以提高性能。其他 Web 服务器使用了之前提到的多种方法：线程、gevent 等等。FastAPI 作为最快的 Python Web 框架之一的原因之一是其整合了异步代码，通过底层 Starlette 包的 ASGI 支持，以及其自己的一些发明。

###### 注意

单独使用 `async` 和 `await` 并不能使代码运行更快。事实上，由于异步设置的开销，它可能会慢一点。`async` 的主要用途是避免长时间等待 I/O。

现在，让我们看看我们之前的 Web 端点调用，并了解如何使它们异步。

FastAPI 文档中将将 URL 映射到代码的函数称为 *路径函数*。我还称它们为 *Web 端点*，你在 第三章 中看到它们的同步示例。让我们制作一些异步的。就像之前的示例一样，我们现在只使用简单的类型，比如数字和字符串。第五章 引入了 *类型提示* 和 Pydantic，我们将需要处理更复杂的数据结构。

示例 4-5 重新访问了上一章的第一个 FastAPI 程序，并将其改为异步。

##### 示例 4-5\. 一个害羞的异步端点（greet_async.py）

```py
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/hi")
async def greet():
    await asyncio.sleep(1)
    return "Hello? World?"
```

要运行那一块 Web 代码，你需要像 Uvicorn 这样的 Web 服务器。

第一种方法是在命令行上运行 Uvicorn：

```py
$ uvicorn greet_async:app
```

第二种方法，就像 示例 4-6 中一样，是在示例代码内部从主程序而不是模块中调用 Uvicorn。

##### 示例 4-6\. 另一个害羞的异步端点（greet_async_uvicorn.py）

```py
from fastapi import FastAPI
import asyncio
import uvicorn

app = FastAPI()

@app.get("/hi")
async def greet():
    await asyncio.sleep(1)
    return "Hello? World?"

if __name__ == "__main__":
    uvicorn.run("greet_async_uvicorn:app")
```

当作为独立程序运行时，Python 将其命名为`main`。那个`if __name__...`的东西是 Python 只在被调用为主程序时运行它的方式。是的，这很丑陋。

这段代码在返回其畏缩的问候之前将暂停一秒钟。与使用标准`sleep(1)`函数的同步函数唯一的区别是，异步示例允许 Web 服务器在此期间处理其他请求。

使用`asyncio.sleep(1)`模拟可能需要一秒钟的真实世界函数，比如调用数据库或下载网页。后面的章节将展示从 Web 层到服务层再到数据层的这些调用的实际示例，实际上在进行真正的工作时花费这段等待时间。

FastAPI 在接收到对 URL */hi* 的`GET`请求时会自行调用这个异步`greet()`路径函数。您不需要在任何地方添加`await`。但是对于您创建的任何其他`async def`函数定义，调用者必须在每次调用前加上`await`。

###### 注意

FastAPI 运行一个异步*事件循环*，协调异步路径函数，并为同步路径函数运行一个*线程池*。开发者不需要了解复杂的细节，这是一个很大的优点。例如，您不需要像之前的（独立的、非 FastAPI）笑话示例中那样运行`asyncio.gather()`或`asyncio.run()`方法。

# 直接使用 Starlette

FastAPI 没有像它对 Pydantic 那样公开 Starlette。Starlette 主要是引擎室内不断运行的机器。

但是如果你感兴趣，你可以直接使用 Starlette 来编写 Web 应用程序。前一章节的示例 3-1 可能看起来像示例 4-7。

##### 示例 4-7\. 使用 Starlette：starlette_hello.py

```py
from starlette.applications import Starlette
from starlette.responses import JSONResponse
from starlette.routing import Route

async def greeting(request):
    return JSONResponse('Hello? World?')

app = Starlette(debug=True, routes=[
    Route('/hi', greeting),
])
```

使用以下命令运行这个 Web 应用程序：

```py
$ uvicorn starlette_hello:app
```

在我看来，FastAPI 的新增功能使得 Web API 的开发更加容易。

# 插曲：清洁线索之屋

你拥有一个小（非常小：只有你一个人）的清洁公司。你一直靠泡面过活，但刚刚签下的合同将让你能够买更好的泡面。

您的客户购买了一座建造在 Clue 棋盘游戏风格中的老宅，并希望很快在那里举办角色派对。但是这个地方一团糟。如果玛丽·康多（Marie Kondo）看到这个地方，她可能会做如下事情：

+   尖叫

+   笑喷

+   逃跑

+   以上所有内容

您的合同包括一个速度奖金。如何在最少的时间内彻底清洁这个地方？最好的方法本来应该是有更多的线索保护单元（CPU），但是你就是全部。

所以你可以尝试以下其中之一：

+   在一个房间里做完所有事情，然后再下一个房间，以此类推。

+   在一个房间内完成一个特定任务，然后再下一个房间，以此类推。比如在厨房和餐厅里擦拭银器，或者在台球室里擦拭台球。

你选择的方法的总时间会不同吗？也许会。但更重要的是要考虑是否需要等待任何步骤的时间。一个例子可能是在地面上：清洁地毯和打蜡后，它们可能需要几个小时才能干透，然后才能把家具搬回去。

所以，这是你每个房间的计划：

1.  清洁所有静态部分（窗户等）。

1.  把房间里的所有家具移到大厅。

1.  从地毯和/或硬木地板上去除多年的污垢。

1.  做以下任何一种：

    1.  等地毯或打蜡干透，但告别你的奖金吧。

    1.  现在去下一个房间，然后重复。在最后一个房间之后，把家具搬回第一个房间，依此类推。

等待干燥的方法是同步的，如果时间不是一个因素并且你需要休息的话可能是最好的选择。第二种是异步的，为每个房间节省等待时间。

假设你选择了异步路径，因为有钱可赚。你让旧的垃圾发光，从你感激的客户那里获得了奖金。后来的派对结果非常成功，除了这些问题：

1.  一个没有梗的客人扮成了马里奥。

1.  你在舞厅里打蜡过度了，在醉醺醺的普拉姆教授穿着袜子溜冰，最终撞到桌子上，把香槟洒在了斯卡雷特小姐身上。

故事的道德：

+   需求可能会有冲突和/或奇怪。

+   估算时间和精力可能取决于许多因素。

+   顺序任务可能更多是一种艺术而不是科学。

+   当所有事情都完成时，你会感觉很棒。嗯，拉面。

# 复习

在总览了增加并发方式后，本章扩展了使用最近的 Python 关键字`async`和`await`的函数。它展示了 FastAPI 和 Starlette 如何处理传统的同步函数和这些新的异步函数。

下一章介绍了 FastAPI 的第二部分：Pydantic 如何帮助您定义数据。
