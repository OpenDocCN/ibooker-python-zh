# 第七章：使用 Coverage.py 测量覆盖率

当您的测试通过时，您对代码更改的信心有多大？

如果您将测试视为检测缺陷的一种方式，您可以描述它们的灵敏度和特异性。

您的测试套件的*灵敏度*是代码存在缺陷时测试失败的概率。如果大部分代码未经测试，或者测试未检查预期行为，则灵敏度低。

您测试的*特异性*是如果代码没有缺陷，则它们通过的概率。如果您的测试*不稳定*（它们偶尔失败）或*脆弱*（更改实现细节时失败），则特异性低。人们总是不重视失败的测试。不过，本章并非讨论特异性。

有一种极有效的策略可以提高您测试的灵敏度：在添加或更改行为时，在能够使其通过的代码之前编写一个失败的测试。如果这样做，您的测试套件将捕捉您对代码的期望。

另一个有效的策略是使用您期望在真实世界中遇到的各种输入和环境约束测试您的软件。覆盖函数的边缘情况，如空列表或负数。测试常见的错误场景，而不仅仅是“快乐路径”。

*代码覆盖率*是测试套件执行代码的程度的一种衡量。完全覆盖并不保证高灵敏度：如果您的测试覆盖了代码中的每一行，仍可能存在错误。尽管如此，它是一个上限。如果您的代码覆盖率为 80%，那么即使有多少错误悄然而至，也有 20%的代码将*永远*不会触发测试失败。它还是一个适合自动化工具的量化指标。这两个属性使覆盖率成为灵敏度的有用代理。

简而言之，覆盖率工具在运行代码时记录每行代码。完成后，它们报告执行的代码行相对于整个代码库的总百分比。

覆盖率工具不仅限于测量测试覆盖率。例如，代码覆盖率可帮助您查找大型代码库中 API 端点使用的模块。或者，您可以使用它来确定代码示例在多大程度上记录了项目。

在本章中，我将解释如何使用 Coverage.py 测量代码覆盖率，这是 Python 的一种覆盖率工具。在本章的主要部分中，您将学习如何安装、配置和运行 Coverage.py，以及如何识别源代码中缺失的行和控制流中缺失的分支。我将解释如何在多个环境和进程中测量代码覆盖率。最后，我将讨论您应该追求的代码覆盖率目标以及如何实现这些目标。

在 Python 中如何进行代码覆盖率测量？解释器允许您注册一个回调—​一个*跟踪函数*—​使用函数`sys.settrace`。从那时起，解释器在执行每行代码时—​以及在某些其他情况下，如进入或返回函数或引发异常时—​调用回调。覆盖工具注册一个跟踪函数，记录每个执行的源代码行到本地数据库中。

# 使用 Coverage.py

