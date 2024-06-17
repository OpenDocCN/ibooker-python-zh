# 第四章：依赖管理

Python 程序员受益于一个丰富的第三方库和工具生态系统。站在巨人的肩膀上是有代价的：你的项目所依赖的包通常也依赖于许多其他包。所有这些都是活动目标——只要任何项目存在，它的维护者就会发布一系列的版本来修复错误、添加功能，并适应不断发展的生态系统。

随着时间的推移，当你维护软件时，管理依赖关系是一个重要的挑战。你需要保持项目的最新状态，即使只是及时关闭安全漏洞。通常，这需要将你的依赖项更新到最新版本——很少有开源项目有资源为旧版本发布安全更新。你会一直更新依赖项！让这个过程尽可能无摩擦、自动化和可靠，会带来巨大的回报。

Python 项目的*依赖项*是必须在其环境中安装的第三方包。¹最常见的情况是，你因为它分发了一个你导入的模块而产生了对一个包的依赖。我们也说该项目*需要*一个包。

许多项目还使用第三方工具来进行开发者任务——比如运行测试套件或构建文档。这些包称为*开发依赖项*：最终用户不需要它们来运行你的代码。一个相关的情况是来自 第三章 的构建依赖项，它们允许你为你的项目创建包。

依赖项就像亲戚一样。如果你依赖于一个包，它的依赖项也是你的依赖项——不管你有多喜欢它们。这些包被称为*间接依赖项*；你可以把它们想象成一个以你的项目为根的树。

本章将解释如何有效地管理依赖项。在下一节中，您将学习如何在 *pyproject.toml* 中指定依赖项，作为项目元数据的一部分。之后，我将讨论开发依赖项和要求文件。最后，我会解释如何*锁定*依赖项以便于可靠的部署和可重复的检查。

# 向示例应用程序添加依赖项

