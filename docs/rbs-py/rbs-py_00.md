# 序言

著名软件工程师和企业家马克·安德森（Marc Andreesen）曾经宣称[“软件正在吞噬世界”](https://oreil.ly/tYaNz)。这是在 2011 年说的，而随着时间的推移，这一说法变得更加真实。软件系统不断变得复杂，并且可以在现代生活的各个方面找到它们的身影。站在这个贪婪野兽的中心是 Python 语言。程序员经常把 Python 作为最喜欢的[语言](https://oreil.ly/RUNNh)，它无处不在：从网页应用到机器学习，再到开发工具，等等。

然而，并非所有闪闪发光的东西都是黄金。随着我们的软件系统变得越来越复杂，理解我们的心智模型如何映射到现实世界变得更加困难。如果不加控制，软件系统会变得臃肿且脆弱，赢得了可怕的“遗留代码”绰号。这些代码库通常伴随着诸如“不要触碰这些文件；我们不知道为什么，但你一旦触碰就会出问题”的警告，以及“哦，只有某某知道那段代码，而他们两年前就去了硅谷高薪工作”。软件开发是一个年轻的领域，但这类说法应该让开发人员和商业人士同样感到恐惧。

事实是，要编写持久的系统，你需要在做出选择时深思熟虑。正如 Titus Winters、Tom Manshreck 和 Hyrum Wright 所述，“软件工程是随时间整合的编程”。¹你的代码可能会持续很长时间——我曾经参与过那些在我上小学时就已经写好代码的项目。你的代码能持续多久？它会比你在当前工作岗位上的任期更长（或者你完成维护该项目的时间）吗？几年后当有人从中构建核心组件时，你希望你的代码如何被接纳？你希望你的继任者因为你的远见而感激你，还是因为你为这个世界带来的复杂性而诅咒你的名字？

Python 是一种很棒的语言，但有时在为未来构建时可能会变得棘手。其他编程语言的支持者曾经抨击 Python 为“不适合生产环境”或“仅适用于原型设计”，但事实是许多开发人员只是浅尝辄止，而没有学习编写健壮 Python 所需的所有工具和技巧。在本书中，你将学习如何做得更好。你将穿越多种方式使 Python 代码更加清晰和可维护。你未来的维护者将喜欢与你的代码一起工作，因为它从一开始就被设计成使事情变得简单。因此，去吧，阅读这本书，展望未来，构建持久的、令人敬畏的软件。

# 谁应该阅读这本书

这本书适用于任何希望以可持续和可维护的方式扩展其工作代码的 Python 开发人员。这并不是您的第一本 Python 教材；我预期您以前已经写过 Python。您应该对 Python 控制流感到舒适，并且以前已经使用过类。如果您正在寻找更入门的教材，我建议您先阅读 [*Learning Python*](https://oreil.ly/iIl2K)，作者是 Mark Lutz（O’Reilly）。

尽管我将涵盖许多高级 Python 主题，但这本书的目标并不是教您如何使用 Python 的所有功能。相反，这些功能是更大对话的背景，讨论稳健性以及您的选择如何影响可维护性。有时我会讨论您几乎不应该或者根本不应该使用的策略。这是因为我想说明稳健性的第一原则；在代码中理解我们为什么以及如何做出决策的旅程比在最佳场景中使用什么工具更重要。在实践中，最佳场景是罕见的。使用本书中的原则从您的代码库中得出您自己的结论。

这本书不是一本参考书。你可以称之为讨论书。每一章都应该是您组织中的开发人员讨论如何最好地应用这些原则的起点。开始一个书籍俱乐部、讨论组或午餐学习，以促进沟通。我在每一章中提出了讨论主题，以启动对话。当您遇到这些主题时，我鼓励您停下来反思您当前的代码库。与您的同行交谈，并利用这些主题作为讨论您的代码状态、流程和工作流的跳板。如果您对 Python 语言的参考书感兴趣，我衷心推荐 [*Fluent Python*](https://oreil.ly/PVbON)，作者是 Luciano Ramalho（O’Reilly；第二版预计将于 2021 年底出版）。

系统可以通过多种方式保持稳健。它可以经过安全强化、可伸缩、容错或者不太可能引入新错误来实现。每一种稳健性的方面都值得一本完整的书籍；这本书专注于防止继承您代码的开发人员在您的系统中引入新故障。我将向您展示如何与未来的开发人员沟通，如何通过架构模式使他们的生活更轻松，并且如何在代码库中捕捉错误，以避免它们进入生产环境。这本书关注的是您的 Python 代码库的稳健性，而不是整个系统的稳健性。

我将涵盖丰富的信息，涵盖软件工程、计算机科学、测试、函数式编程和面向对象编程（OOP）等多个软件领域。我不希望您具有这些领域的背景。有些部分我会以初学者的水平来解释；这通常是为了分解我们如何思考语言核心基础的方式。总体而言，这是一本中级水平的文本。

理想的读者包括：

+   目前在大型代码库工作的开发者们，希望找到更好的方法与他们的同事沟通

+   主要的代码库维护者，寻找方法来帮助减轻未来维护者的负担

+   自学成才的开发者们能够很好地编写 Python，但需要更好地理解我们为什么要做我们所做的事情

+   需要提醒开发实践建议的软件工程毕业生

+   寻找将他们的设计原理与健壮性的第一原则联系起来的高级开发者

本书侧重于随时间编写软件。如果您的大部分代码是原型、一次性或以其他方式可丢弃的，那么本书中的建议将导致比项目需要的更多的工作。同样，如果您的项目很小——比如 Python 代码少于一百行——那么使代码可维护确实增加了复杂性；毫无疑问。但是，我将指导您通过最小化这种复杂性。如果您的代码存在时间超过几周或增长到相当大的规模，则需要考虑代码库的可持续性。

# 关于本书

本书涵盖广泛的知识，分布在多个章节中。它分为四个部分：

第一部分，*用类型为您的代码添加注释*

我们将从 Python 的类型开始。类型对语言非常重要，但通常不会被深入研究。您选择的类型很重要，因为它们传达了非常具体的意图。我们将研究类型注释及其向开发人员传达的具体注释。我们还将讨论类型检查器及其如何帮助及早捕捉错误。

第二部分，*定义您自己的类型*

在讨论如何思考 Python 的类型后，我们将专注于如何创建自己的类型。我们将深入讲解枚举、数据类和类。我们将探索在设计类型时做出某些设计选择如何增加或减少代码的健壮性。

第三部分，*可扩展的 Python*

在学习如何更好地表达您的意图后，我们将专注于如何使开发人员轻松修改您的代码，有信心地在坚实的基础上构建。我们将涵盖可扩展性、依赖关系和允许您在最小影响下修改系统的架构模式。

第四部分，*构建一个安全网*

最后，我们将探讨如何建立一个安全网，这样你可以在他们摔倒时轻轻接住未来的合作者。他们的信心会增强，因为他们有一个强大而健壮的系统，可以毫不畏惧地适应他们的使用案例。最后，我们将介绍多种静态分析和测试工具，帮助你捕捉异常行为。

每章基本上是自包含的，涉及其他章节的引用会在适用时提及。你可以从头到尾阅读本书，也可以跳到你感兴趣的章节。每个部分中的章节相互关联，但书的各个部分之间的关系较少。

所有代码示例都是在 Python 3.9.0 上运行的，我会尽量指出你需要特定的 Python 版本或更高版本来运行示例（例如 Python 3.7 用于使用数据类）。

本书中，我将大部分工作都在命令行上进行。我在一个 Ubuntu 操作系统上运行了所有这些命令，但大多数工具在 Mac 或 Windows 系统上也同样适用。在某些情况下，我会展示某些工具如何与集成开发环境（IDE）如 Visual Studio Code（VS Code）互动。大多数 IDE 在幕后使用命令行选项；你学到的大部分命令行内容都可以直接转化为 IDE 选项。

本书将介绍许多不同的技术，可以提高你代码的健壮性。然而，在软件开发中并不存在万能药。在坚实的工程中，权衡是核心，我介绍的方法也不例外。在讨论这些主题时，我将公开透明地讨论它们的利弊。你对自己的系统了解更多，你最适合选择哪种工具来完成哪项工作。我所做的就是为你的工具箱添砖加瓦。

# 本书中使用的约定

本书中使用了以下印刷约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序清单，以及在段落中引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`等宽字体粗体`**

显示用户应按字面输入的命令或其他文本。

*`等宽字体斜体`*

显示应由用户提供值或由上下文确定值的文本。

###### 提示

这个元素表示一个提示或建议。

###### 注意

这个元素表示一个一般注释。

###### 警告

这个元素表示一个警告或注意。

# 使用代码示例

补充材料（代码示例、练习等）可以在[*https://github.com/pviafore/RobustPython*](https://github.com/pviafore/RobustPython)上下载。

如果你有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。通常情况下，如果本书提供示例代码，您可以在程序和文档中使用它。除非您复制了代码的大部分内容，否则无需征得我们的许可。例如，编写使用本书中几个代码块的程序不需要许可。销售或分发 O’Reilly 书籍中的示例需要许可。引用本书并引用示例代码回答问题不需要许可。将本书中大量示例代码整合到您产品的文档中需要许可。

我们感激，但通常不要求署名。署名通常包括标题，作者，出版商和 ISBN。例如：“*Robust Python* by Patrick Viafore（O’Reilly）。版权所有 2021 年 Kudzera，LLC，978-1-098-10066-7。”

如果您认为您对示例代码的使用超出了合理使用范围或上述许可，欢迎通过邮件联系我们*permissions@oreilly.com*。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和商业培训，知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍，文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问直播培训课程，深度学习路径，交互式编码环境以及来自 O’Reilly 和其他两百多家出版商的广泛的文本和视频集合。欲了解更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media，Inc.

+   1005 Gravenstein Highway North

+   加利福尼亚州塞巴斯托波尔 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设立了一个网页，列出勘误表，示例和任何额外信息。您可以访问[*https://oreil.ly/robust-python*](https://oreil.ly/robust-python)。

通过邮件*bookquestions@oreilly.com*发表评论或提出关于本书的技术问题。

关于我们的书籍和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

关注我们的 Twitter：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://youtube.com/oreillymedia*](http://youtube.com/oreillymedia)

# 致谢

我要感谢我的不可思议的妻子，肯德尔。她是我的支持和听众，我感谢她为确保我有时间和空间来写作本书所做的一切。

没有一本书是孤立写成的，这本书也不例外。我站在软件行业的巨人们的肩膀上，并感谢那些在我之前的人。

我还想感谢所有参与审阅本书的人，确保我的信息传递一致，示例清晰。感谢 Bruce G.、David K.、David P. 和 Don P. 提供的早期反馈，并帮助我决定书籍的方向。感谢我的技术审阅者 Charles Givre、Drew Winstel、Jennifer Wilcox、Jordan Goldmeier、Nathan Stocks 和 Jess Males，他们的宝贵反馈对我帮助很大，特别是那些只在我的脑海中有意义而不在纸上显现的地方。最后，感谢所有阅读早期发布草稿并友善地通过电子邮件分享他们想法的人，特别是 Daniel C. 和 Francesco。

我要感谢所有帮助将我的最终草稿转变为值得投入生产的作品的人。感谢 Justin Billing 作为副本编辑深入探讨，并帮助我完善想法的表达。感谢 Shannon Turlington 进行校对；书籍因为你而更加精致。特别感谢 Ellen Troutman-Zaig 制作了一本让我感到惊叹的索引。

最后，没有 O'Reilly 的出色团队，我做不到这一切。感谢 Amanda Quinn 在提案过程中帮助我，并帮助我聚焦书籍内容。感谢 Kristen Brown 让我在制作阶段感觉异常轻松。感谢 Kate Dullea，她将我的 MS Paint 质量的草图转化为干净、清晰的插图。此外，我想特别感谢我的发展编辑 Sarah Grey。我期待我们每周的会议，她在帮助我打造一个面向广泛读者群体的书籍时表现出色，同时还让我能够深入技术细节。

¹ Titus Winters、Tom Manshreck 和 Hyrum Wright。*Google 软件工程：编程经验教训*。Sebastopol, CA: O’Reilly Media, Inc.，2020 年。
