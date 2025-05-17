# 第五章：5 阅读 Python 代码：第二部分

### 本章内容

+   使用循环按需要的次数重复代码

+   使用缩进来告诉 Python 哪些代码属于同一组

+   构建字典以存储关联的值对

+   设置文件以读取和处理数据

+   使用模块在新领域中工作

在第四章中，我们探讨了五个 Python 特性，这些特性会在你继续编程时经常遇到：函数、变量、条件语句（`if` 语句）、字符串和列表。你需要了解这些特性才能阅读代码，我们也解释了无论是否使用 Copilot，能读懂代码都是非常重要的。

在本章中，我们将继续介绍另外五个 Python 特性，完成我们的前 10 大特性。如同第四章一样，我们将通过结合自己的解释、Copilot 的解释以及在 Python 提示符下的实验来进行讲解。

## 5.1 你需要了解的 10 大编程特性：第二部分

本节详细介绍了你需要了解的 10 大编程特性中的下五个。让我们从上一章的第 6 个特性：循环开始继续讲解。

### 5.1.1 #6\. 循环

循环使计算机能够根据需要重复执行相同的代码块。如果我们需要的 10 大编程特性中有一个能够展示计算机为何如此有用，它就是这个特性。没有循环功能，我们的程序通常会按顺序逐行执行。当然，它们仍然可以调用函数并使用 `if` 语句做出决策，但程序的工作量将与我们编写的代码量成正比。循环不同：一个简单的循环就能轻松处理数千或数百万个值。

循环有两种类型：`for` 循环和 `while` 循环。一般来说，当我们知道循环需要执行多少次时，我们使用 `for` 循环；而当我们不知道时，我们使用 `while` 循环。例如，在第三章中，我们的 `best_word` 函数（在列表 5.1 中重现）使用了 `for` 循环，因为我们知道需要循环多少次：即对 `word_list` 中的每个单词执行一次！但在 `get_strong_password` 函数中，我们使用了 `while` 循环，因为我们无法预知用户输入多少次不安全密码后才会输入一个强密码。我们将从 `for` 循环开始，然后再介绍 `while` 循环。

##### 列表 5.1 第三章中的 `best_word` 函数

```py
def best_word(word_list):
 """
 word_list is a list of words.

 Return the word worth the most points.
 """
    best_word = ""
    best_points = 0
    for word in word_list:       **#1
        points = num_points(word)
        if points > best_points:
            best_word = word
            best_points = points
    return best_word**
```

**#1 这是一个 for 循环的示例。** **`for` 循环允许我们访问字符串或列表中的每个值。我们首先用字符串来试试：**

```py
>>> s = 'vacation'
>>> for char in s:       #1
...     print('Next letter is', char)    #2
...
Next letter is v
Next letter is a
Next letter is c
Next letter is a
Next letter is t
Next letter is i
Next letter is o
Next letter is n
```

#1 这段代码会对字符串 s 中的每个字符执行一次缩进代码。

#2 因为“vacation”有八个字母，所以这段代码将运行八次。

注意我们不需要为`char`赋值语句。那是因为它是一个特殊的变量，叫做循环变量，它由`for`循环自动管理。`char`代表字符，它是人们常用的循环变量的名字。`char`变量会自动赋值为字符串的每个字符。谈到循环时，我们常用*迭代*一词来指代每次通过循环时执行的代码。这里，比如我们可以说在第一次迭代时，`char`代表`v`；在第二次迭代时，`char`代表`a`；依此类推。还要注意，就像函数和`if`语句一样，循环内部的代码也有缩进。虽然这里循环体内只有一行代码，但就像函数和`if`语句一样，我们可以有更多的代码行。

让我们看一下这个`for`循环在列表上的示例（列表 5.2），展示我们如何像处理字符串的每个值一样处理列表中的每个值。我们还会在循环中添加两行代码，而不仅仅是一个，以展示它是如何工作的。

##### 列表 5.2 使用`for`循环的示例

```py
>>> lst = ['cat', 'dog', 'bird', 'fish']
>>> for animal in lst:               #1
...     print('Got', animal)       #2
...     print('Hello,', animal)   ** #2
...
Got cat
Hello, cat
Got dog
Hello, dog
Got bird
Hello, bird
Got fish
Hello, fish**
```

**#1 第一个是列表，所以这是一个在列表上的`for`循环。

#2 这段代码在每次迭代时执行。**列表 5.2 中的代码只是通过列表循环的其中一种方式。`for` `animal` `in` `lst` 的写法会在每次循环时将变量`animal`赋值为列表中的下一个值。或者，你也可以使用索引来访问列表中的每个元素。要做到这一点，我们需要了解内置的`range`函数。

`range`函数会给你一个范围内的数字。我们可以提供一个起始数字和一个结束数字，它会生成从起始数字到结束数字之间的范围，但不包括结束数字。为了查看`range`生成的数字，我们需要将`list`函数放在它周围。以下是使用`range`的示例：

```py
>>> list(range(3, 9))     #1
[3, 4, 5, 6, 7, 8]
```

#1 生成从 3 到 8 的范围（不是从 3 到 9！）

注意，它从`3`开始，包含了从`3`到`8`之间的所有值。也就是说，它包含了从起始值`3`到结束值`9`之间的所有数字，但不包括结束值`9`。

现在，`range`如何帮助我们写循环呢？好吧，和直接在范围中硬编码数字像 3 和 9 不同，我们可以包括一个字符串或列表的长度，像这样：

```py
>>> lst
['cat', 'dog', 'bird', 'fish'] 
>>> list(range(0, len(lst)))      #1
[0, 1, 2, 3]
```

#1 从 0 开始，直到（但不包括）第一个列表的长度。

注意，这里的范围值是 0、1、2、3，它们是我们`lst`列表的有效索引！因此，我们可以使用`range`来控制`for`循环，这样就可以访问字符串或列表中的每个有效索引。

我们可以使用`range`来执行列表 5.2 中的相同任务。请参阅列表 5.3 中的新代码。

##### 列表 5.3 使用`for`循环和`range`的循环示例

