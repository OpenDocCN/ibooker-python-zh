# 第十四章\. 数据库、数据科学和少量 AI

# 预览

本章讨论如何使用 FastAPI 存储和检索数据。它扩展了第十章中简单的 SQLite 示例，包括以下内容：

+   其他开源数据库（关系型和非关系型）

+   SQLAlchemy 的更高级用法

+   更好的错误检查

# 数据存储替代方案

###### 注意

不幸地，“database”这个术语被用来指代三件事：

+   服务器 *type*，如 PostgreSQL、SQLite 或 MySQL

+   运行中的 *server* 实例

+   在该服务器上的 *collection of tables*

为了避免混淆——将上述最后一个项目的实例称为“PostgreSQL 数据库数据库数据库”，我会附加其他术语以表明我的意思。

网站的典型后端是数据库。网站和数据库如花生酱和果冻一般搭配，尽管你可能会考虑其他存储数据的方式（或者把花生酱配上泡菜），但在本书中我们将专注于数据库。

数据库处理了许多问题，否则您将不得不用代码自行解决，例如以下问题：

+   多重访问

+   索引

+   数据一致性

数据库的一般选择如下：

+   关系数据库，带有 SQL 查询语言

+   非关系型数据库，具有各种查询语言

# 关系数据库和 SQL

