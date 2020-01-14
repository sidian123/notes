# 一 介绍

bash shell是一个与sh兼容的命令行解析器，是用户与系统内核交互的接口。通过bash，可以执行命令（程序），bash本身就内置了很多常用的命令。可以将一些常用的命令写入脚本中，让bash运行，用以实现简化和自动化日常的任务。对于其他比较复杂的任务，如字符串操作、算术、数据访问、更强的函数等功能，尽管bash也提供了对应的语法（也有函数！！），但还是应该选择使用其他脚本（python）或编译语言（c）。因此，这里只记录比较常用的bash语法。

# 二 启动过程

shel按照启动方式的不同，可分为`login`、`non-login`、`interactive`、`non-interactive` shell。`login shell`表示该shell用来登录；`interactive shell`的标准输入输出都连接到终端上，即可与用户交互。

shell在启动时会读取配置文件，不同的shell读取的文件都不同。下面是bash shell在不同启动方式下，与执行的配置文件的关系：

* **login、interactive bash**：即命令行登录时用的shell，先读取和执行`/etc/profile`，然后查找`~/.bash_profile`，`~/.bash_login`和`~/.pro‐file`，读取和执行**第一个**找到的文件。当shell结束时，会读取和执行`~/.bash_logout`和`/etc/bash.bash_logout`。
* **non-login、interactive bash**：即桌面环境下打开的虚拟终端使用的shell，会读取和执行`~/.bashrc`
* **non-login、non-interactive bash**：即执行脚本用的bash，一般情况下不会读取文件。

>图形界面登录使用就不是bash了，而是不知道哪款GUI shell了

建议自己的配置一般放入`~/.bashrc`中

配置文件中会定义一些和用户使用相关的环境变量和shell变量，如PATH、命令提示符（prompt）、命令别名（alias）、权限掩码、默认编辑器等配置。

# 三 语法

## 变量

变量无数据类型, 因此可以存入各种类型的数据, 如数字, 字符, 字符串, 数组等.

* 环境变量

* 全局变量

* 局部变量

  局部变量可以隐藏全局变量.

  定义

  ```shell
  local variableReference=value
  ```

  例子

  ```shell
  #!/bin/bash
   
  # bash variable
  SHELL="Unix"
   
  function bashShell {
      # bash local variable
      local SHELL="Bash"
      echo $SHELL
  }
   
  echo $SHELL
  bashShell
  echo $SHELL
  ```

  输出

  ```
  Unix
  Bash
  Unix
  ```

## 简单命令

一条简单的bash命令，以要执行的**命令**开始，接着给出空格分隔的**参数**，最后以**控制操作符**结束，执行结束后，会返回一个**状态码**。

> 执行一条命令类似于其他语言中调用一个函数。

>不要将状态码和命令的标准输出混淆，状态码可以通过`$?`获得，bash中很多复合命令可以检查到命令放回的状态码而执行响应的动作。

>一般状态码为0表示命令正常，其余非正常，但命令状态码的最终解释权归该命令所有。

其中, 控制操作符有`&`，`;`，`换行`

* 以`&`结束的命令语句，会在后台运行，bash不会等待命令的完成，而是直接执行下一条语句；
* 以`;`或`换行`结束的命令语句，bash会依次按顺序执行, 并等待其完成。

多条由 *管道*、*控制操作数*、`&&`、`||` 等符号隔开的命令序列称为 **list**。执行list后，会返回list中最后一条被执行的命令的状态码。

## 复合命令

复合命令是bash提供的更为复杂的命令。<span style="color:red">书写命令时要注意符号间的空格</span>

### 常用复合命令

* `(list)`  list会在子Shell环境中运行，因此list中的命令就算修改了环境变量，也不会影响当前shell的环境变量。

  >于是, 为子Shell设置环境变量而不影响当前Shell的方式如下
  >
  >```shell
  >(PATH=/usr/confusing:$PATH;echo $PATh)
  >```

* `{ list; }` 仅用于代码块分割, 与其他语言的`{}`作用一致, 将返回list最后一条命令的状态码. 

