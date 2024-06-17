# 第五章：迭代器和生成器

当许多有其他语言经验的人开始学习 Python 时，他们对 `for` 循环符号的差异感到惊讶。也就是说，不是写成

```py
*`#` `Other languages`*
for (i=0; i<N; i++) {
    do_work(i);
}
```

他们被介绍了一个名为 `range` 的新函数：

```py
*`# Python`*
for i in range(N):
    do_work(i)
```

在 Python 代码示例中，似乎我们正在调用一个名为 `range` 的函数，它创建了 `for` 循环所需的所有数据。直观地说，这可能是一个耗时的过程——如果我们尝试循环遍历 1 到 100,000,000 的数字，那么我们需要花费大量时间创建该数组！然而，这就是 *生成器* 发挥作用的地方：它们基本上允许我们惰性评估这些函数，因此我们可以使用这些特殊用途函数而不会影响性能。

要理解这个概念，让我们实现一个函数，它通过填充列表和使用生成器来计算多个 Fibonacci 数字：

```py
def fibonacci_list(num_items):
    numbers = []
    a, b = 0, 1
    while len(numbers) < num_items:
        numbers.append(a)
        a, b = b, a+b
    return numbers

def fibonacci_gen(num_items):
    a, b = 0, 1
    while num_items:
        yield a  ![1](img/1.png)
        a, b = b, a+b
        num_items -= 1
```

![1](img/#co_iterators_and_generators_CO1-1)

此函数将会 `yield` 多个值而不是返回一个值。这将使得这个看起来很普通的函数成为一个可以重复轮询下一个可用值的生成器。

首先要注意的是 `fibonacci_list` 实现必须创建并存储所有相关的 Fibonacci 数字列表。因此，如果我们想要获取序列的 10,000 个数字，函数将执行 10,000 次向 `numbers` 列表追加（正如我们在 第三章 中讨论过的，这会带来额外的开销），然后返回它。

另一方面，生成器能够“返回”许多值。每次代码执行到 `yield` 时，函数都会发出其值，当请求另一个值时，函数恢复运行（保持其先前状态）并发出新值。当函数达到结尾时，会抛出 `StopIteration` 异常，表示给定的生成器没有更多的值。因此，尽管这两个函数最终必须执行相同数量的计算，但前面循环的 `fibonacci_list` 版本使用的内存是后者的 10,000 倍（或 `num_items` 倍）。

有了这段代码，我们可以分解使用我们的`fibonacci_list`和`fibonacci_gen`实现的`for`循环。在 Python 中，`for`循环要求我们要循环遍历的对象支持迭代。这意味着我们必须能够从我们想要遍历的对象中创建一个迭代器。为了从几乎任何对象创建迭代器，我们可以使用 Python 内置的`iter`函数。对于列表、元组、字典和集合，这个函数返回对象中的项或键的迭代器。对于更复杂的对象，`iter`返回对象的`__iter__`属性的结果。由于`fibonacci_gen`已经返回一个迭代器，在其上调用`iter`是一个微不足道的操作，它返回原始对象（因此`type(fibonacci_gen(10)) == type(iter(fibonacci_gen(10)))`）。然而，由于`fibonacci_list`返回一个列表，我们必须创建一个新对象，即列表迭代器，它将迭代列表中的所有值。一般来说，一旦创建了迭代器，我们就可以用它调用`next()`函数，获取新的值，直到抛出`StopIteration`异常。这为我们提供了`for`循环的一个良好分解视图，如示例 5-1 所示。

##### 示例 5-1\. Python `for`循环分解

```py
# The Python loop
for i in object:
    do_work(i)

# Is equivalent to
object_iterator = iter(object)
while True:
    try:
        i = next(object_iterator)
    except StopIteration:
        break
    else:
        do_work(i)
