# Apache Commons

## IO

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

### Utility

* [IOUtils](http://commons.apache.org/proper/commons-io/javadocs/api-release/index.html?overview-summary.html) 提供输入输出的便利方法, 主要分为四类

  * `toXxx/read` 从流中读取数据

  * `write` 将数据写入流中
  * `copy` 从一个流中拷贝数据到另一个流中
  * `contentEquals ` 比较两个流中内容是否一致.
  * 关闭流? 不建议用, 请用` try-with-resources`语法

  在*byte-to-char*和*char-to-byte*的方法中, 涉及编码转化, 因此该类方法都提供了设置编码的重载方法.

* [FileUtils](http://commons.apache.org/proper/commons-io/javadocs/api-release/index.html?org/apache/commons/io/FileUtils.html) 文件操作的便利方法
  - writing to a file 
  - reading from a file 
  - make a directory including parent directories 
  - copying files and directories 
  - deleting files and directories 
  - converting to and from a URL 
  - listing files and directories by filter and extension 
  - comparing file content 
  - file last changed date 
  - calculating a checksum 

* [FilenameUtils](http://commons.apache.org/proper/commons-io/javadocs/api-release/index.html?org/apache/commons/io/FilenameUtils.html) 操作文件名和文件路径的便利方法, 主打解决平台差异问题, 个人感觉使用`Path`比较好.

### Filefilter

[filefilter](http://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/filefilter/package-summary.html) 包下提供了很多现成可用的组件, 用于过滤文件目录.

同时, 提供了工具类`FileFilterUtils`, 用于组合不同过滤器以供使用. 部分方法如下:

* `and()` 通过逻辑与, 组合不同过滤器
* `or()` 通过逻辑或, 组合不同过滤器.

### Monitor

监控文件系统事件, 并通知监听器.

用法

* 创建**事件处理器** [FileAlterationListener](http://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/monitor/FileAlterationListener.html)
* [FileAlterationObserver](http://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/monitor/FileAlterationObserver.html) **观察者**, 用于观察事件的发生, 事件监听器在此注册.
* 手动调用`FileAlterationObserver`的方法, 判断事件是否发生; 或者将观察者注册到**监控器** [FileAlterationMonitor](http://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/monitor/FileAlterationMonitor.html) 中, 其会创建线程, 并以一定频率自动执行观察者逻辑.

基本用法

1. 创建观察者, 传入观察目录, 和事件处理器

   ```java
   File directory = new File(new File("."), "src");
   FileAlterationObserver observer = new FileAlterationObserver(directory);
   observer.addListener(...);
   observer.addListener(...);
   ```

2. 手动观察是否发生事件

   ```java
   // initialize
   observer.init();
   ...
   // invoke as required
   observer.checkAndNotify();
   ...
   observer.checkAndNotify();
   ...
   // finished
   observer.finish();
   ```

3. 或者, 注册到监控器中

   ```java
   long interval = ...
       FileAlterationMonitor monitor = new FileAlterationMonitor(interval);
   monitor.addObserver(observer);
   monitor.start();
   ...
       monitor.stop();
   ```

4. 若只观察指定目录下部分内容, 可用过滤器. 如, 仅观察`src`目录下非隐藏目录和有`.java`后缀的文件

   ```java
   // Create a FileFilter
   IOFileFilter directories = FileFilterUtils.and(
       FileFilterUtils.directoryFileFilter(),
       HiddenFileFilter.VISIBLE);
   IOFileFilter files       = FileFilterUtils.and(
       FileFilterUtils.fileFileFilter(),
       FileFilterUtils.suffixFileFilter(".java"));
   IOFileFilter filter = FileFilterUtils.or(directories, files);
   
   // Create the File system observer and register File Listeners
   FileAlterationObserver observer = new FileAlterationObserver(new File("src"), filter);
   observer.addListener(...);
   observer.addListener(...);
   ```

## Exec

### 依赖引入

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-exec</artifactId>
    <version>1.3</version>
</dependency>
```

### 使用

整个库的用法围绕着`DefaultExecutor`展开

* 执行命令

  `execute()`执行命令. 参数有

  * `CommandLine`

    要执行的命令; 

  * `ExecuteResultHandler`

    用于处理进程结束的结果. 该参数存在时`execute()`**异步**执行, 否则**同步**执行.

    * 异步执行

      命令正常结束, 会调用`DefaultExecuteResultHandler.onProcessComplete()`方法; 异常结束, 会调用`DefaultExecuteResultHandler.onProcessFailed()`方法

      > 注意, 1) 覆盖父类方法时, 最好调用下其父方法. 2) 被看门狗杀死时, 并不会触发该处理器执行.

    * 同步执行

      正常结束后, 会返回状态码, 否则抛出异常

* 工作目录和环境变量
  
  `setWorkingDirectory()`设置工作目录, `execute()`的第三个参数设置环境变量, 默认为父进程的工作目录和环境变量.
  
* 看门狗
  
  `setWatchdog()`设置看门狗, 防止子进程长时间运行. 

  有两种使用方式:
  
  * 设置有限的超时时间, 超时时让其自动关闭进程
  * 设置无限的超时时间`ExecuteWatchdog.INFINITE_TIMEOUT`, 然后手动调用`destroyProcess()`关闭进程.
  
* 输入输出

  `setStreamHandler()`获取子进程的输入和输出

* 期待的结束状态码

  `setExitValue()`设置期待的结束状态码, 若子进程返回其他值时, 将抛出异常. 默认0.

* 进程处理器

  主进程销毁时的回调, 通过`setProcessDestroyer()`实现. 

  其中`ShutdownHookProcessDestroyer`可在VM退出时自动关闭所有子进程.

> 参考[API](https://commons.apache.org/proper/commons-exec/apidocs/index.html)

## BeanUtils

### 依赖

```xml
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.4</version>
</dependency>
```

### JavaBean约定

* 必须是`public`类, 且提供`public`的无参构造函数

* Bean的配置都是通过修改**属性 properties ** 完成

* 每个属性都有*getter* **或** *setter*方法, 方法名以`get`或`set`为前缀, 接着属性名, 但首字母大写

  > 其中, 基本类型`boolean`的属性特殊, 其*getter*方法以`is`为前缀

* 属性不必一定存在对应字段.

### JavaBean例子

```java
public class Employee {
    public Address getAddress(String type);
    public void setAddress(String type, Address address);
    public Employee getSubordinate(int index);
    public void setSubordinate(int index, Employee subordinate);
    public String getFirstName();
    public void setFirstName(String firstName);
    public String getLastName();
    public void setLastName(String lastName);
}
```

### 属性类型

> 以下, 除了Simple类型属性, 其他属性几乎不多见.

* **Simple**

  属性只有一个值

  ```java
  public String getFirstName();
  public void setFirstName(String firstName);
  public String getLastName();
  public void setLastName(String lastName);
  ```

  操作方法

  ```java
  PropertyUtils.getSimpleProperty(Object, String)
  PropertyUtils.setSimpleProperty(Object, String, Object)
  ```

  例子

  ```java
  Employee employee = ...;
  String firstName = (String)
      PropertyUtils.getSimpleProperty(employee, "firstName");
  String lastName = (String)
      PropertyUtils.getSimpleProperty(employee, "lastName");
  ... manipulate the values ...
      PropertyUtils.setSimpleProperty(employee, "firstName", firstName);
  PropertyUtils.setSimpleProperty(employee, "lastName", lastName);
  ```

* **Indexed**

  有多个值, 可通过索引操作

  ```java
  public Employee getSubordinate(int index);
  public void setSubordinate(int index, Employee subordinate);
  ```

  操作方法

  ```java
  PropertyUtils.getIndexedProperty(Object, String)
      PropertyUtils.getIndexedProperty(Object, String, int)
      PropertyUtils.setIndexedProperty(Object, String, Object)
      PropertyUtils.setIndexedProperty(Object, String, int, Object)
  ```

  例子

  ```java
  Employee employee = ...;
  int index = ...;
  String name = "subordinate[" + index + "]";
  Employee subordinate = (Employee)
      PropertyUtils.getIndexedProperty(employee, name);
  
  Employee employee = ...;
  int index = ...;
  Employee subordinate = (Employee)
      PropertyUtils.getIndexedProperty(employee, "subordinate", index);
  ```

* **Mapped**

  有多个值, 通过键值操作

  ```java
  public Address getAddress(String type);
  public void setAddress(String type, Address address);
  ```

  操作方法

  ```java
  PropertyUtils.getMappedProperty(Object, String)
  PropertyUtils.getMappedProperty(Object, String, String)
  PropertyUtils.setMappedProperty(Object, String, Object)
  PropertyUtils.setMappedProperty(Object, String, String, Object)
  ```

  例子

  ```java
  Employee employee = ...;
  Address address = ...;
  PropertyUtils.setMappedProperty(employee, "address(home)", address);
  
  Employee employee = ...;
  Address address = ...;
  PropertyUtils.setMappedProperty(employee, "address", "home", address);
  ```

### 使用

主要使用`PropertyUtils`和`BeanUtils`

* `PropertyUtils`

  * 属性操作

    * 获取属性值

      `getXXXProperty()`

    * 设置属性值

      `setXXXProperty()`

  * 获得属性

    * 所有属性

      `getPropertyDescriptors()`

    * 单个属性

      `getPropertyDescriptor()`

  * 反射属性方法

    * 读

      `getReadMethod()`

    * 写

      `getWriteMethod()`

  * 判断属性类型

    * 可读

      `isReadable()`

    * 可写

      `isWriteable()`

* `BeanUtils`

  * 克隆

    `cloneBean()`

  * 拷贝

    `copyProperties()`

> `BeanUtils`与`PropertyUtils`很多功能都相同, 上述我仅根据语义罗列了部分方法.

> 参考:
>
> * [Commons BeanUtils](https://commons.apache.org/proper/commons-beanutils/)
> * [User Guide](https://commons.apache.org/proper/commons-beanutils/)
> * [PropertyUtils](http://commons.apache.org/proper/commons-beanutils/javadocs/v1.9.4/apidocs/org/apache/commons/beanutils/PropertyUtils.html)
> * [BeanUtils](http://commons.apache.org/proper/commons-beanutils/javadocs/v1.9.4/apidocs/org/apache/commons/beanutils/BeanUtils.html)

# Guava

Google提供的一个工具, 先记录下. 一般引入了Swagger的项目都引入Guava.

> 参考
>
> * [Javadoc](https://guava.dev/releases/snapshot-jre/api/docs/)
> * [wiki](https://github.com/google/guava/wiki)

# Hutool (神器)

> [Hutool Doc](https://hutool.cn/docs/#/)

## ArrayUtil

* 包装类数组 --> 基本类型数组 `unWrap()`

# 原生工具

## 进程相关

### Process

[ProcessBuilder.start()](https://docs.oracle.com/javase/8/docs/api/java/lang/ProcessBuilder.html#start--)和[Runtime.exec](https://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#exec-java.lang.String:A-java.lang.String:A-java.io.File-)方法将创建本地进程, 并返回`Process`实例. 

> 该方法只适合创建命令行进程

通过该实例, 可以控制进程, 如

*  获取进程的输入输出流
* 等待进程执行完毕
* 核查进程状态
* 销毁进程等

未及时向进程输入数据或输出数据会造成进程被阻塞.

进程的状态一般不受Java进程影响, 两则是并行执行的, 即使Java结束了, 没有显示销毁被创建的进程, 它就不会被杀死.

### Runtime

代表Java程序的运行环境, 通过它可启动新进程并执行, 即执行命令.

> 注意, 只能执行PATH下的命令, 非PATH下脚本的执行都是通过解析器执行的 ,而解析器在PATH下.

最基础的方法:

* `exec()`

  执行命令, 传入环境变量集合`envp`和指定工作目录`dir`, 将执行命令`cmdarray`.

  ```java
  public Process exec(String[] cmdarray, String[] envp, File dir);
  ```

  - `cmdarray`: 命令中以空格分隔的每个部分
  - `envp`: 环境变量集合, `null`时继承该Java程序的环境变量
  - `dir`工作目录, `null`时继承该Java程序的工作目录

  > 一般使用该方法的其他变种

* `addShutdownHook()`

  注册JVM关闭回调

### System

提供了有用的字段和方法, 一个helper类

* 与输入输出相关的字段
* 访问属性和环境变量的方法
* 加载文件和库的方法
* 数组的快速拷贝
* 获取`logger`
* ...

## 平台环境

### System Properties

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

命令行指定系统属性的方法如下:

```
java -Dname="Spring" -jar app.jar
```

## 安全

* `MessageDigest` 生成摘要的工具

## System Tray

* 用于控制系统托盘. 

  [SystemTray](https://docs.oracle.com/javase/8/docs/api/java/awt/SystemTray.html) 代表系统托盘, 可以有多个托盘图标[TrayIcon](https://docs.oracle.com/javase/8/docs/api/java/awt/TrayIcon.html), 每个托盘图标都可有一个弹出菜单[PopupMenu](https://docs.oracle.com/javase/8/docs/api/java/awt/PopupMenu.html)

* `SystemTray`

  * `isSupported()` 系统是否支持该功能
  * `getSystemTray()` 获取系统托盘对象
  * `add()` 添加托盘图标
  * `remove()` 删除托盘图标

* `TrayIcon`

  * `setImageAutoSize(true)` 设置自动调整图标大小以适应当前平台的托盘图标显示

  * `setImage(Image image)` 根据需要可在随时改变显示的图标
  * `setPopupMenu(PopupMenu popup)` 根据需要可随时改变点击时的弹出菜单
  * `addActionListener(ActionListener listener)` 添加托盘图标的动作监听器（鼠标右键的点击监听）
  * `addMouseListener(MouseListener listener)` 添加托盘图标的鼠标监听器（包括鼠标所有按键的监听）
  * `add(trayIcon)` 添加托盘图标到系统托盘（一个应用程序可添加多个托盘图标）
  * `remove(trayIcon)` 从系统托盘移除图标

* `PopupMenu`

  ...

> 在Spring boot应用中, 默认不支持AWT的使用, 即托盘功能不可用, 需修改启动配置, 如
>
> ```java
>         SpringApplication springApplication = new SpringApplication(ClientApplication.class);
>         springApplication.setHeadless(false);//允许使用awt,即界面
>         springApplication.run(args);
> ```

> 参考
>
> * [How to Use the System Tray](https://docs.oracle.com/javase/tutorial/uiswing/misc/systemtray.html)
> * [Java SystemTray类（系统托盘）和TrayIcon类（托盘图标）](https://blog.csdn.net/qq_36761831/article/details/81516535)

# Socket

* Socket定义

  套接字(Socket)是网络上运行的两个程序之间的双向通信链路的一个端点。

  套接字会绑定到端口上, 让数据能发送到正确的端口上.

* 交互

  Server监听本机的一个端口, 当Client通过该端口与Server成功建立连接后, Server和Client都会产生Socket, 用于交互. 

  Client的Socket临时会绑定到一个端口上 ( 由系统分配 ), 此时该端口不可被其他程序使用. Server则不同, 一个端口可建立无数个连接, 即仍会监听端口, 继续产生新的Socket.

* 基本使用

  Server监听端口, `accpet()`会阻塞, 直到建立一个socket连接

  ```java
  Socket clientSocket = serverSocket.accept();  
  ```

  之后Server和Client都可通过`Socket`通信.

  * `Socket.getOutputStream()` 获取输出流
  * `Socket.getInputStream()` 获取输入流

  在读取过程中, 若对方未返回数据, 会被堵塞.
  
* `Socket.close()` vs. `Socket.shutdownOutput()` vs. `InputStream.close()`或`OutputSream close()`

  `close()`会关闭socket和输入输出流, 此时另一端读取会返回`-1`, 即EOF

  `shutdownOutput()` 关闭输出流, 另一端将读取到EOF. 但是不影响输入流的读取.

  调用`InputStream.close()`或`outputStream.close()`在关闭流的同时, 也会关闭socket.

> 参考[oracle socket tutorial](https://docs.oracle.com/javase/tutorial/networking/sockets/index.html)

# 其他

## java vs. javaw

javaw与java一致, 除了javaw运行程序时不会依附终端, 且执行后立即返回

> 参考[Difference between java and javaw](https://stackoverflow.com/questions/12129505/difference-between-java-and-javaw)

## function

`java.util.function`下提供了很多用于函数式编程的接口, 大致分类

* `Consumer`接收参数并处理, 消费性的接口
* `Function` 处理参数并返回结果, 功能性的接口
* `Predicate` 接收参数并返回`boolean`值, 预测性的接口
* ...

> 参考[java.util.function](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-tree.html)

## UUID

`java.util.UUID`代表不变的全局唯一标识符, UUID由128位标识.

UUID存在不同的变体( variant ), 不管哪种变体, 都有4中版本, 而JDK提供了 Leach-Salz 类型的UUID.

方法:

*  **[randomUUID](https://docs.oracle.com/javase/8/docs/api/java/util/UUID.html#randomUUID--)**() 获取`v4` 版的uuid
*  ...

> 参考[UUID](https://docs.oracle.com/javase/8/docs/api/java/util/UUID.html?is-external=true)

## 远程调试

一张图诠释如何使用

![image-20191220173010738](.Java%20Core/image-20191220173010738.png)

有两种使用方式

* `suspend=n` JVM启动后不暂停, 之后Idea可随时连接上
* `suspend=y` JVM启动后暂停, Idea远程连接后才继续执行下去.

## Optional

值的容器, 常用在方法的返回值上. 常用方法如下

* `isPresent()` 若值存在, 则`true`

* `orElse(other)`

  若值不为空`null`, 则返回; 否则返回默认值`other`

* `ifPresent(action)`

  若值存在, 则执行动作`action`