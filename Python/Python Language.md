# Get Start

## 介绍

* 动态类型

* 缩进作为语句开始或结束标志.

  > 相同块中语句缩进对齐即可, 不限制具体缩进长度

* 无变量声明

* 优缺点

  * 优点: 使用简单, 三方库强大
  * 缺点: 运行速度慢

## 安装

* 下载[Python3.8](https://www.python.org/downloads/release/python-383/)
* 添加到环境变量中

## 输入输出

`print()`, `input()`

# 解析器

## 对比

Python语言规范和解析器都是开源的, 因为出现了很多解析器

* CPython

  Python官网下载的Python, 使用的该解析器

* IPython

  IPython是基于CPython之上的一个交互式解释器，也就是说，IPython只是在交互方式上有所增强，但是执行Python代码的功能和CPython是完全一样的。好比很多国产浏览器虽然外观不同，但内核其实都是调用了IE。

  CPython用`>>>`作为提示符，而IPython用`In [序号]:`作为提示符。

* PyPy

  PyPy是另一个Python解释器，它的目标是执行速度。PyPy采用[JIT技术](http://en.wikipedia.org/wiki/Just-in-time_compilation)，对Python代码进行动态编译（注意不是解释），所以可以显著提高Python代码的执行速度。

  绝大部分Python代码都可以在PyPy下运行，但是PyPy和CPython有一些是不同的，这就导致相同的Python代码在两种解释器下执行可能会有不同的结果。如果你的代码要放到PyPy下执行，就需要了解[PyPy和CPython的不同点](http://pypy.readthedocs.org/en/latest/cpython_differences.html)。

* Jython

  Jython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码执行。

* IronPython

  IronPython和Jython类似，只不过IronPython是运行在微软.Net平台上的Python解释器，可以直接把Python代码编译成.Net的字节码。

Python的解释器很多，但使用最广泛的还是CPython。如果要和Java或.Net平台交互，最好的办法不是用Jython或IronPython，而是通过网络调用来交互，确保各程序之间的独立性。

## 命令

* Python交互模式

  进入交互模式

  ```shell
  python
  # 或
  python3
  ```

  退出交互模式

  `exit()`

  > 该模式下, `_`变量可以获取上一条语句执行的结果

* 命令行上执行python语句

  ```shell
  python -c command [arg] ...
  ```

  > `command`表示python语句, 最好以单引号包裹

* 执行python脚本

  ```shell
  python script.py
  ```

* bash调用python, 再执行脚本

  ```shell
  ./script.py
  ```

  脚本必须是可执行的, 且第一行指定执行脚本的命令, 如

  > 其他执行方式, 没有这个约束

  ```python
  #!/usr/bin/env python3
  
  print('hello, world')
  ```

* 执行模块

  ```shell
  python -m module [arg] ...
  ```

## 命令行参数

脚本中可提供`sys`模块的`argv`变量获取命令行参数, 在不同运行方式下, 第一个元素`sys.argv[0]`值不一样

* 运行普通脚本, 值为脚本名
* 运行模块, 值为`-m`
* 运行语句, 值为`-c`
* 无脚本, 值为`''`



# 内置类型

> [Built-in Types](https://docs.python.org/3.8/library/stdtypes.html#set-types-set-frozenset)

* Sequence Type

  There are six **sequence types**: strings, Unicode strings, lists, tuples, buffers, and xrange objects

## Numbers

* 运算

  * 加减乘除 `+`,`-`,`*`,`/`

  * `//` 除运算后, 获取整数部分

  * `%`  取余

  * `**` 做指数运算

  * `=` 赋值

    > 赋值的变量无需声明

* 类型

  * int

    整数字面值类型为`int`, 如`2`,`4`,`10`

  * float

    例子如`5.0`,`1.6`. 

    另外, `/`运算总是返回`float`. 即使两个整数相除

## string

> [str](https://docs.python.org/3.8/library/stdtypes.html#str)

### 声明

* 单引号

  ```python
  'string'
  ```

  > 字符串中允许存在双引号

* 双引号

  ```python
  "string"
  ```

  > 字符串中允许存在单引号

* raw strings

  ```python
  r'string'
  ```

  > string中允许存在特殊字符, 无需转移.

* 多行文本

  `"""..."""` 或 `'''...'''` 如

  ```python
  print("""\
  Usage: thingy [OPTIONS]
       -h                        Display this usage message
       -H hostname               Hostname to connect to
  """)
  ```

  > `\`表示字符串从下一行算起

  

### 转义&&转义字符

`\`用于转义特殊字符, 如`'str\'ing'` , 此时不会冲突

转移字符提供特殊含义, 如`\n`换行.

> 仅在引号围绕的字符串中使用.

### 操作

* `+` 连接字符串

* `*` 重复字符串, 如

  ```shell
  >>> 3 * 'un' + 'ium'
  'unununium'
  ```

  > `un`重复三遍

* 自动连接 (仅对字面值有效)

  多个字符串紧挨着, 就会自动连接, 如

  ```shell
  >>> 'Py' 'thon'
  'Python'
  ```

  ```shell
  >>> text = ('Put several strings within parentheses '
  ...         'to have them joined together.')
  >>> text
  'Put several strings within parentheses to have them joined together.'
  ```

* 获取长度

  `len(str)`

### 索引 & slice 访问

> 字符串是不可变的, 可以访问, 但不可修改某个字符

* 索引访问

  * 下表从0开始

    ```shell
    >>> word = 'Python'
    >>> word[0]  # character in position 0
    'P'
    >>> word[5]  # character in position 5
    'n'
    ```

  * 负索引, 从`-1`开始, 表示最后一个字符

    ```shell
    >>> word[-1]  # last character
    'n'
    >>> word[-2]  # second-last character
    'o'
    >>> word[-6]
    'P'
    ```

* slice访问

  * 使用

    ```
    string[i1:i2] # 从i1(包含)到i2(不包含)截取字符串
    string[:i]    # 从0(包含)到i(不包含)截取字符串
    string[i:]    # 从i(包含)到最后一个元素(包含)截取字符串
    ```

    即, 左边索引缺省为0, 右边索引缺省为字符串长度`len(str)`

  * 正负索引Demo如下:

    ```
     +---+---+---+---+---+---+
     | P | y | t | h | o | n |
     +---+---+---+---+---+---+
     0   1   2   3   4   5   6
    -6  -5  -4  -3  -2  -1
    ```

  * 浅拷贝

    ```
    string[:]
    ```

## list

### 介绍

列表是多个元素的集合, 并且元素类型可以不一致

```
[1, 3, 4, '23', 23, '的士大夫十分']
[1,2,3,[4,5,6]]
```

与字符串相比, list是可变的, 即可以改变list中的元素.

### 访问, 支持索引和切片

使用见[Strings]

### 操作

* 连接两个list, 组成大list

  ```
  [1,2,3]+[4,5,6]
  ```

* 对元素片段替换

  ```
  list[i:i2]=[1,3,4] # 替换
  list[i:i2]=[] # 删除
  ```

* `append(x)` list尾部新增元素. 相当于`a[len(a):]=[x]`

* `extend(iterable)` 新增可遍历对象的所有元素. 相当于`a[len(a):]=iterable`

* `insert()` 指定位置插入元素

* `remove()` 删除指定值的元素

* `pop()` 在指定索引上删除元素

* `clear()` 清空list中所有元素. 相当于`del a[:]`

* `index()` 查找元素, 返回索引

* `count()` 计算某个元素出现的次数

* `sort()` 排序list中的元素.

  > list元素类型不一致时, 无法排序; 有些类型, 本身无排序规则, 如复数`3+4j`

* `reverse()` 倒置list元素

* `copy()` 返回浅拷贝对象. 相当于`a[:]`

* 遍历索引和值

  ```python
  for i, v in enumerate(['tic', 'tac', 'toe']):
      print(i, v)
  ```

> 参考[More on Lists](https://docs.python.org/3.8/tutorial/datastructures.html)

### 复合操作

* 介绍

  即简化由多个`for`, `if`组成, 最终生成list的操作.

* 语法 & 计算规则

  每个`for`, `if`语句都嵌套在上一个`for`, `if`语句中. 最终由第一个表达式得到结果并存入list中. 

  例子如下

  ```python
  >>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
  [(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
  ```

  等于

  ```python
  >>> combs = []
  >>> for x in [1,2,3]:
  ...     for y in [3,1,4]:
  ...         if x != y:
  ...             combs.append((x, y))
  ...
  >>> combs
  [(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
  ```

* 嵌套复合操作

  即第一个表达式也是复合操作, 如

  ```shell
  >>> matrix = [
  ...     [1, 2, 3, 4],
  ...     [5, 6, 7, 8],
  ...     [9, 10, 11, 12],
  ... ]
  >>> [[row[i] for row in matrix] for i in range(4)]
  [[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
  ```

  等于

  ```
  >>> transposed = []
  >>> for i in range(4):
  ...     transposed.append([row[i] for row in matrix])
  ...
  >>> transposed
  [[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
  ```

  又等于

  ```python
  >>> transposed = []
  >>> for i in range(4):
  ...     # the following 3 lines implement the nested listcomp
  ...     transposed_row = []
  ...     for row in matrix:
  ...         transposed_row.append(row[i])
  ...     transposed.append(transposed_row)
  ...
  >>> transposed
  [[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]
  ```

## tuple

* 介绍

  * 类似list, 但tuple是不可变的

    > 但tuple的元素可以是可变的, 如`([1,2,3],4,5)`

  * 貌似不支持复合操作

* 使用

  ```python
  # 完整赋值
  t=(1,2,3,4)
  # 自动pack
  t=1,2,3,4 # (1,2,3,4)
  # 无元素必须加括号
  t=()
  # 一个元素, 可不加括号, 但要加逗号区分
  t=1, # (1,)
  ```

## set

* 介绍

  含无序不重复元素的集合.

* 操作

  * 支持复合操作, 见list

  * 对象创建

    以`{}`围绕的集合, 或通过`set()`创建. 但空set必须由`set()`创建, 因为`{}`表示dict

    ```python
    basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
    a = set('abracadabra')
    ```

  * 包含测试

    ```python
    >>> 'orange' in basket                 # fast membership testing
    True
    ```

  * 清除重复元素

    ```python
    >>> basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
    >>> print(basket)        # show that duplicates have been removed
    {'orange', 'banana', 'pear', 'apple'}
    ```

    

  * 求并集 `a|b`

    ![Image result for set  union](.Python%20Language/Thu,%2016%20Jul%202020%20160547.png)

  * 求差集 `a-b`

    ![img](.Python%20Language/250px-Venn0010.svg.png)

  * 求交集 `a & b`

  * 求对称差分 `a^b`

    ![img](.Python%20Language/220px-Venn0110.svg.png)

## dict

* 介绍

  * 为键值对的集合, 通过键来访问值
  * 键必须为不可变类型的值, 如string, number, 或不含可变类型元素的tuple

* 操作

  * 支持复合操作

    ```python
    >>> {x: x**2 for x in (2, 4, 6)}
    {2: 4, 4: 16, 6: 36}
    ```

  * 创建

    ```python
    a={}
    tel = {'jack': 4098, 'sape': 4139}
    dict([('sape', 4139), ('guido', 4127), ('jack', 4098)]) # {'sape': 4139, 'guido': 4127, 'jack': 4098}
    ```

  * 访问

    ```python
    >>> tel['jack']
    4098
    >>> tel['guido'] = 4127
    ```

  * 删除

    ```python
    del tel['sape']
    ```

  * 获取所有键 `list(d)` ; 获取排序过的键`sorted(d)`

  * 遍历键值

    ```python
    knights = {'gallahad': 'the pure', 'robin': 'the brave'}
    for k, v in knights.items():
        print(k, v)
    ```

## bool

> [bool](https://docs.python.org/3.8/library/functions.html#bool)

* 判断

  * 非0整数为true
  * 非空序列, 如string,list, 为true
  * 两个常量: `True`, `False`

* 比较操作

  * `<` (less than)
  * `>` (greater than)
  * `==` (equal to)
  * `<=` (less than or equal to)
  * `>=` (greater than or equal to)
  * `!=` (not equal to)

* 包含操作

  `in`, `not in` 检查变量是否在sequence中存在

* 逻辑操作

  * `and` 与
  * `or` 或
  * `not` 非

  > 优先级: not > and > or

* 条件可以串联

  如 ` a < b == c` , 等于`a < b and b == c`

* 在添加比较过程中赋值, 必须使用`:=` , 而非`=`

  ```python
  a=0
  while a:=1:
      print(True)
      break
  ```

* 还支持sequence类型对象之间的比较, [略](https://docs.python.org/3.8/tutorial/datastructures.html#comparing-sequences-and-other-types)

## 内置常量

* `None` 常作用函数返回值, 表示值得缺失
* `False` bool的false值
* `True` bool的true值

> 参考[Built-in Constants](https://docs.python.org/2/library/constants.html)

## 操作

* multiple assignment

  > 实质上就是tuple pack和unpack的一个过程

  按位置赋值, 如

  ```python
  a, b = 0, 1
  ```

* `is` , `==`

  判断对象一致性, 基本类型判断值是否相同, 对象判断引用地址是否相同.

* `in`

  * 在`for`从句中, 依次取集合元素

  * 单独使用, 判断元素是否存在

    ```python
    basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
    'orange' in basket                 # fast membership testing
    ```

### del

* 删除list中元素

  ```python
  >>> del a[:]
  >>> a
  []
  ```

* 整个对象

  ```python
  >>> del a
  ```

* 对象字段

  ```python
  del modname.the_answer
  ```

* dict键值

  ```python
  del tel['sape']
  ```

### 解构(unpack) & pack

sequence类型对象都可解构, 如

```python
a,b,c=(1,2,3) # a=1 b=2 c=3
a,b,c=[1,2,3] # a=1 b=2 c=3
a,b,c='123'   # a='1' b='2' c='3'
```

要求: `=`左边有与右边sequence等量的变量, 如:

```python
>>> a,b='123'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: too many values to unpack (expected 2)
```

---------

只有tuple才能pack, 如

```python
>>> a=1,2,3,4
>>> a
(1, 2, 3, 4)
```

### dir()

若无参数, 返回当前作用域下的所有名字; 若提供参数, 返回参数代表的作用域下的所有名字.

# 控制语句

## if

```python
if x < 0:
    x = 0
    print('Negative changed to zero')
elif x == 0:
    print('Zero')
elif x == 1:
    print('Single')
else:
    print('More')
```

`elif`可有0到多个, `else`可选

## for

* 使用

  ```python
  for w in words:
  	print(w, len(w))
  ```

  > `words`可以是任意序列(字符串或list)

* 操作

  支持遍历同时修改元素, 如删除元素

  ```python
  for user, status in users.copy().items():
      if status == 'inactive':
          del users[user]
  ```

* 通过索引遍历

  用到了[range()](https://docs.python.org/3.8/library/stdtypes.html#range)和`len()`方法

  ```python
  a = ['Mary', 'had', 'a', 'little', 'lamb']
  for i in range(len(a)):
      print(i, a[i])
  ```

* [iterable](https://docs.python.org/3.8/glossary.html#term-iterable)

  能够在`for`中遍历的对象. `range()`返回的就是这种类型的对象

## while

```python
while a < 10:
    print(a)
    a, b = b, a+b
```

## break, continue, else

* `break` 打破最近的循环

* `continue` 跳过, 直接继续下一循环

* 循环语句中的`else`

  一般在循环结束后会执行`else`从句, 但由`break`造成的循环结束, 不会导致`else`从句的执行.

  * `for`: 遍历完后执行
  * `white`: 条件变`false`后执行

## pass

不做任何事, 相当于代码的占位符.

```python
while True:
    pass  # Busy-wait for keyboard interrupt (Ctrl+C)

class MyEmptyClass:
    pass

def initlog(*args):
    pass   # Remember to implement this!
```

# 函数

## 介绍

`def`定义函数

```python
def fib(n):    # write Fibonacci series up to n
    """Print a Fibonacci series up to n."""
    a, b = 0, 1
    while a < n:
        print(a, end=' ')
        a, b = b, a+b
    print()

# Now call the function we just defined:
fib(2000)
```

* 函数文档

  函数体的第一行可以为字符串(可选), 作为该函数的文档. 可通过`<funcName>.__doc__`获取

* 参数

  实参按值传递, 对象传的是引用; 实参位于函数体的局部作用域内. 

  普通参数是必须的, 调用时必须提供

* 函数名

  函数本质就是个对象, 函数名就是对象的引用名, 可赋值给其他对象

* 返回值

  每个函数都有返回值, 可通过`return`返回; 若未使用`return`, 或使用但不提供值, 函数都将默认返回`None`

## 参数定义

### 默认参数

```python
def ask_ok(prompt, retries=4, reminder='Please try again!'):
    pass
```

* 默认参数是可选的, 未提供实参时, 使用默认值
* 若默认值是变量, 将在函数声明时通过变量初始化参数默认值

### 按关键字&位置传参

* 介绍

  函数调用时, 可以按照位置传参, 按照关键字传参, 或两种方式混合使用.

* Demo

  如函数定义

  ```python
  def parrot(voltage, state='a stiff', action='voom', type='Norwegian Blue'):
      print("-- This parrot wouldn't", action, end=' ')
      print("if you put", voltage, "volts through it.")
      print("-- Lovely plumage, the", type)
      print("-- It's", state, "!")
  ```

  函数使用

  ```python
  parrot(1000)                                          # 1 positional argument
  parrot(voltage=1000)                                  # 1 keyword argument
  parrot(voltage=1000000, action='VOOOOOM')             # 2 keyword arguments
  parrot(action='VOOOOOM', voltage=1000000)             # 2 keyword arguments
  parrot('a million', 'bereft of life', 'jump')         # 3 positional arguments
  parrot('a thousand', state='pushing up the daisies')  # 1 positional, 1 keyword
  ```

  **注意**, 一个参数不能被传递多次, 两种方式混合使用时尤其需要注意.

* 特殊关键字参数

  * `**name`

    包含所有未匹配函数参数列表的**关键字参数**, 组成的一个dict

  * `*name`

    包含所有按位置未匹配到参数的**位置参数**, 组成的一个tuple

    > `*name`必须出现在`**name`前

  * Demo

    ```python
    def cheeseshop(kind, *arguments, **keywords):
        print("-- Do you have any", kind, "?")
        print("-- I'm sorry, we're all out of", kind)
        for arg in arguments:
            print(arg)
        print("-" * 40)
        for kw in keywords:
            print(kw, ":", keywords[kw])
    ```

    ```python
    cheeseshop("Limburger", "It's very runny, sir.",
               "It's really very, VERY runny, sir.",
               shopkeeper="Michael Palin",
               client="John Cleese",
               sketch="Cheese Shop Sketch")
    ```

    输出

    ```python
    -- Do you have any Limburger ?
    -- I'm sorry, we're all out of Limburger
    It's very runny, sir.
    It's really very, VERY runny, sir.
    ----------------------------------------
    shopkeeper : Michael Palin
    client : John Cleese
    sketch : Cheese Shop Sketch
    ```

### 参数顺序

* 声明时, 参数声明顺序

  * 必需参数, 必须在前面; 接着默认参数; 接着特殊的关键字参数`*name`, `**name`

  * `*name`后也可以接必需或默认参数, 那么这些参数只能按关键字传参了. 如

    ```python
    >>> def concat(*args, sep="/"):
    ...     return sep.join(args)
    ...
    >>> concat("earth", "mars", "venus")
    'earth/mars/venus'
    >>> concat("earth", "mars", "venus", sep=".")
    'earth.mars.venus'
    ```

* 传参时, 实参传递方式

  一般来说, 一个参数, 使用位置或关键字传参, 都是可以的.

  但我们可通过特殊参数约束参数传递方式, 如

  ```
  def f(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2):
        -----------    ----------     ----------
          |             |                  |
          |        Positional or keyword   |
          |                                - Keyword only
           -- Positional only
  ```

  `/`和`*`都是可选的, 且仅作指示和约束作用. 

  * `/`前的参数只能按位传参
  * `/`后`*`前的参数, 都可
  * `*`后的只能按关键字传参

## 其他

### 可变参数

就是上述的`*name`, 若`*name`还存在必需或默认参数, 那只能按关键字传参

```shell
>>> def concat(*args, sep="/"):
...     return sep.join(args)
...
>>> concat("earth", "mars", "venus")
'earth/mars/venus'
>>> concat("earth", "mars", "venus", sep=".")
'earth.mars.venus'
```

### 解构(unpack)

* list或tuple解构, 转化为一个个位置实参

  ```python
  >>> print(*[1,2,3])
  1 2 3
  ```

* dict解构, 转化为一个个关键字实参

  ```python
  >>> def parrot(voltage, state='a stiff', action='voom'):
  ...     print("-- This parrot wouldn't", action, end=' ')
  ...     print("if you put", voltage, "volts through it.", end=' ')
  ...     print("E's", state, "!")
  ...
  >>> d = {"voltage": "four million", "state": "bleedin' demised", "action": "VOOM"}
  >>> parrot(**d)
  -- This parrot wouldn't VOOM if you put four million volts through it. E's bleedin' demised !
  ```

### lambda表达式

> 功能很简陋的lambda表达式

* 是一个表达式, 应该不产生新的作用域

* 函数体是一个表达式, 表达式的值是函数的返回值

* 例子

  ```python
  lambda a, b: a+b
  ```

  > `a`, `b` 是两个参数

### 函数注解

* 介绍

  函数注解实际上, 就是为函数的参数, 返回值提供**类型信息**和**约束**

* 使用

  `<funcName>.__annotations__`可以获取约束信息

  ```python
  def f(ham: str, eggs: str = 'eggs') -> str:
      print("Annotations:", f.__annotations__)
      print("Arguments:", ham, eggs)
      return ham + ' and ' + eggs
  
  f('spam')
  # Annotations: {'ham': <class 'str'>, 'eggs': <class 'str'>, 'return': <class 'str'>}
  # Arguments: spam eggs
  
  f(11111) # 报错, 类型错误
  ```

  参数`:`后的标识符, 定义了参数的类型; 标识符后的`='eggs'`定义参数默认值.

# 作用域

* 并不是所有语句块都产生作用域, 模块, 类和函数才有作用域, 如

  ```python
  if 1:
      x=2 # 在全局作用域内
  print(x) # 2
  ```

* 可见性

  * 局部作用域内可访问外层作用域的变量
  * 访问一个变量, 先从局部向外的方向查找变量

* 挂载

  一些作用域内的变量会挂载到作用域的名字空间中, 如class下的类属性和方法, 模块/包下的变量和函数. 其他的不会有挂载行为.

  挂载后, 可通过`name.name`的形式访问挂载的变量

* [global](https://docs.python.org/3.8/reference/simple_stmts.html#the-global-statement)

  声明标识符位于全局作用域, 也就是在局部作用域内定义全局变量, 如

  ```python
  def test():
      global x # 声明为全局变量
      x=2 # 必须赋值, 否则算作未定义
  
  test()
  print(x) # 2
  ```

  两个**规范** ( 不强制, 但最好遵守 )

  * 相同作用域内, 声明的全局变量不能在`global`之前出现.

  * 声明的全局变量不能与其他全局变量冲突

* [nonlocal](https://docs.python.org/3.8/reference/simple_stmts.html#the-nonlocal-statement)

  声明一个标识符, 绑定到最近外层作用域 (非全局作用域) 的同名变量上.

  两个**约束**:

  * 外层 (非全局) 必须存在对应变量
  * `nonlocal`声明前, 不能存在同名的变量.

* 例子

  ```python
  def scope_test():
      def do_local():
          spam = "local spam"
  
      def do_nonlocal():
          nonlocal spam
          spam = "nonlocal spam"
  
      def do_global():
          global spam
          spam = "global spam"
  
      spam = "test spam"
      do_local()
      print("After local assignment:", spam)
      do_nonlocal()
      print("After nonlocal assignment:", spam)
      do_global()
      print("After global assignment:", spam)
  
  scope_test()
  print("In global scope:", spam)
  ```

  输出结果

  ```python
  After local assignment: test spam
  After nonlocal assignment: nonlocal spam
  After global assignment: nonlocal spam
  In global scope: global spam
  ```

  

# 模块

## 介绍

* 模块就是一个Python脚本. 可以被其他模块或脚本引入.
  * 作为模块引入时, 在模块中, 可通过全局变量`__name__`获取模块名. 模块名为去掉后缀的文件名
  * 直接作为脚本执行时, `__name__`值为`__main__` , 即主入口的意思.
* 模块被引入时, 模块的语句会被执行一次, 且仅一次, 再次引入时, 将不在执行.

* 一般的, 引入模块后, 模块内的全局变量都挂到了模块名下的名字空间中.

## 入门使用

* 定义模块文件`fibo.py`

  ```python
  # Fibonacci numbers module
  
  def fib(n):    # write Fibonacci series up to n
      a, b = 0, 1
      while a < n:
          print(a, end=' ')
          a, b = b, a+b
      print()
  
  def fib2(n):   # return Fibonacci series up to n
      result = []
      a, b = 0, 1
      while a < n:
          result.append(a)
          a, b = b, a+b
      return result
  ```

* 其他模块或脚本中, 引入模块

  ```python
  import fibo
  ```

* 使用, 通过模块名找到模块的全局变量, 如函数

  ```python
  fibo.fib(1000)
  
  fibo.fib2(100)
  
  fibo.__name__
  ```

## import使用

* 仅引入模块名

  ```python
  import fibo
  ```

* 引入模块中的全局变量

  ```python
  from fibo import fib, fib2
  ```

  > 注意, 未引入模块名`fibo`

* 引入模块中全部全局变量, 除了以`_`为前缀命名的变量

  ```python
  from fibo import *
  ```

* 引入的同时修改名字, 原名将不可用.

  ```python
  import fibo as fib
  from fibo import fib as fibonacci
  ```

## 使用策略

让一个脚本既可以作为模块供人使用, 也可作为主入口文件直接执行, 可添加`__name__`的条件判断, 如

```python
if __name__ == "__main__":
    import sys
    fib(int(sys.argv[1]))
```

## 模块搜索顺序

引入模块时, 只需提供模块名, python会自动寻找模块位置.

寻找顺序如下:

1. 查看内置模块
2. 查找`sys.path`中指定的目录列表. 初始化时, `sys.path`会被赋值为以下目录
   1. 正在执行的脚本所在的目录
   2. 环境变量`PYTHONPATH`指定的目录, 它是一组目录的集合, 类似`PATH`
   3. Python安装目录下的目录, 如`DLLs`, `lib`, `current`, `sit-packages`等

注意点

* 仅搜索目录下的直接文件, 不会深度搜索.

* 指向目录的符号链接不会被搜索

## 包(package)

### 介绍

* 模块(module)是一个名字空间, 防止全局变量间名字冲突. 而包(package)是模块的名字空间, 防止模块间名字冲突.

* 包的名字空间除了模块外, 也可以有自己的全局变量, 函数等, 需要在`__init__.py`文件中给出.

### 包声明

当目录下存在`__init__.py`文件时, 表示该目录代表包.

例子如下, 由很多包组成

```
sound/                          Top-level package
      __init__.py               Initialize the sound package
      formats/                  Subpackage for file format conversions
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  Subpackage for filters
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...
```

### `__init__.py`

* 该文件为包声明的标志, 文件内容可为空, 或含有一些包初始化代码

* 与模块一样, 第一次访问包或包下内容时, 将执行对应包的`__init__.py`代码, 且仅执行一次, 之后的访问不再执行.

### 包引入

* 简单引入模块

  ```python
  import sound.effects.echo
  ```

  使用模块时, 必需给出全名

  ```python
  sound.effects.echo.echofilter(input, output, delay=0.7, atten=4)
  ```

* (推荐) 无需全名使用的引用方式

  ```python
  from sound.effects import echo
  ```

  ```python
  echo.echofilter(input, output, delay=0.7, atten=4)
  ```

* 引入模块中变量

  ```python
  from sound.effects.echo import echofilter
  ```

  ```python
  echofilter(input, output, delay=0.7, atten=4)
  ```

* 修改引入模块名

  ```python
  import sound.effects.echo as echo
  ```

### 注意点

* `from package import item` 时的查找循序

  先在`package`包的`__init__.py`下查找, 未找到, 则尝试去加载包下模块.

* `import item.subitem.subsubitem`时

  * 路径中, 除了最后一项`subsubitem`, 都必须是包
  * 最后一项`subsubitem`可以是模块或包, 但不能是其他的, 如函数, 类, 变量等.

* `from sound.effects import *`

  会将`sound.effects`包下的所有模块引入吗? 不会, 默认仅引入包自身的内容(`__init__.py`) , 其他模块需要在`__init__.py`中显示声明, 如

  ```python
  __all__ = ["echo", "surround", "reverse"]
  ```

  > 将额外引入`sound.effects`包下的直接模块`echo`, `surround`, `reverse`

  除此之外, `__init__.py`中引入的包, 也会被引入, 即使无`__all__`声明.

### 包间引入

不同包下的模块要相互引用, 有两种方式:

* 绝对引用

  以入口模块所在的目录为根路径, 写全模块名, 如

  ```python
  from sound.effects import echo
  ```

  > 注意, `sound`包应与入口模块同级

* 相对引用

  以`.`表示使用相对引用. 一个点`.`相对于当前包, 两个点`..`相对于上级包, 三个点`...`相对于上上级包, 以此类推.

  例子:

  ```python
  from .moduleY import spam # 引入当前包下的moduleY模块的spam变量
  from . import moduleY # 引入当前包下的moduleY模块
  from .. subpackage2.moduleZ import eggs # 引入上级包subpackage2下模块moduleZ的变量eggs
  ```

  > 注意, 包名寻找, 实际上用到了`__name__`变量, 而入口模块的`__name__`一直为`__main__`, 因此入口模块只能使用绝对引用.

# 类

## 介绍&原理

* 类是一个特殊的函数, 类的定义被执行后才生效.

* 执行类时, 创建了一层作用域, 且类定义执行的过程中产生的变量将被挂载到类名上, 如

  ```python
  class Test:
      i=3
      def hello():
          print('hello:',Test.i)
  
  Test.hello() # hello: 3
  print(Test.i) # 3
  ```

* 对象就是个键值对, 与dict类型, 但对象是通过`.`访问属性和方法的; 对象可动态新增属性, 动态删除属性, 如

  ```python
  class Test:
      pass
  
  x = Test()
  x.a=23
  print(x.a)
  del x.a
  print(x.a) # 错误, 属性已经被删除了
  ```

  > 有的对象不允许实例化后修改属性, 不知道怎么设置的

* 类可以实例化对象, 实例化过程如下:

  * 创建空对象

  * 将类的所有属性赋值到对象中, 方法会被wrapper, 第一个参数传入对象自身.

    > 官方教程说的与我这里描述的不一致, 但如果按照官方教程的解释, 我将无法理解下面的Demo例子

  * 修改对象的一些特殊属性, 如`__class__`指向实例化的对象

  * 若类中存在`__init__`方法, 则执行它.

* Demo

  ```python
  class Test:
      """我是类的文档"""
      i=23 # 类属性, 但对象实例时会赋值给对象, 可在实例化过程中被覆盖
      ii=11
  
      def __init__(self):
          self.ii=22 # 覆盖从类继承类的ii字段
          self.aa=222 # 新增字段
  
      def mehtod():
          print(Test.i)
      def method2(self):
          print(self.i)
  
  obj= Test()
  print(obj.ii) # 22 实例化时值被覆盖了
  obj.i=999
  obj.method2() # 999
  print(Test.i) # 23 对象的修改一般不会影响类的属性
  Test.method() # 23
  obj.mehtod() # 错误, 对象的方法被wrapper了, 多传了个参数(对象本身), 因此报错: TypeError: mehtod() takes 0 positional arguments but 1 was given
  func=obj.method2
  func() # 对象的方法是包裹过的, 赋值给其他变量, 方法的第一个参数也指向对象本身.
  ```

  > `self`表示实例化的对象, 参数名可以换成其他的, 无所谓.

## 继承

* 继承过程

  假设A继承B,C, B继承D.

  1. 创建空对象

  2. 根据继承链, 从左到右, 深度优先的顺序, 获取到遍历链, 反过来得到的链(C, D, B) 依次将该类的所有属性赋值给空对象

     > 方法会wrapper, 同名的会被后赋值类覆盖.

  3. **仅**执行派生类A的`__init__`方法.

  > 还是一样, 我的解析与官方文档不一致... 但这种描述能解释大部分语法现象...

* MRO ( Method Resolution Order)

  方法解析顺序, 上述提到了, 从左到右, 深度优先遍历得到的顺序. 如, (B, D, C)

* `super(type,object)`

  返回一个代理对象, 代理方法调用到父类或兄弟类的**方法**上. 说白了, 就是在实例化过程中, 调用父类方法的.

  首先得到MRO顺序, 如上所示. 给定`type`, 那么返回MRO顺序中`type`后面的那个类型. 如`type`为D, 那么`super()`返回C的代理对象.

  `object`只要是`type`的直接或间接实例即可, 在实例化的环境下, 多半为`self`

  若不传参数, `super()`, 默认`type`为当前类, `object`为`self`, 那么在上个例子中, 将返回`B`.

* Demo

  ```python
  class D:
      def method(self):
          print('d')
          super().method()
  
  class A(D):
      a=23
      def method(self):
          print('a')
          super().method()
  
      def __init__(self,width):
          print('a initial')
  
  class B:
      a=11
      b=33
      def method(self):
          print('b')
        
  class C(A,B):
      b=55
      d=66
      def method(self):
          print('c')
          super(A,self).method()
  
      def __init__(self):
          print(self.a)
          super().__init__(32)
          print('c initial')
  
  obj=C()
  obj.a=obj.a+1
  print(obj.a)
  obj.method()
  obj2=C()
  print(obj2.a)
  ```

  ```
  23
  a initial
  c initial
  24
  c
  d
  b
  23
  a initial
  c initial
  23
  ```

  

## 进阶

### 私有变量

Python中没有私有变量的强制约束, 但是有类似的方案:

1. 以`_`为前缀, 如`_name`

   不是强制的, 需要用户间约定和遵守

2. 以`__`为前缀, 如`__name`

   会被Python转化为`_classname_name`的形式, 保证类被其他类继承后, 不会冲突.

### Iterator

* 介绍

  可遍历类型的实例可以在`for`语句中使用. 

* 原理

  * `for`语句调用容器对象的`__iter__()`方法, 获取迭代器

    > `iter()`会调用该方法

  * 调用迭代器的`__next__()`方法, 获取容器内的一个元素

    > `next()`会调用该方法

  > 因此仅需实现两个方法, 便可在`for`中使用; `for`内部就是调用了`iter()`, `next()`两个方法

* Demo

  ```python
  class Reverse:
      """Iterator for looping over a sequence backwards."""
      def __init__(self, data):
          self.data = data
          self.index = len(data)
  
      def __iter__(self):
          return self
  
      def __next__(self):
          if self.index == 0:
              raise StopIteration
          self.index = self.index - 1
          return self.data[self.index]
  ```

    ```
  >>> rev = Reverse('spam')
  >>> iter(rev)
  <__main__.Reverse object at 0x00A1DB50>
  >>> for char in rev:
  ...     print(char)
  ...
  m
  a
  p
  s
    ```

# 错误&异常

## 异常类型

* 语法错误 syntax errors

  解析时的语法错误

* 异常 exceptions

  运行时的抛出的异常, 可以被捕获

## 异常捕获基础

* 介绍
  * 无异常, `catch`从句不执行.
  * 有异常, 未被捕获, 异常往上抛
  * 有异常, 被捕获, 执行对应`catch`从句, `try`语句结束

* 基本Demo

  ```python
  while True:
      try:
          x = int(input("Please enter a number: "))
          break
      except ValueError:
          print("Oops!  That was no valid number.  Try again...")
  ```

  ```python
  except (RuntimeError, TypeError, NameError):
      pass
  ```

## 捕获进阶

* 继承`Exception`的类可作为异常抛出

* `except`上的基类, 能捕获子类, 如

  ```
  class B(Exception):
      pass
  
  class C(B):
      pass
  
  class D(C):
      pass
  
  for cls in [B, C, D]:
      try:
          raise cls()
      except B:
          print("B")
      except D:
          print("D")
      except C:
          print("C")
  ```

  一直输出B,B,B

  -----------------

* `except`可以有多个, 一个`except`可捕获多个异常

* 最后一个`except`可不声明异常, 作用通配符

  ```python
  import sys
  
  try:
      f = open('myfile.txt')
      s = f.readline()
      i = int(s.strip())
  except OSError as err:
      print("OS error: {0}".format(err))
  except ValueError:
      print("Could not convert data to an integer.")
  except:
      print("Unexpected error:", sys.exc_info()[0])
      raise # 重新抛出该异常
  ```

  -------

* `else`从句

  仅当`try`语句中未抛出异常时执行

* 获取捕获的异常实例

  ```python
  try:
      raise Exception('spam', 'eggs')
  except Exception as inst:
      print(type(inst))    # the exception instance
      print(inst.args)     # arguments stored in .args
      print(inst)          # __str__ allows args to be printed directly,
                           # but may be overridden in exception subclasses
      x, y = inst.args     # unpack args
      print('x =', x)
      print('y =', y)
  ```

  实例化异常时的参数可通过

* `finally`从句

  * 无论`try`语句是否抛出异常, 都执行

  * 若`try`语句中有跳转语句, `break`, `continue`, `return`, 在跳转前会去执行`finally`从句
  * 若`finally`和`try`中都有`return`, 那么真正的返回值来之`finally`中的`return`

## 抛出异常

* 捕获的`except`从句中, `raise`可再次抛出被捕获的异常.

  ```python
  try:
      raise NameError('HiThere')
  except NameError:
      print('An exception flew by!')
      raise
  ```

* `raise`后接异常类的实例或异常类 (直接或间接继承`Exception`的类) . 若接异常类, `raise`会隐式调用该类的无参构建函数来生成实例

  ```python
  raise ValueError  # shorthand for 'raise ValueError()'
  ```

## 自动清理资源

类似Java的try-with-resources

```python
with open("myfile.txt") as f:
    for line in f:
        print(line, end="")
```

当资源`f`退出`with`语句块后, 无论正常或异常退出, 都会关闭资源`f`.

> 这种对象的类型必须实现一些特殊方法, 详细见[with](https://docs.python.org/3/reference/compound_stmts.html#with)

# 依赖&管理

## pip

> 若`pip`命令存在`PATH`, 可以去掉前缀`python -m`

* 介绍

  pip是python安装三方模块的工具, 或内置模块

* 安装

    * 安装最新版模块

      ```shell
      python -m pip install SomePackage
      ```

    * 指定版本安装

      ```shell
      python -m pip install SomePackage==1.0.4    # == 指定特定版本
      python -m pip install "SomePackage>=1.0.4"  # >= 指定安装版本的下线
      ```

      > 若已存在不同版本的包, 会被删掉; 若已存在, 则无任何动作.

    * 更新模块到最新

      ```shell
      python -m pip install --upgrade SomePackage
      ```

* 查看

  * 查看某个模块详细信息

    ```shell
    python -m pip show requests
    ```

  * 列出所有已安装模块

    ```shell
    python -m pip list
    ```

* 搜索

  ```shell
  python -m pip search astronomy
  ```

* 保存项目依赖? 见*虚拟环境*

* 模块作用范围

  安装的模块默认全用户可用, 若仅为当前用户安装模块, 可加上选项`--user`

  若为某个项目单独安装模块呢? 需要用到**虚拟环境**, 见下

## 虚拟环境

* 原理

  上述提到过, 引来的包, 会在执行脚本当前目录, `PYTHONPATH`环境变量指定的目录和Python安装目录下寻找模块.

  pip安装的第三方模块一般会被安装到Python家目录下.

  内置模块`venv`创建虚拟环境时, 创建一个轻量级的Python"家目录", 然后执行该目录下的特定脚本`activate`, 来修改环境变量. 此时命令`python`, `pip`指向该目录下的命名的. `pip`安装模块, 也会安装到该目录下.

* 创建虚拟环境

  如, 创建python3的虚拟环境

  ```shell
  python3 -m venv tutorial-env
  ```

  > `tutorial-env`是虚拟环境名

* 进入虚拟环境

  执行脚本`activate`即可, 它会修改环境变量, 将虚拟环境作用Python家目录

  ```shell
  source Scripts/activate
  ```

* 保存项目依赖

  * 导出依赖

    ```shell
    pip freeze > requirements.txt
    ```

  * 安装依赖

    ```shell
    pip install -r requirements.txt
    ```

  > 最好在虚拟环境下操作

# 其他

* 语句结束判断

  同一缩进的语句处于同一语句块中

* 创建空对象

  ```python
  obj=object()
  ```


## 编译缓存

模块加载时会编译... 然后编译的结果会缓存起来, 放在对应模块同级的目录`__pycache__`下.	

> 详细参考[“Compiled” Python files](https://docs.python.org/3.8/tutorial/modules.html#compiled-python-files)


## 注释

```python
# 我是注释
```

## 生成器(Generator)

* 介绍

  * 生成器就是个普通方法, 除了有`yield`表达式. 
  * 或者说, 生成器是特殊的迭代器 (iterator), 会自动生成`__iter__()`, `__next__()`方法
  * 每次执行`next()`时, 执行顺序从方法头或上次终止处(`yield`)开始, 直到碰到`yield`, 返回结果后终止执行

* 使用Demo

  生成器定义

  ```python
  def reverse(data):
      for index in range(len(data)-1, -1, -1):
          yield data[index]
  ```

  在`for`中使用

  ```python
  for char in reverse('golf'):
      print(char)
  ```

  或者分步进行

  ```python
  def reverse(data):
      for index in range(len(data)-1, -1, -1):
          yield data[index]
  
  iter=reverse('abc') # 获取迭代器
  print(next(iter)) # c 第一次迭代
  print(next(iter)) # b 第二次迭代
  print(next(iter)) # a 第三次迭代
  ```

* 生成表达式

  语法类似list复合操作, 不过该表达式得到的结果是迭代器, 且以`()`为标志

  ```python
  (i*i for i in range(10)) # 生成平方的迭代器
  sum(i*i for i in range(10)) # sum传入迭代器, 计算平方和
  ```



# 参考

* [Python 3.8.4 documentation](https://docs.python.org/3.8/) 所有文档
* [The Python Tutorial](https://docs.python.org/3.8/tutorial/index.html) 教程
* [The Python Standard Library](https://docs.python.org/3/library/index.html) 标准库
* [Python教程 廖雪峰](https://www.liaoxuefeng.com/wiki/1016959663602400)

# 代办

* [Built-in Types](https://docs.python.org/3.8/library/stdtypes.html#set-types-set-frozenset)
