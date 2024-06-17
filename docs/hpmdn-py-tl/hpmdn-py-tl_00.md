# 前言

本书是现代 Python 开发工具的指南——这些程序帮助你执行诸如：

+   管理你系统上的 Python 安装

+   安装当前项目的第三方包

+   构建一个 Python 包，用于在包仓库上分发

+   在多个环境中反复运行测试套件

+   对你的代码进行 linting 和类型检查以捕获 bug

你并不严格需要这些工具来编写 Python 软件。启动你系统的 Python 解释器，获取一个交互式提示符。将你的 Python 代码保存为脚本以供以后使用。为什么要使用编辑器和命令行之外的任何东西呢？

这不是一个修辞性问题。你将添加到开发工作流程中的每个工具都应该有一个明确的目的，并带来超过使用它的成本的好处。通常情况下，当你需要使开发能够持续 *长期* 时，开发工具的好处会显现出来。在某些时候，将你的模块发布到 Python 软件包索引上会比将其发送电子邮件给用户更容易。

在从编写一次性脚本到分发和维护包的旅程中，会遇到一些挑战：

+   在多个操作系统上支持多个 Python 版本

+   保持依赖项的最新状态，并扫描它们以发现漏洞

+   保持代码库的可读性和一致性

+   与社区中的 bug 报告和外部贡献进行交互

+   保持高测试覆盖率，以减少代码更改中的缺陷率

+   自动化重复任务，减少摩擦并避免意外

本书将向你展示开发工具如何帮助解决这些挑战。这里描述的工具极大地提升了 Python 项目的代码质量、安全性和可维护性。

但工具也增加了复杂性和开销。本书力求通过将工具组合成易于使用的工具链，并通过可靠和可重复的自动化工作流来最小化这些问题——无论是在开发者的本地机器上执行，还是在跨多个平台和环境的持续集成服务器上执行。尽可能地，你应该能够专注于编写软件，而你的工具链则在后台运行。

懒惰被称为“程序员的最大优点”，¹ 这句话也适用于开发工具：保持你的工作流程简单，不要为了工具而使用工具。与此同时，优秀的程序员也是好奇心重。尝试本书中的工具，看看它们能为你的项目带来什么价值。

# 谁应该阅读本书？

如果你是这些人之一，阅读本书将使你受益匪浅：

+   你精通 Python，但不确定如何创建一个包。

+   多年来，你一直在做这些事情——setuptools、virtualenv 和 pip 是你的朋友。你对工具链的最新发展很感兴趣，以及它们能为你的项目带来什么。

+   你维护在生产环境中运行的重要代码。但肯定有更好的方法来处理所有这些事情。你想了解最先进的工具和不断发展的最佳实践。

+   你希望作为 Python 开发者更加高效。

+   你是一名寻找稳健且现代的项目基础设施的开源维护者。

+   你在项目中使用了许多 Python 工具，但很难看到它们如何完美地配合在一起。你希望减少所有这些工具带来的摩擦。

+   “事情总是出问题 — 为什么 Python 现在找不到我的模块？我刚安装的包为什么无法导入？”

本书假定你具有基本的 Python 编程语言知识。你需要熟悉的唯一工具是 Python 解释器、编辑器或 IDE 以及操作系统的命令行。

# 本书大纲

本书分为三个部分：

第一部分，“使用 Python”

+   第一章，“安装 Python”，教你如何随时间管理不同平台上的 Python 安装。本章还介绍了 Windows 和 Unix 的 Python 启动器 — 你将在全书中使用它们。

+   第二章，“Python 环境”，深入讨论了 Python 安装，并讨论了你的代码如何与之交互。你还将了解帮助你有效使用虚拟环境的工具。

第二部分，“Python 项目”

+   第三章，“Python 包”，教你如何将项目设置为 Python 包，并如何构建和发布打包工件。本章还介绍了贯穿全书的示例应用程序。

+   第四章，“依赖管理”，讲述了如何将第三方包添加到 Python 项目中，以及如何随时间跟踪你的项目依赖关系。

+   第五章，“使用 Poetry 管理项目”，教你如何使用 Poetry 处理 Python 项目。Poetry 让你可以在更高的层次管理环境、依赖关系和打包。

第三部分，“测试与静态分析”

+   第六章，“使用 pytest 进行测试”，讨论如何测试 Python 项目，并有效地使用 pytest 框架及其生态系统。

+   第七章，“使用 Coverage.py 进行代码覆盖率测量”，教你如何通过测量测试套件的代码覆盖率来发现未经测试的代码。

