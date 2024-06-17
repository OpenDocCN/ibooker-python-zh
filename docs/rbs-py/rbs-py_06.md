# 第五章\. 集合类型

在 Python 中，没有遇到*集合类型*就无法深入学习。集合类型存储数据的分组，比如用户列表或餐馆或地址之间的查找。而其他类型（例如，`int`、`float`、`bool`等）可能专注于单个值，集合可以存储任意数量的数据。在 Python 中，你会遇到常见的集合类型，如字典、列表和集合（哦，我的！）。甚至字符串也是一种集合类型；它包含一系列字符。然而，当阅读新代码时，理解集合类型可能会很困难。不同的集合类型有不同的行为。

回顾第一章，我讨论了一些集合之间的差异，讨论了可变性、可迭代性和索引需求。然而，选择正确的集合只是第一步。你必须理解你的集合的影响，并确保用户可以理解它。当标准集合类型无法满足需求时，你还需要知道如何创建自定义集合。但第一步是知道如何向未来传达你的集合选择。为此，我们将转向一位老朋友：类型注解。

# 注释集合

我已经涵盖了非集合类型的类型注解，现在你需要知道如何注释集合类型。幸运的是，这些注解与你已经学过的注解并没有太大的不同。

举例说明，假设我正在构建一个数字食谱应用程序。我希望将所有的食谱书籍数字化，以便可以按照菜系、配料或作者搜索它们。关于食谱集合的一个问题是，我可能会问每位作者有多少本书：

```py
def create_author_count_mapping(cookbooks: list) -> dict:
    counter = defaultdict(lambda: 0)
    for book in cookbooks:
        counter[book.author] += 1
    return counter
```

此函数已经被注释；它接受一个食谱书籍列表，并返回一个字典。不幸的是，虽然这告诉我可以期待哪些集合，但它并没有告诉我如何使用这些集合。没有任何内容告诉我集合内部的元素是什么。例如，我怎么知道食谱书是什么类型？如果你在审查这段代码，你怎么知道使用 `book.author` 是合法的？即使你进行了调查以确保 `book.author` 是正确的，这段代码也不具备未来的可扩展性。如果底层类型发生变化，比如移除 `author` 字段，这段代码将会出问题。我需要一种方式来在类型检查器中捕获这一点。

我将通过使用方括号语法在我的类型中编码更多信息来做到这一点，以指示集合内部的类型信息：

```py
AuthorToCountMapping = dict[str, int]
def create_author_count_mapping(
				cookbooks: list[Cookbook]
                               ) -> AuthorToCountMapping:
    counter = defaultdict(lambda: 0)
    for book in cookbooks:
        counter[book.author] += 1
    return counter
```

###### 注意

我使用一个别名，`AuthorToCountMapping`，来表示一个`dict[str, int]`。我这样做是因为有时我会觉得很难记住`str`和`int`应该表示什么。但是，我承认这会丢失一些信息（代码的读者将不得不找出`AuthorToCountMapping`是什么的别名）。理想情况下，你的代码编辑器可以显示出底层类型，而不需要你查找。

我可以指示集合中预期的确切类型。cookbooks 列表包含`Cookbook`对象，并且函数的返回值返回一个字符串（键）到整数（值）的字典映射。请注意，我正在使用一个类型别名来为我的返回值提供更多含义。从`str`到`int`的映射并没有告诉用户类型的上下文。相反，我创建了一个名为`AuthorToCountMapping`的类型别名，以清楚地说明这个字典与问题域的关系。

为了有效地为集合进行类型提示，你需要考虑集合中包含的类型。为了做到这一点，你需要考虑同类集合和异类集合。

# 同类集合与异类集合

*同类集合*是指每个值都具有相同类型的集合。相比之下，*异类集合*中的值可能在其内部具有不同的类型。从可用性的角度来看，你的列表、集合和字典几乎总是应该是同类的。用户需要一种方法来推理你的集合，如果他们不能保证每个值都是相同类型的，那么他们就无法做到这一点。如果你将列表、集合或字典设为异类集合，那么你就在告诉用户他们需要注意处理特殊情况。假设我想从第一章中复活一个调整配方的示例，用于我的烹饪书应用：

