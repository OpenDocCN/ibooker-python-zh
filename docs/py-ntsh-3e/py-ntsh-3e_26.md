# 第二十六章：从 v3.7 到 v3.n 的迁移

这本书跨越了几个版本的 Python，并涵盖了一些重要的（仍在发展中的！）新功能，包括：

+   保持顺序的字典

+   类型注解

+   := 赋值表达式（俗称“海象操作符”）

+   结构化模式匹配

个别开发者可能会在每个新的 Python 版本发布后进行安装，并在解决兼容性问题时逐步升级。但对于在企业环境中工作或维护共享库的 Python 开发者来说，从一个版本迁移到下一个版本需要深思熟虑和计划。

本章讨论了 Python 语言的变化，从 Python 程序员的视角来看。（Python 内部也有许多变化，包括 Python C API 的变化，但这些超出了本章的范围：详情请参见每个发布版本的在线文档中的“Python 3.*n* 新特性”部分。）

# Python 3.11 中的重大更改

大多数版本都有几个显著的新功能和改进，这些特性和改进可以作为选择特定版本的高层次原因。表 26-1 详细介绍了版本 3.6–3.11 的主要新功能和破坏性更改，这些更改可能影响许多 Python 程序；更完整的列表请参见附录。

表 26-1\. 近期 Python 发布的重大更改

| 版本 | 新功能 | 破坏性更改 |
| --- | --- | --- |
| 3.6 |

+   字典保留顺序（作为 CPython 的实现细节）

+   添加了 F-字符串

+   支持数字文字中的下划线

+   注解可以用于类型，可以通过外部工具如 mypy 进行检查

+   asyncio 不再是一个临时模块

    *初始发布时间：2016 年 12 月*

    *支持结束时间：2021 年 12 月*

|

+   不再支持在大多数 re 函数的模式参数中使用未知转义的 \ 和 ASCII 字母（仅在 re.sub() 中仍然允许）

|

| 3.7 |
| --- |

+   字典保留顺序（作为正式语言的保证）

+   添加了 dataclasses 模块

+   添加了 breakpoint() 函数

    *初始发布时间：2018 年 6 月*

    *计划支持结束时间：2023 年 6 月*

|

+   不再支持在 re.sub() 的模式参数中使用未知转义的 \ 和 ASCII 字母

+   不再支持在 bool()、float()、list() 和 tuple() 中使用命名参数

+   不再支持在 int() 中使用前置命名参数

|

| 3.8 |
| --- |

+   添加了赋值表达式（:=，也称为海象操作符）

+   在函数参数列表中使用 / 和 * 表示位置参数和命名参数

+   在 f-字符串中使用尾部 = 进行调试（f'{x=}' 的简写形式为 f'x={x!r}'）

+   添加了类型类（Literal, TypedDict, Final, Protocol）

    *初始发布时间：2019 年 10 月*

    *计划支持结束时间：2024 年 10 月*

|

+   移除了 time.clock()；使用 time.perf_counter()

+   移除了 pyvenv 脚本；使用 **python -m venv** 替代

+   **yield** 和 **yield from** 不再允许在推导式或生成器表达式中使用

+   添加了对**is**和**is not**对 str 和 int 字面值的语法警告

|

| 3.9 |
| --- |

+   字典上支持联合运算符&#124;和&#124;=

+   添加了 str.removeprefix()和 str.removesuffix()方法

+   添加了 zoneinfo 模块以支持 IANA 时区（替换第三方 pytz 模块）

+   类型提示现在可以在泛型中使用内置类型（例如 list[int]而不是 List[int]）

    *初始发布时间：2020 年 10 月*

    *预计支持结束时间：2025 年 10 月*

|

+   移除了 array.array.tostring()和 fromstring()方法

+   移除了 threading.Thread.isAlive()（请使用 is_alive()代替）

+   移除了 ElementTree 和 Element 的 getchildren()和 getiterator()方法

+   移除了 base64.encodestring()和 decodestring()（请改用 encodebytes()和 decodebytes()）

+   移除了 fractions.gcd()（请使用 math.gcd()代替）

+   移除了 typing.NamedTuple._fields（请改用 __annotations__ 代替）

|

| 3.10 |
| --- |

