# 附录 D. 练习答案

# 1\. Python 的味道

1.1 如果你的计算机上还没有安装 Python 3，请立即安装。参见附录 B 获取详细的安装信息。

1.2 启动 Python 3 交互式解释器。关于如何做的详细信息在附录 B 中。它应该打印几行关于自身的信息，然后一行以`>>>`开头。这是你输入 Python 命令的提示符。

1.3 用解释器玩一会儿。像计算器一样使用它，输入`8 * 9`。按 Enter 键查看结果。Python 应该打印`72`。

1.4 输入数字`47`并按 Enter 键。它是否在下一行为你打印了 47？

1.5 现在输入`print(47)`并按 Enter 键。这会在下一行也打印出 47 吗？

# 2\. 数据：类型、值、变量和名称

2.1 将整数值`99`赋给变量`prince`，并打印出来。

2.2 值`5`的类型是什么？

2.3 值`2.0`的类型是什么？

2.4 表达式`5 + 2.0`的类型是什么？

# 3\. 数字

3.1 一个小时有多少秒？将秒钟数（`60`）乘以小时数（同样是`60`），使用交互式解释器作为计算器。

3.2 将前一个任务（一个小时的秒数）的结果赋给名为`seconds_per_hour`的变量。

3.3 一天有多少秒？使用你的`seconds_per_hour`变量。

3.4 再次计算一天的秒数，但这次将结果保存在名为`seconds_per_day`的变量中。

3.5 将`seconds_per_day`除以`seconds_per_hour`。使用浮点数（`/`）除法。

3.6 使用整数（`//`）除法将`seconds_per_day`除以`seconds_per_hour`。这个数值与前一个问题中的浮点数值是否一致，除了最后的`.0`？

# 4\. 选择`if`

4.1 选择一个介于 1 到 10 之间的数字，并将其赋给变量`secret`。然后，再选择另一个介于 1 到 10 之间的数字，并赋给变量`guess`。接下来，编写条件测试（`if`、`else`和`elif`）来打印字符串`'too low'`（如果`guess`小于`secret`）、`'too high'`（如果大于`secret`）和`'just right'`（如果等于`secret`）。

4.2 将`True`或`False`分配给变量`small`和`green`。编写一些`if`/`else`语句来打印与这些选择匹配的水果：cherry、pea、watermelon、pumpkin。

# 5\. 文本字符串

5.1 将以`m`开头的单词大写：

```py
>>> song = """When an eel grabs your arm,
... And it causes great harm,
... That's - a moray!"""
```

5.2 按正确的匹配打印每个列表问题和它们的答案，格式如下：

Q: *问题*

A: *答案*

```py
>>> questions = [
...     "We don't serve strings around here. Are you a string?",
...     "What is said on Father's Day in the forest?",
...     "What makes the sound 'Sis! Boom! Bah!'?"
...     ]
>>> answers = [
...     "An exploding sheep.",
...     "No, I'm a frayed knot.",
...     "'Pop!' goes the weasel."
...     ]
```

5.3 使用旧式格式编写以下诗歌。将字符串`'roast beef'`、`'ham'`、`'head'`和`'clam'`替换到这个字符串中：

```py
My kitty cat likes %s,
My kitty cat likes %s,
My kitty cat fell on his %s
And now thinks he's a %s.
```

5.4 使用新式格式编写一封表格信。将以下字符串保存为`letter`（你将在下一个练习中使用它）：

```py
Dear {salutation} {name},

Thank you for your letter. We are sorry that our {product}
{verbed} in your {room}. Please note that it should never
be used in a {room}, especially near any {animals}.

Send us your receipt and {amount} for shipping and handling.
We will send you another {product} that, in our tests,
is {percent}% less likely to have {verbed}.

Thank you for your support.

Sincerely,
{spokesman}
{job_title}
```

5.5 为名为`'salutation'`、`'name'`、`'product'`、`'verbed'`（过去式动词）、`'room'`、`'animals'`、`'percent'`、`'spokesman'`和`'job_title'`的字符串变量分配值。使用`letter.format()`打印`letter`与这些值。

