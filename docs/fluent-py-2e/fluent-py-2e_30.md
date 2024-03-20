# 后记

> Python 是一个成年人的语言。
> 
> Alan Runyan，Plone 联合创始人

Alan 的简洁定义表达了 Python 最好的特质之一：它不会干扰你，而是让你做你必须做的事情。这也意味着它不会给你工具来限制别人对你的代码和构建的对象所能做的事情。

30 岁时，Python 仍在不断增长。但当然，它并不完美。对我来说，最令人恼火的问题之一是标准库中对`CamelCase`、`snake_case`和`joinedwords`的不一致使用。但语言定义和标准库只是生态系统的一部分。用户和贡献者的社区是 Python 生态系统中最好的部分。

这是社区最好的一个例子：在第一版中写关于*asyncio*时，我感到沮丧，因为 API 有许多函数，其中几十个是协程，你必须用`yield from`调用协程—现在用`await`—但你不能对常规函数这样做。这在*asyncio*页面中有记录，但有时你必须读几段才能找出特定函数是否是协程。所以我给 python-tulip 发送了一封标题为[“建议：在*asyncio*文档中突出显示协程”](https://fpy.li/a-1)的消息。*asyncio*核心开发者 Victor Stinner；*aiohttp*的主要作者 Andrew Svetlov；Tornado 的首席开发人员 Ben Darnell；以及*Twisted*的发明者 Glyph Lefkowitz 加入了讨论。Darnell 提出了一个解决方案，Alexander Shorin 解释了如何在 Sphinx 中实现它，Stinner 添加了必要的配置和标记。在我提出问题不到 12 小时后，整个*asyncio*在线文档集都更新了今天你可以看到的[*coroutine*标签](https://fpy.li/a-2)。

那个故事并不发生在一个独家俱乐部。任何人都可以加入 python-tulip 列表，当我写这个提案时，我只发过几次帖子。这个故事说明了一个真正对新想法和新成员开放的社区。Guido van Rossum 过去常常出现在 python-tulip 中，并经常回答基本问题。

另一个开放性的例子：Python 软件基金会（PSF）一直致力于增加 Python 社区的多样性。一些令人鼓舞的结果已经出现。2013 年至 2014 年，PSF 董事会首次选举了女性董事：Jessica McKellar 和 Lynn Root。2015 年，Diana Clarke 在蒙特利尔主持了 PyCon 北美大会，大约三分之一的演讲者是女性。PyLadies 成为一个真正的全球运动，我为我们在巴西有这么多 PyLadies 分部感到自豪。

如果你是 Python 爱好者但还没有参与社区，我鼓励你这样做。寻找你所在地区的 PyLadies 或 Python 用户组（PUG）。如果没有，就创建一个。Python 无处不在，所以你不会孤单。如果可以的话，参加活动。也参加线上活动。在新冠疫情期间，我在线会议的“走廊轨道”中学到了很多东西。来参加 PythonBrasil 大会—多年来我们一直有国际演讲者。和其他 Python 爱好者一起交流不仅带来知识分享，还有真正的好处。比如真正的工作和真正的友谊。

我知道如果没有多年来在 Python 社区结识的许多朋友的帮助，我不可能写出这本书。

我的父亲，贾伊罗·拉马尔，曾经说过“Só erra quem trabalha”，葡萄牙语中的“只有工作的人会犯错”，这是一个避免被犯错的恐惧所束缚的好建议。在写这本书的过程中，我肯定犯了很多错误。审阅者、编辑和早期发布的读者发现了很多错误。在第一版早期发布的几个小时内，一位读者在书的勘误页面上报告了错别字。其他读者提供了更多报告，朋友们直接联系我提出建议和更正。O’Reilly 的编辑们在制作过程中会发现其他错误，一旦我停止写作就会开始。我对任何错误和次优的散文负责并致歉。

我很高兴完成这第二版，包括错误，我非常感谢在这个过程中帮助过我的每个人。

希望很快能在某个现场活动中见到你。如果看到我，请过来打个招呼！

# 进一步阅读

我将以关于“Pythonic”的参考资料结束本书——这本书试图解决的主要问题。

Brandon Rhodes 是一位出色的 Python 教师，他的演讲[“Python 美学：美丽和我为什么选择 Python”](https://fpy.li/a-3)非常出色，标题中使用了 Unicode U+00C6（`LATIN CAPITAL LETTER AE`）。另一位出色的教师 Raymond Hettinger 在 PyCon US 2013 上谈到了 Python 中的美：[“将代码转化为美丽、惯用的 Python”](https://fpy.li/a-4)。

伊恩·李在 Python-ideas 上发起的[“风格指南的演变”主题](https://fpy.li/a-5)值得一读。李是[`pep8`](https://fpy.li/a-6)包的维护者，用于检查 Python 源代码是否符合 PEP 8 规范。为了检查本书中的代码，我使用了[`flake8`](https://fpy.li/a-7)，它包含了`pep8`、[`pyflakes`](https://fpy.li/a-8)，以及 Ned Batchelder 的[McCabe 复杂度插件](https://fpy.li/a-9)。

除了 PEP 8，其他有影响力的风格指南还有[*Google Python 风格指南*](https://fpy.li/a-10)和[*Pocoo 风格指南*](https://fpy.li/a-11)，这两个团队为我们带来了 Flake、Sphinx、Jinja 2 等伟大的 Python 库。

[*Python 之旅者指南！*](https://fpy.li/a-12)是关于编写 Pythonic 代码的集体作品。其中最多产的贡献者是 Kenneth Reitz，由于他出色的 Pythonic `requests` 包，他成为了社区英雄。David Goodger 在 PyCon US 2008 上做了一个名为[“像 Pythonista 一样编码：Python 的惯用法”](https://fpy.li/a-13)的教程。如果打印出来，教程笔记有 30 页长。Goodger 创建了 reStructuredText 和 `docutils`——Sphinx 的基础，Python 出色的文档系统（顺便说一句，这也是 MongoDB 和许多其他项目的[官方文档系统](https://fpy.li/a-14)）。

马蒂恩·法森在[“什么是 Pythonic？”](https://fpy.li/a-15)中直面这个问题。在 python-list 中，有一个同名主题的讨论[线程](https://fpy.li/a-16)。马蒂恩的帖子是 2005 年的，而主题是 2003 年的，但 Pythonic 的理念并没有改变太多——语言本身也是如此。一个标题中带有“Pythonic”的很棒的主题是[“Pythonic way to sum n-th list element？”](https://fpy.li/a-17)，我在“Soapbox”中广泛引用了其中的内容。

[PEP 3099 — Python 3000 中不会改变的事情](https://fpy.li/pep3099)解释了为什么许多事情仍然保持原样，即使 Python 3 进行了重大改革。很长一段时间，Python 3 被昵称为 Python 3000，但它提前了几个世纪到来——这让一些人感到沮丧。PEP 3099 是由 Georg Brandl 撰写的，汇编了许多由*BDFL*，Guido van Rossum 表达的观点。[“Python Essays”](https://fpy.li/a-18)页面列出了 Guido 本人撰写的几篇文章。
