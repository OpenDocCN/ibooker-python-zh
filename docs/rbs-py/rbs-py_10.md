# 第八章。用户定义类型：枚举

在本章中，我将专注于用户定义类型是什么，并涵盖最简单的用户定义数据类型：枚举。我将讨论如何创建一个能够防止常见编程错误的枚举。然后我会介绍一些高级特性，允许你更清晰地表达你的想法，比如创建别名、使枚举唯一，或提供自动生成的值。

# 用户定义类型

用户定义类型是你作为开发者创建的一种类型。你定义与类型关联的数据和行为。这些类型中的每一个都应该与一个独特的概念相关联。这将帮助其他开发人员在你的代码库中建立心智模型。

例如，如果我正在编写餐厅销售点系统，我期望在你的代码库中找到关于餐厅领域的概念。像餐厅、菜单项和税收计算这样的概念应该自然地在代码中表示出来。如果我使用列表、字典和元组，我会迫使读者不断重新解释变量的含义以符合其更自然的映射。

考虑一个简单的计算带税总额的函数。你宁愿使用哪个函数？

```py
def calculate_total_with_tax(restaurant: tuple[str, str, str, int],
                             subtotal: float) -> float:
    return subtotal * (1 + tax_lookup[restaurant[2]])
```

或

```py
def calculate_total_with_tax(restaurant: Restaurant,
                             subtotal: decimal.Decimal) -> decimal.Decimal:
    return subtotal * (1 + tax_lookup[restaurant.zip_code])
```

通过使用自定义类型`Restaurant`，你为读者提供了关于代码行为的重要知识。尽管它可能很简单，但构建这些领域概念是非常强大的。埃里克·埃文斯，《领域驱动设计》的作者写道：“软件的核心是解决用户的领域相关问题。”¹ 如果软件的核心是解决领域相关问题，那么领域特定的抽象就是血管。它们是支持系统，是流经你的代码库的网络，所有这些都与作为你的代码存在原因的中心生命赋予者紧密联系在一起。通过建立出色的领域相关类型，你建立了一个更健康的系统。

最可读的代码库是那些可以推理的代码库，而且最容易推理的是你日常遇到的概念。对于代码库的新手来说，如果他们熟悉核心业务概念，他们将已经占据了先机。你在本书的第一部分中专注于通过注释表达意图；接下来的部分将专注于通过建立共享词汇和使该词汇对每个在代码库中工作的开发者可用来传达意图。

你将学习如何将领域概念映射到类型的第一种方式是通过 Python 的枚举类型：`Enum`。

# 枚举

在某些情况下，你希望开发者从列表中选择一个值。交通灯的颜色、网络服务的定价计划和 HTTP 方法都是这种关系的绝佳例子。为了在 Python 中表达这种关系，你应该使用*枚举*。枚举是一种构造，让你定义值的列表，开发者可以选择他们想要的具体值。Python 在 Python 3.4 中首次支持了枚举。

为了说明列举的特殊之处，假设你正在开发一个应用程序，通过提供送货上门网络来使法国烹饪更加可访问，从长棍面包到甜甜圈。它提供了一个菜单，让饥饿的用户可以选择，然后通过邮件收到所有的配料和烹饪说明。

在这个应用程序中最受欢迎的服务之一是定制化。用户可以选择他们想要的肉类、配菜和调料来准备食物。法国烹饪最重要的部分之一是*母酱汁*。这五种广为人知的酱汁是无数其他酱汁的基础，我希望能够通过程序向其中添加新的成分，创造出所谓的*女儿酱汁*。这样，用户在点餐时就可以了解法国酱汁的分类。

假设我用 Python 元组表示母酱汁：

```py
# Note: use UPPER_CASE variable names to denote constant/immutable values
MOTHER_SAUCES = ("Béchamel", "Velouté", "Espagnole", "Tomato", "Hollandaise")
```

这个元组向其他开发者传达了什么信息？

+   这个集合是不可变的。

+   他们可以遍历这个集合来获取所有的酱汁。

+   他们可以通过静态索引检索特定的元素。

