# 第九章：函数

> 函数越小，管理越大。
> 
> C. 北科特·帕金森

到目前为止，我们所有的 Python 代码示例都是小片段。这些对于小任务很好，但没人想一直重复输入片段。我们需要一种方法将更大的代码组织成可管理的片段。

代码重用的第一步是*函数*：一个命名的代码片段，独立于所有其他代码。函数可以接受任意数量和类型的输入*参数*，并返回任意数量和类型的输出*结果*。

你可以用一个函数做两件事情：

+   *定义*它，带有零个或多个参数

+   *调用*它，然后得到零个或多个结果

# 用 def 定义一个函数

要定义一个 Python 函数，你需要输入`def`，函数名，括号包围任何输入*参数*到函数中，然后最后是一个冒号(`:`)。函数名的规则与变量名相同（必须以字母或`_`开头，只能包含字母、数字或`_`）。

让我们一步一步来，首先定义并调用一个没有参数的函数。这是最简单的 Python 函数：

```py
>>> def do_nothing():
...     pass
```

即使对于像这样没有参数的函数，你仍然需要在其定义中使用括号和冒号。下一行需要缩进，就像你在`if`语句下缩进代码一样。Python 需要`pass`语句来表明这个函数什么也不做。这相当于*本页有意留白*（尽管它不再是空白的）。

# 用括号调用一个函数

你只需输入函数名和括号就可以调用这个函数。它像广告一样运行，什么也不做，但做得非常好：

```py
>>> do_nothing()
>>>
```

现在让我们定义并调用另一个没有参数但打印单词的函数：

```py
>>> def make_a_sound():
...     print('quack')
...
>>> make_a_sound()
quack
```

当你调用`make_a_sound()`函数时，Python 会执行其定义内的代码。在这种情况下，它打印了一个单词并返回到主程序。

让我们试试一个没有参数但*返回*值的函数：

```py
>>> def agree():
...    return True
...
```

你可以调用这个函数并使用`if`测试其返回值：

```py
>>> if agree():
...     print('Splendid!')
... else:
...     print('That was unexpected.')
...
Splendid!
```

你刚刚迈出了一个大步。函数与诸如`if`这样的测试以及诸如`while`这样的循环的结合，使你能够做一些以前无法做到的事情。

# 参数和参数

此时，是时候在括号里放些东西了。让我们定义一个名为`echo()`的函数，其中有一个名为`anything`的参数。它使用`return`语句两次将`anything`的值发送回调用者，之间用空格隔开：

```py
>>> def echo(anything):
...    return anything + ' ' + anything
...
>>>
```

现在让我们用字符串`'Rumplestiltskin'`调用`echo()`：

```py
>>> echo('Rumplestiltskin')
'Rumplestiltskin Rumplestiltskin'
```

当你调用带有参数的函数时，你传递的这些值称为*参数*。当你带有参数调用函数时，这些参数的值被复制到函数内部的对应*参数*中。

###### 注意

换句话说：它们在函数外被称为*参数*，但在函数内部称为*参数*。

在上一个示例中，函数`echo()`被调用时带有参数字符串`'Rumplestiltskin'`。此值在`echo()`内部复制到参数`anything`中，然后（在本例中加倍，并带有一个空格）返回给调用者。

这些函数示例非常基本。让我们编写一个函数，它接受一个输入参数并实际处理它。我们将调整早期评论颜色的代码片段。称之为`commentary`，并使其接受名为`color`的输入字符串参数。使其返回字符串描述给它的调用者，调用者可以决定如何处理它：

```py
>>> def commentary(color):
...     if color == 'red':
...         return "It's a tomato."
...     elif color == "green":
...         return "It's a green pepper."
...     elif color == 'bee purple':
...         return "I don't know what it is, but only bees can see it."
...     else:
...         return "I've never heard of the color "  + color +  "."
...
>>>
```

使用字符串参数`'blue'`调用函数`commentary()`。

```py
>>> comment = commentary('blue')
```

函数执行以下操作：

+   将值`'blue'`分配给函数内部的`color`参数

+   通过`if`-`elif`-`else`逻辑链运行

+   返回一个字符串

然后调用者将字符串赋给变量`comment`。

我们得到了什么？

```py
>>> print(comment)
I've never heard of the color blue.
```

函数可以接受任意数量（包括零个）任意类型的输入参数。它可以返回任意数量（包括零个）任意类型的输出结果。如果函数没有显式调用`return`，调用者将得到`None`的结果。

```py
>>> print(do_nothing())
None
```

## None 很有用

`None`是 Python 中的特殊值，表示当没有内容时占据的位置。它与布尔值`False`不同，尽管在布尔值评估时看起来是假的。以下是一个例子：

```py
>>> thing = None
>>> if thing:
...     print("It's some thing")
... else:
...     print("It's no thing")
...
It's no thing
```

要区分`None`和布尔值`False`，请使用 Python 的`is`运算符：

```py
>>> thing = None
>>> if thing is None:
...     print("It's nothing")
... else:
...     print("It's something")
...
It's nothing
```

这似乎是一个微妙的区别，但在 Python 中很重要。您将需要`None`来区分缺失值和空值。请记住，零值整数或浮点数，空字符串（`''`），列表（`[]`），元组（`(,)`），字典（`{}`）和集合（`set()`）都为`False`，但与`None`不同。

