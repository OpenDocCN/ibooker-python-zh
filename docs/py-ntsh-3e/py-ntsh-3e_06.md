# 第六章. 异常

Python 使用 *异常* 来指示错误和异常情况。当 Python 检测到错误时，它 *引发* 一个异常——也就是说，Python 通过将异常对象传递给异常传播机制来表示异常条件的发生。您的代码可以通过执行 **raise** 语句显式地引发异常。

*处理* 异常意味着从传播机制捕获异常对象，并根据需要采取行动来处理异常情况。如果程序未处理异常，则程序将以错误消息和回溯消息终止。然而，程序可以通过使用带有 **except** 子句的 **try** 语句来处理异常并继续运行，尽管存在错误或其他异常情况。

Python 还使用异常来指示一些不是错误，甚至不是异常的情况。例如，如在 “Iterators” 中所述，对迭代器调用内置的 next 函数在迭代器没有更多项时引发 StopIteration。这不是错误；它甚至不是异常，因为大多数迭代器最终会耗尽项目。因此，在 Python 中检查和处理错误及其他特殊情况的最佳策略与其他语言不同；我们在 “Error-Checking Strategies” 中介绍它们。

本章介绍如何使用异常处理错误和特殊情况。还涵盖了标准库中的日志记录模块，在 “Logging Errors” 中，以及 **assert** 语句，在 “The assert Statement” 中。

# The try Statement

**try** 语句是 Python 的核心异常处理机制。它是一个复合语句，具有三种可选的子句：

1.  它可以有零个或多个 **except** 子句，定义如何处理特定类别的异常。

1.  如果它有 **except** 子句，那么紧接着可能还有一个 **else** 子句，仅当 **try** 语句块未引发异常时执行。

1.  无论它是否有 **except** 子句，它都可能有一个单独的 **finally** 子句，无条件执行，其行为在 “try/except/finally” 中介绍。

Python 的语法要求至少有一个 **except** 子句或一个 **finally** 子句，两者都可以在同一语句中存在；**else** 只能在一个或多个 **except** 之后才有效。

## try/except