不可变性和检索属性对我的应用程序很重要。我不希望在运行时添加或删除任何母酱汁（这样做将是烹饪的亵渎）。使用元组清楚地告诉未来的开发者，他们不应该更改这些值。检索让我可以选择只有一个酱汁，尽管有点笨拙。每次我需要引用一个元素时，我可以通过静态索引来做到：

```py
MOTHER_SAUCES[2]
```

不幸的是，这并没有传达出意图。每次开发者看到这个，他们必须记住`2`代表着`"Espagnole"`。不断将数字与酱汁对应会浪费时间。这是脆弱的，必然会导致错误。如果有人对酱汁进行字母排序，索引将会改变，破坏代码。通过静态索引访问这个元组也不会增强代码的健壮性。

为了应对这个问题，我将为每一个添加别名：

```py
BÉCHAMEL = "Béchamel"
VELOUTÉ = "Velouté"
ESPAGNOLE = "Espagnole"
TOMATO = "Tomato"
HOLLANDAISE = "Hollandaise"
MOTHER_SAUCES = (BÉCHAMEL, VELOUTÉ, ESPAGNOLE, TOMATO, HOLLANDAISE)
```

这是更多的代码，但仍然不能使索引到这个元组变得更容易。此外，调用代码仍然存在一个潜在的问题。

考虑一个创建女儿酱汁的函数：

```py
def create_daughter_sauce(mother_sauce: str,
                          extra_ingredients: list[str]):
    # ...
```

我希望你停顿一下，考虑这个函数告诉未来开发者什么。我特意省略了实现部分，因为我想讨论的是第一印象；函数签名是开发者看到的第一件事。仅仅基于函数签名，这个函数是否适当地传达了允许什么？

未来的开发者可能会遇到类似这样的代码：

```py
create_daughter_sauce(MOTHER_SAUCES[0], ["Onions"]) # not super helpful
create_daughter_sauce(BÉCHAMEL, ["Onions"]) # Better
```

或者：

```py
create_daughter_sauce("Hollandaise", ["Horseradish"])
create_daughter_sauce("Veloute", ["Mustard"])

# Definitely wrong
create_daughter_sauce("Alabama White BBQ Sauce", [])
```

这就是问题的关键所在。在正常情况下，开发人员可以使用预定义的变量。但是，如果有人意外地使用了错误的酱料（毕竟，`create_daughter_sauce`期望一个字符串，它可以是任何东西），很快就会出现意外的行为。请记住，我说的是几个月甚至几年后的开发人员。他们被要求向代码库添加一个功能，尽管他们对其不熟悉。通过选择字符串类型，我只是在邀请以后提供错误的值。

###### 警告

即使是诚实的错误也会产生后果。你有没有注意到我在`Velouté`的“e”上漏掉了重音符号？在生产环境中调试这个问题会很有趣。

相反，你希望找到一种方式来传达你希望在特定位置使用非常具体和受限制的一组值。既然你现在在“枚举”章节中，而我还没有展示它们，我相信你可以猜到解决方案是什么。

## 枚举

下面是 Python 枚举`Enum`的示例：

```py
from enum import Enum
class MotherSauce(Enum):
    BÉCHAMEL = "Béchamel"
    VELOUTÉ = "Velouté"
    ESPAGNOLE = "Espagnole"
    TOMATO = "Tomato"
    HOLLANDAISE = "Hollandaise"
```

要访问特定的实例，你只需：

```py
MotherSauce.BÉCHAMEL
MotherSauce.HOLLANDAISE
```

这与字符串别名几乎相同，但有一些额外的好处。

你不会意外创建`MotherSauce`并获得意外值：

```py
>>>MotherSauce("Hollandaise") # OKAY

>>>MotherSauce("Alabama White BBQ Sauce")
ValueError: 'Alabama White BBQ Sauce' is not a valid MotherSauce
```

这肯定会限制错误（无论是无效的酱料还是无辜的拼写错误）。

如果你想打印出枚举的所有值，你可以简单地迭代枚举（无需创建单独的列表）。

```py
>>>for option_number, sauce in enumerate(MotherSauce, start=1):
>>>    print(f"Option {option_number}: {sauce.value}")

Option 1: Béchamel
Option 2: Velouté
Option 3: Espagnole
Option 4: Tomato
Option 5: Hollandaise
```

