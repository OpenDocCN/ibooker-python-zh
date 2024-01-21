# 附录 E：验证

> 原文：[Appendix E: Validation](https://www.cosmicpython.com/book/appendix_validation.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

每当我们教授和讨论这些技术时，一个反复出现的问题是“我应该在哪里进行验证？这属于我的业务逻辑在领域模型中，还是属于基础设施问题？”

与任何架构问题一样，答案是：这取决于情况！

最重要的考虑因素是我们希望保持我们的代码良好分离，以便系统的每个部分都很简单。我们不希望用无关的细节来混淆我们的代码。

# 验证到底是什么？

当人们使用*验证*这个词时，他们通常指的是一种过程，通过这种过程测试操作的输入，以确保它们符合某些标准。符合标准的输入被认为是*有效*的，而不符合标准的输入被认为是*无效*的。

如果输入无效，则操作无法继续，但应该以某种错误退出。换句话说，验证是关于创建*前提条件*。我们发现将我们的前提条件分为三个子类型：语法、语义和语用是有用的。

# 验证语法

在语言学中，语言的*语法*是指控制语法句子结构的规则集。例如，在英语中，“Allocate three units of `TASTELESS-LAMP` to order twenty-seven”是语法正确的，而短语“hat hat hat hat hat hat wibble”则不是。我们可以将语法正确的句子描述为*格式良好*。

这如何映射到我们的应用程序？以下是一些语法规则的示例：

+   一个`Allocate`命令必须有一个订单 ID、一个 SKU 和一个数量。

+   数量是一个正整数。

+   SKU 是一个字符串。

这些是关于传入数据的形状和结构的规则。一个没有 SKU 或订单 ID 的`Allocate`命令不是一个有效的消息。这相当于短语“Allocate three to.”

我们倾向于在系统的边缘验证这些规则。我们的经验法则是，消息处理程序应始终只接收格式良好且包含所有必需信息的消息。

一种选择是将验证逻辑放在消息类型本身上：

*消息类上的验证（`src/allocation/commands.py`）*

```py
from schema import And, Schema, Use


@dataclass
class Allocate(Command):

    _schema = Schema({  #(1)
        'orderid': int,
         sku: str,
         qty: And(Use(int), lambda n: n > 0)
     }, ignore_extra_keys=True)

    orderid: str
    sku: str
    qty: int

    @classmethod
    def from_json(cls, data):  #(2)
        data = json.loads(data)
        return cls(**_schema.validate(data))
```

①

[`schema`库](https://pypi.org/project/schema)让我们以一种好的声明方式描述消息的结构和验证。

②

`from_json`方法将字符串读取为 JSON，并将其转换为我们的消息类型。

不过这可能会变得重复，因为我们需要两次指定我们的字段，所以我们可能希望引入一个辅助库，可以统一验证和声明我们的消息类型：

*带有模式的命令工厂（`src/allocation/commands.py`）*

```py
def command(name, **fields):  #(1)
    schema = Schema(And(Use(json.loads), fields), ignore_extra_keys=True)
    cls = make_dataclass(name, fields.keys())  #(2)
    cls.from_json = lambda s: cls(**schema.validate(s))  #(3)
    return cls

def greater_than_zero(x):
    return x > 0

quantity = And(Use(int), greater_than_zero)  #(4)

Allocate = command(  #(5)
    orderid=int,
    sku=str,
    qty=quantity
)

AddStock = command(
    sku=str,
    qty=quantity
```

①

`command`函数接受一个消息名称，以及消息有效负载字段的 kwargs，其中 kwarg 的名称是字段的名称，值是解析器。

②

我们使用数据类模块的`make_dataclass`函数动态创建我们的消息类型。

③

我们将`from_json`方法打补丁到我们的动态数据类上。

④

我们可以创建可重用的解析器来解析数量、SKU 等，以保持代码的 DRY。

⑤

声明消息类型变成了一行代码。

这是以失去数据类上的类型为代价的，所以要考虑这种权衡。

# Postel's Law 和宽容读者模式

*Postel's law*，或者*鲁棒性原则*，告诉我们，“在接受时要宽容，在发出时要保守。”我们认为这在与其他系统集成的情境中特别适用。这里的想法是，当我们向其他系统发送消息时，我们应该严格要求，但在接收他人消息时尽可能宽容。

例如，我们的系统*可以*验证 SKU 的格式。我们一直在使用像`UNFORGIVING-CUSHION`和`MISBEGOTTEN-POUFFE`这样的虚构 SKU。这遵循一个简单的模式：由破折号分隔的两个单词，其中第二个单词是产品类型，第一个单词是形容词。

开发人员*喜欢*验证消息中的这种内容，并拒绝任何看起来像无效 SKU 的内容。当某个无政府主义者发布名为`COMFY-CHAISE-LONGUE`的产品或供应商出现问题导致`CHEAP-CARPET-2`的发货时，这将在后续过程中造成可怕的问题。

实际上，作为分配系统，SKU 的格式*与我们无关*。我们只需要一个标识符，所以我们可以简单地将其描述为一个字符串。这意味着采购系统可以随时更改格式，而我们不会在意。

同样的原则适用于订单号、客户电话号码等。在大多数情况下，我们可以忽略字符串的内部结构。

同样，开发人员*喜欢*使用 JSON Schema 等工具验证传入消息，或构建验证传入消息并在系统之间共享的库。这同样无法通过健壮性测试。

例如，假设采购系统向`ChangeBatchQuantity`消息添加了记录更改原因和负责更改的用户电子邮件的新字段。

由于这些字段对分配服务并不重要，我们应该简单地忽略它们。我们可以通过传递关键字参数`ignore_extra_keys=True`来在`schema`库中实现这一点。

这种模式，即我们仅提取我们关心的字段并对它们进行最小的验证，就是宽容读者模式。

###### 提示

尽量少进行验证。只读取您需要的字段，不要过度指定它们的内容。这将有助于您的系统在其他系统随着时间的变化而保持健壮。抵制在系统之间共享消息定义的诱惑：相反，使定义您所依赖的数据变得容易。有关更多信息，请参阅 Martin Fowler 的文章[Tolerant Reader pattern](https://oreil.ly/YL_La)。

# 在边缘验证

我们曾经说过，我们希望避免在我们的代码中充斥着无关的细节。特别是，我们不希望在我们的领域模型内部进行防御性编码。相反，我们希望确保在我们的领域模型或用例处理程序看到它们之前，已知请求是有效的。这有助于我们的代码在长期内保持干净和可维护。我们有时将其称为*在系统边缘进行验证*。

除了保持您的代码干净并且没有无休止的检查和断言之外，要记住，系统中漫游的无效数据就像是一颗定时炸弹；它越深入，造成的破坏就越大，而您可以用来应对它的工具就越少。

在第八章中，我们说消息总线是一个很好的放置横切关注点的地方，验证就是一个完美的例子。以下是我们如何改变我们的总线来执行验证的方式：

*验证*

```py
class MessageBus:

    def handle_message(self, name: str, body: str):
        try:
            message_type = next(mt for mt in EVENT_HANDLERS if mt.__name__ == name)
            message = message_type.from_json(body)
            self.handle([message])
        except StopIteration:
            raise KeyError(f"Unknown message name {name}")
        except ValidationError as e:
            logging.error(
                f'invalid message of type {name}\n'
                f'{body}\n'
                f'{e}'
            )
            raise e
```

以下是我们如何从我们的 Flask API 端点使用该方法：

*API 在处理 Redis 消息时出现验证错误（`src/allocation/flask_app.py`）*

```py
@app.route("/change_quantity", methods=['POST'])
def change_batch_quantity():
    try:
        bus.handle_message('ChangeBatchQuantity', request.body)
    except ValidationError as e:
        return bad_request(e)
    except exceptions.InvalidSku as e:
        return jsonify({'message': str(e)}), 400

def bad_request(e: ValidationError):
    return e.code, 400
```

以下是我们如何将其插入到我们的异步消息处理器中：

*处理 Redis 消息时出现验证错误（`src/allocation/redis_pubsub.py`）*

```py
def handle_change_batch_quantity(m, bus: messagebus.MessageBus):
    try:
        bus.handle_message('ChangeBatchQuantity', m)
    except ValidationError:
       print('Skipping invalid message')
    except exceptions.InvalidSku as e:
        print(f'Unable to change stock for missing sku {e}')
```

请注意，我们的入口点仅关注如何从外部世界获取消息以及如何报告成功或失败。我们的消息总线负责验证我们的请求并将其路由到正确的处理程序，而我们的处理程序则专注于用例的逻辑。

###### 提示

当您收到无效的消息时，通常除了记录错误并继续之外，你几乎无能为力。在 MADE，我们使用指标来计算系统接收的消息数量，以及其中有多少成功处理、跳过或无效。如果我们看到坏消息数量的激增，我们的监控工具会向我们发出警报。

# 验证语义

虽然语法涉及消息的结构，*语义*是对消息中*含义*的研究。句子“Undo no dogs from ellipsis four”在语法上是有效的，并且与句子“Allocate one teapot to order five”具有相同的结构，但它是毫无意义的。

我们可以将这个 JSON 块解读为一个“分配”命令，但无法成功执行它，因为它是*无意义的*：

*一个毫无意义的消息*

```py
{
  "orderid": "superman",
  "sku": "zygote",
  "qty": -1
}
```

我们倾向于在消息处理程序层验证语义关注点，采用一种基于合同的编程：

*前提条件（`src/allocation/ensure.py`）*

```py
"""
This module contains preconditions that we apply to our handlers.
"""

class MessageUnprocessable(Exception):  #(1)

    def __init__(self, message):
        self.message = message

class ProductNotFound(MessageUnprocessable):  #(2)
    """"
    This exception is raised when we try to perform an action on a product
    that doesn't exist in our database.
    """"

    def __init__(self, message):
        super().__init__(message)
        self.sku = message.sku

def product_exists(event, uow):  #(3)
    product = uow.products.get(event.sku)
    if product is None:
        raise ProductNotFound(event)
```

①

我们使用一个通用的错误基类，表示消息无效。

②

为此问题使用特定的错误类型使得更容易报告和处理错误。例如，将`ProductNotFound`映射到 Flask 中的 404 很容易。

③

`product_exists`是一个前提条件。如果条件为`False`，我们会引发一个错误。

这样可以使我们的服务层的主要逻辑保持清晰和声明式：

*在服务中确保调用（`src/allocation/services.py`）*

```py
# services.py

from allocation import ensure

def allocate(event, uow):
    line = mode.OrderLine(event.orderid, event.sku, event.qty)
    with uow:
        ensure.product_exists(uow, event)

        product = uow.products.get(line.sku)
        product.allocate(line)
        uow.commit()
```

我们可以扩展这个技术，以确保我们幂等地应用消息。例如，我们希望确保我们不会多次插入一批库存。

如果我们被要求创建一个已经存在的批次，我们将记录一个警告并继续下一个消息：

*为可忽略的事件引发 SkipMessage 异常（`src/allocation/services.py`）*

```py
class SkipMessage (Exception):
    """"
 This exception is raised when a message can't be processed, but there's no
 incorrect behavior. For example, we might receive the same message multiple
 times, or we might receive a message that is now out of date.
 """"

    def __init__(self, reason):
        self.reason = reason

def batch_is_new(self, event, uow):
    batch = uow.batches.get(event.batchid)
    if batch is not None:
        raise SkipMessage(f"Batch with id {event.batchid} already exists")
```

引入`SkipMessage`异常让我们以一种通用的方式处理这些情况在我们的消息总线中：

*公交车现在知道如何跳过（`src/allocation/messagebus.py`）*

```py
class MessageBus:

    def handle_message(self, message):
        try:
           ...
       except SkipMessage as e:
           logging.warn(f"Skipping message {message.id} because {e.reason}")
```

这里需要注意一些陷阱。首先，我们需要确保我们使用的是与用例的主要逻辑相同的 UoW。否则，我们会让自己遭受恼人的并发错误。

其次，我们应该尽量避免将*所有*业务逻辑都放入这些前提条件检查中。作为一个经验法则，如果一个规则*可以*在我们的领域模型内进行测试，那么它*应该*在领域模型中进行测试。

# 验证语用学

*语用学*是研究我们如何在语境中理解语言的学科。在解析消息并理解其含义之后，我们仍然需要在上下文中处理它。例如，如果你在拉取请求上收到一条评论说“我认为这非常勇敢”，这可能意味着评论者钦佩你的勇气，除非他们是英国人，在这种情况下，他们试图告诉你你正在做的事情是非常冒险的，只有愚蠢的人才会尝试。上下文是一切。

###### 提示

一旦在系统边缘验证了命令的语法和语义，领域就是其余验证的地方。验证语用学通常是业务规则的核心部分。

在软件术语中，操作的语用学通常由领域模型管理。当我们收到像“allocate three million units of `SCARCE-CLOCK` to order 76543”这样的消息时，消息在*语法*上有效且*语义*上有效，但我们无法遵守，因为我们没有库存可用。
