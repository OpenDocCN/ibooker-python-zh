# 第六章：高级任务调度：Futures 和 Friends

Dask 的计算流程遵循这四个主要逻辑步骤，每个任务可以并发和递归地进行：

1.  收集并读取输入数据。

1.  定义并构建表示需要对数据执行的计算集的计算图。

1.  运行计算（这在运行`.compute()`时发生）。

1.  将结果作为数据传递给下一步。

现在我们介绍更多使用 futures 控制这一流程的方法。到目前为止，您大部分时间在 Dask 中看到的是惰性操作，Dask 不会做任何工作，直到有事情强制执行计算。这种模式有许多好处，包括允许 Dask 的优化器在合适时合并步骤。然而，并非所有任务都适合惰性评估。一个常见的不适合惰性评估的模式是“fire-and-forget”，我们为其副作用而调用函数¹并且需要关注输出。尝试使用惰性评估（例如`dask.delayed`）来表达这一点会导致不必要的阻塞以强制执行计算。当惰性评估不是您需要的时候，您可以探索 Dask 的 futures。 Futures 可以用于远比 fire-and-forget 更多的用例。本章将探讨 futures 的许多常见用例。

###### 注意

您可能已经熟悉 Python 中的 futures。Dask 的 futures 是 Python concurrent.futures 库的扩展，允许您在其位置使用它们。类似于使用 Dask DataFrames 替代 pandas DataFrames，行为可能略有不同（尽管这里的差异较小）。

Dask futures 是 Dask 的分布式客户端库的一部分，因此您将通过`from dask.distributed import Client`导入它来开始。

###### 提示

尽管名称如此，您可以在本地使用 Dask 的分布式客户端。有关不同的本地部署类型，请参阅“分布式（Dask 客户端和调度器）”。

# 懒惰和热切评估再访

热切评估是编程中最常见的评估形式，包括 Python。虽然大多数热切评估是阻塞的——也就是说，程序在结果完成之前不会移到下一个语句——但您仍然可以进行异步/非阻塞的热切评估。 Futures 是表示非阻塞热切计算的一种方式。

非阻塞热切评估与懒惰评估相比仍然存在一些潜在缺点。其中一些挑战包括：

+   无法合并相邻阶段（有时被称为流水线）

+   不必要的计算：

    +   Dask 的优化器无法检测重复的子图。

    +   即使未依赖于未来结果的任何内容，它也可以计算。²

+   当未来启动并在其他未来上阻塞时，可能会出现过多的阻塞

+   需要更谨慎的内存管理

并非所有 Python 代码都会立即评估。在 Python 3 中，一些内置函数使用惰性评估，像 `map` 返回迭代器并且仅在请求时评估元素。

# Futures 的用例

许多常见用例可以通过仔细应用 futures 加速：

与其他异步服务器（如 Tornado）集成

尽管我们通常认为 Dask 大多数情况下不是“热路径”的正确解决方案，但也有例外情况，如动态计算的分析仪表板。

请求/响应模式

调用远程服务并（稍后）阻塞其结果。这可能包括查询诸如数据库、远程过程调用甚至网站等服务。

IO

输入/输出通常很慢，但你确实希望它们尽快开始。

超时

有时候你只在特定时间内获取结果感兴趣。例如，考虑一个增强的 ML 模型，你需要在一定时间内做出决策，迅速收集所有可用模型的分数，然后跳过超时的模型。

点火并忘记

有时候你可能不关心函数调用的结果，但你确实希望它被调用。Futures 允许你确保计算发生，而无需阻塞等待结果。

Actors

调用 actors 的结果是 futures。我们将在下一章介绍 actors。

在 Dask 中启动 futures 是非阻塞的，而在 Dask 中计算任务是阻塞的。这意味着当你向 Dask 提交一个 future 时，它会立即开始工作，但不会阻止（或阻塞）程序继续运行。

# 启动 Futures

启动 Dask futures 的语法与 `dask.delayed` 稍有不同。Dask futures 是通过 Dask 分布式客户端使用 `submit` 单个 future 或 `map` 多个 futures 启动，如 Example 6-1 所示。

##### Example 6-1. 启动 futures

