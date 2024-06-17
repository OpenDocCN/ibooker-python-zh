# 第十一章。模块、包和好东西

在您从底层向上攀登的过程中，您已经从内置数据类型进展到构建越来越大的数据和代码结构。在本章中，您终于学会如何在 Python 中编写现实的完整程序。您将编写自己的*模块*，并学习如何使用来自 Python 标准库和其他来源的模块。

本书的文本是按层次结构组织的：单词、句子、段落和章节。否则，它会很快变得难以阅读。¹ 代码的组织原则大致相同：数据类型类似于单词；表达式和语句类似于句子；函数类似于段落；模块类似于章节。继续这个类比，在本书中，当我说某些内容将在第八章中解释时，在编程中这就像是引用另一个模块中的代码。

# 模块和 import 语句

我们将在多个文件中创建和使用 Python 代码。*模块*只是包含任何 Python 代码的文件。您不需要做任何特殊处理——任何 Python 代码都可以被其他人用作模块。

我们通过使用 Python 的`import`语句引用其他模块的代码。这使得导入模块中的代码和变量对您的程序可用。

## 导入一个模块

`import`语句的最简单用法是`import` *`module`*，其中*`module`*是另一个 Python 文件的名称，不带*.py*扩展名。

假设你和几个人想要快速解决午餐问题，但又不想进行长时间的讨论，最后总是由最吵闹的那个人决定。让电脑来决定吧！让我们编写一个单一函数的模块，返回一个随机的快餐选择，以及调用该函数并打印选择的主程序。

模块（*fast.py*）显示在示例 11-1 中。

##### 示例 11-1。fast.py

```py
from random import choice

places = ['McDonalds", "KFC", "Burger King", "Taco Bell",
     "Wendys", "Arbys", "Pizza Hut"]

def pick():  # see the docstring below?
    """Return random fast food place"""
    return choice(places)
```

并且示例 11-2 展示了导入它的主程序（称为*lunch.py*）。

##### 示例 11-2。lunch.py

```py
import fast

place = fast.pick()
print("Let's go to", place)
```

如果您将这两个文件放在同一个目录中，并指示 Python 将*lunch.py*作为主程序运行，它将访问`fast`模块并运行其`pick()`函数。我们编写了`pick()`的这个版本，以从字符串列表中返回一个随机结果，因此主程序将获得并打印出这个结果：

```py
$ python lunch.py
Let's go to Burger King
$ python lunch.py
Let's go to Pizza Hut
$ python lunch.py
Let's go to Arbys
```

我们在两个不同的地方使用了导入：

+   主程序*lunch.py*导入了我们的新模块`fast`。

+   模块文件*fast.py*从 Python 的标准库模块`random`中导入了`choice`函数。

我们在主程序和模块中以两种不同的方式使用了导入：

+   在第一种情况下，我们导入了整个 `fast` 模块，但需要使用 `fast` 作为 `pick()` 的前缀。在这个 `import` 语句之后，只要我们在名称前加上 `fast.`，*fast.py* 中的所有内容对主程序都是可用的。通过用模块的名称*限定*模块的内容，我们避免了任何糟糕的命名冲突。其他模块可能有一个 `pick()` 函数，我们不会误调用它。

+   在第二种情况下，我们在一个模块内部，并且知道这里没有其他名为 `choice` 的内容，因此直接从 `random` 模块中导入了 `choice()` 函数。

我们本可以像示例 11-3 中所示那样编写 *fast.py*，在 `pick()` 函数内部导入 `random` 而不是在文件顶部。

##### 示例 11-3\. fast2.py

```py
places = ['McDonalds", "KFC", "Burger King", "Taco Bell",
     "Wendys", "Arbys", "Pizza Hut"]

def pick():
    import random
    return random.choice(places)
```

像编程的许多方面一样，使用最清晰的风格。使用模块限定的名称（`random.choice`）更安全，但需要稍微多输入一些字。

如果导入的代码可能在多个地方使用，请考虑从函数外部导入；如果知道其使用将受限制，请从函数内部导入。有些人喜欢将所有导入都放在文件顶部，以明确代码的所有依赖关系。无论哪种方式都可以。