* `((expression))` ：根据**算法表达式**expression计算的值返回对应的状态码。如果结果值不为零，状态码为0，否则返回1。

* `[[ expression ]]` ：根据**条件表达式**expression计算的值返回对应的状态码。如果表达式true，返回0，否则1。

  > 与`[ expression ]`功能基本一致

### 逻辑运算符组合

通过逻辑运算符组成更复杂的表达式

* `( expression )` :提升表达式执行的优先级
* `! expression` :取反表达式的状态码
* `expression1 && expression2` ：两个表达式为true（状态码为0），才true。expression2不被执行，当且仅当expression1返回0（true）。
* `expression1 || expression2`：任意一个表达式为true，则true。expression2被执行，当且仅当expression1返回1（false）

### 流程控制

* `for name  in  word ... ;  do list ; done`
  name变量每次从in后取得一个值，然后运行一次list。word可使用通配符，自动被expansion（扩展）。

  ```bash
  for var in "$@"
  do
      echo "$var"
  done
  ```
  
  > 打印所有参数.
  
* `for (( expr1 ; expr2 ; expr3 )) ; do list ; done`
  首先执行expr1，之后每次都检测expr2是否为0，如果为0则结束；否则执行list和expr3被执行，然后再重复检测。

* `if list; then list; [ elif list; then list; ] ... [ else list; ] fi`
  如果list（不是表达式了）返回0，则执行then后的list，否则检查elif后的list，如果状态码都不为0，则执行else后的list。

* case

  ```bash
   case word in [ [(] pattern [ | pattern ] ... ) list ;; ] ... esac
  ```

  `word`和`pattern`都可使用通配符

  ```bash
  case $1 in
  	-i|i|install )
          	install_ss $2 $3
  		;;
          -bbr )
          	install_bbr
                  ;;
          -ssr )
          	install_ssr
                  ;;
  	-uninstall )
  		uninstall_ss
  		;;
          -sslink )
                  get_ss_link
                  ;;
  	* )
  		usage
  		;;
  esac
  ```

### 函数

* 介绍

  * 无作用域之说, 仅为命令分组
  * 函数使用与命令一致, 可传入参数
  * 函数状态码为最后一条命令的状态码, 也可通过`return <int>` 手动结束函数和设置函数状态码
  * 函数的输出为所有命令的输出

* 有两种定义形式

  ```shell
  name () compound-command [redirection]
  或
  function name [()] compound-command [redirection]
  ```

  > `redirection`在函数执行后会被执行, 详细见手册
  >
  > `compound-command`中可使用`return <int>`

* 使用

  * 可仅给出函数名,如`name`
  * 可传入参数, 如`name a b c`

* 例子

  ```shell
  #!/bin/sh
  function fun () {
    echo $1
    echo $2
    echo $3
  
    pid=1234
    echo $pid
    echo 'hello world'
  }
  
  fun a b c
  echo $pid
  echo $(fun c d e)
  echo $(fun)
  ```

  输出

  ```
  a
  b
  c
  1234
  hello world
  1234
  c d e 1234 hello world
  1234 hello world
  ```

> 其他内容, 请查看`man bash`手册。

> 复杂逻辑还是建议使用通用语言编写.

## 字面值

shell只有一种类型，字符串。shell在执行命令之前，会查找变量、通配符、其他替换符号，然后替换，最终执行命令并传入处理过的参数。通过引号可以阻止这种行为：

* **无引号**：shell会执行替换过程。多个参数空格分隔，不能出现单、双引号，可以用`\'`  `\"`替代。

* **单引号**：引号内的所有字符不被替换且视为一个参数。但 `'` 不能出现在里面

* **双引号**：与单引号类似，但里面的变量可被替换。 `"` 不能出现在里面

  > 注意, 仅变量可替换

## 特殊变量

shell变量用于存储字面值、参与计算。bash为我们提供了一些特殊变量，用以获取相关的值，如当前shell的参数、进程号等等，但这些特殊变量不能被改变：

