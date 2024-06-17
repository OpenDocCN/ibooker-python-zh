# 第六章：使用 while 和 for 循环

> 对于所有的事情，我们辛劳工作，我们辛劳工作，我们所有的努力都被忽视……
> 
> 罗伯特·彭斯，《为了那些，为了那些》

使用 `if`、`elif` 和 `else` 进行测试时，从上到下执行。有时，我们需要执行多次操作。我们需要一个*循环*，而 Python 给了我们两个选择：`while` 和 `for`。

# 使用 while 重复

Python 中最简单的循环机制是 `while`。使用交互式解释器，尝试这个例子，这是一个简单的循环，打印从 1 到 5 的数字：

```py
>>> count = 1
>>> while count <= 5:
...     print(count)
...     count += 1
...
1
2
3
4
5
>>>
```

我们首先将值 `1` 赋给了 `count`。`while` 循环将 `count` 的值与 `5` 进行比较，如果 `count` 小于或等于 `5`，则继续。在循环内部，我们打印了 `count` 的值，然后使用语句 `count += 1` 将其值*增加*了一。Python 返回循环顶部，再次将 `count` 与 `5` 进行比较。此时 `count` 的值为 `2`，因此再次执行 `while` 循环的内容，并将 `count` 增加到 `3`。

这将持续到在循环底部将 `count` 从 `5` 增加到 `6` 为止。在下一次回到顶部时，`count <= 5` 现在为 `False`，`while` 循环结束。Python 继续执行下一行。

## 使用 break 取消

如果您想循环直到某些事情发生，但不确定什么时候会发生，可以使用带有 `break` 语句的*无限循环*。这次，让我们通过 Python 的 `input()` 函数从键盘读取一行输入，然后将其打印为首字母大写。当键入仅包含字母 `q` 的行时，我们跳出循环：

```py
>>> while True:
...     stuff = input("String to capitalize [type q to quit]: ")
...     if stuff == "q":
...         break
...     print(stuff.capitalize())
...
String to capitalize [type q to quit]: test
Test
String to capitalize [type q to quit]: hey, it works
Hey, it works
String to capitalize [type q to quit]: q
>>>
```

## 使用 continue 跳过

有时，您不想中断循环，而只是想因某种原因跳到下一个迭代。这是一个牵强的例子：让我们读取一个整数，如果它是奇数，则打印它的平方，并在它是偶数时跳过。我们甚至添加了一些注释。同样，我们使用 `q` 来停止循环。

```py
>>> while True:
...     value = input("Integer, please [q to quit]: ")
...     if value == 'q':      # quit
...         break
...     number = int(value)
...     if number % 2 == 0:   # an even number
...        continue
...     print(number, "squared is", number*number)
...
Integer, please [q to quit]: 1
1 squared is 1
Integer, please [q to quit]: 2
Integer, please [q to quit]: 3
3 squared is 9
Integer, please [q to quit]: 4
Integer, please [q to quit]: 5
5 squared is 25
Integer, please [q to quit]: q
>>>
```

## 使用 break 检查与 else 一起使用

如果 `while` 循环正常结束（没有调用 `break`），控制将传递给可选的 `else`。当您已编写了一个 `while` 循环来检查某些内容，并在找到时立即中断时，您会使用它。如果 `while` 循环完成但未找到对象，则会运行 `else`：

```py
>>> numbers = [1, 3, 5]
>>> position = 0
>>> while position < len(numbers):
...     number = numbers[position]
...     if number % 2 == 0:
...         print('Found even number', number)
...         break
...     position += 1
... else:  # break not called
...     print('No even number found')
...
No even number found
```

###### 注意

对于`else`的这种用法可能看起来不直观。将其视为*中断检查器*。

# 使用 for 和 in 进行迭代

Python 经常使用*迭代器*，有很好的理由。它们使您能够遍历数据结构，而无需知道其大小或实现方式。您甚至可以迭代即时创建的数据，允许处理否则无法一次性放入计算机内存中的数据*流*。

要展示迭代，我们需要一个可以迭代的对象。你已经在第五章看到了字符串，但还没有详细了解其他*可迭代对象*，比如列表和元组（见第七章）或字典（见第八章）。这里我将展示两种遍历字符串的方法，并在它们各自的章节中展示其他类型的迭代。

