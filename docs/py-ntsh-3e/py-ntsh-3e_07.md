# 第七章 模块和包

典型的 Python 程序由多个源文件组成。每个源文件都是一个*模块*，用于重复使用代码和数据。模块通常彼此独立，以便其他程序可以重复使用它们需要的特定模块。有时，为了管理复杂性，开发人员将相关模块组合成一个*包*——这是一个相关模块和子包的层次化树状结构。

一个模块通过使用**import**或**from**语句显式地建立对其他模块的依赖关系。在某些编程语言中，全局变量提供了模块之间的隐藏通道。在 Python 中，全局变量不是所有模块的全局变量，而是单个模块对象的属性。因此，Python 模块始终以显式和可维护的方式通信，通过显式地澄清它们之间的耦合关系。

Python 还支持*扩展模块*——用其他语言如 C、C++、Java、C#或 Rust 编写的模块。对于导入模块的 Python 代码来说，无论模块是纯 Python 还是扩展模块，都无关紧要。你始终可以先用 Python 编写一个模块。如果以后需要更快的速度，可以重构和重写模块的某些部分为更低级别的语言，而不需要改变使用该模块的客户端代码。第二十五章（在线版本[链接](https://oreil.ly/python-nutshell-25)）展示了如何用 C 和 Cython 编写扩展。

