# 第十六章 数值处理

您可以使用运算符（见“数值操作”）和内置函数（见“内置函数”）执行一些数值计算。Python 还提供了支持额外数值计算的模块，本章详细介绍：math 和 cmath、statistics、operator、random 和 secrets、fractions 和 decimal。数值处理经常需要更具体地处理数字*数组*；这个主题在“数组处理”中有所涉及，重点介绍标准库模块 array 和流行的第三方扩展 NumPy。最后，“额外的数值包”列出了 Python 社区生产的几个额外数值处理包。本章大多数示例假定您已经导入了适当的模块；**import** 语句仅在情况不明确时包含。

# 浮点数值

Python 用类型为 float 的变量表示实数值（即非整数）。与整数不同，由于其内部实现为固定大小的二进制整数*尾数*（通常错误地称为“尾数”）和固定大小的二进制整数指数，计算机很少能够精确表示浮点数。浮点数有几个限制（其中一些可能导致意外结果）。

对于大多数日常应用程序，浮点数足以进行算术运算，但它们在能够表示的小数位数上有限：

```py
>>> f = 1.1 + 2.2 - 3.3  *`# f should be equal to 0`*
>>> f
```

```py
4.440892098500626e-16
```

它们在能够精确存储的整数值范围上也有限制（即能够区分下一个最大或最小整数值）：

```py
>>> f = 2 ** 53
>>> f
```

```py
9007199254740992
```

```py
>>> f + 1
```

```py
9007199254740993    *# integer arithmetic is not bounded*
```

```py
>>> f + 1.0
```

```py
9007199254740992.0  *# float conversion loses integer precision at 2**53*
```

请始终记住，由于其在计算机中的内部表示，浮点数并不完全精确。对复数的考虑也是同样。

# 不要在浮点数或复数之间使用 == 运算符

鉴于浮点数和运算只能近似地模拟数学中的“实数”行为，几乎没有必要检查两个浮点数 *x* 和 *y* 是否完全相等。每个数的计算方式微小的差异很容易导致意外的差异。

要测试浮点数或复数是否相等，请使用内置模块 math 导出的函数 isclose。以下代码说明了原因：

```py
>>> `import` math
>>> f = 1.1 + 2.2 - 3.3  *`# f intuitively equal to 0`*
>>> f == 0
```

```py
False
```

```py
>>> f
```

```py
4.440892098500626e-16
```

```py
>>> *`# default tolerance fine for this comparison`*
>>> math.isclose(-1, f-1)
```

```py
True
```

对于某些值，您可能需要显式设置容差值（这总是*必需*当您与 0 进行比较时）：

```py
>>> *`# near-0 comparison with default tolerances`*
>>> math.isclose(0, f)
```

```py
False
```

```py
>>> *`# must use abs_tol for comparison with 0`*
>>> math.isclose(0, f, abs_tol=1e-15)
```

```py
True
```

你还可以使用 `isclose` 来进行安全的循环。

# 不要将浮点数用作循环控制变量

常见错误是将浮点值用作循环的控制变量，假设它最终会等于某个结束值，如 0。相反，它很可能最终将永远循环。

以下循环预期循环五次然后结束，但实际上将永远循环：

```py
>>> f = 1
>>> `while` f != 0:
... f -= 0.2 
```

尽管 f 最初是一个整数，现在它是一个浮点数。这段代码展示了为什么：

```py
>>> 1-0.2-0.2-0.2-0.2-0.2  *`# should be 0, but...`*
```

```py
5.551115123125783e-17
```

即使使用不等操作符 > 也会导致不正确的行为，循环六次而不是五次（因为剩余的浮点值仍然大于 0）：

```py
>>> f = 1
>>> count = 0
>>> `while` f > 0:
...     count += 1
...     f -= 0.2
>>> print(count)
```

```py
6   *# one loop too many!*
```

如果您使用 math.isclose 来比较*f*与 0，**for**循环将重复正确的次数：

```py
>>> f = 1
>>> count = 0
>>> `while` `not` math.isclose(0,f,abs_tol=1e-15):
...     count += 1
...     f -= 0.2
>>> print(count)
```

```py
5   *# just right this time!*
```

通常情况下，尽量使用 int 作为循环控制变量，而不是 float。

最终，导致非常大的浮点数的数学运算通常会引发 OverflowError，或者 Python 可能会将它们返回为 inf（无穷大）。在您的计算机上可用的最大浮点数值是 sys.float_info.max：在 64 位计算机上，它是 1.7976931348623157e+308。当处理非常大的数时，这可能导致意外的结果。当您需要处理非常大的数时，我们建议使用 decimal 模块或第三方[gmpy](https://oreil.ly/JWoAx)。

# math 和 cmath 模块

math 模块提供了用于处理浮点数的数学函数；cmath 模块提供了处理复数的等效函数。例如，math.sqrt(-1)会引发异常，但 cmath.sqrt(-1)会返回 1j。

就像对于任何其他模块一样，使用这些模块的最干净、最易读的方法是在代码顶部**import** math，并明确调用，例如 math.sqrt。然而，如果您的代码包含大量调用模块的已知数学函数，您可以（尽管可能会失去一些清晰度和可读性）要么使用**from** math **import** *，要么使用**from** math **import** sqrt，然后仅调用 sqrt。

每个模块都公开了三个浮点属性，绑定到基本数学常数 e、pi 和[tau](https://oreil.ly/2Sbrf)，以及各种函数，包括表 16-1 中显示的函数。math 和 cmath 模块并非完全对称，因此表格中每个方法都指示它是在 math、cmath 还是两者中。所有示例假定您已导入适当的模块。

表 16-1。math 和 cmath 模块的方法和属性

|   |   | math | cmath |
| --- | --- | --- | --- |
| acos, asin, atan, cos, sin, tan | acos(*x*)等，返回*x*的反三角函数，余弦、正弦、正切，单位为弧度。 | ✓ | ✓ |
| acosh, asinh, atanh, cosh, sinh, tanh | acosh(*x*)等，返回*x*的反双曲余弦、反双曲正弦、反双曲正切、双曲余弦、双曲正弦或双曲正切的值，单位为弧度。 | ✓ | ✓ |

| atan2 | atan2(*y, x*) 类似于 atan(*y*/*x*），但 atan2 正确考虑了两个参数的符号。例如：

```py
>>> math.atan(-1./-1.)
```

```py
0.7853981633974483
```

```py
>>> math.atan2(-1., -1.)
```

```py
-2.356194490192345
```

当*x*等于 0 时，atan2 返回π/2，而除以*x*会引发 ZeroDivisionError。 | ✓ |   |

| cbrt | cbrt(x) 3.11+ 返回*x*的立方根。 | ✓ |   |
| --- | --- | --- | --- |
| ceil | ceil(*x*) 返回不小于*x*的最小整数*i*的浮点数*i*。 | ✓ |   |
| comb | comb(*n*, *k*) 3.8+ Returns the number of *combinations* of *n* items taken *k* items at a time, regardless of order. When counting the number of combinations taken from three items *A*, *B*, and *C*, two at a time, comb(3, 2) returns 3 because, for example, *A*-*B* and *B*-*A* are considered the *same* combination (contrast this with perm, later in this table). Raises ValueError when *k* or *n* is negative; raises TypeError when *k* or *n* is not an int. When *k*>*n*, just returns 0, raising no exceptions. | ✓ |   |
| copysign | copysign(*x*, *y*) Returns the absolute value of *x* with the sign of *y*. | ✓ |   |
| degrees | degrees(*x*) Returns the degree measure of the angle *x* given in radians. | ✓ |   |
| dist | dist(*pt0*, *pt1*) 3.8+ Returns the Euclidean distance between two *n*-dimensional points, where each point is represented as a sequence of values (coordinates). Raises ValueError if *pt0* and *pt1* are sequences of different lengths. | ✓ |   |
| e | The mathematical constant *e* (2.718281828459045). | ✓ | ✓ |
| erf | erf(*x*) Returns the error function of *x* as used in statistical calculations. | ✓ |   |
| erfc | erfc(*x*) Returns the complementary error function at *x*, defined as 1.0 - erf(*x*). | ✓ |   |
| exp | exp(*x*) Returns eˣ. | ✓ | ✓ |
| exp2 | exp2(*x*) 3.11+ Returns 2ˣ. | ✓ |   |
| expm1 | expm1(*x*) Returns eˣ - 1. Inverse of log1p. | ✓ |   |
| fabs | fabs(*x*) Returns the absolute value of *x*. Always returns a float, even if *x* is an int (unlike the built-in abs function). | ✓ |   |
| factorial | factorial(*x*) Returns the factorial of *x*. Raises ValueError when *x* is negative, and TypeError when *x* is not integral. | ✓ |   |
| floor | floor(*x*) Returns float(*i*), where *i* is the greatest integer such that *i*<=*x*. | ✓ |   |
| fmod | fmod(*x, y*) Returns the float *r*, with the same sign as *x*, such that *r*==*x*-*n***y* for some integer *n*, and abs(*r*)<abs(*y*). Like *x*%*y*, except that, when *x* and *y* differ in sign, *x%y* has the same sign as *y*, not the same sign as *x*. | ✓ |   |
| frexp | frexp(*x*) Returns a pair (*m*, *e*) where *m* is a floating-point number and *e* is an integer such that *x*==*m**(2**e) and 0.5<=abs(*m*)<1,^(a) except that frexp(0) returns (0.0, 0). | ✓ |   |
| fsum | fsum(*iterable*) Returns the floating-point sum of the values in *iterable* to greater precision than the sum built-in function. | ✓ |   |
| gamma | gamma(*x*) Returns the Gamma function evaluated at *x*. | ✓ |   |
| gcd | gcd(*x, y*) Returns the greatest common divisor of *x* and *y*. When *x* and *y* are both zero, returns 0. (3.9+ gcd can accept any number of values; gcd() without arguments returns 0.) | ✓ |   |
| hypot | hypot(*x, y*) Returns sqrt(*x***x*+*y***y*). (3.8+ hypot can accept any number of values, to compute a hypotenuse length in *n* dimensions.) | ✓ |   |
| inf | A floating-point positive infinity, like float('inf'). | ✓ | ✓ |
| infj | 一个复数的无穷虚数，等于 complex(0, float('inf'))。 |   | ✓ |

| isclose | isclose(*x*, *y*, rel_tol=1e-09, abs_tol=0.0) 返回当*x*和*y*在相对容差 rel_tol 内近似相等时为**True**，最小绝对容差 abs_tol 为 0.0；否则返回**False**。默认情况下，rel_tol 是九位小数。rel_tol 必须大于 0。abs_tol 用于接近零的比较：必须至少为 0.0。NaN 不被认为接近任何值（包括 NaN 本身）；+/- inf 除外，isclose 类似于： |   |   |

```py
abs(x-y) <= max(rel_tol*max(abs(x), 
                abs(y)),abs_tol)
