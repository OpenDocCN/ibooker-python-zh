# 第四章：类型约束

许多开发人员学习了基本类型注解就结束了。但我们远未结束。有许多宝贵的高级类型注解。这些高级类型注解允许您约束类型，进一步限制它们可以表示的内容。您的目标是使非法状态无法表示。如果根本不可能在系统中创建错误，那么您的代码中就不会出现错误。您可以使用类型注解来实现这个目标，节省时间和金钱。在本章中，我将教您六种不同的技术：

`Optional`

用于在您的代码库中替换`None`引用。

`Union`

用于呈现一组类型的选择。

`Literal`

用于限制开发人员使用非常特定的值。

`Annotated`

用于提供类型的附加描述。

`NewType`

用于将类型限制在特定上下文中。

`Final`

用于防止变量被重新绑定到新值。

让我们从使用`Optional`类型处理`None`引用开始。

# 可选类型

空引用通常被称为“十亿美元的错误”，这个术语是由 C.A.R. Hoare 创造的：

> 我称之为我的十亿美元的错误。这是在 1965 年发明的空引用。当时，我正在设计一种用于对象导向语言中引用的第一个全面类型系统。我的目标是确保所有引用的使用都绝对安全，并由编译器自动执行检查。但我无法抵挡诱惑，简单地实现了一个空引用。这导致了无数的错误、漏洞和系统崩溃，这些问题可能在过去四十年中造成了数十亿美元的损失和损害。¹

尽管空引用起源于 Algol 语言，但它们已经渗透到无数其他语言中。C 和 C++经常因空指针解引用而受到嘲笑（导致段错误或其他导致程序崩溃的问题）。Java 因要求用户在整个代码中捕获`NullPointerException`而广为人知。可以毫不夸张地说，这些类型的错误具有以十亿计的代价。想想由于意外的空指针或引用而导致的开发人员时间、客户损失和系统故障。

那么，在 Python 中为什么这很重要呢？Hoare 的引用是关于 60 年代的面向对象编译语言；Python 现在一定会更好，对吧？我很遗憾地告诉您，这个十亿美元的错误也存在于 Python 中。它以不同的名字出现在我们面前：`None`。我将向您展示一种避免昂贵`None`错误的方法，但首先，让我们谈谈为什么`None`如此糟糕。

###### 注意

尤其值得注意的是，霍尔承认，空引用是为了方便而产生的。这表明了选择更快捷的路径如何会在开发生命周期的后期导致各种痛苦。想想你今天的短期决策会如何不利于明天的维护。

让我们考虑一些运行自动热狗摊的代码。我希望我的系统能够拿起一个面包，把一根热狗放在面包里，然后通过自动发泡器挤出番茄酱和芥末，就像图 4-1 中描述的那样。会出什么问题呢？

![自动热狗摊的工作流程](img/00004.gif)

###### 图 4-1\. 自动热狗摊的工作流程

```py
def create_hot_dog():
    bun = dispense_bun()
    frank = dispense_frank()
    hot_dog = bun.add_frank(frank)
    ketchup = dispense_ketchup()
    mustard = dispense_mustard()
    hot_dog.add_condiments(ketchup, mustard)
    dispense_hot_dog_to_customer(hot_dog)
```

相当直接，对吧？不幸的是，没有办法真正确定。想清楚正常路径，或者程序的控制流程当一切顺利时，是很容易的，但是当谈论到健壮的代码时，你需要考虑错误条件。如果这是一个没有人工干预的自动摊位，你能想到什么错误？

这是我能想到的错误的非全面列表：

+   材料不足（面包、热狗或番茄酱/芥末）。

+   订单在过程中取消。

+   调料堵塞。

+   电力中断。

+   客户不想要番茄酱或芥末，并试图在过程中移动面包。

+   竞争对手用番茄酱替换了番茄酱；混乱随之而来。

现在，你的系统是最先进的，会检测所有这些情况，但是当任何一步失败时，它会返回`None`。这对这段代码意味着什么？你开始看到以下错误：

