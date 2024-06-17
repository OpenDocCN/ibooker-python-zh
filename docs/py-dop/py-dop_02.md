# 第二章：自动化文件和文件系统

Python 最强大的功能之一是其处理文本和文件的能力。在 DevOps 的世界中，您不断地解析、搜索和更改文件中的文本，无论是搜索应用程序日志还是传播配置文件。文件是持久化数据、代码和配置状态的手段；它们是您查看日志发生的情况和控制配置发生的方式。使用 Python，您可以在代码中创建、读取和更改文件和文本，以便重复使用。自动化这些任务确实是现代 DevOps 的一个方面，它使其与传统系统管理有所区别。与手动跟随一套指令不同，您可以编写代码。这样可以减少错过步骤或按顺序执行它们的机会。如果您确信每次运行系统时都使用相同的步骤，那么您对过程的理解和信心将会更高。

# 读取和写入文件

使用`open`函数可以创建一个文件对象，该对象可以读取和写入文件。它接受两个参数，文件路径和模式（模式默认为读取）。您可以使用模式指示是否要读取或写入文件，以及文件是文本还是二进制数据等。您可以使用模式 *r* 打开文本文件以读取其内容。文件对象具有一个`read`方法，该方法将文件内容作为字符串返回：

```py
In [1]: file_path = 'bookofdreams.txt'
In [2]: open_file = open(file_path, 'r')
In [3]: text = open_file.read()
In [4]: len(text)
Out[4]: 476909

In [5]: text[56]
Out[5]: 's'

In [6]: open_file
Out[6]: <_io.TextIOWrapper name='bookofdreams.txt' mode='r' encoding='UTF-8'>

In [7]: open_file.close()
```

###### 注意

当您完成文件操作时关闭文件是一个良好的实践。Python 在文件超出范围时会关闭文件，但在此之前文件会消耗资源，并可能阻止其他进程打开它。

您还可以使用 `readlines` 方法读取文件。此方法读取文件并根据换行符拆分其内容。它返回一个字符串列表。每个字符串都是原始文本的一行：

```py
In [8]: open_file = open(file_path, 'r')
In [9]: text = open_file.readlines()
In [10]: len(text)
Out[10]: 8796

In [11]: text[100]
Out[11]: 'science, when it admits the possibility of occasional hallucinations\n'

In [12]: open_file.close()
```

使用 `with` 语句打开文件的一个便捷方法。在这种情况下，您不需要显式关闭文件。Python 在缩进块结束时关闭文件并释放文件资源：

```py
In [13]: with open(file_path, 'r') as open_file:
    ...:     text = open_file.readlines()
    ...:

In [14]: text[101]
Out[14]: 'in the sane and healthy, also admits, of course, the existence of\n'

In [15]: open_file.closed
Out[15]: True
```

不同的操作系统使用不同的转义字符表示换行符。Unix 系统使用 `\n`，而 Windows 系统使用 `\r\n`。Python 在将文件作为文本打开时会将这些转换为 `\n`。如果您以文本方式打开二进制文件，例如 *.jpeg* 图像，则可能会通过此转换损坏数据。但是，您可以通过在模式后附加 *b* 来读取二进制文件：

```py
In [15]: file_path = 'bookofdreamsghos00lang.pdf'
In [16]: with open(file_path, 'rb') as open_file:
    ...:     btext = open_file.read()
    ...:

In [17]: btext[0]
Out[17]: 37

In [18]: btext[:25]
Out[18]: b'%PDF-1.5\n%\xec\xf5\xf2\xe1\xe4\xef\xe3\xf5\xed\xe5\xee\xf4\n18'
```

添加此项将不对行结束进行任何转换。

要写入文件，使用写入模式，表示为参数`w`。工具`direnv`用于自动设置一些开发环境。您可以在名为*.envrc*的文件中定义环境变量和应用程序运行时；`direnv`在进入带有该文件的目录时使用它来设置这些内容。您可以在 Python 中使用带有写入标志的`open`来将环境变量`STAGE`设置为`PROD`，`TABLE_ID`设置为`token-storage-1234`：

```py
In [19]: text = '''export STAGE=PROD
 ...: export TABLE_ID=token-storage-1234'''

In [20]: with open('.envrc', 'w') as opened_file:
    ...:     opened_file.write(text)
    ...:

In [21]: !cat .envrc
export STAGE=PROD
export TABLE_ID=token-storage-1234
```

###### 警告

警告，如果文件已经存在，`pathlib`的`write`方法将覆盖该文件。

`open`函数如果文件不存在将创建文件，并且如果存在则覆盖它。如果您想保留现有内容并仅追加文件，请使用追加标志`a`。此标志将新文本追加到文件末尾，同时保留原始内容。如果您正在写入非文本内容，例如*.jpeg*文件的内容，如果使用`w`或`a`标志，可能会导致其损坏。当 Python 写入文本数据时，它会将行结束符转换为特定于平台的结束符。要写入二进制数据，您可以安全地使用`wb`或`ab`。

第三章深入介绍了`pathlib`。两个有用的功能是便捷函数用于读取和写入文件。`pathlib`在幕后处理文件对象。以下示例允许您从文件中读取文本：