下面是 **try** 语句的 **try**/**except** 形式的语法：

```py
`try`:
    *`statement``(``s``)`*
`except` [*`expression`* [`as` *`target`*]]:
    *`statement``(``s``)`*
[`else`:
    *`statement``(``s``)`*]
[`finally`:
    *`statement``(``s``)`*]
```

这种形式的 **try** 语句具有一个或多个 **except** 子句，以及一个可选的 **else** 子句（和一个可选的 **finally** 子句，其含义不取决于是否存在 **except** 和 **else** 子句：我们在下一节中详细介绍这一点）。

每个 **except** 子句的主体称为 *异常处理程序*。当 **except** 子句中的 *表达式* 与从 **try** 子句传播出的异常对象匹配时，代码执行。*表达式* 是一个类或类元组，用括号括起来，匹配任何一个这些类或它们的子类的实例。可选的 *目标* 是一个标识符，它命名一个变量，Python 在异常处理程序执行之前将异常对象绑定到该变量上。处理程序还可以通过调用模块 sys 的 exc_info 函数（3.11+ 或异常函数）来获取当前的异常对象（在 Table 9-3 中介绍）。

这里是 **try** 语句的 try/except 形式的示例：

```py
`try`:
    1/0
    print('not executed')
`except` ZeroDivisionError:
    print('caught divide-by-0 attempt')
```

当引发异常时，**try** 套件的执行立即停止。如果一个 **try** 语句有多个 **except** 子句，异常传播机制按顺序检查 **except** 子句；第一个表达式匹配异常对象的 **except** 子句作为处理程序执行，异常传播机制在此之后不再检查任何其他 **except** 子句。

# 先具体后一般

将特定情况的处理程序放在一般情况的处理程序之前：当你首先放置一个一般情况时，随后的更特定的 **except** 子句将不会执行。

最后的 **except** 子句不需要指定表达式。一个没有任何表达式的 **except** 子句处理在传播期间达到它的任何异常。这样的无条件处理很少见，但确实会发生，通常在必须在重新引发异常之前执行某些额外任务的“包装器”函数中（参见 “The raise Statement”）。

# 避免“裸的 except” 没有重新引发

小心使用“裸”的 **except**（一个没有表达式的 **except** 子句），除非你在其中重新引发异常：这种粗糙的风格会使得错误非常难以找到，因为裸的 **except** 太宽泛，可以轻易掩盖编码错误和其他类型的错误，允许执行在未预期的异常后继续。

“只是想让事情运行起来”的新程序员甚至可能编写如下的代码：

```py
`try``:`
    *`# ...code that has a problem...`*
`except``:`
    `pass`
```

这是一种危险的做法，因为它捕捉到重要的进程退出异常，如 KeyboardInterrupt 或 SystemExit ——带有这种异常处理程序的循环无法通过 Ctrl-C 退出，并且可能甚至无法通过系统 **kill** 命令终止。至少，这样的代码应该使用 **except** Exception:，这仍然太宽泛，但至少不会捕获导致进程退出的异常。

当它找到一个表达式与异常对象匹配的处理程序时，异常传播终止。当`try`语句嵌套（在源代码的词法上，或在函数调用中动态地）在另一个`try`语句的`try`子句中时，内部`try`建立的处理程序首先在传播时达到，因此当匹配时它处理异常。这可能不是您想要的。考虑这个例子：

```py
`try`:
    `try`:
        1/0
    `except`:
        print('caught an exception')
`except` `ZeroDivisionError`:
    print('caught divide-by-0 attempt')
*`# prints:`* *`caught an exception`*
```

在这种情况下，由外部`try`子句中的`except ZeroDivisionError:`建立的处理程序比内部`try`子句中的通用`except:`更为具体，并不重要。外部`try`不参与其中：异常不会从内部`try`传播出来。有关异常传播的更多信息，请参阅“异常传播”。

`try`/`except`的可选`else`子句仅在`try`子句正常终止时执行。换句话说，当异常从`try`子句传播出来时，或者`try`子句以`break`、`continue`或`return`语句退出时，`else`子句不会执行。由`try`/`except`建立的处理程序仅覆盖`try`子句，而不包括`else`子句。`else`子句对于避免意外处理未预期的异常很有用。例如：

```py
print(repr(value), 'is ', end=' ')
`try`:
    value + 0
`except` TypeError:
    *`# not a number, maybe a string...?`*
    `try`:
        value + ''
    `except` TypeError:
        print('neither a number nor a string')
    `else`:
        print('some kind of string')
`else`:
    print('some kind of number')
```

## `try`/`finally`

下面是`try`语句的`try`/`finally`形式的语法：

```py
`try`:
    *`statement``(``s``)`*
`finally`:
    *`statement``(``s``)`*
```

此形式有一个`finally`子句，没有`else`子句（除非它还有一个或多个`except`子句，如下一节所述）。

`finally`子句建立了所谓的*清理处理程序*。这段代码在`try`子句以任何方式终止后始终执行。当异常从`try`子句传播时，`try`子句终止，清理处理程序执行，异常继续传播。当没有异常发生时，无论`try`子句是否达到其末尾或通过执行`break`、`continue`或`return`语句退出，清理处理程序都会执行。

使用`try`/`finally`建立的清理处理程序提供了一种健壮且明确的方式来指定必须始终执行的最终代码，无论如何，以确保程序状态和/或外部实体（例如文件、数据库、网络连接）的一致性。这种确保的最终化现在通常通过在`with`语句中使用*上下文管理器*来表达最佳（请参阅“with 语句和上下文管理器”）。这里是`try`语句的`try`/`finally`形式的示例：

```py
f = open(some_file, 'w')
`try`:
    do_something_with_file(f)
`finally`:
    f.close()
```

这里是相应的更简洁和可读性更好的示例，使用`with`来达到完全相同的目的：

```py
`with` open(some_file, 'w') `as` f:
    do_something_with_file(f)
```

# 避免在`finally`子句中使用`break`和`return`语句。

**finally** 子句可以包含一个或多个语句 **continue**，3.8+ **break** 或 **return**。 然而，这种用法可能使你的程序变得不太清晰：当这样的语句执行时，异常传播会停止，并且大多数程序员不希望在 **finally** 子句内停止传播。 这种用法可能会使阅读你代码的人感到困惑，因此我们建议你避免使用它。

## try/except/finally

一个 **try**/**except**/**finally** 语句，例如：

```py
`try`:
    ...*`guarded` `clause`*...
`except` ...*`expression`*...:
    ...*`exception` `handler` `code`*...
`finally`:
    ...*`cleanup` `code`*...
```

等价于嵌套语句：

```py
`try`:
    `try`:
        ...*`guarded` `clause`*...
    `except` ...*`expression`*...:
        ...*`exception` `handler` `code`*...
`finally`:
    ...*`cleanup` `code`*...
```

**try** 语句可以有多个 **except** 子句，并且可选地有一个 **else** 子句，在终止的 **finally** 子句之前。 在所有变体中，效果总是像刚才展示的那样 - 即，它就像将一个 **try**/**except** 语句的所有 **except** 子句和 **else** 子句（如果有的话）嵌套到包含的 **try**/**finally** 语句中。

# **raise** 语句

你可以使用 **raise** 语句显式地引发异常。 **raise** 是一个简单语句，其语法如下：

```py
`raise` [*`expression`* [`from` *`exception`*]]
```

只有异常处理程序（或处理程序直接或间接调用的函数）可以使用没有任何表达式的 **raise**。 一个普通的 **raise** 语句会重新引发处理程序收到的相同异常对象。 处理程序终止，异常传播机制继续沿调用堆栈向上搜索其他适用的处理程序。 当处理程序发现无法处理接收到的异常或只能部分处理异常时，使用没有任何表达式的 **raise** 是有用的，因此异常应继续传播以允许调用堆栈上的处理程序执行其自己的处理和清理。

当存在 *expression* 时，它必须是从内置类 BaseException 继承的类的实例，Python 将引发该实例。

当包括 **from** *exception*（只能出现在接收 *exception* 的 **except** 块中）时，Python 会将接收到的表达式“嵌套”在新引发的异常表达式中。 "异常“包裹”其他异常或回溯" 更详细地描述了这一点。

下面是 **raise** 语句的一个典型用例示例：

```py
`def` cross_product(seq1, seq2):
    `if` `not` seq1 `or` `not` seq2:
        `raise` ValueError('Sequence arguments must be non-empty') ![1](img/1.png)
    `return` [(x1, x2) `for` x1 `in` seq1 `for` x2 `in` seq2]
```

![1](img/#comarker1)

有些人认为在此处引发标准异常是不合适的，他们更倾向于引发自定义异常的实例，这在本章后面有所涵盖；本书的作者对此持不同意见。

此 cross_product 示例函数返回由其序列参数中的每个项目组成的所有配对的列表，但首先它测试了两个参数。 如果任一参数为空，则该函数引发 ValueError，而不仅仅像列表推导通常所做的那样返回一个空列表。

# 只检查你需要的内容

无需 cross_product 检查 seq1 和 seq2 是否可迭代：如果其中任一者不可迭代，则列表推导本身会引发适当的异常，通常是 TypeError。

一旦由 Python 本身引发异常，或者在代码中使用显式 **raise** 语句引发异常，就由调用者来处理它（使用合适的 **try**/**except** 语句）或让它继续向调用堆栈上传播。

# 不要为冗余错误检查使用 raise 语句

仅在你的规范定义为错误的情况下，才使用 raise 语句来引发额外的异常。不要使用 raise 来复制 Python 已（隐式地）代表你执行的相同错误检查。

# **with** 语句和上下文管理器

**with** 语句是一个复合语句，具有以下语法：

```py
with *expression* [as *varname*] [, ...]:
    *statement(s)*

*# 3.10+ multiple context managers for a with statement* 
*# can be enclosed in parentheses*
with (*expression* [as *varname*], ...):
    *statement(s)*
```

**with** 的语义等效于

```py
_normal_exit = `True`
_manager = *`expression`*
*`varname`* = _manager.__enter__()
`try`:
    *`statement``(``s``)`*
`except`:
    _normal_exit = `False`
    `if` `not` _manager.__exit_(*sys.exc_info()):
        `raise`
    *`# note that exception does not propagate if __exit__ returns`* 
    *`# a true value`*
`finally`:
    `if` _normal_exit:
        _manager.__exit__(`None``,` `None``,` `None`)
