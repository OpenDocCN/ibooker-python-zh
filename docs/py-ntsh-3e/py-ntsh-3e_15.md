# 第十五章：并发：线程和进程

*进程*是操作系统中彼此保护的运行程序的实例。想要通信的进程必须明确通过*进程间通信*（IPC）机制，以及/或通过文件（在第十一章讨论）、数据库（在第十二章讨论）、或网络接口（在第十八章讨论）来安排通信。进程之间使用文件和数据库等数据存储机制通信的一般方式是一个进程写入数据，另一个进程稍后读取该数据。本章介绍了处理进程的编程，包括 Python 标准库模块 subprocess 和 multiprocessing；模块 os 中与进程相关的部分，包括通过*管道*进行简单 IPC；一种称为*内存映射文件*的跨平台 IPC 机制，在模块 mmap 中可用；以及 3.8+及 multiprocessing.shared_memory 模块。

*线程*（最初称为“轻量级进程”）是与单个进程内的其他线程共享全局状态（内存）的控制流；所有线程看起来同时执行，尽管它们实际上可能在一个或多个处理器/核心上“轮流”执行。线程远非易于掌握，而多线程程序通常难以测试和调试；然而，如“线程、多进程还是异步编程？”所述，适当使用多线程时，与单线程编程相比性能可能会提高。本章介绍了 Python 提供的处理线程的各种功能，包括线程、队列和 concurrent.futures 模块。

在单个进程内共享控制的另一种机制是所谓的*异步*（或*async*）编程。当你阅读 Python 代码时，关键字**async**和**await**的存在表明它是异步的。这样的代码依赖于*事件循环*，它大致相当于进程内部使用的线程切换器。当事件循环是调度器时，每次执行异步函数变成一个*任务*，与多线程程序中的*线程*大致对应。

进程调度和线程切换都是*抢占式*的，这意味着调度器或切换器控制 CPU，并确定何时运行任何特定的代码。然而，异步编程是*协作式*的：每个任务一旦开始执行，可以在选择放弃控制之前运行多长时间（通常是因为正在等待完成某些其他异步任务，通常是面向 I/O 的任务）。

