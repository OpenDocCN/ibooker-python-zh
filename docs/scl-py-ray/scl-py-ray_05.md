# 第四章：远程演员

在前一章中，您了解了 Ray 远程函数，这对于并行执行无状态函数非常有用。但是如果您需要在调用之间维护状态怎么办？这些情况的例子包括从简单计数器到训练中的神经网络到模拟环境。

在这些情况下，维护状态的一种选项是将状态与结果一起返回并传递给下一次调用。尽管从技术上讲这是可行的，但由于需要传递的数据量较大（特别是在状态大小开始增长时），这并不是最佳解决方案。Ray 使用演员来管理状态，我们将在本章中介绍。

###### 注意

就像 Ray 的远程函数一样，所有 Ray 演员都是远程演员，即使在同一台机器上运行时也是如此。

简而言之，*演员*是一个具有地址（句柄）的计算机进程。这意味着演员也可以将东西存储在内存中，私有于演员进程。在深入研究实现和扩展 Ray 演员的详细信息之前，让我们先看看它们背后的概念。演员来自演员模型设计模式。理解演员模型对于有效管理状态和并发至关重要。

# 理解演员模型

[演员模型](https://oreil.ly/aTwGY)由 Carl Hewitt 在 1973 年引入，用于处理并发计算。这个概念模型的核心是演员，这是并发计算的通用原语，具有自己的状态。

一个演员的简单工作是：

+   存储数据

+   接收其他演员的消息

+   [消息传递给其他演员](https://oreil.ly/aTwGY)

+   创建额外的子演员

演员存储的数据是私有的，并且在外部看不到；只有演员本身才能访问和修改它。改变演员的状态需要向将修改状态的演员发送消息。（与面向对象编程中使用方法调用相比较。）

为了确保演员的状态一致性，演员一次只处理一个请求。给定演员的所有演员方法调用都是全局串行化的。为了提高吞吐量，人们经常创建演员池（假设他们可以分片或复制演员的状态）。

演员模型非常适合许多分布式系统场景。以下是演员模型可以带来优势的一些典型用例：

+   你需要处理一个大型分布式状态，这在调用之间很难同步。

+   您希望使用不需要来自外部组件显著交互的单线程对象工作。

在这两种情况下，您将在一个演员内部实现工作的独立部分。您可以将每个独立状态的片段放入其自己的演员中，然后通过演员进行状态的任何更改。大多数演员系统实现通过仅使用单线程演员来避免并发问题。

现在您了解了演员模型的一般原则，让我们更详细地看看 Ray 的远程演员。

# 创建一个基本的 Ray 远程演员

Ray 将远程演员实现为有状态的工作者。创建新的远程演员时，Ray 会创建一个新的工作者，并在该工作者上安排演员的方法。

演员的一个常见示例是银行账户。让我们看看如何使用 Ray 远程演员来实现一个账户。通过使用 `@ray.remote` 装饰器简单地装饰一个 Python 类即可创建一个 Ray 远程演员（示例 4-1）。

##### 示例 4-1\. [实现一个 Ray 远程演员](https://oreil.ly/Dpskz)

```py
@ray.remote
class Account:
    def __init__(self, balance: float, minimal_balance: float):
        self.minimal = minimal_balance
        if balance < minimal_balance:
            raise Exception("Starting balance is less than minimal balance")
        self.balance = balance

    def balance(self) -> float:
        return self.balance

    def deposit(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot deposit negative amount")
        self.balance = self.balance + amount
        return self.balance

    def withdraw(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot withdraw negative amount")
        balance = self.balance - amount
        if balance < self.minimal:
            raise Exception("Withdrawal is not supported by current balance")
        self.balance = balance
        return balance
```

`Account` 演员类本身非常简单，只有四种方法：

构造函数

基于起始和最低余额创建一个账户。它还确保当前余额大于最小值，并在否则情况下抛出异常。

`balance`

返回账户的当前余额。因为演员的状态对演员是私有的，因此只能通过演员的方法访问它。

`deposit`

存款到账户并返回新的余额。

`withdraw`

从账户中提取一定金额并返回新的余额。它还确保剩余余额大于预定义的最低余额，否则会抛出异常。

现在您已经定义了类，您需要使用 `.remote` 来创建此演员的实例（示例 4-2）。

##### 示例 4-2\. [创建您的 Ray 远程演员实例](https://oreil.ly/Dpskz)

```py
account_actor = Account.remote(balance = 100.,minimal_balance=20.)
```

在这里，`account_actor` 表示一个演员句柄。这些句柄在演员生命周期中起着重要作用。在 Python 中，当初始演员句柄超出范围时，演员进程会自动终止（请注意，在这种情况下，演员的状态会丢失）。

###### 提示

您可以从同一类创建多个不同的演员。每个演员将有自己独立的状态。

与 `ObjectRef` 一样，您可以将演员句柄作为参数传递给另一个演员或 Ray 远程函数或 Python 代码。

请注意，示例 4-1 使用 `@ray.remote` 注解将普通的 Python 类定义为 Ray 远程演员。或者，您可以使用 示例 4-3 将 Python 类转换为远程演员，而不是使用注解。

##### 示例 4-3\. [创建一个 Ray 远程演员实例而不使用装饰器](https://oreil.ly/Dpskz)

```py
Account = ray.remote(Account)
account_actor = Account.remote(balance = 100.,minimal_balance=20.)
```

一旦您放置了一个远程演员，您可以通过使用 示例 4-4 调用它。

##### 示例 4-4\. [调用远程演员](https://oreil.ly/Dpskz)

```py
print(f"Current balance {ray.get(account_actor.balance.remote())}")
print(f"New balance {ray.get(account_actor.withdraw.remote(40.))}")
print(f"New balance {ray.get(account_actor.deposit.remote(30.))}")
```

###### 提示

处理异常很重要，在示例中，存款和取款方法的代码都可能出现异常。为了处理这些异常，您应该使用 `try`/`except` 语句扩展 示例 4-4：

```py
try:
  result = ray.get(account_actor.withdraw.remote(-40.))
except Exception as e:
  print(f"Oops! \{e} occurred.")
```

这确保代码将拦截演员代码引发的所有异常并执行所有必要的操作。

您还可以通过使用 示例 4-5 创建具有命名的演员。

##### 示例 4-5\. [创建一个命名演员](https://oreil.ly/Dpskz)

```py
account_actor = Account.options(name='Account')\
    .remote(balance = 100.,minimal_balance=20.)
```

一旦 actor 有了名称，你就可以在代码的任何地方使用它来获取 actor 的句柄：

```py
ray.get_actor('Account')
```

如前所述，默认 actor 的生命周期与 actor 的句柄处于作用域相关联。

actor 的生命周期可以与其句柄处于作用域无关联，允许 actor 在驱动程序进程退出后仍然存在。你可以通过指定生命周期参数为 `detached` 来创建一个脱离的 actor (示例 4-6)。

##### 示例 4-6\. [创建一个脱离的 actor](https://oreil.ly/Dpskz)

```py
account_actor = Account.options(name='Account', lifetime='detached')\
    .remote(balance = 100.,minimal_balance=20.)
```

理论上，你可以使一个 actor 脱离而不指定其名称，但由于 `ray.get_actor` 操作是按名称进行的，脱离的 actors 最好带有名称。你应该给你的脱离 actors 命名，这样你就可以在 actor 的句柄超出作用域后访问它们。脱离的 actor 本身可以拥有任何其他任务和对象。

另外，你可以在一个 actor 内部手动删除 actors，使用 `ray.actor.exit_actor`，或者通过一个 actor 的句柄 `ray.kill(account_actor)`。如果你知道不再需要特定的 actors 并且想要回收资源，这将会很有用。

正如这里所示，创建一个基本的 Ray actor 并管理其生命周期是相当容易的，但是如果运行 actor 的 Ray 节点由于某种原因宕机会发生什么呢？¹ `@ray.remote` 注解允许你指定两个 [参数](https://oreil.ly/VAHBm) 来控制这种情况下的行为：

`max_restarts`

指定 actor 异常死亡时应重新启动的最大次数。最小有效值为 `0`（默认），表示 actor 不需要重新启动。值 `-1` 表示 actor 应该无限重新启动。

`max_task_retries`

指定 actor 任务因系统错误而失败时重试任务的次数。如果设置为 `-1`，系统将重试失败的任务，直到任务成功，或者 actor 达到其 `max_restarts` 限制为止。如果设置为 `n > 0`，系统将重试失败的任务多达 *n* 次，之后任务将在 [`ray.get`](https://oreil.ly/li5RX) 上抛出 `RayActorError` 异常。

正如在下一章和 [Ray 容错文档](https://oreil.ly/S64hX) 中进一步解释的那样，当一个 actor 被重新启动时，Ray 将通过重新运行其构造函数来重新创建其状态。因此，如果在 actor 执行过程中更改了状态，它将丢失。要保留这样的状态，actor 必须实现其自定义持久性。

在我们的示例案例中，由于我们没有使用 actor 持久性，actor 的状态在失败时会丢失。这对于某些用例可能是可以接受的，但对于其他用例来说是不可接受的—​参见 [Ray 设计模式文档](https://oreil.ly/tezP2)。在下一节中，你将学习如何以编程方式实现自定义 actor 持久性。

# 实现 Actor 的持久性

在这个实现中，状态作为一个整体保存，如果状态的大小相对较小且状态更改相对较少，则这种方式足够好。此外，为了保持我们的示例简单，我们使用本地磁盘持久化。实际情况下，对于分布式 Ray 案例，您应考虑使用网络文件系统（NFS）、Amazon 简单存储服务（S3）或数据库来实现对演员数据的访问，使其能够从 Ray 集群中的任何节点访问。

在示例 4-7 中展示了一个持久化的 `Account` 演员。²

##### 示例 4-7\. [定义一个持久化演员，使用文件系统持久化](https://oreil.ly/qHSfR)

```py
@ray.remote
class Account:
    def __init__(self, balance: float, minimal_balance: float, account_key: str,
        basedir: str = '.'):
        self.basedir = basedir
        self.key = account_key
        if not self.restorestate():
            if balance < minimal_balance:
                raise Exception("Starting balance is less than minimal balance")
            self.balance = balance
            self.minimal = minimal_balance
            self.storestate()

    def balance(self) -> float:
        return self.balance

    def deposit(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot deposit negative amount")
        self.balance = self.balance + amount
        self.storestate()
        return self.balance

    def withdraw(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot withdraw negative amount")
        balance = self.balance - amount
        if balance < self.minimal:
            raise Exception("Withdrawal is not supported by current balance")
        self.balance = balance
        self.storestate()
        return balance

    def restorestate(self) -> bool:
        if exists(self.basedir + '/' + self.key):
            with open(self.basedir + '/' + self.key, "rb") as f:
                bytes = f.read()
            state = ray.cloudpickle.loads(bytes)
            self.balance = state['balance']
            self.minimal = state['minimal']
            return True
        else:
            return False

    def storestate(self):
        bytes = ray.cloudpickle.dumps(
            {'balance' : self.balance, 'minimal' : self.minimal})
        with open(self.basedir + '/' + self.key, "wb") as f:
            f.write(bytes)
```

如果我们将此实现与示例 4-1 中的原始实现进行比较，我们将注意到几个重要的变化：

+   这里的构造函数有两个额外的参数：`account_key` 和 `basedir`。账户密钥是账户的唯一标识符，也用作持久化文件的名称。`basedir` 参数指示用于存储持久化文件的基本目录。当调用构造函数时，我们首先检查是否保存了此账户的持久状态，如果有，则忽略传入的余额和最低余额，并从持久状态中恢复它们。

+   在类中添加了两个额外的方法：`store_state` 和 `restore_state`。`store_state` 方法将演员状态存储到文件中。状态信息表示为一个字典，字典的键是状态元素的名称，值是状态元素的值。我们使用 Ray 的云串行化实现将此字典转换为字节字符串，然后将该字节字符串写入由账户密钥和基础目录定义的文件中。(第 5 章详细讨论了云串行化。)`restore_state` 方法从由账户密钥和基础目录定义的文件中恢复状态。该方法从文件中读取一个二进制字符串，并使用 Ray 的云串行化实现将其转换为字典。然后，它使用字典的内容填充状态。

+   最后，`deposit` 和 `withdraw` 方法都会更改状态，使用 `store_state` 方法更新持久化。

在示例 4-7 中展示的实现工作正常，但我们的账户演员实现现在包含太多与持久化相关的代码，并且与文件持久化紧密耦合。一个更好的解决方案是将持久化特定的代码分离到一个单独的类中。

我们首先创建一个抽象类，定义了必须由任何持久化类实现的方法（示例 4-8）。

##### 示例 4-8\. [定义一个基础持久化类](https://oreil.ly/sI7Me)

```py
class BasePersitence:
    def exists(self, key:str) -> bool:
        pass
    def save(self, key: str, data: dict):
        pass
    def restore(self, key:str) -> dict:
        pass
```

此类定义了必须由具体持久化实现实现的所有方法。有了这个基础，可以定义一个实现基础持久化的文件持久化类，如示例 4-9 所示。

##### 示例 4-9\. [定义文件持久化类](https://oreil.ly/sI7Me)

```py
class FilePersistence(BasePersitence):
    def __init__(self, basedir: str = '.'):
        self.basedir = basedir

    def exists(self, key:str) -> bool:
        return exists(self.basedir + '/' + key)

    def save(self, key: str, data: dict):
        bytes = ray.cloudpickle.dumps(data)
        with open(self.basedir + '/' + key, "wb") as f:
            f.write(bytes)

    def restore(self, key:str) -> dict:
        if not self.exists(key):
            return None
        else:
            with open(self.basedir + '/' + key, "rb") as f:
                bytes = f.read()
            return ray.cloudpickle.loads(bytes)
```

此实现从我们最初的实现中分离出大部分与持久化相关的代码，该实现位于示例 4-7 中。现在可以简化和概括账户实现；参见示例 4-10。

##### 示例 4-10\. [实现具有可插拔持久性的持久化 actor](https://oreil.ly/sI7Me)

```py
@ray.remote
class Account:
    def __init__(self, balance: float, minimal_balance: float, account_key: str,
                 persistence: BasePersitence):
        self.persistence = persistence
        self.key = account_key
        if not self.restorestate():
            if balance < minimal_balance:
                raise Exception("Starting balance is less than minimal balance")
            self.balance = balance
            self.minimal = minimal_balance
            self.storestate()

    def balance(self) -> float:
        return self.balance

    def deposit(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot deposit negative amount")
        self.balance = self.balance + amount
        self.storestate()
        return self.balance

    def withdraw(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot withdraw negative amount")
        balance = self.balance - amount
        if balance < self.minimal:
            raise Exception("Withdrawal is not supported by current balance")
        self.balance = balance
        self.storestate()
        return balance

    def restorestate(self) -> bool:
        state = self.persistence.restore(self.key)
        if state != None:
            self.balance = state['balance']
            self.minimal = state['minimal']
            return True
        else:
            return False

    def storestate(self):
        self.persistence.save(self.key,
                    {'balance' : self.balance, 'minimal' : self.minimal})
```

从我们最初的持久化 actor 实现（示例 4-7）中，只展示了代码的变化。请注意，构造函数现在使用`Base​Per⁠sis⁠tence`类，这使得可以轻松地更改持久化实现而不更改 actor 的代码。另外，`restore_state`和`savestate`方法被概括为将所有与持久化相关的代码移到持久化类中。

此实现足够灵活，支持不同的持久化实现，但是，如果持久化实现需要与持久化源（例如，数据库连接）建立永久连接，同时维护太多连接可能导致不可扩展。在这种情况下，我们可以将持久化实现为[附加 actor](https://oreil.ly/gz7wp)。但这需要扩展此 actor。让我们看看 Ray 为扩展 actor 提供的选项。

# 缩放 Ray 远程 Actor

本章前面描述的原始 actor 模型通常假定 actor 是轻量级的（例如，包含单个状态片段）并且不需要扩展或并行化。在 Ray 和类似系统（包括 Akka）中，actor 通常用于更粗粒度的实现，并且可能需要扩展。³

与 Ray 远程函数一样，您可以使用池横向（跨进程/机器）或纵向（使用更多资源）扩展 actor。"资源/纵向扩展"介绍了如何请求更多资源，但现在，让我们专注于横向扩展。

您可以使用 Ray 的 actor 池为横向扩展添加更多进程，该池由`ray.util`模块提供。该类类似于多进程池，并允许您在一组固定的 actor 上安排任务。

actor 池有效地将一组固定的 actor 作为单个实体使用，并管理池中的下一个请求由哪个 actor 处理。请注意，池中的 actor 仍然是各自独立的 actor，并且它们的状态没有合并。因此，此缩放选项仅在 actor 的状态在构造函数中创建且在 actor 执行期间不更改时有效。

让我们看看如何通过在 [Example 4-11](https://oreil.ly/UsjXG) 中添加一个 [actor’s pool](https://oreil.ly/hSXsd) 来提高账户类的可扩展性。

##### 示例 4-11\. [使用 actor 的池实现持久性](https://oreil.ly/UsjXG)

```py
pool = ActorPool([
    FilePersistence.remote(), FilePersistence.remote(), FilePersistence.remote()])

@ray.remote
class Account:
    def __init__(self, balance: float, minimal_balance: float,
            account_key: str, persistence: ActorPool):
        self.persistence = persistence
        self.key = account_key
        if not self.restorestate():
            if balance < minimal_balance:
                raise Exception("Starting balance is less than minimal balance")
            self.balance = balance
            self.minimal = minimal_balance
            self.storestate()

    def balance(self) -> float:
        return self.balance

    def deposit(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot deposit negative amount")
        self.balance = self.balance + amount
        self.storestate()
        return self.balance

    def withdraw(self, amount: float) -> float:
        if amount < 0:
            raise Exception("Cannot withdraw negative amount")
        balance = self.balance - amount
        if balance < self.minimal:
            raise Exception("Withdrawal is not supported by current balance")
        self.balance = balance
        self.storestate()
        return balance

    def restorestate(self) -> bool:
        while(self.persistence.has_next()):
            self.persistence.get_next()
        self.persistence.submit(lambda a, v: a.restore.remote(v), self.key)
        state = self.persistence.get_next()
        if state != None:
            print(f'Restoring state {state}')
            self.balance = state['balance']
            self.minimal = state['minimal']
            return True
        else:
            return False

    def storestate(self):
        self.persistence.submit(
            lambda a, v: a.save.remote(v),
            (self.key,
             {'balance' : self.balance, 'minimal' : self.minimal}))

account_actor = Account.options(name='Account').remote(
    balance=100.,minimal_balance=20.,
    account_key='1234567', persistence=pool)
```

这里仅展示了我们原始实现的代码更改。代码从创建三个相同文件持久性 actor 的池开始，然后将此池传递给账户实现。

基于池的执行语法是一个接受两个参数的 lambda 函数：一个是 actor 引用，一个是要提交给函数的值。这里的限制是值是一个单一对象。对于具有多个参数的函数的解决方案之一是使用可以包含任意数量组件的元组。函数本身被定义为所需 actor 方法上的远程函数。

在池中执行是异步的（它将请求路由到一个远程的 actor）。这允许更快地执行 `store_state` 方法，它不需要来自数据存储的结果。在这里，实现不等待结果的状态存储完成；它只是开始执行。另一方面，`restore_state` 方法需要池调用的结果来进行后续操作。池的实现内部管理等待执行结果就绪的过程，并通过 `get_next` 函数公开这一功能（请注意，这是一个阻塞调用）。池的实现管理一个执行结果队列（按照请求的顺序）。因此，每当我们需要从池中获取结果时，我们必须首先清空池结果队列，以确保获取正确的结果。

除了由 actor 的池提供的基于多处理的扩展之外，Ray 还通过并发支持 actor 的执行扩展。Ray 在 actor 内部提供了两种并发类型：线程和异步执行。

在使用 actors 内部的并发时，请记住 Python 的 [全局解释器锁（GIL）](https://oreil.ly/l7Ytt) 一次只允许运行一个 Python 代码线程。纯 Python 将不提供真正的并行性。另一方面，如果调用 NumPy、Cython、TensorFlow 或 PyTorch 代码，这些库在调用 C/C++函数时会释放 GIL。通过重叠等待 I/O 时间或在本地库中工作，线程和异步 actor 执行都可以实现一定的并行性。

[异步 io 库](https://oreil.ly/PXo8G)可以被看作是协同多任务：你的代码或库需要明确地信号表示正在等待结果，Python 可以继续执行另一个任务，通过明确切换执行上下文。asyncio 通过在事件循环中运行单一进程，并在任务 yield/await 时改变执行哪个任务来工作。与多线程执行相比，asyncio 的开销较低，且更容易理解。Ray actors 与 asyncio 集成，允许你编写异步 actor 方法，但不支持远程函数。

当你的代码大部分时间阻塞但不通过调用 `await` 放弃控制时，应使用线程执行。线程由操作系统决定何时运行哪个线程。使用线程执行可能涉及较少的代码更改，因为不需要明确指示代码何时放弃控制。这也可能使线程执行更难理解。

当使用线程和 asyncio 访问或修改对象时，需要小心并有选择地使用锁。在两种方法中，对象共享同一内存。通过使用锁，确保只有一个线程或任务可以访问特定内存。锁有一些开销（随着更多进程或线程等待锁而增加）。因此，actor 的并发性大多适用于在构造函数中填充状态并且不会更改状态的用例。

要创建使用 asyncio 的 actor，需要至少定义一个 async 方法。在这种情况下，Ray 将为执行 actor 的方法创建一个 asyncio 事件循环。从调用者的角度来看，提交任务到这些 actors 与提交任务到常规 actor 相同。唯一的区别是，当任务在 actor 上运行时，它被发布到后台线程或线程池中运行的 asyncio 事件循环，而不是直接在主线程上运行。（请注意，不允许在异步 actor 方法中使用阻塞的 `ray.get` 或 `ray.wait` 调用，因为它们会阻塞事件循环的执行。）

示例 4-12 展示了一个简单的异步 actor 示例。

##### 示例 4-12\. [创建一个简单的异步 actor](https://oreil.ly/q3WFs)

```py
@ray.remote
class AsyncActor:
    async def computation(self, num):
        print(f'Actor waiting for {num} sec')
        for x in range(num):
            await asyncio.sleep(1)
            print(f'Actor slept for {x+1} sec')
        return num
```

因为方法 `computation` 被定义为 `async`，Ray 将创建一个异步 actor。请注意，与普通的 `async` 方法不同，后者需要使用 `await` 调用，使用 Ray 异步 actors 不需要任何特殊的调用语义。此外，Ray 允许在 actor 创建时指定异步 actor 执行的最大并发性：

```py
actor = AsyncActor.options(max_concurrency=5).remote()
```

要创建一个线程 actor，需要在 actor 创建时指定 `max_concurrency` （示例 4-13）。

##### 示例 4-13\. [创建一个简单的线程 actor](https://oreil.ly/EjTM4)

```py
@ray.remote
class ThreadedActor:
  def computation(self, num):
    print(f'Actor waiting for \{num} sec')
    for x in range(num):
      sleep(1)
      print(f'Actor slept for \{x+1} sec')
    return num

actor = ThreadedActor.options(max_concurrency=3).remote()
```

###### 提示

由于异步和线程化 actor 都使用 `max_concurrency`，所以创建的 actor 类型可能会有些混淆。需要记住的是，如果使用了 `max_concurrency`，actor 可以是异步的或者线程化的。如果 actor 的至少一个方法是异步的，那么 actor 就是异步的；否则，它就是线程化的。

那么，我们应该使用哪种扩展方法来实现我们的应用程序？[“Python 中的多进程 vs. 多线程 vs. AsyncIO”](https://oreil.ly/UF26H) 由 Lei Mao 提供了各种方法特性的良好总结（参见 表 4-1）。

表 4-1\. 比较 actor 的扩展方法

| 扩展方法 | 特性 | 使用条件 |
| --- | --- | --- |
| Actor 池 | 多进程，高 CPU 利用率 | CPU 绑定 |
| 异步 actor | 单进程，单线程，协同多任务处理，任务协同决定切换 | 慢 I/O 绑定 |
| 线程化 actor | 单进程，多线程，抢占式多任务处理，由操作系统决定任务切换 |

快速 I/O 绑定和非异步库您无法控制

|

# Ray 远程 actor 最佳实践

因为 Ray 远程 actor 实际上就是远程函数，因此前一章描述的所有 Ray 远程最佳实践同样适用。此外，Ray 还有一些特定于 actor 的最佳实践。

正如之前提到的，Ray 支持 actor 的容错。特别是对于 actor，您可以指定 `max_restarts` 来自动启用 Ray actor 的重启。当您的 actor 或托管该 actor 的节点崩溃时，actor 将自动重建。然而，这并不提供方法来恢复 actor 中的应用程序级状态。考虑到这一点，可以采用本章描述的 actor 持久化方法来确保执行级状态的恢复。

如果您的应用程序有全局变量需要更改，请不要在远程函数中更改它们。相反，使用 actor 封装它们，并通过 actor 的方法访问它们。这是因为远程函数在不同的进程中运行，并且不共享相同的地址空间。因此，这些更改不会在 Ray 驱动程序和远程函数之间反映出来。

一个常见的应用场景是为不同的数据集多次执行同一个远程函数。直接使用远程函数可能会因为创建新进程而导致延迟。这种方法也可能会因为大量进程而使 Ray 集群不堪重负。一个更为可控的选项是使用 actor 池。在这种情况下，池提供了一组受控的工作进程，这些工作进程可立即可用（无需进程创建延迟）进行执行。由于池在维护其请求队列，因此这种选项的编程模型与启动独立远程函数完全相同，但提供了更好的控制执行环境。

# 结论

在本章中，你学习了如何使用 Ray 远程 Actor 在 Ray 中实现有状态执行。你学习了 Actor 模型以及如何实现 Ray 远程 Actor。请注意，Ray 在内部大量依赖 Actor，例如用于[多节点同步](https://oreil.ly/vYdTi)，流处理（参见第六章）以及微服务实现（参见第七章）。它还被广泛用于机器学习实现，例如用于实现[参数服务器](https://oreil.ly/q33OW)的 Actor。

你还学习了如何通过实现 Actor 的持久化来提高 Actor 的可靠性，并看到了一个简单的持久化实现示例。

最后，你了解了 Ray 提供的用于扩展 Actor、它们的实现以及权衡的选项。

在下一章中，我们将讨论更多 Ray 的设计细节。

¹ Python 异常并不被视为系统错误，不会触发重启。相反，异常会作为调用的结果保存，并且 Actor 将继续正常运行。

² 在这个实现中，我们使用文件系统持久化，但你也可以使用其他类型的持久化，比如 S3 或数据库。

³ *粗粒度* Actor 是一个单一 Actor，可能包含多个状态片段。相比之下，细粒度方法中，每个状态片段都将被表示为一个单独的 Actor。这类似于[粗粒度锁定](https://oreil.ly/WwBrd)的概念。
