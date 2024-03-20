# 第十五章：关于类型提示的更多内容

> 我学到了一个痛苦的教训，对于小程序来说，动态类型很棒。对于大型程序，你需要更加纪律严明的方法。如果语言给予你这种纪律，而不是告诉你“嗯，你可以做任何你想做的事情”，那会更有帮助。
> 
> Guido van Rossum，蒙提·派森的粉丝¹

本章是第八章的续集，涵盖了更多关于 Python 渐进类型系统的内容。主要议题包括：

+   重载函数签名

+   `typing.TypedDict`用于对作为记录使用的`dicts`进行类型提示

+   类型转换

+   运行时访问类型提示

+   通用类型

    +   声明一个通用类

    +   变异：不变、协变和逆变类型

    +   通用静态协议

# 本章的新内容

本章是《流畅的 Python》第二版中的新内容。让我们从重载开始。

# 重载签名

Python 函数可以接受不同组合的参数。`@typing.overload`装饰器允许对这些不同组合进行注释。当函数的返回类型取决于两个或更多参数的类型时，这一点尤为重要。

考虑内置函数`sum`。这是`help(sum)`的文本：

```py
>>> help(sum)
sum(iterable, /, start=0)
    Return the sum of a 'start' value (default: 0) plus an iterable of numbers

    When the iterable is empty, return the start value.
    This function is intended specifically for use with numeric values and may
    reject non-numeric types.
```