```py
>>> for index in range(0, len(lst)):        #1
...     print('Got', lst[index])        #2
...     print('Hello,', lst[index])    ** #2
...
Got cat
Hello, cat
Got dog
Hello, dog
Got bird
Hello, bird
Got fish
Hello, fish**
```

**#1 使用`range`函数的`for`循环

#2 使用索引变量对列表进行索引**  **我们这里使用了一个名为`index`的变量，但你也常常会看到人们为了简便起见使用`i`。该变量在第一次迭代时赋值为`0`，在第二次迭代时为`1`，在第三次迭代时为`2`，最后一次迭代时为`3`。它在`3`时停止，因为列表的长度是 4，而`range`在这个值之前停止。通过对列表进行索引，代码依次获取第一个元素、第二个元素、第三个元素，然后是第四个元素，使用递增的索引值。我们也可以不使用`0`来编写`for`循环；`range`将假设我们想要`0`和给定值之间的值，如下所示：

```py
for index in range(len(lst)):    #1
    print('Got', lst[index])
    print('Hello,', lst[index])
```

#1 使用一个参数时，`range`假设我们想从 0 开始。

我们将在这里结束关于`for`循环的讨论。但我们还没有结束循环的部分，因为有另一种类型的循环我们需要讨论：`while`循环。

我们使用`while`循环当我们不知道循环要执行多少次时。一个典型的例子就是在我们第三章的`get_strong_password`函数中。我们在这里重现了那段代码，见列表 5.4。

##### 列表 5.4 第三章的`get_strong_password`函数

```py
def get_strong_password():
 """
 Keep asking the user for a password until it’s a strong password,
 and return that strong password.
 """
    password = input("Enter a strong password: ")
    while not is_strong_password(password):           #1
        password = input("Enter a strong password: ")
    return password
```

#1 当密码不强时持续循环

我们不知道用户要输入多少次密码，直到他们输入一个强密码。可能是第一次尝试，第二次尝试，还是第 50 次尝试？谁知道呢。这就是我们需要使用`while`循环的原因：它确保用户不断被询问，直到他们提供一个强密码。

要编写`while`循环，我们需要在`while`后面跟一个布尔条件。如果条件为`True`，Python 将执行一次循环迭代。如果该布尔条件在迭代后仍然为`True`，那么它会执行下一次迭代。它会一直这样做，直到条件为`False`，此时循环结束。

现在，我们已经有足够的信息来理解我们的`get_strong_` `password`函数的代码。通过查看代码，我们可以看到它首先使用`input`函数请求用户输入密码。然后，接下来是一个`while`循环。这个循环会一直执行，直到密码是一个强密码。布尔条件`not` `is_strong_password(password)`等价于写成`is_strong_password` `(password)` `==` `False`。为了避免无限循环并提示用户再次尝试，我们在循环内又使用了一次`input`。我们需要在循环之前调用`input`来获取用户的第一次尝试。如果没有那样做，`password`变量将不存在，`while`循环那一行将会报错，因为它没有`password`值可以传递给`is_strong_password`函数。

别忘了，我们可以使用在第四章第 4.2 节学到的内容，请 Copilot 解释这段代码。这在你感觉自己不太理解代码的作用或如何工作时非常有用。以下是我们从 Copilot 得到的解释：

这段 Python 代码定义了一个名为`get_strong_password`的函数。该函数的目的是与用户互动，提示用户输入密码，并持续询问直到用户提供一个被认为是强密码的密码。强密码的标准在此函数中没有定义，而是由另一个名为`is_strong_password`的函数来确定。

该函数首先通过 Python 内置的`input`函数请求用户输入密码。输入的密码随后存储在`password`变量中。

接下来，函数进入一个`while`循环。该循环的条件是`not is_strong_password(password)`。这意味着只要`is_strong_password`函数在调用当前密码时返回`False`，循环就会继续执行。换句话说，循环将一直运行，直到用户输入一个被认为是强密码的密码。

在循环内部，函数再次提示用户输入密码。新的密码会替换之前存储在`password`变量中的密码。如果新的密码仍然不强，循环将继续提示用户。

一旦用户输入了一个被认为是强密码的密码（即`is_strong_password(password)`返回`True`），`while`循环条件变为`False`，循环结束。函数随后返回用户输入的强密码。

这个函数是使用`while`循环反复提示用户输入，直到输入符合某些标准的一个很好的示例。然而，强密码的实际标准并未在此函数中定义，需要在`is_strong_password`函数中实现。

这个解释很好地描述了该函数，并且与我们刚才提供的描述相符！它还包含了一些有用的 Python 常识，包括最后一段关于我们为何使用`while`循环的提醒。解释中还提到我们需要`is_strong_password`函数来判断密码是否强大；我们并未特别强调这个函数，这就是为什么 Copilot 提示我们需要实现它。

##### Copilot 的解释可能是错误的

我们选择了前一个 Copilot 的解释，因为它是在我们要求 Copilot 解释代码三次后得到的最佳答案。它给出的其中一个答案听起来相当合理，直到开始提到一些不存在的函数。我们认为这些解释可以作为学习的辅助工具，如果你多次运行并寻找共同的思路，它们是有帮助的，但本章的主要目标是给你提供理解它犯错时的工具。

我们鼓励你以后使用 Copilot 的解释，并且如果你有兴趣，可以让 Copilot 解释任何你仍然好奇的之前章节中的代码。同样，这些解释可能是错误的，因此你应该向 Copilot 请求多个解释，以避免只依赖一个可能有误的解释。

正如目前与 AI 编程助手相关的一切一样，它们会犯错。但我们在这里给出解释，因为我们认为这个 Copilot 功能现在是一个潜在的强大教学资源，随着 Copilot 的不断改进，这一功能将变得更加有用。

在这种情况下，我们应该使用`while`循环，因为我们不知道迭代次数。但即使我们知道迭代次数，*仍然*可以使用`while`循环。例如，我们可以使用`while`循环处理字符串中的字符或列表中的值。我们有时会看到 Copilot 在生成的代码中这样做，尽管使用`for`循环会是更好的选择。例如，我们可以使用`while`循环处理我们之前的`animals`列表中的动物，如以下列表所示。不过，这需要更多的工作！

