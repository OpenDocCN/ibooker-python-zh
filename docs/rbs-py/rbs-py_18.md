# 第十五章：可扩展性

本章重点介绍可扩展性。可扩展性是本书的重点内容之一；理解这一关键概念非常重要。一旦你了解了可扩展性对稳健性的影响，你就会开始看到在代码库中应用它的机会。可扩展的系统允许其他开发人员自信地增强你的代码库，从而减少错误的机会。让我们来看看是如何做到这一点的。

# 什么是可扩展性？

*可扩展性*是系统的属性，允许在不修改系统现有部分的情况下添加新功能。软件不是静态的；它会发生变化。在你的代码库的生命周期中，开发人员会改变你的软件。*软件*中的*软*部分就表示这一点。这些变化可能是相当大的。想想你需要在扩展规模时交换体系结构的关键部分，或者添加新的工作流程的情况。这些变化会触及代码库的多个部分；简单的类型检查无法在这个级别捕获所有错误。毕竟，你可能正在完全重新设计你的类型。可扩展软件的目标是以一种使未来开发人员能够轻松扩展的方式设计，特别是在经常更改的代码区域。

为了阐明这个想法，让我们考虑一个希望实现某种通知系统以帮助供应商应对需求的连锁餐厅。餐厅可能有特色菜，或者缺少某种成分，或者表明某种成分已经变质。在每种情况下，餐厅都希望供应商能够自动收到通知，提示需要重新补货。供应商已经提供了一个用于实际通知的 Python 库。

实现看起来像下面这样：

```py
def declare_special(dish: Dish, start_date: datetime.datetime,
                    end_time: datetime.datetime):
    # ... snip setup in local system ...
    # ... snip send notification to the supplier ...

def order_dish(dish: Dish):
    # ... snip automated preparation
    out_of_stock_ingredients = {ingred for ingred in dish
                                if out_of_stock(ingred)}
    if out_of_stock_ingredients:
        # ... snip taking dishes off menu ...
        # ... snip send notification to the supplier ...

# called every 24 hours
def check_for_expired_ingredients():
    expired_ingredients = {ing for ing in ingredient in get_items_in_stock()}:
    if expired_ingredients:
        # ... snip taking dishes off menu ...
        # ... snip send notifications to the supplier ...
```

这段代码一开始看起来很简单。每当发生重要事件时，适当的通知就可以发送给供应商（想象一下将某个字典作为 JSON 请求的一部分发送）。

快进几个月，一个新的工作项出现了。你在餐厅的老板对通知系统非常满意，他们想要扩展它。他们希望通知发送到他们的电子邮件地址。听起来很简单，对吧？你让`declare_special`函数也接受电子邮件地址：

```py
def declare_special(notification: NotificationType,
                    start_date: datetime.datetime,
                    end_time: datetime.datetime,
                    email: Email):
    # ... snip ...
```

不过，这有着深远的影响。调用`declare_special`的函数也需要知道传递下去的电子邮件是什么。幸运的是，类型检查器会捕获任何遗漏。但如果出现其他用例会怎样呢？你看了一下你的待办事项清单，下面的任务都在那里：

+   通知销售团队有关特价和缺货商品。

+   通知餐厅的顾客有关新特价。

+   支持为不同的供应商使用不同的 API。

+   支持文本消息通知，以便你的老板也可以收到通知。

+   创建一个新的通知类型：新菜单项。市场营销人员和老板想了解这一点，但供应商不想知道。

当开发人员实现这些功能时，`declare_special`变得越来越庞大。它处理的案例越来越多，逻辑变得越来越复杂，出错的可能性也增加。更糟糕的是，对 API 的任何更改（比如添加一个用于发送短信的电子邮件地址或电话号码列表）都会对所有调用方产生影响。在某个时候，像将新的电子邮件地址添加到营销人员列表中这样的简单操作会触及代码库中的多个文件。这在口语中被称为“散弹手术”：¹，一个变化会在一系列文件中传播开来，影响各种文件。此外，开发人员正在修改现有的代码，增加出错的机会。更糟糕的是，我们只涉及了`declare_special`，但`order_dish`和`check_for_expired_ingredients`也需要它们自己的定制逻辑。处理到处重复的通知代码将会非常乏味。问问自己，如果一个新用户想要文本通知，你会喜欢不得不在整个代码库中寻找每一个通知片段吗？

