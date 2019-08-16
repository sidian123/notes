[TOC]

# 一 介绍

## 历史

Java语言最初由Sun MicroSystems公司开发, Sun于2010年由Oracle公司收购, Java最终归为Oracle所有. 随着Java的不断进化, Java不再仅仅是语言, 还包括保证大量标准的,开源的API和工具, 可称之为Java平台.

xxxx年, oracle宣布收费, 只是它自己的JVM实现收费罢了, 可以使用其他免费的JVM实现, 如OpenJDK的JVM实现.

## Java概念

Java如今不只是Java语言, 而是一个平台, 涉及很多parts, 这里一一介绍:

* Java语言

  Java语言, 存放在以`.java`为后缀的文件中.

  > 有Java语言, 必定存在对应的编译器.

* Java字节码

  Java文件被编译后将生成二进制字节码文件, 以`.class`为后缀.

  > Java字节码是低级语言, 类似与汇编. 现在出现了很多高级语言, 最终编译成Java字节码, 如kotlin, groovy等.

* Java虚拟机(JVM)

  解析并运行Java字节码的程序, 不同的JVM实现对Java程序的性能有极大的影响.

  > 正是因此而被人诟病Java运行速度慢. 实际上现在的JVM已经很强大了, 支持JIT(Just In Time), 即运行时智能的将高频运行的代码块编译成机器码并运行它, 极大的提高了程序的运行效率.

  > JVM在不同系统上都有实现, 因此Java程序可以跨平台运行. 
  >
  > JVM不仅要解析与执行字节码, 还要提供系统方面的native API供Java API使用. 但是涉及系统方面的API在不同系统的JVM实现中会表现出部分不一致, 但是总体行为是相似的. 例子如线程的优先级问题.

* Java APIs

  Java提供的类库, 简化开发难度. Java API根据不同的使用领域, 将API划分为三个:

  * Java Card: 运行在小内存设备上

  * Java ME(Micro Edition): 支持移动终端，Java API因此比较精简.

    > 移动端最流行的就是安卓, 不过安卓使用的是部分Java API和它自己提供的API

  * Java SE(Standard Edition): 支持桌面级应用，含有完整的java核心API

  * Java EE(Enterprise Edition): 对Java SE API做了大量扩充，扩展API一般以`javax.*`作为包名。主要用于服务端.

* JRE

  Java运行环境(Java Runtime Environment ), 包含了运行Java环境的必要组件, 如Java SE API, 但没有编译器等.

* JDK

  Java Software Development Kit等于JRE加上编译器和一套其他开发工具.

  > 开发Java程序都需要装JDK

* 应用领域:

  * Java Applets: 运行在浏览器上的Java程序, 目前Go die了, 该领域已被HTML5和JavaScript统治.
  * Java Application Servers: Java EE大部分只提供了规范, 没有具体实现. 实现了这些规范的服务器(Server)能够运行Java程序, 跑Web服务.
  * JavaFX: 用于制作桌面软件, 作为Swing的替代品.
  * 移动领域? 这里没Oracle啥事了, 除了那场与Goolge的官司.

  > Java目前在后端开发中很火热.

# 二 基础

## 安装

从官网上下载JDK, 如果是压缩包, 则解压并配置以下环境变量:

- `PATH`：让Java命令能够在CLI下运行.

- `JAVA_HOME`：约定俗成的一个环境变量，指向JDK安装目录。可选但最好存在, 理由有二:
  - 其一，有些Java程序（如tomcat）在运行时可以通过该变量获得运行时资源，比如Tomcat。
  - 其二（不确定对不对），当存在多个版本的jdk时，无论运行哪个JDK版本的JVM（即设置`PATH`），最终使用的运行环境还是`JAVA_HOME`指向的JDK。
  