```py
In [35]: import pathlib

In [36]: path = pathlib.Path(
           "/Users/kbehrman/projects/autoscaler/check_pending.py")

In [37]: path.read_text()
```

要读取二进制数据，请使用`path.read_bytes`方法。

当您想要覆盖文件或写入新文件时，有写入文本和写入二进制数据的方法：

```py
In [38]: path = pathlib.Path("/Users/kbehrman/sp.config")

In [39]: path.write_text("LOG:DEBUG")
Out[39]: 9

In [40]: path = pathlib.Path("/Users/kbehrman/sp")
Out[41]: 8
```

使用文件对象的`read`和`write`函数进行读写通常对于非结构化文本已经足够，但是如果你要处理更复杂的数据怎么办？JavaScript 对象表示法（JSON）格式被广泛用于在现代 Web 服务中存储简单的结构化数据。它使用两种数据结构：类似于 Python `dict`的键-值对映射和类似于 Python `list`的项目列表。它定义了数字、字符串、*布尔*（保存 true/false 值）和*nulls*（空值）的数据类型。AWS 身份和访问管理（IAM）Web 服务允许您控制对 AWS 资源的访问。它使用 JSON 文件来定义访问策略，例如以下示例文件：

```py
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "service-prefix:action-name",
        "Resource": "*",
        "Condition": {
            "DateGreaterThan": {"aws:CurrentTime": "2017-07-01T00:00:00Z"},
            "DateLessThan": {"aws:CurrentTime": "2017-12-31T23:59:59Z"}
        }
    }
}
```

您可以使用标准文件对象的`read`或`readlines`方法从此类文件中获取数据：

```py
In [8]: with open('service-policy.json', 'r') as opened_file:
   ...:     policy = opened_file.readlines()
   ...:
   ...:
```

结果可能不能立即使用，因为它将是一个字符串或字符串列表，具体取决于您选择的读取方法：

```py
In [9]: print(policy)
['{\n',
 '    "Version": "2012-10-17",
\n',
 '    "Statement": {\n',
 '        "Effect": "Allow",
\n',
 '        "Action": "service-prefix:action-name",
\n',
 '        "Resource": "*",
\n',
 '        "Condition": {\n',
 '            "DateGreaterThan": {"aws:CurrentTime": "2017-07-01T00:00:00Z"},
\n',
 '            "DateLessThan": {"aws:CurrentTime": "2017-12-31T23:59:59Z"}\n',
 '        }\n',
 '    }\n',
 '}\n']
```

然后，您需要将此字符串（或字符串）解析为与原始数据结构和类型匹配的数据。这可能是一项相当大的工作。更好的方法是使用`json`模块：

```py
In [10]: import json

In [11]: with open('service-policy.json', 'r') as opened_file:
    ...:     policy = json.load(opened_file)
    ...:
    ...:
    ...:
```

此模块会为您解析 JSON 格式，将数据返回为适当的 Python 数据结构：

```py
In [13]: from pprint import pprint

In [14]: pprint(policy)
{'Statement': {'Action': 'service-prefix:action-name',
               'Condition': {'DateGreaterThan':
                                  {'aws:CurrentTime': '2017-07-01T00:00:00Z'},
                             'DateLessThan':
                                  {'aws:CurrentTime': '2017-12-31T23:59:59Z'}},
               'Effect': 'Allow',
               'Resource': '*'},
 'Version': '2012-10-17'}
```

###### 注意

`pprint`模块会自动格式化 Python 对象以便打印。其输出通常更易读，是查看嵌套数据结构的便捷方式。

现在你可以使用原始文件结构中的数据。例如，这里是如何将该策略控制访问的资源更改为 `S3`：

```py
In [15]: policy['Statement']['Resource'] = 'S3'

In [16]: pprint(policy)
{'Statement': {'Action': 'service-prefix:action-name',
               'Condition': {'DateGreaterThan':
                                {'aws:CurrentTime': '2017-07-01T00:00:00Z'},
                             'DateLessThan':
                                {'aws:CurrentTime': '2017-12-31T23:59:59Z'}},
               'Effect': 'Allow',
               'Resource': 'S3'},
 'Version': '2012-10-17'}
```

你可以通过使用 `json.dump` 方法将 Python 字典写入 JSON 文件。这就是你更新刚修改的策略文件的方式：

```py
In [17]: with open('service-policy.json', 'w') as opened_file:
    ...:     policy = json.dump(policy, opened_file)
    ...:
    ...:
    ...:
```

另一种常用于配置文件的语言是 *YAML*（“YAML Ain’t Markup Language”）。它是 JSON 的超集，但有更紧凑的格式，使用与 Python 类似的空白。

Ansible 是一个用于自动化软件配置、管理和部署的工具。Ansible 使用称为 *playbooks* 的文件来定义你想要自动化的操作。这些 playbooks 使用 YAML 格式：

```py
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  ...
```

Python 中最常用的解析 YAML 文件的库是 Py   在 Python 中解析 YAML 文件最常用的库是 PyYAML。它不在 Python 标准库中，但你可以使用 `pip` 安装它：

```py
$ pip install PyYAML
```

安装完成后，你可以像处理 JSON 数据那样使用 PyYAML 导入和导出 YAML 数据：

