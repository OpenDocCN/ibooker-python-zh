# 第十章：哦哦：对象和类

> 没有神秘的对象。神秘的是你的眼睛。
> 
> 伊丽莎白·鲍文
> 
> 拿一个对象。对它做点什么。再对它做点别的事。
> 
> 贾斯珀·约翰斯

正如我在各个页面中提到的，Python 中的所有东西，从数字到函数，都是对象。但是，Python 通过特殊语法隐藏了大部分对象机制。你可以输入`num = 7`来创建一个类型为整数、值为 7 的对象，并将对象引用分配给名称`num`。只有当你想要创建自己的对象或修改现有对象的行为时，你才需要查看对象的内部。在本章中，你将看到如何执行这两个操作。

# 什么是对象？

一个*对象*是一个自定义的数据结构，包含数据（变量，称为*属性*）和代码（函数，称为*方法*）。它代表某个具体事物的唯一实例。把对象看作名词，它的方法是动词。一个对象代表一个个体，它的方法定义了它与其他事物的交互方式。

例如，值为`7`的整数对象是一个对象，可以执行加法和乘法等方法，就像你在第三章中看到的那样。`8`是另一个对象。这意味着 Python 中某处内置了一个整数类，`7`和`8`都属于这个类。字符串`'cat'`和`'duck'`也是 Python 中的对象，具有你在第五章中看到的字符串方法，如`capitalize()`和`replace()`。

与模块不同，你可以同时拥有多个对象（通常称为*实例*），每个对象可能具有不同的属性。它们就像是超级数据结构，内含代码。

# 简单的对象

让我们从基本对象类开始；我们将在几页后讨论继承。

## 使用 class 定义一个类

要创建一个以前从未创建过的新对象，首先需要定义一个*类*，指示它包含什么内容。

在第二章中，我将对象比作一个塑料盒。一个*类*就像是制造那个盒子的模具。例如，Python 有一个内置类，用来创建字符串对象如`'cat'`和`'duck'`，以及其他标准数据类型——列表、字典等等。要在 Python 中创建自定义对象，首先需要使用`class`关键字定义一个类。让我们通过一些简单的示例来详细了解。

假设你想定义对象来表示关于猫的信息。¹ 每个对象将代表一只猫。你首先需要定义一个名为`Cat`的类作为模板。在接下来的示例中，我们将尝试多个版本的这个类，从最简单的类逐步构建到真正有用的类。

###### 注意

