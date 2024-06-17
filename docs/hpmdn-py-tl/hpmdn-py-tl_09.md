# 第六章：使用 pytest 进行测试

如果你回想起编写你的第一个程序时的经历，你可能会想起一个常见的情景：你有一个想法，认为程序可以帮助解决实际任务，然后花了大量时间从头到尾编写它，最后当你运行它时，却看到一屏幕都是令人沮丧的错误消息。或者更糟糕的是，它给出了微妙错误的结果。

我们从这样的经历中学到了一些经验教训。其中之一是从简单开始，并在迭代过程中保持简单。另一个教训是早期和重复测试。最初，这可能只是手动运行程序并验证其是否按预期工作。后来，如果将程序分解为更小的部分，可以独立且自动地测试这些部分。作为副作用，程序也更容易阅读和处理。

在本章中，我将谈论如何测试可以帮助您及时和一致地产生价值。良好的测试相当于对您拥有的代码的可执行规范。它们使您摆脱了团队或公司的制度化知识，通过及时反馈您的变更加快了开发速度。

第三方测试框架 [pytest](https://docs.pytest.org/) 已经成为 Python 世界的一种事实标准。使用 pytest 编写的测试简单易读：你编写的大多数测试都像没有框架一样，使用基本的语言原语如函数和断言。尽管框架简单，但通过测试固件和参数化测试等概念，它强大且富有表现力。Pytest 是可扩展的，并且配备了丰富的插件生态系统。

###### 注意

Pytest 起源于 PyPy 项目，这是一个用 Python 编写的 Python 解释器。早期，PyPy 的开发人员致力于一个名为 `std` 的单独标准库，后来更名为 `py`。其测试模块 `py.test` 成为了一个名为 `pytest` 的独立项目。

# 编写测试

示例 6-1 重新访问了来自第 3 章的维基百科示例。该程序非常简单，但你可能不清楚如何为其编写测试。`main` 函数没有输入和输出，只有副作用，比如向标准输出流写入内容。你会如何测试这样的函数？

##### 示例 6-1\. 来自 `random-wikipedia-article` 的 `main` 函数

```py
def main():
    with urllib.request.urlopen(API_URL) as response:
        data = json.load(response)

    print(data["title"], end="\n\n")
    print(textwrap.fill(data["extract"]))
```

让我们编写一个*端到端测试*，在子进程中运行程序并检查其是否完成且输出非空。端到端测试会像最终用户一样运行整个程序（示例 6-2）。

##### 示例 6-2\. 对 `random-wikipedia-article` 的测试

```py
import subprocess
import sys

def test_output():
    args = [sys.executable, "-m", "random_wikipedia_article"]
    process = subprocess.run(args, capture_output=True, check=True)
    assert process.stdout
```

###### 提示

使用 pytest 编写的测试是以 `test` 开头的函数。使用内置的 `assert` 语句来检查预期行为。Pytest 重写语言构造以提供丰富的错误报告，以防测试失败。

将示例 6-2 的内容放入一个名为*test_main.py*的文件中，放在一个名为*tests*的目录中。包含一个空的*__init__.py*文件将测试套件转换为一个可导入的包。这样可以使您模仿您正在测试的包的布局，¹并且可以选择导入具有测试实用工具的模块。

此时，您的项目应该按以下方式结构化：

```py
random-wikipedia-article
├── pyproject.toml
├── src
│   └── random_wikipedia_article
│       ├── __init__.py
│       └── __main__.py
└── tests
    ├── __init__.py
    └── test_main.py
```

# 管理测试依赖项

测试必须能够导入您的项目及其依赖项，因此您需要在项目环境中安装 pytest。例如，将一个`tests`的额外项添加到您的项目中：

```py
[project.optional-dependencies]
tests = ["pytest>=8.1.1"]
```

您现在可以在项目环境中安装 pytest：

```py
$ uv pip install -e ".[tests]"
```

或者，编译一个需求文件并同步您的环境：

```py
$ uv pip compile --extra=tests pyproject.toml -o dev-requirements.txt
$ uv pip sync dev-requirements.txt
$ uv pip install -e . --no-deps

```

如果您使用 Poetry，请使用`poetry add`将 pytest 添加到您的项目中：

```py
$ poetry add --group=tests "pytest>=8.1.1"
```

请在本章后面我要求您添加测试依赖项时参考这些步骤。

最后，让我们运行测试套件。如果您使用的是 Windows，在运行以下命令之前，请激活环境。

```py
$ py -m pytest
========================= test session starts ==========================
platform darwin -- Python 3.12.2, pytest-8.1.1, pluggy-1.4.0
rootdir: ...
collected 1 item

tests/test_main.py .                                              [100%]
========================== 1 passed in 0.01s ===========================

```

###### 提示

即使在 Poetry 项目中，也要使用`py -m pytest`。这既更短，也更安全，比`poetry run pytest`更安全。如果忘记将 pytest 安装到环境中，Poetry 将退回到您的全局环境中。（安全的变体将是`poetry run python -m pytest`。）

# 设计可测试性

为程序编写更精细的测试要困难得多。API 终点返回一个随机文章，那么测试应该期望*哪个*标题和摘要？每次调用都会向真实的维基百科 API 发送一个 HTTP 请求。这些网络往返将使测试套件变得极其缓慢——而且只有在您的计算机连接到互联网时才能运行测试。

Python 程序员在这种情况下有一系列工具可供使用。其中大部分涉及某种形式的*猴子补丁*，即在运行时替换函数或对象，以使代码更易于测试。例如，您可以通过将`sys.stdout`替换为一个写入内部缓冲区以供后续检查的文件样对象来捕获程序输出。您可以将`urlopen`替换为返回您喜欢的固定 HTTP 响应的函数。像`responses`、`respx`或`vcr.py`这样的库提供了在幕后猴子补丁 HTTP 机制的高级接口。更通用的方法使用标准的`unittest.mock`模块或 pytest 的`monkeypatch`装置。

###### 注意

*猴子补丁*一词是在 Zope 公司首创的用于在运行时替换代码的术语。最初，Zope 公司的人们将这项技术称为“游击补丁”，因为它不遵守常规的补丁提交规则。人们听到的是“大猩猩补丁”——很快，更精心制作的补丁就被称为“猴子补丁”。

虽然这些工具能够完成它们的任务，但我建议你专注于问题的根源：示例 6-1 没有关注点分离。一个函数既作为应用程序的入口点，又与外部 API 通信，并在控制台上呈现结果。这使得很难单独测试其功能。

该程序还缺乏抽象化，有两个方面。首先，在与其他系统交互时，如与维基百科 API 交互或写入终端时，它没有封装实现细节。其次，它的中心概念——维基百科文章——只是一个无定形的 JSON 对象：程序未以任何方式抽象其域模型，例如定义一个`Article`类。

示例 6-3 展示了一种使代码更易于测试的重构。虽然这个版本的程序更长，但它更清晰地表达了逻辑，并且更容易修改。好的测试不仅能捕获错误：它们还改善了代码的设计。

##### 示例 6-3\. 可测试性重构

```py
import sys ![1](img/1.png)
from dataclasses import dataclass

@dataclass
class Article:
    title: str = ""
    summary: str = ""

def fetch(url):
    with urllib.request.urlopen(url) as response:
        data = json.load(response)
    return Article(data["title"], data["extract"])

def show(article, file):
    summary = textwrap.fill(article.summary)
    file.write(f"{article.title}\n\n{summary}\n")

def main():
    article = fetch(API_URL)
    show(article, sys.stdout)
```

![1](img/#co_testing_with_pytest_CO1-1)

为了简洁起见，本章节中的示例仅在第一次使用时显示导入。

重构将`main`中的`fetch`和`show`函数提取出来。它还将`Article`类定义为这些函数的共同基础。让我们看看这些改变如何让你能够以隔离和可重复的方式测试程序的各个部分。

`show`函数接受任何类似文件的对象。虽然`main`传递了`sys.stdout`，测试可以传递一个`io.StringIO`实例以将输出存储在内存中。示例 6-4 使用这种技术来检查输出是否以换行符结束。最后的换行符确保输出不会延伸到下一个 shell 提示符。

##### 示例 6-4\. 测试`show`函数

```py
import io
from random_wikipedia_article import Article, show

def test_final_newline():
    article = Article("Lorem Ipsum", "Lorem ipsum dolor sit amet.")
    file = io.StringIO()
    show(article, file)
    assert file.getvalue().endswith("\n")
```

重构还有另一个好处：函数将实现隐藏在只涉及你的问题域——URL、文章、文件的接口后面。这意味着当你替换实现时，你的测试不太可能会失败。继续更改`show`函数以使用 Rich，就像示例 6-5 中所示那样。² 你不需要调整你的测试！

##### 示例 6-5\. 替换`show`的实现

```py
from rich.console import Console

def show(article, file):
    console = Console(file=file, width=72, highlight=False)
    console.print(article.title, style="bold", end="\n\n")
    console.print(article.summary)
```

实际上，测试的整个目的是在你进行此类更改后依然确保程序正常工作。另一方面，模拟和猴子补丁是脆弱的：它们将你的测试套件与实现细节捆绑在一起，使得今后更改程序变得越来越困难。

# Fixture 和参数化

这里还有一些`show`函数的其他属性，你可能会检查：

+   它应该包含标题和摘要的所有单词。

+   标题后应有一个空行。

+   摘要不应超过 72 个字符的行长。

每个 `show` 函数的测试都以设置输出缓冲区开始。你可以使用 fixture 来消除此代码重复。*Fixture* 是使用 `pytest.fixture` 装饰器声明的函数：

```py
@pytest.fixture
def file():
    return io.StringIO()
```

测试（及 fixture）可以通过包含与相同名称的函数参数使用 fixture。当 pytest 调用测试函数时，它传递 fixture 函数的返回值。让我们重写 Example 6-4 来使用该 fixture：

```py
def test_final_newline(file):
    article = Article("Lorem Ipsum", "Lorem ipsum dolor sit amet.")
    show(article, file)
    assert file.getvalue().endswith("\n")
```

###### 警告

如果忘记将参数 `file` 添加到测试函数中，会出现令人困惑的错误：`'function' object has no attribute 'write'`。这是因为现在名称 `file` 指向同一模块中的 fixture 函数。³

如果每个测试都使用相同的文章，你可能会遗漏一些边界情况。例如，如果一篇文章标题为空，你不希望程序崩溃。Example 6-6 对多篇文章使用 `@pytest.mark.parametrize` 装饰器运行测试。⁴

##### Example 6-6\. 对多篇文章运行测试

```py
articles = [
    Article(),
    Article("test"),
    Article("Lorem Ipsum", "Lorem ipsum dolor sit amet."),
    Article(
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit",
        "Nulla mattis volutpat sapien, at dapibus ipsum accumsan eu."
    ),
]

@pytest.mark.parametrize("article", articles)
def test_final_newline(article, file):
    show(article, file)
    assert file.getvalue().endswith("\n")
```

如果你以相同方式参数化许多测试，可以创建一个*参数化 fixture*，一个带有多个值的 fixture（参见 Example 6-7）。与之前一样，pytest 会对每篇文章在 `articles` 中运行测试一次。

##### Example 6-7\. 参数化 fixture，用于对多篇文章运行测试

```py
@pytest.fixture(params=articles)
def article(request):
    return request.param

def test_final_newline(article, file):
    show(article, file)
    assert file.getvalue().endswith("\n")
```

那么你在这里得到了什么？首先，你不需要为每个测试都添加 `@pytest.mark.parametrize` 装饰器。如果你的测试不都在同一个模块中，还有另一个优势：你可以将 fixture 放在名为 *conftest.py* 的文件中，在整个测试套件中使用而无需导入。

参数化 fixture 的语法有些晦涩。为了保持简单，我喜欢定义一个小助手：

```py
def parametrized_fixture(*params):
    return pytest.fixture(params=params)(lambda request: request.param)
```

使用 helper 简化 Example 6-7 中的 fixture。还可以从 Example 6-6 中内联 `articles` 变量：

```py
article = parametrized_fixture(Article(), Article("test"), ...)
```

###### 提示

如果你有一个使用 `unittest` 编写的测试套件，没有必要重写它以开始使用 pytest——pytest 也“说” `unittest`。立即使用 pytest 作为测试运行器，稍后逐步重写你的测试套件。

# Fixture 的高级技术

对于 `fetch` 函数，测试可以设置一个本地 HTTP 服务器并执行*往返*检查。这在 Example 6-8 中展示：你通过 HTTP 提供一个 `Article` 实例，从服务器获取文章，并检查提供和获取的实例是否相等。

##### Example 6-8\. 测试 `fetch` 函数（版本 1）

```py
def test_fetch(article):
    with serve(article) as url:
        assert article == fetch(url)
```

`serve` 辅助函数接受一篇文章并返回一个用于获取文章的 URL。更确切地说，它将 URL 包装在*上下文管理器*中，这是一个可以在 `with` 块中使用的对象。这允许 `serve` 在退出 `with` 块时进行清理——通过关闭服务器：

```py
from contextlib import contextmanager

@contextmanager
def serve(article):
    ... # start the server
    yield f"http://localhost:{server.server_port}"
    ... # shut down the server
```

您可以使用标准库中的`http.server`模块实现`serve`函数（示例 6-9）。不过，不必太担心细节。本章稍后将介绍`pytest-httpserver`插件，它将承担大部分工作。

##### 示例 6-9\. `serve`函数

```py
import http.server
import json
import threading

@contextmanager
def serve(article):
    data = {"title": article.title, "extract": article.summary}
    body = json.dumps(data).encode()

    class Handler(http.server.BaseHTTPRequestHandler): ![1](img/1.png)
        def do_GET(self):
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)

    with http.server.HTTPServer(("localhost", 0), Handler) as server: ![2](img/2.png)
        thread = threading.Thread(target=server.serve_forever, daemon=True) ![3](img/3.png)
        thread.start()
        yield f"http://localhost:{server.server_port}"
        server.shutdown()
        thread.join()
```

![1](img/#co_testing_with_pytest_CO2-1)

请求处理程序用 UTF-8 编码的 JSON 表示文章响应每个 GET 请求。

![2](img/#co_testing_with_pytest_CO2-2)

服务器仅接受本地连接。操作系统会随机分配端口号。

![3](img/#co_testing_with_pytest_CO2-3)

服务器在后台线程中运行。这样可以使控制权返回到测试中。

每次测试都启动和关闭 Web 服务器是昂贵的。将服务器转换为夹具是否有所帮助？乍一看，效果不大—​每个测试都获得其自己的夹具实例。但是，您可以指示 pytest 在整个测试会话期间仅创建一次夹具，使用*会话范围的夹具*：

```py
@pytest.fixture(scope="session")
def httpserver():
    ...
```

看起来更有希望，但是当测试完成时如何关闭服务器呢？到目前为止，您的夹具仅准备了一个测试对象并返回它。您不能在`return`语句之后运行代码。但是，在`yield`语句之后您可以运行代码—​因此 pytest 允许您将夹具定义为生成器。

*生成器夹具*准备一个测试对象，yield 它，并在结束时清理资源—​类似于上下文管理器。您可以像普通夹具一样使用它，该夹具返回其测试对象。Pytest 在幕后处理设置和拆卸阶段，并使用 yield 的值调用您的测试函数。

示例 6-10 使用生成器技术定义了`httpserver`夹具。

##### 示例 6-10\. `httpserver`夹具

```py
@pytest.fixture(scope="session")
def httpserver():
    class Handler(http.server.BaseHTTPRequestHandler):
        def do_GET(self):
            article = self.server.article ![1](img/1.png)
            data = {"title": article.title, "extract": article.summary}
            body = json.dumps(data).encode()
            ... # as before

    with http.server.HTTPServer(("localhost", 0), Handler) as server:
        thread = threading.Thread(target=server.serve_forever, daemon=True)
        thread.start()
        yield server
        server.shutdown()
        thread.join()
```

![1](img/#co_testing_with_pytest_CO3-1)

在示例 6-9 中，范围内没有`article`。相反，请求处理程序从服务器的`article`属性中访问它（详见下文）。

还有一个遗漏的部分：您需要定义`serve`函数。现在该函数依赖于`httpserver`夹具来完成其工作，因此您不能简单地将其定义为模块级别。让我们暂时将其移动到测试函数中（示例 6-11）。

##### 示例 6-11\. 测试`fetch`函数（版本 2）

```py
def test_fetch(article, httpserver):
    def serve(article):
        httpserver.article = article ![1](img/1.png)
        return f"http://localhost:{httpserver.server_port}"

    assert article == fetch(serve(article))
```

![1](img/#co_testing_with_pytest_CO4-1)

将文章存储在服务器中，以便请求处理程序可以访问它。

`serve` 函数不再返回上下文管理器，只是一个简单的 URL—​`httpserver` fixture 处理所有的设置和拆卸。但是你仍然可以做得更好。内部函数会使测试代码变得混乱—​还有 `fetch` 函数的所有其他测试。相反，让我们在其自己的 fixture 中定义 `serve`—​毕竟，fixtures 可以返回任何对象，包括函数（Example 6-12）。

##### Example 6-12\. `serve` fixture

```py
@pytest.fixture
def serve(httpserver): ![1](img/1.png)
    def f(article): ![2](img/2.png)
        httpserver.article = article
        return f"http://localhost:{httpserver.server_port}"
    return f
```

![1](img/#co_testing_with_pytest_CO5-1)

外部函数定义了一个 `serve` fixture，依赖于 `httpserver`。

![2](img/#co_testing_with_pytest_CO5-2)

内部函数就是你在测试中调用的 `serve` 函数。

多亏了 `serve` fixture，测试函数变成了一个单行代码（Example 6-13）。它也更快，因为你每个会话只启动和停止一次服务器。

##### Example 6-13\. 测试 `fetch` 函数（版本 3）

```py
def test_fetch(article, serve):
    assert article == fetch(serve(article))
```

你的测试与任何特定的 HTTP 客户端库无关。Example 6-14 替换了 `fetch` 函数的实现以使用 HTTPX。⁵ 这可能会破坏使用猴子补丁的任何测试—​但你的测试仍然会通过！

##### Example 6-14\. 替换 `fetch` 的实现

```py
import httpx

from importlib.metadata import metadata

USER_AGENT = "{Name}/{Version} (Contact: {Author-email})"

def fetch(url):
    fields = metadata("random-wikipedia-article")
    headers = {"User-Agent": USER_AGENT.format_map(fields)}

    with httpx.Client(headers=headers, http2=True) as client:
        response = client.get(url, follow_redirects=True)
        response.raise_for_status()
        data = response.json()

    return Article(data["title"], data["extract"])
```

# 使用插件扩展 pytest

正如你在“入口点”中看到的，pytest 的可扩展设计允许任何人贡献 pytest 插件并将其发布到 PyPI，⁶ 因此插件的丰富生态系统已经形成。你已经看到了 `pytest-sugar` 插件，它增强了 pytest 的输出，包括添加了一个进度条。在本节中，你将看到更多插件。

## pytest-httpserver 插件

[`pytest-httpserver`](https://pytest-httpserver.readthedocs.io/) 插件提供了一个更多功能且经过实战测试的 `httpserver` fixture，比 Example 6-10 更加灵活。让我们使用这个插件。

首先，将 `pytest-httpserver` 添加到你的测试依赖中。接下来，从你的测试模块中移除现有的 `httpserver` fixture。最后，更新 `serve` fixture 以使用该插件（Example 6-15）。

##### Example 6-15\. 使用 `pytest-httpserver` 的 `serve` fixture

```py
@pytest.fixture
def serve(httpserver):
    def f(article):
        json = {"title": article.title, "extract": article.summary}
        httpserver.expect_request("/").respond_with_json(json)
        return httpserver.url_for("/")
    return f
```

Example 6-15 配置服务器以响应对 `"/"` 的请求，并返回文章的 JSON 表示。该插件提供了远比这个用例更灵活的功能，例如，你可以添加自定义请求处理程序或者通过 HTTPS 进行通信。

## pytest-xdist 插件

随着你的测试套件的增长，你会寻找加快测试运行速度的方法。这里有一个简单的方法：利用所有的 CPU 核心。[`pytest-xdist`](https://pytest-xdist.readthedocs.io/) 插件在每个处理器上生成一个工作进程，并随机分发测试。这种随机化也有助于检测测试之间的隐藏依赖关系。

将`pytest-xdist`添加到您的测试依赖项并更新您的环境。使用选项`--numprocesses`或`-n`指定工作进程数。指定`auto`以使用系统上的所有物理核心：

```py
$ py -m pytest -n auto

```

## factory-boy 和 faker 库

在“夹具和参数化”中，您硬编码了测试运行的文章。让我们避免这种样板代码—​它使您的测试难以维护。

相反，[`factory-boy`](https://factoryboy.readthedocs.io/)库允许您为测试对象创建工厂。您可以使用序列号生成具有可预测属性的对象批次。或者，您可以使用[`faker`](https://faker.readthedocs.io/)库随机填充属性。

将`factory-boy`添加到您的测试依赖项并更新您的环境。示例 6-16 定义了一个用于随机文章的工厂，并为`article`夹具创建了十篇文章的批次。（如果您想看到`pytest-xdist`的效果，请增加文章数并使用`-n auto`运行 pytest。）

##### 示例 6-16\. 使用`factory-boy`和`faker`创建一批文章

```py
from factory import Factory, Faker

class ArticleFactory(Factory):
    class Meta:
        model = Article ![1](img/1.png)

    title = Faker("sentence") ![2](img/2.png)
    summary = Faker("paragraph")

article = parametrized_fixture(*ArticleFactory.build_batch(10)) ![3](img/3.png)
```

![1](img/#co_testing_with_pytest_CO6-1)

使用`Meta.model`指定测试对象的类。

![2](img/#co_testing_with_pytest_CO6-2)

标题使用随机句子，摘要使用随机段落。

![3](img/#co_testing_with_pytest_CO6-3)

使用`build_batch`方法生成一批文章。

这个简化的工厂不能很好地处理边界情况。对于真实的应用程序，您应该包括空和非常大的字符串，以及控制字符等异常字符。另一个可以让您探索可能输入搜索空间的优秀测试库是[`hypothesis`](https://hypothesis.readthedocs.io/)。

## 其他插件

Pytest 插件执行各种功能（表 6-1）。这些功能包括并行或随机顺序执行测试，以自定义方式呈现或报告测试结果，以及与框架和其他工具集成。许多插件提供有用的夹具，例如用于与外部系统交互或创建测试双。

表 6-1\. 一些 pytest 插件的选择

| 插件 | 类别 | 描述 | 选项 |
| --- | --- | --- | --- |
| `pytest-xdist` | 执行 | 将测试分布到多个 CPU 上 | `--numprocesses` |
| `pytest-sugar` | 展示 | 用进度条增强输出 |  |
| `pytest-icdiff` | 展示 | 在测试失败时显示着色差异 |  |
| `anyio` | 框架 | 使用异步测试与 asyncio 和 trio |  |
| `pytest-httpserver` | 伪服务器 | 启动带有预设响应的 HTTP 服务器 |  |
| `pytest-factoryboy` | 伪数据 | 将工厂转换为夹具 |  |
| `pytest-datadir` | 存储 | 在测试套件中访问静态数据 |  |
| `pytest-cov` | 覆盖率 | 使用 Coverage.py 生成覆盖率报告 | `--cov` |
| `xdoctest` | 文档 | 运行来自文档字符串的代码示例 | `--xdoctest` |
| `typeguard` | 类型检查 | 在运行时对你的代码进行类型检查 | `--typeguard-packages` |

###### 提示

在 PyPI 上找到每个项目：`https://pypi.org/project/<name>`。项目主页在导航栏的*项目链接*下可用。

# 总结

在本章中，你学会了如何使用 pytest 测试你的 Python 项目：

+   测试是运行你的代码并使用内置的`assert`来检查预期行为的函数。将它们的名称前缀—和包含模块的名称—与`test_`，pytest 会自动发现它们。

+   固定装置是设置和撤销测试对象的函数或生成器；使用`@pytest.fixture`装饰器声明它们。你可以通过包含与装置名称相同的参数在测试中使用装置。

+   pytest 的插件可以提供有用的固定装置，以及修改测试执行，增强报告等等。

优秀软件的主要特征之一是易于变更，因为任何实际使用的代码都必须适应不断变化的需求和环境。测试以多种方式简化变更：

+   它们将软件设计引向松散耦合的构建模块，你可以单独测试：减少依赖意味着减少变更障碍。

+   它们记录并强制执行预期行为。这使你有自由和信心不断重构你的代码库—随着它的增长和转变保持可维护性。

+   它们通过早期检测缺陷来降低变更成本。你越早发现问题，修复根本原因和开发修复方案的成本就越低。

端到端测试能够高度确保功能按设计方式工作—但它们速度慢、不稳定，并且很难分离失败的原因。让大部分测试都是单元测试。避免猴子补丁以打破代码的依赖性以进行可测试性。而是，将你的应用程序核心与 I/O、外部系统和第三方框架解耦。良好的软件设计和坚实的测试策略将使你的测试套件速度飞快且对变更具有弹性。

本章侧重于工具方面的事情，但良好的测试实践远不止这些。幸运的是，其他人已经写了一些关于这个主题的精彩文章。以下是我一直喜爱的三本书：

+   Kent Beck，《测试驱动开发：通过示例》（伦敦：Pearson，2002）。

+   Michael Feathers，《与遗留代码高效工作》（伦敦：Pearson，2004）。

+   Harry Percival 和 Bob Gregory，《Python 中的架构模式》（Sebastopol：O’Reilly，2020）。

如果你想了解如何使用 pytest 进行测试的所有内容，请阅读 Brian 的书：

+   Brian Okken，《Python Testing with pytest: Simple, Rapid, Effective, and Scalable》，第二版（Raleigh: The Pragmatic Bookshelf，2022）。

¹ 大型包可能有相同名称的模块——比如，`gizmo.foo.registry` 和 `gizmo.bar.registry`。在 pytest 的默认[导入模式](https://docs.pytest.org/en/7.1.x/explanation/pythonpath.html#import-modes)下，测试模块必须具有唯一的完全限定名称——因此，你必须将 `test_registry` 模块放置在单独的 `tests.foo` 和 `tests.bar` 包中。

² 记得按照 “为项目指定依赖项” 中描述的方式将 Rich 添加到你的项目中。如果你使用 Poetry，请参考 “管理依赖项”。

³ 我的审阅员 Hynek 建议一种避免这种陷阱并获得习惯性的 `NameError` 的技巧。诀窍是使用 `@pytest.fixture(name="file")` 明确地为 fixture 命名。这样你就可以使用一个私有名称来命名函数，比如 `_file`，它不会与参数发生冲突。

⁴ 注意略有不同的拼写变体 *parametrize*，而不是 *parameterize*。

⁵ 记得将 `httpx[http2]` 添加为你项目的依赖。

⁶ [cookiecutter-pytest-plugin](https://github.com/pytest-dev/cookiecutter-pytest-plugin) 模板为你提供了一个稳固的项目结构，用于编写你自己的插件。

⁷ *测试替身* 是测试中使用的各种对象的总称，用以代替生产代码中使用的真实对象。Martin Fowler 在 2007 年 1 月 2 日发表的文章 [“Mocks Aren’t Stubs”](https://martinfowler.com/articles/mocksArentStubs.html) 提供了一个很好的概述。
