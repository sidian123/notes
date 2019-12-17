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

* 依赖引入

    ```xml
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-exec</artifactId>
        <version>1.3</version>
    </dependency>
    ```

* 使用

    整个库的用法围绕着`DefaultExecutor`展开

    * `execute()`执行命令, 需传入`CommandLine`参数, 指定要执行的命令; `ExecuteResultHandler`用于处理进程结束的结果, 此时`execute()`异步执行, 否则同步.
* 可设置子进程工作目录和环境变量, 默认为父进程的工作目录和环境变量.
    * 可设置看门狗, 防止子进程长时间运行. 有两种使用方式:
      * 设置有限的超时时间, 超时时让其自动关闭进程
      * 设置无限的超时时间`ExecuteWatchdog.INFINITE_TIMEOUT`, 然后手动调用`destroyProcess()`关闭进程.
    * 设置`ExecuteStreamHandler`, 捕获子进程的输入输出
    * 设置期待的结束状态码, 若子进程返回其他值时, 将抛出异常.
    * 设置`ShutdownHookProcessDestroyer`, 可在VM退出时自动关闭所有子进程.

> 参考[DefaultExecutor](https://commons.apache.org/proper/commons-exec/apidocs/index.html)

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

# # 安全

* `MessageDigest` 生成摘要的工具

# 其他

## java vs. javaw

javaw与java一致, 除了javaw运行程序时不会依附终端, 且执行后立即返回

> 参考[Difference between java and javaw](https://stackoverflow.com/questions/12129505/difference-between-java-and-javaw)



# 参考

* [java.lang](https://docs.oracle.com/javase/8/docs/api/)

