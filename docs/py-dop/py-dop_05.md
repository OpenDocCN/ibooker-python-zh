# 第五章：包管理

常常，小脚本在有用性和重要性上不断增长，这造成了需要分享和分发其内容的需求。Python 库以及其他代码项目都需要打包。没有打包，分发代码将变得困难且脆弱。

过了概念验证阶段后，跟踪变更是很有帮助的，广告化变更类型（例如，在引入不兼容更新时），并提供一种让用户依赖特定版本的方式。即使在最简单的使用情况下，遵循一些（打包）指南也是有益的。至少，这应该包括跟踪变更日志和确定版本。

有几种跟随的包管理策略，了解其中一些最常用的可以让您采用最佳选项来解决问题。例如，通过 Python 包索引（PyPI）分发 Python 库可能比将其制作为 Debian 和 RPM 的系统包更容易。如果 Python 脚本需要在特定间隔运行，或者它是长时间运行的进程，那么与`systemd`一起工作的系统打包可能更好。

尽管`systemd`不是一个打包工具，但在依赖它管理进程和服务器启动顺序的系统上表现良好。学习如何通过几个`systemd`配置设置和一些打包处理来处理进程，是进一步增强 Python 项目能力的好方法。

原生 Python 打包工具在包公共托管实例（PyPI）上有一个。然而，对于 Debian 和 RPM 包，需要一些努力提供本地仓库。本章涵盖了几个工具，这些工具使创建和管理包仓库更加容易，包括 PyPI 的本地替代方案。

充分了解不同的打包策略以及健康实践，如适当的版本化和保持变更日志，可以在分发软件时提供稳定一致的体验。

# 打包为什么重要？

几个因素使软件打包成为项目中的重要特性（无论大小如何！）。通过变更日志跟踪版本和变更是提供新功能和错误修复洞察的好方法。版本化允许其他人更好地确定在项目内可能有效的工作内容。

在尝试识别问题和错误时，一个准确描述变更的变更日志是帮助确定系统故障潜在原因的无价工具。

版本化项目，描述变更在变更日志中，提供其他人安装和使用项目的方式，需要纪律和辛勤工作。然而，在分发、调试、升级甚至卸载时，好处显著。

## 当不需要打包时

有时候根本不需要将项目分发给其他系统。Ansible Playbooks 通常从一个服务器运行，以管理网络中的其他系统。在像 Ansible 这样的情况下，遵循版本控制并保持变更日志可能就足够了。

像 Git 这样的版本控制系统通过使用标签使这一切变得容易。在 Git 中标记仍然是有用的，如果项目确实需要进行打包，因为大多数工具可以使用标签（只要标签表示一个版本）来生成包。

最近，进行了一个长时间的调试会话，以确定为什么一个大型软件项目的安装程序停止工作。突然间，依赖于安装程序完成部署的一个小 Python 工具的所有功能测试都失败了。安装程序确实有版本，并且保持这些版本与版本控制同步，但却完全没有任何变更日志来解释最近的更改将会破坏现有的 API。为了找出问题，我们不得不浏览所有最近的提交，以确定可能的问题所在。

浏览几个提交应该不难，但尝试在有四千多个提交的项目中做到这一点！找到问题后，开了两个票：一个解释了错误，另一个要求变更日志。

# 打包指南

在打包之前，有几件事情是值得考虑的，以使整个过程尽可能顺利。即使你不打算打包产品，这些指南也有助于改进项目的整体情况。

###### 注意

版本控制系统始终准备好进行打包。

## 描述性版本控制

有许多版本软件的方法，但遵循一个知名模式是一个好主意。Python 开发者指南对版本控制有一个清晰的定义。

版本控制模式应该非常灵活，同时要保持一致性，以便安装工具可以理解并相应地优先考虑（例如，稳定版本优于测试版本）。在其最纯粹的形式下，以及大多数 Python 包中，通常使用以下两种变体：`major.minor` 或 `major.minor.micro`。

有效的版本看起来像：

+   `0.0.1`

+   `1.0`

+   `2.1.1`

###### 注意

尽管 Python 开发者指南描述了许多变体，但集中在较简单的形式上（如上所列）。它们足以生成包，同时也遵循大多数系统和本地 Python 包的指南。

