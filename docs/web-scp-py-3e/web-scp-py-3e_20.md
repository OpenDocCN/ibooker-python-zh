# 第十八章。使用抓取器测试您的网站

在使用具有大型开发堆栈的 Web 项目时，通常只有“堆栈”的“后端”部分会定期进行测试。今天大多数编程语言（包括 Python）都有某种类型的测试框架，但网站前端通常被排除在这些自动化测试之外，尽管它们可能是项目中唯一面向客户的部分。

问题的一部分是，网站经常是许多标记语言和编程语言的混合物。您可以为 JavaScript 的某些部分编写单元测试，但如果它与其交互的 HTML 已更改以使 JavaScript 在页面上没有预期的操作，则此单元测试是无用的，即使它正常工作。

前端网站测试的问题经常被放在后面或者委托给只有最多一个清单和一个 bug 跟踪器的低级程序员。然而，只需稍微付出更多的努力，你就可以用一系列单元测试替换这个清单，并用网页抓取器代替人眼。

想象一下：为 Web 开发进行测试驱动开发。每天测试以确保 Web 界面的所有部分都正常运行。一套测试在有人添加新的网站功能或更改元素位置时运行。本章介绍了测试的基础知识以及如何使用基于 Python 的 Web 抓取器测试各种网站，从简单到复杂。

# 测试简介

如果您以前从未为代码编写过测试，那么现在没有比现在更好的时间开始了。拥有一套可以运行以确保代码按预期执行（至少是您为其编写了测试的范围）的测试集合会节省您时间和担忧，并使发布新更新变得容易。

## 什么是单元测试？

*测试* 和 *单元测试* 这两个词通常可以互换使用。通常，当程序员提到“编写测试”时，他们真正的意思是“编写单元测试”。另一方面，当一些程序员提到编写单元测试时，他们实际上在编写其他类型的测试。

尽管定义和实践往往因公司而异，但一个单元测试通常具有以下特征：

+   每个单元测试测试组件功能的一个方面。例如，它可能确保从银行账户中提取负数美元时会抛出适当的错误消息。

    单元测试通常根据它们所测试的组件分组在同一个类中。你可能会有一个测试，测试从银行账户中提取负美元值，然后是一个测试过度支出的银行账户行为的单元测试。

+   每个单元测试可以完全独立运行，单元测试所需的任何设置或拆卸必须由单元测试本身处理。同样，单元测试不得干扰其他测试的成功或失败，并且它们必须能够以任何顺序成功运行。

+   每个单元测试通常至少包含一个*断言*。例如，一个单元测试可能会断言 2 + 2 的答案是 4。偶尔，一个单元测试可能只包含一个失败状态。例如，如果抛出异常，则可能失败，但如果一切顺利，则默认通过。

+   单元测试与大部分代码分离。虽然它们必须导入和使用它们正在测试的代码，但它们通常保存在单独的类和目录中。

尽管可以编写许多其他类型的测试—例如集成测试和验证测试—但本章主要关注单元测试。不仅单元测试在最近推动的面向测试驱动开发中变得极为流行，而且它们的长度和灵活性使它们易于作为示例进行操作，并且 Python 具有一些内置的单元测试功能，你将在下一节中看到。

# Python 单元测试

Python 的单元测试模块`unittest`已包含在所有标准 Python 安装中。只需导入并扩展`unittest.TestCase`，它将：

+   提供`setUp`和`tearDown`函数，分别在每个单元测试之前和之后运行

+   提供几种类型的“assert”语句，以允许测试通过或失败

+   运行所有以`test_`开头的函数作为单元测试，并忽略未以测试形式开头的函数

下面提供了一个简单的单元测试，用于确保 2 + 2 = 4，根据 Python 的定义：

```py
import unittest

class TestAddition(unittest.TestCase):
    def setUp(self):
        print('Setting up the test')

    def tearDown(self):
        print('Tearing down the test')

    def test_twoPlusTwo(self):
        total = 2+2
        self.assertEqual(4, total);

if __name__ == '__main__':
    unittest.main()
```

尽管在这里`setUp`和`tearDown`不提供任何有用的功能，但它们被包含在内以进行说明。请注意，这些函数在每个单独的测试之前和之后运行，而不是在类的所有测试之前和之后运行。

当从命令行运行测试函数的输出应如下所示：

