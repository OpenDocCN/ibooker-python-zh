# 附录 B：模板项目结构

> 原文：[Appendix B: A Template Project Structure](https://www.cosmicpython.com/book/appendix_project_structure.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

在第四章周围，我们从只在一个文件夹中拥有所有内容转移到了更有结构的树形结构，并且我们认为可能会对梳理各个部分感兴趣。

###### 提示

本附录的代码位于 GitHub 上的 appendix_project_structure 分支中[（https://oreil.ly/1rDRC）](https://oreil.ly/1rDRC)：

```py
git clone https://github.com/cosmicpython/code.git
cd code
git checkout appendix_project_structure
```

基本的文件夹结构如下：

*项目树*

```py
.
├── Dockerfile  (1)
├── Makefile  (2)
├── README.md
├── docker-compose.yml  (1)
├── license.txt
├── mypy.ini
├── requirements.txt
├── src  (3)
│   ├── allocation
│   │   ├── __init__.py
│   │   ├── adapters
│   │   │   ├── __init__.py
│   │   │   ├── orm.py
│   │   │   └── repository.py
│   │   ├── config.py
│   │   ├── domain
│   │   │   ├── __init__.py
│   │   │   └── model.py
│   │   ├── entrypoints
│   │   │   ├── __init__.py
│   │   │   └── flask_app.py
│   │   └── service_layer
│   │       ├── __init__.py
│   │       └── services.py
│   └── setup.py  (3)
└── tests  (4)
    ├── conftest.py  (4)
    ├── e2e
    │   └── test_api.py
    ├── integration
    │   ├── test_orm.py
    │   └── test_repository.py
    ├── pytest.ini  (4)
    └── unit
        ├── test_allocate.py
        ├── test_batches.py
        └── test_services.py
```

①

我们的*docker-compose.yml*和我们的*Dockerfile*是运行我们的应用程序的容器的主要配置部分，它们也可以运行测试（用于 CI）。一个更复杂的项目可能会有几个 Dockerfile，尽管我们发现最小化镜像数量通常是一个好主意。¹

②

*Makefile*提供了开发人员（或 CI 服务器）在其正常工作流程中可能想要运行的所有典型命令的入口点：`make build`，`make test`等。² 这是可选的。您可以直接使用`docker-compose`和`pytest`，但是如果没有其他选择，将所有“常用命令”列在某个地方是很好的，而且与文档不同，Makefile 是代码，因此不太容易过时。

③

我们应用程序的所有源代码，包括领域模型、Flask 应用程序和基础设施代码，都位于*src*内的 Python 包中，³我们使用`pip install -e`和*setup.py*文件进行安装。这使得导入变得容易。目前，此模块内的结构完全是平面的，但对于更复杂的项目，您可以期望增加一个包含*domain_model/*、*infrastructure/*、*services/*和*api/*的文件夹层次结构。

④

测试位于它们自己的文件夹中。子文件夹区分不同的测试类型，并允许您分别运行它们。我们可以在主测试文件夹中保留共享的固定装置（*conftest.py*），并在需要时嵌套更具体的固定装置。这也是保留*pytest.ini*的地方。

###### 提示

[pytest 文档](https://oreil.ly/QVb9Q)在测试布局和可导入性方面非常好。

让我们更详细地看一下这些文件和概念。

# 环境变量、12 因素和配置，内部和外部容器

我们在这里要解决的基本问题是，我们需要不同的配置设置，用于以下情况：

+   直接从您自己的开发机器运行代码或测试，可能是从 Docker 容器的映射端口进行通信

+   在容器本身上运行，使用“真实”端口和主机名

+   不同的容器环境（开发、暂存、生产等）

通过[12 因素宣言](https://12factor.net/config)建议的环境变量配置将解决这个问题，但具体来说，我们如何在我们的代码和容器中实现它呢？

# Config.py

每当我们的应用程序代码需要访问某些配置时，它将从一个名为*config.py*的文件中获取。以下是我们应用程序中的一些示例：

*示例配置函数（`src/allocation/config.py`）*

```py
import os


def get_postgres_uri():  #(1)
    host = os.environ.get("DB_HOST", "localhost")  #(2)
    port = 54321 if host == "localhost" else 5432
    password = os.environ.get("DB_PASSWORD", "abc123")
    user, db_name = "allocation", "allocation"
    return f"postgresql://{user}:{password}@{host}:{port}/{db_name}"


def get_api_url():
    host = os.environ.get("API_HOST", "localhost")
    port = 5005 if host == "localhost" else 80
    return f"http://{host}:{port}"
```

①

我们使用函数来获取当前的配置，而不是在导入时可用的常量，因为这样可以让客户端代码修改`os.environ`。

②

*config.py*还定义了一些默认设置，设计用于在从开发人员的本地机器运行代码时工作。⁴

一个名为[*environ-config*](https://github.com/hynek/environ-config)的优雅 Python 包值得一看，如果您厌倦了手动编写基于环境的配置函数。

###### 提示

不要让这个配置模块成为一个充满了与配置只有模糊关系的东西的倾倒场所，然后在各个地方都导入它。保持事物不可变，并且只通过环境变量进行修改。如果您决定使用[引导脚本](ch13.xhtml#chapter_13_dependency_injection)，您可以将其作为导入配置的唯一位置（除了测试）。

# Docker-Compose 和容器配置

我们使用一个轻量级的 Docker 容器编排工具叫做*docker-compose*。它的主要配置是通过一个 YAML 文件（叹气）：⁵

*docker-compose*配置文件（docker-compose.yml）

```py
version: "3"
services:

  app:  #(1)
    build:
      context: .
      dockerfile: Dockerfile
    depends_on:
      - postgres
    environment:  #(3)
      - DB_HOST=postgres  (4)
      - DB_PASSWORD=abc123
      - API_HOST=app
      - PYTHONDONTWRITEBYTECODE=1  #(5)
    volumes:  #(6)
      - ./src:/src
      - ./tests:/tests
    ports:
      - "5005:80"  (7)


  postgres:
    image: postgres:9.6  #(2)
    environment:
      - POSTGRES_USER=allocation
      - POSTGRES_PASSWORD=abc123
    ports:
      - "54321:5432"
```

①

在*docker-compose*文件中，我们定义了我们应用程序所需的不同*services*（容器）。通常一个主要的镜像包含了我们所有的代码，我们可以使用它来运行我们的 API，我们的测试，或者任何其他需要访问领域模型的服务。

②

您可能会有其他基础设施服务，包括数据库。在生产环境中，您可能不会使用容器；您可能会使用云提供商，但是*docker-compose*为我们提供了一种在开发或 CI 中生成类似服务的方式。

③

`environment`部分允许您为容器设置环境变量，主机名和端口，从 Docker 集群内部看到。如果您有足够多的容器，这些信息开始在这些部分中重复，您可以改用`environment_file`。我们通常称为*container.env*。

④

在集群内，*docker-compose*设置了网络，使得容器可以通过其服务名称命名的主机名相互访问。

⑤

专业提示：如果您将卷挂载到本地开发机器和容器之间共享源文件夹，`PYTHONDONTWRITEBYTECODE`环境变量告诉 Python 不要写入*.pyc*文件，这将使您免受在本地文件系统上到处都是数百万个根文件的困扰，删除起来很烦人，并且会导致奇怪的 Python 编译器错误。

![6](img/6.png)

将我们的源代码和测试代码作为`volumes`挂载意味着我们不需要在每次代码更改时重新构建我们的容器。

![7](img/7.png)

`ports`部分允许我们将容器内部的端口暴露到外部世界⁶——这些对应于我们在*config.py*中设置的默认端口。

###### 注意

在 Docker 内部，其他容器可以通过其服务名称命名的主机名访问。在 Docker 外部，它们可以在`localhost`上访问，在`ports`部分定义的端口上。

# 将您的源代码安装为包

我们所有的应用程序代码（除了测试，实际上）都存放在*src*文件夹内：

*src 文件夹*

```py
├── src
│   ├── allocation (1) 
│   │   ├── config.py
│   │   └── ...
│   └── setup.py (2)
```

①

子文件夹定义了顶级模块名称。您可以有多个。

②

*setup.py*是您需要使其可通过 pip 安装的文件，下面会展示。

*src/setup.py*中的三行可安装的 pip 模块

```py
from setuptools import setup

setup(
    name='allocation',
    version='0.1',
    packages=['allocation'],
)
```

这就是您需要的全部。`packages=`指定要安装为顶级模块的子文件夹的名称。`name`条目只是装饰性的，但是是必需的。对于一个永远不会真正进入 PyPI 的包，它会很好。⁷

# Dockerfile

Dockerfile 将会是非常特定于项目的，但是这里有一些您期望看到的关键阶段：

*我们的 Dockerfile（Dockerfile）*

```py
FROM python:3.9-slim-buster

(1)
# RUN apt install gcc libpq (no longer needed bc we use psycopg2-binary)

(2)
COPY requirements.txt /tmp/
RUN pip install -r /tmp/requirements.txt

(3)
RUN mkdir -p /src
COPY src/ /src/
RUN pip install -e /src
COPY tests/ /tests/

(4)
WORKDIR /src
ENV FLASK_APP=allocation/entrypoints/flask_app.py FLASK_DEBUG=1 PYTHONUNBUFFERED=1
CMD flask run --host=0.0.0.0 --port=80
```

①

安装系统级依赖

②

安装我们的 Python 依赖项（您可能希望将开发和生产依赖项分开；为简单起见，我们没有这样做）

③

复制和安装我们的源代码

④

可选配置默认启动命令（您可能经常需要从命令行覆盖这个）

###### 提示

需要注意的一件事是，我们按照它们可能发生变化的频率安装东西的顺序。这使我们能够最大程度地重用 Docker 构建缓存。我无法告诉你这个教训背后有多少痛苦和挫折。有关此问题以及更多 Python Dockerfile 改进提示，请查看[“可生产使用的 Docker 打包”](https://pythonspeed.com/docker)。

# 测试

我们的测试与其他所有内容一起保存，如下所示：

*测试文件夹树*

```py
└── tests
    ├── conftest.py
    ├── e2e
    │   └── test_api.py
    ├── integration
    │   ├── test_orm.py
    │   └── test_repository.py
    ├── pytest.ini
    └── unit
        ├── test_allocate.py
        ├── test_batches.py
        └── test_services.py
```

这里没有特别聪明的地方，只是一些不同测试类型的分离，您可能希望单独运行，以及一些用于常见固定装置、配置等的文件。

在测试文件夹中没有*src*文件夹或*setup.py*，因为我们通常不需要使测试可通过 pip 安装，但如果您在导入路径方面遇到困难，您可能会发现它有所帮助。

# 总结

这些是我们的基本构建模块：

+   *src*文件夹中的源代码，可以使用*setup.py*进行 pip 安装

+   一些 Docker 配置，用于尽可能模拟生产环境的本地集群

+   通过环境变量进行配置，集中在一个名为*config.py*的 Python 文件中，其中默认值允许事情在*容器外*运行

+   一个用于有用的命令行命令的 Makefile

我们怀疑没有人会得到*完全*与我们相同的解决方案，但我们希望你在这里找到一些灵感。

¹ 有时将图像分离用于生产和测试是一个好主意，但我们倾向于发现进一步尝试为不同类型的应用程序代码（例如，Web API 与发布/订阅客户端）分离不值得麻烦；在复杂性和更长的重建/CI 时间方面的成本太高。你的情况可能有所不同。

² 一个纯 Python 的 Makefile 替代方案是[Invoke](http://www.pyinvoke.org)，值得一试，如果你的团队每个人都懂 Python（或者至少比 Bash 更懂）。

³ Hynek Schlawack 的[“测试和打包”](https://hynek.me/articles/testing-packaging)提供了有关*src*文件夹的更多信息。

⁴ 这为我们提供了一个“只要可能就能工作”的本地开发设置。你可能更喜欢在缺少环境变量时严格失败，特别是如果任何默认值在生产中可能不安全。

⁵ Harry 对 YAML 有点厌倦。它*无处不在*，但他永远记不住语法或应该如何缩进。

⁶ 在 CI 服务器上，您可能无法可靠地暴露任意端口，但这只是本地开发的便利。您可以找到使这些端口映射可选的方法（例如，使用*docker-compose.override.yml*）。

⁷ 有关更多*setup.py*提示，请参阅 Hynek 的[这篇关于打包的文章](https://oreil.ly/KMWDz)。
