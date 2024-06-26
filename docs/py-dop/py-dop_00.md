# 序言

有一次，诺亚在海里，一浪砸在他身上，把他的呼吸夺走，同时把他拖得更深入海中。就在他开始恢复呼吸时，又一浪猛然砸下来。它消耗了他剩余的大部分能量，把他拉得更深。正当他开始恢复时，又有一浪猛地打来。他越是与浪和海抗争，消耗的能量就越多。他严重怀疑他会在那一刻死去。他无法呼吸，全身疼痛，他害怕自己会溺水而亡。濒临死亡让他专注于唯一能救他的事情，那就是保存自己的能量，利用浪潮而非与之对抗。

身处不实践 DevOps 的初创企业就像是在海滩的那一天。有些生产问题燃烧了数月；一切都是手工操作，警报连续几天不断地将你吵醒，损害你的健康。唯一摆脱这种死亡螺旋的途径就是 DevOps 方法。

做一件正确的事情，然后再做另一件，直到找到清晰。首先，建立一个构建服务器，开始测试你的代码，并自动化手动任务。做点什么，可以是任何事情，但要有“偏向行动”的态度。先做对第一件事情，并确保它是自动化的。

初创企业或任何公司的一个常见陷阱是寻找超级英雄。“我们需要一个性能工程师”，因为他们将解决我们的性能问题。“我们需要一位首席营收官”，因为他们将解决所有销售问题。“我们需要 DevOps 工程师”，因为他们将解决我们的部署流程。

在一家公司，诺亚负责的一个项目已经延迟超过一年，而且这个 Web 应用已经重写了三次，使用了多种编程语言。这个即将发布的版本只需要一个“性能工程师”来完成。我记得当时只有我一个人足够勇敢或者愚蠢，问道：“什么是性能工程师？” 这位工程师使得所有事情在规模上运行良好。他在那一刻意识到他们正在寻找一个超级英雄来拯救他们。招募超级英雄综合症是发现新产品或新创企业出现非常严重问题的最佳方式。除非员工首先拯救了自己，否则不会有人能拯救一个公司。

在其他公司，诺亚听到类似的话：“如果我们只能雇一个资深 Erlang 工程师”，或者“如果我们只能雇一个为我们创造收入的人”，或者“如果我们只能雇一个教会我们财务纪律的人”，或者“如果我们只能雇一个 Swift 开发者”，等等。这种雇佣是你的初创企业或新产品最不需要的东西——它需要理解自己在做错了什么，只有超级英雄才能拯救一天。

对于那家想要聘请性能工程师的公司来说，最终问题是技术监督不足。错误的人掌管着事务（并且口头上打击那些可以修复问题的人）。通过移除一位表现不佳的员工，听取一位一直知道如何解决问题的现有团队成员的建议，删除那份工作列表，一次做对一件事情，然后加入合格的工程管理人员，问题就解决了。

在创业公司，没有人会拯救你；你和你的团队必须通过创建出色的团队合作、优秀的流程并信任你的组织来保护自己。问题的解决方案不是新的雇员，而是诚实和关注你所处的情况，如何到达那里，并一次做对一件事情直到摆脱困境。除非是你，否则没有超级英雄。

就像在风暴中在海洋中慢慢淹没，没有人会拯救你或公司，除非是你。你是公司需要的超级英雄，你可能会发现你的同事也是。

有一条走出混乱的路，而这本书可以成为你的指南。让我们开始吧。

# 作者们认为 DevOps 的含义是什么？

软件行业中许多抽象概念很难精确定义。云计算、敏捷和大数据都是可以根据与其讨论的人不同而有多种定义的示例。与其严格定义 DevOps 是什么，不如使用一些表明 DevOps 正在发生的短语：

+   开发和运维团队之间的双向协作。

+   运维任务的周转时间是几分钟到几小时，而不是几天到几周。

+   开发人员的强烈参与是必要的；否则，就会回到开发人员与运维人员的对立。

+   运维人员需要开发技能——至少需要熟练使用 Bash 和 Python。

+   开发人员需要操作技能——他们的责任不仅是编写代码，而是将系统部署到生产环境并监控警报。

+   自动化，自动化，自动化：你不能准确地自动化而没有开发技能，也不能正确地自动化而没有运维技能。

+   理想情况下：开发人员可以自助地进行代码部署。