```

`for`循环代码显示，在使用`fibonacci_list`时，我们调用`iter`会多做一些工作，而使用`fibonacci_gen`时，我们创建一个生成器，该生成器轻松转换为迭代器（因为它本身就是迭代器！）；然而，对于`fibonacci_list`，我们需要分配一个新列表并预先计算其值，然后仍然必须创建一个迭代器。

更重要的是，预计算`fibonacci_list`列表需要分配足够的空间来存储完整的数据集，并将每个元素设置为正确的值，即使我们始终一次只需一个值。这也使得列表分配变得无用。事实上，这可能使循环无法运行，因为它可能尝试分配比可用内存更多的内存（`fibonacci_list(100_000_000)`将创建一个 3.1 GB 大的列表！）。通过时间结果的比较，我们可以非常明确地看到这一点：

```py
def test_fibonacci_list():
    """
 >>> %timeit test_fibonacci_list()
 332 ms ± 13.1 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

 >>> %memit test_fibonacci_list()
 peak memory: 492.82 MiB, increment: 441.75 MiB
 """
    for i in fibonacci_list(100_000):
        pass

def test_fibonacci_gen():
    """
 >>> %timeit test_fibonacci_gen()
 126 ms ± 905 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

 >>> %memit test_fibonacci_gen()
 peak memory: 51.13 MiB, increment: 0.00 MiB
 """
    for i in fibonacci_gen(100_000):
        pass
```

正如我们所见，生成器版本的速度是列表版本的两倍多，并且不需要测量的内存，而`fibonacci_list`则需要 441 MB。此时可能会认为应该在任何地方使用生成器代替创建列表，但这会带来许多复杂性。

比如说，如果你需要多次引用斐波那契数列，`fibonacci_list`会提供一个预先计算好的数字列表，而`fibonacci_gen`则需要一遍又一遍地重新计算。一般来说，改用生成器而非预先计算的数组需要进行有时并不容易理解的算法改动。¹

###### 注意

在设计代码架构时必须做出的一个重要选择是，您是否要优化 CPU 速度还是内存效率。在某些情况下，使用额外的内存来预先计算值并为将来的引用做好准备会节省整体速度。在其他情况下，内存可能受限，唯一的解决方案是重新计算值，而不是将它们保存在内存中。每个问题都有其对于 CPU/内存权衡的考虑因素。

这种的一个简单示例经常在源代码中看到，就是使用一个发生器来创建一个数字序列，然后使用列表推导来计算结果的长度：

```py
divisible_by_three = len([n for n in fibonacci_gen(100_000) if n % 3 == 0])
```

尽管我们仍在使用`fibonacci_gen`生成斐波那契数列作为一个生成器，但我们随后将所有能被 3 整除的值保存到一个数组中，然后只取该数组的长度，然后丢弃数据。在这个过程中，我们无缘无故地消耗了 86 MB 的数据。[²] 事实上，如果我们对足够长的斐波那契数列执行此操作，由于内存问题，上述代码甚至无法运行，即使计算本身非常简单！

记住，我们可以使用形式为[*`<value>`* `for` *`<item>`* `in` *`<sequence>`* `if` *`<condition>`*`]`的语句创建一个列表推导。这将创建一个所有*`<value>`*项的列表。或者，我们可以使用类似的语法来创建一个生成器的*`<value>`*项，而不是一个带有`(`*`<value>`* `for` *`<item>`* `in` *`<sequence>`* `if` *`<condition>`*`)`的列表。

利用列表推导和生成器推导之间的这种细微差别，我们可以优化上述的`divisible_by_three`代码。然而，生成器没有`length`属性。因此，我们将不得不聪明地处理：

```py
divisible_by_three = sum(1 for n in fibonacci_gen(100_000) if n % 3 == 0)
```

这里，我们有一个发生器，每当遇到能被 3 整除的数字时就发出`1`的值，否则不发出任何值。通过对这个发生器中的所有元素求和，我们本质上与列表推导版本做的事情相同，并且不消耗任何显著的内存。

###### 注意

Python 中许多内置函数都是它们自己的生成器（尽管有时是一种特殊类型的生成器）。例如，`range`返回一个值的生成器，而不是指定范围内实际数字的列表。类似地，`map`、`zip`、`filter`、`reversed`和`enumerate`都根据需要执行计算，并不存储完整的结果。这意味着操作`zip(range(100_000), range(100_000))`将始终在内存中只有两个数字，以便返回其对应的值，而不是预先计算整个范围的结果。

这段代码的两个版本在较小的序列长度上的性能几乎相当，但是生成器版本的内存影响远远小于列表推导的影响。此外，我们将列表版本转换为生成器，因为列表中每个元素的重要性都在于其当前值——无论数字是否能被 3 整除，都没有关系；它的位置在数字列表中或前/后值是什么并不重要。更复杂的函数也可以转换为生成器，但是取决于它们对状态的依赖程度，这可能变得难以做到。

# 无限系列的迭代器

如果我们不计算已知数量的斐波那契数，而是尝试计算所有的呢？

```py
def fibonacci():
    i, j = 0, 1
    while True:
        yield j
        i, j = j, i + j
