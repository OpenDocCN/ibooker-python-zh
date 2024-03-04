# 第十章：模块与包

模块与包是任何大型程序的核心，就连 Python 安装程序本身也是一个包。本章重点涉及有关模块和包的常用编程技术，例如如何组织包、把大型模块分割成多个文件、创建命名空间包。同时，也给出了让你自定义导入语句的秘籍。

# 10.1 构建一个模块的层级包

## 问题

你想将你的代码组织成由很多分层模块构成的包。

## 解决方案

封装成包是很简单的。在文件系统上组织你的代码，并确保每个目录都定义了一个**init**.py 文件。 例如：

```py
graphics/
    __init__.py
    primitive/
        __init__.py
        line.py
        fill.py
        text.py
    formats/
        __init__.py
        png.py
        jpg.py

```

一旦你做到了这一点，你应该能够执行各种 import 语句，如下：

```py
import graphics.primitive.line
from graphics.primitive import line
import graphics.formats.jpg as jpg

```

## 讨论

定义模块的层次结构就像在文件系统上建立目录结构一样容易。 文件**init**.py 的目的是要包含不同运行级别的包的可选的初始化代码。 举个例子，如果你执行了语句 import graphics， 文件 graphics/**init**.py 将被导入,建立 graphics 命名空间的内容。像 import graphics.format.jpg 这样导入，文件 graphics/**init**.py 和文件 graphics/graphics/formats/**init**.py 将在文件 graphics/formats/jpg.py 导入之前导入。

绝大部分时候让**init**.py 空着就好。但是有些情况下可能包含代码。 举个例子，**init**.py 能够用来自动加载子模块:

```py
# graphics/formats/__init__.py
from . import jpg
from . import png

```

像这样一个文件,用户可以仅仅通过 import grahpics.formats 来代替 import graphics.formats.jpg 以及 import graphics.formats.png。

**init**.py 的其他常用用法包括将多个文件合并到一个逻辑命名空间，这将在 10.4 小节讨论。

敏锐的程序员会发现，即使没有**init**.py 文件存在，python 仍然会导入包。如果你没有定义**init**.py 时，实际上创建了一个所谓的“命名空间包”，这将在 10.5 小节讨论。万物平等，如果你着手创建一个新的包的话，包含一个**init**.py 文件吧。

# 10.2 控制模块被全部导入的内容

## 问题

当使用’from module import [*](http://python3-cookbook.readthedocs.org/zh_CN/latest/c10/p02_control_the_import_of_everything.html#id3)‘ 语句时，希望对从模块或包导出的符号进行精确控制。

## 解决方案

在你的模块中定义一个变量 **all** 来明确地列出需要导出的内容。

举个例子:

```py
# somemodule.py
def spam():
    pass

def grok():
    pass

blah = 42
# Only export 'spam' and 'grok'
__all__ = ['spam', 'grok']

```

## 讨论

尽管强烈反对使用 ‘from module import [*](http://python3-cookbook.readthedocs.org/zh_CN/latest/c10/p02_control_the_import_of_everything.html#id7)‘, 但是在定义了大量变量名的模块中频繁使用。 如果你不做任何事, 这样的导入将会导入所有不以下划线开头的。 另一方面,如果定义了 **all** , 那么只有被列举出的东西会被导出。

如果你将 **all** 定义成一个空列表, 没有东西将被导出。 如果 **all** 包含未定义的名字, 在导入时引起 AttributeError。

# 10.3 使用相对路径名导入包中子模块

## 问题

将代码组织成包,想用 import 语句从另一个包名没有硬编码过的包的中导入子模块。

## 解决方案

使用包的相对导入，使一个的模块导入同一个包的另一个模块 举个例子，假设在你的文件系统上有 mypackage 包，组织如下：

```py
mypackage/
    __init__.py
    A/
        __init__.py
        spam.py
        grok.py
    B/
        __init__.py
        bar.py

```

如果模块 mypackage.A.spam 要导入同目录下的模块 grok，它应该包括的 import 语句如下：

```py
# mypackage/A/spam.py
from . import grok

```

如果模块 mypackage.A.spam 要导入不同目录下的模块 B.bar，它应该使用的 import 语句如下：

```py
# mypackage/A/spam.py
from ..B import bar

```

两个 import 语句都没包含顶层包名，而是使用了 spam.py 的相对路径。

## 讨论

在包内，既可以使用相对路径也可以使用绝对路径来导入。 举个例子：

```py
# mypackage/A/spam.py
from mypackage.A import grok # OK
from . import grok # OK
import grok # Error (not found)

```

像 mypackage.A 这样使用绝对路径名的不利之处是这将顶层包名硬编码到你的源码中。如果你想重新组织它，你的代码将更脆，很难工作。 举个例子，如果你改变了包名，你就必须检查所有文件来修正源码。 同样，硬编码的名称会使移动代码变得困难。举个例子，也许有人想安装两个不同版本的软件包，只通过名称区分它们。 如果使用相对导入，那一切都 ok，然而使用绝对路径名很可能会出问题。

import 语句的 `<span class="pre" style="box-sizing: border-box;">.</span>` 和 [``](http://python3-cookbook.readthedocs.org/zh_CN/latest/c10/p03_import_submodules_by_relative_names.html#id5)..``看起来很滑稽, 但它指定目录名.为当前目录，..B 为目录../B。这种语法只适用于 import。 举个例子：

```py
from . import grok # OK
import .grok # ERROR

```

尽管使用相对导入看起来像是浏览文件系统，但是不能到定义包的目录之外。也就是说，使用点的这种模式从不是包的目录中导入将会引发错误。

最后，相对导入只适用于在合适的包中的模块。尤其是在顶层的脚本的简单模块中，它们将不起作用。如果包的部分被作为脚本直接执行，那它们将不起作用 例如：

```py
% python3 mypackage/A/spam.py # Relative imports fail

```

另一方面，如果你使用 Python 的-m 选项来执行先前的脚本，相对导入将会正确运行。 例如：

```py
% python3 -m mypackage.A.spam # Relative imports work

```

更多的包的相对导入的背景知识,请看 [PEP 328](http://www.python.org/dev/peps/pep-0328) .

# 10.4 将模块分割成多个文件

## 问题

你想将一个模块分割成多个文件。但是你不想将分离的文件统一成一个逻辑模块时使已有的代码遭到破坏。

## 解决方案

程序模块可以通过变成包来分割成多个独立的文件。考虑下下面简单的模块：

```py
# mymodule.py
class A:
    def spam(self):
        print('A.spam')

class B(A):
    def bar(self):
        print('B.bar')

```

假设你想 mymodule.py 分为两个文件，每个定义的一个类。要做到这一点，首先用 mymodule 目录来替换文件 mymodule.py。 这这个目录下，创建以下文件：

```py
mymodule/
    __init__.py
    a.py
    b.py

```

在 a.py 文件中插入以下代码：

```py
# a.py
class A:
    def spam(self):
        print('A.spam')

```

在 b.py 文件中插入以下代码：

```py
# b.py
from .a import A
class B(A):
    def bar(self):
        print('B.bar')

```

最后，在 **init**.py 中，将 2 个文件粘合在一起：

```py
# __init__.py
from .a import A
from .b import B

```

如果按照这些步骤，所产生的包 MyModule 将作为一个单一的逻辑模块：

```py
>>> import mymodule
>>> a = mymodule.A()
>>> a.spam()
A.spam
>>> b = mymodule.B()
>>> b.bar()
B.bar
>>>

```

## 讨论

在这个章节中的主要问题是一个设计问题，不管你是否希望用户使用很多小模块或只是一个模块。举个例子，在一个大型的代码库中，你可以将这一切都分割成独立的文件，让用户使用大量的 import 语句，就像这样：

```py
from mymodule.a import A
from mymodule.b import B
...

```

这样能工作，但这让用户承受更多的负担，用户要知道不同的部分位于何处。通常情况下，将这些统一起来，使用一条 import 将更加容易，就像这样：

```py
from mymodule import A, B

```

对后者而言，让 mymodule 成为一个大的源文件是最常见的。但是，这一章节展示了如何合并多个文件合并成一个单一的逻辑命名空间。 这样做的关键是创建一个包目录，使用 **init**.py 文件来将每部分粘合在一起。

当一个模块被分割，你需要特别注意交叉引用的文件名。举个例子，在这一章节中，B 类需要访问 A 类作为基类。用包的相对导入 from .a import A 来获取。

整个章节都使用包的相对导入来避免将顶层模块名硬编码到源代码中。这使得重命名模块或者将它移动到别的位置更容易。（见 10.3 小节）

作为这一章节的延伸，将介绍延迟导入。如图所示，**init**.py 文件一次导入所有必需的组件的。但是对于一个很大的模块，可能你只想组件在需要时被加载。 要做到这一点，**init**.py 有细微的变化：

```py
# __init__.py
def A():
    from .a import A
    return A()

def B():
    from .b import B
    return B()

```

在这个版本中，类 A 和类 B 被替换为在第一次访问时加载所需的类的函数。对于用户，这看起来不会有太大的不同。 例如：

```py
>>> import mymodule
>>> a = mymodule.A()
>>> a.spam()
A.spam
>>>

```

延迟加载的主要缺点是继承和类型检查可能会中断。你可能会稍微改变你的代码，例如:

```py
if isinstance(x, mymodule.A): # Error
...

if isinstance(x, mymodule.a.A): # Ok
...

```

延迟加载的真实例子, 见标准库 multiprocessing/**init**.py 的源码.

# 10.5 利用命名空间导入目录分散的代码

## 问题

你可能有大量的代码，由不同的人来分散地维护。每个部分被组织为文件目录，如一个包。然而，你希望能用共同的包前缀将所有组件连接起来，不是将每一个部分作为独立的包来安装。

## 解决方案

从本质上讲，你要定义一个顶级 Python 包，作为一个大集合分开维护子包的命名空间。这个问题经常出现在大的应用框架中，框架开发者希望鼓励用户发布插件或附加包。

在统一不同的目录里统一相同的命名空间，但是要删去用来将组件联合起来的**init**.py 文件。假设你有 Python 代码的两个不同的目录如下：

```py
foo-package/
    spam/
        blah.py

bar-package/
    spam/
        grok.py

```

在这 2 个目录里，都有着共同的命名空间 spam。在任何一个目录里都没有**init**.py 文件。

让我们看看，如果将 foo-package 和 bar-package 都加到 python 模块路径并尝试导入会发生什么

```py
>>> import sys
>>> sys.path.extend(['foo-package', 'bar-package'])
>>> import spam.blah
>>> import spam.grok
>>>

```

两个不同的包目录被合并到一起，你可以导入 spam.blah 和 spam.grok，并且它们能够工作。

## 讨论

在这里工作的机制被称为“包命名空间”的一个特征。从本质上讲，包命名空间是一种特殊的封装设计，为合并不同的目录的代码到一个共同的命名空间。对于大的框架，这可能是有用的，因为它允许一个框架的部分被单独地安装下载。它也使人们能够轻松地为这样的框架编写第三方附加组件和其他扩展。

包命名空间的关键是确保顶级目录中没有**init**.py 文件来作为共同的命名空间。缺失**init**.py 文件使得在导入包的时候会发生有趣的事情：这并没有产生错误，解释器创建了一个由所有包含匹配包名的目录组成的列表。特殊的包命名空间模块被创建，只读的目录列表副本被存储在其**path**变量中。 举个例子：

```py
>>> import spam
>>> spam.__path__
_NamespacePath(['foo-package/spam', 'bar-package/spam'])
>>>

```

在定位包的子组件时，目录**path**将被用到(例如, 当导入 spam.grok 或者 spam.blah 的时候).

包命名空间的一个重要特点是任何人都可以用自己的代码来扩展命名空间。举个例子，假设你自己的代码目录像这样：

```py
my-package/
    spam/
        custom.py

```

如果你将你的代码目录和其他包一起添加到 sys.path，这将无缝地合并到别的 spam 包目录中：

```py
>>> import spam.custom
>>> import spam.grok
>>> import spam.blah
>>>

```

一个包是否被作为一个包命名空间的主要方法是检查其**file**属性。如果没有，那包是个命名空间。这也可以由其字符表现形式中的“namespace”这个词体现出来。

```py
>>> spam.__file__
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
AttributeError: 'module' object has no attribute '__file__'
>>> spam
<module 'spam' (namespace)>
>>>

```

更多的包命名空间信息可以查看 [PEP 420](https://www.python.org/dev/peps/pep-0420/).

# 10.6 重新加载模块

## 问题

你想重新加载已经加载的模块，因为你对其源码进行了修改。

## 解决方案

使用 imp.reload()来重新加载先前加载的模块。举个例子：

```py
>>> import spam
>>> import imp
>>> imp.reload(spam)
<module 'spam' from './spam.py'>
>>>

```

## 讨论

重新加载模块在开发和调试过程中常常很有用。但在生产环境中的代码使用会不安全，因为它并不总是像您期望的那样工作。

reload()擦除了模块底层字典的内容，并通过重新执行模块的源代码来刷新它。模块对象本身的身份保持不变。因此，该操作在程序中所有已经被导入了的地方更新了模块。

尽管如此，reload()没有更新像”from module import name”这样使用 import 语句导入的定义。举个例子：

```py
# spam.py
def bar():
    print('bar')

def grok():
    print('grok')

```

现在启动交互式会话：

```py
>>> import spam
>>> from spam import grok
>>> spam.bar()
bar
>>> grok()
grok
>>>

```

不退出 Python 修改 spam.py 的源码，将 grok()函数改成这样：

```py
def grok():
    print('New grok')

```

现在回到交互式会话，重新加载模块，尝试下这个实验：

```py
>>> import imp
>>> imp.reload(spam)
<module 'spam' from './spam.py'>
>>> spam.bar()
bar
>>> grok() # Notice old output
grok
>>> spam.grok() # Notice new output
New grok
>>>

```

在这个例子中，你看到有 2 个版本的 grok()函数被加载。通常来说，这不是你想要的，而是令人头疼的事。

因此，在生产环境中可能需要避免重新加载模块。在交互环境下调试，解释程序并试图弄懂它。

# 10.7 运行目录或压缩文件

## 问题

您有已经一个复杂的脚本到涉及多个文件的应用程序。你想有一些简单的方法让用户运行程序。

## 解决方案

如果你的应用程序已经有多个文件，你可以把你的应用程序放进它自己的目录并添加一个**main**.py 文件。 举个例子，你可以像这样创建目录：

```py
myapplication/
    spam.py
    bar.py
    grok.py
    __main__.py

```

如果**main**.py 存在，你可以简单地在顶级目录运行 Python 解释器：

```py
bash % python3 myapplication

```

解释器将执行**main**.py 文件作为主程序。

如果你将你的代码打包成 zip 文件，这种技术同样也适用，举个例子：

```py
bash % ls
spam.py bar.py grok.py __main__.py
bash % zip -r myapp.zip *.py
bash % python3 myapp.zip
... output from __main__.py ...

```

## 讨论

创建一个目录或 zip 文件并添加**main**.py 文件来将一个更大的 Python 应用打包是可行的。这和作为标准库被安装到 Python 库的代码包是有一点区别的。相反，这只是让别人执行的代码包。

由于目录和 zip 文件与正常文件有一点不同，你可能还需要增加一个 shell 脚本，使执行更加容易。例如，如果代码文件名为 myapp.zip，你可以创建这样一个顶级脚本：

```py
#!/usr/bin/env python3 /usr/local/bin/myapp.zip

```

# 10.8 读取位于包中的数据文件

## 问题

你的包中包含代码需要去读取的数据文件。你需要尽可能地用最便捷的方式来做这件事。

## 解决方案

假设你的包中的文件组织成如下：

```py
mypackage/
    __init__.py
    somedata.dat
    spam.py

```

现在假设 spam.py 文件需要读取 somedata.dat 文件中的内容。你可以用以下代码来完成：

```py
# spam.py
import pkgutil
data = pkgutil.get_data(__package__, 'somedata.dat')

```

由此产生的变量是包含该文件的原始内容的字节字符串。

## 讨论

要读取数据文件，你可能会倾向于编写使用内置的 I/ O 功能的代码，如 open()。但是这种方法也有一些问题。

首先，一个包对解释器的当前工作目录几乎没有控制权。因此，编程时任何 I/O 操作都必须使用绝对文件名。由于每个模块包含有完整路径的**file**变量，这弄清楚它的路径不是不可能，但它很凌乱。

第二，包通常安装作为.zip 或.egg 文件，这些文件像文件系统上的一个普通目录一样不会被保留。因此，你试图用 open()对一个包含数据文件的归档文件进行操作，它根本不会工作。

pkgutil.get_data()函数是一个读取数据文件的高级工具，不用管包是如何安装以及安装在哪。它只是工作并将文件内容以字节字符串返回给你

get_data()的第一个参数是包含包名的字符串。你可以直接使用包名，也可以使用特殊的变量，比如**package**。第二个参数是包内文件的相对名称。如果有必要，可以使用标准的 Unix 命名规范到不同的目录，只有最后的目录仍然位于包中。

# 10.9 将文件夹加入到 sys.path

## 问题

你无法导入你的 Python 代码因为它所在的目录不在 sys.path 里。你想将添加新目录到 Python 路径，但是不想硬链接到你的代码。

## 解决方案

有两种常用的方式将新目录添加到 sys.path。第一种，你可以使用 PYTHONPATH 环境变量来添加。例如：

```py
bash % env PYTHONPATH=/some/dir:/other/dir python3
Python 3.3.0 (default, Oct 4 2012, 10:17:33)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/some/dir', '/other/dir', ...]
>>>

```

在自定义应用程序中，这样的环境变量可在程序启动时设置或通过 shell 脚本。

第二种方法是创建一个.pth 文件，将目录列举出来，像这样：

```py
# myapplication.pth
/some/dir
/other/dir

```

这个.pth 文件需要放在某个 Python 的 site-packages 目录，通常位于/usr/local/lib/python3.3/site-packages 或者 ~/.local/lib/python3.3/sitepackages。当解释器启动时，.pth 文件里列举出来的存在于文件系统的目录将被添加到 sys.path。安装一个.pth 文件可能需要管理员权限，如果它被添加到系统级的 Python 解释器。

## 讨论

比起费力地找文件，你可能会倾向于写一个代码手动调节 sys.path 的值。例如:

```py
import sys
sys.path.insert(0, '/some/dir')
sys.path.insert(0, '/other/dir')

```

虽然这能“工作”，它是在实践中极为脆弱，应尽量避免使用。这种方法的问题是，它将目录名硬编码到了你的源。如果你的代码被移到一个新的位置，这会导致维护问题。更好的做法是在不修改源代码的情况下，将 path 配置到其他地方。如果您使用模块级的变量来精心构造一个适当的绝对路径，有时你可以解决硬编码目录的问题，比如**file**。举个例子：

```py
import sys
from os.path import abspath, join, dirname
sys.path.insert(0, abspath(dirname('__file__'), 'src'))

```

这将 src 目录添加到 path 里，和执行插入步骤的代码在同一个目录里。

site-packages 目录是第三方包和模块安装的目录。如果你手动安装你的代码，它将被安装到 site-packages 目录。虽然.pth 文件配置的 path 必须出现在 site-packages 里，但代码可以在系统上任何你想要的目录。因此，你可以把你的代码放在一系列不同的目录，只要那些目录包含在.pth 文件里。

# 10.10 通过字符串名导入模块

## 问题

你想导入一个模块，但是模块的名字在字符串里。你想对字符串调用导入命令。

## 解决方案

使用 importlib.import_module()函数来手动导入名字为字符串给出的一个模块或者包的一部分。举个例子：

```py
>>> import importlib
>>> math = importlib.import_module('math')
>>> math.sin(2)
0.9092974268256817
>>> mod = importlib.import_module('urllib.request')
>>> u = mod.urlopen('http://www.python.org')
>>>

```

import_module 只是简单地执行和 import 相同的步骤，但是返回生成的模块对象。你只需要将其存储在一个变量，然后像正常的模块一样使用。

如果你正在使用的包，import_module()也可用于相对导入。但是，你需要给它一个额外的参数。例如：

```py
import importlib
# Same as 'from . import b'
b = importlib.import_module('.b', __package__)

```

## 讨论

使用 import_module()手动导入模块的问题通常出现在以某种方式编写修改或覆盖模块的代码时候。例如，也许你正在执行某种自定义导入机制，需要通过名称来加载一个模块，通过补丁加载代码。

在旧的代码，有时你会看到用于导入的内建函数**import**()。尽管它能工作，但是 importlib.import_module() 通常更容易使用。

自定义导入过程的高级实例见 10.11 小节

# 10.11 通过导入钩子远程加载模块

## 问题

You would like to customize Python’s import statement so that it can transparently load modules from a remote machine.

## 解决方案

First, a serious disclaimer about security. The idea discussed in this recipe would be wholly bad without some kind of extra security and authentication layer. That said, the main goal is actually to take a deep dive into the inner workings of Python’s import statement. If you get this recipe to work and understand the inner workings, you’ll have a solid foundation of customizing import for almost any other purpose. With that out of the way, let’s carry on.

At the core of this recipe is a desire to extend the functionality of the import statement. There are several approaches for doing this, but for the purposes of illustration, start by making the following directory of Python code:

```py
testcode/
    spam.py
    fib.py
    grok/
        __init__.py
        blah.py

```

The content of these files doesn’t matter, but put a few simple statements and functions in each file so you can test them and see output when they’re imported. For example:

```py
# spam.py
print("I'm spam")

def hello(name):
    print('Hello %s' % name)

# fib.py
print("I'm fib")

def fib(n):
    if n < 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)

# grok/__init__.py
print("I'm grok.__init__")

# grok/blah.py
print("I'm grok.blah")

```

The goal here is to allow remote access to these files as modules. Perhaps the easiest way to do this is to publish them on a web server. Simply go to the testcode directory and run Python like this:

```py
bash % cd testcode
bash % python3 -m http.server 15000
Serving HTTP on 0.0.0.0 port 15000 ...

```

Leave that server running and start up a separate Python interpreter. Make sure you can access the remote files using urllib. For example:

```py
>>> from urllib.request import urlopen
>>> u = urlopen('http://localhost:15000/fib.py')
>>> data = u.read().decode('utf-8')
>>> print(data)
# fib.py
print("I'm fib")

def fib(n):
    if n < 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)
>>>

```

Loading source code from this server is going to form the basis for the remainder of this recipe. Specifically, instead of manually grabbing a file of source code using urlop en(), the import statement will be customized to do it transparently behind the scenes.

The first approach to loading a remote module is to create an explicit loading function for doing it. For example:

```py
import imp
import urllib.request
import sys

def load_module(url):
    u = urllib.request.urlopen(url)
    source = u.read().decode('utf-8')
    mod = sys.modules.setdefault(url, imp.new_module(url))
    code = compile(source, url, 'exec')
    mod.__file__ = url
    mod.__package__ = ''
    exec(code, mod.__dict__)
    return mod

```

This function merely downloads the source code, compiles it into a code object using compile(), and executes it in the dictionary of a newly created module object. Here’s how you would use the function:

```py
>>> fib = load_module('http://localhost:15000/fib.py')
I'm fib
>>> fib.fib(10)
89
>>> spam = load_module('http://localhost:15000/spam.py')
I'm spam
>>> spam.hello('Guido')
Hello Guido
>>> fib
<module 'http://localhost:15000/fib.py' from 'http://localhost:15000/fib.py'>
>>> spam
<module 'http://localhost:15000/spam.py' from 'http://localhost:15000/spam.py'>
>>>

```

As you can see, it “works” for simple modules. However, it’s not plugged into the usual import statement, and extending the code to support more advanced constructs, such as packages, would require additional work.

A much slicker approach is to create a custom importer. The first way to do this is to create what’s known as a meta path importer. Here is an example:

```py
# urlimport.py
import sys
import importlib.abc
import imp
from urllib.request import urlopen
from urllib.error import HTTPError, URLError
from html.parser import HTMLParser

# Debugging
import logging
log = logging.getLogger(__name__)

# Get links from a given URL
def _get_links(url):
    class LinkParser(HTMLParser):
        def handle_starttag(self, tag, attrs):
            if tag == 'a':
                attrs = dict(attrs)
                links.add(attrs.get('href').rstrip('/'))
    links = set()
    try:
        log.debug('Getting links from %s' % url)
        u = urlopen(url)
        parser = LinkParser()
        parser.feed(u.read().decode('utf-8'))
    except Exception as e:
        log.debug('Could not get links. %s', e)
    log.debug('links: %r', links)
    return links

class UrlMetaFinder(importlib.abc.MetaPathFinder):
    def __init__(self, baseurl):
        self._baseurl = baseurl
        self._links = { }
        self._loaders = { baseurl : UrlModuleLoader(baseurl) }

    def find_module(self, fullname, path=None):
        log.debug('find_module: fullname=%r, path=%r', fullname, path)
        if path is None:
            baseurl = self._baseurl
        else:
            if not path[0].startswith(self._baseurl):
                return None
            baseurl = path[0]
        parts = fullname.split('.')
        basename = parts[-1]
        log.debug('find_module: baseurl=%r, basename=%r', baseurl, basename)

        # Check link cache
        if basename not in self._links:
            self._links[baseurl] = _get_links(baseurl)

        # Check if it's a package
        if basename in self._links[baseurl]:
            log.debug('find_module: trying package %r', fullname)
            fullurl = self._baseurl + '/' + basename
            # Attempt to load the package (which accesses __init__.py)
            loader = UrlPackageLoader(fullurl)
            try:
                loader.load_module(fullname)
                self._links[fullurl] = _get_links(fullurl)
                self._loaders[fullurl] = UrlModuleLoader(fullurl)
                log.debug('find_module: package %r loaded', fullname)
            except ImportError as e:
                log.debug('find_module: package failed. %s', e)
                loader = None
            return loader
        # A normal module
        filename = basename + '.py'
        if filename in self._links[baseurl]:
            log.debug('find_module: module %r found', fullname)
            return self._loaders[baseurl]
        else:
            log.debug('find_module: module %r not found', fullname)
            return None

    def invalidate_caches(self):
        log.debug('invalidating link cache')
        self._links.clear()

# Module Loader for a URL
class UrlModuleLoader(importlib.abc.SourceLoader):
    def __init__(self, baseurl):
        self._baseurl = baseurl
        self._source_cache = {}

    def module_repr(self, module):
        return '<urlmodule %r from %r>' % (module.__name__, module.__file__)

    # Required method
    def load_module(self, fullname):
        code = self.get_code(fullname)
        mod = sys.modules.setdefault(fullname, imp.new_module(fullname))
        mod.__file__ = self.get_filename(fullname)
        mod.__loader__ = self
        mod.__package__ = fullname.rpartition('.')[0]
        exec(code, mod.__dict__)
        return mod

    # Optional extensions
    def get_code(self, fullname):
        src = self.get_source(fullname)
        return compile(src, self.get_filename(fullname), 'exec')

    def get_data(self, path):
        pass

    def get_filename(self, fullname):
        return self._baseurl + '/' + fullname.split('.')[-1] + '.py'

    def get_source(self, fullname):
        filename = self.get_filename(fullname)
        log.debug('loader: reading %r', filename)
        if filename in self._source_cache:
            log.debug('loader: cached %r', filename)
            return self._source_cache[filename]
        try:
            u = urlopen(filename)
            source = u.read().decode('utf-8')
            log.debug('loader: %r loaded', filename)
            self._source_cache[filename] = source
            return source
        except (HTTPError, URLError) as e:
            log.debug('loader: %r failed. %s', filename, e)
            raise ImportError("Can't load %s" % filename)

    def is_package(self, fullname):
        return False

# Package loader for a URL
class UrlPackageLoader(UrlModuleLoader):
    def load_module(self, fullname):
        mod = super().load_module(fullname)
        mod.__path__ = [ self._baseurl ]
        mod.__package__ = fullname

    def get_filename(self, fullname):
        return self._baseurl + '/' + '__init__.py'

    def is_package(self, fullname):
        return True

# Utility functions for installing/uninstalling the loader
_installed_meta_cache = { }
def install_meta(address):
    if address not in _installed_meta_cache:
        finder = UrlMetaFinder(address)
        _installed_meta_cache[address] = finder
        sys.meta_path.append(finder)
        log.debug('%r installed on sys.meta_path', finder)

def remove_meta(address):
    if address in _installed_meta_cache:
        finder = _installed_meta_cache.pop(address)
        sys.meta_path.remove(finder)
        log.debug('%r removed from sys.meta_path', finder)

```

Here is an interactive session showing how to use the preceding code:

```py
>>> # importing currently fails
>>> import fib
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'
>>> # Load the importer and retry (it works)
>>> import urlimport
>>> urlimport.install_meta('http://localhost:15000')
>>> import fib
I'm fib
>>> import spam
I'm spam
>>> import grok.blah
I'm grok.__init__
I'm grok.blah
>>> grok.blah.__file__
'http://localhost:15000/grok/blah.py'
>>>

```

This particular solution involves installing an instance of a special finder object UrlMe taFinder as the last entry in sys.meta_path. Whenever modules are imported, the finders in sys.meta_path are consulted in order to locate the module. In this example, the UrlMetaFinder instance becomes a finder of last resort that’s triggered when a module can’t be found in any of the normal locations.

As for the general implementation approach, the UrlMetaFinder class wraps around a user-specified URL. Internally, the finder builds sets of valid links by scraping them from the given URL. When imports are made, the module name is compared against this set of known links. If a match can be found, a separate UrlModuleLoader class is used to load source code from the remote machine and create the resulting module object. One reason for caching the links is to avoid unnecessary HTTP requests on repeated imports.

The second approach to customizing import is to write a hook that plugs directly into the sys.path variable, recognizing certain directory naming patterns. Add the following class and support functions to urlimport.py:

```py
# urlimport.py
# ... include previous code above ...
# Path finder class for a URL
class UrlPathFinder(importlib.abc.PathEntryFinder):
    def __init__(self, baseurl):
        self._links = None
        self._loader = UrlModuleLoader(baseurl)
        self._baseurl = baseurl

    def find_loader(self, fullname):
        log.debug('find_loader: %r', fullname)
        parts = fullname.split('.')
        basename = parts[-1]
        # Check link cache
        if self._links is None:
            self._links = [] # See discussion
            self._links = _get_links(self._baseurl)

        # Check if it's a package
        if basename in self._links:
            log.debug('find_loader: trying package %r', fullname)
            fullurl = self._baseurl + '/' + basename
            # Attempt to load the package (which accesses __init__.py)
            loader = UrlPackageLoader(fullurl)
            try:
                loader.load_module(fullname)
                log.debug('find_loader: package %r loaded', fullname)
            except ImportError as e:
                log.debug('find_loader: %r is a namespace package', fullname)
                loader = None
            return (loader, [fullurl])

        # A normal module
        filename = basename + '.py'
        if filename in self._links:
            log.debug('find_loader: module %r found', fullname)
            return (self._loader, [])
        else:
            log.debug('find_loader: module %r not found', fullname)
            return (None, [])

    def invalidate_caches(self):
        log.debug('invalidating link cache')
        self._links = None

# Check path to see if it looks like a URL
_url_path_cache = {}
def handle_url(path):
    if path.startswith(('http://', 'https://')):
        log.debug('Handle path? %s. [Yes]', path)
        if path in _url_path_cache:
            finder = _url_path_cache[path]
        else:
            finder = UrlPathFinder(path)
            _url_path_cache[path] = finder
        return finder
    else:
        log.debug('Handle path? %s. [No]', path)

def install_path_hook():
    sys.path_hooks.append(handle_url)
    sys.path_importer_cache.clear()
    log.debug('Installing handle_url')

def remove_path_hook():
    sys.path_hooks.remove(handle_url)
    sys.path_importer_cache.clear()
    log.debug('Removing handle_url')

```

To use this path-based finder, you simply add URLs to sys.path. For example:

```py
>>> # Initial import fails
>>> import fib
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'

>>> # Install the path hook
>>> import urlimport
>>> urlimport.install_path_hook()

>>> # Imports still fail (not on path)
>>> import fib
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'

>>> # Add an entry to sys.path and watch it work
>>> import sys
>>> sys.path.append('http://localhost:15000')
>>> import fib
I'm fib
>>> import grok.blah
I'm grok.__init__
I'm grok.blah
>>> grok.blah.__file__
'http://localhost:15000/grok/blah.py'
>>>

```

The key to this last example is the handle_url() function, which is added to the sys.path_hooks variable. When the entries on sys.path are being processed, the functions in sys.path_hooks are invoked. If any of those functions return a finder object, that finder is used to try to load modules for that entry on sys.path.

It should be noted that the remotely imported modules work exactly like any other module. For instance:

```py
>>> fib
<urlmodule 'fib' from 'http://localhost:15000/fib.py'>
>>> fib.__name__
'fib'
>>> fib.__file__
'http://localhost:15000/fib.py'
>>> import inspect
>>> print(inspect.getsource(fib))
# fib.py
print("I'm fib")

def fib(n):
    if n < 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)
>>>

```

## 讨论

Before discussing this recipe in further detail, it should be emphasized that Python’s module, package, and import mechanism is one of the most complicated parts of the entire language—often poorly understood by even the most seasoned Python programmers unless they’ve devoted effort to peeling back the covers. There are several critical documents that are worth reading, including the documentation for the [importlib module](https://docs.python.org/3/library/importlib.html) and [PEP 302](http://www.python.org/dev/peps/pep-0302). That documentation won’t be repeated here, but some essential highlights will be discussed.

First, if you want to create a new module object, you use the imp.new_module() function. For example:

```py
>>> import imp
>>> m = imp.new_module('spam')
>>> m
<module 'spam'>
>>> m.__name__
'spam'
>>>

```

Module objects usually have a few expected attributes, including **file** (the name of the file that the module was loaded from) and **package** (the name of the enclosing package, if any).

Second, modules are cached by the interpreter. The module cache can be found in the dictionary sys.modules. Because of this caching, it’s common to combine caching and module creation together into a single step. For example:

```py
>>> import sys
>>> import imp
>>> m = sys.modules.setdefault('spam', imp.new_module('spam'))
>>> m
<module 'spam'>
>>>

```

The main reason for doing this is that if a module with the given name already exists, you’ll get the already created module instead. For example:

```py
>>> import math
>>> m = sys.modules.setdefault('math', imp.new_module('math'))
>>> m
<module 'math' from '/usr/local/lib/python3.3/lib-dynload/math.so'>
>>> m.sin(2)
0.9092974268256817
>>> m.cos(2)
-0.4161468365471424
>>>

```

Since creating modules is easy, it is straightforward to write simple functions, such as the load_module() function in the first part of this recipe. A downside of this approach is that it is actually rather tricky to handle more complicated cases, such as package imports. In order to handle a package, you would have to reimplement much of the underlying logic that’s already part of the normal import statement (e.g., checking for directories, looking for **init**.py files, executing those files, setting up paths, etc.). This complexity is one of the reasons why it’s often better to extend the import statement directly rather than defining a custom function.

Extending the import statement is straightforward, but involves a number of moving parts. At the highest level, import operations are processed by a list of “meta-path” finders that you can find in the list sys.meta_path. If you output its value, you’ll see the following:

```py
>>> from pprint import pprint
>>> pprint(sys.meta_path)
[<class '_frozen_importlib.BuiltinImporter'>,
<class '_frozen_importlib.FrozenImporter'>,
<class '_frozen_importlib.PathFinder'>]
>>>

```

When executing a statement such as import fib, the interpreter walks through the finder objects on sys.meta_path and invokes their find_module() method in order to locate an appropriate module loader. It helps to see this by experimentation, so define the following class and try the following:

```py
>>> class Finder:
...     def find_module(self, fullname, path):
...         print('Looking for', fullname, path)
...         return None
...
>>> import sys
>>> sys.meta_path.insert(0, Finder()) # Insert as first entry
>>> import math
Looking for math None
>>> import types
Looking for types None
>>> import threading
Looking for threading None
Looking for time None
Looking for traceback None
Looking for linecache None
Looking for tokenize None
Looking for token None
>>>

```

Notice how the find_module() method is being triggered on every import. The role of the path argument in this method is to handle packages. When packages are imported, it is a list of the directories that are found in the package’s **path** attribute. These are the paths that need to be checked to find package subcomponents. For example, notice the path setting for xml.etree and xml.etree.ElementTree:

```py
>>> import xml.etree.ElementTree
Looking for xml None
Looking for xml.etree ['/usr/local/lib/python3.3/xml']
Looking for xml.etree.ElementTree ['/usr/local/lib/python3.3/xml/etree']
Looking for warnings None
Looking for contextlib None
Looking for xml.etree.ElementPath ['/usr/local/lib/python3.3/xml/etree']
Looking for _elementtree None
Looking for copy None
Looking for org None
Looking for pyexpat None
Looking for ElementC14N None
>>>

```

The placement of the finder on sys.meta_path is critical. Remove it from the front of the list to the end of the list and try more imports:

```py
>>> del sys.meta_path[0]
>>> sys.meta_path.append(Finder())
>>> import urllib.request
>>> import datetime

```

Now you don’t see any output because the imports are being handled by other entries in sys.meta_path. In this case, you would only see it trigger when nonexistent modules are imported:

```py
>>> import fib
Looking for fib None
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'
>>> import xml.superfast
Looking for xml.superfast ['/usr/local/lib/python3.3/xml']
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ImportError: No module named 'xml.superfast'
>>>

```

The fact that you can install a finder to catch unknown modules is the key to the UrlMetaFinder class in this recipe. An instance of UrlMetaFinder is added to the end of sys.meta_path, where it serves as a kind of importer of last resort. If the requested module name can’t be located by any of the other import mechanisms, it gets handled by this finder. Some care needs to be taken when handling packages. Specifically, the value presented in the path argument needs to be checked to see if it starts with the URL registered in the finder. If not, the submodule must belong to some other finder and should be ignored.

Additional handling of packages is found in the UrlPackageLoader class. This class, rather than importing the package name, tries to load the underlying **init**.py file. It also sets the module **path** attribute. This last part is critical, as the value set will be passed to subsequent find_module() calls when loading package submodules. The path-based import hook is an extension of these ideas, but based on a somewhat different mechanism. As you know, sys.path is a list of directories where Python looks for modules. For example:

```py
>>> from pprint import pprint
>>> import sys
>>> pprint(sys.path)
['',
'/usr/local/lib/python33.zip',
'/usr/local/lib/python3.3',
'/usr/local/lib/python3.3/plat-darwin',
'/usr/local/lib/python3.3/lib-dynload',
'/usr/local/lib/...3.3/site-packages']
>>>

```

Each entry in sys.path is additionally attached to a finder object. You can view these finders by looking at sys.path_importer_cache:

```py
>>> pprint(sys.path_importer_cache)
{'.': FileFinder('.'),
'/usr/local/lib/python3.3': FileFinder('/usr/local/lib/python3.3'),
'/usr/local/lib/python3.3/': FileFinder('/usr/local/lib/python3.3/'),
'/usr/local/lib/python3.3/collections': FileFinder('...python3.3/collections'),
'/usr/local/lib/python3.3/encodings': FileFinder('...python3.3/encodings'),
'/usr/local/lib/python3.3/lib-dynload': FileFinder('...python3.3/lib-dynload'),
'/usr/local/lib/python3.3/plat-darwin': FileFinder('...python3.3/plat-darwin'),
'/usr/local/lib/python3.3/site-packages': FileFinder('...python3.3/site-packages'),
'/usr/local/lib/python33.zip': None}
>>>

```

sys.path_importer_cache tends to be much larger than sys.path because it records finders for all known directories where code is being loaded. This includes subdirectories of packages which usually aren’t included on sys.path.

To execute import fib, the directories on sys.path are checked in order. For each directory, the name fib is presented to the associated finder found in sys.path_im porter_cache. This is also something that you can investigate by making your own finder and putting an entry in the cache. Try this experiment:

```py
>>> class Finder:
... def find_loader(self, name):
...     print('Looking for', name)
...     return (None, [])
...
>>> import sys
>>> # Add a "debug" entry to the importer cache
>>> sys.path_importer_cache['debug'] = Finder()
>>> # Add a "debug" directory to sys.path
>>> sys.path.insert(0, 'debug')
>>> import threading
Looking for threading
Looking for time
Looking for traceback
Looking for linecache
Looking for tokenize
Looking for token
>>>

```

Here, you’ve installed a new cache entry for the name debug and installed the name debug as the first entry on sys.path. On all subsequent imports, you see your finder being triggered. However, since it returns (None, []), processing simply continues to the next entry.

The population of sys.path_importer_cache is controlled by a list of functions stored in sys.path_hooks. Try this experiment, which clears the cache and adds a new path checking function to sys.path_hooks:

```py
>>> sys.path_importer_cache.clear()
>>> def check_path(path):
...     print('Checking', path)
...     raise ImportError()
...
>>> sys.path_hooks.insert(0, check_path)
>>> import fib
Checked debug
Checking .
Checking /usr/local/lib/python33.zip
Checking /usr/local/lib/python3.3
Checking /usr/local/lib/python3.3/plat-darwin
Checking /usr/local/lib/python3.3/lib-dynload
Checking /Users/beazley/.local/lib/python3.3/site-packages
Checking /usr/local/lib/python3.3/site-packages
Looking for fib
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'
>>>

```

As you can see, the check_path() function is being invoked for every entry on sys.path. However, since an ImportError exception is raised, nothing else happens (checking just moves to the next function on sys.path_hooks).

Using this knowledge of how sys.path is processed, you can install a custom path checking function that looks for filename patterns, such as URLs. For instance:

```py
>>> def check_url(path):
...     if path.startswith('http://'):
...         return Finder()
...     else:
...         raise ImportError()
...
>>> sys.path.append('http://localhost:15000')
>>> sys.path_hooks[0] = check_url
>>> import fib
Looking for fib # Finder output!
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'

>>> # Notice installation of Finder in sys.path_importer_cache
>>> sys.path_importer_cache['http://localhost:15000']
<__main__.Finder object at 0x10064c850>
>>>

```

This is the key mechanism at work in the last part of this recipe. Essentially, a custom path checking function has been installed that looks for URLs in sys.path. When they are encountered, a new UrlPathFinder instance is created and installed into sys.path_importer_cache. From that point forward, all import statements that pass through that part of sys.path will try to use your custom finder.

Package handling with a path-based importer is somewhat tricky, and relates to the return value of the find_loader() method. For simple modules, find_loader() returns a tuple (loader, None) where loader is an instance of a loader that will import the module.

For a normal package, find_loader() returns a tuple (loader, path) where loader is the loader instance that will import the package (and execute **init**.py) and path is a list of the directories that will make up the initial setting of the package’s **path** attribute. For example, if the base URL was[`localhost:15000`](http://localhost:15000) and a user executed import grok, the path returned by find_loader() would be [ ‘[`local`](http://local) host:15000/grok’ ].

The find_loader() must additionally account for the possibility of a namespace package. A namespace package is a package where a valid package directory name exists, but no **init**.py file can be found. For this case, find_loader() must return a tuple (None, path) where path is a list of directories that would have made up the package’s **path** attribute had it defined an **init**.py file. For this case, the import mechanism moves on to check further directories on sys.path. If more namespace packages are found, all of the resulting paths are joined together to make a final namespace package. See Recipe 10.5 for more information on namespace packages.

There is a recursive element to package handling that is not immediately obvious in the solution, but also at work. All packages contain an internal path setting, which can be found in **path** attribute. For example:

```py
>>> import xml.etree.ElementTree
>>> xml.__path__
['/usr/local/lib/python3.3/xml']
>>> xml.etree.__path__
['/usr/local/lib/python3.3/xml/etree']
>>>

```

As mentioned, the setting of **path** is controlled by the return value of the find_load er() method. However, the subsequent processing of **path** is also handled by the functions in sys.path_hooks. Thus, when package subcomponents are loaded, the entries in **path** are checked by the handle_url() function. This causes new instances of UrlPathFinder to be created and added to sys.path_importer_cache.

One remaining tricky part of the implementation concerns the behavior of the han dle_url() function and its interaction with the _get_links() function used internally. If your implementation of a finder involves the use of other modules (e.g., urllib.re quest), there is a possibility that those modules will attempt to make further imports in the middle of the finder’s operation. This can actually cause handle_url() and other parts of the finder to get executed in a kind of recursive loop. To account for this possibility, the implementation maintains a cache of created finders (one per URL). This avoids the problem of creating duplicate finders. In addition, the following fragment of code ensures that the finder doesn’t respond to any import requests while it’s in the processs of getting the initial set of links:

```py
# Check link cache
if self._links is None:
    self._links = [] # See discussion
    self._links = _get_links(self._baseurl)

```

You may not need this checking in other implementations, but for this example involving URLs, it was required.

Finally, the invalidate_caches() method of both finders is a utility method that is supposed to clear internal caches should the source code change. This method is triggered when a user invokes importlib.invalidate_caches(). You might use it if you want the URL importers to reread the list of links, possibly for the purpose of being able to access newly added files.

In comparing the two approaches (modifying sys.meta_path or using a path hook), it helps to take a high-level view. Importers installed using sys.meta_path are free to handle modules in any manner that they wish. For instance, they could load modules out of a database or import them in a manner that is radically different than normal module/package handling. This freedom also means that such importers need to do more bookkeeping and internal management. This explains, for instance, why the implementation of UrlMetaFinder needs to do its own caching of links, loaders, and other details. On the other hand, path-based hooks are more narrowly tied to the processing of sys.path. Because of the connection to sys.path, modules loaded with such extensions will tend to have the same features as normal modules and packages that programmers are used to.

Assuming that your head hasn’t completely exploded at this point, a key to understanding and experimenting with this recipe may be the added logging calls. You can enable logging and try experiments such as this:

```py
>>> import logging
>>> logging.basicConfig(level=logging.DEBUG)
>>> import urlimport
>>> urlimport.install_path_hook()
DEBUG:urlimport:Installing handle_url
>>> import fib
DEBUG:urlimport:Handle path? /usr/local/lib/python33.zip. [No]
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
ImportError: No module named 'fib'
>>> import sys
>>> sys.path.append('http://localhost:15000')
>>> import fib
DEBUG:urlimport:Handle path? http://localhost:15000\. [Yes]
DEBUG:urlimport:Getting links from http://localhost:15000
DEBUG:urlimport:links: {'spam.py', 'fib.py', 'grok'}
DEBUG:urlimport:find_loader: 'fib'
DEBUG:urlimport:find_loader: module 'fib' found
DEBUG:urlimport:loader: reading 'http://localhost:15000/fib.py'
DEBUG:urlimport:loader: 'http://localhost:15000/fib.py' loaded
I'm fib
>>>

```

Last, but not least, spending some time sleeping with [PEP 302](http://www.python.org/dev/peps/pep-0302) and the documentation for importlib under your pillow may be advisable.

# 10.12 导入模块的同时修改模块

## 问题

You want to patch or apply decorators to functions in an existing module. However, youonly want to do it if the module actually gets imported and used elsewhere.

## 解决方案

The essential problem here is that you would like to carry out actions in response to amodule being loaded. Perhaps you want to trigger some kind of callback function thatwould notify you when a module was loaded.

This problem can be solved using the same import hook machinery discussed inRecipe 10.11\. Here is a possible solution:

```py
# postimport.py
import importlib
import sys
from collections import defaultdict

_post_import_hooks = defaultdict(list)

class PostImportFinder:
    def __init__(self):
        self._skip = set()

    def find_module(self, fullname, path=None):
        if fullname in self._skip:
            return None
        self._skip.add(fullname)
        return PostImportLoader(self)

class PostImportLoader:
    def __init__(self, finder):
        self._finder = finder

    def load_module(self, fullname):
        importlib.import_module(fullname)
        module = sys.modules[fullname]
        for func in _post_import_hooks[fullname]:
            func(module)
        self._finder._skip.remove(fullname)
        return module

def when_imported(fullname):
    def decorate(func):
        if fullname in sys.modules:
            func(sys.modules[fullname])
        else:
            _post_import_hooks[fullname].append(func)
        return func
    return decorate

sys.meta_path.insert(0, PostImportFinder())

```

To use this code, you use the when_imported() decorator. For example:

```py
>>> from postimport import when_imported
>>> @when_imported('threading')
... def warn_threads(mod):
...     print('Threads? Are you crazy?')
...
>>>
>>> import threading
Threads? Are you crazy?
>>>

```

As a more practical example, maybe you want to apply decorators to existing definitions,such as shown here:

```py
from functools import wraps
from postimport import when_imported

def logged(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print('Calling', func.__name__, args, kwargs)
        return func(*args, **kwargs)
    return wrapper

# Example
@when_imported('math')
def add_logging(mod):
    mod.cos = logged(mod.cos)
    mod.sin = logged(mod.sin)

```

## 讨论

This recipe relies on the import hooks that were discussed in Recipe 10.11, with a slighttwist.

First, the role of the @when_imported decorator is to register handler functions that gettriggered on import. The decorator checks sys.modules to see if a module was alreadyloaded. If so, the handler is invoked immediately. Otherwise, the handler is added to alist in the _post_import_hooks dictionary. The purpose of _post_import_hooks issimply to collect all handler objects that have been registered for each module. In principle,more than one handler could be registered for a given module.

To trigger the pending actions in _post_import_hooks after module import, the PostImportFinder class is installed as the first item in sys.meta_path. If you recall fromRecipe 10.11, sys.meta_path contains a list of finder objects that are consulted in orderto locate modules. By installing PostImportFinder as the first item, it captures all moduleimports.

In this recipe, however, the role of PostImportFinder is not to load modules, but totrigger actions upon the completion of an import. To do this, the actual import is delegatedto the other finders on sys.meta_path. Rather than trying to do this directly, thefunction imp.import_module() is called recursively in the PostImportLoader class. Toavoid getting stuck in an infinite loop, PostImportFinder keeps a set of all the modulesthat are currently in the process of being loaded. If a module name is part of this set, itis simply ignored by PostImportFinder. This is what causes the import request to passto the other finders on sys.meta_path.

After a module has been loaded with imp.import_module(), all handlers currently registeredin _post_import_hooks are called with the newly loaded module as an argument.

From this point forward, the handlers are free to do what they want with the module.A major feature of the approach shown in this recipe is that the patching of a moduleoccurs in a seamless fashion, regardless of where or how a module of interest is actuallyloaded. You simply write a handler function that’s decorated with @when_imported()and it all just magically works from that point forward.

One caution about this recipe is that it does not work for modules that have been explicitlyreloaded using imp.reload(). That is, if you reload a previously loaded module,the post import handler function doesn’t get triggered again (all the more reason to notuse reload() in production code). On the other hand, if you delete the module fromsys.modules and redo the import, you’ll see the handler trigger again.

More information about post-import hooks can be found in PEP 369 . As of this writing,the PEP has been withdrawn by the author due to it being out of date with the currentimplementation of the importlib module. However, it is easy enough to implementyour own solution using this recipe.

# 10.13 安装私有的包

## 问题

You want to install a third-party package, but you don’t have permission to install packagesinto the system Python. Alternatively, perhaps you just want to install a packagefor your own use, not all users on the system.

## 解决方案

Python has a per-user installation directory that’s typically located in a directory suchas ~/.local/lib/python3.3/site-packages. To force packages to install in this directory, givethe –user option to the installation command. For example:

```py
python3 setup.py install --user

```

or

```py
pip install --user packagename

```

The user site-packages directory normally appears before the system site-packages directoryon sys.path. Thus, packages you install using this technique take priority overthe packages already installed on the system (although this is not always the case dependingon the behavior of third-party package managers, such as distribute or pip).

## 讨论

Normally, packages get installed into the system-wide site-packages directory, which isfound in a location such as /usr/local/lib/python3.3/site-packages. However, doing sotypically requires administrator permissions and use of the sudo command. Even if youhave permission to execute such a command, using sudo to install a new, possibly unproven,package might give you some pause.

Installing packages into the per-user directory is often an effective workaround thatallows you to create a custom installation.

As an alternative, you can also create a virtual environment, which is discussed in thenext recipe.

# 10.14 创建新的 Python 环境

## 问题

You want to create a new Python environment in which you can install modules andpackages. However, you want to do this without installing a new copy of Python ormaking changes that might affect the system Python installation.

## 解决方案

You can make a new “virtual” environment using the pyvenv command. This commandis installed in the same directory as the Python interpreter or possibly in the Scriptsdirectory on Windows. Here is an example:

```py
bash % pyvenv Spam
bash %

```

The name supplied to pyvenv is the name of a directory that will be created. Uponcreation, the Spam directory will look something like this:

```py
bash % cd Spam
bash % ls
bin include lib pyvenv.cfg
bash %

```

In the bin directory, you’ll find a Python interpreter that you can use. For example:

```py
bash % Spam/bin/python3
Python 3.3.0 (default, Oct 6 2012, 15:45:22)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from pprint import pprint
>>> import sys
>>> pprint(sys.path)
['',
'/usr/local/lib/python33.zip',
'/usr/local/lib/python3.3',
'/usr/local/lib/python3.3/plat-darwin',
'/usr/local/lib/python3.3/lib-dynload',
'/Users/beazley/Spam/lib/python3.3/site-packages']
>>>

```

A key feature of this interpreter is that its site-packages directory has been set to thenewly created environment. Should you decide to install third-party packages, they willbe installed here, not in the normal system site-packages directory.

## 讨论

The creation of a virtual environment mostly pertains to the installation and managementof third-party packages. As you can see in the example, the sys.path variablecontains directories from the normal system Python, but the site-packages directory hasbeen relocated to a new directory.

With a new virtual environment, the next step is often to install a package manager,such as distribute or pip. When installing such tools and subsequent packages, youjust need to make sure you use the interpreter that’s part of the virtual environment.This should install the packages into the newly created site-packages directory.

Although a virtual environment might look like a copy of the Python installation, itreally only consists of a few files and symbolic links. All of the standard library files andinterpreter executables come from the original Python installation. Thus, creating suchenvironments is easy, and takes almost no machine resources.

By default, virtual environments are completely clean and contain no third-party addons.If you would like to include already installed packages as part of a virtual environment,create the environment using the –system-site-packages option. For example:

```py
bash % pyvenv --system-site-packages Spam
bash %

```

More information about pyvenv and virtual environments can be found in[PEP 405](https://www.python.org/dev/peps/pep-0405/) .

# 10.15 分发包

## 问题

You’ve written a useful library, and you want to be able to give it away to others.

## 解决方案

If you’re going to start giving code away, the first thing to do is to give it a unique nameand clean up its directory structure. For example, a typical library package might looksomething like this:

```py
projectname/
    README.txt
    Doc/
        documentation.txt
    projectname/
        __init__.py
        foo.py
        bar.py
        utils/
            __init__.py
            spam.py
            grok.py
    examples/
        helloworld.py
        ...

```

To make the package something that you can distribute, first write a setup.py file thatlooks like this:

```py
# setup.py
from distutils.core import setup

setup(name='projectname',
    version='1.0',
    author='Your Name',
    author_email='you@youraddress.com',
    url='http://www.you.com/projectname',
    packages=['projectname', 'projectname.utils'],
)

```

Next, make a file MANIFEST.in that lists various nonsource files that you want to includein your package:

```py
# MANIFEST.in
include *.txt
recursive-include examples *
recursive-include Doc *

```

Make sure the setup.py and MANIFEST.in files appear in the top-level directory of yourpackage. Once you have done this, you should be able to make a source distribution bytyping a command such as this:

```py
% bash python3 setup.py sdist

```

This will create a file such as projectname-1.0.zip or projectname-1.0.tar.gz, dependingon the platform. If it all works, this file is suitable for giving to others or uploading tothe [Python Package Index](http://pypi.python.org/) [[`pypi.python.org/`](http://pypi.python.org/)].

## 讨论

For pure Python code, writing a plain setup.py file is usually straightforward. One potentialgotcha is that you have to manually list every subdirectory that makes up thepackages source code. A common mistake is to only list the top-level directory of apackage and to forget to include package subcomponents. This is why the specificationfor packages in setup.py includes the list packages=[‘projectname', ‘projectname.utils'].

As most Python programmers know, there are many third-party packaging options,including setuptools, distribute, and so forth. Some of these are replacements for thedistutils library found in the standard library. Be aware that if you rely on thesepackages, users may not be able to install your software unless they also install therequired package manager first. Because of this, you can almost never go wrong bykeeping things as simple as possible. At a bare minimum, make sure your code can beinstalled using a standard Python 3 installation. Additional features can be supportedas an option if additional packages are available.

Packaging and distribution of code involving C extensions can get considerably morecomplicated. Chapter 15 on C extensions has a few details on this. In particular, seeRecipe 15.2.