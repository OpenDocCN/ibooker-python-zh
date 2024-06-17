# 第七章：实现微服务

最初，Ray 被创建为一个实现 [强化学习](https://oreil.ly/xRIXk) 的框架，但逐渐演变成了一个完整的无服务器平台。同样地，最初作为一种 [更好的机器学习模型服务方式](https://oreil.ly/tFneS) 被引入的 [Ray Serve](https://oreil.ly/970jH) 最近演变成了一个完整的微服务框架。在本章中，你将学习如何使用 Ray Serve 来实现一个通用的微服务框架，以及如何使用该框架进行模型服务。

本章中使用的所有示例的完整代码可以在书籍的 GitHub 存储库的文件夹 [*/ray_examples/serving*](https://oreil.ly/truUQ) 中找到。

# 在 Ray 中理解微服务架构

Ray 微服务架构（Ray Serve）是基于 [Ray actors](https://oreil.ly/12pG5) 来实现的，通过利用 Ray 实现。有三种类型的 actors 被创建来组成一个 Serve 实例：

控制器

Serve 实例中的一个全局 actor，负责管理控制平面。它负责创建、更新和销毁其他 actors。所有 Serve API 调用（例如，创建或获取一个部署）都使用控制器进行执行。

路由器

每个节点有一个路由器。每个路由器是一个 [Uvicorn HTTP 服务器](https://oreil.ly/IexLX)，它接受传入的请求，将它们转发给副本，然后在它们完成后响应。

Worker 副本

Worker 副本根据请求执行用户定义的代码。每个副本从路由器处理单独的请求。

用户定义的代码使用 Ray [deployment](https://oreil.ly/6UnLe) 实现，这是 Ray actor 的一个扩展，具有额外的功能。我们将从检查部署本身开始。

## 部署

Ray Serve 中的核心概念是部署，定义将处理传入请求的业务逻辑以及这个逻辑在 HTTP 或 Python 中的暴露方式。让我们从一个简单的部署开始，实现一个温度控制器（示例 7-1）。

##### 示例 7-1\. [温度控制器部署](https://oreil.ly/8pbXQ)

```py
@serve.deployment
class Converter:
    def __call__(self, request):
        if request.query_params["type"] == 'CF' :
            return {"Fahrenheit temperature":
                        9.0/5.0 * float(request.query_params["temp"]) + 32.0}
        elif request.query_params["type"] == 'FC' :
            return {"Celsius temperature":
                        (float(request.query_params["temp"]) - 32.0) * 5.0/9.0 }
        else:
            return {"Unknown conversion code" : request.query_params["type"]}

Converter.deploy()
```

实现是通过 `@serve.deployment` 注解进行装饰的，告诉 Ray 这是一个部署。这个部署实现了一个名为 `call` 的单一方法，这个方法在部署中具有特殊的意义：它通过 HTTP 被调用。它是一个类方法，接受一个 [`starlette` 请求](https://oreil.ly/F76fs)，它为传入的 HTTP 请求提供了一个方便的接口。在温度控制器的情况下，请求包含两个参数：温度和转换类型。

定义部署后，你需要使用 `Converter.deploy` 进行部署，类似于在部署 actor 时使用 `.remote`。然后你可以立即通过 HTTP 接口访问它（示例 7-2）。

##### 示例 7-2\. [通过 HTTP 访问转换器](https://oreil.ly/8pbXQ)

```py
print(requests.get("http://127.0.0.1:8000/Converter?temp=100.0&type=CF").text)
print(requests.get("http://127.0.0.1:8000/Converter?temp=100.0&type=FC").text)
print(requests.get("http://127.0.0.1:8000/Converter?temp=100.0&type=CC").text)
```

注意这里我们使用 URL 参数（查询字符串）来指定参数。此外，由于服务通过 HTTP 对外开放，请求者可以运行在任何地方，包括在 Ray 外部运行的代码中。

示例 7-3 显示了此调用的结果。

##### 示例 7-3\. 部署的 HTTP 调用结果

```py
{
  "Fahrenheit temperature": 212.0
}
{
  "Celsius temperature": 37.77777777777778
}
{
  "Unknown conversion code": "CC"
}
```

除了能够通过 HTTP 调用部署之外，还可以直接使用 Python 进行调用。要做到这一点，您需要获取到部署的 *handle* 然后使用它进行调用，如 示例 7-4 所示。

##### 示例 7-4\. [通过句柄调用部署](https://oreil.ly/8pbXQ)

```py
from starlette.requests import Request
handle = serve.get_deployment('Converter').get_handle()

print(ray.get(handle.remote(Request(
{"type": "http", "query_string": b"temp=100.0&type=CF"}))))
print(ray.get(handle.remote(Request(
{"type": "http", "query_string": b"temp=100.0&type=FC"}))))
print(ray.get(handle.remote(Request(
{"type": "http", "query_string": b"temp=100.0&type=CC"}))))
```

请注意，在此代码中，我们通过指定请求类型和查询字符串手动创建 `starlette` 请求。

一旦执行，此代码将返回与 示例 7-3 中相同的结果。此示例同时为 HTTP 和 Python 请求使用相同的 `call` 方法。虽然这样做是可行的，但[最佳实践](https://oreil.ly/LqkPK)是为 Python 调用实现额外的方法，以避免在 Python 调用中使用 `Request` 对象。在我们的示例中，我们可以在 示例 7-1 的初始部署中扩展用于 Python 调用的额外方法，在 示例 7-5 中实现。

##### 示例 7-5\. [为 Python 调用实现额外方法](https://oreil.ly/8pbXQ)

```py
@serve.deployment
class Converter:
    def __call__(self, request):
        if request.query_params["type"] == 'CF' :
            return {"Fahrenheit temperature":
                        9.0/5.0 * float(request.query_params["temp"]) + 32.0}
        elif request.query_params["type"] == 'FC' :
            return {"Celsius temperature":
                        (float(request.query_params["temp"]) - 32.0) * 5.0/9.0 }
        else:
            return {"Unknown conversion code" : request.query_params["type"]}
    def celcius_fahrenheit(self, temp):
        return 9.0/5.0 * temp + 32.0

    def fahrenheit_celcius(self, temp):
        return (temp - 32.0) * 5.0/9.0

Converter.deploy()
# list current deploymente
print(serve.list_deployments())
```

有了这些额外的方法，Python 调用可以显著简化（参见 示例 7-6）。

##### 示例 7-6\. [使用基于句柄的额外方法进行调用](https://oreil.ly/8pbXQ)

```py
print(ray.get(handle.celcius_fahrenheit.remote(100.0)))
print(ray.get(handle.fahrenheit_celcius.remote(100.0)))
```

与 示例 7-4 使用默认的 `call` 方法不同，这些调用方法是显式指定的（而不是将请求类型放在请求本身，这里请求类型是隐式的—​它是一个方法名）。

Ray 提供同步和异步句柄。一个 *sync* 标志, `Deployment.get​_han⁠dle(…​, sync=True|False)`，可用于指定句柄类型：

+   默认句柄是同步的。在这种情况下，调用 `handle.remote` 将返回一个 Ray 的 `ObjectRef`。

+   要创建一个异步句柄，请设置 `sync=False`。如其名所示，异步句柄调用是异步的，您需要使用 `await` 来获得一个 Ray 的 `ObjectRef`。要使用 `await`，您必须在 Python `asyncio` 事件循环中运行 `deployment.get_handle` 和 `handle.remote`。

我们将在本章后面演示异步句柄的使用。

最后，可以通过简单修改代码或配置选项并再次调用 `deploy` 来更新部署。除了在此处描述的 HTTP 和直接 Python 调用之外，还可以使用 Python API 来使用 Kafka 调用部署（参见 第六章 中的 Kafka 集成方法）。

现在您已了解部署的基础知识，让我们看看可用于部署的额外能力。

## 额外的部署能力

附加的部署能力以三种方式提供：

+   向注释添加参数

+   使用 FastAPI 进行 HTTP 部署

+   使用部署组合

当然，您可以结合这三种方法来实现您的目标。让我们仔细看看每种方法提供的选项。

### 向注释添加参数

`@serve.deployment` 注释可以接受几个 [参数](https://oreil.ly/mcUML)。其中最常用的是副本数和资源需求。

### 使用资源副本提高可扩展性

默认情况下，`deployment.deploy` 创建部署的单个实例。通过在 `@serve.deployment` 中指定副本数，您可以将部署扩展到多个进程。当请求发送到这样一个复制的部署时，Ray 使用轮询调度来调用各个副本。您可以修改 示例 7-1 来添加多个副本和每个实例的 ID（示例 7-7）。

##### 示例 7-7\. [扩展部署](https://oreil.ly/zImkU)

```py
@serve.deployment(num_replicas=3)
class Converter:
    def __init__(self):
        from uuid import uuid4
        self.id = str(uuid4())
    def __call__(self, request):
        if request.query_params["type"] == 'CF' :
            return {"Deployment": self.id, "Fahrenheit temperature":
                9.0/5.0 * float(request.query_params["temp"]) + 32.0}
        elif request.query_params["type"] == 'FC' :
            return {"Deployment": self.id, "Celsius temperature":
                (float(request.query_params["temp"]) - 32.0) * 5.0/9.0 }
        else:
            return {"Deployment": self.id, "Unknown conversion code":
                request.query_params["type"]}
    def celcius_fahrenheit(self, temp):
        return 9.0/5.0 * temp + 32.0

    def fahrenheit_celcius(self, temp):
        return (temp - 32.0) * 5.0/9.0

Converter.deploy()
# list current deployments
print(serve.list_deployments())
```

现在使用 HTTP 或基于句柄的调用会在 示例 7-8 中生成结果。

##### 示例 7-8\. 调用扩展部署

```py
{'Deployment': '1d...0d', 'Fahrenheit temperature': 212.0}
{'Deployment': '4d...b9', 'Celsius temperature': 37.8}
{'Deployment': '00...aa', 'Unknown conversion code': 'CC'}
```

查看此结果，您可以看到每个请求都由不同的部署实例（不同的 ID）处理。

这是部署的手动扩展。那么自动扩展呢？与 Kafka 监听器的自动扩展类似（见 第六章 讨论），Ray 的自动扩展方法与 Kubernetes 本地方法不同—​例如，请参见 [Knative](https://oreil.ly/2Tj0l)。Ray 的自动扩展方法不是创建新实例，而是创建更多 Ray 节点，并适当重新分配部署。

如果您的部署开始超过每秒大约三千个请求，您也应该将 HTTP 入口扩展到 Ray。默认情况下，入口 HTTP 服务器仅在主节点上启动，但您也可以使用 `serve.start(http_options=\{"location": "EveryNode"})` 在每个节点上启动一个 HTTP 服务器。如果您扩展了 HTTP 入口的数量，您还需要部署一个负载均衡器，可以从您的云提供商获取或在本地安装。

### 部署的资源需求

您可以在 `@serve.deployment` 中请求特定的资源需求。例如，两个 CPU 和一个 GPU 可以如下表示：

```py
@serve.deployment(ray_actor_options={"num_cpus": 2, "num_gpus": 1})
```

`@serve.deployment` 的另一个有用参数是 `route_prefix`。如您在 示例 7-2 中所见，默认前缀是用于此部署的 Python 类的名称。例如，使用 `route_prefix` 允许您显式指定 HTTP 请求使用的前缀：

```py
@serve.deployment(route_prefix="/converter")
```

有关附加配置参数的描述，请参阅 [“Ray 核心 API” 文档](https://oreil.ly/3NWgQ)。

### 使用 FastAPI 实现请求路由

尽管在示例 7-1 中初始的温度转换器部署工作正常，但使用起来并不方便。您需要在每个请求中指定转换类型。更好的方法是为 API 拥有两个单独的端点（URL）——一个用于摄氏度转华氏度的转换，另一个用于华氏度转摄氏度的转换。您可以利用[FastAPI](https://oreil.ly/h8QKh)与[Serve 集成](https://oreil.ly/ue4Mz)来实现这一点。通过这种方式，您可以像在示例 7-9 中所示重写示例 7-1。

##### 示例 7-9\. [在部署中实现多个 HTTP API](https://oreil.ly/OuurE)

```py
@serve.deployment(route_prefix="/converter")
@serve.ingress(app)
class Converter:
    @app.get("/cf")
    def celcius_fahrenheit(self, temp):
        return {"Fahrenheit temperature": 9.0/5.0 * float(temp) + 32.0}

    @app.get("/fc")
    def fahrenheit_celcius(self, temp):
        return {"Celsius temperature": (float(temp) - 32.0) * 5.0/9.0}
```

注意，在这里，我们引入了两个具有不同 URL 的 HTTP 可访问 API（实际上将第二个查询字符串参数转换为一组 URL），每种转换类型一个。这可以简化 HTTP 访问；将示例 7-10 与示例 7-2 进行比较。

##### 示例 7-10\. [使用多个 HTTP 端点调用部署](https://oreil.ly/OuurE)

```py
print(requests.get("http://127.0.0.1:8000/converter/cf?temp=100.0&").text)
print(requests.get("http://127.0.0.1:8000/converter/fc?temp=100.0").text)
```

通过 FastAPI 实现提供的附加功能包括可变路由、自动类型验证、依赖注入（例如数据库连接）、[安全支持](https://oreil.ly/atwGv)等。请参阅[FastAPI 文档](https://oreil.ly/3pw0k)了解如何使用这些功能。

## 部署组合

部署可以构建为其他部署的组合。这允许构建强大的部署流水线。

让我们看一个具体的例子：[金丝雀部署](https://oreil.ly/YT27x)。在这种部署策略中，您会以有限的方式部署您的代码或模型的新版本，以查看其行为如何。您可以通过使用部署组合轻松构建这种类型的部署。我们将从定义并部署两个简单的部署开始，在示例 7-11 中。

##### 示例 7-11\. [两个基本部署](https://oreil.ly/EaD6e)

```py
@serve.deployment
def version_one(data):
    return {"result": "version1"}

version_one.deploy()

@serve.deployment
def version_two(data):
    return {"result": "version2"}

version_two.deploy()
```

这些部署接受任何数据并返回一个字符串："result": "version1"用于部署 1 和"result": “version2"用于部署 2。您可以通过实现金丝雀部署（示例 7-12）将这两个部署组合起来。

##### 示例 7-12\. [金丝雀部署](https://oreil.ly/EaD6e)

```py
@serve.deployment(route_prefix="/versioned")
class Canary:
    def __init__(self, canary_percent):
        from random import random
        self.version_one = version_one.get_handle()
        self.version_two = version_two.get_handle()
        self.canary_percent = canary_percent

    # This method can be called concurrently!
    async def __call__(self, request):
        data = await request.body()
        if(random() < self.canary_percent):
            return await self.version_one.remote(data=data)
        else:
            return await self.version_two.remote(data=data)
```

此部署演示了几个要点。首先，它展示了具有参数的构造函数，这对部署非常有用，允许单个定义使用不同的参数进行部署。其次，我们将`call`函数定义为`async`，以便并发处理查询。`call`函数的实现很简单：生成一个新的随机数，并根据其值和`canary_percent`的值调用版本 1 或版本 2 的部署。

一旦通过`Canary.deploy(.3)`部署了`Canary`类，你可以使用 HTTP 调用它。调用 canary 部署 10 次的结果显示在示例 7-13 中。

##### 示例 7-13\. Canary 部署调用的结果

```py
{'result': 'version2'}
{'result': 'version2'}
{'result': 'version1'}
{'result': 'version2'}
{'result': 'version1'}
{'result': 'version2'}
{'result': 'version2'}
{'result': 'version1'}
{'result': 'version2'}
{'result': 'version2'}
```

正如你在这里看到的，Canary 模型运行得相当好，并且完全符合你的需求。现在你知道如何构建和使用基于 Ray 的微服务之后，让我们看看如何将它们用于模型服务。

# 使用 Ray Serve 进行模型服务

简言之，服务模型与调用任何其他微服务没有什么不同（我们将在本章后面讨论特定的模型服务需求）。只要你能以某种形式获取与 Ray 运行时兼容的 ML 生成模型—例如，以[pickle 格式](https://oreil.ly/DWVE7)，纯 Python 代码或二进制格式以及其处理的 Python 库—你就可以使用这个模型处理推断请求。让我们从一个简单的模型服务示例开始。

## 简单的模型服务示例

一个流行的模型学习应用是基于[Kaggle 红葡萄酒质量数据集](https://oreil.ly/yvPRp)预测红酒的质量。许多博客文章使用这个数据集来构建红酒质量的机器学习实现—例如，参见[Mayur Badole](https://oreil.ly/9yLZ9)和[Dexter Nguyen](https://oreil.ly/JWwgO)的文章。在我们的例子中，我们基于 Terence Shin 的[“使用多种分类技术预测葡萄酒质量”](https://oreil.ly/M6lc4)建立了几个红葡萄酒质量分类模型；实际代码可以在书的[GitHub 仓库](https://oreil.ly/ChZtD)找到。该代码使用多种技术来构建红葡萄酒质量的分类模型，包括以下内容：

+   [决策树](https://oreil.ly/Qnx8W)

+   [随机森林](https://oreil.ly/yXqZz)

+   [AdaBoost](https://oreil.ly/gerOD)

+   [梯度提升](https://oreil.ly/bTNNZ)

+   [XGBoost](https://oreil.ly/csAzq)

所有的实现都利用了 scikit-learn Python 库，允许你生成模型并使用 pickle 导出。在验证模型时，我们从随机森林、梯度提升和 XGBoost 分类中看到了最佳结果，所以我们只保存了这些模型—生成的模型可以在书的[GitHub 仓库](https://oreil.ly/VY9NE)找到。有了这些模型，你可以使用一个简单的部署来服务红葡萄酒质量模型，使用随机森林分类（示例 7-14）。

##### 示例 7-14\. [使用随机森林分类实现模型服务](https://oreil.ly/52qR4)

```py
@serve.deployment(route_prefix="/randomforest")
class RandomForestModel:
    def __init__(self, path):
        with open(path, "rb") as f:
            self.model = pickle.load(f)
    async def __call__(self, request):
        payload = await request.json()
        return self.serve(payload)

    def serve(self, request):
        input_vector = [
            request["fixed acidity"],
            request["volatile acidity"],
            request["citric acid"],
            request["residual sugar"],
            request["chlorides"],
            request["free sulfur dioxide"],
            request["total sulfur dioxide"],
            request["density"],
            request["pH"],
            request["sulphates"],
            request["alcohol"],
        ]
        prediction = self.model.predict([input_vector])[0]
        return {"result": str(prediction)}
```

这个部署有三个方法：

构造函数

加载模型并在本地存储。我们使用模型位置作为参数，这样当模型变化时可以重新部署这个部署。

`call`

通过 HTTP 请求调用，此方法检索特征（作为字典）并调用`serve`方法进行实际处理。通过定义为异步，它可以同时处理多个请求。

`serve`

可用于通过句柄调用部署。它将传入的字典转换为向量，并调用底层模型进行推断。

一旦实施部署，它可以用于模型服务。如果通过 HTTP 调用，它将以 JSON 字符串形式接收负载；对于直接调用，请求是一个字典形式。对于[XGBoost](https://oreil.ly/Kc2oH)和[gradient boost](https://oreil.ly/UbrD7)的实现看起来几乎相同，唯一的区别是在这些情况下生成的模型需要一个二维数组而不是向量，因此您需要在调用模型之前进行此转换。

此外，您还可以查看 Ray 的文档，用于[serving other types of models](https://oreil.ly/qouwp)，包括 TensorFlow 和 PyTorch。

现在您知道如何构建一个简单的模型服务实现，问题是基于 Ray 的微服务是否是模型服务的良好平台。

## 模型服务实现的考虑事项

在模型服务方面，有几个具体要求非常重要。可以在《[*Kubeflow for Machine Learning*](https://oreil.ly/sikPG)》中找到关于模型服务特定要求的良好定义，作者是 Trevor Grant 等人（O’Reilly）。这些要求如下：

1.  实现必须灵活。它应该允许您的训练是实现无关的（即 TensorFlow 与 PyTorch 与 scikit-learn）。对于推断服务调用，不应关心底层模型是使用 PyTorch、scikit-learn 还是 TensorFlow 训练的：服务接口应该是共享的，以便用户的 API 保持一致。

1.  为了实现更好的吞吐量，有时可以在各种设置中对请求进行批处理。实现应该简化支持模型服务请求的批处理。

1.  实现应该提供利用与算法需求匹配的硬件优化器的能力。在评估阶段，有时您会受益于像 GPU 这样的硬件优化器来推断模型。

1.  实现应能够无缝地包含推断图的其他组件。推断图可以包括特征转换器、预测器、解释器和漂移检测器。

1.  实现应该允许服务实例的扩展，无论底层硬件如何，都可以明确地使用自动缩放器。

1.  应该能够通过包括 HTTP 和 Kafka 在内的不同协议公开模型服务功能。

1.  传统的 ML 模型在训练数据分布之外通常表现不佳。因此，如果发生数据漂移，模型性能可能会下降，需要重新训练和重新部署。实现应支持模型的轻松重新部署。

1.  灵活的部署策略实现（包括金丝雀部署、蓝绿部署和 A/B 测试）是必需的，以确保新版本的模型不会表现比现有版本更差。

让我们看看 Ray 微服务框架是如何满足这些需求的：

1.  Ray 的部署清晰地将部署 API 与模型 API 分开。因此，Ray “标准化”了部署 API，并提供了将传入数据转换为模型所需格式的支持。参见 示例 7-14。

1.  Ray 的部署使得实现请求批处理变得简单。有关如何实现和部署接受批处理、配置批处理大小并在 Python 中查询模型的详细信息，请参阅 Ray [“批处理教程”指南](https://oreil.ly/K3up4)。

1.  如本章前文所述，部署支持配置，允许指定执行所需的硬件资源（CPU/GPU）。

1.  如本章前文所述，部署组合允许轻松创建模型服务图，混合和匹配纯 Python 代码和现有部署。我们将在本章后面再介绍一个部署组合的示例。

1.  如本章前文所述，部署支持设置副本数量，因此可以轻松扩展部署。结合 Ray 的自动缩放功能和定义 HTTP 服务器数量的能力，微服务框架允许非常高效地扩展模型服务。

1.  如前文所述，部署可以通过 HTTP 或纯 Python 公开。后一选项允许与任何所需传输进行集成。

1.  如本章前文所述，部署的简单重新部署允许更新模型而无需重新启动 Ray 集群和中断正在利用模型服务的应用程序。

1.  如 示例 7-12 所示，使用部署组合可轻松实现任何部署策略。

如我们在这里展示的，Ray 微服务框架是满足模型服务的所有主要需求的坚实基础。

本章最后要学习的是一种高级模型服务技术的实现——[推测性模型服务](https://oreil.ly/KH8EZ)，使用 Ray 微服务框架。

## 使用 Ray 微服务框架进行推测性模型服务

推测模型服务是 [推测执行](https://oreil.ly/RRvzK) 的一种应用。在这种优化技术中，计算机系统执行可能不需要的任务。在确切知道是否真正需要之前，工作已经完成。这样可以提前获取结果，因此如果确实需要，将无延迟可用。在模型服务中，推测执行很重要，因为它为机器服务应用程序提供以下功能：

保证执行时间

假设您有多个模型，并且最快的提供固定的执行时间，那么可以提供一个模型服务实现，其执行时间上限是固定的，只要该时间大于最简单模型的执行时间。

基于共识的模型服务

假设您有多个模型，您可以实现模型服务，使得预测结果是多数模型返回的结果。

基于质量的模型服务

假设您有一个评估模型服务结果质量的度量标准，这种方法允许您选择质量最佳的结果。

在这里，您将学习如何使用 Ray 的微服务框架实现基于共识的模型服务。

在本章的早些时候，您学习了如何使用三个模型（随机森林、梯度提升和 XGBoost）实现红葡萄酒的质量评分。现在让我们尝试生成一个实现，返回至少两个模型同意的结果。基本实现如 示例 7-15 所示。

##### 示例 7-15\. [基于共识的模型服务](https://oreil.ly/2RyYu)

```py
@serve.deployment(route_prefix="/speculative")
class Speculative:
    def __init__(self):
        self.rfhandle = RandomForestModel.get_handle(sync=False)
        self.xgboosthandle = XGBoostModel.get_handle(sync=False)
        self.grboosthandle = GRBoostModel.get_handle(sync=False)
    async def __call__(self, request):
        payload = await request.json()
        f1, f2, f3 = await asyncio.gather(self.rfhandle.serve.remote(payload),
                self.xgboosthandle.serve.remote(payload), 
                self.grboosthandle.serve.remote(payload))

        rfresurlt = ray.get(f1)['result']
        xgresurlt = ray.get(f2)['result']
        grresult = ray.get(f3)['result']
        ones = []
        zeros = []
        if rfresurlt == "1":
            ones.append("Random forest")
        else:
            zeros.append("Random forest")
        if xgresurlt == "1":
            ones.append("XGBoost")
        else:
            zeros.append("XGBoost")
        if grresult == "1":
            ones.append("Gradient boost")
        else:
            zeros.append("Gradient boost")
        if len(ones) >= 2:
            return {"result": "1", "methods": ones}
        else:
            return {"result": "0", "methods": zeros}
```

此部署的构造函数为所有实现个别模型的部署创建句柄。请注意，这里我们创建的是异步句柄，允许并行执行每个部署。

`call` 方法获取有效载荷并开始并行执行所有三个模型，然后等待所有模型执行完毕。有关使用 `asyncio` 在执行多个协程和并行运行它们时的信息，请参见 Hynek Schlawack 的 [“在 asyncio 中等待”](https://oreil.ly/pKovE)。一旦您获得所有结果，就实施共识计算并返回结果（以及投票支持它的方法）¹。

# 结论

在本章中，你学习了 Ray 实现的微服务框架以及这个框架如何被模型服务所使用。我们从描述基本的微服务部署开始，并介绍了一些扩展，允许更好地控制、扩展和扩展部署的执行。然后，我们展示了一个示例，说明了这个框架如何被用来实现模型服务，分析了典型的模型服务需求，并展示了 Ray 如何满足这些需求。最后，你学习了如何实现一个高级的模型服务示例——基于共识的模型服务——使你能够提高个别模型服务方法的质量。文章["在蚂蚁集团的 Ray 上构建高可用和可扩展的在线应用"](https://oreil.ly/y9BuV)由蔡腾伟等人展示了如何将这里描述的基本构建块汇集到更复杂的实现中。

在下一章中，你将学习有关在 Ray 中实现工作流以及如何使用工作流来自动化你的应用程序执行的内容。

¹ 你还可以实现不同的策略来等待模型的执行。例如，你可以通过`asyncio.wait(tasks). return_when=asyncio.FIRST_COMPLETED)`至少使用一个模型的结果，或者仅通过`asyncio.wait(tasks, interval)`等待一段给定的时间间隔。