这一切都源于代码不太可扩展。您开始要求开发人员了解多个文件的所有细节才能进行更改。维护者实现其功能将需要更多的工作。回想一下第一章中对意外复杂性和必要复杂性的讨论。必要复杂性是与您的问题领域内在相关的；意外复杂性是您引入的复杂性。在这种情况下，通知、接收者和过滤器的组合是必要的；它是系统的必需功能。

然而，您实现系统的方式决定了您承担多少意外复杂性。我描述的方式充满了意外复杂性。添加任何一个简单的东西都是一项相当大的工程。要求开发人员在代码库中搜索需要更改的所有位置只会招来麻烦。简单的更改应该很容易实现。否则，每次扩展系统都会变成一项琐事。

## 重新设计

让我们再次看一下`declare_special`函数：

```py
def declare_special(notification: NotificationType,
                    start_date: datetime.datetime,
                    end_time: datetime.datetime,
                    email: Email):
    # ... snip ...
```

所有问题都始于向函数添加电子邮件作为参数。这导致了涟漪效应，影响了代码库的其他部分。这不是未来开发人员的错；他们通常受到时间的限制，试图将他们的功能塞进他们不熟悉的代码库的某个部分。他们通常会遵循已为他们铺平道路的模式。如果您能奠定基础，引导他们朝着正确的方向发展，您就会提高代码的可维护性。如果您让可维护性滋生，您会开始看到如下方法：

```py
def declare_special(notification: NotificationType,
                    start_date: datetime.datetime,
                    end_time: datetime.datetime,
                    emails: list[Email],
                    texts: list[PhoneNumber],
                    send_to_customer: bool):
    # ... snip ...
```

函数将会不断增长，直到它变成一个纠缠不清的依赖混乱。如果我需要将客户添加到邮件列表，为什么我还需要查看如何声明特价？

需要重新设计通知系统，以便于进行更改。首先，我将查看使用情况，并考虑未来开发人员需要简化的内容。（如果您需要有关设计界面的额外建议，请参阅第二部分，特别是第十一章。）在这个特定的使用情况中，我希望未来的开发人员能够轻松添加三个事项：

+   新通知类型

+   新通知方法（例如电子邮件、短信或 API）

+   新用户需要通知

通知代码散落在代码库中，因此我希望在开发人员进行这些更改时，他们无需进行任何散弹手术。记住，我希望简单的事情变得简单。

现在，考虑我*必要的*复杂性。在这种情况下，将有多种通知方法、多种通知类型和多个需要收到通知的用户。这些是三种不同的复杂性；我希望限制它们之间的交互。`declare_special`的部分问题在于它必须考虑的关注点的组合是令人生畏的。将这种复杂性乘以每个需要稍有不同通知需求的函数，你将面临一个维护的真正噩梦。

首先要做的是尽量解耦意图。我将首先为每种通知类型创建类：

```py
@dataclass
class NewSpecial:
    dish: Dish
    start_date: datetime.datetime
    end_date: datetime.datetime

@dataclass
class IngredientsOutOfStock:
    ingredients: Set[Ingredient]

@dataclass
class IngredientsExpired:
    ingredients: Set[Ingredient]

@dataclass
class NewMenuItem:
    dish: Dish

Notification = Union[NewSpecial, IngredientsOutOfStock,
                     IngredientsExpired, NewMenuItem]
```

如果我考虑`declare_special`如何与代码库进行交互，我真的只希望它知道这个`NotificationType`。声明特殊不应该需要知道谁注册了该特殊以及他们将如何收到通知。理想情况下，`declare_special`（以及任何需要发送通知的其他函数）应该看起来像这样：

```py
def declare_special(dish: Dish, start_date: datetime.datetime,
                    end_time: datetime.datetime):
    # ... snip setup in local system ...
    send_notification(NewSpecial(dish, start_date, end_date))
```

`send_notification`可以像这样声明：

```py
def send_notification(notification: Notification):
    # ... snip ...
```

这意味着如果代码库的任何部分想要发送通知，它只需调用这个函数。你只需要传入一个通知类型。添加新的通知类型很简单；你添加一个新的类，将该类添加到`Union`中，并调用`send_notification`以使用新的通知类型。

接下来，您需要轻松地添加新的通知方法。同样，我将添加新的类型来代表每种通知方法：

```py
@dataclass
class Text:
    phone_number: str

@dataclass
class Email:
    email_address: str

@dataclass
class SupplierAPI:
    pass

NotificationMethod = Union[Text, Email, SupplierAPI]
```

在代码库的某个地方，我需要根据每种方法发送不同的通知类型。我可以创建一些辅助函数来处理这个功能：

