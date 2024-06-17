# 第五章：管理使用 Poetry 的项目

前几章介绍了发布高质量 Python 包的基础模块。到目前为止，你为项目编写了一个*pyproject.toml*；创建了一个环境，并使用 uv、pip 或 pip-tools 安装了依赖；并使用`build`和 Twine 构建并发布了软件包。

通过标准化项目元数据和构建后端，*pyproject.toml*打破了 setuptools 的垄断（见“Python 项目管理器的演变”），并为打包生态系统带来了多样性。定义 Python 包也变得更简单：一个明确定义的文件配合优秀的工具支持，取代了*setup.py*及不计其数的配置文件的传统模板。

然而，仍然存在一些问题。

在你可以开始处理基于*pyproject.toml*的项目之前，你需要研究打包工作流程、配置文件以及相关工具。你必须从多个可用的构建后端中选择一个（见表 3-2）—而很多人不知道这些是什么，更不用说如何选择了。Python 包的重要方面仍然未指明—例如，项目源代码的布局及哪些文件应包含在打包工件中。

依赖和环境管理也可以更简单。你需要手工制作你的依赖规范，并使用 pip-tools 编译它们，使你的项目混乱不堪。在典型的开发者系统上跟踪多个 Python 环境可能很困难。

在*pyproject.toml*的标准规范制定之前，Python 项目管理器 Poetry 已经在解决这些问题。它友好的命令行界面让你可以执行大多数与打包、依赖和环境相关的任务。Poetry 带来了符合标准的构建后端`poetry.core`—但你可以毫不知情地继续使用。它还配备了严格的依赖解析器，并默认锁定所有依赖项，所有这些都在幕后进行。

如果 Poetry 在很大程度上将这些细节抽象化掉，为什么要学习打包标准和底层管道？因为虽然 Poetry 在新领域中大展拳脚，但它仍在打包标准定义的框架内运作。像依赖规范和虚拟环境这样的机制支持其核心功能。互操作性标准使 Poetry 能够与软件包存储库以及其他构建后端和包安装程序进行交互。

对这些底层机制的理解有助于你调试情况，例如 Poetry 方便的抽象出现问题时—例如，当配置错误或错误导致软件包进入错误的环境时。最后，过去几十年的经验告诉我们，工具来了又走，而标准和算法却是长存的。

# 安装 Poetry

使用 pipx 全局安装 Poetry，以使其依赖与系统其余部分隔离开来：

```py
$ pipx install poetry

```

单个 Poetry 安装可以与多个 Python 版本一起使用。但是，Poetry 默认使用其自己的解释器作为默认 Python 版本。因此，建议在最新稳定的 Python 发行版上安装 Poetry。安装新的 Python 功能版本时，请像这样重新安装 Poetry：

```py
$ pipx reinstall --python=3.12 poetry

```

如果 pipx 已经使用新的 Python 版本，请忽略 `--python` 选项（见“配置 Pipx”）。

当 Poetry 的预发行版可用时，您可以将其与稳定版本并行安装：

```py
$ pipx install poetry --suffix=@preview --pip-args=--pre

```

上述示例中，我使用了 `--suffix` 选项来重命名命令，因此您可以将其称为 `poetry@preview`，同时保持 `poetry` 为稳定版本。`--pip-args` 选项允许您将选项传递给 pip，例如用于包括预发行版的 `--pre`。

###### 注意