```py
from dask.distributed import Client
client = Client()

def slow(x):
    time.sleep(3 * x)
    return 3 * x

slow_future = client.submit(slow, 1)
slow_futures = client.map(slow, range(1, 5))
```

与 `dask.delayed` 不同的是，一旦启动了 future，Dask 就开始计算其值。

###### 注意

虽然这里的 `map` 与 Dask bags 上的 `map` 有些相似，但每个项都会生成一个单独的任务，而 bags 能够将任务分组到分区以减少开销（尽管它们是延迟评估的）。

在 Dask 中，像 `persist()` 在 Dask 集合上一样，使用 futures 在内部。通过调用 `futures_of` 可以获取已持久化集合的 futures。这些 futures 的生命周期与您自己启动的 futures 相同。

# Future 生命周期

期货与`dask.delayed`有着不同的生命周期，超出了急切计算。使用`dask.delayed`时，中间计算会自动清理；然而，Dask 期货的结果会一直保存，直到期货显式取消或其引用在 Python 中被垃圾回收。如果你不再需要期货的值，你可以取消它并释放任何存储空间或核心，方法是调用`.cancel`。期货的生命周期在示例 6-2 中有所说明。

##### 示例 6-2\. 期货生命周期

```py
myfuture = client.submit(slow, 5) # Starts running
myfuture = None # future may be GCd and then stop since there are no other references

myfuture = client.submit(slow, 5) # Starts running
del myfuture # future may be GCd and then stop since there are no other references

myfuture = client.submit(slow, 5) # Starts running
# Future stops running, any other references point to canceled future
myfuture.cancel()
```

取消期货的行为与删除或依赖垃圾回收不同。如果有其他引用指向期货，则删除或将单个引用设置为`None`将不会取消期货。这意味着结果将继续存储在 Dask 中。另一方面，取消期货的缺点是，如果你错误地需要期货的值，这将导致错误。

###### 警告

