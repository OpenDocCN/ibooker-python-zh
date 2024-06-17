# 第六章：改进功能测试：确保隔离和移除“巫术”睡眠

在我们深入解决全局列表问题之前，让我们先处理一些日常事务。在上一章的结尾，我们注意到不同的测试运行相互干扰，因此我们会解决这个问题。此外，我对代码中到处都是的`time.sleep`不太满意，它们似乎有些不科学，所以我们会用更可靠的方法替换它们。

这两个变化将使我们朝着测试“最佳实践”迈进，使我们的测试更加确定性和可靠。

# 在功能测试中确保测试隔离

我们在上一章结束时遇到了一个经典的测试问题：如何确保测试之间的*隔离*。我们的功能测试每次运行后都会在数据库中留下列表项，这会影响下次运行测试时的结果。

当我们运行 *单元* 测试时，Django 测试运行器会自动创建一个全新的测试数据库（与真实数据库分开），它可以在每个单独的测试运行之前安全地重置，然后在结束时丢弃。但我们的功能测试目前运行在“真实”的数据库 *db.sqlite3* 上。

解决这个问题的一种方法是自己“摸索”解决方案，并向 *functional_tests.py* 添加一些清理代码。`setUp`和`tearDown`方法非常适合这种情况。

但由于这是一个常见的问题，Django 提供了一个名为`LiveServerTestCase`的测试类来解决这个问题。它将自动创建一个测试数据库（就像在单元测试运行中一样），并启动一个开发服务器，供功能测试运行。尽管作为一个工具它有一些限制，我们稍后需要解决这些限制，但在这个阶段它非常有用，所以让我们来看看它。

`LiveServerTestCase` 期望由 Django 测试运行器使用 *manage.py* 运行，它将运行任何以 *test_* 开头的文件中的测试。为了保持整洁，让我们为我们的功能测试创建一个文件夹，使它看起来像一个应用程序。Django 需要的只是一个有效的 Python 包目录（即其中包含一个 *___init___.py* 文件）：

```py
$ mkdir functional_tests
$ touch functional_tests/__init__.py
```

现在我们想要将我们的功能测试从名为 *functional_tests.py* 的独立文件移动到 `functional_tests` 应用程序的 *tests.py* 中。我们使用 **`git mv`** 让 Git 知道这是同一个文件，并且应该有一个单一的历史记录。

```py
$ git mv functional_tests.py functional_tests/tests.py
$ git status # shows the rename to functional_tests/tests.py and __init__.py
```

此时，你的目录树应该是这样的：

```py
.
├── db.sqlite3
├── functional_tests
│   ├── __init__.py
│   └── tests.py
├── lists
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── 0002_item_text.py
│   │   └── __init__.py
│   ├── models.py
│   ├── templates
│   │   └── home.xhtml
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

*functional_tests.py* 已经不见了，变成了 *functional_tests/tests.py*。现在，每当我们想运行功能测试时，不再运行`python functional_tests.py`，而是使用`python manage.py test functional_tests`。

###### 注意

你可以将功能测试混合到 `lists` 应用的测试中。我倾向于保持它们分开，因为功能测试通常涉及跨不同应用的横切关注点。FT 应该从用户的角度看事情，而你的用户并不关心你是如何在不同应用之间分割工作的！

现在让我们编辑 *functional_tests/tests.py* 并修改我们的 `NewVisitorTest` 类以使其使用 `LiveServerTestCase`：

functional_tests/tests.py (ch06l001)

```py
from django.test import LiveServerTestCase
from selenium import webdriver
[...]

class NewVisitorTest(LiveServerTestCase):
    def setUp(self):
        [...]
```

接下来，不再将访问本地主机端口 8000 硬编码，`LiveServerTestCase` 给了我们一个叫做 `live_server_url` 的属性：

functional_tests/tests.py (ch06l002)

```py
    def test_can_start_a_todo_list(self):
        # Edith has heard about a cool new online to-do app.
        # She goes to check out its homepage
        self.browser.get(self.live_server_url)
