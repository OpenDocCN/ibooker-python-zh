# 第八章：Ray Workflows

由 Carlos Andrade Costa 贡献

在广泛的领域中，现实生活和现代应用程序通常是多个相互依赖步骤的组合。例如，在 AI/ML 工作流中，训练工作负载需要进行多个步骤的数据清洗、平衡和增强，而模型服务通常包括许多子任务和与长期运行的业务流程的集成。工作流中的不同步骤可以依赖于多个上游，并且有时需要不同的扩展工具。

工作流管理的计算机库追溯到 25 年前，新工具聚焦于 AI/ML 的出现。工作流规范涵盖从图形用户界面到自定义格式、YAML Ain't Markup Language (YAML)和 Extensible Markup Language (XML)，以及全功能编程语言中的库。在代码中指定工作流允许您使用通用的编程工具，如版本控制和协作。

在本章中，您将学习 Ray Workflows 实现的基础知识以及其使用的一些简单示例。

# 什么是 Ray Workflows？

*Ray Workflows*通过添加工作流原语扩展了 Ray Core，提供了对任务和演员的程序化工作流执行支持，并提供了与任务和演员共享接口的支持。这使您可以将 Ray 的核心原语作为工作流步骤的一部分使用。Ray Workflows 旨在支持传统的 ML 和数据工作流（例如数据预处理和训练），以及长时间运行的业务工作流，包括模型服务集成。它利用 Ray 任务进行执行，以提供可伸缩性和可靠性。Ray 的工作流原语极大地减少了将工作流逻辑嵌入应用程序步骤的负担。

# 与其他解决方案有何不同？