在 Jupyter 笔记本中使用 Dask 时，笔记本可能会“保留”任何先前单元格的结果，因此即使期货未命名，它也将保留在 Dask 中。有一个关于此的[讨论](https://oreil.ly/zyy2H)，对于有兴趣的人来说，可以了解更多背景。

期货的字符串表示将向你展示它在生命周期中的位置（例如，`Future: slow status: cancelled,`）。

# 火而忘之

有时你不再需要一个期货，但你也不希望它被取消。这种模式称为火而忘之。这在像写数据、更新数据库或其他副作用的情况下最有用。如果所有对期货的引用都丢失了，垃圾回收可能导致期货被取消。为了解决这个问题，Dask 有一个名为`fire_and_forget`的方法，可以让你利用这种模式，就像在示例 6-3 中展示的那样，而不需要保留引用。

##### 示例 6-3\. 火而忘之

```py
from dask.distributed import fire_and_forget

def do_some_io(data):
    """
 Do some io we don't need to block on :)
 """
    import requests
    return requests.get('https://httpbin.org/get', params=data)

def business_logic():
    # Make a future, but we don't really care about its result, just that it
    # happens
    future = client.submit(do_some_io, {"timbit": "awesome"})
    fire_and_forget(future)

business_logic()
```

# 检索结果

更常见的是，你最终会想知道期货计算了什么（甚至只是是否遇到了错误）。对于不仅仅是副作用的期货，你最终会想要从期货获取返回值（或错误）。期货有阻塞方法`result`，如示例 6-4 所示，它会将期货中计算的值返回给你，或者从期货中引发异常。

##### 示例 6-4\. 获取结果

```py
future = client.submit(do_some_io, {"timbit": "awesome"})
future.result()
```

你可以扩展到多个期货，如示例 6-5，但有更快的方法可以做到。

##### 示例 6-5\. 获取结果列表

```py
for f in futures:
    time.sleep(2) # Business numbers logic
    print(f.result())
```

如果你同时拥有多个期货（例如通过`map`创建），你可以在它们逐步可用时获取结果（参见示例 6-6）。如果可以无序处理结果，这可以极大地提高处理时间。

##### 示例 6-6\. 当结果逐步可用时获取结果列表

```py
from dask.distributed import as_completed

for f in as_completed(futures):
    time.sleep(2) # Business numbers logic
    print(f.result())
```

在上面的例子中，通过处理完成的 futures，你可以让主线程在每个元素变得可用时执行其“业务逻辑”（类似于聚合的 `combine` 步骤）。如果 futures 在不同的时间完成，这可能会大大提高速度。

如果你有一个截止期限，比如为广告服务评分³ 或者与股票市场进行一些奇特的操作，你可能不想等待所有的 futures。相反，`wait` 函数允许你在超时后获取结果，如 示例 6-7 所示。

##### 示例 6-7\. 获取第一个 future（在时间限制内）

```py
from dask.distributed import wait
from dask.distributed.client import FIRST_COMPLETED

# Will throw an exception if no future completes in time.
# If it does not throw, the result has two lists:
# The done list may return between one and all futures.
# The not_done list may contain zero or more futures.
finished = wait(futures, 1, return_when=FIRST_COMPLETED)

# Process the returned futures
for f in finished.done:
    print(f.result())

# Cancel the futures we don't need
for f in finished.not_done:
    f.cancel()
```

这个时间限制可以应用于整个集合，也可以应用于一个 future。如果你想要所有的特性在给定时间内完成，那么你需要做更多的工作，如 示例 6-8 所示。

##### 示例 6-8\. 获取在时间限制内完成的任何 futures

```py
max_wait = 10
start = time.time()

while len(futures) > 0 and time.time() - start < max_wait:
    try:
        finished = wait(futures, 1, return_when=FIRST_COMPLETED)
        for f in finished.done:
            print(f.result())
        futures = finished.not_done
    except TimeoutError:
        True # No future finished in this cycle

# Cancel any remaining futures
for f in futures:
    f.cancel()
```

现在你可以从 futures 中获取结果了，你可以比较 `dask.delayed` 与 Dask futures 的执行时间，如 示例 6-9 所示。

##### 示例 6-9\. 查看 futures 可能更快

```py
slow_future = client.submit(slow, 1)
slow_delayed = dask.delayed(slow)(1)
# Pretend we do some other work here
time.sleep(1)
future_time = timeit.timeit(lambda: slow_future.result(), number=1)
delayed_time = timeit.timeit(lambda: dask.compute(slow_delayed), number=1)
print(
    f"""So as you can see by the future time {future_time} v.s. {delayed_time}
 the future starts running right away."""
)
```

在这个（虽然有些牵强的）例子中，你可以看到，通过尽早开始工作，future 在你得到结果时已经完成了，而 `dask.delayed` 则是在你到达时才开始的。

# 嵌套 Futures

与 `dask.delayed` 一样，你也可以从内部启动 futures。语法略有不同，因为你需要获取 `client` 对象的实例，它不可序列化，所以 `dask.distributed` 有一个特殊函数 `get_client` 来在分布式函数中获取 client。一旦你有了 client，你就可以像平常一样启动 future，如 示例 6-10 所示。

##### 示例 6-10\. 启动一个嵌套 future

```py
from dask.distributed import get_client

def nested(x):
    client = get_client() # The client is serializable, so we use get_client
    futures = client.map(slow, range(0, x))
    r = 0
    for f in as_completed(futures):
        r = r + f.result()
    return r

f = client.submit(nested, 3)
f.result()
```

注意，由于 Dask 使用集中式调度程序，客户端正在与该集中式调度程序通信，以确定在哪里放置 future。

# 结论

虽然 Dask 的主要构建块是 `dask.delayed`，但它并不是唯一的选择。你可以通过使用 Dask 的 futures 来控制更多的执行流程。Futures 非常适合 I/O、模型推断和对截止日期敏感的应用程序。作为对这种额外控制的交换，你需要负责管理 futures 的生命周期以及它们产生的数据，这是使用 `dask.delayed` 时不需要的。Dask 还有许多分布式数据结构，包括队列、变量和锁。虽然这些分布式数据结构比它们的本地对应物更昂贵，但它们也为你在控制任务调度方面提供了另一层灵活性。

¹ 就像写入磁盘文件或更新数据库记录一样。

² 不过，如果它的唯一引用被垃圾回收了，可能就不行了。

³ 我们认为这是 Dask 在发展空间较大的领域之一，如果你想要为截止日期关键事件实现微服务，可能需要考虑将 Dask 与 Ray 等其他系统结合使用。
