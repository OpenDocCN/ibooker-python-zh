# 第十九章 客户端网络协议模块

Python 的标准库提供了几个模块，简化了客户端和服务器端使用互联网协议的过程。如今，被称为 *PyPI* 的 [Python 包索引](https://oreil.ly/PGIim) 提供了更多类似的包。因为许多标准库模块可以追溯到上个世纪，您会发现现在第三方包支持更多的协议，并且一些比标准库提供的 API 更好。当您需要使用标准库中缺少的或者您认为标准库中的方式不尽如人意的网络协议时，请务必搜索 PyPI——您很可能会在那里找到更好的解决方案。

本章介绍了一些标准库包，允许相对简单地使用网络协议：这些包使您可以在不需要第三方包的情况下编码，使您的应用程序或库更容易安装在其他计算机上。因此，在处理遗留代码时可能会遇到它们，它们的简单性也使它们成为 Python 学习者感兴趣的阅读材料。

对于 HTTP 客户端和其他最好通过 URL 访问的网络资源（如匿名 FTP 站点）的频繁使用情况，第三方 [requests 包](https://oreil.ly/t4X8r) 甚至被 Python 文档推荐，因此我们涵盖了它并推荐使用，而不是使用标准库模块。

# 电子邮件协议

今天，大多数电子邮件是通过实现简单邮件传输协议（SMTP）的服务器*发送*，并通过使用邮局协议版本 3（POP3）和/或互联网消息访问协议版本 4（IMAP4）的服务器和客户端*接收*。¹ 这些协议的客户端由 Python 标准库模块 smtplib、poplib 和 imaplib 支持，我们在本书中介绍了前两者。当您需要处理电子邮件消息的*解析*或*生成*时，请使用电子邮件包，本书在 第二十一章 中有所介绍。

如果你需要编写一个既可以通过 POP3 也可以通过 IMAP4 连接的客户端，一个标准的建议是选择 IMAP4，因为它更强大，并且——根据 Python 自己的在线文档——通常在服务器端实现得更精确。不幸的是，imaplib 非常复杂，远超过本书的涵盖范围。如果你选择这条路，需要使用[在线文档](https://oreil.ly/3ncIi)，并且不可避免地要补充 IMAP RFCs，可能还包括其他相关的 RFCs，如 5161 和 6855（用于能力）以及 2342（用于命名空间）。除了标准库模块的在线文档外，还必须使用 RFCs：imaplib 函数和方法传递的许多参数以及调用它们的结果，都是字符串格式，只有 RFCs 中有详细说明，而 Python 的文档中没有。一个强烈推荐的替代方案是使用更简单、更高抽象级别的第三方包[IMAPClient](https://oreil.ly/xTc4T)，可以通过**pip install**安装，并且有很好的[在线文档](https://oreil.ly/SuiI_)。

## poplib 模块

poplib 模块提供了一个访问 POP 邮箱的 POP3 类。² 构造函数具有以下签名：

| POP3 | **class** POP3(*host*, port=110) 返回一个连接到指定*host*和端口的 POP3 类实例*p*。类 POP3_SSL 的行为完全相同，但通过安全的 TLS 通道连接到主机（默认端口 995）；这对需要连接到要求一定最小安全性的电子邮件服务器（如 pop.gmail.com）是必需的。^(a) |
| --- | --- |
| ^(a) 要连接到 Gmail 账户，特别是，你需要配置该账户以启用 POP，“允许不安全的应用程序”，并避免两步验证——这些一般情况下我们不推荐，因为它们会削弱你的电子邮件安全性。 |

类 POP3 的实例*p*提供了许多方法；最常用的列在表 19-1 中。在每种情况下，*msgnum*，一个消息的标识号，可以是包含整数值的字符串或整数。

表 19-1\. POP3 类实例*p*的方法

| dele | *p*.dele(*msgnum*) 标记消息*msgnum*以便删除，并返回服务器响应字符串。服务器会排队这样的删除请求，只有在你通过调用*p*.quit 终止连接时才执行。^(a) |
| --- | --- |
| list | *p*.list(*msgnum*=**None**) 返回一个三元组(*response*, *messages*, *octets*)，其中*response*是服务器响应字符串；*messages*是一个由字节串组成的列表，每个字节串由两个词组成 b'*msgnum* *bytes*'，每个消息的消息编号和字节长度；*octets*是总响应的字节长度。当*msgnum*不为**None**时，list 返回一个字符串，给定*msgnum*的响应，而不是一个元组。 |
| pass_ | *p*.pass_(*password*) 向服务器发送密码，并返回服务器响应字符串。必须在 *p*.user 之后调用。名称中的下划线是因为 **pass** 是 Python 的关键字。 |
| quit | *p*.quit() 结束会话并告知服务器执行调用 *p*.dele 请求的删除操作。返回服务器响应字符串。 |
| retr | *p*.retr(*msgnum*) 返回一个三元组（*response*, *lines*, *bytes*），其中 *response* 是服务器响应字符串，*lines* 是消息 *msgnum* 的所有行的列表（以字节串形式），*bytes* 是消息的总字节数。 |
| s⁠e⁠t⁠_​d⁠e⁠b⁠u⁠g⁠l⁠e⁠v⁠e⁠l | *p*.set_debuglevel(*debug_level*) 将调试级别设置为 *debug_level*，一个整数，值为 0（默认）表示无调试输出，1 表示适量的调试输出，2 或更高表示所有与服务器交换的控制信息的完整输出跟踪。 |
| stat | *p*.stat() 返回一个二元组（*num_msgs*, *bytes*），其中 *num_msgs* 是邮箱中的消息数，*bytes* 是总字节数。 |
| top | *p*.top(*msgnum*, *maxlines*) 类似 retr，但最多返回消息体的 *maxlines* 行（除了所有的头部行）。对于查看长消息开头很有用。 |
| user | *p*.user(*username*) 向服务器发送用户名；随后必然调用 *p*.pass_。 |
| ^(a) 标准规定，如果在 quit 调用之前发生断开连接，不应执行删除操作。尽管如此，某些服务器在任何断开连接后（计划或非计划）都会执行删除操作。 |

## smtplib 模块

smtplib 模块提供一个 SMTP 类来通过 SMTP 服务器发送邮件。³ 构造函数的签名如下：

| SMTP | class SMTP([*host*, port=25]) 返回 SMTP 类的实例 *s*。当给定 *host*（和可选的 port）时，隐式调用 *s*.connect(*host*, port)。SMTP_SSL 类的行为完全相同，但通过安全的 TLS 通道连接到主机（默认端口 465），用于连接要求一定最小安全性的电子邮件服务器，如 smtp.gmail.com。 |
| --- | --- |

SMTP 类的实例 *s* 提供许多方法。其中最常用的列在 表 19-2 中。

表 19-2\. SMTP 实例 s 的方法

| connect | *s*.connect(host=127.0.0.1, port=25) 连接到给定主机（默认为本地主机）和端口的 SMTP 服务器（SMTP 服务的默认端口为 25；更安全的“SMTP over TLS”默认端口为 465）。 |
| --- | --- |
| login | *s*.login(*user*, *password*) 使用给定的 *user* 和 *password* 登录服务器。只有在 SMTP 服务器需要身份验证时才需要（几乎所有服务器都需要）。 |
| quit | *s*.quit() 终止 SMTP 会话。 |
| sendmail | *s*.sendmail(*from_addr*, *to_addrs*, *msg_string*) 从字符串*from_addr*中发送邮件消息*msg_string*，并分别发送给列表*to_addrs*中的每个收件人。^(a) *msg_string*必须是一个完整的 RFC 822 消息，是一个多行的字节字符串：包括头部、用于分隔的空行，然后是正文。邮件传输机制仅使用*from_addr*和*to_addrs*来确定路由，忽略*msg_string*中的任何头部。^(b) 要准备符合 RFC 822 的消息，请使用 email 包，该包在“MIME 和电子邮件格式处理”中介绍。 |
| send_message | *s.*send_message(*msg,* from_addr=**None**, to_addrs=**None**) 这是一个便捷函数，第一个参数为 email.message.Message 对象。如果*from_addr*和*to_addrs*中任一或两者为**None**，则会从消息中提取它们。 |
| ^(a) 标准并未限制*from_addr*中收件人的数量，但是各个邮件服务器可能会限制，因此建议每个批处理消息中的收件人数量不要太多。^(b) 这样做可以支持邮件系统实现密送（Bcc）邮件，因为路由不依赖于消息信封。 |

# HTTP 和 URL 客户端

绝大多数情况下，您的代码会通过更高级别的 URL 层使用 HTTP 和 FTP 协议，这些协议由下一节介绍的模块和包支持。Python 标准库还提供了较低级别的、特定于协议的模块，不过这些模块使用频率较低：对于 FTP 客户端，使用[ftplib](https://oreil.ly/O_XHc)；对于 HTTP 客户端，使用 http.client（我们在第二十章介绍 HTTP 服务器）。如果需要编写 FTP 服务器，请考虑第三方模块[pyftpdlib](https://oreil.ly/Qrvcn)。在撰写本书时，较新的[HTTP/2](https://http2.github.io)实现可能尚不完全成熟，但目前最佳选择是第三方模块[HTTPX](https://www.python-httpx.org)。我们在本书中不涉及这些较低级别的模块，而是专注于整个下一节的更高级别、URL 级别的访问。

## URL 访问

URI 是一种统一资源标识符（URI）的一种类型。URI 是一个字符串，用于*标识*资源（但不一定*定位*资源），而 URL 用于在互联网上*定位*资源。URL 由几个部分（某些可选）组成，称为*组件*：*scheme, location, path, query*和*fragment*。（第二个组件有时也被称为*网络位置*或简称*netloc*。）具有所有部分的 URL 看起来像这样：

```py
*scheme://lo.ca.ti.on/pa/th?qu=ery#fragment*
```

例如，在*https://www.python.org/community/awards/psf-awards/#october-2016*中，方案为*http*，位置为*www.python.org*，路径为*/community/awards/psf-awards/*，无查询，片段为*#october-2016*。（大多数方案在未明确指定端口时会默认使用一个*众所周知的端口*；例如，HTTP 方案的众所周知端口为 80。）某些标点符号是其分隔的组件的一部分；其他标点字符只是分隔符，并非任何组件的一部分。省略标点符号意味着缺少组件。例如，在*mailto:me@you.com*中，方案为*mailto*，路径为*me@you.com*（*mailto:me@you.com*），无位置、查询或片段。没有//表示 URI 无位置，没有？表示 URI 无查询，没有#表示 URI 无片段。

如果位置以冒号结尾，后跟一个数字，则表示终点的 TCP 端口。否则，连接使用与方案关联的众所周知端口（例如，HTTP 的端口 80）。

## urllib 包

urllib 包提供了几个模块来解析和利用 URL 字符串及其相关资源。除了这里描述的 urllib.parse 和 urllib.request 模块外，还包括 urllib.robotparser 模块（专门用于根据[RFC 9309](https://oreil.ly/QI7CQ)解析站点的*robots.txt*文件）和 urllib.error 模块，包含其他 urllib 模块引发的所有异常类型。

### urllib.parse 模块

urllib.parse 模块提供了用于分析和合成 URL 字符串的函数，并通常使用**from** urllib **import** parse **as** urlparse 导入。其最常用的函数列在表 19-3 中。

表 19-3\. urllib.parse 模块的常用函数

| urljoin | urljoin(*base_url_string*, *relative_url_string*) 返回一个 URL 字符串*u*，该字符串通过将*relative_url_string*（可能是相对的）与*base_url_string*连接而得到。urljoin 执行的连接过程可总结如下：

+   当任一参数字符串为空时，*u*即为另一个参数。

+   当*relative_url_string*明确指定与*base_url_string*不同的方案时，*u*为*relative_url_string*。否则，*u*的方案为*base_url_string*的方案。

+   当方案不允许相对 URL（例如，mailto）或*relative_url_string*明确指定位置（即使与*base_url_string*的位置相同）时，*u*的所有其他组件均为*relative_url_string*的组件。否则，*u*的位置为*base_url_string*的位置。

+   *u*的路径通过根据绝对和相对 URL 路径的标准语法连接*base_url_string*和*relative_url_string*的路径而获得。^(a) 例如：

```py
urlparse.urljoin(
  'http://host.com/some/path/here','../other/path')
*`# Result is: 'http://host.com/some/other/path'`*
```

|

| urlsplit | urlsplit(*url_string*, default_scheme='', allow_fragments=**True**) 分析 *url_string* 并返回一个五个字符串项的元组（实际上是 SplitResult 实例，可以将其视为元组或与命名属性一起使用）：*scheme*、*netloc*、*path*、*query* 和 *fragment**。当 allow_fragments 为 **False** 时，无论 *url_string* 是否具有片段，元组的最后一项始终为''。对应缺少部分的项也为''。例如：

```py
urlparse.urlsplit(
  'http://www.python.org:80/faq.cgi?src=file')
*`# Result is:`*
*`# 'http','www.python.org:80','/faq.cgi','src=file',''`*
```

|

| urlunsplit | urlunsplit(*url_tuple*) *url_tuple* 是任何具有确切五个项的可迭代对象，全部为字符串。从 urlsplit 调用的任何返回值都是 urlunsplit 的可接受参数。urlunsplit 返回具有给定组件和所需分隔符但无冗余分隔符的 URL 字符串（例如，当 fragment，*url_tuple* 的最后一项为''时，结果中没有 #）。例如：

```py
urlparse.urlunsplit((
  'http','www.python.org','/faq.cgi','src=fie',''))
*`# Result is: 'http://www.python.org/faq.cgi?src=fie'`*
```

urlunsplit(urlsplit(*x*)) 返回 URL 字符串 *x* 的规范形式，这不一定等于 *x*，因为 *x* 不一定是规范化的。例如：

```py
urlparse.urlsplit('http://a.com/path/a?'))
*`# Result is: 'http://a.com/path/a'`*
```

在这种情况下，规范化确保结果中不存在冗余分隔符，例如在 urlsplit 参数中的尾部 ?。

| ^(a) 根据 [RFC 1808](https://oreil.ly/T9v1p)。 |
| --- |

### urllib.request 模块

urllib.request 模块提供了访问标准互联网协议上的数据资源的函数，其中最常用的列在 Table 19-4 中（表中的示例假定您已导入了该模块）。

表 19-4\. urllib.request 模块的有用函数

| urlopen | urlopen(*url*, data=**None***,* timeout*,* context=**None**) 返回一个响应对象，其类型取决于 *url* 中的方案：

+   HTTP 和 HTTPS URL 返回一个 http.client.HTTPResponse 对象（具有修改的 msg 属性以包含与 reason 属性相同的数据；详情请参阅[在线文档](https://oreil.ly/gWFcH)）。您的代码可以像处理可迭代对象一样使用此对象，并作为上下文管理器在 **with** 语句中使用。

+   FTP、文件和数据 URL 返回一个 urllib.response.addinfourl 对象。

    *url* 是要打开的 URL 的字符串或 urllib.request.Request 对象。*data* 是一个可选的 bytes 对象、类文件对象或 bytes 的可迭代对象，用于编码发送到 URL 的额外数据，格式为 *application/x-www-form-urlencoded*。*timeout* 是一个可选参数，用于指定 URL 打开过程中阻塞操作的超时时间，单位为秒，仅适用于 HTTP、HTTPS 和 FTP URL。当给出 *context* 时，必须包含一个 ssl.SSLContext 对象，指定 SSL 选项；*context* 取代了已弃用的 *cafile*、*capath* 和 *cadefault* 参数。以下示例从 HTTPS URL 下载文件并提取为本地的 bytes 对象，unicode_db：

```py
unicode_url = ("`https://www.unicode.org/Public`"
               "/14.0.0/ucd/UnicodeData.txt")
`with` urllib.request.urlopen(unicode_url 
     )`as` url_response:
    unicode_db = url_response.read()
```

|

| u⁠r⁠l​r⁠e⁠t⁠r⁠i⁠e⁠v⁠e | urlretrieve(*url_string*, filename=**None**, report_hook=**None**, data=**None**) 一个兼容性函数，用于支持从 Python 2 遗留代码的迁移。*url_string* 给出要下载资源的 URL。filename 是一个可选的字符串，用于命名从 URL 检索的数据存储在本地文件中。report_hook 是一个可调用对象，支持在下载过程中报告进度，每次检索数据块时调用一次。data 类似于 urlopen 的 data 参数。在其最简单的形式下，urlretrieve 等效于：

```py
`def` urlretrieve(url, filename=`None`):
    `if` filename `is` `None`:
        filename = *`.``.``.``parse filename from url...`*
    `with` urllib.request.urlopen(url 
         )`as` url_response:
        `with` open(filename, "wb") `as` save_file:
            save_file.write(url_response.read())
        `return` filename, url_response.info()
```

由于这个函数是为了 Python 2 的兼容性而开发的，您可能仍然会在现有的代码库中看到它。新代码应该使用 urlopen。

要全面了解 urllib.request，请参阅[在线文档](https://oreil.ly/Vz9IV)和 Michael Foord 的 [HOWTO](https://oreil.ly/6Lrem)，其中包括根据 URL 下载文件的示例。在 “使用 BeautifulSoup 进行 HTML 解析的示例” 中有一个使用 urllib.request 的简短示例。

## 第三方 requests 包

第三方 [requests package](https://oreil.ly/cOiit)（非常好地[在线文档](https://oreil.ly/MiQ76)记录）是我们推荐您访问 HTTP URL 的方式。像其他第三方包一样，最好通过简单的 **pip install requests** 进行安装。在本节中，我们总结了如何在相对简单的情况下最佳地使用它。

请求模块本地仅支持 HTTP 和 HTTPS 传输协议；要访问其他协议的 URL，您需要安装其他第三方包（称为*协议适配器*），例如 [requests-ftp](https://oreil.ly/efT73) 用于 FTP URL，或作为丰富的 [requests-toolbelt](https://oreil.ly/_6nQe) 包的一部分提供的其他实用工具包。

requests 包的功能主要依赖于它提供的三个类：Request，模拟发送到服务器的 HTTP 请求；Response，模拟服务器对请求的 HTTP 响应；以及 Session，提供在一系列请求中的连续性，也称为*会话*。对于单个请求/响应交互的常见用例，您不需要连续性，因此通常可以忽略 Session。

### 发送请求

通常情况下，您不需要显式考虑 Request 类：而是调用实用函数 request，它内部准备并发送请求，并返回 Response 实例。request 有两个必需的位置参数，都是字符串：method，要使用的 HTTP 方法，和 url，要访问的 URL。然后，可能会跟随许多可选的命名参数（在下一节中，我们涵盖了 request 函数最常用的命名参数）。

为了进一步方便，requests 模块还提供了一些函数，它们的名称与 HTTP 方法 delete、get、head、options、patch、post 和 put 相同；每个函数都接受一个必需的位置参数 url，然后是与函数 request 相同的可选命名参数。

当您希望在多个请求中保持一致性时，调用 Session 创建一个实例 *s*，然后使用 *s* 的方法 request、get、post 等，它们就像直接由 requests 模块提供的同名函数一样（然而，*s* 的方法将 *s* 的设置与准备发送到给定 url 的每个请求的可选命名参数合并）。

### request 的可选命名参数

函数 request（就像函数 get、post 等以及类 Session 的实例 *s* 上的同名方法一样）接受许多可选的命名参数。如果您需要高级功能，比如控制代理、身份验证、特殊的重定向处理、流式传输、cookies 等，请参阅 requests 包的优秀[在线文档](https://oreil.ly/0rIwn)获取完整的参数集合。表 19-5 列出了最常用的命名参数。

表 19-5\. request 函数接受的命名参数列表

| data | 一个字典、一组键值对、一个字节串或者一个类似文件的对象，用作请求的主体 |
| --- | --- |
| files | 一个以名称为键、文件对象或*文件元组*为值的字典，与 POST 方法一起使用，用于指定多部分编码文件上传（我们将在下一节中讨论文件值的格式） |
| headers | 发送到请求中的 HTTP 头的字典 |
| json | Python 数据（通常是一个字典）编码为 JSON 作为请求主体 |
| params | 一个(*name*, *value*)项的字典，或者作为查询字符串发送的字节串与请求一起 |
| timeout | 秒数的浮点数，等待响应的最长时间，在引发异常之前 |

data、json 和 files 是用于指定请求主体的相互不兼容的方式；通常您应该最多使用其中一个，只用于使用主体的 HTTP 方法（即 PATCH、POST 和 PUT）。唯一的例外是您可以同时使用传递一个字典的 data 参数和一个 files 参数。这是非常常见的用法：在这种情况下，字典中的键值对和文件形成一个请求主体，作为一个*multipart/form-data*整体。⁴

### files 参数（以及其他指定请求主体的方法）

当您使用 json 或 data（传递一个字节串或者一个必须打开以供读取的类文件对象，通常在二进制模式下）指定请求主体时，生成的字节直接用作请求的主体。当您使用 data（传递一个字典或者一组键值对）指定时，主体以*表单*的形式构建，从键值对按*application/x-www-form-urlencoded*格式进行格式化，根据相关的[网络标准](https://oreil.ly/hHKp4)。

当你用 files 指定请求的主体时，该主体也作为一个表单构建，在这种情况下格式设置为*multipart/form-data*（在 PATCH、POST 或 PUT HTTP 请求中上传文件的唯一方式）。你上传的每个文件都被格式化为表单的一个单独部分；如果你还希望表单向服务器提供进一步的非文件参数，则除了 files 外，还需要传递一个数据参数，其值为字典（或键/值对序列）用于这些进一步的参数。这些参数被编码为多部分表单的补充部分。

为了灵活性，files 参数的值可以是一个字典（其条目被视为(*name*, *value*)对的序列），或者是(*name*, *value*)对的序列（结果请求体中的顺序保持不变）。

无论哪种方式，(*name*, *value*)对中的每个值可以是一个 str（或者更好地，⁵ 是一个字节或字节数组），直接用作上传文件的内容，或者是一个打开用于读取的类文件对象（此时，requests 调用 .read() 方法并使用结果作为上传文件的内容；我们强烈建议在这种情况下以二进制模式打开文件，以避免任何关于内容长度的歧义）。当满足这些条件之一时，requests 使用对的 *name* 部分（例如，字典中的键）作为文件的名称（除非它能够改进这一点，因为打开的文件对象能够显示其基础文件名），尝试猜测内容类型，并为文件的表单部分使用最小的标头。

或者，每个(*name*, *value*)对中的值可以是一个包含两到四个项目的元组，(*fn*, *fp*[, *ft*[, *fh*]])（使用方括号作为元语法来表示可选部分）。在这种情况下，*fn* 是文件的名称，*fp* 提供内容（与前一段中的方式相同），可选的 *ft* 提供内容类型（如果缺失，requests 将猜测它，如前一段中所示），可选的字典 *fh* 提供文件表单部分的额外标头。

### 如何解释 requests 的示例

在实际应用中，通常不需要考虑类 requests.Request 的内部实例 *r*，该类函数类似于 requests.post 在你的代表中构建、准备，然后发送。然而，为了确切了解 requests 的操作，以较低的抽象级别（构建、准备和检查 *r* —— 无需发送它！）是有益的。例如，在导入 requests 后，传递数据如下示例中所示：

```py
r = requests.Request('GET', 'http://www.example.com',
    data={'foo': 'bar'}, params={'fie': 'foo'})
p = r.prepare()
print(p.url)
print(p.headers)
print(p.body)
```

输出（为了可读性，将 *p*.headers 字典的输出拆分）：

```py
http://www.example.com/?fie=foo
{'Content-Length': '7',
 'Content-Type': 'application/x-www-form-urlencoded'}
foo=bar
```

类似地，在传递文件时：

```py
r = requests.Request('POST', 'http://www.example.com',
    data={'foo': 'bar'}, files={'fie': 'foo'})
p = r.prepare()
print(p.headers)
print(p.body)
```

这将输出（几行被拆分以提高可读性）：

```py
{'Content-Length': '228',
 'Content-Type': 'multipart/form-data; boundary=dfd600d8aa58496270'}
b'--dfd600d8aa58496270\r\nContent-Disposition: form-data;
="foo"\r\n\r\nbar\r\n--dfd600d8aa584962709b936134b1cfce\r\n
Content-Disposition: form-data; name="fie" filename="fie"\r\n\r\nfoo\r\n
--dfd600d8aa584962709b936134b1cfce--\r\n'
```

愉快的交互式探索！

### 响应类

requests 模块中你总是需要考虑的一个类是 Response：每个请求，一旦发送到服务器（通常是通过 get 等方法隐式完成），都会返回一个 requests.Response 实例 *r*。

你通常想要做的第一件事是检查*r*.status_code，一个告诉你请求进行情况的整数，在典型的“HTTPese”中：200 表示“一切正常”，404 表示“未找到”，等等。如果你希望对指示某种错误的状态码只获取异常，请调用*r*.raise_for_status；如果请求成功，它将不执行任何操作，但否则将引发 requests.exceptions.HTTPError 异常（其他异常，不对应任何特定的 HTTP 状态码，可能会被引发，而不需要任何明确的调用：例如任何网络问题的 ConnectionError，或者超时的 TimeoutError）。

接下来，你可能想要检查响应的 HTTP 头：为此，请使用*r*.headers，一个字典（具有特殊功能，其大小写不敏感的字符串键指示头名称，例如在[Wikipedia](https://oreil.ly/_nJRX)中列出的 HTTP 规范）。大多数头部可以安全地忽略，但有时你可能更愿意检查。例如，你可以通过*r*.headers.get('content-language')验证响应是否指定了其主体所写的自然语言，以提供不同的呈现选择，例如使用某种语言翻译服务使响应对用户更有用。

通常情况下，你不需要对重定向进行特定的状态或头检查：默认情况下，requests 会自动对除 HEAD 以外的所有方法进行重定向（你可以在请求中显式传递 allow_redirection 命名参数来更改该行为）。如果允许重定向，你可能需要检查*r*.history，一个沿途累积的 Response 实例列表，从最旧到最新，但不包括*r*本身（如果没有重定向，*r*.history 为空）。

大多数情况下，可能在检查状态和头之后，你想使用响应的主体。在简单情况下，只需将响应的主体作为字节字符串访问，*r*.content，或者通过调用*r*.json 将其解码为 JSON（一旦你检查到它是如何编码的，例如通过*r*.headers.get('content-type')）。

通常，你更倾向于将响应的主体作为（Unicode）文本访问，使用属性*r*.text。后者使用编解码器解码（从实际构成响应主体的八位字节），编解码器由请求根据 Content-Type 头和对主体本身的粗略检查认为最佳来确定。你可以通过属性*r*.encoding 检查使用的（或即将使用的）编解码器；其值将是与 codecs 模块注册的编解码器名称，详见“codecs 模块”。你甚至可以通过将*r*.encoding 分配为你选择的编解码器的名称来*覆盖*编解码器的选择。

我们不在本书中涵盖其他高级问题，例如流式传输；请参阅 requests 包的[在线文档](https://oreil.ly/4St1s)获取更多信息。

# 其他网络协议

许多，*许多* 其他网络协议在使用中—一些最好由 Python 标准库支持，但大多数你会在 [PyPI](https://oreil.ly/PGIim) 上找到更好和更新的第三方模块。

要像登录到另一台机器（或自己节点上的单独登录会话）一样连接，可以使用 [Secure Shell (SSH)](https://oreil.ly/HazNC) 协议，由第三方模块 [paramiko](http://www.paramiko.org) 或围绕它的更高抽象层包装器，第三方模块 [spur](https://oreil.ly/vdmrN) 支持。（你也可以，虽然可能存在某些安全风险，仍然使用经典的 [Telnet](https://oreil.ly/5fw-y)，由标准库模块 [telnetlib](https://oreil.ly/GYdGi) 支持。）

其他网络协议包括，但不限于：

+   [NNTP](https://oreil.ly/zCBov)，用于访问 Usenet 新闻服务器，由标准库模块 nntplib 支持。

+   [XML-RPC](https://oreil.ly/7vRm0)，用于基本的远程过程调用功能，由 [xmlrpc.client](https://oreil.ly/K3oDj) 支持。

+   [gRPC](http://www.grpc.io)，用于更现代的远程过程功能，由第三方模块 [grpcio](https://oreil.ly/KHQHs) 支持。

+   [NTP](http://www.ntp.org)，用于通过网络获取精确时间，由第三方模块 [ntplib](https://oreil.ly/R5SDp) 支持。

+   [SNMP](https://oreil.ly/nlhqH)，用于网络管理，由第三方模块 [pysnmp](https://oreil.ly/syh0_) 支持。

没有单一的书籍（甚至包括本书！）能够涵盖所有这些协议及其支持模块。相反，我们在这个问题上的最佳建议是战略性的：每当你决定通过某种网络协议使你的应用程序与其他系统交互时，不要急于实现自己的模块来支持该协议。而是搜索并询问，你很可能会找到优秀的现有 Python 模块（第三方或标准库），支持该协议。⁶

如果你发现这些模块中存在错误或缺失功能，请提交错误或功能请求（并且最好提供一个修复问题并满足你应用程序需求的补丁或拉取请求）。换句话说，成为开源社区的积极成员，而不仅仅是被动用户：你将会受到欢迎，解决你自己的问题，并在此过程中帮助许多其他人。“给予回馈”，因为你无法“回馈”给所有为你提供大部分工具的了不起的人们！

¹ IMAP4，参见 [RFC 1730](https://oreil.ly/fn5aH)；或 IMAP4rev1，参见 [RFC 2060](https://oreil.ly/C5N0w)。

² POP 协议的规范可在 [RFC 1939](https://oreil.ly/NLl6b) 中找到。

³ SMTP 协议的规范可在 [RFC 2821](https://oreil.ly/J9aCH) 中找到。

⁴ 根据 [RFC 2388](https://oreil.ly/7xsOe)。

⁵ 它能让你完全、明确地控制上传的八位字节。

⁶ 更重要的是，如果你认为需要发明一个全新的协议并在套接字之上实现它，再三考虑，并仔细搜索：很可能已经有大量现有的互联网协议完全符合你的需求！
