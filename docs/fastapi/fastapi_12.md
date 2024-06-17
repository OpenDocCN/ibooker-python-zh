# 第九章：服务层

> 那个中间的东西是什么？
> 
> 奥托·韦斯特，《一条名为万达的鱼》

# 预览

本章扩展了服务层——中间层。一个漏水的屋顶可能会花费很多钱。漏水的软件不那么明显，但会花费大量时间和精力。你应该如何构建你的应用程序，以避免层间的泄漏？特别是，什么应该放入服务层中，什么不应该？

# 定义一个服务

服务层是网站的核心，它存在的原因。它接受来自多个来源的请求，访问构成网站 DNA 的数据，并返回响应。

常见的服务模式包括以下组合：

+   创建 / 检索 / 修改（部分或完全）/ 删除

+   一个东西 / 多个东西

在 RESTful 路由器层，名词是*资源*。在本书中，我们的资源最初将包括神秘动物（虚构生物）和人物（神秘动物探险者）。

稍后，可以定义类似这些的相关资源：

+   地点

+   事件（例如，探险、目击）

# 布局

这里是当前的文件和目录布局：

```py
main.py
web
├── __init__.py
├── creature.py
├── explorer.py
service
├── __init__.py
├── creature.py
├── explorer.py
data
├── __init__.py
├── creature.py
├── explorer.py
model
├── __init__.py
├── creature.py
├── explorer.py
fake
├── __init__.py
├── creature.py
├── explorer.py
└── test
```

在本章中，你将会操作*service*目录中的文件。

# 保护

层次结构的一个好处是你不必担心一切。服务层只关心数据的输入和输出。正如你将在第十一章中看到的，一个更高层次（在本书中是*Web*）可以处理认证和授权的混乱问题。创建、修改和删除的功能不应该是完全开放的，甚至`get`函数最终可能也需要一些限制。

# 函数

让我们从*creature.py*开始。此时，*explorer.py*的需求几乎相同，我们几乎可以借用所有内容。编写一个处理两者的单一服务文件是如此诱人，但几乎不可避免地，我们最终会需要不同方式处理它们。

此时的服务文件基本上是一个透传层。这是一个情况，在这种情况下，开始时稍微多做一些结构化工作会在后面得到回报。就像你在第八章中为*web/creature.py*和*web/explorer.py*所做的那样，你将为两者定义服务模块，并暂时将它们都连接到相应的*fake*数据模块（示例 9-1 和 9-2）。

##### 示例 9-1\. 初始的 service/creature.py 文件

```py
from models.creature import Creature
import fake.creature as data

def get_all() -> list[Creature]:
    return data.get_all()

def get_one(name: str) -> Creature | None:
    return data.get(id)

def create(creature: Creature) -> Creature:
    return data.create(creature)

def replace(id, creature: Creature) -> Creature:
    return data.replace(id, creature)

def modify(id, creature: Creature) -> Creature:
    return data.modify(id, creature)

def delete(id, creature: Creature) -> bool:
    return data.delete(id)
```

##### 示例 9-2\. 初始的 service/explorer.py 文件

```py
from models.explorer import Explorer
import fake.explorer as data

def get_all() -> list[Explorer]:
    return data.get_all()

def get_one(name: str) -> Explorer | None:
    return data.get(name)

def create(explorer: Explorer) -> Explorer:
    return data.create(explorer)

def replace(id, explorer: Explorer) -> Explorer:
    return data.replace(id, explorer)

def modify(id, explorer: Explorer) -> Explorer:
    return data.modify(id, explorer)

def delete(id, explorer: Explorer) -> bool:
    return data.delete(id)
```

###### 提示

`get_one()`函数返回值的语法（`Creature | None`）至少需要 Python 3.9。对于早期版本，你需要`Optional`：

```py
from typing import Optional
...
def get_one(name: str) -> Optional[Creature]:
...
```

# 测试！

现在代码库逐渐完善，是引入自动化测试的好时机。（前一章的 Web 测试都是手动测试。）因此，让我们创建一些目录：

测试

一个顶层目录，与*web*、*service*、*data*和*model*并列。

单元

练习单个函数，但不要跨层边界。

web

Web 层单元测试。

service

服务层单元测试。

数据

数据层单元测试。

完整

Also known as *end-to-end* or *contract* tests, these span all layers at once. They address the API endpoints in the Web layer.

The directories have the *test_* prefix or *_test* suffix for use by pytest, which you’ll start to see in Example 9-4 (which runs the test in Example 9-3).

