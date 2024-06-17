# 第八章。使用 Ray Serve 进行在线推断

Edward Oakes

# 在线推断

在之前的章节中，您已经学习了如何使用 Ray 处理数据、训练机器学习（ML）模型，并在批量推断设置中应用它们。然而，许多最令人兴奋的机器学习用例涉及“在线推断”：即使用 ML 模型增强用户直接或间接交互的 API 端点。在线推断在延迟至关重要的情况下非常重要：您不能简单地将模型应用于后台数据并提供结果。有许多现实世界的用例示例，其中在线推断可以提供很大的价值：

**推荐系统**：为产品（例如在线购物）或内容（例如社交媒体）提供推荐是机器学习的基础用例。虽然可以以离线方式执行此操作，但推荐系统通常受益于实时反应用户偏好。这要求使用最近的行为作为关键特征进行在线推断。

**聊天机器人**：在线服务通常具有实时聊天窗口，以便从键盘舒适地为客户提供支持。传统上，这些聊天窗口由客户支持人员负责，但最近的趋势是通过使用全天候在线的 ML 动力聊天机器人来减少劳动成本并改善解决时间。这些聊天机器人需要多种机器学习技术的复杂混合，并且必须能够实时响应客户输入。

**估计到达时间**：乘车共享、导航和食品配送服务都依赖于能够提供准确的到达时间估计（例如，为您的司机、您自己或您的晚餐）。提供准确的估算非常困难，因为它需要考虑交通模式、天气和事故等真实世界因素。估计通常在一次行程中多次刷新。

这些只是一些例子，应用机器学习以在线方式在传统上非常困难的应用领域中提供大量价值。应用领域的应用清单仍在继续扩展：一些新兴领域如自动驾驶汽车、机器人技术、视频处理管道也正在被机器学习重新定义。所有这些应用都共享一个关键特征：延迟至关重要。在在线服务的情况下，低延迟对提供良好的用户体验至关重要，对于涉及到实际世界的应用，如机器人技术或自动驾驶汽车，更高的延迟可能会在安全性或正确性方面产生更强的影响。

本章将介绍 Ray Serve，这是一个基于 Ray 的本地库，可以在 Ray 之上构建在线推理应用程序。首先，我们将讨论 Ray Serve 所解决的在线推理挑战。然后，我们将介绍 Ray Serve 的架构并介绍其核心功能集。最后，我们将使用 Ray Serve 构建一个由多个自然语言处理模型组成的端到端在线推理 API。

# 在线推理的挑战

在前一节中，我们讨论了在线推理的主要目标是与低延迟交互的机器学习模型。然而，这长期以来一直是 API 后端和 Web 服务器的关键要求，因此一个自然的问题是：在为机器学习模型提供服务方面有何不同？

## 1\. 机器学习模型具有计算密集性

许多在线推理的挑战都源于一个关键特征：机器学习模型非常计算密集。与传统的 Web 服务不同，传统的 Web 服务主要处理 I/O 密集型数据库查询或其他 API 调用，大多数机器学习模型归结为执行许多线性代数计算，无论是提供推荐、估计到达时间还是检测图像中的对象。对于最近的“深度学习”趋势尤其如此，单个模型执行的权重和计算数量随时间增长而增加。深度学习模型通常也可以从使用专用硬件（如 GPU 或 TPU）中获得显著好处，这些硬件具有为机器学习计算优化的专用指令，并且可以跨多个输入并行进行矢量化计算。

许多在线推理应用程序通常需要 24/7 运行。与机器学习模型计算密集性结合在一起时，运行在线推理服务可能非常昂贵，需要始终分配大量的 CPU 和 GPU。在线推理的主要挑战在于以最小化端到端延迟和降低成本的方式提供模型服务。在线推理系统提供了一些关键特性以满足这些要求：- 支持诸如 GPU 和 TPU 之类的专用硬件。- 能够根据请求负载调整模型使用的资源的能力。- 支持请求批处理以利用矢量化计算。

## 2\. 机器学习模型在孤立状态下并不实用

当讨论机器学习时，通常在学术或研究环境中，重点放在个体、孤立的任务上，比如对象识别或分类。然而，在真实世界的应用中，问题通常不是那么明确和定义明确的。相反，解决问题端到端需要结合多个机器学习模型和业务逻辑。例如，考虑一个产品推荐的使用案例。虽然我们可以应用多种已知的机器学习技术来解决推荐的核心问题，但在边缘处也存在许多同样重要的挑战，其中许多挑战对每个使用案例可能是特定的：

