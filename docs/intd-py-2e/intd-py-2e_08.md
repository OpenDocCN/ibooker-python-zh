# 第七章。元组和列表

> 人类与低等灵长类动物的区别在于他对列表的热爱。
> 
> H. Allen Smith

在前几章中，我们从 Python 的一些基本数据类型开始：布尔值，整数，浮点数和字符串。如果你把它们看作是原子，那么本章中的数据结构就像分子一样。也就是说，我们将这些基本类型以更复杂的方式组合在一起。你将每天都用到它们。编程的很大一部分就是将数据切割和粘贴成特定形式，而这些就是你的金刚钻和胶枪。

大多数计算机语言可以表示按其整数位置索引的项目序列：第一个，第二个，依此类推直到最后一个。你已经见过 Python 的*字符串*，它们是字符序列。

Python 还有另外两种序列结构：*元组*和*列表*。它们包含零个或多个元素。与字符串不同，元素可以是不同的类型。事实上，每个元素都可以是*任何*Python 对象。这使你可以创建像你喜欢的那样深度和复杂的结构。

为什么 Python 同时包含列表和元组？元组是*不可变*的；当你将元素（仅一次）分配给元组时，它们就成为了固定的部分，不能更改。列表是*可变*的，这意味着你可以兴致勃勃地插入和删除元素。我将展示每种的许多例子，并着重于列表。

# 元组

让我们先把一件事搞清楚。你可能会听到两种不同的*tuple*发音。哪个是正确的？如果你猜错了，是否会被认为是 Python 的冒牌货？别担心。Python 的创造者 Guido van Rossum 在[Twitter 上](http://bit.ly/tupletweet)说过：

> 我在周一/三/五会念作 too-pull，周二/四/六会念作 tub-pull。周日我不谈它们。 :)

## 使用逗号和()创建

创建元组的语法有点不一致，如下面的例子所示。让我们从使用`()`创建一个空元组开始：

```py
>>> empty_tuple = ()
>>> empty_tuple
()
```

要创建一个或多个元素的元组，请在每个元素后面都跟一个逗号。这适用于单元素元组：

```py
>>> one_marx = 'Groucho',
>>> one_marx
('Groucho',)
```

你可以将它们括在括号中，仍然得到相同的元组：

```py
>>> one_marx = ('Groucho',)
>>> one_marx
('Groucho',)
```

这里有一个小陷阱：如果括号中只有一个东西而省略了逗号，你将得不到一个元组，而只是那个东西（在这个例子中是字符串`'Groucho'`）：

```py
>>> one_marx = ('Groucho')
>>> one_marx
'Groucho'
>>> type(one_marx)
<class 'str'>
```

如果有多个元素，请除了最后一个元素外，每个元素后面都跟一个逗号：

```py
>>> marx_tuple = 'Groucho', 'Chico', 'Harpo'
>>> marx_tuple
('Groucho', 'Chico', 'Harpo')
```

Python 在回显元组时包括括号。当你定义一个元组时通常不需要它们，但使用括号会更安全，并且有助于使元组更可见：

```py
>>> marx_tuple = ('Groucho', 'Chico', 'Harpo')
>>> marx_tuple
('Groucho', 'Chico', 'Harpo')
```

在某些情况下，如果逗号可能具有其他用途，则确实需要括号。例如，在这个例子中，你可以只用一个尾随逗号创建并分配一个单元素元组，但你不能将其作为函数的参数传递。

```py
>>> one_marx = 'Groucho',
>>> type(one_marx)
<class 'tuple'>
>>> type('Groucho',)
<class 'str'>
>>> type(('Groucho',))
<class 'tuple'>
```

元组让你一次性赋值多个变量：

```py
>>> marx_tuple = ('Groucho', 'Chico', 'Harpo')
>>> a, b, c = marx_tuple
>>> a
'Groucho'
>>> b
'Chico'
>>> c
'Harpo'
```

有时被称为*元组解包*。

你可以使用元组在一条语句中交换值，而不使用临时变量：

```py
>>> password = 'swordfish'
>>> icecream = 'tuttifrutti'
>>> password, icecream = icecream, password
>>> password
'tuttifrutti'
>>> icecream
'swordfish'
>>>
```

## 使用 tuple()创建

`tuple()`转换函数从其他内容制作元组：

```py
>>> marx_list = ['Groucho', 'Chico', 'Harpo']
>>> tuple(marx_list)
('Groucho', 'Chico', 'Harpo')
```

## 使用`+`组合元组

这类似于组合字符串：

```py
>>> ('Groucho',) + ('Chico', 'Harpo')
('Groucho', 'Chico', 'Harpo')
```

## 使用*复制*项

这就像重复使用`+`一样：

