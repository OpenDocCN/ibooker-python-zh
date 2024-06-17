# 第七章：网络爬虫模型

当你控制数据和输入时，编写干净、可扩展的代码已经足够困难了。编写网页爬虫的代码，可能需要从程序员无法控制的各种网站中抓取和存储各种数据，通常会带来独特的组织挑战。

你可能会被要求从各种网站上收集新闻文章或博客文章，每个网站都有不同的模板和布局。一个网站的 `h1` 标签包含文章的标题，另一个网站的 `h1` 标签包含网站本身的标题，而文章标题在 `<span id="title">` 中。

你可能需要灵活地控制哪些网站被抓取以及它们如何被抓取，并且需要一种快速添加新网站或修改现有网站的方法，而不需要编写多行代码。

你可能被要求从不同网站上抓取产品价格，最终目标是比较相同产品的价格。也许这些价格是以不同的货币计价的，也许你还需要将其与某些其他非网络来源的外部数据结合起来。

虽然网络爬虫的应用几乎是无穷无尽的，但是大型可扩展的爬虫往往会落入几种模式之一。通过学习这些模式，并识别它们适用的情况，你可以极大地提高网络爬虫的可维护性和健壮性。

本章主要关注收集有限数量“类型”数据的网络爬虫（例如餐馆评论、新闻文章、公司简介）从各种网站收集这些数据类型，并将其存储为 Python 对象，从数据库中读写。

# 规划和定义对象

网页抓取的一个常见陷阱是完全基于眼前可见的内容定义你想要收集的数据。例如，如果你想收集产品数据，你可能首先看一下服装店，决定你要抓取的每个产品都需要有以下字段：

+   产品名称

+   价格

+   描述

+   尺码

+   颜色

+   织物类型

+   客户评分

当你查看另一个网站时，你发现它在页面上列出了 SKU（用于跟踪和订购商品的库存单位）你肯定也想收集这些数据，即使在第一个网站上看不到它！你添加了这个字段：

+   商品 SKU

虽然服装可能是一个很好的起点，但你也希望确保你可以将这个爬虫扩展到其他类型的产品上。你开始浏览其他网站的产品部分，并决定你还需要收集以下信息：

+   精装/平装

+   亚光/亮光打印

+   客户评论数量

+   制造商链接

显然，这种方法是不可持续的。每次在网站上看到新的信息时，简单地将属性添加到产品类型中将导致要跟踪的字段过多。不仅如此，每次抓取新的网站时，你都将被迫对网站具有的字段和到目前为止积累的字段进行详细分析，并可能添加新字段（修改你的 Python 对象类型和数据库结构）。这将导致一个混乱且难以阅读的数据集，可能会导致在使用它时出现问题。

在决定收集哪些数据时，你经常最好的做法是忽略网站。你不会通过查看单个网站并说“存在什么？”来开始一个旨在大规模和可扩展的项目，而是通过说“我需要什么？”然后找到从那里获取所需信息的方法。

也许你真正想做的是比较多家商店的产品价格，并随着时间跟踪这些产品价格。在这种情况下，你需要足够的信息来唯一标识产品，就是这样：

+   产品标题

+   制造商

+   产品 ID 编号（如果可用/相关）

重要的是要注意，所有这些信息都不特定于特定商店。例如，产品评论、评级、价格，甚至描述都特定于特定商店中的该产品的实例。这可以单独存储。

其他信息（产品的颜色、材质）是产品特定的，但可能稀疏——并非适用于每种产品。重要的是要退后一步，对你考虑的每个项目执行一个清单，并问自己这些问题：

+   这些信息是否有助于项目目标？如果我没有它，是否会成为一个障碍，还是它只是“好有”但最终不会对任何事情产生影响？

+   如果*可能*将来有用，但我不确定，那么在以后收集这些数据会有多困难？

+   这些数据是否与我已经收集的数据重复了？

+   在这个特定对象中存储数据是否有逻辑意义？（如前所述，如果一个产品的描述在不同网站上发生变化，则在产品中存储描述是没有意义的。）

如果你决定需要收集这些数据，重要的是要提出更多问题，然后决定如何在代码中存储和处理它：

