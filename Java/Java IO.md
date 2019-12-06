# 一 流

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

# 二 文件

* `java.nio.file`: Defines interfaces and classes for the Java virtual machine to access files, file attributes, and file systems.

## File(deprecated)

> `File`是`java.io`包中访问文件的类, 已被`java.nio.file`包中的`File`和`Files`类取代

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

## 坑

* `Files.getParent()`根据地址计算的, 不会考虑系统真实情况. 如`./`的`getParent()`将返回`null`. 因此最好将`Path`转化为绝对路径再计算.

  > 其他方法估计也是这样的

# 参考

* [Java IO Tutorial/Jenkov](http://tutorials.jenkov.com/java-io/index.html)

* [Java IO Tutorial/Oracle](https://docs.oracle.com/javase/tutorial/essential/io/index.html)

* [javadoc java.nio.file](https://docs.oracle.com/javase/8/docs/api/java/nio/file/package-summary.html)