```py
Traceback (most recent call last):
 File "<stdin>", line 4, in <module>
AttributeError: 'NoneType' object has no attribute 'add_frank'

Traceback (most recent call last):
 File "<stdin>", line 7, in <module>
AttributeError: 'NoneType' object has no attribute 'add_condiments'
```

如果这些错误泡沫冒到你的客户那里会很灾难性；你以清爽的用户界面为傲，不希望丑陋的回溯污染你的界面。为了解决这个问题，你开始进行*防御性编程*，或者以一种能够预见每种可能错误情况并加以考虑的方式进行编码。防御性编程是一件好事，但它导致了这样的代码：

```py
def create_hot_dog():
    bun = dispense_bun()
    if bun is None:
        print_error_code("Bun unavailable. Check for bun")
        return

    frank = dispense_frank()
    if frank is None:
        print_error_code("Frank was not properly dispensed")
        return

    hot_dog = bun.add_frank(frank)
    if hot_dog is None:
        print_error_code("Hot Dog unavailable. Check for Hot Dog")
        return

    ketchup = dispense_ketchup()
    mustard = dispense_mustard()
    if ketchup is None or mustard is None:
        print_error_code("Check for invalid catsup")
        return

    hot_dog.add_condiments(ketchup, mustard)
    dispense_hot_dog_to_customer(hot_dog)
```

这感觉，嗯，很烦人。因为在 Python 中任何值都可以是`None`，所以似乎你需要进行防御性编程，并在每次解除引用之前进行一个`is None`检查。这是多余的；大多数开发人员会跟踪调用堆栈，并确保不会将`None`值返回给调用者。这是错误的；你不能指望每个接触你的代码库的开发人员都能本能地知道在哪里检查`None`。此外，你编写代码时所做的原始假设（例如，此函数永远不会返回`None`）可能会在将来被打破，现在你的代码有了一个 bug。问题就在这里：依靠手动干预来捕捉错误情况是不可靠的。

这是如此棘手（也如此昂贵）的原因是，`None` 被视为一种特殊情况。它存在于正常类型层次结构之外。每个变量都可以被赋值为 `None`。为了应对这一点，你需要找到一种在你的类型层次结构中表示 `None` 的方法。你需要使用 `Optional` 类型。

`Optional` 类型为你提供两种选择：要么你有一个值，要么你没有。换句话说，将变量设置为一个值是可选的。

```py
from typing import Optional
maybe_a_string: Optional[str] = "abcdef" # This has a value
maybe_a_string: Optional[str] = None     # This is the absence of a value
```

这段代码指示变量 `maybe_a_string` 可能包含一个字符串。无论 `maybe_a_string` 包含 `"abcdef"` 还是 `None`，该代码都能成功通过类型检查。

乍一看，不明显它能为你带来什么。你仍然需要使用 `None` 来表示值的缺失。不过，我有个好消息告诉你。我将 `Optional` 类型与三个好处关联起来。

首先，你可以更清晰地表达你的意图。如果开发者在类型签名中看到 `Optional` 类型，他们会把它看作一个大红旗，表明他们应该预期可能会出现 `None`。

```py
def dispense_bun() -> Optional[Bun]:
# ...
```

如果你注意到一个函数返回一个 `Optional` 值，请注意并检查 `None` 值。

其次，你能够进一步区分值的缺失和空值。考虑一个无害的列表。如果你进行一个函数调用并接收到一个空列表，会发生什么？是因为没有结果返回给你吗？还是发生了错误，你需要采取明确的行动？如果你收到一个原始列表，你不知道，除非你深入源代码。然而，如果你使用 `Optional`，你传达了三种可能性中的一种：

一个有元素的列表

可操作的有效数据

一个没有元素的列表

没有发生错误，但没有可用数据（假设没有数据不是错误条件）

`None`

发生了一个需要处理的错误

最后，类型检查器可以检测到 `Optional` 类型，并确保你没有让 `None` 值溜过去。

考虑：

```py
def dispense_bun() -> Bun:
    return Bun('Wheat')
```