+   这些数据是稀疏的还是密集的？它在每个列表中是否都是相关且填充的，还是只有一小部分是相关的？

+   数据有多大？

+   尤其是在大数据的情况下，我需要每次运行分析时定期检索它，还是只在某些情况下检索它？

+   这种类型的数据变化多大？我是否经常需要添加新属性，修改类型（如可能经常添加的面料图案），还是它是固定的（鞋码）？

假设您计划围绕产品属性和价格进行一些元分析：例如，一本书的页数，或一件衣服的面料类型，以及未来可能的其他属性，与价格相关。您仔细思考这些问题，并意识到这些数据是稀疏的（相对较少的产品具有任何一个属性），并且您可能经常决定添加或删除属性。在这种情况下，创建一个如下所示的产品类型可能是有意义的：

+   产品标题

+   制造商

+   产品 ID 编号（如适用/相关）

+   属性（可选列表或字典）

以及以下类似的属性类型：

+   属性名称

+   属性值

这使您能够随时间灵活添加新的产品属性，而无需重新设计数据模式或重写代码。在决定如何在数据库中存储这些属性时，您可以将 JSON 写入`attribute`字段，或者将每个属性存储在一个带有产品 ID 的单独表中。有关实现这些类型数据库模型的更多信息，请参见第九章。

您可以将前述问题应用于您需要存储的其他信息。例如，要跟踪每个产品找到的价格，您可能需要以下信息：

+   产品 ID

+   商店 ID

+   价格

+   记录价格发现的日期/时间戳

但是，如果产品的属性实际上会改变产品的价格怎么办？例如，商店可能会对大号衬衫收取更高的价格，因为制作大号衬衫需要更多的人工或材料。在这种情况下，您可以考虑将单个衬衫产品拆分为每种尺寸的独立产品列表（以便每件衬衫产品可以独立定价），或者创建一个新的项目类型来存储产品实例信息，包含以下字段：

+   产品 ID

+   实例类型（本例中为衬衫尺寸）

每个价格看起来会像这样：

+   产品实例 ID

+   商店 ID

+   价格

+   记录价格发现的日期/时间戳

虽然“产品和价格”主题可能显得过于具体，但是在设计 Python 对象时，您需要问自己的基本问题和逻辑几乎适用于每种情况。

如果您正在抓取新闻文章，您可能希望获取基本信息，例如：

+   标题

+   作者

+   日期

+   内容

但是，如果一些文章包含“修订日期”，或“相关文章”，或“社交媒体分享数量”呢？您是否需要这些信息？它们是否与您的项目相关？当不是所有新闻网站都使用所有形式的社交媒体，并且社交媒体站点可能随时间而增长或减少时，您如何高效灵活地存储社交媒体分享数量？

当面对一个新项目时，很容易立即投入到编写 Python 代码以抓取网站的工作中。然而，数据模型往往被第一个抓取的网站的数据的可用性和格式所强烈影响，而被忽视。

然而，数据模型是所有使用它的代码的基础。在模型中做出不好的决定很容易导致以后编写和维护代码时出现问题，或者在提取和高效使用结果数据时出现困难。特别是在处理各种网站（已知和未知的）时，认真考虑和规划你需要收集什么，以及你如何存储它变得至关重要。

# 处理不同的网页布局

象 Google 这样的搜索引擎最令人印象深刻的一个特性是，它能够从各种网站中提取相关且有用的数据，而不需要预先了解网站的结构。尽管我们作为人类能够立即识别页面的标题和主要内容（除了极端糟糕的网页设计情况），但让机器人做同样的事情要困难得多。

幸运的是，在大多数网络爬虫的情况下，你不需要从你以前从未见过的网站收集数据，而是从少数或几十个由人类预先选择的网站收集数据。这意味着你不需要使用复杂的算法或机器学习来检测页面上“看起来最像标题”的文本或者哪些可能是“主要内容”。你可以手动确定这些元素是什么。

最明显的方法是为每个网站编写单独的网络爬虫或页面解析器。每个解析器可能接受一个 URL、字符串或`BeautifulSoup`对象，并返回一个被爬取的 Python 对象。

