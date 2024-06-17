# 第二十五章：经典 Python 的扩展和嵌入

*本章内容已经为本书的印刷版本进行了缩编。完整内容可在[在线版](https://oreil.ly/python-nutshell-25)中找到，详见“在线资料”。*

CPython 运行在一个可移植的、用 C 编写的虚拟机上。Python 的内置对象，比如数字、序列、字典、集合和文件，都是用 C 编写的，Python 标准库中也有几个模块是如此。现代平台支持动态加载库，文件扩展名为 *.dll*（Windows）、*.so*（Linux）和 *.dylib*（Mac），构建 Python 时会生成这些二进制文件。你可以用 C（或任何可以生成 C 可调用库的语言）编写自己的 Python 扩展模块，使用本章节介绍的 Python C API。有了这个 API，你可以生成和部署动态库，供 Python 脚本和交互会话后续使用，使用 **import** 语句导入，详见“导入语句”。

*扩展* Python 意味着构建模块，供 Python 代码**import**以访问模块提供的功能。*嵌入* Python 意味着在另一种语言编写的应用程序中执行 Python 代码。为了使这种执行有用，Python 代码反过来必须能够访问一些你的应用程序的功能。因此，实际上，嵌入暗示了一些扩展，以及一些特定于嵌入的操作。希望扩展 Python 的三个主要原因可以总结如下：

+   在较低级语言中重新实现一些功能（最初用 Python 编写），希望能获得更好的性能。

+   让 Python 代码访问由低级语言编写（或至少可从中调用）的库提供的一些现有功能

+   让 Python 代码访问一个正在将 Python 作为应用程序脚本语言嵌入到应用程序中的应用程序的一些现有功能

Python 的在线文档涵盖了嵌入和扩展；在那里，你可以找到深入的[教程](https://oreil.ly/BMl4L)和广泛的[参考手册](https://oreil.ly/OQXBK)。许多细节最好通过 Python 的广泛文档化的 C 源代码学习。下载 Python 的源代码分发包，并学习 Python 核心的源代码、C 编写的扩展模块以及为此目的提供的示例扩展。

# 在线资料

# 本章假设读者具备一些 C 的知识

虽然我们包括一些非 C 扩展选项，但要使用 C API 扩展或嵌入 Python，你必须了解 C 和/或 C++ 编程语言。我们在本书中不涵盖 C 和 C++，但有许多印刷和在线资源可供学习。本章的在线内容大多假设你至少有一些 C 的知识。

在[本章的在线版本](https://oreil.ly/python-nutshell-25)中，你会找到以下章节：

“用 Python 的 C API 扩展 Python”

包括参考表格和示例，用于创建 C 代码 Python 扩展模块，可以导入到你的 Python 程序中，展示如何编码和构建这些模块。本节包含两个完整示例：

+   一个实现自定义方法以操作字典的扩展

+   一个定义自定义类型的扩展

“不使用 Python 的 C API 扩展 Python”

讨论（或至少提到和链接到）几个工具和库，支持创建 Python 扩展，而无需直接使用 C 或 C++ 编程，¹ 包括第三方工具 [F2PY](https://oreil.ly/JrP_4), [SIP](https://oreil.ly/1l5ub), [CLIF](https://google.github.io/clif), [cppyy](https://cppyy.readthedocs.io), [pybind11](https://pybind11.readthedocs.io), [Cython](https://cython.org), [CFFI](https://cffi.readthedocs.io) 和 [HPy](https://hpyproject.org)，以及标准库模块 [ctypes](https://oreil.ly/xS4bC)。本节包含一个使用 Cython 创建扩展的完整示例。

“嵌入 Python”

包括参考表格和嵌入 Python 解释器到更大应用中的概念概述，使用 Python 的 C API 进行嵌入。

¹ 还有许多其他类似工具，但我们试图仅挑选最流行和有前景的工具。