与其他流行的工作流框架（例如[Apache Airflow](https://oreil.ly/ZKymk)，[Kubeflow Pipelines](https://oreil.ly/dgEk7)等）不同，这些框架侧重于工具集成和部署编排，Ray Workflows 专注于支持较低级别的工作流原语，从而实现程序化工作流。¹这种程序化方法相对于其他实现可以被认为是较低级别的；这种低级别方法允许独特的工作流管理功能。

###### 注意

Ray Workflows 专注于将核心工作流原语嵌入 Ray Core 中，以实现丰富的程序化工作流，而不是支持工具集成和部署编排。

# Ray Workflows 特点

在这一部分中，我们将介绍 Ray Workflows 的主要特性，回顾核心原语，并看看它们在简单示例中的使用方式。

## 主要特性是什么？

Ray Workflows 提供的主要功能包括以下内容：

耐久性

通过添加虚拟演员（参见“虚拟演员”），Ray Workflows 为使用 Ray 的动态任务图执行的步骤添加了耐久性保证。

依赖管理

Ray Workflows 利用 Ray 的运行时环境特性来快照工作流的代码依赖关系。这使得可以随着时间的推移管理工作流和虚拟演员的代码升级。

低延迟和规模

利用 Ray 与 Plasma（共享内存存储）的零拷贝开销，Ray Workflows 在启动任务时提供亚秒级的开销。Ray 的可伸缩性扩展到工作流，允许您创建包含数千个步骤的工作流。

###### 注意

Ray Workflows 使用 Ray 的任何分布式库提供工作流步骤的持久执行，具有低延迟和动态依赖管理。

## 工作流基元

Ray Workflows 提供了构建带有步骤和 *虚拟演员* 的工作流的核心基元。以下列表总结了 Ray Workflows 中的核心基元和基本概念：

步骤

带有 `@workflow.step` 装饰器的注释函数。步骤在成功完成时执行一次，并在失败时重试。步骤可以作为其他步骤未来的参数使用。为确保可恢复性，步骤不支持 `ray.get` 和 `ray.wait` 调用。

对象

存储在 Ray 对象存储中的数据对象，这些对象的引用被传递到步骤中，并从步骤返回。当从步骤最初返回时，对象被检查点并可以通过 Ray 对象存储与其他工作流步骤共享。

工作流

使用 `@Workflow.run` 和 `Workflow.run_async` 创建的执行图。工作流执行在启动后被记录到存储中以保证耐久性，并可以在任何具有访问存储权限的 Ray 集群上失败后恢复。

工作流也可以是动态的，在运行时生成新的子工作流步骤。工作流支持动态循环、嵌套和递归。甚至可以通过从工作流步骤返回更多的工作流步骤来动态添加新的步骤到您的工作流有向无环图（DAG）中。

虚拟演员

虚拟演员类似于常规的 Ray 演员，可以保存成员状态。主要区别在于虚拟演员由耐久性存储支持，而不仅仅是进程内存，这使得其能够在集群重新启动或工作节点失败时存活。

虚拟演员管理长时间运行的业务工作流程。它们将其状态保存到外部存储以保证耐久性。它们还支持从方法调用启动子工作流程，并接收外部触发的事件。

您可以使用虚拟演员为本质上无状态的工作流添加状态。

事件

工作流可以通过定时器和可插拔事件监听器触发，并作为步骤的参数使用，使步骤执行等待直到接收到事件。

# 使用基本工作流概念进行工作

工作流是由各种基元构建而成，您将从学习如何使用步骤和对象开始。

## 工作流、步骤和对象

示例 8-1 展示了一个简单的 Hello World 工作流示例，演示了在简单情况下步骤、对象和工作流基元的工作方式。

##### 示例 8-1\. [Hello World 工作流](https://oreil.ly/4G4VW)

```py
import ray
from ray import workflow
from typing import List

# Creating an arbitrary Ray remote function
@ray.remote
def hello():
    return "hello"

# Defining a workflow step that puts an object into the object store
@workflow.step
def words() -> List[ray.ObjectRef]:
    return [hello.remote(), ray.put("world")]

# Defining a step that receives an object
@workflow.step
def concat(words: List[ray.ObjectRef]) -> str:
    return " ".join([ray.get(w) for w in words])

# Creating workflow
workflow.init("tmp/workflow_data")
output: "Workflow[int]" = concat.step(words.step())

# Running workflow
assert output.run(workflow_id="workflow_1") == "hello world"
assert workflow.get_status("workflow_1") == workflow.WorkflowStatus.SUCCESSFUL
assert workflow.get_output("workflow_1") == "hello world"
```

与 Ray 任务和演员类似（在第三章和第四章中描述），您可以显式地为步骤分配计算资源（例如，CPU 核心、GPU），方法与核心 Ray 中相同：`num_cpus`、`num_gpus` 和 `resources`。参见 示例 8-2。

##### 示例 8-2\. 为步骤添加资源

```py
from ray import workflow
@workflow.step(num_gpus=1)
def train_model() -> Model:
    pass  # This step is assigned a GPU by Ray.

train_model.step().run()
```

## 动态工作流

除了预定义的有向无环图工作流之外，Ray 还允许您根据工作流执行的当前状态以编程方式创建步骤：*动态工作流*。您可以使用这种类型的工作流，例如，来实现递归和更复杂的执行流程。一个简单的递归可以用递归阶乘程序来说明。示例 8-3 展示了如何在工作流中使用递归（请注意，这仅供说明，其他实现方式具有更好的性能，无需 Ray 工作流）。

##### 示例 8-3\. [动态创建工作流步骤](https://oreil.ly/3vtT5)

```py
from ray import workflow

@workflow.step
def factorial(n: int) -> int:
    if n == 1:
        return 1
    else:
        return mult.step(n, factorial.step(n - 1))

@workflow.step
def mult(a: int, b: int) -> int:
    return a * b

# Calculate the factorial of 5 by creating a recursion of 5 steps
factorial_workflow = factorial.step(5).run()
assert factorial_workflow.run() == 120
```

## 虚拟演员

*虚拟演员*是由持久性存储支持的 Ray 演员（参见 第四章），而不是内存；它们是用装饰器 `@virtual_actor` 创建的。示例 8-4 展示了如何使用持久性虚拟演员来实现一个计数器。

##### 示例 8-4\. [虚拟演员](https://oreil.ly/1lV4k)

```py
from ray import workflow

@workflow.virtual_actor
class counter:
    def __init__(self):
        self.count = 0

    def incr(self):
        self.count += 1
        return self.count

workflow.init(storage="/tmp/workflows")

workflow1 = counter.get_or_create("counter_workflw")
assert c1.incr.run() == 1
assert c1.incr.run() == 2
```

###### 警告

因为虚拟演员在执行每一步之前和之后检索和存储其状态，所以其状态必须是 JSON 可序列化的（以状态字典的形式）或者应提供 `getstate` 和 `setstate` 方法，这些方法将演员的状态转换为 JSON 可序列化字典。

# 现实生活中的工作流

让我们来看看使用 Ray 工作流创建和管理参考用例实现的常见步骤。

## 构建工作流

正如前面所看到的，您首先要实现单个工作流步骤，并使用 `@workflow.step` 注解声明它们。与 Ray 任务类似，步骤可以接收一个或多个输入，其中每个输入可以是一个特定值或一个 future—​执行一个或多个先前工作流步骤的结果。工作流的返回类型是 `Workflow[T]`，并且是一个 future，在工作流执行完成后可用该值。示例 8-5 说明了这个过程。在这种情况下，步骤 `get_value1` 和 `get_value2` 返回 future，这些 future 被传递给 `sum` 步骤函数。

##### 示例 8-5\. [实现工作流步骤](https://oreil.ly/Sl5bx)

```py
from ray import workflow

@workflow.step
def sum(x: int, y: int, z: int) -> int:
    return x + y + z

@workflow.step
def get_value1() -> int:
    return 100

@workflow.step
def get_value2(x: int) -> int:
    return 10*x

sum_workflow = sum.step(get_val1.step(), get_val2.step(10), 100)

assert sum_workflow.run("sum_example") == 300
```

为了简化访问步骤执行结果并在步骤之间传递数据，Ray Workflows 允许您显式命名步骤。例如，您可以通过调用`workflow.get_output(workflow_id, name="step_name")`来检索步骤执行结果，这将返回一个`ObjectRef[T]`。如果没有显式命名步骤，Ray 将自动生成一个格式为`<*WORK⁠FLOW_ID*>​.<*MOD⁠ULE_NAME*>.<*FUNC_NAME*>`的名称。

请注意，您可以在返回的引用上调用`ray.get`，它将阻塞直到工作流完成。例如，`ray.get(workflow.get_output("sum​_exam⁠ple")) == 100`。

步骤可以用两种方式命名：

+   使用`.options(name="step_name")`

+   使用装饰器`@workflows.step(name=”step_name”)`

## 管理工作流

Ray Workflows 中的每个工作流都有一个唯一的`workflow_id`。您可以在启动工作流时显式设置工作流 ID，使用`.run(workflow_id="workflow_id")`。对于`.run`和`run_async`调用时如果没有提供 ID，则会生成一个随机 ID。

创建后，工作流可以处于以下状态之一：

正在运行

当前在集群中运行。

失败

应用程序错误失败。可以从失败的步骤恢复。

可恢复

由于系统错误而失败的工作流，可以从失败的步骤恢复。

已取消

工作流已取消。无法恢复，结果不可用。

成功

工作流成功完成。

表 8-1 显示了管理 API 的摘要以及如何使用它们来管理工作流，无论是单独还是批量。

表 8-1\. 工作流管理 API

| 单个工作流 | 操作 | 批量工作流 | 操作 |
| --- | --- | --- | --- |
| `.get_status(​work⁠flow_id=<>)` | 获取工作流状态（运行中、可恢复、失败、已取消、成功） | `.list_all(​<*work⁠flow_state1*, *work⁠flow_state2*, …>)` | 列出所有处于列出状态的工作流 |
| `.resume(​work⁠flow_id=<>)` | 恢复工作流 | `.resume_all` | 恢复所有可恢复的工作流 |
| `.cancel(​work⁠flow_id=<>)` | 取消工作流 |  |  |
| `.delete(​work⁠flow_id=<>)` | 删除工作流 |  |  |

Ray Workflows 将工作流信息存储在您配置的存储位置中。您可以在使用装饰器`workflow.init(storage=<*path*>)`创建工作流时配置位置，或者通过设置环境变量`RAY_WORKFLOW_STORAGE`进行配置。

您可以使用常规/本地存储或使用兼容 S3 API 的分布式存储：

本地文件系统

单节点，仅用于测试目的，或通过集群中节点之间的共享文件系统（例如 NFS 挂载）。位置作为绝对路径传递。

S3 后端

启用将工作流数据写入基于 S3 的后端以供生产使用。

如果未指定路径，则 Workflows 将使用默认位置：*/tmp/ray/work​flow_data*。

###### 警告

如果未指定存储数据位置，则工作流数据将保存在本地，并且仅适用于单节点 Ray 集群。

Ray 的工作流依赖项正在积极开发中。一旦可用，此功能将允许 Ray 在工作流提交时将完整的运行时环境记录到存储中。通过跟踪此信息，Ray 可以确保工作流可以在不同的集群上运行。

## 构建动态工作流

正如前文所述，您可以通过基于给定步骤的当前状态创建步骤来动态创建工作流。当创建这样的步骤时，它将插入到原始工作流 DAG 中。示例 8-6 展示了如何使用动态工作流计算斐波那契数列。

##### 示例 8-6\. [动态工作流](https://oreil.ly/zaIwk)

```py
from ray import workflow

@workflow.step
def add(a: int, b: int) -> int:
    return a + b

@workflow.step
def fib(n: int) -> int:
    if n <= 1:
        return n
    return add.step(fib.step(n - 1), fib.step(n - 2))

assert fib.step(10).run() == 55
```

## 使用条件步骤构建工作流

带有条件步骤的工作流对许多用例至关重要。示例 8-7 展示了实现旅行预订工作流的简化场景。

##### 示例 8-7\. [旅行预订示例](https://oreil.ly/i7jro)

```py
from ray import workflow

@workflow.step
def book_flight(...) -> Flight: ...

@workflow.step
def book_hotel(...) -> Hotel: ...

@workflow.step
def finalize_or_cancel(
    flights: List[Flight],
    hotels: List[Hotel]) -> Receipt: ...

@workflow.step
def book_trip(origin: str, dest: str, dates) ->
        "Workflow[Receipt]":
    # Note that the workflow engine will not begin executing
    # child workflows until the parent step returns.
    # This avoids step overlap and ensures recoverability.
    f1: Workflow = book_flight.step(origin, dest, dates[0])
    f2: Workflow = book_flight.step(dest, origin, dates[1])
    hotel: Workflow = book_hotel.step(dest, dates)
    return finalize_or_cancel.step([f1, f2], [hotel])

fut = book_trip.step("OAK", "SAN", ["6/12", "7/5"])
fut.run()  # Returns Receipt(...)
```

## 处理异常

你可以选择让 Ray 以以下两种方式处理异常：

+   自动重试，直到达到最大重试次数

+   捕获和处理异常

您可以在步饰器或通过 `.options` 中配置此选项。分别指定两种技术的设置如下：

`max_retries`

在失败时，步骤将重试直到达到 `max_retries`。`max_retries` 的默认值为 `3`。

`catch_exceptions`

当设为 `True` 时，此选项将把函数的返回值转换为 `Tuple[Optional[T], Optional[Exception]]`。

您还可以将这些传递给 `workflow.step` 的装饰器。

示例 8-8 说明了使用这些选项进行异常处理。

##### 示例 8-8\. [异常处理](https://oreil.ly/Itn5V)

```py
from ray import workflow
@workflow.step
def random_failure() -> str:
    if random.random() > 0.95:
        raise RuntimeError("Found failure")
    return "OK"

# Run 5 times before giving up
s1 = faulty_function.options(max_retries=5).step()
s1.run()

@workflow.step
def handle_errors(result: Tuple[str, Exception]):
    # Setting the exception field to NONE on success
    err = result[1]
    if err:
        return "There was an error: {}".format(err)
    else:
        return "OK"

# `handle_errors` receives a tuple of (result, exception).
s2 = faulty_function.options(catch_exceptions=True).step()
handle_errors.step(s2).run()
```

## 处理耐久性保证

Ray 的工作流确保一旦步骤成功，将不会重新执行。为了强制执行此保证，Ray 的工作流将步骤结果记录到耐久性存储中，确保在后续步骤中使用之前成功步骤的结果不会改变。

Ray 的工作流不仅限于在集群或单个应用程序内重试的耐久性。工作流实现基于两种状态的故障模型：

集群故障

如果集群发生故障，则在集群上运行的任何工作流将设置为 `RESUMABLE` 状态。处于 `RESUMABLE` 状态的工作流可以在不同的集群上恢复。可以通过 `ray.workflow.resume.all` 完成此操作，它将恢复所有可恢复的工作流作业。

驱动程序故障

工作流将转换为失败状态，一旦问题解决，可以从失败的步骤恢复。

###### 警告

此时写作流恢复性为 beta API，可能会在稳定之前发生变化。

您可以使用持久性保证来创建幂等工作流，其中包含具有副作用的步骤。这是必要的，因为步骤可能在其输出被记录之前失败。示例 8-9 展示了如何使用持久性保证使工作流具有幂等性。

##### 示例 8-9\. [幂等工作流](https://oreil.ly/wmmp1)

```py
from ray import workflow

@workflow.step
def generate_id() -> str:
   # Generate a unique idempotency token.
   return uuid.uuid4().hex

@workflow.step
def book_flight_idempotent(request_id: str) -> FlightTicket:
   if service.has_ticket(request_id):
       # Retrieve the previously created ticket.
       return service.get_ticket(request_id)
   return service.book_flight(request_id)

# SAFE: book_flight is written to be idempotent
request_id = generate_id.step()
book_flight_idempotent.step(request_id).run()
```

## 扩展动态工作流程与虚拟演员

如前所述，虚拟演员还允许从其每个方法中调用子工作流。

当您创建一个虚拟演员时，Ray 将其初始状态和类定义存储在持久性存储中。由于工作流名称在演员的定义中使用，Ray 将其存储在持久性存储中。当演员的方法创建新的步骤时，它们会动态附加到工作流并执行。在这种情况下，步骤定义及其结果都存储在演员的状态中。要检索演员，您可以使用装饰器 `.get_actor(workflow_id="workflow_id")`。

您还可以将工作流定义为只读的。因为它们不需要记录，所以它们的开销较小。此外，由于它们不会与演员中的变异方法冲突，Ray 可以并行执行它们。

示例 8-10 展示了虚拟演员如何用于管理工作流程中的状态。

##### 示例 8-10\. [使用虚拟演员进行工作流管理](https://oreil.ly/zTWOk)

```py
from ray import workflow
import ray

@workflow.virtual_actor
class Counter:
    def __init__(self, init_val):
        self._val = init_val

    def incr(self, val=1):
        self._val += val
        print(self._val)

    @workflow.virtual_actor.readonly
    def value(self):
        return self._val

workflow.init()

# Initialize a Counter actor with id="my_counter".
counter = Counter.get_or_create("my_counter", 0)

# Similar to workflow steps, actor methods support:
# - `run()`, which will return the value
# - `run_async()`, which will return a ObjectRef
counter.incr.run(10)
assert counter.value.run() == 10

# Nonblocking execution.
counter.incr.run_async(10)
counter.incr.run(10)
assert 30 == ray.get(counter.value.run_async())
```

虚拟演员还可以创建涉及虚拟演员中其他方法或定义在演员类外部的步骤以供调用的子工作流。这意味着工作流可以在方法内启动或传递给另一个方法。参见 示例 8-11。

##### 示例 8-11\. [使用子工作流](https://oreil.ly/exFyJ)

```py
from ray import workflow
import ray

@workflow.step
def double(s):
    return 2 * s

@workflow.virtual_actor
class Actor:
    def __init__(self):
        self.val = 1

    def double(self, update):
        step = double.step(self.val)
        if not update:
            # Inside the method, a workflow can be launched
            return step
        else:
            # Workflow can also be passed to another method
            return self.update.step(step)

    def update(self, v):
        self.val = v
        return self.val

handler = Actor.get_or_create("actor")
assert handler.double.run(False) == 2
assert handler.double.run(False) == 2
assert handler.double.run(True) == 2
assert handler.double.run(True) == 4
```

虚拟演员还可用于在多个工作流之间共享数据（甚至在不同的 Ray 集群上运行）。例如，虚拟演员可以用于存储像 Python scikit-learn 流水线中的 ML 模型中已拟合的参数。示例 8-12 展示了一个简单的两阶段流水线，由标准标量和决策树分类器组成。每个阶段都作为工作流步骤实现，直接调用在 `estimator_virtual_actor` 类中定义的虚拟演员的实例。其成员估算器使用 `getstate` 和 `setstate` 方法将其状态转换为和从可 JSON 序列化的字典。当输入元组的第三个输入参数指定为 `'fit'` 时，流水线被训练，并且当该参数指定为 `'predict'` 时，流水线用于预测。

为了训练一个流水线，工作流执行将 `training_tuple` 提交给标准标量，其输出然后通过分类模型管道进行训练：

```py
training_tuple = (X_train, y_train, 'fit')
classification.step(scaling.step(training_tuple, 'standardscalar'),
                    'decisiontree').run('training_pipeline')
```

要将训练好的管道用于预测，工作流执行将 `predict_tuple` 提交给相同的步骤链，尽管其 `'predict'` 参数调用虚拟 actor 中的 `predict` 函数。预测结果作为另一个包含在 `pred_y` 中的标签元组返回：

```py
predict_tuple = (X_test, y_test, 'predict')
(X, pred_y, mode) = classification.step(scaling.step(predict_tuple,
  'standardscalar'),'decisiontree').run('prediction_pipeline')
```

工作流虚拟 actor 的威力在于使训练模型可用于另一个 Ray 集群。此外，由虚拟 actor 支持的 ML 工作流可以增量更新其状态，例如重新计算时间序列特征。这使得实现具有状态的时间序列分析更容易，包括预测、预测和异常检测。

##### 示例 8-12\. [机器学习工作流](https://oreil.ly/mQBVn)

```py
import ray
from ray import workflow

import pandas as pd
import numpy as np
from sklearn import base
from sklearn.base import BaseEstimator
from sklearn.preprocessing import StandardScaler
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split

ray.init(address='auto')
workflow.init()

@ray.workflow.virtual_actor
class estimator_virtual_actor():
    def __init__(self, estimator: BaseEstimator):
        if estimator is not None:
            self.estimator = estimator

    def fit(self, inputtuple):
        (X, y, mode)= inputtuple
        if base.is_classifier(self.estimator) or base.is_regressor(self.estimator):
            self.estimator.fit(X, y)
            return X, y, mode
        else:
            X = self.estimator.fit_transform(X)
            return X, y, mode

    @workflow.virtual_actor.readonly
    def predict(self, inputtuple):
        (X, y, mode) = inputtuple
        if base.is_classifier(self.estimator) or base.is_regressor(self.estimator):
            pred_y = self.estimator.predict(X)
            return X, pred_y, mode
        else:
            X = self.estimator.transform(X)
            return X, y, mode

    def run_workflow_step(self, inputtuple):
        (X, y, mode) = inputtuple
        if mode == 'fit':
            return self.fit(inputtuple)
        elif mode == 'predict':
            return self.predict(inputtuple)

    def __getstate__(self):
        return self.estimator

    def __setstate__(self, estimator):
        self.estimator = estimator

## Prepare the data
X = pd.DataFrame(np.random.randint(0,100,size=(10000, 4)), columns=list('ABCD'))
y = pd.DataFrame(np.random.randint(0,2,size=(10000, 1)), columns=['Label'])

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

@workflow.step
def scaling(inputtuple, name):
    va = estimator_virtual_actor.get_or_create(name, StandardScaler())
    outputtuple = va.run_workflow_step.run_async(inputtuple)
    return outputtuple

@workflow.step
def classification(inputtuple, name):
    va = estimator_virtual_actor.get_or_create(name,
                                               DecisionTreeClassifier(max_depth=3))
    outputtuple = va.run_workflow_step.run_async(inputtuple)
    return outputtuple

training_tuple = (X_train, y_train, 'fit')
classification.step(scaling.step(training_tuple, 'standardscalar'), 'decisiontree').
                    run('training_pipeline')

predict_tuple = (X_test, y_test, 'predict')
(X, pred_y, mode) = classification.step(scaling.step(predict_tuple,
  'standardscalar'),'decisiontree').run('prediction_pipeline')
assert pred_y.shape[0] == 2000
```

长时间运行的工作流在作为子工作流使用时需要特别注意，因为子工作流在运行时会阻塞未来的 actor 调用。为了正确处理长时间运行的工作流，建议使用 Workflows API 来监视执行，并运行具有确定性名称的单独工作流。这种方法可以防止在失败情况下启动重复的工作流。

###### 警告

子工作流会阻塞未来的 actor 方法调用。不建议将长时间运行的工作流作为虚拟 actor 的子工作流运行。

示例 8-13 展示了如何运行一个长时间运行的工作流而不阻塞。

##### 示例 8-13\. [非阻塞工作流](https://oreil.ly/eRs6K)

```py
from ray import workflow
import ray

@workflow.virtual_actor
class ShoppingCart:
    ...
    # Check status via ``self.shipment_workflow_id`` for avoid blocking
    def do_checkout():
        # Deterministically generate a workflow ID for idempotency.
        self.shipment_workflow_id = "ship_{}".format(self.order_id)
        # Run shipping workflow as a separate async workflow.
        ship_items.step(self.items).run_async(
            workflow_id=self.shipment_workflow_id)
```

## 将工作流与其他 Ray 原语集成

Ray 工作流可以与 Ray 的核心原语一起使用。这里我们将描述一些常见的场景，其中 Workflows API 与常见的 Ray 程序集成。在将工作流与任务和 actor 集成时有两种主要场

+   从 Ray 任务或 actor 中运行工作流

+   在工作流步骤中使用 Ray 任务或 actor

另一个常见情况是在工作流的步骤之间传递对象引用。Ray 对象引用可以作为参数传递，并从任何工作流步骤返回，如 示例 8-14 所示。

##### 示例 8-14\. [使用对象引用](https://oreil.ly/NaZs2)

```py
from ray import workflow

@ray.remote
def do_add(a, b):
    return a + b

@workflow.step
def add(a, b):
    return do_add.remote(a, b)

add.step(ray.put(10), ray.put(20)).run() == 30
```

为了确保可恢复性，Ray Workflows 将内容记录到持久存储中。幸运的是，当传递到多个步骤时，Ray 不会多次检查对象。

###### 警告

Ray actor 处理程序无法在步骤之间传递。

在将 actor 和任务与 Workflows 集成时，另一个需要考虑的因素是处理嵌套参数。如前所述，当传递到步骤时，工作流输出会被完全解析，以确保在执行当前步骤之前执行所有祖先步骤。示例 8-15 说明了这种行为。

##### 示例 8-15\. [使用输出参数](https://oreil.ly/RiOl3)

```py
import ray
from ray import workflow
from typing import List

@workflow.step
def add(values: List[int]) -> int:
    return sum(values)

@workflow.step
def get_val() -> int:
    return 10

ret = add.step([get_val.step() for _ in range(3)])
assert ret.run() == 30
```

## 触发工作流（连接到事件）

Workflows 具有可插拔的事件系统，允许外部事件触发工作流。这个框架提供了一个高效的内置等待机制，并保证了一次性事件交付语义。这意味着用户不需要基于运行中的工作流步骤实现触发机制来响应事件。与工作流的其余部分一样，为了容错，事件在发生时进行了检查点。

工作流*事件*可以看作是一种只有在事件发生时才完成的工作流步骤。修饰符`.wait_for_event`用于创建事件步骤。

示例 8-16 显示了一个在 90 秒后完成的工作流步骤，并触发了外部工作流的执行。

##### 示例 8-16。[使用事件](https://oreil.ly/7hwaG)

```py
from ray import workflow
import time

# Create an event that finishes after 60 seconds.
event1_step = workflow.wait_for_event(
    workflow.event_listener.TimerListener, time.time() + 60)

# Create another event that finishes after 30 seconds.
event2_step = workflow.wait_for_event(
    workflow.event_listener.TimerListener, time.time() + 30)

@workflow.step
def gather(*args):
    return args;

# Gather will run after 60 seconds, when both event1 and event2 are done.
gather.step(event1_step, event2_step).run()
```

事件还支持通过子类化`EventListener`接口来实现自定义监听器，如示例 8-17 所示。

##### 示例 8-17。[自定义事件监听器](https://oreil.ly/3j532)

```py
from ray import workflow
class EventListener:
    def __init__(self):
        """Optional constructor. Only the constructor with no arguments will be
 called."""
        pass

    async def poll_for_event(self, *args, **kwargs) -> Event:
        """Should return only when the event is received."""
        raise NotImplementedError

    async def event_checkpointed(self, event: Event) -> None:
        """Optional. Called after an event has been checkpointed and a transaction
 can be safely committed."""
        pass
```

## 处理工作流元数据

工作流执行的一个重要需求是可观察性。通常，你不仅想要看到工作流的执行结果，还想获取关于内部状态的信息（例如，执行所采取的路径，它们的性能以及变量的值）。Ray 的[工作流元数据](https://oreil.ly/kgiX2)支持一些标准和用户定义的元数据选项。标准元数据分为工作流级别的元数据：

`status`

工作流状态，可以是`RUNNING`、`FAILED`、`RESUMABLE`、`CANCELED`或`SUCCESSFUL`

`user_metadata`

用户通过`workflow.run`添加的自定义元数据的 Python 字典

`stats`

工作流运行统计信息，包括工作流开始时间和结束时间

以及步骤级别的元数据：

`name`

步骤的名称，可以是用户通过`step.options`提供的，也可以是系统生成的

`step_options`

步骤的选项，可以是用户通过`step.options`提供的，也可以是系统默认的

`user_metadata`

用户通过`step.options`添加的自定义元数据的 Python 字典

`stats`

步骤的运行统计信息，包括步骤开始时间和结束时间

Ray Workflows 提供了一个简单的 API 来获取标准元数据：

```py
workflow.get_metadata(workflow_id)
```

你还可以获取有关工作流和步骤的元数据：

```py
workflow.get_metadata(workflow_id, name=<*step name*>)
```

API 的两个版本都返回一个包含工作流本身或单个步骤的所有元数据的字典。

除了标准元数据之外，你还可以添加自定义元数据，捕获工作流或特定步骤中感兴趣的参数：

+   可以通过`.run(metadata=metadata)`添加工作流级别的元数据。

+   可以通过`.options(metadata=metadata)`或修饰符`@workflow.step(metadata=metadata)`添加步骤级别的元数据。

最后，你可以从虚拟执行器执行中公开元数据，也可以检索工作流/步骤元数据来控制执行。

###### 提示

你添加到 Ray 指标的指标被公开为 Prometheus 指标，就像 Ray 的内置指标一样。

请注意，`get_metadata`在调用时会立即返回结果，这意味着结果可能不会包含所有字段。

# 结论

在本章中，您学习了 Ray Workflows 如何向 Ray 添加工作流原语，使您能够创建具有丰富工作流管理支持的动态管道。Ray Workflows 允许您创建涉及多个步骤的常见管道，如数据预处理、训练和长时间运行的业务工作流。有了 Ray，通过与 Ray 任务和执行器共享接口，编程式工作流执行引擎的可能性变得可行。这种能力可以极大地减轻编排工作流和将工作流逻辑嵌入应用程序步骤的负担。

这就是说，要注意，Ray 远程函数（参见第三章）基于参数的可用性提供基本的执行顺序和分支/合并功能。因此，对于一些简单的用例来说，使用 Ray Workflows 可能会显得有些大材小用，但如果你需要执行可靠性、可重启性、编程控制和元数据管理（通常都需要），Ray Workflows 是一种首选的实现方法。

¹ 这种方法最初是由[Cadence 工作流](https://oreil.ly/UNfII)引入的。 Cadence 包括一个编程框架（或客户端库），提供了其文档称之为“无视错误”的有状态编程模型，使开发人员能够像编写普通代码一样创建工作流。