下面是一个`Content`类的示例（代表网站上的一篇内容，比如新闻文章），以及两个爬虫函数，它们接受一个`BeautifulSoup`对象并返回一个`Content`的实例：

```py
from bs4 import BeautifulSoup
from urllib.request import urlopen

class Content:
    def __init__(self, url, title, body):
        self.url = url
        self.title = title
        self.body = body

    def print(self):
        print(f'TITLE: {self.title}')
        print(f'URL: {self.url}')
        print(f'BODY: {self.body}')

def scrapeCNN(url):
    bs = BeautifulSoup(urlopen(url))
    title = bs.find('h1').text
    body = bs.find('div', {'class': 'article__content'}).text
    print('body: ')
    print(body)
    return Content(url, title, body)

def scrapeBrookings(url):
    bs = BeautifulSoup(urlopen(url))
    title = bs.find('h1').text
    body = bs.find('div', {'class': 'post-body'}).text
    return Content(url, title, body)

url = 'https://www.brookings.edu/research/robotic-rulemaking/'
content = scrapeBrookings(url)
content.print()

url = 'https://www.cnn.com/2023/04/03/investing/\
dogecoin-elon-musk-twitter/index.html'
content = scrapeCNN(url)
content.print()

```

当你开始为额外的新闻网站添加爬虫函数时，你可能会注意到一种模式正在形成。每个网站的解析函数基本上都在做同样的事情：

+   选择标题元素并提取标题文本

+   选择文章的主要内容

+   根据需要选择其他内容项

+   返回一个通过先前找到的字符串实例化的`Content`对象

这里唯一真正依赖于网站的变量是用于获取每个信息片段的 CSS 选择器。BeautifulSoup 的`find`和`find_all`函数接受两个参数——一个标签字符串和一个键/值属性字典——所以你可以将这些参数作为定义站点结构和目标数据位置的参数传递进去。

更方便的是，你可以使用 BeautifulSoup 的`select`函数，用一个单一的字符串 CSS 选择器来获取每个想要收集的信息片段，并将所有这些选择器放在一个字典对象中：

```py
class Content:
    """
    Common base class for all articles/pages
    """
    def __init__(self, url, title, body):
        self.url = url
        self.title = title
        self.body = body

    def print(self):
        """
        Flexible printing function controls output
        """
        print('URL: {}'.format(self.url))
        print('TITLE: {}'.format(self.title))
        print('BODY:\n{}'.format(self.body))

class Website:
    """ 
    Contains information about website structure
    """
    def __init__(self, name, url, titleTag, bodyTag):
        self.name = name
        self.url = url
        self.titleTag = titleTag
        self.bodyTag = bodyTag

```

注意`Website`类不存储从单个页面收集的信息，而是存储关于*如何*收集这些数据的说明。它不存储标题“我的页面标题”。它只是存储表示标题位置的字符串标签`h1`。这就是为什么这个类被称为`Website`（这里的信息涉及整个网站）而不是`Content`（它只包含来自单个页面的信息）的原因。

当您编写网络爬虫时，您可能会注意到您经常一遍又一遍地执行许多相同的任务。例如：获取页面内容同时检查错误，获取标签内容，如果找不到则优雅地失败。让我们将这些添加到一个`Crawler`类中：

```py
class Crawler:
    def getPage(url):
        try:
            html = urlopen(url)
        except Exception:
            return None
        return BeautifulSoup(html, 'html.parser')

    def safeGet(bs, selector):
        """
        Utility function used to get a content string from a Beautiful Soup
        object and a selector. Returns an empty string if no object
        is found for the given selector
        """
        selectedElems = bs.select(selector)
        if selectedElems is not None and len(selectedElems) > 0:
            return '\n'.join([elem.get_text() for elem in selectedElems])
        return ''

```

请注意，`Crawler`类目前没有任何状态。它只是一组静态方法。它的命名似乎也很差劲——它根本不进行任何爬取！你至少可以通过向其添加一个`getContent`方法稍微增加其实用性，该方法接受一个`website`对象和一个 URL 作为参数，并返回一个`Content`对象：