```py
def notify(notification_method: NotificationMethod, notification: Notification):
    if isinstance(notification_method, Text):
        send_text(notification_method, notification)
    elif isinstance(notification_method, Email):
        send_email(notification_method, notification)
    elif isinstance(notification_method, SupplierAPI):
        send_to_supplier(notification)
    else:
        raise ValueError("Unsupported Notification Method")

def send_text(text: Text, notification: Notification):
    if isinstance(notification, NewSpecial):
        # ... snip send text ...
        pass
    elif isinstance(notification, IngredientsOutOfStock):
        # ... snip send text ...
        pass
    elif isinstance(notification, IngredientsExpired):
        # ... snip send text ...
        pass
    elif isinstance(notification, NewMenuItem):
        # .. snip send text ...
        pass
    raise NotImplementedError("Unsupported Notification Method")

def send_email(email: Email, notification: Notification):
    # .. similar to send_text ...

def send_to_supplier(notification: Notification):
    # .. similar to send_text
```

现在，添加新的通知方法也很简单。我添加一个新类型，将其添加到联合中，在`notify`中添加一个`if`语句，并编写一个相应的方法来处理所有不同的通知类型。

在每个`send_***`方法中处理所有通知类型可能看起来很笨重，但这是必要的复杂性；由于不同的消息、不同的信息和不同的格式，每种方法/类型组合都有不同的功能。如果代码量太大，你可以创建一个动态查找字典（这样添加新的键值对就是需要的所有内容，用于添加通知方法），但在这些情况下，你将牺牲早期错误检测以换取更好的可读性。

现在我有了添加新的通知方法或类型的简单方法。我只需要将它们全部联系起来，以便轻松添加新用户。为此，我将编写一个函数来获取需要通知的用户列表：

```py
users_to_notify: Dict[type, List[NotificationMethod]] = {
    NewSpecial: [SupplierAPI(), Email("boss@company.org"),
                 Email("marketing@company.org"), Text("555-2345")],
    IngredientsOutOfStock: [SupplierAPI(), Email("boss@company.org")],
    IngredientsExpired: [SupplierAPI(), Email("boss@company.org")],
    NewMenuItem: [Email("boss@company.org"), Email("marketing@company.org")]
}
```

在实践中，这些数据可能来自配置文件或其他声明性来源，但为了书中的简洁性，它就足够了。要添加新用户，我只需在这个字典中添加一个新条目。为用户添加新的通知方法或通知类型同样简单。处理需要通知的用户的代码要容易得多。

为了将所有这些概念整合到一起，我将实现`send_notification`：

```py
def send_notification(notification: Notification):
    try:
        users = users_to_notify[type(notification)]
    except KeyError:
        raise ValueError("Unsupported Notification Method")
    for notification_method in users:
        notify(notification_method, notification)
```

就是这样！所有这些通知代码可以放在一个文件中，代码库的其余部分只需要知道一个函数——`send_notification`——与通知系统进行交互。一旦不再需要与代码库的任何其他部分进行交互，这将使得测试变得更加容易。此外，这段代码是可扩展的；开发者可以轻松地添加新的通知类型、方法或用户，而无需搜索整个代码库以查找所有复杂的调用。你希望在最小化对现有代码修改的同时，轻松地向你的代码库添加新功能。这就是开闭原则。

# 开闭原则

*开闭原则*（OCP）指出，代码应该对扩展开放，对修改关闭²。这是可扩展性的核心。我们在前一节的重设计中试图遵循这一原则。与要求新功能触及代码库的多个部分不同，它要求添加新类型或函数。即使现有函数发生了变化，我也只是添加了一个新的条件检查，而不是修改现有的检查。

看起来我一直在追求代码重用，但开闭原则（OCP）更进一步。是的，我已经去重了通知代码，但更重要的是，我让开发者更容易管理复杂性。问问自己，你更喜欢哪个：通过检查调用堆栈来实现一个功能，但不确定是否找到了需要更改的每个位置，还是修改简单且不需要大幅修改的一个文件。我知道我会选择哪个。

你已经在本书中接触过 OCP。鸭子类型（在第二章）、子类型（在第十二章）、以及协议（在第十三章）都是可以帮助实现 OCP 的机制。所有这些机制的共同点是它们允许您以一种通用的方式编程。你不再需要直接处理每个特殊情况，而是为其他开发人员提供扩展点，让他们能够在不修改你的代码的情况下注入自己的功能。

