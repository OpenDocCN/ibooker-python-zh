# 第三章。使用单元测试测试简单的首页

我们在上一章结束时有一个功能测试失败，告诉我们它希望我们网站的首页标题中有“待办事项”。现在是开始开发我们的应用程序的时候了。在这一章中，我们将构建我们的第一个 HTML 页面，了解 URL 处理，并使用 Django 的视图函数创建 HTTP 请求的响应。

# 我们的第一个 Django 应用程序及我们的第一个单元测试

Django 鼓励您将代码结构化为*应用程序*：理论上，一个项目可以有多个应用程序，您可以使用其他人开发的第三方应用程序，甚至可以在不同项目中重用您自己的应用程序... 尽管我承认我从未真正做到过！不过，应用程序是保持代码组织良好的好方法。

让我们为我们的待办事项列表创建一个应用程序：

```py
$ python manage.py startapp lists
```

这将在 *manage.py* 旁边创建一个名为 *lists* 的文件夹，并在其中创建一些占位文件，如模型、视图以及对我们非常感兴趣的测试：

```py
.
├── db.sqlite3
├── functional_tests.py
├── lists
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
└── superlists
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

# 单元测试及其与功能测试的区别

就像我们给许多事物贴上的标签一样，单元测试和功能测试之间的界限有时可能会变得有些模糊。不过，基本的区别在于，功能测试从外部，从用户的角度测试应用程序。单元测试从内部，从程序员的角度测试应用程序。

我演示的 TDD 方法使用两种类型的测试来驱动我们应用程序的开发，并确保其正确性。我们的工作流程将看起来有点像这样：

1.  我们从编写*功能测试*开始，描述我们的新功能的一个典型示例，从用户的角度来看。

1.  一旦我们有一个功能测试失败，我们开始考虑如何编写代码来使其通过（或至少通过当前的失败）。现在我们使用一个或多个*单元测试*来定义我们希望我们的代码如何运行——我们编写的每一行生产代码都应该由（至少）一个我们的单元测试来测试。

1.  一旦我们有一个失败的单元测试，我们编写尽可能少的*应用程序代码*，只要能让单元测试通过即可。我们可能会在步骤 2 和步骤 3 之间迭代几次，直到我们认为功能测试将进展一点。

1.  现在我们可以重新运行我们的功能测试，看它们是否通过或者是否有所进展。这可能会促使我们编写一些新的单元测试，编写一些新代码，等等。

1.  一旦我们确信核心功能端到端运行正常，我们可以扩展测试以覆盖更多的排列组合和边缘情况，现在只使用单元测试。

你可以看到，从始至终，功能测试在高层驱动我们的开发，而单元测试在低层驱动我们的开发。

功能测试的目标不是覆盖应用程序行为的每一个细节，它们是为了确保所有东西都正确连接起来。单元测试则是详尽检查所有低级细节和边缘情况。

###### 注意

功能测试应该帮助你构建一个真正可用的应用程序，并确保你永远不会意外地破坏它。单元测试应该帮助你编写干净和无 bug 的代码。

现在足够理论了 — 让我们看看它在实践中的表现。

表 3-1\. FTs vs 单元测试

| FTs | 单元测试 |
| --- | --- |
| 每个功能/用户故事一个测试 | 每个功能多个测试 |
| 用户角度的测试 | 程序员角度的代码测试 |
| 可以测试 UI “真正” 工作 | 测试内部，单个函数或类 |
| 确保所有东西正确连接在一起的信心 | 可以详尽检查排列组合，细节，边缘情况 |

# Django 中的单元测试

让我们看看如何为我们的首页视图编写一个单元测试。打开 *lists/tests.py* 中的新文件，你会看到类似这样的内容：

lists/tests.py

```py
from django.test import TestCase