```py
def adjust_recipe(recipe, servings):
    """
 Take a meal recipe and change the number of servings
 :param recipe: A list, where the first element is the number of servings,
 and the remainder of elements follow the (name, amount, unit)
 format, such as ("flour", 1.5, "cup")
 :param servings: the number of servings
 :return list: a new list of ingredients, where the first element is the
 number of servings
 """
    new_recipe = [servings]
    old_servings = recipe[0]
    factor = servings / old_servings
    recipe.pop(0)
    while recipe:
            ingredient, amount, unit = recipe.pop(0)
            # please only use numbers that will be easily measurable
            new_recipe.append((ingredient, amount * factor, unit))
    return new_recipe
```

当时，我提到这段代码的部分很丑陋；一个令人困惑的因素是配方列表的第一个元素是一个特殊情况：一个表示份量的整数。这与其余列表元素形成对比，其为表示实际配料的元组，比如`("面粉", 1.5, "杯")`。这突显了异类集合的问题。对于您集合的每次使用，用户都需要记住处理特殊情况。这是建立在开发人员甚至知道特殊情况的假设的基础上的。目前没有办法表示特定元素需要以不同方式处理。因此，类型检查器不会在开发人员忘记时捕捉到。这会导致未来代码脆弱。

当谈到同质性时，重要的是讨论*单一类型*的含义。当我提到单一类型时，并不一定指的是 Python 中的具体类型；我指的是定义该类型的一组行为。单一类型表明消费者必须以完全相同的方式处理该类型的每个值。对于食谱列表，单一类型是`Cookbook`。对于字典示例，键的单一类型是字符串，值的单一类型是整数。对于异构集合，情况并非总是如此。如果您的集合必须具有不同的类型，并且它们之间没有关系，该怎么办？

请考虑一下我从第一章中的丑陋代码所传达的意思：

```py
def adjust_recipe(recipe, servings):
    """
 Take a meal recipe and change the number of servings
 :param recipe: A list, where the first element is the number of servings,
 and the remainder of elements follow the (name, amount, unit)
 format, such as ("flour", 1.5, "cup")
 :param servings: the number of servings
 :return list: a new list of ingredients, where the first element is the
 number of servings
 """
    # ...
```

在文档字符串中包含了大量信息，但文档字符串并不能保证是正确的。它们也不能保护开发者免受意外破坏假设的影响。这段代码未能充分传达未来合作者的意图。未来的合作者无法推理你的代码。你最不希望给他们的是让他们不得不查看代码库，寻找调用和实现方式来使用你的集合。最终，你需要一种方法来协调列表中的第一个元素（一个整数）与列表中剩余元素（元组）之间的关系。为了解决这个问题，我将使用一个`Union`（以及一些类型别名来使代码更易读）：

```py
Ingredient = tuple[str, int, str] # (name, quantity, units)
Recipe = list[Union[int, Ingredient]] # the list can be servings or ingredients
def adjust_recipe(recipe: Recipe, servings):
    # ...
```

这将异构集合（项目可以是整数或成分）转换为开发者可以像处理同质集合一样推理的集合。开发者需要将每个值视为相同——它要么是整数，要么是`Ingredient`——然后再对其进行操作。虽然需要更多的代码来处理类型检查，但您可以放心，您的类型检查器将会捕获到未检查特例的用户。请记住，这并不完美；如果首次没有特例，并且`serving`可以以另一种方式传递给函数，那将更好。但对于您绝对必须处理特例的情况，请将它们表示为一种类型，以便类型检查器为您提供帮助。

###### 小贴士

当异构集合足够复杂，涉及大量验证逻辑散布在代码库中时，考虑将其作为用户定义类型，例如数据类或类。有关创建用户定义类型的更多信息，请参阅第 II 部分。

但是，您可能会在`Union`中添加太多类型。您处理的类型特例越多，每次使用该类型时开发者需要编写的代码就越多，代码库也就变得越来越难以管理。

在光谱的另一端是`Any`类型。`Any`可以用于指示在此上下文中所有类型都是有效的。这听起来很吸引人，可以避开特殊情况，但这也意味着集合的消费者不知道如何处理集合中的值，从而打败了首次使用类型注解的目的。

###### 警告