最后但至关重要的是，你可以在使用这个`Enum`的函数中传达你的意图：

```py
def create_daughter_sauce(mother_sauce: MotherSauce,
                          extra_ingredients: list[str]):
    # ...
```

这告诉所有查看此函数的开发人员，他们应该传入一个`MotherSauce`枚举，而不仅仅是任意的字符串。这样一来，要引入拼写错误或不正确的值就变得更加困难。（用户仍然可以传递错误的值，如果他们真的想这样做，但这将直接违反预期的行为，这更容易捕捉——我在第一部分中讲解了如何捕捉这些错误。）

# 讨论主题

你的代码库中哪些数据集会受益于`Enum`？你有没有开发人员在尽管类型正确但传入了错误的值的代码区域？讨论一下枚举如何改进你的代码库。

## 不适用时机

枚举类型非常适合用于向用户传达静态选择集。如果选项在运行时确定，就不应使用枚举，因为这样会失去它们在传达意图和工具方面的许多优势（如果每次运行都可以更改，代码阅读者很难知道可能的值是什么）。如果你发现自己处于这种情况下，我建议使用字典，它提供了两个值之间的自然映射，可以在运行时更改。但如果你需要限制用户可以选择的值，你将需要执行成员资格检查。

# 高级用法

一旦掌握了枚举的基础知识，您可以做很多事情来进一步完善您的使用。记住，您选择的类型越具体，传达的信息就越具体。

## 自动值

对于某些枚举，您可能希望明确指定您不关心枚举所关联的值。这告诉用户他们不应依赖这些值。为此，您可以使用`auto()`函数。

```py
from enum import auto, Enum
class MotherSauce(Enum):
    BÉCHAMEL = auto()
    VELOUTÉ = auto()
    ESPAGNOLE = auto()
    TOMATO = auto()
    HOLLANDAISE = auto()

>>>list(MotherSauce)
[<MotherSauce.BÉCHAMEL: 1>, <MotherSauce.VELOUTÉ: 2>, <MotherSauce.ESPAGNOLE: 3>,
 <MotherSauce.TOMATO: 4>, <MotherSauce.HOLLANDAISE: 5>]
```

默认情况下，`auto()`将选择单调递增的值（1、2、3、4、5...）。如果您想控制设置的值，您应该实现一个`_generate_next_value_()`函数：

```py
from enum import auto, Enum
class MotherSauce(Enum):
    def _generate_next_value_(name, start, count, last_values):
        return name.capitalize()
    BÉCHAMEL = auto()
    VELOUTÉ = auto()
    ESPAGNOLE = auto()
    TOMATO = auto()
    HOLLANDAISE = auto()

>>>list(MotherSauce)
[<MotherSauce.BÉCHAMEL: 'Béchamel'>, <MotherSauce.VELOUTÉ: 'Velouté'>,
 <MotherSauce.ESPAGNOLE: 'Espagnole'>, <MotherSauce.TOMATO: 'Tomato'>,
 <MotherSauce.HOLLANDAISE: 'Hollandaise'>]
```

很少会看到`_generate_next_value_`像这样定义，直接在具有值的枚举内部。如果`auto`用于指示值无关紧要，那么`_generate_next_value_`指示您希望`auto`的非常具体的值。这感觉矛盾。这就是为什么通常在基本`Enum`类中使用`_generate_next_value_`，这些枚举意味着要被子类型化，并且不包括任何值。接下来您将看到的`Flag`类就是一个很好的基类示例。

## 标志

现在您已经用`Enum`表示了主酱，您决定准备开始用这些酱料来供应餐点。但在您开始之前，您希望意识到顾客的过敏反应，因此您决定为每道菜设置过敏原信息。有了您对`auto()`的新知识，设置`Allergen`枚举就像小菜一碟：

```py
from enum import auto, Enum
from typing import Set
class Allergen(Enum):
    FISH = auto()
    SHELLFISH = auto()
    TREE_NUTS = auto()
    PEANUTS = auto()
    GLUTEN = auto()
    SOY = auto()
    DAIRY = auto()
```

对于食谱，您可能会像这样跟踪一组过敏原：

