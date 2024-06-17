# 序言

这是对 FastAPI——一个现代 Python Web 框架的务实介绍。这也是一个关于我们偶尔会碰到的闪亮新物体如何变得非常有用的故事。当你遇到狼人时，一发银弹可谓非常实用。（而且你将在本书后面遇到狼人。）

我从 70 年代中期开始编写科学应用程序。在 1977 年我第一次在 PDP-11 上遇到 Unix 和 C 之后，我有一种这个 Unix 东西可能会流行起来的感觉。

在 80 年代和 90 年代初期，互联网虽然还没有商业化，但已经是一个免费软件和技术信息的良好来源。当名为 Mosaic 的网络浏览器在 1993 年在初生的开放互联网上发布时，我有一种这个网络东西可能会流行起来的感觉。

几年后我创立了自己的 Web 开发公司时，我的工具还是当时的惯用选择：PHP、HTML 和 Perl。几年后的一个合同工作中，我最终尝试了 Python，并对我能够多快地访问、操作和显示数据感到惊讶。在两周的空闲时间内，我成功复制了一个四名开发者耗时一年才编写完整的 C 应用程序的大部分功能。现在我有一种这个 Python 东西可能会流行起来的感觉。

此后，我的大部分工作涉及 Python 及其 Web 框架，主要是 Flask 和 Django。我特别喜欢 Flask 的简单性，并在许多工作中更倾向于使用它。但就在几年前，我在灌木丛中发现了一抹闪光：一款名为 FastAPI 的新 Python Web 框架，由 Sebastián Ramírez 编写。

在阅读他（优秀）的[文档](https://fastapi.tiangolo.com)时，我对设计和所投入的思考深感印象深刻。特别是他的[历史](https://oreil.ly/Ds-xM)页面展示了他在评估替代方案时所付出的努力。这不是一个自我项目或有趣的实验，而是一个面向现实开发的严肃框架。现在我有一种这个 FastAPI 东西可能会流行起来的感觉。

我使用 FastAPI 编写了一个生物医学 API 站点，效果非常好，以至于我们团队在接下来的一年中用 FastAPI 重写了我们旧的核心 API。这个系统仍在运行中，并表现良好。我们的团队学习了本书中你将阅读到的基础知识，所有人都觉得我们正在写出更好的代码，速度更快，bug 更少。顺便说一句，我们中的一些人以前并没有用过 Python，只有我用过 FastAPI。

所以当我有机会向 O'Reilly 建议续集《介绍 Python》时，FastAPI 是我的首选。在我看来，FastAPI 至少会像 Flask 和 Django 一样有影响力，甚至更大。

正如我之前提到的，FastAPI 网站本身提供了世界级的文档，包括许多关于常见 Web 主题的细节：数据库、身份验证、部署等等。那么为什么要写一本书呢？

这本书并不打算全面无遗漏，因为那样会令人筋疲力尽。它的目的是实用——帮助你快速掌握 FastAPI 的主要思想并应用它们。我将指出各种需要一些侦查的技术，并提供日常最佳实践建议。

每一章都以预览内容开始。接下来，我尽量不忘记刚承诺的内容，提供细节和偶尔的旁白。最后，有一个简短易消化的复习。

俗话说，“这些是我事实依据的观点。”您的经验将是独特的，但我希望您能在这里找到足够有价值的内容，以成为更高效的 Web 开发者。

# 本书中使用的约定

本书中使用了以下排版约定：

*Italic*

指示新术语、网址、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序列表，以及段落内引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应按照字面意义输入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供值或由上下文确定值的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般提示。

# 使用代码示例

补充材料（代码示例、练习等）可在[*https://github.com/madscheme/fastapi*](https://github.com/madscheme/fastapi)下载。

如果您在使用示例代码时遇到技术问题或困难，请发送电子邮件至*support@oreilly.com*。

本书旨在帮助您完成工作。一般来说，如果本书提供了示例代码，您可以在自己的程序和文档中使用它。除非您要复制大部分代码，否则无需联系我们以获取许可。例如，编写一个使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 书籍中的示例需要许可。引用本书并引用示例代码来回答问题不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们感谢，但通常不要求归属。归属通常包括标题、作者、出版商和 ISBN 号码。例如：“*FastAPI* by Bill Lubanovic (O’Reilly)。2024 年版权归 Bill Lubanovic 所有，978-1-098-13550-8。”

如果您认为您使用的示例代码超出了合理使用范围或上述许可，请随时联系我们*permissions@oreilly.com*。

# O’Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](https://oreilly.com)提供技术和商业培训、知识和洞察，帮助公司取得成功。

我们独特的专家和创新者网络通过图书、文章和我们的在线学习平台分享他们的知识和专长。奥莱利的在线学习平台为您提供了按需访问实时培训课程、深入学习路径、交互式编码环境以及奥莱利和其他 200 多家出版商的大量文本和视频的机会。有关更多信息，请访问[*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

有关本书的评论和问题，请联系出版商：

+   奥莱利媒体公司

+   北格拉文斯坦高速公路 1005 号

+   加州塞巴斯托波尔市 95472

+   800-889-8969（美国或加拿大）

+   707-829-7019（国际或本地）

+   707-829-0104（传真）

+   *support@oreilly.com*

+   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

我们有一个为本书准备的网页，我们在其中列出勘误、示例和任何其他信息。您可以在[*https://oreil.ly/FastAPI*](https://oreil.ly/FastAPI)访问此页面。

有关我们的图书和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)。

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)。

在 YouTube 上观看我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)。

# 致谢

感谢许多人，在许多地方，从中我学到了很多：

+   艾拉高中

+   匹兹堡大学

+   生物钟实验室，

    明尼苏达大学

+   Intran

+   Crosfield-Dicomed

+   西北航空公司

+   Tela

+   WAM!NET

+   疯狂计划

+   SSESCO

+   Intradyn

+   Keep

+   汤姆逊路透社

+   Cray

+   企鹅计算

+   互联网档案馆

+   CrowdStrike

+   Flywheel
