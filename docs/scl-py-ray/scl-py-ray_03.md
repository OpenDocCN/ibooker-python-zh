# 第二章. 使用 Ray 入门（本地）

正如我们所讨论的，Ray 可以用来管理从单台计算机到集群的资源。开始使用本地安装更简单，可以利用多核/多 CPU 机器的并行性。即使在部署到集群时，您也会希望在本地安装 Ray 以进行开发。安装完 Ray 后，我们将向您展示如何创建和调用第一个异步并行化函数，并在 actor 中存储状态。

###### 提示

如果您着急的话，也可以在书籍的 GitHub 仓库上使用 [Gitpod](https://oreil.ly/7YUbX) 获取带有示例的 Web 环境，或者查看 [Anyscale 的托管 Ray](https://oreil.ly/UacuZ)。

# 安装

即使在单台机器上，安装 Ray 的复杂程度从相对简单到非常复杂不等。Ray 将轮子发布到 Python 包索引（PyPI），遵循正常的发布周期和每夜版发布。目前这些轮子仅适用于 x86 用户，因此 ARM 用户大多需要从源代码构建 Ray。¹

###### 提示

在 macOS 上的 M1 ARM 用户可以使用 Rosetta 上的 x86 包。会有一些性能下降，但设置起来简单得多。要使用 x86s 包，请安装 macOS 的 Anaconda。

## x86 和 M1 ARM 的安装

大多数用户可以运行 `pip install -U ray` 从 PyPI 自动安装 Ray。当你需要在多台机器上分布计算时，在 Conda 环境中工作通常更容易，这样你可以匹配 Python 版本和集群的包依赖关系。示例 2-1 中的命令设置了一个带有 Python 的全新 Conda 环境，并且使用最少的依赖项安装了 Ray。

##### 示例 2-1\. [在 Conda 环境中安装 Ray](https://oreil.ly/rxdEC)

```py
conda create -n ray python=3.7  mamba -y
conda activate ray
# In a Conda env this won't be auto-installed with Ray, so add them
pip install jinja2 python-dateutil cloudpickle packaging pygments \
    psutil nbconvert ray
```

## ARM 的安装（来自源代码）

对于 ARM 用户或任何没有预先构建轮子的系统架构的用户，您需要从源代码构建 Ray。在我们的 ARM Ubuntu 系统上，我们需要安装额外的软件包，如示例 2-2 所示。

##### 示例 2-2\. [从源代码安装 Ray](https://oreil.ly/k97Lt)

```py
sudo apt-get install -y git tzdata bash libhdf5-dev curl pkg-config wget \
  cmake build-essential zlib1g-dev zlib1g openssh-client gnupg unzip libunwind8 \
  libunwind-dev openjdk-11-jdk git
# Depending on Debian version
sudo apt-get install -y libhdf5-100 || sudo apt-get install -y libhdf5-103
# Install bazelisk to install bazel (needed for Ray's CPP code)
# See https://github.com/bazelbuild/bazelisk/releases
# On Linux ARM
BAZEL=bazelisk-linux-arm64
# On Mac ARM
# BAZEL=bazelisk-darwin-arm64
wget -q https://github.com/bazelbuild/bazelisk/releases/download/v1.10.1/${BAZEL} \
  -O /tmp/bazel
chmod a+x /tmp/bazel
sudo mv /tmp/bazel /usr/bin/bazel
# Install node, needed for the UI
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo bash -
sudo apt-get install -y nodejs
```

如果您是不想使用 Rosetta 的 M1 Mac 用户，您需要安装一些依赖项。您可以使用 Homebrew 和 `pip` 安装它们，如示例 2-3 所示。

##### 示例 2-3\. [在 M1 上安装额外的依赖项](https://oreil.ly/4KDxL)

```py
brew install bazelisk wget python@3.8 npm
# Make sure Homebrew Python is used before system Python
export PATH=$(brew --prefix)/opt/python@3.8/bin/:$PATH
echo "export PATH=$(brew --prefix)/opt/python@3.8/bin/:$PATH" >> ~/.zshrc
echo "export PATH=$(brew --prefix)/opt/python@3.8/bin/:$PATH" >> ~/.bashrc
# Install some libraries vendored incorrectly by Ray for ARM
pip3 install --user psutil cython colorama
```

您需要单独构建一些 Ray 组件，因为它们使用不同的语言编写。这确实使安装变得更加复杂，但您可以按照示例 2-4 中的步骤操作。

##### 示例 2-4\. [安装 Ray 的构建工具](https://oreil.ly/k97Lt)

```py
git clone https://github.com/ray-project/ray.git
cd ray
# Build the Ray UI
pushd python/ray/new_dashboard/client; npm install && npm ci && npm run build; popd
# Specify a specific bazel version as newer ones sometimes break.
export USE_BAZEL_VERSION=4.2.1
cd python
# Mac ARM USERS ONLY: clean up the vendored files
rm -rf ./thirdparty_files
# Install in edit mode or build a wheel
pip install -e .
# python setup.py bdist_wheel
```

###### 提示

构建中最慢的部分是编译 C++代码，即使在现代计算机上也可能需要一个小时。如果您有一台装有多个 ARM 处理器的集群，仅在集群上构建一次 wheel 并在集群上重用它通常是值得的。

# Hello Worlds

现在您已经安装了 Ray，是时候了解一些 Ray API 了。稍后我们会更详细地介绍这些 API，所以现在不要太过于纠结于细节。

## Ray 远程（任务/未来对象）Hello World

Ray 的核心构建块之一是远程函数，它们返回未来对象。这里的术语*远程*表示*远程到我们的主进程*，可以在同一台或不同的机器上。

要更好地理解这一点，您可以编写一个返回其运行位置的函数。Ray 将工作分布在多个进程之间，在分布式模式下，还可以在多台主机之间工作。这个函数的本地（非 Ray）版本显示在示例 2-5 中。

##### 示例 2-5\. [一个本地（常规）函数](https://oreil.ly/perip)

```py
def hi():
    import os
    import socket
    return f"Running on {socket.gethostname()} in pid {os.getpid()}"
```

您可以使用`ray.remote`装饰器创建一个远程函数。调用远程函数与调用本地函数有所不同，需要在函数上调用`.remote`。当您调用远程函数时，Ray 将立即返回一个未来对象，而不是阻塞等待结果。您可以使用`ray.get`来获取这些未来对象返回的值。要将示例 2-5 转换为远程函数，您只需使用`ray.remote`装饰器，如示例 2-6 所示。

##### 示例 2-6\. [将上一个函数转换为远程函数](https://oreil.ly/perip)

```py
@ray.remote
def remote_hi():
    import os
    import socket
    return f"Running on {socket.gethostname()} in pid {os.getpid()}"
future = remote_hi.remote()
ray.get(future)
```

当您运行这两个示例时，您会看到第一个在同一个进程中执行，而 Ray 将第二个调度到另一个进程中。当我们运行这两个示例时，分别得到`Running on jupyter-holdenk in pid 33`和`Running on jupyter-holdenk in pid 173`。

### Sleepy task

通过创建一个故意缓慢的函数（在我们的例子中是`slow_task`），并让 Python 在常规函数调用和 Ray 远程调用中计算，您可以轻松（虽然是人为的）了解远程未来如何帮助。参见示例 2-7。

##### 示例 2-7\. [使用 Ray 并行化一个故意缓慢的函数](https://oreil.ly/perip)

```py
import timeit

def slow_task(x):
    import time
    time.sleep(2) # Do something sciency/business
    return x

@ray.remote
def remote_task(x):
    return slow_task(x)

things = range(10)

very_slow_result = map(slow_task, things)
slowish_result = map(lambda x: remote_task.remote(x), things)

slow_time = timeit.timeit(lambda: list(very_slow_result), number=1)
fast_time = timeit.timeit(lambda: list(ray.get(list(slowish_result))), number=1)
print(f"In sequence {slow_time}, in parallel {fast_time}")
```

当您运行此代码时，您会看到通过使用 Ray 远程函数，您的代码能够同时执行多个远程函数。虽然您可以使用`multiprocessing`在没有 Ray 的情况下做到这一点，但 Ray 会为您处理所有细节，并且还可以最终扩展到多台机器。

### 嵌套和链式任务

Ray 在分布式处理领域中非常显著，因为它允许嵌套和链式任务。在其他任务内部启动更多任务可以使某些类型的递归算法更容易实现。

使用嵌套任务的更直接的示例之一是网络爬虫。在网络爬虫中，我们访问的每个页面都可以启动对该页面上链接的多个额外访问，如示例 2-8 所示。

##### 示例 2-8\. [带有嵌套任务的网络爬虫](https://oreil.ly/perip)

```py
@ray.remote
def crawl(url, depth=0, maxdepth=1, maxlinks=4):
    links = []
    link_futures = []
    import requests
    from bs4 import BeautifulSoup
    try:
        f = requests.get(url)
        links += [(url, f.text)]
        if (depth > maxdepth):
            return links # base case
        soup = BeautifulSoup(f.text, 'html.parser')
        c = 0
        for link in soup.find_all('a'):
            try:
                c = c + 1
                link_futures += [crawl.remote(link["href"], depth=(depth+1),
                                   maxdepth=maxdepth)]
                # Don't branch too much; we're still in local mode and the web is big
                if c > maxlinks:
                    break
            except:
                pass
        for r in ray.get(link_futures):
            links += r
        return links
    except requests.exceptions.InvalidSchema:
        return [] # Skip nonweb links
    except requests.exceptions.MissingSchema:
        return [] # Skip nonweb links

ray.get(crawl.remote("http://holdenkarau.com/"))
```

许多其他系统要求所有任务都在中央协调器节点上启动。即使支持以嵌套方式启动任务的系统，通常也仍然依赖于中央调度器。

## 数据 Hello World

Ray 在处理结构化数据时使用了稍微有限的数据集 API。Apache Arrow 驱动 Ray 的数据集 API。Arrow 是一种面向列的、语言无关的格式，具有一些流行的操作。许多流行的工具支持 Arrow，使它们之间的轻松转移成为可能（例如 Spark、Ray、Dask 和 TensorFlow）。

Ray 最近仅在版本 1.9 中添加了数据集上的键控聚合。最流行的分布式数据示例是词频统计，这需要聚合。我们可以执行令人尴尬的并行任务，如映射转换，构建一个网页数据集，如示例 2-9 所示。

##### 示例 2-9\. [构建一个网页数据集](https://oreil.ly/perip)

```py
# Create a dataset of URL objects. We could also load this from a text file
# with ray.data.read_text()
urls = ray.data.from_items([
    "https://github.com/scalingpythonml/scalingpythonml",
    "https://github.com/ray-project/ray"])

def fetch_page(url):
    import requests
    f = requests.get(url)
    return f.text

pages = urls.map(fetch_page)
# Look at a page to make sure it worked
pages.take(1)
```

Ray 1.9 添加了`GroupedDataset`以支持各种类型的聚合。通过调用`groupby`，使用列名或返回键的函数，您可以获得`GroupedDataset`。`GroupedDataset`内置支持`count`、`max`、`min`和其他常见聚合。您可以使用`GroupedDataset`将示例 2-9 扩展为词频统计示例，如示例 2-10 所示。

##### 示例 2-10\. [将网页数据集转换为单词](https://oreil.ly/perip)

```py
words = pages.flat_map(lambda x: x.split(" ")).map(lambda w: (w, 1))
grouped_words = words.groupby(lambda wc: wc[0])
```

当你需要超越内置操作时，Ray 支持自定义聚合，只要你实现其接口。我们将更详细地讨论数据集，包括聚合函数，在第九章中。

###### 注意

Ray 对其数据集 API 使用*阻塞评估*。当您在 Ray 数据集上调用函数时，它将等待直到完成结果，而不是返回一个 future。Ray 核心 API 的其余部分使用 futures。

如果您希望拥有功能齐全的 DataFrame API，可以将您的 Ray 数据集转换为 Dask。第九章详细介绍了如何使用 Dask 进行更复杂的操作。如果您想了解更多关于 Dask 的信息，请查看[*Scaling Python with Dask*](https://oreil.ly/LKMlO)（O’Reilly），由 Holden 与 Mika Kimmins 共同撰写。

## Actor Hello World

Ray 的一个独特之处在于它强调 Actor。Actor 为您提供管理执行状态的工具，这是扩展系统中较具挑战性的部分之一。Actor 发送和接收消息，根据响应更新其状态。这些消息可以来自其他 Actor、程序，或者您的主执行线程与 Ray 客户端。

对于每个演员，Ray 启动一个专用进程。每个演员都有一个等待处理的消息邮箱。当您调用一个演员时，Ray 将一条消息添加到相应的邮箱中，从而允许 Ray 序列化消息处理，从而避免昂贵的分布式锁。演员可以在响应消息时返回值，因此当您向演员发送消息时，Ray 立即返回一个未来对象，以便在演员处理完您的消息后获取该值。

Ray 演员的创建和调用方式与远程函数类似，但使用 Python 类来实现，这使得演员有一个存储状态的地方。您可以通过修改经典的“你好世界”示例来看到它的运行过程，按顺序向您问候，如示例 2-11 所示。

##### 示例 2-11\. [演员你好世界](https://oreil.ly/perip)

```py
@ray.remote
class HelloWorld(object):
    def __init__(self):
        self.value = 0
    def greet(self):
        self.value += 1
        return f"Hi user #{self.value}"

# Make an instance of the actor
hello_actor = HelloWorld.remote()

# Call the actor
print(ray.get(hello_actor.greet.remote()))
print(ray.get(hello_actor.greet.remote()))
```

该示例相当基础；它缺乏任何容错性或每个演员内部的并发性。我们将在第四章中更深入地探讨这些内容。

# 结论

在本章中，您已经在本地机器上安装了 Ray，并使用了其许多核心 API。大多数情况下，您可以继续在本地模式下运行我们为本书选择的示例。当然，本地模式可能会限制您的规模或运行时间更长。

在接下来的章节中，我们将深入探讨 Ray 背后的一些核心概念。其中一个概念（容错性）更容易通过集群或云来进行说明。因此，如果您可以访问云账户或集群，现在是跳转到附录 B 并查看部署选项的绝佳时机。

¹ 随着 ARM 的普及，Ray 更有可能添加 ARM 版本的安装包，所以这只是暂时的情况。

² 演员仍然比无锁远程函数更昂贵，后者可以进行水平扩展。例如，许多工作进程调用同一个演员更新模型权重仍然比尴尬并行操作慢。
