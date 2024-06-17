# 第十二章：整理和处理数据

> 如果你折磨数据足够久，自然会招认。
> 
> 罗纳德·科斯

到目前为止，我们主要讨论了 Python 语言本身——其数据类型、代码结构、语法等等。本书的其余部分是关于将这些应用到现实世界问题的内容。

在本章中，你将学习许多实用的数据处理技术。有时，这被称为*数据清洗*，或者更商业化的*ETL*（提取/转换/加载）在数据库世界中。尽管编程书籍通常不会明确涵盖这个主题，程序员们花费了大量时间来将数据塑造成符合其目的的正确形式。

*数据科学*这一专业在过去几年变得非常流行。《哈佛商业评论》的一篇文章称数据科学家是“21 世纪最性感的职业”。如果这意味着需求量大且薪资丰厚，那就好，但也有足够的单调乏味。数据科学超越了数据库的 ETL 需求，通常涉及*机器学习*，以发掘人眼看不到的洞察力。

我将从基本的数据格式开始，然后介绍最有用的新数据科学工具。

数据格式大致分为两类：*文本*和*二进制*。Python 的*字符串*用于文本数据，本章包含了我们迄今为止跳过的字符串信息：

+   *Unicode*字符

+   *正则表达式*模式匹配。

然后，我们转向二进制数据，以及 Python 的另外两种内置类型：

+   *字节*用于不可变的八位值

+   *字节数组*用于可变的字节

# 文本字符串：Unicode

你在第五章中看到了 Python 字符串的基础知识。现在是深入了解 Unicode 的时候了。

Python 3 的字符串是 Unicode 字符序列，而不是字节数组。这是从 Python 2 最大的语言变化。

到目前为止，本书中的所有文本示例都是普通的 ASCII（美国标准信息交换码）。ASCII 是在六十年代定义的，在鲑鱼头发流行之前。当时的计算机大小如冰箱，稍微聪明一点。

计算机存储的基本单元是*字节*，它可以存储 256 个独特的值在它的八*位*中。出于各种原因，ASCII 只使用了七位（128 个独特的值）：26 个大写字母、26 个小写字母、10 个数字、一些标点符号、一些间隔字符和一些不可打印的控制码。

不幸的是，世界上的字母比 ASCII 提供的还要多。你可以在餐馆吃热狗，但在咖啡馆永远也买不到 Gewürztraminer¹。已经尝试过许多方法将更多的字母和符号塞入八位中，有时你会看到它们。其中只有一些包括：

+   *Latin-1*，或者*ISO 8859-1*

+   Windows 代码页*1252*

这些字符都使用了所有的八位，但即使如此也不够，特别是在需要非欧洲语言时。*Unicode* 是一个持续进行的国际标准，用于定义所有世界语言的字符，以及数学和其他领域的符号。还有表情符号！

> Unicode 为每个字符提供了一个唯一的编号，无论是什么平台、什么程序、什么语言。
> 
> Unicode 联盟

