# 第八章：核心内置和标准库模块

术语*内置*在 Python 中有多个含义。在许多上下文中，*内置*意味着一个对象可以直接在 Python 代码中访问，而无需**import**语句。“Python 内置对象”节展示了 Python 允许这种直接访问的机制。Python 中的内置类型包括数字、序列、字典、集合、函数（所有在第 3 章中讨论）、类（在“Python 类”中讨论）、标准异常类（在“异常对象”中讨论）和模块（在“模块对象”中讨论）。“io 模块”涵盖了文件类型，而“内部类型”涵盖了 Python 内部操作的一些其他内置类型。本章在开篇部分提供了内置核心类型的额外覆盖，并在“内置函数”中介绍了模块 builtins 中可用的内置函数。

一些模块被称为“内置”，因为它们位于 Python 标准库中（尽管需要**import**语句才能使用），而不是附加模块，也称为 Python *扩展*。

本章涵盖了几个内置核心模块：即标准库模块 sys、copy、collections、functools、heapq、argparse 和 itertools。您将在各自部分“*x* 模块”中找到对每个模块 *x* 的讨论。

第 9 章涵盖了一些与字符串相关的内置核心模块（string、codecs 和 unicodedata），采用相同的部分名称约定。第 10 章介绍了 re 模块在“正则表达式和 re 模块”中的使用。

# 内置类型

表 8-1 提供了 Python 核心内置类型的简要概述。关于这些类型的许多细节以及关于其实例操作的详细信息可以在整个第 3 章中找到。在本节中，“任意数字”特指“任意非复数数字”。此外，许多内置函数至少以某些参数的位置方式接受参数；我们使用 3.8+ 的位置限定符 /，在“位置限定符”中进行了介绍。

表 8-1\. Python 核心内置类型

