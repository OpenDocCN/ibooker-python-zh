# 第十四章：执行定制

Python 公开、支持并记录其许多内部机制。这可能帮助您在高级水平上理解 Python，并允许您将自己的代码连接到这些 Python 机制中，以某种程度上控制它们。例如，“Python 内置函数”介绍了 Python 安排内置函数可见的方式。本章还涵盖了一些其他高级 Python 技术，包括站点定制、终止函数、动态执行、处理内部类型和垃圾回收。我们将在第十五章中讨论使用多线程和进程控制执行的其他问题；第十七章涵盖了与测试、调试和性能分析相关的特定问题。

# 站点定制

Python 提供了一个特定的“钩子”来让每个站点在每次运行开始时定制 Python 行为的某些方面。Python 在主脚本之前加载标准模块 site。如果使用**-S**选项运行 Python，则不加载 site。**-S**允许更快的启动，但会为主脚本增加初始化任务。site 的主要任务是将 sys.path 放置在标准形式中（绝对路径，无重复项），包括根据环境变量、虚拟环境和在 sys.path 中找到的每个*.pth*文件的指示。

其次，如果启动的会话是交互式的，则 site 会添加几个方便的内置函数（例如 exit、copyright 等），并且如果启用了 readline，则配置 Tab 键的自动完成功能。

在任何正常的 Python 安装中，安装过程设置了一切以确保 site 的工作足以让 Python 程序和交互式会话“正常”运行，即与安装了该版本 Python 的任何其他系统上的运行方式相同。在特殊情况下，如果作为系统管理员（或在等效角色，例如已将 Python 安装在其主目录以供个人使用的用户）认为绝对需要进行一些定制，则在名为*sitecustomize.py*的新文件中执行此操作（在与*site.py*相同的目录中创建它）。

# 避免修改 site.py

我们强烈建议您不要修改执行基础定制的*site.py*文件。这样做可能会导致 Python 在您的系统上的行为与其他地方不同。无论如何，*site.py*文件每次更新 Python 安装时都会被覆盖，您的修改将会丢失。

在*sitecustomize.py*存在的罕见情况下，它通常的作用是将更多字典添加到 sys.path 中——执行此任务的最佳方法是让*sitecustomize.py* **import** site，然后调用 site.addsitedir(*path_to_a_dir*)。

# 终止函数