```py
>>> ('yada',) * 3
('yada', 'yada', 'yada')
```

## 比较元组

这与列表比较类似：

```py
>>> a = (7, 2)
>>> b = (7, 2, 9)
>>> a == b
False
>>> a <= b
True
>>> a < b
True
```

## 使用 for 和 in 进行迭代

元组迭代类似于其他类型的迭代：

```py
>>> words = ('fresh','out', 'of', 'ideas')
>>> for word in words:
...     print(word)
...
fresh
out
of
ideas
```

## 修改元组

你不能！与字符串一样，元组是不可变的，因此您不能更改现有元组。就像您之前看到的那样，您可以*连接*（组合）元组以制作新元组，就像您可以连接字符串一样：

```py
>>> t1 = ('Fee', 'Fie', 'Foe')
>>> t2 = ('Flop,')
>>> t1 + t2
('Fee', 'Fie', 'Foe', 'Flop')
```

这意味着您可以看起来修改元组，就像这样：

```py
>>> t1 = ('Fee', 'Fie', 'Foe')
>>> t2 = ('Flop,')
>>> t1 += t2
>>> t1
('Fee', 'Fie', 'Foe', 'Flop')
```

但它不是相同的`t1`。Python 从由`t1`和`t2`指向的原始元组制作了一个新元组，并将名称`t1`指向了这个新元组。您可以使用`id()`查看变量名称何时指向新值：

```py
>>> t1 = ('Fee', 'Fie', 'Foe')
>>> t2 = ('Flop',)
>>> id(t1)
4365405712
>>> t1 += t2
>>> id(t1)
4364770744
```

# 列表

列表适合按其顺序跟踪事物，特别是当顺序和内容可能会变化时。与字符串不同，列表是可变的。您可以就地更改列表，添加新元素，并删除或替换现有元素。相同的值可以在列表中出现多次。

## 使用[]创建

列表由零个或多个元素组成，用逗号分隔，并用方括号括起来：

```py
>>> empty_list = [ ]
>>> weekdays = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']
>>> big_birds = ['emu', 'ostrich', 'cassowary']
>>> first_names = ['Graham', 'John', 'Terry', 'Terry', 'Michael']
>>> leap_years = [2000, 2004, 2008]
>>> randomness = 'Punxsatawney", {"groundhog": "Phil"}, "Feb. 2"}
```

`first_names`列表显示值不需要是唯一的。

###### 注意

