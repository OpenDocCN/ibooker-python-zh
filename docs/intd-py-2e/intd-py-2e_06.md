# 第五章：文本字符串

> 我总是喜欢奇怪的字符。
> 
> 蒂姆·伯顿

计算机书籍通常给人一种编程都是关于数学的印象。实际上，大多数程序员更常用于处理文本的*字符串*，而不是数字。逻辑（和创造性！）思维通常比数学技能更重要。

字符串是 Python 的第一个*序列*示例。在这种情况下，它们是*字符*的序列。但是什么是字符？它是书写系统中的最小单位，包括字母、数字、符号、标点符号，甚至空格或类似换行符的指令。字符由其含义（如何使用它）来定义，而不是它的外观。它可以有多个视觉表示（在不同*字体*中），而且多个字符可以具有相同的外观（比如在拉丁字母表中表示 `H` 音的视觉 `H`，但在西里尔字母表中表示拉丁 `N` 音）。

本章集中讨论如何制作和格式化简单文本字符串，使用 ASCII（基本字符集）示例。两个重要的文本主题推迟到第十二章：*Unicode* 字符（如我刚提到的 `H` 和 `N` 问题）和*正则表达式*（模式匹配）。

与其他语言不同，Python 中的字符串是*不可变的*。你不能直接改变一个字符串，但你可以将字符串的部分复制到另一个字符串以达到相同的效果。我们马上看看如何做到这一点。

# 用引号创建

通过将字符包含在匹配的单引号或双引号中，你可以创建一个 Python 字符串：

```py
>>> 'Snap'
'Snap'
>>> "Crackle"
'Crackle'
```

交互式解释器用单引号回显字符串，但 Python 对所有字符串处理都是完全相同的。

###### 注意

Python 有几种特殊类型的字符串，第一个引号前面的字母指示。`f` 或 `F` 开始一个*f 字符串*，用于格式化，在本章末尾描述。`r` 或 `R` 开始一个*原始字符串*，用于防止字符串中的*转义序列*（参见“用 \ 进行转义” 和 第十二章 中有关它在字符串模式匹配中的用法）。然后，有组合 `fr`（或 `FR`、`Fr` 或 `fR`）开始一个原始 f-string。`u` 开始一个 Unicode 字符串，它与普通字符串相同。`b` 开始一个 `bytes` 类型的值（参见第十二章）。除非我提到这些特殊类型之一，我总是在谈论普通的 Python Unicode 文本字符串。

为什么要有两种引号字符？主要目的是创建包含引号字符的字符串。你可以在双引号字符串中放单引号，或在单引号字符串中放双引号：

```py
>>> "'Nay!' said the naysayer. 'Neigh?' said the horse."
"'Nay!' said the naysayer. 'Neigh?' said the horse."
>>> 'The rare double quote in captivity: ".'
'The rare double quote in captivity: ".'
>>> 'A "two by four" is actually 1 1⁄2" × 3 1⁄2".'
'A "two by four" is actually 1 1⁄2" × 3 1⁄2".'
>>> "'There's the man that shot my paw!' cried the limping hound."
"'There's the man that shot my paw!' cried the limping hound."
```

你也可以使用三个单引号（`'''`）或三个双引号（`"""`）：

```py
>>> '''Boom!'''
'Boom'
>>> """Eek!"""
'Eek!'
```

三重引号对于这些短字符串并不是很有用。它们最常见的用途是创建*多行字符串*，就像爱德华·利尔的这首经典诗歌：

```py
>>> poem =  '''There was a Young Lady of Norway,
... Who casually sat in a doorway;
... When the door squeezed her flat,
... She exclaimed, "What of that?"
... This courageous Young Lady of Norway.'''
>>>
```

（这是在交互式解释器中输入的，第一行我们用 `>>>` 提示，接着是 `...` 直到我们输入最后的三重引号并进入下一行。）

如果你尝试在没有三重引号的情况下创建那首诗，当你转到第二行时，Python 会抱怨：

```py
>>> poem = 'There was a young lady of Norway,
  File "<stdin>", line 1
    poem = 'There was a young lady of Norway,
                                            ^
SyntaxError: EOL while scanning string literal
>>>
```

如果在三重引号中有多行文本，行尾字符将保留在字符串中。如果有前导或尾随空格，它们也将被保留：

```py
>>> poem2 = '''I do not like thee, Doctor Fell.
...     The reason why, I cannot tell.
...     But this I know, and know full well:
...     I do not like thee, Doctor Fell.
... '''
>>> print(poem2)
I do not like thee, Doctor Fell.
 The reason why, I cannot tell.
 But this I know, and know full well:
 I do not like thee, Doctor Fell.

>>>
```

