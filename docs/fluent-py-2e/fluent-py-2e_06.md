# 第五章：数据类构建器

> 数据类就像孩子一样。它们作为一个起点是可以的，但要作为一个成熟的对象参与，它们需要承担一些责任。
> 
> 马丁·福勒和肯特·贝克¹

Python 提供了几种构建简单类的方法，这些类只是一组字段，几乎没有额外功能。这种模式被称为“数据类”，而`dataclasses`是支持这种模式的包之一。本章涵盖了三种不同的类构建器，您可以将它们用做编写数据类的快捷方式：

`collections.namedtuple`

最简单的方法——自 Python 2.6 起可用。

`typing.NamedTuple`

一种需要在字段上添加类型提示的替代方法——自 Python 3.5 起，3.6 中添加了`class`语法。

`@dataclasses.dataclass`

一个类装饰器，允许比以前的替代方案更多的定制化，增加了许多选项和潜在的复杂性——自 Python 3.7 起。

在讨论完这些类构建器之后，我们将讨论为什么*数据类*也是一个代码异味的名称：一种可能是糟糕面向对象设计的症状的编码模式。

###### 注意

`typing.TypedDict`可能看起来像另一个数据类构建器。它使用类似的语法，并在 Python 3.9 的[`typing`模块文档](https://fpy.li/5-1)中的`typing.NamedTuple`之后描述。

但是，`TypedDict`不会构建您可以实例化的具体类。它只是一种语法，用于为将用作记录的映射值接受的函数参数和变量编写类型提示，其中键作为字段名。我们将在第十五章的`TypedDict`中看到它们。

# 本章的新内容

本章是*流畅的 Python*第二版中的新内容。第一版的第二章中出现了“经典命名元组”一节，但本章的其余部分是全新的。

我们从三个类构建器的高级概述开始。

# 数据类构建器概述

考虑一个简单的类来表示地理坐标对，如示例 5-1 所示。

##### 示例 5-1。*class/coordinates.py*

```py
class Coordinate:

    def __init__(self, lat, lon):
        self.lat = lat
        self.lon = lon
```

那个`Coordinate`类完成了保存纬度和经度属性的工作。编写`__init__`样板变得非常乏味，特别是如果你的类有超过几个属性：每个属性都被提及三次！而且那个样板并没有为我们购买我们期望从 Python 对象中获得的基本功能：

```py
>>> from coordinates import Coordinate
>>> moscow = Coordinate(55.76, 37.62)
>>> moscow
<coordinates.Coordinate object at 0x107142f10> # ①
>>> location = Coordinate(55.76, 37.62)
>>> location == moscow  # ②
False >>> (location.lat, location.lon) == (moscow.lat, moscow.lon)  # ③
True
```

①

从`object`继承的`__repr__`并不是很有用。

②

无意义的`==`；从`object`继承的`__eq__`方法比较对象 ID。

③

比较两个坐标需要显式比较每个属性。

本章涵盖的数据类构建器会自动提供必要的`__init__`、`__repr__`和`__eq__`方法，以及其他有用的功能。

###### 注意

这里讨论的类构建器都不依赖继承来完成工作。`collections.namedtuple`和`typing.NamedTuple`都构建了`tuple`子类的类。`@dataclass`是一个类装饰器，不会以任何方式影响类层次结构。它们每个都使用不同的元编程技术将方法和数据属性注入到正在构建的类中。

这里是一个使用`namedtuple`构建的`Coordinate`类——一个工厂函数，根据您指定的名称和字段构建`tuple`的子类：

```py
>>> from collections import namedtuple
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> issubclass(Coordinate, tuple)
True >>> moscow = Coordinate(55.756, 37.617)
>>> moscow
Coordinate(lat=55.756, lon=37.617) # ①
>>> moscow == Coordinate(lat=55.756, lon=37.617)  # ②
True
```

①

有用的`__repr__`。

②

有意义的`__eq__`。

较新的`typing.NamedTuple`提供了相同的功能，为每个字段添加了类型注释：

```py
>>> import typing
>>> Coordinate = typing.NamedTuple('Coordinate',
...     [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True
>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```

###### 提示

一个带有字段作为关键字参数构造的类型命名元组也可以这样创建：

```py
Coordinate = typing.NamedTuple('Coordinate', lat=float, lon=float)

```

这更易读，也让您提供字段和类型的映射作为 `**fields_and_types`。

自 Python 3.6 起，`typing.NamedTuple` 也可以在 `class` 语句中使用，类型注解的写法如 [PEP 526—变量注解的语法](https://fpy.li/pep526) 中描述的那样。这样更易读，也方便重写方法或添加新方法。示例 5-2 是相同的 `Coordinate` 类，具有一对 `float` 属性和一个自定义的 `__str__` 方法，以显示格式为 55.8°N, 37.6°E 的坐标。

##### 示例 5-2\. *typing_namedtuple/coordinates.py*

```py
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

###### 警告

尽管 `NamedTuple` 在 `class` 语句中出现为超类，但实际上并非如此。`typing.NamedTuple` 使用元类的高级功能² 来自定义用户类的创建。看看这个：

```py
>>> issubclass(Coordinate, typing.NamedTuple)
False
>>> issubclass(Coordinate, tuple)
True
```

在 `typing.NamedTuple` 生成的 `__init__` 方法中，字段按照在 `class` 语句中出现的顺序作为参数出现。

像 `typing.NamedTuple` 一样，`dataclass` 装饰器支持 [PEP 526](https://fpy.li/pep526) 语法来声明实例属性。装饰器读取变量注解并自动生成类的方法。为了对比，可以查看使用 `dataclass` 装饰器编写的等效 `Coordinate` 类，如 示例 5-3 中所示。

##### 示例 5-3\. *dataclass/coordinates.py*

```py
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

请注意，示例 5-2 和 示例 5-3 中的类主体是相同的——区别在于 `class` 语句本身。`@dataclass` 装饰器不依赖于继承或元类，因此不应干扰您对这些机制的使用。³ 示例 5-3 中的 `Coordinate` 类是 `object` 的子类。

## 主要特点

不同的数据类构建器有很多共同点，如 表 5-1 所总结的。

表 5-1\. 三种数据类构建器之间的选定特点比较；`x` 代表该类型数据类的一个实例

|  | namedtuple | NamedTuple | dataclass |
| --- | --- | --- | --- |
| 可变实例 | 否 | 否 | 是 |
| class 语句语法 | 否 | 是 | 是 |
| 构造字典 | x._asdict() | x._asdict() | dataclasses.asdict(x) |
| 获取字段名 | x._fields | x._fields | [f.name for f in dataclasses.fields(x)] |
| 获取默认值 | x._field_defaults | x._field_defaults | [f.default for f in dataclasses.fields(x)] |
| 获取字段类型 | 不适用 | x.__annotations__ | x.__annotations__ |
| 使用更改创建新实例 | x._replace(…) | x._replace(…) | dataclasses.replace(x, …) |
| 运行时新类 | namedtuple(…) | NamedTuple(…) | dataclasses.make_dataclass(…) |

###### 警告

`typing.NamedTuple` 和 `@dataclass` 构建的类具有一个 `__annotations__` 属性，其中包含字段的类型提示。然而，不建议直接从 `__annotations__` 中读取。相反，获取该信息的推荐最佳实践是调用 [`inspect.get_annotations(MyClass)`](https://fpy.li/5-2)（Python 3.10 中添加）或 [`typing.​get_​type_​hints(MyClass)`](https://fpy.li/5-3)（Python 3.5 到 3.9）。这是因为这些函数提供额外的服务，如解析类型提示中的前向引用。我们将在本书的后面更详细地讨论这个问题，在 “运行时注解问题” 中。

现在让我们讨论这些主要特点。

### 可变实例

这些类构建器之间的一个关键区别是，`collections.namedtuple` 和 `typing.NamedTuple` 构建 `tuple` 的子类，因此实例是不可变的。默认情况下，`@dataclass` 生成可变类。但是，装饰器接受一个关键字参数 `frozen`—如 示例 5-3 中所示。当 `frozen=True` 时，如果尝试在初始化实例后为字段分配值，类将引发异常。

### 类语句语法

只有`typing.NamedTuple`和`dataclass`支持常规的`class`语句语法，这样可以更容易地向正在创建的类添加方法和文档字符串。

### 构造字典

这两种命名元组变体都提供了一个实例方法（`._asdict`），用于从数据类实例中的字段构造一个`dict`对象。`dataclasses`模块提供了一个执行此操作的函数：`dataclasses.asdict`。

### 获取字段名称和默认值

所有三种类构建器都允许您获取字段名称和可能为其配置的默认值。在命名元组类中，这些元数据位于`._fields`和`._fields_defaults`类属性中。您可以使用`dataclasses`模块中的`fields`函数从装饰的`dataclass`类中获取相同的元数据。它返回一个具有多个属性的`Field`对象的元组，包括`name`和`default`。

### 获取字段类型

使用`typing.NamedTuple`和`@dataclass`帮助定义的类具有字段名称到类型的映射`__annotations__`类属性。如前所述，使用`typing.get_type_hints`函数而不是直接读取`__annotations__`。

### 具有更改的新实例

给定一个命名元组实例`x`，调用`x._replace(**kwargs)`将返回一个根据给定关键字参数替换了一些属性值的新实例。`dataclasses.replace(x, **kwargs)`模块级函数对于`dataclass`装饰的类的实例也是如此。

### 运行时新类

尽管`class`语句语法更易读，但它是硬编码的。一个框架可能需要在运行时动态构建数据类。为此，您可以使用`collections.namedtuple`的默认函数调用语法，该语法同样受到`typing.NamedTuple`的支持。`dataclasses`模块提供了一个`make_dataclass`函数来实现相同的目的。

在对数据类构建器的主要特性进行概述之后，让我们依次专注于每个特性，从最简单的开始。

# 经典的命名元组

`collections.namedtuple`函数是一个工厂，构建了增强了字段名称、类名和信息性`__repr__`的`tuple`子类。使用`namedtuple`构建的类可以在需要元组的任何地方使用，并且实际上，Python 标准库的许多函数现在用于返回元组的地方现在返回命名元组以方便使用，而不会对用户的代码产生任何影响。

###### 提示

由`namedtuple`构建的类的每个实例占用的内存量与元组相同，因为字段名称存储在类中。

示例 5-4 展示了我们如何定义一个命名元组来保存有关城市信息的示例。

##### 示例 5-4\. 定义和使用命名元组类型

```py
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates')  # ①
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))  # ②
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722, 139.691667)) >>> tokyo.population  # ③
36.933 >>> tokyo.coordinates
(35.689722, 139.691667) >>> tokyo[1]
'JP'
```

①

创建命名元组需要两个参数：一个类名和一个字段名称列表，可以作为字符串的可迭代对象或作为单个以空格分隔的字符串给出。

②

字段值必须作为单独的位置参数传递给构造函数（相反，`tuple`构造函数接受一个单一的可迭代对象）。

③

你可以通过名称或位置访问这些字段。

作为`tuple`子类，`City`继承了一些有用的方法，比如`__eq__`和用于比较运算符的特殊方法，包括`__lt__`，它允许对`City`实例的列表进行排序。

除了从元组继承的属性和方法外，命名元组还提供了一些额外的属性和方法。示例 5-5 展示了最有用的：`_fields`类属性，类方法`_make(iterable)`和`_asdict()`实例方法。

##### 示例 5-5\. 命名元组属性和方法（继续自上一个示例）

```py
>>> City._fields  # ①
('name', 'country', 'population', 'location') >>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, Coordinate(28.613889, 77.208889))
>>> delhi = City._make(delhi_data)  # ②
>>> delhi._asdict()  # ③
{'name': 'Delhi NCR', 'country': 'IN', 'population': 21.935, 'location': Coordinate(lat=28.613889, lon=77.208889)} >>> import json
>>> json.dumps(delhi._asdict())  # ④
'{"name": "Delhi NCR", "country": "IN", "population": 21.935, "location": [28.613889, 77.208889]}'
```

①

`._fields` 是一个包含类的字段名称的元组。

②

`._make()` 从可迭代对象构建 `City`；`City(*delhi_data)` 将执行相同操作。

③

`._asdict()` 返回从命名元组实例构建的 `dict`。

④

`._asdict()` 对于将数据序列化为 JSON 格式非常有用，例如。

###### 警告

直到 Python 3.7，`_asdict` 方法返回一个 `OrderedDict`。自 Python 3.8 起，它返回一个简单的 `dict`——现在我们可以依赖键插入顺序了。如果你一定需要一个 `OrderedDict`，[`_asdict` 文档](https://fpy.li/5-4)建议从结果构建一个：`OrderedDict(x._asdict())`。

自 Python 3.7 起，`namedtuple` 接受 `defaults` 关键字参数，为类的 N 个最右字段的每个字段提供一个默认值的可迭代对象。示例 5-6 展示了如何为 `reference` 字段定义一个带有默认值的 `Coordinate` 命名元组。

##### 示例 5-6。命名元组属性和方法，继续自示例 5-5

```py
>>> Coordinate = namedtuple('Coordinate', 'lat lon reference', defaults=['WGS84'])
>>> Coordinate(0, 0)
Coordinate(lat=0, lon=0, reference='WGS84')
>>> Coordinate._field_defaults
{'reference': 'WGS84'}
```

在“类语句语法”中，我提到使用 `typing.NamedTuple` 和 `@dataclass` 支持的类语法更容易编写方法。你也可以向 `namedtuple` 添加方法，但这是一种 hack。如果你对 hack 不感兴趣，可以跳过下面的框。

现在让我们看看 `typing.NamedTuple` 的变化。

# 带类型的命名元组

`Coordinate` 类与示例 5-6 中的默认字段可以使用 `typing.NamedTuple` 编写，如示例 5-8 所示。

##### 示例 5-8。*typing_namedtuple/coordinates2.py*

```py
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float                # ①
    lon: float
    reference: str = 'WGS84'  # ②
```

①

每个实例字段都必须带有类型注释。

②

`reference` 实例字段带有类型和默认值的注释。

由 `typing.NamedTuple` 构建的类除了那些 `collections.namedtuple` 生成的方法和从 `tuple` 继承的方法外，没有任何其他方法。唯一的区别是存在 `__annotations__` 类属性——Python 在运行时完全忽略它。

鉴于 `typing.NamedTuple` 的主要特点是类型注释，我们将在继续探索数据类构建器之前简要介绍它们。

# 类型提示 101

类型提示，又称类型注释，是声明函数参数、返回值、变量和属性预期类型的方式。

你需要了解的第一件事是，类型提示完全不受 Python 字节码编译器和解释器的强制执行。

###### 注意

这是对类型提示的非常简要介绍，足以理解 `typing.NamedTuple` 和 `@dataclass` 声明中使用的注释的语法和含义。我们将在第八章中介绍函数签名的类型提示，以及在第十五章中介绍更高级的注释。在这里，我们将主要看到使用简单内置类型的提示，比如 `str`、`int` 和 `float`，这些类型可能是用于注释数据类字段的最常见类型。

## 无运行时效果

将 Python 类型提示视为“可以由 IDE 和类型检查器验证的文档”。

这是因为类型提示对 Python 程序的运行时行为没有影响。查看示例 5-9。

##### 示例 5-9。Python 不会在运行时强制执行类型提示

```py
>>> import typing
>>> class Coordinate(typing.NamedTuple):
...     lat: float
...     lon: float
...
>>> trash = Coordinate('Ni!', None)
>>> print(trash)
Coordinate(lat='Ni!', lon=None)    # ①
```

①

我告诉过你：运行时不进行类型检查！

如果你在 Python 模块中键入示例 5-9 的代码，它将运行并显示一个无意义的 `Coordinate`，没有错误或警告：

```py
$ python3 nocheck_demo.py
Coordinate(lat='Ni!', lon=None)
```

类型提示主要用于支持第三方类型检查器，如[Mypy](https://fpy.li/mypy)或[PyCharm IDE](https://fpy.li/5-5)内置的类型检查器。这些是静态分析工具：它们检查 Python 源代码“静止”，而不是运行代码。

要看到类型提示的效果，你必须在你的代码上运行其中一个工具—比如一个检查器。例如，这是 Mypy 对前面示例的看法：

```py
$ mypy nocheck_demo.py
nocheck_demo.py:8: error: Argument 1 to "Coordinate" has
incompatible type "str"; expected "float"
nocheck_demo.py:8: error: Argument 2 to "Coordinate" has
incompatible type "None"; expected "float"
```

正如你所看到的，鉴于`Coordinate`的定义，Mypy 知道创建实例的两个参数必须是`float`类型，但对`trash`的赋值使用了`str`和`None`。⁵

现在让我们谈谈类型提示的语法和含义。

## 变量注释语法

`typing.NamedTuple`和`@dataclass`都使用在[PEP 526](https://fpy.li/pep526)中定义的变量注释语法。这是在`class`语句中定义属性的上下文中对该语法的快速介绍。

变量注释的基本语法是：

```py
var_name: some_type
```

[PEP 484 中的“可接受的类型提示”部分](https://fpy.li/5-6)解释了什么是可接受的类型，但在定义数据类的上下文中，这些类型更有可能有用：

+   一个具体的类，例如，`str`或`FrenchDeck`

+   一个参数化的集合类型，如`list[int]`，`tuple[str, float]`，等等。

+   `typing.Optional`，例如，`Optional[str]`—声明一个可以是`str`或`None`的字段

你也可以用一个值初始化变量。在`typing.NamedTuple`或`@dataclass`声明中，如果在构造函数调用中省略了相应的参数，那个值将成为该属性的默认值：

```py
var_name: some_type = a_value
```

## 变量注释的含义

我们在“无运行时效果”中看到类型提示在运行时没有效果。但在导入时—模块加载时—Python 会读取它们以构建`__annotations__`字典，然后`typing.NamedTuple`和`@dataclass`会使用它们来增强类。

我们将从示例 5-10 中的一个简单类开始这个探索，这样我们以后可以看到`typing.NamedTuple`和`@dataclass`添加的额外功能。

##### 示例 5-10\. meaning/demo_plain.py：带有类型提示的普通类

```py
class DemoPlainClass:
    a: int           # ①
    b: float = 1.1   # ②
    c = 'spam'       # ③
```

①

`a`成为`__annotations__`中的一个条目，但在类中不会创建名为`a`的属性。

②

`b`被保存为注释，并且也成为一个具有值`1.1`的类属性。

③

`c`只是一个普通的类属性，不是一个注释。

我们可以在控制台中验证，首先读取`DemoPlainClass`的`__annotations__`，然后尝试获取其名为`a`、`b`和`c`的属性：

```py
>>> from demo_plain import DemoPlainClass
>>> DemoPlainClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}
>>> DemoPlainClass.a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'DemoPlainClass' has no attribute 'a'
>>> DemoPlainClass.b
1.1
>>> DemoPlainClass.c
'spam'
```

注意，`__annotations__`特殊属性是由解释器创建的，用于记录源代码中出现的类型提示—即使在一个普通类中也是如此。

`a`只作为一个注释存在。它不会成为一个类属性，因为没有值与它绑定。⁶ `b`和`c`作为类属性存储，因为它们绑定了值。

这三个属性都不会出现在`DemoPlainClass`的新实例中。如果你创建一个对象`o = DemoPlainClass()`，`o.a`会引发`AttributeError`，而`o.b`和`o.c`将检索具有值`1.1`和`'spam'`的类属性——这只是正常的 Python 对象行为。

### 检查一个`typing.NamedTuple`

现在让我们检查一个使用与示例 5-10 中`DemoPlainClass`相同属性和注释构建的类，该类使用`typing.NamedTuple`（示例 5-11）。

##### 示例 5-11\. meaning/demo_nt.py：使用`typing.NamedTuple`构建的类

```py
import typing

class DemoNTClass(typing.NamedTuple):
    a: int           # ①
    b: float = 1.1   # ②
    c = 'spam'       # ③
```

①

`a`成为一个注释，也成为一个实例属性。

②

`b`是另一个注释，也成为一个具有默认值`1.1`的实例属性。

③

`c`只是一个普通的类属性；没有注释会引用它。

检查`DemoNTClass`，我们得到：

```py
>>> from demo_nt import DemoNTClass
>>> DemoNTClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}
>>> DemoNTClass.a
<_collections._tuplegetter object at 0x101f0f940>
>>> DemoNTClass.b
<_collections._tuplegetter object at 0x101f0f8b0>
>>> DemoNTClass.c
'spam'
```

这里我们对`a`和`b`的注释与我们在示例 5-10 中看到的相同。但是`typing.NamedTuple`创建了`a`和`b`类属性。`c`属性只是一个具有值`'spam'`的普通类属性。

`a`和`b`类属性是*描述符*，这是第二十三章中介绍的一个高级特性。现在，将它们视为类似于属性获取器的属性：这些方法不需要显式调用运算符`()`来检索实例属性。实际上，这意味着`a`和`b`将作为只读实例属性工作——当我们回想起`DemoNTClass`实例只是一种花哨的元组，而元组是不可变的时，这是有道理的。

`DemoNTClass`也有一个自定义的文档字符串：

```py
>>> DemoNTClass.__doc__
'DemoNTClass(a, b)'
```

让我们检查`DemoNTClass`的一个实例：

```py
>>> nt = DemoNTClass(8)
>>> nt.a
8
>>> nt.b
1.1
>>> nt.c
'spam'
```

要构造`nt`，我们至少需要将`a`参数传递给`DemoNTClass`。构造函数还接受一个`b`参数，但它有一个默认值`1.1`，所以是可选的。`nt`对象具有预期的`a`和`b`属性；它没有`c`属性，但 Python 会像往常一样从类中检索它。

如果尝试为`nt.a`、`nt.b`、`nt.c`甚至`nt.z`分配值，您将收到略有不同的错误消息的`AttributeError`异常。尝试一下并思考这些消息。

### 检查使用 dataclass 装饰的类

现在，我们将检查示例 5-12。

##### 示例 5-12\. meaning/demo_dc.py：使用`@dataclass`装饰的类

```py
from dataclasses import dataclass

@dataclass
class DemoDataClass:
    a: int           # ①
    b: float = 1.1   # ②
    c = 'spam'       # ③
```

①

`a`变成了一个注释，也是由描述符控制的实例属性。

②

`b`是另一个注释，也成为一个具有描述符和默认值`1.1`的实例属性。

③

`c`只是一个普通的类属性；没有注释会引用它。

现在让我们检查`DemoDataClass`上的`__annotations__`、`__doc__`和`a`、`b`、`c`属性：

```py
>>> from demo_dc import DemoDataClass
>>> DemoDataClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}
>>> DemoDataClass.__doc__
'DemoDataClass(a: int, b: float = 1.1)'
>>> DemoDataClass.a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'DemoDataClass' has no attribute 'a'
>>> DemoDataClass.b
1.1
>>> DemoDataClass.c
'spam'
```

`__annotations__`和`__doc__`并不奇怪。然而，在`DemoDataClass`中没有名为`a`的属性——与示例 5-11 中的`DemoNTClass`相反，后者具有一个描述符来从实例中获取`a`作为只读属性（那个神秘的`<_collections._tuplegetter>`）。这是因为`a`属性只会存在于`DemoDataClass`的实例中。它将是一个公共属性，我们可以获取和设置，除非类被冻结。但是`b`和`c`存在为类属性，`b`保存了`b`实例属性的默认值，而`c`只是一个不会绑定到实例的类属性。

现在让我们看看`DemoDataClass`实例的外观：

```py
>>> dc = DemoDataClass(9)
>>> dc.a
9
>>> dc.b
1.1
>>> dc.c
'spam'
```

再次，`a`和`b`是实例属性，`c`是我们通过实例获取的类属性。

如前所述，`DemoDataClass`实例是可变的—并且在运行时不进行类型检查：

```py
>>> dc.a = 10
>>> dc.b = 'oops'
```

我们甚至可以做更愚蠢的赋值：

```py
>>> dc.c = 'whatever'
>>> dc.z = 'secret stash'
```

现在`dc`实例有一个`c`属性—但这并不会改变`c`类属性。我们可以添加一个新的`z`属性。这是正常的 Python 行为：常规实例可以有自己的属性，这些属性不会出现在类中。⁷

# 关于 @dataclass 的更多信息

到目前为止，我们只看到了`@dataclass`的简单示例。装饰器接受几个关键字参数。这是它的签名：

```py
@dataclass(*, init=True, repr=True, eq=True, order=False,
              unsafe_hash=False, frozen=False)
```

第一个位置的`*`表示剩余参数只能通过关键字传递。表格 5-2 描述了这些参数。

表格 5-2\. `@dataclass`装饰器接受的关键字参数

| 选项 | 含义 | 默认值 | 注释 |
| --- | --- | --- | --- |
| `init` | 生成`__init__` | `True` | 如果用户实现了`__init__`，则忽略。 |
| `repr` | 生成`__repr__` | `True` | 如果用户实现了`__repr__`，则忽略。 |
| `eq` | 生成`__eq__` | `True` | 如果用户实现了`__eq__`，则忽略。 |
| `order` | 生成`__lt__`、`__le__`、`__gt__`、`__ge__` | `False` | 如果为`True`，则在`eq=False`时引发异常，或者如果定义或继承了将要生成的任何比较方法。 |
| `unsafe_hash` | 生成`__hash__` | `False` | 复杂的语义和几个注意事项—参见：[数据类文档](https://fpy.li/5-7)。 |
| `frozen` | 使实例“不可变” | `False` | 实例将相对安全免受意外更改，但实际上并非不可变。^(a) |
| ^(a) `@dataclass`通过生成`__setattr__`和`__delattr__`来模拟不可变性，当用户尝试设置或删除字段时，会引发`dataclass.FrozenInstanceError`—`AttributeError`的子类。 |

默认设置实际上是最常用的常见用例的最有用设置。你更有可能从默认设置中更改的选项是：

`frozen=True`

防止对类实例的意外更改。

`order=True`

允许对数据类的实例进行排序。

鉴于 Python 对象的动态特性，一个好奇的程序员很容易绕过`frozen=True`提供的保护。但是在代码审查中，这些必要的技巧应该很容易被发现。

如果`eq`和`frozen`参数都为`True`，`@dataclass`会生成一个合适的`__hash__`方法，因此实例将是可散列的。生成的`__hash__`将使用所有未被单独排除的字段数据，使用我们将在“字段选项”中看到的字段选项。如果`frozen=False`（默认值），`@dataclass`将将`__hash__`设置为`None`，表示实例是不可散列的，因此覆盖了任何超类的`__hash__`。

[PEP 557—数据类](https://fpy.li/pep557)对`unsafe_hash`有如下说明：

> 虽然不建议这样做，但你可以通过`unsafe_hash=True`强制数据类创建一个`__hash__`方法。如果你的类在逻辑上是不可变的，但仍然可以被改变，这可能是一个特殊的用例，应该仔细考虑。

我会保留`unsafe_hash`。如果你觉得必须使用该选项，请查看[`dataclasses.dataclass`文档](https://fpy.li/5-7)。

可以在字段级别进一步定制生成的数据类。

## 字段选项

我们已经看到了最基本的字段选项：使用类型提示提供（或不提供）默认值。你声明的实例字段将成为生成的`__init__`中的参数。Python 不允许在具有默认值的参数之后使用没有默认值的参数，因此在声明具有默认值的字段之后，所有剩余字段必须也具有默认值。

可变默认值是初学 Python 开发者常见的错误来源。在函数定义中，当函数的一个调用改变了默认值时，易变默认值很容易被破坏，从而改变了后续调用的行为——这是我们将在“可变类型作为参数默认值：不好的想法”中探讨的问题（第六章）。类属性经常被用作实例的默认属性值，包括在数据类中。`@dataclass`使用类型提示中的默认值生成带有默认值的参数供`__init__`使用。为了防止错误，`@dataclass`拒绝了示例 5-13 中的类定义。

##### 示例 5-13\. *dataclass/club_wrong.py*：这个类会引发`ValueError`。

```py
@dataclass
class ClubMember:
    name: str
    guests: list = []
```

如果加载了具有`ClubMember`类的模块，你会得到这个：

```py
$ python3 club_wrong.py
Traceback (most recent call last):
  File "club_wrong.py", line 4, in <module>
    class ClubMember:
  ...several lines omitted...
ValueError: mutable default <class 'list'> for field guests is not allowed:
use default_factory
```

`ValueError`消息解释了问题并建议解决方案：使用`default_factory`。示例 5-14 展示了如何纠正`ClubMember`。

##### 示例 5-14\. *dataclass/club.py*：这个`ClubMember`定义可行

```py
from dataclasses import dataclass, field

@dataclass
class ClubMember:
    name: str
    guests: list = field(default_factory=list)
```

在示例 5-14 的`guests`字段中，不是使用字面列表作为默认值，而是通过调用`dataclasses.field`函数并使用`default_factory=list`来设置默认值。

`default_factory`参数允许你提供一个函数、类或任何其他可调用对象，每次创建数据类的实例时都会调用它以构建默认值。这样，`ClubMember`的每个实例都将有自己的`list`，而不是所有实例共享来自类的相同`list`，这很少是我们想要的，通常是一个错误。

###### 警告

很好的是`@dataclass`拒绝了具有`list`默认值的字段的类定义。但是，请注意，这是一个部分解决方案，仅适用于`list`、`dict`和`set`。其他用作默认值的可变值不会被`@dataclass`标记。你需要理解问题并记住使用默认工厂来设置可变默认值。

如果你浏览[`dataclasses`](https://fpy.li/5-9)模块文档，你会看到一个用新语法定义的`list`字段，就像示例 5-15 中一样。

##### 示例 5-15\. *dataclass/club_generic.py*：这个`ClubMember`定义更加精确

```py
from dataclasses import dataclass, field

@dataclass
class ClubMember:
    name: str
    guests: list[str] = field(default_factory=list)  # ①
```

①

`list[str]`表示“一个`str`的列表”。

新的语法`list[str]`是一个参数化的泛型类型：自 Python 3.9 以来，`list`内置接受方括号表示法来指定列表项的类型。

###### 警告

在 Python 3.9 之前，内置的集合不支持泛型类型表示法。作为临时解决方法，在`typing`模块中有相应的集合类型。如果你需要在 Python 3.8 或更早版本中使用参数化的`list`类型提示，你必须导入`typing`中的`List`类型并使用它：`List[str]`。有关此问题的更多信息，请参阅“遗留支持和已弃用的集合类型”。

我们将在第八章中介绍泛型。现在，请注意示例 5-14 和 5-15 都是正确的，Mypy 类型检查器不会对这两个类定义提出任何异议。

区别在于`guests: list`表示`guests`可以是任何类型对象的`list`，而`guests: list[str]`表示`guests`必须是每个项都是`str`的`list`。这将允许类型检查器在将无效项放入列表的代码中找到（一些）错误，或者从中读取项。

`default_factory` 很可能是`field`函数的最常见选项，但还有其他几个选项，列在表 5-3 中。

表 5-3\. `field`函数接受的关键字参数

| 选项 | 含义 | 默认值 |
| --- | --- | --- |
| `default` | 字段的默认值 | `_MISSING_TYPE`^(a) |
| `default_factory` | 用于生成默认值的 0 参数函数 | `_MISSING_TYPE` |
| `init` | 在`__init__`参数中包含字段 | `True` |
| `repr` | 在`__repr__`中包含字段 | `True` |
| `compare` | 在比较方法`__eq__`、`__lt__`等中使用字段 | `True` |
| `hash` | 在`__hash__`计算中包含字段 | `None`^(b) |
| `metadata` | 具有用户定义数据的映射；被`@dataclass`忽略 | `None` |
| ^(a) `dataclass._MISSING_TYPE` 是一个标志值，表示未提供选项。它存在的原因是我们可以将`None`设置为实际的默认值，这是一个常见用例。^(b) 选项`hash=None`表示只有在`compare=True`时，该字段才会在`__hash__`中使用。 |

`default`选项的存在是因为`field`调用取代了字段注释中的默认值。如果要创建一个默认值为`False`的`athlete`字段，并且还要在`__repr__`方法中省略该字段，你可以这样写：

```py
@dataclass
class ClubMember:
    name: str
    guests: list = field(default_factory=list)
    athlete: bool = field(default=False, repr=False)
```

## 后初始化处理

由 `@dataclass` 生成的 `__init__` 方法只接受传递的参数并将它们分配给实例字段的实例属性，或者如果缺少参数，则分配它们的默认值。但您可能需要做的不仅仅是这些来初始化实例。如果是这种情况，您可以提供一个 `__post_init__` 方法。当存在该方法时，`@dataclass` 将在生成的 `__init__` 中添加代码，以调用 `__post_init__` 作为最后一步。

`__post_init__` 的常见用例是验证和基于其他字段计算字段值。我们将学习一个简单的示例，该示例使用 `__post_init__` 来实现这两个目的。

首先，让我们看看名为 `HackerClubMember` 的 `ClubMember` 子类的预期行为，如 示例 5-16 中的文档测试所描述。

##### 示例 5-16\. *dataclass/hackerclub.py*: `HackerClubMember` 的文档测试

```py
"""
``HackerClubMember`` objects accept an optional ``handle`` argument::

 >>> anna = HackerClubMember('Anna Ravenscroft', handle='AnnaRaven')
 >>> anna
 HackerClubMember(name='Anna Ravenscroft', guests=[], handle='AnnaRaven')

If ``handle`` is omitted, it's set to the first part of the member's name::

 >>> leo = HackerClubMember('Leo Rochael')
 >>> leo
 HackerClubMember(name='Leo Rochael', guests=[], handle='Leo')

Members must have a unique handle. The following ``leo2`` will not be created,
because its ``handle`` would be 'Leo', which was taken by ``leo``::

 >>> leo2 = HackerClubMember('Leo DaVinci')
 Traceback (most recent call last):
 ...
 ValueError: handle 'Leo' already exists.

To fix, ``leo2`` must be created with an explicit ``handle``::

 >>> leo2 = HackerClubMember('Leo DaVinci', handle='Neo')
 >>> leo2
 HackerClubMember(name='Leo DaVinci', guests=[], handle='Neo')
"""
```

请注意，我们必须将 `handle` 作为关键字参数提供，因为 `HackerClubMember` 继承自 `ClubMember` 的 `name` 和 `guests`，并添加了 `handle` 字段。生成的 `HackerClubMember` 的文档字符串显示了构造函数调用中字段的顺序：

```py
>>> HackerClubMember.__doc__
"HackerClubMember(name: str, guests: list = <factory>, handle: str = '')"
```

这里，`<factory>` 是指某个可调用对象将为 `guests` 生成默认值的简便方式（在我们的例子中，工厂是 `list` 类）。关键是：要提供一个 `handle` 但没有 `guests`，我们必须将 `handle` 作为关键字参数传递。

[`dataclasses` 模块文档中的“继承”部分](https://fpy.li/5-10) 解释了在存在多级继承时如何计算字段的顺序。

###### 注意

在 第十四章 中，我们将讨论错误使用继承，特别是当超类不是抽象类时。创建数据类的层次结构通常不是一个好主意，但在这里，它帮助我们缩短了 示例 5-17 的长度，侧重于 `handle` 字段声明和 `__post_init__` 验证。

示例 5-17 展示了实现方式。

##### 示例 5-17\. *dataclass/hackerclub.py*: `HackerClubMember` 的代码

```py
from dataclasses import dataclass
from club import ClubMember

@dataclass
class HackerClubMember(ClubMember):                         # ①
    all_handles = set()                                     # ②
    handle: str = ''                                        # ③

    def __post_init__(self):
        cls = self.__class__                                # ④
        if self.handle == '':                               # ⑤
            self.handle = self.name.split()[0]
        if self.handle in cls.all_handles:                  # ⑥
            msg = f'handle {self.handle!r} already exists.'
            raise ValueError(msg)
        cls.all_handles.add(self.handle)                    # ⑦
```

①

`HackerClubMember` 扩展了 `ClubMember`。

②

`all_handles` 是一个类属性。

③

`handle` 是一个类型为 `str` 的实例字段，其默认值为空字符串；这使其成为可选的。

④

获取实例的类。

⑤

如果 `self.handle` 是空字符串，则将其设置为 `name` 的第一部分。

⑥

如果 `self.handle` 在 `cls.all_handles` 中，则引发 `ValueError`。

⑦

将新的 `handle` 添加到 `cls.all_handles`。

示例 5-17 的功能正常，但对于静态类型检查器来说并不令人满意。接下来，我们将看到原因以及如何解决。

## 类型化类属性

如果我们使用 Mypy 对 示例 5-17 进行类型检查，我们会受到批评：

```py
$ mypy hackerclub.py
hackerclub.py:37: error: Need type annotation for "all_handles"
(hint: "all_handles: Set[<type>] = ...")
Found 1 error in 1 file (checked 1 source file)
```

不幸的是，Mypy 提供的提示（我在审阅时使用的版本是 0.910）在 `@dataclass` 使用的上下文中并不有用。首先，它建议使用 `Set`，但我使用的是 Python 3.9，因此可以使用 `set`，并避免从 `typing` 导入 `Set`。更重要的是，如果我们向 `all_handles` 添加一个类型提示，如 `set[…]`，`@dataclass` 将找到该注释，并将 `all_handles` 变为实例字段。我们在“检查使用 dataclass 装饰的类”中看到了这种情况。

在 [PEP 526—变量注释的语法](https://fpy.li/5-11) 中定义的解决方法很丑陋。为了编写带有类型提示的类变量，我们需要使用一个名为 `typing.ClassVar` 的伪类型，它利用泛型 `[]` 符号来设置变量的类型，并声明它为类属性。

为了让类型检查器和 `@dataclass` 满意，我们应该在 示例 5-17 中这样声明 `all_handles`：

```py
    all_handles: ClassVar[set[str]] = set()
```

那个类型提示表示：

> `all_handles` 是一个类型为 `set`-of-`str` 的类属性，其默认值为空 `set`。

要编写该注释的代码，我们必须从 `typing` 模块导入 `ClassVar`。

`@dataclass` 装饰器不关心注释中的类型，除了两种情况之一，这就是其中之一：如果类型是 `ClassVar`，则不会为该属性生成实例字段。

在声明*仅初始化变量*时，字段类型对 `@dataclass` 有影响的另一种情况是我们接下来要讨论的。

## 不是字段的初始化变量

有时，您可能需要向 `__init__` 传递不是实例字段的参数。这些参数被 [`dataclasses` 文档](https://fpy.li/initvar) 称为*仅初始化变量*。要声明这样的参数，`dataclasses` 模块提供了伪类型 `InitVar`，其使用与 `typing.ClassVar` 相同的语法。文档中给出的示例是一个数据类，其字段从数据库初始化，并且必须将数据库对象传递给构造函数。

示例 5-18 展示了说明[“仅初始化变量”部分](https://fpy.li/initvar)的代码。

##### 示例 5-18\. 来自 [`dataclasses`](https://fpy.li/initvar) 模块文档的示例

```py
@dataclass
class C:
    i: int
    j: int = None
    database: InitVar[DatabaseType] = None

    def __post_init__(self, database):
        if self.j is None and database is not None:
            self.j = database.lookup('j')

c = C(10, database=my_database)
```

注意 `database` 属性的声明方式。`InitVar` 将阻止 `@dataclass` 将 `database` 视为常规字段。它不会被设置为实例属性，并且 `dataclasses.fields` 函数不会列出它。但是，`database` 将是生成的 `__init__` 将接受的参数之一，并且也将传递给 `__post_init__`。如果您编写该方法，必须在方法签名中添加相应的参数，如示例 5-18 中所示。

这个相当长的 `@dataclass` 概述涵盖了最有用的功能——其中一些出现在之前的部分中，比如“主要特性”，在那里我们并行讨论了所有三个数据类构建器。[`dataclasses` 文档](https://fpy.li/initvar) 和 [PEP 526—变量注释的语法](https://fpy.li/pep526) 中有所有细节。

在下一节中，我将展示一个更长的示例，使用 `@dataclass`。

## @dataclass 示例：Dublin Core 资源记录

经常使用 `@dataclass` 构建的类将具有比目前呈现的非常简短示例更多的字段。[Dublin Core](https://fpy.li/5-12) 为一个更典型的 `@dataclass` 示例提供了基础。

> Dublin Core Schema 是一组可以用于描述数字资源（视频、图像、网页等）以及实体资源（如书籍或 CD）和艺术品等对象的词汇术语。⁸
> 
> 维基百科上的 Dublin Core

标准定义了 15 个可选字段；示例 5-19 中的 `Resource` 类使用了其中的 8 个。

##### 示例 5-19\. *dataclass/resource.py*: `Resource` 类的代码，基于 Dublin Core 术语

```py
from dataclasses import dataclass, field
from typing import Optional
from enum import Enum, auto
from datetime import date

class ResourceType(Enum):  # ①
    BOOK = auto()
    EBOOK = auto()
    VIDEO = auto()

@dataclass
class Resource:
    """Media resource description."""
    identifier: str                                    # ②
    title: str = '<untitled>'                          # ③
    creators: list[str] = field(default_factory=list)
    date: Optional[date] = None                        # ④
    type: ResourceType = ResourceType.BOOK             # ⑤
    description: str = ''
    language: str = ''
    subjects: list[str] = field(default_factory=list)
```

①

这个 `Enum` 将为 `Resource.type` 字段提供类型安全的值。

②

`identifier` 是唯一必需的字段。

③

`title` 是第一个具有默认值的字段。这迫使下面的所有字段都提供默认值。

④

`date` 的值可以是 `datetime.date` 实例，或者是 `None`。

⑤

`type` 字段的默认值是 `ResourceType.BOOK`。

示例 5-20 展示了一个 doctest，演示了代码中 `Resource` 记录的外观。

##### 示例 5-20\. *dataclass/resource.py*: `Resource` 类的代码，基于 Dublin Core 术语

```py
    >>> description = 'Improving the design of existing code'
    >>> book = Resource('978-0-13-475759-9', 'Refactoring, 2nd Edition',
    ...     ['Martin Fowler', 'Kent Beck'], date(2018, 11, 19),
    ...     ResourceType.BOOK, description, 'EN',
    ...     ['computer programming', 'OOP'])
    >>> book  # doctest: +NORMALIZE_WHITESPACE
    Resource(identifier='978-0-13-475759-9', title='Refactoring, 2nd Edition',
    creators=['Martin Fowler', 'Kent Beck'], date=datetime.date(2018, 11, 19),
    type=<ResourceType.BOOK: 1>, description='Improving the design of existing code',
    language='EN', subjects=['computer programming', 'OOP'])
```

由 `@dataclass` 生成的 `__repr__` 是可以的，但我们可以使其更易读。这是我们希望从 `repr(book)` 得到的格式：

```py
    >>> book  # doctest: +NORMALIZE_WHITESPACE
    Resource(
        identifier = '978-0-13-475759-9',
        title = 'Refactoring, 2nd Edition',
        creators = ['Martin Fowler', 'Kent Beck'],
        date = datetime.date(2018, 11, 19),
        type = <ResourceType.BOOK: 1>,
        description = 'Improving the design of existing code',
        language = 'EN',
        subjects = ['computer programming', 'OOP'],
    )
```

示例 5-21 是用于生成最后代码片段中所示格式的`__repr__`的代码。此示例使用`dataclass.fields`来获取数据类字段的名称。

##### 示例 5-21\. `dataclass/resource_repr.py`：在示例 5-19 中实现的`Resource`类中实现的`__repr__`方法的代码

```py
    def __repr__(self):
        cls = self.__class__
        cls_name = cls.__name__
        indent = ' ' * 4
        res = [f'{cls_name}(']                            # ①
        for f in fields(cls):                             # ②
            value = getattr(self, f.name)                 # ③
            res.append(f'{indent}{f.name} = {value!r},')  # ④

        res.append(')')                                   # ⑤
        return '\n'.join(res)                             # ⑥
```

①

开始`res`列表以构建包含类名和开括号的输出字符串。

②

对于类中的每个字段`f`…

③

…从实例中获取命名属性。

④

附加一个缩进的行，带有字段的名称和`repr(value)`—这就是`!r`的作用。

⑤

附加闭括号。

⑥

从`res`构建一个多行字符串并返回它。

通过这个受到俄亥俄州都柏林灵感的例子，我们结束了对 Python 数据类构建器的介绍。

数据类很方便，但如果过度使用它们，您的项目可能会受到影响。接下来的部分将进行解释。

# 数据类作为代码异味

无论您是通过自己编写所有代码来实现数据类，还是利用本章描述的类构建器之一，都要意识到它可能在您的设计中信号问题。

在[*重构：改善现有代码设计，第 2 版*](https://martinfowler.com/books/refactoring.html)（Addison-Wesley）中，Martin Fowler 和 Kent Beck 提供了一个“代码异味”目录—代码中可能表明需要重构的模式。标题为“数据类”的条目开头是这样的：

> 这些类具有字段、获取和设置字段的方法，除此之外什么都没有。这样的类是愚蠢的数据持有者，往往被其他类以过于详细的方式操纵。

在福勒的个人网站上，有一篇标题为[“代码异味”](https://fpy.li/5-14)的启发性文章。这篇文章与我们的讨论非常相关，因为他将*数据类*作为代码异味的一个例子，并建议如何处理。以下是完整的文章。⁹

面向对象编程的主要思想是将行为和数据放在同一个代码单元中：一个类。如果一个类被广泛使用但本身没有重要的行为，那么处理其实例的代码可能分散在整个系统的方法和函数中（甚至重复）—这是维护头痛的根源。这就是为什么福勒的重构涉及将责任带回到数据类中。

考虑到这一点，有几种常见情况下，拥有一个几乎没有行为的数据类是有意义的。

## 数据类作为脚手架

在这种情况下，数据类是一个初始的、简单的类实现，用于启动新项目或模块。随着时间的推移，该类应该拥有自己的方法，而不是依赖其他类的方法来操作其实例。脚手架是临时的；最终，您的自定义类可能会完全独立于您用来启动它的构建器。

Python 也用于快速问题解决和实验，然后保留脚手架是可以的。

## 数据类作为中间表示

数据类可用于构建即将导出到 JSON 或其他交换格式的记录，或者保存刚刚导入的数据，跨越某些系统边界。Python 的数据类构建器都提供了一个方法或函数，将实例转换为普通的`dict`，您总是可以调用构造函数，使用作为关键字参数扩展的`**`的`dict`。这样的`dict`非常接近 JSON 记录。

在这种情况下，数据类实例应被视为不可变对象—即使字段是可变的，也不应在其处于这种中间形式时更改它们。如果这样做，您将失去将数据和行为紧密结合的主要优势。当导入/导出需要更改值时，您应该实现自己的构建器方法，而不是使用给定的“作为字典”方法或标准构造函数。

现在我们改变主题，看看如何编写匹配任意类实例而不仅仅是我们在“使用序列进行模式匹配”和“使用映射进行模式匹配”中看到的序列和映射的模式。

# 匹配类实例

类模式旨在通过类型和—可选地—属性来匹配类实例。类模式的主题可以是任何类实例，不仅仅是数据类的实例。¹⁰

类模式有三种变体：简单、关键字和位置。我们将按照这个顺序来学习它们。

## 简单类模式

我们已经看到了一个简单类模式作为子模式在“使用序列进行模式匹配”中的示例：

```py
        case [str(name), _, _, (float(lat), float(lon))]:
```

该模式匹配一个四项序列，其中第一项必须是`str`的实例，最后一项必须是一个包含两个`float`实例的 2 元组。

类模式的语法看起来像一个构造函数调用。以下是一个类模式，匹配`float`值而不绑定变量（如果需要，case 体可以直接引用`x`）：

```py
    match x:
        case float():
            do_something_with(x)
```

但这很可能是您代码中的一个错误：

```py
    match x:
        case float:  # DANGER!!!
            do_something_with(x)
```

在前面的示例中，`case float:`匹配任何主题，因为 Python 将`float`视为一个变量，然后将其绑定到主题。

`float(x)`的简单模式语法是一个特例，仅适用于列在[“类模式”](https://fpy.li/5-16)部分末尾的 PEP 634—结构化模式匹配：规范中的九个受祝福的内置类型：

```py
bytes   dict   float   frozenset   int   list   set   str   tuple
```

在这些类中，看起来像构造函数参数的变量—例如，在我们之前看到的序列模式中的`str(name)`中的`x`—被绑定到整个主题实例或与子模式匹配的主题部分，如示例中的`str(name)`所示：

```py
        case [str(name), _, _, (float(lat), float(lon))]:
```

如果类不是这九个受祝福的内置类型之一，那么类似参数的变量表示要与该类实例的属性进行匹配的模式。

## 关键字类模式

要了解如何使用关键字类模式，请考虑以下`City`类和示例 5-22 中的五个实例。

##### 示例 5-22\. `City`类和一些实例

```py
import typing

class City(typing.NamedTuple):
    continent: str
    name: str
    country: str

cities = [
    City('Asia', 'Tokyo', 'JP'),
    City('Asia', 'Delhi', 'IN'),
    City('North America', 'Mexico City', 'MX'),
    City('North America', 'New York', 'US'),
    City('South America', 'São Paulo', 'BR'),
]
```

给定这些定义，以下函数将返回一个亚洲城市列表：

```py
def match_asian_cities():
    results = []
    for city in cities:
        match city:
            case City(continent='Asia'):
                results.append(city)
    return results
```

模式`City(continent='Asia')`匹配任何`City`实例，其中`continent`属性值等于`'Asia'`，而不管其他属性的值如何。

如果您想收集`country`属性的值，您可以编写：

```py
def match_asian_countries():
    results = []
    for city in cities:
        match city:
            case City(continent='Asia', country=cc):
                results.append(cc)
    return results
```

模式`City(continent='Asia', country=cc)`匹配与之前相同的亚洲城市，但现在`cc`变量绑定到实例的`country`属性。如果模式变量也称为`country`，这也适用：

```py
        match city:
            case City(continent='Asia', country=country):
                results.append(country)
```

关键字类模式非常易读，并且适用于具有公共实例属性的任何类，但它们有点冗长。

位置类模式在某些情况下更方便，但它们需要主题类的显式支持，我们将在下一节中看到。

## 位置类模式

给定示例 5-22 中的定义，以下函数将使用位置类模式返回一个亚洲城市列表：

```py
def match_asian_cities_pos():
    results = []
    for city in cities:
        match city:
            case City('Asia'):
                results.append(city)
    return results
```

模式`City('Asia')`匹配任何`City`实例，其中第一个属性值为`'Asia'`，而不管其他属性的值如何。

如果您要收集`country`属性的值，您可以编写：

```py
def match_asian_countries_pos():
    results = []
    for city in cities:
        match city:
            case City('Asia', _, country):
                results.append(country)
    return results
```

模式`City('Asia', _, country)`匹配与之前相同的城市，但现在`country`变量绑定到实例的第三个属性。

我提到了“第一个”或“第三个”属性，但这到底是什么意思？

使`City`或任何类与位置模式配合工作的是一个名为`__match_args__`的特殊类属性的存在，这是本章中的类构建器自动创建的。这是`City`类中`__match_args__`的值：

```py
>>> City.__match_args__
('continent', 'name', 'country')
```

如您所见，`__match_args__`声明了属性的名称，按照它们在位置模式中使用的顺序。

在“支持位置模式匹配”中，我们将编写代码为一个我们将在没有类构建器帮助的情况下创建的类定义`__match_args__`。

###### 提示

您可以在模式中组合关键字和位置参数。可能列出用于匹配的实例属性中的一些，但不是全部，可能需要在模式中除了位置参数之外还使用关键字参数。

是时候进行章节总结了。

# 章节总结

本章的主题是数据类构建器`collections.namedtuple`，`typing.NamedTuple`和`dataclasses.dataclass`。我们看到，每个都从作为工厂函数参数提供的描述生成数据类，或者从`class`语句中生成具有类型提示的后两者。特别是，两种命名元组变体都生成`tuple`子类，仅添加按名称访问字段的能力，并提供一个列出字段名称的`_fields`类属性，作为字符串元组。

接下来，我们并排研究了三个类构建器的主要特性，包括如何将实例数据提取为`dict`，如何获取字段的名称和默认值，以及如何从现有实例创建新实例。

这促使我们首次研究类型提示，特别是用于注释`class`语句中属性的提示，使用 Python 3.6 中引入的符号，[PEP 526—变量注释语法](https://fpy.li/pep526)。总体而言，类型提示最令人惊讶的方面可能是它们在运行时根本没有任何影响。Python 仍然是一种动态语言。需要外部工具，如 Mypy，利用类型信息通过对源代码的静态分析来检测错误。在对 PEP 526 中的语法进行基本概述后，我们研究了在普通类和由`typing.NamedTuple`和`@dataclass`构建的类中注释的效果。

接下来，我们介绍了`@dataclass`提供的最常用功能以及`dataclasses.field`函数的`default_factory`选项。我们还研究了在数据类上下文中重要的特殊伪类型提示`typing.ClassVar`和`dataclasses.InitVar`。这个主题以基于 Dublin Core Schema 的示例结束，示例说明了如何使用`dataclasses.fields`在自定义的`__repr__`中迭代`Resource`实例的属性。

然后，我们警告可能滥用数据类，违反面向对象编程的基本原则：数据和触及数据的函数应该在同一个类中。没有逻辑的类可能是放错逻辑的迹象。

在最后一节中，我们看到了模式匹配如何与任何类的实例一起使用，而不仅仅是本章介绍的类构建器构建的类。

# 进一步阅读

Python 对我们涵盖的数据类构建器的标准文档非常好，并且有相当多的小例子。

对于特别的 `@dataclass`，[PEP 557—数据类](https://fpy.li/pep557) 的大部分内容都被复制到了 [`dataclasses`](https://fpy.li/5-9) 模块文档中。但 [PEP 557](https://fpy.li/pep557) 还有一些非常信息丰富的部分没有被复制，包括 [“为什么不只使用 namedtuple？”](https://fpy.li/5-18)，[“为什么不只使用 typing.NamedTuple？”](https://fpy.li/5-19)，以及以这个问答结束的 [“原理” 部分](https://fpy.li/5-20)：

> 在哪些情况下不适合使用数据类？
> 
> API 兼容元组或字典是必需的。需要超出 PEPs 484 和 526 提供的类型验证，或者需要值验证或转换。
> 
> Eric V. Smith，PEP 557 “原理”

在 [*RealPython.com*](https://fpy.li/5-21) 上，Geir Arne Hjelle 写了一篇非常完整的 [“Python 3.7 中数据类的终极指南”](https://fpy.li/5-22)。

在 PyCon US 2018 上，Raymond Hettinger 提出了 [“数据类：终结所有代码生成器的代码生成器”（视频）](https://fpy.li/5-23)。

对于更多功能和高级功能，包括验证，由 Hynek Schlawack 领导的 [*attrs* 项目](https://fpy.li/5-24) 在 `dataclasses` 出现多年之前，并提供更多功能，承诺“通过解除您实现对象协议（也称为 dunder 方法）的繁琐工作，带回编写类的乐趣。” Eric V. Smith 在 PEP 557 中承认 *attrs* 对 `@dataclass` 的影响。这可能包括 Smith 最重要的 API 决定：使用类装饰器而不是基类和/或元类来完成工作。

Glyph——Twisted 项目的创始人——在 [“每个人都需要的一个 Python 库”](https://fpy.li/5-25) 中写了一篇关于 *attrs* 的优秀介绍。*attrs* 文档包括 [替代方案的讨论](https://fpy.li/5-26)。

书籍作者、讲师和疯狂的计算机科学家 Dave Beazley 写了 [*cluegen*](https://fpy.li/5-27)，又一个数据类生成器。如果你看过 Dave 的任何演讲，你就知道他是一个从第一原则开始元编程 Python 的大师。因此，我发现从 *cluegen* 的 *README.md* 文件中了解到鼓励他编写 Python 的 `@dataclass` 替代方案的具体用例，以及他提出解决问题方法的哲学，与提供工具相对立：工具可能一开始使用更快，但方法更灵活，可以带你走得更远。

将 *数据类* 视为代码坏味道，我找到的最好的来源是 Martin Fowler 的书 *重构*，第二版。这个最新版本缺少了本章前言的引语，“数据类就像孩子一样……”，但除此之外，这是 Fowler 最著名的书的最佳版本，特别适合 Python 程序员，因为示例是用现代 JavaScript 编写的，这比 Java 更接近 Python——第一版的语言。

网站 [*Refactoring Guru*](https://fpy.li/5-28) 也对 [数据类代码坏味道](https://fpy.li/5-29) 进行了描述。

¹ 来自《重构》，第一版，第三章，“代码中的坏味道，数据类”部分，第 87 页（Addison-Wesley）。

² 元类是 第二十四章，“类元编程” 中涵盖的主题之一。

³ 类装饰器在 第二十四章，“类元编程” 中有介绍，与元类一起。两者都是超出继承可能的方式来定制类行为。

⁴ 如果你了解 Ruby，你会知道在 Ruby 程序员中，注入方法是一种众所周知但有争议的技术。在 Python 中，这并不常见，因为它不适用于任何内置类型——`str`，`list` 等。我认为这是 Python 的一个福音。

⁵ 在类型提示的背景下，`None`不是`NoneType`的单例，而是`NoneType`本身的别名。当我们停下来思考时，这看起来很奇怪，但符合我们的直觉，并且使函数返回注解在返回`None`的常见情况下更容易阅读。

⁶ Python 没有*未定义*的概念，这是 JavaScript 设计中最愚蠢的错误之一。感谢 Guido！

⁷ 在`__init__`之后设置属性会破坏“dict 工作方式的实际后果”中提到的`__dict__`键共享内存优化。

⁸ 来源：[都柏林核心](https://fpy.li/5-13) 英文维基百科文章。

⁹ 我很幸运在 Thoughtworks 有马丁·福勒作为同事，所以只用了 20 分钟就得到了他的许可。

¹⁰ 我将这部分内容放在这里，因为这是最早关注用户定义类的章节，我认为与类一起使用模式匹配太重要，不能等到书的第二部分。我的理念是：了解如何使用类比定义类更重要。
