# 附录 E. 快速参考表

我发现自己经常查阅某些内容。这里有些希望您会发现有用的表格。

# 运算符优先级

此表格是 Python 3 中运算符优先级的官方文档改编版，优先级最高的运算符位于顶部。

| 运算符 | 描述和示例 |
| --- | --- |
| ``[*`v`*``, `…]`, ``{*`v1`*``, `…}`, ``{*`k1`*:*`v1`*``, `…}`, `(…)` | 列表/集合/字典/生成器的创建或推导，括号表达式 |
| ``*`seq`*[*`n`*]``, ``*`seq`*[*`n`*:*`m`*]``, ``*`func`*(*`args`*…)``, ``*`obj`*.*`attr`*`` | 索引、切片、函数调用、属性引用 |
| `**` | 指数运算 |
| `+`*`n`*, `–`*`n`*, `~`*`n`* | 正数、负数、位运算 `not` |
| `*`, `/`, `//`, `%` | 乘法、浮点除法、整数除法、取余 |
| `+`, `-` | 加法、减法 |
| `<<`, `>>` | 位左移、右移 |
| `&` | 位运算 `and` |
| `&#124;` | 位运算 `or` |
| `in`, `not in`, `is`, `is not`, `<`, `<=`, `>`, `>=`, `!=`, `==` | 成员关系和相等性测试 |
| `not` *x* | 布尔（逻辑）`not` |
| `and` | 布尔 `and` |
| `or` | 布尔 `or` |
| `if` … `else` | 条件表达式 |
| `lambda` … | `lambda` 表达式 |

# 字符串方法

Python 提供了字符串 *方法*（可用于任何 `str` 对象）和一个 `string` 模块，其中包含一些有用的定义。让我们使用这些测试变量：

```py
>>> s = "OH, my paws and whiskers!"
>>> t = "I'm late!"
```

在以下示例中，Python shell 打印方法调用的结果，但原始变量 `s` 和 `t` 并未更改。

## 更改大小写

```py
>>> s.capitalize()
'Oh, my paws and whiskers!'
>>> s.lower()
'oh, my paws and whiskers!'
>>> s.swapcase()
'oh, MY PAWS AND WHISKERS!'
>>> s.title()
'Oh, My Paws And Whiskers!'
>>> s.upper()
'OH, MY PAWS AND WHISKERS!'
```

## 搜索

```py
>>> s.count('w')
2
>>> s.find('w')
9
>>> s.index('w')
9
>>> s.rfind('w')
16
>>> s.rindex('w')
16
>>> s.startswith('OH')
True
```

## 修改

```py
>>> ''.join(s)
'OH, my paws and whiskers!'
>>> ' '.join(s)
'O H ,   m y   p a w s   a n d   w h i s k e r s !'
>>> ' '.join((s, t))
"OH, my paws and whiskers! I'm late!"
>>> s.lstrip('HO')
', my paws and whiskers!'
>>> s.replace('H', 'MG')
'OMG, my paws and whiskers!'
>>> s.rsplit()
['OH,', 'my', 'paws', 'and', 'whiskers!']
>>> s.rsplit(' ', 1)
['OH, my paws and', 'whiskers!']
>>> s.split(' ', 1)
['OH,', 'my paws and whiskers!']
>>> s.split(' ')
['OH,', 'my', 'paws', 'and', 'whiskers!']
>>> s.splitlines()
['OH, my paws and whiskers!']
>>> s.strip()
'OH, my paws and whiskers!'
>>> s.strip('s!')
'OH, my paws and whisker'
```

## 格式

```py
>>> s.center(30)
'  OH, my paws and whiskers!   '
>>> s.expandtabs()
'OH, my paws and whiskers!'
>>> s.ljust(30)
'OH, my paws and whiskers!     '
>>> s.rjust(30)
'     OH, my paws and whiskers!'
```

## 字符串类型

```py
>>> s.isalnum()
False
>>> s.isalpha()
False
>>> s.isprintable()
True
>>> s.istitle()
False
>>> s.isupper()
False
>>> s.isdecimal()
False
>>> s.isnumeric()
False
```

# 字符串模块属性

这些是用作常量定义的类属性。

| 属性 | 示例 |
| --- | --- |
| `ascii_letters` | `'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'` |
| `ascii_lowercase` | `'abcdefghijklmnopqrstuvwxyz'` |
| `ascii_uppercase` | `'ABCDEFGHIJKLMNOPQRSTUVWXYZ'` |
| `digits` | `'0123456789'` |
| `hexdigits` | `'0123456789abcdefABCDEF'` |
| `octdigits` | `'01234567'` |
| `punctuation` | `'!"#$%&\'()*+,-./:;<=>?@[\\]^_\`{&#124;}~'` |
| `printable` | digits + ascii_letters + punctuation + whitespace |
| `whitespace` | `' \t\n\r\x0b\x0c'` |

# 结尾

切斯特希望表达对你勤奋工作的赞赏。如果你需要他，他正在午休中…

![inp2 ae01](img/inp2_ae01.png)

###### 图 E-1\. 切斯特¹

但露西可以回答任何问题。

![inp2 ae02](img/inp2_ae02.png)

###### 图 E-2\. 露西

¹ 自 图 3-1 以来，他向右移动了大约一英尺。