```

| ✓ | ✓ |
| --- | --- |
| isfinite | isfinite(*x*) 返回当*x*（在 cmath 中，*x*的实部和虚部）既不是无穷大也不是 NaN 时为**True**；否则返回**False**。 | ✓ | ✓ |
| isinf | isinf(*x*) 返回当*x*（在 cmath 中，*x*的实部或虚部，或两者）是正无穷大或负无穷大时为**True**；否则返回**False**。 | ✓ | ✓ |
| isnan | isnan(*x*) 返回当*x*（在 cmath 中，*x*的实部或虚部，或两者）是 NaN 时为**True**；否则返回**False**。 | ✓ | ✓ |
| isqrt | isqrt(*x*) 3.8+ 返回 int(sqrt(*x*))。 | ✓ |   |
| lcm | lcm(*x*, ...) 3.9+ 返回给定整数的最小公倍数。如果不是所有值都是整数，则引发 TypeError。 | ✓ |   |
| ldexp | ldexp(*x, i*) 返回*x*乘以*2****i*（*i*必须是整数；当*i*为浮点数时，ldexp 引发 TypeError）。是 frexp 的反函数。 | ✓ |   |
| lgamma | lgamma(*x*) 返回 Gamma 函数在*x*处的绝对值的自然对数。 | ✓ |   |
| log | log(*x*) 返回*x*的自然对数。 | ✓ | ✓ |
| log10 | log10(*x*) 返回*x*的以 10 为底的对数。 | ✓ | ✓ |
| log1p | log1p(*x*) 返回 1+*x*的自然对数。是 expm1 的反函数。 | ✓ |   |
| log2 | log2(*x*) 返回*x*的以 2 为底的对数。 | ✓ |   |
| modf | modf(*x*) 返回一个由*x*的小数部分和整数部分组成的元组(*f*, *i*)，即两个浮点数，其符号与*x*相同，使得*i*==int(*i*)且*x*==*f*+*i*。 | ✓ |   |
| nan | nan 一个浮点数“非数字”（NaN）值，类似于 float('nan')或 complex('nan')。 | ✓ | ✓ |
| nanj | 一个实部为 0.0 且虚部为浮点数“非数字”（NaN）的复数。 |   | ✓ |
| nextafter | nextafter(*a*, *b*) 3.9+ 返回从*a*开始向*b*方向上的下一个更高或更低的浮点值。 | ✓ |   |
| perm | perm(*n*, *k*) 3.8+ 返回从*n*个项目中取*k*个项目的排列数，其中相同项目的不同顺序选择被单独计算。例如，对于三个项目*A*、*B*和*C*，取两个项目的排列数为 perm(3, 2)，返回 6，因为例如*A*-*B*和*B*-*A*被视为不同的排列（与此表中较早的 comb 相对比）。当*k*或*n*为负时引发 ValueError；当*k*或*n*不是整数时引发 TypeError。 | ✓ |   |
| π | 数学常数*π*，3.141592653589793。 | ✓ | ✓ |
| 相位 | phase(*x*) 返回*x*的相位，为一个范围在(*–π*, *π*)的浮点数。类似于 math.atan2(*x*.imag, *x*.real)。详见[在线文档](https://oreil.ly/gXdbT)。 |   | ✓ |
| 极坐标 | polar(*x*) 返回复数*x*的极坐标表示，即一对(*r*, *phi*)，其中*r*为*x*的模，*phi*为*x*的相位。类似于(abs(*x*), cmath.phase(*x*))。详见[在线文档](https://oreil.ly/gXdbT)。 |   | ✓ |
| pow | pow(*x, y*) 返回 float(*x*)的 float(*y*)次幂。对于大整数值的*x*和*y*，为避免 OverflowError 异常，请使用*x****y*或不转换为浮点数的内置 pow 函数。 | ✓ |   |
| prod | prod(*seq*, start=1) 3.8+ 返回序列中所有值的乘积，从给定的起始值（默认为 1）开始。如果*seq*为空，则返回起始值。 | ✓ |   |
| radians | radians(*x*) 返回角度*x*的弧度值。 | ✓ |   |
| rect | rect(*r*, *phi*) 返回以极坐标(*r*, *phi*)表示的复数值，转换为矩形坐标形式为(*x* + *yj*)。 |   | ✓ |
| remainder | remainder(*x*, *y*) 返回*x*/*y*的有符号余数（如果*x*或*y*为负，则结果可能为负）。 | ✓ |   |
| sqrt | sqrt(*x*) 返回*x*的平方根。 | ✓ | ✓ |
| τ | 数学常数*τ* = 2*π*，即 6.283185307179586。 | ✓ | ✓ |
| trunc | trunc(*x*) 返回被截断为整数的*x*。 | ✓ |   |
| ulp | ulp(*x*) 3.9+ 返回浮点数*x*的最低有效位。对于正值，等同于 math.nextafter(*x*, *x*+1) - x。对于负值，等同于 ulp(-*x*)。如果*x*为 NaN 或 inf，则返回*x*。ulp 代表[*最小精度单位*](https://oreil.ly/6cN99)。 | ✓ |   |
| ^(a) 形式上，*m*为尾数，*e*为指数。用于渲染浮点值的跨平台可移植表示。 |

# 统计模块

统计模块提供了 NormalDist 类用于执行分布分析，并提供了表 16-2 中列出的函数来计算常见统计数据。

表 16-2\. 统计模块的函数（包括版本 3.8 和 3.10 新增的函数）

|   | 3.8+ | 3.10+ |
| --- | --- | --- |

| harmonic_mean mean

中位数

中位数（分组）

中位数（高）

中位数（低）

众数

总体标准差

总体方差

标准差

| 方差 | fmean geometric_mean

多模态

分位数

NormalDist | 相关性 协方差

线性回归 |

[在线文档](https://oreil.ly/CY8Pi)详细介绍了这些函数的签名和用法。

# 运算符模块

运算符模块提供了一些等效于 Python 操作符的函数。这些函数在需要存储可调用对象、作为参数传递或作为函数结果返回的情况下非常方便。运算符模块中的函数与相应的特殊方法（在“特殊方法”中介绍）具有相同的名称。每个函数都有两个名称，一个带有“dunder”（前导和尾随双下划线）和一个不带有：“例如，operator.add(*a*, *b*)和 operator.__add__(*a*, *b*)都返回*a + b*。

添加了用于中缀运算符 @ 的矩阵乘法支持，但必须通过定义自己的 __matmul__、__rmatmul__ 和/或 __imatmul__ 方法来实现它；NumPy 目前支持 @（但在撰写本文时尚未支持 @=）进行矩阵乘法。

表 16-3 列出了运算符模块提供的一些函数。有关这些函数及其用法的详细信息，请参阅[在线文档](https://oreil.ly/WrtUH)。

表 16-3\. 运算符模块提供的函数

| Function | Signature | 行为类似 |
| --- | --- | --- |
| abs | abs(*a*) | abs(*a*) |
| add | add(*a*, *b*) | *a* + *b* |
| and_ | and_(*a*, *b*) | *a* & *b* |
| concat | concat(*a*, *b*) | *a* + *b* |
| contains | contains(*a*, *b*) | *b* **在** *a* 中 |
| countOf | countOf(*a*, *b*) | *a*.count(*b*) |
| delitem | delitem(*a*, *b*) | **del** *a*[*b*] |
| delslice | delslice(*a*, *b*, *c*) | **del** *a*[*b*:*c*] |
| eq | eq(*a*, *b*) | *a* == *b* |
| floordiv | floordiv(*a*, *b*) | *a* // *b* |
| ge | ge(*a*, *b*) | *a* >= *b* |
| getitem | getitem(*a*, *b*) | *a*[*b*] |
| getslice | getslice(*a*, *b*, *c*) | *a*[*b*:*c*] |
| gt | gt(*a*, *b*) | *a* > *b* |
| index | index(*a*) | *a*.__index__() |
| indexOf | indexOf(*a*, *b*) | *a*.index(*b*) |
| invert, inv | invert(*a*)*,* inv(*a*) | ~*a* |
| is_ | is_(*a*, *b*) | *a* **是** *b* |
| is_not | is_not(*a*, *b*) | *a* **不是** *b* |
| le | le(*a*, *b*) | *a* <= *b* |
| lshift | lshift(*a*, *b*) | *a* << *b* |
| lt | lt(*a*, *b*) | *a* < *b* |
| matmul | matmul(*m1*, *m2*) | *m1* @ *m2* |
| mod | mod(*a*, *b*) | *a* % *b* |
| mul | mul(*a*, *b*) | *a* * *b* |
| ne | ne(*a*, *b*) | *a* != *b* |
| neg | neg(*a*) | *-a* |
| not_ | not_(*a*) | **not** *a* |
| or_ | or_(*a*, *b*) | *a* &#124; *b* |
| pos | pos(*a*) | +*a* |
| pow | pow(*a*, *b*) | a ** b |
| repeat | repeat(*a*, *b*) | *a* * *b* |
| rshift | rshift(*a*, *b*) | *a* >> *b* |
| setitem | setitem(*a*, *b*, *c*) | *a*[*b*] = *c* |
| setslice | setslice(*a*, *b*, *c*, *d*) | *a*[*b*:*c*] = *d* |
| sub | sub(*a*, *b*) | *a* - *b* |
| truediv | truediv(*a*, *b*) | *a/b* *# 不截断* |
| truth | truth(*a*) | bool(*a*), **非非** a |
| xor | xor(*a*, *b*) | *a* ^ *b* |

operator 模块还提供了其他额外的高阶函数，列在表 16-4 中。其中三个函数，attrgetter、itemgetter 和 methodcaller，返回适合作为命名参数键传递给列表的 sort 方法、内置函数 sorted、min 和 max，以及标准库模块（例如 heapq 和 itertools 中讨论的第八章中的几个函数）的函数的函数。

表 16-4\. operator 模块提供的高阶函数

| attrgetter | attrgetter(*attr*), attrgetter(**attrs*)

返回一个可调用对象*f*，使得*f*(*o*)与 getattr(*o*, *attr*)相同。字符串*attr*可以包含点号(.)，在这种情况下，attrgetter 的可调用结果会重复调用 getattr。例如，operator.attrgetter('a.b')等效于**lambda** *o*: getattr(getattr(*o*, 'a'), 'b')。

当您使用多个参数调用 attrgetter 时，生成的可调用对象将提取每个命名的属性并返回结果值的元组。 |

| itemgetter | itemgetter(*key*), itemgetter(**keys*)

返回一个可调用对象*f*，使得*f*(*o*)与 getitem(*o*, *key*)相同。

当您使用多个参数调用 itemgetter 时，生成的可调用对象将提取每个以此方式键入的项目，并返回结果值的元组。

例如，假设*L*是一个至少三项的子列表列表：您希望基于每个子列表的第三项对*L*进行原地排序，对于具有相同第三项的子列表，则按其第一项排序。这样做的最简单方法是：

```py
L.sort(key=operator.itemgetter(2, 0))
```

|

| length_hint | length_hint(*iterable*, default=0) 用于尝试预分配*iterable*中项目的存储空间。调用对象*iterable*的 __len__ 方法尝试获取准确长度。如果 __len__ 未实现，则 Python 尝试调用*iterable*的 __length_hint__ 方法。如果这也未实现，length_hint 返回给定的默认值。在使用此“提示”助手时出现任何错误可能会导致性能问题，但不会导致静默的错误行为。 |
| --- | --- |
| methodcaller | methodcaller(*methodname*, args...) 返回一个可调用对象*f*，使得*f*(*o*)与 o.*methodname*(args, ...)相同。可选的 args 可以作为位置参数或命名参数传递。 |

# 随机数与伪随机数

标准库的随机模块使用各种分布生成伪随机数。底层均匀伪随机生成器采用强大且流行的[Mersenne Twister 算法](https://oreil.ly/AcAgG)，周期长度为 2¹⁹⁹³⁷-1（巨大！）。

## 随机模块

随机模块的所有函数都是 random.Random 类的一个隐藏全局实例的方法。您可以显式实例化 Random 以获取不共享状态的多个生成器。如果需要在多个线程中使用随机数（线程在 第十五章 中讨论），则建议显式实例化。或者，如果需要更高质量的随机数，请实例化 SystemRandom（有关详细信息，请参阅下一节）。Table 16-5 记录了随机模块提供的最常用函数。

Table 16-5\. 随机模块提供的实用函数

| choice | choice(*seq*) 从非空序列 *seq* 中返回一个随机项。 |
| --- | --- |
| choices | choices(*seq*, *weights*=**None**, *, cum_weights=**None**, k=1) 从非空序列 *seq* 中返回 k 个元素（可重复选择）。默认情况下，元素以相等概率被选择。如果传递了可选的 *weights* 或命名参数 cum_weights（作为浮点数或整数列表），则在选择过程中将按照相应的权重进行加权。cum_weights 参数接受一个浮点数或整数列表，如 itertools.accumulate(*weights*) 返回的那样；例如，如果 *seq* 包含三个项目的 *weights* 为 [1, 2, 1]，则相应的 cum_weights 将为 [1, 3, 4]。（只能指定 *weights* 或 cum_weights 中的一个，并且必须与 *seq* 长度相同。如果使用，cum_weights 和 k 必须作为命名参数给出。） |
| getrandbits | getrandbits(*k*) 返回一个大于等于 0 的整数，具有 *k* 个随机位，类似于 randrange(2 ** *k*)（但更快，并且对于大 *k* 没有问题）。 |
| getstate | getstate() 返回一个可散列且可 pickle 的对象 *S*，表示生成器的当前状态。稍后可以将 *S* 传递给 setstate 函数以恢复生成器的状态。 |
| jumpahead | jumpahead(*n*) 将生成器状态推进，仿佛已生成 *n* 个随机数。这比生成和忽略 *n* 个随机数要快。 |
| randbytes | randbytes(*k*) 3.9+ 生成 *k* 个随机字节。要为安全或加密应用生成字节，请使用 secrets.randbits(*k* * 8)，然后将其返回的整数解包成 *k* 个字节，使用 int.to_bytes(*k*, 'big')。 |
| randint | randint(*start*, *stop*) 返回一个随机整数 *i*，满足 *start* <= *i* <= *stop* 的均匀分布。两个端点都包括在内：在 Python 中这很不自然，因此通常会优先选择 randrange。 |
| random | random() 返回一个来自均匀分布的随机浮点数 *r*，满足 0 <= *r* < 1。 |
| randrange | randrange([*start*,]*stop*[,*step*]) 类似于 choice(range(*start*, *stop*, *step*))，但速度更快。 |
| sample | sample(*seq*, *k*) 返回一个新列表，其中的*k*个项是从*seq*中随机抽取的唯一项。列表是随机排序的，因此它的任何片段都是同样有效的随机样本。*seq*可能包含重复项。在这种情况下，每个项的每次出现都可能被选为样本的候选项，因此样本中也可能包含这些重复项。 |
| seed | seed(*x*=**None**) 初始化生成器状态。*x*可以是任何 int、float、str、bytes 或 bytearray 类型。当*x*为**None**时，在第一次加载 random 模块时，seed 使用当前系统时间（或某些特定于平台的随机源，如果有的话）获取种子。通常情况下，*x*是一个最多为 2²⁵⁶的 int、一个 float 或最多 32 字节大小的 str、bytes 或 bytearray。接受更大的*x*值，但可能产生与较小值相同的生成器状态。seed 在仿真或建模中很有用，用于可重复运行的运行或编写需要可重复随机值序列的测试。 |
| setstate | setstate(*S*) 恢复生成器的状态。*S*必须是先前调用 getstate 的结果（这样的调用可能发生在另一个程序中，或者在本程序的上一次运行中，只要对象*S*已经正确地被传输或保存并恢复）。 |
| shuffle | shuffle(*seq*) 就地对可变序列*seq*进行洗牌。 |
| uniform | uniform(*a, b*) 返回一个从均匀分布中取出的随机浮点数*r*，使得*a* <= *r* < *b*。 |
| ^(a) 如 Python 语言规范定义的那样。特定的 Python 实现可能支持更大的种子值以生成唯一的随机数序列。 |

random 模块还提供了几个其他函数，用于从其他概率分布（Beta、Gamma、指数、高斯、帕累托等）生成伪随机浮点数，内部调用 random.random 作为它们的随机源。详细信息请参见[在线文档](https://oreil.ly/2n8wP)。

## 密码质量随机数：secrets 模块

random 模块提供的伪随机数虽然足以用于仿真和建模，但不具有密码学质量。要获取用于安全和密码学应用的随机数和序列，请使用 secrets 模块中定义的函数。这些函数使用 random.SystemRandom 类，该类又调用 os.urandom。os.urandom 返回从物理随机位源（例如旧版 Linux 上的/dev/urandom 或 Linux 3.17 及以上版本上的 getrandom()系统调用）读取的随机字节。在 Windows 上，os.urandom 使用像 CryptGenRandom API 这样的密码强度源。如果当前系统上没有合适的源，则 os.urandom 会引发 NotImplementedError 异常。

# secrets 函数不能使用已知的种子运行

与 random 模块不同，后者包括一个种子函数来支持生成可重复的随机值序列，secrets 模块没有这样的功能。要编写依赖于 secrets 模块函数生成特定随机值序列的测试，开发人员必须使用自己的模拟版本来模拟这些函数。

secrets 模块提供了 表 16-6 中列出的函数。

表 16-6\. secrets 模块的函数

| choice | choice(*seq*) 从非空序列 *seq* 中随机选择一个项目。 |
| --- | --- |
| randbelow | randbelow(*n*) 返回一个范围在 0 <= *x* < *n* 的随机整数 *x*。 |
| randbits | randbits(*k*) 返回一个具有 *k* 个随机位的整数。 |
| token_bytes | token_bytes(*n*) 返回一个包含 *n* 个随机字节的 bytes 对象。当省略 *n* 时，使用默认值，通常为 32。 |
| token_hex | token_hex(*n*) 返回一个由 *n* 个随机字节的十六进制字符组成的字符串，每字节两个字符。当省略 *n* 时，使用默认值，通常为 32。 |
| token_urlsafe | token_urlsafe(*n*) 返回一个由 *n* 个随机字节的 Base64 编码字符组成的字符串；结果字符串的长度约为 *n* 的 1.3 倍。当省略 *n* 时，使用默认值，通常为 32。 |

Python 的 [在线文档](https://oreil.ly/Yxh4k) 提供了额外的示例和最佳的加密实践。

其他来源的物理随机数可以在线获取，例如来自 [Fourmilab](https://oreil.ly/uNAfT)。

# fractions 模块

fractions 模块提供了一个有理数类 Fraction，其实例可以由一对整数、另一个有理数或字符串构造。Fraction 类实例具有只读属性 numerator 和 denominator。您可以传递一对（可选带符号的）整数作为 *numerator* 和 *denominator*。如果 denominator 是 0，则会引发 ZeroDivisionError。字符串可以是形如 '3.14' 的形式，或者可以包括一个可选带符号的分子，一个斜杠 (/)，和一个分母，例如 '-22/7'。

# Fraction 分数化简至最低项

Fraction 将分数化简至最低项——例如，f = Fraction(226, 452) 创建一个与 Fraction(1, 2) 等效的实例 f。无法从生成的实例中恢复最初传递给 Fraction 的特定分子和分母。

Fraction 还支持从 decimal.Decimal 实例和浮点数构造（后者可能由于浮点数的有限精度而提供您不期望的结果）。以下是使用不同输入使用 Fraction 的一些示例。

```py
>>> `from` fractions `import` Fraction
>>> `from` decimal `import` Decimal
>>> Fraction(1,10)
```

```py
Fraction(1, 10)
```

```py
>>> Fraction(Decimal('0.1'))
```

```py
Fraction(1, 10)
```

```py
>>> Fraction('0.1')
```

```py
Fraction(1, 10)
```

```py
>>> Fraction('1/10')
```

```py
Fraction(1, 10)
```

```py
>>> Fraction(0.1)
```

```py
Fraction(3602879701896397, 36028797018963968)
```

```py
>>> Fraction(-1, 10)
```

```py
Fraction(-1, 10)
```

```py
>>> Fraction(-1,-10)
```

```py
Fraction(1, 10)
```

Fraction 类提供了包括 limit_denominator 在内的方法，允许您创建浮点数的有理数近似——例如，Fraction(0.0999).limit_denominator(10) 返回 Fraction(1, 10)。Fraction 实例是不可变的，可以作为字典的键或集合的成员，以及与其他数字进行算术运算。完整内容请参阅 [在线文档](https://oreil.ly/xyS7U)。

fractions 模块还提供了一个与 math.gcd 完全相同的函数 gcd，详细介绍在表 16-1 中。

# 十进制模块

Python 浮点数是二进制浮点数，通常根据现代计算机硬件上实现的 IEEE 754 标准。关于浮点运算及其问题的出色、简洁、实用介绍可以在 David Goldberg 的论文[“计算机科学家应该知道的浮点运算”](https://oreil.ly/kmCAq)中找到。关于相同问题的针对 Python 的专题论文是 Python 文档中的[教程](https://oreil.ly/0SQ-H)；另一篇出色的摘要（不专注于 Python），Bruce Bush 的“The Perils of Floating Point”，也可在[线上](https://oreil.ly/d8HJE)找到。

经常情况下，特别是对于与货币相关的计算，您可能更喜欢使用*十进制*浮点数。Python 提供了一个符合 IEEE 854 标准的实现，适用于十进制，在标准库模块 decimal 中。该模块有很好的[文档](https://oreil.ly/3np33)：在那里，您可以找到完整的参考资料、适用标准的指针、教程以及关于十进制的宣传。在这里，我们仅涵盖了该模块功能的一个小子集，即模块的最常用部分。

decimal 模块提供了 Decimal 类（其不可变实例是十进制数）、异常类以及处理*算术上下文*的类和函数，该上下文指定精度、四舍五入以及在发生诸如零除以、溢出、下溢等计算异常时引发的异常。在默认上下文中，精度为 28 位小数数字，四舍五入为“银行家舍入”（将结果四舍五入为最接近的可表示十进制数；当结果恰好在两个这样的数中间时，舍入到最后一位是偶数的数），引发异常的异常包括无效操作、零除以及溢出。

要构建十进制数，请使用一个参数调用 Decimal：一个 int、float、str 或元组。如果以浮点数开始，Python 会无损地将其转换为精确的十进制等效值（可能需要 53 位或更多的精度）：

```py
>>> `from` decimal `import` Decimal
>>> df = Decimal(0.1)
>>> df
```

```py
Decimal('0.1000000000000000055511151231257827021181583404541015625')
```

如果这不是您想要的行为，则可以将浮点数作为 str 传递；例如：

```py
>>> ds = Decimal(str(0.1))  *`# or, more directly, Decimal('0.1')`*
>>> ds
```

```py
Decimal('0.1')
```

您可以轻松地编写一个工厂函数，以便通过十进制进行交互式实验：

```py
`def` dfs(x):
    `return` Decimal(str(x))
