# 第十一章：一个 Python 风格的对象

> 使库或框架成为 Pythonic 是为了让 Python 程序员尽可能轻松和自然地学会如何执行任务。
> 
> Python 和 JavaScript 框架的创造者 Martijn Faassen。¹

由于 Python 数据模型，您定义的类型可以像内置类型一样自然地行为。而且这可以在不继承的情况下实现，符合*鸭子类型*的精神：你只需实现对象所需的方法，使其行为符合预期。

在之前的章节中，我们研究了许多内置对象的行为。现在我们将构建行为像真正的 Python 对象一样的用户定义类。你的应用程序类可能不需要并且不应该实现本章示例中那么多特殊方法。但是如果你正在编写一个库或框架，那么将使用你的类的程序员可能希望它们的行为像 Python 提供的类一样。满足这种期望是成为“Pythonic”的一种方式。

本章从第一章结束的地方开始，展示了如何实现在许多不同类型的 Python 对象中经常看到的几个特殊方法。

在本章中，我们将看到如何：

+   支持将对象转换为其他类型的内置函数（例如`repr()`、`bytes()`、`complex()`等）

+   实现一个作为类方法的替代构造函数

+   扩展 f-strings、`format()`内置函数和`str.format()`方法使用的格式迷你语言

+   提供对属性的只读访问

+   使对象可哈希以在集合中使用和作为`dict`键

+   使用`__slots__`节省内存

当我们开发`Vector2d`时，我们将做所有这些工作，这是一个简单的二维欧几里德向量类型。这段代码将是第十二章中 N 维向量类的基础。

示例的演变将暂停讨论两个概念性主题：

+   如何以及何时使用`@classmethod`和`@staticmethod`装饰器

+   Python 中的私有和受保护属性：用法、约定和限制

# 本章的新内容

我在本章的第二段中添加了一个新的引语和一些文字，以解释“Pythonic”的概念——这在第一版中只在最后讨论过。

“格式化显示”已更新以提及在 Python 3.6 中引入的 f-strings。这是一个小改变，因为 f-strings 支持与`format()`内置和`str.format()`方法相同的格式迷你语言，因此以前实现的`__format__`方法可以与 f-strings 一起使用。

本章的其余部分几乎没有变化——自 Python 3.0 以来，特殊方法大部分相同，核心思想出现在 Python 2.2 中。

让我们开始使用对象表示方法。

# 对象表示

每种面向对象的语言至少有一种标准方法可以从任何对象获取字符串表示。Python 有两种：

`repr()`

返回一个表示开发者想要看到的对象的字符串。当 Python 控制台或调试器显示一个对象时，你会得到这个。

`str()`

返回一个表示用户想要看到的对象的字符串。当你`print()`一个对象时，你会得到这个。

特殊方法`__repr__`和`__str__`支持`repr()`和`str()`，正如我们在第一章中看到的。

有两个额外的特殊方法支持对象的替代表示：`__bytes__`和`__format__`。`__bytes__`方法类似于`__str__`：它被`bytes()`调用以获取对象表示为字节序列。关于`__format__`，它被 f-strings、内置函数`format()`和`str.format()`方法使用。它们调用`obj.__format__(format_spec)`以获取使用特殊格式代码的对象的字符串显示。我们将在下一个示例中介绍`__bytes__`，然后介绍`__format__`。

###### 警告

如果您从 Python 2 转换而来，请记住，在 Python 3 中，`__repr__`，`__str__` 和 `__format__` 必须始终返回 Unicode 字符串（类型 `str`）。 只有 `__bytes__` 应该返回字节序列（类型 `bytes`）。

# 向量类 Redux

为了演示生成对象表示所使用的许多方法，我们将使用类似于我们在第一章中看到的 `Vector2d` 类。 我们将在本节和未来的章节中继续完善它。 示例 11-1 说明了我们从 `Vector2d` 实例中期望的基本行为。

##### 示例 11-1。 `Vector2d` 实例有几种表示形式

```py
    >>> v1 = Vector2d(3, 4)
    >>> print(v1.x, v1.y)  # ①
    3.0 4.0
    >>> x, y = v1  # ②
    >>> x, y
    (3.0, 4.0)
    >>> v1  # ③
    Vector2d(3.0, 4.0)
    >>> v1_clone = eval(repr(v1))  # ④
    >>> v1 == v1_clone  # ⑤
    True
    >>> print(v1)  # ⑥
    (3.0, 4.0)
    >>> octets = bytes(v1)  # ⑦
    >>> octets
    b'd\\x00\\x00\\x00\\x00\\x00\\x00\\x08@\\x00\\x00\\x00\\x00\\x00\\x00\\x10@'
    >>> abs(v1)  # ⑧
    5.0
    >>> bool(v1), bool(Vector2d(0, 0))  # ⑨
    (True, False)
```

①

`Vector2d` 的组件可以直接作为属性访问（无需 getter 方法调用）。

②

`Vector2d` 可以解包为一组变量的元组。

③

`Vector2d` 的 `repr` 模拟了构造实例的源代码。

④

在这里使用 `eval` 显示 `Vector2d` 的 `repr` 是其构造函数调用的忠实表示。²

⑤

`Vector2d` 支持与 `==` 的比较；这对于测试很有用。

⑥

`print` 调用 `str`，对于 `Vector2d` 会产生一个有序对显示。

⑦

`bytes` 使用 `__bytes__` 方法生成二进制表示。

⑧

`abs` 使用 `__abs__` 方法返回 `Vector2d` 的大小。

⑨

`bool` 使用 `__bool__` 方法，对于零大小的 `Vector2d` 返回 `False`，否则返回 `True`。

`Vector2d` 来自示例 11-1，在 *vector2d_v0.py* 中实现（示例 11-2）。 该代码基于示例 1-2，除了 `+` 和 `*` 操作的方法，我们稍后会看到在第十六章中。 我们将添加 `==` 方法，因为它对于测试很有用。 到目前为止，`Vector2d` 使用了几个特殊方法来提供 Pythonista 在设计良好的对象中期望的操作。

##### 示例 11-2。 vector2d_v0.py：到目前为止，所有方法都是特殊方法

```py
from array import array
import math

class Vector2d:
    typecode = 'd'  # ①

    def __init__(self, x, y):
        self.x = float(x)    # ②
        self.y = float(y)

    def __iter__(self):
        return (i for i in (self.x, self.y))  # ③

    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)  # ④

    def __str__(self):
        return str(tuple(self))  # ⑤

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +  # ⑥
                bytes(array(self.typecode, self)))  # ⑦

    def __eq__(self, other):
        return tuple(self) == tuple(other)  # ⑧

    def __abs__(self):
        return math.hypot(self.x, self.y)  # ⑨

    def __bool__(self):
        return bool(abs(self))  # ⑩
```