让我们编写一个快速函数，打印其参数是`None`，`True`还是`False`：

```py
>>> def whatis(thing):
...     if thing is None:
...         print(thing, "is None")
...     elif thing:
...         print(thing, "is True")
...     else:
...         print(thing, "is False")
...
```

让我们运行一些健全性测试：

```py
>>> whatis(None)
None is None
>>> whatis(True)
True is True
>>> whatis(False)
False is False
```

一些真实的值如何？

```py
>>> whatis(0)
0 is False
>>> whatis(0.0)
0.0 is False
>>> whatis('')
 is False
>>> whatis("")
 is False
>>> whatis('''''')
 is False
>>> whatis(())
() is False
>>> whatis([])
[] is False
>>> whatis({})
{} is False
>>> whatis(set())
set() is False
```

```py
>>> whatis(0.00001)
1e-05 is True
>>> whatis([0])
[0] is True
>>> whatis([''])
[''] is True
>>> whatis(' ')
 is True
```

## 位置参数

与许多语言相比，Python 处理函数参数的方式非常灵活。最熟悉的类型是*位置参数*，其值按顺序复制到相应的参数中。

此函数从其位置输入参数构建字典并返回它：

```py
>>> def menu(wine, entree, dessert):
...     return {'wine': wine, 'entree': entree, 'dessert': dessert}
...
>>> menu('chardonnay', 'chicken', 'cake')
{'wine': 'chardonnay', 'entree': 'chicken', 'dessert': 'cake'}
```

尽管非常普遍，位置参数的缺点是您需要记住每个位置的含义。如果我们忘记并将`menu()`以葡萄酒作为最后一个参数而不是第一个参数进行调用，那么餐点会非常不同：

```py
>>> menu('beef', 'bagel', 'bordeaux')
{'wine': 'beef', 'entree': 'bagel', 'dessert': 'bordeaux'}
```

## 关键字参数

为了避免位置参数混淆，您可以按照与函数定义中不同顺序的参数名称指定参数：

```py
>>> menu(entree='beef', dessert='bagel', wine='bordeaux')
{'wine': 'bordeaux', 'entree': 'beef', 'dessert': 'bagel'}
```

您可以混合位置和关键字参数。让我们首先指定葡萄酒，但使用关键字参数为主菜和甜点：

```py
>>> menu('frontenac', dessert='flan', entree='fish')
{'wine': 'frontenac', 'entree': 'fish', 'dessert': 'flan'}
```

如果您同时使用位置和关键字参数调用函数，则位置参数需要首先出现。

## 指定默认参数值

您可以为参数指定默认值。如果调用者未提供相应的参数，则使用默认值。这个听起来平淡无奇的特性实际上非常有用。使用前面的例子：

```py
>>> def menu(wine, entree, dessert='pudding'):
...     return {'wine': wine, 'entree': entree, 'dessert': dessert}
```

这次，尝试调用`menu()`而不带`dessert`参数：

```py
>>> menu('chardonnay', 'chicken')
{'wine': 'chardonnay', 'entree': 'chicken', 'dessert': 'pudding'}
```

如果提供了参数，则使用该参数而不是默认值：

```py
>>> menu('dunkelfelder', 'duck', 'doughnut')
{'wine': 'dunkelfelder', 'entree': 'duck', 'dessert': 'doughnut'}
```

###### 注意

默认参数值在函数*定义*时计算，而不是在运行时。对于新手（有时甚至不太新的）Python 程序员来说，常见的错误是使用可变数据类型（如列表或字典）作为默认参数。

在以下测试中，`buggy()`函数预计每次都将使用一个新的空`result`列表运行，并将`arg`参数添加到其中，然后打印一个单项列表。然而，这里有一个错误：它只有在第一次调用时为空。第二次调用时，`result`仍然保留了上一次调用的一个项目：

```py
>>> def buggy(arg, result=[]):
...     result.append(arg)
...     print(result)
...
>>> buggy('a')
['a']
>>> buggy('b')   # expect ['b']
['a', 'b']
```

如果写成这样，它会起作用：

```py
>>> def works(arg):
...     result = []
...     result.append(arg)
...     return result
...
>>> works('a')
['a']
>>> works('b')
['b']
```

修复方法是传递其他内容以指示第一次调用：

```py
>>> def nonbuggy(arg, result=None):
...     if result is None:
...         result = []
...     result.append(arg)
...     print(result)
...
>>> nonbuggy('a')
['a']
>>> nonbuggy('b')
['b']
```

这有时是 Python 工作面试的问题。你已经被警告了。

## 使用*爆炸/聚合位置参数

如果你在 C 或 C++中编程过，你可能会认为 Python 程序中的星号（`*`）与*指针*有关。不，Python 没有指针。

当在函数中与参数一起使用时，星号将可变数量的位置参数组合成单个参数值元组。在下面的例子中，`args`是由传递给函数`print_args()`的零个或多个参数组成的参数元组：

```py
>>> def print_args(*args):
...     print('Positional tuple:', args)
...
```

如果没有参数调用函数，则在`*args`中什么都得不到：

```py
>>> print_args()
Positional tuple: ()
```

无论您给它什么，它都将被打印为`args`元组：

```py
>>> print_args(3, 2, 1, 'wait!', 'uh...')
Positional tuple: (3, 2, 1, 'wait!', 'uh...')
```