```

现在 dfs(0.1)与 Decimal(str(0.1))或 Decimal('0.1')完全相同，但更简洁和更方便书写。

或者，您可以使用 Decimal 的 quantize 方法来通过指定的有效数字四舍五入一个浮点数以构造一个新的十进制数：

```py
>>> dq = Decimal(0.1).quantize(Decimal('.00'))
>>> dq
```

```py
Decimal('0.10')
```

如果您以元组开始，则需要提供三个参数：符号（0 表示正数，1 表示负数），数字的元组和整数指数：

```py
>>> pidigits = (3, 1, 4, 1, 5)
>>> Decimal((1, pidigits, -4))
```

```py
Decimal('-3.1415')
```

一旦你有 Decimal 的实例，你可以对它们进行比较，包括与浮点数的比较（使用 math.isclose 进行比较）；对它们进行 pickle 和 unpickle 操作；并将它们用作字典的键和集合的成员。你还可以在它们之间进行算术运算，以及与整数进行运算，但不能与浮点数进行运算（以避免结果中意外丢失精度），如下所示：

```py
>>> `import` math
>>> `from` decimal `import` Decimal
>>> a = 1.1
>>> d = Decimal('1.1')
>>> a == d
```

```py
False
```

```py
>>> math.isclose(a, d)
```

```py
True
```

```py
>>> a + d
```

```py
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'float' and 
'decimal.Decimal'
```

```py
>>> d + Decimal(a) *`# new decimal constructed from 'a'`*
```

```py
Decimal('2.200000000000000088817841970')
```

```py
>>> d + Decimal(str(a)) *`# convert 'a' to decimal with str(a)`*
```

```py
Decimal('2.20')
```

[在线文档](https://oreil.ly/MygnC)包括有用的[配方](https://oreil.ly/9KnLa)，用于货币格式化、一些三角函数以及常见问题解答。

# 数组处理

大多数语言称为数组或向量的内容，你可以使用列表（在 “列表” 中介绍）或者使用数组标准库模块（在后续小节中介绍）。你可以使用循环、索引和切片、列表推导式、迭代器、生成器和生成表达式（在 第三章 中介绍）来操作数组；使用诸如 map、reduce 和 filter 等内置函数（在 “内置函数” 中介绍）；以及标准库模块如 itertools（在 “itertools 模块” 中介绍）。如果你只需要一个简单类型的轻量级、一维数组的实例，请使用数组。然而，要处理大量的数字数组，这些函数可能比 NumPy 和 SciPy 等第三方扩展更慢、不太方便（在 “用于数值数组计算的扩展” 中介绍）。在进行数据分析和建模时，建议使用 Pandas，它是建立在 NumPy 之上的（但本书未讨论），可能更合适。

## 数组模块

数组模块提供了一种类型，也称为数组，其实例是可变序列，类似于列表。数组 *a* 是一个一维序列，其项可以是字符，或者仅限于你创建 *a* 时指定的某种具体数值类型。数组的构造函数为：

| array | **类** array(*typecode, init*='', /) 创建并返回一个具有给定 *typecode* 的数组对象 *a*。*init* 可以是一个字符串（字节串，除了 *typecode* 为 'u' 外），其长度是 itemsize 的倍数：字符串的字节直接初始化 *a* 的项。另外，*init* 可以是可迭代的对象（当 *typecode* 为 'u' 时是字符，否则是数字）：迭代对象的每一项初始化 *a* 的一项。 |
| --- | --- |

array.array 的优势在于，与列表相比，当你需要保存一系列对象，而这些对象都是同一类型（数值或字符）时，它可以节省内存。数组对象 *a* 在创建时具有一个字符的只读属性 *a*.typecode，用于指定 *a* 的项的类型。表 16-7 展示了数组的可能 typecode 值。

表 16-7\. 数组模块的类型代码

| typecode | C 类型 | Python 类型 | 最小大小 |
| --- | --- | --- | --- |
| 'b' | char | int | 1 字节 |
| 'B' | 无符号 char | int | 1 字节 |
| 'u' | Unicode 字符 | 字符串（长度为 1） | 参见注释 |
| 'h' | short | int | 2 bytes |
| 'H' | unsigned short | int | 2 bytes |
| 'i' | int | int | 2 bytes |
| 'I' | unsigned int | int | 2 bytes |
| 'l' | long | int | 4 bytes |
| 'L' | unsigned long | int | 4 bytes |
| 'q' | long long | int | 8 bytes |
| 'Q' | unsigned long long | int | 8 bytes |
| 'f' | float | float | 4 bytes |
| 'd' | double | float | 8 bytes |

# 类型码 'u' 的最小大小

'u' 在一些平台（特别是 Windows）上的项大小为 2，在几乎所有其他平台上为 4。可以通过使用 array.array('u').itemsize 检查 Python 解释器的构建类型。

数组 *a* 中每个项的字节大小可能大于最小值，取决于机器的体系结构，并且可作为只读属性 *a*.itemsize 获取。

Array 对象暴露了所有可变序列的方法和操作（如“序列操作”中所述），但不包括排序。使用 + 或 += 进行连接以及切片赋值，要求两个操作数都是具有相同类型码的数组；相反，*a*.extend 的参数可以是任何可迭代对象，其项可被 *a* 接受。除了可变序列的方法（append、extend、insert、pop 等），数组对象 *a* 还公开了表 16-8 中列出的方法和属性。

表 16-8\. 数组对象 *a* 的方法和属性

| buffer_info | *a*.buffer_info() 返回一个二元元组（*address*，*array_length*），其中 *array_length* 是 *a* 中可以存储的项数。*a* 的大小（以字节为单位）为 *a*.buffer_info()[1] * *a*.itemsize。 |
| --- | --- |
| byteswap | *a*.byteswap() 交换 *a* 中每个项的字节顺序。 |
| frombytes | *a*.frombytes(*s*) 将字节串 *s*（以机器值解释）附加到 *a*。*s* 的长度必须正好是 *a*.itemsize 的整数倍。 |
| fromfile | *a*.fromfile(*f*, *n*) 从文件对象 *f* 中读取 *n* 个项（以机器值形式），并将这些项附加到 *a*。*f* 应该以二进制模式打开进行读取——通常是 'rb' 模式（参见“使用 open 创建文件对象”）。当 *f* 中的项少于 *n* 时，fromfile 在附加可用项后引发 EOFError（因此，在您的应用程序中如果可以，请务必捕获此异常！）。 |
| fromlist | *a*.fromlist(*L*) 将列表 *L* 的所有项附加到 *a*。 |
| fromunicode | *a*.fromunicode(*s*) 将字符串 *s* 的所有字符附加到 *a*。*a* 必须具有类型码 'u'；否则，Python 将引发 ValueError。 |
| itemsize | *a*.itemsize 返回 *a* 中每个项的字节大小。 |
| tobytes | *a*.tobytes() tobytes 返回数组 *a* 中项的字节表示。对于任何 *a*，len(*a*.tobytes()) == len(*a*)**a*.itemsize。*f*.write(*a*.tobytes()) 等同于 *a*.tofile(*f*)。 |
| tofile | *a*.tofile(*f*) 将 *a* 的所有项目作为机器值写入文件对象 *f*。请注意 *f* 应该以二进制写入模式打开，例如使用模式 'wb'。 |
| tolist | *a*.tolist() 创建并返回一个与 *a* 中项目相同的列表对象，类似于 list(*a*)。 |
| tounicode | *a*.tounicode() 创建并返回一个与 *a* 中项目相同的字符串，类似于 ''.join(*a*)。*a* 必须有类型码 'u'，否则 Python 会引发 ValueError。 |
| typecode | *a*.typecode 返回用于创建 a 的类型码的属性。 |

## 数值数组计算的扩展

正如你所见，Python 在数值处理方面有很好的内置支持。第三方库 [SciPy](https://scipy.org)，以及许多其他库，如 [NumPy](https://numpy.org)，[Matplotlib](https://matplotlib.org)，[SymPy](https://oreil.ly/oVs_S)，[Numba](https://numba.pydata.org)，[Pandas](https://pandas.pydata.org)，[PyTorch](https://pytorch.org)，[CuPy](https://cupy.dev) 和 [TensorFlow](https://www.tensorflow.org)，提供了更多工具。我们在这里介绍 NumPy，然后简要描述了 SciPy 和其他一些库，并提供指向它们文档的链接。

### NumPy

如果你需要一个轻量级的一维数值数组，标准库中的 array 模块可能足够了。如果你的工作涉及科学计算、图像处理、多维数组、线性代数或其他涉及大量数据的应用程序，流行的第三方 NumPy 包能够满足你的需求。在线文档详尽可查看 [这里](https://docs.scipy.org/doc)；Travis Oliphant 的 [*Guide to NumPy*](https://oreil.ly/QA2xJ) 也提供免费的 PDF 下载。²

# NumPy 还是 numpy？

文档中有时将该包称为 NumPy 或 Numpy；但在编码中，该包通常称为 numpy，并使用 **import** numpy **as** np 导入。本节遵循这些约定。

NumPy 提供了 ndarray 类，你可以 [派生子类](https://oreil.ly/FK9qK) 来添加特定需求的功能。一个 ndarray 对象包含 *n* 维同类项（项可能包括异类类型的容器）。每个 ndarray 对象 *a* 有一定数量的维度（也称为 *轴*），称为其 *秩*。一个 *标量*（即单个数）的秩为 0，一个 *向量* 的秩为 1，一个 *矩阵* 的秩为 2，依此类推。ndarray 对象还有一个 *shape* 属性，可以通过属性 shape 访问。例如，对于一个有 2 列 3 行的矩阵 *m*，*m*.shape 是 (3,2)。

NumPy 支持比 Python 更广泛的 [数值类型](https://oreil.ly/HPxtV)（dtype 实例）；默认数值类型为 bool_（1 字节）、int_（根据平台为 int64 或 int32）、float_（float64 的简写）和 complex_（complex128 的简写）。

### 创建 NumPy 数组

有几种方式可以在 NumPy 中创建数组。最常见的包括：

+   使用工厂函数 np.array，从序列（通常是嵌套序列）中进行数组创建，可以进行*type inference*或显式指定*dtype*

+   使用工厂函数 np.zeros、np.ones 或 np.empty，默认使用*dtype*为 float64

+   使用工厂函数 np.indices，默认使用*dtype*为 int64

+   使用工厂函数 np.random.uniform、np.random.normal、np.random.binomial 等，默认使用*dtype*为 float64

+   使用工厂函数 np.arange（通常是*start*、*stop*、*stride*）、或者使用工厂函数 np.linspace（*start*、*stop*、*quantity*）以获得更好的浮点行为

+   通过使用其他 np 函数从文件中读取数据（例如，使用 np.genfromtxt 读取 CSV 文件）

这里是使用刚才描述的各种技术创建数组的一些示例：

```py
>>> `import` numpy `as` np
>>> np.array([1, 2, 3, 4])  *`# from a Python list`*
```

```py
array([1, 2, 3, 4])
```

```py
>>> np.array(5, 6, 7)  *`# a common error: passing items separately (they`*
                       *`# must be passed as a sequence, e.g. a list)`*
