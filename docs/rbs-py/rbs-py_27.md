# 第二十三章：属性化测试

在您的代码库中不可能测试所有东西。您能做的最好的事情就是在如何针对特定用例上变得聪明。您寻找边界情况，代码路径，以及代码的其他有趣属性。您的主要希望是您没有在安全网中留下任何大漏洞。然而，您可以做得比希望更好。您可以使用属性化测试填补这些空白。

在本章中，您将学习如何使用名为[`Hypothesis`](https://oreil.ly/OejR4)的 Python 库进行基于属性的测试。您将使用`Hypothesis`来为您生成测试用例，通常是以您意想不到的方式。您将学习如何跟踪失败的测试用例，以新的方式制定输入数据，甚至让`Hypothesis`创建算法的组合来测试您的软件。`Hypothesis`将保护您的代码库免受一系列新错误的影响。

# 使用 Hypothesis 进行属性化测试

属性化测试是一种*生成式测试*形式，工具会为您生成测试用例。与基于特定输入/输出组合编写测试用例不同，您定义系统的*属性*。在这个上下文中，*属性*是指系统中成立的不变量（在第十章中讨论）的另一个名称。

考虑一个菜单推荐系统，根据顾客提供的约束条件选择菜肴，例如总热量、价格和菜系。对于这个特定示例，我希望顾客能够订购一顿全餐，热量低于特定的热量目标。以下是我为此功能定义的不变量：

+   客户将收到三道菜：前菜、沙拉和主菜。

+   当将所有菜肴的热量加在一起时，总和会小于它们的预期目标。

如果我要将此作为`pytest`测试来专注于测试这些属性，它会看起来像以下内容：

```py
def test_meal_recommendation_under_specific_calories():
    calories = 900
    meals = get_recommended_meal(Recommendation.BY_CALORIES, calories)
    assert len(meals) == 3
    assert is_appetizer(meals[0])
    assert is_salad(meals[1])
    assert is_main_dish(meals[2])
    assert sum(meal.calories for meal in meals) < calories
```

将此与测试特定结果进行对比：

```py
def test_meal_recommendation_under_specific_calories():
    calories = 900
    meals = get_recommended_meal(Recommendation.BY_CALORIES, calories)
    assert meals == [Meal("Spring Roll", 120),
                     Meal("Green Papaya Salad", 230),
                     Meal("Larb Chicken", 500)]
```

第二种方法是测试非常具体的一组餐点；这种测试更具体，但也更*脆弱*。当生产代码发生变化时，比如引入新菜单项或更改推荐算法时，它更容易出现问题。理想的测试是只有在出现真正的错误时才会失败。请记住，测试并非免费。您希望减少维护成本，缩短调整测试所需的时间是一个很好的方法。

在这两种情况下，我正在使用特定输入进行测试：900 卡路里。为了建立更全面的安全网，扩展您的输入领域以测试更多案例是一个好主意。在传统的测试案例中，您通过执行*边界值分析*来选择编写哪些测试。边界值分析是指分析待测试的代码，寻找不同输入如何影响控制流程或代码中的不同执行路径。

例如，假设 `get_recommended_meal` 在卡路里限制低于 650 时引发错误。在这种情况下的边界值是 650；这将输入域分割成两个*等价类*或具有相同属性的值集。一个等价类是所有低于 650 卡路里的数字，另一个等价类是 650 及以上的值。通过边界值分析，应该有三个测试：一个测试低于 650 卡路里的卡路里，一个测试刚好在 650 卡路里的边界处，以及一个测试一个高于 650 卡路里的值。实际上，这验证了开发人员没有搞错关系运算符（例如写成 `<=` 而不是 `<`）或者出现了差一错误。

然而，边界值分析仅在你能够轻松分割输入域时才有用。如果确定在哪里应该分割域很困难，那么挑选边界值将不容易。这就是 `Hypothesis` 的生成性质发挥作用的地方；`Hypothesis` 为测试用例生成输入。它将为你找到边界值。

你可以通过 `pip` 安装 `Hypothesis`：

```py
pip install hypothesis
```

我将修改我的原始属性测试，让 `Hypothesis` 负责生成输入数据。

```py
from hypothesis import given
from hypothesis.strategies import integers

@given(integers())
def test_meal_recommendation_under_specific_calories(calories):
    meals = get_recommended_meal(Recommendation.BY_CALORIES, calories)
    assert len(meals) == 3
    assert is_appetizer(meals[0])
    assert is_salad(meals[1])
    assert is_main_dish(meals[2])
    assert sum(meal.calories for meal in meals) < calories
```

只需一个简单的装饰器，我就可以告诉 `Hypothesis` 为我选择输入。在这种情况下，我要求 `Hypothesis` 生成不同的 `integers` 值。`Hypothesis` 将运行此测试多次，尝试找到违反预期属性的值。如果我用 `pytest` 运行这个测试，我会看到以下输出：

```py
Falsifying example: test_meal_recommendation_under_specific_calories(
    calories=0,
)
============= short test summary info ======================
FAILED code_examples/chapter23/test_basic_hypothesis.py::
    test_meal_recommendation_under_specific_calories - assert 850 < 0
```

`Hypothesis` 在我的生产代码中早期发现了一个错误：代码不能处理零卡路里限制。现在，对于这种情况，我想指定我只应该测试某个特定数量的卡路里或以上：

```py
@given(integers(min_value=900))
def test_meal_recommendation_under_specific_calories(calories)
    # ... snip ...
```

现在，当我用 `pytest` 命令运行时，我想展示一些关于 `Hypothesis` 的更多信息。我会运行：

```py
py.test code_examples/chapter23 --hypothesis-show-statistics
```

这将产生以下输出：

```py
code_examples/chapter23/test_basic_hypothesis.py::
    test_meal_recommendation_under_specific_calories:

  - during generate phase (0.19 seconds):
    - Typical runtimes: 0-1 ms, ~ 48% in data generation
    - 100 passing examples, 0 failing examples, 0 invalid examples

  - Stopped because settings.max_examples=100
```

`Hypothesis` 为我检查了 100 个不同的值，而我不需要提供任何具体的输入。更重要的是，每次运行这个测试时，`Hypothesis` 都会检查新值。与一次又一次地限制自己于相同的测试用例不同，你可以在你测试的内容上获得更广泛的覆盖面。考虑到所有不同的开发人员和持续集成管道系统执行测试，你会意识到你可以多快地捕获边缘情况。

###### 提示

你还可以通过使用 `hypothesis.assume` 来在你的领域上指定约束条件。你可以在你的测试中写入假设，比如 `assume(calories > 850)`，告诉 `Hypothesis` 跳过违反这些假设的任何测试用例。

如果我引入一个错误（例如因某种原因在 5,000 到 5,200 卡路里之间出错），`Hypothesis` 将在四次测试运行内捕获错误（你的测试运行次数可能会有所不同）：

```py
_________ test_meal_recommendation_under_specific_calories _________

    @given(integers(min_value=900))
>   def test_meal_recommendation_under_specific_calories(calories):

code_examples/chapter23/test_basic_hypothesis.py:33:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

calories = 5001

    @given(integers(min_value=900))
    def test_meal_recommendation_under_specific_calories(calories):
        meals = get_recommended_meal(Recommendation.BY_CALORIES, calories)
>       assert len(meals) == 3
E       TypeError: object of type 'NoneType' has no len()

code_examples/chapter23/test_basic_hypothesis.py:35: TypeError
------------------------ Hypothesis --------------------------------
Falsifying example: test_meal_recommendation_under_specific_calories(
    calories=5001,
)
=========== Hypothesis Statistics ========================
code_examples/chapter23/test_basic_hypothesis.py::
   test_meal_recommendation_under_specific_calories:

  - during reuse phase (0.00 seconds):
    - Typical runtimes: ~ 1ms, ~ 43% in data generation
    - 1 passing examples, 0 failing examples, 0 invalid examples

  - during generate phase (0.08 seconds):
    - Typical runtimes: 0-2 ms, ~ 51% in data generation
    - 26 passing examples, 1 failing examples, 0 invalid examples
    - Found 1 failing example in this phase

  - during shrink phase (0.07 seconds):
    - Typical runtimes: 0-2 ms, ~ 37% in data generation
    - 22 passing examples, 12 failing examples, 1 invalid examples
    - Tried 35 shrinks of which 11 were successful

  - Stopped because nothing left to do
```

当您发现错误时，`Hypothesis`会记录失败的错误，以便将来可以专门检查该值。您还可以通过`hypothesis.example`装饰器确保`Hypothesis`始终测试特定情况：

```py
@given(integers(min_value=900))
@example(5001)
def test_meal_recommendation_under_specific_calories(calories)
    # ... snip ...
```

## 魔法假设

`Hypothesis`非常擅长生成会发现错误的测试用例。这似乎像是魔法，但实际上相当聪明。在前面的例子中，您可能已经注意到`Hypothesis`在值 5001 上出现了错误。如果您运行相同的代码并为大于 5000 的值引入一个错误，您将发现测试仍在 5001 处出现错误。如果`Hypothesis`正在测试不同的值，我们难道不应该看到稍微不同的结果吗？

当`Hypothesis`发现失败时，它会为您做一些非常好的事情：它*缩小*了测试用例。缩小是指`Hypothesis`尝试找到仍然导致测试失败的最小输入。对于`integers()`，`Hypothesis`会尝试依次更小的数字（或处理负数时更大的数字），直到输入值达到零。`Hypothesis`试图聚焦（无意冒犯）于仍然导致测试失败的最小值。

要了解有关`Hypothesis`如何生成和缩小值的更多信息，值得阅读原始[QuickCheck 论文](https://oreil.ly/htavw)。QuickCheck 是最早的基于属性的工具之一，尽管它涉及 Haskell 编程语言，但信息量很大。大多数基于属性的测试工具（如`Hypothesis`）都是基于 QuickCheck 提出的思想的后继者。

## 与传统测试的对比

基于属性的测试可以极大地简化编写测试的过程。有一整类问题是你不需要担心的：

更容易测试非确定性

非确定性是大多数传统测试的祸根。随机行为、创建临时目录或从数据库中检索不同的记录可能会使编写测试变得非常困难。您必须在测试中创建一组特定的输出值，为此，您需要是确定性的；否则，您的测试将一直失败。通常，您会尝试通过强制特定行为来控制非确定性，例如强制创建相同的文件夹或对随机数生成器进行种子化。

使用基于属性的测试，非确定性是其中的一部分。`Hypothesis`将为每个测试运行提供不同的输入。您不必再担心测试特定值了；定义属性并接受非确定性。您的代码库会因此变得更好。

更少的脆弱性

在测试特定输入/输出组合时，您受到一大堆硬编码假设的影响。您假设列表始终按照相同的顺序排列，字典不会添加任何键-值对，并且您的依赖项永远不会改变其行为。这些看似不相关的变化中的任何一个都可能破坏您的一个测试。

当测试由于与被测试功能无关的原因而失败时，这是令人沮丧的。测试因易出错而声名狼藉，要么被忽视（掩盖真正的失败），要么开发者不得不面对不断需要修复测试的烦恼。使用基于属性的测试增强您的测试的韧性。

更好地找到错误的机会

基于属性的测试不仅仅是为了减少测试创建和维护成本。它将增加您发现错误的机会。即使今天您编写的测试覆盖了代码的每条路径，仍然有可能会遗漏某些情况。如果您的函数以不向后兼容的方式更改（例如，现在对您先前认为是正常的值出现错误），那么您的运气取决于是否有一个特定值的测试用例。基于属性的测试，通过生成新的测试用例，将在多次运行中更有可能发现该错误。

# 讨论主题

检查您当前的测试用例，并选择阅读起来复杂的测试。搜索需要大量输入和输出以充分测试功能的测试。讨论基于属性的测试如何取代这些测试并简化您的测试套件。

# 充分利用`Hypothesis`

到目前为止，我只是初步了解了`Hypothesis`。一旦你真正深入进行基于属性的测试，你会为自己打开大量的机会。`Hypothesis`提供了一些非常酷的功能，可以显著改进您的测试体验。

## Hypothesis 策略

在上一节中，我向您介绍了`integers()`策略。`Hypothesis`策略定义了如何生成测试用例以及在测试用例失败时如何收缩数据。`Hypothesis`内置了大量的策略。类似于将`integers()`传递给您的测试用例，您可以传递`floats()`、`text()`或`times()`来生成浮点数、字符串或`datetime.time`对象的值。

`Hypothesis`还提供了一些可以组合其他策略的策略，例如构建策略的列表、元组或字典（这是组合性的一个很好的例子，如第十七章所述）。例如，假设我想创建一个策略，将菜名（文本）映射到卡路里（在 100 到 2000 之间的数字）：

```py
from hypothesis import given
from hypothesis.strategies import dictionary, integers, text

@given(dictionaries(text(), integers(min_value=100, max_value=2000)))
def test_calorie_count(ingredient_to_calorie_mapping : dict[str, int]):
    # ... snip ...
```

对于更复杂的数据，你可以使用`Hypothesis`来定义自己的策略。您可以使用`map`和`filter`策略，这些策略的概念类似于内置的`map`和`filter`函数。

您还可以使用`hypothesis.composite`策略装饰器来定义自己的策略。我想创建一个策略，为我创建三道菜的套餐，包括前菜、主菜和甜点。每道菜包含名称和卡路里计数：

```py
from hypothesis import given
from hypothesis.strategies import composite, integers

ThreeCourseMeal = tuple[Dish, Dish, Dish]

@composite
def three_course_meals(draw) -> ThreeCourseMeal:
    appetizer_calories = integers(min_value=100, max_value=900)
    main_dish_calories = integers(min_value=550, max_value=1800)
    dessert_calories = integers(min_value=500, max_value=1000)

    return (Dish("Appetizer", draw(appetizer_calories)),
            Dish("Main Dish", draw(main_dish_calories)),
            Dish("Dessert", draw(dessert_calories)))

@given(three_course_meals)
def test_three_course_meal_substitutions(three_course_meal: ThreeCourseMeal):
    # ... do something with three_course_meal
```

此示例通过定义一个名为`three_course_meals`的新复合策略来工作。我创建了三种整数策略；每种类型的菜品都有自己的策略及其自己的最小/最大值。然后，我创建了一个新的菜品，它具有名称和从策略中*绘制*的值。`draw`是一个传递给您的复合策略的函数，您可以使用它来选择策略中的值。

一旦您定义了自己的策略，就可以在多个测试中重复使用它们，从而轻松为系统生成新数据。要了解更多关于`Hypothesis`策略的信息，建议您阅读[`Hypothesis`文档](https://oreil.ly/QhhnM)。

## 生成算法

在先前的示例中，我专注于生成输入数据以创建您的测试。然而，`Hypothesis`可以进一步生成操作的组合。`Hypothesis`称之为*有状态测试*。

考虑我们的餐饮推荐系统。我展示了如何按卡路里进行过滤，但现在我还想按价格、课程数、接近用户等进行过滤。以下是我想要对系统断言的一些属性：

+   餐饮推荐系统始终返回三种餐饮选择；可能不是所有推荐的选项都符合用户的所有标准。

+   所有三种餐饮选择都是唯一的。

+   餐饮选择是基于最近应用的过滤器排序的。在出现平局的情况下，使用次新的过滤器。

+   新的过滤器会替换相同类型的旧过滤器。例如，如果您将价格过滤器设置为<$20，然后将其更改为<$15，则只应用<$15 过滤器。设置像卡路里过滤器这样的内容，例如<1800 卡路里，并不会影响价格过滤器。

而不是编写大量的测试用例，我将使用`hypothesis.stateful.RuleBasedStateMachine`来表示我的测试。这将允许我使用`Hypothesis`测试整个算法，同时检查不变量。有点复杂，所以我会先展示整段代码，然后逐部分解释。

```py
from functools import reduce
from hypothesis.strategies import integers
from hypothesis.stateful import Bundle, RuleBasedStateMachine, invariant, rule

class RecommendationChecker(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.recommender = MealRecommendationEngine()
        self.filters = []

    @rule(price_limit=integers(min_value=6, max_value=200))
    def filter_by_price(self, price_limit):
        self.recommender.apply_price_filter(price_limit)
        self.filters = [f for f in self.filters if f[0] != "price"]
        self.filters.append(("price", lambda m: m.price))

    @rule(calorie_limit=integers(min_value=500, max_value=2000))
    def filter_by_calories(self, calorie_limit):
        self.recommender.apply_calorie_filter(calorie_limit)
        self.filters = [f for f in self.filters if f[0] != "calorie"]
        self.filters.append(("calorie", lambda m: m.calories))

    @rule(distance_limit=integers(max_value=100))
    def filter_by_distance(self, distance_limit):
        self.recommender.apply_distance_filter(distance_limit)
        self.filters = [f for f in self.filters if f[0] != "distance"]
        self.filters.append(("distance", lambda m: m.distance))

    @invariant()
    def recommender_provides_three_unique_meals(self):
        assert len(self.recommender.get_meals()) == 3
        assert len(set(self.recommender.get_meals())) == 3

    @invariant()
    def meals_are_appropriately_ordered(self):
        meals = self.recommender.get_meals()
        ordered_meals = reduce(lambda meals, f: sorted(meals, key=f[1]),
                               self.filters,
                               meals)
        assert ordered_meals == meals

TestRecommender = RecommendationChecker.TestCase
```

这是相当多的代码，但它真的很酷，因为它是如何运作的。让我们逐步分解它。

首先，我将创建一个`hypothesis.stateful.RuleBasedStateMachine`的子类：

```py
from functools import reduce
from hypothesis.strategies import integers
from hypothesis.stateful import Bundle, RuleBasedStateMachine, invariant, rule

class RecommendationChecker(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.recommender = MealRecommendationEngine()
        self.filters = []
```

此类将负责定义我想要以组合形式测试的离散步骤。在构造函数中，我将`self.recommender`设置为`MealRecommendationEngine`，这是我在此场景中正在测试的内容。我还将跟踪作为此类的一部分应用的过滤器列表。接下来，我将设置`hypothesis.stateful.rule`函数：

```py
    @rule(price_limit=integers(min_value=6, max_value=200))
    def filter_by_price(self, price_limit):
        self.recommender.apply_price_filter(price_limit)
        self.filters = [f for f in self.filters if f[0] != "price"]
        self.filters.append(("price", lambda m: m.price))

    @rule(calorie_limit=integers(min_value=500, max_value=2000))
    def filter_by_calories(self, calorie_limit):
        self.recommender.apply_calorie_filter(calorie_limit)
        self.filters = [f for f in self.filters if f[0] != "calorie"]
        self.filters.append(("calorie", lambda m: m.calories))

    @rule(distance_limit=integers(max_value=100))
    def filter_by_distance(self, distance_limit):
        self.recommender.apply_distance_filter(distance_limit)
        self.filters = [f for f in self.filters if f[0] != "distance"]
        self.filters.append(("distance", lambda m: m.distance))
```

每个规则都充当您想要测试的算法的步骤。`Hypothesis`将使用这些规则生成测试，而不是生成测试数据。在这种情况下，每个规则都将一个过滤器应用于推荐引擎。我还将这些过滤器保存在本地，以便稍后检查结果。

然后，我使用`hypothesis.stateful.invariant`装饰器来定义应在每次规则更改后检查的断言。

```py
    @invariant()
    def recommender_provides_three_unique_meals(self):
        assert len(self.recommender.get_meals()) == 3
        # make sure all of the meals are unique - sets de-dupe elements
        # so we should have three unique elements
        assert len(set(self.recommender.get_meals())) == 3

    @invariant()
    def meals_are_appropriately_ordered(self):
        meals = self.recommender.get_meals()
        ordered_meals = reduce(lambda meals, f: sorted(meals, key=f[1]),
                               self.filters,
                               meals)
        assert ordered_meals == meals
```

我写了两个不变量：一个声明推荐器始终返回三个唯一的餐点，另一个声明这些餐点根据所选择的过滤器按正确的顺序排列。

最后，我将`RecommendationChecker`中的`TestCase`保存到一个以`Test`为前缀的变量中。这样做是为了让`pytest`能够发现这个有状态的`Hypothesis`测试。

```py
TestRecommender = RecommendationChecker.TestCase
```

一旦所有东西都组装好了，`Hypothesis`将开始生成具有不同规则组合的测试用例。例如，通过一个`Hypothesis`测试运行（故意引入错误），`Hypothesis`生成了以下测试。

```py
state = RecommendationChecker()
state.filter_by_distance(distance_limit=0)
state.filter_by_distance(distance_limit=0)
state.filter_by_distance(distance_limit=0)
state.filter_by_calories(calorie_limit=500)
state.filter_by_distance(distance_limit=0)
state.teardown()
```

当我引入不同的错误时，`Hypothesis`会展示给我一个不同的测试用例来捕获错误。

```py
state = RecommendationChecker()
state.filter_by_price(price_limit=6)
state.filter_by_price(price_limit=6)
state.filter_by_price(price_limit=6)
state.filter_by_price(price_limit=6)
state.filter_by_distance(distance_limit=0)
state.filter_by_price(price_limit=16)
state.teardown()
```

这对于测试复杂算法或具有非常特定不变量的对象非常方便。`Hypothesis`会混合匹配不同的步骤，不断寻找能产生错误的步骤顺序。

# 讨论主题

你的代码库中的哪些区域包含难以测试、高度相关的函数？写几个有状态的`Hypothesis`测试作为概念验证，并讨论这些测试如何增强你的测试套件的信心。

# 总结思考

基于属性的测试并不是为了取代传统测试；它是为了补充传统测试。当你的代码具有明确定义的输入和输出时，使用硬编码的前提条件和预期断言进行测试就足够了。然而，随着你的代码变得越来越复杂，你的测试也变得越来越复杂，你会发现自己花费的时间比想象中多，解析和理解测试。

在 Python 中，使用`Hypothesis`很容易实现基于属性的测试。它通过在代码库的整个生命周期内生成新的测试来修补你的安全网。你可以使用`hypothesis.strategies`来精确控制测试数据的生成方式。你甚至可以通过将不同的步骤组合进行`hypothesis.stateful`测试来测试算法。`Hypothesis`将让你专注于代码的属性和不变量，并更自然地表达你的测试。

在下一章中，我将用变异测试来结束本书。变异测试是填补安全网漏洞的另一种方法。与找到测试代码的新方法不同，变异代码专注于衡量你的测试的有效性。它是你更强大测试工具中的另一个工具。
