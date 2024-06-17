# 第十六章：盒子中的数据：持久存储

> 在获得数据之前进行理论化是一个重大的错误。
> 
> 亚瑟·柯南·道尔

活跃的程序访问存储在随机存取存储器（RAM）中的数据。RAM 非常快速，但价格昂贵，并且需要持续的电源供应；如果电源中断，内存中的所有数据都将丢失。磁盘驱动器比 RAM 慢，但容量更大，成本更低，并且即使有人绊倒电源线后，也可以保留数据。因此，计算机系统中的大量工作已经致力于在磁盘和 RAM 之间进行最佳权衡。作为程序员，我们需要*持久性*：使用非易失性介质（如磁盘）存储和检索数据。

本章讨论了为不同目的优化的数据存储的不同类型：平面文件、结构化文件和数据库。除了输入和输出之外的文件操作在第十四章中有所涵盖。

*记录*是指一块相关数据的术语，由各个*字段*组成。

# 平面文本文件

最简单的持久性是普通的平面文件。如果您的数据结构非常简单并且在磁盘和内存之间交换所有数据，则此方法非常有效。纯文本数据可能适合这种处理方式。

# 填充文本文件

在这种格式中，记录中的每个字段都有固定的宽度，并且在文件中（通常用空格字符）填充到该宽度，使得每行（记录）具有相同的宽度。程序员可以使用`seek()`来在文件中跳转，并且仅读取和写入需要的记录和字段。

# 表格式文本文件

对于简单的文本文件，唯一的组织级别是行。有时，您可能需要比这更多的结构。您可能希望将数据保存供程序稍后使用，或者将数据发送到另一个程序。

有许多格式，这里是如何区分它们的方式：

+   *分隔符*或*分隔符*字符，如制表符（`'\t'`）、逗号（`','`）或竖线（`'|'`）。这是逗号分隔值（CSV）格式的一个例子。

+   `'<'` 和 `'>'`围绕*标签*。例如 XML 和 HTML。

+   标点符号。一个例子是 JavaScript 对象表示法（JSON）。

+   缩进。一个例子是 YAML（递归定义为“YAML 不是标记语言”）。

+   杂项，例如程序的配置文件。

这些结构化文件格式中的每一个都可以由至少一个 Python 模块读取和写入。

## CSV

分隔文件通常用作电子表格和数据库的交换格式。您可以手动读取 CSV 文件，一次读取一行，在逗号分隔符处拆分每行成字段，并将结果添加到诸如列表和字典之类的数据结构中。但最好使用标准的`csv`模块，因为解析这些文件可能比您想象的要复杂得多。在处理 CSV 时需要记住以下几个重要特征：

+   有些分隔符除了逗号以外还有：`'|'` 和 `'\t'`（制表符）是常见的。

+   有些有*转义序列*。如果定界符字符可以出现在字段内，则整个字段可能用引号字符括起来或者在某些转义字符之前。

+   文件具有不同的行尾字符。Unix 使用`'\n'`，Microsoft 使用`'\r\n'`，而 Apple 曾使用`'\r'`，但现在使用`'\n'`。

+   第一行可能包含列名。

首先，我们看看如何读取和写入包含列列表的行列表：

```py
>>> import csv
>>> villains = [
...     ['Doctor', 'No'],
...     ['Rosa', 'Klebb'],
...     ['Mister', 'Big'],
...     ['Auric', 'Goldfinger'],
...     ['Ernst', 'Blofeld'],
...     ]
>>> with open('villains', 'wt') as fout:  # a context manager
...     csvout = csv.writer(fout)
...     csvout.writerows(villains)
```

这将创建包含这些行的文件*villains*：

```py
Doctor,No
Rosa,Klebb
Mister,Big
Auric,Goldfinger
Ernst,Blofeld
```

现在，我们试着再次读取它：

```py
>>> import csv
>>> with open('villains', 'rt') as fin:  # context manager
...     cin = csv.reader(fin)
...     villains = [row for row in cin]  # a list comprehension
...
>>> print(villains)
[['Doctor', 'No'], ['Rosa', 'Klebb'], ['Mister', 'Big'],
['Auric', 'Goldfinger'], ['Ernst', 'Blofeld']]
```

我们利用了`reader()`函数创建的结构。它在`cin`对象中创建了可以在`for`循环中提取的行。

使用`reader()`和`writer()`及其默认选项，列由逗号分隔，行由换行符分隔。

数据可以是字典列表而不是列表列表。让我们再次使用新的`DictReader()`函数和指定的列名读取*villains*文件：

```py
>>> import csv
>>> with open('villains', 'rt') as fin:
...     cin = csv.DictReader(fin, fieldnames=['first', 'last'])
...     villains = [row for row in cin]
...
>>> print(villains)
[OrderedDict([('first', 'Doctor'), ('last', 'No')]),
OrderedDict([('first', 'Rosa'), ('last', 'Klebb')]),
OrderedDict([('first', 'Mister'), ('last', 'Big')]),
OrderedDict([('first', 'Auric'), ('last', 'Goldfinger')]),
OrderedDict([('first', 'Ernst'), ('last', 'Blofeld')])]
```

那个 `OrderedDict` 用于兼容 Python 3.6 之前的版本，当时字典默认保持其顺序。

让我们使用新的 `DictWriter()` 函数重新编写 CSV 文件。我们还调用 `writeheader()` 来向 CSV 文件写入初始列名行：

```py
import csv
villains = [
 {'first': 'Doctor', 'last': 'No'},
 {'first': 'Rosa', 'last': 'Klebb'},
 {'first': 'Mister', 'last': 'Big'},
 {'first': 'Auric', 'last': 'Goldfinger'},
 {'first': 'Ernst', 'last': 'Blofeld'},
 ]
with open('villains.txt', 'wt') as fout:
 cout = csv.DictWriter(fout, ['first', 'last'])
 cout.writeheader()
 cout.writerows(villains)
```

这创建了一个带有头行的*villains.csv*文件（示例 16-1）。

##### 示例 16-1\. villains.csv

```py
first,last
Doctor,No
Rosa,Klebb
Mister,Big
Auric,Goldfinger
Ernst,Blofeld
```

现在，让我们读取它。在`DictReader()`调用中省略`fieldnames`参数，告诉它使用文件的第一行的值(`first,last`)作为列标签和匹配的字典键：

```py
>>> import csv
>>> with open('villains.csv', 'rt') as fin:
...     cin = csv.DictReader(fin)
...     villains = [row for row in cin]
...
>>> print(villains)
[OrderedDict([('first', 'Doctor'), ('last', 'No')]),
OrderedDict([('first', 'Rosa'), ('last', 'Klebb')]),
OrderedDict([('first', 'Mister'), ('last', 'Big')]),
OrderedDict([('first', 'Auric'), ('last', 'Goldfinger')]),
OrderedDict([('first', 'Ernst'), ('last', 'Blofeld')])]
```

## XML

分隔文件仅传达两个维度：行（行）和列（行内字段）。如果要在程序之间交换数据结构，需要一种方法将层次结构、序列、集合和其他结构编码为文本。

XML 是一种突出显示的*标记*格式，它使用*标签*来界定数据，就像这个样本*menu.xml*文件中所示。

```py
<?xml version="1.0"?>
<menu>
  <breakfast hours="7-11">
    <item price="$6.00">breakfast burritos</item>
    <item price="$4.00">pancakes</item>
  </breakfast>
  <lunch hours="11-3">
    <item price="$5.00">hamburger</item>
  </lunch>
  <dinner hours="3-10">
    <item price="8.00">spaghetti</item>
  </dinner>
</menu>
```

以下是 XML 的一些重要特征：

+   标签以 `<` 字符开始。这个示例中的标签是 `menu`、`breakfast`、`lunch`、`dinner` 和 `item`。

+   空白符将被忽略。

+   通常像`<menu>`这样的*开始标签*后面跟着其他内容，然后是最终匹配的*结束标签*，例如`</menu>`。

+   标签可以在其他标签内*嵌套*到任何级别。在此示例中，`item`标签是`breakfast`、`lunch`和`dinner`标签的子标签；它们反过来是`menu`的子标签。

+   可选的*属性*可以出现在开始标签内。在这个例子中，`price` 是 `item` 的一个属性。

+   标签可以包含*值*。在这个例子中，每个 `item` 都有一个值，比如第二个早餐项目的 `pancakes`。

+   如果名为`thing`的标签没有值或子项，则可以通过在闭合尖括号之前包含一个斜杠来表示为单个标签，例如`<thing/>`，而不是开始和结束标签，像`<thing></thing>`。

+   将数据放在哪里的选择——属性、值、子标签——有些是任意的。例如，我们可以将最后一个 `item` 标签写成 `<item price="$8.00" food="spaghetti"/>`。

