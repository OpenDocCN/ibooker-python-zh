# 第十三章。生产

> 如果建筑工人建造建筑物的方式就像程序员编写程序一样，那么第一个啄木鸟就会毁掉文明。
> 
> 杰拉尔德·温伯格，计算机科学家

# 预览

你在本地机器上运行着一个应用程序，现在你想要分享它。本章介绍了许多场景，说明了如何将你的应用程序移动到生产环境，并确保它正确高效地运行。由于一些细节可能非常详细，在某些情况下我会参考一些有用的外部文档，而不是在这里堆砌它们。

# 部署

迄今为止，本书中的所有代码示例都使用了在 `localhost` 的端口 `8000` 上运行的单个 `uvicorn` 实例。为了处理大量流量，你需要多个服务器，运行在现代硬件提供的多个核心上。你还需要在这些服务器之上添加一些东西来执行以下操作：

+   使它们保持运行（*监督者*）

+   收集和提供外部请求（*反向代理*）

+   返回响应

+   提供 HTTPS *终止*（SSL 解密）

## 多个工作进程

你可能见过另一个名为 [Gunicorn](https://gunicorn.org) 的 Python 服务器。这个服务器可以监视多个工作进程，但它是一个 WSGI 服务器，而 FastAPI 是基于 ASGI 的。幸运的是，有一个特殊的 Uvicorn 工作进程类可以由 Gunicorn 管理。

示例 13-1 在 `localhost` 的端口 `8000` 上设置了这些 Uvicorn 工作进程（这是从 [官方文档](https://oreil.ly/Svdhx) 改编的）。引号保护 shell 免受任何特殊解释。

##### 示例 13-1。使用 Gunicorn 和 Uvicorn 工作进程

```py
$ pip install "uvicorn[standard]" gunicorn
$ gunicorn main:app --workers 4 --worker-class \
uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

当 Gunicorn 执行你的命令时，你会看到许多行。它将启动一个顶级 Gunicorn 进程，与四个 Uvicorn 工作进程子进程交流，所有进程共享 `localhost` (`0.0.0.0`) 上的端口 `8000`。如果你想要其他内容，可以更改主机、端口或工作进程数。`main:app` 指的是 *main.py* 和带有变量名 `app` 的 FastAPI 对象。Gunicorn 的[文档](https://oreil.ly/TxYIy)宣称如下：

> Gunicorn 只需要 4-12 个工作进程来处理每秒数百或数千个请求。

结果发现 Uvicorn 本身也可以启动多个 Uvicorn 工作进程，就像 示例 13-2 中一样。

##### 示例 13-2。使用 Uvicorn 和 Uvicorn 工作进程

```py
$ uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

但这种方法不涉及进程管理，因此通常更喜欢 gunicorn 方法。其他进程管理器也适用于 Uvicorn：参见其[官方文档](https://www.uvicorn.org/deployment)。

这处理了前一节提到的四项工作中的三项，但不包括 HTTPS 加密。

## HTTPS

官方 FastAPI [HTTPS 文档](https://oreil.ly/HYRW7)，就像所有官方 FastAPI 文档一样，都非常丰富。我建议先阅读它们，然后再阅读 Ramírez 的[描述](https://oreil.ly/zcUWS)，了解如何通过使用 [Traefik](https://traefik.io) 向 FastAPI 添加 HTTPS 支持。Traefik 位于你的 Web 服务器“之上”，类似于作为反向代理和负载均衡器的 nginx，但它包含了 HTTPS 魔法。

尽管这个过程有许多步骤，但比以前简单得多。特别是，以前你经常为数字证书向证书颁发机构支付高昂费用，以便为你的网站提供 HTTPS。幸运的是，这些机构大多被免费服务[Let's Encrypt](https://letsencrypt.org)所取代。

## Docker

当 Docker（在 2013 年 PyCon 的 Solomon Hykes 的五分钟[闪电演讲](https://oreil.ly/25oef)中出现）时，大多数人第一次听说 Linux 容器。随着时间的推移，我们了解到 Docker 比虚拟机更快、更轻。每个容器不是模拟完整的操作系统，而是共享服务器的 Linux 内核，并将进程和网络隔离到自己的命名空间中。突然间，通过使用免费的 Docker 软件，你可以在单台机器上托管多个独立服务，而不必担心它们互相干扰。

十年后，Docker 已被普遍认可和支持。如果你想在云服务上托管你的 FastAPI 应用程序，通常需要首先创建它的*Docker 镜像*。[官方 FastAPI 文档](https://oreil.ly/QnwOW)包含了如何构建你的 FastAPI 应用程序的 Docker 化版本的详细描述。其中一步是编写*Dockerfile*：一个包含 Docker 配置信息的文本文件，如要使用的应用程序代码和要运行的进程。只为证明这不是在火箭发射期间进行的脑外科手术，这里是来自该页面的 Dockerfile：

```py
FROM python:3.9
WORKDIR /code
COPY ./requirements.txt /code/requirements.txt
RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt
COPY ./app /code/app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]
```

我建议阅读官方文档，或者通过谷歌搜索`fastapi docker`得到的其他链接，比如[“使用 Docker 部署你的应用程序的终极 FastAPI 教程第十三部分”](https://oreil.ly/7TUpR) by Christopher Samiullah。

## 云服务

在网上有许多付费或免费的主机来源。一些关于如何与它们一起托管 FastAPI 的指南包括以下内容：

+   [“FastAPI——部署” by Tutorials Point](https://oreil.ly/DBZcm)

+   [“Linode 上的终极 FastAPI 教程第 6b 部分——基本部署” by Christopher Samiullah](https://oreil.ly/s8iar)

+   [“如何免费在 Heroku 上部署 FastAPI 应用程序” by Shinichi Okada](https://oreil.ly/A6gij)

## Kubernetes

Kubernetes 起源于 Google 内部用于管理越来越复杂的内部系统的代码。当时的系统管理员（他们当时被称为）过去常常手动配置诸如负载均衡器、反向代理、湿度计¹等工具。Kubernetes 的目标是获取这些知识并自动化：不要告诉我*如何*处理这个问题；告诉我你*想要*什么。这包括保持服务运行或在流量激增时启动更多服务器等任务。

有很多关于如何在 Kubernetes 上部署 FastAPI 的描述，包括[“在 Kubernetes 上部署 FastAPI 应用程序” by Sumanta Mukhopadhyay](https://oreil.ly/ktTNu)。

# 性能

FastAPI 的性能目前[位于最高水平之一](https://oreil.ly/mxabf)，甚至可以与像 Go 这样更快语言的框架相媲美。但其中很大一部分归功于 ASGI，通过异步避免 I/O 等待。Python 本身是一种相对较慢的语言。以下是一些提高整体性能的技巧和窍门。

## 异步

通常 Web 服务器不需要真正快。它大部分时间都在获取 HTTP 网络请求和返回结果（本书中的 Web 层）。在此期间，Web 服务执行业务逻辑（服务层）并访问数据源（数据层），再次花费大部分时间在网络 I/O 上。

每当 Web 服务中的代码需要等待响应时，使用异步函数（`async def` 而不是 `def`）是个不错的选择。这样可以让 FastAPI 和 Starlette 调度异步函数，在等待获取响应时做其他事情。这也是 FastAPI 的基准测试比基于 WSGI 的框架（如 Flask 和 Django）好的原因之一。

性能有两个方面：

+   处理单个请求所需的时间

+   可同时处理的请求数量

## 缓存

如果您有一个最终从静态源获取数据的 Web 端点（例如几乎不会更改或从不更改的数据库记录），可以在函数中*缓存*数据。这可以在任何层中完成。Python 提供了标准的[functools 模块](https://oreil.ly/8Kg4V)和函数 `cache()` 和 `lru_cache()`。

## 数据库、文件和内存

慢网站最常见的原因之一是数据库表缺少适当大小的索引。通常直到表增长到特定大小之后，查询突然变得更慢才能看到问题。在 SQL 中，`WHERE` 子句中的任何列都应该有索引。

在本书的许多示例中，`creature` 和 `explorer` 表的主键一直是文本字段 `name`。在创建表时，`name` 被声明为 `primary key`。到目前为止，在本书中看到的小表中，SQLite 无论如何都会忽略该键，因为仅扫描表更快。但一旦表达到一个可观的大小，比如一百万行，缺少索引就会显著影响性能。解决方案是运行[查询优化器](https://oreil.ly/YPR3Q)。

即使有一个小表，也可以使用 Python 脚本或开源工具进行数据库负载测试。如果您正在进行大量顺序数据库查询，可能可以将它们合并成一个批处理。如果要上传或下载大文件，请使用流式版本而不是整体读取。

## 队列

如果您执行的任何任务花费超过一小部分秒钟的时间（例如发送确认电子邮件或缩小图像），可能值得将其交给作业队列，例如[Celery](https://docs.celeryq.dev)。

## Python 本身

如果您的 Web 服务似乎因使用 Python 进行大量计算而变慢，您可能需要一个“更快的 Python”。替代方案包括以下内容：

+   使用[PyPy](https://www.pypy.org)而不是标准的 CPython。

+   用 C、C++或 Rust 编写 Python [扩展](https://oreil.ly/BElJa)。

+   将缓慢的 Python 代码转换为[Cython](https://cython.org)（由 Pydantic 和 Uvicorn 自身使用）。

最近一个非常引人注目的公告是[Mojo 语言](https://oreil.ly/C96kx)。它旨在成为 Python 的完整超集，具有新功能（使用相同友好的 Python 语法），可以将 Python 示例的速度提高*数千*倍。主要作者 Chris Lattner 之前曾在 Apple 上开发了像[LLVM](https://llvm.org)，[Clang](https://clang.llvm.org)和[MLIR](https://mlir.llvm.org)以及[Swift](https://www.swift.org)语言等编译器工具。

Mojo 旨在成为 AI 开发的单一语言解决方案，现在（在 PyTorch 和 TensorFlow 中）需要 Python/C/C++三明治，这使得开发、管理和调试变得困难。但 Mojo 也将是一个除了 AI 之外的好的通用语言。

我多年来一直在 C 中编码，并一直在等待一个像 Python 一样易于使用但性能良好的后继者。D、Go、Julia、Zig 和 Rust 都是可能的选择，但如果 Mojo 能够实现其[目标](https://oreil.ly/EojvA)，我将广泛使用 Mojo。

# 故障排除

从遇到问题的时间和地点向下查看。这包括时间和空间性能问题，还包括逻辑和异步陷阱。

## 问题类型

乍一看，您得到了什么 HTTP 响应代码？

`404`

出现身份验证或授权错误。

`422`

通常是 Pydantic 对模型使用的投诉。

`500`

FastAPI 背后的服务失败。

## 记录

Uvicorn 和其他 Web 服务器通常将日志写入 stdout。您可以检查日志以查看实际进行的调用，包括 HTTP 动词和 URL，但不包括正文、标头或 Cookie 中的数据。

如果特定的端点返回 400 级状态码，您可以尝试将相同的输入反馈并查看是否再次出现错误。如果是这样，我的第一个原始调试直觉是在相关的 Web、服务和数据函数中添加`print()`语句。

此外，无论何处引发异常，都要添加详细信息。如果数据库查找失败，请包含输入值和具体错误，例如尝试加倍唯一键字段。

## 指标

术语*指标*、*监控*、*可观察性*和*遥测*可能似乎有重叠之处。在 Python 领域中，使用以下常见实践：

+   [Prometheus](https://prometheus.io)用于收集指标

+   [Grafana](https://grafana.com)用于显示它们

+   [OpenTelemetry](https://opentelemetry.io)用于测量时间

您可以将这些应用于站点的所有层次：Web、服务和数据。服务层可能更加面向业务，其他层更多是技术方面的，对站点开发人员和维护者有用。

这里有一些链接可用于收集 FastAPI 的指标：

+   [Prometheus FastAPI Instrumentator](https://oreil.ly/EYJwR)

+   [“入门指南：使用 Grafana 和 Prometheus 监控 FastAPI 应用程序—逐步指南” by Zoo Codes](https://oreil.ly/Gs90t)

+   [“FastAPI Observability” page of Grafana Labs website](https://oreil.ly/spKwe)

+   [OpenTelemetry FastAPI Instrumentation](https://oreil.ly/wDSNv)

+   [“OpenTelemetry FastAPI Tutorial—Complete Implementation Guide” by Ankit Anand](https://oreil.ly/ZpSXs)

+   [OpenTelemetry Python documentation](https://oreil.ly/nSD4G)

# 回顾

生产显然并不容易。问题包括网络和磁盘超载，以及数据库问题。本章提供了如何获取所需信息的提示，以及在问题出现时从哪里开始挖掘。

¹ 等等，这些可以保持雪茄的新鲜。