顺便提一下，`print()`的输出与交互式解释器的自动回显是有区别的。

```py
>>> poem2
'I do not like thee, Doctor Fell.\n    The reason why, I cannot tell.\n    But
this I know, and know full well:\n    I do not like thee, Doctor Fell.\n'
```

`print()`会去除字符串的引号并打印它们的内容。它适用于人类输出。它会在打印的每个内容之间添加一个空格，并在末尾添加一个换行符：

```py
>>> print('Give', "us", '''some''', """space""")
Give us some space
```

如果你不想要空格或换行符，请参阅第十四章中的说明以避免它们。

交互式解释器打印字符串时带有单独的引号和*转义字符*，例如`\n`，这些在“使用\进行转义”中有解释。

```py
>>> """'Guten Morgen, mein Herr!'
... said mad king Ludwig to his wig."""
"'Guten Morgen, mein Herr!'\nsaid mad king Ludwig to his wig."
```

最后，还有*空字符串*，它完全没有字符但却是完全有效的。你可以用前述任何引号创建空字符串：

```py
>>> ''
''
>>> ""
''
>>> ''''''
''
>>> """"""
''
>>>
```

# 使用`str()`创建字符串。

你可以使用`str()`函数从其他数据类型创建字符串：

```py
>>> str(98.6)
'98.6'
>>> str(1.0e4)
'10000.0'
>>> str(True)
'True'
```

在调用`print()`时，Python 在对象不是字符串且在*字符串格式化*时内部使用`str()`函数，稍后在本章节中你会看到。

# 使用`\`进行转义

Python 允许你*转义*字符串中某些字符的含义，以实现其他难以表达的效果。通过在字符前加上反斜杠（`\`），你赋予它特殊的含义。最常见的转义序列是`\n`，表示开始新的一行。这样你可以从单行字符串创建多行字符串：

```py
>>> palindrome = 'A man,\nA plan,\nA canal:\nPanama.'
>>> print(palindrome)
A man,
A plan,
A canal:
Panama.
```

你会看到`\t`（制表符）的转义序列用于对齐文本：

```py
>>> print('\tabc')
 abc
>>> print('a\tbc')
a    bc
>>> print('ab\tc')
ab	c
>>> print('abc\t')
abc
```

（最终字符串具有终止的制表符，当然，你看不到。）

你可能还需要`\'`或`\"`来指定一个由相同字符引用的字符串中的字面单引号或双引号：

```py
>>> testimony = "\"I did nothing!\" he said. \"Or that other thing.\""
>>> testimony
'"I did nothing!" he said. "Or that other thing."'
>>> print(testimony)
"I did nothing!" he said. "Or that other thing."
```

```py
>>> fact = "The world's largest rubber duck was 54'2\" by 65'7\" by 105'"
>>> print(fact)
The world's largest rubber duck was 54'2" by 65'7" by 105'
```

如果你需要一个字面上的反斜杠，请输入两个（第一个转义第二个）：

```py
>>> speech = 'The backslash (\\) bends over backwards to please you.'
>>> print(speech)
The backslash (\) bends over backwards to please you.
>>>
```

正如本章开头所提到的，*原始字符串*会取消这些转义。

```py
>>> info = r'Type a \n to get a new line in a normal string'
>>> info
'Type a \\n to get a new line in a normal string'
>>> print(info)
Type a \n to get a new line in a normal string
```

（第一个`info`输出中的额外反斜杠是交互式解释器添加的。）

原始字符串不会取消任何真正的（不是`'\n'`）换行符：

```py
>>> poem = r'''Boys and girls, come out to play.
... The moon doth shine as bright as day.'''
>>> poem
'Boys and girls, come out to play.\nThe moon doth shine as bright as day.'
>>> print(poem)
Boys and girls, come out to play.
The moon doth shine as bright as day.
```

# 通过`+`进行组合

在 Python 中，你可以通过使用`+`运算符来组合字面字符串或字符串变量。

```py
>>> 'Release the kraken! ' + 'No, wait!'
'Release the kraken! No, wait!'
```

你还可以通过简单地将一个字符串放在另一个字符串后面来组合*字面字符串*（而不是字符串变量）：

```py
>>> "My word! " "A gentleman caller!"
'My word! A gentleman caller!'
>>> "Alas! ""The kraken!"
'Alas! The kraken!'
```

如果有很多这样的内容，你可以通过将其用括号括起来来避免转义行尾。