Python 有一个名为 [DB-API](https://oreil.ly/StbE4) 的标准关系 API 定义，由所有主要数据库的 Python 驱动程序包支持。表 14-1 列出了一些显著的关系数据库及其主要的 Python 驱动程序包。

表 14-1\. 关系数据库和 Python 驱动程序

| 数据库 | Python 驱动程序 |
| --- | --- |
| 开源 |
| [SQLite](https://www.sqlite.org) | [sqlite3](https://oreil.ly/TNNaA) |
| [PostgreSQL](https://www.postgresql.org) | [psycopg2](https://oreil.ly/nLn5x) 和 [asyncpg](https://oreil.ly/90pvK) |
| [MySQL](https://www.mysql.com) | [MySQLdb](https://oreil.ly/yn1fn) 和 [PyMySQL](https://oreil.ly/Cmup-) |
| 商业 |
| [Oracle](https://www.oracle.com) | [python-oracledb](https://oreil.ly/gynvX) |
| [SQL Server](https://www.microsoft.com/en-us/sql-server) | [pyodbc](https://oreil.ly/_UEYq) 和 [pymssql](https://oreil.ly/FkKUn) |
| [IBM Db2](https://www.ibm.com/products/db2) | [ibm_db](https://oreil.ly/3uwpD) |

Python 主要用于关系数据库和 SQL 的软件包如下：

[SQLAlchemy](https://www.sqlalchemy.org)

可以在多个层次上使用的功能齐全的库

[SQLModel](https://sqlmodel.tiangolo.com)

FastAPI 的作者结合了 SQLAlchemy 和 Pydantic 的组合

[Records](https://github.com/kennethreitz/records)

来自 Requests 包的作者，一个简单的查询 API

## SQLAlchemy

最流行的 Python SQL 包是 SQLAlchemy。尽管许多关于 SQLAlchemy 的解释只讨论其 ORM，但它有多个层次，我将从底层向上讨论这些。

### Core

SQLAlchemy 的基础，称为 *Core*，包括以下内容：

+   实现了 DB-API 标准的`Engine`对象

+   表达 SQL 服务器类型、驱动程序以及该服务器上特定数据库集合的 URL

+   客户端-服务器连接池

+   事务（`COMMIT`和`ROLLBACK`）

+   各种数据库类型之间的 SQL *方言* 差异

+   直接 SQL（文本字符串）查询

+   在 SQLAlchemy 表达语言中进行查询

其中一些功能，如方言处理，使 SQLAlchemy 成为处理各种服务器类型的首选包。你可以用它来执行纯粹的 DB-API SQL 语句或使用 SQLAlchemy 表达语言。

到目前为止我一直在使用原始的 DB-API SQLite 驱动程序，并将继续使用。但对于更大的站点或可能需要利用特殊服务器功能的站点，SQLAlchemy（使用基本的 DB-API、SQLAlchemy 表达语言或完整的 ORM）是非常值得使用的。

### SQLAlchemy 表达语言

SQLAlchemy 表达语言*不是*ORM，而是另一种表达对关系表查询的方式。它将底层存储结构映射到像`Table`和`Column`这样的 Python 类，并将操作映射到像`select()`和`insert()`这样的 Python 方法。这些函数转换为普通的 SQL 字符串，你可以访问它们来查看发生了什么。该语言与 SQL 服务器类型无关。如果你觉得 SQL 困难，这可能值得一试。

让我们比较几个例子。示例 14-1 显示了纯 SQL 版本。

##### 示例 14-1\. 数据/explorer.py 中`get_one()`的直接 SQL 代码

```py
def get_one(name: str) -> Explorer:
    qry = "select * from explorer where name=:name"
    params = {"name": name}
    curs.execute(qry, params)
    return row_to_model(curs.fetchone())
```

示例 14-2 显示了部分 SQLAlchemy 表达语言的等效内容，用于设置数据库、构建表并执行插入。

##### 示例 14-2\. SQLAlchemy 表达语言用于`get_one()`功能

```py
from sqlalchemy import Metadata, Table, Column, Text
from sqlalchemy import connect, insert

conn = connect("sqlite:///cryptid.db")
meta = Metadata()
explorer_table = Table(
    "explorer",
    meta,
    Column("name", Text, primary_key=True),
    Column("country", Text),
    Column("description", Text),
    )
insert(explorer_table).values(
    name="Beau Buffette",
    country="US",
    description="...")
```

要获取更多示例，一些备选[文档](https://oreil.ly/ZGCHv)比官方页面更易读。

### ORM

ORM 将查询表达为领域数据模型的术语，而不是数据库机器底层的关系表和 SQL 逻辑。官方[文档](https://oreil.ly/x4DCi)详细介绍了所有细节。ORM 比 SQL 表达语言复杂得多。偏爱完全*面向对象*模式的开发人员通常更喜欢 ORM。

许多关于 FastAPI 的书籍和文章在数据库部分直接跳到 SQLAlchemy 的 ORM。我理解其吸引力，但也知道这要求你学习另一种抽象。SQLAlchemy 是一个优秀的库，但如果其抽象不总是适用，那么你就会遇到两个问题。最简单的解决方案可能是直接使用 SQL，如果 SQL 变得过于复杂再转向表达语言或 ORM。

## SQLModel

FastAPI 的作者结合了 FastAPI、Pydantic 和 SQLAlchemy 的各个方面，创建了[SQLModel](https://sqlmodel.tiangolo.com)。它重新利用了一些来自网络世界的开发技术到关系数据库中。SQLModel 将 SQLAlchemy 的 ORM 与 Pydantic 的数据定义和验证结合起来。

## SQLite

我在 第十章 中介绍了 SQLite，将其用于数据层示例。它是公有领域的——你不能找到比这更开源的了。SQLite 在每个浏览器和每个智能手机中都被使用，使其成为世界上部署最广泛的软件包之一。在选择关系数据库时，它经常被忽视，但是有可能多个 SQLite “服务器” 也可以支持一些大型服务，就像一个强大的服务器一样，例如 PostgreSQL。

## PostgreSQL

在关系数据库的早期，IBM 的 System R 是先驱，而分支为新市场而战——主要是开源的 Ingres 对商业的 Oracle。Ingres 采用了名为 QUEL 的查询语言，而 System R 采用了 SQL。尽管 QUEL 被一些人认为比 SQL 更好，但是 Oracle 将 SQL 作为标准，再加上 IBM 的影响力，帮助推动了 Oracle 和 SQL 的成功。

几年后，Michael Stonebraker 回归，将 Ingres 迁移到 [PostgreSQL](https://www.postgresql.org)。如今，开源开发者倾向于选择 PostgreSQL，尽管几年前 MySQL 很受欢迎，现在仍然存在。

## EdgeDB

尽管 SQL 多年来取得了成功，但它确实存在一些设计缺陷，使得查询变得笨拙。与 SQL 基于的数学理论（由 E. F. Codd 提出的 *关系演算*）不同，SQL 语言设计本身不具有 *可组合性*。主要是这意味着很难在较大的查询中嵌套查询，导致代码更加复杂和冗长。

所以，就只是为了好玩，我在这里加入了一个新的关系数据库。[EdgeDB](https://www.edgedb.com)（用 Python 写的！）是由 Python 的 asyncio 的作者编写的。它被描述为 *Post-SQL* 或 *graph-relational*。在内部，它使用 PostgreSQL 处理繁重的系统任务。Edge 的贡献是 [EdgeQL](https://oreil.ly/sdK4J)：一种旨在避免 SQL 中那些棘手的边缘的新查询语言；它实际上被转换为 SQL 以供 PostgreSQL 执行。[“我对 EdgeDB 的体验” by Ivan Daniluk](https://oreil.ly/ciNfg) 方便地比较了 EdgeQL 和 SQL。可读的图解[官方文档](https://oreil.ly/ce6y3)与书籍 *Dracula* 相呼应。

EdgeQL 是否会超越 EdgeDB 并成为 SQL 的替代品？时间会告诉我们。

# 非关系（NoSQL）数据库

在开源 NoSQL 或 NewSQL 领域的主要人物在 表 14-2 中列出。

表 14-2\. NoSQL 数据库和 Python 驱动程序

| 数据库 | Python 驱动程序 |
| --- | --- |
| [Redis](https://redis.io) | [redis-py](https://github.com/redis/redis-py) |
| [MongoDB](https://www.mongodb.com) | [PyMongo](https://pymongo.readthedocs.io), [Motor](https://oreil.ly/Cmgtl) |
| [Apache Cassandra](https://cassandra.apache.org) | [DataStax Apache Cassandra 驱动](https://github.com/datastax/python-driver) |
| [Elasticsearch](https://www.elastic.co/elasticsearch) | [Python Elasticsearch Client](https://oreil.ly/e_bDI) |

有时 *NoSQL* 的意思是字面上的 *no SQL*，但有时是 *not only SQL*。关系型数据库对数据强制实施结构，通常可视化为带有列字段和数据行的矩形表，类似于电子表格。为了减少冗余并提高性能，关系型数据库使用 *normal forms*（数据和结构的规则）进行 *normalized*，例如只允许每个单元格（行/列交叉点）有一个值。

NoSQL 数据库放宽了这些规则，有时允许在单个数据行中跨列/字段使用不同类型。通常，*schemas*（数据库设计）可以是杂乱的结构，就像您可以在 JSON 或 Python 中表示的那样，而不是关系型的盒子。

## Redis

*Redis* 是一个完全运行在内存中的数据结构服务器，虽然它可以保存到磁盘并从磁盘恢复。它与 Python 自身的数据结构非常匹配，因此变得非常流行。

## MongoDB

*MongoDB* 类似于 NoSQL 服务器中的 PostgreSQL。*Collection* 相当于 SQL 表，*document* 相当于 SQL 表中的一行。另一个区别，也是使用 NoSQL 数据库的主要原因，是您不需要定义文档的结构。换句话说，没有固定的 *schema*。文档类似于 Python 字典，任何字符串都可以作为键。

## Cassandra

Cassandra 是一个可以分布在数百个节点上的大规模数据库。它是用 Java 编写的。

一种名为 [ScyllaDB](https://www.scylladb.com) 的替代数据库是用 C++ 编写的，声称与 Cassandra 兼容但性能更好。

## Elasticsearch

[Elasticsearch](https://www.elastic.co/elasticsearch) 更像是数据库索引，而不是数据库本身。它通常用于全文搜索。

# SQL 数据库中的 NoSQL 特性

正如前面所述，关系型数据库传统上是规范化的——受到称为 *normal forms* 的不同级别规则的约束。一个基本规则是每个单元格中的值（行列交叉点）必须是 *scalar*（无数组或其他结构）。

NoSQL（或 *document*）数据库直接支持 JSON，并且通常是如果您有“不均匀”或“杂乱”数据结构的唯一选择。它们通常是 *denormalized*：所有需要的文档数据都包含在该文档中。在 SQL 中，您经常需要跨表进行 *join* 来构建完整的文档。

然而，SQL 标准的最近修订允许在关系型数据库中存储 JSON 数据。一些关系型数据库现在允许您在表单元格中存储复杂（非标量）数据，并在其中进行搜索和索引。JSON 函数以各种方式得到支持，包括 [SQLite](https://oreil.ly/h_FNn)、[PostgreSQL](https://oreil.ly/awYrc)、[MySQL](https://oreil.ly/OA_sT)、[Oracle](https://oreil.ly/osOYk) 等等。

具有 JSON 的 SQL 可能是两者兼得的最佳选择。SQL 数据库存在已经很长时间了，并且具有非常有用的功能，如外键和二级索引。此外，SQL 在某种程度上相当标准化，而 NoSQL 查询语言则各不相同。

最后，新的数据设计和查询语言正在尝试结合 SQL 和 NoSQL 的优势，就像我之前提到的 EdgeQL 一样。

因此，如果您的数据无法适应矩形关系框，请考虑 NoSQL 数据库，支持 JSON 的关系数据库，或者“Post-SQL”数据库。

# 数据库负载测试

本书主要讲述的是 FastAPI，但网站经常与数据库相关联。

本书中的数据示例都很小。要真正对数据库进行压力测试，数百万条数据将是很好的选择。与其考虑要添加的内容，不如使用像[Faker](https://faker.readthedocs.io)这样的 Python 包更容易。Faker 可以快速生成许多类型的数据—名称、地点或您定义的特殊类型。

在示例 14-3 中，Faker 生成名称和国家，然后由`load()`加载到 SQLite 中。

##### 示例 14-3\. 在 test_load.py 中加载虚拟探险家

```py
from faker import Faker
from time import perf_counter

def load():
    from error import Duplicate
    from data.explorer import create
    from model.explorer import Explorer

    f = Faker()
    NUM = 100_000
    t1 = perf_counter()
    for row in range(NUM):
        try:
            create(Explorer(name=f.name(),
                country=f.country(),
                description=f.description))
        except Duplicate:
            pass
    t2 = perf_counter()
    print(NUM, "rows")
    print("write time:", t2-t1)

def read_db():
    from data.explorer import get_all

    t1 = perf_counter()
    _ = get_all()
    t2 = perf_counter()
    print("db read time:", t2-t1)

def read_api():
    from fastapi.testclient import TestClient
    from main import app

    t1 = perf_counter()
    client = TestClient(app)
    _ = client.get("/explorer/")
    t2 = perf_counter()
    print("api read time:", t2-t1)

load()
read_db()
read_db()
read_api()
```

在`load()`中捕获`Duplicate`异常并忽略它，因为 Faker 从一个有限的列表中生成名称，偶尔可能会重复。所以结果可能少于加载的 10 万个探险家。

此外，您两次调用了`read_db()`，以便在 SQLite 执行查询时消除任何启动时间。然后`read_api()`的时间应该是公平的。示例 14-4 启动它。

##### 示例 14-4\. 测试数据库查询性能

```py
$ python test_load.py
100000 rows
write time: 14.868232927983627
db read time: 0.4025074450764805
db read time: 0.39750714192632586
api read time: 2.597553930943832
```

所有探险家的 API 读取时间比数据层的读取时间慢得多。其中一些可能是由于 FastAPI 将响应转换为 JSON 的开销。此外，写入数据库的初始时间并不是很快。它一次写入一个探险家，因为数据层 API 有一个单独的`create()`函数，但没有`create_many()`函数；在读取方面，API 可以返回一个(`get_one()`)或所有(`get_all()`)。因此，如果您想要进行批量加载，可能最好添加一个新的数据加载函数和一个新的 Web 端点（带有受限制的授权）。

此外，如果您期望数据库中的任何表增长到 10 万行，也许您不应该允许随机用户在一个 API 调用中获取所有这些行。分页会很有用，或者一种从表中下载单个 CSV 文件的方法。

# 数据科学与人工智能

Python 已经成为数据科学总体上以及机器学习特别是最突出的语言。因此需要大量的数据处理工作，而 Python 擅长于此。

有时开发人员会使用[外部工具](https://oreil.ly/WFHo9)如 pandas 来进行在 SQL 中过于棘手的数据操作。

[PyTorch](https://pytorch.org) 是最流行的 ML 工具之一，因为它充分利用了 Python 在数据处理方面的优势。底层计算可能使用 C 或 C++ 来提高速度，但 Python 或 Go 更适合“更高级”的数据集成任务。（The [Mojo](https://www.modular.com/mojo) 语言，Python 的超集，如果计划成功，可能会处理高低端任务。虽然它是一种通用语言，但专门解决了 AI 开发中的某些当前复杂性。）

一个名为 [Chroma](https://www.trychroma.com) 的新 Python 工具是一个数据库，类似于 SQLite，但专门针对机器学习，特别是大型语言模型（LLMs）。阅读 [Getting Started page](https://oreil.ly/W59nn) 来，你懂的，开始使用。

尽管 AI 开发复杂且发展迅速，但你可以在自己的机器上使用 Python 尝试一些 AI，而不需要像 GPT-4 和 ChatGPT 那样花费巨额资金。让我们构建一个小型 FastAPI 网页接口到一个小型 AI 模型。

###### 注意

*Model* 在 AI 和 Pydantic/FastAPI 中有不同的含义。在 Pydantic 中，一个 model 是一个捆绑相关数据字段的 Python 类。AI models 则涵盖了广泛的技术，用于确定数据中的模式。

[Hugging Face](https://huggingface.co) 提供免费的 AI models、数据集和 Python 代码供使用。首先，安装 PyTorch 和 Hugging Face 代码：

```py
$ pip install torch torchvision
$ pip install transformers
```

Example 14-5 展示了一个 FastAPI 应用程序，它使用 Hugging Face 的 transformers 模块访问预训练的中型开源机器语言模型，并尝试回答你的提示。（这是从 YouTube 频道 CodeToTheMoon 的命令行示例中改编的。）

##### 示例 14-5\. 顶层 LLM 测试（ai.py）

```py
from fastapi import FastAPI

app = FastAPI()

from transformers import (AutoTokenizer,
    AutoModelForSeq2SeqLM, GenerationConfig)
model_name = "google/flan-t5-base"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSeq2SeqLM.from_pretrained(model_name)
config = GenerationConfig(max_new_tokens=200)

@app.get("/ai")
def prompt(line: str) -> str:
    tokens = tokenizer(line, return_tensors="pt")
    outputs = model.generate(**tokens,
        generator_config=config)
    result = tokenizer.batch_decode(outputs,
        skip_special_tokens=True)
    return result[0]
```

使用 `uvicorn ai:app` 运行此程序（像往常一样，首先确保你没有另一个仍在 `localhost` 端口 `8000` 上运行的网络服务器）。在 */ai* 端点输入问题并获得答案，如此（注意 HTTPie 查询参数的双 `==`）：

```py
$ http -b localhost:8000/ai line=="What are you?"
"a sailor"
```

这是一个相当小的模型，正如你所见，它并不能特别好地回答问题。我尝试了其他提示（`line` 参数）并得到了同样值得注意的答案：

+   Q: 猫比狗更好吗？

+   A: 不

+   Q: 大脚怪早餐吃什么？

+   A: 一只鱿鱼

+   Q: 谁会从烟囱里下来？

+   A: 一只尖叫的猪

+   Q: 约翰·克利斯在什么组里？

+   A: 披头士乐队

+   Q: 什么有尖尖的牙齿？

+   A: 一个泰迪熊

这些问题在不同时间可能会有不同的答案！有一次同一端点说大脚怪早餐吃沙子。在 AI 术语中，像这样的答案被称为 *幻觉*。你可以通过使用像 `google/flan-75-xl` 这样的更大模型来获得更好的答案，但在个人电脑上下载模型数据和响应会花费更长时间。当然，像 ChatGPT 这样的模型是使用它们能找到的所有数据训练的（使用每个 CPU、GPU、TPU 和任何其他类型的 PU），并且会给出优秀的答案。

# 回顾

本章扩展了我们在第十章中介绍的 SQLite 的用法，涵盖了其他 SQL 数据库，甚至 NoSQL 数据库。它还展示了一些 SQL 数据库如何通过 JSON 支持实现 NoSQL 技巧。最后，它讨论了随着机器学习持续爆炸性增长，数据库和特殊数据工具的用途变得更加重要。