```

如果希望，我们也可以从末尾移除 `if __name__ == '__main__'`，因为我们将使用 Django 测试运行器来启动 FT。

TODO — 从这里修复

现在我们可以通过告诉 Django 测试运行器仅运行我们新的 `functional_tests` 应用的测试来运行我们的功能测试：

```py
$ python manage.py test functional_tests
Creating test database for alias 'default'...
Found 1 test(s).
System check identified no issues (0 silenced).
.
 ---------------------------------------------------------------------
Ran 1 test in 10.519s

OK
Destroying test database for alias 'default'...
```

FT 依然通过，让我们放心，我们的重构没有出问题。你还会注意到，如果你再次运行测试，之前测试留下的旧列表项都不见了，系统已经自我清理完毕。成功！我们应该将其作为一个原子更改提交：

```py
$ git status # functional_tests.py renamed + modified, new __init__.py
$ git add functional_tests
$ git diff --staged
$ git commit  # msg eg "make functional_tests an app, use LiveServerTestCase"
```

## 仅运行单元测试

现在如果我们运行 `manage.py test`，Django 将运行功能测试和单元测试：

```py
$ python manage.py test
Creating test database for alias 'default'...
Found 7 test(s).
System check identified no issues (0 silenced).
.......
 ---------------------------------------------------------------------
Ran 7 tests in 10.859s

OK
Destroying test database for alias 'default'...
```

为了仅运行单元测试，我们可以指定只运行 `lists` 应用的测试：

```py
$ python manage.py test lists
Creating test database for alias 'default'...
Found 6 test(s).
System check identified no issues (0 silenced).
......
 ---------------------------------------------------------------------
Ran 6 tests in 0.009s

OK
Destroying test database for alias 'default'...
```

# 附注：升级 Selenium 和 Geckodriver

当我今天再次运行这一章时，我发现当我试图运行它们时 FT 被挂起了。

结果 Firefox 在夜间自动更新了，我的 Selenium 和 Geckodriver 版本也需要升级。快速访问 [geckodriver 发布页面](https://github.com/mozilla/geckodriver/releases) 确认有新版本发布。所以需要进行一些下载和升级：

+   首先快速执行 `pip install --upgrade selenium`。

+   然后快速下载新版的 geckodriver。

+   我保存了旧版本的备份副本，并将新版本放在了 `PATH` 中的某个地方。

+   并通过 `geckodriver --version` 快速检查确认新版本已经准备就绪。

然后 FT 又回到了我预期的运行方式。

没有特别的原因会让它在书中的这一点发生；实际上，对你来说现在发生的可能性很小，但某个时候可能会发生，而且既然我们在做一些清理工作，这似乎是一个好地方来谈谈它。

这是使用 Selenium 时必须忍受的事情之一。尽管在 CI 服务器上可能固定浏览器和 Selenium 版本（例如），但是浏览器版本在真实世界中是不断变化的，你需要跟上你的用户的步伐。

###### 注意

如果你的 FT 出现了奇怪的情况，尝试升级 Selenium 总是值得的。

现在回到我们的常规编程。

# 关于隐式等待和显式等待，以及神秘的   关于隐式等待和显式等待，以及巫术式的`time.sleep`。

让我们来谈谈我们功能测试中的`time.sleep`：

functional_tests/tests.py

```py
        # When she hits enter, the page updates, and now the page lists
        # "1: Buy peacock feathers" as an item in a to-do list table
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)

        self.check_for_row_in_list_table("1: Buy peacock feathers")