```

在这段代码中，我们正在做一些以前的`fibonacci_list`代码无法做到的事情：我们正在将无限系列的数字封装到一个函数中。这允许我们从此流中取出尽可能多的值，并在我们的代码认为已经足够时终止。

生成器没有被充分利用的一个原因是其中很多逻辑可以封装在您的逻辑代码中。生成器实际上是一种组织代码并拥有更智能的循环的方式。例如，我们可以用多种方式回答问题“5000 以下有多少个斐波那契数是奇数？”：

```py
def fibonacci_naive():
    i, j = 0, 1
    count = 0
    while j <= 5000:
        if j % 2:
            count += 1
        i, j = j, i + j
    return count

def fibonacci_transform():
    count = 0
    for f in fibonacci():
        if f > 5000:
            break
        if f % 2:
            count += 1
    return count

from itertools import takewhile
def fibonacci_succinct():
    first_5000 = takewhile(lambda x: x <= 5000,
                           fibonacci())
    return sum(1 for x in first_5000
               if x % 2)
```

所有这些方法在运行时特性上都相似（由内存占用和运行时性能测量），但是`fibonacci_transform`函数受益于几个方面。首先，它比`fibonacci_succinct`更冗长，这意味着另一个开发者可以更容易地进行调试和理解。后者主要是为了下一节，我们将在其中涵盖一些使用`itertools`的常见工作流程——虽然该模块可以大大简化许多迭代器的简单操作，但它也可能迅速使 Python 代码变得不太 Pythonic。相反，`fibonacci_naive`一次做多件事情，这隐藏了它实际正在执行的计算！虽然在生成器函数中很明显我们正在迭代斐波那契数，但我们并没有因实际计算而过于繁重。最后，`fibonacci_transform`更具一般性。该函数可以重命名为`num_odd_under_5000`，并通过参数接受生成器，因此可以处理任何系列。

`fibonacci_transform` 和 `fibonacci_succinct` 函数的另一个好处是它们支持计算中有两个阶段的概念：生成数据和转换数据。这些函数明显地对数据执行转换，而 `fibonacci` 函数则生成数据。这种划分增加了额外的清晰度和功能：我们可以将一个转换函数移动到新的数据集上，或者在现有数据上执行多次转换。在创建复杂程序时，这种范式一直很重要；然而，生成器通过使生成器负责创建数据，普通函数负责对生成的数据进行操作，从而清晰地促进了这一点。

# 惰性生成器评估

正如前面提到的，我们通过生成器获得记忆优势的方式是仅处理当前感兴趣的值。在使用生成器进行计算时，我们始终只有当前值，并且不能引用序列中的任何其他项（按这种方式执行的算法通常称为*单通道*或*在线*）。这有时会使得生成器更难使用，但许多模块和函数可以帮助。

主要感兴趣的库是标准库中的`itertools`。它提供许多其他有用的函数，包括这些：

`islice`

允许切片潜在无限生成器

`chain`

将多个生成器链接在一起

`takewhile`

添加一个将结束生成器的条件

`cycle`

通过不断重复使有限生成器变为无限

让我们举一个使用生成器分析大型数据集的例子。假设我们有一个分析例程，对过去 20 年的时间数据进行分析，每秒一条数据，总共有 631,152,000 个数据点！数据存储在文件中，每行一秒，我们无法将整个数据集加载到内存中。因此，如果我们想进行一些简单的异常检测，我们必须使用生成器来节省内存！

问题将是：给定一个形式为“时间戳，值”的数据文件，找出值与正态分布不同的日期。我们首先编写代码，逐行读取文件，并将每行的值输出为 Python 对象。我们还将创建一个`read_fake_data`生成器，生成我们可以用来测试算法的假数据。对于这个函数，我们仍然接受`filename`参数，以保持与`read_data`相同的函数签名；但是，我们将简单地忽略它。这两个函数，如示例 5-2 所示，确实是惰性评估的——我们只有在调用`next()`函数时才会读取文件中的下一行，或生成新的假数据。

##### 示例 5-2\. 惰性读取数据

```py
from random import normalvariate, randint
from itertools import count
from datetime import datetime

