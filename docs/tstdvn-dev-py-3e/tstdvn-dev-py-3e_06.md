# 第二章。使用 unittest 模块扩展我们的功能测试

让我们调整我们的测试，目前它检查的是默认的 Django “it worked” 页面，而不是我们想在站点的真实首页上看到的一些内容。

是时候揭示我们正在构建的 Web 应用程序的类型了：一个待办事项列表网站！我知道，我知道，线上每个其他 Web 开发教程也是一个待办事项列表应用，或者可能是博客或投票应用。我非常跟风。

原因是待办事项列表是一个非常好的例子。在其最基本的形式中，它非常简单—​只是一列文本字符串—​因此很容易启动和运行一个“最小可行”列表应用程序。但是它可以通过各种方式扩展—​不同的持久化模型，添加截止日期、提醒、与其他用户共享，以及改进客户端 UI。并不一定局限于“待办事项”列表；它们可以是任何类型的列表。但关键是它应该允许我展示 Web 编程的所有主要方面以及如何应用 TDD。

# 使用功能测试来确定一个最小可行应用程序

使用 Selenium 进行的测试可以让我们操作真实的网络浏览器，因此真正让我们从用户的角度看到应用程序*的功能*。这就是为什么它们被称为*功能测试*。

这意味着 FT 可以成为您的应用程序的一种规范。它倾向于跟踪您可能称之为*用户故事*的内容，并且遵循用户如何使用特定功能以及应用程序应如何响应他们的方式。

FT 应具有我们可以遵循的人类可读的故事。我们使用随测试代码附带的注释来明确它。创建新的 FT 时，我们可以首先编写注释，以捕捉用户故事的关键点。由于它们是人类可读的，甚至可以与非程序员分享，作为讨论应用程序要求和功能的一种方式。

TDD 和敏捷或精益软件开发方法经常结合在一起，我们经常谈论的一件事就是最小可行应用程序；我们可以构建的最简单有用的东西是什么？让我们从构建它开始，以便我们可以尽快测试一下。

一个最小可行的待办事项列表只需让用户输入一些待办事项，并在下次访问时记住它们即可。

打开*functional_tests.py*并写一个类似于这样的故事：

functional_tests.py（ch02l001）

```py
from selenium import webdriver

browser = webdriver.Firefox()

# Edith has heard about a cool new online to-do app.
# She goes to check out its homepage
browser.get("http://localhost:8000")

# She notices the page title and header mention to-do lists
assert "To-Do" in browser.title

# She is invited to enter a to-do item straight away

# She types "Buy peacock feathers" into a text box
# (Edith's hobby is tying fly-fishing lures)

# When she hits enter, the page updates, and now the page lists
# "1: Buy peacock feathers" as an item in a to-do list

# There is still a text box inviting her to add another item.
# She enters "Use peacock feathers to make a fly" (Edith is very methodical)

# The page updates again, and now shows both items on her list

# Satisfied, she goes back to sleep

browser.quit()
```

除了将测试写成注释外，你会注意到我已经更新了`assert`来查找“To-Do”这个词，而不是 Django 的“Congratulations”。这意味着我们现在期望测试失败。让我们试着运行它。

首先，启动服务器：

```py
$ python manage.py runserver
```

然后，在另一个终端中运行测试：

```py
$ python functional_tests.py
Traceback (most recent call last):
  File "...goat-book/functional_tests.py", line 10, in <module>
    assert "To-Do" in browser.title
           ^^^^^^^^^^^^^^^^^^^^^^^^
AssertionError
```

这就是我们所谓的*预期失败*，这实际上是个好消息—​虽然不如测试通过那么好，但至少它因正确的原因而失败；我们可以相当有信心我们已正确编写了测试。

# Python 标准库的 unittest 模块

有几个小烦恼我们可能需要处理。首先，“AssertionError”这个消息并不是很有帮助——如果测试能告诉我们实际找到的浏览器标题会更好。另外，桌面上留下了一个 Firefox 窗口，所以最好能自动清理掉它。

一个选项是使用 `assert` 关键字的第二个参数，类似于：

```py
assert "To-Do" in browser.title, f"Browser title was {browser.title}"
```

我们还可以使用 `try/finally` 来清理旧的 Firefox 窗口。

但这些问题在测试中相当常见，在标准库的`unittest`模块中已经有一些现成的解决方案可以使用。让我们来使用它！在 *functional_tests.py* 中：

functional_tests.py (ch02l003)

```py
import unittest
from selenium import webdriver

class NewVisitorTest(unittest.TestCase):  ![1](img/1.png)
    def setUp(self):  ![3](img/3.png)
        self.browser = webdriver.Firefox()  ![4](img/4.png)

    def tearDown(self):  ![3](img/3.png)
        self.browser.quit()

    def test_can_start_a_todo_list(self):  ![2](img/2.png)
        # Edith has heard about a cool new online to-do app.
        # She goes to check out its homepage
        self.browser.get("http://localhost:8000")  ![4](img/4.png)

        # She notices the page title and header mention to-do lists
        self.assertIn("To-Do", self.browser.title)  ![5](img/5.png)

        # She is invited to enter a to-do item straight away
        self.fail("Finish the test!")  ![6](img/6.png)

        [...]

        # Satisfied, she goes back to sleep

if __name__ == "__main__":  ![7](img/7.png)
    unittest.main()  ![7](img/7.png)
```

你可能会注意到这里有几点：

![1](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-1)

测试被组织成类，这些类继承自 `unittest.TestCase`。

