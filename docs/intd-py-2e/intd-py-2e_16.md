# 第十四章：文件和目录。

> 我有文件，我有电脑文件，你知道的，在纸上也有文件。但大部分都在我脑子里。所以如果我的脑子出了问题，上帝帮帮我！
> 
> 乔治·R·R·马丁。

当你刚开始学习编程时，你会反复听到一些词，但不确定它们是否具有特定的技术含义还是随意的说法。*文件*和*目录*就是这样的词，它们确实有实际的技术含义。*文件*是一系列字节，存储在某个*文件系统*中，并通过*文件名*访问。*目录*是文件和可能其他目录的集合。术语*文件夹*是目录的同义词。它出现在计算机获得图形用户界面时，模仿办公室概念，使事物看起来更加熟悉。

许多文件系统是分层的，通常被称为类似于树。真实的办公室里不会有树，文件夹类比只有在你能够想象出所有子文件夹的情况下才有效。

# 文件的输入和输出。

最简单的持久性形式是普通的文件，有时称为*平面文件*。你从文件中*读取*到内存中，然后从内存中*写入*到文件中。Python 使得这些工作变得容易。与许多语言一样，它的文件操作在很大程度上是模仿熟悉且受欢迎的 Unix 等效操作。

## 使用`open()`创建或打开。

在执行以下操作之前，您需要调用`open`函数：

+   读取现有文件。

+   写入到一个新文件。

+   追加到现有文件。

+   覆盖现有文件。

```py
*`fileobj`* = open( *`filename`*, *`mode`* )
```

这里是对这个调用的各部分的简要解释：

+   *`fileobj`*是`open()`返回的文件对象。

+   *`filename`*是文件的字符串名称。

+   *`mode`*是一个表示文件类型及其操作的字符串。

*`mode`*的第一个字母表示操作：

+   `r`表示读取。

+   `w`表示写入。如果文件不存在，则创建该文件。如果文件存在，则覆盖它。

+   `x`表示写入，但只有在文件*不存在*时才会写入。

+   `a`表示追加（在末尾写入），如果文件存在。

*`mode`*的第二个字母表示文件的*类型*：

+   `t`（或什么都不写）表示文本。

+   `b`表示二进制。

打开文件后，您可以调用函数来读取或写入数据；这些将在接下来的示例中展示。

最后，您需要*关闭*文件以确保任何写入操作都已完成，并且内存已被释放。稍后，您将看到如何使用`with`来自动化此过程。

此程序打开一个名为*oops.txt*的文件，并在不写入任何内容的情况下关闭它。这将创建一个空文件：

```py
>>> fout = open('oops.txt', 'wt')
>>> fout.close()
```

## 使用`print()`写入文本文件。

让我们重新创建*oops.txt*，然后向其中写入一行内容，然后关闭它：

```py
>>> fout = open('oops.txt', 'wt')
>>> print('Oops, I created a file.', file=fout)
>>> fout.close()
```

我们在上一节创建了一个空的*oops.txt*文件，所以这只是覆盖它。

我们使用了`print`函数的`file`参数。如果没有这个参数，`print`会将内容写入*标准输出*，也就是你的终端（除非你已经告诉你的 shell 程序使用`>`重定向输出到文件或使用`|`管道传输到另一个程序）。

## 使用`write()`写入文本文件。

我们刚刚使用`print`向文件中写入了一行。我们也可以使用`write`。

对于我们的多行数据源，让我们使用这首关于狭义相对论的打油诗作为例子：¹

```py
>>> poem = '''There was a young lady named Bright,
... Whose speed was far faster than light;
... She started one day
... In a relative way,
... And returned on the previous night.'''
>>> len(poem)
150
```

下面的代码一次性将整首诗写入到名为`'relativity'`的文件中：

```py
>>> fout = open('relativity', 'wt')
>>> fout.write(poem)
150
>>> fout.close()
```

`write`函数返回写入的字节数。它不像`print`那样添加空格或换行符。同样，你也可以使用`print`将多行字符串写入文本文件：

```py
>>> fout = open('relativity', 'wt')
>>> print(poem, file=fout)
>>> fout.close()
```

