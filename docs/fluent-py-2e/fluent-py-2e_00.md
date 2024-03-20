# 前言

> 计划是这样的：当有人使用你不理解的特性时，直接开枪打死他们。这比学习新东西要容易得多，不久之后，活下来的程序员只会用一个容易理解的、微小的 Python 0.9.6 子集来编写代码 <wink>。¹
> 
> Tim Peters，传奇的核心开发者，*Python 之禅*的作者

"Python 是一种易于学习、功能强大的编程语言。"这是[官方 Python 3.10 教程](https://fpy.li/p-2)的开篇词。这是真的，但有一个问题：因为这门语言易学易用，许多实践中的 Python 程序员只利用了它强大特性的一小部分。

有经验的程序员可能在几个小时内就开始编写有用的 Python 代码。当最初富有成效的几个小时变成几周和几个月时，许多开发人员会继续用之前学过的语言的强烈口音编写 Python 代码。即使 Python 是你的第一门语言，在学术界和入门书籍中，它通常被小心地避开语言特定的特性来呈现。

作为一名向有其他语言经验的程序员介绍 Python 的老师，我看到了这本书试图解决的另一个问题：我们只会错过我们知道的东西。来自另一种语言，任何人都可能猜测 Python 支持正则表达式，并在文档中查找。但是，如果你以前从未见过元组解包或描述符，你可能不会搜索它们，最终可能不会使用这些特性，只是因为它们是 Python 特有的。

本书不是 Python 的 A 到 Z 详尽参考。它强调 Python 独有的或在许多其他流行语言中找不到的语言特性。这也主要是一本关于核心语言及其一些库的书。我很少会谈论不在标准库中的包，尽管 Python 包索引现在列出了超过 60,000 个库，其中许多非常有用。

# 本书适合的读者

本书是为想要精通 Python 3 的在职 Python 程序员编写的。我在 Python 3.10 中测试了这些示例，大部分也在 Python 3.9 和 3.8 中测试过。如果某个示例需要 Python 3.10，会有明确标注。

如果你不确定自己是否有足够的 Python 知识来跟上，请复习官方 [Python 教程](https://fpy.li/p-3)的主题。除了一些新特性外，本书不会解释教程中涉及的主题。

# 本书不适合的读者

如果你刚开始学习 Python，这本书可能很难理解。不仅如此，如果你在 Python 学习之旅的早期阶段阅读它，可能会给你一种印象，认为每个 Python 脚本都应该利用特殊方法和元编程技巧。过早的抽象和过早的优化一样糟糕。

# 五合一的书

我建议每个人都阅读第一章，"Python 数据模型"。本书的核心读者在阅读完第一章后，应该不会有什么困难直接跳到本书的任何部分，但我经常假设你已经阅读了每个特定部分的前面章节。可以把第一部分到第五部分看作是书中之书。

我试图强调在讨论如何构建自己的东西之前先使用现有的东西。例如，在第一部分中，第二章涵盖了现成可用的序列类型，包括一些不太受关注的类型，如`collections.deque`。用户自定义序列直到第三部分才会讲到，在那里我们还会看到如何利用`collections.abc`中的抽象基类（ABC）。创建自己的 ABC 要更晚在第三部分中讨论，因为我认为在编写自己的 ABC 之前，熟悉使用现有的 ABC 很重要。

这种方法有几个优点。首先，知道什么是现成可用的，可以避免你重新发明轮子。我们使用现有的集合类比实现自己的集合类更频繁，并且我们可以通过推迟讨论如何创建新类，而将更多注意力放在可用工具的高级用法上。我们也更有可能从现有的 ABC 继承，而不是从头开始创建新的 ABC。最后，我认为在你看到这些抽象的实际应用之后，更容易理解它们。

这种策略的缺点是章节中散布着前向引用。我希望现在你知道我为什么选择这条路，这些引用会更容易容忍。

## 本书的组织方式

以下是本书各部分的主要主题：

第 I 部分，"数据结构"

第一章介绍了 Python 数据模型，并解释了为什么特殊方法（例如，`__repr__`）是所有类型的对象行为一致的关键。本书将更详细地介绍特殊方法。本部分的其余章节涵盖了集合类型的使用：序列、映射和集合，以及`str`与`bytes`的分离——这给 Python 3 用户带来了许多欢呼，而让迁移代码库的 Python 2 用户感到痛苦。还介绍了标准库中的高级类构建器：命名元组工厂和`@dataclass`装饰器。第二章、第三章和第五章中的部分介绍了 Python 3.10 中新增的模式匹配，分别讨论了序列模式、映射模式和类模式。第 I 部分的最后一章是关于对象的生命周期：引用、可变性和垃圾回收。

第 II 部分，"作为对象的函数"

在这里，我们讨论作为语言中一等对象的函数：这意味着什么，它如何影响一些流行的设计模式，以及如何通过利用闭包来实现函数装饰器。还涵盖了 Python 中可调用对象的一般概念、函数属性、内省、参数注解以及 Python 3 中新的`nonlocal`声明。第八章介绍了函数签名中类型提示的主要新主题。

第 III 部分，"类和协议"

现在的重点是"手动"构建类——而不是使用第五章中介绍的类构建器。与任何面向对象（OO）语言一样，Python 有其特定的功能集，这些功能可能存在也可能不存在于你和我学习基于类的编程的语言中。这些章节解释了如何构建自己的集合、抽象基类（ABC）和协议，以及如何处理多重继承，以及如何在有意义时实现运算符重载。第十五章继续介绍类型提示。

第 IV 部分，"控制流"

这一部分涵盖了超越传统的使用条件、循环和子程序的控制流的语言构造和库。我们从生成器开始，然后访问上下文管理器和协程，包括具有挑战性但功能强大的新 `yield from` 语法。第十八章包含一个重要的示例，在一个简单但功能齐全的语言解释器中使用模式匹配。第十九章，"Python 中的并发模型"是一个新章节，概述了 Python 中并发和并行处理的替代方案、它们的局限性以及软件架构如何允许 Python 在网络规模下运行。我重写了关于*异步编程*的章节，强调核心语言特性，例如 `await`、`async dev`、`async for` 和 `async with`，并展示了它们如何与 *asyncio* 和其他框架一起使用。

第五部分，"元编程"

这一部分从回顾用于构建具有动态创建属性以处理半结构化数据（如 JSON 数据集）的类的技术开始。接下来，我们介绍熟悉的属性机制，然后深入探讨 Python 中对象属性访问如何在较低级别使用描述符工作。解释了函数、方法和描述符之间的关系。在第五部分中，逐步实现字段验证库，揭示了微妙的问题，这些问题导致了最后一章中的高级工具：类装饰器和元类。

# 动手实践的方法

我们经常会使用交互式 Python 控制台来探索语言和库。我觉得强调这种学习工具的力量很重要，尤其是对那些有更多使用静态编译语言经验而没有提供读取-求值-打印循环（REPL）的读者而言。

标准 Python 测试包之一 [`doctest`](https://fpy.li/doctest)，通过模拟控制台会话并验证表达式是否得出所示的响应来工作。我用 `doctest` 检查了本书中的大部分代码，包括控制台列表。你不需要使用甚至了解 `doctest` 就可以跟随：doctests 的关键特性是它们看起来像是交互式 Python 控制台会话的记录，所以你可以轻松地自己尝试这些演示。

有时，我会在编写使其通过的代码之前，通过展示 doctest 来解释我们想要完成的任务。在考虑如何做之前牢固地确立要做什么，有助于集中我们的编码工作。先编写测试是测试驱动开发（TDD）的基础，我发现它在教学时也很有帮助。如果你不熟悉 `doctest`，请查看其[文档](https://fpy.li/doctest)和本书的[示例代码仓库](https://fpy.li/code)。

我还使用 *pytest* 为一些较大的示例编写了单元测试——我发现它比标准库中的 *unittest* 模块更易于使用且功能更强大。你会发现，通过在操作系统的命令行 shell 中键入 `python3 -m doctest example_script.py` 或 `pytest`，可以验证本书中大多数代码的正确性。[示例代码仓库](https://fpy.li/code)根目录下的 *pytest.ini* 配置确保 doctests 被 `pytest` 命令收集和执行。

# 皂盒：我的个人观点

从 1998 年开始，我一直在使用、教授和探讨 Python，我喜欢研究和比较编程语言、它们的设计以及背后的理论。在一些章节的末尾，我添加了"皂盒"侧边栏，其中包含我自己对 Python 和其他语言的看法。如果你不喜欢这样的讨论，请随意跳过。它们的内容完全是可选的。

# 配套网站：fluentpython.com

为了涵盖新特性（如类型提示、数据类和模式匹配），第二版的内容比第一版增加了近 30%。为了保持书本的便携性，我将一些内容移至 [*fluentpython.com*](http://fluentpython.com)。你会在几个章节中找到我在那里发表的文章的链接。配套网站上也有一些示例章节。完整文本可在 [O'Reilly Learning](https://fpy.li/p-5) 订阅服务的[在线版本](https://fpy.li/p-4)中获得。示例代码仓库在 [GitHub](https://fpy.li/code) 上。

# 本书中使用的约定

本书使用以下排版惯例：

*Italic*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落内引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

请注意，当换行符出现在 `constant_width` 术语中时，不会添加连字符，因为它可能被误解为术语的一部分。

**`Constant width bold`**

显示用户应按字面意思键入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

书中出现的每个脚本和大多数代码片段都可在 GitHub 上的 Fluent Python 代码仓库中找到，网址为 [*https://fpy.li/code*](https://fpy.li/code)。

如果你有技术问题或使用代码示例的问题，请发送电子邮件至 *bookquestions@oreilly.com*。

这本书旨在帮助你完成工作。一般来说，如果本书提供了示例代码，你可以在程序和文档中使用它。除非你要复制大量代码，否则无需联系我们征得许可。例如，编写一个使用本书多个代码片段的程序不需要许可。出售或分发 O'Reilly 图书中的示例需要获得许可。通过引用本书和引用示例代码来回答问题不需要许可。将本书中大量示例代码合并到你的产品文档中确实需要许可。

我们感谢但通常不要求注明出处。出处通常包括标题、作者、出版商和 ISBN，例如："*Fluent Python*，第 2 版，Luciano Ramalho 著（O'Reilly）。2022 Luciano Ramalho 版权所有，978-1-492-05635-5。"

如果你认为你对代码示例的使用超出了合理使用范围或上述许可范围，请随时通过 *permissions@oreilly.com* 与我们联系。

# O'Reilly 在线学习

###### 注意 

40 多年来，[*O'Reilly Media*](http://oreilly.com) 一直在提供技术和商业培训、知识和见解，帮助企业取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业技能。O'Reilly 的在线学习平台让你可以按需访问现场培训课程、深入学习路径、交互式编码环境，以及来自 O'Reilly 和其他 200 多家出版商的大量文本和视频。有关更多信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送给出版商：

+   O'Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

这本书有一个网页，我们在其中列出勘误表、示例和任何额外的信息。你可以通过 [*https://fpy.li/p-4*](https://fpy.li/p-4) 访问此页面。

发送电子邮件至 *bookquestions@oreilly.com* 以评论或询问有关本书的技术问题。

要了解我们的书籍和课程的新闻和信息，请访问 [*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)。

在 Twitter 上关注我们：[*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)。

在 YouTube 上观看我们的视频：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)。

# 致谢

我没想到五年后更新一本 Python 书会是如此重大的任务，但事实如此。我挚爱的妻子 Marta Mello 总是在我需要她的时候出现。我亲爱的朋友 Leonardo Rochael 从最早的写作到最后的技术审核都一直帮助我，包括整合和复核其他技术审核人员、读者和编辑的反馈。说实话，如果没有你们的支持，Marta 和 Leo，我不知道自己是否能做到。非常感谢你们！

Jürgen Gmach、Caleb Hattingh、Jess Males、Leonardo Rochael 和 Miroslav Šedivý 是第二版的杰出技术审查团队。他们审阅了整本书。Bill Behrman、Bruce Eckel、Renato Oliveira 和 Rodrigo Bernardo Pimentel 审阅了特定章节。他们从不同角度提出的许多建议使本书变得更好。

在早期发布阶段，许多读者发送了更正或做出了其他贡献，包括：Guilherme Alves、Christiano Anderson、Konstantin Baikov、K. Alex Birch、Michael Boesl、Lucas Brunialti、Sergio Cortez、Gino Crecco、Chukwuerika Dike、Juan Esteras、Federico Fissore、Will Frey、Tim Gates、Alexander Hagerman、Chen Hanxiao、Sam Hyeong、Simon Ilincev、Parag Kalra、Tim King、David Kwast、Tina Lapine、Wanpeng Li、Guto Maia、Scott Martindale、Mark Meyer、Andy McFarland、Chad McIntire、Diego Rabatone Oliveira、Francesco Piccoli、Meredith Rawls、Michael Robinson、Federico Tula Rovaletti、Tushar Sadhwani、Arthur Constantino Scardua、Randal L. Schwartz、Avichai Sefati、Guannan Shen、William Simpson、Vivek Vashist、Jerry Zhang、Paul Zuradzki 以及其他不愿透露姓名的人，在我交稿后发送了更正，或者因为我没有记录他们的名字而被遗漏——抱歉。

在研究过程中，我在与 Michael Albert、Pablo Aguilar、Kaleb Barrett、David Beazley、J.S.O. Bueno、Bruce Eckel、Martin Fowler、Ivan Levkivskyi、Alex Martelli、Peter Norvig、Sebastian Rittau、Guido van Rossum、Carol Willing 和 Jelle Zijlstra 的互动中了解了类型、并发、模式匹配和元编程。

O'Reilly 编辑 Jeff Bleiel、Jill Leonard 和 Amelia Blevins 提出的建议在许多地方改善了本书的流畅度。Jeff Bleiel 和制作编辑 Danny Elfanbaum 在整个漫长的马拉松中都一直支持我。

他们每个人的见解和建议都让这本书变得更好、更准确。不可避免地，最终产品中仍然会有我自己制造的错误。我提前表示歉意。

最后，我要向我在 Thoughtworks 巴西的同事们表示衷心的感谢，尤其是我的赞助人 Alexey Bôas，他们一直以多种方式支持这个项目。

当然，每一个帮助我理解 Python 并编写第一版的人现在都应该得到双倍的感谢。没有成功的第一版就不会有第二版。

## 第一版致谢

Josef Hartwig 设计的包豪斯国际象棋是优秀设计的典范：美观、简洁、清晰。建筑师之子、字体设计大师之弟 Guido van Rossum 创造了一部语言设计的杰作。我喜欢教授 Python，因为它美观、简洁、清晰。

Alex Martelli 和 Anna Ravenscroft 是最早看到本书大纲并鼓励我将其提交给 O'Reilly 出版的人。他们的书教会了我地道的 Python，是技术写作在清晰、准确和深度方面的典范。[Alex 在 Stack Overflow 上的 6,200 多个帖子](https://fpy.li/p-7)是语言及其正确使用方面的见解源泉。

Martelli 和 Ravenscroft 也是本书的技术评审，还有 Lennart Regebro 和 Leonardo Rochael。这个杰出的技术评审团队中的每个人都有至少 15 年的 Python 经验，对与社区中其他开发人员密切联系的高影响力 Python 项目做出了许多贡献。他们一起给我发来了数百条修正、建议、问题和意见，为本书增添了巨大的价值。Victor Stinner 友好地审阅了第二十一章，将他作为 `asyncio` 维护者的专业知识带到了技术评审团队中。能在过去的几个月里与他们合作，我感到非常荣幸和愉快。

编辑 Meghan Blanchette 是一位杰出的导师，帮助我改进了本书的组织和流程，让我知道什么时候它变得无聊，并阻止我进一步拖延。Brian MacDonald 在 Meghan 不在时编辑了第二部分的章节。我很高兴与他们以及我在 O'Reilly 联系过的每个人合作，包括 Atlas 开发和支持团队（Atlas 是 O'Reilly 的图书出版平台，我很幸运能使用它来写这本书）。  

Mario Domenech Goulart 从第一个早期版本开始就提供了大量详细的建议。我还收到了 Dave Pawson、Elias Dorneles、Leonardo Alexandre Ferreira Leite、Bruce Eckel、J.S. Bueno、Rafael Gonçalves、Alex Chiaranda、Guto Maia、Lucas Vido 和 Lucas Brunialti 的宝贵反馈。

多年来，许多人敦促我成为一名作家，但最有说服力的是 Rubens Prates、Aurelio Jargas、Rudá Moura 和 Rubens Altimari。Mauricio Bussab 为我打开了许多大门，包括我第一次真正尝试写书。Renzo Nuccitelli 一路支持这个写作项目，即使这意味着我们在 [*python.pro.br*](https://fpy.li/p-8) 的合作起步缓慢。

美妙的巴西 Python 社区知识渊博、慷慨大方、充满乐趣。[Python Brasil 小组](https://fpy.li/p-9)有数千人，我们的全国会议汇聚了数百人，但在我的 Pythonista 旅程中最具影响力的是 Leonardo Rochael、Adriano Petrich、Daniel Vainsencher、Rodrigo RBP Pimentel、Bruno Gola、Leonardo Santagada、Jean Ferri、Rodrigo Senra、 J.S. Bueno、David Kwast、Luiz Irber、Osvaldo Santana、Fernando Masanori、Henrique Bastos、Gustavo Niemayer、Pedro Werneck、Gustavo Barbieri、Lalo Martins、Danilo Bellini 和 Pedro Kroger。

Dorneles Tremea 是一位伟大的朋友（他慷慨地奉献时间和知识），一位了不起的黑客，也是巴西 Python 协会最鼓舞人心的领导者。他离开得太早了。

多年来，我的学生通过他们的提问、见解、反馈和创造性的问题解决方案教会了我很多东西。Érico Andrei 和 Simples Consultoria 让我第一次能够专注于当一名 Python 老师。

Martijn Faassen 是我的 Grok 导师，与我分享了关于 Python 和尼安德特人的宝贵见解。他以及 Paul Everitt、Chris McDonough、Tres Seaver、Jim Fulton、Shane Hathaway、Lennart Regebro、Alan Runyan、Alexander Limi、Martijn Pieters、Godefroid Chapelle 等来自 Zope、Plone 和 Pyramid 星球的人的工作对我的职业生涯起到了决定性作用。多亏了 Zope 和冲浪第一波网络浪潮，我能够从 1998 年开始以 Python 谋生。José Octavio Castro Neves 是我在巴西第一家以 Python 为中心的软件公司的合伙人。

在更广泛的 Python 社区中，我有太多的大师无法一一列举，但除了已经提到的那些，我还要感谢 Steve Holden、Raymond Hettinger、A.M. Kuchling、David Beazley、Fredrik Lundh、Doug Hellmann、Nick Coghlan、Mark Pilgrim、Martijn Pieters、Bruce Eckel、Michele Simionato、Wesley Chun、Brandon Craig Rhodes、Philip Guo、Daniel Greenfeld、Audrey Roy 和 Brett Slatkin，感谢他们教会我新的更好的 Python 教学方式。

这些页面大部分是在我的家庭办公室和两个实验室写的：CoffeeLab 和 Garoa Hacker Clube。[CoffeeLab](https://fpy.li/p-10) 是位于巴西圣保罗 Vila Madalena 的咖啡因极客总部。[Garoa Hacker Clube](https://fpy.li/p-11) 是一个向所有人开放的黑客空间：一个社区实验室，任何人都可以自由尝试新想法。

Garoa 社区提供了灵感、基础设施和宽松的环境。我想 Aleph 会喜欢这本书。

我的母亲 Maria Lucia 和父亲 Jairo 总是全力支持我。我希望他能在这里看到这本书，我很高兴能与她分享。

我的妻子 Marta Mello 忍受了 15 个月总是在工作的丈夫，但她仍然保持支持，并在我担心可能会退出这个马拉松项目的一些关键时刻给予我指导。

谢谢你们，感谢一切。

¹ 2002 年 12 月 23 日在 comp.lang.python Usenet 小组的留言："[Acrimony in c.l.p](https://fpy.li/p-1)"。