发布的一个通常被接受的格式是 `major.minor.micro`（也被 [语义化版本控制](https://semver.org) 方案使用）：

+   `major` 用于不兼容的更改。

+   `minor` 添加了向后兼容的功能。

+   `micro` 添加了向后兼容的错误修复。

根据上述列出的版本，您可以推断出对版本为 `1.0.0` 的应用程序的依赖性可能会在版本 `2.0.0` 中中断。

一旦做出发布决定，确定版本号就变得很容易。假设当前开发中项目的已发布版本为 `1.0.0`，这意味着可能出现以下结果：

+   如果发布具有向后不兼容的更改，则版本号为：`2.0.0`

+   如果发布添加的功能不会破坏兼容性，则版本号为 `1.1.0`

+   如果发布用于修复不会破坏兼容性的问题，则版本号为 `1.0.1`

一旦遵循了某个模式，发布过程立即变得清晰明了。尽管希望所有软件都遵循类似的模式，但一些项目有完全不同的自己的模式。例如，[Ceph](https://ceph.com) 项目使用以下模式：`major.[0|1|2].minor`

+   `major` 表示一个主要发布版本，尽管不一定会破坏向后兼容性。

+   `0`、`1` 或 `2` 分别表示开发版本、发布候选版或稳定版本。

+   `minor` 仅用于修复错误，永远不用于功能添加。

该模式意味着 `14.0.0` 是一个开发版本，而 `14.2.1` 是主要发布版本（本例中为 `14`）的稳定版本修复版本。

## 变更日志

正如我们已经提到的，重要的是要跟踪发布和它们在版本号上的含义。一旦选择了版本控制方案，保持变更日志并不难。虽然它可以是一个单一的文件，但大型项目倾向于将其拆分为一个目录中的多个小文件。最佳实践是使用简单且描述性强、易于维护的格式。

下面的例子是生产中一个 Python 工具的*变更日志*文件的一个实际部分：

```py
1.1.3
^^^^^
22-Mar-2019

* No code changes - adding packaging files for Debian

1.1.2
^^^^^
13-Mar-2019

* Try a few different executables (not only ``python``) to check for a working
  one, in order of preference, starting with ``python3`` and ultimately falling
  back to the connection interpreter
```

该示例提供了四个重要的信息：

1.  最新发布的版本号

1.  最新版本是否向后兼容

1.  上一个版本的发布日期

1.  包含在发布中的变更

该文件不需要特定的格式，只要保持一致性和信息性即可。一个合适的变更日志可以在很少的工作量下提供多个信息。尝试自动化写每个发布的变更日志的任务虽然诱人，但我们建议不要完全自动化处理：没有什么能比一个精心编写的关于修复错误或添加功能的条目更好。

一个质量不佳的自动化变更日志是使用所有版本控制提交的变更。这不是一个好的做法，因为你可以通过列出提交来获得相同的信息。

# 选择策略

理解所需的分发类型和可用的基础设施服务有助于确定使用何种类型的打包。为其他 Python 项目扩展功能的纯 Python 库适合作为本地 Python 包，托管在 Python 软件包索引（PyPI）或本地索引上。

独立脚本和长时间运行的进程是系统软件包（如 RPM 或 Debian）的良好候选者，但最终取决于可用的系统类型以及是否可能托管（和管理）存储库。对于长时间运行的进程，打包可以有规则来配置`systemd`单元，使其作为可控制的进程可用。`systemd`允许优雅地处理启动、停止或重启操作。这些是使用本地 Python 打包无法实现的功能。

一般来说，脚本或进程需要与系统交互得越多，就越适合系统软件包或容器。在编写仅限 Python 的脚本时，传统的 Python 打包是正确的选择。

###### 注意

没有硬性要求选择哪种策略。这取决于情况！选择最适合分发的环境（例如，如果服务器是 CentOS，则选择 RPM）。不同类型的打包并不是互斥的；一个项目可以同时提供多种打包格式。

# 打包解决方案

本节介绍如何创建软件包并进行托管的详细信息。

为简化代码示例，假设一个名为`hello-world`的小型 Python 项目具有以下结构：

```py
hello-world
└── hello_world
    ├── __init__.py
    └── main.py

1 directory, 2 files
```

项目有一个名为`hello-world`的顶层目录和一个子目录（`hello_world`），其中包含两个文件。根据打包选择的不同，需要不同的文件来创建软件包。

## 本地 Python 打包

到目前为止，使用本地 Python 打包工具和托管（通过 PyPI）是最简单的解决方案。与其他打包策略一样，该项目需要一些`setuptools`使用的文件。

###### 提示

一个简单的方法来获取虚拟环境是创建一个`bash`或`zsh`的别名，它会 cd 到目录并激活环境，就像这样：`alias sugar="source ~/.sugar/bin/activate && cd ~/src/sugar"`

要继续，请创建一个新的虚拟环境，然后激活：

```py
$ python3 -m venv /tmp/packaging
$ source /tmp/packaging/bin/activate
```

###### 注意

`setuptools`是生成本地 Python 软件包的要求。它是一组工具和助手，用于创建和分发 Python 软件包。

一旦虚拟环境激活，存在以下依赖关系：

`setuptools`

一组用于打包的实用工具

`twine`

一个用于注册和上传软件包的工具

通过运行以下命令来安装它们：

```py
$ pip install setuptools twine
```

###### 注意

要查找已安装内容的一个非常简单的方法是使用`IPython`和以下代码片段，将所有 Python 软件包列出为`JSON`数据结构：

```py
In [1]: !pip list --format=json

[{"name": "appnope", "version": "0.1.0"},
 {"name": "astroid", "version": "2.2.5"},
 {"name": "atomicwrites", "version": "1.3.0"},
 {"name": "attrs", "version": "19.1.0"}]
```

### 软件包文件

要生成本地 Python 软件包，我们必须添加一些文件。为了保持简单，专注于生成软件包所需的最少文件。描述软件包给`setuptools`的文件名为*setup.py*，位于顶层目录。对于示例项目，文件看起来是这样的：

```py
from setuptools import setup, find_packages

setup(
    name="hello-world",
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    url="example.com",
    description="A hello-world example package",
    packages=find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)
```

*setup.py*文件将从`setuptools`模块导入两个助手：`setup`和`find_packages`。`setup`函数需要关于软件包的详细描述。`find_packages`函数是一个实用工具，用于自动检测 Python 文件的位置。此外，该文件导入了描述软件包特定方面的*classifiers*，如许可证、支持的操作系统和 Python 版本。这些*classifiers*称为*trove classifiers*，在[Python 包索引](https://pypi.org/classifiers)上有关于其他可用分类器的详细描述。详细描述有助于软件包上传到 PyPI 后被发现。

只需添加这一个文件，我们就能够生成一个软件包，这种情况下是*源分发*软件包。如果没有*README*文件，在运行命令时会出现警告。为了防止这种情况，请使用以下命令在顶级目录中添加一个空的*README*文件：`touch README`。

项目目录的内容应该如下所示：

```py
hello-world
├── hello_world
│   ├── __init__.py
│   └── main.py
└── README
└── setup.py

1 directory, 4 files
```

要从中生成*源分发*，请运行以下命令：

```py
python3 setup sdist
```

输出应该类似于以下内容：

```py
$ python3 setup.py sdist
running sdist
running egg_info
writing hello_world.egg-info/PKG-INFO
writing top-level names to hello_world.egg-info/top_level.txt
writing dependency_links to hello_world.egg-info/dependency_links.txt
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
running check
creating hello-world-0.0.1
creating hello-world-0.0.1/hello_world
creating hello-world-0.0.1/hello_world.egg-info
copying files to hello-world-0.0.1...
copying README -> hello-world-0.0.1
copying setup.py -> hello-world-0.0.1
copying hello_world/__init__.py -> hello-world-0.0.1/hello_world
copying hello_world/main.py -> hello-world-0.0.1/hello_world
Writing hello-world-0.0.1/setup.cfg
Creating tar archive
removing 'hello-world-0.0.1' (and everything under it)
```

在项目的顶级目录，有一个名为*dist*的新目录；它包含*源分发*：一个名为*hello-world-0.0.1.tar.gz*的文件。如果我们检查目录的内容，它已经再次改变：

```py
hello-world
├── dist
│   └── hello-world-0.0.1.tar.gz
├── hello_world
│   ├── __init__.py
│   └── main.py
├── hello_world.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   └── top_level.txt
├── README
└── setup.py

3 directories, 9 files
```

新创建的*tar.gz*文件是一个可安装的软件包！现在可以将此软件包上传到 PyPI，以便其他人直接从中安装。通过遵循版本模式，安装程序可以请求特定版本（在本例中为`0.0.1`），并且通过传递给`setup()`函数的额外元数据，其他工具可以发现它并显示关于它的信息，如作者、描述和版本。

Python 安装工具`pip`可以直接用于安装*tar.gz*文件。要试试，请使用文件路径作为参数：

```py
$ pip install dist/hello-world-0.0.1.tar.gz
Processing ./dist/hello-world-0.0.1.tar.gz
Building wheels for collected packages: hello-world
  Building wheel for hello-world (setup.py) ... done
Successfully built hello-world
Installing collected packages: hello-world
Successfully installed hello-world-0.0.1
```

### Python 包索引

Python 包索引（PyPI）是一个 Python 软件的仓库，允许用户托管 Python 软件包并从中安装。它由社区维护，并在[Python 软件基金会](https://www.python.org/psf)的赞助和捐款下运行。

###### 注意

此部分需要为 PyPI 的*测试实例*注册。确保您已经有了账户或者[在线注册](https://oreil.ly/lyVVx)。您需要账户的用户名和密码来上传软件包。

在示例的*setup.py*文件中，示例电子邮件地址包含一个占位符。如果要将软件包发布到索引，需要更新为与 PyPI 项目所有者相同的电子邮件地址。更新其他字段，如`author`，`url`和`description`，以更准确地反映正在构建的项目。

为了确保一切正常运作，并且避免*推向生产*，通过将包上传到 PyPI 的测试实例来测试包。这个测试实例的行为与生产环境相同，并验证包的正确功能。

`setuptools` 和 *setup.py* 文件是将包上传到 PyPI 的传统方法。一种新方法叫做 `twine`，可以简化操作。

在本节开始时，`twine` 已安装在虚拟环境中。接下来，可以使用它将包上传到 PyPI 的测试实例。以下命令上传 *tar.gz* 文件，并提示输入用户名和密码：

```py
$ twine upload --repository-url https://test.pypi.org/legacy/ \
  dist/hello-world-0.0.1.tar.gz
Uploading distributions to https://test.pypi.org/legacy/
Enter your username:
Enter your password:
```

要测试包是否安装成功，我们可以尝试使用 `pip` 安装它：

```py
$ python3 -m pip install --index-url https://test.pypi.org/simple/ hello-world
```

命令看起来在 PyPI URL 中有一个空格，但索引 URL 以 `/simple/` 结尾，并且 `hello-world` 是另一个参数，指示要安装的 Python 包的名称。

对于实际的生产发布，需要存在一个帐户或[创建](https://pypi.org/account/register)一个帐户。与上传到测试实例相同的步骤，包括验证，也适用于*真实*的 PyPI。

较旧的 Python 打包指南可能会提到如下命令：

```py
 $ python setup.py register
 $ python setup.py upload
```

这些方法仍然有效，并且是 `setuptools` 一套工具中的一部分，用于打包并上传项目到包索引中。然而，`twine` 提供了通过 HTTPS 进行安全认证，并支持使用 `gpg` 签名。Twine 可以工作，即使 `python setup.py upload` 无法工作，并最终提供了在上传到索引之前测试包的方法。

最后要指出的一点是，创建一个 `Makefile` 并在其中放入一个 `make` 命令可能会有所帮助，自动部署项目并为您构建文档。以下是一个示例：

```py
deploy-pypi:
  pandoc --from=markdown --to=rst README.md -o README.rst
  python setup.py check --restructuredtext --strict --metadata
  rm -rf dist
  python setup.py sdist
  twine upload dist/*
  rm -f README.rst
```

### 托管内部包索引

在某些情况下，托管内部 PyPI 可能更可取。

Alfredo 曾经工作的公司拥有不应公开的私有库，因此必须托管一个 PyPI 实例是一个要求。然而，托管也有其注意事项。所有依赖项及其版本都必须存在于实例中；否则，安装可能会失败。安装程序无法同时从不同来源获取依赖项！不止一次，新版本缺少组件，因此必须上传该包以完成正确的安装。

如果包 *A* 托管在内部，并且依赖于包 *B* 和 *C*，那么所有这三个包（及其所需版本）都需要在同一个实例中存在。

内部 PyPI 可以加快安装速度，可以保持包的私密性，并且从本质上来说，并不难实现。

###### 注意

一个强烈推荐的用于托管内部 PyPI 的功能全面的工具是 `devpi`。它具有镜像、分段、复制和 Jenkins 集成等功能。[项目文档](http://doc.devpi.net)提供了很好的示例和详细信息。

首先，创建一个名为`pypi`的新目录，以便可以创建一个适合托管包的正确结构，然后创建一个名为我们示例包（`hello-world`）的子目录。子目录的名称即是包的名称：

```py
$ mkdir -p pypi/hello-world
$ tree pypi
pypi
└── hello-world

1 directory, 0 files
```

现在将*tar.gz*文件复制到*hello-world*目录中。该目录结构的最终版本应如下所示：

```py
$ tree pypi
pypi
└── hello-world
    └── hello-world-0.0.1.tar.gz

1 directory, 1 file
```

下一步是创建一个启用了自动索引的 Web 服务器。Python 自带一个内置的 Web 服务器，足以尝试这一功能，并且默认情况下已启用自动索引！切换到包含`hello-world`包的*pypi*目录，并启动内置的 Web 服务器：

```py
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

在新的终端会话中，创建一个临时虚拟环境，以尝试从本地 PyPI 实例安装`hello-world`包。激活它，最后通过将`pip`指向自定义本地 URL 来尝试安装它：

```py
$ python3 -m venv /tmp/local-pypi
$ source /tmp/local-pypi/bin/activate
(local-pypi) $ pip install -i http://localhost:8000/ hello-world
Looking in indexes: http://localhost:8000/
Collecting hello-world
  Downloading http://localhost:8000/hello-world/hello-world-0.0.1.tar.gz
Building wheels for collected packages: hello-world
  Building wheel for hello-world (setup.py) ... done
Successfully built hello-world
Installing collected packages: hello-world
Successfully installed hello-world-0.0.1
```

在运行`http.server`模块的会话中，应该有一些日志，展示安装程序为检索`hello-world`包所做的所有请求：

```py
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 [09:58:37] "GET / HTTP/1.1" 200 -
127.0.0.1 [09:59:39] "GET /hello-world/ HTTP/1.1" 200 -
127.0.0.1 [09:59:39] "GET /hello-world/hello-world-0.0.1.tar.gz HTTP/1.1" 200
```

生产环境需要一个性能更好的 Web 服务器。在这个例子中，为了简单起见，使用`http.server`模块，但它不能处理同时的多个请求或扩展。

###### 提示

在没有像`devpi`这样的工具的情况下构建本地索引时，有一个定义明确的规范，其中包括规范化名称的目录结构描述。此规范可以在[PEP 503](https://oreil.ly/sRcAe)中找到。

## Debian 打包

如果目标是 Debian（或基于 Debian 的发行版，如 Ubuntu）来分发项目，则需要额外的文件。了解这些文件是什么，以及 Debian 打包工具如何使用它们，可以改善生成可安装的`.deb`软件包和解决问题的过程。

一些这些纯文本文件需要非常严格的格式，如果格式稍有不正确，打包将无法安装。

###### 注意

本节假设打包是在 Debian 或基于 Debian 的发行版中进行的，因此更容易安装和使用所需的打包工具。

### 包文件

Debian 打包需要一个包含几个文件的*debian*目录。为了缩小生成软件包所需内容的范围，大部分可用选项被跳过，例如在完成构建之前运行测试套件或声明多个 Python 版本。

创建*debian*目录，其中包含所有必需的文件。最终，`hello-world`项目结构应如下所示：

```py
$ tree
.
├── debian
├── hello_world
│   ├── __init__.py
│   └── main.py
├── README
└── setup.py

2 directories, 4 files
```

###### 注意

注意该目录包括本地 Python 打包部分的*setup.py*和*README*文件。这是必需的，因为 Debian 工具使用它们来生成`.deb`软件包。

#### 变更日志文件

如果手动操作这个文件会很复杂。当文件格式不正确时产生的错误不容易调试。大多数 Debian 打包工作流依赖于`dch`工具来增强调试能力。

我之前忽略了我的建议，试图手动创建这个文件。最后我浪费了时间，因为错误报告不是很好，而且很难发现问题。以下是在 *changelog* 文件中导致问题的条目示例：

```py
--Alfredo Deza <alfredo@example.com> Sat, 11 May 2013 2:12:00 -0800
```

出现以下错误：

```py
parsechangelog/debian: warning: debian/changelog(l7): found start of entry where
  expected more change data or trailer
```

你能找到解决方法吗？

```py
-- Alfredo Deza <alfredo@example.com> Sat, 11 May 2013 2:12:00 -0800
```

在破折号和我的名字之间有一个空格是问题的根源。避免自己的麻烦，使用`dch`。该工具是`devscripts`软件包的一部分：

```py
$ sudo apt-get install devscripts
```

`dch`命令行工具有很多选项，通过阅读其文档（主页内容详尽）会很有帮助。我们将运行它来首次创建变更日志（这需要一次性使用`--create`标志）。在运行之前，请导出您的全名和电子邮件，以便它们出现在生成的文件中：

```py
$ export DEBEMAIL="alfredo@example.com"
$ export DEBFULLNAME="Alfredo Deza"
```

现在运行`dch`以生成变更日志：

```py
$ dch --package "hello-world" --create -v "0.0.1" \
  -D stable "New upstream release"
```

新创建的文件应该类似于这样：

```py
hello-world (0.0.1) stable; urgency=medium

  * New upstream release

 -- Alfredo Deza <alfredo@example.com>  Thu, 11 Apr 2019 20:28:08 -0400
```

###### 注意

Debian 变更日志是专用于 Debian 打包的。当格式不符或需要更新其他信息时，项目可以单独保留一个变更日志。许多项目将 Debian 的 *changelog* 文件作为单独的 Debian 专用文件。

#### 控制文件

这个文件定义了软件包的名称、描述以及构建和运行项目所需的任何依赖项。它也有严格的格式，但不需要经常更改（不像 *changelog*）。该文件确保需要 Python 3 并遵循 Debian 的 Python 命名指南。

###### 注意

在从 Python 2 迁移到 Python 3 的过渡中，大多数发行版都采用了以下用于 Python 3 软件包的模式：`python3-{package name}`。

添加依赖项、命名约定和简短描述后，文件应如下所示：

```py
Source: hello-world
Maintainer: Alfredo Deza <alfredo@example.com>
Section: python
Priority: optional
Build-Depends:
 debhelper (>= 11~),
 dh-python,
 python3-all
 python3-setuptools
Standards-Version: 4.3.0

Package: python3-hello-world
Architecture: all
Depends: ${misc:Depends}, ${python3:Depends}
Description: An example hello-world package built with Python 3
```

#### 其他必需的文件

还需要一些其他文件来生成 Debian 软件包。它们大多数只有几行，并且不经常更改。

*rules*文件是一个可执行文件，告诉 Debian 如何运行以生成软件包；在这种情况下，应如下所示：

```py
#!/usr/bin/make -f

export DH_VERBOSE=1

export PYBUILD_NAME=remoto

%:
    dh $@ --with python3 --buildsystem=pybuild
```

*compat* 文件设置对应的 `debhelper`（另一个打包工具）兼容性，推荐在这里设置为 `10`。如果出现错误消息指出需要更高的值，您可能需要检查是否需要更高版本：

```py
$ cat compat
10
```

缺少许可证可能会导致构建过程无法工作，明确声明许可证是个好主意。此特定示例使用 MIT 许可证，应在 *debian/copyright* 中如下所示：

```py
Format: http://www.debian.org/doc/packaging-manuals/copyright-format/1.0
Upstream-Name: hello-world
Source: https://example.com/hello-world

Files: *
Copyright: 2019 Alfredo Deza
License: Expat

License: Expat
  Permission is hereby granted, free of charge, to any person obtaining a
  copy of this software and associated documentation files (the "Software"),
  to deal in the Software without restriction, including without limitation
  the rights to use, copy, modify, merge, publish, distribute, sublicense,
  and/or sell copies of the Software, and to permit persons to whom the
  Software is furnished to do so, subject to the following conditions:
  .
  The above copyright notice and this permission notice shall be included in
  all copies or substantial portions of the Software.
  .
  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
  THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
  FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
  DEALINGS IN THE SOFTWARE.
```

最后，在将所有这些新文件添加到`debian`目录之后，`hello-world`项目如下所示：

```py
.
├── debian
│   ├── changelog
│   ├── compat
│   ├── control
│   ├── copyright
│   └── rules
├── hello_world
│   ├── __init__.py
│   └── main.py
├── README
└── setup.py

2 directories, 9 files
```

### 生成二进制文件

要生成二进制文件，请使用 `debuild` 命令行工具。对于此示例项目，包仍未签名（签名过程需要 GPG 密钥），并且 `debuild` 文档使用的示例允许跳过签名。从源代码树内部运行该脚本以仅构建二进制包。此命令适用于 `hello-world` 项目（此处显示了截断版本）：

```py
$ debuild -i -us -uc -b
...
dpkg-deb: building package 'python3-hello-world'
in '../python3-hello-world_0.0.1_all.deb'.
...
 dpkg-genbuildinfo --build=binary
 dpkg-genchanges --build=binary >../hello-world_0.0.1_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source -i --after-build hello-world-debian
dpkg-buildpackage: info: binary-only upload (no source included)
Now running lintian hello-world_0.0.1_amd64.changes ...
E: hello-world changes: bad-distribution-in-changes-file stable
Finished running lintian.
```

现在上层目录应该存在一个名为 *python3-hello-world_0.0.1_all.deb* 的文件。lintian 调用（一个 Debian 打包工具）在最后报告 *changelog* 文件的发行版本无效，这没关系，因为我们不是针对特定的单个发行版（例如 Debian Buster）。相反，我们正在构建一个包，它很可能在任何符合依赖关系（仅 Python 3，在本例中）的 Debian 基础发行版上安装。

### Debian 仓库

有许多工具可以自动化管理 Debian 仓库，但了解如何创建一个是很有用的（阿尔弗雷多甚至帮助[开发了一个](https://oreil.ly/hJMgY)，适用于 RPM 和 Debian！）。继续之前，请确保之前创建的二进制包在已知位置可用：

```py
$ mkdir /opt/binaries
$ cp python3-hello-world_0.0.1_all.deb /opt/binaries/
```

对于这一部分，需要安装 `reprepro` 工具：

```py
$ sudo apt-get install reprepro
```

在系统的某个位置创建一个新目录以保存包。此示例使用 */opt/repo*。仓库的基本配置需要一个名为 `distributions` 的文件，描述其内容并如下所示：

```py
Codename: sid
Origin: example.com
Label: example.com
Architectures: amd64 source
DscIndices: Sources Release .gz .bz2
DebIndices: Packages Release . .gz .bz2
Components: main
Suite: stable
Description: example repo for hello-world package
Contents: .gz .bz2
```

将此文件保存在 */opt/repo/conf/distributions*。创建另一个目录来保存实际的仓库：

```py
$ mkdir /opt/repo/debian/sid
```

要创建仓库，请指示 `reprepro` 使用先前创建的 *distributions* 文件，并指定基本目录为 */opt/repo/debian/sid*。最后，将之前创建的二进制文件添加为 Debian sid 发行版的目标：

```py
$ reprepro --confdir /opt/repo/conf/distributions -b /opt/repo/debian/sid \
  -C main includedeb sid /opt/binaries/python3-hello-world_0.0.1_all.deb
Exporting indices...
```

此命令为 Debian sid 发行版创建了仓库！这个命令可以用于不同的基于 Debian 的发行版，如 Ubuntu Bionic。要做到这一点，只需将 `sid` 替换为 `bionic`。

现在仓库已创建，下一步是确保其按预期工作。在生产环境中，像 Apache 或 Nginx 这样的稳健 Web 服务器是一个不错的选择，但为了测试，请使用 Python 的 `http.server` 模块。切换到包含仓库的目录，并启动服务器：

```py
$ cd /opt/repo/debian/sid
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Aptitude（或 `apt`，Debian 包管理器）需要一些配置来知道包的新位置。这个配置是一个简单的文件，其中只有一行指向我们仓库的 URL 和组件。在 */etc/apt/sources.lists.d/hello-world.list* 创建一个文件，应如下所示：

```py
$ cat /etc/apt/sources.list.d/hello-world.list
deb [trusted=yes] http://localhost:8000/ sid main
```

`[trusted=yes]` 配置告诉 `apt` 不强制要求已签名的包。在正确签名的仓库上，这一步骤是不必要的。

添加文件后，请更新 `apt` 以使其识别新位置，并查找（并安装）`hello-world` 包：

```py
$ sudo apt-get update
Ign:1 http://localhost:8000 sid InRelease
Get:2 http://localhost:8000 sid Release [2,699 B]
Ign:3 http://localhost:8000 sid Release.gpg
Get:4 http://localhost:8000 sid/main amd64 Packages [381 B]
Get:5 http://localhost:8000 sid/main amd64 Contents (deb) [265 B]
Fetched 3,345 B in 1s (6,382 B/s)
Reading package lists... Done
```

搜索 `python3-hello-world` 软件包提供在配置 `reprepro` 时添加到 *distributions* 文件的描述：

```py
$ apt-cache search python3-hello-world
python3-hello-world - An example hello-world package built with Python 3
```

安装和删除软件包应该没有问题：

```py
$ sudo apt-get install python3-hello-world
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  python3-hello-world
0 upgraded, 1 newly installed, 0 to remove and 48 not upgraded.
Need to get 2,796 B of archives.
Fetched 2,796 B in 0s (129 kB/s)
Selecting previously unselected package python3-hello-world.
(Reading database ... 242590 files and directories currently installed.)
Preparing to unpack .../python3-hello-world_0.0.1_all.deb ...
Unpacking python3-hello-world (0.0.1) ...
Setting up python3-hello-world (0.0.1) ...
$ sudo apt-get remove --purge python3-hello-world
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  python3-hello-world*
0 upgraded, 0 newly installed, 1 to remove and 48 not upgraded.
After this operation, 19.5 kB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 242599 files and directories currently installed.)
Removing python3-hello-world (0.0.1) ...
```

## RPM 打包

就像在 Debian 打包时一样，在 RPM 中工作时，必须已经完成本地 Python 打包。应该可以使用 *setup.py* 文件生成 Python 软件包。然而，与 Debian 不同的是，RPM 打包只需一个文件即可：*spec* 文件。如果目标是像 CentOS 或 Fedora 这样的发行版，那么 RPM 软件包管理器（前身是 Red Hat 软件包管理器）是最佳选择。

### 规范文件

在其最简单的形式中，*spec* 文件（本示例中命名为 *hello-world.spec*）并不难理解，大多数部分都是不言自明的。甚至可以通过使用 `setuptools` 来生成它：

```py
$ python3 setup.py bdist_rpm --spec-only
running bdist_rpm
running egg_info
writing hello_world.egg-info/PKG-INFO
writing dependency_links to hello_world.egg-info/dependency_links.txt
writing top-level names to hello_world.egg-info/top_level.txt
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
writing 'dist/hello-world.spec'
```

*dist/hello-world.spec* 输出文件应该类似于这样：

```py
%define name hello-world
%define version 0.0.1
%define unmangled_version 0.0.1
%define release 1

Summary: A hello-world example pacakge
Name: %{name}
Version: %{version}
Release: %{release}
Source0: %{name}-%{unmangled_version}.tar.gz
License: MIT
Group: Development/Libraries
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-buildroot
Prefix: %{_prefix}
BuildArch: noarch
Vendor: Example Author <author@example.com>
Url: example.com

%description
A Python3 hello-world package

%prep
%setup -n %{name}-%{unmangled_version} -n %{name}-%{unmangled_version}

%build
python3 setup.py build

%install
python3 setup.py install --single-version-externally-managed -O1 \
--root=$RPM_BUILD_ROOT --record=INSTALLED_FILES

%clean
rm -rf $RPM_BUILD_ROOT

%files -f INSTALLED_FILES
%defattr(-,root,root)
```

尽管看起来很简单，但已经存在潜在问题：版本是

`set   `setuptools` 的集成非常有利，允许进一步修改该文件（如果需要），并将其复制到项目的根目录以便永久存放。一些项目使用基础模板，在构建过程中用填充方式生成规范文件。如果遵循严格的发布工作流程，这个过程非常有用。在 [Ceph 项目](https://ceph.com) 的情况下，通过版本控制（Git）标记发布，并使用标签在 `Makefile` 中应用于模板。值得注意的是，还存在其他方法可以进一步自动化这个过程。

###### 提示

生成 *spec* 文件并不总是有用，因为某些部分可能需要硬编码以遵循某些发行规则或特定依赖项，这些依赖项不在生成的文件中。在这种情况下，最好生成一次，进一步配置并最终保存它，使 *spec* 文件成为项目的正式一部分。

### 生成二进制文件

有几种不同的工具可以生成 RPM 二进制文件；其中一种特别的工具是 `rpmbuild` 命令行工具：

```py
$ sudo yum install rpm-build
```

###### 注意

命令行工具是 `rpmbuild`，但软件包名是 `rpm-build`，因此请确保在终端中可用 `rpmbuild`（命令行工具）。

`rpmbuild` 需要一个目录结构来创建二进制文件。在创建这些目录之后，*source* 文件（由 `setuptools` 生成的 *tar.gz* 文件）需要存在于 *SOURCES* 目录中。这就是应该创建的结构以及完成后的样子：

```py
$ mkdir -p /opt/repo/centos/{SOURCES,SRPMS,SPECS,RPMS,BUILD}
$ cp dist/hello-world-0.0.1.tar.gz /opt/repo/centos/SOURCES/
$ tree /opt/repo/centos
/opt/repo/centos
├── BUILD
├── BUILDROOT
├── RPMS
├── SOURCES
│   └── hello-world-0.0.1.tar.gz
├── SPECS
└── SRPMS

6 directories, 1 file
```

目录结构始终是必需的，默认情况下，`rpmbuild`需要在主目录中。为了保持隔离，使用不同位置（在*/opt/repo/centos*中）。这个过程意味着配置`rpmbuild`使用此目录代替。此过程使用`-ba`标志生成二进制和*源*软件包（输出已缩写）：

```py
$ rpmbuild -ba --define "_topdir /opt/repo/centos"  dist/hello-world.spec
...
Executing(%build): /bin/sh -e /var/tmp/rpm-tmp.CmGOdp
running build
running build_py
creating build
creating build/lib
creating build/lib/hello_world
copying hello_world/main.py -> build/lib/hello_world
copying hello_world/__init__.py -> build/lib/hello_world
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.CQgOKD
+ python3 setup.py install --single-version-externally-managed \
-O1 --root=/opt/repo/centos/BUILDROOT/hello-world-0.0.1-1.x86_64
running install
writing hello_world.egg-info/PKG-INFO
writing dependency_links to hello_world.egg-info/dependency_links.txt
writing top-level names to hello_world.egg-info/top_level.txt
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
running install_scripts
writing list of installed files to 'INSTALLED_FILES'
Processing files: hello-world-0.0.1-1.noarch
Provides: hello-world = 0.0.1-1
Wrote: /opt/repo/centos/SRPMS/hello-world-0.0.1-1.src.rpm
Wrote: /opt/repo/centos/RPMS/noarch/hello-world-0.0.1-1.noarch.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.gcIJgT
+ umask 022
+ cd /opt/repo/centos//BUILD
+ cd hello-world-0.0.1
+ rm -rf /opt/repo/centos/BUILDROOT/hello-world-0.0.1-1.x86_64
+ exit 0
```

在*/opt/repo/centos*的目录结构将有许多新文件，但我们只对具有`noarch` RPM 的文件感兴趣：

```py
$ tree /opt/repo/centos/RPMS
/opt/repo/centos/RPMS
└── noarch
    └── hello-world-0.0.1-1.noarch.rpm

1 directory, 1 file
```

`noarch` RPM 是一个可安装的 RPM 包！该工具还生成了其他有用的可以发布的软件包（例如查看*/opt/repo/centos/SRPMS*）。

### RPM 存储库

要创建 RPM 存储库，请使用`createrepo`命令行工具。它从给定目录中找到的二进制文件生成存储库元数据（基于 XML 的 RPM 元数据）。在此部分中，创建（并托管）`noarch`二进制文件：

```py
$ sudo yum install createrepo
```

在生成`noarch`包的相同位置创建存储库，或者使用新的（干净的）目录。如有必要，创建新的二进制文件。一旦完成，包括如下内容：

```py
$ mkdir -p /var/www/repos/centos
$ cp -r /opt/repo/centos/RPMS/noarch /var/www/repos/centos
```

要创建元数据，请运行`createrepo`工具：

```py
$ createrepo -v /var/www/repos/centos/noarch
Spawning worker 0 with 1 pkgs
Worker 0: reading hello-world-0.0.1-1.noarch.rpm
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Starting other db creation: Thu Apr 18 09:13:35 2019
Ending other db creation: Thu Apr 18 09:13:35 2019
Starting filelists db creation: Thu Apr 18 09:13:35 2019
Ending filelists db creation: Thu Apr 18 09:13:35 2019
Starting primary db creation: Thu Apr 18 09:13:35 2019
Ending primary db creation: Thu Apr 18 09:13:35 2019
Sqlite DBs complete
```

虽然不存在`x86_64`包，但为了避免`yum`后续投诉，重复对此新目录调用`createrepo`：

```py
$ mkdir /var/www/repos/centos/x86_64
$ createrepo -v /var/www/repos/centos/x86_64
```

我们将使用`http.server`模块通过 HTTP 提供此目录的服务：

```py
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

要访问此存储库，`yum`需要配置*repo 文件*。在*/etc/yum.repos.d/hello-world.repo*创建一个。它应该如下所示：

```py
[hello-world]
name=hello-world example repo for noarch packages
baseurl=http://0.0.0.0:8000/$basearch
enabled=1
gpgcheck=0
type=rpm-md
priority=1

[hello-world-noarch]
name=hello-world example repo for noarch packages
baseurl=http://0.0.0.0:8000/noarch
enabled=1
gpgcheck=0
type=rpm-md
priority=1
```

注意`gpgcheck`值为`0`。这表示我们尚未签署任何软件包，`yum`不应尝试验证签名，以防止此示例中的故障。现在应该可以搜索软件包，并在输出的一部分中获取描述：

```py
$ yum --enablerepo=hello-world search hello-world
Loaded plugins: fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * base: reflector.westga.edu
 * epel: mirror.vcu.edu
 * extras: mirror.steadfastnet.com
 * updates: mirror.mobap.edu
base                                                                   | 3.6 kB
extras                                                                 | 3.4 kB
hello- world                                                           | 2.9 kB
hello-world-noarch                                                     | 2.9 kB
updates                                                                | 3.4 kB
8 packages excluded due to repository priority protections
===============================================================================
matched: hello-world
===============================================================================
hello-world.noarch : A hello-world example pacakge
```

搜索功能正常工作；安装软件包也应正常工作：

```py
$ yum --enablerepo=hello-world install hello-world
Loaded plugins: fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * base: reflector.westga.edu
 * epel: mirror.vcu.edu
 * extras: mirror.steadfastnet.com
 * updates: mirror.mobap.edu
8 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package hello-world.noarch 0:0.0.1-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved
Installing:
 hello-world          noarch          0.0.1-1            hello-world-noarch

Transaction Summary
Install  1 Package

Total download size: 8.1 k
Installed size: 1.3 k
Downloading packages:
hello-world-0.0.1-1.noarch.rpm                                         | 8.1 kB
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : hello-world-0.0.1-1.noarch
  Verifying  : hello-world-0.0.1-1.noarch

Installed:
  hello-world.noarch 0:0.0.1-1

Complete!
```

移除也必须正常工作：

```py
$ yum remove hello-world
Loaded plugins: fastestmirror, priorities
Resolving Dependencies
--> Running transaction check
---> Package hello-world.noarch 0:0.0.1-1 will be erased
--> Finished Dependency Resolution

Dependencies Resolved
Removing:
 hello-world          noarch          0.0.1-1           @hello-world-noarch

Transaction Summary
Remove  1 Package

Installed size: 1.3 k
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : hello-world-0.0.1-1.noarch
  Verifying  : hello-world-0.0.1-1.noarch
Removed:
  hello-world.noarch 0:0.0.1-1
Complete!
```

`http.server`模块应显示一些活动，证明`yum`正在获取`hello-world`包：

```py
[18/Apr/2019 03:37:24] "GET /x86_64/repodata/repomd.xml HTTP/1.1"
[18/Apr/2019 03:37:24] "GET /noarch/repodata/repomd.xml HTTP/1.1"
[18/Apr/2019 03:37:25] "GET /x86_64/repodata/primary.sqlite.bz2 HTTP/1.1"
[18/Apr/2019 03:37:25] "GET /noarch/repodata/primary.sqlite.bz2 HTTP/1.1"
[18/Apr/2019 03:56:49] "GET /noarch/hello-world-0.0.1-1.noarch.rpm HTTP/1.1"
```

# 使用 systemd 进行管理

`systemd`是 Linux 的*系统和服务管理器*（也称为*init 系统*）。它是许多发行版的默认 init 系统，如 Debian 和 Red Hat。以下是`systemd`提供的众多功能之一：

+   简单并行化

+   钩子和触发器用于按需行为

+   日志集成

+   能够依赖于其他单元以编排复杂的启动

还有许多其他激动人心的`systemd`功能，例如网络、DNS，甚至是设备挂载。在 Python 中轻松处理进程的想法一直是具有挑战性的；曾经有几个类似*init*的 Python 项目供选择，都有各自的配置和处理 API。使用`systemd`允许可移植性，并使与他人协作变得容易，因为它广泛可用。

###### 提示

Python 中两个知名的进程处理程序是 [`supervisord`](http://supervisord.org) 和 [`circus`](https://oreil.ly/adGEj)。

不久前，Alfredo 写了一个小的 Python HTTP API，需要投入生产使用。该项目已从 `supervisord` 迁移到 `circus`，一切都运行正常。不幸的是，生产环境的限制意味着需要将 `systemd` 与操作系统集成。由于 `systemd` 相对较新，过渡过程很粗糙，但一旦一切就绪，我们就从中受益，可以在开发周期的早期阶段进行类似生产环境的处理，并更早地捕获集成问题。当 API 进入发布时，我们已经对 `systemd` 感到满意，可以排除问题，甚至调整配置以应对外部问题。（你有没有见过由于网络未运行而导致 `init` 脚本失败的情况？）

在本节中，我们构建了一个小型 HTTP 服务，它在系统启动时需要可用，并且可以在任何时刻重新启动。单元配置处理日志记录，并确保在尝试启动之前可用特定的系统资源。

## 长期运行的进程

那些应该一直运行的进程非常适合使用 `systemd` 进行处理。想想 DNS 或邮件服务器的工作原理；这些都是*一直在运行*的程序，它们需要一些处理来捕获日志或在配置更改时重新启动。

我们将使用一个基于 [Pecan web framework](https://www.pecanpy.org) 的小型 HTTP API 服务器。

###### 注意

本节对 Pecan 的工作方式没有特别说明，因此示例可以用于其他框架或长期运行的服务。

## 设置

选择一个永久位置用于项目，在 */opt/http* 创建一个目录，然后创建一个新的虚拟环境并安装 Pecan 框架：

```py
$ mkdir -p /opt/http
$ cd /opt/http
$ python3 -m venv .
$ source bin/activate
(http) $ pip install "pecan==1.3.3"
```

Pecan 有一些内置的辅助工具，可以为示例项目创建必要的文件和目录。Pecan 可以用于创建一个基本的“香草”HTTP API 项目，将其连接到 `systemd`。版本`1.3.3` 有两个选项：`base` 和 `rest-api`。

```py
$ pecan create api rest-api
Creating /opt/http/api
Recursing into +package+
  Creating /opt/http/api/api
...
Copying scaffolds/rest-api/config.py_tmpl to /opt/http/api/config.py
Copying scaffolds/rest-api/setup.cfg_tmpl to /opt/http/api/setup.cfg
Copying scaffolds/rest-api/setup.py_tmpl to /opt/http/api/setup.py
```

###### 小贴士

使用一致的路径非常重要，因为稍后在使用 `systemd` 配置服务时会用到它。

通过包含项目脚手架，我们现在可以毫不费力地拥有一个完全功能的项目。它甚至有一个*setup.py*文件，里面包含了所有内容，可以立即成为一个原生的 Python 包！让我们安装该项目，以便运行它：

```py
(http) $ python setup.py install
running install
running bdist_egg
running egg_info
creating api.egg-info
...
creating dist
creating 'dist/api-0.1-py3.6.egg' and adding 'build/bdist.linux-x86_64/egg'
removing 'build/bdist.linux-x86_64/egg' (and everything under it)
Processing api-0.1-py3.6.egg
creating /opt/http/lib/python3.6/site-packages/api-0.1-py3.6.egg
Extracting api-0.1-py3.6.egg to /opt/http/lib/python3.6/site-packages
...
Installed /opt/http/lib/python3.6/site-packages/api-0.1-py3.6.egg
Processing dependencies for api==0.1
Finished processing dependencies for api==0.1
```

`pecan` 命令行工具需要一个配置文件。配置文件已经由脚手架为您创建，并保存在顶级目录中。使用 *config.py* 文件启动服务器：

```py
(http) $ pecan serve config.py
Starting server in PID 17517
serving on 0.0.0.0:8080, view at http://127.0.0.1:8080
```

在浏览器上进行测试应该会产生一个纯文本消息。这是使用 `curl` 命令显示的方式：

```py
(http) $ curl localhost:8080
Hello, World!
```

长时间运行的进程从 `pecan serve config.py` 开始。唯一停止此进程的方法是使用 `Control-C` 发送 `KeyboardInterrupt`。重新启动需要激活虚拟环境，再次运行相同的 `pecan serve` 命令。

## systemd 单元文件

与旧的初始化系统不同，它使用可执行脚本，`systemd` 使用纯文本文件。单元文件的最终版本如下所示：

```py
[Unit]
Description=hello world pecan service
After=network.target

[Service]
Type=simple
ExecStart=/opt/http/bin/pecan serve /opt/http/api/config.py
WorkingDirectory=/opt/http/api
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

将此文件保存为 `hello-world.service`。稍后会将其复制到最终目标位置。

确保获取所有部分名称和配置指令的准确性非常重要，因为它们都区分大小写。如果名称不完全匹配，事情将无法正常工作。让我们详细讨论 HTTP 服务的每个部分：

单元

提供了描述并包括一个 `After` 指令，告诉 `systemd` 此服务单元在启动之前需要具备操作网络环境。其他单元可能有更复杂的要求，不仅仅是启动服务，甚至是*启动后*！ `Condition` 和 `Wants` 是其他非常有用的指令。

服务

当配置 *service* 单元时才需要此部分。默认为 `Type=simple`。此类服务不应分叉，它们必须保持在前台，以便 `systemd` 可以处理它们的操作。 `ExecStart` 行解释了命令应如何运行以启动服务。使用绝对路径是*至关重要*的，以避免找不到正确文件的问题。

虽然不是必需的，我已包含 `WorkingDirectory` 指令，以确保进程在应用程序所在的相同目录中。如果以后有任何更新，它可能会因为已经处于与应用程序相关的位置而受益。

`StandardOutput` 和 `StandardError` 指令非常好用，并展示了 `systemd` 在这方面的强大功能。它会通过 `systemd` 机制处理所有通过 `stdout` 和 `stderr` 发出的日志。在我们解释如何与服务交互时，我们将进一步演示这一点。

安装

`WantedBy` 指令解释了一旦启用，此单元如何处理。 `multi-user.target` 相当于 `runlevel 3`（服务器启动到终端的正常运行级别）。这种类型的配置允许系统确定启用后的行为方式。一旦启用，会在 *multi-user.target.wants* 目录中创建一个符号链接。

# 安装单元

配置文件本身必须放置在特定位置，以便 `systemd` 能够找到并*加载*它。支持多种位置，但 */etc/systemd/system* 是由管理员创建或管理的单元所用的位置。

###### 提示

确保 `ExecStart` 指令与这些路径配合正常非常有用。使用绝对路径可以减少引入拼写错误的机会。要验证，请在终端中运行整行命令，并查找类似于以下输出：

```py
$ /opt/http/bin/pecan serve /opt/http/api/config.py
Starting server in PID 20621
serving on 0.0.0.0:8080, view at http://127.0.0.1:8080
```

验证命令是否有效后，使用`hello-world.service`作为名称将单元文件复制到此目录：

```py
$ cp hello-world.service /etc/systemd/system/
```

放置后，需要重新加载`systemd`以使其意识到这个新单元：

```py
$ systemctl daemon-reload
```

服务现在完全可用并可以启动和停止。通过使用`status`子命令进行验证。让我们看看`systemd`是否识别它。这是其应有的行为以及输出的样子：

```py
$ systemctl status hello-world
● hello-world.service - hello world pecan service
   Loaded: loaded (/etc/systemd/system/hello-world.service; disabled; )
   Active: inactive (dead)
```

由于服务未运行，看到它被报告为`dead`并不奇怪。接下来启动服务，并再次检查状态（`curl`应该报告端口`8080`上没有运行任何内容）：

```py
$ curl localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
$ systemctl start hello-world
$ systemctl status hello-world
● hello-world.service - hello world pecan service
   Loaded: loaded (/etc/systemd/system/hello-world.service; disabled; )
   Active: active (running) since Tue 2019-04-23 13:44:20 EDT; 5s ago
 Main PID: 23980 (pecan)
    Tasks: 1 (limit: 4915)
   Memory: 20.1M
   CGroup: /system.slice/hello-world.service
           └─23980 /opt/http/bin/python /opt/http/bin/pecan serve config.py

Apr 23 13:44:20 huando systemd[1]: Started hello world pecan service.
```

服务正在运行且完全可操作。再次验证端口`8080`上的服务，确保框架正在运行并响应请求：

```py
$ curl localhost:8080
Hello, World!
```

如果使用`systemctl stop hello-world`停止服务，那么`curl`命令将再次报告连接失败。

到目前为止，我们已经创建并安装了该单元，通过启动和停止服务验证了其工作，并检查了 Pecan 框架是否在其默认端口上响应请求。如果服务器在任何时候重新启动，您希望此服务处于运行状态，这就是`Install`部分发挥作用的地方。让我们`enable`该服务：

```py
$ systemctl enable hello-world
Created symlink hello-world.service → /etc/systemd/system/hello-world.service.
```

当服务器重新启动时，小型 HTTP API 服务将会启动并运行。

## 日志处理

由于这是一个配置了日志记录配置（所有`stdout`和`stderr`都直接进入`systemd`）的服务，处理工作是*免费*的。无需配置基于文件的日志记录、旋转或甚至过期。`systemd`提供了一些有趣且非常好用的功能，允许您与日志交互，例如限制时间范围和按单元或进程 ID 过滤。

###### 注意

与单元日志交互的命令通过`journalctl`命令行工具完成。如果期望`systemd`提供另一个子命令来提供日志助手，则这个过程可能会有所不同。

由于我们在前一节中启动了服务并通过`curl`发送了一些请求给它，让我们看看日志显示了什么：

```py
$ journalctl -u hello-world
-- Logs begin at Mon 2019-04-15 09:05:11 EDT, end at Tue 2019-04-23
Apr 23 13:44:20 srv1 systemd[1]: Started hello world pecan service.
Apr 23 13:44:44 srv1 pecan[23980] [INFO    ] [pecan.commands.serve] GET / 200
Apr 23 13:44:55 srv1 systemd[1]: Stopping hello world pecan service...
Apr 23 13:44:55 srv1 systemd[1]: hello-world.service: Main process exited
Apr 23 13:44:55 srv1 systemd[1]: hello-world.service: Succeeded.
Apr 23 13:44:55 srv1 systemd[1]: Stopped hello world pecan service.
```

`-u`标志指定了*单元*，在本例中是`hello-world`，但您也可以使用模式或甚至指定多个单元。

跟踪生成条目的日志的常见方法是使用`tail`命令。具体来说，如下所示：

```py
$ tail -f pecan-access.log
```

使用`journalctl`执行相同操作的命令看起来略有不同，但它的*工作方式相同*：

```py
$ journalctl -fu hello-world
Apr 23 13:44:44 srv1 pecan[23980][INFO][pecan.commands.serve] GET / 200
Apr 23 13:44:44 srv1 pecan[23980][INFO][pecan.commands.serve] GET / 200
Apr 23 13:44:44 srv1 pecan[23980][INFO][pecan.commands.serve] GET / 200
```

###### 提示

如果`systemd`包使用了`pcre2`引擎可用，它允许您使用`--grep`。这进一步基于模式过滤日志条目。

`-f` 标志意味着 *跟随* 日志，并从最近的条目开始，并继续显示它们的进展，就像 `tail -f` 一样。在生产环境中，日志可能太多，并且可能已经出现了错误 *today*。在这些情况下，您可以结合使用 `--since` 和 `--until`。这两个标志都接受几种不同类型的参数：

+   `today`

+   `yesterday`

+   `"3 hours ago"`

+   `-1h`

+   `-15min`

+   `-1h35min`

在我们的小例子中，`journalctl` 在过去的 15 分钟内无法找到任何内容。在输出的开头，它通知我们范围，并生成条目（如果有的话）：

```py
$  journalctl -u hello-world --since "-15min"
-- Logs begin at Mon 2019-04-15 09:05:11 EDT, end at Tue 2019-04-23
-- No entries --
```

# 练习

+   使用三个不同的命令通过 `journalctl` 获取 `systemd` 的日志输出。

+   解释 `systemd` 单元的 `WorkingDirectory` 配置选项是用来做什么的。

+   为什么 changelog 很重要？

+   *setup.py* 文件的作用是什么？

+   列出 Debian 和 RPM 包之间的三个不同之处。

# 案例研究问题

+   使用 `devpi` 创建 PyPI 的本地实例，上传一个 Python 包，然后尝试从本地的 `devpi` 实例安装该 Python 包。