```py
>>> vowels = ( 'a'
... "e" '''i'''
... 'o' """u"""
... )
>>> vowels
'aeiou'
```

Python 在连接字符串时不会为你添加空格，因此在一些早期的示例中，我们需要显式地包含空格。Python 在`print()`语句的每个参数之间添加一个空格，并在末尾添加一个换行符。

```py
>>> a = 'Duck.'
>>> b = a
>>> c = 'Grey Duck!'
>>> a + b + c
'Duck.Duck.Grey Duck!'
>>> print(a, b, c)
Duck. Duck. Grey Duck!
```

# 使用`*`进行复制

你可以使用`*`运算符来复制一个字符串。尝试在交互式解释器中输入这些行，并查看它们打印出什么：

```py
>>> start = 'Na ' * 4 + '\n'
>>> middle = 'Hey ' * 3 + '\n'
>>> end = 'Goodbye.'
>>> print(start + start + middle + end)
```

请注意，`*` 比 `+` 优先级更高，因此在换行符附加之前字符串被复制。

# 通过 [] 获取一个字符

要从字符串中获取单个字符，请在字符串名称后的方括号内指定其 *offset*。第一个（最左边的）偏移量是 0，接下来是 1，依此类推。最后一个（最右边的）偏移量可以用 -1 指定，因此你不必计数；向左是 -2，-3，等等：

```py
>>> letters = 'abcdefghijklmnopqrstuvwxyz'
>>> letters[0]
'a'
>>> letters[1]
'b'
>>> letters[-1]
'z'
>>> letters[-2]
'y'
>>> letters[25]
'z'
>>> letters[5]
'f'
```

如果指定的偏移量等于或超过字符串的长度（记住，偏移量从 0 到长度减 1），将会引发异常：

```py
>>> letters[100]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
```

索引在其他序列类型（列表和元组）中的工作方式相同，我在第七章中介绍。

因为字符串是不可变的，您无法直接插入字符或更改特定索引处的字符。让我们尝试将 `'Henny'` 更改为 `'Penny'` 看看会发生什么：

```py
>>> name = 'Henny'
>>> name[0] = 'P'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
```

相反，您需要使用 `replace()` 或 *slice* 的某些组合（我们马上会看到的）等字符串函数：

```py
>>> name = 'Henny'
>>> name.replace('H', 'P')
'Penny'
>>> 'P' + name[1:]
'Penny'
```

我们没有更改 `name` 的值。交互式解释器只是打印替换的结果。

# 使用 *slice* 提取一个 *substring*

您可以通过使用 *slice* 从字符串中提取 *substring*（字符串的一部分）。您可以通过使用方括号、*`start`* 偏移量、*`end`* 偏移量和它们之间的可选 *`step`* 计数来定义 *slice*。您可以省略其中一些。切片将包括从 *`start`* 偏移量到 *`end`* 偏移量之前的字符：

+   `[:]` 提取从开头到结尾的整个序列。

+   `[` *`start`* `:]` 指定从 *`start`* 偏移量到结尾。

+   `[:` *`end`* `]` 指定从开头到 *`end`* 偏移量减 1。

+   `[` *`start`* `:` *`end`* `]` 表示从 *`start`* 偏移量到 *`end`* 偏移量减 1。

+   `[` *`start`* `:` *`end`* `:` *`step`* `]` 提取从 *`start`* 偏移量到 *`end`* 偏移量减 1，跳过 *`step`* 个字符。

如前所述，偏移量从左到右为 0、1 等等，从右到左为 -1、-2 等等。如果不指定 *`start`*，则切片使用 0（开头）。如果不指定 *`end`*，则使用字符串的末尾。

让我们创建一个包含小写英文字母的字符串：

```py
>>> letters = 'abcdefghijklmnopqrstuvwxyz'
```

使用一个普通的 `:` 等同于 `0:`（整个字符串）：

```py
>>> letters[:]
'abcdefghijklmnopqrstuvwxyz'
```

这里是一个从偏移量 20 开始到结尾的示例：

```py
>>> letters[20:]
'uvwxyz'
```

现在，从偏移量 10 到结尾：

```py
>>> letters[10:]
'klmnopqrstuvwxyz'
```

另一个，偏移量从 12 到 14。Python 不包括切片中的结束偏移量。开始偏移量是 *包含* 的，结束偏移量是 *不包含* 的：

```py
>>> letters[12:15]
'mno'
```

最后三个字符：

```py
>>> letters[-3:]
'xyz'
```

