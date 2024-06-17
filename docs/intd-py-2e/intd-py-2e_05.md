# 第四章：选择使用 if

> 如果你能保持头脑冷静
> 
> 如果你能保持头脑冷静，当周围的一切都在失控
> 
> 鲁道德·吉卜林，《如果——》

在前面的章节中，你已经看到了许多数据示例，但还没有深入研究过。大多数代码示例都使用交互式解释器，并且很简短。在这一章中，你将学习如何构建 Python *代码*，而不仅仅是数据。

许多计算机语言使用诸如花括号 (`{` 和 `}`) 或关键字如 `begin` 和 `end` 来标记代码段落。在这些语言中，使用一致的缩进习惯是提高代码可读性的良好实践，不仅适合自己阅读，也便于他人理解。甚至有工具可以帮助你使代码排列整齐。

当他设计成为 Python 的语言时，吉多·范罗苏姆决定使用缩进来定义程序结构，避免使用所有那些括号和花括号。Python 在使用*空格*来定义程序结构方面非常不同寻常。这是新手注意到的第一个方面，对有其他语言经验的人来说可能会感觉奇怪。但事实证明，使用 Python 一段时间后，这种方式会变得自然，你甚至不再注意到它。你甚至会习惯于在输入更少的情况下做更多事情。

我们最初的代码示例都是单行代码。让我们先看看如何进行注释和多行命令。

# 使用 `#` 进行注释

*注释* 是程序中被 Python 解释器忽略的文本片段。你可以使用注释来澄清附近的 Python 代码，做笔记提醒自己以后修复问题，或者任何你喜欢的目的。通过使用 `#` 字符标记注释；从该点到当前行末尾的所有内容都是注释。通常你会在单独的一行上看到注释，如下所示：

```py
>>> # 60 sec/min * 60 min/hr * 24 hr/day
>>> seconds_per_day = 86400
```

或者，在代码同一行上进行注释：

```py
>>> seconds_per_day = 86400 # 60 sec/min * 60 min/hr * 24 hr/day
```

`#` 字符有很多名称：*井号*、*sharp*、*pound* 或听起来邪恶的 *octothorpe*。¹ 无论你如何称呼它，² 它的效果仅限于出现在该行的末尾。

Python 没有多行注释。你需要明确地用 `#` 开始每一行或每一节注释：

```py
>>> # I can say anything here, even if Python doesn't like it,
... # because I'm protected by the awesome
... # octothorpe.
...
>>>
```

然而，如果它在文本字符串中，强大的井号将恢复其作为普通旧`#`字符的角色：

```py
>>> print("No comment: quotes make the # harmless.")
No comment: quotes make the # harmless.
```

# 使用 `\` 继续多行

当行长度合理时，程序更易读。推荐（非必须）的最大行长度为 80 个字符。如果你无法在这个长度内表达所有想要说的内容，你可以使用*续行字符*：`\`（反斜杠）。只需在行尾加上 `\`，Python 就会认为你仍然在同一行上。

例如，如果我想要添加前五个数字，我可以一行一行地进行：

```py
>>> sum = 0
>>> sum += 1
>>> sum += 2
>>> sum += 3
>>> sum += 4
>>> sum
10
```

或者，我可以使用续行字符一步到位：

```py
>>> sum = 1 + \
...       2 + \
...       3 + \
...       4
>>> sum
10
```

如果我们在表达式中跳过中间的反斜杠，我们会得到一个异常：

```py
>>> sum = 1 +
  File "<stdin>", line 1
    sum = 1 +
            ^
