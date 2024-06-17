# 第八章：自动化与 Nox

当你维护一个 Python 项目时，你面临许多任务。对你的代码进行检查是其中重要的一部分：

+   测试帮助你降低代码的缺陷率（第六章）。

+   覆盖率报告可以找出你代码中未经测试的部分（第七章）。

+   代码检查器分析你的源代码，找到改进的方法（第九章）。

+   代码格式化器以可读的方式排列源代码（第九章）。

+   类型检查器验证你的代码的类型正确性（第十章）。

其他任务包括：

+   你需要为分发构建和发布包（第三章）。

+   你需要更新你的项目的依赖关系（第四章）。

+   你需要部署你的服务（见 示例 5-7 在 第五章）。

+   你需要为你的项目构建文档。

自动化这些任务有许多好处。你专注于编码，而检查套件则保护你的背后。你对将代码从开发到生产的步骤充满信心。你消除了人为错误，并对每个流程进行编码，以便他人可以审查和改进它。

自动化使你能够将每个步骤都尽可能地可重复，每个结果都尽可能地可再现。检查和任务在开发者机器上和持续集成（CI）中以相同的方式运行。它们跨不同的 Python 版本、操作系统和平台运行。

在本章中，你将学习 Nox，一个 Python 自动化框架。Nox 作为你的检查和任务的单一入口点—​为你的团队、外部贡献者和自动化系统如 CI 运行器。

你在纯 Python 中编写 Nox 会话：每个 Nox 会话都是一个执行命令的 Python 函数，它在专用的、隔离的环境中执行命令。使用 Python 作为自动化语言给予了 Nox 很好的简单性、可移植性和表现力。

# Nox 初步

使用 pipx 全局安装 Nox：

```py
$ pipx install --python=3.12 nox

```

指定最新稳定版本的 Python—​Nox 在创建环境时默认使用该版本。Pipx 使命令 `nox` 在全局范围内可用，同时将其依赖项与全局 Python 安装隔离开来（见 “使用 Pipx 安装应用程序”）。

在你的项目中，通过在 *pyproject.toml* 旁边创建一个名为 *noxfile.py* 的 Python 文件来配置 Nox。示例 8-1 展示了一个用于运行测试套件的 *noxfile.py*。它适用于一个没有测试依赖项的简单 Python 项目，除了 pytest。

##### 示例 8-1\. 用于运行测试套件的会话

```py
import nox

@nox.session
def tests(session):
    session.install(".", "pytest")
    session.run("pytest")
```

*会话* 是 Nox 中的核心概念：每个会话包括一个环境和一些在其中运行的命令。您通过编写一个使用 `@nox.session` 装饰的 Python 函数来定义会话。该函数接收一个会话对象作为参数，您可以使用它来在会话环境中安装包 (`session.install`) 和运行命令 (`session.run`)。

您可以尝试使用前几章的示例项目来运行会话。现在，将您的测试依赖项添加到 `session.install` 参数中：

```py
session.install(".", "pytest", "pytest-httpserver", "factory-boy")
```

无需参数即可调用 `nox` 来运行 *noxfile.py* 中的所有会话：

```py
$ nox
nox > Running session tests
nox > Creating virtual environment (virtualenv) using python in .nox/tests
nox > python -m pip install . pytest
nox > pytest
========================= tests session starts =========================
...
========================== 21 passed in 0.94s ==========================
nox > Session tests was successful.

```

正如您从输出中看到的那样，Nox 首先通过使用 `virtualenv` 为 `tests` 会话创建虚拟环境。如果您好奇，您可以在项目的 *.nox* 目录下找到此环境。

###### 注意

默认情况下，环境使用与 Nox 本身相同的解释器。在 “使用多个 Python 解释器” 中，您将学习如何在另一个解释器上运行会话，甚至跨多个解释器运行。

首先，会话将项目和 pytest 安装到其环境中。函数 `session.install` 实际上是 `pip install` 的简单封装。您可以向 pip 传递任何适当的选项和参数。例如，您可以从要求文件中安装依赖项：

```py
session.install("-r", "dev-requirements.txt")
session.install(".", "--no-deps")
```

如果将开发依赖项保存在额外位置，请使用以下方法：

```py
session.install(".[tests]")
```

在上述示例中，您使用了 `session.install(".")` 来安装您的项目。在 *pyproject.toml* 中指定的构建后端下，pip 在包含 *noxfile.py* 的目录中运行 Nox，因此该命令假定这两个文件位于同一个目录中。

Nox 允许您使用 `uv` 替代 virtualenv 和 pip 来创建环境并安装包。您可以通过设置环境变量来将后端切换到 `uv`：