```

其中 _manager 和 _normal_exit 是当前范围中不使用的任意内部名称。如果在 **with** 子句的可选 **as** *varname* 部分中省略 *varname*，Python 仍会调用 _manager.__enter__，但不会将结果绑定到任何名称，并且仍会在块终止时调用 _manager.__exit__。通过 *expression* 返回的对象，具有方法 __enter__ 和 __exit__，被称为 *上下文管理器*。

**with** 语句是 Python 中著名的 C++ 惯用语 [“资源获取即初始化” (RAII)](https://oreil.ly/vROml) 的体现：你只需编写上下文管理器类，即包含两个特殊方法的类，__enter__ 和 __exit__。__enter__ 方法必须可被无参数调用。__exit__ 方法必须接受三个参数：当主体完成且未传播异常时均为 **None**，否则为异常的类型、值和回溯信息。这提供了与 C++ 中自动变量的典型构造函数/析构函数对和 Python 或 Java 中 **try**/**finally** 语句相同的确保最终化行为。此外，它们可以根据传播的异常（如果有的话）以不同方式进行最终化，并且通过从 __exit__ 返回 true 值来选择性地阻止传播的异常。

例如，这里是一个简单的纯示例方式，确保在一些其他输出周围打印 <name> 和 </name> 标签（请注意，上下文管理器类通常具有小写名称，而不是遵循类名的正常大写约定）：

```py
`class` enclosing_tag:
    `def` __init__(self, tagname):
        self.tagname = tagname
    `def` __enter__(self):
        print(f'<{self.tagname}>', end='')
    `def` __exit__(self, etyp, einst, etb):
        print(f'</{self.tagname}>')

*`# to be used as:`*
`with` enclosing_tag('sometag'):
    *`# ...statements printing output to be enclosed in`*
    *``# a matched open/close `sometag` pair...``*
```

构建上下文管理器的一种更简单方法是使用 Python 标准库中 contextlib 模块中的 contextmanager 装饰器。此装饰器将生成器函数转换为上下文管理器对象的工厂。

在 contextlib 模块中导入之后，实现 enclosing_tag 上下文管理器的方式是：

```py
@contextlib.contextmanager
`def` enclosing_tag(tagname):
    print(f'<{tagname}>', end='')
    `try`:
        `yield`
    `finally`:
        print(f'</{tagname}>')
*`# to be used the same way as before`*
```

contextlib 提供了 Table 6-1 中列出的类和函数，以及其他一些类和函数。

表 6-1\. 上下文管理模块中常用的类和函数

| AbstractContextManager | AbstractContextManager 是一个具有两个可重写方法的抽象基类：__enter__ 默认为 **return** self，__exit__ 默认为 **return** None。 |
| --- | --- |
| chdir | chdir(*dir_path*) 3.11+ 一个上下文管理器，其 __enter__ 方法保存当前工作目录路径并执行 os.chdir(*dir_path*)，其 __exit__ 方法执行 os.chdir(*saved_path*)。 |
| closing | closing(*something*) 一个上下文管理器，其 __enter__ 方法返回*something*，而其 __exit__ 方法调用*something*.close()。 |
| contextmanager | contextmanager 将生成器应用为上下文管理器的装饰器。 |
| nullcontext | nullcontext(*something*) 一个上下文管理器，其 __enter__ 方法返回*something*，而其 __exit__ 方法什么也不做。 |
| redirect_stderr | redirect_stderr(*destination*) 一个上下文管理器，可以临时将 sys.stderr 在**with**语句体内重定向到文件或类文件对象*destination*。 |
| redirect_stdout | redirect_stdout(*destination*) 一个上下文管理器，可以临时将 sys.stdout 在**with**语句体内重定向到文件或类文件对象*destination*。 |

| suppress | suppress(*exception_classes*) 一个上下文管理器，可以在列出的*exception_classes*中的任何一种出现在**with**语句体中时，静默地抑制异常。例如，这个删除文件的函数忽略了 FileNotFoundError：

```py
`def` delete_file(filename):
    `with` contextlib.suppress(FileNotFoundError):
        os.remove(filename)
```

使用时要节制，因为静默地抑制异常通常是不好的做法。 |

更多细节、示例、“配方”甚至更多（有些深奥）的类，请参阅 Python 的[在线文档](https://oreil.ly/Jwr_w)。

# Generators and Exceptions

为了帮助生成器与异常协作，**try**/**finally**语句中允许使用**yield**语句。此外，生成器对象还有另外两个相关方法，throw 和 close。给定通过调用生成器函数构建的生成器对象*g*，throw 方法的签名为：

```py
g.throw(*`exc_value`*)
```

当生成器的调用者调用*g*.throw 时，其效果就好像在生成器*g*暂停的**yield**处执行具有相同参数的**raise**语句一样。

生成器方法 close 没有参数；当生成器的调用者调用*g*.close()时，其效果就像调用*g*.throw(GeneratorExit())一样。¹ GeneratorExit 是一个直接继承自 BaseException 的内置异常类。生成器还有一个终结器（特殊方法 __del__），当生成器对象被垃圾回收时会隐式调用 close 方法。

如果生成器引发或传播 StopIteration 异常，Python 会将异常类型转换为 RuntimeError。

# 异常传播

当引发异常时，异常传播机制接管控制。程序的正常控制流停止，Python 寻找合适的异常处理程序。Python 的 **try** 语句通过其 **except** 子句设立异常处理程序。处理程序处理 **try** 子句中引发的异常，以及直接或间接调用该代码的函数中传播的异常。如果在具有适用 **except** 处理程序的 **try** 子句中引发异常，则 **try** 子句终止并执行处理程序。处理程序完成后，继续执行 **try** 语句之后的语句（在没有显式更改控制流程的情况下，例如 **raise** 或 **return** 语句）。

如果引发异常的语句不在具有适用处理程序的 **try** 子句内，则包含该语句的函数终止，并且异常沿着函数调用堆栈向上“传播”到调用该函数的语句。如果终止的函数调用位于具有适用处理程序的 **try** 子句内，则该 **try** 子句终止，并执行处理程序。否则，包含调用的函数终止，并且传播过程重复，*展开* 函数调用堆栈，直到找到适用的处理程序。

如果 Python 找不到任何适用的处理程序，默认情况下程序会将错误消息打印到标准错误流（sys.stderr）。错误消息包括有关在传播过程中终止的函数的详细跟踪信息。您可以通过设置 sys.excepthook（在 表 8-3 中讨论）来更改 Python 的默认错误报告行为。在错误报告之后，Python 返回交互会话（如果有），或者如果执行不是交互的，则终止。当异常类型为 SystemExit 时，终止是静默的，并结束交互会话（如果有）。

这里有一些函数来展示异常传播的工作原理：

```py
`def` f():
    print('in f, before 1/0')
    1/0    *`# raises a ZeroDivisionError exception`*
    print('in f, after 1/0')