+   支持**match**/**case**结构模式匹配

+   允许将联合类型写为 X &#124; Y（在类型注解中和作为 isinstance()的第二个参数）

+   添加了 zip()内置函数的可选 strict 参数，以检测长度不同的序列

+   官方支持了带括号的上下文管理器；例如，**with**(*ctxmgr*, *ctxmgr*, ...)

    *初始发布时间：2021 年 10 月*

    *预计支持结束时间：2026 年 10 月*

|

+   从 collections 中移除了 ABCs 导入（现在必须从 collections.abc 导入）

+   大多数 asyncio 高级 API 中的循环参数已被移除

|

| 3.11 |
| --- |

+   改进了错误消息

+   总体性能提升

+   添加了异常组和 except*表达式

+   添加了类型提示类（Never，Self）

+   将 tomllib TOML 解析器添加到标准库

    *初始发布时间：2022 年 10 月*

    *预计支持结束时间：2027 年 10 月（估计）*

|

+   移除了 binhex 模块

+   将 int 转换为 str 的限制扩展到 4300 位数字

|

# 规划 Python 版本升级

首先为什么要升级？如果您有一个稳定运行的应用程序和一个稳定的部署环境，那么一个合理的决定可能是不做任何更改。但是版本升级确实带来了好处：

+   新版本通常引入新功能，这可能使您能够简化代码。

+   更新的版本包括错误修复和重构，可以提高系统的稳定性和性能。

+   旧版本中发现的安全漏洞可能在新版本中得到修复。²

最终，旧版 Python 版本将不再受支持，运行在旧版本上的项目将变得难以维护且成本更高。因此，升级可能成为必要。

## 选择目标版本

在决定迁移哪个版本之前，有时你必须首先弄清楚，“我现在正在运行哪个版本？”你可能会不愉快地发现，你公司系统中存在运行不受支持的 Python 版本的旧软件。通常情况是，这些系统依赖于某些第三方包，而这些包本身版本落后或没有可用的升级版本。当这种系统在公司运营中扮演重要角色时，情况会更为严峻。你可以通过远程访问 API 隔离落后的包，允许该包在旧版本上运行，同时让你自己的代码安全升级。必须向高级管理层展示存在升级约束的系统，以便他们了解保留、升级、隔离或替换的风险和权衡。

目标版本的选择通常默认为“最新版本”。这是一个合理的选择，因为在进行升级时，它通常是成本效益最高的选项：最新发布的版本将具有最长的支持期。更保守的立场可能是“最新版本减一”。你可以相当确信，版本 *N*–1 在其他公司进行了一段生产测试期间，并且其他人已经解决了大部分问题。

## 确定工作范围

在选择了 Python 的目标版本之后，识别从当前软件使用的版本到目标版本（包括目标版本）之间的所有突破性变化（请参阅附录中的详细功能和版本变化表；更多详细信息可以在[在线文档](https://oreil.ly/pvEtK)的“Python 3.*n* 新特性”部分找到）。通常会有适用于当前版本和目标版本的兼容形式文档化的突破性变化。记录并传达开发团队在升级之前需要进行的源代码更改。（如果您的代码受到大量突破性变化或与相关软件的兼容性问题的影响，直接升级到所选目标版本可能涉及比预期更多的工作。您甚至可能需要重新考虑目标版本的选择，或者考虑采取较小的步骤。也许您会决定首先升级到 *目标*–1，然后推迟升级到 *目标* 或 *目标*+1 的任务，作为后续升级项目。）

确定您的代码库使用的任何第三方或开源库，并确保它们与目标 Python 版本兼容（或计划与之兼容）。即使您自己的代码库已准备好升级到目标版本，落后的外部库可能会阻碍您的升级项目。必要时，您可以将这样的库隔离在单独的运行时环境中（使用虚拟机或容器技术），如果该库提供远程访问编程接口的话。

在开发环境中提供目标 Python 版本，并可选择在部署环境中提供，以便开发人员确认其升级更改是完整和正确的。

## 应用代码更改

一旦确定了目标版本并识别了所有破坏性更改，您将需要在代码库中进行更改，使其与目标版本兼容。理想情况下，您的目标是使代码以与当前版本和目标 Python 版本均兼容的形式存在。

# 导入自 __future__

__future__ 是一个标准库模块，包含各种功能，文档在 [在线文档](https://oreil.ly/3NaU5) 中，用于简化版本之间的迁移。它不同于任何其他模块，因为导入功能可能影响您程序的语法，而不仅仅是语义。这些导入必须是代码的初始可执行语句。

每个“未来特性”都使用以下语句激活：

```py
`from` __future__ `import` *`feature`*
```

其中 *feature* 是您想要使用的功能名称。

在本书涵盖的版本范围内，您可能考虑使用的唯一未来特性是：

```py
`from` __future__ `import` annotations
```

允许引用尚未定义的类型而无需将其括在引号中（如 第五章 所述）。如果您当前的版本是 Python 3.7 或更高版本，则添加此 __future__ 导入将允许在类型注释中使用未引用的类型，因此您以后无需重新执行它们。

首先检查在多个项目中共享的库。从这些库中移除阻碍性更改将是一个关键的第一步，因为在完成此步骤之前，您将无法在目标版本上部署任何依赖的应用程序。一旦库与两个版本兼容，它就可以用于迁移项目中。未来，库代码必须保持与当前 Python 版本和目标版本的兼容性：共享库可能是最后一个能够利用目标版本新功能的项目。

独立应用程序将有更早的机会使用目标版本中的新功能。一旦应用程序移除了所有受到破坏性更改影响的代码，请将其提交到您的源代码控制系统中作为跨版本兼容的快照。之后，您可以向应用程序代码添加新功能，并将其部署到支持目标版本的环境中。

如果版本兼容性变化影响到类型注释，你可以使用*.pyi*存根文件来隔离与版本相关的类型信息与源代码。

## 使用 pyupgrade 进行升级自动化

你可以使用自动化工具（如[pyupgrade 包](https://oreil.ly/01AKX)）来自动化大部分代码升级过程中的枯燥工作。pyupgrade 分析 Python 的 ast.parse 函数返回的抽象语法树（AST），以定位问题并对源代码进行修正。你可以通过命令行开关选择特定的目标 Python 版本。

每当你使用自动代码转换时，请审核转换过程的输出。像 Python 这样的动态语言使得完美的转换是不可能的；尽管测试有所帮助，但无法捕捉所有的不完美之处。

## 多版本测试

确保你的测试尽可能覆盖项目的大部分内容，以便在测试过程中可能会发现版本间错误。目标至少达到 80%的测试覆盖率；超过 90%可能难以达到，因此不要花费过多精力试图达到过高的标准。(*模拟*, 在“单元测试和系统测试”中提到，可以帮助你增加单元测试覆盖的广度，尽管深度不变。)

[tox 包](https://tox.readthedocs.io)对于帮助你管理和测试多版本代码非常有用。它允许你在多个不同的虚拟环境下测试你的代码，并支持多个 CPython 版本以及 PyPy。

## 使用受控部署流程

在部署环境中使目标 Python 版本可用，通过应用环境设置来指示应用程序是否应该使用当前或目标 Python 版本运行。持续跟踪并定期向管理团队报告完成百分比。

## 你应该多久进行一次升级？

PSF 按年度发布 Python 的小版本，每个版本发布后享有五年的支持期。如果采用最新版本减一策略，它为你提供了一个稳定、经过验证的版本来迁移，拥有四年的支持时间窗口（以防未来需要推迟升级）。考虑到四年的时间窗口，每一到两年升级到最新版本减一应该能在升级成本和平台稳定性之间提供合理的平衡。

# 总结

维护组织系统所依赖的软件版本更新是一种持续的良好“软件卫生”习惯，无论是 Python 还是其他任何开发堆栈都是如此。通过每次只升级一到两个版本的常规升级，你可以将这项工作保持在稳定和可管理的水平上，并且它将成为你组织中公认和重视的活动。

¹ 尽管 Python 3.6 超出了本书涵盖范围的版本，但它引入了一些重要的新特性，我们在此提及它以供历史背景参考。

² 当这种情况发生时，通常是“全体出动”的紧急情况，必须赶快进行升级。正是通过实施稳定持续的 Python 版本升级计划，您才能避免或至少减少这些事件。