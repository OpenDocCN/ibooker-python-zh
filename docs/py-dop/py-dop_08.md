# 第八章。DevOps 的 Pytest

持续集成、持续交付、部署以及一般的管道工作流，只要稍加思考，就会充满验证。这种*验证*可以在每一步和达到重要目标时发生。

例如，在生成部署的一长串步骤中，如果调用 `curl` 命令获取一个非常重要的文件，如果失败了，你认为构建应该继续吗？可能不应该！`curl` 有一个标志可以用来产生非零的退出状态（`--fail`），如果发生 HTTP 错误。这个简单的标志用法是一种验证：确保请求成功，否则失败构建步骤。*关键词*是*确保*某事成功，这正是本章的核心：可以帮助您构建更好基础设施的验证和测试策略。

考虑到 Python 混合其中时，思考验证变得更加令人满意，利用像 `pytest` 这样的测试框架来处理系统的验证。

本章回顾了使用出色的 `pytest` 框架进行 Python 测试的一些基础知识，然后深入探讨了框架的一些高级特性，最后详细介绍了 *TestInfra* 项目，这是 `pytest` 的一个插件，可进行系统验证。

# 使用 pytest 进行测试超能力

我们对 `pytest` 框架赞不绝口。由 Holger Krekel 创建，现在由一些人维护，他们出色地生产出一个高质量的软件，通常是我们日常工作的一部分。作为一个功能齐全的框架，很难将范围缩小到足以提供有用的介绍，而不重复项目的完整文档。

###### 小贴士