## 使用另一个名称导入模块

在我们的主 *lunch.py* 程序中，我们调用了 `import fast`。但如果你：

+   还有另一个名为 `fast` 的模块吗？

+   想使用一个更易记的名称吗？

+   门夹到手指想减少打字？

在这些情况下，你可以使用一个*别名*导入，如示例 11-4 所示。让我们使用别名 `f`。

##### 示例 11-4\. fast3.py

```py
import fast as f
place = f.pick()
print("Let's go to", place)
```

## 从一个模块中仅导入你想要的内容

你可以导入整个模块，也可以只导入部分。你刚刚看到了后者：我们只想要 `random` 模块中的 `choice()` 函数。

像模块本身一样，你可以为每个导入的东西使用别名。

让我们再重新做几次 *lunch.py*。首先，从 `fast` 模块中以其原始名称导入 `pick()`（示例 11-5）。

##### 示例 11-5\. fast4.py

```py
from fast import pick
place = pick()
print("Let's go to", place)
```

现在将其导入为 `who_cares`（示例 11-6）。

##### 示例 11-6\. fast5.py

```py
from fast import pick as who_cares
place = who_cares()
print("Let's go to", place)
```

# 包

我们从单行代码、到多行函数、到独立程序、再到同一目录中的多个模块。如果你没有很多模块，同一目录也可以正常工作。

为了使 Python 应用程序能够更好地扩展，你可以将模块组织成称为*包*的文件和模块层次结构。一个包只是包含 *.py* 文件的子目录。而且你可以进行多层次的组织，有目录在其中。

我们刚刚写了一个选择快餐的模块。让我们再添加一个类似的模块来提供人生建议。我们将在当前目录中创建一个名为 *questions.py* 的新主程序。现在在其中创建一个名为 *choices* 的子目录，并将两个模块放入其中——*fast.py* 和 *advice.py*。每个模块都有一个返回字符串的函数。

主程序（*questions.py*）有额外的导入和行（示例 11-7）。

##### 示例 11-7\. questions.py

```py
from sources import fast, advice

print("Let's go to", fast.pick())
print("Should we take out?", advice.give())
```

那个`from sources`让 Python 在当前目录下查找名为*sources*的目录。在*sources*内部，它查找*fast.py*和*advice.py*文件。

第一个模块（*choices/fast.py*）与以前相同的代码，只是移动到了*choices*目录中（示例 11-8）。

##### 示例 11-8\. choices/fast.py

```py
from random import choice

places = ["McDonalds", "KFC", "Burger King", "Taco Bell",
     "Wendys", "Arbys", "Pizza Hut"]

def pick():
    """Return random fast food place"""
    return choice(places)
```

第二个模块（*choices/advice.py*）是新的，但功能与快餐相似（示例 11-9）。

##### 示例 11-9\. choices/advice.py

```py
from random import choice

answers = ["Yes!", "No!", "Reply hazy", "Sorry, what?"]

def give():
    """Return random advice"""
    return choice(answers)
```

###### 注意

如果你的 Python 版本早于 3.3，那么在*sources*子目录中还需要一件事才能使其成为 Python 包：一个名为*\_\_init\_\_.py*的文件。这可以是一个空文件，但是在 3.3 之前的 Python 中，需要这样做才能将包含它的目录视为包。（这是另一个常见的 Python 面试问题。）

运行主程序*questions.py*（从当前目录，而不是*sources*中）来看看会发生什么：

```py
$ python questions.py
Let's go to KFC
Should we take out? Yes!
$ python questions.py
Let's go to Wendys
Should we take out? Reply hazy
$ python questions.py
Let's go to McDonalds
Should we take out? Reply hazy
```

## 模块搜索路径

我刚才说过，Python 会在当前目录下查找子目录*choices*及其模块。实际上，它还会在其他地方查找，并且你可以控制这一过程。

早些时候，我们从标准库的`random`模块导入了函数`choice()`。这不在你的当前目录中，因此 Python 还需要在其他地方查找。

