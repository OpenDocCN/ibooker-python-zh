# 第二章：Python 解释器

在 Python 中开发软件系统时，通常会编写包含 Python 源代码的文本文件。您可以使用任何文本编辑器来完成这项工作，包括我们在“Python 开发环境”中列出的编辑器。然后，您可以使用 Python 编译器和解释器处理源文件。您可以直接执行此操作，也可以在集成开发环境 (IDE) 中执行此操作，或者通过嵌入 Python 的另一个程序执行此操作。Python 解释器还允许您交互地执行 Python 代码，就像 IDE 一样。

# python 程序

Python 解释器程序的运行方式是 **python**（在 Windows 上命名为 *python.exe*）。该程序包括解释器本身和 Python 编译器，后者在需要时会隐式调用已导入的模块。根据您的系统，该程序可能必须位于 PATH 环境变量中列出的目录中。或者，与任何其他程序一样，您可以在命令 (shell) 提示符处提供其完整路径名，或在运行它的 shell 脚本 (或快捷方式目标等) 中提供其完整路径名。¹

在 Windows 上，按下 Windows 键然后开始键入 **python**。出现“Python 3.x”（命令行版本）以及其他选择，例如“IDLE”（Python GUI）。

## 环境变量

除了 PATH 外，其他环境变量也会影响 **python** 程序。其中一些与在命令行传递给 **python** 的选项具有相同的效果，就像我们在下一节中展示的那样，但是有几个环境变量提供了通过命令行选项不可用的设置。下面列出了一些经常使用的环境变量；有关完整详情，请参阅[在线文档](https://oreil.ly/sYdEK)：

PYTHONHOME

Python 安装目录。必须在此目录下包含一个 *lib* 子目录，其中包含 Python 标准库。在类 Unix 系统上，标准库模块应该位于 *lib/python-3.x* 中，其中 *x* 是次要 Python 版本。如果没有设置 PYTHONHOME，则 Python 会对安装目录进行一个明智的猜测。

PYTHONPATH

Python 可以导入模块的目录列表，在类 Unix 系统上用冒号分隔，在 Windows 上用分号分隔。这个列表扩展了 Python 的 sys.path 变量的初始值。我们在第七章中介绍了模块、导入和 sys.path。

PYTHONSTARTUP

每次启动交互式解释器会话时运行的 Python 源文件的名称。如果您没有设置此变量，或者设置为找不到的文件的路径，那么就不会运行这样的文件。当您运行 Python 脚本时，PYTHONSTARTUP 文件不会运行；它只在您启动交互式会话时运行。

如何设置和检查环境变量取决于您的操作系统。 在 Unix 中，使用 shell 命令，通常在启动 shell 脚本中。 在 Windows 上，按 Windows 键并开始键入 **environment** **var**，然后会出现一些快捷方式：一个用于用户环境变量，另一个用于系统环境变量。 在 Mac 上，您可以像在其他类 Unix 系统上一样工作，但您有更多的选择，包括专门针对 MacPython 的 IDE。 有关在 Mac 上使用 Python 的更多信息，请参阅 [在线文档中的“在 Mac 上使用 Python”](https://oreil.ly/Co1au)。

## 命令行语法和选项

Python 解释器的命令行语法可以总结如下：

```py
[*`path`*]`python` {*`options`*} [-`c` *`command`* | -`m` *`module`* | *`file`* | -] {*`args`*}
```

方括号（[]）表示可选内容，大括号（{}）表示可能出现的项，竖线（|）表示多个选项之间的选择。Python 中使用斜杠（/）表示文件路径，就像在 Unix 中一样。

在命令行上运行 Python 脚本可以简单地如下所示：

```py
$ python hello.py
Hello World
```

您还可以明确提供脚本的路径：

```py
$ python ./hello/hello.py
 Hello World
```

脚本的文件名可以是绝对路径或相对路径，并且不一定需要特定的扩展名（虽然使用 *.py* 扩展名是常规做法）。

*选项* 是区分大小写的短字符串，以连字符开头，请求 **python** 以非默认行为运行。 **python** 只接受以连字符（-）开头的选项。 最常用的选项列在 表 2-1 中。 每个选项的描述都给出了环境变量（如果有的话），设置该变量会请求对应的行为。 许多选项都有更长的版本，以两个连字符开头，如 **python -h** 所示。 有关详细信息，请参阅 [在线文档](https://oreil.ly/1ZcA9)。 

表 2-1\. 经常使用的 python 命令行选项

| 选项 | 意义（及对应的环境变量，如果有的话） |
| --- | --- |
| **-B** | 不将字节码文件保存到磁盘上（PYTHONDONTWRITEBYTECODE） |
| **-c** | 在命令行内给出 Python 语句 |
| **-E** | 忽略所有环境变量 |
| **-h** | 显示完整的选项列表，然后终止 |
| **-i** | 在文件或命令运行后运行交互会话（PYTHONINSPECT） |
| **-m** | 指定要作为主脚本运行的 Python 模块 |
| **-O** | 优化字节码（PYTHONOPTIMIZE）—注意这是大写字母 O，而不是数字 0 |
| **-OO** | 类似于 **-O**，但同时从字节码中删除文档字符串 |
| **-S** | 在启动时省略隐式 **import** site（在 “站点定制” 中有介绍） |
| **-t**, **-tt** | 警告不一致的制表符使用（**-tt** 对相同问题发出错误而不仅仅是警告） |
| **-u** | 使用无缓冲的二进制文件进行标准输出和标准错误（PYTHONUNBUFFERED） |
| **-v** | 详细跟踪模块导入和清理操作（PYTHONVERBOSE） |
| **-V** | 打印 Python 版本号，然后终止 |
| **-W arg** | 向警告过滤器添加一个条目（参见“warnings 模块”） |
| **-x** | 排除（跳过）脚本源代码的第一行 |

当您希望在运行某些脚本后立即获得交互式会话并且顶级变量仍然完整且可供检查时，请使用**-i**。对于正常的交互式会话，您不需要**-i**，尽管它也无害。

**-O** 和 **-OO** 在您导入的模块生成的字节码中节省了时间和空间，将**assert**语句转换为无操作，正如我们在“assert 语句”中所述。**-OO** 还会丢弃文档字符串。²

在选项后，如果有的话，通过将文件路径添加到该脚本来告诉 Python 要运行哪个脚本。而不是文件路径，你可以使用**-c** *command*来执行 Python 代码字符串命令。*command*通常包含空格，因此你需要在其周围添加引号以满足操作系统的 shell 或命令行处理器的要求。某些 shell（例如，[**bash**](https://oreil.ly/seIne)）允许您将多行输入作为单个参数，因此*command*可以是一系列 Python 语句。其他 shell（例如 Windows shell）限制您为单行；*command*可以是一个或多个用分号（;）分隔的简单语句，正如我们在“语句”中讨论的那样。

另一种指定要运行的 Python 脚本的方法是使用**-m** *module*。此选项告诉 Python 从 Python 的 sys.path 中的某个目录加载和运行名为*module*的模块（或名为*module*的包或 ZIP 文件的*__main__.py*成员）；这对于使用 Python 标准库中的几个模块非常有用。例如，正如我们在“timeit 模块”中讨论的那样，**-m timeit**通常是执行 Python 语句的最佳方式。

连字符（**-**）或在此位置缺少任何令牌告诉解释器从标准输入读取程序源码，通常是交互式会话。只有在跟随进一步参数时才需要连字符。*args*是任意字符串；您运行的 Python 可以将这些字符串作为 sys.argv 列表的项访问。

例如，在命令提示符处输入以下内容即可使 Python 显示当前日期和时间：

```py
$ python -c "import time; print(time.asctime())"
```

如果 Python 可执行文件的目录在您的 PATH 环境变量中，您可以仅以**python**开头（无需指定完整路径）。（如果您安装了多个版本的 Python，您可以使用例如**python3**或**python3.10**指定版本；然后，如果您只说**python**，则使用的版本是您最近安装的版本。）

## Windows py 启动器

在 Windows 上，Python 提供了 **py** 启动器，用于在计算机上安装和运行多个 Python 版本。安装程序的底部有一个选项，用于为所有用户安装启动器（默认已选中）。当存在多个版本时，您可以使用 **py** 后跟版本选项选择特定版本，而不是简单的 **python** 命令。常见的 **py** 命令选项列在 表 2-2 中（使用 **py -h** 查看所有选项）。

表 2-2\. 经常使用的 py 命令行选项

| 选项 | 意义 |
| --- | --- |
| **-2** | 运行最新安装的 Python 2 版本。 |
| **-3** | 运行最新安装的 Python 3 版本。 |
| **-3.*****x*** 或 **-3.*****x*****-*****nn*** | 运行特定的 Python 3 版本。当仅引用为 **-3.10** 时，使用 64 位版本，如果没有 64 位版本则使用 32 位版本。 **-3.10-32** 或 **-3.10-64** 在两者都安装时选择特定的构建版本。 |
| **-0** 或 **--list** | 列出所有已安装的 Python 版本，包括标识是否为 32 位或 64 位的构建，如 **3.10-64**。 |
| **-h** | 列出所有 **py** 命令选项，后跟标准 Python 帮助。 |

如果未指定版本选项，**py** 将运行最新安装的 Python。

例如，要使用已安装的 Python 3.9 64 位版本显示本地时间，可以运行以下命令：

```py
C:\> py -3.9 -c "import time; print(time.asctime())"
```

（通常不需要指定 **py** 的路径，因为安装 Python 会将 **py** 添加到系统 PATH 中。）

## PyPy 解释器

*PyPy*，用 Python 编写，实现了自己的编译器以生成在 LLVM 后端运行的 LLVM 中间代码。PyPy 项目在性能和多线程方面比标准的 CPython 有一些改进。（截至本文写作时，PyPy 已更新至 Python 3.9。）

**pypy** 可以类似于 **python** 运行：

```py
[*path*]pypy {*options*} [-c *command* | *file* | - ] {*args*}
```

请查看 PyPy 的 [主页](http://pypy.org) 获取安装说明和完整的最新信息。

## 交互式会话

当你运行 **python** 而没有脚本参数时，Python 启动交互会话，并提示你输入 Python 语句或表达式。交互式会话对于探索、检查和使用 Python 作为强大、可扩展的交互式计算器非常有用。（本章末尾简要讨论的 Jupyter Notebook 就像专门用于交互式会话的“强化版 Python”。）这种模式通常称为 *REPL*，即读取-求值-打印循环，因为解释器基本上就是这样做的。

当您输入完整语句时，Python 执行它。当您输入完整表达式时，Python 评估它。如果表达式有结果，Python 输出表示结果的字符串，并将结果分配给名为 _（单个下划线）的变量，以便您可以立即在另一个表达式中使用该结果。当 Python 预期语句或表达式时，提示字符串为 >>>，当已开始但未完成语句或表达式时为 ...。特别地，在您在前一行打开括号、方括号或大括号但尚未关闭它时，Python 使用 ... 提示。

在交互式 Python 环境中工作时，您可以使用内置的 **help()** 函数进入一个帮助实用程序，提供关于 Python 关键字和运算符、安装的模块以及一般主题的有用信息。在浏览长帮助描述时，按 **q** 返回到 help> 提示符。要退出实用程序并返回到 Python >>> 提示符，请输入 **quit**。您还可以通过在 Python 提示符下输入 **help(***obj***)** 来获取有关特定对象的帮助，其中 *obj* 是您想要更多帮助的程序对象。

有几种方式可以结束交互会话。最常见的是：

+   输入您的操作系统的文件结尾按键（在 Windows 上为 Ctrl-Z，在类 Unix 系统上为 Ctrl-D）。

+   执行内置函数 quit 或 exit，使用形式 quit() 或 exit()。（省略尾随 () 将显示消息，如“使用 quit() 或 Ctrl-D（即 EOF）退出”，但仍会保留您在解释器中。）

+   执行语句 **raise** SystemExit，或调用 sys.exit()（我们在 第 6 章 中讨论 SystemExit 和 **raise**，以及在 第 8 章 中的 sys 模块）。

# 使用 Python 交互解释器进行实验。

在交互式解释器中尝试 Python 语句是快速实验 Python 并立即看到结果的一种方式。例如，这里是内置 enumerate 函数的简单使用：

```py
>>> print(list(enumerate("abc")))
```

```py
[(0, 'a'), (1, 'b'), (2, 'c')]
```

交互解释器是学习核心 Python 语法和特性的良好入门平台。（经验丰富的 Python 开发人员经常打开 Python 解释器来快速检查不经常使用的命令或函数。）

行编辑和历史记录功能部分依赖于 Python 的构建方式：如果包含了 readline 模块，则可使用 GNU readline 库的所有功能。Windows 对于像 **python** 这样的交互文本模式程序有一个简单但可用的历史记录功能。

除了内置的 Python 交互式环境和下一节介绍的更丰富的开发环境中提供的环境外，你可以自由下载其他强大的交互式环境。最流行的是[*IPython*](http://ipython.org)，在“IPython”中有详细介绍，提供了丰富的功能。一个更简单、更轻量级但同样非常方便的替代读取行解释器是[*bpython*](https://oreil.ly/UBZVL)。

# Python 开发环境

Python 解释器的内置交互模式是 Python 最简单的开发环境。它比较原始，但是轻量级，占用空间小，启动速度快。配合一个好的文本编辑器（如“带有 Python 支持的免费文本编辑器”中讨论的），以及行编辑和历史记录功能，交互式解释器（或者更强大的 IPython/Jupyter 命令行解释器）是一个可用的开发环境。但是，你还可以使用其他几种开发环境。

## IDLE

Python 的[集成开发与学习环境（IDLE）](https://oreil.ly/1vXr6)随着大多数平台上的标准 Python 发行版一起提供。IDLE 是一个跨平台的、100% 纯 Python 应用程序，基于 Tkinter GUI。它提供一个类似交互式 Python 解释器的 Python shell，但功能更丰富。还包括一个专为编辑 Python 源代码优化的文本编辑器、集成的交互式调试器以及几个专用的浏览器/查看器。

若要在 IDLE 中获得更多功能，请安装[IdleX](https://oreil.ly/cU_aD)，这是一个大量的免费第三方扩展集合。

要在 macOS 上安装并使用 IDLE，请按照 Python 网站上的具体[说明](https://oreil.ly/wHA6I)进行操作。

## 其他 Python IDE

IDLE 是成熟、稳定、易用、功能相当丰富且可扩展的。然而，还有许多其他 IDE：跨平台或特定于平台、免费或商业化（包括带有免费提供的商业 IDE，特别是如果你开发开源软件）、独立或作为其他 IDE 的附加组件。

其中一些 IDE 具有静态分析、GUI 构建器、调试器等功能。Python 的 IDE [wiki 页面](https://oreil.ly/EMpSD)列出了 30 多种，并指向许多其他 URL，包括评测和比较。如果你是 IDE 收集者，祝你好运！

即使是所有可用的 IDE 的一个小小子集，我们也无法完全公正地进行介绍。流行的跨平台、跨语言模块化 IDE [Eclipse](http://www.eclipse.org) 的免费第三方插件 [PyDev](http://www.pydev.org) 具有出色的 Python 支持。史蒂夫长期以来一直使用由 Archaeopteryx 推出的 [Wing](https://wingware.com)，这是最古老的 Python 专用 IDE。保罗的首选 IDE，也可能是当今最流行的第三方 Python IDE，是由 JetBrains 推出的 [PyCharm](https://oreil.ly/uQWxm)。[Thonny](https://thonny.org) 是一款流行的初学者 IDE，轻量但功能齐全，可以轻松安装在 Raspberry Pi（或几乎任何其他流行平台）上。还有不容忽视的是微软的 [Visual Studio Code](https://code.visualstudio.com)，这是一个非常出色且非常流行的跨平台 IDE，支持多种语言，包括 Python（通过插件）。如果您使用 Visual Studio，请查看 [PTVS](https://oreil.ly/VZ7Dl)，这是一个开源插件，特别擅长在需要时允许 Python 和 C 语言混合调试。

## 具有 Python 支持的免费文本编辑器

您可以使用任何文本编辑器编辑 Python 源代码，甚至是简单的，比如在 Windows 上的记事本或在 Linux 上的 *ed*。许多强大的免费编辑器支持 Python，并带有额外功能，如基于语法的着色和自动缩进。跨平台编辑器使您能够在不同平台上以统一的方式工作。优秀的文本编辑器还允许您在编辑器内运行您选择的工具对正在编辑的源代码进行操作。Python 编辑器的最新列表可以在 [PythonEditors wiki](https://oreil.ly/HGAzB) 上找到，其中列出了数十种编辑器。

就编辑能力而言，最出色的可能是经典的[Emacs](https://oreil.ly/MnEBy)（请参阅 Python [wiki 页面](https://oreil.ly/AIocZ)以获取特定于 Python 的附加组件）。Emacs 不易学习，也不是轻量级。³ Alex 的个人最爱⁴ 是另一个经典之作：[Vim](http://www.vim.org)，Bram Moolenaar 改进的传统 Unix 编辑器 vi 的版本。可以说它*几乎*不如 Emacs 强大，但仍然值得考虑——它快速、轻量级、支持 Python 编程，并在文本模式和 GUI 版本中均可运行。对于优秀的 Vim 覆盖范围，请参阅[*Learning the vi and Vim Editors*](https://www.oreilly.com/library/view/learning-the-vi/9781492078791/)，Arnold Robbins 和 Elbert Hannah 编著的第 8 版（O’Reilly）；参阅 Python [wiki 页面](https://oreil.ly/6pQ6t)以获取 Python 特定的技巧和附加组件。Steve 和 Anna 也使用 Vim，并且在可用时，Steve 还使用商业编辑器[Sublime Text](https://www.sublimetext.com)，具有良好的语法着色和足够的集成，可以从编辑器内部运行程序。对于快速编辑和执行短 Python 脚本（甚至对于多兆字节文本文件也是快速且轻量级的通用文本编辑器），Paul 选择[SciTE](https://scintilla.org/SciTE.xhtml)。

## Python 程序检查工具

Python 编译器足以检查程序语法以便运行程序或报告语法错误。如果希望更彻底地检查 Python 代码，可以下载并安装一个或多个第三方工具。[pyflakes](https://oreil.ly/RPeeJ) 是一个非常快速、轻量级的检查器：它不是很彻底，但它不会导入它正在检查的模块，这使得使用它快速且安全。在另一端，[pylint](https://www.pylint.org) 非常强大且高度可配置；它不是轻量级的，但通过可编辑的配置文件可以高度自定义地检查许多样式细节。⁵ [flake8](https://pypi.org/project/flake8) 将 pyflakes 与其他格式化程序和自定义插件捆绑在一起，通过在多个进程之间分配工作可以处理大型代码库。[black](https://pypi.org/project/black) 及其变体[blue](https://pypi.org/project/blue) 故意不太可配置；这使得它们在广泛分散的项目团队和开源项目中流行，以强制执行常见的 Python 风格。为了确保不会忘记运行它们，可以将一个或多个这些检查器/格式化程序整合到您的工作流程中，使用[pre-commit package](https://pypi.org/project/pre-commit)。

对于更彻底地检查 Python 代码的正确类型使用，请使用[mypy](http://mypy-lang.org)等工具；请参阅第五章了解更多相关内容。

# 运行 Python 程序

无论您使用什么工具来生成 Python 应用程序，您都可以将其视为一组 Python 源文件，这些文件是通常具有扩展名 *.py* 的普通文本文件。*脚本* 是可以直接运行的文件。*模块* 是可以导入的文件（详见第七章），为其他文件或交互式会话提供一些功能。Python 文件可以同时是*模块*（导入时提供功能）和*脚本*（可以直接运行）。一个有用且广泛使用的约定是，Python 文件如果主要用于导入为模块，在直接运行时应执行一些自测操作，详见“测试”。

Python 解释器会根据需要自动编译 Python 源文件。Python 会将编译后的字节码保存在模块源代码所在的子目录 *__pycache__* 中，并添加一个版本特定的扩展名来表示优化级别。

要避免将编译后的字节码保存到磁盘上，您可以使用选项 **-B** 运行 Python，当您从只读磁盘导入模块时可能会很方便。此外，当您直接运行脚本时，Python 不会保存脚本的编译后的字节码形式；相反，每次运行时都会重新编译脚本。Python 仅为您导入的模块保存字节码文件。每当必要时，例如编辑模块源代码时，它会自动重建每个模块的字节码文件。最终，您可以使用第二十四章中介绍的工具（在线版可参考[这里](https://oreil.ly/python-nutshell-24)）对 Python 模块进行打包部署。

您可以使用 Python 解释器或者一个 IDE 来运行 Python 代码。⁶ 通常，您通过运行顶层脚本开始执行。要运行一个脚本，请将其路径作为参数传递给 **python**，详见“python 程序”。根据您的操作系统，您可以直接从 shell 脚本或命令文件调用 **python**。在类 Unix 系统上，您可以通过设置文件的权限位 x 和 r，并以 *shebang* 行开头，例如以下行：

```py
#!/usr/bin/env python
```

或者其他以 #! 开头，后跟 Python 解释器程序路径的行，此时您可以选择性地添加一个选项单词，例如：

```py
#!/usr/bin/python -OB
```

在 Windows 上，你可以使用相同的 #! 行风格，符合[PEP 397](https://oreil.ly/lmMal)，指定特定版本的 Python，这样你的脚本可以在类 Unix 和 Windows 系统之间跨平台运行。你还可以通过双击图标等通常的 Windows 机制来运行 Python 脚本。当你通过双击脚本图标来运行 Python 脚本时，Windows 会在脚本终止后自动关闭与脚本关联的文本模式控制台。如果你希望控制台保持开放（以便用户可以在屏幕上看到脚本的输出），确保脚本不要过早终止。例如，在脚本的最后一个语句中使用：

```py
input('Press Enter to terminate')
```

当你从命令提示符运行脚本时，这是不必要的。

在 Windows 上，你还可以使用扩展名 *.pyw* 和解释器程序 *pythonw.exe* 替代 *.py* 和 *python.exe*。*w* 变体运行 Python 时没有文本模式控制台，因此没有标准输入和输出。这对依赖 GUI 或在后台静默运行的脚本非常有用。只有在程序完全调试完成后才使用它们，以便在开发过程中保留标准输出和错误信息以供信息、警告和错误消息使用。

使用其他语言编码的应用程序可能会嵌入 Python，以控制 Python 的执行以实现其自身的目的。我们在“嵌入 Python”中简要讨论了这一点，详见第二十五章（在线版见[此处](https://oreil.ly/python-nutshell-25)）。

# 在浏览器中运行 Python

同样存在在浏览器会话中运行 Python 代码的选项，可以在浏览器进程中或某些独立的基于服务器的组件中执行。PyScript 是前者的典范，而 Jupyter 则是后者。

## PyScript

最近 Python 在浏览器中的一个发展是由 Anaconda 发布的[PyScript](https://pyscript.net)。PyScript 建立在 Pyodide 之上，⁷使用 WebAssembly 在浏览器中启动一个完整的 Python 引擎。PyScript 引入了自定义 HTML 标签，因此你可以在不需要了解或使用 JavaScript 的情况下编写 Python 代码。使用这些标签，你可以创建一个静态 HTML 文件，其中包含 Python 代码，在远程浏览器中运行，无需安装额外的软件。

简单的 PyScript “Hello, World!” HTML 文件可能看起来像这样：

```py
<html>
<head>
    <link rel='stylesheet' 
 href='https://pyscript.net/releases/2022.06.1/pyscript.css' />
    <script defer 
 src='https://pyscript.net/releases/2022.06.1/pyscript.js'></script>
</head>
<body>
<py-script>
`import` time
print('Hello, World!')
print(f'The current local time is {time.asctime()}')
print(f'The current UTC time is {time.asctime(time.gmtime())}')
</py-script>
</body>
</html>
```

即使你的电脑上没有安装 Python，你也可以将这段代码保存为静态 HTML 文件并在客户端浏览器中成功运行。

# PyScript 即将迎来变化

在出版时，PyScript 仍处于早期开发阶段，因此这里显示的特定标签和 API 可能会随着软件包的进一步开发而发生变化。

获取更全面和最新的信息，请参阅[PyScript 网站](https://pyscript.net)。

## Jupyter

IPython 中交互式解释器的扩展（在“IPython”中涵盖）被 [Jupyter 项目](https://jupyter.org) 进一步扩展，这个项目最著名的是 Jupyter Notebook，它为 Python 开发者提供了一种 [“文学编程”](https://oreil.ly/yvn4z) 工具。一个笔记本服务器，通常通过网站访问，保存和加载每个笔记本，创建一个 Python 内核进程来交互地执行其 Python 命令。

笔记本是一个丰富的环境。每个笔记本都是一个单元格序列，其内容可以是代码或使用 Markdown 语言扩展的富文本格式，允许包含复杂的数学公式。代码单元格也可以产生丰富的输出，包括大多数流行的图像格式以及脚本化的 HTML。特殊的集成将 matplotlib 库适应到网络上，有越来越多的机制用于与笔记本代码进行交互。

更多的集成使得笔记本以其他方式出现成为可能。例如，通过适当的扩展，您可以轻松地将 Jupyter 笔记本格式化为 [reveal.js](https://revealjs.com) 幻灯片，用于交互式执行代码单元格的演示。[Jupyter Book](https://jupyterbook.org) 允许您将笔记本集合为章节并将其发布为书籍。GitHub 允许浏览（但不执行）上传的笔记本（一个特殊的渲染器提供正确的笔记本格式）。

互联网上有许多 Jupyter 笔记本的示例。要了解其功能的一个很好的演示，请查看 [Executable Books 网站](https://oreil.ly/Y2WS0)；笔记本支持其发布格式。

¹ 如果路径名包含空格，则可能需要使用引号—同样，这取决于您的操作系统。

² 这可能会影响解析文档字符串以进行有意义目的的代码；我们建议您避免编写此类代码。

³ 一个很好的入门地点是 [*Learning GNU Emacs*, 3rd edition](https://learning.oreilly.com/library/view/learning-gnu-emacs/0596006489)（O’Reilly）。

⁴ 不仅是“一个编辑器”，还是 Alex 最喜欢的“接近 IDE 的工具”！

⁵ pylint 还包括有用的 [pyreverse](https://oreil.ly/vSs_v) 实用工具，可以直接从您的 Python 代码自动生成 [UML](https://learning.oreilly.com/library/view/uml-2-0-in/0596007957) 类和包图。

⁶ 或在线：例如，Paul 维护了一个在线 Python 解释器的 [列表](https://oreil.ly/GVT93)。

⁷ 这是开源项目通过“站在巨人的肩膀上”获得的协同效应的一个很好的例子，这已经成为一种普遍的、日常的事情！