```py
class Crawler:

    ...

    def getContent(website, path):
        """
        Extract content from a given page URL
        """
        url = website.url+path
        bs = Crawler.getPage(url)
        if bs is not None:
            title = Crawler.safeGet(bs, website.titleTag)
            body = Crawler.safeGet(bs, website.bodyTag)
            return Content(url, title, body)
        return Content(url, '', '')

```

以下显示了如何将这些`Content`、`website`和`Crawler`类一起使用来爬取四个不同的网站：

```py
siteData = [
    ['O\'Reilly', 'https://www.oreilly.com', 'h1', 'div.title-description'],
    ['Reuters', 'https://www.reuters.com', 'h1', 'div.ArticleBodyWrapper'],
    ['Brookings', 'https://www.brookings.edu', 'h1', 'div.post-body'],
    ['CNN', 'https://www.cnn.com', 'h1', 'div.article__content']
]
websites = []
for name, url, title, body in siteData:
    websites.append(Website(name, url, title, body))

Crawler.getContent(
    websites[0], 
    '/library/view/web-scraping-with/9781491910283'
    ).print()
Crawler.getContent(
    websites[1],
    '/article/us-usa-epa-pruitt-idUSKBN19W2D0'
    ).print()
Crawler.getContent(
    websites[2],
    '/blog/techtank/2016/03/01/idea-to-retire-old-methods-of-policy-education/'
    ).print()
Crawler.getContent(
    websites[3], 
    '/2023/04/03/investing/dogecoin-elon-musk-twitter/index.html'
    ).print()

```

这种新方法乍看起来可能并不比为每个新网站编写新的 Python 函数更简单，但想象一下当你从一个拥有 4 个网站来源的系统转变为一个拥有 20 个或 200 个来源的系统时会发生什么。

定义一个新网站的每个字符串列表都相对容易编写。它不占用太多空间。它可以从数据库或 CSV 文件中加载。它可以从远程源导入或交给具有一点前端经验的非程序员。这个程序员可以填写它并向爬虫添加新的网站，而无需查看一行代码。

当然，缺点是你放弃了一定的灵活性。在第一个示例中，每个网站都有自己的自由形式函数来选择和解析 HTML，以便获得最终结果。在第二个示例中，每个网站需要具有一定的结构，其中字段被保证存在，数据必须在字段出来时保持干净，每个目标字段必须具有唯一且可靠的 CSS 选择器。

但是，我相信这种方法的力量和相对灵活性远远弥补了其实际或被认为的缺点。下一节将涵盖此基本模板的具体应用和扩展，以便您可以处理丢失的字段，收集不同类型的数据，浏览网站的特定部分，并存储关于页面的更复杂的信息。

# 结构化爬虫

创建灵活和可修改的网站布局类型如果仍然需要手动定位要爬取的每个链接，则不会有太大帮助。第六章展示了通过网站并找到新页面的各种自动化方法。

本节展示了如何将这些方法整合到一个结构良好且可扩展的网站爬虫中，该爬虫可以自动收集链接并发现数据。我在这里仅介绍了三种基本的网页爬虫结构；它们适用于你在野外爬取网站时可能遇到的大多数情况，也许在某些情况下需要进行一些修改。

## 通过搜索爬取网站

通过与搜索栏相同的方法，爬取网站是最简单的方法之一。尽管在网站上搜索关键词或主题并收集搜索结果列表的过程似乎因网站而异，但几个关键点使其出奇地简单：

+   大多数网站通过在 URL 参数中将主题作为字符串传递来检索特定主题的搜索结果列表。例如：`http://example.com?search=myTopic`。这个 URL 的前半部分可以保存为`Website`对象的属性，主题只需简单附加即可。

+   搜索后，大多数网站将结果页面呈现为易于识别的链接列表，通常使用方便的包围标签，如`<span class="result">`，其确切格式也可以作为`Website`对象的属性存储。

+   每个*结果链接*都可以是相对 URL（例如，*/articles/page.html*）或绝对 URL（例如，*http://example.com/articles/page.html*）。无论你期望绝对还是相对 URL，它都可以作为`Website`对象的属性存储。

+   当你定位并规范化搜索页面上的 URL 后，你已成功将问题简化为前一节示例中的情况——在给定网站格式的情况下提取页面数据。