尽管异步编程提供了优化某些问题类别的灵活性，但这是许多程序员不熟悉的编程范例。由于其协作性质，不慎的异步编程可能导致*死锁*，而无限循环则可能使其他任务被剥夺处理器时间：弄清楚如何避免死锁为普通程序员增加了显著的认知负担。我们在本卷中不进一步讨论异步编程，包括模块[asyncio](https://oreil.ly/zRZKX)，认为这是一个足够复杂的主题，值得单独撰写一本书进行探讨。¹

网络机制非常适合 IPC，并且在网络的不同节点上运行的进程之间，以及在同一节点上运行的进程之间同样有效。multiprocessing 模块提供了一些适合在网络上进行 IPC 的机制；第十八章涵盖了提供 IPC 基础的低级网络机制。其他更高级的*分布式计算*机制（如[CORBA](https://www.ibm.com/docs/en/integration-bus/9.0.0?topic=corba-common-object-request-broker-architecture)、[DCOM/COM+](https://whatis.techtarget.com/definition/DCOM-Distributed-Component-Object-Model#:~:text=DCOM%20(Distributed%20Component%20Object%20Model)%20is%20a%20set%20of%20Microsoft,other%20computers%20in%20a%20network.)、[EJB](https://en.wikipedia.org/wiki/Jakarta_Enterprise_Beans#:~:text=EJB%20is%20a%20server%2Dside,processing%2C%20and%20other%20web%20services)、[SOAP](https://en.wikipedia.org/wiki/SOAP)、[XML-RPC](http://xmlrpc.com)、[.NET](https://dotnet.microsoft.com/en-us/learn/dotnet/what-is-dotnet)、[gRPC](https://grpc.io)等）可以使 IPC 变得更加容易，无论是本地还是远程；然而，本书不涵盖分布式计算。

当多处理器计算机出现时，操作系统必须处理更复杂的调度问题，而希望获得最大性能的程序员必须编写他们的应用程序，以便代码可以真正并行执行，即在不同的处理器或核心上（从编程角度看，核心只是在同一块硅片上实现的处理器）。这需要知识和纪律。CPython 实现通过实现*全局解释器锁*（GIL）简化了这些问题。在没有任何 Python 程序员的操作下，在 CPython 中只有持有 GIL 的线程被允许访问处理器，有效地阻止了 CPython 进程充分利用多处理器硬件。诸如[NumPy](https://numpy.org)这样的库通常需要进行长时间的计算，使用的是不使用解释器设施的编译代码，这些库在这些计算期间释放 GIL。这允许有效地使用多个处理器，但如果您的所有代码都是纯 Python，则不能使用这种技术。

# Python 中的线程

Python 支持在支持线程的平台上进行多线程操作，如 Windows、Linux 和几乎所有 Unix 变体（包括 macOS）。当动作在开始和结束之间保证没有线程切换时，该动作被称为*原子*。在实践中，在 CPython 中，看起来是原子的操作（例如，简单赋值和访问）大多数情况下确实是原子的，但只适用于内置类型（但增强和多重赋值不是原子的）。尽管如此，依赖于这种“原子性”通常*不*是一个好主意。您可能正在处理用户编写的类的实例，而不是内置类型的实例，在这种情况下，可能会有隐式调用 Python 代码的情况，这些调用会使原子性假设失效。此外，依赖于实现相关的原子性可能会将您的代码锁定到特定的实现中，从而阻碍未来的更改。建议您在本章的其余部分使用同步设施，而不是依赖于原子性假设。

多线程系统中的关键设计问题是如何最好地协调多个线程。线程模块在下一节中介绍，提供了几种同步对象。队列模块（在“队列模块”中讨论）对于线程同步也非常有用：它提供了同步的、线程安全的队列类型，方便线程间的通信和协调。concurrent 包（在“concurrent.futures 模块”中讨论）提供了一个统一的通信和协调接口，可以由线程池或进程池实现。

# 线程模块

线程模块提供了多线程功能。线程的方法是将锁和条件建模为单独的对象（例如，在 Java 中，此类功能是每个对象的一部分），并且线程不能直接从外部控制（因此没有优先级、组、销毁或停止）。线程模块提供的所有对象的方法都是原子的。

线程模块提供了以下专注于线程的类，我们将在本节中探讨所有这些类：Thread、Condition、Lock、RLock、Event、Semaphore、BoundedSemaphore、Timer 和 Barrier。

threading 还提供了许多有用的函数，包括在表 15-1 中列出的函数。

表 15-1\. 线程模块的函数

| active_count | active_count() 返回一个整数，表示当前存活的线程对象数量（不包括已终止或尚未启动的线程）。 |
| --- | --- |
| c⁠u⁠r⁠r⁠e⁠n⁠t⁠_​t⁠h⁠r⁠e⁠a⁠d | current_thread() 返回调用线程的 Thread 对象。如果调用线程不是由 threading 创建的，则 current_thread 创建并返回一个具有有限功能的半虚拟 Thread 对象。 |
| enumerate | enumerate() 返回当前存活的所有 Thread 对象的列表（不包括已终止或尚未启动的线程）。 |
| excepthook | excepthook(args) 3.8+ 重写此函数以确定如何处理线程内的异常；有关详细信息，请参阅[在线文档](https://oreil.ly/ylw7S)。args 参数具有属性，允许您访问异常和线程详细信息。3.10+ threading.__excepthook__ 保存了模块的原始线程钩子值。 |
| get_ident | get_ident() 返回一个非零整数作为所有当前线程中唯一的标识符。用于管理和跟踪线程数据。线程标识符可能会在线程退出并创建新线程时重复使用。 |
| get_native_id | get_native_id() 3.8+ 返回由操作系统内核分配的当前线程的本机整数 ID。适用于大多数常见操作系统。 |
| stack_size | stack_size([*size*]) 返回用于新线程的当前堆栈大小（以字节为单位），并（如果提供 *size*）为新线程设定值。*size* 的可接受值受到平台特定约束的限制，例如至少为 32768（或某些平台上更高的最小值），并且（在某些平台上）必须是 4096 的倍数。传递值为 0 总是可接受的，表示“使用系统的默认值”。当您传递一个在当前平台上不可接受的 *size* 值时，stack_size 会引发 ValueError 异常。 |

## Thread 对象

线程实例 *t* 模拟一个线程。在创建 *t* 时，您可以将一个函数作为 *t* 的主函数传递给 *target* 参数，或者您可以子类化 Thread 并重写其 run 方法（您还可以重写 __init__，但不应重写其他方法）。在创建时，*t* 还未准备好运行；要使 *t* 准备就绪（活动），请调用 *t*.start。一旦 *t* 处于活动状态，它将在其主函数正常结束或通过传播异常时终止。线程 *t* 可以是 *daemon*，这意味着即使 *t* 仍然活动，Python 也可以终止，而普通（非守护）线程则会一直保持 Python 运行，直到线程终止。Thread 类提供了在 Table 15-2 中详细描述的构造函数、属性和方法。

Table 15-2\. Thread 类的构造函数、方法和属性

| Thread | **class** Thread(name=**None**, target=**None**, args=(), kwargs={}, *, daemon=**None**)

*始终*使用命名参数调用 *Thread*：虽然参数的数量和顺序不受规范保证，但参数名是固定的。在构造 Thread 时有两个选项：

+   使用目标函数（*t*.run 在线程启动时调用 *target*(**args*, ***kwargs*））来实例化 Thread 类本身。

+   扩展 Thread 类并重写其 run 方法。

在任何情况下，仅当调用 *t*.start 时执行将开始。name 成为 *t* 的名称。如果 name 是 **None**，Thread 为 *t* 生成一个唯一名称。如果 Thread 的子类 *T* 重写 __init__，*T*.__init__ *必须* 在调用任何其他 Thread 方法之前调用 Thread.__init__（通常通过 super 内置函数）。daemon 可以分配布尔值，或者如果为 **None**，则将从创建线程的 daemon 属性获取此值。 |

| daemon | daemon 是一个可写的布尔属性，指示 *t* 是否为守护线程（即使 *t* 仍然活动，进程也可以终止；这样的终止也会结束 *t*）。只能在调用 *t*.start 之前对 *t*.daemon 赋值；将 true 值赋给 *t* 将其设置为守护线程。守护线程创建的线程的 *t*.daemon 默认为 **True**。 |
| --- | --- |
| is_alive | *t*.is_alive() 当 *t* 处于活动状态时（即 *t*.start 已执行且 *t*.run 尚未终止），is_alive 返回 **True**；否则返回 **False**。 |
| join | *t*.join(timeout=**None**) join 暂停调用线程（不能是 *t*）直到 *t* 终止（当 *t* 已经终止时，调用线程不会暂停）。timeout 在 “超时参数” 中讨论。只能在 *t*.start 之后调用 *t*.join。可以多次调用 join。 |
| name | *t*.name name 是返回 *t* 名称的属性；分配 name 重新绑定 *t* 的名称（名称仅用于调试帮助；名称在线程中无需唯一）。如果省略，则线程将接收生成的名称 Thread-*n*，其中 *n* 是递增的整数（3.10+ 并且如果指定了 target，则将附加 (target.__name__)）。 |
| run | *t*.run() 是由 *t*.start 调用的方法，执行 *t* 的主函数。Thread 的子类可以重写 run 方法。如果未重写，则 run 方法调用 *t* 创建时传递的 *target* 可调用对象。不要直接调用 *t*.run；调用 *t*.run 的工作由 *t*.start 完成！ |
| start | *t*.start() 使 *t* 变为活动状态，并安排 *t*.run 在单独的线程中执行。对于任何给定的 Thread 对象 *t*，只能调用一次 *t*.start；再次调用会引发异常。 |

## 线程同步对象

threading 模块提供了几种同步原语（允许线程通信和协调的类型）。每种原语类型都有专门的用途，将在以下部分讨论。

# 也许你不需要线程同步原语

只要避免有（非队列）全局变量，这些变量会发生变化，多个线程可以访问，队列（在“队列模块”中介绍）通常可以提供所需的所有协调工作，同时并发（在“concurrent.futures 模块”中介绍）也可以。“线程化程序架构”展示了如何使用队列对象为您的多线程程序提供简单而有效的架构，通常无需显式使用同步原语。

### 超时参数

同步原语 Condition 和 Event 提供接受可选超时参数的 wait 方法。线程对象的 join 方法也接受可选的超时参数（参见表 15-2）。使用默认的超时值 **None** 会导致正常的阻塞行为（调用线程挂起并等待，直到满足所需条件）。当超时值不是 **None** 时，超时参数是一个浮点值，表示时间间隔，单位为秒（超时可以有小数部分，因此可以指示任何时间间隔，甚至非常短的间隔）。当超过超时秒数时，调用线程再次准备好，即使所需条件尚未满足；在这种情况下，等待方法返回 **False**（否则，方法返回 **True**）。超时让您设计能够处理少数线程偶发异常的系统，从而使系统更加健壮。但是，使用超时可能会降低程序的运行速度：在这种情况下，请务必准确测量您代码的速度。

### 锁和重入锁对象

锁和重入锁对象提供相同的三种方法，详见表 15-3。

表 15-3\. 锁实例 L 的方法

| acquire | *L*.acquire(blocking=**True**, timeout=-1) 当 *L* 未锁定时，或者如果 *L* 是由同一个线程获取的重入锁，该线程立即锁定它（如果 *L* 是重入锁，则会增加内部计数器，如后面所述），并返回 **True**。

当 *L* 已经被锁定且 blocking 为 **False** 时，acquire 立即返回 **False**。当 blocking 为 **True** 时，调用线程将被挂起，直到以下情况发生之一：

+   另一个线程释放了锁，则该线程锁定它并返回 **True**。

+   在锁被获取之前操作超时，此时 acquire 返回 **False**。默认的 -1 值永不超时。

|

| locked | *L*.locked() 当 *L* 被锁定时返回 **True**；否则返回 **False**。 |
| --- | --- |
| release | `L.release()` 解锁 `L`，必须已锁定（对于 `RLock`，这意味着减少锁计数，锁计数不得低于零——只有当锁计数为零时才能由新线程获取）。当 `L` 被锁定时，任何线程都可以调用 `L.release`，不仅仅是锁定 `L` 的线程。当多个线程被阻塞在 `L` 上时（即调用了 `L.acquire`，发现 `L` 被锁定，并等待 `L` 解锁），`release` 将唤醒其中任意一个等待的线程。调用 `release` 的线程不会挂起：它仍然准备好并继续执行。 |

下面的控制台会话示例说明了当锁被用作上下文管理器时（以及 Python 在锁的使用过程中维护的其他数据，例如所有者线程 ID 和锁的获取方法被调用的次数）自动获取/释放锁的情况：

```py
>>> lock = threading.RLock()
>>> print(lock)
```

```py
<unlocked _thread.RLock object owner=0 count=0 at 0x102878e00>
```

```py
>>> `with` lock:
...     print(lock)
...
```

```py
<locked _thread.RLock object owner=4335175040 count=1 at 0x102878e00>
```

```py
>>> print(lock)
```

```py
<unlocked _thread.RLock object owner=0 count=0 at 0x102878e00>
```

`RLock` 对象 `r` 的语义通常更为方便（除非在需要线程能够释放不同线程已获取的锁的特殊体系结构中）。 `RLock` 是一个可重入锁，意味着当 `r` 被锁定时，它会跟踪*拥有*它的线程（即锁定它的线程，对于 `RLock` 来说也是唯一能够释放它的线程——当任何其他线程尝试释放 `RLock` 时，会引发 `RuntimeError` 异常）。拥有它的线程可以再次调用 `r.acquire` 而不会阻塞；然后 `r` 只是增加一个内部计数。在涉及 `Lock` 对象的类似情况下，线程会阻塞直到某个其他线程释放该锁。例如，考虑以下代码片段：

```py
lock = threading.RLock()
global_state = []
`def` recursive_function(some, args):
    `with` lock:  *`# acquires lock, guarantees release at end`*
        *`# ...modify global_state...`*
        `if` more_changes_needed(global_state):
            recursive_function(other, args)
```

如果锁是 `threading.Lock` 的一个实例，当 `recursive_function` 递归调用自身时，会阻塞其调用线程：`with` 语句会发现锁已经被获取（尽管是同一个线程获取的），然后会阻塞并等待……等待。而使用 `threading.RLock` 则不会出现这样的问题：在这种情况下，由于锁已经被同一线程获取，再次获取时只是增加其内部计数然后继续。

一个 `RLock` 对象 `r` 只有在释放次数与获取次数相同的情况下才会解锁。当对象的方法相互调用时，`RLock` 对象非常有用；每个方法在开始时可以获取，并在结束时释放同一个 `RLock` 实例。

# 使用 `with` 语句自动获取和释放同步对象

使用**try**/**finally**语句（在“try/finally”中介绍）是确保已获取的锁确实被释放的一种方法。使用**with**语句，通常更好，因为所有的锁、条件和信号量都是上下文管理器，所以这些类型的实例可以直接在**with**子句中使用，以获取它（隐式地带有阻塞）并确保在**with**块的末尾释放它。

### 条件对象

条件对象*c*封装了锁或者递归锁对象*L*。Condition 类公开了在表 15-4 中描述的构造函数和方法。

表 15-4\. Condition 类的构造函数和方法

| Condition | **class** Condition(lock=**None**) 创建并返回一个新的 Condition 对象*c*，并使用锁*L*设置为 lock。如果 lock 为**None**，则*L*设置为新创建的 RLock 对象。 |
| --- | --- |
| acquire, release | *c*.acquire(blocking=**True**), *c*.release() 这些方法只是调用*L*的相应方法。线程除了持有（即已获取）锁*L*之外，绝不应调用*c*的任何其他方法。 |
| notify, notify_all | *c*.notify(), *c*.notify_all() notify 唤醒在*c*上等待的任意一个线程。在调用*c*.notify 之前，调用线程必须持有*L*，并且 notify 不会释放*L*。被唤醒的线程直到再次可以获取*L*时才变为就绪。因此，通常调用线程在调用 notify 后调用 release。notify_all 类似于 notify，但唤醒*所有*等待的线程，而不仅仅是一个。 |
| wait | *c*.wait(timeout=**None**) wait 释放*L*，然后挂起调用线程，直到其他线程在*c*上调用 notify 或 notify_all。在调用*c*.wait 之前，调用线程必须持有*L*。timeout 在“超时参数”中有描述。线程通过通知或超时唤醒后，当再次获取*L*时，线程变为就绪。当 wait 返回**True**（表示正常退出，而不是超时退出）时，调用线程总是再次持有*L*。 |

通常，Condition 对象*c*调节一些在线程间共享的全局状态*s*的访问。当一个线程必须等待*s*改变时，线程循环：

```py
`with` *`c`*:
    `while` `not` is_ok_state(s):
        *`c`*.wait()
    do_some_work_using_state(s)
```

同时，每个修改*s*的线程在每次*s*变化时调用 notify（或者如果需要唤醒所有等待的线程而不仅仅是一个，则调用 notify_all）：

```py
`with` *`c`*:
    do_something_that_modifies_state(*`s`*)
    *`c`*.notify()    *`# or, c.notify_all()`*
*`# no need to call c.release(), exiting 'with' intrinsically does that`*
```

您必须始终在每次使用*c*的方法周围获取和释放*c*：通过**with**语句这样做使得使用 Condition 实例更不容易出错。

### 事件对象

事件对象允许任意数量的线程挂起和等待。当任何其他线程调用 *e*.set 时，所有等待事件对象 *e* 的线程都变为就绪状态。*e* 有一个标志记录事件是否发生；在 *e* 创建时，该标志最初为 **False**。因此，事件类似于简化版的条件变量。事件对象适用于一次性的信号传递，但对于更一般的用途而言，可能不够灵活；特别是依赖于调用 *e*.clear 很容易出错。事件类公开了 表 15-5 中的构造函数和方法。

表 15-5\. 事件类的构造函数和方法

| Event | **class** Event() 创建并返回一个新的事件对象 *e*，并将 *e* 的标志设置为 **False**。 |
| --- | --- |
| clear | *e*.clear() 将 *e* 的标志设置为 **False**。 |
| is_set | *e*.is_set() 返回 *e* 的标志值：**True** 或 **False**。 |
| set | *e*.set() 将 *e* 的标志设置为 **True**。所有等待 *e* 的线程（如果有）都将变为就绪状态。 |
| wait | *e*.wait(timeout=**None**) 如果 *e* 的标志为 **True**，则立即返回；否则，挂起调用线程，直到其他线程调用 set。timeout 参见 “超时参数”。 |

下面的代码显示了如何显式同步多个线程之间的处理过程使用事件对象：

```py
`import` datetime, random, threading, time

`def` runner():
    print('starting')
    time.sleep(random.randint(1, 3))
    print('waiting')
    event.wait()
    print(f'running at {datetime.datetime.now()}')

num_threads = 10
event = threading.Event()

threads = [threading.Thread(target=runner) `for` _ `in` range(num_threads)]
`for` t `in` threads:
    t.start()

event.set()

`for` t `in` threads:
    t.join()
```

### 信号量和有界信号量对象

*信号量*（也称为*计数信号量*）是锁的一种泛化形式。锁的状态可以看作是 **True** 或 **False**；信号量 *s* 的状态是一个在创建 *s* 时设置的介于 0 和某个 *n* 之间的数值（两个边界都包括）。信号量可以用来管理一组固定的资源，例如 4 台打印机或 20 个套接字，尽管对于这种目的，使用队列（本章后面描述）通常更为健壮。有界信号量类与此非常相似，但是如果状态超过初始值，则会引发 ValueError：在许多情况下，这种行为可以作为错误的有用指示器。 表 15-6 显示了信号量和有界信号量类的构造函数以及任何一类对象 *s* 所暴露的方法。

表 15-6\. 信号量和有界信号量类的构造函数和方法

| Semaphore, Boun⁠d⁠e⁠d​S⁠e⁠m⁠aphore | **class** Semaphore(n=1), **class** BoundedSemaphore(n=1)

信号量使用指定状态 *n* 创建并返回一个信号量对象 *s*；有界信号量类似，但如果状态高于 *n*，则 *s*.release 会引发 ValueError。 |

| acquire | *s*.acquire(blocking=**True**) 当 *s* 的状态 >0 时，acquire 将状态减 1 并返回 **True**。当 *s* 的状态为 0 且 blocking 为 **True** 时，acquire 暂停调用线程并等待，直到其他线程调用 *s*.release。当 *s* 的状态为 0 且 blocking 为 **False** 时，acquire 立即返回 **False**。 |
| --- | --- |
| release | *s*.release() 当 *s* 的状态大于 0 时，或者状态为 0 但没有线程在等待 *s* 时，release 将状态增加 1。当 *s* 的状态为 0 且有线程在等待 *s* 时，release 将保持 *s* 的状态为 0，并唤醒其中一个等待的线程。调用 release 的线程不会挂起；它保持就绪状态并继续正常执行。 |

### Timer 对象

Timer 对象在给定延迟后，在新创建的线程中调用指定的可调用对象。Timer 类公开了构造函数和 Table 15-7 中的方法。

Table 15-7\. Timer 类的构造函数和方法

| Timer | **class** Timer(*interval*, *callback*, args=**None**, kwargs=**None**) 创建一个对象 *t*，在启动后 *interval* 秒调用 *callback*（*interval* 是一个浮点秒数）。 |
| --- | --- |
| cancel | *t*.cancel() 停止定时器并取消其动作的执行，只要在调用 cancel 时 *t* 仍在等待（尚未调用其回调）。 |
| start | *t*.start() 启动 *t*。 |

Timer 继承自 Thread 并添加了属性 function、interval、args 和 kwargs。

Timer 是“一次性”的：*t* 仅调用其回调一次。要周期性地每隔 *interval* 秒调用 *callback*，这里是一个简单的方法——周期性定时器每隔 *interval* 秒运行 *callback*，只有在 *callback* 引发异常时才停止：

```py
`class` Periodic(threading.Timer):
    `def` __init__(self, interval, callback, args=`None`, kwargs=`None`):
        super().__init__(interval, self._f, args, kwargs)
        self.callback = callback

    `def` _f(self, *args, **kwargs):
        p = type(self)(self.interval, self.callback, args, kwargs)
        p.start()
        `try`:
            self.callback(*args, **kwargs)
        `except` Exception:
            p.cancel()
```

### Barrier 对象

Barrier 是一种同步原语，允许一定数量的线程等待，直到它们都达到执行中的某一点，然后它们全部恢复。具体来说，当线程调用 *b*.wait 时，它会阻塞，直到指定数量的线程在 *b* 上做出相同的调用；此时，所有在 *b* 上阻塞的线程都被允许恢复。

Barrier 类公开了构造函数、方法和 Table 15-8 中列出的属性。

Table 15-8\. Barrier 类的构造函数、方法和属性

| Barrier | **class** Barrier(*num_threads*, action=**None**, timeout=**None**) 为 *num_threads* 个线程创建一个 Barrier 对象 *b*。action 是一个没有参数的可调用对象：如果传递了此参数，则它会在所有阻塞线程中的任何一个被解除阻塞时执行。timeout 在 “Timeout parameters” 中有说明。 |
| --- | --- |
| abort | *b*.abort() 将 Barrier *b* 置于 *broken* 状态，这意味着当前等待的任何线程都会恢复，并抛出 threading.BrokenBarrierException（在任何后续调用 *b*.wait 时也会引发相同的异常）。这是一种紧急操作，通常用于当等待线程遭遇异常终止时，以避免整个程序死锁。 |
| broken | *b*.broken 当 *b* 处于 *broken* 状态时为 **True**；否则为 **False**。 |
| n_waiting | *b.*n_waiting 当前正在等待 *b* 的线程数。 |
| parties | parties 在 *b* 构造函数中作为 *num_threads* 传递的值。 |
| 重置 | *b.*reset() 将*b*重置为初始的空、未损坏状态；但是，当前正在*b*上等待的任何线程都将恢复，带有 threading.BrokenBarrierException 异常。 |
| 等待 | *b.*wait() 第一个*b*.parties-1 个调用*b*.wait 的线程会阻塞；当阻塞在*b*上的线程数为*b*.parties-1 时，再有一个线程调用*b*.wait，所有阻塞在*b*上的线程都会恢复。*b*.wait 向每个恢复的线程返回一个 int，所有返回值都是不同的且在 range(*b*.parties)范围内，顺序不确定；线程可以使用这个返回值来确定接下来应该做什么（尽管在 Barrier 的构造函数中传递 action 更简单且通常足够）。 |

下面的代码展示了 Barrier 对象如何在多个线程之间同步处理（与之前展示的 Event 对象的示例代码进行对比）：

```py
`import` datetime, random, threading, time

`def` runner():
    print('starting')
    time.sleep(random.randint(1, 3))
    print('waiting')
    `try`:
        my_number = barrier.wait()
    `except` threading.BrokenBarrierError:
        print('Barrier abort() or reset() called, thread exiting...')
        return
    print(f'running ({my_number}) at {datetime.datetime.now()}')

`def` announce_release():
    print('releasing')

num_threads = 10
barrier = threading.Barrier(num_threads, action=announce_release)

threads = [threading.Thread(target=runner) `for` _ `in` range(num_threads)]
`for` t `in` threads:
    t.start()

`for` t `in` threads:
    t.join()
```

## 线程本地存储

threading 模块提供了类 local，线程可以使用它来获取*线程本地存储*，也称为*每个线程的数据*。local 的一个实例*L*有任意命名的属性，你可以设置和获取，存储在字典*L*.__dict__ 中，也可以直接访问。*L*是完全线程安全的，这意味着多个线程同时设置和获取*L*上的属性没有问题。每个访问*L*的线程看到的属性集合是独立的：在一个线程中进行的任何更改对其他线程没有影响。例如：

```py
`import` threading

L = threading.local()
print('in main thread, setting zop to 42')
L.zop = 42

`def` targ():
  print('in subthread, setting zop to 23')
  L.zop = 23
  print('in subthread, zop is now', L.zop)

t = threading.Thread(target=targ)
t.start()
t.join()
print('in main thread, zop is now', L.zop)
*`# prints:`*
*`#`* *`in main thread, setting zop to 42`*
*`#`* *`in subthread, setting zop to 23`*
*`#`* *`in subthread, zop is now 23`*
*`#`* *`in main thread, zop is now 42`*
```

线程本地存储使得编写多线程代码更加容易，因为你可以在多个线程中使用相同的命名空间（即 threading.local 的一个实例），而不会相互干扰。

# 队列模块

队列模块提供支持多线程访问的队列类型，主要类是 Queue，还有一个简化的类 SimpleQueue，主类的两个子类（LifoQueue 和 PriorityQueue），以及两个异常类（Empty 和 Full），在表 15-9 中描述。主类及其子类实例暴露的方法详见表 15-10。

表 15-9\. 队列模块的类

| 队列 | **类** Queue(maxsize=0) Queue 是 queue 模块中的主要类，实现先进先出（FIFO）队列：每次检索的项是最早添加的项。

当 maxsize > 0 时，新的 Queue 实例*q*在*q*达到 maxsize 项时被视为已满。当*q*已满时，插入带有 block=**True**的项的线程将暂停，直到另一个线程提取一个项。当 maxsize <= 0 时，*q*永远不会被视为满，仅受可用内存限制，与大多数 Python 容器一样。 |

| SimpleQueue | **class** SimpleQueue SimpleQueue 是一个简化的 Queue：一个无界的 FIFO 队列，缺少 full、task_done 和 join 方法（请参见 Table 15-10），并且 put 方法忽略其可选参数但保证可重入性（这使其可在 __del__ 方法和 weakref 回调中使用，而 Queue.put 则不能）。 |
| --- | --- |
| LifoQueue | **class** LifoQueue(maxsize=0) LifoQueue 是 Queue 的子类；唯一的区别是 LifoQueue 实现了后进先出（LIFO）队列，意味着每次检索到的项是最近添加的项（通常称为 *stack*）。 |
| PriorityQueue | **class** PriorityQueue(maxsize=0) PriorityQueue 是 Queue 的子类；唯一的区别是 PriorityQueue 实现了一个 *priority* 队列，意味着每次检索到的项是当前队列中最小的项。由于没有指定排序的方法，通常会使用 (*priority*, *payload*) 对作为项，其中 *priority* 的值较低表示较早的检索。 |
| Empty | Empty 是当 *q* 为空时 *q*.get(block=**False**) 抛出的异常。 |
| Full | Full 是当 *q* 满时 *q*.put(*x*, block=**False**) 抛出的异常。 |

实例 *q* 为 Queue 类（或其子类之一）的一个实例，提供了 Table 15-10 中列出的方法，所有方法都是线程安全的，并且保证是原子操作。有关 SimpleQueue 实例公开的方法的详细信息，请参见 Table 15-9。 |

Table 15-10\. 类 Queue、LifoQueue 或 PriorityQueue 的实例 *q* 的方法

| empty | *q*.empty() 返回 **True** 当 *q* 为空时；否则返回 **False**。 |
| --- | --- |
| full | *q*.full() 返回 **True** 当 *q* 满时；否则返回 **False**。 |

| get, get_nowait | *q*.get(block=**True**, timeout=**None**), *q*.get_nowait() |

当 block 为 **False** 时，如果 *q* 中有可用项，则 get 移除并返回一个项；否则 get 抛出 Empty 异常。当 block 为 **True** 且 timeout 为 **None** 时，如果需要，get 移除并返回 *q* 中的一个项，挂起调用线程，直到有可用项。当 block 为 **True** 且 timeout 不为 **None** 时，timeout 必须是 >=0 的数字（可能包括用于指定秒的小数部分），get 等待不超过 timeout 秒（如果到达超时时间仍然没有可用项，则 get 抛出 Empty）。*q*.get_nowait() 类似于 *q*.get(**False**)，也类似于 *q*.get(timeout=0.0)。get 移除并返回项：如果 *q* 是 Queue 的直接实例，则按照 put 插入它们的顺序（FIFO），如果 *q* 是 LifoQueue 的实例，则按 LIFO 顺序，如果 *q* 是 PriorityQueue 的实例，则按最小优先顺序。 |

| put, put_nowait | *q*.put(*item*, block=**True**, timeout=**None**) *q*.put_nowait(*item*) |

当 block 为**False**时，如果*q*不满，则 put 将*item*添加到*q*中；否则，put 会引发 Full 异常。当 block 为**True**且 timeout 为**None**时，如果需要，put 将*item*添加到*q*，挂起调用线程，直到*q*不再满为止。当 block 为**True**且 timeout 不为**None**时，timeout 必须是>=0 的数字（可能包括指定秒的小数部分），put 等待不超过 timeout 秒（如果到时*q*仍然满，则 put 引发 Full 异常）。*q*.put_nowait(*item*)类似于*q*.put(*item*, **False**)，也类似于*q*.put(*item*, timeout=0.0)。

| qsize | *q*.qsize() 返回当前在*q*中的项目数。 |
| --- | --- |

*q*维护一个内部的、隐藏的“未完成任务”计数，起始为零。每次调用 put 都会将计数增加一。要将计数减少一，当工作线程完成处理任务时，它调用*q*.task_done。为了同步“所有任务完成”，调用*q*.join：当未完成任务的计数非零时，*q*.join 阻塞调用线程，稍后当计数变为零时解除阻塞；当未完成任务的计数为零时，*q*.join 继续调用线程。

如果你喜欢以其他方式协调线程，不必使用 join 和 task_done，但是当需要使用队列协调线程系统时，它们提供了一种简单实用的方法。

队列提供了“宁可请求宽恕，不要请求许可”（EAFP）的典型例子，见“错误检查策略”。由于多线程，*q*的每个非变异方法（empty、full、qsize）只能是建议性的。当其他线程变异*q*时，线程从非变异方法获取信息的瞬间和线程根据该信息采取行动的下一瞬间之间可能发生变化。因此，依赖“先看再跳”（LBYL）的习惯是徒劳的，而为了修复问题而摆弄锁定则是大量浪费精力。避免脆弱的 LBYL 代码，例如：

```py
`if` *`q`*.empty():
    print('no work to perform')
`else`:  # Some other thread may now have emptied the queue!
    *`x`* = *`q`*.get_nowait()
    work_on(*`x`*)
```

而是采用更简单和更健壮的 EAFP 方法：

```py
`try`:
    *`x`* = *`q`*.get_nowait()
`except` queue.Empty:  # Guarantees the queue was empty when accessed
    print('no work to perform')
`else`:
    work_on(*`x`*)
```

# 多进程模块

多进程模块提供了函数和类，几乎可以像多线程一样编写代码，但是将工作分布到进程而不是线程中：这些包括类 Process（类似于 threading.Thread）和用于同步原语的类（Lock、RLock、Condition、Event、Semaphore、BoundedSemaphore 和 Barrier，每个类似于线程模块中同名的类，以及 Queue 和 JoinableQueue，这两者类似于 queue.Queue）。这些类使得将用于线程的代码转换为使用多进程的版本变得简单；只需注意我们在下一小节中涵盖的差异即可。

通常最好避免在进程之间共享状态：而是使用队列来显式地在它们之间传递消息。然而，在确实需要共享一些状态的罕见情况下，multiprocessing 提供了访问共享内存的类（Value 和 Array），以及更灵活的（包括在网络上不同计算机之间的协调）但带有更多开销的过程子类 Manager，设计用于保存任意数据并让其他进程通过*代理*对象操作该数据。我们在“共享状态：类 Value、Array 和 Manager”中介绍状态共享。

在编写新代码时，与其移植最初使用线程编写的代码，通常可以使用 multiprocessing 提供的不同方法。特别是 Pool 类（在“进程池”中介绍）通常可以简化您的代码。进行多进程处理的最简单和最高级的方式是与 ProcessPoolExecutor 一起使用的 concurrent.futures 模块（在“concurrent.futures 模块”中介绍）。

基于由 Pipe 工厂函数构建或包装在 Client 和 Listener 对象中的 Connection 对象的其他高级方法，甚至更加灵活，但也更加复杂；我们在本书中不再进一步讨论它们。有关更详尽的 multiprocessing 覆盖，请参阅[在线文档](https://oreil.ly/mq8d1)²以及像[PyMOTW](https://oreil.ly/ApoV0)中的第三方在线教程。

## multiprocessing 与线程之间的区别

您可以相对容易地将使用线程编写的代码移植为使用 multiprocessing 的变体，但是您必须考虑几个不同之处。

### 结构差异

您在进程之间交换的所有对象（例如通过队列或作为进程目标函数参数）都是通过 pickle 序列化的，详见“pickle 模块”。因此，您只能交换可以这样序列化的对象。此外，序列化的字节串不能超过约 32 MB（取决于平台），否则会引发异常；因此，您可以交换的对象大小存在限制。

尤其是在 Windows 系统中，子进程*必须*能够将启动它们的主脚本作为模块导入。因此，请确保将主脚本中的所有顶级代码（指不应由子进程再次执行的代码）用通常的**if** __name__ == '__main__'惯用语包围起来，详见“主程序”。

如果进程在使用队列或持有同步原语时被突然终止（例如通过信号），它将无法对该队列或原语执行适当的清理。因此，队列或原语可能会损坏，导致所有尝试使用它的其他进程出现错误。

### 进程类

类 multiprocessing.Process 与 threading.Thread 非常相似；它提供了所有相同的属性和方法（参见表 15-2），以及一些额外的方法，在表 15-11 中列出。它的构造函数具有以下签名：

| 进程 | **类** Process(name=**None**, target=**None**, args=(), kwargs={}) *始终* *使用命名参数* *调用* *Process*：参数的数量和顺序不受规范保证，但参数名是固定的。要么实例化 Process 类本身，传递一个目标函数（*p*.run 在线程启动时调用 *target*(*args*, ***kwargs*)）；或者，而不是传递目标，扩展 Process 类并覆盖其 run 方法。在任一情况下，只有在调用 *p*.start 时执行将开始。name 成为 *p* 的名称。如果 name 为 **None**，Process 为 *p* 生成唯一名称。如果 Process 的子类 *P* 覆盖 __init__，*P*.__init__ *必须* 在任何其他 Process 方法之前在 self 上调用 Process.__init__（通常通过 super 内置函数）。 |
| --- | --- |

表 15-11\. Process 类的附加属性和方法

| authkey | 进程的授权密钥，一个字节串。这是由 os.urandom 提供的随机字节初始化的，但如果需要，您可以稍后重新分配它。用于授权握手的高级用途我们在本书中没有涵盖。 |
| --- | --- |
| close | close() 关闭进程实例并释放所有与之关联的资源。如果底层进程仍在运行，则引发 ValueError。 |
| exitcode | **None** 当进程尚未退出时；否则，进程的退出码。这是一个整数：成功为 0，失败为 >0，进程被杀死为 <0。 |
| kill | kill() 与 terminate 相同，但在 Unix 上发送 SIGKILL 信号。 |
| pid | **None** 当进程尚未启动时；否则，进程的标识符由操作系统设置。 |
| terminate | terminate() 终止进程（不给予其执行终止代码的机会，如清理队列和同步原语；当进程正在使用队列或持有同步原语时，可能会引发错误！）。 |

### 队列的区别

类 multiprocessing.Queue 与 queue.Queue 非常相似，不同之处在于 multiprocessing.Queue 的实例 *q* 不提供方法 join 和 task_done（在“队列模块”中描述）。当 *q* 的方法由于超时而引发异常时，它们会引发 queue.Empty 或 queue.Full 的实例。multiprocessing 没有 queue 的 LifoQueue 和 PriorityQueue 类的等效物。

类 multiprocessing.JoinableQueue 确实提供了方法 join 和 task_done，但与 queue.Queue 相比有语义上的区别：对于 multiprocessing.JoinableQueue 的实例 *q*，调用 *q*.get 的进程在处理完工作单元后 *必须* 调用 *q*.task_done（这不是可选的，就像使用 queue.Queue 时那样）。

您放入 multiprocessing 队列的所有对象必须能够通过 pickle 进行序列化。在执行 q.put 和对象从 q.get 可用之间可能会有延迟。最后，请记住，进程使用 *q* 的突然退出（崩溃或信号）可能会使 *q* 对于任何其他进程不可用。

## 共享状态：类 Value、Array 和 Manager

为了在两个或多个进程之间共享单个原始值，multiprocessing 提供了类 Value，并且对于固定长度的原始值数组，它提供了类 Array。为了获得更大的灵活性（包括共享非原始值和在网络连接的不同系统之间“共享”但不共享内存的情况），multiprocessing 提供了 Manager 类，它是 Process 的子类，但开销较高。我们将在以下小节中详细看看这些类。

### Value 类

类 Value 的构造函数具有以下签名：

| Value | **class** Value(*typecode*, **args*, ***, lock=**True**) *typecode* 是一个字符串，定义值的基本类型，就像在 “数组模块” 中所讨论的 array 模块一样。（另外，*typecode* 可以是来自 ctypes 模块的类型，这在 [第二十五章“ctypes”](https://oreil.ly/python-nutshell-25) 中讨论过，但这很少需要。）*args* 被传递到类型的构造函数中：因此，*args* 要么不存在（在这种情况下，基本类型会按其默认值初始化，通常为 0），要么是一个单一的值，用于初始化该基本类型。

当 lock 为 **True**（默认情况下），Value 将创建并使用一个新的锁来保护实例。或者，您可以将现有的 Lock 或 RLock 实例作为锁传递。甚至可以传递 lock=**False**，但这很少是明智的选择：当您这样做时，实例不受保护（因此在进程之间不同步），并且缺少方法 get_lock。如果传递 lock，则必须将其作为命名参数，使用 lock=*something*。

一个类值的实例 *v* 提供了方法 get_lock，该方法返回（但不获取也不释放）保护 *v* 的锁，并且具有读/写属性 value，用于设置和获取 *v* 的基本原始值。

为了确保对 *v* 的基本原始值的操作是原子性的，请在 **with** *v*.get_lock(): 语句中保护该操作。一个典型的使用例子可能是增强赋值，如下所示：

```py
`with` v.get_lock():
    v.value += 1
```

然而，如果任何其他进程对相同的原始值执行了不受保护的操作——甚至是原子操作，比如简单的赋值操作，如 *v*.value = *x*——那么一切都不确定：受保护的操作和不受保护的操作可能导致系统出现 *竞态条件*。[³] 为了安全起见：如果 *v*.value 上的 *任何* 操作都不是原子的（因此需要通过位于 `with v.get_lock():` 块中的保护），那么通过将它们放在这些块中来保护 *v*.value 上的 *所有* 操作。

### `Array` 类

`multiprocessing.Array` 是一个固定长度的原始值数组，所有项都是相同的原始类型。`Array` 类的构造函数具有如下签名：

| `Array` | **class** `Array`(*typecode*, *size_or_initializer*, ***, lock=**True**) *typecode* 是定义值的原始类型的字符串，就像在 “数组模块” 中所介绍的那样，与模块 `array` 中的处理方式相同。（另外，*typecode* 可以是模块 `ctypes` 中的类型，讨论在 [第二十五章“ctypes”](https://oreil.ly/python-nutshell-25) 中，但这很少是必要的。）*size_or_initializer* 可以是可迭代对象，用于初始化数组，或者是用作数组长度的整数，在这种情况下，数组的每个项都初始化为 0。

当 `lock` 为 **True**（默认值）时，`Array` 会创建并使用一个新的锁来保护实例。或者，您可以将现有的 `Lock` 或 `RLock` 实例作为锁传递给 `lock`。您甚至可以传递 `lock=False`，但这很少是明智的：当您这样做时，实例不受保护（因此在进程之间不同步），并且缺少 `get_lock` 方法。如果您传递 `lock`，则*必须*将其作为命名参数传递，使用 `lock=*something*`。 |

`Array` 类的实例 *a* 提供了方法 `get_lock`，该方法返回（但不获取也不释放）保护 *a* 的锁。

*a* 通过索引和切片访问，并通过对索引或切片赋值来修改。*a* 是固定长度的：因此，当您对切片赋值时，您必须将一个与您要赋值的切片完全相同长度的可迭代对象赋值给它。*a* 也是可迭代的。

在特殊情况下，当 *a* 使用 `'c'` 类型码构建时，您还可以访问 *a*.value 以获取 *a* 的内容作为字节串，并且您可以将任何长度不超过 len(*a*) 的字节串分配给 *a*.value。当 *s* 是长度小于 len(*a*) 的字节串时，*a*.value = *s* 意味着 *a*[:len(*s*)+1] = *s* + b'\0'；这反映了 C 语言中字符串的表示，以 0 字节终止。例如：

```py
a = multiprocessing.Array('c', b'four score and seven')
a.value = b'five'
print(a.value)   *`# prints`* *`b'five'`*
print(a[:])      *`# prints`* *`b'five\xOOscore and seven'`*
```

### `Manager` 类

multiprocessing.Manager 是 multiprocessing.Process 的子类，具有相同的方法和属性。此外，它还提供方法来构建任何 multiprocessing 同步原语的实例，包括 Queue、dict、list 和 Namespace。Namespace 是一个类，允许您设置和获取任意命名属性。每个方法都以它所构建实例的类名命名，并返回一个*代理*到这样一个实例，任何进程都可以使用它来调用方法（包括 dict 或 list 实例的索引等特殊方法）。

代理对象大多数操作符、方法和属性访问都会传递给它们代理的实例；但是，它们不会传递*比较*操作符 —— 如果你需要比较，就需要获取代理对象的本地副本。例如：

```py
manager = multiprocessing.Manager()
p = manager.list()

p[:] = [1, 2, 3]
print(p == [1, 2, 3])       *`# prints`* *`False`**`,`* * `it compares with p itself`*
print(list(p) == [1, 2, 3]) *`# prints`* *`True`**`,`* * `it compares with copy`*
```

Manager 的构造函数不接受任何参数。有高级方法来定制 Manager 子类，以允许来自无关进程的连接（包括通过网络连接的不同计算机上的进程），并提供不同的构建方法集，但这些内容不在本书的讨论范围之内。使用 Manager 的一个简单而通常足够的方法是显式地将它生成的代理传输给其他进程，通常通过队列或作为进程的*目标*函数的参数。

例如，假设有一个长时间运行的 CPU 绑定函数 *f*，给定一个字符串作为参数，最终返回对应的结果；给定一组字符串，我们希望生成一个字典，其键为字符串，值为相应的结果。为了能够跟踪 *f* 运行在哪些进程上，我们在调用 *f* 前还打印进程 ID。示例 15-1 展示了如何实现这一点。

##### 示例 15-1\. 将工作分配给多个工作进程

```py
`import` multiprocessing `as` mp
`def` f(s):
    *`"""Run a long time, and eventually return a result."""`*
 `import` time`,` random
    time.sleep(random.random()*2)  *`# simulate slowness`*
    `return` s+s                     *`# some computation or other`*

`def` runner(s, d):
    print(os.getpid(), s)
    d[s] = f(s)

`def` make_dict(strings):
    mgr = mp.Manager()
    d = mgr.dict()
    workers = []
    `for` s `in` strings:
        p = mp.Process(target=runner, args=(s, d))
        p.start()
        workers.append(p)
    `for` p `in` workers:
        p.join()
    `return` {**d}
```

## 进程池

在实际应用中，应始终避免创建无限数量的工作进程，就像我们在示例 15-1 中所做的那样。性能的提升仅限于您计算机上的核心数（可通过调用 multiprocessing.cpu_count 获取），或者略高于或略低于此数，具体取决于您的平台、代码是 CPU 绑定还是 I/O 绑定，计算机上运行的其他任务等微小差异。创建比这个最佳数量多得多的工作进程会带来大量额外的开销，却没有任何补偿性的好处。

因此，一个常见的设计模式是启动一个带有有限数量工作进程的*池*，并将工作分配给它们。multiprocessing.Pool 类允许您编排这种模式。

### 池类

类 Pool 的构造函数签名为：

| 池 | **类** Pool(processes=None, initializer=**None**, initargs=(), maxtasksperchild=**None**)

processes 是池中的进程数；默认值为 cpu_count 返回的值。当 initializer 不为 **None** 时，它是一个函数，在池中每个进程开始时调用，带有 initargs 作为参数，例如 initializer(**initargs*)。

当 maxtasksperchild 不为 **None** 时，它是池中每个进程可以执行的最大任务数。当池中的进程执行了这么多任务后，它终止，然后一个新的进程启动并加入池。当 maxtasksperchild 为 **None**（默认值）时，每个进程的生命周期与池一样长。

类 Pool 的实例 *p* 提供了 表 15-12 中列出的方法（每个方法只能在构建实例 *p* 的进程中调用）。

表 15-12\. 类 Pool 的实例 p 的方法

| apply | apply(*func*, args=(), kwds={}) 在工作进程中的任意一个，运行 *func*(**args*, ***kwds*），等待其完成，并返回 *func* 的结果。 |
| --- | --- |
| apply_async | apply_async(*func*, args=(), kwds={}, callback=**None**) 在工作进程中的任意一个，开始运行 *func*(**args*, ***kwds*），并且不等待其完成，立即返回一个 AsyncResult 实例，该实例最终提供 *func* 的结果，当结果准备就绪时。 （AsyncResult 类在下一节中讨论。）当 callback 不为 **None** 时，它是一个函数，用 *func* 的结果作为唯一参数调用（在调用 apply_async 的进程中的新线程中），当结果准备就绪时；callback 应该快速执行，因为否则会阻塞调用进程。如果参数是可变的，则 callback 可以改变其参数；callback 的返回值是无关紧要的（因此，最佳、最清晰的风格是让它返回 **None**）。 |
| close | close() 设置一个标志，禁止进一步向池提交任务。工作进程在完成所有未完成的任务后终止。 |
| imap | imap(*func*, *iterable*, chunksize=1) 返回一个迭代器，在每个 *iterable* 的项上调用 *func*，顺序执行。chunksize 确定连续发送给每个进程的项数；在非常长的 *iterable* 上，较大的 chunksize 可以提高性能。当 chunksize 为 1（默认值）时，返回的迭代器有一个方法 next（即使迭代器方法的规范名称是 __next__），接受一个可选的 *timeout* 参数（浮点数值，单位秒），在 *timeout* 秒后仍未准备就绪时引发 multiprocessing.TimeoutError。 |
| im⁠a⁠p⁠_​u⁠n⁠o⁠r⁠d⁠e⁠r⁠e⁠d | imap_unordered(*func*, *iterable*, chunksize=1) 与 imap 相同，但结果的顺序是任意的（在迭代顺序不重要时，有时可以提高性能）。如果函数的返回值包含足够的信息，以允许将结果与用于生成它们的 iterable 的值关联，则通常很有帮助。 |
| join | join() 等待所有工作进程退出。您必须在调用 join 之前调用 close 或 terminate。 |
| map | map(*func*, *iterable*, chunksize=1) 在池中的工作进程上按顺序对 *iterable* 中的每个项目调用 *func*；等待它们全部完成，并返回结果列表。chunksize 确定每个进程发送多少连续项目；在非常长的 *iterable* 上，较大的 chunksize 可以提高性能。 |

| map_async | map_async(*func*, *iterable*, chunksize=1, callback=**None**) 安排在池中的工作进程上对可迭代对象 *iterable* 中的每个项目调用 *func*；在等待任何操作完成之前，立即返回一个 AsyncResult 实例（在下一节中描述），该实例最终提供 *func* 的结果列表，当该列表准备就绪时。

当 callback 不为 **None** 时，它是一个函数（在调用 map_async 的进程的单独线程中调用），其参数是按顺序排列的 *func* 的结果列表，当该列表准备就绪时。callback 应该快速执行，否则会阻塞进程。callback 可能会改变其列表参数；callback 的返回值是无关紧要的（因此，最好、最清晰的风格是让其返回 **None**）。|

| terminate | terminate() 一次性终止所有工作进程，而无需等待它们完成工作。 |
| --- | --- |

例如，这是一种基于池的方法来执行与 Example 15-1 中代码相同的任务：

```py
`import` os, multiprocessing `as` mp
`def` f(s):
    *`"""Run a long time, and eventually return a result."""`*
 `import` time, random
    time.sleep(random.random()*2)  *`# simulate slowness`*
    `return` s+s                     *`# some computation or other`*

`def` runner(s):
    print(os.getpid(), s)
    `return` s, f(s)

`def` make_dict(strings):
    `with` mp.Pool() `as` pool:
        d = dict(pool.imap_unordered(runner, strings))
        `return` d
```

### 异步结果类

类 Pool 的方法 apply_async 和 map_async 返回类 AsyncResult 的一个实例。类 AsyncResult 的一个实例 *r* 提供了 Table 15-13 中列出的方法。

表 15-13\. 类 AsyncResult 实例 r 的方法

| get | get(timeout=**None**) 阻塞并在结果准备就绪时返回结果，或在计算结果时重新引发引发的异常。当 timeout 不为 **None** 时，它是以秒为单位的浮点值；如果在超时秒后结果尚未准备就绪，则 get 会引发 multiprocessing.TimeoutError。 |
| --- | --- |
| ready | ready() 不阻塞；如果调用已完成并返回结果或已引发异常，则返回 **True**；否则返回 **False**。 |
| successful | successful() 不阻塞；如果结果已准备就绪且计算未引发异常，则返回 **True**；如果计算引发异常，则返回 **False**。如果结果尚未准备就绪，successful 将引发 AssertionError。 |
| wait | wait(timeout=**None**) 阻塞并等待结果准备就绪。当 timeout 不为 **None** 时，它是以秒为单位的浮点值：如果在超时秒后结果尚未准备就绪，则 wait 会引发 multiprocessing.TimeoutError。 |

### 线程池类

multiprocessing.pool 模块还提供了一个名为 ThreadPool 的类，其接口与 Pool 完全相同，但在单个进程中使用多个线程实现（而不是多个进程，尽管模块的名称如此）。使用 ThreadPool 的等效 make_dict 代码，与 示例 15-1 中使用 ThreadPoolExecutor 的代码类似：

```py
`def` make_dict(strings):
    num_workers=3
    `with` mp.pool.ThreadPool(num_workers) `as` pool:
        d = dict(pool.imap_unordered(runner, strings))
        `return` d
```

由于 ThreadPool 使用多个线程但限于在单个进程中运行，因此最适合于其各个线程执行重叠 I/O 的应用程序。正如前面所述，当工作主要受 CPU 限制时，Python 线程提供的优势很小。

在现代 Python 中，您通常应优先使用模块 concurrent.futures 中的抽象类 Executor（下一节将介绍），以及其两个实现，ThreadPoolExecutor 和 ProcessPoolExecutor。特别是，由 concurrent.futures 实现的执行器类的 submit 方法返回的 Future 对象与 asyncio 模块兼容（正如前面提到的，我们不在本书中涵盖 asyncio，但它仍然是 Python 最近版本中许多并发处理的重要部分）。由 multiprocessing 实现的池类的 apply_async 和 map_async 方法返回的 AsyncResult 对象不兼容 asyncio。

# 并发.futures 模块

并发包提供了一个单一模块，即 futures。concurrent.futures 提供了两个类，ThreadPoolExecutor（使用线程作为工作者）和 ProcessPoolExecutor（使用进程作为工作者），它们实现了相同的抽象接口 Executor。通过调用该类并指定一个参数 max_workers 来实例化任何一种池，该参数指定池应包含的线程或进程数。您可以省略 max_workers，让系统选择工作者数。

Executor 类的实例 *e* 支持 表 15-14 中的方法。

表 15-14。类 Executor 的实例 *e* 的方法

| map | map(*func*, **iterables*, timeout=**None**, chunksize=1) 返回一个迭代器 *it*，其项是按顺序使用多个工作线程或进程并行执行 *func* 的结果。当 *timeout* 不为 **None** 时，它是一个浮点秒数：如果在 timeout 秒内 next(*it*) 未产生任何结果，则引发 concurrent.futures.TimeoutError。

您还可以选择（仅按名称）指定参数 chunksize：对于 ThreadPoolExecutor 无效；对于 ProcessPoolExecutor，它设置每个可迭代项中的每个项目传递给每个工作进程的数量。 |

| shutdown | shutdown(wait=**True**) 禁止进一步调用 map 或 submit。当 wait 为 **True** 时，shutdown 阻塞直到所有待处理的 Future 完成；当 **False** 时，shutdown 立即返回。在任何情况下，进程直到所有待处理的 Future 完成后才终止。 |
| --- | --- |
| submit | `submit(*func*, **a*, ***k*)` 确保 *func*(**a*, ***k*) 在线程池的任一进程或线程中执行。不会阻塞，立即返回一个 Future 实例。 |

任何 Executor 实例也是上下文管理器，因此适合在 **with** 语句中使用（__exit__ 行为类似于 shutdown(wait=**True**))。

例如，这里是一个基于并发的方法来执行与 示例 15-1 中相同任务的方法：

```py
`import` concurrent.futures `as` cf
`def` f(s):
    *`"""run a long time and eventually return a result"""`*
    *`# ...`* *`like before!`*

`def` runner(s):
    `return` s, f(s)

`def` make_dict(strings):
    `with` cf.ProcessPoolExecutor() `as` e:
        d = dict(e.map(runner, strings))
    `return` d
```

Executor 的 submit 方法返回一个 Future 实例。Future 实例 *f* 提供了表 15-15 中描述的方法。

表 15-15\. Future 类实例 *f* 的方法

| add_do⁠n⁠e⁠_​c⁠a⁠l⁠lback | `add_done_callback(*func*)` 将可调用对象 *func* 添加到 *f* 上；当 *f* 完成（即取消或完成）时，会调用 *func*，并以 *f* 作为唯一参数。 |
| --- | --- |
| cancel | `cancel()` 尝试取消调用。当调用正在执行且无法被取消时，返回 **False**；否则返回 **True**。 |
| cancelled | `cancelled()` 返回 **True** 如果调用成功取消；否则返回 **False**。 |
| done | `done()` 返回 **True** 当调用完成（即已完成或成功取消）。 |
| exception | `exception(timeout=**None**)` 返回调用引发的异常，如果调用没有引发异常则返回 **None**。当 timeout 不为 **None** 时，它是一个浮点数秒数。如果调用在 timeout 秒内未完成，exception 抛出 concurrent.futures.TimeoutError；如果调用被取消，exception 抛出 concurrent.futures.CancelledError。 |
| result | `result(timeout=**None**)` 返回调用的结果。当 timeout 不为 **None** 时，它是一个浮点数秒数。如果调用在 timeout 秒内未完成，result 抛出 concurrent.futures.TimeoutError；如果调用被取消，result 抛出 concurrent.futures.CancelledError。 |
| running | `running()` 返回 **True** 当调用正在执行且无法被取消；否则返回 **False**。 |

concurrent.futures 模块还提供了两个函数，详见 表 15-16。

表 15-16\. concurrent.futures 模块的函数

| a⁠s⁠_​c⁠o⁠m⁠pleted | `as_completed(*fs*, timeout=**None**)` 返回一个迭代器 *it*，迭代 Future 实例，这些实例是可迭代对象 *fs* 的项。如果 *fs* 中有重复项，则每个项只返回一次。*it* 按顺序逐个返回已完成的 Future 实例。如果 timeout 不为 **None**，它是一个浮点数秒数；如果在 timeout 秒内从上一个已完成实例之后尚未返回新的实例，则 as_completed 抛出 concurrent.futures.Timeout。 |
| --- | --- |

| wait | wait(*fs*, timeout=**None**, return_when=ALL_COMPLETED) 等待 iterable *fs*中作为项目的未来实例。返回一个命名的 2 元组集合：第一个集合名为 done，包含在 wait 返回之前已完成（意味着它们已经完成或被取消）的未来实例；第二个集合名为 not_done，包含尚未完成的未来实例。

如果 timeout 不为**None**，则 timeout 是一个浮点数秒数，表示在返回之前允许等待的最长时间（当 timeout 为**None**时，wait 仅在满足 return_when 时返回，不管之前经过了多少时间）。

return_when 控制 wait 何时返回；它必须是 concurrent.futures 模块提供的三个常量之一：

ALL_COMPLETED

当所有未来实例完成或被取消时返回。

FIRST_COMPLETED

当任何一个未来完成或被取消时返回。

FIRST_EXCEPTION

当任何未来引发异常时返回；如果没有未来引发异常，则变成等价于 ALL_COMPLETED。

|

这个版本的 make_dict 演示了如何使用 concurrent.futures.as_completed 来在每个任务完成时处理它（与使用 Executor.map 的前一个示例形成对比，后者总是按照提交顺序返回任务）：

```py
`import` concurrent.futures `as` cf

`def` make_dict(strings):
    `with` cf.ProcessPoolExecutor() `as` e:
        futures = [e.submit(runner, s) `for` s `in` strings]
        d = dict(f.result() `for` f `in` cf.as_completed(futures))
    `return` d
```

# 线程程序架构

一个线程程序应该总是尽量安排一个*单一*线程“拥有”任何外部于程序的对象或子系统（如文件、数据库、GUI 或网络连接）。虽然有多个处理同一外部对象的线程是可能的，但通常会造成难以解决的问题。

当您的线程程序必须处理某些外部对象时，为这类交互专门分配一个线程，并使用一个队列对象，从中外部交互线程获取其他线程提交的工作请求。外部交互线程通过将结果放在一个或多个其他队列对象中来返回结果。以下示例展示了如何将此架构打包成一个通用的可重用类，假设外部子系统上的每个工作单元可以用可调用对象表示：

```py
`import` threading, queue

`class` ExternalInterfacing(threading.Thread):
    `def` __init__(self, external_callable, **kwds):
        super().__init__(**kwds)
        self.daemon = `True`
        self.external_callable = external_callable
        self.request_queue = queue.Queue()
        self.result_queue = queue.Queue()
        self.start()

    `def` request(self, *args, **kwds):
        *`"""called by other threads as external_callable would be"""`*
        self.request_queue.put((args, kwds))
        `return` self.result_queue.get()

    `def` run(self):
        `while` `True`:
            a, k = self.request_queue.get()
            self.result_queue.put(self.external_callable(*a, **k))
```

一旦实例化了某个 ExternalInterfacing 对象*ei*，任何其他线程都可以调用*ei*.request，就像在缺乏这种机制的情况下调用 external_callable 一样（根据需要带有或不带参数）。ExternalInterfacing 的优点在于对 external_callable 的调用是*串行化*的。这意味着仅有一个线程（绑定到*ei*的 Thread 对象）按照某个定义好的顺序顺序执行它们，没有重叠、竞争条件（依赖于哪个线程“恰好先到达”）或其他可能导致异常情况的问题。

如果您需要将多个可调用对象串行化在一起，可以将可调用对象作为工作请求的一部分传递，而不是在类 ExternalInterfacing 的初始化时传递它，以获得更大的通用性。以下示例展示了这种更通用的方法：

```py
`import` threading, queue

`class` Serializer(threading.Thread):
    def __init__(self, **kwds):
        super().__init__(**kwds)
        self.daemon = `True`
        self.work_request_queue = queue.Queue()
        self.result_queue = queue.Queue()
        self.start()

    `def` apply(self, callable, *args, **kwds):
        *``"""called by other threads as `callable` would be"""``*
        self.work_request_queue.put((callable, args, kwds))
        `return` self.result_queue.get()

    `def` run(self):
        `while` `True`:
            callable, args, kwds = self.work_request_queue.get()
            self.result_queue.put(callable(*args, **kwds))
```

一旦实例化了一个 Serializer 对象*ser*，任何其他线程都可以调用*ser*.apply(external_callable)，就像没有这种机制时会调用 external_callable 一样（根据需要可能带有或不带有进一步的参数）。Serializer 机制与 ExternalInterfacing 具有相同的优点，不同之处在于，所有调用由同一个或不同的可调用对象包装在单个*ser*实例中的调用现在都是串行化的。

整个程序的用户界面是一个外部子系统，因此应该由一个单独的线程来处理——具体来说，是程序的主线程（对于某些用户界面工具包，这是强制性的，即使使用其他不强制的工具包，这样做也是建议的）。因此，Serializer 线程是不合适的。相反，程序的主线程应仅处理用户界面问题，并将所有实际工作委派给接受工作请求的工作线程，这些工作线程在一个队列对象上接受工作请求，并在另一个队列上返回结果。一组工作线程通常被称为*线程池*。正如下面的示例所示，所有工作线程应共享一个请求队列和一个结果队列，因为只有主线程才会发布工作请求并获取结果：

```py
`import` threading

`class` Worker(threading.Thread):
    IDlock = threading.Lock()
    request_ID = 0

    `def` __init__(self, requests_queue, results_queue, **kwds):
        super().__init__(**kwds)
        self.daemon = `True`
        self.request_queue = requests_queue
        self.result_queue = results_queue
        self.start()

    `def` perform_work(self, callable, *args, **kwds):
        *``"""called by main thread as `callable` would be, 			   but w/o return"""``*
        `with` self.IDlock:
            Worker.request_ID += 1
            self.request_queue.put(
                (Worker.request_ID, callable, args, kwds))
            `return` Worker.request_ID

    `def` run(self):
        `while` `True`:
            request_ID, callable, a, k = self.request_queue.get()
            self.result_queue.put((request_ID, callable(*a, **k)))
```

主线程创建两个队列，然后实例化工作线程，如下所示：

```py
`import` queue
requests_queue = queue.Queue()
results_queue = queue.Queue()
number_of_workers = 5
`for` i `in` range(number_of_workers):
    worker = Worker(requests_queue, results_queue)
```

每当主线程需要委派工作（执行可能需要较长时间来生成结果的可调用对象），主线程调用*worker*.perform_work(*callable*)，就像没有这种机制时会调用*callable*一样（根据需要可能带有或不带有进一步的参数）。然而，perform_work 并不返回调用的结果。主线程得到的是一个标识工作请求的 ID。当主线程需要结果时，它可以跟踪该 ID，因为请求的结果在出现时都标有该 ID。这种机制的优点在于，主线程不会阻塞等待可调用的执行完成，而是立即变为可用状态，可以立即回到处理用户界面的主要业务上。

主线程必须安排检查 results_queue，因为每个工作请求的结果最终会出现在那里，标记有请求的 ID，当从队列中取出该请求的工作线程计算出结果时。主线程如何安排检查用户界面事件和从工作线程返回到结果队列的结果，取决于使用的用户界面工具包，或者——如果用户界面是基于文本的——取决于程序运行的平台。

一个广泛适用但并非始终最佳的一般策略是让主线程*轮询*（定期检查结果队列的状态）。在大多数类 Unix 平台上，模块 signal 的 alarm 函数允许轮询。tkinter GUI 工具包提供了一个 after 方法，可用于轮询。某些工具包和平台提供了更有效的策略（例如，让工作线程在将某些结果放入结果队列时警告主线程），但没有通用的、跨平台、跨工具包的方法可以安排这样的操作。因此，以下人工示例忽略了用户界面事件，只是通过在几个工作线程上评估随机表达式并引入随机延迟来模拟工作，从而完成了前面的示例：

```py
`import` random, time, queue, operator
*`# copy here class Worker as defined earlier`*

requests_queue = queue.Queue()
results_queue = queue.Queue()

number_of_workers = 3
workers = [Worker(requests_queue, results_queue)
           `for` i `in` range(number_of_workers)]
work_requests = {}

operations = {
    '+': operator.add,
    '-': operator.sub,
    '*': operator.mul,
    '/': operator.truediv,
    '%': operator.mod,
}

`def` pick_a_worker():
    return random.choice(workers)

`def` make_work():
    o1 = random.randrange(2, 10)
    o2 = random.randrange(2, 10)
    op = random.choice(list(operations))
    `return` f'{o1} {op} {o2}'

`def` slow_evaluate(expression_string):
    time.sleep(random.randrange(1, 5))
    op1, oper, op2 = expression_string.split()
    arith_function = operations[oper]
    `return` arith_function(int(op1), int(op2))

`def` show_results():
    `while` `True`:
        `try`:
            completed_id, results = results_queue.get_nowait()
        `except` queue.Empty:
            `return`
        work_expression = work_requests.pop(completed_id)
        print(f'Result {completed_id}: {work_expression} -> {results}')

`for` i `in` range(10):
    expression_string = make_work()
    worker = pick_a_worker()
    request_id = worker.perform_work(slow_evaluate, expression_string)
    work_requests[request_id] = expression_string
    print(f'Submitted request {request_id}: {expression_string}')
    time.sleep(1.0)
    show_results()

`while` work_requests:
    time.sleep(1.0)
    show_results()
```

# 进程环境

操作系统为每个进程*P*提供一个*环境*，即一组变量，变量名为字符串（通常按约定为大写标识符），其值也为字符串。在“环境变量”中，我们讨论了影响 Python 操作的环境变量。操作系统 shell 通过 shell 命令和其他在该部分提到的方法提供了检查和修改环境的方式。

# 进程环境是自包含的

任何进程*P*的环境在*P*启动时确定。启动后，只有*P*自己能够改变*P*的环境。对*P*环境的更改仅影响*P*本身：环境*不是*进程间通信的手段。*P*的任何操作都不会影响*P*的父进程（启动*P*的进程）的环境，也不会影响任何*P*之前启动的子进程现在正在运行的环境，或者与*P*无关的任何进程。*P*的子进程通常会在*P*创建该进程时获取一个*P*环境的副本作为启动环境。从这个狭义的意义上说，对*P*环境的更改确实会影响在此类更改之后由*P*启动的子进程。

模块 os 提供了属性 environ，这是一个映射，表示当前进程的环境。当 Python 启动时，它从进程环境初始化 os.environ。如果平台支持此类更新，对 os.environ 的更改将更新当前进程的环境。os.environ 中的键和值必须是字符串。在 Windows 上（但在类 Unix 平台上不是这样），os.environ 中的键隐式转换为大写。例如，以下是如何尝试确定您正在运行的 shell 或命令处理器：

```py
`import` os
shell = os.environ.get('COMSPEC')
`if` shell `is` `None`:
    shell = os.environ.get('SHELL')
`if` shell `is` `None`:
    shell = 'an unknown command processor'
print('Running under ', shell)
```

当 Python 程序改变其环境（例如通过`os.environ['X'] = 'Y'`），这不会影响启动该程序的 shell 或命令处理器的环境。正如已经解释过的，对于**所有**编程语言，包括 Python，进程环境的更改仅影响进程本身，而不影响当前正在运行的其他进程。

# 运行其他程序

您可以通过 os 模块中的低级函数运行其他程序，或者（在更高且通常更可取的抽象级别上）使用 subprocess 模块。

## 使用子进程模块

subprocess 模块提供了一个非常广泛的类：Popen，支持许多不同的方式来运行另一个程序。Popen 的构造函数签名如下：

| Popen | **class** Popen(*args*, bufsize=0, executable=**None**, capture_output=**False**, stdin=**None**, stdout=**None**, stderr=**None**, preexec_fn=**None**, close_fds=**False**, shell=**False**, cwd=**None**, env=**None**, text=**None**, universal_newlines=**False**, startupinfo=**None**, creationflags=0) Popen 启动一个子进程来运行一个独立的程序，并创建并返回一个对象*p*，代表该子进程。*args*是必需的参数，而许多可选的命名参数控制子进程运行的所有细节。

当在子进程创建过程中发生任何异常（在明确程序启动之前），Popen 会将该异常与名为 child_traceback 的属性一起重新引发到调用过程中，child_traceback 是子进程的 Python traceback 对象。这样的异常通常是 OSError 的一个实例（或者可能是 TypeError 或 ValueError，表示您传递给 Popen 的参数在类型或值上是无效的）。

# subprocess.run()是 Popen 的一个便捷包装函数。

subprocess 模块包括 run 函数，该函数封装了一个 Popen 实例，并在其上执行最常见的处理流程。run 接受与 Popen 构造函数相同的参数，运行给定的命令，等待完成或超时，并返回一个 CompletedProcess 实例，其中包含返回码以及 stdout 和 stderr 的内容。

如果需要捕获命令的输出，则最常见的参数值将是将 capture_output 和 text 参数设置为**True**。

### 要运行什么，以及如何运行

*args*是一个字符串序列：第一项是要执行的程序的路径，后续的项（如果有）是要传递给程序的参数（当不需要传递参数时，*args*也可以只是一个字符串）。当 executable 不为**None**时，它将覆盖*args*以确定要执行的程序。当 shell 为**True**时，executable 指定要用来运行子进程的 shell；当 shell 为**True**且 executable 为**None**时，在类 Unix 系统上使用的 shell 是*/bin/sh*（在 Windows 上，它是 os.environ['COMSPEC']）。

### 子进程文件

stdin、stdout 和 stderr 分别指定了子进程的标准输入、输出和错误文件。每个可以是 PIPE，这会创建一个到/从子进程的新管道；**None**，意味着子进程要使用与此（“父”）进程相同的文件；或者是已经适当打开的文件对象（或文件描述符）（对于读取，用于标准输入；对于写入，用于标准输出和标准错误）。stderr 还可以是 subprocess.STDOUT，这意味着子进程的标准错误必须使用与其标准输出相同的文件。⁴ 当 capture_output 为 true 时，不能指定 stdout 或 stderr：行为就像每个都指定为 PIPE 一样。bufsize 控制这些文件的缓冲（除非它们已经打开），其语义与 open 函数中的相同参数的语义相同（默认为 0，表示“无缓冲”）。当 text（或其同义词 universal_newlines，提供向后兼容性）为 true 时，stdout 和 stderr（除非它们已经打开）将被打开为文本文件；否则，它们将被打开为二进制文件。当 close_fds 为 true 时，在子进程执行其程序或 shell 之前，所有其他文件（除了标准输入、输出和错误）将被关闭。

### 其他，高级参数

当 preexec_fn 不为 **None** 时，必须是一个函数或其他可调用对象，并且在子进程执行其程序或 shell 之前调用它（仅适用于类 Unix 系统，其中调用发生在 fork 之后和 exec 之前）。

当 cwd 不为 **None** 时，必须是一个给出现有目录的完整路径的字符串；在子进程执行其程序或 shell 之前，当前目录会切换到 cwd。

当 env 不为 **None** 时，必须是一个映射，其中键和值都是字符串，并完全定义了新进程的环境；否则，新进程的环境是当前父进程中活动环境的副本。

startupinfo 和 creationflags 是传递给 CreateProcess Win32 API 调用的 Windows-only 参数，用于创建子进程，用于特定于 Windows 的目的（本书不进一步涵盖它们，因为本书几乎完全专注于 Python 的跨平台使用）。

### subprocess.Popen 实例的属性

类 Popen 的实例 *p* 提供了 Table 15-17 中列出的属性。

Table 15-17\. 类 Popen 的实例 *p* 的属性

| args | Popen 的 *args* 参数（字符串或字符串序列）。 |
| --- | --- |
| pid | 子进程的进程 ID。 |
| returncode | None 表示子进程尚未退出；否则，是一个整数：0 表示成功终止，>0 表示以错误代码终止，或 <0 如果子进程被信号杀死。 |
| stderr, stdin, stdout | 当 Popen 的相应参数是 subprocess.PIPE 时，这些属性中的每一个都是包装相应管道的文件对象；否则，这些属性中的每一个都是 **None**。使用 *p* 的 communicate 方法，而不是从这些文件对象读取和写入，以避免可能的死锁。 |

### subprocess.Popen 实例的方法

类 Popen 的实例 *p* 提供了表 15-18 中列出的方法。

表 15-18\. 类 Popen 的实例 *p* 的方法

| communicate | *p*.communicate(input=**None**, timeout=**None**) 将字符串 input 作为子进程的标准输入（当 input 不为 **None** 时），然后将子进程的标准输出和错误文件读入内存中的字符串 *so* 和 *se*，直到两个文件都完成，最后等待子进程终止并返回对（两项元组）（*so*，*se*）。 |
| --- | --- |
| poll | *p*.poll() 检查子进程是否已终止；如果已终止，则返回 *p*.returncode；否则返回 **None**。 |
| wait | *p*.wait(timeout=**None**) 等待子进程终止，然后返回 *p*.returncode。如果子进程在 timeout 秒内未终止，则引发 TimeoutExpired 异常。 |

## 使用 os 模块运行其他程序

你的程序通常运行其他进程的最佳方式是使用前一节介绍的 subprocess 模块。然而，os 模块（在第十一章介绍）也提供了几种较低级别的方式来实现这一点，在某些情况下可能更简单。

运行另一个程序的最简单方法是通过 os.system 函数，尽管这种方法没有办法*控制*外部程序。os 模块还提供了几个以 exec 开头的函数，这些函数提供了精细的控制。由 exec 函数之一运行的程序会替换当前程序（即 Python 解释器）在同一进程中。因此，在实践中，您主要在支持使用 fork 的平台上使用 exec 函数（即类 Unix 平台）。以 spawn 和 popen 开头的 os 函数提供了中间简单性和强大性：它们是跨平台的，并且不像 system 那样简单，但对于许多目的来说足够简单。

exec 和 spawn 函数运行给定的可执行文件，给定可执行文件的路径、传递给它的参数，以及可选的环境映射。system 和 popen 函数执行一个命令，这是一个字符串传递给平台的默认 shell 的新实例（通常在 Unix 上是 */bin/sh*，在 Windows 上是 *cmd.exe*）。*命令*是一个比*可执行文件*更一般的概念，因为它可以包含特定于当前平台的 shell 功能（管道、重定向和内置 shell 命令）使用的 shell 语法。

os 提供了表 15-19 列出的函数。

表 15-19\. 与进程相关的 os 模块的函数

| execl, execle,

execlp,

execv,

execve,

execvp,

execvpe | execl(*path*, **args*), execle(*path*, **args*),

execlp(*path*,**args*),

execv(*path*, *args*),

execve(*path*, *args*, *env*),

execvp(*path*, *args*),

execvpe(*path*, *args*, *env*)

运行由字符串 *path* 指示的可执行文件（程序），替换当前进程中的当前程序（即 Python 解释器）。函数名中编码的区别（在前缀 exec 之后）控制新程序的发现和运行的三个方面：

+   *path* 是否必须是程序可执行文件的完整路径，还是该函数可以接受一个名称作为 *path* 参数并在多个目录中搜索可执行文件，就像操作系统 shell 一样？execlp、execvp 和 execvpe 可以接受一个 *path* 参数，该参数只是一个文件名而不是完整路径。在这种情况下，函数将在 os.environ['PATH'] 中列出的目录中搜索具有该名称的可执行文件。其他函数要求 *path* 是可执行文件的完整路径。

+   函数是否接受新程序的参数作为单个序列参数 *args*，还是作为函数的单独参数？以 execv 开头的函数接受一个参数 *args*，该参数是要用于新程序的参数序列。以 execl 开头的函数将新程序的参数作为单独的参数（特别是 execle，它使用其最后一个参数作为新程序的环境）。

+   函数是否接受新程序的环境作为显式映射参数 *env*，或者隐式使用 os.environ？execle、execve 和 execvpe 接受一个参数 *env*，该参数是要用作新程序环境的映射（键和值必须是字符串），而其他函数则使用 os.environ 用于此目的。

    每个 exec 函数使用 *args* 中的第一个项作为告知新程序其正在运行的名称（例如，在 C 程序的 main 中的 argv[0]）；只有 args[1:] 是新程序的真正参数。

|

| popen | popen(*cmd*, mode='r', buffering=-1) 运行字符串命令 *cmd* 在一个新进程 *P* 中，并返回一个文件对象 *f*，该对象包装了与 *P* 的标准输入或来自 *P* 的标准输出的管道（取决于模式）；*f* 使用文本流而不是原始字节。模式和缓冲区的含义与 Python 的 open 函数相同，见 “使用 open 创建文件对象”。当模式为 'r'（默认）时，*f* 是只读的，并包装 *P* 的标准输出。当模式为 'w' 时，*f* 是只写的，并包装 *P* 的标准输入。

*f* 与其他类似文件的主要区别在于方法 *f*.close 的行为。 *f*.close 等待 *P* 终止并返回 **None**，正如文件类对象的关闭方法通常所做的那样，当 *P* 成功终止时。然而，如果操作系统将整数错误码 *c* 与 *P* 的终止关联起来，表示 *P* 的终止失败，*f*.close 返回 *c*。在 Windows 系统上，*c* 是子进程的有符号整数返回码。 |

| spawnv, spawnve | spawnv(*mode*, *path*, *args*), spawnve(*mode*, *path*, *args*, *env*)

这些函数在新进程 *P* 中运行由 *path* 指示的程序，参数作为序列 *args* 传递。spawnve 使用映射 *env* 作为 *P* 的环境（键和值必须是字符串），而 spawnv 则使用 os.environ 来实现。仅在 Unix-like 平台上，还有其他 os.spawn 的变体，对应于 os.exec 的变体，但 spawnv 和 spawnve 是 Windows 上唯一存在的两个。

*mode* 必须是 os 模块提供的两个属性之一：os.P_WAIT 表示调用进程等待新进程终止，而 os.P_NOWAIT 表示调用进程继续与新进程同时执行。当 *mode* 是 os.P_WAIT 时，函数返回 *P* 的终止码 *c*：0 表示成功终止，*c* < 0 表示 *P* 被 *信号* 杀死，*c* > 0 表示正常但终止失败。当 *mode* 是 os.P_NOWAIT 时，函数返回 *P* 的进程 ID（或在 Windows 上是 *P* 的进程句柄）。没有跨平台的方法来使用 *P* 的 ID 或句柄；Unix-like 平台上的平台特定方法包括 os.waitpid，而在 Windows 上则包括第三方扩展包 [pywin32](https://oreil.ly/dsHxn)。

例如，假设您希望交互式程序给用户一个机会编辑一个即将读取和使用的文本文件。您必须事先确定用户喜欢的文本编辑器的完整路径，例如在 Windows 上为 *c:\\windows\\notepad.exe* 或在类 Unix 平台上为 */usr/bin/vim*。假设这个路径字符串绑定到变量 editor，并且您要让用户编辑的文本文件的路径绑定到 textfile：

```py
`import` os
os.spawnv(os.P_WAIT, editor, (editor, textfile))
```

参数 *args* 的第一项作为“调用程序的名称”传递给被生成的程序。大多数程序不关心这一点，因此通常可以放置任何字符串。以防编辑程序确实查看这个特殊的第一个参数（例如某些版本的 Vim），最简单和最有效的方法是将与 os.spawnv 的第二个参数相同的字符串 editor 传递给它。

| 系统 | 系统(*cmd*) 在新进程中运行字符串命令 *cmd*，当新进程成功终止时返回 0。当新进程终止失败时，系统返回一个非零整数错误代码（具体的错误代码依赖于你运行的命令：这方面没有广泛接受的标准）。 |
| --- | --- |

# mmap 模块

mmap 模块提供了内存映射文件对象。mmap 对象的行为类似于字节串，因此通常可以在需要字节串的地方传递 mmap 对象。然而，它们也有一些区别：

+   mmap 对象不提供字符串对象的方法。

+   mmap 对象是可变的，类似于 bytearray，而字节对象是不可变的。

+   mmap 对象也对应于一个打开的文件，并且在多态性方面表现为 Python 文件对象（如“类似文件对象和多态性”中所述）。

mmap 对象 *m* 可以进行索引或切片操作，产生字节串。由于 *m* 是可变的，你也可以对 *m* 的索引或切片进行赋值。然而，当你对 *m* 的切片赋值时，赋值语句的右侧必须是与你要赋值的切片具有完全相同长度的字节串。因此，许多在列表切片赋值中可用的有用技巧（如“修改列表”中所述）不适用于 mmap 切片赋值。

mmap 模块在 Unix-like 系统和 Windows 上提供了稍有不同的工厂函数：

| mmap | *Windows:* mmap(*filedesc*, *length*, tagname='', access=**None**, offset=**None**) *Unix:* mmap(*filedesc*, *length*, flags=MAP_SHARED, prot=PROT_READ&#124;PROT_WRITE, access=**None**, offset=0)

创建并返回一个 mmap 对象 *m*，它映射到内存中文件描述符 *filedesc* 指示的文件的前 *length* 字节。*filedesc* 必须是一个同时打开读写的文件描述符，除非在类 Unix 平台上，参数 prot 仅请求读或仅请求写。（文件描述符在“文件描述符操作”中有介绍。）要为 Python 文件对象 *f* 获取 mmap 对象 *m*，可以使用 *m*=mmap.mmap(*f*.fileno(), *length*)。*filedesc* 可以为 -1，以映射匿名内存。

在 Windows 上，所有内存映射都是可读写的，并在进程之间共享，因此所有在文件上有内存映射的进程都可以看到其他进程所做的更改。仅在 Windows 上，你可以传递一个字符串 tagname 来为内存映射指定显式的 *标签名*。这个标签名允许你在同一个文件上拥有几个独立的内存映射，但这很少是必需的。仅使用两个参数调用 mmap 的优点是在 Windows 和 Unix-like 平台之间保持代码的可移植性。 |

| mmap *(续)* | 仅在类 Unix 平台上，您可以传递 mmap.MAP_PRIVATE 作为标志以获得一个对您的进程私有并且写时复制的映射。mmap.MAP_SHARED 是默认值，它获取一个与其他进程共享的映射，以便所有映射文件的进程都可以看到一个进程（与 Windows 上相同）所做的更改。您可以将 mmap.PROT_READ 作为 prot 参数传递，以获取仅可读而不可写的映射。传递 mmap.PROT_WRITE 获取仅可写而不可读的映射。默认值，位或 mmap.PROT_READ&#124;mmap.PROT_WRITE，获取一个既可读又可写的映射。您可以传递命名参数 access 而不是标志和 prot（传递 access 和其他两个参数中的一个或两个是错误的）。access 的值可以是 ACCESS_READ（只读），ACCESS_WRITE（写透传，Windows 上的默认值）或 ACCESS_COPY（写时复制）。

您可以传递命名参数 offset 以在文件开始后开始映射；offset 必须是大于等于 0 的整数，是 ALLOCATIONGRANULARITY 的倍数（或者在 Unix 上是 PAGESIZE 的倍数）。

## *mmap* 对象的方法

*mmap* 对象 *m* 提供了详细方法，参见 Table 15-20。

Table 15-20\. *mmap* 实例 *m* 的方法

| close | *m*.close() 关闭 *m* 的文件。 |
| --- | --- |
| find | *m*.find(*sub*, start=0, end=**None**) 返回大于等于 start 的最低 *i*，使得 *sub* == *m*[*i*:*i*+len(*sub*)]（并且当您传递 end 时，*i*+len(*sub*)-1 <= end）。如果没有这样的 *i*，*m*.find 返回 -1。这与 str 的 find 方法的行为相同，该方法在 Table 9-1 中介绍。 |
| flush | *m*.flush([*offset*, *n*]) 确保所有对 *m* 所做的更改都存在于 *m* 的文件中。在调用 *m*.flush 之前，文件是否反映了 *m* 的当前状态是不确定的。您可以传递起始字节偏移量 *offset* 和字节计数 *n* 来将刷新效果的保证限制为 *m* 的一个切片。传递这两个参数，或者两者都不传递：仅传递一个参数调用 *m*.flush 是错误的。 |
| move | *m*.move(*dstoff*, *srcoff*, *n*) 类似于切片赋值 *m*[*dstoff*:*dstoff*+*n*] = *m*[*srcoff*:*srcoff*+*n*]，但可能更快。源切片和目标切片可以重叠。除了可能的重叠外，move 方法不会影响源切片（即，move 方法*复制*字节但不*移动*它们，尽管该方法的名称为“移动”）。 |
| read | *m*.read(*n*) 读取并返回一个字节字符串 *s*，包含从 *m* 的文件指针开始的最多 *n* 个字节，然后将 *m* 的文件指针前进 *s* 的长度。如果 *m* 的文件指针和 *m* 的长度之间的字节数少于 *n*，则返回可用的字节。特别是，如果 *m* 的文件指针在 *m* 的末尾，则返回空字节串 b''。 |
| read_byte | *m*.read_byte() 返回包含*m*的文件指针处的字节的长度为 1 的字节字符串，然后将*m*的文件指针推进 1。*m*.read_byte()类似于*m*.read(1)。但是，如果*m*的文件指针在*m*的末尾，则*m*.read(1)返回空字符串 b''且不推进，而*m*.read_byte()会引发 ValueError 异常。 |
| readline | *m*.readline() 从*m*文件的当前文件指针读取并返回一个字节字符串，直到下一个'\n'（包括'\n'），然后将*m*的文件指针推进到刚刚读取的字节之后。如果*m*的文件指针在*m*的末尾，则 readline 返回空字符串 b''。 |
| resize | *m*.resize(*n*) 改变*m*的长度，使得 len(*m*)变为*n*。不影响*m*的文件大小。*m*的长度和文件大小是独立的。要将*m*的长度设置为文件的大小，请调用*m*.resize(*m*.size())。如果*m*的长度大于文件的大小，则*m*将填充空字节（\x00）。 |
| rfind | rfind(sub, start=0, end=**None**) 返回大于等于 start 的最高*i*，使得*sub* == *m*[*i*:*i*+len(*sub*)]（当你传递 end 时，*i*+len(*sub*)-1 <= end）。如果不存在这样的*i*，*m*.rfind 返回-1。这与字符串对象的 rfind 方法相同，详见 Table 9-1。 |

| seek | *m*.seek(*pos,* how=0) 将*m*的文件指针设置为整数字节偏移量*pos*，相对于由 how 指示的位置：

0 *or* os.SEEK_SET

偏移量相对于*m*的起始位置

1 *or* os.SEEK_CUR

偏移量相对于*m*的当前文件指针

2 *or* os.SEEK_END

偏移量相对于*m*的末尾

试图将*m*的文件指针设置为负偏移量或超出*m*长度的偏移量会引发 ValueError 异常。

| size | *m*.size() 返回*m*文件的长度（以字节为单位），而不是*m*本身的长度。要获取*m*的长度，使用 len(*m*)。 |
| --- | --- |
| tell | *m*.tell() 返回*m*的当前文件指针位置，即*m*文件中的字节偏移量。 |
| write | *m*.write(*b*) 将字节串*b*写入*m*的当前文件指针位置，覆盖已有的字节，然后将*m*的文件指针推进 len(*b*)。如果*m*的文件指针与*m*的长度之间的字节数少于 len(*b*)，write 会引发 ValueError 异常。 |
| write_byte | *m*.write_byte(*byte*) 向映射 *m* 的当前位置写入必须是整数的 *byte*，覆盖原有的字节，然后将 *m* 的文件指针前移 1。 *m*.write_byte(*x*) 与 *m*.write(*x*.to_bytes(1, 'little')) 类似。然而，如果 *m* 的文件指针位于 *m* 的末尾，则 *m*.write_byte(*x*) 静默不做任何操作，而 *m*.write(*x*.to_bytes(1, 'little')) 会引发 ValueError 异常。请注意，这与文件末尾的 read 和 read_byte 之间的关系相反：write 和 read_byte 可能会引发 ValueError，而 read 和 write_byte 则从不会。 |

## 使用 mmap 对象进行 IPC

进程使用 mmap 通信的方式与它们使用文件基本相同：一个进程写入数据，另一个进程稍后读取相同的数据。由于 mmap 对象有一个底层文件，因此可以有一些进程在文件上进行 I/O（如在 “The io Module” 中介绍的），而其他进程在同一个文件上使用 mmap。在便利性和功能性之间选择 mmap 和文件对象的 I/O，性能大致相当。例如，这里是一个简单的程序，反复使用文件 I/O，使文件的内容等于用户交互式输入的最后一行：

```py
fileob = open('xxx','wb')
`while` `True`:
    data = input('Enter some text:')
    fileob.seek(0)
    fileob.write(data.encode())
    fileob.truncate()
    fileob.flush()
```

并且这里有另一个简单的程序，当在与前者相同的目录中运行时，使用 mmap（以及在 Table 13-2 中涵盖的 time.sleep 函数）每秒钟检查文件的变化，并打印出文件的新内容（如果有任何变化的话）：

```py
`import` mmap, os, time
mx = mmap.mmap(os.open('xxx', os.O_RDWR), 1)
last = `None`
`while` `True`:
    mx.resize(mx.size())
    data = mx[:]
    `if` data != last:
        print(data)
        last = data
    time.sleep(1)
```

¹ 我们遇到的最好的关于异步编程的入门工作，尽管现在已经过时（因为 Python 中的异步方法不断改进），是 [*Using Asyncio in Python*](https://www.oreilly.com/library/view/using-asyncio-in/9781492075325/)，作者是 Caleb Hattingh（O’Reilly）。我们建议你也学习 [Brad Solomon 的 Asyncio 演示](https://oreil.ly/HkGpJ) 在 Real Python 上。

² 在线文档包括一个特别有用的 [“编程指南”部分](https://oreil.ly/6EqPh)，列出了使用 multiprocessing 模块时的许多额外实用建议。

³ 竞争条件是一种情况，其中不同事件的相对时间通常是不可预测的，可能会影响计算的结果... 这从来都不是一件好事！

⁴ 就像在 Unix 风格的 shell 命令行中指定 **2>&1** 一样。
