# 第十三章：时间操作

Python 程序可以以多种方式处理时间。时间*间隔*是以秒为单位的浮点数（时间间隔的小数部分是间隔的小数部分）：所有标准库函数接受以秒为单位表示时间间隔的参数，接受浮点数作为该参数的值。时间中的*瞬间*是自某个参考瞬间以来的秒数，称为* epoch *。（尽管每种语言和每个平台的 epoch 有所不同，但在所有平台上，Python 的 epoch 是 UTC 时间，1970 年 1 月 1 日午夜。）时间瞬间通常还需要以多种单位（例如年、月、日、小时、分钟和秒）的混合形式表示，特别是用于输入输出目的。当然，输入输出还需要能够将时间和日期格式化为人类可读的字符串，并从字符串格式解析它们回来。

# 时间模块

时间模块在某种程度上依赖于底层系统的 C 库，这限定了时间模块可以处理的日期范围。在旧的 Unix 系统中，1970 年和 2038 年是典型的截止点¹（这个限制可以通过使用 datetime 避免，后文将讨论）。时间点通常以 UTC（协调世界时，曾称为 GMT 或格林尼治平均时间）指定。时间模块还支持本地时区和夏令时（DST），但仅限于底层 C 系统库支持的范围²。

作为秒数自纪元以来的一个替代方法，时间点可以用一个包含九个整数的元组表示，称为* timetuple *（在 Table 13-1 中有介绍）。所有项目都是整数：timetuples 不跟踪秒的小数部分。一个 timetuple 是 struct_time 的一个实例。你可以将其用作元组；更有用的是，你可以通过只读属性访问项目，如 *x*.tm_year，*x*.tm_mon 等等，属性名称在 Table 13-1 中列出。在任何需要 timetuple 参数的函数中，你都可以传递 struct_time 的实例或任何其他项目是九个整数且范围正确的序列（表中的所有范围都包括下限和上限，都是包容的）。

表 13-1 元组形式的时间表示

| 项目 | 含义 | 字段名 | 范围 | 备注 |
| --- | --- | --- | --- | --- |
| 0 | Year | tm_year | 1970–2038 | 有些平台支持 0001–9999 |
| 1 | Month | tm_mon | 1–12 | 1 代表一月；12 代表十二月 |
| 2 | Day | tm_mday | 1–31 |   |
| 3 | 小时 | tm_hour | 0–23 | 0 表示午夜；12 表示中午 |
| 4 | 分钟 | tm_min | 0–59 |   |
| 5 | 秒 | tm_sec | 0–61 | 60 和 61 表示闰秒 |
| 6 | 星期几 | tm_wday | 0–6 | 0 表示星期一；6 表示星期日 |
| 7 | 年内天数 | tm_yday | 1–366 | 年内的日期编号 |
| 8 | 夏令时标志 | tm_isdst | −1–1 | −1 表示库确定夏令时 |

要将“自纪元以来的秒数”浮点值转换为时间元组，请将浮点值传递给函数（例如 localtime），该函数返回所有九个有效项目的时间元组。在反向转换时，mktime 忽略元组的多余项目 6（tm_wday）和 7（tm_yday）。在这种情况下，通常将项目 8（tm_isdst）设置为 −1，以便 mktime 自行确定是否应用 DST。

time 提供了 表 13-2 中列出的函数和属性。

表 13-2\. *time* 模块的函数和属性

| asctime | asctime([*tupletime*]) 接受时间元组并返回可读的 24 字符串，例如 'Sun Jan 8 14:41:06 2017'。调用 asctime() 无参数相当于调用 asctime(time.localtime())（格式化当前本地时间）。 |
| --- | --- |
| ctime | ctime([*secs*]) 类似于 asctime(localtime(*secs*))，接受以自纪元以来的秒数表示的瞬时，并返回该瞬时的可读的 24 字符串形式，以本地时间显示。调用 ctime() 无参数相当于调用 asctime()（格式化当前本地时间）。 |
| gmtime | gmtime([*secs*]) 接受以自纪元以来的秒数表示的瞬时，并返回 UTC 时间的时间元组 *t*（*t*.tm_isdst 总是 0）。调用 gmtime() 无参数相当于调用 gmtime(time())（返回当前时间瞬时的时间元组）。 |
| localtime | localtime([*secs*]) 接受自纪元以来经过的秒数的瞬时，并返回本地时间的时间元组 *t*（*t*.tm_isdst 根据本地规则应用于瞬时 *secs* 是 0 或 1）。调用 localtime() 无参数相当于调用 localtime(time())（返回当前时间瞬时的时间元组）。 |
| mktime | mktime(*tupletime*) 接受作为本地时间的时间元组表示的瞬时，并返回以自纪元以来的秒数表示的浮点值（即使在 64 位系统上，只接受 1970–2038 之间的有限纪元日期，而不是扩展范围）^(a)。*tupletime* 中的最后一项 DST 标志具有意义：将其设置为 0 以获取标准时间，设置为 1 以获取 DST，或设置为 −1 让 mktime 计算给定瞬时时是否适用 DST。 |
| monotonic | monotonic() 类似于 time()，返回当前时间瞬时的浮点数秒数；然而，保证时间值在调用之间不会后退，即使系统时钟调整（例如由于闰秒或在切换到或从 DST 时刻）。 |
| perf_counter | perf_counter() 用于测量连续调用之间的经过时间（如秒表），perf_counter 返回使用最高分辨率时钟得到的秒数值，以获取短时间内的精确度。它是系统范围的，并且在休眠期间也包括经过的时间。只使用连续调用之间的差异，因为没有定义的参考点。 |
| process_time | process_time() 像 perf_counter 一样；但是，返回的时间值是进程范围的，并且*不*包括在休眠期间经过的时间。仅使用连续调用之间的差异，因为没有定义的参考点。 |
| sleep | sleep(*secs*) 暂停调用线程* secs *秒。如果是主线程并且某些信号唤醒了它，则在* secs *秒（当它是唯一准备运行的当前线程时）之前或更长时间的暂停之后，调用线程可能会再次开始执行（取决于进程和线程的系统调度）。您可以将* secs *设置为 0 调用 sleep，以便为其他线程提供运行机会，如果当前线程是唯一准备运行的线程，则不会造成显著延迟。 |