```

```py
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: array() takes from 1 to 2 positional arguments, 3 were given
```

```py
>>> s = 'alph', 'abet'  *`# a tuple of two strings`*
>>> np.array(s)
```

```py
array(['alph', 'abet'], dtype='<U4')
```

```py
>>> t = [(1,2), (3,4), (0,1)]  *`# a list of tuples`*
>>> np.array(t, dtype='float64')  *`# explicit type designation`*
```

```py
array([[1., 2.],
 [3., 4.],
 [0., 1.]])
```

```py
>>> x = np.array(1.2, dtype=np.float16)  *`# a scalar`*
>>> x.shape
```

```py
()
```

```py
>>> x.max()
```

```py
1.2
```

```py
>>> np.zeros(3)  *`# shape defaults to a vector`*
```

```py
array([0., 0., 0.])
```

```py
>>> np.ones((2,2))  *`# with shape specified`*
```

```py
array([[1., 1.],
[1., 1.]])
```

```py
>>> np.empty(9)  *`# arbitrary float64s`*
```

```py
array([ 6.17779239e-31, -1.23555848e-30,  3.08889620e-31,
       -1.23555848e-30,  2.68733969e-30, -8.34001973e-31,  
	    3.08889620e-31, -8.34001973e-31,  4.78778910e-31])