OCP 是可扩展性的核心。保持代码的可扩展性将提高鲁棒性。开发人员可以自信地实现功能；只需在一个地方进行更改，而其他代码库已经准备好支持该更改。较少的认知开销和较少的代码更改将导致较少的错误。

## 检测 OCP 违规

如何判断你是否应该编写更易扩展的代码，遵循 OCP？以下是一些指标，当你考虑你的代码库时应该引起注意：

看起来容易的事情难以做到吗？

你的代码库中应该有一些概念上容易的事情。实现这个概念所需的工作量应该与领域复杂性相匹配。我曾经在一个代码库中工作，为了添加一个用户可配置选项需要修改 13 个不同的文件。对于一个有数百个可配置选项的产品来说，这应该是一个容易的任务。可以说，事实并非如此。

你是否遇到了类似特性的阻力？

如果特性请求者经常对特性的时间表提出异议，特别是如果，在他们的话中，它“几乎与以前的特性*X*相同”，请问是否存在复杂性的脱节。可能复杂性固有于领域中，在这种情况下，你应该确保特性请求者与你在同一页面上。但如果复杂性是偶然的，那么你的代码可能需要重新设计，以使其更容易操作。

你是否一直有着高估算？

一些团队使用估算来预测他们在给定时间范围内要完成的工作量。如果特性一直有很高的估算，请问估算的来源是什么。是复杂性驱动了高估算吗？那种复杂性是必要的吗？还是风险和未知的恐惧？如果是后者，请问为什么你的代码库感觉有风险？有些团队将特性分割成独立的估算，通过分割工作。如果你一直这样做，请问重组代码库是否可以减轻分割。

提交包含大的变更集吗？

查找你的版本控制系统中具有大量文件的提交。这很可能表明“散弹手术”正在发生，尤其是如果相同的文件在多个提交中反复出现。请记住，这只是一个指导原则；大提交并不总是表示问题，但如果频繁发生，值得进一步检查。

# 讨论话题

在你的代码库中遇到了哪些 OCP（开闭原则）的违规情况？你如何重构代码以避免这些问题？

## 缺点

可扩展性并非解决所有编码问题的灵丹妙药。事实上，如果过度灵活，你的代码库实际上可能会*降级*。如果你过度使用 OCP 并试图使所有东西可配置和可扩展，你很快会发现自己陷入困境。问题在于，虽然使代码可扩展可以减少在进行更改时的意外复杂性，但它可能会*增加*其他方面的意外复杂性。

首先，可读性下降。你正在创建一个全新的抽象层，将你的业务逻辑与代码库的其他部分分离开来。任何想要理解整体图片的人都必须跳过一些额外的步骤。这将影响新开发人员快速上手，同时也会妨碍调试工作。你可以通过良好的文档和解释代码结构来缓解这种情况。

其次，你引入了一个可能之前不存在的耦合。之前，代码库的各个部分是相互独立的。现在，它们共享一个公共子系统；该子系统的任何变化都会影响所有的使用者。我将在第十六章中深入讨论这一点。通过一套强大的测试来减轻这种影响。

适度使用 OCP，并在应用这些原则时谨慎行事。如果使用过度，你的代码库将被过度抽象化和混乱的依赖关系所困扰。使用过少，则开发人员将需要更长时间来进行更改，并引入更多的错误。在你合理确定某些区域需要再次修改时，定义扩展点将大大改善未来维护者在处理你的代码库时的体验。

# 总结思考

可扩展性是代码库维护中最重要的方面之一。它允许你的协作者在不修改现有代码的情况下添加功能。任何时候，避免修改现有代码，就是避免引入回归的时机。现在添加可扩展的代码可以预防未来的错误。记住 OCP 原则：保持代码对扩展开放，但对修改关闭。合理应用这一原则，你将看到你的代码库变得更易维护。

可扩展性是接下来几章将贯穿始终的重要主题。在下一章中，我将专注于依赖关系以及代码库中的关系如何限制其可扩展性。您将了解不同类型的依赖关系以及如何管理它们。您将学习如何可视化和理解您的依赖关系，以及为什么您的代码库中的某些部分可能具有更多的依赖关系。一旦开始管理您的依赖关系，您将发现扩展和修改代码变得更加容易。

¹ 马丁·福勒。*重构：改善现有代码的设计*。第二版。上沙德尔河，NJ：Addison-Wesley Professional，2018。

² OCP 首次在 Bertrand Meyer（Pearson）的*面向对象软件构建*中描述。