```py
Setting up the test
Tearing down the test
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK

```

这表明测试已成功运行，2 + 2 确实等于 4。

## 测试维基百科

测试网站的前端（不包括我们将在下一节中介绍的 JavaScript）只需将 Python`unittest`库与 Web 爬虫结合使用即可：

```py
from urllib.request import urlopen
from bs4 import BeautifulSoup
import unittest

class TestWikipedia(unittest.TestCase):
    bs = None
    def setUpClass():
        url = 'http://en.wikipedia.org/wiki/Monty_Python'
        TestWikipedia.bs = BeautifulSoup(urlopen(url), 'html.parser')

    def test_titleText(self):
        pageTitle = TestWikipedia.bs.find('h1').get_text()
        self.assertEqual('Monty Python', pageTitle);

    def test_contentExists(self):
        content = TestWikipedia.bs.find('div',{'id':'mw-content-text'})
        self.assertIsNotNone(content)

if __name__ == '__main__':
    unittest.main()
```

这次有两个测试：第一个测试页面的标题是否是预期的“Monty Python”，第二个确保页面具有内容`div`。

请注意，页面内容仅加载一次，并且全局对象`bs`在测试之间共享。这是通过使用`unittest`指定的`setUpClass`函数实现的，该函数在类开始时只运行一次（不像`setUp`，它在每个单独的测试之前运行）。使用`setUpClass`而不是`setUp`可以节省不必要的页面加载；您可以一次获取内容并对其运行多个测试。

`setUpClass`和`setUp`之间的一个主要架构差异，除了它们何时以及多频繁地运行之外，是`setUpClass`是一个静态方法，它“属于”类本身并具有全局类变量，而`setUp`是一个属于类的特定实例的实例函数。这就是为什么`setUp`可以在`self`上设置属性——该类的特定实例——而`setUpClass`只能访问类`TestWikipedia`上的静态类属性。

虽然一次只测试一个页面可能看起来并不那么强大或有趣，正如您可能从第六章中记得的那样，构建可以迭代地移动通过网站所有页面的网络爬虫相对容易。当您将一个网络爬虫与对每个页面进行断言的单元测试结合在一起时会发生什么？

有许多方法可以重复运行测试，但是你必须小心地每次加载每个页面，以及你还必须避免一次在内存中持有大量信息。以下设置正好做到了这一点：

```py
from urllib.request import urlopen
from bs4 import BeautifulSoup
import unittest
import re
import random
from urllib.parse import unquote

class TestWikipedia(unittest.TestCase):

    def test_PageProperties(self):
        self.url = 'http://en.wikipedia.org/wiki/Monty_Python'
        #Test the first 10 pages we encounter
        for i in range(1, 10):
            self.bs = BeautifulSoup(urlopen(self.url), 'html.parser')
            titles = self.titleMatchesURL()
            self.assertEquals(titles[0], titles[1])
            self.assertTrue(self.contentExists())
            self.url = self.getNextLink()
        print('Done!')

    def titleMatchesURL(self):
        pageTitle = self.bs.find('h1').get_text()
        urlTitle = self.url[(self.url.index('/wiki/')+6):]
        urlTitle = urlTitle.replace('_', ' ')
        urlTitle = unquote(urlTitle)
        return [pageTitle.lower(), urlTitle.lower()]

    def contentExists(self):
        content = self.bs.find('div',{'id':'mw-content-text'})
        if content is not None:
            return True
        return False

    def getNextLink(self):
        #Returns random link on page, using technique from Chapter 3
        links = self.bs.find('div', {'id':'bodyContent'}).find_all(
            'a', href=re.compile('^(/wiki/)((?!:).)*$'))
        randomLink = random.SystemRandom().choice(links)
        return 'https://wikipedia.org{}'.format(randomLink.attrs['href'])

if __name__ == '__main__':
    unittest.main()

```

有几件事情要注意。首先，这个类中只有一个实际的测试。其他函数技术上只是辅助函数，尽管它们完成了大部分计算工作来确定测试是否通过。因为测试函数执行断言语句，测试结果被传回测试函数，在那里断言发生。

另外，虽然`contentExists`返回一个布尔值，但`titleMatchesURL`返回用于评估的值本身。要了解为什么你希望传递值而不仅仅是布尔值，请比较布尔断言的结果：