作为一个工作示例，让我们使用 [HTTPX](https://www.python-httpx.org/) 库来增强 示例 3-1 中的 `random-wikipedia-article`，这是一个功能齐全的 HTTP 客户端，支持同步和异步请求，以及更新（和更高效）的协议版本 HTTP/2。你还将使用 [Rich](https://rich.readthedocs.io) 改进程序的输出，这是一个用于丰富文本和美观格式的终端库。

## 使用 HTTPX 消费 API

Wikipedia 要求开发者使用包含联系方式的`User-Agent`标头。这不是为了让他们可以寄明信片祝贺您熟练使用 Wikipedia API。而是为了在客户端无意中打击他们的服务器时提供联系方式。

示例 4-1 展示了如何使用`httpx`发送带有标头的请求到 Wikipedia API。您也可以使用标准库发送带有`User-Agent`标头的请求。但是，即使您不使用其高级功能，`httpx`也提供了更直观、明确和灵活的接口。试试看：

##### 示例 4-1\. 使用`httpx`消耗 Wikipedia API

```py
import textwrap
import httpx

API_URL = "https://en.wikipedia.org/api/rest_v1/page/random/summary"
USER_AGENT = "random-wikipedia-article/0.1 (Contact: you@example.com)"

def main():
    headers = {"User-Agent": USER_AGENT}

    with httpx.Client(headers=headers) as client: ![1](img/1.png)
        response = client.get(API_URL, follow_redirects=True) ![2](img/2.png)
        response.raise_for_status() ![3](img/3.png)
        data = response.json() ![4](img/4.png)

    print(data["title"], end="\n\n")
    print(textwrap.fill(data["extract"]))
```

![1](img/#co_dependency_management_CO1-1)

在创建客户端实例时，可以指定应该随每个请求发送的标头，如`User-Agent`标头。将客户端作为上下文管理器使用确保在`with`块结束时关闭网络连接。

![2](img/#co_dependency_management_CO1-2)

此行执行了两个 HTTP `GET`请求到 API。第一个请求发送到*random*端点，响应一个重定向到实际文章。第二个请求跟随重定向。

![3](img/#co_dependency_management_CO1-3)

`raise_for_status`方法在服务器响应通过其状态代码指示错误时引发异常。

![4](img/#co_dependency_management_CO1-4)

`json`方法抽象了将响应体解析为 JSON 的细节。

## 使用 Rich 进行控制台输出

顺便说一句，让我们改进程序的外观和感觉。示例 4-2 使用 Rich 库在粗体中显示文章标题。这只是展示了 Rich 格式选项的冰山一角。现代终端功能强大，而 Rich 让您可以轻松利用它们的潜力。请查看其[官方文档](https://rich.readthedocs.io/)获取详细信息。

##### 示例 4-2\. 使用 Rich 增强控制台输出

```py
import httpx
from rich.console import Console

def main():
    ...
    console = Console(width=72, highlight=False) ![1](img/1.png)
    console.print(data["title"], style="bold", end="\n\n") ![2](img/2.png)
    console.print(data["extract"])
```

![1](img/#co_dependency_management_CO2-1)

控制台对象提供了一个功能丰富的`print`方法用于控制台输出。将控制台宽度设置为 72 个字符取代了我们早期对`textwrap.fill`的调用。您还会希望禁用自动语法高亮，因为您正在格式化散文而不是数据或代码。

![2](img/#co_dependency_management_CO2-2)

`style`关键字允许您使用粗体字设置标题。

# 为项目指定依赖关系

如果尚未这样做，请为项目创建并激活虚拟环境，并从当前目录执行可编辑安装：

```py
$ uv venv
$ uv pip install --editable .
```

您可能会尝试手动将`httpx`和`rich`安装到环境中。相反，请将它们添加到*pyproject.toml*中的项目依赖项中。这样可以确保每次安装项目时，这两个包都会与之一起安装。

```py
[project]
name = "random-wikipedia-article"
version = "0.1"
dependencies = ["httpx", "rich"]
...
```

如果重新安装项目，您会发现 uv 也会安装其依赖项：

```py
$ uv pip install --editable .
```

`dependencies` 字段中的每个条目都是*依赖说明*。除了包名称外，它还允许您提供其他信息：版本说明符、额外功能和环境标记。以下各节将详细解释这些内容。

## 版本说明符

*版本说明符* 定义了包的可接受版本范围。当您添加新的依赖项时，包括其当前版本作为下限是个好主意——除非您的项目需要与旧版本兼容。每当开始依赖包的新特性时，更新下限。

```py
[project]
dependencies = ["httpx>=0.27.0", "rich>=13.7.1"]
```

为什么要声明依赖的下限？安装程序默认选择依赖的最新版本。有三个原因需要关注。首先，库通常与其他软件包一起安装，这些软件包可能有额外的版本约束。其次，即使应用程序不总是独立安装——例如，Linux 发行版可能为系统范围的环境打包您的应用程序。第三，下限有助于检测您自己依赖树中的版本冲突——例如，当您需要一个包的最新版本时，但另一个依赖项仅适用于其旧版本时。

避免推测性的上限版本——除非您知道它们与您的项目不兼容，否则不应防范更新版本。关于版本上限的问题，请参见“Python 中的上限版本边界”。

*锁文件* 是比上限更好的依赖性破坏解决方案——它们在部署服务或运行自动化检查时请求“已知良好”的依赖版本（参见“锁定依赖”）。

如果一个失败的发布破坏了您的项目，请发布一个修复错误的版本以排除该特定的破损版本：

```py
[project]
dependencies = ["awesome>=1.2,!=1.3.1"]
```

如果依赖项永久性地破坏了兼容性，请作为最后的手段使用上限版本。一旦能够适应您的代码，解除版本限制：

```py
[project]
dependencies = ["awesome>=1.2,<2"]
```

###### 警告

排除后期版本存在一个需要注意的陷阱。依赖解析器可以决定将您的项目降级到没有排除的版本，并且仍然升级依赖项。锁文件可以帮助解决这个问题。

版本说明符支持多个运算符，如表 4-1 所示。简而言之，请使用您从 Python 中了解的相等和比较运算符：`==`, `!=`, `<=`, `>=`, `<`, 和 `>`。

表 4-1\. 版本说明符

| 运算符 | 名称 | 描述 |
| --- | --- | --- |
| `==` | 版本匹配 | 规范化后版本必须相等。尾随零将被去除。 |
| `!=` | 版本排除 | `==` 运算符的反义词 |
| `<=`, `>=` | 包含的有序比较 | 执行词典顺序比较。预发布版本优先于最终版本。 |
| `<`, `>` | 排他性有序比较 | 类似于上述，但版本不能相等 |
| `~=` | 兼容版本 | 等同于`>=x.y,==x.*`到指定的精度 |
| `===` | 任意相等性 | 用于非标准版本的简单字符串比较 |

三个运算符值得额外讨论：

+   `==`运算符支持通配符（`*`），尽管只能在版本字符串的末尾使用。换句话说，您可以要求版本匹配特定的前缀，比如`1.2.*`。

+   `===`运算符允许您执行简单的逐字符比较。最好作为非标准版本的最后一招使用。

+   `~=`运算符用于兼容版本的指定，指定版本应大于或等于给定值，同时以相同的前缀开头。例如，`~=1.2.3`等同于`>=1.2.3,==1.2.*`，而`~=1.2`等同于`>=1.2,==1.*`。

你不需要对预发布版本进行保护—版本指定符默认排除它们。只有三种情况下预发布版本才是有效的候选项：当它们已安装时、当没有其他版本满足依赖规范时、以及明确请求它们时，使用像`>=1.0.0rc1`这样的子句。

## 额外

假设你想要使用更新的 HTTP/2 协议与`httpx`。这只需要对创建 HTTP 客户端的代码进行小修改：

```py
def main():
    headers = {"User-Agent": USER_AGENT}
    with httpx.Client(headers=headers, http2=True) as client:
        ...
```

在幕后，`httpx`将 HTTP/2 的详细信息委托给另一个包`h2`。但是，默认情况下不会引入该依赖项。这样，不需要新协议的用户可以得到更小的依赖树。但是在这里你需要它，所以使用语法`httpx[http2]`来激活可选功能：

```py
[project]
dependencies = ["httpx[http2]>=0.27.0", "rich>=13.7.1"]
```

需要额外依赖的可选功能称为*额外*，您可以有多个。例如，您可以指定`httpx[http2,brotli]`以允许解码使用*Brotli 压缩*的响应，这是 Google 开发的一种压缩算法，在 Web 服务器和内容传递网络中很常见。

### 可选依赖项

让我们从`httpx`的角度看一下这种情况。`h2`和`brotli`依赖项是可选的，因此`httpx`将它们声明在`optional-dependencies`而不是`dependencies`下（示例 4-3）。

##### 示例 4-3。简化的`httpx`可选依赖项

```py
[project]
name = "httpx"

[project.optional-dependencies]
http2 = ["h2>=3,<5"]
brotli = ["brotli"]
```

`optional-dependencies`字段是一个 TOML 表。它可以容纳多个依赖项列表，每个额外由包提供。每个条目都是一个依赖规范，并且使用与`dependencies`字段相同的规则。

如果您向项目添加一个可选依赖项，如何在代码中使用它？不要检查是否使用了额外的包—只需导入可选包。如果用户没有请求额外的内容，您可以捕获`ImportError`异常：

```py
try:
    import h2
except ImportError:
    h2 = None

# Check h2 before use.
if h2 is not None:
    ...
```

Python 中的一个常见模式——**EAFP**（Easier to Ask Forgiveness than Permission，宁愿请求原谅，而不是事先征得许可），已经如此常见，以至于它有一个名字和一个缩写。它不那么符合 Python 风格的对应物被称为**LBYL**（Look Before You Leap，事先审慎而后行）。

## 环境标记

对于依赖项的第三个元数据，你可以指定环境标记。在我解释这些标记有什么作用之前，让我向你展示一个它们何时非常有用的例子。

如果你查看了 示例 4-1 中的 `User-Agent` 头部，并想，“我不应该在代码中重复版本号”，你是完全正确的。正如你在 “单一来源项目版本” 中看到的，你可以从环境中的元数据中读取包的版本。

示例 4-4 展示了如何使用函数 `importlib.metadata.metadata` 从核心元数据字段 `Name`、`Version` 和 `Author-email` 构建 `User-Agent` 头部。这些字段对应于项目元数据中的 `name`、`version` 和 `authors`。³

##### 示例 4-4\. 使用`importlib.metadata`构建`User-Agent`头部

```py
from importlib.metadata import metadata

USER_AGENT = "{Name}/{Version} (Contact: {Author-email})"

def build_user_agent():
    fields = metadata("random-wikipedia-article") ![1](img/1.png)
    return USER_AGENT.format_map(fields) ![2](img/2.png)

def main():
    headers = {"User-Agent": build_user_agent()}
    ...
```

![1](img/#co_dependency_management_CO3-1)

`metadata`函数用于检索包的核心元数据字段。

![2](img/#co_dependency_management_CO3-2)

`str.format_map` 函数查找映射中的每个占位符。

`importlib.metadata`库是在 Python 3.8 中引入的。虽然现在它在所有支持的版本中都可用，但并非总是如此。如果你需要支持一个较旧的 Python 版本，那就不那么幸运了。

不完全是。幸运的是，许多标准库的新增功能都有*后备*——提供老版本解释器功能的第三方包。对于 `importlib.metadata`，你可以从 PyPI 上回退到 `importlib-metadata` 后备包。这个后备包依然有用，因为该库在引入后多次改变。

你只需要在使用特定 Python 版本的环境中使用后备包。环境标记允许你将此表达为条件依赖项：

```py
importlib-metadata; python_version < '3.8'
```

安装程序只会在 Python 3.8 之前的解释器上安装该包。

更一般地说，*环境标记* 表达了环境必须满足的条件，以便应用该依赖项。安装程序会在目标环境的解释器上评估该条件。

环境标记允许你针对特定的操作系统、处理器架构、Python 实现或 Python 版本请求依赖项。表 4-2 列出了所有可用的环境标记，如 PEP 508 中所指定的。⁴

表 4-2\. 环境标记^(a)

| 环境标记 | 标准库 | 描述 | 示例 |
| --- | --- | --- | --- |
| `os_name` | `os.name()` | 操作系统家族 | `posix`, `nt` |
| `sys_platform` | `sys.platform()` | 平台标识符 | `linux`, `darwin`, `win32` |
| `platform_system` | `platform.system()` | 系统名称 | Linux, Darwin, Windows |
| `platform_release` | `platform.release()` | 操作系统版本 | `23.2.0` |
| `platform_version` | `platform.version()` | 系统版本 | `Darwin Kernel Version 23.2.0: ...` |
| `platform_machine` | `platform.machine()` | 处理器架构 | `x86_64`, `arm64` |
| `python_version` | `platform.python_version_tuple()` | 格式为 `x.y` 的 Python 特性版本 | `3.12` |
| `python_full_version` | `platform.python_version()` | 完整的 Python 版本 | `3.12.0`, `3.13.0a4` |
| `platform_python_implementation` | `platform.python_implementation()` | Python 实现 | `CPython`, `PyPy` |
| `implementation_name` | `sys.implementation.name` | Python 实现 | `cpython`, `pypy` |
| `implementation_version` | `sys.implementation.version` | Python 实现版本 | `3.12.0`, `3.13.0a4` |
| ^(a) `python_version` 和 `implementation_version` 标记应用转换。详细信息请参阅 PEP 508。 |

回到 Example 4-4，这里是带有 `importlib-metadata` 的 `dependencies` 字段的完整示例：

```py
[project]
dependencies = [
    "httpx[http2]>=0.27.0",
    "rich>=13.7.1",
    "importlib-metadata>=7.0.2; python_version < '3.8'",
]
```

后备的导入名称为 `importlib_metadata`，而标准库模块名为 `importlib.metadata`。您可以通过检查 `sys.version_info` 中的 Python 版本，在代码中导入适当的模块：

```py
if sys.version_info >= (3, 8):
    from importlib.metadata import metadata
else:
    from importlib_metadata import metadata
```

我刚才听见有人喊“EAFP”了吗？如果您的导入依赖于 Python 版本，最好避免使用 “Optional dependencies” 和 “look before you leap” 技术。显式版本检查能够向静态分析工具（例如 mypy 类型检查器，见 第十章）传达您的意图。由于无法检测每个模块的可用性，EAFP 可能会导致这些工具报错。

标记支持与版本规范符号相同的相等和比较运算符（见 Table 4-1）。此外，您可以使用 `in` 和 `not in` 来匹配标记中的子字符串。例如，表达式 `'arm' in platform_version` 检查 `platform.version()` 是否包含字符串 `'arm'`。

您还可以使用布尔运算符 `and` 和 `or` 结合多个标记。以下是一个相当牵强的示例，结合了所有这些特性：

```py
[project]
dependencies = [""" \
 awesome-package; python_full_version <= '3.8.1' \
 and (implementation_name == 'cpython' or implementation_name == 'pypy') \
 and sys_platform == 'darwin' \
 and 'arm' in platform_version \
"""]
```

该示例还依赖于 TOML 对多行字符串的支持，使用与 Python 相同的三重引号。依赖规范不能跨越多行，因此您必须使用反斜杠转义换行符。

# 开发依赖

开发依赖是你在开发过程中需要的第三方软件包。作为开发者，你可能会使用 pytest 测试框架来运行项目的测试套件，Sphinx 文档系统来构建其文档，或者其他一些工具来帮助项目维护。另一方面，你的用户不需要安装这些软件包来运行你的代码。

## 一个示例：使用 pytest 进行测试

以一个具体例子来说，让我们为示例 4-4 中的`build_user_agent`函数添加一个小测试。创建一个包含两个文件的目录*tests*：一个空的*__init__.py*和一个模块*test_random_wikipedia_article.py*，其中包含来自示例 4-5 的代码。

##### 示例 4-5\. 测试生成的`User-Agent`标头

```py
from random_wikipedia_article import build_user_agent

def test_build_user_agent():
    assert "random-wikipedia-article" in build_user_agent()
```

示例 4-5 仅使用内置的 Python 功能，因此你可以只是导入并手动运行测试。但即使对于这个小测试，pytest 也添加了三个有用的功能。首先，它发现名称以`test`开头的模块和函数，因此你可以通过调用`pytest`而不带参数来运行测试。其次，pytest 在执行测试时显示测试，并在最后显示带有测试结果的摘要。第三，pytest 重写测试中的断言，以便在失败时提供友好且信息丰富的消息。

让我们使用 pytest 运行测试。我假设你已经有了一个带有可编辑安装的活动虚拟环境。在该环境中输入以下命令来安装和运行 pytest：

```py
$ uv pip install pytest
$ py -m pytest
========================= test session starts ==========================
platform darwin -- Python 3.12.2, pytest-8.1.1, pluggy-1.4.0
rootdir: ...
plugins: anyio-4.3.0
collected 1 item

tests/test_random_wikipedia_article.py .                          [100%]

========================== 1 passed in 0.22s ===========================

```

目前看起来一切都很好。测试有助于项目在不破坏功能的情况下发展。对于`build_user_agent`的测试是朝这个方向迈出的第一步。与这些长期利益相比，安装和运行 pytest 只是一个小的基础设施成本。

随着你获得更多的开发依赖项，设置项目环境变得更加困难——文档生成器、代码检查器、代码格式化程序、类型检查器或其他工具。即使你的测试套件可能需要比 pytest 更多的东西：pytest 的插件、用于测量代码覆盖率的工具，或者只是帮助你练习代码的包。

你还需要这些软件包的兼容版本——你的测试套件可能需要最新版本的 pytest，而你的文档可能无法在新的 Sphinx 发布版上构建。每个项目可能有稍微不同的要求。将这些乘以每个项目上工作的开发人员数量，就会清楚地表明你需要一种跟踪开发依赖关系的方式。

截至目前为止，Python 没有声明项目开发依赖项的标准方法——尽管许多 Python 项目管理工具在它们的`[tool]`表中支持它们，并且有一个草案 PEP 存在。⁵ 除了项目管理工具外，人们使用两种方法来填补这一空白：可选依赖和要求文件。

## 可选依赖项

正如你在“额外内容”中看到的，`optional-dependencies` 表格包含了命名为额外内容的可选依赖项组。它具有使其适合跟踪开发依赖项的三个属性。首先，默认情况下不会安装这些包，因此最终用户不会在其 Python 环境中污染它们。其次，它允许你将包分组到有意义的名称下，例如 `tests` 或 `docs`。第三，该字段具有完整的依赖关系规范的表达性，包括版本约束和环境标记。

另一方面，开发依赖项和可选依赖项之间存在阻抗不匹配。可选依赖项通过包元数据向用户公开—它们让用户选择需要额外包的功能。相比之下，用户不应该安装开发依赖项—这些包不需要任何用户可见的功能。

此外，你无法在没有项目本身的情况下安装额外的内容。相比之下，并不是所有的开发工具都需要安装你的项目。例如，代码检查器会分析你的源代码中的错误和潜在改进。你可以在不将项目安装到环境中的情况下运行它们。除了浪费时间和空间外，“臃肿”的环境还会不必要地限制依赖关系的解析。例如，当 Flake8 代码检查器对 `importlib-metadata` 设置版本限制时，许多 Python 项目就无法再升级重要的依赖项。

特别要记住的是，额外内容被广泛用于开发依赖项，并且是包装标准涵盖的唯一方法。它们是一种实用的选择，特别是如果你使用 pre-commit 管理代码检查器（见第九章）。示例 4-6 展示了如何使用额外内容来跟踪测试和文档所需的包。

##### 示例 4-6：使用额外内容表示开发依赖项

```py
[project.optional-dependencies]
tests = ["pytest>=8.1.1", "pytest-sugar>=1.0.0"] ![1](img/1.png)
docs = ["sphinx>=7.2.6"] ![2](img/2.png)
```

![1](img/1.png)：共依赖管理

`pytest-sugar` 插件可以增强 pytest 的输出，添加了进度条并立即显示失败情况。

![2](img/2.png)：共依赖管理

Sphinx 是官方 Python 文档和许多开源项目使用的文档生成器。

现在你可以使用 `tests` 额外内容安装测试依赖项了：

```py
$ uv pip install -e ".[tests]"
$ py -m pytest

```

你还可以定义一个包含所有开发依赖项的 `dev` 额外内容。这样，你可以一次性设置一个开发环境，包括你的项目和它使用的每个工具：

```py
$ uv pip install -e ".[dev]"
```

当你定义 `dev` 时，没有必要重复所有的包。相反，你可以只引用其他的额外内容，就像示例 4-7 所示。

##### 示例 4-7：提供了一个包含所有开发依赖项的 `dev` 额外内容

```py
[project.optional-dependencies]
tests = ["pytest>=8.1.1", "pytest-sugar>=1.0.0"]
docs = ["sphinx>=7.2.6"]
dev = ["random-wikipedia-article[tests,docs]"]
```

声明额外内容的这种风格也被称为*递归可选依赖*，因为具有 `dev` 额外内容的包依赖于自身（具有 `tests` 和 `docs` 额外内容）。

## 需求文件

*Requirements 文件*是带有每行依赖规范的纯文本文件（Example 4-8）。此外，它们可以包含 URL 和路径，可选地以`-e`为前缀进行可编辑安装，以及全局选项，如`-r`用于包含另一个 requirements 文件或`--index-url`用于使用除 PyPI 以外的包索引。该文件格式还支持 Python 风格的注释（以`#`字符开头）和行继续（以`\`字符结尾）。

##### 示例 4-8\. 一个简单的 requirements.txt 文件。

```py
pytest>=8.1.1
pytest-sugar>=1.0.0
sphinx>=7.2.6
```

您可以使用 pip 或 uv 安装 requirements 文件中列出的依赖项。

```py
$ uv pip install -r requirements.txt

```

根据惯例，一个需求文件通常命名为*requirements.txt*。但是，变种是很常见的。你可能有一个*dev-requirements.txt*用于开发依赖，或者一个*requirements*目录，每个依赖组有一个文件（Example 4-9）。

##### 示例 4-9\. 使用 requirements 文件指定开发依赖。

```py
# requirements/tests.txt
-e . ![1](img/1.png)
pytest>=8.1.1
pytest-sugar>=1.0.0

# requirements/docs.txt
sphinx>=7.2.6 ![2](img/2.png)

# requirements/dev.txt
-r tests.txt ![3](img/3.png)
-r docs.txt
```

![1](img/#co_dependency_management_CO5-1)

*tests.txt*文件需要项目的可编辑安装，因为测试套件需要导入应用程序模块。

![2](img/#co_dependency_management_CO5-2)

*docs.txt*文件不需要项目。（这是假设您仅从静态文件构建文档。如果您使用`autodoc` Sphinx 扩展从代码中的文档字符串生成 API 文档，则还需要在此处添加项目。）

![3](img/#co_dependency_management_CO5-3)

*dev.txt*文件包含其他 requirements 文件。

###### 注意。

如果使用`-r`包含其他 requirements 文件，则它们的路径相对于包含文件进行评估。相比之下，依赖项的路径是相对于您当前的目录进行评估的，通常是项目目录。

创建并激活虚拟环境，然后运行以下命令以安装开发依赖项并运行测试套件：

```py
$ uv pip install -r requirements/dev.txt
$ py -m pytest

```

Requirements 文件不是项目元数据的一部分。您可以通过版本控制系统与其他开发人员共享它们，但它们对于您的用户来说是不可见的。对于开发依赖项来说，这正是您想要的。此外，requirements 文件不会隐式地将项目包含在依赖项中。这减少了不需要安装项目的所有任务所需的时间。

Requirements 文件也有缺点。它们不是一个打包的标准，也不太可能成为一个——requirements 文件的每一行基本上是传递给`pip install`的一个参数。“pip 会做什么”可能仍然是 Python 打包中许多边缘情况的潜规则，但社区标准越来越多地取代了它。另一个缺点是，与*pyproject.toml*中的表格相比，这些文件在项目中造成了混乱。

如上所述，Python 项目管理器允许你在*pyproject.toml*中声明依赖组，超出了项目元数据—​Rye、Hatch、PDM 和 Poetry 都提供了这一功能。查看第五章以了解 Poetry 的依赖组描述。

# 锁定依赖关系

你已经在本地环境或持续集成（CI）中安装了依赖项，并运行了测试套件和其他检查。一切看起来都很好，你准备部署你的代码了。但是，如何在生产环境中安装与你运行检查时使用的相同包？

在开发和生产中使用不同的包装载具有后果。生产可能最终会得到一个与你的代码不兼容的包，或者有缺陷或安全漏洞的包，甚至—​在最坏的情况下—​被攻击者劫持的包。如果你的服务曝光度很高，这种情况令人担忧—​而且它可能涉及依赖树中的任何包，而不仅仅是你直接导入的包。

###### 警告

*供应链攻击*通过针对其第三方依赖项来渗透系统。例如，2022 年，“JuiceLedger”威胁行动者通过网络钓鱼活动篡改了合法的 PyPI 项目并上传了恶意包装载。⁶

环境在给定相同依赖规范的情况下最终以不同包装载的原因有很多。其中大部分原因可以归类为两类：上游更改和环境不匹配。首先，如果可用包的集合发生上游更改，你可能会得到不同的包：

+   在你部署之前发布了新版本。

+   为现有版本上传了一个新的工件。例如，当新的 Python 版本发布时，维护者有时会上传额外的轮子。

+   维护者删除或取消发布或工件。*取消发布*是一种软删除，它会隐藏文件以防止依赖解析，除非你明确请求它。

其次，如果你的开发环境与生产环境不匹配，你可能会得到不同的包装载：

+   环境标记在目标解释器上的评估有所不同（参见“环境标记”）。例如，生产环境可能使用一个旧的 Python 版本，需要像`importlib-metadata`这样的后移植。

+   轮子兼容性标签可能会导致安装程序为同一个包选择不同的轮子（参见“轮子兼容性标签”）。例如，如果你在配有苹果硅芯的 Mac 上开发，而生产环境使用 x86-64 架构的 Linux，则可能会发生这种情况。

+   如果发布不包括目标环境的轮子，安装程序会即时从源分布（sdist）构建它。当新的 Python 版本推出时，扩展模块的轮子通常会滞后。

+   如果环境不使用相同的安装程序（或者使用不同版本的相同安装程序），每个安装程序可能会以不同的方式解析依赖关系。例如，`uv` 使用 PubGrub 算法进行依赖关系解析，⁷ 而 `pip` 使用回溯解析器用于 Python 包，`resolvelib`。

+   工具配置或状态也可能导致不同的结果——例如，您可能从不同的软件包索引或本地缓存中安装。

您需要一种定义应用程序所需确切包集合的方法，并且您希望其环境是该包清单的确切映像。这个过程称为*锁定*，或者*固定*，列在*锁定文件*中的项目依赖项。

到目前为止，我已经谈论了为了可靠和可复制的部署而锁定依赖关系。锁定在开发过程中也很有益处，无论是应用程序还是库。通过与团队和贡献者共享锁定文件，您使每个开发人员在运行测试套件、构建文档或执行其他任务时使用相同的依赖项。在强制检查中使用锁定文件可避免出现在本地通过后，在 CI 中检查失败的情况。为了获得这些好处，锁定文件还必须包含开发依赖项。

就目前而言，Python 缺乏用于锁定文件的打包标准——尽管这个话题正在积极考虑中。⁸ 与此同时，许多 Python 项目管理器，如 Poetry、PDM 和 pipenv，已经实现了自己的锁定文件格式；而其他一些项目，如 Rye，则使用要求文件来锁定依赖项。

在本节中，我将介绍使用要求文件锁定依赖项的两种方法：*冻结*和*编译要求*。在第五章中，我将描述 Poetry 的锁定文件。

## 使用 pip 和 uv 冻结要求

要求文件是锁定依赖项的流行格式。它们允许您将依赖信息与项目元数据分开。Pip 和 uv 可以从现有环境生成这些文件：

```py
$ uv pip install .
$ uv pip freeze
anyio==4.3.0
certifi==2024.2.2
h11==0.14.0
h2==4.1.0
hpack==4.0.0
httpcore==1.0.4
httpx==0.27.0
hyperframe==6.0.1
idna==3.6
markdown-it-py==3.0.0
mdurl==0.1.2
pygments==2.17.2
random-wikipedia-article @ file:///Users/user/random-wikipedia-article
rich==13.7.1
sniffio==1.3.1

```

对环境中安装的软件包进行清单的操作被称为*冻结*。将列表存储在*requirements.txt*中，并将文件提交到源代码控制——只有一个更改：将文件 URL 替换为当前目录的点。这样，只要您在项目目录内，就可以在任何地方使用要求文件。

当将项目部署到生产环境时，您可以像这样安装项目及其依赖项：

```py
$ uv pip install -r requirements.txt
```

假设您的开发环境使用的是最新的解释器，那么要求文件中不会列出 `importlib-metadata`——该库仅在 Python 3.8 之前才需要。如果您的生产环境运行的是古老的 Python 版本，您的部署将会失败。这里有一个重要的教训：在与生产环境匹配的环境中锁定您的依赖项。

###### 小贴士

锁定依赖关系与用于生产的相同 Python 版本、Python 实现、操作系统和处理器架构。如果部署到多个环境，请为每个环境生成一个需求文件。

冻结需求有一些限制。首先，每次刷新需求文件时都需要安装依赖项。其次，如果临时安装一个软件包并忘记从头开始创建环境，则很容易意外污染需求文件。第三，冻结不允许记录软件包哈希 —— 它仅仅是对环境进行清单的记录，而环境不记录安装在其中的软件包的哈希（我将在下一节介绍软件包哈希）。

## 使用 pip-tools 和 uv 编译需求

pip-tools 项目使您能够在不受这些限制的情况下锁定依赖关系。您可以直接从 *pyproject.toml* 编译需求，而无需安装软件包。在底层，pip-tools 利用 pip 及其依赖解析器。

Pip-tools 提供两个命令：`pip-compile`，从依赖规范创建需求文件；`pip-sync`，将需求文件应用到现有环境。uv 工具提供了这两个命令的替代方案：`uv pip compile` 和 `uv pip sync`。

在与项目目标环境匹配的环境中运行 `pip-compile`。如果使用 pipx，请指定目标 Python 版本：

```py
$ pipx run --python=3.12 --spec=pip-tools pip-compile
```

默认情况下，`pip-compile` 从 *pyproject.toml* 读取，并写入 *requirements.txt*。您可以使用 `--output-file` 选项指定不同的目标。该工具还将需求打印到标准错误，除非您指定 `--quiet` 关闭终端输出。

Uv 要求您明确指定输入和输出文件：

```py
$ uv pip compile --python-version=3.12 pyproject.toml -o requirements.txt
```

Pip-tools 和 uv 为每个依赖包注释文件，指示依赖关系以及用于生成文件的命令。与 `pip freeze` 的输出还有一个区别：编译后的需求文件不包括您自己的项目。您需要在应用需求文件后单独安装它。

需求文件允许您为每个依赖项指定软件包哈希。这些哈希为您的部署增加了另一层安全性：它们使您只能在生产中安装经过审查的包装成品。选项 `--generate-hashes` 包含需求文件中每个软件包的 SHA256 哈希。例如，以下是 `httpx` 发布的 sdist 和 wheel 文件的哈希：

```py
httpx==0.27.0 \
--hash=sha256:71d5465162c13681bff01ad59b2cc68dd838ea1f10e51574bac27103f00c91a5 \
--hash=sha256:a0cb88a46f32dc874e04ee956e4c2764aba2aa228f650b06788ba6bda2962ab5
```

软件包哈希使安装更加确定性和可重现。它们还是需要筛选每个进入生产的工件的组织中的重要工具。验证软件包的完整性可以防止“中间人”攻击，在此攻击中，威胁行为者拦截软件包下载以提供已篡改的工件。

哈希值还有一个副作用，即 pip 拒绝安装没有哈希值的软件包：要么所有软件包都有哈希值，要么没有。因此，哈希值保护您免受安装未列在需求文件中的文件的影响。

在目标环境中使用 pip 或 uv 安装需求文件，然后再安装项目本身。您可以通过几个选项加固安装：选项 `--no-deps` 确保您只安装需求文件中列出的软件包，选项 `--no-cache` 防止安装程序重复使用下载或本地构建的文件。

```py
$ uv pip install -r requirements.txt
$ uv pip install --no-deps --no-cache .

```

定期更新依赖项。对于生产环境中运行的成熟应用程序，每周更新一次可能是可以接受的。对于正在积极开发的项目，每天更新甚至在发布后立即更新可能更合适。Dependabot 和 Renovate 等工具可帮助完成这些工作：它们在您的存储库中打开拉取请求，以自动升级依赖项。

如果您不定期升级依赖项，您可能会被迫在时间紧迫的情况下进行“大爆炸”升级。单个安全漏洞可能会迫使您将项目移植到多个软件包的主要版本以及 Python 本身的最新版本。

您可以一次性升级所有依赖项，也可以逐个依赖项升级。使用 `--upgrade` 选项将所有依赖项升级到它们的最新版本，或者使用 `--upgrade-package` 选项（`-P`）传递特定软件包。

例如，这是您如何升级 Rich 到最新版本的方式：

```py
$ uv pip compile -p 3.12 pyproject.toml -o requirements.txt -P rich

```

到目前为止，您已从头开始创建了目标环境。您还可以使用 `pip-sync` 同步目标环境与更新后的需求文件。为此不要在目标环境中安装 pip-tools：其依赖可能与您项目的依赖发生冲突。而是像您在 `pip-compile` 中使用 pipx 一样，使用 `--python-executable` 选项将 pip-sync 指向目标解释器：

```py
$ pipx run --spec=pip-tools pip-sync --python-executable=.venv/bin/python
```

该命令会删除项目本身，因为它没有列在需求文件中。在同步后重新安装它：

```py
$ .venv/bin/python -m pip install --no-deps --no-cache .
```

Uv 默认使用 *.venv* 中的环境，因此您可以简化这些命令：

```py
$ uv pip sync requirements.txt
$ uv pip install --no-deps --no-cache .

```

在 “开发依赖项” 中，您看到了声明开发依赖项的两种方式：extras 和需求文件。Pip-tools 和 uv 都支持它们作为输入。如果您在 `dev` extra 中跟踪开发依赖项，请像这样生成 *dev-requirements.txt* 文件：

```py
$ uv pip compile --extra=dev pyproject.toml -o dev-requirements.txt

```

如果您有更精细化的 extras，流程是相同的。您可能希望将需求文件存储在 *requirements* 目录中，以避免混乱。

如果您将开发依赖项指定为需求文件而不是 extras，在此顺序编译每个文件。按照惯例，输入需求使用 *.in* 扩展名，而输出需求使用 *.txt* 扩展名（示例 4-10）。

##### 示例 4-10\. 开发依赖项的输入需求

```py
# requirements/tests.in
pytest>=8.1.1
pytest-sugar>=1.0.0

# requirements/docs.in
sphinx>=7.2.6

# requirements/dev.in
-r tests.in
-r docs.in
```

与 示例 4-9 不同，输入的需求列表不包括项目本身。如果包括，输出的需求会包含项目的路径—每位开发者的路径可能不同。相反，将 *pyproject.toml* 与输入需求一起传递以锁定整套依赖项：

```py
$ uv pip compile requirements/tests.in pyproject.toml -o requirements/tests.txt
$ uv pip compile requirements/docs.in -o requirements/docs.txt
$ uv pip compile requirements/dev.in pyproject.toml -o requirements/dev.txt

```

安装输出需求后，请记得安装项目。

到底为什么要编译 *dev.txt* 呢？它不能仅包括 *docs.txt* 和 *tests.txt* 吗？如果你分别安装已锁定的需求，它们可能会冲突。让依赖解析器看到完整的情况。如果你传递所有输入需求，它将给你一个一致的依赖树作为回报。

表 4-3 总结了本章中您看到的 `pip-compile` 的命令行选项：

表 4-3\. `pip-compile` 的选定命令行选项

| 选项 | 描述 |
| --- | --- |
| `--generate-hashes` | 为每个打包工件包含 SHA256 哈希值 |
| `--output-file` | 指定目标文件 |
| `--quiet` | 不要将需求打印到标准错误输出 |
| `--upgrade` | 将所有依赖项升级到它们的最新版本 |
| `--upgrade-package=*<package>*` | 将特定包升级到其最新版本 |
| `--extra=*<extra>*` | 在 *pyproject.toml* 中包含给定额外依赖项 |

# 摘要

在本章中，您学习了如何使用 *pyproject.toml* 声明项目依赖关系，以及如何使用额外项或需求文件声明开发依赖关系。您还学习了如何使用 pip-tools 锁定依赖项以实现可靠的部署和可重现的检查。在下一章中，您将看到项目管理器 Poetry 如何使用依赖组和锁文件来帮助依赖管理。

¹ 从更广泛的意义上讲，项目的依赖包括所有用户运行其代码所需的所有软件包—包括解释器、标准库、第三方包和系统库。Conda 和像 APT、DNF 和 Homebrew 这样的发行级包管理器支持这种泛化的依赖概念。

² Henry Schreiner: [“是否应使用上限版本约束？”，](https://iscinumpy.dev/post/bound-version-constraints/) 2021 年 12 月 9 日。

³ 为简单起见，代码不处理多个作者—头部最终显示的作者未定义。

⁴ Robert Collins: [“PEP 508 – Python 软件包的依赖规范”，](https://peps.python.org/pep-0508/) 2015 年 11 月 11 日。

⁵ Stephen Rosen: [“PEP 735 – pyproject.toml 中的依赖组”，](https://peps.python.org/pep-0735/) 2023 年 11 月 20 日。

⁶ Dan Goodin: [“PyPI 供应链攻击背后的行动者自 2021 年末以来一直活跃,”](https://arstechnica.com/information-technology/2022/09/actors-behind-pypi-supply-chain-attack-have-been-active-since-late-2021/) 2022 年 9 月 2 日。

⁷ Natalie Weizenbaum: [“PubGrub: 下一代版本解决方案,”](https://nex3.medium.com/pubgrub-2fb6470504f) 2018 年 4 月 2 日

⁸ Brett Cannon: [“再次谈论锁定文件（但这次包括 sdists！）,”](https://discuss.python.org/t/lock-files-again-but-this-time-w-sdists/46593) 2024 年 2 月 22 日。

⁹ 卸载软件包并不足够：安装可能会对您的依赖树产生副作用。例如，它可能会升级或降级其他软件包，或者引入额外的依赖关系。