让我们给这段代码添加一些错误情况：

```py
def dispense_bun() -> Bun:
    if not are_buns_available():
        return None
    return Bun('Wheat')
```

当使用类型检查器运行时，你将得到以下错误：

```py
code_examples/chapter4/invalid/dispense_bun.py:12:
    error: Incompatible return value type (got "None", expected "Bun")
```

太好了！类型检查器将不允许你默认返回 `None` 值。通过将返回类型从 `Bun` 更改为 `Optional[Bun]`，这段代码将成功地通过类型检查。这将为开发者提供提示，表明他们不应该在不在返回类型中编码信息的情况下返回 `None`，可以捕捉一个常见的错误，并使这段代码更加健壮。但是调用代码呢？

结果表明，调用代码也从中受益。考虑：

```py
def create_hot_dog():
    bun = dispense_bun()
    frank = dispense_frank()
    hot_dog = bun.add_frank(frank)
    ketchup = dispense_ketchup()
    mustard = dispense_mustard()
    hot_dog.add_condiments(ketchup, mustard)
    dispense_hot_dog_to_customer(hot_dog)
```

如果 `dispense_bun` 返回一个 `Optional`，这段代码将无法通过类型检查。它会报以下错误：

```py
code_examples/chapter4/invalid/hotdog_invalid.py:27:
    error: Item "None" of "Optional[Bun]" has no attribute "add_frank"
```

###### 警告

根据你的类型检查器，你可能需要专门启用一个选项来捕捉这些错误。始终查看你的类型检查器的文档，了解有哪些选项可用。如果有一个你绝对想要捕捉的错误，你应该测试一下你的类型检查器确实捕捉到了这个错误。我强烈建议专门测试`Optional`的行为。对于我正在运行的`mypy`版本（0.800），我必须使用`--strict-optional`作为命令行标志来捕捉这个错误。

如果你想要消除类型检查器的警告，你需要显式检查`None`并处理`None`值，或者断言该值不能是`None`。以下代码可以成功进行类型检查：

```py
def create_hot_dog():
    bun = dispense_bun()
    if bun is None:
        print_error_code("Bun could not be dispensed")
        return

    frank = dispense_frank()
    hot_dog = bun.add_frank(frank)
    ketchup = dispense_ketchup()
    mustard = dispense_mustard()
    hot_dog.add_condiments(ketchup, mustard)
    dispense_hot_dog_to_customer(hot_dog)
```

`None`值真的是一个十亿美元的错误。如果它们溜过去，程序可能会崩溃，用户会感到沮丧，钱会丢失。使用`Optional`类型告诉其他开发者注意`None`，并从你的工具的自动检查中受益。

# 讨论主题

你的代码库中经常处理`None`吗？你对每个可能的`None`值是否处理正确有多自信？查看错误和失败的测试，看看你被错误的`None`处理咬了多少次。讨论一下`Optional`类型将如何帮助你的代码库。

# 联合类型

`Union`类型是指多个不同的类型可以与同一个变量一起使用的类型。`Union[int,str]`表示一个变量可以使用`int` *或*`str`。例如，考虑以下代码：

```py
def dispense_snack() -> HotDog:
    if not are_ingredients_available():
        raise RuntimeError("Not all ingredients available")
    if order_interrupted():
        raise RuntimeError("Order interrupted")
    return create_hot_dog()
```

我现在希望我的热狗摊能进入利润丰厚的椒盐脆饼业务。而不是试图处理不应存在于热狗和椒盐脆饼之间的奇怪的类继承（我们将在第二部分中更多地介绍继承），你只需返回这两者的`Union`。

```py
from typing import Union
def dispense_snack(user_input: str) -> Union[HotDog, Pretzel]:
    if user_input == "Hot Dog":
        return dispense_hot_dog()
    elif user_input == "Pretzel":
        return dispense_pretzel()
    raise RuntimeError("Should never reach this code,"
                       "as an invalid input has been entered")
```

###### 注意

