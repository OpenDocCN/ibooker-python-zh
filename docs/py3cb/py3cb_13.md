# 第十一章：网络与 Web 编程

本章是关于在网络应用和分布式应用中使用的各种主题。主题划分为使用 Python 编写客户端程序来访问已有的服务，以及使用 Python 实现网络服务端程序。也给出了一些常见的技术，用于编写涉及协同或通信的的代码。

# 11.1 作为客户端与 HTTP 服务交互

## 问题

你需要通过 HTTP 协议以客户端的方式访问多种服务。例如，下载数据或者与基于 REST 的 API 进行交互。

## 解决方案

对于简单的事情来说，通常使用 `<span class="pre" style="box-sizing: border-box;">urllib.request</span>` 模块就够了。例如，发送一个简单的 HTTP GET 请求到远程的服务上，可以这样做：

```py
      from urllib import request, parse

# Base URL being accessed
url = 'http://httpbin.org/get'

# Dictionary of query parameters (if any)
parms = {
   'name1' : 'value1',
   'name2' : 'value2'
}

# Encode the query string
querystring = parse.urlencode(parms)

# Make a GET request and read the response
u = request.urlopen(url+'?' + querystring)
resp = u.read()

```

如果你需要使用 POST 方法在请求主体中发送查询参数，可以将参数编码后作为可选参数提供给`<span class="pre" style="box-sizing: border-box;">urlopen()</span>` 函数，就像这样：

```py
      from urllib import request, parse

# Base URL being accessed
url = 'http://httpbin.org/post'

# Dictionary of query parameters (if any)
parms = {
   'name1' : 'value1',
   'name2' : 'value2'
}

# Encode the query string
querystring = parse.urlencode(parms)

# Make a POST request and read the response
u = request.urlopen(url, querystring.encode('ascii'))
resp = u.read()

```

如果你需要在发出的请求中提供一些自定义的 HTTP 头，例如修改 `<span class="pre" style="box-sizing: border-box;">user-agent</span>` 字段,可以创建一个包含字段值的字典，并创建一个 Request 实例然后将其传给 `<span class="pre" style="box-sizing: border-box;">urlopen()</span>` ，如下：

```py
      from urllib import request, parse
...

# Extra headers
headers = {
    'User-agent' : 'none/ofyourbusiness',
    'Spam' : 'Eggs'
}

req = request.Request(url, querystring.encode('ascii'), headers=headers)

# Make a request and read the response
u = request.urlopen(req)
resp = u.read()

```