```py
======================================================================
FAIL: test_PageProperties (__main__.TestWikipedia)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "15-3.py", line 22, in test_PageProperties
    self.assertTrue(self.titleMatchesURL())
AssertionError: False is not true

```

与`assertEquals`语句的结果一样：

```py
======================================================================
FAIL: test_PageProperties (__main__.TestWikipedia)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "15-3.py", line 23, in test_PageProperties
    self.assertEquals(titles[0], titles[1])
AssertionError: 'lockheed u-2' != 'u-2 spy plane'
```

哪一个更容易调试？（在这种情况下，错误是由于重定向导致的，当文章 *http://wikipedia.org/wiki/u-2%20spy%20plane* 重定向到一个名为“Lockheed U-2”的文章时。）

# 使用 Selenium 进行测试

就像在第十四章中的 Ajax 抓取一样，当进行网站测试时，JavaScript 在处理特定的网站时会出现特殊的挑战。幸运的是，Selenium 已经有了一个处理特别复杂网站的优秀框架；事实上，这个库最初就是为网站测试而设计的！

尽管显然使用相同的语言编写，Python 单元测试和 Selenium 单元测试的语法却惊人地不相似。Selenium 不要求其单元测试被包含在类中的函数中；它的`assert`语句不需要括号；测试在通过时静默通过，仅在失败时产生某种消息：

```py
driver = webdriver.Chrome()
driver.get('http://en.wikipedia.org/wiki/Monty_Python')
assert 'Monty Python' in driver.title
driver.close()
```

当运行时，这个测试应该不会产生任何输出。

以这种方式，Selenium 测试可以比 Python 单元测试更加随意地编写，并且`assert`语句甚至可以集成到常规代码中，当代码执行希望在未满足某些条件时终止时。

## 与站点互动

最近，我想通过一个本地小企业的网站联系他们的联系表单，但发现 HTML 表单已损坏；当我点击提交按钮时什么也没有发生。经过一番调查，我发现他们使用了一个简单的 mailto 表单，旨在用表单内容发送电子邮件给他们。幸运的是，我能够利用这些信息发送电子邮件给他们，解释表单的问题，并雇佣他们，尽管有技术问题。

如果我要编写一个传统的爬虫来使用或测试这个表单，我的爬虫很可能只会复制表单的布局并直接发送电子邮件，完全绕过表单。我该如何测试表单的功能性，并确保它通过浏览器正常工作？

虽然前几章已经讨论了导航链接、提交表单和其他类型的交互活动，但我们所做的一切本质上是为了 *绕过* 浏览器界面，而不是使用它。另一方面，Selenium 可以通过浏览器（在这种情况下是无头 Chrome 浏览器）直接输入文本、点击按钮以及执行所有操作，并检测到诸如损坏的表单、糟糕编码的 JavaScript、HTML 拼写错误以及其他可能困扰实际客户的问题。

这种测试的关键在于 Selenium 元素的概念。这个对象在 第十四章 简要提到，并且可以通过如下调用返回：

```py
usernameField = driver.find_element_by_name('username')
```

正如您可以在浏览器中对网站的各个元素执行多种操作一样，Selenium 可以对任何给定元素执行许多操作。其中包括：

```py
myElement.click()
myElement.click_and_hold()
myElement.release()
myElement.double_click()
myElement.send_keys_to_element('content to enter')
```

除了对元素执行一次性操作外，动作串也可以组合成 *动作链*，可以在程序中存储并执行一次或多次。动作链之所以有用，是因为它们可以方便地串接多个动作，但在功能上与显式调用元素上的动作完全相同，就像前面的例子一样。

