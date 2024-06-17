# 第四章。面向对象的 Python

Python 是一种面向对象（OO）编程语言。然而，与一些其他面向对象语言不同，Python 不强制您专门使用面向对象范式：它还支持过程式编程，具有模块和函数，因此您可以为程序的每个部分选择最佳范式。面向对象范式帮助您将状态（数据）和行为（代码）组合在方便的功能包中。此外，它提供了一些有用的专门机制，如*继承*和*特殊方法*。更简单的过程式方法，基于模块和函数，当您不需要面向对象编程的优点时可能更合适。使用 Python，您可以混合和匹配范式。

除了核心面向对象概念外，本章还涵盖了*抽象基类*、*装饰器*和*元类*。

# 类和实例

如果你熟悉其他面向对象编程语言（如 C++或 Java）中的面向对象编程，你可能对类和实例有很好的理解：*类*是一种用户定义的类型，你可以*实例化*它以构建*实例*，即该类型的对象。Python 通过其类和实例对象支持此功能。

## Python 类

一个*类*是具有以下特征的 Python 对象：

+   你可以像调用函数一样调用类对象。这种调用称为*实例化*，返回一个称为类的*实例*的对象；类也称为实例的*类型*。

+   一个类具有任意命名的属性，您可以绑定和引用。

+   类属性的值可以是*描述符*（包括函数），在“描述符”中有介绍，也可以是普通数据对象。

+   绑定到函数的类属性也称为类的*方法*。

+   一个方法可以有许多 Python 定义的名称之一，其名称前后有两个下划线（称为*双下划线名称*，简称“双下划线名称”——例如，名称 __init__，读作“dunder init”）。当类提供这些特殊方法时，Python 隐式调用它们，当发生类或其实例的各种操作时。

+   一个类可以*继承*自一个或多个类，这意味着它将一些属性的查找委托给其他类对象（包括常规和双下划线方法），这些属性不在类本身中。

类的一个实例是一个 Python 对象，具有任意命名的属性，您可以绑定和引用。对于实例本身没有找到的任何属性，每个实例对象都将属性查找委托给其类。类反过来可能会将查找委托给它继承的类（如果有的话）。

在 Python 中，类是对象（值），与其他对象一样处理。你可以将一个类作为参数传递给函数调用，并且函数可以将一个类作为调用的结果返回。你可以将一个类绑定到一个变量，一个容器中的项，或者一个对象的属性。类也可以作为字典的键。由于在 Python 中类是完全普通的对象，我们经常说类是*一等*对象。

## 类语句

**class** 语句是创建类对象的最常见方式。**class** 是一个单子句复合语句，具有以下语法：

```py
`class` *`Classname`*(*`base``-``classes`*, *, ***`kw`*):
    statement(s)
```