Poetry 还提供了一个[官方安装程序](https://python-poetry.org/docs/#installing-with-the-official-installer)，您可以使用 Python 下载并运行。虽然不像 pipx 那样灵活，但提供了一个现成的替代方案：

```py
$ curl -sSL https://install.python-poetry.org | python3 -
```

定期升级 Poetry 以获取改进和错误修复：

```py
$ pipx upgrade poetry

```

在终端上输入 `poetry` 来检查 Poetry 的安装情况。Poetry 会打印其版本和用法，包括所有可用子命令的有用列表。

```py
$ poetry

```

成功安装 Poetry 后，您可能希望为您的 shell 启用选项卡完成。使用命令 `poetry help completions` 获取特定于 shell 的说明。例如，以下命令在 Bash shell 中启用选项卡完成：

```py
$ poetry completions bash >> ~/.bash_completion
$ echo ". ~/.bash_completion" >> ~/.bashrc

```

重启您的 shell 以使更改生效。

# 创建项目

您可以使用命令 `poetry new` 创建一个新项目。作为示例，我将使用前几章中的 `random-wikipedia-article` 项目。在您想要保存新项目的父目录中运行以下命令：

```py
$ poetry new --src random-wikipedia-article

```

运行此命令后，您将看到 Poetry 创建了一个名为 *random-wikipedia-article* 的项目目录，其结构如下：

```py
random-wikipedia-article
├── README.md
├── pyproject.toml
├── src
│   └── random_wikipedia_article
│       └── __init__.py
└── tests
    └── __init__.py
```

`--src` 选项指示 Poetry 将导入包放置在名为 *src* 的子目录中，而不是直接放在项目目录中。

让我们来看看生成的 *pyproject.toml* 文件（示例 5-1）：

##### 示例 5-1\. Poetry 的 pyproject.toml 文件

```py
[tool.poetry]
name = "random-wikipedia-article"
version = "0.1.0"
description = ""
authors = ["Your Name <you@example.com>"]
readme = "README.md"
packages = [{include = "random_wikipedia_article", from = "src"}]

[tool.poetry.dependencies]
python = "³.12"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

Poetry 使用其构建后端 `poetry.core` 创建了一个标准的 `build-system` 表。这意味着任何人都可以使用 pip 或 uv 从源代码安装您的项目，无需设置或了解 Poetry 项目管理器。同样，您可以使用任何标准的构建前端（例如 `build`）构建包：

```py
$ pipx run build
* Creating venv isolated environment...
* Installing packages in isolated environment... (poetry-core)
* Getting build dependencies for sdist...
* Building sdist...
* Building wheel from sdist
* Creating venv isolated environment...
* Installing packages in isolated environment... (poetry-core)
* Getting build dependencies for wheel...
* Building wheel...
Successfully built random_wikipedia_article-0.1.0.tar.gz
 and random_wikipedia_article-0.1.0-py3-none-any.whl

```

## 项目元数据

你可能会惊讶地看到项目元数据出现在`tool.poetry`下，而不是熟悉的`project`表（参见“项目元数据”）。Poetry 项目计划在其下一个主要发布版中支持项目元数据标准。¹ 正如您在表格 5-1 中所见，大多数字段具有相同的名称、类似的语法和含义。

示例 5-2 填写了项目的元数据。我突出了与示例 3-4 的一些不同之处。（稍后您将使用命令行界面添加依赖项。）

##### 示例 5-2\. Poetry 项目的元数据

```py
[tool.poetry]
name = "random-wikipedia-article"
version = "0.1.0"
description = "Display extracts from random Wikipedia articles"
keywords = ["wikipedia"]
license = "MIT" ![1](img/1.png)
classifiers = [
    "License :: OSI Approved :: MIT License",
    "Development Status :: 3 - Alpha",
    "Environment :: Console",
    "Topic :: Games/Entertainment :: Fortune Cookies",
]
authors = ["Your Name <you@example.com>"] ![2](img/2.png)
readme = "README.md" ![3](img/3.png)
homepage = "https://yourname.dev/projects/random-wikipedia-article" ![4](img/4.png)
repository = "https://github.com/yourname/random-wikipedia-article"
documentation = "https://readthedocs.io/random-wikipedia-article"
packages = [{include = "random_wikipedia_article", from = "src"}]

[tool.poetry.dependencies]
python = ">=3.10" ![5](img/5.png)

[tool.poetry.urls]
Issues = "https://github.com/yourname/random-wikipedia-article/issues"

[tool.poetry.scripts]
random-wikipedia-article = "random_wikipedia_article:main"
```

![1](img/#co_managing_projects_with_poetry_CO1-1)

`license`字段是一个带有 SPDX 标识符的字符串，而不是表格。

![2](img/#co_managing_projects_with_poetry_CO1-2)

`authors`字段包含格式为`"name <email>"`的字符串，而不是表格。Poetry 会从 Git 中预填充此字段的姓名和电子邮件地址。

![3](img/#co_managing_projects_with_poetry_CO1-3)

`readme`字段是一个字符串，其中包含文件路径。您还可以将多个文件指定为字符串数组，例如*README.md*和*CHANGELOG.md*。Poetry 将它们之间用空行连接起来。

![4](img/#co_managing_projects_with_poetry_CO1-4)

Poetry 专门为一些项目 URL 提供了字段，即其主页、存储库和文档；对于其他 URL，还有一个通用的`urls`表。

![5](img/#co_managing_projects_with_poetry_CO1-5)

`dependencies`中的`python`条目允许您声明兼容的 Python 版本。对于这个项目，您需要 Python 3.10 或更高版本。

表格 5-1\. `tool.poetry`中的元数据字段

| 字段 | 类型 | 描述 | `project`字段 |
| --- | --- | --- | --- |
| `name` | string | 项目名称 | `name` |
| `version` | string | 项目的版本号 | `version` |
| `description` | string | 项目的简要描述 | `description` |
| `keywords` | 字符串数组 | 项目的关键词列表 | `keywords` |
| `readme` | string 或字符串数组 | 项目描述文件或文件列表 | `readme` |
| `license` | string | SPDX 许可证标识符，或“Proprietary” | `license` |
| `authors` | 字符串数组 | 作者列表 | `authors` |
| `maintainers` | 字符串数组 | 维护者列表 | `maintainers` |
| `classifiers` | 字符串数组 | 描述项目的分类器列表 | `classifiers` |
| `homepage` | string | 项目主页的 URL | `urls` |
| `repository` | string | 项目存储库的 URL | `urls` |
| `documentation` | string | 项目文档的 URL | `urls` |
| `urls` | 字符串表格 | 项目的 URL 列表 | `urls` |
| `dependencies` | 字符串或表格的数组 | 所需第三方包的列表 | `dependencies` |
| `extras` | 字符串或表的数组的表 | 可选的第三方包的命名列表 | `optional-dependencies` |
| `groups` | 字符串或表的数组的表 | 开发依赖项的命名列表 | *none* |
| `scripts` | 字符串或表的数组的表 | 入口点脚本 | `scripts` |
| `plugins` | 表的表的字符串 | 入口点组 | `entry-points` |

`tool.poetry` 下的一些 `project` 字段没有直接等价项：

+   没有 `requires-python` 字段；相反，您可以使用 `dependencies` 表中的 `python` 键指定所需的 Python 版本。

+   没有专门的 GUI 脚本字段；请改用 `plugins.gui_scripts`。

+   没有 `dynamic` 字段—​所有元数据都是特定于 Poetry 的，因此声明动态字段没有多大意义。

在我们继续之前，让我们确保 *pyproject.toml* 文件是有效的。Poetry 提供了一个方便的命令，可以针对其配置模式验证 TOML 文件：

```py
$ poetry check
All set!

```

## 包内容

Poetry 允许您指定要包含在分发中的文件和目录，这是 *pyproject.toml* 标准中仍然缺失的功能（表 5-2）。

表 5-2\. `tool.poetry` 中的包内容字段

| Field | Type | Description |
| --- | --- | --- |
| `packages` | 表的数组 | 分发中要包含的模块的模式 |
| `include` | 字符串或表的数组 | 分发中要包含的文件的模式 |
| `exclude` | 字符串或表的数组 | 分发中要排除的文件的模式 |

`packages` 下的每个表都有一个 `include` 键，指定文件或目录。您可以在名称和路径中使用 `*` 和 `**` 通配符。 `from` 键允许您包含来自子目录（如 *src*）的模块。最后，您可以使用 `format` 键限制模块到特定的分发格式；有效值为 `sdist` 和 `wheel`。

`include` 和 `exclude` 字段允许您列出要包含在分发中或从分发中排除的其他文件。如果存在 *.gitignore* 文件，Poetry 将使用该文件填充 `exclude` 字段。除了字符串外，还可以使用带有 `path` 和 `format` 键的表，仅适用于 sdist 或 wheel 文件。 示例 5-3 展示了如何在源分发中包含测试套件。

##### 示例 5-3\. 将测试套件包含在源分发中

```py
packages = [{include = "random_wikipedia_article", from = "src"}]
include  = [{path = "tests", format = "sdist"}]
```

## 源代码

将 示例 5-4 的内容复制到新项目的 *__init__.py* 文件中。

##### 示例 5-4\. `random-wikipedia-article` 的源代码

```py
import httpx
from rich.console import Console

from importlib.metadata import metadata

API_URL = "https://en.wikipedia.org/api/rest_v1/page/random/summary"
USER_AGENT = "{Name}/{Version} (Contact: {Author-email})"

def main():
    fields = metadata("random-wikipedia-article")
    headers = {"User-Agent": USER_AGENT.format_map(fields)}

    with httpx.Client(headers=headers, http2=True) as client:
        response = client.get(API_URL, follow_redirects=True)
        response.raise_for_status()
        data = response.json()

    console = Console(width=72, highlight=False)
    console.print(data["title"], style="bold", end="\n\n")
    console.print(data["extract"])
```

在 *pyproject.toml* 的 `scripts` 部分中声明了一个入口点脚本，因此用户可以使用 `random-wikipedia-article` 调用应用程序。如果您希望用户还可以使用 `py -m random_wikipedia_article` 调用程序，请像 示例 3-3 中显示的那样，创建一个与 *__init__.py* 并列的 *__main__.py* 模块。

# 管理依赖项

让我们为`random-wikipedia-article`添加依赖项，首先是控制台输出库 Rich：

```py
$ poetry add rich
Using version ¹³.7.1 for rich

Updating dependencies
Resolving dependencies... (0.2s)

Package operations: 4 installs, 0 updates, 0 removals

  - Installing mdurl (0.1.2)
  - Installing markdown-it-py (3.0.0)
  - Installing pygments (2.17.2)
  - Installing rich (13.7.1)

Writing lock file

```

如果你运行这个命令后检查*pyproject.toml*，你会发现 Poetry 已经将 Rich 添加到`dependencies`表中（见示例 5-5）：

##### 示例 5-5\. 添加 Rich 后的`dependencies`表

```py
[tool.poetry.dependencies]
python = ">=3.10"
rich = "¹³.7.1"
```

Poetry 还会将包安装到项目的环境中。如果你已经有一个名为*.venv*的虚拟环境，Poetry 会使用它。否则，它会在一个共享位置创建一个虚拟环境（参见“管理环境”）。

## 脱字符约束

脱字符（`^`）是版本指定器的 Poetry 特定扩展，从 Node.js 的包管理器 npm 借鉴而来。*脱字符约束*允许以给定的最低版本发布，除了可能包含破坏性更改的版本，符合[语义化版本规范](https://semver.org)。在`1.0.0`之后，脱字符约束允许补丁版本和次要版本的发布，但不允许主要版本的发布。在`0.*`时代之前，只允许补丁版本的发布——次要版本允许引入破坏性更改。

脱字符约束类似于*波浪线约束*（见“版本指定器”），但后者只允许最后一个版本段增加。例如，以下约束是等效的：

```py
rich = "¹³.7.1"
rich = ">=13.7.1,<14"
```

另一方面，波浪线约束通常排除次要版本：

```py
rich = "~13.7.1"
rich = ">=13.7.1,==13.7.*"
```

脱字符约束对版本设置了一个上限。如“Python 中的上限版本边界”中所解释的那样，如果可能的话，应避免上限。只添加 Rich 的下限：

```py
$ poetry add "rich>=13.7.1"
```

你可以使用这个命令从现有的脱字符约束中移除上限。如果你在第一次添加依赖时指定了额外的内容或标记，你需要再次指定它们。²

## 额外和环境标记

让我们添加`random-wikipedia-article`的另一个依赖项，HTTP 客户端库`httpx`。就像在第四章中一样，你将激活`http2`的额外功能以支持 HTTP/2。

```py
$ poetry add "httpx>=0.27.0" --extras=http2
```

Poetry 相应地更新了*pyproject.toml*文件：

```py
[tool.poetry.dependencies]
python = ">=3.10"
rich = ">=13.7.1"
httpx = {version = ">=0.27.0", extras = ["http2"]}
```

该项目需要较新的 Python 版本，因此不需要`importlib-metadata`的后移版本。如果你需要支持 Python 3.8 之前的版本，可以按以下方式为这些版本添加库：

```py
$ poetry add "importlib-metadata>=7.0.2" --python="<3.8"
```

除了`--python`，`poetry add`命令还支持`--platform`选项，用于限制依赖关系到特定操作系统，比如 Windows。这个选项接受一个平台标识符，格式与标准的`sys.platform`属性相同：`linux`、`darwin`、`win32`。对于其他环境标记，编辑*pyproject.toml*并在依赖关系的 TOML 表中使用`markers`属性：

```py
[tool.poetry.dependencies]
awesome = {version = ">=1", markers = "implementation_name == 'pypy'"}
```

## 锁文件

Poetry 在名为 *poetry.lock* 的文件中记录了每个依赖项的当前版本，包括它们的打包工件的 SHA256 哈希值。如果你打开文件看一眼，你会注意到 `rich` 和 `httpx` 的 TOML 段落，以及它们的直接和间接依赖项。示例 5-6 展示了 Rich 的锁定条目的简化版本。

##### 示例 5-6\. Rich 在 *poetry.lock* 中的 TOML 段落（简化版）

```py
[[package]]
name = "rich"
version = "13.7.1"
python-versions = ">=3.7.0"
dependencies = {markdown-it-py = ">=2.2.0", pygments = ">=2.13.0,<3.0.0"}
files = [
    {file = "rich-13.7.1-py3-none-any.whl", hash = "sha256:4edbae3..."},
    {file = "rich-13.7.1.tar.gz", hash = "sha256:9be308c..."},
]
```

使用命令 `poetry show` 在终端中显示锁定的依赖项。这是我添加 Rich 后的输出：

```py
$ poetry show
markdown-it-py 3.0.0  Python port of markdown-it. Markdown parsing, done right!
mdurl          0.1.2  Markdown URL utilities
pygments       2.17.2 Pygments is a syntax highlighting package written in Python.
rich           13.7.1 Render rich text, tables, progress bars, ...

```

你也可以将依赖项显示为树形结构以可视化它们的关系：

```py
$ poetry show --tree
rich 13.7.1 Render rich text, tables, progress bars, ...
├── markdown-it-py >=2.2.0
│   └── mdurl >=0.1,<1.0
└── pygments >=2.13.0,<3.0.0

```

如果你手动编辑了 *pyproject.toml*，请记得更新锁定文件以反映你的更改：

```py
$ poetry lock --no-update
Resolving dependencies... (0.1s)

Writing lock file

```

没有 `--no-update` 选项，Poetry 将每个锁定的依赖项升级到其约束范围内的最新版本。

你可以检查 *poetry.lock* 文件是否与 *pyproject.toml* 保持一致：

```py
$ poetry check --lock

```

提前解决依赖关系可以让你以可靠和可重复的方式部署应用程序。它还为团队中的开发人员提供了一个共同的基线，并使检查更加确定性——避免在持续集成（CI）中出现意外情况。你应该将 *poetry.lock* 提交到源代码控制中以获得这些好处。

Poetry 的锁定文件设计用于跨操作系统和 Python 解释器工作。如果你的代码必须在不同环境中运行，或者如果你是一个开源维护者，有来自世界各地的贡献者，拥有一个单一的环境无关或“通用”的锁定文件是有益的。

相比之下，编译的需求文件很快变得难以管理。如果你的项目支持 Python 的最近四个主要版本上的 Windows、macOS 和 Linux，你将需要管理十几个需求文件。添加另一个处理器架构或 Python 实现只会让情况变得更糟。

通用的锁定文件却并非没有代价。Poetry 在安装包到环境中时会重新解决依赖关系。其锁定文件本质上是一种缩小的世界观：它记录了项目在特定环境中可能需要的每个包。相比之下，编译的需求文件是环境的精确镜像。这使得它们更易于审计，更适合安全部署。

## 更新依赖项

你可以使用单个命令将锁定文件中的所有依赖项更新到它们的最新版本：

```py
$ poetry update

```

你还可以提供一个特定的直接或间接依赖项来进行更新：

```py
$ poetry update rich

```

`poetry update` 命令不会修改 *pyproject.toml* 中的项目元数据。它只会更新兼容版本范围内的依赖项。如果需要更新版本范围，请使用带有新约束的 `poetry add`，包括任何额外的和标记。或者，编辑 *pyproject.toml* 并使用 `poetry lock --no-update` 刷新锁定文件。

如果你不再需要一个包用于你的项目，使用 `poetry remove` 将其移除：

```py
$ poetry remove *<package>*

```

# 管理环境

Poetry 的 `add`、`update` 和 `remove` 命令不仅会更新 *pyproject.toml* 和 *poetry.lock* 文件中的依赖关系。它们还通过安装、更新或删除软件包，将项目环境与锁定文件同步。Poetry 根据需要为项目创建虚拟环境。

默认情况下，Poetry 将所有项目的环境存储在共享文件夹中。配置 Poetry 将环境保存在项目内的 *.venv* 目录中：

```py
$ poetry config virtualenvs.in-project true

```

此设置使得环境在生态系统中的其他工具（如 `py` 和 `uv`）中可发现，当您需要检查其内容时，在项目中具有该目录非常方便。尽管此设置将您限制为单个环境，但这种限制很少是一个问题。像 Nox 和 tox 这样的工具专门用于跨多个环境进行测试（参见 第 8 章）。

您可以使用命令 `poetry env info --path` 检查当前环境的位置。如果您想为项目创建一个干净的状态，请使用以下命令删除现有环境并使用指定的 Python 版本创建新环境：

```py
$ poetry env remove --all
$ poetry env use 3.12

```

您可以重新运行第二条命令以在不同的解释器上重新创建环境。您不仅可以使用类似 `3.12` 的版本，还可以为 PyPy 解释器传递类似 `pypy3` 的命令，或者为系统 Python 传递类似 `/usr/bin/python3` 的完整路径。

在使用环境之前，您应该安装项目。Poetry 执行可编辑安装，因此环境会立即反映任何代码更改：

```py
$ poetry install

```

通过使用 `poetry shell` 启动一个 shell 会话来进入项目环境。Poetry 使用当前 shell 的激活脚本激活虚拟环境。在环境激活后，您可以从 shell 提示符中运行应用程序。完成后，只需退出 shell 会话：

```py
$ poetry shell
(random-wikipedia-article-py3.12) $ random-wikipedia-article
(random-wikipedia-article-py3.12) $ exit

```

您还可以在当前的 shell 会话中运行应用程序，使用命令 `poetry run`：

```py
$ poetry run random-wikipedia-article

```

该命令还适用于在项目环境中启动交互式 Python 会话：

```py
$ poetry run python
>>> from random_wikipedia_article import main
>>> main()

```

使用 `poetry run` 运行程序时，Poetry 会激活虚拟环境，而不启动 shell。这通过将环境添加到程序的 `PATH` 和 `VIRTUAL_ENV` 变量中实现（参见 “激活脚本”）。

###### 提示

只需在 Linux 和 macOS 上输入 `py`，即可为您的 Poetry 项目获取 Python 会话。这需要 Unix 上的 Python 启动器，并且您必须配置 Poetry 以使用项目内的环境。

# 依赖组

Poetry 允许您声明开发依赖项，这些依赖项组织在依赖组中。依赖组不是项目元数据的一部分，对最终用户不可见。让我们从 “开发依赖项” 中添加依赖组：

```py
$ poetry add --group=tests pytest pytest-sugar
$ poetry add --group=docs sphinx

```

Poetry 在 *pyproject.toml* 的 `group` 表下添加依赖组：

```py
[tool.poetry.group.tests.dependencies]
pytest = "⁸.1.1"
pytest-sugar = "¹.0.0"

[tool.poetry.group.docs.dependencies]
sphinx = "⁷.2.6"
```

默认情况下，依赖组会安装到项目环境中。您可以使用其 `optional` 键将组标记为可选，例如：

```py
[tool.poetry.group.docs]
optional = true

[tool.poetry.group.docs.dependencies]
sphinx = "⁷.2.6"
```

###### 警告

使用 `poetry add` 添加依赖组时，请勿指定 `--optional` 标志，因为它不会将组标记为可选。该选项用于指定作为额外项的可选依赖项；在依赖组的上下文中，它没有有效用途。

`poetry install` 命令有几个选项，可更精细地控制哪些依赖项安装到项目环境中（表 5-3）。

表 5-3\. 使用 `poetry install` 安装依赖项

| 选项 | 描述 |
| --- | --- |
| `--with=*<group>*` | 将一个依赖组包含在安装中。 |
| `--without=*<group>*` | 从安装中排除一个依赖组。 |
| `--only=*<group>*` | 从安装中排除所有其他依赖组。 |
| `--no-root` | 从安装中排除项目本身。 |
| `--only-root` | 从安装中排除所有依赖项。 |
| `--sync` | 从环境中删除除安装计划外的包。 |

您可以指定单个组或多个组（用逗号分隔）。特殊组 `main` 指的是在 `tool.poetry.dependencies` 表中列出的包。使用选项 `--only=main` 可以从安装中排除所有开发依赖项。类似地，选项 `--without=main` 可让您限制安装到开发依赖项。

# 包仓库

Poetry 允许您将包上传到 Python 包索引（PyPI）和其他包仓库。它还允许您配置从中添加包到项目的仓库。本节同时涵盖与包仓库的互动的发布者和使用者两方面。

###### 注意

如果您在本节中跟随进行，请不要将示例项目上传到 PyPI。请使用 TestPyPI 仓库，它是一个用于测试、学习和实验的游乐场。

## 发布包到包仓库

在您可以上传包到 PyPI 之前，您需要一个帐户和 API 令牌来验证仓库，如 “使用 Twine 上传包” 中所述。接下来，将 API 令牌添加到 Poetry：

```py
$ poetry config pypi-token.pypi *<token>*

```

您可以使用标准工具（如 `build`）或 Poetry 的命令行界面为 Poetry 项目创建包：

```py
$ poetry build
Building random-wikipedia-article (0.1.0)
  - Building sdist
  - Built random_wikipedia_article-0.1.0.tar.gz
  - Building wheel
  - Built random_wikipedia_article-0.1.0-py3-none-any.whl

```

与 `build` 类似，Poetry 将包放置在 *dist* 目录中。您可以使用 `poetry publish` 发布 *dist* 中的包：

```py
$ poetry publish

```

您还可以将这两个命令合并为一个：

```py
$ poetry publish --build

```

让我们将示例项目上传到 TestPyPI，这是 Python 包索引的一个单独实例，用于测试发布。如果您想将包上传到 PyPI 之外的仓库，需要将该仓库添加到 Poetry 配置中：

```py
$ poetry config repositories.testpypi https://test.pypi.org/legacy/

```

首先，在 TestPyPI 上创建一个帐户和 API 令牌。接下来，配置 Poetry 使用该令牌上传到 TestPyPI：

```py
$ poetry config pypi-token.testpypi *<token>*

```

您现在可以在发布项目时指定存储库。请随意尝试将其应用于您自己版本的示例项目：

```py
$ poetry publish --repository=testpypi
Publishing random-wikipedia-article (0.1.0) to TestPyPI
 - Uploading random_wikipedia_article-0.1.0-py3-none-any.whl 100%
 - Uploading random_wikipedia_article-0.1.0.tar.gz 100%

```

某些包存储库使用带有用户名和密码的 HTTP 基本身份验证。您可以像这样配置此类存储库的凭据：

```py
$ poetry config http-basic.*<repo>* *<username>*

```

该命令提示您输入密码，并将其存储在系统密钥环中（如果可用），或者存储在磁盘上的 *auth.toml* 文件中。

或者，您还可以通过环境变量配置存储库（将 `*<REPO>*` 替换为大写的存储库名称，例如 `PYPI`）：

```py
$ export POETRY_REPOSITORIES_*<REPO>*_URL=*<url>*
$ export POETRY_PYPI_TOKEN_*<REPO>*=*<token>*
$ export POETRY_HTTP_BASIC_*<REPO>*_USERNAME=*<username>*
$ export POETRY_HTTP_BASIC_*<REPO>*_PASSWORD=*<password>*

```

诗歌还支持通过相互 TLS 安全保护或使用自定义证书颁发机构的存储库；详细信息请参阅[官方文档](https://python-poetry.org/docs/repositories/)。

## 从包源获取包

在上文中，您已经了解了如何将您的包上传到除 PyPI 外的存储库。Poetry 还支持消费端的备用存储库：您可以从非 PyPI 的来源向项目添加包。虽然上传目标是用户设置并存储在 Poetry 配置中，但包源是项目设置，并存储在 *pyproject.toml* 中。

使用命令 `poetry source add` 添加包源：

```py
$ poetry source add *<repo>* *<url>* --priority=supplemental

```

如果要禁用 PyPI 并希望从主要来源配置，请在这里配置补充源（默认优先级）：

```py
$ poetry source add *<repo>* *<url>*

```

您可以像配置存储库一样为包源配置凭据：

```py
$ poetry config http-basic.*<repo>* *<username>*

```

您现在可以从备用来源添加包：

```py
$ poetry add httpx --source=*<repo>*

```

下面的命令列出了项目的包源：

```py
$ poetry source show

```

###### 警告

在添加来自辅助来源的包时，请指定来源。否则，Poetry 在查找包时会搜索所有来源。攻击者可以向 PyPI 上传与您内部包相同名称的恶意包（*依赖混淆攻击*）。

# 使用插件扩展 Poetry

Poetry 提供了一个插件系统，可以扩展其功能。使用 pipx 将插件注入到 Poetry 的环境中：

```py
$ pipx inject poetry *<plugin>*

```

用 `*<plugin>*` 替换 PyPI 上插件的名称。

如果插件影响项目的构建阶段，请在 *pyproject.toml* 中的构建依赖项中添加它。例如，请参阅“动态版本插件”。

默认情况下，pipx 在没有注入包的情况下升级应用程序。使用选项 `--include-injected` 也升级应用程序插件。

```py
$ pipx upgrade --include-injected poetry

```

如果您不再需要该插件，请从注入的包中移除它：

```py
$ pipx uninject poetry *<plugin>*

```

如果您不确定已安装了哪些插件，请像这样列出它们：

```py
$ poetry self show plugins

```

在本节中，我将为您介绍 Poetry 的三个有用插件：

+   `poetry-plugin-export` 允许您生成需求和约束文件

+   `poetry-plugin-bundle` 允许您将项目部署到虚拟环境中

+   `poetry-dynamic-versioning` 从版本控制系统中填充项目版本

## 使用导出插件生成需求文件

Poetry 的锁定文件非常适合确保您团队中的每个人和每个部署环境都使用相同的依赖关系。但是如果您在某些情况下无法使用 Poetry 该怎么办？例如，您可能需要在仅具有 Python 解释器和捆绑的 pip 的系统上部署项目。

截至目前为止，在更广泛的 Python 世界中没有锁定文件标准；支持锁定文件的每个打包工具都实现了自己的格式。³ 这些锁定文件格式都不受 pip 支持。但是我们确实有要求文件。

要求文件允许您将软件包固定到确切的版本，要求其构件与加密哈希匹配，并使用环境标记将软件包限制在特定的 Python 版本和平台上。如果您可以从 *poetry.lock* 生成一个要求文件以与非 Poetry 环境进行互操作，那会很好吗？这正是导出插件所实现的。

使用 pipx 安装插件：

```py
$ pipx inject poetry poetry-plugin-export

```

该插件为 `poetry export` 命令提供动力，该命令具有 `--format` 选项以指定输出格式。默认情况下，该命令写入标准输出流；使用 `--output` 选项指定目标文件。

```py
$ poetry export --format=requirements.txt --output=requirements.txt

```

将要求文件分发到目标系统并使用 pip 安装依赖项（通常是安装您项目的 wheel 之后）。

```py
$ python3 -m pip install -r requirements.txt

```

将导出为 requirements 格式对部署之外的用途也很有用。许多工具都使用 requirements 文件作为事实上的行业标准。例如，您可以使用像 [safety](https://pyup.io/safety/) 这样的工具扫描具有已知安全漏洞的依赖关系的要求文件。

## 使用 Bundle 插件部署环境

在上一节中，您看到了如何在没有 Poetry 的系统上部署项目。如果您确实有 Poetry 可用，您可能会想知道：是否可以只使用 `poetry install` 部署？您可以，但是 Poetry 执行的是可编辑安装您的项目—​您将从源代码树运行应用程序。这在生产环境中可能不可接受。可编辑安装还限制了将虚拟环境发送到另一个目的地的能力。

Bundle 插件允许您将项目和已锁定的依赖项部署到您选择的虚拟环境中。它创建环境，从锁定文件安装依赖项，然后构建并安装您项目的 wheel。

使用 pipx 安装插件：

```py
$ pipx inject poetry poetry-plugin-bundle

```

安装后，您将看到一个新的 `poetry bundle` 子命令。让我们使用它将项目捆绑到名为 *app* 的虚拟环境中。使用 `--python` 选项指定环境的解释器，并使用 `--only=main` 选项排除开发依赖关系。

```py
$ poetry bundle venv --python=/usr/bin/python3 --only=main app

  - Bundled random-wikipedia-article (0.1.0) into app

```

通过运行应用程序的入口点脚本来测试环境。⁴

```py
$ app/bin/random-wikipedia-article

```

您可以使用 bundle 插件为生产环境创建一个精简的 Docker 镜像。Docker 支持 *多阶段构建*，第一阶段在全功能的构建环境中构建应用程序，第二阶段将构建产物复制到精简的运行时环境中。这使您能够在生产环境中快速部署、减少冗余并降低潜在漏洞的风险。

在 示例 5-7 中，第一个阶段安装 Poetry 和 bundle 插件，复制 Poetry 项目，并将其打包成一个自包含的虚拟环境。第二个阶段将虚拟环境复制到一个精简的 Python 镜像中。

##### 示例 5-7\. 使用 Poetry 的多阶段 Dockerfile

```py
FROM debian:12-slim AS builder ![1](img/1.png)
RUN apt-get update && \
    apt-get install --no-install-suggests --no-install-recommends --yes pipx
ENV PATH="/root/.local/bin:${PATH}"
RUN pipx install poetry
RUN pipx inject poetry poetry-plugin-bundle
WORKDIR /src
COPY . .
RUN poetry bundle venv --python=/usr/bin/python3 --only=main /venv

FROM gcr.io/distroless/python3-debian12 ![2](img/2.png)
COPY --from=builder /venv /venv ![3](img/3.png)
ENTRYPOINT ["/venv/bin/random-wikipedia-article"] ![4](img/4.png)
```

![1](img/#co_managing_projects_with_poetry_CO2-1)

第二个 `FROM` 指令定义了部署到生产环境的镜像。基础镜像是 *distroless* Python 镜像，适用于 Debian 稳定版：提供 Python 语言支持但不包括操作系统。

![2](img/#co_managing_projects_with_poetry_CO2-2)

第一个 `FROM` 指令定义了构建阶段的基础镜像，用于构建和安装项目。基础镜像是 Debian 稳定版的精简变体。

![3](img/#co_managing_projects_with_poetry_CO2-3)

`COPY` 指令允许您从构建阶段复制虚拟环境过来。

![4](img/#co_managing_projects_with_poetry_CO2-4)

`ENTRYPOINT` 指令允许您在用户使用该镜像运行 `docker run` 时运行入口点脚本。

如果已安装 Docker，则可以尝试此操作。首先，在项目中创建一个 *Dockerfile*，内容来自 示例 5-7。接下来，构建并运行 Docker 镜像：

```py
$ docker build -t random-wikipedia-article .
$ docker run --rm -ti random-wikipedia-article

```

您应该在终端中看到 `random-wikipedia-article` 的输出。

## 动态版本插件

动态版本插件从 Git 标签中填充项目元数据中的版本信息。将版本信息集中管理可减少变更（参见 “Single-sourcing the project version”）。该插件基于 Dunamai，一个用于从版本控制系统标签中派生符合标准的版本字符串的 Python 库。

使用 pipx 安装插件并为您的项目启用它：

```py
$ pipx inject poetry "poetry-dynamic-versioning[plugin]"
$ poetry dynamic-versioning enable

```

第二步在 *pyproject.toml* 的 `tool` 部分启用插件：

```py
[tool.poetry-dynamic-versioning]
enable = true
```

请记住，您已全局安装了 Poetry 插件。显式选择加入确保您不会意外地开始在不相关的 Poetry 项目中重写版本字段。

前端构建工具如 pip 和 `build` 在构建项目时需要该插件。因此，启用插件还将其添加到 *pyproject.toml* 中作为构建依赖项。该插件提供了自己的构建后端，用于包装 Poetry 提供的构建系统：

```py
[build-system]
requires = ["poetry-core>=1.0.0", "poetry-dynamic-versioning>=1.0.0,<2.0.0"]
build-backend = "poetry_dynamic_versioning.backend"
```

Poetry 仍然要求在其自己的部分中有 `version` 字段。将该字段设置为 `"0.0.0"` 表示未使用该字段。

```py
[tool.poetry]
version = "0.0.0"
```

现在你可以添加一个 Git 标签来设置你的项目版本：

```py
$ git tag v1.0.0
$ poetry build
Building random-wikipedia-article (1.0.0)
  - Building sdist
  - Built random_wikipedia_article-1.0.0.tar.gz
  - Building wheel
  - Built random_wikipedia_article-1.0.0-py3-none-any.whl

```

插件还会替换 Python 模块中的 `__version__` 属性。这在大多数情况下都可以直接使用，但如果你使用它，你需要声明 *src* 布局：

```py
[tool.poetry-dynamic-versioning]
enable = true
substitution.folders = [{path = "src"}]
```

让我们为应用程序添加一个 `--version` 选项。编辑包中的 *__init__.py* 文件，添加以下几行：

```py
import argparse

__version__ = "0.0.0"

def main():
    parser = argparse.ArgumentParser(prog="random-wikipedia-article")
    parser.add_argument(
        "--version", action="version", version=f"%(prog)s {__version__}"
    )
    parser.parse_args()
    ...
```

在继续之前，请提交你的更改，但不要添加另一个 Git 标签。让我们在项目的新安装中尝试这个选项：

```py
$ rm -rf .venv
$ uv venv
$ uv pip install --no-cache .
$ py -m random_wikipedia_article --version
random-wikipedia-article 1.0.0.post1.dev0+51c266e

```

正如你所看到的，插件在构建过程中重写了 `__version__` 属性。由于你没有给提交打标签，Dunamai 将版本标记为 `1.0.0` 的开发后发布，并使用本地版本标识符附加提交哈希。

# 摘要

Poetry 提供了统一的工作流来管理打包、依赖和环境。Poetry 项目与标准工具兼容：你可以使用 `build` 构建它们，并使用 Twine 将它们上传到 PyPI。但是 Poetry 命令行界面还提供了这些任务和更多任务的便捷缩写。

Poetry 在其锁文件中记录了精确的软件包工作集，为你提供确定性的部署和检查，以及与他人合作时一致的体验。Poetry 可以跟踪开发依赖项；它将它们组织在你可以单独或一起安装的依赖组中。你可以使用插件扩展 Poetry，例如将项目部署到虚拟环境或从 Git 派生版本号。

如果你需要为一个应用程序进行可复制的部署，如果你的团队在多个操作系统上开发，或者如果你觉得标准的工具链给你的工作流增加了太多负担，那么你应该试试 Poetry。

¹ Sébastien Eustace: [“PEP 621 的支持”，](https://github.com/python-poetry/roadmap/issues/3) 2020 年 11 月 6 日。

² 这个命令还会保持你的锁文件和项目环境保持更新。如果你编辑了 *pyproject.toml* 中的约束条件，你需要自己做这些。继续阅读以了解有关锁文件和环境的更多信息。

³ 除了 Poetry 自己的 *poetry.lock* 和密切相关的 PDM 锁文件格式外，还有 pipenv 的 *Pipfile.lock* 和 Conda 环境的 `conda-lock` 格式。

⁴ 如果你在 Windows 上，将 *bin* 替换为 *Scripts*。
