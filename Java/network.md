# overview

- TCP：提供可靠的、点对点的连接；接收到的数据与发送时的顺序一致

- UDP：非可靠（即数据可能丢失）；数据接收的顺序不被保证，相互独立

  > UDP对IP协议进行了简单的封装，因此速度比较快；TCP为了可靠性，牺牲了一定速度。
  >
  > 所谓可靠，就是丢包就重发

- ip地址确定host，端口确定应用。

- `java.net`

  - TCP相关：`URL`, `URLConnection`, `Socket`, and `ServerSocket` 
  - UDP相关：`DatagramPacket`, `DatagramSocket`, and `MulticastSocket`

# URL

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

