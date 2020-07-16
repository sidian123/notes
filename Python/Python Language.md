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

## Strings

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

## Lists

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

  类似list, 但tuple是不可变的

  > 但tuple的元素可以是可变的, 如`([1,2,3],4,5)`

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

  * 对象创建

    以`{}`围绕的集合, 或通过`set()`创建. 但空set必须由`set()`创建, 因为`{}`表示dict

    ```python
    basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
    a = set('abracadabra')
    ```

  * 包含测试

  * 清除重复元素

  * 求并集

    ![Image result for set  union](.Python%20Language/Thu,%2016%20Jul%202020%20160547.png)

  * 求差集

    ![img](.Python%20Language/250px-Venn0010.svg.png)

  * 求对称差分

    ![img](.Python%20Language/220px-Venn0110.svg.png)



## Booleans

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
* 逻辑操作
  * `and` 与
  * `or` 或
  * `not` 非

## tuple dict ??

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
  
* `del`

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

  * 对象字段?

* `in`

  * 在`for`从句中, 依次取集合元素

  * 单独使用, 判断元素是否存在

    ```python
    basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
    'orange' in basket                 # fast membership testing
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







# 其他

* 语句结束判断

  同一缩进的语句处于同一语句块中

* 注释

  ```python
  # 我是注释
  ```








# 参考

* [Python 3.8.4 documentation](https://docs.python.org/3.8/)

* [The Python Tutorial](https://docs.python.org/3.8/tutorial/index.html)

* [Python教程 廖雪峰](https://www.liaoxuefeng.com/wiki/1016959663602400)

# 代办

* [Built-in Types](https://docs.python.org/3.8/library/stdtypes.html#set-types-set-frozenset)