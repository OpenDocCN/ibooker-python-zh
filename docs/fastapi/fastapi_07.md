# 第五章：Pydantic、类型提示和模型之旅

> 使用 Python 类型提示进行数据验证和设置管理。
> 
> 快速且可扩展，Pydantic 与您的 linters/IDE/大脑完美配合。使用纯粹、规范的 Python 3.6+ 定义数据，然后用 Pydantic 进行验证。
> 
> Samuel Colvin，Pydantic 的开发者

# 预览

FastAPI 主要依赖于一个名为 Pydantic 的 Python 包。它使用*模型*（Python 对象类）来定义数据结构。在编写更大型应用程序时，这些在 FastAPI 应用中广泛使用，并且是一个真正的优势。

# 类型提示

是时候了解更多关于 Python *类型提示* 的内容了。

第二章 提到，在许多计算机语言中，变量直接指向内存中的值。这要求程序员声明其类型，以确定值的大小和位数。在 Python 中，变量只是与对象相关联的名称，而对象才有类型。

在标准编程中，变量通常与相同的对象相关联。如果我们为该变量关联一个类型提示，我们可以避免一些编程错误。因此，Python 在语言中添加了类型提示，位于标准 typing 模块中。Python 解释器会忽略类型提示语法，并将程序运行为若不存在一样。那么，它的意义何在？

你可能在一行中将变量视为字符串，但后来忘记了，并将其分配给不同类型的对象。虽然其他语言的编译器会抱怨，但 Python 不会。标准 Python 解释器将捕获常规语法错误和运行时异常，但不会检查变量类型的混合。像 mypy 这样的辅助工具会关注类型提示，并警告您任何不匹配的情况。

Python 开发者可以利用提示，编写超越类型错误检查的工具。接下来的部分将描述 Pydantic 包的开发过程，以解决一些不太明显的需求。稍后，您将看到它与 FastAPI 的集成，大大简化了许多 Web 开发问题的处理。

顺便说一句，类型提示是什么样的？变量有一种语法，函数返回值有另一种语法。

变量类型提示可能仅包括类型：

```py
*name*: *type*
```

或者初始化变量并赋予一个值：

```py
*name*: *type* = *value*
```

*类型* 可以是标准的 Python 简单类型，如 `int` 或 `str`，也可以是集合类型，如 `tuple`、`list` 或 `dict`：

```py
thing: str = "yeti"
```

###### 注意

在 Python 3.9 之前，您需要从 typing 模块导入这些标准类型名称的大写版本：

```py
from typing import Str
thing: Str = "yeti"
```

这里有一些初始化的示例：

```py
physics_magic_number: float = 1.0/137.03599913
hp_lovecraft_noun: str = "ichor"
exploding_sheep: tuple = "sis", "boom", bah!"
responses: dict = {"Marco": "Polo", "answer": 42}
```

您还可以包含集合的子类型：

```py
*name*: dict[*keytype*, *valtype*] = {*key1*: *val1*, *key2*: *val2*}
```

typing 模块对子类型有有用的额外功能；最常见的如下：

`Any`

任何类型

`Union`

任何指定的类型，比如 `Union[str, int]`。

###### 注意

在 Python 3.10 及以上版本中，您可以使用 `*type1* | *type2*` 而不是 `Union[*type1*, *type2*]`。

对于 Python `dict` 的 Pydantic 定义示例如下：

```py
from typing import Any
responses: dict[str, Any] = {"Marco": "Polo", "answer": 42}
```

或者，稍微具体一点：

```py
from typing import Union
responses: dict[str, Union[str, int]] = {"Marco": "Polo", "answer": 42}
```

或（Python 3.10 及以上版本）：

```py
responses: dict[str, str | int] = {"Marco": "Polo", "answer": 42}
```

注意，类型提示的变量行是合法的 Python 语法，但裸变量行不是：

```py
$ python
...
>>> thing0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name *thing0* is not defined
>>> thing0: str
```

此外，正常的 Python 解释器无法捕捉到错误的类型使用：

```py
$ python
...
>>> thing1: str = "yeti"
>>> thing1 = 47
```

但它们将被 mypy 捕获。如果您尚未安装它，请运行 `pip install mypy`。将这两行保存到一个名为 *stuff.py* 的文件中，¹，然后尝试这样做：

