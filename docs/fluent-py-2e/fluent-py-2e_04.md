# 第三章：字典和集合

> Python 基本上是用大量语法糖包装的字典。
> 
> Lalo Martins，早期数字游牧民和 Pythonista

我们在所有的 Python 程序中都使用字典。即使不是直接在我们的代码中，也是间接的，因为`dict`类型是 Python 实现的基本部分。类和实例属性、模块命名空间和函数关键字参数是内存中由字典表示的核心 Python 构造。`__builtins__.__dict__`存储所有内置类型、对象和函数。

由于其关键作用，Python 字典经过高度优化，并持续改进。*哈希表*是 Python 高性能字典背后的引擎。

其他基于哈希表的内置类型是`set`和`frozenset`。这些提供比您在其他流行语言中遇到的集合更丰富的 API 和运算符。特别是，Python 集合实现了集合理论中的所有基本操作，如并集、交集、子集测试等。通过它们，我们可以以更声明性的方式表达算法，避免大量嵌套循环和条件语句。

以下是本章的简要概述：

+   用于构建和处理`dicts`和映射的现代语法，包括增强的解包和模式匹配

+   映射类型的常见方法

+   丢失键的特殊处理

+   标准库中`dict`的变体

+   `set`和`frozenset`类型

+   哈希表在集合和字典行为中的影响。

# 本章的新内容

这第二版中的大部分变化涵盖了与映射类型相关的新功能：

+   “现代字典语法”介绍了增强的解包语法以及合并映射的不同方式，包括自 Python 3.9 起由`dicts`支持的`|`和`|=`运算符。

+   “使用映射进行模式匹配”演示了自 Python 3.10 起使用`match/case`处理映射。

+   “collections.OrderedDict”现在专注于`dict`和`OrderedDict`之间的细微但仍然相关的差异——考虑到自 Python 3.6 起`dict`保留键插入顺序。

+   由`dict.keys`、`dict.items`和`dict.values`返回的视图对象的新部分：“字典视图”和“字典视图上的集合操作”。

`dict`和`set`的基础实现仍然依赖于哈希表，但`dict`代码有两个重要的优化，可以节省内存并保留键在`dict`中的插入顺序。“dict 工作原理的实际后果”和“集合工作原理的实际后果”总结了您需要了解的内容，以便很好地使用它们。

###### 注意