XML 通常用于数据 *feeds* 和 *messages*，并具有诸如 RSS 和 Atom 之类的子格式。一些行业有许多专门的 XML 格式，比如 [金融领域](http://bit.ly/xml-finance)。

XML 的超级灵活性启发了多个 Python 库，这些库在方法和功能上有所不同。

在 Python 中解析 XML 的最简单方法是使用标准的 `ElementTree` 模块。下面是一个简单的程序，用于解析 *menu.xml* 文件并打印一些标签和属性：

```py
>>> import xml.etree.ElementTree as et
>>> tree = et.ElementTree(file='menu.xml')
>>> root = tree.getroot()
>>> root.tag
'menu'
>>> for child in root:
...     print('tag:', child.tag, 'attributes:', child.attrib)
...     for grandchild in child:
...         print('\ttag:', grandchild.tag, 'attributes:', grandchild.attrib)
...
tag: breakfast attributes: {'hours': '7-11'}
 tag: item attributes: {'price': '$6.00'}
 tag: item attributes: {'price': '$4.00'}
tag: lunch attributes: {'hours': '11-3'}
 tag: item attributes: {'price': '$5.00'}
tag: dinner attributes: {'hours': '3-10'}
 tag: item attributes: {'price': '8.00'}
>>> len(root)     # number of menu sections
3
>>> len(root[0])  # number of breakfast items
2
```

对于嵌套列表中的每个元素，`tag` 是标签字符串，`attrib` 是其属性的字典。`ElementTree` 有很多其他搜索 XML 派生数据，修改它的方法，甚至编写 XML 文件的方法。`ElementTree` [文档](http://bit.ly/elementtree) 上有详细信息。

其他标准的 Python XML 库包括以下内容：

`xml.dom`

文件对象模型（DOM），对于 JavaScript 开发者来说很熟悉，它将 web 文档表示为分层结构。这个模块将整个 XML 文件加载到内存中，并允许您平等地访问所有部分。

`xml.sax`

简单的 XML API，或 SAX，可以实时解析 XML，因此它不必一次加载所有内容到内存中。因此，如果您需要处理非常大的 XML 流，它可能是一个不错的选择。

## XML 安全注意事项

您可以使用本章中描述的所有格式将对象保存到文件中，然后再读取它们。可以利用这个过程来造成安全问题。

例如，下面来自十亿笑话维基百科页面的 XML 片段定义了 10 个嵌套实体，每个实体扩展了低一级 10 次，总扩展量达到了十亿：

```py
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
 <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
 <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
 <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
 <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
 <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
 <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

坏消息：十亿笑话将会使前面提到的所有 XML 库爆炸。[Defused XML](https://bitbucket.org/tiran/defusedxml) 列出了这个攻击和其他攻击，以及 Python 库的漏洞。

该链接展示了如何更改许多库的设置以避免这些问题。此外，您可以使用 `defusedxml` 库作为其他库的安全前端：

```py
>>> # insecure:
>>> from xml.etree.ElementTree import parse
>>> et = parse(xmlfile)
>>> # protected:
>>> from defusedxml.ElementTree import parse
>>> et = parse(xmlfile)
```

标准的 Python 网站上也有关于 [XML 漏洞](https://oreil.ly/Rnsiw) 的页面。

## HTML

庞大的数据以超文本标记语言（HTML）的形式保存，这是 web 的基本文档格式。问题是其中的大部分都不符合 HTML 规则，这可能会使解析变得困难。HTML 比数据交换格式更好地显示格式。因为本章旨在描述相当明确定义的数据格式，我已将有关 HTML 的讨论分离到 第十八章 中。

## JSON

[JavaScript Object Notation (JSON)](http://www.json.org) 已成为非常流行的数据交换格式，超越了其 JavaScript 的起源。JSON 格式是 JavaScript 的子集，通常也是合法的 Python 语法。它与 Python 的密切配合使其成为程序之间数据交换的良好选择。您将在 第十八章 中看到许多用于 Web 开发的 JSON 示例。

与各种 XML 模块不同，主要的 JSON 模块只有一个，其名称令人难忘，即 `json`。此程序将数据编码（转储）为 JSON 字符串，并将 JSON 字符串解码（加载）回数据。在下面的示例中，让我们构建一个包含先前 XML 示例中的数据的 Python 数据结构：

```py
>>> menu = \
... {
... "breakfast": {
...         "hours": "7-11",
...         "items": {
...                 "breakfast burritos": "$6.00",
...                 "pancakes": "$4.00"
...                 }
...         },
... "lunch" : {
...         "hours": "11-3",
...         "items": {
...                 "hamburger": "$5.00"
...                 }
...         },
... "dinner": {
...         "hours": "3-10",
...         "items": {
...                 "spaghetti": "$8.00"
...                 }
...         }
... }
.
```

接下来，使用 `dumps()` 将数据结构 (`menu`) 编码为 JSON 字符串 (`menu_json`)：

```py
>>> import json
>>> menu_json = json.dumps(menu)
>>> menu_json
'{"dinner": {"items": {"spaghetti": "$8.00"}, "hours": "3-10"},
"lunch": {"items": {"hamburger": "$5.00"}, "hours": "11-3"},
"breakfast": {"items": {"breakfast burritos": "$6.00", "pancakes":
"$4.00"}, "hours": "7-11"}}'
```

现在，让我们通过使用 `loads()` 将 JSON 字符串 `menu_json` 转换回 Python 数据结构 (`menu2`)：

```py
>>> menu2 = json.loads(menu_json)
>>> menu2
{'breakfast': {'items': {'breakfast burritos': '$6.00', 'pancakes':
'$4.00'}, 'hours': '7-11'}, 'lunch': {'items': {'hamburger': '$5.00'},
'hours': '11-3'}, 'dinner': {'items': {'spaghetti': '$8.00'}, 'hours': '3-10'}}
```

`menu` 和 `menu2` 都是具有相同键和值的字典。

在尝试编码或解码某些对象时，可能会遇到异常，包括诸如 `datetime` 的对象（在 第十三章 中有详细说明），如下所示：

```py
>>> import datetime
>>> import json
>>> now = datetime.datetime.utcnow()
>>> now
datetime.datetime(2013, 2, 22, 3, 49, 27, 483336)
>>> json.dumps(now)
Traceback (most recent call last):
# ... (deleted stack trace to save trees)
TypeError: datetime.datetime(2013, 2, 22, 3, 49, 27, 483336)
 is not JSON serializable
>>>
```

这可能是因为 JSON 标准未定义日期或时间类型；它期望您定义如何处理它们。您可以将 `datetime` 转换为 JSON 理解的内容，如字符串或 *epoch* 值（见 第十三章）：

```py
>>> now_str = str(now)
>>> json.dumps(now_str)
'"2013-02-22 03:49:27.483336"'
>>> from time import mktime
>>> now_epoch = int(mktime(now.timetuple()))
>>> json.dumps(now_epoch)
'1361526567'
```

如果 `datetime` 值可能出现在通常转换的数据类型中间，这些特殊转换可能会令人困扰。您可以通过使用继承修改 JSON 的编码方式来进行修改，详见 第十章。Python 的 JSON [文档](http://bit.ly/json-docs) 提供了关于复数的示例，这也使 JSON 看起来像是死的。我们修改它以适应 `datetime`：

```py
>>> import datetime
>>> now = datetime.datetime.utcnow()
>>> class DTEncoder(json.JSONEncoder):
...     def default(self, obj):
...         # isinstance() checks the type of obj
...         if isinstance(obj, datetime.datetime):
...             return int(mktime(obj.timetuple()))
...         # else it's something the normal decoder knows:
...         return json.JSONEncoder.default(self, obj)
...
>>> json.dumps(now, cls=DTEncoder)
'1361526567'
```

新的类 `DTEncoder` 是 `JSONEncoder` 的子类或子类，我们需要重写其唯一的 `default()` 方法以添加 `datetime` 处理。继承确保所有其他内容都将由父类处理。

`isinstance()` 函数检查对象 `obj` 是否为 `datetime.datetime` 类的实例。因为 Python 中的所有东西都是对象，所以 `isinstance()` 在任何地方都适用：

```py
>>> import datetime
>>> now = datetime.datetime.utcnow()
>>> type(now)
<class 'datetime.datetime'>
>>> isinstance(now, datetime.datetime)
True
>>> type(234)
<class 'int'>
>>> isinstance(234, int)
True
>>> type('hey')
<class 'str'>
>>> isinstance('hey', str)
True
```

###### 注意

对于 JSON 和其他结构化文本格式，您可以从文件加载到数据结构中，而无需事先了解结构的任何信息。然后，您可以使用 `isinstance()` 和适当类型的方法遍历这些结构以检查其值。例如，如果其中一个项目是字典，您可以通过 `keys()`、`values()` 和 `items()` 提取内容。

在让您以困难的方式处理后，事实证明有一种更简单的方法可以将 `datetime` 对象转换为 JSON：

```py
>>> import datetime
>>> import json
>>> now = datetime.datetime.utcnow()
>>> json.dumps(now, default=str)
'"2019-04-17 21:54:43.617337"'
```

`default=str` 告诉 `json.dumps()` 对于它不理解的数据类型应用 `str()` 转换函数。这是因为 `datetime.datetime` 类的定义包含一个 `__str__()` 方法。

## YAML

类似于 JSON，[YAML](http://www.yaml.org)具有键和值，但处理更多数据类型，如日期和时间。标准的 Python 库尚未包含 YAML 处理，因此您需要安装名为[`yaml`](http://pyyaml.org/wiki/PyYAML)的第三方库来操作它。`((("dump() function")))((("load() function")))load()`将 YAML 字符串转换为 Python 数据，而`dump()`则相反。

以下的 YAML 文件，*mcintyre.yaml*，包含了加拿大诗人詹姆斯·麦金太尔（James McIntyre）的信息，包括他的两首诗：

```py
name:
  first: James
  last: McIntyre
dates:
  birth: 1828-05-25
  death: 1906-03-31
details:
  bearded: true
  themes: [cheese, Canada]
books:
  url: http://www.gutenberg.org/files/36068/36068-h/36068-h.htm
poems:
  - title: 'Motto'
    text: |
      Politeness, perseverance and pluck,
      To their possessor will bring good luck.
  - title: 'Canadian Charms'
    text: |
      Here industry is not in vain,
      For we have bounteous crops of grain,
      And you behold on every field
      Of grass and roots abundant yield,
      But after all the greatest charm
      Is the snug home upon the farm,
      And stone walls now keep cattle warm.
```

值如`true`、`false`、`on`和`off`会转换为 Python 布尔值。整数和字符串会转换为它们的 Python 等效项。其他语法创建列表和字典：

```py
>>> import yaml
>>> with open('mcintyre.yaml', 'rt') as fin:
>>>     text = fin.read()
>>> data = yaml.load(text)
>>> data['details']
{'themes': ['cheese', 'Canada'], 'bearded': True}
>>> len(data['poems'])
2
```

创建的数据结构与 YAML 文件中的相匹配，这在某些情况下是多层次的。您可以使用这个字典/列表/字典引用获取第二首诗的标题：

```py
>>> data['poems'][1]['title']
'Canadian Charms'
```

###### 警告

PyYAML 可以从字符串加载 Python 对象，这是危险的。如果您导入不信任的 YAML，请使用`safe_load()`而不是`load()`。更好的做法是，*总是*使用`safe_load()`。阅读 Ned Batchelder 的博客文章[“War is Peace”](http://bit.ly/war-is-peace)了解未受保护的 YAML 加载如何危及 Ruby on Rails 平台。

## Tablib

在阅读所有先前的章节之后，有一个第三方包可以让您导入、导出和编辑 CSV、JSON 或 YAML 格式的表格数据，¹还有 Microsoft Excel、Pandas DataFrame 和其他几个。您可以用熟悉的叠歌（`pip install tablib`）安装它，并查看[文档](http://docs.python-tablib.org)。

## Pandas

这是介绍[pandas](https://pandas.pydata.org)的好地方——一个用于结构化数据的 Python 库。它是处理现实生活数据问题的优秀工具：

+   读写许多文本和二进制文件格式：

    +   文本，字段由逗号（CSV）、制表符（TSV）或其他字符分隔

    +   固定宽度文本

    +   Excel

    +   JSON

    +   HTML 表格

    +   SQL

    +   HDF5

    +   和[其他](https://oreil.ly/EWlgS)。

+   分组、拆分、合并、索引、切片、排序、选择、标记

+   转换数据类型

+   更改大小或形状

+   处理丢失的数据

+   生成随机值

+   管理时间序列

读取函数返回一个[`DataFrame`](https://oreil.ly/zupYI)对象，Pandas 的标准表示形式，用于二维数据（行和列）。在某些方面类似于电子表格或关系数据库表。它的一维小兄弟是[`Series`](https://oreil.ly/pISZT)。

示例 16-2 演示了一个简单的应用程序，从示例 16-1 中读取我们的*villains.csv*文件。

##### 示例 16-2\. 使用 Pandas 读取 CSV

```py
>>> import pandas
>>>
>>> data = pandas.read_csv('villains.csv')
>>> print(data)
    first        last
0  Doctor          No
1    Rosa       Klebb
2  Mister         Big
3   Auric  Goldfinger
4   Ernst     Blofeld
```

变量`data`刚刚展示的是一个`DataFrame`。它比基本的 Python 字典有更多的技巧。它特别适用于使用 NumPy 进行大量数字工作和为机器学习准备数据。

参考 Pandas 文档的[“入门指南”](https://oreil.ly/VKSrZ)部分以及[“10 分钟入门 Pandas”](https://oreil.ly/CLoVg)中的工作示例。

让我们举个小日历例子使用 Pandas——列出 2019 年前三个月的第一天的列表：

```py
>>> import pandas
>>> dates = pandas.date_range('2019-01-01', periods=3, freq='MS')
>>> dates
DatetimeIndex(['2019-01-01', '2019-02-01', '2019-03-01'],
 dtype='datetime64[ns]', freq='MS')
```

你可以编写一些代码来实现这一点，使用第十三章中描述的时间和日期函数，但这需要更多的工作——特别是调试（日期和时间常常令人沮丧）。Pandas 还处理许多特殊的日期/时间[细节](https://oreil.ly/vpeTP)，如业务月和年。

后面当我谈论映射时，Pandas 会再次出现（“Geopandas”）以及科学应用（“Pandas”）。

## 配置文件

大多数程序提供各种*options*或*settings*。动态的可以作为程序参数提供，但长期的需要保存在某个地方。定义自己快速而脏的*config file*格式的诱惑力很强——但要抵制它。它经常变得脏乱，但并不快速。您需要维护编写程序和读取程序（有时称为*parser*）。有很多好的替代方案可以直接插入您的程序，包括前面的部分中提到的那些。

这里，我们将使用标准`configparser`模块，它处理 Windows 风格的*.ini*文件。这些文件有*key* = *value*定义的部分。这是一个最小的*settings.cfg*文件：

```py
[english]
greeting = Hello

[french]
greeting = Bonjour

[files]
home = /usr/local
# simple interpolation:
bin = %(home)s/bin
```

这是将其读入 Python 数据结构的代码：

```py
>>> import configparser
>>> cfg = configparser.ConfigParser()
>>> cfg.read('settings.cfg')
['settings.cfg']
>>> cfg
<configparser.ConfigParser object at 0x1006be4d0>
>>> cfg['french']
<Section: french>
>>> cfg['french']['greeting']
'Bonjour'
>>> cfg['files']['bin']
'/usr/local/bin'
```

还有其他选择，包括更高级的插值。参见`configparser`的[文档](http://bit.ly/configparser)。如果需要比两级更深的嵌套，请尝试 YAML 或 JSON。

# 二进制文件

有些文件格式设计用于存储特定的数据结构，但既不是关系型数据库也不是 NoSQL 数据库。接下来的部分介绍了其中的一些。

## 填充的二进制文件和内存映射

这些类似于填充的文本文件，但内容可能是二进制的，填充字节可能是`\x00`而不是空格字符。每个记录和记录内的每个字段都有固定的大小。这使得在文件中通过`seek()`查找所需的记录和字段变得更简单。数据的每一项操作都是手动的，因此这种方法通常仅在非常低级别（例如接近硬件）的情况下使用。

这种形式的数据可以使用标准的`mmap`库进行*memory mapped*。参见一些[示例](https://pymotw.com/3/mmap)和标准的[文档](https://oreil.ly/eI0mv)。

## 电子表格

电子表格，特别是 Microsoft Excel，是广泛使用的二进制数据格式。如果可以将电子表格保存为 CSV 文件，可以使用早期描述的标准`csv`模块进行读取。这对于二进制的`xls`文件、[`xlrd`](https://oreil.ly/---YE)或`tablib`（在“Tablib”早些时候提到）都适用。

## HDF5

[HDF5](https://oreil.ly/QTT6x) 是用于多维或层次化数值数据的二进制数据格式。它主要用于科学领域，其中快速访问大数据集（从几千兆字节到几太字节）是常见需求。尽管在某些情况下，HDF5 可能是数据库的良好替代品，但由于某些原因，HDF5 在商业界几乎不为人知。它最适合于*WORM*（写入一次/多次读取）应用程序，这种应用程序不需要数据库对抗冲突写入。以下是一些可能对你有用的模块：

+   `h5py` 是一个功能齐全的低级接口。阅读[文档](http://www.h5py.org)和[代码](https://github.com/h5py/h5py)。

+   `PyTables` 是一个稍微高级的库，具有类似数据库的功能。阅读[文档](http://www.pytables.org)和[代码](http://pytables.github.com)。

这两者都是讨论 Python 在第二十二章科学应用程序方面的应用。我在这里提到 HDF5 是因为你可能需要存储和检索大量数据，并愿意考虑除了传统数据库解决方案以外的其他选择。一个很好的例子是[百万首歌数据集](http://millionsongdataset.com)，其中包含以 HDF5 和 SQLite 格式提供的可下载歌曲数据。

## TileDB

用于密集或稀疏数组存储的最新后继者是[TileDB](https://tiledb.io)。通过运行 `pip install tiledb` 安装[Python 接口](https://github.com/TileDB-Inc/TileDB-Py)（包括 TileDB 库本身）。这专为科学数据和应用程序设计。

# 关系数据库

关系数据库虽然只有大约 40 年的历史，但在计算世界中无处不在。你几乎肯定会在某个时候必须处理它们。当你这样做时，你会感谢它们提供的：

+   多个同时用户访问数据

+   受用户防止损坏

+   高效存储和检索数据的方法

+   由*模式*定义的数据和受*约束*限制

+   *连接*以找到跨多种类型数据的关系

+   一种声明性（而不是命令性）查询语言：*SQL*（结构化查询语言）

这些被称为*关系*数据库，因为它们展示了不同类型数据之间的关系，以矩形*表格*的形式。例如，在我们之前的菜单示例中，每个项目与其价格之间存在关系。

表格是*列*（数据字段）和*行*（单个数据记录）的矩形网格，类似于电子表格。行和列的交集是表格的*单元格*。要创建表格，需要命名它并指定其列的顺序、名称和类型。每行都具有相同的列，尽管可以定义某列允许在单元格中包含缺失数据（称为*nulls*）。在菜单示例中，你可以为每个出售的项目创建一个包含价格等列的表格。

表的列或一组列通常是表的*主键*；其值在表中必须是唯一的。这样可以防止将相同的数据多次添加到表中。此键被*索引*以便在查询期间快速查找。索引的工作方式有点像书籍索引，使得快速找到特定行。

每个表都位于父*数据库*中，就像目录中的文件一样。两个层次的层次结构有助于使事物组织得更好一些。

###### 注意

是的，单词*数据库*以多种方式使用：作为服务器，表容器和其中存储的数据。如果您同时提到它们所有，可能有助于将它们称为*数据库服务器*，*数据库*和*数据*。

如果要通过某些非关键列值查找行，请在该列上定义*二级索引*。否则，数据库服务器必须执行*表扫描*—对每一行进行匹配列值的 brute-force 搜索。

表可以通过*外键*相互关联，并且列值可以受限于这些键。

## SQL

SQL 不是 API 或协议，而是声明性*语言*：您说您想要什么而不是如何做。这是关系数据库的通用语言。SQL 查询是由客户端发送到数据库服务器的文本字符串，数据库服务器然后确定如何处理它们。

有各种各样的 SQL 标准定义，所有数据库供应商都添加了自己的调整和扩展，导致了许多 SQL *方言*。如果您将数据存储在关系型数据库中，SQL 可以提供一些可移植性。然而，方言和操作差异可能会使您将数据移动到另一种类型的数据库变得困难。SQL 语句有两个主要类别：

DDL（数据定义语言）

处理表，数据库和用户的创建，删除，约束和权限。

DML（数据操纵语言）

处理数据插入，选择，更新和删除。

表 16-1 列出了基本的 SQL DDL 命令。

表 16-1\. 基本的 SQL DDL 命令

| 操作 | SQL 模式 | SQL 示例 |
| --- | --- | --- |
| 创建数据库 | `CREATE DATABASE` *dbname* | `CREATE DATABASE d` |
| 选择当前数据库 | `USE` *dbname* | `USE d` |
| 删除数据库及其表 | `DROP DATABASE` *dbname* | `DROP DATABASE d` |
| 创建表 | `CREATE TABLE` *tbname* `(` *coldefs* `)` | `CREATE TABLE t (id INT, count INT)` |
| 删除表 | `DROP TABLE` *tbname* | `DROP TABLE t` |
| 从表中删除所有行 | `TRUNCATE TABLE` *tbname* | `TRUNCATE TABLE t` |

###### 注意

为什么所有的大写字母？SQL 不区分大小写，但在代码示例中大声喊出关键字是一种传统（不要问我为什么），以区分它们和列名。

关系数据库的主要 DML 操作通常以 CRUD 缩写而闻名：

+   使用 SQL `INSERT`语句*C*reate

+   *R*通过`SELECT`读取

+   *U*通过`UPDATE`更新

+   *D*通过`DELETE`删除

表 16-2 查看了可用于 SQL DML 的命令。

表 16-2\. 基本 SQL DML 命令

| 操作 | SQL 模式 | SQL 示例 |
| --- | --- | --- |
| 添加一行 | `INSERT INTO` *tbname* `VALUES(` … `)` | `INSERT INTO t VALUES(7, 40)` |
| 选择所有行和列 | `SELECT * FROM` *tbname* | `SELECT * FROM t` |
| 选择所有行，某些列 | `SELECT` *cols* `FROM` *tbname* | `SELECT id, count FROM t` |
| 选择某些行，某些列 | `SELECT` *cols* `FROM` *tbname* `WHERE` *condition* | `SELECT id, count from t WHERE count > 5 AND id = 9` |
| 更改某列中的一些行 | `UPDATE` *tbname* `SET` *col* `=` *value* `WHERE` *condition* | `UPDATE t SET count=3 WHERE id=5` |
| 删除某些行 | `DELETE FROM` *tbname* `WHERE` *condition* | `DELETE FROM t WHERE count <= 10 OR id = 16` |

## DB-API

应用程序编程接口（API）是一组可以调用以访问某些服务的函数。[DB-API](http://bit.ly/db-api) 是 Python 访问关系型数据库的标准 API。使用它，你可以编写一个程序，可以与多种关系型数据库一起工作，而不是为每种数据库编写单独的程序。它类似于 Java 的 JDBC 或 Perl 的 dbi。

它的主要功能如下：

`connect()`

建立与数据库的连接；这可以包括用户名、密码、服务器地址等参数。

`cursor()`

创建一个 *cursor* 对象来管理查询。

`execute()` 和 `executemany()`

对数据库运行一个或多个 SQL 命令。

`fetchone()`、`fetchmany()` 和 `fetchall()`

从 `execute()` 获取结果。

接下来的章节中的 Python 数据库模块符合 DB-API，通常具有扩展和一些细节上的差异。

## SQLite

[SQLite](http://www.sqlite.org) 是一个优秀、轻量、开源的关系型数据库。它作为标准的 Python 库实现，并且将数据库存储在普通文件中。这些文件可以跨机器和操作系统移植，使 SQLite 成为简单关系数据库应用程序的非常便携的解决方案。虽然不如 MySQL 或 PostgreSQL 功能全面，但它支持 SQL，并且能够管理多个同时用户。Web 浏览器、智能手机和其他应用程序都将 SQLite 用作嵌入式数据库。

首先通过 `connect()` 连接到你要使用或创建的本地 SQLite 数据库文件。这个文件相当于其他服务器中父表所在的类似目录的*数据库*。特殊字符串 `':memory:'` 仅在内存中创建数据库；这对测试很快并且很有用，但在程序终止或计算机关机时会丢失数据。

作为下一个示例，让我们创建一个名为 `enterprise.db` 的数据库和一个名为 `zoo` 的表，以管理我们蓬勃发展的路边宠物动物园业务。表的列如下：

`critter`

变长字符串，以及我们的主键。

`count`

这种动物的当前库存的整数计数。

`damages`

我们当前因动物与人类互动而造成的损失金额。

```py
>>> import sqlite3
>>> conn = sqlite3.connect('enterprise.db')
>>> curs = conn.cursor()
>>> curs.execute('''CREATE TABLE zoo
 (critter VARCHAR(20) PRIMARY KEY,
 count INT,
 damages FLOAT)''')
<sqlite3.Cursor object at 0x1006a22d0>
```

Python 的三引号在创建长字符串（如 SQL 查询）时很方便。

现在，向动物园添加一些动物：

```py
>>> curs.execute('INSERT INTO zoo VALUES("duck", 5, 0.0)')
<sqlite3.Cursor object at 0x1006a22d0>
>>> curs.execute('INSERT INTO zoo VALUES("bear", 2, 1000.0)')
<sqlite3.Cursor object at 0x1006a22d0>
```

有一种更安全的方式可以插入数据，即使用*占位符*：

```py
>>> ins = 'INSERT INTO zoo (critter, count, damages) VALUES(?, ?, ?)'
>>> curs.execute(ins, ('weasel', 1, 2000.0))
<sqlite3.Cursor object at 0x1006a22d0>
```

这次，我们在 SQL 中使用了三个问号来表示我们打算插入三个值，然后将这三个值作为元组传递给 `execute()` 函数。占位符处理繁琐的细节，如引用。它们保护您免受*SQL 注入*的攻击，这是一种将恶意 SQL 命令插入系统的外部攻击（在网络上很常见）。

现在，让我们看看是否可以再次把我们所有的动物都放出来：

```py
>>> curs.execute('SELECT * FROM zoo')
<sqlite3.Cursor object at 0x1006a22d0>
>>> rows = curs.fetchall()
>>> print(rows)
[('duck', 5, 0.0), ('bear', 2, 1000.0), ('weasel', 1, 2000.0)]
```

让我们再次获取它们，但按计数排序：

```py
>>> curs.execute('SELECT * from zoo ORDER BY count')
<sqlite3.Cursor object at 0x1006a22d0>
>>> curs.fetchall()
[('weasel', 1, 2000.0), ('bear', 2, 1000.0), ('duck', 5, 0.0)]
```

嘿，我们希望它们按降序排列：

```py
>>> curs.execute('SELECT * from zoo ORDER BY count DESC')
<sqlite3.Cursor object at 0x1006a22d0>
>>> curs.fetchall()
[('duck', 5, 0.0), ('bear', 2, 1000.0), ('weasel', 1, 2000.0)]
```

哪种类型的动物给我们花费最多？

```py
>>> curs.execute('''SELECT * FROM zoo WHERE
...     damages = (SELECT MAX(damages) FROM zoo)''')
<sqlite3.Cursor object at 0x1006a22d0>
>>> curs.fetchall()
[('weasel', 1, 2000.0)]
```

你可能会认为是熊。最好检查一下实际数据。

在我们离开 SQLite 之前，我们需要清理一下。如果我们打开了连接和游标，那么在完成时我们需要关闭它们：

```py
>>> curs.close()
>>> conn.close()
```

## MySQL

[MySQL](http://www.mysql.com) 是一个非常流行的开源关系型数据库。与 SQLite 不同，它是一个实际的服务器，因此客户端可以从网络上的不同设备访问它。

表 16-3 列出了您可以使用的驱动程序，以从 Python 访问 MySQL。有关所有 Python MySQL 驱动程序的更多详细信息，请参见*python.org* [wiki](https://wiki.python.org/moin/MySQL)。

表 16-3\. MySQL 驱动程序

| Name | Link | Pypi package | Import as | Notes |
| --- | --- | --- | --- | --- |
| mysqlclient | [*https://https://mysqlclient.readthedocs.io*](https://mysqlclient.readthedocs.io/) | mysql-connector-python | `MySQLdb` |  |
| MySQL Connector | [*http://bit.ly/mysql-cpdg*](http://bit.ly/mysql-cpdg) | mysql-connector-python | `mysql.connector` |  |
| PYMySQL | [*https://github.com/petehunt/PyMySQL*](https://github.com/petehunt/PyMySQL/) | pymysql | `pymysql` |  |
| oursql | [*http://pythonhosted.org/oursql*](http://pythonhosted.org/oursql/) | oursql | `oursql` | 需要 MySQL C 客户端库 |

## PostgreSQL

[PostgreSQL](http://www.postgresql.org) 是一个功能齐全的开源关系型数据库。事实上，在许多方面，它比 MySQL 更先进。表 16-4 列出了您可以用来访问它的 Python 驱动程序。

表 16-4\. PostgreSQL 驱动程序

| Name | Link | Pypi package | Import as | Notes |
| --- | --- | --- | --- | --- |
| psycopg2 | [*http://initd.org/psycopg*](http://initd.org/psycopg/) | psycopg2 | psycopg2 | 需要来自 PostgreSQL 客户端工具的 `pg_config` |
| py-postgresql | [*https://pypi.org/project/py-postgresql*](https://pypi.org/project/py-postgresql/) | py-postgresql | postgresql |  |

最受欢迎的驱动程序是 `psycopg2`，但它的安装需要 PostgreSQL 客户端库。

## SQLAlchemy

对于所有关系数据库，SQL 并不完全相同，而 DB-API 也只能带你走到这一步。每个数据库都实现了一个反映其特性和哲学的特定*方言*。许多库试图以一种或另一种方式弥合这些差异。最流行的跨数据库 Python 库是 [SQLAlchemy](http://www.sqlalchemy.org)。

它不在标准库中，但它是众所周知的，并被许多人使用。你可以通过使用这个命令在你的系统上安装它：

```py
$ pip install sqlalchemy
```

你可以在几个层面上使用 SQLAlchemy：

+   最低级别管理数据库连接*池*，执行 SQL 命令并返回结果。这是最接近 DB-API 的层次。

+   接下来是 *SQL 表达式语言*，它允许你以更加面向 Python 的方式表达查询。

+   最高级别是 ORM（对象关系模型）层，它使用 SQL 表达语言并将应用程序代码与关系数据结构绑定。

随着我们的进行，你会理解在这些层次中术语的含义。SQLAlchemy 与前面章节中记录的数据库驱动程序一起使用。你不需要导入驱动程序；你提供给 SQLAlchemy 的初始连接字符串将决定它。这个字符串看起来像这样：

```py
*`dialect`* + *`driver`* :// *`user`* : *`password`* @ *`host`* : *`port`* / *`dbname`*
```

你在这个字符串中放入的值如下：

`dialect`

数据库类型

`driver`

你想要用于该数据库的特定驱动程序

`user` 和 `password`

您的数据库认证字符串

`host` 和 `port`

数据库服务器的位置（仅在不是该服务器的标准端口时需要）

`dbname`

最初连接到服务器的数据库

表 16-5 列出了方言和驱动程序。

表 16-5\. SQLAlchemy 连接

| 方言 | 驱动程序 |
| --- | --- |
| `sqlite` | `pysqlite`（或省略） |
| `mysql` | `mysqlconnector` |
| `mysql` | `pymysql` |
| `mysql` | `oursql` |
| `postgresql` | `psycopg2` |
| `postgresql` | `pypostgresql` |

另请参阅关于 [MySQL](https://oreil.ly/yVHy-)、[SQLite](https://oreil.ly/okP9v)、[PostgreSQL](https://oreil.ly/eDddn) 和 [其他数据库](https://oreil.ly/kp5WS) 的 SQLAlchemy 详细信息。

### 引擎层

首先，让我们尝试 SQLAlchemy 的最低级别，它只比基本的 DB-API 函数多做一些事情。

我们尝试使用内置到 Python 中的 SQLite。SQLite 的连接字符串省略了 *`host`*、*`port`*、*`user`* 和 *`password`*。*`dbname`* 告诉 SQLite 要使用哪个文件来存储你的数据库。如果省略 *`dbname`*，SQLite 将在内存中构建一个数据库。如果 *`dbname`* 以斜杠 (/) 开头，它是计算机上的绝对文件名（如在 Linux 和 macOS 中；例如，在 Windows 中是 `C:\`）。否则，它相对于当前目录。

下面的段落都属于一个程序的一部分，这里分开说明。

要开始，您需要导入所需内容。以下是一个 *导入别名* 的示例，它允许我们使用字符串 `sa` 来引用 SQLAlchemy 方法。我主要这样做是因为 `sa` 比 `sqlalchemy` 更容易输入：

```py
>>> import sqlalchemy as sa
```

连接到数据库并在内存中创建存储它的位置（参数字符串 `'sqlite:///:memory:'` 也适用）：

```py
>>> conn = sa.create_engine('sqlite://')
```

创建一个名为 `zoo` 的数据库表，包含三列：

```py
>>> conn.execute('''CREATE TABLE zoo
...     (critter VARCHAR(20) PRIMARY KEY,
...      count INT,
...      damages FLOAT)''')
<sqlalchemy.engine.result.ResultProxy object at 0x1017efb10>
```

运行 `conn.execute()` 会返回一个称为 `ResultProxy` 的 SQLAlchemy 对象。您很快就会看到如何处理它。

顺便说一句，如果您以前从未制作过数据库表，请恭喜。在您的待办清单中打勾。

现在，将三组数据插入到您的新空表中：

```py
>>> ins = 'INSERT INTO zoo (critter, count, damages) VALUES (?, ?, ?)'
>>> conn.execute(ins, 'duck', 10, 0.0)
<sqlalchemy.engine.result.ResultProxy object at 0x1017efb50>
>>> conn.execute(ins, 'bear', 2, 1000.0)
<sqlalchemy.engine.result.ResultProxy object at 0x1017ef090>
>>> conn.execute(ins, 'weasel', 1, 2000.0)
<sqlalchemy.engine.result.ResultProxy object at 0x1017ef450>
```

接下来，向数据库请求我们刚刚放入的所有内容：

```py
>>> rows = conn.execute('SELECT * FROM zoo')
```

在 SQLAlchemy 中，`rows` 不是一个列表；它是我们无法直接打印的那个特殊的 `ResultProxy` 东西：

```py
>>> print(rows)
<sqlalchemy.engine.result.ResultProxy object at 0x1017ef9d0>
```

但是，您可以像迭代列表一样迭代它，因此我们可以逐行获取：

```py
>>> for row in rows:
...     print(row)
...
('duck', 10, 0.0)
('bear', 2, 1000.0)
('weasel', 1, 2000.0)
```

这几乎与您之前看到的 SQLite DB-API 示例相同。一个优点是我们不需要在顶部导入数据库驱动程序；SQLAlchemy 从连接字符串中找到了这一点。只需更改连接字符串，即可将此代码移植到另一种类型的数据库。另一个优点是 SQLAlchemy 的 *连接池*，您可以在其 [文档站点](http://bit.ly/conn-pooling) 上阅读有关它的信息。

### SQL 表达式语言

上一级是 SQLAlchemy 的 SQL 表达式语言。它引入了函数来创建各种操作的 SQL。表达式语言处理的 SQL 方言差异比底层引擎层更多。对于关系数据库应用程序来说，它可以是一个方便的中间途径。

下面是如何创建和填充 `zoo` 表的方法。同样，这些是单个程序的连续片段。

导入和连接与之前相同：

```py
>>> import sqlalchemy as sa
>>> conn = sa.create_engine('sqlite://')
```

要定义 `zoo` 表，我们开始使用一些表达式语言，而不是 SQL：

```py
>>> meta = sa.MetaData()
>>> zoo = sa.Table('zoo', meta,
...     sa.Column('critter', sa.String, primary_key=True),
...     sa.Column('count', sa.Integer),
...     sa.Column('damages', sa.Float)
...    )
>>> meta.create_all(conn)
```

查看前面示例中多行调用中的括号。`Table()` 方法的结构与表的结构匹配。正如我们的表包含三列一样，在 `Table()` 方法调用的括号内有三次对 `Column()` 的调用。

与此同时，`zoo` 是一个魔术对象，连接了 SQL 数据库世界和 Python 数据结构世界。

使用更多表达式语言函数插入数据：

```py
... conn.execute(zoo.insert(('bear', 2, 1000.0)))
<sqlalchemy.engine.result.ResultProxy object at 0x1017ea910>
>>> conn.execute(zoo.insert(('weasel', 1, 2000.0)))
<sqlalchemy.engine.result.ResultProxy object at 0x1017eab10>
>>> conn.execute(zoo.insert(('duck', 10, 0)))
<sqlalchemy.engine.result.ResultProxy object at 0x1017eac50>
```

接下来，创建 SELECT 语句（`zoo.select()` 选择由 `zoo` 对象表示的表中的所有内容，就像在普通 SQL 中执行 `SELECT * FROM zoo` 一样）：

```py
>>> result = conn.execute(zoo.select())
```

最后，获取结果：

```py
>>> rows = result.fetchall()
>>> print(rows)
[('bear', 2, 1000.0), ('weasel', 1, 2000.0), ('duck', 10, 0.0)]
```

### 对象关系映射器（ORM）

在前一节中，`zoo`对象是 SQL 和 Python 之间的中间连接。在 SQLAlchemy 的顶层，对象关系映射器（ORM）使用 SQL 表达语言，但尝试使实际的数据库机制变得不可见。您定义类，ORM 处理如何将它们的数据进出数据库。复杂短语“对象关系映射器”的基本思想是，您可以在代码中引用对象，从而保持接近 Python 喜欢操作的方式，同时仍然使用关系数据库。

我们将定义一个`Zoo`类，并将其与 ORM 连接起来。这次，我们让 SQLite 使用文件*zoo.db*，以便确认 ORM 的工作。

与前两节类似，接下来的片段实际上是一个程序，由解释分隔开。如果您对某些内容不理解也不要担心。SQLAlchemy 文档中有所有细节——这些内容可能会变得复杂。我只是希望您了解做这件事情需要多少工作，这样您可以决定本章讨论的哪种方法最适合您。

初始导入是相同的，但这次我们还需要另外一些东西：

```py
>>> import sqlalchemy as sa
>>> from sqlalchemy.ext.declarative import declarative_base
```

在这里，我们建立连接：

```py
>>> conn = sa.create_engine('sqlite:///zoo.db')
```

现在，我们进入 SQLAlchemy 的 ORM。我们定义`Zoo`类，并关联其属性与表列：

```py
>>> Base = declarative_base()
>>> class Zoo(Base):
...     __tablename__ = 'zoo'
...     critter = sa.Column('critter', sa.String, primary_key=True)
...     count = sa.Column('count', sa.Integer)
...     damages = sa.Column('damages', sa.Float)
...     def __init__(self, critter, count, damages):
...         self.critter = critter
...         self.count = count
...         self.damages = damages
...     def __repr__(self):
...         return "<Zoo({}, {}, {})>".format(self.critter, self.count,
...           self.damages)
```

下面的代码神奇地创建了数据库和表：

```py
>>> Base.metadata.create_all(conn)
```

您可以通过创建 Python 对象来插入数据。ORM 在内部管理这些数据：

```py
>>> first = Zoo('duck', 10, 0.0)
>>> second = Zoo('bear', 2, 1000.0)
>>> third = Zoo('weasel', 1, 2000.0)
>>> first
<Zoo(duck, 10, 0.0)>
```

接下来，我们让 ORM 带我们进入 SQL 世界。我们创建一个会话来与数据库交互：

```py
>>> from sqlalchemy.orm import sessionmaker
>>> Session = sessionmaker(bind=conn)
>>> session = Session()
```

在会话中，我们将创建的三个对象写入数据库。`add()`函数添加一个对象，而`add_all()`添加一个列表：

```py
>>> session.add(first)
>>> session.add_all([second, third])
```

最后，我们需要强制完成所有操作：

```py
>>> session.commit()
```

它是否起作用？好吧，它在当前目录下创建了一个*zoo.db*文件。您可以使用命令行`sqlite3`程序来检查：

```py
$ sqlite3 zoo.db
SQLite version 3.6.12
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .tables
zoo
sqlite> select * from zoo;
duck|10|0.0
bear|2|1000.0
weasel|1|2000.0
```

本节的目的是展示 ORM 是什么以及它如何在高层次上工作。SQLAlchemy 的作者写了一个完整的[教程](http://bit.ly/obj-rel-tutorial)。阅读后，请决定以下哪种层次最适合您的需求：

+   就像之前的 SQLite 部分中的普通 DB-API 一样

+   SQLAlchemy 引擎

+   SQLAlchemy 表达语言

+   SQLAlchemy ORM

使用 ORM 似乎是避免 SQL 复杂性的自然选择。您应该使用 ORM 吗？有些人认为应该[避免](http://bit.ly/obj-rel-map)使用 ORM，但其他人认为这种批评是[过度的](http://bit.ly/fowler-orm)。不管谁是对的，ORM 都是一种抽象，所有抽象都会[泄漏](http://bit.ly/leaky-law)，并在某些时候出现问题。当 ORM 不能按您的意愿工作时，您必须弄清楚它的工作原理以及如何在 SQL 中修复它。借用互联网迷因：

> 有些人面对问题时会想：“我知道了，我会使用 ORM。”现在他们有两个问题。

对于简单的应用程序或将数据相当直接映射到数据库表的应用程序，请使用 ORM。如果应用程序如此简单，您可以考虑使用纯 SQL 或 SQL 表达式语言。

## 其他数据库访问包

如果您正在寻找能处理多个数据库的 Python 工具，具有比纯 db-api 更多功能但少于 SQLAlchemy 的功能，那么这些工具值得一看：

+   [`dataset`](https://dataset.readthedocs.org)声称其目标是“懒惰人的数据库”。它构建在 SQLAlchemy 之上，并为 SQL、JSON 和 CSV 存储提供了一个简单的 ORM。

+   [`records`](https://pypi.org/project/records)自称为“人类的 SQL”。它仅支持 SQL 查询，内部使用 SQLAlchemy 处理 SQL 方言问题、连接池和其他细节。它与`tablib`的集成（在“Tablib”中提到）允许您将数据导出为 CSV、JSON 和其他格式。

# NoSQL 数据存储

关系表是矩形的，但数据有多种形状，可能非常难以适应，需要进行大量努力和扭曲。这是一个方孔/圆孔问题。

一些非关系数据库已被编写，允许更灵活的数据定义，以及处理非常大的数据集或支持自定义数据操作。它们被统称为*NoSQL*（原意为*no SQL*，现在是更不具对抗性的*not only SQL*）。

最简单类型的 NoSQL 数据库是*键值存储*。一个流行的[排名](https://oreil.ly/_VCKq)展示了我在以下章节中涵盖的一些数据库。

## dbm 家族

`dbm`格式在*NoSQL*标签被创造之前就存在了。它们是简单的键值存储，通常嵌入在诸如 Web 浏览器之类的应用程序中，用于维护各种设置。dbm 数据库类似于 Python 字典的以下方面：

+   您可以为键分配一个值，并且它会自动保存到数据库中。

+   您可以查询键的值。

以下是一个快速示例。以下`open()`方法的第二个参数是`'r'`表示读取，`'w'`表示写入，`'c'`表示两者，如果文件不存在则创建：

```py
>>> import dbm
>>> db = dbm.open('definitions', 'c')
```

要创建键值对，只需像创建字典一样将值分配给键：

```py
>>> db['mustard'] = 'yellow'
>>> db['ketchup'] = 'red'
>>> db['pesto'] = 'green'
```

让我们停下来检查一下我们到目前为止所做的：

```py
>>> len(db)
3
>>> db['pesto']
b'green'
```

现在关闭，然后重新打开以查看它是否实际保存了我们给予的内容：

```py
>>> db.close()
>>> db = dbm.open('definitions', 'r')
>>> db['mustard']
b'yellow'
```

键和值被存储为`bytes`。您不能对数据库对象`db`进行迭代，但可以使用`len()`来获取键的数量。`get()`和`setdefault()`的工作方式与字典相同。

## Memcached

[`memcached`](http://memcached.org)是一个快速的内存中键值*缓存*服务器。它经常被放在数据库前面，或用于存储 Web 服务器会话数据。

您可以在[Linux 和 macOS](https://memcached.org/downloads)以及[Windows](http://bit.ly/memcache-win)下载版本。如果您想尝试本节，您需要一个正在运行的 memcached 服务器和 Python 驱动程序。

有许多 Python 驱动程序；与 Python 3 兼容的一个是 [`python3-memcached`](https://oreil.ly/7FA3-)，您可以使用以下命令安装它：

```py
$ pip install python-memcached
```

要使用它，请连接到 memcached 服务器，之后您可以执行以下操作：

+   为键设置和获取值

+   通过使用`incr`或`decr`增加或减少值

+   删除一个键

数据键和值*不*是持久的，之前写入的数据可能会消失。这是 memcached 的固有特性——它是一个缓存服务器，而不是数据库，并通过丢弃旧数据来避免内存耗尽。

您可以同时连接多个 memcached 服务器。在下一个示例中，我们仅与同一台计算机上的一个服务器通信：

```py
>>> import memcache
>>> db = memcache.Client(['127.0.0.1:11211'])
>>> db.set('marco', 'polo')
True
>>> db.get('marco')
'polo'
>>> db.set('ducks', 0)
True
>>> db.get('ducks')
0
>>> db.incr('ducks', 2)
2
>>> db.get('ducks')
2
```

## Redis

[Redis](http://redis.io)是一个*数据结构服务器*。它处理键及其值，但是与其他键值存储中的值相比，值更丰富。与 memcached 一样，Redis 服务器中的所有数据都应该适合内存。不同于 memcached，Redis 可以执行以下操作：

+   将数据保存到磁盘以保证可靠性和重新启动

+   保留旧数据

+   提供比简单字符串更多的数据结构

Redis 的数据类型与 Python 的非常接近，并且 Redis 服务器可以成为一个或多个 Python 应用程序共享数据的有用中介。我发现它非常有用，因此在这里额外进行一些覆盖是值得的。

Python 驱动程序 `redis-py` 在 [GitHub](https://oreil.ly/aZIbQ) 上有其源代码和测试，以及 [文档](http://bit.ly/redis-py-docs)。您可以使用以下命令安装它：

```py
$ pip install redis
```

[Redis 服务器](http://redis.io)有很好的文档。如果在本地计算机上安装并启动 Redis 服务器（使用网络别名`localhost`），您可以尝试以下部分的程序。

### 字符串

一个具有单个值的键是 Redis *字符串*。简单的 Python 数据类型会自动转换。连接到某个主机上的 Redis 服务器（默认为`localhost`）和端口（默认为`6379`）：

```py
>>> import redis
>>> conn = redis.Redis()
```

连接到`redis.Redis('localhost')`或`redis.Redis('localhost', 6379)`将给出相同的结果。

列出所有键（目前没有）：

```py
>>> conn.keys('*')
[]
```

设置一个简单的字符串（键`'secret'`）、整数（键`'carats'`）和浮点数（键`'fever'`）：

```py
>>> conn.set('secret', 'ni!')
True
>>> conn.set('carats', 24)
True
>>> conn.set('fever', '101.5')
True
```

通过键获取值（作为 Python `byte` 值）：

```py
>>> conn.get('secret')
b'ni!'
>>> conn.get('carats')
b'24'
>>> conn.get('fever')
b'101.5'
```

在这里，`setnx()`方法仅在键不存在时设置一个值：

```py
>>> conn.setnx('secret', 'icky-icky-icky-ptang-zoop-boing!')
False
```

因为我们已经定义了`'secret'`，所以失败了：

```py
>>> conn.get('secret')
b'ni!'
```

`getset()`方法返回旧值，并同时设置为新值：

```py
>>> conn.getset('secret', 'icky-icky-icky-ptang-zoop-boing!')
b'ni!'
```

不要急于前进。这有用吗？

```py
>>> conn.get('secret')
b'icky-icky-icky-ptang-zoop-boing!'
```

现在，使用`getrange()`获取子字符串（与 Python 中一样，偏移量`0`表示起始，`-1`表示结尾）：

```py
>>> conn.getrange('secret', -6, -1)
b'boing!'
```

使用`setrange()`替换子字符串（使用从零开始的偏移量）：

```py
>>> conn.setrange('secret', 0, 'ICKY')
32
>>> conn.get('secret')
b'ICKY-icky-icky-ptang-zoop-boing!'
```

接下来，使用`mset()`一次设置多个键：

```py
>>> conn.mset({'pie': 'cherry', 'cordial': 'sherry'})
True
```

通过使用`mget()`一次获取多个值：

```py
>>> conn.mget(['fever', 'carats'])
[b'101.5', b'24']
```

使用`delete()`方法删除一个键：

```py
>>> conn.delete('fever')
True
```

使用`incr()`或`incrbyfloat()`命令进行增量，并使用`decr()`进行减量：

```py
>>> conn.incr('carats')
25
>>> conn.incr('carats', 10)
35
>>> conn.decr('carats')
34
>>> conn.decr('carats', 15)
19
>>> conn.set('fever', '101.5')
True
>>> conn.incrbyfloat('fever')
102.5
>>> conn.incrbyfloat('fever', 0.5)
103.0
```

没有`decrbyfloat()`。使用负增量来减少发烧：

```py
>>> conn.incrbyfloat('fever', -2.0)
101.0
```

### 列表

Redis 列表只能包含字符串。当您进行第一次插入时，列表被创建。通过使用`lpush()`在开头插入：

```py
>>> conn.lpush('zoo', 'bear')
1
```

在开头插入多个项目：

```py
>>> conn.lpush('zoo', 'alligator', 'duck')
3
```

通过使用`linsert()`在值之前或之后插入：

```py
>>> conn.linsert('zoo', 'before', 'bear', 'beaver')
4
>>> conn.linsert('zoo', 'after', 'bear', 'cassowary')
5
```

通过使用`lset()`在偏移处插入（列表必须已经存在）：

```py
>>> conn.lset('zoo', 2, 'marmoset')
True
```

通过使用`rpush()`在末尾插入：

```py
>>> conn.rpush('zoo', 'yak')
6
```

通过使用`lindex()`按偏移量获取值：

```py
>>> conn.lindex('zoo', 3)
b'bear'
```

通过使用`lrange()`获取偏移范围内的值（0 到-1 获取所有）：

```py
>>> conn.lrange('zoo', 0, 2)
[b'duck', b'alligator', b'marmoset']
```

使用`ltrim()`修剪列表，仅保留偏移范围内的元素：

```py
>>> conn.ltrim('zoo', 1, 4)
True
```

通过使用`lrange()`获取值的范围（使用`0`到`-1`获取所有）：

```py
>>> conn.lrange('zoo', 0, -1)
[b'alligator', b'marmoset', b'bear', b'cassowary']
```

第十五章展示了如何使用 Redis 列表和*发布-订阅*来实现作业队列。

### 哈希

Redis *哈希*类似于 Python 字典，但只能包含字符串。此外，您只能深入到一级，不能创建深度嵌套的结构。以下是创建和操作名为`song`的 Redis 哈希的示例：

通过使用`hmset()`一次设置哈希`song`中的字段`do`和`re`：

```py
>>> conn.hmset('song', {'do': 'a deer', 're': 'about a deer'})
True
```

通过使用`hset()`在哈希中设置单个字段值：

```py
>>> conn.hset('song', 'mi', 'a note to follow re')
1
```

通过使用`hget()`获取一个字段的值：

```py
>>> conn.hget('song', 'mi')
b'a note to follow re'
```

通过使用`hmget()`获取多个字段值：

```py
>>> conn.hmget('song', 're', 'do')
[b'about a deer', b'a deer']
```

通过使用`hkeys()`获取哈希的所有字段键：

```py
>>> conn.hkeys('song')
[b'do', b're', b'mi']
```

通过使用`hvals()`获取哈希的所有字段值：

```py
>>> conn.hvals('song')
[b'a deer', b'about a deer', b'a note to follow re']
```

通过使用`hlen()`获取哈希中字段的数量：

```py
>>> conn.hlen('song')
3
```

通过使用`hgetall()`获取哈希中的所有字段键和值：

```py
>>> conn.hgetall('song')
{b'do': b'a deer', b're': b'about a deer', b'mi': b'a note to follow re'}
```

如果其键不存在，则使用`hsetnx()`设置字段：

```py
>>> conn.hsetnx('song', 'fa', 'a note that rhymes with la')
1
```

### 集合

Redis 集合与 Python 集合类似，您将在以下示例中看到。

向集合添加一个或多个值：

```py
>>> conn.sadd('zoo', 'duck', 'goat', 'turkey')
3
```

获取集合的值的数量：

```py
>>> conn.scard('zoo')
3
```

获取集合的所有值：

```py
>>> conn.smembers('zoo')
{b'duck', b'goat', b'turkey'}
```

从集合中移除一个值：

```py
>>> conn.srem('zoo', 'turkey')
True
```

让我们再建立一个集合来展示一些集合操作：

```py
>>> conn.sadd('better_zoo', 'tiger', 'wolf', 'duck')
0
```

求交集（获取`zoo`和`better_zoo`集合的共同成员）：

```py
>>> conn.sinter('zoo', 'better_zoo')
{b'duck'}
```

获取`zoo`和`better_zoo`的交集，并将结果存储在集合`fowl_zoo`中：

```py
>>> conn.sinterstore('fowl_zoo', 'zoo', 'better_zoo')
1
```

里面有谁？

```py
>>> conn.smembers('fowl_zoo')
{b'duck'}
```

获取`zoo`和`better_zoo`的并集（所有成员）：

```py
>>> conn.sunion('zoo', 'better_zoo')
{b'duck', b'goat', b'wolf', b'tiger'}
```

将联合结果存储在集合`fabulous_zoo`中：

```py
>>> conn.sunionstore('fabulous_zoo', 'zoo', 'better_zoo')
4
>>> conn.smembers('fabulous_zoo')
{b'duck', b'goat', b'wolf', b'tiger'}
```

`zoo`有什么，`better_zoo`没有？使用`sdiff()`获取集合的差集，并使用`sdiffstore()`将其保存在`zoo_sale`集合中：

```py
>>> conn.sdiff('zoo', 'better_zoo')
{b'goat'}
>>> conn.sdiffstore('zoo_sale', 'zoo', 'better_zoo')
1
>>> conn.smembers('zoo_sale')
{b'goat'}
```

### 排序集合

最多用途的 Redis 数据类型之一是*排序集合*，或*zset*。它是一组唯一值，但每个值都有一个关联的浮点数*分数*。您可以通过其值或分数访问每个项目。排序集合有许多用途：

+   排行榜

+   二级索引

+   时间序列，使用时间戳作为分数

我们展示了最后一个用例，通过时间戳跟踪用户登录。我们使用 Python `time()`函数返回的 Unix *epoch*值（更多详情请参见第十五章）：

```py
>>> import time
>>> now = time.time()
>>> now
1361857057.576483
```

让我们添加第一个看起来紧张的客人：

```py
>>> conn.zadd('logins', 'smeagol', now)
1
```

五分钟后，另一位客人：

```py
>>> conn.zadd('logins', 'sauron', now+(5*60))
1
```

两小时后：

```py
>>> conn.zadd('logins', 'bilbo', now+(2*60*60))
1
```

一天后，不要着急：

```py
>>> conn.zadd('logins', 'treebeard', now+(24*60*60))
1
```

`bilbo`以什么顺序到达？

```py
>>> conn.zrank('logins', 'bilbo')
2
```

那是什么时候？

```py
>>> conn.zscore('logins', 'bilbo')
1361864257.576483
```

让我们按登录顺序查看每个人：

```py
>>> conn.zrange('logins', 0, -1)
[b'smeagol', b'sauron', b'bilbo', b'treebeard']
```

请附上它们的时间：

```py
>>> conn.zrange('logins', 0, -1, withscores=True)
[(b'smeagol', 1361857057.576483), (b'sauron', 1361857357.576483),
(b'bilbo', 1361864257.576483), (b'treebeard', 1361943457.576483)]
```

### 缓存和过期

所有的 Redis 键都有一个*到期时间*，默认情况下是永久的。我们可以使用`expire()`函数来指示 Redis 保留键的时间长度。该值是以秒为单位的数字：

```py
>>> import time
>>> key = 'now you see it'
>>> conn.set(key, 'but not for long')
True
>>> conn.expire(key, 5)
True
>>> conn.ttl(key)
5
>>> conn.get(key)
b'but not for long'
>>> time.sleep(6)
>>> conn.get(key)
>>>
```

`expireat()`命令在给定的纪元时间点过期键。键的过期对于保持缓存新鲜和限制登录会话非常有用。类比：在你的杂货店货架后面的冷藏室中，当牛奶达到保质期时，店员们就会将那些加仑装出货。

## 文档数据库

*文档数据库*是一种 NoSQL 数据库，它以不同的字段存储数据。与关系表（每行具有相同列的矩形表）相比，这样的数据是“参差不齐”的，每行具有不同的字段（列），甚至是嵌套字段。你可以使用 Python 字典和列表在内存中处理这样的数据，或者将其存储为 JSON 文件。要将这样的数据存储在关系数据库表中，你需要定义每个可能的列，并使用 null 来表示缺失的数据。

*ODM*可以代表对象数据管理器或对象文档映射器（至少它们同意“O”部分）。ODM 是文档数据库的关系数据库 ORM 对应物。一些[流行的](https://oreil.ly/5Zpxx)文档数据库和工具（驱动程序和 ODM）列在表 16-6 中。 

表 16-6\. 文档数据库

| 数据库 | Python API |
| --- | --- |
| [Mongo](https://www.mongodb.com) | [tools](https://api.mongodb.com/python/current/tools.html) |
| [DynamoDB](https://aws.amazon.com/dynamodb) | [`boto3`](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStarted.Python.html) |
| [CouchDB](http://couchdb.apache.org) | [`couchdb`](https://couchdb-python.readthedocs.io/en/latest/index.html) |

###### 注意

PostgreSQL 可以做一些文档数据库可以做的事情。它的一些扩展允许它逃离关系规范，同时保留事务、数据验证和外键等特性：1）多维[*数组*](https://oreil.ly/MkfLY) - 在表单元格中存储多个值；2）[*jsonb*](https://oreil.ly/K_VJg) - 在单元格中存储 JSON 数据，并进行完整的索引和查询。

## 时间序列数据库

*时间序列*数据可以在固定间隔（如计算机性能指标）或随机时间收集，这导致了许多存储方法。其中[许多](https://oreil.ly/CkjC0)中[的一些](https://oreil.ly/IbOxQ)，一些具有 Python 支持的方法列在表 16-7 中。

表 16-7\. 时间数据库

| 数据库 | Python API |
| --- | --- |
| [InfluxDB](https://www.influxdata.com) | [`influx-client`](https://pypi.org/project/influx-client) |
| [kdb+](https://kx.com) | [PyQ](https://code.kx.com/v2/interfaces/pyq/) |
| [Prometheus](https://prometheus.io) | [`prometheus_client`](https://github.com/prometheus/client_python/blob/master/README.md) |
| [TimescaleDB](https://www.timescale.com) | (PostgreSQL clients) |
| [OpenTSDB](http://opentsdb.net) | [`potsdb`](https://pypi.org/project/potsdb) |
| [PyStore](https://github.com/ranaroussi/pystore) | [`PyStore`](https://pypi.org/project/PyStore) |

## 图数据库

对于需要有自己数据库类别的最后一种数据案例，我们有 *图形*：*节点*（数据）通过 *边缘* 或 *顶点*（关系）相连。一个个体 Twitter *用户* 可以是一个节点，与其他用户的关系如 *关注* 和 *被关注* 为边。

随着社交媒体的增长，图形数据变得更加明显，价值在于连接而非内容本身。一些流行的图数据库在 表 16-8 中概述。

表 16-8\. 图数据库

| 数据库 | Python API |
| --- | --- |
| [Neo4J](https://neo4j.com) | [`py2neo`](https://py2neo.org/v3) |
| [OrientDB](https://orientdb.com) | [`pyorient`](https://orientdb.com/docs/last/PyOrient.html) |
| [ArangoDB](https://www.arangodb.com) | [`pyArango`](https://github.com/ArangoDB-Community/pyArango) |

## 其他 NoSQL

这里列出的 NoSQL 服务器处理比内存更大的数据，并且许多使用多台计算机。表 16-9 展示了显著的服务器及其 Python 库。

表 16-9\. NoSQL 数据库

| 数据库 | Python API |
| --- | --- |
| [Cassandra](http://cassandra.apache.org) | [`pycassa`](https://github.com/pycassa/pycassa) |
| [CouchDB](http://couchdb.apache.org) | [`couchdb-python`](https://github.com/djc/couchdb-python) |
| [HBase](http://hbase.apache.org) | [`happybase`](https://github.com/wbolster/happybase) |
| [Kyoto Cabinet](http://fallabs.com/kyotocabinet) | [`kyotocabinet`](http://bit.ly/kyotocabinet) |
| [MongoDB](http://www.mongodb.org) | [`mongodb`](http://api.mongodb.org/python/current) |
| [Pilosa](https://www.pilosa.com) | [`python-pilosa`](https://github.com/pilosa/python-pilosa) |
| [Riak](http://basho.com/riak) | [`riak-python-client`](https://github.com/basho/riak-python-client) |

# 全文搜索数据库

最后，还有一个专门用于 *全文* 搜索的数据库类别。它们索引所有内容，因此您可以找到那些讲述风车和巨大芝士轮的诗歌。您可以在 表 16-10 中看到一些流行的开源示例及其 Python API。

表 16-10\. 全文搜索数据库

| 网站 | Python API |
| --- | --- |
| [Lucene](http://lucene.apache.org) | [`pylucene`](http://lucene.apache.org/pylucene) |
| [Solr](http://lucene.apache.org/solr) | [`SolPython`](http://wiki.apache.org/solr/SolPython) |
| [ElasticSearch](http://www.elasticsearch.org) | [`elasticsearch`](https://elasticsearch-py.readthedocs.io) |
| [Sphinx](http://sphinxsearch.com) | [`sphinxapi`](http://bit.ly/sphinxapi) |
| [Xapian](http://xapian.org) | [`xappy`](https://code.google.com/p/xappy) |
| [Whoosh](http://bit.ly/mchaput-whoosh) | （用 Python 编写，包含 API） |

# 即将出现

前一章讨论了在时间上交错使用代码（*并发*）。接下来的章节将介绍如何在空间中移动数据（*网络*），这不仅可以用于并发，还有其他用途。

# 待办事项

16.1 将以下文本行保存到名为 *books.csv* 的文件中（注意，如果字段由逗号分隔，如果字段包含逗号，你需要用引号括起来）：

```py
author,book
J R R Tolkien,The Hobbit
Lynne Truss,"Eats, Shoots & Leaves"
```

16.2 使用 `csv` 模块及其 `DictReader` 方法读取 *books.csv* 并存入变量 `books`。打印 `books` 中的值。`DictReader` 是否处理了第二本书标题中的引号和逗号？

16.3 使用以下代码创建一个名为 *books2.csv* 的 CSV 文件：

```py
title,author,year
The Weirdstone of Brisingamen,Alan Garner,1960
Perdido Street Station,China Miéville,2000
Thud!,Terry Pratchett,2005
The Spellman Files,Lisa Lutz,2007
Small Gods,Terry Pratchett,1992
```

16.4 使用 `sqlite3` 模块创建一个名为 *books.db* 的 SQLite 数据库，并创建一个名为 `books` 的表，具有以下字段：`title`（文本）、`author`（文本）和 `year`（整数）。

16.5 读取 *books2.csv* 并将其数据插入到 `book` 表中。

16.6 按字母顺序选择并打印 `book` 表中的 `title` 列。

16.7 按照出版顺序选择并打印 `book` 表中的所有列。

16.8 使用 `sqlalchemy` 模块连接到你在练习 16.4 中刚刚创建的 sqlite3 数据库 *books.db*。如同 16.6，按字母顺序选择并打印 `book` 表中的 `title` 列。

16.9 在你的计算机上安装 Redis 服务器和 Python `redis` 库（`pip install redis`）。创建一个名为 `test` 的 Redis 哈希，具有字段 `count`（`1`）和 `name`（`'Fester Bestertester'`）。打印 `test` 的所有字段。

16.10 增加 `test` 的 `count` 字段并打印它。

¹ 啊，还没到 XML 的时候。