实施在线推理 API 需要能够将所有这些部分整合到一个统一的服务中。因此，有能力将多个模型与自定义业务逻辑组合在一起非常重要。这些部分实际上不能完全孤立地看待： "胶水" 逻辑通常需要随着模型本身的演变而演变。

# 介绍 Ray Serve

Ray Serve 是建立在 Ray 之上的可扩展计算层，用于为机器学习模型提供服务。Serve 是框架无关的，这意味着它不依赖于特定的机器学习库，而是将模型视为普通的 Python 代码。此外，它允许您灵活地将普通的 Python 业务逻辑与机器学习模型结合在一起。这使得完全端到端地构建在线推理服务成为可能：Serve 应用程序可以验证用户输入，查询数据库，在多个机器学习模型之间可扩展地执行推理，并在处理单个推理请求的过程中组合、过滤和验证输出。事实上，结合多个机器学习模型的结果是 Ray Serve 的关键优势之一，正如我们在后面章节中探讨的常见多模型服务模式所示。

Serve 虽然在性质上灵活，但为计算密集型的机器学习模型提供了专门构建的功能，可以实现动态扩展和资源分配，以确保可以有效地处理跨多个 CPU 和/或 GPU 的请求负载。在这里，Serve 从 Ray 构建中继承了许多好处：它可扩展到数百台机器，提供灵活的调度策略，并使用 Ray 的核心 API 在进程间提供低开销的通信。

本节将逐步介绍 Ray Serve 的核心功能，重点是它如何帮助解决上述在线推理的挑战。要跟随本节中的代码示例，您需要在本地安装以下 Python 包：

```py
pip install ray["serve"]==1.12.0 transformers requests
```

运行示例假设你在当前工作目录中已经保存了名为`app.py`的文件。

## 架构概述

Ray Serve 建立在 Ray 之上，因此它继承了许多优点，比如可扩展性、低开销的通信、适合并行的 API，以及通过对象存储利用共享内存的能力。Ray Serve 中的核心原语是**部署**，你可以把它看作是一组受管理的 Ray actor，可以一起使用并处理通过负载平衡分布在它们之间的请求。部署中的每个 actor 称为 Ray Serve 中的**副本**。通常，一个部署将一对一地映射到一个机器学习模型，但部署可以包含任意的 Python 代码，因此它们也可以包含业务逻辑。

Ray Serve 使部署可以通过 HTTP 公开，并定义输入解析和输出逻辑。但 Ray Serve 最重要的功能之一是，部署也可以直接使用本机 Python API 相互调用，这将转换为副本之间的直接 actor 调用。这使得模型和业务逻辑的灵活高性能组合成为可能；您将在本节后面看到这一点的实际应用。

在幕后，组成 Ray Serve 应用程序的部署由一个集中的**控制器**actor 管理。这是一个由 Ray 管理的独立 actor，将在失败时重新启动。控制器负责创建和更新副本 actor，向系统中的其他 actor 广播更新，并执行健康检查和故障恢复。如果由于任何原因副本或整个 Ray 节点崩溃，控制器将检测到失败，并确保 actor 被恢复并可以继续提供服务。

## 定义基本的 HTTP 端点