##### 列表 5.5 使用`while`循环的示例

```py
>>> lst
['cat', 'dog', 'bird', 'fish'] 
>>> index = 0
>>> while index < len(lst):        #1
...     print('Got', lst[index])
...     print('Hello,', lst[index])
...     index += 1          #2
...
Got cat
Hello, cat
Got dog
Hello, dog
Got bird
Hello, bird
Got fish
Hello, fish
```

#1 `len`告诉我们字符串的长度，也就是我们想要的迭代次数。

#2 这是常见的人为错误，容易遗漏！

如果没有`index` `+=` `1`，我们将无法在字符串中增加索引，打印出的信息将一直是第一个值。这被称为*无限循环*。如果回想一下我们写的`for`循环，你会发现我们并不需要手动增加任何索引变量。出于这种原因，许多程序员在可以使用时更倾向于使用`for`循环。我们不需要在`for`循环中手动追踪任何索引，因此自动避免了某些索引问题和无限循环。

### 5.1.2 #7\. 缩进

缩进在 Python 代码中至关重要，因为 Python 通过它来确定哪些代码行是一起执行的。例如，我们总是缩进函数内部的所有代码行，`if`语句的各个部分，以及`for`或`while`循环的代码。这不仅仅是格式问题：如果我们缩进错误，代码就会出错。例如，假设我们想询问用户当前的小时数，然后根据早晨、下午或晚上的时间输出一些文本：

+   如果是早晨，我们想输出“Good morning!”和“Have a nice day.”

+   如果是下午，我们想输出“Good afternoon!”

+   如果是晚上，我们想输出“Good evening!”和“Have a good night.”

看一下我们编写的以下代码，尝试找出缩进的问题：

```py
hour = int(input('Please enter the current hour from 0 to 23: '))

if hour < 12:
    print('Good morning!')
    print('Have a nice day.')
elif hour < 18:
    print('Good afternoon!')
else:
    print('Good evening!')
print('Have a good night.')     #1
```

#1 这一行没有缩进。

问题出在最后一行：它没有缩进，但应该有！由于没有缩进，无论用户输入什么小时，我们都会输出`Have` `a` `good` `night.`。我们需要缩进它，使其成为`if`语句的`else`部分，确保它只有在晚上时才会执行。

每当我们编写代码时，我们需要使用多个缩进级别来表示哪些代码块与函数、`if` 语句、循环等相关联。例如，当我们写一个函数头时，我们需要缩进所有与该函数相关的代码。如果你已经处于函数体内（一层缩进），并且写了一个循环，那么你就需要再缩进一次（二层缩进）来表示循环体，依此类推。

回顾我们第三章的函数，可以看到这一点的应用。例如，在我们的 `larger` 函数中（作为列表 5.6 复印），整个函数体都有缩进，但 `if` 部分和 `else` 部分的 `if` 语句有进一步的缩进。

##### 列表 5.6 用于确定两个值较大的函数

```py
def larger(num1, num2):
    if num1 > num2:       #1
        return num1    #2
    else:                 #3
        return num2       #4
```

#1 这里展示了函数主体的单个缩进。

#2 这里展示了函数体和 `if` 语句体的双重缩进。

#3 这里展示了函数主体的单个缩进。

#4 这里展示了函数体和 `else` 语句体的双重缩进。

接下来，考虑我们之前在列表 5.4 中看到的 `get_strong_password` 函数：像往常一样，函数中的所有内容都有缩进，但 `while` 循环的主体部分有进一步的缩进。

我们的 `num_points` 函数的第一个版本（从第三章作为列表 5.7 复现）有更多的缩进层次。这是因为，在对单词的每个字符进行 `for` 循环时，我们有一个 `if` 语句。正如我们所学，每个 `if` 语句的部分都需要缩进，这导致了额外的缩进层次。

##### 列表 5.7 `num_points` 函数

```py
def num_points(word): 
 """ 
 Each letter is worth the following points: 
 a, e, i, o, u, l, n, s, t, r: 1 point 
 d, g: 2 points 
 b, c, m, p: 3 points 
 f, h, v, w, y: 4 points 
 k: 5 points 
 j, x: 8 points 
 q, z: 10 points 

 word is a word consisting of lowercase characters. 
 Return the sum of points for each letter in word. 
 """
    points = 0
    for char in word:            #1
        if char in "aeioulnstr":     #2
            points += 1          #3
        elif char in "dg":
            points += 2
        elif char in "bcmp":
            points += 3
        elif char in "fhvwy":
            points += 4
        elif char == "k":
            points += 5
        elif char in "jx":
            points += 8
        elif char in "qz":
            points += 10
    return points
```

#1 这里进行了缩进，以便在函数内部。

#2 这里再次进行了缩进，以便在 `for` 循环内部。

#3 这里再次进行了缩进，以便在 if 语句内部。

`is_strong_password` 中也有额外的缩进（从第三章作为列表 5.8 复现），但那只是为了将一行超长代码分布到多行中。注意，这些行的末尾有 `\`，这是允许我们将一行代码延续到下一行的字符。

##### 列表 5.8 `is_strong_password` 函数

```py
def is_strong_password(password):
 """
 A strong password has at least one uppercase character,
 at least one number, and at least one punctuation.

 Return True if the password is a strong password, 
 False if not.
 """
    return any(char.isupper() for char in password) and \     #1
           any(char.isdigit() for char in password) and \     #2
           any(char in string.punctuation for char in password)
```

#1 该行以反斜杠结束，以继续语句。

#2 这个缩进不是必须的，但对视觉上布局单个返回语句很有帮助。

类似地，我们的 `num_points` 第二版本（从第三章作为列表 5.9 复现）也有进一步的缩进，但那只是为了将字典分布到多行中，以提高可读性。

##### 列表 5.9 `num_points` 替代解决方案

```py
 def num_points(word): 
 """ 
 Each letter is worth the following points: 
 a, e, i, o, u, l, n, s, t, r: 1 point 
 d, g: 2 points 
 b, c, m, p: 3 points 
 f, h, v, w, y: 4 points 
 k: 5 points 
 j, x: 8 points 
 q, z: 10 points 

 word is a word consisting of lowercase characters. 
 Return the sum of points for each letter in word. 
 """ 
    points = {'a': 1, 'e': 1, 'i': 1, 'o': 1, 'u': 1, 'l': 1,     #1
              'n': 1, 's': 1, 't': 1, 'r': 1,        #2
              'd': 2, 'g': 2,
              'b': 3, 'c': 3, 'm': 3, 'p': 3,
              'f': 4, 'h': 4, 'v': 4, 'w': 4, 'y': 4,
              'k': 5,
              'j': 8, 'x': 8,
              'q': 10, 'z': 10}
    return sum(points[char] for char in word)
