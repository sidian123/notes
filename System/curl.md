# 一 介绍

linux中一个传输数据的工具，支持多种协议，还支持代理、用户认证、SSL、cookies等等特性。

一些默认行为：

* 默认使用HTTP协议；
* 消息体输出到terminal；
* 消息体未输出到terminal时，显示进度条。

一些常用选项：

* `-X`：指定请求方法，未设置其他选项时，一般为`Get`方法
* `-H`：指定头字段，一般来说，头字段只需要`Content-Type`，其他会自动设置好。
* `--data`：设置请求体

在`post`请求中，一般只使用`-d,--data`选项，因为此时默认`POST`请求，`Content-Type`为`application/x-www-form-urlencoded`；如果想模拟表单文件上传，可使用`-F,-form`选项，它默认`POST`请求，`Content-Type`为`multipart/form-data`

>同类工具还有wget，但curl更强大

# 二 主要选项

## --data与--form

下列选项都默认使用POST请求方法，通过`<data>`填充请求消息体。

* `--data`系列：`Content-Type`默认为`application/x-www-form-urlencoded`

  * `-d,--data <data>`或`--data-ascii <data>`：会自动进行url编码，能够通过`@`指定文件。

    ```bash
    # 会自动进行url编码
    curl -v --data "name=tom&password=12 3 是的456" http://localhost:8080/test
    #注意内容要进行url编码
    curl -v --data "birthyear=1905&press=%200k%20" http://localhost:8080/test
    #可以一次写一个参数
    curl -v -d "name=tom" -d "password=123  sdw 456" http://localhost:8080/test
    ```

  * `--data-binary <data>`：数据直接填充，**没有额外的数据处理过程(即url编码)**。数据以`@`开头时，会被认定为文件，直接将文件的内容读入，但不进行url编码，如：

    ```bash
    curl -v --data-binary "@baidu.html" -H "Content-Type: text/*" http://localhost:8080/test2
    ```

    > 设置content-type是为了防止后端错误解析内容，

  * `--data-raw <data>`：类似`--data`，但**没有`@`的文件解析过程**。

  * `--data-urlencode <data>`：类似`--data`，**url编码功能更多？**

  > 总之，感觉他们功能重叠，有点分不开，总之`--data`是万能的！！！

* `-F,--form <name=content>`：`Content-Type`默认为`multipart/form-data`。使用例子如下：

  ```bash
  #profile为请求参数名，@指定上传文件
  curl -F profile=@portrait.jpg https://example.com/upload.cgi
  #为part添加Content-Type，默认application/octet-stream
  curl -F "web=@index.html;type=text/html" example.com
  #更换http中显示的文件名
  curl -F "file=@localfile;filename=nameinpost" example.com
  #设置part的头字段
  curl -F "submit=OK;headers=\"X-submit-type: OK\"" example.com
  #多个part
  curl -v -F "file=@test2" -F "other=other argument" http://localhost:8080/test3
  ```

## URL

可以指定多条url的，url最好加上引号。url未指定协议时，默认HTTP，但也会根据域名前缀猜测协议。

* 列举

  ```url
  http://site.{one,two,three}.com
  或
  http://site.one.com http://site.two.com http://site.three.com
  ```

* 范围

  ```url
  ftp://ftp.example.com/file[1-100].txt
  ftp://ftp.example.com/file[001-100].txt    (with leading zeros)
  ftp://ftp.example.com/file[a-z].txt
  ```

* 连续

  ```url
  http://example.com/archive[1996-1999]/vol[1-4]/part{a,b,c}.html
  ```

* 间隔

  ```url
  http://example.com/file[1-100:10].txt
  http://example.com/file[a-z:2].txt
  ```

# 三 其他选项

## 常用选项

输出的内容

* `-X`：指定请求方法
* `-H`：设置头字段

* `-i`：输出包含HTTP响应头字段。
* `-v`（**常用**）：输出更多的信息。`>`表示请求头字段，`<`表示响应头字段，`*`表示额外信息

## 进度条

显示传输进度，速度单位字节。进度条默认会显示出来，但是有数据显示在terminal时会被禁用。

* 其他样式：`-#`,`--progress-bar`
* 禁用：`-s`,`--silent`

## 输出到文件

输出到文件可使用

* `>`：重定向
* `-o,--output <file>`：指定要写入的文件
* `-O`：在当前目录保存文件，文件名为远程文件名。

## 其他

* 认证：指定HTTP认证方法，
  * `--basic`：`Basic认证`，默认
  * `其他还有：--ntlm`,`--digest`,`--negotiate`

* 凭证
  
* `-b,--cookie <data>`：设置cookie，默认为上次请求设置的cookie。`data`的格式为`NAME1=VALUE1; NAME2=VALUE2`
  
* 其他
  * `-I`：仅获得头部
  * `-L`：如果响应要求重定向，则向新地址发起请求。

   * 其他例子：

     ```bash
     //下载jdk，允许重定向，以网络文件名保存在本地
     curl -L -b "oraclelicense=a" -O http://download.oracle.com/otn-pub/java/jdk/10.0.2+13/19aef61b38124481863b1413dce1855f/jdk-10.0.2_linux-x64_bin.tar.gz
     //向微信接口发送图片
     curl -F media=@test.jpg "https://api.weixin.qq.com/cgi-bin/media/upload?access_token=ACCESS_TOKEN&type=TYPE"
     
     ```