本章讨论了模块的创建和加载。还涵盖了如何将模块分组成包、使用[setuptools](https://oreil.ly/c4JWE)来安装包以及如何准备包用于分发；后者更详细地在第二十四章（在线版本[链接](https://oreil.ly/python-nutshell-24)）中介绍。我们在本章结尾讨论了如何最好地管理你的 Python 环境。

# 模块对象

在 Python 中，模块是一个具有任意命名属性的对象，你可以绑定和引用这些属性。Python 中的模块与其他对象一样处理。因此，你可以将一个模块作为参数传递给函数调用。类似地，函数可以返回一个模块作为调用的结果。一个模块，就像任何其他对象一样，可以绑定到一个变量、容器中的一个项目或对象的属性。模块可以是字典中的键或值，并且可以是集合的成员。例如，sys.modules 字典讨论在“模块加载”中，保存模块对象作为其值。Python 中模块能像其他值一样对待的事实，经常被表达为模块是*一等*对象。

## 导入语句

名为*aname*的模块的 Python 代码通常存储在名为*aname.py*的文件中，如“在文件系统中搜索模块”中所述。您可以通过在另一个 Python 源文件中执行**import**语句来使用任何 Python 源文件¹作为模块。**import**具有以下语法：

```py
`import` *`modname`* [`as` *`varname`*][,...]
```

**import**关键字后面跟着一个或多个由逗号分隔的模块说明符。在最简单、最常见的情况下，模块说明符只是*modname*，一个标识符—Python 在**import**语句完成时将其绑定到模块对象的变量。在这种情况下，Python 会寻找相同名称的模块以满足**import**请求。例如，这个语句：

```py
`import` mymodule
```

寻找名为 mymodule 的模块，并将当前作用域中的变量名 mymodule 绑定到模块对象。 *modname*也可以是由点（.）分隔的标识符序列，用于命名包中包含的模块，如“包”中所述。

当**as** *varname*是模块说明符的一部分时，Python 会寻找名为*modname*的模块，并将模块对象绑定到变量*varname*。例如，这样：

```py
`import` mymodule `as` alias
```

寻找名为 mymodule 的模块，并将模块对象绑定到当前作用域中的变量*alias*。 *varname*必须始终是一个简单的标识符。

### 模块体

模块的*body*是模块源文件中的语句序列。没有特殊的语法用于指示源文件是一个模块；如前所述，您可以使用任何有效的 Python 源文件作为模块。模块的 body 在程序的给定运行第一次导入它时立即执行。当 body 开始执行时，模块对象已经被创建，已经在 sys.modules 中绑定到模块对象。模块的（全局）命名空间随着模块的 body 执行而逐渐填充。

### 模块对象的属性

**import**语句创建一个新的命名空间，其中包含模块的所有属性。要访问此命名空间中的属性，请使用模块的名称或别名作为前缀：

```py
`import` mymodule
a = mymodule.f()
```

或者：

```py
`import` mymodule `as` alias
a = alias.f()
```

这样可以减少导入模块所需的时间，并确保只有使用该模块的应用程序才会承担创建模块的开销。

通常，是模块体中的语句绑定模块对象的属性。当模块体中的语句绑定（全局）变量时，绑定的是模块对象的属性。

# 模块体存在以绑定模块的属性

模块体的正常目的是创建模块的属性：**def**语句创建和绑定函数，**class**语句创建和绑定类，赋值语句可以绑定任何类型的属性。为了使代码清晰易懂，请注意在模块体的顶层逻辑级别之外不要做任何其他事情，*除了*绑定模块的属性。

在模块范围定义的 __getattr__ 函数可以动态创建新的模块属性。这样做的一个可能原因是延迟定义创建时间较长的属性；在模块级别的 __getattr__ 函数中定义它们将推迟属性的创建，直到它们实际被引用为止。例如，可以将以下代码添加到*mymodule.py*中，以推迟创建包含前一百万个质数的列表，这需要一些时间来计算：

```py
`def` __getattr__(name):
    `if` name == 'first_million_primes':
        `def` generate_n_primes(n):
            *`# ... code to generate 'n' prime numbers ...`*

        `import` sys
        *`# Look up __name__ in sys.modules to get current module`*
        this_module = sys.modules[__name__]
        this_module.first_million_primes = generate_n_primes(1_000_000)
        `return` this_module.first_million_primes

    `raise` AttributeError(f'module {__name__!r}
	                  f' has no attribute {name!r}')
```

使用模块级别的 __getattr__ 函数对导入*mymodule.py*的时间影响很小，只有那些实际使用 mymodule.first_million_primes 的应用程序会产生创建它的开销。

你也可以在代码主体外绑定模块属性（即在其他模块中）；只需将值分配给属性引用语法*M.name*（其中*M*是任何表达式，其值为模块，*name*是属性名称）。然而，为了清晰起见，最好只在模块自身的主体中绑定模块属性。

**import**语句在创建模块对象后立即绑定一些模块属性，而不是在模块主体执行之前。__dict__ 属性是模块用作其属性命名空间的字典对象。与模块的其他属性不同，__dict__ 在模块中不作为全局变量提供给代码使用。模块中的所有其他属性都是 __dict__ 中的项，并且可以作为全局变量供模块中的代码使用。__name__ 属性是模块的名称，__file__ 是加载模块的文件名；其他 dunder 名称属性包含其他模块元数据。（另请参阅“Package Objects 的特殊属性”了解属性 __path__ 的详细信息，仅适用于包）。

对于任何模块对象*M*，任何对象*x*，以及任何标识符字符串*S*（除了 __dict__），绑定*M.S = x*等同于绑定*M*.__dict__['*S*'] = *x*。像*M.S*这样的属性引用也基本等同于*M*.__dict__['*S*']。唯一的区别在于，当*S*不是*M*.__dict__ 的键时，访问*M*.__dict__['*S*']会引发 KeyError，而访问*M.S*会引发 AttributeError。模块属性也可在模块主体中的所有代码中作为全局变量使用。换句话说，在模块主体中，作为全局变量使用的*S*等同于*M.S*（即*M*.__dict__['*S*']），无论是绑定还是引用（然而，当*S*不是*M*.__dict__ 的键时，将*S*作为全局变量引用会引发 NameError）。

### Python 内置函数

Python 提供了许多内置对象（在 第八章 中介绍）。所有内置对象都是预加载模块 builtins 的属性。当 Python 加载一个模块时，该模块会自动获得一个额外的属性 __builtins__，它指向模块 builtins 或其字典。Python 可能选择其中之一，因此不要依赖于 __builtins__。如果需要直接访问模块 builtins（这种情况很少见），请使用 **import** builtins 语句。当您访问当前模块的本地命名空间和全局命名空间中都找不到的变量时，Python 会在当前模块的 __builtins__ 中查找该标识符，然后引发 NameError。

查找是 Python 唯一用来让您的代码访问内置的机制。您自己的代码可以直接使用访问机制（但请适度使用，否则会影响程序的清晰度和简洁性）。内置的名称不是保留的，也不是硬编码在 Python 本身中的——您可以添加自己的内置或替换正常的内置函数，在这种情况下，所有模块都会看到添加或替换的内置函数。由于 Python 仅在无法解析本地或模块命名空间中的名称时才访问内置，因此通常在其中一个命名空间中定义替代项就足够了。以下简单示例展示了如何使用您自己的函数包装内置函数，使 abs 函数接受一个字符串参数（并返回一个相当任意的字符串变形）：

```py
*`# abs takes a numeric argument; let's make it accept a string as well`*
`import` builtins
_abs = builtins.abs                       *`# save original built-in`*
`def` abs(str_or_num):
    `if` isinstance(str_or_num, str):       *`# if arg is a string`*
        `return` ''.join(sorted(set(str_or_num)))  *`# get this instead`*
    `return` _abs(str_or_num)               *`# call real built-in`*
builtins.abs = abs                        *`# override built-in w/wrapper`*
```

### 模块文档字符串

如果模块体中的第一条语句是一个字符串文字，Python 将该字符串绑定为模块的文档字符串属性，命名为 __doc__。有关文档字符串的更多信息，请参见 “文档字符串”。

### 模块私有变量

模块中没有真正私有的变量。然而，按照约定，每个以单个下划线（_）开头的标识符，例如 _secret，*意味着*是私有的。换句话说，前导下划线告诉客户端代码的程序员不应直接访问该标识符。

开发环境和其他工具依赖于以下划线开头的命名惯例来区分模块中哪些属性是公共的（即模块的接口的一部分），哪些是私有的（即仅在模块内部使用）。

# 遵守“前导下划线表示私有”惯例

在编写使用他人编写的模块的客户端代码时，尊重前导下划线表示私有的约定非常重要。避免在这些模块中使用任何以 _ 开头的属性。未来版本的模块将致力于保持其公共接口，但很可能会更改私有实现细节：私有属性正是为此类细节而设的。

## **from** 语句

Python 的 **from** 语句允许您将模块中的特定属性导入到当前命名空间中。 **from** 有两种语法变体：

```py
`from` *`modname`* `import` *`attrname`* [`as` *`varname`*][,...]
`from` *`modname`* `import` *
```

一个 **from** 语句指定一个模块名，后跟一个或多个用逗号分隔的属性说明符。在最简单和最常见的情况下，属性说明符只是一个标识符 *attrname*，它是 Python 绑定到模块 *modname* 中同名属性的变量。例如：

```py
`from` mymodule `import` f
```

*modname* 也可以是由点（.）分隔的标识符序列，用于指定包内的模块，如“包”中所述。

当 **as** *varname* 是属性说明符的一部分时，Python 从模块获取属性 *attrname* 的值，并将其绑定到变量 *varname*。例如：

```py
`from` mymodule `import` f `as` foo
```

*attrname* 和 *varname* 始终是简单标识符。

您可以选择在 **from** 语句中关键字 **import** 后面的所有属性说明符周围加上括号。当您有许多属性说明符时，这样做可以更优雅地将 **from** 语句的单个逻辑行分成多个逻辑行，而不是使用反斜杠 (\)：

```py
`from` some_module_with_a_long_name `import` (
    another_name, and_another `as` x, one_more, and_yet_another `as` y)
```

### from...import *

直接位于模块体（而不是函数或类体）中的代码可以在 **from** 语句中使用星号 (*)：

```py
`from` mymodule `import` *
```

* 请求将模块 *modname* 的“所有”属性作为全局变量绑定到导入模块中。当模块 *modname* 具有名为 __all__ 的属性时，该属性的值是此类 **from** 语句绑定的属性名称列表。否则，此类 **from** 语句会绑定 *modname* 的所有属性，但排除以下划线开头的属性。

# 谨慎使用“from M import *”在您的代码中

由于 **from** *M* **import** * 可以绑定任意一组全局变量，它可能会产生意想不到的、不希望的副作用，比如隐藏内建变量并重新绑定你仍然需要的变量。几乎不要或者根本不要使用 **from** 的 * 形式，并且仅在明确文档支持这种用法的模块中使用。你的代码最好完全避免使用这种形式，这种形式主要是为了便于在交互式 Python 会话中偶尔使用。

### from 与 import

**import** 语句通常比 **from** 语句更好。当你总是使用 **import** *M* 来访问模块 *M*，并且总是使用显式语法 *M.A* 来访问 *M* 的属性时，你的代码可能会略显冗长，但更加清晰和可读。**from** 的一个好用法是从包中导入特定的模块，正如我们在“包”中所讨论的。在大多数其他情况下，**import** 比 **from** 更好。

### 处理导入失败

如果您正在导入不属于标准 Python 的模块，并希望处理导入失败，可以通过捕获 ImportError 异常来实现。例如，如果您的代码使用第三方的 rich 模块进行可选的输出格式化，但如果该模块未安装，则回退到常规输出，您可以这样导入模块：

```py
try:
 `import` rich
`except` ImportError`:`
 rich = `None`
```

然后，在程序的输出部分，您可以写：

```py
`if` rich `is` `not` `None`:
	... output using rich module features ...
`else`:
	... output using normal print() statements ...
```

# 模块加载

模块加载操作依赖于内置 sys 模块的属性（在 “The sys Module” 中有介绍），并且是通过内置函数 __import__ 实现的。您的代码可以直接调用 __import__，但这在现代 Python 中被强烈不推荐；相反，最好使用 **import** importlib 并调用 importlib.import_module，其参数为模块名称字符串。import_module 返回模块对象或者，如果导入失败，则引发 ImportError。然而，最好对 __import__ 的语义有清楚的理解，因为 import_module 和 **import** 语句都依赖于它。

要导入名为 *M* 的模块，__import__ 首先检查字典 sys.modules，使用字符串 *M* 作为键。当键 *M* 在字典中时，__import__ 返回相应值作为请求的模块对象。否则，__import__ 将 sys.modules[*M*] 绑定到一个具有名称 *M* 的新空模块对象，然后查找正确的初始化（加载）模块的方法，如下一节中关于搜索文件系统的部分所述。

由于这种机制，相对较慢的加载操作仅在程序的给定运行中第一次导入模块时发生。当再次导入模块时，由于 __import__ 快速找到并返回模块在 sys.modules 中的条目，因此模块不会重新加载。因此，在第一次导入后的所有给定模块的导入都非常快速：它们只是字典查找。（要 *强制* 重新加载，请参见 “Reloading Modules”。）

## 内置模块

当加载一个模块时，__import__ 首先检查该模块是否为内置模块。元组 sys.builtin_module_names 列出所有内置模块的名称，但重新绑定该元组不会影响模块加载。当加载内置模块时，就像加载任何其他扩展一样，Python 调用模块的初始化函数。搜索内置模块还会在特定于平台的位置（例如 Windows 的注册表）中查找模块。

## 搜索文件系统中的模块

如果模块*M*不是内置模块，__import__ 将其代码作为文件在文件系统上查找。__import__ 按顺序查看 sys.path 列表的项，这些项是字符串。每个项是目录的路径，或者是流行的[ZIP 格式](https://oreil.ly/QrFfL)中的存档文件的路径。sys.path 在程序启动时使用环境变量 PYTHONPATH 进行初始化（在“Environment Variables”中有介绍），如果存在。sys.path 中的第一个项始终是加载主程序的目录。sys.path 中的空字符串表示当前目录。

您的代码可以更改或重新绑定 sys.path，这些更改会影响 __import__ 搜索以加载模块的目录和 ZIP 存档。更改 sys.path 不会影响已加载的模块（因此已在 sys.modules 中记录的模块）。

如果在启动时 PYTHONHOME 目录中存在带有*.pth*扩展名的文本文件，Python 将文件内容逐行添加到 sys.path 中。*.pth*文件可以包含空行和以字符#开头的注释行；Python 会忽略这些行。*.pth*文件还可以包含**import**语句（Python 在程序开始执行之前执行），但不能包含其他类型的语句。

在每个目录和 sys.path 沿途的 ZIP 存档中查找模块*M*的文件时，Python 按以下顺序考虑这些扩展名：

1.  *.pyd*和*.dll*（Windows）或*.so*（大多数类 Unix 平台），指示 Python 扩展模块。（某些 Unix 方言使用不同的扩展名；例如 HP-UX 上的*.sl*。）在大多数平台上，无法从 ZIP 存档中加载扩展—只能从源或字节码编译的 Python 模块中加载。

1.  *.py*，指示 Python 源模块。

1.  *.pyc*，指示字节码编译的 Python 模块。

1.  当发现*.py*文件时，Python 还会查找名为*__pycache__*的目录。如果找到这样的目录，Python 会在该目录中寻找扩展为*.<tag>.pyc*的文件，其中*<tag>*是查找模块的 Python 版本特定的字符串。

Python 寻找模块*M*文件的最后一个路径是*M**/**__init__.py*：一个名为*M*的目录中名为*__init__.py*的文件，如“Packages”中所述。

找到源文件*M.py*后，Python 会将其编译成*M.<tag>.pyc*，除非字节码文件已经存在且比*M.py*新，并且是由相同版本的 Python 编译的。如果*M.py*是从可写目录编译的，Python 会在需要时创建一个*__pycache__*子目录，并将字节码文件保存到该子目录中，以便将来的运行不会不必要地重新编译它。当字节码文件新于源文件（基于字节码文件内部的时间戳，而不是文件系统中记录的日期）时，Python 不会重新编译模块。

一旦 Python 获得了字节码，无论是通过重新编译还是从文件系统中读取，Python 都会执行模块主体来初始化模块对象。如果模块是一个扩展，Python 将调用模块的初始化函数。

# 谨慎命名项目的 .py 文件

对初学者来说，一个常见的问题是，在编写他们的前几个项目时，他们会意外地将一个 *.py* 文件命名为已导入包或标准库（stdlib）中的同名模块。例如，学习 turtle 模块时很容易犯的一个错误是将程序命名为 *turtle.py*。然后当 Python 尝试从 stdlib 导入 turtle 模块时，它将加载本地模块而不是 stdlib 模块，并且通常会在此后不久引发一些意外的 AttributeErrors（因为本地模块不包含 stdlib 模块中定义的所有类、函数和变量）。不要将项目的 *.py* 文件命名为已导入或 stdlib 模块的同名文件！

你可以使用形如 **python -m** ***testname*** 的命令来检查模块名是否已存在。如果显示消息 'no module *testname*'，那么你可以安全地将模块命名为 *testname.py*。

一般来说，当你熟悉 stdlib 中的模块和常见包名时，你会知道要避免使用哪些名称。

## 主程序

Python 应用程序的执行始于顶级脚本（称为*主程序*），如 “python 程序”中所解释的。主程序像加载任何其他模块一样执行，只是 Python 将字节码保存在内存中，而不是保存到磁盘上。主程序的模块名称是 '__main__'，既是 __name__ 变量（模块属性），也是 sys.modules 中的键。

# 不要将正在使用的 .py 文件作为主程序导入

不要导入与主程序相同的 *.py* 文件。如果这样做，Python 将重新加载模块，并在一个不同的模块对象中执行主体，其 __name__ 也不同。

Python 模块中的代码可以通过检查全局变量 __name__ 是否具有值 '__main__' 来测试模块是否被用作主程序的方式。惯用法是：

```py
`if` __name__ == '__main__':
```

经常用来保护某些代码，以便仅在模块作为主程序运行时执行。如果一个模块仅用于导入，通常在作为主程序运行时执行单元测试，详见 “单元测试和系统测试”。

## 重新加载模块

Python 在程序运行期间仅加载模块一次。在交互式开发中，编辑模块后需要 *重新加载* 模块（一些开发环境提供自动重新加载功能）。

要重新加载模块，请将模块对象（*而不是*模块名称）作为 importlib 模块的 reload 函数的唯一参数传递。importlib.reload(*M*) 确保客户端代码使用重新加载的 *M* 版本，该代码依赖于 import *M* 并使用 *M.A* 语法访问属性。但是，importlib.reload(*M*) 对绑定到 *M* 属性先前值的其他现有引用（例如，使用 **from** 语句）没有影响。换句话说，已绑定的变量保持原样，不受重新加载的影响。reload 无法重新绑定这些变量是使用 **import** 而不是 **from** 的进一步动机。

reload 不是递归的：当重新加载模块 *M* 时，这并不意味着通过 *M* 导入的其他模块也会被重新加载。您必须通过显式调用 reload 来重新加载您修改过的每个模块。一定要考虑任何模块引用依赖关系，以便按正确的顺序进行重新加载。

## 循环导入

Python 允许您指定循环导入。例如，可以编写一个包含 **import** b 的模块 *a.py*，而模块 *b.py* 包含 **import** a。

如果因某种原因决定使用循环导入，则需要了解循环导入的工作原理，以避免代码中的错误。

# 避免循环导入

实际上，几乎总是更好地避免循环导入，因为循环依赖是脆弱且难以管理的。

假设主脚本执行 **import** a。如前所述，此 **import** 语句在 sys.modules['a'] 中创建一个新的空模块对象，然后模块 a 的体开始执行。当 a 执行 **import** b 时，这将在 sys.modules['b'] 中创建一个新的空模块对象，然后模块 b 的体开始执行。直到 b 的模块体完成，a 的模块体才能继续执行。

现在，当 b 执行 **import** a 时，**import** 语句发现 sys.modules['a'] 已经绑定，因此将模块 b 中的全局变量 a 绑定到模块 a 的模块对象。由于当前阻塞了 a 的模块体执行，模块 a 此时通常只部分填充。如果 b 的模块体中的代码尝试访问尚未绑定的模块 a 的某个属性，将导致错误。

如果保留循环导入，必须仔细管理每个模块绑定其自己全局变量的顺序，导入其他模块并访问其他模块的全局变量。通过将语句分组到函数中，并按照受控顺序调用这些函数，而不是仅仅依赖模块体顶层语句的顺序执行，可以更好地控制发生的顺序。通过移动导入远离模块范围并进入引用函数，而不是确保用于处理循环依赖的防爆顺序，更容易去除循环依赖。

# sys.modules 条目

__import__ 从不绑定除模块对象以外的任何值到 sys.modules 中。但是，如果 __import__ 在 sys.modules 中找到一个已有的条目，则返回该值，无论它是什么类型。**import**和**from**语句依赖于 __import__，因此它们也可以使用非模块对象。

## 自定义导入器

Python 提供的另一种高级且不经常需要的功能是能够更改某些或所有**import**和**from**语句的语义。

### 重新绑定 __import__

您可以重新绑定内置模块的 __import__ 属性到您自己的自定义导入器函数，例如使用在“Python 内置”中显示的通用内置包装技术。这样的重新绑定会影响所有在重新绑定之后执行的**import**和**from**语句，因此可能会产生不希望的全局影响。通过重新绑定 __import__ 构建的自定义导入器必须实现与内置 __import__ 相同的接口和语义，特别是负责支持 sys.modules 的正确使用。

# 避免重新绑定内置的 __import__

虽然重新绑定 __import__ 可能最初看起来是一种有吸引力的方法，但在大多数需要自定义导入器的情况下，通过*import hooks*（接下来讨论）实现它们会更好。

### 导入钩子

Python 提供了丰富的支持，可以选择性地更改导入行为的细节。自定义导入器是一种高级且不经常调用的技术，但某些应用可能需要它们，例如从 ZIP 文件以外的存档、数据库、网络服务器等导入代码。

对于这种高度高级的需求，最合适的方法是将*importer factory*可调用项记录为模块 sys 的 meta_path 和/或 path_hooks 属性中的项目，详细信息请参见[PEP 451](https://oreil.ly/9Wd9A)。这是 Python 如何连接标准库模块 zipimport 以允许无缝导入来自 ZIP 文件的模块的方式，如前所述。要实现对 sys.path_hooks 和相关属性的实质性使用，必须全面研究 PEP 451 的详细内容，但以下是一个玩具级别的示例，可帮助理解可能的用途，如果您曾经需要的话。

假设我们在开发某个程序的首个大纲时，希望能够使用尚未编写的模块的**import**语句，只会得到消息（以及空模块）作为结果。我们可以通过编写一个自定义导入器模块来实现这样的功能（暂且不考虑与包相关的复杂性，仅处理简单模块）：

```py
`import` sys, types
`class` ImporterAndLoader:
     *`"""importer and loader can be a single class"""`*
     fake_path = '!dummy!'
     `def` __init__(self, path):
         *`# only handle our own fake-path marker`*
         `if` path != self.fake_path:
             `raise` ImportError
     `def` find_module(self, fullname):
         *`# don't even try to handle any qualified module name`*
         `if` '.' `in` fullname:
             `return` `None`
         `return` self
     `def` create_module(self, spec):
         *`# returning None will have Python fall back and`*
 *`# create the module "the default way"`*
 `return` `None`
     `def` exec_module(self, mod):
         *`# populate the already initialized module`*
         *`# just print a message in this toy example`*
         print(f'NOTE: module {mod!r} not yet written')
sys.path_hooks.append(ImporterAndLoader)
sys.path.append(ImporterAndLoader.fake_path)
`if` __name__ == '__main__':      *`# self-test when run as main script`*
    `import` missing_module       *`# importing a simple *missing* module`*
    print(missing_module)       *`# ...should succeed`*
    print(sys.modules.get('missing_module'))  *`# ...should also succeed`*
```

我们刚刚编写了 create_module 的简单版本（在本例中仅返回 None，请求系统以“默认方式”创建模块对象）和 exec_module 的简单版本（接收已初始化带有 dunder 属性的模块对象，其任务通常是适当地填充它）。

作为替代方案，我们还可以使用强大的新模块规范概念，详见 PEP 451。然而，这需要标准库模块 importlib；对于这个玩具示例，我们不需要额外的功能。因此，我们选择实现 find_module 方法，虽然现在已经被弃用，但仍然可以用于向后兼容。

# 包

正如本章开头所提到的，*包* 是包含其他模块的模块。包中的一些或所有模块可以是*子包*，形成一个分层的树形结构。包 *P* 通常位于 sys.path 中某个目录的名为 *P* 的子目录中。包也可以存在于 ZIP 文件中；在本节中，我们解释了包存在于文件系统中的情况，但包存在于 ZIP 文件中的情况类似，依赖于 ZIP 文件内部的分层文件系统结构。

包 *P* 的模块体在文件 *P/__init__.py* 中。此文件*必须*存在（除了在[PEP 420](https://oreil.ly/cVzGw)中描述的命名空间包的情况下），即使它是空的（代表空的模块体），以便告诉 Python 目录 *P* 确实是一个包。当你首次导入包（或包的任何模块）时，Python 会加载包的模块体，就像加载任何其他 Python 模块一样。目录 *P* 中的其他 *.py* 文件是包 *P* 的模块。包含 *__init__.py* 文件的 *P* 的子目录是 *P* 的子包。嵌套可以无限进行。

在包 *P* 中，你可以将名为 *M* 的模块导入为 *P.M**.*。更多的点号可以让你在层次化的包结构中导航。（一个包的模块体总是在任何包中的模块之前加载。）如果你使用 **import** *P.M* 的语法，变量 *P* 将绑定到包 *P* 的模块对象，并且对象 *P* 的属性 *M* 绑定到模块 *P.M*。如果你使用 **import** *P.M* as *V* 的语法，变量 *V* 直接绑定到模块 *P.M*。

使用 **from** *P* **import** *M* 来从包 *P* 导入特定模块 *M* 是一种完全可以接受的、而且确实是非常推荐的做法：在这种情况下，**from** 语句是完全 OK 的。**from** *P* **import** *M* **as** *V* 也是可以的，与 **import** *P.M* **as** *V* 完全等效。你也可以使用*相对*路径：也就是说，包 *P* 中的模块 *M* 可以使用 **from** . **import** X 导入它的“兄弟”模块 *X*（也在包 *P* 中）。

# 在包中的模块之间共享对象

在包 *P* 中，最简单、最清晰的共享对象（例如函数或常量）的方法是将共享对象分组到一个传统上命名为 *common.py* 的模块中。这样，你可以在包中的每个模块中使用 **from** . **import** common 来访问一些共享对象，然后引用这些对象为 common.*f*、common.*K* 等。

## 包对象的特殊属性

包 *P* 的 __file__ 属性是一个字符串，表示 *P* 的模块主体的路径，即文件 *P/__init__.py* 的路径。*P* 的 __package__ 属性是 *P* 的包名。

包 *P* 的模块主体 - 即文件 *P/__init__.py* 中的 Python 源代码 - 可以选择性地设置一个名为 __all__ 的全局变量（就像任何其他模块一样），以控制如果其他 Python 代码执行语句 **from** *P* **import** * 会发生什么。特别是，如果未设置 __all__，**from** *P* **import** * 不会导入 *P* 的模块，而只会导入在 *P* 的模块主体中设置的没有前导 _ 的名称。无论如何，这都*不*推荐使用。

包 *P* 的 __path__ 属性是一个字符串列表，其中包含加载 *P* 的模块和子包的目录路径。最初，Python 将 __path__ 设置为一个列表，其中仅有一个元素：包含文件 *__init__.py* 的目录的路径，该文件是包的模块主体。您的代码可以修改此列表，以影响对此包的模块和子包的未来搜索。这种高级技术很少必要，但在您想要将包的模块放置在多个目录中时可能会很有用（然而，命名空间包是实现此目标的通常方法）。

## 绝对导入与相对导入

如前所述，**import** 语句通常期望在 sys.path 的某处找到其目标，这种行为称为*绝对*导入。或者，您可以显式使用*相对*导入，意味着从当前包中导入对象。使用相对导入可以使您更轻松地重构或重新组织包中的子包。相对导入使用以一个或多个点开头的模块或包名称，并且仅在 **from** 语句中可用。**from** . **import** *X* 在当前包中查找名为 *X* 的模块或对象；**from** .*X* **import** *y* 在当前包的模块或子包 *X* 中查找名为 *y* 的模块或对象。如果您的包有子包，它们的代码可以通过在 **from** 和 **import** 之间放置的模块或子包名称的起始处使用多个点来访问包中的更高级对象。每一个额外的点都会提升目录层次结构一级。对这个特性的过度使用可能会损害您代码的清晰度，因此请谨慎使用，只在必要时使用。

# 分发工具（distutils）和 setuptools

Python 模块、扩展和应用可以以多种形式打包和分发：

压缩归档文件

通常的 *.zip*、*.tar.gz*（也称为 *.tgz*）、*.tar.bz2* 或 *.tar.xz* 文件 - 所有这些形式都是可移植的，还有许多其他形式的文件和目录树的压缩归档存在

自解包或自安装的可执行文件

通常用于 Windows 的 *.exe* 文件

自包含、即时运行的可执行文件，无需安装

例如，对于 Windows 是*.exe*，在 Unix 上是带有短脚本前缀的 ZIP 存档文件，对于 Mac 是*.app*，等等

平台特定的安装程序

例如，在许多 Linux 发行版上是*.rpm*和*.srpm*，在 Debian GNU/Linux 和 Ubuntu 上是*.deb*，在 macOS 上是*.pkg*。

Python Wheels

流行的第三方扩展，详见下文

# Python Wheels

一个 Python *wheel* 是一个包含结构化元数据和 Python 代码的存档文件。Wheels 提供了一个出色的方式来打包和分发你的 Python 包，而且 setuptools（通过**pip install wheel** 轻松安装 wheel 扩展）与它们无缝配合。在[PythonWheels.com](http://pythonwheels.com)和第二十四章（在线版本[在此](https://oreil.ly/python-nutshell-24)）了解更多信息。

当你将一个包作为一个自安装可执行文件或平台特定的安装程序进行分发时，用户只需运行安装程序。如何运行这样的程序取决于平台，但程序是用哪种语言编写的并不重要。我们在[第二十四章](https://oreil.ly/python-nutshell-24)中介绍了为各种平台构建自包含可运行可执行文件的方法。

当你将一个包作为一个存档文件或一个解压但不安装自身的可执行文件进行分发时，重要的是这个包是用 Python 编写的。在这种情况下，用户必须首先将存档文件解压到某个适当的目录中，比如在 Windows 机器上是 *C:\Temp\MyPack*，在类 Unix 机器上是 *~/MyPack*。在提取的文件中应该有一个脚本，按照惯例命名为 *setup.py*，它使用 Python 设施称为 *distribution utilities*（现在已经被弃用，但仍然有效的标准库包 distutils^(2））或者更好的是，更流行、现代和强大的第三方包 [setuptools](https://oreil.ly/MHZby)。然后，分发的包几乎和自安装可执行文件一样容易安装；用户只需打开一个命令提示窗口，切换到解压存档文件的目录，然后运行，例如：

```py
C:\Temp\MyPack> `python` `setup``.``py` `install`
```

（另一种常用的选择是使用 pip；我们将立即描述它。）运行此 **install** 命令的 *setup.py* 脚本会根据包的作者在设置脚本中指定的选项将包安装为用户的 Python 安装的一部分。当然，用户需要适当的权限才能写入 Python 安装目录的目录，因此可能也需要像 sudo 这样提高权限的命令；或者更好的做法是，您可以安装到虚拟环境中，如下一节所述。distutils 和 setuptools 在用户运行 *setup.py* 时，默认会打印一些信息。在 **install** 命令之前包含 **--quiet** 选项可以隐藏大部分详细信息（用户仍然可以看到错误消息，如果有的话）。以下命令可以详细了解 distutils 或 setuptools，具体取决于包作者在其 *setup.py* 中使用的工具集：

```py
C:\Temp\MyPack> `python` `setup``.``py` `-``-``help`
```

这个过程的另一个选择，也是现在安装包的首选方式，是使用与 Python 随附的优秀安装程序 pip。pip——“pip installs packages”的递归缩写——在大多数情况下使用起来非常简单，但[在线](https://oreil.ly/G7zMK)有大量文档支持。**pip install** ***package*** 会查找*package*的在线版本（通常在巨大的 [PyPI](https://oreil.ly/PGIim) 仓库中，在撰写本文时托管了超过 400,000 个包），下载并为您安装它（如果已激活虚拟环境，则在其中安装；有关详细信息，请参阅下一节）。这本书的作者已经使用这种简单而强大的方法安装了超过 90% 的安装已经相当长时间了。

即使您已经将包本地下载（比如到 */tmp/mypack*），出于任何原因（也许它不在 PyPI 上，或者您正在尝试一个尚未发布的实验版本），pip 仍然可以为您安装它：只需运行 **pip install --no-index --find-links=/tmp/mypack**，pip 将完成其余操作。

# Python 环境

典型的 Python 程序员同时在多个项目上工作，每个项目都有自己的依赖列表（通常是第三方库和数据文件）。当所有项目的依赖项都安装到同一个 Python 解释器时，很难确定哪些项目使用了哪些依赖项，并且无法处理使用某些依赖项冲突版本的项目。

早期的 Python 解释器是基于这样的假设构建的：每台计算机系统都安装有“一个 Python 解释器”，用于在该系统上运行任何 Python 程序。操作系统发行版很快开始将 Python 包含在它们的基础安装中，但由于 Python 一直在积极开发中，用户经常抱怨他们希望使用比其操作系统提供的更更新的语言版本。

出现了让系统安装多个版本的语言的技术，但第三方软件的安装仍然是非标准和具有侵入性的。通过引入*site-packages*目录作为添加到 Python 安装的模块的存储库来缓解这个问题，但仍然不可能使用相同的解释器维护具有冲突要求的多个项目。

习惯于命令行操作的程序员熟悉*shell 环境*的概念。在进程中运行的 shell 程序有一个当前目录，您可以使用 shell 命令设置变量（与 Python 命名空间非常相似），以及各种其他进程特定状态数据。Python 程序可以通过 os.environ 访问 shell 环境。

如“环境变量”中所述，shell 环境的各个方面都会影响 Python 的运行。例如，PATH 环境变量决定了哪个程序会对**python**和其他命令做出响应。你可以把影响 Python 运行的 shell 环境的各个方面称为你的*Python 环境*。通过修改它，你可以确定哪个 Python 解释器会对**python**命令做出响应，哪些包和模块在特定名称下可用，等等。

# 不要将系统的 Python 用于系统

我们建议控制你的 Python 环境。特别是，不要在系统分发的 Python 上构建应用程序。相反，独立安装另一个 Python 发行版，并调整你的 shell 环境，以便**python**命令运行你本地安装的 Python，而不是系统的 Python。

## 进入虚拟环境

pip 实用程序的引入为在 Python 环境中安装（并且首次卸载）包和模块提供了一种简单的方法。修改系统 Python 的*site-packages*仍然需要管理员权限，因此 pip 也需要（虽然它可以选择安装到*site-packages*之外的地方）。安装在中央*site-packages*中的模块对所有程序都可见。

缺失的部分是能够对 Python 环境进行受控更改，以指导使用特定的解释器和一组特定的 Python 库。这正是*虚拟环境*（*virtualenvs*）所提供的功能。基于特定 Python 解释器创建的虚拟环境会复制或链接到该解释器安装的组件。关键是，每个虚拟环境都有自己的*site-packages*目录，你可以将你选择的 Python 资源安装到其中。

创建虚拟环境比安装 Python *简单得多*，并且需要的系统资源远远少于后者（典型的新创建的虚拟环境占用不到 20 MB）。您可以随时轻松创建和激活虚拟环境，并且同样轻松地取消激活和销毁它们。在其生命周期内，您可以任意次数地激活和取消激活虚拟环境，并且必要时使用 pip 来更新已安装的资源。当您完成时，删除其目录树会回收虚拟环境占用的所有存储空间。虚拟环境的生命周期可以是几分钟或几个月。

## 什么是虚拟环境？

虚拟环境本质上是您的 Python 环境的一个自包含子集，您可以根据需要随时切换。对于 Python 3.*x* 解释器，它包括，除其他外，一个包含 Python 3.*x* 解释器的 *bin* 目录，以及一个包含预安装版本的 easy-install、pip、pkg_resources 和 setuptools 的 *lib/python3.x/site-packages* 目录。维护这些重要的分发相关资源的独立副本使您可以根据需要更新它们，而不是强迫您依赖基本 Python 发行版。

虚拟环境在（在 Windows 上）或符号链接到（在其他平台上）Python 发行文件上有其自己的副本。它调整了 sys.prefix 和 sys.exec_prefix 的值，从而解释器和各种安装工具确定了一些库的位置。这意味着 pip 可以将依赖项安装在与其他环境隔离的情况下，在虚拟环境的 *site-packages* 目录中。实际上，虚拟环境重新定义了运行 **python** 命令时的解释器以及可用于解释器的大多数库，但保留了 Python 环境的大多数方面（如 PYTHONPATH 和 PYTHONHOME 变量）。由于其更改会影响您的 shell 环境，因此它们也会影响您运行命令的任何子 shell。

使用单独的虚拟环境，您可以例如，测试项目中相同库的两个不同版本，或者使用多个 Python 版本测试您的项目。您还可以为您的 Python 项目添加依赖项，而无需任何特殊权限，因为您通常在您具有写权限的地方创建您的虚拟环境。

处理虚拟环境的现代方法是使用标准库的 venv 模块：只需运行 **python -m venv** *envpath*。

## 创建和删除虚拟环境

命令 **python -m venv** *envpath* 创建一个基于运行该命令的 Python 解释器的虚拟环境（在必要时还会创建 *envpath* 目录）。您可以提供多个目录参数来创建多个虚拟环境（使用相同的 Python 解释器），然后可以在每个虚拟环境中安装不同的依赖项集。venv 可以接受多个选项，如表 7-1 所示。

表 7-1\. venv 选项

| 选项 | 目的 |
| --- | --- |
| **--clear** | 在安装虚拟环境之前移除任何现有目录内容 |
| **--copies** | 在类 Unix 平台上使用复制方式安装文件，这是默认使用符号链接的系统 |
| **--h** 或 **--help** | 打印出命令行摘要和可用选项列表 |
| **--symlinks** | 在平台上使用符号链接安装文件，这是默认复制的系统 |
| **--system-site-packages** | 将标准系统 *site-packages* 目录添加到环境的搜索路径中，使基础 Python 中已安装的模块在环境内可用 |
| **--upgrade** | 在虚拟环境中安装当前正在运行的 Python，替换最初创建环境的版本 |
| **--without-pip** | 阻止调用 ensurepip 来将 pip 安装器引导到环境中的常规行为 |

# 知道您正在使用哪个 Python

当您在命令行输入 **python** 命令时，您的 shell 有一些规则（在 Windows、Linux 和 macOS 中有所不同），决定您运行的程序。如果您清楚这些规则，您就始终知道您正在使用哪个解释器。

使用 **python -m venv** *directory_path* 命令创建虚拟环境可以保证它基于与创建时所用解释器相同的 Python 版本。类似地，使用 **python -m pip** *package_name* 将为与 **python** 命令关联的解释器安装包。激活虚拟环境会改变与 **python** 命令的关联：这是确保包安装到虚拟环境中的最简单方法。

下面的 Unix 终端会话显示了虚拟环境的创建及创建的目录结构。*bin* 子目录的列表显示了这个特定用户默认使用的解释器安装在 */usr/local/bin* 中。³

```py
$ python3 -m venv /tmp/tempenv
$ tree -dL 4 /tmp/tempenv
/tmp/tempenv
|--- bin
|--- include
|___ lib
     |___ python3.5
          |___ site-packages
               |--- __pycache__
               |--- pip
               |--- pip-8.1.1.dist-info
               |--- pkg_resources
               |--- setuptools
               |___ setuptools-20.10.1.dist-info

11 directories
$ ls -l /tmp/tempenv/bin/
total 80
-rw-r--r-- 1 sh wheel 2134 Oct 24 15:26 activate
-rw-r--r-- 1 sh wheel 1250 Oct 24 15:26 activate.csh
-rw-r--r-- 1 sh wheel 2388 Oct 24 15:26 activate.fish
-rwxr-xr-x 1 sh wheel  249 Oct 24 15:26 easy_install
-rwxr-xr-x 1 sh wheel  249 Oct 24 15:26 easy_install-3.5
-rwxr-xr-x 1 sh wheel  221 Oct 24 15:26 pip
-rwxr-xr-x 1 sh wheel  221 Oct 24 15:26 pip3
-rwxr-xr-x 1 sh wheel  221 Oct 24 15:26 pip3.5
lrwxr-xr-x 1 sh wheel    7 Oct 24 15:26 python->python3
lrwxr-xr-x 1 sh wheel   22 Oct 24 15:26 python3->/usr/local/bin/python3
```

删除虚拟环境就像删除它所在的目录一样简单（以及树中的所有子目录和文件：在类 Unix 系统中使用 **rm -rf** *envpath*）。易于删除是使用虚拟环境的一个有用方面。

venv 模块包括一些功能，帮助编程创建定制环境（例如，在环境中预安装某些模块或执行其他创建后步骤）。在线文档提供了详细说明 [online](https://oreil.ly/DVwfT)；我们在本书中不再详细介绍 API。

## 使用虚拟环境

要使用虚拟环境，需要在正常的 shell 环境中 *activate* 它。一次只能激活一个虚拟环境，激活不像函数调用那样可以“堆叠”。激活告诉你的 Python 环境使用虚拟环境的 Python 解释器和 *site-packages*（以及解释器的完整标准库）。当你想要停止使用这些依赖时，取消激活虚拟环境，你的标准 Python 环境将再次可用。虚拟环境目录树继续存在，直到被删除，因此可以随意激活和取消激活。

在 Unix-based 环境中激活虚拟环境需要使用 **source** shell 命令，以便激活脚本中的命令可以修改当前的 shell 环境。简单运行脚本会导致其命令在子 shell 中执行，当子 shell 终止时，更改将会丢失。对于 bash、zsh 和类似的 shell，你可以使用以下命令激活位于路径 *envpath* 的环境：

```py
$ source *envpath*/bin/activate
```

或者：

```py
$ *. envpath*/bin/activate
```

其他 shell 的用户可以使用同一目录中的 *activate.csh* 和 *activate.fish* 脚本来获得支持。在 Windows 系统上，使用 *activate.bat*（或者如果使用 Powershell，则使用 *Activate.ps1*）：

```py
C:\> *envpath*/Scripts/activate.bat
```

激活会执行许多操作。最重要的是：

+   将虚拟环境的 *bin* 目录添加到 shell 的 PATH 环境变量的开头，这样它的命令优先于已存在于 PATH 中同名的任何内容。

+   定义了一个 deactivate 命令，用于取消激活的所有效果，并将 Python 环境恢复到其原始状态。

+   修改 shell 提示符，在开头包含虚拟环境的名称。

+   定义了一个 VIRTUAL_ENV 环境变量，指向虚拟环境的根目录（脚本可以使用此变量来检查虚拟环境）。

由于这些操作的结果，一旦激活了虚拟环境，**python** 命令将运行与该虚拟环境关联的解释器。解释器可以看到安装在该环境中的库（模块和包），而 pip——现在是虚拟环境中的 pip，因为安装模块也安装了命令到虚拟环境的 *bin* 目录——默认将新的包和模块安装到环境的 *site-packages* 目录中。

对于刚接触虚拟环境的人来说，需要理解虚拟环境与任何项目目录都无关。完全可以在同一个虚拟环境中处理多个项目，每个项目都有自己的源代码树。激活虚拟环境后，你可以根据需要在文件系统中移动，完成编程任务，使用相同的库（因为虚拟环境确定了 Python 环境）。

当你想要禁用虚拟环境并停止使用那组资源时，只需执行命令 **deactivate**。这将撤销激活时所做的更改，将虚拟环境的 *bin* 目录从你的 PATH 中移除，因此 **python** 命令将再次运行你通常的解释器。只要不删除它，虚拟环境将保持可用状态供将来使用：只需重复执行命令来激活它。

# 不要在 Windows 的虚拟环境中使用 py –3.x

Windows py 启动器对虚拟环境的支持提供了混合支持。它使得使用特定 Python 版本定义虚拟环境变得非常简单，例如使用以下命令：

```py
C:\> `py` `-``3.7` `-``m` `venv` `C``:``\``path``\``to``\``new_virtualenv`
```

这将创建一个新的虚拟环境，运行已安装的 Python 3.7 版本。

一旦激活，你可以使用 **python** 命令或不指定版本的裸露 **py** 命令在虚拟环境中运行 Python 解释器。然而，如果你使用版本选项指定 **py** 命令，即使它是用来构建虚拟环境的相同版本，你也将不会运行 *virtualenv* Python。相反，你将运行相应的系统安装版本的 Python。

## 管理依赖要求

由于虚拟环境设计为与 pip 安装相辅相成，因此不足为奇，pip 是在虚拟环境中维护依赖项的首选方式。由于 pip 已经有了广泛的文档，我们在这里只提到足够展示它在虚拟环境中的优势。创建了虚拟环境、激活了它并安装了依赖项后，你可以使用 **pip freeze** 命令来了解这些依赖项的确切版本：

```py
(tempenv) $ pip freeze
appnope==0.1.0
decorator==4.0.10
ipython==5.1.0
ipython-genutils==0.1.0
pexpect==4.2.1
pickleshare==0.7.4
prompt-toolkit==1.0.8
ptyprocess==0.5.1
Pygments==2.1.3
requests==2.11.1
simplegeneric==0.8.1
six==1.10.0
traitlets==4.3.1
wcwidth==0.1.7
```

如果你将此命令的输出重定向到一个名为 *filename* 的文件中，你可以使用命令 **pip install -r** *filename* 在另一个虚拟环境中重新创建相同的依赖项集。

在为他人使用的代码分发时，Python 开发人员通常包含一个列出必要依赖项的 *requirements.txt* 文件。当从 PyPI 安装软件时，pip 会安装任何指示的依赖项以及你请求的包。在开发软件时，拥有一个要求文件也很方便，因为你可以使用它向活跃的虚拟环境添加必要的依赖项（除非它们已经安装），只需简单地执行 **pip install -r requirements.txt**。

要在多个虚拟环境中保持相同的依赖关系集，使用同一个要求文件向每个虚拟环境添加依赖项。这是一个方便的方法来开发可以在多个 Python 版本上运行的项目：为每个所需版本创建基于的虚拟环境，然后在每个环境中从同一个要求文件安装。虽然前面的示例使用了由 **pip freeze** 生成的精确版本化的依赖规范，但在实践中，你可以以非常复杂的方式指定依赖关系和版本要求；详细信息请参阅 [文档](https://oreil.ly/wB9LB)。

## 其他环境管理解决方案

Python 虚拟环境专注于提供一个隔离的 Python 解释器，你可以在其中为一个或多个 Python 应用程序安装依赖项。最初创建和管理 virtualenv 的方式是使用 [virtualenv](https://oreil.ly/bUfe0) 包。它具有广泛的功能，包括从任何可用的 Python 解释器创建环境的能力。现在由 Python Packaging Authority 团队维护，其部分功能已提取为标准库 venv 模块，但如果需要更多控制，还是值得了解 virtualenv。

[pipenv](https://oreil.ly/vfi9I) 包是另一个用于 Python 环境的依赖管理器。它维护虚拟环境，其内容记录在名为 *Pipfile* 的文件中。类似于类似的 JavaScript 工具，通过 *Pipfile.lock* 文件提供确定性环境，允许部署与原始安装完全相同的依赖项。

conda，在 “Anaconda and Miniconda” 中提到，具有更广泛的范围，可以为任何语言提供包、环境和依赖管理。conda 使用 Python 编写，在基础环境中安装自己的 Python 解释器。而标准的 Python virtualenv 通常使用创建时的 Python 解释器；在 conda 中，Python 本身（当其包含在环境中时）只是另一个依赖项。这使得在需要时更新环境中使用的 Python 版本成为可能。如果愿意，你也可以在基于 Python 的 conda 环境中使用 pip 安装软件包。conda 可以将环境内容转储为 YAML 文件，你可以使用该文件在其他地方复制环境。

因其额外的灵活性，加上由其创始者 Anaconda, Inc.（前身为 Continuum）主导的全面开源支持，conda 在学术环境中被广泛使用，特别是在数据科学与工程、人工智能和金融分析领域。它从所谓的 *channels* 安装软件。Anaconda 维护的默认 channel 包含各种软件包，第三方维护专门的 channel（如生物信息学软件的 *bioconda* channel）。还有一个基于社区的 [*conda-forge* channel](https://oreil.ly/fEBZo)，欢迎任何希望加入并添加软件的人。在 [Anaconda.org](https://anaconda.org) 注册账户可以让你创建自己的 channel 并通过 *conda-forge* channel 分发软件。

## 使用虚拟环境的最佳实践

关于如何最佳管理虚拟环境的建议非常少，但有几个 sound tutorials：任何一个好的搜索引擎都可以让你访问到最新的教程。不过，我们可以提供一些希望能帮助你充分利用虚拟环境的简单建议。

当您在多个 Python 版本中使用相同的依赖关系时，在环境名称中指示版本并使用共同的前缀是有用的。因此，对于项目*mutex*，您可以维护称为*mutex_39*和*mutex_310*的环境，以在两个不同版本的 Python 下进行开发。当环境名称在您的 shell 提示符中显而易见时（记住，您可以看到环境名称），测试使用错误版本的风险较小。您可以使用共同的需求来维护依赖项，以控制在两者中的资源安装。

将需求文件保留在源代码控制下，而不是整个环境。有了需求文件，重新创建虚拟环境很容易，它仅依赖于 Python 版本和需求。您可以分发您的项目，并让用户决定在哪些 Python 版本上运行它并创建适当的虚拟环境。

将您的虚拟环境保持在项目目录之外。这样可以避免显式强制源代码控制系统忽略它们。它们存储在哪里真的无关紧要。

您的 Python 环境独立于文件系统中项目的位置。您可以激活一个虚拟环境，然后切换分支并在变更控制的源代码树中移动以在任何方便的地方使用它。

要调查一个新的模块或包，创建并激活一个新的虚拟环境，然后**pip install**您感兴趣的资源。您可以尽情地在这个新环境中玩耍，确信您不会将不需要的依赖项安装到其他项目中。

在虚拟环境中进行实验可能需要安装不是当前项目需求的资源。与其“污染”您的开发环境，不如分叉它：从相同的需求创建一个新的虚拟环境，再加上测试功能。稍后，为了使这些更改永久化，使用变更控制从分叉分支合并您的源代码和需求更改回来。

如果您愿意，您可以基于 Python 的调试版本创建虚拟环境，从而可以访问关于您的 Python 代码性能（当然还有解释器本身）的丰富信息。

开发虚拟环境还需要变更控制，而虚拟环境创建的便利性在这方面也起到了帮助作用。假设你最近发布了模块的 4.3 版本，并且你想要用其两个依赖项的新版本来测试你的代码。如果你足够有技巧，可以说服 pip 替换现有虚拟环境中的依赖项副本。不过，更简单的方法是使用源代码控制工具分支你的项目，更新依赖项，并基于更新后的需求创建一个全新的虚拟环境。原始虚拟环境保持不变，你可以在不同的虚拟环境之间切换，以调查可能出现的任何迁移问题的特定方面。一旦调整了代码，使所有测试都通过了更新的依赖项，你就可以提交你的代码和需求更改，并合并到版本 4.4 完成更新，通知同事你的代码已准备好使用更新的依赖版本。

虚拟环境并不能解决所有 Python 程序员的问题：工具总是可以变得更复杂，或者更通用。但是，天哪，虚拟环境确实有效，我们应该尽可能充分利用它们。

¹ 我们的一位技术审阅员报告说在 Windows 上的 *.pyw* 文件是一个例外。

² 在 Python 3.12 中，计划删除 distutils。

³ 在运行这些命令时，如果使用减少了占用空间的 Linux 发行版，可能需要单独安装 venv 或其他支持包。