那么，应该使用`write`还是`print`？正如你所见，默认情况下，`print`在每个参数后添加一个空格，并在末尾添加换行符。在前一个示例中，它向`relativity`文件附加了一个换行符。要使`print`像`write`一样工作，将以下两个参数传递给它：

+   `sep`（分隔符，默认为空格，`' '`）

+   `end`（结束字符串，默认为换行符，`'\n'`）

我们将使用空字符串来替换这些默认值：

```py
>>> fout = open('relativity', 'wt')
>>> print(poem, file=fout, sep='', end='')
>>> fout.close()
```

如果你有一个大的源字符串，你也可以写入分片（使用切片），直到源字符串处理完毕：

```py
>>> fout = open('relativity', 'wt')
>>> size = len(poem)
>>> offset = 0
>>> chunk = 100
>>> while True:
...     if offset > size:
...          break
...     fout.write(poem[offset:offset+chunk])
...     offset += chunk
...
100
50
>>> fout.close()
```

这一次在第一次尝试中写入了 100 个字符，下一次写入了最后 50 个字符。切片允许你“超过结尾”而不会引发异常。

如果对我们来说`relativity`文件很重要，让我们看看使用模式`x`是否真的保护我们免受覆盖：

```py
>>> fout = open('relativity', 'xt')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileExistsError: [Errno 17] File exists: 'relativity'
```

你可以将其与异常处理器一起使用：

```py
>>> try:
...     fout = open('relativity', 'xt')]
...     fout.write('stomp stomp stomp')
... except FileExistsError:
...     print('relativity already exists!. That was a close one.')
...
relativity already exists!. That was a close one.
```

## 使用`read()`、`readline()`或`readlines()`读取文本文件

你可以不带参数调用`read()`一次性读取整个文件，就像下面的示例一样（在处理大文件时要小心；一个 1GB 文件将消耗 1GB 内存）：

```py
>>> fin = open('relativity', 'rt' )
>>> poem = fin.read()
>>> fin.close()
>>> len(poem)
150
```

你可以提供一个最大字符数来限制`read()`一次返回多少内容。让我们一次读取 100 个字符，并将每个块追加到`poem`字符串以重建原始内容：

```py
>>> poem = ''
>>> fin = open('relativity', 'rt' )
>>> chunk = 100
>>> while True:
...     fragment = fin.read(chunk)
...     if not fragment:
...         break
...     poem += fragment
...
>>> fin.close()
>>> len(poem)
150
```

当你读取到结尾后，进一步调用`read()`会返回一个空字符串（`''`），这在`if not fragment`中被视作`False`。这会跳出`while True`循环。

你也可以使用`readline()`一次读取一行。在下一个示例中，我们将每一行追加到`poem`字符串中以重建原始内容：

```py
>>> poem = ''
>>> fin = open('relativity', 'rt' )
>>> while True:
...     line = fin.readline()
...     if not line:
...         break
...     poem += line
...
>>> fin.close()
>>> len(poem)
150
```

对于文本文件，即使是空行也有长度为一（换行符），并且被视作`True`。当文件被读取完毕时，`readline()`（和`read()`一样）也会返回一个空字符串，同样被视作`False`。

读取文本文件的最简单方法是使用*迭代器*。它一次返回一行。与前面的示例类似，但代码更少：

```py
>>> poem = ''
>>> fin = open('relativity', 'rt' )
>>> for line in fin:
...     poem += line
...
>>> fin.close()
>>> len(poem)
150
```

所有前述示例最终构建了单个字符串`poem`。`readlines()`方法逐行读取，返回一个包含每行字符串的列表：

```py
>>> fin = open('relativity', 'rt' )
>>> lines = fin.readlines()
>>> fin.close()
>>> print(len(lines), 'lines read')
5 lines read
>>> for line in lines:
...     print(line, end='')
...
There was a young lady named Bright,
Whose speed was far faster than light;
She started one day
In a relative way,
And returned on the previous night.>>>
```

我们告诉`print()`不要自动换行，因为前四行已经有了换行。最后一行没有换行，导致交互提示符`>>>`出现在最后一行之后。

## 使用`write()`写入二进制文件

如果在*模式*字符串中包含 `'b'`，文件将以二进制模式打开。在这种情况下，你读取和写入的是 `bytes` 而不是字符串。

