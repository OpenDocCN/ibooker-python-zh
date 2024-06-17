# 第十二章。持久性和数据库

Python 支持几种持久化数据的方式。一种方式是*序列化*，将数据视为 Python 对象的集合。这些对象可以*序列化*（保存）到字节流中，稍后可以从字节流中*反序列化*（加载和重新创建）。*对象持久性*依赖于序列化，添加了诸如对象命名等功能。本章介绍了支持序列化和对象持久性的 Python 模块。

另一种使数据持久化的方法是将其存储在数据库（DB）中。一个简单的 DB 类别是使用*键访问*的文件，以便选择性地读取和更新数据的部分。本章涵盖了支持几种此类文件格式变体的 Python 标准库模块，称为*DBM*。

关系型 DB 管理系统（RDBMS），如 PostgreSQL 或 Oracle，提供了一种更强大的方法来存储、搜索和检索持久化数据。关系型 DB 依赖于结构化查询语言（SQL）的方言来创建和更改 DB 的模式，在 DB 中插入和更新数据，并使用搜索条件查询 DB。（本书不提供 SQL 的参考资料；为此，我们推荐 O'Reilly 的[*SQL in a Nutshell*](https://www.oreilly.com/library/view/sql-in-a/9781492088851)，作者是 Kevin Kline、Regina Obe 和 Leo Hsu。）不幸的是，尽管存在 SQL 标准，但没有两个 RDBMS 实现完全相同的 SQL 方言。

