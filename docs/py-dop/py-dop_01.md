# 第一章。DevOps 的 Python 基础知识

DevOps 是软件开发与信息技术运营相结合的领域，在过去的十年中非常热门。传统的软件开发、部署、维护和质量保证之间的界限已经被打破，使得团队更加整合。Python 在传统 IT 运营和 DevOps 中都非常流行，因为它兼具灵活性、强大性和易用性。

Python 编程语言于 1990 年代初公开发布，用于系统管理。在这一领域取得了巨大成功，并广泛应用。Python 是一种通用编程语言，几乎在所有领域都有使用。视觉效果和电影行业都采用了它。最近，它已成为数据科学和机器学习（ML）的事实标准语言。它已经被应用于从航空到生物信息学的各个行业。Python 拥有丰富的工具库，以满足用户广泛的需求。学习整个 Python 标准库（任何 Python 安装都带有的功能）将是一项艰巨的任务。试图学习所有为 Python 生态系统注入活力的第三方包将是一项巨大的工程。好消息是，您不需要做这些事情。通过学习 Python 的一个小子集，您可以成为强大的 DevOps 实践者。

在本章中，我们利用几十年的 Python DevOps 经验，仅教授您需要的语言元素。这些是日常使用的 Python DevOps 部分。它们构成了完成工作的基本工具箱。一旦掌握了这些核心概念，您可以添加更复杂的工具，正如您将在后续章节中看到的那样。

# 安装和运行 Python

如果您想尝试本概述中的代码，则需要安装 Python 3.7 或更高版本（截至本文撰写时的最新版本为 3.8.0），并且可以访问一个 shell。在 macOS X、Windows 和大多数 Linux 发行版中，您可以打开终端应用程序以访问 shell。要查看正在使用的 Python 版本，请打开 shell 并键入 `python` `--version`：

```py
$ python --version
Python 3.8.0
```