```py
In [18]: import yaml

In [19]: with open('verify-apache.yml', 'r') as opened_file:
    ...:     verify_apache = yaml.safe_load(opened_file)
    ...:
```

数据加载为熟悉的 Python 数据结构（一个包含 `dict` 的 `list`）：

```py
In [20]: pprint(verify_apache)
[{'handlers': [{'name': 'restart apache',
                'service': {'name': 'httpd', 'state': 'restarted'}}],
  'hosts': 'webservers',
  'remote_user': 'root',
  'tasks': [{'name': 'ensure apache is at the latest version',
             'yum': {'name': 'httpd', 'state': 'latest'}},
            {'name': 'write the apache config file',
             'notify': ['restart apache'],
             'template': {'dest': '/etc/httpd.conf', 'src': '/srv/httpd.j2'}},
            {'name': 'ensure apache is running',
             'service': {'name': 'httpd', 'state': 'started'}}],
  'vars': {'http_port': 80, 'max_clients': 200}}]
```

你还可以将 Python 数据保存到 YAML 格式的文件中：

```py
In [22]: with open('verify-apache.yml', 'w') as opened_file:
    ...:     yaml.dump(verify_apache, opened_file)
    ...:
    ...:
    ...:
```

另一种广泛用于表示结构化数据的语言是可扩展标记语言（XML）。它由带标签的元素的层次文档组成。历史上，许多 web 系统使用 XML 传输数据。其中一种用途是用于实时简单聚合（RSS）订阅源。RSS 订阅源用于跟踪和通知用户网站更新，并已被用于跟踪来自各种来源的文章的发布。RSS 订阅源使用 XML 格式的页面。Python 提供了 `xml` 库来处理 XML 文档。它将 XML 文档的层次结构映射到类似树状的数据结构。树的节点是元素，使用父子关系来建模层次结构。最顶层的父节点称为根元素。要解析一个 RSS XML 文档并获取其根节点：

```py
In [1]: import xml.etree.ElementTree as ET
In [2]: tree = ET.parse('http_feeds.feedburner.com_oreilly_radar_atom.xml')

In [3]: root = tree.getroot()

In [4]: root
Out[4]: <Element '{http://www.w3.org/2005/Atom}feed' at 0x11292c958>
```

你可以通过迭代子节点来遍历树：

```py
In [5]: for child in root:
   ...:     print(child.tag, child.attrib)
   ...:
{http://www.w3.org/2005/Atom}title {}
{http://www.w3.org/2005/Atom}id {}
{http://www.w3.org/2005/Atom}updated {}
{http://www.w3.org/2005/Atom}subtitle {}
{http://www.w3.org/2005/Atom}link {'href': 'https://www.oreilly.com'}
{http://www.w3.org/2005/Atom}link {'rel': 'hub',
                                   'href': 'http://pubsubhubbub.appspot.com/'}
{http://www.w3.org/2003/01/geo/wgs84_pos#}long {}
{http://rssnamespace.org/feedburner/ext/1.0}emailServiceId {}
...
```

XML 允许进行 *命名空间*（使用标签分组数据）。XML 在标签前加上用括号括起来的命名空间。如果你知道层次结构的结构，可以使用路径搜索元素。你可以提供一个定义命名空间的字典，方便使用：

```py
In [108]: ns = {'default':'http://www.w3.org/2005/Atom'}
In [106]: authors = root.findall("default:entry/default:author/default:name", ns)

In [107]: for author in authors:
     ...:     print(author.text)
     ...:
Nat Torkington
VM Brasseur
Adam Jacob
Roger Magoulas
Pete Skomoroch
Adrian Cockcroft
Ben Lorica
Nat Torkington
Alison McCauley
Tiffani Bell
Arun Gupta
```

你可能会遇到需要处理逗号分隔值（CSV）格式的数据。这个格式常用于电子表格数据。你可以使用 Python 的 `csv` 模块轻松读取这些数据：

```py
In [16]: import csv
In [17]: file_path = '/Users/kbehrman/Downloads/registered_user_count_ytd.csv'

In [18]: with open(file_path, newline='') as csv_file:
    ...:     off_reader = csv.reader(csv_file, delimiter=',')
    ...:     for _ in range(5):
    ...:         print(next(off_reader))
    ...:
['Date', 'PreviousUserCount', 'UserCountTotal', 'UserCountDay']
['2014-01-02', '61', '5336', '5275']
['2014-01-03', '42', '5378', '5336']
['2014-01-04', '26', '5404', '5378']
['2014-01-05', '65', '5469', '5404']
```

`csv` 读取器对象逐行遍历 *.csv* 文件，让你可以逐行处理数据。以这种方式处理文件对于不希望一次性读取到内存的大 *.csv* 文件特别有用。当然，如果你需要进行跨列的多行计算且文件不太大，你应该一次性加载所有数据。

Pandas 包是数据科学界的主要工具。它包括一个数据结构，`pandas.DataFrame`，它的作用类似于非常强大的电子表格。如果您有类似表格的数据，想要进行统计分析或者按行和列进行操作，DataFrame 是您的工具。它是一个第三方库，因此您需要使用`pip`安装它。您可以使用各种方法将数据加载到 DataFrame 中；其中最常见的方法之一是从*.csv*文件中加载：