这对编写像`print()`这样接受可变数量参数的函数非常有用。如果您的函数还有*必需*的位置参数，将它们放在第一位；`*args`放在最后并获取其余所有参数：

```py
>>> def print_more(required1, required2, *args):
...     print('Need this one:', required1)
...     print('Need this one too:', required2)
...     print('All the rest:', args)
...
>>> print_more('cap', 'gloves', 'scarf', 'monocle', 'mustache wax')
Need this one: cap
Need this one too: gloves
All the rest: ('scarf', 'monocle', 'mustache wax')
```

###### 注意

当使用`*`时，您不需要将元组参数称为`*args`，但在 Python 中这是一种常见的习惯用法。在函数内部使用`*args`也很常见，就像前面的例子中描述的那样，尽管严格来说它被称为参数，可以称为`*params`。

总结：

+   您可以将位置参数传递给函数，它们将在内部与位置参数匹配。这是你在本书中到目前为止看到的内容。

+   您可以将元组参数传递给函数，其中它将成为元组参数的一部分。这是前面一个示例的简单情况。

+   您可以将位置参数传递给函数，并将它们聚合在参数`*args`中，该参数解析为元组`args`。这在本节中已经描述过。

+   您还可以将名为`args`的元组参数“爆炸”为函数内部的位置参数`*args`，然后将其重新聚合到元组参数`args`中：

```py
>>> print_args(2, 5, 7, 'x')
Positional tuple: (2, 5, 7, 'x')
>>> args = (2,5,7,'x')
>>> print_args(args)
Positional tuple: ((2, 5, 7, 'x'),)
>>> print_args(*args)
Positional tuple: (2, 5, 7, 'x')
```

您只能在函数调用或定义中使用`*`语法：

```py
>>> *args
  File "<stdin>", line 1
SyntaxError: can't use starred expression here
```

所以：

+   在函数外部，`*args` 将元组 `args` 展开为逗号分隔的位置参数。

+   在函数内部，`*args` 将所有位置参数收集到一个名为 `args` 的元组中。你可以使用名称 `*params` 和 `params`，但通常在外部参数和内部参数中都使用 `*args` 是常见的做法。

有音感共鸣的读者也可能会在外部听到 `*args` 声称为 *puff-args*，在内部听到 *inhale-args*，因为值要么被展开要么被汇集。

## 用 ** 扩展/汇集关键字参数

你可以使用两个星号 (`**`) 将关键字参数组合成一个字典，其中参数名是键，它们的值是对应的字典值。下面的示例定义了函数 `print_kwargs()` 来打印它的关键字参数：

```py
>>> def print_kwargs(**kwargs):
...     print('Keyword arguments:', kwargs)
...
```

现在试着使用一些关键字参数调用它：

```py
>>> print_kwargs()
Keyword arguments: {}
>>> print_kwargs(wine='merlot', entree='mutton', dessert='macaroon')
Keyword arguments: {'dessert': 'macaroon', 'wine': 'merlot',
'entree': 'mutton'}
```

在函数内部，`kwargs` 是一个字典参数。

参数顺序是：

+   必需的位置参数

+   可选的位置参数 (`*args`)

+   可选的关键字参数 (`**kwargs`)

与 `args` 一样，你不需要将这个关键字参数称为 `kwargs`，但这是常见用法：¹

`**` 语法仅在函数调用或定义中有效：²

```py
>>> **kwparams
  File "<stdin>", line 1
    **kwparams
     ^
SyntaxError: invalid syntax
```

总结：

+   你可以向函数传递关键字参数，函数内部将它们与关键字参数匹配。这就是你迄今为止看到的内容。

+   你可以将字典参数传递给一个函数，函数内部会解析这些字典参数。这是前面讨论的一个简单情况。

+   你可以向函数传递一个或多个关键字参数 (*name=value*)，并在函数内部作为 `**kwargs` 收集它们，这已经在本节中讨论过了。

+   在函数外部，`**kwargs` *展开* 字典 `kwargs` 为 *name=value* 参数。

+   在函数内部，`**kwargs` *收集* `name=value` 参数到单个字典参数 `kwargs` 中。

如果听觉幻觉有所帮助，想象每个星号都在函数外面爆炸一下，而每个星号聚集在里面时会有一点吸气的声音。

## 仅关键字参数

可以传入一个与位置参数同名的关键字参数，这可能不会得到你想要的结果。Python 3 允许你指定 *仅关键字参数*。顾名思义，它们必须作为 *name=value* 而不是作为位置参数 *value* 提供。函数定义中的单个 `*` 意味着接下来的参数 `start` 和 `end` 如果我们不想使用它们的默认值，必须作为命名参数提供：

```py
>>> def print_data(data, *, start=0, end=100):
...     for value in (data[start:end]):
...         print(value)
...
>>> data = ['a', 'b', 'c', 'd', 'e', 'f']
>>> print_data(data)
a
b
c
d
e
f
>>> print_data(data, start=4)
e
f
>>> print_data(data, end=2)
a
b
```

## 可变与不可变参数

记住，如果你将同一个列表分配给两个变量，你可以通过任何一个变量来修改它？而如果这两个变量都引用像整数或字符串之类的不可变对象时就不行了？那是因为列表是可变的，而整数和字符串是不可变的。

在将参数传递给函数时，需要注意相同的行为。如果一个参数是可变的，它的值可以通过相应的参数*在函数内部*被改变：³

