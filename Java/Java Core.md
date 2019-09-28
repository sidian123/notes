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

# 参考

* [java.lang](https://docs.oracle.com/javase/8/docs/api/)