| strftime | strftime(*fmt*[, *tupletime*]) 接受表示本地时间的时间元组* tupletime *，并返回字符串，该字符串表示按* fmt *指定的即时时间。如果省略* tupletime *，strftime 使用本地时间（time（））（格式化当前即时时间）。* fmt *的语法类似于“使用％进行传统字符串格式化”，尽管转换字符不同，如表 13-3 所示。参考* tupletime *指定的时间即时；格式无法指定宽度和精度。

例如，你可以使用 asctime 格式（例如，'Tue Dec 10 18:07:14 2002'）获取日期，格式字符串为'%a %b %d %H:%M:%S %Y'。

您可以使用格式字符串'%a, %d %b %Y %H:%M:%S %Z'获取与 RFC 822 兼容的日期（例如'Tue, 10 Dec 2002 18:07:14 EST'）。

这些字符串也可用于使用“用户编码类的格式化”中讨论的机制进行日期时间格式化，允许您等效地编写，对于 datetime.datetime 对象 d，可以写为 f'{d:%Y/%m/%d}'或'{:%Y/%m/%d}'.format(d)，两者都会给出例如'2022/04/17'的结果。对于 ISO 8601 格式的日期时间，请参阅“日期类”中涵盖的 isoformat（）和 fromisoformat（）方法。|

| strptime | strptime(*str*, *fmt*) 根据格式字符串*fmt*（例如'%a %b %d %H:%M:%S %Y'，详见 strftime 讨论）解析*str*，并返回时间元组作为即时。如果未提供时间值，默认为午夜。如果未提供日期值，默认为 1900 年 1 月 1 日。例如：

```py
>>> print(time.strptime("Sep 20, 2022", '%b %d, %Y'))
```

```py
time.struct_time(tm_year=2022, tm_mon=9, tm_mday=20, 
tm_hour=0, tm_min=0, tm_sec=0, tm_wday=1, 
tm_yday=263, tm_isdst=-1)
```

|

| time | time() 返回当前时间即时，一个从纪元以来的浮点数秒数。在一些（主要是较旧的）平台上，此时间的精度低至一秒。如果系统时钟在调用之间向后调整（例如由于闰秒），则可能在后续调用中返回较低的值。 |
| --- | --- |
| timezone | 本地时区（无夏令时）与 UTC 的偏移量（<0 为美洲；>=0 为大部分欧洲、亚洲和非洲）。 |
| tzname | 本地时间区域依赖的一对字符串，即无夏令时和有夏令时的本地时区名称。 |
| ^(a) mktime 的结果小数部分总是 0，因为其 timetuple 参数不考虑秒的小数部分。 |

表 13-3\. strftime 的转换字符

| 类型字符 | 含义 | 特殊说明 |
| --- | --- | --- |
| a | 星期几名称（缩写） | 取决于区域设置 |
| A | 星期几名称（完整） | 取决于区域设置 |
| b | 月份名称（缩写） | 取决于区域设置 |
| B | 月份名称（完整） | 取决于区域设置 |
| c | 完整的日期和时间表示 | 取决于区域设置 |
| d | 月份中的第几天 | 从 1 到 31 |
| f | 微秒数以小数形式，零填充到六位数 | 一到六位数字 |
| G | ISO 8601:2000 标准的基于周的年份编号 |   |
| H | 小时数（24 小时制钟） | 从 0 到 23 |
| I | 小时数（12 小时制钟） | 从 1 到 12 |
| j | 年份中的第几天 | 从 1 到 366 |
| m | 月份编号 | 从 1 到 12 |
| M | 分钟数 | 从 0 到 59 |
| p | 上午或下午的等价项 | 取决于区域设置 |
| S | 秒数 | 从 0 到 61 |
| u | 星期几 | 星期一为 1，最多为 7 |
| U | 周数（以星期天为第一天） | 从 0 到 53 |
| V | ISO 8601:2000 标准的基于周的周数 |   |
| w | 星期几编号 | 0 表示星期天，最大为 6 |
| W | 周数（以星期一为第一天） | 从 0 到 53 |
| x | 完整的日期表示 | 取决于区域设置 |
| X | 完整的时间表示 | 取决于区域设置 |
| y | 世纪内的年份编号 | 从 0 到 99 |
| Y | 年份编号 | 从 1970 到 2038，或更宽 |
| z | UTC 偏移量作为字符串：±HHMM[SS[.ffffff]] |   |
| Z | 时区名称 | 如果不存在时区则为空 |
| % | 字面上的 % 字符 | 编码为 %% |