```

#1 我们可以将字典值写成多行。

#2 这个缩进不是必须的，但对视觉上布局字典很有帮助。

缩进对我们程序的最终行为有巨大影响。例如，让我们比较将两个循环放在一起与使用缩进将一个循环嵌套在另一个循环中的效果。以下是两个并列的循环：

```py
>>> countries = ['Canada', 'USA', 'Japan']
>>> for country in countries:       #1
...     print(country)
...
Canada
USA
Japan
>>> for country in countries:      #2
...     print(country)
...
Canada
USA
Japan
```

#1 这是第一个循环。

#2 这是第二个循环（发生在第一个循环之后）。

这导致我们输出相同的结果两次，因为我们在国家列表中分别循环了两次。现在，如果我们将这两个循环嵌套在一起，就会出现以下情况：

```py
>>> for country1 in countries:            #1
...     for country2 in countries:        #2
...         print(country1, country2)    #3
...
Canada Canada
Canada USA
Canada Japan
USA Canada
USA USA
USA Japan
Japan Canada
Japan USA
Japan Japan
```

#1 这是第一个循环。

#2 这是嵌套在第一个循环中的第二个循环。

#3 打印语句嵌套在第二个循环中，该循环又嵌套在第一个循环中。

我们为每个 `for` 循环使用了不同的变量名，`country1` 和 `country2`，以便我们可以同时引用它们。在 `country1` 循环的第一次迭代中，`country1` 引用的是 `加拿大`。在 `country2` 循环的第一次迭代中，`country2` 同样引用的是 `加拿大`。这就是为什么输出的第一行是 `加拿大` `加拿大`。你是否期望接下来的输出行是 `美国` `美国`？但实际情况并非如此！相反，`country2` 循环会进入它的下一次迭代，但 `country1` 循环仍然没有前进。`country1` 循环只有在 `country2` 循环完成时才会向前移动。这就是为什么我们会先看到 `加拿大` `美国` 和 `加拿大` `日本`，然后 `country1` 循环才会进入它的第二次迭代。当一个循环嵌套在另一个循环内部时，这叫做 *嵌套循环*。一般来说，当有嵌套时，内层循环（`for country2 in countries`）会完成它的所有步骤，外层循环（`for country1 in countries`）才会继续进行下一步，这又会重新启动内层循环。

如果你看到一个循环嵌套在另一个循环中，很有可能这两个循环是用来处理二维数据的。二维数据是按行和列组织的，就像你在表格中看到的那样（例如，表 5.1）。这种数据在计算机中非常常见，因为它包括基本的电子表格数据，如 CSV 文件、图片（例如照片或视频的一帧）、或者计算机屏幕的数据。

在 Python 中，我们可以使用列表来存储二维数据，其中的值本身也是其他列表。整体列表中的每个子列表代表一行数据，每一行都有每一列的值。例如，假设我们有一些关于 2018 年冬奥会花样滑冰奖牌的数据，如表 5.1 所示。

##### 表 5.1 2018 年冬奥会奖牌

| 国家 | 金牌 | 银牌 | 铜牌 |
| --- | --- | --- | --- |
| 加拿大  | 2  | 0  | 2  |
| OAR  | 1  | 2  | 0  |
| 日本  | 1  | 1  | 0  |
| 中国  | 0  | 1  | 0  |
| 德国  | 1  | 0  | 0  |

我们可以将其存储为一个列表，每个国家占一行：

```py
>>> medals = [[2, 0, 2],
...           [1, 2, 0],
...           [1, 1, 0],
...           [0, 1, 0],
...           [1, 0, 0]]
```

请注意，我们的列表列表仅存储了数值，并且我们可以通过引用其行和列来在列表列表中查找一个值（例如，日本的金牌对应的是索引 2 行和索引 0 列）。我们可以使用索引获取一整行数据：

```py
>>> medals[0]    #1
[2, 0, 2]
>>> medals[1]    **#2
[1, 2, 0]
>>> medals[-1]    **#3
[1, 0, 0]****
```

****#1 这是第 0 行（第一行）。

#2 这是第 1 行（第二行）。

#3 这是最后一行。****  ****如果我们对这个列表进行`for`循环，我们将一次获得每一整行：

```py
>>> for country_medals in medals:     #1
...     print(country_medals)
...
[2, 0, 2]
[1, 2, 0]
[1, 1, 0]
[0, 1, 0]
[1, 0, 0]
```

#1 `for`循环一次给我们一个列表的值（即一次一个子列表）。

如果我们只想从奖牌列表中获取特定的值（而不是整行），我们需要索引两次：

```py
>>> medals[0][0]   #1
2
>>> medals[0][1]    #2
0
>>> medals[1][0]    #3
1
```

#1 这是第 0 行，第 0 列。

#2 这是第 0 行，第 1 列。

#3 这是第 1 行，第 0 列。

假设我们想要逐个遍历每个值。为此，我们可以使用嵌套的`for`循环。为了帮助我们准确跟踪当前位置，我们将使用`range` `for`循环，这样我们就可以打印出当前的行和列号，以及存储在其中的值。

外部循环将遍历行，因此我们需要使用`range` `(len(medals))`来控制它。内部循环将遍历列。有多少列呢？列数就是某一行中的值的数量，因此我们可以使用`range(len(medals[0]))`来控制这个循环。

每一行输出将提供三个数字：行坐标、列坐标以及该行列中的值（奖牌数）。以下是代码和输出：

```py
>>> for i in range(len(medals)):          #1
...     for j in range(len(medals[i])):      #2
...             print(i, j, medals[i][j])
...
0 0 2
0 1 0
0 2 2
1 0 1
1 1 2
1 2 0
2 0 1
2 1 1
2 2 0
3 0 0
3 1 1
3 2 0
4 0 1
4 1 0
4 2 0
```

#1 遍历行

#2 循环当前行的列

请注意，在输出的前三行中，行保持不变，而列从 0 变化到 2。我们就是这样遍历第一行的。只有当行号增加到 1 时，我们才会开始处理第二行，并完成对该行的列 0 到 2 的工作。

嵌套循环为我们提供了一种系统化的方式来遍历二维列表中的每个值。当处理二维数据时，您将经常遇到它们，例如图像、棋盘游戏和电子表格。

### 5.1.3 #8\. 字典

请记住，在 Python 中，每个值都有一个特定的类型。因为我们可能会使用许多不同种类的值，所以类型非常多！我们已经讨论过使用数字来处理数值，布尔值来处理`True`/`False`值，字符串来处理文本，以及列表来处理其他值的序列，例如数字或字符串。

还有一个在 Python 中经常出现的类型，它叫做*字典*。当我们在 Python 中谈论字典时，我们并不是指单词及其定义的列表。在 Python 中，字典是一种非常有用的存储数据的方式，尤其是当你需要跟踪数据之间的关联时。例如，假设你想知道在你最喜欢的书中哪些单词使用得最多。你可以使用字典将每个单词映射到它使用的次数。这个字典可能非常庞大，但一个小版本的字典可能是这样的：

```py
>>> freq = {'DNA': 11, 'acquire': 11, 'Taxxon': 13, \
... 'Controller': 20, 'morph': 41}
```

字典中的每个条目都将一个单词映射到它的频率。例如，我们可以从这个字典中得知，单词 *DNA* 出现了 11 次，单词 *Taxxon* 出现了 13 次。这里的单词（*DNA*、*acquire*、*Taxxon* 等）被称为 *键*，而频率（11、11、13 等）被称为 *值*。因此，字典将每个键映射到它的值。我们不允许有重复的键，但正如这里显示的那样，具有重复值并不成问题。

在第二章（列出 2.1）我们看到一个字典，存储了每个四分卫的名字和他们的传球码数。在第三章，我们再次在第二个解决方案中看到了一个字典，用来表示 `num_points`（在 5.9 列表中复现）。在那里，字典将每个字母映射到使用该字母时获得的点数。

就像字符串和列表一样，字典也有方法可以让你与它们进行交互。以下是一些操作我们的`freq`字典的方法：

```py
>>> freq
{'DNA': 11, 'acquire': 11, 'Taxxon': 13, 'Controller': 20, 'morph': 41}
>>> freq.keys()                **#1
dict_keys(['DNA', 'acquire', 'Taxxon', 'Controller', 'morph'])
>>> freq.values()                   #2
dict_values([11, 11, 13, 20, 41])
>>> freq.pop('Controller')         #3
20
>>> freq
{'DNA': 11, 'acquire': 11, 'Taxxon': 13, 'morph': 41}**
```

**#1 获取所有键

**#2 获取所有值

**#3 删除键及其关联值**  **你还可以使用索引符号来访问给定键的值：

```py
>>> freq['dna']  # Oops, wrong key name because it is case sensitive
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'dna'
>>> freq['DNA']       #1
11
>>> freq['morph']
41
```

**#1 获取与键 “DNA” 关联的值

字典像列表一样是可变的。这意味着我们可以更改字典中的键和值，这对于建模随时间变化的数据非常有用。我们可以使用索引来更改一个值。当前与 `'morph'` 关联的值是 `41`，我们将其改为 `6`：

```py
>>> freq['morph'] = 6     #1
>>> freq
{'DNA': 11, 'acquire': 11, 'Taxxon': 13, 'morph': 6}
```

**#1 将与键 “morph” 关联的值改为 6

我们的 `freq` 字典允许我们从任何想要的单词开始，并找到它的频率。更一般地说，字典允许我们从 *键到值*。然而，它并不容易让我们从值到键反向查找。如果我们想做到这一点，我们需要生成一个反向字典——例如，键是频率，值是具有这些频率的单词列表。这样我们就能回答以下问题：哪些单词的频率恰好是 5？哪些单词的频率是所有单词中最小或最大的？

就像字符串和列表一样，我们也可以使用循环来处理字典中的信息。`for` 循环为我们提供字典的键，我们可以使用索引来获取每个键的关联值：

```py
>>> for word in freq:                         #1
...     print('Word', word, 'has frequency', freq[word])    #2
...
Word DNA has frequency 11
Word acquire has frequency 11
Word Taxxon has frequency 13
Word morph has frequency 6
```

*#1 遍历 freq 字典中的每个键

**#2 使用键（单词）及其关联值（freq[word]）**  *### 5.1.4 #9\. 文件

我们通常需要处理存在于文件中的数据集。例如，在第二章，我们使用了 NFL 统计数据文件来确定最有效的四分卫。使用文件是其他数据科学任务中常见的做法。例如，如果你正在绘制世界各地的地震信息或确定两本书是否由同一位作者写作，你也需要处理这些数据集，而这些数据集通常会存储在文件中。

在第二章，我们处理了一个名为 nfl_offensive_stats.csv 的文件。确保该文件位于当前程序目录中，因为我们将用它来进一步理解我们在第二章中使用的一些代码。

处理文件数据的第一步是使用 Python 的`open`函数打开文件：

```py
>>> nfl_file = open('nfl_offensive_stats.csv')
```

有时你会看到 Copilot 在这里添加`r`作为第二个参数：

```py
>>> nfl_file = open('nfl_offensive_stats.csv', 'r')
```

但我们不需要`r`；`r`只是表示我们想从文件中读取数据，但如果不指定，默认就是读取模式。

我们使用赋值语句将打开的文件分配给名为`nfl_file`的变量。现在，我们可以使用`nfl_file`来访问文件的内容。一个打开的文件是 Python 中的一种类型，就像数字、字符串以及你迄今为止看到的所有其他类型一样。因此，我们可以调用方法与文件进行交互。一个方法是`readline`，它返回文件的下一行作为字符串。我们现在就使用它来获取我们打开文件的第一行，但不用担心这一行，因为它很长，包含了很多关于我们最终不会使用的列的信息：

```py
>>> line = nfl_file.readline()     #1
>>> line
'game_id,player_id,position,player,team,pass_cmp,pass_att,pass_yds,pass_td,pass_int,pass_sacked,pass_sacked_yds,pass_long,pass_rating,rush_att,
rush_yds,rush_td,rush_long,targets,rec,rec_yds,rec_td,rec_long,
fumbles_lost,rush_scrambles,designed_rush_att,comb_pass_rush_play,
comb_pass_play,comb_rush_play,Team_abbrev,Opponent_abbrev,two_point_conv,
total_ret_td,offensive_fumble_recovery_td,pass_yds_bonus,rush_yds_bonus,
rec_yds_bonus,Total_DKP,Off_DKP,Total_FDP,Off_FDP,Total_SDP,Off_SDP,
pass_target_yds,pass_poor_throws,pass_blitzed,pass_hurried,
rush_yds_before_contact,rush_yac,rush_broken_tackles,rec_air_yds,rec_yac,
rec_drops,offense,off_pct,vis_team,home_team,vis_score,home_score,OT,Roof,
Surface,Temperature,Humidity,Wind_Speed,Vegas_Line,Vegas_Favorite,
Over_Under,game_date\n'
```

#1 从文件中读取一行

从这样的混乱字符串中提取单个值并不容易。因此，我们通常会先将这行数据拆分为各个列数据。我们可以使用字符串的`split`方法来做到这一点。该方法以分隔符作为参数，并使用该分隔符将字符串拆分成一个列表：

```py
>>> lst = line.split(',')    #1
>>> len(lst)
69
```

#1 使用逗号（,）作为分隔符来拆分字符串

现在我们可以查看各个列名：

```py
>>> lst[0]
'game_id'
>>> lst[1]
'player_id'
>>> lst[2]
'position '     #1
>>> lst[3]
'player'
>>> lst[7]
'pass_yds'
```

#1 这个词末尾的空格在原始数据集中存在，但其他列标题没有空格。

我们看到的文件中的第一行并不是真正的数据行——它只是告诉我们每列名称的标题。下次我们调用`readline`时，将获得第一行真正的数据：

```py
>>> line = nfl_file.readline()
>>> lst = line.split(',')
>>> lst[3]
'Aaron Rodgers'
>>> lst[7]
'203'
```

像这样逐行读取文件对于探索文件内容是没问题的，但最终我们可能希望处理整个文件。为此，我们可以对文件使用`for`循环。每次迭代它都会返回一行，我们可以按需要处理这行数据。一旦我们处理完文件，就应该调用`close`来关闭它：

```py
>>> nfl_file.close()
```

关闭后，我们将无法再使用该文件。现在我们已经讨论了如何读取、处理和关闭文件，让我们来看一个完整的例子。在列表 5.10 中，我们提供了来自第二章的新版本程序，该程序按四分卫的总传球码数对其进行排序。除了展示文件外，我们还使用了在第四章和本章中看到的许多 Python 特性，包括条件语句、字符串、列表、循环和字典。

##### 列表 5.10 没有 csv 模块的替代 NFL 统计代码

```py
nfl_file = open('nfl_offensive_stats.csv')
passing_yards = {}                    #1

