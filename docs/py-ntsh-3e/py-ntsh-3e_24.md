# 第二十四章：打包程序和扩展

*本章内容有所删减以适应印刷出版，我们在这本书的 GitHub 存储库中的[在线版本](https://oreil.ly/python-nutshell-24)提供了更多材料。在线版本中我们还描述了诗歌（poetry），这是一个符合现代标准的 Python 构建系统，并将其与传统的 setuptools 方法进行了比较，此外还涵盖了其他主题（完整列表请参见“在线材料”）。*

假设您有一些 Python 代码需要交付给其他人和团体。它在您的机器上运行正常，但现在您需要使其在其他人那里也能正常运行。这涉及将您的代码打包成适合的格式并提供给预期的受众。

自上一版以来，Python 打包生态系统的质量和多样性有了很大改善，其文档组织更为完善、内容更加完整。这些改进基于对 Python 源树格式的仔细规定，独立于任何特定构建系统的规定，详见[PEP 517](https://oreil.ly/Vm1QZ)，“独立于构建系统的 Python 源树格式”，以及[PEP 518](https://oreil.ly/KwMjb)，“为 Python 项目指定最低构建系统要求”，后者的“原理”部分简要描述了为何需要进行这些改变，其中最重要的是不再需要运行 *setup.py* 文件来发现（据推测是通过观察回溯信息）构建的要求。

PEP 517 的主要目的是指定一个名为 *pyproject.toml* 的文件中的构建定义格式。该文件被组织成称为“表”的部分，每个表的标题包含在方括号中的表名，类似于配置文件。每个表包含各种参数的值，包括名称、等号和值。Python 3.11+ 包括了用于提取这些定义的[tomllib](https://oreil.ly/fdSIV)模块，具有类似于 json 模块中的 load 和 loads 方法。¹

虽然 Python 生态系统中越来越多的工具使用这些现代标准，但您仍应该预期会继续遇到更传统的基于 setuptools 的构建系统（它本身正在[过渡](https://oreil.ly/aF454)到 PEP 517 推荐的*pyproject.toml*基础）。关于可用的打包工具的优秀概述，请参阅由 Python Packaging Authority (PyPA) 维护的[列表](https://oreil.ly/ttIW6)。

为了解释打包，我们首先描述其发展，然后讨论 poetry 和 setuptools。其他符合 PEP 517 标准的构建工具包括 [flit](https://oreil.ly/sF7Zp) 和 [hatch](https://oreil.ly/AKylH)，随着互操作性的不断改善，你应该期待它们的数量将继续增长。对于分发相对简单的纯 Python 包，我们还介绍了标准库模块 zipapp，并在本章中通过一个简短的部分解释如何访问作为包一部分捆绑的数据。

# 本章未涵盖的内容

除了 PyPA 认可的方法外，还有许多其他可能的 Python 代码分发方式，远远超出了单章节的覆盖范围。我们不涵盖以下打包和分发主题，这些主题可能会对希望分发 Python 代码的人感兴趣：

+   使用 [conda](https://docs.conda.io)

+   使用 [Docker](https://docs.docker.com)

+   从 Python 代码创建二进制可执行文件的各种方法，例如以下工具（这些工具对于复杂项目的设置可能有些棘手，但通过扩大应用程序的潜在受众来回报努力）：

    +   [PyInstaller](https://pyinstaller.org)，它接受一个 Python 应用程序并将所有必需的依赖（包括 Python 解释器和必要的扩展库）打包成一个单独的可执行程序，可以作为独立应用程序分发。每个体系结构都有适用于 Windows、macOS 和 Linux 的版本，但每个体系结构只能生成自己的可执行文件。

    +   [PyOxidizer](https://oreil.ly/GC_5w)，这是同名实用工具集中的主要工具，不仅允许创建独立的可执行文件，还可以创建 Windows 和 macOS 安装程序及其他工件。

    +   [cx_Freeze](https://oreil.ly/pnWdA)，它创建一个包含 Python 解释器、扩展库和 Python 代码 ZIP 文件的文件夹。你可以将其转换为 Windows 安装程序或 macOS 磁盘映像。

# Python 打包的简要历史

在虚拟环境出现之前，维护多个 Python 项目并避免它们不同依赖需求的冲突是一项复杂的任务，需要仔细管理 `sys.path` 和 `PYTHONPATH` 环境变量。如果不同项目需要同一个依赖的不同版本，没有一个单独的 Python 环境可以同时支持它们。如今，每个虚拟环境（参见“Python 环境”以深入了解此主题）都有自己的 *site_packages* 目录，可以通过多种便捷的方式安装第三方和本地的包和模块，大大减少了对机制的需求，使得这些问题基本上不再需要考虑。²

当 Python 包索引于 2003 年构思时，没有这样的功能可用，也没有统一的方法来打包和分发 Python 代码。开发人员必须仔细地为他们所工作的每个不同项目调整他们的环境。随着 distutils 标准库包的开发，情况发生了变化，很快被第三方的 setuptools 包及其 easy_install 实用工具所利用。现在已过时的跨平台 *egg* 打包格式是 Python 包分发的第一个单文件格式的定义，允许从网络源轻松下载和安装 eggs。安装一个包使用了 *setup.py* 组件，其执行将使用 setuptools 的特性将包的代码集成到现有的 Python 环境中。要求使用第三方（即不是标准发行版的一部分）模块，如 setuptools，显然不是一个完全令人满意的解决方案。

与这些发展同时进行的是 virtualenv 包的创建，它通过为不同项目使用的 Python 环境提供清晰的分离大大简化了普通 Python 程序员的项目管理。在此之后不久，基于 setuptools 背后的思想，pip 实用程序被引入。使用源树而不是 eggs 作为其分发格式，pip 不仅可以安装软件包，还可以卸载它们。它还可以列出虚拟环境的内容，并接受项目依赖项的带版本的列表，按照约定存储在名为 *requirements.txt* 的文件中。

setuptools 的开发有些古怪，对社区需求反应不够灵活，因此创建了一个名为 distribute 的分支作为一个可直接替换的解决方案（它安装在 setuptools 名下），以便允许更具合作性的开发工作进行。这最终被合并回了 setuptools 代码库中，现在由 PyPA 控制：能够做到这一点肯定了 Python 的开源许可政策的价值。

-3.11 distutils 包最初设计为标准库组件，帮助安装扩展模块（特别是那些用编译语言编写的模块，在 [第二十五章](https://oreil.ly/python-nutshell-25) 中有介绍）。尽管它目前仍然存在于标准库中，但它已被弃用，并计划在版本 3.12 中删除，届时可能会并入 setuptools。出现了许多其他工具，符合 PEP 517 和 518。在本章中，我们将介绍不同的方法来将额外的功能安装到 Python 环境中。

随着[PEP 425](https://oreil.ly/vB13q)，“内置分发的兼容性标签”，以及[PEP 427](https://oreil.ly/B_xwu)，“Wheel 二进制包格式”，Python 终于有了一个二进制分发格式的规范（*wheel*，其定义已经[更新](https://oreil.ly/XYnsg)），允许在不同架构下分发编译的扩展包，当没有合适的二进制 wheel 可用时则回退到源码安装。

[PEP 453](https://oreil.ly/FhWDt)，“Python 安装中 pip 的显式引导”，决定 pip 实用程序应成为 Python 中首选的安装包的方式，并建立了一个独立于 Python 的更新过程，以便可以不必等待新的语言发布而提供新的部署功能。

这些发展以及许多其他使 Python 生态系统合理化的努力都归功于 PyPA，Python 的领导“Steering Council”已将与打包和分发相关的大多数事项委托给他们。要深入了解本章节的更高级内容，请参阅[“Python 打包用户指南”](https://packaging.python.org)，该指南为希望广泛发布其 Python 软件的任何人提供了明智的建议和有用的指导。

# 在线资料

正如本章节开头所提到的，[本章节的在线版本](https://oreil.ly/python-nutshell-24)包含额外的材料。讨论的主题包括：

+   构建过程

+   入口点

+   发布格式

+   poetry

+   setuptools

+   分发您的包

+   zipapp

+   访问与您的代码一起提供的数据

¹ 旧版本的用户可以使用**pip install toml**从 PyPI 安装该库。

² 请注意，某些软件包对虚拟环境不太友好。幸运的是，这种情况很少见。