内置函数`sum`是用 C 编写的，但*typeshed*为其提供了重载类型提示，在[*builtins.pyi*](https://fpy.li/15-2)中有：

```py
@overload
def sum(__iterable: Iterable[_T]) -> Union[_T, int]: ...
@overload
def sum(__iterable: Iterable[_T], start: _S) -> Union[_T, _S]: ...
```

首先让我们看看重载的整体语法。这是存根文件（*.pyi*）中关于`sum`的所有代码。实现将在另一个文件中。省略号（`...`）除了满足函数体的语法要求外没有其他作用，类似于`pass`。因此，*.pyi*文件是有效的 Python 文件。

正如在“注释位置参数和可变参数”中提到的，`__iterable`中的两个下划线是 PEP 484 对位置参数的约定，由 Mypy 强制执行。这意味着你可以调用`sum(my_list)`，但不能调用`sum(__iterable = my_list)`。

类型检查器尝试将给定的参数与每个重载签名进行匹配，按顺序。调用`sum(range(100), 1000)`不匹配第一个重载，因为该签名只有一个参数。但它匹配第二个。

你也可以在普通的 Python 模块中使用`@overload`，只需在函数的实际签名和实现之前写上重载的签名即可。示例 15-1 展示了如何在 Python 模块中注释和实现`sum`。

##### 示例 15-1。*mysum.py*：带有重载签名的`sum`函数的定义

```py
import functools
import operator
from collections.abc import Iterable
from typing import overload, Union, TypeVar

T = TypeVar('T')
S = TypeVar('S')  # ①

@overload
def sum(it: Iterable[T]) -> Union[T, int]: ...  # ②
@overload
def sum(it: Iterable[T], /, start: S) -> Union[T, S]: ...  # ③
def sum(it, /, start=0):  # ④
    return functools.reduce(operator.add, it, start)
```

①

我们在第二个重载中需要这第二个`TypeVar`。

②

这个签名是针对简单情况的：`sum(my_iterable)`。结果类型可能是`T`——`my_iterable`产生的元素的类型，或者如果可迭代对象为空，则可能是`int`，因为`start`参数的默认值是`0`。

③

当给定`start`时，它可以是任何类型`S`，因此结果类型是`Union[T, S]`。这就是为什么我们需要`S`。如果我们重用`T`，那么`start`的类型将必须与`Iterable[T]`的元素类型相同。

④

实际函数实现的签名没有类型提示。

这是为了注释一行函数而写的很多行代码。我知道这可能有点过头了。至少这不是一个`foo`函数。

如果你想通过阅读代码了解`@overload`，*typeshed*有数百个示例。在*typeshed*上，Python 内置函数的[存根文件](https://fpy.li/15-3)在我写这篇文章时有 186 个重载——比标准库中的任何其他函数都多。

# 利用渐进类型

追求 100% 的注释代码可能会导致添加大量噪音但很少价值的类型提示。简化类型提示以简化重构可能会导致繁琐的 API。有时最好是务实一些，让一段代码没有类型提示。

我们称之为 Pythonic 的方便 API 往往很难注释。在下一节中，我们将看到一个例子：需要六个重载才能正确注释灵活的内置 `max` 函数。

## Max Overload

给利用 Python 强大动态特性的函数添加类型提示是困难的。

在研究 typeshed 时，我发现了 bug 报告 [#4051](https://fpy.li/shed4051)：Mypy 没有警告说将 `None` 作为内置 `max()` 函数的参数之一是非法的，或者传递一个在某个时刻产生 `None` 的可迭代对象也是非法的。在任一情况下，你会得到像这样的运行时异常：

```py
TypeError: '>' not supported between instances of 'int' and 'NoneType'
```

`max` 的文档以这句话开头：

> 返回可迭代对象中的最大项或两个或多个参数中的最大项。

对我来说，这是一个非常直观的描述。

但如果我必须为以这些术语描述的函数注释，我必须问：它是哪个？一个可迭代对象还是两个或更多参数？

实际情况更加复杂，因为 `max` 还接受两个可选关键字参数：`key` 和 `default`。

我在 Python 中编写了 `max` 来更容易地看到它的工作方式和重载注释之间的关系（内置的 `max` 是用 C 编写的）；参见 Example 15-2。

##### Example 15-2\. *mymax.py*：`max` 函数的 Python 重写

```py
# imports and definitions omitted, see next listing

MISSING = object()
EMPTY_MSG = 'max() arg is an empty sequence'

# overloaded type hints omitted, see next listing

def max(first, *args, key=None, default=MISSING):
    if args:
        series = args
        candidate = first
    else:
        series = iter(first)
        try:
            candidate = next(series)
        except StopIteration:
            if default is not MISSING:
                return default
            raise ValueError(EMPTY_MSG) from None
    if key is None:
        for current in series:
            if candidate < current:
                candidate = current
    else:
        candidate_key = key(candidate)
        for current in series:
            current_key = key(current)
            if candidate_key < current_key:
                candidate = current
                candidate_key = current_key
    return candidate
```

这个示例的重点不是 `max` 的逻辑，所以我不会花时间解释它的实现，除了解释 `MISSING`。`MISSING` 常量是一个用作哨兵的唯一 `object` 实例。它是 `default=` 关键字参数的默认值，这样 `max` 可以接受 `default=None` 并仍然区分这两种情况：

1.  用户没有为 `default=` 提供值，因此它是 `MISSING`，如果 `first` 是一个空的可迭代对象，`max` 将引发 `ValueError`。

1.  用户为 `default=` 提供了一些值，包括 `None`，因此如果 `first` 是一个空的可迭代对象，`max` 将返回该值。

为了修复 [问题 #4051](https://fpy.li/shed4051)，我写了 Example 15-3 中的代码。²

##### Example 15-3\. *mymax.py*：模块顶部，包括导入、定义和重载

```py
from collections.abc import Callable, Iterable
from typing import Protocol, Any, TypeVar, overload, Union

class SupportsLessThan(Protocol):
    def __lt__(self, other: Any) -> bool: ...

T = TypeVar('T')
LT = TypeVar('LT', bound=SupportsLessThan)
DT = TypeVar('DT')

MISSING = object()
EMPTY_MSG = 'max() arg is an empty sequence'

@overload
def max(__arg1: LT, __arg2: LT, *args: LT, key: None = ...) -> LT:
    ...
@overload
def max(__arg1: T, __arg2: T, *args: T, key: Callable[[T], LT]) -> T:
    ...
@overload
def max(__iterable: Iterable[LT], *, key: None = ...) -> LT:
    ...
@overload
def max(__iterable: Iterable[T], *, key: Callable[[T], LT]) -> T:
    ...
@overload
def max(__iterable: Iterable[LT], *, key: None = ...,
        default: DT) -> Union[LT, DT]:
    ...
@overload
def max(__iterable: Iterable[T], *, key: Callable[[T], LT],
        default: DT) -> Union[T, DT]:
    ...
```

我的 Python 实现的 `max` 与所有那些类型导入和声明的长度大致相同。由于鸭子类型，我的代码没有 `isinstance` 检查，并且提供了与那些类型提示相同的错误检查，但当然只在运行时。

`@overload` 的一个关键优势是尽可能精确地声明返回类型，根据给定的参数类型。我们将通过逐组一到两个地研究`max`的重载来看到这个优势。

### 实现了 SupportsLessThan 的参数，但未提供 key 和 default

```py
@overload
def max(__arg1: LT, __arg2: LT, *_args: LT, key: None = ...) -> LT:
    ...
# ... lines omitted ...
@overload
def max(__iterable: Iterable[LT], *, key: None = ...) -> LT:
    ...
```

在这些情况下，输入要么是实现了 `SupportsLessThan` 的类型 `LT` 的单独参数，要么是这些项目的 `Iterable`。`max` 的返回类型与实际参数或项目相同，正如我们在 “Bounded TypeVar” 中看到的。

符合这些重载的示例调用：

```py
max(1, 2, -3)  # returns 2
max(['Go', 'Python', 'Rust'])  # returns 'Rust'
```

### 提供了 key 参数，但没有提供 default

```py
@overload
def max(__arg1: T, __arg2: T, *_args: T, key: Callable[[T], LT]) -> T:
    ...
# ... lines omitted ...
@overload
def max(__iterable: Iterable[T], *, key: Callable[[T], LT]) -> T:
    ...
```

输入可以是任何类型 `T` 的单独项目或单个 `Iterable[T]`，`key=` 必须是一个接受相同类型 `T` 的参数并返回一个实现 `SupportsLessThan` 的值的可调用对象。`max` 的返回类型与实际参数相同。

符合这些重载的示例调用：

```py
max(1, 2, -3, key=abs)  # returns -3
max(['Go', 'Python', 'Rust'], key=len)  # returns 'Python'
```

### 提供了 default 参数，但没有 key

```py
@overload
def max(__iterable: Iterable[LT], *, key: None = ...,
        default: DT) -> Union[LT, DT]:
    ...
```

输入是一个实现 `SupportsLessThan` 的类型 `LT` 的项目的可迭代对象。`default=` 参数是当 `Iterable` 为空时的返回值。因此，`max` 的返回类型必须是 `LT` 类型和 `default` 参数类型的 `Union`。

符合这些重载的示例调用：

```py
max([1, 2, -3], default=0)  # returns 2
max([], default=None)  # returns None
```

### 提供了 key 和 default 参数

```py
@overload
def max(__iterable: Iterable[T], *, key: Callable[[T], LT],
        default: DT) -> Union[T, DT]:
    ...
```

输入是：

+   任何类型 `T` 的项目的可迭代对象

+   接受类型为`T`的参数并返回实现`SupportsLessThan`的类型`LT`的值的可调用函数

+   任何类型`DT`的默认值

`max`的返回类型必须是类型`T`或`default`参数的类型的`Union`：

```py
max([1, 2, -3], key=abs, default=None)  # returns -3
max([], key=abs, default=None)  # returns None
```

## 从重载`max`中得到的经验教训

类型提示允许 Mypy 标记像`max([None, None])`这样的调用，并显示以下错误消息：

```py
mymax_demo.py:109: error: Value of type variable "_LT" of "max"
  cannot be "None"
```

另一方面，为了维持类型检查器而写这么多行可能会阻止人们编写方便灵活的函数，如`max`。如果我不得不重新发明`min`函数，我可以重构并重用大部分`max`的实现。但我必须复制并粘贴所有重载的声明——尽管它们对于`min`来说是相同的，除了函数名称。

我的朋友 João S. O. Bueno——我认识的最聪明的 Python 开发者之一——在推特上发表了[这篇推文](https://fpy.li/15-4)：

> 尽管很难表达`max`的签名——但它很容易理解。我理解的是，与 Python 相比，注释标记的表现力非常有限。

现在让我们来研究`TypedDict`类型构造。一开始我认为它并不像我想象的那么有用，但它有其用途。尝试使用`TypedDict`来处理动态结构（如 JSON 数据）展示了静态类型处理的局限性。

# TypedDict

###### 警告

使用`TypedDict`来保护处理动态数据结构（如 JSON API 响应）中的错误是很诱人的。但这里的示例清楚地表明，对 JSON 的正确处理必须在运行时完成，而不是通过静态类型检查。要使用类型提示对类似 JSON 的结构进行运行时检查，请查看 PyPI 上的[*pydantic*](https://fpy.li/15-5)包。

Python 字典有时被用作记录，其中键用作字段名称，不同类型的字段值。

例如，考虑描述 JSON 或 Python 中的一本书的记录：

```py
{"isbn": "0134757599",
 "title": "Refactoring, 2e",
 "authors": ["Martin Fowler", "Kent Beck"],
 "pagecount": 478}
```

在 Python 3.8 之前，没有很好的方法来注释这样的记录，因为我们在“通用映射”中看到的映射类型限制所有值具有相同的类型。

这里有两个尴尬的尝试来注释类似前述 JSON 对象的记录：

`Dict[str, Any]`

值可以是任何类型。

`Dict[str, Union[str, int, List[str]]]`

难以阅读，并且不保留字段名称和其相应字段类型之间的关系：`title`应该是一个`str`，不能是一个`int`或`List[str]`。

[PEP 589—TypedDict: 具有固定键集的字典的类型提示](https://fpy.li/pep589)解决了这个问题。示例 15-4 展示了一个简单的`TypedDict`。

##### 示例 15-4。*books.py*：`BookDict`定义

```py
from typing import TypedDict

class BookDict(TypedDict):
    isbn: str
    title: str
    authors: list[str]
    pagecount: int
```

乍一看，`typing.TypedDict`可能看起来像是一个数据类构建器，类似于`typing.NamedTuple`—在第五章中介绍过。

语法上的相似性是误导的。`TypedDict`非常不同。它仅存在于类型检查器的利益，并且在运行时没有影响。

`TypedDict`提供了两个东西：

+   类似类的语法来注释每个“字段”的值的`dict`类型提示。

+   一个构造函数，告诉类型检查器期望一个带有指定键和值的`dict`。

在运行时，像`BookDict`这样的`TypedDict`构造函数是一个安慰剂：它与使用相同参数调用`dict`构造函数具有相同效果。

`BookDict`创建一个普通的`dict`也意味着：

+   伪类定义中的“字段”不会创建实例属性。

+   你不能为“字段”编写具有默认值的初始化程序。

+   不允许方法定义。

让我们在运行时探索一个`BookDict`的行为（示例 15-5）。

##### 示例 15-5。使用`BookDict`，但并非完全按照预期

```py
>>> from books import BookDict
>>> pp = BookDict(title='Programming Pearls',  # ①
...               authors='Jon Bentley',  # ②
...               isbn='0201657880',
...               pagecount=256)
>>> pp  # ③
{'title': 'Programming Pearls', 'authors': 'Jon Bentley', 'isbn': '0201657880',
 'pagecount': 256} >>> type(pp)
<class 'dict'> >>> pp.title  # ④
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
AttributeError: 'dict' object has no attribute 'title'
>>> pp['title']
'Programming Pearls' >>> BookDict.__annotations__  # ⑤
{'isbn': <class 'str'>, 'title': <class 'str'>, 'authors': typing.List[str],
 'pagecount': <class 'int'>}
```

①

你可以像使用`dict`构造函数一样调用`BookDict`，使用关键字参数，或传递一个`dict`参数，包括`dict`文字。

②

糟糕…我忘记了 `authors` 接受一个列表。但渐进式类型意味着在运行时没有类型检查。

③

调用 `BookDict` 的结果是一个普通的 `dict`…

④

…因此您不能使用 `object.field` 记法读取数据。

⑤

类型提示位于 `BookDict.__annotations__` 中，而不是 `pp`。

没有类型检查器，`TypedDict` 就像注释一样有用：它可以帮助人们阅读代码，但仅此而已。相比之下，来自 第五章 的类构建器即使不使用类型检查器也很有用，因为在运行时它们会生成或增强一个自定义类，您可以实例化。它们还提供了 表 5-1 中列出的几个有用的方法或函数。

示例 15-6 构建了一个有效的 `BookDict`，并尝试对其进行一些操作。这展示了 `TypedDict` 如何使 Mypy 能够捕获错误，如 示例 15-7 中所示。

##### 示例 15-6\. *demo_books.py*: 在 `BookDict` 上进行合法和非法操作

```py
from books import BookDict
from typing import TYPE_CHECKING

def demo() -> None:  # ①
    book = BookDict(  # ②
        isbn='0134757599',
        title='Refactoring, 2e',
        authors=['Martin Fowler', 'Kent Beck'],
        pagecount=478
    )
    authors = book['authors'] # ③
    if TYPE_CHECKING:  # ④
        reveal_type(authors)  # ⑤
    authors = 'Bob'  # ⑥
    book['weight'] = 4.2
    del book['title']

if __name__ == '__main__':
    demo()
```

①

记得添加返回类型，这样 Mypy 不会忽略函数。

②

这是一个有效的 `BookDict`：所有键都存在，并且具有正确类型的值。

③

Mypy 将从 `BookDict` 中 `'authors'` 键的注释中推断出 `authors` 的类型。

④

`typing.TYPE_CHECKING` 仅在程序进行类型检查时为 `True`。在运行时，它始终为 false。

⑤

前一个 `if` 语句阻止了在运行时调用 `reveal_type(authors)`。`reveal_type` 不是运行时 Python 函数，而是 Mypy 提供的调试工具。这就是为什么没有为它导入的原因。在 示例 15-7 中查看其输出。

⑥

`demo` 函数的最后三行是非法的。它们会在 示例 15-7 中导致错误消息。

对 *demo_books.py* 进行类型检查，来自 示例 15-6，我们得到 示例 15-7。

##### 示例 15-7\. 对 *demo_books.py* 进行类型检查

```py
…/typeddict/ $ mypy demo_books.py
demo_books.py:13: note: Revealed type is 'built-ins.list[built-ins.str]'  # ①
demo_books.py:14: error: Incompatible types in assignment
                  (expression has type "str", variable has type "List[str]")  # ②
demo_books.py:15: error: TypedDict "BookDict" has no key 'weight'  # ③
demo_books.py:16: error: Key 'title' of TypedDict "BookDict" cannot be deleted  # ④
Found 3 errors in 1 file (checked 1 source file)
```

①

这个注释是 `reveal_type(authors)` 的结果。

②

`authors` 变量的类型是从初始化它的 `book['authors']` 表达式的类型推断出来的。您不能将 `str` 赋给类型为 `List[str]` 的变量。类型检查器通常不允许变量的类型更改。³

③

无法为不属于 `BookDict` 定义的键赋值。

④

无法删除属于 `BookDict` 定义的键。

现在让我们看看在函数签名中使用 `BookDict`，以进行函数调用的类型检查。

想象一下，你需要从书籍记录生成类似于这样的 XML：

```py
<BOOK>
  <ISBN>0134757599</ISBN>
  <TITLE>Refactoring, 2e</TITLE>
  <AUTHOR>Martin Fowler</AUTHOR>
  <AUTHOR>Kent Beck</AUTHOR>
  <PAGECOUNT>478</PAGECOUNT>
</BOOK>
```

如果您正在编写要嵌入到微型微控制器中的 MicroPython 代码，您可能会编写类似于 示例 15-8 中所示的函数。⁴

##### 示例 15-8\. *books.py*: `to_xml` 函数

```py
AUTHOR_ELEMENT = '<AUTHOR>{}</AUTHOR>'

def to_xml(book: BookDict) -> str:  # ①
    elements: list[str] = []  # ②
    for key, value in book.items():
        if isinstance(value, list):  # ③
            elements.extend(
                AUTHOR_ELEMENT.format(n) for n in value)  # ④
        else:
            tag = key.upper()
            elements.append(f'<{tag}>{value}</{tag}>')
    xml = '\n\t'.join(elements)
    return f'<BOOK>\n\t{xml}\n</BOOK>'
```

①

示例的整个重点：在函数签名中使用 `BookDict`。

②

经常需要注释开始为空的集合，否则 Mypy 无法推断元素的类型。⁵

③

Mypy 理解 `isinstance` 检查，并在此块中将 `value` 视为 `list`。

④

当我将`key == 'authors'`作为`if`条件来保护这个块时，Mypy 在这一行发现了一个错误：“"object"没有属性"__iter__"”，因为它推断出从`book.items()`返回的`value`类型为`object`，而`object`不支持生成器表达式所需的`__iter__`方法。通过`isinstance`检查，这可以工作，因为 Mypy 知道在这个块中`value`是一个`list`。

示例 15-9（#from_json_any_ex）展示了一个解析 JSON `str`并返回`BookDict`的函数。

##### 示例 15-9\. books_any.py：`from_json`函数

```py
def from_json(data: str) -> BookDict:
    whatever = json.loads(data)  # ①
    return whatever  # ②
```

①

`json.loads()`的返回类型是`Any`。⁶

②

我可以返回`whatever`—类型为`Any`—因为`Any`与每种类型都*一致*，包括声明的返回类型`BookDict`。

示例 15-9 的第二点非常重要要记住：Mypy 不会在这段代码中标记任何问题，但在运行时，`whatever`中的值可能不符合`BookDict`结构—实际上，它可能根本不是`dict`！

如果你使用`--disallow-any-expr`运行 Mypy，它会抱怨`from_json`函数体中的两行代码：

```py
…/typeddict/ $ mypy books_any.py --disallow-any-expr
books_any.py:30: error: Expression has type "Any"
books_any.py:31: error: Expression has type "Any"
Found 2 errors in 1 file (checked 1 source file)
```

前一段代码中提到的第 30 行和 31 行是`from_json`函数的主体。我们可以通过在`whatever`变量初始化时添加类型提示来消除类型错误，就像示例 15-10 中那样。

##### 示例 15-10\. books.py：带有变量注释的`from_json`函数。

```py
def from_json(data: str) -> BookDict:
    whatever: BookDict = json.loads(data)  # ①
    return whatever  # ②
```

①

当将类型为`Any`的表达式立即分配给带有类型提示的变量时，`--disallow-any-expr`不会导致错误。

②

现在`whatever`的类型是`BookDict`，即声明的返回类型。

###### 警告

不要被示例 15-10 的虚假类型安全感所蒙蔽！从静态代码看，类型检查器无法预测`json.loads()`会返回任何类似于`BookDict`的东西。只有运行时验证才能保证这一点。

静态类型检查无法防止与本质上动态的代码出现错误，比如`json.loads()`，它在运行时构建不同类型的 Python 对象，正如示例 15-11、15-12 和 15-13 所展示的。

##### 示例 15-11\. demo_not_book.py：`from_json`返回一个无效的`BookDict`，而`to_xml`接受它

```py
from books import to_xml, from_json
from typing import TYPE_CHECKING

def demo() -> None:
    NOT_BOOK_JSON = """
 {"title": "Andromeda Strain",
         "flavor": "pistachio",
         "authors": true}
    """
    not_book = from_json(NOT_BOOK_JSON)  # ①
    if TYPE_CHECKING:  # ②
        reveal_type(not_book)
        reveal_type(not_book['authors'])

    print(not_book)  # ③
    print(not_book['flavor'])  # ④

    xml = to_xml(not_book)  # ⑤
    print(xml)  # ⑥

if __name__ == '__main__':
    demo()
```

①

这行代码不会产生有效的`BookDict`—查看`NOT_BOOK_JSON`的内容。

②

让我们揭示一些类型。

③

这不应该是问题：`print`可以处理`object`和其他任何类型。

④

`BookDict`没有`'flavor'`键，但 JSON 源有…会发生什么？

⑤

记住签名：`def to_xml(book: BookDict) -> str:`

⑥

XML 输出会是什么样子？

现在我们用 Mypy 检查*demo_not_book.py*（示例 15-12）。

##### 示例 15-12\. *demo_not_book.py*的 Mypy 报告，为了清晰起见重新格式化

```py
…/typeddict/ $ mypy demo_not_book.py
demo_not_book.py:12: note: Revealed type is
   'TypedDict('books.BookDict', {'isbn': built-ins.str,
                                 'title': built-ins.str,
                                 'authors': built-ins.list[built-ins.str],
                                 'pagecount': built-ins.int})'  # ①
demo_not_book.py:13: note: Revealed type is 'built-ins.list[built-ins.str]'  # ②
demo_not_book.py:16: error: TypedDict "BookDict" has no key 'flavor'  # ③
Found 1 error in 1 file (checked 1 source file)
```

①

显式类型是名义类型，而不是`not_book`的运行时内容。

②

同样，这是`not_book['authors']`的名义类型，如`BookDict`中定义的那样。而不是运行时类型。

③

这个错误是针对`print(not_book['flavor'])`这一行的：该键在名义类型中不存在。

现在让我们运行*demo_not_book.py*，并在示例 15-13 中显示输出。

##### 示例 15-13\. 运行 `demo_not_book.py` 的输出

```py
…/typeddict/ $ python3 demo_not_book.py
{'title': 'Andromeda Strain', 'flavor': 'pistachio', 'authors': True}  # ①
pistachio  # ②
<BOOK>  # ③
        <TITLE>Andromeda Strain</TITLE>
        <FLAVOR>pistachio</FLAVOR>
        <AUTHORS>True</AUTHORS>
</BOOK>
```

①

这实际上不是一个 `BookDict`。

②

`not_book['flavor']` 的值。

③

`to_xml` 接受一个 `BookDict` 参数，但没有运行时检查：垃圾进，垃圾出。

示例 15-13 显示 *demo_not_book.py* 输出了无意义的内容，但没有运行时错误。在处理 JSON 数据时使用 `TypedDict` 并没有提供太多类型安全性。

如果你通过鸭子类型的视角查看示例 15-8 中`to_xml`的代码，那么参数`book`必须提供一个返回类似`(key, value)`元组可迭代对象的`.items()`方法，其中：

+   `key` 必须有一个 `.upper()` 方法

+   `value` 可以是任何东西

这个演示的重点是：当处理具有动态结构的数据，比如 JSON 或 XML 时，`TypedDict` 绝对不能替代运行时的数据验证。为此，请使用[*pydantic*](https://fpy.li/15-5)。

`TypedDict` 具有更多功能，包括支持可选键、有限形式的继承以及另一种声明语法。如果您想了解更多，请查看 [PEP 589—TypedDict: Type Hints for Dictionaries with a Fixed Set of Keys](https://fpy.li/pep589)。

现在让我们将注意力转向一个最好避免但有时不可避免的函数：`typing.cast`。

# 类型转换

没有完美的类型系统，静态类型检查器、*typeshed* 项目中的类型提示或具有类型提示的第三方包也不是完美的。

`typing.cast()` 特殊函数提供了一种处理类型检查故障或代码中不正确类型提示的方法。[Mypy 0.930 文档](https://fpy.li/15-14)解释：

> Casts 用于消除杂乱的类型检查器警告，并在类型检查器无法完全理解情况时为其提供一点帮助。

在运行时，`typing.cast` 绝对不起作用。这是它的[实现](https://fpy.li/15-15)：

```py
def cast(typ, val):
    """Cast a value to a type.
 This returns the value unchanged.  To the type checker this
 signals that the return value has the designated type, but at
 runtime we intentionally don't check anything (we want this
 to be as fast as possible).
 """
    return val
```

PEP 484 要求类型检查器“盲目相信”`cast` 中声明的类型。[PEP 484 的“Casts”部分](https://fpy.li/15-16)提供了一个需要 `cast` 指导的示例：

```py
from typing import cast

def find_first_str(a: list[object]) -> str:
    index = next(i for i, x in enumerate(a) if isinstance(x, str))
    # We only get here if there's at least one string
    return cast(str, a[index])
```

对生成器表达式的 `next()` 调用将返回 `str` 项的索引或引发 `StopIteration`。因此，如果没有引发异常，`find_first_str` 将始终返回一个 `str`，而 `str` 是声明的返回类型。

但如果最后一行只是 `return a[index]`，Mypy 将推断返回类型为 `object`，因为 `a` 参数声明为 `list[object]`。因此，需要 `cast()` 来指导 Mypy。⁷

这里是另一个使用 `cast` 的示例，这次是为了纠正 Python 标准库中过时的类型提示。在示例 21-12 中，我创建了一个 *asyncio* `Server` 对象，并且我想获取服务器正在侦听的地址。我编写了这行代码：

```py
addr = server.sockets[0].getsockname()
```

但 Mypy 报告了这个错误：

```py
Value of type "Optional[List[socket]]" is not indexable
```

2021 年 5 月 *typeshed* 中 `Server.sockets` 的类型提示对 Python 3.6 是有效的，其中 `sockets` 属性可以是 `None`。但在 Python 3.7 中，`sockets` 变成了一个始终返回 `list` 的属性，如果服务器没有 sockets，则可能为空。自 Python 3.8 起，getter 返回一个 `tuple`（用作不可变序列）。

由于我现在无法修复 *typeshed*，⁸ 我添加了一个 `cast`，就像这样：

```py
from asyncio.trsock import TransportSocket
from typing import cast

# ... many lines omitted ...

    socket_list = cast(tuple[TransportSocket, ...], server.sockets)
    addr = socket_list[0].getsockname()
```

在这种情况下使用 `cast` 需要花费几个小时来理解问题，并阅读 *asyncio* 源代码以找到正确的 sockets 类型：来自未记录的 `asyncio.trsock` 模块的 `TransportSocket` 类。我还必须添加两个 `import` 语句和另一行代码以提高可读性。⁹ 但代码更安全。

细心的读者可能会注意到，如果 `sockets` 为空，`sockets[0]` 可能会引发 `IndexError`。但就我对 `asyncio` 的理解而言，在 示例 21-12 中不会发生这种情况，因为 `server` 在我读取其 `sockets` 属性时已准备好接受连接，因此它不会为空。无论如何，`IndexError` 是一个运行时错误。Mypy 甚至在像 `print([][0])` 这样的简单情况下也无法发现问题。

###### 警告

不要过于依赖 `cast` 来消除 Mypy 的警告，因为当 Mypy 报告错误时，通常是正确的。如果你经常使用 `cast`，那是一个[代码异味](https://fpy.li/15-20)。你的团队可能在误用类型提示，或者你的代码库中可能存在低质量的依赖项。

尽管存在缺点，`cast` 也有其有效用途。以下是 Guido van Rossum 关于它的一些观点：

> 有什么问题，偶尔调用 `cast()` 或添加 `# type: ignore` 注释吗？¹⁰

完全禁止使用 `cast` 是不明智的，特别是因为其他解决方法更糟糕：

+   `# type: ignore` 提供的信息较少。¹¹

+   使用 `Any` 是具有传染性的：由于 `Any` 与所有类型*一致*，滥用它可能通过类型推断产生级联效应，削弱类型检查器在代码其他部分检测错误的能力。

当然，并非所有类型错误都可以使用 `cast` 修复。有时我们需要 `# type: ignore`，偶尔需要 `Any`，甚至可以在函数中不留类型提示。

接下来，让我们谈谈在运行时使用注释。

# 在运行时读取类型提示

在导入时，Python 读取函数、类和模块中的类型提示，并将它们存储在名为 `__annotations__` 的属性中。例如，考虑 示例 15-14 中的 `clip` 函数。¹²

##### 示例 15-14\. clipannot.py：`clip` 函数的带注释签名

```py
def clip(text: str, max_len: int = 80) -> str:
```

类型提示存储为函数的 `__annotations__` 属性中的 `dict`：

```py
>>> from clip_annot import clip
>>> clip.__annotations__
{'text': <class 'str'>, 'max_len': <class 'int'>, 'return': <class 'str'>}
```

`'return'` 键映射到 `->` 符号后的返回类型提示，在 示例 15-14 中。

请注意，注释在导入时由解释器评估，就像参数默认值也会被评估一样。这就是为什么注释中的值是 Python 类 `str` 和 `int`，而不是字符串 `'str'` 和 `'int'`。注释的导入时评估是 Python 3.10 的标准，但如果 [PEP 563](https://fpy.li/pep563) 或 [PEP 649](https://fpy.li/pep649) 成为标准行为，这可能会改变。

## 运行时的注释问题

类型提示的增加使用引发了两个问题：

+   当使用许多类型提示时，导入模块会消耗更多的 CPU 和内存。

+   引用尚未定义的类型需要使用字符串而不是实际类型。

这两个问题都很重要。第一个问题是因为我们刚刚看到的：注释在导入时由解释器评估并存储在 `__annotations__` 属性中。现在让我们专注于第二个问题。

有时需要将注释存储为字符串，因为存在“前向引用”问题：当类型提示需要引用在同一模块下定义的类时。然而，在源代码中问题的常见表现根本不像前向引用：当方法返回同一类的新对象时。由于在 Python 完全评估类体之前类对象未定义，类型提示必须使用类名作为字符串。以下是一个示例：

```py
class Rectangle:
    # ... lines omitted ...
    def stretch(self, factor: float) -> 'Rectangle':
        return Rectangle(width=self.width * factor)
```

将前向引用类型提示写为字符串是 Python 3.10 的标准和必需做法。静态类型检查器从一开始就设计用于处理这个问题。

但在运行时，如果编写代码读取 `stretch` 的 `return` 注释，你将得到一个字符串 `'Rectangle'` 而不是实际类型，即 `Rectangle` 类的引用。现在你的代码需要弄清楚那个字符串的含义。

`typing`模块包括三个函数和一个分类为[内省助手](https://fpy.li/15-24)的类，其中最重要的是`typing.get_type_hints`。其部分文档如下：

`get_type_hints(obj, globals=None, locals=None, include_extras=False)`

[…] 这通常与`obj.__annotations__`相同。此外，以字符串文字编码的前向引用通过在`globals`和`locals`命名空间中评估来处理。[…]

###### 警告

自 Python 3.10 开始，应该使用新的[`inspect.get_annotations(…)`](https://fpy.li/15-25)函数，而不是`typing.​get_​type_​hints`。然而，一些读者可能尚未使用 Python 3.10，因此在示例中我将使用`typing.​get_​type_​hints`，自从`typing`模块在 Python 3.5 中添加以来就可用。

[PEP 563—注释的延迟评估](https://fpy.li/pep563)已经获得批准，使得不再需要将注释写成字符串，并减少类型提示的运行时成本。其主要思想在[“摘要”](https://fpy.li/15-26)的这两句话中描述：

> 本 PEP 建议更改函数注释和变量注释，使其不再在函数定义时评估。相反，它们以字符串形式保留在*注释*中。

从 Python 3.7 开始，这就是在任何以此`import`语句开头的模块中处理注释的方式：

```py
from __future__ import annotations
```

为了展示其效果，我将与顶部的`__future__`导入行相同的`clip`函数的副本放在了一个名为*clip_annot_post.py*的模块中。

在控制台上，当我导入该模块并读取`clip`的注释时，这是我得到的结果：

```py
>>> from clip_annot_post import clip
>>> clip.__annotations__
{'text': 'str', 'max_len': 'int', 'return': 'str'}
```

如您所见，所有类型提示现在都是普通字符串，尽管它们在`clip`的定义中并非作为引号字符串编写（示例 15-14）。

`typing.get_type_hints`函数能够解析许多类型提示，包括`clip`中的类型提示：

```py
>>> from clip_annot_post import clip
>>> from typing import get_type_hints
>>> get_type_hints(clip)
{'text': <class 'str'>, 'max_len': <class 'int'>, 'return': <class 'str'>}
```

调用`get_type_hints`会给我们真实的类型，即使在某些情况下原始类型提示是作为引号字符串编写的。这是在运行时读取类型提示的推荐方式。

PEP 563 的行为原计划在 Python 3.10 中成为默认行为，无需`__future__`导入。然而，*FastAPI* 和 *pydantic* 的维护者发出警告，称这一变化将破坏依赖运行时类型提示的代码，并且无法可靠使用`get_type_hints`。

在 python-dev 邮件列表上的讨论中，PEP 563 的作者 Łukasz Langa 描述了该函数的一些限制：

> […] 结果表明，`typing.get_type_hints()`存在一些限制，使得其在一般情况下在运行时成本高昂，并且更重要的是无法解析所有类型。最常见的例子涉及生成类型的非全局上下文（例如，内部类、函数内的类等）。但是，一个前向引用的典型例子是：具有接受或返回其自身类型对象的方法的类，如果使用类生成器，则`typing.get_type_hints()`也无法正确处理。我们可以做一些技巧来连接这些点，但总体来说并不是很好。¹³

Python 的指导委员会决定将 PEP 563 的默认行为推迟到 Python 3.11 或更高版本，以便开发人员有更多时间提出解决 PEP 563 试图解决的问题的解决方案，而不会破坏运行时类型提示的广泛使用。[PEP 649—使用描述符推迟评估注释](https://fpy.li/pep649)正在考虑作为可能的解决方案，但可能会达成不同的妥协。

总结一下：截至 Python 3.10，运行时读取类型提示并不是 100%可靠的，可能会在 2022 年发生变化。

###### 注意

在大规模使用 Python 的公司中，他们希望获得静态类型的好处，但不想在导入时评估类型提示的代价。静态检查发生在开发人员的工作站和专用 CI 服务器上，但在生产容器中，模块的加载频率和数量要高得多，这种成本在规模上是不可忽略的。

这在 Python 社区中引发了紧张气氛，一方面是希望类型提示仅以字符串形式存储，以减少加载成本，另一方面是希望在运行时也使用类型提示的人，比如 *pydantic* 和 *FastAPI* 的创建者和用户，他们更希望将类型对象存储起来，而不是评估这些注释，这是一项具有挑战性的任务。

## 处理问题

鉴于目前的不稳定局势，如果您需要在运行时阅读注释，我建议：

+   避免直接读取`__annotations__`；而是使用`inspect.get_annotations`（从 Python 3.10 开始）或`typing.get_type_hints`（自 Python 3.5 起）。

+   编写自己的自定义函数，作为`in​spect​.get_annotations`或`typing.get_type_hints`周围的薄包装，让您的代码库的其余部分调用该自定义函数，以便将来的更改局限于单个函数。

为了演示第二点，这里是在 示例 24-5 中定义的`Checked`类的前几行，我们将在 第二十四章 中学习：

```py
class Checked:
    @classmethod
    def _fields(cls) -> dict[str, type]:
        return get_type_hints(cls)
    # ... more lines ...
```

`Checked._fields` 类方法保护模块的其他部分不直接依赖于`typing.get_type_hints`。如果`get_type_hints`在将来发生变化，需要额外的逻辑，或者您想用`inspect.get_annotations`替换它，更改将局限于`Checked._fields`，不会影响程序的其余部分。

###### 警告

鉴于关于运行时检查类型提示的持续讨论和提出的更改，官方的[“注释最佳实践”](https://fpy.li/15-28)文档是必读的，并且可能会在通往 Python 3.11 的道路上进行更新。这篇指南是由 Larry Hastings 撰写的，他是 [PEP 649—使用描述符延迟评估注释](https://fpy.li/pep649) 的作者，这是一个解决由 [PEP 563—延迟评估注释](https://fpy.li/pep563) 提出的运行时问题的替代提案。

本章的其余部分涵盖了泛型，从如何定义一个可以由用户参数化的泛型类开始。

# 实现一个通用类

在 示例 13-7 中，我们定义了`Tombola` ABC：一个类似于宾果笼的接口。来自 示例 13-10 的`LottoBlower` 类是一个具体的实现。现在我们将研究一个通用版本的`LottoBlower`，就像在 示例 15-15 中使用的那样。

##### 示例 15-15\. generic_lotto_demo.py：使用通用抽奖机类

```py
from generic_lotto import LottoBlower

machine = LottoBlowerint)  # ①

first = machine.pick()  # ②
remain = machine.inspect()  # ③
```

①

要实例化一个通用类，我们给它一个实际的类型参数，比如这里的`int`。

②

Mypy 将正确推断`first`是一个`int`…

③

… 而`remain`是一个整数的元组。

此外，Mypy 还报告了参数化类型的违规情况，并提供了有用的消息，就像 示例 15-16 中显示的那样。

##### 示例 15-16\. generic_lotto_errors.py：Mypy 报告的错误

```py
from generic_lotto import LottoBlower

machine = LottoBlowerint
## error: List item 1 has incompatible type "float"; # ①
##        expected "int"

machine = LottoBlowerint)

machine.load('ABC')
## error: Argument 1 to "load" of "LottoBlower" # ②
##        has incompatible type "str";
##        expected "Iterable[int]"
## note:  Following member(s) of "str" have conflicts:
## note:      Expected:
## note:          def __iter__(self) -> Iterator[int]
## note:      Got:
## note:          def __iter__(self) -> Iterator[str]
```

①

在实例化`LottoBlower[int]`时，Mypy 标记了`float`。

②

在调用`.load('ABC')`时，Mypy 解释了为什么`str`不行：`str.__iter__`返回一个`Iterator[str]`，但`LottoBlower[int]`需要一个`Iterator[int]`。

示例 15-17 是实现。

##### 示例 15-17\. generic_lotto.py：一个通用的抽奖机类

```py
import random

from collections.abc import Iterable
from typing import TypeVar, Generic

from tombola import Tombola

T = TypeVar('T')

class LottoBlower(Tombola, Generic[T]):  # ①

    def __init__(self, items: Iterable[T]) -> None:  # ②
        self._balls = listT

    def load(self, items: Iterable[T]) -> None:  # ③
        self._balls.extend(items)

    def pick(self) -> T:  # ④
        try:
            position = random.randrange(len(self._balls))
        except ValueError:
            raise LookupError('pick from empty LottoBlower')
        return self._balls.pop(position)

    def loaded(self) -> bool:  # ⑤
        return bool(self._balls)

    def inspect(self) -> tuple[T, ...]:  # ⑥
        return tuple(self._balls)
```

①

泛型类声明通常使用多重继承，因为我们需要子类化`Generic`来声明形式类型参数——在本例中为`T`。

②

`__init__`中的`items`参数的类型为`Iterable[T]`，当实例声明为`LottoBlower[int]`时，变为`Iterable[int]`。

③

`load`方法也受到限制。

④

`T`的返回类型现在在`LottoBlower[int]`中变为`int`。

⑤

这里没有类型变量。

⑥

最后，`T`设置了返回的`tuple`中项目的类型。

###### 提示

`typing`模块文档中的[“用户定义的泛型类型”](https://fpy.li/15-29)部分很简短，提供了很好的例子，并提供了一些我这里没有涵盖的更多细节。

现在我们已经看到如何实现泛型类，让我们定义术语来谈论泛型。

## 泛型类型的基本术语

这里有几个我在学习泛型时发现有用的定义：¹⁴

泛型类型

声明有一个或多个类型变量的类型。

例子：`LottoBlower[T]`，`abc.Mapping[KT, VT]`

形式类型参数

出现在泛型类型声明中的类型变量。

例子：前面例子`abc.Mapping[KT, VT]`中的`KT`和`VT`

参数化类型

声明为具有实际类型参数的类型。

例子：`LottoBlower[int]`，`abc.Mapping[str, float]`

实际类型参数

在声明参数化类型时给定的实际类型。

例子：`LottoBlower[int]`中的`int`

下一个主题是如何使泛型类型更灵活，引入协变、逆变和不变的概念。

# 方差

###### 注意

根据您在其他语言中对泛型的经验，这可能是本书中最具挑战性的部分。方差的概念是抽象的，严谨的表述会使这一部分看起来像数学书中的页面。

在实践中，方差主要与想要支持新的泛型容器类型或提供基于回调的 API 的库作者有关。即使如此，通过仅支持不变容器，您可以避免许多复杂性——这基本上是我们现在在 Python 标准库中所拥有的。因此，在第一次阅读时，您可以跳过整个部分，或者只阅读关于不变类型的部分。

我们首次在“可调用类型的方差”中看到了*方差*的概念，应用于参数化泛型`Callable`类型。在这里，我们将扩展这个概念，涵盖泛型集合类型，使用“现实世界”的类比使这个抽象概念更具体。

想象一下学校食堂有一个规定，只能安装果汁分配器。只有果汁分配器是被允许的，因为它们可能提供被学校董事会禁止的苏打水。¹⁵¹⁶

## 不变的分配器

让我们尝试用一个可以根据饮料类型进行参数化的泛型`BeverageDispenser`类来模拟食堂场景。请参见例 15-18。

##### 例 15-18\. invariant.py：类型定义和`install`函数

```py
from typing import TypeVar, Generic

class Beverage:  # ①
    """Any beverage."""

class Juice(Beverage):
    """Any fruit juice."""

class OrangeJuice(Juice):
    """Delicious juice from Brazilian oranges."""

T = TypeVar('T')  # ②

class BeverageDispenser(Generic[T]):  # ③
    """A dispenser parameterized on the beverage type."""
    def __init__(self, beverage: T) -> None:
        self.beverage = beverage

    def dispense(self) -> T:
        return self.beverage

def install(dispenser: BeverageDispenser[Juice]) -> None:  # ④
    """Install a fruit juice dispenser."""
```

①

`Beverage`、`Juice`和`OrangeJuice`形成一个类型层次结构。

②

简单的`TypeVar`声明。

③

`BeverageDispenser`的类型参数化为饮料的类型。

④

`install`是一个模块全局函数。它的类型提示强制执行只有果汁分配器是可接受的规则。

鉴于例 15-18 中的定义，以下代码是合法的：

```py
juice_dispenser = BeverageDispenser(Juice())
install(juice_dispenser)
```

然而，这是不合法的：

```py
beverage_dispenser = BeverageDispenser(Beverage())
install(beverage_dispenser)
## mypy: Argument 1 to "install" has
## incompatible type "BeverageDispenser[Beverage]"
##          expected "BeverageDispenser[Juice]"
```

任何`饮料`的分配器都是不可接受的，因为食堂需要专门用于`果汁`的分配器。

令人惊讶的是，这段代码也是非法的：

```py
orange_juice_dispenser = BeverageDispenser(OrangeJuice())
install(orange_juice_dispenser)
## mypy: Argument 1 to "install" has
## incompatible type "BeverageDispenser[OrangeJuice]"
##          expected "BeverageDispenser[Juice]"
```

专门用于`橙汁`的分配器也是不允许的。只有`BeverageDispenser[Juice]`才行。在类型术语中，我们说`BeverageDispenser(Generic[T])`是不变的，当`BeverageDispenser[OrangeJuice]`与`BeverageDispenser[Juice]`不兼容时——尽管`OrangeJuice`是`Juice`的*子类型*。

Python 可变集合类型——如`list`和`set`——是不变的。来自示例 15-17 的`LottoBlower`类也是不变的。

## 一个协变分配器

如果我们想更灵活地建模分配器作为一个通用类，可以接受某种饮料类型及其子类型，我们必须使其协变。示例 15-19 展示了如何声明`BeverageDispenser`。

##### 示例 15-19\. *covariant.py*：类型定义和`install`函数

```py
T_co = TypeVar('T_co', covariant=True)  # ①

class BeverageDispenser(Generic[T_co]):  # ②
    def __init__(self, beverage: T_co) -> None:
        self.beverage = beverage

    def dispense(self) -> T_co:
        return self.beverage

def install(dispenser: BeverageDispenser[Juice]) -> None:  # ③
    """Install a fruit juice dispenser."""
```

①

在声明类型变量时，设置`covariant=True`；`_co`是*typeshed*上协变类型参数的常规后缀。

②

使用`T_co`来为`Generic`特殊类进行参数化。

③

对于`install`的类型提示与示例 15-18 中的相同。

以下代码有效，因为现在`Juice`分配器和`OrangeJuice`分配器都在协变`BeverageDispenser`中有效：

```py
juice_dispenser = BeverageDispenser(Juice())
install(juice_dispenser)

orange_juice_dispenser = BeverageDispenser(OrangeJuice())
install(orange_juice_dispenser)
```

但是，任意`饮料`的分配器也是不可接受的：

```py
beverage_dispenser = BeverageDispenser(Beverage())
install(beverage_dispenser)
## mypy: Argument 1 to "install" has
## incompatible type "BeverageDispenser[Beverage]"
##          expected "BeverageDispenser[Juice]"
```

这就是协变性：参数化分配器的子类型关系与类型参数的子类型关系方向相同变化。

## 逆变垃圾桶

现在我们将模拟食堂设置垃圾桶的规则。让我们假设食物和饮料都是用生物降解包装，剩菜剩饭以及一次性餐具也是生物降解的。垃圾桶必须适用于生物降解的废物。

###### 注意

为了这个教学示例，让我们做出简化假设，将垃圾分类为一个整洁的层次结构：

+   `废物`是最一般的垃圾类型。所有垃圾都是废物。

+   `生物降解`是一种可以随时间被生物分解的垃圾类型。一些`废物`不是`生物降解`的。

+   `可堆肥`是一种特定类型的`生物降解`垃圾，可以在堆肥桶或堆肥设施中高效地转化为有机肥料。在我们的定义中，并非所有`生物降解`垃圾都是`可堆肥`的。

为了模拟食堂中可接受垃圾桶的规则，我们需要通过一个示例引入“逆变性”概念，如示例 15-20 所示。

##### 示例 15-20\. *contravariant.py*：类型定义和`install`函数

```py
from typing import TypeVar, Generic

class Refuse:  # ①
    """Any refuse."""

class Biodegradable(Refuse):
    """Biodegradable refuse."""

class Compostable(Biodegradable):
    """Compostable refuse."""

T_contra = TypeVar('T_contra', contravariant=True)  # ②

class TrashCan(Generic[T_contra]):  # ③
    def put(self, refuse: T_contra) -> None:
        """Store trash until dumped."""

def deploy(trash_can: TrashCan[Biodegradable]):
    """Deploy a trash can for biodegradable refuse."""
```

①

垃圾的类型层次结构：`废物`是最一般的类型，`可堆肥`是最具体的。

②

`T_contra`是逆变类型变量的常规名称。

③

`TrashCan`在废物类型上是逆变的。

根据这些定义，以下类型的垃圾桶是可接受的：

```py
bio_can: TrashCan[Biodegradable] = TrashCan()
deploy(bio_can)

trash_can: TrashCan[Refuse] = TrashCan()
deploy(trash_can)
```

更一般的`TrashCan[Refuse]`是可接受的，因为它可以接受任何类型的废物，包括`生物降解`。然而，`TrashCan[Compostable]`不行，因为它不能接受`生物降解`：

```py
compost_can: TrashCan[Compostable] = TrashCan()
deploy(compost_can)
## mypy: Argument 1 to "deploy" has
## incompatible type "TrashCan[Compostable]"
##          expected "TrashCan[Biodegradable]"
```

让我们总结一下我们刚刚看到的概念。

## 变异回顾

变异是一个微妙的属性。以下部分总结了不变、协变和逆变类型的概念，并提供了一些关于它们推理的经验法则。

### 不变类型

当两个参数化类型之间没有超类型或子类型关系时，泛型类型 `L` 是不变的，而不管实际参数之间可能存在的关系。换句话说，如果 `L` 是不变的，那么 `L[A]` 不是 `L[B]` 的超类型或子类型。它们在两个方面都不一致。

如前所述，Python 的可变集合默认是不变的。`list` 类型是一个很好的例子：`list[int]` 与 `list[float]` 不一致，反之亦然。

一般来说，如果一个形式类型参数出现在方法参数的类型提示中，并且相同的参数出现在方法返回类型中，那么为了确保在更新和读取集合时的类型安全，该参数必须是不变的。

例如，这是 `list` 内置的类型提示的一部分[*typeshed*](https://fpy.li/15-30)：

```py
class list(MutableSequence[_T], Generic[_T]):
    @overload
    def __init__(self) -> None: ...
    @overload
    def __init__(self, iterable: Iterable[_T]) -> None: ...
    # ... lines omitted ...
    def append(self, __object: _T) -> None: ...
    def extend(self, __iterable: Iterable[_T]) -> None: ...
    def pop(self, __index: int = ...) -> _T: ...
    # etc...
```

注意 `_T` 出现在 `__init__`、`append` 和 `extend` 的参数中，以及 `pop` 的返回类型中。如果 `_T` 在 `_T` 中是协变或逆变的，那么没有办法使这样的类类型安全。

### 协变类型

考虑两种类型 `A` 和 `B`，其中 `B` 与 `A` 一致，且它们都不是 `Any`。一些作者使用 `<:` 和 `:>` 符号来表示这样的类型关系：

`A :> B`

`A` 是 `B` 的超类型或相同。

`B <: A`

`B` 是 `A` 的子类型或相同。

给定 `A :> B`，泛型类型 `C` 在 `C[A] :> C[B]` 时是协变的。

注意 `:>` 符号的方向在 `A` 在 `B` 的左侧时是相同的。协变泛型类型遵循实际类型参数的子类型关系。

不可变容器可以是协变的。例如，`typing.FrozenSet` 类是如何 [文档化](https://fpy.li/15-31) 作为一个协变的，使用传统名称 `T_co` 的类型变量：

```py
class FrozenSet(frozenset, AbstractSet[T_co]):
```

将 `:>` 符号应用于参数化类型，我们有：

```py
           float :> int
frozenset[float] :> frozenset[int]
```

迭代器是协变泛型的另一个例子：它们不是只读集合，如 `frozenset`，但它们只产生输出。任何期望一个产生浮点数的 `abc.Iterator[float]` 的代码可以安全地使用一个产生整数的 `abc.Iterator[int]`。`Callable` 类型在返回类型上是协变的，原因类似。

### 逆变类型

给定 `A :> B`，泛型类型 `K` 在 `K[A] <: K[B]` 时是逆变的。

逆变泛型类型颠倒了实际类型参数的子类型关系。

`TrashCan` 类是一个例子：

```py
          Refuse :> Biodegradable
TrashCan[Refuse] <: TrashCan[Biodegradable]
```

逆变容器通常是一个只写数据结构，也称为“接收器”。标准库中没有这样的集合的例子，但有一些具有逆变类型参数的类型。

`Callable[[ParamType, …], ReturnType]` 在参数类型上是逆变的，但在 `ReturnType` 上是协变的，正如我们在 “Callable 类型的方差” 中看到的。此外，[`Generator`](https://fpy.li/15-32)、[`Coroutine`](https://fpy.li/typecoro) 和 [`AsyncGenerator`](https://fpy.li/15-33) 有一个逆变类型参数。`Generator` 类型在 “经典协程的泛型类型提示” 中有描述；`Coroutine` 和 `AsyncGenerator` 在 第二十一章 中有描述。

对于关于方差的讨论，主要观点是逆变的形式参数定义了用于调用或发送数据到对象的参数类型，而不同的协变形式参数定义了对象产生的输出类型——产生类型或返回类型，取决于对象。 “发送” 和 “产出” 的含义在 “经典协程” 中有解释。

我们可以从这些关于协变输出和逆变输入的观察中得出有用的指导方针。

### 协变的经验法则

最后，以下是一些关于推理方差时的经验法则：

+   如果一个形式类型参数定义了从对象中输出的数据类型，那么它可以是协变的。

+   如果形式类型参数定义了一个类型，用于在对象初始构建后进入对象的数据，它可以是逆变的。

+   如果形式类型参数定义了一个用于从对象中提取数据的类型，并且同一参数定义了一个用于将数据输入对象的类型，则它必须是不变的。

+   为了保险起见，使形式类型参数不变。

`Callable[[ParamType, …], ReturnType]`展示了规则#1 和#2：`ReturnType`是协变的，而每个`ParamType`是逆变的。

默认情况下，`TypeVar`创建的形式参数是不变的，这就是标准库中的可变集合是如何注释的。

“经典协程的通用类型提示”继续讨论关于方差的内容。

接下来，让我们看看如何定义通用的静态协议，将协变的思想应用到几个新的示例中。

# 实现通用的静态协议

Python 3.10 标准库提供了一些通用的静态协议。其中之一是`SupportsAbs`，在[typing 模块](https://fpy.li/15-34)中实现如下：

```py
@runtime_checkable
class SupportsAbs(Protocol[T_co]):
    """An ABC with one abstract method __abs__ that is covariant in its
 return type."""
    __slots__ = ()

    @abstractmethod
    def __abs__(self) -> T_co:
        pass
```

`T_co`根据命名约定声明：

```py
T_co = TypeVar('T_co', covariant=True)
```

由于`SupportsAbs`，Mypy 将此代码识别为有效，如您在示例 15-21 中所见。

##### 示例 15-21。*abs_demo.py*：使用通用的`SupportsAbs`协议

```py
import math
from typing import NamedTuple, SupportsAbs

class Vector2d(NamedTuple):
    x: float
    y: float

    def __abs__(self) -> float:  # ①
        return math.hypot(self.x, self.y)

def is_unit(v: SupportsAbs[float]) -> bool:  # ②
    """'True' if the magnitude of 'v' is close to 1."""
    return math.isclose(abs(v), 1.0)  # ③

assert issubclass(Vector2d, SupportsAbs)  # ④

v0 = Vector2d(0, 1)  # ⑤
sqrt2 = math.sqrt(2)
v1 = Vector2d(sqrt2 / 2, sqrt2 / 2)
v2 = Vector2d(1, 1)
v3 = complex(.5, math.sqrt(3) / 2)
v4 = 1  # ⑥

assert is_unit(v0)
assert is_unit(v1)
assert not is_unit(v2)
assert is_unit(v3)
assert is_unit(v4)

print('OK')
```

①

定义`__abs__`使`Vector2d`与`SupportsAbs`*一致*。

②

使用`float`参数化`SupportsAbs`确保…

③

…Mypy 接受`abs(v)`作为`math.isclose`的第一个参数。

④

在`SupportsAbs`的定义中，感谢`@runtime_checkable`，这是一个有效的运行时断言。

⑤

剩下的代码都通过了 Mypy 检查和运行时断言。

⑥

`int`类型也与`SupportsAbs`*一致*。根据[*typeshed*](https://fpy.li/15-35)，`int.__abs__`返回一个`int`，这与`is_unit`类型提示中为`v`参数声明的`float`类型参数*一致*。

类似地，我们可以编写`RandomPicker`协议的通用版本，该协议在示例 13-18 中介绍，该协议定义了一个返回`Any`的单个方法`pick`。

示例 15-22 展示了如何使通用的`RandomPicker`在`pick`的返回类型上具有协变性。

##### 示例 15-22。*generic_randompick.py*：定义通用的`RandomPicker`

```py
from typing import Protocol, runtime_checkable, TypeVar

T_co = TypeVar('T_co', covariant=True)  # ①

@runtime_checkable
class RandomPicker(Protocol[T_co]):  # ②
    def pick(self) -> T_co: ...  # ③
```

①

将`T_co`声明为`协变`。

②

这使`RandomPicker`具有协变的形式类型参数。

③

使用`T_co`作为返回类型。

通用的`RandomPicker`协议可以是协变的，因为它的唯一形式参数用于返回类型。

有了这个，我们可以称之为一个章节。

# 章节总结

章节以一个简单的使用`@overload`的例子开始，接着是一个我们详细研究的更复杂的例子：正确注释`max`内置函数所需的重载签名。

接下来是`typing.TypedDict`特殊构造。我选择在这里介绍它，而不是在第五章中看到`typing.NamedTuple`，因为`TypedDict`不是一个类构建器；它只是一种向需要具有特定一组字符串键和每个键特定类型的`dict`添加类型提示的方式——当我们将`dict`用作记录时，通常在处理 JSON 数据时会发生这种情况。该部分有点长，因为使用`TypedDict`可能会给人一种虚假的安全感，我想展示在尝试将静态结构化记录转换为本质上是动态的映射时，运行时检查和错误处理是不可避免的。

接下来我们讨论了`typing.cast`，这是一个旨在指导类型检查器工作的函数。仔细考虑何时使用`cast`很重要，因为过度使用会妨碍类型检查器。

接下来是运行时访问类型提示。关键点是使用`typing.​get_type_hints`而不是直接读取`__annotations__`属性。然而，该函数可能对某些注解不可靠，我们看到 Python 核心开发人员仍在努力找到一种方法，在减少对 CPU 和内存使用的影响的同时使类型提示在运行时可用。

最后几节是关于泛型的，首先是`LottoBlower`泛型类——我们后来了解到它是一个不变的泛型类。该示例后面是四个基本术语的定义：泛型类型、形式类型参数、参数化类型和实际类型参数。

接下来介绍了主题的主要内容，使用自助餐厅饮料分配器和垃圾桶作为不变、协变和逆变通用类型的“现实生活”示例。接下来，我们对 Python 标准库中的示例进行了复习、形式化和进一步应用这些概念。

最后，我们看到了如何定义通用的静态协议，首先考虑`typing.SupportsAbs`协议，然后将相同的思想应用于`RandomPicker`示例，使其比第十三章中的原始协议更加严格。

###### 注意

Python 的类型系统是一个庞大且快速发展的主题。本章不是全面的。我选择关注那些广泛适用、特别具有挑战性或在概念上重要且因此可能长期相关的主题。

# 进一步阅读

Python 的静态类型系统最初设计复杂，随着每年的发展变得更加复杂。表 15-1 列出了截至 2021 年 5 月我所知道的所有 PEP。要覆盖所有内容需要一整本书。

表 15-1。关于类型提示的 PEP，标题中带有链接。带有*号的 PEP 编号在[`typing`文档](https://fpy.li/typing)的开头段落中提到。Python 列中的问号表示正在讨论或尚未实施的 PEP；“n/a”出现在没有特定 Python 版本的信息性 PEP 中。

| PEP | 标题 | Python | 年份 |
| --- | --- | --- | --- |
| 3107 | [函数注解](https://fpy.li/pep3107) | 3.0 | 2006 |
| 483* | [类型提示理论](https://fpy.li/pep483) | n/a | 2014 |
| 484* | [类型提示](https://fpy.li/pep484) | 3.5 | 2014 |
| 482 | [类型提示文献综述](https://fpy.li/pep482) | n/a | 2015 |
| 526* | [变量注解的语法](https://fpy.li/pep526) | 3.6 | 2016 |
| 544* | [协议：结构子类型（静态鸭子类型）](https://fpy.li/pep544) | 3.8 | 2017 |
| 557 | [数据类](https://fpy.li/pep557) | 3.7 | 2017 |
| 560 | [类型模块和泛型类型的核心支持](https://fpy.li/pep560) | 3.7 | 2017 |
| 561 | [分发和打包类型信息](https://fpy.li/pep561) | 3.7 | 2017 |
| 563 | [注解的延迟评估](https://fpy.li/pep563) | 3.7 | 2017 |
| 586* | [字面类型](https://fpy.li/pep586) | 3.8 | 2018 |
| 585 | [标准集合中的泛型类型提示](https://fpy.li/pep585) | 3.9 | 2019 |
| 589* | [TypedDict：具有固定键集的字典的类型提示](https://fpy.li/pep589) | 3.8 | 2019 |
| 591* | [向 typing 添加 final 修饰符](https://fpy.li/pep591) | 3.8 | 2019 |
| 593 | [灵活的函数和变量注释](https://fpy.li/pep593) | ? | 2019 |
| 604 | [将联合类型写为 X &#124; Y](https://fpy.li/pep604) | 3.10 | 2019 |
| 612 | [参数规范变量](https://fpy.li/pep612) | 3.10 | 2019 |
| 613 | [显式类型别名](https://fpy.li/pep613) | 3.10 | 2020 |
| 645 | [允许将可选类型写为 x?](https://fpy.li/pep645) | ? | 2020 |
| 646 | [可变泛型](https://fpy.li/pep646) | ? | 2020 |
| 647 | [用户定义的类型守卫](https://fpy.li/pep647) | 3.10 | 2021 |
| 649 | [使用描述符延迟评估注释](https://fpy.li/pep649) | ? | 2021 |
| 655 | [将个别 TypedDict 项目标记为必需或可能缺失](https://fpy.li/pep655) | ? | 2021 |

Python 的官方文档几乎无法跟上所有内容，因此[Mypy 的文档](https://fpy.li/mypy)是一个必不可少的参考。[*强大的 Python*](https://fpy.li/15-36) 作者：帕特里克·维亚福雷（O’Reilly）是我知道的第一本广泛涵盖 Python 静态类型系统的书籍，于 2021 年 8 月出版。你现在可能正在阅读第二本这样的书籍。

关于协变的微妙主题在 PEP 484 的[章节中](https://fpy.li/15-37)有专门讨论，同时也在 Mypy 的[“泛型”](https://fpy.li/15-38)页面以及其宝贵的[“常见问题”](https://fpy.li/15-39)页面中有涵盖。

阅读值得的[PEP 362—函数签名对象](https://fpy.li/pep362)，如果你打算使用补充`typing.get_type_hints`函数的`inspect`模块。

如果你对 Python 的历史感兴趣，你可能会喜欢知道，Guido van Rossum 在 2004 年 12 月 23 日发布了[“向 Python 添加可选静态类型”](https://fpy.li/15-40)。

[“Python 3 中的类型在野外：两种类型系统的故事”](https://fpy.li/15-41) 是由 Rensselaer Polytechnic Institute 和 IBM TJ Watson 研究中心的 Ingkarat Rak-amnouykit 等人撰写的研究论文。该论文调查了 GitHub 上开源项目中类型提示的使用情况，显示大多数项目并未使用它们，而且大多数具有类型提示的项目显然也没有使用类型检查器。我发现最有趣的是对 Mypy 和 Google 的 *pytype* 不同语义的讨论，他们得出结论称它们“本质上是两种不同的类型系统”。

两篇关于渐进式类型的重要论文是吉拉德·布拉查的[“可插入式类型系统”](https://fpy.li/15-42)，以及埃里克·迈杰和彼得·德雷顿撰写的[“可能时使用静态类型，需要时使用动态类型：编程语言之间的冷战结束”](https://fpy.li/15-43)¹⁷

通过阅读其他语言实现相同思想的一些书籍的相关部分，我学到了很多：

+   [*原子 Kotlin*](https://fpy.li/15-44) 作者：布鲁斯·埃克尔和斯维特兰娜·伊萨科娃（Mindview）

+   [*Effective Java*，第三版](https://fpy.li/15-45) 作者：乔舒亚·布洛克（Addison-Wesley）

+   [*使用类型编程：TypeScript 示例*](https://fpy.li/15-46) 作者：弗拉德·里斯库蒂亚（Manning）

+   [*编程 TypeScript*](https://fpy.li/15-47) 作者：鲍里斯·切尔尼（O’Reilly）

+   [*Dart 编程语言*](https://fpy.li/15-48) 作者：吉拉德·布拉查（Addison-Wesley）¹⁸

对于一些关于类型系统的批判观点，我推荐阅读维克多·尤代肯的文章[“类型理论中的坏主意”](https://fpy.li/15-49)和[“类型有害 II”](https://fpy.li/15-50)。

最后，我惊讶地发现了 Ken Arnold 的[“泛型有害论”](https://fpy.li/15-51)，他是 Java 的核心贡献者，也是官方*Java 编程语言*书籍（Addison-Wesley）前四版的合著者之一——与 Java 的首席设计师 James Gosling 合作。

遗憾的是，Arnold 的批评也适用于 Python 的静态类型系统。在阅读许多有关类型提示 PEP 的规则和特例时，我不断想起 Gosling 文章中的这段话：

> 这就提出了我总是为 C++引用的问题：我称之为“例外规则的 N^(th)次例外”。听起来是这样的：“你可以做 x，但在情况 y 下除外，除非 y 做 z，那么你可以如果...”

幸运的是，Python 比 Java 和 C++有一个关键优势：可选的类型系统。当类型提示变得太繁琐时，我们可以关闭类型检查器并省略类型提示。

¹ 来自 YouTube 视频“语言创作者对话：Guido van Rossum、James Gosling、Larry Wall 和 Anders Hejlsberg”，于 2019 年 4 月 2 日直播。引用开始于[1:32:05](https://fpy.li/15-1)，经过简化编辑。完整的文字记录可在[*https://github.com/fluentpython/language-creators*](https://github.com/fluentpython/language-creators)找到。

² 我要感谢 Jelle Zijlstra——一个*typeshed*的维护者——教会了我很多东西，包括如何将我最初的九个重载减少到六个。

³ 截至 2020 年 5 月，pytype 允许这样做。但其[常见问题解答](https://fpy.li/15-6)中表示将来会禁止这样做。请参见 pytype[常见问题解答](https://fpy.li/15-6)中的“为什么 pytype 没有捕捉到我更改了已注释变量的类型？”问题。

⁴ 我更喜欢使用[lxml](https://fpy.li/15-8)包来生成和解析 XML：它易于上手，功能齐全且速度快。不幸的是，lxml 和 Python 自带的[*ElementTree*](https://fpy.li/15-9)不适用于我假想的微控制器的有限 RAM。

⁵ Mypy 文档在其[“常见问题和解决方案”页面](https://fpy.li/15-10)中讨论了这个问题，在“空集合的类型”一节中有详细说明。

⁶ Brett Cannon、Guido van Rossum 等人自 2016 年以来一直在讨论如何为`json.loads()`添加类型提示，在[Mypy 问题＃182：定义 JSON 类型](https://fpy.li/15-12)中。

⁷ 示例中使用`enumerate`旨在混淆类型检查器。Mypy 可以正确分析直接生成字符串而不经过`enumerate`索引的更简单的实现，因此不需要`cast()`。

⁸ 我报告了*typeshed*的[问题＃5535](https://fpy.li/15-17)，“asyncio.base_events.Server sockets 属性的错误类型提示”，Sebastian Rittau 很快就修复了。然而，我决定保留这个例子，因为它展示了`cast`的一个常见用例，而我写的`cast`是无害的。

⁹ 老实说，我最初在带有`server.sockets[0]`的行末添加了一个`# type: ignore`注释，因为经过一番调查，我在*asyncio* [文档](https://fpy.li/15-18)和一个[测试用例](https://fpy.li/15-19)中找到了类似的行，所以我怀疑问题不在我的代码中。

¹⁰ [2020 年 5 月 19 日消息](https://fpy.li/15-21)发送至 typing-sig 邮件列表。

¹¹ 语法`# type: ignore[code]`允许您指定要消除的 Mypy 错误代码，但这些代码并不总是容易解释。请参阅 Mypy 文档中的[“错误代码”](https://fpy.li/15-22)。

¹² 我不会详细介绍 `clip` 的实现，但如果你感兴趣，可以阅读 [*clip_annot.py*](https://fpy.li/15-23) 中的整个模块。

¹³ 2021 年 4 月 16 日发布的信息 [“PEP 563 in light of PEP 649”](https://fpy.li/15-27)。

¹⁴ 这些术语来自 Joshua Bloch 的经典著作 *Effective Java*，第三版（Addison-Wesley）。定义和示例是我自己的。

¹⁵ 我第一次看到 Erik Meijer 在 Gilad Bracha 的 *The Dart Programming Language* 一书（Addison-Wesley）的 *前言* 中使用自助餐厅类比来解释方差。

¹⁶ 比禁书好多了！

¹⁷ 作为脚注的读者，你可能记得我将 Erik Meijer 归功于用自助餐厅类比来解释方差。

¹⁸ 那本书是为 Dart 1 写的。Dart 2 有重大变化，包括类型系统。尽管如此，Bracha 是编程语言设计领域的重要研究者，我发现这本书对 Dart 的设计视角很有价值。

¹⁹ 参见 PEP 484 中 [“Covariance and Contravariance”](https://fpy.li/15-37) 部分的最后一段。
