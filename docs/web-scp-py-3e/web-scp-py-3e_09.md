# 第八章 Scrapy

第七章介绍了构建大型、可扩展且（最重要的！）可维护网络爬虫的一些技术和模式。尽管手动操作很容易做到，但许多库、框架甚至基于 GUI 的工具都可以为您完成这些工作，或者至少试图让您的生活变得更轻松。

自 2008 年发布以来，Scrapy 迅速发展成为 Python 中最大且维护最好的网络爬虫框架。目前由 Zyte（前身为 Scrapinghub）维护。

编写网络爬虫的一个挑战是经常要重复执行相同的任务：查找页面上的所有链接，评估内部和外部链接之间的差异，并浏览新页面。这些基本模式是有用的，也可以从头编写，但 Scrapy 库可以为您处理其中的许多细节。

当然，Scrapy 不能读心。您仍然需要定义页面模板，指定要开始抓取的位置，并为您正在寻找的页面定义 URL 模式。但在这些情况下，它提供了一个清晰的框架来保持代码的组织。

# 安装 Scrapy

Scrapy 提供了从其网站[下载](http://scrapy.org/download/)的工具，以及使用 pip 等第三方安装管理器安装 Scrapy 的说明。

由于其相对较大的大小和复杂性，Scrapy 通常不是可以通过传统方式安装的框架：

```py
$ pip install Scrapy

```

请注意，我说“通常”是因为尽管理论上可能存在，但我通常会遇到一个或多个棘手的依赖问题、版本不匹配和无法解决的错误。

如果您决定从 pip 安装 Scrapy，则强烈建议使用虚拟环境（有关虚拟环境的更多信息，请参阅“使用虚拟环境保持库的整洁”）。

我偏爱的安装方法是通过[Anaconda 包管理器](https://docs.continuum.io/anaconda/)。Anaconda 是由 Continuum 公司推出的产品，旨在简化查找和安装流行的 Python 数据科学包的过程。后续章节将使用它管理的许多包，如 NumPy 和 NLTK。

安装完 Anaconda 后，您可以使用以下命令安装 Scrapy：

```py
`conda` `install` `-``c` `conda``-``forge` `scrapy`
```

如果遇到问题或需要获取最新信息，请查阅[Scrapy](https://doc.scrapy.org/en/latest/intro/install.html)安装指南获取更多信息。

## 初始化新的 Spider

安装完 Scrapy 框架后，每个 Spider 需要进行少量设置。*Spider*是一个 Scrapy 项目，类似于其命名的蜘蛛，专门设计用于爬取网络。在本章中，我使用“Spider”来描述特定的 Scrapy 项目，“crawler”指“使用 Scrapy 或不使用任何通用程序爬取网络”的任何通用程序。

要在当前目录下创建一个新的 Spider，请从命令行运行以下命令：

```py
$ scrapy startproject wikiSpider
```

这会在项目创建的目录中创建一个新的子目录，名为*wikiSpider*。在这个目录中，有以下文件结构：

+   *scrapy.cfg*

+   *wikiSpider*

    +   *spiders*

        +   *__init.py__*

    +   *items.py*

    +   *middlewares.py*

    +   *pipelines.py*

    +   *settings.py*

    +   *__init.py__*

这些 Python 文件以存根代码初始化，以提供创建新爬虫项目的快速方式。本章中的每个部分都与这个*wikiSpider*项目一起工作。

# 编写一个简单的爬虫

要创建一个爬虫，您将在*wiki​S⁠pider/wikiSpider/spiders/article.py*的子*spiders*目录中添加一个新文件。这是所有爬虫或扩展 scrapy.Spider 的内容的地方。在您新创建的*article.py*文件中，写入：

```py
from scrapy import Spider, Request

class ArticleSpider(Spider):
    name='article'

    def start_requests(self):
        urls = [
            'http://en.wikipedia.org/wiki/Python_%28programming_language%29',
            'https://en.wikipedia.org/wiki/Functional_programming',
            'https://en.wikipedia.org/wiki/Monty_Python']
        return [Request(url=url, callback=self.parse) for url in urls]

    def parse(self, response):
        url = response.url
        title = response.css('h1::text').extract_first()
        print(f'URL is: {url}')
        print(f'Title is: {title}')

```

此类的名称（`ArticleSpider`）根本没有涉及“wiki”或“维基百科”，这表明此类特别负责爬取文章页面，属于更广泛的*wikiSpider*类别，稍后您可能希望使用它来搜索其他页面类型。

对于内容类型繁多的大型网站，您可能会为每种类型（博客文章、新闻稿、文章等）设置单独的 Scrapy 项目。每个爬虫的名称在项目内必须是唯一的。

关于此爬虫的另外两个关键点是函数`start_requests`和`parse`：

`start_requests`

Scrapy 定义的程序入口点用于生成`Request`对象，Scrapy 用它来爬取网站。

`parse`

由用户定义的回调函数，并通过`Request`对象传递给`callback=self.parse`。稍后，您将看到更多可以使用`parse`函数完成的功能，但现在只打印页面的标题。

你可以通过转到外部*wikiSpider*目录并运行以下命令来运行这个`article`爬虫：

```py
$ scrapy runspider wikiSpider/spiders/article.py

```

默认的 Scrapy 输出相当冗长。除了调试信息外，这应该会打印出类似以下的行：

```py
2023-02-11 21:43:13 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/robots.txt> (referer: None)
2023-02-11 21:43:14 [scrapy.downloadermiddlewares.redirect] DEBUG: Redirecting (3
01) to <GET https://en.wikipedia.org/wiki/Python_%28programming_language%29> from
 <GET http://en.wikipedia.org/wiki/Python_%28programming_language%29>
2023-02-11 21:43:14 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/wiki/Functional_programming> (referer: None)
2023-02-11 21:43:14 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/wiki/Monty_Python> (referer: None)
URL is: https://en.wikipedia.org/wiki/Functional_programming
Title is: Functional programming
2023-02-11 21:43:14 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/wiki/Python_%28programming_language%29> (referer: None)
URL is: https://en.wikipedia.org/wiki/Monty_Python
Title is: Monty Python
URL is: https://en.wikipedia.org/wiki/Python_%28programming_language%29
Title is: Python (programming language)

```

爬虫访问列出为 URL 的三个页面，收集信息，然后终止。

# 使用规则进行爬取

前一节中的爬虫并不是一个真正的爬虫，只能限于仅抓取提供的 URL 列表。它没有能力自行查找新页面。要将其转变为一个完全成熟的爬虫，你需要使用 Scrapy 提供的`CrawlSpider`类。

# 在 GitHub 仓库中的代码组织

不幸的是，Scrapy 框架不能在 Jupyter 笔记本中轻松运行，这使得线性编码难以实现。为了在文本中展示所有代码示例的目的，前一节中的爬虫存储在*article.py*文件中，而下面的示例，创建一个遍历多个页面的 Scrapy 爬虫，存储在*articles.py*中（注意使用了复数形式）。

后续示例也将存储在单独的文件中，每个部分都会给出新的文件名。运行这些示例时，请确保使用正确的文件名。

此类可以在 GitHub 仓库的 spiders 文件*articles.py*中找到：

```py
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule

class ArticleSpider(CrawlSpider):
    name = 'articles'
    allowed_domains = ['wikipedia.org']
    start_urls = ['https://en.wikipedia.org/wiki/Benevolent_dictator_for_life']
    rules = [
        Rule(
            LinkExtractor(allow=r'.*'),
​            callback='parse_items',
​            follow=True
​        )
    ]

    def parse_items(self, response):
        url = response.url
        title = response.css('span.mw-page-title-main::text').extract_first()
        text = response.xpath('//div[@id="mw-content-text"]//text()').extract()
        lastUpdated = response.css(
            'li#footer-info-lastmod::text'
        ).extract_first()
        lastUpdated = lastUpdated.replace('This page was last edited on ', '')
        print(f'URL is: {url}')
        print(f'Title is: {title} ')
        print(f'Text is: {text}')
        print(f'Last updated: {lastUpdated}')

```

这个新的`ArticleSpider`扩展了`CrawlSpider`类。它不是提供一个`start_requests`函数，而是提供一个`start_urls`和`allowed_domains`列表。这告诉爬虫从哪里开始爬取，并根据域名是否应跟踪或忽略链接。

还提供了一个`rules`列表。这提供了进一步的说明，指导哪些链接应该跟踪或忽略（在本例中，允许所有使用正则表达式`.*`的 URL）。

除了提取每个页面的标题和 URL 之外，还添加了一些新的项目。使用 XPath 选择器提取每个页面的文本内容。XPath 在检索文本内容时经常被使用，包括文本在子标签中的情况（例如，在一段文本中的`<a>`标签内）。如果您使用 CSS 选择器来做这件事，所有子标签中的文本都将被忽略。

也从页脚解析并存储了最后更新日期字符串到`lastUpdated`变量中。

您可以通过导航到*wikiSpider*目录并运行以下命令来运行此示例：

```py
$ scrapy runspider wikiSpider/spiders/articles.py
```

# 警告：这将永远运行下去

尽管这个新的爬虫在命令行中的运行方式与前一节中构建的简单爬虫相同，但它不会终止（至少不会很长时间），直到您使用 Ctrl-C 终止执行或关闭终端为止。请注意，对 Wikipedia 服务器负载要友善，不要长时间运行它。

运行此爬虫时，它将遍历*wikipedia.org*，跟踪* wikipedia.org*域下的所有链接，打印页面标题，并忽略所有外部（站外）链接：

```py
2023-02-11 22:13:34 [scrapy.spidermiddlewares.offsite] DEBUG: Filtered offsite
 request to 'drupal.org': <GET https://drupal.org/node/769>
2023-02-11 22:13:34 [scrapy.spidermiddlewares.offsite] DEBUG: Filtered offsite
 request to 'groups.drupal.org': <GET https://groups.drupal.org/node/5434>
2023-02-11 22:13:34 [scrapy.spidermiddlewares.offsite] DEBUG: Filtered offsite
 request to 'www.techrepublic.com': <GET https://www.techrepublic.com/article/
open-source-shouldnt-mean-anti-commercial-says-drupal-creator-dries-buytaert/>
2023-02-11 22:13:34 [scrapy.spidermiddlewares.offsite] DEBUG: Filtered offsite
 request to 'www.acquia.com': <GET https://www.acquia.com/board-member/dries-b
uytaert>

```

到目前为止，这是一个相当不错的爬虫，但它可以使用一些限制。它不仅可以访问维基百科上的文章页面，还可以自由漫游到非文章页面，例如：

```py
title is: Wikipedia:General disclaimer

```

通过使用 Scrapy 的`Rule`和`LinkExtractor`来仔细查看这一行：

```py
rules = [Rule(LinkExtractor(allow=r'.*'), callback='parse_items',
​    follow=True)]
```

此行提供了一个 Scrapy `Rule`对象的列表，该列表定义了所有找到的链接通过的规则。当存在多个规则时，每个链接都会根据规则进行检查，按顺序进行。第一个匹配的规则将用于确定如何处理链接。如果链接不符合任何规则，则会被忽略。

可以为`Rule`提供四个参数：

`link_extractor`

唯一必需的参数，一个`LinkExtractor`对象。

`callback`

应该用于解析页面内容的函数。

`cb_kwargs`

要传递给回调函数的参数字典。这个字典格式为`{arg_name1: arg_value1, arg_name2: arg_value2}`，对于稍微不同的任务重用相同的解析函数非常方便。

`follow`

指示是否希望在未来的爬行中包含在该页面上找到的链接。如果未提供回调函数，则默认为`True`（毕竟，如果您对页面没有做任何事情，那么至少您可能想要继续通过站点进行爬行）。如果提供了回调函数，则默认为`False`。

`LinkExtractor`是一个简单的类，专门设计用于识别并返回基于提供的规则在 HTML 内容页中的链接。它具有许多参数，可用于接受或拒绝基于 CSS 和 XPath 选择器、标签（您不仅可以查找锚点标签中的链接！）、域名等的链接。

`LinkExtractor`类甚至可以扩展，并且可以创建自定义参数。详见 Scrapy 的[链接提取器文档](https://doc.scrapy.org/en/latest/topics/link-extractors.html)获取更多信息。

尽管`LinkExtractor`类具有灵活的特性，但您最常使用的参数是这些：

`allow`

允许所有符合提供的正则表达式的链接。

`deny`

拒绝所有符合提供的正则表达式的链接。

使用两个单独的`Rule`和`LinkExtractor`类以及一个解析函数，您可以创建一个爬虫，该爬虫可以爬取维基百科，识别所有文章页面，并标记非文章页面（*articleMoreRules.py*）：

```py
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule

class ArticleSpider(CrawlSpider):
    name = 'articles'
    allowed_domains = ['wikipedia.org']
    start_urls = ['https://en.wikipedia.org/wiki/Benevolent_dictator_for_life']
    rules = [
        Rule(
            LinkExtractor(allow='(/wiki/)((?!:).)*$'),
            callback='parse_items',
            follow=True,
            cb_kwargs={'is_article': True}
        ),
        Rule(
            LinkExtractor(allow='.*'),
            callback='parse_items',
            cb_kwargs={'is_article': False}
        )
    ]

    def parse_items(self, response, is_article):
        print(response.url)
        title = response.css('span.mw-page-title-main::text').extract_first()
        if is_article:
            url = response.url
            text = response.xpath(
                '//div[@id="mw-content-text"]//text()'
            ).extract()
            lastUpdated = response.css(
                'li#footer-info-lastmod::text'
            ).extract_first()
            lastUpdated = lastUpdated.replace(
                'This page was last edited on ',
                ''
            )
            print(f'URL is: {url}')
            print(f'Title is: {title}')
            print(f'Text is: {text}')
        else:
            print(f'This is not an article: {title}')

```

请记住，规则适用于列表中呈现的每个链接。首先将所有文章页面（以*/wiki/*开头且不包含冒号的页面）传递给`parse_items`函数，并使用默认参数`is_article=True`。然后将所有其他非文章链接传递给`parse_items`函数，并使用参数`is_article=False`。

当然，如果您只想收集文章类型页面并忽略所有其他页面，这种方法将不太实际。忽略不匹配文章 URL 模式的页面，并完全省略第二个规则（以及`is_article`变量），会更加简单。但在 URL 中收集的信息或在爬取过程中收集的信息影响页面解析方式的奇特情况下，这种方法可能会很有用。

# 创建项目

到目前为止，您已经学习了许多在 Scrapy 中查找、解析和爬取网站的方法，但 Scrapy 还提供了有用的工具，可以将您收集的项目组织并存储在具有明确定义字段的自定义对象中。

为了帮助组织您收集的所有信息，您需要创建一个名为`Article`的对象。在*items.py*文件内定义一个新的`Article`项目。

当您打开*items.py*文件时，它应该看起来像这样：

```py
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

import scrapy

class WikispiderItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

```

用一个新的扩展`scrapy.Item`的`Article`类替换此默认的`Item`存根：

```py
import scrapy

class Article(scrapy.Item):
    url = scrapy.Field()
    title = scrapy.Field()
    text = scrapy.Field()
    lastUpdated = scrapy.Field()

```

您正在定义将从每个页面收集的四个字段：URL、标题、文本内容和页面最后编辑日期。

如果您正在收集多种页面类型的数据，应将每种单独类型定义为*items.py*中的一个单独类。如果您的项目很大，或者您开始将更多的解析功能移动到项目对象中，您可能还希望将每个项目提取到自己的文件中。然而，当项目很小时，我喜欢将它们保存在单个文件中。

在文件*articleItems.py*中，请注意在 `ArticleSpider` 类中所做的更改，以创建新的 `Article` 项目：

```py
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from wikiSpider.items import Article

class ArticleSpider(CrawlSpider):
    name = 'articleItems'
    allowed_domains = ['wikipedia.org']
    start_urls = ['https://en.wikipedia.org/wiki/Benevolent'
​    ​    '_dictator_for_life']
    rules = [
        Rule(LinkExtractor(allow='(/wiki/)((?!:).)*$'),
​    ​    ​    callback='parse_items', follow=True),
    ]

    def parse_items(self, response):
        article = Article()
        article['url'] = response.url
        article['title'] = response.css('h1::text').extract_first()
        article['text'] = response.xpath('//div[@id='
​    ​    ​    '"mw-content-text"]//text()').extract()
        lastUpdated = response.css('li#footer-info-lastmod::text'
​    ​    ​    ).extract_first()
        article['lastUpdated'] = lastUpdated.replace('This page was '
​    ​    ​    'last edited on ', '')
        return article

```

运行此文件时

```py
$ scrapy runspider wikiSpider/spiders/articleItems.py

```

它将输出通常的 Scrapy 调试数据以及每个文章项目作为 Python 字典：

```py
2023-02-11 22:52:26 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/wiki/Benevolent_dictator_for_life#bodyContent> (referer: https://en.wi
kipedia.org/wiki/Benevolent_dictator_for_life)
2023-02-11 22:52:26 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/wiki/OCaml> (referer: https://en.wikipedia.org/wiki/Benevolent_dictato
r_for_life)
2023-02-11 22:52:26 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://en.wik
ipedia.org/wiki/Xavier_Leroy> (referer: https://en.wikipedia.org/wiki/Benevolent_
dictator_for_life)
2023-02-11 22:52:26 [scrapy.core.scraper] DEBUG: Scraped from <200 https://en.wik
ipedia.org/wiki/Benevolent_dictator_for_life>
{'lastUpdated': ' 7 February 2023, at 01:14',
'text': 'Title given to a small number of open-source software development '
          'leaders',
          ...

```

使用 Scrapy `Items` 并不仅仅是为了促进良好的代码组织或以可读的方式布置事物。项目提供了许多工具，用于输出和处理数据，这些将在接下来的部分中详细介绍。

# 输出项目

Scrapy 使用 `Item` 对象来确定应从其访问的页面中保存哪些信息。这些信息可以由 Scrapy 以各种方式保存，例如 CSV、JSON 或 XML 文件，使用以下命令：

```py
$ scrapy runspider articleItems.py -o articles.csv -t csv
$ scrapy runspider articleItems.py -o articles.json -t json
$ scrapy runspider articleItems.py -o articles.xml -t xml
```

每次运行抓取器 `articleItems` 并将输出按指定格式写入提供的文件。如果不存在，则将创建此文件。

您可能已经注意到，在先前示例中爬虫创建的文章中，文本变量是一个字符串列表，而不是单个字符串。此列表中的每个字符串表示单个 HTML 元素内的文本，而`<div id="mw-content-text">`内的内容（您正在收集的文本数据）由许多子元素组成。

Scrapy 很好地管理这些更复杂的值。例如，在 CSV 格式中，它将列表转换为字符串，并转义所有逗号，以便文本列表显示在单个 CSV 单元格中。

在 XML 中，此列表的每个元素都被保留在子值标签中：

```py
<items>
<item>
    <url>https://en.wikipedia.org/wiki/Benevolent_dictator_for_life</url>
    <title>Benevolent dictator for life</title>
    <text>
        <value>For the political term, see </value>
        <value>Benevolent dictatorship</value>
        ...
    </text>
    <lastUpdated> 7 February 2023, at 01:14.</lastUpdated>
</item>
....

```

在 JSON 格式中，列表被保留为列表。

当然，您可以自己使用 `Item` 对象，并通过将适当的代码添加到爬虫的解析函数中，以任何您想要的方式将它们写入文件或数据库。

# 项目管道

尽管 Scrapy 是单线程的，但它能够异步地进行许多请求的处理。这使其比本书中迄今编写的爬虫更快，尽管我一直坚信，在涉及网络抓取时，更快并不总是更好。

您正在尝试抓取的网站的网络服务器必须处理这些请求中的每一个，评估这种服务器压力是否合适（甚至对您自己的利益是否明智，因为许多网站有能力和意愿阻止它们可能认为是恶意的抓取活动）。有关网络抓取伦理以及适当地调节抓取程序重要性的更多信息，请参阅[第十九章。

话虽如此，使用 Scrapy 的项目管道可以通过在等待请求返回时执行所有数据处理来进一步提高网络爬虫的速度，而不是等待数据处理完成后再发出另一个请求。当数据处理需要大量时间或处理器密集型计算时，这种优化甚至可能是必需的。

要创建一个项目管道，请重新访问本章开头创建的 *settings.py* 文件。您应该看到以下被注释的行：

```py
# Configure item pipelines
# See http://scrapy.readthedocs.org/en/latest/topics/item-pipeline.html
#ITEM_PIPELINES = {
#    'wikiSpider.pipelines.WikispiderPipeline': 300,
#}

```

取消对最后三行的注释，并替换为：

```py
ITEM_PIPELINES = {
    'wikiSpider.pipelines.WikispiderPipeline': 300,
}

```

这提供了一个 Python 类 `wikiSpider.pipelines.WikispiderPipeline`，用于处理数据，以及一个表示运行管道顺序的整数。虽然可以使用任何整数，但通常使用 0 到 1000 的数字，并将按升序运行。

现在，您需要添加管道类并重写原始爬虫，以便爬虫收集数据，管道执行数据处理的繁重工作。也许会诱人地在原始爬虫中编写 `parse_items` 方法以返回响应并让管道创建 `Article` 对象：

```py
    def parse_items(self, response):
        return response

```

然而，Scrapy 框架不允许这样做，必须返回一个 `Item` 对象（例如扩展 `Item` 的 `Article`）。因此，现在 `parse_items` 的目标是提取原始数据，尽可能少地进行处理，以便可以传递到管道：

```py
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from wikiSpider.items import Article

class ArticleSpider(CrawlSpider):
    name = 'articlePipelines'
    allowed_domains = ['wikipedia.org']
    start_urls = ['https://en.wikipedia.org/wiki/Benevolent_dictator_for_life']
    rules = [
        Rule(LinkExtractor(allow='(/wiki/)((?!:).)*$'),
​    ​    ​    callback='parse_items', follow=True),
    ]

    def parse_items(self, response):
        article = Article()
        article['url'] = response.url
        article['title'] = response.css('h1::text').extract_first()
        article['text'] = response.xpath('//div[@id='
​    ​    ​    '"mw-content-text"]//text()').extract()
        article['lastUpdated'] = response.css('li#'
​    ​    ​    'footer-info-lastmod::text').extract_first()
        return article

```

此文件保存为 *articlePipelines.py* 在 GitHub 仓库中。

当然，现在您需要通过添加管道将 *pipelines.py* 文件和更新的爬虫联系在一起。当最初初始化 Scrapy 项目时，会在 *wikiSpider/wikiSpider/pipelines.py* 创建一个文件：

```py
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html

class WikispiderPipeline(object):
    def process_item(self, item, spider):
        return item
```

这个存根类应该用您的新管道代码替换。在之前的章节中，您已经以原始格式收集了两个字段，并且这些字段可能需要额外的处理：`lastUpdated`（一个糟糕格式化的表示日期的字符串对象）和 `text`（一个杂乱的字符串片段数组）。

以下应该用于替换 *wikiSpider/wikiSpider/pipelines.py* 中的存根代码：

```py
from datetime import datetime
from wikiSpider.items import Article
from string import whitespace

class WikispiderPipeline(object):
    def process_item(self, article, spider):
        dateStr = article['lastUpdated']
        article['lastUpdated'] = article['lastUpdated']
​    ​    ​    .replace('This page was last edited on', '')
        article['lastUpdated'] = article['lastUpdated'].strip()
        article['lastUpdated'] = datetime.strptime(
​    ​    ​    article['lastUpdated'], '%d %B %Y, at %H:%M.')
        article['text'] = [line for line in article['text']
​    ​    ​    if line not in whitespace]
        article['text'] = ''.join(article['text'])
        return article

```

类 `WikispiderPipeline` 具有一个 `process_item` 方法，接受一个 `Article` 对象，将 `lastUpdated` 字符串解析为 Python 的 `datetime` 对象，并从字符串列表中清理和连接文本为单个字符串。

`process_item` 是每个管道类的必需方法。Scrapy 使用此方法异步传递由爬虫收集的 `Items`。例如，如果您在上一节中输出到 JSON 或 CSV，这里返回的解析的 `Article` 对象将由 Scrapy 记录或打印。

现在，在决定数据处理位置时，您有两个选择：在爬虫中的 `parse_items` 方法或在管道中的 `process_items` 方法。

可以在 *settings.py* 文件中声明具有不同任务的多个管道。但是，Scrapy 将所有项目不论类型都传递给每个管道以便处理。在数据传入管道之前，可能更好地在爬虫中处理特定项目的解析。但是，如果此解析需要很长时间，可以考虑将其移到管道中（可以异步处理），并添加一个项目类型检查：

```py
def process_item(self, item, spider):    
    if isinstance(item, Article):
        # Article-specific processing here

```

在编写 Scrapy 项目时，特别是在处理大型项目时，确定要执行的处理及其执行位置是一个重要考虑因素。

# 使用 Scrapy 记录日志

Scrapy 生成的调试信息可能很有用，但你可能已经注意到，它通常过于冗长。你可以通过在 Scrapy 项目的 *settings.py* 文件中添加一行来轻松调整日志级别：

```py
LOG_LEVEL = 'ERROR'
```

Scrapy 使用标准的日志级别层次结构，如下：

+   `CRITICAL`

+   `ERROR`

+   `WARNING`

+   `DEBUG`

+   `INFO`

如果日志级别设置为 `ERROR`，则只会显示 `CRITICAL` 和 `ERROR` 级别的日志。如果设置为 `INFO`，则会显示所有日志，依此类推。

除了通过 *settings.py* 文件控制日志记录外，还可以通过命令行控制日志输出位置。在命令行中运行时，可以定义一个日志文件，将日志输出到该文件而不是终端：

```py
$ scrapy crawl articles -s LOG_FILE=wiki.log
```

这将在当前目录中创建一个新的日志文件（如果不存在），并将所有日志输出到该文件，使得你的终端只显示手动添加的 Python 打印语句。

# 更多资源

Scrapy 是一个强大的工具，可以处理与网络爬取相关的许多问题。它会自动收集所有 URL，并将它们与预定义的规则进行比较，确保所有 URL 都是唯一的，在必要时会对相对 URL 进行标准化，并递归深入页面。

鼓励您查阅[Scrapy 文档](https://doc.scrapy.org/en/latest/news.html)以及[Scrapy 的官方教程页面](https://docs.scrapy.org/en/latest/intro/tutorial.html)，这些资源详细介绍了该框架。

Scrapy 是一个非常庞大的库，具有许多功能。它的特性可以无缝协同工作，但也存在许多重叠的区域，使用户可以轻松地在其中开发自己的风格。如果你想要使用 Scrapy 做一些这里未提到的事情，很可能有一种（或几种）方法可以实现！
