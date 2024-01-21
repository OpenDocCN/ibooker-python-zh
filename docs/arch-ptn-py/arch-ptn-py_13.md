# 第十章：命令和命令处理程序

> 原文：[10: Commands and Command Handler](https://www.cosmicpython.com/book/chapter_10_commands.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

在上一章中，我们谈到使用事件作为表示系统输入的一种方式，并将我们的应用程序转变为一个消息处理机器。

为了实现这一点，我们将所有的用例函数转换为事件处理程序。当 API 接收到一个创建新批次的 POST 请求时，它构建一个新的`BatchCreated`事件，并将其处理为内部事件。这可能感觉反直觉。毕竟，批次*还没有*被创建；这就是我们调用 API 的原因。我们将通过引入命令并展示它们如何通过相同的消息总线处理，但规则略有不同来解决这个概念上的问题。

###### 提示

本章的代码在 GitHub 的 chapter_10_commands 分支中[`oreil.ly/U_VGa`](https://oreil.ly/U_VGa)：

```py
git clone https://github.com/cosmicpython/code.git
cd code
git checkout chapter_10_commands
# or to code along, checkout the previous chapter:
git checkout chapter_09_all_messagebus
```

# 命令和事件

与事件一样，*命令*是一种消息类型——由系统的一部分发送给另一部分的指令。我们通常用愚蠢的数据结构表示命令，并且可以以与事件相同的方式处理它们。

然而，命令和事件之间的差异是重要的。

命令由一个参与者发送给另一个特定的参与者，并期望作为结果发生特定的事情。当我们向 API 处理程序发布表单时，我们正在发送一个命令。我们用祈使句动词短语命名命令，比如“分配库存”或“延迟发货”。

命令捕获*意图*。它们表达了我们希望系统执行某些操作的愿望。因此，当它们失败时，发送者需要接收错误信息。

*事件*由一个参与者广播给所有感兴趣的监听器。当我们发布`BatchQuantityChanged`时，我们不知道谁会接收到它。我们用过去时动词短语命名事件，比如“订单分配给库存”或“发货延迟”。

我们经常使用事件来传播关于成功命令的知识。

事件捕获*过去发生的事情*的事实。由于我们不知道谁在处理事件，发送者不应关心接收者成功与否。表 10-1 总结了差异。

表 10-1。事件与命令的差异

|  | 事件 | 命令 |
| --- | --- | --- |
| 命名 | 过去式 | 祈使句 |
| 错误处理 | 独立失败 | 失败时有噪音 |
| 发送至 | 所有监听器 | 一个接收者 |

我们的系统现在有什么样的命令？

*提取一些命令（`src/allocation/domain/commands.py`）*

```py
class Command:
    pass


@dataclass
class Allocate(Command):  #(1)
    orderid: str
    sku: str
    qty: int


@dataclass
class CreateBatch(Command):  #(2)
    ref: str
    sku: str
    qty: int
    eta: Optional[date] = None


@dataclass
class ChangeBatchQuantity(Command):  #(3)
    ref: str
    qty: int
```

①

`commands.Allocate`将替换`events.AllocationRequired`。

②

`commands.CreateBatch`将替换`events.BatchCreated`。

③

`commands.ChangeBatchQuantity`将替换`events.BatchQuantityChanged`。

# 异常处理的差异

只是改变名称和动词是很好的，但这不会改变我们系统的行为。我们希望以类似的方式处理事件和命令，但不完全相同。让我们看看我们的消息总线如何改变：

*不同的调度事件和命令（`src/allocation/service_layer/messagebus.py`）*

```py
Message = Union[commands.Command, events.Event]


def handle(  #(1)
    message: Message,
    uow: unit_of_work.AbstractUnitOfWork,
):
    results = []
    queue = [message]
    while queue:
        message = queue.pop(0)
        if isinstance(message, events.Event):
            handle_event(message, queue, uow)  #(2)
        elif isinstance(message, commands.Command):
            cmd_result = handle_command(message, queue, uow)  #(2)
            results.append(cmd_result)
        else:
            raise Exception(f"{message} was not an Event or Command")
    return results
```

①

它仍然有一个主要的`handle()`入口，接受一个`message`，它可以是一个命令或一个事件。

②

我们将事件和命令分派给两个不同的辅助函数，如下所示。

这是我们处理事件的方式：

*事件不能中断流程（`src/allocation/service_layer/messagebus.py`）*

```py
def handle_event(
    event: events.Event,
    queue: List[Message],
    uow: unit_of_work.AbstractUnitOfWork,
):
    for handler in EVENT_HANDLERS[type(event)]:  #(1)
        try:
            logger.debug("handling event %s with handler %s", event, handler)
            handler(event, uow=uow)
            queue.extend(uow.collect_new_events())
        except Exception:
            logger.exception("Exception handling event %s", event)
            continue  #(2)

```

①

事件发送到一个分发器，可以将每个事件委派给多个处理程序。

②

它捕获并记录错误，但不允许它们中断消息处理。

这是我们处理命令的方式：

*命令重新引发异常（`src/allocation/service_layer/messagebus.py`）*

```py
def handle_command(
    command: commands.Command,
    queue: List[Message],
    uow: unit_of_work.AbstractUnitOfWork,
):
    logger.debug("handling command %s", command)
    try:
        handler = COMMAND_HANDLERS[type(command)]  #(1)
        result = handler(command, uow=uow)
        queue.extend(uow.collect_new_events())
        return result  #(3)
    except Exception:
        logger.exception("Exception handling command %s", command)
        raise  #(2)
```

①

命令调度程序期望每个命令只有一个处理程序。

②

如果引发任何错误，它们会快速失败并上升。

③

`return result`只是临时的；如[“临时的丑陋的黑客：消息总线必须返回结果”](ch09.xhtml#temporary_ugly_hack)中所述，这是一个临时的黑客，允许消息总线返回 API 使用的批次引用。我们将在第十二章中修复这个问题。

我们还将单个`HANDLERS`字典更改为命令和事件的不同字典。根据我们的约定，命令只能有一个处理程序：

*新处理程序字典（`src/allocation/service_layer/messagebus.py`）*

```py
EVENT_HANDLERS = {
    events.OutOfStock: [handlers.send_out_of_stock_notification],
}  # type: Dict[Type[events.Event], List[Callable]]

COMMAND_HANDLERS = {
    commands.Allocate: handlers.allocate,
    commands.CreateBatch: handlers.add_batch,
    commands.ChangeBatchQuantity: handlers.change_batch_quantity,
}  # type: Dict[Type[commands.Command], Callable]
```

# 讨论：事件、命令和错误处理

许多开发人员在这一点感到不舒服，并问：“当事件处理失败时会发生什么？我应该如何确保系统处于一致状态？”如果我们在`messagebus.handle`处理一半的事件之前由于内存不足错误而终止进程，我们如何减轻因丢失消息而引起的问题？

让我们从最坏的情况开始：我们未能处理事件，系统处于不一致状态。会导致这种情况的是什么样的错误？在我们的系统中，当只完成了一半的操作时，我们经常会陷入不一致状态。

例如，我们可以为客户的订单分配三个单位的`DESIRABLE_BEANBAG`，但在某种程度上未能减少剩余库存量。这将导致不一致的状态：三个单位的库存既被分配*又*可用，这取决于你如何看待它。后来，我们可能会将这些相同的沙发床分配给另一个客户，给客户支持带来麻烦。

然而，在我们的分配服务中，我们已经采取了措施来防止发生这种情况。我们已经仔细确定了作为一致性边界的*聚合*，并且我们引入了一个*UoW*来管理对聚合的更新的原子成功或失败。

例如，当我们为订单分配库存时，我们的一致性边界是`Product`聚合。这意味着我们不能意外地过度分配：要么特定订单行分配给产品，要么不分配——没有不一致状态的余地。

根据定义，我们不需要立即使两个聚合保持一致，因此如果我们未能处理事件并仅更新单个聚合，我们的系统仍然可以最终保持一致。我们不应违反系统的任何约束。

有了这个例子，我们可以更好地理解将消息分割为命令和事件的原因。当用户想要让系统执行某些操作时，我们将他们的请求表示为*命令*。该命令应修改单个*聚合*，并且要么完全成功，要么完全失败。我们需要做的任何其他簿记，清理和通知都可以通过*事件*来进行。我们不需要事件处理程序成功才能使命令成功。

让我们看另一个例子（来自不同的、虚构的项目）来看看为什么不行。

假设我们正在构建一个销售昂贵奢侈品的电子商务网站。我们的营销部门希望奖励重复访问的客户。他们在第三次购买后将客户标记为 VIP，并且这将使他们有资格获得优先处理和特别优惠。我们对这个故事的验收标准如下：

```py
Given a customer with two orders in their history,
When the customer places a third order,
Then they should be flagged as a VIP.

When a customer first becomes a VIP
Then we should send them an email to congratulate them
```

使用我们在本书中已经讨论过的技术，我们决定要构建一个新的`History`聚合，记录订单并在满足规则时引发领域事件。我们将按照以下结构编写代码：

*VIP 客户（另一个项目的示例代码）*

```py
class History:  # Aggregate

    def __init__(self, customer_id: int):
        self.orders = set()  # Set[HistoryEntry]
        self.customer_id = customer_id

    def record_order(self, order_id: str, order_amount: int): #(1)
        entry = HistoryEntry(order_id, order_amount)

        if entry in self.orders:
            return

        self.orders.add(entry)

        if len(self.orders) == 3:
            self.events.append(
                CustomerBecameVIP(self.customer_id)
            )


def create_order_from_basket(uow, cmd: CreateOrder): #(2)
    with uow:
        order = Order.from_basket(cmd.customer_id, cmd.basket_items)
        uow.orders.add(order)
        uow.commit()  # raises OrderCreated


def update_customer_history(uow, event: OrderCreated): #(3)
    with uow:
        history = uow.order_history.get(event.customer_id)
        history.record_order(event.order_id, event.order_amount)
        uow.commit()  # raises CustomerBecameVIP


def congratulate_vip_customer(uow, event: CustomerBecameVip): #(4)
    with uow:
        customer = uow.customers.get(event.customer_id)
        email.send(
            customer.email_address,
            f'Congratulations {customer.first_name}!'
        )
```

①

`History`聚合捕获了指示客户何时成为 VIP 的规则。这使我们能够在未来规则变得更加复杂时处理变化。

②

我们的第一个处理程序为客户创建订单，并引发领域事件`OrderCreated`。

③

我们的第二个处理程序更新`History`对象，记录已创建订单。

④

最后，当客户成为 VIP 时，我们会给他们发送一封电子邮件。

使用这段代码，我们可以对事件驱动系统中的错误处理有一些直觉。

在我们当前的实现中，我们在将状态持久化到数据库之后才引发关于聚合的事件。如果我们在持久化之前引发这些事件，并同时提交所有的更改，会怎样呢？这样，我们就可以确保所有的工作都已完成。这样不是更安全吗？

然而，如果电子邮件服务器稍微过载会发生什么呢？如果所有工作都必须同时完成，繁忙的电子邮件服务器可能会阻止我们接受订单的付款。

如果`History`聚合的实现中存在错误，会发生什么？难道因为我们无法识别您为 VIP 而放弃收取您的钱吗？

通过分离这些关注点，我们使得事情可以独立失败，这提高了系统的整体可靠性。这段代码中*必须*完成的部分只有创建订单的命令处理程序。这是客户关心的唯一部分，也是我们的业务利益相关者应该优先考虑的部分。

请注意，我们故意将事务边界与业务流程的开始和结束对齐。代码中使用的名称与我们的业务利益相关者使用的行话相匹配，我们编写的处理程序与我们的自然语言验收标准的步骤相匹配。名称和结构的一致性帮助我们推理系统在变得越来越大和复杂时的情况。

# 同步恢复错误

希望我们已经说服您，事件可以独立于引发它们的命令而失败。那么，当错误不可避免地发生时，我们应该怎么做才能确保我们能够从错误中恢复呢？

我们首先需要知道错误发生的*时间*，通常我们依赖日志来做到这一点。

让我们再次看看我们消息总线中的`handle_event`方法：

*当前处理函数（`src/allocation/service_layer/messagebus.py`）*

```py
def handle_event(
    event: events.Event,
    queue: List[Message],
    uow: unit_of_work.AbstractUnitOfWork
):
    for handler in EVENT_HANDLERS[type(event)]:
        try:
            logger.debug('handling event %s with handler %s', event, handler)
            handler(event, uow=uow)
            queue.extend(uow.collect_new_events())
        except Exception:
            logger.exception('Exception handling event %s', event)
            continue
```

当我们在系统中处理消息时，我们首先要做的是写入日志记录我们即将要做的事情。对于`CustomerBecameVIP`用例，日志可能如下所示：

```py
Handling event CustomerBecameVIP(customer_id=12345)
with handler <function congratulate_vip_customer at 0x10ebc9a60>
```

因为我们选择使用数据类来表示我们的消息类型，我们可以得到一个整洁打印的摘要，其中包含了我们可以复制并粘贴到 Python shell 中以重新创建对象的传入数据。

当发生错误时，我们可以使用记录的数据来在单元测试中重现问题，或者将消息重新播放到系统中。

手动重放对于需要在重新处理事件之前修复错误的情况非常有效，但我们的系统将始终经历一定程度的瞬态故障。这包括网络故障、表死锁以及部署导致的短暂停机等情况。

对于大多数情况，我们可以通过再次尝试来优雅地恢复。正如谚语所说：“如果一开始你没有成功，就用指数增长的等待时间重试操作。”

*带重试的处理（`src/allocation/service_layer/messagebus.py`）*

```py
from tenacity import Retrying, RetryError, stop_after_attempt, wait_exponential #(1)

...

def handle_event(
    event: events.Event,
    queue: List[Message],
    uow: unit_of_work.AbstractUnitOfWork,
):
    for handler in EVENT_HANDLERS[type(event)]:
        try:
            for attempt in Retrying(  #(2)
                stop=stop_after_attempt(3),
                wait=wait_exponential()
            ):

                with attempt:
                    logger.debug("handling event %s with handler %s", event, handler)
                    handler(event, uow=uow)
                    queue.extend(uow.collect_new_events())
        except RetryError as retry_failure:
            logger.error(
                "Failed to handle event %s times, giving up!",
                retry_failure.last_attempt.attempt_number
            )
            continue
```

①

Tenacity 是一个实现重试常见模式的 Python 库。

②

在这里，我们配置我们的消息总线，最多重试三次，在尝试之间等待的时间会指数增长。

重试可能会失败的操作可能是改善软件弹性的最佳方法。再次，工作单元和命令处理程序模式意味着每次尝试都从一致的状态开始，并且不会留下半成品。

###### 警告

在某个时候，无论`tenacity`如何，我们都必须放弃尝试处理消息。构建可靠的分布式消息系统很困难，我们必须略过一些棘手的部分。在[结语](afterword01.xhtml#epilogue_1_how_to_get_there_from_here)中有更多参考资料的指针。

# 总结

在本书中，我们决定在介绍命令的概念之前先介绍事件的概念，但其他指南通常是相反的。通过为系统可以响应的请求命名并为它们提供自己的数据结构，是一件非常基本的事情。有时你会看到人们使用“命令处理程序”模式来描述我们在事件、命令和消息总线中所做的事情。

表 10-2 讨论了在你加入之前应该考虑的一些事情。

表 10-2。分割命令和事件：权衡利弊

| 优点 | 缺点 |
| --- | --- |
| 将命令和事件区分对待有助于我们理解哪些事情必须成功，哪些事情可以稍后整理。 | 命令和事件之间的语义差异可能是微妙的。对于这些差异可能会有很多争论。 |
| `CreateBatch`绝对比`BatchCreated`更清晰。我们明确了用户的意图，而明确比隐含更好，对吧？ | 我们明确地邀请失败。我们知道有时会出现问题，我们选择通过使失败变得更小更隔离来处理这些问题。这可能会使系统更难以理解，并需要更好的监控。 |

在第十一章中，我们将讨论使用事件作为集成模式。
