# 第一部分：TDD 和 Django 的基础

在这第一部分中，我将介绍*测试驱动开发*（TDD）的基础。我们将从头开始构建一个真实的 Web 应用程序，在每个阶段都先写测试。

我们将介绍使用 Selenium 进行功能测试，以及单元测试，并看到它们之间的区别。我将介绍 TDD 的工作流程，红/绿/重构。

我还会使用版本控制系统（Git）。我们将讨论何时以及如何进行提交，并将其与 TDD 和 Web 开发工作流程集成。

我们将使用 Django，这是 Python 世界中最流行的 Web 框架（可能）。我试图慢慢地、一步一步地介绍 Django 的概念，并提供大量进一步阅读的链接。如果你是 Django 的完全新手，我强烈建议你花时间去阅读它们。如果你感到有点迷茫，花几个小时阅读一下[官方 Django 教程](https://docs.djangoproject.com/en/4.2/intro/)，然后再回到本书。

在第一部分中，你还将会遇见测试山羊……​

# 小心复制和粘贴

如果你在使用电子版的书籍，当你阅读过程中自然会想要从书中复制和粘贴代码清单。但最好不要这样做：手动输入可以帮助你将信息记入肌肉记忆，并感觉更真实。你也难免会偶尔出现拼写错误，学会调试这些错误是很重要的事情。

除此之外，你会发现 PDF 格式的怪癖经常导致尝试复制/粘贴时出现奇怪的问题……​