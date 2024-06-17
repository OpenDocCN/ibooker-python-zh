# 第三章 远程函数

构建现代大规模应用程序时，通常需要某种形式的分布式或并行计算。许多 Python 开发者通过 [multiprocessing 模块](https://oreil.ly/qj72E) 来了解并行计算。Multiprocessing 在处理现代应用程序需求方面存在局限性。这些需求包括：

+   在多个核心或机器上运行相同的代码

+   使用工具处理机器和处理故障

+   高效处理大参数

+   在进程之间轻松传递信息

与多进程不同，Ray 的远程函数满足这些要求。*远程*并不一定指的是一个独立的计算机，尽管名字如此；该函数可能在同一台机器上运行。Ray 提供的是将函数调用映射到正确进程的服务。在调用远程函数时，实际上是在多核或不同机器上异步运行，而无需关心如何或在哪里。

###### 注意

*异步*是指在同一时间内同时运行多个任务，而无需等待彼此完成的一种花哨方式。

在本章中，您将学习如何创建远程函数，等待它们完成并获取结果。一旦掌握了基础知识，您将学会组合远程函数以创建更复杂的操作。在深入学习之前，让我们首先理解前一章中忽略的一些内容。

# Ray 远程函数的基本要素

在 示例 2-7 中，您学会了如何创建基本的 Ray 远程函数。

当您调用远程函数时，它会立即返回一个 `ObjectRef`（即 future），这是对远程对象的引用。Ray 在后台创建并执行任务，并在完成时将结果写入原始引用中。然后，您可以调用 `ray.get` 来获取值。请注意，`ray.get` 是一个阻塞方法，等待任务执行完成后返回结果。

示例 2-7 中的一些细节值得理解。该示例在将迭代器传递给 `ray.get` 之前将其转换为列表。在调用 `ray.get` 时，需要这样做，以便传入一个 futures 列表或单个 future。¹ 该函数等待直到获取到所有对象，然后按顺序返回列表。

###### 小贴士

就像普通的 Ray 远程函数一样，重要的是考虑每次远程调用内部执行的工作量。例如，使用 `ray.remote` 递归计算阶乘会比在本地执行慢，因为每个函数内部的工作量很小，即使整体工作量可能很大。确切的时间取决于您的集群繁忙程度，但一般规则是，如果没有特殊资源，几秒钟内执行的任何内容都不值得远程调度。

到目前为止，在我们的例子中，使用 `ray.get` 是可以接受的，因为所有的 future 具有相同的执行时间。如果执行时间不同，例如在不同大小的数据批次上训练模型时，并且您不需要同时获取所有结果，这样做可能会非常浪费。不要直接调用 `ray.get`，而是应该使用 `ray.wait`，它会返回已经完成的请求数量的 futures。要查看性能差异，您需要修改远程函数以具有可变的睡眠时间，就像 示例 3-1 中的示例一样。

##### 示例 3-1\. [具有不同执行时间的远程函数](https://oreil.ly/UdVmt)

```py
@ray.remote
def remote_task(x):
    time.sleep(x)
    return x
```

正如您记得的那样，示例远程函数根据输入参数休眠。由于范围是按升序排列的，对其进行远程函数调用将导致按顺序完成的 futures。为了确保 futures 不会按顺序完成，您需要修改列表。一种方法是在映射远程函数到 `things` 之前调用 `things.sort(reverse=True)`。

要查看使用 `ray.get` 和 `ray.wait` 的差异，您可以编写一个函数，使用一些时间延迟收集 futures 的值，以模拟业务逻辑。

第一种选择，即不使用 `ray.wait`，在阅读上更简单和更清晰，如 示例 3-2 中所示，但不建议用于生产环境。

##### 示例 3-2\. [`ray.get` 无需等待](https://oreil.ly/UdVmt)

```py
# Process in order
def in_order():
    # Make the futures
    futures = list(map(lambda x: remote_task.remote(x), things))
    values = ray.get(futures)
    for v in values:
        print(f" Completed {v}")
        time.sleep(1) # Business logic goes here
```

第二种选择稍微复杂一些，如 示例 3-3 中所示。这通过调用 `ray.wait` 来找到下一个可用的 future 并迭代直到所有的 futures 完成。`ray.wait` 返回两个列表，一个是已完成任务的对象引用列表（请求的大小，默认为 1），另一个是剩余对象引用的列表。

##### 示例 3-3\. [使用 `ray.wait`](https://oreil.ly/UdVmt)

```py
# Process as results become available
def as_available():
    # Make the futures
    futures = list(map(lambda x: remote_task.remote(x), things))
    # While we still have pending futures
    while len(futures) > 0:
        ready_futures, rest_futures = ray.wait(futures)
        print(f"Ready {len(ready_futures)} rest {len(rest_futures)}")
        for id in ready_futures:
            print(f'completed value {id}, result {ray.get(id)}')
            time.sleep(1) # Business logic goes here
        # We just need to wait on the ones that are not yet available
        futures = rest_futures
```

将这些函数与 `timeit.time` 并行运行，您可以看到性能上的差异。需要注意的是，这种性能改进取决于非并行化业务逻辑（循环中的逻辑）所花费的时间。如果只是对结果求和，直接使用 `ray.get` 可能没问题，但如果执行更复杂的操作，则应该使用 `ray.wait`。当我们运行时，发现 `ray.wait` 大约快了两倍。您可以尝试变化睡眠时间并查看其效果。

您可能希望指定 `ray.wait` 的少数可选参数之一：

`num_returns`

在 Ray 返回之前等待完成的 `ObjectRef` 对象数量。您应该将 `num_returns` 设置为小于或等于 `ObjectRef` 对象输入列表的长度；否则，函数将抛出异常。² 默认值为 1。

`timeout`

在返回之前等待的最长时间（以秒为单位）。默认为 -1（表示无限）。

`fetch_local`

如果您只关心确保 futures 完成而不获取结果，则可以将其设置为 `false` 来禁用结果获取。

###### 提示

在 `ray.get` 和 `ray.wait` 中，`timeout` 参数非常重要。如果未指定此参数并且您的某个远程函数表现不佳（永远不会完成），`ray.get` 或 `ray.wait` 将永远不会返回，并且您的程序将永远阻塞。³ 因此，对于任何生产代码，我们建议在这两者中使用 `timeout` 参数以避免死锁。

Ray 的 `get` 和 `wait` 函数在处理超时时略有不同。当 `ray.wait` 发生超时时，Ray 不会引发异常；而是简单地返回比 `num_returns` 更少的准备好的 futures。然而，如果 `ray.get` 遇到超时，Ray 将引发 `GetTimeoutError`。请注意，`wait`/`get` 函数的返回并不意味着您的远程函数将被终止；它仍将在专用进程中运行。如果要释放资源，可以显式终止未来（请参阅以下提示）。

###### 提示

由于 `ray.wait` 可以以任意顺序返回结果，因此不依赖于结果顺序是非常重要的。如果需要对不同的记录进行不同的处理（例如，测试 A 组和 B 组的混合），则应在结果中进行编码（通常使用类型）。

如果某个任务在合理时间内未完成（例如，落后任务），可以使用与 `wait`/`get` 使用相同的 `ObjectRef` 使用 `ray.cancel` 取消该任务。您可以修改前面的 `ray.wait` 示例以添加超时并取消任何“坏”任务，从而得到类似 示例 3-4 的结果。

##### 示例 3-4\. [使用带有超时和取消的 `ray.wait`](https://oreil.ly/UdVmt)

```py
futures = list(map(lambda x: remote_task.remote(x), [1, threading.TIMEOUT_MAX]))
# While we still have pending futures
while len(futures) > 0:
    # In practice, 10 seconds is too short for most cases
    ready_futures, rest_futures = ray.wait(futures, timeout=10, num_returns=1)
    # If we get back anything less than num_returns 
    if len(ready_futures) < 1:
        print(f"Timed out on {rest_futures}")
        # Canceling is a good idea for long-running, unneeded tasks
        ray.cancel(*rest_futures)
        # You should break since you exceeded your timeout
        break
    for id in ready_futures:
        print(f'completed value {id}, result {ray.get(id)}')
        futures = rest_futures
```

###### 警告

取消任务不应是您正常程序流的一部分。如果发现自己经常需要取消任务，应调查出现的原因。取消已取消任务的任何后续 `wait` 或 `get` 调用未指定，并可能引发异常或返回不正确的结果。

另一个在上一章中跳过的细微点是，虽然到目前为止的示例仅返回单个值，但 Ray 远程函数可以像常规 Python 函数一样返回多个值。

对于在分布式环境中运行的人来说，容错性是一个重要的考虑因素。假设执行任务的工作进程意外死亡（因为进程崩溃或者机器故障），Ray 将重新运行任务（有延迟），直到任务成功或者超过最大重试次数为止。我们在第五章中更详细地讨论了容错性。

# 远程 Ray 函数的组合

您可以通过组合远程函数使您的远程函数变得更加强大。Ray 中与远程函数组合最常见的两种方法是流水线化和嵌套并行处理。您可以使用嵌套并行处理来表达递归函数。Ray 还允许您表达顺序依赖关系，而无需阻塞或收集驱动程序中的结果，这被称为*流水线化*。

您可以通过使用来自前一个`ray.remote`的`ObjectRef`对象作为新远程函数调用的参数来构建流水线函数。Ray 将自动获取`ObjectRef`对象并将底层对象传递给您的函数。这种方法允许在函数调用之间轻松协调。此外，这种方法最小化了数据传输；结果将直接发送到执行第二个远程函数的节点。在示例 3-5 中展示了这种顺序计算的简单示例。

##### 示例 3-5\. [Ray 管道化/顺序远程执行与任务依赖](https://oreil.ly/UdVmt)

```py
@ray.remote
def generate_number(s: int, limit: int, sl: float) -> int :
   random.seed(s)
   time.sleep(sl)
   return random.randint(0, limit)

@ray.remote
def sum_values(v1: int, v2: int, v3: int) -> int :
   return v1+v2+v3

# Get result
print(ray.get(sum_values.remote(generate_number.remote(1, 10, .1),
       generate_number.remote(5, 20, .2), generate_number.remote(7, 15, .3))))
```

此代码定义了两个远程函数，然后启动了第一个函数的三个实例。所有三个实例的`ObjectRef`对象随后被用作第二个函数的输入。在这种情况下，Ray 会等待所有三个实例完成后再开始执行`sum_values`。您不仅可以用这种方法传递数据，还可以表达基本的工作流风格的依赖关系。您可以传递的`ObjectRef`对象数量没有限制，同时还可以传递“普通”的 Python 对象。

您*不能*使用包含`ObjectRef`而不是直接使用`ObjectRef`的 Python 结构（例如列表、字典或类）。Ray 仅等待和解析直接传递给函数的`ObjectRef`对象。如果尝试传递结构，则必须在函数内部执行自己的`ray.wait`和`ray.get`。示例 3-6 是示例 3-5 的一个不起作用的变体。

##### 示例 3-6\. [断裂的顺序远程函数执行与任务依赖](https://oreil.ly/UdVmt)

```py
@ray.remote
def generate_number(s: int, limit: int, sl: float) -> int :
   random.seed(s)
   time.sleep(sl)
   return random.randint(0, limit)

@ray.remote
def sum_values(values: []) -> int :
   return sum(values)

# Get result
print(ray.get(sum_values.remote([generate_number.remote(1, 10, .1),
       generate_number.remote(5, 20, .2), generate_number.remote(7, 15, .3)])))
```

示例 3-6 已经从示例 3-5 修改为接受`ObjectRef`对象的列表作为参数，而不是`ObjectRef`对象本身。Ray 不会“查看”任何被传递的结构。因此，函数会立即被调用，由于类型不匹配，函数将失败并显示错误`TypeError: unsupported operand type(s) for +: 'int' and 'ray._raylet.ObjectRef'`。您可以通过使用`ray.wait`和`ray.get`来修复此错误，但这仍会过早地启动函数，导致不必要的阻塞。

在另一种组合方法*嵌套并行性*中，您的远程函数启动额外的远程函数。这在许多情况下都很有用，包括实现递归算法和将超参数调整与并行模型训练结合起来。⁴ 让我们看看两种实现嵌套并行性的方法（示例 3-7）。

##### 示例 3-7\. [实现嵌套并行性](https://oreil.ly/UdVmt)

```py
@ray.remote
def generate_number(s: int, limit: int) -> int :
   random.seed(s)
   time.sleep(.1)
   return randint(0, limit)

@ray.remote
def remote_objrefs():
   results = []
   for n in range(4):
       results.append(generate_number.remote(n, 4*n))
   return results

@ray.remote
def remote_values():
   results = []
   for n in range(4):
       results.append(generate_number.remote(n, 4*n))
   return ray.get(results)

print(ray.get(remote_values.remote()))
futures = ray.get(remote_objrefs.remote())
while len(futures) > 0:
    ready_futures, rest_futures = ray.wait(futures, timeout=600, num_returns=1)
    # If we get back anything less than num_returns, there was a timeout
    if len(ready_futures) < 1:
        ray.cancel(*rest_futures)
        break
    for id in ready_futures:
        print(f'completed result {ray.get(id)}')
        futures = rest_futures
```

这段代码定义了三个远程函数：

`generate_numbers`

一个生成随机数的简单函数

`remote_objrefs`

调用多个远程函数并返回生成的`ObjectRef`对象

`remote_values`

调用多个远程函数，等待它们完成并返回结果值

正如您从这个例子中可以看到的那样，嵌套并行性允许两种方法。在第一种情况（`remote_objrefs`）中，您将所有的`ObjectRef`对象返回给聚合函数的调用者。调用代码负责等待所有远程函数的完成并处理结果。在第二种情况（`remote_values`）中，聚合函数等待所有远程函数的执行完成，并返回实际的执行结果。

返回所有的`ObjectRef`对象可以更灵活地进行非顺序消耗，就像在`ray.await`中描述的那样，但对于许多递归算法来说并不适用。对于许多递归算法（例如快速排序、阶乘等），我们有许多级别的组合步骤需要执行，需要在每个递归级别上组合结果。

# Ray 远程最佳实践

当您使用远程函数时，请记住不要将它们设计得太小。如果任务非常小，使用 Ray 可能比不使用 Ray 的 Python 花费更长的时间。其原因是每个任务调用都有非常复杂的开销，例如调度、数据传递、进程间通信（IPC）和更新系统状态。要从并行执行中获得真正的优势，您需要确保这些开销与函数本身的执行时间相比是可以忽略不计的。⁵

如本章所述，Ray `remote` 最强大的功能之一是能够并行执行函数。一旦你调用远程函数，远程对象（future）的句柄会立即返回，调用者可以继续在本地执行或者继续执行其他远程函数。如果此时调用 `ray.get`，你的代码将会阻塞，等待远程函数完成，结果就是你将失去并行性。为了确保你的代码并行化，你应该只在绝对需要数据继续主线程执行的时候调用 `ray.get`。此外，正如我们所描述的，推荐使用 `ray.wait` 而不是直接使用 `ray.get`。另外，如果一个远程函数的结果需要用于执行另一个远程函数（们），考虑使用流水线处理（前面描述过）来利用 Ray 的任务协调。

当你将参数提交给远程函数时，Ray 并不直接将它们提交给远程函数，而是将参数复制到对象存储中，然后将`ObjectRef`作为参数传递。因此，如果你将相同的参数发送给多个远程函数，你将为将相同的数据存储到对象存储中多次支付（性能）代价。数据的大小越大，惩罚越大。为了避免这种情况，如果你需要将相同的数据传递给多个远程函数，一个更好的选择是首先将共享数据放入对象存储中，然后使用生成的`ObjectRef`作为函数的参数。我们在“Ray 对象”中说明了如何做到这一点。

正如我们将在第五章中展示的那样，远程函数调用是由 Raylet 组件完成的。如果你从单个客户端调用了大量远程函数，所有这些调用都是由一个单独的 Raylet 完成的。因此，对于给定的 Raylet 来处理这些请求，需要一定的时间，这可能会延迟所有函数的启动。一个更好的方法，正如[“Ray 设计模式”文档](https://oreil.ly/PTZOI)中所描述的，是使用调用树——即前面部分描述的嵌套函数调用。基本上，一个客户端创建了几个远程函数，每个远程函数依次创建更多的远程函数，依此类推。在这种方法中，调用被分布在多个 Raylet 之间，允许调度更快地进行。

每次你使用`@ray.remote`装饰器定义一个远程函数时，Ray 都会将这些定义导出到所有 Ray 工作节点，这需要一些时间（特别是如果你有很多节点）。为了减少函数导出的数量，一个好的实践是在顶层定义尽可能多的远程任务，避免在循环和本地函数中定义它们。

# 通过一个示例将其整合

由其他模型组成的 ML 模型（例如集成模型）非常适合使用 Ray 进行评估。示例 3-8 展示了使用 Ray 的函数组合来处理假设的网页链接垃圾邮件模型的样子。

##### 示例 3-8\. [集成模型](https://oreil.ly/UdVmt)

```py
import random

@ray.remote
def fetch(url: str) -> Tuple[str, str]:
    import urllib.request
    with urllib.request.urlopen(url) as response:
       return (url, response.read())

@ray.remote
def has_spam(site_text: Tuple[str, str]) -> bool:
    # Open the list of spammers or download it
    spammers_url = (
        "https://raw.githubusercontent.com/matomo-org/" + 
        "referrer-spam-list/master/spammers.txt"
    )
    import urllib.request
    with urllib.request.urlopen(spammers_url) as response:
            spammers = response.readlines()
            for spammer in spammers:
                if spammer in site_text[1]:
                    return True
    return False

@ray.remote
def fake_spam1(us: Tuple[str, str]) -> bool:
    # You should do something fancy here with TF or even just NLTK
    time.sleep(10)
    if random.randrange(10) == 1:
        return True
    else:
        return False

@ray.remote
def fake_spam2(us: Tuple[str, str]) -> bool:
    # You should do something fancy here with TF or even just NLTK
    time.sleep(5)
    if random.randrange(10) > 4:
        return True
    else:
        return False

@ray.remote
def combine_is_spam(us: Tuple[str, str], model1: bool, model2: bool, model3: bool) -> 
Tuple[str, str, bool]:
    # Questionable fake ensemble
    score = model1 * 0.2 + model2 * 0.4 + model3 * 0.4
    if score > 0.2:
        return True
    else:
        return False
```

使用 Ray 而不是对所有模型的评估时间求和，您只需等待最慢的模型，而所有更快完成的模型则是“免费”的。例如，如果这些模型运行时间相等，那么在没有 Ray 的情况下串行评估这些模型将需要几乎三倍的时间。

# 结论

在本章中，您了解了一个基本的 Ray 特性——远程函数的调用及其在跨多个核心和机器上创建并行异步执行中的应用。您还学习了多种等待远程函数执行完成的方法，以及如何使用 `ray.wait` 避免代码中的死锁。

最后，您了解了远程函数组合及其如何用于基本执行控制（迷你工作流）。您还学会了实现嵌套并行处理，使您能够并行调用多个函数，而每个这些函数又可以依次调用更多并行函数。在下一章中，您将学习如何通过使用 actors 在 Ray 中管理状态。

¹ Ray 不会“深入”类或结构以解析 futures，因此如果您有一系列 futures 的列表或包含 future 的类，Ray 将不会解析“内部”future。

² 当传入的 `ObjectRef` 对象列表为空时，Ray 将其视为特殊情况，并立即返回，而不管 `num_returns` 的值如何。

³ 如果您正在交互式地工作，您可以通过 `SIGINT` 或 Jupyter 中的停止按钮来解决这个问题。

⁴ 然后，您可以并行训练多个模型，并使用数据并行梯度计算来训练每个模型，从而实现嵌套并行处理。

⁵ 作为练习，您可以从 示例 2-7 的函数中移除 `sleep`，您会发现使用 Ray 在远程函数执行上比常规函数调用花费的时间长几倍。开销并非恒定，而是取决于网络、调用参数的大小等因素。例如，如果您只有少量数据要传输，那么开销将比传输整个维基百科文本作为参数时要低。