```

这就是所谓的“显式等待”。这与“隐式等待”相对：在某些情况下，Selenium 会试图在你认为页面正在加载时“自动”等待。它甚至提供了一种名为`implicitly_wait`的方法，让你控制如果你请求的元素似乎还不在页面上，它会等多久。

实际上，在第一版中，我完全可以依赖隐式等待。问题在于隐式等待总是有些不稳定，随着 Selenium 4 的发布，隐式等待默认被禁用。同时，Selenium 团队普遍认为隐式等待只是一个坏主意， [应该避免使用](https://www.selenium.dev/documentation/webdriver/waits/)。

因此，这个版本一开始就有显式等待。但问题是那些`time.sleep`有自己的问题。

目前我们正在等待一秒钟，但谁又能说这是正确的时间呢？对于我们在自己机器上运行的大多数测试，一秒钟时间太长了，这将极大地减慢我们的功能测试运行速度。0.1 秒就足够了。但问题是，如果你将其设置得太低，每隔一段时间你会因为某种原因，笔记本电脑刚好运行得慢而导致虚假失败。即使是 1 秒，也不能完全确定不会出现随机失败，这些随机失败并不表示真正的问题，测试中的假阳性是真正的烦恼（在[马丁·福勒的文章](https://martinfowler.com/articles/nonDeterminism.xhtml)中有更多内容）。

小贴士：意外的`NoSuchElementException`和`StaleElementException`错误通常是你需要显式等待的信号。

所以让我们用一个工具替换我们的`sleeps`，它只会等待所需的时间，最长到一个合适的超时时间以捕捉任何故障。我们将`check_for_row_in_list_table`重命名为`wait_for_row_in_list_table`，并添加一些轮询/重试逻辑：

functional_tests/tests.py（ch06l004）

```py
[...]
from selenium.common.exceptions import WebDriverException
import time

MAX_WAIT = 5  ![1](img/1.png)

class NewVisitorTest(LiveServerTestCase):
    def setUp(self):
        [...]
    def tearDown(self):
        [...]

    def wait_for_row_in_list_table(self, row_text):
        start_time = time.time()
        while True:  ![2](img/2.png)
            try:
                table = self.browser.find_element(By.ID, "id_list_table")  ![3](img/3.png)
                rows = table.find_elements(By.TAG_NAME, "tr")
                self.assertIn(row_text, [row.text for row in rows])
                return  ![4](img/4.png)
            except (AssertionError, WebDriverException):  ![5](img/5.png)
                if time.time() - start_time > MAX_WAIT:  ![6](img/6.png)
                    raise  ![6](img/6.png)
                time.sleep(0.5)  ![5](img/5.png)
