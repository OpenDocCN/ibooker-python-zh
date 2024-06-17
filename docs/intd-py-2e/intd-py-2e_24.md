# 第二十二章：Py Sci

> 在她统治期间，蒸汽的力量
> 
> 在陆地和海上变得无与伦比，
> 
> 现在所有都有强烈的依赖
> 
> 在科学的新胜利中。
> 
> 詹姆斯·麦金泰尔，《女王 1887 年的银禧颂歌》

近几年来，主要因为你将在本章看到的软件，Python 在科学家中变得非常流行。如果你是科学家或学生，你可能已经使用过类似 MATLAB 和 R 的工具，或者传统语言如 Java、C 或 C++。现在你将看到 Python 如何成为科学分析和出版的优秀平台。

# 标准库中的数学和统计

首先，让我们回到标准库，看看我们忽略了的一些特性和模块。

## 数学函数

Python 在标准[math](https://oreil.ly/01SHP)库中拥有丰富的数学函数。只需输入**`import math`**即可从你的程序中访问它们。

它还有一些常量，如`pi`和`e`：

```py
>>> import math
>>> math.pi
>>> 3.141592653589793
>>> math.e
2.718281828459045
```

大部分由函数组成，让我们看看最有用的几个。

`fabs()`返回其参数的绝对值：

```py
>>> math.fabs(98.6)
98.6
>>> math.fabs(-271.1)
271.1
```

获取某个数字下方（`floor()`）和上方（`ceil()`）的整数：

```py
>>> math.floor(98.6)
98
>>> math.floor(-271.1)
-272
>>> math.ceil(98.6)
99
>>> math.ceil(-271.1)
-271
```

使用`factorial()`来计算阶乘（数学中的*n*`!`）：

```py
>>> math.factorial(0)
1
>>> math.factorial(1)
1
>>> math.factorial(2)
2
>>> math.factorial(3)
6
>>> math.factorial(10)
3628800
```

使用`log()`在自然对数`e`的基础上获取参数的对数：

```py
>>> math.log(1.0)
0.0
>>> math.log(math.e)
1.0
```

如果你想要不同基数的对数，可以提供第二个参数：

```py
>>> math.log(8, 2)
3.0
```

函数`pow()`执行相反操作，将一个数提升为某个幂次方：

```py
>>> math.pow(2, 3)
8.0
```

Python 还有内置的指数运算符`**`来完成相同的功能，但如果底数和指数都是整数，结果不会自动转换为浮点数：

```py
>>> 2**3
8
>>> 2.0**3
8.0
```

使用`sqrt()`获取平方根：

```py
>>> math.sqrt(100.0)
10.0
```

不要试图愚弄这个函数；它以前见过所有的东西：

```py
>>> math.sqrt(-100.0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: math domain error
```

所有常见的三角函数都在这里，我会在这里列出它们的名字：`sin()`，`cos()`，`tan()`，`asin()`，`acos()`，`atan()`，和`atan2()`。如果你记得毕达哥拉斯定理（或者能快速连续三次说出来而不喷出口水），数学库还有一个`hypot()`函数用于计算两边的斜边：

```py
>>> x = 3.0
>>> y = 4.0
>>> math.hypot(x, y)
5.0
```

如果你不信任所有这些花哨的函数，你可以自己解决：

```py
>>> math.sqrt(x*x + y*y)
5.0
>>> math.sqrt(x**2 + y**2)
5.0
```

一组最后的函数转换角坐标：

```py
>>> math.radians(180.0)
3.141592653589793
>>> math.degrees(math.pi)
180.0
```

## 处理复数

复数在基本的 Python 语言中得到了完全支持，具有熟悉的*实部*和*虚部*的表示法：

```py
>>> # a real number
... 5
5
>>> # an imaginary number
... 8j
8j
>>> # an imaginary number
... 3 + 2j
(3+2j)
```

因为虚数`i`（Python 中的`1j`）定义为–1 的平方根，我们可以执行以下操作：

```py
>>> 1j * 1j
(-1+0j)
>>> (7 + 1j) * 1j
(-1+7j)
```

一些复杂的数学函数在标准[`cmath`](https://oreil.ly/1EZQ0)模块中。

## 使用十进制计算准确的浮点数

计算机中的浮点数不像我们在学校学到的实数那样：

```py
>>> x = 10.0 / 3.0
>>> x
3.3333333333333335
```

嘿，末尾的`5`是什么？应该一路都是`3`。这是因为计算机 CPU 寄存器中只有有限的位数，而不是 2 的整数次幂的数字无法被精确表示。

使用 Python 的[`decimal`](https://oreil.ly/o-bmR)模块，您可以将数字表示为您所需的显著位数。这对涉及货币的计算特别重要。美元货币的最小单位是一分（美元的百分之一），因此如果我们将金额表示为美元和分，我们希望准确到分。如果我们尝试使用浮点值（如 19.99 和 0.06）表示美元和分，我们将在开始计算之前失去一些显著性。我们如何处理这个问题？很简单，我们使用`decimal`模块：

```py
>>> from decimal import Decimal
>>> price = Decimal('19.99')
>>> tax = Decimal('0.06')
>>> total = price + (price * tax)
>>> total
Decimal('21.1894')
```

我们创建了价格和税费的字符串值以保留它们的意义。`total`的计算保留了所有显著的分数部分，但我们希望获得最接近的分数：

```py
>>> penny = Decimal('0.01')
>>> total.quantize(penny)
Decimal('21.19')
```

可能会使用普通的浮点数和四舍五入得到相同的结果，但并非总是如此。您也可以将所有内容乘以 100，并在计算中使用整数分（cents），但最终这也会给您带来麻烦。

## 使用 fractions 执行有理算术

您可以通过标准 Python [`fractions`](https://oreil.ly/l286g)模块将数字表示为分子除以分母。这里是一个简单的操作，将三分之一乘以三分之二：

```py
>>> from fractions import Fraction
>>> Fraction(1,  3) * Fraction(2, 3)
Fraction(2, 9)
```

浮点数参数可能不精确，因此您可以在`Fraction`内使用`Decimal`：

```py
>>> Fraction(1.0/3.0)
Fraction(6004799503160661, 18014398509481984)
>>> Fraction(Decimal('1.0')/Decimal('3.0'))
Fraction(3333333333333333333333333333, 10000000000000000000000000000)
```

使用`gcd`函数获取两个数字的最大公约数：

```py
>>> import fractions
>>> fractions.gcd(24, 16)
8
```

## 使用`array`处理打包的序列

Python 列表更像是链表而不是数组。如果您想要相同类型的一维序列，请使用[`array`](https://oreil.ly/VejPU)类型。它比列表使用更少的空间，并支持许多列表方法。使用`array(` *`typecode`* , *`initializer`* `)`来创建一个。*typecode*指定数据类型（如`int`或`float`），可选的*`initializer`*包含初始值，可以指定为列表、字符串或可迭代对象。

我从未真正使用这个包进行实际工作。它是一个低级数据结构，适用于诸如图像数据之类的事物。如果您确实需要一个数组——尤其是带有多个维度——来进行数值计算，那么使用 NumPy 要好得多，稍后我们会讨论它。

## 使用`statistics`处理简单的统计数据

从 Python 3.4 开始，[`statistics`](https://oreil.ly/DELnM)是一个标准模块。它具有常见的函数：均值、中位数、众数、标准差、方差等等。输入参数是各种数值数据类型的序列（列表或元组）或迭代器：int、float、decimal 和 fraction。一个函数，`mode`，还接受字符串。许多更多的统计函数可以在诸如 SciPy 和 Pandas 等包中找到，稍后在本章中介绍。

## 矩阵乘法

从 Python 3.5 开始，你会看到 `@` 字符有些不同寻常的用法。它仍然用于装饰器，但也有一个新的用法用于 [*矩阵乘法*](https://oreil.ly/fakoD)。如果你正在使用较旧版本的 Python，NumPy（即将介绍）是你的最佳选择。

# 科学 Python

本章的其余部分涵盖了用于科学和数学的第三方 Python 包。虽然你可以单独安装它们，但考虑作为科学 Python 发行版的一部分同时下载所有这些包会更好。以下是你的主要选择：

[Anaconda](https://www.anaconda.com)

免费、全面、最新，支持 Python 2 和 3，并且不会破坏现有的系统 Python。

[Enthought Canopy](https://assets.enthought.com/downloads)

提供免费和商业版本

[Python(x,y)](https://python-xy.github.io)

仅适用于 Windows 版本

[Pyzo](http://www.pyzo.org)

基于 Anaconda 的一些工具，以及其他一些工具

我建议安装 Anaconda。它很大，但本章的所有示例都包含在其中。本章的其余示例将假定你已安装了所需的包，无论是单独安装还是作为 Anaconda 的一部分。

# NumPy

[NumPy](http://www.numpy.org) 是 Python 在科学家中流行的主要原因之一。你听说过动态语言如 Python 通常比编译语言如 C 或甚至其他解释语言如 Java 慢。NumPy 的出现是为了提供快速的多维数值数组，类似于科学语言如 FORTRAN。你既得到了 C 的速度，又保留了 Python 的开发者友好性。

如果你已经下载了其中一个科学 Python 发行版，那么你已经拥有了 NumPy。如果没有，请按照 NumPy 的 [下载页面](https://oreil.ly/HcZZi) 上的说明操作。

要开始使用 NumPy，你应该了解一个核心数据结构，称为 *ndarray*（*N-dimensional array* 的缩写）或简称 *array* 的多维数组。与 Python 的列表和元组不同，每个元素都需要是相同类型的。NumPy 将数组的维数称为其 *rank*。一维数组类似于值的行，二维数组类似于行和列的表，三维数组类似于魔方。各维度的长度不必相同。

###### 注意

NumPy 的 `array` 和标准的 Python `array` 不是一回事。本章的其余部分中，当我说 *array* 时，我指的是 NumPy 数组。

但是为什么你需要一个数组？

+   科学数据通常包含大量的数据序列。

+   在这些数据上进行科学计算通常使用矩阵数学、回归、模拟以及其他处理大量数据点的技术。

+   NumPy 处理数组比标准的 Python 列表或元组快得多。

有许多方法可以创建一个 NumPy 数组。

## 使用 array() 创建一个数组

你可以从普通列表或元组创建一个数组：

```py
>>> b = np.array( [2, 4, 6, 8] )
>>> b
array([2, 4, 6, 8])
```

属性 `ndim` 返回数组的维数：

```py
>>> b.ndim
1
```

数组中的值的总数由`size`给出：

```py
>>> b.size
4
```

每个秩中的值的数量由`shape`返回：

```py
>>> b.shape
(4,)
```

## 使用`arange()`创建一个数组

NumPy 的`arange()`方法类似于 Python 的标准`range()`。如果你用一个整数参数`num`调用`arange()`，它会返回一个从`0`到`num-1`的`ndarray`：

```py
>>> import numpy as np
>>> a = np.arange(10)
>>> a
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> a.ndim
1
>>> a.shape
(10,)
>>> a.size
10
```

用两个值，它创建一个从第一个到最后一个的数组，减去一个：

```py
>>> a = np.arange(7, 11)
>>> a
array([ 7,  8,  9, 10])
```

你也可以提供一个步长作为第三个参数来使用：

```py
>>> a = np.arange(7, 11, 2)
>>> a
array([7, 9])
```

到目前为止，我们的例子都使用整数，但浮点数也完全可以：

```py
>>> f = np.arange(2.0, 9.8, 0.3)
>>> f
array([ 2\. ,  2.3,  2.6,  2.9,  3.2,  3.5,  3.8,  4.1,  4.4,  4.7,  5\. ,
 5.3,  5.6,  5.9,  6.2,  6.5,  6.8,  7.1,  7.4,  7.7,  8\. ,  8.3,
 8.6,  8.9,  9.2,  9.5,  9.8])
```

还有一个小技巧：`dtype`参数告诉`arange`要产生什么类型的值：

```py
>>> g = np.arange(10, 4, -1.5, dtype=np.float)
>>> g
array([ 10\. ,   8.5,   7\. ,   5.5])
```

## 使用 zeros()、ones() 或 random() 创建一个数组

`zeros()`方法返回一个所有值都为零的数组。你提供的参数是一个你想要的形状的元组。这是一个一维数组：

```py
>>> a = np.zeros((3,))
>>> a
array([ 0.,  0.,  0.])
>>> a.ndim
1
>>> a.shape
(3,)
>>> a.size
3
```

这个是二阶的：

```py
>>> b = np.zeros((2, 4))
>>> b
array([[ 0.,  0.,  0.,  0.],
 [ 0.,  0.,  0.,  0.]])
>>> b.ndim
2
>>> b.shape
(2, 4)
>>> b.size
8
```

用相同值填充数组的另一个特殊函数是`ones()`：

```py
>>> import numpy as np
>>> k = np.ones((3, 5))
>>> k
array([[ 1.,  1.,  1.,  1.,  1.],
 [ 1.,  1.,  1.,  1.,  1.],
 [ 1.,  1.,  1.,  1.,  1.]])
```

最后一个初始化器创建一个具有介于 0.0 和 1.0 之间的随机值的数组：

```py
>>> m = np.random.random((3, 5))
>>> m
array([[  1.92415699e-01,   4.43131404e-01,   7.99226773e-01,
 1.14301942e-01,   2.85383430e-04],
 [  6.53705749e-01,   7.48034559e-01,   4.49463241e-01,
 4.87906915e-01,   9.34341118e-01],
 [  9.47575562e-01,   2.21152583e-01,   2.49031209e-01,
 3.46190961e-01,   8.94842676e-01]])
```

## 使用`reshape()`改变数组的形状

到目前为止，数组似乎并不比列表或元组不同。一个区别是你可以让它执行诸如使用`reshape()`改变其形状的技巧：

```py
>>> a = np.arange(10)
>>> a
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> a = a.reshape(2, 5)
>>> a
array([[0, 1, 2, 3, 4],
 [5, 6, 7, 8, 9]])
>>> a.ndim
2
>>> a.shape
(2, 5)
>>> a.size
10
```

你可以以不同的方式重塑相同的数组：

```py
>>> a = a.reshape(5, 2)
>>> a
array([[0, 1],
 [2, 3],
 [4, 5],
 [6, 7],
 [8, 9]])
>>> a.ndim
2
>>> a.shape
(5, 2)
>>> a.size
10
```

将一个具有形状的元组分配给`shape`会做同样的事情：

```py
>>> a.shape = (2, 5)
>>> a
array([[0, 1, 2, 3, 4],
 [5, 6, 7, 8, 9]])
```

形状的唯一限制是秩大小的乘积需要等于值的总数（在本例中为 10）：

```py
>>> a = a.reshape(3, 4)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: total size of new array must be unchanged
```

## 使用[]获取一个元素

一维数组的工作方式类似于列表：

```py
>>> a = np.arange(10)
>>> a[7]
7
>>> a[-1]
9
```

但是，如果数组的形状不同，请在方括号内使用逗号分隔的索引：

```py
>>> a.shape = (2, 5)
>>> a
array([[0, 1, 2, 3, 4],
 [5, 6, 7, 8, 9]])
>>> a[1,2]
7
```

这与二维 Python 列表不同，后者的索引位于单独的方括号中：

```py
>>> l = [ [0, 1, 2, 3, 4], [5, 6, 7, 8, 9] ]
>>> l
[[0, 1, 2, 3, 4], [5, 6, 7, 8, 9]]
>>> l[1,2]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: list indices must be integers, not tuple
>>> l[1][2]
7
```

最后一件事：切片是有效的，但再次强调，只能在一组方括号内。让我们再次创建我们熟悉的测试数组：

```py
>>> a = np.arange(10)
>>> a = a.reshape(2, 5)
>>> a
array([[0, 1, 2, 3, 4],
       [5, 6, 7, 8, 9]])
```

使用切片获取第一行，从偏移量`2`到末尾的元素：

```py
>>> a[0, 2:]
array([2, 3, 4])
```

现在，获取最后一行，从末尾倒数第三个元素开始：

```py
>>> a[-1, :3]
array([5, 6, 7])
```

你也可以使用切片为多个元素赋值。以下语句将值`1000`赋给所有行的列（偏移量）`2`和`3`：

```py
>>> a[:, 2:4] = 1000
>>> a
array([[   0,    1, 1000, 1000,    4],
 [   5,    6, 1000, 1000,    9]])
```

## 数组数学

制作和重塑数组是如此有趣，以至于我们几乎忘记了实际上*要*用它们做些什么。作为我们的第一个技巧，我们使用 NumPy 的重新定义的乘法（`*`）运算符一次性将 NumPy 数组中的所有值相乘：

```py
>>> from numpy import *
>>> a = arange(4)
>>> a
array([0, 1, 2, 3])
>>> a *= 3
>>> a
array([0, 3, 6, 9])
```

如果你尝试将普通的 Python 列表中的每个元素乘以一个数字，你将需要一个循环或列表推导：

```py
>>> plain_list = list(range(4))
>>> plain_list
[0, 1, 2, 3]
>>> plain_list = [num * 3 for num in plain_list]
>>> plain_list
[0, 3, 6, 9]
```

这种一次性行为也适用于 NumPy 库中的加法、减法、除法和其他函数。例如，你可以使用`zeros()`和`+`将数组的所有成员初始化为任何值：

```py
>>> from numpy import *
>>> a = zeros((2, 5)) + 17.0
>>> a
array([[ 17.,  17.,  17.,  17.,  17.],
 [ 17.,  17.,  17.,  17.,  17.]])
```

## 线性代数

NumPy 包含许多线性代数函数。例如，让我们定义这个线性方程组：

```py
4x + 5y = 20
 x + 2y = 13
```

我们如何解出`x`和`y`？我们构建两个数组：

+   *系数*（`x`和`y`的乘数）

+   *依赖*变量（方程的右侧）

```py
>>> import numpy as np
>>> coefficients = np.array([ [4, 5], [1, 2] ])
>>> dependents = np.array( [20, 13] )
```

现在，在`linalg`模块中使用`solve()`函数：

```py
>>> answers = np.linalg.solve(coefficients, dependents)
>>> answers
array([ -8.33333333,  10.66666667])
```

结果显示`x`约为–8.3，`y`约为 10.6。这些数字是否解出了方程？

```py
>>> 4 * answers[0] + 5 * answers[1]
20.0
>>> 1 * answers[0] + 2 * answers[1]
13.0
```

太棒了。为了避免输入所有内容，您还可以要求 NumPy 为您获取数组的*点积*：

```py
>>> product = np.dot(coefficients, answers)
>>> product
array([ 20.,  13.])
```

如果此解决方案正确，则`product`数组中的值应接近`dependents`中的值。您可以使用`allclose()`函数来检查数组是否大致相等（由于浮点数精度问题，它们可能并不完全相等）：

```py
>>> np.allclose(product, dependents)
True
```

NumPy 还包括用于多项式、傅里叶变换、统计以及一些概率分布的模块。

# SciPy

在 NumPy 之上构建的数学和统计函数库还有更多：[SciPy](http://www.scipy.org)。SciPy 的[发布](https://oreil.ly/Yv7G-)包括 NumPy、SciPy、Pandas（本章后续会讨论）、以及其他库。

SciPy 包括许多模块，包括以下任务：

+   优化

+   统计

+   插值

+   线性回归

+   积分

+   图像处理

+   信号处理

如果您曾经使用过其他科学计算工具，您会发现 Python、NumPy 和 SciPy 与商业软件[MATLAB](https://oreil.ly/jOPMO)或开源软件[R](http://www.r-project.org)涵盖了一些相同的领域。

# SciKit

在类似之前软件的模式下，[SciKit](https://scikits.appspot.com)是基于 SciPy 构建的一组科学软件包。[SciKit-Learn](https://scikit-learn.org)是一个重要的*机器学习*软件包：它支持建模、分类、聚类以及各种算法。

# Pandas

最近，“数据科学”这个术语变得很常见。我见过一些定义，包括“在 Mac 上进行的统计分析”或“在旧金山进行的统计分析”。不管您如何定义它，本章讨论的工具——NumPy、SciPy 以及本节的主题 Pandas——都是一个日益流行的数据科学工具包的组成部分。（Mac 和旧金山是可选的。）

[Pandas](http://pandas.pydata.org)是一种用于交互式数据分析的新软件包。它特别适用于真实世界的数据处理，结合了 NumPy 的矩阵数学和电子表格以及关系数据库的处理能力。Wes McKinney（O’Reilly）的书籍[*Python for Data Analysis: Data Wrangling with Pandas, NumPy, and IPython*](http://bit.ly/python_for_data_analysis)介绍了使用 NumPy、IPython 和 Pandas 进行数据整理。

NumPy 致力于传统的科学计算，通常用于处理单一类型的多维数据集，通常是浮点数。而 Pandas 更像是一个数据库编辑器，可以处理多种数据类型的组合。在某些语言中，这些组合被称为*记录*或*结构*。Pandas 定义了一个名为`DataFrame`的基本数据结构。它是具有名称和类型的列的有序集合。它与数据库表、Python 的命名元组和嵌套字典有些相似。其目的在于简化您可能会遇到的数据处理，不仅仅局限于科学领域，还包括商业领域。事实上，Pandas 最初是为了处理金融数据而设计的，而其最常见的替代品是电子表格。

Pandas 是一个处理现实世界中混乱数据（缺失值、奇怪的格式、零散的测量）的 ETL 工具，支持各种数据类型。您可以分割、连接、扩展、填充、转换、重塑、切片、加载和保存文件。它与我们刚讨论过的工具集成在一起——NumPy、SciPy、iPython——用于计算统计数据、将数据拟合到模型、绘制图表、发布等等。

大多数科学家只想完成他们的工作，而不是花数月时间成为奇特计算机语言或应用的专家。使用 Python，他们可以更快地变得高效。

# Python 与科学领域

我们已经看过几乎适用于任何科学领域的 Python 工具。那么针对特定科学领域的软件和文档呢？以下是 Python 在特定问题和一些特殊用途库中的应用示例：

一般

+   [科学与工程中的 Python 计算](http://bit.ly/py-comp-sci)

+   [科学家的 Python 速成课程](http://bit.ly/pyforsci)

物理学

+   [计算物理学](http://bit.ly/comp-phys-py)

+   [Astropy](https://www.astropy.org)

+   [SunPy](https://sunpy.org)（太阳数据分析）

+   [MetPy](https://unidata.github.io/MetPy)（气象数据分析）

+   [Py-ART](https://arm-doe.github.io/pyart)（天气雷达）

+   [社区比较套件](http://www.cistools.net)（大气科学）

+   [Freud](https://freud.readthedocs.io)（轨迹分析）

+   [Platon](https://platon.readthedocs.io)（系外行星大气层）

+   [PSI4](http://psicode.org)（量子化学）

+   [OpenQuake Engine](https://github.com/gem/oq-engine)

+   [yt](https://yt-project.org)（体积数据分析）

生物学和医学

+   [Biopython](https://biopython.org)

+   [生物学家的 Python](http://pythonforbiologists.com)

+   [应用生物信息学导论](http://readiab.org)

+   [Python 中的神经影像学](http://nipy.org)

+   [MNE](https://www.martinos.org/mne)（神经生理数据可视化）

+   [PyMedPhys](https://pymedphys.com)

+   [Nengo](https://www.nengo.ai)（神经模拟器）

Python 和科学数据的国际会议包括以下内容：

+   [PyData](http://pydata.org)

+   [SciPy](http://conference.scipy.org)

+   [EuroSciPy](https://www.euroscipy.org)

# 即将到来

我们已经触及了可观察的 Python 宇宙的尽头，除了多元宇宙附录。

# 要做的事情

安装 Pandas。获取 示例 16-1 中的 CSV 文件。运行 示例 16-2 中的程序。尝试一些 Pandas 命令。