```py
In [54]: import pandas as pd

In [55]: df = pd.read_csv('sample-data.csv')

In [56]: type(df)
Out[56]: pandas.core.frame.DataFrame
```

您可以使用`head`方法查看 DataFrame 的前几行：

```py
In [57]: df.head(3)
Out[57]:
   Attributes     open       high        low      close     volume
0     Symbols        F          F          F          F          F
1        date      NaN        NaN        NaN        NaN        NaN
2  2018-01-02  11.3007    11.4271    11.2827    11.4271   20773320
```

您可以使用`describe`方法获得统计洞见：

```py
In [58]: df.describe()
Out[58]:
        Attributes    open      high    low     close     volume
count          357     356       356    356       356        356
unique         357     290       288    297       288        356
top     2018-10-18  10.402    8.3363   10.2    9.8111   36298597
freq             1       5         4      3         4          1
```

或者，您可以使用方括号中的名称查看单列数据：

```py
In [59]: df['close']
Out[59]:
0            F
1          NaN
2      11.4271
3      11.5174
4      11.7159
        ...
352       9.83
353       9.78
354       9.71
355       9.74
356       9.52
Name: close, Length: 357, dtype: object
```

Pandas 有更多用于分析和操作类似表格的数据的方法，也有许多关于其使用的书籍。如果您需要进行数据分析，那么这是您应该了解的工具。

# 使用正则表达式搜索文本

Apache HTTP 服务器是一个广泛用于提供 Web 内容的开源 Web 服务器。Web 服务器可以配置为以不同的格式保存日志文件。一个广泛使用的格式是通用日志格式（CLF）。各种日志分析工具都能理解这种格式。以下是此格式的布局：

```py
<IP Address> <Client Id> <User Id> <Time> <Request> <Status> <Size>
```

下面是这种格式的日志的示例行：

```py
127.0.0.1 - swills [13/Nov/2019:14:43:30 -0800] "GET /assets/234 HTTP/1.0" 200 2326
```

第一章向您介绍了正则表达式和 Python 的`re`模块，所以让我们使用它从常见日志格式中提取信息。构建正则表达式的一个技巧是分段进行。这样做可以使您让每个子表达式都能正常工作，而不必调试整个表达式。您可以使用命名组创建正则表达式来从一行中提取 IP 地址：

```py
In [1]: line = '127.0.0.1 - rj [13/Nov/2019:14:43:30] "GET HTTP/1.0" 200'

In [2]: re.search(r'(?P<IP>\d+\.\d+\.\d+\.\d+)', line)
Out[2]: <re.Match object; span=(0, 9), match='127.0.0.1'>

In [3]: m = re.search(r'(?P<IP>\d+\.\d+\.\d+\.\d+)', line)

In [4]: m.group('IP')
Out[4]: '127.0.0.1'
```

您还可以创建一个正则表达式来获取时间：

```py
In [5]: r = r'\[(?P<Time>\d\d/\w{3}/\d{4}:\d{2}:\d{2}:\d{2})\]'

In [6]: m = re.search(r, line)

In [7]: m.group('Time')
Out[7]: '13/Nov/2019:14:43:30'
```

您可以像这样获取多个元素：IP、用户、时间和请求：

```py
In [8]:  r = r'(?P<IP>\d+\.\d+\.\d+\.\d+)'

In [9]: r += r' - (?P<User>\w+) '

In [10]: r += r'\[(?P<Time>\d\d/\w{3}/\d{4}:\d{2}:\d{2}:\d{2})\]'

In [11]: r += r' (?P<Request>".+")'

In [12]:  m = re.search(r, line)

In [13]: m.group('IP')
Out[13]: '127.0.0.1'

In [14]: m.group('User')
Out[14]: 'rj'

In [15]: m.group('Time')
Out[15]: '13/Nov/2019:14:43:30'

In [16]: m.group('Request')
Out[16]: '"GET HTTP/1.0"'
```

解析日志的单行很有趣，但不是非常有用。但是，您可以使用这个正则表达式作为从整个日志中提取信息的基础。假设您想要提取 2019 年 11 月 8 日发生的所有`GET`请求的所有 IP 地址。使用前面的表达式，根据您请求的具体情况进行修改：

```py
In [62]: r = r'(?P<IP>\d+\.\d+\.\d+\.\d+)'
In [63]: r += r'- (?P<User>\w+)'
In [64]: r += r'\[(?P<Time>08/Nov/\d{4}:\d{2}:\d{2}:\d{2} [-+]\d{4})\]'
In [65]: r += r' (?P<Request>"GET .+")'
```

使用`finditer`方法处理日志，打印匹配行的 IP 地址：

```py
In [66]: matched = re.finditer(r, access_log)

In [67]: for m in matched:
    ...:     print(m.group('IP'))
    ...:
127.0.0.1
342.3.2.33
```

有很多可以用正则表达式和各种文本做的事情。如果它们不使你畏惧，你会发现它们是处理文本最强大的工具之一。

# 处理大文件

有时需要处理非常大的文件。如果文件包含可以一次处理一行的数据，则使用 Python 很容易。与其像您到目前为止所做的那样将整个文件加载到内存中，您可以一次读取一行，处理该行，然后继续下一个。Python 的垃圾收集器会自动从内存中删除这些行，释放内存。