要查看 Python 解释器查找的所有位置，导入标准的`sys`模块并使用它的`path`列表。这是一个目录名称和 ZIP 存档文件列表，Python 按顺序搜索以找到要导入的模块。

你可以访问并修改这个列表。这是我 Mac 上 Python 3.7 的`sys.path`值：

```py
>>> import sys
>>> for place in sys.path:
...     print(place)
...

/Library/Frameworks/Python.framework/Versions/3.7/lib/python37.zip
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/lib-dynload
```

那个初始的空输出行是空字符串`''`，它表示当前目录。如果`''`在`sys.path`的第一位，当你尝试导入某些东西时，Python 首先查找当前目录：`import fast`会寻找*fast.py*。这是 Python 的常规设置。此外，当我们创建名为*sources*的子目录并在其中放置 Python 文件时，它们可以通过`import sources`或`from sources import fast`来导入。

将使用第一个匹配项。这意味着如果你定义了一个名为`random`的模块，并且它在标准库之前的搜索路径中，那么现在就无法访问标准库的`random`了。

你可以在代码中修改搜索路径。假设你希望 Python 在其他任何位置之前查找*/my/modules*目录：

```py
>>> import sys
>>> sys.path.insert(0, "/my/modules")
```

## 相对和绝对导入

到目前为止，在我们的示例中，我们从以下位置导入了我们自己的模块：

+   当前目录

+   子目录*choices*

+   Python 标准库

这在你有与标准模块同名的本地模块时效果很好。你想要哪一个？

Python 支持*绝对*或*相对*导入。到目前为止你看到的例子都是绝对导入。如果你键入`import rougarou`，对于搜索路径中的每个目录，Python 会查找名为*rougarou.py*（一个模块）或名为*rougarou*（一个包）的文件。

+   如果*rougarou.py*与你调用问题的同一目录中，你可以用`from . import rougarou`来相对于你所在位置导入它。

+   如果它位于你的上一级目录中：`from .. import rougarou`。

+   如果它位于名为`creatures`的同级目录下：`from ..creatures import rougarou`。

`.`和`..`的符号借鉴于 Unix 对*当前目录*和*父目录*的简写。