```py
allergens: Set[Allergen] = {Allergen.FISH, Allergen.SOY}
```

这告诉读者过敏原的集合将是唯一的，并且可能有零个、一个或多个过敏原。这正是您想要的。但如果我希望系统中的所有过敏原信息都像这样被跟踪呢？我不希望依赖每个开发人员记住使用集合（仅使用列表或字典的一种用法可能会引发错误行为）。我希望有某种方式通用地表示一组唯一的枚举值的集合。

`enum`模块为您提供了一个便利的基类可供使用——`Flag`：

```py
from enum import Flag
class Allergen(Flag):
    FISH = auto()
    SHELLFISH = auto()
    TREE_NUTS = auto()
    PEANUTS = auto()
    GLUTEN = auto()
    SOY = auto()
    DAIRY = auto()
```

这允许您执行位操作来组合过敏原或检查特定过敏原是否存在。

```py
>>>allergens = Allergen.FISH | Allergen.SHELLFISH
>>>allergens
<Allergen.SHELLFISH|FISH: 3>

>>>if allergens & Allergen.FISH:
>>>    print("This recipe contains fish.")
This recipe contains fish.
```

当您希望表示一组值（比如通过多选下拉或位掩码设置的值）时，这非常有用。但也有一些限制。这些值必须支持位操作（|、&等）。字符串将是不支持的类型的示例，而整数则支持。此外，在进行位操作时，这些值不能重叠。例如，您不能使用 1 到 4（包括）的值用于您的`Enum`，因为 4 会对 1、2 和 4 的值进行“位与”操作，这可能不是您想要的。`auto()`会为您处理这些问题，因为`Flag`的`_generate_next_value_`自动使用 2 的幂。

```py
class Allergen(Flag):
    FISH = auto()
    SHELLFISH = auto()
    TREE_NUTS = auto()
    PEANUTS = auto()
    GLUTEN = auto()
    SOY = auto()
    DAIRY = auto()
    SEAFOOD = Allergen.FISH | Allergen.SHELLFISH
    ALL_NUTS = Allergen.TREE_NUTS | Allergen.PEANUTS
```

使用标志可以在非常具体的情况下表达你的意图，但如果你希望更好地控制你的值，或者在列举不支持按位操作的值时，请使用非标志的`Enum`。

最后要注意的是，你可以自由地为内置的多个枚举选择创建自己的别名，就像我上面为`SEAFOOD`和`ALL_NUTS`所做的那样。

## 整数转换

还有两个特殊情况的枚举称为`IntEnum`和`IntFlag`。它们分别映射到`Enum`和`Flag`，但允许降级到原始整数进行比较。实际上，我不推荐使用这些特性，并且了解其原因是非常重要的。首先，让我们看看它们想解决的问题。

在法国烹饪中，某些成分的测量对成功至关重要，因此您需要确保这一点得到了覆盖。您创建了米制和英制液体测量（毕竟您希望国际化工作）作为枚举，但却发现您无法将您的枚举值简单地与整数进行比较。

这段代码不起作用：

```py
class ImperialLiquidMeasure(Enum):
    CUP = 8
    PINT = 16
    QUART = 32
    GALLON = 128

>>>ImperialLiquidMeasure.CUP == 8
False
```

但是，如果你从`IntEnum`派生子类，它就可以正常工作：

```py
class ImperialLiquidMeasure(IntEnum):
    CUP = 8
    PINT = 16
    QUART = 32
    GALLON = 128

>>>ImperialLiquidMeasure.CUP == 8
True
```

`IntFlag`的表现类似。当在系统之间或可能是硬件之间进行互操作时，您会更频繁地看到这一点。如果没有使用`IntEnum`，你可能需要做类似以下的事情：

```py
>>>ImperialLiquidMeasure.CUP.value == 8
True
```

使用`IntEnum`的便利性往往不超过它作为一种较弱类型所带来的弊端。任何对整数的隐式转换都隐藏了类的真正意图。由于隐式整数转换的发生，您可能会在不想要的情况下遇到复制/粘贴错误（我们都曾经犯过这种错误，对吧？）。

考虑：

