# 第二章：现代 Python

> 这都是 Confuse-a-Cat 每天的工作内容。
> 
> Monty Python

# 预览

Python 在与我们变化的技术世界保持同步时在进化。本章讨论了适用于前一章问题的具体 Python 功能，以及一些额外的：

+   工具

+   API 和服务

+   变量和类型提示

+   数据结构

+   Web 框架

# 工具

每种计算语言都有以下内容：

+   核心语言和内置标准包

+   添加外部包的方法

+   推荐的外部包

+   开发工具环境

接下来的章节列出了本书所需或推荐的 Python 工具。

这些可能随时间而改变！Python 的打包和开发工具是移动的目标，时不时会有更好的解决方案出现。

# 入门

你应该能够编写和运行像示例 2-1 这样的 Python 程序。

##### 示例 2-1。这个 Python 程序是这样的：this.py

```py
def paid_promotion():
    print("(that calls this function!)")

print("This is the program")
paid_promotion()
print("that goes like this.")
```

要在文本窗口或终端命令行中执行此程序，我将使用一个`$` *提示*（系统请求您输入一些内容）。您在提示后键入的内容显示为**`bold print`**。如果您已将示例 2-1 保存为*this.py*文件，则可以像在示例 2-2 中显示的那样运行它。

##### 示例 2-2。测试 this.py

```py
$ python this.py
This is the program
(that calls this function!)
that goes like this.
```

一些代码示例使用交互式的 Python 解释器，只需键入**`python`**即可获得：