* 命令相关

  * `$0` 执行脚本时的路径名 ( 一般是相对路径 )

  * `${0##*/}` 脚本的base name

    > 实际上是路径名以`/`分隔, 得到后部分

  * `${0%/*}` 脚本的目录

    > 实际上将路径名以`/`分隔, 得到前部分

  > `${0%/*}`在某种情况下会出现问题的, 如`bash test.sh`, 将无法得到脚本目录, 如下给出解决方案:
  >
  > ```bash
  > if [ "${0%/*}" != "${0##*/}" ]
  > then                          
  >     echo "${0%/*}"        
  > else                          
  >     echo "."  
  > fi                            
  > ```

* 参数相关

  * `$1,$2,...` 脚本参数, 第一个参数`$1`, 第二个`$2` , ...

    > `shift`可以删除第一个参数，其他参数前进补齐，即`$2`移入`$1`，`$3`移入`$2`，...

  * `$#` 参数个数, 无则0

  * `$@`所有参数组成的数组

  * `$*`所有参数组成的字符串

* 其他

  * `$$`当前shell的进程号

  * `$?`上一条命令的exit code。

    > 在脚本中，exit code一般为最后一条命令的exit code。也可以通过命令`exit num`，手动返回exit code，并结束当前shell。如果num不给出，默认上一条命令的exit code

