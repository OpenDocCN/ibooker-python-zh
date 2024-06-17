# 第二十三章：结构化文本：XML

XML，即 *可扩展标记语言*，是广泛使用的数据交换格式。除了 XML 本身外，XML 社区（主要在万维网联盟（W3C）内）还标准化了许多其他技术，如模式语言、命名空间、XPath、XLink、XPointer 和 XSLT。

行业联盟还定义了基于 XML 的行业特定标记语言，用于其各自领域应用程序之间的数据交换。XML、基于 XML 的标记语言和其他与 XML 相关的技术经常用于特定领域中的应用程序间、跨语言、跨平台的数据交换。

出于历史原因，Python 标准库在 xml 包下支持多个支持 XML 的模块，具有重叠的功能；本书并未覆盖所有内容，但感兴趣的读者可以在 [在线文档](https://oreil.ly/nHs5w) 中找到详细信息。

本书（特别是本章）仅涵盖了处理 XML 的最 Pythonic 方法：ElementTree，由[我们深感怀念的 Fredrik Lundh](https://oreil.ly/FjHRs)，即“effbot”创建。它的优雅、速度、通用性、多种实现和 Pythonic 架构使其成为 Python XML 应用程序的首选包。有关 xml.etree.ElementTree 模块的教程和完整详情，超出了本章提供的内容，请参阅[在线文档](https://oreil.ly/pPDh8)。本书默认读者具有一些关于 XML 本身的基础知识；如果你需要了解更多关于 XML 的知识，我们推荐 [*XML in a Nutshell*](http://shop.oreilly.com/product/9780596007645.do) 由 Elliotte Rusty Harold 和 W. Scott Means（O’Reilly）编写。

从不受信任的源解析 XML 会使你的应用程序面临许多可能的攻击风险。我们并未专门讨论这个问题，但 [在线文档](https://oreil.ly/jiWUx) 建议使用第三方模块来帮助保护你的应用程序，如果你确实需要从无法完全信任的源解析 XML。特别是，如果你需要具有针对解析不受信任源的安全防护的 ElementTree 实现，请考虑使用 [defusedxml.ElementTree](https://oreil.ly/dl21V)。

# ElementTree

Python 和第三方插件提供了几种 ElementTree 功能的替代实现；你始终可以依赖的是标准库中的模块 xml.etree.ElementTree。只需导入 xml.etree.ElementTree，即可获得你的 Python 安装中标准库中最快的实现。本章介绍的第三方包 defusedxml，提供了略慢但更安全的实现，如果你需要从不受信任的源解析 XML；另一个第三方包 [lxml](http://lxml.de) 则提供了更快的性能和一些额外的功能，通过 [lxml.etree](http://lxml.de/api.xhtml)。

传统上，您可以使用类似以下的 **from**...**import**...**as** 语句获取您喜欢使用的 ElementTree 实现：

```py
`from` xml.etree `import` ElementTree `as` et
```

或者尝试导入 lxml，并在无法导入时退回到标准库中提供的版本：

```py
`try`:
    `from` lxml `import` etree `as` et
`except` ImportError:
    `from` xml.etree `import` ElementTree `as` et
```

成功导入实现后，将其作为 et 使用（有些人喜欢大写变体 ET）在您的代码的其余部分中使用它。

ElementTree 提供了一个表示 XML 文档中节点的基本类：Element 类。ElementTree 还提供了其他重要的类，主要是代表整个树的类，具有输入和输出方法以及许多方便的类，等效于其 Element *root* 上的类——即 ElementTree 类。此外，ElementTree 模块提供了几个实用函数和次要重要的辅助类。

## Element 类

Element 类代表了 XML 文档中的一个节点，是整个 ElementTree 生态系统的核心。每个元素有点像映射，具有将字符串键映射到字符串值的*属性*，也有点像序列，具有其他元素（有时称为元素的“子元素”）的*子节点*。此外，每个元素还提供了一些额外的属性和方法。每个 Element 实例 *e* 有四个数据属性或属性，详见 Table 23-1。

Table 23-1\. Element 实例 e 的属性

| attrib | 包含 XML 节点所有属性的字典，以字符串为键（通常相应的值也是字符串）。例如，解析 XML 片段 <a x="y">b</a>c，得到一个 *e* 其中 *e*.attrib 是 {'x': 'y'}。

# 避免访问 Element 实例上的 attrib

在可能的情况下，最好避免访问 *e*.attrib，因为实现可能需要在访问时动态构建它。*e* 本身提供了一些典型的映射方法（列在 Table 23-2 中），您可能希望在 *e*.attrib 上调用这些方法；通过 *e* 自己的方法让实现可以为您优化性能，而不是通过实际的字典 *e*.attrib 获得的性能。

|

| tag | 节点的 XML 标签：一个字符串，有时也称为元素的*类型*。例如，解析 XML 片段 <a x="y">b</a>c，得到一个 *e* 其中 *e*.tag 设置为 'a'。 |
| --- | --- |
| tail | 紧随元素之后的任意数据（字符串）。例如，解析 XML 片段 <a x="y">b</a>c，得到一个 *e* 其中 *e*.tail 设置为 'c'。 |
| text | 直接“在”元素内的任意数据（字符串）。例如，解析 XML 片段 <a x="y">b</a>c，得到一个 *e* 其中 *e*.text 设置为 'b'。 |

*e* 有一些类似映射的方法，避免了需要显式请求 *e*.attrib 字典。这些方法在 Table 23-2 中列出。

表 23-2\. Element 实例 *e* 的类似映射方法

| clear | *e*.clear() “清空” *e*，除了其标签外，移除所有属性和子元素，并将文本和尾部设置为 **None**。 |
| --- | --- |
| get | *e*.get(*key*, default=**None**) 类似于 *e*.attrib.get(*key*, *default*)，但可能更快。不能使用 *e*[*key*]，因为在 *e* 上进行索引用于访问子元素，而不是属性。 |
| items | *e*.items() 返回所有属性的 (*name*, *value*) 元组列表，顺序任意。 |
| keys | *e*.keys() 返回所有属性名的列表，顺序任意。 |
| set | *e*.set(*key*, *value*) 将名为 *key* 的属性设置为 *value*。 |

*e* 的其他方法（包括使用 *e*[i] 语法进行索引和获取长度的方法，如 len(*e*)）处理 *e* 的所有子元素作为一个序列，或者在某些情况下——如本节其余部分所示——处理 *e* 的所有后代（以 *e* 为根的子树中的元素，也称为 *e* 的子元素）。

# 不要依赖于 Element 的隐式布尔转换

在所有 Python 3.11 及之前的版本中，如果 Element 实例 *e* 没有子元素，*e* 的布尔值为假，这遵循了 Python 容器隐式布尔转换的常规规则。然而，文档记录表明，这种行为可能会在未来的某个版本中发生变化。为了未来的兼容性，如果你想检查 *e* 是否没有子元素，请显式地检查 **if** len(*e*) == 0: 而不是使用通常的 Python 习惯用法 **if** **not** *e*:。

*e* 的命名方法处理子元素或后代的详细信息列在 表 23-3 中（本书不涵盖 XPath：有关该主题的信息，请参阅 [在线文档](https://oreil.ly/6E174)）。许多以下方法接受一个可选参数 namespaces，默认为 **None**。当存在时，namespaces 是一个映射，XML 命名空间前缀作为键，相应的 XML 命名空间全名作为值。

表 23-3\. Element 实例 *e* 处理子元素或后代的方法

| append | *e*.append(*se*) 在 *e* 的子元素末尾添加子元素 *se*（*se* 必须是一个 Element）。 |
| --- | --- |
| extend | *e*.extend(*ses*) 将可迭代对象 *ses* 中的每个元素（每个元素必须是一个 Element）添加到 *e* 的子元素末尾。 |
| find | *e*.find(*match*, namespaces=**None**) 返回第一个匹配 *match* 的后代元素，*match* 可以是标签名或 XPath 表达式（在当前 ElementTree 实现支持的子集内）。如果没有后代元素匹配 *match*，则返回 **None**。 |
| findall | *e*.findall(*match*, namespaces=**None**) 返回匹配 *match* 的所有后代元素列表，*match* 可以是标签名或 XPath 表达式（在当前 ElementTree 实现支持的子集内）。如果没有后代元素匹配 *match*，则返回 []。 |
| findtext | *e*.findtext(*match*, default=**None**, namespaces=**None**) 返回匹配 *match* 的第一个后代的文本，*match* 可以是标签名或当前 ElementTree 实现支持的 XPath 表达式的子集。如果匹配的第一个后代没有文本，则结果可能是空字符串 ''。如果没有后代匹配 *match*，则返回 default。 |
| insert | *e*.insert(*index*, *se*) 在 *e* 的子元素序列中的索引 *index* 处添加子元素 *se*（*se* 必须是 Element 类型）。 |
| iter | *e*.iter(*tag*='*') 返回一个迭代器，按深度优先顺序遍历所有 *e* 的后代。当 *tag* 不为 '*' 时，仅产生标签等于 *tag* 的子元素。在循环 *e*.iter 时，请不要修改以 *e* 为根的子树。 |
| iterfind | *e*.iterfind(*match*, namespaces=**None**) 返回一个迭代器，按深度优先顺序遍历所有匹配 *match* 的后代，*match* 可以是标签名或当前 ElementTree 实现支持的 XPath 表达式的子集。当没有后代匹配 *match* 时，结果迭代器为空。 |
| itertext | *e*.itertext(*match*, namespaces=**None**) 返回一个迭代器，按深度优先顺序遍历所有匹配 *match* 的后代的文本（不包括尾部），*match* 可以是标签名或当前 ElementTree 实现支持的 XPath 表达式的子集。当没有后代匹配 *match* 时，结果迭代器为空。 |
| remove | *e*.remove(*se*) 删除元素 *se*（如在 表 3-4 中所述）。 |

## **ElementTree 类**

ElementTree 类表示映射 XML 文档的树。ElementTree 实例 *et* 的核心附加值是具有用于整体解析（输入）和写入（输出）整个树的方法。这些方法在 表 23-4 中描述。

表 23-4\. ElementTree 实例解析和写入方法

| parse | *et*.parse(*source*, parser=**None**) *source* 可以是打开以供读取的文件，或要打开并读取的文件名（要解析字符串，请将其包装在 io.StringIO 中，如 “内存文件：io.StringIO 和 io.BytesIO” 中所述），其中包含 XML 文本。*et*.parse 解析该文本，构建其元素树作为 *et* 的新内容（丢弃 *et* 的先前内容（如果有）），并返回树的根元素。parser 是一个可选的解析器实例；默认情况下，*et*.parse 使用由 ElementTree 模块提供的 XMLParser 类的实例（本书不涵盖 XMLParser；请参阅 [在线文档](https://oreil.ly/TXwf5)）。 |
| --- | --- |
| write | *et*.write(*file*, encoding='us-ascii', xml_declaration=**None**, default_namespace=**None**, method='xml', short_empty_elements=True) *file* 可以是已打开并用于写入的文件，或要打开并写入的文件名称（要写入字符串，请将 *file* 作为 io.StringIO 的实例传递，详见 “内存文件：io.StringIO 和 io.BytesIO”）。*et*.write 将文本写入该文件，表示树的 XML 文档内容，该树是 *et* 的内容。 |

| write *(续)* | *encoding* 应该按照 [标准](https://oreil.ly/Vlj0C) 拼写，而不是使用常见的“昵称” — 例如，'iso-8859-1'，而不是 'latin-1'，尽管 Python 本身接受这两种编码拼写方式，并且类似地，'utf-8' 带有破折号，而不是 'utf8' 没有破折号。通常最好选择将 encoding 传递为 'unicode'。当 *file*.write 接受这样的字符串时，这会输出文本（Unicode）字符串；否则，*file*.write 必须接受字节串，而 *et*.write 输出的字符串类型将是这种类型，对于不在编码中的字符，将使用 XML 字符引用输出 — 例如，默认的 US-ASCII 编码，“带重音符的 e”，é，将输出为 &#233;。您可以将 xml_declaration 传递为 **False** 以避免在生成的文本中包含声明，或者传递为 **True** 以包含声明；默认情况下，仅在编码不是 'us-ascii'、'utf-8' 或 'unicode' 之一时才包含声明。

您可以选择性地传递 default_namespace 来设置 xmlns 结构的默认命名空间。

您可以将 method 传递为 'text' 以仅输出每个节点的文本和尾部（无标记）。您可以将 method 传递为 'html' 以 HTML 格式输出文档（例如，在 HTML 中不需要的结束标记，如 </br> 将被省略）。默认为 'xml'，以 XML 格式输出。

您可以通过名称（而不是位置）选择性地将 short_empty_elements 传递为 **False**，以始终使用显式的开始和结束标记，即使对于没有文本或子元素的元素也是如此；默认情况下，对于这种空元素使用 XML 简短形式。例如，默认情况下，具有标签 a 的空元素将输出为 <a/>，如果将 short_empty_elements 传递为 **False**，则将输出为 <a></a>。 |

此外，ElementTree 的一个实例 *et* 提供了方法 getroot（返回树的根）和便利方法 find、findall、findtext、iter 和 iterfind，每个方法与在树的根上调用相同的方法完全等效，也就是说，在 *et*.getroot 的结果上调用。

## ElementTree 模块中的函数

ElementTree 模块还提供了几个函数，详见 表 23-5。

表 23-5\. ElementTree 函数

| Comment | Comment(text=**None**) 返回一个元素，在插入 ElementTree 作为节点后，将作为 XML 注释输出，注释文本字符串被封闭在'<!--'和'-->'之间。XMLParser 跳过任何文档中的 XML 注释，因此这个函数是插入注释节点的唯一方法。 |
| --- | --- |
| dump | dump(*e*) 将 e（可以是 Element 或 ElementTree）以 XML 形式写入 sys.stdout。此函数仅用于调试目的。 |
| fromstring | fromstring(*text*, parser=**None**) 从 *text* 字符串解析 XML 并返回一个 Element，就像刚刚介绍的 XML 函数一样。 |
| fromstringlist | fromstringlist(*sequence*, parser=**None**) 就像 fromstring(''.join(*sequence*))，但通过避免连接，可能会更快一些。 |
| iselement | iselement(*e*) 如果 *e* 是一个 Element，则返回 **True**；否则返回 **False**。 |

| iterparse | iterparse(*source*, events=['end'], parser=**None**) 解析 XML 文档并逐步构建相应的 ElementTree。*source* 可以是打开进行读取的文件，或要打开并读取的文件名，包含 XML 文档作为文本。iterparse 返回一个迭代器，产生两项元组 (*event*, *element*)，其中 *event* 是参数 events 中列出的字符串之一（每个字符串必须是 'start'、'end'、'start-ns' 或 'end-ns'），随着解析的进行而变化。*element* 是 'start' 和 'end' 事件的 Element，'end-ns' 事件的 **None**，以及 'start-ns' 事件的两个字符串元组（*namespace_prefix*, *namespace_uri*）。parser 是一个可选的解析器实例；默认情况下，iterparse 使用 ElementTree 模块提供的 XMLParser 类的实例（有关 XMLParser 类的详细信息，请参阅[在线文档](https://oreil.ly/wG429)）。

iterparse 的目的是在可行的情况下，允许你逐步解析一个大型 XML 文档，而不必一次性将所有生成的 ElementTree 存储在内存中。我们在“逐步解析 XML”中详细讨论了 iterparse。

| parse | parse(*source*, parser=**None**) 就像 ElementTree 的 parse 方法，在表 23-4 中介绍的一样，但它返回它创建的 ElementTree 实例。 |
| --- | --- |
| P⁠r⁠o⁠c⁠e⁠s⁠s⁠i⁠n⁠g​I⁠n⁠s⁠t⁠r⁠u⁠c⁠t⁠i⁠o⁠n | ProcessingInstruction(*target*, text=**None**) 返回一个元素，在插入 ElementTree 作为节点后，将作为 XML 处理指令输出，目标和文本字符串被封闭在'<?'和'?>'之间。XMLParser 跳过任何文档中的 XML 处理指令，因此这个函数是插入处理指令节点的唯一方法。 |
| r⁠e⁠g⁠i⁠s⁠t⁠e⁠r⁠_​n⁠a⁠m⁠e⁠s⁠p⁠a⁠c⁠e | register_namespace(*prefix*, *uri*) 将字符串 *prefix* 注册为字符串 *uri* 的命名空间前缀；命名空间中的元素将使用此前缀进行序列化。 |
| SubElement | SubElement(*parent*, *tag*, attrib={}, ***extra*) 创建一个带有给定*tag*和来自字典 attrib 的属性以及作为额外命名参数传递的其他内容的 Element，并将其作为 Element *parent*的最右边子节点添加；返回它创建的 Element。 |
| tostring | tostring(*e*, encoding='us-ascii', method='xml', short_empty_elements=**True**) 返回一个字符串，其中包含以 Element *e*为根的子树的 XML 表示。参数的含义与 ElementTree 的 write 方法相同，见表 23-4。 |
| tostringlist | tostringlist(*e*, encoding='us-ascii', method='xml', short_empty_elements=**True**) 返回一个字符串列表，其中包含以 Element *e*为根的子树的 XML 表示。参数的含义与 ElementTree 的 write 方法相同，见表 23-4。 |
| XML | XML(*text*, parser=**None**) 从文本字符串*text*解析 XML 并返回一个 Element。parser 是可选的解析器实例；默认情况下，XML 使用由 ElementTree 模块提供的 XMLParser 类的实例（本书不涵盖 XMLParser 类；详见[在线文档](https://oreil.ly/wG429)）。 |
| XMLID | XMLID(*text*, parser=**None**) 从文本字符串*text*解析 XML 并返回一个包含两个条目的元组：一个 Element 和一个将 id 属性映射到每个唯一 Element 的字典（XML 禁止重复 id）。parser 是可选的解析器实例；默认情况下，XMLID 使用由 ElementTree 模块提供的 XMLParser 类的实例（本书不涵盖 XMLParser 类；详见[在线文档](https://oreil.ly/wG429)）。 |

ElementTree 模块还提供了 QName、TreeBuilder 和 XMLParser 类，这些我们在本书中不涵盖，以及 XMLPullParser 类，见“迭代解析 XML”。

# 使用 ElementTree.parse 解析 XML

在日常使用中，创建 ElementTree 实例最常见的方法是从文件或类似文件的对象中解析它，通常使用模块函数 parse 或 ElementTree 类实例的方法 parse。

在本章剩余的示例中，我们使用在[*http://www.w3schools.com/xml/simple.xml*](http://www.w3schools.com/xml/simple.xml)找到的简单 XML 文件；它的根标记是'breakfast_menu'，根的子节点是标记为'food'的元素。每个'food'元素都有一个标记为'name'的子元素，其文本是食物的名称，以及一个标记为'calories'的子元素，其文本是该食物一份中的卡路里数的整数表示。换句话说，对于示例感兴趣的 XML 文件内容的简化表示如下：

```py
`<breakfast_menu``>`
 `<food``>`
    `<name``>`Belgian Waffles`</name>`
    `<calories``>`650`</calories>`
 `</food>`
 `<food``>`
    `<name``>`Strawberry Belgian Waffles`</name>`
    `<calories``>`900`</calories>`
 `</food>`
 `<food``>`
    `<name``>`Berry-Berry Belgian Waffles`</name>`
    `<calories``>`900`</calories>`
 `</food>`
 `<food``>`
    `<name``>`French Toast`</name>`
    `<calories``>`600`</calories>`
 `</food>`
 `<food``>`
    `<name``>`Homestyle Breakfast`</name>`
    `<calories``>`950`</calories>`
 `</food>`
`</breakfast_menu>`
```

因为 XML 文档位于 WWW URL 上，所以首先获取一个具有该内容的类似文件的对象，并将其传递给 parse；最简单的方法使用 urllib.request 模块：

```py
`from` `urllib` `import` request
`from` `xml``.``etree` `import` ElementTree `as` et
content = request.urlopen('http://www.w3schools.com/xml/simple.xml')
tree = et.parse(content)
```

## 从 ElementTree 中选择元素

假设我们想要在标准输出上打印出各种食物的卡路里和名称，按升序卡路里排序，按字母顺序打破平局。以下是此任务的代码：

```py
`def` bycal_and_name(e):
    `return` int(e.find('calories').text), e.find('name').text

`for` `e` `in` sorted(tree.findall('food'), key=bycal_and_name):
    print(f"{e.find('calories').text} {e.find('name').text}")
```

当运行时，这将打印： 

```py
600 French Toast
650 Belgian Waffles
900 Berry-Berry Belgian Waffles
900 Strawberry Belgian Waffles
950 Homestyle Breakfast
```

## 编辑 ElementTree

构建好一个 ElementTree（无论是通过解析还是其他方式），你可以通过 ElementTree 和 Element 类的各种方法以及模块函数来“编辑”它——插入、删除和/或修改节点（元素）。例如，假设我们的程序可靠地通知我们菜单上添加了一种新食物——涂了黄油的烤面包，两片白面包烤过涂了黄油，含有“berry”字样的食物已被删除（不区分大小写）。针对这些规格的“编辑树”部分可以编码如下：

```py
*`# add Buttered Toast to the menu`*
menu = tree.getroot()
toast = et.SubElement(menu, 'food')
tcals = et.SubElement(toast, 'calories')
tcals.text = '180'
tname = et.SubElement(toast, 'name')
tname.text = 'Buttered Toast'
*`# remove anything related to 'berry' from the menu`*
`for` `e` `in` menu.findall('food'):
    name = e.find('name').text
    `if` 'berry' `in` name.lower():
        menu.remove(e)
```

一旦我们在解析树的代码和从中选择性打印的代码之间插入这些“编辑”步骤，后者将打印：

```py
180 Buttered Toast
600 French Toast
650 Belgian Waffles
950 Homestyle Breakfast
```

有时，编辑 ElementTree 的便捷性可能是一个关键考虑因素，值得你将其全部保留在内存中。

# 从头开始构建 ElementTree

有时，你的任务并不是从现有 XML 文档开始：相反，你需要根据代码从不同来源（如 CSV 文件或某种类型的数据库）获得的数据制作一个 XML 文档。

对于这类任务的代码类似于我们展示的用于编辑现有 ElementTree 的代码——只需添加一个小片段来构建一个最初为空的树。

例如，假设你有一个 CSV 文件，*menu.csv*，其中两列逗号分隔的是各种食物的卡路里和名称，每行一种食物。你的任务是构建一个 XML 文件，*menu.xml*，与我们在之前示例中解析过的类似。下面是你可以这样做的一种方式：

```py
import csv
`from` `xml``.``etree` `import` ElementTree `as` et

menu = et.Element('menu')
tree = et.ElementTree(menu)
`with` open('menu.csv') `as` f:
    r = csv.reader(f)
    `for` calories, namestr `in` r:
        food = et.SubElement(menu, 'food')
        cals = et.SubElement(food, 'calories')
        cals.text = calories
        name = et.SubElement(food, 'name')
        name.text = namestr

tree.write('menu.xml')
```

# 逐步解析 XML

针对从现有 XML 文档中选择元素的任务，有时你不需要将整个 ElementTree 构建在内存中——这一点特别重要，如果 XML 文档非常大时（对于我们处理的微小示例文档不适用，但可以想象类似的以菜单为中心的文档，列出了数百万种不同的食物）。

假设我们有这样一个大型文档，并且我们想要在标准输出上打印出卡路里最低的 10 种食物的卡路里和名称，按升序卡路里排序，按字母顺序打破平局。我们的 *menu.xml* 文件现在是一个本地文件，假设它列出了数百万种食物，因此我们宁愿不将其全部保存在内存中（显然，我们不需要一次性完全访问所有内容）。

以下代码代表了一种无需在内存中构建整个结构的简单尝试来解析：

```py
import heapq
`from` `xml``.``etree` `import` ElementTree `as` et

def cals_and_name():
 *`# generator for (calories, name) pairs`*
 `for` _, elem `in` et.iterparse('menu.xml'):
        `if` elem.tag != 'food':
            `continue`
     *`# just finished parsing a food, get calories and name`*
     cals = int(elem.find('calories').text)
    name = elem.find('name').text
        `yield` (cals, name)

lowest10 = heapq.nsmallest(10, cals_and_name())

`for` cals, name `in` lowest10:
    print(cals, name)
```

这种方法确实有效，但不幸的是，它消耗的内存几乎与基于完整 et.parse 的方法相同！这是因为 iterparse 在内存中逐步构建了整个 ElementTree，尽管它仅仅回传事件，如（默认情况下仅）'end'，意味着“我刚刚完成了对这个元素的解析”。

要真正节省内存，我们至少可以在处理完元素后立即丢弃每个元素的所有内容——也就是说，在 **yield** 后，我们可以添加 elem.clear() 使刚处理过的元素为空。

这种方法确实可以节省一些内存，但并非全部，因为树的根仍然会有一个巨大的空子节点列表。要真正节省内存，我们需要获取'开始'事件，以便获取正在构建的 ElementTree 的根，并在使用每个元素后从中移除每个元素，而不仅仅是清除元素。也就是说，我们希望将生成器改为：

```py
def cals_and_name():
 *`# memory-thrifty generator for (calories, name) pairs`*
    root = `None`
    `for` event, elem `in` et.iterparse('menu.xml', ['start', 'end']):
        `if` event == 'start':
            `if` root `is` `None`:
                root = elem
    `continue`
        `if` elem.tag != 'food':
            `continue`
        *`# just finished parsing a food, get calories and name`*
        cals = int(elem.find('calories').text)
        name = elem.find('name').text
        `yield` (cals, name)
        root.remove(elem)
```

这种方法尽可能地节省内存，同时完成任务！

# 在异步循环中解析 XML

虽然 iterparse 在正确使用时可以节省内存，但仍不足以在异步循环中使用。这是因为 iterparse 对传递给它的文件对象进行阻塞读取调用：在异步处理中这种阻塞调用是不可取的。

ElementTree 提供了 XMLPullParser 类来解决这个问题；请参阅[在线文档](https://oreil.ly/WxMoH)了解该类的使用模式。

¹ Alex 太谦虚了，不过从 1995 年到 2005 年，他和 Fredrik 以及 Tim Peters 一起，*都是* Python 的权威。他们以其对语言的百科全书式和详细的了解而闻名，effbot、martellibot 和 timbot 创建的软件和文档对数百万人至关重要。