我们手头没有二进制诗歌，所以我们只会生成从 0 到 255 的 256 个字节值：

```py
>>> bdata = bytes(range(0, 256))
>>> len(bdata)
256
```

以二进制模式打开文件进行写入，并一次性写入所有数据：

```py
>>> fout = open('bfile', 'wb')
>>> fout.write(bdata)
256
>>> fout.close()
```

同样，`write()` 返回写入的字节数。

和文本一样，你可以将二进制数据分块写入：

```py
>>> fout = open('bfile', 'wb')
>>> size = len(bdata)
>>> offset = 0
>>> chunk = 100
>>> while True:
...     if offset > size:
...          break
...     fout.write(bdata[offset:offset+chunk])
...     offset += chunk
...
100
100
56
>>> fout.close()
```

## 使用 `read()` 读取二进制文件

这个很简单；你只需要用 `'rb'` 打开即可：

```py
>>> fin = open('bfile', 'rb')
>>> bdata = fin.read()
>>> len(bdata)
256
>>> fin.close()
```

## 使用 `with` 自动关闭文件

如果你忘记关闭已打开的文件，在不再引用后 Python 会关闭它。这意味着如果你在函数中打开文件但没有显式关闭它，在函数结束时文件会被自动关闭。但你可能在长时间运行的函数或程序的主要部分中打开了文件。应该关闭文件以确保所有未完成的写入被完成。

Python 有*上下文管理器*来清理诸如打开的文件之类的资源。你可以使用形式 `with` *`表达式`* `as` *`变量`*：

```py
>>> with open('relativity', 'wt') as fout:
...     fout.write(poem)
...
```

就是这样。在上下文管理器（在本例中就是一行代码块）完成（正常完成 *或* 通过抛出异常）后，文件会自动关闭。

## 使用 `seek()` 改变位置

当你读取和写入时，Python 会跟踪你在文件中的位置。`tell()` 函数返回你当前从文件开头的偏移量，以字节为单位。`seek()` 函数让你跳到文件中的另一个字节偏移量。这意味着你不必读取文件中的每个字节来读取最后一个字节；你可以 `seek()` 到最后一个字节并只读取一个字节。

对于这个示例，使用你之前写的 256 字节二进制文件 `'bfile'`：

```py
>>> fin = open('bfile', 'rb')
>>> fin.tell()
0
```

使用 `seek()` 跳转到文件末尾前一个字节：

```py
>>> fin.seek(255)
255
```

读取直到文件末尾：

```py
>>> bdata = fin.read()
>>> len(bdata)
1
>>> bdata[0]
255
```

`seek()` 也会返回当前偏移量。

你可以给 `seek()` 调用一个第二参数：``seek(*`offset`*, *`origin`*)``：

+   如果 `origin` 是 `0`（默认值），就从文件开头向后 *`offset`* 字节

+   如果 `origin` 是 `1`，就从当前位置向后 *`offset`* 字节

+   如果 `origin` 是 `2`，就从文件末尾相对 *`offset`* 字节

这些值也在标准的 `os` 模块中定义：

```py
>>> import os
>>> os.SEEK_SET
0
>>> os.SEEK_CUR
1
>>> os.SEEK_END
2
```

因此，我们可以用不同的方式读取最后一个字节：

```py
>>> fin = open('bfile', 'rb')
```

文件末尾前一个字节：

```py
>>> fin.seek(-1, 2)
255
>>> fin.tell()
255
```

读取直到文件末尾：

```py
>>> bdata = fin.read()
>>> len(bdata)
1
>>> bdata[0]
255
```

###### 注意

你不需要调用 `tell()` 来让 `seek()` 工作。我只是想展示它们报告相同的偏移量。

下面是从文件当前位置进行搜索的示例：

```py
>>> fin = open('bfile', 'rb')
```

这个示例最终会在文件末尾前两个字节处结束：

```py
>>> fin.seek(254, 0)
254
>>> fin.tell()
254
```

现在向前移动一个字节：

```py
>>> fin.seek(1, 1)
255
>>> fin.tell()
255
```

最后，读取直到文件末尾：

```py
>>> bdata = fin.read()
>>> len(bdata)
1
>>> bdata[0]
255
```