`def` g():
    print('in g, before f()')
    f()
    print('in g, after f()')
`def` h():
    print('in h, before g()')
    `try`:
        g()
        print('in h, after g()')
    `except` ZeroDivisionError:
        print('ZD exception caught')
    print('function h ends')
```

调用 h 函数会打印以下内容：

```py
in h, before g()
in g, before f()
in f, before 1/0
ZD exception caught
function h ends
```

也就是说，由于异常传播的流程切断了它们，没有一个“after”打印语句被执行。

函数 h 设立了一个 **try** 语句并在 **try** 子句中调用函数 g。g 反过来调用 f，而 f 进行了除以 0 的操作，引发了 ZeroDivisionError 类型的异常。异常传播直到 h 的 **except** 子句。函数 f 和 g 在异常传播阶段终止，这就是为什么它们的“after”消息都没有打印出来。h 的 **try** 子句的执行也在异常传播阶段终止，因此它的“after”消息也没有打印出来。在处理程序之后，h 的 **try**/**except** 块结束时，继续执行。

# 异常对象

异常是 BaseException 的实例（更具体地说，是其子类之一的实例）。Table 6-2 列出了 BaseException 的属性和方法。

Table 6-2\. BaseException 类的属性和方法

| __cause__ | *exc*.__cause__ 返回使用**raise** **from**引发的异常的父异常。 |
| --- | --- |
| __notes__ | *exc*.__notes__ 3.11+ 返回一个包含使用 add_note 添加到异常中的字符串列表。只有在至少调用一次 add_note 之后才存在此属性，因此安全的访问此列表的方法是使用 getattr(*exc*, '__notes__', [])。 |
| add_note | *exc*.add_note(*note*) 3.11+ 将字符串 *note* 添加到此异常的注释中。在显示异常时，这些注释会显示在回溯信息之后。 |
| args | *exc.*args 返回用于构造异常的参数的元组。这些特定于错误的信息对诊断或恢复目的非常有用。某些异常类解释 args 并在类的实例上设置便捷的命名属性。 |
| wi⁠t⁠h⁠_​t⁠r⁠a⁠c⁠e⁠b⁠a⁠c⁠k | *exc*.with_traceback(*tb*) 返回一个新的异常，用新的回溯 *tb* 替换原始异常的回溯，如果 *tb* 是 **None** 则不包含回溯。可用于修剪原始回溯以删除内部库函数调用帧。 |

## 标准异常的层次结构

如前所述，异常是 BaseException 的子类的实例。异常类的继承结构很重要，因为它决定了哪些**except**子句处理哪些异常。大多数异常类扩展自 Exception 类；然而，KeyboardInterrupt、GeneratorExit 和 SystemExit 直接继承自 BaseException，并不是 Exception 的子类。因此，一个处理器子句**except** Exception **as** e 无法捕获 KeyboardInterrupt、GeneratorExit 或 SystemExit（我们在“try/except”和“Generators and Exceptions”中介绍了异常处理程序）。SystemExit 的实例通常是通过 sys 模块中的 exit 函数引发的（在 Table 8-3 中有所涵盖）。当用户按 Ctrl-C、Ctrl-Break 或其他中断键时，会引发 KeyboardInterrupt。

内置异常类的层次结构大致如下：

```py
BaseException
  Exception
    AssertionError, AttributeError, BufferError, EOFError,
    MemoryError, ReferenceError, OsError, StopAsyncIteration,
    StopIteration, SystemError, TypeError
    ArithmeticError (abstract)
      OverflowError, ZeroDivisionError
    ImportError
      ModuleNotFoundError, ZipImportError
    LookupError (abstract)
      IndexError, KeyError
    NameError
      UnboundLocalError
    OSError
      ...
    RuntimeError
      RecursionError
      NotImplementedError
    SyntaxError
      IndentationError
        TabError
    ValueError
      UnsupportedOperation
      UnicodeError
        UnicodeDecodeError, UnicodeEncodeError,
        UnicodeTranslateError
    Warning
      ...
  GeneratorExit
  KeyboardInterrupt
  SystemExit
```

其他异常子类（特别是警告和 OSError 有很多，这里用省略号表示），但这就是要点。完整列表可在 Python 的[在线文档](https://oreil.ly/pLihr)中找到。

标记为“(abstract)”的类永远不会直接实例化；它们的目的是使您能够指定处理一系列相关错误的**except**子句。

## 标准异常类

Table 6-3 列出了由常见运行时错误引发的异常类。

Table 6-3\. 标准异常类

| 异常类 | 抛出时机 |
| --- | --- |
| AssertionError | 一个**assert**语句失败。 |
| AttributeError | 属性引用或赋值失败。 |
| ImportError | 一个**import**或**from**...**import**语句（详见“import 语句”）找不到要导入的模块（在这种情况下，Python 实际引发的是 ImportError 的子类 ModuleNotFoundError），或找不到要从模块导入的名称。 |
| IndentationError | 解析器由于不正确的缩进而遇到语法错误。子类 SyntaxError。 |
| IndexError | 用于索引序列的整数超出范围（使用非整数作为序列索引会引发 TypeError）。子类 LookupError。 |
| KeyboardInterrupt | 用户按下中断键组合（Ctrl-C、Ctrl-Break、Delete 或其他，取决于平台对键盘的处理）。 |
| KeyError | 用于索引映射的键不在映射中。子类 LookupError。 |
| MemoryError | 操作耗尽了内存。 |
| NameError | 引用了一个名称，但它没有绑定到当前作用域中的任何变量。 |
| N⁠o⁠t⁠I⁠m⁠p⁠l⁠e⁠m⁠e⁠n⁠t⁠e⁠d​E⁠r⁠r⁠o⁠r | 抽象基类引发以指示必须重写方法的具体子类。 |
| OSError | 由 os 模块中的函数引发（详见“os 模块”和“使用 os 模块运行其他程序”），以指示平台相关的错误。OSError 有许多子类，详见下一小节。 |
| RecursionError | Python 检测到递归深度已超出。子类 RuntimeError。 |
| RuntimeError | 为未归类的任何错误或异常引发。 |
| SyntaxError | Python 解析器遇到语法错误。 |
| SystemError | Python 检测到自己代码或扩展模块中的错误。请向您的 Python 版本维护者或相关扩展的维护者报告此问题，包括错误消息、确切的 Python 版本（sys.version），如果可能的话，请附上程序源代码。 |
| TypeError | 应用于不适当类型的对象的操作或函数。 |
| UnboundLocalError | 引用了一个本地变量，但当前未绑定任何值到该本地变量。子类 NameError。 |
| UnicodeError | 在转换 Unicode（即 str）到字节字符串或反之过程中发生错误。子类 ValueError。 |
| ValueError | 应用于具有正确类型但不合适值的对象的操作或函数，且没有更具体的异常适用（例如 KeyError）。 |
| ZeroDivisionError | 除数（/、//或%运算符的右操作数，或内置函数 divmod 的第二个参数）为 0。子类 ArithmeticError。 |

### OSError 的子类

OSError 代表操作系统检测到的错误。为了更优雅地处理这些错误，OSError 有许多子类，其实例是实际抛出的内容；完整列表请参阅 Python 的[在线文档](https://oreil.ly/3vJ3W)。

例如，考虑这个任务：尝试读取并返回某个文件的内容，如果文件不存在则返回默认字符串，并传播使文件不可读的任何其他异常（除了文件不存在）。使用现有的 OSError 子类，您可以很简单地完成这个任务：

```py
`def` read_or_default(filepath, default):
    `try`:
        `with` open(filepath) `as` f:
            `return` f.read()
    `except` FileNotFoundError:
        `return` default
