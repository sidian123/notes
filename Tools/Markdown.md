[TOC]

# 一、介绍
Markdown是一种轻量级标记语言，它以纯文本形式(易读、易写、易更改)编写文档，并最终以HTML格式发布。

Markdown也可以理解为将以MARKDOWN语法编写的语言转换成HTML内容的工具。

markdown语法由一些标点符号组成，同时可以嵌套html元素（但不支持style属性）。因此一些markdown不能表现的内容可以通过html来实现。但是不支持javascript和css（有的编辑器支持）。如果自己使用的标点符号可markdown语法冲突了，可以通过转义将之转化为相应字面值，一般情况下markdown编辑器自己会处理的很好。

但是不同的Markdown编辑器对对应语法的解析效果不尽相同，因此在使用一种markdown编辑器时最好参考它的文档说明。所以下面记录的markdown语法也不一定对所有markdown编辑器有效。**注意**，markdown编辑器一般都会扩展markdown的功能。

# 二、基础语法
所谓基础语法，就是所有markdown编辑器几乎都支持的语法。

html所有可视元素可以大致分为block和inline元素。因此markdown也最终表现为这两种形式，但是接下来是通过功能来展开介绍的。

**注意!!!**，一般markdown语法中的标号通常需要使用空格和文本隔开。block块元素上下最好有一空行。markdown编辑器中多个空格或多行，一般会被当作一个空格或一行处理。不像html，markdown是可以自动转化`<`和`&`为对应实体的。

## 2.1、标题
有两种形式。
（1）使用`=`和`-`标记一级和二级标题。
```
一级标题
=======
二级标题
------------
```
（2）使用#，可以表示1-6级标题。
```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```
## 2.2、段落
段落前后要有空行，最终转化为html中的`<p>`元素。

段内可强制换行，行末使用**两个**以上空格+回车。但是目前很多编辑器支持回车换行。最终转化为html中的`<br/>`元素。

## 2.3、区块引用
在段落的每行或者只在第一行使用符号`>`,还可使用多个嵌套引用。对应html中的`<blockquote>`元素。
```
> 区块应用
> >嵌套引用
```
**注意**，还有其他功能，可参考2.6列表。

## 2.4、代码块
代码块有三种，一种是block类型的代码块；一种是inline类型的代码块，可以插入文本中；一种是许多编辑器**扩展**的代码块，提供了语法高亮。
### 2.4.1、code blocks
每行前面都加上四个空行或一个tab。最终会转化为`<pre><code>内容在此处</pre></code>`，因此代码块的格式会被保留。
```
注意前面的四个空格
	void main()
	{
		printf("Hello, Markdown.");
	}
```
### 2.4.2、inline code
使用`` ` ``将代码围住，对应html中的`<code>`元素，因此多余空格换行会被转化。代码中的`<`和`&`会被转化为html实体，因此代码中的html元素不会起作用。
```
一般情况下
` code `
如果code中含有` 则
`` aaa`bbb ``
```

### 2.4.3、扩展代码块
由于是编辑器扩展的，也许就有的就不支持，所以这里只提一下：

	```[代码类型]
		code in here
	```
**注意**，代码中是不能包含同时含有三个`` ` ``的，此时应该使用markdown自身的代码块（2.4.1小结所示）。

## 2.5、强调
在强调内容两侧分别加上`*`或者`_`，如：
> \**斜体*\* \__斜体_\_
> \*\***粗体**\*\* \_\___粗体__\_\_

## 2.6、列表
列表有有序和无序的列表。无序列表可以使用`*`,`+`,`-` 作为list标记；有序列表使用：数字+`.`+空格的形式作为list标记。

列表中每一列都可以有多个块类元素，如段落，代码块（此时需要两个tab键）等等。不过要注意对齐。而段落可对齐可不对齐，最好对齐。
```
无序
* bird
*  magic

有序
1. bird
2. magic

多段落
1. i am a paragraph 
	i am another paragraph
2. aaaaaa

含有代码块
1. 接下来是代码，需要对齐后，另起一个段落，空两个tab键

			code in here
2. aaaaa
```
## 2.7、分割线
分割线最常使用就是三个或以上`*`，还可以使用`-`和`_`。