让我们来看一下这个算法在代码中的实现。`Content`类与之前的示例大致相同。你正在添加 URL 属性来跟踪内容的来源：

```py
class Content:
    """Common base class for all articles/pages"""

    def __init__(self, topic, url, title, body):
        self.topic = topic
        self.title = title
        self.body = body
        self.url = url

    def print(self):
        """
        Flexible printing function controls output
        """
        print('New article found for topic: {}'.format(self.topic))
        print('URL: {}'.format(self.url))
        print('TITLE: {}'.format(self.title))
        print('BODY:\n{}'.format(self.body))    

```

`Website`类添加了一些新属性。`searchUrl`定义了你应该去哪里获取搜索结果，如果你附加你要查找的主题。`resultListing`定义了包含每个结果信息的“框”，`resultUrl`定义了这个框内部的标签，将为你提供结果的确切 URL。`absoluteUrl`属性是一个布尔值，告诉你这些搜索结果是绝对 URL 还是相对 URL。

```py
class Website:
    """Contains information about website structure"""

    def __init__(self, name, url, searchUrl, resultListing,
​    ​    resultUrl, absoluteUrl, titleTag, bodyTag):
        self.name = name
        self.url = url
        self.searchUrl = searchUrl
        self.resultListing = resultListing
        self.resultUrl = resultUrl
        self.absoluteUrl=absoluteUrl
        self.titleTag = titleTag
        self.bodyTag = bodyTag

```

`Crawler`类也有所扩展。它现在有一个`Website`对象，以及一个指向`Content`对象的 URL 字典，用于跟踪它之前所见过的内容。请注意，`getPage`和`safeGet`方法没有更改，这里省略了它们：

```py
class Crawler:
    def __init__(self, website):
        self.site = website
        self.found = {}

    def getContent(self, topic, url):
        """
        Extract content from a given page URL
        """
        bs = Crawler.getPage(url)
        if bs is not None:
            title = Crawler.safeGet(bs, self.site.titleTag)
            body = Crawler.safeGet(bs, self.site.bodyTag)
            return Content(topic, url, title, body)
        return Content(topic, url, '', '')

    def search(self, topic):
        """
        Searches a given website for a given topic and records all pages found
        """
        bs = Crawler.getPage(self.site.searchUrl + topic)
        searchResults = bs.select(self.site.resultListing)
        for result in searchResults:
            url = result.select(self.site.resultUrl)[0].attrs['href']
            # Check to see whether it's a relative or an absolute URL
            url = url if self.site.absoluteUrl else self.site.url + url
            if url not in self.found:
                self.found[url] = self.getContent(topic, url)
            self.found[url].print()

```

你可以像这样调用你的爬虫：

```py
siteData = [
    ['Reuters', 'http://reuters.com',
     'https://www.reuters.com/search/news?blob=',
     'div.search-result-indiv', 'h3.search-result-title a', 
      False, 'h1', 'div.ArticleBodyWrapper'],
    ['Brookings', 'http://www.brookings.edu',
     'https://www.brookings.edu/search/?s=',
        'div.article-info', 'h4.title a', True, 'h1', 'div.core-block']
]
sites = []
for name, url, search, rListing, rUrl, absUrl, tt, bt in siteData:
    sites.append(Website(name, url, search, rListing, rUrl, absUrl, tt, bt))

crawlers = [Crawler(site) for site in sites]
topics = ['python', 'data%20science']

for topic in topics:
    for crawler in crawlers:
        crawler.search(topic)

```

就像以前一样，关于每个网站的数据数组被创建：标签的外观、URL 和用于跟踪目的的名称。然后将这些数据加载到`Website`对象的列表中，并转换为`Crawler`对象。

然后它会循环遍历`crawlers`列表中的每个爬虫，并为每个特定主题的每个特定站点爬取信息。每次成功收集有关页面的信息时，它都会将其打印到控制台：

```py
New article found for topic: python
URL: http://reuters.com/article/idUSKCN11S04G
TITLE: Python in India demonstrates huge appetite
BODY:
By 1 Min ReadA 20 feet rock python was caught on camera ...

```