```

FileNotFoundError 的 OSError 子类使得这种常见任务在代码中表达起来简单直接。

### 异常“包装”其他异常或 traceback

有时候，在尝试处理另一个异常时会引发异常。为了让您清楚地诊断此问题，每个异常实例都持有其自己的 traceback 对象；您可以使用 with_traceback 方法创建另一个具有不同 traceback 的异常实例。

此外，Python 自动存储它正在处理的异常作为任何后续异常的 __context__ 属性（除非您使用 **raise**...**from** 语句并将异常的 __suppress_context__ 属性设置为 **True**，这部分我们稍后会讲到）。如果新异常传播，Python 的错误消息将使用该异常的 __context__ 属性显示问题的详细信息。例如，看看这个（故意！）有问题的代码：

```py
`try`:
    1/0
`except` ZeroDivisionError:
    1+'x'
```

显示的错误是：

```py
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
 File "<stdin>", line 3, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

因此，Python 明确显示原始异常和干预异常。

如果您希望更多地控制错误显示，可以使用 **raise**...**from** 语句。当您执行 **raise** *e* **from** *ex* 时，*e* 和 *ex* 都是异常对象：*e* 是传播的异常，*ex* 是其“原因”。Python 记录 *ex* 作为 *e.__cause__* 的值，并将 *e.__suppress_context__* 设置为 true。（或者，*ex* 可以为 **None**：然后，Python 将 *e.__cause__* 设置为 **None**，但仍将 *e.__suppress_context__* 设置为 true，并因此保持 *e.__context__* 不变）。

作为另一个示例，这里是一个使用 Python 字典实现模拟文件系统目录的类，其中文件名作为键，文件内容作为值：

```py
`class` FileSystemDirectory:
    `def` __init__(self):
        self._files = {}

    `def` write_file(self, filename, contents):
        self._files[filename] = contents

    `def` read_file(self, filename):
        `try`:
            return self._files[filename]
        `except` KeyError:
            `raise` FileNotFoundError(filename)
```

当使用不存在的文件名调用 read_file 时，对 self._files 字典的访问会引发 KeyError。由于此代码旨在模拟文件系统目录，read_file 捕获 KeyError 并抛出 FileNotFoundError。

就像现在，访问名为 'data.txt' 的不存在文件将输出类似以下的异常消息：

```py
Traceback (most recent call last):
 File "C:\dev\python\faux_fs.py", line 11, in read_file
 return self._files[filename]
KeyError: 'data.txt'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
 File "C:\dev\python\faux_fs.py", line 20, in <module>
 print(fs.read_file("data.txt"))
 File "C:\dev\python\faux_fs.py", line 13, in read_file
 raise FileNotFoundError(filename)
FileNotFoundError: data.txt
```

此异常报告显示了 KeyError 和 FileNotFoundError。为了抑制内部 KeyError 异常（隐藏 FileSystemDirectory 的实现细节），我们在 read_file 中的 **raise** 语句改为：

```py
 `raise` FileNotFoundError(filename) `from` `None`
```

现在异常只显示 FileNotFoundError 的信息：

```py
Traceback (most recent call last):
 File "C:\dev\python\faux_fs.py", line 20, in <module>
 print(fs.read_file("data.txt"))
 File "C:\dev\python\faux_fs.py", line 13, in read_file
 raise FileNotFoundError(filename) from None
FileNotFoundError: data.txt
```