Before testing, a few API design choices need to be made. What should be returned by the `get_one()` function if a matching `Creature` or `Explorer` isn’t found? You can return `None`, as in Example 9-2. Or you could raise an exception. None of the built-in Python exception types deal directly with missing values:

+   `TypeError` may be the closest, because `None` is a different type than `Creature`.

+   `ValueError` is more suited for the wrong value for a given type, but I guess you could say that passing a missing string `id` to `get_one(id)` qualifies.

+   You could define your own `MissingError` if you really want to.

Whichever method you choose, the effects will bubble up all the way to the top layer.

Let’s go with the `None` alternative rather than the exception for now. After all, that’s what *none* means. Example 9-3 is a test.

##### Example 9-3\. Service test test/unit/service/test_creature.py

```py
from model.creature import Creature
from service import creature as code

sample = Creature(name="yeti",
        country="CN",
        area="Himalayas",
        description="Hirsute Himalayan",
        aka="Abominable Snowman",
        )

def test_create():
    resp = code.create(sample)
    assert resp == sample

def test_get_exists():
    resp = code.get_one("yeti")
    assert resp == sample

def test_get_missing():
    resp = code.get_one("boxturtle")
    assert data is None
```

Run the test in Example 9-4.

##### Example 9-4\. Run the service test

```py
$ pytest -v test/unit/service/test_creature.py
test_creature.py::test_create PASSED                         [ 16%]
test_creature.py::test_get_exists PASSED                     [ 50%]
test_creature.py::test_get_missing PASSED                    [ 66%]

======================== 3 passed in 0.06s =========================
```

###### Note

In Chapter 10, `get_one()` will no longer return `None` for a missing creature, and the `test_get_missing()` test in Example 9-4 would fail. But that will be fixed.

# Other Service-Level Stuff

We’re in the middle of the stack now—the part that really defines our site’s purpose. And so far, we’ve used it only to forward web requests to the (next chapter’s) Data layer.

So far, this book has developed the site iteratively, building a minimal base for future work. As you learn more about what you have, what you can do, and what users might want, you can branch out and experiment. Some ideas might benefit only larger sites, but here are some technical site-helper ideas:

+   Logging

+   Metrics

+   Monitoring

+   Tracing

This section discusses each of these. We’ll revisit these options in “Troubleshooting”, to see if they can help diagnose problems.

## Logging

FastAPI logs each API call to an endpoint—including the timestamp, method, and URL—but not any data delivered via the body or headers.

## Metrics, Monitoring, Observability

If you run a website, you probably want to know how it’s doing. For an API website, you might want to know which endpoints are being accessed, how many people are visiting, and so on. Statistics on such factors are called *metrics*, and the gathering of them is *monitoring* or *observability*.

Popular metrics tools nowadays include [Prometheus](https://prometheus.io) for gathering metrics and [Grafana](https://grafana.com) for displaying metrics.

## Tracing

网站表现如何？通常情况下，整体指标可能很好，但这里或那里的结果令人失望。或者整个网站可能一团糟。无论哪种情况，拥有一个工具来测量 API 调用的全过程时间是很有用的——不仅仅是总体时间，还包括每个中间步骤的时间。如果某些步骤很慢，你可以找到链条中的薄弱环节。这就是*追踪*。

一个新的开源项目已经将早期的追踪产品（如[Jaeger](https://www.jaegertracing.io)）打造成[OpenTelemetry](https://opentelemetry.io)。它具有[Python API](https://oreil.ly/gyL70)，并且至少与一个[FastAPI 的集成](https://oreil.ly/L6RXV)。

要使用 Python 安装和配置 OpenTelemetry，请按照[OpenTelemetry Python 文档](https://oreil.ly/MBgd5)中的说明操作。

## 其他

这些生产问题将在第十三章讨论。除此之外，还有我们的领域——神秘动物及其相关内容？除了探险家和生物的基本信息，还有什么其他事情可能需要你考虑？你可能会想出需要对模型和其他层进行更改的新想法。以下是一些你可以尝试的想法：

+   探险家与他们寻找的生物之间的链接

+   观测数据

+   探险

+   照片和视频

+   大脚杯子和 T 恤（见图 9-1)

![fapi 0901](img/fapi_0901.png)

###### 图 9-1\. 我们的赞助商发来的一句话

这些类别通常需要定义一个或多个新模型，并创建新的模块和函数。其中一些将会在第四部分中添加，这是一个基于第三部分构建的应用程序库。

# 回顾

在本章中，你复制了 Web 层的一些函数，并移动了它们所使用的虚假数据。目标是启动新的服务层。到目前为止，这一过程一直是标准化的，但在此之后将会发展和分歧。下一章将构建最终的数据层，使网站真正活跃起来。