```

```py
>>> np.indices((3,3))
```

```py
array([[[0, 0, 0],
 [1, 1, 1],
 [2, 2, 2]],

 [[0, 1, 2],
 [0, 1, 2],
 [0, 1, 2]]])
```

```py
>>> np.arange(0, 10, 2)  *`# upper bound excluded`*
```

```py
array([0, 2, 4, 6, 8])
```

```py
>>> np.linspace(0, 1, 5)  *`# default: endpoint included`*
```

```py
array([0\.  , 0.25, 0.5 , 0.75, 1\.  ])
```

```py
>>> np.linspace(0, 1, 5, endpoint=False)  *`# endpoint not included`*
```

```py
array([0\. , 0.2, 0.4, 0.6, 0.8])
```

```py
>>> np.genfromtxt(io.BytesIO(b'1 2 3\n4 5 6'))  *`# using a pseudo-file`*
```

```py
array([[1., 2., 3.],
 [4., 5., 6.]])
```

```py
>>> `with` open('x.csv', 'wb') as f:
...     f.write(b'2,4,6\n1,3,5')
...
```

```py
11
```

```py
>>> np.genfromtxt('x.csv', delimiter=',')  *`# using an actual CSV file`*
```

```py
array([[2., 4., 6.],
 [1., 3., 5.]])