在下一个示例中，我们从偏移量 18 到倒数第 4 个提取；请注意与上一个示例的区别，在上一个示例中，从 -3 开始获取 `x`，但在 -3 结束实际上停在 -4，`w`：

```py
>>> letters[18:-3]
'stuvw'
```

在以下示例中，我们从倒数第 6 个到倒数第 3 个提取：

```py
>>> letters[-6:-2]
'uvwx'
```

如果要使用除 1 外的步长大小，请在第二个冒号后指定它，如下一系列示例所示。

从开头到结尾，步长为 7 个字符：

```py
>>> letters[::7]
'ahov'
```

从偏移量 4 到 19，步长为 3：

```py
>>> letters[4:20:3]
'ehknqt'
```

从偏移量 19 到末尾，步进为 4：

```py
>>> letters[19::4]
'tx'
```

从开头到偏移量 20 加 5：

```py
>>> letters[:21:5]
'afkpu'
```

（同样，*结束*需要比实际偏移量多一位。）

这还不是全部！给定一个负步长，这个方便的 Python 切片器还可以向后步进。它从末尾开始，直到开头，跳过一切：

```py
>>> letters[-1::-1]
'zyxwvutsrqponmlkjihgfedcba'
```

结果表明，你可以通过以下方式获得相同的结果：

```py
>>> letters[::-1]
'zyxwvutsrqponmlkjihgfedcba'
```

切片对于错误的偏移量更宽容，不像单索引查找`[]`那样严格。一个早于字符串开始的切片偏移量将被视为`0`，一个超过末尾的将被视为`-1`，正如在下面的示例中展示的那样。

从倒数第 50 位到末尾：

```py
>>> letters[-50:]
'abcdefghijklmnopqrstuvwxyz'
```

从倒数第 51 位到倒数第 50 位之前：

```py
>>> letters[-51:-50]
''
```

从开头到开头后的第 69 位：

```py
>>> letters[:70]
'abcdefghijklmnopqrstuvwxyz'
```

从开头后的第 70 位到开头后的第 70 位：

```py
>>> letters[70:71]
''
```

# 使用 len()获取长度

到目前为止，我们已经使用特殊的标点字符如`+`来操作字符串。但这些字符并不多。现在让我们开始使用一些 Python 内置的*函数*：这些是执行特定操作的命名代码片段。

`len()`函数用于计算字符串中的字符数：

```py
>>> len(letters)
26
>>> empty = ""
>>> len(empty)
0
```

你可以像在第七章中看到的那样，使用`len()`处理其他序列类型。

# 使用 split()分割

与`len()`不同，有些函数专门用于字符串。要使用字符串函数，输入字符串名称，一个点，函数名称和函数需要的*参数*：`*string*.*function*(*arguments*)`。关于函数的更长讨论请参见第九章。

你可以使用内置的字符串`split()`函数根据某个*分隔符*将字符串分割成一个*列表*。我们将在第七章中讨论列表。列表是一系列由逗号分隔并用方括号括起来的值：

```py
>>> tasks = 'get gloves,get mask,give cat vitamins,call ambulance'
>>> tasks.split(',')
['get gloves', 'get mask', 'give cat vitamins', 'call ambulance']
```

在前面的例子中，字符串称为`tasks`，字符串函数称为`split()`，带有单一的分隔符参数`','`。如果不指定分隔符，`split()`将使用任何连续的空白字符——换行符、空格和制表符：

```py
>>> tasks.split()
['get', 'gloves,get', 'mask,give', 'cat', 'vitamins,call', 'ambulance']
```

在不带参数调用`split`时，你仍然需要括号——这是 Python 知道你在调用函数的方式。

# 使用 join()合并

不太意外的是，`join()`函数是`split()`的反向操作：它将字符串列表合并成一个单独的字符串。看起来有点反向，因为你首先指定将所有东西粘合在一起的字符串，然后是要粘合的字符串列表：*`string`* `.join(` *`list`* `)`。所以，要使用换行符将列表`lines`连接起来，你会说`'\n'.join(lines)`。在下面的示例中，让我们用逗号和空格将列表中的一些名字连接起来：

```py
>>> crypto_list = ['Yeti', 'Bigfoot', 'Loch Ness Monster']
>>> crypto_string = ', '.join(crypto_list)
>>> print('Found and signing book deals:', crypto_string)
Found and signing book deals: Yeti, Bigfoot, Loch Ness Monster
```

# 使用 replace()替换