要了解这种差异，请查看 [*http://pythonscraping.com/pages/files/form.html*](http://pythonscraping.com/pages/files/form.html) 上的表单页面（这在 第十三章 中曾作为示例使用）。我们可以以这种方式填写表单并提交：

```py
from selenium import webdriver
from selenium.webdriver.remote.webelement import WebElement
from selenium.webdriver.common.keys import Keys
from selenium.webdriver import ActionChains
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')

driver = webdriver.Chrome(
    executable_path='drivers/chromedriver', options=chrome_options)
driver.get('http://pythonscraping.com/pages/files/form.html')

firstnameField = driver.find_element_by_name('firstname')
lastnameField = driver.find_element_by_name('lastname')
submitButton = driver.find_element_by_id('submit')

### METHOD 1 ###
#firstnameField.send_keys('Ryan')
lastnameField.send_keys('Mitchell')
submitButton.click()
################

### METHOD 2 ###
actions = ActionChains(driver).click(firstnameField)
    .send_keys('Ryan')
    .click(lastnameField)
    .send_keys('Mitchell')
    .send_keys(Keys.RETURN)
actions.perform()
################

print(driver.find_element_by_tag_name('body').text)

driver.close()

```

方法 1 在两个字段上调用 `send_keys`，然后点击提交按钮。方法 2 使用单个动作链在调用 `perform` 方法后依次点击和输入每个字段的文本。无论使用第一种方法还是第二种方法，此脚本的操作方式都相同，并打印此行：

```py
Hello there, Ryan Mitchell!
```

两种方法之间还有另一种变化，除了它们用于处理命令的对象之外：请注意，第一种方法点击“提交”按钮，而第二种方法在提交表单时使用回车键。因为完成相同动作的事件序列有很多思考方式，使用 Selenium 可以完成相同的动作的方法也有很多。

### 拖放

点击按钮和输入文本是一回事，但是 Selenium 真正发光的地方在于它处理相对新颖的 Web 交互形式的能力。Selenium 允许轻松操作拖放接口。使用其拖放功能需要指定一个*源*元素（要拖动的元素）和要拖动到的目标元素或偏移量。

该演示页面位于[*http://pythonscraping.com/pages/javascript/draggableDemo.html*](http://pythonscraping.com/pages/javascript/draggableDemo.html)，展示了这种类型界面的一个示例：

```py
from selenium import webdriver
from selenium.webdriver.remote.webelement import WebElement
from selenium.webdriver import ActionChains
from selenium.webdriver.chrome.options import Options
import unittest

class TestDragAndDrop(unittest.TestCase):
    driver = None

    def setUp(self):
        chrome_options = Options()
        chrome_options.add_argument('--headless')
        self.driver = webdriver.Chrome(
            executable_path='drivers/chromedriver', options=chrome_options)
        url = 'http://pythonscraping.com/pages/javascript/draggableDemo.html'
        self.driver.get(url)

    def tearDown(self):
        driver.close()

    def test_drag(self):
        element = self.driver.find_element_by_id('draggable')
        target = self.driver.find_element_by_id('div2')
        actions = ActionChains(self.driver)
        actions.drag_and_drop(element, target).perform()
        self.assertEqual('You are definitely not a bot!',
                         self.driver.find_element_by_id('message').text)

```

从演示页面的`message div`中打印出两条消息。第一条消息是：

```py
Prove you are not a bot, by dragging the square from the blue area to the red 
area!
```

然后，在任务完成后，内容再次打印出来，现在读取：

```py
You are definitely not a bot!
```

正如演示页面所示，将元素拖动以证明你不是机器人是许多验证码的共同主题。尽管机器人早就能够拖动物体（只需点击、按住和移动），但“拖动此物”作为验证人类的想法似乎无法消亡。

此外，这些可拖动的验证码库很少使用任何对机器人困难的任务，例如“将小猫的图片拖到牛的图片上”（这需要你识别图片为“小猫”和“牛”，并解析指令）；相反，它们通常涉及数字排序或类似前面示例中的其他相当琐碎的任务。

当然，它们的强大之处在于其变化如此之多，而且使用频率如此之低；可能没有人会费力去制作一个能够击败所有验证码的机器人。无论如何，这个例子足以说明为什么你不应该在大型网站上使用这种技术。

### 拍摄截图

除了通常的测试功能外，Selenium 还有一个有趣的技巧，可能会使你的测试（或者让你的老板印象深刻）更加轻松：截图。是的，可以从运行的单元测试中创建照片证据，而无需实际按下 PrtScn 键：

```py
driver = webdriver.Chrome()
driver.get('http://www.pythonscraping.com/')
driver.get_screenshot_as_file('tmp/pythonscraping.png')
```

此脚本导航到[*http://pythonscraping.com*](http://pythonscraping.com)，然后将首页的截图存储在本地的*tmp*文件夹中（此文件夹必须已经存在才能正确存储）。截图可以保存为多种图像格式。