```py
>>> outside = ['one', 'fine', 'day']
>>> def mangle(arg):
...    arg[1] = 'terrible!'
...
>>> outside
['one', 'fine', 'day']
>>> mangle(outside)
>>> outside
['one', 'terrible!', 'day']
```

最好的做法是，呃，不要这样做。⁴ 要么记录参数可能被更改，要么通过`return`返回新值。

# 文档字符串

*可读性很重要*，Python 禅宗确实如此。你可以通过在函数体的开头包含一个字符串来附加文档到函数定义中。这就是函数的*文档字符串*：

```py
>>> def echo(anything):
...     'echo returns its input argument'
...     return anything
```

你可以使文档字符串相当长，并且如果你愿意，甚至可以添加丰富的格式：

```py
def print_if_true(thing, check):
 '''
 Prints the first argument if a second argument is true.
 The operation is:
 1\. Check whether the *second* argument is true.
 2\. If it is, print the *first* argument.
 '''
 if check:
 print(thing)
```

要打印函数的文档字符串，请调用 Python 的`help()`函数。传递函数的名称以获取带有精美格式的参数列表和文档字符串：

```py
>>> help(echo)
Help on function echo in module __main__:

echo(anything)
 echo returns its input argument
```

如果你只想看到原始的文档字符串，而没有格式：

```py
>>> print(echo.__doc__)
echo returns its input argument
```

那个看起来奇怪的`__doc__`是函数内部文档字符串作为变量的内部名称。双下划线（Python 术语中称为*dunder*）在许多地方用于命名 Python 内部变量，因为程序员不太可能在自己的变量名中使用它们。

# 函数是一等公民

我提到了 Python 的口头禅，*一切皆为对象*。这包括数字、字符串、元组、列表、字典——还有函数。在 Python 中，函数是一等公民。你可以将它们赋值给变量，将它们用作其他函数的参数，并从函数中返回它们。这使得你能够在 Python 中做一些在许多其他语言中难以或不可能实现的事情。

要测试这个，让我们定义一个简单的函数叫做`answer()`，它没有任何参数；它只是打印数字`42`：

```py
>>> def answer():
...     print(42)
```

如果你运行这个函数，你知道会得到什么：

```py
>>> answer()
42
```

现在让我们定义另一个名为`run_something`的函数。它有一个名为`func`的参数，一个要运行的函数。进入函数后，它只是调用这个函数：

```py
>>> def run_something(func):
...     func()
```

如果我们将`answer`传递给`run_something()`，我们正在使用函数作为数据，就像其他任何东西一样：

```py
>>> run_something(answer)
42
```

注意，你传递的是`answer`，而不是`answer()`。在 Python 中，那些括号意味着*调用这个函数*。没有括号，Python 只是将函数视为任何其他对象一样对待。这是因为，像 Python 中的一切其他东西一样，它*是*一个对象：

```py
>>> type(run_something)
<class 'function'>
```

让我们尝试运行一个带有参数的函数。定义一个名为`add_args()`的函数，打印其两个数值参数`arg1`和`arg2`的和：

```py
>>> def add_args(arg1, arg2):
...     print(arg1 + arg2)
```

`add_args()`是什么？

```py
>>> type(add_args)
<class 'function'>
```

此时，让我们定义一个名为`run_something_with_args()`的函数，它接受三个参数：

`func`

要运行的函数

`arg1`

`func`的第一个参数

`arg2`

`func`的第二个参数

```py
>>> def run_something_with_args(func, arg1, arg2):
...     func(arg1, arg2)
```

当您调用`run_something_with_args()`时，调用者传递的函数被赋给`func`参数，而`arg1`和`arg2`得到了在参数列表中跟随的值。然后，运行`func(arg1, arg2)`使用这些参数执行该函数，因为括号告诉 Python 这样做。

让我们通过向函数名`add_args`和参数`5`及`9`传递给`run_something_with_args()`来测试它：

```py
>>> run_something_with_args(add_args, 5, 9)
14
```

在函数`run_something_with_args()`中，函数名参数`add_args`被赋给了参数`func`，`5`被赋给了参数`arg1`，`9`被赋给了参数`arg2`。这最终执行了：

```py
add_args(5, 9)
```

您可以将此与`*args`和`**kwargs`技术结合使用。

让我们定义一个测试函数，它接受任意数量的位置参数，通过使用`sum()`函数计算它们的总和，然后返回该总和：

```py
>>> def sum_args(*args):
...    return sum(args)
```

我之前没有提到`sum()`。它是一个内置的 Python 函数，用于计算其可迭代数值（int 或 float）参数中值的总和。

让我们定义新函数`run_with_positional_args()`，它接受一个函数和任意数量的位置参数以传递给它：

```py
>>> def run_with_positional_args(func, *args):
...    return func(*args)
```

现在继续调用它：

```py
>>> run_with_positional_args(sum_args, 1, 2, 3, 4)
10
```

您可以将函数用作列表、元组、集合和字典的元素。函数是不可变的，因此您也可以将它们用作字典键。

# 内部函数

你可以在一个函数内定义另一个函数：

```py
>>> def outer(a, b):
...     def inner(c, d):
...         return c + d
...     return inner(a, b)
...
>>>
>>> outer(4, 7)
11
```

当在另一个函数内部执行某个复杂任务超过一次时，内部函数可以很有用，以避免循环或代码重复。例如，对于字符串示例，此内部函数向其参数添加一些文本：