```

### 形状、索引和切片

每个 ndarray 对象*a*都有一个属性*a*.shape，它是一个整数元组。len(*a*.shape)是*a*的秩；例如，一个一维数字数组（也称为*vector*）的秩为 1，*a*.shape 只有一个项目。更一般地，*a*.shape 的每个项目是*a*相应维度的长度。*a*的元素数量，称为其*size*，是*a*.shape 所有项目的乘积（也可以作为属性*a*.size 获得）。*a*的每个维度也称为一个*axis*。轴索引从 0 开始，如 Python 中通常所见。负轴索引是允许的，并且从右边计数，因此-1 是最后（最右边）的轴。

每个数组*a*（除了标量，即秩为 0 的数组）都是 Python 序列。*a*的每个项*a*[*i*]是*a*的子数组，意味着它的秩比*a*低一级：*a*[*i*].shape == *a*.shape[1:]。例如，如果*a*是一个二维矩阵（*a*的秩为 2），对于任何有效的索引*i*，*a*[*i*]是*a*的一个一维子数组，对应于矩阵的一行。当*a*的秩为 1 或 0 时，*a*的项是*a*的元素（对于秩为 0 的数组，只有一个元素）。由于*a*是一个序列，您可以使用常规索引语法对*a*进行索引以访问或更改*a*的项。请注意，*a*的项是*a*的子数组；只有对于秩为 1 或 0 的数组，数组的*items*才与数组的*elements*相同。

与任何其他序列一样，您也可以对*a*进行*slice*操作。在*b* = *a*[*i*:*j*]之后，*b*与*a*具有相同的秩，且*b*.shape 等于*a*.shape，除了*b*.shape[0]是切片*a*[*i*:*j*]的长度（即当*a*.shape[0] > *j* >= *i* >= 0 时，切片的长度为*j* - *i*，如“切片序列”中所述）。

一旦您有了数组*a*，您可以调用*a*.reshape（或等效地，使用*a*作为第一个参数调用 np.reshape）。结果的形状必须与*a*.size 相匹配：当*a*.size 为 12 时，您可以调用*a*.reshape(3, 4)或*a*.reshape(2, 6)，但*a*.reshape(2, 5)会引发 ValueError。请注意，reshape 不会在原地工作：您必须显式地绑定或重新绑定数组，例如，*a* = *a*.reshape(*i*, *j*)或*b* = *a*.reshape(*i*, *j*)。

你也可以像处理任何其他序列一样，使用 **for** 对（非标量）*a* 进行循环。例如，这样：

```py
`for` x `in` a:
    process(x)