```py
class Kitchenware(IntEnum):
    # Note to future programmers: these numbers are customer-defined
    # and apt to change
    PLATE = 7
    CUP = 8
    UTENSILS = 9
```

假设有人错误地执行了以下操作：

```py
def pour_liquid(volume: ImperialLiquidMeasure):
    if volume == Kitchenware.CUP:
        pour_into_smaller_vessel()
    else:
        pour_into_larger_vessel()
```

如果这段代码进入生产环境，一切都会很顺利，没有异常抛出，所有测试都通过了。然而，一旦`Kitchenware`枚举发生变化（也许将一个`BOWL`添加到值`8`，并将`CUP`移动到`10`），这段代码现在会做完全相反于预期的事情。`Kitchenware.CUP`不再与`ImperialLiquidMeasure.CUP`相同（它们没有理由关联）；然后你将开始向更大的容器倾倒，这可能会造成溢出（液体的溢出，而不是整数的溢出）。

这是一个关于不够健壮的代码如何导致微妙错误的教科书案例，这些错误直到代码库的后期才会成为问题。这可能是一个快速修复，但这个 bug 却带来了很大的实际成本。测试失败（或更糟的是，客户抱怨将错误的液体量倒入容器），有人必须爬行源代码，找到 bug，修复它，然后在思考这为何之前如此工作后，长时间的咖啡休息。所有这些都是因为有人决定懒惰地使用`IntEnum`，这样他们就不必一遍又一遍地输入`.value`。所以请为你未来的维护者着想：除非出于遗留目的，否则不要使用`IntEnum`。

## 独特的

枚举的一个很棒的特性是能够给值取别名。让我们回到`MotherSauce`枚举。也许在法国键盘开发的代码库需要适应美国键盘，其中键盘布局不利于在元音字母上添加重音符号。删除重音以使法语原生拼写变得类似英语是许多开发人员不愿意接受的（他们坚持使用原始拼写）。为了避免国际事件，我将为一些酱汁添加别名。

```py
from enum import Enum
class MotherSauce(Enum):
    BÉCHAMEL = "Béchamel"
    BECHAMEL = "Béchamel"
    VELOUTÉ = "Velouté"
    VELOUTE = "Velouté"
    ESPAGNOLE = "Espagnole"
    TOMATO = "Tomato"
    HOLLANDAISE = "Hollandaise"
```

有关此事，所有键盘所有者都欢欣鼓舞。枚举绝对允许这种行为；它们可以具有重复值，只要键不重复即可。

然而，有些情况下，您可能希望对值强制唯一性。也许您依赖枚举始终包含一组固定数量的值，或者可能会影响向客户显示的某些字符串表示。无论情况如何，如果要在您的`Enum`中保持唯一性，只需添加一个`@unique`装饰器。

```py
from enum import Enum, unique
@unique
class MotherSauce(Enum):
    BÉCHAMEL = "Béchamel"
    VELOUTÉ = "Velouté"
    ESPAGNOLE = "Espagnole"
    TOMATO = "Tomato"
    HOLLANDAISE = "Hollandaise"
```

在大多数我遇到的用例中，创建别名比保持唯一性更有可能，因此我默认首先使枚举非唯一，并仅在需要时添加唯一装饰器。

# 总结思考

枚举很简单，通常被忽视作为一种强大的通信方法。每当您想要从静态值集合中表示单个值时，枚举应该是您首选的用户定义类型。定义和使用它们都很容易。它们提供了丰富的操作，包括迭代，在位操作中（在`Flag`枚举的情况下），以及对唯一性的控制。

记住这些关键限制：

+   枚举不适用于在运行时动态更改的键值映射。使用字典来实现此功能。

+   `Flag`枚举仅适用于支持与非重叠值进行位操作的值。

+   避免除非绝对必要与系统互操作性，否则使用`IntEnum`和`IntFlag`。

接下来，我将探索另一种用户定义类型：一个`dataclass`。虽然枚举在只需一个变量就能指定一组值的关系方面非常出色，但数据类定义了多个变量之间的关系。

¹ Eric Evans. *领域驱动设计：软件核心复杂性的应对*. Upper Saddle River, NJ: Addison-Wesley Professional, 2003.