```py
>>> def knights(saying):
...     def inner(quote):
...         return "We are the knights who say: '%s'" % quote
...     return inner(saying)
...
>>> knights('Ni!')
"We are the knights who say: 'Ni!'"
```

## 闭包

内部函数可以充当*闭包*。这是由另一个函数动态生成的函数，可以改变并记住在函数外部创建的变量的值。

以下示例是基于先前的`knights()`示例构建的。让我们称其为`knights2()`，因为我们没有想象力，并将`inner()`函数转变为称为`inner2()`的闭包。以下是它们之间的区别：

+   `inner2()`直接使用外部的`saying`参数，而不是作为参数获取它。

+   `knights2()`返回了`inner2`函数名而不是调用它：

    ```py
    >>> def knights2(saying):
    ...     def inner2():
    ...         return "We are the knights who say: '%s'" % saying
    ...     return inner2
    ...
    ```

`inner2()`函数知道传入的`saying`值，并记住它。`return inner2`这一行返回了这个专门的`inner2`函数副本（但没有调用它）。这是一种闭包：一个动态创建的函数，记住了它来自何处。

让我们两次调用`knights2()`，使用不同的参数：

```py
>>> a = knights2('Duck')
>>> b = knights2('Hasenpfeffer')
```

好的，那么`a`和`b`是什么？

```py
>>> type(a)
<class 'function'>
>>> type(b)
<class 'function'>
```

它们是函数，但它们也是闭包：

```py
>>> a
<function knights2.<locals>.inner2 at 0x10193e158>
>>> b
<function knights2.<locals>.inner2 at 0x10193e1e0>
```

如果我们调用它们，它们会记住由`knights2`创建时使用的`saying`：

```py
>>> a()
"We are the knights who say: 'Duck'"
>>> b()
"We are the knights who say: 'Hasenpfeffer'"
```

# 匿名函数：lambda

Python *lambda 函数* 是作为单个语句表示的匿名函数。您可以使用它来代替普通的小函数。

为了说明这一点，让我们首先做一个使用普通函数的示例。首先，让我们定义函数 `edit_story()`。它的参数如下：

+   `words`—一个单词列表

+   `func`—应用于 `words` 中每个单词的函数

```py
>>> def edit_story(words, func):
...     for word in words:
...         print(func(word))
```

现在我们需要一个单词列表和一个应用到每个单词的函数。对于单词，这里是我家猫（假设的情况下）如果（假设的情况下）错过了其中一个楼梯可能发出的一系列（假设的）声音：

```py
>>> stairs = ['thud', 'meow', 'thud', 'hiss']
```

而对于函数，这将使每个单词大写并追加一个感叹号，非常适合猫类小报的头条新闻：

```py
>>> def enliven(word):   # give that prose more punch
...     return word.capitalize() + '!'
```

混合我们的成分：

```py
>>> edit_story(stairs, enliven)
Thud!
Meow!
Thud!
Hiss!
```

最后，我们来到 lambda。`enliven()` 函数如此简短，以至于我们可以用 lambda 替换它：

```py
>>> edit_story(stairs, lambda word: word.capitalize() + '!')
Thud!
Meow!
Thud!
Hiss!
```

Lambda 函数有零个或多个逗号分隔的参数，后跟一个冒号（`:`），然后是函数的定义。我们给这个 lambda 一个参数 `word`。你不像调用 `def` 创建的函数那样在 lambda 函数中使用括号。

通常，使用真正的函数如 `enliven()` 要比使用 lambda 函数更清晰。Lambda 函数主要用于在否则需要定义许多小函数并记住它们名称的情况下非常有用。特别是在图形用户界面中，你可以用 lambda 来定义 *回调函数*；详见第二十章的示例。

# 生成器

一个 *生成器* 是 Python 的序列创建对象。利用它，你可以在不一次性创建和存储整个序列于内存中的情况下遍历可能非常庞大的序列。生成器通常是迭代器的数据源。如果你还记得，在之前的代码示例中我们已经使用了其中一个，`range()`，来生成一系列整数。在 Python 2 中，`range()` 返回一个列表，这限制了它的内存使用。Python 2 还有生成器 `xrange()`，在 Python 3 中变成了普通的 `range()`。这个示例将所有的整数从 1 加到 100：

```py
>>> sum(range(1, 101))
5050
```

每次你遍历一个生成器时，它都会记住上次被调用时的位置，并返回下一个值。这与普通函数不同，普通函数没有记忆先前调用的状态，每次都从第一行开始执行。

## 生成器函数

如果你想创建一个可能很大的序列，可以编写一个 *生成器函数*。它是一个普通函数，但它通过 `yield` 语句而不是 `return` 返回它的值。让我们来写我们自己的 `range()` 版本：

```py
>>> def my_range(first=0, last=10, step=1):
...     number = first
...     while number < last:
...         yield number
...         number += step
...
```

这是一个普通函数：

```py
>>> my_range
<function my_range at 0x10193e268>
```

并返回一个生成器对象：

```py
>>> ranger = my_range(1, 5)
>>> ranger
<generator object my_range at 0x101a0a168>
```

我们可以遍历这个生成器对象：

```py
>>> for x in ranger:
...     print(x)
...
1
2
3
4
```

###### 注意