- `CLASSPATH`：指向所依赖 Jar库所在地的路径(System ClassLoader会用到该路径)。一般不用给出，默认当前工作目录`.`。如果有，最好也将`.`加入到变量中。在命令行，通过选项`-cp`能够覆盖该`classpath`路径。

  > JDK提供的类由其他类加载器加载, 路径由其他方式给出, 见[类加载器](https://blog.csdn.net/jdbdh/article/details/82593573)

## 编译运行

通常在项目的根目录下, 存在一个源目录(假设`src/`), 存放所有Java源文件, 和一个目的目录(假设`classes/`), 存放所有编译后的class字节码文件.

这里假设Java源文件全在`myfirstapp`包下, 则编译如下:

```bash
javac src/myfirstapp/*.java -d classes
```

* 第一个参数指向Java源文件
* `-d`指向存放生成的字节码的路径. 

> 问题来了, 如果我不止一个包(package)呢? 并且还有子包呢? 我难道要全部列出来才编译? 

然后运行

```bash
java -cp classes myfirstapp.MyJavaApp
```

* `-cp`告诉JVM你所有的class文件在哪. 未给出时默认当前目录, 即`.`
* 最后一个指定含入口方法`main`的类的全限定名, 即包名+类名. 这样JVM才能从classpath路径下找到主类并运行.

> 问题来了, 项目依赖的第三方jar怎么指定? 可以用通配符`*`, 如下面即指定项目编译的class文件和所有三方jar
>
> ```bash
> java -cp classes:/path/to/third-library/* myfistapp.MyJavaApp
> ```

---------------

当项目复杂时, 编译和运行都很复杂, 尤其是编译, 需要列出所有package才能编译. 于是人们将构建过程交给了IDE或构建工具(如Maven)管理, 目前通常都是使用Idea集成Maven的方式来构建项目.

## Java源文件结构

必备要求:

* Java源文件必须以`.java`为后缀
* 文件中的`public`类,接口等类型的声明时, 名字必须与文件名一致.

Java源文件中含有以下元素:

- Package declaration
  - 必须位于第一行
  - 必须与实际的package位置一致
- Import statement**s**
- Type declaration**s**: 必须存在一个`public`类或接口.
  - Fields
  - Class initializers
  - Constructors
  - Methods

## 核心概念

* **变量**: 

  * 存储数据的地方. 
  * 每个变量都对应一个**数据类型**, 数据类型决定了变量可以存储何种类型的数据, 以及可对该数据进行何种**操作**.
  * 每种变量都有它自己的作用域.

  > 变量, 字段, 对象, 这些概念大致相同, 本文可能会混淆的使用这些概念.
  >
  > 但是变量主要侧重于指栈中的变量和堆中对象的字段, 对象则侧重于堆中的对象.
  >
  > 或者说变量就是基本变量+引用变量(指向对象的变量).

* **操作**: 处理变量中数据的操作指令. 可分为两种:

  - Variable operations: 主要用于读写变量
    - Variable assignment of values.
    - Variable reading of values.
    - Variable arithmetic.
    - Object instantiation.
  - Program flow: 主要是通过读取变量数据, 而决定下一条流程执行分支.
    - [`for` loops](http://tutorials.jenkov.com/java/for.html).
    - [`while` loops](http://tutorials.jenkov.com/java/while.html).
    - [`if` statements (branches)](http://tutorials.jenkov.com/java/if.html).
    - [`switch` statements](http://tutorials.jenkov.com/java/switch.html).
    - Method calls: 方法调用本质就是跳转到方法代码块执行.

* **类和对象**: 

  * 类是抽象的模板, 泛指某一类事物, 而对象是具体的实例. 好比人类和具体的某个人. 
  * 类定义了一种数据类型, 它的**字段**声明定义了对象能够存储的数据, 它的**方法**声明定义了对象能够拥有的操作.
  * 对象的概念主要来源于世界
  
* **接口**: 接口与类相比, 接口普通方法不能有实现, 也即接口只是定义了行为, 需要实现类去实现.

* **Packages**: 是用于管理组织有关联的一组类和接口的名字空间, 在文件系统中的体现就是目录.

# 三 语法

## 语言基础

描述语言最基本的特性.

### 变量

#### 声明

变量声明格式如下:

```
VarType varName [=initialValue];
```

#### 分类

变量按照它声明的位置可分为四种

- **Instance Variables(Non-static fields)**: 属于对象的字段, 每个对象都有一份, 只能被对象访问.
- **Class Variable(Static fields)**: 属于类的字段, 每个类只有一份, 可以通过类和对象访问到.
- **Local variables**: 方法内的局部变量, 方法会将它的临时数据存入局部变量. **局部变量必须被初始化后才能使用.**
- **Parameters**: 方法参数, 方法获取输入的途径.

> 实例变量和类变量未初始化时会有默认值, 其中引用变量的默认值为`null`.

从变量是否指向对象可分为

- **基本变量**(primitive variable): 如`int`,`long`
- **引用变量**(reference variable)或**对象引用**(Objecgt references): 如`Integer`,`Long`

#### 命令规则

- 大小写敏感

- 必须以字母,`$`或`_`开始, 之后变量名可含数字

  > 这条是几百年的规则了吧,,, 我用中文名也行

- 不能是保留字

- 变量建议使用名词; 方法使用动词; 常量全大写, `_`分隔; 建议驼峰命名法

#### 局部变量类型推断

局部变量类型推断(JDK10特性), 即当声明局部变量时, 编译器能够从初始值中推断出变量类型. 使用`var`推断类型, 如

```java
var myVar = "A string!";
```

`myVar`被推断为`String`

### 数据类型

#### 介绍

Java是静态类型语言, 意味着声明变量时必须给出数据类型. 数据类型决定了变量可以存储什么样的数据, 和对这些数据进行什么样的操作.

之前说了, 数据类型可分为**基本类型**和**对象引用**. 基本类型变量直接存储数据, 而对象引用存储的是对象的引用或地址.

#### 基本类型

Java语言定义了8中基本类型:

| **Data type** | **Description**                                              |          | 包装类      |
| ------------- | ------------------------------------------------------------ | -------- | ----------- |
| `boolean`     | A binary value of either `true` or `false`                   | 0        | `Boolean`   |
| `byte`        | 8 bit signed value, values from -128 to 127                  | 0        | `Byte`      |
| `short`       | 16 bit signed value, values from <br />-32.768 to 32.767     | 0        | `Short`     |
| `char`        | 16 bit Unicode character                                     | '\u0000' | `Character` |
| `int`         | 32 bit signed value, values from <br />-2.147.483.648 to 2.147.483.647 | 0        | `Integer`   |
| `long`        | 64 bit signed value, values from <br />-9.223.372.036.854.775.808 to 9.223.372.036.854.775.808 | 0L       | `Long`      |
| `float`       | 32 bit floating point value                                  | 0.0f     | `Float`     |
| `double`      | 64 bit floating point value                                  | 0.0d     | `Double`    |

> 默认值是针对字段的, 其中引用变量的默认值为`null`.

> 包装类就是基本类型对应的对象类型, 它们之间可以相互转换(Auto Boxing). 并且包装类是不可变的(immutable)

其他的都是引用类型的变量, 指向堆中的对象. 对象也是有类型的, 即它的类, 你也可以自己定义类, 即定义了新的数据类型.

JDK5后, 基本类型与包装类之间会**自动装箱**(Auto Boxing), 即有必要时, 基本类型会自动转化为包装类, 反之亦然.

#### 字面值

基本类型和字符串是特殊的数据类型, 被内置到语言中, 可以直接使用字面值来创建出来, 如:

```java
boolean result = true;
char capitalC = 'C';
byte b = 100;
short s = 10000;
int i = 100000;
int[] myIntArray = {1, 2, 3};
String[] myStringArray = {"a", "b", "c"};
```

> `String[] myStringArray = new String[]{"a", "b", "c"};`应该不算字面值吧.
>
> **注意**!!! 数组字面值只能在声明时使用 因此
>
> ```java
> method({"aa","bb","cc"});//错误
> method(new String[]{"aa","bb","cc"});//正确
> ```

### 数组

#### 介绍

数组是一个容器对象, 储存固定数量的相同类型变量.

```java
//仅声明
int[] ints;
String[] strs;
String[][] strs2;
//声明时同时初始化
int[] ints2={1,2,3,4};//字面值
int[] ints3=new int[10];//默认初始化为0
String[] strs3={"aa","bb","cc"};
//分两步
int[] ints4=new int[]{1,2,3};//此时不能使用字面值.也不需要制定数量
```

#### 操作

* Array与Collection之间的转化

### 字符串

* 只需字符串加上引号, 字符串就会被自动创建, 如

  ```java
  String s="this is a string";
  ```

* 字符串是不可变的(immutable)

### Operations

#### 变量操作



#### 控制语句

##### if

```
if (<expression>)
	statement
[else
	statement]
```

注意

* `<expression>`要返回`boolean`类型值
* `statement`是一条语句, 多条时可用语句块.
* `else`语句可选
* `if`语句也是`statement`, 因此可以嵌套使用`if`语句

##### switch

```
switch (<expression>){
	case <label> : statements
	case <label> : statements
	...
	default : statements
}
```

* `<expression>`最终值可以为基本类型, 其包装类, 枚举类型, 和`String`

### 表达式,语句,语句块

* 语句块: 由0到多个语句组成,  用`{}`围绕起来. **可以被视作一个语句**

## 类与对象

## 类

## 接口

## 注解

### 介绍

**注解**（annotation），一种元数据，提供一些关于程序的数据，但不是程序的一部分，对程序的执行没有**直接**的影响。

注解有三种用处：

* 提供信息给编译器，指导编译器行为；
* 提供构建信息给构建工具，这些工具有ant、mavent等，构建工具会根据这些注解产生源码或其他文件；
* 运行时提供信息，可以通过反射来获得这些注解信息。

> 通过某些API, 程序本身可以参与构建时期的注解处理.

定义的注解默认继承于`java.lang.annotation.Annotation`接口，因此你可以继承该注解，但是没多大的实际用处。

### 注解声明

```java
@Retention(RetentionPolicy.RUNTIME)
@interface ClassPreamble {
   String author();
   String date();
   int currentRevision() default 1;
   String lastModified() default "N/A";
   String lastModifiedBy() default "N/A";
   // Note use of array
   String[] reviewers();
}
```

注解声明与接口类似，多了个@。注解中的元素声明与接口方法类似，元素也可以赋予默认值，通过default实现。~~注解的元素类型只能是基本类型、数组、字符串、枚举、注解、Class~~。定义该注解时可以被其他**元注解**注释，如上面的@Retention，该注解说明自定义的注解ClassPreamble可以保存到运行时。

### 注解使用

注解一般可以使用在类、方法、字段等java元素上，这是在定义注解时声明的通过Target设置的。如：

```java
@Author(
   name = "Benjamin Franklin",
   date = "3/27/2003"
)
class MyClass() { ... }
```

如果只有一个元素，该元素名字为value的情况下，名字可以忽略：

```java
@SuppressWarnings(value = "unchecked")
void myMethod() { ... }
可以写成如下方式
@SuppressWarnings("unchecked")
void myMethod() { ... }
```


如果有多个元素，其他的有默认值，value元素没有，也可以忽略其他元素，和value名字：

```java
@MyAnnotation("aaa")
class A<T,Y>{
}
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation{
	public String value();
	public String name() default "";
}
```

如果没有元素或者都有默认值，连括号也可以忽略：

```java
@EBook
class MyClass { ... }
```

如果元素是数组，且赋予数组一个值，则不需要大括号：

```java
@MyAnnotation("aaa")
class A<T,Y>{
}
@MyAnnotation({"bbb","ccc"})
class B{
	
}
@MyAnnotation("aaa","bbb")
class C{

}
@interface MyAnnotation{
	String[] value();
	String[] name() default "bbb";
}
```

### 预定义的注解类型

java se api中已经定义了一些注解，可以用于编译器和用于其他注解。

#### 被编译器使用的注解

##### @Deprecated

告诉编译器被注解的元素是被弃用的，使用会被编译器警告。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

可以看出，`@Deprecated`可以被使用到很多地方，不局限于方法、类、字段。可以配合javadoc的@deprecated标签使用，通过该标签解释下为何弃用。

```java
   // Javadoc comment follows
    /**
     * @deprecated
     * explanation of why it was deprecated
     */
    @Deprecated
    static void deprecatedMethod() { }
}
```

##### @Override

通知编译器被注释的方法必须覆盖父类方法。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

##### @SuppressWarnings

用于抑制警告

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

还有其他的如`@SafeVarargs`、`@FunctionalInterface`。

#### 用于其他注解的注解

用于注释其他注解的注解称为元注解。

##### @Retention

指定被标记的注解如何被存储，比如存在源码中、字节码中、运行时中。如果`@Retention`不存在，则默认使用`RetentionPolicy.CLASS`策略，就是存在字节码中。

* `RetentionPolicy.SOURCE` – The marked annotation is retained only in the source level and is ignored by the compiler.

* `RetentionPolicy.CLASS` – The marked annotation is retained by the compiler at compile time, but is ignored by the Java Virtual Machine (JVM).

* `RetentionPolicy.RUNTIME` – The marked annotation is retained by the JVM so it can be used by the runtime environment.

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
/**

- Returns the retention policy.
@return the retention policy
*/
RetentionPolicy value();
}
```

##### @Documented

被`@Document`注释的注解在其他地方被使用时，可以显示在javadoc导出的文档中。

![img](.注解/2018091611492297.png)

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

##### @Target

指定注解可以使用在哪种java元素上：

* `ElementType.ANNOTATION_TYPE` can be applied to an annotation type.
* `ElementType.CONSTRUCTOR` can be applied to a constructor.
* `ElementType.FIELD` can be applied to a field or property.
* `ElementType.LOCAL_VARIABLE` can be applied to a local variable.
* `ElementType.METHOD` can be applied to a method-level annotation.
* `ElementType.PACKAGE` can be applied to a package declaration.
* `ElementType.PARAMETER` can be applied to the parameters of a method.
* `ElementType.TYPE` can be applied to any element of a class.

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

##### @Inherited

指示注解可以被继承。假设被`@Inherited`注释的注解为`@A`，如果父类被`@A`注释，子类便可以继承这个注解。但是并不完全对，因为通过反射，子类不能找到`@A`。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

##### @Repeatable

被`@Repeatable`注解的注解可以在一个地方使用多次，比如下面的`@Schedule`可以使用多次：

```java
@Schedule(dayOfMonth="last")
@Schedule(dayOfWeek="Fri", hour="23")
public void doPeriodicCleanup() { ... }
```

但是由于兼容原因，`repeatable`注解（就是被`@Repeatable`注释过的注解）需要被存入容器注解（container annotation）。也就是说定义`repeatable`注解时还需要定义容器注解。下面声明`repeatable`注解：

```java
@Repeatable(Schedules.class)
public @interface Schedule {
  String dayOfMonth() default "first";
  String dayOfWeek() default "Mon";
  int hour() default 12;
}
```

`@Repeatable`中需要指定容器注解，容器注解的定义为：

```java
public @interface Schedules {
    Schedule[] value();
}
```

注意，容器注解的元素value数组中的类型一定要为repeatable注解类型。很拗口吧，但是既然要存入repeatable注解（也就是Schedule），当然要定义它的数组（Schedule[]）。

### 其他

一些注解可以用于类型的使用上，被称为类型注解（type annotation），通常被用于类型检测。

### 参考

* https://docs.oracle.com/javase/tutorial/java/annotations/index.html

* http://tutorials.jenkov.com/java/annotations.html

* https://www.developer.com/java/other/article.php/10936_3556176_3/An-Introduction-to-Java-Annotations.htm

* https://blog.usejournal.com/how-much-do-you-actually-know-about-annotations-in-java-b999e100b929


# 其他

* 包名必须全小写? 

* `main`方法是运行Java程序的入口, 在启动时需指定含`main`方法的类文件.

  ```java
  package myjavacode;
  
  public class MyClass {
  
      public static void main(String[] args) {
  
      }
  }
  ```

  > 并且入口可以有多个, 但是一次JVM只能运行一个.


# 参考

* [Java Language Tutorial jenkov.com](http://tutorials.jenkov.com/java/index.html)
* [Java Tutorial Oracle.com](https://docs.oracle.com/javase/tutorial/index.html)










