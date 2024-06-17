# 第十一章\. 文件和文本操作

本章涵盖了 Python 中与文件和文件系统相关的问题。*文件*是程序可以读取和/或写入的文本或字节流；*文件系统*是计算机系统上文件的分层存储库。

# 其他同样涉及文件处理的章节

文件是编程中至关重要的概念：因此，尽管本章是本书中最大的章节之一，其他章节也包含处理特定类型文件相关的内容。特别是，第十二章涉及与持久性和数据库功能相关的多种文件（CSV 文件在第十二章，JSON 文件在“json 模块”，pickle 文件在“pickle 模块”，shelve 文件在“shelve 模块”，DBM 和类似 DBM 的文件在“dbm 包”，以及 SQLite 数据库文件在“SQLite”），第二十二章处理 HTML 格式文件，第二十三章处理 XML 格式文件。

文件和流具有多种变体。它们的内容可以是任意字节或文本。它们可能适合读取、写入或两者兼而有之，并且它们可能是*缓冲的*，因此数据在进出文件时在内存中暂时保留。文件还可以允许*随机访问*，在文件内前进和后退，或跳转到文件中的特定位置进行读取或写入。本章涵盖了这些每一个主题。

此外，本章还涵盖了文件类对象的多态概念（实际上不是文件但在某种程度上像文件的对象）、处理临时文件和文件类对象的模块，以及帮助您访问文本和二进制文件内容并支持压缩文件和其他数据存档的模块。Python 的标准库支持多种[无损压缩](https://oreil.ly/iwsUw)，包括（按文本文件的压缩比例从高到低排序）：

+   [LZMA](https://oreil.ly/DuCbU)（例如由[xz](https://tukaani.org/xz)程序使用），请参见模块[lzma](https://oreil.ly/Kw54K)

+   [bzip2](https://oreil.ly/9rUEj)（例如由[bzip2](http://www.bzip.org)程序使用），请参见模块[bz2](https://oreil.ly/LXyg_)

+   [deflate](https://oreil.ly/k_iKs)（例如由[gzip](https://oreil.ly/kCtWx)和[zip](https://oreil.ly/an46l)程序使用），请参见模块[zlib](https://oreil.ly/-lWQP)，[gzip](https://oreil.ly/WyFJf)，和[zipfile](https://oreil.ly/c7VxO)

[tarfile 模块](https://oreil.ly/ZJlzh)可让您读取和写入使用任何这些算法压缩的 TAR 文件。zipfile 模块允许您读取和写入 ZIP 文件，并处理 bzip2 和 LZMA 压缩。本章中我们涵盖了这两个模块。本书不涵盖压缩的详细内容；详见[在线文档](https://oreil.ly/U0-Zx)。

在本章的其余部分，我们将所有文件和类似文件的对象称为文件。

在现代 Python 中，输入/输出（I/O）由标准库的 io 模块处理。os 模块提供了许多操作文件系统的函数，因此本章还介绍了该模块。然后讨论了操作文件系统的内容（比较、复制和删除目录和文件；处理文件路径；以及访问低级文件描述符），这些功能由 os 模块、os.path 模块以及新的更可取的 pathlib 模块提供，后者提供了一种面向对象的文件系统路径的方法。有关称为*内存映射文件*的跨平台进程间通信（IPC）机制，请参见第 15 章中介绍的 mmap 模块。

虽然大多数现代程序依赖于图形用户界面（GUI），通常通过浏览器或智能手机应用程序，但基于文本的非图形“命令行”用户界面因其易用性、使用速度和可脚本化而仍然非常受欢迎。本章最后讨论了 Python 中的非 GUI 文本输入和输出，包括“文本输入和输出”中的终端文本 I/O，“更丰富的文本 I/O”，以及如何构建能够跨语言和文化理解的软件，在“国际化”中有所介绍。

# io 模块

如本章介绍的那样，io 是 Python 标准库中负责提供读取或写入文件的最常见方式的模块。在现代 Python 中，内置函数 open 是函数 io.open 的别名。使用 io.open（或其内置别名 open）可以创建一个 Python 文件对象，用于从文件中读取和/或写入数据，这个文件对象在底层操作系统中是可见的。传递给 open 的参数决定了返回的对象类型。如果是文本类型，返回的对象可以是 io.TextIOWrapper 的实例；如果是二进制类型，则可能是 io.BufferedReader、io.BufferedWriter 或 io.BufferedRandom 中的一个，具体取决于它是只读、只写还是读写。本节涵盖了各种类型的文件对象，以及如何创建和使用*临时*文件（在磁盘上或甚至是在内存中）的重要问题。

# I/O Errors Raise OSError

Python 对与文件对象相关的任何 I/O 错误作出响应，通过引发内置异常类 OSError 的实例来处理（有许多有用的子类存在，如“OSError 子类”中所述）。导致此异常的错误包括打开调用失败、对文件对象调用不适用于该方法的方法（例如，在只读文件上进行写入或在不可寻址文件上进行寻址），以及由文件对象的方法诊断出的实际 I/O 错误。

io 模块还提供了底层类（抽象和具体），通过继承和组合（也称为*包装*），构成了您的程序通常使用的文件对象。本书不涵盖这些高级主题。如果您可以访问数据的非常规通道或非文件系统数据存储，并希望为这些通道或存储提供文件接口，可以通过适当的子类化和包装使用 io 模块中的其他类来简化任务。有关这些高级任务的帮助，请参阅[在线文档](https://docs.python.org/3/library/io.xhtml)。

## 使用 open 创建文件对象

要创建 Python 文件对象，请使用以下语法调用 open：

```py
open(file, mode='r', buffering=-1, encoding=`None`, errors='strict', 
     newline=`None`, closefd=`True`, opener=os.open)
```

file 可以是字符串或 pathlib.Path 实例（作为底层操作系统所见的任何文件路径），也可以是整数（由 os.open 返回的操作系统级*文件描述符*，或您传递为 opener 参数的任何函数返回的值）。当 file 是路径（字符串或 pathlib.Path 实例）时，open 打开指定的文件（根据模式参数可能会创建文件——尽管其名称是 open，它不仅仅用于打开现有文件：它也可以创建新文件）。当 file 是整数时，操作系统文件必须已通过 os.open 打开。

# 以 Python 风格打开文件

open 是一个上下文管理器：请使用 **with** open(...) **as** *f*:，而不是 *f* = open(...)，以确保文件 *f* 在 **with** 语句的主体完成后关闭。

open 创建并返回适当的 io 模块类的实例 *f*，具体取决于模式和缓冲设置。我们将所有这些实例称为文件对象；它们在彼此之间是多态的。

### 模式

mode 是一个可选的字符串，指示如何打开（或创建）文件。模式的可能取值列在表 11-1 中。

表 11-1\. 模式设置

| 模式 | 含义 |
| --- | --- |
| 'a' | 文件以只写方式打开。如果文件已经存在，则保持文件不变，并将写入的数据附加到现有内容。如果文件不存在，则创建文件。在此模式下打开的文件，调用*f*.seek 方法会改变*f*.tell 方法的结果，但不会改变写入位置：该写入位置始终保持在文件末尾。 |
| 'a+' | 文件同时用于读取和写入，因此可以调用所有*f*的方法。如果文件已经存在，则保持不变，并且您写入的数据将附加到现有内容中。如果文件不存在，则创建文件。在文件上调用*f*.seek，根据底层操作系统的不同，可能在下一个*f* *写*数据的 I/O 操作时没有效果，但在下一个*f* *读*数据的 I/O 操作时通常会正常工作。 |
| 'r' | 文件必须已经存在，并且以只读模式打开（这是默认设置）。 |
| 'r+' | 文件必须存在，并且同时用于读取和写入，因此可以调用所有*f*的方法。 |
| 'w' | 文件以只写模式打开。如果文件已经存在，则将其截断为零长度并覆盖，如果文件不存在，则创建文件。 |
| 'w+' | 文件同时用于读取和写入，因此可以调用所有*f*的方法。如果文件已经存在，则将其截断为零长度并覆盖，如果文件不存在，则创建文件。 |

### 二进制和文本模式

模式字符串可以包括表 11-1 中的任何值，后跟 b 或 t。b 表示文件应以二进制模式打开（或创建），而 t 表示文本模式。当没有包括 b 或 t 时，默认为文本（即'r'类似于'rt'，'w+'类似于'w+t'，依此类推），但根据[Python 之禅](https://oreil.ly/QO8-Y)，“显式胜于隐式”。

二进制文件允许您读取和/或写入字节类型的字符串，而文本文件允许您读取和/或写入 Unicode 文本字符串类型的 str。对于文本文件，当底层通道或存储系统处理字节（大多数情况下是这样）时，编码（已知 Python 的编码名称）和错误（一个错误处理程序名称，如'strict'、'replace'等，在表 9-1 的 decode 下有所涵盖）很重要，因为它们指定了如何在文本和字节之间进行转换，以及在编码和解码错误时该如何处理。

### 缓冲

缓冲是一个整数值，表示您请求的文件缓冲策略。当缓冲为 0 时，文件（必须为二进制模式）是无缓冲的；其效果就像每次写入文件时都刷新文件的缓冲区一样。当缓冲为 1 时，文件（必须在文本模式下打开）是行缓冲的，这意味着每次写入\n 到文件时都会刷新文件的缓冲区。当缓冲大于 1 时，文件使用大约缓冲字节数的缓冲区，通常向上舍入为某个方便驱动程序软件的值。当缓冲小于 0 时，默认值由文件流类型决定。通常，这个默认值是与交互式流相对应的行缓冲，并且对于其他文件使用 io.DEFAULT_BUFFER_SIZE 字节的缓冲区。

### 顺序和非顺序（“随机”）访问

文件对象*f*本质上是顺序的（一系列字节或文本）。读取时，按照它们出现的顺序获取字节或文本。写入时，按照写入的顺序添加字节或文本。

对于文件对象*f*支持非顺序访问（也称为随机访问），它必须跟踪其当前位置（存储中下一个读取或写入操作开始传输数据的位置），并且文件的底层存储必须支持设置当前位置。当*f*.seekable 返回**True**时，表示*f*支持非顺序访问。

打开文件时，默认的初始读/写位置是在文件的开头。使用模式'a'或'a+'打开*f*将*f*的读/写位置设置为在写入数据之前的文件末尾。当向文件对象*f*写入或读取*n*字节时，*f*的位置将前进*n*。您可以通过调用*f*.tell 查询当前位置，并通过调用*f*.seek 更改位置，这两者在下一节中有详细介绍。

在文本模式的*f*上调用*f*.seek 时，传递的偏移量必须为 0（将*f*定位在开头或结尾，取决于*f*.seek 的第二个参数），或者是先前调用*f*.tell 返回的不透明结果¹，以将*f*定位回之前“书签”过的位置。

## 文件对象的属性和方法

文件对象*f*提供了表 11-2 中记录的属性和方法。

表 11-2\. 文件对象的属性和方法

| close | close() 关闭文件。在*f*.close 之后不能再调用*f*的其他方法。允许多次调用*f*.close 而无害。 |
| --- | --- |
| closed | *f*.closed 是一个只读属性。当*f*.close()已调用时，返回**True**；否则返回**False**。 |
| encoding | *f*.encoding 是一个只读属性，是指定的编码名称（如在“Unicode”中介绍）。该属性在二进制文件上不存在。 |
| fileno | fileno() 返回*f*的文件在操作系统级别的文件描述符（一个整数）。文件描述符在“os 模块的文件和目录函数”中有介绍。 |
| flush | flush() 请求将*f*的缓冲区写入操作系统，使得系统看到的文件与 Python 代码写入的内容完全一致。根据平台和*f*的底层文件性质，*f*.flush 可能无法确保所需的效果。 |
| isatty | isatty() 当*f*的底层文件是交互流（例如来自终端或到终端）时返回**True**；否则返回**False**。 |
| mode | *f*.mode 是一个只读属性，其值是在创建*f*的 io.open 调用中使用的模式字符串。 |
| name | *f*.name 是一个只读属性，其值是创建*f*的 io.open 调用中使用的文件（str 或 bytes）或 int。当 io.open 使用 pathlib.Path 实例*p*调用时，*f*.name 是 str(*p*)。 |
| read | read(*size*=-1, /) 当*f*以二进制模式打开时，从*f*的文件中读取最多*size*字节并将它们作为字节字符串返回。如果文件在读取*size*字节之前结束，则 read 读取并返回少于*size*字节。当*size*小于 0 时，read 读取并返回直到文件末尾的所有字节。当文件的当前位置在文件末尾或*size*等于 0 时，read 返回一个空字符串。当*f*以文本模式打开时，*size*表示字符数而不是字节数，read 返回一个文本字符串。 |
| readline | readline(*size*=-1, /) 从*f*的文件中读取并返回一行，直到行尾（\n）为止。当*size*大于或等于 0 时，最多读取*size*字节。在这种情况下，返回的字符串可能不以\n 结尾。当文件的当前位置在文件末尾或*size*等于 0 时，readline 返回一个空字符串。当 readline 读取到文件末尾且未找到\n 时，\n 可能也会不存在。 |
| readlines | readlines(*size*=-1, /) 读取并返回文件*f*中所有行的列表，每行都是以\n 结尾的字符串。如果*size* > 0，则 readlines 在收集了约*size*字节的数据后停止并返回列表；在这种情况下，列表中的最后一个字符串可能不以\n 结尾。 |

| seek | seek(*pos*, *how*=io.SEEK_SET, /) 将*f*的当前位置设置为离参考点*pos*字节偏移的整数。*how*指示参考点。io 模块有名为 SEEK_SET、SEEK_CUR 和 SEEK_END 的属性，分别指定参考点为文件的开头、当前位置或结尾。 |

当*f*以文本模式打开时，*f*.seek 必须为 0，或者对于 io.SEEK_SET，*f*.seek 的*pos*必须是上一次调用*f*.tell 的结果。

当*f*以模式'a'或'a+'打开时，在某些平台上，写入*f*的数据会追加到已经存在*f*中的数据，而不管对*f*.seek 的调用。 |

| tell | tell() 返回*f*的当前位置：对于二进制文件，这是从文件开头的字节偏移量，对于文本文件，这是一个不透明的值，可在将来调用*f*.seek 将*f*定位回当前位置使用。 |
| --- | --- |
| truncate | truncate(*size*=None, /) 截断*f*的文件，*f*必须已经打开以进行写入。当*size*存在时，将文件截断为最多*size*字节。当*size*不存在时，使用*f*.tell()作为文件的新大小。*size*可能比当前文件大小大；在这种情况下，结果行为取决于平台。 |
| write | write(*s*, /) 将字符串*s*的字节（根据*f*的模式是二进制还是文本）写入文件。 |

| writelines | writelines(*lst*, /) 类似于：

```py
`for` *`line`* `in` *`lst`*: *`f`*.write(*`line`*)
```

*lst* 可迭代的字符串是否为行并不重要：尽管其名称如此，writelines 方法只是将每个字符串依次写入文件。特别地，writelines 不会添加行结束标记：如果需要的话，这些标记必须已经存在于 *lst* 的项目中。

## 文件对象的迭代

一个用于读取的文件对象 *f* 也是一个迭代器，其项是文件的行。因此，下面的循环：

```py
`for` *`line`* `in` *`f`*:
```

遍历文件的每一行。由于缓冲问题，如果在这样的循环中过早中断（例如使用 **break**），或者调用 next(*f*) 而不是 *f*.readline()，则文件的位置将设置为任意值。如果您希望从在 *f* 上进行迭代切换到在 *f* 上调用其他读取方法，请确保通过适当地调用 *f*.seek 将文件的位置设置为已知值。好处是，直接在 *f* 上进行循环具有非常好的性能，因为这些规格允许循环使用内部缓冲以最小化 I/O，即使对于大文件也不会占用过多的内存。

## 文件类对象与多态性

当一个对象 *x* 表现得像一个由 io.open 返回的文件对象时，它就是类似文件的。这意味着我们可以像使用文件一样使用 *x*。使用这样一个对象的代码（称为对象的 *客户端代码*）通常通过参数获取对象，或者通过调用返回该对象的工厂函数获取对象。例如，如果客户端代码在 *x* 上唯一调用的方法是 *x*.read（无参数），那么为了这段代码，*x* 需要提供的仅仅是一个可调用的、无参数的 read 方法，并且返回一个字符串。其他客户端代码可能需要 *x* 实现更大的文件方法子集。文件类对象和多态性并非绝对概念：它们是相对于某些特定客户端代码对对象提出的要求而言的。

多态是面向对象编程中强大的一个方面，文件类对象是多态的一个很好的例子。一个写入或读取文件的客户端模块可以自动地被重用于其他数据，只要该模块不通过类型检查来破坏多态性。当我们讨论内置类型和 isinstance 在表 8-1 中时，我们提到类型检查通常最好避免，因为它会阻碍 Python 的正常多态性。通常，为了支持你的客户端代码的多态性，你只需要避免类型检查。

你可以通过编写自己的类来实现类似文件的对象（如在第四章中介绍的），并定义客户端代码所需的特定方法，比如 read。文件样对象*fl*不必实现真实文件对象*f*的所有属性和方法。如果你可以确定客户端代码在*fl*上调用哪些方法，你可以选择只实现那些子集。例如，当*fl*只用于写入时，*fl*不需要“读取”方法，如 read、readline 和 readlines。

如果你想要一个类似文件的对象而不是真实文件对象的主要原因是将数据保存在内存中而不是磁盘上，可以使用 io 模块的 StringIO 或 BytesIO 类（详见“内存中的文件：io.StringIO 和 io.BytesIO”）。这些类提供了在内存中保存数据并且在大多数情况下表现得与其他文件对象多态的文件对象。如果你正在运行多个进程，希望通过类似文件的对象进行通信，考虑使用 mmap，详见第十五章。

# tempfile 模块

tempfile 模块允许你以平台所允许的最安全方式创建临时文件和目录。当你处理的数据量可能不适合放入内存，或者你的程序必须写入稍后由另一个进程使用的数据时，临时文件通常是一个好主意。

此模块中函数的参数顺序有点混乱：为了使你的代码更易读，请始终使用命名参数语法调用这些函数。tempfile 模块公开了表格 11-3 中概述的函数和类。

表格 11-3\. tempfile 模块的函数和类

| mkdtemp | mkdtemp(suffix=**None**, prefix=**None**, dir=None) 安全创建一个新的临时目录，只有当前用户可以读取、写入和搜索，然后返回临时目录的绝对路径。你可以选择传递参数来指定用作临时文件名开头（前缀）和结尾（后缀）的字符串，以及创建临时文件的目录路径（dir）。确保在使用完毕后删除临时目录是你程序的责任。 |
| --- | --- |

| mkdtemp *(cont.)* | 这里是一个典型的使用示例，它创建一个临时目录，将其路径传递给另一个函数，最后确保目录（及其所有内容）被删除：

```py
`import` tempfile, shutil
path = tempfile.mkdtemp()
`try`:
    use_dirpath(path)
`finally`:
    shutil.rmtree(path)
```

|

| mkstemp | mkstemp(suffix=**None**, prefix=**None**, dir=**None**, text=**False**) 安全地创建一个新的临时文件，只有当前用户可读可写，不可执行，并且不被子进程继承；返回一对（*fd*，*path*），其中 *fd* 是临时文件的文件描述符（由 os.open 返回，见 Table 11-18）, 字符串 *path* 是临时文件的绝对路径。可选参数 suffix，prefix 和 dir 类似于函数 mkdtemp。如果你想让临时文件成为文本文件，请显式传递参数 text=**True**。

确保在使用完临时文件后将其删除由你来负责。mkstemp 不是一个上下文管理器，所以你不能使用 **with** 语句；最好使用 **try**/**finally** 代替。以下是一个典型的使用示例，创建一个临时文本文件，关闭它，将其路径传递给另一个函数，最后确保文件被删除：

```py
`import` tempfile, os
fd, path = tempfile.mkstemp(suffix='.txt', 
                            text=`True`)
`try`:
    os.close(fd)
    use_filepath(path)
`finally`:
    os.unlink(path)
```

|

| Nam⁠e⁠d​T⁠e⁠m⁠p⁠o⁠r⁠a⁠r⁠y​F⁠i⁠l⁠e | NamedTemporaryFile(mode='w+b', bufsize=-1, suffix=**None**, prefix=**None**, dir=**None**)

与 TemporaryFile（稍后在此表中讨论）类似，不同之处在于临时文件确实在文件系统上有一个名称。使用文件对象的 name 属性来访问该名称。一些平台（主要是 Windows）不允许再次打开文件；因此，如果要确保程序跨平台运行，名称的有用性是有限的。如果需要将临时文件的名称传递给另一个打开文件的程序，可以使用函数 mkstemp 而不是 NamedTemporaryFile 来保证正确的跨平台行为。当然，当你选择使用 mkstemp 时，确保在完成后删除文件是需要注意的。从 NamedTemporaryFile 返回的文件对象是一个上下文管理器，因此可以使用 with 语句。|

| Spoo⁠l⁠e⁠d​T⁠e⁠m⁠p⁠o⁠r⁠a⁠r⁠y​F⁠i⁠l⁠e | SpooledTemporaryFile(mode='w+b', bufsize=-1, suffix=**None**, prefix=**None**, dir=**None**) 类似于 TemporaryFile（见下文），不同之处在于 SpooledTemporaryFile 返回的文件对象可以保留在内存中，如果空间允许，直到你调用其 fileno 方法（或其 rollover 方法，它确保文件被写入磁盘，无论其大小如何）。因此，只要有足够的内存未被其他方式使用，使用 SpooledTemporaryFile 可能性能更好。 |
| --- | --- |
| TemporaryDirectory | TemporaryDirectory(suffix=None, prefix=**None**, dir=**None**, ignore_cleanup_errors=**False**) 创建临时目录，类似于 mkdtemp（传递可选参数 suffix、prefix 和 dir）。返回的目录对象是一个上下文管理器，因此您可以使用**with**语句确保在完成后立即删除它。或者，当您不将其用作上下文管理器时，可以使用其内置类方法 cleanup（*而不是* shutil.rmtree）显式删除和清理目录。将 ignore_cleanup_errors 设置为 **True** 可以忽略清理过程中的未处理异常。临时目录及其内容在目录对象关闭时（无论是隐式地由垃圾回收还是显式地通过 cleanup 调用）立即删除。 |

| TemporaryFile | TemporaryFile(mode='w+b', bufsize=-1, suffix=**None**, prefix=**None**, dir=**None**)

使用 mkstemp 创建临时文件（传递给 mkstemp 可选参数 suffix、prefix 和 dir），使用 os.fdopen 将其转换为文件对象，见 Table 11-18（传递给 fdopen 可选参数 mode 和 bufsize），然后返回文件对象。临时文件在文件对象关闭时立即删除（隐式或显式）。为了提高安全性，如果您的平台允许（类 Unix 平台允许，Windows 不允许），临时文件在文件系统上没有名称。从 TemporaryFile 返回的文件对象是一个上下文管理器，因此您可以使用**with**语句确保在完成后立即删除它。

# 用于文件 I/O 的辅助模块

文件对象提供了文件 I/O 所需的功能。然而，其他 Python 库模块提供了方便的附加功能，在几个重要情况下使 I/O 更加简单和方便。我们将在这里看到其中的两个模块。

## fileinput 模块

fileinput 模块允许您循环遍历文本文件列表中的所有行。性能很好——与直接迭代每个文件的性能相当，因为使用缓冲区减少 I/O。因此，无论何时发现其丰富功能方便，都可以使用该模块进行基于行的文件输入，而不必担心性能问题。模块的关键函数是 input；fileinput 还提供了支持相同功能的 FileInput 类。两者在 Table 11-4 中描述。

Table 11-4\. fileinput 模块的关键类和函数

| FileInput | **类** FileInput(files=**None**, inplace=**False**, backup='', mode='r', openhook=**None**, encoding=**None**, errors=**None**) 创建并返回 FileInput 类的实例*f*。参数与接下来介绍的 fileinput.input 相同，*f*的方法具有与文件输入模块的其他函数相同的名称、参数和语义（见表 11-5）。*f*还提供了一个 readline 方法，它读取并返回下一行。使用 FileInput 类来嵌套或混合从多个文件序列中读取行的循环。 |
| --- | --- |

| input | input(files=None, inplace=**False**, backup='', mode='r', openhook=**None**, encoding=**None**, errors=**None**)

返回一个 FileInput 的实例，它是一个在文件中生成行的可迭代对象；该实例是全局状态，因此文件输入模块的所有其他函数（见表 11-5）都操作相同的共享状态。文件输入模块的每个函数直接对应于类 FileInput 的一个方法。

files 是要依次打开和读取的文件名序列。当 files 是一个字符串时，它是要打开和读取的单个文件名。当 files 为**None**时，input 使用 sys.argv[1:]作为文件名列表。文件名'-'表示标准输入（sys.stdin）。当文件名序列为空时，input 读取 sys.stdin。

当 inplace 为**False**（默认值）时，input 只读取文件。当 inplace 为**True**时，input 将正在读取的每个文件（除标准输入外）移动到备份文件，并重定向标准输出（sys.stdout）以写入具有与正在读取的原始文件相同路径的新文件。这样，您可以模拟就地覆盖文件。如果 backup 是以点开头的字符串，则 input 将 backup 用作备份文件的扩展名，并且不会删除备份文件。如果 backup 是一个空字符串（默认值），input 将使用*.bak*并在关闭输入文件时删除每个备份文件。关键字参数 mode 可以是'r'，默认值，也可以是'rb'。

您可以选择传递一个 openhook 函数来替代 io.open。例如，openhook=fileinput.hook_compressed 可以解压缩带有扩展名*.gz*或*.bz2*的任何输入文件（与 inplace=**True**不兼容）。您可以编写自己的 openhook 函数来解压缩其他文件类型，例如使用 LZMA 解压缩*.xz*文件；使用[fileinput.hook_compressed 的 Python 源代码](https://oreil.ly/qXxx1)作为模板。3.10+ 您还可以传递 encoding 和 errors，它们将作为关键字参数传递给钩子。 |

| ^(a) LZMA 支持可能需要使用可选的附加库构建 Python。 |
| --- |

列在表 11-5 中的文件输入模块的函数在由 fileinput.input 创建的全局状态上运行，如果有的话；否则，它们会引发 RuntimeError。

表 11-5\. fileinput 模块的附加函数

| close | close() 关闭整个序列，停止迭代并且没有文件保持打开状态。 |
| --- | --- |
| filelineno | filelineno() 返回目前为止从正在读取的文件中读取的行数。例如，如果刚刚从当前文件读取了第一行，则返回 1。 |
| filename | filename() 返回当前正在读取的文件的名称，如果尚未读取任何行则返回 **None**。 |
| isfirstline | isfirstline() 返回 **True** 或 **False**，就像 filelineno() == 1 一样。 |
| isstdin | isstdin() 当前正在读取的文件是 sys.stdin 时返回 **True**；否则返回 **False**。 |
| lineno | lineno() 返回自调用输入函数以来读取的总行数。 |
| nextfile | nextfile() 关闭当前正在读取的文件：下一行要读取的是下一个文件的第一行。 |

这里是使用 fileinput 进行“多文件搜索和替换”的典型示例，将一个字符串更改为另一个字符串，应用于通过命令行参数传递给脚本的文本文件：

```py
`import` fileinput
`for` line `in` fileinput.input(inplace=`True`):
    print(line.replace('foo', 'bar'), end='')
```

在这种情况下，包括 end='' 参数来打印是很重要的，因为每行在末尾都有其换行符 \n，并且您需要确保 print 不会再添加另一个（否则每个文件最终会变成“双空格”）。

你也可以使用由 fileinput.input 返回的 FileInput 实例作为上下文管理器。与 io.open 一样，这将在退出 **with** 语句时关闭由 FileInput 打开的所有文件，即使发生异常也是如此：

```py
`with` fileinput.input('file1.txt', 'file2.txt') `as` infile:
    dostuff(infile)
```

## 结构体 Module

结构体模块允许您将二进制数据打包成字节串，并将这样的字节串解包回它们所表示的 Python 数据。这对于许多类型的低级编程很有用。通常，您使用 struct 来解释来自具有某些指定格式的二进制文件的数据记录，或者准备要写入此类二进制文件的记录。模块的名称来自于 C 语言中的关键字 struct，用于类似的目的。在任何错误情况下，结构模块的函数会引发异常，这些异常是 struct.error 类的实例。

结构体模块依赖于遵循特定语法的 *结构格式字符串*。格式字符串的第一个字符给出了打包数据的[字节顺序](https://oreil.ly/rqogV)、大小和对齐方式；选项列在表 11-6 中。

表 11-6\. 结构格式字符串可能的第一个字符

| Character | Meaning |
| --- | --- |
| @ | 本地字节顺序、本地数据大小和本地对齐方式适用于当前平台；如果第一个字符不是列出的字符之一，则这是默认值（请注意，在 表 11-7 中，格式 P 仅适用于这种类型的结构格式字符串）。在需要检查系统字节顺序时，请查看字符串 sys.byteorder；大多数 CPU 今天使用 'little'，但 'big' 是互联网核心协议 TCP/IP 的“网络标准”。 |
| = | 当前平台的本机字节顺序，但标准大小和对齐方式。 |
| < | 小端字节顺序；标准大小和对齐方式。 |
| >, ! | 大端/网络标准字节顺序；标准大小和对齐方式。 |

标准大小在 表 11-7 中指示。标准对齐方式表示没有强制对齐，根据需要使用显式填充字节。本机大小和对齐方式是平台的 C 编译器使用的内容。本机字节顺序可以将最高有效字节放在最低地址（大端）或最高地址（小端），这取决于平台。

在可选的第一个字符之后，格式字符串由一个或多个格式字符组成，每个格式字符可选择地由一个计数（用十进制数字表示的整数）前置。常用格式字符列在 表 11-7 中；完整列表请参阅 [在线文档](https://oreil.ly/yz7bI)。对于大多数格式字符，计数表示重复（例如，'3h' 等同于 'hhh'）。当格式字符为 s 或 p —— 即字节串时，计数不表示重复：它是字符串中的总字节数。您可以在格式之间自由使用空白字符，但计数和其格式字符之间不可有空格。格式 s 表示长度固定的字节串，与其计数相同（Python 字符串会被截断或根据需要使用空字节 b'\0' 进行填充）。格式 p 表示“类似帕斯卡”的字节串：第一个字节是随后的有效字节数，实际内容从第二个字节开始。计数是总字节数，包括长度字节在内。

表 11-7\. struct 的常用格式字符

| 字符 | C 类型 | Python 类型 | 标准大小 |
| --- | --- | --- | --- |
| B | 无符号字符 | int | 1 byte |
| b | 有符号字符 | int | 1 byte |
| c | char | bytes (length 1) | 1 byte |
| d | 双精度浮点数 | float | 8 bytes |
| f | 浮点数 | float | 4 bytes |
| H | 无符号短整型 | int | 2 bytes |
| h | 有符号短整型 | int | 2 bytes |
| I | 无符号整型 | long | 4 bytes |
| i | 有符号整型 | int | 4 bytes |
| L | 无符号长整型 | long | 4 bytes |
| l | 有符号长整型 | int | 4 bytes |
| P | void* | int | N/A |
| p | char[] | bytes | N/A |
| s | char[] | bytes | N/A |
| x | 填充字节 | 无值 | 1 byte |

struct 模块提供了 表 11-8 中涵盖的函数。

表 11-8\. struct 模块的函数

| calcsize | calcsize(*fmt*, /) 返回格式字符串 *fmt* 对应的字节大小。 |
| --- | --- |
| iter_unpack | iter_unpack(*fmt*, *buffer*, /) 从 *buffer* 中按格式字符串 *fmt* 迭代解包。返回一个迭代器，该迭代器将从 *buffer* 中读取大小相等的块，直到消耗完所有内容；每次迭代都会产生一个由 *fmt* 指定的元组。*buffer* 的大小必须是格式所需的大小的倍数，如 struct.calcsize(*fmt*) 所反映的那样。 |
| pack | pack(*fmt*, **values*, /) 将值按格式字符串 *fmt* 打包，并返回生成的字节串。*values* 必须与 *fmt* 所需的值的数量和类型匹配。 |
| pack_into | pack_into(*fmt*, *buffer*, *offset*, **values*, /) 将值按格式字符串 *fmt* 打包到可写入的缓冲区 *buffer*（通常是 `bytearray` 的实例），从索引 *offset* 开始。*values* 必须与 *fmt* 所需的值的数量和类型匹配。`len(buffer[offset:])` 必须 >= `struct.calcsize(*fmt*)`。 |
| unpack | unpack(*fmt*, *s*, /) 根据格式字符串 *fmt* 解包字节串 *s*，并返回值的元组（如果只有一个值，则返回一个单项元组）。`len(*s*)` 必须等于 `struct.calcsize(*fmt*)`。 |
| unpack_from | unpack_from(*fmt*, /, buffer, offset=0*) 从偏移量 *offset* 开始，根据格式字符串 *fmt* 解包字节串（或其他可读取的缓冲区）*buffer*，返回一个值的元组（如果只有一个值，则返回一个单项元组）。`len(buffer[offset:])` 必须 >= `struct.calcsize(*fmt*)`。 |

`struct` 模块还提供了一个 `Struct` 类，它以格式字符串作为参数进行实例化。该类的实例实现了 `pack`、`pack_into`、`unpack`、`unpack_from` 和 `iter_unpack` 方法，这些方法对应于前面表格中描述的函数；它们接受与相应模块函数相同的参数，但省略了在实例化时提供的 `fmt` 参数。这允许类只编译一次格式字符串并重复使用。`Struct` 对象还有一个 `format` 属性，保存对象的格式字符串，并且有一个 `size` 属性，保存结构的计算大小。

# 内存中的文件：`io.StringIO` 和 `io.BytesIO`

你可以通过编写 Python 类来实现类似文件的对象，这些类提供你需要的方法。如果你只想让数据驻留在内存中，而不是作为操作系统可见的文件，可以使用 `io` 模块的 `StringIO` 或 `BytesIO` 类。它们之间的区别在于 `StringIO` 的实例是文本模式文件，因此读取和写入消耗或产生文本字符串，而 `BytesIO` 的实例是二进制文件，因此读取和写入消耗或产生字节串。这些类在测试和其他需要将程序输出重定向用于缓冲或日志记录的应用程序中特别有用；"The print Function" 包括一个有用的上下文管理器示例 `redirect`，演示了这一点。

当你实例化任何一个类时，可以选择传递一个字符串参数，分别是 `str` 或 `bytes`，作为文件的初始内容。此外，你可以向 `StringIO`（但不是 `BytesIO`）传递参数 `newline='\n'` 来控制如何处理换行符（如在 [TextIoWrapper](https://oreil.ly/-cD05) 中所述）；如果 `newline` 是 `None`，则在所有平台上都将换行符写为 `\n`。除了 表 11-2 中描述的方法之外，任何一个类的实例 `f` 还提供一个额外的方法：

| getvalue | getvalue() 返回 *f* 的当前数据内容作为字符串（文本或字节）。在调用 *f*.close 后不能再调用 *f*.getvalue：close 释放 *f* 内部保留的缓冲区，而 getvalue 需要将缓冲区作为其结果返回。 |
| --- | --- |

# 存档和压缩文件

存储空间和传输带宽变得越来越便宜和充裕，但在许多情况下，通过使用压缩可以节省这些资源，尽管需要额外的计算工作。计算能力的成本比其他资源（如带宽）增长更快，因此压缩的流行度不断增长。Python 使得您的程序能够轻松支持压缩。本书不涵盖压缩的详细信息，但您可以在[在线文档](https://oreil.ly/QJzCW)中找到有关相关标准库模块的详细信息。

本节的其余部分涵盖了“存档”文件（在单个文件中收集文件和可选目录的集合），这些文件可能会压缩也可能不会压缩。Python 的标准库提供了两个模块来处理两种非常流行的存档格式：tarfile（默认情况下不会压缩其捆绑的文件）和 zipfile（默认情况下会压缩其捆绑的文件）。

## tarfile 模块

tarfile 模块允许您读取和编写[TAR 文件](https://oreil.ly/TvKqN)（与流行的打包程序（如 tar）处理的文件兼容的存档文件），可选择使用 gzip、bzip2 或 LZMA 压缩。TAR 文件通常以 *.tar* 或 *.tar.(压缩类型)* 扩展名命名。3.8+ 新建存档的默认格式为 POSIX.1-2001（pax）。 **python -m tarfile** 提供了一个有用的命令行接口以访问模块的功能：运行它而不带参数可以获取简短的帮助消息。

tarfile 模块提供了 T 表 11-9 中列出的函数。处理无效的 TAR 文件时，tarfile 的函数会引发 tarfile.TarError 的实例。

表 11-9\. tarfile 模块的类和函数

| is_tarfile | is_tarfile(*filename*) 当名为*filename*（可以是字符串，3.9+ 或文件或类文件对象）的文件在前几个字节上看起来像是一个有效的 TAR 文件（可能带有压缩），则返回 **True**；否则返回 **False**。 |
| --- | --- |

| open | open(name=**None**, mode='r', fileobj=**None**, bufsize=10240, ***kwargs*) 创建并返回一个 TarFile 实例 *f*，用于通过类文件对象 fileobj 读取或创建 TAR 文件。当 fileobj 为 **None** 时，name 可以是指定文件的字符串名称或路径对象；open 使用给定的模式（默认为 'r'）打开文件，并且 *f* 包装生成的文件对象。open 可以作为上下文管理器使用（例如，**with** tarfile.open(...) **as** *f*）。

# f.close 可能不会关闭 fileobj

当使用未为 **None** 的 fileobj 打开 *f* 时，调用 *f*.close 并不会关闭 fileobj。当 fileobj 是 io.BytesIO 的实例时，此 *f*.close 的行为非常重要：您可以在 *f*.close 后调用 fileobj.getvalue 来获取归档的可能已压缩数据作为字符串。这种行为也意味着您必须在调用 *f*.close 后显式调用 fileobj.close。

mode 可以是 'r' 以读取具有任何压缩（如果有的话）的现有 TAR 文件；'w' 以编写新的 TAR 文件，或截断并重写现有文件，而不使用压缩；或 'a' 以追加到现有的 TAR 文件，而不使用压缩。不支持向已压缩的 TAR 文件追加。要使用压缩编写新的 TAR 文件，mode 可以是 'w:gz' 以进行 gzip 压缩，'w:bz2' 以进行 bzip2 压缩，或 'w:xz' 以进行 LZMA 压缩。您可以使用 mode 字符串 'r:' 或 'w:' 以使用 bufsize 字节缓冲区读取或写入未压缩的不可寻址 TAR 文件；对于读取 TAR 文件，请使用普通 'r'，因为这将根据需要自动解压缩。

在指定压缩方式的模式字符串中，你可以使用竖线（&#124;）而不是冒号（:），以强制顺序处理和固定大小的块；这在（尽管非常不太可能的）情况下很有用，你可能会发现自己处理磁带设备！ |

### TarFile 类

[TarFile](https://oreil.ly/RwYNA) 是大多数 tarfile 方法的底层类，但不直接使用。通过使用 tarfile.open 创建的 TarFile 实例 *f*，提供了 Table 11-10 中详细描述的方法。

表 11-10\. TarFile 实例 *f* 的方法

| add | *f*.add(*name*, arcname=**None**, recursive=**True**, *, filter=**None**) 向归档 *f* 添加名为 *name* 的文件（可以是任何类型的文件、目录或符号链接）。当 arcname 不为 **None** 时，将其用作归档成员的名称，而不是使用 *name*。当 *name* 是一个目录且 recursive 为 **True** 时，add 方法将以排序顺序递归地添加该目录中以该目录为根的整个文件系统子树。可选的（仅命名的）参数 filter 是一个函数，用于调用每个要添加的对象。它接受一个 TarInfo 对象参数，并返回可能修改后的 TarInfo 对象或 **None**。在后一种情况下，add 方法将从归档中排除此 TarInfo 对象。 |
| --- | --- |
| addfile | *f*.addfile(*tarinfo*, fileobj=**None**) 向归档 *f* 添加一个 TarInfo 对象 *tarinfo*。如果 fileobj 不为 **None**，则将二进制类文件对象 fileobj 的前 *tarinfo*.size 字节添加到归档中。 |
| close | *f*.close() 关闭存档 *f*。您必须调用 close，否则可能会在磁盘上留下一个不完整的、无法使用的 TAR 文件。最好使用 **try**/**finally**（在 “try/finally”中讨论）进行此类必需的最终操作，或者更好地，使用 “with 语句和上下文管理器”中讨论的 **with** 语句。调用 *f*.close 不会关闭 fileobj，如果 *f* 是使用非 **None** fileobj 创建的。这在 fileobj 是 io.BytesIO 的实例时尤为重要：您可以在 *f*.close 后调用 fileobj.getvalue 来获取压缩的数据字符串。因此，您始终必须在 *f*.close **后**调用 fileobj.close（显式地或通过使用 **with** 语句隐式地）。 |
| extract | *f.*extract(*member*, path='', set_attrs=**True**, numeric_owner=**False**) 将由 *member*（名称或 TarInfo 实例）标识的存档成员提取到目录（或类似路径的对象）中由 path（默认为当前目录）命名的相应文件中。如果 set_attrs 为 **True**，则所有者和时间戳将设置为它们在 TAR 文件中保存的值；否则，提取文件的所有者和时间戳将使用当前用户和时间值进行设置。如果 numeric_owner 为 **True**，则从 TAR 文件中使用的 UID 和 GID 数字将用于设置提取文件的所有者/组；否则，将使用 TAR 文件中的命名值。（[在线文档](https://oreil.ly/Ugj8v)建议使用 extractall 而不是直接调用 extract，因为 extractall 在内部执行了额外的错误处理。） |

| extractall | *f*.extractall(path='.', members=**None**, numeric_owner=**False**) 类似于对 TAR 文件 *f* 的每个成员调用 extract，或者只对在成员参数中列出的成员调用 extract，并且对在写入提取的成员时发生的 chown、chmod 和 utime 错误进行额外的错误检查。

# 不要对来自不受信任来源的 Tarfile 使用 extractall

extractall 不检查提取文件的路径，因此存在提取文件具有绝对路径（或包含一个或多个..组件）并因此覆盖潜在敏感文件的风险。^(a) 最好逐个成员地读取并仅在其具有安全路径（即，没有绝对路径或相对路径带有任何..路径组件）时提取它。

|

| extractfile | *f*.extractfile(*member*) 提取由 *member*（名称或 TarInfo 实例）标识的存档成员，并返回一个具有方法 read、readline、readlines、seek 和 tell 的 io.BufferedReader 对象。 |
| --- | --- |
| getmember | *f*.getmember(*name*) 返回一个 TarInfo 实例，其中包含有关字符串 *name* 命名的存档成员的信息。 |
| getmembers | *f*.getmembers() 返回一个 TarInfo 实例列表，其中每个实例对应于存档 *f* 中的每个成员，顺序与存档中条目的顺序相同。 |
| getnames | *f*.getnames() 返回一个字符串列表，存档 *f* 中每个成员的名称，顺序与存档中条目的顺序相同。 |
| gettarinfo | *f*.gettarinfo(name=**None**, arcname=**None**, fileobj=**None**) 返回一个 TarInfo 实例，其中包含有关打开的文件对象 fileobj 的信息，当 fileobj 不是 **None** 时；否则，是路径字符串 name 的现有文件。name 可能是类似路径的对象。当 arcname 不是 **None** 时，它被用作生成的 TarInfo 实例的 name 属性。 |
| list | *f*.list(verbose=**True**, *, members=**None**) 输出存档 *f* 的目录到 sys.stdout。如果可选参数 verbose 是 **False**，则只输出存档成员的名称。如果给定可选参数 members，则必须是 getmembers 返回的列表的子集。 |
| next | *f*.next() 返回下一个可用的存档成员作为 TarInfo 实例；如果没有可用的，则返回 **None**。 |
| ^(a) 进一步描述，请参阅 [CVE-2007-4559](https://oreil.ly/7hJ89)。 |

### The TarInfo class

The methods getmember and getmembers of TarFile instances return instances of TarInfo, supplying information about members of the archive. You can also build a TarInfo instance with a TarFile instance’s method gettarinfo. The *name* argument may be a path-like object. The most useful attributes and methods supplied by a TarInfo instance *t* are listed in Table 11-11.

Table 11-11\. TarInfo 实例 *t* 的有用属性

| isdir() | 如果文件是目录，则返回 **True** |
| --- | --- |
| isfile() | 如果文件是常规文件，则返回 **True** |
| issym() | 如果文件是符号链接，则返回 **True** |
| linkname | 当 *t*.type 是 LNKTYPE 或 SYMTYPE 时，目标文件的名称（一个字符串） |
| mode | *t* 标识的文件的权限和其他模式位 |
| mtime | 最后修改时间，由 *t* 标识的文件 |
| name | *t* 标识的存档中文件的名称 |
| size | *t* 标识的文件的大小（未压缩，以字节为单位） |
| type | 文件类型之一，是 tarfile 模块的属性常量之一（符号链接的 SYMTYPE，常规文件的 REGTYPE，目录的 DIRTYPE 等等；请参阅 [在线文档](https://oreil.ly/RwYNA) 获取完整列表） |

## The zipfile Module

The zipfile module can read and write ZIP files (i.e., archive files compatible with those handled by popular compression programs such as zip and unzip, pkzip and pkunzip, WinZip, and so on, typically named with a *.zip* extension). **python -m zipfile** 提供了一个有用的命令行界面，用于访问模块的功能：运行它而不带其他参数，以获取简短的帮助消息。

有关 ZIP 文件的详细信息可以在[PKWARE](https://oreil.ly/fVfmV)和[Info-ZIP](https://oreil.ly/rMHiL)网站上找到。您需要研究这些详细信息以使用 zipfile 执行高级 ZIP 文件处理。如果您不需要与使用 ZIP 文件标准的其他程序进行互操作，则通常最好使用 lzma、gzip 和 bz2 模块来处理压缩，tarfile 则是创建（可选压缩的）存档的更好方法。

zipfile 模块不能处理多磁盘 ZIP 文件，也不能创建加密存档（它可以解密，尽管速度相对较慢）。该模块还不能处理使用除通常的*stored*（未压缩复制到存档的文件）和*deflated*（使用 ZIP 格式的默认算法压缩的文件）之外的压缩类型的存档成员。zipfile 还处理 bzip2 和 LZMA 压缩类型，但请注意：并非所有工具都能处理这些类型，因此如果您使用它们，则牺牲了一些可移植性以获得更好的压缩。

zipfile 模块提供 is_zipfile 函数和类 Path，如表 11-12 中列出。此外，它还提供了稍后描述的类 ZipFile 和 ZipInfo。与无效 ZIP 文件相关的错误，zipfile 的函数会引发 zipfile.error 异常的实例。

表 11-12\. zipfile 模块的辅助函数和类

| is_zipfile | is_zipfile(*file*) 当由字符串、路径样式对象或类似文件对象*file*命名的文件在文件的前几个和最后几个字节中看起来是有效的 ZIP 文件时返回**True**；否则返回**False**。 |
| --- | --- |
| Path | **class** Path(*root*, at='') 3.8+ 用于 ZIP 文件的 pathlib 兼容包装器。从*root*，一个 ZIP 文件（可以是 ZipFile 实例或适合传递给 ZipFile 构造函数的文件）返回一个 pathlib.Path 对象*p*。字符串参数 at 是指定*p*在 ZIP 文件中位置的路径：默认是根。*p*公开了几个 pathlib.Path 方法：详细信息请参阅[在线文档](https://oreil.ly/sMeC_)。 |

### ZipFile 类

zipfile 提供的主要类是 ZipFile。其构造函数具有以下签名：

| ZipFile | **class** ZipFile(*file*, mode='r', compression=zipfile.ZIP_STORED, allowZip64=**True**, compresslevel=**None**, *, strict_timestamps=**True**) 打开名为*file*（字符串、类似文件对象或路径样式对象）的 ZIP 文件。mode 可以是'r'以读取现有的 ZIP 文件，'w'以写入新的 ZIP 文件或截断并重写现有文件，或者'a'以追加到现有文件。也可以是'x'，类似于'w'，但如果 ZIP 文件已存在则引发异常——这里的'x'表示“排他”。

当模式为 'a' 时，*file* 可以命名为现有的 ZIP 文件（在这种情况下，新成员将添加到现有存档中）或现有的非-ZIP 文件。在后一种情况下，将创建一个新的类似 ZIP 文件的存档，并将其附加到现有文件上。后一种情况的主要目的是让您构建一个在运行时自行解压缩的可执行文件。然后，现有文件必须是自解压缩可执行文件前缀的原始副本，由 *www.info-zip.org* 和其他 ZIP 文件压缩工具供应商提供。

压缩是写入存档时要使用的 ZIP 压缩方法：ZIP_STORED（默认值）请求存档不使用压缩，ZIP_DEFLATED 请求存档使用压缩的 *deflation* 模式（ZIP 文件中使用的最常见和有效的压缩方法）。它还可以是 ZIP_BZIP2 或 ZIP_LZMA（牺牲可移植性以获得更多压缩；这些需要分别使用 bz2 或 lzma 模块）。未被识别的值将引发 NotImplementedError。|

| ZipFile *(cont.)* | 当 allowZip64 为 **True**（默认值）时，允许 ZipFile 实例使用 ZIP64 扩展来生成大于 4 GB 的存档；否则，任何尝试生成这样一个大存档的尝试都会引发 LargeZipFile 异常。compresslevel 是一个整数（在使用 ZIP_STORED 或 ZIP_LZMA 时被忽略），从 0 表示 ZIP_DEFLATED（1 表示 ZIP_BZIP2），它请求适度的压缩但操作速度较快，到 9 请求最佳压缩以换取更多计算。

将 strict_timestamps 设置为 **False** 可以存储早于 1980-01-01（将时间戳设置为 1980-01-01）或晚于 2107-12-31（将时间戳设置为 2107-12-31）的文件。|

ZipFile 是一个上下文管理器；因此，您可以在 **with** 语句中使用它，以确保在完成后关闭底层文件。例如：

```py
`with` zipfile.ZipFile('archive.zip') `as` z:    
    data = z.read('data.txt')
```

除了实例化时提供的参数之外，ZipFile 实例 *z* 还具有属性 fp 和 filename，它们是 *z* 工作的类似文件的对象和其文件名（如果已知）；comment，可能是空字符串的存档评论；和 filelist，存档中的 ZipInfo 实例列表。此外，*z* 还有一个名为 debug 的可写属性，一个从 0 到 3 的 int，您可以分配它来控制在输出到 sys.stdout 时要发出多少调试输出：² 从 *z*.debug 为 0 时什么都不输出，到 *z*.debug 为 3 时发出的调试输出为可用的最大量。

ZipFile 实例 *z* 提供了 Table 11-13 中列出的方法。

表 11-13\. ZipFile 实例 *z* 提供的方法

| close | close() 关闭存档文件 *z*。确保调用 *z*.close()，否则可能会在磁盘上留下不完整且无法使用的 ZIP 文件。这种强制性的最终化通常最好使用 “try/finally” 中介绍的 **try**/**finally** 语句执行，或者更好地使用 “The with Statement and Context Managers” 中介绍的 **with** 语句执行。 |
| --- | --- |

| extract | extract(*member*, path=**None**, pwd=**None**) 将存档成员提取到磁盘，到目录或类似路径的对象路径，或者默认情况下提取到当前工作目录；*member* 是成员的完整名称，或者标识成员的 ZipInfo 实例。extract 会在 *member* 中规范化路径信息，将绝对路径转换为相对路径，移除任何 .. 组件，并且在 Windows 上，将文件名中非法的字符转换为下划线 (_)。pwd（如果存在）是用于解密加密成员的密码。 |

extract 返回它创建的文件（如果已存在则覆盖），或者返回它创建的目录（如果已存在则不变）。在已关闭的 ZipFile 上调用 extract 会引发 ValueError。 |

| extractall | extractall(path=**None**, members=**None**, pwd=**None**) 将存档成员提取到磁盘（默认情况下为全部成员），到目录或类似路径的对象路径，或者默认情况下提取到当前工作目录；members 可选地限制要提取的成员，并且必须是由 *z*.namelist 返回的字符串列表的子集。extractall 会在提取的成员中规范化路径信息，将绝对路径转换为相对路径，移除任何 .. 组件，并且在 Windows 上，将文件名中非法的字符转换为下划线 (_)。pwd（如果存在）是用于解密加密成员（如果有）的密码。 |
| --- | --- |
| getinfo | getinfo(*name*) 返回一个 ZipInfo 实例，该实例提供有关由字符串 *name* 指定的存档成员的信息。 |
| infolist | infolist() 返回一个 ZipInfo 实例列表，其中包含存档 *z* 中的每个成员的信息，顺序与存档中的条目相同。 |
| namelist | namelist() 返回一个字符串列表，其中包含存档 *z* 中每个成员的名称，顺序与存档中的条目相同。 |
| open | open(*name*, mode='r', pwd=**None**, *, force_zip64=**False**) 提取并返回由 *name*（成员名称字符串或 ZipInfo 实例）标识的存档成员作为（可能是只读的）类文件对象。*mode* 可以是 'r' 或 'w'。pwd（如果存在）是用于解密加密成员的密码。在可能的情况下，当未知文件大小可能超过 2 GiB 时，请传递 force_zip64=**True**，以确保标题格式能够支持大文件。当您预先知道大文件大小时，请使用适当设置文件大小的 ZipInfo 实例进行 *name* 的操作。 |
| printdir | printdir() 将存档 *z* 的文本目录输出到 sys.stdout。 |
| read | read(*name*, *pwd*) 提取由 *name*（成员名称字符串或 ZipInfo 实例）标识的归档成员并返回其内容的字节串（如果在关闭的 ZipFile 上调用则引发 ValueError）。*pwd*（如果存在）是用于解密加密成员的密码。 |
| setpassword | setpassword(*pwd*) 将字符串 *pwd* 设置为解密加密文件时使用的默认密码。 |
| testzip | testzip() 读取并检查归档 *z* 中的文件。返回损坏的第一个归档成员的名称的字符串，如果归档完好则返回 **None**。 |
| write | write(*filename*, arcname=**None**, compress_type=**None**, compresslevel=**None**) 将由字符串 *filename* 指定的文件写入归档 *z*，并使用 arcname 作为归档成员名称。当 arcname 为 **None** 时，write 使用 *filename* 作为归档成员名称。当 compress_type 或 compresslevel 为 **None**（默认值）时，write 使用 *z* 的压缩类型和级别；否则，compress_type 和/或 compresslevel 指定如何压缩文件。*z* 必须以 'w'、'x' 或 'a' 模式打开；否则将引发 ValueError。 |

| writestr | writestr(*zinfo_arc*, *data,* compress_type=**None***,* compresslevel=**None**)

使用指定的元数据 *zinfo_arc* 和 *data* 中的数据向归档 *z* 添加成员。*zinfo_arc* 必须是指定至少 *filename* 和 *date_time* 的 ZipInfo 实例，或者是用作归档成员名称的字符串，日期和时间设置为当前时刻。*data* 是 bytes 或 str 的实例。当 compress_type 或 compresslevel 为 **None**（默认值）时，writestr 使用 *z* 的压缩类型和级别；否则，compress_type 和/或 compresslevel 指定如何压缩文件。*z* 必须以 'w'、'x' 或 'a' 模式打开；否则将引发 ValueError。

当你有内存中的数据并需要将数据写入到 ZIP 文件归档 *z* 中时，使用 *z*.writestr 比 *z*.write 更简单更快。后者需要你首先将数据写入磁盘，然后再删除无用的磁盘文件；而前者可以直接编码：

```py
`import` zipfile
`with` zipfile.ZipFile('z.zip', 'w') `as` zz:
    data = 'four score\nand seven\nyears ago\n'
    zz.writestr('saying.txt', data)
```

这里是如何打印由上一个示例创建的 ZIP 文件归档中包含的所有文件的列表，以及每个文件的名称和内容：

```py
`with` zipfile.ZipFile('z.zip') `as` zz:
    zz.printdir()
    `for` name `in` zz.namelist():
        print(f'{name}: {zz.read(name)!r}')
```

|

### ZipInfo 类

ZipFile 实例的 getinfo 和 infolist 方法返回 ZipInfo 类的实例，用于提供有关归档成员的信息。Table 11-14 列出了 ZipInfo 实例 *z* 提供的最有用的属性。

Table 11-14\. ZipInfo 实例 z 的有用属性

| comment | 归档成员的评论字符串 |
| --- | --- |
| compress_size | 归档成员压缩数据的字节大小 |
| compress_type | 记录归档成员压缩类型的整数代码 |
| date_time | 一个由六个整数组成的元组，表示文件的最后修改时间：年（>=1980）、月、日（1+）、小时、分钟、秒（0+） |
| file_size | 存档成员的未压缩数据大小，以字节为单位 |
| filename | 存档中文件的名称 |

# os 模块

os 是一个综合模块，提供了对各种操作系统能力的几乎统一的跨平台视图。它提供了低级方法来创建和处理文件和目录，以及创建、管理和销毁进程。本节介绍了 os 的与文件系统相关的函数；“使用 os 模块运行其他程序” 则涵盖了与进程相关的函数。大多数情况下，你可以使用更高级的抽象级别的其他模块来提高生产力，但理解底层 os 模块中的“底层”内容仍然非常有用（因此我们进行了覆盖）。

os 模块提供了一个 name 属性，这是一个字符串，用于标识 Python 运行的平台类型。name 的常见值包括 'posix'（各种 Unix 类平台，包括 Linux 和 macOS）和 'nt'（各种 Windows 平台）；'java' 是老旧但仍然想念的 Jython。你可以通过 os 提供的函数利用某个平台的一些独特功能。然而，本书关注的是跨平台编程，而不是特定于平台的功能，因此我们不涵盖仅存在于一个平台上的 os 部分，也不涵盖特定于平台的模块：本书涵盖的功能至少在 'posix' 和 'nt' 平台上可用。然而，我们确实涵盖了在各个平台上提供给定功能的方式之间的一些差异。

## 文件系统操作

使用 os 模块，你可以以多种方式操作文件系统：创建、复制和删除文件和目录；比较文件；以及检查关于文件和目录的文件系统信息。本节记录了你用于这些目的的 os 模块的属性和方法，并涵盖了一些操作文件系统的相关模块。

### os 模块的路径字符串属性

文件或目录由一个字符串标识，称为其*路径*，其语法取决于平台。在类 Unix 和 Windows 平台上，Python 接受 Unix 路径的语法，斜线 (/) 作为目录分隔符。在非 Unix 类平台上，Python 还接受特定于平台的路径语法。特别是在 Windows 上，你可以使用反斜杠 (\) 作为分隔符。然而，在字符串文字中，你需要将每个反斜杠双写为 \\，或者使用原始字符串文字语法（如 “字符串” 中所述）；但这会使程序失去可移植性。Unix 路径语法更加方便，可以在任何地方使用，因此我们强烈建议*始终*使用它。在本章的其余部分，我们在解释和示例中都使用 Unix 路径语法。

`os` 模块提供了关于当前平台路径字符串的属性，详见 Table 11-15。通常应使用高级别的路径操作（见 “The os.path Module”³），而非基于这些属性的低级别字符串操作。然而，在某些情况下，这些属性可能会很有用。

Table 11-15\. `os` 模块提供的属性

| curdir | 表示当前目录的字符串（在 Unix 和 Windows 上为 '.'） |
| --- | --- |
| defpath | 程序的默认搜索路径，如果环境缺少 PATH 环境变量则使用 |
| extsep | 文件名扩展部分与其余部分之间的分隔符（在 Unix 和 Windows 上为 '.'） |
| linesep | 终止文本行的字符串（在 Unix 上为 '\n'；在 Windows 上为 '\r\n'） |
| pardir | 表示父目录的字符串（在 Unix 和 Windows 上为 '..'） |
| pathsep | 字符串列表中路径之间的分隔符，比如用于环境变量 PATH 的（在 Unix 上为 ':'；在 Windows 上为 ';'） |
| sep | 路径组件的分隔符（在 Unix 上为 '/'；在 Windows 上为 '\\'） |

### 权限

类 Unix 平台为每个文件或目录关联九个位：三个位为文件的所有者、其组和其他人（即“others”或“the world”），指示该文件或目录能否被给定主体读取、写入和执行。这九个位称为文件的 *权限位*，是文件 *模式* 的一部分（一个包括描述文件的其他位的位字符串）。通常以八进制表示这些位，每个数字表示三个位。例如，模式 0o664 表示一个文件，其所有者和组可以读取和写入，任何其他人只能读取，不能写入。在 Unix 类似系统上，当任何进程创建文件或目录时，操作系统将应用称为进程的 *umask* 的位掩码，可以移除一些权限位。

非 Unix 类似平台以非常不同的方式处理文件和目录权限。然而，处理文件权限的 `os` 函数接受一个根据前述 Unix 类似方法的 *模式* 参数。每个平台将这九个权限位映射到适合其的方式。例如，在 Windows 上，它只区分只读和读写文件，并且不记录文件所有权，文件的权限位显示为 0o666（读/写）或 0o444（只读）。在这样的平台上，创建文件时，实现只关注位 0o200，当该位为 1 时文件为读/写，为 0 时为只读。

### `os` 模块的文件和目录函数

os 模块提供了几个函数（在 Table 11-16 中列出）来查询和设置文件和目录状态。在所有版本和平台上，这些函数的参数 *path* 都可以是给定所涉及文件或目录路径的字符串，也可以是路径类对象（特别是 pathlib.Path 的实例，在本章后面介绍）。在一些 Unix 平台上还有一些特殊性：

+   一些函数还支持 *文件描述符*（*fd*）——一个整数，表示例如由 os.open 返回的文件作为 *path* 参数。模块属性 os.supports_fd 是支持此行为的 os 模块中的函数集合（在不支持此类功能的平台上，该模块属性将缺失）。

+   一些函数支持可选的仅限关键字参数 follow_symlinks，默认为 **True**。当此参数为 **True** 时，如果 *path* 指示一个符号链接，则函数跟随它以达到实际的文件或目录；当此参数为 **False** 时，函数在符号链接本身上操作。模块属性 os.supports_follow_symlinks（如果存在）是支持此参数的 os 模块中的函数集合。

+   一些函数支持可选的仅限命名参数 dir_fd，默认为 None。当 dir_fd 存在时，*path*（如果是相对路径）被视为相对于在该文件描述符上打开的目录；当缺少 dir_fd 时，*path*（如果是相对路径）被视为相对于当前工作目录。如果 *path* 是绝对路径，则忽略 dir_fd。模块属性 os.supports_dir_fd（如果存在）是支持该参数的 os 模块函数集合。

此外，某些平台上的命名参数 effective_ids，默认为 **False**，允许您选择使用有效的而不是真实的用户和组标识符。通过 os.supports_effective_ids 检查它在您的平台上是否可用。

Table 11-16\. os 模块函数

| access | access(*path*, *mode*, *, dir_fd=**None**, effective_ids=**False**, follow_symlinks=**True**)

当文件或类似路径对象 *path* 具有整数 *mode* 编码的所有权限时返回 **True**；否则返回 **False**。*mode* 可以是 os.F_OK 以测试文件存在性，或者是 os.R_OK、os.W_OK 和 os.X_OK 中的一个或多个（如果有多个，则使用按位或运算符 &#124; 连接）以测试读取、写入和执行文件的权限。如果 dir_fd 不是 **None**，则 access 在相对于提供的目录上操作 *path*（如果 *path* 是绝对路径，则忽略 dir_fd）。传递关键字参数 effective_ids=**True**（默认为 **False**）以使用有效的而不是真实的用户和组标识符（这在所有平台上可能不起作用）。如果传递 follow_symlinks=**False** 并且 *path* 的最后一个元素是符号链接，则 access 在符号链接本身上操作，而不是在链接指向的文件上操作。

access 不使用其 *mode* 参数的标准解释，详细内容请参阅前一节。相反，access 仅测试此特定进程的真实用户和组标识符对文件的请求权限。如果需要更详细地研究文件的权限位，请参阅后面在此表中介绍的 stat 函数。

不要使用 access 来检查用户在打开文件之前是否被授权打开文件；这可能存在安全漏洞。 |

| chdir | chdir(*path*) 将进程的当前工作目录设置为 *path*，*path* 可以是文件描述符或类似路径的对象。 |
| --- | --- |

| chmod, lchmod | chmod(*path*, *mode, *,* dir_fd=**None**, follow_symlinks=**True**) lchmod(*path*, *mode*)

更改文件（或文件描述符或类似路径的对象）*path* 的权限，权限由整数 *mode* 编码。 *mode* 可以是 os.R_OK、os.W_OK 和 os.X_OK 的任意组合（如果有多个则用按位或运算符 &#124; 连接）来表示读、写和执行权限。在类 Unix 平台上，*mode* 可以是更复杂的位模式（如前一节所述），用于指定用户、组和其他对象的不同权限，还可以定义模块 stat 中的其他特殊且不常用的位。具体详细信息请参阅 [在线文档](https://oreil.ly/Ue-aa)。传递 follow_symlinks=**False**（或使用 lchmod）来更改符号链接的权限，而不是该链接的目标文件。 |

| DirEntry | 类 DirEntry 的实例 *d* 提供属性 *name* 和 *path*，分别保存条目的基本名称和完整路径，以及几种方法，其中最常用的是 is_dir、is_file 和 is_symlink。is_dir 和 is_file 默认会跟随符号链接：传递 follow_symlinks=**False** 可以避免此行为。*d* 在尽可能的情况下避免系统调用，并在需要时缓存结果。如果需要确保获取的信息是最新的，可以调用 os.stat(*d*.path) 并使用其返回的 stat_result 实例；不过这可能会牺牲 scandir 的性能提升。有关更详细的信息，请参阅 [在线文档](https://oreil.ly/4ZsbW)。 |
| --- | --- |

| getcwd, getcwdb | getcwd(), getcwdb()

getcwd 返回一个 str，表示当前工作目录的路径。getcwdb 返回一个 bytes 字符串（在 Windows 上使用 UTF-8 编码，3.8+ 版本）。 |

| link | link(*src*, *dst*, ***, src_dir_fd=**None**, dst_dir_fd=**None**, follow_symlinks=**True**)

创建名为 *dst* 的硬链接，指向 *src*。 *src* 和 *dst* 都可以是类似路径的对象。设置 src_dir_fd 和/或 dst_dir_fd 以便链接操作使用相对路径，并传递 follow_symlinks=**False** 只在符号链接上操作，而不是该链接的目标文件。要创建符号（“软”）链接，请使用稍后在此表中介绍的 symlink 函数。 |

| listdir | listdir(path='.') 返回列表，其中的项是目录中所有文件和子目录的名称，文件描述符（指向目录）或类路径对象 path。列表顺序任意，并且*不*包括特殊目录名称 '.'（当前目录）和 '..'（父目录）。当 path 类型为 bytes 时，返回的文件名也为 bytes 类型；否则为 str 类型。请参阅表中稍后的替代函数 scandir，在某些情况下可提供性能改进。在调用此函数期间，请勿从目录中删除或添加文件：这可能会产生意外的结果。 |
| --- | --- |

| mkdir, makedirs | mkdir(*path*, mode=0777, dir_fd=**None**), makedirs(*path*, mode=0777, exist_ok=**False**)

mkdir 仅创建 *path* 中最右侧的目录，如果 *path* 中的前面任何目录不存在，则引发 OSError。mkdir 可接受 dir_fd 以相对于文件描述符的方式访问路径。makedirs 创建 *path* 中尚不存在的所有目录（传递 exist_ok=**True** 可避免引发 FileExistsError）。

两个函数都使用 mode 作为它们创建的目录的权限位，但某些平台和某些新创建的中间级目录可能会忽略 mode；使用 chmod 明确设置权限。

| remove, unlink | remove(*path*, ***, dir_fd*=***None**), unlink(*path*, ***, dir_fd=**None**)

删除文件或类路径对象 *path*（相对于 dir_fd）。请参阅表中稍后的 rmdir 来删除目录而不是文件。unlink 是 remove 的同义词。

| removedirs | removedirs(*path*) 从右到左遍历 *path* 的目录部分（可能是类路径对象），依次移除每个目录。循环在遇到异常（通常是因为目录不为空）时结束。只要 removedirs 至少删除了一个目录，就不会传播异常。 |
| --- | --- |

| rename, renames | rename(src, dst, ***, src_dir_fd=**None**, dst_dir_fd=**None**), renames(*src*, *dst, /*)

renames（“移动”）文件、类路径对象或名为 src 的目录到 dst。如果 dst 已存在，rename 可能会替换 dst 或引发异常；要保证替换，请调用 os.replace 函数。要使用相对路径，传递 src_dir_fd 和/或 dst_dir_fd。

renames 功能与 rename 类似，但它会为 *dst* 创建所有必要的中间目录。重命名后，renames 使用 removedirs 从路径 *src* 中移除空目录。如果重命名未清空 *src* 的起始目录，不会传播任何导致的异常；这并不算是错误。renames 无法接受相对路径参数。

| rmdir | rmdir(*path*, ***, dir_fd=**None**) 会删除名为 *path*（可能相对于 dir_fd）的空目录或类路径对象。如果删除失败，特别是如果目录不为空，则会引发 OSError。 |
| --- | --- |
| scandir | scandir(path='.') 返回一个迭代器，针对路径中的每个项目产生一个 os.DirEntry 实例，该路径可以是字符串、类似路径的对象或文件描述符。 使用 scandir 并调用每个结果项的方法以确定其特征可以提供性能改进，相比于使用 listdir 和 stat，这取决于底层平台。 scandir 可以用作上下文管理器：例如，**with** os.scandir(*path*) **as** *itr*: 在完成时确保关闭迭代器（释放资源）。 |

| stat, lstat,

fstat | stat(path, ***, dir_fd=**None**, follow_symlinks=**True**), lstat(path, ***, dir_fd*=***None**),

fstat(fd)

stat 返回类型为 stat_result 的值 *x*，它提供关于路径的（至少）10 个信息项。 path 可以是文件、文件描述符（在这种情况下，您可以使用 stat(fd) 或 fstat，它仅接受文件描述符）、类似路径的对象或子目录。 *path* 可以是 dir_fd 的相对路径，也可以是符号链接（如果 follow_symlinks=**False**，或者使用 lstat；在 Windows 上，除非 follow_symlinks=**False**，否则会遵循操作系统可以解析的所有[重解析点](https://oreil.ly/AvIiq)）。 stat_result 值是一个包含值元组，也支持对其包含值的每个命名访问（类似于 collections.namedtuple，尽管未实现为此类）。 访问 stat_result 的项目通过其数字索引是可能的，但不建议，因为结果代码不可读； 相反，请使用相应的属性名称。 表 11-17 列出了 stat_result 实例的主要 10 个属性以及相应项的含义。

表 11-17\. stat_result 实例的项（属性）

&#124; 项目索引 &#124; 属性名称 &#124; 含义 &#124;

&#124; --- &#124; --- &#124; --- &#124;

&#124; 0 &#124; st_mode &#124; 保护和其他模式位 &#124;

&#124; 1 &#124; st_ino &#124; inode 号码 &#124;

&#124; 2 &#124; st_dev &#124; 设备 ID &#124;

&#124; 3 &#124; st_nlink &#124; 硬链接数量 &#124;

&#124; 4 &#124; st_uid &#124; 所有者的用户 ID &#124;

&#124; 5 &#124; st_gid &#124; 所有者的组 ID &#124;

&#124; 6 &#124; st_size &#124; 大小（以字节为单位） &#124;

&#124; 7 &#124; st_atime &#124; 最后访问时间 &#124;

&#124; 8 &#124; st_mtime &#124; 最后修改时间 &#124;

&#124; 9 &#124; st_ctime &#124; 最后状态更改时间 &#124;

例如，要打印文件 *path* 的大小（以字节为单位），您可以使用以下任何一种：

```py
`import` os
print(os.stat(path)[6])       *`# works but unclear`*
print(os.stat(path).st_size)  *`# easier to understand`*
print(os.path.getsize(path))  *`# convenience function`*
                            *`# that wraps stat`*

```

时间值以自纪元以来的秒数表示，如 第十三章 中所述（在大多数平台上为整数）。 无法为项目提供有意义值的平台使用虚拟值。 对于 stat_result 实例的其他、特定于平台的属性，请参阅 [在线文档](https://oreil.ly/o7wOH)。

| symlink | symlink(*target*, *symlink_path*, target_is_directory=**False**, *, dir_fd=**None**) 创建名为*symlink_path*的符号链接，指向文件、目录或路径对象*target*。*target*可以相对于 dir_fd。target_is_directory 仅在 Windows 系统上使用，用于指定创建的符号链接应表示文件还是目录；在非 Windows 系统上，此参数将被忽略。（在 Windows 上运行 os.symlink 通常需要提升的权限。） |
| --- | --- |
| utime | utime(*path*, times=**None**, ***, [*ns*, ]dir_fd=**None**, follow_symlinks=**True**) 设置文件、目录或路径对象*path*的访问时间和修改时间。*path*可以相对于 dir_fd，并且如果 follow_symlinks=**False**，*path*可以是符号链接。如果 times 为**None**，utime 使用当前时间。否则，times 必须是一对数字（自纪元以来的秒数，详见第十三章），顺序为（*accessed*，*modified*）。要指定纳秒，请将*ns*传递为（*acc_ns*，*mod_ns*），其中每个成员都是表示自纪元以来的纳秒数的整数。不要同时指定 times 和*ns*。 |

| walk, fwalk | walk(*top*, topdown=**True**, onerror=**None**, followlinks=**False**), fwalk(top='.', topdown=**True**, onerror=**None**, *, follow_symlinks=**False**, dir_fd=**None**)

walk 是一个生成器，为树的每个目录生成一个条目，树的根是目录或路径对象*top*。当 topdown 为**True**（默认）时，walk 从树的根向下访问目录；当 topdown 为**False**时，walk 从树的叶子向上访问目录。默认情况下，walk 捕获并忽略树遍历过程中引发的任何 OSError 异常；将 onerror 设置为可调用对象，以捕获树遍历过程中引发的任何 OSError 异常，并将其作为唯一参数传递给 onerror，可对其进行处理、忽略或**引发**以终止树遍历并传播异常（文件名可作为异常对象的 filename 属性访问）。

walk 每生成的条目是一个包含三个子项的元组：*dirpath*，是目录的路径字符串；*dirnames*，是该目录的直接子目录名称列表（特殊目录'.'和'..' *不* 包括在内）；*filenames*，是该目录中直接的文件名称列表。如果 topdown 为**True**，您可以直接修改*dirnames*列表，删除一些项目和/或重新排序其他项目，以影响从*dirpath*开始的子树遍历；walk 仅迭代剩余在*dirnames*中的子目录，按照它们留下的顺序。如果 topdown 为**False**，这些修改将不会生效（在这种情况下，walk 在访问当前目录和生成其条目时已经访问了所有子目录）。 |

| walk, fwalk

*(续)* | 默认情况下，walk 不会遍历解析为目录的符号链接。要获得这样的额外遍历，请传递 followlinks=**True**，但请注意：如果符号链接解析为其祖先的目录，这可能会导致无限循环。walk 对此异常没有采取预防措施。

# followlinks 与 follow_symlinks

注意，在所有情况下，对于 os.walk **仅**，名为 follow_symlinks 的参数实际上被命名为 followlinks。

fwalk（仅 Unix）类似于 walk，但 top 可能是文件描述符 dir_fd 的相对路径，并且 fwalk 产生四元组：前三个成员（dirpath、dirnames 和 filenames）与 walk 的生成值相同，第四个成员是 dirfd，即 dirpath 的文件描述符。请注意，无论是 walk 还是 fwalk，默认情况下 **不** 跟随符号链接。|

### 文件描述符操作

除了前面介绍的许多函数外，os 模块还提供了几个专门与文件描述符一起使用的函数。*文件描述符* 是操作系统用作不透明句柄以引用打开文件的整数。尽管通常最好使用 Python 文件对象（在 “The io Module” 中介绍），有时使用文件描述符可以让您执行某些操作更快，或者（可能牺牲可移植性）以不直接可用于 io.open 的方式执行操作。文件对象和文件描述符不能互换。

要获取 Python 文件对象 *f* 的文件描述符 *n*，请调用 *n* = *f*.fileno()。要使用现有打开的文件描述符 *fd* 创建新的 Python 文件对象 *f*，请使用 *f* = os.fdopen(*fd*)，或者将 *fd* 作为 io.open 的第一个参数传递。在类 Unix 和 Windows 平台上，某些文件描述符在进程启动时预分配：0 是进程的标准输入的文件描述符，1 是进程的标准输出的文件描述符，2 是进程的标准错误的文件描述符。调用诸如 dup 或 close 等 os 模块方法来操作这些预分配的文件描述符，对于重定向或操作标准输入和输出流可能会很有用。

os 模块提供了许多处理文件描述符的函数；其中一些最有用的列在 Table 11-18 中。

表 11-18\. 处理文件描述符的有用 os 模块函数

| close | close(*fd*) 关闭文件描述符 *fd*。 |
| --- | --- |
| closerange | closerange(*fd_low*, *fd_high*) 关闭从 *fd_low*（包括 *fd_low*）到 *fd_high*（不包括 *fd_high*）的所有文件描述符，忽略可能发生的任何错误。 |
| dup | dup(*fd*) 返回复制文件描述符 *fd* 的文件描述符。 |
| dup2 | dup2(*fd*, *fd2*) 复制文件描述符 *fd* 到文件描述符 *fd2*。当文件描述符 *fd2* 已经打开时，dup2 首先关闭 *fd2*。 |
| fdopen | fdopen(*fd*, **a*, ***k*) 类似于 io.open，不同之处在于 *fd* **必须** 是打开文件描述符的整数。 |
| fstat | fstat(*fd*) 返回一个 stat_result 实例 *x*，其中包含有关打开在文件描述符 *fd* 上的文件的信息。表 11-17 涵盖了 *x* 的内容。 |
| lseek | lseek(*fd*, *pos*, *how*) 将文件描述符 *fd* 的当前位置设置为有符号整数字节偏移量 *pos*，并返回从文件开头的结果字节偏移量。*how* 表示参考点（点 0）。当 *how* 是 os.SEEK_SET 时，*pos* 为 0 表示文件开头；对于 os.SEEK_CUR，它表示当前位置；对于 os.SEEK_END，它表示文件末尾。例如，lseek(*fd*, 0, os.SEEK_CUR) 返回从文件开头到当前位置的当前位置的字节偏移量，而不影响当前位置。普通磁盘文件支持寻址；对不支持寻址的文件（例如，向终端输出的文件）调用 lseek 会引发异常。 |

| open | open(*file*, *flags*, mode=0o777) 返回一个文件描述符，打开或创建名为 *file* 的文件。当 open 创建文件时，使用 mode 作为文件的权限位。*flags* 是一个整数，通常是 os 模块以下一个或多个属性的按位 OR 运算（使用操作符 &#124; ）：

O_APPEND

追加任何新数据到 *file* 当前的内容

O_BINARY

在 Windows 平台上以二进制而非文本模式打开 *file*（在类 Unix 平台上引发异常）

O_CREAT

如果 *file* 不存在，则创建 *file*

O_DSYNC, O_RSYNC, O_SYNC, O_NOCTTY

根据平台支持的情况设置同步模式

O_EXCL

如果 *file* 已经存在，则抛出异常

O_NDELAY, O_NONBLOCK

如果平台支持，以非阻塞模式打开 *file*

O_RDONLY, O_WRONLY, O_RDWR

分别打开 *file* 以进行只读、只写或读/写访问（互斥：这些属性中必须有且仅有一个 *flags*）

O_TRUNC

丢弃 *file* 的先前内容（与 O_RDONLY 不兼容）

|

| pipe | pipe() 创建一个管道并返回一对文件描述符（*r_fd*、*w_fd*），分别用于读取和写入。 |
| --- | --- |
| read | read(*fd, n*) 从文件描述符 *fd* 读取最多 *n* 字节，并将它们作为字节串返回。当只有 *m* 个字节当前可供从文件读取时，读取并返回 *m < n* 字节。特别地，在没有更多字节当前可用于读取时，通常因为文件已经结束，返回空字符串。 |
| write | write(*fd*, *s*) 将字节串 *s* 中的所有字节写入文件描述符 *fd* 并返回写入的字节数。 |

## os.path 模块

os.path 模块提供了用于分析和转换路径字符串和类似路径对象的函数。该模块中最常用的有用函数列在 表 11-19 中。

表 11-19\. os.path 模块的常用函数

| abspath | abspath(*path*) 返回与 *path* 等效的标准化绝对路径字符串，就像（在 *path* 是当前目录中文件名的情况下）：

os.path.normpath(os.path.join(os.getcwd(), *path*))

例如，os.path.abspath(os.curdir) 与 os.getcwd() 相同。

| basename | basename(*path*) 返回 *path* 的基本名称部分，就像 os.path.split(*path*)[1]。例如，os.path.basename('b/c/d.e') 返回 'd.e'。 |
| --- | --- |
| commonpath | commonpath(*list*) 接受字符串或类似路径对象的序列，并返回最长公共子路径。与 commonprefix 不同，只返回有效路径；如果 *list* 为空、包含绝对和相对路径的混合或包含不同驱动器上的路径，则引发 ValueError。 |
| com⁠m⁠o⁠n​p⁠r⁠e⁠fix | commonprefix(*list*) 接受字符串或类似路径对象的列表，并返回列表中所有项目的最长公共前缀字符串，如果 *list* 为空则返回 '.'。例如，os.path.commonprefix(['foobar', 'foolish']) 返回 'foo'。可能返回无效路径；如果要避免此问题，请参阅 commonpath。 |
| dirname | dirname(*path*) 返回 *path* 的目录部分，就像 os.path.split(*path*)[0]。例如，os.path.dirname('b/c/d.e') 返回 'b/c'。 |
| exists, lexists | exists(*path*), lexists(*path*) 当 *path* 是现有文件或目录的名称时，exists 返回 **True**（*path* 也可以是打开的文件描述符或类似路径对象）；否则返回 **False**。换句话说，os.path.exists(*x*) 与 os.access(*x*, os.F_OK) 相同。lexists 也是如此，但在 *path* 是指示不存在的文件或目录的现有符号链接（有时称为*损坏的符号链接*）时，也返回 **True**，而在这种情况下，exists 返回 **False**。对于包含操作系统级别不可表示的字符或字节的路径，两者均返回 **False**。 |

| expandvars, expanduser | expandvars(*path*), expanduser(*path*) 返回字符串或类似路径对象 *path* 的副本，在其中形如 $*name* 或 ${*name*}（仅在 Windows 上为 *%name%*）的每个子字符串都替换为环境变量 *name* 的值。例如，如果环境变量 HOME 设置为 /u/alex，则以下代码：

```py
`import` os
print(os.path.expandvars('$HOME/foo/'))
```

发射 /u/alex/foo/.

os.path.expanduser 将前导的 ~ 或 ~user（如果有）扩展为当前用户的主目录路径。

| getatime, getctime,

getmtime,

getsize | getatime(*path*), getctime(*path*), getmtime(*path*), getsize(*path*) 每个函数调用 os.stat(*path*) 并从结果中返回一个属性：分别是 st_atime、st_ctime、st_mtime 和 st_size。有关这些属性的更多详细信息，请参阅 Table 11-17。

| isabs | isabs(*path*) 当 *path* 是绝对路径时返回 **True**（路径以斜杠 (/ 或 \) 或在某些非类 Unix 平台（如 Windows）上以驱动器指示符开头，后跟 os.sep）。否则，isabs 返回 **False**。 |
| --- | --- |
| isdir | isdir(*path*) 当 *path* 指定的是现有目录时返回 **True**（isdir 跟随符号链接，因此 isdir 和 islink 可能都返回 **True**）；否则返回 **False**。 |
| isfile | isfile(*path*) 当 *path* 指定的是现有常规文件时返回 **True**（isfile 跟随符号链接，因此 islink 也可能为 **True**）；否则返回 **False**。 |
| islink | islink(*path*) 当 *path* 指定的是符号链接时返回 **True**；否则返回 **False**。 |
| ismount | ismount(*path*) 当 *path* 指定的是 [挂载点](https://oreil.ly/JYbY5) 时返回 **True**；否则返回 **False**。 |

| join | join(*path*, **paths*) 返回一个字符串，它使用当前平台的适当路径分隔符连接参数（字符串或类路径对象）。例如，在 Unix 上，相邻路径组件之间使用一个斜杠字符 / 分隔。如果任何参数是绝对路径，join 将忽略先前的参数。例如：

```py
print(os.path.join('a/b', 'c/d', 'e/f'))
*`# on Unix prints: a/b/c/d/e/f`*
print(os.path.join('a/b', '/c/d', 'e/f'))
*`# on Unix prints:`* *`/c/d/e/f`*
```

第二次调用 os.path.join 忽略了其第一个参数 'a/b'，因为其第二个参数 '/c/d' 是绝对路径。 |

| normcase | normcase(*path*) 返回 *path* 的大小写标准化副本，以适应当前平台。在区分大小写的文件系统（如 Unix 类系统）中，返回 *path* 本身。在不区分大小写的文件系统（如 Windows）中，将字符串转换为小写。在 Windows 上，normcase 还会将每个 / 转换为 \\\\。 |
| --- | --- |
| normpath | normpath(*path*) 返回一个等效于 *path* 的标准化路径名，移除冗余的分隔符和路径导航部分。例如，在 Unix 上，当 *path* 是 'a//b'、'a/./b' 或 'a/c/../b' 之一时，normpath 返回 'a/b'。normpath 使路径分隔符适合当前平台。例如，在 Windows 上，分隔符变成 \\\\。 |
| realpath | realpath(*path*, ***, strict=**False**) 返回指定文件或目录或路径类对象的实际路径，同时解析符号链接。3.10+ 设置 strict=**True** 可以在 *path* 不存在或存在符号链接循环时引发 OSError。 |
| relpath | relpath(*path*, start=os.curdir) 返回到目录 start 的相对于文件或目录 *path*（一个字符串或路径类对象）的路径。 |
| samefile | samefile(*path1*, *path2*) 如果两个参数（字符串或路径类对象）引用同一文件或目录，则返回 **True**。 |
| sameopenfile | sameopenfile(*fd1*, *fd2*) 如果两个参数（文件描述符）引用同一文件或目录，则返回 **True**。 |
| samestat | samestat(*stat1*, *stat2*) 如果两个参数（通常是 os.stat 调用的结果 os.stat_result 实例）引用同一文件或目录，则返回 **True**。 |
| split | split(*path*) 返回一对字符串(*dir*, *base*)，使得 join(*dir*, *base*)等于*path*。*base*是最后的组成部分，永远不包含路径分隔符。当*path*以分隔符结尾时，*base*为空字符串。*dir*是*path*的前导部分，直到最后一个分隔符之前的部分。例如，os.path.split('a/b/c/d') 返回 ('a/b/c', 'd')。 |
| splitdrive | splitdrive(*path*) 返回一对字符串(*drv*, *pth*)，使得*drv*+*pth*等于*path*。*drv*是驱动器规范，或者是''；在没有驱动器规范的平台（如类 Unix 系统）上，它始终为''。在 Windows 上，os.path.splitdrive('c:d/e') 返回 ('c:', 'd/e')。 |
| splitext | splitext(*path*) 返回一对(*root*, *ext*)，使得*root*+*ext*等于*path*。*ext*要么是空字符串，要么以'.'开头且没有其他'.'或路径分隔符。例如，os.path.splitext('a.a/b.c.d') 返回一对 ('a.a/b.c', '.d')。 |

## OSError 异常

当操作系统请求失败时，os 会引发异常，即 OSError 的一个实例。os 还使用 os.error 作为 OSError 的同义词。OSError 的实例具有三个有用的属性，详见 Table 11-20。

表 11-20\. OSError 实例的属性

| errno | 操作系统错误的数值错误代码 |
| --- | --- |
| filename | 操作失败的文件名（仅限文件相关函数） |
| strerror | 简要描述错误的字符串 |

OSError 有多个子类用于指定问题所在，详见“OSError 子类”。

当使用无效的参数类型或值调用 os 函数时，它们还可以引发其他标准异常，如 TypeError 或 ValueError，以便它们甚至不尝试基础操作系统功能。

# errno 模块

errno 模块为操作系统错误代码数值提供了几十个符号名称。使用 errno 根据错误代码有选择地处理可能的系统错误，这将增强程序的可移植性和可读性。然而，使用适当的 OSError 子类进行选择性的异常处理通常比 errno 更好。例如，为了处理“文件未找到”错误，同时传播所有其他类型的错误，可以使用：

```py
`import` errno
`try`:
    os.some_os_function_or_other()
`except` FileNotFoundError `as` err:
    print(f'Warning: file {err.filename!r} not found; continuing')
`except` OSError `as` oserr:
    print(f'Error {errno.errorcode[oserr.errno]}; continuing')
```

errno 提供了一个名为 errorcode 的字典：键是错误代码数值，对应的值是错误名称字符串，如'ENOENT'。在一些 OSError 实例的错误背后解释中显示 errno.errorcode[err.errno]通常可以使诊断对专门从事特定平台的读者更加清晰和易懂。

# pathlib 模块

pathlib 模块提供了一种面向对象的文件系统路径处理方法，整合了多种处理路径和文件的方法，将它们作为对象而不是字符串处理（与 os.path 不同）。对于大多数用例，pathlib.Path 将提供所需的一切。在少数情况下，您可能需要实例化一个特定于平台的路径，或者一个不与操作系统交互的“纯”路径；如果需要此类高级功能，请参阅 [在线文档](https://oreil.ly/ZWExX)。pathlib.Path 最常用的函数列在 Table 11-21 中，包括一个 pathlib.Path 对象 *p* 的示例。在 Windows 上，pathlib.Path 对象返回为 WindowsPath；在 Unix 上，返回为 PosixPath，如 Table 11-21 中的示例所示。（为了清晰起见，我们只是导入 pathlib 而不使用更常见和惯用的 **from** pathlib **import** Path。）

# pathlib 方法返回路径对象，而非字符串

请记住，pathlib 方法通常返回路径对象，而不是字符串，因此与 os 和 os.path 中类似的方法的结果 *不* 相同。

Table 11-21\. pathlib.Path 的常用方法

| chmod, lchmod | *p*.chmod(*mode*, follow_symlinks=**True**), *p*.lchmod(*mode*)

chmod 修改文件模式和权限，如 os.chmod（参见 Table 11-16）。在 Unix 平台上，3.10+ 将 follow_symlinks 设置为**False**，以修改符号链接的权限而不是其目标，或使用 lchmod。有关 chmod 设置的更多信息，请参见 [在线文档](https://oreil.ly/23S7z)。lchmod 类似于 chmod，但当 *p* 指向一个符号链接时，会更改符号链接而不是其目标。相当于 pathlib.Path.chmod(follow_symlinks=**False**)。 |

| cwd | pathlib.Path.cwd() 返回当前工作目录作为路径对象。 |
| --- | --- |
| exists | *p*.exists() 当 *p* 指定的是现有文件或目录（或指向现有文件或目录的符号链接）时返回 **True**；否则返回 **False**。 |
| expanduser | *p*.expanduser() 返回一个新的路径对象，其中 leading ~ 扩展为当前用户的主目录路径，或者 ~user 扩展为给定用户的主目录路径。另请参见本表中稍后的 home。 |

| glob, rglob | *p*.glob(*pattern*), *p*.rglob(*pattern*)

在 *p* 目录中以任意顺序生成所有匹配的文件。*pattern* 可以包含 ** 以允许在 *p* 或任何子目录中进行递归匹配；rglob 总是在 *p* 和所有子目录中执行递归匹配，就像 *pattern* 以 '**/' 开始一样。例如：

```py
>>> sorted(td.glob('*'))
```

```py
[WindowsPath('tempdir/bar'), 
WindowsPath('tempdir/foo')]
```

```py
>>> sorted(td.glob('**/*'))
```

```py
[WindowsPath('tempdir/bar'),
WindowsPath('tempdir/bar/baz'), 
WindowsPath('tempdir/bar/boo'), 
WindowsPath('tempdir/foo')]
```

```py
>>> sorted(td.glob('*/**/*')) *`# expanding at 2nd+ level`*
```

```py
[WindowsPath('tempdir/bar/baz'), 
WindowsPath('tempdir/bar/boo')]
```

```py
>>> sorted(td.rglob('*'))  *`# just like glob('**/*')`*
```

```py
[WindowsPath('tempdir/bar'), 
WindowsPath('tempdir/bar/baz'), 
WindowsPath('tempdir/bar/boo'), 
WindowsPath('tempdir/foo')]
```

|

| hardlink_to | *p*.hardlink_to(*target*) 3.10+ 将 *p* 设置为与 *target* 相同文件的硬链接。取代了已弃用的 link_to 3.8+，-3.10 注意：link_to 的参数顺序类似于 os.link，在 Table 11-16 中描述；而 hardlink_to 的顺序与此表后面的 symlink_to 相似，正好相反。 |
| --- | --- |
| home | pathlib.Path.home() 返回用户的主目录作为一个路径对象。 |
| is_dir | *p*.is_dir() 当*p*表示现有目录（或指向目录的符号链接）时返回**True**；否则返回**False**。 |
| is_file | *p*.is_file() 当*p*表示现有文件（或指向文件的符号链接）时返回**True**；否则返回**False**。 |
| is_mount | *p*.is_mount() 当*p*是一个挂载点（文件系统中已经挂载了另一个文件系统的地方）时返回**True**；否则返回**False**。详细信息请参阅[在线文档](https://oreil.ly/-g8r0)。在 Windows 上未实现。 |
| is_symlink | *p*.is_symlink() 当*p*表示现有符号链接时返回**True**；否则返回**False**。 |
| iterdir | *p*.iterdir() 以任意顺序生成目录*p*的内容的路径对象（不包括'.'和'..'）。当*p*不是目录时引发 NotADirectoryError。如果在创建迭代器后并且在使用完成之前，从*p*中删除文件或向*p*中添加文件，则可能产生意外结果。 |

| mkdir | *p*.mkdir(mode=0o777, parents=**False**, exist_ok=**False**) 在路径处创建一个新目录。使用 mode 设置文件模式和访问标志。设置 parents=**True**以根据需要创建任何缺少的父目录。设置 exist_ok=**True**以忽略 FileExistsError 异常。例如： |

```py
>>> td=pathlib.Path('tempdir/')
>>> td.mkdir(exist_ok=True)
>>> td.is_dir()
```

```py
True
```

查看详细内容请参阅[在线文档](https://oreil.ly/yrvuL)。 |

| open | *p*.open(mode='r', buffering=-1, encoding=**None**, errors=**None**, newline=**None**) 打开路径指向的文件，类似于内置的 open(*p*)（其他参数相同）。 |
| --- | --- |
| read_bytes | *p*.read_bytes() 返回*p*的二进制内容作为一个 bytes 对象。 |
| read_text | *p*.read_text(encoding=**None**, errors=**None**) 返回以字符串形式解码后的*p*的内容。 |
| readlink | *p*.readlink() 3.9+ 返回符号链接指向的路径。 |
| rename | *p*.rename(*target*) 将*p*重命名为*target*，并且 3.8+返回一个指向*target*的新 Path 实例。*target*可以是字符串，绝对路径或相对路径；但是，相对路径将相对于*当前*工作目录而不是*p*的目录进行解释。在 Unix 上，当*target*为现有文件或空目录时，rename 在用户有权限时静默替换它；在 Windows 上，rename 会引发 FileExistsError。 |

| replace | *p*.replace(*target*) 类似*p*.rename(*target*)，但在任何平台上，当*target*为现有文件（或在 Windows 以外的平台上为空目录）时，replace 会在用户有权限时静默替换它。例如： |

```py
>>> p.read_text()
```

```py
'spam'
```

```py
>>> t.read_text()
```

```py
'and eggs'
```

```py
>>> p.replace(t)
```

```py
WindowsPath('C:/Users/annar/testfile.txt')
```

```py
>>> t.read_text()
```

```py
'spam'
```

```py
>>> p.read_text()
```

```py
Traceback (most recent call last):
```

```py
...
```

```py
FileNotFoundError: [Errno 2] No such file...
```

|

| resolve | *p*.resolve(strict=**False**) 返回一个新的绝对路径对象，解析了符号链接并消除了任何'..'组件。设置 strict=**True**可引发异常：当路径不存在时引发 FileNotFoundError，或在遇到无限循环时引发 RuntimeError。例如，在此表格前面创建的临时目录中： |

```py
>>> td.resolve()
```

```py
PosixPath('/Users/annar/tempdir')
```

|

| rmdir | *p*.rmdir() 移除目录*p*。如果*p*不为空，则引发 OSError。 |
| --- | --- |
| samefile | *p*.samefile(*target*) 当*p*和*target*指示相同的文件时返回**True**；否则返回**False**。*target*可以是字符串或路径对象。 |
| stat | p.stat(*, follow_symlinks=**True**) 返回与路径对象有关的信息，包括权限和大小；参见 Table 11-16 中的 os.stat 以获取返回值。3.10+ 若要对符号链接本身进行 stat 操作，而不是其目标，请传递 follow_symlinks=**False**。 |
| symlink_to | *p*.symlink_to(*target*, target_is_directory=**False**) 将*p*创建为指向*target*的符号链接。在 Windows 上，如果*target*是目录，则必须设置 target_is_directory=**True**。（POSIX 忽略此参数。）（在 Windows 10+上，与 os.symlink 类似，需要开发者模式权限；有关详细信息，请参见[在线文档](https://oreil.ly/_aI9U)。）注意：参数顺序与 os.link 和 os.symlink 的顺序相反，这在 Table 11-16 中有描述。 |

| touch | *p*.touch(mode=0o666, exist_ok=**True**) 类似于 Unix 上的 touch，会在给定路径处创建一个空文件。当文件已存在时，如果 exist_ok=**True**，则更新修改时间为当前时间；如果 exist_ok=**False**，则引发 FileExistsError。例如： |

```py
>>> d
```

```py
WindowsPath('C:/Users/annar/Documents')
```

```py
>>> f = d / 'testfile.txt'
>>> f.is_file()
```

```py
False
```

```py
>>> f.touch()
>>> f.is_file()
```

```py
True
```

|

| unlink | *p*.unlink(missing_ok=**False**) 移除文件或符号链接*p*。（对于目录，请参见本表前面描述的 rmdir。）3.8+ 传递 missing_ok=**True**以忽略[FileExistsError](https://oreil.ly/wuPTp)。 |
| --- | --- |
| write_bytes | *p*.write_bytes(*data*) 以字节模式打开（或创建，如果需要的话）指向的文件，将*data*写入其中，然后关闭文件。如果文件已存在，则覆盖该文件。 |
| write_text | *p*.write_text(data, encoding=**None**, errors=**None**, newline=**None**) 以文本模式打开（或创建，如果需要的话）指向的文件，将*data*写入其中，然后关闭文件。如果文件已存在，则覆盖该文件。3.10+ 当 newline 为**None**（默认值）时，将任何'\n'转换为系统默认换行符；当为'\r'或'\r\n'时，将'\n'转换为给定的字符串；当为''或'\n'时，不进行任何转换。 |

pathlib.Path 对象还支持在 Table 11-22 中列出的属性，以访问路径字符串的各个组成部分。请注意，一些属性是字符串，而其他属性是 Path 对象。（为简洁起见，OS 特定类型（如 PosixPath 或 WindowsPath）仅使用抽象 Path 类显示。）

Table 11-22\. pathlib.Path 实例 p 的属性

| Attribute | 描述 | Unix 路径 Path('/usr/bin/ python')的值 | Windows 路径 Path(r’c:\Python3\ python.exe')的值 |
| --- | --- | --- | --- |
| anchor | 驱动器和根的组合 | '/' | 'c:\\' |
| drive | *p*的驱动器字母 | '' | 'c:' |
| name | *p*的末尾组件 | 'python' | 'python.exe' |
| parent | *p*的父目录 | Path('/usr/bin') | Path('c:\\Python3') |
| parents | *p* 的祖先目录 | (Path('/usr/ bin'), Path('/usr'), Path('/')) | (Path('c:\\Python3'), Path('c:\\')) |
| parts | *p* 的所有组成部分的元组 | ('/', 'usr', 'bin', 'python') | ('c:\\', 'Python3', 'python.exe') |
| root | *p* 的根目录 | '/' | '\\' |
| stem | *p* 的名称，不包括后缀 | 'python' | 'python' |
| suffix | *p* 的结束后缀 | '' | '.exe' |
| suffixes | 由 '.' 字符分隔的 *p* 的所有后缀的列表 | [] | ['.exe'] |

[在线文档](https://oreil.ly/uBDrd) 包含了更多带有额外组件（如文件系统和 UNC 共享）的路径示例。

pathlib.Path 对象还支持`/`操作符，是 os.path.join 或 Path 模块中 Path.joinpath 的一个很好的替代方案。请参见 Table 11-21 中 Path.touch 描述中的示例代码。

# stat 模块

函数 os.stat（在 Table 11-16 中介绍）返回 stat_result 的实例，其项索引、属性名称和含义也在那里介绍了。 stat 模块提供了类似于 stat_result 属性的大写名称的属性，并且相应的值是相应的项索引。

stat 模块的更有趣的内容是用于检查 stat_result 实例的 st_mode 属性并确定文件类型的函数。os.path 也提供了用于这种任务的函数，它们直接在文件的 *path* 上操作。与 os 的函数相比，在对同一文件执行多个测试时，stat 提供的函数（在 Table 11-23 中显示）更快：它们在一系列测试开始时只需要一个 os.stat 系统调用来获取文件的 st_mode，而 os.path 中的函数在每次测试时隐式地向操作系统请求相同的信息。每个函数在 *mode* 表示给定类型的文件时返回 **True**；否则，返回 **False**。

Table 11-23\. stat 模块函数用于检查 st_mode

| S_ISBLK | S_ISBLK(*mode*) 表示 *mode* 是否表示一个块设备文件 |
| --- | --- |
| S_ISCHR | S_ISCHR(*mode*) 表示 *mode* 是否表示一个字符设备文件 |
| S_ISDIR | S_ISDIR(*mode*) 表示 *mode* 是否表示一个目录 |
| S_ISFIFO | S_ISFIFO(*mode*) 表示 *mode* 是否表示一个 FIFO（也称为“命名管道”） |
| S_ISLNK | S_ISLNK(*mode*) 表示 *mode* 是否表示一个符号链接 |
| S_ISREG | S_ISREG(*mode*) 表示 *mode* 是否表示一个普通文件（不是目录、特殊设备文件等） |
| S_ISSOCK | S_ISSOCK(*mode*) 表示 *mode* 是否表示一个 Unix 域套接字 |

这些函数中的几个仅在类 Unix 系统上有意义，因为其他平台不会将设备和套接字等特殊文件保留在与常规文件相同的命名空间中；类 Unix 系统会这样做。

`stat` 模块还提供了两个函数，从文件的 *mode* (*x*.st_mode，对于 `os.stat` 函数的某些结果 *x*) 提取相关部分，列在表 11-24。

表 11-24\. `stat` 模块从模式中提取位的函数

| S_IFMT | S_IFMT(*mode*) 返回描述文件类型的*mode* 中的位（即函数 `S_ISDIR`、`S_ISREG` 等检查的位） |
| --- | --- |
| S_IMODE | S_IMODE(*mode*) 返回*mode* 中可以由函数 `os.chmod` 设置的位（即权限位，以及在类 Unix 平台上，一些特殊位，如设置用户标识标志） |

`stat` 模块提供了一个实用函数 `stat.filemode(*mode*)`，将文件的模式转换为形如 '-rwxrwxrwx' 的人类可读字符串。

# `filecmp` 模块

`filecmp` 模块提供了一些有用的用于比较文件和目录的函数，列在表 11-25。

表 11-25\. `filecmp` 模块的有用函数

| clear_cache | clear_cache() 清除 `filecmp` 缓存，这在快速文件比较中可能很有用。 |
| --- | --- |
| cmp | cmp(*f1*, *f2*, shallow=**True**) 比较由路径字符串 *f1* 和 *f2* 标识的文件（或 `pathlib.Paths`）。如果文件被认为相等，则 `cmp` 返回 **True**；否则返回 **False**。如果 `shallow` 为 **True**，则如果它们的 stat 元组相等，文件被视为相等。当 `shallow` 为 **False** 时，`cmp` 读取并比较 stat 元组相等的文件的内容。 |
| cmpfiles | cmpfiles(*dir1*, *dir2*, *common*, shallow=**True**) 循环处理序列 *common*。*common* 中的每个项都是同时存在于目录 *dir1* 和 *dir2* 中的文件名。`cmpfiles` 返回一个元组，其项为三个字符串列表：（*equal*，*diff* 和 *errs*）。*equal* 是两个目录中相等文件的名称列表，*diff* 是不同目录之间不同文件的名称列表，*errs* 是无法比较的文件名称列表（因为它们不同时存在于两个目录中，或者没有权限读取它们的其中一个或两个）。参数 `shallow` 与 `cmp` 相同。 |

`filecmp` 模块还提供了 `dircmp` 类。该类的构造函数签名如下：

| dircmp | **class** dircmp(*dir1*, *dir2*, ignore=**None**, hide=**None**) 创建一个新的目录比较实例对象，比较目录 *dir1* 和 *dir2*，忽略 `ignore` 中列出的名称，隐藏 `hide` 中列出的名称（默认为 '.' 和 '..' 当 `hide` 为 **None** 时）。`ignore` 的默认值由 `filecmp` 模块的 `DEFAULT_IGNORE` 属性提供；在撰写本文时，默认值为 `['RCS', 'CVS', 'tags', '.git', '.hg', '.bzr', '_darcs', '__pycache__']`。目录中的文件与 `filecmp.cmp` 使用 `shallow=**True**` 进行比较。 |
| --- | --- |

`dircmp` 实例 *d* 提供三种方法，详见表 11-26。

Table 11-26\. dircmp 实例 d 提供的方法

| report | report_full_closure() 输出 *dir1* 和 *dir2* 以及它们所有共同的子目录之间的比较结果到 sys.stdout，递归进行 |
| --- | --- |
| report_fu⁠l⁠l⁠_​c⁠l⁠o⁠sure | report_full_closure() 输出 *dir1* 和 *dir2* 以及它们所有共同的子目录之间的比较结果到 sys.stdout，递归进行 |
| report_parti⁠a⁠l⁠_​c⁠l⁠o⁠sure | report_partial_closure() 输出 *dir1* 和 *dir2* 及它们共同的直接子目录之间的比较结果到 sys.stdout |

另外，*d* 提供了几个属性，详见 Table 11-27。这些属性是“按需计算”的（即，只在需要时才计算，多亏了 __getattr__ 特殊方法），因此使用 dircmp 实例不会产生不必要的开销。

Table 11-27\. dircmp 实例 d 提供的属性

| common | *dir1* 和 *dir2* 中的文件和子目录 |
| --- | --- |
| common_dirs | *dir1* 和 *dir2* 中的子目录 |
| common_files | *dir1* 和 *dir2* 中的文件 |
| common_funny | *dir1* 和 *dir2* 中的名称，其中 os.stat 报告错误或者两个目录中版本的类型不同 |
| diff_files | *dir1* 和 *dir2* 中内容不同的文件 |
| funny_files | *dir1* 和 *dir2* 中的文件，但无法进行比较 |
| left_list | *dir1* 中的文件和子目录 |
| left_only | *dir1* 中的文件和子目录，但不在 *dir2* 中 |
| right_list | *dir2* 中的文件和子目录 |
| right_only | *dir2* 中的文件和子目录，但不在 *dir1* 中 |
| same_files | *dir1* 和 *dir2* 中内容相同的文件 |
| subdirs | 一个字典，其键是 common_dirs 中的字符串；相应的值是 dircmp 的实例（或者 3.10+ 中与 *d* 同一种类的 dircmp 子类的实例），用于每个子目录 |

# fnmatch 模块

fnmatch 模块（*filename match* 的缩写）用于匹配类 Unix shell 使用的模式的文件名字符串或路径，如 Table 11-28 所示。

Table 11-28\. fnmatch 模式匹配惯例

| 模式 | 匹配项 |
| --- | --- |
| * | 任意字符序列 |
| ? | 任意单个字符 |
| [*chars*] | *chars* 中的任意一个字符 |
| [!*chars*] | 不属于 *chars* 中任意一个字符 |

fnmatch 不遵循类 Unix shell 模式匹配的其他惯例，比如特殊对待斜杠 (/) 或前导点 (.)。它也不允许转义特殊字符：而是要匹配特殊字符，将其括在方括号中。例如，要匹配一个文件名为单个右括号，使用 '[]]'。

fnmatch 模块提供了 Table 11-29 中列出的函数。

Table 11-29\. fnmatch 模块的函数

| filter | filter(*names*, *pattern*) 返回 *names*（一个字符串序列）中与 *pattern* 匹配的项目列表。 |
| --- | --- |
| fnmatch | fnmatch(*filename*, *pattern*) 当字符串 *filename* 与 *pattern* 匹配时返回 **True**；否则返回 **False**。当平台为大小写敏感时（例如典型的类 Unix 系统），匹配区分大小写；否则（例如在 Windows 上），不区分大小写；请注意，如果处理的文件系统的大小写敏感性与您的平台不匹配（例如 macOS 是类 Unix 的，但其典型文件系统不区分大小写）。 |
| fnmatchcase | fnmatchcase(*filename*, *pattern*) 当字符串 *filename* 与 *pattern* 匹配时返回 **True**；否则返回 **False**。此匹配在任何平台上均区分大小写。 |
| translate | translate(*pattern*) 返回等同于 fnmatch 模式 *pattern* 的正则表达式模式（详见“模式字符串语法”）。 |

# glob 模块

glob 模块以任意顺序列出与 *路径模式* 匹配的文件路径名，使用与 fnmatch 相同的规则；此外，它特别处理前导点（.）、分隔符（/）和 **，就像 Unix shell 一样。Table 11-30 列出了 glob 模块提供的一些有用函数。

Table 11-30\. glob 模块的函数

| escape | escape(*pathname*) 转义所有特殊字符（'?'、'*' 和 ''），因此可以匹配可能包含特殊字符的任意字面字符串。 |
| --- | --- |
| glob | glob(*pathname*, *, root_dir=**None**, dir_fd=**None**, recursive=**False**) 返回与模式 *pathname* 匹配的文件的路径名列表。root_dir（如果不是 **None**）是指定搜索的根目录的字符串或类似路径的对象（这类似于在调用 glob 之前更改当前目录）。如果 *pathname* 是相对路径，则返回的路径是相对于 root_dir 的。要搜索相对于目录描述符的路径，请传递 dir_fd。可选地传递命名参数 recursive=**True** 以使路径组件递归地匹配零个或多个子目录级别。 |
| iglob | iglob(*pathname, *,* root_dir=**None**, dir_fd=**None**, recursive=**False**) 类似于 glob，但返回一个迭代器，每次生成一个相关的路径名。 |

# shutil 模块

shutil 模块（*shell utilities* 的缩写）提供了复制、移动文件以及删除整个目录树的函数。在某些 Unix 平台上，大多数函数支持可选的仅关键字参数 follow_symlinks，默认为 **True**。当 follow_symlinks=**True** 时，如果路径指示为符号链接，则函数会跟随它以达到实际文件或目录；当 **False** 时，函数操作的是符号链接本身。[Table 11-31 列出了 shutil 模块提供的函数。

Table 11-31\. shutil 模块的函数

| copy | copy(*src*, *dst*) 复制由*src*指定的文件的内容（*src*必须存在），并创建或覆盖由*dst*指定的文件（*src*和*dst*可以是字符串或 pathlib.Path 实例）。如果*dst*是一个目录，则目标是与*src*同名的文件，但位于*dst*中。copy 还复制权限位，但不复制最后访问和修改时间。返回已复制到的目标文件的路径。 |
| --- | --- |
| copy2 | copy2(*src*, *dst*) 类似于 copy，但也复制最后访问时间和修改时间。 |
| copyfile | copyfile(*src*, *dst*) 仅复制由*src*指定的文件的内容（不包括权限位、最后访问和修改时间），创建或覆盖由*dst*指定的文件。 |
| copyfileobj | copyfileobj(*fsrc*, *fdst*, bufsize=16384) 从已打开用于读取的文件对象*fsrc*向已打开用于写入的文件对象*fdst*复制所有字节。如果*bufsize*大于 0，则每次最多复制*bufsize*字节。文件对象的具体内容请参见“The io Module”。 |
| copymode | copymode(*src*, *dst*) 复制由*src*指定的文件或目录的权限位到由*dst*指定的文件或目录。*src*和*dst*都必须存在。不更改*dst*的内容，也不更改其作为文件或目录的状态。 |
| copystat | copystat(*src*, *dst*) 复制由*src*指定的文件或目录的权限位、最后访问时间和修改时间到由*dst*指定的文件或目录。*src*和*dst*都必须存在。不更改*dst*的内容，也不更改其作为文件或目录的状态。 |

| copytree | copytree(*src*, *dst*, symlinks=**False**, ignore=**None**, copy_function=copy2, ignore_dangling_symlinks=**False**, dirs_exist_ok=**False**) 复制以*src*命名的目录为根的目录树到以*dst*命名的目标目录中。*dst*不得已存在：copytree 将创建它（以及创建任何缺失的父目录）。copytree 默认使用 copy2 函数复制每个文件；您可以选择作为命名参数 copy_function 传递不同的文件复制函数。如果在复制过程中发生任何异常，copytree 将在最后记录它们并继续执行，最终引发包含所有记录异常列表的错误。

当 symlinks 为**True**时，copytree 在新树中创建符号链接，如果在源树中发现符号链接。当 symlinks 为**False**时，copytree 跟随它找到的每个符号链接，并复制链接的文件，使用链接的名称记录异常（如果 ignore_dangling_symlinks 为**True**，则忽略此异常）。在不支持符号链接概念的平台上，copytree 会忽略 symlinks 参数。 |

| copytree *(续)* | 当 ignore 不为**None**时，它必须是一个接受两个参数（目录路径和目录的直接子项列表）并返回要在复制过程中忽略的子项列表的可调用对象。如果存在，ignore 通常是对 shutil.ignore_patterns 的调用结果。例如，以下代码：

```py
`import` shutil
ignore = shutil.ignore_patterns('.*', '*.bak')
shutil.copytree('src', 'dst', ignore=ignore)
```

将源自目录 src 的树复制到新目录 dst 中的树，忽略任何以点开头的文件或子目录以及任何以*.bak*结尾的文件或子目录。

默认情况下，如果目标目录已经存在，copytree 会记录 FileExistsError 异常。从 3.8 版本开始，可以将 dirs_exist_ok 设置为**True**，允许 copytree 在复制过程中写入已存在的目录（可能会覆盖其内容）。

| ignore_patterns | ignore_patterns(**patterns*) 返回一个可调用对象，选出与*patterns*匹配的文件和子目录，类似于 fnmatch 模块中使用的模式（见“fnmatch 模块”）。结果适合作为 copytree 函数的 ignore 参数传递。 |
| --- | --- |
| move | move(*src*, *dst*, copy_function=copy2) 将名为*src*的文件或目录移动到名为*dst*的位置。move 首先尝试使用 os.rename。然后，如果失败（因为*src*和*dst*位于不同的文件系统上，或者*dst*已经存在），move 将*src*复制到*dst*（默认情况下使用 copy2 复制文件或 copytree 复制目录；您可以选择传递名为 copy_function 的文件复制函数作为命名参数），然后删除*src*（对于文件使用 os.unlink，对于目录使用 rmtree）。 |
| rmtree | rmtree(*path*, ignore_errors=**False**, onerror=**None**) 删除以*path*为根的目录树。当 ignore_errors 为**True**时，rmtree 忽略错误。当 ignore_errors 为**False**且 onerror 为**None**时，错误会引发异常。当 onerror 不为**None**时，必须是一个可调用的函数，接受三个参数：*func*是引发异常的函数（os.remove 或 os.rmdir），*path*是传递给*func*的路径，*ex*是 sys.exc_info 返回的信息元组。当 onerror 引发异常时，rmtree 终止，并且异常传播。 |

除了提供直接有用的函数外，Python 标准库中的源文件*shutil.py*是如何使用许多 os 函数的优秀示例。

# 文本输入和输出

Python 将非 GUI 文本输入和输出流呈现为 Python 程序的文件对象，因此可以使用文件对象的方法（在“文件对象的属性和方法”中介绍）来操作这些流。

## 标准输出和标准错误

sys 模块（在“sys 模块”中讨论）具有 stdout 和 stderr 属性，它们都是可写的文件对象。除非你正在使用 shell 重定向或管道，否则这些流连接到运行你的脚本的“终端”。如今，实际的终端非常罕见：所谓的终端通常是支持文本 I/O 的屏幕窗口。

sys.stdout 和 sys.stderr 之间的区别是惯例的问题。sys.stdout，即*标准输出*，是程序输出结果的地方。sys.stderr，即*标准错误*，是错误、状态或进度消息等输出的地方。将程序输出与状态和错误消息分开有助于有效地使用 shell 重定向。Python 遵循这一惯例，将 sys.stderr 用于自身的错误和警告消息。

## print 函数

将结果输出到标准输出的程序通常需要写入 sys.stdout。Python 的 print 函数（在表 8-2 中讨论）可以作为一个丰富而方便的替代方案，用于在开发过程中进行非正式输出，以帮助调试代码，但是对于生产输出，你可能需要比 print 提供的格式控制更多。例如，你可能需要控制间距、字段宽度、浮点值的小数位数等。如果是这样，你可以将输出准备为 f-string（在“字符串格式化”中讨论），然后将字符串输出，通常使用适当文件对象的 write 方法。（你可以将格式化的字符串传递给 print，但 print 可能会添加空格和换行；write 方法根本不会添加任何内容，因此更容易控制输出的确切内容。）

如果你需要将输出直接定向到一个打开写入模式的文件*f*，直接调用*f*.write 通常是最好的方法，而 print(..., file=*f*)有时是一个方便的替代方法。要重复地将 print 调用的输出定向到某个文件，你可以临时改变 sys.stdout 的值。下面的示例是一个通用的重定向函数，可用于这种临时更改；在多任务存在的情况下，请确保还添加一个锁以避免任何争用（参见也在表 6-1 中描述的 contextlib.redirect_stdout 装饰器）：

```py
`def` redirect(func: Callable, *a, **k) -> (str, Any):
    *`"""redirect(func, *a, **k) -> (func's results, return value)`*
 *`func is a callable emitting results to standard output.`*
 *`redirect captures the results as a str and returns a pair`*
 *`(output string, return value).`*
 *`"""`*
    `import` sys, io
    save_out = sys.stdout
    sys.stdout = io.StringIO()
    `try`:
        retval = func(*args, **kwds)
        `return` sys.stdout.getvalue(), retval
    `finally`:
        sys.stdout.close()
        sys.stdout = save_out
```

## 标准输入

除了 stdout 和 stderr 之外，sys 模块还提供了 stdin 属性，它是一个可读的文件对象。当你需要从用户那里获取一行文本时，可以调用内置函数 input（在表 8-2 中讨论），可选地使用一个字符串参数作为提示。

当你需要的输入不是字符串时（例如，当你需要一个数字时），使用 `input` 从用户那里获取一个字符串，然后使用其他内置函数如 `int`、`float` 或 `ast.literal_eval` 将字符串转换为你需要的数字。要评估来自不受信任来源的表达式或字符串，建议使用标准库模块 `ast` 中的 `literal_eval` 函数（如在[在线文档](https://oreil.ly/6kb6T)中描述）。`ast.literal_eval(*astring*)` 在可能时返回一个有效的 Python 值（例如 int、float 或 list），对于给定的字面值 *astring*，否则会引发 SyntaxError 或 ValueError 异常；它永远不会产生任何副作用。为了确保完全安全，*astring* 不能包含任何运算符或非关键字标识符；然而，+ 和 - 可以被接受作为数字的正负号，而不是运算符。例如：

```py
`import` ast
print(ast.literal_eval('23'))     *`# prints 23`*
print(ast.literal_eval(' 23'))   *`# prints 23 (3.10++)`*
print(ast.literal_eval('[2,-3]')) *`# prints [2, -3]`*
print(ast.literal_eval('2+3'))    *`# raises ValueError`*
print(ast.literal_eval('2+'))     *`# raises SyntaxError`*
```

# `eval` 可能存在危险

不要对任意未经处理的用户输入使用 `eval`：一个恶意（或者无意中疏忽的）用户可以通过这种方式突破安全性或者造成其他损害。没有有效的防御措施——避免在来自不完全信任的来源的输入上使用 `eval`（和 `exec`）。

## `getpass` 模块

偶尔，你可能希望用户以一种屏幕上看不到用户输入内容的方式输入一行文本。例如，当你要求用户输入密码时。`getpass` 模块提供了这种功能，以及获取当前用户用户名的函数（参见 Table 11-32）。

Table 11-32\. `getpass` 模块的功能

| `getpass` | `getpass(prompt='Password: ')` 与 `input` 类似（在 Table 8-2 中介绍），不同之处在于用户输入的文本在用户输入时不会显示在屏幕上，并且默认提示与 `input` 不同。 |
| --- | --- |
| `getuser` | `getuser()` 返回当前用户的用户名。`getuser` 尝试将用户名作为环境变量 `LOGNAME`、`USER`、`LNAME` 或 `USERNAME` 的一个值来获取，依次尝试。如果 `os.environ` 中没有这些变量的值，`getuser` 会向操作系统询问。 |

# 富文本 I/O

到目前为止，所涵盖的文本 I/O 模块在所有平台终端上提供基本的文本 I/O 功能。大多数平台还提供增强的文本 I/O 功能，例如响应单个按键（而不仅仅是整行）、在任何终端行和列位置打印文本，以及使用背景和前景颜色以及加粗、斜体和下划线等字体效果增强文本。对于这种功能，你需要考虑使用第三方库。我们在这里关注 `readline` 模块，然后快速查看一些控制台 I/O 选项，包括 `mscvrt`，并简要提及 `curses`、`rich` 和 `colorama`，我们不会进一步介绍它们。

## `readline` 模块

readline 模块封装了 [GNU Readline Library](https://oreil.ly/Z0A5t)，允许用户在交互输入期间编辑文本行并回溯先前的行以进行编辑和重新输入。Readline 预安装在许多类 Unix 平台上，并且可在线获取。在 Windows 上，您可以安装和使用第三方模块 [pyreadline](https://oreil.ly/9XExm)。

当可用时，Python 使用 readline 处理所有基于行的输入，例如 input。交互式 Python 解释器始终尝试加载 readline 以启用交互会话的行编辑和回溯。某些 readline 函数控制高级功能，特别是 *history* 用于回溯在先前会话中输入的行，以及 *completion* 用于正在输入的单词的上下文敏感完成（详见 [Python readline 文档](https://oreil.ly/6SMkN) 关于配置命令的完整详细信息）。您可以使用 Table 11-33 中的函数访问模块的功能。

Table 11-33\. readline 模块的函数

| add_history | add_history(*s*, */*) 将字符串 *s* 作为一行添加到历史缓冲区的末尾。要临时禁用 add_history，请调用 set_auto_history(**False**)，这将仅在本次会话中禁用 add_history（不会跨会话持久保存）；set_auto_history 默认为 **True**。 |
| --- | --- |
| app⁠e⁠n⁠d⁠_​h⁠i⁠s⁠t⁠o⁠r⁠y_​f⁠i⁠l⁠e | append_history_file(*n*, *filename*='~/.history', /) 将最后 *n* 项追加到现有文件 *filename*。 |
| clear_history | clear_history() 清除历史缓冲区。 |
| get_completer | get_completer() 返回当前的补全函数（由 set_completer 最后设置），如果没有设置补全函数，则返回 **None**。 |
| g⁠e⁠t⁠_​h⁠i⁠s⁠tory_length | get_history_length() 返回要保存到历史文件中的行数。当结果小于 0 时，保存所有历史行。 |

| parse_and_bind | parse_and_bind(*readline_cmd*, */*) 给 readline 提供配置命令。要让用户按 Tab 键请求完成，请调用 parse_and_bind('tab: complete')。查看 [readline 文档](https://oreil.ly/Wv2dm) 获取字符串 *readline_cmd* 的其他有用值。

一个良好的补全函数位于标准库模块 rlcompleter 中。在交互式解释器中（或在交互会话开始时执行的启动文件中，见 “Environment Variables”），输入：

```py
`import` readline, rlcompleter
readline.parse_and_bind('tab: complete')
```

在本次交互会话的其余部分，您可以在行编辑过程中按 Tab 键并获得全局名称和对象属性的完成。

| re⁠a⁠d⁠_​h⁠i⁠s⁠t⁠o⁠r⁠y_​f⁠i⁠l⁠e | read_history_file(*filename*='~/.history', /) 从路径 *filename* 的文本文件中加载历史行。 |
| --- | --- |
| re⁠a⁠d⁠_​i⁠n⁠i⁠t_​f⁠i⁠l⁠e | read_init_file(*filename*=**None, /**) 使 readline 加载文本文件：每行是一个配置命令。当 *filename* 为 **None** 时，加载与上次相同的文件。 |
| set_completer | set_completer(*func, /*) 设置完成函数。当 *func* 为 **None** 或省略时，readline 禁用完成。否则，当用户键入部分单词 *start*，然后按 Tab 键时，readline 调用 *func*(*start*, *i*)，其中 *i* 最初为 0。*func* 返回以 *start* 开头的第 *i* 个可能的单词，或在没有更多单词时返回 **None**。readline 循环调用 *func*，*i* 设置为 0、1、2 等，直到 *func* 返回 **None**。 |
| s⁠e⁠t⁠_​h⁠i⁠s⁠t⁠o⁠r⁠y⁠_​l⁠e⁠n⁠g⁠t⁠h | set_history_length(*x, /*) 设置要保存到历史文件中的历史行数。当 *x* 小于 0 时，将保存历史中的所有行。 |
| wri⁠t⁠e⁠_​h⁠i⁠s⁠t⁠o⁠r⁠y⁠_​f⁠i⁠l⁠e | write_history_file(*filename*='~/.history') 将历史记录行保存到名为 *filename* 或路径为 *filename* 的文本文件中，覆盖任何现有文件。 |

## 控制台 I/O

如前所述，“终端”今天通常是图形屏幕上的文本窗口。理论上，您也可以使用真正的终端，或者（也许略微现实些，但这些天不太可能）个人计算机的控制台（主屏幕）以文本模式运行。今天使用的所有这些“终端”都提供平台相关的高级文本 I/O 功能。低级 curses 包适用于类 Unix 平台。对于跨平台（Windows、Unix、macOS）解决方案，您可以使用第三方包 [rich](https://oreil.ly/zuTRT)；除了其出色的 [在线文档](https://oreil.ly/BHr83) 外，还有在线 [教程](https://oreil.ly/GYAnd) 可帮助您入门。要在终端上输出彩色文本，请参阅 PyPI 上的 colorama。接下来介绍的 **msvcrt** 提供了一些低级（仅限 Windows）函数。

### curses

增强的终端 I/O 的经典 Unix 方法因历史原因被称为 curses。Python 包 curses 允许您在需要时进行详细控制。本书不涵盖 curses；更多信息请参阅 A.M. Kuchling 和 Eric Raymond 的在线教程 [“Curses Programming with Python”](https://oreil.ly/Pbpbh)。

### **msvcrt** 模块

**msvcrt** 模块只能在 Windows 上使用（你可能需要使用 pip 安装），它提供了一些低级函数，使 Python 程序可以访问由 Microsoft Visual C++ 运行时库 *msvcrt.dll* 提供的专有附加功能。例如，Table 11-34 中列出的函数允许你逐字符而不是逐行读取用户输入。

Table 11-34\. **msvcrt** 模块的一些有用函数

| getch, getche | getch(), getche() 从键盘输入读取并返回单个字符的字节，并在必要时阻塞，直到有一个可用（即按下一个键）。getche 将字符回显到屏幕上（如果可打印），而 getch 则不会。当用户按下特殊键（箭头键、功能键等）时，它被视为两个字符：首先是 chr(0) 或 chr(224)，然后是第二个字符，这两个字符共同定义用户按下的特殊键。这意味着程序必须调用两次 getch 或 getche 才能读取这些按键。要了解 getch 对任何键返回什么，请在 Windows 机器上运行以下小脚本：

```py
`import` msvcrt
print("press z to exit, or any other key "
      "to see the key's code:")
`while` `True`:
    c = msvcrt.getch()
    `if` c == b'z':
        `break`
    print(f'{ord(c)} ({c!r})')
```

|

| kbhit | kbhit() 当有字符可读时返回**True**（调用 getch 时立即返回）；否则返回**False**（调用 getch 时等待）。 |
| --- | --- |
| ungetch | ungetch(*c*) “取消”字符*c*；接下来的 getch 或 getche 调用会返回*c*。调用两次 ungetch 而没有调用中间的 getch 或 getche 是错误的。 |

# 国际化

许多程序将某些信息以文本形式呈现给用户。这样的文本应该能够被不同地区的用户理解和接受。例如，在某些国家和文化中，“March 7”这个日期可以简洁地表示为“3/7”。而在其他地方，“3/7”表示“7 月 3 日”，而表示“March 7”的字符串则是“7/3”。在 Python 中，这样的文化习惯是通过标准库模块 locale 处理的。

同样，一个问候语可能用字符串“Benvenuti”在一种自然语言中表达，而在另一种语言中，则需要使用“Welcome”这个字符串。在 Python 中，这些翻译是通过标准库模块 gettext 处理的。

这两种问题通常在称为*国际化*的总称下处理（通常缩写为*i18n*，因为英文全拼中*i*和*n*之间有 18 个字母）——这个名称不准确，因为这些问题不仅适用于国家之间，还适用于同一国家内不同的语言或文化。⁵

## 模块 locale

Python 对文化习惯的支持模仿了 C 语言，稍作简化。程序在称为*locale*的文化习惯环境中运行。Locale 设置渗透到程序中，并通常在程序启动时设置。Locale 不是线程特定的，而且 locale 模块不是线程安全的。在多线程程序中，应在主线程中设置程序的 locale；即在启动次要线程之前设置。

# locale 的局限性

locale 只对进程范围的设置有效。如果您的应用程序需要在同一个进程中同时处理多种 locale——无论是在线程中还是异步地——由于其进程范围的特性，locale 并不适合。考虑使用像[PyICU](https://pypi.org/project/PyICU)这样的替代方案，见“更多国际化资源”。

如果程序没有调用 locale.setlocale，则使用 *C locale*（因为 Python 的 C 语言根源而得名）；它与美国英语区域类似但不完全相同。另外，程序可以查找并接受用户的默认区域设置。在这种情况下，区域模块通过与操作系统的交互（通过环境或其他系统相关方式）尝试找到用户首选的区域设置。最后，程序可以设置特定的区域设置，据此确定要设置的区域设置，可能基于用户交互或持久配置设置。

区域设置通常会跨文化习惯的相关类别进行全面设置。这种广泛的设置由区域模块的常量属性 LC_ALL 表示。然而，由区域处理的文化习惯被分成不同类别，在某些罕见的情况下，程序可以选择混合和匹配这些类别，以构建合成的复合区域。这些类别由 表 11-35 中列出的属性标识。

表 11-35\. 区域模块的常量属性

| LC_COLLATE | 字符串排序；影响区域设置下的函数 strcoll 和 strxfrm |
| --- | --- |
| LC_CTYPE | 字符类型；影响与小写和大写字母相关的 string 模块（以及其方法）的某些方面 |
| LC_MESSAGES | 消息；可能影响操作系统显示的消息（例如函数 os.strerror 和模块 gettext 显示的消息） |
| LC_MONETARY | 货币值格式化；影响区域设置下的函数 localeconv 和 currency |
| LC_NUMERIC | 数字格式化；影响区域设置下的函数 atoi、atof、format_string、localeconv 和 str，以及在格式字符串（如 f-strings 和 str.format）中使用格式字符 'n' 时使用的数字分隔符 |
| LC_TIME | 时间和日期格式化；影响函数 time.strftime |

某些类别的设置（由 LC_CTYPE、LC_MESSAGES 和 LC_TIME 表示）会影响其他模块中的行为（如 string、os、gettext 和 time）。其他类别（由 LC_COLLATE、LC_MONETARY 和 LC_NUMERIC 表示）仅影响区域本身的某些函数（以及在 LC_NUMERIC 情况下的字符串格式化）。

区域模块提供了 表 11-36 中列出的函数，用于查询、更改和操作区域设置，以及实施 LC_COLLATE、LC_MONETARY 和 LC_NUMERIC 类别的文化习惯的函数。

表 11-36\. 区域模块的有用函数

| atof | atof(*s*) 使用当前 LC_NUMERIC 设置将字符串 *s* 解析为浮点数。 |
| --- | --- |
| atoi | atoi(*s*) 使用当前 LC_NUMERIC 设置将字符串 *s* 解析为整数。 |
| currency | currency(*data,* grouping=**False**, international=**False**) 返回带有货币符号的字符串或数字 *data*，如果 grouping 为 **True**，则使用货币千位分隔符和分组。当 international 为 **True** 时，使用后面表中描述的 int_curr_symbol 和 int_frac_digits。 |

| f⁠o⁠r⁠m⁠a⁠t⁠_​s⁠t⁠r⁠i⁠n⁠g | format_string(*fmt*, *num*, grouping=**False**, monetary=**False**) 返回通过根据格式字符串 *fmt* 和 LC_NUMERIC 或 LC_MONETARY 设置对 *num* 进行格式化而获得的字符串。除了文化习惯问题外，结果类似于旧式 *fmt* % *num* 字符串格式化，详见 “使用 % 进行传统字符串格式化”。如果 *num* 是数字类型的实例，并且 *fmt* 是 *%d* 或 *%f*，将 grouping 设置为 **True** 可以根据 LC_NUMERIC 设置在结果字符串中对数字进行分组。如果 monetary 为 **True**，字符串将使用 mon_decimal_point 格式化，并且 *grouping* 使用 mon_thousands_sep 和 mon_grouping 而不是 LC_NUMERIC 提供的设置（有关这些设置的更多信息，请参见表后的 localeconv）。例如：

```py
>>> locale.setlocale(locale.LC_NUMERIC, 
...                  'en_us')
```

```py
'en_us'
```

```py
>>> n=1000*1000
>>> locale.format_string('%d', n)
```

```py
'1000000'
```

```py
>>> locale.setlocale(locale.LC_MONETARY, 
...                  'it_it')
```

```py
'it_it'
```

```py
>>> locale.format_string('%f', n)
```

```py
'1000000.000000'  *# uses decimal_point*
```

```py
>>> locale.format_string('%f', n, 
...                      monetary=True)
```

```py
'1000000,000000'  *# uses mon_decimal_point*
```

```py
>>> locale.format_string('%0.2f', n, 
...                      grouping=True)
```

```py
'1,000,000.00' *# separators & decimal from*
 *# LC_NUMERIC*
```

```py
>>> locale.format_string('%0.2f', n, 
...                      grouping=True,
...                      monetary=True)
```

```py
'1.000.000,00'    *# separators & decimal from* 
 *# LC_MONETARY*
```

在这个例子中，由于数字区域设置为美国英语，在参数 grouping 为 **True** 时，format_string 会使用逗号将数字每三位分组，并使用点 (.) 作为小数点。然而，货币区域设置为意大利时，当参数 monetary 为 **True** 时，format_string 使用逗号 (,) 作为小数点，分组使用点 (.) 作为千位分隔符。通常，在任何给定的区域设置内，货币和非货币数字的语法是相同的。 |

| g⁠e⁠t​d⁠e⁠f⁠a⁠u⁠l⁠t​l⁠o⁠c⁠a⁠l⁠e | getdefaultlocale(envvars=('LANGUAGE', 'LC_ALL', 'LC_TYPE', 'LANG')) 检查按顺序指定的 envvars 环境变量。在环境中找到的第一个变量确定默认区域设置。getdefaultlocale 返回一对符合 [RFC 1766](https://oreil.ly/BbYK1) 的字符串 (*lang*, *encoding*)（除了 'C' 区域设置），例如 ('en_US', 'UTF-8')。如果 gedefaultlocale 无法确定某个项的值，则每对中的每一项可能为 **None**。 |
| --- | --- |
| g⁠e⁠t​l⁠o⁠c⁠a⁠l⁠e | getlocale(category=LC_CTYPE) 返回给定类别的当前设置的一对字符串 (*lang, encoding*)。类别不能是 LC_ALL。 |

| localeconv | localeconv() 返回一个字典*d*，其中包含当前区域设置的 LC_NUMERIC 和 LC_MONETARY 类别指定的文化约定。虽然 LC_NUMERIC 最好间接使用，通过其他 locale 函数，但 LC_MONETARY 的细节只能通过*d*访问。本地和国际使用的货币格式不同。例如，'$'符号仅用于*本地*使用；在*国际*使用中是模糊的，因为相同的符号用于许多称为“dollars”的货币（美元、加拿大元、澳大利亚元、香港元等）。因此，在国际使用中，美元货币的符号是明确的字符串'USD'。该函数临时将 LC_CTYPE 区域设置为 LC_NUMERIC 区域设置，或者如果区域设置不同且数字或货币字符串为非 ASCII，则为 LC_MONETARY 区域设置。此临时更改影响所有线程。用于货币格式的*d*中的键是以下字符串：

'currency_symbol'

用于本地使用的货币符号

'frac_digits'

本地使用的小数位数

'int_curr_symbol'

用于国际使用的货币符号

'int_frac_digits'

用于国际使用的小数位数

'mon_decimal_point'

用作货币值的“小数点”（又称*基数点*）的字符串

'mon_grouping'

货币值的数字分组数字列表

'mon_thousands_sep'

用于货币值的数字组分隔符的字符串

'negative_sign', 'positive_sign'

用作负（正）货币值符号的字符串

'n_cs_precedes', 'p_cs_precedes'

**True** 当货币符号位于负（正）货币值之前时

'n_sep_by_space', 'p_sep_by_space'

**True** 当符号和负（正）货币值之间有空格时

'n_sign_posn', 'p_sign_posn'

请参阅 Table 11-37 查看格式化负（正）货币值的数字代码列表。

CHAR_MAX

表示当前区域设置不指定此格式的任何约定

|

| localeconv *(cont.)* | *d*['mon_grouping'] 是一个数字列表，用于在格式化货币值时分组数字（但要注意：在某些区域设置中，*d*['mon_grouping'] 可能是一个空列表）。当*d*['mon_grouping'][-1] 为 0 时，除了指定的数字之外，没有进一步的分组。当*d*['mon_grouping'][-1] 为 locale.CHAR_MAX 时，分组将无限继续，就像*d*['mon_grouping'][-2] 无限重复一样。locale.CHAR_MAX 是用于当前区域设置不指定任何约定的所有*d*条目的值的常量。 |
| --- | --- |
| localize | localize(*normstr*, grouping=**False**, monetary=**False**) 从规范化的数字字符串*normstr* 返回一个按照 LC_NUMERIC（或当 monetary 为**True**时，LC_MONETARY）设置的格式化字符串。 |
| normalize | normalize(*localename*) 返回一个字符串，适合作为 setlocale 的参数，即 *localename* 的规范化形式。当 normalize 无法规范化字符串 *localename* 时，返回不变的 *localename*。 |
| resetlocale | resetlocale(category=LC_ALL) 将类别 *category* 的区域设置为由 getdefaultlocale 给出的默认值。 |
| setlocale | setlocale(*category*, locale=**None**) 将类别 *category* 的区域设置为 locale，如果不是 **None**，则返回设置（当 locale 是 **None** 时返回现有设置；否则返回新设置）。 locale 可以是一个字符串，或者一个 (*lang*, *encoding*) 对。 *lang* 通常是基于 [ISO 639](https://oreil.ly/aT8Js) 两字母代码的语言代码（'en' 为英语，'nl' 为荷兰语等）。当 locale 是空字符串 '' 时，setlocale 设置用户的默认区域设置。要查看有效的区域设置，请查看 locale.locale_alias 字典。 |
| str | str(*num*) 类似于 locale.format_string('%f', *num*)。 |
| strcoll | strcoll(*str1*, *str2*) 在 LC_COLLATE 设置下，当 *str1* 在排序中排在 *str2* 之前时返回 -1，当 *str2* 在 *str1* 之前时返回 1，在排序目的上两个字符串相等时返回 0。 |

| strxfrm | strxfrm(*s*) 返回一个字符串 *sx*，使得 Python 对两个或多个经过此转换的字符串进行比较时类似于调用 locale.strcoll。 strxfrm 让你可以轻松地在需要区域设置兼容的排序和比较中使用键参数。例如，

```py
`def` locale_sort_inplace(list_of_strings):
    list_of_strings.sort(key=locale.strxfrm)
```

|

表 11-37\. 格式化货币值的数值代码

| 0 | 值和货币符号放在括号内 |
| --- | --- |
| 1 | 标记放在值和货币符号之前 |
| 2 | 标记放在值和货币符号之后 |
| 3 | 标记直接放在值之前 |
| 4 | 标记直接放在值之后 |

## gettext 模块

国际化的一个关键问题是能够在不同的自然语言中使用文本，这被称为*本地化*（有时是*l10n*）。 Python 通过标准库模块 gettext 支持本地化，受到 GNU gettext 的启发。 gettext 模块可以选择性地使用后者的基础设施和 API，但也提供了一种更简单、更高级的方法，因此你不需要安装或研究 GNU gettext 就能有效地使用 Python 的 gettext。

详细了解从不同角度覆盖 gettext，请参阅 [在线文档](https://oreil.ly/43fgn)。

### 使用 gettext 进行本地化

gettext 不涉及自然语言之间的自动翻译。 相反，它帮助您提取、组织和访问程序使用的文本消息。 将每个需要翻译的字符串文字（也称为*消息*）传递给一个名为 _（下划线）的函数，而不是直接使用它。 gettext 通常在内置模块中安装一个名为 _ 的函数。 为确保您的程序能够有或没有 gettext 运行，有条件地定义一个名为 _ 的无操作函数，它只是返回其未更改的参数。 然后，您可以安全地在需要翻译的文字使用 _*message*_ 的地方使用它。 以下示例显示了如何启动用于有条件使用 gettext 的模块：

```py
`try`:
    _
`except` NameError:
    `def` _(s): `return` s
`def` greet():
    print(_('Hello world'))
```

如果在您运行此示例代码之前某个其他模块已安装了 gettext，则函数 greet 会输出一个适当本地化的问候语。 否则，greet 输出未更改的字符串'Hello world'。

编辑您的源代码，使用函数 _ 装饰消息文字。 然后使用各种工具之一将消息提取到一个文本文件中（通常命名为*messages.pot*），并将该文件分发给负责将消息翻译为各种自然语言的人员。 Python 提供了一个脚本*pygettext.py*（位于 Python 源分发中的*Tools/i18n*目录中）来执行对您的 Python 源文件的消息提取。

每位翻译员编辑*messages.pot*以生成一个翻译消息的文本文件，扩展名为*.po.* 使用各种工具之一将*.po*文件编译成扩展名为*.mo*的二进制文件，适合快速搜索。 Python 提供了一个名为*msgfmt.py*（也在*Tools/i18n*中）的脚本用于此目的。 最后，在适当的目录中使用适当的名称安装每个*.mo*文件。

关于哪些目录和名称适合的约定在平台和应用程序之间有所不同。 gettext 的默认目录是目录*sys.prefix*下的子目录*share/locale/<lang>/LC_MESSAGES/*，其中*<lang>*是语言代码（两个字母）。 每个文件命名为*<name>.mo*，其中*<name>*是您的应用程序或软件包的名称。

一旦您准备好并安装了您的*.mo*文件，通常在应用程序启动时，您会执行以下类似的代码：

```py
`import` os, gettext
os.environ.setdefault('LANG', 'en')  *`# application-default`* *`language`*
gettext.install('your_application_name')
```

这确保像 _('message')这样的调用返回适当的翻译字符串。 您可以选择不同的方式在程序中访问 gettext 功能； 例如，如果您还需要本地化 C 编码的扩展，或者在运行期间切换语言。 另一个重要考虑因素是您是本地化整个应用程序还是仅分开分发的软件包。

### 重要的 gettext 函数

gettext 提供了许多功能。最常用的功能列在第 11-38 表中；请查看[在线文档](https://oreil.ly/Yk1yv)获取完整列表。

表 11-38\. gettext 模块的有用函数

| 安装 | 安装(*domain*, localedir=**None**, names=**None**) 在 Python 的内置命名空间中安装一个名为 _ 的函数，以在目录 localedir 中给定的文件<lang>/LC_MESSAGES/<domain>.mo 中执行翻译，使用语言代码<lang>按照 getdefaultlocale。当 localedir 为**None**时，install 使用目录 os.path.join(sys.prefix，'share'，'locale')。当提供 names 时，它必须是包含要在内置命名空间中安装的函数名称的序列，以外加 _。支持的名称有'gettext'，'lgettext'，'lngettext'，'ngettext'，3.8+'npgettext'和 3.8+'pgettext'。 |
| --- | --- |

| 翻译 | 翻译(*domain*, localedir=**None**, languages=**None**, class_=**None**, fallback=**False**) 搜索*.mo*文件，就像 install 函数一样；如果找到多个文件，则翻译将较晚的文件用作较早的文件的回退。将 fallback 设置为**True**以返回一个 NullTranslations 实例；否则，当找不到任何*.mo*文件时，函数会引发 OSError。

当 languages 为**None**时，翻译会在环境中查找要使用的<lang>，就像 install 一样。它按顺序检查环境变量 LANGUAGE、LC_ALL、LC_MESSAGES 和 LANG，并在第一个非空变量上用':'拆分以给出语言名称的列表（例如，它将'de:en'拆分为['de'，'en']）。当 languages 不为**None**时，languages 必须是一个包含一个或多个语言名称的列表（例如，['de'，'en']）。翻译使用列表中找到的第一个语言名称，该名称对应于找到的*.mo*文件。 |

| 翻译 *(cont.)* | 翻译返回一个翻译类的实例对象（默认为 GNUTranslations；如果存在，则类的构造函数必须接受单个文件对象参数），该对象提供了 gettext 方法（用于翻译 str）和 install 方法（将 gettext 安装在 Python 内置命名空间中的名称 _ 下）。翻译提供了比 install 更详细的控制，其功能类似于 translation(*domain*, *localedir*).install(*unicode*)。使用翻译，您可以在不影响内置命名空间的情况下，按模块绑定名称 _，以本地化单个包，例如：

```py
_ = translation(*`domain`*).ugettext
```

|

## 更多国际化资源

国际化是一个非常庞大的主题。有关一般介绍，请参见[Wikipedia](https://oreil.ly/raRbx)。国际化的最佳代码和信息包之一，作者乐意推荐的是[ICU](https://icu.unicode.org)，其中还包括 Unicode Consor 国际化是一个非常庞大的主题。有关一般介绍，请参见[Wikipedia](https://oreil.ly/raRbx)。国际化的最佳代码和信息包之一，作者乐意推荐的是[ICU](https://icu.unicode.org)，其中还包括 Unicode Consortium 的通用区域设置数据存储库（CLDR）数据库的区域设置约定和访问 CLDR 的代码。要在 Python 中使用 ICU，请安装第三方包[PyICU](https://oreil.ly/EkwTk)。

¹ 对于文本文件，tell 的值对于不定长字符是不透明的，因为它们包含可变长度的字符。对于二进制文件，它只是一个直接的字节计数。

² 遗憾的是，是的——不是 sys.stderr，如常见做法和逻辑所建议的那样！

³ 或者更好的是，更高级别的 pathlib 模块，在本章后面进行介绍。

⁴ “诅咒” 很好地描述了程序员面对这种复杂的低级别方法时的典型呻吟。

⁵ I18n 包括了“本地化”的过程，即将国际软件适应本地语言和文化习惯。