```py
$ mypy stuff.py
stuff.py:2: error: Incompatible types in assignment
(expression has type "int", variable has type "str")
Found 1 error in 1 file (checked 1 source file)
```

函数返回类型提示使用箭头而不是冒号：

```py
*function*(*args*) -> *type*:
```

下面是一个 Pydantic 函数返回的示例：

```py
def get_thing() -> str:
   return "yeti"
```

您可以使用任何类型，包括您定义的类或它们的组合。您将在几页中看到这一点。

# 数据分组

通常，我们需要将一组相关的变量放在一起，而不是传递许多单独的变量。我们如何将多个变量作为一组集成，并保留类型提示？

让我们抛开前几章中单调的问候例子，从现在开始使用更丰富的数据。就像本书的其余部分一样，我们将使用*神秘动物*（虚构的生物）的例子，以及（同样虚构的）探险家。我们的初始神秘动物定义仅包含以下字符串变量：

`name`

键

`country`

两个字符的 ISO 国家代码（3166-1 alpha 2）或 `*` = all

`area`

可选；美国州或其他国家的分区

`description`

自由格式

`aka`

也称为…​

探险家将拥有以下内容：

`name`

键

`country`

两个字符的 ISO 国家代码

`description`

自由格式

Python 的历史数据分组结构（超出基本的 `int`、`string` 等）在这里列出：

`tuple`

一个不可变的对象序列

`list`

一个可变的对象序列

`set`

可变的不同对象

`dict`

可变的键-值对象对（键必须是不可变类型）

元组（示例 5-1）和列表（示例 5-2）仅允许您通过其偏移访问成员变量，因此您必须记住每个位置的内容。

##### 示例 5-1\. 使用元组

```py
>>> tuple_thing = ("yeti", "CN", "Himalayas",
    "Hirsute Himalayan", "Abominable Snowman")
>>> print("Name is", tuple_thing[0])
Name is yeti
```

##### 示例 5-2\. 使用列表

```py
>>> list_thing = ["yeti", "CN", "Himalayas",
    "Hirsute Himalayan", "Abominable Snowman"]
>>> print("Name is", list_thing[0])
Name is yeti
```

示例 5-3 表明，通过为整数偏移定义名称，您可以得到更详细的解释。

##### 示例 5-3\. 使用元组和命名偏移

```py
>>> NAME = 0
>>> COUNTRY = 1
>>> AREA = 2
>>> DESCRIPTION = 3
>>> AKA = 4
>>> tuple_thing = ("yeti", "CN", "Himalayas",
    "Hirsute Himalayan", "Abominable Snowman")
>>> print("Name is", tuple_thing[NAME])
Name is yeti
```

在 示例 5-4 中，字典更好一些，可以通过描述性键访问。

##### 示例 5-4\. 使用字典

```py
>>> dict_thing = {"name": "yeti",
...     "country": "CN",
...     "area": "Himalayas",
...     "description": "Hirsute Himalayan",
...     "aka": "Abominable Snowman"}
>>> print("Name is", dict_thing["name"])
Name is yeti
```

集合仅包含唯一值，因此对于聚类不是非常有用。

在 示例 5-5 中，*命名元组* 是一个可以通过整数偏移或名称访问的元组。

##### 示例 5-5\. 使用命名元组

```py
>>> from collections import namedtuple
>>> CreatureNamedTuple = namedtuple("CreatureNamedTuple",
...     "name, country, area, description, aka")
>>> namedtuple_thing = CreatureNamedTuple("yeti",
...     "CN",
...     "Himalaya",
...     "Hirsute HImalayan",
...     "Abominable Snowman")
>>> print("Name is", namedtuple_thing[0])
Name is yeti
>>> print("Name is", namedtuple_thing.name)
Name is yeti
```

###### 注意

您不能说 `namedtuple_thing["name"]`。它是一个 `tuple`，而不是一个 `dict`，所以索引需要是一个整数。

示例 5-6 定义了一个新的 Python `class`，并添加了所有属性与 `self`。但您需要大量键入才能定义它们。

##### 示例 5-6\. 使用标准类