```py
$ export NOX_DEFAULT_VENV_BACKEND=uv
```

其次，会话运行刚刚安装的 `pytest` 命令。如果命令失败，则会将会话标记为失败。默认情况下，Nox 继续执行下一个会话，但如果任何会话失败，则最终将以非零状态退出。在上述运行中，测试套件通过，Nox 报告成功。

示例 8-2 添加了一个用于为项目构建包的会话（参见 第三章）。该会话还使用 Twine 的 `check` 命令验证包。

##### 示例 8-2\. 用于构建包的一个会话

```py
import shutil
from pathlib import Path

@nox.session
def build(session):
    session.install("build", "twine")

    distdir = Path("dist")
    if distdir.exists():
        shutil.rmtree(distdir)

    session.run("python", "-m", "build")
    session.run("twine", "check", *distdir.glob("*"))
```

示例 8-2 依赖于标准库来清除陈旧的包并定位新构建的包：`Path.glob` 使用通配符匹配文件，而 `shutil.rmtree` 删除目录及其内容。

###### 提示

Nox 不像 `make` 等工具那样隐式在 shell 中运行命令。由于平台之间的 shell 差异很大，它们会使 Nox 会话的可移植性降低。出于同样的原因，在会话中避免使用类 Unix 工具如 `rm` 或 `find` ——应使用 Python 的标准库代替！

使用 `session.run` 调用的程序应该在环境内可用。如果不可用，Nox 会打印友好的警告并回退到系统范围的环境。在 Python 世界中，以错误的环境运行程序是一个容易犯而难以诊断的错误。将警告转为错误！示例 8-3 展示了如何做到这一点。

##### 示例 8-3\. 在 Nox 会话中阻止外部命令

```py
nox.options.error_on_external_run = True ![1](img/1.png)
```

