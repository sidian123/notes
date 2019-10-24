# System

提供了有用的字段和方法, 一个helper类

* 与输入输出相关的字段
* 访问属性和环境变量的方法
* 加载文件和库的方法
* 数组的快速拷贝
* ...

# 进程相关

## Process

[ProcessBuilder.start()](https://docs.oracle.com/javase/8/docs/api/java/lang/ProcessBuilder.html#start--)和[Runtime.exec](https://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#exec-java.lang.String:A-java.lang.String:A-java.io.File-)方法将创建本地进程, 并返回`Process`实例. 

> 该方法只适合创建命令行进程

通过该实例, 可以控制进程, 如

*  获取进程的输入输出流
* 等待进程执行完毕
* 核查进程状态
* 销毁进程等

未及时向进程输入数据或输出数据会造成进程被阻塞.

进程的状态一般不受Java进程影响, 两则是并行执行的, 即使Java结束了, 没有显示销毁被创建的进程, 它就不会被杀死.

## Runtime

代表Java程序的运行环境, 通过它可启动新进程并执行, 即执行命令.

最基础的方法:

- 执行命令： 传入环境变量集合`envp`和指定工作目录`dir`, 将执行命令`cmdarray`.

  ```java
  public Process exec(String[] cmdarray, String[] envp, File dir);
  ```

  - `cmdarray`: 命令中以空格分隔的每个部分
  - `envp`: 环境变量集合, `null`时继承该Java程序的环境变量
  - `dir`工作目录, `null`时继承该Java程序的工作目录

  > 一般使用该方法的其他变种

## 弃坑

限制终究是太多了, 见[Why does Runtime.exec(String) work for some but not all commands?](https://stackoverflow.com/questions/31776546/why-does-runtime-execstring-work-for-some-but-not-all-commands)

# ThreadLocal

- 介绍

  `ThreadLocal`类型的字段是线程局部的, 意味着即使是类变量, 在不同的线程中也有不同的拷贝.

- 使用

  `ThreadLocal`字段通常声明为`private static`, 然后重写它的`initialValue`方法来设置它的初始值. 而`get`, `set`方法用于设置或获得该字段的值.

- 原理

  当线程首次调用`get`方法时, 会执行它的`initialValue`方法来初始化, 并返回该值. 之后的`get`调用仅仅只是获取该值.

  当线程die并且无其他引用时, 该变量go die, 并交由垃圾收集器处理.

> 参考: [ThreadLocal<T>](https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html)

# 平台环境

## System Properties

Jvm会维护所处环境的信息, 称之为系统属性, 可通过`System`类获取.

比较重要的系统属性如下

| Key                 | Meaning                                                      |
| ------------------- | :----------------------------------------------------------- |
| `"file.separator"`  | Character that separates components of a file path. This is "`/`" on UNIX and "`\`" on Windows. |
| `"java.class.path"` | Path used to find directories and JAR archives containing class files. Elements of the class path are separated by a platform-specific character specified in the `path.separator` property. |
| `"java.home"`       | Installation directory for Java Runtime Environment (JRE)    |
| `"java.vendor"`     | JRE vendor name                                              |
| `"java.vendor.url"` | JRE vendor URL                                               |
| `"java.version"`    | JRE version number                                           |
| `"line.separator"`  | Sequence used by operating system to separate lines in text files |
| `"os.arch"`         | Operating system architecture                                |
| `"os.name"`         | Operating system name                                        |
| `"os.version"`      | Operating system version                                     |
| `"path.separator"`  | Path separator character used in `java.class.path`           |
| `"user.dir"`        | User working directory                                       |
| `"user.home"`       | User home directory                                          |
| `"user.name"`       | User account name                                            |

# UUID

`java.util.UUID`代表不变的全局唯一标识符, UUID由128位标识.

UUID存在不同的变体( variant ), 不管哪种变体, 都有4中版本, 而JDK提供了 Leach-Salz 类型的UUID.

方法:

*  **[randomUUID](https://docs.oracle.com/javase/8/docs/api/java/util/UUID.html#randomUUID--)**() 获取`v4` 版的uuid
* ...

> 参考[UUID](https://docs.oracle.com/javase/8/docs/api/java/util/UUID.html?is-external=true)

# 参考

* [java.lang](https://docs.oracle.com/javase/8/docs/api/)