> 参考[How do I know the script file name in a Bash script?](https://stackoverflow.com/questions/192319/how-do-i-know-the-script-file-name-in-a-bash-script)

## 表达式

### 算术表达式

3.2小节提到了`((expression))`可以计算算术表达式，下面给出所有的操作符：

* `id++` `id--` 
  variable post-increment and post-decrement
* `++id` `--id`
      variable pre-increment and pre-decrement
* `-` `+`    unary minus and plus
* `!` `~`    logical and bitwise negation
* `**`     exponentiation
* `*` `/` `%`  multiplication, division, remainder
* `+` `-`    addition, subtraction
* `<<` `>>`  left and right bitwise shifts
* `<=` `>=` comparison
* `==` `!=`  equality and inequality
* `&`      bitwise AND
* `^`      bitwise exclusive OR
* `|`      bitwise OR
* `&&`     logical AND
* `||`     logical OR
* `expr?expr:expr`
      conditional operator
* `=` `*=` `/=` `%=` `+=` `-=` `<<=` `>>=` `&=` `^=` `|=`
      assignment
* `expr1 , expr2`  comma

### 条件表达式

复合命令 `[[ ]]` 与bash内置`[ ]`命令作用基本一致，可以进行文件测试、字符串测试、算术测试。部分功能与8.3.5小节重叠，具体参数参考`man test`

常用的如下:

* `-n <string>` string长度不为0
* `-z <string>` string长度为0
* `-f <file>` 文件存在且为普通文件
* `-d <file>` 文件存在且为目录
* `<string> = <string>` 字符串相等
* `<string> = <string>` 字符串不等

## 替换Expansion

Bash提供了很多种替换, 这里仅介绍最常用的

### 命令替换

`$(COMMAND)`允许**命令**的**标准输出**替换为**命令行参数**。

如下面例子所示，`echo 23`的输出`23`替换了`$(echo 23)`

```bash
[root@sidian Desktop]# echo 23
23
[root@sidian Desktop]# a=$(echo 23)
[root@sidian Desktop]# echo $a
23
```

### 算术替换

与`((expression))`不同, `$((expression))`将**算数表达式的值**替换为**命令行参数**

```bash
[root@sidian Desktop]# a=$((23+3))
[root@sidian Desktop]# echo $a
26
```

也可不用算术替换，只用算术表达式

```bash
[root@sidian Desktop]# ((b=23+33))
[root@sidian Desktop]# echo $b
56
```

# 四 脚本

## 介绍

* 脚本就是Bash命令的集合。

## 运行方式

* 赋予可执行权限后直接运行

  ```shell
  chmod a+x script.sh
  ./script.sh
  ```

  > 应明确给出脚本路径，否则将从PATH路径下查找

  具体使用解析器执行脚本需在第一行指定, 如

  ```shell
  #!/usr/bin/env bash
  ```

  ```shell
  #!/bin/bash
  ```

  第二种不推荐, 因为不同环境下解析器存放位置不同; 而`env`命令会从`PATH`路径下寻找解析器, 如`bash`

* 启动新`bash`执行

  ```shell
  bash script.sh
  ```

> 上述两种都是开启了新的Bash Shell来执行脚本

----

* 在当前bash中执行

  ```shell
  . script.sh
  ```

  或

  ```shell
  source script.sh
  ```

# 五 其他

## 成为孤儿进程

> 原理见[linux core]

### 结束父进程

```bash
(command & exit)
```

启动了一个子Shell, 子Shell中后台启动了命令command, 接着子Shell结束, 导致command变成孤儿进程. 如:

```bash
(sleep 30 & exit)
```

> 参考[How to prevent a background process from being stopped after closing SSH client in Linux](https://stackoverflow.com/a/38165810/12574399)

### nohup

该命令仅运行指定的命令, 需手动后台化, 如

```bash
nohup COMMAND &
```

* 原理

  与上述方案一样, 运行父进程nohup, nohup运行指定命令, Bash结束后, nohup被杀死, 但nohup运行的命令未死, 成了孤儿进程.

* 使用

  nohup默认将准输入流重定向到`/dev/null`，标准输出流重定向到nohup.out，标准错误流重定向到标准输出。若不满足使用, 需自己需改重定向.

### exec

> 貌似替换了Bash进程后, 其父进程变成了init, 直接成为孤儿进程. 注意, 需要后台化.

Bash内置命令, 用于修改或替换当前Shell进程.

```shell
exec [-c] [-l] [-a name] [command [arguments ...]] [redirection ...]
```

* `command` 要替换的命令
* `arguments` 命令执行的参数
* `redirection` 重定向
* `-a name` 修改`$0`
* `-c` 在空环境变量下执行
* `-l` 插入dash符号, 主要用于替换Login Shell时提示用户的

---

一种特殊的使用情况

```shell
exec > output.txt
```

即仅修改当前Shell的重定向, 

> 参考[Bash exec builtin command](https://www.computerhope.com/unix/bash/exec.htm)

## 重定向

* 脚本自己重定向自己, 见[How do I redirect the output of an entire shell script within the script itself?](https://stackoverflow.com/questions/314675/how-do-i-redirect-the-output-of-an-entire-shell-script-within-the-script-itself)

## 转义(坑之所在)

###  坑死人的规则

凡是用到字符串的地方, 都要被转义一次

### 例子

> 若不想看例子, 请直接看总结

在WSL中, 若要跳转到Windows中某个目录中, 如

```shell
cd /mnt/c/Program\ Files/Typora/
```

这里有个`\` 用于转义空格, 然后整个路径被当作一个字符串

----

现在先定义变量后再使用

```shell
export typora_home="/mnt/c/Program Files/Typora"
```

然后跳转

```shell
cd $typora_home
```

将报错

```shell
-bash: cd: too many arguments
```

因为有了空格, 被当作多个参数

----

那么修改字符串

```shell
export typora_home="/mnt/c/Program\ Files/Typora"
```

然后跳转

```shell
cd $typora_home
```

将报同样错误. 

来分析下

1. 赋值变量时, 根据规则, 先判断转义, 但在双引号中不构成转义关系.

   > `\"`才会构成转义关系

2. `cd`时, 根据规则, 需判断转义; 然而, 上一步转义中, `\`不构成转义关系, 因此本身被转义了, 不再参与转义过程, 导致之后的空格被Bash解析为两个字符串, 因此跳转失败.

-----

那么解决办法: 修改字符串如下所示

```shell
export typora_home="/mnt/c/Program Files/Typora"
```

然后跳转

```shell
cd "$typora_home"
```

### 总结

* 任何用到字符串的地方都会被跳转
* 未参与转义的`\`本身将被转义, 即下次不参与转义过程了
* 最好使用`"`, 保证字符串不被分割
* 字符串中还要注意**替换语法**

## 疑问

* 多次见人使用`exec`, 这个有何用?

  ```shell
  exec zsh
  source .zshrc
  ```

# 待学习 

* https://www.tutorialspoint.com/unix/index.htm
* https://www.tutorialkart.com/bash-shell-scripting/bash-variable/ ( 感觉这个比较好 )