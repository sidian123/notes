# 可变长参数Varargs

- 可变长参数可以接受0个或多个参数，在方法体中被当做数组使用。
- 方法定义中，只能存在一个可变长参数，且位于最后一个参数。

例子：

```java
 int nums(int a, float b, double … c)
```

> 参考：[Variable Arguments (Varargs) in Java](<https://www.geeksforgeeks.org/variable-arguments-varargs-in-java/>)

# JVM hot swapping

JVM提供了热置换的功能，即在程序已运行在debug模式，在程序运行时置换程序部分代码。但有一定限制，如只能热更新方法体的内容。

> 参考：[JVM hot swapping](<https://stackoverflow.com/questions/13564785/jvm-hot-swapping>)

# 系统属性

- 运行java是的`-D`参数，如`-Dname=luo`

# Module

## 优点

- 由于JAVA API的package越来越大，直接引入包后，打包后的文件会异常的大。而使用Module，只会打包你需要的module，**使发布文件更小**。

  > 具体为啥，暂不知

- 封装：module含多个package，但允许在被其他module使用时只暴露部分package。

- module的启动检测：Java 9后，程序必须被打包成module。当程序启动时，会检查依赖的module，如果丢失则停止运行。Java 9之前，只有当程序使用到该package、class后才报道错误。

## 基础

- 一个module可含有多个package

- module命名：与package命名一样，但java 9后`_`作为了保留字，因此不要使用。

  > 话说包名命令规则啥？
  >
  > 建议module名与根package名一致，除了一个module含多个package的情况外。

- module根目录：一个module会被编译成一个module根目录，不会出现子目录。如，当一个`com.module`模块含有`com.package1`和`com.package2`时，编译后的目录结构为：

  - com.module
    - com
      - package1
      - package2

- module-info.java：用于描述哪个package被导出（**export**），自身需要（**require**）什么module。编译后位于module根目录下。

- 。。。

- 未命名module：为了兼容，未使用的class或jar位于未命令模块中。未命令模块导出所有package，但已命令模块不用读取。未命令模块可以require模块导出的package。

# 网络

## overview

- TCP：提供可靠的、点对点的连接；接收到的数据与发送时的顺序一致

- UDP：非可靠（即数据可能丢失）；数据接收的顺序不被保证，相互独立

  > UDP对IP协议进行了简单的封装，因此速度比较快；TCP为了可靠性，牺牲了一定速度。
  >
  > 所谓可靠，就是丢包就重发

- ip地址确定host，端口确定应用。

- `java.net`

  - TCP相关：`URL`, `URLConnection`, `Socket`, and `ServerSocket` 
  - UDP相关：`DatagramPacket`, `DatagramSocket`, and `MulticastSocket`

## URL

- 定义：指向网络上一个资源的地址。主要由两部分组成，**协议**和**地址**，由`://`分隔。协议定义了获取资源的方式，地址还可细分，如何细分取决于所用协议。但大多数协议的地址包含如下成分：
  - Host Name：主机名字，即域名
  - Filename：主机内指向文件的路径
  - Port Number（可选）：主机内提供服务的应用对应的端口号
  - Reference（可选）：定位资源内某个部分的引用（锚），如http中的`#anchor`
- [URL](<https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URL.html>)创建
  - 绝对地址：`URL myURL = new URL("http://example.com/");`
  - 相对地址：`URL page1URL = new URL(myURL, "page1.html");`由第一个参数给出基址。
  - URL传入String时，注意转义，但在分成分传参构建URL的构造函数中，会自动转义。
- URL解析：含有解析URL的方法，并不是所有协议都支持这些方法。
- 直接读取：`openStream()`，内部通过`openConnection().getInputStream()`实现。
- `URLConnection`：`openConnection` 返回`URLConnection` ，该类表示一个具体的连接，常用为http协议，然后强制转化类型为[HttpURLConnection](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/HttpURLConnection.html)。
  - 得到`URLConnection` 后，可以设置一些请求参数，然后执行`connect`方法，这时才真正的创建连接。
  - 一些需要连接的操作会隐式连接，不用执行`connect`，如`getInputStream`、`getOutputStream`
  - 默认不允许获得输出流，需打开：`setDoOutput(true);`

# ThreadLocal

* 介绍

  `ThreadLocal`类型的字段是线程局部的, 意味着即使是类变量, 在不同的线程中也有不同的拷贝.

* 使用

  `ThreadLocal`字段通常声明为`private static`, 然后重写它的`initialValue`方法来设置它的初始值. 而`get`, `set`方法用于设置或获得该字段的值.

* 原理

  当线程首次调用`get`方法时, 会执行它的`initialValue`方法来初始化, 并返回该值. 之后的`get`调用仅仅只是获取该值.

  当线程die并且无其他引用时, 该变量go die, 并交由垃圾收集器处理.

> 参考: [ThreadLocal<T>](https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html)