在静态类型语言中工作的开发人员无需花费太多精力确保集合是同构的；静态类型系统已经为他们完成了这项工作。Python 的挑战在于 Python 的动态类型特性。对于开发人员来说，创建一个异构集合而不受语言本身的任何警告要容易得多。

异构的集合类型仍然有很多用途；不要假设你应该为每种集合类型使用同构性，因为这样更容易理解。例如，元组经常是异构的。

假设一个包含名称和页数的元组表示一个`Cookbook`：

```py
Cookbook = tuple[str, int] # name, page count
```

我正在描述此元组的特定字段：名称和页数。这是一个异构集合的典型例子：

+   每个字段（名称和页数）将始终按相同顺序出现。

+   所有名称都是字符串；所有页数都是整数。

+   很少迭代元组，因为我不会将两种类型视为相同。

+   名称和页数是根本不同的类型，不应视为相等。

访问元组时，您通常会索引到您想要的特定字段：

```py
food_lab: Cookbook = ("The Food Lab", 958)
odd_bits: Cookbook = ("Odd Bits", 248)

print(food_lab[0])
>>> "The Food Lab"

print(odd_bits[1])
>>> 248
```

然而，在许多代码库中，这样的元组很快就会变得繁琐。开发人员厌倦了每次想要名称时都写`cookbook[0]`。更好的做法是找到一种方法来为这些字段命名。第一选择可能是一个字典：

```py
food_lab = {
    "name": "The Food Lab",
    "page_count": 958
}
```

现在，他们可以将字段称为`food_lab['name']`和`food_lab['page_count']`。问题是，字典通常用于表示从键到值的同构映射。但是，当字典用于表示异构数据时，您会遇到与上述写有效类型注释时相似的问题。如果我想尝试使用类型系统来表示这个字典，我最终会得到以下结果：

```py
def print_cookbook(cookbook: dict[str, Union[str,int]])
    # ...
```

这种方法存在以下问题：

+   大字典可能有许多不同类型的值。编写一个`Union`非常麻烦。

+   对于用户来说，处理每个字典访问的每种情况都很烦琐。（因为我表明字典是同构的，我向开发人员传达了他们需要将每个值视为相同类型的信息，这意味着对每个值访问进行类型检查。*我*知道`name`总是`str`，`page_count`总是`int`，但这种类型的消费者不会知道。）

+   开发人员没有任何指示字典中有哪些键。他们必须从字典创建时间到当前访问的所有代码中查找已添加的字段。

+   当字典增长时，开发者往往倾向于将值的类型用`Any`表示。在这种情况下，使用`Any`会使类型检查器失去作用。

###### 注意

`Any`可以用于有效的类型注解；它仅表示你对类型没有任何假设。例如，如果你想要复制一个列表，类型签名将是`def copy(coll: list[Any]) -> list[Any]`。当然，你也可以这样做`def copy(coll: list) -> list`，它的含义是一样的。

所有这些问题都源于同质数据集合中的异构数据。你要么把负担转嫁给调用者，要么完全放弃类型注解。在某些情况下，你希望调用者在每次值访问时明确检查每个类型，但在其他情况下，这种方法会显得过于复杂和乏味。那么，在处理异构类型时，特别是在像 API 交互或用户可配置数据这样自然使用字典的情况下，你该如何解释你的理由呢？对于这些情况，你应该使用`TypedDict`。

# TypedDict

`TypedDict`，引入于 Python 3.8，用于必须在字典中存储异构数据的场景。这些通常是你无法避免异构数据的情况。JSON API、YAML、TOML、XML 和 CSV 都有易于使用的 Python 模块，可以将这些数据格式转换为字典，并且自然是异构的。这意味着返回的数据具有前面章节列出的所有问题。你的类型检查器帮不上忙，用户也不知道可用的键和值是什么。

###### 提示

如果你完全控制字典，即你在自己的代码中创建它并在自己的代码中处理它，你应该考虑使用`dataclass`（见第九章）或`class`（见第十章）。

