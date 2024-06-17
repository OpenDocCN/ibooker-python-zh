# 第二十二章 结构化文本：HTML

网络上的大多数文档使用 HTML，即超文本标记语言。 *标记* 是在文本文档中插入特殊标记（称为 *标签* ）以结构化文本。HTML 理论上是一种大而普遍的标准应用，称为 [标准通用标记语言（SGML）](https://oreil.ly/X-3xi)。然而，在实践中，许多网络文档以松散或不正确的方式使用 HTML。

HTML 设计用于在浏览器中呈现文档。随着网络内容的发展，用户意识到它缺乏 *语义标记* 的能力，其中标记指示划分文本的意义而不仅仅是其外观。完全、精确地提取 HTML 文档中的信息通常是不可行的。一个更严格的标准称为 XHTML 试图弥补这些缺点。XHTML 类似于传统的 HTML，但是它是以 XML（可扩展标记语言）的术语来定义的，比 HTML 更加精确。您可以使用 第二十三章 中涵盖的工具处理格式良好的 XHTML。然而，截至本文撰写时，XHTML 并未取得压倒性成功，而是被更为实用的 HTML5 所取代。

尽管存在困难，通常可以从 HTML 文档中提取至少一些有用信息（称为 *网页抓取*、*蜘蛛行动* 或仅为 *抓取* 的任务）。Python 标准库尝试帮助您，提供了用于解析 HTML 文档的 html 包，无论是为了呈现文档还是更典型地作为尝试从中提取信息的一部分。然而，当处理有些不完整的网页（这几乎总是情况！）时，第三方模块 [BeautifulSoup](https://oreil.ly/9-cUQ) 通常是您最后的、最好的希望。在本书中，出于实际原因，我们主要涵盖 BeautifulSoup，忽略与其竞争的标准库模块。寻求替代方案的读者也应该调查越来越流行的 [scrapy 包](https://scrapy.org)。

生成 HTML 和在 HTML 中嵌入 Python 也是相当频繁的任务。标准的 Python 库不支持 HTML 生成或嵌入，但可以使用 Python 字符串格式化，并且第三方模块也可以提供帮助。BeautifulSoup 允许您修改 HTML 树（因此，特别是可以程序化地构建一个，甚至“从头”开始）；一个常见的替代方法是 *模板化*，例如由第三方模块 [jinja2](http://jinja.pocoo.org) 支持，我们在 “jinja2 包” 中提供了基本内容。

# html.entities 模块

Python 标准库中的 html.entities 模块提供了几个属性，全部都是映射关系（参见 表 22-1）。无论你用什么一般方法解析、编辑或生成 HTML，包括下一节介绍的 BeautifulSoup 包，这些属性都很有用。

表 22-1\. html.entities 的属性

| codepoi⁠n⁠t⁠2​n⁠a⁠m⁠e | 将 Unicode 代码点映射到 HTML 实体名称。例如，entities.codepoint2name[228] 是 'auml'，因为 Unicode 字符 228，ä，“带分音符的小写 a”，在 HTML 中编码为 '&auml;'。 |
| --- | --- |
| entitydefs | 将 HTML 实体名称映射到相应的 Unicode 等效单字符字符串。例如，entities.entitydefs['auml'] 是 'ä'，而 entities.entitydefs['sigma'] 是 'σ'。 |
| html5 | 将 HTML5 命名字符引用映射到等效的单字符字符串。例如，entities.xhtml5['gt;'] 是 '>'。键中的尾部分号 *确实* 重要 —— 少数但远非所有 HTML5 命名字符引用可以选择性地省略尾部分号，在这些情况下，entities.xhtml5 中会同时存在带有和不带有尾部分号的键。 |
| name2codepoint | 将 HTML 实体名称映射到 Unicode 代码点。例如，entities.name2codepoint['auml'] 是 228。 |

# BeautifulSoup 第三方包

[BeautifulSoup](https://oreil.ly/xx57e) 允许你解析 HTML，即使它的格式相当糟糕。它使用简单的启发式方法来弥补典型的 HTML 损坏，并且在大多数情况下成功地完成这一艰巨任务。当前的 BeautifulSoup 主要版本是版本 4，也称为 bs4。在本书中，我们特别涵盖了版本 4.10；截至撰写本文时，这是 bs4 的最新稳定版本。

# 安装与导入 BeautifulSoup

BeautifulSoup 是那些包装要求你在 Python 内外使用不同名称的烦人模块之一。你可以通过在 shell 命令提示符下运行 **pip install beautifulsoup4** 来安装该模块，但在 Python 代码中导入时，你使用 **import** bs4。

## BeautifulSoup 类

bs4 模块提供了 BeautifulSoup 类，通过调用它并传入一个或两个参数来实例化：第一个是 htmltext —— 可以是类似文件的对象（读取其中的 HTML 文本以解析），或者是字符串（作为要解析的文本）—— 第二个是可选的解析器参数。

### BeautifulSoup 使用的解析器

如果您没有传递解析器参数，BeautifulSoup 会“嗅探周围”以选择最佳解析器（但在这种情况下可能会收到 GuessedAtParserWarning 警告）。 如果您没有安装其他解析器，则 BeautifulSoup 将默认使用 Python 标准库中的 html.parser； 如果您已安装其他解析器，则 BeautifulSoup 将默认使用其中之一（目前首选的是 lxml）。 除非另有说明，否则以下示例使用默认的 Python html.parser。 为了获得更多控制并避免 BeautifulSoup 文档中提到的解析器之间的差异，请在实例化 BeautifulSoup 时将要使用的解析器库的名称作为第二个参数传递。¹

例如，如果你已经安装了第三方包 html5lib（以与所有主流浏览器相同的方式解析 HTML，尽管速度较慢），你可以调用：

```py
soup = bs4.BeautifulSoup(thedoc, 'html5lib')
```

当您将'xml'作为第二个参数传递时，您必须已经安装了第三方包 lxml。 BeautifulSoup 然后将文档解析为 XML，而不是 HTML。 在这种情况下，soup 的属性 is_xml 为**True**； 否则，soup.is_xml 为**False**。 如果您将'xml'作为第二个参数传递，也可以使用 lxml 解析 HTML。 更一般地说，您可能需要根据传递给 bs4.BeautifulSoup 调用的第二个参数选择安装适当的解析器库； 如果您没有这样做，BeautifulSoup 会通过警告消息提醒您。

这是在同一字符串上使用不同解析器的示例：

```py
>>> `import` bs4, lxml, html5lib
>>> sh = bs4.BeautifulSoup('<p>hello', 'html.parser')
>>> sx = bs4.BeautifulSoup('<p>hello', 'xml')
>>> sl = bs4.BeautifulSoup('<p>hello', 'lxml')
>>> s5 = bs4.BeautifulSoup('<p>hello', 'html5lib')
>>> `for` s `in` [sh, sx, sl, s5]:
...   print(s, s.is_xml)
...
```

```py
<p>hello</p> False
<?xml version="1.0" encoding="utf-8"?>
<p>hello</p> True
<html><body><p>hello</p></body></html> False
<html><head></head><body><p>hello</p></body></html> False
```

# 修复无效 HTML 输入中解析器之间的差异

在上面的示例中，'html.parser'仅插入了输入中缺失的结束标记</p>。 其他解析器在通过添加所需标记修复无效的 HTML 输入方面有所不同，例如<html>、<head>和<body>，您可以在示例中看到。

### BeautifulSoup、Unicode 和编码

BeautifulSoup 使用 Unicode，根据输入是否为字节串或二进制文件来推断或猜测编码²。 对于输出，prettify 方法返回树的 str 表示，包括标签及其属性。 prettify 使用空格和换行符添加到元素中以缩进元素，显示嵌套结构。 为了使其返回给定编码的 bytes 对象（字节串），请将编码名称作为参数传递给它。 如果您不想结果“漂亮”，请使用 encode 方法获取字节串，并使用 decode 方法获取 Unicode 字符串。 例如：

```py
>>> s = bs4.BeautifulSoup('<p>hello', 'html.parser')
>>> print(s.prettify())
```

```py
<p>
 hello
</p>
```

```py
>>> print(s.decode())
```

```py
<p>hello</p>
```

```py
>>> print(s.encode())
```

```py
b'<p>hello</p>'
```

## bs4 的可导航类

类 BeautifulSoup 的实例*b*提供了“导航”解析 HTML 树的属性和方法，返回*navigable classes* Tag 和 NavigableString 的实例，以及 NavigableString 的子类（CData、Comment、Declaration、Doctype 和 ProcessingInstruction，仅在输出时的不同）。 。

每个可导航类的实例都可以让您继续导航——即几乎使用与*b*本身相同的一组导航属性和搜索方法获取更多信息。存在一些差异：Tag 的实例可以在 HTML 树中具有 HTML 属性和子节点，而 NavigableString 的实例不能（NavigableString 的实例始终具有一个文本字符串，一个父 Tag 和零个或多个同级，即同一父标记的其他子节点）。

# 可导航类术语

当我们说“NavigableString 的实例”时，我们包括其任何子类的实例；当我们说“Tag 的实例”时，我们包括 BeautifulSoup 的实例，因为后者是 Tag 的子类。可导航类的实例也称为树的*元素*或*节点*。

所有可导航类的实例都有属性名称：对于 Tag 实例，它是标签字符串，对于 BeautifulSoup 实例，它是'[document]'，对于 NavigableString 实例，它是**None**。

Tag 的实例允许您通过索引访问它们的 HTML 属性，或者您可以通过实例的.attrs 属性将它们全部作为字典获取。

### 索引 Tag 的实例

当*t*是 Tag 的实例时，*t*['foo']会查找*t*的 HTML 属性中名为 foo 的属性，并返回 foo 属性的字符串。当*t*没有名为 foo 的 HTML 属性时，*t*['foo']会引发 KeyError 异常；类似于字典上的操作，可以调用*t*.get('foo', default=**None**)来获取默认参数值，而不是异常。

一些属性，如 class，在 HTML 标准中被定义为可以具有多个值（例如，<body class="foo bar">...</body>）。在这些情况下，索引返回一个值列表，例如 soup.body['class']将是['foo', 'bar']（如果属性不存在，再次，您将得到一个 KeyError 异常；使用 get 方法而不是索引来获取默认值）。

要获得将属性名称映射到值（或在 HTML 标准中定义的少数情况下，值列表）的字典，请使用属性*t*.attrs：

```py
>>> s = bs4.BeautifulSoup('<p foo="bar" class="ic">baz')
>>> s.get('foo')
>>> s.p.get('foo')
```

```py
'bar'
```

```py
>>> s.p.attrs
```

```py
{'foo': 'bar', 'class': ['ic']}
```

# 如何检查 Tag 实例是否具有某个特定属性

要检查 Tag 实例*t*的 HTML 属性是否包含名为'foo'的属性，请*不要*使用 if 'foo' in *t*：——在 Tag 实例上的 in 运算符会在 Tag 的*子级*中查找，而*不是*在其*属性*中查找。而是，请使用 if 'foo' in *t*.attrs：或者更好地，使用 if *t*.has_attr('foo')：。

### 获取实际字符串

当您有一个 NavigableString 的实例时，通常希望访问它包含的实际文本字符串。当您有一个 Tag 的实例时，您可能希望访问它包含的唯一字符串，或者如果包含多个字符串，则希望访问所有这些字符串，也许还带有其周围任何空格的文本剥离。以下是您可以完成这些任务的最佳方法。

当你有一个 NavigableString 实例 *s* 并且需要将其文本存储或处理在其他地方，而不需要进一步对其进行导航时，请调用 str(*s*)。或者，使用 *s*.encode(codec='utf8') 得到一个字节串，或者 *s*.decode() 得到一个文本字符串（即 Unicode）。这些方法给出了实际的字符串，而不包含对 BeautifulSoup 树的引用，这会妨碍垃圾回收（*s* 支持 Unicode 字符串的所有方法，因此如果这些方法满足你的需求，可以直接调用它们）。

给定一个包含单个 NavigableString 实例 *s* 的 Tag 实例 *t*，你可以使用 *t*.string 来获取 *s*（或者，如果只想从 *s* 获取你想要的文本，可以使用 *t*.string.decode()）。当 *t* 有一个单一子项是 NavigableString，或者有一个单一子项是 Tag，其唯一子项是 NavigableString 时，*t*.string 才有效；否则，*t*.string 为 **None**。

作为所有包含的（可导航的）字符串的迭代器，使用 *t*.strings。你可以使用 ''.join(*t*.strings) 将所有字符串连接成一个单独的字符串，一次完成。要忽略每个包含字符串周围的空白，请使用迭代器 *t*.stripped_strings（它还会跳过所有空白字符串）。

或者，调用 *t*.get_text()：这将返回一个单一的（Unicode）字符串，其中包含 *t* 的后代中所有的文本，按照树的顺序（等同于访问属性 *t*.text）。你可以选择传递一个字符串作为分隔符的唯一位置参数。默认为空字符串，''。传递命名参数 strip=**True** 可以使每个字符串去除周围的空白，并跳过所有空白字符串。

以下示例演示了从标签内获取字符串的这些方法：

```py
>>> soup = bs4.BeautifulSoup('<p>Plain <b>bold</b></p>')
>>> print(soup.p.string)
```

```py
None
```

```py
>>> print(soup.p.b.string)
```

```py
bold
```

```py
>>> print(''.join(soup.strings))
```

```py
Plain bold
```

```py
>>> print(soup.get_text())
```

```py
Plain bold
```

```py
>>> print(soup.text)
```

```py
Plain bold
```

```py
>>> print(soup.get_text(strip=True))
```

```py
Plainbold
```

### BeautifulSoup 和 Tag 的实例上的属性引用。

在 bs4 中，导航 HTML 树或子树的最简单、最优雅的方法是使用 Python 的属性引用语法（只要你命名的每个标签都是唯一的，或者你只关心每个下降级别的第一个命名标签）。

给定 Tag 的任何实例 *t*，类似 *t*.foo.bar 的结构会查找 *t* 的后代中的第一个 foo 标签，并获取它的 Tag 实例 *ti*，然后查找 *ti* 的后代中的第一个 bar 标签，并返回 bar 标签的 Tag 实例。

当你知道在可导航实例的后代中有某个标签的单一出现，或者当你只关心第一个出现的几个时，这是一种简洁而优雅的导航树的方式。但要注意：如果任何查找层级找不到正在寻找的标签，则属性引用的值为 **None**，然后任何进一步的属性引用都会引发 AttributeError。

# 警惕标签实例属性引用中的拼写错误。

由于这种 BeautifulSoup 的行为，如果在 Tag 实例的属性引用中存在任何拼写错误，将会得到 **None** 的值，而不是 AttributeError 异常——因此，请特别小心！

bs4 还提供了更一般的方法沿树向下、向上和侧向导航。特别地，每个可导航类实例都有属性，用于标识单个“相对”的或者复数形式下的所有相同类的迭代器。

### contents、children 和 descendants

给定 Tag 的实例*t*，您可以获取其所有子节点的列表作为*t*.contents，或者作为所有子节点的迭代器的*t*.children。要获取所有*descendants*（子节点、子节点的子节点等），请使用*t*.descendants 的迭代器：

```py
>>> soup = bs4.BeautifulSoup('<p>Plain <b>bold</b></p>')
>>> list(t.name `for` t `in` soup.p.children)
```

```py
[None, 'b']
```

```py
>>> list(t.name `for` t `in` soup.p.descendants)
```

```py
[None, 'b', None]
```

为**None**的名称对应于 NavigableString 节点；它们中的第一个是 p 标签的*child*，但两者都是该标签的*descendants*。

### parent 和 parents

给定任何可导航类的实例*n*，其父节点是*n*.parent：

```py
>>> soup = bs4.BeautifulSoup('<p>Plain <b>bold</b></p>')
>>> soup.b.parent.name
```

```py
'p'
```

一个在树中向上迭代所有祖先节点的迭代器是*n*.parents。这也包括 NavigableString 的实例，因为它们也有父节点。BeautifulSoup 的实例*b*的*b*.parent 是**None**，并且*b*.parents 是一个空迭代器。

### next_sibling、previous_sibling、next_siblings 和 previous_siblings

给定任何可导航类的实例*n*，其紧邻左侧的兄弟节点是*n*.previous_sibling，紧邻右侧的兄弟节点是*n*.next_sibling；如果*n*没有这样的兄弟节点，则可以是**None**。在树中向左迭代所有左侧兄弟节点的迭代器是*n*.previous_siblings；在树中向右迭代所有右侧兄弟节点的迭代器是*n*.next_siblings（这两个迭代器都可能为空）。这也包括 NavigableString 的实例，因为它们也有兄弟节点。对于 BeautifulSoup 的实例*b*，*b*.previous_sibling 和*b*.next_sibling 都是**None**，其兄弟节点迭代器都是空的：

```py
>>> soup = bs4.BeautifulSoup('<p>Plain <b>bold</b></p>')
>>> soup.b.previous_sibling, soup.b.next_sibling
```

```py
('Plain ', None)
```

### next_element、previous_element、next_elements 和 previous_elements

给定任何可导航类的实例*n*，其解析的前一个节点是*n*.previous_element，解析的后一个节点是*n*.next_element；当*n*是第一个或最后一个解析的节点时，其中一个或两者可以是**None**。在树中向后迭代所有先前元素的迭代器是*n*.previous_elements；在树中向前迭代所有后续元素的迭代器是*n*.next_elements（这两个迭代器都可能为空）。NavigableString 的实例也具有这些属性。对于 BeautifulSoup 的实例*b*，*b*.previous_element 和*b*.next_element 都是**None**，其元素迭代器都是空的：

```py
>>> soup = bs4.BeautifulSoup('<p>Plain <b>bold</b></p>')
>>> soup.b.previous_element, soup.b.next_element
```

```py
('Plain ', 'bold')
```

如前例所示，b 标签没有 next_sibling（因为它是其父节点的最后一个子节点）；但是，它确实有一个 next_element（紧随其后解析的节点，在本例中是其包含的'bold'字符串）。

## bs4 的 find…方法（又称搜索方法）

每个可导航类在 bs4 中提供了几种方法，这些方法的名称以 find 开头，称为*搜索方法*，用于定位满足指定条件的树节点。

搜索方法成对出现——每对方法中的一个方法遍历树的所有相关部分并返回满足条件的节点列表，而另一个方法在找到满足所有条件的单个节点时停止并返回它（或在找不到这样的节点时返回 **None**）。因此，调用后者的方法就像调用前者的方法，并使用参数 limit=1，然后索引结果为单项目列表以获得其单个项目，但更快更优雅。

因此，例如，对于任何标签实例 *t* 和由 ... 表示的任何位置参数和命名参数组，以下等价性总是成立：

```py
just_one = *`t`*.find(...)
other_way_list = *`t`*.find_all(..., limit=1)
other_way = other_way_list[0] `if` other_way_list `else` `None`
`assert` just_one == other_way
```

方法对列在 表 22-2 中。

表格 22-2\. bs4 find... 方法对

| find, find_all | *b*.find(...)，*b*.find_all(...)

搜索 *b* 的 *descendants* 或者当你传递命名参数 recursive=**False**（仅适用于这两种方法，而不适用于其他搜索方法）时，仅限于 *b* 的 *children*。这些方法在 NavigableString 实例上不可用，因为它们没有后代；所有其他搜索方法在标签和 NavigableString 实例上都可用。

由于经常需要 find_all，bs4 提供了一个优雅的快捷方式：调用一个标签就像调用它的 find_all 方法一样。换句话说，当 *b* 是一个标签时，*b*(...) 等同于 *b*.find_all(...)。

另一个快捷方式，已在 “BeautifulSoup 和 Tag 实例上的属性引用” 中提到，即 *b*.foo.bar 等同于 *b*.find('foo').find('bar')。

| find_next, find_all_next | *b*.find_next(...)，*b*.find_all_next(...)

搜索 *b* 的下一个元素。

| find_next_sibling, find_next_siblings | *b*.find_next_sibling(...)，*b*.find_next_siblings(...)

搜索 *b* 的下一个兄弟。

| find_parent, find_parents | *b*.find_parent(...)，*b*.find_parents(...)

搜索 *b* 的父元素。

| find_previous, find_all_previous | *b*.find_previous(...)，*b*.find_all_previous(...)

搜索 *b* 的前一个元素。

| find_previous_sibling, find_previous_siblings | *b*.find_previous_sibling(...)，*b*.find_previous_siblings(...) 搜索 *b* 的前一个兄弟。 |
| --- | --- |

### 搜索方法的参数

每个搜索方法都有三个可选参数：*name*、*attrs* 和 *string*。*name* 和 *string* 是 *filters*，如下一小节所述；*attrs* 是一个字典，如本节后面所述。另外，如 表 22-2 中所述，仅 find 和 find_all（而不是其他搜索方法）可以选择使用命名参数 recursive=**False** 进行调用，以限制搜索范围仅限于子代，而不是所有后代。

返回列表的任何搜索方法（即其名称为复数或以 find_all 开头）可以选择接受命名参数 limit：其值（如果有）为整数，将返回的列表长度上限化（当您传递 limit 时，如有必要，返回的列表结果将被截断）。

在这些可选参数之后，每个搜索方法可以选择具有任意数量的任意命名参数：参数名称可以是任何标识符（除了搜索方法的特定参数名称），而值是筛选器。

#### 筛选器

*filter*应用于*target*，*target*可以是标签的名称（当作为*name*参数传递时）、Tag 的字符串或 NavigableString 的文本内容（当作为*string*参数传递时）、或 Tag 的属性（当作为命名参数的值传递或在*attrs*参数中）。 每个筛选器可以是：

Unicode 字符串

筛选器成功时，字符串完全等于目标。

字节字符串

使用 utf-8 解码为 Unicode，当生成的 Unicode 字符串完全等于目标时，筛选器成功。

正则表达式对象（由 re.compile 生成，详见“正则表达式和 re 模块”）

当 RE 的搜索方法以目标作为参数调用成功时，筛选器成功。

字符串列表

如果任何字符串完全等于目标（如果任何字符串为字节字符串，则使用 utf-8 解码为 Unicode）则筛选器成功。

函数对象

当使用 Tag 或 NavigableString 实例作为参数调用函数时返回 True 时，筛选器成功。

**True**

筛选器总是成功。

作为“筛选器成功”的同义词，我们也说“目标与筛选器匹配”。

每个搜索方法都会查找所有与其所有筛选器匹配的相关节点（即，在每个候选节点上隐式执行逻辑**and**操作）。 （不要将此逻辑与具有列表作为参数值的特定筛选器的逻辑混淆。其中一个筛选器匹配列表中的任何项时，该筛选器隐式执行逻辑**or**操作。）

#### 名称

要查找名称匹配筛选器的标签，请将筛选器作为搜索方法的第一个位置参数传递，或将其作为 name=*filter*传递：

```py
*`# return all instances of Tag 'b' in the document`*
soup.find_all('b') *`# or soup.find_all(name='b')`*

*`# return all instances of Tags 'b' and 'bah' in the document`*
soup.find_all(['b', 'bah'])

*`# return all instances of Tags starting with 'b' in the document`*
soup.find_all(re.compile(r'^b'))

*`# return all instances of Tags including string 'bah' in the document`*
soup.find_all(re.compile(r'bah'))

*`# return all instances of Tags whose parent's name is 'foo'`*
`def` child_of_foo(tag):
    `return` tag.parent.name == 'foo'

soup.find_all(child_of_foo)
```

#### 字符串

要查找其.string 文本与筛选器匹配的 Tag 节点，或者文本与筛选器匹配的 NavigableString 节点，请将筛选器作为字符串=*filter*传递：

```py
*`# return all instances of NavigableString whose text is 'foo'`*
soup.find_all(string='foo')

*`# return all instances of Tag 'b' whose .string's text is 'foo'`*
soup.find_all('b', string='foo')
```

#### 属性

要查找具有值匹配筛选器的属性的 Tag 节点，请使用将属性名称作为键和相应筛选器作为相应值的字典*d*。 然后，将*d*作为搜索方法的第二个位置参数传递，或将 attrs=*d*传递。

作为特例，您可以使用*d*中的值**None**而不是筛选器；这将匹配缺少相应属性的节点。

作为单独的特殊情况，如果 attrs 的值 *f* 不是字典，而是过滤器，则相当于具有 `attrs={'class': *f*}`。 （此便捷快捷方式非常有用，因为查找具有特定 CSS 类的标签是频繁的任务。）

你不能同时应用这两种特殊情况：要搜索没有任何 CSS 类的标签，必须显式地传递 `attrs={'class': **None**}`（即使用第一个特殊情况，但不能同时使用第二个）：

```py
*`# return all instances of Tag 'b' w/an attribute 'foo' and no 'bar'`*
soup.find_all('b', {'foo': `True`, 'bar': `None`})
```

# 匹配具有多个 CSS 类的标签

与大多数属性不同，标签的 `'class'` 属性可以具有多个值。这些在 HTML 中显示为以空格分隔的字符串（例如，`'<p class='foo bar baz'>...'`），在 bs4 中作为字符串列表显示（例如，*t*['class'] 为 `['foo', 'bar', 'baz']`）。

在任何搜索方法中按 CSS 类过滤时，如果标签的多个 CSS 类中有一个匹配，过滤器将匹配该标签。

要通过多个 CSS 类匹配标签，可以编写自定义函数并将其作为过滤器传递给搜索方法；或者，如果不需要搜索方法的其他增加功能，则可以避免搜索方法，而是使用后续部分中介绍的 `*t*.select` 方法，并按 CSS 选择器的语法进行操作。

#### 其他命名参数

命名参数，超出搜索方法已知名称的参数，用于增强已指定的 attrs 约束（如果有）。例如，调用搜索方法带有 `*foo*=*bar*` 相当于带有 `attrs={'*foo*': *bar*}`。

## bs4 CSS 选择器

bs4 标签提供 `select` 和 `select_one` 方法，大致相当于 `find_all` 和 `find`，但接受一个字符串作为参数，该字符串是 [CSS 选择器](https://oreil.ly/8bNZk)，分别返回满足该选择器的 Tag 节点列表或第一个这样的 Tag 节点。例如：

```py
`def` foo_child_of_bar(t):
    `return` t.name=='foo' `and` t.parent `and` t.parent.name=='bar'

*`# return tags with name 'foo' children of tags with name 'bar'`*
soup.find_all(foo_child_of_bar)

*`# equivalent to using find_all(), with no custom filter function needed`*
soup.select('bar > foo')
```

bs4 仅支持丰富的 CSS 选择器功能的子集，在本书中不再详细介绍 CSS 选择器。（要完整了解 CSS，建议阅读 O’Reilly 的 [*CSS: The Definitive Guide*](https://www.oreilly.com/library/view/css-the-definitive/9781449325053/)，作者是 Eric Meyer 和 Estelle Weyl。）在大多数情况下，前一节中介绍的搜索方法是更好的选择；然而，在一些特殊情况下，调用 `select` 可以避免编写自定义过滤函数（稍微麻烦的小事）。

## 使用 BeautifulSoup 进行 HTML 解析的示例

以下示例使用 bs4 执行典型任务：从 Web 获取页面、解析页面并输出页面中的 HTTP 超链接：

```py
`import` urllib.request, urllib.parse, bs4

f = urllib.request.urlopen('http://www.python.org')
b = bs4.BeautifulSoup(f)

seen = set()
`for` anchor `in` b('a'):
    url = anchor.get('href')
    `if` url `is` `None` `or` url `in` seen:
        `continue`
    seen.add(url)
    pieces = urllib.parse.urlparse(url)
    `if` pieces[0].startswith('http'):
        print(urllib.parse.urlunparse(pieces))
```

首先调用类 `bs4.BeautifulSoup` 的实例（等同于调用其 `find_all` 方法），以获取特定标签（这里是 `<a>` 标签）的所有实例，然后再获取该标签实例的 `get` 方法来获取属性的值（这里是 `'href'`），或者在该属性缺失时返回 **None**。

# 生成 HTML

Python 没有专门用于生成 HTML 的工具，也没有让你直接在 HTML 页面中嵌入 Python 代码的工具。通过*模板化*（在“模板化”中讨论），通过分离逻辑和表示问题来简化开发和维护。还可以使用 bs4 在 Python 代码中创建 HTML 文档，逐步修改非常简单的初始文档。由于这些修改依赖于 bs4 解析某些 HTML，因此使用不同的解析器会影响输出，如在“BeautifulSoup 使用哪个解析器”中提到的那样。

## 使用 bs4 编辑和创建 HTML

编辑 Tag 的实例*t*有各种选项。你可以通过赋值给*t*.name 改变标签名，通过将*t*视为映射来改变*t*的属性：赋值给索引以添加或更改属性，或删除索引以移除属性（例如，**del** *t*['foo']移除属性 foo）。如果你将一些字符串赋给*t*.string，那么所有先前的*t*.contents（标签和/或字符串—*t*的整个子树）都将被丢弃，并替换为具有该字符串作为其文本内容的新 NavigableString 实例。

给定 NavigableString 实例*s*，你可以替换其文本内容：调用*s*.replace_with('other')将*s*的文本替换为'other'。

### 构建和添加新节点

修改现有节点很重要，但从头开始构建 HTML 文档时创建新节点并将其添加到树中至关重要。

要创建一个新的 NavigableString 实例，请调用类并将文本内容作为唯一参数：

```py
s = bs4.NavigableString(' some text ')
```

要创建一个新的 Tag 实例，请调用 BeautifulSoup 实例的 new_tag 方法，将标签名作为唯一的位置参数，并（可选地）为属性命名参数。

```py
>>> soup = bs4.BeautifulSoup()
>>> t = soup.new_tag('foo', bar='baz')
>>> print(t)
```

```py
<foo bar="baz"></foo>
```

要将节点添加到 Tag 的子节点中，请使用 Tag 的 append 方法。这将在任何现有子节点之后添加节点：

```py
>>> t.append(s)
>>> print(t)
```

```py
<foo bar="baz"> some text </foo>
```

如果你希望新节点不是在结尾，而是在*t*的子节点中的某个索引处，请调用*t*.insert(*n, s*)将*s*放置在*t*.contents 的索引*n*处（*t*.append 和*t*.insert 的工作方式就像*t*是其子节点列表一样）。

如果你有一个可导航的元素*b*，想要将一个新节点*x*添加为*b*的 previous_sibling，请调用*b*.insert_before(*x*)。如果你希望*x*代替*b*的 next_sibling，请调用*b*.insert_after(*x*)。

如果你想将新的父节点*t*包裹在*b*周围，调用*b*.wrap(*t*)（这也返回新包裹的标签）。例如：

```py
>>> print(t.string.wrap(soup.new_tag('moo', zip='zaap')))
```

```py
<moo zip="zaap"> some text </moo>
```

```py
>>> print(t)
```

```py
<foo bar="baz"><moo zip="zaap"> some text </moo></foo>
```

### 替换和移除节点

你可以在任何标签*t*上调用*t*.replace_with：该调用将替换*t*及其先前的所有内容为参数，并返回具有其原始内容的*t*。例如：

```py
>>> soup = bs4.BeautifulSoup(
...        '<p>first <b>second</b> <i>third</i></p>', 'lxml')
>>> i = soup.i.replace_with('last')
>>> soup.b.append(i)
>>> print(soup)
```

```py
<html><body><p>first <b>second<i>third</i></b> last</p></body></html>
```

你可以在任何标签*t*上调用*t*.unwrap：该调用将替换*t*及其内容，并返回“清空”的*t*（即，没有内容）。例如：

```py
>>> empty_i = soup.i.unwrap()
>>> print(soup.b.wrap(empty_i))
```

```py
<i><b>secondthird</b></i>
```

```py
>>> print(soup)
```

```py
<html><body><p>first <i><b>secondthird</b></i> last</p></body></html>
```

*t*.clear 移除*t*的内容，销毁它们，并将*t*留空（但仍然位于树中的原始位置）。*t*.decompose 移除并销毁*t*本身及其内容：

```py
>>> *`# remove everything between <i> and </i> but leave tags`*	
>>> soup.i.clear()
>>> print(soup)
```

```py
<html><body><p>first <i></i> last</p></body></html>
```

```py
>>> *`# remove everything between <p> and </p> incl. tags`*
>>> soup.p.decompose()
>>> print(soup)
```

```py
<html><body></body></html>
```

```py
>>> *`# remove <body> and </body>`*
>>> soup.body.decompose()
>>> print(soup)
```

```py
<html></html>
```

最后，*t*.extract 提取并返回*t*及其内容，但不销毁任何内容。

## 使用 bs4 构建 HTML

下面是一个示例，展示了如何使用 bs4 的方法生成 HTML。具体来说，以下函数接受一个“行”（序列）的序列，并返回一个字符串，该字符串是一个 HTML 表格，用于显示它们的值：

```py
`def` mktable_with_bs4(seq_of_rows):
    tabsoup = bs4.BeautifulSoup('<table>')
    tab = tabsoup.table
    `for` row `in` seq_of_rows:
        tr = tabsoup.new_tag('tr')
        tab.append(tr)
        `for` item `in` row:
            td = tabsoup.new_tag('td')
            tr.append(td)
            td.string = str(item)
    `return` tab
```

这里是使用我们刚刚定义的函数的示例：

```py
>>> example = (
...     ('foo', 'g>h', 'g&h'),
...     ('zip', 'zap', 'zop'),
... )
>>> print(mktable_with_bs4(example))
```

```py
<table><tr><td>foo</td><td>g&gt;h</td><td>g&amp;h</td></tr>
<tr><td>zip</td><td>zap</td><td>zop</td></tr></table>
```

注意，bs4 会自动将标记字符如<, >和&转换为它们对应的 HTML 实体；例如，'g>h'呈现为'g&gt;h'。

## 模板化

要生成 HTML，通常最好的方法是*模板化*。您可以从一个*模板*开始——一个文本字符串（通常从文件、数据库等读取），它几乎是有效的 HTML，但包含*占位符*（称为*占位符*），在动态生成的文本必须插入的位置；您的程序生成所需的文本并将其替换到模板中。

在最简单的情况下，您可以使用形式为{*name*}的标记。将动态生成的文本设置为某个字典*d*中键'*name*'的值。Python 的字符串格式化方法.format（在“字符串格式化”中讨论）让您完成剩下的工作：当*t*是模板字符串时，*t*.format(*d*)是模板的副本，所有值都得到了正确的替换。

一般来说，除了替换占位符之外，您还会希望使用条件语句，执行循环，并处理其他高级格式和展示任务；在将“业务逻辑”与“展示问题”分离的精神下，您更喜欢所有后者作为模板的一部分。这就是专门的第三方模板化包的用武之地。这里有许多这样的包，但本书的所有作者，都曾使用过并[编写](https://learning.oreilly.com/library/view/python-cookbook/0596001673/ch03s23.xhtml)过其中一些，目前更倾向于使用[jinja2](https://oreil.ly/PYYm5)，接下来进行详细介绍。

## jinja2 包

对于严肃的模板化任务，我们推荐使用 jinja2（在[PyPI](https://oreil.ly/1DgV9)上可用，像其他第三方 Python 包一样，因此可以轻松通过**pip install jinja2**安装）。

[jinja2 文档](https://oreil.ly/w6IiV)非常出色和详尽，涵盖了模板语言本身（在概念上模仿 Python，但有许多不同之处，以支持在 HTML 中嵌入它和特定于展示问题的独特需求）；您的 Python 代码用于连接到 jinja2 的 API，并在必要时扩展或扩展它；以及其他问题，从安装到国际化，从代码沙箱到从其他模板引擎移植——更不用说宝贵的提示和技巧了。

在本节中，我们仅涵盖了 jinja2 强大功能的一小部分，这些足以让你在安装后开始使用。我们强烈建议阅读 jinja2 的文档，以获取大量额外有用的信息。

### jinja2.Environment 类

当你使用 jinja2 时，总会涉及一个 Environment 实例——在少数情况下，你可以让它默认为一个通用的“共享环境”，但这不推荐。只有在非常高级的用法中，当你从不同来源获取模板（或使用不同的模板语言语法）时，才会定义多个环境实例——通常情况下，你会实例化一个单独的 Environment 实例 *env*，用于渲染所有需要的模板。

你可以在构建 *env* 时通过向其构造函数传递命名参数的方式进行多种方式的定制（包括修改关键的模板语言语法方面，比如哪些定界符用于开始和结束块、变量、注释等）。在实际使用中，你几乎总是会传递一个名为 loader 的命名参数（其他很少设置）。

环境的 loader 指定了如何在请求时加载模板——通常是文件系统中的某个目录，或者也许是某个数据库（你需要编写 jinja2.Loader 的自定义子类来实现后者），但也有其他可能性。你需要一个 loader 来让模板享受 jinja2 的一些最强大的特性，比如 [*template inheritance*](https://oreil.ly/yhG7y)。

你可以在实例化 *env* 时配备自定义 [filters](https://oreil.ly/ouZ9A), [tests](https://oreil.ly/2NL9l), [extensions](https://oreil.ly/K4wHT) 等（这些也可以稍后添加）。

在稍后的示例中，我们假设 *env* 是通过 loader=jinja2.FileSystemLoader('*/path/to/templates*') 实例化的，没有进一步的增强——事实上，为简单起见，我们甚至不会使用 loader 参数。

*env*.get_template(*name*) 获取、编译并返回基于 env.loader(*name*) 返回内容的 jinja2.Template 实例。在本节末尾的示例中，为简单起见，我们将使用罕见的 env.from_string(*s*) 来从字符串 *s* 构建 jinja2.Template 的实例。

### jinja2.Template 类

jinja2.Template 的一个实例 *t* 拥有许多属性和方法，但在实际生活中你几乎只会使用以下这个：

| 渲染 | *t*.render(...*context*...) *context* 参数与传递给 dict 构造函数的内容相同——一个映射实例，和/或丰富和潜在覆盖映射键值连接的命名参数。

*t*.render(*context*) 返回一个（Unicode）字符串，该字符串是应用于模板 *t* 的 *context* 参数后生成的结果。

### 使用 jinja2 构建 HTML

这里是一个使用 jinja2 模板生成 HTML 的示例。具体来说，就像在“用 bs4 构建 HTML”中一样，以下函数接受一个“行”（序列）的序列，并返回一个 HTML 表格来显示它们的值：

```py
TABLE_TEMPLATE = '''\ <table>
{% for s in s_of_s %}
 <tr>
  {% for item in s %}
 <td>{{item}}</td>
  {% endfor %}
 </tr>
{% endfor %}
</table>'''
`def` mktable_with_jinja2(s_of_s):
    env = jinja2.Environment(
        trim_blocks=`True`,
        lstrip_blocks=`True`,
        autoescape=`True`)
    t = env.from_string(TABLE_TEMPLATE)
    `return` t.render(s_of_s=s_of_s)
```

函数使用选项 autoescape=**True**，自动“转义”包含标记字符如<, >和&的字符串；例如，使用 autoescape=**True**，'g>h'渲染为'g&gt;h'。

选项 trim_blocks=**True**和 lstrip_blocks=**True**纯粹是为了美观起见，以确保模板字符串和渲染的 HTML 字符串都能被良好地格式化；当然，当浏览器渲染 HTML 时，HTML 文本本身是否良好格式化并不重要。

通常情况下，您会始终使用加载器参数构建环境，并通过方法调用如*t* = *env*.get_template(*template_name*)从文件或其他存储加载模板。在这个示例中，为了一目了然，我们省略了加载器，并通过调用方法*env*.from_string 从字符串构建模板。请注意，jinja2 不是 HTML 或 XML 特定的，因此仅使用它并不能保证生成内容的有效性，如果需要符合标准，您应该仔细检查生成的内容。

该示例仅使用 jinja2 模板语言提供的众多功能中最常见的两个特性：*循环*（即，用{% for ... %}和{% endfor %}括起来的块）和*参数替换*（内联表达式用{{和}}括起来）。

这里是我们刚定义的函数的一个示例用法：

```py
>>> example = (
...   ('foo', 'g>h', 'g&h'),
...   ('zip', 'zap', 'zop'),
... )
>>> print(mktable_with_jinja2(example))
```

```py
<table>
 <tr>
 <td>foo</td>
 <td>g&gt;h</td>
 <td>g&amp;h</td>
 </tr>
 <tr>
 <td>zip</td>
 <td>zap</td>
 <td>zop</td>
 </tr>
</table>
```

¹ BeautifulSoup 的[文档](https://oreil.ly/B-xCI)提供了关于安装各种解析器的详细信息。

² 正如在 BeautifulSoup 的[文档](https://oreil.ly/vTXcK)中解释的那样，它还展示了各种指导或完全覆盖 BeautifulSoup 关于编码猜测的方法。
