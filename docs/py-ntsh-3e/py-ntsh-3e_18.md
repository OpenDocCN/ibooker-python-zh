# 第十八章 网络基础

*面向连接*协议的工作方式类似于打电话。您请求与特定的*网络端点*建立连接（类似于拨打某人的电话号码），您的对方要么接听要么不接听。如果接听，您可以与他们交谈并听到他们的回答（如果需要可以同时进行），并且您知道没有任何信息丢失。在对话结束时，您都会说再见并挂断电话，因此如果没有发生这种关闭事件，就明显表明出了问题（例如，如果突然听不到对方的声音）。传输控制协议（TCP）是互联网的主要面向连接传输协议，被 Web 浏览器、安全外壳、电子邮件和许多其他应用程序使用。

*无连接*或者*数据报*协议更像是通过发送明信片进行通信。大多数情况下，消息可以传递，但是如果出了问题，你必须准备好应对后果——协议不会通知你消息是否已接收，而且消息可能会无序到达。对于交换短消息并获取答案，数据报协议的开销比面向连接的协议小，前提是整体服务能够处理偶发的中断。例如，域名服务（DNS）服务器可能无法响应：直到最近，大多数 DNS 通信都是无连接的。用户数据报协议（UDP）是互联网通信的主要无连接传输协议。

如今，安全性变得越来越重要：理解安全通信的基础知识有助于确保您的通信达到所需的安全水平。如果这个摘要让您在没有充分了解问题和风险的情况下尝试实现这样的技术，那么它将发挥出有价值的作用。

所有网络接口之间的通信都是通过字节串交换的。要传输文本或者其他大多数信息，发送方必须将其编码为字节，接收方必须解码。在本章中，我们将讨论单个发送方和单个接收方的情况。

# 伯克利套接字接口

如今大多数网络使用*套接字*。套接字提供了独立端点之间的管道访问，使用*传输层协议*在这些端点之间传输信息。套接字的概念足够通用，使得端点可以位于同一台计算机上，也可以位于联网的不同计算机上，无论是在本地还是通过广域网连接。

