# 前言

Python 3.4 引入了`asyncio`库，Python 3.5 引入了`async`和`await`关键字以更加方便地使用它。这些新添加的内容允许所谓的*异步*编程。

所有这些新特性，我将统称为*Asyncio*，在 Python 社区中有些谨慎的接受；部分社区成员似乎认为它们复杂且难以理解。这种观点不仅限于初学者：Python 社区的几位知名贡献者对 Asyncio API 的复杂性表示怀疑，社区中的教育工作者也对如何最好地教授 Asyncio 表示关注。

大多数有几年 Python 经验的人之前都用过线程，即使你没有，你也很可能已经经历了*阻塞*。例如，如果你使用了出色的`requests`库编写程序，你肯定会注意到你的程序在执行`requests.get(url)`时会暂停一会儿；这是阻塞行为。

对于一次性任务来说，这样做是可以的；但如果你想同时获取*一万个*URL，使用`requests`将会很困难。大规模并发是学习和使用 Asyncio 的一个重要原因，但 Asyncio 比抢占式线程的另一个吸引之处在于安全性：使用 Asyncio 将更容易避免竞态条件错误。

我写这本书的目标是让你基本了解为什么引入了这些新特性以及如何在自己的项目中使用它们。更具体地说，我旨在提供以下内容：

+   *asyncio*和*threading*在并发网络编程中的关键比较

+   对新的`async/await`语言语法的理解

+   Python 中新`asyncio`标准库特性的概述

+   详细的案例研究和代码示例，展示如何使用一些较流行的 Asyncio 兼容的第三方库

我们将以一个故事开始，说明从线程到异步编程的思维转变。然后，我们将看看 Python 语言本身为适应异步编程所做的变化。最后，我们将探讨如何最有效地使用这些新特性。

新的 Asyncio 特性不会彻底改变你编写程序的方式。它们提供了特定工具，只在特定情况下才有意义；但在适当的情况下，`asyncio`是异常有用的。在本书中，我们将探讨这些情况以及如何通过使用新的 Asyncio 特性最好地解决它们。

# 本书使用的约定

本书使用以下排版约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落内引用程序元素，例如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`等宽粗体`**

显示用户应逐字输入的命令或其他文本。

*`等宽斜体`*

显示应由用户提供值或由上下文确定值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# O’Reilly 在线学习

###### 注意

超过 40 年来，[*O’Reilly Media*](http://oreilly.com) 一直为公司提供技术和商业培训、知识和见解，帮助其成功。

我们独特的专家和创新者网络通过图书、文章、会议和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的现场培训课程、深度学习路径、交互式编码环境以及来自 O’Reilly 和其他 200 多个出版商的大量文本和视频。欲了解更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大地区）

+   707-829-0515（国际或当地）

+   707-829-0104（传真）

我们为本书制作了一个网页，其中列出了勘误、示例和任何额外信息。网址为：[*https://oreil.ly/using-asyncio-in-python*](https://oreil.ly/using-asyncio-in-python)。

有关此书的评论或技术问题，请发送电子邮件至*bookquestions@oreilly.com*。

欲了解更多关于我们的图书、课程、会议和新闻的信息，请访问我们的网站：[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上查找我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上关注我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

非常感谢 Ashwini Balnaves 和 Kevin Baker 在本书的早期草稿中付出的努力，并提供了宝贵的反馈。我深表感激 Yury Selivanov 抽出宝贵的时间审阅本书的早期版本，当时它是作为 O’Reilly 报告首次出版的。最后，我还要感谢 O’Reilly 团队的出色编辑支持。