5.6 在公众投票中命名事物后，出现了一种模式：英国潜艇（Boaty McBoatface）、澳大利亚赛马（Horsey McHorseface）和瑞典火车（Trainy McTrainface）。使用`%`格式化打印国家博览会上的获奖名字，用于鸭子、葫芦和斯皮茨。

5.7 使用`format()`格式化相同。

5.8 再来一次，使用*f 字符串*。

# 6\. 使用 while 和 for 循环

6.1 使用`for`循环打印列表`[3, 2, 1, 0]`的值。

6.2 将值`7`赋给变量`guess_me`，将值`1`赋给变量`number`。编写一个`while`循环，将`number`与`guess_me`进行比较。如果`number`小于`guess_me`，则打印`'too low'`。如果`number`等于`guess_me`，则打印`'found it!'`，然后退出循环。如果`number`大于`guess_me`，则打印`'oops'`，然后退出循环。在循环结束时增加`number`。

6.3 将值`5`赋给变量`guess_me`。使用`for`循环迭代名为`number`的变量，范围为`range(10)`。如果`number`小于`guess_me`，则打印`'too low'`。如果等于`guess_me`，则打印`'found it!'`，然后退出循环。如果`number`大于`guess_me`，则打印`'oops'`，然后退出循环。

# 7\. 元组和列表

7.1 创建名为`years_list`的列表，以你的出生年份开始，直到你五岁生日的年份。例如，如果你是 1980 年出生，列表将是`years_list = [1980, 1981, 1982, 1983, 1984, 1985]`。

7.2 在这些年份中，哪一年是你的第三个生日？请记住，你的第一年是 0 岁。

7.3 在`years_list`中哪一年你最年长？

7.4 创建并打印一个名为`things`的列表，其中包含这三个字符串作为元素：`"mozzarella"`、`"cinderella"`、`"salmonella"`。

7.5 将`things`中指向人的元素大写，然后打印列表。它改变了列表中的元素吗？

7.6 将`things`中的奶酪元素全部大写，然后打印列表。

7.7 删除疾病元素，领取你的诺贝尔奖，然后打印列表。

7.8 创建名为`surprise`的列表，元素为`"Groucho"`、`"Chico"`和`"Harpo"`。

7.9 将`surprise`列表的最后一个元素小写，反转它，然后大写它。

7.10 使用列表推导创建一个名为`even`的列表，其中包含`range(10)`中的偶数。

7.11 让我们创建一个跳绳童谣生成器。你将打印一系列两行押韵的诗句。从这个程序片段开始：

```py
start1 = ["fee", "fie", "foe"]
rhymes = [
    ("flop", "get a mop"),
    ("fope", "turn the rope"),
    ("fa", "get your ma"),
    ("fudge", "call the judge"),
    ("fat", "pet the cat"),
    ("fog", "walk the dog"),
    ("fun", "say we're done"),
    ]
start2 = "Someone better"
```

对于`rhymes`中的每对字符串（`first`、`second`）：

对于第一行：

+   打印`start1`中的每个字符串，大写并后跟感叹号和空格。

+   打印大写的`first`，后跟感叹号。

对于第二行：

+   打印`start2`和一个空格。

+   打印`second`和一个句号。

# 8\. 字典

8.1 创建一个英语到法语的字典`e2f`并打印它。这是你的起始词汇：`dog`是`chien`，`cat`是`chat`，`walrus`是`morse`。

8.2 使用你的三词字典`e2f`，打印`walrus`的法语单词。

8.3 从`e2f`创建一个法语到英语的字典`f2e`。使用`items`方法。

8.4 打印法语单词`chien`的英语对应词。

8.5 打印`e2f`中的英语单词集合。

8.6 创建一个名为`life`的多级字典。使用以下字符串作为顶层键：`'animals'`、`'plants'`和`'other'`。使`'animals'`键参考另一个字典，其中包含键`'cats'`、`'octopi'`和`'emus'`。使`'cats'`键参考一个包含值`'Henri'`、`'Grumpy'`和`'Lucy'`的字符串列表。使所有其他键参考空字典。