本节将通过定义一个简单的 HTTP 端点来介绍 Ray Serve。我们将部署的模型是一个情感分类器：给定一个文本输入，它将预测输出是否具有积极或消极的情感。我们将使用[Hugging Face](https://huggingface.co/) `transformers`库中的预训练情感分类器，该库提供了一个简单的 Python API，用于预训练模型，它将隐藏模型的细节，使我们可以专注于服务逻辑。要使用 Ray Serve 部署此模型，我们需要定义一个 Python 类，并使用`@serve.deployment`装饰器将其转换为 Serve 的**部署**。该装饰器允许我们传递多个有用的选项来配置部署；我们将在本章后面探讨其中的一些选项。

##### 示例 8-1。

```py
from ray import serve

from starlette.requests import Request
from transformers import pipeline

@serve.deployment
class SentimentAnalysis:
    def __init__(self):
        self._classifier = pipeline("sentiment-analysis")

    def __call__(self, request: Request) -> str:
        input_text = request.query_params["input_text"]
        return self._classifier(input_text)[0]["label"]
```

这里有几个重要的要点需要注意。首先，在类的构造函数中实例化我们的模型。这个模型可能非常庞大，因此下载和加载到内存中可能会很慢（可能需要 10 秒或更长时间）。在 Ray Serve 中，构造函数中的代码将仅在每个副本在启动时运行一次，并且任何属性都可以被缓存以供将来使用。其次，我们在 `__call__` 方法中定义处理请求的逻辑。这个方法以 `Starlette` 的 HTTP 请求作为输入，并可以返回任何可 JSON 序列化的输出。在本例中，我们将从模型的输出中返回一个字符串："POSITIVE" 或 "NEGATIVE"。

一旦一个部署被定义，我们使用 `.bind()` API 来实例化它的一个副本。这是我们可以传递给构造函数的可选参数来配置部署（例如远程路径以下载模型权重）。请注意，这实际上并没有运行部署，而只是将其与其参数打包在一起（当我们将多个模型组合在一起时，这将变得更加重要）。

##### 示例 8-2\.

```py
basic_deployment = SentimentAnalysis.bind()
```

我们可以使用 `serve.run` Python API 或相应的 `serve run` CLI 命令来运行绑定的部署。假设你将以上代码保存在一个名为 `app.py` 的文件中，你可以用以下命令在本地运行它：

```py
serve run app:basic_deployment
```

这将实例化我们部署的单个副本，并将其托管在本地 HTTP 服务器后面。要测试它，我们可以使用 Python 的 `requests` 包：

##### 示例 8-3\.

```py
import requests
print(requests.get("http://localhost:8000/", params={"input_text": "Hello friend!"}).json())
```

在样本输入文本 `"Hello friend!"` 上测试情感分类器，它正确地将文本分类为正面！

这个示例实际上是 Ray Serve 的 "hello world"：我们在一个基本的 HTTP 端点后部署了一个单一模型。然而，请注意，我们不得不手动解析输入的 HTTP 请求并将其馈送到我们的模型中。对于这个基本示例来说，这只是一行代码，但现实世界的应用程序通常需要更复杂的输入模式，并且手动编写 HTTP 逻辑可能会很繁琐且容易出错。为了能够编写更具表现力的 HTTP API，Serve 与 [FastAPI](https://fastapi.tiangolo.com/) Python 框架集成。

Serve 部署可以包装一个 `FastAPI` 应用程序，利用其表达性的 API 来解析输入并配置 HTTP 行为。在下面的示例中，我们依赖于 `FastAPI` 来处理 `input_text` 查询参数，从而允许我们删除样板解析代码。

##### 示例 8-4\.

```py
from fastapi import FastAPI

app = FastAPI()

@serve.deployment
@serve.ingress(app)
class SentimentAnalysis:
    def __init__(self):
        self._classifier = pipeline("sentiment-analysis")

    @app.get("/")
    def classify(self, input_text: str) -> str:
        return self._classifier(input_text)[0]["label"]

fastapi_deployment = SentimentAnalysis.bind()
```

修改后的部署应该在上述示例中表现完全相同（试着使用 `serve run` 运行它！），但会优雅地处理无效输入。对于这个简单的示例来说，这些可能看起来像是小小的好处，但对于更复杂的 API 来说，这可能产生天壤之别。我们在这里不会深入探讨 FastAPI 的细节，但如果想了解其功能和语法的更多细节，请查看他们出色的[文档](https://fastapi.tiangolo.com/)。

## 扩展和资源分配

如上所述，机器学习模型通常需要大量计算资源。因此，重要的是能够为您的 ML 应用程序分配正确数量的资源，以处理请求负载并最大程度地减少成本。Ray Serve 允许您通过调整部署的资源以两种方式来调整：通过调整部署的**副本数**和调整分配给每个副本的资源。默认情况下，部署由一个使用单个 CPU 的副本组成，但这些参数可以在`@serve.deployment`装饰器中（或使用相应的`deployment.options` API）进行调整。

让我们修改上面的`SentimentClassifier`示例，以扩展到多个副本，并调整资源分配，使每个副本使用 2 个 CPU 而不是 1 个（在实践中，您将希望分析和了解您的模型以正确设置此参数）。我们还将添加一个打印语句来记录处理每个请求的进程 ID，以显示请求现在跨两个副本进行负载平衡。

##### 示例 8-5\.

```py
app = FastAPI()

@serve.deployment(num_replicas=2, ray_actor_options={"num_cpus": 2})
@serve.ingress(app)
class SentimentAnalysis:
    def __init__(self):
        self._classifier = pipeline("sentiment-analysis")

    @app.get("/")
    def classify(self, input_text: str) -> str:
        import os
        print("from process:", os.getpid())
        return self._classifier(input_text)[0]["label"]

scaled_deployment = SentimentAnalysis.bind()
```

运行我们分类器的新版本时，使用`serve run app:scaled_deployment`，并像上面一样使用`requests`进行查询，您会看到现在有两个模型副本处理请求！我们可以通过简单调整`num_replicas`来轻松地扩展到数十个或数百个副本：Ray 可以在单个集群中扩展到数百台机器和数千个进程。

在这个例子中，我们扩展到了一个静态副本数量，每个副本使用两个完整的 CPU，但 Serve 还支持更具表现力的资源分配策略： - 启用部署使用 GPU 只需设置`num_gpus`而不是`num_cpus`。Serve 支持与 Ray 核心相同的资源类型，因此部署也可以使用 TPU 或其他自定义资源。 - 资源可以是**分数**的，允许副本被高效地装箱。例如，如果单个副本不饱和一个完整的 GPU，则可以为其分配`num_gpus=0.5`，并与另一个模型进行复用。 - 对于请求负载变化的应用程序，可以配置部署根据当前正在进行的请求数量动态调整副本的数量。

有关资源分配选项的更多详细信息，请参阅最新的 Ray Serve 文档。

## 请求批处理

许多机器学习模型可以被有效地**向量化**，这意味着可以更有效地并行运行多个计算，而不是顺序运行它们。当在 GPU 上运行模型时，这尤其有益，因为 GPU 专为高效地并行执行许多计算而构建。在在线推断的背景下，这为优化提供了一条路径：并行服务多个请求（可能来自不同的来源）可以显着提高系统的吞吐量（从而节省成本）。

利用请求批处理的两种高级策略：**客户端**批处理和**服务器端**批处理。在客户端批处理中，服务器接受单个请求中的多个输入，并且客户端包含逻辑以批量发送而不是逐个发送。这在单个客户端频繁发送多个推理请求的情况下非常有用。相比之下，服务器端批处理使服务器能够批处理多个请求，而无需客户端进行任何修改。这也可用于跨多个客户端批处理请求，这使得即使在每个客户端每次发送相对较少请求的情况下，也能实现高效批处理。

Ray Serve 提供了一个内置的实用程序用于服务器端批处理，即`@serve.batch`装饰器，只需进行少量代码更改即可。此批处理支持使用 Python 的`asyncio`能力将多个请求排队到单个函数调用中。该函数应接受一个输入列表并返回相应的输出列表。

再次回顾之前的情感分类器，并将其修改为执行服务器端批处理。底层的 Hugging Face `pipeline` 支持矢量化推理，我们只需传递一个输入文本列表，它将返回相应的输出列表。我们将调用分类器的部分拆分为一个新方法`classify_batched`，它将接受一个输入文本列表作为输入，在它们之间执行推理，并以格式化列表返回输出。`classify_batched`将使用`@serve.batch`装饰器自动执行批处理。可以使用`max_batch_size`和`batch_timeout_wait_s`参数配置行为，这里我们将最大批处理大小设置为 10，并等待最多 100 毫秒。

##### 示例 8-6。

```py
from typing import List

app = FastAPI()

@serve.deployment
@serve.ingress(app)
class SentimentAnalysis:
    def __init__(self):
        self._classifier = pipeline("sentiment-analysis")

    @serve.batch(max_batch_size=10, batch_wait_timeout_s=0.1)
    async def classify_batched(self, batched_inputs: List[str]) -> List[str]:
        print("Got batch size:", len(batched_inputs))
        results = self._classifier(batched_inputs)
        return [result["label"] for result in results]

    @app.get("/")
    async def classify(self, input_text: str) -> str:
        return await self.classify_batched(input_text)

batched_deployment = SentimentAnalysis.bind()
```

请注意，现在`classify`和`classify_batched`方法都使用了 Python 的`async`和`await`语法，这意味着在同一个进程中可以并发运行许多这些调用。为了测试这种行为，我们将使用`serve.run` Python API 使用本地 Python 句柄发送请求到我们的部署中。

```py
from app import batched_deployment

# Get a handle to the deployment so we can send requests in parallel.
handle = serve.run(batched_deployment)
ray.get([handle.classify.remote("sample text") for _ in range(10)])
```

通过`serve.run`返回的句柄可用于并行发送多个请求：在这里，我们并行发送 10 个请求并等待它们全部返回。如果没有批处理，每个请求将按顺序处理，但因为我们启用了批处理，我们应该看到所有请求一次处理（在`classify_batched`方法中打印的批处理大小证明了这一点）。在 CPU 上运行时，这可能比顺序运行稍快，但在 GPU 上运行相同处理程序时，批处理版本将显著加快速度。

## 多模型推理图表。

到目前为止，我们一直在部署和查询一个 Serve 部署，包装一个 ML 模型。如前所述，单独使用机器学习模型通常是不够有用的：许多应用程序需要将多个模型组合在一起，并将业务逻辑与机器学习交织在一起。Ray Serve 的真正强大之处在于它能够将多个模型与常规 Python 逻辑组合成一个单一的应用程序。通过实例化许多不同的部署，并在它们之间传递引用，这是可能的。这些部署可以使用我们到目前为止讨论过的所有功能：它们可以独立缩放，执行请求批处理，并使用灵活的资源分配。

本节提供了常见的多模型服务模式的示例，但实际上不包含任何 ML 模型，以便专注于 Serve 提供的核心功能。稍后在本章中，我们将探讨一个端到端的多模型推理图，其中包含 ML 模型。

### 核心功能：绑定多个部署

Ray Serve 中的所有类型的多模型推理图都围绕着将一个部署的引用传递给另一个的能力。为了做到这一点，我们使用 `.bind()` API 的另一个特性：一个绑定的部署可以传递给另一个 `.bind()` 调用，并且这将在运行时解析为对部署的“句柄”。这使得部署可以独立部署和实例化，然后在运行时互相调用。以下是一个多部署 Serve 应用程序的最基本示例。

##### 示例 8-7。

```py
@serve.deployment
class DownstreamModel:
    def __call__(self, inp: str):
        return "Hi from downstream model!"

@serve.deployment
class Driver:
    def __init__(self, downstream):
        self._d = downstream

    async def __call__(self, *args) -> str:
        return await self._d.remote()

downstream = DownstreamModel.bind()
driver = Driver.bind(downstream)
```

在这个例子中，下游模型被传递到“驱动器”部署中。然后在运行时，驱动器部署调用下游模型。驱动器可以接收任意数量的传入模型，并且下游模型甚至可以接收其自己的其他下游模型。

### 模式 1: 管道化

机器学习应用程序中的第一个常见多模型模式是“管道化”：依次调用多个模型，其中一个模型的输入取决于前一个模型的输出。例如，图像处理通常由多个转换阶段的管道组成，如裁剪、分割、物体识别或光学字符识别（OCR）。每个模型可能具有不同的属性，其中一些是可以在 CPU 上运行的轻量级转换，而其他一些是在 GPU 上运行的重型深度学习模型。

这样的管道可以很容易地使用 Serve 的 API 表达。管道的每个阶段被定义为独立的部署，并且每个部署被传递到一个顶层的“管道驱动器”中。在下面的例子中，我们将两个部署传递到一个顶层驱动器中，驱动器按顺序调用它们。请注意，可能会有许多请求同时发送到驱动器，因此可以有效地饱和管道的所有阶段。

##### 示例 8-8。

```py
@serve.deployment
class DownstreamModel:
    def __init__(self, my_val: str):
        self._my_val = my_val

    def __call__(self, inp: str):
        return inp + "|" + self._my_val

@serve.deployment
class PipelineDriver:
    def __init__(self, model1, model2):
        self._m1 = model1
        self._m2 = model2

    async def __call__(self, *args) -> str:
        intermediate = self._m1.remote("input")
        final = self._m2.remote(intermediate)
        return await final

m1 = DownstreamModel.bind("val1")
m2 = DownstreamModel.bind("val2")
pipeline_driver = PipelineDriver.bind(m1, m2)
```

要测试此示例，可以再次使用`serve run` API。向管道发送测试请求将`"'input|val1|val2'"`作为输出返回：每个下游“模型”都添加了自己的值来构建最终结果。在实践中，每个部署可能会封装自己的 ML 模型，单个请求可能会在集群中的多个物理节点之间流动。

### 模式 2：广播

除了顺序链接模型之外，同时对多个模型进行推断通常也很有用。这可以是为了执行“集成学习”，即将多个独立模型的结果合并为一个结果，或者在不同输入上不同模型表现更好的情况下。通常需要将模型的结果以某种方式结合成最终结果：简单地连接在一起或者可能从中选择一个单一结果。

这与流水线示例非常相似：许多下游模型传递到顶级“驱动程序”。在这种情况下，重要的是我们并行调用模型：在调用下一个模型之前等待每个模型的结果将显著增加系统的总体延迟。

##### 示例 8-9\.

```py
@serve.deployment
class DownstreamModel:
    def __init__(self, my_val: str):
        self._my_val = my_val

    def __call__(self):
        return self._my_val

@serve.deployment
class BroadcastDriver:
    def __init__(self, model1, model2):
        self._m1 = model1
        self._m2 = model2

    async def __call__(self, *args) -> str:
        output1, output2 = self._m1.remote(), self._m2.remote()
        return [await output1, await output2]

m1 = DownstreamModel.bind("val1")
m2 = DownstreamModel.bind("val2")
broadcast_driver = BroadcastDriver.bind(m1, m2)
```

测试此端点在再次运行`serve run`后返回`'["val1", "val2"]'`，这是并行调用两个模型的组合输出。

### 模式 3：条件逻辑

最后，尽管许多机器学习应用程序大致符合上述模式之一，但静态控制流往往会非常限制。例如，构建一个从用户上传的图像中提取车牌号码的服务。在这种情况下，我们可能需要构建如上讨论的图像处理流水线，但我们也不只是盲目地将任何图像输入到流水线中。如果用户上传的是除了汽车或质量低下的图像，我们可能希望进行短路，避免调用重量级和昂贵的流水线，并提供有用的错误消息。类似地，在产品推荐用例中，我们可能希望根据用户输入或中间模型的结果选择下游模型。每个示例都需要在我们的 ML 模型旁嵌入自定义逻辑。

我们可以轻松地使用 Serve 的多模型 API 来实现这一点，因为我们的计算图是作为普通 Python 逻辑而不是作为静态定义的图形来定义的。例如，在下面的示例中，我们使用一个简单的随机数生成器（RNG）来决定调用哪个下游模型。在实际示例中，RNG 可以替换为业务逻辑、数据库查询或中间模型的结果。

##### 示例 8-10\.

```py
@serve.deployment
class DownstreamModel:
    def __init__(self, my_val: str):
        self._my_val = my_val

    def __call__(self):
        return self._my_val

@serve.deployment
class ConditionalDriver:
    def __init__(self, model1, model2):
        self._m1 = model1
        self._m2 = model2

    async def __call__(self, *args) -> str:
        import random
        if random.random() > 0.5:
            return await self._m1.remote()
        else:
            return await self._m2.remote()

m1 = DownstreamModel.bind("val1")
m2 = DownstreamModel.bind("val2")
conditional_driver = ConditionalDriver.bind(m1, m2)
```

对此端点的每次调用都以 50/50 的概率返回`"val1"`或`"val2"`。

## 在 Kubernetes 上部署

TODO：Ray Serve 的 Kubernetes 部署故事目前正在重新制定，完成后将更新本节内容。

# 端到端示例：构建基于 NLP 的 API。

在这一部分，我们将使用 Ray Serve 构建一个端到端的自然语言处理（NLP）流水线，用于在线推断。我们的目标是提供一个维基百科摘要端点，利用多个 NLP 模型和一些自定义逻辑来提供给定搜索项最相关维基百科页面的简明摘要。

此任务将汇集上述讨论的许多概念和特性：

我们的在线推断流水线将被结构化如下：

这个流水线将通过 HTTP 公开，并以结构化格式返回结果。通过本节结束时，我们将在本地运行端到端流水线，并准备在集群上进行扩展。让我们开始吧！

## 步骤 0：安装依赖项。

在我们深入代码之前，您需要在本地安装以下 Python 包，以便跟随操作。

```py
pip install ray["serve"]==1.12.0 transformers requests wikipedia
```

另外，在本节中，我们假设所有代码示例都在名为`app.py`的文件中本地可用，以便我们可以从相同目录使用`serve run`运行部署。

## 步骤 1：获取内容与预处理。

第一步是根据用户提供的搜索项从维基百科获取最相关的页面。为此，我们将利用 PyPI 上的`wikipedia`包来进行繁重的工作。我们首先搜索该术语，然后选择顶部结果并返回其页面内容。如果找不到结果，我们将返回`None`，这种边缘情况将在我们定义 API 时处理。

##### 示例 8-11。

```py
from typing import Optional

import wikipedia

def fetch_wikipedia_page(search_term: str) -> Optional[str]:
    results = wikipedia.search(search_term)
    # If no results, return to caller.
    if len(results) == 0:
        return None

    # Get the page for the top result.
    return wikipedia.page(results[0]).content
```

## 步骤 2：NLP 模型。

接下来，我们需要定义将在 API 中负责重型工作的 ML 模型。与介绍部分一样，我们将使用[Hugging Face](https://huggingface.co/)的`transformers`库，因为它提供了预训练的最先进 ML 模型的便捷 API，这样我们可以专注于服务逻辑。

我们将使用的第一个模型是情感分类器，与我们在上面示例中使用的相同。此模型的部署将利用 Serve 的批处理 API 进行向量化计算。

##### 示例 8-12\.

```py
from transformers import pipeline

@serve.deployment
class SentimentAnalysis:
    def __init__(self):
        self._classifier = pipeline("sentiment-analysis")

    @serve.batch(max_batch_size=10, batch_wait_timeout_s=0.1)
    async def is_positive_batched(self, inputs: List[str]) -> List[bool]:
        results = self._classifier(inputs, truncation=True)
        return [result["label"] == "POSITIVE" for result in results]

    async def __call__(self, input_text: str) -> bool:
        return await self.is_positive_batched(input_text)
```

我们还将使用文本摘要模型为所选文章提供简洁的摘要。此模型接受一个可选的“max_length”参数来限制摘要的长度。因为我们知道这是模型中计算成本最高的，所以我们设置 `num_replicas=2` — 这样，如果我们同时有多个请求进来，它可以跟上其他模型的吞吐量。实际上，我们可能需要更多的副本来跟上输入负载，但这只能通过分析和监控来确定。

##### 示例 8-13\.

```py
@serve.deployment(num_replicas=2)
class Summarizer:
    def __init__(self, max_length: Optional[int] = None):
        self._summarizer = pipeline("summarization")
        self._max_length = max_length

    def __call__(self, input_text: str) -> str:
        result = self._summarizer(
            input_text, max_length=self._max_length, truncation=True)
        return result[0]["summary_text"]
```

我们管道中的最终模型将是一个命名实体识别模型：它将尝试从文本中提取命名实体。每个结果都有一个置信度分数，因此我们可以设置一个阈值，只接受超过某个阈值的结果。我们还可能希望限制返回的实体总数。该部署的请求处理程序调用模型，然后使用一些基本的业务逻辑来执行所提供的置信度阈值和实体数量的限制。

##### 示例 8-14\.

```py
@serve.deployment
class EntityRecognition:
    def __init__(self, threshold: float = 0.90, max_entities: int = 10):
        self._entity_recognition = pipeline("ner")
        self._threshold = threshold
        self._max_entities = max_entities

    def __call__(self, input_text: str) -> List[str]:
        final_results = []
        for result in self._entity_recognition(input_text):
            if result["score"] > self._threshold:
                final_results.append(result["word"])
            if len(final_results) == self._max_entities:
                break

        return final_results
```

## 第三步：HTTP 处理和驱动逻辑

随着输入预处理和 ML 模型的定义，我们准备定义 HTTP API 和驱动逻辑。首先，我们使用 [Pydantic](https://pydantic-docs.helpmanual.io/) 定义了从 API 返回的响应模式的架构。响应包括请求是否成功以及状态消息，另外还有我们的摘要和命名实体。这将允许我们在错误条件下返回有用的响应，例如当未找到结果或情感分析结果为负面时。

##### 示例 8-15\.

```py
from pydantic import BaseModel

class Response(BaseModel):
    success: bool
    message: str = ""
    summary: str = ""
    named_entities: List[str] = []
```

接下来，我们需要定义实际的控制流逻辑，这些逻辑将在驱动程序部署中运行。驱动程序本身不会执行任何实际的重型工作，而是调用我们的三个下游模型部署并解释它们的结果。它还将承载 FastAPI 应用程序定义，解析输入并根据管道的结果返回正确的`Response`模型。

##### 示例 8-16\.

```py
from fastapi import FastAPI

app = FastAPI()

@serve.deployment
@serve.ingress(app)
class NLPPipelineDriver:
    def __init__(self, sentiment_analysis, summarizer, entity_recognition):
        self._sentiment_analysis = sentiment_analysis
        self._summarizer = summarizer
        self._entity_recognition = entity_recognition

    @app.get("/", response_model=Response)
    async def summarize_article(self, search_term: str) -> Response:
        # Fetch the top page content for the search term if found.
        page_content = fetch_wikipedia_page(search_term)
        if page_content is None:
            return Response(success=False, message="No pages found.")

        # Conditionally continue based on the sentiment analysis.
        is_positive = await self._sentiment_analysis.remote(page_content)
        if not is_positive:
            return Response(success=False, message="Only positivitiy allowed!")

        # Query the summarizer and named entity recognition models in parallel.
        summary_result = self._summarizer.remote(page_content)
        entities_result = self._entity_recognition.remote(page_content)
        return Response(
            success=True,
            summary=await summary_result,
            named_entities=await entities_result
        )
```

在主处理程序的主体中，我们首先使用我们的 `fetch_wikipedia_page` 逻辑获取页面内容（如果未找到结果，则返回错误）。然后，我们调用情感分析模型。如果返回负面结果，我们提前终止并返回错误，以避免调用其他昂贵的 ML 模型。最后，我们并行广播文章内容到摘要和命名实体识别模型。两个模型的结果被拼接到最终响应中，我们返回成功。请记住，我们可能有许多对此处理程序的调用同时运行：对下游模型的调用不会阻塞驱动程序，并且它可以协调对重型模型的多个副本的调用。

## 第四步：将所有内容整合在一起

到这一步，所有核心逻辑都已经定义好了。现在剩下的就是将部署的图形绑定在一起并运行它！

##### 示例 8-17\.

```py
sentiment_analysis = SentimentAnalysis.bind()
summarizer = Summarizer.bind()
entity_recognition = EntityRecognition.bind(threshold=0.95, max_entities=5)
nlp_pipeline_driver = NLPPipelineDriver.bind(
    sentiment_analysis, summarizer, entity_recognition)
# end::final_driver[]
```

首先，我们需要为每个部署实例化，使用任何相关的输入参数。例如，在这里，我们为实体识别模型传递了阈值和限制。最重要的部分是我们将三个模型的引用传递给驱动程序，以便它可以协调计算。现在我们已经定义了完整的自然语言处理流水线，我们可以使用 `serve run` 来运行它：

```py
serve run ch_08_model_serving:nlp_pipeline_driver
```

这将在本地部署每个四个部署，并使驱动程序在 `http://localhost:8000/` 上可用。我们可以使用 `requests` 查询流水线，看看它如何工作。首先，让我们尝试查询 Ray Serve 上的一个条目。

##### 示例 8-18\.

```py
print(requests.get("http://localhost:8000/", params={"search_term": "rayserve"}).text)
'{"success":false,"message":"No pages found.","summary":"","named_entities":[]}'
```

不幸的是，这个页面还不存在！验证业务逻辑的第一块代码生效并返回了“未找到页面”的消息。让我们尝试寻找一些更常见的内容：

##### 示例 8-19\.

```py
print(requests.get("http://localhost:8000/", params={"search_term": "war"}).text)
'{"success":false,"message":"Only positivitiy allowed!","summary":"","named_entities":[]}'
```

也许我们只是对了解历史感兴趣，但是这篇文章对我们的情感分类器来说有点太消极了。这次让我们试试更中立一点的东西——科学怎么样？

##### 示例 8-20\.

```py
print(requests.get("http://localhost:8000/", params={"search_term": "physicist"}).text)
'{"success":true,"message":"","summary":" Physics is the natural science that studies matter, its fundamental constituents, its motion and behavior through space and time, and the related entities of energy and force . During the Scientific Revolution in the 17th century these natural sciences emerged as unique research endeavors in their own right . Physics intersects with many interdisciplinary areas of research, such as biophysics and quantum chemistry .","named_entities":["Scientific","Revolution","Ancient","Greek","Egyptians"]}'
```

这个示例成功地通过了完整的流水线：API 返回了文章的简明摘要和相关命名实体的列表。

总结一下，在这一部分，我们能够使用 Ray Serve 构建一个在线自然语言处理 API。这个推理图包含了多个机器学习模型以及自定义业务逻辑和动态控制流。每个模型可以独立扩展并拥有自己的资源分配，我们可以利用服务器端批处理来进行向量化计算。现在我们已经能够在本地测试 API，下一步将是部署到生产环境。Ray Serve 通过使用 Ray 集群启动器，可以轻松部署到 Kubernetes 或其他云提供商提供的服务，并且通过调整部署的资源分配，可以轻松扩展以处理多个用户。

# 摘要

# 关于作者

**马克斯·普姆佩拉** 是一位数据科学教授和软件工程师，位于德国汉堡。他是一位活跃的开源贡献者，维护了几个 Python 包，撰写了机器学习书籍，并在国际会议上发表演讲。作为 Pathmind 公司产品研究负责人，他正在使用 Ray 开发规模化的工业应用强化学习解决方案。Pathmind 与 AnyScale 团队密切合作，并且是 Ray 的 RLlib、Tune 和 Serve 库的高级用户。马克斯曾是 Skymind 公司的 DL4J 核心开发者，帮助扩展和发展 Keras 生态系统，并担任 Hyperopt 维护者。

**爱德华·奥克斯**

**理查德·李奥**