```

![1](img/#co_improving_functional_tests__ensuring_isolation_and_removing_voodoo_sleeps_CO1-1)

我们将使用一个常量`MAX_WAIT`来设置我们准备等待的最大时间。5 秒应该足够捕捉任何故障或随机延迟。

![2](img/#co_improving_functional_tests__ensuring_isolation_and_removing_voodoo_sleeps_CO1-2)

这里是循环，除非我们到达两种可能的退出路线之一，否则它将一直运行。

![3](img/#co_improving_functional_tests__ensuring_isolation_and_removing_voodoo_sleeps_CO1-3)

这是我们从方法旧版本中提取的三行断言。

![4](img/#co_improving_functional_tests__ensuring_isolation_and_removing_voodoo_sleeps_CO1-4)

如果我们通过了它们，并且我们的断言通过，我们将从函数中返回并退出循环。

![5](img/#co_improving_functional_tests__ensuring_isolation_and_removing_voodoo_sleeps_CO1-5)

但是如果我们捕获到异常，我们会等待一小段时间然后循环重试。我们想要捕获两种类型的异常：`WebDriverException`，当页面尚未加载并且 Selenium 无法在页面上找到表元素时，以及`AssertionError`，当表存在，但可能是页面重新加载前的表，因此还没有我们的行。

![6](img/#co_improving_functional_tests__ensuring_isolation_and_removing_voodoo_sleeps_CO1-6)

这是我们的第二个逃生路线。如果我们到达这一点，那意味着我们的代码每次尝试都会引发异常，直到超过超时时间。所以这一次，我们重新引发异常，并让它冒泡到我们的测试中，很可能最终会出现在我们的回溯中，告诉我们测试失败的原因。

您是否认为这段代码有点丑陋，让人有点难以理解我们到底在做什么？我同意。稍后（[Link to Come]），我们将重构一个通用的`wait_for`辅助函数，将时间和重新引发逻辑与测试断言分开。但我们会等到我们需要在多个地方使用它。

###### 注意

如果您之前使用过 Selenium，您可能知道它有一些[等待的辅助函数](https://www.selenium.dev/documentation/webdriver/waits/#explicit-wait)。不过，我对它们不是很感冒，虽然真的没有任何客观理由。在本书的过程中，我们将构建一些等待的辅助工具，我认为这将产生漂亮、可读的代码，但当然您应该在自己的时间内查看自制的 Selenium 等待，并看看您是否更喜欢它们。

现在我们可以重命名我们的方法调用，并移除那些神秘的`time.sleep`：

functional_tests/tests.py (ch06l005)

```py
    [...]
    # When she hits enter, the page updates, and now the page lists
    # "1: Buy peacock feathers" as an item in a to-do list table
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table("1: Buy peacock feathers")

    # There is still a text box inviting her to add another item.
    # She enters "Use peacock feathers to make a fly"
    # (Edith is very methodical)
    inputbox = self.browser.find_element(By.ID, "id_new_item")
    inputbox.send_keys("Use peacock feathers to make a fly")
    inputbox.send_keys(Keys.ENTER)

    # The page updates again, and now shows both items on her list
    self.wait_for_row_in_list_table("1: Buy peacock feathers")
    self.wait_for_row_in_list_table("2: Use peacock feathers to make a fly")
    [...]
```

然后重新运行测试：

```py
$ python manage.py test
Creating test database for alias 'default'...
Found 7 test(s).
System check identified no issues (0 silenced).
.......
 ---------------------------------------------------------------------
Ran 7 tests in 4.552s

OK
Destroying test database for alias 'default'...
```

哦耶，我们又通过了，并且注意我们的执行时间减少了几秒钟。现在可能不算太多，但这一切都会累积起来。

为了确认我们已经做对了事情，让我们故意破坏测试的几种方式，并查看一些错误。首先，让我们检查一下，如果我们搜索一些永远不会出现的行文本，我们会得到正确的错误：

functional_tests/tests.py (ch06l006)

```py
def wait_for_row_in_list_table(self, row_text):
    [...]
        rows = table.find_elements(By.TAG_NAME, "tr")
        self.assertIn("foo", [row.text for row in rows])
        return
```

我们看到我们仍然得到了一个很好的自说明的测试失败消息：

```py
    self.assertIn("foo", [row.text for row in rows])
AssertionError: 'foo' not found in ['1: Buy peacock feathers']
```

###### 注意

您是否对等待测试失败等待 5 秒感到有点无聊？这是显式等待的一个缺点。在等待足够长时间以确保小故障不会干扰您与等待时间过长以至于期望失败令人痛苦之间存在着棘手的权衡。使 MAX_WAIT 可配置，以便在本地开发时快速，在持续集成（CI）服务器上更为保守，可能是一个不错的主意。请参阅[Link to Come]，了解持续集成的简介。

让我们把它改回原样，然后破坏其他东西：

functional_tests/tests.py (ch06l007)

```py
    try:
        table = self.browser.find_element(By.ID, "id_nothing")
        rows = table.find_elements(By.TAG_NAME, "tr")
        self.assertIn(row_text, [row.text for row in rows])
        return
    [...]
```

确实，我们得到了当页面不包含我们寻找的元素时的错误：

```py
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate
element: [id="id_nothing"]
```

一切看起来都井然有序。让我们把我们的代码恢复到应该的状态，并进行最后一次测试运行：

```py
$ python manage.py test
[...]
OK
```

很好。随着这小插曲的结束，让我们继续为多个列表使我们的应用程序实际工作。不要忘记先提交！