你用`replace()`进行简单的子字符串替换。给它旧的子字符串、新的子字符串，以及要替换的旧子字符串的实例数量。它返回更改后的字符串，但不修改原始字符串。如果省略这个最后的计数参数，它会替换所有实例。在这个例子中，只有一个字符串（`'duck'`）在返回的字符串中被匹配并替换：

```py
>>> setup = "a duck goes into a bar..."
>>> setup.replace('duck', 'marmoset')
'a marmoset goes into a bar...'
>>> setup
'a duck goes into a bar...'
```

更改多达 100 个：

```py
>>> setup.replace('a ', 'a famous ', 100)
'a famous duck goes into a famous bar...'
```

当你知道确切的子字符串要更改时，`replace()` 是一个很好的选择。但要小心。在第二个例子中，如果我们替换为单个字符字符串`'a'`而不是两个字符字符串`'a '`（`a`后跟一个空格），我们也会改变其他单词中间的`a`：

```py
>>> setup.replace('a', 'a famous', 100)
'a famous duck goes into a famous ba famousr...'
```

有时，你想确保子字符串是一个完整的单词，或者是一个单词的开头等。在这些情况下，你需要*正则表达式*，在第十二章中详细描述。

# 用 strip() 去除

从字符串中去除前导或尾随的“填充”字符，尤其是空格，这是非常常见的。这里显示的`strip()`函数假设你想要去除空白字符（`' '`, `'\t'`, `'\n'`），如果你不给它们参数的话。`strip()`会去除两端，`lstrip()`只从左边，`rstrip()`只从右边。假设字符串变量`world`包含字符串`"earth"`浮动在空格中：

```py
>>> world = "    earth   "
>>> world.strip()
'earth'
>>> world.strip(' ')
'earth'
>>> world.lstrip()
'earth   '
>>> world.rstrip()
'    earth'
```

如果字符不在那里，什么也不会发生：

```py
>>> world.strip('!')
'    earth   '
```

除了没有参数（意味着空白字符）或单个字符外，你还可以告诉`strip()`去除多字符字符串中的任何字符：

```py
>>> blurt = "What the...!!?"
>>> blurt.strip('.?!')
'What the'
```

附录 E 显示了一些对于`strip()`有用的字符组的定义：

```py
>>> import string
>>> string.whitespace
' \t\n\r\x0b\x0c'
>>> string.punctuation
'!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'
>>> blurt = "What the...!!?"
>>> blurt.strip(string.punctuation)
'What the'
>>> prospector = "What in tarnation ...??!!"
>>> prospector.strip(string.whitespace + string.punctuation)
'What in tarnation'
```

# 搜索和选择

Python 有一组庞大的字符串函数。让我们探讨它们中最常见的一些如何工作。我们的测试对象是以下字符串，其中包含了玛格丽特·卡文迪许，纽卡斯尔公爵夫人的不朽诗作“液体是什么？”的文字：

```py
>>> poem = '''All that doth flow we cannot liquid name
... Or else would fire and water be the same;
... But that is liquid which is moist and wet
... Fire that property can never get.
... Then 'tis not cold that doth the fire put out
... But 'tis the wet that makes it die, no doubt.'''
```

鼓舞人心！

首先，获取前 13 个字符（偏移量 0 到 12）：

```py
>>> poem[:13]
'All that doth'
```

这首诗有多少个字符？（空格和换行符都包括在计数中。）

```py
>>> len(poem)
250
```

它以`All`开头吗？

```py
>>> poem.startswith('All')
True
```

它以`That's all, folks!`结尾吗？

```py
>>> poem.endswith('That\'s all, folks!')
False
```

Python 有两个方法(`find()`和`index()`)用于找到子字符串的偏移量，并且有两个版本（从开始或结尾）。如果找到子字符串，它们的工作方式相同。如果找不到，`find()`返回`-1`，而`index()`引发异常。

让我们找到诗中单词`the`的第一次出现的偏移量：

```py
>>> word = 'the'
>>> poem.find(word)
73
>>> poem.index(word)
73
```

最后一个`the`的偏移量：

```py
>>> word = 'the'
>>> poem.rfind(word)
214
>>> poem.rindex(word)
214
```

但如果子字符串不在其中：

```py
>>> word = "duck"
>>> poem.find(word)
-1
>>> poem.rfind(word)
-1
>>> poem.index(word)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: substring not found
>>> poem.rfind(word)
-1
>>> poem.rindex(word)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: substring not found
```

三个字母序列`the`出现了多少次？

```py
>>> word = 'the'
>>> poem.count(word)
3
```

诗中的所有字符都是字母或数字吗？

```py
>>> poem.isalnum()
False
```