```py
$ python
Python 3.9.1 (v3.9.1:1e5d33e9b9, Dec  7 2020, 12:10:52)
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

前几行是特定于您的操作系统和 Python 版本。这里的`>>>`是您的提示符。交互解释器的一个方便额外功能是，如果您键入其名称，它将为您打印变量的值：

```py
>>> wrong_answer = 43
>>> wrong_answer
43
```

这也适用于表达式：

```py
>>> wrong_answer = 43
>>> wrong_answer - 3
40
```

如果你对 Python 比较陌生或想要快速复习，可以阅读接下来的几节。

## Python 本身

你至少需要 Python 3.7 作为基本要求。这包括像类型提示和 asyncio 这样的功能，这些是 FastAPI 的核心要求。我建议至少使用 Python 3.9，它将有更长的支持周期。Python 的标准来源是[Python 软件基金会](https://www.python.org)。

## 包管理

你会想要在计算机上安全下载外部的 Python 包并将其安装。这方面的经典工具是[pip](https://pip.pypa.io)。

但是你如何下载这个下载器呢？如果你从 Python 软件基金会安装了 Python，你应该已经有了 pip。如果没有，请按照 pip 网站上的说明获取它。在本书中，当我介绍一个新的 Python 包时，我会包括下载它的 pip 命令。

虽然你可以用普通的 pip 做很多事情，但你可能也想使用虚拟环境，并考虑像 Poetry 这样的替代工具。

## 虚拟环境

Pip 将下载和安装软件包，但它们应该放在哪里？尽管标准 Python 及其包含的库通常安装在操作系统的标准位置，但您可能（并且可能不应该）能够在那里进行任何更改。Pip 使用除系统目录之外的默认目录，因此您不会覆盖系统的标准 Python 文件。您可以更改此设置；有关详细信息，请参阅 pip 网站，了解适用于您操作系统的详情。

但是通常会使用多个版本的 Python，或者为项目安装特定的版本，这样你就能确切地知道其中包含哪些软件包。为此，Python 支持*虚拟环境*。这些只是目录（在非 Unix 世界中称为*文件夹*），pip 将下载的软件包写入其中。当你*激活*一个虚拟环境时，你的 Shell（主系统命令解释器）在加载 Python 模块时首先查找这些目录。

这个程序就是[venv](https://oreil.ly/9kv5T)，自 Python 3.4 版本起就已经包含在标准 Python 中。

让我们创建一个名为`venv1`的虚拟环境。您可以像独立程序一样运行 venv 模块：

```py
$ venv venv1
```

或作为 Python 模块：

```py
$ python -m venv venv1
```

要使这成为您当前的 Python 环境，请运行此 Shell 命令（在 Linux 或 Mac 上；有关 Windows 和其他系统的详情，请参阅 venv 文档）：

```py
$ source venv1/bin/activate
```

现在，每次你运行`pip install`，它将在`venv1`下安装软件包。当你运行 Python 程序时，Python 解释器和模块就在那里。

要*停用*你的虚拟环境，请按 Control-D（Linux 或 Mac），或者键入`**deactivate**`（Windows）。

你可以创建像`venv2`这样的备选环境，并在它们之间进行停用/激活操作（尽管我希望你的命名想象力比我更强）。

## Poetry

pip 和 venv 的这种组合非常常见，人们开始将它们组合在一起以节省步骤，并避免那些`source`命令的复杂性。其中一个这样的包是[Pipenv](https://pipenv.pypa.io)，但一个更新的竞争对手叫做[Poetry](https://python-poetry.org)正在变得更加流行。

使用了 pip、Pipenv 和 Poetry 之后，我现在更喜欢 Poetry。用`pip install poetry`来获取它。Poetry 有许多子命令，比如`poetry add`用于向您的虚拟环境添加软件包，`poetry install`用于实际下载和安装它等等。查看 Poetry 网站或运行`poetry`命令以获取帮助。

除了下载单个软件包外，pip 和 Poetry 还管理配置文件中的多个软件包：*requirements.txt*用于 pip，*pyproject.toml*用于 Poetry。Poetry 和 pip 不仅下载软件包，还管理软件包可能对其他软件包的复杂依赖关系。您可以指定所需软件包的版本，如最小值、最大值、范围或确切值（也称为*固定版本*）。随着项目的增长和所依赖的软件包发生变化，这可能很重要。如果您使用的功能首次出现在某个版本中，您可能需要该软件包的最小版本，或者如果删除了某个功能，则可能需要该软件包的最大版本。

## 源代码格式化

源代码格式化比前几节的主题不那么重要，但仍然有帮助。避免使用工具对源代码进行格式化（*小工具论*）争论。一个不错的选择是 [Black](https://black.readthedocs.io)。使用 `pip install black` 安装它。

## 测试

测试在第十二章中有详细说明。尽管标准的 Python 测试包是 unittest，但大多数 Python 开发人员使用的产业强度 Python 测试包是 [pytest](https://docs.pytest.org)。使用 `pip install pytest` 安装它。

## 源代码控制和持续集成

现在几乎普遍的源代码控制解决方案是*Git*，使用像 GitHub 和 GitLab 这样的存储库（*repos*）。使用 Git 不限于 Python 或 FastAPI，但你可能会在开发中花费大量时间使用 Git。[pre-commit](https://pre-commit.com) 工具在提交到 Git 之前在本地运行各种测试（如 `black` 和 `pytest`）。推送到远程 Git 存储库后，可能会在那里运行更多的持续集成（CI）测试。

第十二章 和 “故障排除” 有更多细节。

## Web 工具

第三章 展示了如何安装和使用本书中使用的主要 Python Web 工具：

FastAPI

Web 框架本身

Uvicorn

异步 Web 服务器

HTTPie

一个类似于 curl 的文本 Web 客户端

Requests

同步 Web 客户端包

HTTPX

同步/异步 Web 客户端包

# API 和服务

Python 的模块和包对于创建不会成为[“大块泥巴”](https://oreil.ly/zzX5T)的大型应用程序至关重要。即使在单进程 Web 服务中，通过模块和导入的精心设计，你也可以保持第一章中讨论的分离。 

Python 的内置数据结构非常灵活，非常诱人，可以在各处使用。但在接下来的章节中，你会看到我们可以定义更高级的*模型*来使我们的层间通信更清洁。这些模型依赖于一个相对较新的 Python 添加功能称为*类型提示*。让我们深入了解一下，但首先简要了解 Python 如何处理*变量*。这不会伤害到你。

# 变量是名称

在软件世界中，术语*对象*有许多定义——也许太多了。在 Python 中，对象是程序中每个不同数据的数据结构，从像 `5` 这样的整数，到函数，到你可能定义的任何东西。它指定了，除其他事务信息外，以下内容：

+   一个独特的*标识*值

+   与硬件匹配的低级*类型*

+   特定的*值*（物理位）

+   变量的*引用计数*，即指向它的变量数目

Python 在对象级别上是*强类型*的（它的*类型*不会改变，尽管其*值*可能会）。如果一个对象的值可以被改变，则称其为*可变*的，否则称其为*不可变*的。

但在*变量*层面上，Python 与许多其他计算语言不同，这可能令人困惑。在许多其他语言中，*变量*本质上是指向内存区域的直接指针，该区域包含按照计算机硬件设计存储的原始*值*的位。如果您给该变量赋予一个新值，语言将会用新值覆盖内存中的旧值。

这是直接且快速的。编译器跟踪了每个东西的位置。这是 C 等语言比 Python 更快的一个原因。作为开发者，您需要确保每个变量只分配正确类型的值。

现在，这里是 Python 的一个重要区别：Python 变量只是一个*名称*，暂时关联到内存中的一个更高级的*对象*。如果你给一个引用不可变对象的变量赋予一个新值，实际上你创建了一个包含该值的新对象，然后让该名称指向这个新对象。旧对象（名称曾经引用的对象）随后变为自由状态，并且如果没有其他名称仍然引用它（即其引用计数为 0），其内存可以被回收。

在*Introducing Python*（O’Reilly）中，我将对象比作坐在内存架子上的塑料盒子，而名称/变量则是粘在这些盒子上的便签。或者你可以将名称想象为附在这些盒子上的带有字符串的标签。

通常情况下，当你使用一个名称时，你将其分配给一个对象，并且它会保持附着状态。这种简单的一致性有助于你理解你的代码。变量的*作用域*是名称在其中引用相同对象的代码区域，例如在函数内部。你可以在不同的作用域中使用相同的名称，但每个作用域都引用不同的对象。

虽然在 Python 程序中，您可以使一个变量引用不同的对象，但这并不一定是一个好的实践。如果不查看，您无法确定第 100 行的名称`x`是否与第 20 行的名称`x`在相同的作用域内。（顺便说一句，`x`是一个糟糕的名称。我们应该选择那些实际上有意义的名称。）

# 类型提示

所有这些背景都有一个重点。

Python 3.6 增加了*类型提示*，用于声明变量引用的对象的类型。这些提示**并不**由 Python 解释器在运行时强制执行！相反，它们可以被各种工具用来确保您对变量的使用是一致的。标准类型检查器称为*mypy*，我稍后会展示给你看。

类型提示可能看起来只是一件好事，就像程序员使用的许多 lint 工具，用于避免错误。例如，它可能提醒您，您的变量`count`引用了一个 Python 类型为`int`的对象。但是提示，尽管它们是可选的并且是未强制执行的注释（字面上是提示），却有意想不到的用途。在本书的后面，您将看到 FastAPI 如何调整 Pydantic 包以巧妙利用类型提示。

类型声明的添加可能是其他以前无类型语言的趋势。例如，许多 JavaScript 开发人员已转向[TypeScript](https://www.typescriptlang.org)。

# 数据结构

在第五章中，您将了解有关 Python 和数据结构的详细信息。

# Web 框架

作为其他功能之一，Web 框架在 HTTP 字节和 Python 数据结构之间进行转换。它可以节省大量精力。另一方面，如果其中一部分不按您的需求工作，则可能需要入侵解决方案。俗话说，不要重复造轮子——除非您不能获得圆形的。

[Web 服务器网关接口（WSGI）](https://wsgi.readthedocs.io)是将应用程序代码连接到 Web 服务器的同步 Python[标准规范](https://peps.python.org/pep-3333)。传统的 Python Web 框架都建立在 WSGI 之上。但同步通信可能意味着等待一些比 CPU 慢得多的东西，如磁盘或网络。然后您将寻找更好的*并发性*。近年来，并发性变得更加重要。因此，开发了 Python 的[异步服务器网关接口（ASGI）规范](https://asgi.readthedocs.io)。第四章详细讨论了这一点。

## Django

[Django](https://www.djangoproject.com)是一个全功能的 Web 框架，自称为“完美主义者的截止日期 Web 框架”。它由 Adrian Holovaty 和 Simon Willison 于 2003 年宣布，并以 20 世纪比利时爵士吉他手 Django Reinhardt 命名。Django 经常用于数据库支持的企业网站。在第七章中，我将更多地详细介绍 Django。

## Flask

相比之下，由 Armin Ronacher 在 2010 年推出的[Flask](https://flask.palletsprojects.com)是一个*微框架*。第七章更多地讨论了 Flask 及其与 Django 和 FastAPI 的比较。

## FastAPI

在舞会上与其他求婚者见面后，我们最终遇到了引人入胜的 FastAPI，这本书的主题。尽管 FastAPI 由 Sebastián Ramírez 于 2018 年发布，但它已经攀升至 Python Web 框架的第三位，仅次于 Flask 和 Django，并且增长速度更快。2022 年的[比较](https://oreil.ly/36WTQ)显示它可能在某个时候超过它们。

###### 注意

截至 2023 年 10 月底，GitHub 上的星标数如下：

+   Django：73.8 千

+   Flask：64.8 千

+   FastAPI：64 千

经过对[替代方案](https://oreil.ly/JDDOm)的仔细调查，Ramírez 提出了一个[设计](https://oreil.ly/zJFTX)，该设计主要基于两个第三方 Python 包：

+   *Starlette* 用于 Web 的详细信息

+   *Pydantic* 的数据详情

他还为最终产品添加了自己的成分和特殊酱汁。您将在下一章中看到我所指的。

# 回顾

本章涵盖了与今天的 Python 相关的许多内容：

+   Python Web 开发者的有用工具

+   API 和服务的显著性

+   Python 的类型提示、对象和变量

+   Web 服务的数据结构

+   Web 框架