for line in nfl_file:                #2
    lst = line.split(',')
    if lst[2] == 'QB':               #3
        if lst[3] in passing_yards:                   #4
            passing_yards[lst[3]] += int(lst[7])      #5
        else:                                         #6
            passing_yards[lst[3]] = int(lst[7])       #7

nfl_file.close()

for player in sorted(passing_yards, 
                     key=passing_yards.get, 
                     reverse=True):         #8
    print(player, passing_yards[player])
```

#1 这个字典将四分卫的名字映射到他们的传球码数。

#2 遍历文件的每一行

#3 只关注四分卫

#4 四分卫已经在我们的字典中了。

#5 累加四分卫的总数；`int` 将像 '203' 这样的字符串转换为整数。

#6 四分卫尚未出现在我们的字典中。

#7 设置四分卫的初始总数

#8 从最高到最低的传球码数遍历四分卫

最后面的那一行 `for player in sorted(passing_yards, key=passing_yards.get, reverse=True):` 涉及了很多内容。我们在注释中解释了这行代码是从最高到最低遍历四分卫。`reverse=True` 让我们按从高到低排序，而不是默认的从低到高排序。`key=passing_yards.get` 使排序的重点放在传球码数上（而不是，比如，球员的名字）。如果你想进一步解析这行代码，随时可以向 Copilot 请求更多解释。这突出了我们在这里要保持的平衡：了解足够的内容，能够把握代码的大意，而不一定需要理解每个细节。

这个程序运行得很好；如果你运行它，你会看到和运行第二章代码时相同的输出。不过，有时候，使用模块编写程序会更简单（我们将在下一节深入讲解模块），这正是第二章中的程序所做的。由于 CSV 文件非常常见，Python 提供了一个模块来简化对它们的处理。在第二章中，我们得到的解决方案使用了 csv 模块。所以，让我们讨论一下我们在列表 5.10 中未使用模块的代码和第二章代码之间的主要区别，下面的列表将再次展示第二章代码（我们提供给 Copilot 的提示未显示）。

##### 列表 5.11 使用 csv 模块的 NFL 统计代码

```py
# import the csv module
import csv