def read_data(filename):
    with open(filename) as fd:
        for line in fd:
            data = line.strip().split(',')
            timestamp, value = map(int, data)
            yield datetime.fromtimestamp(timestamp), value

def read_fake_data(filename):
    for timestamp in count():
        #  We insert an anomalous data point approximately once a week
        if randint(0, 7 * 60 * 60 * 24 - 1) == 1:
            value = normalvariate(0, 1)
        else:
            value = 100
        yield datetime.fromtimestamp(timestamp), value
```

现在，我们想创建一个函数，输出出现在同一天的数据组。为此，我们可以使用`itertools`中的`groupby`函数（示例 5-3）。该函数通过接受一个项目序列和一个用于分组这些项目的键来工作。输出是一个生成器，产生元组，其项目是组的键和组中项目的生成器。作为我们的键函数，我们将输出数据记录的日历日期。这个“键”函数可以是任何东西——我们可以按小时、按年或按实际值中的某个属性对数据进行分组。唯一的限制是只有顺序数据才会形成组。因此，如果我们有输入`A A A A B B A A`，并且`groupby`按字母分组，我们将得到三组：`(A, [A, A, A, A])`，`(B, [B, B])`和`(A, [A, A])`。

##### 示例 5-3\. 数据分组

```py
from itertools import groupby

def groupby_day(iterable):
    key = lambda row: row[0].day
    for day, data_group in groupby(iterable, key):
        yield list(data_group)
```

现在进行实际的异常检测。我们在示例 5-4 中通过创建一个函数来完成这个任务，该函数在给定一个数据组时返回其是否符合正态分布（使用`scipy.stats.normaltest`）。我们可以使用`itertools.filterfalse`来仅过滤掉完整数据集中不通过测试的输入。这些输入被认为是异常的。

###### 注意

在示例 5-3 中，我们将`data_group`转换为列表，即使它是以迭代器形式提供给我们的。这是因为`normaltest`函数需要一个类似数组的对象。然而，我们可以编写自己的“一次通过”`normaltest`函数，它可以在数据的单个视图上操作。通过使用[Welford 的在线平均算法](https://oreil.ly/p2g8Q)计算数字的偏度和峰度，我们可以轻松实现这一点。这将通过始终仅在内存中存储数据集的单个值而不是整天的方式来节省更多内存。然而，性能时间回归和开发时间应该考虑进去：将一天的数据存储在内存中是否足以解决这个问题，或者是否需要进一步优化？

##### 示例 5-4\. 基于生成器的异常检测

```py
from scipy.stats import normaltest
from itertools import filterfalse

def is_normal(data, threshold=1e-3):
    _, values = zip(*data)
    k2, p_value = normaltest(values)
    if p_value < threshold:
        return False
    return True

def filter_anomalous_groups(data):
    yield from filterfalse(is_normal, data)
```

最后，我们可以将生成器链结合起来以获取具有异常数据的日期（示例 5-5）。

##### 示例 5-5\. 连接我们的生成器

```py
from itertools import islice

def filter_anomalous_data(data):
    data_group = groupby_day(data)
    yield from filter_anomalous_groups(data_group)

data = read_data(filename)
anomaly_generator = filter_anomalous_data(data)
first_five_anomalies = islice(anomaly_generator, 5)

for data_anomaly in first_five_anomalies:
    start_date = data_anomaly[0][0]
    end_date = data_anomaly[-1][0]
    print(f"Anomaly from {start_date} - {end_date}")