如果您只想跟踪唯一值并且不关心顺序，则 Python *set*可能比列表更好。在前面的示例中，`big_birds`可以是一个集合。我们在[第八章中探讨了集合。

## 使用 list()创建或转换

您还可以使用`list()`函数创建一个空列表：

```py
>>> another_empty_list = list()
>>> another_empty_list
[]
```

Python 的`list()`函数还将其他*可迭代*数据类型（如元组、字符串、集合和字典）转换为列表。以下示例将字符串转换为一个字符的字符串列表：

```py
>>> list('cat')
['c', 'a', 't']
```

该示例将元组转换为列表：

```py
>>> a_tuple = ('ready', 'fire', 'aim')
>>> list(a_tuple)
['ready', 'fire', 'aim']
```

## 使用 split()从字符串创建

正如我之前在“使用 split()拆分”中提到的，使用`split()`通过某个分隔符将字符串切割为列表：

```py
>>> talk_like_a_pirate_day = '9/19/2019'
>>> talk_like_a_pirate_day.split('/')
['9', '19', '2019']
```

如果您的原始字符串中有多个连续的分隔符字符串怎么办？好吧，你会得到一个空字符串作为列表项：

```py
>>> splitme = 'a/b//c/d///e'
>>> splitme.split('/')
['a', 'b', '', 'c', 'd', '', '', 'e']
```

如果您使用两个字符的分隔符字符串`//`，则会得到这个结果：

```py
>>> splitme = 'a/b//c/d///e'
>>> splitme.split('//')
>>>
['a/b', 'c/d', '/e']
```

## 通过[ *偏移量* ]获取项

与字符串类似，您可以通过指定其偏移量从列表中提取单个值：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes[0]
'Groucho'
>>> marxes[1]
'Chico'
>>> marxes[2]
'Harpo'
```

同样，与字符串类似，负索引从末尾向后计数：

```py
>>> marxes[-1]
'Harpo'
>>> marxes[-2]
'Chico'
>>> marxes[-3]
'Groucho'
>>>
```

###### 注意

偏移量必须是此列表的有效偏移量，即您先前分配了一个值的位置。如果指定了开始之前或结束之后的偏移量，您将收到一个异常（错误）。这是如果我们尝试获取第六个马克思兄弟（偏移量`5`从`0`开始计数），或倒数第五个会发生的情况：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes[5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```

```py
>>> marxes[-5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```

## 使用切片获取项目

您可以通过使用*切片*提取列表的子序列：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes[0:2]
['Groucho', 'Chico']
```

列表的切片也是一个列表。

与字符串类似，切片可以步进除了一之外的其他值。下一个示例从开始处开始，每次向右移动 2 个位置：

```py
>>> marxes[::2]
['Groucho', 'Harpo']
```

在这里，我们从末尾开始，左移 2 个位置：

```py
>>> marxes[::-2]
['Harpo', 'Groucho']
```

最后，反转列表的窍门：

```py
>>> marxes[::-1]
['Harpo', 'Chico', 'Groucho']
```

这些切片都没有改变`marxes`列表本身，因为我们没有把它们赋给`marxes`。要就地反转列表，请使用`*列表*.reverse()`：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes.reverse()
>>> marxes
['Harpo', 'Chico', 'Groucho']
```

###### 注意

`reverse()`函数改变了列表但不返回其值。

正如你在字符串中看到的，切片可以指定一个无效的索引，但不会导致异常。它会“捕捉”到最接近的有效索引或者返回空：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes[4:]
[]
>>> marxes[-6:]
['Groucho', 'Chico', 'Harpo']
>>> marxes[-6:-2]
['Groucho']
>>> marxes[-6:-4]
[]
```

## 通过 append()在末尾添加项目

添加项目到列表的传统方法是一个接一个地用`append()`将它们添加到末尾。在前面的例子中，我们忘记了 Zeppo，但这没关系，因为列表是可变的，所以我们现在可以添加他：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes.append('Zeppo')
>>> marxes
['Groucho', 'Chico', 'Harpo', 'Zeppo']
```

## 通过 insert()在偏移处添加项目

`append()`函数只在列表末尾添加项目。当你想在列表的任何偏移之前添加项目时，请使用`insert()`。偏移`0`在开头插入。超出列表末尾的偏移会像`append()`一样在末尾插入，所以你不需要担心 Python 抛出异常：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes.insert(2, 'Gummo')
>>> marxes
['Groucho', 'Chico', 'Gummo', 'Harpo']
>>> marxes.insert(10, 'Zeppo')
>>> marxes
['Groucho', 'Chico', 'Gummo', 'Harpo', 'Zeppo']
```

## 用*复制所有项目

在第五章中，你看到你可以用`*`来复制字符串的字符。对列表也同样适用：

```py
>>> ["blah"] * 3
['blah', 'blah', 'blah']
```

## 通过 extend()或者`+`合并列表

你可以通过使用`extend()`将一个列表合并到另一个列表中。假设一个好心的人给了我们一个名为`others`的新马克思斯列表，并且我们想要将它们合并到主要的`marxes`列表中：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
>>> others = ['Gummo', 'Karl']
>>> marxes.extend(others)
>>> marxes
['Groucho', 'Chico', 'Harpo', 'Zeppo', 'Gummo', 'Karl']
```

或者，你可以使用`+`或者`+=`：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
>>> others = ['Gummo', 'Karl']
>>> marxes += others
>>> marxes
['Groucho', 'Chico', 'Harpo', 'Zeppo', 'Gummo', 'Karl']
```

如果我们使用了`append()`，`others`会被添加为*单个*列表项，而不是合并其项目：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
>>> others = ['Gummo', 'Karl']
>>> marxes.append(others)
>>> marxes
['Groucho', 'Chico', 'Harpo', 'Zeppo', ['Gummo', 'Karl']]
```

这再次证明了列表可以包含不同类型的元素。在这种情况下，四个字符串和一个包含两个字符串的列表。

## 通过[offset]更改项目

就像你可以通过偏移获取列表项的值一样，你也可以修改它：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes[2] = 'Wanda'
>>> marxes
['Groucho', 'Chico', 'Wanda']
```

再次，列表偏移需要是这个列表的有效偏移之一。

你不能用这种方式改变字符串中的字符，因为字符串是不可变的。列表是可变的。你可以改变列表包含的项目数量以及项目本身。

## 使用切片更改项目

前一节展示了如何使用切片获取子列表。你也可以使用切片为子列表赋值：

```py
>>> numbers = [1, 2, 3, 4]
>>> numbers[1:3] = [8, 9]
>>> numbers
[1, 8, 9, 4]
```

您分配给列表的右侧对象甚至不需要与左侧切片具有相同数量的元素：

```py
>>> numbers = [1, 2, 3, 4]
>>> numbers[1:3] = [7, 8, 9]
>>> numbers
[1, 7, 8, 9, 4]
```

```py
>>> numbers = [1, 2, 3, 4]
>>> numbers[1:3] = []
>>> numbers
[1, 4]
```

实际上，右边的东西甚至不需要是一个列表。任何 Python*可迭代对象*都可以，分离其项目并将其分配给列表元素：

```py
>>> numbers = [1, 2, 3, 4]
>>> numbers[1:3] = (98, 99, 100)
>>> numbers
[1, 98, 99, 100, 4]
```

```py
>>> numbers = [1, 2, 3, 4]
>>> numbers[1:3] = 'wat?'
>>> numbers
[1, 'w', 'a', 't', '?', 4]
```

## 通过 del 按偏移删除项目

我们的事实核查员刚刚告诉我们，Gummo 确实是马克思兄弟之一，但卡尔不是，并且早先插入他的人非常无礼。让我们来修复一下：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Gummo', 'Karl']
>>> marxes[-1]
'Karl'
>>> del marxes[-1]
>>> marxes
['Groucho', 'Chico', 'Harpo', 'Gummo']
```

当你按列表中的位置删除一个项目时，随后的项目会向后移动以填补删除项目的空间，并且列表的长度会减少一个。如果我们从`marxes`列表的最后一个版本中删除了`'Chico'`，我们会得到这样的结果：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Gummo']
>>> del marxes[1]
>>> marxes
['Groucho', 'Harpo', 'Gummo']
```

###### 注意

`del` 是一个 Python *语句*，不是列表的方法 —— 你不会说 `marxes[-1].del()`。这与赋值 (`=`) 的相反：它会将一个名称从 Python 对象中分离出来，如果该名称是对该对象的最后一个引用，则可以释放该对象的内存。

## 使用`remove()`按值删除项目

如果不确定或不关心项目在列表中的位置，请使用`remove()`按值删除它。再见，Groucho：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes.remove('Groucho')
>>> marxes
['Chico', 'Harpo']
```

如果列表中有相同值的重复项目，`remove()`仅删除找到的第一个。

## 使用偏移获取项目并使用`pop()`删除它

你可以通过使用`pop()`从列表中获取项目并同时删除它。如果使用偏移调用`pop()`，它将返回该偏移量处的项目；如果没有参数，则使用`-1`。因此，`pop(0)`返回列表的头部（起始处），而`pop()`或`pop(-1)`返回尾部（结束处），如下所示：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
>>> marxes.pop()
'Zeppo'
>>> marxes
['Groucho', 'Chico', 'Harpo']
>>> marxes.pop(1)
'Chico'
>>> marxes
['Groucho', 'Harpo']
```

###### 注意

现在是计算机术语时间！别担心，这些不会出现在期末考试中。如果你使用`append()`在末尾添加新项目，并使用`pop()`从同一端移除它们，你就实现了一种称为 *LIFO*（后进先出）队列的数据结构。这更常被称为 *栈*。`pop(0)`将创建一个 *FIFO*（先进先出）队列。当你希望按到达顺序收集数据并首先使用最旧的数据（FIFO），或者首先使用最新的数据（LIFO）时，这些非常有用。

## 使用`clear()`删除所有项目

Python 3.3 引入了清空列表所有元素的方法：

```py
>>> work_quotes = ['Working hard?', 'Quick question!', 'Number one priorities!']
>>> work_quotes
['Working hard?', 'Quick question!', 'Number one priorities!']
>>> work_quotes.clear()
>>> work_quotes
[]
```

## 使用`index()`按值查找项目的偏移量

如果想知道列表中项目按其值的偏移量，使用`index()`：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
>>> marxes.index('Chico')
1
```

如果该值在列表中出现多次，只返回第一个的偏移量：

```py
>>> simpsons = ['Lisa', 'Bart', 'Marge', 'Homer', 'Bart']
>>> simpsons.index('Bart')
1
```

## 使用`in`测试值是否存在

在列表中检查值是否存在的 Python 方式是使用`in`：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo', 'Zeppo']
>>> 'Groucho' in marxes
True
>>> 'Bob' in marxes
False
```

同一个值可能在列表中出现多次。只要至少出现一次，`in` 就会返回 `True`：

```py
>>> words = ['a', 'deer', 'a' 'female', 'deer']
>>> 'deer' in words
True
```

###### 注意

如果经常检查列表中某个值的存在性，并且不关心项目的顺序，Python *set* 是存储和查找唯一值的更合适方式。我们在第八章中讨论了集合。

## 使用`count()`计算值的出现次数

要统计列表中特定值出现的次数，请使用`count()`：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> marxes.count('Harpo')
1
>>> marxes.count('Bob')
0
```

```py
>>> snl_skit = ['cheeseburger', 'cheeseburger', 'cheeseburger']
>>> snl_skit.count('cheeseburger')
3
```

## 使用`join()`将列表转换为字符串

“使用 join()组合” 更详细地讨论了`join()`，这里是另一个示例：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> ', '.join(marxes)
'Groucho, Chico, Harpo'
```

你可能会认为这似乎有点反向。`join()`是一个字符串方法，而不是一个列表方法。你不能说`marxes.join(', ')`，即使它看起来更直观。`join()`的参数是一个字符串或任何可迭代的字符串序列（包括列表），其输出是一个字符串。如果`join()`只是一个列表方法，你不能将其与其他可迭代对象如元组或字符串一起使用。如果你确实希望它能处理任何可迭代类型，你需要为每种类型编写特殊的代码来处理实际的连接。记住——`join()` *与* `split()`*是相反的*，如下所示：

```py
>>> friends = ['Harry', 'Hermione', 'Ron']
>>> separator = ' * '
>>> joined = separator.join(friends)
>>> joined
'Harry * Hermione * Ron'
>>> separated = joined.split(separator)
>>> separated
['Harry', 'Hermione', 'Ron']
>>> separated == friends
True
```

## 使用`sort()`或`sorted()`重新排序项目

通常需要按值而不是偏移量对列表中的项目进行排序。Python 提供了两个函数：

+   列表方法`sort()`会*原地*对列表进行排序。

+   通用函数`sorted()`返回列表的已排序*副本*。

如果列表中的项是数字，则默认按升序数字顺序排序。如果它们是字符串，则按字母顺序排序：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> sorted_marxes = sorted(marxes)
>>> sorted_marxes
['Chico', 'Groucho', 'Harpo']
```

`sorted_marxes`是一个新列表，创建它并未改变原始列表：

```py
>>> marxes
['Groucho', 'Chico', 'Harpo']
```

但在`marxes`列表上调用列表函数`sort()`确实会改变`marxes`：

```py
>>> marxes.sort()
>>> marxes
['Chico', 'Groucho', 'Harpo']
```

如果列表的元素都是相同类型的（例如在`marxes`中的字符串），`sort()`将正常工作。有时甚至可以混合类型——例如整数和浮点数——因为 Python 在表达式中会自动转换它们：

```py
>>> numbers = [2, 1, 4.0, 3]
>>> numbers.sort()
>>> numbers
[1, 2, 3, 4.0]
```

默认排序顺序是升序，但可以添加参数`reverse=True`将其设置为降序：

```py
>>> numbers = [2, 1, 4.0, 3]
>>> numbers.sort(reverse=True)
>>> numbers
[4.0, 3, 2, 1]
```

## 使用`len()`获取长度

`len()`返回列表中的项数：

```py
>>> marxes = ['Groucho', 'Chico', 'Harpo']
>>> len(marxes)
3
```

## 使用`=`分配

当你将一个列表分配给多个变量时，在一个地方改变列表也会在另一个地方改变，如下所示：

```py
>>> a = [1, 2, 3]
>>> a
[1, 2, 3]
>>> b = a
>>> b
[1, 2, 3]
>>> a[0] = 'surprise'
>>> a
['surprise', 2, 3]
```

那么现在`b`里面是什么？它还是`[1, 2, 3]`，还是`['surprise', 2, 3]`？让我们看看：

```py
>>> b
['surprise', 2, 3]
```

记住第二章中的盒子（对象）和带有注释的字符串（变量名）类比？`b`只是引用与`a`相同的列表对象（两个名称字符串引导到同一个对象盒子）。无论我们使用名称`a`还是`b`改变列表内容，都会反映在两者上：

```py
>>> b
['surprise', 2, 3]
>>> b[0] = 'I hate surprises'
>>> b
['I hate surprises', 2, 3]
>>> a
['I hate surprises', 2, 3]
```

## 使用`copy()`、`list()`或切片复制

你可以通过以下任一方法将列表的值复制到独立的新列表中：

+   列表`copy()`方法

+   `list()`转换函数

+   列表切片`[:]`

我们的原始列表将再次是`a`。我们用列表`copy()`函数制作`b`，用`list()`转换函数制作`c`，用列表切片制作`d`：

```py
>>> a = [1, 2, 3]
>>> b = a.copy()
>>> c = list(a)
>>> d = a[:]
```

同样，`b`、`c`和`d`是`a`的*副本*：它们是具有自己值且与原始列表对象`[1, 2, 3]`没有连接的新对象。改变`a`不会影响副本`b`、`c`和`d`：

```py
>>> a[0] = 'integer lists are boring'
>>> a
['integer lists are boring', 2, 3]
>>> b
[1, 2, 3]
>>> c
[1, 2, 3]
>>> d
[1, 2, 3]
```

## 使用`deepcopy()`复制所有内容

如果列表的值全部是不可变的，`copy()` 函数可以很好地工作。正如之前所见，可变值（如列表、元组或字典）是引用。对原始对象或副本的更改将反映在两者中。

让我们使用前面的例子，但将列表 `a` 中的最后一个元素更改为列表 `[8, 9]` 而不是整数 `3`：

```py
>>> a = [1, 2, [8, 9]]
>>> b = a.copy()
>>> c = list(a)
>>> d = a[:]
>>> a
[1, 2, [8, 9]]
>>> b
[1, 2, [8, 9]]
>>> c
[1, 2, [8, 9]]
>>> d
[1, 2, [8, 9]]
```

目前为止一切顺利。现在更改 `a` 中的子列表中的一个元素：

```py
>>> a[2][1] = 10
>>> a
[1, 2, [8, 10]]
>>> b
[1, 2, [8, 10]]
>>> c
[1, 2, [8, 10]]
>>> d
[1, 2, [8, 10]]
```

现在，`a[2]` 的值是一个列表，它的元素可以被改变。我们使用的所有列表复制方法都是 *浅复制*（不是价值判断，而是深度判断）。

要修复这个问题，我们需要使用 `deepcopy()` 函数：

```py
>>> import copy
>>> a = [1, 2, [8, 9]]
>>> b = copy.deepcopy(a)
>>> a
[1, 2, [8, 9]]
>>> b
[1, 2, [8, 9]]
>>> a[2][1] = 10
>>> a
[1, 2, [8, 10]]
>>> b
[1, 2, [8, 9]]
```

`deepcopy()` 可以处理深度嵌套的列表、字典和其他对象。

你将在第九章更多地了解 `import`。

## 比较列表

你可以直接使用比较运算符如 `==`、`<` 等来比较列表。这些运算符遍历两个列表，比较相同偏移量的元素。如果列表 `a` 比列表 `b` 短，并且所有元素都相等，则 `a` 小于 `b`：

```py
>>> a = [7, 2]
>>> b = [7, 2, 9]
>>> a == b
False
>>> a <= b
True
>>> a < b
True
```

## 使用 for 和 in 迭代

在第六章中，你看到了如何使用 `for` 迭代字符串，但更常见的是迭代列表：

```py
>>> cheeses = ['brie', 'gjetost', 'havarti']
>>> for cheese in cheeses:
...     print(cheese)
...
brie
gjetost
havarti
```

与以前一样，`break` 结束 `for` 循环，`continue` 跳到下一个迭代：

```py
>>> cheeses = ['brie', 'gjetost', 'havarti']
>>> for cheese in cheeses:
...     if cheese.startswith('g'):
...         print("I won't eat anything that starts with 'g'")
...         break
...     else:
...         print(cheese)
...
brie
I won't eat anything that starts with 'g'
```

如果 `for` 循环完成而没有 `break`，你仍然可以使用可选的 `else`：

```py
>>> cheeses = ['brie', 'gjetost', 'havarti']
>>> for cheese in cheeses:
...     if cheese.startswith('x'):
...         print("I won't eat anything that starts with 'x'")
...         break
...     else:
...         print(cheese)
... else:
...     print("Didn't find anything that started with 'x'")
...
brie
gjetost
havarti
Didn't find anything that started with 'x'
```

如果初始的 `for` 从未运行，则控制也转到 `else`：

```py
>>> cheeses = []
>>> for cheese in cheeses:
...     print('This shop has some lovely', cheese)
...     break
... else:  # no break means no cheese
...     print('This is not much of a cheese shop, is it?')
...
This is not much of a cheese shop, is it?
```

因为在这个例子中 `cheeses` 列表为空，所以 `for cheese in cheeses` 从未完成过单个循环，它的 `break` 语句从未执行过。

## 使用 `zip()` 迭代多个序列

还有一个很好的迭代技巧：通过使用 `zip()` 函数并行迭代多个序列：

```py
>>> days = ['Monday', 'Tuesday', 'Wednesday']
>>> fruits = ['banana', 'orange', 'peach']
>>> drinks = ['coffee', 'tea', 'beer']
>>> desserts = ['tiramisu', 'ice cream', 'pie', 'pudding']
>>> for day, fruit, drink, dessert in zip(days, fruits, drinks, desserts):
...     print(day, ": drink", drink, "- eat", fruit, "- enjoy", dessert)
...
Monday : drink coffee - eat banana - enjoy tiramisu
Tuesday : drink tea - eat orange - enjoy ice cream
Wednesday : drink beer - eat peach - enjoy pie
```

`zip()` 在最短的序列结束时停止。 其中一个列表（`desserts`）比其他列表长，因此除非我们扩展其他列表，否则没有人会得到任何布丁。

第八章 展示了 `dict()` 函数如何从包含两个元素的序列（如元组、列表或字符串）创建字典。你可以使用 `zip()` 遍历多个序列，并从相同偏移量的项目创建元组。让我们创建两个对应的英文和法文单词的元组：

```py
>>> english = 'Monday', 'Tuesday', 'Wednesday'
>>> french = 'Lundi', 'Mardi', 'Mercredi'
```

现在，使用 `zip()` 将这些元组配对。`zip()` 返回的值本身不是元组或列表，而是一个可迭代的值，可以转换为元组：

```py
>>> list( zip(english, french) )
[('Monday', 'Lundi'), ('Tuesday', 'Mardi'), ('Wednesday', 'Mercredi')]
```

将 `zip()` 的结果直接提供给 `dict()`，完成：一个微小的英法词典！

```py
>>> dict( zip(english, french) )
{'Monday': 'Lundi', 'Tuesday': 'Mardi', 'Wednesday': 'Mercredi'}
```

## 利用列表推导式创建列表

你已经了解如何使用方括号或 `list()` 函数创建列表。这里，我们将看看如何使用 *列表推导式* 创建列表，它包含了你刚刚看到的 `for`/`in` 迭代。

你可以像这样逐个项地构建从 `1` 到 `5` 的整数列表：

```py
>>> number_list = []
>>> number_list.append(1)
>>> number_list.append(2)
>>> number_list.append(3)
>>> number_list.append(4)
>>> number_list.append(5)
>>> number_list
[1, 2, 3, 4, 5]
```

或者，你也可以使用迭代器和 `range()` 函数：

```py
>>> number_list = []
>>> for number in range(1, 6):
...     number_list.append(number)
...
>>> number_list
[1, 2, 3, 4, 5]
```

或者，你可以直接将 `range()` 的输出转换为列表：

```py
>>> number_list = list(range(1, 6))
>>> number_list
[1, 2, 3, 4, 5]
```

所有这些方法都是有效的 Python 代码，并且会产生相同的结果。然而，更 Pythonic（而且通常更快）的构建列表的方式是使用*列表推导式*。列表推导式的最简单形式看起来像这样：

```py
[*`expression`* for *`item`* in *`iterable`*]
```

下面是一个列表推导式如何构建整数列表：

```py
>>> number_list = [number for number in range(1,6)]
>>> number_list
[1, 2, 3, 4, 5]
```

在第一行，你需要第一个`number`变量为列表生成值：也就是说，将循环的结果放入`number_list`中。第二个`number`是循环的一部分。为了显示第一个`number`是一个表达式，请尝试这个变体：

```py
>>> number_list = [number-1 for number in range(1,6)]
>>> number_list
[0, 1, 2, 3, 4]
```

列表推导式将循环移到方括号内部。这个推导式示例并不比之前的示例更简单，但你可以做更多。列表推导式可以包含条件表达式，看起来像这样：

```py
[*`expression`* for *`item`*
in *`iterable`* if *`condition`*]
```

让我们创建一个新的推导式，构建一个仅包含`1`到`5`之间奇数的列表（记住`number % 2`对于奇数为`True`，对于偶数为`False`）：

```py
>>> a_list = [number for number in range(1,6) if number % 2 == 1]
>>> a_list
[1, 3, 5]
```

现在，推导式比其传统的对应部分更紧凑了一点：

```py
>>> a_list = []
>>> for number in range(1,6):
...     if number % 2 == 1:
...         a_list.append(number)
...
>>>  a_list
[1, 3, 5]
```

最后，就像可以有嵌套循环一样，对应的推导式中还可以有超过一个`for ...`子句集。为了展示这一点，让我们首先尝试一个简单的嵌套循环并打印结果：

```py
>>> rows = range(1,4)
>>> cols = range(1,3)
>>> for row in rows:
...     for col in cols:
...         print(row, col)
...
1 1
1 2
2 1
2 2
3 1
3 2
```

现在，让我们使用一个推导式，并将其分配给变量`cells`，使其成为一个`(row, col)`元组的列表：

```py
>>> rows = range(1,4)
>>> cols = range(1,3)
>>> cells = [(row, col) for row in rows for col in cols]
>>> for cell in cells:
...     print(cell)
...
(1, 1)
(1, 2)
(2, 1)
(2, 2)
(3, 1)
(3, 2)
```

顺便说一句，你也可以使用*元组解包*从每个元组中获取`row`和`col`值，当你迭代`cells`列表时：

```py
>>> for row, col in cells:
...     print(row, col)
...
1 1
1 2
2 1
2 2
3 1
3 2
```

列表推导式中的`for row ...`和`for col ...`片段也可以有它们自己的`if`测试。

## 列表的列表

列表可以包含不同类型的元素，包括其他列表，如下所示：

```py
>>> small_birds = ['hummingbird', 'finch']
>>> extinct_birds = ['dodo', 'passenger pigeon', 'Norwegian Blue']
>>> carol_birds = [3, 'French hens', 2, 'turtledoves']
>>> all_birds = [small_birds, extinct_birds, 'macaw', carol_birds]
```

那么，作为列表的列表的`all_birds`看起来是什么样子？

```py
>>> all_birds
[['hummingbird', 'finch'], ['dodo', 'passenger pigeon', 'Norwegian Blue'], 'macaw',
[3, 'French hens', 2, 'turtledoves']]
```

让我们看看其中的第一个项目：

```py
>>> all_birds[0]
['hummingbird', 'finch']
```

第一项是一个列表：实际上，它是我们创建`all_birds`时指定的`small_birds`的第一项。你应该能够猜到第二项是什么：

```py
>>> all_birds[1]
['dodo', 'passenger pigeon', 'Norwegian Blue']
```

它是我们指定的第二项，`extinct_birds`。如果我们想要`extinct_birds`的第一项，我们可以通过指定两个索引从`all_birds`中提取它：

```py
>>> all_birds[1][0]
'dodo'
```

`[1]`指的是`all_birds`中第二个项目的列表，而`[0]`指的是该内部列表中的第一个项目。

# 元组与列表

你经常可以在列表的地方使用元组，但它们的功能要少得多——没有`append()`、`insert()`等方法——因为它们创建后无法修改。为什么不在任何地方都使用列表呢？

+   元组使用更少的空间。

+   你不会因为错误而破坏元组项。

+   你可以使用元组作为字典的键（参见第八章）。

+   *命名元组*（参见“命名元组”）可以作为对象的简单替代。

我不会在这里详细讨论元组。在日常编程中，你将更多地使用列表和字典。

# 没有元组推导式

可变类型（列表、字典和集合）有理解式。不可变类型如字符串和元组需要使用其各自章节中列出的其他方法创建。

您可能认为将列表推导的方括号更改为圆括号将创建元组推导。它似乎确实有效，因为如果您键入以下内容，将不会出现异常：

```py
>>> number_thing = (number for number in range(1, 6))
```

括号中的东西完全不同：*生成器推导*，它返回一个 *生成器对象*：

```py
>>> type(number_thing)
<class 'generator'>
```

我将在“生成器”一章中详细讨论生成器。生成器是向迭代器提供数据的一种方式。

# 即将到来

它们如此出色，它们有自己的章节：*字典* 和 *集合*。

# 要做的事情

使用列表和元组与数字（第三章）和字符串（第五章）来表示具有丰富多样性的现实世界元素。

7.1 创建一个名为`years_list`的列表，从您的出生年份开始，直到您五岁生日的年份。例如，如果您出生于 1980 年，则列表将是`years_list = [1980, 1981, 1982, 1983, 1984, 1985]`。如果您不到五岁正在阅读本书，那我也不知道该怎么办。

7.2 `years_list`中哪一年是你三岁生日的那一年？记住，你的第一年是 0 岁。

7.3 `years_list`中哪一年你最大？

7.4 使用这三个字符串作为元素创建名为`things`的列表：`"mozzarella"`、`"cinderella"`、`"salmonella"`。

7.5 将`things`中指向人的元素大写，然后打印列表。它改变了列表中的元素吗？

7.6 将`things`中“cheesy”的元素全部大写，然后打印列表。

7.7 删除`things`中的"disease"元素，收集您的诺贝尔奖，并打印列表。

7.8 创建一个名为`surprise`的列表，其中包含元素`"Groucho"`、`"Chico"`和`"Harpo"`。

7.9 将`surprise`列表的最后一个元素转为小写，反转它，然后大写化。

7.10 使用列表推导创建一个名为`even`的列表，其中包含`range(10)`中的偶数。

7.11 让我们创建一个跳绳打油诗生成器。您将打印一系列两行打油诗。从以下程序片段开始：

```py
start1 = ["fee", "fie", "foe"]
rhymes = [
    ("flop", "get a mop"),
    ("fope", "turn the rope"),
    ("fa", "get your ma"),
    ("fudge", "call the judge"),
    ("fat", "pet the cat"),
    ("fog", "walk the dog"),
    ("fun", "say we're done"),
    ]
start2 = "Someone better"
```

对于`rhymes`中的每个元组（`first`，`second`）：

对于第一行：

+   打印`start1`中的每个字符串，大写化，并跟一个感叹号和一个空格。

+   打印`first`，并将其大写化，然后跟一个感叹号。

对于第二行：

+   打印`start2`和一个空格。

+   打印`second`和一个句点。