生成器只能运行一次。列表、集合、字符串和字典存在于内存中，但生成器会即时创建其值，并逐个通过迭代器分发它们。它不记住它们，因此你无法重新启动或备份生成器。

如果你尝试再次迭代这个生成器，你会发现它已经耗尽了：

```py
>>> for try_again in ranger:
...     print(try_again)
...
>>>
```

## 生成器推导式

您已经看到了用于列表、字典和集合的理解。*生成器理解* 看起来像那些，但是用圆括号而不是方括号或花括号括起来。这就像是生成器函数的简写版本，隐式地执行 `yield` 并返回生成器对象：

```py
>>> genobj = (pair for pair in zip(['a', 'b'], ['1', '2']))
>>> genobj
<generator object <genexpr> at 0x10308fde0>
>>> for thing in genobj:
...     print(thing)
...
('a', '1')
('b', '2')
```

# 装饰器

有时，您希望修改现有函数而不更改其源代码。一个常见的例子是添加调试语句以查看传递的参数是什么。

*装饰器* 是一个接受一个函数作为输入并返回另一个函数的函数。让我们深入探讨一下我们的 Python 技巧，并使用以下内容：

+   `*args` 和 `**kwargs`

+   内部函数

+   函数作为参数

函数 `document_it()` 定义了一个装饰器，该装饰器将执行以下操作：

+   打印函数的名称和其参数的值

+   使用参数运行函数

+   打印结果

+   返回修改后的函数以供使用

以下是代码的样子：

```py
>>> def document_it(func):
...     def new_function(*args, **kwargs):
...         print('Running function:', func.__name__)
...         print('Positional arguments:', args)
...         print('Keyword arguments:', kwargs)
...         result = func(*args, **kwargs)
...         print('Result:', result)
...         return result
...     return new_function
```

无论您传递给 `document_it()` 的 `func` 是什么，您都会得到一个包含 `document_it()` 添加的额外语句的新函数。装饰器实际上不必从 `func` 运行任何代码，但 `document_it()` 在执行过程中调用 `func`，以便您既可以获得 `func` 的结果，又可以获得所有额外的内容。

那么，您如何使用它呢？您可以手动应用装饰器：

```py
>>> def add_ints(a, b):
...    return a + b
...
>>> add_ints(3, 5)
8
>>> cooler_add_ints = document_it(add_ints)  # manual decorator assignment
>>> cooler_add_ints(3, 5)
Running function: add_ints
Positional arguments: (3, 5)
Keyword arguments: {}
Result: 8
8
```

作为手动装饰器分配的替代方案，您可以在您想要装饰的函数之前添加 *@*`decorator_name`* ：

```py
>>> @document_it
... def add_ints(a, b):
...     return a + b
...
>>> add_ints(3, 5)
Start function add_ints
Positional arguments: (3, 5)
Keyword arguments: {}
Result: 8
8
```

一个函数可以有多个装饰器。让我们再写一个名为 `square_it()` 的装饰器，用于对结果进行平方：

```py
>>> def square_it(func):
...     def new_function(*args, **kwargs):
...         result = func(*args, **kwargs)
...         return result * result
...     return new_function
...
```

最接近函数的装饰器（就在 `def` 的上方）首先运行，然后是它上面的装饰器。无论顺序如何，最终结果都相同，但您可以看到中间步骤如何改变：

```py
>>> @document_it
... @square_it
... def add_ints(a, b):
...     return a + b
...
>>> add_ints(3, 5)
Running function: new_function
Positional arguments: (3, 5)
Keyword arguments: {}
Result: 64
64
```

让我们试试颠倒装饰器的顺序：

```py
>>> @square_it
... @document_it
... def add_ints(a, b):
...     return a + b
...
>>> add_ints(3, 5)
Running function: add_ints
Positional arguments: (3, 5)
Keyword arguments: {}
Result: 8
64
```

# 命名空间和作用域

> 渴望这个人的艺术和那个人的视野
> 
> 威廉·莎士比亚

一个名称可以根据其使用的位置引用不同的事物。Python 程序有各种*命名空间* —— 某个名称在其中是唯一的，并且与其他命名空间中相同名称无关。

每个函数定义其自己的命名空间。如果您在主程序中定义了一个名为 `x` 的变量，并在函数中定义了另一个名为 `x` 的变量，它们将指代不同的事物。但是，墙壁是可以打破的：如果需要，可以通过各种方式访问其他命名空间中的名称。

程序的主要部分定义了*全局*命名空间；因此，在该命名空间中的变量是*全局变量*。

您可以从函数内部获取全局变量的值：

```py
>>> animal = 'fruitbat'
>>> def print_global():
...     print('inside print_global:', animal)
...
>>> print('at the top level:', animal)
at the top level: fruitbat
>>> print_global()
inside print_global: fruitbat
```

但是，如果您尝试在函数内获取全局变量的值*并且*更改它，您将会得到一个错误：

```py
>>> def change_and_print_global():
...     print('inside change_and_print_global:', animal)
...     animal = 'wombat'
...     print('after the change:', animal)
...
>>> change_and_print_global()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in change_and_print_global
UnboundLocalError: local variable 'animal' referenced before assignment
```

如果您只是更改它，则还会更改另一个名为 `animal` 的变量，但此变量位于函数内部：

```py
>>> def change_local():
...     animal = 'wombat'
...     print('inside change_local:', animal, id(animal))
...
>>> change_local()
inside change_local: wombat 4330406160
>>> animal
'fruitbat'
>>> id(animal)
4330390832
```

