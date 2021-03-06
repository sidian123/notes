# IO

* `java.io`: 含有对文件, 网络流, 内存缓存进行输入输出操作的类. 
* 流分为字节流和字符流, 字符流也就是在字节流的基础上多了个编码或节码器.
* 使用缓存流, 可以提高性能

* IO类使用概述表

| Byte Based       | Byte Based                                                   |                                                              | Character Based                                              |                                                              |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|                  | Input                                                        | Output                                                       | Input                                                        | Output                                                       |
| Basic            | [InputStream](http://tutorials.jenkov.com/java-io/inputstream.html) | [OutputStream](http://tutorials.jenkov.com/java-io/outputstream.html) | [Reader](http://tutorials.jenkov.com/java-io/reader.html) [InputStreamReader](http://tutorials.jenkov.com/java-io/inputstreamreader.html) | [Writer](http://tutorials.jenkov.com/java-io/writer.hml) [OutputStreamWriter](http://tutorials.jenkov.com/java-io/outputstreamwriter.html) |
| Arrays           | [ByteArrayInputStream](http://tutorials.jenkov.com/java-io/bytearrayinputstream.html) | [ByteArrayOutputStream](http://tutorials.jenkov.com/java-io/bytearrayoutputstream.html) | [CharArrayReader](http://tutorials.jenkov.com/java-io/chararrayreader.html) | [CharArrayWriter](http://tutorials.jenkov.com/java-io/chararraywriter.html) |
| Files            | [FileInputStream](http://tutorials.jenkov.com/java-io/fileinputstream.html) [RandomAccessFile](http://tutorials.jenkov.com/java-io/randomaccessfile.html) | [FileOutputStream](http://tutorials.jenkov.com/java-io/fileoutputstream.html) [RandomAccessFile](http://tutorials.jenkov.com/java-io/randomaccessfile.html) | [FileReader](http://tutorials.jenkov.com/java-io/filereader.html) | [FileWriter](http://tutorials.jenkov.com/java-io/filewriter.html) |
| Pipes            | [PipedInputStream](http://tutorials.jenkov.com/java-io/pipedinputstream.html) | [PipedOutputStream](http://tutorials.jenkov.com/java-io/pipedoutputstream.html) | [PipedReader](http://tutorials.jenkov.com/java-io/pipedreader.html) | [PipedWriter](http://tutorials.jenkov.com/java-io/pipedwriter.html) |
| Buffering        | [BufferedInputStream](http://tutorials.jenkov.com/java-io/bufferedinputstream.html) | [BufferedOutputStream](http://tutorials.jenkov.com/java-io/bufferedoutputstream.html) | [BufferedReader](http://tutorials.jenkov.com/java-io/bufferedreader.html) | [BufferedWriter](http://tutorials.jenkov.com/java-io/bufferedwriter.html) |
| Filtering        | [FilterInputStream](http://tutorials.jenkov.com/java-io/filterinputstream.html) | [FilterOutputStream](http://tutorials.jenkov.com/java-io/filteroutputstream.html) | [FilterReader](http://tutorials.jenkov.com/java-io/filterreader.html) | [FilterWriter](http://tutorials.jenkov.com/java-io/filterwriter.html) |
| Parsing          | [PushbackInputStream](http://tutorials.jenkov.com/java-io/pushbackinputstream.html) [StreamTokenizer](http://tutorials.jenkov.com/java-io/streamtokenizer.html) |                                                              | [PushbackReader](http://tutorials.jenkov.com/java-io/pushbackreader.html) [LineNumberReader](http://tutorials.jenkov.com/java-io/linenumberreader.html) |                                                              |
| Strings          |                                                              |                                                              | [StringReader](http://tutorials.jenkov.com/java-io/stringreader.html) | [StringWriter](http://tutorials.jenkov.com/java-io/stringwriter.html) |
| Data             | [DataInputStream](http://tutorials.jenkov.com/java-io/datainputstream.html) | [DataOutputStream](http://tutorials.jenkov.com/java-io/dataoutputstream.html) |                                                              |                                                              |
| Data - Formatted |                                                              | [PrintStream](http://tutorials.jenkov.com/java-io/printstream.html) |                                                              | [PrintWriter](http://tutorials.jenkov.com/java-io/printwriter.html) |
| Objects          | [ObjectInputStream](http://tutorials.jenkov.com/java-io/objectinputstream.html) | [ObjectOutputStream](http://tutorials.jenkov.com/java-io/objectoutputstream.html) |                                                              |                                                              |
| Utilities        | [SequenceInputStream](http://tutorials.jenkov.com/java-io/sequenceinputstream.html) |                                                              |                                                              |                                                              |

## PushbackInputStream

`PushbackInputStream`有能力让你将读出来的字节压回流中, 如

```java
PushbackInputStream inputstream = new PushbackInputStream(
                                new FileInputStream("c:\\data\\input.txt"));

int data = inputstream.read();

inputstream.unread(data);
```

* `read()` 读出字节
* `unread()` 让字节放回去

# 文件

* `java.nio.file`: Defines interfaces and classes for the Java virtual machine to access files, file attributes, and file systems.

## File(弃)

* 该类表示为抽象的, 独立于系统的层次化路径名, 自称为**抽象路径**`abstract pathname`.

  > 即这里将路径分为真实路径(与系统相关), 与抽象路径.

* 抽象路径组成

  * 可选的系统前缀, 如`C:\`或`/`
  * 目录名或文件名组成的字符串, 以`\`或`/`分隔

* 抽象路径与真实路径名的转换是与系统相关的

  > 如不同系统中分隔符不同

* 路径有绝对路径和相对路径之分.

  * 绝对路径拥有定位文件的完整信息

  * 相对路径需要获取额外信息, 才能定位到文件.

    > 通常相对于工作目录, 注意, 非classpath目录

* `File`不保证所定位的文件或目录一定存在.

* 关于获取字符串形式的绝对路径

  * `getAbsolutePath`获取的绝对路径中可以含有`..`和`.`, 如`dir/../test.md`'
  * `getCanonicalPath`获取标准的绝对路径, 无`..`和`.`

  > `File`之间的比较`equals`是通过`getAbsolutePath`获取的绝对路径比较的, 因此两个`File`即使指向同一文件或目录, 它们的`equals`比较也不一定为`true`
  
* **弃用**

  `File`是`java.io`包中访问文件的类 , 已被`java.nio.file`包中的`File`和`Files`类取代 

  弃用理由:

  - Many methods didn't throw exceptions when they failed, so it was  impossible to obtain a useful error message. For example, if a file  deletion failed, the program would receive a "delete fail" but wouldn't  know if it was because the file didn't exist, the user didn't have  permissions, or there was some other problem.
  - The `rename` method didn't work consistently across platforms.
  - There was no real support for symbolic links.
  - More support for metadata was desired, such as file permissions, file owner, and other security attributes.
  - Accessing file metadata was inefficient.
  - Many of the `File` methods didn't scale. Requesting a  large directory listing over a server could result in a hang. Large  directories could also cause memory resource problems, resulting in a  denial of service.
  - It was not possible to write reliable code that could recursively  walk a file tree and respond appropriately if there were circular  symbolic links.

  **注意**, 尽管nio是多通道非阻塞IO, 但文件通道仍是阻塞的, 不能再Selector中注册.

## java.nio.file

> 详细使用, 极力推荐看[File I/O](https://docs.oracle.com/javase/tutorial/essential/io/fileio.html)

* 功能

  * 更强大的路径操作, 见`Path`
  * 更强大的文件操作, 见`Files`
    * 遍历文件, 同时见`FileVisitor`

  * 文件树事件监听, 见`WatchService`
  * 与Legacy API--`File`互操作

* 部分操作
  
	* 使用`Paths`创建`Path`
  
* 其他注意

  * 该Package的API能够识别符号链接了, 如

    ```java
    Path file = ...;
    boolean isSymbolicLink =
        Files.isSymbolicLink(file);
    ```

    > 关于符号链接
    >
    > 符号链接类似于一个指针, 含有所引用文件的位置信息, 符号链接的读写会被重定向到目标文件, 但是对符号链接本身的操作, 如更名, 删除, 不会作用到目标文件

> 关于为什么内容这么少...
>
> 在对内容有了大局观并熟悉后, 除了要点要记下外, 其他内容真没记下的必要, 到时知道在哪并翻阅即可...

### Path

* `Path relativize(@NotNull Path other)`

  获取`other`相对于该`Path`的相对路径.

  > 若该`Path`为`/a/b`, `other`为`a/b/c/d`, 那么将返回`c/d`
  
* `Path resolve(Path other)` 

  连接该`Path`与`other`.
  
* 名字

  * 获取文件名

    ```java
    path.getFileName.toString();
    ```

  * 获取文件名的字符表示, 可能相对, 可能绝对

    ```java
    path.getName();
    ```

### Files

* 获取文件属性

  ```java
  Files.readAttributes(path, BasicFileAttributes.class)
  ```

  可以读取文件的常用属性

## 坑

* `Files.getParent()`根据地址计算的, 不会考虑系统真实情况. 如`./`的`getParent()`将返回`null`. 因此最好将`Path`转化为绝对路径再计算.

  > 其他方法估计也是这样的

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
  
* EOF

  读到EOF, 会返回`-1` , 并且EOF仍在流中, 下次读还是一样的结果. 此外, 输入输出流都处于EOF状态并不代表Socket处于close状态, 仍需手动关闭.

> 参考[oracle socket tutorial](https://docs.oracle.com/javase/tutorial/networking/sockets/index.html)

# NIO

[Java NIO Tutorial](http://tutorials.jenkov.com/java-nio/index.html)

* 核心组件

  * Channel 类似流的概念, 是数据的源或目的地

  * Buffer 数据只能从Channel流向Buffer, 或Buffer流向Channel

    ![img](.Java%20IO/overview-channels-buffers1.png)

  * Selector 多Channel的单线程处理器或调度器, 非阻塞

    > 使用

    ![img](.Java%20IO/overview-selectors.png)


# 参考

* [Java IO Tutorial/Jenkov](http://tutorials.jenkov.com/java-io/index.html)

* [Java IO Tutorial/Oracle](https://docs.oracle.com/javase/tutorial/essential/io/index.html)

* [javadoc java.nio.file](https://docs.oracle.com/javase/8/docs/api/java/nio/file/package-summary.html)

