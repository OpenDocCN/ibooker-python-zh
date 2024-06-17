# 第六章：依赖项

# 预览

FastAPI 的一个非常好的设计特性之一是一种称为 *依赖注入* 的技术。这个术语听起来技术性和神秘，但它是 FastAPI 的一个关键方面，并在多个层面上都非常有用。本章将介绍 FastAPI 的内置能力以及如何编写您自己的能力。

# 什么是依赖项？

*依赖* 是您在某些时候需要的特定信息。获取此信息的通常方法是编写代码以获取它，就在您需要它的时候。

当您编写 Web 服务时，某时您可能需要执行以下操作：

+   从 HTTP 请求中收集输入参数

+   验证输入

+   检查某些端点的用户认证和授权

+   从数据源（通常是数据库）查找数据

+   发出度量、日志或跟踪信息

Web 框架将 HTTP 请求字节转换为数据结构，并且在您的 Web 层函数内部逐步从中获取您需要的内容。

# 依赖项的问题

在你需要的时候获取你想要的内容，并且不需要外部代码知道你是如何获取它的，似乎是相当合理的。但事实证明，这样做会带来一些后果：

测试

您无法测试可能以不同方式查找依赖项的函数变体。

隐藏的依赖项

隐藏细节意味着你的函数所需的代码可能会在外部代码更改时中断。

代码重复

如果你的依赖是常见的（比如在数据库中查找用户或者组合来自 HTTP 请求的值），你可能会在多个函数中重复查找代码。

OpenAPI 可见性

FastAPI 为您生成的自动测试页面需要依赖注入机制提供的信息。

# 依赖注入

*依赖注入* 这个术语比听起来简单：将函数所需的任何 *特定* 信息 *传递* 到函数中。传统的方法是传递一个辅助函数，然后您调用它以获取特定的数据。

# FastAPI 依赖项

FastAPI 更进一步：您可以将依赖项定义为函数的参数，并由 FastAPI *自动* 调用并传递它们返回的 *值*。例如，`user_dep` 依赖项可以从 HTTP 参数获取用户的用户名和密码，查找它们在数据库中，并返回一个标记，您可以用它来跟踪该用户之后的活动。您的 Web 处理函数永远不会直接调用这个函数；这是在函数调用时处理的。

你已经看到了一些依赖项，但并没有看到它们被称为这样：像 `Path`、`Query`、`Body` 和 `Header` 这样的 HTTP 数据源。这些是从 HTTP 请求中的各个区域获取请求数据的函数或 Python 类。它们隐藏了细节，如有效性检查和数据格式。

为什么不编写您自己的函数来执行此操作？您可以这样做，但您将不会有这些功能：

+   数据有效性检查

+   格式转换

+   自动文档

在许多其他 Web 框架中，你会在自己的函数内部进行这些检查。你将在 第七章 中看到这方面的例子，该章节将 FastAPI 与 Flask 和 Django 等 Python Web 框架进行了比较。但在 FastAPI 中，你可以处理自己的依赖项，就像内置的依赖项一样。

# 编写一个依赖项

在 FastAPI 中，一个依赖项是一个被执行的东西，所以一个依赖项对象需要是 `Callable` 类型，其中包括函数和类——你会用到括号和可选参数。

示例 6-1 展示了一个 `user_dep()` 依赖函数，它接受名称和密码字符串参数，并且如果用户有效则返回 `True`。对于这个第一个版本，让我们让函数对任何情况都返回 `True`。

##### 示例 6-1\. 一个依赖函数

```py
from fastapi import FastAPI, Depends, Params

app = FastAPI()

# the dependency function:
def user_dep(name: str = Params, password: str = Params):
    return {"name": name, "valid": True}

# the path function / web endpoint:
@app.get("/user")
def get_user(user: dict = Depends(user_dep)) -> dict:
    return user
```