发生了什么？第一行将字符串 `'fruitbat'` 分配给名为 `animal` 的全局变量。`change_local()` 函数中也有一个名为 `animal` 的变量，但它在其局部命名空间中。

我在这里使用了 Python 函数 `id()` 来打印每个对象的唯一值，并证明 `change_local()` 内部的变量 `animal` 不同于程序主层级的 `animal`。

在函数内部访问全局变量而不是局部变量，你需要显式使用 `global` 关键字（你知道这是必须的：*显式优于隐式*）：

```py
>>> animal = 'fruitbat'
>>> def change_and_print_global():
...     global animal
...     animal = 'wombat'
...     print('inside change_and_print_global:', animal)
...
>>> animal
'fruitbat'
>>> change_and_print_global()
inside change_and_print_global: wombat
>>> animal
'wombat'
```

如果在函数内部不使用 `global`，Python 使用局部命名空间，变量是局部的。函数执行完成后，它就消失了。

Python 提供了两个函数来访问你的命名空间的内容：

+   `locals()` 返回本地命名空间内容的字典。

+   `globals()` 返回全局命名空间内容的字典。

并且它们在使用中：

```py
>>> animal = 'fruitbat'
>>> def change_local():
...     animal = 'wombat'  # local variable
...     print('locals:', locals())
...
>>> animal
'fruitbat'
>>> change_local()
locals: {'animal': 'wombat'}
>>> print('globals:', globals()) # reformatted a little for presentation
globals: {'animal': 'fruitbat',
'__doc__': None,
'change_local': <function change_local at 0x1006c0170>,
'__package__': None,
'__name__': '__main__',
'__loader__': <class '_frozen_importlib.BuiltinImporter'>,
'__builtins__': <module 'builtins'>}
>>> animal
'fruitbat'
```

`change_local()` 内的局部命名空间仅包含局部变量 `animal`。全局命名空间包含独立的全局变量 `animal` 和许多其他内容。

# 在名称中使用 _ 和 __

以两个下划线 (`__`) 开头和结尾的名称保留供 Python 使用，因此你不应该将它们用于自己的变量。选择这种命名模式是因为看起来不太可能被应用开发人员选为其自己的变量名。

例如，函数的名称在系统变量 *`function`*`.__name__` 中，其文档字符串在 *`function`*`.__doc__` 中：

```py
>>> def amazing():
...     '''This is the amazing function.
...     Want to see it again?'''
...     print('This function is named:', amazing.__name__)
...     print('And its docstring is:', amazing.__doc__)
...
>>> amazing()
This function is named: amazing
And its docstring is: This is the amazing function.
 Want to see it again?
```

正如你在前面的 `globals` 打印中看到的那样，主程序被赋予特殊名称 `__main__`。

# 递归

到目前为止，我们已经调用了一些直接执行某些操作的函数，并可能调用其他函数。但如果一个函数调用自身呢？⁵ 这就是*递归*。就像使用 `while` 或 `for` 的无限循环一样，你不希望出现无限递归。我们仍然需要担心时空连续性的裂缝吗？

Python 再次拯救了宇宙，如果你深入太多，它会引发异常：

```py
>>> def dive():
...     return dive()
...
>>> dive()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in dive
  File "<stdin>", line 2, in dive
  File "<stdin>", line 2, in dive
 [Previous line repeated 996 more times]
RecursionError: maximum recursion depth exceeded
```

当处理像列表的列表的列表这样的“不均匀”数据时，递归非常有用。假设你想要“展平”列表的所有子列表，⁶ 无论嵌套多深，生成器函数正是你需要的：

```py
>>> def flatten(lol):
...     for item in lol:
...         if isinstance(item, list):
...             for subitem in flatten(item):
...                 yield subitem
...         else:
...             yield item
...
>>> lol = [1, 2, [3,4,5], [6,[7,8,9], []]]
>>> flatten(lol)
<generator object flatten at 0x10509a750>
>>> list(flatten(lol))
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Python 3.3 添加了 `yield from` 表达式，允许生成器将一些工作交给另一个生成器。我们可以用它来简化 `flatten()`：

```py
>>> def flatten(lol):
...     for item in lol:
...         if isinstance(item, list):
...             yield from flatten(item)
...         else:
...             yield item
...
>>> lol = [1, 2, [3,4,5], [6,[7,8,9], []]]
>>> list(flatten(lol))
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

# 异步函数

关键字 `async` 和 `await` 被添加到 Python 3.5 中，用于定义和运行*异步函数*。它们是：

+   相对较新

+   足够不同，以至于更难理解

+   随着时间的推移，它会变得更重要且更为人熟知

出于这些原因，我已将这些及其他异步主题的讨论移到了附录 C 中的 Appendix C。

现在，你需要知道，如果在函数的`def`行之前看到`async`，那么这是一个异步函数。同样，如果在函数调用之前看到`await`，那么该函数是异步的。

异步函数和普通函数的主要区别在于异步函数可以“放弃控制”，而不是一直运行到完成。

# 异常

在某些语言中，错误通过特殊的函数返回值来指示。当事情出错时，Python 使用*异常*：当关联的错误发生时执行的代码。

你已经看到了一些例子，比如访问列表或元组时使用超出范围的位置，或者使用不存在的键访问字典。当你运行在某些情况下可能失败的代码时，还需要适当的*异常处理程序*来拦截任何潜在的错误。

