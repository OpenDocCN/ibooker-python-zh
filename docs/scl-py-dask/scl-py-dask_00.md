# 前言

我们为熟悉 Python 和 pandas 的数据科学家和数据工程师编写了这本书，他们希望处理比当前工具允许的更大规模的问题。目前的 PySpark 用户会发现，这些材料有些与他们对 PySpark 的现有知识重叠，但我们希望它们仍然有所帮助，并不仅仅是为了远离 Java 虚拟机（JVM）。

如果您对 Python 不太熟悉，一些优秀的 O’Reilly 书籍包括[*学习 Python*](https://learning.oreilly.com/library/view/learning-python-5th/9781449355722)和[*Python 数据分析*](https://learning.oreilly.com/library/view/python-for-data/9781098104023)。如果您和您的团队更频繁地使用 JVM 语言（如 Java 或 Scala），虽然我们有些偏见，但我们鼓励您同时查看 Apache Spark 以及[*学习 Spark*](https://learning.oreilly.com/library/view/learning-spark-2nd/9781492050032)（O’Reilly）和[*高性能 Spark*](https://learning.oreilly.com/library/view/high-performance-spark/9781098145842)（O’Reilly）。

本书主要集中在数据科学及相关任务上，因为在我们看来，这是 Dask 最擅长的领域。如果您有一个更一般的问题，Dask 似乎并不是最合适的解决方案，我们（再次有点偏见地）建议您查看[*使用 Ray 扩展 Python*](https://learning.oreilly.com/library/view/scaling-python-with/9781098118792)（O’Reilly），这本书的数据科学内容较少。

# 关于责任的说明

正如俗语所说，能力越大责任越大。像 Dask 这样的工具使您能够处理更多数据并构建更复杂的模型。重要的是不要因为简单而收集数据，并停下来问问自己，将新字段包含在模型中可能会带来一些意想不到的现实影响。您不必费力去寻找那些好心的工程师和数据科学家不小心建立了具有破坏性影响的模型或工具的故事，比如增加对少数族裔的审计、基于性别的歧视或像[词嵌入](https://oreil.ly/tqjth)（将单词的含义表示为向量的一种方法）中的偏见等更微妙的事情。请在使用您新获得的这些潜力时牢记这些潜在后果，因为永远不要因错误原因出现在教科书中。

# 本书中使用的约定

本书中使用了以下排版约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`固定宽度`

用于程序清单，以及段落内引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

###### 提示

此元素表示提示或建议。

###### 注释

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 在线图表

印刷版读者可以在 [*https://oreil.ly/SPWD-figures*](https://oreil.ly/SPWD-figures) 找到一些图表的更大、彩色版本。每个图表的链接也出现在它们的标题中。

# 许可证

一旦在印刷版发布，并且不包括 O’Reilly 独特的设计元素（即封面艺术、设计格式、“外观和感觉”）或 O’Reilly 的商标、服务标记和商业名称，本书在 Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International Public License 下可用。我们希望感谢 O’Reilly 允许我们在 Creative Commons 许可下提供本书，并希望您选择通过购买几本书（无论哪个假期季节即将来临）来支持本书（和我们）。

# 使用代码示例

[Scaling Python Machine Learning GitHub 仓库](https://oreil.ly/scaling-python-dask-code) 包含本书大部分示例。它们主要位于 *dask* 目录下，更奥义的部分（如跨平台 CUDA 容器）位于单独的顶级目录中。

如果您有技术问题或使用代码示例时遇到问题，请发送电子邮件至 *support@oreilly.com*。

这本书旨在帮助你完成工作任务。一般来说，如果本书提供了示例代码，你可以在你的程序和文档中使用它。除非你在复制大部分代码，否则无需联系我们获得许可。例如，编写一个使用本书多个代码片段的程序不需要许可。出售或分发 O'Reilly 书籍的示例需要许可。引用本书并引用示例代码回答问题不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们欢迎，但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Scaling Python with Dask* by Holden Karau and Mika Kimmins (O’Reilly). Copyright 2023 Holden Karau and Mika Kimmins, 978-1-098-11987-4.”

如果您认为您对代码示例的使用超出了合理使用或上述许可，请随时通过 *permissions@oreilly.com* 联系我们。

# O'Reilly 在线学习

###### 注意

40 多年来，[*O’Reilly Media*](https://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多家出版商的广泛文本和视频集合。有关更多信息，请访问 [*https://oreilly.com*](https://oreilly.com)。

# 如何联系我们

请将关于本书的评论和问题发送至出版社：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-889-8969（美国或加拿大）

+   707-829-7019（国际或本地）

+   707-829-0104（传真）

+   *support@oreilly.com*

+   [*https://www.oreilly.com/about/contact.html*](https://www.oreilly.com/about/contact.html)

我们为这本书建立了一个网页，列出了勘误、示例和任何额外信息。您可以访问[*https://oreil.ly/scaling-python-dask*](https://oreil.ly/scaling-python-dask)。

欲了解有关我们的书籍和课程的新闻和信息，请访问[*https://oreilly.com*](https://oreilly.com)。

在 LinkedIn 上找到我们：[*https://linkedin.com/company/oreilly-media*](https://linkedin.com/company/oreilly-media)

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*https://youtube.com/oreillymedia*](https://youtube.com/oreillymedia)

# 致谢

这是两位生活在美国的跨性别移民写的一本书，在这个时候，墙似乎正在向我们逼近。我们选择将这本书献给那些为更公正世界而战的人，无论方式多么微小——谢谢你们。对于我们失去或未能见面的所有人，我们怀念你们。对于我们尚未见面的人，我们期待与你们相遇。

如果没有构建这本书的社区支持，它将无法存在。从 Dask 社区到 PyData 社区，谢谢你们。感谢所有早期的读者和评论者对你们的贡献和指导。这些评论者包括 Ruben Berenguel、Adam Breindel、Tom Drabas、Joseph Gnanaprakasam、John Iannone、Kevin Kho、Jess Males 等。特别感谢 Ann Spencer 在最终成为这本书和*Scaling Python with Ray*的提案的早期审查中提供的帮助。任何剩余的错误完全是我们自己的责任，有时候我们违背了评论者的建议。¹

Holden 还要感谢她的妻子和伙伴们忍受她长时间的写作时间（有时候在浴缸里）。特别感谢 Timbit 保护房子并给 Holden 一个起床的理由（尽管对她来说有时候会太早）。

![spwd 00in01](img/spwd_00in01.png)

Mika 还要特别感谢 Holden 对她的指导和帮助，并感谢哈佛数据科学系的同事们为她提供无限量的免费咖啡。

¹ 有时我们固执到了极点。