如果需要交互的服务比上面的例子都要复杂，也许应该去看看 requests 库（[`pypi.python.org/pypi/requests`](https://pypi.python.org/pypi/requests)）。例如，下面这个示例采用 requests 库重新实现了上面的操作：

```py
      import requests

# Base URL being accessed
url = 'http://httpbin.org/post'

# Dictionary of query parameters (if any)
parms = {
   'name1' : 'value1',
   'name2' : 'value2'
}

# Extra headers
headers = {
    'User-agent' : 'none/ofyourbusiness',
    'Spam' : 'Eggs'
}

resp = requests.post(url, data=parms, headers=headers)

# Decoded text returned by the request
text = resp.text

```

关于 requests 库，一个值得一提的特性就是它能以多种方式从请求中返回响应结果的内容。从上面的代码来看， `<span class="pre" style="box-sizing: border-box;">resp.text</span>` 带给我们的是以 Unicode 解码的响应文本。但是，如果去访问 `<span class="pre" style="box-sizing: border-box;">resp.content</span>`，就会得到原始的二进制数据。另一方面，如果访问 `<span class="pre" style="box-sizing: border-box;">resp.json</span>` ，那么就会得到 JSON 格式的响应内容。

下面这个示例利用 `<span class="pre" style="box-sizing: border-box;">requests</span>` 库发起一个 HEAD 请求，并从响应中提取出一些 HTTP 头数据的字段：

```py
      import requests

resp = requests.head('http://www.python.org/index.html')

status = resp.status_code
last_modified = resp.headers['last-modified']
content_type = resp.headers['content-type']
content_length = resp.headers['content-length']

Here is a requests example that executes a login into the Python Package index using
basic authentication:
import requests

resp = requests.get('http://pypi.python.org/pypi?:action=login',
                    auth=('user','password'))

Here is an example of using requests to pass HTTP cookies from one request to the
next:

import requests

# First request
resp1 = requests.get(url)
...

# Second requests with cookies received on first requests
resp2 = requests.get(url, cookies=resp1.cookies)

Last, but not least, here is an example of using requests to upload content:

import requests
url = 'http://httpbin.org/post'
files = { 'file': ('data.csv', open('data.csv', 'rb')) }

r = requests.post(url, files=files)

```

## 讨论

对于真的很简单 HTTP 客户端代码，用内置的 `<span class="pre" style="box-sizing: border-box;">urllib</span>` 模块通常就足够了。但是，如果你要做的不仅仅只是简单的 GET 或 POST 请求，那就真的不能再依赖它的功能了。这时候就是第三方模块比如`<span class="pre" style="box-sizing: border-box;">requests</span>` 大显身手的时候了。

例如，如果你决定坚持使用标准的程序库而不考虑像 `<span class="pre" style="box-sizing: border-box;">requests</span>` 这样的第三方库，那么也许就不得不使用底层的 `<span class="pre" style="box-sizing: border-box;">http.client</span>` 模块来实现自己的代码。比方说，下面的代码展示了如何执行一个 HEAD 请求：

```py
      from http.client import HTTPConnection
from urllib import parse

c = HTTPConnection('www.python.org', 80)
c.request('HEAD', '/index.html')
resp = c.getresponse()

print('Status', resp.status)
for name, value in resp.getheaders():
    print(name, value)

```

同样地，如果必须编写涉及代理、认证、cookies 以及其他一些细节方面的代码，那么使用 `<span class="pre" style="box-sizing: border-box;">urllib</span>` 就显得特别别扭和啰嗦。比方说，下面这个示例实现在 Python 包索引上的认证：

```py
      import urllib.request

auth = urllib.request.HTTPBasicAuthHandler()
auth.add_password('pypi','http://pypi.python.org','username','password')
opener = urllib.request.build_opener(auth)

r = urllib.request.Request('http://pypi.python.org/pypi?:action=login')
u = opener.open(r)
resp = u.read()

# From here. You can access more pages using opener
...

```

坦白说，所有的这些操作在 `<span class="pre" style="box-sizing: border-box;">requests</span>` 库中都变得简单的多。

在开发过程中测试 HTTP 客户端代码常常是很令人沮丧的，因为所有棘手的细节问题都需要考虑（例如 cookies、认证、HTTP 头、编码方式等）。要完成这些任务，考虑使用 httpbin 服务（[`httpbin.org`](http://httpbin.org)）。这个站点会接收发出的请求，然后以 JSON 的形式将相应信息回传回来。下面是一个交互式的例子：

```py
      >>> import requests
>>> r = requests.get('http://httpbin.org/get?name=Dave&n=37',
...     headers = { 'User-agent': 'goaway/1.0' })
>>> resp = r.json
>>> resp['headers']
{'User-Agent': 'goaway/1.0', 'Content-Length': '', 'Content-Type': '',
'Accept-Encoding': 'gzip, deflate, compress', 'Connection':
'keep-alive', 'Host': 'httpbin.org', 'Accept': '*/*'}
>>> resp['args']
{'name': 'Dave', 'n': '37'}
>>>

```

在要同一个真正的站点进行交互前，先在 httpbin.org 这样的网站上做实验常常是可取的办法。尤其是当我们面对 3 次登录失败就会关闭账户这样的风险时尤为有用（不要尝试自己编写 HTTP 认证客户端来登录你的银行账户）。

尽管本节没有涉及， `<span class="pre" style="box-sizing: border-box;">request</span>` 库还对许多高级的 HTTP 客户端协议提供了支持，比如 OAuth。`<span class="pre" style="box-sizing: border-box;">requests</span>` 模块的文档（[`docs.python-requests.org`](http://docs.python-requests.org))质量很高（坦白说比在这短短的一节的篇幅中所提供的任何信息都好），可以参考文档以获得更多地信息。

# 11.2 创建 TCP 服务器

## 问题

你想实现一个服务器，通过 TCP 协议和客户端通信。

## 解决方案

创建一个 TCP 服务器的一个简单方法是使用 `<span class="pre" style="box-sizing: border-box;">socketserver</span>` 库。例如，下面是一个简单的应答服务器：

```py
      from socketserver import BaseRequestHandler, TCPServer

class EchoHandler(BaseRequestHandler):
    def handle(self):
        print('Got connection from', self.client_address)
        while True:

            msg = self.request.recv(8192)
            if not msg:
                break
            self.request.send(msg)

if __name__ == '__main__':
    serv = TCPServer(('', 20000), EchoHandler)
    serv.serve_forever()

```

在这段代码中，你定义了一个特殊的处理类，实现了一个 `<span class="pre" style="box-sizing: border-box;">handle()</span>` 方法，用来为客户端连接服务。`<span class="pre" style="box-sizing: border-box;">request</span>` 属性是客户端 socket，`<span class="pre" style="box-sizing: border-box;">client_address</span>` 有客户端地址。 为了测试这个服务器，运行它并打开另外一个 Python 进程连接这个服务器：

```py
      >>> from socket import socket, AF_INET, SOCK_STREAM
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s.connect(('localhost', 20000))
>>> s.send(b'Hello')
5
>>> s.recv(8192)
b'Hello'
>>>

```

很多时候，可以很容易的定义一个不同的处理器。下面是一个使用 `<span class="pre" style="box-sizing: border-box;">StreamRequestHandler</span>` 基类将一个类文件接口放置在底层 socket 上的例子：

```py
      from socketserver import StreamRequestHandler, TCPServer

class EchoHandler(StreamRequestHandler):
    def handle(self):
        print('Got connection from', self.client_address)
        # self.rfile is a file-like object for reading
        for line in self.rfile:
            # self.wfile is a file-like object for writing
            self.wfile.write(line)

if __name__ == '__main__':
    serv = TCPServer(('', 20000), EchoHandler)
    serv.serve_forever()

```

## 讨论

`<span class="pre" style="box-sizing: border-box;">socketserver</span>` 可以让我们很容易的创建简单的 TCP 服务器。 但是，你需要注意的是，默认情况下这种服务器是单线程的，一次只能为一个客户端连接服务。 如果你想处理多个客户端，可以初始化一个`<span class="pre" style="box-sizing: border-box;">ForkingTCPServer</span>` 或者是 `<span class="pre" style="box-sizing: border-box;">ThreadingTCPServer</span>` 对象。例如：

```py
      from socketserver import ThreadingTCPServer

if __name__ == '__main__':
    serv = ThreadingTCPServer(('', 20000), EchoHandler)
    serv.serve_forever()

```

使用 fork 或线程服务器有个潜在问题就是它们会为每个客户端连接创建一个新的进程或线程。 由于客户端连接数是没有限制的，因此一个恶意的黑客可以同时发送大量的连接让你的服务器奔溃。

如果你担心这个问题，你可以创建一个预先分配大小的工作线程池或进程池。 你先创建一个普通的非线程服务器，然后在一个线程池中使用 `<span class="pre" style="box-sizing: border-box;">serve_forever()</span>` 方法来启动它们。

```py
      if __name__ == '__main__':
    from threading import Thread
    NWORKERS = 16
    serv = TCPServer(('', 20000), EchoHandler)
    for n in range(NWORKERS):
        t = Thread(target=serv.serve_forever)
        t.daemon = True
        t.start()
    serv.serve_forever()

```

一般来讲，一个 `<span class="pre" style="box-sizing: border-box;">TCPServer</span>` 在实例化的时候会绑定并激活相应的 `<span class="pre" style="box-sizing: border-box;">socket</span>` 。 不过，有时候你想通过设置某些选项去调整底下的 socket` ，可以设置参数 `bind_and_activate=False` 。如下：

```py
      if __name__ == '__main__':
    serv = TCPServer(('', 20000), EchoHandler, bind_and_activate=False)
    # Set up various socket options
    serv.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
    # Bind and activate
    serv.server_bind()
    serv.server_activate()
    serv.serve_forever()

```

上面的 `<span class="pre" style="box-sizing: border-box;">socket</span>` 选项是一个非常普遍的配置项，它允许服务器重新绑定一个之前使用过的端口号。 由于要被经常使用到，它被放置到类变量中，可以直接在 `<span class="pre" style="box-sizing: border-box;">TCPServer</span>` 上面设置。 在实例化服务器的时候去设置它的值，如下所示：

```py
      if __name__ == '__main__':
    TCPServer.allow_reuse_address = True
    serv = TCPServer(('', 20000), EchoHandler)
    serv.serve_forever()

```

在上面示例中，我们演示了两种不同的处理器基类（ `<span class="pre" style="box-sizing: border-box;">BaseRequestHandler</span>` 和`<span class="pre" style="box-sizing: border-box;">StreamRequestHandler</span>` ）。 `<span class="pre" style="box-sizing: border-box;">StreamRequestHandler</span>` 更加灵活点，能通过设置其他的类变量来支持一些新的特性。比如：

```py
      import socket

class EchoHandler(StreamRequestHandler):
    # Optional settings (defaults shown)
    timeout = 5                      # Timeout on all socket operations
    rbufsize = -1                    # Read buffer size
    wbufsize = 0                     # Write buffer size
    disable_nagle_algorithm = False  # Sets TCP_NODELAY socket option
    def handle(self):
        print('Got connection from', self.client_address)
        try:
            for line in self.rfile:
                # self.wfile is a file-like object for writing
                self.wfile.write(line)
        except socket.timeout:
            print('Timed out!')

```

最后，还需要注意的是巨大部分 Python 的高层网络模块（比如 HTTP、XML-RPC 等）都是建立在`<span class="pre" style="box-sizing: border-box;">socketserver</span>` 功能之上。 也就是说，直接使用 `<span class="pre" style="box-sizing: border-box;">socket</span>` 库来实现服务器也并不是很难。 下面是一个使用 `<span class="pre" style="box-sizing: border-box;">socket</span>` 直接编程实现的一个服务器简单例子：

```py
      from socket import socket, AF_INET, SOCK_STREAM

def echo_handler(address, client_sock):
    print('Got connection from {}'.format(address))
    while True:
        msg = client_sock.recv(8192)
        if not msg:
            break
        client_sock.sendall(msg)
    client_sock.close()

def echo_server(address, backlog=5):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(address)
    sock.listen(backlog)
    while True:
        client_sock, client_addr = sock.accept()
        echo_handler(client_addr, client_sock)

if __name__ == '__main__':
    echo_server(('', 20000))

```

# 11.3 创建 UDP 服务器

## 问题

你想实现一个基于 UDP 协议的服务器来与客户端通信。

## 解决方案

跟 TCP 一样，UDP 服务器也可以通过使用 `<span class="pre" style="box-sizing: border-box;">socketserver</span>` 库很容易的被创建。 例如，下面是一个简单的时间服务器：

```py
      from socketserver import BaseRequestHandler, UDPServer
import time

class TimeHandler(BaseRequestHandler):
    def handle(self):
        print('Got connection from', self.client_address)
        # Get message and client socket
        msg, sock = self.request
        resp = time.ctime()
        sock.sendto(resp.encode('ascii'), self.client_address)

if __name__ == '__main__':
    serv = UDPServer(('', 20000), TimeHandler)
    serv.serve_forever()

```

跟之前一样，你先定义一个实现 `<span class="pre" style="box-sizing: border-box;">handle()</span>` 特殊方法的类，为客户端连接服务。 这个类的 `<span class="pre" style="box-sizing: border-box;">request</span>`属性是一个包含了数据报和底层 socket 对象的元组。`<span class="pre" style="box-sizing: border-box;">client_address</span>` 包含了客户端地址。

我们来测试下这个服务器，首先运行它，然后打开另外一个 Python 进程向服务器发送消息：

```py
      >>> from socket import socket, AF_INET, SOCK_DGRAM
>>> s = socket(AF_INET, SOCK_DGRAM)
>>> s.sendto(b'', ('localhost', 20000))
0
>>> s.recvfrom(8192)
(b'Wed Aug 15 20:35:08 2012', ('127.0.0.1', 20000))
>>>

```

## 讨论

一个典型的 UPD 服务器接收到达的数据报(消息)和客户端地址。如果服务器需要做应答， 它要给客户端回发一个数据报。对于数据报的传送， 你应该使用 socket 的 `<span class="pre" style="box-sizing: border-box;">sendto()</span>` 和 `<span class="pre" style="box-sizing: border-box;">recvfrom()</span>` 方法。 尽管传统的 `<span class="pre" style="box-sizing: border-box;">send()</span>` 和 `<span class="pre" style="box-sizing: border-box;">recv()</span>` 也可以达到同样的效果， 但是前面的两个方法对于 UDP 连接而言更普遍。

由于没有底层的连接，UPD 服务器相对于 TCP 服务器来讲实现起来更加简单。 不过，UDP 天生是不可靠的（因为通信没有建立连接，消息可能丢失）。 因此需要由你自己来决定该怎样处理丢失消息的情况。这个已经不在本书讨论范围内了， 不过通常来说，如果可靠性对于你程序很重要，你需要借助于序列号、重试、超时以及一些其他方法来保证。 UDP 通常被用在那些对于可靠传输要求不是很高的场合。例如，在实时应用如多媒体流以及游戏领域， 无需返回恢复丢失的数据包（程序只需简单的忽略它并继续向前运行）。

`<span class="pre" style="box-sizing: border-box;">UDPServer</span>` 类是单线程的，也就是说一次只能为一个客户端连接服务。 实际使用中，这个无论是对于 UDP 还是 TCP 都不是什么大问题。 如果你想要并发操作，可以实例化一个 `<span class="pre" style="box-sizing: border-box;">ForkingUDPServer</span>` 或`<span class="pre" style="box-sizing: border-box;">ThreadingUDPServer</span>` 对象：

```py
      from socketserver import ThreadingUDPServer

   if __name__ == '__main__':
    serv = ThreadingUDPServer(('',20000), TimeHandler)
    serv.serve_forever()

```

直接使用 `<span class="pre" style="box-sizing: border-box;">socket</span>` 来是想一个 UDP 服务器也不难，下面是一个例子：

```py
      from socket import socket, AF_INET, SOCK_DGRAM
import time

def time_server(address):
    sock = socket(AF_INET, SOCK_DGRAM)
    sock.bind(address)
    while True:
        msg, addr = sock.recvfrom(8192)
        print('Got message from', addr)
        resp = time.ctime()
        sock.sendto(resp.encode('ascii'), addr)

if __name__ == '__main__':
    time_server(('', 20000))

```

# 11.4 通过 CIDR 地址生成对应的 IP 地址集

## 问题

你有一个 CIDR 网络地址比如“123.45.67.89/27”，你想将其转换成它所代表的所有 IP （比如，“123.45.67.64”, “123.45.67.65”, …, “123.45.67.95”)）

## 解决方案

可以使用 `<span class="pre" style="box-sizing: border-box;">ipaddress</span>` 模块很容易的实现这样的计算。例如：

```py
      >>> import ipaddress
>>> net = ipaddress.ip_network('123.45.67.64/27')
>>> net
IPv4Network('123.45.67.64/27')
>>> for a in net:
...     print(a)
...
123.45.67.64
123.45.67.65
123.45.67.66
123.45.67.67
123.45.67.68
...
123.45.67.95
>>>

>>> net6 = ipaddress.ip_network('12:3456:78:90ab:cd:ef01:23:30/125')
>>> net6
IPv6Network('12:3456:78:90ab:cd:ef01:23:30/125')
>>> for a in net6:
...     print(a)
...
12:3456:78:90ab:cd:ef01:23:30
12:3456:78:90ab:cd:ef01:23:31
12:3456:78:90ab:cd:ef01:23:32
12:3456:78:90ab:cd:ef01:23:33
12:3456:78:90ab:cd:ef01:23:34
12:3456:78:90ab:cd:ef01:23:35
12:3456:78:90ab:cd:ef01:23:36
12:3456:78:90ab:cd:ef01:23:37
>>>

```

`<span class="pre" style="box-sizing: border-box;">Network</span>` 也允许像数组一样的索引取值，例如：

```py
      >>> net.num_addresses
32
>>> net[0]

IPv4Address('123.45.67.64')
>>> net[1]
IPv4Address('123.45.67.65')
>>> net[-1]
IPv4Address('123.45.67.95')
>>> net[-2]
IPv4Address('123.45.67.94')
>>>

```

另外，你还可以执行网络成员检查之类的操作：

```py
      >>> a = ipaddress.ip_address('123.45.67.69')
>>> a in net
True
>>> b = ipaddress.ip_address('123.45.67.123')
>>> b in net
False
>>>

```

一个 IP 地址和网络地址能通过一个 IP 接口来指定，例如：

```py
      >>> inet = ipaddress.ip_interface('123.45.67.73/27')
>>> inet.network
IPv4Network('123.45.67.64/27')
>>> inet.ip
IPv4Address('123.45.67.73')
>>>

```

## 讨论

`<span class="pre" style="box-sizing: border-box;">ipaddress</span>` 模块有很多类可以表示 IP 地址、网络和接口。 当你需要操作网络地址（比如解析、打印、验证等）的时候会很有用。

要注意的是，`<span class="pre" style="box-sizing: border-box;">ipaddress</span>` 模块跟其他一些和网络相关的模块比如 `<span class="pre" style="box-sizing: border-box;">socket</span>` 库交集很少。 所以，你不能使用 `<span class="pre" style="box-sizing: border-box;">IPv4Address</span>` 的实例来代替一个地址字符串，你首先得显式的使用 `<span class="pre" style="box-sizing: border-box;">str()</span>` 转换它。例如：

```py
      >>> a = ipaddress.ip_address('127.0.0.1')
>>> from socket import socket, AF_INET, SOCK_STREAM
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s.connect((a, 8080))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't convert 'IPv4Address' object to str implicitly
>>> s.connect((str(a), 8080))
>>>

```

更多相关内容，请参考 [An Introduction to the ipaddress Module](https://docs.python.org/3/howto/ipaddress.html)

# 11.5 生成一个简单的 REST 接口

## 问题

你想使用一个简单的 REST 接口通过网络远程控制或访问你的应用程序，但是你又不想自己去安装一个完整的 web 框架。

## 解决方案

构建一个 REST 风格的接口最简单的方法是创建一个基于 WSGI 标准（PEP 3333）的很小的库，下面是一个例子：

```py
      # resty.py

import cgi

def notfound_404(environ, start_response):
    start_response('404 Not Found', [ ('Content-type', 'text/plain') ])
    return [b'Not Found']

class PathDispatcher:
    def __init__(self):
        self.pathmap = { }

    def __call__(self, environ, start_response):
        path = environ['PATH_INFO']
        params = cgi.FieldStorage(environ['wsgi.input'],
                                  environ=environ)
        method = environ['REQUEST_METHOD'].lower()
        environ['params'] = { key: params.getvalue(key) for key in params }
        handler = self.pathmap.get((method,path), notfound_404)
        return handler(environ, start_response)

    def register(self, method, path, function):
        self.pathmap[method.lower(), path] = function
        return function

```

为了使用这个调度器，你只需要编写不同的处理器，就像下面这样：

```py
      import time

_hello_resp = '''\
<html>
  <head>
     <title>Hello {name}</title>
   </head>
   <body>
     <h1>Hello {name}!</h1>
   </body>
</html>'''

def hello_world(environ, start_response):
    start_response('200 OK', [ ('Content-type','text/html')])
    params = environ['params']
    resp = _hello_resp.format(name=params.get('name'))
    yield resp.encode('utf-8')

_localtime_resp = '''\
<?xml version="1.0"?>
<time>
  <year>{t.tm_year}</year>
  <month>{t.tm_mon}</month>
  <day>{t.tm_mday}</day>
  <hour>{t.tm_hour}</hour>
  <minute>{t.tm_min}</minute>
  <second>{t.tm_sec}</second>
</time>'''

def localtime(environ, start_response):
    start_response('200 OK', [ ('Content-type', 'application/xml') ])
    resp = _localtime_resp.format(t=time.localtime())
    yield resp.encode('utf-8')

if __name__ == '__main__':
    from resty import PathDispatcher
    from wsgiref.simple_server import make_server

    # Create the dispatcher and register functions
    dispatcher = PathDispatcher()
    dispatcher.register('GET', '/hello', hello_world)
    dispatcher.register('GET', '/localtime', localtime)

    # Launch a basic server
    httpd = make_server('', 8080, dispatcher)
    print('Serving on port 8080...')
    httpd.serve_forever()

```

要测试下这个服务器，你可以使用一个浏览器或 `<span class="pre" style="box-sizing: border-box;">urllib</span>` 和它交互。例如：

```py
      >>> u = urlopen('http://localhost:8080/hello?name=Guido')
>>> print(u.read().decode('utf-8'))
<html>
  <head>
     <title>Hello Guido</title>
   </head>
   <body>
     <h1>Hello Guido!</h1>
   </body>
</html>

>>> u = urlopen('http://localhost:8080/localtime')
>>> print(u.read().decode('utf-8'))
<?xml version="1.0"?>
<time>
  <year>2012</year>
  <month>11</month>
  <day>24</day>
  <hour>14</hour>
  <minute>49</minute>
  <second>17</second>
</time>
>>>

```

## 讨论

在编写 REST 接口时，通常都是服务于普通的 HTTP 请求。但是跟那些功能完整的网站相比，你通常只需要处理数据。 这些数据以各种标准格式编码，比如 XML、JSON 或 CSV。 尽管程序看上去很简单，但是以这种方式提供的 API 对于很多应用程序来讲是非常有用的。

例如，长期运行的程序可能会使用一个 REST API 来实现监控或诊断。 大数据应用程序可以使用 REST 来构建一个数据查询或提取系统。 REST 还能用来控制硬件设备比如机器人、传感器、工厂或灯泡。 更重要的是，REST API 已经被大量客户端编程环境所支持，比如 Javascript, Android, iOS 等。 因此，利用这种接口可以让你开发出更加复杂的应用程序。

为了实现一个简单的 REST 接口，你只需让你的程序代码满足 Python 的 WSGI 标准即可。 WSGI 被标准库支持，同时也被绝大部分第三方 web 框架支持。 因此，如果你的代码遵循这个标准，在后面的使用过程中就会更加的灵活！

在 WSGI 中，你可以像下面这样约定的方式以一个可调用对象形式来实现你的程序。

```py
      import cgi

def wsgi_app(environ, start_response):
    pass

```

`<span class="pre" style="box-sizing: border-box;">environ</span>` 属性是一个字典，包含了从 web 服务器如 Apache[参考 Internet RFC 3875]提供的 CGI 接口中获取的值。 要将这些不同的值提取出来，你可以像这么这样写：

```py
      def wsgi_app(environ, start_response):
    method = environ['REQUEST_METHOD']
    path = environ['PATH_INFO']
    # Parse the query parameters
    params = cgi.FieldStorage(environ['wsgi.input'], environ=environ)

```

我们展示了一些常见的值。`<span class="pre" style="box-sizing: border-box;">environ['REQUEST_METHOD']</span>` 代表请求类型如 GET、POST、HEAD 等。`<span class="pre" style="box-sizing: border-box;">environ['PATH_INFO']</span>` 表示被请求资源的路径。 调用 `<span class="pre" style="box-sizing: border-box;">cgi.FieldStorage()</span>` 可以从请求中提取查询参数并将它们放入一个类字典对象中以便后面使用。

`<span class="pre" style="box-sizing: border-box;">start_response</span>` 参数是一个为了初始化一个请求对象而必须被调用的函数。 第一个参数是返回的 HTTP 状态值，第二个参数是一个(名,值)元组列表，用来构建返回的 HTTP 头。例如：

```py
      def wsgi_app(environ, start_response):
    pass
    start_response('200 OK', [('Content-type', 'text/plain')])

```

为了返回数据，一个 WSGI 程序必须返回一个字节字符串序列。可以像下面这样使用一个列表来完成：

```py
      def wsgi_app(environ, start_response):
    pass
    start_response('200 OK', [('Content-type', 'text/plain')])
    resp = []
    resp.append(b'Hello World\n')
    resp.append(b'Goodbye!\n')
    return resp

```

或者，你还可以使用 `<span class="pre" style="box-sizing: border-box;">yield</span>` ：

```py
      def wsgi_app(environ, start_response):
    pass
    start_response('200 OK', [('Content-type', 'text/plain')])
    yield b'Hello World\n'
    yield b'Goodbye!\n'

```

这里要强调的一点是最后返回的必须是字节字符串。如果返回结果包含文本字符串，必须先将其编码成字节。 当然，并没有要求你返回的一点是文本，你可以很轻松的编写一个生成图片的程序。

尽管 WSGI 程序通常被定义成一个函数，不过你也可以使用类实例来实现，只要它实现了合适的`<span class="pre" style="box-sizing: border-box;">__call__()</span>` 方法。例如：

```py
      class WSGIApplication:
    def __init__(self):
        ...
    def __call__(self, environ, start_response)
       ...

```

我们已经在上面使用这种技术创建 `<span class="pre" style="box-sizing: border-box;">PathDispatcher</span>` 类。 这个分发器仅仅只是管理一个字典，将(方法,路径)对映射到处理器函数上面。 当一个请求到来时，它的方法和路径被提取出来，然后被分发到对应的处理器上面去。 另外，任何查询变量会被解析后放到一个字典中，以 `<span class="pre" style="box-sizing: border-box;">environ['params']</span>` 形式存储。 后面这个步骤太常见，所以建议你在分发器里面完成，这样可以省掉很多重复代码。 使用分发器的时候，你只需简单的创建一个实例，然后通过它注册各种 WSGI 形式的函数。 编写这些函数应该超级简单了，只要你遵循 `<span class="pre" style="box-sizing: border-box;">start_response()</span>` 函数的编写规则，并且最后返回字节字符串即可。

当编写这种函数的时候还需注意的一点就是对于字符串模板的使用。 没人愿意写那种到处混合着`<span class="pre" style="box-sizing: border-box;">print()</span>` 函数 、XML 和大量格式化操作的代码。 我们上面使用了三引号包含的预先定义好的字符串模板。 这种方式的可以让我们很容易的在以后修改输出格式(只需要修改模板本身，而不用动任何使用它的地方)。

最后，使用 WSGI 还有一个很重要的部分就是没有什么地方是针对特定 web 服务器的。 因为标准对于服务器和框架是中立的，你可以将你的程序放入任何类型服务器中。 我们使用下面的代码测试测试本节代码：

```py
      if __name__ == '__main__':
    from wsgiref.simple_server import make_server

    # Create the dispatcher and register functions
    dispatcher = PathDispatcher()
    pass

    # Launch a basic server
    httpd = make_server('', 8080, dispatcher)
    print('Serving on port 8080...')
    httpd.serve_forever()

```

上面代码创建了一个简单的服务器，然后你就可以来测试下你的实现是否能正常工作。 最后，当你准备进一步扩展你的程序的时候，你可以修改这个代码，让它可以为特定服务器工作。

WSGI 本身是一个很小的标准。因此它并没有提供一些高级的特性比如认证、cookies、重定向等。 这些你自己实现起来也不难。不过如果你想要更多的支持，可以考虑第三方库，比如 `<span class="pre" style="box-sizing: border-box;">WebOb</span>` 或者`<span class="pre" style="box-sizing: border-box;">Paste</span>`

# 11.6 通过 XML-RPC 实现简单的远程调用

## 问题

You want an easy way to execute functions or methods in Python programs running onremote machines.

## 解决方案

Perhaps the easiest way to implement a simple remote procedure call mechanism is touse XML-RPC. Here is an example of a simple server that implements a simple key-value store:

from xmlrpc.server import SimpleXMLRPCServer

class KeyValueServer:
_rpc*methods* = [‘get', ‘set', ‘delete', ‘exists', ‘keys']def **init**(self, address):

> > self._data = {}self._serv = SimpleXMLRPCServer(address, allow_none=True)for name in self._rpc*methods*:
> > 
> > self._serv.register_function(getattr(self, name))

def get(self, name):return self._data[name]def set(self, name, value):self._data[name] = valuedef delete(self, name):del self._data[name]def exists(self, name):return name in self._datadef keys(self):return list(self._data)def serve_forever(self):self._serv.serve_forever()

# Exampleif **name** == ‘**main**':

> kvserv = KeyValueServer((‘', 15000))kvserv.serve_forever()

Here is how you would access the server remotely from a client:

```py
      >>> from xmlrpc.client import ServerProxy
>>> s = ServerProxy('http://localhost:15000', allow_none=True)
>>> s.set('foo', 'bar')
>>> s.set('spam', [1, 2, 3])
>>> s.keys()
['spam', 'foo']
>>> s.get('foo')
'bar'
>>> s.get('spam')
[1, 2, 3]
>>> s.delete('spam')
>>> s.exists('spam')
False
>>>

```

## 讨论

XML-RPC can be an extremely easy way to set up a simple remote procedure call service.All you need to do is create a server instance, register functions with it using the register_function() method, and then launch it using the serve_forever() method. Thisrecipe packages it up into a class to put all of the code together, but there is no suchrequirement. For example, you could create a server by trying something like this:

from xmlrpc.server import SimpleXMLRPCServerdef add(x,y):

> return x+y

serv = SimpleXMLRPCServer((‘', 15000))serv.register_function(add)serv.serve_forever()

Functions exposed via XML-RPC only work with certain kinds of data such as strings,numbers, lists, and dictionaries. For everything else, some study is required. For in‐stance, if you pass an instance through XML-RPC, only its instance dictionary ishandled:

```py
      >>> class Point:
...     def __init__(self, x, y):
...             self.x = x
...             self.y = y
...
>>> p = Point(2, 3)
>>> s.set('foo', p)
>>> s.get('foo')
{'x': 2, 'y': 3}
>>>

```

Similarly, handling of binary data is a bit different than you expect:

```py
      >>> s.set('foo', b'Hello World')
>>> s.get('foo')
<xmlrpc.client.Binary object at 0x10131d410>

>>> _.data
b'Hello World'
>>>

```

As a general rule, you probably shouldn’t expose an XML-RPC service to the rest of theworld as a public API. It often works best on internal networks where you might wantto write simple distributed programs involving a few different machines.A downside to XML-RPC is its performance. The SimpleXMLRPCServer implementa‐tion is only single threaded, and wouldn’t be appropriate for scaling a large application,although it can be made to run multithreaded, as shown in Recipe 11.2\. Also, sinceXML-RPC serializes all data as XML, it’s inherently slower than other approaches.However, one benefit of this encoding is that it’s understood by a variety of other pro‐gramming languages. By using it, clients written in languages other than Python will beable to access your service.Despite its limitations, XML-RPC is worth knowing about if you ever have the need tomake a quick and dirty remote procedure call system. Oftentimes, the simple solutionis good enough.

# 11.7 在不同的 Python 解释器之间交互

## 问题

You are running multiple instances of the Python interpreter, possibly on different ma‐chines, and you would like to exchange data between interpreters using messages.

## 解决方案

It is easy to communicate between interpreters if you use the multiprocessing.connection module. Here is a simple example of writing an echo server:

from multiprocessing.connection import Listenerimport traceback

def echo_client(conn):try:while True:msg = conn.recv()conn.send(msg)except EOFError:print(‘Connection closed')def echo_server(address, authkey):
serv = Listener(address, authkey=authkey)while True:

> try:> client = serv.accept()
> 
> echo_client(client)

except Exception:traceback.print_exc()

echo_server((‘', 25000), authkey=b'peekaboo')

Here is a simple example of a client connecting to the server and sending variousmessages:

```py
      >>> from multiprocessing.connection import Client
>>> c = Client(('localhost', 25000), authkey=b'peekaboo')
>>> c.send('hello')
>>> c.recv()
'hello'
>>> c.send(42)
>>> c.recv()
42
>>> c.send([1, 2, 3, 4, 5])
>>> c.recv()
[1, 2, 3, 4, 5]
>>>

```

Unlike a low-level socket, messages are kept intact (each object sent using send() isreceived in its entirety with recv()). In addition, objects are serialized using pickle.So, any object compatible with pickle can be sent or received over the connection.

## 讨论

There are many packages and libraries related to implementing various forms of mes‐sage passing, such as ZeroMQ, Celery, and so forth. As an alternative, you might alsobe inclined to implement a message layer on top of low-level sockets. However, some‐times you just want a simple solution. The multiprocessing.connection library is justthat—using a few simple primitives, you can easily connect interpreters together andhave them exchange messages.If you know that the interpreters are going to be running on the same machine, you canuse alternative forms of networking, such as UNIX domain sockets or Windows namedpipes. To create a connection using a UNIX domain socket, simply change the addressto a filename such as this:

s = Listener(‘/tmp/myconn', authkey=b'peekaboo')

To create a connection using a Windows named pipe, use a filename such as this:

s = Listener(r'.pipemyconn', authkey=b'peekaboo')

As a general rule, you would not be using multiprocessing to implement public-facingservices. The authkey parameter to Client() and Listener() is there to help authen‐ticate the end points of the connection. Connection attempts with a bad key raise anexception. In addition, the module is probably best suited for long-running connections

(not a large number of short connections). For example, two interpreters might establisha connection at startup and keep the connection active for the entire duration of aproblem.Don’t use multiprocessing if you need more low-level control over aspects of the con‐nection. For example, if you needed to support timeouts, nonblocking I/O, or anythingsimilar, you’re probably better off using a different library or implementing such featureson top of sockets instead.

# 11.8 实现远程方法调用

## 问题

You want to implement simple remote procedure call (RPC) on top of a message passinglayer, such as sockets, multiprocessing connections, or ZeroMQ.

## 解决方案

RPC is easy to implement by encoding function requests, arguments, and return valuesusing pickle, and passing the pickled byte strings between interpreters. Here is anexample of a simple RPC handler that could be incorporated into a server:

# rpcserver.py

import pickleclass RPCHandler:

> def **init**(self):self._functions = { }def register_function(self, func):self._functions[func.**name**] = funcdef handle_connection(self, connection):try:while True:> # Receive a messagefunc_name, args, kwargs = pickle.loads(connection.recv())# Run the RPC and send a responsetry:
> 
> > r = self._functionsfunc_nameargs,**kwargs)connection.send(pickle.dumps(r))

except Exception as e:connection.send(pickle.dumps(e))except EOFError:pass

To use this handler, you need to add it into a messaging server. There are many possiblechoices, but the multiprocessing library provides a simple option. Here is an exampleRPC server:

from multiprocessing.connection import Listenerfrom threading import Thread

def rpc_server(handler, address, authkey):
sock = Listener(address, authkey=authkey)while True:

> client = sock.accept()t = Thread(target=handler.handle_connection, args=(client,))t.daemon = Truet.start()

# Some remote functionsdef add(x, y):

> return x + y

def sub(x, y):return x - y

# Register with a handlerhandler = RPCHandler()handler.register_function(add)handler.register_function(sub)

# Run the serverrpc_server(handler, (‘localhost', 17000), authkey=b'peekaboo')

To access the server from a remote client, you need to create a corresponding RPC proxyclass that forwards requests. For example:

import pickle

class RPCProxy:def **init**(self, connection):self._connection = connectiondef **getattr**(self, name):def do_rpc(*args, **kwargs):
self._connection.send(pickle.dumps((name, args, kwargs)))result = pickle.loads(self._connection.recv())if isinstance(result, Exception):

> raise result

return result

return do_rpc

To use the proxy, you wrap it around a connection to the server. For example:

```py
      >>> from multiprocessing.connection import Client
>>> c = Client(('localhost', 17000), authkey=b'peekaboo')
>>> proxy = RPCProxy(c)
>>> proxy.add(2, 3)

```

5>>> proxy.sub(2, 3)-1>>> proxy.sub([1, 2], 4)Traceback (most recent call last):

> > File “<stdin>”, line 1, in <module>File “rpcserver.py”, line 37, in do_rpc</module></stdin>
> > 
> > raise result

TypeError: unsupported operand type(s) for -: ‘list' and ‘int'>>>

It should be noted that many messaging layers (such as multiprocessing) already se‐rialize data using pickle. If this is the case, the pickle.dumps() and pickle.loads()calls can be eliminated.

## 讨论

The general idea of the RPCHandler and RPCProxy classes is relatively simple. If a clientwants to call a remote function, such as foo(1, 2, z=3), the proxy class creates a tuple(‘foo', (1, 2), {‘z': 3}) that contains the function name and arguments. Thistuple is pickled and sent over the connection. This is performed in the do_rpc() closurethat’s returned by the **getattr**() method of RPCProxy. The server receives andunpickles the message, looks up the function name to see if it’s registered, and executesit with the given arguments. The result (or exception) is then pickled and sent back.As shown, the example relies on multiprocessing for communication. However, thisapproach could be made to work with just about any other messaging system. For ex‐ample, if you want to implement RPC over ZeroMQ, just replace the connection objectswith an appropriate ZeroMQ socket object.Given the reliance on pickle, security is a major concern (because a clever hacker cancreate messages that make arbitrary functions execute during unpickling). In particular,you should never allow RPC from untrusted or unauthenticated clients. In particular,you definitely don’t want to allow access from just any machine on the Internet—thisshould really only be used internally, behind a firewall, and not exposed to the rest ofthe world.As an alternative to pickle, you might consider the use of JSON, XML, or some otherdata encoding for serialization. For example, this recipe is fairly easy to adapt to JSONencodingif you simply replace pickle.loads() and pickle.dumps() withjson.loads() and json.dumps(). For example:

# jsonrpcserver.pyimport json

class RPCHandler:def **init**(self):self._functions = { }def register_function(self, func):self._functions[func.**name**] = funcdef handle_connection(self, connection):try:while True:

# Receive a messagefunc_name, args, kwargs = json.loads(connection.recv())# Run the RPC and send a responsetry:

> r = self._functionsfunc_nameargs,**kwargs)connection.send(json.dumps(r))

except Exception as e:connection.send(json.dumps(str(e)))except EOFError:pass

# jsonrpcclient.pyimport json

class RPCProxy:def **init**(self, connection):self._connection = connectiondef **getattr**(self, name):def do_rpc(*args, **kwargs):self._connection.send(json.dumps((name, args, kwargs)))result = json.loads(self._connection.recv())return result
return do_rpc

One complicated factor in implementing RPC is how to handle exceptions. At the veryleast, the server shouldn’t crash if an exception is raised by a method. However, themeans by which the exception gets reported back to the client requires some study. Ifyou’re using pickle, exception instances can often be serialized and reraised in theclient. If you’re using some other protocol, you might have to think of an alternativeapproach. At the very least, you would probably want to return the exception string inthe response. This is the approach taken in the JSON example.For another example of an RPC implementation, it can be useful to look at the imple‐mentation of the SimpleXMLRPCServer and ServerProxy classes used in XML-RPC, asdescribed in Recipe 11.6.

# 11.9 简单的客户端认证

## 问题

You want a simple way to authenticate the clients connecting to servers in a distributedsystem, but don’t need the complexity of something like SSL.

## 解决方案

Simple but effective authentication can be performed by implementing a connectionhandshake using the hmac module. Here is sample code:

import hmacimport os

def client_authenticate(connection, secret_key):‘''Authenticate client to a remote service.connection represents a network connection.secret_key is a key known only to both client/server.‘''message = connection.recv(32)hash = hmac.new(secret_key, message)digest = hash.digest()connection.send(digest)def server_authenticate(connection, secret_key):‘''Request client authentication.‘''message = os.urandom(32)connection.send(message)hash = hmac.new(secret_key, message)digest = hash.digest()response = connection.recv(len(digest))return hmac.compare_digest(digest,response)
The general idea is that upon connection, the server presents the client with a messageof random bytes (returned by os.urandom(), in this case). The client and server bothcompute a cryptographic hash of the random data using hmac and a secret key knownonly to both ends. The client sends its computed digest back to the server, where it iscompared and used to decide whether or not to accept or reject the connection.Comparison of resulting digests should be performed using the hmac.compare_digest() function. This function has been written in a way that avoids timing-analysis-based attacks and should be used instead of a normal comparison operator (==).To use these functions, you would incorporate them into existing networking or mes‐saging code. For example, with sockets, the server code might look something like this:

from socket import socket, AF_INET, SOCK_STREAM

secret_key = b'peekaboo'def echo_handler(client_sock):

> if not server_authenticate(client_sock, secret_key):client_sock.close()return> while True:
> 
> > > > msg = client_sock.recv(8192)if not msg:
> > > 
> > > break
> > 
> > client_sock.sendall(msg)

def echo_server(address):
s = socket(AF_INET, SOCK_STREAM)s.bind(address)s.listen(5)while True:

> c,a = s.accept()echo_handler(c)

echo_server((‘', 18000))

Within a client, you would do this:

from socket import socket, AF_INET, SOCK_STREAM

secret_key = b'peekaboo'

s = socket(AF_INET, SOCK_STREAM)s.connect((‘localhost', 18000))client_authenticate(s, secret_key)s.send(b'Hello World')resp = s.recv(1024)...

## 讨论

A common use of hmac authentication is in internal messaging systems and interprocesscommunication. For example, if you are writing a system that involves multiple pro‐cesses communicating across a cluster of machines, you can use this approach to makesure that only allowed processes are allowed to connect to one another. In fact, HMAC-based authentication is used internally by the multiprocessing library when it sets upcommunication with subprocesses.It’s important to stress that authenticating a connection is not the same as encryption.Subsequent communication on an authenticated connection is sent in the clear, andwould be visible to anyone inclined to sniff the traffic (although the secret key knownto both sides is never transmitted).The authentication algorithm used by hmac is based on cryptographic hashing functions,such as MD5 and SHA-1, and is described in detail in IETF RFC 2104.

# 11.10 在网络服务中加入 SSL

## 问题

You want to implement a network service involving sockets where servers and clientsauthenticate themselves and encrypt the transmitted data using SSL.

## 解决方案

The ssl module provides support for adding SSL to low-level socket connections. Inparticular, the ssl.wrap_socket() function takes an existing socket and wraps an SSLlayer around it. For example, here’s an example of a simple echo server that presents aserver certificate to connecting clients:

from socket import socket, AF_INET, SOCK_STREAMimport ssl

KEYFILE = ‘server_key.pem' # Private key of the serverCERTFILE = ‘server_cert.pem' # Server certificate (given to client)

def echo_client(s):while True:
data = s.recv(8192)if data == b'‘:

> break

s.send(data)

s.close()print(‘Connection closed')

def echo_server(address):
s = socket(AF_INET, SOCK_STREAM)s.bind(address)s.listen(1)

# Wrap with an SSL layer requiring client certss_ssl = ssl.wrap_socket(s,

> keyfile=KEYFILE,certfile=CERTFILE,server_side=True)

# Wait for connectionswhile True:

> try:c,a = s_ssl.accept()print(‘Got connection', c, a)echo_client(c)except Exception as e:print(‘{}: {}'.format(e.**class**.**name**, e))

echo_server((‘', 20000))

Here’s an interactive session that shows how to connect to the server as a client. Theclient requires the server to present its certificate and verifies it:

```py
      >>> from socket import socket, AF_INET, SOCK_STREAM
>>> import ssl
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s_ssl = ssl.wrap_socket(s,
...                         cert_reqs=ssl.CERT_REQUIRED,
...                         ca_certs = 'server_cert.pem')
>>> s_ssl.connect(('localhost', 20000))
>>> s_ssl.send(b'Hello World?')
12
>>> s_ssl.recv(8192)
b'Hello World?'
>>>

```

The problem with all of this low-level socket hacking is that it doesn’t play well withexisting network services already implemented in the standard library. For example,most server code (HTTP, XML-RPC, etc.) is actually based on the socketserver library.Client code is also implemented at a higher level. It is possible to add SSL to existingservices, but a slightly different approach is needed.First, for servers, SSL can be added through the use of a mixin class like this:

import ssl

class SSLMixin:
‘''Mixin class that adds support for SSL to existing servers basedon the socketserver module.‘''def **init**(self, *args,

> > > keyfile=None, certfile=None, ca_certs=None,cert_reqs=ssl.NONE,**kwargs):
> 
> self._keyfile = keyfileself._certfile = certfileself._ca_certs = ca_certsself._cert_reqs = cert_reqssuper().**init**(*args, **kwargs)

def get_request(self):
client, addr = super().get_request()client_ssl = ssl.wrap_socket(client,

> keyfile = self._keyfile,certfile = self._certfile,ca_certs = self._ca_certs,cert_reqs = self._cert_reqs,server_side = True)

return client_ssl, addr

To use this mixin class, you can mix it with other server classes. For example, here’s anexample of defining an XML-RPC server that operates over SSL:

# XML-RPC server with SSL

from xmlrpc.server import SimpleXMLRPCServer

class SSLSimpleXMLRPCServer(SSLMixin, SimpleXMLRPCServer):pass
Here’s the XML-RPC server from Recipe 11.6 modified only slightly to use SSL:

import sslfrom xmlrpc.server import SimpleXMLRPCServerfrom sslmixin import SSLMixin

class SSLSimpleXMLRPCServer(SSLMixin, SimpleXMLRPCServer):passclass KeyValueServer:
_rpc*methods* = [‘get', ‘set', ‘delete', ‘exists', ‘keys']def **init**(self, *args, **kwargs):

> > self._data = {}self._serv = SSLSimpleXMLRPCServer(*args, allow_none=True, **kwargs)for name in self._rpc*methods*:
> > 
> > self._serv.register_function(getattr(self, name))

def get(self, name):return self._data[name]def set(self, name, value):self._data[name] = valuedef delete(self, name):del self._data[name]def exists(self, name):return name in self._datadef keys(self):return list(self._data)def serve_forever(self):self._serv.serve_forever()if **name** == ‘**main**':
KEYFILE='server_key.pem' # Private key of the serverCERTFILE='server_cert.pem' # Server certificatekvserv = KeyValueServer((‘', 15000),

> keyfile=KEYFILE,certfile=CERTFILE),

kvserv.serve_forever()

To use this server, you can connect using the normal xmlrpc.client module. Just spec‐ify a https: in the URL. For example:

```py
      >>> from xmlrpc.client import ServerProxy
>>> s = ServerProxy('https://localhost:15000', allow_none=True)
>>> s.set('foo','bar')
>>> s.set('spam', [1, 2, 3])
>>> s.keys()
['spam', 'foo']
>>> s.get('foo')
'bar'
>>> s.get('spam')
[1, 2, 3]
>>> s.delete('spam')
>>> s.exists('spam')
False
>>>

```

One complicated issue with SSL clients is performing extra steps to verify the servercertificate or to present a server with client credentials (such as a client certificate).Unfortunately, there seems to be no standardized way to accomplish this, so research isoften required. However, here is an example of how to set up a secure XML-RPC con‐nection that verifies the server’s certificate:

from xmlrpc.client import SafeTransport, ServerProxyimport ssl

class VerifyCertSafeTransport(SafeTransport):def **init**(self, cafile, certfile=None, keyfile=None):
SafeTransport.**init**(self)self._ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLSv1)self._ssl_context.load_verify_locations(cafile)if cert:

> self._ssl_context.load_cert_chain(certfile, keyfile)

self._ssl_context.verify_mode = ssl.CERT_REQUIRED

def make_connection(self, host):

# Items in the passed dictionary are passed as keyword# arguments to the http.client.HTTPSConnection() constructor.# The context argument allows an ssl.SSLContext instance to# be passed with information about the SSL configurations = super().make_connection((host, {‘context': self._ssl_context}))

return s

# Create the client proxys = ServerProxy(‘[`localhost:15000`](https://localhost:15000)‘,

> transport=VerifyCertSafeTransport(‘server_cert.pem'),allow_none=True)

As shown, the server presents a certificate to the client and the client verifies it. Thisverification can go both directions. If the server wants to verify the client, change theserver startup to the following:if **name** == ‘**main**':

> > KEYFILE='server_key.pem' # Private key of the serverCERTFILE='server_cert.pem' # Server certificateCA_CERTS='client_cert.pem' # Certificates of accepted clients

kvserv = KeyValueServer((‘', 15000),keyfile=KEYFILE,certfile=CERTFILE,ca_certs=CA_CERTS,cert_reqs=ssl.CERT_REQUIRED,)> kvserv.serve_forever()

To make the XML-RPC client present its certificates, change the ServerProxy initiali‐zation to this:

# Create the client proxys = ServerProxy(‘[`localhost:15000`](https://localhost:15000)‘,

> transport=VerifyCertSafeTransport(‘server_cert.pem',‘client_cert.pem',‘client_key.pem'),> allow_none=True)

## 讨论

Getting this recipe to work will test your system configuration skills and understandingof SSL. Perhaps the biggest challenge is simply getting the initial configuration of keys,certificates, and other matters in order.To clarify what’s required, each endpoint of an SSL connection typically has a privatekey and a signed certificate file. The certificate file contains the public key and is pre‐sented to the remote peer on each connection. For public servers, certificates are nor‐mally signed by a certificate authority such as Verisign, Equifax, or similar organization(something that costs money). To verify server certificates, clients maintain a file con‐taining the certificates of trusted certificate authorities. For example, web browsersmaintain certificates corresponding to the major certificate authorities and use them toverify the integrity of certificates presented by web servers during HTTPS connections.For the purposes of this recipe, you can create what’s known as a self-signed certificate.Here’s how you do it:

bash % openssl req -new -x509 -days 365 -nodes -out server_cert.pem -keyout server_key.pem
Generating a 1024 bit RSA private key..........................................++++++...++++++

writing new private key to ‘server_key.pem'
You are about to be asked to enter information that will be incorporatedinto your certificate request.What you are about to enter is what is called a Distinguished Name or a DN.There are quite a few fields but you can leave some blankFor some fields there will be a default value,If you enter ‘.', the field will be left blank.

Country Name (2 letter code) [AU]:USState or Province Name (full name) [Some-State]:IllinoisLocality Name (eg, city) []:ChicagoOrganization Name (eg, company) [Internet Widgits Pty Ltd]:Dabeaz, LLCOrganizational Unit Name (eg, section) []:Common Name (eg, YOUR name) []:localhostEmail Address []:bash %

When creating the certificate, the values for the various fields are often arbitrary. How‐ever, the “Common Name” field often contains the DNS hostname of servers. If you’rejust testing things out on your own machine, use “localhost.” Otherwise, use the domainname of the machine that’s going to run the server.As a result of this configuration, you will have a server_key.pem file that contains theprivate key. It looks like this:

> —–BEGIN RSA PRIVATE KEY—–MIICXQIBAAKBgQCZrCNLoEyAKF+f9UNcFaz5Osa6jf7qkbUl8si5xQrY3ZYC7juunL1dZLn/VbEFIITaUOgvBtPv1qUWTJGwga62VSG1oFE0ODIx3g2Nh4sRf+rySsx2L4442nx0z4O5vJQ7k6eRNHAZUUnCL50+YvjyLyt7ryLSjSuKhCcJsbZgPwIDAQABAoGAB5evrr7eyL4160tM5rHTeATlaLY3UBOe5Z8XN8Z6gLiB/ucSX9AysviVD/6F3oD6z2aL8jbeJc1vHqjt0dC2dwwm32vVl8mRdyoAsQpWmiqXrkvP4Bsl04VpBeHwQt8xNSW9SFhceL3LEvw9M8i9MV39viih1ILyH8OuHdvJyFECQQDLEjl2d2ppxND9PoLqVFAirDfX2JnLTdWbc+M11a9Jdn3hKF8TcxfEnFVs5Gav1MusicY5KB0ylYPbYbTvqKc7AkEAwbnRBO2VYEZsJZp2X0IZqP9ovWokkpYx+PE4+c6MySDgaMcigL7vWDIHJG1CHudD09GbqENasDzyb2HAIW4CzQJBAKDdkv+xoW6gJx42Auc2WzTcUHCAeXR/+BLpPrhKykzbvOQ8YvS5W764SUO1u1LWs3G+wnRMvrRvlMCZKgggBjkCQQCGJewto2+a+WkOKQXrNNScCDE5aPTmZQc5waCYq4UmCZQcOjkUOiN3ST1U5iuxRqfbV/yX6fw0qh+fLWtkOs/JAkA+okMSxZwqRtfgOFGBfwQ8/iKrnizeanTQ3L6scFXICHZXdJ3XQ6qUmNxNn7iJ7S/LDawo1QfWkCfD9FYoxBlg—–END RSA PRIVATE KEY—–

The server certificate in server_cert.pem looks similar:

> > —–BEGIN CERTIFICATE—–MIIC+DCCAmGgAwIBAgIJAPMd+vi45js3MA0GCSqGSIb3DQEBBQUAMFwxCzAJBgNVBAYTAlVTMREwDwYDVQQIEwhJbGxpbm9pczEQMA4GA1UEBxMHQ2hpY2FnbzEUMBIGA1UEChMLRGFiZWF6LCBMTEMxEjAQBgNVBAMTCWxvY2FsaG9zdDAeFw0xMzAxMTExODQyMjdaFw0xNDAxMTExODQyMjdaMFwxCzAJBgNVBAYTAlVTMREwDwYDVQQIEwhJbGxpbm9pczEQMA4GA1UEBxMHQ2hpY2FnbzEUMBIGA1UEChMLRGFiZWF6LCBMTEMxEjAQBgNVBAMTCWxvY2FsaG9zdDCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEAmawjS6BMgChfn/VDXBWs+TrGuo3+6pG1JfLIucUK2N2WAu47rpy9XWS5/1WxBSCE2lDoLwbT79alFkyRsIGutlUhtaBRNDgyMd4NjYeLEX/q8krMdi+OONp8dM+DubyU
> 
> O5OnkTRwGVFJwi+dPmL48i8re68i0o0rioQnCbG2YD8CAwEAAaOBwTCBvjAdBgNVHQ4EFgQUrtoLHHgXiDZTr26NMmgKJLJLFtIwgY4GA1UdIwSBhjCBg4AUrtoLHHgXiDZTr26NMmgKJLJLFtKhYKReMFwxCzAJBgNVBAYTAlVTMREwDwYDVQQIEwhJbGxpbm9pczEQMA4GA1UEBxMHQ2hpY2FnbzEUMBIGA1UEChMLRGFiZWF6LCBMTEMxEjAQBgNVBAMTCWxvY2FsaG9zdIIJAPMd+vi45js3MAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQEFBQADgYEAFci+dqvMG4xF8UTnbGVvZJPIzJDRee6Nbt6AHQo9pOdAIMAuWsGCplSOaDNdKKzl+b2UT2Zp3AIW4Qd51bouSNnR4M/gnr9ZD1ZctFd3jS+C5XRpD3vvcW5lAnCCC80P6rXy7d7hTeFu5EYKtRGXNvVNd/06NALGDflrrOwxF3Y=—–END CERTIFICATE—–

In server-related code, both the private key and certificate file will be presented to thevarious SSL-related wrapping functions. The certificate is what gets presented to clients.The private key should be protected and remains on the server.In client-related code, a special file of valid certificate authorities needs to be maintainedto verify the server’s certificate. If you have no such file, then at the very least, you canput a copy of the server’s certificate on the client machine and use that as a means forverification. During connection, the server will present its certificate, and then you’lluse the stored certificate you already have to verify that it’s correct.Servers can also elect to verify the identity of clients. To do that, clients need to havetheir own private key and certificate key. The server would also need to maintain a fileof trusted certificate authorities for verifying the client certificates.If you intend to add SSL support to a network service for real, this recipe really onlygives a small taste of how to set it up. You will definitely want to consult the documen‐tation for more of the finer points. Be prepared to spend a significant amount of timeexperimenting with it to get things to work.

# 11.11 进程间传递 Socket 文件描述符

## 问题

You have multiple Python interpreter processes running and you want to pass an openfile descriptor from one interpreter to the other. For instance, perhaps there is a serverprocess that is responsible for receiving connections, but the actual servicing of clientsis to be handled by a different interpreter.

## 解决方案

To pass a file descriptor between processes, you first need to connect the processestogether. On Unix machines, you might use a Unix domain socket, whereas on Win‐dows, you could use a named pipe. However, rather than deal with such low-levelmechanics, it is often easier to use the multiprocessing module to set up such aconnection.

Once a connection is established, you can use the send_handle() and recv_handle()functions in multiprocessing.reduction to send file descriptors between processes.The following example illustrates the basics:

import multiprocessingfrom multiprocessing.reduction import recv_handle, send_handleimport socket

def worker(in_p, out_p):
out_p.close()while True:

> > fd = recv_handle(in_p)print(‘CHILD: GOT FD', fd)with socket.socket(socket.AF_INET, socket.SOCK_STREAM, fileno=fd) as s:
> > 
> > while True:> > msg = s.recv(1024)if not msg:
> > 
> > > break
> > 
> > print(‘CHILD: RECV {!r}'.format(msg))s.send(msg)

def server(address, in_p, out_p, worker_pid):
in_p.close()s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)s.bind(address)s.listen(1)while True:

> client, addr = s.accept()print(‘SERVER: Got connection from', addr)send_handle(out_p, client.fileno(), worker_pid)client.close()

if **name** == ‘**main**':
c1, c2 = multiprocessing.Pipe()worker_p = multiprocessing.Process(target=worker, args=(c1,c2))worker_p.start()

server_p = multiprocessing.Process(target=server,args=((‘', 15000), c1, c2, worker_p.pid))
server_p.start()

c1.close()c2.close()

In this example, two processes are spawned and connected by a multiprocessing Pipeobject. The server process opens a socket and waits for client connections. The workerprocess merely waits to receive a file descriptor on the pipe using recv_handle(). Whenthe server receives a connection, it sends the resulting socket file descriptor to the worker

using send_handle(). The worker takes over the socket and echoes data back to theclient until the connection is closed.If you connect to the running server using Telnet or a similar tool, here is an exampleof what you might see:

> bash % python3 passfd.pySERVER: Got connection from (‘127.0.0.1', 55543)CHILD: GOT FD 7CHILD: RECV b'Hellorn'CHILD: RECV b'Worldrn'

The most important part of this example is the fact that the client socket accepted in theserver is actually serviced by a completely different process. The server merely hands itoff, closes it, and waits for the next connection.

## 讨论

Passing file descriptors between processes is something that many programmers don’teven realize is possible. However, it can sometimes be a useful tool in building scalablesystems. For example, on a multicore machine, you could have multiple instances of thePython interpreter and use file descriptor passing to more evenly balance the numberof clients being handled by each interpreter.The send_handle() and recv_handle() functions shown in the solution really onlywork with multiprocessing connections. Instead of using a pipe, you can connect in‐terpreters as shown in Recipe 11.7, and it will work as long as you use UNIX domainsockets or Windows pipes. For example, you could implement the server and workeras completely separate programs to be started separately. Here is the implementation ofthe server:

# servermp.pyfrom multiprocessing.connection import Listenerfrom multiprocessing.reduction import send_handleimport socket

def server(work_address, port):

# Wait for the worker to connectwork_serv = Listener(work_address, authkey=b'peekaboo')worker = work_serv.accept()worker_pid = worker.recv()

# Now run a TCP/IP server and send clients to workers = socket.socket(socket.AF_INET, socket.SOCK_STREAM)s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)s.bind((‘', port))s.listen(1)while True:

> > client, addr = s.accept()print(‘SERVER: Got connection from', addr)
> 
> send_handle(worker, client.fileno(), worker_pid)client.close()

if **name** == ‘**main**':
import sysif len(sys.argv) != 3:

> print(‘Usage: server.py server_address port', file=sys.stderr)raise SystemExit(1)

server(sys.argv[1], int(sys.argv[2]))

To run this server, you would run a command such as python3 servermp.py /tmp/servconn 15000\. Here is the corresponding client code:

# workermp.py

from multiprocessing.connection import Clientfrom multiprocessing.reduction import recv_handleimport osfrom socket import socket, AF_INET, SOCK_STREAM

def worker(server_address):
serv = Client(server_address, authkey=b'peekaboo')serv.send(os.getpid())while True:

> > fd = recv_handle(serv)print(‘WORKER: GOT FD', fd)with socket(AF_INET, SOCK_STREAM, fileno=fd) as client:
> > 
> > while True:> > msg = client.recv(1024)if not msg:
> > 
> > > break
> > 
> > print(‘WORKER: RECV {!r}'.format(msg))client.send(msg)

if **name** == ‘**main**':
import sysif len(sys.argv) != 2:

> print(‘Usage: worker.py server_address', file=sys.stderr)raise SystemExit(1)

worker(sys.argv[1])

To run the worker, you would type python3 workermp.py /tmp/servconn. The result‐ing operation should be exactly the same as the example that used Pipe().Under the covers, file descriptor passing involves creating a UNIX domain socket andthe sendmsg() method of sockets. Since this technique is not widely known, here is adifferent implementation of the server that shows how to pass descriptors using sockets:

# server.pyimport socket

import struct

def send_fd(sock, fd):
‘''Send a single file descriptor.‘''sock.sendmsg([b'x'],

> [(socket.SOL_SOCKET, socket.SCM_RIGHTS, struct.pack(‘i', fd))])

ack = sock.recv(2)assert ack == b'OK'

def server(work_address, port):

# Wait for the worker to connectwork_serv = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)work_serv.bind(work_address)work_serv.listen(1)worker, addr = work_serv.accept()

# Now run a TCP/IP server and send clients to workers = socket.socket(socket.AF_INET, socket.SOCK_STREAM)s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)s.bind((‘',port))s.listen(1)while True:

> client, addr = s.accept()print(‘SERVER: Got connection from', addr)send_fd(worker, client.fileno())client.close()

if **name** == ‘**main**':
import sysif len(sys.argv) != 3:

> print(‘Usage: server.py server_address port', file=sys.stderr)raise SystemExit(1)

server(sys.argv[1], int(sys.argv[2]))

Here is an implementation of the worker using sockets:

# worker.pyimport socketimport struct

def recv_fd(sock):
‘''Receive a single file descriptor‘''msg, ancdata, flags, addr = sock.recvmsg(1,

> socket.CMSG_LEN(struct.calcsize(‘i')))

cmsg_level, cmsg_type, cmsg_data = ancdata[0]assert cmsg_level == socket.SOL_SOCKET and cmsg_type == socket.SCM_RIGHTSsock.sendall(b'OK')

return struct.unpack(‘i', cmsg_data)[0]

def worker(server_address):
serv = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)serv.connect(server_address)while True:

> > fd = recv_fd(serv)print(‘WORKER: GOT FD', fd)with socket.socket(socket.AF_INET, socket.SOCK_STREAM, fileno=fd) as client:
> > 
> > while True:> > msg = client.recv(1024)if not msg:
> > 
> > > break
> > 
> > print(‘WORKER: RECV {!r}'.format(msg))client.send(msg)

if **name** == ‘**main**':
import sysif len(sys.argv) != 2:

> print(‘Usage: worker.py server_address', file=sys.stderr)raise SystemExit(1)

worker(sys.argv[1])

If you are going to use file-descriptor passing in your program, it is advisable to readmore about it in an advanced text, such as Unix Network Programming by W. RichardStevens (Prentice Hall, 1990). Passing file descriptors on Windows uses a differenttechnique than Unix (not shown). For that platform, it is advisable to study the sourcecode to multiprocessing.reduction in close detail to see how it works.

# 11.12 理解事件驱动的 IO

## 问题

You have heard about packages based on “event-driven” or “asynchronous” I/O, butyou’re not entirely sure what it means, how it actually works under the covers, or howit might impact your program if you use it.

## 解决方案

At a fundamental level, event-driven I/O is a technique that takes basic I/O operations(e.g., reads and writes) and converts them into events that must be handled by yourprogram. For example, whenever data was received on a socket, it turns into a “receive”event that is handled by some sort of callback method or function that you supply torespond to it. As a possible starting point, an event-driven framework might start witha base class that implements a series of basic event handler methods like this:

class EventHandler:def fileno(self):‘Return the associated file descriptor'raise NotImplemented(‘must implement')def wants_to_receive(self):‘Return True if receiving is allowed'return Falsedef handle_receive(self):‘Perform the receive operation'passdef wants_to_send(self):‘Return True if sending is requested'return Falsedef handle_send(self):‘Send outgoing data'pass
Instances of this class then get plugged into an event loop that looks like this:

import select

def event_loop(handlers):while True:
wants_recv = [h for h in handlers if h.wants_to_receive()]wants_send = [h for h in handlers if h.wants_to_send()]can_recv, can*send,* = select.select(wants_recv, wants_send, [])for h in can_recv:

> h.handle_receive()

for h in can_send:h.handle_send()
That’s it! The key to the event loop is the select() call, which polls file descriptors foractivity. Prior to calling select(), the event loop simply queries all of the handlers tosee which ones want to receive or send. It then supplies the resulting lists to select().As a result, select() returns the list of objects that are ready to receive or send. Thecorresponding handle_receive() or handle_send() methods are triggered.To write applications, specific instances of EventHandler classes are created. For ex‐ample, here are two simple handlers that illustrate two UDP-based network services:

import socketimport time

class UDPServer(EventHandler):def **init**(self, address):self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)self.sock.bind(address)def fileno(self):return self.sock.fileno()def wants_to_receive(self):return Trueclass UDPTimeServer(UDPServer):def handle_receive(self):msg, addr = self.sock.recvfrom(1)self.sock.sendto(time.ctime().encode(‘ascii'), addr)class UDPEchoServer(UDPServer):def handle_receive(self):msg, addr = self.sock.recvfrom(8192)self.sock.sendto(msg, addr)if **name** == ‘**main**':handlers = [ UDPTimeServer((‘',14000)), UDPEchoServer((‘',15000)) ]event_loop(handlers)
To test this code, you can try connecting to it from another Python interpreter:

```py
      >>> from socket import *
>>> s = socket(AF_INET, SOCK_DGRAM)
>>> s.sendto(b'',('localhost',14000))
0
>>> s.recvfrom(128)
(b'Tue Sep 18 14:29:23 2012', ('127.0.0.1', 14000))
>>> s.sendto(b'Hello',('localhost',15000))
5
>>> s.recvfrom(128)
(b'Hello', ('127.0.0.1', 15000))
>>>

```

Implementing a TCP server is somewhat more complex, since each client involves theinstantiation of a new handler object. Here is an example of a TCP echo client.

class TCPServer(EventHandler):def **init**(self, address, client_handler, handler_list):self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)self.sock.bind(address)self.sock.listen(1)self.client_handler = client_handlerself.handler_list = handler_listdef fileno(self):return self.sock.fileno()def wants_to_receive(self):return Truedef handle_receive(self):client, addr = self.sock.accept()# Add the client to the event loop's handler listself.handler_list.append(self.client_handler(client, self.handler_list))class TCPClient(EventHandler):def **init**(self, sock, handler_list):self.sock = sockself.handler_list = handler_listself.outgoing = bytearray()def fileno(self):return self.sock.fileno()def close(self):self.sock.close()# Remove myself from the event loop's handler listself.handler_list.remove(self)def wants_to_send(self):return True if self.outgoing else Falsedef handle_send(self):nsent = self.sock.send(self.outgoing)self.outgoing = self.outgoing[nsent:]class TCPEchoClient(TCPClient):def wants_to_receive(self):return Truedef handle_receive(self):
data = self.sock.recv(8192)if not data:

> self.close()

else:self.outgoing.extend(data)if **name** == ‘**main**':handlers = []handlers.append(TCPServer((‘',16000), TCPEchoClient, handlers))event_loop(handlers)
The key to the TCP example is the addition and removal of clients from the handler list.On each connection, a new handler is created for the client and added to the list. Whenthe connection is closed, each client must take care to remove themselves from the list.If you run this program and try connecting with Telnet or some similar tool, you’ll seeit echoing received data back to you. It should easily handle multiple clients.

## 讨论

Virtually all event-driven frameworks operate in a manner that is similar to that shownin the solution. The actual implementation details and overall software architecturemight vary greatly, but at the core, there is a polling loop that checks sockets for activityand which performs operations in response.One potential benefit of event-driven I/O is that it can handle a very large number ofsimultaneous connections without ever using threads or processes. That is, the select() call (or equivalent) can be used to monitor hundreds or thousands of socketsand respond to events occuring on any of them. Events are handled one at a time by theevent loop, without the need for any other concurrency primitives.The downside to event-driven I/O is that there is no true concurrency involved. If anyof the event handler methods blocks or performs a long-running calculation, it blocksthe progress of everything. There is also the problem of calling out to library functionsthat aren’t written in an event-driven style. There is always the risk that some librarycall will block, causing the event loop to stall.Problems with blocking or long-running calculations can be solved by sending the workout to a separate thread or process. However, coordinating threads and processes withan event loop is tricky. Here is an example of code that will do it using the concurrent.futures module:

from concurrent.futures import ThreadPoolExecutorimport os

class ThreadPoolHandler(EventHandler):def **init**(self, nworkers):if os.name == ‘posix':self.signal_done_sock, self.done_sock = socket.socketpair()else:
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)server.bind((‘127.0.0.1', 0))server.listen(1)self.signal_done_sock = socket.socket(socket.AF_INET,

> socket.SOCK_STREAM)

self.signal_done_sock.connect(server.getsockname())self.done*sock,* = server.accept()server.close()

self.pending = []self.pool = ThreadPoolExecutor(nworkers)

def fileno(self):return self.done_sock.fileno()

# Callback that executes when the thread is donedef _complete(self, callback, r):

> self.pending.append((callback, r.result()))self.signal_done_sock.send(b'x')

# Run a function in a thread pooldef run(self, func, args=(), kwargs={},*,callback):

> r = self.pool.submit(func, *args, **kwargs)r.add_done_callback(lambda r: self._complete(callback, r))

def wants_to_receive(self):return True

# Run callback functions of completed workdef handle_receive(self):

> > # Invoke all pending callback functionsfor callback, result in self.pending:
> > 
> > callback(result)self.done_sock.recv(1)
> 
> self.pending = []

In this code, the run() method is used to submit work to the pool along with a callbackfunction that should be triggered upon completion. The actual work is then submittedto a ThreadPoolExecutor instance. However, a really tricky problem concerns the co‐ordination of the computed result and the event loop. To do this, a pair of sockets arecreated under the covers and used as a kind of signaling mechanism. When work iscompleted by the thread pool, it executes the _complete() method in the class. Thismethod queues up the pending callback and result before writing a byte of data on oneof these sockets. The fileno() method is programmed to return the other socket. Thus,when this byte is written, it will signal to the event loop that something has happened.The handle_receive() method, when triggered, will then execute all of the callbackfunctions for previously submitted work. Frankly, it’s enough to make one’s head spin.Here is a simple server that shows how to use the thread pool to carry out a long-runningcalculation:

# A really bad Fibonacci implementationdef fib(n):

> if n < 2:return 1else:return fib(n - 1) + fib(n - 2)

class UDPFibServer(UDPServer):def handle_receive(self):msg, addr = self.sock.recvfrom(128)n = int(msg)pool.run(fib, (n,), callback=lambda r: self.respond(r, addr))def respond(self, result, addr):self.sock.sendto(str(result).encode(‘ascii'), addr)if **name** == ‘**main**':pool = ThreadPoolHandler(16)handlers = [ pool, UDPFibServer((‘',16000))]event_loop(handlers)
To try this server, simply run it and try some experiments with another Python program:

from socket import *sock = socket(AF_INET, SOCK_DGRAM)for x in range(40):

> sock.sendto(str(x).encode(‘ascii'), (‘localhost', 16000))resp = sock.recvfrom(8192)print(resp[0])

You should be able to run this program repeatedly from many different windows andhave it operate without stalling other programs, even though it gets slower and sloweras the numbers get larger.Having gone through this recipe, should you use its code? Probably not. Instead, youshould look for a more fully developed framework that accomplishes the same task.However, if you understand the basic concepts presented here, you’ll understand thecore techniques used to make such frameworks operate. As an alternative to callback-based programming, event-driven code will sometimes use coroutines. See Recipe 12.12for an example.

# 11.13 发送与接收大型数组

## 问题

You want to send and receive large arrays of contiguous data across a network connec‐tion, making as few copies of the data as possible.

## 解决方案

The following functions utilize memoryviews to send and receive large arrays:

# zerocopy.py

def send_from(arr, dest):
view = memoryview(arr).cast(‘B')while len(view):

> nsent = dest.send(view)view = view[nsent:]

def recv_into(arr, source):
view = memoryview(arr).cast(‘B')while len(view):

> nrecv = source.recv_into(view)view = view[nrecv:]

To test the program, first create a server and client program connected over a socket.In the server:

```py
      >>> from socket import *
>>> s = socket(AF_INET, SOCK_STREAM)
>>> s.bind(('', 25000))
>>> s.listen(1)
>>> c,a = s.accept()
>>>

```

In the client (in a separate interpreter):

```py
      >>> from socket import *
>>> c = socket(AF_INET, SOCK_STREAM)
>>> c.connect(('localhost', 25000))
>>>

```

Now, the whole idea of this recipe is that you can blast a huge array through the con‐nection. In this case, arrays might be created by the array module or perhaps numpy.For example:# Server>>> import numpy>>> a = numpy.arange(0.0, 50000000.0)>>> send_from(a, c)>>>

# Client>>> import numpy>>> a = numpy.zeros(shape=50000000, dtype=float)>>> a[0:10]array([ 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.])>>> recv_into(a, c)>>> a[0:10]array([ 0., 1., 2., 3., 4., 5., 6., 7., 8., 9.])>>>

## 讨论

In data-intensive distributed computing and parallel programming applications, it’s notuncommon to write programs that need to send/receive large chunks of data. However,to do this, you somehow need to reduce the data down to raw bytes for use with low-level network functions. You may also need to slice the data into chunks, since mostnetwork-related functions aren’t able to send or receive huge blocks of data entirely allat once.One approach is to serialize the data in some way—possibly by converting into a bytestring. However, this usually ends up making a copy of the data. Even if you do thispiecemeal, your code still ends up making a lot of little copies.

This recipe gets around this by playing a sneaky trick with memoryviews. Essentially, amemoryview is an overlay of an existing array. Not only that, memoryviews can be castto different types to allow interpretation of the data in a different manner. This is thepurpose of the following statement:view = memoryview(arr).cast(‘B')

It takes an array arr and casts into a memoryview of unsigned bytes.In this form, the view can be passed to socket-related functions, such as sock.send()or send.recv_into(). Under the covers, those methods are able to work directly withthe memory region. For example, sock.send() sends data directly from memorywithout a copy. send.recv_into() uses the memoryview as the input buffer for thereceive operation.The remaining complication is the fact that the socket functions may only work withpartial data. In general, it will take many different send() and recv_into() calls totransmit the entire array. Not to worry. After each operation, the view is sliced by thenumber of sent or received bytes to produce a new view. The new view is also a memoryoverlay. Thus, no copies are made.One issue here is that the receiver has to know in advance how much data will be sentso that it can either preallocate an array or verify that it can receive the data into anexisting array. If this is a problem, the sender could always arrange to send the size first,followed by the array data.