```py
>>> class CreatureClass():
...     def __init__(self,
...       name: str,
...       country: str,
...       area: str,
...       description: str,
...       aka: str):
...         self.name = name
...         self.country = country
...         self.area = area
...         self.description = description
...         self.aka = aka
...
>>> class_thing = CreatureClass(
...     "yeti",
...     "CN",
...     "Himalayas"
...     "Hirsute Himalayan",
...     "Abominable Snowman")
>>> print("Name is", class_thing.name)
Name is yeti
```

###### 注意

你可能会想，没什么大不了的？使用常规类，你可以添加更多数据（属性），但特别是行为（方法）。你可能会在一个疯狂的日子里决定添加一个查找探险者最爱歌曲的方法。（这不适用于某些生物。²)但这里的用例只是为了在各层之间无干扰地移动一堆数据，并在进出时进行验证。同时，方法是方形的钉子，会在数据库的圆孔中挣扎着不合适。

Python 有没有类似于其他计算机语言所说的*记录*或*结构体*（一组名称和值）？Python 的一个新特性是*数据类*。示例 5-7 展示了在数据类中，所有的`self`内容如何消失。

##### 示例 5-7\. 使用数据类

```py
>>> from dataclasses import dataclass
>>>
>>> @dataclass
... class CreatureDataClass():
...     name: str
...     country: str
...     area: str
...     description: str
...     aka: str
...
>>> dataclass_thing = CreatureDataClass(
...     "yeti",
...     "CN",
...     "Himalayas"
...     "Hirsute Himalayan",
...     "Abominable Snowman")
>>> print("Name is", dataclass_thing.name)
Name is yeti
```

这对于保持变量在一起的部分相当不错。但我们想要更多，所以让我们向圣诞老人要这些：

+   一个可能的替代类型的*联合*

+   缺失/可选值

+   默认值

+   数据验证

+   序列化为和从 JSON 等格式

# 替代方案

使用 Python 的内置数据结构，特别是字典，是很有吸引力的。但你最终会发现字典有点太“松散”。自由是有代价的。你需要检查*所有的*：

+   键是可选的吗？

+   如果键缺失，是否有默认值？

+   键是否存在？

+   如果是，键的值是正确的类型吗？

+   如果是，值是否在正确的范围内或匹配某个模式？

至少有三个解决方案至少解决了一些这些要求：

