# 第二十一章：电子邮件、MIME 和其他网络编码

网络上传输的是字节流，也被网络行话称为*八位字节*。字节当然可以表示文本，通过多种可能的编码方式之一。然而，你希望通过网络发送的内容往往比单纯的文本或字节流有更复杂的结构。多用途互联网邮件扩展（[MIME](https://oreil.ly/dwyZi)）和其他编码标准填补了这一差距，它们规定了如何将结构化数据表示为字节或文本。虽然这些编码通常最初是为电子邮件设计的，但也被用于网络和许多其他网络系统。Python 通过各种库模块支持这些编码，如 base64、quopri 和 uu（在“将二进制数据编码为 ASCII 文本”中介绍），以及 email 包的模块（在下一节中介绍）。这些编码允许我们无缝地创建一个编码中包含附件的消息，避免了许多麻烦的任务。

# MIME 和电子邮件格式处理

email 包处理 MIME 文件（如电子邮件消息）、网络新闻传输协议（NNTP）帖子、HTTP 交互等的解析、生成和操作。Python 标准库还包含其他处理这些工作部分的模块。然而，email 包提供了一种完整和系统的方法来处理这些重要任务。我们建议您使用 email，而不是部分重叠 email 功能的旧模块。尽管名为 email，但它与接收或发送电子邮件无关；对于这些任务，请参见 imaplib、poplib 和 smtplib 模块（在“电子邮件协议”中介绍）。相反，email 处理的是在接收到 MIME 消息之后或在发送之前正确构造它们的任务。

## email 包中的函数

email 包提供了四个工厂函数，从字符串或文件中返回一个类 email.message.Message 的实例 *m*（参见表 21-1）。这些函数依赖于类 email.parser.Parser，但工厂函数更方便、更简单。因此，本书不再深入介绍 email.parser 模块。

表 21-1\. 构建来自字符串或文件的消息对象的电子邮件工厂函数

| m⁠e⁠s⁠s⁠a⁠g⁠e⁠_⁠f⁠r⁠o⁠m⁠_⁠b⁠i⁠n⁠a⁠r⁠y⁠_​f⁠i⁠l⁠e | 使用二进制文件对象 *f*（必须已打开以供读取）的内容解析构建 *m* |
| --- | --- |
| message_from_bytes | 使用字节串 *s* 解析构建 *m* |
| message_from_file | 使用文本文件对象 *f*（必须已打开以供读取）的内容解析构建 *m* |
| message_from_string | 使用字符串 *s* 解析构建 *m* |

## email.message 模块

email.message 模块提供了 Message 类。电子邮件包的所有部分都创建、修改或使用 Message 实例。 Message 的一个实例 *m* 模拟了 MIME 消息，包括 *headers* 和 *payload*（数据内容）。 *m* 是一个映射，以头部名称为键，以头部值字符串为值。

要创建一个最初为空的 *m*，请不带参数调用 Message。更常见的情况是，通过 Table 21-1 中的工厂函数之一解析来创建 *m*，或者通过 “创建消息” 中涵盖的其他间接方式。 *m* 的有效载荷可以是字符串、另一个 Message 实例或者 *多部分消息*（一组递归嵌套的其他 Message 实例）。

你可以在构建的电子邮件消息上设置任意头部。几个互联网 RFC（请求评论）指定了各种目的的头部。主要适用的 RFC 是 [RFC 2822](https://oreil.ly/xyfF_)；你可以在非规范性的 [RFC 2076](https://oreil.ly/IpSCO) 中找到关于头部的许多其他 RFC 的摘要。

为了使 *m* 更方便，作为映射的语义与字典的语义不同。 *m* 的键大小写不敏感。 *m* 保持您添加的顺序的头部，方法 keys、values 和 items 返回按照该顺序排列的头部列表（而不是视图！）。 *m* 可以有多个名为 *key* 的头部：*m*[*key*] 返回任意一个这样的头部（或者头部缺失时返回 **None**），**del** *m*[*key*] 删除所有这样的头部（如果头部缺失则不会报错）。

要获取所有具有特定名称的头部的列表，请调用 *m*.get_all(*key*)。 len(*m*) 返回总头部数，计算重复项，而不仅仅是不同头部名称的数量。当没有名为 *key** 的头部时，*m*[*key*] 返回 **None** 并且不会引发 KeyError（即它的行为类似于 *m*.get(*key*): **del** *m*[*key*] 在这种情况下不起作用，而 *m*.get_all(*key*) 返回 **None**）。您可以直接在 *m* 上循环：这就像在 *m*.keys() 上循环一样。

Message 的一个实例 *m* 提供了各种处理 *m* 的头部和有效载荷的属性和方法，列在 Table 21-2 中。

表 21-2\. Message 实例 *m* 的属性和方法

| add_header | *m*.add_header(_*name*, *_value*, ***_params*) 类似于 *m*[_*name*]=_*value*，但您还可以作为命名参数提供头部参数。对于每个命名参数 *pname*=*pvalue*，add_header 将 *pname* 中的任何下划线更改为破折号，然后将一个形式为的字符串附加到头部的值：

; *pname*="*pvalue*"

当 *pvalue* 为 **None** 时，add_header 仅附加形式为的字符串：

; *pname*

当参数的值包含非 ASCII 字符时，将其指定为一个三项元组，(*CHARSET*, *LANGUAGE*, *VALUE*)。*CHARSET* 指定用于值的编码。*LANGUAGE* 通常为 **None** 或 ''，但可以根据 [RFC 2231](https://oreil.ly/FKtQA) 设置任何语言值。*VALUE* 是包含非 ASCII 字符的字符串值。

| as_string | *m*.as_string(unixfrom=**False**) 返回整个消息作为字符串。当 unixfrom 为 true 时，还包括第一行，通常以 'From ' 开头，称为消息的 *envelope header*。 |
| --- | --- |
| attach | *m*.attach(*payload*) 将 *payload*，即消息，添加到 *m* 的载荷中。当 *m* 的载荷为 **None** 时，*m* 的载荷现在是单个项目列表 [*payload*]。当 *m* 的载荷为消息列表时，将 *payload* 追加到列表中。当 *m* 的载荷为其他任何内容时，*m*.attach(*payload*) 引发 MultipartConversionError。 |
| epilogue | 属性 *m*.epilogue 可以是 **None**，或者是一个字符串，在最后一个边界线之后成为消息字符串形式的一部分。邮件程序通常不显示此文本。epilogue 是 *m* 的正常属性：在处理任何 *m* 时，您的程序可以访问它，并在构建或修改 *m* 时将其绑定。 |
| get_all | *m*.get_all(*name*, default=**None**) 返回一个列表，其中包含按照添加到 *m* 的顺序命名为 *name* 的所有头部的所有值。当 *m* 没有名为 *name* 的头部时，get_all 返回 default。 |
| get_boundary | *m*.get_boundary(default=**None**) 返回 *m* 的 Content-Type 头部的 boundary 参数的字符串值。当 *m* 没有 Content-Type 头部或头部没有 boundary 参数时，get_boundary 返回 default。 |
| get_charsets | *m*.get_charsets(default=**None**) 返回参数 charset 在 *m* 的 Content-Type 头部中的字符串值列表 *L*。当 *m* 是多部分时，*L* 每个部分有一项；否则，*L* 的长度为 1。对于没有 Content-Type 头部、没有 charset 参数或者主类型与 'text' 不同的部分，*L* 中对应的项目是 default。 |
| g⁠e⁠t⁠_⁠c⁠o⁠n⁠t⁠e⁠n⁠t⁠_​m⁠a⁠i⁠n⁠t⁠y⁠p⁠e | *m*.get_content_maintype(default=**None**) 返回 *m* 的主内容类型：从头部 Content-Type 中取出的小写字符串 '*maintype*'。例如，当 Content-Type 是 'Text/Html' 时，get_content_maintype 返回 'text'。当 *m* 没有 Content-Type 头部时，get_content_maintype 返回 default。 |
| g⁠e⁠t⁠_⁠c⁠o⁠n⁠t⁠e⁠n⁠t⁠_​s⁠u⁠b⁠t⁠y⁠p⁠e | *m*.get_content_subtype(default=**None**) 返回 *m* 的内容子类型：从头部 Content-Type 中取出的小写字符串 '*subtype*'。例如，当 Content-Type 是 'Text/Html' 时，get_content_subtype 返回 'html'。当 *m* 没有 Content-Type 头部时，get_content_subtype 返回 default。 |
| g⁠e⁠t⁠_⁠c⁠o⁠n⁠t⁠e⁠n⁠t⁠_​t⁠y⁠p⁠e | *m*.get_content_type(default=**None**) 返回 *m* 的内容类型：从头部 Content-Type 中取得一个小写字符串 '*maintype/subtype*'。例如，当 Content-Type 为 'Text/Html' 时，get_content_type 返回 'text/html'。当 *m* 没有 Content-Type 头部时，get_content_type 返回 default。 |
| get_filename | *m*.get_filename(default=**None**) 返回 *m* 的 Content-Disposition 头部的 filename 参数的字符串值。当 *m* 没有 Content-Disposition 头部，或头部没有 filename 参数时，get_filename 返回 default。 |
| get_param | *m*.get_param(*param*, *d*efault=**None**, header='Content-Type') 返回 *m* 的头部 header 中参数 *param* 的字符串值。对于仅由名称指定（没有值）的参数，返回 ''。当 *m* 没有头部 header，或头部没有名为 *param* 的参数时，get_param 返回 default。 |
| get_params | *m*.get_params(default=**None**, header='Content-Type') 返回 *m* 的头部 header 的参数，一个由字符串对组成的列表，每个参数给出其名称和值。对于仅由名称指定（没有值）的参数，使用 '' 作为值。当 *m* 没有头部 header 时，get_params 返回 default。 |

| get_payload | *m*.get_payload(i=**None**, decode=**False**) 返回 *m* 的载荷。当 *m*.is_multipart 为 **False** 时，i 必须为 **None**，*m*.get_payload 返回 *m* 的整个载荷，一个字符串或消息实例。如果 decode 为 true，并且头部 Content-Transfer-Encoding 的值为 'quoted-printable' 或 'base64'，*m*.get_payload 还会对载荷进行解码。如果 decode 为 false，或头部 Content-Transfer-Encoding 缺失或具有其他值，*m*.get_payload 返回未更改的载荷。

当 *m*.is_multipart 为 **True** 时，decode 必须为 false。当 i 为 **None** 时，*m*.get_payload 返回 *m* 的载荷作为列表。否则，*m*.get_payload(*i*) 返回载荷的第 i 项，如果 *i* < 0 或 i 太大，则引发 TypeError。

| get_unixfrom | *m*.get_unixfrom() 返回 *m* 的信封头字符串，当 *m* 没有信封头时返回 **None**。 |
| --- | --- |
| is_multipart | *m*.is_multipart() 当 *m* 的载荷为列表时返回 **True**；否则返回 **False**。 |
| preamble | 属性 *m*.preamble 可以是 **None**，或成为消息的字符串形式的一部分，出现在第一个边界线之前。邮件程序仅在不支持多部分消息时显示此文本，因此您可以使用此属性来提醒用户需要使用其他邮件程序来查看。preamble 是 *m* 的普通属性：在处理由任何手段构建的 *m* 时，您的程序可以访问它，并在构建或修改 *m* 时绑定、重新绑定或解绑它。 |
| set_boundary | *m*.set_boundary(*boundary*) 将 *m* 的 Content-Type 头部的 boundary 参数设置为 *boundary**.*。当 *m* 没有 Content-Type 头部时，引发 HeaderParseError。 |
| set_payload | *m*.set_payload(*payload*) 将 *m* 的 payload 设置为 *payload*，它必须是字符串或适合 *m* 的 Content-Type 的 Message 实例列表。 |
| set_unixfrom | *m*.set_unixfrom(*unixfrom*) 设置 *m* 的信封头字符串为 *unixfrom*。*unixfrom* 是整个信封头行，包括前导的 'From ' 但不包括尾部的 '\n'。 |
| walk | *m*.walk() 返回一个迭代器，用于遍历 *m* 的所有部分和子部分，深度优先（参见 “递归”）。 |

## email.Generator 模块

email.Generator 模块提供了 Generator 类，您可以使用它来生成消息 *m* 的文本形式。*m*.as_string() 和 str(*m*) 或许已经足够，但 Generator 提供了更多的灵活性。使用必需参数 *outfp* 和两个可选参数实例化 Generator 类：

| Generator | **class** Generator(*outfp*, mangle_from_=**False**, maxheaderlen=78) *outfp* 是一个文件或类文件对象，供应写入方法。当 mangle_from_ 为真时，*g* 会在 payload 中以 'From ' 开头的任何行前添加大于号 (>)，以便更容易解析消息的文本形式。*g* 将每个标题行在分号处包装成不超过 maxheaderlen 字符的物理行。要使用 *g*，调用 *g*.flatten；例如：

```py
*`g`*.flatten(*`m`*, unixfrom=`False`)
```

这将 *m* 以文本形式发射到 *outfp*，类似于（但比以下代码消耗更少的内存）：

```py
*`outfp`*.write(*`m`*.as_string(*`unixfrom`*))
```

. |

## 创建消息

email.mime 子包提供各种模块，每个模块都有一个名为模块的 Message 子类。模块的名称为小写（例如，email.mime.text），而类名为混合大小写。这些类列在 Table 21-3 中，帮助您创建不同 MIME 类型的 Message 实例。

Table 21-3\. email.mime 提供的类

| MIMEAudio | **class** MIMEAudio(_*audiodata*, _subtype=**None**, _encoder=**None**, **_*params*) 创建主类型为 'audio' 的 MIME 消息对象。*_audiodata* 是音频数据的字节串，用于打包为 'audio/_subtype' MIME 类型的消息。当 _subtype 为 **None** 时，*_audiodata* 必须可被标准 Python 库模块 sndhdr 解析以确定子类型；否则，MIMEAudio 会引发 TypeError。3.11+ 由于 sndhdr 已被弃用，您应该始终指定 _subtype。当 _encoder 为 **None** 时，MIMEAudio 会将数据编码为 Base64，这通常是最佳的。否则，_encoder 必须是可调用的，带有一个参数 *m*，即正在构建的消息；_encoder 必须调用 *m*.get_payload 获取载荷，对载荷进行编码，通过调用 *m*.set_payload 将编码形式放回，并设置 *m* 的 Content-Transfer-Encoding 头。MIMEAudio 将 *_params* 字典的命名参数名和值传递给 *m*.add_header 以构造 *m* 的 Content-Type 头。 |
| --- | --- |

| MIMEBase | **class** MIMEBase(_*maintype*, _*subtype*, **_*params*) MIME 类的基类，扩展自 Message。实例化：

```py
*`m`* = MIMEBase(*`mainsub`*, ***`params`*)
```

等同于更长且稍微不太方便的习语：

```py
*`m`* = Message()
*`m`*.add_header('Content-Type', f'{*`main`*}/{*`sub`*}', 
             ***`params`*)
*`m`*.add_header('Mime-Version', '1.0')
```

|

| MIMEImage | **class** MIMEImage(_*imagedata*, _subtype=**None**, _encoder=**None**, ***_params*) 类似于 MIMEAudio，但主类型为 'image'；使用标准 Python 模块 imghdr 来确定子类型（如果需要）。3.11+ 由于 imghdr 已被弃用，因此应始终指定 _subtype。 |
| --- | --- |
| MIMEMessage | **class** MIMEMessage(*msg*, _subtype='rfc822') 将 *msg*（必须是 Message 的一个实例（或子类））打包为 MIME 类型为 'message/_subtype' 的消息的有效载荷。 |
| MIMEText | **class** MIMEText(_*text*, _subtype='plain', _charset='us-ascii', _encoder=**None**) 将文本字符串 *_text* 打包为 MIME 类型为 'text/_subtype' 的消息的有效载荷，并使用给定的 _charset。当 _encoder 为 **None** 时，MIMEText 不对文本进行编码，这通常是最佳选择。否则，_encoder 必须是可调用的，带有一个参数 *m*，即正在构造的消息；然后，_encoder 必须调用 *m*.get_payload 获取有效载荷，对有效载荷进行编码，通过调用 *m*.set_payload 将编码形式放回，然后适当设置 *m* 的 Content-Transfer-Encoding 标题。 |

## email.encoders 模块

email.encoders 模块提供了一些函数，这些函数以一个 *nonmultipart* 消息 *m* 作为它们唯一的参数，对 *m* 的有效载荷进行编码，并适当设置 *m* 的标题。这些函数列在表 21-4 中。

表 21-4\. email.encoders 模块的函数

| encode_base64 | encode_base64(*m*) 使用 Base64 编码，通常对任意二进制数据最优（参见“The base64 Module”）。 |
| --- | --- |
| encode_noop | encode_noop(*m*) 不对 *m* 的有效载荷和标题进行任何操作。 |
| encode_quopri | encode_quopri(*m*) 使用 Quoted Printable 编码，通常对几乎但不完全是 ASCII 的文本最优（参见“The quopri Module”）。 |
| encode_7or8bit | encode_7or8bit(*m*) 不对 *m* 的有效载荷进行任何操作，但在 *m* 的有效载荷的任何字节具有高位设置时，将标题 Content-Transfer-Encoding 设置为 '8bit'；否则，将其设置为 '7bit'。 |

## email.utils 模块

email.utils 模块提供了几个用于电子邮件处理的函数，这些函数在表 21-5 中列出。

表 21-5\. email.utils 模块的函数

| formataddr | formataddr(*pair*) 接受一对字符串（*realname*, *email_address*），并返回一个字符串 *s*，该字符串可插入到标题字段（如 To 和 Cc）中。当 *realname* 为假（例如空字符串 ''）时，formataddr 返回 *email_address*。 |
| --- | --- |
| formatdate | formatdate(timeval=**None**, localtime=**False**) 返回一个按照 RFC 2822 指定格式的时间瞬间的字符串。timeval 是自纪元以来的秒数。当 timeval 为 **None** 时，formatdate 使用当前时间。当 localtime 为 **True** 时，formatdate 使用本地时区；否则，它使用 UTC。 |
| getaddresses | getaddresses(*L*) 解析 *L* 的每个项目，*L* 是地址字符串的列表，如标题字段 To 和 Cc 中使用的，返回字符串对的列表 (*name*, *address*)。当 getaddresses 无法将 *L* 的项目解析为电子邮件地址时，它将 ('', '') 设置为列表中相应的项目。 |
| mktime_tz | mktime_tz(*t*) 返回一个浮点数，表示自纪元以来的秒数（UTC 时间），对应于 *t* 所指示的时刻。*t* 是一个包含 10 项的元组。*t* 的前九项与模块 time 中使用的格式相同，详见“时间模块”。*t*[-1] 是一个时间偏移量，单位为秒，相对于 UTC（与 time.timezone 的相反符号，由 RFC 2822 指定）。当 *t*[-1] 为 **None** 时，mktime_tz 使用本地时区。 |
| parseaddr | parseaddr(*s*) 解析字符串 *s*，其中包含像 To 和 Cc 这样的标题字段中通常指定的地址，并返回一个字符串对 (*realname*, *address*)。当 parseaddr 无法将 *s* 解析为地址时，它返回 ('', '')。 |
| parsedate | parsedate(*s*) 根据 RFC 2822 中的规则解析字符串 *s*，并返回一个包含九项的元组 *t*，如模块 time 中使用的（项 *t*[-3:] 无意义）。parsedate 还尝试解析一些通常遇到的邮件客户端使用的 RFC 2822 的错误变体。当 parsedate 无法解析 *s* 时，它返回 None。 |
| parsedate_tz | parsedate_tz(*s*) 类似于 parsedate，但返回一个包含 10 项的元组 *t*，其中 *t*[-1] 是 *s* 的时区，单位为秒，与 mktime_tz 接受的参数一样，但符号与 time.timezone 相反，如 RFC 2822 所指定。*t*[-4:-1] 项无意义。当 *s* 没有时区时，*t*[-1] 为 **None**。 |
| quote | quote(*s*) 返回字符串 *s* 的副本，其中每个双引号 (") 都变为 '\"'，每个现有的反斜杠都重复。 |
| unquote | unquote(*s*) 返回字符串 *s* 的副本，其中移除了前导和尾随的双引号 (") 和尖括号 (<>)，如果它们包围着 *s* 的其余部分。 |

## 邮件包的示例用法

邮件包不仅帮助您阅读和撰写邮件和类似邮件的消息（但不涉及接收和传输此类消息：这些任务属于单独的模块，在第十九章中涵盖）。以下是如何使用邮件来读取可能是多部分消息并将每个部分解包到给定目录中的文件的示例：

```py
`import` pathlib, email
`def` unpack_mail(mail_file, dest_dir):
    *`'''Given file object mail_file, open for reading, and dest_dir,`*
       *`a string that is a path to an existing, writable directory,`*
       *`unpack each part of the mail message from mail_file to a`*
       *`file within dest_dir.`*
    *`'''`*
    dest_dir_path = pathlib.Path(dest_dir)
    `with` mail_file:
        msg = email.message_from_file(mail_file)
    `for` part_number, part `in` enumerate(msg.walk()):
        `if` part.get_content_maintype() == 'multipart':
            *`# we get each specific part later in the loop,`*
            *`# so, nothing to do for the 'multipart' itself`*
 `continue`
        dest = part.get_filename()
        `if` dest `is` `None`: dest = part.get_param('name')
        `if` dest `is` `None`: dest = f'part-{part_number}'
        *`# in real life, make sure that dest is a reasonable filename`*
        *`# for your OS; otherwise, mangle that name until it is`*
        part_payload = part.get_payload(decode=`True`)
        (dest_dir_path / dest).write_text(part_payload)
```

这里有一个执行大致相反任务的示例，将直接位于给定源目录下的所有文件打包成一个适合邮件发送的单个文件：

```py
`def` pack_mail(source_dir, **headers):
     *`'''Given source_dir, a string that is a path to an existing,`*
        *`readable directory, and arbitrary header name/value pairs`*
        *`passed in as named arguments, packs all the files directly`*
        *`under source_dir (assumed to be plain text files) into a`*
        *`mail message returned as a MIME-formatted string.`*
     *`'''`*
     source_dir_path = pathlib.Path(source_dir)
     msg = email.message.Message()
     `for` name, value `in` headers.items():
         msg[name] = value
     msg['Content-type'] = 'multipart/mixed'
     filepaths = [path for path in source_dir_path.iterdir() 
                  if path.is_file()]
     `for` filepath `in` filepaths:
         m = email.message.Message()
         m.add_header('Content-type', 'text/plain', name=filename)
         m.set_payload(filepath.read_text())
         msg.attach(m)
     `return` msg.as_string()
```

# 将二进制数据编码为 ASCII 文本

几种媒体（例如电子邮件消息）只能包含 ASCII 文本。当您想通过这些媒体传输任意二进制数据时，需要将数据编码为 ASCII 文本字符串。Python 标准库提供支持名为 Base64、Quoted Printable 和 Unix-to-Unix 的标准编码的模块，下面将对这些进行描述。

## 模块 base64

base64 模块支持 [RFC 3548](https://oreil.ly/Cbhkl) 中指定的编码，包括 Base16、Base32 和 Base64。这些编码是一种将任意二进制数据表示为 ASCII 文本的紧凑方式，没有尝试生成可读的结果。base64 提供 10 个函数：6 个用于 Base64，以及 2 个用于 Base32 和 Base16。六个 Base64 函数列在 表 21-6 中。

表 21-6\. base64 模块的 Base64 函数

| b64decode | b64decode(*s*, altchars=**None**, validate=**False**) 解码 B64 编码的字节串 *s*，并返回解码后的字节串。altchars，如果不为 **None**，必须是至少两个字符的字节串（多余字符将被忽略），指定要使用的两个非标准字符，而不是 + 和 /（可能对解码 URL 安全或文件系统安全的 B64 编码字符串有用）。当 validate 为 **True** 时，如果 *s* 包含任何无效的 B64 编码字符串（默认情况下，这些字节只是被忽略和跳过），则调用会引发异常。如果 *s* 没有按照 Base64 标准正确填充，则调用会引发异常。 |
| --- | --- |
| b64encode | b64encode(*s*, altchars=**None**) 编码字节串 *s*，并返回具有相应 B64 编码数据的字节串。altchars，如果不为 **None**，必须是至少两个字符的字节串（多余字符将被忽略），指定要使用的两个非标准字符，而不是 + 和 /（可能对制作 URL 安全或文件系统安全的 B64 编码字符串有用）。 |
| standa⁠r⁠d⁠_​b⁠6⁠4⁠decode | standard_b64decode(*s*) 类似于 b64decode(*s*)。 |
| standa⁠r⁠d⁠_​b⁠6⁠4⁠encode | standard_b64encode(*s*) 类似于 b64encode(*s*)。 |
| urlsa⁠f⁠e⁠_​b⁠6⁠4⁠decode | urlsafe_b64decode(*s*) 类似于 b64decode(*s*, '-_')。 |
| urlsa⁠f⁠e⁠_​b⁠6⁠4⁠encode | urlsafe_b64encode(*s*) 类似于 b64encode(*s*, '-_')。 |

四个 Base16 和 Base32 函数列在 表 21-7 中。

表 21-7\. base64 模块的 Base16 和 Base32 函数

| b16decode | b16decode(*s*, casefold=**False**) 解码 B16 编码的字节串 *s*，并返回解码后的字节串。当 casefold 为 **True** 时，*s* 中的小写字符将被视为其大写等价字符；默认情况下，如果存在小写字符，调用会引发异常。 |
| --- | --- |
| b16encode | b16encode(*s*) 编码字节串 *s*，并返回具有相应 B16 编码数据的字节串。 |
| b32decode | b32decode(*s*, casefold=**False**, map01=**None**) 解码 B32 编码的字节串 *s*，返回解码后的字节串。当 casefold 为 **True** 时，将 *s* 中的小写字符视为它们的大写形式；默认情况下，如果 *s* 中存在小写字符，调用将引发异常。当 map01 为 **None** 时，输入中不允许字符 0 和 1；当 map01 不为 **None** 时，必须是一个指定 1 映射到的单字符字节串，即小写 'l' 或大写 'L'；0 总是映射到大写 'O'。 |
| b32encode | b32encode(*s*) 编码字节串 *s*，返回相应的 B32 编码数据的字节串。 |

该模块还提供了用于编码和解码非标准但流行的编码 Base85 和 Ascii85 的函数，这些编码虽然未在 RFC 中规范，也不互通，但使用更大的字母表对编码的字节串进行了空间节省，节省率达 15%。详细信息请参阅[在线文档](https://oreil.ly/rndpn)中的相关函数。

## quopri 模块

quopri 模块支持 RFC 1521 指定的 *Quoted Printable*（QP）编码。QP 可将任何二进制数据表示为 ASCII 文本，但主要用于大部分为文本且带有高位设置（即 ASCII 范围之外的字符）的数据。对于这样的数据，QP 生成的结果既紧凑又易读。quopri 模块提供了四个函数，列在表 21-8 中。

表 21-8\. quopri 模块的功能

| decode | decode(*infile*, *outfile*, header=**False**) 通过调用 *infile*.readline 读取类似文件的二进制对象 *infile*，直到文件末尾（即直到 *infile*.readline 返回空字符串），解码读取的 QP 编码的 ASCII 文本，并将结果写入类似文件的二进制对象 *outfile**.* 当 header 为 true 时，decode 也会将 _（下划线）转换为空格（根据 RFC 1522）。 |
| --- | --- |
| decodestring | decodestring(*s*, header=**False**) 解码 QP 编码的 ASCII 文本字节串 *s*，返回解码后的字节串。当 header 为 true 时，decodestring 也会将 _（下划线）转换为空格。 |
| encode | encode(*infile*, *outfile*, *quotetabs*, header=**False**) 通过调用 *infile*.readline 读取类似文件的二进制对象 *infile*，直到文件末尾（即直到 *infile*.readline 返回空字符串），使用 QP 编码读取的数据，并将编码的 ASCII 文本写入类似文件的二进制对象 *outfile**.* 当 *quotetabs* 为 true 时，encode 也会编码空格和制表符。当 header 为 true 时，encode 将空格编码为 _（下划线）。 |
| encodestring | encodestring(*s*, quotetabs=**False**, header=**False**) 编码包含任意字节的字节串 *s*，返回包含 QP 编码的 ASCII 文本的字节串。当 quotetabs 为 true 时，encodestring 也会编码空格和制表符。当 header 为 true 时，encodestring 将空格编码为 _（下划线）。 |

## uu 模块

支持经典*Unix-to-Unix*（UU）编码的**uu**模块¹受到 Unix 程序*uuencode*和*uudecode*的启发。 UU 将编码数据以开始行开头，包括正在编码的文件的文件名和权限，并以结束行结尾。 因此，UU 编码允许您将编码数据嵌入其他非结构化文本中，而 Base64 编码（在“base64 模块”中讨论）依赖于其他指示编码数据开始和结束的存在。 uu 模块提供了两个函数，列在表 21-9 中。

表 21-9\. uu 模块的函数

| decode | decode(*infile*, outfile=**None**, mode=**None**) 通过调用*infile*.readline 读取类文件对象*infile*，直到文件末尾（即，直到调用*infile*.readline 返回空字符串）或终止行（由任意数量的空白包围的字符串'end'）。 decode 解码读取的 UU 编码文本，并将解码后的数据写入类文件对象 outfile。 当 outfile 为**None**时，decode 会根据 UU 格式的开始行创建文件，并使用 mode 指定的权限位（当 mode 为**None**时，在开始行中指定的权限位）。 在这种情况下，如果文件已经存在，decode 会引发异常。 |
| --- | --- |
| encode | encode(*infile*, *outfile*, name='-', mode=0o666) 通过调用*infile*.read（每次读取 45 字节数据，这是 UU 编码后每行 60 个字符数据的量）读取类文件对象*infile*，直到文件末尾（即，直到调用*infile*.read 返回空字符串）。 它将读取的数据以 UU 编码方式编码，并将编码文本写入类文件对象*outfile*。 encode 还在文本之前写入一个 UU 格式的开始行，并在文本之后写入 UU 格式的结束行。 在开始行中，encode 指定文件名为 name，并指定 mode 为 mode。 |

¹ 在 Python 3.11 中已弃用，将在 Python 3.13 中删除；在线文档建议用户更新现有代码，使用 base64 模块处理数据内容，并使用 MIME 头处理元数据。