这些函数对于二进制文件最有用。你可以用它们处理文本文件，但除非文件是 ASCII（每个字符一个字节），否则计算偏移量会很困难。这将取决于文本编码，而最流行的编码（UTF-8）使用不同数量的字节表示每个字符。

# 内存映射

读取和写入文件的替代方法是使用标准`mmap`模块将其*内存映射*。 这使得文件内容在内存中看起来像一个`bytearray`。 有关详细信息，请参阅[文档](https://oreil.ly/GEzkf)和一些[示例](https://oreil.ly/GUtdx)。

# 文件操作

Python，像许多其他语言一样，根据 Unix 模式化其文件操作。 一些函数，例如`chown()`和`chmod()`，具有相同的名称，但还有一些新函数。

首先，我将展示 Python 如何使用`os.path`模块的函数以及使用较新的`pathlib`模块处理这些任务。

## 使用`exists()`检查存在性。

要验证文件或目录是否确实存在，或者您只是想象了它，您可以提供`exists()`，并提供相对或绝对路径名，如此示例所示：

```py
>>> import os
>>> os.path.exists('oops.txt')
True
>>> os.path.exists('./oops.txt')
True
>>> os.path.exists('waffles')
False
>>> os.path.exists('.')
True
>>> os.path.exists('..')
True
```

## 使用`isfile()`检查类型。

此部分中的函数检查名称是否引用文件、目录或符号链接（有关链接讨论的示例，请参见后续内容）。

我们将首先查看的第一个函数`isfile`，它提出一个简单的问题：这是一个普通的老实文件吗？

```py
>>> name = 'oops.txt'
>>> os.path.isfile(name)
True
```

下面是确定目录的方法：

```py
>>> os.path.isdir(name)
False
```

单个点(`.`)是当前目录的简写，两个点(`..`)代表父目录。 这些始终存在，因此像以下语句将始终报告`True`：

```py
>>> os.path.isdir('.')
True
```

`os`模块包含许多处理*路径名*（完全合格的文件名，以`/`开头并包括所有父级）的函数。 其中一个函数`isabs()`确定其参数是否为绝对路径名。 参数不需要是真实文件的名称：

```py
>>> os.path.isabs(name)
False
>>> os.path.isabs('/big/fake/name')
True
>>> os.path.isabs('big/fake/name/without/a/leading/slash')
False
```

## 使用`copy()`复制。

`copy()`函数来自另一个模块`shutil`。 例如，将文件*oops.txt*复制到文件*ohno.txt*：

```py
>>> import shutil
>>> shutil.copy('oops.txt', 'ohno.txt')
```

`shutil.move()`函数复制文件，然后删除原始文件。

## 使用`rename()`函数更改名称。

此函数正如其名称所示。 在此示例中，它将*ohno.txt*重命名为*ohwell.txt*：

```py
>>> import os
>>> os.rename('ohno.txt', 'ohwell.txt')
```

## 使用`link()`或`symlink()`链接。

在 Unix 中，文件存在于一个位置，但可以有多个名称，称为*链接*。 在低级*硬链接*中，很难找到给定文件的所有名称。 *符号链接*是一种替代方法，它将新名称存储为自己的文件，使您可以同时获取原始名称和新名称。 `link()`调用创建硬链接，`symlink()`创建符号链接。 `islink()`函数检查文件是否是符号链接。

下面是如何为现有文件*oops.txt*创建硬链接到新文件*yikes.txt*的方法：

```py
>>> os.link('oops.txt', 'yikes.txt')
>>> os.path.isfile('yikes.txt')
True
>>> os.path.islink('yikes.txt')
False
```

要为现有文件*oops.txt*创建到新文件*jeepers.txt*的符号链接，请使用以下命令：

```py
>>> os.symlink('oops.txt', 'jeepers.txt')
>>> os.path.islink('jeepers.txt')
True
```

## 使用`chmod()`更改权限。

在 Unix 系统中，`chmod()` 改变文件权限。对于用户（通常是你，如果你创建了该文件）、用户所在的主要组和其余世界，有读、写和执行权限。该命令使用紧凑的八进制（基数 8）值，结合用户、组和其他权限。例如，要使 *oops.txt* 只能由其所有者读取，输入以下内容：

```py
>>> os.chmod('oops.txt', 0o400)
```

如果你不想处理晦涩的八进制值，而宁愿处理（稍微不那么）晦涩的符号，可以从 `stat` 模块导入一些常量，并使用如下语句：

```py
>>> import stat
>>> os.chmod('oops.txt', stat.S_IRUSR)
```

## 使用 chown() 更改所有权

这个函数同样适用于 Unix/Linux/Mac。你可以通过指定数值用户 ID (*uid*) 和组 ID (*gid*) 来改变文件的所有者和/或组所有权：

```py
>>> uid = 5
>>> gid = 22
>>> os.chown('oops', uid, gid)
```

## 使用 remove() 删除文件

在这段代码中，我们使用 `remove()` 函数，告别 *oops.txt*：

```py
>>> os.remove('oops.txt')
>>> os.path.exists('oops.txt')
False
```

# 目录操作

在大多数操作系统中，文件存在于层级结构的*目录*（通常称为*文件夹*）中。所有这些文件和目录的容器是一个*文件系统*（有时称为*卷*）。标准的 `os` 模块处理这些操作系统的具体细节，并提供以下函数，用于对它们进行操作。

## 使用 mkdir() 创建

此示例展示了如何创建一个名为 `poems` 的目录来存储那些珍贵的诗句：

```py
>>> os.mkdir('poems')
>>> os.path.exists('poems')
True
```

## 使用 rmdir() 删除目录

经过重新考虑²，你决定其实根本不需要那个目录。以下是如何删除它的方法：

```py
>>> os.rmdir('poems')
>>> os.path.exists('poems')
False
```

## 使用 listdir() 列出内容

好的，重来一次；让我们再次创建 `poems`，并添加一些内容：

```py
>>> os.mkdir('poems')
```

现在获取其内容列表（到目前为止还没有）：

```py
>>> os.listdir('poems')
[]
```

接下来，创建一个子目录：

```py
>>> os.mkdir('poems/mcintyre')
>>> os.listdir('poems')
['mcintyre']
```

在这个子目录中创建一个文件（如果你真的感觉有诗意，才输入所有这些行；确保使用匹配的单引号或三重引号开头和结尾）：

```py
>>> fout = open('poems/mcintyre/the_good_man', 'wt')
>>> fout.write('''Cheerful and happy was his mood,
... He to the poor was kind and good,
... And he oft' times did find them food,
... Also supplies of coal and wood,
... He never spake a word was rude,
... And cheer'd those did o'er sorrows brood,
... He passed away not understood,
... Because no poet in his lays
... Had penned a sonnet in his praise,
... 'Tis sad, but such is world's ways.
... ''')
344
>>> fout.close()
```

最后，让我们看看我们有什么。它最好在那里： 

```py
>>> os.listdir('poems/mcintyre')
['the_good_man']
```

## 使用 chdir() 更改当前目录

使用此函数，你可以从一个目录切换到另一个目录。让我们离开当前目录，花一点时间在 `poems` 中：

```py
>>> import os
>>> os.chdir('poems')
>>> os.listdir('.')
['mcintyre']
```

## 使用 glob() 列出匹配的文件

`glob()` 函数使用 Unix shell 规则而非更完整的正则表达式语法来匹配文件或目录名。以下是这些规则：

+   `*` 匹配任何内容（`re` 应该期望 `.*`）

+   `?` 匹配一个单字符

+   `[abc]` 匹配字符 `a`、`b` 或 `c`

+   `[!abc]` 匹配除了 `a`、`b` 或 `c` 之外的任何字符

尝试获取所有以 `m` 开头的文件或目录：

```py
>>> import glob
>>> glob.glob('m*')
['mcintyre']
```

任何两个字母的文件或目录如何？

```py
>>> glob.glob('??')
[]
```

我在想一个以 `m` 开头、以 `e` 结尾的八个字母的单词：

```py
>>> glob.glob('m??????e')
['mcintyre']
```

那么任何以 `k`、`l` 或 `m` 开头、以 `e` 结尾的内容呢？

```py
>>> glob.glob('[klm]*e')
['mcintyre']
```

# 路径名

几乎所有的计算机都使用层次化文件系统，其中目录（“文件夹”）包含文件和其他目录，向下延伸到不同的层级。当您想引用特定的文件或目录时，您需要它的 *路径名*：到达那里所需的目录序列，可以是 *绝对* 从顶部（*根*）或 *相对* 到您当前目录。

当您指定名称时，您经常会听到人们混淆正斜杠（`'/'`，而不是 Guns N’ Roses 的家伙）和反斜杠（`'\'`）。³ Unix 和 Mac（以及 Web URL）使用正斜杠作为 *路径分隔符*，而 Windows 使用反斜杠。⁴

Python 允许您在指定名称时使用斜杠作为路径分隔符。在 Windows 上，您可以使用反斜杠，但是您知道反斜杠在 Python 中是普遍的转义字符，所以您必须在所有地方加倍使用它，或者使用 Python 的原始字符串：

```py
>>> win_file = 'eek\\urk\\snort.txt'
>>> win_file2 = r'eek\urk\snort.txt'
>>> win_file
'eek\\urk\\snort.txt'
>>> win_file2
'eek\\urk\\snort.txt'
```

当您构建路径名时，您可以做以下操作：

+   使用适当的路径分隔符 (`'/'` 或 `'\'`)

+   使用 os.path.join() 构建路径名（参见 “使用 os.path.join() 构建路径名”）

+   使用 pathlib（参见 “使用 pathlib”）

## 通过 abspath() 获取路径名

这个函数将相对名扩展为绝对名。如果您的当前目录是 */usr/gaberlunzie* 并且文件 *oops.txt* 就在那里，您可以输入以下内容：

```py
>>> os.path.abspath('oops.txt')
'/usr/gaberlunzie/oops.txt'
```

## 使用 realpath() 获取符号链接路径名

在较早的某一部分中，我们从新文件 *jeepers.txt* 创造了对 *oops.txt* 的符号链接。在这种情况下，您可以使用 `realpath()` 函数从 *jeepers.txt* 获取 *oops.txt* 的名称，如下所示：

```py
>>> os.path.realpath('jeepers.txt')
'/usr/gaberlunzie/oops.txt'
```

## 使用 os.path.join() 构建路径名

当您构建一个多部分的路径名时，您可以使用 `os.path.join()` 将它们成对地组合，使用适合您操作系统的正确路径分隔符：

```py
>>> import os
>>> win_file = os.path.join("eek", "urk")
>>> win_file = os.path.join(win_file, "snort.txt")
```

如果我在 Mac 或 Linux 系统上运行这个程序，我会得到这个结果：

```py
>>> win_file
'eek/urk/snort.txt'
```

在 Windows 上运行会产生这个结果：

```py
>>> win_file
'eek\\urk\\snort.txt'
```

但是如果相同的代码在不同位置运行会产生不同的结果，这可能是一个问题。新的 `pathlib` 模块是这个问题的一个便携解决方案。

## 使用 pathlib

Python 在版本 3.4 中添加了 `pathlib` 模块。它是我刚刚描述的 `os.path` 模块的一个替代方案。但我们为什么需要另一个模块呢？

它不是把文件系统路径名当作字符串，而是引入了 `Path` 对象来在稍高一级处理它们。使用 `Path()` 类创建一个 `Path`，然后用裸斜线（而不是 `'/'` 字符）将您的路径编织在一起：

```py
>>> from pathlib import Path
>>> file_path = Path('eek') / 'urk' / 'snort.txt'
>>> file_path
PosixPath('eek/urk/snort.txt')
>>> print(file_path)
eek/urk/snort.txt
```

这个斜杠技巧利用了 Python 的 “魔术方法”。一个 `Path` 可以告诉您关于自己的一些信息：

```py
>>> file_path.name
'snort.txt'
>>> file_path.suffix
'.txt'
>>> file_path.stem
'snort'
```

您可以像对待任何文件名或路径名字符串一样将 `file_path` 提供给 `open()`。

您还可以看到如果在另一个系统上运行此程序会发生什么，或者如果需要在您的计算机上生成外国路径名：

```py
>>> from pathlib import PureWindowsPath
>>> PureWindowsPath(file_path)
PureWindowsPath('eek/urk/snort.txt')
>>> print(PureWindowsPath(file_path))
eek\urk\snort.txt
```

参见[文档](https://oreil.ly/yN87f)以获取所有细节。

# BytesIO 和 StringIO

你已经学会了如何修改内存中的数据以及如何将数据读取到文件中和从文件中获取数据。如果你有内存中的数据，但想调用一个期望文件的函数（或者反过来），你会怎么做？你想修改数据并传递这些字节或字符，而不是读取和写入临时文件。

你可以使用 `io.BytesIO` 处理二进制数据（`bytes`）和 `io.StringIO` 处理文本数据（`str`）。使用其中任何一个都可以将数据包装为*类文件对象*，适用于本章中介绍的所有文件函数。

这种情况的一个用例是数据格式转换。让我们将其应用于 PIL 库（详细信息将在“PIL 和 Pillow”中介绍），该库读取和写入图像数据。其 `Image` 对象的 `open()` 和 `save()` 方法的第一个参数是文件名*或*类文件对象。示例 14-1 中的代码使用 `BytesIO` 在内存中读取 *并且* 写入数据。它从命令行读取一个或多个图像文件，将其图像数据转换为三种不同的格式，并打印这些输出的长度和前 10 个字节。

##### 示例 14-1\. convert_image.py

```py
from io import BytesIO
from PIL import Image
import sys

def data_to_img(data):
    """Return PIL Image object, with data from in-memory <data>"""
    fp = BytesIO(data)
    return Image.open(fp)    # reads from memory

def img_to_data(img, fmt=None):
    """Return image data from PIL Image <img>, in <fmt> format"""
    fp = BytesIO()
    if not fmt:
        fmt = img.format     # keeps the original format
    img.save(fp, fmt)        # writes to memory
    return fp.getvalue()

def convert_image(data, fmt=None):
    """Convert image <data> to PIL <fmt> image data"""
    img = data_to_img(data)
    return img_to_data(img, fmt)

def get_file_data(name):
    """Return PIL Image object for image file <name>"""
    img = Image.open(name)
    print("img", img, img.format)
    return img_to_data(img)

if __name__ == "__main__":
    for name in sys.argv[1:]:
        data = get_file_data(name)
        print("in", len(data), data[:10])
        for fmt in ("gif", "png", "jpeg"):
            out_data = convert_image(data, fmt)
            print("out", len(out_data), out_data[:10])
```

###### 注意

因为它的行为类似于文件，所以你可以像处理普通文件一样使用 `seek()`、`read()` 和 `write()` 方法处理 `BytesIO` 对象；如果你执行了 `seek()` 后跟着一个 `read()`，你将只获得从该 seek 位置到结尾的字节。`getvalue()` 返回 `BytesIO` 对象中的所有字节。

这是输出结果，使用了你将在第二十章中看到的输入图像文件：

```py
$ python convert_image.py ch20_critter.png
img <PIL.PngImagePlugin.PngImageFile image mode=RGB size=154x141 at 0x10340CF28> PNG
in 24941 b'\\x89PNG\\r\\n\\x1a\\n\\x00\\x00'
out 14751 b'GIF87a\\x9a\\x00\\x8d\\x00'
out 24941 b'\\x89PNG\\r\\n\\x1a\\n\\x00\\x00'
out 5914 b'\\xff\xd8\\xff\\xe0\\x00\\x10JFIF'

```

# 即将到来

下一章内容稍微复杂一些。它涉及*并发*（即大约同时执行多个任务的方式）和*进程*（运行程序）。

# 要做的事情

14.1 列出当前目录中的文件。

14.2 列出父目录中的文件。

14.3 将字符串 `'This is a test of the emergency text system'` 赋值给变量 `test1`，并将 `test1` 写入名为 *test.txt* 的文件。

14.4 打开文件 *test.txt* 并将其内容读取到字符串 `test2` 中。`test1` 和 `test2` 是否相同？

¹ 在本书的第一份手稿中，我说的是*广义*相对论，被一位物理学家审阅者友善地纠正了。

² 为什么它从来都不是第一个？

³ 一种记忆方法是：斜杠向*前*倾斜，反斜杠向*后*倾斜。

⁴ 当 IBM 联系比尔·盖茨，询问他们的第一台个人电脑时，他以 $50,000 购买了操作系统 QDOS，以获得“MS-DOS”。它模仿了使用斜杠作为命令行参数的 CP/M。当 MS-DOS 后来添加了文件夹时，它不得不使用反斜杠。
