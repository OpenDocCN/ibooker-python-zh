# 附录. Python 3.7 至 3.11 的新功能和更改

下表列出了 Python 版本 3.7 到 3.11 中语言和标准库的变更，这些变更最可能出现在 Python 代码中。使用这些表格来规划您的升级策略，受您代码库中断变更的限制。

以下类型的更改被视为“breaking”，并在最后一列标有 **!** 符号：

+   引入新关键字或内置函数（可能与现有 Python 源代码中使用的名称冲突）

+   从标准库模块或内置类型中删除方法

+   更改内置或标准库方法的签名，这种更改不向后兼容（例如删除参数或重命名命名参数）

新警告（包括 DeprecatedWarning）也显示为“breaking”，但在最后一列用 ***** 符号标记。

另请参阅标准库中拟议的弃用和移除表（“死电池”）在 [PEP 594](https://oreil.ly/4Sy73) 中列出，该表列出了计划在哪些版本中删除的模块（从 Python 3.12 开始）及其推荐的替代方案。

# Python 3.7

以下表格总结了 Python 版本 3.7 的更改。更多细节请参阅[在线文档](https://oreil.ly/iIePL)中的“Python 3.7 新特性”。

| Python 3.7 | Added | Deprecated | Removed | Breaking change |
| --- | --- | --- | --- | --- |
| 函数接受 > 255 个参数 | **+** |   |   |   |
| argparse.ArgumentParser.parse_intermixed_args() | **+** |   |   |   |
| ast.literal_eval() 不再评估加法和减法 |   |   |   | **!** |
| **async** 和 **await** 成为保留语言关键字 | **+** |   |   | **!** |

| asyncio.all_tasks(), asyncio.create_task(),

asyncio.current_task(),

asyncio.get_running_loop(),

asyncio.Future.get_loop(),

asyncio.Handle.cancelled(),

asyncio.loop.sock_recv_into(),

asyncio.loop.sock_sendfile(),

asyncio.loop.start_tls(),

asyncio.ReadTransport.is_reading(),

asyncio.Server.is_serving(),

asyncio.Server.get_loop(),

asyncio.Task.get_loop(),

asyncio.run()（暂定） | **+** |   |   |   |

| asyncio.Server 是异步上下文管理器 | **+** |   |   |   |
| --- | --- | --- | --- | --- |

| asyncio.loop.call_soon(), asyncio.loop.call_soon_threadsafe(),

asyncio.loop.call_later(),

asyncio.loop.call_at(), and

asyncio.Future.add_done_callback() 都接受可选的命名 context 参数 | **+** |   |   |   |

| asyncio.loop.create_server(), asyncio.loop.create_unix_server(),

asyncio.Server.start_serving(),

并且 asyncio.Server.serve_forever() 都接受可选的命名 start_serving 参数 | **+** |   |   |   |

| asyncio.Task.current_task() 和 asyncio.Task.all_tasks() 已弃用；请使用 asyncio.current_task() 和

asyncio.all_tasks() |   | **—** |   | ***** |

| binascii.b2a_uu() 接受命名的 backtick 参数 | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| bool() 构造函数不再接受命名参数（仅位置参数） |   |   |   | **!** |
| breakpoint() built-in function | **+** |   |   | **!** |
| bytearray.isascii() | **+** |   |   |   |
| bytes.isascii() | **+** |   |   |   |
| collections.namedtuple supports default values | **+** |   |   |   |
| concurrent.Futures.ProcessPoolExecutor and concurrent.Futures.ThreadPoolExecutor constructors accept optional initializer and initargs arguments | **+** |   |   |   |

-   contextlib.AbstractAsyncContextManager, contextlib.asynccontextmanager(),

-   contextlib.AsyncExitStack,

contextlib.nullcontext() | **+** |   |   |   |

| contextvars module (similar to thread-local vars, with asyncio support) | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| dataclasses module | **+** |   |   |   |
| datetime.datetime.fromisoformat() | **+** |   |   |   |
| DeprecationWarning shown by default in __main__ module | **+** |   |   | ***** |
| dict maintaining insertion order now guaranteed; dict.popitem() returns items in LIFO order | **+** |   |   |   |
| __dir__() at module level | **+** |   |   |   |
| dis.dis() method accepts named depth argument | **+** |   |   |   |
| float() constructor no longer accepts a named argument (positional only) |   |   |   | **!** |
| fpectl module removed |   |   | **X** | **!** |
| **from** __future__ **import** annotations enables referencing as-yet-undefined types in type annotations without enclosing in quotes | **+** |   |   |   |
| gc.freeze() | **+** |   |   |   |
| __getattr__() at module level | **+** |   |   |   |
| hmac.digest() | **+** |   |   |   |
| http.client.HTTPConnection and http.client.HTTPSConnection constructors accept optional blocksize argument | **+** |   |   |   |
| http.server.ThreadingHTTPServer | **+** |   |   |   |

-   importlib.abc.ResourceReader, importlib.resources module,

-   importlib.source_hash() | **+** |   |   |   |

| int() constructor no longer accepts a named *x* argument (positional only; named base argument is still supported) |   |   |   | **!** |
| --- | --- | --- | --- | --- |
| io.TextIOWrapper.reconfigure() | **+** |   |   |   |
| ipaddress.IPv*Network.subnet_of(), ipaddress.IPv*Network.supernet_of() | **+** |   |   |   |
| list() constructor no longer accepts a named argument (positional only) |   |   |   | **!** |
| logging.StreamHandler.setStream() | **+** |   |   |   |
| math.remainder() | **+** |   |   |   |
| multiprocessing.Process.close(), multiprocessing.Process.kill() | **+** |   |   |   |
| ntpath.splitunc() removed; use ntpath.splitdrive() |   |   | **X** | **!** |
| os.preadv(), os.pwritev(), os.register_at_fork() | **+** |   |   |   |
| os.stat_float_times() removed (compatibility function with Python 2; all timestamps in stat result are floats in Python 3) |   |   | **X** | **!** |
| pathlib.Path.is_mount() | **+** |   |   |   |
| pdb.set_trace() accepts named header argument | **+** |   |   |   |

-   plist.Dict, plist.Plist, and

plist._InternalDict removed |   |   | **X** | **!** |

| queue.SimpleQueue | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| re 编译表达式和匹配对象可以使用 copy.copy 和 copy.deepcopy 进行复制 | **+** |   |   |   |
| re.sub() no longer supports unknown escapes of \ and an ASCII letter |   |   | **X** | **!** |

| socket.close(), socket.getblocking(), socket.TCP_CONGESTION,

socket.TCP_USER_TIMEOUT,

socket.TCP_NOTSENT_LOWAT（仅限 Linux 平台）| **+** |   |   |   |

| sqlite3.Connection.backup() | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| 生成器中的 StopIteration 处理 | **+** |   |   |   |
| str.isascii() | **+** |   |   |   |
| subprocess.run() 的命名参数 capture_output=**True** 简化了 stdin/stdout 捕获 | **+** |   |   |   |

| subprocess.run() 和 subprocess.Popen() 的命名参数 text,

通用换行符的别名 | **+** |   |   |   |

| subprocess.run(), subprocess.call(), 和 subprocess.Popen() 已改进

中断处理 | **+** |   |   |   |

| sys.breakpointhook(), sys.getandroidapilevel(),

sys.get_coroutine_origin_tracking_depth(),

sys.set_coroutine_origin_tracking_depth() | **+** |   |   |   |

| time.clock_gettime_ns(), time.clock_settime_ns(),

time.monotonic_ns(),

time.perf_counter_ns(),

time.process_time_ns(), time.time_ns(),

time.CLOCK_BOOTTIME, time.CLOCK_PROF,

time.CLOCK_UPTIME | **+** |   |   |   |

| time.thread_time() 和 time.thread_time_ns() 用于线程级别的 CPU 计时 | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| tkinter.ttk.Spinbox | **+** |   |   |   |
| tuple() 构造函数不再接受命名参数（仅限位置参数） |   |   |   | **!** |

| types.ClassMethodDescriptorType, types.MethodDescriptorType, types.MethodWrapperType,

types.WrapperDescriptorType | **+** |   |   |   |

| types.resolve_bases() | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| uuid.UUID.is_safe | **+** |   |   |   |
| **yield** 和 **yield** **from** 在推导式或生成器表达式中已废弃 |   | **—** |   | ***** |
| zipfile.ZipFile 构造函数接受了命名的 compresslevel 参数 | **+** |   |   |   |

# Python 3.8

以下表格总结了 Python 3.8 版本中的变更。更多详情请参见[在线文档](https://oreil.ly/wSZXj)中的“What’s New in Python 3.8”。

| Python 3.8 | 添加 | 废弃 | 移除 | 破坏性变更 |
| --- | --- | --- | --- | --- |
| 赋值表达式（:= “海象” 运算符） | **+** |   |   |   |
| 位置参数和命名参数（/ 和 * 参数分隔符） | **+** |   |   |   |
| F-string trailing = for debugging | **+** |   |   |   |
| 对于 str 和 int 字面量的 **is** 和 **is not** 测试会发出 SyntaxWarning |   |   |   | ***** |
| ast AST 节点的 end_lineno 和 end_col_offset 属性 | **+** |   |   |   |
| ast.get_source_segment() | **+** |   |   |   |
| ast.parse() 现在接受了命名参数 type_comments, mode, 和 feature_version | **+** |   |   |   |
| **async** REPL 可通过 **python -m asyncio** 运行 | **+** |   |   |   |
| asyncio 任务可以命名 | **+** |   |   |   |
| asyncio.coroutine 装饰器已废弃 |   | **—** |   | ***** |
| asyncio.run() 可直接执行协程 | **+** |   |   |   |
| asyncio.Task.get_coro() | **+** |   |   |   |
| bool.as_integer_ratio() | **+** |   |   |   |
| collections.namedtuple._asdict() returns dict instead of OrderedDict | **+** |   |   |   |
| **continue** permitted in **finally** block | **+** |   |   |   |
| cgi.parse_qs, cgi.parse_qsl, and cgi.escape removed; import from urllib.parse and html modules |   |   | **X** | **!** |
| csv.DictReader returns dicts instead of OrderedDicts | **+** |   |   |   |
| datetime.date.fromisocalendar(), datetime.datetime.fromisocalendar() | **+** |   |   |   |
| dict comprehensions compute key first, value second |   |   |   | **!** |
| dict and dictviews returned from dict.keys(), dict.values() and dict.items() now iterable with reversed() | **+** |   |   |   |
| fractions.Fraction.as_integer_ratio() | **+** |   |   |   |
| functools.cached_property() decorator (see cautionary notes [here](https://oreil.ly/s3V1X) and [here](https://oreil.ly/svOZb)) | **+** |   |   |   |
| functools.lru_cache can be used as a decorator without () | **+** |   |   |   |
| functools.singledispatchmethod decorator | **+** |   |   |   |
| gettext.pgettext() | **+** |   |   |   |
| importlib.metadata module | **+** |   |   |   |
| int.as_integer_ratio() | **+** |   |   |   |
| itertools.accumulate() accepts named initial argument | **+** |   |   |   |
| macpath module removed |   |   | **X** | **!** |
| math.comb(), math.dist(), math.isqrt(), math.perm(), math.prod() | **+** |   |   |   |
| math.hypot() added support for > 2 dimensions | **+** |   |   |   |
| multiprocessing.shared_memory module | **+** |   |   |   |
| namedtuple._asdict() returns dict instead of OrderedDict | **+** |   |   |   |
| os.add_dll_directory() on Windows | **+** |   |   |   |
| os.memfd_create() | **+** |   |   |   |
| pathlib.Path.link_to() | **+** |   |   |   |
| platform.popen() removed; use os.popen() |   |   | **X** | **!** |
| pprint.pp() | **+** |   |   |   |
| pyvenv script removed; use **python -m venv** |   |   | **X** | **!** |
| re regular expression patterns support \N{*name*} escapes | **+** |   |   |   |
| shlex.join() (inverse of shlex.split()) | **+** |   |   |   |
| shutil.copytree() accepts named dirs_exist_ok argument | **+** |   |   |   |
| __slots__ accepts a dict of {*name*: *docstring*} | **+** |   |   |   |
| socket.create_server(), socket.has_dualstack_ipv6() | **+** |   |   |   |
| socket.if_nameindex(), socket.if_nametoindex(), and socket.if_indextoname() are all supported on Windows | **+** |   |   |   |
| sqlite3 Cache and Statement objects no longer user-visible |   |   | **X** | **!** |
| ssl.post_handshake_auth(), ssl.verify_client_post_handshake() | **+** |   |   |   |

| statistics.fmean(), statistics.geometric_mean(),

statistics.multimode(),

statistics.NormalDist,

statistics.quantiles() | **+** |   |   |   |

| sys.get_coroutine_wrapper() and sys.set_coroutine_wrapper() removed |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |
| sys.unraisablehook() | **+** |   |   |   |
| tarfile.filemode() 已移除 |   |   | **X** | **!** |

| threading.excepthook()，threading.get_native_id()，

threading.Thread.native_id | **+** |   |   |   |

| time.clock() 已移除；请使用 time.perf_counter() |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |

| tkinter.Canvas.moveto()，tkinter.PhotoImage.transparency_get()，

tkinter.PhotoImage.transparency_set()，

tkinter.Spinbox.selection_from()，

tkinter.Spinbox.selection_present(),

tkinter.Spinbox.selection_range(),

tkinter.Spinbox.selection_to() | **+** |   |   |   |

| typing.Final，typing.get_args()，typing.get_origin()，typing.Literal，

typing.Protocol，typing.SupportsIndex，typing.TypedDict | **+** |   |   |   |

| typing.NamedTuple._field_types 已弃用 |   | **—** |   | ***** |
| --- | --- | --- | --- | --- |
| unicodedata.is_normalized() | **+** |   |   |   |
| unittest 支持协程作为测试用例 | **+** |   |   |   |
| unittest.addClassCleanup()，unittest.addModuleCleanup()，unittest.AsyncMock | **+** |   |   |   |

| xml.etree.Element.getchildren()，xml.etree.Element.getiterator()，xml.etree.ElementTree.getchildren()，以及

xml.etree.ElementTree.getiterator() 已弃用 |   | **—** |   | ***** |

| XMLParser.doctype() 已移除 |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |
| xmlrpc.client.ServerProxy 接受命名的 headers 参数 | **+** |   |   |   |
| **yield** 和 **return** 解包不再需要括号 | **+** |   |   |   |
| **yield** 和 **yield** **from** 不再允许在推导式或生成器表达式中使用 |   |   | **X** | **!** |

# Python 3.9

下表总结了 Python 版本 3.9 的更改。更多详情，请参阅[在线文档](https://oreil.ly/KIMuX)中的“Python 3.9 新特性”部分。

| Python 3.9 | 添加 | 弃用 | 移除 | 破坏性变更 |
| --- | --- | --- | --- | --- |
| 类型注解现在可以在泛型中使用内置类型（例如，list[int] 而非 List[int]） | **+** |   |   |   |

| array.array.tostring() 和 array.array.fromstring() 已移除；

使用 tobytes() 和 frombytes() |   |   | **X** | **!** |

| ast.unparse() | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| asyncio.loop.create_datagram_endpoint() 参数 reuse_address 禁用 |   |   |   | **!** |

| asyncio.PidfdChild Watcher，asyncio.shutdown_default_executor()，

asyncio.to_thread() | **+** |   |   |   |

| asyncio.Task.all_tasks 已移除；请使用 asyncio.all_tasks() |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |
| asyncio.Task.current_task 已移除；请使用 asyncio.current_task() |   |   | **X** | **!** |
| base64.encodestring() 和 base64.decodestring() 已移除；请使用 base64.encodebytes() 和 base64.decodebytes() |   |   | **X** | **!** |
| concurrent.futures.Executor.shutdown() 接受命名的 cancel_futures 参数 | **+** |   |   |   |

| curses.get_escdelay()，curses.get_tabsize()，

curses.set_escdelay()，

curses.set_tabsize() | **+** |   |   |   |

| dict 支持联合运算符 &#124; 和 &#124;= | **+** |   |   |   |
| --- | --- | --- | --- | --- |

| fcntl.F_OFD_GETLK，fcntl.F_OFD_SETLK，

fcntl.F_OFD_SETKLW | **+** |   |   |   |

| fractions.gcd() 已移除；请使用 math.gcd() |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |
| functools.cache() (lightweight/faster version of lru_cache) | **+** |   |   |   |
| gc.is_finalized() | **+** |   |   |   |
| graphlib module with TopologicalSorter class | **+** |   |   |   |
| html.parser.HTMLParser.unescape() removed |   |   | **X** | **!** |
| imaplib.IMAP4.unselect() | **+** |   |   |   |
| importlib.resources.files() | **+** |   |   |   |
| inspect.BoundArguments.arguments returns dict instead of OrderedDict | **+** |   |   |   |
| ipaddress module does not accept leading zeros in IPv4 address strings |   |   |   | **!** |
| logging.getLogger('root') returns the root logger | **+** |   |   | **!** |
| math.gcd() accepts multiple arguments | **+** |   |   |   |
| math.lcm(), math.nextafter(), math.ulp() | **+** |   |   |   |
| multiprocessing.SimpleQueue.close() | **+** |   |   |   |
| nntplib.NNTP.xpath() and nntplib.xgtitle() removed |   |   | **X** | **!** |
| os.pidfd_open() | **+** |   |   |   |
| os.unsetenv() available on Windows | **+** |   |   |   |
| os.waitstatus_to_exitcode() | **+** |   |   |   |
| parser module deprecated |   | **—** |   | ***** |
| pathlib.Path.readlink() | **+** |   |   |   |
| plistlib API removed |   |   | **X** | **!** |
| pprint supports types.SimpleNamespace | **+** |   |   |   |
| random.choices() with weights argument raises ValueError if weights are all 0 |   |   |   | **!** |
| random.Random.randbytes() | **+** |   |   |   |
| socket.CAN_RAW_JOIN_FILTERS, socket.send_fds(), socket.recv_fds() | **+** |   |   |   |
| str.removeprefix(), str.removesuffix() | **+** |   |   |   |
| symbol module deprecated |   | **—** |   | ***** |
| sys.callstats(), sys.getcheckinterval(), sys.getcounts(), and sys.setcheckinterval() removed |   |   | **X** | **!** |

| sys.getcheckinterval() and sys.setcheckinterval() removed;

use sys.getswitchinterval() and

sys.setswitchinterval() |   |   | **X** | **!** |

| sys.platlibdir attribute | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| threading.Thread.isAlive() removed; use threading.Thread.is_alive() |   |   | **X** | **!** |
| tracemalloc.reset_peak() | **+** |   |   |   |
| typing.Annotated type | **+** |   |   |   |
| typing.Literal deduplicates values; equality matching is order independent (3.9.1) |   |   |   | **!** |
| typing.NamedTuple._field_types removed; use __annotations__ |   |   | **X** | **!** |
| urllib.parse.parse_qs() and urllib.parse.parse_qsl() accept ; or & query parameter separator, but not both (3.9.2) |   |   |   | **!** |
| urllib.parse.urlparse() changed handling of numeric paths; a string like 'path:80' is no longer parsed as a path but as a scheme ('path') and a path ('80') |   |   |   | **!** |

| **with** (**await** asyncio.Condition) and **with** (**yield from** asyncio.Condition) removed;

use **async with** condition |   |   | **X** | **!** |

| **with** (**await** asyncio.lock) and **with** (**yield from** asyncio.lock) removed;

use **async with** lock |   |   | **X** | **!** |

| **with** (**await** asyncio.Semaphore) and **with** (**yield from** asyncio.Semaphore) removed; use **async with** semaphore |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |

| xml.etree.Element.getchildren(), xml.etree.Element.getiterator(),

xml.etree.ElementTree.getchildren(), and

xml.etree.ElementTree.getiterator() removed |   |   | **X** | **!** |

| zoneinfo module for IANA time zone support | **+** |   |   |   |
| --- | --- | --- | --- | --- |

# Python 3.10

以下表格总结了 Python 3.10 版本的变更。更多详情请参见 [在线文档](https://oreil.ly/TCpF4) 中的“Python 3.10 新特性”。

| Python 3.10 | Added | Deprecated | Removed | Breaking change |
| --- | --- | --- | --- | --- |
| Building requires OpenSSL 1.1.1 or newer | **+** |   |   |   |
| Debugging improved with precise line numbers | **+** |   |   |   |
| Structural pattern matching using **match**, **case**, and **_** soft keywords^(a) | **+** |   |   |   |
| aiter() and anext() built-ins | **+** |   |   | **!** |
| array.array.index() accepts optional arguments start and stop | **+** |   |   |   |
| ast.literal_eval(*s*) strips leading spaces and tabs from input string *s* | **+** |   |   |   |
| asynchat module deprecated |   | **—** |   | ***** |
| asyncio functions remove loop parameter |   |   | **X** | **!** |
| asyncio.connect_accepted_socket() | **+** |   |   |   |
| asyncore module deprecated |   | **—** |   | ***** |
| base64.b32hexdecode, base64.b32hexencode | **+** |   |   |   |
| bdb.clearBreakpoints() | **+** |   |   |   |
| bisect.bisect, bisect.bisect_left, bisect.bisect_right, bisect.insort, bisect.insort_left, and bisect.insert_right all accept optional key argument | **+** |   |   |   |
| cgi.log deprecated |   | **—** |   | ***** |
| codecs.unregister() | **+** |   |   |   |
| collections module compatibility definitions of ABCs removed; use collections.abc |   |   | **X** | **!** |
| collections.Counter.total() | **+** |   |   |   |
| contextlib.aclosing() decorator, contextlib.AsyncContextDecorator | **+** |   |   |   |
| curses.has_extended_color_support() | **+** |   |   |   |
| dataclasses.dataclass() decorator accepts optional slots argument | **+** |   |   |   |
| dataclasses.KW_ONLY | **+** |   |   |   |
| distutils deprecated, to be removed in Python 3.12 |   | **—** |   | ***** |
| enum.StrEnum | **+** |   |   |   |
| fileinput.input() and fileinput.FileInput accept optional encoding and errors arguments | **+** |   |   |   |
| formatter module removed |   |   | **X** | **!** |
| glob.glob() and glob.iglob() accept optional root_dir and dir_fd arguments to specify root search directory | **+** |   |   |   |
| importlib.metadata.package_distributions() | **+** |   |   |   |
| inspect.get_annotations() | **+** |   |   |   |
| int.bit_count() | **+** |   |   |   |
| isinstance(obj, (atype, btype)) can be written isinstance(obj, atype&#124;btype) | **+** |   |   |   |
| issubclass(cls, (atype, btype)) 可以写作 issubclass(cls, atype&#124;btype) | **+** |   |   |   |
| itertools.pairwise() | **+** |   |   |   |
| os.eventfd(), os.splice() | **+** |   |   |   |
| os.path.realpath() 接受可选的严格参数 | **+** |   |   |   |
| os.EVTONLY, os.O_FSYNC, os.O_SYMLINK 和 os.O_NOFOLLOW_ANY 在 macOS 上新增 | **+** |   |   |   |
| parser 模块已移除 |   |   | **X** | **!** |

| pathlib.Path.chmod() 和 pathlib.Path.stat() 接受可选

follow_symlinks 关键字参数 | **+** |   |   |   |

| pathlib.Path.hardlink_to() | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| pathlib.Path.link_to() 已弃用；请使用 hardlink_to() |   | **—** |   | ***** |
| platform.freedesktop_os_release() | **+** |   |   |   |
| pprint.pprint() 接受可选的 underscore_numbers 关键字参数 | **+** |   |   |   |
| smtpd 模块已弃用 |   | **—** |   | ***** |
| ssl.get_server_certificate 接受可选的超时参数 | **+** |   |   |   |

| statistics.correlation(), statistics.covariance(),

statistics.linear_regression() | **+** |   |   |   |

| SyntaxError.end_line_no 和 SyntaxError.end_offset 属性 | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| sys.flags.warn_default_encoding 发出 EncodingWarning | **+** |   |   | ***** |
| sys.orig_argv 和 sys.stdlib_module_names 属性 | **+** |   |   |   |
| threading.__excepthook__ | **+** |   |   |   |
| threading.getprofile(), threading.gettrace() | **+** |   |   |   |
| threading.Thread 将生成的线程名称附加了'(<target.__name__>)' | **+** |   |   |   |

| traceback.format_exception(), traceback.format_exception_only(),

和 traceback.print_exception() 签名变更 |   |   |   | **!** |

| types.EllipsisType, types.NoneType, types.NotImplementedType | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| typing 模块包括用于指定 Callable 类型的参数规范变量 | **+** |   |   |   |
| typing.io 模块已弃用；请使用 typing |   | **—** |   | ***** |
| typing.is_typeddict() | **+** |   |   |   |
| typing.Literal 去重值；相等匹配是无序的 |   |   |   | **!** |
| typing.Optional[X] 可以写成 X &#124; None | **+** |   |   |   |
| typing.re 模块已弃用；请使用 typing |   | **—** |   | ***** |
| typing.TypeAlias 用于定义显式类型别名 | **+** |   |   |   |
| typing.TypeGuard | **+** |   |   |   |
| typing.Union[X, Y] 可以使用 &#124; 运算符表示为 X &#124; Y | **+** |   |   |   |
| unittest.assertNoLogs() | **+** |   |   |   |
| urllib.parse.parse_qs() 和 urllib.parse.parse_qsl() 接受 ; 或 & 查询参数分隔符，但不能同时使用 |   |   |   | **!** |
| **with** 语句接受括号内的上下文管理器: **with**(ctxmgr, ctxmgr, ...) | **+** |   |   |   |
| xml.sax.handler.LexicalHandler | **+** |   |   |   |
| zip 内建函数接受可选的严格命名参数以进行长度检查 | **+** |   |   |   |

| zipimport.find_spec(), zipimport.zipimporter.create_module(),

zipimport.zipimporter.exec_module(),

zipimport.zipimporter.invalidate_caches() | **+** |   |   |   |

| ^(a) 由于这些被定义为 *soft* 关键字，因此它们不会破坏使用同样名称的现有代码。 |
| --- |

# Python 3.11

下表总结了 Python 版本 3.11 的更改。更多详情，请参见[在线文档](https://oreil.ly/4Df8q)中的“Python 3.11 新功能”。

| Python 3.11 | 已添加 | 已弃用 | 已移除 | 破坏性变更 |
| --- | --- | --- | --- | --- |

| **在 Python 3.11.0 中发布的安全补丁并回溯到版本 3.7–3.10：** int 转换为 str 和 str 转换为

int 在除了 2、4、8、16 或 32 进制外的其他基数中引发

当生成的字符串 > 4,300 位时引发 ValueError（涉及 [CVE-2020-10735](https://oreil.ly/lS-gO)） |   |   |   | **!** |

| 性能改进 | **+** |   |   |   |
| --- | --- | --- | --- | --- |
| 改进的错误消息 | **+** |   |   |   |
| 新语法：**for** *x* **in** **values* | **+** |   |   |   |
| aifc 模块已弃用 |   | **—** |   | ***** |
| asynchat 和 asyncore 模块已弃用 |   | **—** |   | ***** |

| asyncio.Barrier，asyncio.start_tls()，

asyncio.TaskGroup | **+** |   |   |   |

| asyncio.coroutine 装饰器已移除 |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |
| asyncio.loop.create_datagram_endpoint() 参数 reuse_address 已移除 |   |   | **X** | **!** |
| asyncio.TimeoutError 已弃用；使用 TimeoutError |   | **—** |   | ***** |
| 音频操作模块已弃用 |   | **—** |   | ***** |
| BaseException.add_note()，BaseException.__notes__ 属性 | **+** |   |   |   |

| binascii.a2b_hqx()，binascii.b2a_hqx()，

binascii.rlecode_hqx()，和

binascii.rledecode_hqx() 已移除 |   |   | **X** | **!** |

| binhex 模块已移除 |   |   | **X** | **!** |
| --- | --- | --- | --- | --- |
| cgi 和 cgitb 模块已弃用 |   | **—** |   | ***** |
| chunk 模块已弃用 |   | **—** |   | ***** |
| concurrent.futures.ProcessPoolExecutor() max_tasks_per_child 参数 | **+** |   |   |   |
| concurrent.futures.TimeoutError 已弃用；使用内置的 TimeoutError |   | **—** |   | ***** |
| contextlib.chdir 上下文管理器（更改当前工作目录然后恢复它） | **+** |   |   |   |
| crypt 模块已弃用 |   | **—** |   | ***** |
| dataclasses 对于可变默认值的检查不允许任何非可哈希值（以前允许任何非 dict、list 或 set 的值） |   |   |   | **!** |
| datetime.UTC 作为 datetime.timezone.utc 的方便别名 | **+** |   |   |   |
| enum.Enum 的 str() 输出仅提供名称 | **+** |   |   |   |
| enum.EnumCheck，enum.FlagBoundary，enum.global_enum() 装饰器，enum.member() 装饰器，enum.nonmember() 装饰器，enum.property，enum.ReprEnum，enum.StrEnum 和 enum.verify() | **+** |   |   |   |
| ExceptionGroups 和 except* | **+** |   |   |   |
| fractions.Fraction 从字符串初始化 | **+** |   |   |   |
| gettext.l*gettext() 方法已移除 |   |   | **X** | **!** |
| glob.glob() 和 glob.iglob() 接受可选的 include_hidden 参数 | **+** |   |   |   |
| hashlib.file_digest() | **+** |   |   |   |
| imghdr 模块已弃用 |   | **—** |   | ***** |
| inspect.formatargspec() 和 inspect.getargspec() 已移除；请使用 inspect.signature() |   |   | **X** | **!** |
| inspect.getmembers_static()、inspect.ismethodwrapper() | **+** |   |   |   |
| locale.getdefaultlocale() 和 locale.resetlocale() 已弃用 |   | **—** |   | ***** |
| locale.getencoding() | **+** |   |   |   |
| logging.getLevelNamesMapping() | **+** |   |   |   |
| mailcap 模块已弃用 |   | **—** |   | ***** |
| math.cbrt()（立方根）、math.exp2()（计算 2ⁿ） | **+** |   |   |   |
| msilib 模块已弃用 |   | **—** |   | ***** |
| nis 模块已弃用 |   | **—** |   | ***** |
| nntplib 模块已弃用 |   | **—** |   | ***** |
| operator.call | **+** |   |   |   |
| ossaudiodev 模块已弃用 |   | **—** |   | ***** |
| pipes 模块已弃用 |   | **—** |   | ***** |
| re 模式语法支持 *+, ++, ?+ 和 {m,n}+ 占有量词，以及 (?>...) 原子分组 | **+** |   |   |   |
| re.template() 已弃用 |   | **—** |   | ***** |
| smtpd 模块已弃用 |   | **—** |   | ***** |
| sndhdr 模块已弃用 |   | **—** |   | ***** |
| spwd 模块已弃用 |   | **—** |   | ***** |

| sqlite3.Connection.blobopen()、sqlite3.Connection.create_window_function()、sqlite3.Connection.deserialize()、

sqlite3.Connection.getlimit()、

sqlite3.Connection.serialize()、

sqlite3.Connection.setlimit() | **+** |   |   |   |

| sre_compile、sre_constants 和 sre_parse 已弃用 |   | **—** |   | ***** |
| --- | --- | --- | --- | --- |
| statistics.fmean() 可选的 weights 参数 | **+** |   |   |   |
| sunau 模块已弃用 |   | **—** |   | ***** |
| sys.exception()（相当于 sys.exc_info()[1]） | **+** |   |   |   |
| telnetlib 模块已弃用 |   | **—** |   | ***** |
| time.nanosleep()（仅适用于类 Unix 系统） | **+** |   |   |   |
| tomllib TOML 解析模块 | **+** |   |   |   |

| typing.assert_never()、typing.assert_type()、

typing.LiteralString、typing.Never、

typing.reveal_type()、typing.Self | **+** |   |   |   |

| typing.Text 已弃用；请使用 str |   | **—** |   | ***** |
| --- | --- | --- | --- | --- |
| typing.TypedDict 的条目可以标记为 Required 或 NotRequired | **+** |   |   |   |
| typing.TypedDict(a=int, b=str) 形式已弃用 |   | **—** |   | ***** |
| unicodedata 更新至 Unicode 14.0.0 | **+** |   |   |   |

| unittest.enterModuleContext()、unittest.IsolatedAsyncioTestCase.enterAsyncContext()、

unittest.TestCase.enterClassContext()、

unittest.TestCase.enterContext() | **+** |   |   |   |

| unittest.findTestCases()、unittest.getTestCaseName()、

和 unittest.makeSuite() 已弃用；

请使用 unittest.TestLoader 的方法 |

| uu 模块已弃用 |   | **—** |   | ***** |
| --- | --- | --- | --- | --- |
| **with** 语句现在对不支持上下文管理器协议的对象抛出 TypeError 异常 |   |   |   | **!** |
| xdrlib 模块已弃用 |   | **—** |   | ***** |
| 添加了 z 字符串格式说明符，用于接近零值的负数标志 | **+** |   |   |   |
| 添加了 zipfile.ZipFile.mkdir() | **+** |   |   |   |
| *在此处添加您自己的笔记:* |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
|   |   |   |   |   |