不，有一些标点字符。

# 案例

在这一部分，我们将看一些内置字符串函数的更多用法。我们的测试字符串再次是以下内容：

```py
>>> setup = 'a duck goes into a bar...'
```

从两端去除`.`序列：

```py
>>> setup.strip('.')
'a duck goes into a bar'
```

###### 注意

由于字符串是不可变的，这些示例中没有一个实际上更改了`setup`字符串。每个示例只是取`setup`的值，对其进行处理，并将结果作为新字符串返回。

将第一个单词大写：

```py
>>> setup.capitalize()
'A duck goes into a bar...'
```

将所有单词大写：

```py
>>> setup.title()
'A Duck Goes Into A Bar...'
```

将所有字符转换为大写：

```py
>>> setup.upper()
'A DUCK GOES INTO A BAR...'
```

将所有字符转换为小写：

```py
>>> setup.lower()
'a duck goes into a bar...'
```

交换大写和小写：

```py
>>> setup.swapcase()
'A DUCK GOES INTO A BAR...'
```

# 对齐

现在，让我们使用一些布局对齐函数。字符串在指定的总空间内（这里为`30`）居中对齐。

在 30 个空格内居中对齐字符串：

```py
>>> setup.center(30)
'  a duck goes into a bar...   '
```

左对齐：

```py
>>> setup.ljust(30)
'a duck goes into a bar...     '
```

右对齐：

```py
>>> setup.rjust(30)
'     a duck goes into a bar...'
```

接下来，我们看一下更多如何对齐字符串的方法。

# 格式化

您已经看到可以使用`+`来*连接*字符串。让我们看看如何使用各种格式将数据值*插值*到字符串中。您可以用此方法生成需要外观精确的报告、表格和其他输出。

除了上一节中的函数外，Python 还有三种格式化字符串的方法：

+   *旧风格*（支持 Python 2 和 3）

+   *新风格*（Python 2.6 及更高版本）

+   *f-strings*（Python 3.6 及更高版本）

## 旧风格：% 

旧式字符串格式化的形式为*`format_string`* `%` *`data`*。格式字符串中包含插值序列。表 5-1 说明了最简单的序列是一个`%`后跟一个指示要格式化的数据类型的字母。

表 5-1\. 转换类型

| `%s` | 字符串 |
| --- | --- |
| `%d` | 十进制整数 |
| `%x` | 十六进制整数 |
| `%o` | 八进制整数 |
| `%f` | 十进制浮点数 |
| `%e` | 指数浮点数 |
| `%g` | 十进制或指数浮点数 |
| `%%` | 一个字面量`%` |

您可以使用`%s`来表示任何数据类型，Python 会将其格式化为无额外空格的字符串。

以下是一些简单的示例。首先，一个整数：

```py
>>> '%s' % 42
'42'
>>> '%d' % 42
'42'
>>> '%x' % 42
'2a'
>>> '%o' % 42
'52'
```

一个浮点数：

```py
>>> '%s' % 7.03
'7.03'
>>> '%f' % 7.03
'7.030000'
>>> '%e' % 7.03
'7.030000e+00'
>>> '%g' % 7.03
'7.03'
```

一个整数和一个字面量`%`：

```py
>>> '%d%%' % 100
'100%'
```

让我们尝试一些字符串和整数插值：

```py
>>> actor = 'Richard Gere'
>>> cat = 'Chester'
>>> weight = 28
```

```py
>>> "My wife's favorite actor is %s" % actor
"My wife's favorite actor is Richard Gere"
```

```py
>>> "Our cat %s weighs %s pounds" % (cat, weight)
'Our cat Chester weighs 28 pounds'
```

字符串中的`%s`表示插入一个字符串。字符串中`%`的数量需要与跟随字符串之后的数据项的数量匹配。单个数据项，如`actor`，直接放在最后一个`%`之后。多个数据必须分组成*元组*（详细信息见第七章; 由括号界定，逗号分隔），如`(cat, weight)`。

尽管`weight`是一个整数，但字符串中的`%s`将其转换为字符串。

您可以在格式字符串的`%`和类型说明符之间添加其他值来指定最小宽度、最大宽度、对齐和字符填充。这本质上是一种小语言，比接下来的两个部分的语言更有限。让我们快速看看这些值：

+   初始`'%'`字符。

+   可选的*对齐*字符：没有或`'+'`表示右对齐，`'-'`表示左对齐。

+   可选的*最小宽度*字段宽度。

+   可选的`'.'`字符用于分隔*最小宽度*和*最大字符数*。

