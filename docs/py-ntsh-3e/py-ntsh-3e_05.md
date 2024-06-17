# 第五章. 类型注释

使用类型信息对 Python 代码进行注释是一个可选步骤，在开发和维护大型项目或库时可能非常有用。静态类型检查器和 lint 工具可帮助识别和定位函数参数和返回值的数据类型不匹配。IDE 可以使用这些*类型注释*（也称为*类型提示*）来改进自动完成，并提供弹出式文档。第三方软件包和框架可以使用类型注释来定制运行时行为，或者根据方法和变量的类型注释自动生成代码。

Python 中的类型注释和检查仍在不断发展，并涉及许多复杂的问题。本章涵盖了类型注释的一些最常见用例；您可以在本章末尾列出的资源中找到更全面的资料。

# Python 版本的类型注释支持因版本而异

支持类型注释的 Python 功能已经从一个版本发展到另一个版本，其中包括一些重大的增加和删除。本章的其余部分将描述 Python 最新版本（3.10 及更高版本）中的类型注释支持，其中包含一些可能在其他版本中存在或不存在的功能的注释。

# 历史

Python 本质上是一种*动态类型*语言。这使您能够通过命名和使用变量来快速开发代码，而无需声明它们。动态类型允许使用灵活的编码习惯、通用容器和多态数据处理，而无需显式定义接口类型或类层次结构。缺点是，在开发过程中，语言无法在传递给函数或从函数返回的不兼容类型的变量上提供帮助。Python 不像一些语言那样利用开发时编译步骤来检测和报告数据类型问题，而是依靠开发人员通过一系列测试用例在运行时环境中重建来发现数据类型错误。

# 类型注释不是强制性的

类型注释在运行时*不*被强制执行。Python 不执行任何基于类型注释的类型验证或数据转换；可执行的 Python 代码仍然负责正确使用变量和函数参数。但是，类型注释必须在语法上是正确的。包含无效类型注释的延迟导入或动态导入模块会在运行中的 Python 程序中引发 SyntaxError 异常，就像任何无效的 Python 语句一样。