`Optional`只是`Union`的一种特殊版本。`Optional[int]`与`Union[int, None]`完全相同。

使用`Union`提供了与`Optional`几乎相同的好处。首先，你能得到同样的沟通优势。遇到`Union`的开发者知道他们必须能够处理调用代码中的多个类型。此外，类型检查器和`Optional`一样了解`Union`。

你会发现`Union`在各种应用中都很有用：

+   根据用户输入处理返回的不同类型（如上所述）

+   处理错误返回类型，比如`Optional`，但提供更多信息，比如字符串或错误代码

+   处理不同的用户输入（例如，如果用户能够提供一个列表或一个字符串）

+   返回不同的类型，比如为了向后兼容性（根据请求的操作返回一个旧版本的对象或一个新版本的对象）

+   以及其他任何你可能合理地有多个值表示的情况

假设你有一个调用`dispense_snack`函数的代码，但只期望返回一个`HotDog`（或`None`）：

```py
from typing import Union
def place_order() -> Optional[HotDog]:
    order = get_order()
    result = dispense_snack(order.name)
    if result is None
        print_error_code("An error occurred" + result)
        return None
    # Return our HotDog
    return result
```

一旦`dispense_snack`开始返回`Pretzels`，这段代码就无法通过类型检查。

```py
code_examples/chapter4/invalid/union_hotdog.py:22:
    error: Incompatible return value type (got "Union[HotDog, Pretzel]",
                                           expected "Optional[HotDog]")
```

在这种情况下，类型检查器出错的事实是非常好的。如果您依赖的任何函数更改为返回新类型，则其返回签名必须更新为`Union`新类型，这将强制您更新代码以处理新类型。这意味着，当您的依赖项以与您的假设相矛盾的方式发生变化时，您的代码将被标记。通过您今天做出的决策，您可以在未来捕捉错误。这是健壮代码的标志；您使开发人员犯错的难度越来越大，从而降低了他们的错误率，减少了用户可能遇到的错误数量。

使用`Union`还有一个更根本的好处，但是要解释清楚，我需要教你一点*类型理论*，这是关于类型系统的一种数学分支。

## 产品和总和类型

`Union`很有用，因为它们有助于限制可表示的状态空间。 *可表示的状态空间* 是对象可以采用的所有可能组合的集合。

取这个`dataclass`：

```py
from dataclasses import dataclass
# If you aren't familiar with data classes, you'll learn more in chapter 10
# but for now, treat this as four fields grouped together and what types they are
@dataclass
class Snack:
    name: str
    condiments: set[str]
    error_code: int
    disposed_of: bool

Snack("Hotdog", {"Mustard", "Ketchup"}, 5, False)
```

我有一个名称，可以放在顶部的调味料，如果出了问题，还有一个错误代码，并且如果出了问题，有一个布尔值来跟踪我是否正确处理了该项。可以将多少种不同的值组合放入这个字典？可能是无限多个，对吧？`name`单独可以是任何有效值（“热狗”或“椒盐脆饼”）到无效值（“萨摩萨”、“泡菜”或“布丁”）甚至是荒谬的（“12345”、“”或“(╯°□°)╯︵ ┻━┻”）。 `condiments`也存在类似的问题。目前，无法计算可能的选项。

为了简单起见，我将人为地限制这种类型：

+   名称可以是三个值之一：热狗、椒盐脆饼或素食汉堡。

+   调味料可以是空的、芥末、番茄酱或两者兼有。

+   有六个错误代码（0-5）；0 表示成功。

+   `disposed_of`只能是`True`或`False`。

现在这组字段可以表示多少种不同的值？答案是 144，这是一个极其庞大的数字。我通过以下方式实现这一点：

> 名称有三种可能类型 × 调味品有四种可能类型 × 六个错误代码 × 两个布尔值用于记录条目是否已处理 = 3×4×6×2 = 144。

如果你接受这些值中的任何一个可以是`None`，则总数扩展到 420。虽然编码时应始终考虑`None`（请参阅本章前面关于`Optional`的内容），但对于这个思维实验，我将忽略`None`值。