+   可选的*maxchars*（如果转换类型为`s`）指定要从数据值中打印多少个字符。如果转换类型为`f`，则指定*精度*（小数点后要打印多少位数）。

+   早期表格中的*转换类型*字符。

这很令人困惑，所以这里有一些字符串的示例：

```py
>>> thing = 'woodchuck'
>>> '%s' % thing
'woodchuck'
>>> '%12s' % thing
'   woodchuck'
>>> '%+12s' % thing
'   woodchuck'
>>> '%-12s' % thing
'woodchuck   '
>>> '%.3s' % thing
'woo'
>>> '%12.3s' % thing
'         woo'
>>> '%-12.3s' % thing
'woo         '
```

再来一次，和一个带有`%f`变体的浮点数：

```py
>>> thing = 98.6
>>> '%f' % thing
'98.600000'
>>> '%12f' % thing
'   98.600000'
>>> '%+12f' % thing
'  +98.600000'
>>> '%-12f' % thing
'98.600000   '
>>> '%.3f' % thing
'98.600'
>>> '%12.3f' % thing
'      98.600'
>>> '%-12.3f' % thing
'98.600      '
```

和一个整数与`%d`：

```py
>>> thing = 9876
>>> '%d' % thing
'9876'
>>> '%12d' % thing
'        9876'
>>> '%+12d' % thing
'       +9876'
>>> '%-12d' % thing
'9876        '
>>> '%.3d' % thing
'9876'
>>> '%12.3d' % thing
'        9876'
>>> '%-12.3d' % thing
'9876        '
```

对于整数，`%+12d`只是强制打印符号，并且带有`.3`的格式字符串对其无效，就像对浮点数一样。

## 新风格：{} 和 format()

仍然支持旧风格格式化。在 Python 2 中，将冻结在版本 2.7 上，将永远支持。对于 Python 3，请使用本节中描述的“新风格”格式化。如果你使用的是 Python 3.6 或更新版本，*f-strings*（“最新风格：f-strings”）更加推荐。

“新风格”格式化的形式为`*format_string*.format(*data*)`。

格式字符串与前一节不完全相同。这里演示了最简单的用法：

```py
>>> thing = 'woodchuck'
>>> '{}'.format(thing)
'woodchuck'
```

`format()`函数的参数需要按照格式字符串中的`{}`占位符的顺序：

```py
>>> thing = 'woodchuck'
>>> place = 'lake'
>>> 'The {} is in the {}.'.format(thing, place)
'The woodchuck is in the lake.'
```

使用新风格格式，你还可以像这样按位置指定参数：

```py
>>> 'The {1} is in the {0}.'.format(place, thing)
'The woodchuck is in the lake.'
```

值`0`指的是第一个参数`place`，`1`指的是`thing`。

`format()`的参数也可以是命名参数

```py
>>> 'The {thing} is in the {place}'.format(thing='duck', place='bathtub')
'The duck is in the bathtub'
```

或者是一个字典：

```py
>>> d = {'thing': 'duck', 'place': 'bathtub'}
```

在以下示例中，`{0}`是`format()`的第一个参数（字典`d`）：

```py
>>> 'The {0[thing]} is in the {0[place]}.'.format(d)
'The duck is in the bathtub.'
```

这些示例都使用默认格式打印它们的参数。新风格格式化与旧风格的格式字符串定义略有不同（示例如下）：

+   初始冒号（`':'`）。

+   可选的*填充*字符（默认为`' '`）以填充值字符串，如果比*minwidth*短。

+   可选的*对齐*字符。这次，左对齐是默认的。`'<'`也表示左对齐，`'>'`表示右对齐，`'^'`表示居中。

+   数字的可选*符号*。没有意味着仅为负数添加减号（`'-'`）。`' '`表示负数前添加减号，正数前添加空格（`' '`）。

+   可选的*minwidth*。一个可选的句点（`'.'`）用于分隔*minwidth*和*maxchars*。

+   可选的*maxchars*。

+   *转换类型*。

```py
>>> thing = 'wraith'
>>> place = 'window'
>>> 'The {} is at the {}'.format(thing, place)
'The wraith is at the window'
>>> 'The {:10s} is at the {:10s}'.format(thing, place)
'The wraith     is at the window    '
>>> 'The {:<10s} is at the {:<10s}'.format(thing, place)
'The wraith     is at the window    '
>>> 'The {:¹⁰s} is at the {:¹⁰s}'.format(thing, place)
'The   wraith   is at the   window  '
>>> 'The {:>10s} is at the {:>10s}'.format(thing, place)
'The     wraith is at the     window'
>>> 'The {:!¹⁰s} is at the {:!¹⁰s}'.format(thing, place)
'The !!wraith!! is at the !!window!!'
```