①

`typecode` 是我们在将 `Vector2d` 实例转换为/从 `bytes` 时将使用的类属性。

②

在 `__init__` 中将 `x` 和 `y` 转换为 `float` 可以及早捕获错误，这在 `Vector2d` 被使用不合适的参数调用时很有帮助。

③

`__iter__` 使 `Vector2d` 可迭代；这就是解包工作的原因（例如，`x, y = my_vector`）。 我们简单地通过使用生成器表达式逐个产生组件来实现它。³

④

`__repr__` 通过使用 `{!r}` 插值组件来构建字符串；因为 `Vector2d` 是可迭代的，`*self` 将 `x` 和 `y` 组件提供给 `format`。

⑤

从可迭代的 `Vector2d` 中，很容易构建一个用于显示有序对的 `tuple`。

⑥

要生成 `bytes`，我们将类型码转换为 `bytes` 并连接...

⑦

...通过迭代实例构建的 `array` 转换为的 `bytes`。

⑧

要快速比较所有组件，将操作数构建为元组。 这适用于 `Vector2d` 的实例，但存在问题。 请参阅以下警告。

⑨

大小是由`x`和`y`分量形成的直角三角形的斜边的长度。

⑩

`__bool__`使用`abs(self)`来计算大小，然后将其转换为`bool`，因此`0.0`变为`False`，非零为`True`。

###### 警告

示例 11-2 中的`__eq__`方法适用于`Vector2d`操作数，但当将`Vector2d`实例与持有相同数值的其他可迭代对象进行比较时也返回`True`（例如，`Vector(3, 4) == [3, 4]`）。这可能被视为一个特性或一个错误。进一步讨论需要等到第十六章，当我们讨论运算符重载时。

我们有一个相当完整的基本方法集，但我们仍然需要一种方法来从`bytes()`生成的二进制表示中重建`Vector2d`。

# 另一种构造方法

由于我们可以将`Vector2d`导出为字节，自然我们需要一个从二进制序列导入`Vector2d`的方法。在标准库中寻找灵感时，我们发现`array.array`有一个名为`.frombytes`的类方法，非常适合我们的目的——我们在“数组”中看到了它。我们采用其名称，并在*vector2d_v1.py*中的`Vector2d`类方法中使用其功能（示例 11-3）。

##### 示例 11-3\. vector2d_v1.py 的一部分：此片段仅显示了`frombytes`类方法，添加到 vector2d_v0.py 中的`Vector2d`定义中（示例 11-2）

```py
    @classmethod  # ①
    def frombytes(cls, octets):  # ②
        typecode = chr(octets[0])  # ③
        memv = memoryview(octets[1:]).cast(typecode)  # ④
        return cls(*memv)  # ⑤
```

①

`classmethod`装饰器修改了一个方法，使其可以直接在类上调用。

②

没有`self`参数；相反，类本身作为第一个参数传递—按照惯例命名为`cls`。

③

从第一个字节读取`typecode`。

④

从`octets`二进制序列创建一个`memoryview`，并使用`typecode`进行转换。⁴

⑤

将从转换结果中得到的`memoryview`解包为构造函数所需的一对参数。

我刚刚使用了`classmethod`装饰器，它非常特定于 Python，所以让我们谈谈它。

# 类方法与静态方法

Python 教程中没有提到`classmethod`装饰器，也没有提到`staticmethod`。任何在 Java 中学习面向对象编程的人可能会想知道为什么 Python 有这两个装饰器而不是其中的一个。

让我们从`classmethod`开始。示例 11-3 展示了它的用法：定义一个在类上而不是在实例上操作的方法。`classmethod`改变了方法的调用方式，因此它接收类本身作为第一个参数，而不是一个实例。它最常见的用途是用于替代构造函数，就像示例 11-3 中的`frombytes`一样。请注意`frombytes`的最后一行实际上通过调用`cls`参数来使用`cls`参数以构建一个新实例：`cls(*memv)`。

相反，`staticmethod`装饰器改变了一个方法，使其不接收特殊的第一个参数。实质上，静态方法就像一个普通函数，只是它存在于类体中，而不是在模块级别定义。示例 11-4 对比了`classmethod`和`staticmethod`的操作。

##### 示例 11-4\. 比较`classmethod`和`staticmethod`的行为

```py
>>> class Demo:
...     @classmethod
...     def klassmeth(*args):
...         return args  # ①
...     @staticmethod
...     def statmeth(*args):
...         return args  # ②
...
>>> Demo.klassmeth()  # ③
(<class '__main__.Demo'>,) >>> Demo.klassmeth('spam')
(<class '__main__.Demo'>, 'spam') >>> Demo.statmeth()   # ④
() >>> Demo.statmeth('spam')
('spam',)
```

①

`klassmeth`只返回所有位置参数。

②

`statmeth`也是如此。

③

无论如何调用，`Demo.klassmeth`都将`Demo`类作为第一个参数接收。

④

`Demo.statmeth`的行为就像一个普通的旧函数。

###### 注意

`classmethod`装饰器显然很有用，但在我的经验中，`staticmethod`的好用例子非常少见。也许这个函数即使从不涉及类也与之密切相关，所以你可能希望将其放在代码附近。即使如此，在同一模块中在类的前面或后面定义函数大多数情况下已经足够接近了。⁵

现在我们已经看到了`classmethod`的用途（以及`staticmethod`并不是很有用），让我们回到对象表示的问题，并看看如何支持格式化输出。

# 格式化显示

f-strings、`format()`内置函数和`str.format()`方法通过调用它们的`.__format__(format_spec)`方法将实际格式化委托给每种类型。`format_spec`是一个格式说明符，它可以是：

+   `format(my_obj, format_spec)`中的第二个参数，或

+   无论在 f-string 中的用`{}`括起来的替换字段中的冒号后面的内容，还是在`fmt.str.format()`中的`fmt`中

例如：

```py
>>> brl = 1 / 4.82  # BRL to USD currency conversion rate
>>> brl
0.20746887966804978 >>> format(brl, '0.4f')  # ①
'0.2075' >>> '1 BRL = {rate:0.2f} USD'.format(rate=brl)  # ②
'1 BRL = 0.21 USD' >>> f'1 USD = {1 / brl:0.2f} BRL'  # ③
'1 USD = 4.82 BRL'
```

①

格式说明符是`'0.4f'`。

②

格式说明符是`'0.2f'`。替换字段中的`rate`部分不是格式说明符的一部分。它确定哪个关键字参数进入该替换字段。

③

再次，说明符是`'0.2f'`。`1 / brl`表达式不是其中的一部分。