# open the csv file
with open('nfl_offensive_stats.csv', 'r') as f:    #1
    # read the csv data
    data = list(csv.reader(f))    #2

# create a dictionary to hold the player name and passing yards
passing_yards = {}

# loop through the data
for row in data:                  #3
    # check if the player is a quarterback
    if row[2] == 'QB':
        # check if the player is already in the dictionary
        if row[3] in passing_yards:
            # add the passing yards to the existing value
            passing_yards[row[3]] += int(row[7])
        else:
            # add the player to the dictionary
            passing_yards[row[3]] = int(row[7])

for player in sorted(passing_yards, key=passing_yards.get, reverse=True):
    print(player, passing_yards[player])
```

#1 显示打开文件的另一种语法

#2 使用特殊的 csv 模块；从文件中读取所有数据

#3 遍历每一行数据

首先，列表 5.11 使用 csv 模块来简化处理 CSV 文件的过程。csv 模块知道如何操作 CSV 文件，因此，例如，我们不需要担心将一行数据拆分成各列。其次，列表 5.11 使用了 `with` 关键字，这样文件在程序完成后会自动关闭。第三，列表 5.11 在进行任何处理之前会先读取整个文件。相比之下，在列表 5.10 中，我们在读取每一行时会立即对其进行处理。

##### 解决编程问题有多种方法

总是有多种不同的程序可以用来解决同一个任务。有些程序可能比其他程序更容易阅读。代码最重要的标准是它是否能正确完成任务。之后，我们最关心的是可读性和效率。因此，如果你发现自己难以理解某些代码的工作原理，可能值得花些时间看看 Copilot 的其他代码，看看是否有更简单或更易懂的解决方案。

文件在计算任务中广泛使用，因为它们是常见的数据来源，包括本节中的 CSV 文件、记录计算机或网站事件的日志文件，以及存储视频游戏中可能看到的图形数据的文件等等。由于文件的使用如此普遍，毫不奇怪有很多模块可以帮助我们读取各种文件格式。这引出了我们要讨论的更大话题：模块。

### 5.1.5 #10\. 模块

人们使用 Python 来做各种各样的事情——游戏、网站、数据分析应用程序、自动化重复任务、控制机器人等等。你可能会好奇，Python 如何让你创建这么多不同类型的程序。肯定的，Python 的创建者不可能预见到所有需要的支持，或者为此编写所有代码！

事实是，默认情况下，您的 Python 程序仅能访问一些核心的 Python 功能（比如我们在上一章和本章中展示的那些功能）。要获得更多功能，我们需要使用模块。而且，要使用模块，您需要先导入它。

##### Python 中的模块

一个*模块*是为特定目的设计的代码集合。回想一下，我们在使用函数时并不需要了解它是如何工作的。模块也是如此：我们不需要了解模块是如何工作的，就能使用它，就像我们不需要了解电灯开关是如何内部工作的就能使用它一样。作为模块的使用者，我们只需要知道模块能帮助我们做什么，以及如何编写正确的代码来调用它的函数。当然，Copilot 可以帮助我们编写这种代码。

一些模块在安装 Python 时会随 Python 一起提供，但我们仍然需要导入它们。其他模块则需要我们先安装，才能导入它们。相信我们，如果你有某种特定的任务要用 Python 完成，可能已经有人写过一个模块来帮你了。

你可能会想知道如何判断应该使用哪些 Python 模块。如何知道有哪些模块？与 Copilot 聊天或进行 Google 搜索通常能帮助你。例如，如果我们在 Google 上搜索“Python 模块来创建 zip 文件”，第一个结果告诉我们，我们需要的模块是 Python 标准库的一部分，这意味着它随 Python 一起提供。如果我们搜索“Python 可视化模块”，我们会了解到像 matplotlib、plotly、seaborn 等模块。搜索这些模块的结果应该会带你看到一些展示它们能力的可视化内容，以及它们通常用于哪些场景。大多数模块都是免费的，尽管你的搜索结果可以帮助你确认模块是否免费以及其具体的使用许可证。我们会在第九章时再开始安装和使用新安装的模块，但到那时，你会看到如何查找、安装并使用相关模块来帮助我们完成任务。

表 5.2 列出了常用的 Python 模块，以及它们是否是内置模块。如果一个模块是内置的，你可以直接导入并开始使用；如果不是，你需要先安装它。

##### 表 5.2 常用 Python 模块的总结

| 模块 | 内置 | 描述 |
| --- | --- | --- |
| csv  | 是  | 帮助读取、写入和分析 CSV 文件  |
| zipfile  | 是  | 帮助创建和提取压缩的.zip 归档文件  |
| matplotlib  | 否  | 用于绘图的图形库，是其他图形库的基础，并提供高度定制化功能  |
| plotly  | 否  | 一个用于创建交互式网页图表的图形库  |
| seaborn  | 否  | 基于 matplotlib 构建的图形库，比 matplotlib 更容易创建高质量的图表  |
| pandas  | 否  | 一个数据处理库，专注于数据框（类似于电子表格）  |
| scikit-learn  | 否  | 包含用于机器学习的基本工具（即帮助从数据中学习并做出预测）  |
| numpy  | 否  | 提供高效的数据处理功能  |
| pygame  | 否  | 一个游戏编程库，帮助在 Python 中构建交互式图形游戏  |
| django  | 否  | 一个帮助设计网站和网页应用程序的 Web 开发库  |

在第二章中，我们的代码使用了 Python 自带的 csv 模块。让我们继续学习一个 Python 自带的不同模块。

当人们想要整理他们的文件，可能是在备份或上传之前，他们通常会先将它们打包成.zip 文件。然后他们可以传送那个单独的.zip 文件，而不是可能数百个甚至数千个单独的文件。Python 自带一个叫做 zipfile 的模块，可以帮助你创建.zip 文件。

为了尝试这个，创建几个文件并将它们的扩展名都设为.csv。你可以从你的 nfl_offensive_stats.csv 文件开始，然后再添加几个。例如，你可以添加一个名为 actors.csv 的文件，列出几个演员的名字和他们的年龄，如

```py
Actor Name, Age
Anne Hathaway, 40
Daniel Radcliffe, 33
```

你可以添加一个名为 chores.csv 的文件，里面列出了一些家务事以及是否完成：

```py
Chore, Finished?
Clean dishes, Yes
Read Chapter 6, No
```

只要你有几个.csv 文件用于测试，内容并不重要。现在我们可以使用 zipfile 模块将它们全部添加到一个新的.zip 文件中！

```py
>>> import zipfile
>>> zf = zipfile.ZipFile('my_stuff.zip', 'w',
    ↪ zipfile.ZIP_DEFLATED)     **#1