在 Python 中，通过以下方式逐步遍历字符串是合法的：

```py
>>> word = 'thud'
>>> offset = 0
>>> while offset < len(word):
...     print(word[offset])
...     offset += 1
...
t
h
u
d
```

但有一个更好、更符合 Python 风格的方法：

```py
>>> for letter in word:
...     print(letter)
...
t
h
u
d
```

字符串迭代每次产生一个字符。

## 通过`break`取消

`for`循环中的`break`会跳出循环，就像在`while`循环中一样：

```py
>>> word = 'thud'
>>> for letter in word:
...     if letter == 'u':
...         break
...     print(letter)
...
t
h
```

## 使用`continue`跳过

在`for`循环中插入`continue`会跳到下一个迭代，就像在`while`循环中一样。

## 检查`break`与`else`的用法

类似于`while`，`for`也有一个可选的`else`语句，用于检查`for`是否正常完成。如果没有调用`break`，则会执行`else`语句。

当你希望确认前一个`for`循环是否完全执行而不是因为`break`而提前停止时，这是非常有用的：

```py
>>> word = 'thud'
>>> for letter in word:
...     if letter == 'x':
...         print("Eek! An 'x'!")
...         break
...     print(letter)
... else:
...     print("No 'x' in there.")
...
t
h
u
d
No 'x' in there.
```

###### 注意

与`while`一样，使用`for`和`else`可能看起来不直观。如果你把`for`看作在寻找某些东西，那么当你没有找到时，`else`会被调用。如果想要在没有`else`的情况下达到相同的效果，可以使用某个变量来指示在`for`循环中是否找到了想要的内容。

## 使用`range()`生成数字序列

`range()`函数返回在指定范围内的一系列数字。无需首先创建和存储大数据结构（如列表或元组），就可以创建大范围，避免占用计算机所有内存并导致程序崩溃。

使用`range()`与使用切片类似：`range(` *`start`*, *`stop`*, *`step`* `)`。如果省略*`start`*，范围将从`0`开始。与切片一样，创建的最后一个值将恰好在*`stop`*之前。*`step`*的默认值是`1`，但可以使用`-1`向后遍历。

类似于`zip()`，`range()`返回一个*可迭代对象*，因此你需要用`for ... in`逐个遍历其值，或者将该对象转换为像列表这样的序列。让我们创建一个范围为`0, 1, 2`的示例：

```py
>>> for x in range(0,3):
...     print(x)
...
0
1
2
>>> list( range(0, 3) )
[0, 1, 2]
```

下面是如何从`2`到`0`生成一个范围：

```py
>>> for x in range(2, -1, -1):
...     print(x)
...
2
1
0
>>> list( range(2, -1, -1) )
[2, 1, 0]
```

以下片段使用步长为`2`来获取从`0`到`10`的偶数：

```py
>>> list( range(0, 11, 2) )
[0, 2, 4, 6, 8, 10]
```

# 其他迭代器

第十四章展示了如何迭代文件。在第十章，你可以看到如何启用对自定义对象的迭代。此外，第十一章讨论了`itertools`——一个带有许多有用快捷方式的标准 Python 模块。

# 即将到来

将各个数据链入*列表*和*元组*中。

# 待完成的事情

6.1 使用`for`循环打印列表`[3, 2, 1, 0]`的值。

将值`7`赋给变量`guess_me`，并将值`1`赋给变量`number`。编写一个`while`循环，比较`number`与`guess_me`。如果`number`小于`guess me`，则打印`'too low'`。如果`number`等于`guess_me`，则打印`'found it!'`，然后退出循环。如果`number`大于`guess_me`，则打印`'oops'`，然后退出循环。在循环结束时增加`number`。

将值`5`赋给变量`guess_me`。使用`for`循环迭代名为`number`的变量在`range(10)`上。如果`number`小于`guess_me`，则打印`'too low'`。如果它等于`guess_me`，则打印`'found it!'`，然后退出 for 循环。如果`number`大于`guess_me`，则打印`'oops'`，然后退出循环。