###### 注意

Python 会自动分配和释放内存。垃圾收集是一种方法。Python 的垃圾收集器可以使用`gc`包进行控制，尽管这很少需要。

操作系统使用替代行结束符时，读取在不同操作系统上创建的文件可能会很麻烦。Windows 创建的文件除了`\n`外还有`\r`字符。这些字符在 Linux 系统中显示为文本的一部分。如果您有一个大文件，并且希望校正行结束符以适应当前操作系统，您可以打开文件，一次读取一行，并将其保存到新文件中。Python 会为您处理行结束符的转换：

```py
In [23]: with open('big-data.txt', 'r') as source_file:
    ...:     with open('big-data-corrected.txt', 'w') as target_file:
    ...:         for line in source_file:
    ...:             target_file.write(line)
    ...:
```

注意，您可以嵌套`with`语句以同时打开两个文件，并逐行处理源文件对象。如果需要一次处理多个文件的单行，可以定义生成器函数来处理：

```py
In [46]: def line_reader(file_path):
    ...:     with open(file_path, 'r') as source_file:
    ...:         for line in source_file:
    ...:             yield line
    ...:

In [47]: reader = line_reader('big-data.txt')

In [48]: with open('big-data-corrected.txt', 'w') as target_file:
    ...:     for line in reader:
    ...:         target_file.write(line)
    ...:
```

如果您不使用行结束符来分隔数据，如大型二进制文件的情况，则可以按块读取数据。您将每个块中读取的字节数传递给文件对象的`read`方法。当没有剩余内容可读时，表达式将返回空字符串：

```py
In [27]: with open('bb141548a754113e.jpg', 'rb') as source_file:
    ...:     while True:
    ...:         chunk = source_file.read(1024)
    ...:         if chunk:
    ...:             process_data(chunk)
    ...:         else:
    ...:             break
    ...:
```

# 加密文本

您有许多时候需要加密文本以确保安全性。除了 Python 的内置包`hashlib`外，还有一个广泛使用的第三方包称为`cryptography`。让我们来看看它们。

## 使用 Hashlib 进行哈希处理

为了安全起见，用户密码必须加密存储。处理这一常见方法是使用单向函数将密码加密成位串，这样很难进行逆向工程。执行此操作的函数称为*哈希函数*。除了遮蔽密码外，哈希函数还确保在传输过程中未更改的文档。您对文档运行哈希函数并将结果与文档一起发送。接收者可以通过对文档进行哈希验证值是否相同。`hashlib`包括用于执行此操作的安全算法，包括*SHA1*、*SHA224*、*SHA384*、*SHA512*和 RSA 的*MD5*。这是使用 MD5 算法对密码进行哈希的方法：

```py
In [62]: import hashlib

In [63]: secret = "This is the password or document text"

In [64]: bsecret = secret.encode()

In [65]: m = hashlib.md5()

In [66]: m.update(bsecret)

In [67]: m.digest()
Out[67]: b' \xf5\x06\xe6\xfc\x1c\xbe\x86\xddj\x96C\x10\x0f5E'
```

注意，如果您的密码或文档是字符串，则需要使用`encode`方法将其转换为二进制字符串。

## 使用密码学进行加密

`cryptography`库是 Python 中处理加密问题的热门选择。它是一个第三方包，所以你必须使用`pip`安装它。*对称密钥加密*是一组基于共享密钥的加密算法。这些算法包括高级加密标准（AES）、Blowfish、数据加密标准（DES）、Serpent 和 Twofish。共享密钥类似于用于加密和解密文本的密码。与稍后我们将讨论的*非对称密钥加密*相比，创建者和读者都需要共享密钥这一事实是其缺点。然而，对称密钥加密更快更简单，因此适合加密大文件。Fernet 是流行 AES 算法的实现。你首先需要生成一个密钥：

```py
In [1]: from cryptography.fernet import Fernet

In [2]: key = Fernet.generate_key()

In [3]: key
Out[3]: b'q-fEOs2JIRINDR8toMG7zhQvVhvf5BRPx3mj5Atk5B8='
```

你需要安全存储这个密钥，因为你需要它来解密。请记住，任何有权访问它的人也能解密你的文件。如果选择将密钥保存到文件中，请使用二进制数据类型。下一步是使用`Fernet`对象加密数据：

```py
In [4]: f = Fernet(key)

In [5]: message = b"Secrets go here"

In [6]: encrypted = f.encrypt(message)

In [7]: encrypted
Out[7]: b'gAAAAABdPyg4 ... plhkpVkC8ezOHaOLIA=='
```

可以使用使用相同密钥创建的`Fernet`对象解密数据：

```py
In [1]: f = Fernet(key)

In [2]: f.decrypt(encrypted)
Out[2]: b'Secrets go here'
```

非对称加密使用一对密钥，一个是公钥，一个是私钥。公钥设计为广泛共享，而单个用户持有私钥。只有使用私钥才能解密使用你的公钥加密的消息。这种加密方式被广泛用于在本地网络和互联网上保密传递信息。一个非常流行的非对称密钥算法是 Rivest-Shamir-Adleman（RSA），它被广泛用于网络通信。加密库提供了创建公钥/私钥对的能力：