*Classname* 是一个标识符：当类语句完成时，它绑定（或重新绑定）到刚刚创建的类对象。Python 命名[约定](https://oreil.ly/orJJ1) 建议对类名使用大写字母开头，例如 Item、PrivilegedUser、MultiUseFacility 等。

*base-classes* 是一个以逗号分隔的表达式序列，其值是类对象。各种编程语言对这些类对象使用不同的名称：你可以称它们为类的 *bases, superclasses*，或者 *parents*。你可以说创建的类从其基类 *继承*，*派生*，*扩展*，或者 *子类化*；在本书中，我们通常使用 *extend*。这个类是其基类的 *直接子类* 或者 *后代*。***kw* 可以包括一个命名参数 `metaclass=` 来建立类的 *元类*，如 “Python 如何确定类的元类” 中所述。

从语法上讲，包括 *base-classes* 是可选的：要指示你正在创建一个没有基类的类，只需省略 *base-classes*（并且可选地也省略括号，将冒号直接放在类名后面）。每个类都继承自 object，无论你是否指定了显式基类。

类之间的子类关系是传递的：如果 *C1* 扩展了 *C2*，而 *C2* 扩展了 *C3*，那么 *C1* 扩展了 *C3*。内置函数 `issubclass(*C1*, *C2*)` 接受两个类对象：当 *C1* 扩展了 *C2* 时返回 **True**，否则返回 **False**。任何类都是其自身的子类；因此，对于任何类 *C*，`issubclass(*C*, *C*)` 返回 **True**。我们在 “继承” 中讨论了基类如何影响类的功能。

跟随**class**语句的缩进语句的非空序列是*class body*。类主体作为**class**语句的一部分立即执行。直到主体执行完毕，新的类对象才存在，并且*Classname*标识符尚未绑定（或重新绑定）。“元类如何创建类”提供了关于**class**语句执行时发生的详细信息。请注意，**class**语句不会立即创建任何新类的实例，而是定义了稍后通过调用该类创建实例时所有实例共享的属性集合。

## 类主体

类的主体通常是您指定类属性的地方；这些属性可以是描述符对象（包括函数）或任何类型的普通数据对象。类的一个属性可以是另一个类，所以，例如，您可以在另一个**class**语句内“嵌套”一个**class**语句。

### 类对象的属性

您通常通过在类主体内将值绑定到标识符来指定类对象的属性。例如：

```py
`class` C1:
    x = 23
print(C1.x)                      *`# prints:`* *`23`*
```

在这里，类对象 C1 有一个名为 x 的属性，绑定到值 23，并且 C1.x 引用该属性。这样的属性也可以通过实例访问：c = C1(); print(c.x)。然而，在实践中，这并不总是可靠的。例如，当类实例 c 有一个 x 属性时，c.x 访问的是该属性，而不是类级别的属性。因此，要从实例中访问类级别的属性，例如，使用 print(c.__class__.x)可能是最好的选择。

您还可以在类主体外绑定或解绑类属性。例如：

```py
`class` C2:
    `pass`
C2.x = 23
print(C2.x)                      *`# prints:`* *`23`*
```

如果您只在类主体内部的语句中绑定类属性，您的程序通常会更易读。但是，如果您希望在类级别而不是实例级别传递状态信息，则可能需要在其他地方重新绑定它们；如果您愿意，Python 允许您这样做。通过将属性赋值给属性，在类主体内部绑定的类属性与在类主体外部绑定或重新绑定的属性之间没有区别。

正如我们将很快讨论的那样，所有类实例共享类的所有属性。

**class**语句隐含地设置了一些类属性。属性 __name__ 是在**class**语句中使用的*Classname*标识符字符串。属性 __bases__ 是作为**class**语句中基类给定（或隐含）的类对象元组。例如，使用我们刚刚创建的 C1 类：

```py
print(C1.__name__, C1.__bases__) *`# prints:`* *`C1 (<class 'object'>,)`*
```

类还有一个名为 __dict__ 的属性，它是类使用的只读映射，用于保存其他属性（也可以非正式地称为类的*命名空间*）。

直接在类主体中的语句中，对类属性的引用必须使用简单名称，而不是完全限定名称。例如：

```py
`class` C3:
    x = 23
    y = x + 22                   *`# must use just x`*, *`not`* *`C3.x`*
```

但是，在类主体中定义的*方法*中的语句中，对类属性的引用必须使用完全限定名称，而不是简单名称。例如：

```py
`class` C4:
    x = 23
    `def` amethod(self):
        print(C4.x)              *`# must use C4.x or self.x`*, *`not`* *`just x!`*
```

属性引用（即，表达式如*C.x*）的语义比属性绑定更丰富。我们在“属性参考基础”中详细介绍这样的引用。

### 类主体中的函数定义

大多数类主体包括一些**def**语句，因为函数（在此上下文中称为*方法*）对于大多数类实例都是重要的属性。类主体中的**def**语句遵循“函数”中涵盖的规则。此外，类主体中定义的方法有一个强制的第一个参数，通常总是命名为 self，它引用调用方法的实例。self 参数在方法调用中起着特殊的作用，如“绑定和非绑定方法”中所述。

下面是一个包含方法定义的类的示例：

```py
`class` C5:
    `def` hello(self):
        print('Hello')
```

一个类可以定义各种与其实例上的特定操作相关的特殊双下划线方法。我们在“特殊方法”中详细讨论这些方法。

### 类私有变量

当类主体中的语句（或主体中的方法）使用以两个下划线开头（但不以两个下划线结尾）的标识符时，例如*__ident*，Python 隐式地将标识符更改为*_Classname__ident*，其中*Classname*是类的名称。这种隐式更改允许类使用“私有”名称来命名属性、方法、全局变量和其他用途，从而减少意外重复使用其他地方使用的名称（特别是在子类中）的风险。

根据约定，以*单个*下划线开头的标识符是私有的，无论绑定它们的作用域是或不是一个类。Python 编译器不强制执行此隐私约定：由程序员来尊重它。

### 类文档字符串

如果类主体中的第一个语句是一个字符串文字，则编译器将该字符串绑定为类的*文档字符串*（或*docstring*）；如果类主体中的第一个语句*不是*字符串文字，则其值为**None**。有关文档字符串的更多信息，请参见“文档字符串”。

## 描述符

*描述符*是一个对象，其类提供一个或多个名为 __get__、__set__ 或 __delete__ 的特殊方法。作为类属性的描述符控制访问该类实例上的属性的语义。粗略地说，当您访问一个实例属性时，Python 通过调用相应的描述符上的 __get__ 来获取属性的值，如果有的话。例如：

```py
`class` Const:  *`# class with an overriding descriptor, see later`*
    `def` __init__(self, value):
        self.__dict__['value'] = value
    `def` __set__(self, *_):  
        # silently ignore any attempt at setting
        # (a better design choice might be to raise AttributeError)
        `pass`
    `def` __get__(self, *_):
        *`# always return the constant value`*
        `return` self.__dict__['value']
    `def` __delete__(self, *_): 
        *`# silently ignore any attempt at deleting`* 
        # (a better design choice might be to raise AttributeError)
        `pass`

`class` X:
    c = Const(23)

x = X()
print(x.c)  *`# prints:`* *`23`*
x.c = 42    *`# silently ignored (unless you raise AttributeError)`*
print(x.c)  *`# prints:`* *`23`*
`del` x.c *`# silently ignored again (ditto)`*
print(x.c)  *`# prints:`* *`23`*
```

欲了解更多详细信息，请参阅“属性参考基础”。

### 覆盖和非覆盖描述符

当描述符的类提供了名为 __set__ 的特殊方法时，该描述符称为 *覆盖描述符*（或者，使用较旧且令人困惑的术语，*数据描述符*）；当描述符的类提供了 __get__ 而没有提供 __set__ 时，该描述符称为 *非覆盖描述符*。

例如，函数对象的类提供了 __get__，但没有提供 __set__；因此，函数对象是非覆盖描述符。粗略地说，当您使用具有对应覆盖描述符的值分配实例属性时，Python 通过调用描述符的 __set__ 方法设置属性值。有关详细信息，请参见“实例对象的属性”。

描述符协议的第三个双下划线方法是 __delete__，当使用 **del** 语句删除描述符实例时调用。如果不支持 **del**，实现 __delete__ 并引发适当的 AttributeError 异常是一个好主意；否则，调用者将得到一个神秘的 AttributeError: __delete__ 异常。

[在线文档](https://oreil.ly/0yGz3)包含许多描述符及其相关方法的示例。

## 实例

要创建类的实例，请将类对象视为函数进行调用。每次调用返回一个类型为该类的新实例：

```py
an_instance = C5()
```

内置函数 isinstance(*i*, *C*)，其参数 *C* 是一个类，当 *i* 是类 *C* 或其任何子类的实例时返回 **True**。否则返回 **False**。如果 *C* 是类型元组（3.10+ 或使用 | 运算符连接的多个类型），isinstance 在 *i* 是任何给定类型的实例或子类实例时返回 **True**，否则返回 **False**。

### __init__

当一个类定义或继承了名为 __init__ 的方法时，在调用类对象时会执行 __init__ 方法来对新实例进行每个实例的初始化。调用时传递的参数必须对应于 __init__ 的参数，除了参数 self。例如，考虑以下类定义：

```py
`class` C6:
    `def` __init__(self, n):
        self.x = n
```

下面是创建 C6 类实例的方法：

```py
another_instance = C6(42)
```

如 C6 类定义所示，__init__ 方法通常包含绑定实例属性的语句。__init__ 方法不能返回除 **None** 以外的值；如果返回其他值，Python 将引发 TypeError 异常。

__init__ 的主要目的是绑定并创建新创建实例的属性。您也可以在 __init__ 方法之外绑定、重新绑定或解绑实例属性。然而，当您最初在 __init__ 方法中绑定所有类实例属性时，您的代码更易读。

当 __init__ 方法不存在（且未从任何基类继承）时，必须以无参数调用类，并且新实例没有实例特定的属性。

### 实例对象的属性

创建实例后，可以使用点（.）运算符访问其属性（数据和方法）。例如：

```py
an_instance.hello()                      *`# prints:`* *`Hello`*
print(another_instance.x)                *`# prints:`* *`42`*
```

Python 中这样的属性引用具有相当丰富的语义；我们会在“属性引用基础”中详细介绍它们。

你可以通过将值绑定到属性引用来为实例对象添加属性。例如：

```py
`class` C7:
    `pass`
z = C7()
z.x = 23
print(z.x)                               *`# prints:`* *`23`*
```

实例对象*z*现在有一个名为 x 的属性，绑定到值 23，*z.x* 引用该属性。如果存在 __setattr__ 特殊方法，则会拦截每次绑定属性的尝试。（我们在表 4-1 中介绍了 __setattr__。）

当尝试绑定到一个实例属性时，如果该属性名称对应于类中的重写描述符，描述符的 __set__ 方法会拦截该尝试：如果 C7.x 是一个重写描述符，z.x=23 会执行 type(z).x.__set__(z, 23)。

创建实例会设置两个实例属性。对于任何实例*z*，*z.*__class__ 是*z*所属的类对象，*z.*__dict__ 是*z*用来保存其其他属性的映射。例如，对于我们刚创建的实例*z*：

```py
print(z.__class__.__name__, z.__dict__)  *`# prints:`* *`C7 {'x':23}`*
```

你可以重新绑定（但不能解绑）这两个属性中的任何一个或两个，但这很少是必要的。

对于任何实例*z*、任何对象*x*和任何标识符*S*（除了 __class__ 和 __dict__），*z.S*=*x* 等同于 *z*.__dict__['*S*']=*x*（除非 __setattr__ 特殊方法或重写描述符的 __set__ 特殊方法拦截绑定尝试）。例如，再次引用我们刚创建的*z*：

```py
z.y = 45
z.__dict__['z'] = 67
print(z.x, z.y, z.z)                     *`# prints:`* *`23 45 67`*
```

在 Python 中，通过给属性赋值或显式绑定*z*.__dict__ 中的条目创建的实例属性之间没有区别。

### 工厂函数习语

经常需要根据某些条件创建不同类的实例，或者如果可重用的实例已存在则避免创建新实例。一个常见的误解是通过让 __init__ 方法返回特定对象来满足这些需求。然而，这种方法行不通：如果 __init__ 返回除了**None**之外的任何值，Python 会引发异常。实现灵活对象创建的最佳方式是使用函数而不是直接调用类对象。以这种方式使用的函数称为*工厂函数*。

调用工厂函数是一种灵活的方法：一个函数可以返回一个现有可重用的实例或通过调用适当的类创建一个新实例。假设你有两个几乎可以互换的类，SpecialCase 和 NormalCase，并且想要根据参数灵活地生成其中任何一个类的实例。下面这个适当的工厂函数示例允许你做到这一点（我们将在“绑定和未绑定方法”中更多地讨论 self 参数）：

```py
`class` SpecialCase:
    `def` amethod(self):
        print('special')
`class` NormalCase:
    `def` amethod(self):
        print('normal')
`def` appropriate_case(isnormal=`True`):
    `if` isnormal:
        `return` NormalCase()
    `else``:`
 `return` SpecialCase()
aninstance = appropriate_case(isnormal=`False`)
aninstance.amethod()                  *`# prints:`* *`special`*
```

### __new__

每个类都有（或继承）一个名为 __new__ 的类方法（我们在“类方法”中讨论）。当您调用*C*(**args*, ***kwds*)来创建类*C*的新实例时，Python 首先调用*C.*__new__(*C*, **args*, ***kwds*)，并使用 __new__ 的返回值*x*作为新创建的实例。然后 Python 调用*C.*__init__(*x*, **args*, ***kwds*)，但仅当*x*确实是*C*或其任何子类的实例时（否则，*x*的状态将保持 __new__ 留下的状态）。因此，例如，语句*x*=*C*(23)等同于：

```py
*`x`* = *`C`*.__new__(*`C`*, 23)
`if` isinstance(*`x`*, *`C`*):
    type(*`x`*).__init__(*`x`*, 23)
```

object.__new__ 创建一个新的未初始化实例，该实例作为其第一个参数接收的类的实例。当该类具有 __init__ 方法时，它会忽略其他参数，但当它接收到第一个参数之外的其他参数，并且第一个参数的类没有 __init__ 方法时，它会引发异常。当您在类体内部重写 __new__ 时，您无需添加 __new__=classmethod(__new__)，也不需要使用@classmethod 装饰器，因为 Python 在此上下文中识别名称 __new__ 并将其视为特殊名称。在那些偶发情况下，您稍后在类*C*的体外重新绑定*C*.__new__ 时，您确实需要使用*C*.__new__=classmethod(*whatever*)。

__new__ 具有工厂函数的大部分灵活性，如前一节所述。__new__ 可以选择返回现有实例或根据需要创建新实例。当 __new__ 确实创建新实例时，它通常将创建委托给 object.__new__ 或*C*的另一个超类的 __new__ 方法。

以下示例显示如何重写类方法 __new__ 以实现 Singleton 设计模式的版本：

```py
`class` Singleton:
    _singletons = {}
    `def` __new__(cls, *args, **kwds):
        `if` cls `not` `in` cls._singletons:
            cls._singletons[cls] = obj = super().__new__(cls)
            obj._initialized = False
        `return` cls._singletons[cls]
```

（我们在“协作超类方法调用”中介绍了内置的 super。）

任何 Singleton 的子类（不进一步重写 __new__ 的子类）都有且仅有一个实例。当子类定义 __init__ 时，它必须确保 __init__ 可以安全地重复调用（在每次子类调用时）子类的唯一实例。³ 在此示例中，我们插入 _initialized 属性，设置为**False**，当 __new__ 实际上创建新实例时。子类的 __init__ 方法可以测试 self._initialized 是否为**False**，如果是，则将其设置为**True**并继续执行 __init__ 方法的其余部分。当后续创建单例实例再次调用 __init__ 时，self._initialized 将为**True**，表示实例已初始化，并且 __init__ 通常可以直接返回，避免某些重复工作。

## 属性引用基础知识

*属性引用*是形式为*x.name*的表达式，其中*x*是任何表达式，*name*是称为*属性名称*的标识符。许多 Python 对象都有属性，但是当*x*引用类或实例时，属性引用具有特殊而丰富的语义。方法也是属性，因此我们对一般属性的所有说法也适用于可调用属性（即方法）。

假设*x*是类*C*的实例，该类继承自基类*B*。这些类和实例都有几个属性（数据和方法），如下所示：

```py
`class` B:
    a = 23
    b = 45
    `def` f(self):
        print('method f in class B')
    `def` g(self):
        print('method g in class B')
`class` C(B):
    b = 67
    c = 89
    d = 123
    `def` g(self):
        print('method g in class C')
    `def` h(self):
        print('method h in class C')
x = C()
x.d = 77
x.e = 88
```

几个属性 dunder 名称是特殊的。*C*.__name__ 是字符串'*C*'，类的名称。*C*.__bases__ 是元组(*B*,)，*C*的基类的元组。*x*.__class__ 是*x*所属的类*C*。当您使用这些特殊名称之一引用属性时，属性引用直接查找类或实例对象中的专用槽，并获取找到的值。您不能解绑这些属性。您可以即时重新绑定它们，更改类或实例的名称或基类，但这种高级技术很少必要。

类*C*和实例*x*各自还有一个特殊属性：名为 __dict__ 的映射（对*x*通常是可变的，但对*C*不是）。类或实例的所有其他属性，⁴除了少数特殊属性外，都保存为类或实例的 __dict__ 属性中的项。

### 从类获取属性

当您使用语法*C.name*引用类对象*C*的属性时，查找进行两个步骤：

1.  当'*name*'是*C*.__dict__ 中的键时，*C.name*从*C*.__dict__['*name*']中获取值*v*。然后，当*v*是描述符时（即，type(*v*)提供名为 __get__ 的方法），*C.name*的值是调用 type(*v*).__get__(*v*, None, *C*)的结果。当*v*不是描述符时，*C.name*的值是*v*。

1.  当'*name*'不是*C*.__dict__ 中的键时，*C.name*将查找委托给*C*的基类，这意味着它在*C*的祖先类上循环，并在每个类上尝试*name*查找（按照"继承"中详述的方法解析顺序）。

### 从实例获取属性

当您使用语法*x.name*引用类*C*的实例*x*的属性时，查找进行三个步骤：

1.  当'*name*'出现在*C*（或*C*的祖先类之一）中作为覆盖描述符*v*的名称时（即，type(*v*)提供方法 __get__ 和 __set__），*x.name*的值是 type(*v*).__get__(*v, x, C*)的结果。

1.  否则，当'*name*'是*x*.__dict__ 中的键时，*x.name*获取并返回*x*.__dict__['*name*']中的值。

1.  否则，*x.name*将查找委托给*x*的类（按照与*C.name*相同的两步查找过程）：

    +   当这找到描述符*v*时，属性查找的整体结果再次是 type(*v*).__get__(*v, x, C*)。

    +   当此查找到一个非描述符值*v*时，属性查找的整体结果就是*v*。

当这些查找步骤未找到属性时，Python 会引发 AttributeError 异常。然而，对于*x.name*的查找，当*C*定义或继承特殊方法 __getattr__ 时，Python 会调用*C*.__getattr__(*x, '*name*')而不是引发异常。然后由 __getattr__ 决定返回适当的值或引发适当的异常，通常是 AttributeError。

考虑前面定义的以下属性引用：

```py
print(x.e, x.d, x.c, x.b, x.a)             *`# prints:`* *`88 77 89 67 23`*
```

当在步骤 2 的实例查找过程中，x.e 和 x.d 成功时，因为没有涉及描述符，并且'e'和'd'都是 x.__dict__ 中的键。因此，查找不会继续，而是返回 88 和 77。另外三个引用必须继续到步骤 3 的实例查找过程，并查找 x.__class__（即 C）。x.c 和 x.b 在类查找过程的步骤 1 中成功，因为'c'和'b'都是 C.__dict__ 中的键。因此，查找不会继续，而是返回 89 和 67。x.a 一直到类查找过程的步骤 2，查找 C.__bases__[0]（即 B）。'a'是 B.__dict__ 中的键；因此，x.a 最终成功并返回 23。

### 设置一个属性

注意，属性查找步骤只有在*引用*属性时才会像刚才描述的那样发生，而在*绑定*属性时不会。当绑定到一个名称不是特殊的类或实例属性时（除非 __setattr__ 方法或覆盖描述符的 __set__ 方法拦截实例属性的绑定），你只影响该属性的 __dict__ 条目（在类或实例中分别）。换句话说，对于属性绑定，除了检查覆盖描述符外，不涉及查找过程。

## 绑定方法和非绑定方法

函数对象的 __get__ 方法可以返回函数对象本身，或者包装该函数的*绑定方法对象*；绑定方法与从特定实例获取它时关联。

在前一节的代码中，属性 f、g 和 h 是函数；因此，对它们中的任何一个进行属性引用都会返回一个包装相应函数的方法对象。考虑以下内容：

```py
print(x.h, x.g, x.f, C.h, C.g, C.f)
```

此语句输出三个绑定方法，用如下字符串表示：

```py
<bound method C.h of <__main__.C object at 0x8156d5c>>
```

然后是三个函数对象，用字符串表示如下：

```py
<function C.h at 0x102cabae8>
```

# 绑定方法与函数对象的比较

当属性引用在实例*x*上时，我们得到绑定方法，而当属性引用在类*C*上时，我们得到函数对象。

因为绑定方法已经与特定实例关联，所以可以按以下方式调用方法：

```py
x.h()                   *`# prints:`* *`method h in class C`*
```

这里需要注意的关键点是，你不会通过通常的参数传递语法传递方法的第一个参数 self。相反，实例 *x* 的绑定方法会将 self 参数隐式绑定到对象 *x*。因此，方法体可以访问实例的属性，就像它们是 self 的属性一样，即使我们没有显式地向方法传递参数。

让我们仔细看看绑定方法。当实例上的属性引用在查找过程中找到一个在实例类中作为属性的函数对象时，查找会调用函数的 __get__ 方法来获取属性的值。在这种情况下，调用会创建并返回一个*绑定方法*，它包装了该函数。

注意，当属性引用的查找直接在 *x*.__dict__ 中找到一个函数对象时，属性引用操作不会创建绑定方法。在这种情况下，Python 不会将函数视为描述符，也不会调用函数的 __get__ 方法；相反，函数对象本身就是属性的值。同样地，对于不是普通函数的可调用对象，如内置函数（而不是 Python 编写的函数），Python 不会创建绑定方法。

除了包装的函数对象的属性外，绑定方法还有三个只读属性：im_class 是提供方法的类对象，im_func 是被包装的函数，im_self 指的是来自你获取方法的实例 *x*。

你可以像使用其 im_func 函数一样使用绑定方法，但是对绑定方法的调用不会显式提供一个对应于第一个参数（通常命名为 self）的参数。当你调用绑定方法时，在给定调用点的其他参数（如果有）之前，绑定方法会将 im_self 作为第一个参数传递给 im_func。

让我们详细地跟随一下使用常规语法 *x.name*(*arg*) 进行方法调用所涉及的概念步骤。

```py
`def` f(a, b): ...              *`# a function f with two arguments`*

`class` C:
    name = f
x = C()
```

x 是类 C 的实例对象，name 是 x 的方法名称（C 的属性，其值是一个函数，在本例中是函数 f 的属性），*arg*是任何表达式。Python 首先检查'name'是否是 C 中覆盖描述符的属性名称，但它不是——函数是描述符，因为它们的类型定义了方法 __get__，但不是覆盖的描述符，因为它们的类型没有定义方法 __set__。Python 接下来检查'name'是否是 x._dict__ 中的一个键，但它不是。所以，Python 在 C 中找到了 name（如果 name 通过继承在 C 的一个 __bases__ 中找到，则一切都将同样工作）。Python 注意到属性的值，函数对象 f，是一个描述符。因此，Python 调用 f.__get__(x, C)，返回一个绑定方法对象，其 im_func 设置为 f，im_class 设置为 C，im_self 设置为 x。然后 Python 调用这个绑定方法对象，*arg*作为唯一的参数。绑定方法将 im_self（即 x）插入为调用绑定方法的 im_func（即函数 f）的第一个参数，*arg*成为第二个参数。整体效果就像调用：

```py
x.__class__.__dict__'name'
```

当绑定方法的函数体执行时，它与其 self 对象或任何类之间没有特殊的命名空间关系。引用的变量是局部或全局的，就像任何其他函数一样，详见“命名空间”。变量不会隐式地指示 self 中的属性，也不会指示任何类对象中的属性。当方法需要引用、绑定或解绑其 self 对象的属性时，它通过标准的属性引用语法来完成（例如，self.*name*）。⁵ 缺乏隐式作用域可能需要一些时间来适应（因为在这一点上，Python 与许多面向对象的语言不同），但它确保了清晰性、简单性并消除了潜在的歧义。

绑定方法对象是一类一等公民对象：你可以在任何可调用对象的地方使用它们。由于绑定方法同时持有对其包装的函数和执行它的 self 对象的引用，它是闭包的一个强大而灵活的替代方案（详见“嵌套函数和嵌套作用域”）。如果一个实例对象的类提供了特殊方法 __call__（详见表 4-1），那么这是另一种可行的替代方案。这些构造允许你将一些行为（代码）和一些状态（数据）打包到一个可调用对象中。闭包最简单，但在适用性上有些限制。以下是来自嵌套函数和嵌套作用域部分的闭包示例：

```py
`def` make_adder_as_closure(augend):
    `def` add(addend, _augend=augend):
        `return` addend + _augend
    `return` add
```

绑定方法和可调用实例比闭包更丰富和灵活。以下是如何使用绑定方法实现相同功能的方式：

```py
`def` make_adder_as_bound_method(augend):
    `class` Adder:
        `def` __init__(self, augend):
            self.augend = augend
        `def` add(self, addend):
            `return` addend+self.augend
    `return` Adder(augend).add
```

这是如何使用可调用实例（一个其类提供特殊方法 __call__ 的实例）来实现它的方式：

```py
`def` make_adder_as_callable_instance(augend):
    `class` Adder:
        `def` __init__(self, augend):
            self.augend = augend
        `def` __call__(self, addend):
            `return` addend+self.augend
    `return` Adder(augend)
```

从调用函数的代码视角来看，所有这些工厂函数都是可互换的，因为它们都返回多态的可调用对象。在实现方面，闭包是最简单的；面向对象的方法，即绑定方法和可调用实例，使用更灵活、通用和强大的机制，但在这个简单的例子中并不需要这种额外的功能（因为除了加数之外，不需要其他状态，闭包和面向对象方法都可以轻松处理）。

## 继承

当您在类对象*C*上使用属性引用*C.name*，并且'*name*'不是*C*.__dict__ 中的键时，查找将隐式地在*C*.__bases__ 中的每个类对象上进行，按特定顺序（由于历史原因称为方法解析顺序或 MRO，但实际上适用于所有属性，而不仅仅是方法）。*C*的基类可能会有它们自己的基类。查找将逐个在 MRO 中的直接和间接祖先中进行，停止在找到'*name*'时。

### 方法解析顺序

在类中查找属性名的查找基本上是通过按左到右、深度优先顺序访问祖先类来进行的。然而，在多重继承的情况下（使得继承图成为一般*有向无环图*（DAG），而不仅仅是特定的树），这种简单方法可能导致某些祖先类被访问两次。在这种情况下，解析顺序在查找序列中只保留任何给定类的*最右*出现。

每个类和内置类型都有一个特殊的只读类属性称为 __mro__，它是用于方法解析的类型元组，按顺序排列。只能在类上引用 __mro__，而不能在实例上引用，并且由于 __mro__ 是只读属性，因此无法重新绑定或解绑。有关 Python MRO 的所有方面的详细且高度技术性的解释，请参阅 Michele Simionato 的文章[“Python 2.3 方法解析顺序”](https://oreil.ly/pf6RF)⁶和 Guido van Rossum 关于[“Python 历史”](https://oreil.ly/hetjd)的文章。特别要注意，Python 可能无法确定某个类的任何明确的 MRO：在这种情况下，当 Python 执行该**类**语句时会引发 TypeError 异常。

### 覆盖属性

正如我们刚刚看到的，对属性的搜索沿着 MRO（通常是沿着继承树向上）进行，并且一旦找到属性就会停止。子类始终在其祖先之前进行检查，因此当子类定义与超类中同名的属性时，搜索将找到子类中的定义并在此处停止。这被称为子类*覆盖*超类中的定义。考虑以下代码：

```py
`class` B:
    a = 23
    b = 45
    `def` f(self):
        print('method f in class B')
    `def` g(self):
        print('method g in class B')
`class` C(B):
    b = 67
    c = 89
    d = 123
    `def` g(self):
        print('method g in class C')
    `def` h(self):
        print('method h in class C')
```

在这里，类 C 覆盖了其超类 B 的属性 b 和 g。请注意，与某些其他语言不同，Python 中你可以像轻松覆盖可调用属性（方法）一样覆盖数据属性。

### 委托给超类方法

当子类 *C* 覆盖其超类 *B* 的方法 *f* 时，*C* 的 *f* 方法体通常希望将其操作的某部分委托给超类方法的实现。这有时可以使用函数对象来完成，如下所示：

```py
`class` Base:
    `def` greet(self, name):
        print('Welcome', name)
`class` Sub(Base):
    `def` greet(self, name):
        print('Well Met and', end=' ')
        Base.greet(self, name)
x = Sub()
x.greet('Alex')
```

在 Sub 类的 greet 方法体中，委托到超类的方法使用了通过属性引用 Base.greet 获得的函数对象，因此通常会传递所有参数，包括 self。（如果显式使用基类看起来有点丑陋，请耐心等待；在本节中很快你会看到更好的方法）。委托到超类实现是这种函数对象的常见用法。

委托（Delegation）的一种常见用法出现在特殊方法 **__init__** 中。当 Python 创建一个实例时，不像一些其他面向对象的语言那样自动调用任何基类的 **__init__** 方法。这由子类来初始化其超类，必要时使用委托。例如：

```py
`class` Base:
    `def` __init__(self):
        self.anattribute = 23
`class` Derived(Base):
    `def` __init__(self):
        Base.__init__(self)
        self.anotherattribute = 45
```

如果 Derived 类的 **__init__** 方法没有显式调用 Base 类的 **__init__** 方法，那么 Derived 的实例将缺少其初始化的部分。因此，这些实例将违反 [里氏替换原则（LSP）](https://oreil.ly/0jxrp)，因为它们将缺少属性 anattribute。如果子类不定义 **__init__**，则不会出现此问题，因为在这种情况下，它会从超类继承 **__init__**。因此，*绝对不*有理由编写以下代码：

```py
`class` Derived(Base):
    `def` __init__(self):
        Base.__init__(self)
```

# 绝对不要编写仅委托给超类的方法。

永远不要定义一个语义上空的 **__init__**（即仅委托给超类的方法）。相反，应该从超类继承 **__init__**。这条建议适用于*所有*方法，特殊的或不是，但出于某种原因，编码这种语义上空的方法似乎最常见于 **__init__**。

上述代码说明了将委托概念应用于对象的超类，但在今天的 Python 中，通过名称显式编码这些超类实际上是一种不良实践。如果基类重命名，所有对它的调用点都必须更新。或者更糟的是，如果重构类层次结构在 Derived 和 Base 类之间引入新层，则新插入类的方法将被静默跳过。

推荐的方法是使用内置的 super 类型调用定义在超类中的方法。要调用继承链中的方法，只需调用 super()，不带参数：

```py
`class` Derived(Base):
    `def` __init__(self):
        super().__init__()
        self.anotherattribute = 45
```

### 合作超类方法调用

在多重继承和所谓的“菱形图”情况下，使用超类名称显式调用超类版本的方法也会带来很多问题。考虑以下代码：

```py
`class` A:
    `def` met(self):
        print('A.met')
`class` B(A):
    `def` met(self):
        print('B.met')
        A.met(self)
`class` C(A):
    `def` met(self):
        print('C.met')
        A.met(self)
`class` D(B,C):
    `def` met(self):
        print('D.met')
        B.met(self)
        C.met(self)
```

当我们调用 D().met() 时，A.met 实际上被调用了两次。如何确保每个祖先方法的实现仅被调用一次？解决方案是使用 super：

```py
`class` A:
    `def` met(self):
        print('A.met')
`class` B(A):
    `def` met(self):
        print('B.met')
        super().met()
`class` C(A):
    `def` met(self):
        print('C.met')
        super().met()
`class` D(B,C):
    `def` met(self):
        print('D.met')
        super().met()
```

现在，D().met() 将确保每个类的 met 方法仅被调用一次。如果你养成了使用 super 来编码超类调用的好习惯，你的类将在复杂的继承结构中表现得很顺畅——即使继承结构实际上很简单也不会有任何负面影响。

唯一的情况可能更喜欢通过显式语法调用超类方法的粗糙方法是，当不同类具有相同方法的不同和不兼容签名时。在许多方面，这种情况令人不快；如果你确实必须处理它，显式语法有时可能是最不受欢迎的方法。正确使用多重继承受到严重阻碍；但是，即使在面向对象编程的最基本属性中，如基类和子类实例之间的多态性中，在超类和其子类中为相同名称的方法指定不同签名时，也会受到影响。

### 使用内置函数 type 进行动态类定义

除了使用 type(*obj*) 的方式外，你还可以使用三个参数调用 type 来定义一个新的类：

```py
NewClass = type(name, bases, class_attributes, **kwargs)
```

其中 *name* 是新类的名称（应与目标变量匹配），*bases* 是直接超类的元组，*class_attributes* 是要在新类中定义的类级方法和属性的字典，***kwargs* 是要传递给其中一个基类的元类的可选命名参数。

例如，使用简单的 Vehicle 类层次结构（如 LandVehicle、WaterVehicle、AirVehicle、SpaceVehicle 等），你可以在运行时动态创建混合类，如：

```py
AmphibiousVehicle = type('AmphibiousVehicle', 
                         (LandVehicle, WaterVehicle), {})
```

这相当于定义一个多重继承的类：

```py
`class` AmphibiousVehicle(LandVehicle, WaterVehicle): `pass`
```

当你调用 type 在运行时创建类时，你无需手动定义所有 Vehicle 子类的组合扩展，并且添加新的子类也不需要大量扩展已定义的混合类。⁷ 欲了解更多注解和示例，请参阅[在线文档](https://oreil.ly/aNrSu)。

### “删除”类属性

继承和重写提供了一种简单有效的方式来非侵入性地添加或修改（重写）类属性（如方法）——即在子类中添加或重写属性而无需修改定义属性的基类。然而，继承并未提供一种非侵入性地删除（隐藏）基类属性的方法。如果子类简单地未定义（重写）某个属性，则 Python 会找到基类的定义。如果需要执行此类删除操作，则可能的选择包括以下几种：

+   重写方法并在方法体中引发异常。

+   避免继承，将属性保存在子类的 __dict__ 之外，并为选择性委派定义 __getattr__ 方法。

+   覆盖 __getattribute__ 以类似的效果。

这些技术的最后一个在 “__getattribute__” 中演示。

# 考虑使用聚合而不是继承

继承的替代方法是使用 *聚合*：而不是从基类继承，而是将基类的实例作为私有属性。通过在包含类中提供公共方法（即调用属性上的等效方法）委托给包含的属性，您可以完全控制属性的生命周期和公共接口。这样，包含类对于属性的创建和删除有更多的控制权；此外，对于属性类提供的任何不需要的方法，您只需不在包含类中编写委派方法即可。

## 内置的 object 类型

内置的 object 类型是所有内置类型和类的祖先。object 类型定义了一些特殊方法（在 “特殊方法” 中记录），实现了对象的默认语义：

__new__, __init__

您可以通过调用 object() 而不传递任何参数来创建对象的直接实例。该调用使用 object.__new__ 和 object.__init__ 来创建并返回一个没有属性（甚至没有用于保存属性的 __dict__）的实例对象。这样的实例对象可能作为“哨兵”非常有用，确保与任何其他不同对象比较时不相等。

__delattr__, __getattr__, __getattribute__, __setattr__

默认情况下，任何对象都使用对象的这些方法处理属性引用（如 “属性引用基础知识” 中所述）。

__hash__, __repr__, __str__

将对象传递给 hash、repr 或 str 调用对象的相应 dunder 方法。

对象的子类（即任何类）可以——而且通常会！——覆盖这些方法中的任何一个，和/或添加其他方法。

## 类级方法

Python 提供了两种内置的非覆盖描述符类型，这使得类具有两种不同类型的“类级方法”：*静态方法* 和 *类方法*。

### 静态方法

*静态方法* 是可以在类上调用，或者在类的任何实例上调用的方法，而不受普通方法关于第一个参数的特殊行为和约束的影响。静态方法可以具有任何签名；它可以没有参数，并且如果有的话，第一个参数也不起任何特殊作用。您可以将静态方法视为一种普通函数，您可以正常调用它，尽管它恰好绑定到类属性上。

虽然定义静态方法从未 *必需*（您可以选择定义一个普通函数，而不是在类外部定义），但某些程序员认为它们是一种优雅的语法替代品，当函数的目的与某个特定类紧密绑定时。

要创建一个静态方法，调用内置的 type staticmethod，并将其结果绑定到一个类属性。与所有绑定类属性的方式一样，通常应在类的主体中完成，但您也可以选择在其他地方执行。staticmethod 的唯一参数是 Python 调用静态方法时要调用的函数。以下示例展示了定义和调用静态方法的一种方式：

```py
`class` AClass:
    `def` astatic():
        print('a static method')
    astatic = staticmethod(astatic)

an_instance = AClass()
print(AClass.astatic())             *`# prints:`* *`a static method`*
print(an_instance.astatic())        *`# prints:`* *`a static method`*
```

此示例将同一名称用于传递给 staticmethod 的函数和绑定到 staticmethod 结果的属性。这种命名惯例并非强制性，但是是个好主意，我们建议您始终使用它。Python 提供了一种特殊的简化语法来支持这种风格，详见“装饰器”。

### 类方法

*类方法*是您可以在类上或在类的任何实例上调用的方法。Python 将方法的第一个参数绑定到调用该方法的类或调用该方法的实例的类；它不将其绑定到实例，如普通绑定方法。类方法的第一个参数通常被命名为 cls。

与静态方法一样，虽然定义类方法从不是*必需*的（您始终可以选择在类外定义一个普通函数，并将类对象作为其第一个参数），但类方法是这种函数的一种优雅替代方式（特别是在需要在子类中重写它们时）。

要创建一个类方法，调用内置的 type classmethod，并将其结果绑定到一个类属性。与所有绑定类属性的方式一样，通常应在类的主体中完成，但您也可以选择在其他地方执行。classmethod 的唯一参数是 Python 调用类方法时要调用的函数。以下是定义和调用类方法的一种方式：

```py
`class` ABase:
    `def` aclassmet(cls):
        print('a class method for', cls.__name__)
    aclassmet = classmethod(aclassmet)
`class` `ADeriv`(ABase):
    `pass`

b_instance = ABase()
d_instance = ADeriv()
print(ABase.aclassmet())        *`# prints:`* *`a class method for ABase`*
print(b_instance.aclassmet())   *`# prints:`* *`a class method for ABase`*
print(ADeriv.aclassmet())       *`# prints:`* *`a class method for ADeriv`*
print(d_instance.aclassmet())   *`# prints:`* *`a class method for ADeriv`*
```

此示例将同一名称用于传递给 classmethod 的函数和绑定到 classmethod 结果的属性。同样，这种命名约定并非强制性，但是是个好主意，我们建议您始终使用它。Python 提供了一种特殊的简化语法来支持这种风格，详见“装饰器”。

## 属性

Python 提供了一种内置的重写描述符类型，可用于给类的实例提供*属性*。属性是具有特殊功能的实例属性。您可以使用普通语法（例如，print(*x.prop*)，*x*.*prop*=23，**del** *x.prop*）引用、绑定或解绑属性。但是，与通常的属性引用、绑定和解绑语义不同，这些访问会在实例*x*上调用您作为内置 type property 的参数指定的方法。以下是定义只读属性的一种方式：

```py
`class` Rectangle:
    `def` __init__(self, width, height):
        self.width = width
        self.height = height
    `def` area(self):
        `return` self.width * self.height
    area = property(area, doc='area of the rectangle')
```

类 Rectangle 的每个实例*r*都有一个合成的只读属性*r*.area，方法*r*.area()通过动态计算乘以边的方法来生成。Rectangle.area.__doc__ 的文档字符串是'rectangle 的面积'。*r*.area 属性是只读的（尝试重新绑定或解绑它会失败），因为我们在 property 调用中仅指定了一个 get 方法，而没有 set 或 del 方法。

属性执行与特殊方法 __getattr__、__setattr__ 和 __delattr__（在“通用特殊方法”中介绍）类似的任务，但属性更快更简单。要构建一个属性，请调用内置类型 property 并将其结果绑定到一个类属性。与类属性的所有绑定一样，通常在类的主体中完成，但您可以选择在其他地方完成。在类*C*的主体内部，您可以使用以下语法：

```py
*`attrib`* = property(fget=`None`, fset=`None`, fdel=`None`, doc=`None`)
```

当*x*是类*C*的一个实例，并且您引用*x*.*attrib*时，Python 会在*x*上调用作为 fget 参数传递给属性构造函数的方法，不带参数。当您赋值*x*.*attrib* = *value*时，Python 会调用作为 fset 参数传递的方法，并将*value*作为唯一的参数传递给它。当您执行**del** *x*.*attrib*时，Python 会调用作为 fdel 参数传递的方法，不带参数。Python 使用作为 doc 参数传递的参数作为属性的文档字符串。属性的所有参数都是可选的。当缺少某个参数时，当某些代码尝试进行该操作时，Python 会引发异常。例如，在矩形示例中，我们使属性 area 为只读，因为我们仅为参数 fget 传递了一个参数，而没有为参数 fset 和 fdel 传递参数。

在类中创建属性的一种优雅语法是使用 property 作为*装饰器*（参见“装饰器”）：

```py
`class` Rectangle:
    `def` __init__(self, width, height):
        self.width = width
        self.height = height
    @property
    `def` area(self):
        *`"""area of the rectangle"""`*
        `return` self.width * self.height
```

要使用这种语法，您*必须*将 getter 方法命名为您希望属性具有的相同名称；该方法的文档字符串将成为属性的文档字符串。如果您还想添加设置器和/或删除器，请使用名为（在此示例中）area.setter 和 area.deleter 的装饰器，并将如此装饰的方法命名为属性的相同名称。例如：

```py
`import` math
`class` Rectangle:
    `def` __init__(self, width, height):
        self.width = width
        self.height = height
    @property
    `def` area(self):
        *`"""area of the rectangle"""`*
        `return` self.width * self.height
    @area.setter
    `def` area(self, value):
        scale = math.sqrt(value/self.area)
        self.width *= scale
        self.height *= scale
```

### 为什么属性很重要

属性的关键重要性在于它们的存在使得将公共数据属性作为类公共接口的一部分完全安全（事实上也是建议性的）。如果在将来的类版本或者需要与之多态的其他类中，需要在引用、重新绑定或解绑属性时执行一些代码，您可以将普通属性更改为属性，并获得所需效果，而不会对使用您的类的任何代码（即“客户端代码”）产生任何影响。这让您可以避免面向对象语言中缺乏属性而需要使用的笨拙惯用法，如*访问器*和*修改器*方法。例如，客户端代码可以使用如下自然的惯用法：

```py
some_instance.widget_count += 1
```

而不是被迫进入这样的复杂嵌套访问器和修改器：

```py
some_instance.set_widget_count(some_instance.get_widget_count() + 1)
```

如果你有时候想要编写方法的自然名称像 get_*this* 或 set_*that*，最好将这些方法包装成属性，以增加代码的清晰度。

### 属性和继承

属性的继承与任何其他属性一样工作。然而，对于不留心的人来说，有一个小陷阱：*用于访问属性的方法是在定义属性的类中定义的方法*，而不是使用后续在子类中发生的进一步覆盖。考虑这个例子：

```py
`class` B:
    `def` f(self):
        `return` 23
    g = property(f)
`class` C(B):
    `def` f(self):
        `return` 42

c = C()
print(c.g)                *`# prints:`* *`23`*, *`not`* *`42`*
```

访问属性 c.g 会调用 B.f，而不是你可能期望的 C.f。原因非常简单：属性构造函数（直接或通过装饰器语法）接收的是 *函数对象* f（这发生在执行 B 的 **class** 语句时，因此问题中的函数对象也称为 B.f）。因此，稍后在子类 C 中重新定义名称 f 是无关紧要的，因为属性在创建时并不会查找该名称，而是使用它在创建时收到的函数对象。如果需要解决这个问题，可以通过手动添加额外的查找间接性来实现：

```py
`class` B:
    `def` f(self):
        `return` 23
    `def` _f_getter(self):
        `return` self.f()
    g = property(_f_getter)
`class` C(B):
    `def` f(self):
        `return` 42

c = C()
print(c.g)                *`# prints:`* *`42`**`,`* *`as expected`*
```

在这里，属性所持有的函数对象是 B._f_getter，它反过来确实会查找名称 f（因为它调用 self.f()）；因此，对 f 的覆盖具有预期的效果。正如 David Wheeler 所说，“计算机科学中的所有问题都可以通过另一级间接性来解决。”⁸

## __slots__

通常，任何类 *C* 的实例对象 *x* 都有一个字典 *x.*__dict__，Python 使用它让你在 *x* 上绑定任意属性。为了节省一点内存（以只允许 *x* 有预定义的一组属性名称为代价），可以在类 *C* 中定义一个类属性名为 __slots__，一个序列（通常是元组）的字符串（通常是标识符）。当类 *C* 有 __slots__ 时，类 *C* 的实例 *x* 就没有 __dict__：试图在 *x* 上绑定一个不在 *C.*__slots__ 中的属性名将会引发异常。

使用 __slots__ 可以减少小实例对象的内存消耗，这些对象可以没有强大和便利的能力拥有任意命名的属性。只有在类可能同时有成百上千个实例的情况下，才值得为类添加 __slots__，以便每个实例节省几十个字节的内存。然而，与大多数其他类属性不同，只有在类体中的赋值将其绑定为类属性时，__slots__ 才能像我们刚才描述的那样工作。以后对 __slots__ 的任何更改、重绑定或取消绑定，以及从基类继承 __slots__ 都没有效果。以下是如何在前面定义的 Rectangle 类中添加 __slots__ 以获得更小（虽然不太灵活）的实例：

```py
`class` OptimizedRectangle(Rectangle):
    __slots__ = 'width', 'height'
```

不需要为 area 属性定义槽：__slots__ 不限制属性，只限制普通实例属性，如果没有定义 __slots__，则这些属性将存储在实例的 __dict__ 中。

3.8+ __slots__ 属性也可以使用以属性名称为键和以文档字符串为值的字典来定义。OptimizedRectangle 可以更详细地声明为：

```py
`class` OptimizedRectangle(Rectangle):
    __slots__ = {'width': 'rectangle width in pixels',
                 'height': 'rectangle height in pixels'}
```

## __getattribute__

所有对实例属性的引用都通过特殊方法 __getattribute__。这个方法来自 object，在那里它实现了属性引用的语义（如“属性引用基础”中所述）。你可以覆盖 __getattribute__ 来隐藏子类实例的继承类属性等目的。例如，以下示例展示了一种实现无需 append 方法的列表的方法：

```py
`class` listNoAppend(list):
    `def` __getattribute__(self, name):
        `if` name == 'append':
            `raise` AttributeError(name)
        `return` list.__getattribute__(self, name)
```

类 listNoAppend 的实例 x 几乎与内置的列表对象无法区分，唯一的区别是其运行时性能显著较差，并且任何对 x.append 的引用都会引发异常。

实现 __getattribute__ 可能会比较棘手；使用内置函数 getattr 和 setattr 以及实例的 __dict__（如果有的话）或重新实现 __getattr__ 和 __setattr__ 通常更容易。当然，在某些情况下（如前面的例子），没有其他选择。

## 每个实例方法

实例可以具有所有属性的实例特定绑定，包括可调用属性（方法）。对于方法，与任何其他属性（除了绑定到覆盖描述符的属性之外），实例特定绑定会隐藏类级绑定：属性查找在找到直接在实例中绑定时不考虑类。对于可调用属性的实例特定绑定不执行“绑定和非绑定方法”中详细描述的任何转换：属性引用返回完全相同的可调用对象，该对象之前直接绑定到实例属性。

然而，这对于 Python 隐式调用的各种操作的每个实例绑定的特殊方法可能不像你期望的那样工作，如“特殊方法”中所述。这些特殊方法的隐式使用总是依赖于特殊方法的*类级*绑定（如果有的话）。例如：

```py
`def` fake_get_item(idx):
    `return` idx
`class` MyClass:
    `pass`
n = MyClass()
n.__getitem__ = fake_get_item
print(n[23])                      *`# results in:`*
*`# Traceback (most recent call last):`*
*`#   File "<stdin>", line 1, in ?`*
*`# TypeError: unindexable object`*
```

## 从内置类型继承

类可以从内置类型继承。但是，类只能直接或间接扩展多个内置类型，前提是这些类型专门设计为允许这种互操作兼容性。Python 不支持从多个任意内置类型无约束地继承。通常，新式类最多只扩展一个实质性的内置类型。例如，这样：

```py
`class` noway(dict, list):
    `pass`
```

引发 TypeError 异常，并详细说明“多个基类具有实例布局冲突。” 当你看到这样的错误消息时，意味着你试图直接或间接地从多个不特别设计以在如此深层次上合作的内置类型继承。

# 特殊方法

一个类可以定义或继承特殊方法，通常被称为“dunder”方法，因为如前所述，它们的名称前后都有双下划线。每个特殊方法与特定操作相关联。Python 在你对实例对象执行相关操作时会隐式调用特殊方法。在大多数情况下，方法的返回值是操作的结果，当操作所关联的方法不存在时，尝试该操作会引发异常。

在本节中，我们指出一般规则不适用的情况。在以下讨论中，*x* 是执行操作的类 *C* 的实例，*y* 是另一个操作数（如果有的话）。每个方法的参数 self 也指代实例对象 *x*。每当我们提到对 *x.*__*whatever*__(...) 的调用时，请记住，严格来说，正在发生的确切调用实际上是 *x.*__class__.__*whatever*__(*x*, ...)。

## 通用特殊方法

一些 dunder 方法与通用操作相关。定义或继承这些方法的类允许其实例控制这些操作。这些操作可以分为几类：

初始化和结束处理

类可以通过特殊方法 __new__ 和 __init__ 控制其实例的初始化（这是一个非常常见的需求），并且/或者通过 __del__ 控制其终结处理（这是一个罕见的需求）。

字符串表示

类可以通过特殊方法 __repr__, __str__, __format__ 和 __bytes__ 控制 Python 如何将其实例呈现为字符串。

比较、哈希和在布尔上下文中的使用

一个类可以控制其实例如何与其他对象比较（通过特殊方法 __lt__, __le__, __gt__, __ge__, __eq__ 和 __ne__），字典如何将其用作键以及集合如何将其用作成员（通过 __hash__），以及它们在布尔上下文中是否评估为真值或假值（通过 __bool__）。

属性引用、绑定和解绑

一个类可以通过特殊方法 __getattribute__, __getattr__, __setattr__ 和 __delattr__ 控制对其实例属性（引用、绑定、解绑）的访问。

可调用实例

类可以通过特殊方法 __call__ 使其实例可调用，就像函数对象一样。

表 4-1 记录了通用的特殊方法。

表 4-1\. 通用特殊方法

| __bool__ | __bool__(self) 当对*x*进行真假判断（参见“布尔值”）时，例如在调用 bool(*x*)时，Python 会调用*x*.__bool__()，该方法应返回**True**或**False**。当不存在 __bool__ 方法时，Python 会调用 __len__ 方法，并且当*x*.__len__()返回 0 时将*x*视为假（为了检查容器是否非空，避免编写**if** len(*container*)>0:；而是使用**if** *container*:）。当既不存在 __bool__ 方法也不存在 __len__ 方法时，Python 将*x*视为真。 |
| --- | --- |
| __bytes__ | __bytes__(self) 调用 bytes(*x*)时会调用*x*.__bytes__()，如果存在的话。如果一个类同时提供了 __bytes__ 和 __str__ 特殊方法，它们分别应返回“等效”的 bytes 类型和 str 类型字符串。 |
| __call__ | __call__(self[, *args*...]) 当调用*x*([*args*...])时，Python 会将此操作转换为对*x*.__call__([*args*...])的调用。调用操作的参数对应于 __call__ 方法的参数，去除第一个参数。第一个参数，通常称为 self，引用*x*：Python 会隐式提供它，就像对绑定方法的任何其他调用一样。 |

| __del__ | __del__(self) 当*x*通过垃圾收集消失之前，Python 会调用*x*.__del__()让*x*完成最后操作。如果没有 __del__ 方法，Python 在回收*x*时不会进行特殊处理（这是最常见的情况：很少有类需要定义 __del__）。Python 忽略 __del__ 的返回值，并且不会隐式调用*C*类超类的 __del__ 方法。*C*.__del__ 必须显式执行任何需要的最终操作，包括必要时通过委托来完成。当类*C*有需要终结的基类时，*C*.__del__ 必须调用 super().__del__()。

__del__ 方法与“del 语句”中涵盖的**del**语句没有特定联系。

一般情况下，当您需要及时和确保的最终操作时，__del__ 并不是最佳选择。对于这种需求，应使用“try/finally”中涵盖的**try**/**finally**语句（或者更好的是“The with Statement”中涵盖的**with**语句）。定义有 __del__ 方法的类的实例不参与循环垃圾收集，详见“垃圾收集”。注意避免涉及这些实例的引用循环：只有在没有可行的替代方案时才定义 __del__。

| __delattr__ | __delattr__(self, *name*) 每次请求解绑属性*x.y*（通常是**del** *x.y*），Python 会调用*x*.__delattr__('*y*')。所有后续讨论的 __setattr__ 都适用于 __delattr__。Python 忽略 __delattr__ 的返回值。如果不存在 __delattr__，Python 会将**del** *x.y*转换为**del** *x*.__dict__['*y*']。 |
| --- | --- |
| __dir__ | __dir__(self) 当调用 dir(*x*)时，Python 将操作转换为调用*x*.__dir__()，它必须返回*x*的属性的排序列表。当*x*的类没有 __dir__ 时，dir(*x*)执行内省以返回*x*的属性的排序列表，努力产生相关而非完整的信息。 |

| __eq__, __ge__, __gt__, __le__,

__lt__, __ne__ | __eq__(self, *other*), __ge__(self, *other*), __gt__(self, *other*), __le__(self, *other*),

__lt__(self, *other*), __ne__(self, *other*)

比较*x* == *y*, *x* >= *y*, *x* > *y*, *x* <= *y*, *x* < *y*, 和*x* != *y*，分别调用列出的特殊方法，应返回**False**或**True**。每个方法可以返回 NotImplemented 告知 Python 以替代方式处理比较（例如，Python 可能尝试*y* > *x*来代替*x* < *y*）。

最佳实践是仅定义一个不等比较方法（通常是 __lt__）加上 __eq__，并用 functools.total_ordering 修饰类（在表 8-7 中有介绍），以避免模板和比较中的逻辑矛盾风险。 |

| __format__ | __format__(self, format_string='') 调用 format(*x*)会调用*x*.__format__('')，调用 format(*x, format_string*)会调用*x*.__format__(*format_string*)。类负责解释格式字符串（每个类可以定义自己的小型格式规范语言，受内置类型实现的启发，如在“字符串格式化”中介绍）。当从 object 继承 __format__ 时，它委托给 __str__ 并且不接受非空格式字符串。 |
| --- | --- |
| __getattr__ | __getattr__(self, *name*) 当找不到*x.y*的常规步骤时（即通常会引发 AttributeError 时），Python 会调用*x*.__getattr__('*y*')。Python 不会对通过常规方式找到的属性调用 __getattr__（如作为*x*.__dict__ 中的键或通过*x*.__class__ 访问）。如果希望 Python 对*每个*属性都调用 __getattr__，可以将属性存放在其他位置（例如通过私有名称引用的另一个字典中），或者改写 __getattribute__。如果 __getattr__ 找不到*y*，应该引发 AttributeError。 |

| __getattribute__ | __getattribute_(self, *name*) 每次访问属性*x*.*y*时，Python 调用*x*.__getattribute__('*y*')，它必须获取并返回属性值，否则引发 AttributeError。属性访问的通常语义（*x*.__dict__、*C.*__slots__、*C*的类属性、*x*.__getattr__）都归因于 object.__getattribute__。

当类*C*重写 __getattribute__ 时，必须实现它想要提供的所有属性语义。实现属性访问的典型方式是委托（例如，在重写 __getattribute__ 的操作中调用 object.__getattribute__(self, ...)）。

# 重写 __getattribute__ 会减慢属性访问速度

当一个类覆盖了 __getattribute__，则该类实例上的所有属性访问变得缓慢，因为覆盖的代码会在每次属性访问时执行。

|

| __hash__ | __hash__(self) 调用 hash(*x*) 会调用 *x*.__hash__()（以及其他需要知道 *x* 哈希值的上下文，如将 *x* 作为字典键使用，如 *D*[*x*] 其中 *D* 是一个字典，或将 *x* 作为集合成员使用）。__hash__ 必须返回一个 int，以便当 *x*==*y* 时意味着 hash(*x*)==hash(*y*)，并且对于给定的对象必须始终返回相同的值。

当缺少 __hash__ 时，调用 hash(*x*) 会调用 id(*x*)，只要同时缺少 __eq__。其他需要知道 *x* 哈希值的上下文行为相同。

任何 *x*，使得 hash(*x*) 返回一个结果而不是引发异常，被称为*可哈希对象*。当缺少 __hash__，但存在 __eq__ 时，调用 hash(*x*) 会引发异常（以及其他需要知道 *x* 的哈希值的上下文）。在这种情况下，*x* 不可哈希，因此不能作为字典键或集合成员。

通常只为不可变对象定义 __hash__，而且还定义了 __eq__。请注意，如果存在任何 *y* 使得 *x*==*y*，即使 *y* 是不同类型的对象，并且 *x* 和 *y* 都是可哈希的，*必须*确保 hash(*x*)==hash(*y*)。（在 Python 内置类型中，存在一些情况，其中对象的不同类型之间可以相等。最重要的是不同数字类型之间的相等性：int 可以等于 bool、float、fractions.Fraction 实例或 decimal.Decimal 实例。）

| __init__ | __init__(self[, *args*...]) 当调用 *C*([*args...*]) 创建类 *C* 的实例 *x* 时，Python 调用 *x*.__init__([*args...*]) 让 *x* 初始化自己。如果缺少 __init__（即从 object 继承），必须无参数调用 *C*，即 *C*()，并且在创建时 *x* 没有实例特定的属性。Python 不会对 *C* 类及其超类的 __init__ 方法进行隐式调用。*C*.__init__ 必须显式执行任何初始化操作，包括必要时委托。例如，当类 *C* 有一个需要无参数初始化的基类 *B* 时，*C*.__init__ 中的代码必须显式调用 super().__init__()。__init__ 的继承与任何其他方法或属性相同：如果 *C* 本身没有覆盖 __init__，它会从其 __mro__ 中的第一个超类继承它，就像其他属性一样。

`__init__` 必须返回 None；否则，调用该类会引发 TypeError。

| `__new__` | `__new__(cls[, *args*...])` 当你调用 `*C*([*args*...])` 时，Python 通过调用 `*C*.__new__(*C*[, *args*...])` 来获取你正在创建的新实例 *x*。每个类都有类方法 `__new__`（通常只是从 object 继承而来），它可以返回任何值 *x*。换句话说，`__new__` 不需要返回 *C* 的新实例，尽管期望如此。如果 `__new__` 返回的值 *x* 是 *C* 的实例或 *C* 的任何子类的实例（无论是新的还是之前存在的实例），Python 就会在 *x* 上调用 `__init__`（使用最初传递给 `__new__` 的相同 [*args*...]）。

# 在 `__new__` 中初始化不可变对象，其他对象在 `__init__` 中初始化。

你可以在 `__init__` 或 `__new__` 中执行大多数类型的新实例初始化，所以你可能想知道最好将它们放在哪里。最佳实践是只将初始化放在 `__init__` 中，除非你有一个特定的理由将其放在 `__new__` 中。（当类型是不可变的时，`__init__` 不能改变其实例：在这种情况下，`__new__` 必须执行所有的初始化。）

| |

| `__repr__` | `__repr__(self)` 调用 `repr(*x*)`（当 *x* 是表达式语句的结果时，在交互解释器中会隐式发生）会调用 `*x*.__repr__()` 来获取并返回 *x* 的完整字符串表示。如果没有 `__repr__`，Python 就会使用默认的字符串表示。 `__repr__` 应返回一个包含关于 *x* 的无歧义信息的字符串。在可行的情况下，尝试使 `eval(repr(*x*))==*x*`（但不要为了达到这个目标而过度努力！）。 |
| --- | --- |
| `__setattr__` | `__setattr__(self, *name*, *value*)` 对于绑定属性 `*x.y*` 的任何请求（通常是赋值语句 `*x.y=value*`，但也可以是 `setattr(*x*, '*y*', *value*)`），Python 调用 `*x*.__setattr__('*y*', *value*)`。Python 总是对 *x* 的 *任何* 属性绑定调用 `__setattr__` —— 这与 `__getattr__` 的主要区别（在这一点上，`__setattr__` 更接近 `__getattribute__`）。为了避免递归，当 `*x*.__setattr__` 绑定 *x* 的属性时，它必须直接修改 `*x*.__dict__`（例如，通过 `*x*.__dict__[*name*]=*value*`）；或者更好的是，`__setattr__` 可以委托给超类（调用 `super().__setattr__('*y*', *value*)`）。Python 忽略 `__setattr__` 的返回值。如果 `__setattr__` 不存在（即从 object 继承），并且 *C.y* 不是覆盖描述符，Python 通常会将 `*x.y=z*` 翻译为 `*x*.__dict__['*y*']=*z*`（但 `__setattr__` 也可以与 `__slots__` 一起很好地工作）。 |
| `__str__` | `__str__(self)` 类似于 `print(*x*)`，`str(*x*)` 调用 `*x*.__str__()` 来获取 *x* 的非正式、简洁的字符串表示。如果没有 `__str__`，Python 就会调用 `*x*.__repr__`。 `__str__` 应返回一个方便阅读的字符串，即使这可能需要一些近似。 |

## 容器的特殊方法

实例可以是一个 *container*（序列、映射或集合——相互排斥的概念⁹）。为了最大限度地提高实用性，容器应提供特殊方法 __getitem__、__contains__ 和 __iter__（如果可变，则还应提供 __setitem__ 和 __delitem__），以及后续章节讨论的常规方法。在许多情况下，您可以通过扩展 collections.abc 模块中的适当抽象基类（例如 Sequence、MutableSequence 等）来获得合适的常规方法实现，如 “Abstract Base Classes” 中所述。

### 序列

在每个项目访问的特殊方法中，一个包含 *L* 个项目的序列应接受任何整数 *key*，使得 *-L*<=*key*<*L*。¹⁰ 为了与内置序列兼容，负索引 *key*，0>*key*>=-*L*，应等同于 *key*+*L*。当 *key* 具有无效类型时，索引应引发 TypeError 异常。当 *key* 是有效类型的值但超出范围时，索引应引发 IndexError 异常。对于不定义 __iter__ 的序列类，**for** 语句依赖于这些要求，以及接受可迭代参数的内置函数也依赖于这些要求。每个序列的项目访问特殊方法也应（如果有可能）接受作为其索引参数的内置类型切片的实例，其 start、step 和 stop 属性为 int 或 None；*slicing* 语法依赖于此要求，如 “Container slicing” 中所述。

序列还应允许通过 + 进行连接（与同类型的另一个序列），并通过 *（乘以整数）进行重复。因此，序列应具有特殊方法 __add__、__mul__、__radd__ 和 __rmul__，如 “Special Methods for Numeric Objects” 中所述；此外，*可变*序列应具有等效的就地方法 __iadd__ 和 __imul__。序列应与同类型的另一个序列有意义地进行比较，实现 [*字典序*比较](https://oreil.ly/byfuT)，就像列表和元组一样。（继承自 Sequence 或 MutableSequence 抽象基类不能满足所有这些要求；最多只能从 MutableSequence 继承，只提供 __iadd__。）

每个序列都应包括 “List methods” 中介绍的常规方法：不区分大小写的 count 和 index 方法，如果可变，则还应包括 append、insert、extend、pop、remove、reverse 和 sort 方法，其签名和语义与列表的相应方法相同。（继承自 Sequence 或 MutableSequence 抽象基类足以满足这些要求，除了 sort 方法。）

如果一个不可变序列的所有项都是可散列的，那么它本身也应该是可散列的。序列类型可能以某些方式限制其项（例如，仅接受字符串项），但这不是强制性的。

### 映射

映射的元素访问特殊方法在接收到无效的 *key* 参数值（有效类型）时应引发 KeyError 异常，而不是 IndexError。任何映射都应定义在“字典方法”中介绍的非特殊方法：copy、get、items、keys 和 values。可变映射还应定义方法 clear、pop、popitem、setdefault 和 update。（从 Mapping 或 MutableMapping 抽象基类继承可以满足这些要求，但不包括 copy。）

如果其所有项都是可哈希的，则不可变映射类似类型应该是可哈希的。映射类似类型可以在某些方面对其键进行约束，例如仅接受可哈希键，或者（更具体地）例如仅接受字符串键，但这不是强制性的。任何映射应该与相同类型的另一个映射有意义地可比较（至少在等式和不等式方面，尽管不一定是有序比较）。

### 集合

集合是一种特殊的容器：它们既不是序列也不是映射，不能被索引，但是有长度（元素个数）并且可迭代。集合还支持许多运算符（&、|、^ 和 -），以及成员测试和比较，还有等效的非特殊方法（交集、并集等）。如果你实现了类似集合的容器，它应该对 Python 内置的集合具有多态性，详见“集合”。（从 Set 或 MutableSet 抽象基类继承可以满足这些要求。）

如果其所有元素都是可哈希的，则不可变集合类似类型应该是可哈希的。集合类似类型可以在某些方面对其元素进行约束，例如仅接受可哈希元素，或（更具体地）例如仅接受整数元素，但这不是强制性的。

### 容器切片

当你引用、绑定或取消绑定容器 *x* 上的切片，例如 *x*[*i*:*j*] 或 *x*[*i*:*j*:*k*]（在实践中，这仅用于序列时），Python 调用 *x* 的适用的元素访问特殊方法，将一个名为 *slice object* 的内置类型的对象作为 *key*。切片对象具有属性 start、stop 和 step。如果在切片语法中省略了相应的值，则每个属性都是 **None**。例如，**del** *x*[:3] 调用 *x*.__delitem__(*y*)，其中 *y* 是一个切片对象，使得 *y*.stop 为 3，*y*.start 为 **None**，*y*.step 为 **None**。容器对象 *x* 应适当地解释传递给 *x* 的特殊方法的切片对象参数。切片对象的方法 indices 可以帮助：以你的容器长度作为其唯一参数调用它，它返回一个包含三个非负索引的元组，适合作为循环索引切片中每个项目的开始、停止和步长。例如，在序列类的 __getitem__ 特殊方法中，完全支持切片的常见习惯用法是：

```py
`def` __getitem__(self, index):
    *`# Recursively special-case slicing`*
    `if` isinstance(index, slice):
        `return` self.__class__(self[x]
                              `for` x `in` range(*index.indices(len(self))))
    *`# Check index, and deal with a negative and/or out-of-bounds index`*
    index = operator.index(index)
    `if` index < 0:
        index += len(self)
    `if` `not` (0 <= index < len(self)):
        `raise` `IndexError`
    *`# Index is now a correct int, within range(len(self))`*
 *`# ...rest of __getitem__, dealing with single-item access...`*
```

这种习惯用法使用生成器表达式（genexp）语法，并假定你的类的 __init__ 方法可以使用可迭代参数调用，以创建适当的新实例。

### 容器方法

特殊方法 __getitem__、__setitem__、__delitem__、__iter__、__len__ 和 __contains__ 公开容器功能（参见表 4-2）。

表格 4-2\. 容器方法

| __contains__ | 布尔测试*y* **in** *x* 调用*x*.__contains__(*y*)。当*x*是一个序列或类似集合时，__contains__ 应该在*y*等于*x*中的一个项的值时返回**True**。当*x*是一个映射时，__contains__ 应该在*y*等于*x*中的一个键的值时返回**True**。否则，__contains__ 应该返回**False**。当 __contains__ 不存在且*x*是可迭代的时候，Python 执行*y* **in** *x*如下，时间与 len(*x*)成正比：

```py
`for` *`z`* `in` *`x`*:
    `if` *`y`*==*`z`*:
        `return` `True`
`return` `False`
```

|

| __delitem__ | 当一个请求要解除*x*的一个项或片段的绑定（通常是 **del** *x*[*key*]），Python 调用*x*.__delitem__(*key*)。如果*x*是可变的且可以删除项（及可能的片段），则容器*x*应该有 __delitem__。 |
| --- | --- |
| __getitem__ | 当你访问*x*[*key*]（即当你索引或切片容器*x*时），Python 调用*x*.__getitem__(*key*)。所有（非类似集合的）容器都应该有 __getitem__。 |
| __iter__ | 当一个请求要循环遍历*x*的所有项（通常是 **for** *item* **in** *x*），Python 调用*x*.__iter__()来获取*x*上的迭代器。内置函数 iter(*x*)也调用*x*.__iter__()。当 __iter__ 不存在时，iter(*x*)会合成并返回一个迭代器对象，该对象包装*x*并产生*x*[0]、*x*[1]等，直到这些索引中的一个引发 IndexError 异常以指示容器的末尾。但是，最好确保你编写的所有容器类都有 __iter__。 |
| __len__ | 调用 len(*x*)会调用*x*.__len__()（其他需要知道容器*x*中有多少项的内置函数也会这样）。__len__ 应该返回一个整数，即*x*中的项数。当 __bool__ 不存在时，Python 还会调用*x*._len__()来评估*x*在布尔上下文中的值；在这种情况下，当且仅当容器为空时（即容器的长度为 0 时），容器是虚假的。所有容器都应该有 __len__，除非容器确定包含的项数太昂贵。 |
| __setitem__ | 当一个请求要绑定*x*的一个项或片段（通常是一个赋值*x*[*key*]=*value*），Python 调用*x*.__setitem__(*key, value*)。如果*x*是可变的，则容器*x*应该有 __setitem__，因此可以添加或重新绑定项，也许还有片段。 |

## 抽象基类

抽象基类（ABCs）是面向对象设计中的重要模式：它们是不能直接实例化的类，而是存在于被具体类扩展的目的（通常的类，可以被实例化的那种）。

一个推荐的面向对象设计方法（归功于 Arthur J. Riel）是永远不要扩展一个具体类。¹¹ 如果两个具体类有足够的共同点，使你想让其中一个继承另一个，那么可以通过创建一个 *抽象* 基类来替代，该抽象基类涵盖它们所有的共同点，并让每个具体类扩展该 ABC。这种方法避免了继承中许多微妙的陷阱和问题。

Python 对 ABCs 提供了丰富的支持，足以使它们成为 Python 对象模型的一部分。¹²

### abc 模块

标准库模块 abc 提供了元类 ABCMeta 和类 ABC（继承 abc.ABC 使得 abc.ABCMeta 成为元类，且没有其他效果）。

当你将 abc.ABCMeta 作为任何类 *C* 的元类时，这使得 *C* 成为一个 ABC，并提供了类方法 *C*.register，可用一个参数调用：该参数可以是任何现有类（或内置类型） *X*。

调用 *C*.register(*X*) 使 *X* 成为 *C* 的一个 *虚拟* 子类，这意味着 issubclass(*X,* *C*) 返回 **True**，但 *C* 不出现在 *X*.__mro__ 中，*X* 也不继承 *C* 的任何方法或其他属性。

当然，也可以像通常的子类化方式一样，让一个新类 *Y* 继承自 *C*，在这种情况下 *C* 会出现在 *Y*.__mro__ 中，并且 *Y* 继承 *C* 的所有方法，就像通常的子类化一样。

一个 ABC *C* 还可以选择重写类方法 __subclasshook__，当 issubclass(*X, C*) 调用时会传入单一参数 *X*（*X* 是任何类或类型）。当 *C*.__subclasshook__(*X*) 返回 **True** 时，issubclass(*X, C*) 也返回 **True**；当 *C*.__subclasshook__(*X*) 返回 **False** 时，issubclass(*X, C*) 也返回 **False**。当 *C*.__subclasshook__(*X*) 返回 NotImplemented 时，issubclass(*X, C*) 会按照通常的方式进行。

abc 模块还提供了 decorator abstractmethod 来指定必须在继承类中实现的方法。你可以通过先使用 property 然后是 abstractmethod decorators 的顺序来将属性定义为抽象。¹³ 抽象方法和属性可以有实现（通过 super 内建函数对子类可见），但将方法和属性设为抽象的目的是，只有当 *X* 覆盖了 ABC *C* 的每个抽象属性和方法时，才能实例化 ABC *C* 的非虚拟子类 *X*。

### collections 模块中的 ABCs

collections 提供了许多 ABCs，在 collections.abc.¹⁴ 中列出了一些这样的 ABCs，这些 ABCs 接受作为虚拟子类任何定义或继承特定抽象方法的类，如 表 4-3 中所列。

表 4-3\. 单方法 ABCs

| ABC | 抽象方法 |
| --- | --- |
| Callable | __call__ |
| Container | __contains__ |
| Hashable | __hash__ |
| Iterable | __iter__ |
| Sized | __len__ |

collections.abc 中的其他 ABCs 扩展了其中一个或多个，添加了更多基于抽象方法的抽象方法和/或 *mixin* 方法。（当你在具体类中扩展任何 ABC 时，你 *必须* 覆盖抽象方法；你也可以覆盖一些或所有的 mixin 方法，以帮助提高性能，但这不是必须的——当这样做能够获得足够的性能以满足你的目的时，你可以直接继承它们。）

表 4-4 详细说明了 collections.abc 中直接扩展了前述 ABCs 的 ABCs。

表 4-4\. 具有附加方法的 ABCs

| ABC | 扩展 | 抽象方法 | Mixin 方法 |
| --- | --- | --- | --- |
| Iterator | Iterable | __next__ | __iter__ |

| Mapping | Container Iterable

Sized | __getitem__ __iter__

__len__ | __contains__ __eq__

__ne__

getitems

keys

values |

| MappingView | Sized |   | __len__ |
| --- | --- | --- | --- |

| Sequence | Container Iterable

Sized | __getitem__ __len__ | __contains__ __iter__

__reversed__

count

index |

| Set | Container Iterable

Sized | __contains__ __iter

__len__ | __and__^(a) __eq__

__ge__^(b)

__gt__

__le__

__lt__

__ne__

__or__

__sub__

__xor__

isdisjoint |

| ^(a) 对于集合和可变集合，许多 dunder 方法等效于具体类 set 中的非特殊方法；例如，__add__ 就像交集，而 __iadd__ 就像 intersection_update。^(b) 对于集合，排序方法反映了“子集”的概念：*s1* <= *s2* 意味着“*s1* 是 *s2* 的子集或等于 *s2*。” |
| --- |

表 4-5 详细说明了本模块中进一步扩展的 ABCs。

表 4-5\. collections.abc 中剩余的 ABCs

| ABC | 扩展 | 抽象方法 | Mixin 方法 |
| --- | --- | --- | --- |
| ItemsView | MappingView Set |   | __contains__ __iter__ |
| KeysView | MappingView Set |   | __contains__ __iter__ |

| MutableMapping | Mapping | __delitem__ __getitem__

__iter__

__len_

__setitem__ | Mapping’s methods, plus: clear

pop

popitem

setdefault

update |

| MutableSequence | Sequence | __delitem__ __getitem__

__len__

__setitem__

insert | Sequence’s methods, plus: __iadd__

append

extend

pop

remove

reverse |

| MutableSet | Set | __contains__ __iter

__len__

add

discard | Set’s methods, plus: __iand__

__ior__

__isub__

__ixor__

clear

pop

remove |

| ValuesView | MappingView |   | __contains__ __iter__ |
| --- | --- | --- | --- |

查看 [在线文档](https://oreil.ly/AVoUU) 获取更多详细信息和使用示例。

### numbers 模块中的 ABCs

numbers 提供了一个层次结构（也称为 *tower*）的 ABCs，表示各种类型的数字。 表 4-6 列出了 numbers 模块中的 ABCs。

表 4-6\. numbers 模块提供的 ABCs

| ABC | 描述 |
| --- | --- |
| Number | 层次结构的根。包括任意类型的数值；不需要支持任何给定的操作。 |
| Complex | 扩展自 Number。必须支持（通过特殊方法）转换为 complex 和 bool，以及 +，-，*，/，==，!=，和 abs，以及直接的方法 conjugate 和属性 real 和 imag。 |
| Real | 扩展自 Complex。此外，必须支持（通过特殊方法）转换为 float，math.trunc，round，math.floor，math.ceil，divmod，//，%，<，<=，>，和 >=。 |
| Rational | 扩展自 Real。此外，必须支持 numerator 和 denominator 属性。 |
| Integral | 扩展自 Rational。此外，必须支持（通过特殊方法）转换为 int，**，和位运算 <<，>>，&，^， | ，和 ~。 |
| ^(a) 因此，每个整数或浮点数都有一个 real 属性等于其值，以及一个 imag 属性等于 0.^(b) 因此，每个整数都有一个 numerator 属性等于其值，以及一个 denominator 属性等于 1. |

查看 [在线文档](https://oreil.ly/ViRw9) 获取关于实现自定义数值类型的说明。

## 数值对象的特殊方法

一个实例可以通过多个特殊方法支持数值操作。一些不是数字的类也支持 [Table 4-7](https://oreil.ly/ViRw9) 中的一些特殊方法，以重载如 + 和 * 的运算符。特别是，序列应该有特殊方法 __add__, __mul__, __radd__, 和 __rmul__，如 “Sequences” 所述。当二进制方法之一（如 __add__, __sub__ 等）被调用时，如果操作数的类型不支持该方法，则该方法应返回内置的 NotImplemented 单例。

表 4-7\. 数值对象的特殊方法

| __abs__, __invert__,

__neg__,

__pos__ | __abs_(self), __invert__(self), __neg__(self), __pos__(self) 一元运算符 abs(*x*), ~*x*, -*x*, 和 +*x*，分别调用这些方法。

| __add__, __mod__,

__mul__,

__sub__ | __add__ (self, *other*), __mod__(self, *other*),

__mul__(self, *other*),

__sub__(self, *other*)

运算符 *x +* *y*, *x %* *y*, *x ** *y*, 和 *x -* *y* 分别调用这些方法，通常用于算术计算。

| __and__, __lshift__,

__or__,

__rshift__,

__xor__ | __and__(self, *other*), __lshift__(self, *other*), __or__(self, *other*), __rshift_(self, *other*),

__xor__(self, *other*)

运算符 *x &* *y*, *x <<* *y*, *x \|* *y*, *x >>* *y*, 和 *x ^* *y* 分别调用这些方法，通常用于位运算。

| __complex__, __float__,

__int__ | __complex__(self), __float__(self), __int__(self) 内置类型 complex(*x*), float(*x*), 和 int(*x*)，分别调用这些方法。

| __divmod__ | __divmod__(self, o*ther*) 内置函数 divmod(*x, y*) 调用 *x*.__divmod__(*y*)。__divmod__ 应返回一对 (*quotient, remainder*) 等于 (*x* // *y, x* % *y*)。 |
| --- | --- |

| __floordiv__, __truediv__ | __floordiv__(self, *other*)，__truediv__(self, *other*)，

运算符 *x* // *y* 和 *x* / *y*，通常用于算术除法。 |

| __iadd__, __ifloordiv__,

__imod__，

__imul__，

__isub__，

__itruediv__，

__imatmul__ | __iadd__(self, *other*)，__ifloordiv__(self, *other*)，

__imod__(self, *other*)，

__imul__(self, *other*)，

__isub__(self, *other*)，

__itruediv__(self, *other*)，

__imatmul__(self, *other*)，

增强赋值 *x* += *y*，*x* //= *y*，*x* %= *y*，*x* *= *y*，*x* -= *y*，*x* /= *y*，和 *x* @= *y*，分别调用这些方法。每个方法应该就地修改 *x* 并返回 self。当 *x* 是可变的时候定义这些方法（即，当 *x* *可以* 就地更改时）。 |

| __iand__, __ilshift__,

__ior__，

__irshift__，

__ixor__ | __iand_(self, *other*)，__ilshift_(self, *other*)，

__ior__(self, *other*)，

__irshift__(self, *other*)，

__ixor__(self, *other*)，

增强赋值 *x* &= *y*，*x* <<= *y*，*x* \= *y*，*x* >>= *y*，和 *x* ^= *y*，分别调用这些方法。每个方法应该就地修改 *x* 并返回 self。当 *x* 是可变的时候定义这些方法（即，当 *x* *可以* 就地更改时）。 |

| __index__ | __index__(self) 像 __int__ 一样，但只应由整数的替代实现类型提供（换句话说，该类型的所有实例都可以精确映射到整数）。例如，所有内置类型中，只有 int 提供 __index__；float 和 str 不提供，尽管它们提供 __int__。序列的索引和切片内部使用 __index__ 来获取所需的整数索引。 |
| --- | --- |
| __ipow__ | __ipow__(self,*other*) 增强赋值 *x* **= *y* 调用 *x*.__ipow__(*y*)。__ipow__ 应该就地修改 *x* 并返回 self。 |
| __matmul__ | __matmul__(self, *other*) 运算符 *x* @ *y* 调用这个方法，通常用于矩阵乘法。 |
| __pow__ | __pow__(self,*other*[, *modulo*]) *x* ** *y* 和 pow(*x, y*) 都调用 *x*.__pow__(*y*)，而 pow(*x, y, z*) 调用 *x*.__pow__(*y*, *z*)。 *x*.__pow__(*y, z*) 应该返回等于表达式 *x*.__pow__(*y*) % *z* 的值。 |

| __radd__, __rmod__,

__rmul__，

__rsub__，

__rmatmul__ | __radd__(self, *other*)，__rmod__(self, *other*)，

__rmul__(self, *other*)，

__rsub__(self, *other*)，

__rmatmul__(self, *other*)，

运算符 *y* + *x*，*y* / *x*，*y* % *x*，*y* * *x*，*y - x*，和 *y* @ *x*，分别在 *y* 没有所需方法 __add__，__truediv__ 等，或者当该方法返回 NotImplemented 时，在 *x* 上调用这些方法。 |

| __rand__, __rlshift__,

__ror__，

__rrshift__，

__rxor__ | __rand__(self, *other*)，__rlshift__(self, *other*)，

__ror__(self, *other*)，

__rrshift__(self, *other*)，

__rxor__(self, *other*)，

运算符 *y* & *x*，*y* << *x*，*y* &#124; *x*，*y* >> *x*，以及 *x* ^ *y* 分别在 *y* 没有所需方法 __and__，__lshift__，等等，或者当该方法返回 NotImplemented 时，在 *x* 上调用这些方法。|

| __rdivmod__ | __rdivmod_(self, *other*) 内置函数 divmod(*y*, *x*) 调用 *x*.__rdivmod__(*y*) 当 *y* 没有 __divmod__，或者当该方法返回 NotImplemented 时。__rdivmod__ 应返回一个对 (*remainder*, *quotient*)。 |
| --- | --- |
| __rpow__ | __rpow__(self,*other*) *y* ** *x* 和 pow(*y*, *x*) 调用 *x*.__rpow__(*y*) 当 *y* 没有 __pow__，或者当该方法返回 NotImplemented 时。在这种情况下，没有三个参数的形式。 |

# 装饰器

在 Python 中，经常使用 *高阶函数*：接受函数作为参数并返回函数作为结果的可调用对象。例如，描述符类型，如 staticmethod 和 classmethod，在类体内可以使用，如 “类级方法” 所述：

```py
`def` f(cls, ...):
 *`# ...definition of f snipped...`*
f = classmethod(f)
```

然而，将 classmethod 的调用在 **def** 语句之后的文本上，对代码的可读性有所影响：当阅读 *f* 的定义时，代码的读者尚不知道 *f* 将成为类方法而不是实例方法。如果在 **def** 前面提到 classmethod，则代码更易读。为此，使用称为 *装饰* 的语法形式：

```py
@classmethod
`def` f(cls, ...):
 *`# ...definition of f snipped...`*
```

装饰器，这里的 @classmethod，必须紧随其后的 **def** 语句，并意味着 *f* = *classmethod*(*f*) 在 **def** 语句之后立即执行（无论 *f* 是 **def** 定义的任何名称）。更一般地，@*expression* 评估表达式（必须是一个名称，可能是限定的，或者是一个调用），并将结果绑定到一个内部临时名称（比如，*__aux*）；任何装饰器必须紧跟在 **def**（或 **class**）语句之后，并意味着 *f* = *__aux*(*f*) 在 **def** 或 **class** 语句之后立即执行（无论 *f* 是 **def** 或 **class** 定义的任何名称）。绑定到 *__aux* 的对象称为 *装饰器*，它被称为 *装饰* 函数或类 *f*。

装饰器是一种便捷的高阶函数缩写。你可以将装饰器应用于任何 **def** 或 **class** 语句，不仅限于类体内。你可以编写自定义装饰器，它们只是接受函数或类对象作为参数，并返回函数或类对象作为结果的高阶函数。例如，这是一个简单的装饰器示例，它不修改其装饰的函数，而是在函数定义时将函数的文档字符串打印到标准输出：

```py
`def` showdoc(f):
    `if` f.__doc__:
        print(f'{f.__name__}: {f.__doc__}')
    `else`:
        print(f'{f.__name__}: No docstring!')
    `return` f

@showdoc
`def` f1():
    """a docstring"""  *`# prints:`* *`f1: a docstring`*

@showdoc
`def` f2():
    `pass`               *`# prints:`* *`f2: No docstring!`*
```

标准库模块 functools 提供了一个方便的装饰器 wraps，用于增强常见的“包装”习惯建立的装饰器：

```py
import functools

`def` announce(f):
    @functools.wraps(f)
    `def` wrap(*a, **k):
        print(f'Calling {f.__name__}')
        `return` f(*a, **k)
    `return` wrap
```

使用 @announce 装饰函数 *f* 导致在每次调用 *f* 之前打印一行公告。由于 functools.wraps(*f*) 装饰器，包装器采用被包装函数的名称和文档字符串：例如，在调用这样一个装饰过的函数时调用内置帮助是有用的。

# 元类

任何对象，甚至是类对象，都有一种类型。在 Python 中，类型和类也是一等对象。类对象的类型也称为类的 *元类*。¹⁵ 对象的行为主要由对象的类型确定。对于类也是如此：类的行为主要由类的元类确定。元类是一个高级主题，您可能想跳过本节的其余部分。但是，完全掌握元类可以带您深入了解 Python；偶尔定义自己的自定义元类可能会有用。

## 简单类定制的替代方法元类。

虽然自定义元类允许您以几乎任何想要的方式调整类的行为，但通常可以通过编写自定义元类来更简单地实现一些自定义。

当类 *C* 具有或继承类方法 __init_subclass__ 时，Python 在每次对 *C* 进行子类化时调用该方法，将新构建的子类作为唯一的位置参数传递。__init_subclass__ 也可以有命名参数，在这种情况下，Python 会传递在执行子类化的类语句中找到的相应命名参数。作为一个纯粹的说明性例子：

```py
>>> `class` C:
...     `def` __init_subclass__(cls, foo=None, **kw):
...         print(cls, kw)
...         cls.say_foo = staticmethod(lambda: f'*{foo}*')
...         super().__init_subclass__(**kw)
... 
>>> `class` D(C, foo='bar'):
...     `pass`
...
```

```py
<class '__main__.D'> {}
```

```py
>>> D.say_foo()
```

```py
'*bar*'
```

__init_subclass__ 中的代码可以以适用的方式修改 cls，在类创建后工作方式上本质上像一个 Python 自动应用于 C 的任何子类的类装饰器。

另一个用于定制的特殊方法是 __set_name__，它允许您确保将描述符的实例添加为类属性时，它们知道您正在向其添加的类和名称。在将 *ca* 添加到名为 *C* 的类并命名为 *n* 的类语句的末尾，当 *ca* 的类型具有方法 __set_name__ 时，Python 调用 *ca.*__set_name__(*C,* *n*)。例如：

```py
>>> `class` Attrib:
...     `def` __set_name__(self, cls, name):
...         print(f'Attribute {name!r} added to {cls}')
... 
>>> `class` AClass:
...     some_name = Attrib()
...
```

```py
Attribute 'some_name' added to <class '__main__.AClass'>
```

```py
>>>
```

## 如何确定 Python 类的元类。

**class** 语句接受可选的命名参数（在基类之后，如果有）。最重要的命名参数是 metaclass，如果存在，则标识新类的元类。如果存在非类型元类，则还允许其他命名参数，此时这些参数传递给元类的可选 __prepare__ 方法（完全由 __prepare__ 方法决定如何使用此类命名参数）。¹⁶ 当命名参数 metaclass 不存在时，Python 通过继承来确定元类；对于没有明确指定基类的类，默认元类为 type。

Python 调用 __prepare__ 方法（如果存在）来确定元类后立即调用元类，如下所示：

```py
`class` M:
    `def` __prepare__(classname, *classbases, **kwargs):
        `return` {}
 *`# ...rest of M snipped...`*
`class` X(onebase, another, metaclass=M, foo='bar'):
 *`# ...body of X snipped...`*
```

在这里，调用等同于 M.__prepare__('X', onebase, another, foo='bar')。如果存在 __prepare__，则必须返回映射（通常只是字典），Python 将其用作执行类体的*d*映射。如果不存在 __prepare__，Python 将使用一个新的、最初为空的字典作为*d*。

## 元类如何创建类

确定了元类*M*后，Python 使用三个参数调用*M*：类名（一个字符串）、基类元组*t*和字典（或其他由 __prepare__ 生成的映射）*d*，其中类体刚刚执行完毕。¹⁷ 这个调用返回类对象*C*，Python 随后将其绑定到类名上，完成**class**语句的执行。注意，这实际上是类型*M*的实例化，因此对*M*的调用执行*M.*__init__(*C*, *namestring*, *t*, *d*)，其中*C*是*M.*__new__(*M, namestring, t, d*)的返回值，就像在任何其他实例化中一样。

Python 创建类对象*C*之后，类*C*与其类型（通常为*M*的类型）之间的关系与任何对象与其类型之间的关系相同。例如，当你调用类对象*C*（创建*C*的实例）时，*M.*__call__ 执行，类对象*C*作为第一个参数。

注意，在这种情况下，描述的方法（“按实例方法”）的方法，仅在类上查找特殊方法，而不是在实例上。调用*C*实例化它必须执行元类的*M.*__call__，无论*C*是否具有每实例属性（方法）__call__（即，独立于*C*的实例是否可调用）。这种方式，Python 对象模型避免了必须将类及其元类的关系作为专门情况的问题。避免专门情况是 Python 强大的关键：Python 有少量、简单、通用的规则，并且一贯地应用这些规则。

### 自定义元类的定义和使用

定义自定义元类很容易：继承自 type 并重写其部分方法。你还可以使用 __new__、__init__、__getattribute__ 等方法执行大多数你可能考虑创建元类的任务，而不涉及元类。然而，自定义元类可能会更快，因为特殊处理仅在类创建时执行，这是一种罕见的操作。自定义元类允许你在框架中定义一整类具有你编码的任何有趣行为的类，这与类本身可能选择定义的特殊方法完全独立。

要以明确的方式修改特定的类，一个很好的替代方法通常是使用类装饰器，如 “装饰器” 中所述。然而，装饰器不会被继承，因此必须显式地将装饰器应用于每个感兴趣的类。¹⁸ 另一方面，元类是可以继承的；事实上，当你定义一个自定义元类 *M* 时，通常也会定义一个否则为空的类 *C*，其元类为 *M*，这样需要 *M* 的其他类可以直接继承自 *C*。

类对象的某些行为只能在元类中定制。下面的示例展示了如何使用元类来更改类对象的字符串格式：

```py
`class` MyMeta(type):
    `def` __str__(cls):
        `return` f'Beautiful class {cls.__name__!r}'
`class` MyClass(metaclass=MyMeta):
    `pass`
x = MyClass()
print(type(x))      *`# prints:`* *`Beautiful class 'MyClass'`*
```

### 一个实质性的自定义元类示例

假设在 Python 编程中，我们想念 C 语言的结构体类型：一个按顺序排列、具有固定名称的数据属性对象（数据类，在下一节中详细讨论，完全满足此需求，这使得此示例纯粹是说明性的）。Python 允许我们轻松定义一个通用的 Bunch 类，它与固定顺序和名称除外是类似的：

```py
`class` Bunch:
    `def` __init__(self, **fields):
        self.__dict__ = fields
p = Bunch(x=2.3, y=4.5)
print(p)       *`# prints:`* *`<_main__.Bunch object at 0x00AE8B10>`*
```

自定义元类可以利用属性名称在类创建时固定的事实。在 示例 4-1 中显示的代码定义了一个元类 MetaBunch 和一个类 Bunch，使我们能够编写如下代码：

```py
`class` Point(Bunch):
    *`"""A Point has x and y coordinates, defaulting to 0.0,`*
       *`and a color, defaulting to 'gray'-and nothing more,`*
       *`except what Python and the metaclass conspire to add,`*
       *`such as __init__ and __repr__.`*
    *`"""`*
    x = 0.0
    y = 0.0
    color = 'gray'
*`# example uses of class Point`*
q = Point()
print(q)                    *`# prints:`* *`Point()`*
p = Point(x=1.2, y=3.4)
print(p)                    *`# prints:`* *`Point(x=1.2, y=3.4)`*
```

在这段代码中，print 调用会生成我们的 Point 实例的可读字符串表示。Point 实例非常节省内存，并且它们的性能基本上与前面示例中简单类 Bunch 的实例相同（由于对特殊方法的隐式调用没有额外开销）。示例 4-1 非常实质性，要理解其所有细节需要掌握本书后面讨论的 Python 方面，比如字符串（在 第九章 中讨论）和模块警告（在 “warnings 模块” 中讨论）。在 示例 4-1 中使用的标识符 mcl 表示“元类”，在这种特殊的高级情况下比 cls 表示“类”更清晰。

##### 示例 4-1\. MetaBunch 元类

```py
`import` warnings
`class` MetaBunch(type):
    *`"""`*
    *`Metaclass for new and improved "Bunch": implicitly defines`*
    *`__slots__, __init__, and __repr__ from variables bound in`*
    *`class scope.`*
    *`A class statement for an instance of MetaBunch (i.e., for a`*
    *`class whose metaclass is MetaBunch) must define only`*
    *`class-scope data attributes (and possibly special methods, but`*
    *`NOT __init__ and __repr__). MetaBunch removes the data`*
    *`attributes from class scope, snuggles them instead as items in`*
    *`a class-scope dict named __dflts__, and puts in the class a`*
    *`__slots__ with those attributes' names, an __init__ that takes`*
    *`as optional named arguments each of them (using the values in`*
    *`__dflts__ as defaults for missing ones), and a __repr__ that`*
    *`shows the repr of each attribute that differs from its default`*
    *`value (the output of __repr__ can be passed to __eval__ to make`*
    *`an equal instance, as per usual convention in the matter, if`*
    *`each non-default-valued attribute respects that convention too).`*
    *`The order of data attributes remains the same as in the`* *`class body.`*
    *`"""`*
    `def` __new__(mcl, classname, bases, classdict):
        *`"""Everything needs to be done in __new__, since`*
           *`type.__new__ is where __slots__ are taken into account.`*
        *`"""`*
        *`# Define as local functions the __init__ and __repr__ that`*
        *`# we'll use in the new class`*
        `def` __init__(self, **kw):
            *`"""__init__ is simple: first, set attributes without`*
 *`explicit values to their defaults; then, set`* *`those`*
 *`explicitly`* *`passed in kw.`*
            *`"""`*
            `for` k `in` self.__dflts__:
                `if` `not` k `in` kw:
                    setattr(self, k, self.__dflts__[k])
            `for` k `in` kw:
                setattr(self, k, kw[k])
        `def` __repr__(self):
            *`"""__repr__ is minimal: shows only attributes that`*
               *`differ`* *`from default values, for compactness.`*
            *`"""`*
            rep = [f'{k}={getattr(self, k)!r}'
                    `for` k `in` self.__dflts__
                    `if` getattr(self, k) != self.__dflts__[k]
                  ]
            `return` f'{classname}({', '.join(rep)})'
        *`# Build the newdict that we'll use as class dict for the`*
        *`# new class`*
        newdict = {'__slots__': [], '__dflts__': {},
                   '__init__': __init__, '__repr__' :__repr__,}
        `for` k `in` classdict:
            `if` k.startswith('__') `and` k.endswith('__'):
                *`# Dunder methods: copy to newdict, or warn`*
                *`# about conflicts`*
                `if` k `in` newdict:
                    warnings.warn(f'Cannot set attr {k!r}'
                                  f' in bunch-class {classname!r}')
 `else``:`
                    newdict[k] = classdict[k]
 `else``:`
                *`# Class variables: store name in __slots__, and`*
                *`# name and value as an item in __dflts__`*
                newdict['__slots__'].append(k)
                newdict['__dflts__'][k] = classdict[k]
        *`# Finally, delegate the rest of the work to type.__new__`*
        `return` super().__new__(mcl, classname, bases, newdict)

`class` Bunch(metaclass=MetaBunch):
    *`"""For convenience: inheriting from Bunch can be used to get`*
       *`the new metaclass (same as defining metaclass= yourself).`*
    *`"""`*
 `pass`
```

## 数据类

正如前面的 Bunch 类所示，一个其实例仅仅是一组命名数据项的类是非常方便的。Python 的标准库通过 dataclasses 模块涵盖了这一点。

你将使用 dataclass 函数，它是 dataclasses 模块的主要特性：一种装饰器，你可以将其应用于希望成为一组命名数据项的任何类的实例。作为一个典型的例子，考虑以下代码：

```py
`import` dataclasses
`@`dataclasses.dataclass
`class` Point:
    x: float
    y: float
```

现在，您可以调用例如 pt = Point(0.5, 0.5)，并获得一个具有 pt.x 和 pt.y 属性的变量，每个属性都等于 0.5。默认情况下，dataclass 装饰器已经为 Point 类赋予了一个接受属性 x 和 y 的初始浮点值的 __init__ 方法，并准备好适当显示类的任何实例的 __repr__ 方法：

```py
>>> pt
```

```py
Point(x=0.5, y=0.5)
```

dataclass 函数接受许多可选的命名参数，以便调整装饰的类的详细信息。您可能经常明确使用的参数列在 Table 4-8 中。

Table 4-8\. dataclass 函数常用参数

| Parameter name | Default value and resulting behavior |
| --- | --- |
| eq | **True** 当为 **True** 时，生成一个 __eq__ 方法（除非类已定义了一个） |
| frozen | **False** 当为 **True** 时，使得类的每个实例为只读（不允许重新绑定或删除属性） |
| init | **True** 当为 **True** 时，生成一个 __init__ 方法（除非类已定义了一个） |
| kw_only | **False** 3.10+ 当为 **True** 时，强制要求将参数传递给 __init__ 方法时使用命名方式，而非位置方式 |
| order | **False** 当为 **True** 时，生成顺序比较的特殊方法（如 __le__、__lt__ 等），除非类已定义这些方法 |
| repr | **True** 当为 **True** 时，生成一个 __repr__ 方法（除非类已定义了一个） |
| slots | **False** 3.10+ 当为 **True** 时，向类添加适当的 __slots__ 属性（为每个实例节省一些内存，但不允许向类实例添加其他任意属性） |

当设置 frozen 为 **True** 时，装饰器还会为类添加一个 __hash__ 方法（允许实例作为字典的键和集合的成员），当这是安全的时候（通常是这样的情况）。即使在不安全的情况下，您也可以强制添加 __hash__ 方法，但我们强烈建议您不要这样做；如果您坚持要这样做，请查阅 [online docs](https://oreil.ly/rOJTW) 了解详细信息。

如果需要在自动生成的 __init__ 方法完成为每个实例属性分配核心工作后调整数据类的每个实例，请定义一个名为 __post_init__ 的方法，装饰器将确保在 __init__ 完成后立即调用它。

假设您希望向 Point 添加一个属性，以捕获创建点的时间。可以将其添加为在 __post_init__ 中分配的属性，为 Point 的定义成员添加名为 create_time 的属性，类型为 float，默认值为 0，并添加一个 __post_init__ 的实现：

```py
`def` __post_init__(self):
    self.create_time = time.time()
```

现在，如果您创建变量 pt = Point(0.5, 0.5)，打印它将显示创建时间戳，类似于以下内容：

```py
>>> pt
```

```py
Point(x=0.5, y=0.5, create_time=1645122864.3553088)
```

与常规类似，dataclass 还支持额外的方法和属性，例如计算两个点之间距离的方法以及返回到原点的点的距离的属性：

```py
`def` distance_from(self, other):
    dx, dy = self.x - other.x, self.y - other.y
    `return` math.hypot(dx, dy)

@property
`def` distance_from_origin(self):
    `return` self.distance_from(Point(0, 0))
```

例如：

```py
>>> pt.distance_from(Point(-1, -1))
```

```py
2.1213203435596424
```

```py
>>> pt.distance_from_origin
```

```py
0.7071067811865476
```

dataclasses 模块还提供了 asdict 和 astuple 函数，每个函数的第一个参数都是 dataclass 实例，分别返回一个字典和一个元组，这些字典和元组包含类的字段。此外，该模块还提供了一个 field 函数，用于自定义数据类字段（即实例属性）的处理方式，以及几个其他专门用于非常高级、神秘目的的函数和类；要了解有关它们的全部信息，请查阅[在线文档](https://oreil.ly/rOJTW)。

## 枚举类型（Enums）

在编程时，通常希望创建一组相关的值，用于列举特定属性或程序设置的可能值，¹⁹ 无论它们是什么：终端颜色、日志级别、进程状态、扑克牌花色、服装尺寸，或者你能想到的任何其他东西。*枚举类型*（enum）是定义这种值组的一种类型，具有可作为类型化全局常量使用的符号名称。Python 提供了 enum 模块中的 Enum 类及其相关子类用于定义枚举。

定义一个枚举为你的代码提供了一组代表枚举中的值的符号常量。在没有枚举的情况下，常量可能会被定义为整数，如下所示：

```py
*`# colors`*
RED = 1
GREEN = 2
BLUE = 3

*`# sizes`*
XS = 1
S = 2
M = 3
L = 4
XL = 5
```

然而，在这种设计中，没有机制可以警告类似 RED > XL 或 L * BLUE 这样的无意义表达式，因为它们都只是整数。也没有颜色或尺码的逻辑分组。

相反，你可以使用 Enum 子类来定义这些值：

```py
`from` enum `import` Enum, auto

`class` Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

`class` Size(Enum):
    XS = auto()
    S = auto()
    M = auto()
    L = auto()
    XL = auto()
```

现在，像 Color.RED > Size.S 这样的代码在视觉上显得不正确，并且在运行时会引发 Python TypeError。使用 auto() 自动分配从 1 开始递增的整数值（在大多数情况下，分配给枚举成员的实际值是无意义的）。

# 调用 Enum 创建一个类，而不是一个实例

令人惊讶的是，当你调用 enum.Enum() 时，它不会返回一个新建的*实例*，而是一个新建的*子类*。因此，前面的片段等效于：

```py
`from` enum `import` Enum
Color = Enum('Color', ('RED', 'GREEN', 'BLUE'))
Size = Enum('Size', 'XS S M L XL')
```

当你*调用* Enum（而不是在类语句中显式地对其进行子类化）时，第一个参数是你正在构建的子类的名称；第二个参数给出了该子类成员的所有名称，可以是字符串序列或单个以空格分隔（或逗号分隔）的字符串。

我们建议您使用类继承语法定义 Enum 子类，而不是这种简写形式。**类**形式更加视觉明确，因此更容易看出是否缺少、拼写错误或以后添加的成员。

枚举内部的值称为其*成员*。习惯上，使用全大写字符来命名枚举成员，将它们视为显式常量。枚举成员的典型用法包括赋值和身份检查：

```py
`while` process_state `is` ProcessState.RUNNING:
 *`# running process code goes here`*
    `if` processing_completed():
        process_state = ProcessState.IDLE
```

通过迭代枚举类本身或从类的 __members__ 属性获取，你可以获得枚举的所有成员。枚举成员都是全局单例，因此推荐使用 **is** 和 **is not** 进行比较，而不是 == 或 !=。

枚举模块包含几个类²⁰，支持不同形式的枚举，列在 表 4-9 中。

表 4-9\. 枚举类

| 类 | 描述 |
| --- | --- |
| 枚举 | 基本枚举类；成员值可以是任何 Python 对象，通常是整数或字符串，但不支持整数或字符串方法。适用于定义成员为无序组的枚举类型。 |
| Flag | 用于定义可以使用操作符 &#124;, &, ^ 和 ~ 进行组合的枚举；成员值必须定义为整数以支持这些位操作（Python 但是不假定它们之间的顺序）。值为 0 的 Flag 成员为假；其他成员为真。在创建或检查使用位操作的值时非常有用（例如文件权限）。为了支持位操作，通常使用 2 的幂次方（1、2、4、8 等）作为成员值。 |
| IntEnum | 相当于 **class** IntEnum(*int*, *Enum*)；成员值为整数并支持所有整数操作，包括排序。在需要对值进行排序时非常有用，比如定义日志级别。 |
| IntFlag | 相当于 **class** IntFlag(*int*, *Flag*)；成员值为整数（通常是 2 的幂次方），支持所有整数操作，包括比较。 |
| StrEnum | 3.11+ 相当于 **class** StrEnum(*str*, *Enum*)；成员值为字符串并支持所有字符串操作。 |

枚举模块还定义了一些支持函数，列在 表 4-10 中。

表 4-10\. 枚举支持函数

| 支持函数 | 描述 |
| --- | --- |
| 自动 | 在定义成员时自动递增成员值。通常值从 1 开始，每次增加 1；对于 Flag，增量为 2 的幂次方。 |
| 唯一 | 类装饰器，确保成员值彼此不同。 |

下面的示例展示了如何定义一个 Flag 子类，以处理从调用 os.stat 或 Path.stat 返回的 st_mode 属性中的文件权限（有关 stat 函数的描述，请参见 第十一章）：

```py
`import` enum
`import` stat

`class` Permission(enum.Flag):
    EXEC_OTH = stat.S_IXOTH
    WRITE_OTH = stat.S_IWOTH
    READ_OTH = stat.S_IROTH
    EXEC_GRP = stat.S_IXGRP
    WRITE_GRP = stat.S_IWGRP
    READ_GRP = stat.S_IRGRP
    EXEC_USR = stat.S_IXUSR
    WRITE_USR = stat.S_IWUSR
    READ_USR = stat.S_IRUSR

    @classmethod
    `def` from_stat(cls, stat_result):
        `return` cls(stat_result.st_mode & 0o777)

`from` pathlib `import` Path

cur_dir = Path.cwd()
dir_perm = Permission.from_stat(cur_dir.stat())
`if` dir_perm & Permission.READ_OTH:
    print(f'{cur_dir} is readable by users outside the owner group')

*`# the following raises TypeError: Flag enums do not support order`* 
*`# comparisons`*
print(Permission.READ_USR > Permission.READ_OTH)
```

在代码中使用枚举替代任意的整数或字符串可以提升可读性和类型完整性。你可以在 Python [文档](https://oreil.ly/d57vE) 中找到枚举模块的更多类和方法的详细信息。

¹ 或者，根据一位评论者的观点，也可以说是“缺点”。有人的福祸相依。

² 当情况如此时，在 metaclass= 后也可以有其他命名参数。这些参数（如果有）将传递给元类。

³ 这种需求是因为在 Singleton 的任何子类上定义了这个特殊方法的情况下，__init__ 会在每次你实例化子类时重复执行，在每个 Singleton 子类的唯一实例上执行。

⁴ 除了定义了 __slots__ 的类的实例外，涵盖在 “__slots__” 中。

⁵ 其他一些面向对象语言，如[Modula-3](https://en.wikipedia.org/wiki/Modula-3)，同样需要显式地使用 self。

⁶ 多个 Python 版本之后，Michele 的论文仍然适用！

⁷ 其中一位作者使用这种技术动态组合小的混合测试类，创建复杂的测试用例类来测试多个独立的产品特性。

⁸ 为了完整引用常被截断的名言：“当然除了太多的间接问题。”

⁹ 第三方扩展还可以定义不是序列、映射或集合的容器类型。

¹⁰ 包含下限，排除上限——这一点对于 Python 来说一直是规范。

¹¹ 参见例如 [“避免扩展类”](https://oreil.ly/5B4nm)，作者是 Bill Harlan。

¹² 关于类型检查的相关概念，请参阅 typing.Protocols，涵盖在 “协议” 中。

¹³ abc 模块确实包含 abstractproperty 装饰器，它结合了这两者，但 abstractproperty 已经弃用，新代码应该按描述使用这两个装饰器。

¹⁴ 为了向后兼容性，这些 ABCs 在 Python 3.9 之前也可以在 collections 模块中访问，但在 Python 3.10 中移除了兼容性导入。新代码应该从 collections.abc 导入这些 ABCs。

¹⁵ 严格来说，类 *C* 的类型可以说是 *C* 的实例的元类，而不是 *C* 本身，但这种微妙的语义区别在实践中很少被注意到。

¹⁶ 或者当基类有 __init_subclass__ 的情况下，命名参数将传递给该方法，如 “简单类定制的替代方案” 中描述的那样。

¹⁷ 这类似于调用 type 函数的三个参数版本，如 “使用 type 内置函数动态定义类” 中描述的那样。

¹⁸ __init_subclass__，在 “简单类定制的替代方案” 中讨论过，工作方式类似于“继承装饰器”，因此通常是自定义元类的替代选择。

¹⁹ 不要将这个概念与无关的内置函数 `enumerate` 混淆，该函数在 第八章 中介绍，它从可迭代对象生成 (*序号*, *项*) 对。

²⁰ `enum` 的专用元类与通常的类型元类行为差异如此之大，以至于值得指出 `enum.Enum` 和普通类之间的所有差异。你可以在 Python 在线文档的 [“枚举有何不同？”章节](https://oreil.ly/xpp5N) 中阅读有关内容。