历史上，Python 缺乏任何类型检查被认为是其短板之一，一些程序员因此选择其他编程语言。然而，社区希望 Python 保持其运行时类型自由，因此逻辑上的做法是增加对由类似 lint 工具（在下一节进一步描述）和 IDE 执行的静态类型检查的支持。一些尝试是基于解析函数签名或文档字符串进行类型检查。Guido van Rossum 在[Python 开发者邮件列表](https://oreil.ly/GFMBC)上引用了几个案例，显示类型注解可以帮助，例如在维护大型遗留代码库时。使用注解语法，开发工具可以执行静态类型检查，以突出显示与预期类型冲突的变量和函数使用。

类型注解的第一个官方版本使用特殊格式的注释来指示变量类型和返回代码，如[PEP 484](https://oreil.ly/61GSZ)所定义的，这是 Python 3.5 的一项临时 PEP。² 使用注释可以快速实现和尝试新的类型语法，而无需修改 Python 编译器本身。³ 第三方包[mypy](http://mypy-lang.org)通过使用这些注释进行静态类型检查得到了广泛接受。随着 Python 3.6 采纳了[PEP 526](https://oreil.ly/S8kI3)，类型注解已完全整合到 Python 语言本身，并在标准库中添加了一个支持的 typing 模块。

# 类型检查工具

随着类型注解成为 Python 的一个已确立部分，类型检查工具和 IDE 插件也成为 Python 生态系统的一部分。

## mypy

独立的[mypy](https://oreil.ly/6fMPM)实用程序继续作为静态类型检查的主要工具，始终与 Python 类型注解形式的演变保持最新状态（只要考虑 Python 版本！）。mypy 还作为插件提供给编辑器，包括 Vim、Emacs 和 SublimeText，以及 Atom、PyCharm 和 VS Code IDE。 （PyCharm、VS Code 和 Wing IDE 还单独包含了自己的类型检查功能，与 mypy 分开）。运行 mypy 的最常见命令只是**mypy my_python_script.py**。

您可以在[mypy 在线文档](https://oreil.ly/rQPK0)中找到更详细的用法示例和命令行选项，以及一个作为便捷参考的[速查表](https://oreil.ly/CT6FE)。本节后面的代码示例将包含 mypy 错误消息示例，以说明可以通过类型检查捕获的 Python 错误类型。

## 其他类型检查器

其他考虑使用的类型检查器包括：

MonkeyType

Instagram 的 [MonkeyType](https://oreil.ly/RHqNo) 使用 sys.setprofile 钩子在运行时动态检测类型；像 pytype 一样（见下文），它也可以生成 *.pyi*（存根）文件，而不是或者除了在 Python 代码文件中插入类型注解。

pydantic

[pydantic](https://oreil.ly/-zNQj) 也可以在运行时工作，但不生成存根或插入类型注解；它的主要目标是解析输入并确保 Python 代码获得干净的数据。正如[在线文档](https://oreil.ly/0Ucvm)中所述，它还允许您扩展其验证功能以适应自己的环境。参见“FastAPI”的简单示例。

Pylance

[Pylance](https://oreil.ly/uB5XN) 是一个类型检查模块，主要用于将 Pyright（见下文）嵌入到 VS Code 中。

Pyre

Facebook 的 [Pyre](https://oreil.ly/HJ-qQ) 也可以生成 *.pyi* 文件。目前在 Windows 上无法运行，除非安装了 [Windows Subsystem for Linux (WSL)](https://oreil.ly/DwB82)。

Pyright

[Pyright](https://oreil.ly/wwuA8) 是微软的静态类型检查工具，作为命令行实用程序和 VS Code 扩展提供。

pytype

[pytype](https://oreil.ly/QuhCB) 是谷歌的静态类型检查器，专注于*类型推断*（即使在没有类型提示的情况下也能提供建议），除了类型注解。类型推断提供了强大的能力，即使在没有注解的代码中也能检测类型错误。pytype 还可以生成 *.pyi* 文件，并将存根文件合并回 *.py* 源代码中（最新版本的 mypy 也在效仿）。目前，pytype 在 Windows 上无法运行，除非你首先安装 [WSL](https://oreil.ly/7G_j-)。

多个主要软件组织开发的类型检查应用的出现，证明了 Python 开发者社区在使用类型注解方面的广泛兴趣。

# 类型注解语法

*类型注解* 在 Python 中使用以下形式指定：

```py
*`identifier`*: *`type_specification`*
```

*type_specification* 可以是任何 Python 表达式，但通常涉及一个或多个内置类型（例如，仅提到 Python 类型就是一个完全有效的表达式）和/或从 typing 模块导入的属性（在下一节中讨论）。典型的形式是：

```py
*`type_specifier`*[*`type_parameter`*, ...]
```

这里是一些作为变量类型注解使用的类型表达式示例：

```py
`import` typing

*`# an int`*
count: int

*`# a list of ints, with a default value`*
counts: list[int] = []

*`# a dict with str keys, values are tuples containing 2 ints and a str`*
employee_data: dict[str, tuple[int, int, str]]

*`# a callable taking a single str or bytes argument and returning a bool`*
str_predicate_function: typing.Callable[[str | bytes], bool]

*`# a dict with str keys, whose values are functions that take and return`* 
*`# an int`*
str_function_map: dict[str, typing.Callable[[int], int]] = {
    'square': `lambda` x: x * x,
    'cube': `lambda` x: x * x * x,
}
```

请注意，**lambda** 不接受类型注解。

要为函数添加返回类型注解，请使用以下形式：

```py
`def` identifier(argument, ...) -> type_specification :
```

每个 *参数* 的形式如下：

```py
*`identifier`*[: *`type_specification`*[ = *`default_value`*]]
```

这是一个带有注解函数的示例：

```py
`def` pad(a: list[str], min_len: int = 1, padstr: str = ' ') -> list[str]:
    *`"""Given a list of strings and a minimum length, return a copy of`*
 *`the list extended with "padding" strings to be at least the`*
 *`minimum length.`*
 *`"""`*
    `return` a + ([padstr] * (min_len - len(a)))
```

注意，当带有默认值的注解参数时，PEP 8 建议在等号周围使用空格。

# 还未完全定义的前向引用类型

有时，函数或变量定义需要引用尚未定义的类型。这在类方法或必须定义当前类类型的参数或返回值的方法中非常常见。这些函数签名在编译时解析，此时类型尚未定义。例如，此类方法无法编译通过：

```py
`class` A:
    @classmethod
    `def` factory_method(cls) -> A:
        *`# ... method body goes here ...`*
```

由于 Python 编译 `factory_method` 时，类 A 尚未定义，因此代码会引发 NameError 错误。

问题可以通过在类型 A 的返回类型中添加引号来解决：

```py
`class` A:
    @classmethod
    `def` factory_method(cls) -> 'A':
        *`# ... method body goes here ...`*
```

未来版本的 Python 可能会推迟对类型注解的评估，直到运行时，从而使封闭引号变得不必要（Python 的指导委员会正在评估各种可能性）。您可以使用 **from** __future__ **import** annotations 预览此行为。

# typing 模块

typing 模块支持类型提示。它包含在创建类型注释时有用的定义，包括：

+   用于定义类型的类和函数

+   用于修改类型表达式的类和函数

+   抽象基类（ABCs）

+   协议

+   实用程序和装饰器

+   用于定义自定义类型的类

## 类型

typing 模块最初的实现包括对应于 Python 内置容器和其他类型的类型定义，以及标准库模块中的类型。许多这些类型已被弃用（见下文），但某些仍然有用，因为它们不直接对应任何 Python 内置类型。 Table 5-1 列出了在 Python 3.9 及更高版本中仍然有用的 typing 类型。

表 5-1\. typing 模块中有用的定义

| Type | 描述 |
| --- | --- |
| Any | 匹配任何类型。 |
| AnyStr | 等效于 str &#124; bytes。AnyStr 用于注释函数参数和返回类型，其中任一字符串类型都可以接受，但不应在多个参数之间混合使用，或者在参数和返回类型之间混合使用。 |
| BinaryIO | 匹配具有二进制（bytes）内容的流，例如使用 mode='b' 打开的流或 io.BytesIO。 |
| Callable | Callable[[*argument_type*, ...], *return_type*] 定义可调用对象的类型签名。接受与可调用对象的参数对应的类型列表，以及函数返回值的类型。如果可调用对象不接受任何参数，请使用空列表 []。如果可调用对象没有返回值，请使用 **None** 作为 *return_type*。 |
| IO | 等效于 BinaryIO &#124; TextIO。 |
| Lit⁠e⁠r⁠a⁠l​[*⁠e⁠x⁠p⁠ression*,...] | 3.8+ 指定变量可能采用的有效值列表。 |
| LiteralString | 3.11+ 指定必须实现为文字引号值的 str。用于防止代码易受到注入攻击。 |
| NoReturn | 用作“永久运行”函数的返回类型，比如调用 http.serve_forever 或 event_loop.run_forever 而没有返回值的情况。这不适用于简单返回无明确值的函数；对于这种情况，请使用 -> **None**。更多有关返回类型的讨论详见“为现有代码添加类型注解（逐步类型化）”。 |
| Self | 3.11+ 用作实例函数返回类型，**返回** self（以及其他少数情况，详见[PEP 673](https://oreil.ly/NMMaw)）。 |
| TextIO | 匹配文本流（str 类型内容），比如使用 mode='t' 打开的文件或者 io.StringIO 返回的对象。 |

-3.9 在 Python 3.9 之前，typing 模块的定义用于创建表示内置类型的类型，例如 List[int] 表示整数列表。从 Python 3.9 开始，这些名称已弃用，因为其对应的内置或标准库类型现在支持 [] 语法：整数列表现在简单地使用 list[int] 类型声明。Table 5-2 列出了在 Python 3.9 之前使用 typing 模块进行类型注解时必要的定义。

Table 5-2\. Python 内置类型及其在 typing 模块中 3.9 之前的定义

| 内置类型 | Python 3.9 前的 typing 模块等效类型 |
| --- | --- |
| dict | Dict |
| frozenset | FrozenSet |
| list | List |
| set | Set |
| str | Text |
| tuple | Tuple |
| type | Type |
| collections.ChainMap | ChainMap |
| collections.Counter | Counter |
| collections.defaultdict | DefaultDict |
| collections.deque | Deque |
| collections.OrderedDict | OrderedDict |
| re.Match | Match |
| re.Pattern | Pattern |

## Type Expression Parameters

在 typing 模块中定义的某些类型修改其他类型表达式。Table 5-3 列出的类型提供了关于 *type_expression* 修改类型的额外类型信息或约束。

Table 5-3\. 类型表达式参数

| Parameter | 用法和描述 |
| --- | --- |
| Annotated | Annotated[*type_expression, expression, ...*] 3.9+ 用额外的元数据扩展 *type_expression*。函数 *fn* 的额外元数据可以在运行时使用 get_type_hints(*fn*, include_extras=**True**) 获取。 |
| ClassVar | ClassVar[*type_expression*] 表示变量是类变量，不应该作为实例变量赋值。 |
| Final | Final[*type_expression*] 3.8+ 表示变量不应该在子类中写入或重写。 |
| Optional | Optional[*type_expression*] Equivalent to *type_expression* &#124; **None**. Often used for named arguments with a default value of **None**. (Optional does not automatically define **None** as the default value, so you must still follow it with =**None** in a function signature.) 3.10+ With the availability of the &#124; operator for specifying alternative type attributes, there is a growing consensus to prefer *type_expression* &#124; **None** over using Optional[*type_expression*]. |

## 抽象基类

与内置类型类似，typing 模块的初始实现包括了与 collections.abc 模块中的抽象基类对应的类型定义。许多这些类型后来已被弃用（见下文），但两个定义已保留为 collections.abc 中 ABCs 的别名（见 表格 5-4）。

表格 5-4\. 抽象基类别名

| Type | Method subclasses must implement |
| --- | --- |
| Hashable | __hash__ |
| Sized | __len__ |

-3.9 在 Python 3.9 之前，typing 模块中的以下定义表示在 collections.abc 模块中定义的抽象基类，例如 Sequence[int] 用于整数序列。从 3.9 开始，typing 模块中这些名称已被弃用，因为它们在 collections.abc 中对应的类型现在支持 [] 语法：

| AbstractSet | Container | Mapping |
| --- | --- | --- |
| AsyncContextManager | ContextManager | MappingView |
| AsyncGenerator | Coroutine | MutableMapping |
| AsyncIterable | Generator | MutableSequence |
| AsyncIterator | ItemsView | MutableSet |
| Awaitable | Iterable | Reversible |
| ByteString | Iterator | Sequence |
| Collection | KeysView | ValuesView |

## 协议

typing 模块定义了几个*协议*，类似于其他一些语言称为“接口”的概念。协议是抽象基类，旨在简洁表达对类型的约束，确保其包含某些方法。typing 模块中当前定义的每个协议都与单个特殊方法相关，其名称以 Supports 开头，后跟方法名（然而，如 [typeshed](https://oreil.ly/adB9Z) 中定义的其他库可能不遵循相同的约束）。协议可用作确定类对该协议功能支持的最小抽象类：要遵守协议，类所需做的就是实现协议的特殊方法。

表格 5-5 列出了 typing 模块中定义的协议。

表格 5-5\. typing 模块中的协议及其必需方法

| Protocol | Has method |
| --- | --- |
| SupportsAbs | __abs__ |
| SupportsBytes | __bytes__ |
| SupportsComplex | __complex__ |
| SupportsFloat | __float__ |
| SupportsIndex 3.8+ | __index__ |
| SupportsInt | __int__ |
| SupportsRound | __round__ |

类不必显式从协议继承以满足 issubclass（*cls*，*protocol_type*），或使其实例满足 isinstance（*obj*，*protocol_type*）。类只需实现协议中定义的方法即可。例如，想象一个实现罗马数字的类：

```py
`class` RomanNumeral:
    *`"""Class representing some Roman numerals and their int`* 
 *`values.`*
 *`"""`*
    int_values = {'I': 1, 'II': 2, 'III': 3, 'IV': 4, 'V': 5}

    `def` __init__(self, label: str):
        self.label = label

    `def` __int__(self) -> int:
        `return` RomanNumeral.int_values[self.label]
```

要创建此类的实例（例如，表示电影标题中的续集）并获取其值，您可以使用以下代码：

```py
>>> movie_sequel = RomanNumeral('II')
>>> print(int(movie_sequel))
```

```py
2
```

RomanNumeral 满足 issubclass，并且由于实现了 __int__，与 SupportsInt 进行 isinstance 检查，即使它不是显式从协议类 SupportsInt 继承:⁴

```py
>>> issubclass(RomanNumeral, typing.SupportsInt)
```

```py
True
```

```py
>>> isinstance(movie_sequel, typing.SupportsInt)
```

```py
True
```

## 实用程序和装饰器

Table 5-6 列出了在 typing 模块中定义的常用函数和装饰器；接下来是一些示例。

表 5-6.定义在 typing 模块中的常用函数和装饰器

| Function/decorator | 用法和描述 |
| --- | --- |
| cast | cast（*type*，*var*）向静态类型检查器发出信号，*var*应被视为*type*类型。返回*var*；在运行时，*var*没有更改、转换或验证。请参见表后的示例。 |
| final | @final 3.8+ 用于装饰类定义中的方法，如果该方法在子类中被重写则发出警告。也可以用作类装饰器，用于警告是否正在对类本身进行子类化。 |
| get_args | get_args（*custom_type*）返回用于构造自定义类型的参数。 |
| get_origin | get_origin（*custom_type*）3.8+ 返回用于构造自定义类型的基础类型。 |
| get_type_hints | get_type_hints（*obj*）返回结果，就像访问*obj*.__annotations__ 一样。可以使用可选的 globalns 和 localns 命名空间参数调用，以解析作为字符串给出的前向类型引用，和/或使用包含 Annotations 的任何非类型注释的可选 Boolean include_extras 参数。 |
| NewType | NewType（*type_name*，*type*）定义了从*type*派生的自定义类型。*type_name*是一个字符串，应与分配 NewType 的局部变量匹配。用于区分常见类型的不同用途，例如用于员工姓名的 str 与用于部门名称的 str。有关此函数的更多信息，请参见“NewType”。 |
| no_type_check | @no_type_check 用于指示注释不打算用作类型信息。可应用于类或函数。 |
| no_type_che⁠c⁠k⁠_​d⁠e⁠c⁠orator | @no_type_check_decorator 用于向另一个装饰器添加 no_type_check 行为。 |
| overload | @overload 用于允许定义多个方法，名称相同但签名类型不同。请参见表后的示例。 |
| r⁠u⁠n⁠t⁠i⁠m⁠e⁠_​c⁠h⁠e⁠c⁠k⁠a⁠b⁠l⁠e | @runtime_checkable 3.8+ 用于为自定义协议类添加 isinstance 和 issubclass 支持。有关此装饰器的更多信息，请参见 “Using Type Annotations at Runtime” 。 |
| TypeAlias | *name*: TypeAlias = *type_expression* 3.10+ 用于区分类型别名的定义和简单赋值。在 *type_expression* 是简单类名或字符串值引用尚未定义的类的情况下最有用，这可能看起来像是一个赋值。TypeAlias 只能在模块范围内使用。一个常见用法是使得一致重用冗长的类型表达式变得更容易，例如：Number: TypeAlias = int &#124; float &#124; Fraction。更多关于此注解的信息，请参见 “TypeAlias” 。 |
| type_check_only | @type_check_only 用于指示类或函数仅在类型检查时使用，而在运行时不可用。 |
| TYPE_CHECKING | 一个特殊的常量，静态类型检查器将其评估为 **True**，但在运行时设置为 **False**。使用它可以跳过导入用于支持类型检查的大型、导入缓慢的模块（以便在运行时不需要该导入）。 |
| TypeVar | TypeVar(*type_name*, **types*) 定义用于复杂泛型类型中的类型表达式元素，使用 Generic。*type_name* 是一个字符串，应与分配给 TypeVar 的局部变量匹配。如果未提供 *types*，则相关的 Generic 将接受任何类型的实例。如果提供了 *types*，则 Generic 将仅接受提供的类型或其子类的实例。还接受名为协变和逆变（默认为 False）的布尔参数，以及 bound 参数。关于这些参数的详细信息，请参见 “Generics and TypeVars” 和 [typing 模块文档](https://oreil.ly/069u4) 。 |

在类型检查时使用 overload 来标记必须以特定组合使用的命名参数。在这种情况下，fn 必须以 str 键和 int 值对或单个布尔值调用：

```py
@typing.overload
`def` fn(*, key: str, value: int):
    `.``.``.`

@typing.overload
`def` fn(*, strict: bool):
    `.``.``.`

`def` fn(**kwargs):
    *`# implementation goes here, including handling of differing`* 
    *`# named arguments`*
    `pass`

*`# valid calls`*
fn(key='abc', value=100)
fn(strict=True)

*`# invalid calls`*
fn(1)
fn('abc')
fn('abc', 100)
fn(key='abc')
fn(`True`)
fn(strict=True, value=100)
```

请注意，overload 装饰器仅用于静态类型检查。要根据参数类型在运行时实际分派到不同方法，请使用 functools.singledispatch。

使用 cast 函数可以强制类型检查器在 cast 的作用域内将变量视为特定类型：

```py
`def` func(x: list[int] | list[str]):
    `try`:
        `return` sum(x)
    `except` TypeError:
        x = cast(list[str], x)
        `return` ','.join(x)
```

# 谨慎使用 cast

cast 是一种覆盖代码中特定位置可能存在的所有推断或先前注释的方法。它可能隐藏代码中的实际类型错误，导致类型检查通行不完整或不准确。在前面的示例中，func 本身不会引发任何 mypy 警告，但如果传递了混合整数和字符串的列表，则在运行时会失败。

## 定义自定义类型

正如 Python 的**class**语法允许创建新的运行时类型和行为一样，本节讨论的 typing 模块构造使得能够创建用于高级类型检查的专门类型表达式。

typing 模块包括三个类，你可以继承这些类来获取类型定义和其他默认特性，详见表 5-7。

表 5-7\. 定义自定义类型的基类

| Generic | Generic[*type_var*, ...] 定义了一个类型检查抽象基类，用于类的方法引用一个或多个 TypeVar 定义的类型。泛型将在以下小节详述。 |
| --- | --- |
| NamedTuple | NamedTuple 是 collections.namedtuple 的有类型实现。详见“NamedTuple”获取更多详情和示例。 |
| TypedDict | TypedDict 3.8+ 定义了一个类型检查字典，其具有每个键的特定键和值类型。详见“TypedDict”了解详情。 |

### 泛型和 TypeVar

*泛型* 是定义类模板的类型，这些类可以根据一个或多个类型参数调整其方法签名的类型注释。例如，dict 是一个泛型，接受两个类型参数：字典键的类型和字典值的类型。以下是如何使用 dict 来定义一个将颜色名称映射到 RGB 三元组的字典：

```py
color_lookup: dict[str, tuple[int, int, int]] = {}
```

变量 color_lookup 将支持如下语句：

```py
color_lookup['red'] = (255, 0, 0)
color_lookup['red'][2]
```

然而，以下语句由于键或值类型不匹配而生成了 mypy 错误：

```py
color_lookup[0]
```

```py
error: Invalid index type "int" for "dict[str, tuple[int, int, int]]";
expected type "str"
```

```py
color_lookup['red'] = (255, 0, 0, 0)
```

```py
error: Incompatible types in assignment (expression has type
"tuple[int, int, int, int]", target has type "tuple[int, int, int]")
```

泛型类型允许在一个类中定义与该类所处理对象的具体类型无关的行为。泛型通常用于定义容器类型，如 dict、list、set 等。通过定义泛型类型，我们避免了对 DictOfStrInt、DictOfIntEmployee 等详细定义类型的必要性。相反，泛型 dict 被定义为 dict[*KT*, *VT*]，其中*KT*和*VT*是字典的键类型和值类型的占位符，并且可以在实例化字典时定义任何特定类型。

举个例子，让我们定义一个假想的泛型类：一个累加器，可以更新值，但也支持撤销方法。由于累加器是一个泛型容器，我们声明一个 TypeVar 来表示所包含对象的类型：

```py
`import` typing
T = typing.TypeVar('T')
```

累加器类被定义为泛型的子类，其中 T 作为类型参数。以下是类声明及其 __init__ 方法，它创建了一个包含对象类型 T 的初始为空的列表：

```py
`class` Accumulator(typing.Generic[T]):
    `def` __init__(self):
        self._contents: list[T] = []
```

要添加 update 和 undo 方法，我们定义引用类型 T 的参数，表示所包含的对象类型：

```py
    `def` update(self, *args: T) -> `None`:
        self._contents.extend(args)

    `def` undo(self) -> `None`:
        *`# remove last value added`*
        `if` self._contents:
            self._contents.pop()
```

最后，我们添加 __len__ 和 __iter__ 方法，以便可以对累加器实例进行迭代：

```py
    `def` __len__(self) -> int:
        `return` len(self._contents)

    `def` __iter__(self) -> typing.Iterator[T]:
        `return` iter(self._contents)
```

现在，可以使用 Accumulator[int]编写代码来收集多个整数值：

```py
acc: Accumulator[int] = Accumulator()
acc.update(1, 2, 3)
print(sum(acc))  # prints 6
acc.undo()
print(sum(acc))  # prints 3
```

因为 acc 是包含 ints 的 Accumulator，所以下面的语句会生成 mypy 错误消息：

```py
acc.update('A')
```

```py
error: Argument 1 to "update" of "Accumulator" has incompatible type
"str"; expected "int"
```

```py
print(''.join(acc))
```

```py
error: Argument 1 to "join" of "str" has incompatible type
"Accumulator[int]"; expected "Iterable[str]"
```

### 限制 TypeVar 为特定类型

在我们的 Accumulator 类中，我们从未直接调用所包含的 T 对象的方法。对于这个示例，T TypeVar 是纯粹无类型的，因此像 mypy 这样的类型检查器无法推断出 T 对象的任何属性或方法的存在。如果泛型需要访问其包含的 T 对象的属性，则应使用 TypeVar 的修改形式来定义 T。

下面是一些 TypeVar 定义的示例：

```py
*`# T must be one of the types listed (int, float, complex, or str)`*
T = typing.TypeVar('T', int, float, complex, str)
*`# T must be the class MyClass or a subclass of the class MyClass`*
T = typing.TypeVar('T', bound=MyClass)
*`# T must implement __len__ to be a valid subclass of the Sized protocol`*
T = typing.TypeVar('T', bound=collections.abc.Sized)
```

这些形式的 T 允许在 T 的 TypeVar 定义中使用这些类型的方法。

### NamedTuple

collections.namedtuple 函数简化了支持对元组元素进行命名访问的类似类的元组类型的定义。NamedTuple 提供了此功能的类型化版本，使用类似于数据类（在“数据类”中介绍）的属性样式语法的类。下面是一个具有四个元素的 NamedTuple，带有名称、类型和可选默认值：

```py
`class` HouseListingTuple(typing.NamedTuple):
    address: str
    list_price: int
    square_footage: int = 0
    condition: str = 'Good'
```

NamedTuple 类生成一个默认的构造函数，接受每个命名字段的位置参数或命名参数：

```py
listing1 = HouseListingTuple(
    address='123 Main',
    list_price=100_000,
    square_footage=2400,
    condition='Good',
)

print(listing1.address)  *`# prints: 123 Main`*
print(type(listing1))    *`# prints: <class 'HouseListingTuple'>`*
```

尝试创建元组时如果元素数量过少会引发运行时错误：

```py
listing2 = HouseListingTuple(
    '123 Main',
)
*`# raises a runtime error: TypeError: HouseListingTuple.__new__()` 
`# missing 1 required positional argument: 'list_price'`*
```

### TypedDict

3.8+ Python 字典变量在旧代码库中经常难以理解，因为字典有两种用法：作为键/值对的集合（例如，从用户 ID 到用户名的映射），以及将已知字段名映射到值的记录。通常很容易看出函数参数将作为字典传递，但实际的键和值类型取决于可能调用该函数的代码。除了简单地定义字典可以是一个 str 到 int 值的映射，例如 dict[str, int]，TypedDict 还定义了预期的键和每个相应值的类型。以下示例定义了之前房屋列表类型的 TypedDict 版本（注意，TypedDict 定义不接受默认值定义）：

```py
`class` HouseListingDict(typing.TypedDict):
    address: str
    list_price: int
    square_footage: int
    condition: str
```

TypedDict 类生成一个默认的构造函数，为每个定义的键接受命名参数：

```py
listing1 = HouseListingDict(
    address='123 Main',
    list_price=100_000,
    square_footage=2400,
    condition='Good',
)

print(listing1['address'])  # prints *`123 Main`*
print(type(listing1))  # prints *`<class 'dict'>`*

listing2 = HouseListingDict(
    address='124 Main',
    list_price=110_000,
)
```

与 NamedTuple 示例不同，listing2 不会引发运行时错误，只是创建一个具有给定键的字典。但是，mypy 将使用消息标记 listing2 为类型错误：

```py
error: Missing keys ("square_footage", "condition") for TypedDict
"HouseListing"
```

要向类型检查器指示某些键可能被省略（但仍然验证给定的键），请将 total=False 添加到类声明中：

```py
`class` HouseListing(typing.TypedDict, total=False):
    *`# ...`*
```

3.11+ 个别字段还可以使用 Required 或 NotRequired 类型注释显式地标记它们为必需或可选：

```py
`class` HouseListing(typing.TypedDict):
    address: typing.Required[str]
    list_price: int
    square_footage: typing.NotRequired[int]
    condition: str
```

TypedDict 也可以用来定义泛型类型：

```py
T = typing.TypeVar('T')

`class` Node(typing.TypedDict, typing.Generic[T]):
    label: T
    neighbors: list[T]

n = Node(label='Acme', neighbors=['anvil', 'magnet', 'bird seed'])
```

# 不要使用传统的 TypedDict(name, **fields) 格式

为了支持向较旧版本的 Python 进行回溯，TypedDict 的初始版本也允许您使用类似于 namedtuple 的语法，例如：

```py
HouseListing = TypedDict('HouseListing',
                         address=str, 
                         list_price=int, 
                         square_footage=int, 
                         condition=str)
```

或：

```py
HouseListing = TypedDict('HouseListing',
                         {'address': str, 
                          'list_price': int, 
                          'square_footage': int,
                          'condition': str})
```

这些形式在 Python 3.11 中已被弃用，并计划在 Python 3.13 中移除。

请注意，TypedDict 实际上不定义新类型。通过从 TypedDict 继承创建的类实际上充当字典工厂，从而创建的实例 *是* 字典。通过重新使用定义 Node 类的先前代码片段，我们可以看到这一点，使用 type 内置函数：

```py
n = Node(label='Acme', neighbors=['anvil', 'magnet', 'bird seed'])
print(type(n))           *`# prints: <class 'dict'>`*
print(type(n) is dict)   *`# prints: True`*
```

使用 TypedDict 时没有特殊的运行时转换或初始化；TypedDict 的好处来自静态类型检查和自我文档化，这些自然地通过使用类型注解积累。

### TypeAlias

3.10+ 定义简单类型别名可能会被误解为将类分配给变量。例如，在这里我们为数据库中的记录标识符定义了一个类型：

```py
Identifier = int
```

为了澄清这个声明是为了定义用于类型检查的自定义类型名称，请使用 TypeAlias：

```py
Identifier: TypeAlias = int
```

TypeAlias 在定义尚未定义的类型并以字符串值引用时非常有用：

```py
*`# Python will treat this like a standard str assignment`*
TBDType = 'ClassNotDefinedYet'

*`# indicates that this is actually a forward reference to a class`*
TBDType: TypeAlias = 'ClassNotDefinedYet'
```

TypeAlias 类型只能在模块范围内定义。使用 TypeAlias 定义的自定义类型与目标类型可互换。与后续章节中涵盖的 NewType 相对比（NewType 不创建新类型，仅为现有类型提供新名称），TypeAlias 仅为现有类型提供新名称。

### NewType

NewType 允许您定义特定于应用程序的子类型，以避免使用相同类型为不同变量可能导致的混淆。例如，如果您的程序使用 str 值来表示不同类型的数据，很容易意外地交换值。假设您有一个模拟员工和部门的程序。以下类型声明不够描述清楚——哪一个是关键，哪一个是值？

```py
employee_department_map: dict[str, str] = {}
```

为员工和部门 ID 定义类型使得声明更清晰：

```py
EmpId = typing.NewType('EmpId', str)
DeptId = typing.NewType('DeptId', str)
employee_department_map: dict[EmpId, DeptId] = {}
```

这些类型定义也将允许类型检查器标记此不正确的使用：

```py
`def` transfer_employee(empid: EmpId, to_dept: DeptId):
 *`# update department for employee`
*     employee_department_map[to_dept] = empid
```

运行 mypy 时会报告这些错误，如下所示：employee_department_map[to_dept] = empid。

```py
error: Invalid index type "DeptId" for "Dict[EmpId, DeptId]"; expected
type "EmpId"
error: Incompatible types in assignment (expression has type "EmpId",
target has type "DeptId")
```

使用 NewType 通常需要您也使用 typing.cast；例如，要创建一个 EmpId，您需要将一个 str 强制转换为 EmpId 类型。

您还可以使用 NewType 指示应用程序特定类型的所需实现类型。例如，基本的美国邮政编码是五位数字。通常会看到这种实现使用 int，这在具有前导 0 的邮政编码时会出现问题。为了指示邮政编码应使用 str 实现，您的代码可以定义此类型检查类型：

```py
ZipCode = typing.NewType("ZipCode", str)
```

使用 ZipCode 注释变量和函数参数将有助于标记错误的 int 用于邮政编码值的使用。

# 在运行时使用类型注解

函数和类变量的注释可以通过访问函数或类的 __annotations__ 属性进行内省（尽管更好的做法是调用 inspect.get_annotations()）：

```py
>>> `def` f(a:list[str], b) -> int:
...     `pass`
...
>>> f.__annotations__
```

```py
{'a': list[str], 'return': <class 'int'>}
```

```py
>>> `class` Customer:
...     name: str
...     reward_points: int = 0
...
>>> Customer.__annotations__
```

```py
{'name': <class 'str'>, 'reward_points': <class 'int'>}
```

此功能被 pydantic 和 FastAPI 等第三方包使用，以提供额外的代码生成和验证功能。

3.8+ 要定义自己的自定义协议类，以支持运行时检查的子类和 isinstance，请将该类定义为 typing.Protocol 的子类，并对所需的协议方法进行空方法定义，并使用@runtime_checkable（在表 5-6 中介绍）。如果*不*使用@runtime_checkable 装饰它，您仍然定义了一个非常适用于静态类型检查的协议，但它不会使用 issubclass 和 isinstance 进行运行时检查。

例如，我们可以定义一个协议，指示一个类实现了更新和撤销方法，如下所示（Python 中的省略号`...`是指示空方法定义的便捷语法）：

```py
T = typing.TypeVar('T')

@typing.runtime_checkable
`class` SupportsUpdateUndo(typing.Protocol):
    `def` update(self, *args: T) -> `None`:
        ...
    `def` undo(self) -> `None`:
        ...
```

在不对 Accumulator 的继承路径进行任何更改（在“泛型和 TypeVars”中定义）的情况下，它现在满足了对 SupportsUpdateUndo 的运行时类型检查：

```py
>>> issubclass(Accumulator, SupportsUpdateUndo)
```

```py
True
```

```py
>>> isinstance(acc, SupportsUpdateUndo)
```

```py
True
```

另外，现在任何其他实现了更新和撤销方法的类都将被视为`SupportsUpdateUndo`的“子类”。

# 如何为您的代码添加类型注解

看到了使用类型注解提供的一些特性和功能，您可能想知道最佳的入门方式。本节描述了添加类型注解的几种情景和方法。

## 向新代码添加类型注解

当您开始编写一个简短的 Python 脚本时，添加类型注解可能会显得多余。作为[“两个披萨规则”](https://oreil.ly/SWLnG)的一个衍生，我们建议使用“两个函数规则”：一旦您的脚本包含两个函数或方法，就回头添加方法签名的类型注解，以及必要时添加任何共享变量或类型。使用 TypedDict 来注释任何在类的位置使用的 dict 结构，以便在一开始就清晰地定义 dict 键或在进行过程中进行文档化；使用 NamedTuples（或数据类：本书的一些作者*强烈*倾向于后者）来定义所需的特定属性，以用于这些数据“捆”。

如果您开始一个具有许多模块和类的重大项目，那么您一定应该从一开始就使用类型注解。它们可以让您更加高效，因为它们有助于避免常见的命名和类型错误，并确保您在 IDE 中获得更全面的支持自动完成。在具有多个开发人员的项目中，这一点尤为重要：在代码中记录类型有助于告诉团队中的每个人对类型和值的期望。将这些类型捕获在代码中使它们在开发过程中立即可访问和可见，比单独的文档或规范要更加方便。

如果你正在开发一个要在多个项目中共享的库，那么最好从一开始就使用类型注解，很可能与你 API 设计中的函数签名并行。在库中添加类型注解将会为客户开发者简化生活，因为所有现代 IDE 都包含类型注解插件来支持静态类型检查、函数自动完成和文档编写。它们在编写单元测试时也会帮助你，因为你将受益于相同的丰富 IDE 支持。

对于任何这些项目，将类型检查实用程序添加到你的预提交挂钩中，这样你可以及时解决任何可能潜入你新代码库中的类型违规。这样一来，你可以在出现问题时修复它们，而不是等到做大的提交后才发现在多个地方都出现了基本的类型错误。

## 给现有代码添加类型注解（渐进式类型）

有几家公司已经运行了将类型注解应用于大型现有代码库的项目，推荐采用渐进式的方法，称为*渐进式类型*。通过渐进式类型，你可以逐步地按步骤处理你的代码库，逐步添加和验证类型注解到几个类或模块。

有些工具，比如 mypy，会让你逐个函数地添加类型注解。默认情况下，mypy 会跳过没有类型签名的函数，因此你可以逐步地逐个函数地处理你的代码库。这种增量的过程允许你将精力集中在代码的各个部分，而不是一次性地在所有地方添加类型注解，然后试图解决一堆类型检查器错误。

推荐的一些方法包括：

+   确定你使用最频繁的模块，并逐步添加类型，逐个方法地进行。（这些可能是核心应用程序类模块，或广泛共享的实用程序模块。）

+   逐个方法地添加注解，以便逐步引发并解决类型检查问题。

+   使用 pytype 或 pyre 推断生成初始的 *.pyi* 桩文件（在下一节中讨论）。然后，逐步从 *.pyi* 文件中迁移类型，可以手动进行，也可以使用像 pytype 的 merge_pyi 工具这样的自动化工具。

+   开始使用类型检查器的宽松默认模式，这样大部分代码会被跳过，你可以将注意力集中在特定的文件上。随着工作的进行，逐渐转向更严格的模式，以突出剩余的项目，并且已经注释的文件不会因为接受新的非注释代码而退步。

## 使用 .pyi 桩文件

有时候你可能无法访问 Python 的类型注解。例如，你可能正在使用一个没有类型注解的库，或者使用一个其函数是用 C 实现的模块。

在这些情况下，可以使用单独的*.pyi*存根文件，其中只包含相关的类型注解。本章开头提到的多个类型检查器可以生成这些存根文件。您可以从[typeshed 存储库](https://oreil.ly/jKhNR)下载流行的 Python 库以及 Python 标准库本身的存根文件。您可以从 Python 源文件中维护存根文件，或者使用某些类型检查器中可用的合并工具将其集成回原始 Python 源代码中。

# 摘要

Python 作为一个强大的语言和编程生态系统已经稳步崛起，支持重要的企业应用程序。曾经作为脚本和任务自动化的实用语言，现在已经成为影响数百万用户的重要和复杂应用程序平台，用于关键任务甚至是地外系统。⁵ 添加类型注解是开发和维护这些系统的重要一步。

Python 的[类型注解在线文档](https://oreil.ly/Zg_NX)提供了最新的描述、示例和[最佳实践](https://oreil.ly/xhq5g)，因为类型注解的语法和实践不断演变。作者还特别推荐了[*流畅的 Python*第二版](http://oreilly.com/library/view/fluent-python-2nd/9781492056348)，作者是 Luciano Ramalho（O'Reilly），尤其是第八章和第十五章，这些章节专门讲解了 Python 类型注解。

¹ 强大而广泛的单元测试也将防范许多商业逻辑问题，这是任何类型检查都无法捕捉的—所以，类型提示不应该*代替*单元测试，而应该*与*单元测试一起使用。

² 类型注解的*语法*在 Python 3.0 中引入，但其*语义*则是后来才明确指定的。

³ 这种方法也兼容 Python 2.7 代码，当时广泛使用。

⁴ 并且 SupportsInt 使用了 runtime_checkable 装饰器。

⁵ NASA 的喷气推进实验室使用 Python 开发了坚韧号火星车和毅力号火星直升机；负责发现引力波的团队既用 Python 协调仪器，也用 Python 分析了得到的大量数据。