例如，假设我想扩展我的数字食谱应用程序，以提供列出的食谱的营养信息。我决定使用[Spoonacular API](https://oreil.ly/joTNh)，并编写一些代码来获取营养信息：

```py
nutrition_information = get_nutrition_from_spoonacular(recipe_name)
# print grams of fat in recipe
print(nutrition_information["fat"]["value"])
```

如果你正在审查代码，你如何知道这段代码是正确的？如果你还想打印卡路里，你如何访问这些数据？你对这个字典内部字段有什么保证？要回答这些问题，你有两个选择：

+   查阅 API 文档（如果有的话），确认是否使用了正确的字段。在这种情况下，你希望文档实际上是完整和正确的。

+   运行代码并打印返回的字典。在这种情况下，你希望测试响应与生产响应基本相同。

问题在于，你要求每个读者、审阅者和维护者必须执行这两个步骤之一才能理解代码。如果他们没有这样做，你将得不到良好的代码审查反馈，开发者将冒着使用响应不正确的风险。这会导致错误的假设和脆弱的代码。`TypedDict`允许你直接将你对 API 的了解编码到你的类型系统中。

```py
from typing import TypedDict
class Range(TypedDict):
    min: float
    max: float

class NutritionInformation(TypedDict):
    value: int
    unit: str
    confidenceRange95Percent: Range
    standardDeviation: float

class RecipeNutritionInformation(TypedDict):
    recipes_used: int
    calories: NutritionInformation
    fat: NutritionInformation
    protein: NutritionInformation
    carbs: NutritionInformation

nutrition_information:RecipeNutritionInformation = \
	get_nutrition_from_spoonacular(recipe_name)
```

现在非常明显，你可以依赖哪些数据类型。如果 API 有所更改，开发者可以更新所有`TypedDict`类，并让类型检查器捕捉到任何不一致之处。你的类型检查器现在完全理解你的字典，代码的读者可以在不进行任何外部搜索的情况下推理出响应。

更好的是，这些`TypedDict`集合可以随意复杂化，以满足你的需求。你会看到我嵌套了`TypedDict`实例以提高复用性，但你也可以嵌入自己的自定义类型、`Union`和`Optional`，以反映 API 可能返回的情况。虽然我大多数时候在谈论 API，但请记住，这些好处适用于任何异质字典，比如读取 JSON 或 YAML 时。

###### 注意

`TypedDict`仅用于类型检查器的利益。完全没有运行时验证；运行时类型只是一个字典。

到目前为止，我教你如何处理内置的集合类型：列表/集合/字典用于同质集合，元组/`TypedDict`用于异质集合。如果这些类型不能满足你的*所有*需求呢？如果你想创建易于使用的新集合呢？为此，你需要一套新的工具。

# 创建新集合

当你要编写一个新的集合时，你应该问自己：我是想编写一个无法用其他集合类型表示的新集合，还是想修改一个现有集合以提供一些新的行为？根据答案的不同，你可能需要采用不同的技术来实现你的目标。

如果你编写了一个无法用其他集合类型表示的集合类型，你在某个时候必然会遇到*泛型*。

## 泛型

泛型类型表示你不关心使用什么类型。然而，它有助于阻止用户在不合适的地方混合类型。

考虑这个无害的反转列表函数：

```py
def reverse(coll: list) -> list:
    return coll[::-1]
```

我如何表明返回的列表应该包含与传入列表相同类型的类型？为了实现这一点，我使用了一个泛型，在 Python 中使用`TypeVar`来实现：

```py
from typing import TypeVar
T = TypeVar('T')
def reverse(coll: list[T]) -> list[T]:
    return coll[::-1]
```

这意味着对于类型`T`，`reverse`接受一个类型为`T`的元素列表，并返回一个类型为`T`的元素列表。我不能混合类型：如果这些列表没有使用相同的`TypeVar`，那么整数列表永远无法变成字符串列表。

我可以使用这种模式来定义整个类。假设我想将一个烹饪书推荐服务集成到烹饪集合应用程序中。我想要根据客户的评分推荐烹饪书或食谱。为此，我想将每个这些评分信息存储在一个*图*中。图是一种包含一系列实体（称为*节点*）并跟踪*边*（这些节点之间的关系）的数据结构。但是，我不想为烹饪图和食谱图编写单独的代码。因此，我定义了一个可以用于通用类型的`Graph`类：

```py
from collections import defaultdict
from typing import Generic, TypeVar

Node = TypeVar("Node")
Edge = TypeVar("Edge")

# directed graph
class Graph(Generic[Node, Edge]):
    def __init__(self):
        self.edges: dict[Node, list[Edge]] = defaultdict(list)

    def add_relation(self, node: Node, to: Edge):
        self.edges[node].append(to)

    def get_relations(self, node: Node) -> list[Edge]:
        return self.edges[node]
```

有了这段代码，我可以定义各种类型的图并且仍然可以成功进行类型检查：

```py
cookbooks: Graph[Cookbook, Cookbook] = Graph()
recipes: Graph[Recipe, Recipe] = Graph()

cookbook_recipes: Graph[Cookbook, Recipe] = Graph()

recipes.add_relation(Recipe('Pasta Bolognese'),
                     Recipe('Pasta with Sausage and Basil'))

cookbook_recipes.add_relation(Cookbook('The Food Lab'),
                              Recipe('Pasta Bolognese'))
```

而这段代码无法进行类型检查：

```py
cookbooks.add_relation(Recipe('Cheeseburger'), Recipe('Hamburger'))
```

```py
code_examples/chapter5/invalid/graph.py:25:
    error: Argument 1 to "add_relation" of "Graph" has
           incompatible type "Recipe"; expected "Cookbook"
```

使用泛型可以帮助您编写在其整个生命周期中一致使用类型的集合。这减少了代码库中的重复量，从而减少了错误的机会并减轻了认知负担。

## 修改现有类型

泛型非常适合创建自己的集合类型，但是如果您只想调整现有集合类型（例如列表或字典）的某些行为怎么办？完全重新编写集合的所有语义将是乏味且容易出错的。幸运的是，存在可以轻松完成这项工作的方法。让我们回到我们的烹饪应用程序。我之前写过代码来获取营养信息，但现在我想将所有这些营养信息存储在一个字典中。

但是，我遇到了一个问题：同一种成分在不同地方具有非常不同的名称。以沙拉中常见的深色叶绿素为例。美国厨师可能称之为“火箭”，而欧洲厨师可能称之为“火箭菜”。这甚至还不包括除英语以外的其他语言中的名称。为了应对这个问题，我想创建一个类似字典的对象，可以自动处理这些别名：

```py
>>> nutrition = NutritionalInformation()
>>> nutrition["arugula"] = get_nutrition_information("arugula")
>>> print(nutrition["rocket"]) # arugula is the same as rocket
{
    "name": "arugula",
    "calories_per_serving": 5,
    # ... snip ...
}
```

那么我如何让`NutritionalInformation`的行为像字典一样呢？

许多开发人员的第一反应是对字典进行子类化。如果您对子类化不是很擅长也不用担心；我将在第十二章中更加深入地讨论这个问题。目前，只需将子类化视为一种表达“我希望我的子类的行为与父类完全相同”的方式即可。但是，您将会发现，子类化字典可能并不总是您想要的。考虑以下代码：

```py
class NutritionalInformation(dict): ![1](img/00002.gif)
    def __getitem__(self, key): ![2](img/00005.gif)
        try:
            return super().__getitem__(key) ![3](img/00006.gif)
        except KeyError:
            pass
        for alias in get_aliases(key):
            try: ![4](img/00007.gif)
                return super().__getitem__(alias)
            except KeyError:
                pass
        raise KeyError(f"Could not find {key} or any of its aliases") ![5](img/00008.gif)
```

![1](img/part0008_split_006.html#co_collection_types_CO1-1)

`(dict)`语法表示我们正在从字典进行子类化。

![2](img/part0008_split_006.html#co_collection_types_CO1-2)

`__getitem__`是在字典中使用括号检查键时调用的方法：(`nutrition["rocket"]`)调用 `__getitem__(nutrition, "rocket")`。

![3](img/part0008_split_006.html#co_collection_types_CO1-3)

如果找到键，则使用父字典的键检查。

![4](img/part0008_split_006.html#co_collection_types_CO1-4)

对于每个别名，请检查它是否在字典中。

![5](img/part0008_split_006.html#co_collection_types_CO1-5)

如果找不到键或其任何别名，则抛出`KeyError`异常。

我们正在重写`__getitem__`函数，这样就可以了！

如果我尝试在上面的片段中访问`nutrition["rocket"]`，我将获得与`nutrition["arugula"]`相同的营养信息。太棒了！于是你将其部署到生产环境，并算是一天结束了。

但是（总会有个但是），随着时间的推移，一位开发人员向你抱怨说有时字典不起作用。你花了些时间进行调试，但是它对你来说总是有效的。你寻找竞态条件、线程问题、API 问题或任何其他非确定性因素，但却完全找不到潜在的错误。最终，你找到了一些时间可以和另一位开发人员坐下来看看他们在做什么。

现在他们的终端上显示以下行：

```py
# arugula is the same as rocket
>>> nutrition = NutritionalInformation()
>>> nutrition["arugula"] = get_nutrition_information("arugula")
>>> print(nutrition.get("rocket", "No Ingredient Found"))
"No Ingredient Found"
```

字典上的`get`函数尝试获取键，如果找不到，则返回第二个参数（在本例中是“No Ingredient Found”）。这里出现了问题：当从字典派生并重写方法时，你无法保证其他方法在字典中调用这些方法。内置集合类型的设计考虑了性能；许多方法使用内联代码以提高速度。这意味着重写一个方法（如`__getitem__`）将不会被大多数字典方法使用。这显然违反了最小惊讶法则，我们在第一章中讨论过这一点。

###### 注意

如果仅添加方法，则可以从内置集合类继承，但是因为将来的修改可能会犯同样的错误，我仍然更喜欢使用其他一种方法来构建自定义集合。

因此，覆盖`dict`是不行的。我将使用`collections`模块中的类型。在这种情况下，有一个便利的类型叫做`collections.UserDict`。`UserDict`正好符合我需要的用例：我可以从`UserDict`派生，重写关键方法，并获得我期望的行为。

```py
from collections import UserDict
class NutritionalInformation(UserDict):
    def __getitem__(self, key):
        try:
            return self.data[key]
        except KeyError:
            pass
        for alias in get_aliases(key):
            try:
                return self.data[alias]
            except KeyError:
                pass
        raise KeyError(f"Could not find {key} or any of its aliases")
```

这正好符合你的使用场景。你应该从`UserDict`而不是`dict`派生，然后使用`self.data`来访问底层字典。

你再次运行你同事的代码：

```py
# arugula is the same as rocket
>>> print(nutrition.get("rocket", "No Ingredient Found"))
{
    "name": "arugula",
    "calories_per_serving": 5,
    # ... snip ...
}
```

你可以获取芝麻菜的营养信息。

在这种情况下，`UserDict`并不是你可以重写的唯一集合类型。`collections`模块中还有`UserString`和`UserList`。每当你想要调整字典、字符串或列表时，这些集合就派上用场了。

###### 警告

从这些类继承确实会增加性能成本。内置集合做了一些假设以实现性能优化。对于`UserDict`、`UserString`和`UserList`，方法无法内联，因为你可能会重写它们。如果你需要在性能关键代码中使用这些结构，请确保进行基准测试和测量，找出潜在问题。

您会注意到，我上面谈到了字典、列表和字符串，但是遗漏了一个重要的内置类型：集合。在 `collections` 模块中不存在 `UserSet`。我将不得不从 `collections.abc` 中选择一个不同的抽象。更具体地说，我需要抽象基类，这些基类位于 `collections.abc` 中。

## ABC 就这么简单

`collections.abc` 模块中的抽象基类（ABC）提供了另一组可以重写以创建自定义集合的类。ABC 是打算作为子类化的类，并要求子类实现非常具体的函数。对于 `collections.abc`，这些 ABC 都围绕着自定义集合展开。为了创建自定义集合，您必须覆盖特定的函数，具体取决于您想要模拟的类型。一旦实现了这些必需的函数，ABC 就会自动填充其他函数。您可以在 `collections.abc` 的[模块文档](https://oreil.ly/kb8j3)中找到要实现的全部必需函数列表。

###### 注

与 `User*` 类不同，`collections.abc` 类中没有内置存储，如 `self.data`。您必须提供自己的存储。

让我们来看一个 `collections.abc.Set`，因为在 `collections` 中没有 `UserSet`。我想创建一个自定义集合，自动处理成分的别名（如 rocket 和 arugula）。为了创建此自定义集合，我需要按照 `collections.abc.Set` 的要求实现三种方法：

`__contains__`

用于成员检查：`"arugula" in ingredients`。

`__iter__`

用于迭代：`for ingredient in ingredients`。

`__len__`

用于检查长度：`len(ingredients)`。

一旦定义了这三种方法，像关系操作、相等操作和集合操作（并集、交集、差集、不相交）等方法就能正常工作。这就是 `collections.abc` 的美妙之处。一旦定义了少数几个方法，其他方法就会自动补充。它在这里得以实现：

```py
import collections
class AliasedIngredients(collections.abc.Set):
    def __init__(self, ingredients: set[str]):
        self.ingredients = ingredients

    def __contains__(self, value: str):
        return value in self.ingredients or any(alias in self.ingredients
                                                for alias in get_aliases(value))

    def __iter__(self):
        return iter(self.ingredients)

    def __len__(self):
        return len(self.ingredients)

>>> ingredients = AliasedIngredients({'arugula', 'eggplant', 'pepper'})
>>> for ingredient in ingredients:
>>>    print(ingredient)
'arugula'
'eggplant'
'pepper'

>>> print(len(ingredients))
3

>>> print('arugula' in ingredients)
True

>>> print('rocket' in ingredients)
True

>>> list(ingredients | AliasedIngredients({'garlic'}))
['pepper', 'arugula', 'eggplant', 'garlic']
```

`collections.abc` 的另一个酷炫之处在于，使用它进行类型注释可以帮助你编写更通用的代码。回到第二章中的这段代码：

```py
def print_items(items):
    for item in items:
        print(item)

print_items([1,2,3])
print_items({4, 5, 6})
print_items({"A": 1, "B": 2, "C": 3})
```

我谈到了鸭子类型如何成为可靠代码的福音和诅咒。能够编写一个可以接受多种不同类型的单一函数是很棒的，但是通过类型注释来传达意图变得具有挑战性。幸运的是，我可以使用 `collections.abc` 类来提供类型提示：

```py
def print_items(items: collections.abc.Iterable):
    for item in items:
        print(item)
```

在这种情况下，我指出项目仅通过 `Iterable` ABC 可迭代。只要参数支持 `__iter__` 方法（大多数集合都支持），此代码将进行类型检查。

自 Python 3.9 起，有 25 种不同的 ABC 可供使用。在 [Python 文档](https://oreil.ly/lDeak) 中查看它们的全部内容。

# 总结

在 Python 中，少不了与集合打交道。列表、字典和集合很常见，因此向未来提供关于你正在使用的集合类型的提示是至关重要的。考虑你的集合是同质的还是异质的，以及这对未来的读者有何启发。对于使用异质集合的情况，提供足够的信息让其他开发者能够推理，比如`TypedDict`。一旦学会了让其他开发者理解你的集合的技巧，你的代码库将变得更加易读。

创建新集合时，务必仔细考虑各种选择：

+   如果你只是在扩展类型，比如添加新方法，可以直接从集合（如列表或字典）派生子类。然而，要注意粗糙的边缘，因为如果用户覆盖了内置方法，Python 会有一些出人意料的行为。

+   如果你想要更改列表、字典或字符串中的一小部分，请分别使用`collections.UserList`、`collections.UserDict`或`collections.UserString`。记得引用`self.data`来访问相应类型的存储。

+   如果需要编写接口与其他集合类型相似的更复杂的类，请使用`collections.abc`。你需要为类内部的数据提供自己的存储，并实现所有必需的方法，但一旦完成，你可以根据心情自定义该集合。

# 讨论主题

查看代码库中对集合和泛型的使用情况，并评估向未来开发者传达了多少信息。你的代码库中有多少自定义的集合类型？新开发者只需查看类型签名和名称就能了解集合类型的多少信息？你是否可以更通用地定义一些集合？其他类型是否可以使用泛型？

现在，类型注解没有一个类型检查器的帮助就无法充分发挥其潜力。在接下来的章节中，我将专注于类型检查器本身。你将学会如何有效地配置类型检查器、生成报告并评估不同的检查器。你了解的工具越多，就能越有效地使用它。对于你的类型检查器来说，这一点尤为重要。
