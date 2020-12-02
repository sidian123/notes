# 介绍

类Unix系统的简化历史如下:
![在这里插入图片描述](../.Linux%25E5%2585%25A5%25E9%2597%25A8/20181216164434675.png)

这里主要介绍和Unix相关的命令和工具，因此所有unix-like的操作系统都可以使用这些命令，如linux、BSD和Solaris等等。

# Shell

shell是一个用于访问操作系统服务的用户接口，分为两类：CLI shell（command-line interface）和GUI shell（graphical user interface）。由于CLI shell功能的强大，即使在基于GUI的操作系统中也会提供CLI shell。

不过shell只是特殊的应用程序而已，容易被替代。最初的shell为Bourne shell，之后出现了很多基于Bourne shell的shell。而linux默认使用Bourne shell的增强版本：**bash shell**。

在登录后，会打开一个shell window，如在Gnome的GUI中，打开一个terminal应用，它会在新窗口中启动一个shell。shell启动后会弹出类似`name@host:path$`的命令行提示符。其中`#`表示特权用户，`$`表示普通用户。

bash有很多特性，比如自动补全、别名、通配符、流的重定向、脚本语言等等。

## 通配符

shell在执行命令时，会有一个globbing-->expansion的过程：

1. **globbing**：含有通配符的参数匹配文件或目录名。如果没有匹配，则通配符视作字面值，转3。
2. **expansion**：匹配的多个名字以空格分隔的方式替换原有的参数（含有通配符）
3. 执行该命名。

![在这里插入图片描述](../.Linux%25E5%2585%25A5%25E9%2597%25A8/20181216225352592.png)

通过单引号将参数括起来，将通配符当作普通字符。

通配符：

* `*`：匹配0至多个的任意字符
* `?`：匹配一个任意字符
* 等等。。。

例子（下面说的文件名包含目录名，**并且文件名也即路径**）：

>* `at*`匹配当前目录中以at开始的文件名
>* `*at`匹配当前目录中以at结尾的文件名
>* `*at*`匹配当前目录中含有at的文件名
>* `*/a`中的`*`匹配当前目录的所有目录（**不会匹配多个目录**），`a`匹配次级目录的`a`。所以匹配次级目录中含有a的文件名
>* `*/*/a`匹配次次级目录中含有a的文件名。

注意，globbing过程中`*`不会匹配多个目录。但是其他匹配过程不一定，比如匹配字符串（如locate pattern）。

注意，**只对参数有效**，对option无效，比如-name fil	ename，这个option没有globbing的过程。

## Dot Files

点文件或点目录的名字以`.`开始，一般用作配置文件。点文件和普通文件没有什么区别，只是一些程序默认不显示它们罢了。globbing过程也不匹配点文件，除非显示指定，如`.*`

## 环境和shell变量

* 介绍

  shell变量是shell存储的临时变量，存储字符串值。

  ```bash
  # 赋值或创建shell变量：
  $ STUFF=blah
  # 访问变量时需加上前缀$
  $ echo $STUFF
  ```

  环境变量类似于shell变量，但不特定于任何一个shell。**所有进程都有一个环境变量存储区**。通过export可声明一个环境变量：

  ```bash
  STUFF=blah
  export STUFF
  ```

* 区别

  环境变量和shell变量的**主要区别**在于：操作系统会将shell的所有环境变量**拷贝**给shell运行的程序，因此shell运行的程序不能访问shell变量，除了环境变量。

* 相关命令

	* `printenv`打印所有环境变量

## 常见环境变量

### PATH

`PATH`是一个特殊的环境变量，含有一些命名路径，通过分号`:`分隔。shell在执行命令时会根据PATH指定的目录查找命令所在位置，如果程序在多个地方存在，则运行第一个匹配的程序。

命令在多个路径下存在时，先找到的优先使用。

```bash
> $ echo $PATH
> /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```

> 添加新路径时, 经常会将`PATH`置于最后, 这是为了让自己的命令优先级更高.

### 默认编辑器

在`.bashrc`下添加

```shell
export EDITOR='program'
export VISUAL='program'
```

> 关于`EDITOR`和`VISUAL`, Bash默认使用`VISUAL`, 失败后使用`EDITOR`

### 其他

* 当前Shell

  `SHELL`

## input、output和redirect

所有进程都是通过**io流**对数据读入和写入的。io流分为两种：输入流和输出流。内核为进程提供了**标准输入流**、**标准输出流**和**标准错误流**，标准输出流和标准错误流都属于输出流。一般情况下，三者都被连接在terminal上。

还可以重定向标准流，而不是连接在terminal上：

1. 标准输出流

   1. 重定向到文件
      `>`以覆盖的方式写入文件中，`>>`以追加的方式写入到文件。如果文件不存在，则创建。如：

      >$ command > file
      >$ command >> file

   2. 重定向到输入流
      管道`|`将标准输出流定向到标准输入流。需要命令支持标准输入流，大部分命令都支持，可以通过man查看，如文法中出现类似`[FILE]`的情况，表示该文件可选，即从标准输入流中读入。

      >$ head /proc/cpuinfo | tr a-z A-Z

2. 标准错误流
   重定向标准输出流并不会重定向标准错误流。

   1. 重定向到文件
      `2>`用于重定向标准错误流，2是标准错误流的流ID（stream ID），默认1（标准输出流）

      > ls /ffffffff >f 2>e
      > 重定向标准输出流到文件f，重定向标准错误流到文件e

   2. 重定向到标准输出流
      `>&`可以重定向到其他流

      >$ ls /fffffffffff  >f 2>&1

3. 标准输入流
   `<`可以重定向文件到标准输入流，不常用，因为很多命令本身支持文件作文输入流。

   >$ head < /proc/cpuinfo

## navigation

* `Ctrl+a`去行首
* `Ctrl+e`去行尾
* `Ctrl+l`清屏

# 基本命令

## ls

`ls`命令列出目录中的全部内容（不包括以前缀`.`开始的文件）。默认显示当前目录。