第二个和第三个标注指出了一个重要的观点：例如`'{0.mass:5.3e}'`这样的格式字符串实际上使用了两种不同的表示法。冒号左边的`'0.mass'`是替换字段语法的`field_name`部分，它可以是 f-string 中的任意表达式。冒号后面的`'5.3e'`是格式说明符。格式说明符中使用的表示法称为[格式规范迷你语言](https://fpy.li/11-3)。

###### 提示

如果 f-strings、`format()`和`str.format()`对你来说是新的，课堂经验告诉我最好先学习`format()`内置函数，它只使用[格式规范迷你语言](https://fpy.li/fmtspec)。在你掌握了这个要领之后，阅读[“格式化字符串字面值”](https://fpy.li/11-4)和[“格式化字符串语法”](https://fpy.li/11-5)来了解在 f-strings 和`str.format()`方法中使用的`{:}`替换字段符号，包括`!s`、`!r`和`!a`转换标志。f-strings 并不使`str.format()`过时：大多数情况下 f-strings 解决了问题，但有时最好在其他地方指定格式化字符串，而不是在将要呈现的地方。

一些内置类型在格式规范迷你语言中有自己的表示代码。例如——在几个其他代码中——`int`类型支持分别用于输出基数 2 和基数 16 的`b`和`x`，而`float`实现了用于固定点显示的`f`和用于百分比显示的`%`：

```py
>>> format(42, 'b')
'101010'
>>> format(2 / 3, '.1%')
'66.7%'
```

格式规范迷你语言是可扩展的，因为每个类都可以根据自己的喜好解释`format_spec`参数。例如，`datetime`模块中的类使用`strftime()`函数和它们的`__format__`方法中的相同格式代码。以下是使用`format()`内置函数和`str.format()`方法的几个示例：

```py
>>> from datetime import datetime
>>> now = datetime.now()
>>> format(now, '%H:%M:%S')
'18:49:05'
>>> "It's now {:%I:%M %p}".format(now)
"It's now 06:49 PM"
```

如果一个类没有`__format__`，则从`object`继承的方法返回`str(my_object)`。因为`Vector2d`有一个`__str__`，所以这样可以：

```py
>>> v1 = Vector2d(3, 4)
>>> format(v1)
'(3.0, 4.0)'
```

然而，如果传递了格式说明符，`object.__format__`会引发`TypeError`：

```py
>>> format(v1, '.3f')
Traceback (most recent call last):
  ...
TypeError: non-empty format string passed to object.__format__
```

我们将通过实现自己的格式迷你语言来解决这个问题。第一步是假设用户提供的格式说明符是用于格式化向量的每个`float`组件。这是我们想要的结果：

```py
>>> v1 = Vector2d(3, 4)
>>> format(v1)
'(3.0, 4.0)'
>>> format(v1, '.2f')
'(3.00, 4.00)'
>>> format(v1, '.3e')
'(3.000e+00, 4.000e+00)'
```

示例 11-5 实现了`__format__`以产生刚才显示的内容。

##### 示例 11-5\. `Vector2d.__format__` 方法，第一部分

```py
    # inside the Vector2d class

    def __format__(self, fmt_spec=''):
        components = (format(c, fmt_spec) for c in self)  # ①
        return '({}, {})'.format(*components)  # ②
```

①

使用内置的`format`应用`fmt_spec`到每个向量组件，构建格式化字符串的可迭代对象。

②

将格式化字符串插入公式`'(x, y)'`中。

现在让我们向我们的迷你语言添加自定义格式代码：如果格式说明符以`'p'`结尾，我们将以极坐标形式显示向量：`<r, θ>`，其中`r`是幅度，θ（theta）是弧度角。格式说明符的其余部分（在`'p'`之前的任何内容）将像以前一样使用。

###### 提示

在选择自定义格式代码的字母时，我避免与其他类型使用的代码重叠。在[格式规范迷你语言](https://fpy.li/11-3)中，我们看到整数使用代码`'bcdoxXn'`，浮点数使用`'eEfFgGn%'`，字符串使用`'s'`。因此，我选择了`'p'`来表示极坐标。因为每个类都独立解释这些代码，所以在新类型的自定义格式中重用代码字母不是错误，但可能会让用户感到困惑。

要生成极坐标，我们已经有了用于幅度的`__abs__`方法，我们将使用`math.atan2()`函数编写一个简单的`angle`方法来获取角度。这是代码：

```py
    # inside the Vector2d class

    def angle(self):
        return math.atan2(self.y, self.x)
```

有了这个，我们可以增强我们的`__format__`以生成极坐标。参见示例 11-6。

##### 示例 11-6. `Vector2d.__format__` 方法，第二部分，现在包括极坐标

```py
    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('p'):  # ①
            fmt_spec = fmt_spec[:-1]  # ②
            coords = (abs(self), self.angle())  # ③
            outer_fmt = '<{}, {}>'  # ④
        else:
            coords = self  # ⑤
            outer_fmt = '({}, {})'  # ⑥
        components = (format(c, fmt_spec) for c in coords)  # ⑦
        return outer_fmt.format(*components)  # ⑧
```

①

格式以`'p'`结尾：使用极坐标。

②

从`fmt_spec`中删除`'p'`后缀。

③

构建极坐标的`tuple`：`(magnitude, angle)`。

④

用尖括号配置外部格式。

⑤

否则，使用`self`的`x, y`组件作为直角坐标。

⑥

用括号配置外部格式。

⑦

生成组件格式化字符串的可迭代对象。

⑧

将格式化字符串插入外部格式。

通过示例 11-6，我们得到类似于以下结果：

```py
>>> format(Vector2d(1, 1), 'p')
'<1.4142135623730951, 0.7853981633974483>'
>>> format(Vector2d(1, 1), '.3ep')
'<1.414e+00, 7.854e-01>'
>>> format(Vector2d(1, 1), '0.5fp')
'<1.41421, 0.78540>'
```

正如本节所示，扩展格式规范迷你语言以支持用户定义的类型并不困难。

现在让我们转向一个不仅仅关于外观的主题：我们将使我们的`Vector2d`可散列，这样我们就可以构建向量集，或者将它们用作`dict`键。

# 一个可散列的 Vector2d

截至目前，我们的`Vector2d`实例是不可散列的，因此我们无法将它们放入`set`中：

```py
>>> v1 = Vector2d(3, 4)
>>> hash(v1)
Traceback (most recent call last):
  ...
TypeError: unhashable type: 'Vector2d'
>>> set([v1])
Traceback (most recent call last):
  ...
TypeError: unhashable type: 'Vector2d'
```

要使`Vector2d`可散列，我们必须实现`__hash__`（`__eq__`也是必需的，我们已经有了）。我们还需要使向量实例不可变，正如我们在“什么是可散列”中看到的。

现在，任何人都可以执行`v1.x = 7`，而代码中没有任何提示表明更改`Vector2d`是被禁止的。这是我们想要的行为：

```py
>>> v1.x, v1.y
(3.0, 4.0)
>>> v1.x = 7
Traceback (most recent call last):
  ...
AttributeError: can't set attribute
```

我们将通过在示例 11-7 中使`x`和`y`组件成为只读属性来实现这一点。

##### 示例 11-7. vector2d_v3.py：仅显示使`Vector2d`成为不可变的更改；在示例 11-11 中查看完整清单

```py
class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)  # ①
        self.__y = float(y)

    @property  # ②
    def x(self):  # ③
        return self.__x  # ④

    @property  # ⑤
    def y(self):
        return self.__y

    def __iter__(self):
        return (i for i in (self.x, self.y))  # ⑥

    # remaining methods: same as previous Vector2d
```

①

使用正好两个前导下划线（零个或一个尾随下划线）使属性私有化。⁶

②

`@property`装饰器标记属性的 getter 方法。

③

getter 方法的名称与其公共属性相对应：`x`。

④

只需返回`self.__x`。

⑤

重复相同的公式用于`y`属性。

⑥

每个仅读取`x`、`y`分量的方法都可以保持原样，通过`self.x`和`self.y`读取公共属性而不是私有属性，因此此列表省略了类的其余代码。

###### 注意

`Vector.x`和`Vector.y`是只读属性的示例。读/写属性将在第二十二章中介绍，我们将深入探讨`@property`。

现在，我们的向量相对安全免受意外变异，我们可以实现`__hash__`方法。它应返回一个`int`，理想情况下应考虑在`__eq__`方法中也使用的对象属性的哈希值，因为比较相等的对象应具有相同的哈希值。`__hash__`特殊方法的[文档](https://fpy.li/11-7)建议计算一个包含组件的元组的哈希值，这就是我们在示例 11-8 中所做的。

##### 示例 11-8。vector2d_v3.py：*hash*的实现

```py
    # inside class Vector2d:

    def __hash__(self):
        return hash((self.x, self.y))
```

通过添加`__hash__`方法，我们现在有了可散列的向量：

```py
>>> v1 = Vector2d(3, 4)
>>> v2 = Vector2d(3.1, 4.2)
>>> hash(v1), hash(v2)
(1079245023883434373, 1994163070182233067)
>>> {v1, v2}
{Vector2d(3.1, 4.2), Vector2d(3.0, 4.0)}
```

###### 提示

实现属性或以其他方式保护实例属性以创建可散列类型并不是绝对必要的。正确实现`__hash__`和`__eq__`就足够了。但是，可散列对象的值永远不应更改，因此这提供了一个很好的借口来谈论只读属性。

如果您正在创建具有合理标量数值的类型，还可以实现`__int__`和`__float__`方法，这些方法由`int()`和`float()`构造函数调用，在某些情况下用于类型强制转换。还有一个`__complex__`方法来支持`complex()`内置构造函数。也许`Vector2d`应该提供`__complex__`，但我会把这留给你作为一个练习。

# 支持位置模式匹配

到目前为止，`Vector2d`实例与关键字类模式兼容——在“关键字类模式”中介绍。

在示例 11-9 中，所有这些关键字模式都按预期工作。

##### 示例 11-9。`Vector2d`主题的关键字模式——需要 Python 3.10

```py
def keyword_pattern_demo(v: Vector2d) -> None:
    match v:
        case Vector2d(x=0, y=0):
            print(f'{v!r} is null')
        case Vector2d(x=0):
            print(f'{v!r} is vertical')
        case Vector2d(y=0):
            print(f'{v!r} is horizontal')
        case Vector2d(x=x, y=y) if x==y:
            print(f'{v!r} is diagonal')
        case _:
            print(f'{v!r} is awesome')
```

但是，如果您尝试使用这样的位置模式：

```py
        case Vector2d(_, 0):
            print(f'{v!r} is horizontal')
```

你会得到：

```py
TypeError: Vector2d() accepts 0 positional sub-patterns (1 given)
```

要使`Vector2d`与位置模式配合使用，我们需要添加一个名为`__match_args__`的类属性，按照它们将用于位置模式匹配的顺序列出实例属性：

```py
class Vector2d:
    __match_args__ = ('x', 'y')

    # etc...
```

现在，当编写用于匹配`Vector2d`主题的模式时，我们可以节省一些按键，如您在示例 11-10 中所见。

##### 示例 11-10。`Vector2d`主题的位置模式——需要 Python 3.10

```py
def positional_pattern_demo(v: Vector2d) -> None:
    match v:
        case Vector2d(0, 0):
            print(f'{v!r} is null')
        case Vector2d(0):
            print(f'{v!r} is vertical')
        case Vector2d(_, 0):
            print(f'{v!r} is horizontal')
        case Vector2d(x, y) if x==y:
            print(f'{v!r} is diagonal')
        case _:
            print(f'{v!r} is awesome')
```

`__match_args__`类属性不需要包括所有公共实例属性。特别是，如果类`__init__`具有分配给实例属性的必需和可选参数，可能合理地在`__match_args__`中命名必需参数，但不包括可选参数。

让我们退后一步，回顾一下我们到目前为止在`Vector2d`中编码的内容。

# Vector2d 的完整列表，版本 3

我们已经在`Vector2d`上工作了一段时间，只展示了一些片段，因此示例 11-11 是*vector2d_v3.py*的综合完整列表，包括我在开发时使用的 doctests。

##### 示例 11-11。vector2d_v3.py：完整的版本

```py
"""
A two-dimensional vector class

 >>> v1 = Vector2d(3, 4)
 >>> print(v1.x, v1.y)
 3.0 4.0
 >>> x, y = v1
 >>> x, y
 (3.0, 4.0)
 >>> v1
 Vector2d(3.0, 4.0)
 >>> v1_clone = eval(repr(v1))
 >>> v1 == v1_clone
 True
 >>> print(v1)
 (3.0, 4.0)
 >>> octets = bytes(v1)
 >>> octets
 b'd\\x00\\x00\\x00\\x00\\x00\\x00\\x08@\\x00\\x00\\x00\\x00\\x00\\x00\\x10@'
 >>> abs(v1)
 5.0
 >>> bool(v1), bool(Vector2d(0, 0))
 (True, False)

Test of ``.frombytes()`` class method:

 >>> v1_clone = Vector2d.frombytes(bytes(v1))
 >>> v1_clone
 Vector2d(3.0, 4.0)
 >>> v1 == v1_clone
 True

Tests of ``format()`` with Cartesian coordinates:

 >>> format(v1)
 '(3.0, 4.0)'
 >>> format(v1, '.2f')
 '(3.00, 4.00)'
 >>> format(v1, '.3e')
 '(3.000e+00, 4.000e+00)'

Tests of the ``angle`` method::

 >>> Vector2d(0, 0).angle()
 0.0
 >>> Vector2d(1, 0).angle()
 0.0
 >>> epsilon = 10**-8
 >>> abs(Vector2d(0, 1).angle() - math.pi/2) < epsilon
 True
 >>> abs(Vector2d(1, 1).angle() - math.pi/4) < epsilon
 True

Tests of ``format()`` with polar coordinates:

 >>> format(Vector2d(1, 1), 'p')  # doctest:+ELLIPSIS
 '<1.414213..., 0.785398...>'
 >>> format(Vector2d(1, 1), '.3ep')
 '<1.414e+00, 7.854e-01>'
 >>> format(Vector2d(1, 1), '0.5fp')
 '<1.41421, 0.78540>'

Tests of `x` and `y` read-only properties:

 >>> v1.x, v1.y
 (3.0, 4.0)
 >>> v1.x = 123
 Traceback (most recent call last):
 ...
 AttributeError: can't set attribute 'x'

Tests of hashing:

 >>> v1 = Vector2d(3, 4)
 >>> v2 = Vector2d(3.1, 4.2)
 >>> len({v1, v2})
 2

"""

from array import array
import math

class Vector2d:
    __match_args__ = ('x', 'y')

    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)

    @property
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y

    def __iter__(self):
        return (i for i in (self.x, self.y))

    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) +
                bytes(array(self.typecode, self)))

    def __eq__(self, other):
        return tuple(self) == tuple(other)

    def __hash__(self):
        return hash((self.x, self.y))

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def angle(self):
        return math.atan2(self.y, self.x)

    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('p'):
            fmt_spec = fmt_spec[:-1]
            coords = (abs(self), self.angle())
            outer_fmt = '<{}, {}>'
        else:
            coords = self
            outer_fmt = '({}, {})'
        components = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(*components)

    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)
```

总结一下，在本节和前几节中，我们看到了一些您可能希望实现以拥有完整对象的基本特殊方法。

###### 注意

只有在您的应用程序需要时才实现这些特殊方法。最终用户不在乎构成应用程序的对象是否“Pythonic”。

另一方面，如果您的类是其他 Python 程序员使用的库的一部分，您实际上无法猜测他们将如何处理您的对象，他们可能期望我们正在描述的更多“Pythonic”行为。

如示例 11-11 中所编码的，`Vector2d`是一个关于对象表示相关特殊方法的教学示例，而不是每个用户定义类的模板。

在下一节中，我们将暂时离开`Vector2d`，讨论 Python 中私有属性机制的设计和缺点——`self.__x`中的双下划线前缀。

# Python 中的私有和“受保护”的属性

在 Python 中，没有像 Java 中的`private`修饰符那样创建私有变量的方法。在 Python 中，我们有一个简单的机制来防止在子类中意外覆盖“私有”属性。

考虑这种情况：有人编写了一个名为`Dog`的类，其中内部使用了一个`mood`实例属性，但没有暴露它。你需要将`Dog`作为`Beagle`的子类。如果你在不知道名称冲突的情况下创建自己的`mood`实例属性，那么你将覆盖从`Dog`继承的方法中使用的`mood`属性。这将是一个令人头疼的调试问题。

为了防止这种情况发生，如果你将一个实例属性命名为`__mood`（两个前导下划线和零个或最多一个尾随下划线），Python 会将该名称存储在实例`__dict__`中，前缀是一个前导下划线和类名，因此在`Dog`类中，`__mood`变成了`_Dog__mood`，而在`Beagle`中变成了`_Beagle__mood`。这种语言特性被称为*名称修饰*。

示例 11-12 展示了来自示例 11-7 中`Vector2d`类的结果。

##### 示例 11-12\. 私有属性名称通过前缀`_`和类名“修饰”

```py
>>> v1 = Vector2d(3, 4)
>>> v1.__dict__
{'_Vector2d__y': 4.0, '_Vector2d__x': 3.0}
>>> v1._Vector2d__x
3.0
```

名称修饰是关于安全性，而不是安全性：它旨在防止意外访问，而不是恶意窥探。图 11-1 展示了另一个安全设备。

知道私有名称是如何被修饰的人可以直接读取私有属性，就像示例 11-12 的最后一行所示的那样——这对调试和序列化实际上是有用的。他们还可以通过编写`v1._Vector2d__x = 7`来直接为`Vector2d`的私有组件赋值。但如果你在生产代码中这样做，如果出现问题，就不能抱怨了。

名称修饰功能并不受所有 Python 爱好者的喜爱，以及写作为`self.__x`的名称的倾斜外观也不受欢迎。一些人更喜欢避免这种语法，只使用一个下划线前缀通过约定“保护”属性（例如，`self._x`）。对于自动双下划线修饰的批评者，他们建议通过命名约定来解决意外属性覆盖的问题。Ian Bicking——pip、virtualenv 等项目的创建者写道：

> 永远不要使用两个前导下划线。这是非常私有的。如果担心名称冲突，可以使用显式的名称修饰（例如，`_MyThing_blahblah`）。这与双下划线基本相同，只是双下划线会隐藏，而显式名称修饰则是透明的。⁷

![带有安全盖的开关](img/flpy_1101.png)

###### 图 11-1\. 开关上的盖子是一个*安全*设备，而不是*安全*设备：它防止事故，而不是破坏。

单个下划线前缀在属性名称中对 Python 解释器没有特殊含义，但在 Python 程序员中是一个非常强烈的约定，你不应该从类外部访问这样的属性。⁸。尊重一个将其属性标记为单个下划线的对象的隐私是很容易的，就像尊重将`ALL_CAPS`中的变量视为常量的约定一样容易。

在 Python 文档的某些角落中，带有单个下划线前缀的属性被称为“受保护的”⁹。通过约定以`self._x`的形式“保护”属性的做法很普遍，但将其称为“受保护的”属性并不那么常见。有些人甚至将其称为“私有”属性。

总之：`Vector2d`的组件是“私有的”，我们的`Vector2d`实例是“不可变的”——带有引号——因为没有办法使它们真正私有和不可变。¹⁰

现在我们回到我们的`Vector2d`类。在下一节中，我们将介绍一个特殊的属性（不是方法），它会影响对象的内部存储，对内存使用可能有巨大影响，但对其公共接口影响很小：`__slots__`。

# 使用`__slots__`节省内存

默认情况下，Python 将每个实例的属性存储在名为`__dict__`的`dict`中。正如我们在“dict 工作原理的实际后果”中看到的，`dict`具有显着的内存开销——即使使用了该部分提到的优化。但是，如果你定义一个名为`__slots__`的类属性，其中包含一系列属性名称，Python 将使用替代的存储模型来存储实例属性：`__slots__`中命名的属性存储在一个隐藏的引用数组中，使用的内存比`dict`少。让我们通过简单的示例来看看它是如何工作的，从示例 11-13 开始。

##### 示例 11-13。`Pixel`类使用`__slots__`

```py
>>> class Pixel:
...     __slots__ = ('x', 'y')  # ①
...
>>> p = Pixel()  # ②
>>> p.__dict__  # ③
Traceback (most recent call last):
  ...
AttributeError: 'Pixel' object has no attribute '__dict__'
>>> p.x = 10  # ④
>>> p.y = 20
>>> p.color = 'red'  # ⑤
Traceback (most recent call last):
  ...
AttributeError: 'Pixel' object has no attribute 'color'
```

①

在创建类时必须存在`__slots__`；稍后添加或更改它没有效果。属性名称可以是`tuple`或`list`，但我更喜欢`tuple`，以明确表明没有改变的必要。

②

创建一个`Pixel`的实例，因为我们看到`__slots__`对实例的影响。

③

第一个效果：`Pixel`的实例没有`__dict__`。

④

正常设置`p.x`和`p.y`属性。

⑤

第二个效果：尝试设置一个未在`__slots__`中列出的属性会引发`AttributeError`。

到目前为止，一切顺利。现在让我们在示例 11-14 中创建`Pixel`的一个子类，看看`__slots__`的反直觉之处。

##### 示例 11-14。`OpenPixel`是`Pixel`的子类

```py
>>> class OpenPixel(Pixel):  # ①
...     pass
...
>>> op = OpenPixel()
>>> op.__dict__  # ②
{} >>> op.x = 8  # ③
>>> op.__dict__  # ④
{} >>> op.x  # ⑤
8 >>> op.color = 'green'  # ⑥
>>> op.__dict__  # ⑦
{'color': 'green'}
```

①

`OpenPixel`没有声明自己的属性。

②

惊喜：`OpenPixel`的实例有一个`__dict__`。

③

如果你设置属性`x`（在基类`Pixel`的`__slots__`中命名）…

④

…它不存储在实例`__dict__`中…

⑤

…但它存储在实例的隐藏引用数组中。

⑥

如果你设置一个未在`__slots__`中命名的属性…

⑦

…它存储在实例`__dict__`中。

示例 11-14 显示了`__slots__`的效果只被子类部分继承。为了确保子类的实例没有`__dict__`，你必须在子类中再次声明`__slots__`。

如果你声明`__slots__ = ()`（一个空元组），那么子类的实例将没有`__dict__`，并且只接受基类`__slots__`中命名的属性。

如果你希望子类具有额外的属性，请在`__slots__`中命名它们，就像示例 11-15 中所示的那样。

##### 示例 11-15。`ColorPixel`，`Pixel`的另一个子类

```py
>>> class ColorPixel(Pixel):
...    __slots__ = ('color',)  # ①
>>> cp = ColorPixel()
>>> cp.__dict__  # ②
Traceback (most recent call last):
  ...
AttributeError: 'ColorPixel' object has no attribute '__dict__'
>>> cp.x = 2
>>> cp.color = 'blue'  # ③
>>> cp.flavor = 'banana'
Traceback (most recent call last):
  ...
AttributeError: 'ColorPixel' object has no attribute 'flavor'
```

①

本质上，超类的`__slots__`被添加到当前类的`__slots__`中。不要忘记单项元组必须有一个尾随逗号。

②

`ColorPixel`实例没有`__dict__`。

③

你可以设置此类和超类的`__slots__`中声明的属性，但不能设置其他属性。

“既能节省内存又能使用它”是可能的：如果将`'__dict__'`名称添加到`__slots__`列表中，那么你的实例将保留`__slots__`中命名的属性在每个实例的引用数组中，但也将支持动态创建的属性，这些属性将存储在通常的`__dict__`中。如果你想要使用`@cached_property`装饰器（在“第 5 步：使用 functools 缓存属性”中介绍），这是必要的。

当然，在`__slots__`中有`'__dict__'`可能完全打败它的目的，这取决于每个实例中静态和动态属性的数量以及它们的使用方式。粗心的优化比过早的优化更糟糕：你增加了复杂性，但可能得不到任何好处。

另一个你可能想要保留的特殊每实例属性是`__weakref__`，这对于对象支持弱引用是必要的（在“del 和垃圾回收”中简要提到）。该属性默认存在于用户定义类的实例中。但是，如果类定义了`__slots__`，并且你需要实例成为弱引用的目标，则需要在`__slots__`中包含`'__weakref__'`。

现在让我们看看将`__slots__`添加到`Vector2d`的效果。

## 简单的**槽**节省度量

示例 11-16 展示了在`Vector2d`中实现`__slots__`。

##### 示例 11-16\. vector2d_v3_slots.py：`__slots__`属性是`Vector2d`的唯一添加

```py
class Vector2d:
    __match_args__ = ('x', 'y')  # ①
    __slots__ = ('__x', '__y')  # ②

    typecode = 'd'
    # methods are the same as previous version
```

①

`__match_args__`列出了用于位置模式匹配的公共属性名称。

②

相比之下，`__slots__`列出了实例属性的名称，这些属性在这种情况下是私有属性。

为了测量内存节省，我编写了*mem_test.py*脚本。它接受一个带有`Vector2d`类变体的模块名称作为命令行参数，并使用列表推导式构建一个包含 10,000,000 个`Vector2d`实例的`list`。在示例 11-17 中显示的第一次运行中，我使用`vector2d_v3.Vector2d`（来自示例 11-7）；在第二次运行中，我使用具有`__slots__`的版本，来自示例 11-16。

##### 示例 11-17\. mem_test.py 创建了 10 百万个`Vector2d`实例，使用了命名模块中定义的类

```py
$ time python3 mem_test.py vector2d_v3
Selected Vector2d type: vector2d_v3.Vector2d
Creating 10,000,000 Vector2d instances
Initial RAM usage:      6,983,680
  Final RAM usage:  1,666,535,424

real	0m11.990s
user	0m10.861s
sys	0m0.978s
$ time python3 mem_test.py vector2d_v3_slots
Selected Vector2d type: vector2d_v3_slots.Vector2d
Creating 10,000,000 Vector2d instances
Initial RAM usage:      6,995,968
  Final RAM usage:    577,839,104

real	0m8.381s
user	0m8.006s
sys	0m0.352s
```

如示例 11-17 所示，当每个 10 百万个`Vector2d`实例中使用`__dict__`时，脚本的 RAM 占用量增长到了 1.55 GiB，但当`Vector2d`具有`__slots__`属性时，降低到了 551 MiB。`__slots__`版本也更快。这个测试中的*mem_test.py*脚本基本上处理加载模块、检查内存使用情况和格式化结果。你可以在[*fluentpython/example-code-2e*存储库](https://fpy.li/11-11)中找到它的源代码。

###### 提示

如果你处理数百万个具有数值数据的对象，你应该真的使用 NumPy 数组（参见“NumPy”），它们不仅内存高效，而且具有高度优化的数值处理函数，其中许多函数一次操作整个数组。我设计`Vector2d`类只是为了在讨论特殊方法时提供背景，因为我尽量避免在可以的情况下使用模糊的`foo`和`bar`示例。

## 总结`__slots__`的问题

如果正确使用，`__slots__`类属性可能会提供显著的内存节省，但有一些注意事项：

+   你必须记得在每个子类中重新声明`__slots__`，以防止它们的实例具有`__dict__`。

+   实例只能拥有`__slots__`中列出的属性，除非在`__slots__`中包含`'__dict__'`（但这样做可能会抵消内存节省）。

+   使用`__slots__`的类不能使用`@cached_property`装饰器，除非在`__slots__`中明确命名`'__dict__'`。

+   实例不能成为弱引用的目标，除非在`__slots__`中添加`'__weakref__'`。

本章的最后一个主题涉及在实例和子类中覆盖类属性。

# 覆盖类属性

Python 的一个显著特点是类属性可以用作实例属性的默认值。在`Vector2d`中有`typecode`类属性。它在`__bytes__`方法中使用了两次，但我们设计上将其读取为`self.typecode`。因为`Vector2d`实例是在没有自己的`typecode`属性的情况下创建的，所以`self.typecode`将默认获取`Vector2d.typecode`类属性。

但是，如果写入一个不存在的实例属性，就会创建一个新的实例属性，例如，一个`typecode`实例属性，而同名的类属性则保持不变。但是，从那时起，每当处理该实例的代码读取`self.typecode`时，实例`typecode`将被检索，有效地遮蔽了同名的类属性。这打开了使用不同`typecode`自定义单个实例的可能性。

默认的`Vector2d.typecode`是`'d'`，意味着每个向量分量在导出为`bytes`时将被表示为 8 字节的双精度浮点数。如果在导出之前将`Vector2d`实例的`typecode`设置为`'f'`，则每个分量将以 4 字节的单精度浮点数导出。示例 11-18 演示了这一点。

###### 注意

我们正在讨论添加自定义实例属性，因此示例 11-18 使用了没有`__slots__`的`Vector2d`实现，如示例 11-11 中所列。

##### 示例 11-18。通过设置以前从类继承的`typecode`属性来自定义实例

```py
>>> from vector2d_v3 import Vector2d
>>> v1 = Vector2d(1.1, 2.2)
>>> dumpd = bytes(v1)
>>> dumpd
b'd\x9a\x99\x99\x99\x99\x99\xf1?\x9a\x99\x99\x99\x99\x99\x01@' >>> len(dumpd)  # ①
17 >>> v1.typecode = 'f'  # ②
>>> dumpf = bytes(v1)
>>> dumpf
b'f\xcd\xcc\x8c?\xcd\xcc\x0c@' >>> len(dumpf)  # ③
9 >>> Vector2d.typecode  # ④
'd'
```

①](#co_a_pythonic_object_CO13-1)

默认的`bytes`表示长度为 17 字节。

②

在`v1`实例中将`typecode`设置为`'f'`。

③

现在`bytes`转储的长度为 9 字节。

④

`Vector2d.typecode`保持不变；只有`v1`实例使用`typecode`为`'f'`。

现在应该清楚为什么`Vector2d`的`bytes`导出以`typecode`为前缀：我们想要支持不同的导出格式。

如果要更改类属性，必须直接在类上设置，而不是通过实例。你可以通过以下方式更改所有实例（没有自己的`typecode`）的默认`typecode`：

```py
>>> Vector2d.typecode = 'f'
```

然而，在 Python 中有一种惯用的方法可以实现更持久的效果，并且更明确地说明更改。因为类属性是公共的，它们会被子类继承，所以习惯上是通过子类来定制类数据属性。Django 类基视图广泛使用这种技术。示例 11-19 展示了如何实现。

##### 示例 11-19。`ShortVector2d`是`Vector2d`的子类，只覆盖了默认的`typecode`

```py
>>> from vector2d_v3 import Vector2d
>>> class ShortVector2d(Vector2d):  # ①
...     typecode = 'f'
...
>>> sv = ShortVector2d(1/11, 1/27)  # ②
>>> sv
ShortVector2d(0.09090909090909091, 0.037037037037037035) # ③
>>> len(bytes(sv))  # ④
9
```

①

创建`ShortVector2d`作为`Vector2d`的子类，只是为了覆盖`typecode`类属性。

②

为演示构建`ShortVector2d`实例`sv`。

③

检查`sv`的`repr`。

④

检查导出字节的长度为 9，而不是之前的 17。

这个例子还解释了为什么我没有在`Vector2d.​__repr__`中硬编码`class_name`，而是从`type(self).__name__`获取它，就像这样：

```py
    # inside class Vector2d:

    def __repr__(self):
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)
```

如果我在`class_name`中硬编码，`Vector2d`的子类如`ShortVector2d`将不得不覆盖`__repr__`以更改`class_name`。通过从实例的`type`中读取名称，我使`__repr__`更安全地继承。

我们结束了构建一个简单类的覆盖，利用数据模型与 Python 的其他部分协作：提供不同的对象表示，提供自定义格式代码，公开只读属性，并支持 `hash()` 以与集合和映射集成。

# 章节总结

本章的目的是演示在构建一个良好的 Python 类时使用特殊方法和约定。

*vector2d_v3.py*（在 示例 11-11 中显示）比 *vector2d_v0.py*（在 示例 11-2 中显示）更符合 Python 风格吗？*vector2d_v3.py* 中的 `Vector2d` 类显然展示了更多的 Python 特性。但是第一个或最后一个 `Vector2d` 实现是否合适取决于它将被使用的上下文。Tim Peter 的“Python 之禅”说：

> 简单胜于复杂。

对象应该尽可能简单，符合需求，而不是语言特性的大杂烩。如果代码是为了一个应用程序，那么它应该专注于支持最终用户所需的内容，而不是更多。如果代码是为其他程序员使用的库，那么实现支持 Python 程序员期望的特殊方法是合理的。例如，`__eq__` 可能不是支持业务需求所必需的，但它使类更容易测试。

我在扩展 `Vector2d` 代码的目标是为了讨论 Python 特殊方法和编码约定提供背景。本章的示例演示了我们在 Table 1-1（第一章）中首次看到的几个特殊方法：

+   字符串/字节表示方法：`__repr__`、`__str__`、`__format__` 和 `__bytes__`

+   将对象转换为数字的方法：`__abs__`、`__bool__` 和 `__hash__`

+   `__eq__` 运算符，用于支持测试和哈希（以及 `__hash__`）

在支持转换为 `bytes` 的同时，我们还实现了一个替代构造函数 `Vector2d.frombytes()`，这为讨论装饰器 `@classmethod`（非常方便）和 `@staticmethod`（不太有用，模块级函数更简单）提供了背景。`frombytes` 方法受到了 `array.array` 类中同名方法的启发。

我们看到 [格式规范迷你语言](https://fpy.li/fmtspec) 可通过实现 `__format__` 方法来扩展，该方法解析提供给 `format(obj, format_spec)` 内置函数或在 f-strings 中使用的替换字段 `'{:«format_spec»}'` 中的 `format_spec`。

为了使 `Vector2d` 实例可哈希，我们努力使它们是不可变的，至少通过将 `x` 和 `y` 属性编码为私有属性，然后将它们公开为只读属性来防止意外更改。然后，我们使用推荐的异或实例属性哈希的技术实现了 `__hash__`。

我们随后讨论了在 `Vector2d` 中声明 `__slots__` 属性的内存节省和注意事项。因为使用 `__slots__` 会产生副作用，所以只有在处理非常大量的实例时才是有意义的——考虑的是百万级的实例，而不仅仅是千个。在许多这种情况下，使用 [pandas](https://fpy.li/pandas) 可能是最佳选择。

我们讨论的最后一个主题是覆盖通过实例访问的类属性（例如，`self.typecode`）。我们首先通过创建实例属性，然后通过子类化和在类级别上重写来实现。

在整个章节中，我提到示例中的设计选择是通过研究标准 Python 对象的 API 而得出的。如果这一章可以用一句话总结，那就是：

> 要构建 Pythonic 对象，观察真实的 Python 对象的行为。
> 
> 古老的中国谚语

# 进一步阅读

本章涵盖了数据模型的几个特殊方法，因此主要参考资料与第一章中提供的相同，该章节提供了相同主题的高层次视图。为方便起见，我将在此重复之前的四个推荐，并添加一些其他的：

*Python 语言参考*的[“数据模型”章节](https://fpy.li/dtmodel)

我们在本章中使用的大多数方法在[“3.3.1\.基本自定义”](https://fpy.li/11-12)中有文档记录。

[*Python 速查手册*, 第 3 版](https://fpy.li/pynut3)，作者 Alex Martelli, Anna Ravenscroft 和 Steve Holden

深入讨论了特殊方法。

[*Python 食谱*, 第 3 版](https://fpy.li/pycook3)，作者 David Beazley 和 Brian K. Jones

通过示例演示了现代 Python 实践。特别是第八章“类和对象”中有几个与本章讨论相关的解决方案。

*Python 基础参考*, 第 4 版，作者 David Beazley

详细介绍了数据模型，即使只涵盖了 Python 2.6 和 3.0（在第四版中）。基本概念都是相同的，大多数数据模型 API 自 Python 2.2 以来都没有改变，当时内置类型和用户定义类被统一起来。

在 2015 年，我完成第一版*流畅的 Python*时，Hynek Schlawack 开始了`attrs`包。从`attrs`文档中：

> `attrs`是 Python 包，通过解除你实现对象协议（也称为 dunder 方法）的繁琐，为**编写类**带来**乐趣**。

我在“进一步阅读”中提到`attrs`作为`@dataclass`的更强大替代品。来自第五章的数据类构建器以及`attrs`会自动为你的类配备几个特殊方法。但了解如何自己编写这些特殊方法仍然是必要的，以理解这些包的功能，决定是否真正需要它们，并在必要时覆盖它们生成的方法。

在本章中，我们看到了与对象表示相关的所有特殊方法，除了`__index__`和`__fspath__`。我们将在第十二章中讨论`__index__`，“一个切片感知的 __getitem__”。我不会涉及`__fspath__`。要了解更多信息，请参阅[PEP 519—添加文件系统路径协议](https://fpy.li/pep519)。

早期意识到对象需要不同的字符串表示的需求出现在 Smalltalk 中。1996 年 Bobby Woolf 的文章[“如何将对象显示为字符串：printString 和 displayString”](https://fpy.li/11-13)讨论了该语言中`printString`和`displayString`方法的实现。从那篇文章中，我借用了“开发者想要看到的方式”和“用户想要看到的方式”这两个简洁的描述，用于定义`repr()`和`str()`在“对象表示”中。

¹ 来自 Faassen 的博客文章[“什么是 Pythonic？”](https://fpy.li/11-1)

² 我在这里使用`eval`来克隆对象只是为了说明`repr`；要克隆一个实例，`copy.copy`函数更安全更快。

³ 这一行也可以写成`yield self.x; yield.self.y`。关于`__iter__`特殊方法、生成器表达式和`yield`关键字，我在第十七章中还有很多要说。

⁴ 我们在“内存视图”中简要介绍了`memoryview`，解释了它的`.cast`方法。

⁵ 本书的技术审阅员之一 Leonardo Rochael 不同意我对 `staticmethod` 的低评价，并推荐 Julien Danjou 的博文[“如何在 Python 中使用静态、类或抽象方法的权威指南”](https://fpy.li/11-2)作为反驳意见。Danjou 的文章非常好；我推荐它。但这并不足以改变我的对 `staticmethod` 的看法。你需要自己决定。

⁶ 私有属性的利弊是即将到来的“Python 中的私有和‘受保护’属性”的主题。

⁷ 来自[“粘贴风格指南”](https://fpy.li/11-8)。

⁸ 在模块中，顶层名称前的单个 `_` 确实有影响：如果你写 `from mymod import *`，带有 `_` 前缀的名称不会从 `mymod` 中导入。然而，你仍然可以写 `from mymod import _privatefunc`。这在[*Python 教程*，第 6.1 节，“关于模块的更多内容”](https://fpy.li/11-9)中有解释。

⁹ 一个例子在[gettext 模块文档](https://fpy.li/11-10)中。

¹⁰ 如果这种情况让你沮丧，并且让你希望 Python 在这方面更像 Java，那就不要阅读我对 Java `private` 修饰符相对强度的讨论，见“Soapbox”。

¹¹ 参见[“可能的最简单的工作方式：与沃德·坎宁安的对话，第五部分”](https://fpy.li/11-14)。
