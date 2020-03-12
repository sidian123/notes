[TOC]

# 一、介绍

http是一种请求/响应式的协议，无状态的协议。http经历了很多版本，但http1.0和http1.1需要了解一下，而http1.1现在正在广泛使用，05年出现了http2.0，但未普及使用。

http1.0中浏览器与服务器交互需要经历建立连接、发送请求信息、回送响应信息、关闭连接四个步骤。每次只能处理一个请求，因此因此多个请求时比较耗时，效率低。

http1.1支持持久连接，也就是一个tcp连接上可以传送多个http请求和响应，而不是一个请求响应一个tcp连接和关闭，因此减少了建立和关闭连接的消耗和延时。

# 二、消息

http消息分为http请求消息和http响应消息，消息的第一行和头部字段使用acsii编码。指的是0到127的acsii码，因此如果不符合的值需要转码，比如请求参数、Content-MD5字段等。消息体如果有字符，估计使用Content-type字段的编码吧，，不确定。。

## HTTP请求消息

请求消息由请求行、请求头、空白行和可选的消息体组成。

```
GET /index.html HTTP/1.1
Host: www.example.com
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请求行和请求头必须通过回车换行（\r\n）结束，空行必须是回车换行（\r\n）。

请求行由请求方法、资源路径名（有的书把它称为uri，这是不对的）和http协议组成。请求头由键值对组成，注意冒号后有一个空格！！，在http1.1中除了host字段其他的都是可选的。在使用post等请求方法时可以存在消息体。

## HTTP响应消息

由响应状态行、响应头、空行和可选的消息体（实体内容）组成。

```
HTTP/1.1 200 OK
Date: Mon, 23 May 2005 22:38:34 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 138
Last-Modified: Wed, 08 Jan 2003 23:11:55 GMT
Server: Apache/1.3.3.7 (Unix) (Red-Hat/Linux)
ETag: "3f80f-1b6-3e1cb03b"
Accept-Ranges: bytes
Connection: close

<html>
<head>
  <title>An Example Page</title>
</head>
<body>
  Hello World, this is a very simple HTML document.
</body>
</html>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

和请求消息一样，请求行和请求头需要\r\n结束本行，空行必须为\r\n。请求头字段的键值由冒号和空格分隔。

请求行由http协议、状态码和状态码对应的描述组成。

# 三 请求方法

请求方法用来表示对指定资源的动作，比如get获取资源、post提交资源。http1.0定义了GET、POST和HEAD，http1.1新添OPTIONS、PUT、DELETE、TRACE和CONNECT。每个对应的方法在服务器中如何被对待，可以由后台程序员自己确定，但是最好遵守语义。

> 参考[请求方法](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)

## GET

在浏览器地址输入URL、点击网页上超链接时，浏览器使用get方法发送请求，提交form表单时也默认使用get请求方法。如果有请求参数，则将参数放入请求行的资源路径后。

如URL：http://www.example.com/home/filename?name=luo&password=12345

那么请求行：GET /home/filename?name=luo&password=12345 HTTP/1.1

get方式可以传递参数给服务器，但是不能超过1kb，因此如果上传文件则需要POST请求方式。

## POST

当form表单的method属性为POST时，则使用post提交表当内容，表单参数作为消息体发给服务器。如下面的例子：