# datetime 模块

datetime 提供了用于建模日期和时间对象的类，这些对象可以是*有意识*的时区或*无意识*的（默认）。类 tzinfo 的实例用于建模时区，是抽象的：datetime 模块仅提供一个简单的实现 datetime.timezone（更多详细信息，请参阅[在线文档](https://oreil.ly/8Bt8N)）。在下一节中讨论的 zoneinfo 模块提供了 tzinfo 的更丰富的具体实现，它允许您轻松创建时区感知的 datetime 对象。datetime 中的所有类型都有不可变的实例：属性是只读的，实例可以是字典中的键或集合中的项，所有函数和方法都返回新对象，从不改变作为参数传递的对象。

## date 类

date 类的实例表示一个日期（特定日期内的无特定时间），满足 date.min <= d <= date.max，始终是无意识的，并假设格里高利历始终有效。date 实例具有三个只读整数属性：*year*、*month* 和 *day*。此类的构造函数签名如下：

| date | **class** date(*year, month, day*) 返回给定 *year*、*month* 和 *day* 参数的日期对象，有效范围为 1 <= *year* <= 9999，1 <= *month* <= 12，以及 1 <= *day* <= *n*，其中 *n* 是给定月份和年份的天数。如果给出无效值，则引发 ValueError。 |
| --- | --- |

日期类还提供了作为替代构造函数可用的三个类方法，列在 表 13-4 中。

表 13-4\. 替代日期构造函数

| fromordinal | date.fromordinal(*ordinal*) 返回一个日期对象，对应于普通格里高利纪元中的 *ordinal*，其中值为 1 对应于公元 1 年的第一天。 |
| --- | --- |
| fromtimestamp | date.fromtimestamp(*timestamp*) 返回一个日期对象，对应于自纪元以来以秒表示的 *timestamp*。 |
| today | date.today() 返回表示今天日期的日期对象。 |

日期类的实例支持一些算术操作。日期实例之间的差异是一个 timedelta 实例；您可以将 timedelta 添加到日期实例或从日期实例中减去 timedelta 以创建另一个日期实例。您也可以比较日期类的任意两个实例（后面的日期较大）。

类 date 的实例 *d* 提供了列表中列出的方法，详见 表 13-5。

表 13-5\. 类 date 的实例 *d* 的方法

| ctime | *d*.ctime() 返回一个字符串，表示日期 *d* 的格式与 time.ctime 中的 24 字符格式相同（日期设置为 00:00:00，午夜）。 |
| --- | --- |
| isocalendar | *d*.isocalendar() 返回一个包含三个整数的元组（ISO 年份、ISO 周数和 ISO 工作日）。更多关于 ISO（国际标准化组织）日历的详细信息，请参见 [ISO 8601 标准](https://oreil.ly/e5yfG)。 |
| isoformat | *d*.isoformat() 返回一个字符串，表示日期 *d* 的格式为 'YYYY-MM-DD'；与 str(*d*) 相同。 |
| isoweekday | *d*.isoweekday() 返回日期 *d* 的星期几作为整数，星期一为 1，星期日为 7；类似于 d.weekday() + 1。 |

| replace | *d*.replace(year=**None**, month=**None**, day=**None**) 返回一个新的日期对象，类似于 *d*，但显式指定为参数的那些属性被替换。例如：

```py
date(x,y,z).replace(month=m) == date(x,m,z)
```

|

| strftime | *d*.strftime(*fmt*) 返回一个字符串，表示日期 *d* 按字符串 *fmt* 指定的格式。例如：

```py
time.strftime(*`fmt`*, *`d`*.timetuple())
```

|

| timetuple | *d*.timetuple() 返回一个 timetuple，对应于日期 *d* 的时间为 00:00:00（午夜）。 |
| --- | --- |

| toordinal | *d*.toordinal() 返回日期 *d* 的普通格里高利日期。例如：

```py
date(1,1,1).toordinal() == 1
```

|

| weekday | *d*.weekday() 返回日期 *d* 的星期几作为整数，星期一为 0，星期日为 6；类似于 d.isoweekday() - 1。 |
| --- | --- |

## 时间类

时间类的实例表示一天中的时间（没有特定的日期），可以是关于时区的明确或无意识的，并且总是忽略闰秒。它们有五个属性：四个只读整数（小时、分钟、秒和微秒）和一个可选的只读 tzinfo 属性（无意识实例的情况下为 **None**）。时间类的构造函数的签名为：

| time | **class** time(hour=0, minute=0, second=0, microsecond=0, tzinfo=**None**) 类时间的实例不支持算术运算。可以比较两个时间实例（时间较晚的为较大），但仅当它们都是明确或都是无意识的时才行。 |
| --- | --- |

类时间的实例 *t* 提供了 表 13-6 中列出的方法。

表 13-6\. 类时间实例 t 的方法

| isoformat | *t*.isoformat() 返回一个表示时间 *t* 的字符串，格式为 'HH:MM:SS'；与 str(*t*) 相同。如果 *t*.microsecond != 0，则结果字符串较长：'HH:MM:SS.mmmmmm'。如果 *t* 是明确的，则在末尾添加六个字符 '+HH:MM'，表示时区与 UTC 的偏移量。换句话说，此格式化操作遵循 [ISO 8601 标准](https://oreil.ly/e5yfG)。 |
| --- | --- |

| replace | *t*.replace(hour=**None**, minute=**None**, second=**None**, microsecond=**None**[, *tzinfo*]) 返回一个新的时间对象，类似于 *t*，除了那些显式指定为参数的属性将被替换。例如：

```py
time(x,y,z).replace(minute=m) == time(x,m,z)
```

|

| strftime | *t*.strftime(*fmt*) 返回一个字符串，表示按照字符串 *fmt* 指定的时间 *t*。 |
| --- | --- |

类时间的实例 *t* 还提供了方法 dst、tzname 和 utcoffset，它们不接受参数并委托给 *t*.tzinfo，当 *t*.tzinfo 为 **None** 时返回 **None**。

## 日期时间类

日期时间类的实例表示一个瞬间（一个日期，在该日期内具体的时间），可以是关于时区的明确或无意识的，并且总是忽略闰秒。日期时间扩展了日期并添加了时间的属性；它的实例有只读整数属性年、月、日、小时、分钟、秒和微秒，以及一个可选的 tzinfo 属性（无意识实例的情况下为 **None**）。此外，日期时间实例有一个只读的 fold 属性，用于在时钟回滚期间区分模糊的时间戳（例如夏令时结束时的“回退”，在凌晨 1 点到 2 点之间创建重复的无意识时间）。fold 取值为 0 或 1，0 对应于回滚前的时间；1 对应于回滚后的时间。

日期时间实例支持一些算术运算：两个日期时间实例之间的差异（均为明确或均为无意识）是一个 timedelta 实例，并且可以将 timedelta 实例添加到或从日期时间实例中减去以构造另一个日期时间实例。可以比较两个日期时间类的实例（较晚的为较大），只要它们都是明确或都是无意识的。此类的构造函数的签名为：

| datetime | **class** datetime(*year*, *month*, *day*, hour=0, minute=0, second=0, microsecond=0, tzinfo=**None**, *, fold=0) 返回一个日期时间对象，遵循与日期类构造函数类似的约束。fold 是一个整数，值为 0 或 1，如前所述。 |
| --- | --- |

datetime 还提供一些类方法，可用作替代构造函数，详见 Table 13-7。

Table 13-7\. 替代日期时间构造函数

| combine | datetime.combine(*date*, *time*) 返回一个日期时间对象，日期属性来自 *date*，时间属性（包括时区信息）来自 *time*。datetime.combine(*d, t*) 的作用类似于： |

```py
datetime(d.year, d.month, d.day,
         t.hour, t.minute, t.second,
         t.microsecond, t.tzinfo)
```

|

| fromordinal | datetime.fromordinal(*ordinal*) 返回一个日期对象，表示普通格里高利历的序数日期 *ordinal*，其中值为 1 表示公元 1 年的第一天午夜。 |
| --- | --- |
| fromtimestamp | datetime.fromtimestamp(*timestamp*, tz=**None**) 返回一个日期时间对象，表示自纪元以来经过的秒数 *timestamp* 对应的时刻，以本地时间表示。当 tz 不是 **None** 时，返回一个带有给定 tzinfo 实例 tz 的带时区信息的日期时间对象。 |
| now | datetime.now(tz=**None**) 返回一个表示当前本地日期和时间的无时区信息的日期时间对象。当 tz 不是 **None** 时，返回一个带有给定 tzinfo 实例 tz 的带时区信息的日期时间对象。 |
| strptime | datetime.strptime(*str*, *fmt*) 返回一个日期时间对象，表示按照字符串 *fmt* 指定的格式解析的 *str*。当 *fmt* 中包含 %z 时，生成的日期时间对象是带时区信息的。 |
| today | datetime.today() 返回一个表示当前本地日期和时间的无时区信息的日期时间对象；与 now 类方法相同，但不接受可选参数 *tz*。 |
| utcfromtimestamp | datetime.utcfromtimestamp(*timestamp*) 返回一个表示自纪元以来经过的秒数 *timestamp* 对应时刻的无时区信息的日期时间对象，使用的是协调世界时（UTC）。 |
| utcnow | datetime.utcnow() 返回一个表示当前日期和时间的无时区信息的日期时间对象，使用的是协调世界时（UTC）。 |

日期时间实例 *d* 还提供了 Table 13-8 中列出的方法。

Table 13-8\. datetime 实例 *d* 的方法

| astimezone | *d*.astimezone(*tz*) 返回一个新的带时区信息的日期时间对象，类似于 *d*，但日期和时间与时区 *tz* 一起转换。^(a) *d* 必须是带时区信息的，以避免潜在的 bug。传递一个无时区信息的 *d* 可能导致意外结果。 |
| --- | --- |
| ctime | *d*.ctime() 返回一个字符串，表示与 *d* 的日期时间在与 time.ctime 相同的 24 字符格式中。 |
| date | *d*.date() 返回一个表示与 *d* 相同日期的日期对象。 |
| isocalendar | *d*.isocalendar() 返回一个包含三个整数的元组（ISO 年份、ISO 周号和 ISO 工作日），表示 *d* 的日期。 |
| isoformat | *d*.isoformat(sep='T') 返回一个字符串，表示*d*的格式为'YYYY-MM-DDxHH:MM:SS'，其中*x*是参数 sep 的值（必须是长度为 1 的字符串）。如果*d*.microsecond != 0，则在字符串的'SS'部分之后添加七个字符'.mmmmmm'。如果*t*是已知的，则在最后添加六个字符'+HH:MM'，以表示时区与 UTC 的偏移量。换句话说，此格式化操作遵循 ISO 8601 标准。str(*d*)与*d*.isoformat(sep=' ')相同。 |
| isoweekday | *d*.isoweekday() 返回*d*日期的星期几，返回一个整数，星期一为 1，星期日为 7。 |

| replace | *d*.replace(year=**None**, month=**None**, day=**None**, hour=**None**, minute=**None**, second=**None**, microsecond=**None**, tzinfo=**None**,*, fold=0) 返回一个新的 datetime 对象，类似于*d*，但指定为参数的那些属性被替换（但不进行任何时区转换——如果要转换时间，请使用 astimezone）。您还可以使用 replace 从 naive 创建一个已知的 datetime 对象。例如：

```py
*`# create datetime replacing just month with no`* 
*`# other changes (== datetime(x,m,z))`*
datetime(x,y,z).replace(month=m) 
*`# create aware datetime from naive datetime.now()`*
*`d`* = datetime.now().replace(tzinfo=ZoneInfo(
                    "US/Pacific"))
```

|

| strftime | *d*.strftime(*fmt*) 返回一个字符串，表示根据格式字符串*fmt*指定的格式显示的*d*。 |
| --- | --- |
| time | *d*.time() 返回一个表示与*d*相同一天中的时间的 naive 时间对象。 |
| timestamp | *d*.timestamp() 返回自纪元以来的秒数的浮点数。假设 naive 实例处于本地时区。 |
| timetuple | *d*.timetuple() 返回与时刻*d*对应的时间元组。 |
| timetz | *d*.timetz() 返回一个时间对象，表示与*d*相同的一天中的时间，具有相同的时区信息。 |

| toordinal | *d*.toordinal() 返回*d*日期的公历序数。例如：

datetime(1, 1, 1).toordinal() == 1 |

| utct⁠i⁠m⁠e​t⁠u⁠p⁠le | *d*.utctimetuple() 返回一个时间元组，对应于时刻*d*，如果*d*是已知的，则规范化为 UTC。 |
| --- | --- |
| weekday | *d*.weekday() 返回*d*日期的星期几，返回一个整数，星期一为 0，星期日为 6。 |
| ^(a) 请注意*d*.astimezone(*tz*)与*d*.replace(tzinfo=*tz*)非常不同：replace 不进行时区转换，而只是复制了*d*的所有属性，但*d*.tzinfo 除外。 |

类 datetime 的实例*d*还提供了方法 dst、tzname 和 utcoffset，这些方法不接受参数，并委托给*d*.tzinfo，在*d*.tzinfo 为**None**（即*d*是 naive 时）时返回**None**。

## timedelta 类

timedelta 类的实例表示具有三个只读整数属性的时间间隔：days、seconds 和 microseconds。此类的构造函数的签名为：

| timedelta | timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0) 将所有单位按照明显的因子转换（一周为 7 天，一小时为 3,600 秒等），并将一切标准化为三个整数属性，确保 0 <= seconds < 24 * 60 * 60 且 0 <= microseconds < 1000000。例如：

```py
>>> print(repr(timedelta(minutes=0.5)))
```

```py
datetime.timedelta(days=0, seconds=30)
```

```py
>>> print(repr(timedelta(minutes=-0.5)))
```

```py
datetime.timedelta(days=-1, seconds=86370)
```

timedelta 的实例支持算术运算：与 timedelta 类型的实例之间的 + 和 -；与整数之间的 *；与整数和 timedelta 实例之间的 /（地板除法、真除法、divmod、%）。它们还支持彼此之间的比较。 |

虽然可以使用此构造函数创建 timedelta 实例，但更常见的是通过两个日期、时间或 datetime 实例相减创建 timedelta，使得结果 timedelta 表示经过的时间段。timedelta 的实例 *td* 提供了一个方法 *td*.total_seconds()，返回表示 timedelta 实例的总秒数的浮点数。

## tzinfo 抽象类

tzinfo 类定义了在表 13-9 中列出的抽象类方法，用于支持创建和使用带有时区意识的 datetime 和 time 对象。

表 13-9\. tzinfo 类的方法

| dst | dst(*dt*) 返回给定 datetime 的夏令时偏移量，作为 timedelta 对象 |
| --- | --- |
| tzname | tzname(*dt*) 返回给定 datetime 的时区缩写 |
| utcoffset | utcoffset(*dt*) 返回给定 datetime 的与 UTC 的偏移量，作为 timedelta 对象 |

tzinfo 还定义了一个 fromutc 抽象实例方法，主要供 datetime.astimezone 方法内部使用。

## timezone 类

timezone 类是 tzinfo 类的具体实现。您可以使用表示与 UTC 时间偏移量的 timedelta 构造一个 timezone 实例。timezone 提供一个类属性 utc，代表 UTC 时区（相当于 timezone(timedelta(0))）。

# zoneinfo 模块

3.9+ zoneinfo 模块是 datetime 的 tzinfo 的具体实现，用于时间区域的表示。³ zoneinfo 默认使用系统的时区数据，以 [tzdata](https://oreil.ly/i1PF6) 作为后备。（在 Windows 上，您可能需要 **pip install tzdata**；一旦安装完成，您不需要在程序中导入 tzdata—zoneinfo 会自动使用它。）

zoneinfo 提供一个类：ZoneInfo，它是 datetime.tzinfo 抽象类的具体实现。您可以在创建带有时区意识的 datetime 实例时将其赋给 tzinfo，或者在 datetime.replace 或 datetime.astimezone 方法中使用它。要构造 ZoneInfo，请使用定义的 IANA 时区名称之一，例如 "America/Los_Angeles" 或 "Asia/Tokyo"。您可以通过调用 zoneinfo.available_timezones() 获取这些时区名称的列表。更多有关每个时区的详细信息（例如与 UTC 的偏移和夏令时信息）可以在[Wikipedia](https://oreil.ly/0u4KW)上找到。

这里有一些使用 ZoneInfo 的示例。我们将从获取加州当前本地日期和时间开始：

```py
>>> `from` datetime `import` datetime
>>> `from` zoneinfo `import` ZoneInfo
>>> d=datetime.now(tz=ZoneInfo("America/Los_Angeles"))
>>> d
```

```py
datetime.datetime(2021,10,21,16,32,23,96782,tzinfo=zoneinfo.ZoneInfo(key
='America/Los_Angeles'))
```

现在我们可以将时区更新为另一个时区，而不改变其他属性（即不将时间转换为新时区）：

```py
>>> dny=d.replace(tzinfo=ZoneInfo("America/New_York"))
>>> dny
```

```py
datetime.datetime(2021,10,21,16,32,23,96782,tzinfo=zoneinfo.ZoneInfo(key
='America/New_York')) 
```

将 datetime 实例转换为 UTC：

```py
>>> dutc=d.astimezone(tz=ZoneInfo("UTC"))
>>> dutc
```

```py
datetime.datetime(2021,10,21,23,32,23,96782,tzinfo=zoneinfo.ZoneInfo(key
='UTC')) 
```

获取当前时间的*明晰*时间戳在 UTC 时区：

```py
>>> daware=datetime.utcnow().replace(tzinfo=ZoneInfo("UTC"))
>>> daware
```

```py
datetime.datetime(2021,10,*21,23*,32,23,96782,tzinfo=zoneinfo.ZoneInfo(key
='UTC'))
```

在不同时区显示 datetime 实例：

```py
>>> dutc.astimezone(ZoneInfo("Asia/Katmandu")) *`# offset +5h 45m`*
```

```py
datetime.datetime(2021,10,*22,5*,17,23,96782,tzinfo=zoneinfo.ZoneInfo(key
='Asia/Katmandu')) 
```

获取本地时区：

```py
>>> tz_local=datetime.now().astimezone().tzinfo
>>> tz_local
```

```py
datetime.timezone(datetime.timedelta(days=-1, seconds=61200), 'Pacific
Daylight Time')
```

将 UTC datetime 实例转换回本地时区：

```py
>>> dt_loc=dutc.astimezone(tz_local)
>>> dt_loc
```

```py
datetime.datetime(2021, 10, 21, 16, 32, 23, 96782, tzinfo=datetime.time
(datetime.timedelta(days=-1, seconds=61200), 'Pacific Daylight Time'))
```

```py
>>> d==dt_local
```

```py
True
```

并获取所有可用时区的排序列表：

```py
>>> tz_list=zoneinfo.available_timezones()
>>> sorted(tz_list)[0],sorted(tz_list)[-1]
```

```py
('Africa/Abidjan', 'Zulu')
```

# 始终在内部使用 UTC 时区

规避时区的陷阱和问题的最佳方法是始终在内部使用 UTC 时区，在输入时从其他时区转换，并仅在显示目的使用 datetime.astimezone。

即使您的应用程序仅在自己的位置运行，并且永远不打算使用其他时区的时间数据，也适用这个技巧。如果您的应用程序连续运行几天或几周，并且为您的系统配置的时区遵循夏令时，如果不在 UTC 内部工作，您将会遇到与时区相关的问题。

# dateutil 模块

第三方包[dateutil](https://oreil.ly/KKEf6)（您可以通过 **pip install python-dateutil** 安装）提供了许多操作日期的模块。表格 13-10 列出了它提供的主要模块，除了用于时区相关操作（现在最好使用 zoneinfo，在前一节中讨论）的模块。

表格 13-10\. dateutil 模块

| easter | easter.easter(*year*) 返回给定 *year* 的复活节 datetime.date 对象。例如：

```py
>>> `from` dateutil `import` easter
>>> print(easter.easter(2023))
```

```py
2023-04-09
```

|

| parser | parser.parse(*s*) 返回由字符串 *s* 表示的 datetime.datetime 对象，具有非常宽松（或“模糊”）的解析规则。例如：

```py
>>> `from` dateutil `import` parser
>>> print(parser.parse('Saturday, January 28,'
                       ' 2006, at 11:15pm'))
```

```py
2006-01-28 23:15:00
```

|

| relativedelta | relativedelta.relativedelta(...) 提供了一种简便的方法，用于查找“下个星期一”、“去年”等。dateutil 的[文档](https://oreil.ly/1zJqi)详细解释了定义 relativedelta 实例行为复杂性规则。 |
| --- | --- |
| rrule | rrule.rrule(*freq*, ...) 实现了[RFC 2445](https://oreil.ly/Xs_NN)（也称为 iCalendar RFC），完整呈现其超过 140 页的荣耀。rrule 允许您处理重复事件，提供了诸如 after、before、between 和 count 等方法。 |

详细信息请查看[dateutil 模块的文档](https://oreil.ly/dmYej)，了解其丰富功能。

# `sched` 模块

`sched` 模块实现了事件调度程序，让您可以轻松处理在“真实”或“模拟”时间尺度上安排的事件。这个事件调度程序在单线程和多线程环境中使用都是安全的。sched 提供了一个调度程序类，它接受两个可选参数，timefunc 和 delayfunc。

| scheduler | **class** scheduler(timefunc=time.monotonic, delayfunc=time.sleep) 可选参数 timefunc 必须是可调用的，没有参数以获取当前时间时刻（以任何度量单位）；例如，您可以传递 time.time。可选参数 delayfunc 是可调用的，具有一个参数（时间持续时间，与 timefunc 相同单位），以延迟当前线程的该时间。调度器在每个事件之后调用 delayfunc(0) 给其他线程一个机会；这与 time.sleep 兼容。通过接受函数作为参数，调度器可以让您使用适合应用程序需要的任何“模拟时间”或“伪时间”^(a)。

如果单调时间（即使系统时钟在调用之间向后调整也无法倒退的时间，例如由于闰秒导致）对您的应用程序至关重要，请为您的调度器使用默认的 time.monotonic。

| ^(a) [依赖注入设计模式](https://oreil.ly/F8W_Z) 的一个很好的示例，用于与测试无关的目的。 |
| --- |

调度器实例 *s* 提供了 表 13-11 中详细描述的方法。

表 13-11\. 调度器实例 s 的方法

| cancel | *s*.cancel(*event_token*) 从 *s* 的队列中移除一个事件。*event_token* 必须是对 *s*.enter 或 *s*.enterabs 的先前调用的结果，并且事件尚未发生；否则，cancel 将引发 RuntimeError。 |
| --- | --- |
| empty | *s*.empty() 当 *s* 的队列当前为空时返回 **True**；否则返回 **False**。 |

| enter | *s*.enter(*delay*, *priority*, *func*, argument=(), kwargs={}) 类似于 enterabs，不同之处在于 *delay* 是相对时间（从当前时刻正向的正差），而 enterabs 的参数 *when* 是绝对时间（未来时刻）。要为 *重复* 执行安排事件，请使用一个小的包装函数；例如：

```py
`def` enter_repeat(s, first_delay, period, priority,
        func, args):
    `def` repeating_wrapper():
        s.enter(period, priority,
                repeating_wrapper, ())
        func(*args)
    s.enter(first_delay, priority,
        repeating_wrapper, args)
```

|

| enterabs | *s*.enterabs(*when*, *priority*, *func*, argument=(), kwargs={}) 在时间 *when* 安排一个未来事件（回调 *func*(*args*, *kwargs*)）。*when* 使用 *s* 的时间函数使用的单位。如果为同一时间安排了几个事件，*s* 将按 *priority* 的增加顺序执行它们。enterabs 返回一个事件令牌 *t*，您可以稍后将其传递给 *s*.cancel 来取消此事件。 |
| --- | --- |
| run | *s*.run(blocking=**True**) 运行已安排的事件。如果 blocking 为**True**，*s*.run 会循环直到*s*.empty 返回**True**，使用*s*初始化时传递的 delayfunc 来等待每个已安排的事件。如果 blocking 为**False**，执行任何即将到期的事件，然后返回下一个事件的截止时间（如果有的话）。当回调函数*func*引发异常时，*s*会传播它，但*s*保持自己的状态，按顺序执行已安排的事件。如果回调函数*func*运行时间超过下一个已安排事件之前的可用时间，则*s*会落后但会继续按顺序执行已安排的事件，不会丢弃任何事件。如果不再对某个事件感兴趣，调用*s*.cancel 显式地丢弃事件。 |

# 日历模块

日历模块提供与日历相关的函数，包括打印给定月份或年份的文本日历的函数。默认情况下，calendar 将星期一作为一周的第一天，星期日作为最后一天。要更改此设置，请调用 calendar.setfirstweekday。calendar 处理模块时间范围内的年份，通常为 1970 到 2038（至少）。

日历模块提供了表 13-12 中列出的函数。

表 13-12\. 日历模块的函数

| calendar | calendar(*year,* w=2, li=1, c=6) 返回一个多行字符串，其中包含*year*年的日历，以每个日期间隔 c 个空格分隔成三列。w 是每个日期的字符宽度；每行的长度为 21*w+18+2*c。li 是每周的行数。 |
| --- | --- |
| firstweekday | firstweekday() 返回每周起始的当前设置的工作日。默认情况下，当导入 calendar 时，这是 0（表示星期一）。 |
| isleap | isleap(*year*) 如果*year*是闰年则返回**True**；否则返回**False**。 |
| leapdays | leapdays(*y1*, *y2*) 返回范围(*y1*, *y2*)内的闰年天数总计（注意，这意味着*y2*是不包括的）。 |
| month | month(*year*, *month*, w=2, li=1) 返回一个多行字符串，其中包含*year*年*month*月的日历，每周一行加上两个标题行。w 是每个日期的字符宽度；每行的长度为 7*w+6。li 是每周的行数。 |
| mo⁠n⁠t⁠h​c⁠a⁠l⁠e⁠n⁠d⁠a⁠r | monthcalendar(*year*, *month*) 返回一个整数列表的列表。每个子列表表示一周。年*year*月*month*之外的天数设为 0；该月内的天数设为它们的日期，从 1 开始。 |
| monthrange | monthrange(*year*, *month*) 返回两个整数。第一个整数是*year*年*month*月第一天的工作日代码；第二个整数是该月的天数。工作日代码为 0（星期一）到 6（星期日）；月份编号为 1 到 12。 |
| prcal | prcal(*year*, w=2, li=1, c=6) 类似于 print(calendar.calendar(*year*, *w*, *li*, *c*))。 |
| prmonth | prmonth(*year*, *month*, w=2, li=1) 类似于 print(calendar.month(*year*, *month*, *w*, *li*))。 |
| setfirstweekday | setfirstweekday(*weekday*) 设置每周的第一天为星期代码 *weekday*。星期代码为从 0（星期一）到 6（星期日）。calendar 提供了 MONDAY、TUESDAY、WEDNESDAY、THURSDAY、FRIDAY、SATURDAY 和 SUNDAY 这些属性，它们的值为整数 0 到 6。在代码中表示工作日时（例如，calendar.FRIDAY 而不是 4），使用这些属性可以使您的代码更清晰和更易读。 |
| timegm | timegm(*tupletime*) 就像 time.mktime 一样：接受时间元组形式的时间点，并将该时间点作为距离纪元的浮点秒数返回。 |
| weekday | weekday(*year*, *month*, *day*) 返回给定日期的星期代码。星期代码为 0（星期一）到 6（星期日）；月份编号为 1（一月）到 12（十二月）。 |

**python -m calendar** 提供了一个有用的命令行界面以访问该模块的功能：运行 **python -m calendar -h** 可以获取简短的帮助信息。

¹ 在旧的 Unix 系统中，1970-01-01 是纪元的开始，而 2038-01-19 是 32 位时间回到纪元的时间点。大多数现代系统现在使用 64 位时间，许多时间方法可以接受从 0001 到 9999 年的年份，但一些方法或旧系统（特别是嵌入式系统）可能仍然有限制。

² time 和 datetime 不考虑闰秒，因为它们的计划未来不可预知。

³ 在 3.9 之前，请使用第三方模块 [pytz](https://oreil.ly/65xFP)。
