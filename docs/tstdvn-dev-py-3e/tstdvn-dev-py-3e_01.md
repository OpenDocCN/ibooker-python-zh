# 先决条件和假设

这是我对你的假设以及你已经知道的内容的概述，以及你需要在你的电脑上准备并安装哪些软件。

# Python 3 和编程

我尽量以初学者为考虑对象来写这本书，但是如果你是新手程序员，我假设你已经学会了 Python 的基础知识。所以如果你还没有，先跑一遍 Python 初学者教程或者获取一本介绍性的书籍，比如[*The Quick Python Book*](https://www.manning.com/books/the-quick-python-book-third-edition) 或 [*Think Python*](https://greenteapress.com/thinkpython/html/index.xhtml)，又或者，只是为了好玩，[*Invent Your Own Computer Games with Python*](https://inventwithpython.com/#invent)，它们都是很好的入门材料。

如果你是一位有经验的程序员，但是对 Python 还很陌生，你应该能够顺利进行。Python 简单易懂。

你应该能够在 Mac、Windows 或 Linux 上跟着这本书。每个操作系统的详细安装说明如下。

###### 提示

这本书是在 Python 3.11 上测试的。如果你使用的是早期版本，你会发现我的命令输出列表中的东西看起来有轻微的差异（例如，追踪不会有 `^^^^^^` 标记错误位置），所以最好是升级，如果可能的话。

如果你考虑使用 [PythonAnywhere](http://www.pythonanywhere.com) 而不是本地安装的 Python，则在开始之前你应该去快速看一下 [Link to Come]。

无论如何，我希望你能够访问 Python，并知道如何从命令行启动它，以及如何编辑一个 Python 文件并运行它。如果你有任何疑问，再次查看我之前推荐的三本书籍。

# HTML 的工作原理

我还假设你对网络的工作原理有基本的了解 —— HTML 是什么，什么是 POST 请求等等。如果你对这些不确定，你需要找一些基本的 HTML 教程；在[*http://www.webplatform.org/*](http://www.webplatform.org/)上有几个。如果你能够弄清楚如何在你的电脑上创建一个 HTML 页面并在浏览器中查看它，并理解表单是什么以及它可能是如何工作的，那么你可能已经没问题了。

# Django

本书使用 Django 框架，这可能是 Python 世界中最成熟的 Web 框架。我写这本书的时候假设读者对 Django 没有任何先前的了解，但是如果你是 Python 和 Web 开发的新手，并且对测试也不熟悉，你可能会偶尔发现有一些主题和概念太多了，难以掌握。如果是这样的话，我建议你暂时离开这本书，去看一看 Django 的教程。[DjangoGirls](https://tutorial.djangogirls.org/) 是我知道的最好的、最适合初学者的教程。官方教程也非常适合有经验的程序员。

# JavaScript

本书的后半部分有一点 JavaScript。如果您不了解 JavaScript，请直到那时不要担心，如果您发现自己有些困惑，我会在那时推荐一些指南。

继续阅读安装说明。

# 必要的软件安装

除了 Python，您还需要：

Firefox 网页浏览器

Selenium 实际上可以驱动任何主流浏览器，但我选择了 Firefox，因为它受企业利益的控制最少。

Git 版本控制系统

这适用于任何平台，地址为[*http://git-scm.com/*](http://git-scm.com/)。在 Windows 上，它附带了 *Bash* 命令行，这是本书所需的。请参阅 “Windows 注”。

一个包含 Python 3.11、Django 4.2 和 Selenium 4 的虚拟环境

Python 的 virtualenv 和 pip 工具现在与 Python 捆绑在一起（它们以前并不总是如此，所以这是一个大好消息）。接下来是准备虚拟环境的详细说明。

## 安装 Firefox

Firefox 可在 Windows 和 MacOS 上下载安装，地址为[*https://www.mozilla.org/firefox/*](https://www.mozilla.org/firefox/)。在 Linux 上，你可能已经安装了它，但如果没有的话，你可以通过包管理器安装。

确保您拥有最新版本，以便“geckodriver”浏览器自动化模块可用。

# 设置您的虚拟环境

Python 虚拟环境（简称虚拟环境）是您为不同 Python 项目设置环境的方式。它允许您在每个项目中使用不同的软件包（例如，不同版本的 Django，甚至不同版本的 Python）。由于您不是系统范围内安装软件，因此意味着您不需要 root 权限。

让我们创建一个虚拟环境。我假设您在一个名为 *goat-book* 的文件夹中工作，但您可以根据喜好命名您的工作文件夹。但请确保虚拟环境的名称为 “.venv”。

```py
$ cd goat-book
$ py -3.11 -m venv .venv
```

在 Windows 上，`py` 可执行文件是不同 Python 版本的快捷方式。在 Mac 或 Linux 上，我们使用 `python3.11`：

```py
$ cd goat-book
$ python3.11 -m venv .venv
```

## 激活和停用虚拟环境

每当您阅读本书时，都应确保您的虚拟环境已“激活”。您可以根据提示符中是否显示 `(.venv)` 来确定您的虚拟环境是否处于活动状态。但您也可以通过运行 `which python` 检查当前是否为系统安装的 Python，还是虚拟环境的 Python。

激活虚拟环境的命令是在 Windows 上执行 `source .venv/Scripts/activate`，在 Mac/Linux 上执行 `source .venv/bin/activate`。停用的命令只是 `deactivate`。

这样尝试一下：

```py
$ source .venv/Scripts/activate
(.venv)$
(.venv)$ which python
/C/Users/harry/goat-book/.venv/Scripts/python
(.venv)$ deactivate
$
$ which python
/c/Users/harry/AppData/Local/Programs/Python/Python311-32/python
```

```py
$ source .venv/bin/activate
(.venv)$
(.venv)$ which python
/home/myusername/goat-book/.venv/bin/python
(.venv)$ deactivate
$
$ which python
/usr/bin/python
```

###### 提示

在编写本书时，请始终确保您的虚拟环境处于活动状态。请注意您的提示符中是否有 `(.venv)`，或者运行 `which python` 进行检查。

## 安装 Django 和 Selenium

我们将安装 Django 4.2 和最新的 Selenium²。请确保你的虚拟环境已激活！

```py
(.venv) $ pip install "django<4.3" "selenium"
Collecting django<4.3
  Downloading Django-4.2-py3-none-any.whl (8.0 MB)
     ---------------------------------------- 8.1/8.1 MB 7.6 MB/s eta 0:00:00
Collecting selenium
  Downloading selenium-4.9.0-py3-none-any.whl (6.5 MB)
     ---------------------------------------- 6.5/6.5 MB 6.3 MB/s eta 0:00:00
Installing collected packages: django, selenium
Successfully installed [...] django-4.2 [...] selenium-4.9.0 [...]
```

检查是否正常工作：

```py
(.venv) $ python -c "from selenium import webdriver; webdriver.Firefox()"
```

这应该会弹出一个 Firefox 网页浏览器，你需要关闭它。

###### 小提示

如果你看到一个错误，你需要在继续之前进行调试。在 Linux/Ubuntu 上，我遇到了 [这个 bug](https://github.com/mozilla/geckodriver/issues/2010)，你需要通过设置一个名为 `TMPDIR` 的环境变量来修复它。

## 当你 *不可避免* 地无法激活你的虚拟环境时，你可能会看到一些错误信息。

如果你是虚拟环境的新手——或者坦白说，即使你不是——在某个时候你肯定会忘记激活它，然后你会盯着一个错误信息。这时我经常遇到。这里是一些需要注意的事项：

```py
ModuleNotFoundError: No module named 'selenium'
```

或者：

```py
ImportError: No module named django.core.management
```

一如既往，留意命令提示符中的 `(.venv)`，只需快速输入 `source .venv/Scripts/activate` 或 `source .venv/bin/activate`，就可能是你重新运行它所需要的。

这里还有更多的错误信息，作为参考：

```py
bash: .venv/Scripts/activate: No such file or directory
```

这意味着你当前不在项目的正确目录中。尝试 `cd goat-book` 或类似的命令。

或者，如果你确定自己在正确的位置，可能遇到了一个旧版 Python 的 bug，导致无法安装与 Git-Bash 兼容的激活脚本。重新安装 Python 3，确保你有 3.6.3 及以上版本，然后删除并重新创建你的虚拟环境。

如果你看到类似的情况，那很可能是同一个问题，你需要升级 Python：

```py
bash: @echo: command not found
bash: .venv/Scripts/activate.bat: line 4:
      syntax error near unexpected token `(
bash: .venv/Scripts/activate.bat: line 4: `if not defined PROMPT ('
```

最后一个！如果你看到这个：

```py
'source' is not recognized as an internal or external command,
operable program or batch file.
```

这是因为你启动了默认的 Windows 命令提示符 `cmd`，而不是 Git-Bash。关闭它并打开后者。

编码愉快！ 

###### 注意

这些说明对你不起作用吗？或者你有更好的建议？请联系：obeythetestinggoat@gmail.com!

¹ 不过我不会推荐通过 Homebrew 安装 Firefox：`brew` 会将 Firefox 二进制文件放在一个奇怪的位置，这会让 Selenium 感到困惑。你可以绕过这个问题，但直接以正常方式安装 Firefox 更简单。

² 你可能会想知道为什么我没有提到 Selenium 的特定版本。这是因为 Selenium 不断更新，以跟上网页浏览器的变化，并且由于我们无法将浏览器固定在特定版本，我们最好使用最新的 Selenium。写作时是版本 4.9。