+   可以通过 CI/CD 流水线实现。

+   GitOps。

+   开发和运维之间的双向 *everything*（工具、知识等）交流。

+   设计、实施、部署以及是的，自动化的持续协作如果没有合作，就不可能成功。

+   如果不自动化，就会出问题。

+   文化层次结构 < 过程。

+   微服务 > 单块式架构。

+   软件团队的核心是持续部署系统。

+   没有超级英雄。

+   持续交付不是选择，而是必须。

# 如何使用本书

本书可以按任意顺序阅读。您可以随意打开任何您喜欢的章节，应该能够找到有助于工作的有用内容。如果您是一位经验丰富的 Python 程序员，您可能想要浏览第一章。同样，如果您对战争故事、案例研究和访谈感兴趣，您可能想先阅读第十六章。

## 概念主题

内容分为几个概念主题。第一组是 Python 基础，涵盖了语言的简要介绍，以及自动化文本、编写命令行工具和自动化文件系统。

接下来是运维，包括有用的 Linux 工具、软件包管理、构建系统、监控与仪表、以及自动化测试。这些都是成为称职的 DevOps 从业者必须掌握的关键主题。

云基础将在下一节详细介绍，包括云计算、基础设施即代码、Kubernetes 和无服务器的章节。当前软件行业在云计算领域缺乏足够的人才，掌握本节将立即提升您的薪资和职业发展。

数据部分接下来。机器学习运营和数据工程都从 DevOps 的视角进行了探讨。此外，还有一个全面的机器学习项目演示，介绍如何使用 Flask、Sklearn、Docker 和 Kubernetes 构建、部署和运营化机器学习模型。

最后一部分是关于案例研究、访谈和 DevOps 战争故事的第十六章。这章节非常适合晚上阅读。

Python 基础

+   第一章，*DevOps 的 Python 基础*

+   第二章，*自动化文件和文件系统*

+   第三章，*使用命令行*

运维

+   第四章，*有用的 Linux 工具*

+   第五章，*软件包管理*

+   第六章，*持续集成与持续部署*

+   第七章，*监控和日志*

+   第八章，*DevOps 的 Pytest*

云基础

+   第九章，*云计算*

+   第十章，*基础设施即代码*

+   第十一章，*容器技术：Docker 和 Docker Compose*

+   第十二章，*容器编排：Kubernetes*

+   第十三章，*无服务器技术*

数据

+   第十四章，*MLOps 和机器学习工程*

+   第十五章，*数据工程*

案例研究

+   第十六章，*DevOps 战争故事和访谈*

# 本书使用的约定

本书使用以下排版约定：

*斜体*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`常量宽度`

用于程序列表，以及段落内引用程序元素，如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`常量宽度粗体`**

显示用户应直接输入的命令或其他文本。

*`常量宽度斜体`*

显示应由用户提供值或由上下文确定值的文本。

###### 提示

此元素表示提示或建议。

###### 注释

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。

# 使用代码示例