注意，它首先遍历所有主题，然后在内部循环中遍历所有网站。为什么不反过来做，先从一个网站收集所有主题，然后再从下一个网站收集所有主题呢？首先遍历所有主题可以更均匀地分配对任何一个 Web 服务器的负载。如果你有数百个主题和几十个网站的列表，这一点尤为重要。你不是一次性向一个网站发送成千上万个请求；你发送 10 个请求，等待几分钟，然后再发送 10 个请求，依此类推。

尽管无论哪种方式，请求的数量最终是相同的，但是通常最好尽量合理地分布这些请求的时间。注意如何结构化你的循环是做到这一点的简单方法。

## 通过链接爬取站点

第六章介绍了在网页上识别内部和外部链接的一些方法，然后利用这些链接来跨站点爬取。在本节中，你将把这些基本方法结合起来，形成一个更灵活的网站爬虫，可以跟随任何匹配特定 URL 模式的链接。

当你想要从一个站点收集所有数据时——而不仅仅是特定搜索结果或页面列表的数据时，这种类型的爬虫效果很好。当站点的页面可能不太有序或广泛分布时，它也能很好地工作。

这些类型的爬虫不需要像在前一节中爬取搜索页面那样定位链接的结构化方法，因此在`Website`对象中不需要描述搜索页面的属性。但是，由于爬虫没有为其正在查找的链接位置/位置提供具体指令，因此您需要一些规则来告诉它选择哪种类型的页面。您提供一个`target​Pat⁠tern`（目标 URL 的正则表达式），并留下布尔变量`absoluteUrl`来完成此操作：

```py
class Website:
    def __init__(self, name, url, targetPattern, absoluteUrl, titleTag, bodyTag):
        self.name = name
        self.url = url
        self.targetPattern = targetPattern
        self.absoluteUrl = absoluteUrl
        self.titleTag = titleTag
        self.bodyTag = bodyTag

class Content:
    def __init__(self, url, title, body):
        self.url = url
        self.title = title
        self.body = body

    def print(self):
        print(f'URL: {self.url}')
        print(f'TITLE: {self.title}')
        print(f'BODY:\n{self.body}')

```

`Content`类与第一个爬虫示例中使用的相同。

`Crawler`类被设计为从每个站点的主页开始，定位内部链接，并解析每个找到的内部链接的内容：

```py
class Crawler:
    def __init__(self, site):
      self.site = site
      self.visited = {}

    def getPage(url):
      try:
            html = urlopen(url)
      except Exception as e:
            print(e)
            return None
      return BeautifulSoup(html, 'html.parser')

    def safeGet(bs, selector):
      selectedElems = bs.select(selector)
      if selectedElems is not None and len(selectedElems) > 0:
            return '\n'.join([elem.get_text() for elem in selectedElems])
      return ''

    def getContent(self, url):
      """
      Extract content from a given page URL
      """
      bs = Crawler.getPage(url)
      if bs is not None:
          title = Crawler.safeGet(bs, self.site.titleTag)
          body = Crawler.safeGet(bs, self.site.bodyTag)
          return Content(url, title, body)
        return Content(url, '', '')

    def crawl(self):
        """
        Get pages from website home page
        """
        bs = Crawler.getPage(self.site.url)
        targetPages = bs.findAll('a', href=re.compile(self.site.targetPattern))
        for targetPage in targetPages:
          url = targetPage.attrs['href']
          url = url if self.site.absoluteUrl else f'{self.site.url}{targetPage}'
          if url not in self.visited:
                self.visited[url] = self.getContent(url)
                self.visited[url].print()

brookings = Website(
    'Brookings', 'https://brookings.edu', '\/(research|blog)\/',
     True, 'h1', 'div.post-body')
crawler = Crawler(brookings)
crawler.crawl()

```

与先前的示例一样，`Website`对象是`Crawler`对象本身的属性。这样可以很好地存储爬虫中访问的页面(`visited`)，但意味着必须为每个网站实例化一个新的爬虫，而不是重复使用同一个爬虫来爬取网站列表。

无论你选择将爬虫设计成与网站无关还是将网站作为爬虫的属性，都是你必须在特定需求背景下权衡的设计决策。两种方法通常都可以接受。