## 2.8、链接
两种形式：**行内式**和**参考式**。但是还有一种简单**自动链接**。
### 2.8.1、行内式
语法如下，分别为链接显示内容，url（可相对地址）和可选的title属性。
```
[element context](url  "optional title")
示例：
[百度](http:www.baidu.com "baidu")
或
[百度](http:www.baidu.com)
```
### 2.8.2、参考式
语法如下，分为两部分，一部分给出显示内容，指定id（如果没有，则默认context）；第二部分可以写在任何地方，包含id、url（可相对地址）和可选的title属性。
```
[context][id]
...
[id]:url "optional title"
示例
[baidu][]
...
[baidu]:http://www.baidu.com "baidu"
```
### 2.8.3、自动链接
这是一种简便的方法，地址和显示内容都为address：
```
<url>
示例：
<http://www.baidu.com/>
```
*注意，目前大多编辑器都可以直接检测链接，不需要尖括号*

## 2.9、图片
与链接类似，就是在链接的基础上加个`!`。下个直接给出语法，不给示例了。
```
行内式
![alt text](url "optional title")
参考式
![alt text][id]
...
[id]: url  "optional title"
```
**注意**，csdn支持设置图片大小，可自行查阅相关资料。

## 2.10、转义
有时需要用到一些标点符号，但是markdown中有特殊含义，此时可以通过反斜杠`\`来转义成普通字符。

markdown可以转义的字符有：

	\   backslash
	`   backtick
	*   asterisk
	_   underscore
	{}  curly braces
	[]  square brackets
	()  parentheses
	#   hash mark
	+   plus sign
	-   minus sign (hyphen)
	.   dot
	!   exclamation mark

# 三、扩展语法
每个编辑器都会对markdown扩展自己的语法，因此这里的语法不一定在所有编辑器中使用。在使用不同编辑器时最好先浏览一下相应的文档。

一般来说，大部分都扩展了代码块高亮、流程图、甘特图、uml、数学公式、目录之类的功能，甚至还有快捷键的使用，极其方便。需要用到这些功能时，则更要仔细阅读它的文档了。

**注意**，在这里，不会完整将它们记录下来，在博主用到时再记录，，因此此时内容不多。。

## 3.1、目录
csdn的目录：
```markdown
@[toc](可选的目录)
```

----------

typora的目录

```markdown
[toc]
```

## 3.2、扩展代码块

上面已经提到过了，可以让代码高亮，但是需要知道该编辑器支持什么代码高亮。

语法：

	```[代码类型]
		code in here
	```
	如：
	```java
		public class A{
			private int a;
			public A(){
				System.out.println(this.a);
			}
		}
	```

效果：
```java
		public class A{
			private int a;
			public A(){
				System.out.println(this.a);
			}
		}
```

**注意**，代码中是不能包含同时含有三个`` ` ``的，此时应该使用markdown自身的代码块（2.4.1小结所示）。

代码类型参考：[markdown 语法高亮支持](https://blog.csdn.net/qq_32126633/article/details/78838494#t1)

## 3.3、表格
一般为如下形式，`-`分隔表头，`|`分隔列。`:`可以对齐整列：
```markdown
默认样式
header 1 | header 2
---|---
row 1 col 1 | row 1 col 2
row 2 col 1 | row 2 col 2
第二列左对齐
header 1 | header 2
---|:---
row 1 col 1 | row 1 col 2
row 2 col 1 | row 2 col 2
第二列右对齐
header 1 | header 2
---|---:
row 1 col 1 | row 1 col 2
row 2 col 1 | row 2 col 2
居中对齐
header 1 | header 2
---|:---:
row 1 col 1 | row 1 col 2
row 2 col 1 | row 2 col 2
```
工具：[将html表格转化为markdown标记](https://jmalarcon.github.io/markdowntables/)

## 3.4、任务列表
表示该任务完成或者未完成：
```markdown
- [x] 完成
- [ ] 未完成
```
效果如下：
- [x] 完成
- [ ] 未完成

## 3.5、注脚
```markdown
[^number]
...
[^number]:text
```

# 参考
介绍最为详细：
<https://daringfireball.net/projects/markdown/syntax>
中文版翻译：
<http://www.markdown.cn/#overview>
简略版：
<https://github.com/younghz/Markdown#%E6%80%8E%E4%B9%88%E4%BD%BF%E7%94%A8>
<https://markdown-guide.readthedocs.io/en/latest/basics.html>
github的文档：
<https://guides.github.com/features/mastering-markdown/>
可用在线编辑器：
<https://dillinger.io/>
<https://stackedit.io/editor#welcome-to-stackedit>
csdn教程：
<https://blog.csdn.net/cacacai/article/details/80003498>