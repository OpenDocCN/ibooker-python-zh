# 第十四章：使用 pydantic 进行运行时检查

健壮代码的核心主题是使错误检测变得更加容易。错误是开发复杂系统不可避免的一部分；你无法避免它们。通过编写自己的类型，您创建了一个词汇表，使引入不一致性变得更加困难。使用类型注解为您提供了一个安全网，让您在开发过程中捕获错误。这两者都是“将错误左移”的示例；而不是在测试期间（或更糟的是，在生产中）找到错误，您可以更早地找到它们，理想情况下是在开发代码时。

然而，并非每个错误都能通过代码检查和静态分析轻松发现。有一整类错误只能在运行时检测到。每当与程序外部提供的数据交互时（例如数据库、配置文件、网络请求），您都会面临输入无效数据的风险。您的代码在检索和解析数据方面可能非常可靠，但是您无法阻止用户传递无效数据。

您的第一个倾向可能是编写大量*验证逻辑*：`if`语句和检查以确保所有传入的数据都是正确的。问题在于验证逻辑通常是复杂的、扩展的，一眼看去很难理解。验证越全面，情况越糟。如果您的目标是查找错误，阅读所有代码（和测试）将是您最好的选择。在这种情况下，您需要最小化查看的代码量。这就是难题所在：您阅读的代码越多，您理解的越多，但阅读的越多，认知负担就越大，降低您找到错误的机会。

在本章中，您将学习如何使用 pydantic 库解决此问题。pydantic 允许您定义建模类，减少需要编写的验证逻辑量，而不会牺牲可读性。pydantic 可以轻松解析用户提供的数据，并为输出数据结构提供保证。我将介绍一些基本的使用示例，然后以一些高级 pydantic 用法结束本章。

# 动态配置

在本章中，我将构建描述餐馆的类型。我将首先提供一种通过配置文件指定餐馆的方法。以下是每家餐馆可配置字段（及其约束）的列表：

+   餐馆名称

    +   基于传统原因，名称必须少于 32 个字符，并且只能包含字母、数字、引号和空格（抱歉，没有 Unicode）。

+   业主全名

+   地址

+   员工列表

    +   必须至少有一位厨师和一位服务员。

    +   每位员工都有姓名和职位（厨师、服务员、主持人、副厨师或送货司机）。

    +   每个员工都有邮寄地址用于支票或直接存款详细信息。

+   菜品列表

    +   每道菜品都有名称、价格和描述。名称限制在 16 个字符以内，描述限制在 80 个字符以内。可选地，每道菜还带有图片（以文件名的形式）。

    +   每道菜品必须有唯一的名称。

    +   菜单上至少要有三道菜。

+   座位数

+   提供外卖订单（布尔值）

+   提供送货服务（布尔值）