这种操作被称为*乘类型*；可表示状态的数量由可能值的乘积决定。问题在于，并非所有这些状态都是有效的。如果将变量`disposed_of`设置为非零错误码，则应该将其设置为`True`。开发人员会做出这种假设，并相信非法状态永远不会出现。然而，一个无辜的错误可能导致整个系统崩溃。考虑以下代码：

```py
def serve(snack):
    # if something went wrong, return early
    if snack.disposed_of:
        return
    # ...
```

在这种情况下，开发人员在检查非零错误码之前检查`disposed_of`。这是一个等待发生的逻辑炸弹。只要`disposed_of`为`True` *且*错误码为非零，这段代码就完全正常。如果一个有效的小吃错误地将`disposed_of`标志设置为`True`，这段代码将开始产生无效的结果。这很难找到，因为创建小吃的开发人员没有理由检查此代码。目前为止，除了手动检查每个用例之外，您没有办法捕获这种错误，对于大型代码库来说是不可行的。通过允许可表示非法状态，您打开了脆弱代码的大门。

要解决这个问题，我需要使这个非法状态不可表示。为了做到这一点，我将重新调整我的示例，并使用`Union`：

```py
from dataclasses import dataclass
from typing import Union
@dataclass
class Error:
    error_code: int
    disposed_of: bool

@dataclass
class Snack:
    name: str
    condiments: set[str]

snack: Union[Snack, Error] = Snack("Hotdog", {"Mustard", "Ketchup"})

snack = Error(5, True)
```

在这种情况下，`snack`可以是一个`Snack`（仅包含`name`和`condiments`）或一个`Error`（仅包含一个数字和一个布尔值）。通过使用`Union`，现在有多少个可表示的状态呢？

对于`Snack`，有 3 个名称和 4 个可能的列表值，总共有 12 个可表示的状态。对于`ErrorCode`，我可以移除 0 错误码（因为那只是成功的情况），这给了我 5 个错误码的值和 2 个布尔值的总计 10 个可表示的状态。由于`Union`是一个选择一个或另一个的结构，我可以在一种情况下有 12 个可表示的状态，或者在另一种情况下有 10 个，总共 22 个。这是一个*和类型*的示例，因为我是将可表示状态的数量相加而不是相乘。

这是 22 个总可表示的状态。将其与将所有字段合并到单个实体时的 144 个状态进行比较。我将我的可表示状态空间减少了几乎 85%。我使得不兼容的字段无法混合和匹配。这样做变得更难出错，并且需要测试的组合数量大大减少。任何时候您使用和类型，例如`Union`，都会大幅减少可能的可表示状态数量。

# 字面类型

在计算可表示状态的数量时，我在上一节中做了一些假设。我限制了可能的值的数量，但这有点作弊，不是吗？正如我之前所说的，可能的值几乎是无限的。幸运的是，有一种方法可以通过 Python 限制这些值：`Literals`。`Literal`类型允许您将变量限制为非常特定的值集。

I’ll change my earlier `Snack` class to employ `Literal` values:

```py
from typing import Literal
@dataclass
class Error:
    error_code: Literal[1,2,3,4,5]
    disposed_of: bool

@dataclass
class Snack:
    name: Literal["Pretzel", "Hot Dog", "Veggie Burger"]
    condiments: set[Literal["Mustard", "Ketchup"]]
```

Now, if I try to instantiate these data classes with wrong values:

```py
Error(0, False)
Snack("Invalid", set())
Snack("Pretzel", {"Mustard", "Relish"})
```

I receive the following typechecker errors:

```py
code_examples/chapter4/invalid/literals.py:14: error: Argument 1 to "Error" has
    incompatible type "Literal[0]";
                      expected "Union[Literal[1], Literal[2], Literal[3],
                                      Literal[4], Literal[5]]"

code_examples/chapter4/invalid/literals.py:15: error: Argument 1 to "Snack" has
    incompatible type "Literal['Invalid']";
                       expected "Union[Literal['Pretzel'], Literal['Hotdog'],
                                       Literal['Veggie Burger']]"

code_examples/chapter4/invalid/literals.py:16: error: Argument 2 to <set> has
    incompatible type "Literal['Relish']";
                       expected "Union[Literal['Mustard'], Literal['Ketchup']]"
```