>>> zf.write('nfl_offensive_stats.csv')       #2
>>> zf.write('actors.csv')       #3
>>> zf.write('chores.csv')   #4
>>> zf.close()**
```

**#1 创建新的.zip 文件

#2 添加第一个文件

#3 添加第二个文件

#4 添加第三个文件**  **如果你运行那个代码，你会发现一个名为 my_stuff.zip 的新文件，里面包含了你的三个.csv 文件。直接处理.zip 文件在其他早期编程语言中通常是一个非常专业且容易出错的任务，但在 Python 中并非如此。Python 自带的一些模块对数据科学、游戏开发、处理各种文件格式等方面都很有帮助，但同样，Python 并不能包含所有东西。当我们需要更多时，我们会转向可下载的模块，正如我们将在第九章中看到的那样。

在本章中，我们介绍了 Python 特性排名前 10 的后半部分，概览请见表 5.3。我们在前一章和本章已经讲了很多关于如何阅读代码的内容。虽然我们没有涵盖 Copilot 可能生成的所有代码，但你现在已经具备了检查 Copilot 代码的能力，能够判断它是否正确生成了你请求的代码。我们还展示了如何使用 Copilot 解释工具帮助你理解新的代码。在接下来的章节中，我们将学习如何测试 Copilot 生成的代码以确定其正确性，以及在代码不正确时可以做些什么。

##### 表 5.3 本章 Python 代码特性总结

| 代码元素 | 示例 | 简短描述 |
| --- | --- | --- |
| 循环  | `for` 循环：`for country in countries: print(country)` `while` 循环：`index = 0 while index < 4: print(index) index = index + 1`  | 循环允许我们根据需要多次运行相同的代码。当我们知道循环次数时（例如字符串中的字符数），使用 `for` 循环；当我们不知道循环次数时（例如，询问用户输入强密码），使用 `while` 循环。 |
| 缩进  | `for country in countries: print(country)`  | 缩进告诉 Python 代码块之间的关系，例如，`print` 调用属于 `for` 循环的一部分。 |
| 字典  | `points = {'a': 1, 'b': 3}`  | 字典允许我们将一个键与一个值关联。例如，键 `'a'` 与值 `1` 关联。 |
| 文件  | `file = open('chores.csv') first_line = file.readline()`  | 文件包含数据并存储在你的计算机上。Python 可以用来打开多种类型的文件并读取其内容，允许你处理文件中的数据。 |
| 模块  | `import` `csv`  | 模块是已经存在的库，提供额外的功能。常用的模块包括 csv、numpy、matplotlib、pandas 和 scikit-learn。一些模块随标准 Python 配发；另一些则需要单独安装。 |

## 5.2 练习

1.  回想一下我们在列表 5.3 中看到的 `for` 循环代码，用来打印列表中的动物。与本章中的原始示例相比，这段修改后的代码有什么不同之处？具体来说，它产生了哪些额外的输出？

```py
lst = ['cat', 'dog', 'bird', 'fish']