```
POST /javaForum HTTP/1.1
Host: www.itcast.cn
Content-Type: application/x-www-form-urlencoded
Content-Length: 17

name=lee&psd=hnxy
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

由于使用了消息体，所以必须指定Content-Type字段，Content-Length指定消息体长度（可以使用其他方法）。Host在http1.1中必须指定。

其他的请求方式参考链接。

# 四 状态字

在http响应消息时会在响应状态行中放回状态字，根据状态字就可以知道响应的大致结果，详细结果需要和响应头具体确定。状态字有三位数字，最高位确定大致分类：

- Informational `1XX`
- Successful `2XX`
- Redirection `3XX`
- Client Error `4XX`
- Server Error `5XX`

其中比较常见的有：

- 200：表示服务器成功处理了客户端的请求
- 302：表示重定向，需要Location字段给出临时资源的URL
- 404：表示服务器找不到请求的资源
- 500：表示服务器发送错误，无法处理客户端的请求

> 参考[List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

# [五、头字段](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Effects_of_selected_fields)

请求头和响应头是定义了HTTP事务的操作参数。头字段分为**请求字段**、**响应字段**。其中有些字段是服务器浏览器都可以使用的**通用字段**，和与消息体具体相关的**实体字段**。一些非标准的、正在实验的字段名通常（不是所有）以X-开头。
下面简单介绍下字段，详细介绍参考标题链接。

## 请求字段

浏览器发出请求时给服务器信息的字段。

1. Accept：浏览器能够处理的MIME类型。
2. Accept-Charset：告知服务器客户端使用的字符集
3. Accept-Encoding：客户端能够解码的编码方法
4. Accept-Language：客户端期待的指定国家语言的文档，因为不同国家对相同字符有不同的渲染方式和含义，有时需要设置此字段。
5. Authorization、Proxy-Authorization：访问受保护的网页时需要的这些字段提交账户密码。
6. Host：指定资源所在的主机名和端口号。http1.1强制使用，web服务器通过该字段可以区分[虚拟web站点](https://blog.csdn.net/jdbdh/article/details/82631376#六、配置虚拟主机)。
7. If-Match：附着Etag实体标签，用作使用使用缓存的条件。
8. If-Modified-Since：与If-Match类型，存入GMT格式的时间，服务器会比较此字段，判断浏览器缓存的是否是最新的文档，作为浏览器是否使用缓存的条件。该字段的值是上次响应消息中Last-Modified字段的值。
9. Range和If-Range：指定服务器返回文档中的部分内容及内容范围。
10. Max-Forward：当前请求可以经过的代理服务器数量，经过一个代理服务器该值减一。
11. Referer：通过在浏览器中输入URL访问超链接，通过其他方式访问时会使用Referer字段。比如点击网页上超链接，发出请求时会使用referer字段，该字段指定该网页的URL。比如打开一个网页时，该网页需要其他文件，如css、js、image，那么请求这些资源时会通过referer指定该网页。该字段用来检测访问者如何导航进入该网站的，也可用于防止盗链的存在。
12. User-Agent：指定浏览器的操作系统及版本、浏览器及版本、浏览器渲染引擎、浏览器语言等信息。

## 响应字段

服务器响应时使用的字段。

1. Accept-Range：表明服务器是否接受客户端的Range字段请求。
2. Age：客户端和代理服务器可以缓存的有效时间，可以作为客户端是否使用缓存的条件。
3. Etag：代表实体内容特征的标记信息，每个版本的资源的实体标签是不同的，可以作用是否使用缓存的条件。
4. Location：配合3xx状态码使用，用于重定向，该字段指定请求文档的新地址，值为URL的绝对地址。因为此时响应消息没有消息体，因此没有Content-Type，所以Location和Content-Type不能同时出现。
5. Retry-After：与503状态码配置使用，告诉浏览器什么时间内可以重新发送请求。
6. Server：指定服务器软件产品的名称。
7. Vary：指定影响了服务器所生产的响应内容的那些请求头字段名。
8. WWW-Authenticate、Proxy-Authenticate：客户端访问受保护的网页时要使用的字段。
9. Refresh：浏览器自动刷新页面的时间。
10. [Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)：如果让用户选择将响应的实体内容保存到一个文件中时需要使用该字段。也可以用在multipart/form-data类型的请求消息体的subpart中，给与字段相关信息，比如名字、文件名。

## 通用字段

通用字段就是既可适用于请求消息也可适用于响应消息的字段。

1. Cache-Control：用在请求消息，表明代理服务器如何使用已缓存的页面；使用在响应消息中，通知客户端和代理服务器如何缓存页面。
2. Connection：指定处理完本次请求/响应后，是否还需要保存连接，http1.1默认采用持久连接，值为Keep-Alive。
3. Date：消息产生的当前时间，值为GMT。
4. Pragma：用于http1.0中通知代理服务器和客户端如何缓存页面。在http1.1中Cache-Control代替了Pragma的使用。
5. Transfer-Encoding：在http1.1中默认使用持久连接，为了区分不同请求，需要提供Content-Length字段。但是一些数据是动态生成的，必须全部生成完后才能确定长度，因此通过该字段指定传输的编码方式。比如chuncked，分段传输。
6. Trailer：一些字段可以放置于尾部，这些字段需要通过Trailer指定。
7. Upgrade：指定要切换的新的通信协议。
8. Via：指定途径的代理服务器所使用的协议和主机名。
9. Warning：用于添加其他字段和状态码不能表达的附加警告信息。

## 实体字段

请求消息和响应消息都可以传递实体内容，则实体字段作为元信息可以描述实体内容的属性。

1. Allow：指定请求资源所支持的请求方式，必须与405响应状态码使用。
2. Content-Language：指定返回网页文档的国家语言类型。
3. Content-Length：实体内容的长度。响应消息中如果使用了Transfer-Encoding，则不需要使用该字段了。
4. Content-Location：实体内容的实际位置路径（实体内容所在路径不一定就在请求资源路径）。
5. Content-Range：指定服务器返回部分实体内容的位置信息，只有客户端使用了Range时响应头才会包含Content-Range。
6. Content-MD5：用于提供对实体内容的完整性检查，他的值为对实体内容MD5数字摘要后再进行Base64编码结果。因为MD5一些字节无法转化为ascii才进行Base64编码的。
7. Content-Type：实体内容的MIME类型。大多服务器会在配置文件中这是文件扩展名与MIME类型的映射关系，来自动确定请求资源的MIME类型，比如tomcat，使用<mime-mapping>来设置。Content-Type字段后还可以指定实体内容的编码，如content-Type: text/html; charset=GB2312，由分号和空格隔开。如果没有指定，则默认表示为ISO-8859-1字符编码。
8. Content-Encoding：实体内容的压缩编码方式。
9. Expires：文档过期时间，GMT格式时间。
10. Last-Modified：文档最后的更改时间，为GMT格式。客户端收到该字段后，作为下次请求该文档的If-Modified-Since的值。

# [六、缓存](https://en.wikipedia.org/wiki/Web_cache#cite_note-4)

从缓存所处的位置可以分为forward cache和reverse cache。

forward cache位于客户端与互联网之间，拦截客户端的请求，缓存内容主机的文档，返回给客户端。forward cache又分私有cahe和共享cache，比如浏览器缓存网页的cache为私有cache，一个局域网内的代理服务器缓存的为共享cache。

reverse cache位于一个或多个web服务器和互联网之间，用于加速请求，降低负载峰值。

通常http缓存被限制于get请求方式。缓存的主键由请求方法和URI组成。而Vary字段也可以作为次键。

## [Freshness（新鲜）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#Freshness)

响应消息通过设置Cache-Control:max-age或Expires告诉cache响应可以被缓存多久。未过期之前是新鲜的，因此下一次对此资源请求时不用访问服务器，直接从缓存中获取。如果过期了也不会立马清除该项缓存，而是向服务器询问该缓是否真的过期。通过在请求消息中添加If-Match和If-Modified-Since字段用于验证。如果源文件没有被修改，缓存还是新的，此时服务器返回状态字304。

## [Cache validation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#Cache_validation)

响应消息中含有Cache-control:must-revalidate时，该资源被缓存，但是每次客户端请求时代理服务器必须向服务器验证缓存的文档是否过时。验证分为强验证和弱验证，使用ETags验证时为强验证，在请求消息中通过If-Match字段传递ETags。使用Last-Modified验证时为弱验证，在请求消息中通过If-Modified-Since字段传递。

## 其他

由于缓存主键由请求方法和URI组成，因此通过不通请求方法访问同一URL时，缓存的响应将无效。见[Cache control](https://en.wikipedia.org/wiki/Web_cache#Cache_control)中的Invalidation。

Vary字段作为次键，如果有Vary字段，但是Vary指定的字段的值不一致，则不使用缓存。比如不同客户端使用的文档不一样，那么设置Vary：User-Agent，那么只有User-Agent一样的才获取缓存的文档。见：[Varying responses](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#Varying_responses)。

Pragma、Cache-Control和Expires都可以达到控制缓存的目的，但是考虑到兼容性，当要设置不缓存时，一般将这三个头字段一起使用。

# 参考

* 《Java Web 程序开发入门》传智博客高教产品研发部

* 关于http的维基百科：https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Status_codes

* http头字段：https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Request_fields

* http状态字：https://en.wikipedia.org/wiki/List_of_HTTP_status_codes

* web cache：https://en.wikipedia.org/wiki/Web_cache#Cache_control

* HTTP caching：https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#Varying_responses

* reverse cache和forward cache的区别：https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.edge.doc/concepts/concepts20.html#graphic_content_tr_px_single