`pytest` 项目在其[文档中](https://oreil.ly/PSAu2)有大量信息、示例和特性细节值得查看。随着项目持续提供新版本和改进测试的不同方法，总是有新东西可以学习。

当 Alfredo 首次接触这个框架时，他在尝试编写测试时遇到了困难，并发现要遵循 Python 的内置测试方式与 `unittest` 相比有些麻烦（本章稍后将详细讨论这些差异）。他花了几分钟时间就迷上了 `pytest` 的神奇报告功能。它不强迫他改变他编写测试的方式，而且可以立即使用，无需修改！这种灵活性贯穿整个项目，即使今天可能无法做到的事情，也可以通过插件或配置文件扩展其功能。

通过了解如何编写更简单的测试用例，并利用命令行工具、报告引擎、插件可扩展性和框架实用程序，您将希望编写更多无疑更好的测试。

# 开始使用 pytest

在其最简单的形式中，`pytest`是一个命令行工具，用于发现 Python 测试并执行它们。它不强迫用户理解其内部机制，这使得入门变得容易。本节演示了一些最基本的功能，从编写测试到布置文件（以便它们被自动发现），最后看看它与 Python 内置测试框架`unittest`的主要区别。

###### 提示

大多数集成开发环境（IDE），如 PyCharm 和 Visual Studio Code，都内置了对运行`pytest`的支持。如果使用像 Vim 这样的文本编辑器，则可以通过[`pytest.vim`](https://oreil.ly/HowKu)插件进行支持。从编辑器中使用`pytest`可以节省时间，使调试失败变得更容易，但要注意，并非每个选项或插件都受支持。

## 使用 pytest 进行测试

确保您已安装并可以在命令行中使用`pytest`：

```py
$ python3 -m venv testing
$ source testing/bin/activate
```

创建一个名为*test_basic.py*的文件；它应该看起来像这样：

```py
def test_simple():
    assert True

def test_fails():
    assert False
```

如果`pytest`在没有任何参数的情况下运行，它应该显示一个通过和一个失败：

```py
$ (testing) pytest
============================= test session starts =============================
platform linux -- Python 3.6.8, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
rootdir: /home/alfredo/python/testing
collected 2 items

test_basic.py .F                                                        [100%]

================================== FAILURES ===================================
_________________________________ test_fails __________________________________

    def test_fails():
>       assert False
E       assert False

test_basic.py:6: AssertionError
===================== 1 failed, 1 passed in 0.02 seconds ======================
```

输出从一开始就非常有益；它显示了收集了多少个测试、通过了多少个测试以及失败了哪一个测试（包括其行号）。

###### 提示

`pytest`的默认输出非常方便，但可能过于冗长。您可以通过配置控制输出量，使用`-q`标志来减少输出量。

不需要创建一个包含测试的类；函数被发现并正确运行。测试套件可以同时包含两者的混合，而框架在这种环境下也能正常工作。

### 布局和约定

在 Python 中进行测试时，`pytest`隐含遵循一些约定。这些约定大多数是关于命名和结构的。例如，尝试将*test_basic.py*文件重命名为*basic.py*，然后运行`pytest`看看会发生什么：

```py
$ (testing) pytest -q

no tests ran in 0.00 seconds
```

由于将测试文件前缀为*test_*的约定，没有运行任何测试。如果将文件重命名回*test_basic.py*，它应该能够被自动发现并运行测试。

###### 注意

布局和约定对于自动发现测试非常有帮助。可以配置框架以使用其他命名约定或直接测试具有唯一名称的文件。但是，遵循基本预期有助于避免测试不运行时的混淆。

这些都是将工具用于发现测试的约定：

+   测试目录需要命名为*tests*。

+   测试文件需要以*test*作为前缀；例如，*test_basic.py*，或者以*test.py*作为后缀。

+   测试函数需要以`test_`作为前缀；例如，`def test_simple():`。

+   测试类需要以`Test`作为前缀；例如，`class TestSimple`。

+   测试方法遵循与函数相同的约定，以`test_`作为前缀；例如，`def test_method(self):`。

因为在自动发现和执行测试时需要前缀`test_`，所以可以引入带有不同名称的帮助函数和其他非测试代码，以便自动排除它们。

## 与 unittest 的差异

Python 已经提供了一套用于测试的实用工具和辅助程序，它们是`unittest`模块的一部分。了解`pytest`与其不同之处以及为什么强烈推荐使用它是很有用的。

`unittest`模块强制使用类和类继承。对于了解面向对象编程和类继承的经验丰富的开发人员，这不应该是一个问题，但对于初学者来说，*这是一个障碍*。写基本测试不应该要求使用类和继承！

强制用户从`unittest.TestCase`继承的部分是，您必须理解（并记住）用于验证结果的大多数断言方法。使用`pytest`时，有一个可以完成所有工作的单一断言助手：`assert`。

这些是在使用`unittest`编写测试时可以使用的几个断言方法。其中一些很容易理解，而另一些则非常令人困惑：

+   `self.assertEqual(a, b)`

+   `self.assertNotEqual(a, b)`

+   `self.assertTrue(x)`

+   `self.assertFalse(x)`

+   `self.assertIs(a, b)`

+   `self.assertIsNot(a, b)`

+   `self.assertIsNone(x)`

+   `self.assertIsNotNone(x)`

+   `self.assertIn(a, b)`

+   `self.assertNotIn(a, b)`

+   `self.assertIsInstance(a, b)`

+   `self.assertNotIsInstance(a, b)`

+   `self.assertRaises(exc, fun, *args, **kwds)`

+   `self.assertRaisesRegex(exc, r, fun, *args, **kwds)`

+   `self.assertWarns(warn, fun, *args, **kwds)`

+   `self.assertWarnsRegex(warn, r, fun, *args, **kwds)`

+   `self.assertLogs(logger, level)`

+   `self.assertMultiLineEqual(a, b)`

+   `self.assertSequenceEqual(a, b)`

+   `self.assertListEqual(a, b)`

+   `self.assertTupleEqual(a, b)`

+   `self.assertSetEqual(a, b)`

+   `self.assertDictEqual(a, b)`

+   `self.assertAlmostEqual(a, b)`

+   `self.assertNotAlmostEqual(a, b)`

+   `self.assertGreater(a, b)`

+   `self.assertGreaterEqual(a, b)`

+   `self.assertLess(a, b)`

+   `self.assertLessEqual(a, b)`

+   `self.assertRegex(s, r)`

+   `self.assertNotRegex(s, r)`

+   `self.assertCountEqual(a, b)`

`pytest`允许您专门使用`assert`，并且不强制您使用上述任何一种方法。此外，它*确实允许*您使用`unittest`编写测试，并且甚至执行这些测试。我们强烈建议不要这样做，并建议您专注于只使用普通的 assert 语句。

不仅使用普通的 assert 更容易，而且`pytest`在失败时还提供了丰富的比较引擎（下一节将详细介绍）。

# pytest 功能

除了使编写测试和执行测试更容易外，该框架还提供了许多可扩展的选项，例如钩子。钩子允许您在运行时的不同点与框架内部进行交互。例如，如果要修改测试的收集，可以添加一个收集引擎的钩子。另一个有用的示例是，如果要在测试失败时实现更好的报告。

在开发 HTTP API 时，我们发现有时测试中使用 HTTP 请求针对应用程序的失败并不有益：断言失败会被报告，因为预期的响应（HTTP 200）是 HTTP 500 错误。我们想要了解更多关于请求的信息：到哪个 URL 端点？如果是 POST 请求，是否有数据？它是什么样子的？这些信息已经包含在 HTTP 响应对象中，因此我们编写了一个钩子来查看这个对象，并将所有这些项目包含在失败报告中。

钩子是`pytest`的高级功能，您可能根本不需要，但了解框架可以灵活适应不同需求是有用的。接下来的章节涵盖如何扩展框架，为什么使用`assert`如此宝贵，如何参数化测试以减少重复，如何使用`fixtures`制作帮助工具，以及如何使用内置工具。

## conftest.py

大多数软件都允许您通过插件扩展功能（例如，Web 浏览器称其为*扩展*）；同样地，`pytest`有一个丰富的 API 用于开发插件。这里没有涵盖完整的 API，但其更简单的方法是：*conftest.py*文件。在这个文件中，工具可以像插件一样扩展。无需完全理解如何创建单独的插件、打包它并安装它。如果存在*conftest.py*文件，框架将加载它并消费其中的任何特定指令。这一切都是自动进行的！

通常，您会发现*conftest.py*文件用于保存钩子、fixtures 和这些 fixtures 的 helpers。如果声明为参数，这些*fixtures*可以在测试中使用（该过程稍后在 fixture 部分描述）。

当多个测试模块将使用它时，将 fixtures 和 helpers 添加到此文件是有意义的。如果只有一个单独的测试文件，或者只有一个文件将使用 fixture 或 hook，那么无需创建或使用*conftest.py*文件。Fixtures 和 helpers 可以在与测试相同的文件中定义并表现相同的行为。

加载*conftest.py*文件的唯一条件是存在于*tests*目录中并正确匹配名称。此外，尽管此名称是可配置的，但我们建议不要更改它，并鼓励您遵循默认命名约定以避免潜在问题。

## 令人惊叹的**assert**

当我们不得不描述`pytest`工具的强大之处时，我们首先描述`assert`语句的重要用途。幕后，框架检查对象并提供丰富的比较引擎以更好地描述错误。通常会遇到抵制，因为 Python 中的裸`assert`很糟糕地描述错误。以比较两个长字符串为例：

```py
>>> assert "using assert for errors" == "using asert for errors"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError
```

差异在哪里？如果不花时间仔细查看这两行长字符串，很难说清楚。这会导致人们建议不要这样做。一个小测试展示了`pytest`在报告失败时如何增强：

```py
$ (testing) pytest test_long_lines.py
============================= test session starts =============================
platform linux -- Python 3.6.8, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
collected 1 item

test_long_lines.py F                                                    [100%]

================================== FAILURES ===================================
_______________________________ test_long_lines _______________________________

    def test_long_lines():
>      assert "using assert for errors" == "using asert for errors"
E      AssertionError: assert '...rt for errors' == '...rt for errors'
E        - using assert for errors
E        ?        -
E        + using asert for errors

test_long_lines.py:2: AssertionError
========================== 1 failed in 0.04 seconds ===========================
```

你能说出错误在哪里吗？这*极大地简化了*。它不仅告诉你失败了，还指出*失败发生的确切位置*。例如，一个简单的断言与一个长字符串，但是这个框架可以处理其他数据结构，如列表和字典，毫无问题。你有没有在测试中比较过非常长的列表？很难轻松地分辨出哪些项目不同。这里是一个有长列表的小片段：

```py
    assert ['a', 'very', 'long', 'list', 'of', 'items'] == [
            'a', 'very', 'long', 'list', 'items']
E   AssertionError: assert [...'of', 'items'] == [...ist', 'items']
E     At index 4 diff: 'of' != 'items'
E     Left contains more items, first extra item: 'items'
E     Use -v to get the full diff
```

在通知用户测试失败后，它精确指向索引号（第四个或第五个项目），最后说一个列表有一个额外的项目。没有这种深入的反思，调试失败将需要很长时间。报告中的额外奖励是，默认情况下，在进行比较时省略非常长的项目，因此输出中只显示相关部分。毕竟，你想知道的不仅是列表（或任何其他数据结构）不同之处，而且*确切地在哪里*不同。

## 参数化

参数化是一个需要一些时间来理解的特性，因为它在`unittest`模块中不存在，是 pytest 框架独有的特性。一旦你发现自己编写非常相似的测试，只是输入稍有不同，但测试的是同一个东西时，它就会变得很清晰。举个例子，这个类正在测试一个函数，如果一个字符串暗示一个真实的值，则返回`True`。`string_to_bool`是测试的函数：

```py
from my_module import string_to_bool

class TestStringToBool(object):

    def test_it_detects_lowercase_yes(self):
        assert string_to_bool('yes')

    def test_it_detects_odd_case_yes(self):
        assert string_to_bool('YeS')

    def test_it_detects_uppercase_yes(self):
        assert string_to_bool('YES')

    def test_it_detects_positive_str_integers(self):
        assert string_to_bool('1')

    def test_it_detects_true(self):
        assert string_to_bool('true')

    def test_it_detects_true_with_trailing_spaces(self):
        assert string_to_bool('true ')

    def test_it_detects_true_with_leading_spaces(self):
        assert string_to_bool(' true')
```

看看所有这些测试如何从相似的输入评估相同的结果？这就是参数化发挥作用的地方，因为它可以将所有这些值分组并传递给测试；它可以有效地将它们减少到单个测试中：

```py
import pytest
from my_module import string_to_bool

true_values = ['yes', '1', 'Yes', 'TRUE', 'TruE', 'True', 'true']

class TestStrToBool(object):

    @pytest.mark.parametrize('value', true_values)
    def test_it_detects_truish_strings(self, value)
        assert string_to_bool(value)
```

这里发生了几件事情。首先导入了`pytest`（框架）以使用`pytest.mark.parametrize`模块，然后将`true_values`定义为应评估相同的所有值的（列表）变量，并最后将所有测试方法替换为单一方法。测试方法使用`parametrize`装饰器，定义了两个参数。第一个是字符串`*value*`，第二个是先前定义的列表的名称。这可能看起来有点奇怪，但它告诉框架`*value*`是在测试方法中使用的参数名称。这就是`value`参数的来源！

如果在运行时增加了详细输出，输出将显示确切传入的值。它几乎看起来像单个测试被复制到每次迭代中传入的值中：

```py
test_long_lines.py::TestLongLines::test_detects_truish_strings[yes] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[1] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[Yes] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[TRUE] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[TruE] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[True] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[true] PASSED
```

输出包括 *单个测试* 中每次迭代中使用的值，用方括号括起来。它将非常冗长的测试类简化为单个测试方法，多亏了 `parametrize`。下次您发现自己编写非常相似的测试并且使用不同的输入来断言相同的结果时，您将知道可以通过 `parametrize` 装饰器简化它。

# *Fixture*

我们把 [`pytest` *Fixture*](https://oreil.ly/gPoM5) 想象成可以注入到测试中的小助手。无论您是编写单个测试函数还是一堆测试方法，*Fixture* 都可以以相同的方式使用。如果它们不会被其他测试文件共享，那么可以在同一个测试文件中定义它们；否则它们可以放在 *conftest.py* 文件中。*Fixture* 就像帮助函数一样，可以是任何您需要的测试用例，从预创建的简单数据结构到为 Web 应用程序设置数据库等更复杂的功能。

这些助手还可以有定义的 *scope*。它们可以有特定的代码，为每个测试方法、类和模块进行清理，或者甚至允许为整个测试会话设置它们一次。通过在测试方法（或测试函数）中定义它们，您实际上是在运行时获取 *Fixture* 的注入。如果这听起来有点混乱，通过下几节中的示例将变得清晰起来。

## **入门**

用来定义和使用的 *Fixture* 如此简单，以至于它们经常被滥用。我们知道我们创建了一些本来可以简化为简单帮助方法的 *Fixture*！正如我们已经提到的，*Fixture* 有许多不同的用例——从简单的数据结构到更复杂的用例，例如为单个测试设置整个数据库。

最近，Alfredo 不得不测试一个解析特定文件内容的小应用程序，该文件称为 *keyring file*。它具有类似 INI 文件的结构，某些值必须是唯一的，并且遵循特定的格式。在每次测试中重新创建文件结构可能非常繁琐，因此创建了一个 *Fixture* 来帮助。这就是 *keyring file* 的外观：

```py
[mon.]
    key = AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A==
    caps mon = "allow *"
```

*Fixture* 是一个返回 *keyring file* 内容的函数。让我们创建一个名为 test_keyring.py 的新文件，其中包含 *Fixture* 的内容，以及验证默认键的小测试函数：

```py
import pytest
import random

@pytest.fixture
def mon_keyring():
    def make_keyring(default=False):
        if default:
            key = "AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A=="
        else:
            key = "%032x==" % random.getrandbits(128)

        return """
 [mon.]
 key = %s
 caps mon = "allow *"
 """ % key
    return make_keyring

def test_default_key(mon_keyring):
    contents = mon_keyring(default=True)
    assert "AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A==" in contents
```

*Fixture* 使用一个执行繁重工作的嵌套函数，允许使用一个 *default* 键值，并在调用者希望有随机键时返回嵌套函数。在测试中，它通过声明为测试函数的参数的一部分接收 *Fixture*（在本例中为 `mon_keyring`），并使用 `default=True` 调用 *Fixture*，以便使用默认键，然后验证它是否按预期生成。

###### **注意**

在真实场景中，生成的内容将被传递给解析器，确保解析后的行为符合预期，并且没有错误发生。

使用此固件的生产代码最终发展到执行其他类型的测试，并且在某些时候，测试希望验证解析器能够处理不同条件下的文件。该固件返回一个字符串，因此需要扩展它。现有测试已经使用了 `mon_keyring` 固件，因此为了在不更改当前固件的情况下扩展功能，创建了一个新的固件，该固件使用了框架的一个特性。固件可以 *请求* 其他固件！您将所需的固件定义为参数（就像测试函数或测试方法一样），因此在执行时框架会注入它。

这是创建（并返回）文件的新固件的方式：

```py
@pytest.fixture
def keyring_file(mon_keyring, tmpdir):
    def generate_file(default=False):
        keyring = tmpdir.join('keyring')
        keyring.write_text(mon_keyring(default=default))
        return keyring.strpath
    return generate_file
```

按行解释，`pytest.fixture`装饰器告诉框架这个函数是一个固件，然后定义了固件，请求 *两个固件* 作为参数：`mon_keyring` 和 `tmpdir`。第一个是前面在 *test_keyring.py* 文件中创建的，第二个是框架提供的内置固件（关于内置固件的更多内容将在下一节讨论）。`tmpdir` 固件允许您使用一个临时目录，在测试完成后将其删除，然后创建 *keyring* 文件，并写入由 `mon_keyring` 固件生成的文本，传递 `default` 参数。最后，它返回新创建文件的绝对路径，以便测试可以使用它。

这是测试函数如何使用它的方式：

```py
def test_keyring_file_contents(keyring_file):
    keyring_path = keyring_file(default=True)
    with open(keyring_path) as fp:
        contents = fp.read()
    assert "AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A==" in contents
```

您现在应该对固件是什么，您可以在哪里定义它们以及如何在测试中使用它们有了一个很好的理解。下一部分将介绍一些最有用的内置固件，这些固件是框架的一部分。

## 内置固件

前一节简要介绍了 `pytest` 提供的众多内置固件之一：`tmpdir` 固件。框架提供了更多固件。要验证可用固件的完整列表，请运行以下命令：

```py
$ (testing) pytest  -q --fixtures
```

我们经常使用的两个固件是 `monkeypatch` 和 `capsys`，当运行上述命令时，它们都在生成的列表中。这是您将在终端看到的简要描述：

```py
capsys
    enables capturing of writes to sys.stdout/sys.stderr and makes
    captured output available via ``capsys.readouterr()`` method calls
    which return a ``(out, err)`` tuple.
monkeypatch
    The returned ``monkeypatch`` funcarg provides these
    helper methods to modify objects, dictionaries or os.environ::

    monkeypatch.setattr(obj, name, value, raising=True)
    monkeypatch.delattr(obj, name, raising=True)
    monkeypatch.setitem(mapping, name, value)
    monkeypatch.delitem(obj, name, raising=True)
    monkeypatch.setenv(name, value, prepend=False)
    monkeypatch.delenv(name, value, raising=True)
    monkeypatch.syspath_prepend(path)
    monkeypatch.chdir(path)

    All modifications will be undone after the requesting
    test function has finished. The ``raising``
    parameter determines if a KeyError or AttributeError
    will be raised if the set/deletion operation has no target.
```

`capsys` 捕获测试中产生的任何 `stdout` 或 `stderr`。您是否尝试过验证某些命令输出或日志记录在单元测试中的输出？这很难做到，并且需要一个单独的插件或库来 *patch* Python 的内部，然后检查其内容。

这是验证分别在 `stderr` 和 `stdout` 上产生的输出的两个测试函数：

```py
import sys

def stderr_logging():
    sys.stderr.write('stderr output being produced')

def stdout_logging():
    sys.stdout.write('stdout output being produced')

def test_verify_stderr(capsys):
    stderr_logging()
    out, err = capsys.readouterr()
    assert out == ''
    assert err == 'stderr output being produced'

def test_verify_stdout(capsys):
    stdout_logging()
    out, err = capsys.readouterr()
    assert out == 'stdout output being produced'
    assert err == ''
```

`capsys` 固件处理所有补丁，设置和助手，以检索测试中生成的 `stderr` 和 `stdout`。每次测试都会重置内容，这确保变量填充了正确的输出。

`monkeypatch`可能是我们最常使用的装置。在测试时，有些情况下测试的代码不在我们的控制之下，*修补*就需要发生来覆盖模块或函数以具有特定的行为。Python 中有相当多的*修补*和*模拟*库（*模拟*是帮助设置修补对象行为的助手），但`monkeypatch`足够好，你可能不需要安装额外的库来帮忙。

以下函数运行系统命令以捕获设备的详细信息，然后解析输出，并返回一个属性（由`blkid`报告的`ID_PART_ENTRY_TYPE`）：

```py
import subprocess

def get_part_entry_type(device):
    """
 Parses the ``ID_PART_ENTRY_TYPE`` from the "low level" (bypasses the cache)
 output that uses the ``udev`` type of output.
 """
    stdout = subprocess.check_output(['blkid', '-p', '-o', 'udev', device])
    for line in stdout.split('\n'):
        if 'ID_PART_ENTRY_TYPE=' in line:
            return line.split('=')[-1].strip()
    return ''
```

要进行测试，设置所需的行为在`subprocess`模块的`check_output`属性上。这是使用`monkeypatch`装置的测试函数的外观：

```py
def test_parses_id_entry_type(monkeypatch):
    monkeypatch.setattr(
        'subprocess.check_output',
        lambda cmd: '\nID_PART_ENTRY_TYPE=aaaaa')
    assert get_part_entry_type('/dev/sda') == 'aaaa'
```

`setattr`调用*设置*修补过的可调用对象（在本例中为`check_output`）。*补丁*它的是一个返回感兴趣行的 lambda 函数。由于`subprocess.check_output`函数不在我们的直接控制之下，并且`get_part_entry_type`函数不允许任何其他方式来注入值，修补是唯一的方法。

我们倾向于使用其他技术，如在尝试修补之前注入值（称为*依赖注入*），但有时没有其他方法。提供一个可以修补和处理所有测试清理工作的库，这是`pytest`是一种愉悦的工作方式的更多原因之一。

# 基础设施测试

本节解释了如何使用[Testinfra 项目](https://oreil.ly/e7Afx)进行基础设施测试和验证。这是一个依赖于装置的`pytest`插件，允许您编写 Python 测试，就像测试代码一样。

前几节详细讨论了`pytest`的使用和示例，本章以系统级验证的概念开始。我们解释基础设施测试的方式是通过问一个问题：*如何确定部署成功？*大多数情况下，这意味着一些手动检查，如加载网站或查看进程，这是不够的；这是错误的，并且如果系统很重要的话可能会变得乏味。

虽然最初可以将`pytest`视为编写和运行 Python 单元测试的工具，但将其重新用于基础设施测试可能是有利的。几年前，阿尔弗雷多被委托制作一个安装程序，通过 HTTP API 公开其功能。该安装程序旨在创建一个 [Ceph 集群](https://ceph.com)，涉及多台机器。在启动 API 的质量保证阶段，常常会收到集群未按预期工作的报告，因此他会获取凭据以登录这些机器并进行检查。一旦必须调试包含多台机器的分布式系统时，就会产生乘数效应：多个配置文件、不同的硬盘、网络设置，任何和所有的东西都可能不同，即使它们看起来很相似。

每当阿尔弗雷多需要调试这些系统时，他都会有一个日益增长的检查清单。服务器上的配置是否相同？权限是否符合预期？特定用户是否存在？最终他会忘记某些事情，并花时间试图弄清楚自己漏掉了什么。这是一个不可持续的过程。*如果我能写一些简单的测试用例来针对集群？* 阿尔弗雷多编写了几个简单的测试来验证清单上的项目，并执行它们以检查构成集群的机器。在他意识到之前，他已经拥有了一套很好的测试，只需几秒钟即可运行，可以识别各种问题。

这对于改进交付流程是一个令人难以置信的启示。他甚至可以在开发安装程序时执行这些（功能）测试，并发现不正确的地方。如果 QA 团队发现任何问题，他可以针对其设置运行相同的测试。有时测试会捕捉到环境问题：一个硬盘*脏了*并导致部署失败；来自不同集群的配置文件遗留下来并引发问题。自动化、精细化测试以及频繁运行它们使工作变得更好，并减轻了 QA 团队需要处理的工作量。

TestInfra 项目具有各种夹具，可高效测试系统，并包含一整套用于连接服务器的后端，无论其部署类型如何：Ansible、Docker、SSH 和 Kubernetes 是一些支持的连接方式。通过支持多种不同的连接后端，可以执行相同的一组测试，而不受基础设施更改的影响。

接下来的章节将介绍不同的后端，并展示一个生产项目的示例。

## 什么是系统验证？

系统验证可以在不同级别（使用监控和警报系统）和应用程序生命周期的不同阶段进行，例如在预部署阶段、运行时或部署期间。最近由 Alfredo 投入生产的应用程序需要在重新启动时优雅地处理客户端连接，即使有任何中断也不受影响。为了维持流量，应用程序进行了负载均衡：在系统负载较重时，新的连接会被发送到负载较轻的其他服务器。

当部署新版本时，应用程序*必须重新启动*。重新启动意味着客户端在最佳情况下会遇到奇怪的行为，最坏的情况下会导致非常糟糕的体验。为了避免这种情况，重新启动过程等待所有客户端连接终止，系统拒绝新的连接，允许其完成来自现有客户端的工作，其余系统继续工作。当没有活动连接时，部署继续并停止服务以获取更新的代码。

沿途的每一步都进行了验证：在部署之前通知负载均衡器停止发送新的客户端，并且后来验证没有新的客户端处于活动状态。如果该工作流程转换为测试，标题可能类似于：`确保当前没有客户端在运行`。一旦新代码就位，另一个验证步骤检查负载均衡器是否已确认服务器再次准备好生成工作。这里的另一个测试可能是：`负载均衡器已将服务器标记为活动`。最后，确保服务器正在接收新的客户端连接——又一个要编写的测试！

在这些步骤中，验证已经到位，可以编写测试来验证这种类型的工作流程。

系统验证也可以与监控服务器的整体健康状况（或集群环境中的多个服务器）相关联，或者作为开发应用程序和功能测试的持续集成的一部分。验证的基础知识适用于这些情况以及可能从状态验证中受益的任何其他情况。它不应仅用于测试，尽管这是一个很好的开始！

## Testinfra 简介

针对基础设施编写单元测试是一个强大的概念，使用 Testinfra 一年多以来，我们可以说它提高了我们必须交付的生产应用程序的质量。以下部分详细介绍了如何连接到不同节点并执行验证测试，并探讨了可用的固定装置类型。

要创建新的虚拟环境，请安装`pytest`：

```py
$ python3 -m venv validation
$ source testing/bin/activate
(validation) $ pip install pytest
```

安装`testinfra`，确保使用版本`2.1.0`：

```py
(validation) $ pip install "testinfra==2.1.0"
```

###### 注意

`pytest`固定装置提供了 Testinfra 项目提供的所有测试功能。要利用本节，您需要了解它们是如何工作的。

## 连接到远程节点

因为存在不同的后端连接类型，当未直接指定连接时，Testinfra 默认到某些类型。最好明确指定连接类型并在命令行中定义它。

这些是 Testinfra 支持的所有连接类型：

+   本地

+   Paramiko（Python 中的 SSH 实现）

+   Docker

+   SSH

+   Salt

+   Ansible

+   Kubernetes（通过 kubectl）

+   WinRM

+   LXC

在帮助菜单中会出现一个 `testinfra` 部分，提供一些关于提供的标志的上下文。这是来自 `pytest` 与其与 Testinfra 集成的一个不错的特性。这两个项目的帮助来自相同的命令：

```py
(validation) $ pytest --help
...

testinfra:
  --connection=CONNECTION
                        Remote connection backend (paramiko, ssh, safe-ssh,
                        salt, docker, ansible)
  --hosts=HOSTS         Hosts list (comma separated)
  --ssh-config=SSH_CONFIG
                        SSH config file
  --ssh-identity-file=SSH_IDENTITY_FILE
                        SSH identify file
  --sudo                Use sudo
  --sudo-user=SUDO_USER
                        sudo user
  --ansible-inventory=ANSIBLE_INVENTORY
                        Ansible inventory file
  --nagios              Nagios plugin
```

有两台服务器正在运行。为了演示连接选项，让我们检查它们是否在运行 CentOS 7，查看 */etc/os-release* 文件的内容。这是测试函数的外观（保存为 `test_remote.py`）：

```py
def test_release_file(host):
    release_file = host.file("/etc/os-release")
    assert release_file.contains('CentOS')
    assert release_file.contains('VERSION="7 (Core)"')
```

它是一个单一的测试函数，接受 `host` fixture，并针对所有指定的节点运行。

`--hosts` 标志接受一个主机列表，并使用连接方案（例如 SSH 将使用 `*ssh://hostname*`），还允许使用通配符进行一些其他变体。如果一次测试多个远程服务器，则在命令行中传递它们变得很麻烦。以下是使用 SSH 测试两台服务器的方式：

```py
(validation) $ pytest -v --hosts='ssh://node1,ssh://node2' test_remote.py
============================= test session starts =============================
platform linux -- Python 3.6.8, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
cachedir: .pytest_cache
rootdir: /home/alfredo/python/python-devops/samples/chapter16
plugins: testinfra-3.0.0, xdist-1.28.0, forked-1.0.2
collected 2 items

test_remote.py::test_release_file[ssh://node1] PASSED                   [ 50%]
test_remote.py::test_release_file[ssh://node2] PASSED                   [100%]

========================== 2 passed in 3.82 seconds ===========================
```

使用增加冗余信息（使用 `-v` 标志）显示 Testinfra 在调用中指定的两台远程服务器中执行一个测试函数。

###### 注意

在设置主机时，具有无密码连接是重要的。不应该有任何密码提示，如果使用 SSH，则应该使用基于密钥的配置。

当自动化这些类型的测试（例如作为 CI 系统中的作业的一部分）时，您可以从生成主机、确定它们如何连接以及任何其他特殊指令中受益。Testinfra 可以使用 SSH 配置文件确定要连接的主机。在上一次测试运行中，使用了 [Vagrant](https://www.vagrantup.com)，它创建了具有特殊密钥和连接设置的这些服务器。Vagrant 可以为其创建的服务器生成临时 SSH 配置文件：

```py
(validation) $ vagrant ssh-config

Host node1
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/alfredo/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL

Host node2
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/alfredo/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

将输出内容导出到文件，然后传递给 Testinfra 可以提供更大的灵活性，特别是在使用多个主机时：

```py
(validation) $ vagrant ssh-config > ssh-config
(validation) $ pytest --hosts=default --ssh-config=ssh-config test_remote.py
```

使用 `--hosts=default` 避免在命令行中直接指定它们，并且引擎从 SSH 配置中获取。即使没有 Vagrant，SSH 配置提示仍然对连接到具有特定指令的多个主机有用。

[Ansible](https://www.ansible.com) 是另一种选择，如果节点是本地、SSH 或 Docker 容器。测试设置可以从使用主机清单（类似于 SSH 配置）中受益，它可以将主机分组到不同的部分。主机组也可以指定，以便您可以单独针对主机进行测试，而不是针对所有主机执行。

对于在先前示例中使用的 `node1` 和 `node2`，清单文件的定义如下（保存为 `hosts`）：

```py
[all]
node1
node2
```

如果对所有这些执行，命令将更改为：

```py
$ pytest --connection=ansible --ansible-inventory=hosts test_remote.py
```

如果在清单中定义了需要排除的其他主机，则还可以指定一个组。假设两个节点都是 Web 服务器，并且属于 `nginx` 组，则此命令将仅在该组上运行测试：

```py
$ pytest --hosts='ansible://nginx' --connection=ansible \
  --ansible-inventory=hosts test_remote.py
```

###### 提示

许多系统命令需要超级用户权限。为了允许特权升级，Testinfra 允许指定 `--sudo` 或 `--sudo-user`。`--sudo` 标志使引擎在执行命令时使用 `sudo`，而 `--sudo-user` 命令允许以不同用户的更高权限运行。这个 fixture 也可以直接使用。

## 功能和特殊的 fixtures。

到目前为止，在示例中，仅使用 `host` fixture 来检查文件及其内容。然而，这是具有误导性的。`host` fixture 是一个 *全包含* 的 fixture；它包含了 Testinfra 提供的所有其他强大 fixtures。这意味着示例已经使用了 `host.file`，其中包含了大量额外的功能。也可以直接使用该 fixture：

```py
In [1]: import testinfra

In [2]: host = testinfra.get_host('local://')

In [3]: node_file = host.file('/tmp')

In [4]: node_file.is_directory
Out[4]: True

In [5]: node_file.user
Out[5]: 'root'
```

全功能的 `host` fixture 利用了 Testinfra 的广泛 API，它为连接到的每个主机加载了所有内容。其想法是编写单个测试，针对从同一 `host` fixture 访问的不同节点执行。

这些是一些可用的 [几十个](https://oreil.ly/2_J-o) 属性。以下是其中一些最常用的：

`host.ansible`

在运行时提供对任何 Ansible 属性的完全访问，例如主机、清单和变量。

`host.addr`

网络工具，如检查 IPV4 和 IPV6，主机是否可达，主机是否可解析。

`host.docker`

代理到 Docker API，允许与容器交互，并检查它们是否在运行。

`host.interface`

用于检查给定接口地址的辅助工具。

`host.iptables`

用于验证防火墙规则（如 `host.iptables` 所见）的辅助工具。

`host.mount_point`

检查挂载点、文件系统类型在路径中的存在以及挂载选项。

`host.package`

*非常有用* 以查询包是否已安装及其版本。

`host.process`

检查运行中的进程。

`host.sudo`

允许您使用 `host.sudo` 执行命令，或作为不同用户执行。

`host.system_info`

各种系统元数据，如发行版版本、发布和代号。

`host.check_output`

运行系统命令，检查其输出（如果成功运行），可以与 `host.sudo` 结合使用。

`host.run`

运行命令，允许检查返回代码，`host.stderr` 和 `host.stdout`。

`host.run_expect`

验证返回代码是否符合预期。

# 示例

无摩擦地开始开发系统验证测试的方法是在创建实际部署时执行。与*测试驱动开发*（TDD）有些类似，任何进展都需要一个新的测试。在本节中，需要安装并配置 Web 服务器在端口 80 上运行以提供静态着陆页面。在取得进展的同时，将添加测试。编写测试的一部分是理解失败，因此将引入一些问题来帮助我们确定要修复的内容。

在*干净的* Ubuntu 服务器上，首先安装 Nginx 包：

```py
$ apt install nginx
```

在取得进展后创建一个名为*test_webserver.py*的新测试文件。Nginx 安装后，让我们再创建一个测试：

```py
def test_nginx_is_installed(host):
    assert host.package('nginx').is_installed
```

使用`-q`标志减少`pytest`输出的冗长以集中处理失败。远程服务器称为`node4`，使用 SSH 连接到它。这是运行第一个测试的命令：

```py
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
.
1 passed in 1.44 seconds
```

进展！Web 服务器需要运行，因此添加了一个新的测试来验证其行为：

```py
def test_nginx_is_running(host):
    assert host.service('nginx').is_running
```

再次运行*应该*再次成功：

```py
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
.F
================================== FAILURES ===================================
_____________________ test_nginx_is_running[ssh://node4] ______________________

host = <testinfra.host.Host object at 0x7f629bf1d668>

    def test_nginx_is_running(host):
>       assert host.service('nginx').is_running
E       AssertionError: assert False
E        +  where False = <service nginx>.is_running
E        +    where <service nginx> = <class 'SystemdService'>('nginx')

test_webserver.py:7: AssertionError
1 failed, 1 passed in 2.45 seconds
```

一些 Linux 发行版不允许在安装时启动包服务。此外，测试捕获到`systemd`（默认单元服务）报告 Nginx 服务未运行。手动启动 Nginx 并运行测试应该再次使一切顺利通过：

```py
(validate) $ systemctl start nginx
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
..
2 passed in 2.38 seconds
```

如本节开头所述，Web 服务器应在端口 80 上提供静态着陆页面。添加另一个测试（在*test_webserver.py*中）以验证端口是下一步：

```py
def test_nginx_listens_on_port_80(host):
    assert host.socket("tcp://0.0.0.0:80").is_listening
```

此测试更为复杂，需要关注一些细节。它选择检查*服务器中任何 IP 上*端口`80`的 TCP 连接。虽然对于此测试来说这没问题，但如果服务器有多个接口并配置为绑定到特定地址，则必须添加新的测试。添加另一个检查端口`80`是否在给定地址上监听的测试可能看起来有些多余，但如果考虑到报告，它有助于解释发生了什么：

1.  测试`nginx`是否在端口`80`上监听：PASS

1.  测试`nginx`是否在地址`192.168.0.2`和端口`80`上监听：FAIL

以上告诉我们 Nginx 绑定到端口`80`，*只是没有绑定到正确的接口*。额外的测试是提供细粒度的好方法（以牺牲额外的冗长）。

再次运行新添加的测试：

```py
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
..F
================================== FAILURES ===================================
_________________ test_nginx_listens_on_port_80[ssh://node4] __________________

host = <testinfra.host.Host object at 0x7fbaa64f26a0>

    def test_nginx_listens_on_port_80(host):
>       assert host.socket("tcp://0.0.0.0:80").is_listening
E       AssertionError: assert False
E        +  where False = <socket tcp://0.0.0.0:80>.is_listening
E        +    where <socket tcp://0.0.0.0:80> = <class 'LinuxSocketSS'>

test_webserver.py:11: AssertionError
1 failed, 2 passed in 2.98 seconds
```

没有任何地址在端口`80`上有监听。查看 Nginx 的配置发现，它设置为使用默认站点中的指令在端口`8080`上进行监听：

```py
(validate) $ grep "listen 8080" /etc/nginx/sites-available/default
    listen 8080 default_server;
```

将其改回到端口`80`并重新启动`nginx`服务，测试再次通过：

```py
(validate) $ grep "listen 80" /etc/nginx/sites-available/default
    listen 80 default_server;
(validate) $ systemctl restart nginx
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
...
3 passed in 2.92 seconds
```

由于没有内置的夹具来处理向地址的 HTTP 请求，最后一个测试使用`wget`实用程序检索正在运行的网站的内容，并对输出进行断言以确保静态站点渲染：

```py
def test_get_content_from_site(host):
    output = host.check_output('wget -qO- 0.0.0.0:80')
    assert 'Welcome to nginx' in output
```

再次运行*test_webserver.py*以验证所有我们的假设是正确的：

```py
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
....
4 passed in 3.29 seconds
```

理解 Python 中测试概念，并将其重新用于系统验证，具有非常强大的功能。在开发应用程序时自动化测试运行，甚至在现有基础设施上编写和运行测试，都是简化日常操作的极佳方式，因为这些操作往往容易出错。pytest 和 Testinfra 是可以帮助你入门的优秀项目，并且在需要扩展时使用起来非常方便。测试是技能的*升级*。

# 使用 pytest 测试 Jupyter Notebooks

在数据科学和机器学习中，如果忘记应用软件工程最佳实践，很容易在公司引入大问题。解决这个问题的一种方法是使用 pytest 的`nbval`插件，它允许你测试你的笔记本。看看这个`Makefile`：

```py
setup:
    python3 -m venv ~/.myrepo

install:
    pip install -r requirements.txt

test:
    python -m pytest -vv --cov=myrepolib tests/*.py
    python -m pytest --nbval notebook.ipynb

lint:
    pylint --disable=R,C myrepolib cli web

all: install lint test
```

关键项目是`--nbval`标志，它还允许建立服务器测试仓库中的笔记本。

# 练习

+   至少列出三个约定，以便`pytest`可以发现测试。

+   *conftest.py*文件的作用是什么？

+   解释测试参数化。

+   什么是 fixture，如何在测试中使用它？它方便吗？为什么？

+   解释如何使用`monkeypatch` fixture。

# 案例研究问题

+   创建一个测试模块，使用`testinfra`连接到远程服务器。测试 Nginx 是否安装，在`systemd`下运行，并且服务器是否绑定到端口 80。当所有测试通过时，尝试通过配置 Nginx 监听不同的端口来使它们失败。