>```bash
>ls [OPTION]... [FILE]...
>```
>
>* `-l`	显示文件的详细信息，不同类型的文件显示不同的颜色
>* `-a` 列出所有的文件，包括隐藏文件（“.”开头）
>* `-d` 查看目录，而不是目录中的内容
>* `-S` 通过文件大小排序
>* `ll` 为`ls -l` 的别名
>* `-h` 文件大小显示为人类可读的方式
>
>------------
>
>* 样式类选项
>
> * `--color[=WHEN]`: 文件类型是否以颜色区分
>
>   * `always`(默认): 一直是.
>   * `auto`: 在终端中有颜色
>   * `never`: 一直不
>
> * `-F,--classify`: 是否添加后缀以区分文件类型
>
>   > 如`*`表示普通文件, `/`表示目录, 等等
>
> * `--file-type`(推荐使用): 同样添加后缀, 但普通文件不加`*`

`ls -l`的各项内容为：
![在这里插入图片描述](../.Linux%25E5%2585%25A5%25E9%2597%25A8/20181216190751146.png)

第一个字符表示文件类型，有：

* d：目录文件	
* l：链接文件	
* b：块设备文件	
* c：字符设备文件	
* p：管道文件	
* -：表示普通文件

链接个数：指向文件的引用，包括`.` , `..` 和hard link，不包括symbolic link

## cp

拷贝文件，主要有两种形式：

>cp SOURCE DEST    如cp file1 file2
>cp SOURCE... DIRECTORY  如 cp file1 ... fileN dir
>
>* `i`表示覆盖文件前提示
>* `-n`不覆盖，因此不用提示
>* `-f`强制写入，目的文件不能打开则删除重试
>* `-r`递归拷贝目录

source也可以为目录，但是需要`-r`;cp被别名为`cp -i`，由于别名缘故，覆盖必提示，即使使用`-f`，[解决办法][1]：

>/bin/cp -rf /zzz/zzz/* /xxx/xxx
>或
>yes | cp -rf /zzz/zzz/* /xxx/xxx

[1]:https://stackoverflow.com/a/8488293/10248407

## type

从上面看到了，bash shell提供了别名功能，比如`ls`，`ll`，`cp`都被别名。type用来查看输入的命令是如何解析的，比如是内部命令、外部命令，还是别名，`-a`可以打印更多信息，比如命令位置。

>type [-a] name

>[root@localhost Desktop]# type ls
>ls is aliased to 'ls --color=auto'
>[root@localhost Desktop]# type -a ls
>ls is aliased to `ls --color=auto'
>ls is /usr/bin/ls

## cat

cat用来合并多个文件，然后输出。

>cat [OPTION]... [FILE]...
>
>* `-n`：编号所有行
>* `-A`：显示所有特殊字符，相当于`-vET`，其中`E`显示`\n`为`$`，`T`显示`\t`为`^I`，`v`用`^`和`M`显示其他特殊字符（貌似非ascii都被当做特殊字符了）

## mv

移动（或重命名）文件。

>mv [OPTION]... [-T] SOURCE DEST
>mv [OPTION]... SOURCE... DIRECTORY
>
>* `-i`：覆盖前提示
>* `-n`：不覆盖
>* `-f`：覆盖前不提示

source可以为目录，无需`-r`；同样mv别名为`mv -i`；注意与cp的不同。

## touch

创建文件，如果已存在，则仅更新文件的`修改时间`（可通过`ls -l`查看）。

>$ touch file

## rm

删除文件或目录。默认不删除目录，需要加`-rf`，慎用。

>rm [OPTION]... FILE...

## echo

显示字符串参数到标准输出。常用于流的重定向、输出shell变量等。

>echo "a string" >> filename
>echo $HOME

# 目录相关

unix的目录体系结构从`/`开始，即根目录。

* 绝对路径：以`/`开始的路径，如`/usr/lib`
* 相对路径：不以`/`开始的路径，如`dir/file`
* `..`：指向父目录
* `.`：指向当前目录

## cd

更改**当前工作目录**，即当前进程运行的目录。默认`HOME`变量，即无参数时跳到家目录。

```bash
#跳转到目录
$ cd dir
#无参数，默认家目录：
$ cd  
# ~ 也表示家目录
$ cd ~ 
# - 等于$OLDPWD，即上一个目录
$ cd -
```

## mkdir

创建新目录

>`mkdir dir`
>
>* `-p`：如果父目录不存在则创建

## rmdir

删除**空**目录，`-p`删除目录即祖先，如`rmdir -p a/b/c` 等于 `rmdir a/b/c a/b a`

*有必要可以尝试`rm -rf dir`*

## pwd

pwd（print working directory）打印当前工作目录。

>`-P`：避免所有的符号链接，即打印文件真正位置所在。

# Intermediate Commands

## grep

打印在文件或者输入流中**含有**被正则表达式匹配内容的**行**。

>grep [OPTIONS] PATTERN [FILE...]
>
>* `-i`：忽略大小写
>* `-v`：Invert the sense of matching, to select non-matching lines. 反向匹配
>* `-E`(`egrep`)：使用扩展正规表达式

看了`man grep`后，发现**基本正则**（grep）和**扩展正则**（egrep）基本没什么区别，但grep更符合我的预期。所有语言的正则语法都差不多，可参考：[正则表达式--java][2]

[2]:https://blog.csdn.net/jdbdh/article/details/82702285

## less

查看文件内容，一次一屏幕，more的增强版。less开始时不会读取全部文件内容，因此比vim、cat更快。

实用选项:

* `-R` : 打印有颜色的字符, 而非直接输出`^ESC`等控制字符(`-r`模式)

查看内容时一些方便的命令：

* 翻页

  * `f`  ：下一页 `b` ：上一页
  * `j` ：下一行 `k` ：上一行
  * `g` ：跳到第一行 `G` ：跳到最后一行

* 搜索

  * `/pattern`：前向搜索，正则匹配
  * `?pattern`：后向搜索，正则匹配
  * `n`：搜索下一个
  * `N`：搜索上一个

* **实用**

  * `h`：显示命令的总结（重要，必记）

  * `v`：使用默认编辑器编辑该文件

  * `F`: 向下滚, 如果已经到了文件尾, 则实时监控, 此时相当于`tail -f`

    > 实时监控功能还是没有`tail`强, 如文件内容被清空时`less`不能检测到

  * `R`: 丢弃缓存, 重绘屏幕.
  
  * `-N` 打印行号

## head and tail

快速浏览文件或数据流的部分内容。

>head file 默认显示前10行
>`-n k`：显示前k行

>tail file 默认显示后10行
>`-n k`：显示后k行

小技巧!

* 显示第16到第20行：

  ```bash
  $ cat -n filename | head -n 20 | tail -n 5
  ```

* 实时输出

  ```bash
  tail -f filename
  ```

## file

**猜测**文件类型，通常很准确。

>$ file file

## diff

一行一行的比较两个文件

>$ diff file1 file2

## sort

Write sorted concatenation of all FILE(s) to standard output.

和cat类似。

# 文件查找

## find

在以给定目录为根的目录树中，搜索文件。该命令比较复杂，先给出最简形式，以后补充：

```bash
$ find dir -name fileBaseName
$ #或
$ find dir -regex fullPath
```

* 查找的目录

  `dir`可以含有通配符；

* 查找的文件

  * `-name <pattern>` 匹配文件的base name（即没有leading directories）, 可以使用通配符；
  * `-iname <pattern>` 同`-name`, 但大小写不敏感
  * `-regex <pattern>` 匹配文件的全路径名, 可以使用正则表达式
  * `-iregex <pattern>` 同`-regex`, 但大小写不敏感

* 对匹配项的动作

  * `-print` (默认) 打印找到的文件的全名（full file name）。

## locate

在系统内建的数据库中查找文件，因此速度更快。该数据库会周期性的更新，如果一个文件在更新前添加进来，locate则不能找到该文件。

>locate pattern
>pattern含有通配符，匹配整个字符串；如果没有，相当于\*pattern\*
>例子（匹配/usr/bin/cp）：
>$ locate */cp