关于异常链和嵌入的详细信息和动机，请参阅 [PEP 3134](https://oreil.ly/wE9rL)。

# 自定义异常类

你可以扩展任何标准异常类来定义自己的异常类。通常，这样的子类除了一个文档字符串外没有其他内容：

```py
`class` InvalidAttributeError(AttributeError):
    *`"""Used to indicate attributes that could never be valid."""`*
```

# 空类或函数应该有一个文档字符串。

如“pass 语句”所述，你不需要一个 pass 语句来构成一个类的主体。只要有文档字符串（如果什么都不写的话，也应该写一个来记录类的目的！），Python 就足够了。对于所有“空”类（无论是否为异常类），最佳实践通常是只有一个文档字符串，而没有**pass**语句。

鉴于**try**/**except**的语义，引发自定义异常类的实例（如 InvalidAttributeError）几乎与引发其标准异常超类 AttributeError 的实例相同，但具有一些优势。任何可以处理 AttributeError 的**except**子句也可以很好地处理 InvalidAttributeError。此外，了解你的 InvalidAttributeError 自定义异常类的客户端代码可以专门处理它，而不必在不准备处理其他情况的情况下处理所有其他 AttributeError 的情况。例如，假设你编写如下代码：

```py
`class` SomeFunkyClass:
    *`"""much hypothetical functionality snipped"""`*
    `def` __getattr__(self, name):
        *`"""only clarifies the kind of attribute error"""`*
        `if` name.startswith('_'):
            `raise` InvalidAttributeError(
                f'Unknown private attribute {name!r}'
            )
        `else`:
            `raise` AttributeError(f'Unknown attribute {name!r}')
```

现在，客户端代码可以选择性地更加精确地处理异常。例如：

```py
s = SomeFunkyClass()
`try`:
    value = getattr(s, thename)
`except` InvalidAttributeError as err:
    warnings.warn(str(err), stacklevel=2)
    value = `None`
*`# other cases of AttributeError just propagate, as they're unexpected`*
```

# 使用自定义异常类

在你的模块中定义和引发自定义异常类的实例，而不是普通的标准异常，是一个绝佳的主意。通过使用扩展标准异常的自定义异常类，你使调用者更容易单独处理来自你模块的异常，如果他们选择这样做的话。

## 自定义异常和多重继承

使用自定义异常类的有效方法是从你模块的特定自定义异常类和标准异常类中多重继承异常类，如下片段所示：

```py
`class` CustomAttributeError(CustomException, AttributeError):
    *`"""An AttributeError which is ALSO a CustomException."""`*
```

现在，CustomAttributeError 的实例只能被显式和有意地引发，显示了与你的代码特别相关的错误，*也*恰好是一个 AttributeError。当你的代码引发 CustomAttributeError 的实例时，该异常可以被设计为捕获所有 AttributeError 情况的调用代码以及被设计为仅处理你模块引发的所有异常情况的代码捕获。

# 使用多重继承定义自定义异常

每当您必须决定是否引发特定标准异常的实例，例如 AttributeError，或者您在模块中定义的自定义异常类的实例时，请考虑这种多重继承的方法，在本书作者的观点中，这种方法在这些情况下是最佳选择。确保您清楚地记录模块的这一方面，因为这种技术虽然方便，但使用并不广泛。除非您明确和明确地记录您正在做什么，否则模块的用户可能不会预期到这一点。

## 标准库中使用的其他异常

Python 标准库中的许多模块定义了它们自己的异常类，这些异常类相当于您自己模块可以定义的自定义异常类。通常，这类标准库模块中的所有函数可能会引发这些类的异常，除了“标准异常类”中覆盖的标准层次结构的异常之外。我们将在本书的其余部分中涵盖此类异常类的主要情况，这些情况涵盖了提供和可能引发它们的标准库模块的章节。

# ExceptionGroup 和 except*

3.11+ 在某些情况下，例如对某些输入数据执行多个条件验证时，能够一次引发多个异常是有用的。Python 3.11 引入了一种使用 ExceptionGroup 实例一次引发多个异常并使用 **except*** 形式处理多个异常的机制。

要引发 ExceptionGroup，验证代码会将多个异常捕获到一个列表中，然后使用该列表构造一个 ExceptionGroup 并引发它。以下是一些搜索拼写错误和无效单词的代码，并引发包含所有找到错误的 ExceptionGroup 的示例：

```py
`class` GrammarError(Exception):
 *`"""Base exception for grammar checking"""`*
 `def` __init__(self, found, suggestion):
        self.found = found
        self.suggestion = suggestion

`class` InvalidWordError(GrammarError):
  *`"""Misused or nonexistent word"""`*

`class` MisspelledWordError(GrammarError):
 *`"""Spelling error"""`*

invalid_words = {
    'irregardless': 'regardless',
    "ain't": "isn't",
} 
misspelled_words = {
    'tacco': 'taco',
}

`def` check_grammar(s):
    exceptions = []
 `for` word `in` s.lower().split():
 `if` (suggestion := invalid_words.get(word)) `is` `not` `None`:
 exceptions.append(InvalidWordError(word, suggestion))
 `elif` (suggestion := misspelled_words.get(word)) `is` `not` `None``:`
 exceptions.append(MisspelledWordError(word, suggestion))
 `if` exceptions:
 `raise` ExceptionGroup('Found grammar errors', exceptions)
```

下面的代码验证了一个示例文本字符串，并列出了所有找到的错误：

```py
text = "Irregardless a hot dog ain't a tacco"
`try``:`
 check_grammar(text)
except* InvalidWordError `as` iwe`:`
 print('\n'.join(f'{e.found!r} is not a word, use {e.suggestion!r}'
 `for` e `in` iwe`.`exceptions))
except* MisspelledWordError `as` mwe:
  print('\n'.join(f'Found {e.found!r}, perhaps you meant'
                    f' {e.suggestion!r}?'
 `for` e `in` mwe`.`exceptions))
`else``:`
  print('No errors!')
```

给出以下输出：

```py
'irregardless' is not a word, use 'regardless'
"ain't" is not a word, use "isn't"
Found 'tacco', perhaps you meant 'taco'?
```

与 **except** 不同，在找到初始匹配后，**except*** 会继续寻找匹配引发的 ExceptionGroup 中异常类型的其他异常处理程序。

# 错误检查策略

大多数支持异常的编程语言只在极少数情况下引发异常。Python 的重点不同。Python 认为在使程序更简单和更健壮时适当的地方引发异常，即使这样做使异常相当频繁。

## LBYL 与 EAFP

在其他语言中的一种常见习惯用法，有时称为“先入为主”（LBYL），是在尝试操作之前提前检查可能使操作无效的任何内容。这种方法并不理想，原因如下：

+   这些检查可能会降低在一切正常的常见主流情况下的可读性和清晰度。

+   为了进行检查而需要的工作可能会重复操作本身的大部分工作。

+   程序员可能很容易通过省略所需的检查而出错。

+   在你执行检查的时刻和稍后（即使只有一小部分时间！）尝试操作的时刻之间，情况可能会发生变化。

Python 中首选的习惯用法是在 **try** 子句中尝试操作，并在一个或多个 **except** 子句中处理可能引发的异常。这种习惯用法称为[“宁愿请求原谅，也不要事先征求许可”（EAFP）](https://oreil.ly/rGXC9)，这是一个经常引用的格言，被广泛认为是 COBOL 的共同发明者之一的 Rear Admiral Grace Murray Hopper 所创。EAFP 不具有 LBYL 的任何缺陷。以下是一个使用 LBYL 惯用法的函数：

```py
`def` safe_divide_1(x, y):
    `if` y==0:
        print('Divide-by-0 attempt detected')
 `return` `None`
    `else`:
        `return` x/y
```

使用 LBYL，检查首先进行，主流情况在函数末尾有点隐藏。

这是使用 EAFP 惯用法的等效函数：

```py
`def` safe_divide_2(x, y):
    `try`:
        `return` x/y
    `except` ZeroDivisionError:
        print('Divide-by-0 attempt detected')
 `return` `None`
```

使用 EAFP，主流情况在 **try** 子句中前置，异常情况在接下来的 **except** 子句中处理，使整个函数更易于阅读和理解。

EAFP 是一种很好的错误处理策略，但它并非包治百病。特别是，您必须小心不要铺得太宽，捕捉到您没有预期到的错误，因此也没有打算捕捉到的错误。以下是这种风险的典型案例（我们在表 8-2 中介绍了内置函数 getattr）：

```py
`def` trycalling(obj, attrib, default, *args, **kwds):
    `try`:
        `return` getattr(obj, attrib)(*args, **kwds)
    `except` AttributeError:
        `return` default
```

函数 trycalling 的意图是尝试在对象 obj 上调用名为 *attrib* 的方法，但如果 obj 没有这样的方法，则返回 *default*。然而，所编写的函数并不仅仅如此：它还意外隐藏了在所寻找的方法内部引发 AttributeError 的任何错误情况，默默地在这些情况下返回 *default*。这可能会轻易隐藏其他代码中的 bug。要完全达到预期效果，函数必须多加小心：

```py
`def` trycalling(obj, attrib, default, *args, **kwds):
    `try`:
        method = getattr(obj, attrib)
    `except` AttributeError:
        `return` default
    `else`:
        `return` method(*args, **kwds)
```

这个 trycalling 的实现将 getattr 调用与方法调用分开，getattr 调用放置在 **try** 子句中，因此受到 **except** 子句的保护，而方法调用放置在 **else** 子句中，因此可以自由传播任何异常。正确的 EAFP 方法涉及在 **try**/**except** 语句中频繁使用 **else** 子句（这比在整个 **try**/**except** 语句后放置非受保护代码更加明确，因此更符合 Python 风格）。

## 处理大型程序中的错误

在大型程序中，特别容易犯错的地方是将你的**try**/**except**语句设计得过于宽泛，特别是一旦你深信 EAFP 作为一种通用的错误检查策略的强大之后。当一个**try**/**except**组合捕获了太多不同的错误，或者在太多不同的位置可能发生错误时，这种组合就显得过于宽泛了。当你需要准确区分出错原因及位置，并且回溯信息不足以精确定位这些细节（或者你丢弃了回溯信息中的一些或全部信息）时，后者会成为问题。为了有效地处理错误，你必须清楚地区分你预期的（因此知道如何处理）和意外的错误和异常，后者可能表明你程序中存在漏洞。

有些错误和异常并非真正错误，也许甚至不算太反常：它们只是特殊的“边缘”情况，也许比较罕见，但仍然完全可以预料到，你选择通过 EAFP 处理而不是通过 LBYL 来避免 LBYL 的许多内在缺陷。在这种情况下，你应该只是处理这种异常，通常甚至不需要记录或报告它。

# 保持你的 try/except 结构尽可能狭窄

非常小心地保持**try**/**except**结构尽可能狭窄。使用一个小的**try**子句，其中包含少量不调用太多其他函数的代码，并在**except**子句中使用非常具体的异常类元组。如果需要，进一步分析异常的详细信息在你的处理代码中，当你知道这个处理程序无法处理的情况时，尽快**raise**出来。

依赖用户输入或其他不受你控制的外部条件导致的错误和异常总是可以预期的，这正是因为你无法控制它们的根本原因。在这种情况下，你应该集中精力优雅地处理这些异常，记录和记录其确切的性质和细节，并保持你程序的内部状态和持久状态不受损害。尽管当你使用 EAFP 来处理并非真正错误的特殊/边缘情况时，这并不像那么关键，但你的**try**/**except**子句仍应该相对狭窄。

最后，完全意想不到的错误和异常表明你程序的设计或编码存在漏洞。在大多数情况下，处理这类错误的最佳策略是避免使用**try**/**except**，直接让程序带着错误和回溯信息终止运行。（你可能想要在 sys.excepthook 的应用特定钩子中记录这些信息和/或更适当地显示，我们稍后会讨论这个。）在极少数情况下，即使在极端情况下你的程序必须继续运行，也可能适合使用相当宽泛的**try**/**except**语句，其中**try**子句保护调用涵盖大片程序功能的函数，并且广泛的**except**子句。

在长时间运行的程序中，务必记录异常或错误的所有细节到某个持久化存储位置，以便日后研究（同时也向自己报告问题的某些指示，这样您就知道需要进行这样的后续研究）。关键在于确保您能将程序的持久状态还原到某个未受损、内部一致的状态点。使长时间运行的程序能够克服其自身的某些缺陷以及环境逆境的技术被称为[检查点技术](https://oreil.ly/GX4hz)（基本上是周期性保存程序状态，并编写程序以便重新加载保存的状态并从那里继续）和[事务处理](https://oreil.ly/0MaWS)；在本书中我们不进一步介绍它们。

## 记录错误

当 Python 将异常传播到堆栈的顶部而没有找到适用的处理程序时，解释器通常会在终止程序之前将错误回溯打印到进程的标准错误流（sys.stderr）。您可以重新绑定 sys.stderr 到任何可用于输出的类似文件的对象，以便将此信息重定向到更适合您目的的位置。

当您希望在这些情况下更改输出的信息数量和类型时，重新绑定 sys.stderr 是不够的。在这种情况下，您可以将自己的函数分配给 sys.excepthook：当由于未处理的异常而终止程序时，Python 会调用它。在您的异常报告函数中，输出任何有助于诊断和调试问题的信息，并将该信息定向到任何您希望的位置。例如，您可以使用 traceback 模块（在“traceback 模块”中介绍）格式化堆栈跟踪。当您的异常报告函数终止时，程序也会终止。

### 日志模块

Python 标准库提供了丰富而强大的日志记录模块，让您以系统化、灵活的方式组织应用程序的日志记录消息。在极限情况下，您可以编写一整套 Logger 类和其子类；您可以将记录器与 Handler 类（及其子类）的实例或者插入 Filter 类的实例结合起来，以微调决定哪些消息以何种方式记录的标准。

消息由 Formatter 类的实例格式化——消息本身是 LogRecord 类的实例。日志模块甚至包括动态配置功能，通过该功能，您可以通过从磁盘文件读取或者通过专用线程中的专用套接字接收它们，动态设置日志配置文件。

虽然日志模块拥有一个复杂且强大的架构，适用于实现可能在庞大和复杂的软件系统中需要的高度复杂的日志策略和策略，但在大多数应用程序中，您可能只需使用该包的微小子集。首先，**导入** logging。然后，通过将其作为字符串传递给模块的任何函数 debug、info、warning、error 或 critical，按严重性递增地发出您的消息。如果您传递的字符串包含诸如 %s（如 “使用 % 进行遗留字符串格式化” 中所述）的格式说明符，则在字符串之后，传递所有要在该字符串中格式化的值作为进一步的参数。例如，不要调用：

```py
logging.debug('foo is %r' % foo)
```

这会执行格式化操作，无论是否需要；相反，调用：

```py
logging.debug('foo is %r', foo)
```

这会执行格式化，仅在需要时执行（即仅当调用 debug 会导致日志输出时，取决于当前的阈值日志级别）。如果 foo 仅用于日志记录，并且创建 foo 特别耗费计算或 I/O，您可以使用 isEnabledFor 来有条件地执行创建 foo 的昂贵代码：

```py
`if` logging.getLogger().isEnabledFor(logging.DEBUG):
    foo = cpu_intensive_function()
    logging.debug('foo is %r', foo)
```

### 配置日志记录

不幸的是，日志模块不支持在 “字符串格式化” 中涵盖的更易读的格式化方法，而仅支持前面子节中提到的遗留方法。幸运的是，很少需要超出 %s（调用 __str__）和 %r（调用 __repr__）之外的任何格式化说明符。

默认情况下，阈值级别为 WARNING：任何 warning、error 或 critical 函数都会产生日志输出，但 debug 和 info 函数不会。要随时更改阈值级别，请调用 logging.getLogger().setLevel，并将 logging 模块提供的相应常量之一作为唯一参数传递：DEBUG、INFO、WARNING、ERROR 或 CRITICAL。例如，一旦调用：

```py
logging.getLogger().setLevel(logging.DEBUG)
```

所有从 debug 到 critical 的所有日志函数都会产生日志输出，直到再次更改级别。如果稍后调用：

```py
logging.getLogger().setLevel(logging.ERROR)
```

那么只有 error 和 critical 函数会产生日志输出（debug、info 和 warning 不会产生日志输出）；这个条件也会持续，直到再次更改级别，依此类推。

默认情况下，日志输出到进程的标准错误流（sys.stderr，如在表 8-3 中所述），并使用相对简单的格式（例如，每行输出不包括时间戳）。您可以通过实例化适当的处理程序实例、合适的格式化程序实例，并创建和设置一个新的记录器实例来控制这些设置。在简单且常见的情况下，您只需设置这些日志参数一次，然后它们在程序运行期间将保持不变，最简单的方法是通过命名参数调用 logging.basicConfig 函数。只有对 logging.basicConfig 的第一次调用才会产生任何效果，并且只有在调用任何日志函数（例如 debug、info 等）之前调用它才会产生效果。因此，最常见的用法是在程序的最开始调用 logging.basicConfig。例如，在程序的开始处常见的习惯用法如下：

```py
`import` logging
logging.basicConfig(
    format='%(asctime)s %(levelname)8s %(message)s',
    filename='/tmp/logfile.txt', filemode='w')
```

此设置将日志消息写入文件，并以精确的人类可读时间戳进行格式化，后跟右对齐的八字符字段，然后是适当的消息内容。

要获取有关日志模块及其所有功能的详细信息，请务必查阅 Python 的[丰富在线文档](https://oreil.ly/AO1Xa)。

# assert 语句

**assert** 语句允许您在程序中引入“健全性检查”。**assert** 是一个简单语句，其语法如下：

```py
`assert` *`condition`*[, *`expression`*]
```

运行 Python 时，如果使用优化标志（**-O**，如在“命令行语法和选项”中介绍的），**assert** 是一个空操作：编译器不会为其生成任何代码。否则，**assert** 会评估 *condition*。当 *condition* 满足时，**assert** 什么也不做。当 *condition* 不满足时，**assert** 会实例化 AssertionError，并将 *expression* 作为参数（如果没有 *expression*，则不带参数），然后引发该实例³。

**assert** 语句可以是记录程序的有效方法。当您想要声明在程序执行的某一点上已知存在一个重要且不明显的条件 *C*（称为程序的 *不变量*）时，**assert** *C* 通常比仅仅声明 *C* 成立的注释更好。

# 不要过度使用 assert。

除了用于健全性检查程序不变性之外，永远不要将 **assert** 用于其他目的。一个严重但非常常见的错误是在输入或参数的值上使用 **assert**。检查错误的参数或输入最好更加明确，特别是不能使用 **assert**，因为它可以通过 Python 命令行标志变成空操作。

**assert** 的优势在于，当 *C* 实际上 *不* 成立时，**assert** 会立即通过引发 AssertionError 警示问题，如果程序在没有 **-O** 标志的情况下运行。一旦代码彻底调试完成，使用 **-O** 运行它，将 **assert** 转换为一个空操作且不会产生任何开销（**assert** 保留在源代码中以记录不变量）。

# __debug__ 内置变量

在没有 **-O** 选项运行 Python 时，__debug__ 内置变量为 **True**。在使用 **-O** 选项运行 Python 时，__debug__ 为 **False**。此外，在后一种情况下，编译器对唯一保护条件为 __debug__ 的任何 **if** 语句不生成代码。

为了利用这一优化，用 **if** __debug__: 包围仅在 **assert** 语句中调用的函数的定义。这种技术使得在使用 **-O** 运行 Python 时，编译代码更小更快，并通过显示这些函数仅用于执行健全性检查来增强程序的清晰度。

¹ 除了允许多次调用 close 并且无害外：除第一个外的所有调用均不执行操作。

² 这在某种程度上是有争议的：虽然本书的作者认为这是“最佳实践”，但一些人坚决主张应始终避免多重继承，包括在这个特定情况下。

³ 一些第三方框架，如 [pytest](http://docs.pytest.org/en/latest)，实质上提高了 **assert** 语句的实用性。