在任何可能发生异常的地方添加异常处理是一个好习惯，以便让用户知道发生了什么。你可能无法修复问题，但至少可以记录情况并优雅地关闭程序。如果异常发生在某个函数中并且没有在那里捕获，它会*冒泡*直到在某个调用函数中找到匹配的处理程序。如果不提供自己的异常处理程序，Python 会打印错误消息和关于错误发生位置的一些信息，然后终止程序，如以下代码片段所示：

```py
>>> short_list = [1, 2, 3]
>>> position = 5
>>> short_list[position]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```

## 使用 try 和 except 处理错误

> 做，或者不做。没有尝试。
> 
> 尤达

不要留事情交给机会，使用`try`包装你的代码，并使用`except`提供错误处理：

```py
>>> short_list = [1, 2, 3]
>>> position = 5
>>> try:
...     short_list[position]
... except:
...     print('Need a position between 0 and', len(short_list)-1, ' but got',
...            position)
...
Need a position between 0 and 2 but got 5
```

执行`try`块内的代码。如果出现错误，将引发异常并执行`except`块内的代码。如果没有错误，则跳过`except`块。

在这里使用没有参数的纯`except`来指定是一个通用的异常捕获。如果可能会出现多种类型的异常，最好为每种类型提供单独的异常处理程序。没有人强迫你这样做；你可以使用裸露的`except`来捕获所有异常，但你对它们的处理可能是通用的（类似于打印*发生了一些错误*）。你可以使用任意数量的特定异常处理程序。

有时候，你希望除了类型以外的异常详情。如果使用以下形式，你会在变量*name*中得到完整的异常对象：

```py
except *`exceptiontype`* as *`name`*
```

以下示例首先查找`IndexError`，因为当你向序列提供非法位置时，会引发该异常类型。它将`IndexError`异常保存在变量`err`中，将其他任何异常保存在变量`other`中。示例打印`other`中存储的所有内容，以展示你在该对象中得到的内容：

```py
>>> short_list = [1, 2, 3]
>>> while True:
...     value = input('Position [q to quit]? ')
...     if value == 'q':
...         break
...     try:
...         position = int(value)
...         print(short_list[position])
...     except IndexError as err:
...         print('Bad index:', position)
...     except Exception as other:
...         print('Something else broke:', other)
...
Position [q to quit]? 1
2
Position [q to quit]? 0
1
Position [q to quit]? 2
3
Position [q to quit]? 3
Bad index: 3
Position [q to quit]? 2
3
Position [q to quit]? two
Something else broke: invalid literal for int() with base 10: 'two'
Position [q to quit]? q
```

输入位置`3`引发了预期的`IndexError`。输入`two`使`int()`函数感到恼火，我们在第二个通用的`except`代码中处理了它。

## 创建自定义异常

上一节讨论了处理异常，但所有的异常（如`IndexError`）都是在 Python 或其标准库中预定义的。你可以为自己的程序使用其中任何一个。你也可以定义自己的异常类型来处理可能在你自己的程序中出现的特殊情况。

###### 注意

这需要定义一个新的对象类型，使用一个*类*——这是我们直到第十章才会详细讨论的内容。所以，如果你对类不熟悉，可能需要稍后再回到本节。

异常是一个类。它是类`Exception`的子类。让我们创建一个叫做`UppercaseException`的异常，在字符串中遇到大写字母时引发它：

```py
>>> class UppercaseException(Exception):
...     pass
...
>>> words = ['eenie', 'meenie', 'miny', 'MO']
>>> for word in words:
...     if word.isupper():
...         raise UppercaseException(word)
...
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
__main__.UppercaseException: MO
```

我们甚至没有为`UppercaseException`定义任何行为（注意我们只是使用了`pass`），让它的父类`Exception`来决定在引发异常时打印什么。

你可以访问异常对象本身并打印它：

```py
>>> try:
...     raise OopsException('panic')
... except OopsException as exc:
...     print(exc)
...
panic
```

# 即将出现

对象！在一本关于面向对象语言的书中，我们必须介绍它们。

# 要做的事情

9.1 定义一个名为`good()`的函数，返回以下列表：`['Harry', 'Ron', 'Hermione']`。

9.2 定义一个名为`get_odds()`的生成器函数，返回`range(10)`中的奇数。使用`for`循环找到并打印第三个返回的值。

9.3 定义一个名为`test`的装饰器，在调用函数时打印`'start'`，在函数结束时打印`'end'`。

9.4 定义一个名为`OopsException`的异常。引发这个异常看看会发生什么。然后，编写代码捕捉这个异常并打印`'Caught an oops'`。

¹ 虽然*Args*和*Kwargs*听起来像海盗鹦鹉的名字。

² 或者，如 Python 3.5 中的字典合并形式`{**a, **b}`，就像你在第八章看到的那样。

³ 就像那些青少年陷入危险的电影中他们学会了“电话是从房子里打来的！”

⁴ 就像那个老医生笑话：“当我这样做时很痛。” “那么，就别这样做。”

⁵ 这就像说，“如果我每次希望有一美元，我就能有一美元。”

⁶ 又是一个 Python 面试问题。收集整套吧！

⁷ 这是北半球主义吗？澳大利亚人和新西兰人说东西乱了会说“north”吗？