| bool | bool(*x*=**False**, /) 当 *x* 评估为假时返回**False**；当 *x* 评估为真时返回**True**（参见“布尔值”）。bool 扩展自 int：内置名称**False**和**True**分别指 bool 的唯一两个实例。这些实例也是等于 0 和 1 的 int，但 str(**True**) 是 'True'，str(**False**) 是 'False'。 |
| --- | --- |
| bytearray | bytearray(*x*=b'', /[, codec[, errors]]) 返回一个可变的字节序列（值为 0 到 255 的 int），支持可变序列的通常方法，以及 str 的方法。当*x*是一个 str 时，您还必须传递 codec 并可能传递 errors；结果就像调用 bytearray(*x*.encode(codec, errors))一样。当*x*是一个 int 时，它必须>=0：生成的实例的长度为*x*，并且每个项目都初始化为 0。当*x*符合[缓冲区协议](https://oreil.ly/HlOmv)时，从*x*读取的只读字节缓冲区初始化实例。否则，*x*必须是一个产生 int >=0 且<256 的可迭代对象；例如，bytearray([1,2,3,4]) == bytearray(b'\x01\x02\x03\x04')。 |
| bytes | bytes(*x*=b'', /[, codec[, errors]]) 返回一个不可变的字节序列，具有与 bytearray 相同的非变异方法和相同的初始化行为。 |
| complex | complex(*real*=0, *imag*=0) 将任何数字或适当的字符串转换为复数。当*real*是一个数字时，*imag*可以存在，并且在这种情况下*imag*也是一个数字：生成的复数的虚部。另请参阅“复数”。 |

| dict | dict(*x*={}, /) 返回一个具有与*x*相同项目的新字典。（我们在“字典”中介绍字典。）当*x*是一个字典时，dict(*x*)返回*x*的浅拷贝，就像*x*.copy()一样。另外，*x*可以是一个其项目为成对项（每个项为两个项目的可迭代对象）的可迭代对象。在这种情况下，dict(*x*)返回一个字典，其键是*x*中每对的第一个项目，其值是对应的第二个项目。当*x*中的键出现多次时，Python 使用与键的最后一次出现对应的值。换句话说，当*x*是任何产生成对项的可迭代对象时，*c* = dict(*x*)与以下等效：

```py
c = {}
`for` key, value `in` x:
    c[key] = value
```

您还可以使用命名参数调用 dict，除了或代替位置参数*x*。每个命名参数都成为字典中的一项，名称作为键：这样的额外项可能会“覆盖”*x*中的一项。

| float | float(*x*=0.0, /) 将任何数字或适当的字符串转换为浮点数。参见“浮点数”。 |
| --- | --- |
| frozenset | frozenset(*seq*=(), /) 返回一个新的冻结（即不可变）集合对象，其包含与可迭代对象*seq*相同的项目。当*seq*是一个 frozenset 时，frozenset(*seq*)返回*seq*本身，就像*seq*.copy()一样。另请参阅“集合操作”。 |
| int | int(*x*=0, /, base=10) 将任何数字或适当的字符串转换为整数。当 *x* 是一个数字时，int 朝向 0 截断，“舍弃”任何小数部分。当 *x* 是一个字符串时，base 可能存在：然后，base 是转换基数，介于 2 和 36 之间，其中 10 是默认值。你可以显式地将 base 传递为 0：然后，基数为 2、8、10 或 16，具体取决于字符串 *x* 的形式，就像对整数字面值一样，如“整数”中所述。 |
| list | list(*seq*=(), /) 返回一个具有与可迭代对象 *seq* 相同项、相同顺序的新列表对象。当 *seq* 是一个列表时，list(*seq*) 返回 *seq* 的一个浅拷贝，就像 *seq*[:] 一样。请参见“列表”。 |

| memoryview | memoryview(*x*, /) 返回一个对象 *m*，“查看”与 *x* 完全相同的底层内存，*x* 必须是支持[缓冲区协议](https://oreil.ly/HlOmv)的对象（例如 bytes、bytearray 或 array.array 的实例），每个 *m*.itemsize 字节的项目。在 *m* 是“一维”的普通情况下（本书不涵盖“多维”memoryview 实例的复杂情况），len(*m*) 是项目数。你可以对 *m* 进行索引（返回 int）或对其进行切片（返回“查看”相同底层内存的适当子集的 memoryview 实例）。当 *x* 是可变的时，*m* 也是可变的（但你不能改变 *m* 的大小，所以，当你对一个切片赋值时，它必须来自与切片相同长度的可迭代对象）。*m* 是一个序列，因此可迭代，并且在 *x* 是可散列且 *m*.itemsize 是一个字节时是可散列的。

*m* 提供了几个只读属性和方法；详细信息请参见[在线文档](https://oreil.ly/SIsvF)。两个特别有用的方法是 *m*.tobytes（将 *m* 的数据作为 bytes 实例返回）和 *m*.tolist（将 *m* 的数据作为整数列表返回）。

| object | object() 返回一个新的 object 实例，Python 中最基本的类型。类型为 object 的实例没有功能：这些实例的唯一用途是作为“哨兵”——即不等于任何不同对象的对象。例如，当函数接受一个可选参数，其中 **None** 是一个合法值时，你可以使用哨兵作为参数的默认值来指示参数被省略了：

```py
MISSING = object()
`def` check_for_none(obj=MISSING):
    `if` obj `is` MISSING:
        `return` -1
    `return` 0 `if` obj `is` `None` `else` 1
```

|

| set | set(*seq*=(), /) 返回一个具有与可迭代对象 *seq* 相同项的新的可变集合对象。当 *seq* 是一个集合时，set(*seq*) 返回 *seq* 的一个浅拷贝，就像 *seq*.copy() 一样。请参见“集合”。 |
| --- | --- |
| slice | slice([start, ]stop[, step], /) 返回具有只读属性 start、stop 和 step 绑定到相应参数值的切片对象，当缺少时，默认为**None**。对于正索引，这样的切片表示与 range(start, stop, step)相同的索引。切片语法，*obj*[start:stop:step]，将切片对象作为参数传递给对象*obj*的 __getitem__、__setitem__ 或 __delitem__ 方法。由*obj*的类来解释其方法接收的切片。还请参见“容器切片”。 |
| str | str(*obj*='', /) 返回*obj*的简明可读字符串表示。如果*obj*是字符串，则 str 返回*obj*。还请参见表 8-2 中的 repr 和表 4-1 中的 __str__。 |
| super | super(), super(*cls*, *obj*, /) 返回对象*obj*的超级对象（*obj*必须是类*cls*或任何*cls*子类的实例），适合调用超类方法。只在方法代码内实例化此内建类型。super(*cls*, *obj*)语法是 Python 2 中保留的遗留形式，用于兼容性。在新代码中，通常在方法内部调用 super()而不带参数，Python 通过内省（如 type(self)和 self 分别确定*cls*和*obj*）。参见“协作超类方法调用”。 |
| tuple | tuple(*seq*=(), /) 返回具有与可迭代对象*seq*中相同项目的元组，顺序相同。当*seq*是元组时，tuple 返回*seq*本身，类似于*seq*[:]。参见“元组”。 |
| type | type(*obj*, /) 返回类型对象，即*obj*的类型（即*obj*是实例的最终类型，也称为*leafmost*类型）。对于任何*x*，type(*x*)与*x*.__class__ 相同。避免检查类型的相等性或身份（详情请参阅下面的警告）。这个函数通常用于调试；例如，当值*x*的行为不如预期时，插入 print(type(*x*), *x*)。它还可以用于在运行时动态创建类，如第四章中所述。 |

# 内建函数

表 8-2 涵盖了 Python 内建函数（实际上，有些类型仅在*似乎*它们被作为函数使用时）在模块 builtins 中按字母顺序排列。内建函数的名称*不*是关键字。这意味着您可以在本地或全局作用域中绑定一个标识符作为内建名称，尽管我们建议避免这样做（请参见下面的警告！）。在本地或全局作用域中绑定的名称会覆盖内建作用域中绑定的名称，因此本地和全局名称*隐藏*内建名称。您还可以重新绑定内建作用域中的名称，如“Python 内建”中所述。 |

# 不要隐藏内建函数

避免意外隐藏内置函数：你的代码可能后面会用到它们。常常会诱人地使用像 input、list 或 filter 这样的自然命名作为你自己变量的名称，但是*不要这样做*：这些都是 Python 内置类型或函数的名称，如果重复使用它们作为你自己的变量名，会导致这些内置类型和函数无法访问。除非你养成*永远*不用自己的名字隐藏内置函数名的习惯，否则迟早会因为意外隐藏而在代码中遇到神秘的错误。

许多内置函数只能使用位置参数调用，不能使用命名参数。在表 8-2 中，我们提到了这种限制不适用的情况；当适用时，我们还使用了 3.8+的位置参数专用标记 /，详见“位置参数专用标记”。

表 8-2\. Python 的核心内置函数

| __import__ | __import__(*module_name*[, *globals*[, *locals*[, *fromlist*]]], /) 在现代 Python 中已弃用；请使用 importlib.import_module，详见“模块加载”。 |
| --- | --- |
| abs | abs(*x*, /) 返回数*x*的绝对值。当*x*为复数时，abs 返回*x*.imag ** 2 + *x*.real ** 2 的平方根（也称为复数的模）。否则，当*x* < 0 时，abs 返回*-x*；当*x* >= 0 时，abs 返回*x*。详见表 4-4 中的 __abs__、__invert__、__neg__ 和 __pos__。 |

| all | all(*seq*, /) *seq*是一个可迭代对象。当*seq*的任何项为假时，all 返回**False**；否则，all 返回**True**。类似于操作符**and**和**or**，详见“短路运算符”，all 在知道答案后立即停止评估并返回结果；对于 all 来说，这意味着当遇到假值项时停止，但如果*seq*的所有项都为真，则会遍历整个*seq*。下面是 all 的典型玩具示例：

```py
`if` all(x>0 `for` x `in` the_numbers):
    print('all of the numbers are positive')
`else`:
    print('some of the numbers are not positive')
```

当*seq*为空时，all 返回**True**。 |

| any | any(*seq*, /) *seq*是一个可迭代对象。如果*seq*的任何项为真，则 any 返回**True**；否则，any 返回**False**。类似于操作符**and**和**or**，详见“短路运算符”，any 在知道答案后立即停止评估并返回结果；对于 any 来说，这意味着当遇到真值项时停止，但如果*seq*的所有项都为假，则会遍历整个*seq*。下面是 any 的典型玩具示例：

```py
`if` any(x<0 `for` x `in` the_numbers):
    print('some of the numbers are negative')
`else`:
    print('none of the numbers are negative')
```

当*seq*为空时，any 返回**False**。 |

| ascii | ascii(*x*, /) 类似于 repr，但在返回的字符串中转义非 ASCII 字符；结果通常与 repr 的结果相似。 |
| --- | --- |
| bin | bin(*x*, /) 返回整数*x*的二进制字符串表示。例如，bin(23)=='0b10111‘。 |
| breakpoint | breakpoint() 调用 pdb Python 调试器。如果要 breakpoint 调用替代调试器，请将 sys.breakpointhook 设置为可调用函数。 |
| callable | callable(*obj*, /) 当*obj*可调用时返回**True**；否则返回**False**。如果对象是函数、方法、类或类型，或者是具有 __call__ 方法的类的实例，则可以调用对象。另请参见表 4-1 中的 __call__。 |
| chr | chr(*code*, /) 返回长度为 1 的字符串，一个与 Unicode 中整数*code*对应的单个字符。另请参见本表中稍后的 ord。 |
| compile | compile(*source*, *filename*, *mode*) 编译字符串并返回可供 exec 或 eval 使用的代码对象。当*source*不是 Python 语法上有效时，compile 会引发 SyntaxError。当*source*是多行复合语句时，最后一个字符必须为'\n'。当*source*是表达式且结果用于 eval 时，*mode*必须为'eval'；否则，当字符串用于 exec 时，*mode*必须为'exec'（对于单个或多个语句字符串）或'single'（对于包含单个语句的字符串）。*filename*必须是一个字符串，仅在出现错误消息时使用（如果发生错误）。另请参见本表中稍后的 eval，以及“编译和代码对象”。 （compile 还接受可选参数 flags、dont_inherit、optimize 和**3.11+** _feature_version，尽管这些很少使用；有关这些参数的更多信息，请参见[在线文档](https://oreil.ly/oYj2U)。） |
| delattr | delattr(*obj*, *name*, /) 从*obj*中移除属性*name*。delattr(*obj*, '*ident*')类似于 del *obj.ident**。如果*obj*具有名为*name*的属性，仅因为其类具有该属性（通常情况下，例如，对于*obj*的*方法*），则无法从*obj*本身删除该属性。如果元类允许，您可以从*类*中删除该属性。如果您可以删除类属性，则*obj*将不再具有该属性，该类的每个其他实例也是如此。 |
| dir | dir([*obj*, ]/) 不带参数调用时，dir 返回当前作用域中绑定的所有变量名的排序列表。dir(*obj*)返回一个排序后的*obj*属性名称列表，其中包括来自*obj*类型或通过继承的属性。另请参见本表中稍后的 vars。 |
| divmod | divmod(*dividend*, *divisor*, /) 计算两个数的商和余数，并返回一对。另请参见表 4-4 中的 __divmod__。 |

| enumerate | enumerate(iterable, start=0) 返回一个新的迭代器，其项为对。对于每对，第二项是 iterable 中的对应项，而第一项是一个整数：start，start+1，start+2.... 例如，以下代码段在整数列表 L 上循环，通过对每个偶数值除以二来就地更改 L： |

```py
`for` i, num `in` enumerate(L):
    `if` num % 2 == 0:
        L[i] = num // 2
```

enumerate 是少数几个支持命名参数的内置函数之一。 |

| eval | eval(*expr*[, *globals*[, *locals*]]*,* /) 返回表达式的结果。*expr* 可以是准备好进行评估的代码对象，或者是一个字符串；如果是字符串，eval 通过内部调用 compile(*expr*, '<string>', 'eval') 来获取一个代码对象。eval 将代码对象作为表达式进行评估，使用 *globals* 和 *locals* 字典作为命名空间（当它们缺失时，eval 使用当前命名空间）。eval 不执行语句，它只评估表达式。尽管如此，eval 是危险的；除非你知道并信任 *expr* 来自于一个你确定是安全的源，否则应避免使用。另请参见 ast.literal_eval（在 “标准输入” 中介绍）和 “动态执行和 exec”。 |
| --- | --- |
| exec | exec(*statement*[, *globals*[, *locals*]]*, /*) 类似于 eval，但适用于任何语句并返回 **None**。exec 非常危险，除非你知道并信任 *statement* 来自于一个你确定是安全的源。另请参见 “语句” 和 “动态执行和 exec”。 |

| filter | filter(*func*, *seq*, /) 返回 *seq* 中使得 *func* 为真的项的迭代器。*func* 可以是接受单个参数的任何可调用对象，或者 **None**。*seq* 可以是任何可迭代对象。当 *func* 是可调用对象时，filter 对 *seq* 中的每个项调用 *func*，类似于以下的生成器表达式：

```py
(*`item`* `for` *`item`* `in` *`seq`* `if` *`func`*(*`item`*)
```

当 *func* 是 **None** 时，filter 测试真值项，就像：

```py
(*`item`* `for` *`item`* `in` *`seq`* `if` *`item`*)
```

|

| format | format(*x*, *format_spec*='', /) 返回 *x*.__format__(*format_spec*)。参见 表 4-1。 |
| --- | --- |
| getattr | getattr(*obj*, *name*[, *default*], /) 返回 *obj* 的名为 *name* 的属性。getattr(*obj*, '*ident*') 就像 *obj.ident*。当 *default* 存在且在 *obj* 中找不到 *name* 时，getattr 返回 *default* 而不是引发 AttributeError。另请参见 “对象属性和项目” 和 “属性引用基础”。 |
| globals | globals() 返回调用模块的 __dict__（即，在调用点用作全局命名空间的字典）。另请参见本表后面的 locals。 （与 locals() 不同，globals() 返回的字典是可读/写的，并且对该字典的更新等效于普通的名称定义。） |
| hasattr | hasattr(*obj*, *name*, /) 当 *obj* 没有属性 *name* 时返回 **False**（即，当 getattr(*obj, name*) 会引发 AttributeError 时）；否则，返回 True。另请参见 “属性引用基础”。 |
| 哈希 | hash(*obj,* /) 返回 *obj* 的哈希值。*obj* 可以是字典键，或集合中的项，前提是 *obj* 可以被哈希。所有相等的对象必须具有相同的哈希值，即使它们的类型不同也是如此。如果 *obj* 的类型没有定义相等比较，hash(*obj*) 通常返回 id(*obj*)（参见本表中的 id 和 Table 4-1 中的 __hash__）。 |
| 帮助 | help([*obj*, /]) 调用时如果没有 *obj* 参数，会开始一个交互式帮助会话，输入 **quit** 退出。如果给定了 *obj*，help 会打印 *obj* 及其属性的文档，并返回 **None**。在交互式 Python 会话中使用 help 可以快速查看对象功能的参考资料。 |
| 十六进制 | hex(*x*, /) 返回 int *x* 的十六进制字符串表示。参见 Table 4-4 中的 __hex__。 |
| id | id(*obj*, /) 返回 *obj* 的标识整数值。*obj* 的 id 在 *obj* 的生命周期内是唯一且恒定的^(a)（但在 *obj* 被垃圾回收后的任何时间后可能被重用，因此不要依赖存储或检查 id 值）。当一个类型或类没有定义相等比较时，Python 使用 id 来比较和哈希实例。对于任何对象 *x* 和 *y*，身份检查 *x* **is** *y* 等同于 id(*x*)==id(*y*)，但更可读且性能更佳。 |
| 输入 | input(*prompt*='', /) 将 *prompt* 写入标准输出，从标准输入读取一行，并将该行（不带 \n）作为 str 返回。在文件结尾处，input 会引发 EOFError。 |
| isinstance | isinstance(*obj, cls*, /) 当 *obj* 是类 *cls* 的实例（或 *cls* 的任何子类，或者实现了协议或 ABC *cls*）时，返回 **True**；否则返回 **False**。*cls* 可以是一个包含类的元组（或 3.10+ 使用 &#124; 运算符连接的多个类型）：在这种情况下，如果 *obj* 是 *cls* 的任何项的实例，则 isinstance 返回 **True**；否则返回 **False**。参见 “抽象基类” 和 “协议”。 |
| 是否为子类 | issubclass(*cls1*, *cls2*, /) 当 *cls1* 是 *cls2* 的直接或间接子类，或者定义了协议或 ABC *cls2* 的所有元素时，返回 **True**；否则返回 **False**。*cls1* 和 *cls2* 必须是类。*cls2* 也可以是一个包含类的元组。在这种情况下，如果 *cls1* 是 *cls2* 的任何项的直接或间接子类，则 issubclass 返回 **True**；否则返回 **False**。对于任何类 *C*，issubclass(*C*, *C*) 返回 **True**。 |

| 迭代器 | iter(*obj*, /), iter(*func*, *sentinel, /*)

创建并返回一个迭代器（一个你可以重复传递给 next 内置函数以逐个获取项的对象；参见 “迭代器”）。当带有一个参数调用时，iter(*obj*) 通常返回 *obj*.__iter__()。当 *obj* 是没有特殊方法 __iter__ 的序列时，iter(*obj*) 等同于生成器：

```py
`def` iter_sequence(obj):
    i = 0
    `while` `True`:
        `try`:
 `yield` obj[i]
 `except` IndexError:
 `raise` StopIteration
        i += 1
```

另请参见 “序列” 和 Table 4-2 中的 __iter__。

| iter *(cont.)* | 当调用时带有两个参数时，第一个参数必须是可无参数调用的可调用对象，而 iter(*func*, *sentinel*) 相当于生成器：

```py
`def` iter_sentinel(func, sentinel):
    `while` `True`:
        item = func()
        `if` item == sentinel:
            `raise` StopIteration
        `yield` item
```

# 不要在 for 语句中调用 iter

如 “for 语句” 中所讨论的，语句 **for** *x* **in** *obj* 等同于 **for** *x* **in** iter(*obj*)；因此，在这样的 **for** 语句中不要显式调用 iter。那会是多余的，因此不符合良好的 Python 风格，还会更慢且不易读。

iter 是幂等的。换句话说，当 *x* 是一个迭代器时，iter(*x*) 就是 *x*，只要 *x* 的类提供了一个体现为 **return** self 的 __iter__ 方法，就像迭代器类应该的那样。

| len | len(*container*, /) 返回 *container* 中的项数，*container* 可能是序列、映射或集合。另请参见 “容器方法” 中的 __len__。 |
| --- | --- |
| locals | locals() 返回表示当前局部命名空间的字典。将返回的字典视为只读；试图修改它可能会影响局部变量的值，也可能会引发异常。另请参见本表中的 globals 和 vars。 |

| map | map(*func, seq*, /), map(*func*, /, **seqs*)

map 在可迭代的 *seq* 的每个项上调用 *func* 并返回结果的迭代器。当你使用多个 *seqs* 可迭代对象调用 map 时，*func* 必须是一个接受 *n* 个参数的可调用对象（其中 *n* 是 *seqs* 参数的数量）。map 将重复调用 *func*，每个可迭代对象中的相应项作为参数。

例如，map(*func*, *seq*) 就像生成器表达式一样：

```py
(*`func`*(*`item`*) `for` *`item`* `in` *`seq`*).map(*`func`*, *`seq1`*, *`seq2`*)
```

就像生成器表达式一样：

```py
(*`func`*(*`a``,` `b`*) `for` *`a`*, *`b`* `in` zip(*`seq1`*, *`seq2`*))
```

当 map 的可迭代参数长度不同时，map 行为就像将较长的那些截断一样（就像 zip 本身一样）。

| max | max(*seq*, /, *, key=**None**[, default=...]), max(**args*, key=**None**[, default=...])

返回可迭代参数 *seq* 中的最大项，或多个位置参数 *args* 中的最大项。你可以传递一个 key 参数，具有与 “排序列表” 中介绍的相同语义。你还可以传递一个默认参数，即当 *seq* 为空时返回的值；当你不传递默认参数且 *seq* 为空时，max 将引发 ValueError。（当你传递 key 和/或 default 时，必须将其中一个或两个作为命名参数传递。）

| min | min(*seq,* /, *, key*=***None**[, default=...]), min(**args*, key*=***None**[, default=...])

返回可迭代参数 *seq* 中的最小项，或多个位置参数 *args* 中的最小项之一。您可以传递一个 key 参数，其语义与 “排序列表” 中介绍的相同。您还可以传递一个 default 参数，如果 *seq* 为空则返回该值；当不传递 default 且 *seq* 为空时，min 将引发 ValueError 异常。（当您传递 key 和/或 default 时，必须将其中一个或两者作为命名参数传递。） |

| next | next(*it*[, *default*], /) 从迭代器 *it* 中返回下一个项目，并使其前进到下一个项目。当 *it* 没有更多项目时，next 返回 *default*；当没有传递 *default* 时，next 抛出 StopIteration 异常。 |
| --- | --- |
| oct | oct(*x*, /) 将整数 *x* 转换为八进制字符串。另见 表 4-4 中的 __oct__。 |

| open | open(file, mode='r', buffering=-1) 打开或创建文件并返回新的文件对象。open 还接受许多更多的可选参数；详情请参见 “io 模块”。 |

open 是少数几个可以使用命名参数调用的内置函数。 |

| ord | ord(*ch*, /) 返回一个介于 0 和 sys.maxunicode（包括）之间的整数，对应于单字符 str 参数 *ch*。另见本表中较早的 chr。 |
| --- | --- |
| pow | pow(*x*, *y*[, *z*], /) 当 *z* 存在时，pow(*x*, *y*, *z*) 返回 (*x* ** *y*) % *z*。当 *z* 不存在时，pow(*x*, *y*) 返回 *x* ** *y*。另见 表 4-4 中的 __pow__。当 *x* 是整数且 *y* 是非负整数时，pow 返回一个整数，并使用 Python 的整数全值范围（尽管对于大的整数 *x* 和 *y* 的值，计算 pow 可能需要一些时间）。当 *x* 或 *y* 是浮点数，或 *y* < 0 时，pow 返回一个浮点数（或复数，当 *x* < 0 且 *y* 不等于 int(y) 时）；在这种情况下，如果 *x* 或 *y* 太大，pow 将引发 OverflowError。 |
| print | print(/, **args*, sep=' ', end='\n', file=sys.stdout, flush=**False**) 使用 str 格式化并发送到文件流，对于 *args* 的每个项（如果有的话），以 sep 分隔，所有项打印完毕后，根据 flush 的真假决定是否刷新流。 |

| range | range([start=0, ]stop[, step=1], /) 返回一个整数迭代器，表示算术级数：

```py
start, start+step, start+(2*step), ...
```

当缺少 start 参数时，默认为 0。当缺少 step 参数时，默认为 1。当 step 为 0 时，range 抛出 ValueError。当 step > 0 时，最后一个项是最大的 start+(*i*step) 严格小于 stop。当 step < 0 时，最后一个项是最小的 start+(*i*step) 严格大于 stop。当 start 大于或等于 stop 且 step 大于 0，或当 start 小于或等于 stop 且 step 小于 0 时，迭代器为空。否则，迭代器的第一个项总是 start。

当你需要一个算术级数的整数列表时，请调用 list(range(...))。 |

| repr | repr(*obj*, /) 返回*obj*的完整且明确的字符串表示形式。在可行的情况下，repr 返回一个字符串，您可以将其传递给 eval 以创建一个与*obj*具有相同值的新对象。还请参阅表 8-1 中的 str 和表 4-1 中的 __repr__。 |
| --- | --- |
| reversed | reversed(*seq*, /) 返回一个新的迭代器对象，该对象按逆序产生*seq*（*seq*必须明确地是一个序列，而不仅仅是任何可迭代对象）的项目。 |
| round | round(number, ndigits=0) 返回一个浮点数，其值为 int 或浮点数 number 四舍五入到小数点后 ndigits 位（即，最接近 number 的 10**-ndigits 的倍数）。当两个这样的倍数与 number 等距时，round 返回*偶数*倍数。由于当今的计算机以二进制而不是十进制表示浮点数，大多数 round 的结果都不精确，正如文档中的[教程](https://oreil.ly/qHMNz)详细解释的那样。还请参阅“decimal 模块”和大卫·戈德伯格（David Goldberg）关于浮点算术的著名的与语言无关的[文章](https://oreil.ly/TVFMb)。 |
| setattr | setattr(*obj*, *name*, *value*, /) 将*obj*的属性*name*绑定到*value*。setattr(*obj*, '*ident*', *val*) 类似于*obj*.*ident*=*val*。还请参阅此表中早期的 getattr，“对象属性和项目”以及“设置属性”。 |

| sorted | sorted(*seq*, /, *, key=**None**, reverse=**False**) 返回一个与可迭代*seq*中的相同项目以排序顺序排列的列表。与下面相同：

```py
`def` sorted(*`seq`*, /, *, key=`None`, reverse=`False`):
    result = list(*`seq`*)
    result.sort(key, reverse)
    `return` result
```

查看“列表排序”以了解参数的含义。如果要传递键（key）和/或反转（reverse），*必须*按名称传递。

| sum | sum(*seq*, /, start=0) 返回可迭代*seq*（应为数字，并且特别是不能为字符串）的项目的总和，加上 start 的值。当*seq*为空时，返回 start。要对字符串可迭代对象进行“求和”（连接），请使用''.join(*iterofstrs*），如表 8-1 和“从片段构建字符串”中所述。 |
| --- | --- |
| vars | vars([*obj*, ]*/*) 调用时不带参数时，vars 返回一个字典，其中包含在当前范围（如此表中较早涵盖的局部变量）中绑定的所有变量。请将此字典视为只读。vars(*obj*) 返回一个字典，其中包含当前在*obj*中绑定的所有属性，类似于 dir，此表中较早涵盖的内容。此字典可以是可修改的也可以是不可修改的，具体取决于*obj*的类型。 |
| zip | zip(*seq*, /, **seqs*, strict*=***False**) Returns an iterator of tuples, where the *n*th tuple contains the *n*th item from each of the argument iterables. You must call zip with at least one (positional) argument, and all positional arguments must be iterable. zip returns an iterator with as many items as the shortest iterable, ignoring trailing items in the other iterable objects. **3.10+** When the iterables have different lengths and strict is **True**, zip raises ValueError once it reaches the end of the shortest iterable. See also map earlier in this table and zip_longest in Table 8-10. |
| ^(a) Otherwise arbitrary; often, an implementation detail, *obj*’s address in memory. |

# sys 模块

sys 模块的属性绑定到提供关于 Python 解释器状态或直接影响解释器的数据和函数。 Table 8-3 涵盖了 sys 的最常用属性。大多数 sys 属性专为调试器、分析器和集成开发环境使用；详细信息请参阅 [online docs](https://oreil.ly/2KBRg)。

平台特定的信息最好使用 platform 模块访问，本书不涵盖此模块；详细信息请参阅 [online docs](https://oreil.ly/YJKQD)。

Table 8-3\. Functions and attributes of the sys module

| argv | The list of command-line arguments passed to the main script. argv[0] is the name of the main script,^(a) or '-c' if the command line used the **-c** option. See “The argparse Module” for one good way to use sys.argv. |
| --- | --- |
| audit | audit(*event*, /, **args*) Raises an *audit event* whose name is str *event* and whose arguments are *args*. The rationale for Python’s audit system is laid out in exhaustive detail in [PEP 578](https://oreil.ly/pMcEY); Python itself raises the large variety of events listed in the [online docs](https://oreil.ly/SjLW1). To *listen* for events, call sys.addaudithook(*hook*), where *hook* is a callable whose arguments are a str, the event’s name, followed by arbitrary positional arguments. For more details, see the [docs](https://oreil.ly/4os3i). |
| buil⁠t⁠i⁠n⁠_​m⁠o⁠d⁠ule_names | A tuple of strs: the names of all the modules compiled into this Python interpreter. |

| displayhook | displayhook(*value*, /) In interactive sessions, the Python interpreter calls displayhook, passing it the result of each expression statement you enter. The default displayhook does nothing when *value* is **None**; otherwise, it saves *value* in the built-in variable whose name is _ (an underscore) and displays it via repr:

```py
`def` _default_sys_displayhook(value, /):
    `if` value `is` `not` `None`:
        __builtins__._ = value
        print(repr(value))
```

你可以重新绑定 sys.displayhook 以更改交互行为。原始值可通过 sys.__displayhook__ 获取。

| dont_wri⁠t⁠e⁠_​b⁠y⁠t⁠ecode | 当为 **True** 时，Python 在导入源文件（扩展名为 *.py*）时不将字节码文件（扩展名为 *.pyc*）写入磁盘。 |
| --- | --- |
| excepthook | excepthook(*type*, *value*, *traceback*, /) 当异常没有被任何处理程序捕获并传播到调用堆栈的最顶层时，Python 调用 excepthook，传递给它异常类、对象和回溯，如 “异常传播” 中所述。默认的 excepthook 显示错误和回溯。您可以重新绑定 sys.excepthook 来更改未捕获异常（在 Python 返回交互循环或终止之前）的显示和/或记录方式。原始值可通过 sys.__excepthook__ 获取。 |
| exception | exception() 3.11+ 在 **except** 子句中调用时，返回当前异常实例（等效于 sys.exc_info()[1]）。 |

| exc_info | exc_info() 当前线程正在处理异常时，exc_info 返回一个包含三个元素的元组：异常的类、对象和回溯（traceback）。当线程没有处理异常时，exc_info 返回 (**None**, **None**, **None**)。要显示来自回溯的信息，请参阅 “回溯模块”。

# 持有回溯对象可能使一些垃圾成为不可回收

traceback 对象间接地持有对调用堆栈上所有变量的引用；如果您持有回溯的引用（例如通过将变量绑定到 exc_info 返回的元组间接地持有），Python 必须保留可能被垃圾回收的数据。确保对回溯对象的任何绑定持续时间很短，例如使用 **try**/**finally** 语句（在 “try/finally” 中讨论）。如果必须持有异常 *e* 的引用，请清除 *e* 的回溯：*e*.__traceback__=**None**。^(b) |

|

| exit | exit(*arg*=0, /) 触发一个 SystemExit 异常，通常在执行**try**/**finally**语句、**with**语句以及 atexit 模块安装的清理处理程序后终止执行。当 *arg* 是一个整数时，Python 使用 *arg* 作为程序的退出代码：0 表示成功终止；任何其他值表示程序未成功终止。大多数平台要求退出代码在 0 到 127 之间。当 *arg* 不是整数时，Python 将 *arg* 打印到 sys.stderr，并且程序的退出代码为 1（通用的“未成功终止”代码）。 |
| --- | --- |
| float_info | 一个只读对象，其属性保存了该 Python 解释器中浮点类型的底层实现细节。详细信息请参阅 [在线文档](https://oreil.ly/9vMpw)。 |
| g⁠e⁠t⁠r⁠e⁠c⁠u⁠r⁠s⁠i⁠o⁠n​l⁠i⁠m⁠i⁠t | getrecursionlimit() 返回 Python 调用堆栈深度的当前限制。也请参阅 “递归” 和本表后面的 setrecursionlimit。 |
| getrefcount | getrefcount(*obj*, /) 返回*obj*的引用计数。引用计数在“垃圾回收”中有介绍。 |
| getsizeof | getsizeof(*obj*[, *default*], /) 返回*obj*的大小，以字节为单位（不包括*obj*可能引用的任何项或属性），或在*obj*无法提供其大小的情况下返回*default*（在后一种情况下，如果*default*不存在，则 getsizeof 引发 TypeError）。 |
| maxsize | 在此 Python 版本中对象的最大字节数（至少为 2 ** 31 - 1，即 2147483647）。 |
| maxunicode | 在此 Python 版本中 Unicode 字符的最大代码点；当前始终为 1114111（0x10FFFF）。Python 使用的 Unicode 数据库版本在 unicodedata.unidata_version 中。 |
| modules | 一个字典，其项为所有已加载模块的名称和模块对象。有关 sys.modules 的更多信息，请参见“模块加载”。 |
| path | 一个字符串列表，指定 Python 在查找要加载的模块时搜索的目录和 ZIP 文件。有关 sys.path 的更多信息，请参见“在文件系统中搜索模块”。 |
| platform | 一个字符串，指定运行此程序的平台名称。典型的值是简短的操作系统名称，如'darwin'、'linux2'和'win32'。对于 Linux，检查 sys.platform.startswith('linux')以实现在不同 Linux 版本间的可移植性。还请参阅模块[platform](https://oreil.ly/LH4IP)的在线文档，本书不涵盖该部分。 |

| ps1, ps2 | ps1 和 ps2 分别指定主提示符和次要提示符字符串，初始值分别为 >>> 和 ...。这些 sys 属性仅存在于交互式解释器会话中。如果将任何属性绑定到非字符串对象*x*，Python 将在每次输出提示时调用该对象的 str(*x*)方法来生成提示符。此功能允许动态提示：编写一个定义了 __str__ 方法的类，然后将该类的实例分配给 sys.ps1 和/或 sys.ps2。例如，要获取编号提示符：

```py
>>> `import` `sys`
>>> `class` Ps1(object):
...     `def` __init__(self):
...         self.p = 0
...     `def` __str__(self):
...         self.p += 1
...         `return` f'[{self.p}]>>> '
...
>>> `class` Ps2(object):
...     `def` __str__(self):
...         `return` f'[{sys.ps1.p}]... '
...
>>> sys.ps1, sys.ps2 = Ps1(), Ps2()
[1]>>> (2 +
[1]... 2)
```

```py
4
```

```py
[2]>>>
```

|

| s⁠e⁠t⁠r⁠e⁠c⁠u⁠r⁠s⁠i⁠o⁠n​l⁠i⁠m⁠i⁠t | setrecursionlimit(*limit*, /) 设置 Python 调用栈深度限制（默认为 1000）。该限制可防止递归无限扩展导致 Python 崩溃。对于依赖深度递归的程序，可能需要提高此限制，但大多数平台无法支持非常大的调用栈深度限制。更有用的是，*降低*此限制可以帮助您在测试和调试过程中检查程序是否能够优雅地降级，而不是在几乎无限递归的情况下突然崩溃并引发 RecursionError。还请参见“递归”和本表中 getrecursionlimit 的前文。 |
| --- | --- |

| stdin, stdout, | 

stderr | stdin、stdout 和 stderr 是预定义的类似文件的对象，分别对应于 Python 的标准输入、输出和错误流。您可以将 stdout 和 stderr 重新绑定到打开以供写入的类似文件的对象（提供接受字符串参数的 write 方法的对象），以重定向输出和错误消息的目标。您可以将 stdin 重新绑定到打开以供读取的类似文件的对象（提供返回字符串的 readline 方法的对象），以重定向内置函数 input 读取的源。原始值可用作 __stdin__、__stdout__ 和 __stderr__。我们在 “io 模块” 中讨论文件对象。 |

| tracebacklimit | 未处理异常的回溯显示的最大级数。默认情况下，此属性未定义（即没有限制）。当 sys.tracebacklimit <= 0 时，Python 仅打印异常类型和值，而不打印回溯。 |
| --- | --- |
| version | 描述 Python 版本、构建号和日期以及所使用的 C 编译器的字符串。仅在日志记录或交互式输出时使用 sys.version；要执行版本比较，请使用 sys.version_info。 |
| version_info | 运行中的 Python 版本的主要、次要、微版本、发布级别和序列号字段的命名元组。例如，在 Python 3.10 的第一个正式版之后，sys.version_info 是 sys.version_info(major=3, minor=10, micro=0, releaselevel='final', serial=0)，相当于元组 (3, 10, 0, 'final', 0)。此形式被定义为版本间直接可比较；若要检查当前运行版本是否大于或等于 3.8，可以测试 sys.version_info[:3] >= (3, 8, 0)。（不要对 *string* sys.version 进行字符串比较，因为字符串 "3.10" 会被认为小于 "3.9"！） |
| ^(a) 当然，它也可以是脚本的路径，如果是这样，它也可以是对它的符号链接，如果是这样，它就是你给 Python 的^(b)。本书的一位作者在 pyparsing 中记住了返回值和引发的异常时，遇到了这个问题：缓存的异常回溯包含了许多对象引用，并且干扰了垃圾收集。解决方案是在将异常放入缓存之前清除异常的回溯。 |

# **copy 模块**

如 “赋值语句” 中所讨论的，Python 中的赋值不会 *复制* 被分配的右侧对象。相反，赋值 *添加引用* 到 RHS 对象。当您想要对象 *x* 的 *副本* 时，请求 *x* 自身的副本，或者请求 *x* 的类型创建从 *x* 复制的新实例。如果 *x* 是列表，则 list(*x*) 返回 *x* 的副本，*x*[:] 也是如此。如果 *x* 是字典，则 dict(*x*) 和 *x*.copy() 返回 *x* 的副本。如果 *x* 是集合，则 set(*x*) 和 *x*.copy() 返回 *x* 的副本。在每种情况下，本书的作者更倾向于使用统一和可读的惯用法来调用类型，但在 Python 社区中对此风格问题没有共识。

复制模块提供了一个复制函数用于创建和返回多种类型对象的副本。正常副本，例如对于列表 *x* 返回的 list(*x*) 和对于任何 *x* 返回的 copy.copy(*x*)，称为 *浅层* 副本：当 *x* 引用其他对象（作为项或属性时），*x* 的普通（浅层）副本具有对相同对象的不同引用。然而，有时您需要一个 *深层* 副本，在这种副本中，引用的对象被递归地进行深层复制（幸运的是，这种需求很少，因为深层复制可能需要大量内存和时间）；对于这些情况，复制模块还提供了一个 deepcopy 函数。这些函数在 表 8-4 中进一步讨论。

表 8-4\. 复制模块函数

| copy | copy(*x*) 创建并返回 *x* 的浅层副本，适用于多种类型的 *x*（模块、文件、帧以及其他内部类型，但不支持）。当 *x* 是不可变的时候，作为优化，copy.copy(*x*) 可能返回 *x* 本身。一个类可以通过拥有特殊方法 __copy__(self)，返回一个新对象，即 self 的浅层副本，来自定义 copy.copy 复制其实例的方式。 |
| --- | --- |

| deepcopy | deepcopy(*x*,[memo]) 对 *x* 进行深层复制并返回。深层复制意味着对引用图进行递归遍历。请注意，为了重现图形的确切形状，当在遍历过程中多次遇到对同一对象的引用时，您不能制作不同的副本；相反，必须使用对同一复制对象的 *引用*。考虑以下简单示例：

```py
sublist = [1,2]
original = [sublist, sublist]
thecopy = copy.deepcopy(original)
```

original[0] **is** original[1] is **True**（即，original 的两个项引用同一对象）。这是 original 的一个重要属性，任何宣称“一个副本”的东西都必须保留它。copy.deepcopy 的语义确保 thecopy[0] **is** thecopy[1] 也是 **True**：original 和 thecopy 的引用图具有相同的形状。避免重复复制具有一个重要的有益副作用：它防止引用图中存在循环时会发生的无限循环。

copy.deepcopy 接受第二个可选参数：memo，一个将每个已复制对象的 id 映射到其副本的字典。memo 被 deepcopy 的所有递归调用传递给自己；如果您需要获取原始对象和副本之间的对应映射（通常作为一个最初为空的字典），您也可以显式地传递它（memo 的最终状态将是这样一个映射）。

类可以通过拥有特殊方法 __deepcopy__(self, memo) 来定制 copy.deepcopy 复制其实例的方式，该方法返回一个新对象，即 self 的深层副本。当 __deepcopy__ 需要深层复制一些引用对象 *subobject* 时，必须通过调用 copy.deepcopy(*subobject*, memo) 来实现。当一个类没有特殊方法 __deepcopy__ 时，对该类的实例使用 copy.deepcopy 也会尝试调用特殊方法 __getinitargs__、__getnewargs__、__getstate__ 和 __setstate__，详见 “实例的序列化”。

# collections 模块

collections 模块提供了有用的类型，这些类型是集合（即，容器），以及 “抽象基类” 中涵盖的 ABC。自 Python 3.4 起，ABC 在 collections.abc 中提供；为了向后兼容，直到 Python 3.9 仍然可以直接在 collections 中访问，但此功能在 3.10 中已移除。

## ChainMap

ChainMap 将多个映射“链接”在一起；给定 ChainMap 实例 *c*，访问 *c*[*key*] 返回具有该键的第一个映射中的值，而对 *c* 的所有更改仅影响 *c* 中的第一个映射。为了进一步解释，您可以如下近似：

```py
`class` ChainMap(collections.abc.MutableMapping):
    `def` __init__(self, *maps):
        self.maps = list(maps)
        self._keys = set()
        `for` m `in` self.maps:
            self._keys.update(m)
    `def` __len__(self): `return` len(self._keys)
    `def` __iter__(self): `return` iter(self._keys)
    `def` __getitem__(self, key):
        `if` key `not` `in` self._keys: `raise` KeyError(key)
        `for` m `in` self.maps:
            `try`: `return` m[key]
            `except` KeyError: `pass`
    `def` __setitem__(self, key, value):
        self.maps[0][key] = value
        self._keys.add(key)
    `def` __delitem__(self, key):
        `del` self.maps[0][key]
        self._keys = set()
        `for` m `in` self.maps:
            self._keys.update(m)
```

其他方法可以为了效率而定义，但这是 MutableMapping 要求的最小集合。更多详情和一组关于如何使用 ChainMap 的示例，请参阅 [在线文档](https://oreil.ly/WgfFo)。

## 计数器

计数器是 int 值的 dict 的子类，用于*计数*键已经被看到多少次（尽管允许值 <= 0）；它大致相当于其他语言中称为“bag”或“multiset”类型的类型。通常，Counter 实例是从其项是可散列的可迭代对象构建的：*c* = collections.Counter(*iterable*)。然后，您可以使用 *iterable* 的任何项对 *c* 进行索引以获取该项出现的次数。当您使用任何缺失的键对 *c* 进行索引时，结果为 0（要 *移除* *c* 中的条目，请使用 **del** *c*[*entry*]；将 *c*[*entry*]=0 留下 *c* 中的 *entry*，其值为 0）。

*c* 支持字典的所有方法；特别是，*c*.update(*otheriterable*) 更新所有计数，根据 *otheriterable* 中的出现次数递增它们。例如：

```py
>>> c = collections.Counter('moo')
>>> c.update('foo')
```

离开 *c*['o'] 得到 4，*c*['f'] 和 *c*['m'] 分别得到 1。注意，从 *c* 中移除条目（用 **del**）可能 *不会* 减少计数器，但是减去（在下表中描述）会：

```py
>>> `del` c['foo']    
>>> c['o']
```

```py
4
```

```py
>>> c.subtract('foo')
>>> c['o']
```

```py
2
```

除了字典方法外，*c* 还支持详细说明的额外方法，见 表 8-5。

表 8-5\. 计数器实例 c 的方法

| elements | *c*.elements() 以任意顺序产生 *c* 中 *c*[*key*]>0 的键，每个键产生的次数与其计数相同。 |
| --- | --- |
| mo⁠s⁠t⁠_​c⁠o⁠m⁠mon | *c*.most_common([*n*, /]) 返回 *c* 中计数最高的前 *n* 个键的列表（如果省略 *n*，则为所有键），按计数降序排序（具有相同计数的键之间的“平局”将任意解决）；每对形式为 (*k*, *c*[*k*])，其中 *k* 是 *c* 中前 *n* 个最常见的键之一。 |
| subtract | *c*.subtract(*iterable=***None**, /, ***kwds*) 类似于 *c*.update(*iterable*) 的“反向操作”——即减去计数而不是增加它们。*c* 中的结果计数可以 <= 0。 |
| total | *c.*total() **3.10+** 返回所有个体计数的总和。等同于 sum(*c*.values())。 |

Counter 对象支持常见的算术运算符，如 +、-、& 和 |，用于加法、减法、并集和交集。详见 [在线文档](https://oreil.ly/MylAp) 获取更多详情和一系列关于如何使用 Counter 的实用示例。

## OrderedDict

OrderedDict 是 dict 的子类，具有额外的方法来按插入顺序访问和操作条目。*o*.popitem() 删除并返回最近插入的键的条目；*o*.move_to_end(*key*, last=**True**) 将具有键 *key* 的条目移动到末尾（当 last 为 **True** 时，默认为末尾）或开头（当 last 为 **False** 时）。两个 OrderedDict 实例之间的相等性测试是顺序敏感的； OrderedDict 实例与 dict 或其他映射的相等性测试则不是。自 Python 3.7 起，dict 插入顺序已保证保持不变：许多以前需要 OrderedDict 的用法现在可以直接使用普通的 Python dict。两者之间仍存在一个显著的差异，即 OrderedDict 与其他 OrderedDict 的相等性测试是顺序敏感的，而 dict 的相等性测试则不是。详见 [在线文档](https://oreil.ly/JSvPS) 获取更多详情和一系列关于如何使用 OrderedDict 的示例。

## defaultdict

defaultdict 是 dict 的扩展，每个实例都添加了一个名为 default_factory 的属性。当 defaultdict 实例 *d* 的 *d*.default_factory 值为 **None** 时，*d* 表现得和 dict 完全一样。否则，*d*.default_factory 必须是一个可调用的无参数函数，*d* 的行为和 dict 类似，除非你使用一个不存在于 *d* 中的键 *k* 访问 *d*。在这种情况下，索引 *d*[*k*] 调用 *d*.default_factory()，将结果分配为 *d*[*k*] 的值，并返回结果。换句话说，defaultdict 类型的行为很像下面这样用 Python 编写的类：

```py
`class` defaultdict(dict):
    `def` __init__(self, default_factory=`None`, *a, **k):
        super().__init__(*a, **k)
        self.default_factory = default_factory
    `def` __getitem__(self, key):
        `if` key `not` `in` self `and` self.default_factory `is` `not` `None`:
            self[key] = self.default_factory()
        `return` dict.__getitem__(self, key)
```

正如这个 Python 等效物所暗示的，要实例化 defaultdict，通常需要将额外的第一个参数传递给它（在任何其他参数之前，位置和/或命名，如果有的话，传递给普通 dict）。额外的第一个参数将成为 default_factory 的初始值；你也可以稍后访问和重新绑定 default_factory，尽管在正常的 Python 代码中这样做是不常见的。

defaultdict 的所有行为基本上与此 Python 等效内容暗示的行为相同（除了 str 和 repr，它们返回的字符串与 dict 返回的字符串不同）。命名方法，如 get 和 pop，并不受影响。与键相关的所有行为（方法 keys、迭代、通过运算符 in 进行成员测试等）完全反映了当前容器中的键（无论是显式放置还是通过调用 default_factory 隐式放置）。

defaultdict 的典型用途是，例如将 default_factory 设置为 list，以创建从键到值列表的映射：

```py
`def` make_multi_dict(items):
    d = collections.defaultdict(list)
    `for` key, value `in` items:
        d[key].append(value)
    `return` d
```

使用任何元素为形式为 (*key*, *value*) 对的可迭代对象调用 make_multi_dict 函数时，此函数返回一个映射，将每个键关联到伴随其出现的一个或多个值列表（如果要得到纯字典结果，请将最后一条语句更改为 return dict(d)——这很少是必要的）。

如果不希望结果中有重复项，并且每个 *value* 都是可哈希的，请使用 collections.defaultdict(set)，并在循环中添加而不是追加。²

## deque

deque 是一种序列类型，其实例为“双端队列”（在任一端的添加和移除都很快且线程安全）。deque 实例 *d* 是可变序列，可选择具有最大长度，并且可以进行索引和迭代（但是，*d* 不能进行切片；它只能一次索引一个项目，无论是用于访问、重新绑定还是删除）。如果 deque 实例 *d* 具有最大长度，当向 *d* 的任一侧添加项目以使 *d* 的长度超过该最大长度时，将从另一侧静默删除项目。

deque 特别适用于实现先进先出（FIFO）队列。³ deque 也很适合维护“最新 *N* 个看到的事物”，在其他一些语言中也称为环形缓冲区。

表 8-6 列出了 deque 类型提供的方法。

表 8-6\. deque 方法

| deque | deque(*seq*=(), /, maxlen=**None**) *d* 的初始项目是 *seq* 的项目，顺序相同。 *d*.maxlen 是只读属性：当其值为 **None** 时，*d* 没有最大长度；当为整数时，必须 >=0。 *d* 的最大长度是 *d*.maxlen。 |
| --- | --- |
| append | *d*.append(*item*, /) 将 *item* 追加到 *d* 的右侧（末尾）。 |
| appendleft | *d*.appendleft(*item*, /) 将 *item* 追加到 *d* 的左侧（开头）。 |
| clear | *d*.clear() 从 *d* 中移除所有项目，使其为空。 |
| extend | *d*.extend(*iterable*, /) 将 *iterable* 中的所有项目追加到 *d* 的右侧（末尾）。 |
| extendleft | *d*.extendleft(*iterable*, /) 将 *iterable* 中的所有项目以相反的顺序追加到 *d* 的左侧（开头）。 |
| pop | *d*.pop() 从 *d* 中移除并返回最后一个（最右边的）项目。如果 *d* 是空的，则引发 IndexError 异常。 |
| popleft | *d*.popleft() 从 *d* 中移除并返回第一个（最左边的）项目。如果 *d* 是空的，则引发 IndexError 异常。 |
| rotate | *d*.rotate(*n*=1, /) 将 *d* 向右旋转 *n* 步（如果 *n* < 0，则向左旋转）。 |

# 避免对 deque 进行索引或切片

deque 主要用于从 deque 的开头或结尾访问、添加和删除项目的情况。虽然可以对 deque 进行索引或切片，但当使用 deque[i] 形式访问内部值时，性能可能为 O(n)（而列表为 O(1)）。如果必须访问内部值，请考虑使用列表。

# functools 模块

functools 模块提供支持 Python 函数式编程的函数和类型，列在 表 8-7 中。

表 8-7\. functools 模块的函数和属性

| cach⁠e⁠d⁠_​p⁠r⁠o⁠p⁠erty | cached_property(func) 3.8+ 一个缓存版本的属性修饰器。第一次计算属性时会缓存返回的值，因此后续调用可以返回缓存的值，而不是重复计算属性。cached_property 使用线程锁确保在多线程环境中只执行一次属性计算。^(a) |
| --- | --- |

| lru_cache, cache | lru_cache(max_size=128, typed=**False**), cache()

适合装饰一个函数的 *memoizing* 装饰器，其所有参数都是可散列的，向函数添加一个缓存，存储最后的 max_size 个结果（max_size 应为 2 的幂，或 **None** 以保留所有先前的结果）；当再次使用与缓存中的参数调用装饰函数时，它将立即返回先前缓存的结果，绕过底层函数的主体代码。当 typed 设置为 **True** 时，相等但类型不同的参数（例如 23 和 23.0）将分别缓存。3.9+ 如果将 max_size 设置为 **None**，则使用 cache 替代。有关更多详细信息和示例，请参阅 [在线文档](https://oreil.ly/hLRYd)。3.8+ lru_cache 也可作为无括号的装饰器使用。

| partial | partial(*func*, /, **a, **k*) 返回一个可调用对象 *p*，它与 *func*（任何可调用对象）类似，但某些位置参数和/或命名参数已绑定到给定的 *a* 和 *k* 的值。换句话说，*p* 是 *func* 的部分应用，通常也被称为 *func* 对给定参数的 *柯里化*（在数学家 Haskell Curry 的荣誉上，尽管正确性有争议，但用语色彩丰富）。例如，假设我们有一个数字列表 L，并希望将负数剪切为 0。一种方法是：

```py
L = map(functools.partial(max, 0), L)
```

作为**lambda**使用的替代片段：

```py
L = map(`lambda` x: max(0, x), L)
```

以及最简洁的方法，即列表推导：

```py
L = [max(0, x) `for` x `in` L]
```

functools.partial 在需要回调的情况下特别适用，例如某些 GUI 和网络应用的事件驱动编程。

partial 返回一个带有 func（包装函数）、args（预绑定的位置参数元组）和 keywords（预绑定的命名参数字典，或者 **None**）属性的可调用对象。 |

| reduce | reduce(*func*, *seq*[, *init*], /) 将 *func* 应用于 *seq* 的项，从左到右，将可迭代对象减少为单个值。*func* 必须是可调用的，接受两个参数。reduce 首先将 *func* 应用于 *seq* 的前两个项，然后将第一次调用的结果和第三个项进行调用，依此类推，返回最后一次调用的结果。如果存在 *init*，reduce 在 *seq* 的第一项之前使用它（如果有的话）。如果 *init* 缺失，*seq* 必须非空。如果 *init* 缺失且 *seq* 只有一个项，则 reduce 返回 *seq*[0]。类似地，如果存在 *init* 且 *seq* 为空，则 reduce 返回 *init*。因此，reduce 大致等同于：

```py
`def` reduce_equiv(func, seq, init=`None`):
    seq = iter(seq)
    `if` init `is` `None`:
        init = next(seq)
    `for` item `in` seq: 
        init = func(init, item)
    `return` init
```

一个 reduce 的示例用法是计算一系列数字的乘积：

```py
prod=reduce(operator.mul, seq, 1)
```

|

| singledispatch, singledispatchmethod | 函数装饰器，支持具有不同类型的第一个参数的方法的多个实现。有关详细描述，请参阅 [在线文档](https://oreil.ly/1nle3)。 |
| --- | --- |
| total_ordering | 一个类装饰器，适用于装饰至少提供一个不等比较方法（如 __lt__）的类，并且最好也提供 __eq__。基于类的现有方法，total_ordering 类装饰器为类添加了所有其他不等比较方法，这些方法在类本身或任何超类中都没有实现，从而避免您为它们添加样板代码。 |
| wraps | wraps(wrapped) 适用于装饰包装另一个函数 wrapped 的函数的装饰器（通常是在另一个装饰器内部的嵌套函数）。wraps 复制了 wrapped 的 __name__、__doc__ 和 __module__ 属性到装饰函数上，从而改善了内置函数 help 和 doctests 的行为，详情请参阅 “The doctest Module”。 |
| ^(a) 在 Python 版本 3.8 到 3.11 中，cached_property 是使用类级锁实现的。因此，它对于类或任何子类的所有实例都同步，而不仅仅是当前实例。因此，在多线程环境中，cached_property 可能会降低性能，*不*推荐使用。 |

# heapq 模块

heapq 模块使用 [*min-heap*](https://oreil.ly/RU6F_) 算法，在插入和提取项目时将列表保持“几乎排序”的顺序。heapq 的操作速度比每次插入后调用列表的排序方法要快得多，比 bisect（在 [在线文档](https://oreil.ly/nZ_9m) 中介绍）要快得多。对于许多目的，比如实现“优先队列”，heapq 支持的几乎排序顺序和完全排序顺序一样好，并且建立和维护速度更快。heapq 模块提供了 Table 8-8 中列出的函数。

Table 8-8\. heapq 模块的函数列表

| heapify | heapify(*alist*, /) 根据需要重新排列列表 *alist*，使其满足（最小）堆条件：

+   对于任何 *i* >= 0：

+   alist[*i*] <= alist[2 * *i* + 1] and

+   alist[*i*] <= alist[2 * *i* + 2]

+   只要所有相关的索引都小于 len(*alist*)。

如果一个列表满足（最小）堆条件，则列表的第一个项是最小的（或者相等最小的）项。排序后的列表满足堆条件，但列表的许多其他排列也可以满足堆条件，而不需要列表完全排序。heapify 运行时间为 O(len(*alist*))。

| heappop | heappop(*alist*, /) 移除并返回 *alist* 中最小（第一个）项，一个满足堆条件的列表，并重新排列 *alist* 的一些剩余项，以确保在移除后仍满足堆条件。heappop 运行时间为 O(log(len(*alist*)))。 |
| --- | --- |
| heappush | heappush(*alist*, *item*, /) 在满足堆条件的 *alist* 中插入 *item*，并重新排列一些 *alist* 的项，以确保在插入后仍满足堆条件。heappush 运行时间为 O(log(len(*alist*)))。 |

| heappushpop | heappushpop(*alist*, *item*, /) 逻辑上等价于先执行 heappush，然后执行 heappop，类似于：

```py
`def` heappushpop(alist, item):
    heappush(alist, item)
    `return` heappop(alist)
```

heappushpop 运行时间为 O(log(len(*alist*)))，通常比逻辑上等价的函数更快。heappushpop 可以在空的 *alist* 上调用：在这种情况下，它返回 *item* 参数，就像当 *item* 小于 *alist* 的任何现有项时一样。

| heapreplace | heapreplace(*alist*, *item*, /) 逻辑上等价于先执行 heappop，然后执行 heappush，类似于：

```py
`def` heapreplace(alist, item):
    `try`: `return` heappop(alist)
    `finally`: heappush(alist, item)
```

heapreplace 运行时间为 O(log(len(*alist*)))，通常比逻辑上等价的函数更快。heapreplace 不能在空的 *alist* 上调用：heapreplace 总是返回已经在 *alist* 中的项，而不是刚刚被推送到 *alist* 上的 *item*。

| merge | merge(**iterables*) 返回一个迭代器，按照从小到大的顺序（排序顺序）产生 *iterables* 的项，其中每个项必须是从小到大排序的。 |
| --- | --- |
| nlargest | nlargest(*n*, *seq*, /, key=**None**) 返回一个逆序排列的列表，包含可迭代序列 *seq* 的 *n* 个最大项（或者少于 *n* 项，如果 *seq* 的项少于 *n* 项）；当 *n* 相对于 len(*seq*) 足够小时比 sorted(*seq*, reverse=**True**)[:*n*] 更快。您也可以指定一个（命名或位置的）key= 参数，就像您可以为 sorted 指定的那样。 |
| nsmallest | nsmallest(*n*, *seq*, /, key=**None**) 返回一个已排序的列表，包含可迭代序列 *seq* 的 *n* 个最小项（或者少于 *n* 项，如果 *seq* 的项少于 *n* 项）；当 *n* 相对于 len(*seq*) 足够小时比 sorted(*seq*)[:*n*] 更快。您也可以指定一个（命名或位置的）key= 参数，就像您可以为 sorted 指定的那样。 |
| ^(a) 要了解特定的 *n* 值和 len(*seq*) 如何影响 nlargest、nsmallest 和 sorted 在您的具体 Python 版本和机器上的时间，可以使用 timeit，详见“timeit 模块”。 |

## 装饰-排序-去装饰模式

堆模块中的几个函数虽然执行比较，但不接受 key=参数以自定义比较。这是不可避免的，因为这些函数在原地操作一个简单的项列表：它们没有地方“存放”一次性计算好的自定义比较键。

当你需要堆功能和自定义比较时，你可以应用古老的[*装饰-排序-去装饰（DSU）*惯用语](https://oreil.ly/7iR8O)⁴（在 Python 早期版本引入 key=功能之前，这个惯用语曾是优化排序的关键）。

DSU 惯用语，适用于 heapq，具有以下组件：

装饰

建立一个辅助列表*A*，其中每个项都是一个元组，以排序键开头，并以原始列表*L*的项结尾。

排序

在*A*上调用 heapq 函数，通常以 heapq.heapify(*A*)开始。⁵

去装饰

当你从*A*中提取一个项，通常通过调用 heapq.heappop(*A*)，只返回结果元组的最后一项（这是原始列表*L*的一项）。

当你通过调用 heapq.heappush(*A*, /, *item*)向*A*中添加项时，装饰你要插入的实际项到以排序键开头的元组中。

这些操作序列可以封装在一个类中，如本例所示：

```py
`import` `heapq`

`class` KeyHeap(object):
    `def` __init__(self, alist, /, key):
        self.heap = [(key(o), i, o) `for` i, o `in` enumerate(alist)]
        heapq.heapify(self.heap)
        self.key = key
        `if` alist:
            self.nexti = self.heap[-1][1] + 1
        `else`:
            self.nexti = 0

    `def` __len__(self):
        `return` len(self.heap)

    `def` push(self, o, /):
        heapq.heappush(self.heap, (self.key(o), self.nexti, o))
        self.nexti += 1

    `def` pop(self):
        `return` heapq.heappop(self.heap)[-1]
```

在本例中，我们在装饰的元组中间（排序键后、实际项前）使用一个递增的数字，以确保即使它们的排序键相等，实际项也*永远*不会直接比较（这种语义保证是排序功能中关键参数的重要方面）。

# argparse 模块

当你编写一个 Python 程序，意图从命令行运行（或从类 Unix 系统的 shell 脚本，或从 Windows 的批处理文件），你通常希望让用户传递*命令行参数*（包括*命令行选项*，按约定以一个或两个破折号字符开头的参数）。在 Python 中，你可以访问这些参数作为 sys.argv，这是 sys 模块的一个属性，将这些参数作为字符串列表存放（sys.argv[0]是用户启动程序的名称或路径；参数在子列表 sys.argv[1:]中）。Python 标准库提供了三个模块来处理这些参数；我们只涵盖最新和最强大的一个，argparse，并且只涵盖了一个小而*核心*的 argparse 丰富功能子集。请参阅在线[参考资料](https://oreil.ly/v_ml0)和[教程](https://oreil.ly/QWg01)，了解更多内容。argparse 提供一个类，其签名如下：

| ArgumentParser | ArgumentParser(**kwargs*) ArgumentParser 是执行参数解析的类。它接受许多命名参数，大多用于改善程序在命令行参数包含 -h 或 --help 时显示的帮助消息。你应该始终传递一个命名参数 description=，这是一个总结你的程序目的的字符串。 |
| --- | --- |

给定 ArgumentParser 的实例 *ap*，通过一个或多个调用 *ap*.add_argument 进行准备，然后通过调用 *ap*.parse_args()（不带参数，因此它解析 sys.argv）使用它。该调用返回一个 argparse.Namespace 的实例，其中包含你程序的参数和选项作为属性。

add_argument 有一个强制的第一个参数：用于位置命令行参数的标识符字符串，或用于命令行选项的标志名称。在后一种情况下，传递一个或多个标志名称；选项可以同时具有短名称（破折号，然后一个字符）和长名称（两个破折号，然后一个标识符）。

在位置参数之后，传递零个或多个命名参数以控制其行为。表 8-9 列出了最常用的几个。

表 8-9\. add_argument 的常见命名参数

| action | 解析器对此参数的操作。默认值：'store'，将参数的值存储在命名空间中（使用 dest 指定的名称，在本表后述）。还有几个有用的选项：'store_true' 和 'store_false'，将选项转换为布尔值（如果选项不存在，则默认为相反的布尔值），以及 'append'，将参数值附加到列表中（因此允许选项重复）。 |
| --- | --- |
| choices | 允许参数的一组值（如果值不在其中，则解析参数会引发异常）。默认值：无约束。 |
| default | 如果参数不存在时的值。默认值：**None**。 |
| dest | 用于此参数的属性名称。默认值：与第一个位置参数去掉前导破折号相同，如果有的话。 |
| help | 描述参数用途的字符串，用于帮助消息。 |
| nargs | 逻辑参数使用的命令行参数的数量。默认值：1，存储在命名空间中。可以是大于 0 的整数（使用指定数量的参数，将它们存储为列表），'?'（1 或无，此时使用默认值），'*'（0 或多个，存储为列表），'+'（1 或多个，存储为列表），或 argparse.REMAINDER（所有剩余参数，存储为列表）。 |
| type | 接受字符串的可调用对象，通常是类型（如 int），用于将值从字符串转换为其他类型。可以是 argparse.FileType 的实例，将字符串作为文件名打开（如果 FileType('r')，则读取，如果 FileType('w')，则写入，等等）。 |

这是一个简单的 argparse 示例——将此代码保存在名为 *greet.py* 的文件中：

```py
`import` argparse
ap = argparse.ArgumentParser(description='Just an example')
ap.add_argument('who', nargs='?', default='World')
ap.add_argument('--formal', action='store_true')
ns = ap.parse_args()
`if` ns.formal:
    greet = 'Most felicitous salutations, o {}.'
`else`:
    greet = 'Hello, {}!'
print(greet.format(ns.who))
```

现在，**python** **greet.py** 打印 Hello, World!，而 **python** **greet.py** **--formal Cornelia** 打印 Most felicitous salutations, o Cornelia.

# itertools 模块

itertools 模块提供了高性能的构建块来构建和操作迭代器。为了处理大量的项目，迭代器通常比列表更好，这要归功于迭代器固有的“惰性评估”方法：迭代器逐个生成项目，而列表（或其他序列）的所有项目必须同时在内存中。这种方法甚至使得构建和使用无界迭代器成为可能，而列表必须始终具有有限数量的项目（因为任何计算机都具有有限的内存量）。

表 8-10 涵盖了 itertools 中最常用的属性；每个属性都是一个迭代器类型，您可以调用它们来获取相应类型的实例，或者行为类似的工厂函数。有关更多 itertools 属性，请参阅 [在线文档](https://oreil.ly/d5Eew)，其中包括 *组合* 生成器的排列、组合和笛卡尔积，以及有用的 itertools 属性分类。

在线文档还提供了有关组合和使用 itertools 属性的配方。这些配方假设您在模块顶部有 **from** itertools **import** *；这 *不* 是推荐的用法，只是假设以使配方的代码更紧凑。最好 **import** itertools **as** it，然后使用引用如 it.*something* 而不是更冗长的 itertools.*something*。⁶

表 8-10\. itertools 模块的函数和属性

| accumulate | accumulate(*seq*, *func*, /[, initial*=init*]) 类似于 functools.reduce(*func*, *seq*)，但返回所有中间计算值的迭代器，而不仅仅是最终值。 **3.8+** 您还可以传递一个初始值 *init*，其工作方式与 functools.reduce 中的相同（请参阅 表 8-7）。 |
| --- | --- |

| chain | chain(**iterables*) 生成器，从第一个参数开始生成项目，然后从第二个参数开始生成项目，依此类推，直到最后一个参数结束。这就像生成器表达式一样：

```py
(*`it`* `for` *`iterable`* `in` *`iterables`* `for` *`it`* `in` *`iterable`*)
```

|

| chain.from_ iterable | chain.from_iterable(*iterables*, /) 生成器，按顺序从参数中的可迭代对象中生成项目，就像生成器表达式一样：

```py
(*`it`* `for` *`iterable`* `in` *`iterables`* `for` *`it`* `in` *`iterable`*)
```

|

| compress | compress(*data*, *conditions*, /) 生成器，生成 *conditions* 中的 true 项对应的 *data* 中的每个项目，就像生成器表达式一样：

```py
(*`it`* `for` *`it`*, *`cond`* `in` zip(*`data`*, *`conditions`*) `if` *`cond`*)
```

|

| count | count(start=0, step=1) 生成连续整数，从 *start* 开始，就像生成器一样：

```py
`def` count(start=0, step=1):
    `while` `True`:
        `yield` start
        start += step
```

count 返回一个无尽的迭代器，因此请小心使用，始终确保您显式终止对它的任何循环。 |

| cycle | cycle(*iterable*, /) 生成器，无限重复 *iterable* 中的每个项目，每次到达末尾时从开头重新开始，就像生成器一样：

```py
`def` cycle(iterable):
    saved = []
    `for` item `in` iterable:
        `yield` item
        saved.append(item)
    `while` saved:
        `for` item `in` saved:
            `yield` item
```

cycle 返回一个无尽的迭代器，因此请小心使用，始终确保您显式终止对它的任何循环。 |

| dropwhile | dropwhile(*func*, *iterable*, /) 丢弃*iterable*中*func*为真的前面的 0+项，然后产生每个剩余的项，就像生成器一样：

```py
`def` dropwhile(func, iterable):
    iterator = iter(iterable)
    `for` item `in` iterator:
        `if` `not` func(item):
            `yield` item
 `break`
    `for` item `in` iterator:
        `yield` item
```

|

| filterfalse | filterfalse(*func*, *iterable*, /) 产生*iterable*中*func*为假的项，就像生成器表达式一样：

```py
(*`it`* `for` *`it`* `in` *`iterable`* `if` `not` *`func`*(*`it`*))
```

当*func*可以是接受单个参数的任何可调用对象，或**None**时。当*func*为**None**时，filterfalse 产生假的项，就像生成器表达式一样：

```py
(*`it`* `for` *`it`* `in` *`iterable`* `if` `not` *`it`*)
```

|

| groupby | groupby(*iterable*, /, key=**None**) *iterable*通常需要根据 key（通常为**None**，表示恒等函数，**lambda** x: x）已排序。groupby 产生对（*k*，*g*）的对，每个对表示*iterable*中具有相同值*key*(*item*)的相邻项组的*group*；每个*g*是一个迭代器，产生组中的项。当 groupby 对象前进时，先前的迭代器*g*变为无效（因此，如果需要稍后处理一组项，则最好在某处存储一个“快照”列表“它”，list(*g*)）。

另一种查看 groupby 生成的组的方法是，每个组在*key*(*item*)更改时终止（这就是为什么通常只在已经按*key*排序的*iterable*上调用 groupby）。

例如，假设我们有一组小写单词，我们想要一个字典，将每个首字母映射到具有该首字母的最长单词（在“并列情况”下任意地打破）。我们可以写成：

```py
`import` itertools `as` it
`import` operator
`def` set2dict(aset):
    first = operator.itemgetter(0)
    words = sorted(aset, key=first)
    adict = {}
    `for` init, group `in` it.groupby(words, key=first):
        adict[init] = max(group, key=len)
    `return` adict
```

|

| islice | islice(*iterable*[, *start*], *stop*[, *step*], /) 返回*iterable*的项目（默认情况下跳过第一个*start*个项目，通常为 0），直到但不包括*stop*，以*step*（默认为 1）递增。所有参数必须是非负整数（或**None**），*step*必须大于 0。除了检查和可选参数外，它类似于生成器：

```py
`def` islice(iterable, start, stop, step=1):
    en = enumerate(iterable)
    n = stop
    `for` n, item `in` en:
        `if` n>=start:
            `break`
    `while` n<stop:
        `yield` item
        `for` x `in` range(step):
            n, item = next(en)
```

|

| pairwise | pairwise(*seq*, /) **3.10+** 对*seq*中的项成对出现，允许重叠（例如，pairwise('ABCD')将生成'AB'，'BC'和'CD'）。等同于 zip(*seq*, *seq*[1:])返回的迭代器。 |
| --- | --- |

| repeat | repeat(*item*, /[*,* times]) 重复地产生*item*，就像生成器表达式一样：

```py
(item `for` _ `in` range(times))
```

当*times*不存在时，迭代器是无界的，产生可能无限的项，每个项都是对象*item*，就像生成器一样：

```py
`def` repeat_unbounded(item):
    `while` `True`:
        `yield` item
```

|

| starmap | starmap(*func*, *iterable*, /) 对*iterable*中的每个*item*（通常是元组的可迭代对象）产生 func(**item*)，就像生成器表达式一样：

```py
`def` starmap(func, iterable):
    `for` item `in` iterable:
        `yield` func(*item)
```

|

| takewhile | takewhile(*func*, *iterable*, /) 只要*func*(*item*)为真，从*iterable*中产生项，然后完成，就像生成器表达式一样：

```py
`def` takewhile(func, iterable):
    `for` item `in` iterable:
        `if` func(item):
            `yield` item
        `else`:
 `break`
```

|

| tee | tee(*iterable*, *n*=2, /) 返回*n*个独立迭代器的元组，每个迭代器产生与*iterable*的项目相同的项。返回的迭代器彼此独立，但它们与*iterable*不独立；在仍然使用返回的任何迭代器时，请避免以任何方式更改对象*iterable*。 |
| --- | --- |
| zip_longest | zip_longest(**可迭代对象*, /, fillvalue=**None**) 从每个*可迭代对象*中产生一个元组；当最长的*可迭代对象*耗尽时停止，行为就像其他每个对象都使用 fillvalue“填充”到相同长度一样。如果**None**是*可迭代对象*中一个有效的值（以至于它可能会与用于填充的**None**值混淆），你可以使用 Python 省略号（...）或一个哨兵对象 FILL=object()作为 fillvalue。 |

我们已经展示了许多 itertools 属性的等效生成器和生成表达式，但重要的是要考虑 itertools 的速度优势。作为一个微不足道的例子，考虑重复某些操作 10 次：

```py
`for` _ `in` itertools.repeat(None, 10): `pass`
```

结果是这种方式比直接的替代方式快大约 10 到 20%，这取决于 Python 版本和平台：

```py
`for` _ `in` range(10): `pass`
```

¹ 即根据[里氏替换原则](https://oreil.ly/3jMaN)，这是面向对象编程的核心概念之一。

² 首次引入时，defaultdict(int)常用于维护项目计数。由于 Counter 现在是 collections 模块的一部分，对于特定的项目计数任务，使用 Counter 而不是 defaultdict(int)。

³ 对于后进先出（LIFO）队列，又称为“堆栈”，列表及其附加和弹出方法完全足够。

⁴ 也称为[*施瓦茨变换*](https://oreil.ly/FHlZB)。

⁵ 这一步不完全是“排序”，但看起来足够接近，至少如果你眯起眼睛看的话。

⁶ 一些专家建议**from** itertools **import** *，但本书的作者持不同意见。