SyntaxError: invalid syntax
```

这里有一个小技巧——如果你在成对的括号（或方括号或花括号）中间，Python 不会对行结束发出警告：

```py
>>> sum = (
...     1 +
...     2 +
...     3 +
...     4)
>>>
>>> sum
10
```

在第五章中，你还会看到成对的三重引号让你创建多行字符串。

# 与 if、elif 和 else 比较

现在，我们终于迈出了进入编程的第一步，这是一个小小的 Python 程序，检查布尔变量`disaster`的值，并打印相应的注释：

```py
>>> disaster = True
>>> if disaster:
...     print("Woe!")
... else:
...     print("Whee!")
...
Woe!
>>>
```

`if`和`else`行是 Python 的*语句*，用于检查条件（这里是`disaster`的值）是否为布尔`True`值，或者可以评估为`True`。记住，`print()`是 Python 的内置*函数*，用于打印东西，通常打印到屏幕上。

###### 注意

如果你在其他语言中编程过，请注意，对于`if`测试，不需要括号。例如，不要写像`if (disaster == True)`这样的内容（相等操作符`==`在几段后面描述）。但是需要在末尾加上冒号（`:`）。如果像我一样有时会忘记输入冒号，Python 会显示错误消息。

每个`print()`行在其测试下缩进。我使用四个空格来缩进每个子节。虽然你可以使用任何你喜欢的缩进方式，但 Python 期望你在一个部分内保持一致——每行都需要缩进相同的数量，左对齐。推荐的风格，称为[*PEP-8*](http://bit.ly/pep-8)，是使用四个空格。不要使用制表符，也不要混合制表符和空格；这会搞乱缩进计数。

在本节逐渐展开时，我们做了很多事情，我会详细解释：

+   将布尔值`True`赋给名为`disaster`的变量。

+   通过使用`if`和`else`执行*条件比较*。

+   *调用* `print()` *函数*来打印一些文本。

你可以进行嵌套测试，需要多少层都可以：

```py
>>> furry = True
>>> large = True
>>> if furry:
...     if large:
...         print("It's a yeti.")
...     else:
...         print("It's a cat!")
... else:
...     if large:
...         print("It's a whale!")
...     else:
...         print("It's a human. Or a hairless cat.")
...
It's a yeti.
```

在 Python 中，缩进决定了如何配对`if`和`else`部分。我们的第一个测试是检查`furry`。因为`furry`是`True`，Python 进入缩进的`if large`测试。因为我们将`large`设为`True`，`if large`评估为`True`，忽略以下的`else`行。这使得 Python 运行缩进在`if large:`下的行，并打印`It's a yeti.`。

如果有超过两个可能性需要测试，使用`if`来进行第一个测试，`elif`（意为*else if*）来进行中间的测试，`else`用于最后一个：

```py
>>> color = "mauve"
>>> if color == "red":
...     print("It's a tomato")
... elif color == "green":
...     print("It's a green pepper")
... elif color == "bee purple":
...     print("I don't know what it is, but only bees can see it")
... else:
...     print("I've never heard of the color", color)
...
I've never heard of the color mauve
```

在上面的例子中，我们使用`==`操作符进行了相等性测试。这里是 Python 的*比较操作符*：

| 等于 | `==` |
| --- | --- |
| 不等于 | `!=` |
| 小于 | `<` |
| 小于或等于 | `<=` |
| 大于 | `>` |
| 大于或等于 | `>=` |

这些返回布尔值`True`或`False`。让我们看看它们如何工作，但首先，给`x`赋一个值：

```py
>>> x = 7
```

现在，让我们尝试一些测试：

```py
>>> x == 5
False
>>> x == 7
True
>>> 5 < x
True
>>> x < 10
True
```

注意，两个等号 (`==`) 用于*测试相等性*；记住，单个等号 (`=`) 用于给变量赋值。

如果你需要同时进行多个比较，可以使用*逻辑*（或*布尔*）*运算符* `and`、`or` 和 `not` 来确定最终的布尔结果。

逻辑运算符比它们比较的代码块具有较低的*优先级*。这意味着首先计算这些代码块，然后再比较。在这个例子中，因为我们将 `x` 设置为 `7`，`5 < x` 计算为 `True`，`x < 10` 也是 `True`，所以最终我们得到 `True and True`：

```py
>>> 5 < x and x < 10
True
```

正如“优先级”所指出的，避免关于优先级混淆的最简单方法是添加括号：

```py
>>> (5 < x) and (x < 10)
True
```

这里有一些其他的测试：

```py
>>> 5 < x or x < 10
True
>>> 5 < x and x > 10
False
>>> 5 < x and not x > 10
True
```

如果你在一个变量上进行多个 `and` 运算的比较，Python 允许你这样做：

```py
>>> 5 < x < 10
True
```

这与 `5 < x and x < 10` 是一样的。你也可以编写更长的比较：

```py
>>> 5 < x < 10 < 999
True
```

# 什么是真？

如果我们检查的元素不是布尔值，Python 认为什么是 `True` 和 `False`？

一个 `false` 值并不一定需要显式地是布尔 `False`。例如，下面这些都被认为是 `False`：

| 布尔 | `False` |
| --- | --- |
| 空 | `None` |
| 零整数 | `0` |
| 零浮点数 | `0.0` |
| 空字符串 | `''` |
| 空列表 | `[]` |
| 空元组 | `()` |
| 空字典 | `{}` |
| 空集合 | `set()` |