![1](img/#co_automation_with_nox_CO1-1)

在 *noxfile.py* 的顶部（会话之外）修改 `nox.options`。

有时，您确实需要运行非 Python 构建工具等外部命令。您可以通过向 `session.run` 传递 `external` 标志来允许外部命令。示例 8-4 展示了如何使用系统中现有的 Poetry 安装构建软件包。

##### 示例 8-4\. 使用外部命令构建软件包

```py
@nox.session
def build(session):
    session.install("twine")
    session.run("poetry", "build", external=True)
    session.run("twine", "check", *Path().glob("dist/*"))
```

在这里，您正在权衡可靠性与速度。示例 8-2 可与 *pyproject.toml* 中声明的任何构建后端一起工作，并在每次运行时将其安装在隔离环境中。示例 8-4 假设贡献者系统中有最新版本的 Poetry，并且如果没有，会出问题。除非每个开发环境都有一个已知的 Poetry 版本，否则请优先考虑第一种方法。

# 会话操作

随着时间推移，*noxfile.py* 可能会积累多个会话。 `--list` 选项可以快速概览它们。如果您为模块和函数添加了有用的描述性文档字符串，Nox 也会将它们包含在列表中。

```py
$ nox --list
Run the checks and tasks for this project.

Sessions defined in /path/to/noxfile.py:

* tests -> Run the test suite.
* build -> Build the package.

sessions marked with * are selected, sessions marked with - are skipped.

```

使用 `--session` 选项运行 Nox，可以按名称选择单个会话：

```py
$ nox --session tests

```

在开发过程中，反复运行 `nox` 可以让您及早捕捉错误。另一方面，您不需要每次验证包。幸运的是，您可以通过设置 `nox.options.sessions` 来更改默认运行的会话：

```py
nox.options.sessions = ["tests"]
```

现在，当您无参数运行 `nox` 时，只会运行 `tests` 会话。您仍然可以使用 `--session` 选项选择 `build` 会话。命令行选项会覆盖 *noxfile.py* 中 `nox.options` 中指定的值。¹

###### 小贴士

保持默认会话与项目的强制检查一致。贡献者应该能够无需参数运行 `nox` 来检查他们的代码更改是否可接受。

每次会话运行时，Nox 都会创建一个新的虚拟环境并安装依赖项。这是一个很好的默认设置，因为它使检查严格、可预测且可重复。您不会因为会话环境中的过期软件包而错过代码问题。

然而，Nox 给了您选择的余地。如果您在编码时快速连续重新运行测试，每次设置环境可能会有点慢。您可以使用 `-r` 或 `--reuse-existing-virtualenvs` 选项重用环境。此外，您可以通过指定 `--no-install` 跳过安装命令，或者使用 `-R` 简写组合这些选项。

```py
$ nox -R
nox > Running session tests
nox > Re-using existing virtual environment at .nox/tests.
nox > pytest
...
nox > Session tests was successful.

```

# 使用多个 Python 解释器

如果你的项目支持多个版本的 Python，你应该在所有这些版本上运行测试。当涉及到在多个解释器上运行会话时，Nox 真的非常出色。当你使用 `@nox.session` 定义会话时，可以使用 `python` 关键字请求一个或多个特定的 Python 版本，如 示例 8-5 所示。

##### 示例 8-5\. 在多个 Python 版本上运行测试

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install(".[tests]")
    session.run("pytest")
```

Nox 为每个版本创建一个环境，并依次在这些环境中运行命令：

```py
$ nox
nox > Running session tests-3.12
nox > Creating virtual environment (virtualenv) using python3.12 in .nox/tests-3-12
nox > python -m pip install . pytest
nox > pytest
...
nox > Session tests-3.12 was successful.
nox > Running session tests-3.11
...
nox > Running session tests-3.10
...
nox > Ran multiple sessions:
nox > * tests-3.12: success
nox > * tests-3.11: success
nox > * tests-3.10: success

```

###### 小贴士

刚才在运行 Nox 时是否收到了来自 pip 的错误？不要为每个 Python 版本使用相同的编译后的依赖文件。你需要为每个环境单独锁定依赖项（参见 “会话依赖项”）。

你可以使用 `--python` 选项按 Python 版本缩小会话范围：

```py
$ nox --python 3.12
nox > Running session tests-3.12
...

```

在开发过程中，`--python` 选项非常方便，因为它只让你在最新版本上运行测试，从而节省时间。

Nox 通过在 `PATH` 中搜索 `python3.12`、`python3.11` 等命令来发现解释器。你还可以指定类似 `"pypy3.10"` 的字符串来请求 PyPy 解释器—任何可以在 `PATH` 中解析的命令都可以工作。在 Windows 上，Nox 还查询 Python 启动器以查找可用的解释器。

假设你已经安装了 Python 的预发行版本，并想在其上测试你的项目。`--python` 选项将要求会话列出预发行版。相反，你可以指定 `--force-python`：它将覆盖解释器以进行单次运行。例如，以下调用在 Python 3.13 上运行 `tests` 会话：

```py
$ nox --session tests --force-python 3.13

```

# 会话参数

到目前为止，`tests` 会话以无参数运行 pytest：

```py
session.run("pytest")
```

你*可以*传递额外的选项—比如 `--verbose`，它会在输出中单独列出每个单独的测试：

```py
session.run("pytest", "--verbose")
```

但并非每次都需要相同的选项来运行 pytest。例如，`--pdb` 选项会在测试失败时启动 Python 调试器。在调查神秘错误时，调试提示符可能是救命稻草。但在 CI 环境中却比毫无用处更糟糕：它会永远挂起，因为没有人来输入命令。同样，当你在开发一个功能时，`-k` 选项允许你运行具有特定关键字名称的测试，但你也不希望在 *noxfile.py* 中硬编码它。

幸运的是，Nox 允许你为会话传递额外的命令行参数。会话可以将这些参数转发给一个命令或者用于自己的目的。会话参数可以在会话中作为 `session.posargs` 使用。示例 8-6 展示了如何将它们转发给像 pytest 这样的命令。

##### 示例 8-6\. 将会话参数转发给 pytest

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install(".[tests]")
    session.run("pytest", *session.posargs)
```

你必须使用 `--` 分隔符将会话参数与 Nox 自己的命令行参数分开：

```py
$ nox --session tests -- --verbose

```

# 自动化覆盖率

覆盖工具可以帮助您了解您的测试覆盖了多少代码库（参见第 7 章）。简而言之，您需要安装`coverage`包，并通过`coverage run`调用 pytest。示例 8-7 展示了如何使用 Nox 自动化此过程：

##### 示例 8-7\. 带代码覆盖运行测试

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install(".[tests]")
    session.run("coverage", "run", "-m", "pytest", *session.posargs)
```

当您在多个环境中进行测试时，您需要将每个环境的覆盖数据存储在单独的文件中（参见“并行覆盖”）。要启用此模式，请向*pyproject.toml*添加以下行：

```py
[tool.coverage.run]
parallel = true
```

在第 7 章中，您以可编辑模式安装了您的项目。Nox 会话将构建并安装您项目的轮子。这确保您正在测试最终分发给用户的工件。但这也意味着 Coverage.py 需要将安装的文件映射回您的源树。请在*pyproject.toml*中配置映射：

```py
[tool.coverage.paths]
source = ["src", "*/site-packages"] ![1](img/1.png)
```

![1](img/#co_automation_with_nox_CO2-1)

这将环境中安装的文件映射到您*src*目录中的文件。键`source`是任意标识符；这是必需的，因为此部分可以有多个映射。

示例 8-8 聚合覆盖文件并显示覆盖报告：

##### 示例 8-8\. 报告代码覆盖率

```py
@nox.session
def coverage(session):
    session.install("coverage[toml]")
    if any(Path().glob(".coverage.*")):
        session.run("coverage", "combine")
    session.run("coverage", "report")
```

仅当存在覆盖数据文件时，会话才会调用`coverage combine`—否则该命令会失败。因此，您可以安全地使用`nox -s coverage`检查测试覆盖率，而无需首先重新运行测试。

与示例 8-7 不同，此会话在默认 Python 版本上运行，并且仅安装 Coverage.py。您无需安装项目即可生成覆盖报告。

如果您在示例项目上运行这些会话，请确保按照第 7 章中所示配置 Coverage.py。如果您的项目使用条件导入`importlib-metadata`，请在`tests`会话中包含 Python 3.7。

```py
$ nox --session coverage
nox > Running session coverage
nox > Creating virtual environment (uv) using python in .nox/coverage
nox > uv pip install 'coverage[toml]'
nox > coverage combine
nox > coverage report
Name                  Stmts   Miss Branch BrPart  Cover   Missing
-----------------------------------------------------------------
src/.../__init__.py      29      2      8      0    95%   42-43
src/.../__main__.py       2      2      0      0     0%   1-3
tests/__init__.py         0      0      0      0   100%
tests/test_main.py       36      0      6      0   100%
-----------------------------------------------------------------
TOTAL                    67      4     14      0    95%
Coverage failure: total of 95 is less than fail-under=100
nox > Command coverage report failed with exit code 2
nox > Session coverage failed.

```

###### 注意

`coverage`会话仍然报告了`main`函数和`__main__`模块的缺失覆盖。您将在“子进程中自动化覆盖率”中处理这些问题。

# 会话通知

就目前而言，这个*noxfile.py*存在一个微妙的问题。在运行`coverage`会话之前，您的项目将充斥着等待处理的数据文件。如果您最近没有运行`tests`会话，则这些文件中的数据可能已过时—因此您的覆盖报告将不会反映代码库的最新状态。

示例 8-9 在测试套件之后自动触发`coverage`会话运行。Nox 通过`session.notify`方法支持此功能。如果通知的会话尚未选定，则它会在其他会话完成后运行。

##### 示例 8-9\. 从测试触发覆盖率报告

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install(".[tests]")
    try:
        session.run("coverage", "run", "-m", "pytest", *session.posargs)
    finally:
        session.notify("coverage")
```

`try...finally`块确保即使测试失败，您也能获得覆盖报告。当您从一个失败的测试开始开发时，这非常有帮助：您希望确保测试覆盖您正在编写的代码以使其通过。

# 在子进程中自动化覆盖率

Alan Kay，面向对象编程和图形用户界面设计的先驱，曾经说过：“简单的事情应该是简单的；复杂的事情应该是可能的。”² 许多 Nox 会话只有两行：一行用于安装依赖项，一行用于运行命令。然而，一些自动化任务需要更复杂的逻辑，而 Nox 也擅长处理这些，主要是通过不阻碍您并推迟到 Python 作为一种通用编程语言。

让我们在`tests`会话中进行迭代，并在子进程中测量覆盖率。正如您在第七章中看到的那样，设置这个需要一些技巧。首先，您需要将*.pth*文件安装到环境中；这使得在子进程启动时`coverage`有机会初始化。其次，您需要设置一个环境变量来指向`coverage`的配置文件。这些操作都有点繁琐且需要一定技巧才能做好。让我们“自动化”它吧！

首先，您需要确定*.pth*文件的位置。目录名为*site-packages*，但确切的路径取决于您的平台和 Python 版本。您可以通过查询`sysconfig`模块来获取它，而不是猜测：

```py
sysconfig.get_path("purelib")
```

如果您直接在会话中调用函数，它将返回您已经安装 Nox 的环境中的位置。而实际上，您需要查询*会话环境*中的解释器。您可以通过使用`session.run`来运行`python`来实现这一点：

```py
output = session.run(
    "python",
    "-c",
    "import sysconfig; print(sysconfig.get_path('purelib'))",
    silent=True,
)
```

`silent`关键字允许您捕获输出而不是将其回显到终端。由于标准库中的`pathlib`，现在编写*.pth*文件只需要几个语句：

```py
purelib = Path(output.strip())
(purelib / "_coverage.pth").write_text(
    "import coverage; coverage.process_startup()"
)
```

示例 8-10 将这些语句提取到一个助手函数中。该函数接受一个`session`参数，但它不是一个 Nox 会话——它缺少`@nox.session`装饰器。换句话说，如果不从会话中调用它，该函数将不会运行。

##### 示例 8-10\. 将*coverage.pth*安装到环境中

```py
def install_coverage_pth(session):
    output = session.run(...)  # see above
    purelib = Path(output.strip())
    (purelib / "_coverage.pth").write_text(...)  # see above
```

您已经快完成了。剩下的是从`tests`会话中调用助手函数并将环境变量传递给`coverage`。示例 8-11 展示了最终的会话。

##### 示例 8-11\. 启用子进程覆盖率的`tests`会话

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install(".[tests]") ![1](img/1.png)
    install_coverage_pth(session)

    try:
        args = ["coverage", "run", "-m", "pytest", *session.posargs]
        session.run(*args, env={"COVERAGE_PROCESS_START": "pyproject.toml"})
    finally:
        session.notify("coverage")
```

![1](img/#co_automation_with_nox_CO3-1)

在*.pth*文件之前安装依赖项。顺序很重要，因为*.pth*文件会导入`coverage`包。

启用子进程覆盖率后，端到端测试将为`main`函数和`__main__`模块生成缺失的覆盖数据。调用`nox`并观看它运行您的测试并生成覆盖报告。以下是报告的示例内容：

```py
$ nox --session coverage
nox > coverage report
Name                  Stmts   Miss Branch BrPart  Cover   Missing
-----------------------------------------------------------------
src/.../__init__.py      29      0      8      0   100%
src/.../__main__.py       2      0      0      0   100%
tests/__init__.py         0      0      0      0   100%
tests/test_main.py       36      0      6      0   100%
-----------------------------------------------------------------
TOTAL                    67      0     14      0   100%
nox > Session coverage was successful.

```

# 参数化会话

“对我有效”这个短语描述了一个常见的情况：用户报告您代码的问题，但您无法在自己的环境中重现该错误。真实世界的运行时环境在许多方面都是不同的。跨 Python 版本进行测试涵盖了一个重要变量。另一个常见的令人惊讶的原因是您的项目直接或间接使用的包——即其依赖树。

Nox 为测试项目与不同版本依赖提供了强大的技术。*参数化*允许您向会话函数添加参数并为其提供预定义的值；Nox 会使用每个值运行会话。

您在名为`@nox.parametrize`的装饰器中声明参数及其值。³示例 8-12 演示了此功能及其如何允许您针对 Django Web 框架的不同版本进行测试。

##### 示例 8-12\. 使用多个 Django 版本测试项目

```py
@nox.session
@nox.parametrize("django", ["5.*", "4.*", "3.*"])
def tests(session, django):
    session.install(".", "pytest-django", f"django=={django}")
    session.run("pytest")
```

参数化会话类似于 pytest 中的参数化测试（参见第六章），这是 Nox 借用的概念。您可以堆叠`@nox.parametrize`装饰器，以针对所有参数组合运行会话：

```py
@nox.session
@nox.parametrize("a", ["1.0", "0.9"])
@nox.parametrize("b", ["2.2", "2.1"])
def tests(session, a, b):
    print(a, b)  # all combinations of a and b
```

如果您只想检查某些组合，可以将参数组合在单个`@nox.parametrize`装饰器中：

```py
@nox.session
@nox.parametrize(["a", "b"], [("1.0", "2.2"), ("0.9", "2.1")])
def tests(session, a, b):
    print(a, b)  # only the combinations listed above
```

当跨 Python 版本运行会话时，您实际上是通过解释器对会话进行参数化。事实上，Nox 让您可以这样写，而不是将版本传递给`@nox.session`：⁴

```py
@nox.session
@nox.parametrize("python", ["3.12", "3.11", "3.10"])
def tests(session):
    ...
```

在您想要特定 Python 和依赖组合时，此语法非常有用。以下是一个示例：截至本文撰写时，Django 3.2（LTE）并不正式支持比 3.10 更新的 Python 版本。因此，您需要从测试矩阵中排除这些组合。示例 8-13 展示了如何实现这一点。

##### 示例 8-13\. 使用有效的 Python 和 Django 组合进行参数化

```py
@nox.session
@nox.parametrize(
    ["python", "django"],
    [
        (python, django)
        for python in ["3.12", "3.11", "3.10"]
        for django in ["3.2.*", "4.2.*"]
        if (python, django) not in [("3.12", "3.2"), ("3.11", "3.2")]
    ]
)
def tests(session, django):
    ...
```

# 会话依赖

如果您仔细阅读了第四章，您可能会注意到示例 8-8 和示例 8-11 安装包的方式存在一些问题。这里再次列出相关部分：

```py
@nox.session
def tests(session):
    session.install(".[tests]")
    ...

@nox.session
def coverage(session):
    session.install("coverage[toml]")
    ...
```

首先，`coverage`会话没有指定项目需要的`coverage`版本。`tests`会话做得对：它引用了*pyproject.toml*中的`tests`额外部分，其中包含适当的版本说明符（参见“管理测试依赖”）。

`coverage`会话不需要项目，所以额外的似乎不太合适。但在我继续之前，让我指出上述会话的另一个问题：它们没有锁定它们的依赖关系。

在不锁定依赖项的情况下运行检查有两个缺点。首先，检查不是确定性的：同一会话的后续运行可能会安装不同的包。其次，如果一个依赖关系破坏了你的项目，检查将失败，直到你排除该发布或另一个发布修复了问题。⁵ 换句话说，你依赖的任何项目，甚至间接依赖的项目，都有可能阻塞你整个持续集成流水线。

另一方面，锁定文件的更新是一个不断变化的过程，并且会混淆你的 Git 历史记录。减少它们的频率是以使用过时依赖项来运行检查为代价的。如果你没有其他原因要求锁定，比如安全部署，且愿意在不兼容的发布混乱你的持续集成时快速修复构建，你可能更愿意保持你的依赖项未锁定。没有免费的午餐。

在“开发依赖”中，你将额外的依赖项分组并从每个编译需求文件。在本节中，我将向你展示一种更轻量级的锁定方法：*约束文件*。你只需要一个额外的依赖项。它也不需要像通常那样安装项目本身，这有助于`coverage`会话。

约束文件看起来类似于需求文件：每行列出一个带有版本说明符的包。然而，与需求文件不同，约束文件不会导致 pip 安装一个包——它们只控制 pip 在需要安装包时选择的版本。

约束文件非常适合用于锁定会话依赖项。你可以在会话之间共享它，同时仅安装每个会话所需的包。与使用一组需求文件相比，它唯一的缺点是需要一起解决所有依赖关系，因此存在更高的依赖冲突的可能性。

你可以使用 pip-tools 或 uv 生成约束文件（请参阅“使用 pip-tools 和 uv 编译需求”）。Nox 也可以自动化这一部分，如示例 8-14 所示。

##### 示例 8-14\. 使用 uv 锁定依赖项

```py
@nox.session(venv_backend="uv") ![1](img/1.png)
def lock(session):
    session.run(
        "uv",
        "pip",
        "compile",
        "pyproject.toml",
        "--upgrade",
        "--quiet",
        "--all-extras",
        "--output-file=constraints.txt",
    )
```

![1](img/#co_automation_with_nox_CO4-1)

明确要求将 uv 作为环境后端。（你也可以在会话中安装 uv，方法是`session.install("uv")`，但 uv 支持比你可以用来安装其 PyPI 包的 Python 版本范围更广。）

`--output-file`选项指定约束文件的传统名称，*constraints.txt*。`--upgrade`选项确保您在每次运行会话时获取最新的依赖关系。`--all-extras`选项包含项目的所有可选依赖项。

###### 提示

不要忘记将约束文件提交到源代码控制。你需要与每个贡献者分享这个文件，并且它需要在持续集成中可用。

使用 `--constraint` 或 `-c` 选项将约束文件传递给 `session.install`。示例 8-15 显示了带有锁定依赖项的 `tests` 和 `coverage` 会话。

##### 示例 8-15\. 使用约束文件锁定会话依赖项

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install("-c", "constraints.txt", ".[tests]")
    ...

@nox.session
def coverage(session):
    session.install("-c", "constraints.txt", "coverage[toml]")
    ...
```

使用单个约束文件需要定位一个众所周知的解释器和平台。你不能在不同的环境中使用相同的约束文件，因为每个环境可能需要不同的包。

如果你支持多个 Python 版本、操作系统或处理器架构，请为每个环境编译一个单独的约束文件。将约束文件保存在子目录中以避免混乱。示例 8-16 显示了一个辅助函数，用于构建类似 *constraints/python3.12-linux-arm64.txt* 的文件名。

##### 示例 8-16\. 构建约束文件的路径

```py
import platform, sys
from pathlib import Path

def constraints(session):
    filename = f"python{session.python}-{sys.platform}-{platform.machine()}.txt"
    return Path("constraints") / filename
```

示例 8-17 更新了 `lock` 会话以生成约束文件。该会话现在在每个 Python 版本上运行。它使用辅助函数构建约束文件的路径，确保目标目录存在，并将文件名传递给 uv。

##### 示例 8-17\. 在多个 Python 版本上锁定依赖项

```py
@nox.session(python=["3.12", "3.11", "3.10"], venv_backend="uv")
def lock(session):
    filename = constraints(session)
    filename.parent.mkdir(exist_ok=True)
    session.run("uv", "pip", "compile", ..., f"--output-file={filename}")
```

`tests` 和 `coverage` 会话现在可以引用每个 Python 版本的适当约束文件。为使其正常工作，你必须为 `coverage` 会话声明一个 Python 版本。

##### 示例 8-18\. `tests` 和 `coverage` 会话与多 Python 约束

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    session.install("-c", constraints(session), ".", "pytest", "coverage[toml]")
    ...

@nox.session(python="3.12")
def coverage(session):
    session.install("-c", constraints(session), "coverage[toml]")
    ...
```

# 使用 Nox 处理 Poetry 项目

如果你使用 Poetry 管理项目，则将依赖项组织在依赖组中（参见 “Dependency Groups”）。依赖组与 Nox 会话自然对齐：`tests` 会话的包进入 `tests` 组，`docs` 会话的包进入 `docs` 组，依此类推。使用 Poetry 作为安装程序意味着你免费获得了锁定的依赖项——所有安装都遵循锁定文件。

在我向你展示如何在 Nox 会话中使用 Poetry 之前，让我先指出 Poetry 环境和 Nox 环境之间的一些区别。

首先，Poetry 环境非常全面：默认情况下，它包括项目、其主要依赖项以及每个非可选的依赖组。Nox 环境仅安装任务自动化所需的包。

其次，Poetry 环境使用可编辑安装项目，因此你不需要在每次代码更改后重新安装。Nox 环境构建并安装轮子，因此自动化检查会看到与最终用户相同的项目。

这里没有对错之分。Poetry 环境非常适合在开发过程中与项目进行临时交互，每个工具都只需执行 `poetry run` 即可。另一方面，Nox 环境则针对可靠和可重复的检查进行了优化；它们旨在尽可能地隔离和确定性。

在 Nox 会话中使用 Poetry 时，需要注意这些差异。我建议在使用 Nox 调用`poetry install`时遵循以下准则：

+   使用选项`--no-root`避免对项目进行可编辑的安装。如果会话需要安装项目（并非每个会话都需要），请在命令后跟随`session.install(".")`来构建和安装一个 wheel。

+   使用选项`--only=*<group>*`为会话安装适当的依赖组。如果会话需要安装项目，请同时列出特殊组`main`—​这确保每个软件包都使用*poetry.lock*进行固定。

+   使用选项`--sync`从重用的会话环境中删除项目不再依赖的软件包。

示例 8-19 将此逻辑放入一个辅助函数中，您可以在会话函数之间共享它。

##### 示例 8-19\. 使用 Poetry 安装会话依赖项

```py
def install(session, groups, root=True):
    if root:
        groups = ["main", *groups]

    session.run_install(
        "poetry",
        "install",
        "--no-root",
        "--sync",
        f"--only={','.join(groups)}",
        external=True,
    )
    if root:
        session.install(".")
```

辅助函数使用`session.run_install`而不是`session.run`。这两个函数的工作方式完全相同，但`session.run_install`将命令标记为安装操作。这在您使用`--no-install`或`-R`重用环境时避免了软件包安装。

如果您已经在第六章和第七章中使用 Poetry 进行了跟随，那么您的*pyproject.toml*应该有一个名为`tests`的依赖组，其中包含 pytest 和 coverage。让我们将`coverage`依赖项拆分到单独的组中，因为您不需要`coverage`会话的测试依赖项:⁶

```py
[tool.poetry.group.coverage.dependencies]
coverage = {extras = ["toml"], version = ">=7.4.4"}

[tool.poetry.group.tests.dependencies]
pytest = ">=8.1.1"
```

下面是如何在`tests`会话中安装依赖项的示例：

```py
@nox.session(python=["3.12", "3.11", "3.10"])
def tests(session):
    install(session, groups=["coverage", "tests"])
    ...
```

下面是带有辅助函数的`coverage`会话的样子：

```py
@nox.session
def coverage(session):
    install(session, groups=["coverage"], root=False)
    ...
```

###### 小贴士

Poetry 如何知道要使用 Nox 环境而不是 Poetry 环境？Poetry 会将软件包安装到活动环境中（如果存在）。当 Nox 运行 Poetry 时，它通过导出`VIRTUAL_ENV`环境变量来激活会话环境（参见“虚拟环境”）。

# 使用`nox-poetry`锁定依赖关系

使用 Nox 与 Poetry 项目的另一种方法是选择。`nox-poetry`包是 Nox 的非官方插件，允许您使用简单的`session.install`编写会话，无需担心锁定。在幕后，`nox-poetry`通过`poetry export`导出一个 pip 的约束文件。

将`nox-poetry`安装到与 Nox 相同的环境中：

```py
$ pipx inject nox nox-poetry

```

使用`nox-poetry`包中的`@session`装饰您的会话，它是`@nox.session`的替代品：

```py
from nox_poetry import session

@session
def tests(session):
    session.install(".", "coverage[toml]", "pytest")
    ...
```

使用`session.install`安装软件包时，约束文件会将它们的版本与`poetry.lock`保持同步。您可以将依赖项管理在单独的依赖组中，也可以将它们放入单个`dev`组中。

尽管这种方法方便且简化了结果的 *noxfile.py*，但它并非免费的。将 Poetry 的依赖信息翻译为约束文件并非无损—例如，它不包括包哈希或私有包存储库的 URL。另一个缺点是贡献者需要担心另一个全局依赖。当我在 2020 年编写 `nox-poetry` 时，不存在依赖组。截至撰写本文时，我建议直接使用 Poetry，正如前一节所述。

# 摘要

Nox 让你能够自动化项目的检查和任务。它的 Python 配置文件 *noxfile.py* 将它们组织成一个或多个会话。会话是使用 `@nox.session` 装饰的函数。它们接收一个名为 `session` 的参数，提供会话 API（表 8-1）。每个会话在独立的虚拟环境中运行。如果你向 `@nox.session` 传递一个 Python 版本列表，Nox 将在所有这些版本上运行该会话。

表 8-1\. 会话对象

| 属性 | 描述 | 示例 |
| --- | --- | --- |
| `run()` | 运行命令 | `session.run("coverage", "report")` |
| `install()` | 使用 pip 安装包 | `session.install(".", "pytest")` |
| `run_install()` | 运行安装命令 | `session.run_install("poetry", "install")` |
| `notify()` | 将另一个会话加入队列 | `session.notify("coverage")` |
| `python` | 该会话的解释器 | `"3.12"` |
| `posargs` | 额外的命令行参数 | `nox -- --verbose --pdb` |

命令 `nox`（表 8-2）为你的一套检查提供了单一入口点。如果没有参数，它将运行 *noxfile.py* 中定义的每个会话（或者你在 `nox.options.sessions` 中列出的会话）。尽早发现代码问题，修复起来就越便宜—因此使用 Nox 在本地运行与持续集成（CI）中相同的检查。除了检查，你还可以自动化许多其他任务，如构建包或文档。

表 8-2\. `nox` 的命令行选项

| 选项 | 描述 | 示例 |
| --- | --- | --- |
| `--list` | 列出可用的会话 | `nox -l` |
| `--session` | 通过名称选择会话 | `nox -s tests` |
| `--python` | 选择解释器会话 | `nox -p 3.12` |
| `--force-python` | 为会话强制指定解释器 | `nox --force-python 3.13` |
| `--reuse-existing-virtualenvs` | 重用现有的虚拟环境 | `nox -rs tests` |
| `--no-install` | 跳过安装命令 | `nox -Rs tests` |

Nox 还有更多内容本章没有覆盖到。例如，你可以使用 Conda 或 Mamba 创建环境并安装包。你可以使用关键字和标签来组织会话，并使用 `nox.param` 分配友好的标识符。最后但同样重要的是，Nox 提供了一个 GitHub Action，方便在 CI 中运行 Nox 会话。请查阅[官方文档](https://nox.thea.codes)以了解更多。

¹ 如果你在想，始终在*noxfile.py*中使用复数形式`nox.options.sessions`。在命令行中，`--session`和`--sessions`都可用。这些选项可以指定任意数量的会话。

² 艾伦·凯，[“艾伦·凯的格言 *简单的事物应该简单，复杂的事物应该是可能的* 的背后有什么故事？”](https://qr.ae/pyjKrs)，*Quora 回答*，2020 年 6 月 19 日。

³ 与 pytest 类似，Nox 使用替代拼写“parametrize”来保护你的“E”键帽免受过度磨损。

⁴ 眼尖的读者可能会注意到，这里`python`不是一个函数参数。如果你确实需要在会话函数中使用它，请改用`session.python`。

⁵ 在这里，语义版本控制的约束比它的帮助更有害。所有版本都可能出现错误，而你的上游定义的破坏性更改可能比你想象的要狭窄。请参见 Hynek Schlawack 的文章：[“语义版本控制无法拯救你”，](https://hynek.me/articles/semver-will-not-save-you/) 2021 年 3 月 2 日。

⁶ 编辑*pyproject.toml*后运行`poetry lock --no-update`，以更新*poetry.lock*文件。
