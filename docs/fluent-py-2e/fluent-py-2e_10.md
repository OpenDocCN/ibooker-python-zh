# 第八章：函数中的类型提示

> 还应强调**Python 将保持动态类型语言，并且作者从未希望通过约定使类型提示成为强制要求。**
> 
> Guido van Rossum，Jukka Lehtosalo 和Łukasz Langa，PEP 484—类型提示¹

类型提示是自 2001 年发布的 Python 2.2 中的[类型和类的统一](https://fpy.li/descr101)以来 Python 历史上最大的变化。然而，并非所有 Python 用户都同等受益于类型提示。这就是为什么它们应该始终是可选的。

[PEP 484—类型提示](https://fpy.li/pep484)引入了函数参数、返回值和变量的显式类型声明的语法和语义。其目标是通过静态分析帮助开发人员工具在不实际运行代码测试的情况下发现 Python 代码库中的错误。

主要受益者是使用 IDE（集成开发环境）和 CI（持续集成）的专业软件工程师。使类型提示对该群体具有吸引力的成本效益分析并不适用于所有 Python 用户。

Python 的用户群比这个宽广得多。它包括科学家、交易员、记者、艺术家、制造商、分析师和许多领域的学生等。对于他们中的大多数人来说，学习类型提示的成本可能更高——除非他们已经了解具有静态类型、子类型和泛型的语言。对于许多这些用户来说，由于他们与 Python 的交互方式以及他们的代码库和团队的规模较小——通常是“一个人的团队”，因此收益会较低。Python 的默认动态类型在编写用于探索数据和想法的代码时更简单、更具表现力，比如数据科学、创意计算和学习，

本章重点介绍 Python 函数签名中的类型提示。第十五章探讨了类的上下文中的类型提示，以及其他`typing`模块功能。

本章的主要主题包括：

+   一个关于使用 Mypy 逐渐类型化的实践介绍

+   鸭子类型和名义类型的互补视角

+   注解中可能出现的主要类型类别概述——这大约占了本章的 60%

+   类型提示可变参数（`*args`，`**kwargs`）

+   类型提示和静态类型化的限制和缺点

# 本章的新内容

本章是全新的。类型提示出现在我完成第一版*流畅的 Python*之后的 Python 3.5 中。

鉴于静态类型系统的局限性，PEP 484 的最佳想法是引入*逐渐类型系统*。让我们从定义这个概念开始。

# 关于逐渐类型化

PEP 484 向 Python 引入了*逐渐类型系统*。其他具有逐渐类型系统的语言包括微软的 TypeScript、Dart（由 Google 创建的 Flutter SDK 的语言）和 Hack（Facebook 的 HHVM 虚拟机支持的 PHP 方言）。Mypy 类型检查器本身起初是一种语言：一种逐渐类型化的 Python 方言，带有自己的解释器。Guido van Rossum 说服了 Mypy 的创造者 Jukka Lehtosalo，使其成为检查带注释的 Python 代码的工具。

逐渐类型系统：

是可选的

默认情况下，类型检查器不应对没有类型提示的代码发出警告。相反，当无法确定对象类型时，类型检查器会假定`Any`类型。`Any`类型被认为与所有其他类型兼容。

不会在运行时捕获类型错误

静态类型检查器、linter 和 IDE 使用类型提示来发出警告。它们不能阻止在运行时将不一致的值传递给函数或分配给变量。

不会增强性能

类型注释提供的数据理论上可以允许在生成的字节码中进行优化，但截至 2021 年 7 月，我所知道的任何 Python 运行时都没有实现这样的优化。²

逐步类型化最好的可用性特性是注释始终是可选的。

使用静态类型系统，大多数类型约束很容易表达，许多很繁琐，一些很困难，而一些则是不可能的。³ 你很可能会写出一段优秀的 Python 代码，具有良好的测试覆盖率和通过的测试，但仍然无法添加满足类型检查器的类型提示。没关系；只需省略有问题的类型提示并发布！

类型提示在所有级别都是可选的：你可以有完全没有类型提示的整个包，当你将其中一个这样的包导入到使用类型提示的模块时，你可以让类型检查器保持沉默，并且你可以添加特殊注释来让类型检查器忽略代码中特定的行。

###### 提示

寻求 100% 的类型提示覆盖可能会刺激没有经过适当思考的类型提示，只是为了满足指标。这也会阻止团队充分利用 Python 的强大和灵活性。当注释会使 API 不够用户友好，或者不必要地复杂化其实现时，应该自然地接受没有类型提示的代码。

# 实践中的逐步类型化

让我们看看逐步类型化在实践中是如何工作的，从一个简单的函数开始，逐渐添加类型提示，由 Mypy 指导。

###### 注意

有几个与 PEP 484 兼容的 Python 类型检查器，包括 Google 的 [pytype](https://fpy.li/8-4)、Microsoft 的 [Pyright](https://fpy.li/8-5)、Facebook 的 [Pyre](https://fpy.li/8-6)—以及嵌入在 IDE 中的类型检查器，如 PyCharm。我选择了 [Mypy](https://fpy.li/mypy) 作为示例，因为它是最知名的。然而，其他类型检查器可能更适合某些项目或团队。例如，Pytype 设计用于处理没有类型提示的代码库，并仍然提供有用的建议。它比 Mypy 更宽松，还可以为您的代码生成注释。

我们将为一个返回带有计数和单数或复数词的字符串的 `show_count` 函数添加注释：

```py
>>> show_count(99, 'bird')
'99 birds'
>>> show_count(1, 'bird')
'1 bird'
>>> show_count(0, 'bird')
'no birds'
```

示例 8-1 展示了`show_count`的源代码，没有注释。

##### 示例 8-1\. *messages.py* 中没有类型提示的 `show_count`

```py
def show_count(count, word):
    if count == 1:
        return f'1 {word}'
    count_str = str(count) if count else 'no'
    return f'{count_str} {word}s'
```

## 从 Mypy 开始

要开始类型检查，我在 *messages.py* 模块上运行 `mypy` 命令：

```py
…/no_hints/ $ pip install mypy
[lots of messages omitted...]
…/no_hints/ $ mypy messages.py
Success: no issues found in 1 source file
```

使用默认设置的 Mypy 在 示例 8-1 中没有发现任何问题。

###### 警告

我正在使用 Mypy 0.910，在我审阅这篇文章时是最新版本（2021 年 7 月）。Mypy 的 [“介绍”](https://fpy.li/8-7) 警告说它“正式是测试版软件。偶尔会有破坏向后兼容性的更改。” Mypy 给我至少一个与我在 2020 年 4 月写这一章时不同的报告。当你阅读这篇文章时，你可能会得到与这里显示的不同的结果。

如果函数签名没有注释，Mypy 默认会忽略它—除非另有配置。

对于 示例 8-2，我还有 `pytest` 单元测试。这是 *messages_test.py* 中的代码。

##### 示例 8-2\. *messages_test.py* 中没有类型提示

```py
from pytest import mark

from messages import show_count

@mark.parametrize('qty, expected', [
    (1, '1 part'),
    (2, '2 parts'),
])
def test_show_count(qty, expected):
    got = show_count(qty, 'part')
    assert got == expected

def test_show_count_zero():
    got = show_count(0, 'part')
    assert got == 'no parts'
```

现在让我们根据 Mypy 添加类型提示。

## 使 Mypy 更严格

命令行选项 `--disallow-untyped-defs` 会使 Mypy 标记任何没有为所有参数和返回值添加类型提示的函数定义。

在测试文件上使用 `--disallow-untyped-defs` 会产生三个错误和一个注意：

```py
…/no_hints/ $ mypy --disallow-untyped-defs messages_test.py
messages.py:14: error: Function is missing a type annotation
messages_test.py:10: error: Function is missing a type annotation
messages_test.py:15: error: Function is missing a return type annotation
messages_test.py:15: note: Use "-> None" if function does not return a value
Found 3 errors in 2 files (checked 1 source file)
```

对于逐步类型化的第一步，我更喜欢使用另一个选项：`--disallow-incomplete-defs`。最初，它对我毫无意义：

```py
…/no_hints/ $ mypy --disallow-incomplete-defs messages_test.py
Success: no issues found in 1 source file
```

现在我可以只为 *messages.py* 中的 `show_count` 添加返回类型：

```py
def show_count(count, word) -> str:
```

这已经足够让 Mypy 查看它。使用与之前相同的命令行检查 *messages_test.py* 将导致 Mypy 再次查看 *messages.py*：

```py
…/no_hints/ $ mypy --disallow-incomplete-defs messages_test.py
messages.py:14: error: Function is missing a type annotation
for one or more arguments
Found 1 error in 1 file (checked 1 source file)
```

现在我可以逐步为每个函数添加类型提示，而不会收到关于我没有注释的函数的警告。这是一个完全注释的签名，满足了 Mypy：

```py
def show_count(count: int, word: str) -> str:
```

###### 提示

与其像`--disallow-incomplete-defs`这样输入命令行选项，你可以按照[Mypy 配置文件](https://fpy.li/8-8)文档中描述的方式保存你喜欢的选项。你可以有全局设置和每个模块的设置。以下是一个简单的*mypy.ini*示例：

```py
[mypy]
python_version = 3.9
warn_unused_configs = True
disallow_incomplete_defs = True
```

## 默认参数值

示例 8-1 中的`show_count`函数只适用于常规名词。如果复数不能通过添加`'s'`来拼写，我们应该让用户提供复数形式，就像这样：

```py
>>> show_count(3, 'mouse', 'mice')
'3 mice'
```

让我们进行一点“类型驱动的开发”。首先我们添加一个使用第三个参数的测试。不要忘记为测试函数添加返回类型提示，否则 Mypy 将不会检查它。

```py
def test_irregular() -> None:
    got = show_count(2, 'child', 'children')
    assert got == '2 children'
```

Mypy 检测到了错误：

```py
…/hints_2/ $ mypy messages_test.py
messages_test.py:22: error: Too many arguments for "show_count"
Found 1 error in 1 file (checked 1 source file)
```

现在我编辑`show_count`，在示例 8-3 中添加了可选的`plural`参数。

##### 示例 8-3\. *hints_2/messages.py*中带有可选参数的`showcount`

```py
def show_count(count: int, singular: str, plural: str = '') -> str:
    if count == 1:
        return f'1 {singular}'
    count_str = str(count) if count else 'no'
    if not plural:
        plural = singular + 's'
    return f'{count_str} {plural}'
```

现在 Mypy 报告“成功”。

###### 警告

这里有一个 Python 无法捕捉的类型错误。你能发现吗？

```py
def hex2rgb(color=str) -> tuple[int, int, int]:
```

Mypy 的错误报告并不是很有帮助：

```py
colors.py:24: error: Function is missing a type
    annotation for one or more arguments
```

`color`参数的类型提示应为`color: str`。我写成了`color=str`，这不是一个注释：它将`color`的默认值设置为`str`。

根据我的经验，这是一个常见的错误，很容易忽视，特别是在复杂的类型提示中。

以下细节被认为是类型提示的良好风格：

+   参数名和`:`之间没有空格；`:`后有一个空格

+   在默认参数值之前的`=`两侧留有空格

另一方面，PEP 8 表示如果对于特定参数没有类型提示，则`=`周围不应有空格。

## 使用`None`作为默认值

在示例 8-3 中，参数`plural`被注释为`str`，默认值为`''`，因此没有类型冲突。

我喜欢那个解决方案，但在其他情况下，`None`是更好的默认值。如果可选参数期望一个可变类型，那么`None`是唯一明智的默认值——正如我们在“可变类型作为参数默认值：不好的主意”中看到的。

要将`None`作为`plural`参数的默认值，签名将如下所示：

```py
from typing import Optional

def show_count(count: int, singular: str, plural: Optional[str] = None) -> str:
```

让我们解开这个问题：

+   `Optional[str]`表示`plural`可以是`str`或`None`。

+   你必须明确提供默认值`= None`。

如果你没有为`plural`分配默认值，Python 运行时将把它视为必需参数。记住：在运行时，类型提示会被忽略。

请注意，我们需要从`typing`模块导入`Optional`。在导入类型时，使用语法`from typing import X`是一个好习惯，可以缩短函数签名的长度。

###### 警告

`Optional`不是一个很好的名称，因为该注释并不使参数变为可选的。使其可选的是为参数分配默认值。`Optional[str]`只是表示：该参数的类型可以是`str`或`NoneType`。在 Haskell 和 Elm 语言中，类似的类型被命名为`Maybe`。

现在我们已经初步了解了渐进类型，让我们考虑在实践中“类型”这个概念意味着什么。

# 类型由支持的操作定义

> 文献中对类型概念有许多定义。在这里，我们假设类型是一组值和一组可以应用于这些值的函数。
> 
> PEP 483—类型提示的理论

在实践中，将支持的操作集合视为类型的定义特征更有用。⁴

例如，从适用操作的角度来看，在以下函数中`x`的有效类型是什么？

```py
def double(x):
    return x * 2
```

`x`参数类型可以是数值型（`int`、`complex`、`Fraction`、`numpy.uint32`等），但也可以是序列（`str`、`tuple`、`list`、`array`）、N 维`numpy.array`，或者任何实现或继承接受`int`参数的`__mul__`方法的其他类型。

然而，请考虑这个带注释的 `double`。现在请忽略缺失的返回类型，让我们专注于参数类型：

```py
from collections import abc

def double(x: abc.Sequence):
    return x * 2
```

类型检查器将拒绝该代码。如果告诉 Mypy `x` 的类型是 `abc.Sequence`，它将标记 `x * 2` 为错误，因为 [`Sequence` ABC](https://fpy.li/8-13) 没有实现或继承 `__mul__` 方法。在运行时，该代码将与具体序列（如 `str`、`tuple`、`list`、`array` 等）以及数字一起工作，因为在运行时会忽略类型提示。但类型检查器只关心显式声明的内容，`abc.Sequence` 没有 `__mul__`。

这就是为什么这一节的标题是“类型由支持的操作定义”。Python 运行时接受任何对象作为 `x` 参数传递给 `double` 函数的两个版本。计算 `x * 2` 可能有效，也可能会引发 `TypeError`，如果 `x` 不支持该操作。相比之下，Mypy 在分析带注释的 `double` 源代码时会声明 `x * 2` 为错误，因为它对于声明的类型 `x: abc.Sequence` 是不支持的操作。

在渐进式类型系统中，我们有两种不同类型观点的相互作用：

鸭子类型

Smalltalk——开创性的面向对象语言——以及 Python、JavaScript 和 Ruby 采用的视角。对象具有类型，但变量（包括参数）是无类型的。实际上，对象的声明类型是什么并不重要，只有它实际支持的操作才重要。如果我可以调用 `birdie.quack()`，那么在这个上下文中 `birdie` 就是一只鸭子。根据定义，鸭子类型只在运行时强制执行，当尝试对对象进行操作时。这比*名义类型*更灵活，但会在运行时允许更多的错误。⁵

名义类型

C++、Java 和 C# 采用的视角，由带注释的 Python 支持。对象和变量具有类型。但对象只在运行时存在，类型检查器只关心在变量（包括参数）被注释为类型提示的源代码中。如果 `Duck` 是 `Bird` 的一个子类，你可以将一个 `Duck` 实例分配给一个被注释为 `birdie: Bird` 的参数。但在函数体内，类型检查器认为调用 `birdie.quack()` 是非法的，因为 `birdie` 名义上是一个 `Bird`，而该类不提供 `.quack()` 方法。在运行时实际参数是 `Duck` 也无关紧要，因为名义类型是静态强制的。类型检查器不运行程序的任何部分，它只读取源代码。这比*鸭子类型*更严格，优点是在构建流水线中更早地捕获一些错误，甚至在代码在 IDE 中输入时。

Example 8-4 是一个愚蠢的例子，对比了鸭子类型和名义类型，以及静态类型检查和运行时行为。⁶

##### 示例 8-4\. *birds.py*

```py
class Bird:
    pass

class Duck(Bird):  # ①
    def quack(self):
        print('Quack!')

def alert(birdie):  # ②
    birdie.quack()

def alert_duck(birdie: Duck) -> None:  # ③
    birdie.quack()

def alert_bird(birdie: Bird) -> None:  # ④
    birdie.quack()
```

①

`Duck` 是 `Bird` 的一个子类。

②

`alert` 没有类型提示，因此类型检查器会忽略它。

③

`alert_duck` 接受一个 `Duck` 类型的参数。

④

`alert_bird` 接受一个 `Bird` 类型的参数。

使用 Mypy 对 *birds.py* 进行类型检查，我们发现了一个问题：

```py
…/birds/ $ mypy birds.py
birds.py:16: error: "Bird" has no attribute "quack"
Found 1 error in 1 file (checked 1 source file)
```

通过分析源代码，Mypy 发现 `alert_bird` 是有问题的：类型提示声明了 `birdie` 参数的类型为 `Bird`，但函数体调用了 `birdie.quack()`，而 `Bird` 类没有这样的方法。

现在让我们尝试在 *daffy.py* 中使用 `birds` 模块，参见 Example 8-5。

##### 示例 8-5\. *daffy.py*

```py
from birds import *

daffy = Duck()
alert(daffy)       # ①
alert_duck(daffy)  # ②
alert_bird(daffy)  # ③
```

①

这是有效的调用，因为 `alert` 没有类型提示。

②

这是有效的调用，因为 `alert_duck` 接受一个 `Duck` 参数，而 `daffy` 是一个 `Duck`。

③

有效的调用，因为`alert_bird`接受一个`Bird`参数，而`daffy`也是一个`Bird`——`Duck`的超类。

在*daffy.py*上运行 Mypy 会引发与在*birds.py*中定义的`alert_bird`函数中的`quack`调用相同的错误：

```py
…/birds/ $ mypy daffy.py
birds.py:16: error: "Bird" has no attribute "quack"
Found 1 error in 1 file (checked 1 source file)
```

但是 Mypy 对*daffy.py*本身没有任何问题：这三个函数调用都是正确的。

现在，如果你运行*daffy.py*，你会得到以下结果：

```py
…/birds/ $ python3 daffy.py
Quack!
Quack!
Quack!
```

一切正常！鸭子类型万岁！

在运行时，Python 不关心声明的类型。它只使用鸭子类型。Mypy 在`alert_bird`中标记了一个错误，但在运行时使用`daffy`调用它是没有问题的。这可能会让许多 Python 爱好者感到惊讶：静态类型检查器有时会发现我们知道会执行的程序中的错误。

然而，如果几个月后你被要求扩展这个愚蠢的鸟类示例，你可能会感激 Mypy。考虑一下*woody.py*模块，它也使用了`birds`，在示例 8-6 中。

##### 示例 8-6\. *woody.py*

```py
from birds import *

woody = Bird()
alert(woody)
alert_duck(woody)
alert_bird(woody)
```

Mypy 在检查*woody.py*时发现了两个错误：

```py
…/birds/ $ mypy woody.py
birds.py:16: error: "Bird" has no attribute "quack"
woody.py:5: error: Argument 1 to "alert_duck" has incompatible type "Bird";
expected "Duck"
Found 2 errors in 2 files (checked 1 source file)
```

第一个错误在*birds.py*中：在`alert_bird`中的`birdie.quack()`调用，我们之前已经看过了。第二个错误在*woody.py*中：`woody`是`Bird`的一个实例，所以调用`alert_duck(woody)`是无效的，因为该函数需要一个`Duck`。每个`Duck`都是一个`Bird`，但并非每个`Bird`都是一个`Duck`。

在运行时，*woody.py*中的所有调用都失败了。这些失败的连续性在示例 8-7 中的控制台会话中最好地说明。

##### 示例 8-7\. 运行时错误以及 Mypy 如何帮助

```py
>>> from birds import *
>>> woody = Bird()
>>> alert(woody)  # ①
Traceback (most recent call last):
  ...
AttributeError: 'Bird' object has no attribute 'quack'
>>>
>>> alert_duck(woody) # ②
Traceback (most recent call last):
  ...
AttributeError: 'Bird' object has no attribute 'quack'
>>>
>>> alert_bird(woody)  # ③
Traceback (most recent call last):
  ...
AttributeError: 'Bird' object has no attribute 'quack'
```

①

Mypy 无法检测到这个错误，因为`alert`中没有类型提示。

②

Mypy 报告了问题："alert_duck"的第 1 个参数类型不兼容："Bird"；预期是"Duck"。

③

自从示例 8-4 以来，Mypy 一直在告诉我们`alert_bird`函数的主体是错误的："Bird"没有属性"quack"。

这个小实验表明，鸭子类型更容易上手，更加灵活，但允许不支持的操作在运行时引发错误。名义类型在运行前检测错误，但有时可能会拒绝实际运行的代码，比如在示例 8-5 中的调用`alert_bird(daffy)`。即使有时候能够运行，`alert_bird`函数的命名是错误的：它的主体确实需要支持`.quack()`方法的对象，而`Bird`没有这个方法。

在这个愚蠢的例子中，函数只有一行。但在实际代码中，它们可能会更长；它们可能会将`birdie`参数传递给更多函数，并且`birdie`参数的来源可能相距多个函数调用，这使得很难准确定位运行时错误的原因。类型检查器可以防止许多这样的错误在运行时发生。

###### 注意

类型提示在适合放在书中的小例子中的价值是有争议的。随着代码库规模的增长，其好处也会增加。这就是为什么拥有数百万行 Python 代码的公司——如 Dropbox、Google 和 Facebook——投资于团队和工具，支持公司范围内采用类型提示，并在 CI 管道中检查其 Python 代码库的重要部分。

在本节中，我们探讨了鸭子类型和名义类型中类型和操作的关系，从简单的`double()`函数开始——我们没有为其添加适当的类型提示。当我们到达“静态协议”时，我们将看到如何为`double()`添加类型提示。但在那之前，还有更基本的类型需要了解。

# 可用于注释的类型

几乎任何 Python 类型都可以用作类型提示，但存在限制和建议。此外，`typing`模块引入了有时令人惊讶的语义的特殊构造。

本节涵盖了您可以在注释中使用的所有主要类型：

+   `typing.Any`

+   简单类型和类

+   `typing.Optional`和`typing.Union`

+   泛型集合，包括元组和映射

+   抽象基类

+   通用可迭代对象

+   参数化泛型和`TypeVar`

+   `typing.Protocols`—*静态鸭子类型*的关键

+   `typing.Callable`

+   `typing.NoReturn`—一个结束这个列表的好方法

我们将依次介绍每一个，从一个奇怪的、显然无用但至关重要的类型开始。

## 任意类型

任何渐进式类型系统的基石是`Any`类型，也称为*动态类型*。当类型检查器看到这样一个未标记的函数时：

```py
def double(x):
    return x * 2
```

它假设这个：

```py
def double(x: Any) -> Any:
    return x * 2
```

这意味着`x`参数和返回值可以是任何类型，包括不同的类型。假定`Any`支持每种可能的操作。

将`Any`与`object`进行对比。考虑这个签名：

```py
def double(x: object) -> object:
```

这个函数也接受每种类型的参数，因为每种类型都是`object`的*子类型*。

然而，类型检查器将拒绝这个函数：

```py
def double(x: object) -> object:
    return x * 2
```

问题在于`object`不支持`__mul__`操作。这就是 Mypy 报告的内容：

```py
…/birds/ $ mypy double_object.py
double_object.py:2: error: Unsupported operand types for * ("object" and "int")
Found 1 error in 1 file (checked 1 source file)
```

更一般的类型具有更窄的接口，即它们支持更少的操作。`object`类实现的操作比`abc.Sequence`少，`abc.Sequence`实现的操作比`abc.MutableSequence`少，`abc.MutableSequence`实现的操作比`list`少。

但`Any`是一个神奇的类型，它同时位于类型层次结构的顶部和底部。它同时是最一般的类型—所以一个参数`n: Any`接受每种类型的值—和最专门的类型，支持每种可能的操作。至少，这就是类型检查器如何理解`Any`。

当然，没有任何类型可以支持每种可能的操作，因此使用`Any`可以防止类型检查器实现其核心任务：在程序因运行时异常而崩溃之前检测潜在的非法操作。

### 子类型与一致性

传统的面向对象的名义类型系统依赖于*子类型*关系。给定一个类`T1`和一个子类`T2`，那么`T2`是`T1`的*子类型*。

考虑这段代码：

```py
class T1:
    ...

class T2(T1):
    ...

def f1(p: T1) -> None:
    ...

o2 = T2()

f1(o2)  # OK
```

调用`f1(o2)`是对 Liskov 替换原则—LSP 的应用。Barbara Liskov⁷实际上是根据支持的操作定义*是子类型*：如果类型`T2`的对象替代类型`T1`的对象并且程序仍然正确运行，那么`T2`就是`T1`的*子类型*。

继续上述代码，这显示了 LSP 的违反：

```py
def f2(p: T2) -> None:
    ...

o1 = T1()

f2(o1)  # type error
```

从支持的操作的角度来看，这是完全合理的：作为一个子类，`T2`继承并且必须支持`T1`支持的所有操作。因此，`T2`的实例可以在期望`T1`的实例的任何地方使用。但反之不一定成立：`T2`可能实现额外的方法，因此`T1`的实例可能无法在期望`T2`的实例的任何地方使用。这种对支持的操作的关注体现在名称[*行为子类型化*](https://fpy.li/8-15)中，也用于指代 LSP。

在渐进式类型系统中，还有另一种关系：*与一致*，它适用于*子类型*适用的地方，对于类型`Any`有特殊规定。

*与一致*的规则是：

1.  给定`T1`和子类型`T2`，那么`T2`是*与*`T1`一致的（Liskov 替换）。

1.  每种类型都*与一致*`Any`：你可以将每种类型的对象传递给声明为`Any`类型的参数。

1.  `Any`是*与每种类型一致*的：你总是可以在需要另一种类型的参数时传递一个`Any`类型的对象。

考虑前面定义的对象`o1`和`o2`，这里是有效代码的示例，说明规则#2 和#3：

```py
def f3(p: Any) -> None:
    ...

o0 = object()
o1 = T1()
o2 = T2()

f3(o0)  #
f3(o1)  #  all OK: rule #2
f3(o2)  #

def f4():  # implicit return type: `Any`
    ...

o4 = f4()  # inferred type: `Any`

f1(o4)  #
f2(o4)  #  all OK: rule #3
f3(o4)  #
```

每个渐进类型系统都需要像`Any`这样的通配类型。

###### 提示

动词“推断”是“猜测”的花哨同义词，在类型分析的背景下使用。Python 和其他语言中的现代类型检查器并不要求在每个地方都有类型注释，因为它们可以推断出许多表达式的类型。例如，如果我写`x = len(s) * 10`，类型检查器不需要一个显式的本地声明来知道`x`是一个`int`，只要它能找到`len`内置函数的类型提示即可。

现在我们可以探索注解中使用的其余类型。

## 简单类型和类

像`int`、`float`、`str`和`bytes`这样的简单类型可以直接在类型提示中使用。标准库、外部包或用户定义的具体类——`FrenchDeck`、`Vector2d`和`Duck`——也可以在类型提示中使用。

抽象基类在类型提示中也很有用。当我们研究集合类型时，我们将回到它们，并在“抽象基类”中看到它们。

在类之间，*一致*的定义类似于*子类型*：子类与其所有超类一致。

然而，“实用性胜过纯粹性”，因此有一个重要的例外情况，我将在下面的提示中讨论。

# int 与复杂一致

内置类型`int`、`float`和`complex`之间没有名义子类型关系：它们是`object`的直接子类。但 PEP 484[声明](https://fpy.li/cardxvi) `int`与`float`一致，`float`与`complex`一致。在实践中是有道理的：`int`实现了`float`的所有操作，而且`int`还实现了额外的操作——位运算如`&`、`|`、`<<`等。最终结果是：`int`与`complex`一致。对于`i = 3`，`i.real`是`3`，`i.imag`是`0`。

## 可选和联合类型

我们在“使用 None 作为默认值”中看到了`Optional`特殊类型。它解决了将`None`作为默认值的问题，就像这个部分中的示例一样：

```py
from typing import Optional

def show_count(count: int, singular: str, plural: Optional[str] = None) -> str:
```

构造`Optional[str]`实际上是`Union[str, None]`的快捷方式，这意味着`plural`的类型可以是`str`或`None`。

# Python 3.10 中更好的可选和联合语法

自 Python 3.10 起，我们可以写`str | bytes`而不是`Union[str, bytes]`。这样打字更少，而且不需要从`typing`导入`Optional`或`Union`。对比`show_count`的`plural`参数的类型提示的旧语法和新语法：

```py
plural: Optional[str] = None    # before
plural: str | None = None       # after
```

`|`运算符也适用于`isinstance`和`issubclass`来构建第二个参数：`isinstance(x, int | str)`。更多信息，请参阅[PEP 604—Union[]的补充语法](https://fpy.li/pep604)。

`ord`内置函数的签名是`Union`的一个简单示例——它接受`str`或`bytes`，并返回一个`int`:⁸

```py
def ord(c: Union[str, bytes]) -> int: ...
```

这是一个接受`str`但可能返回`str`或`float`的函数示例：

```py
from typing import Union

def parse_token(token: str) -> Union[str, float]:
    try:
        return float(token)
    except ValueError:
        return token
```

如果可能的话，尽量避免创建返回`Union`类型的函数，因为这会给用户增加额外的负担——迫使他们在运行时检查返回值的类型以知道如何处理它。但在前面代码中的`parse_token`是一个简单表达式求值器上下文中合理的用例。

###### 提示

在“双模式 str 和 bytes API”中，我们看到接受`str`或`bytes`参数的函数，但如果参数是`str`则返回`str`，如果参数是`bytes`则返回`bytes`。在这些情况下，返回类型由输入类型确定，因此`Union`不是一个准确的解决方案。为了正确注释这样的函数，我们需要一个类型变量—在“参数化泛型和 TypeVar”中介绍—或重载，我们将在“重载签名”中看到。

`Union[]`需要至少两种类型。嵌套的`Union`类型与扁平化的`Union`具有相同的效果。因此，这种类型提示：

```py
Union[A, B, Union[C, D, E]]
```

与以下相同：

```py
Union[A, B, C, D, E]
```

`Union` 对于彼此不一致的类型更有用。例如：`Union[int, float]` 是多余的，因为 `int` 与 `float` 是一致的。如果只使用 `float` 来注释参数，它也将接受 `int` 值。

## 泛型集合

大多数 Python 集合是异构的。例如，你可以在 `list` 中放入任何不同类型的混合物。然而，在实践中，这并不是非常有用：如果将对象放入集合中，你可能希望以后对它们进行操作，通常这意味着它们必须至少共享一个公共方法。⁹

可以声明带有类型参数的泛型类型，以指定它们可以处理的项目的类型。

例如，一个 `list` 可以被参数化以限制其中元素的类型，就像你在 示例 8-8 中看到的那样。

##### 示例 8-8\. `tokenize` 中的 Python ≥ 3.9 类型提示

```py
def tokenize(text: str) -> list[str]:
    return text.upper().split()
```

在 Python ≥ 3.9 中，这意味着 `tokenize` 返回一个每个项目都是 `str` 类型的 `list`。

注释 `stuff: list` 和 `stuff: list[Any]` 意味着相同的事情：`stuff` 是任意类型对象的列表。

###### 提示

如果你使用的是 Python 3.8 或更早版本，概念是相同的，但你需要更多的代码来使其工作，如可选框中所解释的 “遗留支持和已弃用的集合类型”。

[PEP 585—标准集合中的泛型类型提示](https://fpy.li/8-16) 列出了接受泛型类型提示的标准库集合。以下列表仅显示那些使用最简单形式的泛型类型提示 `container[item]` 的集合：

```py
list        collections.deque        abc.Sequence   abc.MutableSequence
set         abc.Container            abc.Set        abc.MutableSet
frozenset   abc.Collection
```

`tuple` 和映射类型支持更复杂的类型提示，我们将在各自的部分中看到。

截至 Python 3.10，目前还没有很好的方法来注释 `array.array`，考虑到 `typecode` 构造参数，该参数确定数组中存储的是整数还是浮点数。更难的问题是如何对整数范围进行类型检查，以防止在向数组添加元素时在运行时出现 `OverflowError`。例如，具有 `typecode='B'` 的 `array` 只能容纳从 0 到 255 的 `int` 值。目前，Python 的静态类型系统还无法应对这一挑战。

现在让我们看看如何注释泛型元组。

## 元组类型

有三种注释元组类型的方法：

+   元组作为记录

+   具有命名字段的元组作为记录

+   元组作为不可变序列

### 元组作为记录

如果将 `tuple` 用作记录，则使用内置的 `tuple` 并在 `[]` 中声明字段的类型。

例如，类型提示将是 `tuple[str, float, str]`，以接受包含城市名称、人口和国家的元组：`('上海', 24.28, '中国')`。

考虑一个接受一对地理坐标并返回 [Geohash](https://fpy.li/8-18) 的函数，用法如下：

```py
>>> shanghai = 31.2304, 121.4737
>>> geohash(shanghai)
'wtw3sjq6q'
```

示例 8-11 展示了如何定义 `geohash`，使用了来自 PyPI 的 `geolib` 包。

##### 示例 8-11\. *coordinates.py* 中的 `geohash` 函数

```py
from geolib import geohash as gh  # type: ignore # ①

PRECISION = 9

def geohash(lat_lon: tuple[float, float]) -> str:  # ②
    return gh.encode(*lat_lon, PRECISION)
```

①

此注释阻止 Mypy 报告 `geolib` 包没有类型提示。

②

`lat_lon` 参数注释为具有两个 `float` 字段的 `tuple`。

###### 提示

对于 Python < 3.9，导入并在类型提示中使用 `typing.Tuple`。它已被弃用，但至少会保留在标准库中直到 2024 年。

### 具有命名字段的元组作为记录

要为具有许多字段的元组或代码中多处使用的特定类型的元组添加注释，我强烈建议使用 `typing.NamedTuple`，如 第五章 中所示。示例 8-12 展示了使用 `NamedTuple` 对 示例 8-11 进行变体的情况。

##### 示例 8-12\. *coordinates_named.py* 中的 `NamedTuple` `Coordinates` 和 `geohash` 函数

```py
from typing import NamedTuple

from geolib import geohash as gh  # type: ignore

PRECISION = 9

class Coordinate(NamedTuple):
    lat: float
    lon: float

def geohash(lat_lon: Coordinate) -> str:
    return gh.encode(*lat_lon, PRECISION)
```

如“数据类构建器概述”中所解释的，`typing.NamedTuple`是`tuple`子类的工厂，因此`Coordinate`与`tuple[float, float]`是*一致的*，但反之则不成立——毕竟，`Coordinate`具有`NamedTuple`添加的额外方法，如`._asdict()`，还可以有用户定义的方法。

在实践中，这意味着将`Coordinate`实例传递给以下定义的`display`函数是类型安全的：

```py
def display(lat_lon: tuple[float, float]) -> str:
    lat, lon = lat_lon
    ns = 'N' if lat >= 0 else 'S'
    ew = 'E' if lon >= 0 else 'W'
    return f'{abs(lat):0.1f}°{ns}, {abs(lon):0.1f}°{ew}'
```

### 元组作为不可变序列

要注释用作不可变列表的未指定长度元组，必须指定一个类型，后跟逗号和`...`（这是 Python 的省略号标记，由三个句点组成，而不是 Unicode `U+2026`—`水平省略号`）。

例如，`tuple[int, ...]`是一个具有`int`项的元组。

省略号表示接受任意数量的元素>= 1。无法指定任意长度元组的不同类型字段。

注释`stuff: tuple[Any, ...]`和`stuff: tuple`意思相同：`stuff`是一个未指定长度的包含任何类型对象的元组。

这里是一个`columnize`函数，它将一个序列转换为行和单元格的表格，形式为未指定长度的元组列表。这对于以列形式显示项目很有用，就像这样：

```py
>>> animals = 'drake fawn heron ibex koala lynx tahr xerus yak zapus'.split()
>>> table = columnize(animals)
>>> table
[('drake', 'koala', 'yak'), ('fawn', 'lynx', 'zapus'), ('heron', 'tahr'),
 ('ibex', 'xerus')]
>>> for row in table:
...     print(''.join(f'{word:10}' for word in row))
...
drake     koala     yak
fawn      lynx      zapus
heron     tahr
ibex      xerus
```

示例 8-13 展示了`columnize`的实现。注意返回类型：

```py
list[tuple[str, ...]]
```

##### 示例 8-13\. *columnize.py*返回一个字符串元组列表

```py
from collections.abc import Sequence

def columnize(
    sequence: Sequence[str], num_columns: int = 0
) -> list[tuple[str, ...]]:
    if num_columns == 0:
        num_columns = round(len(sequence) ** 0.5)
    num_rows, reminder = divmod(len(sequence), num_columns)
    num_rows += bool(reminder)
    return [tuple(sequence[i::num_rows]) for i in range(num_rows)]
```

## 通用映射

通用映射类型被注释为`MappingType[KeyType, ValueType]`。内置的`dict`和`collections`以及`collections.abc`中的映射类型在 Python ≥ 3.9 中接受该表示法。对于早期版本，必须使用`typing.Dict`和`typing`模块中的其他映射类型，如“遗留支持和已弃用的集合类型”中所述。

示例 8-14 展示了一个函数返回[倒排索引](https://fpy.li/8-19)以通过名称搜索 Unicode 字符的实际用途——这是示例 4-21 的一个变体，更适合我们将在第二十一章中学习的服务器端代码。

给定起始和结束的 Unicode 字符代码，`name_index`返回一个`dict[str, set[str]]`，这是一个将每个单词映射到具有该单词在其名称中的字符集的倒排索引。例如，在对 ASCII 字符从 32 到 64 进行索引后，这里是映射到单词`'SIGN'`和`'DIGIT'`的字符集，以及如何找到名为`'DIGIT EIGHT'`的字符：

```py
>>> index = name_index(32, 65)
>>> index['SIGN']
{'$', '>', '=', '+', '<', '%', '#'}
>>> index['DIGIT']
{'8', '5', '6', '2', '3', '0', '1', '4', '7', '9'}
>>> index['DIGIT'] & index['EIGHT']
{'8'}
```

示例 8-14 展示了带有`name_index`函数的*charindex.py*源代码。除了`dict[]`类型提示外，这个示例还有三个本书中首次出现的特性。

##### 示例 8-14\. *charindex.py*

```py
import sys
import re
import unicodedata
from collections.abc import Iterator

RE_WORD = re.compile(r'\w+')
STOP_CODE = sys.maxunicode + 1

def tokenize(text: str) -> Iterator[str]:  # ①
    """return iterable of uppercased words"""
    for match in RE_WORD.finditer(text):
        yield match.group().upper()

def name_index(start: int = 32, end: int = STOP_CODE) -> dict[str, set[str]]:
    index: dict[str, set[str]] = {}  # ②
    for char in (chr(i) for i in range(start, end)):
        if name := unicodedata.name(char, ''):  # ③
            for word in tokenize(name):
                index.setdefault(word, set()).add(char)
    return index
```

①

`tokenize`是一个生成器函数。第十七章是关于生成器的。

②

局部变量`index`已经被注释。没有提示，Mypy 会说：`需要为'index'注释类型（提示：“index: dict[<type>, <type>] = ...”）`。

③

我在`if`条件中使用了海象操作符`:=`。它将`unicodedata.name()`调用的结果赋给`name`，整个表达式的值就是该结果。当结果为`''`时，为假值，`index`不会被更新。¹¹

###### 注意

当将`dict`用作记录时，通常所有键都是`str`类型，具体取决于键的不同类型的值。这在“TypedDict”中有所涵盖。

## 抽象基类

> 在发送内容时要保守，在接收内容时要开放。
> 
> 波斯特尔法则，又称韧性原则

表 8-1 列出了几个来自 `collections.abc` 的抽象类。理想情况下，一个函数应该接受这些抽象类型的参数，或者在 Python 3.9 之前使用它们的 `typing` 等效类型，而不是具体类型。这样可以给调用者更多的灵活性。

考虑这个函数签名：

```py
from collections.abc import Mapping

def name2hex(name: str, color_map: Mapping[str, int]) -> str:
```

使用 `abc.Mapping` 允许调用者提供 `dict`、`defaultdict`、`ChainMap`、`UserDict` 子类或任何其他是 `Mapping` 的*子类型*的类型的实例。

相比之下，考虑这个签名：

```py
def name2hex(name: str, color_map: dict[str, int]) -> str:
```

现在 `color_map` 必须是一个 `dict` 或其子类型之一，比如 `defaultDict` 或 `OrderedDict`。特别是，`collections.UserDict` 的子类不会通过 `color_map` 的类型检查，尽管这是创建用户定义映射的推荐方式，正如我们在 “子类化 UserDict 而不是 dict” 中看到的那样。Mypy 会拒绝 `UserDict` 或从它派生的类的实例，因为 `UserDict` 不是 `dict` 的子类；它们是同级。两者都是 `abc.MutableMapping` 的子类。¹²

因此，一般来说最好在参数类型提示中使用 `abc.Mapping` 或 `abc.MutableMapping`，而不是 `dict`（或在旧代码中使用 `typing.Dict`）。如果 `name2hex` 函数不需要改变给定的 `color_map`，那么 `color_map` 的最准确的类型提示是 `abc.Mapping`。这样，调用者不需要提供实现 `setdefault`、`pop` 和 `update` 等方法的对象，这些方法是 `MutableMapping` 接口的一部分，但不是 `Mapping` 的一部分。这与 Postel 法则的第二部分有关：“在接受输入时要宽容。”

Postel 法则还告诉我们在发送内容时要保守。函数的返回值始终是一个具体对象，因此返回类型提示应该是一个具体类型，就像来自 “通用集合” 的示例一样—使用 `list[str]`：

```py
def tokenize(text: str) -> list[str]:
    return text.upper().split()
```

在 [`typing.List`](https://fpy.li/8-20) 的条目中，Python 文档中写道：

> `list` 的泛型版本。用于注释返回类型。为了注释参数，最好使用抽象集合类型，如 `Sequence` 或 `Iterable`。

在 [`typing.Dict`](https://fpy.li/8-21) 和 [`typing.Set`](https://fpy.li/8-22) 的条目中也有类似的评论。

请记住，`collections.abc` 中的大多数 ABCs 和其他具体类，以及内置集合，都支持类似 `collections.deque[str]` 的泛型类型提示符号，从 Python 3.9 开始。相应的 `typing` 集合仅需要支持在 Python 3.8 或更早版本中编写的代码。变成泛型的类的完整列表出现在 [“实现”](https://fpy.li/8-16) 部分的 [PEP 585—标准集合中的类型提示泛型](https://fpy.li/pep585) 中。

结束我们关于类型提示中 ABCs 的讨论，我们需要谈一谈 `numbers` ABCs。

### 数字塔的崩塌

[`numbers`](https://fpy.li/8-24) 包定义了在 [PEP 3141—为数字定义的类型层次结构](https://fpy.li/pep3141) 中描述的所谓*数字塔*。该塔是一种线性的 ABC 层次结构，顶部是 `Number`：

+   `Number`

+   `Complex`

+   `Real`

+   `Rational`

+   `Integral`

这些 ABCs 对于运行时类型检查非常有效，但不支持静态类型检查。PEP 484 的 [“数字塔”](https://fpy.li/cardxvi) 部分拒绝了 `numbers` ABCs，并规定内置类型 `complex`、`float` 和 `int` 应被视为特殊情况，如 “int 与 complex 一致” 中所解释的那样。

我们将在 “numbers ABCs 和数字协议” 中回到这个问题，在 第十三章 中，该章节专门对比协议和 ABCs。

实际上，如果您想要为静态类型检查注释数字参数，您有几个选择：

1.  使用 `int`、`float` 或 `complex` 中的一个具体类型—正如 PEP 488 建议的那样。

1.  声明一个联合类型，如 `Union[float, Decimal, Fraction]`。

1.  如果想避免硬编码具体类型，请使用像 `SupportsFloat` 这样的数值协议，详见“运行时可检查的静态协议”。

即将到来的章节“静态协议”是理解数值协议的先决条件。

与此同时，让我们来看看对于类型提示最有用的 ABC 之一：`Iterable`。

## 可迭代对象

我刚引用的 [`typing.List`](https://fpy.li/8-20) 文档建议在函数参数类型提示中使用 `Sequence` 和 `Iterable`。

`Iterable` 参数的一个示例出现在标准库中的 `math.fsum` 函数中：

```py
def fsum(__seq: Iterable[float]) -> float:
```

# 存根文件和 Typeshed 项目

截至 Python 3.10，标准库没有注释，但 Mypy、PyCharm 等可以在 [Typeshed](https://fpy.li/8-26) 项目中找到必要的类型提示，形式为*存根文件*：特殊的带有 *.pyi* 扩展名的源文件，具有带注释的函数和方法签名，但没有实现——类似于 C 中的头文件。

`math.fsum` 的签名在 [*/stdlib/2and3/math.pyi*](https://fpy.li/8-27) 中。`__seq` 中的前导下划线是 PEP 484 中关于仅限位置参数的约定，解释在“注释仅限位置参数和可变参数”中。

示例 8-15 是另一个使用 `Iterable` 参数的示例，产生的项目是 `tuple[str, str]`。以下是函数的使用方式：

```py
>>> l33t = [('a', '4'), ('e', '3'), ('i', '1'), ('o', '0')]
>>> text = 'mad skilled noob powned leet'
>>> from replacer import zip_replace
>>> zip_replace(text, l33t)
'm4d sk1ll3d n00b p0wn3d l33t'
```

示例 8-15 展示了它的实现方式。

##### 示例 8-15\. *replacer.py*

```py
from collections.abc import Iterable

FromTo = tuple[str, str]  # ①

def zip_replace(text: str, changes: Iterable[FromTo]) -> str:  # ②
    for from_, to in changes:
        text = text.replace(from_, to)
    return text
```

①

`FromTo` 是一个*类型别名*：我将 `tuple[str, str]` 赋给 `FromTo`，以使 `zip_replace` 的签名更易读。

②

`changes` 需要是一个 `Iterable[FromTo]`；这与 `Iterable[tuple[str, str]]` 相同，但更短且更易读。

# Python 3.10 中的显式 TypeAlias

[PEP 613—显式类型别名](https://fpy.li/pep613)引入了一个特殊类型，`TypeAlias`，用于使创建类型别名的赋值更加可见和易于类型检查。从 Python 3.10 开始，这是创建类型别名的首选方式：

```py
from typing import TypeAlias

FromTo: TypeAlias = tuple[str, str]
```

### abc.Iterable 与 abc.Sequence

`math.fsum` 和 `replacer.zip_replace` 都必须遍历整个 `Iterable` 参数才能返回结果。如果给定一个无限迭代器，比如 `itertools.cycle` 生成器作为输入，这些函数将消耗所有内存并导致 Python 进程崩溃。尽管存在潜在的危险，但在现代 Python 中，提供接受 `Iterable` 输入的函数即使必须完全处理它才能返回结果是相当常见的。这样一来，调用者可以选择将输入数据提供为生成器，而不是预先构建的序列，如果输入项的数量很大，可能会节省大量内存。

另一方面，来自示例 8-13 的 `columnize` 函数需要一个 `Sequence` 参数，而不是 `Iterable`，因为它必须获取输入的 `len()` 来提前计算行数。

与 `Sequence` 类似，`Iterable` 最适合用作参数类型。作为返回类型太模糊了。函数应该更加精确地说明返回的具体类型。

与 `Iterable` 密切相关的是 `Iterator` 类型，在 示例 8-14 中用作返回类型。我们将在第十七章中回到这个话题，讨论生成器和经典迭代器。

## 参数化泛型和 TypeVar

参数化泛型是一种泛型类型，写作 `list[T]`，其中 `T` 是一个类型变量，将在每次使用时绑定到特定类型。这允许参数类型反映在结果类型上。

示例 8-16 定义了`sample`，一个接受两个参数的函数：类型为`T`的元素的`Sequence`和一个`int`。它从第一个参数中随机选择的相同类型`T`的元素的`list`。

示例 8-16 展示了实现。

##### 示例 8-16。*sample.py*

```py
from collections.abc import Sequence
from random import shuffle
from typing import TypeVar

T = TypeVar('T')

def sample(population: Sequence[T], size: int) -> list[T]:
    if size < 1:
        raise ValueError('size must be >= 1')
    result = list(population)
    shuffle(result)
    return result[:size]
```

这里有两个例子说明我在`sample`中使用了一个类型变量：

+   如果使用类型为`tuple[int, ...]`的元组——这与`Sequence[int]`一致——那么类型参数是`int`，因此返回类型是`list[int]`。

+   如果使用`str`——这与`Sequence[str]`一致——那么类型参数是`str`，因此返回类型是`list[str]`。

# 为什么需要 TypeVar？

PEP 484 的作者希望通过添加`typing`模块引入类型提示，而不改变语言的其他任何内容。通过巧妙的元编程，他们可以使`[]`运算符在类似`Sequence[T]`的类上起作用。但括号内的`T`变量名称必须在某处定义，否则 Python 解释器需要进行深层更改才能支持通用类型符号作为`[]`的特殊用途。这就是为什么需要`typing.TypeVar`构造函数：引入当前命名空间中的变量名称。像 Java、C#和 TypeScript 这样的语言不需要事先声明类型变量的名称，因此它们没有 Python 的`TypeVar`类的等价物。

另一个例子是标准库中的`statistics.mode`函数，它返回系列中最常见的数据点。

这里是来自[文档](https://fpy.li/8-28)的一个使用示例：

```py
>>> mode([1, 1, 2, 3, 3, 3, 3, 4])
3
```

如果不使用`TypeVar`，`mode`可能具有示例 8-17 中显示的签名。

##### 示例 8-17。*mode_float.py*：对`float`和子类型进行操作的`mode`¹³

```py
from collections import Counter
from collections.abc import Iterable

def mode(data: Iterable[float]) -> float:
    pairs = Counter(data).most_common(1)
    if len(pairs) == 0:
        raise ValueError('no mode for empty data')
    return pairs[0][0]
```

许多`mode`的用法涉及`int`或`float`值，但 Python 还有其他数值类型，希望返回类型遵循给定`Iterable`的元素类型。我们可以使用`TypeVar`来改进该签名。让我们从一个简单但错误的参数化签名开始：

```py
from collections.abc import Iterable
from typing import TypeVar

T = TypeVar('T')

def mode(data: Iterable[T]) -> T:
```

当类型参数`T`首次出现在签名中时，它可以是任何类型。第二次出现时，它将意味着与第一次相同的类型。

因此，每个可迭代对象都与`Iterable[T]`一致，包括`collections.Counter`无法处理的不可哈希类型的可迭代对象。我们需要限制分配给`T`的可能类型。我们将在接下来的两节中看到两种方法。

### 限制的 TypeVar

`TypeVar`接受额外的位置参数来限制类型参数。我们可以改进`mode`的签名，接受特定的数字类型，就像这样：

```py
from collections.abc import Iterable
from decimal import Decimal
from fractions import Fraction
from typing import TypeVar

NumberT = TypeVar('NumberT', float, Decimal, Fraction)

def mode(data: Iterable[NumberT]) -> NumberT:
```

这比以前好，这是 2020 年 5 月 25 日`typeshed`上`statistics.pyi`存根文件中`mode`的签名。

然而，[`statistics.mode`](https://fpy.li/8-28)文档中包含了这个例子：

```py
>>> mode(["red", "blue", "blue", "red", "green", "red", "red"])
'red'
```

匆忙之间，我们可以将`str`添加到`NumberT`的定义中：

```py
NumberT = TypeVar('NumberT', float, Decimal, Fraction, str)
```

当然，这样做是有效的，但如果它接受`str`，那么`NumberT`的命名就非常不合适。更重要的是，我们不能永远列出类型，因为我们意识到`mode`可以处理它们。我们可以通过`TypeVar`的另一个特性做得更好，接下来介绍。

### 有界的 TypeVar

查看示例 8-17 中`mode`的主体，我们看到`Counter`类用于排名。Counter 基于`dict`，因此`data`可迭代对象的元素类型必须是可哈希的。

起初，这个签名似乎可以工作：

```py
from collections.abc import Iterable, Hashable

def mode(data: Iterable[Hashable]) -> Hashable:
```

现在的问题是返回项的类型是`Hashable`：一个只实现`__hash__`方法的 ABC。因此，类型检查器不会让我们对返回值做任何事情，除了调用`hash()`。并不是很有用。

解决方案是`TypeVar`的另一个可选参数：`bound`关键字参数。它为可接受的类型设置了一个上限。在示例 8-18 中，我们有`bound=Hashable`，这意味着类型参数可以是`Hashable`或其任何*子类型*。¹⁴

##### 示例 8-18。*mode_hashable.py*：与示例 8-17 相同，但具有更灵活的签名

```py
from collections import Counter
from collections.abc import Iterable, Hashable
from typing import TypeVar

HashableT = TypeVar('HashableT', bound=Hashable)

def mode(data: Iterable[HashableT]) -> HashableT:
    pairs = Counter(data).most_common(1)
    if len(pairs) == 0:
        raise ValueError('no mode for empty data')
    return pairs[0][0]
```

总结一下：

+   限制类型变量将被设置为`TypeVar`声明中命名的类型之一。

+   有界类型变量将被设置为表达式的推断类型——只要推断类型与`TypeVar`的`bound=`关键字参数中声明的边界一致即可。

###### 注意

不幸的是，声明有界`TypeVar`的关键字参数被命名为`bound=`，因为动词“绑定”通常用于表示设置变量的值，在 Python 的引用语义中最好描述为将名称绑定到值。如果关键字参数被命名为`boundary=`会更少令人困惑。

`typing.TypeVar`构造函数还有其他可选参数——`covariant`和`contravariant`——我们将在第十五章中介绍，“Variance”中涵盖。

让我们用`AnyStr`结束对`TypeVar`的介绍。

### 预定义的 AnyStr 类型变量

`typing`模块包括一个预定义的`TypeVar`，名为`AnyStr`。它的定义如下：

```py
AnyStr = TypeVar('AnyStr', bytes, str)
```

`AnyStr`在许多接受`bytes`或`str`的函数中使用，并返回给定类型的值。

现在，让我们来看看`typing.Protocol`，这是 Python 3.8 的一个新特性，可以支持更具 Python 风格的类型提示的使用。

## 静态协议

###### 注意

在面向对象编程中，“协议”概念作为一种非正式接口的概念早在 Smalltalk 中就存在，并且从一开始就是 Python 的一个基本部分。然而，在类型提示的背景下，协议是一个`typing.Protocol`子类，定义了一个类型检查器可以验证的接口。这两种类型的协议在第十三章中都有涉及。这只是在函数注释的背景下的简要介绍。

如[PEP 544—Protocols: Structural subtyping (static duck typing)](https://fpy.li/pep544)中所述，`Protocol`类型类似于 Go 中的接口：通过指定一个或多个方法来定义协议类型，并且类型检查器验证在需要该协议类型的地方这些方法是否被实现。

在 Python 中，协议定义被写作`typing.Protocol`子类。然而，*实现*协议的类不需要继承、注册或声明与*定义*协议的类的任何关系。这取决于类型检查器找到可用的协议类型并强制执行它们的使用。

这是一个可以借助`Protocol`和`TypeVar`解决的问题。假设您想创建一个函数`top(it, n)`，返回可迭代对象`it`中最大的`n`个元素：

```py
>>> top([4, 1, 5, 2, 6, 7, 3], 3)
[7, 6, 5]
>>> l = 'mango pear apple kiwi banana'.split()
>>> top(l, 3)
['pear', 'mango', 'kiwi']
>>>
>>> l2 = [(len(s), s) for s in l]
>>> l2
[(5, 'mango'), (4, 'pear'), (5, 'apple'), (4, 'kiwi'), (6, 'banana')]
>>> top(l2, 3)
[(6, 'banana'), (5, 'mango'), (5, 'apple')]
```

一个参数化的泛型`top`看起来像示例 8-19 中所示的样子。

##### 示例 8-19。带有未定义`T`类型参数的`top`函数

```py
def top(series: Iterable[T], length: int) -> list[T]:
    ordered = sorted(series, reverse=True)
    return ordered[:length]
```

问题是如何约束`T`？它不能是`Any`或`object`，因为`series`必须与`sorted`一起工作。`sorted`内置实际上接受`Iterable[Any]`，但这是因为可选参数`key`接受一个函数，该函数从每个元素计算任意排序键。如果您给`sorted`一个普通对象列表但不提供`key`参数会发生什么？让我们试试：

```py
>>> l = [object() for _ in range(4)]
>>> l
[<object object at 0x10fc2fca0>, <object object at 0x10fc2fbb0>,
<object object at 0x10fc2fbc0>, <object object at 0x10fc2fbd0>]
>>> sorted(l)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'object' and 'object'
```

错误消息显示`sorted`在可迭代对象的元素上使用`<`运算符。这就是全部吗？让我们做另一个快速实验：¹⁵

```py
>>> class Spam:
...     def __init__(self, n): self.n = n
...     def __lt__(self, other): return self.n < other.n
...     def __repr__(self): return f'Spam({self.n})'
...
>>> l = [Spam(n) for n in range(5, 0, -1)]
>>> l
[Spam(5), Spam(4), Spam(3), Spam(2), Spam(1)]
>>> sorted(l)
[Spam(1), Spam(2), Spam(3), Spam(4), Spam(5)]
```

那证实了：我可以对`Spam`列表进行`sort`，因为`Spam`实现了`__lt__`——支持`<`运算符的特殊方法。

因此，示例 8-19 中的 `T` 类型参数应该限制为实现 `__lt__` 的类型。在 示例 8-18 中，我们需要一个实现 `__hash__` 的类型参数，因此我们可以使用 `typing.Hashable` 作为类型参数的上界。但是现在在 `typing` 或 `abc` 中没有适合的类型，因此我们需要创建它。

示例 8-20 展示了新的 `SupportsLessThan` 类型，一个 `Protocol`。

##### 示例 8-20\. *comparable.py*: `SupportsLessThan` `Protocol` 类型的定义

```py
from typing import Protocol, Any

class SupportsLessThan(Protocol):  # ①
    def __lt__(self, other: Any) -> bool: ...  # ②
```

①

协议是 `typing.Protocol` 的子类。

②

协议的主体有一个或多个方法定义，方法体中有 `...`。

如果类型 `T` 实现了 `P` 中定义的所有方法，并且类型签名匹配，则类型 `T` 与协议 `P` *一致*。

有了 `SupportsLessThan`，我们现在可以在 示例 8-21 中定义这个可工作的 `top` 版本。

##### 示例 8-21\. *top.py*: 使用 `TypeVar` 和 `bound=SupportsLessThan` 定义 `top` 函数

```py
from collections.abc import Iterable
from typing import TypeVar

from comparable import SupportsLessThan

LT = TypeVar('LT', bound=SupportsLessThan)

def top(series: Iterable[LT], length: int) -> list[LT]:
    ordered = sorted(series, reverse=True)
    return ordered[:length]
```

让我们来测试 `top`。示例 8-22 展示了一部分用于 `pytest` 的测试套件。首先尝试使用生成器表达式调用 `top`，该表达式生成 `tuple[int, str]`，然后使用 `object` 列表。对于 `object` 列表，我们期望得到一个 `TypeError` 异常。

##### 示例 8-22\. *top_test.py*: `top` 测试套件的部分清单

```py
from collections.abc import Iterator
from typing import TYPE_CHECKING  # ①

import pytest

from top import top

# several lines omitted

def test_top_tuples() -> None:
    fruit = 'mango pear apple kiwi banana'.split()
    series: Iterator[tuple[int, str]] = (  # ②
        (len(s), s) for s in fruit)
    length = 3
    expected = [(6, 'banana'), (5, 'mango'), (5, 'apple')]
    result = top(series, length)
    if TYPE_CHECKING:  # ③
        reveal_type(series)  # ④
        reveal_type(expected)
        reveal_type(result)
    assert result == expected

# intentional type error
def test_top_objects_error() -> None:
    series = [object() for _ in range(4)]
    if TYPE_CHECKING:
        reveal_type(series)
    with pytest.raises(TypeError) as excinfo:
        top(series, 3)  # ⑤
    assert "'<' not supported" in str(excinfo.value)
```

①

`typing.TYPE_CHECKING` 常量在运行时始终为 `False`，但类型检查器在进行类型检查时会假装它为 `True`。

②

显式声明 `series` 变量的类型，以使 Mypy 输出更易读。¹⁶

③

这个 `if` 阻止了接下来的三行在测试运行时执行。

④

`reveal_type()` 不能在运行时调用，因为它不是常规函数，而是 Mypy 的调试工具—这就是为什么没有为它导入任何内容。对于每个 `reveal_type()` 伪函数调用，Mypy 将输出一条调试消息，显示参数的推断类型。

⑤

这一行将被 Mypy 标记为错误。

前面的测试通过了—但无论是否在 *top.py* 中有类型提示，它们都会通过。更重要的是，如果我用 Mypy 检查该测试文件，我会看到 `TypeVar` 正如预期的那样工作。查看 示例 8-23 中的 `mypy` 命令输出。

###### 警告

截至 Mypy 0.910（2021 年 7 月），`reveal_type` 的输出在某些情况下并不精确显示我声明的类型，而是显示兼容的类型。例如，我没有使用 `typing.Iterator`，而是使用了 `abc.Iterator`。请忽略这个细节。Mypy 的输出仍然有用。在讨论输出时，我会假装 Mypy 的这个问题已经解决。

##### 示例 8-23\. *mypy top_test.py* 的输出（为了可读性而拆分的行）

```py
…/comparable/ $ mypy top_test.py
top_test.py:32: note:
    Revealed type is "typing.Iterator[Tuple[builtins.int, builtins.str]]" # ①
top_test.py:33: note:
    Revealed type is "builtins.list[Tuple[builtins.int, builtins.str]]"
top_test.py:34: note:
    Revealed type is "builtins.list[Tuple[builtins.int, builtins.str]]" # ②
top_test.py:41: note:
    Revealed type is "builtins.list[builtins.object*]" # ③
top_test.py:43: error:
    Value of type variable "LT" of "top" cannot be "object"  # ④
Found 1 error in 1 file (checked 1 source file)
```

①

在 `test_top_tuples` 中，`reveal_type(series)` 显示它是一个 `Iterator[tuple[int, str]]`—这是我明确声明的。

②

`reveal_type(result)` 确认了 `top` 调用返回的类型是我想要的：给定 `series` 的类型，`result` 是 `list[tuple[int, str]]`。

③

在 `test_top_objects_error` 中，`reveal_type(series)` 显示为 `list[object*]`。Mypy 在任何推断的类型后面加上 `*`：我没有在这个测试中注释 `series` 的类型。

④

Mypy 标记了这个测试故意触发的错误：`Iterable` `series`的元素类型不能是`object`（必须是`SupportsLessThan`类型）。

协议类型相对于 ABCs 的一个关键优势是，一个类型不需要任何特殊声明来*与协议类型一致*。这允许创建一个协议利用预先存在的类型，或者在我们无法控制的代码中实现的类型。我不需要派生或注册`str`、`tuple`、`float`、`set`等类型到`SupportsLessThan`以在期望`SupportsLessThan`参数的地方使用它们。它们只需要实现`__lt__`。而类型检查器仍然能够完成其工作，因为`SupportsLessThan`被明确定义为`Protocol`—与鸭子类型常见的隐式协议相反，这些协议对类型检查器是不可见的。

特殊的`Protocol`类在[PEP 544—Protocols: Structural subtyping (static duck typing)](https://fpy.li/pep544)中引入。示例 8-21 展示了为什么这个特性被称为*静态鸭子类型*：注释`top`的`series`参数的解决方案是说“`series`的名义类型并不重要，只要它实现了`__lt__`方法。”Python 的鸭子类型总是允许我们隐式地说这一点，让静态类型检查器一头雾水。类型检查器无法阅读 CPython 的 C 源代码，或者执行控制台实验来发现`sorted`只需要元素支持`<`。

现在我们可以为静态类型检查器明确地定义鸭子类型。这就是为什么说`typing.Protocol`给我们*静态鸭子类型*是有意义的。¹⁷

还有更多关于`typing.Protocol`的内容。我们将在第四部分回来讨论它，在第十三章中对比结构化类型、鸭子类型和 ABCs——另一种形式化协议的方法。此外，“重载签名”（第十五章）解释了如何使用`@typing.overload`声明重载函数签名，并包括了一个使用`typing.Protocol`和有界`TypeVar`的广泛示例。

###### 注意

`typing.Protocol`使得可以注释“类型由支持的操作定义”中提到的`double`函数而不会失去功能。关键是定义一个带有`__mul__`方法的协议类。我邀请你将其作为练习完成。解决方案出现在“类型化的 double 函数”中（第十三章）。

## Callable

为了注释回调参数或由高阶函数返回的可调用对象，`collections.abc`模块提供了`Callable`类型，在尚未使用 Python 3.9 的情况下在`typing`模块中可用。`Callable`类型的参数化如下：

```py
Callable[[ParamType1, ParamType2], ReturnType]
```

参数列表—`[ParamType1, ParamType2]`—可以有零个或多个类型。

这是在我们将在“lis.py 中的模式匹配：案例研究”中看到的一个`repl`函数的示例：¹⁸

```py
def repl(input_fn: Callable[[Any], str] = input]) -> None:
```

在正常使用中，`repl`函数使用 Python 的`input`内置函数从用户那里读取表达式。然而，对于自动化测试或与其他输入源集成，`repl`接受一个可选的`input_fn`参数：一个与`input`具有相同参数和返回类型的`Callable`。

内置的`input`在 typeshed 上有这个签名：

```py
def input(__prompt: Any = ...) -> str: ...
```

`input`的签名与这个`Callable`类型提示*一致*：

```py
Callable[[Any], str]
```

没有语法来注释可选或关键字参数类型。`typing.Callable`的[文档](https://fpy.li/8-34)说“这样的函数类型很少用作回调类型。”如果你需要一个类型提示来匹配具有灵活签名的函数，用`...`替换整个参数列表—就像这样：

```py
Callable[..., ReturnType]
```

泛型类型参数与类型层次结构的交互引入了一个新的类型概念：variance。

### Callable 类型中的 variance

想象一个简单的温度控制系统，其中有一个简单的`update`函数，如示例 8-24 所示。`update`函数调用`probe`函数获取当前温度，并调用`display`显示温度给用户。`probe`和`display`都作为参数传递给`update`是为了教学目的。示例的目标是对比两个`Callable`注释：一个有返回类型，另一个有参数类型。

##### 示例 8-24。说明 variance。

```py
from collections.abc import Callable

def update(  # ①
        probe: Callable[[], float],  # ②
        display: Callable[[float], None]  # ③
    ) -> None:
    temperature = probe()
    # imagine lots of control code here
    display(temperature)

def probe_ok() -> int:  # ④
    return 42

def display_wrong(temperature: int) -> None:  # ⑤
    print(hex(temperature))

update(probe_ok, display_wrong)  # type error # ⑥

def display_ok(temperature: complex) -> None:  # ⑦
    print(temperature)

update(probe_ok, display_ok)  # OK # ⑧
```

①

`update`接受两个可调用对象作为参数。

②

`probe`必须是一个不带参数并返回`float`的可调用对象。

③

`display`接受一个`float`参数并返回`None`。

④

`probe_ok`与`Callable[[], float]`一致，因为返回一个`int`不会破坏期望`float`的代码。

⑤

`display_wrong`与`Callable[[float], None]`不一致，因为没有保证一个期望`int`的函数能处理一个`float`；例如，Python 的`hex`函数接受一个`int`但拒绝一个`float`。

⑥

Mypy 标记这行是因为`display_wrong`与`update`的`display`参数中的类型提示不兼容。

⑦

`display_ok`与`Callable[[float], None]`一致，因为一个接受`complex`的函数也可以处理一个`float`参数。

⑧

Mypy 对这行很满意。

总结一下，当代码期望返回`float`的回调时，提供返回`int`的回调是可以的，因为`int`值总是可以在需要`float`的地方使用。

正式地说，`Callable[[], int]`是*subtype-of*`Callable[[], float]`——因为`int`是*subtype-of*`float`。这意味着`Callable`在返回类型上是*协变*的，因为类型`int`和`float`的*subtype-of*关系与使用它们作为返回类型的`Callable`类型的关系方向相同。

另一方面，当需要处理`float`时，提供一个接受`int`参数的回调是类型错误的。

正式地说，`Callable[[int], None]`不是*subtype-of*`Callable[[float], None]`。虽然`int`是*subtype-of*`float`，但在参数化的`Callable`类型中，关系是相反的：`Callable[[float], None]`是*subtype-of*`Callable[[int], None]`。因此我们说`Callable`在声明的参数类型上是*逆变*的。

“Variance”在第十五章中详细解释了 variance，并提供了不变、协变和逆变类型的更多细节和示例。

###### 提示

目前，可以放心地说，大多数参数化的泛型类型是*invariant*，因此更简单。例如，如果我声明`scores: list[float]`，那告诉我可以分配给`scores`的对象。我不能分配声明为`list[int]`或`list[complex]`的对象：

+   一个`list[int]`对象是不可接受的，因为它不能容纳`float`值，而我的代码可能需要将其放入`scores`中。

+   一个`list[complex]`对象是不可接受的，因为我的代码可能需要对`scores`进行排序以找到中位数，但`complex`没有提供`__lt__`，因此`list[complex]`是不可排序的。

现在我们来讨论本章中最后一个特殊类型。

## NoReturn

这是一种特殊类型，仅用于注释永远不返回的函数的返回类型。通常，它们存在是为了引发异常。标准库中有数十个这样的函数。

例如，`sys.exit()`引发`SystemExit`来终止 Python 进程。

它在`typeshed`中的签名是：

```py
def exit(__status: object = ...) -> NoReturn: ...
```

`__status`参数是仅位置参数，并且具有默认值。存根文件不详细说明默认值，而是使用`...`。`__status`的类型是`object`，这意味着它也可能是`None`，因此标记为`Optional[object]`将是多多的。

在第二十四章中，示例 24-6 在`__flag_unknown_attrs`中使用`NoReturn`，这是一个旨在生成用户友好和全面错误消息的方法，然后引发`AttributeError`。

这一史诗般章节的最后一节是关于位置和可变参数。

# 注释位置参数和可变参数

回想一下从示例 7-9 中的`tag`函数。我们上次看到它的签名是在“仅位置参数”中：

```py
def tag(name, /, *content, class_=None, **attrs):
```

这里是`tag`，完全注释，写成几行——长签名的常见约定，使用换行符的方式，就像[*蓝色*](https://fpy.li/8-10)格式化程序会做的那样：

```py
from typing import Optional

def tag(
    name: str,
    /,
    *content: str,
    class_: Optional[str] = None,
    **attrs: str,
) -> str:
```

注意对于任意位置参数的类型提示`*content: str`；这意味着所有这些参数必须是`str`类型。函数体中`content`的类型将是`tuple[str, ...]`。

在这个例子中，任意关键字参数的类型提示是`**attrs: str`，因此函数内部的`attrs`类型将是`dict[str, str]`。对于像`**attrs: float`这样的类型提示，函数内部的`attrs`类型将是`dict[str, float]`。

如果`attrs`参数必须接受不同类型的值，你需要使用`Union[]`或`Any`：`**attrs: Any`。

仅位置参数的`/`符号仅适用于 Python ≥ 3.8。在 Python 3.7 或更早版本中，这将是语法错误。[PEP 484 约定](https://fpy.li/8-36)是在每个位置参数名称前加上两个下划线。这里是`tag`签名，再次以两行的形式，使用 PEP 484 约定：

```py
from typing import Optional

def tag(__name: str, *content: str, class_: Optional[str] = None,
        **attrs: str) -> str:
```

Mypy 理解并强制执行声明位置参数的两种方式。

为了结束这一章，让我们简要地考虑一下类型提示的限制以及它们支持的静态类型系统。

# 不完美的类型和强大的测试

大型公司代码库的维护者报告说，许多错误是由静态类型检查器发现的，并且比在代码运行在生产环境后才发现这些错误更便宜修复。然而，值得注意的是，在我所知道的公司中，自动化测试在静态类型引入之前就是标准做法并被广泛采用。

即使在它们最有益处的情况下，静态类型也不能被信任为正确性的最终仲裁者。很容易找到：

假阳性

工具会报告代码中正确的类型错误。

假阴性

工具不会报告代码中不正确的类型错误。

此外，如果我们被迫对所有内容进行类型检查，我们将失去 Python 的一些表现力：

+   一些方便的功能无法进行静态检查；例如，像`config(**settings)`这样的参数解包。

+   属性、描述符、元类和一般元编程等高级功能对类型检查器的支持较差或超出理解范围。

+   类型检查器落后于 Python 版本，拒绝甚至在分析具有新语言特性的代码时崩溃——在某些情况下超过一年。

通常的数据约束无法在类型系统中表达，甚至是简单的约束。例如，类型提示无法确保“数量必须是大于 0 的整数”或“标签必须是具有 6 到 12 个 ASCII 字母的字符串”。总的来说，类型提示对捕捉业务逻辑中的错误并不有帮助。

鉴于这些注意事项，类型提示不能成为软件质量的主要支柱，强制性地使其成为例外会放大缺点。

将静态类型检查器视为现代 CI 流水线中的工具之一，与测试运行器、代码检查器等一起。CI 流水线的目的是减少软件故障，自动化测试可以捕获许多超出类型提示范围的错误。你可以在 Python 中编写的任何代码，都可以在 Python 中进行测试，无论是否有类型提示。

###### 注

本节的标题和结论受到 Bruce Eckel 的文章[“强类型 vs. 强测试”](https://fpy.li/8-37)的启发，该文章也发表在 Joel Spolsky（Apress）编辑的文集[*The Best Software Writing I*](https://fpy.li/8-38)中。Bruce 是 Python 的粉丝，也是关于 C++、Java、Scala 和 Kotlin 的书籍的作者。在那篇文章中，他讲述了他是如何成为静态类型支持者的，直到学习 Python 并得出结论：“如果一个 Python 程序有足够的单元测试，它可以和有足够单元测试的 C++、Java 或 C#程序一样健壮（尽管 Python 中的测试编写速度更快）。”

目前我们的 Python 类型提示覆盖到这里。它们也是第十五章的主要内容，该章涵盖了泛型类、变异、重载签名、类型转换等。与此同时，类型提示将在本书的几个示例中做客串出现。

# 章节总结

我们从对渐进式类型概念的简要介绍开始，然后转向实践方法。没有一个实际读取类型提示的工具，很难看出渐进式类型是如何工作的，因此我们开发了一个由 Mypy 错误报告引导的带注解函数。

回到渐进式类型的概念，我们探讨了它是 Python 传统鸭子类型和用户更熟悉的 Java、C++等静态类型语言的名义类型的混合体。

大部分章节都致力于介绍注解中使用的主要类型组。我们涵盖的许多类型与熟悉的 Python 对象类型相关，如集合、元组和可调用对象，扩展以支持类似`Sequence[float]`的泛型表示。许多这些类型是在 Python 3.9 之前在`typing`模块中实现的临时替代品，直到标准类型被更改以支持泛型。

一些类型是特殊实体。`Any`、`Optional`、`Union`和`NoReturn`与内存中的实际对象无关，而仅存在于类型系统的抽象领域中。

我们研究了参数化泛型和类型变量，这为类型提示带来了更多灵活性，而不会牺牲类型安全性。

使用`Protocol`使参数化泛型变得更加表达丰富。因为它仅出现在 Python 3.8 中，`Protocol`目前并不广泛使用，但它非常重要。`Protocol`实现了静态鸭子类型：Python 鸭子类型核心与名义类型之间的重要桥梁，使静态类型检查器能够捕捉错误。

在介绍一些类型的同时，我们通过 Mypy 进行实验，以查看类型检查错误，并借助 Mypy 的神奇`reveal_type()`函数推断类型。

最后一节介绍了如何注释位置参数和可变参数。

类型提示是一个复杂且不断发展的主题。幸运的是，它们是一个可选功能。让我们保持 Python 对最广泛用户群体的可访问性，并停止宣扬所有 Python 代码都应该有类型提示的说法，就像我在类型提示布道者的公开布道中看到的那样。

我们的退休 BDFL¹⁹领导了 Python 中类型提示的推动，因此这一章的开头和结尾都以他的话语开始：

> 我不希望有一个我在任何时候都有道义义务添加类型提示的 Python 版本。我真的认为类型提示有它们的位置，但也有很多时候不值得，而且很棒的是你可以选择使用它们。²⁰
> 
> Guido van Rossum

# 进一步阅读

Bernát Gábor 在他的优秀文章中写道，[“Python 中类型提示的现状”](https://fpy.li/8-41)：

> 只要值得编写单元测试，就应该使用类型提示。

我是测试的忠实粉丝，但我也做很多探索性编码。当我在探索时，测试和类型提示并不有用。它们只是累赘。

Gábor 的文章是我发现的关于 Python 类型提示的最好介绍之一，还有 Geir Arne Hjelle 的[“Python 类型检查（指南）”](https://fpy.li/8-42)。Claudio Jolowicz 的[“超现代 Python 第四章：类型”](https://fpy.li/8-43)是一个更简短的介绍，也涵盖了运行时类型检查验证。

想要更深入的了解，[Mypy 文档](https://fpy.li/8-44)是最佳来源。它对于任何类型检查器都很有价值，因为它包含了关于 Python 类型提示的教程和参考页面，不仅仅是关于 Mypy 工具本身。在那里你还会找到一份方便的[速查表](https://fpy.li/8-45)和一个非常有用的页面，介绍了[常见问题和解决方案](https://fpy.li/8-46)。

[`typing`](https://fpy.li/typing)模块文档是一个很好的快速参考，但它并没有详细介绍。[PEP 483—类型提示理论](https://fpy.li/pep483)包括了关于协变性的深入解释，使用`Callable`来说明逆变性。最终的参考资料是与类型提示相关的 PEP 文档。已经有 20 多个了。PEP 的目标受众是 Python 核心开发人员和 Python 的指导委员会，因此它们假定读者具有大量先前知识，绝对不是轻松阅读。

如前所述，第十五章涵盖了更多类型相关主题，而“进一步阅读”提供了额外的参考资料，包括表 15-1，列出了截至 2021 年底已批准或正在讨论的类型 PEPs。

[“了不起的 Python 类型提示”](https://fpy.li/8-47)是一个有价值的链接集合，包含了工具和参考资料。

¹ [PEP 484—类型提示](https://fpy.li/8-1)，“基本原理和目标”；粗体强调保留自原文。

² PyPy 中的即时编译器比类型提示有更好的数据：它在 Python 程序运行时监视程序，检测使用的具体类型，并为这些具体类型生成优化的机器代码。

³ 例如，截至 2021 年 7 月，不支持递归类型—参见`typing`模块问题[#182，定义 JSON 类型](https://fpy.li/8-2)和 Mypy 问题[#731，支持递归类型](https://fpy.li/8-3)。

⁴ Python 没有提供控制类型可能值集合的语法—除了在`Enum`类型中。例如，使用类型提示，你无法将`Quantity`定义为介于 1 和 1000 之间的整数，或将`AirportCode`定义为 3 个字母的组合。NumPy 提供了`uint8`、`int16`和其他面向机器的数值类型，但在 Python 标准库中，我们只有具有非常小值集合（`NoneType`、`bool`）或极大值集合（`float`、`int`、`str`、所有可能的元组等）的类型。

⁵ 鸭子类型是一种隐式的*结构类型*形式，Python ≥ 3.8 也支持引入`typing.Protocol`。这将在本章后面—“静态协议”—进行介绍，更多细节请参见第十三章。

⁶ 继承经常被滥用，并且很难在现实但简单的示例中证明其合理性，因此请接受这个动物示例作为子类型的快速说明。

⁷ 麻省理工学院教授、编程语言设计师和图灵奖获得者。维基百科：[芭芭拉·利斯科夫](https://fpy.li/8-14)。

⁸ 更准确地说，`ord`仅接受`len(s) == 1`的`str`或`bytes`。但目前的类型系统无法表达这个约束。

⁹ 在 ABC 语言——最初影响 Python 设计的语言中——每个列表都受限于接受单一类型的值：您放入其中的第一个项目的类型。

¹⁰ 我对`typing`模块文档的贡献之一是在 Guido van Rossum 的监督下将[“模块内容”](https://fpy.li/8-17)下的条目重新组织为子部分，并添加了数十个弃用警告。

¹¹ 在一些示例中，我使用`:=`是有意义的，但我在书中没有涵盖它。请参阅[PEP 572—赋值表达式](https://fpy.li/pep572)获取所有详细信息。

¹² 实际上，`dict`是`abc.MutableMapping`的虚拟子类。虚拟子类的概念在第十三章中有解释。暂时知道`issubclass(dict, abc.MutableMapping)`为`True`，尽管`dict`是用 C 实现的，不继承任何东西自`abc.MutableMapping`，而只继承自`object`。

¹³ 这里的实现比 Python 标准库中的[`statistics`](https://fpy.li/8-29)模块更简单。

¹⁴ 我向`typeshed`贡献了这个解决方案，这就是为什么`mode`在[*statistics.pyi*](https://fpy.li/8-32)中的注释截至 2020 年 5 月 26 日。

¹⁵ 多么美妙啊，打开一个交互式控制台并依靠鸭子类型来探索语言特性，就像我刚才做的那样。当我使用不支持它的语言时，我非常想念这种探索方式。

¹⁶ 没有这个类型提示，Mypy 会将`series`的类型推断为`Generator[Tuple[builtins.int, builtins.str*], None, None]`，这是冗长的但与`Iterator[tuple[int, str]]`一致，正如我们将在“通用可迭代类型”中看到的。

¹⁷ 我不知道谁发明了术语*静态鸭子类型*，但它在 Go 语言中变得更加流行，该语言的接口语义更像 Python 的协议，而不是 Java 的名义接口。

¹⁸ REPL 代表 Read-Eval-Print-Loop，交互式解释器的基本行为。

¹⁹ “终身仁慈独裁者”。参见 Guido van Rossum 关于[“BDFL 起源”](https://fpy.li/bdfl)。

²⁰ 来自 YouTube 视频，[“Guido van Rossum 关于类型提示（2015 年 3 月）”](https://fpy.li/8-39)。引用开始于[13’40”](https://fpy.li/8-40)。我进行了一些轻微的编辑以提高清晰度。

²¹ 来源：[“与艾伦·凯的对话”](https://fpy.li/8-54)。
