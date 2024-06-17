# 第九章。字符串和其他内容

Python 的 str 类型实现了 Unicode 文本字符串，支持运算符、内置函数、方法和专用模块。有些相似的 bytes 类型表示任意二进制数据作为一系列字节，也称为*bytestring*或*byte string*。可以对这两种类型的对象进行许多文本操作：由于这些类型是不可变的，方法大多数情况下会创建并返回一个新的字符串，除非返回原始字符串未更改。可变字节序列可以表示为 bytearray，在“bytearray objects”中简要介绍。

本章首先介绍了这三种类型可用的方法，然后讨论了字符串模块和字符串格式化（包括格式化字符串字面值），接着是 textwrap、pprint 和 reprlib 模块。专门涉及 Unicode 的问题在本章末尾讨论。

# 字符串对象的方法

str、bytes 和 bytearray 对象都是序列，如“Strings”中所述；其中只有 bytearray 对象是可变的。所有不可变序列操作（重复、连接、索引和切片）都适用于这三种类型的实例，并返回相同类型的新对象。除非在 Table 9-1 中另有规定，否则方法适用于这三种类型的对象。str、bytes 和 bytearray 对象的大多数方法返回相同类型的值，或者专门用于在表示之间转换。

术语如“letters”、“whitespace”等，指的是字符串模块的相应属性，在接下来的章节中会详细介绍。虽然 bytearray 对象是可变的，但是返回 bytearray 结果的方法不会改变对象，而是返回一个新的 bytearray，即使结果与原始字符串相同。

为了简洁起见，在下表中，术语 bytes 指代 bytes 和 bytearray 对象。但是在混合使用这两种类型时要小心：虽然它们通常是可互操作的，但结果的类型通常取决于操作数的顺序。

在 Table 9-1 中，由于 Python 中的整数值可以任意大，为了简洁起见，我们使用 sys.maxsize 表示整数默认值，实际上意味着“无限大的整数”。

表 9-1。重要的 str 和 bytes 方法