另一个需要注意的是，这个爬虫将从主页获取页面，但在记录了所有这些页面后，它将不会继续爬取。你可能希望编写一个爬虫，采用本章中的模式之一，并让它在访问的每个页面上查找更多目标。甚至可以跟踪每个页面上的所有 URL（不仅限于与目标模式匹配的 URL），以寻找包含目标模式的 URL。

## 爬取多种页面类型

与通过预定页面集合进行爬取不同，通过网站上所有内部链接进行爬取可能会带来挑战，因为你永远不知道确切的内容。幸运的是，有几种方法可以识别页面类型：

通过 URL

网站上的所有博客文章可能都包含一个 URL（例如*http://example.com/blog/title-of-post*）。

通过网站上特定字段的存在或缺失

如果一个页面有日期但没有作者名字，你可能会将其归类为新闻稿。如果它有标题、主图像和价格但没有主要内容，它可能是一个产品页面。

通过页面上特定的标签来识别页面

即使你不收集标签内的数据，你也可以利用标签。例如，你的爬虫可能会查找诸如`<div id="related-products">`这样的元素来识别页面是否为产品页面，尽管爬虫并不关心相关产品的内容。

要跟踪多种页面类型，你需要在 Python 中拥有多种类型的页面对象。有两种方法可以做到这一点。

如果页面都很相似（它们基本上都具有相同类型的内容），你可能希望为现有的网页对象添加一个`pageType`属性：

```py
class Website:
    def __init__(self, name, url, titleTag, bodyTag, pageType):
        self.name = name
        self.url = url
        self.titleTag = titleTag
        self.bodyTag = bodyTag
        self.pageType = pageType

```

如果你将这些页面存储在类似 SQL 的数据库中，这种模式表明所有这些页面可能都存储在同一张表中，并且会添加一个额外的`pageType`列。

如果你要抓取的页面/内容差异很大（它们包含不同类型的字段），这可能需要为每种页面类型创建新的类。当然，一些东西对所有网页都是通用的——它们都有一个 URL，很可能还有一个名称或页面标题。这是使用子类的理想情况：

```py
class Product(Website):
    """Contains information for scraping a product page"""
    def __init__(self, name, url, titleTag, productNumberTag, priceTag):
        Website.__init__(self, name, url, TitleTag)
        self.productNumberTag = productNumberTag
        self.priceTag = priceTag

class Article(Website):
    """Contains information for scraping an article page"""
    def __init__(self, name, url, titleTag, bodyTag, dateTag):
        Website.__init__(self, name, url, titleTag)
        self.bodyTag = bodyTag
        self.dateTag = dateTag

```

这个`Product`页面扩展了`Website`基类，并添加了仅适用于产品的`productNumber`和`price`属性；`Article`类添加了`body`和`date`属性，这些属性不适用于产品。

你可以使用这两个类来抓取，例如，一个商店网站可能除了产品之外还包含博客文章或新闻稿。

# 思考网络爬虫模型

从互联网收集信息有如饮水从火龙头喝水。那里有很多东西，而且并不总是清楚您需要什么或者您如何需要它。任何大型网络抓取项目（甚至某些小型项目）的第一步应该是回答这些问题。

在收集多个领域或多个来源的类似数据时，几乎总是应该尝试对其进行规范化。处理具有相同和可比较字段的数据要比处理完全依赖于其原始来源格式的数据容易得多。

在许多情况下，您应该在假设将来将添加更多数据源到抓取器的基础上构建抓取器，目标是最小化添加这些新来源所需的编程开销。即使一个网站乍一看似乎与您的模型不符，也可能有更微妙的方式使其符合。能够看到这些潜在模式可以在长远中为您节省时间、金钱和许多头疼的问题。

数据之间的关联也不应被忽视。您是否在寻找跨数据源具有“类型”、“大小”或“主题”等属性的信息？您如何存储、检索和概念化这些属性？

软件架构是一个广泛而重要的主题，可能需要整个职业生涯来掌握。幸运的是，用于网络抓取的软件架构是一组相对容易获取的有限且可管理的技能。随着您继续抓取数据，您会发现相同的基本模式一次又一次地出现。创建一个良好结构化的网络爬虫并不需要太多神秘的知识，但确实需要花时间退后一步，思考您的项目。