`atexit` 模块允许你注册终止函数（即，在程序终止时按 LIFO 顺序调用的函数）。终止函数类似于由 **try**/**finally** 或 **with** 建立的清理处理程序。然而，终止函数是全局注册的，在整个程序结束时调用，而清理处理程序是在特定 **try** 子句或 **with** 语句结束时调用的。终止函数和清理处理程序在程序正常或异常终止时都会被调用，但不会在通过调用 `os._exit` 终止程序时调用（所以通常调用 `sys.exit`）。`atexit` 模块提供了一个名为 `register` 的函数，接受 *func*、**args** 和 **kwds** 作为参数，并确保在程序终止时调用 *func*（**args**，***kwds***）。

# 动态执行和 `exec`

Python 的内置函数 `exec` 可以在程序运行时执行你读取、生成或以其他方式获取的代码。`exec` 动态执行一个语句或一组语句。其语法如下：

```py
exec(*`code`*, *`globals`*=`None`, *`locals`*=`None`, /)
```

*code* 可以是 str、bytes、bytearray 或 code 对象。*globals* 是一个字典，*locals* 可以是任何映射。

如果同时传递 *globals* 和 *locals*，它们分别是代码运行的全局和局部命名空间。如果只传递 *globals*，`exec` 将同时使用 *globals* 作为全局和局部命名空间。如果两者都不传递，*code* 将在当前作用域中运行。

# 永远不要在当前作用域中运行 `exec`

在当前作用域中运行 `exec` 是一个特别糟糕的主意：它可以绑定、重新绑定或解绑任何全局名称。为了保持控制，请只在特定、显式的字典中使用 `exec`，如果必须的话。

## 避免使用 `exec`

Python 中经常被问到的一个问题是“如何设置一个我刚刚读取或构建的变量的名称？”确实，对于一个 *global* 变量，`exec` 允许这样做，但是为此目的使用 `exec` 是一个非常糟糕的主意。例如，如果变量名是 *varname*，你可能会考虑使用：

```py
exec(*`varname`* + ' = 23')
```

*不要这样做*。在当前作用域中这样的 `exec` 会使你失去命名空间的控制，导致极难找到的错误，并使你的程序难以理解。将你需要动态设置的“变量”保存为字典条目（例如，*mydict*）而不是实际变量。然后可以考虑使用：

```py
exec(*`varname`*+'=23', *`mydict`*) *`# Still a bad idea`*
```

虽然这种方式不如前面的例子那么糟糕，但仍然不是一个好主意。将这些“变量”保存为字典条目意味着你不需要使用 `exec` 来设置它们！只需编写代码：

```py
mydict[varname] = 23
```

这样，你的程序会更清晰、直接、优雅且更快。有一些情况下确实可以使用 `exec`，但这些情况非常罕见：最好是使用显式字典。

# 努力避免 `exec`

只有在确实不可或缺时才使用 exec，这种情况*极为*罕见。通常最好避免 exec，并选择更具体、更受控制的机制：exec 会削弱您对代码命名空间的控制，可能损害程序的性能，并使您面临许多难以发现的错误和巨大的安全风险。

## 表达式

exec 可以执行表达式，因为任何表达式也都是有效的语句（称为*表达式语句*）。但是，Python 会忽略表达式语句返回的值。要评估表达式并获取表达式的值，请使用内置函数 eval，如表格 8-2 中所述。（但请注意，exec 的几乎所有安全风险警告同样适用于 eval。）

## 编译和代码对象

要使一个代码对象用于 exec，调用内置函数 compile 并将最后一个参数设置为'exec'（如表格 8-2 中所述）。

一个代码对象*c*展示了许多有趣的只读属性，它们的名称都以'co_'开头，比如在表格 14-1 中列出的那些。

表格 14-1\. 代码对象的只读属性

| co_argcount | *c*所代表的函数的参数个数（当*c*不是函数的代码对象而是直接由 compile 构建时为 0） |
| --- | --- |
| co_code | 一个字节对象，包含*c*的字节码 |
| co_consts | *c*中使用的常量的元组 |
| co_filename | *c*编译生成的文件名（当*c*是这种方式构建时为 compile 的第二个参数的字符串） |
| c⁠o⁠_⁠f⁠i⁠r⁠s⁠t​l⁠i⁠n⁠e⁠n⁠o | 源代码的初始行号（位于由 co_filename 命名的文件中），用于编译生成*c*，如果*c*是通过编译文件构建的话 |
| co_name | *c*所代表的函数的名称（当*c*不是函数的代码对象而是直接由 compile 构建时为'<module>'） |
| co_names | *c*中使用的所有标识符的元组 |
| co_varnames | *c*中局部变量标识符的元组，以参数名称开头 |

这些属性大多仅用于调试目的，但有些可能有助于高级内省，正如本节后面所示的例子。

如果您从包含一个或多个语句的字符串开始，首先对字符串使用 compile，然后在生成的代码对象上调用 exec—这比直接将字符串传递给 exec 来编译和执行要好一些。 这种分离允许您单独检查语法错误和执行时错误。 您通常可以安排事务以便字符串只编译一次，而代码对象重复执行，这样可以加快速度。 eval 也可以从这种分离中受益。 此外，编译步骤本质上是安全的（如果在您不完全信任的代码上执行 exec 和 eval 非常危险），您可以在执行代码对象之前检查它，以减少风险（尽管风险永远不会完全为零）。

正如 表 14-1 中所述，代码对象具有只读属性 co_names，该属性是代码中使用的名称的元组。 例如，假设您希望用户输入仅包含文字常量和操作符的表达式—不包含函数调用或其他名称。 在评估表达式之前，您可以检查用户输入的字符串是否满足这些约束：

```py
`def` safer_eval(s):
    code = compile(s, '<user-entered string>', 'eval')
    `if` code.co_names:
        `raise` ValueError(
            f'Names {code.co_names!r} not allowed in expression {s!r}')
    `return` eval(code)
```

函数 safer_eval 评估作为参数传递的表达式 *s*，仅当字符串是语法上有效的表达式（否则，compile 会引发 SyntaxError），且完全不包含任何名称（否则，safer_eval 明确引发 ValueError）。 （这类似于标准库函数 ast.literal_eval，详见 “标准输入”，但更为强大，因为它允许使用操作符。）

了解代码即将访问的名称有时可能有助于优化您需要传递给 exec 或 eval 作为命名空间的字典的准备工作。 由于您只需要为这些名称提供值，因此您可以通过不准备其他条目来节省工作。 例如，假设您的应用程序动态接受来自用户的代码，并且约定以 data_ 开头的变量名引用存储在子目录 *data* 中的文件，用户编写的代码不需要显式读取。 用户编写的代码反过来可能会计算并将结果留在以 result_ 开头的全局变量中，您的应用程序将这些结果作为文件写回 *data* 子目录。 由于这种约定，稍后您可以将数据移动到其他位置（例如，到数据库中的 BLOBs，而不是子目录中的文件），用户编写的代码不会受到影响。 这是您可能如何有效实现这些约定的方法：

```py
`def` exec_with_data(user_code_string):
    user_code = compile(user_code_string, '<user code>', 'exec')
    datadict = {}
    `for` name `in` user_code.co_names:
        `if` name.startswith('data_'):
            `with` open(f'data/{name[5:]}', 'rb') `as` datafile:
                datadict[name] = datafile.read()
        `elif` name.startswith('result_'):
            `pass`  *`` # user code assigns to variables named `result_...` ``*
        `else`:
            `raise` ValueError(f'invalid variable name {name!r}')
    exec(user_code, datadict)
    `for` name `in` datadict:
        `if` name.startswith('result_'):
            `with` open(f'data/{name[7:]}', 'wb') `as` datafile:
                datafile.write(datadict[name])
```

## 永远不要执行或评估不受信任的代码。

早期版本的 Python 提供了旨在减轻使用 `exec` 和 `eval` 风险的工具，称为“受限执行”，但这些工具从未完全安全，无法抵御有能力的黑客的狡猾攻击。最近的 Python 版本已经放弃了这些工具，以避免为用户提供不合理的安全感。如果需要防范此类攻击，请利用操作系统提供的保护机制：在一个单独的进程中运行不受信任的代码，并尽可能限制其权限（研究操作系统提供的用于此目的的机制，如 chroot、setuid 和 jail；在 Windows 上，可以尝试第三方商业附加组件 [WinJail](https://oreil.ly/a4hf4)，或者如果你是容器安全化专家，可以在一个单独且高度受限的虚拟机或容器中运行不受信任的代码）。为了防范服务拒绝攻击，主进程应监控单独的进程，并在资源消耗过多时终止后者。进程在 “运行其他程序” 中有详细描述。

# `exec` 和 `eval` 在不受信任的代码中是不安全的。

在上一节中定义的 `exec_with_data` 函数对不受信任的代码根本不安全：如果将作为参数 `user_code` 传递给它的字符串来自于你不能完全信任的方式，那么它可能会造成无法估量的损害。不幸的是，这几乎适用于任何使用 `exec` 或 `eval` 的情况，除非你能对要执行或评估的代码设置极其严格和完全可检查的限制，就像 `safer_eval` 函数的情况一样。

# 内部类型

本节描述的一些内部 Python 对象使用起来很困难，事实上并不是大多数情况下你应该使用的。正确地使用这些对象并产生良好效果需要一些研究你的 Python 实现的 C 源码。这种黑魔法很少需要，除非用于构建通用开发工具和类似的高级任务。一旦你深入理解事物，Python 赋予你在必要时施加控制的能力。由于 Python 将许多种类的内部对象暴露给你的 Python 代码，即使需要理解 C 来阅读 Python 的源代码并理解正在发生的事情，你也可以通过在 Python 中编码来施加这种控制。

## 类型对象

名为`type`的内置类型充当可调用的工厂，返回类型对象。类型对象除了等价比较和表示为字符串外，不需要支持任何特殊操作。但是，大多数类型对象是可调用的，并在调用时返回该类型的新实例。特别地，内置类型如`int`、`float`、`list`、`str`、`tuple`、`set`和`dict`都是这样工作的；具体来说，当不带参数调用它们时，它们返回一个新的空实例，或者对于数字，返回等于 0 的实例。类型模块的属性是没有内置名称的内置类型。除了调用以生成实例外，类型对象之所以有用，还因为你可以从中继承，正如“类和实例”中所涵盖的那样。

## 代码对象类型

除了使用内置函数`compile`，你还可以通过函数或方法对象的`__code__`属性获取代码对象。（有关代码对象属性的讨论，请参见“编译和代码对象”。）代码对象本身不能被调用，但是你可以重新绑定函数对象的`__code__`属性，使用正确数量的参数将代码对象包装成可调用形式。例如：

```py
`def` g(x): 
    print('g', x)
code_object = g.__code__
`def` f(x): 
 `pass`
f.__code__ = code_object
f(23)     *`# prints: g 23`*
```

没有参数的代码对象也可以与`exec`或`eval`一起使用。直接创建代码对象需要许多参数；请参阅 Stack Overflow 的[非官方文档](https://oreil.ly/pM3m7)了解如何操作（但请记住，通常最好调用`compile`而不是直接创建）。

## 帧类型

模块`sys`中的函数`_getframe`从 Python 调用栈返回一个帧对象。帧对象具有属性，提供关于在帧中执行的代码和执行状态的信息。`traceback`和`inspect`模块帮助你访问和显示这些信息，特别是在处理异常时。第十七章提供了关于帧和回溯的更多信息，并涵盖了模块`inspect`，这是执行此类内省的最佳方式。如函数名`_getframe`中的前导下划线提示的那样，该函数是“有些私有”的；它仅供调试器等工具使用，这些工具不可避免地需要深入内省 Python 的内部。

# 垃圾回收

Python 的垃圾收集通常是透明且自动进行的，但您可以选择直接进行一些控制。一般原则是，Python 在对象 *x* 成为不可达时的某个时刻收集 *x*，即当没有从正在执行的函数实例的本地变量或从已加载模块的全局变量开始的引用链能够到达 *x* 时。通常，当没有任何引用指向 *x* 时，对象 *x* 变得不可达。此外，当一组对象彼此引用但没有全局或局部变量间接引用它们时，它们也可能是不可达的（这种情况称为*相互引用循环*）。

经典 Python 对每个对象 *x* 都保留一个称为*引用计数*的计数，记录了有多少引用指向 *x*。当 *x* 的引用计数降至 0 时，CPython 立即收集 *x*。模块 sys 的函数 getrefcount 接受任何对象并返回其引用计数（至少为 1，因为 getrefcount 本身对要检查的对象有一个引用）。其他版本的 Python（如 Jython 或 PyPy）依赖于由其运行的平台提供的其他垃圾收集机制（例如 JVM 或 LLVM）。因此，模块 gc 和 weakref 仅适用于 CPython。

当 Python 回收 *x* 并且没有对 *x* 的引用时，Python 会完成 *x* 的最终处理（即调用 *x*.__del__）并释放 *x* 占用的内存。如果 *x* 持有对其他对象的引用，Python 会移除这些引用，从而可能使其他对象因无法访问而可回收。

## gc 模块

gc 模块公开了 Python 垃圾收集器的功能。gc 处理了属于相互引用循环的不可达对象。如前所述，在这样的循环中，循环中的每个对象都引用另一个或多个其他对象，保持所有对象的引用计数为正数，但没有外部引用指向这组相互引用的对象集。因此，整个组，也称为*循环垃圾*，是不可达的，因此可以进行垃圾收集。寻找这样的循环垃圾需要时间，这也是为什么 gc 模块存在的原因：帮助您控制程序是否以及何时花费这些时间。默认情况下，循环垃圾收集功能处于启用状态，并具有一些合理的默认参数；但是，通过导入 gc 模块并调用其函数，您可以选择禁用功能、更改其参数和/或详细了解这方面的情况。

gc 提供了属性和函数来帮助您管理和调整循环垃圾回收，包括 表 14-2 中列出的内容。这些函数可以帮助您追踪内存泄漏 —— 尽管 *应该* 不再有对它们的引用，但仍然没有被回收的对象 —— 通过帮助您发现确实持有对它们引用的其他对象来发现它们。请注意，gc 实现了计算机科学中称为 [分代垃圾回收](https://oreil.ly/zeGDK) 的体系结构。

表 14-2\. gc 函数和属性

| callbacks | 垃圾收集器将在收集之前和之后调用的回调函数列表。有关详细信息，请参阅 “仪器化垃圾回收”。 |
| --- | --- |
| collect | collect() 立即强制执行完整的循环垃圾回收运行。 |
| disable | disable() 暂停自动周期性循环垃圾回收。 |
| enable | enable() 重新启用先前使用 disable 暂停的周期性循环垃圾回收。 |
| freeze | freeze() 冻结 gc 跟踪的所有对象：将它们移动到“永久代”，即一组在所有未来收集中被忽略的对象。 |
| garbage | 不可达但不可收集的对象列表（仅读）。当循环垃圾回收环中的任何对象具有 __del__ 特殊方法时，可能不存在明显安全的顺序来终结这些对象。 |
| get_count | get_count() 返回当前收集计数的元组，形式为 (count0, count1, count2)。 |
| get_debug | get_debug() 返回一个整数位串，表示使用 set_debug 设置的垃圾回收调试标志。 |
| get_freeze_count | get_freeze_count() 返回永久代中对象的数量。 |
| get_objects | get_objects(generation=**None**) 返回被收集器跟踪的对象列表。3.8+ 如果选择的 generation 参数不是 **None**，则仅列出所选代中的对象。 |
| get_referents | get_referents(**objs*) 返回由参数的 C 级 tp_traverse 方法访问的对象列表，这些对象被参数中任何一个引用。 |
| get_referrers | get_referrers(**objs*) 返回当前由循环垃圾回收器跟踪的所有容器对象列表，这些对象引用参数中的任意一个或多个对象。 |
| get_stats | get_stats() 返回三个字典的列表，每个字典代表一代，包含收集次数、收集的对象数和不可收集对象数。 |
| get_threshold | get_threshold() 返回当前收集阈值，以三个整数的元组形式返回。 |
| isenabled | isenabled() 当前循环垃圾回收启用时返回 **True**；否则返回 **False**。 |
| is_finalized | is_finalized(*obj*) 3.9+ 当垃圾回收器已经完成对 *obj* 的终结时返回 **True**；否则返回 **False**。 |
| is_tracked | is_tracked(*obj*) 当 *obj* 当前被垃圾收集器跟踪时返回 **True**；否则返回 **False**。 |

| set_debug | set_debug(*flags*) 设置在垃圾收集期间的调试行为标志。 *flags* 是一个整数，被解释为通过按位 OR（位或运算符，&#124;）零个或多个模块 gc 提供的常量来构建的位字符串。 每个位启用一个特定的调试功能：

DEBUG_COLLECTABLE

打印在垃圾收集期间发现的可收集对象的信息。

DEBUG_LEAK

结合 DEBUG_COLLECTABLE、DEBUG_UNCOLLECTABLE 和 DEBUG_SAVEALL 的行为。 这些通常是用于帮助诊断内存泄漏的最常见标志。

DEBUG_SAVEALL

将所有可收集对象保存到列表 gc.garbage 中（其中不可收集对象也始终保存）以帮助您诊断泄漏问题。

DEBUG_STATS

打印在垃圾收集期间收集的统计信息，以帮助您调整阈值。

DEBUG_UNCOLLECTABLE

打印在垃圾收集期间发现的不可收集对象的信息。

|

| set_threshold | set_threshold(*thresh0*[, *thresh1*[, *thresh2*]]) 设置控制循环垃圾收集周期运行频率的阈值。 *thresh0* 为 0 会禁用垃圾收集。 垃圾收集是一个高级的专题，Python 中使用的分代垃圾收集方法的详细内容（以及因此这些阈值的详细含义）超出了本书的范围； 有关详细信息，请参阅 [在线文档](https://oreil.ly/b3rm6)。 |
| --- | --- |
| unfreeze | unfreeze() 解冻永久代中的所有对象，将它们全部移回到最老的代中。 |

当您知道程序中没有循环垃圾环路，或者在某些关键时刻不能承受循环垃圾收集的延迟时，通过调用 gc.disable() 暂时停止自动垃圾收集。 您可以稍后通过调用 gc.enable() 再次启用收集。 您可以通过调用 gc.isenabled() 测试当前是否启用了自动收集，它返回 **True** 或 **False**。 要控制收集时机，可以调用 gc.collect() 强制立即执行完整的循环收集运行。 要包装一些时间关键的代码：

```py
`import` gc
gc_was_enabled = gc.isenabled()
`if` gc_was_enabled:
    gc.collect()
    gc.disable()
*`# insert some time-critical code here`*
`if` gc_was_enabled:
    gc.enable()
```

如果将其实现为上下文管理器，您可能会发现这更容易使用：

```py
`import` gc
`import` contextlib

@contextlib.contextmanager
`def` gc_disabled():
    gc_was_enabled = gc.isenabled()
    `if` gc_was_enabled:
        gc.collect()
        gc.disable()
    `try`:
        `yield`
    `finally`:
        `if` gc_was_enabled:
            gc.enable()

`with` gc_disabled():
 *`# ...insert some time-critical code here...`*
```

该模块 gc 中的其他功能更为高级且很少使用，可以分为两个领域。 函数 get_threshold 和 set_threshold 以及调试标志 DEBUG_STATS 帮助您微调垃圾收集以优化程序的性能。 gc 的其余功能可以帮助您诊断程序中的内存泄漏。 虽然 gc 本身可以自动修复许多泄漏问题（只要避免在类中定义 __del__，因为存在 __del__ 可以阻止循环垃圾收集），但是如果首先避免创建循环垃圾，程序运行速度将更快。

### 垃圾收集工具

gc.callbacks 是一个最初为空的列表，您可以向其中添加函数 f(*phase*, *info*)，Python 将在垃圾回收时调用这些函数。当 Python 调用每个这样的函数时，*phase* 为 'start' 或 'stop'，用于标记收集的开始或结束，*info* 是一个字典，包含由 CPython 使用的分代收集的信息。您可以向此列表添加函数，例如用于收集有关垃圾回收的统计信息。有关更多详细信息，请参阅[文档](https://oreil.ly/GjJF-)。

## 弱引用模块

谨慎的设计通常可以避免引用循环。然而，有时你需要对象彼此知道对方的存在，避免相互引用可能会扭曲和复杂化你的设计。例如，一个容器引用了其项目，但是对象知道容器持有它也常常很有用。结果就是引用循环：由于相互引用，容器和项目保持彼此存活，即使所有其他对象都忘记了它们。弱引用通过允许对象引用其他对象而不保持它们存活来解决了这个问题。

*弱引用* 是一个特殊对象 *w*，它引用某个其他对象 *x* 而不增加 *x* 的引用计数。当 *x* 的引用计数降至 0 时，Python 将终止并收集 *x*，然后通知 *w* *x* 的消亡。弱引用 *w* 现在可以消失或以受控方式标记为无效。在任何时候，给定的 *w* 要么引用创建 *w* 时的相同对象 *x*，要么完全不引用；弱引用永远不会被重新定位。并非所有类型的对象都支持成为弱引用 *w* 的目标 *x*，但是类、实例和函数支持。

弱引用模块公开了用于创建和管理弱引用的函数和类型，详见表格 14-3。

表格 14-3\. 弱引用模块的函数和类

| getweakrefcount | getweakrefcount(*x*) 返回 len(getweakrefs(*x*))。 |
| --- | --- |
| getweakrefs | getweakrefs(*x*) 返回所有目标为 *x* 的弱引用和代理的列表。 |
| proxy | proxy(*x*[, *f*]) 返回类型为 ProxyType（当 *x* 是可调用对象时为 CallableProxyType）的弱代理 *p*，以 *x* 为目标。使用 *p* 就像使用 *x* 一样，但是当使用 *p* 时，*x* 被删除后，Python 将引发 ReferenceError。*p* 永远不可哈希（您不能将 *p* 用作字典键）。当 *f* 存在时，它必须是一个接受一个参数的可调用对象，并且是 *p* 的最终化回调（即在 *x* 不再从 *p* 可达时，Python 调用 *f*(*p*)）。*f* 在 *x* 不再从 *p* 可达后立即执行。 |
| ref | ref(*x*[, *f*]) 返回类型为 ReferenceType 的对象 *x* 作为目标的弱引用 *w*。*w* 可以无参数地调用：调用 *w*() 在 *x* 仍然存活时返回 *x*；否则，调用 *w*() 返回 None。当 *x* 可散列时，*w* 是可散列的。您可以比较弱引用的相等性（==、!=），但不能进行顺序比较（<、>、<=、>=）。当它们的目标存活且相等时，或者 *x* 等于 *y* 时，两个弱引用 *x* 和 *y* 是相等的。当存在 *f* 时，它必须是一个带有一个参数的可调用对象，并且是 *w* 的最终化回调（即，在 *x* 从 *w* 不再可达之后，Python 调用 *f*(*w*)）。*f* 在 *x* 不再从 *w* 可达后立即执行。 |
| 弱键字典 | class WeakKeyDictionary(adict={}) 弱键字典 *d* 是一个弱引用其键的映射。当 *d* 中键 *k* 的引用计数为 0 时，项目 *d*[*k*] 消失。adict 用于初始化映射。 |
| 弱引用集 | class WeakSet(elements=[]) 弱引用集 *s* 是一个弱引用其内容元素的集合，从元素初始化。当 *s* 中元素 *e* 的引用计数为 0 时，*e* 从 *s* 中消失。 |
| 弱值字典 | class WeakValueDictionary(adict={}) 弱值字典 *d* 是一个弱引用其值的映射。当 *d* 中值 *v* 的引用计数为 0 时，*d* 中所有 *d*[*k*] **为** *v* 的项目消失。adict 用于初始化映射。 |

WeakKeyDictionary 允许您在一些可散列对象上非侵入式地关联附加数据，而无需更改这些对象。WeakValueDictionary 允许您非侵入式地记录对象之间的瞬时关联，并构建缓存。在每种情况下，使用弱映射而不是字典，以确保其他情况下可被垃圾回收的对象不会仅因在映射中使用而保持活动状态。类似地，WeakSet 在普通集合的位置提供了相同的弱包含功能。

典型示例是一个跟踪其实例但不仅仅是为了跟踪它们而使它们保持活动状态的类：

```py
`import` weakref
`class` Tracking:
    _instances_dict = weakref.WeakValueDictionary()

    def __init__(self):
        Tracking._instances_dict[id(self)] = self

    @classmethod
    def instances(cls):
        `return` cls._instances_dict.values()
```

当跟踪实例是可散列的时候，可以使用一个实例的 WeakSet 类实现类似的类，或者使用一个 WeakKeyDictionary，其中实例作为键，**None**作为值。