## 最新的风格：f-strings

*f-strings*出现在 Python 3.6 中，现在是推荐的字符串格式化方式。

制作 f-string：

+   直接在初始引号之前输入字母`f`或`F`。

+   在大括号（`{}`）中包含变量名或表达式，以将它们的值放入字符串中。

这就像前一节的“新风格”格式化，但没有`format()`函数，并且格式字符串中没有空括号（`{}`）或位置参数（`{1}`）。

```py
>>> thing = 'wereduck'
>>> place = 'werepond'
>>> f'The {thing} is in the {place}'
'The wereduck is in the werepond'
```

正如我之前提到的，大括号内也允许表达式：

```py
>>> f'The {thing.capitalize()} is in the {place.rjust(20)}'
'The Wereduck is in the             werepond'
```

这意味着在前一节的`format()`中可以做的事情，在主字符串的`{}`内部现在也可以做到。这看起来更容易阅读。

f-strings 使用与新式格式化相同的格式化语言（宽度、填充、对齐），在`':'`之后。

```py
>>> f'The {thing:>20} is in the {place:.²⁰}'
'The             wereduck is in the ......werepond......'
```

从 Python 3.8 开始，f-strings 增加了一个新的快捷方式，当你想打印变量名及其值时非常有帮助。在调试时非常方便。窍门是在 f-string 的`{}`括号内的变量名后面加上一个单独的`=`：

```py
>>> f'{thing =}, {place =}'
thing = 'wereduck', place = 'werepond'
```

名称实际上可以是一个表达式，并且会按字面意思打印出来：

```py
>>> f'{thing[-4:] =}, {place.title() =}'
thing[-4:] = 'duck', place.title() = 'Werepond'
```

最后，`=`后面可以跟着一个`:`和格式化参数，如宽度和对齐方式：

```py
>>> f'{thing = :>4.4}'
thing = 'were'
```

# 更多字符串事项

Python 有比我展示的更多字符串函数。一些将出现在稍后的章节中（尤其是第十二章），但你可以在[标准文档链接](http://bit.ly/py-docs-strings)找到所有细节。

# 即将到来

你会在杂货店找到 Froot Loops，但 Python 循环在下一章的第一个柜台上。

# 待办事项

5.1 将以`m`开头的单词大写：

```py
>>> song = """When an eel grabs your arm,
... And it causes great harm,
... That's - a moray!"""
```

5.2 打印每个列表问题及其正确匹配的答案，格式为：

Q: *问题*

A: *答案*

```py
>>> questions = [
...     "We don't serve strings around here. Are you a string?",
...     "What is said on Father's Day in the forest?",
...     "What makes the sound 'Sis! Boom! Bah!'?"
...     ]
>>> answers = [
...     "An exploding sheep.",
...     "No, I'm a frayed knot.",
...     "'Pop!' goes the weasel."
...     ]
```

5.3 通过旧式格式编写以下诗歌。将字符串`'roast beef'`、`'ham'`、`'head'`和`'clam'`替换为此字符串中的内容：

```py
My kitty cat likes %s,
My kitty cat likes %s,
My kitty cat fell on his %s
And now thinks he's a %s.
```

5.4 使用新式格式化编写一封表单信。将以下字符串保存为`letter`（你将在下一个练习中使用它）：

```py
Dear {salutation} {name},

Thank you for your letter. We are sorry that our {product}
{verbed} in your {room}. Please note that it should never
be used in a {room}, especially near any {animals}.

Send us your receipt and {amount} for shipping and handling.
We will send you another {product} that, in our tests,
is {percent}% less likely to have {verbed}.

Thank you for your support.

Sincerely,
{spokesman}
{job_title}
```

5.5 为字符串变量`'salutation'`、`'name'`、`'product'`、`'verbed'`（过去时动词）、`'room'`、`'animals'`、`'percent'`、`'spokesman'`和`'job_title'`分配值。使用`letter.format()`打印`letter`。

5.6 在公众投票之后为事物命名，出现了一个模式：英国潜艇（Boaty McBoatface）、澳大利亚赛马（Horsey McHorseface）和瑞典火车（Trainy McTrainface）。使用`%`格式化来打印国家集市上的获奖名字，以及鸭子、葫芦和 spitz 的奖品。

5.7 使用`format()`格式化方法做同样的事情。

5.8 再来一次，使用*f strings*。
