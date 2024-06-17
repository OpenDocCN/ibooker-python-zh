# 第十四章。JavaScript 抓取

客户端脚本语言是在浏览器内部而不是在 Web 服务器上运行的语言。客户端语言的成功取决于浏览器正确解释和执行语言的能力。

虽然有数百种服务器端编程语言，但只有一种客户端编程语言。这是因为让每个浏览器制造商达成标准协议的难度很大。在进行网页抓取时，语言种类越少越好。

# 其他客户端编程语言

有些读者可能会对这句话提出异议：“只有一种客户端编程语言。”技术上存在 ActionScript 和 VBScript 等语言。然而，这些语言已不再受支持，并且在 VBScript 的情况下，仅被单个浏览器支持过。因此，它们很少被看到。

如果你想对此挑剔，任何人都可以创建新的客户端编程语言！可能有很多这样的语言存在！唯一的问题是得到浏览器的广泛支持，使该语言有效并被其他人使用。

有些人还争论说 CSS 和 HTML 本身就是编程语言。在理论上我同意这一点。Lara Schenck 在这个主题上有一篇出色而有趣的博客文章：[*https://notlaura.com/is-css-turing-complete/*](https://notlaura.com/is-css-turing-complete/)。

然而，在实际操作中，CSS 和 HTML 通常被视为与“编程语言”分开的标记语言，并且本书对它们有广泛的覆盖。

JavaScript 是迄今为止在 Web 上最常见和最受支持的客户端脚本语言。它可以用于收集用户跟踪信息，无需重新加载页面即可提交表单，嵌入多媒体，甚至支持整个在线游戏。即使看似简单的页面通常也包含多个 JavaScript 片段。你可以在页面的源代码中的`script`标签之间找到它：

```py
<script>
    alert("This creates a pop-up using JavaScript");
</script>
```

# JavaScript 简介

对于您正在抓取的代码至少有一些了解是非常有帮助的。考虑到这一点，熟悉 JavaScript 是个好主意。

*JavaScript* 是一种弱类型语言，其语法经常与 C++和 Java 进行比较。尽管语法的某些元素，如操作符、循环和数组，可能类似，但语言的弱类型和脚本化特性可能使它对某些程序员来说成为难以应付的“野兽”。

例如，以下递归计算斐波那契数列的值，直到 100，并将它们打印到浏览器的开发者控制台：

```py
<script> 
function fibonacci(a, b){ 
    var nextNum = a + b; 
    console.log(nextNum+" is in the Fibonacci sequence"); 
    if(nextNum < 100){ 
        fibonacci(b, nextNum); 
    } 
} 
fibonacci(1, 1);
</script>
```

注意，所有变量都是通过在其前面加上 `var` 来标识的。这类似于 PHP 中的 `$` 符号或 Java 或 C++ 中的类型声明（`int`、`String`、`List` 等）。Python 不同之处在于它没有这种明确的变量声明。

JavaScript 也擅长传递函数：

```py
<script>
var fibonacci = function() { 
    var a = 1; var b = 1;
    return function () { 
        [b, a] = [a + b, b];
        return b; 
    } 
}
var fibInstance = fibonacci();
console.log(fibInstance()+" is in the Fibonacci sequence"); 
console.log(fibInstance()+" is in the Fibonacci sequence"); 
console.log(fibInstance()+" is in the Fibonacci sequence");
</script>
```

起初可能会感到艰难，但如果你从 lambda 表达式的角度来思考（在 第五章 中介绍），问题就会变得简单起来。变量 `fibonacci` 被定义为一个函数。它的函数值返回一个函数，该函数打印出 Fibonacci 序列中逐渐增大的值。每次调用它时，它返回计算 Fibonacci 的函数，并再次执行并增加函数中的值。

你还可能会看到像这样用 JavaScript ES6 引入的箭头语法编写的函数：

```py
<script>
const fibonacci = () => { 
    var a = 1; var b = 1;
    return () => { 
        [b, a] = [a + b, b];
        return b; 
    } 
}
const fibInstance = fibonacci();
console.log(fibInstance()+" is in the Fibonacci sequence"); 
console.log(fibInstance()+" is in the Fibonacci sequence"); 
console.log(fibInstance()+" is in the Fibonacci sequence");
</script>
```

在这里，我使用 JavaScript 关键字 `const` 表示一个常量变量，它以后不会被重新分配。你可能还会看到关键字 `let`，表示可以重新分配的变量。这些关键字也是在 ES6 中引入的。

将函数作为变量传递也在处理用户操作和回调时非常有用，当涉及阅读 JavaScript 时，习惯这种编程风格是值得的。

## 常见的 JavaScript 库

尽管核心 JavaScript 语言很重要，但在现代网络上，如果不使用该语言的众多第三方库之一，你无法走得太远。查看页面源代码时，你可能会看到一种或多种常用的库。

通过 Python 执行 JavaScript 可能会非常耗时和处理器密集，特别是在大规模执行时。熟悉 JavaScript 并能够直接解析它（而无需执行以获取信息）可能非常有用，并且能节省你大量麻烦。

### jQuery

*jQuery* 是一种极为常见的库，被超过 70% 的网站使用¹。使用 jQuery 的网站很容易识别，因为它的代码中某处会包含对 jQuery 的导入：

```py
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></
 script>
```

如果在网站上找到了 jQuery，你在进行抓取时必须小心。jQuery 擅长动态创建 HTML 内容，这些内容仅在执行 JavaScript 后才出现。如果使用传统方法抓取页面内容，你只能获取到在 JavaScript 创建内容之前预加载的页面（这个抓取问题在 “Ajax 和动态 HTML” 中有更详细的讨论）。

此外，这些页面更有可能包含动画、交互内容和嵌入媒体，这可能会使抓取变得更具挑战性。

### Google Analytics

*Google Analytics* 被大约 50% 的所有网站使用²，使其成为可能是互联网上最常见的 JavaScript 库和最受欢迎的用户跟踪工具。[*http://pythonscraping.com*](http://pythonscraping.com) 和 [*http://www.oreilly.com/*](http://www.oreilly.com/) 均使用 Google Analytics。

确定页面是否使用 Google Analytics 很容易。它将在底部具有类似以下内容的 JavaScript（摘自 O’Reilly Media 网站）：

```py
<!-- Google Analytics -->
<script type="text/javascript">

var _gaq = _gaq || []; 
_gaq.push(['_setAccount', 'UA-4591498-1']);
_gaq.push(['_setDomainName', 'oreilly.com']);
_gaq.push(['_addIgnoredRef', 'oreilly.com']);
_gaq.push(['_setSiteSpeedSampleRate', 50]);
_gaq.push(['_trackPageview']);

(function() { var ga = document.createElement('script'); ga.type =
'text/javascript'; ga.async = true; ga.src = ('https:' ==
document.location.protocol ? 'https://ssl' : 'http://www') +
'.google-analytics.com/ga.js'; var s =
document.getElementsByTagName('script')[0];
s.parentNode.insertBefore(ga, s); })();

</script>
```

此脚本处理了用于跟踪您从页面到页面访问的 Google Analytics 特定 cookie。对于那些设计为执行 JavaScript 和处理 cookie（例如稍后在本章中讨论的 Selenium 使用的那些）的网络爬虫，这有时可能会成为一个问题。

如果网站使用 Google Analytics 或类似的 Web 分析系统，而您不希望该网站知道它正在被爬取或抓取，则确保丢弃任何用于分析的 cookie 或完全丢弃 cookie。

### Google 地图

如果您在互联网上花费了一些时间，几乎可以肯定地看到嵌入到网站中的 *Google 地图*。其 API 使得在任何网站上轻松嵌入具有自定义信息的地图成为可能。

如果您正在抓取任何类型的位置数据，了解 Google 地图的工作原理将使您能够轻松获取格式良好的纬度/经度坐标，甚至地址。在 Google 地图中表示位置的最常见方式之一是通过 *标记*（也称为 *图钉*）。

标记可以通过以下代码插入到任何 Google 地图中：

```py
var marker = new google.maps.Marker({
      position: new google.maps.LatLng(-25.363882,131.044922),
      map: map,
      title: 'Some marker text'
  });
```

Python 使得可以轻松提取在 `google.maps.LatLng(` 和 `)` 之间发生的所有坐标实例，以获取纬度/经度坐标列表。

使用 [Google 逆地理编码 API](https://developers.google.com/maps/documentation/javascript/examples/geocoding-reverse)，您可以将这些坐标对解析为适合存储和分析的地址。

# Ajax 和动态 HTML

到目前为止，我们与 web 服务器通信的唯一方法是通过检索新页面发送某种 HTTP 请求。如果您曾经提交过表单或在不重新加载页面的情况下从服务器检索信息，那么您可能使用过使用 Ajax 的网站。

与一些人的观点相反，Ajax 不是一种语言，而是一组用于完成特定任务的技术（与网页抓取类似）。*Ajax* 代表 *异步 JavaScript 和 XML*，用于向 web 服务器发送信息并接收信息，而无需发出单独的页面请求。

###### 注意

您不应该说，“这个网站将用 Ajax 编写。”而应该说，“这个网站将使用 Ajax 与 web 服务器通信。”

像 Ajax 一样，*动态 HTML*（DHTML）是用于共同目的的一组技术。DHTML 是 HTML 代码、CSS 语言或两者的组合，它随着客户端脚本在页面上改变 HTML 元素而改变。例如，当用户移动鼠标时，可能会出现一个按钮，点击时可能会改变背景颜色，或者 Ajax 请求可能会触发加载一个新的内容块。

请注意，尽管“动态”一词通常与“移动”或“变化”等词语相关联，交互式 HTML 组件的存在、移动图像或嵌入式媒体并不一定使页面成为 DHTML 页面，尽管它可能看起来很动态。此外，互联网上一些看起来最无聊、最静态的页面背后可能运行着依赖于 JavaScript 来操作 HTML 和 CSS 的 DHTML 进程。

如果你经常爬取多个网站，很快你就会遇到这样一种情况：你在浏览器中看到的内容与你从网站源代码中检索到的内容不匹配。当你查看爬虫的输出时，可能会摸不着头脑，试图弄清楚为什么你在浏览器上看到的内容在网页源代码中竟然找不到。

该网页还可能有一个加载页面，看起来似乎会将您重定向到另一个结果页面，但您会注意到，当此重定向发生时，页面的 URL 没有发生变化。

这两种情况都是由于你的爬虫未能执行页面上发生魔法的 JavaScript 导致的。没有 JavaScript，HTML 就只是呆呆地呈现在那里，而网站可能看起来与在你的网页浏览器中看到的完全不同。

页面可能有几个迹象表明它可能在使用 Ajax 或 DHTML 来更改或加载内容，但在这种情况下，只有两种解决方案：直接从 JavaScript 中获取内容；或使用能够执行 JavaScript 并在浏览器中查看网站的 Python 包来爬取网站。

# 在 Python 中执行 Selenium 中的 JavaScript

[Selenium](http://www.seleniumhq.org) 是一个强大的网页抓取工具，最初用于网站测试。如今，当需要准确地呈现网页在浏览器中的外观时，也会使用 Selenium。Selenium 通过自动化浏览器加载网页，获取所需数据，甚至拍摄屏幕截图或验证网站上发生的某些动作来工作。

Selenium 不包含自己的网络浏览器；它需要与第三方浏览器集成才能运行。例如，如果你用 Firefox 运行 Selenium，你会看到一个 Firefox 实例在你的屏幕上打开，导航到网站，并执行你在代码中指定的操作。尽管这可能很有趣，我更喜欢我的脚本在后台静静地运行，并经常使用 Chrome 的*无头*模式来做到这一点。

一个*无头浏览器*将网站加载到内存中，并在页面上执行 JavaScript，但不会向用户显示网站的图形渲染。通过将 Selenium 与无头 Chrome 结合使用，你可以运行一个非常强大的网页抓取器，轻松处理 cookies、JavaScript、标头以及其他所有你需要的东西，就像使用渲染浏览器一样。

## 安装和运行 Selenium

你可以从[其网站](https://pypi.python.org/pypi/selenium)下载 Selenium 库，或者使用像 pip 这样的第三方安装程序从命令行安装它。

```py
$ pip install selenium
```

以前的 Selenium 版本要求你手动下载一个 webdriver 文件，以便它与你的网络浏览器进行交互。这个 webdriver 之所以被称为这样，是因为它是网页浏览器的软件*驱动程序*。就像硬件设备的软件驱动程序一样，它允许 Python Selenium 包与你的浏览器进行接口和控制。

不幸的是，由于浏览器的新版本频繁发布，并且多亏了自动更新，这意味着 Selenium 驱动程序也必须经常更新。导航到浏览器驱动程序的网站（例如[*http://chromedriver.chromium.org/downloads*](http://chromedriver.chromium.org/downloads)），下载新文件，并替换旧文件是一个频繁的烦恼。在 2021 年十月发布的 Selenium 4 中，这整个过程被 webdriver 管理器 Python 包替代。

webdriver 管理器可以通过 pip 安装：

```py
$ pip install webdriver-manager

```

调用时，webdriver 管理器会下载最新的驱动程序：

```py
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
driver.get("http://www.python.org")
time.sleep(2)
driver.close()
```

当然，如果这个脚本经常运行，每次运行时都安装一个新的驱动文件以防止 Chrome 浏览器自上次运行以来被更新是低效的。驱动程序管理器安装的输出只是驱动程序位于你的`driver`目录中的路径：

```py
CHROMEDRIVER_PATH = ChromeDriverManager().install()
driver = webdriver.Chrome(service=Service(CHROMEDRIVER_PATH))
```

如果你仍然喜欢手动下载文件，你可以通过将自己的路径传递给`Service`对象来实现：

```py
from selenium import webdriver
from selenium.webdriver.chrome.service import Service

CHROMEDRIVER_PATH = 'drivers/chromedriver_mac_arm64/chromedriver'
driver = webdriver.Chrome(service=Service(CHROMEDRIVER_PATH))
driver.get("http://www.python.org")
time.sleep(2)
driver.close()

```

尽管许多页面使用 Ajax 加载数据，我已经创建了一个样例页面[*http://pythonscraping.com/pages/javascript/ajaxDemo.html*](http://pythonscraping.com/pages/javascript/ajaxDemo.html)供你对抗抓取器。这个页面包含一些样例文本，硬编码到页面的 HTML 中，在两秒延迟后被 Ajax 生成的内容替换。如果你使用传统方法来抓取这个页面的数据，你只会得到加载页面，而无法获取你想要的数据。

Selenium 库是调用[`webdriver`](https://selenium-python.readthedocs.io/api.html)对象的 API。请注意，这是一个代表或充当你下载的 webdriver 应用程序的 Python 对象。虽然“driver”和`webdriver`这两个术语通常可以互换使用来指代这两个东西（Python 对象和应用程序本身），但在概念上区分它们是很重要的。

`webdriver`对象有点像浏览器，它可以加载网站，但也可以像`BeautifulSoup`对象一样用于查找页面元素，与页面上的元素交互（发送文本，单击等），并执行其他操作以驱动网络爬虫。

以下代码检索测试页面上 Ajax“墙”后面的文本：

```py
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
import time

chrome_options = Options()
chrome_options.add_argument("--headless")
driver = webdriver.Chrome(
    service=Service(CHROMEDRIVER_PATH),
    options=chrome_options
)
driver.get('http://pythonscraping.com/pages/javascript/ajaxDemo.html')
time.sleep(3)
print(driver.find_element(By.ID, 'content').text)
driver.close()

```

这将使用 Chrome 库创建一个新的 Selenium webdriver，告诉 webdriver 加载页面，然后在查看页面以检索（希望加载的）内容之前暂停执行三秒钟。

当您在 Python 中实例化一个新的 Chrome webdriver 时，可以通过`Options`对象传递各种选项。在这种情况下，我们使用`--headless`选项使 webdriver 在后台运行：

```py
chrome_options = Options()
chrome_options.add_argument('--headless')

```

无论您是使用驱动程序管理程序包安装驱动程序还是自行下载驱动程序，都必须将此路径传递给`Service`对象，并传递您的选项，以创建新的 webdriver：

```py
driver = webdriver.Chrome(
    service=Service(CHROMEDRIVER_PATH),
    options=chrome_options
)

```

如果一切配置正确，脚本应该在几秒钟内运行，然后结果如下所示：

```py
Here is some important text you want  to retrieve!
A button to click!
```

## Selenium 选择器

在以前的章节中，您已使用`BeautifulSoup`选择器（如`find`和`find_all`）选择页面元素。 Selenium 使用非常相似的一组方法来选择元素：`find_element`和`find_elements`。

从 HTML 中找到和选择元素的方法有很多，您可能会认为 Selenium 会使用各种参数和关键字参数来执行这些方法。但是，对于`find_element`和`find_elements`，这两个函数都只有两个参数：`By`对象和字符串选择器。

`By`对象指定选择器字符串应如何解释，有以下选项列表：

`By.ID`

在示例中使用；通过它们的 HTML `id`属性查找元素。

`By.NAME`

通过它们的`name`属性查找 HTML 标记。这对于 HTML 表单很方便。

`By.XPATH`

使用 XPath 表达式选择匹配的元素。XPath 语法将在本章的后面更详细地介绍。

`By.LINK_TEXT`

通过它们包含的文本查找 HTML `<a>`标签。例如，可以使用`(By.LINK_TEXT，'Next')`选择标记为“Next”的链接。

`By.PARTIAL_LINK_TEXT`

类似于`LINK_TEXT`，但匹配部分字符串。

`By.TAG_NAME`

通过标记名称查找 HTML 标记。

`By.CLASS_NAME`

用于通过它们的 HTML `class`属性查找元素。为什么这个函数是`CLASS_NAME`而不仅仅是`CLASS`？使用形式`object.CLASS`会为 Selenium 的 Java 库创建问题，其中`.class`是保留方法。为了保持各种语言之间的 Selenium 语法一致，使用了`CLASS_NAME`。

`By.CSS_SELECTOR`

使用`class`、`id`或`tag`名称查找元素，使用`#idName`、`.className`、`tagName`约定。

在前面的示例中，您使用了选择器`driver.find_element(By.ID，'content')`，虽然以下选择器也可以使用：

```py
driver.find_element(By.CSS_SELECTOR, '#content')
driver.find_element(By.TAG_NAME, 'div')
```

当然，如果要在页面上选择多个元素，大多数这些元素选择器都可以通过使用`elements`（即，使其复数化）返回一个 Python 元素列表：

```py
driver.find_elements(By.CSS_SELECTOR, '#content')
driver.find_elements(By.TAG_NAME, 'div')
```

如果仍然希望使用 BeautifulSoup 解析此内容，则可以通过使用 webdriver 的 `page_source` 函数来实现，该函数将页面的源代码作为字符串返回，正如在当前时间由 DOM 查看的那样：

```py
pageSource = driver.page_source
bs = BeautifulSoup(pageSource, 'html.parser')
print(bs.find(id='content').get_text())
```

## 等待加载

请注意，尽管页面本身包含一个 HTML 按钮，但 Selenium 的 `.text` 函数以与检索页面上所有其他内容相同的方式检索按钮的文本值。

如果将`time.sleep`暂停时间更改为一秒而不是三秒，则返回的文本将变为原始文本：

```py
This is some content that will appear on the page while it's loading.
 You don't care about scraping this.
```

尽管此解决方案有效，但效率略低，并且实施它可能在大规模上造成问题。页面加载时间不一致，取决于任何特定毫秒的服务器负载，并且连接速度会自然变化。尽管此页面加载应仅需超过两秒，但您将其整整三秒时间来确保其完全加载。更高效的解决方案将重复检查完全加载页面上特定元素的存在，并仅在该元素存在时返回。

以下程序使用带有 ID `loadedButton` 的按钮的出现来声明页面已完全加载：

```py
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

chrome_options = Options()
chrome_options.add_argument("--headless")
driver = webdriver.Chrome(
    service=Service(CHROMEDRIVER_PATH),
    options=chrome_options)

driver.get('http://pythonscraping.com/pages/javascript/ajaxDemo.html')
try:
    element = WebDriverWait(driver, 10).until(
                       EC.presence_of_element_located((By.ID, 'loadedButton')))
finally:
    print(driver.find_element(By.ID, 'content').text)
    driver.close()

```

此脚本引入了几个新的导入项，特别是`WebDriverWait`和`expected_conditions`，两者在此处组合以形成 Selenium 称之为的*隐式等待*。

隐式等待与显式等待不同，它在继续之前等待 DOM 中的某种状态发生，而显式等待则定义了一个硬编码时间，例如前面示例中的三秒等待时间。在隐式等待中，触发 DOM 状态由`expected_condition`定义（请注意，此处导入被转换为`EC`，这是一种常用的简洁约定）。在 Selenium 库中，预期条件可以是许多事物，包括：

+   弹出警报框。

+   元素（例如文本框）处于*选定*状态。

+   页面标题更改，或现在在页面上或特定元素中显示文本。

+   现在，一个元素对 DOM 可见，或者一个元素从 DOM 中消失。

大多数这些预期条件要求您首先指定要监视的元素。元素使用定位器指定。请注意，定位器与选择器不同（有关选择器的更多信息，请参见“Selenium Selectors”）。*定位器*是一种抽象查询语言，使用`By`对象，可以以多种方式使用，包括制作选择器。

在下面的代码中，使用定位器查找具有 ID `loadedButton` 的元素：

```py
EC.presence_of_element_located((By.ID, 'loadedButton'))
```

定位器还可以用来创建选择器，使用`find_element` webdriver 函数：

```py
print(driver.find_element(By.ID, 'content').text)
```

如果不需要使用定位器，请不要使用；这将节省您的导入。然而，这个方便的工具用于各种应用，并具有极大的灵活性。

## XPath

*XPath*（即*XML Path*）是用于导航和选择 XML 文档部分的查询语言。1999 年由 W3C 创立，偶尔在 Python、Java 和 C#等语言中处理 XML 文档时使用。

虽然 BeautifulSoup 不支持 XPath，但本书中的许多其他库（如 Scrapy 和 Selenium）支持。它通常可以像 CSS 选择器（例如`mytag#idname`）一样使用，尽管它设计用于与更广义的 XML 文档一起工作，而不是特定的 HTML 文档。

XPath 语法有四个主要概念：

根节点与非根节点

`/div` 仅在文档根部选择 div 节点。

`//div` 在文档中选择所有的 div。

属性选择

`//@href` 选择具有属性`href`的任何节点。

`//a[@href='http://google.com']` 选择文档中指向 Google 的所有链接。

通过位置选择节点

`//a[3]` 选择文档中的第三个链接。

`//table[last()]` 选择文档中的最后一个表格。

`//a[position() < 3]` 选择文档中的前两个链接。

星号（*）匹配任何字符或节点，并可在各种情况下使用。

`//table/tr/*` 选择所有表格中`tr`标签的子节点（这对使用`th`和`td`标签选择单元格很有用）。

`//div[@*]` 选择所有带有任何属性的`div`标签。

XPath 语法还具有许多高级功能。多年来，它发展成为一个相对复杂的查询语言，具有布尔逻辑、函数（如`position()`）和各种此处未讨论的运算符。

如果您有无法通过此处显示的功能解决的 HTML 或 XML 选择问题，请参阅[Microsoft 的 XPath 语法页面](https://msdn.microsoft.com/en-us/enus/library/ms256471)。

# 其他 Selenium WebDrivers

在前一节中，Chrome WebDriver（ChromeDriver）与 Selenium 一起使用。大多数情况下，不需要浏览器弹出屏幕并开始网页抓取，因此在无头模式下运行会很方便。然而，以非无头模式运行，并/或使用不同的浏览器驱动程序，可以出于多种原因进行良好的实践：

+   故障排除。如果您的代码在无头模式下运行失败，可能很难在没有看到页面的情况下诊断失败原因。

+   您还可以暂停代码执行并与网页交互，或在您的抓取器运行时使用检查工具来诊断问题。

+   测试可能依赖于特定的浏览器才能运行。一个浏览器中的失败而另一个浏览器中没有可能指向特定于浏览器的问题。

在大多数情况下，最好使用 webdriver 管理器获取您的浏览器驱动程序。例如，您可以使用 webdriver 管理器来获取 Firefox 和 Microsoft Edge 的驱动程序：

```py
from webdriver_manager.firefox import GeckoDriverManager
​from webdriver_manager.microsoft import EdgeChromiumDriverManager

print(GeckoDriverManager().install())
print(EdgeChromiumDriverManager().install())

```

然而，如果您需要一个已弃用的浏览器版本或者通过 webdriver 管理器无法获取的浏览器（例如 Safari），您可能仍然需要手动下载驱动程序文件。

今天的每个主要浏览器都有许多官方和非官方团体参与创建和维护 Selenium Web 驱动程序。Selenium 团队整理了一个用于参考的[这些 Web 驱动程序的集合](http://www.seleniumhq.org/download/)。

# 处理重定向

客户端重定向是由 JavaScript 在您的浏览器中执行的页面重定向，而不是在服务器上执行的重定向，在发送页面内容之前。当您访问网页时，有时很难区分两者的区别。重定向可能发生得如此之快，以至于您没有注意到加载时间的任何延迟，并假定客户端重定向实际上是服务器端重定向。

然而，在网页抓取时，差异是显而易见的。一个服务器端的重定向，根据它是如何处理的，可以通过 Python 的 urllib 库轻松地遍历，而无需 Selenium 的帮助（有关如何执行此操作的更多信息，请参见第六章）。客户端重定向除非执行 JavaScript，否则根本不会处理。

Selenium 能够以处理其他 JavaScript 执行的方式处理这些 JavaScript 重定向；然而，这些重定向的主要问题在于何时停止页面执行，即如何判断页面何时完成重定向。一个演示页面在[*http://pythonscraping.com/pages/javascript/redirectDemo1.html*](http://pythonscraping.com/pages/javascript/redirectDemo1.html)上展示了这种类型的重定向，带有两秒的暂停。

您可以通过“观察”页面初始加载时的 DOM 中的一个元素来巧妙地检测重定向，然后重复调用该元素，直到 Selenium 抛出`StaleElementReferenceException`；元素不再附加到页面的 DOM 中，网站已重定向：

```py
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.common.exceptions import StaleElementReferenceException
import time

def waitForLoad(driver):
    elem = driver.find_element(By.TAG_NAME, "html")
    count = 0
    for _ in range(0, 20):
        try:
            elem == driver.find_element(By.TAG_NAME, "html")
        except StaleElementReferenceException:
            return
        time.sleep(0.5)
    print("Timing out after 10 seconds and returning")

chrome_options = Options()
chrome_options.add_argument("--headless")
driver = webdriver.Chrome(
    service=Service(CHROMEDRIVER_PATH),
    options=chrome_options
)
driver.get("http://pythonscraping.com/pages/javascript/redirectDemo1.html")
waitForLoad(driver)
print(driver.page_source)
driver.close()

```

这个脚本每隔半秒检查一次页面，超时时间为 10 秒，尽管检查时间和超时时间可以根据需要轻松调整。

或者，您可以编写一个类似的循环来检查页面当前的 URL，直到 URL 发生变化或者匹配您正在寻找的特定 URL 为止。

在 Selenium 中等待元素出现和消失是一个常见任务，您也可以像上一个按钮加载示例中使用的`WebDriverWait`函数一样使用相同的方法。在这里，您提供了一个 15 秒的超时时间和一个 XPath 选择器，用于查找页面主体内容来完成相同的任务：

```py
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException

chrome_options = Options()
chrome_options.add_argument("--headless")
driver = webdriver.Chrome(
    executable_path='drivers/chromedriver', 
    options=chrome_options)
driver.get('http://pythonscraping.com/pages/javascript/redirectDemo1.html')
try:
    txt = 'This is the page you are looking for!'
    bodyElement = WebDriverWait(driver, 15).until(
        EC.presence_of_element_located((
            By.XPATH,
            f'//body[contains(text(), "{txt}")]'
        ))
    )
    print(bodyElement.text)
except TimeoutException:
    print('Did not find the element')

```

# JavaScript 的最后一点说明

当今大多数互联网上的网站都在使用 JavaScript。³ 幸运的是，对我们来说，在许多情况下，这种 JavaScript 的使用不会影响你对页面的抓取。JavaScript 可能仅限于为站点的跟踪工具提供动力，控制站点的一小部分或操作下拉菜单，例如。在它影响到你抓取网站的方式时，可以使用像 Selenium 这样的工具来执行 JavaScript，以生成你在本书第一部分学习抓取的简单 HTML 页面。

记住：仅因为一个网站使用 JavaScript 并不意味着所有传统的网络爬取工具都不再适用。JavaScript 的目的最终是生成可以由浏览器渲染的 HTML 和 CSS 代码，或通过 HTTP 请求和响应与服务器动态通信。一旦使用 Selenium，页面上的 HTML 和 CSS 可以像处理其他任何网站代码一样读取和解析，通过本书前几章的技术可以发送和处理 HTTP 请求和响应，即使不使用 Selenium 也可以。

此外，JavaScript 甚至可以成为网络爬虫的一个好处，因为它作为“浏览器端内容管理系统”的使用可能向外界公开有用的 API，使您可以更直接地获取数据。有关更多信息，请参见第十五章。

如果你在处理某个棘手的 JavaScript 情况时仍然遇到困难，你可以在第十七章中找到关于 Selenium 和直接与动态网站交互的信息，包括拖放界面等。

¹ 查看 Web Technology Surveys 分析，网址为[*https://w3techs.com/technologies/details/js-jquery*](https://w3techs.com/technologies/details/js-jquery)，W3Techs 使用网络爬虫随时间监测技术使用趋势。

² W3Techs，《Google Analytics 网站使用统计和市场份额》。

³ W3Techs，《网站上作为客户端编程语言使用的 JavaScript 的使用统计》。