在这里，`user_dep()` 是一个依赖函数。它的作用类似于一个 FastAPI 路径函数（它知道诸如 `Params` 等的内容），但它上面没有路径装饰器。它是一个辅助函数，而不是一个 Web 端点本身。

路径函数 `get_user()` 表明它期望一个名为 `user` 的参数变量，并且该变量将从依赖函数 `user_dep()` 中获取其值。

###### 注意

在 `get_user()` 的参数中，我们不能说 `user = user_dep`，因为 `user_dep` 是一个 Python 函数对象。我们也不能说 `user = user_dep()`，因为那样会在定义 `get_user()` 时调用 `user_dep()` 函数，而不是在使用时调用。所以我们需要额外的帮助 FastAPI `Depends()` 函数来在需要时调用 `user_dep()`。

你可以在路径函数的参数列表中定义多个依赖项。

# 依赖范围

你可以定义依赖项来覆盖单个路径函数、一组路径函数或整个 Web 应用程序。

## 单一路径

在你的 *路径函数* 中，包含一个像这样的参数：

```py
def *pathfunc*(*name*: *depfunc* = Depends(*depfunc*)):
```

或者只是这样：

```py
def *pathfunc*(*name*: *depfunc* = Depends()):
```

*`name`* 是你想要称呼由 *`depfunc`* 返回的值的任何名称。

来自先前示例：

+   *`pathfunc`* 是 `get_user()`。

+   *`depfunc`* 是 `user_dep()`。

+   *`name`* 是 `user`。

示例 6-2 使用这个路径和依赖项来返回一个固定的用户 `name` 和一个 `valid` 布尔值。

##### 示例 6-2\. 返回一个用户依赖项

```py
from fastapi import FastAPI, Depends, Params

app = FastAPI()

# the dependency function:
def user_dep(name: str = Params, password: str = Params):
    return {"name": name, "valid": True}

# the path function / web endpoint:
@app.get("/user")
def get_user(user: dict = Depends(user_dep)) -> dict:
    return user
```

如果你的依赖函数只是检查一些东西而不返回任何值，你也可以在你的路径 *装饰器* 中定义这个依赖项（前一行，以 `@` 开头）：

```py
@*app*.*method*(*url*, dependencies=[Depends(*depfunc*)])
```

让我们在 示例 6-3 中试试看。

##### 示例 6-3\. 定义一个用户检查依赖项

```py
from fastapi import FastAPI, Depends, Params

app = FastAPI()

# the dependency function:
def check_dep(name: str = Params, password: str = Params):
    if not name:
        raise

# the path function / web endpoint:
@app.get("/check_user", dependencies=[Depends(check_dep)])
def check_user() -> bool:
    return True
```

## 多条路径

第九章 提供了有关如何构建更大的 FastAPI 应用程序的详细信息，包括在顶级应用程序下定义多个 *路由器* 对象，而不是将每个端点附加到顶级应用程序。示例 6-4 勾勒了这个想法。

##### 示例 6-4\. 定义一个子路由器依赖项

```py
from fastapi import FastAPI, Depends, APIRouter

router = APIRouter(..., dependencies=[Depends(*`depfunc`*)])
```

这将导致 *`depfunc()`* 在 `router` 下的所有路径函数中被调用。

## 全局

在定义你的顶级 FastAPI 应用对象时，你可以向其添加依赖项，这些依赖项将应用于其所有路径函数，如示例 6-5 所示。

##### 示例 6-5\. 定义应用级别依赖

```py
from fastapi import FastAPI, Depends

def depfunc1():
    pass

def depfunc2():
    pass

app = FastAPI(dependencies=[Depends(depfunc1), Depends(depfunc2)])

@app.get("/main")
def get_main():
    pass
```

在这种情况下，你正在使用 `pass` 来忽略其他细节，以展示如何附加依赖项。

# 回顾

本章讨论了依赖项和依赖注入——在需要时以直接的方式获取所需数据的方法。下一章内容预告：Flask、Django 和 FastAPI 走进酒吧……