Python 安装程序可以直接从 [Python.org 网站](https://www.python.org/downloads) 下载。或者，您可以使用像 Apt、RPM、MacPorts、Homebrew、Chocolatey 或其他许多包管理器。

## Python Shell

运行 Python 的最简单方式是使用内置的交互式解释器。只需在 shell 中键入 `python`。然后可以交互地运行 Python 语句。输入 `exit()` 来退出 shell。

```py
$ python
Python 3.8.0 (default, Sep 23 2018, 09:47:03)
[Clang 9.0.0 (clang-900.0.38)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 1 + 2
3
>>> exit()
```

### Python 脚本

Python 代码通过 *.py* 扩展名的文件运行：

```py
# This is my first Python script
print('Hello world!')
```

将此代码保存到名为*hello.py*的文件中。要调用脚本，在 shell 中运行`python`，然后是文件名：

```py
$ python hello.py
Hello world!
```

Python 脚本是大多数生产 Python 代码的运行方式。

### IPython

除了内置的交互式 shell 外，还有几个第三方交互式 shell 可以运行 Python 代码。其中最受欢迎的之一是 [IPython](https://ipython.org)。IPython 提供*内省*（动态获取对象信息的能力）、语法高亮、特殊的*魔术*命令（我们稍后在本章介绍），以及许多其他功能，使其成为探索 Python 的乐趣所在。要安装 IPython，请使用 Python 包管理器 `pip`：

```py
$ pip install ipython
```

运行类似于在上一节中描述的内置交互式 shell 运行：

```py
$ ipython
Python 3.8.0 (default, Sep 23 2018, 09:47:03)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.5.0 -- An enhanced Interactive Python. Type '?' for help.
In [1]: print('Hello')
Hello

In [2]: exit()
```

## Jupyter 笔记本

iPython 项目的一个分支，Jupyter 项目允许包含文本、代码和可视化的文档。这些文档是结合运行代码、输出和格式化文本的强大工具。Jupyter 使得可以将文档与代码一起交付。它在数据科学领域尤其广受欢迎。以下是安装和运行 Jupyter 笔记本的方法：

```py
$ pip install jupyter
$ jupyter notebook
```

此命令打开一个网页浏览器标签，显示当前工作目录。从这里，您可以打开当前项目中的现有笔记本或创建新的笔记本。

# 过程式编程

如果您对编程有所了解，可能已经听说过面向对象编程（OOP）和函数式编程等术语。这些是用于组织程序的不同架构范式。其中最基本的范式之一，过程式编程，是一个很好的起点。*过程式编程*是按顺序向计算机发出指令：

```py
>>> i = 3
>>> j = i +1
>>> i + j
7
```

正如您在此示例中看到的那样，有三个语句按顺序执行，从第一行到最后一行。每个语句都使用前面语句产生的状态。在本例中，第一个语句将值 3 分配给名为 `i` 的变量。在第二个语句中，使用此变量的值将一个值分配给名为 `j` 的变量，在第三个语句中，从两个变量中的值相加。暂时不用担心这些语句的细节；请注意它们按顺序执行并依赖于前面语句创建的状态。

## 变量

变量是指向某个值的名称。在前面的例子中，变量是 `i` 和 `j`。Python 中的变量可以分配新值：

```py
>>> dog_name = 'spot'
>>> dog_name
'spot'
>>> dog_name = 'rex'
>>> dog_name
'rex'
>>> dog_name = 't-' + dog_name
>>> dog_name
't-rex'
>>>
```

Python 变量使用动态类型。实际上，这意味着它们可以被重新分配给不同类型或类的值：

```py
>>> big = 'large'
>>> big
'large'
>>> big = 1000*1000
>>> big
1000000
>>> big = {}
>>> big
{}
>>>
```

在这里，同一个变量分别设置为字符串、数字和字典。变量可以重新分配为任何类型的值。

## 基本数学

可使用内置数学运算符执行基本的数学运算，如加法、减法、乘法和除法：

```py
>>> 1 + 1
2
>>> 3 - 4
–1
>>> 2*5
10
>>> 2/3
0.6666666666666666
```

请注意，`//` 符号用于整数除法。符号 `**` 表示指数运算，`%` 是取模运算符：

```py
>>> 5/2
2.5
>>> 5//2
2
>>> 3**2
9
>>> 5%2
1
```

## 注释

注释是 Python 解释器忽略的文本。它们对代码的文档化很有用，可以被某些服务用来提供独立的文档。单行注释以`#`开头。单行注释可以从行的开头开始，或者之后的任何地方开始。`#`之后的所有内容都是注释，直到新的换行符出现为止。

```py
 # This is a comment
 1 + 1 # This comment follows a statement
```

多行注释本身被封闭在以`"""`或`'''`开头和结尾的块中：

```py
"""
This statement is a block comment.
It can run for multiple lines
"""

'''
This statement is also a block comment
'''
```

## 内置函数

函数是作为一个单元分组的语句。通过键入函数名，后跟括号来调用函数。如果函数带有参数，则参数出现在括号内。Python 有许多内置函数。其中两个最常用的内置函数是`print`和`range`。

## 打印

`print`函数生成用户程序可以查看的输出。在交互式环境中它不太相关，但在编写 Python 脚本时是一种基本工具。在前面的示例中，`print`函数的参数在脚本运行时作为输出写入：

```py
# This is my first Python script
print("Hello world!")

$ python hello.py
Hello world!
```

`print`可以用于查看变量的值或提供程序状态的反馈。`print`通常将标准输出流输出，并在 shell 中作为程序输出可见。

## 范围

虽然`range`是一个内置函数，但技术上它根本不是一个函数。它是表示数字序列的类型。调用`range()`构造函数时，会返回一个表示数字序列的对象。范围对象逐个数字计数。`range`函数最多接受三个整数参数。如果只有一个参数出现，则序列由从零到该数字（但不包括该数字）的数字表示。如果出现第二个参数，则表示起始点，而不是从 0 开始的默认值。第三个参数可用于指定步长距离，默认为 1。

```py
>>> range(10)
range(0, 10)
>>> list(range(10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list(range(5, 10))
[5, 6, 7, 8, 9]
>>> list(range(5, 10, 3))
[5, 8]
>>>
```

`range`维护一个小的内存占用，即使在长序列中也是如此，因为它只存储起始、停止和步长值。`range`函数可以迭代长序列的数字而不受性能约束。

# 执行控制

Python 有许多构造来控制语句执行的流程。你可以将希望一起运行的语句组成一个代码块。这些代码块可以使用`for`和`while`循环多次运行，或者仅在某些条件下运行，例如`if`语句、`while`循环或`try-except`块。使用这些构造是利用编程力量的第一步。不同的语言使用不同的约定来标记代码块。许多类似 C 语言（Unix 中使用的一种非常有影响力的语言）的语法的语言，使用花括号来定义一组语句的块。在 Python 中，缩进用来表示一个代码块。语句通过缩进分组成为一个执行单元的块。

###### 注意

Python 解释器不在乎您是使用制表符还是空格缩进，只要您保持一致即可。然而，Python 样式指南[PEP-8](https://oreil.ly/b5yU4)建议每级缩进使用四个空格。

## if/elif/else

`if/elif/else`语句是在代码中做出决策分支的常见方式。直接跟在`if`语句后面的代码块会在该语句评估为`True`时运行：

```py
>>> i = 45
>>> if i == 45:
...     print('i is 45')
...
...
i is 45
>>>
```

在这里，我们使用了`==`运算符，如果项目相等则返回`True`，否则返回`False`。可选地，此代码块可以在伴随着`elif`或`else`语句的情况下跟随。对于`elif`语句，只有在`elif`评估为`True`时才会执行此代码块：

```py
>>> i = 35
>>> if i == 45:
...     print('i is 45')
... elif i == 35:
...     print('i is 35')
...
...
i is 35
>>>
```

多个`elif`循环可以连接在一起。如果您熟悉其他语言中的`switch`语句，则这模拟了从多个选择中选择的相同行为。在末尾添加`else`语句会在没有其他条件评估为`True`时运行一个代码块：

```py
>>> i = 0
>>> if i == 45:
...     print('i is 45')
... elif i == 35:
...     print('i is 35')
... elif i > 10:
...     print('i is greater than 10')
... elif i%3 == 0:
...     print('i is a multiple of 3')
... else:
...     print('I don't know much about i...')
...
...
i is a multiple of 3
>>>
```

您可以嵌套`if`语句，创建包含仅在外部`if`语句为`True`时才执行的`if`语句的代码块：

```py
>>> cat = 'spot'
>>> if 's' in cat:
...     print("Found an 's' in a cat")
...     if cat == 'Sheba':
...         print("I found Sheba")
...     else:
...         print("Some other cat")
... else:
...     print(" a cat without 's'")
...
...
Found an 's' in a cat
Some other cat
>>>
```

## for 循环

`for`循环允许您重复执行一组语句（代码块），每个成员的*序列*（有序的项目组）一次。当您迭代序列时，当前项目可以通过代码块访问。循环的最常见用途之一是通过`range`对象迭代来执行固定次数的任务：

```py
>>> for i in range(10):
...     x = i*2
...     print(x)
...
...
0
2
4
6
8
10
12
14
16
18
>>>
```

在本示例中，我们的代码块如下所示：

```py
...     x = i*2
...     print(x)
```

我们重复这段代码 10 次，每次将变量`i`分配给从 0 到 9 的整数序列中的下一个数字。`for`循环可用于迭代 Python 中的任何序列类型。您将在本章后面看到这些内容。

### continue

`continue`语句跳过循环中的一步，直接跳到序列中的下一个项目：

```py
>>> for i in range(6):
...     if i == 3:
...         continue
...     print(i)
...
...
0
1
2
4
5
>>>
```

# while 循环

`while`循环会在条件评估为`True`时重复执行一个代码块：

```py
>>> count = 0
>>> while count < 3:
...     print(f"The count is {count}")
...     count += 1
...
...
The count is 0
The count is 1
The count is 2
>>>
```

定义循环结束的方法至关重要。否则，您将陷入循环直到程序崩溃。处理这种情况的一种方法是定义条件语句，使其最终评估为`False`。另一种模式使用`break`语句来使用嵌套条件退出循环：

```py
>>> count = 0
>>> while True:
...     print(f"The count is {count}")
...     if count > 5:
...         break
...     count += 1
...
...
The count is 0
The count is 1
The count is 2
The count is 3
The count is 4
The count is 5
The count is 6
>>>
```

# 处理异常

异常是一种导致程序崩溃的错误类型，如果不进行处理（捕获），程序将崩溃。使用`try-except`块捕获它们允许程序继续运行。通过在可能引发异常的块中缩进，放置`try`语句在其前面并在其后放置`except`语句，后跟应在错误发生时运行的代码块创建这些块：

```py
>>> thinkers = ['Plato', 'PlayDo', 'Gumby']
>>> while True:
...     try:
...         thinker = thinkers.pop()
...         print(thinker)
...     except IndexError as e:
...         print("We tried to pop too many thinkers")
...         print(e)
...         break
...
...
...
Gumby
PlayDo
Plato
We tried to pop too many thinkers
pop from empty list
>>>
```

有许多内置异常，如`IOError`、`KeyError`和`ImportError`。许多第三方包还定义了它们自己的异常类。它们指示出现了严重问题，因此只有在确信问题对软件不会致命时才值得捕获它们。您可以明确指定将捕获的异常类型。理想情况下，应捕获确切的异常类型（在我们的示例中，这是异常`IndexError`）。

# 内置对象

在此概述中，我们不会涉及面向对象编程。然而，Python 语言提供了许多内置类。

## 什么是对象？

在面向对象编程中，数据或状态与功能一起出现。在使用对象时需要理解的基本概念包括*类实例化*（从类创建对象）和*点语法*（访问对象属性和方法的语法）。类定义了其对象共享的属性和方法，可以将其视为汽车模型的技术图纸。然后可以实例化类以创建实例。实例或对象是基于这些图纸构建的单个汽车。

```py
>>> # Define a class for fancy defining fancy cars
>>> class FancyCar():
...     pass
...
>>> type(FancyCar)
<class 'type'>
>>> # Instantiate a fancy car
>>> my_car = FancyCar()
>>> type(my_car)
<class '__main__.FancyCar'>
```

您在这一点上不需要担心创建自己的类。只需理解每个对象都是类的实例化。

## 对象方法和属性

对象将数据存储在属性中。这些属性是附加到对象或对象类的变量。对象使用*对象方法*（为类中所有对象定义的方法）和*类方法*（附加到类并由类中所有对象共享的方法）定义功能，这些方法是附加到对象的函数。

###### 注意

在 Python 文档中，附加到对象和类的函数称为方法。

这些函数可以访问对象的属性并修改和使用对象的数据。要调用对象的方法或访问其属性之一，使用点语法：

```py
>>> # Define a class for fancy defining fancy cars
>>> class FancyCar():
...     # Add a class variable
...     wheels = 4
...     # Add a method
...     def driveFast(self):
...         print("Driving so fast")
...
...
...
>>> # Instantiate a fancy car
>>> my_car = FancyCar()
>>> # Access the class attribute
>>> my_car.wheels
4
>>> # Invoke the method
>>> my_car.driveFast()
Driving so fast
>>>
```

因此，在我们的`FancyCar`类中定义了一个名为`driveFast`的方法和一个名为`wheels`的属性。当您实例化名为`my_car`的`FancyCar`实例时，可以使用点语法访问属性并调用方法。

## 序列

序列是一组内置类型，包括*列表*、*元组*、*范围*、*字符串*和*二进制*类型。序列表示有序且有限的项目集合。

### 序列操作

有许多操作适用于所有类型的序列。我们在此处介绍了一些最常用的操作。

使用`in`和`not in`运算符可以测试序列中是否存在某个项：

```py
>>> 2 in [1,2,3]
True
>>> 'a' not in 'cat'
False
>>> 10 in range(12)
True
>>> 10 not in range(2, 4)
True
```

您可以通过使用其索引号引用序列的内容。要访问某个索引处的项，请使用带有索引号的方括号作为参数。第一个索引的项在位置 0，第二个在 1，依此类推，直到比项数少一个的数字：

```py
>>> my_sequence = 'Bill Cheatham'
>>> my_sequence[0]
'B'
>>> my_sequence[2]
'l'
>>> my_sequence[12]
'm'
```

可以使用负数从序列的末尾而不是从前面进行索引。最后一项的索引为-1，倒数第二项的索引为-2，依此类推：

```py
>>> my_sequence = "Bill Cheatham"
>>> my_sequence[–1]
'm'
>>> my_sequence[–2]
'a'
>>> my_sequence[–13]
'B'
```

项目的索引来自`index`方法。默认情况下，它返回项目的第一次出现的索引，但可选参数可以定义要搜索的子范围：

```py
>>> my_sequence = "Bill Cheatham"
>>> my_sequence.index('C')
5
>>> my_sequence.index('a')
8
>>> my_sequence.index('a',9, 12)
11
>>> my_sequence[11]
'a'
>>>
```

您可以使用切片从序列生成新序列。切片通过在方括号中调用带有可选的`start`、`stop`和`step`参数来显示：

```py
my_sequence[start:stop:step]
```

`start`是新序列中要使用的第一项的索引，`stop`是超出该点的第一个索引，`step`是项之间的距离。这些参数都是可选的，如果省略则替换为默认值。此语句生成原始序列的副本。`start`的默认值为 0，`stop`的默认值为序列的长度，`step`的默认值为 1。注意，如果步骤未显示，则相应的*:*也可以省略：

```py
>>> my_sequence = ['a', 'b', 'c', 'd', 'e', 'f', 'g']
>>> my_sequence[2:5]
['c', 'd', 'e']
>>> my_sequence[:5]
['a', 'b', 'c', 'd', 'e']
>>> my_sequence[3:]
['d', 'e', 'f', 'g']
>>>
```

负数可用于向后索引：

```py
>>> my_sequence[–6:]
['b', 'c', 'd', 'e', 'f', 'g']
>>> my_sequence[3:–1]
['d', 'e', 'f']
>>>
```

序列共享许多操作以获取有关它们及其内容的信息。`len`返回序列的长度，`min`返回最小成员，`max`返回最大成员，`count`返回特定项的数量。`min`和`max`仅适用于具有可比较项的序列。请记住，这些操作适用于任何序列类型：

```py
>>> my_sequence = [0, 1, 2, 0, 1, 2, 3, 0, 1, 2, 3, 4]
>>> len(my_sequence)
12
>>> min(my_sequence)
0
>>> max(my_sequence)
4
>>> my_sequence.count(1)
3
>>>
```

### 列表

列表是 Python 中最常用的数据结构之一，表示任何类型的有序集合。使用方括号表示列表语法。

函数`list()`可用于创建空列表或基于另一个有限可迭代对象（如另一个序列）的列表：

```py
>>> list()
[]
>>> list(range(10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> list("Henry Miller")
['H', 'e', 'n', 'r', 'y', ' ', 'M', 'i', 'l', 'l', 'e', 'r']
>>>
```

直接使用方括号创建的列表是最常见的形式。在这种情况下，列表中的项需要显式枚举。请记住，列表中的项可以是不同类型的：

```py
>>> empty = []
>>> empty
[]
>>> nine = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> nine
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> mixed = [0, 'a', empty, 'WheelHoss']
>>> mixed
[0, 'a', [], 'WheelHoss']
>>>
```

向列表中添加单个项目的最有效方法是将项目`append`到列表的末尾。一种效率较低的方法`insert`允许您在选择的索引位置插入项目：

```py
>>> pies = ['cherry', 'apple']
>>> pies
['cherry', 'apple']
>>> pies.append('rhubarb')
>>> pies
['cherry', 'apple', 'rhubarb']
>>> pies.insert(1, 'cream')
>>> pies
['cherry', 'cream', 'apple', 'rhubarb']
>>>
```

一个列表的内容可以使用`extend`方法添加到另一个列表中：

```py
>>> pies
['cherry', 'cream', 'apple', 'rhubarb']
>>> desserts = ['cookies', 'paste']
>>> desserts
['cookies', 'paste']
>>> desserts.extend(pies)
>>> desserts
['cookies', 'paste', 'cherry', 'cream', 'apple', 'rhubarb']
>>>
```

从列表中删除最后一项并返回其值的最有效和常见方法是将其`pop`出来。此方法可以提供一个索引参数，从而删除并返回该索引处的项目。这种技术效率较低，因为需要重新索引列表：

```py
>>> pies
['cherry', 'cream', 'apple', 'rhubarb']
>>> pies.pop()
'rhubarb'
>>> pies
['cherry', 'cream', 'apple']
>>> pies.pop(1)
'cream'
>>> pies
['cherry', 'apple']
```

还有一个`remove`方法，用于删除一个项目的第一次出现。

```py
>>> pies.remove('apple')
>>> pies
['cherry']
>>>
```

最有效和成语化的 Python 功能之一，列表推导允许您在一行中使用`for`循环的功能。让我们看一个简单的例子，从一个`for`循环开始，将 0-9 的所有数字平方，并将它们附加到列表中：

```py
>>> squares = []
>>> for i in range(10):
...     squared = i*i
...     squares.append(squared)
...
...
>>> squares
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>>
```

为了用列表推导式替换它，我们做以下操作：

```py
>>> squares = [i*i for i in range(10)]
>>> squares
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>>
```

注意内部代码块的功能先被描述，接着是 `for` 语句。你也可以在列表推导式中加入条件语句，过滤结果：

```py
>>> squares = [i*i for i in range(10) if i%2==0]
>>> squares
[0, 4, 16, 36, 64]
>>>
```

列表推导式的其他技巧包括嵌套和使用多个变量，但这里展示的更为简单的形式是最常见的。

### 字符串

字符串序列类型是一组由引号包围的有序字符集。Python 3 默认使用 *UTF-8* 编码。

可以使用字符串构造方法 `str()` 或直接用引号将文本包裹来创建字符串：

```py
>>> str()
''
>>> "some new string!"
'some new string!'
>>> 'or with single quotes'
'or with single quotes'
```

字符串构造方法可以从其他对象创建字符串：

```py
>>> my_list = list()
>>> str(my_list)
'[]'
```

可以通过在内容周围使用三重引号来创建多行字符串：

```py
>>> multi_line = """This is a
... multi-line string,
... which includes linebreaks.
... """
>>> print(multi_line)
This is a
multi-line string,
which includes linebreaks.
>>>
```

除了所有序列共享的方法之外，字符串还有一些特定于其类别的方法。

用户文本通常会有末尾或开头的空白字符。如果有人在表单中输入 " yes " 而不是 “yes”，通常希望将它们视为相同。Python 字符串提供了专门的 `strip` 方法来处理这种情况。它返回一个去掉开头和结尾空白的字符串。还有方法仅移除字符串左侧或右侧的空白：

```py
>>> input = "  I want more  "
>>> input.strip()
'I want more'
>>> input.rstrip()
'  I want more'
>>> input.lstrip()
'I want more  '
```

另一方面，如果你想要给字符串添加填充，可以使用 `ljust` 或 `rjust` 方法。默认情况下，它们使用空格进行填充，也可以指定一个字符作为填充：

```py
>>> output = 'Barry'
>>> output.ljust(10)
'Barry     '
>>> output.rjust(10, '*')
'*****Barry'
```

有时候你想要将一个字符串分割成子字符串列表。也许你有一个句子想要变成单词列表，或者有一串用逗号分隔的单词字符串。`split` 方法可以将一个字符串分割成多个子字符串的列表。默认情况下，它使用空白作为分隔符。还可以用一个可选的参数来指定其他字符作为分隔符：

```py
>>> text = "Mary had a little lamb"
>>> text.split()
['Mary', 'had', 'a', 'little', 'lamb']
>>> url = "gt.motomomo.io/v2/api/asset/143"
>>> url.split('/')
['gt.motomomo.io', 'v2', 'api', 'asset', '143']
```

可以轻松地从一组字符串中创建新的字符串，并使用 `join` 方法将它们连接成一个单独的字符串。这个方法将一个字符串作为分隔符插入到其他字符串列表之间：

```py
>>> items = ['cow', 'milk', 'bread', 'butter']
>>> " and ".join(items)
'cow and milk and bread and butter'
```

改变文本大小写是常见的操作，无论是为了进行比较而统一大小写，还是为了准备用户使用。Python 字符串有几种方法可以轻松完成这个过程：

```py
>>> name = "bill monroe"
>>> name.capitalize()
'Bill monroe'
>>> name.upper()
'BILL MONROE'
>>> name.title()
'Bill Monroe'
>>> name.swapcase()
'BILL MONROE'
>>> name = "BILL MONROE"
>>> name.lower()
'bill monroe'
```

Python 还提供了方法来了解字符串的内容。无论是检查文本的大小写，还是查看它是否表示一个数字，都有相当多的内置方法可供查询。以下是一些最常用的方法：

```py
>>> "William".startswith('W')
True
>>> "William".startswith('Bill')
False
>>> "Molly".endswith('olly')
True
>>> "abc123".isalnum()
True
>>> "abc123".isalpha()
False
>>> "abc".isalnum()
True
>>> "123".isnumeric()
True
>>> "Sandy".istitle()
True
>>> "Sandy".islower()
False
>>> "SANDY".isupper()
True
```

可以在运行时向字符串中插入内容并控制其格式。你的程序可以在字符串中使用变量的值或其他计算出的内容。这种方法既用于创建用户可用的文本，也用于编写软件日志。

Python 中的旧式字符串格式化形式来自 C 语言的`printf`函数。你可以使用模数运算符`%`将格式化值插入到字符串中。这种技术适用于形式`string % values`，其中 values 可以是单个非元组或多个值的元组。字符串本身必须为每个值都有一个转换说明符。转换说明符至少以`%`开头，后跟表示插入值类型的字符：

```py
>>> "%s + %s = %s" % (1, 2, "Three")
'1 + 2 = Three'
>>>
```

其他格式参数包括转换说明符。例如，你可以控制浮点数`%f`打印的位数：

```py
>>> "%.3f" % 1.234567
'1.235'
```

这种字符串格式化机制多年来一直是 Python 中的主流，你会在遗留代码中遇到它。这种方法具有一些引人注目的特性，例如与其他语言共享语法。但也存在一些缺陷。特别是由于使用序列来保存参数，与显示`tuple`和`dict`对象相关的错误很常见。我们建议采用更新的格式化选项，如字符串`format`方法、模板字符串和 f-strings，既可以避免这些错误，又可以增加代码的简洁性和可读性。

Python 3 引入了一种使用字符串方法`format`格式化字符串的新方法。这种格式化方式已经回溯到 Python 2。此规范使用字符串中的花括号来表示替换字段，而不是旧式格式化的基于模数的转换说明符。插入值变为字符串`format`方法的参数。参数的顺序决定它们在目标字符串中的放置顺序：

```py
>>> '{} comes before {}'.format('first', 'second')
'first comes before second'
>>>
```

你可以在括号中指定索引号，以按不同于参数列表顺序插入值。你还可以通过在多个替换字段中指定相同的索引号来重复值：

```py
>>> '{1} comes after {0}, but {1} comes before {2}'.format('first',
                                                           'second',
                                                           'third')
'second comes after first, but second comes before third'
>>>
```

更强大的功能是可以按名称指定插入值：

```py
>>> '''{country} is an island.
... {country} is off of the coast of
... {continent} in the {ocean}'''.format(ocean='Indian Ocean',
...                                      continent='Africa',
...                                      country='Madagascar')
'Madagascar is an island.
Madagascar is off of the coast of
Africa in the Indian Ocean'
```

这里一个`dict`用于提供基于名称的替换字段的键值：

```py
>>> values = {'first': 'Bill', 'last': 'Bailey'}
>>> "Won't you come home {first} {last}?".format(**values)
"Won't you come home Bill Bailey?"
```

你也可以指定格式规范参数。在这里它们使用`>`和`<`添加左右填充。在第二个示例中，我们指定了用于填充的字符：

```py
>>> text = "|{0:>22}||{0:<22}|"
>>> text.format('O','O')
'|                     O||O                     |'
>>> text = "|{0:<>22}||{0:><22}|"
>>> text.format('O','O')
'|<<<<<<<<<<<<<<<<<<<<<O||O>>>>>>>>>>>>>>>>>>>>>|'
```

使用[格式规范迷你语言](https://oreil.ly/ZOFJg)来进行格式规范。我们的主题还使用了另一种称为*f-strings*的语言类型。

Python 的 f-strings 使用与`format`方法相同的格式化语言，但提供了一个更简单直观的机制来使用它们。f-strings 在第一个引号之前用*f*或*F*标记。与前述的`format`字符串类似，f-strings 使用大括号来标识替换字段。然而，在 f-string 中，替换字段的内容是一个表达式。这种方法意味着它可以引用当前范围内定义的变量或涉及计算：

```py
>>> a = 1
>>> b = 2
>>> f"a is {a}, b is {b}. Adding them results in {a + b}"
'a is 1, b is 2\. Adding them results in 3'
```

与`format`字符串一样，f-字符串中的格式规范位于值表达式后的大括号内，并以`:`开头：

```py
>>> count = 43
>>> f"|{count:5d}"
'|   43'
```

值表达式可以包含嵌套表达式，在父表达式的构造中引用变量和表达式：

```py
>>> padding = 10
>>> f"|{count:{padding}d}"
'|        43'
```

###### 提示

我们强烈建议您在大多数字符串格式化时使用 f-字符串。它们结合了规范迷你语言的强大功能与简单直观的语法。

模板字符串旨在提供简单的字符串替换机制。这些内置方法适用于国际化等任务，其中需要简单的单词替换。它们使用`$`作为替换字符，并在其周围可选地使用大括号。紧跟在`$`后面的字符标识要插入的值。当字符串模板的`substitute`方法执行时，这些名称用于分配值。

###### 注意

当您运行 Python 代码时，内置类型和函数是可用的，但要访问 Python 生态系统中提供的更广泛的功能，您需要使用`import`语句。此方法允许您将 Python 标准库或第三方服务的功能添加到您的环境中。您可以使用`from`关键字选择性地从包中导入部分功能：

```py
>>> from string import Template
>>> greeting = Template("$hello Mark Anthony")
>>> greeting.substitute(hello="Bonjour")
'Bonjour Mark Anthony'
>>> greeting.substitute(hello="Zdravstvuyte")
'Zdravstvuyte Mark Anthony'
>>> greeting.substitute(hello="Nǐn hǎo")
'Nǐn hǎo Mark Anthony'
```

### 字典

除了字符串和列表外，字典可能是 Python 内置类中使用最多的。*Dict*是键到值的映射。使用键查找任何特定值的操作非常高效和快速。键可以是字符串、数字、自定义对象或任何其他不可变类型。

###### 注意

可变对象是指其内容可以就地更改的对象。列表是一个主要的例子；列表的内容可以更改而列表的身份不会改变。字符串是不可变的。每次更改现有字符串的内容时，都会创建一个新字符串。

字典被表示为由大括号包围的逗号分隔的键/值对。键/值对包括键、冒号（*:*）和值。

您可以使用`dict()`构造函数创建一个字典对象。如果没有参数，则创建一个空字典。它还可以接受一系列键/值对作为参数：

```py
>>> map = dict()
>>> type(map)
<class 'dict'>
>>> map
{}
>>> kv_list = [['key-1', 'value-1'], ['key-2', 'value-2']]
>>> dict(kv_list)
{'key-1': 'value-1', 'key-2': 'value-2'}
```

您也可以直接使用大括号创建*dict*：

```py
>>> map = {'key-1': 'value-1', 'key-2': 'value-2'}
>>> map
{'key-1': 'value-1', 'key-2': 'value-2'}
```

您可以使用方括号语法访问与键相关联的值：

```py
>>> map['key-1']
'value-1'
>>> map['key-2']
'value-2'
```

您可以使用相同的语法设置一个值。如果键不在字典中，则将其添加为新条目。如果已存在，则该值将更改为新值：

```py
>>> map
{'key-1': 'value-1', 'key-2': 'value-2'}
>>> map['key-3'] = 'value-3'
>>> map
{'key-1': 'value-1', 'key-2': 'value-2', 'key-3': 'value-3'}
>>> map['key-1'] = 13
>>> map
{'key-1': 13, 'key-2': 'value-2', 'key-3': 'value-3'}
```

如果尝试访问在字典中未定义的键，则会引发`KeyError`异常：

```py
>>> map['key-4']
Traceback (most recent call last):
  File "<input>", line 1, in <module>
    map['key-4']
KeyError: 'key-4'
```

您可以使用我们在序列中看到的`in`语法检查字典中键是否存在。对于字典，它检查键的存在：

```py
>>> if 'key-4' in map:
...     print(map['key-4'])
... else:
...     print('key-4 not there')
...
...
key-4 not there
```

更直观的解决方案是使用`get()`方法。如果在字典中未定义键，则返回提供的默认值。如果未提供默认值，则返回`None`：

```py
>>> map.get('key-4', 'default-value')
'default-value'
```

使用`del`从字典中删除一个键值对：

```py
>>> del(map['key-1'])
>>> map
{'key-2': 'value-2', 'key-3': 'value-3'}
```

`keys()`方法返回一个带有字典键的`dict_keys`对象。`values()`方法返回一个带有字典值的`dict_values`对象，而`items()`方法返回键值对。这个最后的方法对于迭代字典内容非常有用：

```py
>>> map.keys()
dict_keys(['key-1', 'key-2'])
>>> map.values()
dict_values(['value-1', 'value-2'])
>>> for key, value in map.items():
...     print(f"{key}: {value}")
...
...
key-1: value-1
key-2: value-2
```

与列表推导式类似，字典推导式是通过迭代一个序列返回一个字典的一行语句：

```py
>>> letters = 'abcde'
>>> # mapping individual letters to their upper-case representations
>>> cap_map = {x: x.upper() for x in letters}
>>> cap_map['b']
'B'
```

# 函数

您已经看到了一些 Python 内置函数。现在开始编写您自己的函数。记住，*函数*是一种封装代码块的机制。您可以在多个位置重复此代码块的行为，而无需重复代码。您的代码将更有组织性，更易于测试、维护和理解。

## 函数的解剖

函数定义的第一行以关键字`def`开头，后面跟着函数名、括号括起来的函数参数，然后是`:`。函数的其余部分是一个缩进的代码块：

```py
def <FUNCTION NAME>(<PARAMETERS>):
    <CODE BLOCK>
```

如果在缩进块中首先提供使用多行语法的字符串，则它作为文档。使用这些描述函数的作用、参数如何工作以及可以预期返回什么。您会发现这些文档字符串对与代码的未来用户进行交流非常宝贵。各种程序和服务也使用它们来创建文档。提供文档字符串被认为是最佳实践，并强烈推荐使用：

```py
>>> def my_function():
...    '''This is a doc string.
...
...    It should describe what the function does,
...    what parameters work, and what the
...    function returns.
...    '''
```

函数参数出现在函数名后面的括号中。它们可以是位置参数或关键字参数。位置参数使用参数的顺序来分配值：

```py
>>> def positioned(first, second):
...     """Assignment based on order."""
...     print(f"first: {first}")
...     print(f"second: {second}")
...
...
>>> positioned(1, 2)
first: 1
second: 2
>>>
```

使用关键字参数，为每个参数分配一个默认值：

```py
>>> def keywords(first=1, second=2):
...     '''Default values assigned'''
...     print(f"first: {first}")
...     print(f"second: {second}")
...
...
```

当在函数调用时没有传递值时，将使用默认值。关键字参数在函数调用时可以通过名称调用，此时顺序将无关紧要：

```py
>>> keywords(0)
first: 0
second: 2
>>> keywords(3,4)
first: 3
second: 4
>>> keywords(second='one', first='two')
first: two
second: one
```

在使用关键字参数时，所有在关键字参数之后定义的参数必须也是关键字参数。所有函数都返回一个值。使用`return`关键字来设置这个值。如果从函数定义中未设置，则函数返回`None`：

```py
>>> def no_return():
...     '''No return defined'''
...     pass
...
>>> result = no_return()
>>> print(result)
None
>>> def return_one():
...     '''Returns 1'''
...     return 1
...
>>> result = return_one()
>>> print(result)
1
```

## 函数作为对象

函数是对象。它们可以被传递，或者存储在数据结构中。您可以定义两个函数，将它们放入列表中，然后遍历列表以调用它们：

```py
>>> def double(input):
...     '''double input'''
...     return input*2
...
>>> double
<function double at 0x107d34ae8>
>>> type(double)
<class 'function'>
>>> def triple(input):
...     '''Triple input'''
...     return input*3
...
>>> functions = [double, triple]
>>> for function in functions:
...     print(function(3))
...
...
6
9
```

## 匿名函数

当需要创建一个非常有限的函数时，可以使用`lambda`关键字创建一个未命名（匿名）函数。一般情况下，应将它们的使用限制在函数期望小函数作为参数的情况下。在这个例子中，您接受一个列表的列表并对其进行排序。默认的排序机制基于每个子列表的第一个项目进行比较：

```py
>>> items = [[0, 'a', 2], [5, 'b', 0], [2, 'c', 1]]
>>> sorted(items)
[[0, 'a', 2], [2, 'c', 1], [5, 'b', 0]]
```

若要根据除第一个条目之外的其他内容进行排序，可以定义一个返回项目第二个条目的方法，并将其传递给排序函数的`key`参数：

```py
>>> def second(item):
...     '''return second entry'''
...     return item[1]
...
>>> sorted(items, key=second)
[[0, 'a', 2], [5, 'b', 0], [2, 'c', 1]]
```

使用`lambda`关键字，您可以在没有完整函数定义的情况下执行相同的操作。Lambda 使用`lambda`关键字，后跟参数名称、冒号和返回值：

```py
lambda <PARAM>: <RETURN EXPRESSION>
```

使用 lambda 进行排序，首先使用第二个条目，然后使用第三个：

```py
>>> sorted(items, key=lambda item: item[1])
[[0, 'a', 2], [5, 'b', 0], [2, 'c', 1]]
>>> sorted(items, key=lambda item: item[2])
[[5, 'b', 0], [2, 'c', 1], [0, 'a', 2]]
```

在更一般地使用 lambda 时要谨慎，因为如果用作一般函数的替代品，它们可能会创建文档不足且难以阅读的代码。

# 使用正则表达式

反复出现需要在字符串中匹配模式的情况。您可能正在查找日志文件中的标识符，或者检查用户输入的关键字或其他许多情况。您已经看到了使用`in`操作符进行简单模式匹配，或者字符串`.endswith`和`.startswith`方法。要进行更复杂的匹配，您需要更强大的工具。正则表达式，通常称为 regex，是答案。正则表达式使用一系列字符来定义搜索模式。Python 的`re`包提供了类似于 Perl 中的正则表达式操作。`re`模块使用反斜杠（*\\*）来标示匹配中使用的特殊字符。为了避免与常规字符串转义序列混淆，在定义正则表达式模式时建议使用原始字符串。原始字符串在第一个引号前加上*r*。

###### 注意

Python 字符串具有几个转义序列。其中最常见的是换行符`\n`和制表符`\t`。

## 搜索

假设您有来自电子邮件的抄送列表作为文本，并且您想进一步了解谁在此列表中：

```py
In [1]: cc_list = '''Ezra Koenig <ekoenig@vpwk.com>,
 ...: Rostam Batmanglij <rostam@vpwk.com>,
 ...: Chris Tomson <ctomson@vpwk.com,
 ...: Bobbi Baio <bbaio@vpwk.com'''
```

如果您想知道这段文本中是否有一个名称，您可以使用`in`序列成员语法：

```py
In [2]: 'Rostam' in cc_list
Out[2]: True
```

要获得类似的行为，您可以使用`re.search`函数，仅在有匹配时返回`re.Match`对象：

```py
In [3]: import re

In [4]: re.search(r'Rostam', cc_list)
Out[4]: <re.Match object; span=(32, 38), match='Rostam'>
```

您可以将此用作测试成员资格的条件：

```py
>>> if re.search(r'Rostam', cc_list):
...     print('Found Rostam')
...
...
Found Rostam
```

## 字符集

到目前为止，`re`还没有给您使用`in`运算符获得的任何内容。但是，如果您在文本中寻找一个人，但无法记住名字是*Bobbi*还是*Robby*，该怎么办？

使用正则表达式，您可以使用一组字符，其中任何一个都可以出现在某个位置。这些称为字符集。在正则表达式定义中，匹配应选择的字符由方括号括起来。您可以匹配*B*或*R*，接着是*obb*，然后是*i*或*y*：

```py
In [5]: re.search(r'[R,B]obb[i,y]', cc_list)
Out[5]: <re.Match object; span=(101, 106), match='Bobbi'>
```

您可以将逗号分隔的单个字符放入字符集中，也可以使用范围。范围`A–Z`包括所有大写字母；范围`0–9`包括从零到九的数字：

```py
In [6]: re.search(r'Chr[a-z][a-z]', cc_list)
Out [6]: <re.Match object; span=(69, 74), match='Chris'>
```

在正则表达式中，`+`匹配一个或多个项目。括号中的数字匹配精确数量的字符：

```py
In [7]: re.search(r'[A-Za-z]+', cc_list)
Out [7]: <re.Match object; span=(0, 4), match='Ezra'>
In [8]: re.search(r'[A-Za-z]{6}', cc_list)
Out [8]: <re.Match object; span=(5, 11), match='Koenig'>
```

我们可以使用字符集和其他字符的组合构建匹配以匹配电子邮件地址的原始匹配器。*.*字符具有特殊含义。它是一个通配符，匹配任何字符。要匹配实际的*.*字符，您必须使用反斜杠进行转义：

```py
In [9]: re.search(r'[A-Za-z]+@[a-z]+\.[a-z]+', cc_list)
Out[9]: <re.Match object; span=(13, 29), match='ekoenig@vpwk.com'>
```

此示例仅演示了字符集。它并不代表用于电子邮件的生产就绪正则表达式的全部复杂性。

## 字符类

除了字符集外，Python 的`re`还提供字符类。这些是预定义的字符集。一些常用的是`\w`，它等效于`[a-zA-Z0-9_]`，以及`\d`，它等效于`[0-9]`。您可以使用*+*修饰符匹配多个字符：

```py
>>> re.search(r'\w+', cc_list)
<re.Match object; span=(0, 4), match='Ezra'>
```

您还可以用`\w`替换我们原始的电子邮件匹配器：

```py
>>> re.search(r'\w+\@\w+\.\w+', cc_list)
<re.Match object; span=(13, 29), match='ekoenig@vpwk.com'>
```

## 组

您可以使用括号在匹配中定义分组。可以从匹配对象访问这些组。它们按它们出现的顺序编号，零号组为完整匹配：

```py
>>> re.search(r'(\w+)\@(\w+)\.(\w+)', cc_list)
<re.Match object; span=(13, 29), match='ekoenig@vpwk.com'>
>>> matched = re.search(r'(\w+)\@(\w+)\.(\w+)', cc_list)
>>> matched.group(0)
'ekoenig@vpwk.com'
>>> matched.group(1)
'ekoenig'
>>> matched.group(2)
'vpwk'
>>> matched.group(3)
'com'
```

## 命名分组

您还可以通过在组定义中添加`?P<NAME>`为组添加名称。然后可以按名称而不是编号访问组：

```py
>>> matched = re.search(r'(?P<name>\w+)\@(?P<SLD>\w+)\.(?P<TLD>\w+)', cc_list)
>>> matched.group('name')
'ekoenig'
>>> print(f'''name: {matched.group("name")}
... Secondary Level Domain: {matched.group("SLD")}
... Top Level Domain: {matched.group("TLD")}''')
name: ekoenig
Secondary Level Domain: vpwk
Top Level Domain: com
```

## 查找全部

到目前为止，我们演示了只返回找到的第一个匹配项。我们也可以使用`findall`将所有匹配项作为字符串列表返回：

```py
>>> matched = re.findall(r'\w+\@\w+\.\w+', cc_list)
>>> matched
['ekoenig@vpwk.com', 'rostam@vpwk.com', 'ctomson@vpwk.com', 'cbaio@vpwk.com']
>>> matched = re.findall(r'(\w+)\@(\w+)\.(\w+)', cc_list)
>>> matched
[('ekoenig', 'vpwk', 'com'), ('rostam', 'vpwk', 'com'),
 ('ctomson', 'vpwk', 'com'), ('cbaio', 'vpwk', 'com')]
>>> names = [x[0] for x in matched]
>>> names
['ekoenig', 'rostam', 'ctomson', 'cbaio']
```

## 查找迭代器

处理大文本（如日志）时，最好不要一次性处理文本。您可以使用`finditer`方法生成一个*迭代器*对象。此对象处理文本直到找到匹配项然后停止。将其传递给`next`函数返回当前匹配并继续处理直到找到下一个匹配。通过这种方式，您可以单独处理每个匹配项，而无需一次性处理所有输入以节省资源：

```py
>>> matched = re.finditer(r'\w+\@\w+\.\w+', cc_list)
>>> matched
<callable_iterator object at 0x108e68748>
>>> next(matched)
<re.Match object; span=(13, 29), match='ekoenig@vpwk.com'>
>>> next(matched)
<re.Match object; span=(51, 66), match='rostam@vpwk.com'>
>>> next(matched)
<re.Match object; span=(83, 99), match='ctomson@vpwk.com'>
```

迭代器对象`matched`也可以在`for`循环中使用：

```py
>>> matched = re.finditer("(?P<name>\w+)\@(?P<SLD>\w+)\.(?P<TLD>\w+)", cc_list)
>>> for m in matched:
...     print(m.groupdict())
...
...
{'name': 'ekoenig', 'SLD': 'vpwk', 'TLD': 'com'}
{'name': 'rostam', 'SLD': 'vpwk', 'TLD': 'com'}
{'name': 'ctomson', 'SLD': 'vpwk', 'TLD': 'com'}
{'name': 'cbaio', 'SLD': 'vpwk', 'TLD': 'com'}
```

## 替换

除了搜索和匹配外，正则表达式还可用于替换字符串的一部分或全部：

```py
>>> re.sub("\d", "#", "The passcode you entered was  09876")
'The passcode you entered was  #####'
>>> users = re.sub("(?P<name>\w+)\@(?P<SLD>\w+)\.(?P<TLD>\w+)",
                   "\g<TLD>.\g<SLD>.\g<name>", cc_list)
>>> print(users)
Ezra Koenig <com.vpwk.ekoenig>,
Rostam Batmanglij <com.vpwk.rostam>,
Chris Tomson <com.vpwk.ctomson,
Chris Baio <com.vpwk.cbaio
```

## 编译

到目前为止，所有示例都直接在`re`模块上调用方法。这对许多情况是足够的，但如果同一匹配会发生多次，则通过将正则表达式编译成对象可以获得性能增益。这个对象可以重复用于匹配而不需要重新编译：

```py
>>> regex = re.compile(r'\w+\@\w+\.\w+')
>>> regex.search(cc_list)
<re.Match object; span=(13, 29), match='ekoenig@vpwk.com'>
```

正则表达式提供的功能远远超出我们在这里讨论的范围。实际上，有许多书籍专门讨论它们的使用，但是现在您应该准备好处理大多数基本情况了。

# 惰性评估

*惰性评估*是一个概念，特别是在处理大量数据时，您不希望在使用结果之前处理所有数据。您已经在`range`类型中看到了这一点，在其中，即使表示大量数字的一个`range`对象的内存占用量也是相同的。

## 生成器

您可以像使用`range`对象一样使用生成器。它们按需对数据执行一些操作，并在调用之间暂停其状态。这意味着您可以存储需要计算输出的变量，并且每次调用生成器时都会访问它们。

要编写生成器函数，使用`yield`关键字而不是返回语句。每次调用生成器时，它都返回`yield`指定的值，然后暂停其状态，直到下次调用。让我们编写一个简单地计数的生成器：

```py
>>> def count():
...     n = 0
...     while True:
...         n += 1
...         yield n
...
...
>>> counter = count()
>>> counter
<generator object count at 0x10e8509a8>
>>> next(counter)
1
>>> next(counter)
2
>>> next(counter)
3
```

请注意，生成器会跟踪其状态，因此每次调用生成器时，变量`n`都反映了先前设置的值。让我们实现一个 Fibonacci 生成器：

```py
>>> def fib():
...     first = 0
...     last = 1
...     while True:
...         first, last = last, first + last
...         yield first
...
>>> f = fib()
>>> next(f)
1
>>> next(f)
1
>>> next(f)
2
>>> next(f)
3
```

我们也可以在`for`循环中使用生成器进行迭代：

```py
>>> f = fib()
>>> for x in f:
...     print(x)
...     if x > 12:
...         break
...
1
1
2
3
5
8
13
```

## 生成器推导式

我们可以使用生成器推导式创建单行生成器。它们使用类似于列表推导式的语法，但使用圆括号而不是方括号：

```py
>>> list_o_nums = [x for x in range(100)]
>>> gen_o_nums = (x for x in range(100))
>>> list_o_nums
[0, 1, 2, 3, ...  97, 98, 99]
>>> gen_o_nums
<generator object <genexpr> at 0x10ea14408>
```

即使是这个小例子，我们也可以看到使用`sys.getsizeof`方法来查看对象的内存使用情况的差异，该方法以字节为单位返回对象的大小：

```py
>>> import sys
>>> sys.getsizeof(list_o_nums)
912
>>> sys.getsizeof(gen_o_nums)
120
```

# 更多 IPython 特性

您在本章开头看到了一些 IPython 的特性。现在让我们看一些更高级的功能，例如从 IPython 解释器内部运行 shell 命令和使用魔术函数。

## 使用 IPython 运行 Unix Shell 命令

你可以使用 IPython 运行 shell 命令。这是在 IPython shell 中执行 DevOps 操作的最有说服力的原因之一。让我们看一个非常简单的例子，其中`!`字符用于标识 IPython 中的 shell 命令，并放在`ls`命令的前面：

```py
In [3]: var_ls = !ls -l
In [4]: type(var_ls)
Out[4]: IPython.utils.text.SList
```

命令的输出被分配给一个 Python 变量`var_ls`。此变量的`type`是`IPython.utils.text.SList`。`SList`类型将常规 shell 命令转换为具有三个主要方法的对象：`fields`、`grep`和`sort`。以下是使用 Unix 的`df`命令进行排序的示例，`sort`方法可以解释此 Unix 命令中的空格，并按大小对第三列进行排序：

```py
In [6]: df = !df
In [7]: df.sort(3, nums = True)
```

接下来让我们来看看`SList`和`.grep`。以下是一个示例，搜索安装在*/usr/bin*目录中名称包含`kill`的命令：

```py
In [10]: ls = !ls -l /usr/bin
In [11]: ls.grep("kill")
Out[11]:
['-rwxr-xr-x   1 root   wheel      1621 Aug 20  2018 kill.d',
 '-rwxr-xr-x   1 root   wheel     23984 Mar 20 23:10 killall',
 '-rwxr-xr-x   1 root   wheel     30512 Mar 20 23:10 pkill']
```

这里的关键是，IPython 是一个非常适合用来玩弄小型 shell 脚本的理想环境。

### 使用 IPython 的魔术命令

如果你习惯使用 IPython，也应该养成使用内置魔术命令的习惯。它们本质上是一些强大的快捷方式。魔术命令通过在其前面加上`%%`来表示。这里有一个在 IPython 中写内联 Bash 的例子。注意，这只是一个小命令，但它可以是一个完整的 Bash 脚本：

```py
In [13]: %%bash
    ...: uname -a
    ...:
    ...:
Darwin nogibjj.local 18.5.0 Darwin Kernel Version 18.5.0: Mon Mar ...
```

`%%writefile` 很棘手，因为你可以即兴编写和测试 Python 或 Bash 脚本，使用 IPython 执行它们。这绝对不是一个坏点子：

```py
In [16]: %%writefile print_time.py
    ...: #!/usr/bin/env python
    ...: import datetime
    ...: print(datetime.datetime.now().time())
    ...:
    ...:
    ...:
Writing print_time.py

In [17]: cat print_time.py
#!/usr/bin/env python
import datetime
print(datetime.datetime.now().time())

In [18]: !python print_time.py
19:06:00.594914
```

另一个非常有用的命令 `%who`，将显示加载到内存中的内容。当你在一个长时间运行的终端中工作时，它非常方便：

```py
In [20]: %who
df     ls     var_ls
```

# 练习

+   编写一个 Python 函数，接受一个名称作为参数并打印该名称。

+   编写一个 Python 函数，接受一个字符串作为参数并打印它是大写还是小写。

+   编写一个列表推导，将单词 *smogtether* 中的每个字母都大写。

+   编写一个生成器，交替返回 *Even* 和 *Odd*。