```py
In [1]: from cryptography.hazmat.backends import default_backend

In [2]: from cryptography.hazmat.primitives.asymmetric import rsa

In [3]: private_key = rsa.generate_private_key(public_exponent=65537,
                                               key_size=4096,
                                               backend=default_backend())

In [4]: private_key
Out[4]: <cryptography.hazmat.backends.openssl.rsa._RSAPrivateKey at 0x10d377c18>

In [5]: public_key = private_key.public_key

In [6]: public_key = private_key.public_key()

In [7]: public_key
Out[7]: <cryptography.hazmat.backends.openssl.rsa._RSAPublicKey at 0x10da642b0>
```

然后可以使用公钥进行加密：

```py
In [8]: message = b"More secrets go here"

In [9]: from cryptography.hazmat.primitives.asymmetric import padding
In [11]: from cryptography.hazmat.primitives import hashes

In [12]: encrypted = public_key.encrypt(message,
    ...:    padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()),
    ...:    algorithm=hashes.SHA256(),
    ...:    label=None))
```

可以使用私钥解密消息：

```py
In [13]: decrypted = private_key.decrypt(encrypted,
    ...:    padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()),
    ...:    algorithm=hashes.SHA256(),
    ...:    label=None))

In [14]: decrypted
Out[14]: b'More secrets go here'
```

# os 模块

`os`模块是 Python 中最常用的模块之一。这个模块处理许多低级操作系统调用，并尝试在多个操作系统之间提供一致的接口，如果你的应用程序可能在 Windows 和 Unix-based 系统上运行，这点非常重要。它确实提供了一些特定于操作系统的特性（在 Windows 上为`os.O_TEXT`，在 Linux 上为`os.O_CLOEXEC`），这些特性跨平台不可用。只有在确信你的应用程序不需要在操作系统之间可移植时才使用这些特性。示例 2-1 展示了`os`模块中一些最有用的附加方法。

##### 示例 2-1. 更多 os 方法

```py
In [1]: os.listdir('.') ![1](img/1.png)
Out[1]: ['__init__.py', 'os_path_example.py']

In [2]: os.rename('_crud_handler', 'crud_handler') ![2](img/2.png)

In [3]: os.chmod('my_script.py', 0o777) ![3](img/3.png)

In [4]: os.mkdir('/tmp/holding') ![4](img/4.png)

In [5]: os.makedirs('/Users/kbehrman/tmp/scripts/devops') ![5](img/5.png)

In [6]: os.remove('my_script.py') ![6](img/6.png)

In [7]: os.rmdir('/tmp/holding') ![7](img/7.png)

In [8]: os.removedirs('/Users/kbehrman/tmp/scripts/devops') ![8](img/8.png)

In [9]: os.stat('crud_handler') ![9](img/9.png)
Out[9]: os.stat_result(st_mode=16877,
                       st_ino=4359290300,
                       st_dev=16777220,
                       st_nlink=18,
                       st_uid=501,
                       st_gid=20,
                       st_size=576,
                       st_atime=1544115987,
                       st_mtime=1541955837,
                       st_ctime=1567266289)
```