我们遵循 Python 的命名约定[PEP-8](https://oreil.ly/gAJOF)。

我们的第一个尝试是最简单的类，一个空类：

```py
>>> class Cat():
...     pass
```

你也可以说：

```py
>>> class Cat:
...     pass
```

就像函数一样，我们需要使用`pass`来指示这个类是空的。这个定义是创建对象的最低要求。

通过像调用函数一样调用类名，你可以从类创建一个对象：

```py
>>> a_cat = Cat()
>>> another_cat = Cat()
```

在这种情况下，调用`Cat()`创建了两个来自`Cat`类的单独对象，并将它们分配给了名称`a_cat`和`another_cat`。但是我们的`Cat`类没有其他代码，所以我们从它创建的对象只是存在，不能做太多其他事情。

嗯，它们可以做一点点。

## 属性

*属性*是类或对象内部的变量。在创建对象或类期间以及之后，你可以给它赋予属性。属性可以是任何其他对象。让我们再次创建两个猫对象：

```py
>>> class Cat:
...     pass
...
>>> a_cat = Cat()
>>> a_cat
<__main__.Cat object at 0x100cd1da0>
>>> another_cat = Cat()
>>> another_cat
<__main__.Cat object at 0x100cd1e48>
```

当我们定义`Cat`类时，并没有指定如何打印来自该类的对象。Python 则会打印类似 `<__main__.Cat object at 0x100cd1da0>` 的东西。在“魔术方法”中，你会看到如何改变这个默认行为。

现在给我们的第一个对象分配一些属性：

```py
>>> a_cat.age = 3
>>> a_cat.name = "Mr. Fuzzybuttons"
>>> a_cat.nemesis = another_cat
```

我们能访问它们吗？我们当然希望如此：

```py
>>> a_cat.age
3
>>> a_cat.name
'Mr. Fuzzybuttons'
>>> a_cat.nemesis
<__main__.Cat object at 0x100cd1e48>
```

因为`nemesis`是指向另一个`Cat`对象的属性，我们可以使用`a_cat.nemesis`来访问它，但是这个其他对象还没有`name`属性：

```py
>>> a_cat.nemesis.name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Cat' object has no attribute 'name'
```

让我们给我们的大猫起个名字：

```py
>>> a_cat.nemesis.name = "Mr. Bigglesworth"
>>> a_cat.nemesis.name
'Mr. Bigglesworth'
```

即使是像这样的最简单的对象也可以用来存储多个属性。因此，你可以使用多个对象来存储不同的值，而不是使用类似列表或字典的东西。

当你听到*attributes*（属性）时，通常指的是对象的属性。还有*class attributes*（类属性），稍后你会在“类和对象属性”中看到它们的区别。

## 方法

*方法*是类或对象中的函数。方法看起来像任何其他函数，但可以用特殊的方式使用，你将在“属性访问的属性”和“方法类型”中看到。

## 初始化

如果你想在创建时分配对象属性，你需要使用特殊的 Python 对象初始化方法`__init__()`：

```py
>>> class Cat:
...     def __init__(self):
...         pass
```

这就是你在真实的 Python 类定义中会看到的内容。我承认`__init__()`和`self`看起来很奇怪。`__init__()`是 Python 中一个特殊的方法名，用于初始化一个类定义中的单个对象。² `self`参数指定它是指向个体对象本身。

当你在类定义中定义`__init__()`时，它的第一个参数应该命名为`self`。虽然`self`在 Python 中不是一个保留字，但它是常见的用法。如果你使用`self`，日后阅读你的代码的人（包括你自己！）不会猜测你的意图。

但是即使是这第二个`Cat`类定义也没有创建一个真正做任何事情的对象。第三次尝试是真正展示如何在 Python 中创建简单对象并分配其属性的方法。这次，我们将`name`参数添加到初始化方法中：

```py
>>> class Cat():
...     def __init__(self, name):
...         self.name = name
...
>>>
```

现在，我们可以通过为`name`参数传递一个字符串来从`Cat`类创建一个对象：

```py
>>> furball = Cat('Grumpy')
```

下面是这行代码的作用：

+   查找`Cat`类的定义

+   *实例化*（创建）一个新的内存对象

+   调用对象的`__init__()`方法，将这个新创建的对象作为`self`传递，并将另一个参数（`'Grumpy'`）作为`name`传递

+   将`name`的值存储在对象中

+   返回新对象

+   将变量`furball`附加到对象上

这个新对象像 Python 中的任何其他对象一样。你可以将它用作列表、元组、字典或集合的元素，可以将它作为参数传递给函数，或将它作为结果返回。

那么我们传入的`name`值呢？它以属性的形式保存在对象中。你可以直接读取和写入它：

```py
>>> print('Our latest addition: ', furball.name)
Our latest addition: Grumpy
```

记住，在`Cat`类定义的内部，你通过`self.name`访问`name`属性。当你创建一个实际对象并将其赋给像`furball`这样的变量时，你可以使用`furball.name`来引用它。

并非每个类定义都必须有一个`__init__()`方法；它用于执行任何需要区分此对象与同类其他对象的操作。它并不是某些其他语言所称的“构造函数”。Python 已经为你构造好了对象。将`__init__()`视为*初始化方法*。

###### 注意

你可以从一个类创建许多个体对象。但要记住，Python 将数据实现为对象，因此类本身也是一个对象。但是，在你的程序中只有一个类对象。如果像我们这里定义了`class Cat`，它就像《猎魔人》一样——只能有一个。

# 继承

当你试图解决某个编程问题时，通常会发现一个现有的类可以创建几乎符合你需求的对象。你能做什么？

你可以修改这个旧类，但会使它变得更加复杂，并且可能会破坏一些曾经工作的功能。

或者你可以编写一个新类，从旧类中剪切和粘贴代码并合并新代码。但这意味着你需要维护更多的代码，并且原来和新类中曾经相同的部分可能会因为它们现在位于不同的位置而有所不同。

一个解决方案是*继承*：从现有类创建一个新类，并进行一些添加或更改。这是代码重用的一个很好的方式。使用继承时，新类可以自动使用旧类的所有代码，而不需要你复制任何代码。

## 继承自父类

你只需定义新类中需要添加或更改的内容，这样就可以覆盖旧类的行为。原始类称为*父类*、*超类*或*基类*；新类称为*子类*、*子类*或*派生类*。这些术语在面向对象编程中是可以互换使用的。

所以，让我们继承一些东西。在下一个例子中，我们定义一个空类称为 `Car`。接下来，我们定义 `Car` 的一个子类称为 `Yugo`。³ 您可以使用相同的 `class` 关键字，但在括号中使用父类名称（这里是 `class Yugo(Car)`）来定义子类：

```py
>>> class Car():
...     pass
...
>>> class Yugo(Car):
...     pass
...
```

您可以使用 `issubclass()` 来检查一个类是否派生自另一个类：

```py
>>> issubclass(Yugo, Car)
True
```

接下来，从每个类创建一个对象：

```py
>>> give_me_a_car = Car()
>>> give_me_a_yugo = Yugo()
```

子类是父类的一种特殊化；在面向对象的术语中，`Yugo` *是一个* `Car`。名为 `give_me_a_yugo` 的对象是 `Yugo` 类的实例，但它也继承了 `Car` 的所有功能。在这种情况下，`Car` 和 `Yugo` 就像潜水艇上的水手一样有用，因此让我们尝试实际做点事的新类定义：

```py
>>> class Car():
...     def exclaim(self):
...         print("I'm a Car!")
...
>>> class Yugo(Car):
...     pass
...
```

最后，分别从每个类创建一个对象并调用 `exclaim` 方法：

```py
>>> give_me_a_car = Car()
>>> give_me_a_yugo = Yugo()
>>> give_me_a_car.exclaim()
I'm a Car!
>>> give_me_a_yugo.exclaim()
I'm a Car!
```

不做任何特殊处理，`Yugo` 继承了 `Car` 的 `exclaim()` 方法。事实上，`Yugo` 表示它*是*一辆 `Car`，这可能导致身份危机。让我们看看我们能做些什么。

###### 注意

继承很吸引人，但可能被滥用。多年的面向对象编程经验表明，过多使用继承会使程序难以管理。相反，通常建议强调其他技术，如聚合和组合。我们在本章中介绍这些替代方法。

## 覆盖一个方法

正如你刚才看到的，一个新类最初会从其父类继承所有东西。接下来，您将看到如何替换或覆盖父类方法。`Yugo` 可能在某种方式上应该与 `Car` 不同；否则，定义一个新类有什么意义？让我们改变 `exclaim()` 方法在 `Yugo` 中的工作方式：

```py
>>> class Car():
...     def exclaim(self):
...         print("I'm a Car!")
...
>>> class Yugo(Car):
...     def exclaim(self):
...         print("I'm a Yugo! Much like a Car, but more Yugo-ish.")
...
```

现在从这些类中创建两个对象：

```py
>>> give_me_a_car = Car()
>>> give_me_a_yugo = Yugo()
```

他们说什么？

```py
>>> give_me_a_car.exclaim()
I'm a Car!
>>> give_me_a_yugo.exclaim()
I'm a Yugo! Much like a Car, but more Yugo-ish.
```

在这些示例中，我们覆盖了 `exclaim()` 方法。我们可以覆盖任何方法，包括 `__init__()`。这里有另一个使用 `Person` 类的例子。让我们创建代表医生（`MDPerson`）和律师（`JDPerson`）的子类：

```py
>>> class Person():
...     def __init__(self, name):
...         self.name = name
...
>>> class MDPerson(Person):
...     def __init__(self, name):
...         self.name = "Doctor " + name
...
>>> class JDPerson(Person):
...     def __init__(self, name):
...         self.name = name + ", Esquire"
...
```

在这些情况下，初始化方法 `__init__()` 接受与父类 `Person` 相同的参数，但在对象实例内部以不同的方式存储 `name` 的值：

```py
>>> person = Person('Fudd')
>>> doctor = MDPerson('Fudd')
>>> lawyer = JDPerson('Fudd')
>>> print(person.name)
Fudd
>>> print(doctor.name)
Doctor Fudd
>>> print(lawyer.name)
Fudd, Esquire
```

## 添加一个方法

子类还可以*添加*在其父类中不存在的方法。回到 `Car` 和 `Yugo` 类，我们将为仅 `Yugo` 类定义新方法 `need_a_push()`：

```py
>>> class Car():
...     def exclaim(self):
...         print("I'm a Car!")
...
>>> class Yugo(Car):
...     def exclaim(self):
...         print("I'm a Yugo! Much like a Car, but more Yugo-ish.")
...     def need_a_push(self):
...         print("A little help here?")
...
```

接下来，创建一个 `Car` 和一个 `Yugo`：

```py
>>> give_me_a_car = Car()
>>> give_me_a_yugo = Yugo()
```

`Yugo` 对象可以对 `need_a_push()` 方法调用做出反应：

```py
>>> give_me_a_yugo.need_a_push()
A little help here?
```

但是一个普通的 `Car` 对象不能：

```py
>>> give_me_a_car.need_a_push()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Car' object has no attribute 'need_a_push'
```

到这一点，一个 `Yugo` 能做一些 `Car` 不能的事情，`Yugo` 的独特个性可以显现出来。

## 通过 `super()` 从父类获得帮助

我们看到子类如何添加或覆盖父类的方法。如果它想调用那个父类方法呢？“我很高兴你问”，`super()` 这里，我们定义了一个名为 `EmailPerson` 的新类，代表一个带有电子邮件地址的 `Person`。首先是我们熟悉的 `Person` 定义：

```py
>>> class Person():
...     def __init__(self, name):
...         self.name = name
...
```

请注意，以下子类中的`__init__()`调用具有额外的`email`参数：

```py
>>> class EmailPerson(Person):
...     def __init__(self, name, email):
...         super().__init__(name)
...         self.email = email
```

当为类定义`__init__()`方法时，您正在替换其父类的`__init__()`方法，后者不再自动调用。因此，我们需要显式调用它。以下是发生的情况：

+   `super()`获取父类`Person`的定义。

+   `__init__()`方法调用了`Person.__init__()`方法。它负责将`self`参数传递给超类，因此您只需提供任何可选参数。在我们的情况下，`Person()`接受的唯一其他参数是`name`。

+   `self.email = email`行是使这个`EmailPerson`与`Person`不同的新代码。

继续，让我们制作其中的一个生物：

```py
>>> bob = EmailPerson('Bob Frapples', 'bob@frapples.com')
```

我们应该能够访问`name`和`email`属性：

```py
>>> bob.name
'Bob Frapples'
>>> bob.email
'bob@frapples.com'
```

为什么我们不直接定义我们的新类如下？

```py
>>> class EmailPerson(Person):
...     def __init__(self, name, email):
...         self.name = name
...         self.email = email
```

我们本可以这样做，但那样会破坏我们对继承的使用。我们使用`super()`使`Person`执行其工作，就像一个普通的`Person`对象一样。还有另一个好处：如果将来`Person`的定义发生变化，使用`super()`将确保`EmailPerson`从`Person`继承的属性和方法将反映这些变化。

当子类以自己的方式执行某些操作但仍需要来自父类的东西（就像现实生活中一样）时，请使用`super()`。

## 多重继承

你刚刚看到一些没有父类的经典示例，还有一些有一个父类的示例。实际上，对象可以从多个父类继承。

如果您的类引用其没有的方法或属性，Python 将在所有父类中查找。如果其中有多个类具有相同名称的东西，谁会胜出？

与人类的遗传不同，在那里无论来自谁的优势基因都会获胜，Python 中的继承取决于*方法解析顺序*。每个 Python 类都有一个特殊的方法称为`mro()`，返回一个访问该类对象的方法或属性时要访问的类的列表。类似的属性称为`__mro__`，是这些类的元组。就像突然死亡的季后赛一样，第一个赢家胜出。

在这里，我们定义了一个顶级`Animal`类，两个子类（`Horse`和`Donkey`），然后从这些类派生出两个：⁴

```py
>>> class Animal:
...     def says(self):
 return 'I speak!'
...
>>> class Horse(Animal):
...     def says(self):
...         return 'Neigh!'
...
>>> class Donkey(Animal):
...     def says(self):
...         return 'Hee-haw!'
...
>>> class Mule(Donkey, Horse):
...     pass
...
>>> class Hinny(Horse, Donkey):
...     pass
...
```

如果我们查找`Mule`的方法或属性，Python 将按照以下顺序查找：

1.  对象本身（类型为`Mule`）

1.  对象的类（`Mule`）

1.  类的第一个父类（`Donkey`）

1.  类的第二个父类（`Horse`）

1.  祖父类（`Animal`）类

对于`Hinny`来说，情况大致相同，但是`Horse`在`Donkey`之前：

```py
>>> Mule.mro()
[<class '__main__.Mule'>, <class '__main__.Donkey'>,
<class '__main__.Horse'>, <class '__main__.Animal'>,
<class 'object'>]
>>> Hinny.mro()
[<class '__main__.Hinny'>, <class '__main__.Horse'>,
<class '__main__.Donkey'>, <class '__main__.Animal'>,
class 'object'>]
```

那么这些优雅的动物怎么说呢？

```py
>>> mule = Mule()
>>> hinny = Hinny()
>>> mule.says()
'hee-haw'
>>> hinny.says()
'neigh'
```

我们按（父亲，母亲）顺序列出了父类，所以它们说话像他们的爸爸一样。

如果`Horse`和`Donkey`没有`says()`方法，那么骡或骡马将使用祖父类`Animal`类的`says()`方法，并返回`'I speak!'`。

## Mixins

您可以在类定义中包含一个额外的父类，但仅作为助手使用。也就是说，它不与其他父类共享任何方法，并避免了我在上一节中提到的方法解析歧义。

这样的父类有时被称为*mixin*类。用途可能包括像日志记录这样的“副”任务。这里是一个漂亮打印对象属性的 mixin：

```py
>>> class PrettyMixin():
...     def dump(self):
...         import pprint
...         pprint.pprint(vars(self))
...
>>> class Thing(PrettyMixin):
...     pass
...
>>> t = Thing()
>>> t.name = "Nyarlathotep"
>>> t.feature = "ichor"
>>> t.age = "eldritch"
>>> t.dump()
{'age': 'eldritch', 'feature': 'ichor', 'name': 'Nyarlathotep'}
```

# 在自卫中

除了使用空格之外，Python 的一个批评是需要将`self`作为实例方法的第一个参数（您在前面的示例中看到的方法类型）。Python 使用`self`参数来查找正确的对象属性和方法。例如，我将展示如何调用对象的方法，以及 Python 在幕后实际做了什么。

还记得之前例子中的`Car`类吗？让我们再次调用它的`exclaim()`方法：

```py
>>> a_car = Car()
>>> a_car.exclaim()
I'm a Car!
```

下面是 Python 在幕后实际做的事情：

+   查找对象`a_car`的类（`Car`）。

+   将对象`a_car`作为`self`参数传递给`Car`类的`exclaim()`方法。

只是为了好玩，您甚至可以以这种方式自行运行它，它将与正常的（`a_car.exclaim()`）语法相同运行：

```py
>>> Car.exclaim(a_car)
I'm a Car!
```

然而，永远没有理由使用那种更冗长的风格。

# 属性访问

在 Python 中，对象的属性和方法通常是公开的，您被期望自律行事（这有时被称为“成年人同意”政策）。让我们比较直接方法与一些替代方法。

## 直接访问

正如您所见，您可以直接获取和设置属性值：

```py
>>> class Duck:
...     def __init__(self, input_name):
...         self.name = input_name
...
>>> fowl = Duck('Daffy')
>>> fowl.name
'Daffy'
```

但是如果有人行为不端呢？

```py
>>> fowl.name = 'Daphne'
>>> fowl.name
'Daphne'
```

接下来的两个部分展示了如何为不希望被意外覆盖的属性获取一些隐私。

## Getter 和 Setter

一些面向对象的语言支持私有对象属性，这些属性无法从外部直接访问。程序员可能需要编写*getter*和*setter*方法来读取和写入这些私有属性的值。

Python 没有私有属性，但是您可以使用名称混淆的 getter 和 setter 来获得一些隐私。（最佳解决方案是使用下一节描述的*属性*。）

在下面的示例中，我们定义了一个名为`Duck`的类，具有一个名为`hidden_name`的单个实例属性。我们不希望直接访问这个属性，因此我们定义了两个方法：一个 getter（`get_name()`）和一个 setter（`set_name()`）。每个方法都通过名为`name`的属性访问。我在每个方法中添加了一个`print()`语句，以显示它何时被调用：

```py
>>> class Duck():
...     def __init__(self, input_name):
...         self.hidden_name = input_name
...     def get_name(self):
...         print('inside the getter')
...         return self.hidden_name
...     def set_name(self, input_name):
...         print('inside the setter')
...         self.hidden_name = input_name
```

```py
>>> don = Duck('Donald')
>>> don.get_name()
inside the getter
'Donald'
>>> don.set_name('Donna')
inside the setter
>>> don.get_name()
inside the getter
'Donna'
```

## 用于属性访问的属性

对于属性隐私的 Python 解决方案是使用*属性*。

有两种方法可以做到这一点。第一种方法是将`name = property(get_name, set_name)`添加为我们之前`Duck`类定义的最后一行：

```py
>>> class Duck():
>>>     def __init__(self, input_name):
>>>         self.hidden_name = input_name
>>>     def get_name(self):
>>>         print('inside the getter')
>>>         return self.hidden_name
>>>     def set_name(self, input_name):
>>>         print('inside the setter')
>>>         self.hidden_name = input_name
>>>     name = property(get_name, set_name)
```

旧的 getter 和 setter 仍然有效：

```py
>>> don = Duck('Donald')
>>> don.get_name()
inside the getter
'Donald'
>>> don.set_name('Donna')
inside the setter
>>> don.get_name()
inside the getter
'Donna'
```

现在您还可以使用属性`name`来获取和设置隐藏的名称：

```py
>>> don = Duck('Donald')
>>> don.name
inside the getter
'Donald'
>>> don.name = 'Donna'
inside the setter
>>> don.name
inside the getter
'Donna'
```

在第二种方法中，您添加了一些装饰器，并用`name`替换了方法名`get_name`和`set_name`：

+   `@property`，放在 getter 方法之前

+   `@*name*.setter`，放在 setter 方法之前

这是它们在代码中的实际表现：

```py
>>> class Duck():
...     def __init__(self, input_name):
...         self.hidden_name = input_name
...     @property
...     def name(self):
...         print('inside the getter')
...         return self.hidden_name
...     @name.setter
...     def name(self, input_name):
...         print('inside the setter')
...         self.hidden_name = input_name
```

你仍然可以像访问属性一样访问`name`：

```py
>>> fowl = Duck('Howard')
>>> fowl.name
inside the getter
'Howard'
>>> fowl.name = 'Donald'
inside the setter
>>> fowl.name
inside the getter
'Donald'
```

###### 注意

如果有人猜到我们称呼我们的属性为`hidden_name`，他们仍然可以直接作为`fowl.hidden_name`读取和写入它。在“隐私的名称混淆”中，你将看到 Python 提供了一种特殊的方式来隐藏属性名称。

## 用于计算值的属性

在之前的例子中，我们使用`name`属性来引用存储在对象内部的单个属性(`hidden_name`)。

属性还可以返回一个*计算值*。让我们定义一个`Circle`类，它有一个`radius`属性和一个计算出的`diameter`属性：

```py
>>> class Circle():
...     def __init__(self, radius):
...         self.radius = radius
...     @property
...     def diameter(self):
...         return 2 * self.radius
...
```

创建一个带有其`radius`初始值的`Circle`对象：

```py
>>> c = Circle(5)
>>> c.radius
5
```

我们可以将`diameter`称为像`radius`这样的属性：

```py
>>> c.diameter
10
```

这是有趣的一部分：我们可以随时改变`radius`属性，并且`diameter`属性将从当前的`radius`值计算出来：

```py
>>> c.radius = 7
>>> c.diameter
14
```

如果你没有为属性指定一个 setter 属性，你不能从外部设置它。这对于只读属性很方便：

```py
>>> c.diameter = 20
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
```

使用属性而不是直接访问属性有另一个优点：如果你改变属性的定义，你只需要修复类定义内的代码，而不是所有的调用者。

## 用于隐私的名称混淆

在稍早的`Duck`类示例中，我们称呼我们的（不完全）隐藏属性为`hidden_name`。Python 有一个属性命名约定，这些属性不应该在其类定义之外可见：以双下划线(`__`)开头。

让我们将`hidden_name`重命名为`__name`，如此所示：

```py
>>> class Duck():
...     def __init__(self, input_name):
...         self.__name = input_name
...     @property
...     def name(self):
...         print('inside the getter')
...         return self.__name
...     @name.setter
...     def name(self, input_name):
...         print('inside the setter')
...         self.__name = input_name
...
```

看一看是否一切仍在正常工作：

```py
>>> fowl = Duck('Howard')
>>> fowl.name
inside the getter
'Howard'
>>> fowl.name = 'Donald'
inside the setter
>>> fowl.name
inside the getter
'Donald'
```

看起来不错。而且你不能访问`__name`属性：

```py
>>> fowl.__name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Duck' object has no attribute '__name'
```

这种命名约定并不完全使其私有，但 Python 确实*mangle*属性名称以使外部代码不太可能偶然发现它。如果你好奇并且承诺不告诉每个人，⁵ 这里是它变成什么样子：

```py
>>> fowl._Duck__name
'Donald'
```

请注意，它没有打印`inside the getter`。虽然这并不是完美的保护，但名称混淆阻止了对属性的意外或故意的直接访问。

## 类和对象属性

您可以将属性分配给类，并且它们将被其子对象继承：

```py
>>> class Fruit:
...     color = 'red'
...
>>> blueberry = Fruit()
>>> Fruit.color
'red'
>>> blueberry.color
'red'
```

但是如果您更改子对象中属性的值，则不会影响类属性：

```py
>>> blueberry.color = 'blue'
>>> blueberry.color
'blue'
>>> Fruit.color
'red'
```

如果您稍后更改类属性，它不会影响现有的子对象：

```py
>>> Fruit.color = 'orange'
>>> Fruit.color
'orange'
>>> blueberry.color
'blue'
```

但这将影响到新的：

```py
>>> new_fruit = Fruit()
>>> new_fruit.color
'orange'
```

# 方法类型

一些方法是类本身的一部分，一些是从该类创建的对象的一部分，还有一些都不是：

+   如果没有前置装饰器，那么它是一个*实例方法*，其第一个参数应该是`self`，用于引用对象本身。

+   如果有一个前置的`@classmethod`装饰器，那么它是一个*类方法*，其第一个参数应该是`cls`（或任何其他，只要不是保留字`class`），用于指代类本身。

+   如果有一个前置的`@staticmethod`装饰器，那么它是一个*静态方法*，它的第一个参数不是对象或类。

以下各节有一些详细信息。

## 实例方法

当你在类定义中的方法中看到初始的`self`参数时，它是一个*实例方法*。这些是您通常在创建自己的类时编写的方法类型。实例方法的第一个参数是`self`，当您调用它时，Python 会将对象传递给方法。到目前为止，您已经看到了这些。

## 类方法

相反，*类方法*影响整个类。对类进行的任何更改都会影响其所有对象。在类定义中，前置的`@classmethod`装饰器表示随后的函数是一个类方法。此方法的第一个参数也是类本身。Python 的传统是将参数称为`cls`，因为`class`是一个保留字，不能在这里使用。让我们为`A`定义一个类方法来统计有多少个对象实例已经创建了：

```py
>>> class A():
...     count = 0
...     def __init__(self):
...         A.count += 1
...     def exclaim(self):
...         print("I'm an A!")
...     @classmethod
...     def kids(cls):
...         print("A has", cls.count, "little objects.")
...
>>>
>>> easy_a = A()
>>> breezy_a = A()
>>> wheezy_a = A()
>>> A.kids()
A has 3 little objects.
```

注意在`__init__()`中我们引用了`A.count`（类属性），而不是`self.count`（这将是一个对象实例属性）。在`kids()`方法中，我们使用了`cls.count`，但我们也可以使用`A.count`。

## 静态方法

类定义中的第三种方法既不影响类也不影响其对象；它只是为了方便而存在，而不是漂浮在自己周围。它是一个*静态方法*，前面有一个`@staticmethod`装饰器，没有初始的`self`或`cls`参数。以下是作为`CoyoteWeapon`类的商业广告的示例：

```py
>>> class CoyoteWeapon():
...     @staticmethod
...     def commercial():
...         print('This CoyoteWeapon has been brought to you by Acme')
...
>>>
>>> CoyoteWeapon.commercial()
This CoyoteWeapon has been brought to you by Acme
```

注意，我们不需要从`CoyoteWeapon`类创建对象来访问此方法。非常“class-y”。

# 鸭子类型

Python 对*多态性*有着宽松的实现；它根据方法的名称和参数，无论它们的类如何，都将相同的操作应用于不同的对象。

现在让我们为所有三个`Quote`类使用相同的`__init__()`初始化程序，但添加两个新函数：

+   `who()`只返回保存的`person`字符串的值

+   `says()`返回具有特定标点符号的保存的`words`字符串

现在让我们看看它们的运作方式：

```py
>>> class Quote():
...     def __init__(self, person, words):
...         self.person = person
...         self.words = words
...     def who(self):
...         return self.person
...     def says(self):
...         return self.words + '.'
...
>>> class QuestionQuote(Quote):
...      def says(self):
...          return self.words + '?'
...
>>> class ExclamationQuote(Quote):
...      def says(self):
...          return self.words + '!'
...
>>>
```

我们没有改变`QuestionQuote`或`ExclamationQuote`的初始化方式，因此我们没有重写它们的`__init__()`方法。然后 Python 会自动调用父类`Quote`的`__init__()`方法来存储实例变量`person`和`words`。这就是为什么我们可以在从子类`QuestionQuote`和`ExclamationQuote`创建的对象中访问`self.words`的原因。

接下来，让我们创建一些对象：

```py
>>> hunter = Quote('Elmer Fudd', "I'm hunting wabbits")
>>> print(hunter.who(), 'says:', hunter.says())
Elmer Fudd says: I'm hunting wabbits.
```

```py
>>> hunted1 = QuestionQuote('Bugs Bunny', "What's up, doc")
>>> print(hunted1.who(), 'says:', hunted1.says())
Bugs Bunny says: What's up, doc?
```

```py
>>> hunted2 = ExclamationQuote('Daffy Duck', "It's rabbit season")
>>> print(hunted2.who(), 'says:', hunted2.says())
Daffy Duck says: It's rabbit season!
```

`says()` 方法的三个不同版本为三个类提供了不同的行为。这是面向对象语言中的传统多态性。Python 更进一步，让你运行具有这些方法的 *任何* 对象的 `who()` 和 `says()` 方法。让我们定义一个名为 `BabblingBrook` 的类，它与我们之前的树林猎人和被猎物（`Quote` 类的后代）没有关系：

```py
>>> class BabblingBrook():
...     def who(self):
...         return 'Brook'
...     def says(self):
...         return 'Babble'
...
>>> brook = BabblingBrook()
```

现在运行各种对象的 `who()` 和 `says()` 方法，其中一个（`brook`）与其他对象完全无关：

```py
>>> def who_says(obj):
...     print(obj.who(), 'says', obj.says())
...
>>> who_says(hunter)
Elmer Fudd says I'm hunting wabbits.
>>> who_says(hunted1)
Bugs Bunny says What's up, doc?
>>> who_says(hunted2)
Daffy Duck says It's rabbit season!
>>> who_says(brook)
Brook says Babble
```

有时这种行为被称为 *鸭子类型*，来自一句古老的谚语：

> 如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子。
> 
> 一个聪明人

我们能否对鸭子的一句智慧的话提出异议？

![inp2 1001](img/inp2_1001.png)

###### 图 10-1\. 鸭子类型并非按字母顺序寻找

# 魔术方法

现在你可以创建并使用基本对象了。你将在本节中学到的内容可能会让你感到惊讶——是一种好的方式。

当你键入诸如 `a = 3 + 8` 这样的内容时，整数对象如何知道如何实现 `+` 呢？或者，如果你键入 `name = "Daffy" + " " + "Duck"`，Python 如何知道 `+` 现在意味着将这些字符串连接起来？`a` 和 `name` 如何知道如何使用 `=` 来得到结果？你可以通过使用 Python 的 *特殊方法*（或者更戏剧化地说，*魔术方法*）来解决这些运算符。

这些方法的名称以双下划线 (`__`) 开头和结尾。为什么呢？它们极不可能被程序员选为变量名。你已经见过一个：`__init__()` 从其类定义和传入的任何参数中初始化一个新创建的对象。你还见过（“用于隐私的名称混淆”）“dunder”命名如何帮助混淆类属性名称以及方法。

假设你有一个简单的 `Word` 类，并且你想要一个 `equals()` 方法来比较两个单词但忽略大小写。也就是说，包含值 `'ha'` 的 `Word` 将被视为等于包含 `'HA'` 的一个。

接下来的示例是一个首次尝试，使用一个我们称之为 `equals()` 的普通方法。`self.text` 是这个 `Word` 对象包含的文本字符串，`equals()` 方法将其与 `word2` 的文本字符串（另一个 `Word` 对象）进行比较：

```py
>>> class Word():
...    def __init__(self, text):
...        self.text = text
...
...    def equals(self, word2):
...        return self.text.lower() == word2.text.lower()
...
```

然后，从三个不同的文本字符串创建三个 `Word` 对象：

```py
>>> first = Word('ha')
>>> second = Word('HA')
>>> third = Word('eh')
```

当字符串 `'ha'` 和 `'HA'` 与小写比较时，它们应该相等：

```py
>>> first.equals(second)
True
```

但字符串 `'eh'` 将不匹配 `'ha'`：

```py
>>> first.equals(third)
False
```

我们定义了方法 `equals()` 来进行这个小写转换和比较。只需说 `if first == second` 就好了，就像 Python 的内置类型一样。那么，我们就这样做吧。我们将 `equals()` 方法更改为特殊名称 `__eq__()`（一会儿你就会明白为什么）：

```py
>>> class Word():
...     def __init__(self, text):
...         self.text = text
...     def __eq__(self, word2):
...         return self.text.lower() == word2.text.lower()
...
```

让我们看看它是否奏效：

```py
>>> first = Word('ha')
>>> second = Word('HA')
>>> third = Word('eh')
>>> first == second
True
>>> first == third
False
```

太神奇了！我们只需要 Python 的特殊方法名称来测试相等性，`__eq__()`。表 10-1 和 10-2 列出了最有用的魔术方法的名称。

Table 10-1\. 比较的魔术方法

| 方法 | 描述 |
| --- | --- |
| `__eq__(` *self*, *other* `)` | *self* `==` *other* |
| `__ne__(` *self*, *other* `)` | *self* `!=` *other* |
| `__lt__(` *self*, *other* `)` | *self* `<` *other* |
| `__gt__(` *self*, *other* `)` | *self* `>` *other* |
| `__le__(` *self*, *other* `)` | *self* `<=` *other* |
| `__ge__(` *self*, *other* `)` | *self* `>=` *other* |

Table 10-2\. 数学运算的魔术方法

| 方法 | 描述 |
| --- | --- |
| `__add__(` *self*, *other* `)` | *self* `+` *other* |
| `__sub__(` *self*, *other* `)` | *self* `–` *other* |
| `__mul__(` *self*, *other* `)` | *self* `*` *other* |
| `__floordiv__(` *self*, *other* `)` | *self* `//` *other* |
| `__truediv__(` *self*, *other* `)` | *self* `/` *other* |
| `__mod__(` *self*, *other* `)` | *self* `%` *other* |
| `__pow__(` *self*, *other* `)` | *self* `**` *other* |

您并不受限于使用数学运算符，如`+`（魔术方法`__add__()`）和`–`（魔术方法`__sub__()`）与数字。例如，Python 字符串对象使用`+`进行连接和`*`进行复制。还有许多其他方法，在[特殊方法名称](http://bit.ly/pydocs-smn)中有详细记录。其中最常见的方法在表 10-3 中呈现。

Table 10-3\. 其他杂项魔术方法

| 方法 | 描述 |
| --- | --- |
| `__str__(` *self* `)` | `str(` *self* `)` |
| `__repr__(` *self* `)` | `repr(` *self* `)` |
| `__len__(` *self* `)` | `len(` *self* `)` |

除了`__init__()`之外，您可能会发现自己在自己的方法中最常使用`__str__()`。这是您打印对象的方式。它被`print()`、`str()`和字符串格式化器使用，您可以在第五章中了解更多。交互式解释器使用`__repr__()`函数将变量回显到输出。如果您未定义`__str__()`或`__repr__()`中的任何一个，您将获得 Python 对象的默认字符串版本：

```py
>>> first = Word('ha')
>>> first
<__main__.Word object at 0x1006ba3d0>
>>> print(first)
<__main__.Word object at 0x1006ba3d0>
```

让我们为`Word`类添加`__str__()`和`__repr__()`方法，使其更加美观：

```py
>>> class Word():
...     def __init__(self, text):
...         self.text = text
...     def __eq__(self, word2):
...         return self.text.lower() == word2.text.lower()
...     def __str__(self):
...         return self.text
...     def __repr__(self):
...         return 'Word("'  + self.text  + '")'
...
>>> first = Word('ha')
>>> first          # uses __repr__
Word("ha")
>>> print(first)   # uses __str__
ha
```

要深入了解更多特殊方法，请查看[Python 文档](http://bit.ly/pydocs-smn)。

# 聚合与组合

继承是一种很好的技术，当你希望子类在大多数情况下像其父类一样时使用（当子类 *是一个* 父类）。在构建精细的继承层次时很诱人，但有时*组合*或*聚合*更有意义。它们的区别是什么？在组合中，一个东西是另一个东西的一部分。一只鸭子 *是一个* 鸟（继承），但 *有一个* 尾巴（组合）。尾巴不是鸭子的一种，而是鸭子的一部分。在下面的例子中，让我们创建`bill`和`tail`对象，并将它们提供给一个新的`duck`对象：

```py
>>> class Bill():
...     def __init__(self, description):
...         self.description = description
...
>>> class Tail():
...     def __init__(self, length):
...         self.length = length
...
>>> class Duck():
...     def __init__(self, bill, tail):
...         self.bill = bill
...         self.tail = tail
...     def about(self):
...         print('This duck has a', self.bill.description,
...             'bill and a', self.tail.length, 'tail')
...
>>> a_tail = Tail('long')
>>> a_bill = Bill('wide orange')
>>> duck = Duck(a_bill, a_tail)
>>> duck.about()
This duck has a wide orange bill and a long tail
```

聚合表达了关系，但要松散一些：一个东西 *使用* 另一个东西，但两者都是独立存在的。一只鸭子 *使用* 一片湖水，但它们并非彼此的一部分。

# 对象或其他东西何时使用

这里有一些指南，帮助你决定是否将代码和数据放入类、模块（在第十一章中讨论）或其他某些地方：

+   当你需要许多具有相似行为（方法）但在内部状态（属性）上不同的个体实例时，对象是最有用的。

+   类支持继承，而模块不支持。

+   如果只需要一个东西，模块可能是最好的选择。无论在程序中引用多少次 Python 模块，只加载一个副本。（Java 和 C++程序员：你可以将 Python 模块用作*单例*。）

+   如果你有多个变量，其中包含多个值并且可以作为多个函数的参数传递，那么将它们定义为类可能更好。例如，你可以使用一个带有`size`和`color`等键的字典来表示彩色图像。你可以为程序中的每个图像创建一个不同的字典，并将它们作为参数传递给`scale()`或`transform()`等函数。随着键和函数的增加，这可能会变得混乱。定义一个`Image`类更加一致，包含`size`或`color`属性以及`scale()`和`transform()`方法会更好。这样，所有与彩色图像相关的数据和方法都定义在同一个地方。

+   使用问题的最简解决方案。字典、列表或元组比模块更简单、更小、更快，通常比类更简单。

    Guido 的建议：

    > 避免过度工程化的数据结构。元组比对象更好（尝试使用命名元组也是如此）。优先选择简单字段而不是 getter/setter 函数……内置数据类型是你的朋友。使用更多的数字、字符串、元组、列表、集合、字典。还要查看 collections 库，特别是 deque。
    > 
    > Guido van Rossum

+   一个更新的替代方案是*数据类*，在“数据类”中。

# 命名元组

因为 Guido 刚提到它们，而我还没有，这是谈论*命名元组*的好地方。命名元组是元组的一个子类，可以通过名称（使用*`.name`*）以及位置（使用*`[偏移]`*）访问值。

让我们从上一节的示例中获取示例，并将`Duck`类转换为一个命名元组，其中`bill`和`tail`是简单的字符串属性。我们将调用`namedtuple`函数并传入两个参数：

+   名称

+   字段名称的字符串，用空格分隔

Python 不会自动提供命名元组，因此在使用它们之前需要加载一个模块。我们在以下示例的第一行中执行了这样的操作：

```py
>>> from collections import namedtuple
>>> Duck = namedtuple('Duck', 'bill tail')
>>> duck = Duck('wide orange', 'long')
>>> duck
Duck(bill='wide orange', tail='long')
>>> duck.bill
'wide orange'
>>> duck.tail
'long'
```

你也可以从一个字典中创建一个命名元组：

```py
>>> parts = {'bill': 'wide orange', 'tail': 'long'}
>>> duck2 = Duck(**parts)
>>> duck2
Duck(bill='wide orange', tail='long')
```

在上面的代码中，看看`**parts`。这是一个*关键字参数*。它从`parts`字典中提取键和值，并将它们作为参数提供给`Duck()`。它与以下代码具有相同的效果：

```py
>>> duck2 = Duck(bill = 'wide orange', tail = 'long')
```

命名元组是不可变的，但可以替换一个或多个字段并返回另一个命名元组：

```py
>>> duck3 = duck2._replace(tail='magnificent', bill='crushing')
>>> duck3
Duck(bill='crushing', tail='magnificent')
```

我们本可以将`duck`定义为一个字典：

```py
>>> duck_dict = {'bill': 'wide orange', 'tail': 'long'}
>>> duck_dict
{'tail': 'long', 'bill': 'wide orange'}
```

你可以向字典添加字段：

```py
>>> duck_dict['color'] = 'green'
>>> duck_dict
{'color': 'green', 'tail': 'long', 'bill': 'wide orange'}
```

但不是一个命名元组：

```py
>>> duck.color = 'green'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Duck' object has no attribute 'color'
```

总结一下，以下是命名元组的一些优点：

+   它看起来和表现得像一个不可变对象。

+   它比对象更省空间和时间。

+   你可以使用点符号而不是类似字典风格的方括号来访问属性。

+   你可以将其用作字典键。

# 数据类

许多人喜欢创建对象主要用于存储数据（作为对象属性），而不是行为（方法）。刚才你看到命名元组可以作为替代数据存储的一种选择。Python 3.7 引入了*数据类*。

这是一个普通的对象，只有一个名为`name`的属性：

```py
>> class TeenyClass():
...     def __init__(self, name):
...         self.name = name
...
>>> teeny = TeenyClass('itsy')
>>> teeny.name
'itsy'
```

使用数据类做同样的事情看起来有些不同：

```py
>>> from dataclasses import dataclass
>>> @dataclass
... class TeenyDataClass:
...     name: str
...
>>> teeny = TeenyDataClass('bitsy')
>>> teeny.name
'bitsy'
```

除了需要一个`@dataclass`装饰器外，你还需要使用[*变量注释*](https://oreil.ly/NyGfE)的形式定义类的属性，如`*name*: *type*`或`*name*: *type* = *val*`，比如`color: str`或`color: str = "red"`。`*type*`可以是任何 Python 对象类型，包括你创建的类，而不仅限于像`str`或`int`这样的内置类型。

当你创建数据类对象时，可以按照类中指定的顺序提供参数，或者以任意顺序使用命名参数：

```py
>>> from dataclasses import dataclass
>>> @dataclass
... class AnimalClass:
...     name: str
...     habitat: str
...     teeth: int = 0
...
>>> snowman = AnimalClass('yeti', 'Himalayas', 46)
>>> duck = AnimalClass(habitat='lake', name='duck')
>>> snowman
AnimalClass(name='yeti', habitat='Himalayas', teeth=46)
>>> duck
AnimalClass(name='duck', habitat='lake', teeth=0)
```

`AnimalClass`为其`teeth`属性定义了默认值，因此在创建`duck`时无需提供它。

你可以像访问任何其他对象一样引用对象属性：

```py
>>> duck.habitat
'lake'
>>> snowman.teeth
46
```

数据类还有很多功能。参见这篇[指南](https://oreil.ly/czTf-)或官方（详尽的）[文档](https://oreil.ly/J19Yl)。

# 属性

你已经看到如何创建类并添加属性，以及它们可能涉及大量打字的事情，比如定义`__init__()`，将其参数分配给`self`的对应项，并创建所有这些 dunder 方法，如`__str__()`。命名元组和数据类是标准库中的替代品，当你主要想创建数据集时可能更容易。

[每个人都需要的一个 Python 库](https://oreil.ly/QbbI1)比较了普通类、命名元组和数据类。它推荐第三方包[`attrs`](https://oreil.ly/Rdwlx)，原因很多——少打字、数据验证等等。看一看，看看你是否喜欢它胜过内置解决方案。

# 即将到来

在下一章中，你将在代码结构中进一步提升到 Python 的*模块*和*包*。

# 要做的事情

10.1 创建一个名为`Thing`的类，不包含内容并打印它。然后，从这个类创建一个名为`example`的对象并打印它。打印的值是相同的还是不同的？

10.2 创建一个名为`Thing2`的新类，并将值`'abc'`赋给名为`letters`的类属性。打印`letters`。

10.3 再次创建一个名为`Thing3`的类，这次将值`'xyz'`赋给名为`letters`的实例（对象）属性。打印`letters`。你需要从该类创建一个对象吗？

10.4 创建一个名为`Element`的类，具有实例属性`name`、`symbol`和`number`。使用值`'Hydrogen'`、`'H'`和`1`创建该类的一个对象。

10.5 使用这些键和值创建一个字典：`'name': 'Hydrogen', 'symbol': 'H', 'number': 1`。然后，使用这个字典从`Element`类创建一个名为`hydrogen`的对象。

10.6 对于`Element`类，定义一个名为`dump()`的方法，打印对象属性（`name`、`symbol`和`number`）的值。使用这个新定义创建`hydrogen`对象，并使用`dump()`打印其属性。

10.7 调用`print(hydrogen)`。在`Element`的定义中，将方法`dump`的名称更改为`__str__`，创建一个新的`hydrogen`对象，并再次调用`print(hydrogen)`。

10.8 修改`Element`使属性`name`、`symbol`和`number`变为私有。为每个属性定义一个 getter 属性来返回其值。

10.9 定义三个类：`Bear`、`Rabbit`和`Octothorpe`。对于每个类，只定义一个方法：`eats()`。这应该返回`'berries'`（`Bear`）、`'clover'`（`Rabbit`）或`'campers'`（`Octothorpe`）。分别创建一个对象并打印它们吃的是什么。

10.10 定义这些类：`Laser`、`Claw`和`SmartPhone`。每个类只有一个方法：`does()`。它返回`'disintegrate'`（`Laser`）、`'crush'`（`Claw`）或`'ring'`（`SmartPhone`）。然后，定义一个名为`Robot`的类，其中包含每个组件对象的一个实例。为`Robot`定义一个`does()`方法，打印其组件对象的功能。

¹ 即使你不想。

² Python 中的命名经常会看到双下划线的例子；为了节省音节，有些人将其发音为*dunder*。

³ 一个 80 年代的便宜但不怎么样的汽车。

⁴ 骡马的父亲是驴，母亲是马；小母驴的父亲是马，母亲是驴。

⁵ 你能保守秘密吗？显然，我不能。