for index in range(len(lst)):
    print('Got', lst[index])
    if lst[index] == 'bird':
        print('Found the bird!')
    print('Hello,', lst[index])
```

1.  2. 考虑以下 `while` 循环代码，试图重复我们在列表 5.3 中使用 `for` 循环做的事情。当我们运行代码时，我们发现它会无限运行。你能找出并修复导致它无限运行的错误吗？

```py
lst = ['cat', 'dog', 'bird', 'fish']

index = 0
while index < len(lst):
    print('Got', lst[index])
    print('Hello,', lst[index])
```

1.  3. 将以下代码行排列成一个 `while` 循环，打印列表中的每个数字，直到遇到数字 7。注意缩进！

```py
 index += 1
 while index < len(numbers) and numbers[index] != 7:
 index = 0
 numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
 print(numbers[index])
```

1.  4. 想一想，在什么样的现实场景下使用 `while` 循环比 `for` 循环更合适。描述这个场景并解释为什么 `while` 循环是更好的选择。

1.  5\. 修改`get_strong_password`函数（或它调用的`is_strong_password`函数），以提供关于输入密码不够强的具体反馈。例如，如果密码没有大写字母，则打印“密码必须包含大写字母”；如果没有数字，则打印“密码必须至少包含一个数字”。

1.  6\. 给定以下`print_quarterbacks`函数，你能否重写它，使用“with”语句来打开和关闭文件？为什么关闭文件很重要？

```py
def print_quarterbacks():
    nfl_file = open('nfl_offensive_stats.csv')
    for line in nfl_file:
        lst = line.split(',')
        if lst[2] == 'QB':
            print(f"{lst[3]}: {lst[7]} passing yards")
    nfl_file.close()
```

1.  7\. 在这个练习中，我们将进一步练习使用 zipfile 模块来创建一个包含多个 CSV 文件的.zip 文件。按照以下步骤完成任务并回答问题：

    1.  首先，在当前目录下创建三个 CSV 文件：

        +   nfl_offensive_stats.csv（你应该已经有这个文件）

        +   actors.csv，包含以下内容：

            ```py
                          Actor Name, Age
                          Anne Hathaway, 40
                          Daniel Radcliffe, 33

            ```

        +   chores.csv，包含以下内容：

            ```py
                          Chore, Finished?
                          Clean dishes, Yes
                          Read Chapter 6, No

            ```

    1.  使用 Copilot（不要像我们在本章中那样直接输入代码），编写一个 Python 脚本，使用 zipfile 模块将这三个 CSV 文件添加到名为 my_stuff.zip 的.zip 文件中。

    1.  Copilot 建议的 zipfile 模块提供了哪些其他功能？它们如何有用？

## 总结

+   循环用于根据需要重复执行代码。

+   当我们知道循环将执行多少次时，我们使用`for`循环；当我们不知道循环将执行多少次时，我们使用`while`循环。

+   Python 使用缩进来确定哪些代码行是一起执行的。

+   字典是从键（例如，书中的单词）到值（例如，它们的频率）的映射。

+   在读取文件之前，我们需要先打开它。

+   一旦文件被打开，我们可以使用方法（例如，readline）或循环来读取文件的行。

+   一些模块，例如 csv 和 zipfile，是 Python 自带的，可以通过导入它们来使用。

+   其他模块，如 matplotlib，需要先安装，然后才能导入并使用。***************