```

意思与以下相同：

```py
`for` _ `in` range(len(*`a`*)):
    x = *`a`*[_]
    process(*`x`*)
```

在这些示例中，循环中的每个 *x* 都是 *a* 的子数组。例如，如果 *a* 是二维矩阵，则在任何一个循环中，每个 *x* 都是对应于矩阵行的一维子数组。

你也可以通过元组对 *a* 进行索引或切片。例如，当 *a* 的秩 >= 2 时，你可以将 *a*[*i*][*j*] 写为 *a*[*i*, *j*]，用于重新绑定和访问；元组索引更快更方便。*不要在括号内*使用括号来指示你正在通过元组对 *a* 进行索引：只需将索引逐个写出，用逗号分隔。*a*[*i*, *j*] 和 *a*[(*i*, *j*)] 的意思完全相同，但没有括号的形式更易读。

索引是一种切片，其中元组的一个或多个项是切片，或者（每个切片最多一次）是特殊形式 ...（Python 内置的省略号）。... 扩展为尽可能多的全轴切片（:），以“填充”你正在切片的数组的秩。例如，*a*[1,...,2] 当 *a* 的秩是 4 时，相当于 *a*[1,:,:,2]，但当 *a* 的秩是 6 时，相当于 *a*[1,:,:,:,:,2]。

下面的代码段展示了循环、索引和切片：

```py
>>> a = np.arange(8)
>>> a
```

```py
array([0, 1, 2, 3, 4, 5, 6, 7])
```

```py
>>> a = a.reshape(2,4)
>>> a
```

```py
array([[0, 1, 2, 3],
 [4, 5, 6, 7]])