# Create your tests here.
```

Django 已经很贴心地建议我们使用它提供的 `TestCase` 的特殊版本。它是标准的 `unittest.TestCase` 的增强版本，带有一些额外的 Django 特定功能，我们将在接下来的几章中了解到。

你已经看到 TDD 周期包括从失败的测试开始，然后编写代码使其通过。那么，在我们甚至达到那一步之前，我们希望知道我们编写的单元测试一定会被我们的自动化测试运行器运行，不管它是什么。在 *functional_tests.py* 的情况下，我们直接运行它，但这个由 Django 创建的文件有点像魔术。所以，为了确保，让我们写一个故意愚蠢的失败测试：

lists/tests.py (ch03l002)

```py
from django.test import TestCase

class SmokeTest(TestCase):
    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
```

现在让我们调用这个神秘的 Django 测试运行器。像往常一样，这是一个 *manage.py* 命令：

```py
$ python manage.py test
Creating test database for alias 'default'...
Found 1 test(s).
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_bad_maths (lists.tests.SmokeTest.test_bad_maths)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...goat-book/lists/tests.py", line 6, in test_bad_maths
    self.assertEqual(1 + 1, 3)
AssertionError: 2 != 3

 ---------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

很好。机制似乎在工作。这是一个提交的好时机：

```py
$ git status  # should show you lists/ is untracked
$ git add lists
$ git diff --staged  # will show you the diff that you're about to commit
$ git commit -m "Add app for lists, with deliberately failing unit test"
```

正如你无疑猜到的那样，`-m` 标志允许你在命令行传递提交消息，这样你就不需要使用编辑器了。选择如何使用 Git 命令行取决于你；我只是展示我见过的主要方式。对我来说，VCS 卫生的主要部分是：*确保在提交之前始终审查即将提交的内容*。

# Django 的 MVC、URL 和视图函数

Django 沿着经典的 *Model-View-Controller* (MVC) 模式结构化。嗯，*大体上* 是这样。它确实有模型，但 Django 称为视图的东西实际上是控制器，而视图部分实际上由模板提供，但你可以看到总体思路是一致的！

