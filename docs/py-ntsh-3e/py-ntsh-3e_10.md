# 第十章 正则表达式

正则表达式（REs，也称为 regexps）允许程序员指定模式字符串并进行搜索和替换。正则表达式并不容易掌握，但它们是处理文本的强大工具。Python 通过内置的 re 模块提供了丰富的正则表达式功能。在本章中，我们全面介绍了关于 Python 的正则表达式的所有内容。

# 正则表达式和 re 模块

正则表达式是由表示模式的字符串构建而成的。使用 RE 功能，你可以检查任何字符串，并查看字符串的哪些部分（如果有的话）匹配了模式。

re 模块提供了 Python 的 RE 功能。compile 函数从模式字符串和可选标志中构建一个 RE 对象。RE 对象的方法查找字符串中的 RE 匹配项或执行替换。re 模块还公开了等效于 RE 对象方法的函数，但将 RE 的模式字符串作为第一个参数。

本章涵盖了在 Python 中使用 REs；它并不教授如何创建 RE 模式的每一个细节。对于正则表达式的一般覆盖，我们推荐 Jeffrey Friedl 的书《*精通正则表达式*》（O’Reilly），该书在教程和高级水平上都对正则表达式进行了详尽的介绍。许多关于正则表达式的教程和参考资料也可以在网上找到，包括 Python 的[在线文档](https://oreil.ly/tj7jh)中的优秀详细教程。像[Pythex](http://pythex.org)和[regex101](https://regex101.com)这样的网站可以让你交互式地测试你的 REs。或者，你可以启动 IDLE，Python REPL 或任何其他交互式解释器，**import** re，并直接进行实验。

## REs 和 bytes 与 str

在 Python 中，REs 根据要匹配的对象的类型以两种方式工作：当应用于 str 实例时，REs 根据相应匹配（例如，Unicode 字符 *c* 如果 'LETTER' **in** unicodedata.name(*c*) 则被认为是“字母”）；当应用于 bytes 实例时，REs 根据 ASCII 进行匹配（例如，字节 *c* 如果 *c* **in** string.ascii_letters 则被认为是“字母”）。例如：

```py
`import` re
print(re.findall(r'\w+', 'cittá'))            *`# prints: ['cittá']`*
print(re.findall(rb'\w+', 'cittá'.encode())) *`# prints: [b'citt']`*
```

## 模式字符串语法

表示正则表达式的模式字符串遵循特定的语法：

+   字母和数字字符代表它们自身。一个 RE，其模式是由字母和数字组成的字符串，匹配相同的字符串。

+   许多字母数字字符在模式中以反斜杠（\）或 *转义* 的方式具有特殊含义。

+   标点符号字符的工作方式正好相反：它们在转义时代表它们自身，但在未转义时具有特殊含义。

+   反斜杠字符可以通过重复的反斜杠（\\）来匹配。

RE 模式是一个将一个或多个模式元素串联起来的字符串；每个元素本身也是一个 RE 模式。例如，r'a' 是一个单元素 RE 模式，匹配字母 a，而 r'ax' 是一个两元素 RE 模式，匹配紧接着的 a 后面跟着的 x。

由于 RE 模式经常包含反斜杠，最好总是以原始字符串字面形式指定 RE 模式（见 “字符串”）。模式元素（例如 r'\t'，等效于字符串字面量 '\\t'）确实匹配相应的特殊字符（在本例中是制表符 \t），因此即使需要字面匹配这些特殊字符，也可以使用原始字符串字面量。

表 10-1 列出了 RE 模式语法中的特殊元素。某些模式元素的确切含义会因可选标志与模式字符串一起构建 RE 对象而改变。可选标志在 “可选标志” 中介绍。

表 10-1\. RE 模式语法

| 元素 | 含义 |
| --- | --- |
| . | 匹配任何单个字符，除了换行符 \n（如果启用 DOTALL，则也匹配换行符 \n） |
| ^ | 匹配字符串的开头（如果启用 MULTILINE，则也匹配换行符后面的位置） |
| $ | 匹配字符串的结尾（如果启用 MULTILINE，则也匹配换行符前面的位置） |
| * | 匹配前一个 RE 的零次或多次；贪婪模式（尽可能多地匹配） |
| + | 匹配前一个 RE 的一次或多次；贪婪模式（尽可能多地匹配） |
| ? | 匹配前一个 RE 的零次或一次；贪婪模式（如果可能，则匹配一次） |
| *?, +?, ?? | *非贪婪版本* 的 *, +, ?，分别匹配尽可能少的情况 |
| {*m*} | 匹配前一个 RE 的 *m* 次 |
| {*m*, *n*} | 匹配前一个 RE 的 *m* 至 *n* 次；可以省略 *m* 或 *n*（或两者），默认 *m*=0 和 *n*=无穷大（贪婪模式） |
| {*m*, *n*}? | 匹配前一个 RE 的 *m* 至 *n* 次；非贪婪模式 |
| [...] | 匹配括号内任一字符集合中的一个字符 |
| [^...] | 匹配括号内不含字符的一个字符，^ 后面紧跟的字符 |
| &#124; | 匹配前一个 RE 或后一个 RE |
| (...) | 匹配括号内的 RE 并指示一个 *组* |
| (?aiLmsux) | 设置可选标志的另一种方式^(a) |
| (?:...) | 类似于 (...)，但不捕获匹配的字符组 |
| (?P*<id>*...) | 类似于 (...)，但该组同时获取名称 *<id>* |
| (?P=*<id>*) | 匹配先前由名称为 *<id>* 的组匹配的内容 |
| (?#...) | 括号内的内容仅作为注释；对匹配没有影响 |
| (?=...) | *前行断言*：如果 RE ... 匹配接下来的内容，则匹配，并且不消耗字符串的任何部分 |
| (?!...) | *负向前行断言*：如果 RE ... 不匹配接下来的内容，则匹配，并且不消耗字符串的任何部分 |
| (?<=...) | *后行断言*：如果 RE ... 的匹配正好结束于当前位置，则匹配 |
| (?<!...) | *负向后行断言*：如果 RE ... 的匹配正好不结束于当前位置，则匹配 |
| \ *number* | 匹配之前由编号 *number* 的组匹配的内容（组自动从左到右编号，从 1 到 99） |
| \A | 只在整个字符串的开头匹配一个空字符串 |
| \b | 只在单词的开头或结尾匹配一个空字符串（一个最大的字母数字字符序列；参见也 \w） |
| \B | 匹配一个空字符串，但不匹配单词的开头或结尾 |
| \d | 匹配一个数字，类似于集合 [0-9]（在 Unicode 模式下，许多其他 Unicode 字符也被视为“数字”对于 \d，但不适用于 [0-9]） |
| \D | 匹配一个非数字字符，类似于集合 [⁰-9]（在 Unicode 模式下，许多其他 Unicode 字符也被视为“数字”对于 \D，但不适用于 [⁰-9]） |
| \N{*name*} | 3.8+ 匹配与 *name* 对应的 Unicode 字符 |
| \s | 匹配一个空白字符，类似于集合 [\t\n\r\f\v] |
| \S | 匹配一个非空白字符，类似于集合 [^\t\n\r\f\v] |
| \w | 匹配一个字母数字字符；除非在 Unicode 模式下，或者 LOCALE 或 UNICODE 已设置，否则 \w 就像 [a-zA-Z0-9_] |
| \W | 匹配一个非字母数字字符，与 \w 的反义 |
| \Z | 只在整个字符串的结尾匹配一个空字符串 |
| \\ | 匹配一个反斜杠字符 |
| ^(a) 总是将设置标志（如果有）的 (?...) 结构放在模式的开头，以提高可读性；在其他位置放置会引发 DeprecationWarning。 |

使用字符 \ 后跟字母字符（不包括此处列出的字符或 表 3-4）会引发 re.error 异常。

## 常见的正则表达式惯用法

# 始终使用 r'...' 语法来表示 RE 模式文字

对于所有 RE 模式文字，请使用原始字符串文字，仅限于它们。这样可以确保您永远不会忘记转义反斜杠 (\)，并提高代码的可读性，因为它使您的 RE 模式文字更加突出。

.* 作为正则表达式模式字符串的子串意味着“任意数量（零或多个）的任意字符。”换句话说，.* 匹配目标字符串的任何子串，包括空子串。. + 类似，但只匹配非空子串。例如，这样：

```py
r'pre.*post'
```

匹配包含子字符串 'pre' 后接后续子字符串 'post' 的字符串，即使后者紧邻前者（例如，它同时匹配 'prepost' 和 'pre23post'）。另一方面，此模式：

```py
r'pre.+post'
```

只有当 'pre' 和 'post' 不相邻时才匹配（例如，它匹配 'pre23post' 但不匹配 'prepost'）。这两种模式还会匹配在 'post' 后继续的字符串。为了将模式限制为仅匹配以 'post' 结尾的字符串，请在模式结尾处使用 \Z。例如，这样：

```py
r'pre.*post\Z'
```

匹配 'prepost' 但不匹配 'preposterous'。

所有这些示例都是 *贪婪* 的，意味着它们匹配从第一个出现的 'pre' 开始到最后一个出现的 'post' 的子串。当您关心匹配字符串的哪个部分时，您通常会希望指定 *非贪婪* 匹配，在我们的例子中，它将匹配从第一个出现的 'pre' 开始，但仅限到下一个出现的 'post' 的第一次匹配。

例如，当字符串为 'preposterous and post facto' 时，贪婪的正则表达式模式 r'pre.*post' 将匹配子串 'preposterous and post'；非贪婪的变体 r'pre.*?post' 则仅匹配子串 'prepost'。

正则表达式模式中另一个经常使用的元素是 \b，它匹配单词边界。要仅匹配单词 'his' 而不是它在诸如 'this' 和 'history' 中出现的子串，正则表达式模式为：

```py
r'\bhis\b'
```

在单词边界之前和之后。要匹配以 'her' 开头的任何单词的开头，如 'her' 本身和 'hermetic'，但不是其他地方仅包含 'her' 的单词，如 'ether' 或 'there'，使用：

```py
r'\bher'
```

在相关字符串之前的单词边界，但不是之后。要匹配以 'its' 结尾的任何单词的结尾，如 'its' 本身和 'fits'，但不是其他地方包含 'its' 的单词，如 'itsy' 或 'jujitsu'，使用：

```py
r'its\b'
```

在相关字符串之后的单词边界，但不是之前。为了匹配这样受限的完整单词，而不仅仅是它们的开头或结尾，添加模式元素 \w* 来匹配零个或多个单词字符。要匹配以 'her' 开头的任何完整单词，使用：

```py
r'\bher\w*'
```

要仅匹配以 'her' 开头的任何单词的前三个字母，但不包括单词 'her' 本身，使用负单词边界 \B：

```py
r'\bher\B'
```

要匹配以 'its' 结尾的任何完整单词，包括 'its' 本身，使用：

```py
r'\w*its\b'
```

## 字符集

您可以通过在方括号([])内列出字符来表示模式中的字符集。除了列出字符外，还可以通过用连字符(-)分隔的第一个和最后一个字符来表示范围。范围的最后一个字符包括在集合中，与其他 Python 范围不同。在集合内，特殊字符代表它们自己，除了 \、] 和 -，当它们的位置使它们不转义时（通过在它们前面加上反斜杠），它们将形成集合的语法部分。您可以通过转义字母表示法，如 \d 或 \S，在集合中表示字符类。在集合的模式中，\b 表示退格字符(chr(8))，而不是单词边界。如果集合模式中的第一个字符，紧跟在 [ 之后，是一个插入符（^），则集合是 *补充的*：这样的集合匹配除了在集合模式表示法中的^后跟随的字符之外的任何字符。

字符集的一个常见用法是匹配“单词”，使用与\w 默认值（字母和数字）不同的字符定义可以组成单词。要匹配一个或多个字符的单词，每个字符可以是 ASCII 字母，撇号或连字符，但不能是数字（例如，“Finnegan-O'Hara”），使用：

```py
r"[a-zA-Z'\-]+"
```

# 总是需要在字符集中转义连字符。

在这种情况下，严格来说不必在字符集中使用反斜杠转义连字符，因为其位置位于集合的末尾，使得情况在语法上不模棱两可。然而，使用反斜杠是明智的，因为它使模式更可读，通过视觉上区分你希望作为集合中的字符的连字符，而不是用于表示范围的连字符。（当你想在字符集中包含反斜杠时，当然，你通过转义反斜杠本身来表示：将其写为\\。）

## 替代项

正则表达式模式中的竖线（|），用于指定替代项，具有较低的语法优先级。除非括号改变了分组，|应用于两侧整个模式，直到模式的开始或结束，或者另一个|。模式可以由任意数量的由|连接的子模式组成。重要的是要注意，由|连接的子模式的 RE 将匹配*第一个*匹配的子模式，而不是最长的子模式。像 r'ab|abc'的模式永远不会匹配'abc'，因为'ab'匹配首先得到评估。

给定单词列表*L*，匹配任意一个单词的 RE 模式是：

```py
'|'.join(rf'\b{word}\b' `for` word `in` L)
```

# 转义字符串

如果*L*的项目可以是更一般的字符串，而不仅仅是单词，您需要使用 re.escape 函数（在[表 10-6](https://example.org/additional_re_functions)中介绍）对每个项目进行*转义*，并且可能不希望在两侧使用\b 单词边界标记。在这种情况下，您可以使用以下 RE 模式（通过长度逆序对列表进行排序，以避免意外“掩盖”较长单词的较短单词）：

```py
'|'.join(re.escape(s) `for` s `in` sorted(
         L, key=len, reverse=`True`))
```

## 分组

正则表达式可以包含从零到 99 个（甚至更多，但只完全支持前 99 个）*分组*。模式字符串中的括号表示一个组。元素(?P<*id*>...)也表示一个组，并为组命名一个名为*id*的名称，该名称可以是任何 Python 标识符。所有组，包括命名和未命名的，都按从左到右，从 1 到 99 进行编号；“组 0”表示整个 RE 匹配的字符串。

对于 RE 与字符串的任何匹配，每个组都匹配一个子字符串（可能为空）。当 RE 使用|时，一些组可能不匹配任何子字符串，尽管整个 RE 确实匹配字符串。当一个组不匹配任何子字符串时，我们说该组不*参与*匹配。对于任何不参与匹配的组，空字符串（''）用作匹配的子字符串，除非本章后面另有说明。例如，这个：

```py
r'(.+)\1+\Z'
```

匹配由两个或更多次重复的非空子字符串组成的字符串。模式的 (.+) 部分匹配任何非空子字符串（任何字符，一次或多次），并定义了一个组，因为有括号。模式的 \1+ 部分匹配组的一次或多次重复，并且 \Z 将匹配锚定到字符串的结尾。

# 可选标志

函数 compile 的可选标志参数是通过使用 Python 的按位或运算符（|）对模块 re 的以下属性之一或多个进行按位或运算构建的编码整数。每个属性都有一个简短名称（一个大写字母），以方便使用，以及一个更可读的长名称（一个大写的多字母标识符），因此通常更可取：

A *or* ASCII

仅使用 ASCII 字符来匹配 \w、\W、\b、\B、\d 和 \D；覆盖默认的 UNICODE 标志

I *or* IGNORECASE

使匹配不区分大小写

L *or* LOCALE

使用 Python LOCALE 设置来确定 \w、\W、\b、\B、\d 和 \D 标记的字符；你只能在字节模式中使用此选项

M *or* MULTILINE

使得特殊字符 ^ 和 $ 匹配每一行的开头和结尾（即，在换行符之后/之前），以及整个字符串的开头和结尾（\A 和 \Z 仅匹配整个字符串的开头和结尾）

S *or* DOTALL

导致特殊字符 . 匹配任何字符，包括换行符

U *or* UNICODE

使用完整的 Unicode 来确定 \w、\W、\b、\B、\d 和 \D 标记的字符；虽然保留了向后兼容性，但此标志现在是默认的

X *or* VERBOSE

导致忽略模式中的空格，除非转义或在字符集中，并使得模式中的非转义 # 字符成为从该行到行尾的注释的开始

标志还可以通过在 (? 和 ) 之间插入一个或多个字母 aiLmsux 的模式元素来指定，而不是通过 re 模块的 compile 函数的标志参数（这些字母对应于前述列表中给出的大写标志）。选项应始终放在模式的开头；不这样做会产生弃用警告。特别是，如果 x（用于详细 RE 解析的内联标志字符）位于选项中，则必须将其放在模式的开头，因为 x 会改变 Python 解析模式的方式。选项适用于整个 RE，但 aLu 选项可以在组内局部应用。

使用显式的标志参数比在模式中放置选项元素更易读。例如，以下是使用 compile 函数定义等效 RE 的三种方法。这些 RE 中的每一个都匹配任何大小写字母组合的单词“hello”：

```py
`import` re
r1 = re.compile(r'(?i)hello')
r2 = re.compile(r'hello', re.I)
r3 = re.compile(r'hello', re.IGNORECASE)
```

第三种方法显然是最易读的，因此也是最易维护的，尽管稍微冗长。在这里使用原始字符串形式并非完全必要，因为模式不包含反斜杠。然而，使用原始字符串字面量没有坏处，我们建议您始终使用它们来改善 RE 模式的清晰度和可读性。

选项 re.VERBOSE（或 re.X）允许您通过适当使用空白和注释使模式更易读和理解。通常，复杂和冗长的 RE 模式最好由占据多行的字符串表示，因此您通常希望为这些模式字符串使用三重引号原始字符串字面量。例如，要匹配可能以八进制、十六进制或十进制格式表示的整数字符串，您可以使用以下任一种：

```py
repat_num1 = r'(0o[0-7]*|0x[\da-fA-F]+|[1-9]\d*)\Z'
repat_num2 = r'''(?x) *`# (re.VERBOSE) pattern matching int literals`*
 (  0o [0-7]* *`# octal: leading 0o, 0+ octal digits`*
 | 0x [\da-fA-F]+ *`# hex: 0x, then 1+ hex digits`*
 | [1-9] \d* *`# decimal: leading non-0, 0+ digits`*
 )\Z *`# end of string`*
              '''
```

此示例中定义的两个模式是等效的，但第二个模式通过注释和自由使用空白来以逻辑方式可视化分组模式，使其更易读和理解。

# 匹配与搜索

到目前为止，我们一直在使用正则表达式来 *匹配* 字符串。例如，带有模式 r'box' 的 RE 可以匹配字符串 'box' 和 'boxes'，但不能匹配 'inbox'。换句话说，RE 的 *匹配* 隐含地锚定在目标字符串的开头，就好像 RE 的模式以 \A 开头。

经常你会对 RE 中的可能匹配的位置感兴趣，无论在字符串的哪个位置（例如，在诸如 'inbox' 中找到 r'box' 的匹配，以及在 'box' 和 'boxes' 中）。在这种情况下，Python 中的术语称为 *search*，而不是匹配。对于这样的搜索，请使用 RE 对象的 search 方法，而不是 match 方法，后者仅从字符串的开头进行匹配。例如：

```py
`import` re
r1 = re.compile(r'box')
`if` r1.match('inbox'):
    print('match succeeds')
`else`:
    print('match fails')          *`# prints: match fails`*

`if` r1.search('inbox'):
    print('search succeeds')      *`# prints: search succeeds`*
`else`:
    print('search fails')
```

如果你想检查整个字符串是否匹配，而不仅仅是其开头，你可以使用 fullmatch 方法。所有这些方法都包含在 表 10-3 中。

# 在字符串的起始和结束锚定

\A 和 \Z 是确保正则表达式匹配在字符串的起始或结尾处 *锚定* 的模式元素。元素 ^ 用于起始，而 $ 用于结尾，也用于类似的角色。对于未标记为 MULTILINE 的 RE 对象，^ 等同于 \A，而 $ 等同于 \Z。然而，对于多行 RE，^ 可以锚定在字符串的起始或任何行的起始（“行”是基于 \n 分隔符字符确定的）。类似地，对于多行 RE，$ 可以锚定在字符串的结尾或任何行的结尾。无论 RE 对象是否为多行，\A 和 \Z 始终只锚定在字符串的起始和结尾处。

例如，这里是检查文件是否有任何以数字结尾的行的方法：

```py
`import` re
digatend = re.compile(r'\d$', re.MULTILINE)
`with` open('afile.txt') `as` f:
    `if` digatend.search(f.read()):
        print('some lines end with digits')
    `else`:
        print('no line ends with digits')
```

模式 r'\d\n' 几乎等效，但在这种情况下，如果文件的最后一个字符是一个不跟随换行符的数字，则搜索失败。使用前述示例，即使数字位于文件内容的最后，搜索也将成功，这与通常情况下数字后跟换行符的情况相同。

# 正则表达式对象

表 10-2 涵盖了正则表达式对象 *r* 的只读属性，详细说明了 *r* 是如何构建的（由模块 re 的 compile 函数完成，见 表 10-6）。

Table 10-2\. RE 对象的属性

| flags | flags 参数传递给 compile 函数时的参数，或者当省略 flags 时为 re.UNICODE；还包括使用前导 (?...) 元素在模式本身中指定的任何标志 |
| --- | --- |
| groupindex | 一个字典，其键是由元素 (?P<*id*>...) 定义的组名；相应的值是命名组的编号 |
| pattern | 编译 *r* 的模式字符串 |

这些属性使得可以轻松地从已编译的 RE 对象中检索其原始模式字符串和标志，因此您不必单独存储它们。

RE 对象 *r* 也提供了方法来在字符串中查找 *r* 的匹配项，并对这些匹配项执行替换操作（参见 表 10-3）。这些匹配项由特殊对象表示，在下一节中详细介绍。

Table 10-3\. RE 对象的方法

| findall | *r*.findall(*s*) 当 r 没有分组时，findall 返回一个字符串列表，其中每个字符串都是 *s* 中与 *r* 非重叠匹配的子串。例如，要打印文件中的所有单词，每行一个：

```py
`import` re
reword = re.compile(r'\w+')
`with` open('afile.txt') `as` f:
    `for` aword `in` reword.findall(f.read()):
        print(aword)
```

|

| findall *(cont.)* | 当 *r* 恰好有一个分组时，findall 也返回一个字符串列表，但每个字符串都是与 *r* 的组匹配的 *s* 的子串。例如，要仅打印后面跟有空白字符的单词（而不是跟有标点符号或字符串末尾的单词），您只需更改前面示例中的一个语句：

```py
reword = re.compile('(\w+)\s')
```

当 *r* 有 *n* 个分组（其中 *n* > 1）时，findall 返回一个元组列表，每个元组对应 *r* 的一个非重叠匹配。每个元组有 *n* 个项，对应 *r* 的每个分组匹配的 *s* 中的子串。例如，要打印每行中至少有两个单词的第一个和最后一个单词：

```py
`import` re
first_last = re.compile(r'^\W*(\w+)\b.*\b(\w+)\W*$', 
                        re.MULTILINE)
`with` open('afile.txt') `as` f:
    `for` first, last `in` first_last.findall(f.read()):
        print(first, last)
```

|

| finditer | *r*.finditer(*s*) finditer 类似于 findall，不同之处在于它返回一个迭代器，其项是匹配对象（在下一节中讨论）。因此，在大多数情况下，finditer 比 findall 更灵活，通常性能更好。 |
| --- | --- |
| fullmatch | *r*.fullmatch(*s*, start=0, end=sys.maxsize) 当完整子串 *s*（从索引 start 开始，结束于索引 end 之前）与 *r* 匹配时，返回一个匹配对象。否则，fullmatch 返回 **None**。 |

| match | *r*.match(*s*, start=0, end=sys.maxsize) 当 *s* 中以索引 start 开始且不到索引 end 的子字符串与 *r* 匹配时，返回一个适当的匹配对象。否则，match 返回 **None**。match 在 *s* 中的起始位置 start 隐式锚定。要在 *s* 中的任何位置搜索 *r* 的匹配项，从 start 开始，请调用 *r*.search，而不是 *r*.match。例如，这是一种打印所有以数字开头的行的方法:

```py
`import` re
digs = re.compile(r'\d')
`with` open('afile.txt') `as` f:
    `for` line `in` f:
        `if` digs.match(line):
            print(line, end='')
```

|

| search | *r*.search(*s*, start=0, end=sys.maxsize) 返回 *s* 的左侧子字符串的适当匹配对象，其起始位置不早于索引 start，且不达到索引 end，并与 *r* 匹配。当不存在这样的子字符串时，search 返回 **None**。例如，要打印包含数字的所有行，一个简单的方法如下:

```py
`import` re
digs = re.compile(r'\d')
`with` open('afile.txt') `as` f:
    `for` line in f:
        `if` digs.search(line):
            print(line, end='')
```

|

| split | *r*.split(*s*, maxsplit=0) 返回一个由 *s* 的 *r* 分割的列表 *L*（即由与 *r* 非重叠、非空匹配分隔的 *s* 的子字符串）。例如，这是一种从字符串中消除所有出现的 'hello'（不管大小写）的方法:

```py
`import` re
rehello = re.compile(r'hello', re.IGNORECASE)
astring = ''.join(rehello.split(astring))
```

当 *r* 有 *n* 组时，在 *L* 的每一对分割之间，*n* 个额外的项被交错插入。每个额外的项是 *s* 中与 *r* 对应组匹配的子字符串，如果该组未参与匹配，则为 **None**。例如，这是一种仅在冒号和数字之间出现空白时删除空白的方法:

```py
`import` re
re_col_ws_dig = re.compile(r'(:)\s+(\d)')
astring = ''.join(re_col_ws_dig.split(astring))
```

如果 maxsplit 大于 0，则 *L* 中最多有 maxsplit 个分割，每个分割后跟随 *n* 项，而 *s* 的最后一个匹配 *r* 的尾随子字符串（如果有的话）是 *L* 的最后一项。例如，要仅删除子字符串 'hello' 的第一个出现而不是全部出现，将第一个示例中的最后一条语句更改为:

```py
astring=''.join(rehello.split(astring, 1))
```

|

| sub | *r*.sub(*repl*, *s*, count=0) 返回一个 *s* 的副本，其中与 *r* 非重叠的匹配项被 *repl* 替换，*repl* 可以是字符串或可调用对象（如函数）。只有当空匹配不紧邻前一个匹配时，才替换空匹配。当 count 大于 0 时，仅替换 *s* 中前 count 次出现的 *r*。当 count 等于 0 时，替换 *s* 中所有的 *r* 匹配。例如，这是另一种更自然的方法，用于从任意大小写混合的字符串中仅删除子字符串 'hello' 的第一个出现:

```py
`import` re
rehello = re.compile(r'hello', re.IGNORECASE)
astring = rehello.sub('', astring, 1)
```

未提供 sub 的最后一个参数 1（一个），示例将删除所有 'hello' 的出现。

当 *repl* 是一个可调用对象时，*repl* 必须接受一个参数（匹配对象），并返回一个字符串（或 **None**，等效于返回空字符串 ''）作为匹配的替换内容。在这种情况下，sub 对每个与 *r* 匹配并替换的匹配调用 *repl*，并使用适当的匹配对象参数。例如，这是一种大写所有以 'h' 开头并以 'o' 结尾的单词的所有出现的方法:

```py
`import` re
h_word = re.compile(r'\bh\w*o\b', re.IGNORECASE)
`def` up(mo):
    `return` mo.group(0).upper()
astring = h_word.sub(up, astring)
```

|

| sub *(续)* | 当*repl*是一个字符串时，sub 使用*repl*本身作为替换，除了它会扩展反向引用。反向引用是*repl*中形式为 \g<*id*> 的子串，其中*id*是*r*中的一个组的名称（由*r*的模式字符串中的 (?P<*id*>...) 语法确定），或者 \dd，其中*dd*被视为一个组编号，可以是一位或两位数字。每个命名或编号的反向引用都将被替换为与所指示的*r*组匹配的*s*的子串。例如，这是一种在每个单词周围加上大括号的方法：

```py
`import` re
grouped_word = re.compile('(\w+)')
astring = grouped_word.sub(r'{\1}', astring)
```

|

| subn | *r*.subn(*repl*, *s*, count=0) subn 与 sub 相同，只是 subn 返回一对（*new_string*，*n*），其中*n*是 subn 执行的替换数。例如，这是一种计算任何大小写混合中子字符串 'hello' 出现次数的方法：

```py
`import` re
rehello = re.compile(r'hello', re.IGNORECASE)
_, count = rehello.subn('', astring)
print(f'Found {count} occurrences of "hello"') 
```

|

# 匹配对象

*匹配对象*由正则表达式对象的方法 fullmatch、match 和 search 创建并返回，并且是方法 finditer 返回的迭代器的项。当*repl*是可调用的时，它们也是方法 sub 和 subn 隐式创建的，因为在这种情况下，适当的匹配对象在每次调用*repl*时作为唯一参数传递。匹配对象*m*提供了以下只读属性，详细说明了搜索或匹配如何创建*m*，见 Table 10-4。

Table 10-4\. 匹配对象的属性

| pos | 传递给 search 或 match 的*start*参数（即，在*s*中开始匹配的索引） |
| --- | --- |
| endpos | 传递给 search 或 match 的*end*参数（即，匹配子串*s*必须结束的*s*中的索引） |
| lastgroup | 最后匹配的组的名称（如果最后匹配的组没有名称或者没有组参与匹配，则为**None**） |
| lastindex | 最后匹配的组的整数索引（如果没有组参与匹配，则为**None**） |
| re | 创建*m*的方法所创建的 RE 对象*r* |
| string | 传递给 finditer、fullmatch、match、search、sub 或 subn 的字符串*s* |

此外，匹配对象提供了 Table 10-5 中详细介绍的方法。

Table 10-5\. 匹配对象的方法

| end, span,

start | *m*.end(groupid=0), *m*.span(groupid=0),

*m*.start(groupid=0)

这些方法返回*m*.string 中与*groupid*（组号或名称；0，groupid 的默认值，表示“整个 RE”）标识的组匹配的子串的索引。当匹配的子串是*m*.string[*i*:*j*]时，*m*.start 返回*i*，*m*.end 返回*j*，*m*.span 返回(*i*，*j*)。如果该组没有参与匹配，则*i*和*j*均为-1。 |

| expand | *m*.expand(*s*) 返回一个副本，其中转义序列和反向引用的替换方式与方法*r*.sub 中描述的方式相同，见 Table 10-3。 |
| --- | --- |

| group | *m*.group(groupid=0, **groupids*) 使用单个参数 groupid（组号或名称），*m*.group 返回与由 groupid 标识的组匹配的子字符串，如果该组没有参与匹配，则返回 **None**。*m*.group()—或 *m*.group(0)—返回整个匹配的子字符串（组 0 表示整个 RE）。也可以使用 *m*[*index*] 访问组，就像使用 *m*.group(*index*) 调用一样（在任一情况下，*index* 可以是 int 或 str）。

当 group 以多个参数调用时，每个参数必须是组号或名称。然后，group 返回一个元组，每个参数一个项目，对应组匹配的子字符串，如果该组没有参与匹配，则返回 **None**。

| groupdict | *m*.groupdict(default=**None**) 返回一个字典，其键是 *r* 中所有命名组的名称。每个名称的值是与相应组匹配的子字符串，如果该组没有参与匹配，则为默认值。 |
| --- | --- |
| groups | *m*.groups(default=**None**) 返回一个元组，其中包含 *r* 中每个组的一个项目。每个项目是与相应组匹配的子字符串，如果该组没有参与匹配，则为默认值。元组不包括表示完整模式匹配的 0 组。 |

# re 模块的函数

除了 “可选标志” 中列出的属性之外，re 模块还为正则表达式对象的每个方法提供了一个函数（findall、finditer、fullmatch、match、search、split、sub 和 subn，在 表 10-3 中描述），每个函数都有一个额外的第一个参数，即模式字符串，该函数隐式编译为 RE 对象。通常最好显式地将模式字符串编译为 RE 对象并调用 RE 对象的方法，但有时，对于一次性使用 RE 模式，调用 re 模块的函数可能更方便。例如，要计算任何大小写混合中 'hello' 出现的次数，一种简洁的、基于函数的方法是：

```py
`import` re
_, count = re.subn(r'hello', '', astring, flags=re.I)
print(f'Found {count} occurrences of "hello"')
```

re 模块在内部缓存从传递给函数的模式创建的 RE 对象；要清除缓存并回收一些内存，调用 re.purge。

re 模块还提供了 error，即在出错时引发的异常类（通常是模式字符串的语法错误），以及另外两个函数，在 表 10-6 中列出。

表 10-6\. 其他 re 函数

| compile | compile(*pattern*, flags=0) 创建并返回一个 RE 对象，解析字符串 *pattern*，其语法如 “模式字符串语法” 中所述，并使用整数标志，如 “可选标志” 中所描述 |
| --- | --- |
| escape | escape(*s*) 返回字符串 *s* 的副本，其中每个非字母数字字符都被转义（即，在其前面加上反斜杠，\）；这对于将字符串 *s* 作为 RE 模式字符串的一部分进行文字匹配非常有用 |

# RE 和 := 运算符

在 Python 3.8 中引入的 := 操作符支持了一种类似于 Perl 中常见的连续匹配习语。在这种习语中，一系列的 **if**/**elsif** 分支根据不同的正则表达式测试字符串。在 Perl 中，**if** ($var =~ /regExpr/) 语句既评估正则表达式，又将成功的匹配保存在变量 var 中：¹

```py
if    ($statement =~ /I love (\w+)/) {
  print "He loves $1\n";
}
elsif ($statement =~ /Ich liebe (\w+)/) {
  print "Er liebt $1\n";
}
elsif ($statement =~ /Je t\'aime (\w+)/) {
  print "Il aime $1\n";
}
```

在 Python 3.8 之前，这种评估和存储行为在单个 **if**/**elif** 语句中是不可能的；开发者必须使用繁琐的嵌套 **if**/**else** 语句级联：

```py
m = re.match('I love (\w+)', statement)
`if` m:
    print(f'He loves {m.group(1)}')
`else`:
    m = re.match('Ich liebe (\w+)', statement)
    `if` m:
        print(f'Er liebt {m.group(1)}')
    `else`:
         m = re.match('J'aime (\w+)', statement)
        `if` m:
            print(f'Il aime {m.group(1)}')
```

使用 := 操作符，此代码简化为：

```py
`if` m := re.match(r'I love (\w+)', statement):
    print(f'He loves {m.group(1)}')

`elif` m := re.match(r'Ich liebe (\w+)', statement):
    print(f'Er liebt {m.group(1)}') 

`elif` m := re.match(r'J'aime (\w+)', statement):
    print(f'Il aime {m.group(1)}')
```

# 第三方 regex 模块

作为 Python 标准库 re 模块的替代方案，第三方正则表达式包 [regex module](https://oreil.ly/2wV-d)，由 Matthew Barnett 开发，非常流行。regex 提供与 re 模块兼容的 API，并添加了多种扩展功能，包括：

+   递归表达式

+   通过 Unicode 属性/值定义字符集

+   重叠匹配

+   模糊匹配

+   多线程支持（在匹配期间释放 GIL）

+   匹配超时

+   Unicode 不区分大小写匹配中的大小写折叠

+   嵌套集合

¹ 此示例取自正则表达式；参见[“Python 中的匹配组”](https://oreil.ly/czLsu)在 Stack Overflow 上的讨论。
