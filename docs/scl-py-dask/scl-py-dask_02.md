# 第二章：开始使用 Dask

我们非常高兴您决定通过尝试来探索是否 Dask 是适合您的系统。在本章中，我们将专注于在本地模式下启动 Dask。使用这种方式，我们将探索一些更为简单的并行计算任务（包括大家喜爱的单词统计）。¹

# 在本地安装 Dask

在本地安装 Dask 相对来说是比较简单的。如果您想要在多台机器上运行，当您从 conda 环境（或 virtualenv）开始时，通常会更容易。这使得您可以通过运行 `pip freeze` 来确定您依赖的软件包，在扩展时确保它们位于所有工作节点上。

虽然您可以直接运行 `pip install -U dask`，但我们更倾向于使用 conda 环境，因为这样更容易匹配集群上的 Python 版本，这使得您可以直接连接本地机器到集群。² 如果您的机器上还没有 conda，[Miniforge](https://oreil.ly/qVDa7) 是一个快速好用的方式来在多个平台上安装 conda。在新的 conda 环境中安装 Dask 的过程显示在 Example 2-1 中。

##### Example 2-1\. 在新的 conda 环境中安装 Dask

```py
conda create -n dask python=3.8.6  mamba -y
conda activate dask
mamba install --yes python==3.8.6 cytoolz dask==2021.7.0 numpy \
      pandas==1.3.0 beautifulsoup4 requests
```

在这里，我们安装的是特定版本的 Dask，而不仅仅是最新版本。如果您计划稍后连接到集群，选择与集群上安装的相同版本的 Dask 将非常有用。

###### 注意

您不必在本地安装 Dask。有一个带有 Dask 的 [BinderHub 示例](https://oreil.ly/EK5n5) 和分布式选项，包括 [Dask 的创建者提供的一个](https://oreil.ly/3UEq-)，您可以使用这些选项来运行 Dask，以及其他提供者如 [SaturnCloud](https://oreil.ly/_6SyV)。尽管如此，即使最终使用了这些服务之一，我们还是建议在本地安装 Dask。

# Hello Worlds

现在您已经在本地安装了 Dask，是时候通过其各种 API 版本的“Hello World”来尝试了。开始 Dask 的选项有很多。目前，您应该使用 LocalCluster，如 Example 2-2 中所示。

##### Example 2-2\. 使用 LocalCluster 启动 Dask

```py
import dask
from dask.distributed import Client
client = Client() # Here we could specify a cluster, defaults to local mode
```

## 任务 Hello World

Dask 的核心构建块之一是 `dask.delayed`，它允许您并行运行函数。如果您在多台机器上运行 Dask，这些函数也可以分布（或者说散布）到不同的机器上。当您用 `dask.delayed` 包装一个函数并调用它时，您会得到一个代表所需计算的“延迟”对象。当您创建了一个延迟对象时，Dask 只是记下了您可能希望它执行的操作。就像懒惰的青少年一样，您需要明确告知它。您可以通过 `dask.submit` 强制 Dask 开始计算值，这会产生一个“future”。您可以使用 `dask.compute` 来启动计算延迟对象和 futures，并返回它们的值。³

### 睡眠任务

通过编写一个意图上慢的函数，比如调用`sleep`的`slow_task`，可以轻松地看到性能差异。然后，您可以通过在几个元素上映射该函数，使用或不使用`dask.delayed`，来比较 Dask 与“常规”Python 的性能，如示例 2-3 所示。

##### 示例 2-3\. 睡眠任务

```py
import timeit

def slow_task(x):
    import time
    time.sleep(2) # Do something sciency/business
    return x

things = range(10)

very_slow_result = map(slow_task, things)
slowish_result = map(dask.delayed(slow_task), things)

slow_time = timeit.timeit(lambda: list(very_slow_result), number=1)
fast_time = timeit.timeit(
    lambda: list(
        dask.compute(
            *slowish_result)),
    number=1)
print("In sequence {}, in parallel {}".format(slow_time, fast_time))
```

当我们运行这个例子时，我们得到了`In sequence 20.01662155520171, in parallel 6.259156636893749`，这显示了 Dask 可以并行运行部分任务，但并非所有任务。⁴

### 嵌套任务

`dask.delayed`的一个很好的特点是您可以在其他任务内启动任务。⁵这的一个简单的现实世界例子是网络爬虫，在这个例子中，当您访问一个网页时，您希望从该页面获取所有链接，如示例 2-4 所示。

##### 示例 2-4\. 网络爬虫

```py
@dask.delayed
def crawl(url, depth=0, maxdepth=1, maxlinks=4):
    links = []
    link_futures = []
    try:
        import requests
        from bs4 import BeautifulSoup
        f = requests.get(url)
        links += [(url, f.text)]
        if (depth > maxdepth):
            return links # base case
        soup = BeautifulSoup(f.text, 'html.parser')
        c = 0
        for link in soup.find_all('a'):
            if "href" in link:
                c = c + 1
                link_futures += crawl(link["href"],
                                      depth=(depth + 1),
                                      maxdepth=maxdepth)
                # Don't branch too much; we're still in local mode and the web is
                # big
                if c > maxlinks:
                    break
        for r in dask.compute(link_futures):
            links += r
        return links
    except requests.exceptions.InvalidSchema:
        return [] # Skip non-web links

dask.compute(crawl("http://holdenkarau.com/"))
```

###### 注意

实际上，幕后仍然涉及一些中央协调（包括调度器），但以这种嵌套方式编写代码的自由性非常强大。

我们在“任务依赖关系”中涵盖了其他类型的任务依赖关系。

## 分布式集合

除了低级任务 API 之外，Dask 还有分布式集合。这些集合使您能够处理无法放入单台计算机的数据，并在其上自然分发工作，这被称为*数据并行性*。Dask 既有称为*bag*的无序集合，也有称为*array*的有序集合。Dask 数组旨在实现一些 ndarray 接口，而 bags 则更专注于函数式编程（例如`map`和`filter`）。您可以从文件加载 Dask 集合，获取本地集合并进行分发，或者将`dask.delayed`任务的结果转换为集合。

在分布式集合中，Dask 使用分区来拆分数据。分区用于降低与操作单个行相比的调度成本，详细信息请参见“分区/分块集合”。

### Dask 数组

Dask 数组允许您超越单个计算机内存或磁盘容量的限制。Dask 数组支持许多标准 NumPy 操作，包括平均值和标准差等聚合操作。Dask 数组中的`from_array`函数将类似本地数组的集合转换为分布式集合。示例 2-5 展示了如何从本地数组创建分布式数组，然后计算平均值。

##### 示例 2-5\. 创建分布式数组并计算平均值

```py
import dask.array as da
distributed_array = da.from_array(list(range(0, 1000)))
avg = dask.compute(da.average(distributed_array))
```

与所有分布式集合一样，Dask 数组上的昂贵操作与本地数组上的操作并不相同。在下一章中，您将更多地了解 Dask 数组的实现方式，并希望能更好地直觉到它们的性能。

创建一个分布式集合从本地集合使用分布式计算的两个基本构建块，称为*分散-聚集模式*。虽然原始数据集必须来自本地计算机，适合单台机器，但这已经扩展了您可以使用的处理器数量，以及您可以利用的中间内存，使您能够更好地利用现代云基础设施和扩展。一个实际的用例可能是分布式网络爬虫，其中要爬行的种子 URL 列表可能是一个小数据集，但在爬行时需要保存的内存可能是数量级更大，需要分布式计算。

### Dask 包和词频统计

Dask 包实现了比 Dask 数组更多的函数式编程接口。大数据的“Hello World”是词频统计，使用函数式编程接口更容易实现。由于您已经编写了一个爬虫函数，您可以使用`from_delayed`函数将其输出转换为 Dask 包（参见示例 2-6）。

##### 示例 2-6\. 将爬虫函数的输出转换为 Dask 包

```py
import dask.bag as db
githubs = [
    "https://github.com/scalingpythonml/scalingpythonml",
    "https://github.com/dask/distributed"]
initial_bag = db.from_delayed(map(crawl, githubs))
```

现在您有了一个 Dask 包集合，您可以在其上构建每个人最喜欢的词频示例。第一步是将您的文本包转换为词袋，您可以通过使用`map`来实现（参见示例 2-7）。一旦您有了词袋，您可以使用 Dask 的内置`frequency`方法（参见示例 2-8），或者使用函数转换编写自己的`frequency`方法（参见示例 2-9）。

##### 示例 2-7\. 将文本包转换为词袋

```py
words_bag = initial_bag.map(
    lambda url_contents: url_contents[1].split(" ")).flatten()
```

##### 示例 2-8\. 使用 Dask 的内置`frequency`方法

```py
dask.compute(words_bag.frequencies())
```

##### 示例 2-9\. 使用函数转换编写自定义`frequency`方法

```py
def make_word_tuple(w):
    return (w, 1)

def get_word(word_count):
    return word_count[0]

def sum_word_counts(wc1, wc2):
    return (wc1[0], wc1[1] + wc2[1])

word_count = words_bag.map(make_word_tuple).foldby(get_word, sum_word_counts)
```

在 Dask 包上，`foldby`，`frequency`和许多其他的归约返回一个单分区包，这意味着归约后的数据需要适合单台计算机。Dask DataFrame 处理归约方式不同，没有同样的限制。

## Dask DataFrame（Pandas/人们希望大数据是什么）

Pandas 是最流行的 Python 数据库之一，而 Dask 有一个 DataFrame 库，实现了大部分 Pandas API。由于 Python 的鸭子类型，您通常可以在 Pandas 的位置使用 Dask 的分布式 DataFrame 库。不是所有的 API 都会完全相同，有些部分没有实现，所以请确保您有良好的测试覆盖。

###### 警告

您在使用 Pandas 时的慢和快的直觉并不适用。我们将在“Dask DataFrames”中进一步探讨。

为了演示您如何使用 Dask DataFrame，我们将重新编写示例 2-6 到 2-8 来使用它。与 Dask 的其他集合一样，您可以从本地集合、未来数据或分布式文件创建 DataFrame。由于您已经创建了一个爬虫函数，您可以使用 `from_delayed` 函数将其输出转换为 Dask bag。您可以使用像 `explode` 和 `value_counts` 这样的 pandas API，而不是使用 `map` 和 `foldby`，如 示例 2-10 所示。

##### 示例 2-10\. DataFrame 单词计数

```py
import dask.dataframe as dd

@dask.delayed
def crawl_to_df(url, depth=0, maxdepth=1, maxlinks=4):
    import pandas as pd
    crawled = crawl(url, depth=depth, maxdepth=maxdepth, maxlinks=maxlinks)
    return pd.DataFrame(crawled.compute(), columns=[
                        "url", "text"]).set_index("url")

delayed_dfs = map(crawl_to_df, githubs)
initial_df = dd.from_delayed(delayed_dfs)
wc_df = initial_df.text.str.split().explode().value_counts()

dask.compute(wc_df)
```

# 结论

在本章中，您已经在本地机器上成功运行了 Dask，并且看到了大部分 Dask 内置库的不同“Hello World”（或入门）示例。随后的章节将更详细地探讨这些不同的工具。

现在您已经在本地机器上成功运行了 Dask，您可能想跳转到 第十二章 并查看不同的部署机制。在大多数情况下，您可以在本地模式下运行示例，尽管有时速度可能会慢一些或规模较小。然而，下一章将讨论 Dask 的核心概念，即将要介绍的一个示例强调了在多台机器上运行 Dask 的好处，并且在集群上探索通常更容易。如果您没有可用的集群，您可能希望使用类似 [MicroK8s](https://microk8s.io) 的工具设置一个模拟集群。

¹ 单词计数可能是一个有些陈旧的例子，但它是一个重要的例子，因为它涵盖了既可以通过最小的协调完成的工作（将文本分割成单词），也可以通过多台计算机之间的协调完成的工作（对单词求和）。

² 以这种方式部署您的 Dask 应用程序存在一些缺点，如 第十二章 中所讨论的，但它可以是一种极好的调试技术。

³ 只要它们适合内存。

⁴ 当我们在集群上运行时，性能会变差，因为与小延迟相比，将任务分发到远程计算机存在一定的开销。

⁵ 这与 Apache Spark 十分不同，后者只有驱动程序/主节点可以启动任务。