`Literals` were introduced in Python 3.8, and they are an invaluable way of restricting possible values of a variable. They are a little more lightweight than Python enumerations (which I’ll cover in Chapter 8).

# Annotated Types

What if I wanted to get even deeper and specify more complex constraints? It would be tedious to write hundreds of literals, and some constraints aren’t able to be modeled by `Literal` types. There’s no way with a `Literal` to constrain a string to a certain size or to match a specific regular expression. This is where `Annotated` comes in. With `Annotated`, you can specify arbitrary metadata alongside your type annotation.

```py
x: Annotated[int, ValueRange(3,5)]
y: Annotated[str, MatchesRegex('[0-9]{4}')]
```

Unfortunately, the above code will not run, as `ValueRange` and `MatchesRegex` are not built-in types; they are arbitrary expressions. You will need to write your own metadata as part of an `Annotated` variable. Secondly, there are no tools that will typecheck this for you. The best you can do until such a tool exists is write dummy annotations or use strings to describe your constraints. At this point, `Annotated` is best served as a communication method.

# NewType

While waiting for tooling to support `Annotated`, there is another way to represent more complicated constraints: `NewType`. `NewType` allows you to, well, create a new type.

Suppose I want to separate my hot dog stand code to handle two separate cases: a hot dog in its unservable form (no plate, no napkins) and a hot dog that is ready to serve (plated, with napkins). In my code, there exist some functions that should only be operating on the hot dog in one case or the other. For example, an unservable hot dog should never be dispensed to the customer.

```py
class HotDog:
    # ... snip hot dog class implementation ...

def dispense_to_customer(hot_dog: HotDog):
    # note, this should only accept ready-to-serve hot dogs.
    # ...
```

However, nothing prevents someone from passing in an unservable hot dog. If a developer makes a mistake and passes an unservable hot dog to this function, customers will be quite surprised to see just their order with no plate or napkins come out of the machine.

Rather than relying on developers to catch these errors whenever they happen, you need a way for your typechecker to catch this. To do that, you can use `NewType`:

```py
from typing import NewType

class HotDog:
    ''' Used to represent an unservable hot dog'''
    # ... snip hot dog class implementation ...

ReadyToServeHotDog = NewType("ReadyToServeHotDog", HotDog)

def dispense_to_customer(hot_dog: ReadyToServeHotDog):
    # ...
```

`NewType`接受现有类型并创建一个全新的类型，该类型具有与现有类型相同的所有字段和方法。在本例中，我正在创建一个名为`ReadyToServeHotDog`的类型，它与`HotDog`不同；它们不可互换。这种美妙之处在于，该类型限制了隐式类型转换。你不能在需要`ReadyToServeHotDog`的任何地方使用`HotDog`（尽管你可以用`ReadyToServeHotDog`替换`HotDog`）。在前面的例子中，我正在限制`dispense_to_customer`仅接受`ReadyToServeHotDog`值作为参数。这可以防止开发者无意间使假设失效。如果开发者试图向该方法传递一个`HotDog`，类型检查器会警告他们：

```py
code_examples/chapter4/invalid/newtype.py:10: error:
	Argument 1 to "dispense_to_customer"
	has incompatible type "HotDog";
	expected "ReadyToServeHotDog"
```

强调这种类型转换的单向性是很重要的。作为开发者，你可以控制旧类型何时成为新类型。

例如，我将创建一个函数，它接受一个无法服务的`HotDog`并使其准备好服务：

```py
def prepare_for_serving(hot_dog: HotDog) -> ReadyToServeHotDog:
    assert not hot_dog.is_plated(), "Hot dog should not already be plated"
    hot_dog.put_on_plate()
    hot_dog.add_napkins()
    return ReadyToServeHotDog(hot_dog)
```