对于 Python 导入系统中可能遇到的问题的深入讨论，请参阅[Python 导入系统的陷阱](https://oreil.ly/QMWHY)。

## 命名空间包

你已经看到可以将 Python 模块打包为：

+   一个单一的*模块*（*.py* 文件）

+   *包*（包含模块及可能其他包的目录）

你也可以通过*命名空间包*在多个目录中分割一个包。假设你想要一个名为*critters*的包，其中包含每种危险生物（真实或想象中，据说具有背景信息和防护提示）的 Python 模块。随着时间的推移，这可能会变得很庞大，你可能希望按地理位置细分。一种选择是在*critters*下添加位置子包，并将现有的*.py*模块文件移到它们下面，但这会打破其他导入它们的模块。相反，我们可以向*上*进行如下操作：

+   在*critters*上创建新的位置目录

+   在这些新父目录下创建表兄弟目录*critters*

+   将现有模块移动到它们各自的目录中。

这需要一些说明。假设我们从这样的文件布局开始：

```py
critters
 ⌞ rougarou.py
 ⌞ wendigo.py
```

这些模块的正常导入看起来像这样：

```py
from critters import wendigo, rougarou
```

现在如果我们决定在美国的*北*和*南*地点，文件和目录看起来会像这样：

```py
north
 ⌞ critters
   ⌞ wendigo.py
south
 ⌞ critters
   ⌞ rougarou.py
```

如果*north*和*south*都在你的模块搜索路径中，你可以像它们仍然共存于单目录包一样导入这些模块：

```py
from critters import wendigo, rougarou
```

## 模块与对象

你应该把你的代码放入一个模块中，还是放入一个对象中？什么时候适合？

它们在许多方面看起来很相似。一个名为`thing`的对象或模块，具有称为`stuff`的内部数据值，让你可以像`thing.stuff`那样访问该值。`stuff`可能在创建模块或类时已经定义，也可能是后来分配的。

模块中的所有类、函数和全局变量对外部都是可用的。对象可以使用属性和“dunder”（`__` …）命名来隐藏或控制对它们数据属性的访问。

这意味着你可以这样做：

```py
>>> import math
>>> math.pi
3.141592653589793
>>> math.pi = 3.0
>>> math.pi
3.0
```

你刚刚搞砸了这台计算机上每个人的计算吗？是的！不，开玩笑的。² 这并未影响 Python 的 `math` 模块。你只是改变了你调用程序导入的 `math` 模块代码的副本中 `pi` 的值，并且所有关于你罪行的证据将在程序结束时消失。

你的程序导入的任何模块只有一个副本，即使你多次导入它。你可以用它保存对任何导入代码感兴趣的全局事物。这与类似，尽管你可以从中创建许多对象，但类也只有一个副本。

# Python 标准库中的好东西

Python 的一个显著特点是其“即插即用”——一个包含许多有用任务的大型标准库模块。它们被保持分开，以避免膨胀核心语言。当你打算写一些 Python 代码时，经常值得先检查是否已经有标准模块实现了你想要的功能。令人惊讶的是，你会经常遇到标准库中的一些小宝石。Python 还为这些模块提供了权威的 [文档](http://docs.python.org/3/library)，以及一个 [教程](http://bit.ly/library-tour)。Doug Hellmann 的网站 [Python Module of the Week](http://bit.ly/py-motw) 和书籍 [*The Python Standard Library by Example*](http://bit.ly/py-libex)（Addison-Wesley Professional）也是非常有用的指南。

本书的即将到来的章节涵盖了许多特定于网络、系统、数据库等的标准模块。在本节中，我讨论一些具有通用用途的标准模块。

## 使用 `setdefault()` 和 `defaultdict()` 处理缺失键

你已经看到尝试访问字典的不存在键会引发异常。使用字典的 `get()` 函数返回一个默认值可以避免异常。`setdefault()` 函数类似于 `get()`，但也会在键缺失时向字典中分配一个项目：

```py
>>> periodic_table = {'Hydrogen': 1, 'Helium': 2}
>>> periodic_table
{'Hydrogen': 1, 'Helium': 2}
```

如果键原先 *不* 在字典中，新值就会被使用：

```py
>>> carbon = periodic_table.setdefault('Carbon', 12)
>>> carbon
12
>>> periodic_table
{'Hydrogen': 1, 'Helium': 2, 'Carbon': 12}
```

如果我们尝试为 *现有* 键分配不同的默认值，将返回原始值且不会发生任何更改：

```py
>>> helium = periodic_table.setdefault('Helium', 947)
>>> helium
2
>>> periodic_table
{'Hydrogen': 1, 'Helium': 2, 'Carbon': 12}
```

`defaultdict()` 类似，但在创建字典时就指定了任何新键的默认值。它的参数是一个函数。在这个例子中，我们传递了函数 `int`，它将被调用为 `int()` 并返回整数 `0`：

```py
>>> from collections import defaultdict
>>> periodic_table = defaultdict(int)
```

现在任何缺失的值将是整数 (`int`), 其值为 `0`:

```py
>>> periodic_table['Hydrogen'] = 1
>>> periodic_table['Lead']
0
>>> periodic_table
defaultdict(<class 'int'>, {'Hydrogen': 1, 'Lead': 0})
```

`defaultdict()` 的参数是一个返回要分配给缺失键的值的函数。在下面的例子中，当需要时执行 `no_idea()` 来返回一个值：

```py
>>> from collections import defaultdict
>>>
>>> def no_idea():
...     return 'Huh?'
...
>>> bestiary = defaultdict(no_idea)
>>> bestiary['A'] = 'Abominable Snowman'
>>> bestiary['B'] = 'Basilisk'
>>> bestiary['A']
'Abominable Snowman'
>>> bestiary['B']
'Basilisk'
>>> bestiary['C']
'Huh?'
```

你可以使用 `int()`、`list()` 或 `dict()` 函数来返回这些类型的默认空值：`int()` 返回 `0`，`list()` 返回一个空列表 (`[]`)，`dict()` 返回一个空字典 (`{}`)。如果省略参数，新键的初始值将设置为 `None`。

顺便说一下，您可以使用`lambda`在调用内部定义您的默认生成函数：

```py
>>> bestiary = defaultdict(lambda: 'Huh?')
>>> bestiary['E']
'Huh?'
```

使用`int`是制作自己的计数器的一种方法：

```py
>>> from collections import defaultdict
>>> food_counter = defaultdict(int)
>>> for food in ['spam', 'spam', 'eggs', 'spam']:
...     food_counter[food] += 1
...
>>> for food, count in food_counter.items():
...     print(food, count)
...
eggs 1
spam 3
```

在上面的示例中，如果`food_counter`是一个普通的字典而不是`defaultdict`，每次尝试增加字典元素`food_counter[food]`时，Python 都会引发一个异常，因为它不会被初始化。我们需要做一些额外的工作，如下所示：

```py
>>> dict_counter = {}
>>> for food in ['spam', 'spam', 'eggs', 'spam']:
...     if not food in dict_counter:
...         dict_counter[food] = 0
...     dict_counter[food] += 1
...
>>> for food, count in dict_counter.items():
...     print(food, count)
...
spam 3
eggs 1
```

## 使用 Counter()计算项数

说到计数器，标准库中有一个可以执行前面示例工作以及更多工作的计数器：

```py
>>> from collections import Counter
>>> breakfast = ['spam', 'spam', 'eggs', 'spam']
>>> breakfast_counter = Counter(breakfast)
>>> breakfast_counter
Counter({'spam': 3, 'eggs': 1})
```

函数`most_common()`以降序返回所有元素，或者如果给定了计数，则仅返回前`count`个元素：

```py
>>> breakfast_counter.most_common()
[('spam', 3), ('eggs', 1)]
>>> breakfast_counter.most_common(1)
[('spam', 3)]
```

您可以组合计数器。首先，让我们再次看看`breakfast_counter`中有什么：

```py
>>> breakfast_counter
>>> Counter({'spam': 3, 'eggs': 1})
```

这一次，我们创建了一个名为`lunch`的新列表，以及一个名为`lunch_counter`的计数器：

```py
>>> lunch = ['eggs', 'eggs', 'bacon']
>>> lunch_counter = Counter(lunch)
>>> lunch_counter
Counter({'eggs': 2, 'bacon': 1})
```

我们组合两个计数器的第一种方法是通过加法，使用`+`：

```py
>>> breakfast_counter + lunch_counter
Counter({'spam': 3, 'eggs': 3, 'bacon': 1})
```

正如您所预期的，您可以使用`-`从另一个计数器中减去一个计数器。早餐吃什么而午餐不吃呢？

```py
>>> breakfast_counter - lunch_counter
Counter({'spam': 3})
```

好的，现在我们可以吃午餐了，但是我们早餐不能吃什么呢？

```py
>>> lunch_counter - breakfast_counter
Counter({'bacon': 1, 'eggs': 1})
```

类似于第八章中的集合，您可以使用交集运算符`&`获取共同的项：

```py
>>> breakfast_counter & lunch_counter
Counter({'eggs': 1})
```

交集选择了具有较低计数的共同元素（`'eggs'`）。这是有道理的：早餐只提供了一个鸡蛋，所以这是共同的计数。

最后，您可以使用并集运算符`|`获取所有项：

```py
>>> breakfast_counter | lunch_counter
Counter({'spam': 3, 'eggs': 2, 'bacon': 1})
```

项目`'eggs'`再次是两者共同的。与加法不同，联合操作并未将它们的计数相加，而是选择计数较大的那个。

## 使用 OrderedDict()按键排序

这是使用 Python 2 解释器运行的示例：

```py
>>> quotes = {
...     'Moe': 'A wise guy, huh?',
...     'Larry': 'Ow!',
...     'Curly': 'Nyuk nyuk!',
...     }
>>> for stooge in quotes:
...  print(stooge)
...
Larry
Curly
Moe
```

###### 注意

从 Python 3.7 开始，字典会按照它们被添加的顺序保留键。`OrderedDict`对于早期版本非常有用，因为它们具有不可预测的顺序。本节中的示例仅在您使用的 Python 版本早于 3.7 时才相关。

`OrderedDict()`记住键添加的顺序，并从迭代器中以相同的顺序返回它们。尝试从一个(*键*, *值*)元组序列创建一个`OrderedDict`：

```py
>>> from collections import OrderedDict
>>> quotes = OrderedDict([
...     ('Moe', 'A wise guy, huh?'),
...     ('Larry', 'Ow!'),
...     ('Curly', 'Nyuk nyuk!'),
...     ])
>>>
>>> for stooge in quotes:
...     print(stooge)
...
Moe
Larry
Curly
```

## 栈+队列==deque

一个`deque`（发音为*deck*）是一个双端队列，具有栈和队列的特性。当你想要从序列的任一端添加或删除项时，它非常有用。在这里，我们从单词的两端向中间工作，以查看它是否是回文。函数`popleft()`从 deque 中删除最左边的项并返回它；`pop()`则删除最右边的项并返回它。它们一起从两端向中间工作。只要末尾字符匹配，它就会持续弹出，直到达到中间位置：

```py
>>> def palindrome(word):
...     from collections import deque
...     dq = deque(word)
...     while len(dq) > 1:
...        if dq.popleft() != dq.pop():
...            return False
...     return True
...
...
>>> palindrome('a')
True
>>> palindrome('racecar')
True
>>> palindrome('')
True
>>> palindrome('radar')
True
>>> palindrome('halibut')
False
```

我将其用作双端队列的简单说明。如果你真的想要一个快速的回文检查器，只需将字符串与其反转比较就简单得多了。Python 没有字符串的 `reverse()` 函数，但它确实有一种通过切片来反转字符串的方法，如下例所示：

```py
>>> def another_palindrome(word):
...     return word == word[::-1]
...
>>> another_palindrome('radar')
True
>>> another_palindrome('halibut')
False
```

## 用 itertools 遍历代码结构

[`itertools`](http://bit.ly/py-itertools)包含特殊用途的迭代器函数。每次在`for`…`in`循环中调用时，它返回一个项目，并在调用之间记住其状态。

`chain()` 将其参数视为单个可迭代对象运行：

```py
>>> import itertools
>>> for item in itertools.chain([1, 2], ['a', 'b']):
...     print(item)
...
1
2
a
b
```

`cycle()`是一个无限迭代器，循环遍历其参数：

```py
>>> import itertools
>>> for item in itertools.cycle([1, 2]):
...     print(item)
...
1
2
1
2
.
.
.
```

等等。

`accumulate()` 计算累积值。默认情况下，它计算总和：

```py
>>> import itertools
>>> for item in itertools.accumulate([1, 2, 3, 4]):
...     print(item)
...
1
3
6
10
```

您可以将一个函数作为`accumulate()`的第二个参数提供，它将被用于代替加法。该函数应该接受两个参数并返回一个单一的结果。这个例子计算一个累积乘积：

```py
>>> import itertools
>>> def multiply(a, b):
...     return a * b
...
>>> for item in itertools.accumulate([1, 2, 3, 4], multiply):
...     print(item)
...
1
2
6
24
```

itertools 模块还有许多其他函数，尤其是一些组合和排列函数，在需要时可以节省时间。

## 使用`pprint()`进行漂亮打印

我们所有的例子都使用`print()`（或者在交互式解释器中仅使用变量名）来打印东西。有时，结果很难读取。我们需要一个*漂亮打印机*，比如`pprint()`：

```py
>>> from pprint import pprint
>>> quotes = OrderedDict([
...     ('Moe', 'A wise guy, huh?'),
...     ('Larry', 'Ow!'),
...     ('Curly', 'Nyuk nyuk!'),
...     ])
>>>
```

简单的`print()`只是将东西倒出来：

```py
>>> print(quotes)
OrderedDict([('Moe', 'A wise guy, huh?'), ('Larry', 'Ow!'),
 ('Curly', 'Nyuk nyuk!')])
```

但是，`pprint()`试图对齐元素以提高可读性：

```py
>>> pprint(quotes)
{'Moe': 'A wise guy, huh?',
 'Larry': 'Ow!',
 'Curly': 'Nyuk nyuk!'}
```

## 获取随机数

我们在本章的开头玩了`random.choice()`。它从给定的序列（列表、元组、字典、字符串）中返回一个值：

```py
>>> from random import choice
>>> choice([23, 9, 46, 'bacon', 0x123abc])
1194684
>>> choice( ('a', 'one', 'and-a', 'two') )
'one'
>>> choice(range(100))
68
>>> choice('alphabet')
'l'
```

使用`sample()`函数一次获取多个值：

```py
>>> from random import sample
>>> sample([23, 9, 46, 'bacon', 0x123abc], 3)
[1194684, 23, 9]
>>> sample(('a', 'one', 'and-a', 'two'), 2)
['two', 'and-a']
>>> sample(range(100), 4)
[54, 82, 10, 78]
>>> sample('alphabet', 7)
['l', 'e', 'a', 't', 'p', 'a', 'b']
```

要从任意范围获取一个随机整数，您可以使用`choice()`或`sample()`与`range()`，或者使用`randint()`或`randrange()`：

```py
>>> from random import randint
>>> randint(38, 74)
71
>>> randint(38, 74)
60
>>> randint(38, 74)
61
```

`randrange()`像`range()`一样，有起始（包含）和结束（不包含）整数的参数，还有一个可选的整数步长：

```py
>>> from random import randrange
>>> randrange(38, 74)
65
>>> randrange(38, 74, 10)
68
>>> randrange(38, 74, 10)
48
```

最后，获取一个在 0.0 到 1.0 之间的随机实数（浮点数）：

```py
>>> from random import random
>>> random()
0.07193393312692198
>>> random()
0.7403243673826271
>>> random()
0.9716517846775018
```

# 更多电池：获取其他 Python 代码

有时，标准库没有您需要的功能，或者没有以正确的方式执行。有一个完整的开源、第三方 Python 软件世界。良好的资源包括以下内容：

+   [PyPi](http://pypi.python.org)（也称为奶酪商店，源自老蒙提·派森小品）

+   [GitHub](https://github.com/Python)

+   [readthedocs](https://readthedocs.org)

你可以在[activestate](https://oreil.ly/clMAi)找到许多较小的代码示例。

本书几乎所有的 Python 代码都使用您计算机上的标准 Python 安装，其中包括所有内置函数和标准库。某些地方特别提到了`requests`在第一章中；更多细节请参见第十八章。附录 B 展示了如何安装第三方 Python 软件，以及许多其他开发细节。

# 即将发生的事情

下一章是一个实用章节，涵盖 Python 中许多数据操作的方面。您将遇到二进制*bytes*和*bytearray*数据类型，在文本字符串中处理 Unicode 字符，并使用正则表达式搜索文本字符串。

# 要做的事情

11.1 创建一个名为*zoo.py*的文件。在其中，定义一个名为`hours()`的函数，打印字符串`'Open 9-5 daily'`。然后，使用交互解释器导入`zoo`模块并调用其`hours()`函数。

11.2 在交互解释器中，将`zoo`模块作为`menagerie`导入，并调用其`hours()`函数。

11.3 仍然在解释器中，直接从`zoo`中导入`hours()`函数并调用它。

11.4 将`hours()`函数作为`info`导入并调用它。

11.5 创建一个名为`plain`的字典，其键值对为`'a': 1`，`'b': 2`和`'c': 3`，然后打印它。

11.6 从上一个问题中列出的相同对创建一个名为`fancy`的`OrderedDict`并打印它。它是否按照`plain`的顺序打印？

11.7 创建一个名为`dict_of_lists`的`defaultdict`，并传递`list`作为参数。用一次赋值操作将列表`dict_of_lists['a']`并附加值`'something for a'`。打印`dict_of_lists['a']`。

¹ 至少，比它现在的阅读性少一点。

² 还是会？布娃哈哈。