[Unicode 代码图表页面](http://www.unicode.org/charts)包含所有当前定义的字符集的链接及其图像。最新版本（12.0）定义了超过 137,000 个字符，每个字符都有唯一的名称和标识号码。Python 3.8 可以处理所有这些字符。这些字符被分为称为*平面*的八位集合。前 256 个平面是*基本多语言平面*。详细信息请参阅关于[Unicode 平面](http://bit.ly/unicode-plane)的维基百科页面。

## Python 3 Unicode 字符串

如果您知道字符的 Unicode ID 或名称，可以在 Python 字符串中使用它。以下是一些示例：

+   `\u`后跟*四*个十六进制数字²指定 Unicode 的 256 个基本多语言平面中的一个字符。前两个数字是平面编号（`00`到`FF`），后两个数字是平面内字符的索引。平面`00`是老旧的 ASCII，该平面内的字符位置与 ASCII 相同。

+   对于高平面中的字符，我们需要更多的位数。Python 中这些字符的转义序列是`\U`后跟*八*个十六进制字符；最左边的数字需要是`0`。

+   对于所有字符，``\N{*`name`*}`` 允许您通过其标准*名称*指定它。[Unicode 字符名称索引页面](http://www.unicode.org/charts/charindex.html)列出了这些名称。

Python 的 `unicodedata` 模块具有双向转换的功能：

+   `lookup()`—接受一个不区分大小写的名称，并返回一个 Unicode 字符。

+   `name()`—接受一个 Unicode 字符并返回其大写名称。

在下面的示例中，我们将编写一个测试函数，该函数接受一个 Python Unicode 字符，查找其名称，然后根据名称再次查找字符（应该与原始字符匹配）：

```py
>>> def unicode_test(value):
...     import unicodedata
...     name = unicodedata.name(value)
...     value2 = unicodedata.lookup(name)
...     print('value="%s", name="%s", value2="%s"' % (value, name, value2))
...
```

让我们尝试一些字符，首先是一个普通的 ASCII 字母：

```py
>>> unicode_test('A')
value="A", name="LATIN CAPITAL LETTER A", value2="A"
```

ASCII 标点符号：

```py
>>> unicode_test('$')
value="$", name="DOLLAR SIGN", value2="$"
```

Unicode 货币字符：

```py
>>> unicode_test('\u00a2')
value="¢", name="CENT SIGN", value2="¢"
```

另一个 Unicode 货币字符：

```py
>>> unicode_test('\u20ac')
value="€", name="EURO SIGN", value2="€"
```

你可能遇到的唯一问题是字体显示文本的限制。很少有字体包含所有 Unicode 字符的图像，可能会为缺失的字符显示一些占位符字符。例如，这是 `SNOWMAN` 的 Unicode 符号，类似于装饰符字体中的符号：

```py
>>> unicode_test('\u2603')
value="☃", name="SNOWMAN", value2="☃"
```

假设我们想要在 Python 字符串中保存单词 `café`。一种方法是从文件或网站复制并粘贴它，然后希望它能正常工作：

```py
>>> place = 'café'
>>> place
'café'
```

这有效是因为我从使用 UTF-8 编码的源复制和粘贴了文本。

如何指定最后的 `é` 字符？如果你查看 [E](http://bit.ly/e-index) 的字符索引，你会看到名称 `E WITH ACUTE,` `LATIN SMALL LETTER` 具有值 `00E9`。让我们用刚才玩过的 `name()` 和 `lookup()` 函数来检查。首先给出获取名称的代码：

```py
>>> unicodedata.name('\u00e9')
'LATIN SMALL LETTER E WITH ACUTE'
```

接下来，给出查找代码的名称：

```py
>>> unicodedata.lookup('E WITH ACUTE, LATIN SMALL LETTER')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: "undefined character name 'E WITH ACUTE, LATIN SMALL LETTER'"
```

###### 注意

Unicode 字符名称索引页上列出的名称已经重新格式化，使其在显示时可以很好地排序。要将它们转换为真实的 Unicode 名称（Python 使用的名称），去掉逗号并将逗号后面的部分移动到开头。因此，将 `E WITH ACUTE, LATIN SMALL LETTER` 改为 `LATIN SMALL LETTER E WITH ACUTE`：

```py
>>> unicodedata.lookup('LATIN SMALL LETTER E WITH ACUTE')
'é'
```

现在我们可以通过代码或名称指定字符串 `café`：

```py
>>> place = 'caf\u00e9'
>>> place
'café'
>>> place = 'caf\N{LATIN SMALL LETTER E WITH ACUTE}'
>>> place
'café'
```

在前面的代码片段中，我们直接在字符串中插入了 é，但我们也可以通过附加构建字符串：

```py
>>> u_umlaut = '\N{LATIN SMALL LETTER U WITH DIAERESIS}'
>>> u_umlaut
'ü'
>>> drink = 'Gew' + u_umlaut + 'rztraminer'
>>> print('Now I can finally have my', drink, 'in a', place)
Now I can finally have my Gewürztraminer in a café
```

字符串 `len()` 函数计算 Unicode *字符* 数量，而不是字节数：

```py
>>> len('$')
1
>>> len('\U0001f47b')
1
```

###### 注意

如果你知道 Unicode 的数值 ID，你可以使用标准的 `ord()` 和 `chr()` 函数快速转换整数 ID 和单字符 Unicode 字符串：

```py
>>> chr(233)
'é'
>>> chr(0xe9)
'é'
>>> chr(0x1fc6)
'ῆ'
```

## UTF-8

在正常的字符串处理中，你不需要担心 Python 如何存储每个 Unicode 字符。

但是，当你与外界交换数据时，你需要一些东西：

+   *编码* 字符字符串为字节的方法

+   *解码* 字节到字符字符串的方法

如果 Unicode 中的字符少于 65,536 个，我们可以将每个 Unicode 字符 ID 塞入两个字节中。不幸的是，字符太多了。我们可以将每个 ID 编码为四个字节，但这将使常见文本字符串的内存和磁盘存储空间需求增加四倍。

Ken Thompson 和 Rob Pike，Unix 开发者熟悉的名字，设计了一夜之间在新泽西餐馆的餐垫上的 *UTF-8* 动态编码方案。它每个 Unicode 字符使用一到四个字节：

+   ASCII 占一个字节

+   大多数拉丁衍生（但不包括西里尔语）语言需要两个字节

+   基本多语言平面的其余部分需要三个字节

+   其余部分包括一些亚洲语言和符号需要四个字节

UTF-8 是 Python、Linux 和 HTML 中的标准文本编码。它快速、全面且运行良好。如果你在代码中始终使用 UTF-8 编码，生活将比试图在各种编码之间跳转要容易得多。

###### 注意

如果你从网页等其他源复制粘贴创建 Python 字符串，请确保源以 UTF-8 格式编码。经常看到将以 Latin-1 或 Windows 1252 编码的文本复制到 Python 字符串中，这将导致后来出现无效字节序列的异常。

## 编码

你可以将字符串 *编码* 为 *字节*。字符串 `encode()` 函数的第一个参数是编码名称。选择包括 [Table 12-1](https://bit.ly/table_12-1) 中的那些。

表 12-1\. 编码

| 编码名称 | 描述 |
| --- | --- |
| `'ascii'` | 七比特 ASCII 编码 |
| `'utf-8'` | 八位变长编码，几乎总是你想要使用的 |
| `'latin-1'` | 也称为 ISO 8859-1 |
| `'cp-1252'` | 常见的 Windows 编码 |
| `'unicode-escape'` | Python Unicode 文本格式，`\u`*xxxx* 或 `\U`*xxxxxxxx* |

你可以将任何东西编码为 UTF-8。让我们将 Unicode 字符串`'\u2603'`赋给名称`snowman`：

```py
>>> snowman = '\u2603'
```

`snowman`是一个 Python Unicode 字符串，只有一个字符，无论内部存储它需要多少字节：

```py
>>> len(snowman)
1
```

接下来，让我们将这个 Unicode 字符编码为一个字节序列：

```py
>>> ds = snowman.encode('utf-8')
```

正如我之前提到的，UTF-8 是一种变长编码。在这种情况下，它用三个字节来编码单个`snowman` Unicode 字符：

```py
>>> len(ds)
3
>>> ds
b'\xe2\x98\x83'
```

现在，`len()`返回字节数（3），因为`ds`是一个`bytes`变量。

你可以使用除 UTF-8 之外的其他编码，但如果 Unicode 字符串不能被处理，你会得到错误。例如，如果你使用`ascii`编码，除非你的 Unicode 字符恰好是有效的 ASCII 字符，否则会失败：

```py
>>> ds = snowman.encode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode character '\u2603'
in position 0: ordinal not in range(128)
```

`encode()`函数接受第二个参数，帮助你避免编码异常。它的默认值，在前面的例子中你可以看到，是`'strict'`；如果遇到非 ASCII 字符，它会引发一个`UnicodeEncodeError`。还有其他编码方式。使用`'ignore'`来丢弃任何无法编码的内容：

```py
>>> snowman.encode('ascii', 'ignore')
b''
```

使用`'replace'`来用`?`替换未知字符：

```py
>>> snowman.encode('ascii', 'replace')
b'?'
```

使用`'backslashreplace'`来生成一个 Python Unicode 字符字符串，比如`unicode-escape`：

```py
>>> snowman.encode('ascii', 'backslashreplace')
b'\\u2603'
```

如果你需要一个 Unicode 转义序列的可打印版本，你可以使用这个方法。

使用`'xmlcharrefreplace'`来生成 HTML 安全字符串：

```py
>>> snowman.encode('ascii', 'xmlcharrefreplace')
b'&#9731;'
```

我在“HTML 实体”中提供了更多 HTML 转换的细节。

## 解码

我们将字节字符串*解码*为 Unicode 文本字符串。每当我们从外部来源（文件、数据库、网站、网络 API 等）获取文本时，它都会被编码为字节字符串。棘手的部分是知道实际使用了哪种编码方式，这样我们才能*逆向操作*并获取 Unicode 字符串。

问题在于字节字符串本身没有说明使用了哪种编码。我之前提到过从网站复制粘贴的危险。你可能访问过一些奇怪字符的网站，本应是普通的 ASCII 字符。

让我们创建一个名为`place`的 Unicode 字符串，其值为`'café'`：

```py
>>> place = 'caf\u00e9'
>>> place
'café'
>>> type(place)
<class 'str'>
```

用 UTF-8 格式编码，存入一个名为`place_bytes`的`bytes`变量：

```py
>>> place_bytes = place.encode('utf-8')
>>> place_bytes
b'caf\xc3\xa9'
>>> type(place_bytes)
<class 'bytes'>
```

注意`place_bytes`有五个字节。前三个与 ASCII 相同（UTF-8 的优势），最后两个编码了`'é'`。现在让我们将该字节字符串解码回 Unicode 字符串：

```py
>>> place2 = place_bytes.decode('utf-8')
>>> place2
'café'
```

这个方法的原因是我们编码为 UTF-8 并解码为 UTF-8。如果我们告诉它从其他编码解码会怎样呢？

```py
>>> place3 = place_bytes.decode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position 3:
ordinal not in range(128)
```

ASCII 解码器因为字节值`0xc3`在 ASCII 中是非法的而抛出了异常。有些 8 位字符集编码允许 128（十六进制`80`）到 255（十六进制`FF`）之间的值合法，但与 UTF-8 不同。

```py
>>> place4 = place_bytes.decode('latin-1')
>>> place4
'cafÃ©'
>>> place5 = place_bytes.decode('windows-1252')
>>> place5
'cafÃ©'
```

唔。

这个故事的教训是：只要可能，请使用 UTF-8 编码。它适用，被到处支持，可以表示每个 Unicode 字符，并且快速解码和编码。

###### 注意

尽管您可以指定任何 Unicode 字符，这并不意味着您的计算机将显示所有这些字符。这取决于您使用的*字体*，该字体可能对许多字符显示空白或填充图像。苹果为 Unicode 联盟创建了[最后的应急字体](https://oreil.ly/q5EZD)，并在其自己的操作系统中使用它。这个[Wikipedia 页面](https://oreil.ly/Zm_uZ)有更多细节。另一种包含从`\u0000`到`\uffff`以及更多字符的字体是[Unifont](https://oreil.ly/APKlj)。

## HTML 实体

Python 3.4 增加了另一种转换 Unicode 的方法，但是使用 HTML *字符实体*。³ 这可能比查找 Unicode 名称更容易，特别是在您在网络上工作时：

```py
>>> import html
>>> html.unescape("&egrave;")
'è'
```

这种转换也适用于编号实体，十进制或十六进制：

```py
>>> import html
>>> html.unescape("&#233;")
'é'
>>> html.unescape("&#xe9;")
'é'
```

您甚至可以将命名实体转换导入为字典并自行进行转换。删除字典键的初始`'&'`（您也可以删除最后的`;`，但似乎两种方式都可以工作）：

```py
>>> from html.entities import html5
>>> html5["egrave"]
'è'
>>> html5["egrave;"]
'è'
```

要从单个 Python Unicode 字符向 HTML 实体名称的另一方向转换，请首先使用`ord()`获取字符的十进制值：

```py
>>> import html
>>> char = '\u00e9'
>>> dec_value = ord(char)
>>> html.entities.codepoint2name[dec_value]
'eacute'
```

对于超过一个字符的 Unicode 字符串，请使用这两步转换：

```py
>>> place = 'caf\u00e9'
>>> byte_value = place.encode('ascii', 'xmlcharrefreplace')
>>> byte_value
b'caf&#233;'
>>> byte_value.decode()
'caf&#233;'
```

表达式`place.encode('ascii', 'xmlcharrefreplace')`返回 ASCII 字符，但是作为类型`bytes`（因为它是*编码的）。需要以下`byte_value.decode()`来将`byte_value`转换为 HTML 兼容字符串。

## 标准化

一些 Unicode 字符可以用多种 Unicode 编码表示。它们看起来一样，但由于具有不同的内部字节序列，它们不能进行比较。例如，在`'café'`中，急性重音`'é'`可以用多种方式制作单个字符`'é'`：

```py
>>> eacute1 = 'é'                              # UTF-8, pasted
>>> eacute2 = '\u00e9'                         # Unicode code point
>>> eacute3 = \                                # Unicode name
...     '\N{LATIN SMALL LETTER E WITH ACUTE}'
>>> eacute4 = chr(233)                         # decimal byte value
>>> eacute5 = chr(0xe9)                        # hex byte value
>>> eacute1, eacute2, eacute3, eacute4, eacute5
('é', 'é', 'é', 'é', 'é')
>>> eacute1 == eacute2 == eacute3 == eacute4 == eacute5
True
```

尝试几个健全性检查：

```py
>>> import unicodedata
>>> unicodedata.name(eacute1)
'LATIN SMALL LETTER E WITH ACUTE'
>>> ord(eacute1)             # as a decimal integer
233
>>> 0xe9                     # Unicode hex integer
233
```

现在让我们通过将一个普通的`e`与一个重音符号结合来制作一个带重音的`e`：

```py
>>> eacute_combined1 = "e\u0301"
>>> eacute_combined2 = "e\N{COMBINING ACUTE ACCENT}"
>>> eacute_combined3 = "e" + "\u0301"
>>> eacute_combined1, eacute_combined2, eacute_combined3
('é', 'é', 'é'))
>>> eacute_combined1 == eacute_combined2 == eacute_combined3
True
>>> len(eacute_combined1)
2
```

我们用两个字符构建了一个 Unicode 字符，它看起来与原始的`'é'`相同。但正如他们在芝麻街上所说的那样，其中一个与其他不同：

```py
>>> eacute1 == eacute_combined1
False
```

如果您有来自不同来源的两个不同的 Unicode 文本字符串，一个使用`eacute1`，另一个使用`eacute_combined1`，它们看起来相同，但是神秘地不起作用。

您可以使用`unicodedata`模块中的`normalize()`函数修复这个问题：

```py
>>> import unicodedata
>>> eacute_normalized = unicodedata.normalize('NFC', eacute_combined1)
>>> len(eacute_normalized)
1
>>> eacute_normalized == eacute1
True
>>> unicodedata.name(eacute_normalized)
'LATIN SMALL LETTER E WITH ACUTE'
```

`'NFC'`的意思是*组合的正常形式*。

## 更多信息

如果您想了解更多关于 Unicode 的信息，这些链接特别有帮助：

+   [Unicode HOWTO](http://bit.ly/unicode-howto)

+   [实用 Unicode](http://bit.ly/pragmatic-uni)

+   [每个软件开发人员绝对必须了解的绝对最低限度关于 Unicode 和字符集的知识（无任何借口！）](http://bit.ly/jspolsky)

# 文本字符串：正则表达式

第五章讨论了简单的字符串操作。掌握了这些基础知识后，你可能已经在命令行上使用了简单的“通配符”模式，比如 UNIX 命令 `ls *.py`，意思是*列出所有以 .py 结尾的文件名*。

是时候通过使用*正则表达式*来探索更复杂的模式匹配了。这些功能在标准模块 `re` 中提供。你定义一个要匹配的字符串*模式*，以及要匹配的*源*字符串。对于简单的匹配，用法如下：

```py
>>> import re
>>> result = re.match('You', 'Young Frankenstein')
```

在这里，`'You'` 是我们要查找的*模式*，`'Young Frankenstein'` 是*源*（我们要搜索的字符串）。`match()` 检查*源*是否以*模式*开头。

对于更复杂的匹配，你可以先*编译*你的模式以加快后续的匹配速度：

```py
>>> import re
>>> youpattern = re.compile('You')
```

然后，你可以对编译后的模式执行匹配：

```py
>>> import re
>>> result = youpattern.match('Young Frankenstein')
```

###### 注意

因为这是一个常见的 Python 陷阱，我在这里再次强调：`match()` 只匹配从*源*的*开头*开始的模式。`search()` 则可以在*源*的*任何位置*匹配模式。

`match()` 不是比较模式和源的唯一方法。以下是你可以使用的几种其他方法（我们在下面的各节中讨论每一种方法）：

+   `search()` 如果有的话返回第一个匹配项。

+   `findall()` 返回所有非重叠匹配项的列表（如果有的话）。

+   `split()` 在*源*中匹配*模式*并返回字符串片段列表。

+   `sub()` 还需要另一个*替换*参数，并将*源*中与*模式*匹配的所有部分更改为*替换*。

###### 注意

这里大多数正则表达式示例都使用 ASCII，但 Python 的字符串函数，包括正则表达式，可以处理任何 Python 字符串和任何 Unicode 字符。

## 使用 `match()` 找到确切的起始匹配

字符串 `'Young Frankenstein'` 是否以 `'You'` 开头？以下是带有注释的代码：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.match('You', source)  # match starts at the beginning of source
>>> if m:  # match returns an object; do this to see what matched
...     print(m.group())
...
You
>>> m = re.match('^You', source) # start anchor does the same
>>> if m:
...     print(m.group())
...
You
```

`'Frank'` 怎么样？

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.match('Frank', source)
>>> if m:
...     print(m.group())
...
```

这次，`match()` 没有返回任何内容，因此 `if` 语句没有运行 `print` 语句。

正如我在 “新功能：我是海象” 中提到的，在 Python 3.8 中，你可以使用所谓的*海象操作符*简化这个例子：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> if m := re.match('Frank', source):
...     print(m.group())
...
```

现在让我们使用 `search()` 来查看 `'Frank'` 是否出现在源字符串中：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.search('Frank', source)
>>> if m:
...      print(m.group())
...
Frank
```

让我们改变模式，再次尝试使用 `match()` 进行起始匹配：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.match('.*Frank', source)
>>> if m:  # match returns an object
...     print(m.group())
...
Young Frank
```

这里简要解释了我们新的 `'.*Frank'` 模式的工作原理：

+   `.` 表示*任何单个字符*。

+   `*` 表示*前一个内容的零个或多个*。`.*` 在一起表示*任意数量的字符*（甚至是零个）。

+   `Frank` 是我们想要匹配的短语，某个地方。

`match()` 返回与 `.*Frank` 匹配的字符串：`'Young Frank'`。

## 使用 `search()` 找到第一个匹配项

你可以使用 `search()` 在字符串 `'Young Frankenstein'` 中找到模式 `'Frank'` 的任何位置，而不需要使用 `.*` 通配符：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.search('Frank', source)
>>> if m:  # search returns an object
...     print(m.group())
...
Frank
```

## 使用 `findall()` 查找所有匹配项

前面的例子只查找了一个匹配。但是如果你想知道字符串中单字母 `'n'` 的实例数量呢？

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.findall('n', source)
>>> m   # findall returns a list
['n', 'n', 'n', 'n']
>>> print('Found', len(m), 'matches')
Found 4 matches
```

后面跟着任何字符的 `'n'` 是怎样的？

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.findall('n.', source)
>>> m
['ng', 'nk', 'ns']
```

注意它没有匹配最后的 `'n'`。我们需要说 `'n'` 后面的字符是可选的，用 `?`：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.findall('n.?', source)
>>> m
['ng', 'nk', 'ns', 'n']
```

## 使用 `split()` 在匹配处分割

下一个示例展示了如何通过模式而不是简单字符串（正常字符串 `split()` 方法会执行的方式）将字符串分割成列表：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.split('n', source)
>>> m    # split returns a list
['You', 'g Fra', 'ke', 'stei', '']
```

## 使用 `sub()` 替换匹配项

这类似于字符串 `replace()` 方法，但用于模式而不是字面字符串：

```py
>>> import re
>>> source = 'Young Frankenstein'
>>> m = re.sub('n', '?', source)
>>> m   # sub returns a string
'You?g Fra?ke?stei?'
```

## 模式：特殊字符

许多正则表达式的描述从如何定义它们的所有细节开始。我认为这是一个错误。正则表达式是一个不那么小的语言，有太多细节无法一次掌握。它们使用了很多标点符号，看起来像卡通人物在咒骂。

掌握了这些表达式 (`match()`、`search()`、`findall()` 和 `sub()`）之后，让我们深入了解如何构建它们的细节。你制作的模式适用于这些函数中的任何一个。

你已经了解了基础知识：

+   匹配所有非特殊字符的文字

+   除 `\n` 外的任何单个字符用 `.`

+   任意数量的前一个字符（包括零）用 `*`

+   前一个字符的可选（零次或一次）用 `?`

首先，特殊字符显示在 表 12-2 中。

表 12-2\. 特殊字符

| 模式 | 匹配项 |
| --- | --- |
| `\d` | 单个数字 |
| `\D` | 单个非数字字符 |
| `\w` | 字母数字字符 |
| `\W` | 非字母数字字符 |
| `\s` | 空白字符 |
| `\S` | 非空白字符 |
| `\b` | 单词边界（在 `\w` 和 `\W` 之间，顺序不限） |
| `\B` | 非单词边界 |

Python 的 `string` 模块有预定义的字符串常量，我们可以用它们进行测试。让我们使用 `printable`，其中包含 100 个可打印的 ASCII 字符，包括大小写字母、数字、空格字符和标点符号：

```py
>>> import string
>>> printable = string.printable
>>> len(printable)
100
>>> printable[0:50]
'0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMN'
>>> printable[50:]
'OPQRSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~ \t\n\r\x0b\x0c'
```

`printable` 中哪些字符是数字？

```py
>>> re.findall('\d', printable)
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
```

哪些字符是数字、字母或下划线？

```py
>>> re.findall('\w', printable)
['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b',
'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n',
'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',
'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L',
'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X',
'Y', 'Z', '_']
```

哪些是空格？

```py
>>> re.findall('\s', printable)
[' ', '\t', '\n', '\r', '\x0b', '\x0c']
```

按顺序，这些是：普通空格、制表符、换行符、回车符、垂直制表符和换页符。

正则表达式不仅限于 ASCII。`\d` 将匹配任何 Unicode 所谓的数字，而不仅仅是 ASCII 字符 `'0'` 到 `'9'`。让我们从 [FileFormat.info](http://bit.ly/unicode-letter) 添加两个非 ASCII 小写字母：

在这个测试中，我们将加入以下内容：

+   三个 ASCII 字母

+   三个标点符号不应该与 `\w` 匹配

+   Unicode *带抑音的拉丁小写字母 E* (\u00ea)

+   Unicode *带抑音的拉丁小写字母 E* (\u0115)

```py
>>> x = 'abc' + '-/*' + '\u00ea' + '\u0115'
```

如预期，这个模式仅找到了字母：

```py
>>> re.findall('\w', x)
['a', 'b', 'c', 'ê', 'ĕ']
```

## 模式：使用限定符

现在让我们制作“标点披萨”，使用正则表达式的主要模式限定符，这些限定符在 表 12-3 中介绍。

在表中，*expr*和其他斜体字表示任何有效的正则表达式。

表 12-3。模式说明符

| 模式 | 匹配 |
| --- | --- |
| `abc` | 字面`abc` |
| `(` *expr* `)` | *expr* |
| *expr1* `&#124;` *expr2* | *expr1* 或 *expr2* |
| `.` | 除`\n`外的任意字符 |
| `^` | 源字符串的开头 |
| `$` | 源字符串的结尾 |
| *prev* `?` | 零或一个*prev* |
| *prev* `*` | 零或多个*prev*，尽可能多地匹配 |
| *prev* `*?` | 零或多个*prev*，尽可能少地匹配 |
| *prev* `+` | 一个或多个*prev*，尽可能多地匹配 |
| *prev* `+?` | 一个或多个*prev*，尽可能少地匹配 |
| *prev* `{` *m* `}` | *m*个连续的*prev* |
| *prev* `{` *m*, *n* `}` | *m*到*n*个连续的*prev*，尽可能多地匹配 |
| *prev* `{` *m*, *n* `}?` | *m*到*n*个连续的*prev*，尽可能少地匹配 |
| `[` *abc* `]` | `a`或`b`或`c`（等同于`a&#124;b&#124;c`） |
| `[^` *abc* `]` | *非*（`a`或`b`或`c`） |
| *prev* `(?=` *next* `)` | 若紧随其后则*prev* |
| *prev* `(?!` *next* `)` | 若不紧随其后则*prev* |
| `(?<=` *prev* `)` *next* | 若之前有*prev*则*next* |
| `(?<!` *prev* `)` *next* | 若不紧随其前则*next* |

当试图阅读这些示例时，你的眼睛可能永久地交叉了。首先，让我们定义我们的源字符串：

```py
>>> source = '''I wish I may, I wish I might
... Have a dish of fish tonight.'''
```

现在我们应用不同的正则表达式模式字符串来尝试在`source`字符串中匹配某些内容。

###### 注意

在以下示例中，我使用普通的引号字符串表示模式。在本节稍后，我将展示如何使用原始模式字符串（在初始引号前加上`r`）来避免 Python 正常字符串转义与正则表达式转义之间的冲突。因此，为了更安全，所有以下示例中的第一个参数实际上应该是原始字符串。

首先，在任意位置找到`wish`：

```py
>>> re.findall('wish', source)
['wish', 'wish']
```

接下来，在任意位置找到`wish`或`fish`：

```py
>>> re.findall('wish|fish', source)
['wish', 'wish', 'fish']
```

查找开头的`wish`：

```py
>>> re.findall('^wish', source)
[]
```

查找开头的`I wish`：

```py
>>> re.findall('^I wish', source)
['I wish']
```

查找结尾的`fish`：

```py
>>> re.findall('fish$', source)
[]
```

最后，在结尾找到`fish tonight.`：

```py
>>> re.findall('fish tonight.$', source)
['fish tonight.']
```

字符`^`和`$`称为*锚点*：`^`锚定搜索到搜索字符串的开始，而`$`锚定到结尾。`. $`匹配行尾的任意字符，包括句号，所以它起作用了。为了更精确，我们应该转义句点以确实匹配它：

```py
>>> re.findall('fish tonight\.$', source)
['fish tonight.']
```

从找到`w`或`f`后面跟着`ish`开始：

```py
>>> re.findall('[wf]ish', source)
['wish', 'wish', 'fish']
```

查找一个或多个`w`、`s`或`h`的连续序列：

```py
>>> re.findall('[wsh]+', source)
['w', 'sh', 'w', 'sh', 'h', 'sh', 'sh', 'h']
```

找到以非字母数字字符跟随的`ght`：

```py
>>> re.findall('ght\W', source)
['ght\n', 'ght.']
```

找到以`I`开头的`wish`：

```py
>>> re.findall('I (?=wish)', source)
['I ', 'I ']
```

最后，`wish`之前有`I`：

```py
>>> re.findall('(?<=I) wish', source)
[' wish', ' wish']
```

我之前提到过，有几种情况下正则表达式模式规则与 Python 字符串规则相冲突。以下模式应匹配以`fish`开头的任何单词：

```py
>>> re.findall('\bfish', source)
[]
```

为什么不这样做呢？如第五章中所述，Python 为字符串使用了一些特殊的转义字符。例如，`\b` 在字符串中表示退格，但在正则表达式的迷你语言中表示单词的开头。通过在定义正则表达式字符串时始终在其前加上 `r` 字符，可以避免意外使用转义字符，这样将禁用 Python 转义字符，如下所示：

```py
>>> re.findall(r'\bfish', source)
['fish']
```

## 模式：指定 `match()` 输出

在使用 `match()` 或 `search()` 时，所有匹配项都作为结果对象 `m` 的 `m.group()` 返回。如果将模式括在括号中，则匹配将保存到自己的组中，并作为 `m.groups()` 的元组可用，如下所示：

```py
>>> m = re.search(r'(. dish\b).*(\bfish)', source)
>>> m.group()
'a dish of fish'
>>> m.groups()
('a dish', 'fish')
```

如果使用此模式 `(?P<` *`name`* `>` *`expr`* `)`，它将匹配 *`expr`*，并将匹配保存在组 *`name`* 中：

```py
>>> m = re.search(r'(?P<DISH>. dish\b).*(?P<FISH>\bfish)', source)
>>> m.group()
'a dish of fish'
>>> m.groups()
('a dish', 'fish')
>>> m.group('DISH')
'a dish'
>>> m.group('FISH')
'fish'
```

# 二进制数据

文本数据可能会有挑战，但二进制数据可能会更加有趣。您需要了解诸如字节顺序（计算机处理器如何将数据分解为字节）和整数的符号位等概念。您可能需要深入了解二进制文件格式或网络数据包，以提取甚至更改数据。本节向您展示了在 Python 中进行二进制数据处理的基础知识。

## bytes 和 bytearray

Python 3 引入了以下八位整数序列，可能值为 0 到 255，有两种类型：

+   *bytes* 不可变，类似于字节元组

+   *bytearray* 可变，类似于字节列表

以名为 `blist` 的列表开始，下一个示例创建了名为 `the_bytes` 的 `bytes` 变量和名为 `the_byte_array` 的 `bytearray` 变量：

```py
>>> blist = [1, 2, 3, 255]
>>> the_bytes = bytes(blist)
>>> the_bytes
b'\x01\x02\x03\xff'
>>> the_byte_array = bytearray(blist)
>>> the_byte_array
bytearray(b'\x01\x02\x03\xff')
```

###### 注意

`bytes` 值的表示以 `b` 和引号字符开头，后跟诸如 `\x02` 或 ASCII 字符的十六进制序列，并以匹配的引号字符结束。Python 将十六进制序列或 ASCII 字符转换为小整数，但显示也有效的 ASCII 编码的字节值作为 ASCII 字符：

```py
>>> b'\x61'
b'a'
```

```py
>>> b'\x01abc\xff'
b'\x01abc\xff'
```

下一个示例演示了您不能更改 `bytes` 变量：

```py
>>> blist = [1, 2, 3, 255]
>>> the_bytes = bytes(blist)
>>> the_bytes[1] = 127
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'bytes' object does not support item assignment
```

但 `bytearray` 变量温和且可变：

```py
>>> blist = [1, 2, 3, 255]
>>> the_byte_array = bytearray(blist)
>>> the_byte_array
bytearray(b'\x01\x02\x03\xff')
>>> the_byte_array[1] = 127
>>> the_byte_array
bytearray(b'\x01\x7f\x03\xff')
```

这些操作会生成一个包含 0 到 255 的 256 元素结果：

```py
>>> the_bytes = bytes(range(0, 256))
>>> the_byte_array = bytearray(range(0, 256))
```

在打印 `bytes` 或 `bytearray` 数据时，Python 使用 `\x`*`xx`* 表示不可打印字节及其 ASCII 等效字符，对于可打印字符则显示其 ASCII 值（以及一些常见的转义字符，例如 `\n` 而非 `\x0a`）。以下是手动重新格式化以显示每行 16 个字节的 `the_bytes` 的打印表示：

```py
>>> the_bytes
b'\x00\x01\x02\x03\x04\x05\x06\x07\x08\t\n\x0b\x0c\r\x0e\x0f
\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f
!"#$%&\'()*+,-./
0123456789:;<=>?
@ABCDEFGHIJKLMNO
PQRSTUVWXYZ[\\]^_
`abcdefghijklmno
pqrstuvwxyz{|}~\x7f
\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f
\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f
\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf
\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf
\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf
\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf
\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef
\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff'
```

这可能会令人困惑，因为它们是字节（小整数），而不是字符。

## 使用 struct 转换二进制数据

正如您所见，Python 有许多用于操作文本的工具。用于二进制数据的工具则较少。标准库包含了处理类似于 C 和 C++ 中结构体的数据的 `struct` 模块。使用 `struct`，您可以将二进制数据转换为 Python 数据结构，反之亦然。

我们来看看如何处理来自 PNG 文件的数据——一种常见的图像格式，通常与 GIF 和 JPEG 文件一起出现。我们将编写一个小程序，从一些 PNG 数据中提取图像的宽度和高度。

我们将使用奥莱利的商标——在 图 12-1 中展示的小眼睛猫熊。

![inp2 1201](img/inp2_1201.png)

###### 图 12-1\. 奥莱利猫熊

此图像的 PNG 文件可在 [维基百科](http://bit.ly/orm-logo) 上找到。在 第 14 章 我才会介绍如何读取文件，因此我下载了这个文件，编写了一个小程序将其值作为字节打印出来，并只在一个名为 `data` 的 Python `bytes` 变量中键入了前 30 个字节的值，用于接下来的示例中。（PNG 格式规范指出宽度和高度存储在前 24 字节中，因此我们现在不需要更多。）

```py
>>> import struct
>>> valid_png_header = b'\x89PNG\r\n\x1a\n'
>>> data = b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR' + \
...     b'\x00\x00\x00\x9a\x00\x00\x00\x8d\x08\x02\x00\x00\x00\xc0'
>>> if data[:8] == valid_png_header:
...     width, height = struct.unpack('>LL', data[16:24])
...     print('Valid PNG, width', width, 'height', height)
... else:
...     print('Not a valid PNG')
...
Valid PNG, width 154 height 141
```

以下是此代码的功能：

+   `data` 包含来自 PNG 文件的前 30 个字节。为了适应页面，我用 `+` 和续行符（`\`）连接了两个字节字符串。

+   `valid_png_header` 包含标记有效 PNG 文件起始的八字节序列。

+   `width` 从第 16 至 19 字节提取，`height` 从第 20 至 23 字节提取。

`>LL` 是格式字符串，指示 `unpack()` 如何解释其输入字节序列并将其组装成 Python 数据类型。以下是详细说明：

+   `>` 表示整数以 *大端* 格式存储。

+   每个 `L` 指定一个四字节无符号长整数。

你可以直接检查每个四字节值：

```py
>>> data[16:20]
b'\x00\x00\x00\x9a'
>>> data[20:24]0x9a
b'\x00\x00\x00\x8d'
```

大端整数将最重要的字节放在左边。因为宽度和高度都小于 255，它们适合每个序列的最后一个字节。你可以验证这些十六进制值是否与预期的十进制值匹配：

```py
>>> 0x9a
154
>>> 0x8d
141
```

当你想反向操作并将 Python 数据转换为字节时，请使用 `struct` `pack()` 函数：

```py
>>> import struct
>>> struct.pack('>L', 154)
b'\x00\x00\x00\x9a'
>>> struct.pack('>L', 141)
b'\x00\x00\x00\x8d'
```

表 12-4 和 12-5 显示了 `pack()` 和 `unpack()` 的格式说明符。

字节顺序说明符在格式字符串中优先。

表 12-4\. 字节顺序说明符

| 格式说明符 | 字节顺序 |
| --- | --- |
| `<` | 小端 |
| `>` | 大端 |

表 12-5\. 格式说明符

| 格式说明符 | 描述 | 字节 |
| --- | --- | --- |
| `x` | 跳过一个字节 | 1 |
| `b` | 有符号字节 | 1 |
| `B` | 无符号字节 | 1 |
| `h` | 有符号短整数 | 2 |
| `H` | 无符号短整数 | 2 |
| `i` | 有符号整数 | 4 |
| `I` | 无符号整数 | 4 |
| `l` | 有符号长整数 | 4 |
| `L` | 无符号长整数 | 4 |
| `Q` | 无符号长长整数 | 8 |
| `f` | 单精度浮点数 | 4 |
| `d` | 双精度浮点数 | 8 |
| `p` | *count* 和字符 | 1 + *count* |
| `s` | 字符 | *count* |

类型说明符跟在字节顺序字符之后。任何说明符前都可以加一个数字，表示 *`count`*；`5B` 等同于 `BBBBB`。

你可以使用 *`count`* 前缀代替 `>LL`：

```py
>>> struct.unpack('>2L', data[16:24])
(154, 141)
```

我们使用切片 `data[16:24]` 直接抓取感兴趣的字节。我们也可以使用 `x` 标识符来跳过不感兴趣的部分：

```py
>>> struct.unpack('>16x2L6x', data)
(154, 141)
```

这意味着：

+   使用大端整数格式 (`>`)

+   跳过 16 个字节 (`16x`)

+   读取八个字节——两个无符号长整数 (`2L`)

+   跳过最后六个字节 (`6x`)

## 其他二进制数据工具

一些第三方开源软件包提供了以下更具声明性的方法来定义和提取二进制数据：

+   [bitstring](http://bit.ly/py-bitstring)

+   [construct](http://bit.ly/py-construct)

+   [hachoir](https://pypi.org/project/hachoir)

+   [binio](http://spika.net/py/binio)

+   [kaitai struct](http://kaitai.io)

附录 B 中详细介绍了如何下载和安装外部包，例如这些。在下一个示例中，您需要安装 `construct`。这是您需要做的全部工作：

```py
$ pip install construct
```

以下是如何通过使用 `construct` 从我们的 `data` 字节串中提取 PNG 尺寸的方法：

```py
>>> from construct import Struct, Magic, UBInt32, Const, String
>>> # adapted from code at https://github.com/construct
>>> fmt = Struct('png',
...     Magic(b'\x89PNG\r\n\x1a\n'),
...     UBInt32('length'),
...     Const(String('type', 4), b'IHDR'),
...     UBInt32('width'),
...     UBInt32('height')
...     )
>>> data = b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR' + \
...     b'\x00\x00\x00\x9a\x00\x00\x00\x8d\x08\x02\x00\x00\x00\xc0'
>>> result = fmt.parse(data)
>>> print(result)
Container:
 length = 13
 type = b'IHDR'
 width = 154
 height = 141
>>> print(result.width, result.height)
154, 141
```

## 使用 `binascii()` 转换字节/字符串

标准的 `binascii` 模块具有将二进制数据与各种字符串表示形式（十六进制（基数 16）、Base64、uuencoded 等）之间转换的函数。例如，在下一段代码中，让我们将那八字节的 PNG 头部打印为一系列十六进制值，而不是 Python 用来显示 *bytes* 变量的混合 ASCII 和 `\x` *xx* 转义的方式：

```py
>>> import binascii
>>> valid_png_header = b'\x89PNG\r\n\x1a\n'
>>> print(binascii.hexlify(valid_png_header))
b'89504e470d0a1a0a'
```

嘿，这个东西也可以反向操作：

```py
>>> print(binascii.unhexlify(b'89504e470d0a1a0a'))
b'\x89PNG\r\n\x1a\n'
```

## 位运算符

Python 提供了类似 C 语言的位级整数操作符。表 12-6 总结了这些操作符，并包括对整数变量 `x`（十进制 `5`，二进制 `0b0101`）和 `y`（十进制 `1`，二进制 `0b0001`）的示例。

表 12-6\. 位级整数运算符

| 操作符 | 描述 | 示例 | 十进制结果 | 二进制结果 |
| --- | --- | --- | --- | --- |
| `&` | 与 | `x & y` | `1` | `0b0001` |
| `&#124;` | 或 | `x &#124; y` | `5` | `0b0101` |
| `^` | 异或 | `x ^ y` | `4` | `0b0100` |
| `~` | 反转位 | `~x` | `-6` | *二进制表示取决于整数大小* |
| `<<` | 左移 | `x << 1` | `10` | `0b1010` |
| `>>` | 右移 | `x >> 1` | `2` | `0b0010` |

这些操作符的工作方式类似于 第 8 章 中的集合操作符。`&` 操作符返回两个参数中相同的位，`|` 返回两个参数中设置的位。`^` 操作符返回一个参数中的位，而不是两者都有的位。`~` 操作符反转其单个参数中的所有位；这也反转了符号，因为整数的最高位在 *二进制补码* 算术中表示其符号（`1` = 负数），这种算法在所有现代计算机中使用。`<<` 和 `>>` 操作符只是将位向左或向右移动。向左移动一位与乘以二相同，向右移动相当于除以二。

# 一个珠宝类比

Unicode 字符串就像魅力手链，而字节则像串珠。

# 即将到来

接下来是另一个实用章节：如何处理日期和时间。

# 待办事项

12.1 创建一个名为`mystery`的 Unicode 字符串，并将其赋值为`'\U0001f984'`。打印`mystery`及其 Unicode 名称。

12.2 使用 UTF-8 对`mystery`进行编码，并将结果存入名为`pop_bytes`的`bytes`变量中。打印`pop_bytes`。

12.3 使用 UTF-8 将`pop_bytes`解码为字符串变量`pop_string`。打印`pop_string`。`pop_string`等于`mystery`吗？

12.4 当您处理文本时，正则表达式非常方便。我们将以多种方式应用它们到我们特色的文本样本中。这是一首名为“Ode on the Mammoth Cheese”的诗，由詹姆斯·麦金泰尔于 1866 年写作，致敬于一个重七千磅的奶酪，在安大略省制作并发送国际巡回展。如果您不想全部输入，请使用您喜爱的搜索引擎并将单词剪切并粘贴到 Python 程序中，或者直接从[Project Gutenberg](http://bit.ly/mcintyre-poetry)获取。将文本字符串命名为`mammoth`。

##### 例子 12-1\. mammoth.txt

```py
We have seen thee, queen of cheese,
Lying quietly at your ease,
Gently fanned by evening breeze,
Thy fair form no flies dare seize.

All gaily dressed soon you'll go
To the great Provincial show,
To be admired by many a beau
In the city of Toronto.

Cows numerous as a swarm of bees,
Or as the leaves upon the trees,
It did require to make thee please,
And stand unrivalled, queen of cheese.

May you not receive a scar as
We have heard that Mr. Harris
Intends to send you off as far as
The great world's show at Paris.

Of the youth beware of these,
For some of them might rudely squeeze
And bite your cheek, then songs or glees
We could not sing, oh! queen of cheese.

We'rt thou suspended from balloon,
You'd cast a shade even at noon,
Folks would think it was the moon
About to fall and crush them soon.
```

12.5 导入`re`模块以使用 Python 的正则表达式函数。使用`re.findall()`打印所有以`c`开头的单词。

12.6 找出所有以`c`开头的四字单词。

12.7 找出所有以`r`结尾的单词。

12.8 找出所有包含恰好三个连续元音字母的单词。

12.9 使用`unhexlify`将这个十六进制字符串（从两个字符串组合成一个以适应页面）转换为名为`gif`的`bytes`变量：

```py
'47494638396101000100800000000000ffffff21f9' +
'0401000000002c000000000100010000020144003b'
```

12.10 `gif`中的字节定义了一个像素的透明 GIF 文件，这是最常见的图形文件格式之一。合法的 GIF 以 ASCII 字符*GIF89a*开头。`gif`是否符合这个规范？

12.11 GIF 的像素宽度是从字节偏移量 6 开始的 16 位小端整数，高度也是相同大小，从偏移量 8 开始。提取并打印这些值以供`gif`使用。它们都是`1`吗？

¹ 这种酒在德国有一个分音符号，但在去法国的途中在阿尔萨斯地区失去了它。

² 基数 16，由字符`0`-`9`和`A`-`F`指定。

³ 参见 HTML5 命名字符引用[图表](https://oreil.ly/pmBWO)。