8.7 打印`life`的顶层键。

8.8 打印`life['animals']`的键。

8.9 打印`life['animals']['cats']`的值。

8.10 使用字典推导式创建字典`squares`。使用`range(10)`返回键，并使用每个键的平方作为其值。

8.11 使用集合推导式从`range(10)`中的奇数创建集合`odd`。

8.12 使用生成器推导式返回字符串`'Got '`和`range(10)`中的数字。通过使用`for`循环来迭代这个。

8.13 使用`zip()`从键元组`('optimist', 'pessimist', 'troll')`和值元组`('The glass is half full', 'The glass is half empty', 'How did you get a glass?')`创建一个字典。

8.14 使用`zip()`创建一个名为`movies`的字典，将这些列表配对：`titles = ['Creature of Habit', 'Crewel Fate', 'Sharks On a Plane']`和`plots = ['A nun turns into a monster', 'A haunted yarn shop', 'Check your exits']`

# 9\. 函数

9.1 定义一个名为`good()`的函数，返回以下列表：`['Harry', 'Ron', 'Hermione']`。

9.2 定义一个生成器函数`get_odds()`，返回`range(10)`中的奇数。使用`for`循环找到并打印第三个返回的值。

9.3 定义一个名为`test`的装饰器。当调用函数时，打印`'start'`，当函数完成时，打印`'end'`。

9.4 定义一个名为`OopsException`的异常。引发此异常以查看发生了什么。然后编写代码来捕获此异常并打印`'Caught an oops'`。

# 10\. Oh Oh：对象和类

10.1 创建一个名为`Thing`的类，没有内容并打印它。然后，从这个类创建一个名为`example`的对象并打印它。打印出的值是相同的还是不同的？

10.2 创建一个名为`Thing2`的新类，并将值`'abc'`分配给一个名为`letters`的类变量。打印`letters`。

10.3 再次创建一个名为`Thing3`的类。这次，将值`'xyz'`赋给一个名为`letters`的实例（对象）变量。打印`letters`。你需要创建一个类的对象来执行这个操作吗？

10.4 创建一个名为`Element`的类，具有实例属性`name`、`symbol`和`number`。使用值`'Hydrogen'`、`'H'`和`1`创建该类的对象`hydrogen`。

10.5 创建一个具有这些键和值的字典：`'name': 'Hydrogen'`、`'symbol': 'H'`、`'number': 1`。然后，使用这个字典从`Element`类创建一个名为`hydrogen`的对象。

10.6 对于`Element`类，定义一个名为`dump()`的方法，打印对象属性`name`、`symbol`和`number`的值。从这个新定义创建`hydrogen`对象，并使用`dump()`打印其属性。

10.7 调用`print(hydrogen)`。在`Element`的定义中，将方法`dump`的名称更改为`__str__`，创建一个新的`hydrogen`对象，并再次调用`print(hydrogen)`。

10.8 将`Element`修改为使`name`、`symbol`和`number`属性私有化。为每个属性定义一个 getter 属性以返回其值。

10.9 定义三个类：`Bear`、`Rabbit`和`Octothorpe`。对于每个类，只定义一个方法：`eats()`。分别返回`'berries'`（`Bear`）、`'clover'`（`Rabbit`）和`'campers'`（`Octothorpe`）。分别创建一个对象并打印它吃的东西。

10.10 定义这些类：`Laser`、`Claw`和`SmartPhone`。每个类只有一个方法：`does()`。分别返回`'disintegrate'`（`Laser`）、`'crush'`（`Claw`）或`'ring'`（`SmartPhone`）。然后定义一个`Robot`类，该类包含这三个组件对象的一个实例。为`Robot`定义一个`does()`方法，打印其组件对象的功能。

# 11\. 模块、包和好东西

11.1 创建一个名为*zoo.py*的文件。在其中定义一个名为`hours`的函数，打印字符串`'Open 9-5 daily'`。然后，在交互式解释器中导入`zoo`模块并调用其`hours`函数。

11.2 在交互式解释器中，将`zoo`模块作为`menagerie`导入，并调用其`hours()`函数。