>locate -r regexp
>使用正则表达式；一直是匹配字符串部分内容。
>例子（匹配/usr/bin/cp）：
>$ locate -r .*/cp$

## whereis

查找$PATH目录下的文件，定位含有name的文件名

>$ whereis name

## which

在$PATH目录下查找命令的全路径名

>$ which programname

除此之外，`type -a`能够查找一些命令所在位置。

# Miscellaneous

## passwd

passwd用来改变密码。需要输入旧密码和新密码两次。通过调用Linux-PAM和Libuser API接口实现该命令。

## chsh

改变你的登录shell，默认使用bash。

>* chsh -s shell ：设置新的shell，必须存在于/etc/shells。shell可以为全路径名，或者可执行文件名。
>* chsh -l ：打印/etc/shells中的所有shell。

## sleep

延迟一定时间

>sleep NUMBER[SUFFIX]...
>suffix默认s，可选m、h、d

## cal

显示日历（calendar），默认显示当月

>cal [options] [[[day] month] year]

## date

打印或设置日期。

```shell
$ date
Wed Dec 19 18:19:46 CST 2018
```

> 这里不展开, 详细见下方*时间*

## groups

显示当前用户所属组

## dmesg

显示系统缓存（日记）。

## lsb_release

打印特定于发行版的信息，如`lsb_release -a`

## alias

给命令起别名, 如

```bash
#简化git操作
alias push="git add . && git commit -m '补充' && git push"
#取消ls匿名
alias ls=ls
```

> 通过覆盖, `ls`可以取消高亮

# Vim

vim是一个非常强大的编辑器，也是linux默认的命令行编辑器。源于vi，而如今linux上的vi命令成了vim的别名。vim功能即使强大，但太繁杂了，这里只给出必要内容。

参考：https://www.tutorialspoint.com/vim/index.htm

## vim模式

* **Command mode**：进入vim后的默认模式。该模式下，可以使用很多命令，如拷贝、粘贴、删除、查找等等。
* **Insert mode**：Command模式下，很多方法可以进入该模式，如`i`（insert插入），`a`（append添加）等。该模式下可以进行内容编辑。按`ESC`退回Command模式。
* **Command line mode**：Command模式下，键入`:`进入该模式。该模式下可以执行一些命令，如保存、退出等等。按`ESC`退回Command模式。
* **Visual mode**：Command模式下，键入`v`进入该模式。该模式下可以可视化选择文本，并在之上执行命令。执行完命令后（如拷贝），会自动回到Command模式。

有时候这些模式不用特意区分，因为界限有点模糊。

## navigating

在Command和Command line模式下，都可以通过命令移动光标：

* `h`：向左移一位
* `l`：向右移一位
* `j`：向下移一位
* `k`：向上移一位

注意，可以移多位，只需在前面加数字，如下移10位：10j

* `0`：移到行开始
* `$`：移到行尾
* `:0`：移到第一个行
* `:$`：移到最后一行
* `ctrl+f`：下滚一屏
* `ctr+b`：上滚一屏
* `:n`：跳到第n行

## editing

从Command进入insert模式有很多中方法，下面给出有用的：

* `i`：插入文本在光标前
* `I`：插入文本在行首
* `a`：添加文本在光标后
* `A`：添加文本在行尾
* `o`：光标之下添加新行
* `O`：光标之上添加新行

未进入insert模式前可以进行拷贝、粘贴、删除等操作：

* `x`：从光标后删除一个字符
* `X`：从光标前删除一个字符
* `dd`或`D`：删除整行
* `y`：拷贝整行，但是在visual模式下是拷贝被选中的内容。
* `p`：在光标后粘贴

注意，在visual模式下被选中的内容可以看做一个字符。

撤销、重做：

* `u`：撤销
* `ctrl+r`：重做

## 搜索

* `/expression`：向下搜索，不存在正则。
* `?expression`：向上搜索。
* `n`：查找下一个
* `N`：查找上一个

## Buffer和Swap

打开文件时，文件内容会被读入**Buffer**中，如果文件不存在，则Buffer为空。buffer位于RAM。在写入时，才将buffer中的内容写入到真正的文件中，如果文件不存在则创建。

为了防止意外发生，导致buffer未及时写入文件中（即编辑过程中断电）。vim会周期性地将buffer写入到对应的**swap**文件中。通过比对文件和swap文件时间可以查出是否发生异常，然后利用swap文件恢复。

## 其他

* `:w`：保存文件
* `:q`：离开

如果操作失败，则使用`!`强制保存或退出。通常`wq`一起使用，必要时`wq!`

------------

可以为单个用户配置vim，如每次开始vim时默认显示行号：

>$ cd  //进入家目录
>$ vim .vimrc //打开该文件，没有将被创建
>set nu //设置文本内容
>:wq //保存

# 获得帮助

## man

通过man可以查阅相关命令指南（manual pages），但是内容很多，不会告诉你重点，也不适合作为教程。man手册是通过`less`命令显示的，因此less的快捷键能够使用。