```

```py
>>> print(a[1,2])
```

```py
6
```

```py
>>> a[:,:2]
```

```py
array([[0, 1],
 [4, 5]])
```

```py
>>> for row in a:
...     print(row)
...
```

```py
[0 1 2 3]
[4 5 6 7]
```

```py
>>> for row in a:
...     for col in row[:2]:  *`# first two items in each row`*
...         print(col)
...
```

```py
0
1
4
5
```

### 在 NumPy 中的矩阵操作

正如在“运算符模块”中提到的，NumPy 使用运算符 @ 来实现数组的矩阵乘法。*a1* @ *a2* 就像 np.matmul(*a1*, *a2*)。当两个矩阵是二维的时，它们被视为传统的矩阵。当一个参数是向量时，你可以概念上将其提升为二维数组，方法是根据需要临时添加或预置 1 到其形状。不要使用 @ 来与标量相乘；请使用 *。矩阵还可以与标量以及具有兼容形状的向量和其他矩阵进行加法（使用 +）。矩阵的点积也可使用 np.dot(*a1*, *a2*)。下面是几个这些运算符的简单示例：

```py
>>> a = np.arange(6).reshape(2,3)  *`# a 2D matrix`*
>>> b = np.arange(3)               *`# a vector`*
>>>
>>> a
```

```py
array([[0, 1, 2],
 [3, 4, 5]])
```

```py
>>> a + 1    *`# adding a scalar`*
```

```py
array([[1, 2, 3],
 [4, 5, 6]])
```

```py
>>> a + b    *`# adding a vector`*
```

```py
array([[0, 2, 4],
 [3, 5, 7]])
```

```py
>>> a * 2    *`# multiplying by a scalar`*
```

```py
array([[ 0,  2,  4],
 [ 6,  8, 10]])
```

```py
>>> a * b    *`# multiplying by a vector`*
```

```py
array([[ 0,  1,  4],
 [ 0,  4, 10]])
```

```py
>>> a @ b    *`# matrix-multiplying by a vector`*
```

```py
array([ 5, 14])
```

```py
>>> c = (a*2).reshape(3,2)  *`# using scalar multiplication to create`*
>>> c
```

```py
array([[ 0,  2],
 [ 4,  6],
 [ 8, 10]])
```

```py
>>> a @ c    *`# matrix-multiplying two 2D matrices`*
```

```py
array([[20, 26],
 [56, 80]])
```

NumPy 强大而丰富到足以值得整本书来详细讨论；我们只是浅尝辄止。请参阅 NumPy 的[文档](https://oreil.ly/UceLt)以深入了解其众多特性。

### SciPy

虽然 NumPy 包含用于处理数组的类和函数，但 SciPy 库支持更高级的数值计算。例如，虽然 NumPy 提供了一些线性代数方法，但 SciPy 提供了高级分解方法，并支持更高级的函数，例如允许第二个矩阵参数来解决广义特征值问题。一般来说，当进行高级数值计算时，安装 SciPy 和 NumPy 是个好主意。

[SciPy.org](https://oreil.ly/WO3ON) 也托管了许多其他包的[文档](https://oreil.ly/zf6-O)，这些包与 SciPy 和 NumPy 集成，包括提供 2D 绘图支持的[Matplotlib](https://matplotlib.org)；支持符号数学的[SymPy](https://oreil.ly/fbfld)；强大的交互式控制台和 Web 应用内核[Jupyter Notebook](http://jupyter.org)；以及支持数据分析和建模的[Pandas](https://pandas.pydata.org)。您可能还想看一看用于任意精度的[mpmath](https://mpmath.org)，以及用于更丰富功能的[sagemath](https://www.sagemath.org)。

### 额外的数值处理包

Python 社区在数值处理领域产生了更多的包。其中一些包括：

[Anaconda](https://www.anaconda.com)

一个整合环境，简化了 Pandas、NumPy 和许多相关数值处理、分析和可视化包的安装，并通过其自己的 conda 包安装程序提供包管理。

[gmpy2](https://pypi.org/project/gmpy2)

一个模块³，支持 GMP/MPIR、MPFR 和 MPC 库，以扩展和加速 Python 在多精度算术方面的能力。

[Numba](https://numba.pydata.org)

一个即时编译器，用于将 Numba 装饰的 Python 函数和 NumPy 代码转换为 LLVM。Python 中使用 Numba 编译的数值算法可以接近 C 或 FORTRAN 的速度。

[PyTorch](https://pytorch.org)

一个开源机器学习框架。

[TensorFlow](https://www.tensorflow.org/api_docs/python)

一个全面的机器学习平台，可以在大规模和混合环境中运行，使用数据流图来表示计算、共享状态和状态操作。TensorFlow 支持在集群中跨多台机器进行处理，在单台机器上跨多核 CPU、GPU 和定制 ASIC 进行处理。TensorFlow 的主要 API 使用 Python。

¹ 在技术上被更近期的、非常相似的标准[754-2008](https://oreil.ly/qL5nI)取代，但在实际中仍然有用！

² Python 和 NumPy 项目多年来密切合作，Python 引入了专门为 NumPy 引入的语言特性（如@操作符和扩展切片），尽管这些新颖的语言特性（至今？）尚未在 Python 标准库的任何地方使用。

³ 最初来源于本书的一位作者的工作。