| capitalize | *s*.capitalize() 返回*s*的副本，其中第一个字符（如果是字母）大写，其余字符（如果有）小写。 |
| --- | --- |
| casefold | *s*.casefold() **str** **only**. 返回按照[Unicode 标准第 3.13 节](https://oreil.ly/PjWUT)描述的算法处理过的字符串。这类似于后面在本表中描述的*s*.lower，但还考虑到例如德语中的 'ß' 和 'ss' 之间的等价性，因此在处理可以包含不止基本 ASCII 字符的文本时更为适用。 |   |
| center | *s*.center(*n*, *fillchar*=' ', /) 返回长度为 max(len(*s*), *n*)的字符串，其中*s*的副本位于中心部分，两侧分别用相同数量的字符*fillchar*填充。默认的*fillchar*是空格字符。例如，'ciao'.center(2)是'ciao'，'x'.center(4, '_')是'_x__'。 |   |
| count | *s*.count(*sub*, *start*=0, *end*=sys.maxsize, /) 返回在*s*[*start*:*end*]中子字符串*sub*的非重叠出现次数。 |   |
| decode | *s*.decode(encoding='utf-8', errors='strict') **bytes** **only**. 根据给定的编码从字节*s*解码为 str 对象。errors 参数指定如何处理解码错误：'strict'会导致错误引发 UnicodeError 异常；'ignore'会忽略格式错误的值；'replace'会用问号替换它们（详见“Unicode”）。其他值可以通过 codecs.register_error 注册，见表 9-10。 |   |
| encode | *s*.encode(encoding='utf-8', errors='strict') **str** **only**. 返回从 str *s*按照给定编码和错误处理获得的 bytes 对象。详见“Unicode”了解更多详情。 |   |
| endswith | *s*.endswith(*suffix*, *start*=0, *end*=sys.maxsize, /) 当*s*[*start*:*end*]以字符串*suffix*结尾时返回**True**；否则返回**False**。*suffix*可以是字符串元组，此时当*s*[*start*:*end*]以元组中任一字符串结尾时返回**True**。 |   |
| expandtabs | *s*.expandtabs(tabsize=8) 返回一个将每个制表符字符更改为一个或多个空格字符的副本，其中每*tabsize*个字符设置一个制表位。 |   |
| find | *s*.find(*sub*, *start*=0, *end*=sys.maxsize, /) 返回在*s*[*start*:*end*]中找到子字符串*sub*的最低索引，其中*sub*完全包含在内。例如，'banana'.find('na')返回 2，'banana'.find('na', 1)也返回 2，而'banana'.find('na', 3)返回 4，'banana'.find('na', -2)也返回 4。如果未找到*sub*，则返回-1。 |   |
| format | *s*.format(**args*, ***kwargs*) **str** **only**. 根据字符串*s*中包含的格式说明，格式化位置和命名参数。详见“字符串格式化”了解更多详情。 |   |
| format_map | *s*.format_map(mapping) **仅限 str**。根据字符串 *s* 中包含的格式化指令格式化映射参数。等同于 *s*.format(**mapping)，但直接使用映射。详情请参见“字符串格式化”。 |   |
| index | *s*.index(*sub*, *start*=0, *end*=sys.maxsize, /) 类似于 find，但是当找不到 *sub* 时会引发 ValueError。 |   |
| isalnum | *s*.isalnum() 当 *s* 的长度大于 0 且 *s* 中的所有字符都是 Unicode 字母或数字时返回**True**。当 *s* 为空或者 *s* 中至少有一个字符既不是字母也不是数字时返回**False**。 |   |
| isalpha | *s*.isalpha() 当 *s* 的长度大于 0 且 *s* 中的所有字符都是字母时返回**True**。当 *s* 为空或者 *s* 中至少有一个字符不是字母时返回**False**。 |   |
| isascii | *s*.isascii() 当字符串为空或字符串中的所有字符都是 ASCII 时返回**True**，否则返回**False**。ASCII 字符的码点范围在 U+0000–U+007F。 |   |
| isdecimal | *s*.isdecimal() **仅限 str**。当 *s* 的长度大于 0 且 *s* 中的所有字符都可用于形成十进制数时返回**True**。这包括被定义为阿拉伯数字的 Unicode 字符。^(a) |   |
| isdigit | *s*.isdigit() 当 *s* 的长度大于 0 且 *s* 中的所有字符都是 Unicode 数字时返回**True**。当 *s* 为空或者 *s* 中至少有一个字符不是 Unicode 数字时返回**False**。 |   |
| isidentifier | *s*.isidentifier() **仅限 str**。根据 Python 语言的定义，当 *s* 是有效的标识符时返回**True**；关键字也满足该定义，因此，例如 'class'.isidentifier() 返回**True**。 |   |
| islower | *s*.islower() 当 *s* 中的所有字母都是小写字母时返回**True**。当 *s* 不包含字母或者 *s* 中至少有一个大写字母时返回**False**。 |   |
| isnumeric | *s*.isnumeric() **仅限 str**。类似于 *s*.isdigit()，但使用了更广泛的数字符号定义，包括 Unicode 标准中定义的所有数字符号（如分数）。 |   |
| isprintable | *s*.isprintable() **仅限 str**。当 *s* 中的所有字符都是空格 ('\x20') 或在 Unicode 标准中定义为可打印字符时返回**True**。因为空字符串不包含不可打印字符，''.isprintable() 返回**True**。 |   |
| isspace | *s*.isspace() 当 *s* 的长度大于 0 且 *s* 中的所有字符都是空白字符时返回**True**。当 *s* 为空或者 *s* 中至少有一个字符不是空白字符时返回**False**。 |   |
| istitle | *s*.istitle() 返回 **True** 当字符串 *s* 是 *titlecased*：即每个连续字母序列的开头都是大写字母，其他字母都是小写字母时（例如，'King Lear'.istitle() 返回 **True**）。 当 *s* 不包含字母，或者 *s* 的至少一个字母不符合标题大小写条件时，istitle 返回 **False**（例如，'1900'.istitle() 和 'Troilus and Cressida'.istitle() 返回 **False**）。 |   |
| isupper | *s*.isupper() 返回 **True** 当 *s* 中所有字母都是大写时。 当 *s* 不包含字母，或者 *s* 的至少一个字母是小写时，isupper 返回 **False**。 |   |
| join | *s*.join(*seq*, /) 返回由 *seq* 中的项目连接而成的字符串，每个项目之间由 *s* 的副本分隔（例如，''.join(str(x) **for** x **in** range(7)) 返回 '0123456'，'x'.join('aeiou') 返回 'axexixoxu'）。 |   |
| ljust | *s*.ljust(*n*, *fillchar*=' ', /) 返回长度为 max(len(*s*),*n*) 的字符串，其中以 *fillchar* 字符填充的 *s* 的副本在开头，后跟零个或多个尾随 *fillchar* 的副本。 |   |
| lower | *s*.lower() 返回 *s* 的副本，其中所有字母（如果有）都转换为小写。 |   |
| lstrip | *s*.lstrip(*x*=string.whitespace, /) 返回删除字符串 *x* 中任何前导字符后的 *s* 的副本。 例如，'banana'.lstrip('ab') 返回 'nana'。 |   |
| removeprefix | *s*.removeprefix(*prefix*, /) 3.9+ 当 *s* 以 *prefix* 开头时，返回 *s* 的剩余部分；否则返回 *s*。 |   |
| removesuffix | *s*.removesuffix(*suffix*, /) 3.9+ 当 *s* 以 *suffix* 结尾时，返回 *s* 的剩余部分；否则返回 *s*。 |   |
| replace | *s*.replace(*old*, *new*, *count*=sys.maxsize, /) 返回一个副本，其中第一个 *count* （或更少，如果更少）非重叠出现的子字符串 *old* 被字符串 *new* 替换（例如，'banana'.replace('a', 'e', 2) 返回 'benena'）。 |   |
| rfind | *s*.rfind(*sub*, *start*=0, *end*=sys.maxsize, /) 返回 *s* 中子字符串 *sub* 的最高索引，使得 *sub* 完全包含在 *s*[*start*:*end*] 中。 如果未找到 *sub*，则 rfind 返回 -1。 |   |
| rindex | *s*.rindex(*sub*, *start*=0, *end*=sys.maxsize, /) 类似于 rfind，但如果未找到 *sub*，则引发 ValueError。 |   |
| rjust | *s*.rjust(*n*, *fillchar*=' ', /) 返回长度为 max(len(*s*),*n*) 的字符串，其中以 *fillchar* 字符填充的 *s* 的副本在末尾，前面跟零个或多个前导 *fillchar* 的副本。 |   |
| rstrip | *s*.rstrip(*x*=string.whitespace, /) 返回 *s* 的副本，删除在字符串 *x* 中找到的尾部字符。 例如，'banana'.rstrip('ab') 返回 'banan'。 |   |

| split | *s*.split(sep=None, maxsplit=sys.maxsize) 返回一个最多包含 maxsplit+1 个字符串的列表 *L*。*L* 的每个项是 *s* 的一个“单词”，其中字符串 sep 分隔单词。当 *s* 的单词数大于 maxsplit 时，*L* 的最后一个项是 *s* 中跟随第一个 maxsplit 个单词之后的子字符串。当 sep 为 **None** 时，任何空白字符串分隔单词（例如，'four score and seven years'.split(**None**, 3) 返回 ['four', 'score', 'and', 'seven years']）。

注意在使用 **None**（任何连续的空白字符作为分隔符）和 ' '（每个单独空格字符作为分隔符，不包括制表符和换行符，也不包括空格字符串）之间的区别。例如：

```py
>>> x = 'a  bB'  *`# two spaces between a and bB`*
>>> x.split()    *`# or x.split(`**`None`**`)`*
```

```py
['a', 'bB']
```

```py
>>> x.split(' ')
```

```py
['a', '', 'bB']
```

在第一种情况下，中间的两个空格被视为单一分隔符；在第二种情况下，每个单独的空格被视为分隔符，因此在两个空格之间有一个空字符串。 |   |

| splitlines | *s*.splitlines(keepends=**False**) 类似于 *s*.split('\n')。但是，当 keepends 为 **True** 时，结果列表中每个项的末尾 '\n' 也包括在内（如果 *s* 不以 '\n' 结尾，则最后一个项除外）。 |   |
| --- | --- | --- |
| startswith | *s*.startswith(*prefix*, *start*=0, *end*=sys.maxsize, /) 当 *s*[*start*:*end*] 以字符串 *prefix* 开头时返回 **True**；否则返回 **False**。*prefix* 可以是字符串元组，此时如果 *s*[*start*:*end*] 以其中任何一个字符串开头，则返回 **True**。 |   |
| strip | *s*.strip(*x*=string.whitespace, /) 返回 *s* 的副本，删除开头和结尾处位于字符串 *x* 中的字符。例如，'banana'.strip('ab') 返回 'nan'。 |   |
| swapcase | *s*.swapcase() 返回 *s* 的副本，所有大写字母转换为小写字母，所有小写字母转换为大写字母。 |   |
| title | *s*.title() 返回 *s* 的副本，转换为标题格式：每个连续字母序列的开头字母大写，其余字母（如果有）小写。 |   |

| translate | *s*.translate(*table*, /, delete*=*b'') 返回 *s* 的副本，其中 *table* 中的字符被翻译或删除。当 *s* 是字符串时，不能传递 delete 参数；*table* 是一个字典，其键是 Unicode 码点，值可以是 Unicode 码点、Unicode 字符串或 **None**（表示删除对应的字符）。例如：

```py
tbl = {ord('a'):`None`, ord('n'):'ze'}
print('banana'.translate(tbl))  *`# prints:`* *`'bzeze'`*
```

当 *s* 是字节时，*table* 是长度为 256 的字节对象；*s*.translate(*t*, *b*) 的结果是一个字节对象，其中的每个项 *b* 如果是 delete 的项之一，则被省略，否则改为 *t*[ord(*b*)]。

bytes 和 str 各自有一个名为 maketrans 的类方法，可用于构建适合于相应 translate 方法的表。 |   |

| upper | *s*.upper() 返回 *s* 的副本，所有字母（如果有）都转换为大写。 |   |
| --- | --- | --- |
| ^(a) 这不包括用作基数的标点符号，例如句点（.）或逗号（,）。 |

# 字符串模块

字符串模块提供了几个有用的字符串属性，列在 Table 9-2 中。

Table 9-2\. 字符串模块中的预定义常量

| ascii_letters | 包含 ascii_lowercase 和 ascii_uppercase 这两个常量的字符串（将下列两个常量连接在一起）。 |
| --- | --- |
| ascii_lowercase | 字符串 'abcdefghijklmnopqrstuvwxyz' |
| ascii_uppercase | 字符串 'ABCDEFGHIJKLMNOPQRSTUVWXYZ' |
| digits | 字符串 '0123456789' |
| hexdigits | 字符串 '0123456789abcdefABCDEF' |
| octdigits | 字符串 '01234567' |
| punctuation | 字符串 '!"#$%&\'()*+,-./:;<=>?@[\]^_'{&#124;}~'（即在 C 区域中被视为标点字符的所有 ASCII 字符；不依赖于活动的区域设置） |
| printable | 包含所有被认为是可打印字符的 ASCII 字符的字符串（即数字、字母、标点符号和空白字符）。 |
| whitespace | 包含所有被认为是空白字符的 ASCII 字符的字符串：至少包括空格、制表符、换行符和回车符，但可能会根据当前的区域设置包含更多字符（例如某些控制字符）。 |

不应重新绑定这些属性；这样做的效果是未定义的，因为 Python 库的其他部分可能依赖于它们。

字符串模块还提供了 Formatter 类，在下一节中介绍。

# 字符串格式化

Python 提供了一种灵活的机制来格式化字符串（但不适用于字节串：关于字节串，请参见 “使用 % 进行旧式字符串格式化”）。*格式字符串* 简单地是一个包含用大括号（{}）括起来的 *替换字段* 的字符串，由 *值部分*、可选的 *转换部分* 和可选的 *格式说明符* 组成：

```py
{*`value``-``part`*[!*`conversion``-``part`*][:*`format``-``specifier`*]}
```

值部分根据字符串类型而异：

+   对于格式化字符串字面量或 *f-strings*，值部分会作为 Python 表达式进行求值（有关详细信息，请参见后续章节）；表达式不能以感叹号结尾。

+   对于其他字符串，值部分选择一个参数或参数的一个元素用于格式方法。

可选的转换部分是感叹号（!）后跟一个字符 s、r 或 a（在 “值转换” 中描述）。

可选的格式说明符以冒号（:）开头，并确定如何在格式字符串中替换字段的原始替换值。

## 格式化字符串字面量（F-Strings）

此特性允许您插入需要内插的值，用大括号括起来。要创建格式化字符串字面量，请在字符串的开头引号之前加上 f（这就是它们被称为 *f-strings* 的原因），例如 f'{value}'：

```py
>>> name = 'Dawn'
>>> print(f'{name!r} is {len(name)} characters long')
```

```py
'Dawn' is 4 characters long
```

您可以使用嵌套的大括号来指定格式表达式的组件：

```py
>>> `for` width `in` 8, 11:
...     `for` precision `in` 2, 3, 4, 5:
...         print(f'{2.7182818284:{width}.{precision}}')
...
```

```py
 2.7
 2.72
 2.718
 2.7183
 2.7
 2.72
 2.718
 2.7183
```

我们已经尝试更新书中的大多数示例以使用 f-strings，因为它们是 Python 中格式化字符串的最紧凑方式。但请记住，这些字符串文字并不是常量——它们在每次包含它们的语句执行时都会计算，这会涉及运行时开销。

在格式化字符串文字中的要格式化的值已经被引号包围：因此，在使用包含字符串引号的值部分表达式时要注意避免语法错误。使用四种不同的字符串引号以及能够使用转义序列，大多数情况都是可能的，尽管可读性可能会受到影响。

# F-Strings 对国际化没有帮助

给定一个需要适应多种语言内容的格式，最好使用格式化方法，因为要插值的值可以在提交格式化之前独立计算。

### 使用 f-strings 进行调试打印

3.8+ 为了便于调试，在格式化字符串文字中的值表达式的最后一个非空字符后面可以跟一个等号（=），可选地包含空格。在这种情况下，表达式本身的文本和等号，包括任何前导和尾随空格，在值之前输出。在等号存在的情况下，当没有指定格式时，Python 使用值的 repr() 作为输出；否则，Python 使用值的 str()，除非指定了 !r 值转换：

```py
>>> a = '*-'
>>> s = 12
>>> f'{a*s=}'
```

```py
"a*s='*-*-*-*-*-*-*-*-*-*-*-*-'"
```

```py
>>> f'{a*s = :30}'
```

```py
'a*s = *-*-*-*-*-*-*-*-*-*-*-*-      '
```

请注意，此形式仅在格式化字符串文字中可用。

这里是一个简单的 f-string 示例。请注意，包括任何周围的文本，包括任何空白字符，在结果中都会字面复制：

```py
>>> n = 10
>>> s = ('zero', 'one', 'two', 'three')
>>> i = 2
>>> f'start {"-"*n} : {s[i]} end'
```

```py
'start ---------- : two end'
```

## 使用格式化调用格式化

在格式化字符串文字中可用的相同格式化操作也可以通过调用字符串的 format 方法执行。在这些情况下，替换字段以选择该调用的参数的值部分开始。您可以指定位置参数和命名参数。以下是一个简单的 format 方法调用示例：

```py
>>> name = 'Dawn'
>>> print('{name} is {n} characters long'
... .format(name=name, n=len(name)))
```

```py
'Dawn' is 4 characters long
```

```py
>>> "This is a {1}, {0}, type of {type}".format("green", "large", 
...                                             type="vase")
```

```py
'This is a large, green, type of vase'
```

简单起见，此示例中的替换字段均不包含转换部分或格式说明符。

如前所述，使用格式化方法时的参数选择机制可以处理位置参数和命名参数。最简单的替换字段是空括号对（{}），表示自动位置参数指定器。每个这样的替换字段自动引用下一个要格式化的位置参数的值：

```py
>>> 'First: {} second: {}'.format(1, 'two')
```

```py
'First: 1 second: two'
```

要重复选择参数或者按顺序使用参数，请使用编号的替换字段来指定参数在参数列表中的位置（从零开始计数）：

```py
>>> 'Second: {1}, first: {0}'.format(42, 'two')
```

```py
'Second: two, first: 42'
```

您不能混合自动和编号的替换字段：这是二选一的选择。

对于命名参数，请使用参数名称。如果需要，可以将它们与（自动或编号）位置参数混合使用：

```py
>>> 'a: {a}, 1st: {}, 2nd: {}, a again: {a}'.format(1, 'two', a=3)
```

```py
'a: 3, 1st: 1, 2nd: two, a again: 3'
```

```py
>>> 'a: {a} first:{0} second: {1} first: {0}'.format(1, 'two', a=3)
```

```py
'a: 3 first:1 second: two first: 1'
```

如果参数是一个序列，则可以使用数值索引来选择参数的特定元素作为要格式化的值。这适用于位置（自动或编号）和命名参数：

```py
>>> 'p0[1]: {[1]} p1[0]: {[0]}'.format(('zero', 'one'),
...                                    ('two', 'three'))
```

```py
'p0[1]: one p1[0]: two'
```

```py
>>> 'p1[0]: {1[0]} p0[1]: {0[1]}'.format(('zero', 'one'),
...                                      ('two', 'three'))
```

```py
'p1[0]: two p0[1]: one'
```

```py
>>> '{} {} {a[2]}'.format(1, 2, a=(5, 4, 3))
```

```py
'1 2 3'
```

如果参数是一个复合对象，则可以通过将属性访问点符号应用于参数选择器来选择其各个属性作为要格式化的值。以下是使用复数的示例，复数具有分别保存实部和虚部的 real 和 imag 属性：

```py
>>> 'First r: {.real} Second i: {a.imag}'.format(1+2j, a=3+4j)
```

```py
'First r: 1.0 Second i: 4.0'
```

索引和属性选择操作可以根据需要多次使用。

## 值转换

您可以通过在选择器后跟!s 应用对象的 __str__ 方法，!r 应用其 __repr__ 方法，或者使用 ascii 内置!a 来为值应用默认转换：

```py
>>> "String: {0!s} Repr: {0!r} ASCII: {0!a}".format("banana 😀")
```

```py
"String: banana 😀 Repr: 'banana 😀' ASCII: 'banana\\U0001f600'"
```

当存在转换时，将在格式化之前应用该转换。由于同一值需要多次使用，在此示例中，格式调用比格式化字符串文字更为合理，后者需要重复三次相同的值。

## 值格式化：格式说明符

替换字段的最后（可选）部分称为*格式说明符*，由冒号（:）引入，提供（可能转换后的）值的进一步所需格式化。如果在替换字段中没有冒号，则意味着使用转换后的值（如果尚未以字符串形式表示）而不进行进一步格式化。如果存在格式说明符，则应按以下语法提供：

```py
[[*`fill`*]*`align`*][*`sign`*][z][#][0][*`width`*][*`grouping_option`*][.*`precision`*][*`type`*]
```

详细信息在以下各小节中提供。

### 填充和对齐

默认填充字符为空格。要使用替代填充字符（不能是开括号或闭括号），请以填充字符开头。填充字符（如果有）应跟随*对齐指示器*（见表 9-3）。

表 9-3\. 对齐指示器

| 字符 | 作为对齐指示器的重要性 |
| --- | --- |
| '<' | 将值左对齐在字段内 |
| '>' | 将值右对齐在字段内 |
| '^' | 将值居中对齐在字段内 |
| '=' | 仅适用于数值类型：在符号和数值的第一个数字之间添加填充字符 |

如果第一个和第二个字符都是*有效*的对齐指示器，则第一个用作填充字符，第二个用于设置对齐方式。

当未指定对齐时，除数字外的值均左对齐。除非稍后在格式说明符中指定字段宽度（见“字段宽度”），否则不添加填充字符，无论填充和对齐如何：

```py
>>> s = 'a string'
>>> f'{s:>12s}'
```

```py
'    a string'
```

```py
>>> f'{s:>>12s}'
```

```py
'>>>>a string'
```

```py
>>> f'{s:><12s}'
```

```py
'a string>>>>'
```

### 符号指示

仅适用于数值的情况下，您可以通过包含符号指示器来区分正负数（详见表 9-4）。

表 9-4\. 符号指示器

| 字符 | 作为符号指示器的重要性 |
| --- | --- |
| '+' | 对于正数，插入+作为符号；对于负数，插入-作为符号 |
| '-' | 对于负数，插入-作为符号；对于正数，不插入任何符号（如果未包含符号指示符，则为默认行为） |
| ' ' | 对于正数，插入一个空格作为符号；对于负数，插入-作为符号 |

空格是默认的符号指示符。如果指定了填充，则会出现在符号和数值之间；在=之后放置符号指示符以避免其被用作填充字符：

```py
>>> n = -1234
>>> f'{n:12}'    *`# 12 spaces before the number`*
```

```py
'       -1234'
```

```py
>>> f'{-n:+12}'  *`# - to flip n's sign, + as sign indicator`* 
```

```py
'       +1234'
```

```py
>>> f'{n:+=12}'  *`# + as fill character between sign and number`*
```

```py
'-+++++++1234'
```

```py
*`# + as sign indicator, spaces fill between sign and number`*
>>> f'{n:=+12}'
```

```py
'-       1234'
```

```py
*`# * as fill between sign and number, + as sign indicator`*
>>> f'{n:*=+12}'
```

```py
'-*******1234'
```

### 零归一化（z）

3.11+某些数字格式能够表示负零，这往往是一个令人惊讶且不受欢迎的结果。当在格式说明符中的这个位置出现 z 字符时，这样的负零将被规范化为正零：

```py
>>> x = -0.001
>>> f'{x:.1f}'
```

```py
'-0.0'
```

```py
>>> f'{x:z.1f}'
```

```py
'0.0'
```

```py
>>> f'{x:+z.1f}'
```

```py
'+0.0'
```

### 基数指示符（#）

仅适用于数值*整数*格式，您可以包含基数指示符，即#字符。如果存在，则表示应在二进制格式化的数字的数字之前加上'0b'，在八进制格式化的数字的数字之前加上'0o'，在十六进制格式化的数字的数字之前加上'0x'。例如，'{23:x}'为'17'，而'{23:#x}'为'0x17'，清楚地标识出值为十六进制。

### 前导零指示符（0）

仅*数值类型*，当字段宽度以零开头时，数值将使用前导零而不是前导空格进行填充：

```py
>>> f"{-3.1314:12.2f}"
```

```py
'       -3.13'
```

```py
>>> f"{-3.1314:012.2f}"
```

```py
'-00000003.13'
```

### 字段宽度

你可以指定要打印的字段的宽度。如果指定的宽度小于值的长度，则使用值的长度（但对于字符串值，请参见下一节“精度规范”）。如果未指定对齐方式，则该值为左对齐（但对于数字，则为右对齐）：

```py
>>> s = 'a string'
>>> f'{s:¹²s}'
```

```py
'  a string  '
```

```py
>>> f'{s:.>12s}'
```

```py
'....a string'
```

使用嵌套大括号时，调用格式方法，字段宽度也可以是格式参数：

```py
>>> '{:.>{}s}'.format(s, 20)
```

```py
'............a string'
```

有关此技术的更详细讨论，请参阅“嵌套格式规范”。

### 分组选项

对于十进制（默认）格式类型中的数值，您可以插入逗号（**，**）或下划线（**_**）来请求结果整数部分的每个三位数（*数字组*）之间用该字符分隔。例如：

```py
>>> f'{12345678.9:,}'
```

```py
'12,345,678.9'
```

此行为忽略了系统区域设置；对于区域设置感知的数字分组和小数点字符使用，请参阅表 9-5 中的格式类型 n。

### 精度规范

精度（例如，.2）对于不同的格式类型具有不同的含义（有关详细信息，请参见下一小节），大多数数字格式的默认值为.6。对于 f 和 F 格式类型，它指定应在格式化时四舍五入的小数点后的位数；对于 g 和 G 格式类型，它指定应四舍五入的*有效数字*的数量；对于非数值的值，它指定在格式化之前将值截断为其最左边的字符。例如：

```py
>>> x = 1.12345
>>> f'as f: {x:.4f}'  *`# rounds to 4 digits after decimal point`*
```

```py
'as f: 1.1235'
```

```py
>>> f'as g: {x:.4g}'  *`# rounds to 4 significant digits`*
```

```py
'as g: 1.123'
```

```py
>>> f'as s: {"1234567890":.6s}'  *`# string truncated to 6 characters`*
```

```py
'as s: 123456'
```

### 格式类型

格式规范以可选的*格式类型*结束，该类型确定如何在给定宽度和给定精度下表示该值。如果没有显式指定格式类型，则正在格式化的值确定默认格式类型。

s 格式类型始终用于格式化 Unicode 字符串。

整数数值具有一系列可接受的格式类型，在表 9-5 中列出。

表 9-5\. 整数格式类型

| 格式类型 | 格式说明 |
| --- | --- |
| b | 二进制格式—一系列 1 和 0 |
| c | 其序数值是格式化值的 Unicode 字符 |
| d | 十进制（默认格式类型） |
| n | 十进制格式，使用系统区域设置时的区域特定分隔符（在英国和美国为逗号） |
| o | 八进制格式—一系列八进制数字 |
| x 或 X | 十六进制格式—一系列十六进制数字，相应的字母为小写或大写 |

浮点数具有不同的格式类型集，如表 9-6 所示。

表 9-6\. 浮点格式类型

| 格式类型 | 格式说明 |
| --- | --- |
| e 或 E | 指数格式—科学记数法，整数部分介于一到九之间，指数之前使用 e 或 E |
| f 或 F | 固定点格式，其中无穷大（inf）和非数字（nan）为小写或大写 |
| g 或 G | 通用格式（默认格式类型）—在可能的情况下使用固定点格式，否则使用指数格式；使用小写或大写表示 e、inf 和 nan，具体取决于格式类型的大小写 |
| n | 类似于通用格式，但在设置系统区域设置时使用区域特定分组的分隔符，用于每三位数字和小数点 |
| % | 百分比格式—将值乘以 100，并将其格式化为固定点，后跟% |

当未指定格式类型时，浮点数使用 g 格式，小数点后至少有一位数字，默认精度为 12。

下面的代码接受一个数字列表，并将每个数字右对齐在九个字符宽的字段中；指定每个数字的符号始终显示，并在每组三位数字之间添加逗号，并将每个数字四舍五入到小数点后两位，根据需要将整数转换为浮点数：

```py
>>> `for` num `in` [3.1415, -42, 1024.0]:
...     f'{num:>+9,.2f}' 
...
```

```py
'    +3.14'
'   -42.00'
'+1,024.00'
```

## 嵌套格式规范

在某些情况下，您可能希望使用表达式值来帮助确定所使用的精确格式：您可以使用嵌套格式化来实现这一点。例如，要将字符串格式化为比字符串本身宽四个字符的字段，可以将一个宽度值传递给格式化，如下所示：

```py
>>> s = 'a string'
>>> '{0:>{1}s}'.format(s, len(s)+4)
```

```py
'    a string'
```

```py
>>> '{0:_^{1}s}'.format(s, len(s)+4)
```

```py
'__a string__'
```

通过仔细设置宽度规范和嵌套格式化，您可以将一系列元组打印成对齐的列。例如：

```py
`def` columnar_strings(str_seq, widths):
    `for` cols `in` str_seq:
        row = [f'{c:{w}.{w}s}'
               `for` c, w `in` zip(cols, widths)]
        print(' '.join(row))
```

鉴于此函数，以下代码：

```py
c = [
        'four score and'.split(),
        'seven years ago'.split(),
        'our forefathers brought'.split(),
        'forth on this'.split(),
    ]

columnar_strings(c, (8, 8, 8))
```

输出：

```py
four     score    and
seven    years    ago
our      forefath brought
forth    on       this
```

## 用户编码类的格式化

最终，值通过调用其 __format__ 方法并使用格式说明符作为参数进行格式化。内置类型要么实现自己的方法，要么继承自 object，其不太有用的 format 方法只接受空字符串作为参数：

```py
>>> object().__format__('')
```

```py
'<object object at 0x110045070>'
```

```py
>>> import math
>>> math.pi.__format__('18.6')
```

```py
'           3.14159'
```

您可以利用这些知识实现自己的完全不同的格式化迷你语言，如果您愿意的话。下面的简单示例演示了格式规范的传递和（恒定的）格式化字符串结果的返回。格式规范的解释由您控制，您可以选择实现任何格式化标记法：

```py
>>> class S:
...     def __init__(self, value):
...         self.value = value
...     `def` __format__(self, fstr):
...         `match` fstr:
...             `case` 'U':
...                 `return` self.value.upper()
...             `case` 'L':
...                 `return` self.value.lower()
...             `case` 'T':
...                 `return` self.value.title()
...             `case` _:
...                 `return` ValueError(f'Unrecognized format code'
...                                   f' {fstr!r}')
>>> my_s = S('random string')
>>> f'{my_s:L}, {my_s:U}, {my_s:T}'
```

```py
'random string, RANDOM STRING, Random String'
```

__format__ 方法的返回值被替换为格式化输出中的替换字段，允许对格式字符串进行任何所需的解释。

此技术在 datetime 模块中使用，以允许使用 strftime 风格的格式字符串。因此，以下所有方式都会得到相同的结果：

```py
>>> `import` datetime
>>> d = datetime.datetime.now()
>>> d.__format__('%d/%m/%y')
```

```py
'10/04/22'
```

```py
>>> '{:%d/%m/%y}'.format(d)
```

```py
'10/04/22'
```

```py
>>> f'{d:%d/%m/%y}'
```

```py
'10/04/22'
```

为了更轻松地格式化对象，字符串模块提供了一个 Formatter 类，具有许多有用的方法来处理格式化任务。详细信息请参阅 [在线文档](https://oreil.ly/aUmUs)。

## 使用 % 进行遗留字符串格式化

Python 中的一个遗留字符串格式化表达式的语法是：

```py
*`format`* % *`values`*
```

其中 *format* 是一个包含格式说明符的 str、bytes 或 bytearray 对象，而 *values* 是要格式化的值，通常作为一个元组。¹ 与 Python 的较新格式化功能不同，您也可以使用 % 格式化来处理 bytes 和 bytearray 对象，而不仅仅是 str 对象。

在日志记录中的等效用法，例如：

```py
logging.info(*`format`*, **`values`*)
```

*values* 是在 *format* 后作为位置参数传入的。

遗留字符串格式化方法大致具有与 C 语言的 printf 相同的功能集，并以类似的方式运行。每个格式说明符都是以百分号（%）开头，并以 表 9-7 中显示的转换字符之一结束。

表 9-7\. 字符串格式转换字符

| 字符 | 输出格式 | 注释 |
| --- | --- | --- |
| d, i | 有符号十进制整数 | 值必须是一个数字 |
| u | 无符号十进制整数 | 值必须是一个数字 |
| o | 无符号八进制整数 | 值必须是一个数字 |
| x | 无符号十六进制整数（小写字母） | 值必须是一个数字 |
| X | 无符号十六进制整数（大写字母） | 值必须是一个数字 |
| e | 浮点数以指数形式表示（指数部分小写 e） | 值必须是一个数字 |
| E | 浮点数以指数形式表示（指数部分大写 E） | 值必须是一个数字 |
| f, F | 浮点数以十进制形式表示 | 值必须是一个数字 |
| g, G | 当 *exp* >=4 或 < 精度时，类似于 e 或 E；否则类似于 f 或 F | *exp* 是被转换的数的指数部分 |
| a | 字符串 | 使用 ascii 转换任何值 |
| r | 字符串 | 使用 repr 转换任何值 |
| s | 字符串 | 使用 str 转换任意值 |
| % | 百分号字符 | 不消耗任何值 |

日志模块最常使用的是 a、r 和 s 转换字符。在%和转换字符之间，可以指定一系列可选的修饰符，稍后我们将讨论。

格式表达式记录的内容是*format*，其中每个格式规范都会被*values*的相应项替换，并根据规范转换为字符串。以下是一些简单的示例：

```py
`import` logging
logging.getLogger().setLevel(logging.INFO)
x = 42
y = 3.14
z = 'george'
logging.info('result = %d', x)        *`# logs:`* *`result = 42`*
logging.info('answers: %d %f', x, y)  *`# logs:`* *`answers: 42 3.140000`*
logging.info('hello %s', z)           *`# logs:`* *`hello george`*
```

## 格式规范语法

每个格式规范按位置对应于*values*中的一项。格式规范可以包括修改器，以控制如何将*values*中的对应项转换为字符串。格式规范的组成部分依次为：

+   指示转换规范开始的强制前导百分号字符

+   零个或多个可选的转换标志：

    '#'

    转换使用一个备用形式（如果其类型存在的话）。

    '0'

    转换进行零填充。

    '-'

    转换左对齐。

    ' '

    负数带有符号，正数前有一个空格。

    '+'

    数字前会放置一个符号（+或-）。

+   可选的转换最小宽度：一个或多个数字，或者一个星号(*)，表示宽度从*values*的下一项获取

+   可选的转换精度：点(.)后跟零个或多个数字或*，表示精度从*values*的下一项获取

+   来自表格 9-7 的强制转换类型

*values*必须与*format*的规范数量完全相同（对于由*给出的宽度或精度，额外消耗一个*values*中的项，该项必须是整数，并且用作该转换的宽度或精度的字符数）。

# 始终使用%r（或%a）来记录可能错误的字符串

大多数情况下，*format*字符串中的格式规范都将是%s；偶尔，您可能希望确保输出的水平对齐（例如，在六个字符的右对齐、可能被截断的空间中），在这种情况下，您将使用%6.6s。但是，对于%r 或%a，有一个重要的特殊情况。

当您记录可能错误的字符串值（例如，找不到的文件名）时，请不要使用%s：当错误是字符串具有不必要的前导或尾随空格，或者包含一些非打印字符如\b 时，%s 可能会使您通过研究日志难以发现。相反，请使用%r 或%a，以便所有字符都清晰显示，可能通过转义序列。 （对于 f-strings，相应的语法将是{variable!r}或{variable!a}）。

# 文本包装和填充

textwrap 模块提供一个类和几个函数，以给定的最大长度将字符串格式化为多行。要微调填充和换行效果，可以实例化 textwrap 提供的 TextWrapper 类并应用详细控制。然而，大多数情况下，textwrap 提供的函数足够使用；最常用的函数在表 9-8 中介绍。

表 9-8\. textwrap 模块的常用函数

| dedent | dedent(text) 函数接受一个多行字符串，并返回一个去除了所有行相同数量前导空白的副本，使得某些行没有前导空白。 |
| --- | --- |
| fill | fill(text, width=70) 返回一个等于 '\n'.join(wrap(text, width)) 的多行字符串。 |
| wrap | wrap(text, width=70) 返回一个字符串列表（不带结束换行符），每个字符串的长度不超过 width 个字符。wrap 还支持其他命名参数（相当于 TextWrapper 类实例的属性）；对于这些高级用法，请参阅[在线文档](https://oreil.ly/TjsSm)。 |

# pprint 模块

pprint 模块可以对数据结构进行漂亮打印，其格式化比内置函数 repr 更易读（详见表 8-2）。要微调格式，可以实例化 pprint 提供的 PrettyPrinter 类，并应用详细控制，辅助函数也由 pprint 提供。然而，大多数情况下，pprint 提供的函数足够使用（参见表 9-9）。

表 9-9\. pprint 模块的常用函数

| pformat | pformat(object) 返回表示对象的漂亮打印的字符串。 |
| --- | --- |

| pp, pprint | pp(object, stream=*sys.stdout*), pprint(object, stream=*sys.stdout*) |

将对象的漂亮打印输出到打开写入文件对象流中，并以换行符结尾。

以下语句实际上完成了相同的操作：

```py
print(pprint.pformat(x))
pprint.pprint(x)
```

在许多情况下，这两个构造在效果上基本相同于 print(*x*) —— 例如，对于可以在单行内显示的容器。但是，对于像 *x*=list(range(30)) 这样的情况，print(*x*) 会在 2 行显示 *x*，在任意点断开，而使用 pprint 模块会将 *x* 按每项一行的方式展示出来。在希望使用模块特定的显示效果而不是正常字符串表示的情况下，请使用 pprint。

pprint 和 pp 支持额外的格式化参数；详细信息请参阅[在线文档](https://oreil.ly/xwrN8)。 |

# reprlib 模块

reprlib 模块提供了一个替代内置函数 repr（在表格 8-2 中介绍），用于表示字符串的长度限制。要微调长度限制，您可以实例化或子类化 reprlib 模块提供的 Repr 类，并应用详细控制。然而，大多数情况下，模块公开的唯一函数就足够了：repr(*obj*)，它返回表示*obj*的字符串，具有合理的长度限制。

# Unicode

要将字节串转换为 Unicode 字符串，请使用字节串的 decode 方法（参见表格 9-1）。转换必须始终是显式的，并使用称为*编解码器*（缩写为*编码器-解码器*）的辅助对象执行。编解码器还可以使用字符串的 encode 方法将 Unicode 字符串转换为字节串。要标识编解码器，请将编解码器名称传递给 decode 或 encode。当您不传递编解码器名称时，Python 使用默认编码，通常为'utf-8'。

每次转换都有一个参数错误，一个字符串指定如何处理转换错误。合理地，默认值为'strict'，意味着任何错误都会引发异常。当错误为'replace'时，转换将在字节串结果中用'?'替换导致错误的每个字符，在 Unicode 结果中用 u'\ufffd'替换。当错误为'ignore'时，转换会默默地跳过导致错误的字符。当错误为'xmlcharrefreplace'时，转换会将导致错误的每个字符替换为 XML 字符引用表示形式的结果中。您可以编写自己的函数来实现转换错误处理策略，并通过调用 codecs.register_error 并在接下来的部分的表格中覆盖的适当名称下注册它。

## 编解码器模块

编解码器名称到编解码器对象的映射由 codecs 模块处理。此模块还允许您开发自己的编解码器对象并注册它们，以便可以像内置编解码器一样按名称查找它们。它提供一个函数，让您可以显式查找任何编解码器，获取编码和解码使用的函数，以及用于包装类似文件对象的工厂函数。这些高级功能很少使用，我们在本书中不涵盖它们。

codecs 模块与 Python 标准库的编码包一起提供了对处理国际化问题有用的内置编解码器。Python 自带超过 100 种编解码器；您可以在[在线文档](https://oreil.ly/3iAbC)中找到完整列表，并对每种编解码器进行简要解释。在模块 sitecustomize 中安装编解码器作为全局默认不是良好的做法；相反，推荐的用法是每次在字节和 Unicode 字符串之间转换时始终通过名称指定编解码器。Python 的默认 Unicode 编码是'utf-8'。

codecs 模块为大多数 ISO 8859 编码提供了 Python 实现的编解码器，编解码器名称从'iso8859-1'到'iso8859-15'。在西欧地区流行的编解码器是'latin-1'，它是 ISO 8859-1 编码的快速内置实现，提供了一个每字符一个字节的编码，包括西欧语言中的特殊字符（注意，它缺少欧元货币符号'€'；但如果需要该符号，请使用'iso8859-15'）。仅在 Windows 系统上，名为'mbcs'的编解码器包装了平台的多字节字符集转换过程。codecs 模块还提供了各种代码页，名称从'cp037'到'cp1258'，以及 Unicode 标准编码'utf-8'（可能是最佳选择，因此推荐使用，也是默认的）和'utf-16'（具有特定的大端和小端变体：'utf-16-be'和'utf-16-le'）。对于 UTF-16 的使用，codecs 还提供了属性 BOM_BE 和 BOM_LE，分别用于大端和小端机器的字节顺序标记，以及 BOM，当前平台的字节顺序标记。

除了用于更高级用途的各种函数外，如前所述，codecs 模块还提供了一个函数，允许您注册自己的转换错误处理函数：

| regis⁠t⁠e⁠r⁠_​e⁠r⁠r⁠o⁠r | register_error(*name*, *func*, /) *name*必须是一个字符串。*func*必须是一个可调用的函数，接受一个参数*e*，它是 UnicodeDecodeError 的一个实例，并且必须返回一个包含两个项目的元组：要插入转换后字符串结果的 Unicode 字符串，以及继续转换的索引（通常为*e*.end）。该函数可以使用*e*.encoding，即此转换的编解码器名称，以及*e*.object[*e*.start:*e*.end]，导致转换错误的子字符串。 |
| --- | --- |

## unicodedata 模块

unicodedata 模块提供了对 Unicode 字符数据库的简单访问。对于任何 Unicode 字符，您可以使用 unicodedata 提供的函数获取字符的 Unicode 类别、官方名称（如果有的话）和其他相关信息。您还可以查找与给定官方名称相对应的 Unicode 字符（如果有的话）：

```py
>>> `import` unicodedata
>>> unicodedata.name('⚀')
```

```py
'DIE FACE-1'
```

```py
>>> unicodedata.name('Ⅵ')
```

```py
'ROMAN NUMERAL SIX'
```

```py
>>> int('Ⅵ')
```

```py
ValueError: invalid literal for int() with base 10: 'Ⅵ'
```

```py
>>> unicodedata.numeric('Ⅵ')  *`# use unicodedata to get numeric value`*
```

```py
6.0
```

```py
>>> unicodedata.lookup('RECYCLING SYMBOL FOR TYPE-1 PLASTICS')
```

```py
'♳'
```

¹ 本书仅涵盖了这一遗留特性的子集，即格式说明符，您必须了解它以正确使用日志模块（讨论在“日志模块”）。