在这第二版中增加了 200 多页后，我将可选部分[“集合和字典的内部”](https://fpy.li/hashint)移至[*fluentpython.com*](http://fluentpython.com)伴随网站。更新和扩展的[18 页文章](https://fpy.li/hashint)包括关于以下内容的解释和图表：

+   哈希表算法和数据结构，从在`set`中的使用开始，这更容易理解。

+   保留`dict`实例中键插入顺序的内存优化（自 Python 3.6 起）。

+   用于保存实例属性的字典的键共享布局——用户定义对象的`__dict__`（自 Python 3.3 起实现的优化）。

# 现代字典语法

接下来的部分描述了用于构建、解包和处理映射的高级语法特性。其中一些特性在语言中并不新鲜，但对您可能是新的。其他需要 Python 3.9（如`|`运算符）或 Python 3.10（如`match/case`）的特性。让我们从其中一个最好且最古老的特性开始。

## 字典推导式

自 Python 2.7 起，列表推导和生成器表达式的语法已经适应了 `dict` 推导（以及我们即将讨论的 `set` 推导）。*dictcomp*（dict 推导）通过从任何可迭代对象中获取 `key:value` 对来构建一个 `dict` 实例。示例 3-1 展示了使用 `dict` 推导从相同的元组列表构建两个字典的用法。

##### 示例 3-1\. `dict` 推导示例

```py
>>> dial_codes =                                                   ![1
...     (880, 'Bangladesh'),
...     (55,  'Brazil'),
...     (86,  'China'),
...     (91,  'India'),
...     (62,  'Indonesia'),
...     (81,  'Japan'),
...     (234, 'Nigeria'),
...     (92,  'Pakistan'),
...     (7,   'Russia'),
...     (1,   'United States'),
... ]
>>> country_dial = {country: code for code, country in dial_codes}  # ②
>>> country_dial
{'Bangladesh': 880, 'Brazil': 55, 'China': 86, 'India': 91, 'Indonesia': 62, 'Japan': 81, 'Nigeria': 234, 'Pakistan': 92, 'Russia': 7, 'United States': 1} >>> {code: country.upper()                                          # ③
...     for country, code in sorted(country_dial.items())
...     if code < 70}
{55: 'BRAZIL', 62: 'INDONESIA', 7: 'RUSSIA', 1: 'UNITED STATES'}
```

①

可以直接将类似 `dial_codes` 的键值对可迭代对象传递给 `dict` 构造函数，但是…

②

…在这里我们交换了键值对：`country` 是键，`code` 是值。

③

按名称对 `country_dial` 进行排序，再次反转键值对，将值大写，并使用 `code < 70` 过滤项。

如果你习惯于列表推导，那么字典推导是一个自然的下一步。如果你不熟悉，那么理解推导语法的传播意味着现在比以往任何时候都更有利可图。

## 解包映射

[PEP 448—额外的解包泛化](https://fpy.li/pep448) 自 Python 3.5 以来增强了对映射解包的支持。

首先，我们可以在函数调用中对多个参数应用 `**`。当键都是字符串且在所有参数中唯一时，这将起作用（因为禁止重复关键字参数）：

```py
>>> def dump(**kwargs):
...     return kwargs
...
>>> dump(**{'x': 1}, y=2, **{'z': 3})
{'x': 1, 'y': 2, 'z': 3}
```

第二，`**` 可以在 `dict` 字面量内使用——也可以多次使用：

```py
>>> {'a': 0, **{'x': 1}, 'y': 2, **{'z': 3, 'x': 4}}
{'a': 0, 'x': 4, 'y': 2, 'z': 3}
```

在这种情况下，允许重复的键。后续出现的键会覆盖先前的键—请参见示例中映射到 `x` 的值。

这种语法也可以用于合并映射，但还有其他方法。请继续阅读。

## 使用 | 合并映射

Python 3.9 支持使用 `|` 和 `|=` 来合并映射。这是有道理的，因为这些也是集合的并运算符。

`|` 运算符创建一个新的映射：

```py
>>> d1 = {'a': 1, 'b': 3}
>>> d2 = {'a': 2, 'b': 4, 'c': 6}
>>> d1 | d2
{'a': 2, 'b': 4, 'c': 6}
```

通常，新映射的类型将与左操作数的类型相同—在示例中是 `d1`，但如果涉及用户定义的类型，则可以是第二个操作数的类型，根据我们在第十六章中探讨的运算符重载规则。

要就地更新现有映射，请使用 `|=`。继续前面的例子，`d1` 没有改变，但现在它被改变了：

```py
>>> d1
{'a': 1, 'b': 3}
>>> d1 |= d2
>>> d1
{'a': 2, 'b': 4, 'c': 6}
```

###### 提示

如果你需要维护能在 Python 3.8 或更早版本上运行的代码，[PEP 584—为 dict 添加 Union 运算符](https://fpy.li/pep584) 的 [“动机”](https://fpy.li/3-1) 部分提供了其他合并映射的方法的简要总结。

现在让我们看看模式匹配如何应用于映射。

# 使用映射进行模式匹配

`match/case` 语句支持作为映射对象的主题。映射的模式看起来像 `dict` 字面量，但它们可以匹配 `collections.abc.Mapping` 的任何实际或虚拟子类的实例。¹

在第二章中，我们只关注了序列模式，但不同类型的模式可以组合和嵌套。由于解构，模式匹配是处理结构化为嵌套映射和序列的记录的强大工具，我们经常需要从 JSON API 和具有半结构化模式的数据库（如 MongoDB、EdgeDB 或 PostgreSQL）中读取这些记录。示例 3-2 演示了这一点。`get_creators` 中的简单类型提示清楚地表明它接受一个 `dict` 并返回一个 `list`。

##### 示例 3-2\. creator.py：`get_creators()` 从媒体记录中提取创作者的名称

```py
def get_creators(record: dict) -> list:
    match record:
        case {'type': 'book', 'api': 2, 'authors': [*names]}:  # ①
            return names
        case {'type': 'book', 'api': 1, 'author': name}:  # ②
            return [name]
        case {'type': 'book'}:  # ③
            raise ValueError(f"Invalid 'book' record: {record!r}")
        case {'type': 'movie', 'director': name}:  # ④
            return [name]
        case _:  # ⑤
            raise ValueError(f'Invalid record: {record!r}')
```

①

匹配任何具有 `'type': 'book', 'api' :2` 的映射，并且一个 `'authors'` 键映射到一个序列。将序列中的项作为新的 `list` 返回。

②

匹配任何具有 `'type': 'book', 'api' :1` 的映射，并且一个 `'author'` 键映射到任何对象。将对象放入一个 `list` 中返回。

③

具有`'type': 'book'`的任何其他映射都是无效的，引发`ValueError`。

④

匹配任何具有`'type': 'movie'`和将`'director'`键映射到单个对象的映射。返回`list`中的对象。

⑤

任何其他主题都是无效的，引发`ValueError`。

示例 3-2 展示了处理半结构化数据（如 JSON 记录）的一些有用实践：

+   包括描述记录类型的字段（例如，`'type': 'movie'`）

+   包括标识模式版本的字段（例如，`'api': 2'）以允许公共 API 的未来演变

+   有`case`子句来处理特定类型（例如，`'book'`）的无效记录，以及一个全捕捉

现在让我们看看`get_creators`如何处理一些具体的 doctests：

```py
>>> b1 = dict(api=1, author='Douglas Hofstadter',
...         type='book', title='Gödel, Escher, Bach')
>>> get_creators(b1)
['Douglas Hofstadter']
>>> from collections import OrderedDict
>>> b2 = OrderedDict(api=2, type='book',
...         title='Python in a Nutshell',
...         authors='Martelli Ravenscroft Holden'.split())
>>> get_creators(b2)
['Martelli', 'Ravenscroft', 'Holden']
>>> get_creators({'type': 'book', 'pages': 770})
Traceback (most recent call last):
    ...
ValueError: Invalid 'book' record: {'type': 'book', 'pages': 770}
>>> get_creators('Spam, spam, spam')
Traceback (most recent call last):
    ...
ValueError: Invalid record: 'Spam, spam, spam'
```

注意，模式中键的顺序无关紧要，即使主题是`OrderedDict`，如`b2`。

与序列模式相比，映射模式在部分匹配上成功。在 doctests 中，`b1`和`b2`主题包括一个在任何`'book'`模式中都不出现的`'title'`键，但它们匹配。

不需要使用`**extra`来匹配额外的键值对，但如果要将它们捕获为`dict`，可以使用`**`前缀一个变量。它必须是模式中的最后一个，并且`**_`是被禁止的，因为它是多余的。一个简单的例子：

```py
>>> food = dict(category='ice cream', flavor='vanilla', cost=199)
>>> match food:
...     case {'category': 'ice cream', **details}:
...         print(f'Ice cream details: {details}')
...
Ice cream details: {'flavor': 'vanilla', 'cost': 199}
```

在“缺失键的自动处理”中，我们将研究`defaultdict`和其他映射，其中通过`__getitem__`（即，`d[key]`）进行键查找成功，因为缺失项会动态创建。在模式匹配的上下文中，只有在主题已经具有`match`语句顶部所需键时，匹配才成功。

###### 提示

不会触发缺失键的自动处理，因为模式匹配总是使用`d.get(key, sentinel)`方法——其中默认的`sentinel`是一个特殊的标记值，不能出现在用户数据中。

从语法和结构转向，让我们研究映射的 API。

# 映射类型的标准 API

`collections.abc`模块提供了描述`dict`和类似类型接口的`Mapping`和`MutableMapping` ABCs。参见图 3-1。

ABCs 的主要价值在于记录和规范映射的标准接口，并作为需要支持广义映射的代码中`isinstance`测试的标准：

```py
>>> my_dict = {}
>>> isinstance(my_dict, abc.Mapping)
True
>>> isinstance(my_dict, abc.MutableMapping)
True
```

###### 提示

使用 ABC 进行`isinstance`通常比检查函数参数是否为具体`dict`类型更好，因为这样可以使用替代映射类型。我们将在第十三章中详细讨论这个问题。

![`Mapping`和`MutableMapping`的 UML 类图`](img/flpy_0301.png)

###### 图 3-1。`collections.abc`中`MutableMapping`及其超类的简化 UML 类图（继承箭头从子类指向超类；斜体名称是抽象类和抽象方法）。

要实现自定义映射，最好扩展`collections.UserDict`，或通过组合包装`dict`，而不是继承这些 ABCs。`collections.UserDict`类和标准库中的所有具体映射类在其实现中封装了基本的`dict`，而`dict`又建立在哈希表上。因此，它们都共享一个限制，即键必须是*可哈希*的（值不需要是可哈希的，只有键需要是可哈希的）。如果需要复习，下一节会解释。

## 什么是可哈希的

这里是从[Python *术语表*](https://fpy.li/3-3)中适应的可哈希定义的部分：

> 如果对象具有永远不会在其生命周期内更改的哈希码（它需要一个`__hash__()`方法），并且可以与其他对象进行比较（它需要一个`__eq__()`方法），则该对象是可哈希的。比较相等的可哈希对象必须具有相同的哈希码。²

数值类型和扁平不可变类型`str`和`bytes`都是可哈希的。如果容器类型是不可变的，并且所有包含的对象也是可哈希的，则它们是可哈希的。`frozenset`始终是可哈希的，因为它包含的每个元素必须根据定义是可哈希的。仅当元组的所有项都是可哈希的时，元组才是可哈希的。参见元组`tt`、`tl`和`tf`：

```py
>>> tt = (1, 2, (30, 40))
>>> hash(tt)
8027212646858338501
>>> tl = (1, 2, [30, 40])
>>> hash(tl)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'list'
>>> tf = (1, 2, frozenset([30, 40]))
>>> hash(tf)
-4118419923444501110
```

对象的哈希码可能因 Python 版本、机器架构以及出于安全原因添加到哈希计算中的*盐*而有所不同。³ 正确实现的对象的哈希码仅在一个 Python 进程中保证是恒定的。

默认情况下，用户定义的类型是可哈希的，因为它们的哈希码是它们的`id()`，并且从`object`类继承的`__eq__()`方法只是简单地比较对象 ID。如果一个对象实现了一个考虑其内部状态的自定义`__eq__()`，那么只有当其`__hash__()`始终返回相同的哈希码时，它才是可哈希的。实际上，这要求`__eq__()`和`__hash__()`只考虑在对象生命周期中永远不会改变的实例属性。

现在让我们回顾 Python 中最常用的映射类型`dict`、`defaultdict`和`OrderedDict`的 API。

## 常见映射方法概述

映射的基本 API 非常丰富。表 3-1 显示了`dict`和两个流行变体：`defaultdict`和`OrderedDict`的方法，它们都定义在`collections`模块中。

表 3-1\. 映射类型`dict`、`collections.defaultdict`和`collections.OrderedDict`的方法（为简洁起见省略了常见对象方法）；可选参数用`[…]`括起来

|  | dict | defaultdict | OrderedDict |   |
| --- | --- | --- | --- | --- |
| `d.clear()` | ● | ● | ● | 移除所有项 |
| `d.__contains__(k)` | ● | ● | ● | `k in d` |
| `d.copy()` | ● | ● | ● | 浅拷贝 |
| `d.__copy__()` |  | ● |  | 支持`copy.copy(d)` |
| `d.default_factory` |  | ● |  | `__missing__`调用的可调用对象，用于设置缺失值^(a) |
| `d.__delitem__(k)` | ● | ● | ● | `del d[k]`—删除键为`k`的项 |
| `d.fromkeys(it, [initial])` | ● | ● | ● | 从可迭代对象中的键创建新映射，可选初始值（默认为`None`） |
| `d.get(k, [default])` | ● | ● | ● | 获取键为`k`的项，如果不存在则返回`default`或`None` |
| `d.__getitem__(k)` | ● | ● | ● | `d[k]`—获取键为`k`的项 |
| `d.items()` | ● | ● | ● | 获取项的*视图*—`(key, value)`对 |
| `d.__iter__()` | ● | ● | ● | 获取键的迭代器 |
| `d.keys()` | ● | ● | ● | 获取键的*视图* |
| `d.__len__()` | ● | ● | ● | `len(d)`—项数 |
| `d.__missing__(k)` |  | ● |  | 当`__getitem__`找不到键时调用 |
| `d.move_to_end(k, [last])` |  |  | ● | 将`k`移动到第一个或最后一个位置（默认情况下`last`为`True`） |
| `d.__or__(other)` | ● | ● | ● | 支持`d1 &#124; d2`创建新的`dict`合并`d1`和`d2`（Python ≥ 3.9） |
| `d.__ior__(other)` | ● | ● | ● | 支持`d1 &#124;= d2`更新`d1`与`d2`（Python ≥ 3.9） |
| `d.pop(k, [default])` | ● | ● | ● | 移除并返回键为`k`的值，如果不存在则返回`default`或`None` |
| `d.popitem()` | ● | ● | ● | 移除并返回最后插入的项为`(key, value)` ^(b) |
| `d.__reversed__()` | ● | ● | ● | 支持`reverse(d)`—返回从最后插入到第一个插入的键的迭代器 |
| `d.__ror__(other)` | ● | ● | ● | 支持`other &#124; dd`—反向联合运算符（Python ≥ 3.9）^(c) |
| `d.setdefault(k, [default])` | ● | ● | ● | 如果`k`在`d`中，则返回`d[k]`；否则设置`d[k] = default`并返回 |
| `d.__setitem__(k, v)` | ● | ● | ● | `d[k] = v`—在`k`处放置`v` |
| `d.update(m, [**kwargs])` | ● | ● | ● | 使用映射或`(key, value)`对的可迭代对象更新`d` |
| `d.values()` | ● | ● | ● | 获取*视图*的值 |
| ^(a) `default_factory` 不是一个方法，而是在实例化`defaultdict`时由最终用户设置的可调用属性。^(b) `OrderedDict.popitem(last=False)` 移除第一个插入的项目（FIFO）。`last`关键字参数在 Python 3.10b3 中不支持`dict`或`defaultdict`。^(c) 反向运算符在第十六章中有解释。 |

`d.update(m)` 处理其第一个参数`m`的方式是*鸭子类型*的一个典型例子：它首先检查`m`是否有一个`keys`方法，如果有，就假定它是一个映射。否则，`update()`会回退到迭代`m`，假设其项是`(key, value)`对。大多数 Python 映射的构造函数在内部使用`update()`的逻辑，这意味着它们可以从其他映射或从产生`(key, value)`对的任何可迭代对象初始化。

一种微妙的映射方法是`setdefault()`。当我们需要就地更新项目的值时，它避免了冗余的键查找。下一节将展示如何使用它。

## 插入或更新可变值

符合 Python 的*失败快速*哲学，使用`d[k]`访问`dict`时，当`k`不是现有键时会引发错误。Python 程序员知道，当默认值比处理`KeyError`更方便时，`d.get(k, default)`是`d[k]`的替代方案。然而，当您检索可变值并希望更新它时，有一种更好的方法。

考虑编写一个脚本来索引文本，生成一个映射，其中每个键是一个单词，值是该单词出现的位置列表，如示例 3-3 所示。

##### 示例 3-3\. 示例 3-4 处理“Python 之禅”时的部分输出；每行显示一个单词和一对出现的编码为（`行号`，`列号`）的列表。

```py
$ python3 index0.py zen.txt
a [(19, 48), (20, 53)]
Although [(11, 1), (16, 1), (18, 1)]
ambiguity [(14, 16)]
and [(15, 23)]
are [(21, 12)]
aren [(10, 15)]
at [(16, 38)]
bad [(19, 50)]
be [(15, 14), (16, 27), (20, 50)]
beats [(11, 23)]
Beautiful [(3, 1)]
better [(3, 14), (4, 13), (5, 11), (6, 12), (7, 9), (8, 11), (17, 8), (18, 25)]
...
```

示例 3-4 是一个次优脚本，用于展示`dict.get`不是处理缺失键的最佳方式的一个案例。我从亚历克斯·马特利的一个示例中进行了改编。⁴

##### 示例 3-4\. index0.py 使用`dict.get`从索引中获取并更新单词出现列表的脚本（更好的解决方案在示例 3-5 中）

```py
"""Build an index mapping word -> list of occurrences"""

import re
import sys

WORD_RE = re.compile(r'\w+')

index = {}
with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start() + 1
            location = (line_no, column_no)
            # this is ugly; coded like this to make a point
            occurrences = index.get(word, [])  # ①
            occurrences.append(location)       # ②
            index[word] = occurrences          # ③

# display in alphabetical order
for word in sorted(index, key=str.upper):      # ④
    print(word, index[word])
```

①

获取`word`的出现列表，如果找不到则为`[]`。

②

将新位置附加到`occurrences`。

③

将更改后的`occurrences`放入`index`字典中；这需要通过`index`进行第二次搜索。

④

在`sorted`的`key=`参数中，我没有调用`str.upper`，只是传递了对该方法的引用，以便`sorted`函数可以使用它来对单词进行规范化排序。⁵

示例 3-4 中处理`occurrences`的三行可以用`dict.setdefault`替换为一行。示例 3-5 更接近亚历克斯·马特利的代码。

##### 示例 3-5\. index.py 使用`dict.setdefault`从索引中获取并更新单词出现列表的脚本，一行搞定；与示例 3-4 进行对比

```py
"""Build an index mapping word -> list of occurrences"""

import re
import sys

WORD_RE = re.compile(r'\w+')

index = {}
with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start() + 1
            location = (line_no, column_no)
            index.setdefault(word, []).append(location)  # ①

# display in alphabetical order
for word in sorted(index, key=str.upper):
    print(word, index[word])
```

①

获取`word`的出现列表，如果找不到则将其设置为`[]`；`setdefault`返回值，因此可以在不需要第二次搜索的情况下进行更新。

换句话说，这行的最终结果是…

```py
my_dict.setdefault(key, []).append(new_value)
```

…等同于运行…

```py
if key not in my_dict:
    my_dict[key] = []
my_dict[key].append(new_value)
```

…除了后者的代码至少执行两次对`key`的搜索—如果找不到，则执行三次—而`setdefault`只需一次查找就可以完成所有操作。

一个相关问题是，在任何查找中处理缺失键（而不仅仅是在插入时）是下一节的主题。

# 缺失键的自动处理

有时，当搜索缺失的键时返回一些虚构的值是很方便的。有两种主要方法：一种是使用`defaultdict`而不是普通的`dict`。另一种是子类化`dict`或任何其他映射类型，并添加一个`__missing__`方法。接下来将介绍这两种解决方案。

## defaultdict：另一种处理缺失键的方法

一个`collections.defaultdict`实例在使用`d[k]`语法搜索缺失键时按需创建具有默认值的项目。示例 3-6 使用`defaultdict`提供了另一个优雅的解决方案来完成来自示例 3-5 的单词索引任务。

它的工作原理是：在实例化`defaultdict`时，你提供一个可调用对象，每当`__getitem__`传递一个不存在的键参数时产生一个默认值。

例如，给定一个创建为`dd = defaultdict(list)`的`defaultdict`，如果`'new-key'`不在`dd`中，表达式`dd['new-key']`会执行以下步骤：

1.  调用`list()`来创建一个新列表。

1.  使用`'new-key'`作为键将列表插入`dd`。

1.  返回对该列表的引用。

产生默认值的可调用对象保存在名为`default_factory`的实例属性中。

##### 示例 3-6。index_default.py：使用`defaultdict`而不是`setdefault`方法

```py
"""Build an index mapping word -> list of occurrences"""

import collections
import re
import sys

WORD_RE = re.compile(r'\w+')

index = collections.defaultdict(list)     # ①
with open(sys.argv[1], encoding='utf-8') as fp:
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start() + 1
            location = (line_no, column_no)
            index[word].append(location)  # ②

# display in alphabetical order
for word in sorted(index, key=str.upper):
    print(word, index[word])
```

①

使用`list`构造函数创建一个`defaultdict`作为`default_factory`。

②

如果`word`最初不在`index`中，则调用`default_factory`来生成缺失值，这种情况下是一个空的`list`，然后将其分配给`index[word]`并返回，因此`.append(location)`操作总是成功的。

如果没有提供`default_factory`，则对于缺失的键会引发通常的`KeyError`。

###### 警告

`defaultdict`的`default_factory`仅在为`__getitem__`调用提供默认值时才会被调用，而不会为其他方法调用。例如，如果`dd`是一个`defaultdict`，`k`是一个缺失的键，`dd[k]`将调用`default_factory`来创建一个默认值，但`dd.get(k)`仍然返回`None`，`k in dd`为`False`。

使`defaultdict`工作的机制是调用`default_factory`的`__missing__`特殊方法，这是我们接下来要讨论的一个特性。

## `__missing__`方法

映射处理缺失键的基础是名为`__missing__`的方法。这个方法在基本的`dict`类中没有定义，但`dict`知道它：如果你子类化`dict`并提供一个`__missing__`方法，标准的`dict.__getitem__`将在找不到键时调用它，而不是引发`KeyError`。

假设你想要一个映射，其中键在查找时被转换为`str`。一个具体的用例是物联网设备库，其中一个具有通用 I/O 引脚（例如树莓派或 Arduino）的可编程板被表示为一个`Board`类，具有一个`my_board.pins`属性，它是物理引脚标识符到引脚软件对象的映射。物理引脚标识符可能只是一个数字或一个字符串，如`"A0"`或`"P9_12"`。为了一致性，希望`board.pins`中的所有键都是字符串，但也方便通过数字查找引脚，例如`my_arduino.pin[13]`，这样初学者在想要闪烁他们的 Arduino 上的 13 号引脚时不会出错。示例 3-7 展示了这样一个映射如何工作。

##### 示例 3-7。当搜索非字符串键时，`StrKeyDict0`在未找到时将其转换为`str`

```py
Tests for item retrieval using `d[key]` notation::

    >>> d = StrKeyDict0([('2', 'two'), ('4', 'four')])
    >>> d['2']
    'two'
    >>> d[4]
    'four'
    >>> d[1]
    Traceback (most recent call last):
      ...
    KeyError: '1'

Tests for item retrieval using `d.get(key)` notation::

    >>> d.get('2')
    'two'
    >>> d.get(4)
    'four'
    >>> d.get(1, 'N/A')
    'N/A'

Tests for the `in` operator::

    >>> 2 in d
    True
    >>> 1 in d
    False
```

示例 3-8 实现了一个通过前面的 doctests 的`StrKeyDict0`类。

###### 提示

创建用户定义的映射类型的更好方法是子类化`collections.UserDict`而不是`dict`（正如我们将在示例 3-9 中所做的那样）。这里我们子类化`dict`只是为了展示内置的`dict.__getitem__`方法支持`__missing__`。

##### 示例 3-8。`StrKeyDict0`在查找时将非字符串键转换为`str`（请参见示例 3-7 中的测试）

```py
class StrKeyDict0(dict):  # ①

    def __missing__(self, key):
        if isinstance(key, str):  # ②
            raise KeyError(key)
        return self[str(key)]  # ③

    def get(self, key, default=None):
        try:
            return self[key]  # ④
        except KeyError:
            return default  # ⑤

    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()  # ⑥
```

①

`StrKeyDict0`继承自`dict`。

②

检查`key`是否已经是`str`。如果是，并且它丢失了，那么引发`KeyError`。

③

从`key`构建`str`并查找它。

④

`get`方法通过使用`self[key]`符号委托给`__getitem__`；这给了我们的`__missing__`发挥作用的机会。

⑤

如果引发`KeyError`，则`__missing__`已经失败，因此我们返回`default`。

⑥

搜索未修改的键（实例可能包含非`str`键），然后搜索从键构建的`str`。

花点时间考虑一下为什么在`__missing__`实现中需要测试`isinstance(key, str)`。

没有这个测试，我们的`__missing__`方法对于任何键`k`——`str`或非`str`——都能正常工作，只要`str(k)`产生一个现有的键。但是如果`str(k)`不是一个现有的键，我们将会有一个无限递归。在`__missing__`的最后一行，`self[str(key)]`会调用`__getitem__`，传递那个`str`键，然后会再次调用`__missing__`。

在这个例子中，`__contains__`方法也是必需的，因为操作`k in d`会调用它，但从`dict`继承的方法不会回退到调用`__missing__`。在我们的`__contains__`实现中有一个微妙的细节：我们不是用通常的 Python 方式检查键——`k` in `my_dict`——因为`str(key) in self`会递归调用`__contains__`。我们通过在`self.keys()`中明确查找键来避免这种情况。

在 Python 3 中，像`k in my_dict.keys()`这样的搜索对于非常大的映射也是高效的，因为`dict.keys()`返回一个视图，类似于集合，正如我们将在“dict 视图上的集合操作”中看到的。然而，请记住，`k in my_dict`也能完成同样的工作，并且更快，因为它避免了查找属性以找到`.keys`方法。

我在示例 3-8 中的`__contains__`方法中有一个特定的原因使用`self.keys()`。检查未修改的键——`key in self.keys()`——对于正确性是必要的，因为`StrKeyDict0`不强制字典中的所有键都必须是`str`类型。我们这个简单示例的唯一目标是使搜索“更友好”，而不是强制类型。

###### 警告

派生自标准库映射的用户定义类可能会或可能不会在它们的`__getitem__`、`get`或`__contains__`实现中使用`__missing__`作为回退，如下一节所述。

## 标准库中对`__missing__`的不一致使用

考虑以下情况，以及缺失键查找是如何受影响的：

`dict`子类

一个只实现`__missing__`而没有其他方法的`dict`子类。在这种情况下，`__missing__`只能在`d[k]`上调用，这将使用从`dict`继承的`__getitem__`。

`collections.UserDict`子类

同样，一个只实现`__missing__`而没有其他方法的`UserDict`子类。从`UserDict`继承的`get`方法调用`__getitem__`。这意味着`__missing__`可能被调用来处理`d[k]`和`d.get(k)`的查找。

具有最简单可能的`__getitem__`的`abc.Mapping`子类

一个实现了`__missing__`和所需抽象方法的最小的`abc.Mapping`子类，包括一个不调用`__missing__`的`__getitem__`实现。在这个类中，`__missing__`方法永远不会被触发。

具有调用`__missing__`的`__getitem__`的`abc.Mapping`子类

一个最小的`abc.Mapping`子类实现了`__missing__`和所需的抽象方法，包括调用`__missing__`的`__getitem__`的实现。在这个类中，对使用`d[k]`、`d.get(k)`和`k in d`进行的缺失键查找会触发`__missing__`方法。

在示例代码库中查看[*missing.py*](https://fpy.li/3-7)以演示这里描述的场景。

刚才描述的四种情况假设最小实现。如果你的子类实现了`__getitem__`、`get`和`__contains__`，那么你可以根据需要让这些方法使用`__missing__`或不使用。本节的重点是要表明，在子类化标准库映射时要小心使用`__missing__`，因为基类默认支持不同的行为。

不要忘记，`setdefault`和`update`的行为也受键查找影响。最后，根据你的`__missing__`的逻辑，你可能需要在`__setitem__`中实现特殊逻辑，以避免不一致或令人惊讶的行为。我们将在“Subclassing UserDict Instead of dict”中看到一个例子。

到目前为止，我们已经介绍了`dict`和`defaultdict`这两种映射类型，但标准库中还有其他映射实现，接下来我们将讨论它们。

# dict 的变体

本节概述了标准库中包含的映射类型，除了已在“defaultdict: Another Take on Missing Keys”中介绍的`defaultdict`。

## collections.OrderedDict

自从 Python 3.6 开始，内置的`dict`也保持了键的有序性，使用`OrderedDict`的最常见原因是编写与早期 Python 版本向后兼容的代码。话虽如此，Python 的文档列出了`dict`和`OrderedDict`之间的一些剩余差异，我在这里引用一下——只重新排列项目以便日常使用：

+   `OrderedDict`的相等操作检查匹配的顺序。

+   `OrderedDict`的`popitem()`方法具有不同的签名。它接受一个可选参数来指定要弹出的项目。

+   `OrderedDict`有一个`move_to_end()`方法，可以高效地将一个元素重新定位到末尾。

+   常规的`dict`被设计为在映射操作方面非常出色。跟踪插入顺序是次要的。

+   `OrderedDict`被设计为在重新排序操作方面表现良好。空间效率、迭代速度和更新操作的性能是次要的。

+   从算法上讲，`OrderedDict`比`dict`更擅长处理频繁的重新排序操作。这使得它适用于跟踪最近的访问（例如，在 LRU 缓存中）。

## collections.ChainMap

`ChainMap`实例保存了一个可以作为一个整体搜索的映射列表。查找是按照构造函数调用中出现的顺序在每个输入映射上执行的，并且一旦在这些映射中的一个中找到键，查找就成功了。例如：

```py
>>> d1 = dict(a=1, b=3)
>>> d2 = dict(a=2, b=4, c=6)
>>> from collections import ChainMap
>>> chain = ChainMap(d1, d2)
>>> chain['a']
1
>>> chain['c']
6
```

`ChainMap`实例不会复制输入映射，而是保留对它们的引用。对`ChainMap`的更新或插入只会影响第一个输入映射。继续上一个例子：

```py
>>> chain['c'] = -1
>>> d1
{'a': 1, 'b': 3, 'c': -1}
>>> d2
{'a': 2, 'b': 4, 'c': 6}
```

`ChainMap`对于实现具有嵌套作用域的语言的解释器非常有用，其中每个映射表示一个作用域上下文，从最内部的封闭作用域到最外部作用域。[`collections`文档中的“ChainMap objects”部分](https://fpy.li/3-8)有几个`ChainMap`使用示例，包括这个受 Python 变量查找基本规则启发的代码片段：

```py
import builtins
pylookup = ChainMap(locals(), globals(), vars(builtins))
```

示例 18-14 展示了一个用于实现 Scheme 编程语言子集解释器的`ChainMap`子类。

## collections.Counter

一个为每个键保存整数计数的映射。更新现有键会增加其计数。这可用于计算可散列对象的实例数量或作为多重集（稍后在本节讨论）。`Counter` 实现了 `+` 和 `-` 运算符来组合计数，并提供其他有用的方法，如 `most_common([n])`，它返回一个按顺序排列的元组列表，其中包含 *n* 个最常见的项目及其计数；请参阅[文档](https://fpy.li/3-9)。这里是 `Counter` 用于计算单词中的字母：

```py
>>> ct = collections.Counter('abracadabra')
>>> ct
Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
>>> ct.update('aaaaazzz')
>>> ct
Counter({'a': 10, 'z': 3, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
>>> ct.most_common(3)
[('a', 10), ('z', 3), ('b', 2)]
```

请注意，`'b'` 和 `'r'` 键并列第三，但 `ct.most_common(3)` 只显示了三个计数。

要将 `collections.Counter` 用作多重集，假装每个键是集合中的一个元素，计数是该元素在集合中出现的次数。

## shelve.Shelf

标准库中的 `shelve` 模块为字符串键到以 `pickle` 二进制格式序列化的 Python 对象的映射提供了持久存储。当你意识到 pickle 罐子存放在架子上时，`shelve` 这个奇怪的名字就有了意义。

`shelve.open` 模块级函数返回一个 `shelve.Shelf` 实例——一个简单的键-值 DBM 数据库，由 `dbm` 模块支持，具有以下特点：

+   `shelve.Shelf` 是 `abc.MutableMapping` 的子类，因此它提供了我们期望的映射类型的基本方法。

+   此外，`shelve.Shelf` 提供了一些其他的 I/O 管理方法，如 `sync` 和 `close`。

+   `Shelf` 实例是一个上下文管理器，因此您可以使用 `with` 块来确保在使用后关闭它。

+   每当将新值分配给键时，键和值都会被保存。

+   键必须是字符串。

+   值必须是 `pickle` 模块可以序列化的对象。

[shelve](https://fpy.li/3-10)、[dbm](https://fpy.li/3-11) 和 [pickle](https://fpy.li/3-12) 模块的文档提供了更多细节和一些注意事项。

###### 警告

Python 的 `pickle` 在最简单的情况下很容易使用，但也有一些缺点。在采用涉及 `pickle` 的任何解决方案之前，请阅读 Ned Batchelder 的[“Pickle 的九个缺陷”](https://fpy.li/3-13)。在他的帖子中，Ned 提到了其他要考虑的序列化格式。

`OrderedDict`、`ChainMap`、`Counter` 和 `Shelf` 都可以直接使用，但也可以通过子类化进行自定义。相比之下，`UserDict` 只是作为一个可扩展的基类。

## 通过继承 `UserDict` 而不是 `dict` 来创建新的映射类型

最好通过扩展 `collections.UserDict` 来创建新的映射类型，而不是 `dict`。当我们尝试扩展我们的 `StrKeyDict0`（来自示例 3-8）以确保将任何添加到映射中的键存储为 `str` 时，我们意识到这一点。

更好地通过子类化 `UserDict` 而不是 `dict` 的主要原因是，内置类型有一些实现快捷方式，最终迫使我们覆盖我们可以从 `UserDict` 继承而不会出现问题的方法。⁷

请注意，`UserDict` 不继承自 `dict`，而是使用组合：它有一个内部的 `dict` 实例，称为 `data`，用于保存实际的项目。这避免了在编写特殊方法如 `__setitem__` 时出现不必要的递归，并简化了 `__contains__` 的编写，与示例 3-8 相比更加简单。

由于 `UserDict` 的存在，`StrKeyDict`（示例 3-9）比 `StrKeyDict0`（示例 3-8）更简洁，但它做得更多：它将所有键都存储为 `str`，避免了如果实例被构建或更新时包含非字符串键时可能出现的令人不快的情况。

##### 示例 3-9\. `StrKeyDict` 在插入、更新和查找时总是将非字符串键转换为 `str`。

```py
import collections

class StrKeyDict(collections.UserDict):  # ①

    def __missing__(self, key):  # ②
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]

    def __contains__(self, key):
        return str(key) in self.data  # ③

    def __setitem__(self, key, item):
        self.data[str(key)] = item   # ④
```

①

`StrKeyDict` 扩展了 `UserDict`。

②

`__missing__` 与示例 3-8 中的一样。

③

`__contains__` 更简单：我们可以假定所有存储的键都是 `str`，并且可以在 `self.data` 上进行检查，而不是像在 `StrKeyDict0` 中那样调用 `self.keys()`。

④

`__setitem__` 将任何 `key` 转换为 `str`。当我们可以委托给 `self.data` 属性时，这种方法更容易被覆盖。

因为 `UserDict` 扩展了 `abc.MutableMapping`，使得使 `StrKeyDict` 成为一个完整的映射的剩余方法都是从 `UserDict`、`MutableMapping` 或 `Mapping` 继承的。尽管后者是抽象基类（ABC），但它们有几个有用的具体方法。以下方法值得注意：

`MutableMapping.update`

这种强大的方法可以直接调用，但也被 `__init__` 用于从其他映射、从 `(key, value)` 对的可迭代对象和关键字参数加载实例。因为它使用 `self[key] = value` 来添加项目，所以最终会调用我们的 `__setitem__` 实现。

`Mapping.get`

在 `StrKeyDict0`（示例 3-8）中，我们不得不编写自己的 `get` 来返回与 `__getitem__` 相同的结果，但在 示例 3-9 中，我们继承了 `Mapping.get`，它的实现与 `StrKeyDict0.get` 完全相同（请参阅 [Python 源代码](https://fpy.li/3-14)）。

###### 提示

安托万·皮特鲁（Antoine Pitrou）撰写了 [PEP 455—向 collections 添加一个键转换字典](https://fpy.li/pep455) 和一个增强 `collections` 模块的补丁，其中包括一个 `TransformDict`，比 `StrKeyDict` 更通用，并保留提供的键，然后应用转换。PEP 455 在 2015 年 5 月被拒绝—请参阅雷蒙德·赫廷格的 [拒绝消息](https://fpy.li/3-15)。为了尝试 `TransformDict`，我从 [issue18986](https://fpy.li/3-16) 中提取了皮特鲁的补丁，制作成了一个独立的模块（[*03-dict-set/transformdict.py*](https://fpy.li/3-17) 在 [*Fluent Python* 第二版代码库](https://fpy.li/code) 中）。

我们知道有不可变的序列类型，但不可变的映射呢？在标准库中确实没有真正的不可变映射，但有一个替代品可用。接下来是。

# 不可变映射

标准库提供的映射类型都是可变的，但您可能需要防止用户意外更改映射。再次在硬件编程库中找到一个具体的用例，比如 *Pingo*，在 “缺失方法” 中提到：`board.pins` 映射表示设备上的物理 GPIO 引脚。因此，防止意外更新 `board.pins` 是有用的，因为硬件不能通过软件更改，所以映射的任何更改都会使其与设备的物理现实不一致。

`types` 模块提供了一个名为 `MappingProxyType` 的包装类，给定一个映射，它返回一个 `mappingproxy` 实例，这是原始映射的只读但动态代理。这意味着可以在 `mappingproxy` 中看到对原始映射的更新，但不能通过它进行更改。参见 示例 3-10 进行简要演示。

##### 示例 3-10\. `MappingProxyType` 从 `dict` 构建一个只读的 `mappingproxy` 实例。

```py
>>> from types import MappingProxyType
>>> d = {1: 'A'}
>>> d_proxy = MappingProxyType(d)
>>> d_proxy
mappingproxy({1: 'A'}) >>> d_proxy[1]  # ①
'A' >>> d_proxy[2] = 'x'  # ②
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: 'mappingproxy' object does not support item assignment
>>> d[2] = 'B'
>>> d_proxy  # ③
mappingproxy({1: 'A', 2: 'B'}) >>> d_proxy[2]
'B' >>>
```

①

`d` 中的项目可以通过 `d_proxy` 看到。

②

不能通过 `d_proxy` 进行更改。

③

`d_proxy` 是动态的：`d` 中的任何更改都会反映出来。

在硬件编程场景中，这个方法在实践中可以这样使用：具体的 `Board` 子类中的构造函数会用 pin 对象填充一个私有映射，并通过一个实现为 `mappingproxy` 的公共 `.pins` 属性将其暴露给 API 的客户端。这样，客户端就无法意外地添加、删除或更改 pin。

接下来，我们将介绍视图—它允许在 `dict` 上进行高性能操作，而无需不必要地复制数据。

# 字典视图

`dict`实例方法`.keys()`、`.values()`和`.items()`返回类`dict_keys`、`dict_values`和`dict_items`的实例，分别。这些字典视图是`dict`实现中使用的内部数据结构的只读投影。它们避免了等效 Python 2 方法的内存开销，这些方法返回了重复数据的列表，这些数据已经在目标`dict`中，它们还替换了返回迭代器的旧方法。

示例 3-11 展示了所有字典视图支持的一些基本操作。

##### 示例 3-11。`.values()`方法返回字典中值的视图

```py
>>> d = dict(a=10, b=20, c=30)
>>> values = d.values()
>>> values
dict_values([10, 20, 30]) # ①
>>> len(values)  # ②
3 >>> list(values)  # ③
[10, 20, 30] >>> reversed(values)  # ④
<dict_reversevalueiterator object at 0x10e9e7310> >>> values[0] # ⑤
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: 'dict_values' object is not subscriptable
```

①

视图对象的`repr`显示其内容。

②

我们可以查询视图的`len`。

③

视图是可迭代的，因此很容易从中创建列表。

④

视图实现了`__reversed__`，返回一个自定义迭代器。

⑤

我们不能使用`[]`从视图中获取单个项目。

视图对象是动态代理。如果源`dict`被更新，您可以立即通过现有视图看到更改。继续自示例 3-11：

```py
>>> d['z'] = 99
>>> d
{'a': 10, 'b': 20, 'c': 30, 'z': 99}
>>> values
dict_values([10, 20, 30, 99])
```

类`dict_keys`、`dict_values`和`dict_items`是内部的：它们不通过`__builtins__`或任何标准库模块可用，即使你获得了其中一个的引用，也不能在 Python 代码中从头开始创建视图：

```py
>>> values_class = type({}.values())
>>> v = values_class()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: cannot create 'dict_values' instances
```

`dict_values`类是最简单的字典视图——它只实现了`__len__`、`__iter__`和`__reversed__`特殊方法。除了这些方法，`dict_keys`和`dict_items`实现了几个集合方法，几乎和`frozenset`类一样多。在我们讨论集合之后，我们将在“字典视图上的集合操作”中更多地谈到`dict_keys`和`dict_items`。

现在让我们看一些由`dict`在幕后实现的规则和提示。

# `dict`工作方式的实际后果

Python 的`dict`的哈希表实现非常高效，但重要的是要了解这种设计的实际影响：

+   键必须是可散列的对象。它们必须实现适当的`__hash__`和`__eq__`方法，如“什么是可散列”中所述。

+   通过键访问项目非常快速。一个`dict`可能有数百万个键，但 Python 可以通过计算键的哈希码并推导出哈希表中的索引偏移量直接定位一个键，可能会有少量尝试来找到匹配的条目的开销。

+   键的顺序保留是 CPython 3.6 中`dict`更紧凑的内存布局的副作用，在 3.7 中成为官方语言特性。

+   尽管其新的紧凑布局，字典不可避免地具有显着的内存开销。对于容器来说，最紧凑的内部数据结构将是一个指向项目的指针数组。⁸ 相比之下，哈希表需要存储更多的数据，而 Python 需要保持至少三分之一的哈希表行为空以保持高效。

+   为了节省内存，避免在`__init__`方法之外创建实例属性。

最后一条关于实例属性的提示来自于 Python 的默认行为是将实例属性存储在一个特殊的`__dict__`属性中，这是一个附加到每个实例的`dict`。自从 Python 3.3 实现了[PEP 412—Key-Sharing Dictionary](https://fpy.li/pep412)以来，一个类的实例可以共享一个与类一起存储的公共哈希表。当`__init__`返回时，具有相同属性名称的每个新实例的`__dict__`都共享该公共哈希表。然后，每个实例的`__dict__`只能保存自己的属性值作为指针的简单数组。在`__init__`之后添加一个实例属性会强制 Python 为`__dict__`创建一个新的哈希表，用于该实例的`__dict__`（这是 Python 3.3 之前所有实例的默认行为）。根据 PEP 412，这种优化可以减少面向对象程序的内存使用量 10%至 20%。

紧凑布局和键共享优化的细节相当复杂。更多信息，请阅读[*fluentpython.com*](http://fluentpython.com)上的[“集合和字典的内部”](https://fpy.li/hashint)。

现在让我们深入研究集合。

# 集合理论

在 Python 中，集合并不新鲜，但仍然有些被低估。`set`类型及其不可变的姊妹`frozenset`首次出现在 Python 2.3 标准库中作为模块，并在 Python 2.6 中被提升为内置类型。

###### 注意

在本书中，我使用“集合”一词来指代`set`和`frozenset`。当专门讨论`set`类型，我使用等宽字体：`set`。

集合是一组唯一对象。一个基本用例是去除重复项：

```py
>>> l = ['spam', 'spam', 'eggs', 'spam', 'bacon', 'eggs']
>>> set(l)
{'eggs', 'spam', 'bacon'}
>>> list(set(l))
['eggs', 'spam', 'bacon']
```

###### 提示

如果你想去除重复项但又保留每个项目的第一次出现的顺序，你现在可以使用一个普通的`dict`来实现，就像这样：

```py
>>> dict.fromkeys(l).keys()
dict_keys(['spam', 'eggs', 'bacon'])
>>> list(dict.fromkeys(l).keys())
['spam', 'eggs', 'bacon']
```

集合元素必须是可散列的。`set`类型不可散列，因此你不能用嵌套的`set`实例构建一个`set`。但是`frozenset`是可散列的，所以你可以在`set`中包含`frozenset`元素。

除了强制唯一性外，集合类型还实现了许多集合操作作为中缀运算符，因此，给定两个集合`a`和`b`，`a | b`返回它们的并集，`a & b`计算交集，`a - b`表示差集，`a ^ b`表示对称差。巧妙地使用集合操作可以减少 Python 程序的行数和执行时间，同时使代码更易于阅读和理解——通过消除循环和条件逻辑。

例如，想象一下你有一个大型的电子邮件地址集合（`haystack`）和一个较小的地址集合（`needles`），你需要计算`needles`在`haystack`中出现的次数。由于集合交集（`&`运算符），你可以用一行代码实现这个功能（参见示例 3-12）。

##### 示例 3-12. 计算在一个集合中针的出现次数，两者都是集合类型

```py
found = len(needles & haystack)
```

没有交集运算符，你将不得不编写示例 3-13 来完成与示例 3-12 相同的任务。

##### 示例 3-13. 计算在一个集合中针的出现次数（与示例 3-12 的结果相同）

```py
found = 0
for n in needles:
    if n in haystack:
        found += 1
```

示例 3-12 比示例 3-13 运行速度稍快。另一方面，示例 3-13 适用于任何可迭代对象`needles`和`haystack`，而示例 3-12 要求两者都是集合。但是，如果你手头没有集合，你可以随时动态构建它们，就像示例 3-14 中所示。

##### 示例 3-14. 计算在一个集合中针的出现次数；这些行适用于任何可迭代类型

```py
found = len(set(needles) & set(haystack))

# another way:
found = len(set(needles).intersection(haystack))
```

当然，在构建示例 3-14 中的集合时会有额外的成本，但如果`needles`或`haystack`中的一个已经是一个集合，那么示例 3-14 中的替代方案可能比示例 3-13 更便宜。

任何前述示例中的一个都能在`haystack`中搜索 1,000 个元素，其中包含 10,000,000 个项目，大约需要 0.3 毫秒，即每个元素接近 0.3 微秒。

除了极快的成员测试（由底层哈希表支持），`set` 和 `frozenset` 内置类型提供了丰富的 API 来创建新集合或在`set`的情况下更改现有集合。我们将很快讨论这些操作，但首先让我们谈谈语法。

## 集合字面量

`set`字面量的语法—`{1}`，`{1, 2}`等—看起来与数学符号一样，但有一个重要的例外：没有空`set`的字面表示，因此我们必须记得写`set()`。

# 语法怪癖

不要忘记，要创建一个空的`set`，应该使用没有参数的构造函数：`set()`。如果写`{}`，你将创建一个空的`dict`—在 Python 3 中这一点没有改变。

在 Python 3 中，集合的标准字符串表示总是使用`{…}`符号，除了空集：

```py
>>> s = {1}
>>> type(s)
<class 'set'>
>>> s
{1}
>>> s.pop()
1
>>> s
set()
```

字面`set`语法如`{1, 2, 3}`比调用构造函数（例如，`set([1, 2, 3])`）更快且更易读。后一种形式较慢，因为要评估它，Python 必须查找`set`名称以获取构造函数，然后构建一个列表，最后将其传递给构造函数。相比之下，要处理像`{1, 2, 3}`这样的字面量，Python 运行一个专门的`BUILD_SET`字节码。¹⁰

没有特殊的语法来表示`frozenset`字面量—它们必须通过调用构造函数创建。在 Python 3 中的标准字符串表示看起来像一个`frozenset`构造函数调用。请注意控制台会话中的输出：

```py
>>> frozenset(range(10))
frozenset({0, 1, 2, 3, 4, 5, 6, 7, 8, 9})
```

谈到语法，列表推导的想法也被用来构建集合。

## 集合推导式

集合推导式（*setcomps*）在 Python 2.7 中添加，与我们在“dict 推导式”中看到的 dictcomps 一起。示例 3-15 展示了如何。

##### 示例 3-15\. 构建一个拉丁-1 字符集，其中 Unicode 名称中包含“SIGN”一词

```py
>>> from unicodedata import name  # ①
>>> {chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i),'')}  # ②
{'§', '=', '¢', '#', '¤', '<', '¥', 'µ', '×', '$', '¶', '£', '©', '°', '+', '÷', '±', '>', '¬', '®', '%'}
```

①

从`unicodedata`导入`name`函数以获取字符名称。

②

构建字符集，其中字符代码从 32 到 255，名称中包含 `'SIGN'` 一词。

输出的顺序会因为“什么是可哈希的”中提到的盐哈希而对每个 Python 进程进行更改。

语法问题在一边，现在让我们考虑集合的行为。

# 集合工作方式的实际后果

`set` 和 `frozenset` 类型都是使用哈希表实现的。这会产生以下影响：

+   集合元素必须是可哈希对象。它们必须实现适当的`__hash__`和`__eq__`方法，如“什么是可哈希的”中所述。

+   成员测试非常高效。一个集合可能有数百万个元素，但可以通过计算其哈希码并推导出索引偏移量来直接定位一个元素，可能需要少量尝试来找到匹配的元素或耗尽搜索。

+   与低级数组指针相比，集合具有显着的内存开销—后者更紧凑但搜索超过少量元素时也更慢。

+   元素顺序取决于插入顺序，但并不是以有用或可靠的方式。如果两个元素不同但具有相同的哈希码，则它们的位置取决于哪个元素先添加。

+   向集合添加元素可能会改变现有元素的顺序。这是因为如果哈希表超过三分之二满，算法会变得不那么高效，因此 Python 可能需要在增长时移动和调整表格。当发生这种情况时，元素将被重新插入，它们的相对顺序可能会改变。

详细信息请参见[“集合和字典的内部”](https://fpy.li/hashint)在[*fluentpython.com*](http://fluentpython.com)。

现在让我们来看看集合提供的丰富操作。

## 集合操作

|  |  | `s.difference(it, …)` | `s` 和从可迭代对象 `it` 构建的所有集合的差集 |

###### | S ⊆ Z | `s <= z` | `s.__le__(z)` | `s` 是 `z` 集合的子集 |

[费曼学习法](https://wiki.example.org/feynmans_learning_method)的灵感源于**理查德·费曼**，这位物理学诺贝尔奖得主。

![`Set` 和 `MutableSet` 的 UML 类图](img/flpy_0302.png)

###### |  |  |  |  |

| 数学符号 | Python 运算符 | 方法 | 描述 |

|  |  | `s.intersection(it, …)` | `s` 和从可迭代对象 `it` 构建的所有集合的交集 |
| --- | --- | --- | --- |
|  | `s -= z` | `s.__isub__(z)` | `s` 更新为 `s` 和 `z` 的差集 |
| S \ Z | `s - z` | `s.__sub__(z)` | `s` 和 `z` 的相对补集或差集 |
|  | `z ^ s` | `s.__rxor__(z)` | 反转 `^` 运算符 |
|  |  | `s.difference_update(it, …)` | `s` 更新为 `s` 和从可迭代对象 `it` 构建的所有集合的差集 |
|  | `s &= z` | `s.__iand__(z)` | `s` 更新为 `s` 和 `z` 的交集 |
|  |  | `s.union(it, …)` | `s` 和从可迭代对象 `it` 构建的所有集合的并集 |
| 图 3-2 概述了可变和不可变集合上可用的方法。其中许多是重载运算符的特殊方法，如 `&` 和 `>=`。表 3-2 显示了在 Python 中具有对应运算符或方法的数学集合运算符。请注意，一些运算符和方法会对目标集合进行就地更改（例如 `&=`，`difference_update` 等）。这样的操作在数学集合的理想世界中毫无意义，并且在 `frozenset` 中未实现。 |
|  |  | `s.update(it, …)` | `s` 更新为 `s` 和从可迭代对象 `it` 构建的所有集合的并集 |
|  | `z & s` | `s.__rand__(z)` | 反转 `&` 运算符 |
|  |  |  |  |
| S ∆ Z | `s ^ z` | `s.__xor__(z)` | 对称差集（`s & z` 的补集） |
| 表 3-2\. 数学集合操作：这些方法要么生成新集合，要么在原地更新目标集合（如果可变） |
|  |  |  |  |
| S ∩ Z = ∅ |  | `s.isdisjoint(z)` | `s` 和 `z` 互不相交（没有共同元素） |
|  |  | `s.symmetric_difference(it)` | `s & set(it)` 的补集 |
| S ∩ Z | `s & z` | `s.__and__(z)` | `s` 和 `z` 的交集 |
| 使用费曼的技巧，你可以在短短`20 min`内深入理解知识点，而且记忆深刻，*难以遗忘*。 |
|  | `s ^= z` | `s.__ixor__(z)` | `s` 更新为 `s` 和 `z` 的对称差集 |
| S ∪ Z | `s &#124; z` | `s.__or__(z)` | `s` 和 `z` 的并集 |
| e ∈ S | `e in s` | `s.__contains__(e)` | 元素 `e` 是 `s` 的成员 |
|  |  | `s.intersection_update(it, …)` | `s` 更新为 `s` 和从可迭代对象 `it` 构建的所有集合的交集 |
| 数学符号 | Python 运算符 | 方法 | 描述 |
| 表 3-3\. 返回布尔值的集合比较运算符和方法 |
|  |  | `s.symmetric_difference_update(it, …)` | `s` 更新为 `s` 和从可迭代对象 `it` 构建的所有集合的对称差 |

提示

|  |  |  |  |

| 表 3-3 列出了集合谓词：返回 `True` 或 `False` 的运算符和方法。 |
| --- | --- | --- | --- |
| --- | --- | --- | --- |
| 图 3-2\. `MutableSet` 及其来自 `collections.abc` 的超类的简化 UML 类图（斜体名称为抽象类和抽象方法；为简洁起见省略了反转运算符方法） |
|  | `z &#124; s` | `s.__ror__(z)` | 反转 `&#124;` 运算符 |
|  | `z - s` | `s.__rsub__(z)` | 反转 `-` 运算符 |
|  | `s &#124;= z` | `s.__ior__(z)` | `s` 更新为 `s` 和 `z` 的并集 |
|  |  | `s.issubset(it)` | `s` 是从可迭代对象 `it` 构建的集合的子集 |
|  |  |  |  |
| S ⊂ Z | `s < z` | `s.__lt__(z)` | `s` 是 `z` 集合的真子集 |
|  |  |  |  |
| S ⊇ Z | `s >= z` | `s.__ge__(z)` | `s` 是 `z` 集合的超集 |
|  |  | `s.issuperset(it)` | `s` 是从可迭代对象 `it` 构建的集合的超集 |
|  |  |  |  |
| S ⊃ Z | `s > z` | `s.__gt__(z)` | `s` 是 `z` 集合的真超集 |
|  |  |  |  |

除了从数学集合理论中派生的运算符和方法外，集合类型还实现了其他实用的方法，总结在表 3-4 中。

表 3-4\. 额外的集合方法

|  | 集合 | 冻结集合 |   |
| --- | --- | --- | --- |
| `s.add(e)` | ● |  | 向 `s` 添加元素 `e` |
| `s.clear()` | ● |  | 移除 `s` 的所有元素 |
| `s.copy()` | ● | ● | `s` 的浅复制 |
| `s.discard(e)` | ● |  | 如果存在则从 `s` 中移除元素 `e` |
| `s.__iter__()` | ● | ● | 获取 `s` 的迭代器 |
| `s.__len__()` | ● | ● | `len(s)` |
| `s.pop()` | ● |  | 从 `s` 中移除并返回一个元素，如果 `s` 为空则引发 `KeyError` |
| `s.remove(e)` | ● |  | 从 `s` 中移除元素 `e`，如果 `e` 不在 `s` 中则引发 `KeyError` |

这完成了我们对集合特性的概述。如“字典视图”中承诺的，我们现在将看到两种字典视图类型的行为非常类似于 `frozenset`。

# 字典视图上的集合操作

表 3-5 显示了由 `dict` 方法 `.keys()` 和 `.items()` 返回的视图对象与 `frozenset` 非常相似。

表 3-5\. `frozenset`、`dict_keys` 和 `dict_items` 实现的方法

|  | 冻结集合 | dict_keys | dict_items | 描述 |
| --- | --- | --- | --- | --- |
| `s.__and__(z)` | ● | ● | ● | `s & z`（`s` 和 `z` 的交集） |
| `s.__rand__(z)` | ● | ● | ● | 反转 `&` 运算符 |
| `s.__contains__()` | ● | ● | ● | `e in s` |
| `s.copy()` | ● |  |  | `s` 的浅复制 |
| `s.difference(it, …)` | ● |  |  | `s` 和可迭代对象 `it` 等的差集 |
| `s.intersection(it, …)` | ● |  |  | `s` 和可迭代对象 `it` 等的交集 |
| `s.isdisjoint(z)` | ● | ● | ● | `s` 和 `z` 不相交（没有共同元素） |
| `s.issubset(it)` | ● |  |  | `s` 是可迭代对象 `it` 的子集 |
| `s.issuperset(it)` | ● |  |  | `s` 是可迭代对象 `it` 的超集 |
| `s.__iter__()` | ● | ● | ● | 获取 `s` 的迭代器 |
| `s.__len__()` | ● | ● | ● | `len(s)` |
| `s.__or__(z)` | ● | ● | ● | `s &#124; z`（`s` 和 `z` 的并集） |
| `s.__ror__()` | ● | ● | ● | 反转 `&#124;` 运算符 |
| `s.__reversed__()` |  | ● | ● | 获取 `s` 的反向迭代器 |
| `s.__rsub__(z)` | ● | ● | ● | 反转 `-` 运算符 |
| `s.__sub__(z)` | ● | ● | ● | `s - z`（`s` 和 `z` 之间的差集） |
| `s.symmetric_difference(it)` | ● |  |  | `s & set(it)` 的补集 |
| `s.union(it, …)` | ● |  |  | `s` 和可迭代对象 `it` 等的并集 |
| `s.__xor__()` | ● | ● | ● | `s ^ z`（`s` 和 `z` 的对称差集） |
| `s.__rxor__()` | ● | ● | ● | 反转 `^` 运算符 |

特别地，`dict_keys` 和 `dict_items` 实现了支持强大的集合运算符 `&`（交集）、`|`（并集）、`-`（差集）和 `^`（对称差集）的特殊方法。

例如，使用 `&` 很容易获得出现在两个字典中的键：

```py
>>> d1 = dict(a=1, b=2, c=3, d=4)
>>> d2 = dict(b=20, d=40, e=50)
>>> d1.keys() & d2.keys()
{'b', 'd'}
```

请注意 `&` 的返回值是一个 `set`。更好的是：字典视图中的集合运算符与 `set` 实例兼容。看看这个：

```py
>>> s = {'a', 'e', 'i'}
>>> d1.keys() & s
{'a'}
>>> d1.keys() | s
{'a', 'c', 'b', 'd', 'i', 'e'}
```

###### 警告

一个 `dict_items` 视图仅在字典中的所有值都是可哈希的情况下才能作为集合使用。尝试在具有不可哈希值的 `dict_items` 视图上进行集合操作会引发 `TypeError: unhashable type 'T'`，其中 `T` 是有问题值的类型。

另一方面，`dict_keys` 视图始终可以用作集合，因为每个键都是可哈希的—按定义。

使用视图和集合运算符将节省大量循环和条件语句，当检查代码中字典内容时，让 Python 在 C 中高效实现为您工作！

就这样，我们可以结束这一章了。

# 章节总结

字典是 Python 的基石。多年来，熟悉的 `{k1: v1, k2: v2}` 文字语法得到了增强，支持使用 `**`、模式匹配以及 `dict` 推导式。

除了基本的 `dict`，标准库还提供了方便、即用即用的专用映射，如 `defaultdict`、`ChainMap` 和 `Counter`，都定义在 `collections` 模块中。随着新的 `dict` 实现，`OrderedDict` 不再像以前那样有用，但应该保留在标准库中以保持向后兼容性，并具有 `dict` 没有的特定特性，例如在 `==` 比较中考虑键的顺序。`collections` 模块中还有 `UserDict`，一个易于使用的基类，用于创建自定义映射。

大多数映射中可用的两个强大方法是 `setdefault` 和 `update`。`setdefault` 方法可以更新持有可变值的项目，例如在 `list` 值的 `dict` 中，避免为相同键进行第二次搜索。`update` 方法允许从任何其他映射、提供 `(key, value)` 对的可迭代对象以及关键字参数进行批量插入或覆盖项目。映射构造函数也在内部使用 `update`，允许实例从映射、可迭代对象或关键字参数初始化。自 Python 3.9 起，我们还可以使用 `|=` 运算符更新映射，使用 `|` 运算符从两个映射的并集创建一个新映射。

映射 API 中一个巧妙的钩子是 `__missing__` 方法，它允许你自定义当使用 `d[k]` 语法（调用 `__getitem__`）时找不到键时发生的情况。

`collections.abc` 模块提供了 `Mapping` 和 `MutableMapping` 抽象基类作为标准接口，对于运行时类型检查非常有用。`types` 模块中的 `MappingProxyType` 创建了一个不可变的外观，用于保护不希望意外更改的映射。还有用于 `Set` 和 `MutableSet` 的抽象基类。

字典视图是 Python 3 中的一个重要补充，消除了 Python 2 中 `.keys()`、`.values()` 和 `.items()` 方法造成的内存开销，这些方法构建了重复数据的列表，复制了目标 `dict` 实例中的数据。此外，`dict_keys` 和 `dict_items` 类支持 `frozenset` 的最有用的运算符和方法。

# 进一步阅读

在 Python 标准库文档中，[“collections—Container datatypes”](https://fpy.li/collec) 包括了几种映射类型的示例和实用配方。模块 *Lib/collections/__init__.py* 的 Python 源代码是任何想要创建新映射类型或理解现有映射逻辑的人的绝佳参考。David Beazley 和 Brian K. Jones 的 [*Python Cookbook*, 3rd ed.](https://fpy.li/pycook3)（O’Reilly）第一章有 20 个方便而富有见地的数据结构配方，其中大部分使用 `dict` 以巧妙的方式。

Greg Gandenberger 主张继续使用 `collections.OrderedDict`，理由是“显式胜于隐式”，向后兼容性，以及一些工具和库假定 `dict` 键的顺序是无关紧要的。他的帖子：[“Python Dictionaries Are Now Ordered. Keep Using OrderedDict”](https://fpy.li/3-18)。

[PEP 3106—Revamping dict.keys(), .values() and .items()](https://fpy.li/pep3106) 是 Guido van Rossum 为 Python 3 提出字典视图功能的地方。在摘要中，他写道这个想法来自于 Java 集合框架。

[PyPy](https://fpy.li/3-19)是第一个实现 Raymond Hettinger 提出的紧凑字典建议的 Python 解释器，他们在[“PyPy 上更快、更节省内存和更有序的字典”](https://fpy.li/3-20)中发表了博客，承认 PHP 7 中采用了类似的布局，描述在[PHP 的新哈希表实现](https://fpy.li/3-21)中。当创作者引用先前的作品时，总是很棒。

在 PyCon 2017 上，Brandon Rhodes 介绍了[“字典更强大”](https://fpy.li/3-22)，这是他经典动画演示[“强大的字典”](https://fpy.li/3-23)的续集——包括动画哈希冲突！另一部更加深入的关于 Python `dict`内部的视频是由 Raymond Hettinger 制作的[“现代字典”](https://fpy.li/3-24)，他讲述了最初未能向 CPython 核心开发人员推销紧凑字典的经历，他游说了 PyPy 团队，他们采纳了这个想法，这个想法得到了推广，并最终由 INADA Naoki 贡献给了 CPython 3.6，详情请查看[*Objects/dictobject.c*](https://fpy.li/3-26)中的 CPython 代码的详细注释和设计文档[*Objects/dictnotes.txt*](https://fpy.li/3-27)。

为了向 Python 添加集合的原因在[PEP 218—添加内置集合对象类型](https://fpy.li/pep218)中有记录。当 PEP 218 被批准时，没有采用特殊的文字语法来表示集合。`set`文字是为 Python 3 创建的，并与`dict`和`set`推导一起回溯到 Python 2.7。在 PyCon 2019 上，我介绍了[“集合实践：从 Python 的集合类型中学习”](https://fpy.li/3-29)，描述了实际程序中集合的用例，涵盖了它们的 API 设计以及使用位向量而不是哈希表的整数元素的集合类[`uintset`](https://fpy.li/3-30)的实现，灵感来自于 Alan Donovan 和 Brian Kernighan 的优秀著作[*The Go Programming Language*](http://gopl.io)第六章中的一个示例（Addison-Wesley）。

IEEE 的*Spectrum*杂志有一篇关于汉斯·彼得·卢恩的故事，他是一位多产的发明家，他申请了一项关于根据可用成分选择鸡尾酒配方的穿孔卡片盒的专利，以及其他包括…哈希表在内的多样化发明！请参阅[“汉斯·彼得·卢恩和哈希算法的诞生”](https://fpy.li/3-31)。

¹ 通过调用 ABC 的`.register()`方法注册的任何类都是虚拟子类，如“ABC 的虚拟子类”中所解释的。如果设置了特定的标记位，通过 Python/C API 实现的类型也是合格的。请参阅[`Py_TPFLAGS_MAPPING`](https://fpy.li/3-2)。

² [Python *术语表*](https://fpy.li/3-3)中关于“可散列”的条目使用“哈希值”一词，而不是*哈希码*。我更喜欢*哈希码*，因为在映射的上下文中经常讨论这个概念，其中项由键和值组成，因此提到哈希码作为值可能会令人困惑。在本书中，我只使用*哈希码*。

³ 请参阅[PEP 456—安全和可互换的哈希算法](https://fpy.li/pep456)以了解安全性问题和采用的解决方案。

⁴ 原始脚本出现在 Martelli 的[“重新学习 Python”演示](https://fpy.li/3-5)的第 41 页中。他的脚本实际上是`dict.setdefault`的演示，如我们的示例 3-5 所示。

⁵ 这是将方法作为一等函数使用的示例，是第七章的主题。

⁶ 其中一个库是[*Pingo.io*](https://fpy.li/3-6)，目前已不再进行活跃开发。

⁷ 关于子类化`dict`和其他内置类型的确切问题在“子类化内置类型是棘手的”中有所涵盖。

⁸ 这就是元组的存储方式。

⁹ 除非类有一个`__slots__`属性，如“使用 __slots__ 节省内存”中所解释的那样。

¹⁰ 这可能很有趣，但并不是非常重要。加速只会在评估集合字面值时发生，而这最多只会发生一次 Python 进程—当模块最初编译时。如果你好奇，可以从`dis`模块中导入`dis`函数，并使用它来反汇编`set`字面值的字节码—例如，`dis('{1}')`—和`set`调用—`dis('set([1])')`。