Python 标准库没有提供 RDBMS 接口。然而，许多第三方模块让您的 Python 程序访问特定的 RDBMS。这些模块大多遵循[Python 数据库 API 2.0](https://oreil.ly/sktml)标准，也称为*DBAPI*。本章介绍了 DBAPI 标准，并提到了一些最受欢迎的实现它的第三方模块。

特别方便的 DBAPI 模块（因为它随着每个 Python 标准安装而提供）是[sqlite3](https://oreil.ly/mAq7b)，它封装了[SQLite](https://www.sqlite.org)。SQLite 是“一个自包含的、无服务器的、零配置的、事务性的 SQL DB 引擎”，是世界上部署最广泛的关系型 DB 引擎。我们在“SQLite”中介绍 sqlite3。

除了关系型 DB 和本章介绍的更简单的方法之外，还存在几种[NoSQL](http://nosql-database.org) DB，如[Redis](https://redis.io)和[MongoDB](https://www.mongodb.com)，每种都有 Python 接口。本书不涵盖高级非关系型 DB。

# 序列化

Python 提供了几个模块来将 Python 对象*序列化*（保存）到各种字节流中，并从流中*反序列化*（加载和重新创建）Python 对象。序列化也称为*编组*，意味着格式化用于*数据交换*。

序列化方法涵盖了一个广泛的范围，从低级别的、特定于 Python 版本的 marshal 和独立于语言的 JSON（两者都限于基本数据类型），到更丰富但特定于 Python 的 pickle 和跨语言格式，如 XML、[YAML](http://yaml.org)、[协议缓冲区](https://developers.google.com/protocol-buffers) 和 [MessagePack](http://msgpack.org)。

在本节中，我们涵盖了 Python 的 csv、json、pickle 和 shelve 模块。我们在 第二十三章 中介绍了 XML。marshal 过于低级，不适合在应用程序中使用；如果你需要维护使用它的旧代码，请参考[在线文档](https://oreil.ly/wZZ3s)。至于协议缓冲区、MessagePack、YAML 和其他数据交换/序列化方法（每种都具有特定的优点和缺点），我们无法在本书中覆盖所有内容；我们建议通过网络上可用的资源来学习它们。

## CSV 模块

尽管 CSV（代表*逗号分隔值*¹）格式通常不被视为一种序列化形式，但它是一种广泛使用且方便的表格数据交换格式。由于许多数据是表格形式的，因此尽管在如何在文件中表示它存在一些争议，但 CSV 数据仍然被广泛使用。为了解决这个问题，csv 模块提供了一些*方言*（特定来源编码 CSV 数据方式的规范），并允许你定义自己的方言。你可以注册额外的方言，并通过调用 csv.list_dialects 函数列出可用的方言。有关方言的更多信息，请参阅[模块文档](https://oreil.ly/3_o6_)。

### csv 函数和类

csv 模块公开了 表 12-1 中详细介绍的函数和类。它提供了两种读取器和写入器，让你在 Python 中处理 CSV 数据行时可以选择使用列表或字典。

表 12-1\. csv 模块的函数和类

| reader | reader(*csvfile*, dialect='excel', ***kw*) 创建并返回一个 reader 对象 *r*。*csvfile* 可以是任何产生文本行（通常是行列表或使用 newline='' 打开的文件）的可迭代对象，dialect 是已注册方言的名称。要修改方言，请添加命名参数：它们的值将覆盖相同名称的方言字段。对 *r* 进行迭代将产生一个列表序列，每个列表包含 *csvfile* 的一行元素。 |
| --- | --- |
| writer | writer(*csvfile*, dialect='excel', ***kw*) 创建并返回一个写入对象 *w*。 *csvfile* 是一个带有写入方法的对象（如果是文件，请使用 newline='' 打开）； *dialect* 是一个已注册方言的名称。要修改方言，请添加命名参数：它们的值将覆盖同名的方言字段。 *w.*writerow 接受值序列，并将它们的 CSV 表示作为一行写入 *csvfile*。 *w.*writerows 接受这样的序列的可迭代对象，并对每个调用 *w.*writerow。您有责任关闭 *csvfile*。 |

| D⁠i⁠c⁠t​R⁠e⁠a⁠d⁠e⁠r | DictReader(*csvfile,* fieldnames=**None**, restkey=**None**, restval=**None**, dialect='excel', **args,**kw*)

创建并返回一个对象 *r*，该对象迭代 *csvfile* 以生成一个字典的可迭代对象（-3.8 有序字典），每一行一个字典。当给出 fieldnames 参数时，它用于命名 *csvfile* 中的字段；否则，字段名来自 *csvfile* 的第一行。如果一行包含比字段名更多的列，则额外的值保存为带有键 restkey 的列表。如果任何行中的值不足，则将这些列值设置为 restval。 dialect、*kw* 和 *args* 传递给底层的读取器对象。 |

| DictWriter | DictWriter(*csvfile*, *fieldnames*, restval='', extrasaction='raise', dialect='excel'*, *args, **kwds*)

创建并返回一个对象 *w*，其 writerow 和 writerows 方法接受字典或字典的可迭代对象，并使用 *csvfile* 的写入方法写入它们。 *fieldnames* 是一个 strs 序列，字典的键。 *restval* 是用于填充缺少某些键的字典的值。 extrasaction 指定字典具有未列在 *fieldnames* 中的额外键时该如何处理：当 'raise' 时，默认时，函数在这些情况下引发 ValueError；当 'ignore' 时，函数忽略此类错误。 dialect、*kw* 和 *args* 传递给底层的读取器对象。您有责任关闭 *csvfile*（通常是使用 newline='' 打开的文件）。 |

| ^(a) 使用 newline='' 打开文件允许 csv 模块使用自己的换行处理，并正确处理文本字段可能包含换行符的方言。 |
| --- |

### 一个 csv 示例

这里有一个简单的示例，使用 csv 从字符串列表中读取颜色数据：

```py
`import` csv

color_data = '''\ color,r,g,b
red,255,0,0
green,0,255,0
blue,0,0,255
cyan,0,255,255
magenta,255,0,255
yellow,255,255,0
'''.splitlines()

colors = {row['color']: 
          row `for` row `in` csv.DictReader(color_data)}

print(colors['red']) 
*`# prints: {'color': 'red', 'r': '255', 'g': '0', 'b': '0'}`*
```

请注意，整数值被读取为字符串。csv 不执行任何数据转换；这需要通过您的程序代码与从 DictReader 返回的字典来完成。

## json 模块

标准库的 json 模块支持 Python 本地数据类型（元组、列表、字典、整数、字符串等）的序列化。要序列化自定义类的实例，应实现继承自 JSONEncoder 和 JSONDecoder 的相应类。

### json 函数

json 模块提供了四个关键函数，详见 表 12-2。

表 12-2\. json 模块的函数

| dump | dump(*value*, *fileobj*, skipkeys=**False**, ensure_ascii=**True**, check_circular=**True**, allow_nan=**True**, cls=JSONEncoder, indent=**None**, separators=(', ', ': '), default=**None**, sort_keys=**False**, ***kw*) 将对象*value*的 JSON 序列化写入到文件对象*fileobj*中，*fileobj*必须以文本模式打开进行写入，通过调用*fileobj*.write 来传递文本字符串作为参数。

当 skipkeys 为**True**（默认为**False**）时，非标量类型（即不是 bool、float、int、str 或**None**的键）会引发异常。无论如何，标量类型的键会被转换为字符串（例如，**None**会变成'null'）：JSON 只允许在其映射中使用字符串作为键。

| dump *(cont.)* | 当 ensure_ascii 为**True**（默认值）时，输出中的所有非 ASCII 字符都会被转义；当其为**False**时，它们将原样输出。当 check_circular 为**True**（默认值）时，*value*中的容器会检查循环引用，如果发现任何循环引用，则会引发 ValueError 异常；当其为**False**时，则跳过检查，并可能引发多种不同的异常（甚至可能导致崩溃）。

当 allow_nan 为**True**（默认值）时，浮点标量 nan、inf 和-inf 会输出为它们相应的 JavaScript 等效项 NaN、Infinity 和-Infinity；当其为**False**时，存在这些标量会引发 ValueError 异常。

您可以选择传递 cls 来使用 JSONEncoder 的自定义子类（这种高级定制很少需要，在本书中我们不涵盖这部分）；在这种情况下，***kw*会在实例化 cls 时传递给它的调用中使用。默认情况下，编码使用 JSONEncoder 类直接进行。

当缩进为大于 0 的整数时，dump 函数会在每个数组元素和对象成员前面加上相应数量的空格来实现“美观打印”；当缩进为小于等于 0 的整数时，dump 函数仅插入换行符。当缩进为**None**（默认值）时，dump 函数使用最紧凑的表示方式。缩进也可以是一个字符串，例如'\t'，在这种情况下，dump 函数使用该字符串作为缩进。

separators 必须是一个包含两个元素的元组，分别是用于分隔项的字符串和用于分隔键值对的字符串。您可以显式地传递 separators=(',', ':')来确保 dump 函数不插入任何空白字符。

您可以选择传递 default 以将一些本来不能被序列化的对象转换为可序列化对象。default 是一个函数，接受一个非序列化对象作为参数，并且必须返回一个可序列化对象，或者引发 ValueError 异常（默认情况下，存在非序列化对象会引发 ValueError 异常）。

当 sort_keys 为**True**（默认为**False**）时，映射将按其键的排序顺序输出；当**False**时，它们将按照它们的自然迭代顺序输出（如今，对于大多数映射，是插入顺序）。

| dumps | dumps(*value*, skipkeys=**False**, ensure_ascii=**True**, check_circular=**True**, allow_nan=**True**, cls=JSONEncoder, indent=**None**, separators=(', ', ': '), default=**None**, sort_keys=**False**, ***kw*)

返回一个字符串，该字符串是对象*value*的 JSON 序列化结果，即 dump 将写入其文件对象参数的字符串。dumps 的所有参数与 dump 的参数完全相同。

# JSON 仅序列化一个对象到每个文件中

JSON 并非所谓的*框架格式*：这意味着无法多次调用 dump 来将多个对象序列化到同一个文件中，也不能稍后多次调用 load 来反序列化对象，就像使用 pickle（在下一节讨论）那样。因此，从技术上讲，JSON 仅序列化单个对象到一个文件中。但是，该对象可以是一个列表或字典，其中可以包含任意数量的条目。

|

| load | load(*fileobj*, encoding='utf-8', cls=JSONDecoder, object_hook=**None**, parse_float=float, parse_int=int, parse_constant=**None**, object_pairs_hook=**None**, ***kw*) 创建并返回先前序列化为文件类对象*fileobj*中的对象*v*，*fileobj*必须以文本模式打开，通过调用*fileobj*.read 获取*fileobj*的内容。调用*fileobj*.read 必须返回文本（Unicode）字符串。

函数 load 和 dump 是互补的。换句话说，单个调用 load(*f*)将反序列化在调用 dump(*v*, *f*)时序列化的相同值（可能会有一些修改：例如，所有字典键都变成字符串时）的值。

可以选择传递 cls 以使用 JSONDecoder 的自定义子类（这种高级定制很少需要，在本书中我们不涵盖它）；在这种情况下，***kw*将在调用 cls 时传递，并由其实例化。默认情况下，解码直接使用 JSONDecoder 类。

还可以选择传递 object_hook 或 object_pairs_hook（如果两者都传递，则 object_hook 将被忽略，只使用 object_pairs_hook），这是一个允许您实现自定义解码器的函数。当传递 object_hook 但没有传递 object_pairs_hook 时，每次将对象解码为字典时，load 都会使用 object_hook，并以该字典作为唯一参数调用 object_hook，并使用 object_hook 的返回值而不是该字典。当传递 object_pairs_hook 时，每次解码对象时，load 将使用 object_pairs_hook，并将对象的(*key*, *value*)对的列表作为唯一参数传递，顺序与输入中的顺序相同，并使用 object_pairs_hook 的返回值。这使您可以执行依赖于输入中(*key*, *value*)对顺序的专门解码。

parse_float、parse_int 和 parse_constant 是使用单个参数调用的函数：表示浮点数、整数或三个特殊常量之一（'NaN'、'Infinity' 或 '-Infinity'）的 str。每次识别输入中表示数字的 str 时，load 调用适当的函数，并使用函数的返回值。默认情况下，parse_float 是内置的 float 函数，parse_int 是 int，parse_constant 是一个返回特殊浮点数标量 nan、inf 或 -inf 的函数。例如，可以传递 parse_float=decimal.Decimal 以确保结果中的所有数字正常情况下都是小数（如 “decimal 模块” 中所述）。 |

| loads | loads(*s*, cls=JSONDecoder, object_hook=**None**, parse_float=float, parse_int=int, parse_constant=**None**, object_pairs_hook=**None**, ***kw*) 创建并返回之前已序列化为字符串 *s* 的对象 *v*。loads 的所有参数与 load 的参数完全相同。 |
| --- | --- |

### 一个 JSON 示例

如果需要读取多个文本文件，其文件名作为程序参数给出，并记录每个单词在文件中出现的位置，你需要记录每个单词的 (*文件名*, *行号*) 对列表。以下示例使用 fileinput 模块迭代所有作为程序参数给出的文件，并使用 json 将 (*文件名*, *行号*) 对列表编码为字符串，并存储在类似 DBM 的文件中（如 “DBM 模块” 中所述）。由于这些列表包含元组，每个元组包含字符串和数字，它们可以被 json 序列化：

```py
`import` collections, fileinput, json, dbm
word_pos = collections.defaultdict(list)
`for` line `in` fileinput.input():
    pos = fileinput.filename(), fileinput.filelineno()
    `for` word `in` line.split():
        word_pos[word].append(pos)
`with` dbm.open('indexfilem', 'n') `as` dbm_out:
    `for` word, word_positions `in` word_pos.items():
        dbm_out[word] = json.dumps(word_positions)
```

然后，我们可以使用 json 解序列化存储在类似 DBM 文件 *indexfilem* 中的数据，如以下示例所示：

```py
`import` sys, json, dbm, linecache
`with` dbm.open('indexfilem') `as` dbm_in:
 `for` word `in` sys.argv[1:]:
        `if` word `not` `in` dbm_in:
             print(f'Word {word!r} not found in index file',                                             file=sys.stderr)
 `continue`
        places = json.loads(dbm_in[word])
        `for` fname, lineno `in` places:
            print(f'Word {word!r} occurs in line {lineno}'
                  f' of file {fname!r}:')
            print(linecache.getline(fname, lineno), end='')
```

## pickle 模块

pickle 模块提供了名为 Pickler 和 Unpickler 的工厂函数，用于生成对象（不可子类化类型的实例，而不是类），这些对象包装文件并提供 Python 特定的序列化机制。通过这些模块进行序列化和反序列化也称为 *pickling* 和 *unpickling*。

序列化与深拷贝具有某些相同的问题，如 “copy 模块” 中所述。pickle 模块处理这些问题的方式与 copy 模块非常相似。序列化，就像深拷贝一样，意味着在引用的有向图上进行递归遍历。pickle 保留了图的形状：当多次遇到相同的对象时，仅第一次序列化该对象，其他出现的相同对象序列化为对该单一值的引用。pickle 还正确地序列化具有引用循环的图。然而，这意味着如果可变对象 *o* 被序列化多次到同一个 Pickler 实例 *p*，则在第一次将 *o* 序列化到 *p* 后对 *o* 的任何更改都不会被保存。

# 在序列化正在进行时不要更改对象

为了清晰、正确和简单起见，在 Pickler 实例序列化过程中不要更改正在序列化的对象。

pickle 可以使用遗留 ASCII 协议或多个紧凑的二进制协议进行序列化。表 12-3 列出了可用的协议。

表 12-3\. pickle 协议

| 协议 | 格式 | Python 版本新增 | 描述 |
| --- | --- | --- | --- |
| 0 | ASCII | 1.4^(a) | 可读性强，序列化/反序列化速度慢 |
| 1 | 二进制 | 1.5 | 早期二进制格式，被协议 2 取代 |
| 2 | 二进制 | 2.3 | 改进对后期 Python 2 特性的支持 |
| 3 | 二进制 | 3.0 | （-3.8 默认）增加对字节对象的具体支持 |
| 4 | 二进制 | 3.4 | （3.8+ 默认）支持非常大的对象 |
| 5 | 二进制 | 3.8 | 3.8+ 添加了支持作为传输过程中序列化的 pickling 特性，参见 [PEP 574](https://oreil.ly/PcSYs) |
| ^(a) 或可能更早。这是可在 [Python.org](http://Python.org) 上找到的最古老版本的文档。 |

# 始终使用协议 2 或更高版本进行 Pickle

始终使用*至少*协议 2。尺寸和速度节省可观，并且二进制格式基本没有任何缺点，除了导致生成的 pickle 与真正古老版本的 Python 不兼容之外。

当重新加载对象时，pickle 会透明地识别并使用当前 Python 版本支持的任何协议。

pickle（腌制）通过名称而非数值序列化类和函数²。因此，pickle 只能在反序列化时从与 pickle 序列化时相同模块中导入类或函数。特别地，pickle 通常只能序列化和反序列化类和函数，如果它们是其各自模块的顶级名称（即属性）。考虑以下示例：

```py
`def` adder(augend):
    `def` inner(addend, augend=augend):
        `return` addend+augend
    `return` inner
plus5 = adder(5)
```

此代码将一个闭包绑定到名称 plus5（如“嵌套函数和嵌套作用域”中所述）——一个内部函数 inner 与适当的外部作用域。因此，尝试对 plus5 进行 pickle 会引发 AttributeError：只有当函数处于顶级时，才能对其进行 pickle，而此代码中其闭包绑定到名称 plus5 的函数 inner 并非顶级，而是嵌套在函数 adder 内部。类似的问题也适用于序列化嵌套函数和嵌套类（即不在顶层的类）。

### pickle 函数和类

pickle 模块公开了表 12-4 中列出的函数和类。

表 12-4\. pickle 模块的函数和类

| dump, dumps | dump(*value*, *fileobj,* protocol=**None**, bin=**None**), dumps(*value*, protocol=**None**, bin=**None**)

dumps 返回表示对象 *value* 的字节串。dump 将相同的字符串写入类似文件的对象 *fileobj*，该对象必须已打开以供写入。dump(*v*, *f*) 就像 *f*.write(dumps(*v*))。protocol 参数可以是 0（ASCII 输出，最慢和最庞大的选项），或者更大的整数表示各种类型的二进制输出（参见 Table 12-3）。除非 protocol 是 0，否则传递给 dump 的 *fileobj* 参数必须已打开以供二进制写入。不要传递 bin 参数，它仅为与旧版本 Python 的兼容性而存在。 |

| load, loads | load(*fileobj*), loads(*s*, *, fix_imports=True, encoding="ASCII", errors="strict")

函数 load 和 dump 是互补的。换句话说，对 load(*f*) 的一系列调用将反序列化与之前通过对 dump(*v, f*) 进行一系列调用而创建 *f* 内容时序列化的相同值。load 从类似文件的对象 *fileobj* 中读取正确数量的字节，并创建并返回由这些字节表示的对象 *v*。load 和 loads 透明地支持在任何二进制或 ASCII 协议中执行的 pickles。如果数据以任何二进制格式进行 pickle，文件必须对于 dump 和 load 都以二进制方式打开。load(*f*) 就像 Unpickler(*f*).load()。 |

| load, loads

*(cont.)* | loads 创建并返回由字节串 *s* 表示的对象 *v*，因此对于任何支持的类型的对象 *v*，*v*==loads(dumps(*v*))。如果 *s* 比 dumps(*v*) 长，loads 会忽略额外的字节。提供了可选参数 fix_imports、encoding 和 errors 来处理由 Python 2 代码生成的流；请参阅 [pickle.loads 文档](https://oreil.ly/VSepJ) 以获取更多信息。

# 永远不要反序列化不受信任的数据

从不受信任的数据源进行反序列化是一种安全风险；攻击者可能利用此漏洞执行任意代码。

|

| 序列化器 | 序列化器(*fileobj*, protocol=**None**, bin=**None**) 创建并返回一个对象 *p*，使得调用 *p*.dump 相当于调用 dump 函数并传递给 Pickler *fileobj*、protocol 和 bin 参数。为了将多个对象序列化到文件中，Pickler 比重复调用 dump 更方便且更快。你可以子类化 pickle.Pickler 来覆盖 Pickler 方法（尤其是 persistent_id 方法）并创建你自己的持久化框架。然而，这是一个高级主题，在本书中不再进一步讨论。 |
| --- | --- |
| 反序列化器 | 反序列化器(*fileobj*) 创建并返回一个对象 *u*，使得调用 *u*.load 相当于调用 load 并传递 *fileobj* 参数给 Unpickler。为了从文件中反序列化多个对象，Unpickler 比重复调用 load 函数更方便且更快。你可以子类化 pickle.Unpickler 来覆盖 Unpickler 方法（尤其是 persistent_load 方法）并创建你自己的持久化框架。然而，这是一个高级主题，在本书中不再进一步讨论。 |

### 一个序列化的例子

以下示例处理与之前显示的 json 示例相同的任务，但使用 pickle 而不是 json 将(*filename*, *linenumber*)对的列表序列化为字符串：

```py
`import` collections, fileinput, pickle, dbm
word_pos = collections.defaultdict(list)
`for` line `in` fileinput.input():
    pos = fileinput.filename(), fileinput.filelineno()
    `for` word `in` line.split():
        word_pos[word].append(pos)

`with` dbm.open('indexfilep', 'n') `as` dbm_out:
    `for` word, word_positions `in` word_pos.items():
        dbm_out[word] = pickle.dumps(word_positions, protocol=2)
```

然后，我们可以使用 pickle 从类似 DBM 的文件*indexfilep*中读回存储的数据，如下例所示：

```py
`import` sys, pickle, dbm, linecache
`with` dbm.open('indexfilep') `as` dbm_in:
 `for` word `in` sys.argv[1:]:
        `if` word `not` `in` dbm_in:
            print(f'Word {word!r} not found in index file',
                  file=sys.stderr)
 `continue`
        places = pickle.loads(dbm_in[word])
        `for` fname, lineno `in` places:
            print(f'Word {word!r} occurs in line {lineno}'
                  f' of file {fname!r}:')
            print(linecache.getline(fname, lineno), end='')
```

### 对实例进行 pickle

为了让 pickle 重新加载实例*x*，pickle 必须能够从 pickle 保存实例时定义类的同一模块中导入*x*的类。以下是 pickle 如何保存类*T*的实例对象*x*的状态，并将保存的状态重新加载到类*T*的新实例*y*中（重新加载的第一步始终是创建*T*的新空实例*y*，除非另有明确说明）：

+   当*T*提供方法 __getstate__ 时，pickle 保存调用*T*.__getstate__(*x*)的结果*d*。

+   当*T*提供方法 __setstate__ 时，*d*可以是任何类型，并且 pickle 通过调用*T*.__setstate__(*y, d*)重新加载保存的状态。

+   否则，*d*必须是一个字典，pickle 只需设置*y*.__dict__ = *d*。

+   否则，当*T*提供方法 __getnewargs__，并且 pickle 使用协议 2 或更高版本进行 pickle 时，pickle 保存调用*T*.__getnewargs__(*x*)的结果*t*；*t*必须是一个元组。

+   在这种情况下，pickle 不会从空*y*开始，而是通过执行*y* = *T*.__new__(*T*, **t*)来创建*y*，从而完成重新加载。

+   否则，默认情况下，pickle 将*x*.__dict__ 保存为字典*d*。

+   当*T*提供方法 __setstate__ 时，pickle 通过调用*T*.__setstate__(*y, d*)重新加载保存的状态。

+   否则，pickle 只需设置*y*.__dict__ = *d*。

pickle 保存和重新加载的*d*或*t*对象中的所有项（通常是字典或元组）必须依次是适合进行 pickle 和 unpickle（即*pickleable*）的类型的实例，并且如有必要，该过程可以递归重复进行，直到 pickle 到达原始的 pickleable 内置类型（如字典、元组、列表、集合、数字、字符串等）。

如“copy 模块”中所述，__getnewargs__、__getstate__ 和 __setstate__ 特殊方法还控制实例对象的复制和深度复制方式。如果一个类定义了 __slots__，因此其实例没有 __dict__ 属性，pickle 会尽力保存和恢复等同于 slots 名称和值的字典。然而，这样的类应该定义 __getstate__ 和 __setstate__；否则，其实例可能无法正确进行 pickle 和复制。

### 使用 copyreg 模块进行 pickle 定制

通过向 copyreg 模块注册工厂和减少函数，可以控制 pickle 如何序列化和反序列化任意类型的对象。当您在 C 代码的 Python 扩展中定义类型时，这尤为有用。copyreg 模块提供了 表 12-5 中列出的函数。

表 12-5\. copyreg 模块的函数

| constructor | constructor(*fcon*) 将 *fcon* 添加到构造函数表中，该表列出 pickle 可能调用的所有工厂函数。*fcon* 必须是可调用的，通常是一个函数。 |
| --- | --- |

| pickle | pickle(*type*, *fred*, fcon=**None**) 将函数 *fred* 注册为类型 *type* 的减少函数，其中 *type* 必须是一个类型对象。要保存类型为 *type* 的对象 *o*，模块 pickle 调用 *fred*(*o*) 并保存结果。*fred*(*o*) 必须返回一个元组 (fcon, *t*) 或 (fcon, *t*, *d*)，其中 *fcon* 是一个构造函数，*t* 是一个元组。要重新加载 *o*，pickle 使用 *o*=fcon(**t*)。然后，当 *fred* 还返回 *d* 时，pickle 使用 *d* 来恢复 *o* 的状态（如果 *o* 提供了 __setstate__，则 *o*.__setstate__(*d*)；否则，*o*.__dict__.update(*d*)），如前一节所述。如果 *fcon* 不为 **None**，pickle 还会调用构造函数 (*fcon*) 来注册 *fcon* 作为构造函数。

pickle 不支持对代码对象的 pickle 操作，但 marshal 支持。以下是如何通过利用 copyreg 将 pickle 定制以支持代码对象的示例：

```py
>>> `import` pickle, copyreg, marshal
>>> `def` marsh(x):
...     `return` marshal.loads, (marshal.dumps(x),)
...
>>> c=compile('2+2','','eval')
>>> copyreg.pickle(type(c), marsh)
>>> s=pickle.dumps(c, 2)
>>> cc=pickle.loads(s)
>>> print(eval(cc))
```

```py
4
```

# 使用 marshal 使你的代码依赖于 Python 版本

在你的代码中使用 marshal 时要小心，就像前面的示例一样。marshal 的序列化不能保证跨版本稳定，因此使用 marshal 意味着其他版本的 Python 编写的程序可能无法加载你的程序序列化的对象。

|

## shelve 模块

shelve 模块通过协调 pickle、io 和 dbm（及其底层访问 DBM 类型归档文件的模块，如下一节所述），提供了一个简单、轻量级的持久化机制。

shelve 提供了一个函数 open，其多态性类似于 dbm.open。shelve.open 返回的映射 *s* 比 dbm.open 返回的映射 *a* 更为灵活。*a* 的键和值必须是字符串。³ *s* 的键也必须是字符串，但 *s* 的值可以是任何可 pickle 的类型。pickle 定制（copyreg、__getnewargs__、__getstate__ 和 __setstate__）同样适用于 shelve，因为 shelve 将序列化工作委托给 pickle。键和值以字节形式存储。使用字符串时，在存储之前会隐式地转换为默认编码。

当你在使用 shelve 与可变对象时要小心一个微妙的陷阱：当你对一个存储在 shelf 中的可变对象进行操作时，除非将更改的对象重新分配回相同的索引，否则更改不会存储回去。例如：

```py
`import` shelve
s = shelve.open('data')
s['akey'] = list(range(4))
print(s['akey'])           *`# prints: [0, 1, 2, 3]`*
s['akey'].append(9)        *`# trying direct mutation`*
print(s['akey'])           *`# doesn't "take"; prints: [0, 1, 2, 3]`*
x = s['akey']              *`# fetch the object`*
x.append(9)                *`# perform mutation`*
s['akey'] = x              *`# key step: store the object back!`*
print(s['akey'])           *`# now it "takes", prints: [0, 1, 2, 3, 9]`*
```

当调用 shelve.open 时，通过传递命名参数 writeback=**True**，可以解决这个问题，但这可能严重影响程序的性能。

### 一个 shelve 示例

下面的示例处理与之前的 json 和 pickle 示例相同的任务，但使用 shelve 来持久化 (*filename*, *linenumber*) 对的列表：

```py
`import` collections, fileinput, shelve
word_pos = collections.defaultdict(list)
`for` line `in` fileinput.input():
    pos = fileinput.filename(), fileinput.filelineno()
    `for` word `in` line.split():
        word_pos[word].append(pos)
`with` shelve.open('indexfiles','n') `as` sh_out:
    sh_out.update(word_pos)
```

然后，我们必须使用 shelve 读取存储到类似 DBM 的文件 *indexfiles* 中的数据，如下例所示：

```py
`import` sys, shelve, linecache
`with` shelve.open('indexfiles') `as` sh_in:
 `for` word `in` sys.argv[1:]:
        if word `not` `in` sh_in:
            print(f'Word {word!r} not found in index file',
			      file=sys.stderr)
 `continue`
        places = sh_in[word]
        `for` fname, lineno `in` places:
            print(f'Word {word!r} occurs in line {lineno}'
                  f' of file {fname!r}:')
            print(linecache.getline(fname, lineno), end='')
```

这两个示例是本节中显示的各种等效示例中最简单、最直接的示例。这反映了 shelve 比之前示例中使用的模块更高级的事实。

# DBM 模块

[DBM](https://oreil.ly/osARc)，长期以来是 Unix 的主要组成部分，是一组支持包含字节串对 (*key*, *data*) 的数据文件的库。DBM 提供了根据键快速获取和存储数据的功能，这种使用模式称为 *键访问*。尽管键访问远不及关系数据库的数据访问功能强大，但它的开销较小，对于某些程序的需求可能足够。如果 DBM 类似的文件对您的目的足够，通过这种方法，您可以得到一个比使用关系数据库更小更快的程序。

# DBM 数据库以字节为导向

DBM 数据库要求键和值都是字节值。稍后包含的示例中，您将看到文本输入在存储之前被明确编码为 UTF-8。类似地，在读取值时必须执行逆解码。

Python 标准库中的 DBM 支持以一种清晰而优雅的方式组织：dbm 包公开了两个通用函数，同一包中还有其他模块提供特定的实现。

# Berkeley DB 接口

bsddb 模块已从 Python 标准库中移除。如果您需要与 BSD DB 存档交互，我们建议使用出色的第三方包 [bsddb3](https://oreil.ly/xizEg)。

## DBM 包

dbm 包提供了 表 12-6 中描述的顶层函数。

表 12-6\. dbm 包的函数

| open | open(*filepath*, flag='r', mode=0o666) 打开或创建由 *filepath*（任何文件的路径）指定的 DBM 文件，并返回与 DBM 文件对应的映射对象。当 DBM 文件已经存在时，open 使用 whichdb 函数来确定哪个 DBM 子模块可以处理文件。当 open 创建新的 DBM 文件时，它会按照以下偏好顺序选择第一个可用的 dbm 子模块：gnu、ndbm、dumb。

flag 是一个告诉 open 如何打开文件以及是否创建文件的单字符字符串，根据 表 12-7 中所示的规则。mode 是一个整数，如果 open 创建文件，则 open 会将其用作文件的权限位，如 “使用 open 创建文件对象” 中所述。

表 12-7\. dbm.open 的标志值

&#124; 标志 &#124; 只读？ &#124; 如果文件存在： &#124; 如果文件不存在： &#124;

&#124; --- &#124; --- &#124; --- &#124; --- &#124;

&#124; 'r' &#124; 是 &#124; 打开文件 &#124; 报错 &#124;

&#124; 'w' &#124; 否 &#124; 打开文件 &#124; 报错 &#124;

&#124; 'c' &#124; 否 &#124; 打开文件 &#124; 创建文件 &#124;

&#124; 'n' &#124; 否 &#124; 截断文件 &#124; 创建文件 &#124;

dbm.open 返回一个映射对象*m*，其功能子集类似于字典（详见“Dictionary Operations”）。*m*只接受字节作为键和值，而且*m*提供的唯一非特殊映射方法是*m*.get、*m*.keys 和*m*.setdefault。你可以使用与字典相同的索引语法*m*[*键*]绑定、重新绑定、访问和解绑*m*中的项目。如果标志是'r'，则*m*是只读的，因此你只能访问*m*的项目，而不能绑定、重新绑定或解绑它们。你可以使用通常的表达式*s* **in** *m*检查字符串*s*是否是*m*的键；你不能直接在*m*上进行迭代，但可以等效地在*m*.keys()上进行迭代。

*m*提供的一个额外方法是*m*.close，其语义与文件对象的 close 方法相同。与文件对象一样，当你使用完*m*后，应确保调用*m*.close。使用“try/finally”中介绍的**try**/**finally**语句是确保最终化的一种方式，但使用“The with Statement and Context Managers”中介绍的**with**语句更好（因为*m*是上下文管理器，你可以使用**with**）。 |

| &#124; whichdb &#124; whichdb(*文件路径*) 打开并读取指定的*文件路径*，以确定创建文件的 dbm 子模块。当文件不存在或无法打开和读取时，whichdb 返回**None**。当文件存在并可以打开和读取，但无法确定创建文件的 dbm 子模块时（通常意味着文件不是 DBM 文件），whichdb 返回''。如果可以找出哪个模块可以读取类似 DBM 的文件，whichdb 返回一个命名 dbm 子模块的字符串，例如'dbm.ndbm'、'dbm.dumb'或'dbm.gdbm'。 |
| --- |

除了这两个顶级函数外，dbm 包还包含特定模块，如 ndbm、gnu 和 dumb，提供各种 DBM 功能的实现，通常你只通过这些顶级函数访问。第三方包可以在 dbm 中安装更多的实现模块。

DBM 包唯一保证在所有平台上存在的实现模块是 dumb。dumb 提供了最小的 DBM 功能和一般性能；其唯一优点是您可以在任何地方使用，因为 dumb 不依赖于任何库。通常您不会直接 **import** dbm.dumb：而是 **import** dbm，让 dbm.open 在当前 Python 安装中提供最好的可用 DBM 模块，如果没有更好的子模块，则默认使用 dumb。唯一需要直接导入 dumb 的情况是在需要创建一个在任何 Python 安装中都可读的类似 DBM 的文件时。dumb 模块提供了一个 open 函数，与 dbm 的 open 函数多态。

## DBM-Like 文件的使用示例

DBM 的键控访问适合在程序需要持久记录类似 Python 字典的等效内容时使用，其中键和值都是字符串。例如，假设您需要分析几个文本文件，这些文件名是作为程序参数给出的，并记录每个单词在这些文件中出现的位置。在这种情况下，键是单词，因此本质上是字符串。您需要记录的每个单词的数据是一个 (*filename*, *linenumber*) 对的列表。然而，您可以通过几种方法将数据编码为字符串，例如利用路径分隔符字符串 `os.pathsep`（在“os 模块的路径字符串属性”中介绍），因为该字符串通常不会出现在文件名中。（关于将数据编码为字符串的更一般方法在本章开头的部分有介绍，使用了相同的例子。）在这种简化情况下，编写一个记录文件中单词位置的程序可能如下所示：

```py
`import` collections, fileinput, os, dbm
word_pos = collections.defaultdict(list)
`for` line `in` fileinput.input():
    pos = f'{fileinput.filename()}{os.pathsep}{fileinput.filelineno()}'
    `for` word `in` line.split():
        word_pos[word].append(pos)
sep2 = os.pathsep * 2
`with` dbm.open('indexfile','n') `as` dbm_out:
    `for` word `in` word_pos:
        dbm_out[word.encode('utf-8')] = sep2.join(
			word_pos[word]
		).encode('utf-8')
```

您可以通过几种方式读取存储在类似 DBM 文件 *indexfile* 中的数据。下面的示例接受单词作为命令行参数，并打印请求单词出现的行：

```py
`import` sys, os, dbm, linecache

sep = os.pathsep
sep2 = sep * 2
`with` dbm.open('indexfile') `as` dbm_in:
 `for` word `in` sys.argv[1:]:
        e_word = word.encode('utf-8')
        `if` e_word `not` `in` dbm_in:
            print(f'Word {word!r} not found in index file',
                  file=sys.stderr)
            `continue`
        places = dbm_in[e_word].decode('utf-8').split(sep2)
        `for` place `in` places:
            fname, lineno = place.split(sep)
            print(f'Word {word!r} occurs in line {lineno}'
                  f' of file {fname!r}:')
            print(linecache.getline(fname, int(lineno)), end='')
```

# Python 数据库 API（DBAPI）

正如前面提到的，Python 标准库并不附带关系数据库管理系统接口（除了 sqlite3，在“SQLite”中介绍的，它是一个丰富的实现，而不仅仅是接口）。许多第三方模块允许您的 Python 程序访问特定的数据库。这些模块大多遵循 Python 数据库 API 2.0 标准，也称为 DBAPI，如 [PEP 249](https://oreil.ly/-yhzm) 中所述。

导入任何符合 DBAPI 标准的模块后，您可以使用特定于 DB 的参数调用模块的 connect 函数。connect 返回 *x*，Connection 的一个实例，表示与 DB 的连接。*x* 提供 commit 和 rollback 方法来处理事务，提供一个 close 方法，在您完成 DB 操作后调用，以及一个 cursor 方法来返回 *c*，Cursor 的一个实例。*c* 提供了用于 DB 操作的方法和属性。符合 DBAPI 标准的模块还提供了异常类、描述性属性、工厂函数和类型描述属性。

## 异常类

符合 DBAPI 标准的模块提供了异常类 Warning、Error 和 Error 的几个子类。Warning 指示插入时的数据截断等异常。Error 的子类指示您的程序在处理与 DB 和与之接口的符合 DBAPI 标准的模块时可能遇到的各种错误。通常，您的代码使用以下形式的语句：

```py
`try`:
    ...
`except` module.Error `as` err:
    ...
```

以捕获您需要处理而不终止的所有与 DB 相关的错误。

## 线程安全

当 DBAPI 兼容的模块具有大于 0 的 threadsafety 属性时，该模块在 DB 接口中断言了某种程度的线程安全性。与依赖于此不同，通常更安全，且始终更可移植，要确保单个线程对任何给定的外部资源（如 DB）具有独占访问权，如 “线程化程序架构” 中所述。

## 参数样式

符合 DBAPI 标准的模块有一个称为 paramstyle 的属性，用于识别用作参数占位符的标记样式。在传递给 Cursor 实例方法（如 execute 方法）的 SQL 语句字符串中插入这些标记，以使用运行时确定的参数值。举个例子，假设您需要获取字段 *AFIELD* 等于 Python 变量 *x* 当前值的 DB 表 *ATABLE* 的行。假设光标实例命名为 *c*，您*理论上*（但非常不建议！）可以使用 Python 的字符串格式化执行此任务：

```py
c.execute(f'SELECT * FROM ATABLE WHERE AFIELD={x!r}')
```

# 避免使用 SQL 查询字符串格式化：使用参数替换

字符串格式化*不*是推荐的方法。它为每个 *x* 的值生成不同的字符串，每次都需要解析和准备语句；它还存在安全弱点的可能性，如 [SQL 注入](https://oreil.ly/hpUlv) 漏洞。使用参数替换，您将传递一个具有占位符而不是参数值的单个语句字符串给 execute。这样一来，execute 只需解析和准备语句一次，以获得更好的性能；更重要的是，参数替换提高了稳固性和安全性，阻碍了 SQL 注入攻击。

例如，当模块的 paramstyle 属性（下文描述）为 'qmark' 时，您可以将前面的查询表示为：

```py
c.execute('SELECT * FROM ATABLE WHERE AFIELD=?', (some_value,))
```

只读字符串属性 paramstyle 告诉您的程序如何使用该模块进行参数替换。paramstyle 的可能值如 表 12-8 中所示。

表 12-8\. paramstyle 属性的可能值

| format | 标记是 %s，就像旧式字符串格式化一样（始终使用 s：不要使用其他类型指示符字母，无论数据的类型是什么）。一个查询看起来像：

```py
c.execute('SELECT * FROM ATABLE WHERE AFIELD=%s', 
          (some_value,))
```

|

| named | 标记是：*name*，参数是命名的。一个查询看起来像：

```py
c.execute('SELECT * FROM ATABLE WHERE AFIELD=:x', 
          {'x':some_value})
```

|

| numeric | 标记是：n，给出参数的编号，从 1 开始。一个查询看起来像：

```py
c.execute('SELECT * FROM ATABLE WHERE AFIELD=:1', 
          (some_value,))
```

|

| pyformat | 标记为 %(*name*)s，参数带有命名。始终使用 s：永不使用其他类型指示符，无论数据类型如何。查询的样子是：

```py
c.execute('SELECT * FROM ATABLE WHERE AFIELD=%(x)s',
          {'x':some_value})
```

|

| qmark | 标记为 ?。查询的样子是：

```py
c.execute('SELECT * FROM ATABLE WHERE AFIELD=?', (x,))
```

|

当参数被命名时（即 paramstyle 是 'pyformat' 或 'named'），execute 方法的第二个参数是一个映射。否则，第二个参数是一个序列。

# format 和 pyformat 只接受类型指示符 s

格式或 pyformat 的*唯一*有效类型指示符是 s；不接受任何其他类型指示符——例如，永远不要使用 %d 或 %(*name*)d。无论数据类型如何，都要使用 %s 或 %(*name*)s 进行所有参数替换。

## 工厂函数

通过占位符传递给数据库的参数通常必须是正确的类型：这意味着 Python 数字（整数或浮点数值）、字符串（字节或 Unicode）以及**None**表示 SQL NULL。没有普遍用于表示日期、时间和二进制大对象（BLOBs）的类型。DBAPI 兼容模块提供工厂函数来构建这些对象。大多数 DBAPI 兼容模块用于此目的的类型由 datetime 模块提供（在第 13 章中详述），并且用于 BLOBs 的类型为字符串或缓冲区类型。DBAPI 指定的工厂函数列在表 12-9 中。（*FromTicks 方法接受整数时间戳 *s*，表示自模块 time 纪元以来的秒数，在第 13 章中详述。）

表 12-9\. DBAPI 工厂函数

| 二进制 | Binary(*string*) 返回表示给定字节*string*的对象作为 BLOB。 |
| --- | --- |
| 日期 | Date(*year*, *month*, *day*) 返回表示指定日期的对象。 |
| DateFromTicks | DateFromTicks(*s*) 返回表示整数时间戳 *s* 的日期对象。例如，DateFromTicks(time.time()) 表示“今天”。 |
| 时间 | Time(*hour*, *minute*, *second*) 返回表示指定时间的对象。 |
| TimeFromTicks | TimeFromTicks(*s*) 返回表示整数时间戳 *s* 的时间对象。例如，TimeFromTicks(time.time()) 表示“当前时间”。 |
| 时间戳 | Timestamp(*year*, *month*, *day*, *hour*, *minute*, *second*) 返回表示指定日期和时间的对象。 |
| TimestampFromTicks | TimestampFromTicks(*s*) 返回表示整数时间戳 *s* 的日期和时间对象。例如，TimestampFromTicks(time.time()) 是当前日期和时间。 |

## 类型描述属性

游标实例的 description 属性描述了您在该游标上最后执行的每个 SELECT 查询的列的类型和其他特征。每列的*类型*（描述列的元组的第二项）等于 DBAPI 兼容模块的以下属性之一：

| 二进制 | 描述包含 BLOBs 的列 |
| --- | --- |
| DATETIME | 描述包含日期、时间或两者的列 |
| NUMBER | 描述包含任何类型数字的列 |
| ROWID | 描述包含行标识号的列 |
| STRING | 描述包含任何类型文本的列 |

一个 cursor 的描述，尤其是每一列的类型，对于了解程序所使用的 DB 是非常有用的。这种内省可以帮助你编写通用的模块，并使用不同模式的表，包括在编写代码时可能不知道的模式。

## connect 函数

一个 DBAPI 兼容模块的 `connect` 函数接受依赖于 DB 类型和具体模块的参数。DBAPI 标准建议 `connect` 接受命名参数。特别是，`connect` 至少应接受以下名称的可选参数：

| database | 要连接的具体数据库名称 |
| --- | --- |
| dsn | 用于连接的数据源名称 |
| host | 数据库运行的主机名 |
| password | 用于连接的密码 |
| user | 用于连接的用户名 |

## Connection 对象

一个 DBAPI 兼容模块的 `connect` 函数返回一个对象 *x*，它是 `Connection` 类的一个实例。 *x* 提供了 Table 12-10 中列出的方法。

Table 12-10\. 类 Connection 的实例 x 的方法

| close | *x*.close() 终止 DB 连接并释放所有相关资源。在完成 DB 操作后立即调用 close。不必要地保持 DB 连接开启可能会严重消耗系统资源。 |
| --- | --- |
| commit | *x*.commit() 提交当前的 DB 事务。如果 DB 不支持事务，则 *x*.commit() 不会有任何操作。 |
| cursor | *x*.cursor() 返回 `Cursor` 类的一个新实例（在下一节中介绍）。 |
| rollback | *x*.rollback() 回滚当前的 DB 事务。如果 DB 不支持事务，则 *x*.rollback() 会引发异常。DBAPI 建议，对于不支持事务的 DB，`Connection` 类不应提供回滚方法，因此 *x*.rollback() 会引发 AttributeError：你可以通过 hasattr(*x*, 'rollback') 测试是否支持事务。 |

## Cursor 对象

连接实例提供了一个 cursor 方法，返回一个名为 *c* 的 Cursor 类实例对象。SQL 游标表示查询结果集，并允许您按顺序逐个处理该集合中的记录。由 DBAPI 建模的游标是一个更丰富的概念，因为它是程序执行 SQL 查询的唯一方式。另一方面，DBAPI 游标只允许您在结果序列中前进（一些关系型数据库，但不是所有的，还提供更高功能的游标，可以前后移动），并且不支持 SQL 子句 WHERE CURRENT OF CURSOR。DBAPI 游标的这些限制使得 DBAPI 兼容模块能够在根本不提供真正 SQL 游标的 RDBMS 上提供 DBAPI 游标。类 Cursor 的实例 *c* 提供了许多属性和方法；最常用的属性和方法如 Table 12-11 所示。 |

表 12-11\. 类 Cursor 的实例 *c* 的常用属性和方法

| close | *c*.close() 关闭游标并释放所有相关资源。 |
| --- | --- |

| description | 只读属性，是一个由七项元组组成的序列，每项对应最后执行的查询中的一个列：名称、类型代码、显示大小、内部大小、精度、比例。

可为空

*c*.description 如果最后对 *c* 的操作不是 SELECT 查询或返回的列描述不可用，则为 **None**。游标的描述主要用于关于程序正在使用的数据库的内省。这种内省可以帮助您编写通用模块，能够处理使用不同模式的表，包括编写代码时可能不完全了解的模式。 |

| execute | *c*.execute(*statement*, parameters=**None**) 在给定参数的情况下，在 DB 上执行 SQL *statement* 字符串。当模块的 paramstyle 为 'format'、'numeric' 或 'qmark' 时，parameters 是一个序列；当 paramstyle 为 'named' 或 'pyformat' 时，parameters 是一个映射。某些 DBAPI 模块要求序列必须明确为元组。 |
| --- | --- |

| executemany | *c*.executemany(*statement*, **parameters*) 对 DB 执行 SQL *statement*，对给定的 *parameters* 中的每个项执行一次。当模块的 paramstyle 为 'format'、'numeric' 或 'qmark' 时，parameters 是一个序列的序列；当 paramstyle 为 'named' 或 'pyformat' 时，parameters 是映射的序列。例如，当 paramstyle 为 'qmark' 时，语句：

```py
c.executemany('UPDATE atable SET x=? '
              'WHERE y=?',(12,23),(23,34))
```

等同于但比以下两个语句更快：

```py
c.execute('UPDATE atable SET x=12 WHERE y=23')
c.execute('UPDATE atable SET x=23 WHERE y=34')
```

|

| fetchall | *c*.fetchall() 返回最后查询的所有剩余行作为元组序列。如果最后的操作不是 SELECT，则会引发异常。 |
| --- | --- |
| fetchmany | *c*.fetchmany(*n*) 返回最后查询的最多 *n* 行作为元组序列。如果最后的操作不是 SELECT，则会引发异常。 |
| fetchone | c.fetchone() 将从上次查询中返回下一行作为元组。如果上次操作不是 SELECT，则引发异常。 |
| rowcount | 一个只读属性，指定了最后一个操作获取或影响的行数，如果模块无法确定这个值，则为 -1。 |

## DBAPI 兼容模块

无论你想使用哪种关系型数据库，至少有一个（通常是多个）Python DBAPI 兼容模块可以从互联网上下载。有这么多的数据库和模块，可能性的集合如此不断变化，我们不可能列出所有的，也无法长期维护这个列表。因此，我们建议你从社区维护的 [wiki 页面](https://oreil.ly/ubKe7) 开始，这个页面有可能在任何时候都是完整和最新的。

因此，接下来只是一个非常短暂且特定时间的，与撰写时非常流行且与非常流行的开源数据库接口的非常少量 DBAPI 兼容模块的列表：

ODBC 模块

Open Database Connectivity (ODBC) 是连接许多不同的数据库的标准方法，包括一些其他 DBAPI 兼容模块不支持的数据库。对于具有自由开源许可的 ODBC 兼容 DBAPI 兼容模块，请使用 [pyodbc](https://oreil.ly/MNAt9)；对于商业支持的模块，请使用 [mxODBC](https://oreil.ly/hPUU0)。

MySQL 模块

MySQL 是一个流行的开源关系型数据库管理系统，于 2010 年被 Oracle 收购。Oracle 提供的“官方”DBAPI 兼容接口是 [mysql-connector-python](https://oreil.ly/iWzpg)。MariaDB 项目也提供了一个 DBAPI 兼容接口，[mariadb](https://oreil.ly/zmCLT)，连接到 MySQL 和 MariaDB（一个 GPL 许可的分支）。

PostgreSQL 模块

PostgreSQL 是另一个流行的开源关系型数据库管理系统。它的一个广泛使用的 DBAPI 兼容接口是 [psycopg3](https://oreil.ly/pXc-t)，它是受欢迎的 [psycopg2](https://oreil.ly/gOTn7) 包的合理化重写和扩展。

## SQLite

[SQLite](http://www.sqlite.org) 是一个用 C 编写的库，实现了一个关系型数据库，可以存储在单个文件中，甚至在内存中用于足够小和短暂的情况。Python 的标准库提供了 sqlite3 包，它是与 SQLite 兼容的 DBAPI 接口。

SQLite 具有丰富的高级功能，有许多选项可供选择；sqlite3 提供了访问这些功能的大部分能力，同时提供更多可能性，使得 Python 代码与底层数据库之间的交互更加平稳和自然。我们无法在本书中涵盖这两个强大软件系统的每一个细节；相反，我们专注于最常用和最有用的函数子集。要获取更详细的信息，包括示例和最佳实践建议，请参阅 [SQLite](https://oreil.ly/-6LhJ) 和 [sqlite3](https://oreil.ly/S6VE1) 的文档，以及 Jay Kreibich 的 [*Using SQLite*](https://learning.oreilly.com/library/view/using-sqlite/9781449394592)（O’Reilly）。

sqlite3 包还提供了 表 12-12 中的函数，以及其他函数。

表 12-12\. sqlite3 模块的一些有用函数

| connect | connect(*filepath*, timeout=5.0, detect_types=0, isolation_level='', check_same_thread=**True**, factory=Connection, cached_statements=100, uri=**False**) 连接到名为 *filepath* 的 SQLite 数据库文件（如果需要则创建），并返回 Connection 类的实例（或传递的子类）。要创建内存中的数据库，请将 ':memory:' 作为第一个参数传递给 *filepath*。

如果 **True**，uri 参数激活 SQLite 的 [URI 功能](https://oreil.ly/S2h8r)，允许通过 *filepath* 参数一起传递一些额外的选项。

timeout 是在事务中有另一个连接锁定数据库时等待抛出异常之前的秒数。

sqlite3 直接支持以下 SQLite 原生类型，将其转换为相应的 Python 类型：

+   BLOB：转换为字节

+   INTEGER：转换为整数

+   NULL：转换为 **None**

+   REAL：转换为浮点数

+   TEXT：取决于 Connection 实例的 text_factory 属性，在 表 12-13 中有所涉及；默认为 str

除此以外的任何类型名称被视为 TEXT，除非经过适当的检测并通过函数 register_converter 注册的转换器传递。为了允许类型名称检测，可以传递 detect_types 参数，该参数可以是 sqlite3 包提供的 PARSE_COLNAMES 或 PARSE_DECLTYPES 常量（或两者都使用，通过 &#124; 位 OR 运算符连接）。

当你传递 detect_types=sqlite3.PARSE_COLNAMES 时，类型名称取自于 SQL SELECT 语句中检索列的列名；例如，检索为 *foo* AS [*foo* CHAR(10)] 的列具有类型名称 CHAR。

当你传递 detect_types=sqlite3.PARSE_DECLTYPES 时，类型名称取自于原始 CREATE TABLE 或 ALTER TABLE SQL 语句中添加列的声明；例如，声明为 *foo* CHAR(10) 的列具有类型名称 CHAR。

当你传递 detect_types=sqlite3.PARSE_COLNAMES &#124; sqlite3.PARSE_DECLTYPES 时，两种机制都会被使用，优先使用列名（当列名至少有两个单词时，第二个单词在这种情况下给出类型名），如果没有则回退到声明时给定的类型（在这种情况下，声明类型的第一个单词给出类型名）。

isolation_level 允许你在 SQLite 处理事务时行使一些控制；它可以是 ''（默认值）、**None**（使用 *autocommit* 模式）或三个字符串之一：'DEFERRED'、'EXCLUSIVE' 或 'IMMEDIATE'。[SQLite 在线文档](https://oreil.ly/IuKIz)详细介绍了 [事务类型](https://oreil.ly/AnFtn) 及其与 SQLite 固有执行的各种 [文件锁定](https://oreil.ly/cpWkt) 的关系。

| 连接 *(续)* | 默认情况下，连接对象只能在创建它的 Python 线程中使用，以避免因程序中的轻微错误（多线程编程中常见的不幸问题）而导致数据库损坏。如果你对线程在使用锁和其他同步机制方面完全自信，并且需要在多个线程之间重用连接对象，你可以传递 check_same_thread=**False**。sqlite3 将不再执行任何检查，相信你知道自己在做什么，并且你的多线程架构没有任何缺陷——祝你好运！cached_statements 是 sqlite3 缓存的 SQL 语句数量，以解析和准备的状态保存，以避免重复解析它们所带来的开销。你可以传递一个低于默认值 100 的值以节省一些内存，或者如果你的应用程序使用了多种多样的 SQL 语句，可以传递一个更大的值。 |
| --- | --- |
| r⁠e⁠g⁠i⁠s⁠t⁠e⁠r⁠_​a⁠d⁠a⁠p⁠t⁠e⁠r | register_adapter(*type*, *callable*) 将 *callable* 注册为从任何 Python 类型 *type* 的对象到 sqlite3 直接处理的几种 Python 类型之一（int、float、str 和 bytes）的相应值的 *adapter*。*callable* 必须接受一个参数，即要适配的值，并返回一个 sqlite3 直接处理的类型的值。 |
| r⁠e⁠g⁠i⁠s⁠t⁠e⁠r⁠_​c⁠o⁠n⁠v⁠e⁠r⁠t⁠e⁠r | register_converter(*typename*, *callable*) 将 *callable* 注册为从 SQL 中标识为某种 *typename* 类型的值（请参阅 connect 函数的 detect_types 参数的描述，了解类型名是如何确定的）到相应 Python 对象的 *converter*。*callable* 必须接受一个参数，即从 SQL 获取的值的字符串形式，并返回相应的 Python 对象。*typename* 的匹配区分大小写。 |

另外，sqlite3 还提供了 Connection、Cursor 和 Row 类。每个类都可以进一步定制为子类；但这是一个我们在本书中不再详细讨论的高级主题。Cursor 类是标准的 DBAPI 游标类，除了一个额外的便利方法 executescript，接受一个参数，即由多条语句以 ; 分隔的字符串（无参数）。其他两个类将在接下来的章节中介绍。

### sqlite3.Connection 类

除了所有符合 DBAPI 标准模块 Connection 类通用的方法外，详情请参见 “Connection Objects”，sqlite3.Connection 还提供了 Table 12-13 中的方法和属性。

Table 12-13\. sqlite3.Connection 类的附加方法和属性

| crea⁠t⁠e⁠_​a⁠g⁠g⁠r⁠e⁠g⁠a⁠t⁠e | create_aggregate(*name*, *num_params*, *aggregate_class*) *aggregate_class* 必须是一个类，提供两个实例方法：step，接受确切 *num_param* 个参数；finalize，不接受参数，并返回聚合的最终结果，即 sqlite3 原生支持的类型的值。这个聚合函数可以通过给定的 *name* 在 SQL 语句中使用。 |
| --- | --- |
| c⁠r⁠e⁠a⁠t⁠e⁠_​c⁠o⁠l⁠l⁠a⁠t⁠i⁠o⁠n | crea⁠t⁠e⁠_​c⁠o⁠l⁠l⁠a⁠t⁠i⁠o⁠n(*name*, *callable*) *callable* 必须接受两个字节字符串参数（编码为 'utf-8'），如果第一个参数应被视为“小于”第二个参数，则返回 -1；如果应被视为“大于”，则返回 1；如果应被视为“等于”，则返回 0。此类排序规则可以在 SQL SELECT 语句的 ORDER BY 子句中以 *name* 命名使用。 |
| crea⁠t⁠e⁠_​f⁠u⁠n⁠c⁠t⁠i⁠o⁠n | create_function(*name*, *num_params*, *func*) *func* 必须接受确切 *num_params* 个参数，并返回 sqlite3 原生支持的类型的值；这样的用户定义函数可以通过给定的 *name* 在 SQL 语句中使用。 |
| interrupt | interrupt() 可以从任何其他线程调用，以中止在此连接上执行的所有查询（在使用连接的线程中引发异常）。 |
| isolati⁠o⁠n⁠_​l⁠e⁠v⁠e⁠l | 一个只读属性，其值为连接函数的 isolation_level 参数提供的值。 |
| iterdump | iterdump() 返回一个迭代器，生成字符串：构建当前数据库的 SQL 语句，包括模式和内容。例如，可用于将内存中的数据库持久化到磁盘以供将来重用。 |
| row_factory | 一个可调用对象，接受游标和原始行作为元组，并返回用作真实结果行的对象。一个常见的惯用法是 *x*.row_factory=sqlite3.Row，使用在下一节详细介绍的高度优化的 Row 类，提供基于索引和不区分大小写的名称访问列，几乎没有额外开销。 |
| text_factory | 一个接受单一字节字符串参数并返回用于 TEXT 列值的对象的可调用函数——默认为 str，但你可以设置为任何类似的可调用函数。 |
| total_changes | 自连接创建以来已修改、插入或删除的总行数。 |

连接对象还可以作为上下文管理器使用，以自动提交数据库更新或在发生异常时回滚；然而，在这种情况下，你需要显式调用 Connection.close() 来关闭连接。

### sqlite3.Row 类

sqlite3 还提供了 Row 类。Row 对象大部分类似于元组，但还提供了 keys 方法，返回列名列表，并支持通过列名而不是列编号进行索引。

### 一个 sqlite3 示例

下面的示例处理与本章早些时候展示的示例相同的任务，但使用 sqlite3 进行持久化，而不是在内存中创建索引：

```py
`import` fileinput, sqlite3
connect = sqlite3.connect('database.db')
cursor = connect.cursor()
`with` connect:
    cursor.execute('CREATE TABLE IF NOT EXISTS Words '
                   '(Word TEXT, File TEXT, Line INT)')
 `for` line `in` fileinput.input():
        f, l = fileinput.filename(), fileinput.filelineno()
        cursor.executemany('INSERT INTO Words VALUES (:w, :f, :l)',
            [{'w':w, 'f':f, 'l':l} `for` w `in` line.split()])
connect.close()
```

然后我们可以使用 sqlite3 读取存储在 DB 文件 *database.db* 中的数据，如下例所示：

```py
`import` sys, sqlite3, linecache
connect = sqlite3.connect('database.db')
cursor = connect.cursor()
`for` word `in` sys.argv[1:]:
    cursor.execute('SELECT File, Line FROM Words '
                   'WHERE Word=?', [word])
    places = cursor.fetchall()
    `if` `not` places:
         print(f'Word {word!r} not found in index file',
               file=sys.stderr)
 `continue`
    `for` fname, lineno `in` places:
        print(f'Word {word!r} occurs in line {lineno}'
              f' of file {fname!r}:')
        print(linecache.getline(fname, lineno), end='')
connect.close()
```

¹ 实际上，“CSV” 有点名不副实，因为一些方言使用制表符或其他字符作为字段分隔符，而不是逗号。更容易将它们视为“分隔符分隔的值”。

² 如果你需要在此及其他方面扩展 pickle，请考虑第三方包 [dill](https://oreil.ly/mU15t)。

³ dbm 的键和值必须是字节；shelve 将接受字节或 str，并且会透明地对字符串进行编码。
