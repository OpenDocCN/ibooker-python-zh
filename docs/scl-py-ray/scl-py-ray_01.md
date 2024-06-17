# 序言

我们为那些希望在 Python 中构建和扩展应用程序而不成为系统管理员的开发人员和数据科学家编写了本书。我们期望本书对于那些处理从单线程解决方案到多线程解决方案，再到分布式计算的问题的复杂性和规模不断增长的个人和团队最为有益。

虽然你可以在 Java 中使用 Ray，但本书使用 Python，并假设你对 Python 生态系统有一般的了解。如果你对 Python 不熟悉，优秀的 O’Reilly 书籍包括[《学习 Python》](https://oreil.ly/uPun0)（Mark Lutz 著）和[《Python 数据分析》](https://oreil.ly/F1xgP)（Wes McKinney 著）。

*无服务器* 是一个有点炒作的词，尽管它的名字是这样，但无服务器模型确实涉及相当多的服务器，但这个想法是你不必显式地管理它们。对于许多开发人员和数据科学家来说，不用担心服务器的细节就能让事情神奇地扩展的承诺是相当诱人的。另一方面，如果你喜欢深入研究你的服务器、部署机制和负载均衡器，那么这可能不适合你——但希望你会向同事推荐本书。

# 你将学到什么

在阅读本书时，您将学习如何利用您现有的 Python 技能使程序扩展到超越单个计算机的规模。您将学习有关分布式计算的技术，从远程过程调用到 actors，一直到分布式数据集和机器学习。我们在附录 A 中以一个“真实的”示例结束了本书，该示例使用了许多这些技术来构建可扩展的后端，并与基于 Python 的 Web 应用程序集成，并在 Kubernetes 上部署。

# 关于责任的说明

俗话说，大权必责。Ray 及其类似工具使您能够构建处理更多数据和用户的更复杂系统。重要的是不要过于兴奋和沉迷于解决问题，因为它们很有趣，要停下来问问自己的决定会带来什么影响。

寻找关于善意的工程师和数据科学家意外构建导致灾难性影响的模型或工具的故事并不难，比如破坏了新的美国退伍军人事务部支付系统，或者歧视性别的招聘算法。我们要求你在使用你的新发现的力量时牢记这一点，因为谁也不想因为错误的原因而进入教科书。

# 本书中使用的约定

本书中使用以下印刷约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`等宽字体`

用于程序清单，以及在段落内用于指代程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

*`等宽斜体`*

显示应由用户提供的值或由上下文确定的值。

###### 提示

此元素表示一个提示或建议。

###### 注意

此元素表示一般说明。

###### 警告

此元素指示警告或注意事项。

# 许可

一旦在印刷版中发布，不包括 O’Reilly 的独特设计元素（例如封面艺术、设计格式、“外观和感觉”）或 O’Reilly 的商标、服务标记和商业名称，本书可根据 [知识共享署名-非商业性使用-禁止演绎 4.0 国际公共许可证](https://oreil.ly/z976G) 使用。我们感谢 O’Reilly 允许我们在 Creative Commons 许可下提供本书。我们希望您选择通过公司费用账户购买数本此书（它是即将到来的任何假期的极好礼物）以支持本书（及其作者）。

# 使用代码示例

[使用 Ray 扩展 Python 机器学习 GitHub 代码库](https://oreil.ly/scaling-python-with-ray-code)包含本书大部分示例。本书中大多数示例位于 *ray_examples* 目录中。与 Dask on Ray 相关的示例位于 *dask* 目录中，而使用 Spark on Ray 的示例位于 *spark* 目录中。

如果您有技术问题或使用代码示例时遇到问题，请发送电子邮件至 *bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般来说，如果本书提供示例代码，则可以在您的程序和文档中使用它。除非您复制了大量代码，否则无需征得我们的许可。例如，编写使用本书中几个代码块的程序不需要许可。出售或分发来自 O’Reilly 书籍的示例需要许可。引用本书并引用示例代码回答问题不需要许可。将本书中大量示例代码整合到您产品的文档中需要许可。

我们感谢，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*使用 Ray 扩展 Python* 由 Holden Karau 和 Boris Lublinsky（O’Reilly）编写。版权所有 2023 Holden Karau 和 Boris Lublinsky，978-1-098-11880-8。”

如果您认为您使用的代码示例超出了公平使用或上述许可的范围，请随时通过 *permissions@oreilly.com* 联系我们。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly Media*](https://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频。欲了解更多信息，请访问：[*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为这本书创建了一个网页，列出勘误、示例和任何额外信息。你可以访问这个页面：[*https://oreil.ly/scaling-python-ray*](https://oreil.ly/scaling-python-ray)。

发送邮件至*bookquestions@oreilly.com* 对本书发表评论或提出技术问题。

要获取关于我们的书籍和课程的新闻和信息，请访问：[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)。

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)。

观看我们的 YouTube 频道：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)。

# 致谢

我们要感谢 Carlos Andrade Costa 的贡献，他与我们共同撰写了第八章。如果没有构建在社区基础上，本书将不会存在。感谢 Ray/Berkeley 社区和 PyData 社区。感谢所有早期读者和评论者对你们的贡献和指导。这些评论者包括 Dean Wampler、Jonathan Dinu、Adam Breindel、Bill Chambers、Trevor Grant、Ruben Berenguel、Michael Behrendt 等等。特别感谢 Ann Spencer 对最终成为这本书和[*使用 Dask 扩展 Python*](https://oreil.ly/fm857)（O’Reilly）的早期提案进行审查。特别感谢 O’Reilly 编辑和制作团队，尤其是 Virginia Wilson 和 Gregory Hyman，帮助我们整理文章并不知疲倦地与我们合作，以尽量减少错误、错别字等。任何剩余的错误都是作者的责任，有时违背了评论者和编辑的建议。

## 作者 Holden

我还要感谢我的妻子和合作伙伴们，他们忍受了我长时间泡在浴缸里写作的时光。特别感谢 Timbit 守卫家园，通常让我有理由早点起床（尽管我常常觉得时间太早）。

## 作者 Boris

我还要感谢我的妻子玛丽娜，她忍受了我长时间的写作会议，有时候会忽视她几个小时，以及我在 IBM 的同事们，他们进行了许多富有成效的讨论，帮助我更好地理解 Ray 的力量。