+   第八章，“使用 Nox 进行自动化”，介绍了 Nox 自动化框架。你将使用它在 Python 环境中运行测试，并在项目中自动化检查和其他开发任务。

+   第九章，“使用 Ruff 和 pre-commit 进行检查”，展示了如何找到和修复可能的错误，并使用 Ruff 格式化代码。您还将学习有关 pre-commit 的知识，这是一个与 Git 集成的跨语言代码检查框架。

+   第十章，“使用类型进行安全和检查”，教您如何使用静态和运行时类型检查器验证类型安全性，并在运行时检查类型以执行真正的魔术（适用条款和条件）。

# 参考资料和进一步阅读

在查阅本书之外的第一手参考文档时，请访问每个工具的官方文档。此外，许多有趣的与包装相关的讨论都发生在[Python Discourse](https://discuss.python.org/)。包装类别的讨论经常是塑造 Python 包装和工具生态系统未来的地方，通过[Python Enhancement Proposal](https://peps.python.org/)（PEP）流程形成包装标准。最后，[Python Packaging Authority](https://pypa.io/)（PyPA）是一个维护 Python 包装中使用的核心软件项目的工作组。他们的网站跟踪当前活动的互操作性标准列表，管理 Python 包装。PyPA 还发布[Python Packaging User Guide](https://packaging.python.org/)。

# 本书使用的约定

本书使用以下印刷约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序列表，以及段落内用于引用程序元素（如变量或函数名、数据库、数据类型、环境变量、语句和关键字）。

**`等宽字体加粗`**

显示用户应直接输入的命令或其他文本。

*`等宽字体斜体`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

可以在[*https://oreil.ly/hmpt-code*](https://oreil.ly/hmpt-code)下载补充材料（代码示例、练习等）。

如果您有技术问题或使用代码示例遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

此书旨在帮助您完成工作任务。通常情况下，如果此书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需征得我们的许可。例如，编写一个使用此书中多个代码片段的程序不需要许可。售卖或分发 O’Reilly 书籍中的示例代码需要许可。引用本书并引述示例代码以回答问题不需要许可。将本书中大量示例代码整合到产品文档中需要许可。

我们感谢，但通常不需要署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Hypermodern Python Tooling* by Claudio Jolowicz (O’Reilly)。Copyright 2024 Claudio Jolowicz, 978-1-098-13958-2。”

如果您认为您对代码示例的使用超出了合理使用范围或上述许可，请随时联系我们：*permissions@oreilly.com*。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](https://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专长。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。更多信息，请访问[*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

有关此书的评论和问题，请联系出版商：

+   O’Reilly Media, Inc.

+   Gravenstein Highway North 1005

+   加利福尼亚州塞巴斯托波尔 95472

+   800-889-8969（美国或加拿大）

+   707-827-7019（国际或本地）

+   707-829-0104（传真）

+   *support@oreilly.com*

+   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

我们有这本书的网页，上面列出了勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/hypermodern-python-tooling*](https://oreil.ly/hypermodern-python-tooling)。

有关我们的书籍和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在 YouTube 上观看我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

这本书涵盖了许多开源 Python 项目。我非常感谢它们的作者和维护者，他们大多数在业余时间长达多年来致力于这些项目。特别是，我要感谢 PyPA 的无名英雄们，他们在打包标准上的工作使得生态系统能朝着更好的工具化方向发展。特别感谢 Thea Flowers 编写了 Nox 并建立了一个友好的社区。

在这本书之前，有《超现代 Python》文章系列。我要感谢 Brian Okken、Michael Kennedy 和 Paul Everitt 帮助传播这些内容，以及 Brian 给我鼓励将其改编成书的勇气。

我要感谢我的审阅者 Pat Viafore、Jürgen Gmach、Hynek Schlawack、William Jamir Silva、Ganesh Hark 和 Karandeep Johar，他们提供了深刻的见解和富有主见的反馈。没有他们，这本书将不会如此。我对任何剩余错误负责。

制作一本书需要整个村庄的帮助。我要感谢我的编辑 Zan McQuade、Brian Guerin、Sarah Grey 和 Greg Hyman，以及 O’Reilly 的整个团队。特别感谢 Sarah 在这段旅程中帮助我保持方向并改进我的写作。我要感谢 Cloudflare 的经理 Jakub Borys 给予我时间来完成这本书。

这本书献给我生命中的爱人 Marianna。没有她的支持、鼓励和灵感，我不可能完成这本书。

¹ Larry Wall, [*Programming Perl*](https://learning.oreilly.com/library/view/programming-perl-4th/9781449321451/), (Sebastopol: O’Reilly, 1991).