[Coverage.py](https://coverage.readthedocs.io/)是一个成熟且广泛使用的 Python 代码覆盖工具。创建于 20 多年前—​早于 PyPI 和 setuptools—​并自那时以来一直活跃维护，它已在 Python 2.1 及以后的每个解释器上测量覆盖率。

将`coverage[toml]`添加到您的测试依赖项（参见“管理测试依赖项”）。`toml`额外允许 Coverage.py 从*pyproject.toml*文件中读取配置，适用于旧版本的解释器。自 Python 3.11 起，标准库包括用于解析 TOML 文件的`tomllib`模块。

测量覆盖率是一个两步过程。首先，您使用`coverage run`在测试运行期间收集覆盖数据。其次，您使用`coverage report`从数据中编制汇总报告。每个命令在*pyproject.toml*文件的`tool.coverage`下都有一个表格。

首先配置您想要测量的包—​它让 Coverage.py 报告从未在执行中显示过的模块，就像之前的`__main__`模块一样。（即使没有此设置，它也不会淹没您关于标准库的报告。）指定您的顶级导入包，以及测试套件：

```py
[tool.coverage.run]
source = ["random_wikipedia_article", "tests"]
```

###### 提示

为测试套件测量代码覆盖率可能看起来很奇怪—​但您应该始终这样做。它在测试未运行时会提醒您，并帮助您识别其中的不可达代码。将您的测试视为任何其他代码一样对待。¹

您可以通过 Python 脚本调用`coverage run`，然后跟上其命令行参数。或者，您可以使用其`-m`选项与可导入模块。使用第二种方法—​确保您从当前环境运行 pytest：

```py
$ py -m coverage run -m pytest

```

运行此命令后，您会在当前目录中找到一个名为*.coverage*的文件。Coverage.py 使用它来存储测试运行期间收集的覆盖数据。²

覆盖报告显示代码覆盖率的整体百分比，以及每个源文件的详细情况。使用`show_missing`设置还可以包括缺少覆盖的语句的行号：

```py
[tool.coverage.report]
show_missing = true
```

运行`coverage report`以在终端中显示覆盖报告：

```py
$ py -m coverage report
Name                                       Stmts   Miss  Cover   Missing
------------------------------------------------------------------------
src/random_wikipedia_article/__init__.py      26      2    92%   38-39
src/random_wikipedia_article/__main__.py       2      2     0%   1-3
tests/__init__.py                              0      0   100%
tests/test_main.py                            33      0   100%
------------------------------------------------------------------------
TOTAL                                         61      4    93%

```

总体而言，您的项目覆盖率为 93%—4 条语句在测试中从未出现。测试套件本身具有完整的覆盖率，正如您所期望的那样。

让我们仔细看看那些缺失的语句。覆盖率报告中的`Missing`列按行号列出它们。您可以使用代码编辑器显示带有行号的源代码，或者在 Linux 和 macOS 上使用标准的`cat -n`命令。同样，整个`__main__`模块的覆盖率也没有：

```py
   1  from random_wikipedia_article import main  # missing
   2
   3  main()                                     # missing
```

*__init__.py*中缺失的行对应于`main`函数的主体：

```py
  37  def main():
  38      article = fetch(API_URL)   # missing
  39      show(article, sys.stdout)  # missing
```

这很令人惊讶——示例 6-2 中的端到端测试运行整个程序，因此所有这些行肯定都在进行测试。暂时禁用对`__main__`模块的覆盖率测量：

```py
[tool.coverage.run]
omit = ["*/__main__.py"]
```

您可以使用特殊注释来排除`main`函数：

```py
def main():  # pragma: no cover
    article = fetch(API_URL)
    show(article, sys.stdout)
```

如果感觉这样做有些作弊，请继续阅读“在子进程中进行测量”，在那里您将重新启用这些行的覆盖率测量。

如果再次运行这两个步骤，Coverage.py 将报告完整的代码覆盖率。确保您能够注意到您的测试没有执行到的任何行。再次配置 Coverage.py，如果百分比再次低于 100%，则失败：

```py
[tool.coverage.report]
fail_under = 100
```

# 分支覆盖率

如果一篇文章有空摘要，`random-wikipedia-article`会打印尾随空行（哎呀）。这些空摘要很少见，但它们确实存在，这应该是一个快速修复。示例 7-1 修改`show`以仅打印非空摘要。

##### 示例 7-1\. 仅在`show`中打印非空摘要

```py
def show(article, file):
    console = Console(file=file, width=72, highlight=False)
    console.print(article.title, style="bold", end="\n\n")
    if article.summary:
        console.print(article.summary)
```

有趣的是，覆盖率保持在 100%——尽管您没有先编写测试。

默认情况下，Coverage.py 测量*语句覆盖率*——解释器在测试期间执行的模块中语句的百分比。如果摘要不为空，则会执行函数中的每条语句。

另一方面，测试只执行了函数中两条代码路径中的一条——它们从未跳过`if`体。Coverage.py 还支持*分支覆盖率*，它查看代码中所有语句之间的所有转换，并测量测试期间遍历这些转换的百分比。您应该始终启用它，因为它比语句覆盖更精确：

```py
[tool.coverage.run]
branch = true
```

重新运行测试，您将看到 Coverage.py 标记了从第 34 行的`if`语句到函数退出的缺失转换：

```py
$ py -m coverage run -m pytest
$ py -m coverage report
Name                  Stmts   Miss Branch BrPart  Cover   Missing
-----------------------------------------------------------------
src/.../__init__.py      24      0      6      1    97%   34->exit
tests/__init__.py         0      0      0      0   100%
tests/test_main.py       33      0      6      0   100%
-----------------------------------------------------------------
TOTAL                    57      0     12      1    99%
Coverage failure: total of 99 is less than fail-under=100

```

示例 7-2 使覆盖率恢复到 100%。它包括一个具有空摘要的文章，并添加了对尾随空行缺失测试。

##### 示例 7-2\. 测试空摘要的文章

```py
article = parametrized_fixture(
    Article("test"), *ArticleFactory.build_batch(10)
)

def test_trailing_blank_lines(article, file):
    show(article, file)
    assert not file.getvalue().endswith("\n\n")
```

再次运行测试——它们失败了！您能找出示例 7-1 中的错误吗？

空摘要会产生两行空白行：一行用于分隔标题和摘要，另一行用于打印空摘要。您只删除了第二行。示例 7-3 也删除了第一行。感谢，Coverage.py！

##### 示例 7-3\. 在`show`中避免尾随空行

```py
def show(article, file):
    console = Console(file=file, width=72, highlight=False)
    console.print(article.title, style="bold")
    if article.summary:
        console.print(f"\n{article.summary}")
```

# 在多个环境中进行测试

你经常需要支持各种 Python 版本。Python 每年发布新版本，而长期支持（LTS）发行版可以回溯到 Python 历史的十年前。终止生命周期的 Python 版本可能会有出人意料的余生—​发行商可能在核心 Python 团队结束支持后数年提供安全补丁。

让我们更新`random-wikipedia-article`以支持 Python 3.7，该版本在 2023 年 6 月已经终止生命周期。我假设你的项目需要 Python 3.10，并对所有依赖项设置了较低限制。首先，在*pyproject.toml*中放宽 Python 要求：

```py
[project]
requires-python = ">=3.7"
```

接下来，检查你的依赖项是否与 Python 版本兼容。使用`uv`为 Python 3.7 环境编译一个单独的要求文件：

```py
$ uv venv -p 3.7
$ uv pip compile --extra=tests pyproject.toml -o py37-dev-requirements.txt
  × No solution found when resolving dependencies: ...

```

错误表明你首选的 HTTPX 版本已经放弃了 Python 3.7。移除你的较低版本限制并重试。经过几个类似的错误和移除其他包的较低限制后，依赖解析最终成功。使用这些包的旧版本恢复较低限制。

你还需要后移植的`importlib-metadata`（参见“环境标记”）。将以下条目添加到`project.dependencies`字段：

```py
importlib-metadata>=6.7.0; python_version < '3.8'
```

更新*__init__.py*模块以回退到后移植：

```py
if sys.version_info >= (3, 8):
    from importlib.metadata import metadata
else:
    from importlib_metadata import metadata
```

再次编译要求。最后，更新你的项目环境：

```py
$ uv pip sync py37-dev-requirements.txt
$ uv pip install -e . --no-deps
```

# 并行覆盖率

如果现在在 Python 3.7 下重新运行 Coverage.py，它会报告`if`语句的第一个分支缺失。这是有道理的：你的代码执行`else`分支并导入后移植而不是标准库。

可能会诱人地排除此行不计入覆盖率测量—​但不要这样做。像后移植这样的第三方依赖项也可能破坏你的代码。相反，从两个环境收集覆盖率数据。

首先，使用原始要求将环境切换回 Python 3.12：

```py
$ uv venv -p 3.12
$ uv pip sync dev-requirements.txt
$ uv pip install -e . --no-deps

```

默认情况下，`coverage run`会覆盖任何现有的覆盖率数据—​但你可以告诉它追加数据。使用`--append`选项重新运行 Coverage.py，并确认你有完整的测试覆盖率：

```py
$ py -m coverage run --append -m pytest
$ py -m coverage report

```

有一个覆盖率数据的单个文件，很容易意外擦除数据。如果忘记传递`--append`选项，你将不得不重新运行测试。你可以配置 Coverage.py 默认追加，但这也容易出错：如果忘记定期运行`coverage erase`，你的报告中将出现陈旧的数据。

有一种更好的方法可以跨多个环境收集覆盖率。Coverage.py 允许你在每次运行时在单独的文件中记录覆盖率数据。使用`parallel`设置启用此行为：³

```py
[tool.coverage.run]
parallel = true
```

覆盖率报告始终基于单个数据文件，即使在并行模式下也是如此。你可以使用命令`coverage combine`合并数据文件。这将之前的两步过程变成三步过程：`coverage run` — `coverage combine` — `coverage report`。

让我们把所有这些都整合在一起。对于每个 Python 版本，设置环境并运行测试，如下所示，例如 Python 3.7：

```py
$ uv venv -p 3.7
$ uv pip sync py37-dev-requirements.txt
$ uv pip install -e . --no-deps
$ py -m coverage run -m pytest

```

此时，项目中将会有多个 *.coverage.* 文件。使用命令 `coverage combine` 将它们聚合成一个 *.coverage* 文件：

```py
$ py -m run coverage combine
Combined data file .coverage.somehost.26719.001909
Combined data file .coverage.somehost.26766.146311

```

最后，使用 `coverage report` 生成覆盖率报告：

```py
$ py -m coverage report
```

这样收集覆盖率听起来*极其*乏味吗？在 第八章 中，你将学习如何在多个 Python 环境中自动化测试。你将使用一个简单的三个字母的命令来运行整个过程。

# 在子进程中测量

在 “Using Coverage.py” 的结尾，你必须为 `main` 函数和 `__main__` 模块禁用覆盖率。但端到端测试肯定会执行此代码。让我们移除 `# pragma` 注释和 `omit` 设置，搞清楚这个问题。

想想 Coverage.py 如何注册一个跟踪函数来记录执行的行。也许你已经猜到这里正在发生什么：端到端测试在单独的进程中运行你的程序。Coverage.py 从未在该进程的解释器上注册其跟踪函数。没有这些执行的行被记录在任何地方。

Coverage.py 提供了一个公共 API 来启用当前进程的跟踪：`coverage.process_startup` 函数。你可以在应用程序启动时调用此函数。但肯定有更好的方法 —— 你不应该修改代码来支持代码覆盖率。

结果表明，你并不需要这样做。你可以在环境中放置一个 *.pth* 文件，在解释器启动时调用该函数。这利用了一个鲜为人知的 Python 特性（见 “Site Packages”）：解释器会执行 *.pth* 文件中以 `import` 语句开头的行。

将一个 *_coverage.pth* 文件安装到你的环境的 *site-packages* 目录中，内容如下：

```py
import coverage; coverage.process_startup()
```

在 Linux 和 macOS 下，可以在 *lib/python3.x* 目录下找到 *site-packages* 目录，在 Windows 下可以在 *Lib* 目录下找到。

另外，你需要设置环境变量 `COVERAGE_PROCESS_START`。在 Linux 和 macOS 上，使用以下语法：

```py
$ export COVERAGE_PROCESS_START=pyproject.toml

```

在 Windows 上，改用以下语法：

```py
> $env:COVERAGE_PROCESS_START = 'pyproject.toml'

```

重新运行测试套件，合并数据文件，并显示覆盖率报告。由于在子进程中测量覆盖率，程序应该再次实现全面覆盖。

###### 注意

测量子进程中的覆盖率仅在并行模式下有效。没有并行模式，主进程会覆盖子进程的覆盖数据，因为两者使用同一个数据文件。

# 目标覆盖率是多少

任何低于 100%的覆盖率意味着你的测试无法检测到代码库某些部分的 bug。如果你正在开发一个新项目，没有其他有意义的覆盖率目标。

这并不意味着您应该测试每一行代码。 考虑用于调试罕见情况的日志语句。 从测试中执行该语句可能会很困难。 同时，它可能是低风险，微不足道的代码。 编写该测试不会显着增加代码的信心。 使用*pragma*注释将该行从覆盖范围中排除：

```py
if rare_condition:
    print("got rare condition")  # pragma: no cover
```

不要因为测试麻烦而将代码排除在覆盖范围之外。 当您开始使用新库或与新系统进行接口时，通常需要一些时间来弄清楚如何测试您的代码。 但是，通常这些测试最终会检测到在生产中可能未被注意到并引起问题的错误。

传统项目通常由具有最小测试覆盖率的大型代码库组成。 作为一般规则，在这种项目中，覆盖范围应*单调增加*—没有个别更改应导致覆盖率下降。

您经常会在这里陷入困境：要进行测试，您需要重构代码，但是没有测试的重构太有风险。 找到最小的安全重构来增加可测试性。 通常，这包括破坏被测试代码的依赖关系。⁴

例如，您可能正在测试一个大型函数，该函数除了其他功能之外还连接到生产数据库。 添加一个可选参数，让您可以从外部传递连接。 然后，测试可以将连接传递给内存数据库。

示例 7-4 概述了本章中使用的 Coverage.py 设置。

##### 示例 7-4\. 在*pyproject.toml*中配置 Coverage.py

```py
[tool.coverage.run]
source = ["random_wikipedia_article", "tests"]
branch = true
parallel = true
omit = ["*/__main__.py"]  # avoid this if you can

[tool.coverage.report]
show_missing = true
fail_under = 100
```

# 总结

您可以使用 Coverage.py 来测量测试套件对项目的影响程度。 覆盖报告对发现未测试的行很有用。 分支覆盖率捕获程序的控制流，而不是源代码的孤立行。 并行覆盖允许您在多个环境中测量覆盖范围。 您需要在报告之前合并数据文件。 在子进程中测量覆盖率需要设置*.pth*文件和环境变量。

有效地为项目测量测试覆盖范围需要一定量的配置（），以及正确的工具咒语。 在下一章中，您将看到如何使用 Nox 自动化这些步骤。 您将设置检查，以便在更改时给您信心，同时不妨碍您的工作。

¹ Ned Batchelder：[“您应该将测试包含在覆盖范围内，”](https://nedbatchelder.com/blog/202008/you_should_include_your_tests_in_coverage.html) 2020 年 8 月 11 日。

² 在幕后，*.coverage*文件只是一个 SQLite 数据库。 如果您的系统已准备好`sqlite3`命令行实用程序，则可以随意查看。

³ 名称`parallel`有点误导性； 这个设置与并行执行无关。

⁴ 马丁·福勒：[“遗留系统的缝隙,”](https://www.martinfowler.com/bliki/LegacySeam.html) 2024 年 1 月 4 日。