如今最常用的传输层是 UDP（用于无连接的网络）和 TCP（用于面向连接的网络）；每种传输层在通用 Internet 协议（IP）网络层上运行。这些协议堆栈以及运行在其上的许多应用协议总称为*TCP/IP*。Gordon McMillan 的（有些陈旧但仍然有效的）[*Socket 编程指南*](https://oreil.ly/9Y5pc)提供了很好的介绍。

两种最常见的套接字家族是基于 TCP/IP 通信的*互联网套接字*（提供现代 IPv6 和更传统的 IPv4 两种版本）和*Unix 套接字*，尽管还有其他家族可用。互联网套接字允许任何两台可以交换 IP 数据报的计算机进行通信；Unix 套接字只能在同一 Unix 机器上的进程之间通信。

为了支持许多并发的互联网套接字，TCP/IP 协议栈使用由 IP 地址、*端口号*和协议标识的端点。端口号允许协议处理软件在相同 IP 地址上使用相同协议时区分不同的端点。连接的套接字还与一个*远程端点*关联，即连接并能够通信的对方套接字。

大多数 Unix 套接字在 Unix 文件系统中具有名称。在 Linux 平台上，以零字节开头的套接字存在于内核维护的名称池中。例如，这些对于与[chroot-jail 进程](https://oreil.ly/qvgaC)通信非常有用，例如，两个进程之间没有共享文件系统。

互联网套接字和 Unix 套接字均支持无连接和面向连接的网络，因此如果您仔细编写程序，它们可以在任何套接字家族上工作。本书不讨论其他套接字家族的范围，尽管我们应该提到*原始套接字*，它是互联网套接字家族的一个子类型，允许直接发送和接收链路层数据包（例如以太网数据包）。这对于一些实验性应用和[数据包嗅探](https://oreil.ly/bmYSI)很有用。

创建互联网套接字后，您可以将特定的端口号与套接字关联（只要该端口号没有被其他套接字使用）。这是许多服务器使用的策略，提供所谓的[*众所周知的端口号*](https://oreil.ly/Y2XeE)，由互联网标准定义为 1-1,023 范围内的端口号。在 Unix 系统上，需要*root*权限才能访问这些端口。典型的客户端不关心使用的端口号，因此通常请求由协议驱动程序分配并保证在主机上唯一的*临时端口*。无需绑定客户端端口。

考虑同一台计算机上的两个进程，每个进程都作为同一个远程服务器的客户端。它们套接字的完整关联具有五个组成部分，（本地 IP 地址、本地端口号、协议、远程 IP 地址、远程端口号）。当数据包到达远程服务器时，目标 IP 地址、源 IP 地址、目标端口号和协议对两个客户端来说是相同的。短暂端口号的唯一性保证了服务器能区分来自两个客户端的流量。这就是 TCP/IP 处理同一对 IP 地址之间的多个会话的方式。¹

## 套接字地址

不同类型的套接字使用不同的地址格式：

+   Unix 套接字地址是命名文件系统中节点的字符串（在 Linux 平台上，是以 b'\0' 开头的字节串，并对应内核表中的名称）。

+   IPv4 套接字地址是 (*地址*, *端口*) 对。第一项是 IPv4 地址，第二项是范围在 1 到 65,535 的端口号。

+   IPv6 套接字地址是四项 (*地址*, *端口*, *流信息*, *作用域 ID*) 元组。在提供地址作为参数时，通常可以省略 *流信息* 和 *作用域 ID*，只要 [地址作用域](https://oreil.ly/RcIfb) 不重要即可。

## 客户端/服务器计算

我们接下来讨论的模式通常称为 *客户端/服务器* 网络，其中 *服务器* 在特定端点上监听来自需要服务的 *客户端* 的流量。我们不涵盖 *点对点* 网络，因为缺少任何中央服务器，必须包含对等方发现的能力。

大多数（虽然并非全部）网络通信是通过客户端/服务器技术进行的。服务器在预定或公布的网络端点处监听传入的流量。在缺少此类输入时，它不做任何操作，只是坐在那里等待来自客户端的输入。连接在面向无连接和面向连接的端点之间的通信有所不同。

在面向无连接的网络中（例如通过 UDP），请求随机到达服务器并立即处理：响应立即发送给请求者。每个请求单独处理，通常不参考先前在两方之间可能发生的任何通信。面向无连接的网络非常适合短期无状态交互，例如 DNS 或网络引导所需的交互。

在面向连接的网络中，客户端与服务器进行初始交换，有效地在两个进程之间建立了网络管道上的连接（有时称为[*虚拟电路*](https://oreil.ly/ePVQo)），在这些进程可以进行通信，直到两者表示愿意结束连接。在这种情况下，服务需要使用并行处理（通过线程、进程或异步机制：参见第十五章）来异步或同时处理每个传入的连接。如果没有并行处理，服务器将无法在早期的连接终止之前处理新的传入连接，因为对套接字方法的调用通常会*阻塞*（即它们会暂停调用它们的线程，直到它们终止或超时）。连接是处理诸如邮件交换、命令行交互或传输 Web 内容等长时间交互的最佳方法，并在使用 TCP 时提供自动错误检测和纠正。

### 无连接的客户端和服务器结构。

无连接服务器的整体逻辑流程如下：

1.  通过调用 socket.socket 创建类型为 socket.SOCK_DGRAM 的套接字。

1.  通过调用套接字的 bind 方法将套接字与服务端点关联。

1.  重复以下步骤*无限期*：

    1.  通过调用套接字的 recvfrom 方法请求来自客户端的传入数据报；此调用会阻塞，直到接收到数据报。

    1.  计算或查找结果。

    1.  通过调用套接字的 sendto 方法将结果发送回客户端。

服务器大部分时间都在步骤 3a 中等待来自客户端的输入。

无连接客户端与服务器的交互如下：

1.  通过调用 socket.socket 创建类型为 socket.SOCK_DGRAM 的套接字。

1.  可选地，通过调用套接字的 bind 方法将套接字与特定端点关联。

1.  通过调用套接字的 sendto 方法向服务器端点发送请求。

1.  通过调用套接字的 recvfrom 方法等待服务器的回复；此调用会阻塞，直到收到响应。必须对此调用应用*超时*，以处理数据报丢失的情况，程序必须重试或中止尝试：无连接套接字不保证传递。

1.  在客户端程序的剩余逻辑中使用结果。

单个客户端程序可以与同一个或多个服务器执行多次交互，具体取决于需要使用的服务。许多这样的交互对应用程序员来说是隐藏在库代码中的。一个典型的例子是将主机名解析为适当的网络地址，通常使用 `gethostbyname` 库函数（在 Python 的 `socket` 模块中实现，稍后讨论）。无连接的交互通常涉及向服务器发送一个数据包并接收一个响应数据包。主要的例外情况涉及*流*协议，如实时传输协议（RTP），² 这些协议通常构建在 UDP 之上，以最小化延迟和延迟：在流式传输中，发送和接收许多数据报。

### 连接导向客户端和服务器结构

连接导向服务器的逻辑流程如下：

1.  通过调用 `socket.socket` 创建 `socket.SOCK_STREAM` 类型的套接字。

1.  通过调用套接字的 `bind` 方法将套接字与适当的服务器端点关联起来。

1.  通过调用套接字的 `listen` 方法开始端点监听连接请求。

1.  无限重复以下步骤*ad infinitum*：

    1.  通过调用套接字的 `accept` 方法等待传入的客户端连接；服务器进程会阻塞，直到收到传入的连接请求。当这样的请求到达时，会创建一个新的套接字对象，其另一个端点是客户端程序。

    1.  创建一个新的控制线程或进程来处理这个特定的连接，将新创建的套接字传递给它；主控线程然后通过回到步骤 4a 来继续。

    1.  在新的控制线程中，使用新套接字的 `recv` 和 `send` 方法与客户端进行交互，分别用于从客户端读取数据和向其发送数据。`recv` 方法会阻塞，直到从客户端接收到数据（或客户端指示希望关闭连接，在这种情况下 `recv` 返回空结果）。当服务器希望关闭连接时，可以通过调用套接字的 `close` 方法来实现，可选择先调用其 `shutdown` 方法。

服务器大部分时间都在步骤 4a 中等待来自客户端的连接请求。

连接导向客户端的总体逻辑如下：

1.  通过调用 `socket.socket` 创建 `socket.SOCK_STREAM` 类型的套接字。

1.  可选地，通过调用套接字的 `bind` 方法将套接字与特定端点关联。

1.  通过调用套接字的 `connect` 方法建立与服务器的连接。

1.  使用套接字的 recv 和 send 方法与服务器进行交互，分别用于从服务器读取数据和向其发送数据。recv 方法会阻塞，直到从服务器接收到数据（或者服务器指示希望关闭连接，在这种情况下，recv 调用会返回空结果）。send 方法仅在网络软件缓冲区有大量数据时才会阻塞，导致通信暂停，直到传输层释放部分缓冲内存。当客户端希望关闭连接时，可以调用套接字的 close 方法，可选地先调用其 shutdown 方法。

面向连接的交互通常比无连接的更复杂。具体来说，确定何时读取和写入数据更加复杂，因为必须解析输入以确定何时传输完毕。在面向连接的网络中使用的更高层协议适应了这种确定性；有时通过在内容中指示数据长度来实现，有时则采用更复杂的方法。

## socket 模块

Python 的 socket 模块通过 socket 接口处理网络。虽然各平台间有些许差异，但该模块隐藏了大部分差异，使得编写可移植的网络应用相对容易。

该模块定义了三个异常类，均为内置异常类 OSError 的子类（见 Table 18-1）。

Table 18-1\. socket 模块异常类

| herror | 用于识别主机名解析错误：例如，socket.gethostbyname 无法将名称转换为网络地址，或者 socket.gethostbyaddr 找不到网络地址对应的主机名。相关的值是一个两元组（*h_errno*，*string*），其中 *h_errno* 是来自操作系统的整数错误号，*string* 是错误的描述。 |
| --- | --- |
| gaierror | 用于识别在 socket.getaddrinfo 或 socket.getnameinfo 中遇到的地址解析错误。 |
| timeout | 当操作超过超时限制时（依据 socket.setdefaulttimeout，可以在每个套接字上覆盖），引发此异常。 |

该模块定义了许多常量。其中最重要的是地址家族（AF_*）和套接字类型（SOCK_*），列在 Table 18-2 中，作为 IntEnum 集合的成员。此外，该模块还定义了许多其他用于设置套接字选项的常量，但文档未对其进行详细定义：要使用它们，您必须熟悉 C 套接字库和系统调用的文档。

Table 18-2\. socket 模块中定义的重要常量

| AF_BLUETOOTH | 用于创建蓝牙地址家族的套接字，用于移动和个人区域网络（PAN）应用中。 |
| --- | --- |
| AF_CAN | 用于创建 Controller Area Network (CAN) 地址家族的套接字，在自动化、汽车和嵌入式设备应用中广泛使用。 |
| AF_INET | 用于创建 IPv4 地址族的套接字。 |
| AF_INET6 | 用于创建 IPv6 地址族的套接字。 |
| AF_UNIX | 用于创建 Unix 地址族的套接字。此常量仅在支持 Unix 套接字的平台上定义。 |
| SOCK_DGRAM | 用于创建无连接套接字，提供尽力而为的消息传递，无连接能力或错误检测。 |
| SOCK_RAW | 用于创建直接访问链路层驱动程序的套接字；通常用于实现较低级别的网络功能。 |
| SOCK_RDM | 用于创建在透明进程间通信（TIPC）协议中使用的可靠连接的无连接消息套接字。 |
| SOCK_SEQPACKET | 用于创建在 TIPC 协议中使用的可靠连接的面向连接的消息套接字。 |
| SOCK_STREAM | 用于创建面向连接的套接字，提供完整的错误检测和修正功能。 |

该模块定义了许多函数来创建套接字、操作地址信息，并辅助标准数据的表示。本书未涵盖所有函数，因为套接字模块的[文档](https://oreil.ly/LU9FI)非常全面；我们只处理编写网络应用程序中必需的部分。

套接字模块包含许多函数，其中大多数仅在特定情况下使用。例如，当网络端点之间进行通信时，端点可能存在架构差异，并以不同方式表示相同的数据，因此存在处理有限数据类型转换的函数，以及从网络中立形式转换的函数。Table 18-3 列出了此模块提供的一些更普遍适用的函数。

Table 18-3\. 套接字模块的有用函数

| getaddrinfo | socket.getaddrinfo(*host*, *port*, family=0, type=0, proto=0, flags=0) 接受*host*和*port*，返回形如(family, type, proto, *canonical_name, socket*)的五元组列表，可用于创建到特定服务的套接字连接。当您传递主机名而不是 IP 地址时，getaddrinfo 返回一个元组列表，每个 IP 地址与名称关联。 |
| --- | --- |
| getdefa⁠u⁠l⁠t​t⁠i⁠m⁠eout | socket.getdefaulttimeout() 返回套接字操作的默认超时值（以秒为单位），如果未设置值则返回**None**。某些函数允许您指定显式超时。 |
| getfqdn | socket.getfqdn([*host*]) 返回与主机名或网络地址关联的完全限定域名（默认情况下是调用它的计算机的域名）。 |
| gethostbyaddr | socket.gethostbyaddr(*ip_address*) 接受包含 IPv4 或 IPv6 地址的字符串，并返回一个形如 (*hostname*, *aliaslist*, *ipaddrlist*) 的三元组。 *hostname* 是 IP 地址的规范名称，*aliaslist* 是一个替代名称列表，*ipaddrlist* 是一个 IPv4 和 IPv6 地址列表。 |
| gethostbyname | socket.gethostbyname(hostname) 返回一个包含与给定主机名关联的 IPv4 地址的字符串。如果使用 IP 地址调用，则返回该地址。此函数不支持 IPv6：请使用 getaddrinfo 获取 IPv6。 |
| getnameinfo | socket.getnameinfo(*sock_addr*, flags=0) 接受套接字地址并返回一个 (*host*, *port*) 对。没有标志*，* *host* 是一个 IP 地址，*port* 是一个整数。 |
| setdefaulttimeout | socket.setdefaulttimeout(*timeout*) 将套接字的默认超时设置为浮点秒值。新创建的套接字按照 *timeout* 值确定的模式运行，如下一节所述。将 *timeout* 作为 **None** 传递以取消随后创建的套接字上的隐式超时使用。 |

## Socket 对象

socket 对象是 Python 中网络通信的主要手段。当 SOCK_STREAM 套接字接受连接时，也会创建一个新的套接字，每个这样的套接字都用于与相应的客户端通信。

# Socket 对象和 with 语句

每个 socket 对象都是上下文管理器：您可以在 **with** 语句中使用任何 socket 对象，以确保在退出语句体时正确终止套接字。有关详细信息，请参见“with 语句和上下文管理器”。

有几种创建 socket 的方式，如下一节所述。根据超时值，套接字可以在三种不同的模式下运行，如表 18-4 所示，可以通过不同的方式设置超时值：

+   通过在创建 socket 时提供超时值作为参数

+   通过调用 socket 对象的 settimeout 方法

+   根据 socket 模块的默认超时值，由 socket.getdefaulttimeout 函数返回

建立每种可能模式的超时值列在 表 18-4 中。

表 18-4\. 超时值及其关联模式

| **None** | 设置 *阻塞* 模式。每个操作都会暂停线程（*阻塞*），直到操作完成，除非操作系统引发异常。 |
| --- | --- |
| 0 | 设置 *非阻塞* 模式。每个操作在无法立即完成或发生错误时引发异常。使用 [selectors 模块](https://oreil.ly/UBypi) 查找操作是否可以立即完成。 |
| >0.0 | 设置 *超时* 模式。每个操作都会阻塞，直到完成，或超时（在这种情况下会引发 socket.timeout 异常），或发生错误。 |

套接字对象表示网络端点。socket 模块提供了多个函数来创建套接字（参见 表 18-5）。

表 18-5\. 套接字创建函数

| cre⁠a⁠t⁠e⁠_​c⁠o⁠n⁠n⁠e⁠c⁠t⁠i⁠o⁠n | create_connection([*address*[, *timeout*[, *source_address*]]]) 创建一个连接到地址（一个（*host*, *port*）对）的 TCP 端点的套接字。*host* 可以是数字网络地址或 DNS 主机名；在后一种情况下，将尝试为 AF_INET 和 AF_INET6（顺序不确定）进行名称解析，然后依次尝试连接返回的每个地址——这是创建既能使用 IPv6 又能使用 IPv4 的客户端程序的便捷方式。

*timeout* 参数（如果提供）指定连接超时时间（单位为秒），从而设置套接字的模式（参见表 18-4）；当参数不存在时，将调用 socket.getdefaulttimeout 函数来确定该值。如果提供 *source_address* 参数，那么它也必须是一个（*host, port*）对，远程套接字将其作为连接端点传递。当 *host* 为 '' 或 *port* 为 0 时，将使用默认的操作系统行为。

| socket | socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=**None**)

创建并返回适当地址族和类型的套接字（默认为 IPv4 上的 TCP 套接字）。子进程不会继承创建的套接字。协议编号 proto 仅在 CAN 套接字中使用。当传递 fileno 参数时，将忽略其他参数：函数返回已关联给定文件描述符的套接字。

| socketpair | socketpair([*family*[, *type*[, *proto*]]]) 返回给定地址族、套接字类型和（仅对于 CAN 套接字）协议的连接对套接字。当未指定 *family* 时，在支持该族的平台上，套接字为 AF_UNIX；否则，它们为 AF_INET。当未指定 *type* 时，默认为 SOCK_STREAM。 |
| --- | --- |

套接字对象 *s* 提供了 表 18-6 中列出的方法。那些涉及连接或需要已连接套接字的方法仅适用于 SOCK_STREAM 套接字，而其他方法适用于 SOCK_STREAM 和 SOCK_DGRAM 套接字。对于带有 *flags* 参数的方法，可用的确切标志集取决于您的特定平台（可用值在 Unix 手册页面的 [recv(2)](https://oreil.ly/boM-c) 和 [send(2)](https://oreil.ly/JAaNO) 中以及 [Windows 文档](https://oreil.ly/90h4R) 中有记录）；如果省略，*flags* 默认为 0。

表 18-6\. 套接字实例 *s* 的方法

| accept | accept() 阻塞直到客户端与 *s* 建立连接（*s* 必须已绑定到一个地址（通过调用 *s*.bind）并设置为侦听状态（通过调用 *s*.listen））。返回一个*新*套接字对象，可用于与连接的另一端点通信。 |
| --- | --- |
| bind | bind(*address*) 将 *s* 绑定到特定地址。*address* 参数的形式取决于套接字的地址族（参见“套接字地址”）。 |
| close | close() 标记套接字为关闭状态。调用 *s*.close 并不一定会立即关闭连接，这取决于是否还有对套接字的其他引用。如果需要立即关闭连接，首先调用 *s*.shutdown 方法。确保套接字及时关闭的最简单方法是在 **with** 语句中使用它，因为套接字是上下文管理器。 |
| connect | connect(*address*) 连接到地址为 *address* 的远程套接字。*address* 参数的形式取决于地址族（参见“套接字地址”）。 |
| detach | detach() 将套接字置于关闭模式，但允许套接字对象用于进一步的连接（通过再次调用 connect）。 |
| dup | dup() 返回套接字的副本，不能被子进程继承。 |
| fileno | fileno() 返回套接字的文件描述符。 |
| getblocking | getblocking() 如果套接字被设置为阻塞模式，则返回**True**，可以通过调用 *s*.setblocking(**True**) 或 *s*.settimeout(**None**) 进行设置。否则，返回**False**。 |
| g⁠e⁠t⁠_​i⁠n⁠h⁠e⁠r⁠i⁠t⁠a⁠b⁠l⁠e | get_inheritable() 当套接字能够被子进程继承时返回**True**。否则，返回**False**。 |
| getpeername | getpeername() 返回此套接字连接的远程端点的地址。 |
| getsockname | getsockname() 返回此套接字正在使用的地址。 |
| gettimeout | gettimeout() 返回与此套接字关联的超时时间。 |
| listen | listen([*backlog*]) 开始监听套接字的关联端点上的流量。如果给定，整数参数 *backlog* 确定操作系统在开始拒绝连接之前允许排队的未接受连接数量。 |
| makefile | makefile(*mode*, buffering=**None**, *, encoding=**None**, newline=**None**) 返回一个文件对象，允许套接字用于类似文件的操作，如读和写。参数类似于内置的 open 函数（参见“使用 open 创建文件对象”）。*mode* 可以是 'r' 或 'w'；对于二进制传输，可以添加 'b'。套接字必须处于阻塞模式；如果设置了超时值，当超时发生时可能会观察到意外的结果。 |
| recv | recv(*bufsiz*[, *flags*]) 接收并返回来自套接字 *s* 的最多 *bufsiz* 字节数据。 |
| recvfrom | recvfrom(*bufsiz*[, *flags*]) 从 *s* 接收最多 *bufsiz* 字节的数据。返回一对（*bytes*，*address*）：*bytes* 是接收到的数据，*address* 是发送数据的对方套接字的地址。 |
| recvfrom_into | recvfrom_into(*buffer*[, *nbytes*[, *flags*]]) 从 *s* 接收最多 *nbytes* 字节的数据，并将其写入给定的 *buffer* 对象中。如果省略 *nbytes* 或为 0，则使用 len(*buffer*)。返回一个二元组 (*nbytes*, *address*)：*nbytes* 是接收的字节数，*address* 是发送数据的对方套接字的地址（**_into* 函数比分配新缓冲区的“普通”函数更快）。 |
| recv_into | recv_into(*buffer*[, *nbytes*[, *flags*]]) 从 *s* 接收最多 *nbytes* 字节的数据，并将其写入给定的 *buffer* 对象中。如果省略 *nbytes* 或为 0，则使用 len(*buffer*)。返回接收的字节数。 |
| recvmsg | recvmsg(*bufsiz*[, *ancbufsiz*[, *flags*]]) 在套接字上接收最多 *bufsiz* 字节的数据和最多 *ancbufsiz* 字节的辅助（“带外”）数据。返回一个四元组 (*data*, *ancdata*, *msg_flags*, *address*)，其中 *data* 是接收的数据，*ancdata* 是表示接收的辅助数据的三元组 (*cmsg_level*, *cmsg_type*, *cmsg_data*) 列表，*msg_flags* 包含与消息一起接收的任何标志（在 Unix 手册页中记录了 [recv(2)](https://oreil.ly/boM-c) 系统调用或 [Windows 文档](https://oreil.ly/90h4R)中有详细说明），*address* 是发送数据的对方套接字的地址（如果套接字已连接，则此值未定义，但可以从套接字中确定发送方）。 |
| send | send(*bytes*[, *flags*]) 将给定的数据 *bytes* 发送到已连接到远程端点的套接字上。返回发送的字节数，应检查：调用可能不会传输所有数据，此时必须单独请求剩余部分的传输。 |
| sendall | sendall(*bytes*[, *flags*]) 将所有给定的数据 *bytes* 发送到已连接到远程端点的套接字上。套接字的超时值适用于所有数据的传输，即使需要多次传输也是如此。 |
| sendfile | sendfile(*file,* offset=0, count=**None**) 将文件对象 *file* 的内容（必须以二进制模式打开）发送到连接的端点。在支持 os.sendfile 的平台上，使用该函数；否则，使用 send 调用。如果指定了 offset，则确定从文件中哪个字节位置开始传输；count 设置要传输的最大字节数。返回传输的总字节数。 |
| sendmsg | sendmsg(*buffers*[, *ancdata*[, *flags*[, *address*]]]) 向连接的端点发送普通和辅助（带外）数据。 *buffers* 应该是类字节对象的可迭代对象。 *ancdata* 参数应该是 (*data, ancdata, msg_flags, address*) 元组的可迭代对象，表示辅助数据。 *msg_flags* 是在 Unix 手册页上的 send(2) 系统调用或在 [Windows 文档](https://oreil.ly/90h4R) 中记录的标志位。 *address* 应仅在未连接的套接字中提供，并确定要发送数据的端点。 |
| sendto | sendto(*bytes*,[*flags*,]*address*) 将 *bytes*（*s* 不能连接）传输到给定的套接字地址，并返回发送的字节数。 可选的 *flags* 参数与 recv 中的含义相同。 |
| setblocking | setblocking(*flag*) 根据 *flag* 的真值确定 *s* 是否以阻塞模式运行（见 “套接字对象”）。 *s*.setblocking(**True**) 的作用类似于 *s*.settimeout(**None**)； *s*.set_blocking(**False**) 的作用类似于 *s*.settimeout(0.0)。 |
| set_inheritable | set_inheritable(*flag*) 根据 *flag* 的真值确定套接字是否由子进程继承。 |
| settimeout | settimeout(*timeout*) 根据 *timeout* 的值（见 “套接字对象”）建立 *s* 的模式。 |

| shutdown | shutdown(*how*) 根据 *how* 参数的值关闭套接字连接的一个或两个部分，如此处详细说明：

`socket.SHUT_RD`

*s* 上不能再执行更多的接收操作。

`socket.SHUT_RDWR`

*s* 上不能再执行更多的接收或发送操作。

`socket.SHUT_WR`

*s* 上不能再执行更多的发送操作。

|

套接字对象 *s* 还具有属性 family（*s* 的套接字家族）和 type（*s* 的套接字类型）。

## 无连接套接字客户端

考虑一个简单的数据包回显服务，在这个服务中，客户端将使用 UTF-8 编码的文本发送到服务器，服务器将相同的信息返回给客户端。 在无连接服务中，客户端只需将每个数据块发送到定义的服务器端点：

```py
`import` socket

UDP_IP = 'localhost'
UDP_PORT = 8883
MESSAGE = """\ This is a bunch of lines, each
of which will be sent in a single
UDP datagram. No error detection
or correction will occur.
Crazy bananas! £€ should go through."""

server = UDP_IP, UDP_PORT
encoding = 'utf-8'
`with` socket.socket(socket.AF_INET,    *`# IPv4`*
                   socket.SOCK_DGRAM, *`# UDP`*
                  ) `as` sock:
 `for` line `in` MESSAGE.splitlines():
        data = line.encode(encoding)
        bytes_sent = sock.sendto(data, server)
        print(f'SENT {data!r} ({bytes_sent} of {len(data)})'
		      f' to {server}')
        response, address = sock.recvfrom(1024)  *`# buffer size: 1024`*
        print(f'RCVD {response.decode(encoding)!r}'
              f' from {address}')

print('Disconnected from server')
```

请注意，服务器仅执行基于字节的回显功能。 因此，客户端将其 Unicode 数据编码为字节串，并使用相同的编码将从服务器接收的字节串响应解码为 Unicode 文本。

## 无连接套接字服务器

前一节描述的数据包回显服务的服务器也非常简单。 它绑定到其端点，在该端点接收数据包（数据报），并向客户端返回每个数据报具有完全相同数据的数据包。 服务器平等地对待所有客户端，不需要使用任何类型的并发（尽管这种最后一个方便的特性可能不适用于处理请求时间更长的服务）。

以下服务器工作，但除了通过中断（通常从键盘上的 Ctrl-C 或 Ctrl-Break）无其他终止服务的方法：

```py
`import` socket

UDP_IP = 'localhost'
UDP_PORT = 8883
with socket.socket(socket.AF_INET,    *`# IPv4`*
                   socket.SOCK_DGRAM  *`# UDP`*
                   ) as sock:
    sock.bind((UDP_IP, UDP_PORT))
    print(f'Serving at {UDP_IP}:{UDP_PORT}')
 `while` `True`:
        data, sender_addr = sock.recvfrom(1024)  *`# 1024-byte buffer`*
        print(f'RCVD {data!r}) from {sender_addr}')
        bytes_sent = sock.sendto(data, sender_addr)
        print(f'SENT {data!r} ({bytes_sent}/{len(data)})'
              f' to {sender_addr}')
```

同样没有任何机制来处理丢包和类似的网络问题；这在简单服务中通常是可以接受的。

可以使用 IPv6 运行相同的程序：只需将套接字类型 AF_INET 替换为 AF_INET6。

## 面向连接的套接字客户端

现在考虑一个简单的面向连接的“回显式”协议：服务器允许客户端连接到其监听套接字，从客户端接收任意字节，并将服务器接收到的相同字节发送回每个客户端，直到客户端关闭连接。以下是一个基本测试客户端的示例：³

```py
`import` socket

IP_ADDR = 'localhost'
IP_PORT = 8881
MESSAGE = """\ A few lines of text
including non-ASCII characters: €£
to test the operation
of both server
and client."""

encoding = 'utf-8'
`with` socket.socket(socket.AF_INET,     *`# IPv4`*
                   socket.SOCK_STREAM  *`# TCP`*
                   ) `as` sock:
    sock.connect((IP_ADDR, IP_PORT))
    print(f'Connected to server {IP_ADDR}:{IP_PORT}')
    `for` line `in` MESSAGE.splitlines():
        data = line.encode(encoding)
        sock.sendall(data)
        print(f'SENT {data!r} ({len(data)})')
        response, address = sock.recvfrom(1024)  *`# buffer size: 1024`*
        print(f'RCVD {response.decode(encoding)!r}'
              f' ({len(response)}) from {address}')

print('Disconnected from server')
```

注意数据是文本，因此必须用适当的表示方法进行编码。我们选择了常见的 UTF-8 编码。服务器以字节为单位工作（因为是字节（也称为八位字节）在网络上传输）；接收到的字节对象在打印之前会用 UTF-8 解码为 Unicode 文本。也可以选择其他合适的编解码器：关键是在传输之前对文本进行编码，在接收后进行解码。服务器在字节方面工作，甚至不需要知道使用的是哪种编码，除了可能用于日志记录之类的目的。

## 面向连接的套接字服务器

这是一个简单的服务器，对应于前一节中显示的测试客户端，使用并发.future 进行多线程处理（详见“concurrent.futures 模块”）：

```py
`import` concurrent
`import` socket

IP_ADDR = 'localhost'
IP_PORT = 8881

`def` handle(new_sock, address):
    print('Connected from', address)
    `with` new_sock:
        `while` True:
            received = new_sock.recv(1024)
            `if` `not` received:
                `break`
            s = received.decode('utf-8', errors='replace')
            print(f'Recv: {s!r}')
            new_sock.sendall(received)
            print(f'Echo: {s!r}')
    print(f'Disconnected from {address}')

`with` socket.socket(socket.AF_INET,     # IPv4 
                   socket.SOCK_STREAM  # TCP
                   ) `as` servsock:
    servsock.bind((IP_ADDR, IP_PORT))
    servsock.listen(5)
    print(f'Serving at {servsock.getsockname()}')
    `with` cconcurrent.futures.ThreadPoolExecutor(20) `as` e:
        `while` True:
            new_sock, address = servsock.accept()
            e.submit(handle, new_sock, address)
```

此服务器有其局限性。特别是，它仅运行 20 个线程，因此无法同时为超过 20 个客户端提供服务；当 20 个其他客户端正在接受服务时，试图连接的进一步客户端将等待在 servsock 的监听队列中。如果队列填满了等待接受的五个客户端，试图连接的进一步客户端将被直接拒绝。此服务器仅作为演示示例而非坚固、可扩展或安全系统。

与之前一样，可以通过将套接字类型 AF_INET 替换为 AF_INET6 来使用 IPv6 运行相同的程序。

# 传输层安全性

传输层安全性（TLS）是安全套接字层（SSL）的后继者，提供 TCP/IP 上的隐私和数据完整性，帮助防止服务器冒充、窃听交换的字节以及恶意修改这些字节。对于 TLS 的介绍，我们推荐阅读广泛的[Wikipedia 条目](https://oreil.ly/EzLWt)。

在 Python 中，您可以通过标准库的 ssl 模块使用 TLS。要很好地使用 ssl，您需要深入理解其丰富的[在线文档](https://oreil.ly/2EGr0)，以及 TLS 本身的深入广泛理解（尽管作为一个庞大而复杂的主题，维基百科的文章只是开始涵盖这个话题）。特别是，您必须研究并彻底理解[在线文档安全考虑部分](https://oreil.ly/ohqtT)，以及该部分提供的众多有用链接中的所有材料。

如果这些警告使您觉得完美实施安全预防措施是一项艰巨的任务，那是因为它确实如此。在安全领域，您需要将智慧和技能与那些可能更熟悉所涉问题细节的高级攻击者的智慧和技能相比较：他们专注于发现漏洞和入侵方法，而您的焦点通常不仅限于此类问题——相反，您试图在代码中提供一些有用的服务。将安全视为事后或次要问题是有风险的——它必须始终处于核心位置，以赢得技术和智慧之战。

尽管如此，我们强烈建议所有读者进行上述 TLS 学习——开发者对安全考虑的理解越深入，我们就越安全（除了那些渴望破解安全系统的人）。

除非您已经深入广泛地了解了 TLS 和 Python 的 ssl 模块（在这种情况下，您会知道应该做什么——比我们可能的建议更好！），我们建议使用 SSLContext 实例来保存 TLS 使用的所有细节。使用 ssl.create_default_context 函数构建该实例，如果需要，添加您的证书（如果您正在编写安全服务器，则需要这样做），然后使用实例的 wrap_socket 方法将您创建的每个 socket.socket 实例包装成 ssl.SSLSocket 实例——它的行为几乎与包装的 socket 对象相同，但几乎透明地添加了安全检查和验证“在一侧”。

默认的 TLS 上下文在安全性和广泛适用性之间取得了良好的折衷，我们建议您坚持使用它们（除非您足够了解以便为特殊需求调整和加强安全性）。如果您需要支持无法使用最新、最安全的 TLS 实现的过时对应方案，您可能会有种放松安全要求的冲动。但请自行承担风险——我们绝对不建议冒险进入这样的领域！

在接下来的章节中，我们将介绍您在仅想遵循我们建议时需要熟悉的 ssl 的最小子集。但即使是这种情况，也请*务必*阅读有关 TLS 和 ssl 的内容，以便对所涉及的复杂问题有所了解。这可能在某一天对您大有裨益！

# SSLContext

ssl 模块提供了 ssl.SSLContext 类，其实例保存关于 TLS 配置的信息（包括证书和私钥），并提供许多方法来设置、更改、检查和使用这些信息。如果你确切知道自己在做什么，你可以手动实例化、设置和使用自己专门目的的 SSLContext 实例。

但是，我们建议你使用经过良好调整的函数 ssl.create_default_context 实例化 SSLContext，只需一个参数：ssl.Purpose.CLIENT_AUTH 如果你的代码是服务器（因此可能需要对客户端进行认证），或者 ssl.Purpose.SERVER_AUTH 如果你的代码是客户端（因此绝对需要对服务器进行认证）。如果你的代码既是某些服务器的客户端又是其他客户端的服务器（例如一些互联网代理），那么你将需要两个 SSLContext 实例，每个目的一个。

对于大多数客户端使用场景，你的 SSLContext 已经准备好了。如果你在编写服务器端或者一个少数需要对客户端进行 TLS 认证的服务器的客户端，你需要有一个证书文件和一个密钥文件（参见[在线文档](https://oreil.ly/mBPJ0)了解如何获取这些文件）。通过将证书和密钥文件的路径传递给 load_cert_chain 方法，将它们添加到 SSLContext 实例中（以便对方可以验证你的身份），例如：

```py
ctx = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
ctx.load_cert_chain(certfile='mycert.pem', keyfile='mykey.key')
```

一旦你的上下文实例 *ctx* 准备好了，如果你在编写客户端，只需调用 *ctx*.wrap_socket 来包装你即将连接到服务器的任何套接字，并使用包装后的结果（一个 ssl.SSLSocket 实例）而不是刚刚包装的套接字。例如：

```py
sock = socket.socket(socket.AF_INET)
sock = ctx.wrap_socket(sock, server_hostname='www.example.com')
sock.connect(('www.example.com', 443))  
*`# use 'sock' normally from here on`*
```

注意，在客户端的情况下，你还应该传递一个 server_hostname 参数给 wrap_socket，该参数对应你即将连接的服务器；这样，连接可以验证你最终连接到的服务器的身份是否确实正确，这是绝对关键的安全步骤。

在服务器端，*不要* 包装绑定到地址、监听或接受连接的套接字；只需包装 accept 返回的新套接字。例如：

```py
sock = socket.socket(socket.AF_INET)
sock.bind(('www.example.com', 443))
sock.listen(5)
`while` `True``:`
    newsock, fromaddr = sock.accept()
    newsock = ctx.wrap_socket(newsock, server_side=`True`)
    *`# deal with 'newsock' as usual; shut down, then close it, when done`*
```

在这种情况下，你需要向 wrap_socket 传递参数 server_side=**True**，以便它知道你是服务器端的操作。

再次强烈建议查阅在线文档，尤其是[示例](https://oreil.ly/r6hQ7)，以便更好地理解，即使你仅仅使用 SSL 操作的这个简单子集。

¹ 当你编写应用程序时，通常会通过更高级别的抽象层（如第十九章中涵盖的那些）来使用套接字。

² 还有相对较新的多路复用连接传输协议 [QUIC](https://oreil.ly/1XwoM)，在 Python 中由第三方 [aioquic](https://oreil.ly/uh_1O) 支持。

³ 这个客户端示例并不安全；参见“传输层安全性”了解如何使其安全。

⁴ 我们说“几乎”是因为，当你编写服务器时，你不会包装你绑定、监听和接受连接的套接字。