其他任何情况都被认为是 `True`。Python 程序使用这些“真实性”和“虚假性”的定义来检查空数据结构以及 `False` 条件：

```py
>>> some_list = []
>>> if some_list:
...     print("There's something in here")
... else:
...     print("Hey, it's empty!")
...
Hey, it's empty!
```

如果你要测试的是一个表达式而不是一个简单的变量，Python 会评估该表达式并返回一个布尔结果。因此，如果你输入：

```py
if color == "red":
```

Python 评估 `color == "red"`。在我们之前的例子中，我们将字符串 `"mauve"` 分配给 `color`，所以 `color == "red"` 是 `False`，Python 继续下一个测试：

```py
elif color == "green":
```

# 使用 `in` 进行多个比较

假设你有一个字母，并想知道它是否是元音字母。一种方法是编写一个长长的 `if` 语句：

```py
>>> letter = 'o'
>>> if letter == 'a' or letter == 'e' or letter == 'i' \
...     or letter == 'o' or letter == 'u':
...     print(letter, 'is a vowel')
... else:
...     print(letter, 'is not a vowel')
...
o is a vowel
>>>
```

每当你需要进行大量使用 `or` 分隔的比较时，使用 Python 的*成员运算符* `in` 更加 Pythonic。下面是如何使用由元音字符组成的字符串与 `in` 结合来检查元音性：

```py
>>> vowels = 'aeiou'
>>> letter = 'o'
>>> letter in vowels
True
>>> if letter in vowels:
...     print(letter, 'is a vowel')
...
o is a vowel
```

下面是如何在接下来的几章节中详细阅读的一些数据类型的使用示例：

```py
>>> letter = 'o'
>>> vowel_set = {'a', 'e', 'i', 'o', 'u'}
>>> letter in vowel_set
True
>>> vowel_list = ['a', 'e', 'i', 'o', 'u']
>>> letter in vowel_list
True
>>> vowel_tuple = ('a', 'e', 'i', 'o', 'u')
>>> letter in vowel_tuple
True
>>> vowel_dict = {'a': 'apple', 'e': 'elephant',
...               'i': 'impala', 'o': 'ocelot', 'u': 'unicorn'}
>>> letter in vowel_dict
True
>>> vowel_string = "aeiou"
>>> letter in vowel_string
True
```

对于字典，`in` 查看的是键（`:` 的左边），而不是它们的值。

# 新内容：我是海象

在 Python 3.8 中引入了*海象运算符*，它看起来像这样：

```py
*`name`* := *`expression`*
```

看到海象了吗？（像笑脸一样，但更多了一些象牙。）

通常，赋值和测试需要两个步骤：

```py
>>> tweet_limit = 280
>>> tweet_string = "Blah" * 50
>>> diff = tweet_limit - len(tweet_string)
>>> if diff >= 0:
...     print("A fitting tweet")
... else:
...     print("Went over by", abs(diff))
...
A fitting tweet
```

通过我们的新的[分配表达式](https://oreil.ly/fHPtL)，我们可以将这些组合成一个步骤：

```py
>>> tweet_limit = 280
>>> tweet_string = "Blah" * 50
>>> if diff := tweet_limit - len(tweet_string) >= 0:
...     print("A fitting tweet")
... else:
...     print("Went over by", abs(diff))
...
A fitting tweet
```

“海象运算符”还与 `for` 和 `while` 很好地配合，我们将在第六章中详细讨论。

# 即将到来

玩弄字符串，并遇见有趣的字符。

# 要做的事情

4.1 选择一个 1 到 10 之间的数字，并将其赋给变量 `secret`。然后，再选择另一个 1 到 10 之间的数字，并将其赋给变量 `guess`。接下来，编写条件测试（`if`、`else`和`elif`）来打印字符串`'too low'`，如果 `guess` 小于 `secret`，打印`'too high'`，如果 `guess` 大于 `secret`，打印`'just right'`，如果 `guess` 等于 `secret`。

4.2 为变量 `small` 和 `green` 赋值`True`或`False`。编写一些`if`/`else`语句来打印这些选择匹配哪些选项：cherry（樱桃）、pea（豌豆）、watermelon（西瓜）、pumpkin（南瓜）。

¹ 就像那只八脚的绿色*东西*就*在你后面*！

² 请不要打电话给它。它可能会回来。