[数据类](https://oreil.ly/mxANA)

Python 的标准部分。

[attrs](https://www.attrs.org)

第   第三方，但比数据类更加完善。

[Pydantic](https://docs.pydantic.dev)

也是第三方，但已集成到 FastAPI 中，如果你已经在使用 FastAPI，这是一个容易的选择。如果你正在阅读这本书，那很可能是这样。

三者的一个方便比较在[YouTube](https://oreil.ly/pkQD3)上。一个结论是 Pydantic 在验证方面脱颖而出，并且与 FastAPI 的集成捕捉了许多潜在的数据错误。另一个是 Pydantic 依赖于继承（从`BaseModel`类），而其他两个使用 Python 装饰器来定义它们的对象。这更像是风格问题。

在[另一项比较](https://oreil.ly/gU28a)中，Pydantic 在[marshmallow](https://marshmallow.readthedocs.io)和令人着迷的[Voluptuous](https://github.com/alecthomas/voluptuous)这样的旧验证包上表现更优。Pydantic 的另一个大优点是它使用标准的 Python 类型提示语法；旧的库在类型提示出现之前就已经存在，并自己开发了类型提示。

所以我在这本书中选择使用 Pydantic，但如果你没有使用 FastAPI，你可能会找到这两个替代方案的用法。

Pydantic 提供了指定任何组合这些检查的方法：

+   必需与可选

+   如果未指定但必需的默认值

+   预期的数据类型或类型

+   值范围限制

+   其他基于函数的检查（如有需要）。

+   序列化和反序列化

# 一个简单的示例

你已经看到如何通过 URL、查询参数或 HTTP 主体向 Web 端点提供简单的字符串。问题在于，通常请求和接收多种类型的数据组。这就是 Pydantic 模型首次出现在 FastAPI 中的地方。

此初始示例将使用三个文件：

+   *model.py* 定义了一个 Pydantic 模型。

+   *data.py* 是一个虚假数据源，定义了一个模型实例。

+   *web.py* 定义了一个 FastAPI Web 端点，返回虚假数据。

为了简单起见，在本章中，让我们将所有文件放在同一个目录中。在讨论更大网站的后续章节中，我们将把它们分开放置到各自的层中。首先，在 Example 5-8 中定义一个 *model* 用于一个生物。

##### Example 5-8\. 定义一个生物模型：model.py

```py
from pydantic import BaseModel

class Creature(BaseModel):
    name: str
    country: str
    area: str
    description: str
    aka: str

thing = Creature(
    name="yeti",
    country="CN",
    area="Himalayas",
    description="Hirsute Himalayan",
    aka="Abominable Snowman")
)
print("Name is", thing.name)
```

`Creature` 类继承自 Pydantic 的 `BaseModel`。`: str` 在 `name`、`country`、`area`、`description` 和 `aka` 后面是类型提示，表示每个都是 Python 字符串。

###### 注意

在本示例中，所有字段都是必需的。在 Pydantic 中，如果类型描述中没有 `Optional`，则字段必须具有值。

在 Example 5-9 中，如果包括它们的名称，则参数可以按任意顺序传递。

##### Example 5-9\. 创建一个生物

```py
>>> thing = Creature(
...     name="yeti",
...     country="CN",
...     area="Himalayas"
...     description="Hirsute Himalayan",
...     aka="Abominable Snowman")
>>> print("Name is", thing.name)
Name is yeti
```

现在，Example 5-10 定义了一个数据的小来源；在后续章节中，数据库将执行此操作。类型提示 `list[Creature]` 告诉 Python 这是一个 `Creature` 对象列表。

##### Example 5-10\. 在 data.py 中定义虚假数据

```py
from model import Creature

_creatures: list[Creature] = [
    Creature(name="yeti",
             country="CN",
             area="Himalayas",
             description="Hirsute Himalayan",
             aka="Abominable Snowman"
             ),
    Creature(name="sasquatch",
             country="US",
             area="*",
             description="Yeti's Cousin Eddie",
             aka="Bigfoot")
]

def get_creatures() -> list[Creature]:
    return _creatures
```

（因为大脚几乎无处不在，我们在 `"*"` 处使用 Bigfoot 的 `area`。）

此代码导入了我们刚刚编写的 *model.py*。它通过调用 `_creatures` 的 `Creature` 对象列表来进行一些数据隐藏，并提供 `get_creatures()` 函数来返回它们。

Example 5-11 列出了 *web.py*，一个定义 FastAPI Web 端点的文件。

##### Example 5-11\. 定义一个 FastAPI Web 端点：web.py

```py
from model import Creature
from fastapi import FastAPI

app = FastAPI()

@app.get("/creature")
def get_all() -> list[Creature]:
    from data import get_creatures
    return get_creatures()
```

现在，在 Example 5-12 中启动此单端点服务器。

##### Example 5-12\. 启动 Uvicorn

```py
$ uvicorn creature:app
INFO:     Started server process [24782]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

在另一个窗口中，Example 5-13 使用 HTTPie Web 客户端访问 Web 应用程序（如果喜欢也可以尝试浏览器或 Requests 模块）。

##### Example 5-13\. 使用 HTTPie 进行测试

```py
$ http http://localhost:8000/creature
HTTP/1.1 200 OK
content-length: 183
content-type: application/json
date: Mon, 12 Sep 2022 02:21:15 GMT
server: uvicorn


    {
        "aka": "Abominable Snowman",
        "area": "Himalayas",
        "country": "CN",
        "name": "yeti",
        "description": "Hirsute Himalayan"
    },
    {
        "aka": "Bigfoot",
        "country": "US",
        "area": "*",
        "name": "sasquatch",
        "description": "Yeti's Cousin Eddie"
    }
```

FastAPI 和 Starlette 自动将原始 `Creature` 模型对象列表转换为 JSON 字符串。这是 FastAPI 中的默认输出格式，因此我们无需指定它。

另外，你最初启动 Uvicorn Web 服务器的窗口应该打印了一行日志：

```py
INFO:     127.0.0.1:52375 - "GET /creature HTTP/1.1" 200 OK
```

# 验证类型

前一节展示了如何执行以下操作：

+   将类型提示应用于变量和函数

+   定义和使用 Pydantic 模型

+   从数据源返回模型列表

+   将模型列表返回给 Web 客户端，自动将模型列表转换为 JSON

现在，让我们真正开始验证数据工作。

尝试将错误类型的值分配给一个或多个 `Creature` 字段。让我们使用一个独立的测试来进行测试（Pydantic 不适用于任何 Web 代码；它是一个数据处理工具）。

[Example 5-14 列出了 *test1.py*。

##### 示例 5-14\. 测试 Creature 模型

```py
from model import Creature

dragon = Creature(
    name="dragon",
    description=["incorrect", "string", "list"],
    country="*" ,
    area="*",
    aka="firedrake")
```

现在在 示例 5-15 中尝试测试。

##### 示例 5-15\. 运行测试

```py
$ python test1.py
Traceback (most recent call last):
  File ".../test1.py", line 3, in <module>
    dragon = Creature(
  File "pydantic/main.py", line 342, in
    pydantic.main.BaseModel.*init*
    pydantic.error_wrappers.ValidationError:
    1 validation error for Creature description
  str type expected (type=type_error.str)
```

发现我们已经将字符串列表分配给 `description` 字段，但它希望是一个普通的字符串。

# 验证值

即使值的类型与其在 `Creature` 类中的规格相匹配，可能仍需要通过更多检查。一些限制可以放置在值本身上：

+   整数（`conint`）或浮点数：

    `gt`

    大于

    `lt`

    小于

    `ge`

    大于或等于

    `le`

    小于或等于

    `multiple_of`

    值的整数倍

+   字符串（`constr`）：

    `min_length`

    最小字符（非字节）长度

    `max_length`

    最大字符长度

    `to_upper`

    转换为大写

    `to_lower`

    转换为小写

    `regex`

    匹配 Python 正则表达式

+   元组、列表或集合：

    `min_items`

    最小元素数量

    `max_items`

    最大元素数量

这些在模型的类型部分指定。

示例 5-16 确保 `name` 字段始终至少为两个字符长。否则，`""`（空字符串）是有效的字符串。

##### 示例 5-16\. 查看验证失败

```py
>>> from pydantic import BaseModel, constr
>>>
>>> class Creature(BaseModel):
...     name: constr(min_length=2)
...     country: str
...     area: str
...     description: str
...     aka: str
...
>>> bad_creature = Creature(name="!",
...     description="it's a raccoon",
...     area="your attic")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "pydantic/main.py", line 342,
  in pydantic.main.BaseModel.__init__
pydantic.error_wrappers.ValidationError:
1 validation error for Creature name
  ensure this value has at least 2 characters
  (type=value_error.any_str.min_length; limit_value=2)
```

`constr` 意味着*受限字符串*。 示例 5-17 使用另一种方式，即 Pydantic 的 `Field` 规范。

##### 示例 5-17\. 另一个验证失败，使用 `Field`

```py
>>> from pydantic import BaseModel, Field
>>>
>>> class Creature(BaseModel):
...     name: str = Field(..., min_length=2)
...     country: str
...     area: str
...     description: str
...     aka: str
...
>>> bad_creature = Creature(name="!",
...     area="your attic",
...     description="it's a raccoon")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "pydantic/main.py", line 342,
  in pydantic.main.BaseModel.__init__
pydantic.error_wrappers.ValidationError:
1 validation error for Creature name
  ensure this value has at least 2 characters
  (type=value_error.any_str.min_length; limit_value=2)
```

`...` 参数传递给 `Field()` 意味着需要一个值，并且没有默认值。

这是 Pydantic 的简要介绍。主要的收获是它允许您自动验证数据。当从 Web 或数据层获取数据时，您将看到这是多么有用。

# 回顾

在您的 Web 应用程序中传递的最佳数据定义方式是模型。Pydantic 利用 Python 的*类型提示*来定义在应用程序中传递的数据模型。接下来是：定义*依赖项*以将特定细节与通用代码分离。

¹ 我有任何可察觉的想象力吗？嗯……没有。

² 除了那些尖叫的雪人小团体（一个乐队的好名字）。