![2](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-5)

测试的主体在一个名为 `test_can_start_a_todo_list` 的方法中。任何以 `test_` 开头的方法都是测试方法，并且将由测试运行器运行。你可以在同一个类中拥有多个 `test_` 方法。为我们的测试方法取一个好的描述性名称也是个好主意。

![3](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-2)

`setUp` 和 `tearDown` 是特殊方法，它们在每个测试之前和之后运行。我在这里用它们来启动和停止我们的浏览器。它们有点像 `try/finally`，因为即使在测试过程中出现错误，`tearDown` 也会运行。¹ 不会再有未关闭的 Firefox 窗口了！

![4](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-3)

`browser`，之前是一个全局变量，现在成为了测试类的属性 `self.browser`。这样我们可以在 `setUp`、`tearDown` 和测试方法之间传递它。

![5](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-7)

我们使用 `self.assertIn` 而不是简单的 `assert` 来进行测试断言。`unittest` 提供了许多像这样的辅助函数，如 `assertEqual`、`assertTrue`、`assertFalse` 等等。你可以在 [`unittest` 文档](http://docs.python.org/3/library/unittest.xhtml) 中找到更多信息。

![6](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-8)

`self.fail` 无论如何都会失败，并输出给定的错误消息。我将其用作完成测试的提醒。

![7](img/#co_extending_our_functional_test_using___span_class__keep_together__the_unittest_module__span__CO1-9)

最后，我们有了`if __name__ == '__main__'`子句（如果你之前没见过，这是 Python 脚本检查是否从命令行执行，而不仅仅是被另一个脚本导入的方式）。我们调用`unittest.main()`，它启动`unittest`测试运行器，它将自动查找文件中的测试类和方法并运行它们。

###### 注

如果你读过 Django 的测试文档，你可能看到过叫做`LiveServerTestCase`的东西，并想知道我们现在是否应该使用它。恭喜你阅读了友好的手册！现在`LiveServerTestCase`对现在来说有点复杂，但我保证我会在后面的章节中使用它。

让我们试试我们新改进的 FT！²

```py
$ python functional_tests.py
F
======================================================================
FAIL: test_can_start_a_todo_list
(__main__.NewVisitorTest.test_can_start_a_todo_list)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...goat-book/functional_tests.py", line 18, in
test_can_start_a_todo_list
    self.assertIn("To-Do", self.browser.title)
AssertionError: 'To-Do' not found in 'The install worked successfully!
Congratulations!'

 ---------------------------------------------------------------------
Ran 1 test in 1.747s

FAILED (failures=1)
```

这好看多了，不是吗？它整理了我们的 Firefox 窗口，给了我们一个格式漂亮的报告，显示运行了多少测试和多少失败了，`assertIn`还给了我们一个有用的带有调试信息的错误消息。棒极了！

# 提交

这是进行提交的好时机；这是一个很好的自包含变更。我们扩展了我们的功能测试，包括描述我们设定的任务的注释，我们还重写了它以使用 Python 的`unittest`模块及其各种测试辅助函数。

执行**`git status`**—这应该会告诉你只有*functional_tests.py*这个文件发生了变化。然后执行**`git diff -w`**，它将显示最后一次提交与当前磁盘上的文件之间的差异，使用`-w`表示“忽略空白变化”。

这应该告诉你，*functional_tests.py*发生了相当大的变化：

```py
$ git diff -w
diff --git a/functional_tests.py b/functional_tests.py
index d333591..b0f22dc 100644
--- a/functional_tests.py
+++ b/functional_tests.py
@@ -1,15 +1,24 @@
+import unittest
 from selenium import webdriver

-browser = webdriver.Firefox()

+class NewVisitorTest(unittest.TestCase):
+    def setUp(self):
+        self.browser = webdriver.Firefox()
+
+    def tearDown(self):
+        self.browser.quit()
+
+    def test_can_start_a_todo_list(self):
         # Edith has heard about a cool new online to-do app.
         # She goes to check out its homepage
-browser.get("http://localhost:8000")
+        self.browser.get("http://localhost:8000")

         # She notices the page title and header mention to-do lists
-assert "To-Do" in browser.title
+        self.assertIn("To-Do", self.browser.title)

         # She is invited to enter a to-do item straight away
+        self.fail("Finish the test!")

[...]
```

现在让我们执行：

```py
$ git commit -a
```

`-a`表示“自动将任何更改添加到已跟踪的文件”（即，任何我们之前提交过的文件）。它不会添加任何全新的文件（你必须明确使用`git add`添加它们），但通常，就像这种情况一样，没有新文件，所以这是一个有用的快捷方式。

当编辑器弹出时，请添加一个描述性的提交消息，比如“首次在注释中规定了 FT，并且现在使用单元测试。”

现在我们的 FT 使用了一个真正的测试框架，并且我们已经用占位符注释了我们希望它做什么，我们现在可以非常出色地开始为我们的列表应用编写一些真正的代码了。继续阅读！

¹ 唯一的例外是如果`setUp`中有一个异常，那么`tearDown`就不会运行。

² 你是否无法继续前进，因为你想知道那些*ch02l00x*是什么，就在某些代码清单旁边？它们指的是书本示例库中特定的[提交](https://github.com/hjwp/book-example/commits/chapter_02_unittest)。这都与我书中自己的[测试](https://github.com/hjwp/Book-TDD-Web-Dev-Python/tree/master/tests)有关。你知道的，关于测试的书中的测试。它们当然也有自己的测试。