这些信息存储在一个[YAML 文件](https://yaml.org)，看起来像这样：

```py
name: Viafore's
owner: Pat Viafore
address: 123 Fake St. Fakington, FA 01234
employees:
  - name: Pat Viafore
    position: Chef
    payment_details:
      bank_details:
        routing_number: "123456789"
        account_number: "123456789012"
  - name: Made-up McGee
    position: Server
    payment_details:
      bank_details:
        routing_number: "123456789"
        account_number: "123456789012"
  - name: Fabricated Frank
    position: Sous Chef
    payment_details:
      bank_details:
        routing_number: "123456789"
        account_number: "123456789012"
  - name: Illusory Ilsa
    position: Host
    payment_details:
      bank_details:
        routing_number: "123456789"
        account_number: "123456789012"
dishes:
  - name: Pasta and Sausage
    price_in_cents: 1295
    description: Rigatoni and sausage with a tomato-garlic-basil sauce
  - name: Pasta Bolognese
    price_in_cents: 1495
    description: Spaghetti with a rich tomato and beef Sauce
  - name: Caprese Salad
    price_in_cents: 795
    description: Tomato, buffalo mozzarella, and basil
    picture: caprese.png
number_of_seats: 12
to_go: true
delivery: false
```

可以通过可安装的 Python 库`yaml`轻松读取此文件，返回一个字典：

```py
with open('code_examples/chapter14/restaurant.yaml') as yaml_file:
    restaurant = yaml.safe_load(yaml_file)

print(restaurant)
>>> {
    "name": "Viafore's",
    "owner": "Pat Viafore",
    "address": "123 Fake St. Fakington, FA 01234",
    "employees": [{
        "name": "Pat Viafore",
        "position": "Chef",
        "payment_details": {
            "bank_details": {
                "routing_number": '123456789',
                "account_number": '123456789012'
            }
        }
    },
    {
        "name": "Made-up McGee",
        "position": "Server",
        "payment_details": {
            "bank_details": {
                "routing_number": '123456789',
                "account_number": '123456789012'
            }
        }
    },
    {
        "name": "Fabricated Frank",
        "position": "Sous Chef",
        "payment_details": {
            "bank_details": {
                "routing_number": '123456789',
                "account_number": '123456789012'
            }
        }
    },
    {
        "name": "Illusory Ilsa",
        "position": "Host",
        "payment_details": {
            "bank_details": {
                "routing_number": '123456789',
                "account_number": '123456789012'
            }
        }
    }],
    "dishes": [{
        "name": "Pasta and Sausage",
        "price_in_cents": 1295,
        "description": "Rigatoni and sausage with a tomato-garlic-basil sauce"
    },
    {
        "name": "Pasta Bolognese",
        "price_in_cents": 1495,
        "description": "Spaghetti with a rich tomato and beef Sauce"
    },
    {
        "name": "Caprese Salad",
        "price_in_cents": 795,
        "description": "Tomato, buffalo mozzarella, and basil",
        "picture": "caprese.png"
    }],
    'number_of_seats': 12,
    "to_go": True,
    "delivery": False
}
```

我希望您可以戴上测试帽子。我刚刚给出的要求显然并不详尽；您会如何完善它们？请花几分钟时间列出您认为的所有不同约束条件，只需使用给定的字典假设 YAML 文件解析并返回一个字典，您能想到多少个无效的测试用例？

###### 注意

您可能会注意到上面示例中的路由号和账号都是字符串。这是有意为之。尽管是数字串，我不希望它成为数值类型。数值运算（比如加法或乘法）毫无意义，我也不希望账号 000000001234 被截断为 1234。

这里有一些思考枚举测试用例时的想法：

+   Python 是一种动态语言。您确定每个东西都是正确的类型吗？

+   字典不需要任何必填字段——您确定每个字段都存在吗？

+   所有问题陈述中的约束条件都被测试过了吗？

+   额外的约束条件如何（正确的路由号、账号和地址）？

+   在不应存在的地方使用负数怎么办？

我花了大约五分钟时间，想出了 67 种不同的带有无效数据的测试用例。我的一些测试用例包括（完整列表包含在[此书的 GitHub 存储库](https://github.com/pviafore/RobustPython)中）：

+   名称长度为零字符。

+   名称不是字符串。

+   没有厨师。

+   员工没有银行详细信息或地址。

+   员工的路由号被截断（0000123 变成 123）。

+   座位数为负数。

诚然，这不是一个非常复杂的类。您能想象一个更复杂的类需要多少测试用例吗？即使有 67 个测试用例，您能想象打开一个类型的构造函数并检查 67 个不同的条件吗？在我工作过的大多数代码库中，验证逻辑远不如此详尽。然而，这是用户可配置的数据，我希望在运行时尽早捕获错误。您应该优先在数据注入时捕获错误，而不是在首次使用时。毕竟，这些值的首次使用可能发生在您与解析逻辑分离的另一个系统中。

# 讨论主题

思考一下在你的系统中以数据类型表示的某些用户数据。这些数据有多复杂？有多少种方法可以构造出错误的数据？讨论创建这些数据错误的影响以及你对代码能否捕获所有错误的信心。

在本章中，我将向你展示如何创建一个易于阅读并模拟所有列出约束条件的类型。由于我非常关注类型注解，如果在类型检查时可以捕获缺少字段或错误类型，那将是很好的。首先的想法是使用 `TypedDict`（更多关于 `TypedDict` 的信息请参见第五章）：

```py
from typing import Literal, TypedDict, Union
class AccountAndRoutingNumber(TypedDict):
    account_number: str
    routing_number: str

class BankDetails(TypedDict):
    bank_details: AccountAndRoutingNumber

AddressOrBankDetails = Union[str, BankDetails]

Position = Literal['Chef', 'Sous Chef', 'Host',
                   'Server', 'Delivery Driver']

class Dish(TypedDict):
    name: str
    price_in_cents: int
    description: str

class DishWithOptionalPicture(Dish, TypedDict, total=False):
    picture: str

class Employee(TypedDict):
    name: str
    position: Position
    payment_information: AddressOrBankDetails

class Restaurant(TypedDict):
    name: str
    owner: str
    address: str
    employees: list[Employee]
    dishes: list[Dish]
    number_of_seats: int
    to_go: bool
    delivery: bool
```

这对于提高可读性是一个巨大的进步；你可以准确地知道构建你的类型所需的类型。你可以编写以下函数：

```py
def load_restaurant(filename: str) -> Restaurant:
    with open(filename) as yaml_file:
        return yaml.safe_load(yaml_file)
```

下游消费者将自动从我刚刚建立的类型中受益。然而，这种方法存在一些问题：

+   我无法控制 `TypedDict` 的构建，因此无法在类型构建的过程中验证任何字段。我必须强制消费者进行验证。

+   `TypedDict` 不能在其上有额外的方法。

+   `TypedDict` 隐式地不进行验证。如果你从 YAML 中创建了错误的字典，类型检查器不会抱怨。

最后一点很重要。事实上，我可以将以下内容作为我的 YAML 文件的全部内容，代码仍将进行类型检查：

```py
invalid_name: "This is the wrong file format"
```

类型检查不会在运行时捕获错误。你需要更强大的工具。这时候就需要 pydantic 出马了。

# pydantic

[*pydantic*](https://pydantic-docs.helpmanual.io) 是一个提供运行时类型检查且不牺牲可读性的库。你可以像这样使用 pydantic 来建模你的类：

```py
from pydantic.dataclasses import dataclass
from typing import Literal, Optional, TypedDict, Union

@dataclass
class AccountAndRoutingNumber:
    account_number: str
    routing_number: str

@dataclass
class BankDetails:
    bank_details: AccountAndRoutingNumber

AddressOrBankDetails = Union[str, BankDetails]

Position = Literal['Chef', 'Sous Chef', 'Host',
                   'Server', 'Delivery Driver']

@dataclass
class Dish:
    name: str
    price_in_cents: int
    description: str
    picture: Optional[str] = None

@dataclass
class Employee:
    name: str
    position: Position
    payment_information: AddressOrBankDetails

@dataclass
class Restaurant:
    name: str
    owner: str
    address: str
    employees: list[Employee]
    dishes: list[Dish]
    number_of_seats: int
    to_go: bool
    delivery: bool
```

你使用 `pydantic.dataclasses.dataclass` 装饰每个类，而不是继承自 `TypedDict`。一旦你这样做了，pydantic 将在类型构建时进行验证。

要构建 pydantic 类型，我将按以下方式更改我的加载函数：

```py
def load_restaurant(filename: str) -> Restaurant:
    with open(filename) as yaml_file:
        data = yaml.safe_load(yaml_file)
        return Restaurant(**data)
```

如果将来的开发者违反了任何约束条件，pydantic 将抛出异常。以下是一些示例异常：

如果缺少字段，比如缺少描述：

```py
pydantic.error_wrappers.ValidationError: 1 validation error for Restaurant
dishes -> 2
  __init__() missing 1 required positional argument:
    'description' (type=type_error)
```

当提供无效类型时，例如将数字 3 作为员工职位：

```py
pydantic.error_wrappers.ValidationError: 1 validation error for Restaurant
employees -> 0 -> position
  unexpected value; permitted: 'Chef', 'Sous Chef', 'Host',
                               'Server', 'Delivery Driver'
                               (type=value_error.const; given=3;
                                permitted=('Chef', 'Sous Chef', 'Host',
                                           'Server', 'Delivery Driver'))
```

###### 警告

Pydantic 可以与 mypy 配合使用，但你可能需要在 *mypy.ini* 中启用 pydantic 插件以利用所有功能。你的 *mypy.ini* 需要包含以下内容：

```py
[mypy]
plugins = pydantic.mypy
```

欲了解更多信息，请查阅 [pydantic 文档](https://oreil.ly/FBQXX)。

通过使用 pydantic 建模类型，我可以在不编写自己的验证逻辑的情况下捕获整类错误。上述的 pydantic 数据类能够捕获我之前提出的 67 个测试用例中的 38 个。但我可以做得更好。这段代码仍然缺少对其他 29 个测试用例的功能，但我可以使用 pydantic 的内置验证器在类型构建时捕获更多错误。

## 验证器

Pydantic 提供了大量内置的*验证器*。验证器是检查特定约束条件的自定义类型。例如，如果我想确保字符串的大小或所有整数都是正数，我可以使用 pydantic 的约束类型：

```py
from typing import Optional

from pydantic.dataclasses import dataclass
from pydantic import constr, PositiveInt

@dataclass
class AccountAndRoutingNumber:
    account_number: constr(min_length=9,max_length=9) ![1](img/00002.gif)
    routing_number: constr(min_length=8,max_length=12)

@dataclass
class Address:
    address: constr(min_length=1)

# ... snip ...

@dataclass
class Dish:
    name: constr(min_length=1, max_length=16)
    price_in_cents: PositiveInt
    description: constr(min_length=1, max_length=80)
    picture: Optional[str] = None

@dataclass
class Restaurant:
    name: constr(regex=r'^[a-zA-Z0-9 ]*$', ![2](img/00005.gif)
                  min_length=1, max_length=16)
    owner: constr(min_length=1)
    address: constr(min_length=1)
    employees: List[Employee]
    dishes: List[Dish]
    number_of_seats: PositiveInt
    to_go: bool
    delivery: bool
```

![1](img/part0018_split_004.html#co_runtime_checking_with_pydantic_CO1-1)

我正在限制一个字符串的长度。

![2](img/part0018_split_004.html#co_runtime_checking_with_pydantic_CO1-2)

我正在限制一个字符串以匹配一个正则表达式（在本例中，只包括字母数字字符和空格）。

如果我传入一个无效类型（例如带有特殊字符的餐馆名称或负座位数），我将收到以下错误：

```py
pydantic.error_wrappers.ValidationError: 2 validation errors for Restaurant
name
  string does not match regex "^[a-zA-Z0-9 ]$" (type=value_error.str.regex;
                                                pattern=^[a-zA-Z0-9 ]$)
number_of_seats
  ensure this value is greater than 0
    (type=value_error.number.not_gt; limit_value=0)
```

我甚至可以约束列表以强制执行进一步的限制。

```py
from pydantic import conlist,constr
@dataclass
class Restaurant:
    name: constr(regex=r'^[a-zA-Z0-9 ]*$',
                   min_length=1, max_length=16)
    owner: constr(min_length=1)
    address: constr(min_length=1)
    employees: conlist(Employee, min_items=2) ![1](img/00002.gif)
    dishes: conlist(Dish, min_items=3) ![2](img/00005.gif)
    number_of_seats: PositiveInt
    to_go: bool
    delivery: bool
```

![1](img/part0018_split_004.html#co_runtime_checking_with_pydantic_CO2-1)

此列表仅限于`Employee`类型，必须至少有两名员工。

![2](img/part0018_split_004.html#co_runtime_checking_with_pydantic_CO2-2)

此列表仅限于`Dish`类型，必须至少有三种菜肴。

如果我传入不符合这些约束的内容（例如忘记一个菜品）：

```py
pydantic.error_wrappers.ValidationError: 1 validation error for Restaurant
dishes
  ensure this value has at least 3 items
    (type=value_error.list.min_items; limit_value=3)
```

在受限类型下，我额外捕获了我之前设想的 17 个测试用例，总数达到了 67 个测试用例中的 55 个。相当不错，不是吗？

为了捕捉剩余的错误集，我可以使用自定义验证器来嵌入那些最后的验证逻辑：

```py
from pydantic import validator
@dataclass
class Restaurant:
    name: constr(regex=r'^[a-zA-Z0-9 ]*$',
                   min_length=1, max_length=16)
    owner: constr(min_length=1)
    address: constr(min_length=1)
    employees: conlist(Employee, min_items=2)
    dishes: conlist(Dish, min_items=3)
    number_of_seats: PositiveInt
    to_go: bool
    delivery: bool

    @validator('employees')
    def check_chef_and_server(cls, employees):
        if (any(e for e in employees if e.position == 'Chef') and
            any(e for e in employees if e.position == 'Server')):
                return employees
        raise ValueError('Must have at least one chef and one server')
```

如果接着没有提供至少一位厨师和服务员：

```py
pydantic.error_wrappers.ValidationError: 1 validation error for Restaurant
employees
  Must have at least one chef and one server (type=value_error)
```

我将让你为其他错误情况编写自定义验证器（例如有效地址、有效路由号码或存在于文件系统上的有效图像）。

## 验证与解析的区别

诚然，pydantic 并非严格的验证库，而是一个*parsing*库。这两者的区别细微，但需要明确。在我所有的示例中，我一直在使用 pydantic 来检查参数和类型，但它并不是一个严格的验证器。Pydantic 宣传自己是一个*parsing*库，这意味着它提供了一个保证，即从数据模型中得到的内容是什么，而不是输入的内容。也就是说，当你定义 pydantic 模型时，pydantic 将尽其所能将数据强制转换为你定义的类型。

如果你有一个模型：

```py
from pydantic import dataclass
@dataclass
class Model:
    value: int
```

将字符串或浮点数传递给此模型没有问题；pydantic 将尽最大努力将该值强制转换为整数（或者如果该值不可强制转换，则抛出异常）。此代码不会抛出异常：

```py
Model(value="123") # value is set to the integer 123
Model(value=5.5) # this truncates the value to 5
```

Pydantic 正在解析这些值，而不是验证它们。你不能保证将整数传递给模型，但你始终可以保证另一端输出一个`int`（或者抛出异常）。

如果你希望限制这种行为，可以使用 pydantic 的严格字段：

```py
from pydantic.dataclasses import dataclass
from pydantic import StrictInt
@dataclass
class Model:
    value: StrictInt
```

现在，当从另一种类型构建时，

```py
x = Model(value="0023").value
```

你会得到一个错误：

```py
pydantic.error_wrappers.ValidationError: 1 validation error for Model
value
  value is not a valid integer (type=type_error.integer)
```

因此，虽然 pydantic 自称为一个解析库，但在你的数据模型中可以强制执行更严格的行为。

# 总结思考

在整本书中，我一直强调类型检查器的重要性，但这并不意味着在运行时捕获错误毫无意义。虽然类型检查器可以捕获大部分错误并减少运行时检查，但它们无法捕获所有问题。你仍然需要验证逻辑来填补这些空白。

对于这类检查，pydantic 库是你工具箱中的一个好工具。通过将验证逻辑直接嵌入到你的类型定义中（而不是编写大量乏味的`if`语句），你提升了健壮性。首先，大大提高了可读性；阅读你的类型定义的开发人员将清楚地知道施加在其上的约束。其次，它为你提供了那个急需的具备运行时检查的保护层。

我发现 pydantic 还有助于填补数据类和类之间的中间地带。每个约束技术上都在维护关于该类的不变量。我通常建议不要为你的数据类设置不变量，因为你无法保护它；你不能控制构造和属性访问是公开的。然而，pydantic 在你调用构造函数或设置字段时仍然保护不变量。但是，如果你有相互依赖的字段（例如需要同时设置两个字段或根据另一个字段的值设置一个字段），那么就坚持使用类。

第二部分到此为止。你已经学会了如何使用`Enums`、数据类和类来创建自己的类型。每种类型都适用于特定的用例，因此在编写类型时要注意你的意图。你学会了类型如何通过子类型化来建模*is-a*关系。你还了解了为什么你的 API 对每个类都如此重要；这是其他开发人员第一次了解你在做什么的机会。在本章结束时，你学到了除了静态类型检查之外，进行运行时验证的必要性。

在接下来的部分中，我将退后一步，从一个更广泛的视角来看待健壮性。本书前两部分的几乎所有指导都集中在类型注解和类型检查器上。可读性和错误检查是健壮性的重要好处，但它们并不是全部。其他维护者需要能够对你的代码库进行重大更改，引入新功能，而不仅仅是与你的类型交互进行小的更改。第三部分将专注于可扩展性。