如果你感兴趣，你可以查看 [Django FAQ](https://docs.djangoproject.com/en/4.2/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names) 中讨论的更细节的内容。

无论如何，就像任何 Web 服务器一样，Django 的主要工作是决定当用户请求站点上特定的 URL 时该做什么。Django 的工作流程大致如下：

1.  一个 HTTP *请求* 来自于特定的 *URL*。

1.  Django 使用一些规则来决定哪个 *视图* 函数应该处理请求（这称为 *解析* URL）。

1.  视图函数处理请求并返回一个 HTTP *响应*。

所以，我们想测试两件事：

+   我们能让这个视图函数返回我们需要的 HTML 吗？

+   我们能告诉 Django 在我们请求站点根目录（“/”）时使用这个视图函数吗？

让我们从第一个开始。

# 测试一个视图

打开 *lists/tests.py*，将我们愚蠢的测试更改为类似这样的内容：

lists/tests.py (ch03l003)

```py
from django.test import TestCase
from django.http import HttpRequest  ![1](img/1.png)
from lists.views import home_page

class HomePageTest(TestCase):
    def test_home_page_returns_correct_html(self):
        request = HttpRequest()  ![1](img/1.png)
        response = home_page(request)  ![2](img/2.png)
        html = response.content.decode("utf8")  ![3](img/3.png)
        self.assertIn("<title>To-Do lists</title>", html)  ![4](img/4.png)
        self.assertTrue(html.startswith("<html>"))  ![5](img/5.png)
        self.assertTrue(html.endswith("</html>"))  ![5](img/5.png)
```

这个新测试中发生了什么？嗯，记住，一个视图函数以 HTTP 请求作为输入，并生成一个 HTTP 响应。所以，为了测试：

![1](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO1-1)

我们导入 `HttpRequest` 类，这样我们就可以在测试中创建一个请求对象。这是当用户的浏览器请求页面时 Django 会创建的对象。

![2](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO1-3)

我们将 `HttpRequest` 对象传递给我们的 `home_page` 视图，这给了我们一个响应。你可能不会感到惊讶，响应是一个名为 `HttpResponse` 的类的实例。

![3](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO1-4)

然后，我们提取响应的 `.content`。这些是原始字节，即将发送到用户浏览器的 0 和 1。我们调用 `.decode()` 将它们转换为 HTML 字符串，这些将发送给用户的内容。

![4](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO1-5)

现在我们可以进行一些断言：我们知道我们希望在其中的某处有一个 html `<title>` 标签，标签中包含“待办事项列表”这几个字——因为这是我们在功能测试中指定的内容。

![5](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO1-6)

然后我们可以做一个粗略的检查，确认它是有效的 html，通过检查它是否以 `<html>` 标签开头，并在结尾处关闭。

所以，你认为当我们运行测试时会发生什么？

```py
$ python manage.py test
Found 1 test(s).
System check identified no issues (0 silenced).
E
======================================================================
ERROR: lists.tests (unittest.loader._FailedTest.lists.tests)
 ---------------------------------------------------------------------
ImportError: Failed to import test module: lists.tests
Traceback (most recent call last):
[...]
  File "...goat-book/lists/tests.py", line 3, in <module>
    from lists.views import home_page
ImportError: cannot import name 'home_page' from 'lists.views'
```

这是一个非常可预测且不太有趣的错误：我们试图导入尚未编写的东西。但这仍然是好消息—​对于 TDD 来说，预料之中的异常算作是预期的失败。因为我们既有一个失败的功能测试又有一个失败的单元测试，我们得到了测试山羊的充分祝福，可以继续编码。

## 终于！我们真正写一些应用代码！

这是令人兴奋的，不是吗？请注意，TDD 意味着长时间的期待只能逐渐化解，并且只能通过微小的增量来解决。特别是因为我们在学习阶段，刚刚开始，我们只允许自己每次只改变（或添加）一行代码—​每次，我们只进行最小的变更来解决当前的测试失败。

我故意夸张一下，但是我们当前的测试失败是什么？我们无法从`lists.views`导入`home_page`？好的，让我们解决这个问题—​只解决这个问题。在*lists/views.py*中：

lists/views.py（ch03l004）

```py
from django.shortcuts import render

# Create your views here.
home_page = None
```

*“你一定在开玩笑！”* 我听到你说。

我可以听到你的声音，因为这正是我以前（带有强烈感情地）对我的同事展示 TDD 时所说的话。好吧，忍耐一下，我们稍后再讨论这是否有些过火。但现在，让自己跟随一下，即使有些恼怒，看看我们的测试是否可以帮助我们一次一小步地编写正确的代码。

让我们再次运行测试：

```py
[...]
  File "...goat-book/lists/tests.py", line 9, in
test_home_page_returns_correct_html
    response = home_page(request)
               ^^^^^^^^^^^^^^^^^^
TypeError: 'NoneType' object is not callable
```

我们仍然收到错误消息，但情况有所改变。不再是导入错误，我们的测试告诉我们我们的`home_page`“函数”不可调用。这给了我们正当理由将其从`None`改为实际的函数。在最微小的细节层面，每一次代码变更都可以由测试驱动！

回到*lists/views.py*：

lists/views.py（ch03l005）

```py
from django.shortcuts import render

def home_page():
    pass
```

再次，我们只需进行最小、最愚蠢的变更，精确解决当前的测试失败。我们的测试希望得到可调用的内容，因此我们提供了可能的最简单可调用的东西，一个不接受任何参数并且不返回任何内容的函数。

让我们再次运行测试，看看它们的反应：

```py
    response = home_page(request)
               ^^^^^^^^^^^^^^^^^^
TypeError: home_page() takes 0 positional arguments but 1 was given
```

再次，我们的错误消息略有变化，并引导我们修复接下来出现的问题。

## 单元测试/代码循环

现在我们可以开始进入 TDD 的*单元测试/代码循环*：

1.  在终端中运行单元测试，看看它们如何失败。

1.  在编辑器中，进行最小化的代码变更来解决当前的测试失败。

然后重复！

我们对代码正确性越紧张，每次代码变更就越小、越简单—​这个想法是确保每一行代码都经过测试的验证。

这看起来可能很繁琐，起初确实是这样。但一旦你进入了节奏，即使采取微小的步骤，你也会发现自己编码速度很快—​这是我们在工作中编写所有生产代码的方式。

让我们看看我们能多快地进行这个循环：

+   最小化代码变更：

    lists/views.py（ch03l006）

    ```py
    def home_page(request):
        pass
    ```

+   测试：

    ```py
        html = response.content.decode("utf8")
               ^^^^^^^^^^^^^^^^
    AttributeError: 'NoneType' object has no attribute 'content'
    ```

+   代码—​我们使用`django.http.HttpResponse`，正如预期的那样：

    lists/views.py (ch03l007)

    ```py
    from django.http import HttpResponse

    def home_page(request):
        return HttpResponse()
    ```

+   再次测试：

    ```py
    AssertionError: '<title>To-Do lists</title>' not found in ''
    ```

+   再来一段代码：

    lists/views.py (ch03l008)

    ```py
    def home_page(request):
        return HttpResponse("<title>To-Do lists</title>")
    ```

+   再次测试：

    ```py
        self.assertTrue(html.startswith("<html>"))
    AssertionError: False is not true
    ```

+   再来一段代码：

    lists/views.py (ch03l009)

    ```py
    def home_page(request):
        return HttpResponse("<html><title>To-Do lists</title>")
    ```

+   测试—​快了吗？

    ```py
        self.assertTrue(html.endswith("</html>"))
    AssertionError: False is not true
    ```

+   再努把力：

    lists/views.py (ch03l010)

    ```py
    def home_page(request):
        return HttpResponse("<html><title>To-Do lists</title></html>")
    ```

+   当然？

    ```py
    $ python manage.py test
    Creating test database for alias 'default'...
    Found 1 test(s).
    System check identified no issues (0 silenced).
    .
     ---------------------------------------------------------------------
    Ran 1 test in 0.001s

    OK
    Destroying test database for alias 'default'...
    ```

好极了！我们有史以来的第一个单元测试通过了！这是如此重要，我认为值得提交：

```py
$ git diff  # should show changes to tests.py, and views.py
$ git commit -am "First unit test and view function"
```

那就是我将展示的最后一种 `git commit` 变体了，`a` 和 `m` 标志一起使用，它将所有更改添加到已跟踪文件并使用命令行中的提交消息。

###### 警告

`git commit -am` 是最快的组合，但也给出了关于正在提交的内容最少的反馈，所以确保你之前已经执行了 `git status` 和 `git diff`，并且清楚即将进行的更改。 

# 我们的功能测试告诉我们我们还没有完成。

我们的单元测试通过了，所以让我们回到运行我们的功能测试，看看我们是否有所进展。如果开发服务器还没有运行，请不要忘记重新启动它。

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
Ran 1 test in 1.609s

FAILED (failures=1)
```

看起来有些不太对劲。这就是我们进行功能测试的原因！

你还记得在本章开头，我们说过我们需要做两件事，首先是创建一个视图函数来为请求产生响应，其次是告诉服务器哪些函数应该响应哪些 URL 吗？多亏了我们的 FT，我们被提醒我们仍然需要做第二件事。

我们如何编写 URL 解析的测试呢？目前我们只是直接导入并调用视图函数进行测试。但我们想要测试 Django 堆栈的更多层。Django，像大多数 Web 框架一样，提供了一个工具来做这件事，称为[Django 测试客户端](https://docs.djangoproject.com/en/4.2/topics/testing/tools/#the-test-client)。

让我们看看如何使用它，通过向我们的单元测试添加第二个替代测试：

lists/tests.py (ch03l011)

```py
class HomePageTest(TestCase):
    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        html = response.content.decode("utf8")
        self.assertIn("<title>To-Do lists</title>", html)
        self.assertTrue(html.startswith("<html>"))
        self.assertTrue(html.endswith("</html>"))

    def test_home_page_returns_correct_html_2(self):
        response = self.client.get("/")  ![1](img/1.png)
        self.assertContains(response, "<title>To-Do lists</title>")  ![2](img/2.png)
```

![1](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO2-1)

我们可以通过 `self.client` 访问测试客户端，它在任何使用 `django.test.TestCase` 的测试中都可用。它提供了像 `.get()` 这样的方法，模拟浏览器发出 http 请求，并将 URL 作为其第一个参数。我们使用它来代替手动创建请求对象并直接调用视图函数

![2](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO2-2)

Django 还提供了一些断言辅助函数，比如 `assertContains`，它们可以帮助我们避免手动提取和解码响应内容，并且还有一些其他好处，正如我们将看到的那样。

让我们看看它是如何工作的：

```py
$ python manage.py test
Found 2 test(s).
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.F
======================================================================
FAIL: test_home_page_returns_correct_html_2
(lists.tests.HomePageTest.test_home_page_returns_correct_html_2)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...goat-book/lists/tests.py", line 17, in
test_home_page_returns_correct_html_2
    self.assertContains(response, "<title>To-Do lists</title>")
[...]
AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404
(expected 200)

 ---------------------------------------------------------------------
Ran 2 tests in 0.004s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

嗯，关于 404 的问题？让我们深入了解一下。

# 阅读回溯

让我们花一点时间来谈谈如何阅读回溯，因为这是我们在 TDD 中经常要做的事情。你很快就会学会扫描它们并收集相关线索：

```py
======================================================================
FAIL: test_home_page_returns_correct_html_2  ![2](img/2.png)
(lists.tests.HomePageTest.test_home_page_returns_correct_html_2)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...goat-book/lists/tests.py", line 17, in
test_home_page_returns_correct_html_2
    self.assertContains(response, "<title>To-Do lists</title>")  ![3](img/3.png)
  File ".../django/test/testcases.py", line 647, in assertContains
    text_repr, real_count, msg_prefix = self._assert_contains(
                                        ^^^^^^^^^^^^^^^^^^^^^^  ![4](img/4.png)
  File ".../django/test/testcases.py", line 610, in _assert_contains
    self.assertEqual(
AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404  ![1](img/1.png)
(expected 200)

 ---------------------------------------------------------------------
[...]
```

![1](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO3-4)

通常，你首先要查看的地方就是*错误本身*。有时这就是你需要看到的一切，它会让你立即识别问题。但有时，就像在这种情况下一样，情况并不是那么明显。

![2](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO3-1)

下一步要仔细检查的是：*哪个测试失败了？* 它确实是我们期望的那个测试吗——也就是说，我们刚刚编写的测试吗？在这种情况下，答案是肯定的。

![3](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO3-2)

接着，我们寻找导致失败的*测试代码*所在的位置。我们从回溯的顶部开始向下查找，查找测试文件的文件名，以检查是哪个测试函数，以及失败来自哪一行代码。在这种情况下，是我们调用`assertContains`方法的那一行。

![4](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO3-3)

在 Python 3.11 及更高版本中，你还可以查看小尖括号组成的字符串，它们试图告诉你异常来自哪里。这对于意外异常比我们现在的断言失败更有用。

通常还有第五步，我们会进一步查找*我们自己的应用代码*中是否涉及该问题。在这种情况下，这都是 Django 代码，但我们将在本书的后面看到许多这样的第五步示例。

汇总一下，我们将回溯解释为告诉我们，在我们尝试对响应内容进行断言时，Django 的测试助手失败，因为它们无法执行该操作，因为响应是 HTML 404“未找到”错误，而不是正常的 200 OK 响应。

换句话说，Django 尚未配置为响应对我们站点根 URL（“/”）的请求。现在让我们来实现这个功能。

# urls.py

Django 使用一个名为*urls.py*的文件来将 URL 映射到视图函数。这种映射也称为*路由*。整个站点的主*urls.py*位于*superlists*文件夹中。让我们去看一下：

superlists/urls.py

```py
"""
URL configuration for superlists project.

The `urlpatterns` list routes URLs to views. For more information please see:
 https://docs.djangoproject.com/en/4.2/topics/http/urls/
Examples:
Function views
 1\. Add an import:  from my_app import views
 2\. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
 1\. Add an import:  from other_app.views import Home
 2\. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
 1\. Import the include() function: from django.urls import include, path
 2\. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
"""
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path("admin/", admin.site.urls),
]
```

###### 警告

如果你的*urls.py*看起来不同，或者提到了一个名为`url()`而不是`path()`的函数，那是因为你安装了错误版本的 Django。本书是针对 Django v4 编写的。再看一下先决条件和假设部分，并在继续之前获取正确的版本。

通常情况下，Django 提供了大量有用的注释和默认建议。实际上，第一个示例就是我们想要的！让我们使用它，并进行一些小的更改。

superlists/urls.py (ch03l012)

```py
from django.urls import path  ![1](img/1.png)
from lists.views import home_page  ![2](img/2.png)

urlpatterns = 
    path("", home_page, name="home"),  ![3
]
```

![1](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO4-1)

无需从`django.contrib`导入`admin`。Django 的管理站点非常棒，但这是另一本书的话题。

![2](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO4-2)

但是我们将导入我们的主页视图函数。

![3](img/#co_testing_a_simple_home_page_with___span_class__keep_together__unit_tests__span__CO4-3)

我们将其连接在这里，作为`urlpatterns`全局变量中的`path()`条目。Django 会从所有 URL 中去掉开头的斜杠，所以`"/url/path/to"`变成了`"url/path/to"`，而基础 URL 就是空字符串`""`。因此，这个配置表示：“基础 URL 应该指向我们的主页视图”

现在我们可以再次运行我们的单元测试，使用**`python manage.py test`**命令：

```py
[...]
..
 ---------------------------------------------------------------------
Ran 2 tests in 0.003s

OK
```

万岁！

是时候稍作整理了。我们不需要两个单独的测试，让我们将所有内容从直接调用视图函数的低级测试中移出，放入使用 Django 测试客户端的测试中：

lists/tests.py（ch03l013）

```py
class HomePageTest(TestCase):
    def test_home_page_returns_correct_html(self):
        response = self.client.get("/")
        self.assertContains(response, "<title>To-Do lists</title>")
        self.assertContains(response, "<html>")
        self.assertContains(response, "</html>")
```

但现在真相大白了，我们的功能测试会通过吗？

```py
$ python functional_tests.py
[...]
======================================================================
FAIL: test_can_start_a_todo_list
(__main__.NewVisitorTest.test_can_start_a_todo_list)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...goat-book/functional_tests.py", line 21, in
test_can_start_a_todo_list
    self.fail("Finish the test!")
AssertionError: Finish the test!
```

失败了？什么？哦，这只是我们的小提醒？是的？是的！我们有一个网页！

哎呀。嗯，*我*认为这是章节的一个激动人心的结尾。你可能还有点困惑，也许急于听到所有这些测试的理由，别担心，一切都会有的，但我希望你在最后感受到了一丝兴奋。

只需稍作提交，冷静下来，回顾一下我们所涵盖的内容：

```py
$ git diff  # should show our modified test in tests.py, and the new config in urls.py
$ git commit -am "url config, map / to home_page view"
```

那真是一个精彩的章节！为什么不尝试输入`git log`命令，可能会使用`--oneline`选项，来回顾我们的活动：

```py
$ git log --oneline
a6e6cc9 url config, map / to home_page view
450c0f3 First unit test and view function
ea2b037 Add app for lists, with deliberately failing unit test
[...]
```

还不错——我们涵盖了：

+   开始一个 Django 应用程序

+   Django 单元测试运行器

+   功能测试和单元测试之间的区别

+   Django 视图函数、请求和响应对象

+   Django URL 解析和*urls.py*

+   Django 测试客户端

+   并从视图返回基本的 HTML。
