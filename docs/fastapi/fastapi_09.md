# 第七章 框架比较

> 你不需要一个框架。你需要一幅画，而不是一个框架。
> 
> 演员克劳斯·金斯基

# 预览

对于已经使用过 Flask、Django 或流行的 Python Web 框架的开发人员，本章节指出了 FastAPI 的相似之处和差异。本章并未详尽说明每个细节，因为否则这本书的粘合剂就无法将其结合在一起。如果你考虑从这些框架之一迁移到 FastAPI 或者只是好奇，本章的比较可能会有所帮助。

关于新的 Web 框架，你可能最想知道的第一件事是如何开始，并且一种自上而下的方法是通过定义 *路由*（将 URL 和 HTTP 方法映射到函数）来实现。接下来的部分将比较如何在 FastAPI 和 Flask 中实现这一点，因为它们彼此之间比 Django 更相似，更有可能被同时考虑用于类似的应用程序。

# Flask

[Flask](https://flask.palletsprojects.com) 自称为 *微框架*。它提供了基本功能，并允许您根据需要下载第三方包来补充。与 Django 相比，它更小，对于初学者来说学习起来更快。

Flask 是基于 WSGI 而不是 ASGI 的同步框架。一个名为 [quart](https://quart.palletsprojects.com) 的新项目正在复制 Flask 并添加 ASGI 支持。

让我们从顶层开始，展示 Flask 和 FastAPI 如何定义 Web 路由。

## 路径

在顶层，Flask 和 FastAPI 都使用装饰器来将路由与 Web 端点关联。在 示例 7-1 中，让我们复制 示例 3-11（来自 第三章）的内容，该示例从 URL 路径中获取要问候的人。

##### 示例 7-1 FastAPI 路径

```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/hi/{who}")
def greet(who: str):
    return f"Hello? {who}?"
```

默认情况下，FastAPI 将 `f"Hello? {who}?"` 字符串转换为 JSON 并返回给 Web 客户端。

示例 7-2 展示了 Flask 的操作方式。

##### 示例 7-2 Flask 路径

```py
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/hi/<who>", methods=["GET"])
def greet(who: str):
    return jsonify(f"Hello? {who}?")
```

注意，装饰器中的 `who` 现在被 `<` 和 `>` 绑定起来了。在 Flask 中，方法需要作为参数包含——除非是默认的 `GET`。所以 `meth⁠ods=​["GET"]` 在这里可以省略，但明确表达从未有过伤害。

###### 注意

Flask 2.0 支持类似 FastAPI 风格的装饰器，如 `@app.get`，而不是 `app.route`。

Flask 的 `jsonify()` 函数将其参数转换为 JSON 字符串并返回，同时返回带有指示其为 JSON 的 HTTP 响应头。如果返回的是 `dict`（而不是其他数据类型），Flask 的最新版本将自动将其转换为 JSON 并返回。显式调用 `jsonify()` 对所有数据类型都有效，包括 `dict`。

## 查询参数

在 示例 7-3 中，让我们重复 示例 3-15，其中 `who` 作为查询参数传递（在 URL 中的 `?` 后面）。

##### 示例 7-3 FastAPI 查询参数

```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/hi")
def greet(who):
    return f"Hello? {who}?"
```

Flask 的等效方法显示在 示例 7-4 中。

##### 示例 7-4 Flask 查询参数

```py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/hi", methods=["GET"])
def greet():
    who: str = request.args.get("who")
    return jsonify(f"Hello? {who}?")
```

在 Flask 中，我们需要从 `request` 对象中获取请求值。在这种情况下，`args` 是包含查询参数的 `dict`。

## 主体

在示例 7-5 中，让我们复制旧的示例 3-21。

##### 示例 7-5\. FastAPI 主体

```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/hi")
def greet(who):
    return f"Hello? {who}?"
```

Flask 版本看起来像示例 7-6。

##### 示例 7-6\. Flask 主体

```py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/hi", methods=["GET"])
def greet():
    who: str = request.json["who"]
    return jsonify(f"Hello? {who}?")
```

Flask 将 JSON 输入存储在*request.json*中。

## 头部

最后，让我们重复一下示例 3-24，在示例 7-7 中。

##### 示例 7-7\. FastAPI 头部

```py
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/hi")
def greet(who:str = Header()):
    return f"Hello? {who}?"
```

Flask 版本显示在示例 7-8 中。

##### 示例 7-8\. Flask 头部

```py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/hi", methods=["GET"])
def greet():
    who: str = request.headers.get("who")
    return jsonify(f"Hello? {who}?")
```

与查询参数类似，Flask 将请求数据保存在`request`对象中。这一次，是`headers dict`属性。头部键应该是大小写不敏感的。

# Django

[Django](https://www.djangoproject.com) 比 Flask 或 FastAPI 更大更复杂，其目标是“有截止期的完美主义者”，根据其网站。其内置的对象关系映射器（ORM）对于具有主要数据库后端的站点非常有用。它更像是一个单体而不是一个工具包。是否值得额外的复杂性和学习曲线取决于你的应用程序。

虽然 Django 是传统的 WSGI 应用程序，但 3.0 版本添加了对 ASGI 的支持。

与 Flask 和 FastAPI 不同，Django 喜欢在单个`URLConf`表中定义路由（将 URL 与 Web 函数关联，它称之为*视图函数*），而不是使用装饰器。这使得在一个地方查看所有路由更容易，但在仅查看函数时，很难看出哪个 URL 与哪个函数关联。

# 其他 Web 框架特性

在前几节中比较这三个框架时，我主要比较了如何定义路由。一个 Web 框架可能也会在这些其他领域提供帮助：

表单

这三个包都支持标准的 HTML 表单。

文件

所有这些包都处理文件的上传和下载，包括多部分 HTTP 请求和响应。

模板

*模板语言*允许你混合文本和代码，并且对于一个*内容导向*的网站（HTML 文本与动态插入的数据），非常有用，而不是一个 API 网站。最著名的 Python 模板包是[Jinja](https://jinja.palletsprojects.com)，并且得到了 Flask、Django 和 FastAPI 的支持。Django 还有其自己的[模板语言](https://oreil.ly/OIbVJ)。

如果你想在基本 HTTP 之外使用网络方法，请尝试这些：

服务器发送事件

根据需要向客户端推送数据。由 FastAPI（[sse-starlette](https://oreil.ly/Hv-QP)）、Flask（[Flask-SSE](https://oreil.ly/oz518)）和 Django（[Django EventStream](https://oreil.ly/NlBE5)）支持。

队列

作业队列、发布-订阅和其他网络模式由外部包支持，如 ZeroMQ、Celery、Redis 和 RabbitMQ。

WebSockets

直接由 FastAPI 支持，Django（[Django Channels](https://channels.readthedocs.io)），以及 Flask（第三方包）。

# 数据库

Flask 和 FastAPI 的基础包中不包括任何数据库处理，但数据库处理是 Django 的关键特性。

你的站点的数据层可能在不同级别访问数据库：

+   直接 SQL（PostgreSQL，SQLite）

+   直接 NoSQL（Redis、MongoDB、Elasticsearch）

+   生成 SQL 的*ORM*

+   生成 NoSQL 的对象文档/数据映射器/管理器（ODM）

对于关系数据库，[SQLAlchemy](https://www.sqlalchemy.org)是一个很好的包，包括从直接 SQL 到 ORM 的多个访问级别。这是 Flask 和 FastAPI 开发人员的常见选择。FastAPI 的作者利用了 SQLAlchemy 和 Pydantic 来创建[SQLModel 包](https://sqlmodel.tiangolo.com)，这在第十四章中进行了更多讨论。

Django 通常是需要大量数据库的网站的框架选择。它拥有自己的[ORM](https://oreil.ly/eFzZn)和一个自动化的[数据库管理页面](https://oreil.ly/_al42)。尽管一些来源建议非技术人员使用这个管理页面进行常规数据管理，但要小心。在一个案例中，我曾见过一个非专家误解了管理页面的警告信息，导致数据库需要手动从备份中恢复。

第十四章对 FastAPI 和数据库进行了更深入的讨论。

# 推荐

对于基于 API 的服务，FastAPI 现在似乎是最佳选择。Flask 和 FastAPI 在快速启动服务方面几乎相同。Django 需要更多时间理解，但为更大的站点提供了许多有用的特性，特别是对于那些依赖重度数据库的站点。

# 其他 Python Web 框架

当前最主要的三个 Python Web 框架是 Flask、Django 和 FastAPI。谷歌**`python web frameworks`**，你会得到许多建议，我这里不会重复了。一些在这些列表中可能不太突出但因某种原因有趣的包括以下几个：

[Bottle](https://bottlepy.org/docs/dev)

一个*非常*精简（单个 Python 文件）的包，适用于快速概念验证

[Litestar](https://litestar.dev)

类似于 FastAPI——它基于 ASGI/Starlette 和 Pydantic，但有自己的观点

[AIOHTTP](https://docs.aiohttp.org)

带有有用的演示代码的 ASGI 客户端和服务器

[Socketify.py](https://docs.socketify.dev)

声称性能非常高的新参与者

# 回顾

Flask 和 Django 是最流行的 Python Web 框架，尽管 FastAPI 的受欢迎程度增长速度更快。这三个框架都处理基本的 Web 服务器任务，学习曲线不同。FastAPI 似乎具有更清晰的语法来指定路由，并且它的 ASGI 支持使得在许多情况下运行速度比竞争对手更快。接下来：让我们开始建立一个网站吧。