注意我如何明确地返回一个`ReadyToServeHotDog`而不是普通的`HotDog`。这充当了一个“被认可”的函数；这是我希望开发者创建`ReadyToServeHotDog`的唯一合法方式。任何试图使用接受`ReadyToServeHotDog`的方法的用户都需要首先使用`prepare_for_serving`来创建它。

重要的是要通知用户，创建你的新类型的唯一方法是通过一组“被认可”的函数。你不希望用户在任何情况下都创建你的新类型，因为那样会失去意义。

```py
def make_snack():
    serve_to_customer(ReadyToServeHotDog(HotDog()))
```

不幸的是，Python 没有很好的方式来告诉用户这一点，除了通过注释。

```py
from typing import NewType
# NOTE: Only create ReadyToServeHotDog using prepare_for_serving method.
ReadyToServeHotDog = NewType("ReadyToServeHotDog", HotDog)
```

依然，`NewType`适用于许多现实世界的场景。例如，以下是我遇到过的`NewType`可以解决的所有场景：

+   将`str`与`SanitizedString`分离，以捕捉诸如 SQL 注入漏洞之类的错误。通过将`SanitizedString`设为`NewType`，我确保只有经过适当处理的字符串才能进行操作，消除了 SQL 注入的可能性。

+   分别跟踪`User`对象和`LoggedInUser`。通过将`Users`限制为`NewType`，我编写了仅适用于已登录用户的函数。

+   追踪一个应该代表有效用户 ID 的整数。通过将用户 ID 限制为`NewType`，我可以确保某些函数只操作有效的 ID，而无需检查`if`语句。

在第十章中，你将看到如何使用类和不变量做类似的事情，以更强的保证避免非法状态。然而，`NewType`仍然是一个值得关注的有用模式，比起完整的类要轻便得多。

# 最终类型

最后（意在说笑），你可能想要限制类型不要改变其值。这就是`Final`的用武之地。`Final`在 Python 3.8 中引入，指示类型检查器变量不能绑定到另一个值。例如，我想开始特许经营我的热狗摊，但我不希望名称因错误而改变。

```py
VENDOR_NAME: Final = "Viafore's Auto-Dog"
```

如果开发者不小心之后改变了名称，他们会看到一个错误。

```py
def display_vendor_information():
    vendor_info = "Auto-Dog v1.0"
    # whoops, copy-paste error, this code should be vendor_info += VENDOR_NAME
    VENDOR_NAME += VENDOR_NAME
    print(vendor_info)
```

```py
code_examples/chapter4/invalid/final.py:3: error:
	Cannot assign to final name "VENDOR_NAME"
Found 1 error in 1 file (checked 1 source file)
```

一般来说，`Final` 最好在变量的范围跨度较大时使用，比如一个模块。在这些情况下，让类型检查器捕获不可变性保证是一个福音，因为开发者很难跟踪变量在这些大范围内的所有用途。

###### 警告

`Final` 在通过函数改变对象时不会出错。它只是防止变量被重新绑定（设置为新值）。

# 总结思考

在本章中，你学到了许多约束类型的不同方法。它们每一个都有特定的目的，从使用`Optional`处理`None`到使用`Literal`限制特定值，再到使用`Final`防止变量重新绑定。通过使用这些技术，你可以直接将假设和限制编码到你的代码库中，避免未来的读者猜测你的逻辑。类型检查器将使用这些高级类型注释为你提供更严格的代码保证，这将使维护者在处理你的代码库时更加自信。有了这种信心，他们会犯更少的错误，你的代码库也因此变得更加健壮。

在下一章中，你将继续从为单个值添加类型注释，学习如何正确为集合类型添加注释。集合类型在大多数 Python 中无处不在；你必须小心地表达你的意图。你需要精通所有可以表示集合的方式，包括必须自己创建的情况。

¹ C.A.R. Hoare. “空引用：十亿美元的错误。” *历史上的糟糕想法*。2009 年在 Qcon London 上展示，无日期。
