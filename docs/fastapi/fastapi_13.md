# 第十章：数据层

> 如果我没记错，我认为 Data 是该节目中的喜剧缓解角色。
> 
> Brent Spiner，《星际迷航：下一代》

# 预览

本章最终为我们网站的数据创建了一个持久的家，最终连接了三个层次。它使用关系数据库 SQLite，并引入了 Python 的数据库 API，名为 DB-API。第十四章更详细地介绍了数据库，包括 SQLAlchemy 包和非关系数据库。

# DB-API

20 多年来，Python 已经包含了一个名为 DB-API 的关系数据库接口的基本定义：[PEP 249](https://oreil.ly/4Gp9T)。任何编写 Python 关系数据库驱动程序的人都应至少包括对 DB-API 的支持，尽管可能包括其他特性。

这些是主要的 DB-API 函数：

+   使用`connect()`创建到数据库的连接`conn`。

+   使用`conn.cursor()`创建一个游标`curs`。

+   使用`curs.execute(stmt)`执行 SQL 字符串`stmt`。

`execute...()`函数运行一个带有可选参数的 SQL 语句`*stmt*`字符串，列在此处：

+   如果没有参数，则使用`execute(*stmt*)`

+   `execute(*stmt*, *params*)`，使用单个序列（列表或元组）或字典中的参数`*params*`

+   `executemany(*stmt*, *params_seq*)`，在序列`*params_seq*`中有多个参数组。

有五种指定参数的方法，并非所有数据库驱动程序都支持。如果我们有一个以`"select * from creature where"`开头的语句`*stmt*`，并且我们想要为生物的`name`或`country`指定字符串参数，那么剩余的`*stmt*`字符串及其参数看起来像表 10-1 中的那些。

表 10-1。指定语句和参数

| 类型 | 语句部分 | 参数部分 |
| --- | --- | --- |
| qmark | `name=? or country=?` | `(*name*, *country*)` |
| numeric | `name=:0 or country=:1` | `(*name*, *country*)` |
| format | `name=%s or country=%s` | `(*name*, *country*)` |
| named | `name=:name or country=:country` | `{"name": *name*, "country": *​coun⁠try*}` |
| pyformat | `name=%(name)s or country=%(country)s` | `{"name": *name*, "country": *​coun⁠try*}` |

前三个采用元组参数，其中参数顺序与语句中的`?`，`:N`或`%s`匹配。最后两个采用字典，其中键与语句中的名称匹配。

因此，命名样式的完整调用将如示例 10-1 所示。

##### 示例 10-1。使用命名样式参数。

```py
stmt = """select * from creature where
 name=:name or country=:country"""
params = {"name": "yeti", "country": "CN"}
curs.execute(stmt, params)
```

对于 SQL `INSERT`，`DELETE`和`UPDATE`语句，`execute()`的返回值告诉您它的工作原理。对于`SELECT`，您遍历返回的数据行（作为 Python 元组），使用`fetch`方法：

+   `fetchone()`返回一个元组，或者`None`。

+   `fetchall()`返回一个元组序列。

+   `fetchmany(*num*)`最多返回`*num*`个元组。

# SQLite

Python 包括对一个数据库（[SQLite](https://www.sqlite.org)）的支持，使用其标准包中的 [sqlite3](https://oreil.ly/CcYtJ) 模块。

SQLite 是不寻常的：它没有单独的数据库服务器。所有的代码都在一个库中，并且存储在一个单独的文件中。其他数据库运行独立的服务器，客户端通过特定的协议通过 TCP/IP 与它们通信。让我们将 SQLite 作为这个网站的第一个物理数据存储。第十四章 将包括其他数据库，关系型和非关系型，以及更高级的包如 SQLAlchemy 和像 ORM 这样的技术。

首先，我们需要定义我们在网站中使用的数据结构（*模型*）如何在数据库中表示。到目前为止，我们唯一的模型是简单而相似的，但并非完全相同：`Creature` 和 `Explorer`。随着我们考虑到更多要处理的事物并允许数据不断演变而无需大规模的代码更改，它们将会改变。

示例 10-2 显示了裸的 DB-API 代码和 SQL 来创建和处理第一个表。它使用了 `*name*` 形式的命名参数字符串（值被表示为 `*name*`），这是 sqlite3 包支持的。

##### 示例 10-2\. 使用 sqlite3 创建文件 data/creature.py

```py
import sqlite3
from model.creature import Creature

DB_NAME = "cryptid.db"
conn = sqlite3.connect(DB_NAME)
curs = conn.cursor()

def init():
    curs.execute("create table creature(name, description, country, area, aka)")

def row_to_model(row: tuple) -> Creature:
    name, description, country, area, aka = row
    return Creature(name, description, country, area, aka)

def model_to_dict(creature: Creature) -> dict:
    return creature.dict()

def get_one(name: str) -> Creature:
    qry = "select * from creature where name=:name"
    params = {"name": name}
    curs.execute(qry, params)
    row = curs.fetchone()
    return row_to_model(row)

def get_all(name: str) -> list[Creature]:
    qry = "select * from creature"
    curs.execute(qry)
    rows = list(curs.fetchall())
    return [row_to_model(row) for row in rows]

def create(creature: Creature):
    qry = """insert into creature values
 (:name, :description, :country, :area, :aka)"""
    params = model_to_dict(creature)
    curs.execute(qry, params)

def modify(creature: Creature):
    return creature

def replace(creature: Creature):
    return creature

def delete(creature: Creature):
    qry = "delete from creature where name = :name"
    params = {"name": creature.name}
    curs.execute(qry, params)
```

在顶部附近，`init()` 函数连接到 sqlite3 和数据库假的 *cryptid.db*。它将此存储在变量 `conn` 中；这在 *data/creature.py* 模块内是全局的。接下来，`curs` 变量是用于迭代通过执行 SQL `SELECT` 语句返回的数据的 *cursor*；它也是该模块的全局变量。

两个实用函数在 Pydantic 模型和 DB-API 之间进行转换：

+   `row_to_model()` 将由 *fetch* 函数返回的元组转换为模型对象。

+   `model_to_dict()` 将 Pydantic 模型转换为字典，适合用作 *named* 查询参数。

到目前为止，在每一层下（Web → Service → Data）存在的虚假的 CRUD 函数现在将被替换。它们仅使用纯 SQL 和 sqlite3 中的 DB-API 方法。

# 布局

到目前为止，（虚假的）数据已经被分步修改：

1.  在 第八章，我们在 *web/creature.py* 中制作了虚假的 `*creatures*` 列表。

1.  在 第八章，我们在 *web/explorer.py* 中制作了虚假的 `*explorers*` 列表。

1.  在 第九章，我们将假的 `*creatures*` 移动到 *service/creature.py*。

1.  在 第九章，我们将假的 `*explorers*` 移动到 *service/explorer.py*。

现在数据已经最后一次移动，到 *data/creature.py*。但它不再是虚假的：它是真实的数据，存储在 SQLite 数据库文件 *cryptids.db* 中。生物数据，由于缺乏想象力，再次存储在此数据库中的 SQL 表 `creature` 中。

一旦保存了这个新文件，Uvicorn 应该从你的顶级 *main.py* 重新启动，它调用 *web/creature.py*，这将调用 *service/creature.py*，最终到这个新的 *data/creature.py*。

# 让它工作

我们有一个小问题：这个模块从未调用它的 `init()` 函数，因此没有 SQLite 的 `conn` 或 `curs` 可供其他函数使用。这是一个配置问题：如何在启动时提供数据库信息？可能的解决方案包括以下几种：

+   将数据库信息硬编码到代码中，就像 Example 10-2 中那样。

+   将信息传递到各层。但这样做会违反层次间的分离原则；Web 层和服务层不应了解数据层的内部。

+   从不同的外部源传递信息，比如

    +   配置文件

    +   一个环境变量

环境变量很简单，并且得到了像 [Twelve-Factor App](https://12factor.net/config) 这样的建议的支持。如果环境变量未指定，代码可以包含一个默认值。这种方法也可以用于测试，以提供一个与生产环境不同的测试数据库。

在 Example 10-3 中，让我们定义一个名为 `CRYPTID_SQLITE_DB` 的环境变量，默认值为 `cryptid.db`。创建一个名为 *data/init.py* 的新文件用于新数据库初始化代码，以便它也可以在探险者代码中重用。

##### Example 10-3\. 新数据初始化模块 data/init.py

```py
"""Initialize SQLite database"""

import os
from pathlib import Path
from sqlite3 import connect, Connection, Cursor, IntegrityError

conn: Connection | None = None
curs: Cursor | None = None

def get_db(name: str|None = None, reset: bool = False):
    """Connect to SQLite database file"""
    global conn, curs
    if conn:
        if not reset:
            return
        conn = None
    if not name:
        name = os.getenv("CRYPTID_SQLITE_DB")
        top_dir = Path(__file__).resolve().parents[1] # repo top
        db_dir = top_dir / "db"
        db_name = "cryptid.db"
        db_path = str(db_dir / db_name)
        name = os.getenv("CRYPTID_SQLITE_DB", db_path)
    conn = connect(name, check_same_thread=False)
    curs = conn.cursor()

get_db()
```

Python 模块是一个 *单例*，即使多次导入也只调用一次。因此，当首次导入 *init.py* 时，初始化代码只会运行一次。

最后，在 Example 10-4 中修改 *data/creature.py* 使用这个新模块：

+   主要是，删除第 4 至 8 行代码。

+   哦，还要在第一次创建 `creature` 表的时候！

+   表字段都是 SQL 的 `text` 字符串。这是 SQLite 的默认列类型（不像大多数 SQL 数据库），所以之前不需要显式包含 `text`，但明确写出来也无妨。

+   `if not exists` 避免在表已创建后覆盖它。

+   `name` 字段是该表的显式 `primary key`。如果该表存储了大量探险者数据，这个键将对快速查找至关重要。另一种选择是可怕的 *表扫描*，在这种情况下，数据库代码需要查看每一行，直到找到 `name` 的匹配项。

##### Example 10-4\. 将数据库配置添加到 data/creature.py

```py
from .init import conn, curs
from model.creature import Creature

curs.execute("""create table if not exists creature(
 name text primary key,
 description text,
 country text,
 area text,
 aka text)""")

def row_to_model(row: tuple) -> Creature:
    (name, description, country, area, aka) = row
    return Creature(name, description, country, area, aka)

def model_to_dict(creature: Creature) -> dict:
    return creature.dict()

def get_one(name: str) -> Creature:
    qry = "select * from creature where name=:name"
    params = {"name": name}
    curs.execute(qry, params)
    return row_to_model(curs.fetchone())

def get_all() -> list[Creature]:
    qry = "select * from creature"
    curs.execute(qry)
    return [row_to_model(row) for row in curs.fetchall()]

def create(creature: Creature) -> Creature:
    qry = "insert into creature values"
          "(:name, :description, :country, :area, :aka)"
    params = model_to_dict(creature)
    curs.execute(qry, params)
    return get_one(creature.name)

def modify(creature: Creature) -> Creature:
    qry = """update creature
 set country=:country,
 name=:name,
 description=:description,
 area=:area,
 aka=:aka
 where name=:name_orig"""
    params = model_to_dict(creature)
    params["name_orig"] = creature.name
    _ = curs.execute(qry, params)
    return get_one(creature.name)

def delete(creature: Creature) -> bool:
    qry = "delete from creature where name = :name"
    params = {"name": creature.name}
    res = curs.execute(qry, params)
    return bool(res)
```

通过从 *init.py* 中导入 `conn` 和 `curs`，就不再需要 *data/creature.py* 自己导入 sqlite3 —— 除非有一天需要调用 `conn` 或 `curs` 对象之外的另一个 sqlite3 方法。

再次，这些更改应该能够促使 Uvicorn 重新加载所有内容。从现在开始，使用迄今为止看到的任何方法进行测试（如 HTTPie 和其他工具，或自动化的 */docs* 表单），都将显示持久化的数据。如果你添加了一个生物，下次获取所有生物时它将会存在。

让我们对 Example 10-5 中的探险者做同样的事情。

##### Example 10-5\. 将数据库配置添加到 data/explorer.py

```py
from .init import curs
from model.explorer import Explorer

curs.execute("""create table if not exists explorer(
 name text primary key,
 country text,
 description text)""")

def row_to_model(row: tuple) -> Explorer:
    return Explorer(name=row[0], country=row[1], description=row[2])

def model_to_dict(explorer: Explorer) -> dict:
    return explorer.dict() if explorer else None

def get_one(name: str) -> Explorer:
    qry = "select * from explorer where name=:name"
    params = {"name": name}
    curs.execute(qry, params)
    return row_to_model(curs.fetchone())

def get_all() -> list[Explorer]:
    qry = "select * from explorer"
    curs.execute(qry)
    return [row_to_model(row) for row in curs.fetchall()]

def create(explorer: Explorer) -> Explorer:
    qry = """insert into explorer (name, country, description)
 values (:name, :country, :description)"""
    params = model_to_dict(explorer)
    _ = curs.execute(qry, params)
    return get_one(explorer.name)

def modify(name: str, explorer: Explorer) -> Explorer:
    qry = """update explorer
 set country=:country,
 name=:name,
 description=:description
 where name=:name_orig"""
    params = model_to_dict(explorer)
    params["name_orig"] = explorer.name
    _ = curs.execute(qry, params)
    explorer2 = get_one(explorer.name)
    return explorer2

def delete(explorer: Explorer) -> bool:
    qry = "delete from explorer where name = :name"
    params = {"name": explorer.name}
    res = curs.execute(qry, params)
    return bool(res)
```

# 测试！

这是大量没有测试的代码。一切都有效吗？如果一切都有效，我会感到惊讶。所以，让我们设置一些测试。

在 *test* 目录下创建这些子目录：

单元

在一个层内

完整

跨所有层

你应该先写和运行哪种类型的测试？大多数人先写自动化单元测试；它们规模较小，而且其他所有层的组件可能尚不存在。本书的开发是自顶向下的，现在我们正在完成最后一层。此外，我们在第 8 和 9 章节做了手动测试（使用 HTTPie 和朋友们）。这些测试帮助快速暴露出错误和遗漏；自动化测试确保你以后不会反复犯同样的错误。因此，我建议以下操作：

+   在你第一次编写代码时进行一些手动测试

+   在你修复了 Python 语法错误之后的单元测试

+   在你的数据流经过所有层后进行完整测试

## 完整测试

这些调用 web 端点，通过 Service 到 Data 的代码电梯，然后再次上升。有时这些被称为 *端到端* 或 *合同* 测试。

### 获取所有探险家

在还不知道测试是否会被食人鱼侵袭的情况下，勇敢的志愿者是 示例 10-6。

##### 示例 10-6\. 获取所有探险家的测试

```py
$ http localhost:8000/explorer
HTTP/1.1 405 Method Not Allowed
allow: POST
content-length: 31
content-type: application/json
date: Mon, 27 Feb 2023 20:05:18 GMT
server: uvicorn

{
    "detail": "Method Not Allowed"
}
```

唉呀！发生了什么事？

哦。测试要求 `/explorer`，而不是 `/explorer/`，并且没有适用于 URL */explorer*（没有最终斜杠的情况下）的 `GET` 方法路径函数。在 *web/explorer.py* 中，`get_all()` 路径函数的路径装饰器如下所示：

```py
@router.get("/")
```

这，再加上之前的代码

```py
router = APIRouter(prefix = "/explorer")
```

意味着这个 `get_all()` 路径函数提供了一个包含 */explorer/* 的 URL。

示例 10-7 高兴地展示，你可以为一个路径函数添加多个路径装饰器。

##### 示例 10-7\. 为 `get_all()` 路径函数添加一个非斜杠路径装饰器

```py
@router.get("")
@router.get("/")
def get_all() -> list[Explorer]:
    return service.get_all()
```

在示例 10-8 和 10-9 中的两个网址的测试。

##### 示例 10-8\. 测试非斜杠端点

```py
$ http localhost:8000/explorer
HTTP/1.1 200 OK
content-length: 2
content-type: application/json
date: Mon, 27 Feb 2023 20:12:44 GMT
server: uvicorn

[]
```

##### 示例 10-9\. 测试斜杠端点

```py
$ http localhost:8000/explorer/
HTTP/1.1 200 OK
content-length: 2
content-type: application/json
date: Mon, 27 Feb 2023 20:14:39 GMT
server: uvicorn

[]
```

现在这两者都正常工作后，创建一个探险家，并在之后重新测试。示例 10-10 尝试这样做，但有一个情节转折。

##### 示例 10-10\. 测试创建探险家时的输入错误

```py
$ http post localhost:8000/explorer name="Beau Buffette", contry="US"
HTTP/1.1 422 Unprocessable Entity
content-length: 95
content-type: application/json
date: Mon, 27 Feb 2023 20:17:45 GMT
server: uvicorn

{
    "detail": [
        {
            "loc": [
                "body",
                "country"
            ],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```

我拼错了 `country`，虽然我的拼写通常是无可挑剔的。Pydantic 在 Web 层捕获了这个错误，返回了 `422` 的 HTTP 状态码和问题的描述。一般来说，如果 FastAPI 返回 `422`，那么 Pydantic 找到了问题的源头。`"loc"` 部分指出了错误发生的位置：字段 `"country"` 丢失，因为我是一个笨拙的打字员。

修正拼写并在 示例 10-11 中重新测试。

##### 示例 10-11\. 使用正确的值创建一个探险家

```py
$ http post localhost:8000/explorer name="Beau Buffette" country="US"
HTTP/1.1 201 Created
content-length: 55
content-type: application/json
date: Mon, 27 Feb 2023 20:20:49 GMT
server: uvicorn

{
    "name": "Beau Buffette,",
    "country": "US",
    "description": ""
}
```

这次调用返回了 `201` 状态码，这在资源创建时是传统的（所有 `2*xx*` 状态码都被认为表示成功，其中最通用的是 `200`）。响应还包含刚刚创建的 `Explorer` 对象的 JSON 版本。

现在回到最初的测试：Beau 会出现在获取所有探险家的测试中吗？示例 10-12 回答了这个激动人心的问题。

##### 示例 10-12\. 最新的`create()`工作了吗？

```py
$ http localhost:8000/explorer
HTTP/1.1 200 OK
content-length: 57
content-type: application/json
date: Mon, 27 Feb 2023 20:26:26 GMT
server: uvicorn

[
    {
        "name": "Beau Buffette",
        "country": "US",
        "description": ""
    }
]
```

耶。

### 获取一个资源管理器

现在，如果您尝试使用 Get One 端点（示例 10-13）查找 Beau 会发生什么？

##### 示例 10-13\. 测试获取单个端点

```py
$ http localhost:8000/explorer/"Beau Buffette"
HTTP/1.1 200 OK
content-length: 55
content-type: application/json
date: Mon, 27 Feb 2023 20:28:48 GMT
server: uvicorn

{
    "name": "Beau Buffette",
    "country": "US",
    "description": ""
}
```

我使用引号保留了名字中的第一个和最后一个名字之间的空格。在 URL 中，您也可以使用`Beau%20Buffette`；`%20`是空格字符在 ASCII 中的十六进制代码。

### 缺少和重复的数据

到目前为止，我忽略了两个主要的错误类别：

缺少数据

如果您尝试通过数据库中不存在的名称获取、修改或删除资源管理器。

重复数据

如果您尝试多次使用相同的名称创建资源管理器。

那么，如果您请求一个不存在的或重复的资源管理器会发生什么？到目前为止，代码过于乐观，异常将从深渊中冒出。

我们的朋友 Beau 刚刚被添加到数据库中。想象一下，他的邪恶克隆人（与他同名）密谋在一个黑暗的夜晚替换他，使用 示例 10-14。

##### 示例 10-14\. 重复错误：尝试多次创建资源管理器

```py
$ http post localhost:8000/explorer name="Beau Buffette" country="US"
HTTP/1.1 500 Internal Server Error
content-length: 3127
content-type: text/plain; charset=utf-8
date: Mon, 27 Feb 2023 21:04:09 GMT
server: uvicorn

Traceback (most recent call last):
  File ".../starlette/middleware/errors.py", line 162, in *call*
... (lots of confusing innards here) ...
  File ".../service/explorer.py", line 11, in create
    return data.create(explorer)
           ^^^^^^^
  File ".../data/explorer.py", line 37, in create
    curs.execute(qry, params)
sqlite3.IntegrityError: UNIQUE constraint failed: explorer.name
```

我省略了错误跟踪中的大部分行（并用省略号替换了一些部分），因为它主要包含 FastAPI 和底层 Starlette 进行的内部调用。但是最后一行：Web 层中的一个 SQLite 异常！晕厥沙发在哪里？

紧随其后，另一个恐怖在 示例 10-15 中出现：一个缺少的资源管理器。

##### 示例 10-15\. 获取一个不存在的资源管理器

```py
$ http localhost:8000/explorer/"Beau Buffalo"
HTTP/1.1 500 Internal Server Error
content-length: 3282
content-type: text/plain; charset=utf-8
date: Mon, 27 Feb 2023 21:09:37 GMT
server: uvicorn

Traceback (most recent call last):
  File ".../starlette/middleware/errors.py", line 162, in *call*
... (many lines of ancient cuneiform) ...
  File ".../data/explorer.py", line 11, in row_to_model
    name, country, description = row
    ^^^^^^^
TypeError: cannot unpack non-iterable NoneType object
```

什么是在底层（数据）层捕获这些异常并将详细信息传达给顶层（Web）的好方法？可能的方法包括以下几种：

+   让 SQLite 吐出一个毛球（异常），并在 Web 层处理它。

    +   但是：这混淆了各个层级，这是*不好*的。Web 层不应该知道任何关于具体数据库的信息。

+   让服务和数据层中的每个函数返回`Explorer | None`，而不是返回`Explorer`。然后，`None`表示失败。（您可以通过在*model/explorer.py*中定义`OptExplorer = Explorer | None`来缩短这个过程。）

    +   但是：函数可能因多种原因失败，您可能需要详细信息。这需要大量的代码编辑。

+   为`Missing`和`Duplicate`数据定义异常，包括问题的详细信息。这些异常将通过各个层级传播，无需更改代码，直到 Web 路径函数捕获它们。它们也是应用程序特定的，而不是数据库特定的，保持了各个层级的独立性。

    +   但是：实际上，我喜欢这个，所以它放在 示例 10-16 中。

##### 示例 10-16\. 定义一个新的顶级 errors.py

```py
class Missing(Exception):
    def __init__(self, msg:str):
        self.msg = msg

class Duplicate(Exception):
    def __init__(self, msg:str):
        self.msg = msg
```

每个异常都有一个`msg`字符串属性，可以通知高级别代码发生了什么。

要实现这一点，在 示例 10-17 中，让*data/init.py*导入 SQLite 会为重复操作引发的异常。

##### 示例 10-17\. 在 data/init.py 中添加一个 SQLite 异常导入

```py
from sqlite3 import connect, IntegrityError
```

在 示例 10-18 中导入和捕获此错误。

##### 示例 10-18\. 修改 data/explorer.py 以捕获并引发这些异常

```py
from init import (conn, curs, IntegrityError)
from model.explorer import Explorer
from error import Missing, Duplicate

curs.execute("""create table if not exists explorer(
 name text primary key,
 country text,
 description text)""")

def row_to_model(row: tuple) -> Explorer:
    name, country, description = row
    return Explorer(name=name,
        country=country, description=description)

def model_to_dict(explorer: Explorer) -> dict:
    return explorer.dict()

def get_one(name: str) -> Explorer:
    qry = "select * from explorer where name=:name"
    params = {"name": name}
    curs.execute(qry, params)
    row = curs.fetchone()
    if row:
        return row_to_model(row)
    else:
        raise Missing(msg=f"Explorer {name} not found")

def get_all() -> list[Explorer]:
    qry = "select * from explorer"
    curs.execute(qry)
    return [row_to_model(row) for row in curs.fetchall()]

def create(explorer: Explorer) -> Explorer:
    if not explorer: return None
    qry = """insert into explorer (name, country, description) values
 (:name, :country, :description)"""
    params = model_to_dict(explorer)
    try:
        curs.execute(qry, params)
    except IntegrityError:
        raise Duplicate(msg=
            f"Explorer {explorer.name} already exists")
    return get_one(explorer.name)

def modify(name: str, explorer: Explorer) -> Explorer:
    if not (name and explorer): return None
    qry = """update explorer
 set name=:name,
 country=:country,
 description=:description
 where name=:name_orig"""
    params = model_to_dict(explorer)
    params["name_orig"] = explorer.name
    curs.execute(qry, params)
    if curs.rowcount == 1:
        return get_one(explorer.name)
    else:
        raise Missing(msg=f"Explorer {name} not found")

def delete(name: str):
    if not name: return False
    qry = "delete from explorer where name = :name"
    params = {"name": name}
    curs.execute(qry, params)
    if curs.rowcount != 1:
        raise Missing(msg=f"Explorer {name} not found")
```

这消除了需要声明任何函数返回`Explorer | None`或`Optional[Explorer]`的需要。你只为正常的返回类型指定类型提示，而不是异常。因为异常独立于调用堆栈向上传播，直到有人捕获它们，所以你不必在服务层更改任何内容。但是这是在示例 10-19 中的新的*web/explorer.py*，带有异常处理程序和适当的 HTTP 状态代码返回。

##### 示例 10-19\. 在 web/explorer.py 中处理`Missing`和`Duplicate`异常

```py
from fastapi import APIRouter, HTTPException
from model.explorer import Explorer
from service import explorer as service
from error import Duplicate, Missing

router = APIRouter(prefix = "/explorer")

@router.get("")
@router.get("/")
def get_all() -> list[Explorer]:
    return service.get_all()

@router.get("/{name}")
def get_one(name) -> Explorer:
    try:
        return service.get_one(name)
    except Missing as exc:
        raise HTTPException(status_code=404, detail=exc.msg)

@router.post("", status_code=201)
@router.post("/", status_code=201)
def create(explorer: Explorer) -> Explorer:
    try:
        return service.create(explorer)
    except Duplicate as exc:
        raise HTTPException(status_code=404, detail=exc.msg)

@router.patch("/")
def modify(name: str, explorer: Explorer) -> Explorer:
    try:
        return service.modify(name, explorer)
    except Missing as exc:
        raise HTTPException(status_code=404, detail=exc.msg)

@router.delete("/{name}", status_code=204)
def delete(name: str):
    try:
        return service.delete(name)
    except Missing as exc:
        raise HTTPException(status_code=404, detail=exc.msg)
```

在示例 10-20 中测试这些更改。

##### 示例 10-20\. 再次测试不存在的一次性探索者，使用新的`Missing`异常

```py
$ http localhost:8000/explorer/"Beau Buffalo"
HTTP/1.1 404 Not Found
content-length: 44
content-type: application/json
date: Mon, 27 Feb 2023 21:11:27 GMT
server: uvicorn

{
    "detail": "Explorer Beau Buffalo not found"
}
```

很好。现在，在示例 10-21 中再次尝试邪恶的克隆尝试。

##### 示例 10-21\. 测试重复修复

```py
$ http post localhost:8000/explorer name="Beau Buffette" country="US"
HTTP/1.1 404 Not Found
content-length: 50
content-type: application/json
date: Mon, 27 Feb 2023 21:14:00 GMT
server: uvicorn

{
    "detail": "Explorer Beau Buffette already exists"
}
```

缺失检查也将适用于修改和删除端点。你可以尝试为它们编写类似的测试。

## 单元测试

单元测试仅涉及数据层，检查数据库调用和 SQL 语法。我将此部分放在完整测试之后，因为我希望已经定义、解释和编码了`Missing`和`Duplicate`异常，存储在*data/creature.py*中。示例 10-22 列出了测试脚本*test/unit/data/test_creature.py*。以下是一些需要注意的点：

+   在导入`data`中的`init`或`creature`之前，将环境变量`CRYPTID_SQLITE_DATABASE`设置为`":memory:"`。这个值使 SQLite 完全在内存中运行，不会覆盖任何现有的数据库文件，甚至不会在磁盘上创建文件。在首次导入该模块时，*data/init.py*会检查它。

+   名为`sample`的*fixture*被传递给需要`Creature`对象的函数。

+   测试按顺序运行。在本例中，相同的数据库在整个过程中保持开启状态，而不是在函数之间重置。原因是允许前一个函数的更改保持持久化。使用 pytest，fixture 可能具有以下内容：

    函数范围（默认）

    每个测试函数之前都会调用它。

    会话范围

    它仅在开始时被调用一次。

+   一些测试强制引发`Missing`或`Duplicate`异常，并验证它们是否被捕获。

因此，示例 10-22 中的每个测试都会获得一个全新的、未更改的名为`sample`的`Creature`对象。

##### 示例 10-22\. data/creature.py 的单元测试

```py
import os
import pytest
from model.creature import Creature
from error import Missing, Duplicate

# set this before data imports below for data.init
os.environ["CRYPTID_SQLITE_DB"] = ":memory:"
from data import creature

@pytest.fixture
def sample() -> Creature:
    return Creature(name="yeti", country="CN", area="Himalayas",
        description="Harmless Himalayan",
        aka="Abominable Snowman")

def test_create(sample):
    resp = creature.create(sample)
    assert resp == sample

def test_create_duplicate(sample):
    with pytest.raises(Duplicate):
        _ = creature.create(sample)

def test_get_one(sample):
    resp = creature.get_one(sample.name)
    assert resp == sample

def test_get_one_missing():
    with pytest.raises(Missing):
        _ = creature.get_one("boxturtle")

def test_modify(sample):
    creature.area = "Sesame Street"
    resp = creature.modify(sample.name, sample)
    assert resp == sample

def test_modify_missing():
    thing: Creature = Creature(name="snurfle", country="RU", area="",
        description="some thing", aka="")
    with pytest.raises(Missing):
        _ = creature.modify(thing.name, thing)

def test_delete(sample):
    resp = creature.delete(sample.name)
    assert resp is None

def test_delete_missing(sample):
    with pytest.raises(Missing):
        _ = creature.delete(sample.name)
```

提示：你可以制作自己版本的*test/unit/data/test_explorer.py*。

# 回顾

本章介绍了一个简单的数据处理层，根据需要在层栈中上下移动几次。第十二章包含每个层的单元测试，以及跨层集成和完整的端到端测试。第十四章深入探讨了数据库的更多细节和详细示例。