可下载附加材料（代码示例、练习等）位于 [*https://pythondevops.com*](https://pythondevops.com)。您也可以在 [Pragmatic AI Labs YouTube 频道](https://oreil.ly/QIYte) 上查看与本书代码相关的 DevOps 内容。

如果您对作者有技术问题或使用代码示例遇到问题，请发送电子邮件至 *technical@pythondevops.com*。

本书旨在帮助您完成工作。通常情况下，如果本书提供示例代码，您可以在自己的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需征得我们的许可。例如，编写一个使用本书中几个代码片段的程序不需要许可。销售或分发来自 O’Reilly 图书的示例则需要许可。引用本书并引用示例代码来回答问题则无需许可。将本书中大量示例代码整合到产品文档中则需要许可。

我们感谢您的使用，但通常不要求署名。署名通常包括书名、作者、出版社和 ISBN。例如：“*Python for DevOps* 由 Noah Gift、Kennedy Behrman、Alfredo Deza 和 Grig Gheorghiu 编写。 (O’Reilly). 版权 2020 Noah Gift、Kennedy Behrman、Alfredo Deza、Grig Gheorghiu，978-1-492-05769-7。”

如果您认为您对代码示例的使用超出了合理使用或上述许可的范围，请随时通过 *permissions@oreilly.com* 联系我们。

# O’Reilly 在线学习

###### 注释

40 多年来，[*O’Reilly Media*](http://oreilly.com) 提供技术和商业培训、知识和见解，帮助公司取得成功。

我们独特的专家和创新者网络通过书籍、文章、会议和在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境，以及来自 O’Reilly 和其他 200 多家出版商的大量文本和视频资源。欲了解更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将有关本书的评论和问题发送至出版社：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为本书设立了一个网页，其中列出了勘误、示例和任何额外信息。你可以访问[*oreil.ly/python-for-devops*](https://oreil.ly/python-for-devops)获取相关信息。

发送电子邮件至*bookquestions@oreilly.com*以提出关于本书的评论或技术问题。

欲了解更多有关我们的书籍、课程、会议和新闻的信息，请访问我们的网站[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

观看我们的 YouTube 频道：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

# 致谢

首先，作者要感谢本书的两位主要技术审阅者：

Wes Novack 是一位专注于公共云系统和 Web 规模 SaaS 应用的架构师和工程师。他设计、构建和管理复杂系统，支持高可用基础设施、持续交付管道，并在 AWS 和 GCP 上托管大型、多语言微服务生态系统中进行快速发布。Wes 广泛使用各种语言、框架和工具来定义基础设施即代码，推动自动化，消除重复劳动。他通过导师制、研讨会和会议积极参与技术社区，还是 Pluralsight 视频课程的作者。Wes 支持 DevOps 的 CALMS 理念：文化（Culture）、自动化（Automation）、精益（Lean）、度量（Measurement）和分享（Sharing）。你可以在 Twitter 上找到他，账号是@WesleyTech，或者访问他的[个人博客](https://wesnovack.com)。

Brad Andersen 是一位软件工程师和架构师。他已经在专业领域设计和开发软件 30 年。他致力于推动变革和创新，在从企业组织到初创公司的各个领域担任领导和开发角色。目前，Brad 正在加利福尼亚大学伯克利分校攻读数据科学硕士学位。你可以在[Brad 的 LinkedIn 资料](https://www.linkedin.com/in/andersen-bradley)找到更多信息。

我们还要感谢 Jeremy Yabrow 和 Colin B. Erdman 为本书提供了许多优秀的想法和反馈。

## Noah

我要感谢这本书的合著者们：Grig、Kennedy 和 Alfredo。能和如此高效的团队合作真是一种不可思议的体验。

## Kennedy

感谢我的合著者们，能和你们一起工作真是一种乐趣。还要感谢家人对我的耐心和理解。

## Alfredo

在撰写本文九年前的 2010 年，我踏入了我的第一份软件工程工作。当时我 31 岁，没有大学学历，也没有任何工程经验。这份工作意味着我接受了较低的薪水和没有健康保险。通过不懈的努力，我学到了很多，结识了很多了不起的人，并通过这些年获得了专业知识。在这些年里，如果没有人们为我提供的机会和指引我走向正确方向，我是无法走到今天的。

感谢 Chris Benson，他看到了我渴望学习的心，一直找机会让我参与。

感谢 Alejandro Cadavid，他意识到我能解决其他人不愿意解决的问题。在大家（包括我自己）认为我毫无用处时，你帮助我找到了工作。

Carlos Coll 让我开始编程，即使我请求退出他也不让我放弃。学习编程改变了我的生活，Carlos 有耐心地推动我学习，并帮助我完成了第一个投入生产的程序。

特别感谢 Joni Benton，因为她相信我，并帮助我找到了我的第一份全职工作。

感谢 Jonathan LaCour，你是一个鼓舞人心的老板，一直帮助我走向更好的地方。你的建议对我来说是无价的。

Noah，感谢你的友谊和指导，你对我来说是一种巨大的动力源。我们一起工作总是让我很愉快，就像那次我们从头开始重建基础设施的经历。当我对 Python 毫无头绪时，你的耐心和指导改变了我的生活。

最后，要特别感谢我的家人。我的妻子克劳迪娅从不怀疑我的学习和进步能力，她对我花在这本书上的时间如此慷慨理解。我的孩子们，埃弗雷恩、伊格纳西奥和阿兰娜：我爱你们。

## Grig

我要感谢所有开源软件的创作者。没有他们，我们的工作将会更加单调乏味。还要感谢所有无私分享知识的博主。最后，我也要感谢这本书的合著者们。这真是一段愉快的旅程。