manual pages被分为了很多部分，如：

| section | description                                                  |
| ------- | :----------------------------------------------------------- |
| 1       | Executable programs or shell commands                        |
| 2       | System calls (functions provided by the kernel)              |
| 3       | Library calls (functions within program libraries)           |
| 4       | Special files (usually found in /dev)                        |
| 5       | File formats and conventions eg /etc/passwd                  |
| 6       | Games                                                        |
| 7       | Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7) |
| 8       | System administration commands (usually only for root)       |
| 9       | Kernel routines [Non standard]                               |

man在查找手册时，默认只会显示最近的一页。`

* `-a`会查找全部

  >$ man -a passwd

* 如果知道关键字，不知道具体名字

  >$ man -k keyword

* 如果知道具体section，如passwd(5)

  >$ man 5 passwd

----------

man手册的语法：
![在这里插入图片描述](../.Linux%25E5%2585%25A5%25E9%2597%25A8/20181217231834418.png)

------------

命令的选项（options）表示的形式**至多**三种，如下：

1.   **UNIX options**, which may be grouped and must be preceded by a dash.
2.   **BSD options**, which may be grouped and must not be used with a dash.
3.   **GNU long options**, which are preceded by two dashes.

一般使用第一种。

## 其他帮助

* info：内容比man更多，更复杂
* `--help`或`-h`：一些命令自带简单帮助，通过该option可打印。
* /usr/share/doc：一些帮助文档会被放入此处
* Internert：网络上要啥有啥

## man页面安装

在`/usr/share/man/`和`/usr/local/shar/man/`下有很多子文件夹，每一个子文件夹存放一类的man页面。如果有额外的man页面，则应该放入`/usr/local/share/man/`下对应的子文件夹中，然后执行`mandb`以更新man的内部数据库，便能使用`man command`查看命令手册了。

>实际上，man page就是个文本，可直接用less查看。

# 进程管理

## ps

显示当前进程信息。常用BSD风格的选项（option）：

| 命令     | 描述                                                      |
| -------- | :-------------------------------------------------------- |
| ps       | 显示**当前用户**和**同一终端**下的所有进程                |
| ps x     | 显示**当前用户**的所有进程                                |
| ps ax    | 显示系统内的所有进程                                      |
| ps u     | 显示更详细的信息                                          |
| ps u pid | 打印进程号为pid的进程的详细信息，其中$$含有当前shell的pid |

显示时一些列的含义：

| USER         | PID    | %CPU      | %MEM         | VSZ                | RSS                | TTY    | STAT     | START    | TIME            | COMMAND |
| ------------ | ------ | --------- | ------------ | ------------------ | ------------------ | ------ | -------- | -------- | --------------- | ------- |
| 进程所属用户 | 进程id | cpu利用率 | 驻留内存比例 | 虚拟内存大小（KB） | 驻留内存大小（KB） | 终端？ | 进程状态 | 启动时间 | cpu累积使用时间 | 命令    |

stat第一个字符表示总的状态，其他字符表示额外的信息：

| 第一个字符 | 含义                                                         |
| ---------- | :----------------------------------------------------------- |
| D          | uninterruptible sleep (usually IO)                           |
| R          | running or runnable (on run queue)                           |
| S          | interruptible sleep (waiting for an event to complete)       |
| T          | stopped by job control signal                                |
| t          | stopped by debugger during the tracing                       |
| W          | paging (not valid since the 2.6.xx kernel)                   |
| X          | dead (should never be seen)                                  |
| Z          | defunct ("zombie") process, terminated but not reaped by its parent |

| 其他字符 | 含义                                                         |
| -------- | :----------------------------------------------------------- |
| <        | high-priority (not nice to other users)                      |
| N        | low-priority (nice to other users)                           |
| L        | has pages locked into memory (for real-time and custom IO)   |
| s        | is a session leader                                          |
| l        | is multi-threaded (using CLONE_THREAD, like NPTL pthreads do) |
| +        | is in the foreground process group                           |

## pstree

打印所有进程的树状结构

## kill

让内核发送一个**信号**给进程，常用于结束进程。

>* 命令形式
> * `kill [-s signal] pid...`  
> * `kill [-signal] pid...`
> * `kill -l`
>* 选项
>
>* `-s`指定要发送的信号，默认`term`。`signal`可以是信号名或对应的数值。完整信号名前3个字符“SIG”可以省略。
>* `-l`打印所有信号及对应数值。

信号按照功能（Action）可以分为5类，每一类都有默认处理方式，如：

| Action | 默认处理                                                     |
| ------ | :----------------------------------------------------------- |
| Term   | Default action is to terminate the process.                  |
| Ign    | Default action is to ignore the signal.                      |
| Core   | Default action is to terminate the process and dump core     |
| Stop   | Default action is to stop the process.                       |
| Cont   | Default action is to continue the process if it is currently stopped. |

进程可以捕获（catch）信号，然后执行自己的处理，如果没有捕获则会执行默认处理。如ping捕获`INT`信号（`ctrl+c`），打印最终结果并结束程序。但是`kill`和`stop`信号不能够被捕获，因此必定执行默认行为。

完整信号参考：`man 7 signal`，部分信号如下：

| signal  | value    | action | comment                      |
| ------- | -------- | ------ | :--------------------------- |
| SIGINT  | 2        | Term   | `ctrl+c`发出                 |
| SIGQUIT | 3        | Core   | `ctrl+\`发出                 |
| SIGKILL | 9        | Term   | 不会被捕获，用于强制结束进程 |
| SIGTERM | 15       | Term   | kill默认发送信号             |
| SIGCONT | 19,18,25 | Cont   |                              |
| SIGSTOP | 17,19,23 | Stop   | 不会被捕获，用于强制停止进程 |
| SIGTSTP | 18,20,24 | Stop   | `ctr+z`发出                  |

*为什么有的信号对应多个数值？因为一些信号在不同处理器架构上不兼容，所以最好使用信号名*

*那`ctrl+D`会发出信号吗？不会，它只是在terminal的标准输入中表示文件结束（EOF）。*

## Job Control

job control是shell提供的一个功能。job就是一行命令的抽象概念，这个命令可以是管道，因此一个job可以是多个命令的组合。在操作系统中job被表示成**进程组**（process group），每个job都有个**job ID**。

**job控制**就是暂停、恢复或结束job（进程组）中的所有进程。更复杂的操作可以通过发送信号（kill命令）实现。

进程可以在后台（background）或前台运行。一般进程运行在前台，如果运行在后台，为了确保后台进程的不会输入输出到terminal上，可以使用重定向防止此事发生。

主要使用的命令：

* `jobs`：列出**当前shell**中所有**暂停**或**后台运行**状态的job。同时显示它的job id。
* `fg [%jobID]`：指定job id，将该job带入前台。如果没有job id，默认最近进入jobs列表的job。
* `bg [%jobID]`：指定job id，带入后台。若有没有，默认最近的job。
* 暂停：如果是前进进程，使用`ctrl+z`；其他的使用`kill -stop %jobID`
* 结束：`ctrl+c`，如果不行，则`kill -kill %jobID`强制结束。

参考：
https://en.wikipedia.org/wiki/Job_control_(Unix) 
http://www.aboutlinux.info/2005/05/job-control-in-linux.html

## top

> 进阶使用见6.2小节

top类似ps，用来显示进程，但能够动态显示进程，并且默认按cpu利用率排序。

一些单词描述：

* PID – the process ID of the task.
* USER – task’s owner.
* PR – the priority of the task.
* NI – the nice value of the task.
* VIRT – the total amount of virtual memory used by the task.
* RES – the non-swapped physical memory the task has used.
* %CPU – the task’s share of the CPU time.
* %MEM – the task’s share of the physical memory.
* COMMAND – the command used to start the task.

排序：

* P：按cpu利用率排序
* M：按内存使用率排序
* R：反转排序
* h：帮助
* q：离开

杀死进程：
键入k和进程的pid

# 文件

## 文件权限

`ls -l`输出的第一列为**file mode**，由四部分组成：文件类型、user权限、group权限和other权限。如：
![在这里插入图片描述](../.Linux%25E5%2585%25A5%25E9%2597%25A8/20181219140507506.png)
这些权限都有三位权限位组成，分别是`r`、`w`、`x`。

实际上还有一组特殊权限SUID（`s`）、SGID（`s`）、SBIT（`t`），如果有的话会分别显示在user、group、other权限的**第三位**。如果原先该位置上的`x`权限不存在的话，特殊权限位会显示为**大写**。

每位的含义：

* `r`：文件或目录内容可读。
* `w`：文件或目录内容可写。
* `x`：文件可执行或目录可搜索（即访问目录中文件内容的“开关”）。
* `-`：表示此位无权限。
* SUID（`s`）：仅对二进制程序有效，该程序设置user ID，即以user的身份运行。例如，`passwd`的权限和拥有者为`-rwsr-xr-x. 1 root root`，作为普通用户有`r-x`权限，可执行它；由于suid权限的存在，程序以root身份运行程序，并读取只有root权限才可访问的/etc/passwd文件。
* SGID（`s`）：仅对二进制程序有效，该程序设置group ID，即拥有group的身份。
* SBIT（`t`）：仅对目录有效，该目录下的文件或目录只能它的拥有者（user）删除或重命名。

---------------

**提示**

* 目录存有文件的文件名和指向I-Node的引用，因此rw针对的也是这些内容；如果要访问I-Node，则需要x权限。
  因此如果用户对目录只有`rw-`权限，可读取文件名（如ls），但不能读取更多内容（如`ls -l`出现很多问号），但可以删除新建文件（w权限）；如果用户对目录只有`--x`权限，那么不能访问目录的内容，但是能访问I-Node。*如果I-Node代表可执行文件，可以执行吗？仅限二进制文件（见下一点）*
* 脚本文件必须有read权限，因为解析器需要读取脚本内容；二进制文件不必，因为是内核读取文件内容的。
* I-Node节点含有文件属性和所有数据块的引用。
* 文件设置不可写，为何vim还能强制写入？因为vim写入时，是通过新文件（swap）替换旧文件实现的，是对目录操作，和目录权限有关！！！同理，文件不可读时，vim是真的没有读取文件，创建了个空Buffer。
* 文件所属者（user），可以使用组身份（group）或者其他人身份（other）吗？不能！！即使other权限比user权限大。同理组成员不能使用other身份。

## umask

umask和文件、目录建立时的权限默认值有关。也就是文件或目录初始值减去umask。先看umask默认值

>$ umask
>0002
>$ umask -S
>u=rwx,g=rwx,o=rx
>第一个表示特殊权限，其他三个数值分别表示user、group、other权限

* 文件：普通文件没有可执行权限，因此初始`-rw-rw-rw-`，即666，减去umask（002），为664（`-rw-rw-r--`）
* 目录：进入目录需要x权限，因此初始`drwxrwxrwx`，即777，减去umask（002），为775（`drwxrwxr-x`）

umask命令也可以设置默认值

>umask octal-mode
>4位八进制，可以省略最高位特殊权限

## chmod

改变文件权限。

```bash
chmod [OPTION]... MODE[,MODE]... FILE...
chmod [OPTION]... OCTAL-MODE FILE...
```

* `-R` 递归改变文件mode

可以通过符号表示或八进制数修改权限：

* **symbolic mode**(符号模式)

  语法形式：`[ugoa...][[+-=][perms...]...]`

  * 其中`perms`为`rwxXst`中的零个或多个。

  * `u`为user用户, `g`为group组, `o`为other其他人, `a`为all所有人；

  * `+`表示添加权限位；`-`表示删除权限位；`=`表示设置该三位权限，没有设置的默认无。其他三位是特殊权限位.

  * 例子：

    ```bash
    $ chmod g+r file
    $ chmod o-r file
    $ chmod go+r file
    $ chmod u+s file
    $ chmod o+t file
    ```

* **octal-mode**(八进制模式)

  * 使用四个八进制数表示，第一个设置特殊权限，可以省略不写。其他三个分别对应用户,组,其他人的权限. 每一位都是由`rwx`组成的三位二进制数的数值, 因此`rwx`的权重分别为4,2,1. 相应的, 特殊权限位的权重分别为`suid`(4),`sgid`(2),`sbit`(1)。

  * 例子：

    ```bash
    chmod 664 file
    chmod 775 directory
    chmod 1775 directory  //设置了sbit位
    ```

  > **注意**，特殊权限位除非明确设置，否则不会被清楚或改变；貌似只有文件拥有者或root可以修改文件的权限位。

## chown

改变文件所属用户或组

>```bash
>chown [OPTION]... [OWNER][:[GROUP]] FILE...
>```
>
>* `-R`：递归更改


## 链接文件

创建链接文件后，对链接文件操作相当于对源文件操作。链接文件分为：硬链接文件（hard link）和符号链接文件（symbolic link）。

hard link是一个直接指向源文件的文件，即I-Node与源文件一致；symbolic link是一个间接指向源文件的文件，即该文件含有源文件路径的文本。

>`ln [-s] target linkname`
>**target必须是全地址名**！！可借助环境变量`$PWD`
>
>* `-s`表示创建符号链接，目标文件可以不存在；否则创建硬链接，目标文件必须存在

# 归档及压缩

## 压缩

`gzip`是最常用的一个压缩命令，将文件压缩，并自动为名字添加后缀：`.gz`。`gunzip`是对应解压命令，将文件解压，去掉后缀。压缩和解压后源文件都会消失。除此之外，还有其他同类工具，如下表格：

| 压缩命令 | 解压命令 | 默认后缀 | 压缩率 | 速度 | 备注                         |
| -------- | :------- | :------- | :----- | :--- | :--------------------------- |
| gzip     | gunzip   | `.gz`    | 低     | 最快 | gunzip也可以解压 `.Z`        |
| bzip     | bunzip2  | `.bz2`   | 高     | 较快 |                              |
| xz       | unxz     | `.xz`    | 最高   | 最慢 |                              |
| zip      | unzip    | `zip`    | ？     | ？   | 主要为了兼容windows的zip档案 |

## tar--归档

tar主要是让多个文件或压缩包归为一个档案。除此之外它还可以使用上面的压缩工具对档案压缩或解压。

>tar [OPTION...] [FILE]...
>file可以是文件或者目录
>主要操作：
>
>* `-c`：创建档案
>* `-x`：提取档案内容
>* `-t`：列出档案所有文件
>* `-r`：添加文件到档案后
>
>常用选项：
>
>* `-v`：详细列出所有操作步骤
>* `-f`：档案名，后面必须接档案名
>* `-z`：使用压缩工具gzip
>* `-j`：使用压缩工具bzip2
>* `-J`：使用压缩工具xz
>
>其他选项：
>
>* `--skip-old-files`：提取文件时不覆盖以存在文件。**默认覆盖**。
>* `-p`：提取文件时，保留它的文件权限。root用户默认此选项。
>* `-C`: 解压文件存放路径, 前提路径已存在

一些例子：

```bash
$ tar -cvzf archive.tar.gz file1 file2 directory
$ tar -xvzf archive.tar.gz
```

创建档案时，对档案后缀没有强制规定，但是最好使用常用后缀名。比如：归档不压缩，`file.tar`；归档并用gzip压缩，`file.tar.gz`等等之类。注意到一些后缀，如`.tgz`与`.gz`相同，`.taz`与`.tar.Z`相同。

# 加解密

> 参考[linux下文件加密方法总结](https://www.cnblogs.com/wuchangsoft/p/11747739.html)

## ZIP加解密

加密

```shell
zip -e test.txt.zip test.txt
```

> 选项`-e`指定加密后的文件名; 之后会被要求输入密码两次

解密

```shell
unzip -j test.txt.zip
```

> 选项`-j`创建归档路径目录; 之后会被要求输入密码两次

> 或者使用`--password`或`-P`选项, 直接指定密码

# 目录结构

linux的发行版都准守Filesystem Hierarchy Standard（FHS），每个目录都有自己的含义。但会FHS也有一些模糊的地方，因此不同的发行版的目录结构会有一定的不同。
![在这里插入图片描述](../.Linux%25E5%2585%25A5%25E9%2597%25A8/201812192235513.png)

补充:

* `var/`
  * `run/` 含守护进程的PID文件
  * `log/`含守护进程的日志

注意：

* linux中，一切皆为文件。
* `/sbin`下的命令通常需要root权限才能执行。
* `/tmp`目录下的内容一般会在主机自启时清空

极力推荐，里面的目录结构概括的比我好：[An introduction to Linux filesystems](https://opensource.com/life/16/10/introduction-linux-filesystems)

参考：
[redhat filesystem-fhs](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Reference_Guide/s1-filesystem-fhs.html)
[wiki fhs](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard#cite_ref-/opt_6-0)
[fhs geeksforgeeks](https://www.geeksforgeeks.org/linux-file-hierarchy-structure/)

# 权限转换

## sudo

运行命令时以其他用户身份运行。默认以root身份运行。它会同时设置`euid`和`ruid`, 见6.1小节

```bash
$ sudo groups
root
```

首次运行sudo时，需要输入当前用户密码，然后默认持续5分钟不用再次输入密码。你没看错，不是输入root用户密码，如果需要，可以使用`visudo`命令修改`/etc/sudoers`文件，参考：https://superuser.com/a/1134306

使用时, 可能会碰到命令不存在的问题, 这是因为sudo只会在特定目录下寻找命令, 与用户或root的`PATH`变量无关. 由`/etc/sudoers`的`secure_path`决定, 可自行修改它.

## su

开启**新shell**，以其他用户身份登录。默认root登录。

>```bash
>su [options...] [-] [user [args...]]
>```
>
>* `user`：登陆用户, 默认以root身份登录, 不改变当前目录，仅改变环境变量：HOME、SHELL、USER、LOGNAME
>* `-`: 不存在时, 仅改变部分环境变量, 如`PATH`,`HOME`,`SHELL`,`USER`,`LOGNAME`. 存在时, 与用户登陆时的环境基本一致.

> `exit`命令将退出当前shell

# 用户

## 介绍

linux是一个多用户的系统，有用户、组的概念。不同的用户、组对于不同的文件（夹）有着不同的访问权限（在文件inode中定义），见[linux入门 2.10小结][61]。一个用户有一个primary group，用户也可以加入其它组。每个用户、组都有对应的ID，内核中只使用ID，用户名只在用户空间中使用。

运行的进程会记录用户ID，但有很多中ID，常见的如下：

* **有效用户ID（euid）**：它给与了进程对应的访问权限；如`passwd`运行时将euid换成root，因此可以访问`/etc/shadow`，修改密码。
* **真实用户ID（ruid）**：指定了可以操作进程的用户；`passwd`即使修改了euid，用户也可以`kill`它，如果`sudo`运行命令，非root用户不能`kill`它，因为sudo同时修改了euid和ruid。

## 登录过程

注意，这块内容并不一定对，纯属个人猜想

* 在主机前登录，getty会连接终端，启动login程序进行用户验证，成功后为你打开shell。
* 远程ssh登录，终端连接、用户验证都由ssh server完成，然后打开shell。
* 桌面环境登录，由其他软件（不知道啥子）进行用户验证，然后就如桌面系统（可看做为GUI shell）。

[61]:https://blog.csdn.net/jdbdh/article/details/85038340#210__545

## 配置文件

`/etc/passwd`中每一行存储了一个用户的账号，每个字段如下：
![在这里插入图片描述](.Basic/2019031513453198.png)
其中

* Password：现在密码存在`/etc/shadow`中，因此填入`x`表示加密过的密码在shadow文件中；`*`则阻止用户认证（即不能登录）
* Group ID：primary group的id，必须在`/etc/group`中存在
* GECOS：就是注释，额外的信息
* Shell：登录成功后运行的程序。通常是shell，默认使用`/bin/sh`，nologin表示不能登录，见`/etc/shells`

>在`/etc/shadow`的第二个字段是真正存储加密密码的地方，`!`，`*`表示不用密码登录，空表示登录不需要密码。详细见`man 5 shadow`

**注意**！！不能登录并不意味着进程的uid不能为该用户，反而使用不能登录的用户运行程序更安全。一般在使用服务进程，systemd unit中可以配置运行用户。

-------------

`/etc/group`中每一条每一个组，组的用户除了passwd对应的用户，还有最后一个字段指定的所有用户。
![在这里插入图片描述](.Basic/20190315143530103.png)

## 命令

### useradd

用户添加用户（通过修改和用户相关的配置文件实现）。

默认行为：

* 创建用户的同时创建对应的组

* 默认bash shell

* 没有过期时间

* 无密码且不能登录，需要使用`passwd`修改密码。

* 应该有家目录

  > 实际测试, 无, 需要手动创建

>默认行为在`/etc/default/useradd`和`/etc/login.defs`中配置。

----

>```bash
>useradd [options] LOGIN
>```
>
>因为默认行为很好的满足了需求，因此没有给出完整选项
>
>* `-c`：passwd中的注释
>* `-s`：登录的shell，默认bash
>* `-p`：设置密码，不建议使用，因为没有加密。建议使用`passwd`命令。
>* `-r,--system`：创建系统账户，默认不创建家目录

### userdel

删除用户（实现同上）

>userdel [options] LOGIN
>默认仅删除账户和用户组
>
>* `-r`：同时删除家目录的文件和邮件。其他地方的文件需要手动删除
>* `-f`：同上，但强制删除，即使该用户在登录状态。

### passwd

修改密码

```shell
passwd # 修改当前账户密码
sudo passwd root # 修改root账户密码
sudo passwd -l root # 让账户密码处于过期状态
```

### chsh

修改shell

# top

top能够动态显示系统总的cpu、内存使用状态和各个进程的资源使用情况。

## 字段含义

第一行分别表示目前的时钟、系统已运行时间、用户个数和分别为1、5、15分钟的平均负载。

>平均负载是对运行队列的长度的一种度量。单核下，0.5表示cpu一半时间是空闲的，1表示cpu是时刻负载的，1.5表示1/3进程在等待cpu；假设4核，则4表示所有的核刚好负载。当负载很高时，很有可能内存不足，导致时刻发生对内存的置换（swap）。[load average](https://unix.stackexchange.com/a/303703)

-----

第二行表示当前进程总个数、运行进程个数、睡眠进程个数、停止进程个数等等。按`H`可以看到线程的个数。

---

第三行表示cpu使用情况，各个单位如下：

* `us`（userspace）：程序在用户空间执行所需时间
* `sy`（system call）：系统调用所需时间，不包含内核自己花掉的时间
* `ni`（nice）：不知道
* `id`（idle）：cpu空闲时间，没有进程运行
* `wa`（wait）：cpu花在等待io上的时间
* ...

>这里显示的是CPU总的使用情况，按`1`可以看到各个CPU的使用情况。

----

第四、五行分别表示内存、置换内存的使用情况。按`m`可以看到图形表示。

-----

下面一部分是进程的一些信息和对资源的使用情况。每个字段如下：

* `PID`：进程id
* `USER`：有效用户（euid）。
* `PR`：进程的优先级。值从-20到20，越低优先级越低。会随时间而改变。
* `NI`：进程的nice值，是给与内核调度器该进程优先级的暗示，进程优先级下次改变时会考虑到该值。
* `VIRT`：虚拟内存大小（KB）。包含内存与置换内存中进程的所有页。
* `RES`：驻留内存大小（KB），被使用的物理内存大小。
* `SHR`：共享内存大小（KB）。
* `S`：进程状态。
  * `R`：运行；
  * `S`：睡眠；
  * `T`：被job控制信号中断；
  * `t`：被调试器中断；
  * `D`：不可中断睡眠；
  * `Z`：僵死。
* `%CPU`：cpu使用率。是进程的所有线程的cpu使用率综合，且每个核心最大100%，因此此项可能会超过100%。~~使用`H`显示线程使用率而不是进程，或者按`I`关闭`Irix`模式（默认模式），此时cpu总使用率为100%。~~
  * `Irix`模式(默认): 每条记录显示的是线程使用单个CPU的使用率
  * 非`Irix`模式: 显示进程占所有CPU的使用率. 
  * 通过`I`进行切换.
* `%MEM`：物理内存使用率
* `TIME+`：进程已运行时间
* `COMMAND`：运行该进程使用的命令

## 快捷键

* `h`：**最常用！！！内容必看**
* `L`，`&`：L查找进程，&搜索下一个
* `F` 添加, 删除, 排序, 移动列
* **导航**：方向键都可以用，如up,down,left,right,home,end
* `k`：与kill命令一致，给出PID和信号（可数字或信号名）。停止进程建议term，后kill。
* `d`或`s`：设置刷新间隔，默认3秒
* `x`，`y`：x排序列高亮，y运行进程行高亮
* 颜色：z设置颜色，b设置粗体。Z全局设置颜色
* `f`：重新设置显示的字段、排序字段等
* `W`：设置好后，按W保存设置到配置文件中。**有用！！！**
* `q`：离开。

# 时间

在Linux中有**硬件时钟**与**系统时钟**等两种时钟。硬件时钟是指主机板上的时钟设备，也就是通常可在BIOS画面设定的时钟；系统时钟则是指kernel中 的时钟；所有Linux相关指令与函数都是读取系统时钟的设定。当Linux启动时，系统时钟会去读取硬件时钟的设定，之后系统时钟即独立运作。

>至于网络时间，以后补充

## date

查看或设置系统时钟

>`date` 查看本时区当前时钟
>`date -s "2018-11-2 22:30"` 设置系统时钟，但重启后失效
>`date -u` 显示UTC时区的时钟

## hwclock

查询或者设置硬件时钟

>`hwclock`   查询硬件时钟
>`hwclock -w` 将系统时间写入硬件时间

## 时区

默认时区由`/etc/localtime`给出，它是一个指向`/usr/share/zoneinfo`中文件的一个符号链接。可以通过拷贝其他文件，来永久修改时区。或者修改环境变量`TZ`为当前shell会话设置临时时区，如：

```bash
[root@localhost Desktop]# date
Sat Mar 16 11:00:47 CST 2019
[root@localhost Desktop]# export TZ=US/Central
[root@localhost Desktop]# date
Fri Mar 15 22:00:54 CDT 2019
```

>如果不知道时区名字，可以参考`/usr/share/zoneinfo`目录或执行`tzselect`命令

# 计划任务

>计划任务有望被init进程（如systemd）取代，但目前远没有那么强大。

## cron

cron是一个守护进程，用于在规定时间点上执行任务（job）。任务在配置文件中配置，一行为一条任务。cron每分钟都会被唤醒，检查配置文件是否被更新。任务的标准输出会以邮件的形式记录，可以被修改。cron任务分为两类：

* 系统cron任务

  以root用户执行，常用于执行系统维护。需在`/etc/crontab`中配置。

  >`/etc/cron.d/`呢？不太懂

* 用户cron任务

  普通用户创建的任务，配置位于`/var/spool/cron`目录下。建议使用命令`crontab`创建任务，而不是直接操作配置文件。

  >为什么系统cron任务不用crontab命令创建？因为格式稍有不同，且anacron更适合系统cron任务，见下。

### 系统cron任务

* 使用

  直接在`/etc/crontab`中添加一行，格式如下：
  ![在这里插入图片描述](.Basic/20190316112911405.png)

  > 每个字段空格分隔。

* 特殊符号

  * `*`表示匹配所有可能的值；
  * `,`指定所有能匹配的值
  * `-`指定能匹配值的范围
  * `/`步进匹配, 如`0/20`, 0为初始值, 匹配一致, 后面每隔20个单位匹配一次

* 例子

  ```
  00 21 * * * root rm /home/bob/trash/*
  ```

  表示每天~~21~~ 22 点清空bob用户的垃圾。

> 参考：
>
> * [Schedule jobs with cron](https://geek-university.com/linux/schedule-jobs-with-cron/)
> * [Cron表达式的详细用法](https://www.jianshu.com/p/e9ce1a7e1ed1)

### 用户cron任务

`crontab`命令用户安装、展示、删除对应用户的cron任务。使用命令安装而不是直接修改对应配置文件的好处在于，在任意地点建立自己配置文件后，使用crontab安装的同时会检测语法是否正确，然后提示修改或安装到`/var/spool/cron`中。
![在这里插入图片描述](.Basic/20190316134732292.png)

>可以看出，用户配置少了user-name

>* `crontab [-u user] file`：安装自己的配置文件，默认当前用户
>* `crontab -e`：拷贝配置文件到临时文件中，用户编辑后，直接安装（覆盖）。如果配置文件不存在，则新建。**推荐**
>* `crontab -l`：显示所有该用户的cron任务
>* `crontab -r`：删除用户cron配置文件

参考：[User cron jobs](https://geek-university.com/linux/user-cron-jobs/)

## anacron

与cron不同，如果任务自从上次更新时间超过了某个时间间隔，则执行任务。任务只在`/etc/anacrontab`中配置，每个字段如下：
![在这里插入图片描述](.Basic/20190316140439800.png)

* `period`：命令执行的频率，单位：天。
* `delay`：执行命令前的延时
* `job-identifier`：任务标识，必须有
* `command`：任务执行的命令。

anacron运行过程

>anacron通过启动脚本或cron任务启动，anacron检查每个任务，判断距离上次执行是否已经超过（或等于）周期（period），如果是则执行命令，然后记录更新日期（不记录小时），当今天没有任务执行时，anacron结束。

参考：[Schedule jobs with anacron](https://geek-university.com/linux/schedule-jobs-with-anacron/)

## at

在某个时间点只执行一次。

>* `at time`：从标准输入中获得要执行的命令，按`ctrl+d`退出。表示在time时执行命令。
>* `at -f file time`：同上，不过此时命令通过文件给出
>* `atq`：列出所有的用户悬停的任务
>* `atrm job_num`：删除对应job num的任务

注意，时间可以为两种形式：

* `HH:MM [YY-MM-DD]`：如果只给出HH:MM并且过期，则认作明天
* `now + count time-units`：time-units可以是minutes，hours，days 或weeks。例子：`at now +2 hours`

# 其他

## nohup

运行指定的命令，并让该命令忽略掉停止信号，以至于**后台**应用能够在用户登出时继续运行

>* `nohup COMMAND`：标准输入流重定向到`/dev/null`，标准输出流重定向到nohup.out，标准错误流重定向到标准输出。
>* `nohup COMMAND > FILE`：标准输出流重定向到文件。

一些进程能够自我守护进程化，即自动后台运行并与控制终端脱离，因此当前会话结束不会关闭守护进程。如果进程不能自我守护进程化，则需要nohup的帮助，如：`nohup COMMAND &`

## time

测试命令所花掉的时间

>`time command`
>其中
>
>* `user`为进程在用户空间执行的时间
>* `sys`为系统调用在内核空间执行的时间（不包括上下文切换的时间）
>* `real`为总共花掉的时间。
>* `real-sys-user`为进程等待内核的时间。

如

```bash
[root@localhost Desktop]# time ls
nohup.out  RunLongSecond.class  RunLongSecond.java  test

real	0m0.004s
user	0m0.000s
sys	    0m0.004s
```

# 待补充

* top命令
* 链接文件
* find命令
* ...