![1](img/#co_automating_files_and_the_filesystem_CO1-1)

列出目录的内容。

![2](img/#co_automating_files_and_the_filesystem_CO1-2)

重命名文件或目录。

![3](img/#co_automating_files_and_the_filesystem_CO1-3)

更改文件或目录的权限设置。

![4](img/#co_automating_files_and_the_filesystem_CO1-4)

创建一个目录。

![5](img/#co_automating_files_and_the_filesystem_CO1-5)

递归创建目录路径。

![6](img/#co_automating_files_and_the_filesystem_CO1-6)

删除文件。

![7](img/#co_automating_files_and_the_filesystem_CO1-7)

删除单个目录。

![8](img/#co_automating_files_and_the_filesystem_CO1-8)

删除一棵目录树，从叶子目录开始向上遍历树。该操作会在遇到第一个非空目录时停止。

![9](img/#co_automating_files_and_the_filesystem_CO1-9)

获取文件或目录的统计信息。这些统计信息包括 `st_mode`，文件类型和权限，以及 `st_atime`，项目上次访问的时间。

# 使用 `os.path` 管理文件和目录

在 Python 中，您可以使用字符串（无论是二进制还是其他）来表示路径。`os.path` 模块提供了大量用于创建和操作路径的方法。如前所述，`os` 模块尝试提供跨平台行为，`os.path` 子模块也不例外。该模块根据当前操作系统解释路径，Unix-like 系统中使用斜杠分隔目录，Windows 中使用反斜杠。您的程序可以动态构造适合当前系统的路径。轻松拆分和连接路径可能是 `os.path` 最常用的功能之一。用于拆分路径的三个方法是 `split`，`basename` 和 `dirname`：

```py
In [1]: import os

In [2]: cur_dir = os.getcwd() ![1](img/1.png)

In [3]: cur_dir
Out[3]: '/Users/kbehrman/Google-Drive/projects/python-devops/samples/chapter4'

In [4]: os.path.split(cur_dir) ![2](img/2.png)
Out[4]: ('/Users/kbehrman/Google-Drive/projects/python-devops/samples',
         'chapter4')

In [5]: os.path.dirname(cur_dir) ![3](img/3.png)
Out[5]: '/Users/kbehrman/Google-Drive/projects/python-devops/samples'

In [6]: os.path.basename(cur_dir) ![4](img/4.png)
Out[6]: 'chapter4'
```

![1](img/#co_automating_files_and_the_filesystem_CO2-1)

获取当前工作目录。

![2](img/#co_automating_files_and_the_filesystem_CO2-2)

`os.path.split` 将路径的叶级与父路径分开。

![3](img/#co_automating_files_and_the_filesystem_CO2-3)

`os.path.dirname` 返回父路径。

![4](img/#co_automating_files_and_the_filesystem_CO2-4)

`os.path.basename` 返回叶子名称。

您可以轻松使用 `os.path.dirname` 来向上遍历目录树：

```py
In [7]: while os.path.basename(cur_dir):
   ...:     cur_dir = os.path.dirname(cur_dir)
   ...:     print(cur_dir)
   ...:
/Users/kbehrman/projects/python-devops/samples
/Users/kbehrman/projects/python-devops
/Users/kbehrman/projects
/Users/kbehrman
/Users
/
```

在运行时使用文件配置应用程序是常见的做法；Unix-like 系统中的文件通常按照以*rc*结尾的约定命名。Vim 的*.vimrc*文件和 Bash shell 的*.bashrc*是两个常见的示例。您可以将这些文件存储在不同的位置。通常，程序会定义一个层次结构来检查这些位置。例如，您的工具可能首先查找一个定义了要使用的*rc*文件的环境变量，如果没有找到，则检查工作目录，然后是用户主目录。在示例 2-2 中，我们尝试在这些位置中找到一个*rc*文件。我们使用 Python 自动设置的`*file*`变量，该变量是相对于当前工作目录的路径，而不是绝对路径或完整路径。Python 不会自动展开路径，这在类 Unix 系统中很常见，因此在使用它构建要检查的*rc*文件路径之前，我们必须将此路径展开。同样，Python 不会自动展开路径中的环境变量，因此我们必须显式展开这些变量。

##### 示例 2-2\. find_rc 方法

```py
def find_rc(rc_name=".examplerc"):

    # Check for Env variable
    var_name = "EXAMPLERC_DIR"
    if var_name in os.environ: ![1](img/1.png)
        var_path = os.path.join(f"${var_name}", rc_name) ![2](img/2.png)
        config_path = os.path.expandvars(var_path) ![3](img/3.png)
        print(f"Checking {config_path}")
        if os.path.exists(config_path): ![4](img/4.png)
            return config_path

    # Check the current working directory
    config_path = os.path.join(os.getcwd(), rc_name)  ![5](img/5.png)
    print(f"Checking {config_path}")
    if os.path.exists(config_path):
        return config_path

    # Check user home directory
    home_dir = os.path.expanduser("~/")  ![6](img/6.png)
    config_path = os.path.join(home_dir, rc_name)
    print(f"Checking {config_path}")
    if os.path.exists(config_path):
        return config_path

    # Check Directory of This File
    file_path = os.path.abspath(__file__) ![7](img/7.png)
    parent_path = os.path.dirname(file_path) ![8](img/8.png)
    config_path = os.path.join(parent_path, rc_name)
    print(f"Checking {config_path}")
    if os.path.exists(config_path):
        return config_path

    print(f"File {rc_name} has not been found")
```

![1](img/#co_automating_files_and_the_filesystem_CO3-1)

检查当前环境中是否存在环境变量。

![2](img/#co_automating_files_and_the_filesystem_CO3-2)

使用`join`结合环境变量名构建路径。这将看起来像是`$EXAMPLERC_DIR/.examplerc`。

![3](img/#co_automating_files_and_the_filesystem_CO3-3)

展开环境变量以将其值插入路径中。

![4](img/#co_automating_files_and_the_filesystem_CO3-4)

检查文件是否存在。

![5](img/#co_automating_files_and_the_filesystem_CO3-5)

使用当前工作目录构建路径。

![6](img/#co_automating_files_and_the_filesystem_CO3-6)

使用`expanduser`函数获取用户主目录的路径。

![7](img/#co_automating_files_and_the_filesystem_CO3-7)

将存储在`*file*`中的相对路径扩展为绝对路径。

![8](img/#co_automating_files_and_the_filesystem_CO3-8)

使用`dirname`获取当前文件所在目录的路径。

`path`子模块还提供了查询路径统计信息的方法。您可以确定路径是文件、目录、链接还是挂载点。您可以获取大小、最后访问时间或修改时间等统计信息。在示例 2-3 中，我们使用`path`遍历目录树，并报告其中所有文件的大小和最后访问时间。

##### 示例 2-3\. os_path_walk.py

```py
#!/usr/bin/env python

import fire
import os

def walk_path(parent_path):
    print(f"Checking: {parent_path}")
    childs = os.listdir(parent_path) ![1](img/1.png)

    for child in childs:
        child_path = os.path.join(parent_path, child) ![2](img/2.png)
        if os.path.isfile(child_path): ![3](img/3.png)
            last_access = os.path.getatime(child_path) ![4](img/4.png)
            size = os.path.getsize(child_path) ![5](img/5.png)
            print(f"File: {child_path}")
            print(f"\tlast accessed: {last_access}")
            print(f"\tsize: {size}")
        elif os.path.isdir(child_path): ![6](img/6.png)
            walk_path(child_path) ![7](img/7.png)

if __name__ == '__main__':
    fire.Fire()
```

![1](img/#co_automating_files_and_the_filesystem_CO4-1)

`os.listdir`返回目录的内容。

![2](img/#co_automating_files_and_the_filesystem_CO4-2)

构建父目录中项目的完整路径。

![3](img/#co_automating_files_and_the_filesystem_CO4-3)

检查路径是否表示一个文件。

![4](img/#co_automating_files_and_the_filesystem_CO4-4)

获取文件上次访问的时间。

![5](img/#co_automating_files_and_the_filesystem_CO4-5)

获取文件的大小。

![6](img/#co_automating_files_and_the_filesystem_CO4-6)

检查路径是否表示目录。

![7](img/#co_automating_files_and_the_filesystem_CO4-7)

从此目录向下检查树。

您可以使用类似于此的脚本来识别大文件或未访问的文件，然后报告、移动或删除它们。

# 使用 os.walk 遍历目录树

`os` 模块提供了一个便捷的函数用于遍历目录树，称为 `os.walk`。该函数返回一个生成器，依次返回每个迭代的元组。该元组包括当前路径、目录列表和文件列表。在 示例 2-4 中，我们重新编写了来自 示例 2-3 的 `walk_path` 函数，以使用 `os.walk`。正如您在此示例中看到的那样，使用 `os.walk`，您无需测试哪些路径是文件，也无需在每个子目录中重新调用函数。

##### 示例 2-4\. 重写 walk_path

```py
def walk_path(parent_path):
    for parent_path, directories, files in os.walk(parent_path):
        print(f"Checking: {parent_path}")
        for file_name in files:
            file_path = os.path.join(parent_path, file_name)
            last_access = os.path.getatime(file_path)
            size = os.path.getsize(file_path)
            print(f"File: {file_path}")
            print(f"\tlast accessed: {last_access}")
            print(f"\tsize: {size}")
```

# 使用 Pathlib 作为对象的路径

`pathlib` 库将路径表示为对象而不是字符串。在 示例 2-5 中，我们使用 `pathlib` 而不是 `os.path` 重写了 示例 2-2。

##### 示例 2-5\. 重写 find_rc

```py
def find_rc(rc_name=".examplerc"):

    # Check for Env variable
    var_name = "EXAMPLERC_DIR"
    example_dir = os.environ.get(var_name) ![1](img/1.png)
    if example_dir:
        dir_path = pathlib.Path(example_dir) ![2](img/2.png)
        config_path = dir_path / rc_name ![3](img/3.png)
        print(f"Checking {config_path}")
        if config_path.exists(): ![4](img/4.png)
            return config_path.as_postix() ![5](img/5.png)

    # Check the current working directory
    config_path = pathlib.Path.cwd() / rc_name ![6](img/6.png)
    print(f"Checking {config_path}")
    if config_path.exists():
        return config_path.as_postix()

    # Check user home directory
    config_path = pathlib.Path.home() / rc_name ![7](img/7.png)
    print(f"Checking {config_path}")
    if config_path.exists():
        return config_path.as_postix()

    # Check Directory of This File
    file_path = pathlib.Path(__file__).resolve() ![8](img/8.png)
    parent_path = file_path.parent ![9](img/9.png)
    config_path = parent_path / rc_name
    print(f"Checking {config_path}")
    if config_path.exists():
        return config_path.as_postix()

    print(f"File {rc_name} has not been found")
```

![1](img/#co_automating_files_and_the_filesystem_CO5-1)

在撰写本文时，`pathlib` 不会展开环境变量。相反，您可以从 `os.environ` 中获取变量的值。

![2](img/#co_automating_files_and_the_filesystem_CO5-2)

这将创建适用于当前运行操作系统的 `pathlib.Path` 对象。

![3](img/#co_automating_files_and_the_filesystem_CO5-3)

您可以通过在父路径后跟正斜杠和字符串来构建新的 `pathlib.Path` 对象。

![4](img/#co_automating_files_and_the_filesystem_CO5-4)

`pathlib.Path` 对象本身具有 `exists` 方法。

![5](img/#co_automating_files_and_the_filesystem_CO5-5)

调用 `as_postix` 以将路径作为字符串返回。根据您的用例，您可以返回 `pathlib.Path` 对象本身。

![6](img/#co_automating_files_and_the_filesystem_CO5-6)

类方法 `pathlib.Path.cwd` 返回当前工作目录的 `pathlib.Path` 对象。此对象在此处立即用于通过与字符串 `rc_name` 连接来创建 `config_path`。

![7](img/#co_automating_files_and_the_filesystem_CO5-7)

类方法 `pathlib.Path.home` 返回当前用户的主目录的 `pathlib.Path` 对象。

![8](img/#co_automating_files_and_the_filesystem_CO5-8)

使用存储在 `*file*` 中的相对路径创建 `pathlib.Path` 对象，然后调用其 `resolve` 方法以获取绝对路径。

![9](img/#co_automating_files_and_the_filesystem_CO5-9)

这将直接从对象本身返回一个父级`pathlib.Path`对象。