```

```py
# Output of above code using "read_fake_data"
Anomaly from 1970-01-10 00:00:00 - 1970-01-10 23:59:59
Anomaly from 1970-01-17 00:00:00 - 1970-01-17 23:59:59
Anomaly from 1970-01-18 00:00:00 - 1970-01-18 23:59:59
Anomaly from 1970-01-23 00:00:00 - 1970-01-23 23:59:59
Anomaly from 1970-01-29 00:00:00 - 1970-01-29 23:59:59
```

这种方法使我们能够获取异常日的列表，而无需加载整个数据集。只读取足够的数据以生成前五个异常。此外，`anomaly_generator`对象可以进一步阅读以继续检索异常数据。这被称为*惰性评估*——只有明确请求的计算才会执行，这可以显著减少总体运行时间，如果存在早期终止条件的话。

另一个有关以这种方式组织分析的好处是，它使我们能够更轻松地进行更加广泛的计算，而无需重做大部分代码。例如，如果我们想要一个移动窗口为一天而不是按天分块，我们可以在 Example 5-3 中用类似以下方式替换 `groupby_day`：

```py
from datetime import datetime

def groupby_window(data, window_size=3600):
    window = tuple(islice(data, window_size))
    for item in data:
        yield window
        window = window[1:] + (item,)
```

在此版本中，我们还可以非常明确地看到这种方法及其前一种方法的内存保证——它只会将窗口大小的数据作为状态存储（在两种情况下均为一天或 3,600 个数据点）。请注意，`for` 循环检索的第一项是第 `window_size` 个值。这是因为 `data` 是一个迭代器，在前一行我们消耗了前 `window_size` 个值。

最后说明：在 `groupby_window` 函数中，我们不断地创建新的元组，将它们填充数据，并将它们 `yield` 给调用者。我们可以通过使用 `collections` 模块中的 `deque` 对象来大大优化此过程。该对象提供了 `O(1)` 的向右（或末尾）附加和删除操作（而普通列表对于向末尾附加或删除操作是 `O(1)`，对于向列表开头相同操作是 `O(n)`）。使用 `deque` 对象，我们可以将新数据追加到列表的右侧（或末尾），并使用 `deque.popleft()` 从左侧（或开头）删除数据，而无需分配更多空间或执行长达 `O(n)` 的操作。然而，我们必须就地使用 `deque` 对象并销毁先前的滚动窗口视图（参见 “内存分配和就地操作” 了解有关就地操作的更多信息）。唯一的解决方法是在将数据复制到元组之前，将其销毁并返回给调用者，这将消除任何更改的好处！

# 总结

通过使用迭代器来制定我们的异常查找算法，我们可以处理比内存容量更大得多的数据。更重要的是，我们可以比使用列表时更快地执行操作，因为我们避免了所有昂贵的 `append` 操作。

由于迭代器在 Python 中是一种原始类型，这应该始终是尝试减少应用程序内存占用的方法。其好处在于结果是惰性评估的，因此您只处理所需的数据，并且节省内存，因为我们不存储以前的结果，除非显式需要。在 Chapter 11 中，我们将讨论其他可以用于更具体问题的方法，并介绍一些在 RAM 成为问题时看待问题的新方法。

使用迭代器解决问题的另一个好处是，它能够使你的代码准备好在多个 CPU 或多台计算机上使用，正如我们将在第九章和第十章中看到的那样。正如我们在“无限级数的迭代器”中讨论的那样，在使用迭代器时，你必须始终考虑算法运行所必需的各种状态。一旦你弄清楚了如何打包算法运行所需的状态，它在哪里运行就不重要了。我们可以在`multiprocessing`和`ipython`模块中看到这种范式，它们都使用类似于 `map` 的函数来启动并行任务。

¹ 一般来说，*在线* 或 *单遍* 算法非常适合使用生成器。然而，在切换时，你必须确保你的算法仍然可以在没有能够多次引用数据的情况下运行。

² 通过 `%memit len([n for n in fibonacci_gen(100_000) if n % 3 == 0])` 计算。
