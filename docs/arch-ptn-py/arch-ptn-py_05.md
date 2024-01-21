# 第三章：简短插曲：耦合和抽象

> 原文：[3: A Brief Interlude: On Coupling and Abstractions](https://www.cosmicpython.com/book/chapter_03_abstractions.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

请允许我们在抽象主题上稍作偏离，亲爱的读者。我们已经谈论了*抽象*很多。例如，存储库模式是对永久存储的抽象。但是什么样的抽象才是好的？我们从抽象中想要什么？它们与测试有什么关系？

###### 提示

本章的代码在 GitHub 的 chapter_03_abstractions 分支中[链接](https://oreil.ly/k6MmV)：

```py
git clone https://github.com/cosmicpython/code.git
git checkout chapter_03_abstractions
```

本书的一个关键主题，隐藏在花哨的模式中，是我们可以使用简单的抽象来隐藏混乱的细节。当我们写代码时，可以自由地玩耍，或者在一个 kata 中，我们可以自由地玩耍，大幅度地重构。然而，在大规模系统中，我们受到系统其他地方做出的决定的限制。

当我们因为担心破坏组件 B 而无法更改组件 A 时，我们说这些组件已经*耦合*。在本地，耦合是一件好事：这表明我们的代码正在一起工作，每个组件都在支持其他组件，所有这些组件都像手表的齿轮一样完美地配合在一起。在行话中，我们说当耦合元素之间存在高*内聚*时，这是有效的。

总体上，耦合是一种麻烦事：它增加了更改代码的风险和成本，有时甚至到了我们感到无法进行任何更改的地步。这就是“泥球”模式的问题：随着应用程序的增长，如果我们无法阻止没有内聚力的元素之间的耦合，那么耦合会超线性地增加，直到我们无法有效地更改我们的系统。

我们可以通过将细节抽象化（图 3-1）来减少系统内的耦合程度。

![apwp 0301](img/apwp_0301.png)

###### 图 3-1：耦合很多

```py
[ditaa,apwp_0301]
+--------+      +--------+
| System | ---> | System |
|   A    | ---> |   B    |
|        | ---> |        |
|        | ---> |        |
|        | ---> |        |
+--------+      +--------+
```

![apwp 0302](img/apwp_0302.png)

###### 图 3-2：耦合减少

```py
[ditaa,apwp_0302]
+--------+                           +--------+
| System |      /-------------\      | System |
|   A    | ---> |             | ---> |   B    |
|        | ---> | Abstraction | ---> |        |
|        |      |             | ---> |        |
|        |      \-------------/      |        |
+--------+                           +--------+
```

在这两个图中，我们有一对子系统，其中一个依赖于另一个。在图 3-1 中，两者之间存在高度耦合；箭头的数量表示两者之间有很多种依赖关系。如果我们需要更改系统 B，很可能这种更改会传播到系统 A。

然而，在图 3-2 中，我们通过插入一个新的、更简单的抽象来减少耦合程度。因为它更简单，系统 A 对抽象的依赖种类更少。抽象用于保护我们免受变化的影响，它隐藏了系统 B 的复杂细节——我们可以在不更改左侧箭头的情况下更改右侧的箭头。

# 抽象状态辅助测试

让我们看一个例子。想象一下，我们想要编写代码来同步两个文件目录，我们将其称为*源*和*目标*：

+   如果源中存在文件，但目标中不存在文件，则复制文件。

+   如果源文件存在，但与目标文件名不同，则将目标文件重命名为匹配的文件。

+   如果目标中存在文件，但源中不存在文件，则删除它。

我们的第一个和第三个要求都很简单：我们只需比较两个路径列表。但是我们的第二个要求就比较棘手了。为了检测重命名，我们将不得不检查文件的内容。为此，我们可以使用 MD5 或 SHA-1 等哈希函数。从文件生成 SHA-1 哈希的代码非常简单：

*对文件进行哈希（`sync.py`）*

```py
BLOCKSIZE = 65536

def hash_file(path):
    hasher = hashlib.sha1()
    with path.open("rb") as file:
        buf = file.read(BLOCKSIZE)
        while buf:
            hasher.update(buf)
            buf = file.read(BLOCKSIZE)
    return hasher.hexdigest()
```

现在我们需要编写决定要做什么的部分——商业逻辑，如果你愿意的话。

当我们必须从第一原则解决问题时，通常我们会尝试编写一个简单的实现，然后朝着更好的设计进行重构。我们将在整本书中使用这种方法，因为这是我们在现实世界中编写代码的方式：从问题的最小部分开始解决方案，然后迭代地使解决方案更加丰富和更好设计。

我们的第一个 hackish 方法看起来是这样的：

*基本同步算法（`sync.py`）*

```py
import hashlib
import os
import shutil
from pathlib import Path

def sync(source, dest):
    # Walk the source folder and build a dict of filenames and their hashes
    source_hashes = {}
    for folder, _, files in os.walk(source):
        for fn in files:
            source_hashes[hash_file(Path(folder) / fn)] = fn

    seen = set()  # Keep track of the files we've found in the target

    # Walk the target folder and get the filenames and hashes
    for folder, _, files in os.walk(dest):
        for fn in files:
            dest_path = Path(folder) / fn
            dest_hash = hash_file(dest_path)
            seen.add(dest_hash)

            # if there's a file in target that's not in source, delete it
            if dest_hash not in source_hashes:
                dest_path.remove()

            # if there's a file in target that has a different path in source,
            # move it to the correct path
            elif dest_hash in source_hashes and fn != source_hashes[dest_hash]:
                shutil.move(dest_path, Path(folder) / source_hashes[dest_hash])

    # for every file that appears in source but not target, copy the file to
    # the target
    for src_hash, fn in source_hashes.items():
        if src_hash not in seen:
            shutil.copy(Path(source) / fn, Path(dest) / fn)
```

太棒了！我们有一些代码，它*看起来*不错，但在我们在硬盘上运行它之前，也许我们应该测试一下。我们如何测试这种类型的东西？

*一些端到端测试（`test_sync.py`）*

```py
def test_when_a_file_exists_in_the_source_but_not_the_destination():
    try:
        source = tempfile.mkdtemp()
        dest = tempfile.mkdtemp()

        content = "I am a very useful file"
        (Path(source) / 'my-file').write_text(content)

        sync(source, dest)

        expected_path = Path(dest) /  'my-file'
        assert expected_path.exists()
        assert expected_path.read_text() == content

    finally:
        shutil.rmtree(source)
        shutil.rmtree(dest)

def test_when_a_file_has_been_renamed_in_the_source():
    try:
        source = tempfile.mkdtemp()
        dest = tempfile.mkdtemp()

        content = "I am a file that was renamed"
        source_path = Path(source) / 'source-filename'
        old_dest_path = Path(dest) / 'dest-filename'
        expected_dest_path = Path(dest) / 'source-filename'
        source_path.write_text(content)
        old_dest_path.write_text(content)

        sync(source, dest)

        assert old_dest_path.exists() is False
        assert expected_dest_path.read_text() == content

    finally:
        shutil.rmtree(source)
        shutil.rmtree(dest)
```

哇哦，为了两个简单的情况需要做很多设置！问题在于我们的领域逻辑，“找出两个目录之间的差异”，与 I/O 代码紧密耦合。我们无法在不调用`pathlib`、`shutil`和`hashlib`模块的情况下运行我们的差异算法。

问题是，即使在当前的要求下，我们还没有编写足够的测试：当前的实现存在一些错误（例如`shutil.move()`是错误的）。获得良好的覆盖率并揭示这些错误意味着编写更多的测试，但如果它们都像前面的测试一样难以管理，那将会变得非常痛苦。

除此之外，我们的代码并不是很可扩展。想象一下，试图实现一个`--dry-run`标志，让我们的代码只是打印出它将要做的事情，而不是实际去做。或者，如果我们想要同步到远程服务器或云存储呢？

我们的高级代码与低级细节耦合在一起，这让生活变得困难。随着我们考虑的场景变得更加复杂，我们的测试将变得更加难以管理。我们肯定可以重构这些测试（例如，一些清理工作可以放入 pytest fixtures 中），但只要我们进行文件系统操作，它们就会保持缓慢，并且难以阅读和编写。

# 选择正确的抽象

我们能做些什么来重写我们的代码，使其更具可测试性？

首先，我们需要考虑我们的代码需要文件系统提供什么。通过阅读代码，我们可以看到三个不同的事情正在发生。我们可以将这些视为代码具有的三个不同*责任*：

1.  我们通过使用`os.walk`来询问文件系统，并为一系列路径确定哈希值。这在源和目标情况下都是相似的。

1.  我们决定文件是新的、重命名的还是多余的。

1.  我们复制、移动或删除文件以匹配源。

记住，我们希望为这些责任中的每一个找到*简化的抽象*。这将让我们隐藏混乱的细节，以便我们可以专注于有趣的逻辑。²

###### 注意

在本章中，我们通过识别需要完成的各个任务并将每个任务分配给一个明确定义的执行者，按照类似于[“duckduckgo”示例](preface02.xhtml#ddg_example)的方式，将一些混乱的代码重构为更具可测试性的结构。

对于步骤 1 和步骤 2，我们已经直观地开始使用一个抽象，即哈希到路径的字典。您可能已经在想，“为什么不为目标文件夹构建一个字典，然后我们只需比较两个字典？”这似乎是一种很好的抽象文件系统当前状态的方式：

```py
source_files = {'hash1': 'path1', 'hash2': 'path2'}
dest_files = {'hash1': 'path1', 'hash2': 'pathX'}
```

那么，我们如何从第 2 步移动到第 3 步呢？我们如何将实际的移动/复制/删除文件系统交互抽象出来？

我们将在这里应用一个技巧，我们将在本书的后面大规模地使用。我们将*想要做什么*与*如何做*分开。我们将使我们的程序输出一个看起来像这样的命令列表：

```py
("COPY", "sourcepath", "destpath"),
("MOVE", "old", "new"),
```

现在我们可以编写测试，只使用两个文件系统字典作为输入，并且我们期望输出表示操作的字符串元组列表。

我们不是说，“给定这个实际的文件系统，当我运行我的函数时，检查发生了什么操作”，而是说，“给定这个*文件系统的抽象*，会发生什么*文件系统操作的抽象*？”

*我们的测试中简化了输入和输出（`test_sync.py`）*

```py
    def test_when_a_file_exists_in_the_source_but_not_the_destination():
        src_hashes = {'hash1': 'fn1'}
        dst_hashes = {}
        expected_actions = [('COPY', '/src/fn1', '/dst/fn1')]
        ...

    def test_when_a_file_has_been_renamed_in_the_source():
        src_hashes = {'hash1': 'fn1'}
        dst_hashes = {'hash1': 'fn2'}
        expected_actions == [('MOVE', '/dst/fn2', '/dst/fn1')]
        ...
```

# 实现我们选择的抽象

这一切都很好，但我们*实际上*如何编写这些新测试，以及如何改变我们的实现使其正常工作？

我们的目标是隔离系统的聪明部分，并且能够彻底测试它，而不需要设置真实的文件系统。我们将创建一个“核心”代码，它不依赖外部状态，然后看看当我们从外部世界输入时它如何响应（这种方法被 Gary Bernhardt 称为[Functional Core, Imperative Shell](https://oreil.ly/wnad4)，或 FCIS）。

让我们从将代码分离出有状态的部分和逻辑开始。

我们的顶层函数几乎不包含任何逻辑；它只是一系列命令式的步骤：收集输入，调用我们的逻辑，应用输出：

*将我们的代码分成三部分（`sync.py`）*

```py
def sync(source, dest):
    # imperative shell step 1, gather inputs
    source_hashes = read_paths_and_hashes(source)  #(1)
    dest_hashes = read_paths_and_hashes(dest)  #(1)

    # step 2: call functional core
    actions = determine_actions(source_hashes, dest_hashes, source, dest)  #(2)

    # imperative shell step 3, apply outputs
    for action, *paths in actions:
        if action == "COPY":
            shutil.copyfile(*paths)
        if action == "MOVE":
            shutil.move(*paths)
        if action == "DELETE":
            os.remove(paths[0])
```

①

这是我们分解的第一个函数，`read_paths_and_hashes()`，它隔离了我们应用程序的 I/O 部分。

②

这是我们切出功能核心，业务逻辑的地方。

构建路径和哈希字典的代码现在非常容易编写：

*只执行 I/O 的函数（`sync.py`）*

```py
def read_paths_and_hashes(root):
    hashes = {}
    for folder, _, files in os.walk(root):
        for fn in files:
            hashes[hash_file(Path(folder) / fn)] = fn
    return hashes
```

`determine_actions()`函数将是我们业务逻辑的核心，它说：“给定这两组哈希和文件名，我们应该复制/移动/删除什么？”。它接受简单的数据结构并返回简单的数据结构：

*只执行业务逻辑的函数（`sync.py`）*

```py
def determine_actions(src_hashes, dst_hashes, src_folder, dst_folder):
    for sha, filename in src_hashes.items():
        if sha not in dst_hashes:
            sourcepath = Path(src_folder) / filename
            destpath = Path(dst_folder) / filename
            yield 'copy', sourcepath, destpath

        elif dst_hashes[sha] != filename:
            olddestpath = Path(dst_folder) / dst_hashes[sha]
            newdestpath = Path(dst_folder) / filename
            yield 'move', olddestpath, newdestpath

    for sha, filename in dst_hashes.items():
        if sha not in src_hashes:
            yield 'delete', dst_folder / filename
```

我们的测试现在直接作用于`determine_actions()`函数：

*更好看的测试（`test_sync.py`）*

```py
def test_when_a_file_exists_in_the_source_but_not_the_destination():
    src_hashes = {'hash1': 'fn1'}
    dst_hashes = {}
    actions = determine_actions(src_hashes, dst_hashes, Path('/src'), Path('/dst'))
    assert list(actions) == [('copy', Path('/src/fn1'), Path('/dst/fn1'))]
...

def test_when_a_file_has_been_renamed_in_the_source():
    src_hashes = {'hash1': 'fn1'}
    dst_hashes = {'hash1': 'fn2'}
    actions = determine_actions(src_hashes, dst_hashes, Path('/src'), Path('/dst'))
    assert list(actions) == [('move', Path('/dst/fn2'), Path('/dst/fn1'))]
```

因为我们已经将程序的逻辑——识别更改的代码——与 I/O 的低级细节分离开来，我们可以轻松测试我们代码的核心部分。

通过这种方法，我们已经从测试我们的主入口函数`sync()`，转而测试更低级的函数`determine_actions()`。您可能会认为这没问题，因为`sync()`现在非常简单。或者您可能决定保留一些集成/验收测试来测试`sync()`。但还有另一种选择，那就是修改`sync()`函数，使其既可以进行单元测试*又*可以进行端到端测试；这是 Bob 称之为*边缘到边缘测试*的方法。

## 使用伪造和依赖注入进行边缘到边缘测试

当我们开始编写新系统时，我们经常首先关注核心逻辑，通过直接的单元测试驱动它。不过，某个时候，我们希望一起测试系统的更大块。

*我们*可以*回到端到端测试，但这些仍然像以前一样难以编写和维护。相反，我们经常编写调用整个系统但伪造 I/O 的测试，有点*边缘到边缘*：

*显式依赖项（`sync.py`）*

```py
def sync(source, dest, filesystem=FileSystem()):  #(1)
    source_hashes = filesystem.read(source)  #(2)
    dest_hashes = filesystem.read(dest)  #(2)

    for sha, filename in source_hashes.items():
        if sha not in dest_hashes:
            sourcepath = Path(source) / filename
            destpath = Path(dest) / filename
            filesystem.copy(sourcepath, destpath)  #(3)

        elif dest_hashes[sha] != filename:
            olddestpath = Path(dest) / dest_hashes[sha]
            newdestpath = Path(dest) / filename
            filesystem.move(olddestpath, newdestpath)  #(3)

    for sha, filename in dest_hashes.items():
        if sha not in source_hashes:
            filesystem.delete(dest / filename)  #(3)
```

①

我们的顶层函数现在公开了两个新的依赖项，一个`reader`和一个`filesystem`。

②

我们调用`reader`来生成我们的文件字典。

③

我们调用`filesystem`来应用我们检测到的更改。

###### 提示

虽然我们正在使用依赖注入，但没有必要定义抽象基类或任何明确的接口。在本书中，我们经常展示 ABCs，因为我们希望它们能帮助您理解抽象是什么，但它们并不是必需的。Python 的动态特性意味着我们总是可以依赖鸭子类型。

*使用 DI 的测试*

```py
class FakeFilesystem:
    def __init__(self, path_hashes):  #(1)
        self.path_hashes = path_hashes
        self.actions = []  #(2)

    def read(self, path):
        return self.path_hashes[path]  #(1)

    def copy(self, source, dest):
        self.actions.append(('COPY', source, dest))  #(2)

    def move(self, source, dest):
        self.actions.append(('MOVE', source, dest))  #(2)

    def delete(self, dest):
        self.actions.append(('DELETE', dest))  #(2)
```

①

Bob *非常*喜欢使用列表来构建简单的测试替身，尽管他的同事们会生气。这意味着我们可以编写像`assert *foo* not in database`这样的测试。

②

我们`FakeFileSystem`中的每个方法都只是将一些内容附加到列表中，以便我们以后可以检查它。这是间谍对象的一个例子。

这种方法的优势在于我们的测试作用于与我们的生产代码使用的完全相同的函数。缺点是我们必须明确地表达我们的有状态组件并传递它们。Ruby on Rails 的创始人 David Heinemeier Hansson 曾经著名地将这描述为“测试诱导的设计损害”。

无论哪种情况，我们现在都可以着手修复实现中的所有错误；为所有边缘情况列举测试现在变得更容易了。

## 为什么不只是补丁？

在这一点上，你可能会挠头想，“为什么你不只是使用`mock.patch`，省点力气呢？”

我们在本书和我们的生产代码中都避免使用模拟。我们不会卷入圣战，但我们的直觉是，模拟框架，特别是 monkeypatching，是一种代码异味。

相反，我们喜欢清楚地确定代码库中的责任，并将这些责任分离成小而专注的对象，这些对象易于用测试替身替换。

###### 注意

你可以在第八章中看到一个例子，我们在那里使用`mock.patch()`来替换一个发送电子邮件的模块，但最终我们在第十三章中用显式的依赖注入替换了它。

我们对我们的偏好有三个密切相关的原因：

+   消除你正在使用的依赖关系，可以对代码进行单元测试，但这对改进设计没有任何帮助。使用`mock.patch`不会让你的代码与`--dry-run`标志一起工作，也不会帮助你针对 FTP 服务器运行。为此，你需要引入抽象。

+   使用模拟测试的测试*倾向于*更多地与代码库的实现细节耦合。这是因为模拟测试验证了事物之间的交互：我们是否用正确的参数调用了`shutil.copy`？根据我们的经验，代码和测试之间的这种耦合*倾向于*使测试更加脆弱。

+   过度使用模拟会导致复杂的测试套件，无法解释代码。

###### 注意

为可测试性而设计实际上意味着为可扩展性而设计。我们为了更清晰的设计而进行了一些复杂性的折衷，这样可以容纳新的用例。

我们首先将 TDD 视为一种设计实践，其次是一种测试实践。测试作为我们设计选择的记录，并在我们长时间离开代码后为我们解释系统的作用。

使用太多模拟的测试会被隐藏在设置代码中，这些设置代码隐藏了我们关心的故事。

Steve Freeman 在他的演讲[“测试驱动开发”](https://oreil.ly/jAmtr)中有一个很好的过度模拟测试的例子。你还应该看看这个 PyCon 演讲，[“Mocking and Patching Pitfalls”](https://oreil.ly/s3e05)，由我们尊敬的技术审阅人员 Ed Jung 进行，其中也涉及了模拟和其替代方案。还有，我们推荐的演讲，不要错过 Brandon Rhodes 关于[“提升你的 I/O”](https://oreil.ly/oiXJM)的讲解，他用另一个简单的例子很好地涵盖了我们正在讨论的问题。

###### 提示

在本章中，我们花了很多时间用单元测试替换端到端测试。这并不意味着我们认为你永远不应该使用端到端测试！在本书中，我们展示了一些技术，让你尽可能地获得一个体面的测试金字塔，并且尽可能少地需要端到端测试来获得信心。继续阅读[“总结：不同类型测试的经验法则”](ch05.xhtml#types_of_test_rules_of_thumb)以获取更多细节。

# 总结

我们将在本书中一再看到这个想法：通过简化业务逻辑和混乱的 I/O 之间的接口，我们可以使我们的系统更容易测试和维护。找到合适的抽象是棘手的，但以下是一些启发和问题，你可以问问自己：

+   我可以选择一个熟悉的 Python 数据结构来表示混乱系统的状态，然后尝试想象一个可以返回该状态的单个函数吗？

+   我在哪里可以在我的系统中划出一条线，在哪里可以开辟一个[接缝](https://oreil.ly/zNUGG)来放置这个抽象？

+   将事物划分为具有不同责任的组件的合理方式是什么？我可以将隐含的概念变得明确吗？

+   有哪些依赖关系，什么是核心业务逻辑？

练习使不完美减少！现在回到我们的常规编程…

¹ 代码卡塔是一个小的、独立的编程挑战，通常用于练习 TDD。请参阅 Peter Provost 的《Kata—The Only Way to Learn TDD》。

² 如果你习惯以接口的方式思考，那么这就是我们在这里尝试定义的内容。

³ 这并不是说我们认为伦敦学派的人是错的。一些非常聪明的人就是这样工作的。只是我们不习惯这种方式。