11.3 在解释器中保持不变，直接从`zoo`导入`hours()`函数并调用它。

11.4 将`hours()`函数作为`info`导入并调用。

11.6 创建一个名为`fancy`的`OrderedDict`，使用相同的键值对并打印它。它和`plain`打印出来的顺序一样吗？

11.7 创建一个名为`dict_of_lists`的`defaultdict`，并传递`list`作为参数。创建列表`dict_of_lists['a']`并通过一次赋值将值`'something for a'`附加到其中。打印`dict_of_lists['a']`。

# 12\. 数据处理与管理

12.1 创建一个名为`mystery`的 Unicode 字符串，并将其赋值为`'\U0001f984'`。打印`mystery`及其 Unicode 名称。

12.2 使用 UTF-8 对`mystery`进行编码，将结果存入名为`popbytes`的`bytes`变量中。打印`pop_bytes`。

12.3 使用 UTF-8 解码`popbytes`，将结果存入名为`pop_string`的字符串变量中。打印`pop_string`。`pop_string`等于`mystery`吗？

12.4 当你处理文本时，正则表达式非常方便。我们将以多种方式应用它们到我们特选的文本样本中。这是一首名为《关于庞大奶酪的颂歌》的诗，由詹姆斯·麦金泰尔在 1866 年创作，以向安大略州制造的一块七千磅重的奶酪致敬，并送往国际巡回演出。如果你不想全部输入，可以使用你喜欢的搜索引擎复制并粘贴这些词到你的 Python 程序中，或者直接从[Project Gutenberg](http://bit.ly/mcintyre-poetry)中获取。将文本字符串命名为 `mammoth`。

12.5 导入`re`模块以使用 Python 的正则表达式功能。使用`re.findall()`打印所有以`c`开头的单词。

12.6 查找所有以`c`开头且长度为四个字母的单词。

12.7 找出所有以`r`结尾的单词。

12.8 找出所有包含恰好三个连续元音字母的单词。

12.9 使用`unhexlify()`将这个十六进制字符串（由两个字符串组合以适应一页）转换为名为`gif`的`bytes`变量：

```py
'47494638396101000100800000000000ffffff21f9' +
'0401000000002c000000000100010000020144003b'
```

12.10 `gif`中的字节定义了一个像素的透明 GIF 文件，这是最常见的图形文件格式之一。一个合法的 GIF 以字符串*GIF89a*开始。`gif`是否匹配这个？

12.11 GIF 的像素宽度是从字节偏移 6 开始的 16 位小端整数，高度也是同样大小，从偏移 8 开始。提取并打印这些值到变量`gif`中。它们都是`1`吗？

# 13\. 日历与时钟

13.1 将当前日期作为字符串写入名为*today.txt*的文本文件。

13.2 将名为*today.txt*的文本文件读入字符串`today_string`。

13.3 从`today_string`中解析日期。

13.4 创建一个你出生日期的日期对象。

13.5 你出生的那一天是星期几？

13.6 你什么时候会（或者你何时）满 10000 天？

# 14\. 文件与目录

14.1 列出当前目录中的文件。

14.2 列出你的父目录中的文件。

14.3 将字符串`'This is a test of the emergency text system'`赋给变量`test1`，并将`test1`写入名为*test.txt*的文件。

14.4 打开名为*test.txt*的文件并将其内容读入字符串`test2`。`test1`和`test2`相同吗？

# 15\. 时间中的数据：进程与并发

15.1 使用`multiprocessing`创建三个单独的进程。每个进程在 0 到 1 秒之间等待一个随机数，打印当前时间，然后退出。

# 16\. 数据盒子：持久存储

16.1 将以下文本行保存到名为*books.csv*的文件中（注意，如果字段用逗号分隔，如果包含逗号，则需要用引号括起来）：

```py
author,book
J R R Tolkien,The Hobbit
Lynne Truss,"Eats, Shoots & Leaves"
```

16.2 使用`csv`模块及其`DictReader`方法将*books.csv*读取到变量`books`中。打印`books`的值。`DictReader`处理了第二本书标题中的引号和逗号吗？

16.3 通过以下行创建名为*books2.csv*的 CSV 文件：

```py
title,author,year
The Weirdstone of Brisingamen,Alan Garner,1960
Perdido Street Station,China Miéville,2000
Thud!,Terry Pratchett,2005
The Spellman Files,Lisa Lutz,2007
Small Gods,Terry Pratchett,1992
```

16.4 使用`sqlite3`模块创建一个名为*books.db*的 SQLite 数据库，并创建一个名为`books`的表，包含以下字段：`title`（文本）、`author`（文本）和`year`（整数）。

16.5 读取*books2.csv*并将其数据插入`book`表中。

16.6 按字母顺序选择并打印`book`表中的`title`列。

16.7 按出版顺序选择并打印`book`表中的所有列。

16.8 使用`sqlalchemy`模块连接到您刚刚在练习 8.6 中创建的 sqlite3 数据库*books.db*。像 8.8 中一样，按字母顺序选择并打印`book`表中的`title`列。

16.9 在您的机器上安装 Redis 服务器（参见附录 B）和 Python 的`redis`库（`pip install redis`）。创建一个名为`test`的 Redis 哈希，具有字段`count`（`1`）和`name`（`'Fester Bestertester'`）。打印`test`的所有字段。

16.10 增加`test`的`count`字段并打印它。

# 17\. 空间数据：网络

17.1 使用普通的`socket`实现一个当前时间服务。当客户端向服务器发送字符串`'time'`时，返回当前日期和时间的 ISO 字符串。

17.2 使用 ZeroMQ 的 REQ 和 REP 套接字来执行相同的操作。

17.3\. 尝试使用 XMLRPC 做同样的事情。

17.4 你可能看过经典的*I Love Lucy*电视剧集，其中 Lucy 和 Ethel 在巧克力工厂工作。随着供应他们加工的糖果的传送带速度越来越快，二人开始落后。编写一个模拟程序，将不同类型的巧克力推送到 Redis 列表中，Lucy 作为客户端进行阻塞弹出这个列表。她需要 0.5 秒处理一块巧克力。打印每块巧克力到达 Lucy 手中的时间和类型，以及剩余待处理的数量。

17.5 使用 ZeroMQ 发布从练习 12.4（来自示例 12-1）的诗歌，逐字发布。编写一个 ZeroMQ 消费者，打印以元音字母开头的每个单词，以及包含五个字母的每个单词。忽略标点符号字符。

# 18\. 解析 Web

18.1 如果您还没有安装`flask`，请立即安装。这也将安装`werkzeug`、`jinja2`和可能其他包。

18.2 使用 Flask 的调试/重载开发 Web 服务器构建一个骨架网站。确保服务器在默认端口`5000`上启动，主机名为`localhost`。如果您的机器已经在使用端口 5000 做其他事情，请使用其他端口号。

18.3 添加一个`home()`函数来处理对主页的请求。设置它返回字符串`It's alive!`。

18.4 创建一个名为*home.html*的 Jinja2 模板文件，内容如下：

```py
I'm of course referring to {{thing}},
which is {{height}} feet tall and {{color}}.
```

创建一个名为*templates*的目录，并创建文件*home.html*，其内容如上所示。如果您之前的 Flask 服务器仍在运行，它将检测到新内容并重新启动自身。

18.5 修改您服务器的`home()`函数以使用*home.html*模板。为其提供三个`GET`参数：`thing`、`height`和`color`。

# 19\. 成为 Pythonista

(Pythonistas 今天没有作业。)

# 20\. Py 艺术

20.1 安装`matplotlib`。绘制这些(x, y)对的散点图：`( (0, 0), (3, 5), (6, 2), (9, 8), (14, 10) )`。

20.2 绘制相同数据的折线图。

20.3 绘制相同数据的图表（带有标记的折线图）。

# 21\. Py at Work

21.1 安装`geopandas`并运行 示例 21-1。尝试修改颜色和标记大小。

# 22\. PySci

22.1 安装 Pandas。获取 示例 16-1 中的 CSV 文件。运行 示例 16-2 中的程序。尝试使用一些 Pandas 命令。
