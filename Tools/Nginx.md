# 了解

* 介绍: 静态web服务器
* 用途
  * 反向代理
  * 负载均衡
  * 微服务
  * API网关
  * 邮件代理!
  * HTTP缓存

* 历史

  * 2004年由[Igor Sysoev](https://en.wikipedia.org/wiki/Igor_Sysoev)创建
  * 2011年,同名公司创建, 提供Nginx Plus付费软件
  * 2019年,该公司被Netcraft收购

  > 尽管Nginx Plus是付费软件, 但Nginx仍是发布在类BSD许可下的开源免费软件.

* 特性

  * 使用异步事件驱动机制, 而非多线程来处理请求, 能够提供更好的负载能力.

    > 不是说异步事件驱动就不会使用多线程处理请求, 仅是从请求到来时, 如何处理请求这个方式来区分的.

  * 可以和其他CGI应用组合, 服务动态网页
  
  * ...

# 入门

* 组成

  * 一个主进程: 读取和evaluate配置和维护工作进程
  * 多个工作进程: 处理请求的地方

  > 基于事件驱动模型来分发请求, 这个功能谁来干? 不知道...

* 安装

  * 安装前准备必要命令

    ```bash
    sudo apt install curl gnupg2 ca-certificates lsb-release
    ```

  * 设置nginx的apt仓库

    * 使用稳定版本nginx的仓库

        ```bash
        echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
        ```
        
    * 使用主线版本nginx的仓库

        ```bash
        echo "deb http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
        ```

  * 导入nginx的签名密钥, 校验仓库的可靠性

    ```bash
    curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
    ```

  * 从nginx仓库中安装软件

    ```bash
    sudo apt update && sudo apt install nginx
    ```

* 管理

  * `nginx`启动

  * `nginx -s stop`快速关闭

  * `nginx -s quit`完整关闭

    > 其他的关闭方法:
    >
    > `kill -s QUIT 1628`, 进程号可从pid文件中获取
    >
    > 使用`QUIT`, 主线程就会去关闭工作线程. 最好不要用`KILL`

  * `nginx -s reload`重载配置

    > 修改配置后需要手动加载.
    >
    > 加载原理: 主线程解析配置, 成功则通知老工作线程做扫尾工作, 并开启新工作进程. 否则仍使用老配置

  * `nginx -s reopen`重开日记文件

  * `ps aux|grep nginx`查看所有nginx进程

* 文件
  * 日志目录`/usr/local/nginx/logs` 或 `/var/log/nginx`.
  * 配置文件`/etc/nginx/nginx.conf`

# 配置

## 介绍

  * 位置: 位于`/etc/nginx.conf`文件中

## 组成

配置由指令和注解组成, 指令可分为

* 简单指令

  * 由名字和参数组成

  * 多个参数以空格分割

  * 指令以`;`结束

  * 例子

    ```bash
    pid        /var/run/nginx.pid;
    ```

* 块指令

  * 与简单指令结构类似

  * 指令内容被包裹在`{}`之内, 里面可以有其他简单指令或块指令

  * 块指令构成了一个上下文`context`, 用以归类配置

  * 最外层的指令默认处于`main`上下文中

  * 例子

    ```bash
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
    
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        access_log  /var/log/nginx/access.log  main;
    
        sendfile        on;
        #tcp_nopush     on;
    
        keepalive_timeout  65;
    
        #gzip  on;
    
        include /etc/nginx/conf.d/*.conf;
    }
    ```

  * 注解: 以`#`开始的行为注解

## 资源映射

* 指令层次结构
  * 一个`http`可含有多个`server`, 即可监听多个端口
  * 一个`server`可含有多个`location`, 即多个路径匹配, 资源可存多个地方
  * 一个`location`含一个`root`或`proxy_pass`
    * `root`表示资源文件的根路径, `root`路径+请求路径=资源的实际路径
    * 资源也可来自其他URL, 通过`proxy_pass`指定, `proxy_pass`的url+请求路径=资源的实际路径

* `http`: 仅仅只是上下文, 分隔环境

* `server`: 代表一个虚拟服务器, 可以有多个时, 该使用哪个配置? 可通过服务名和IP:Port选择

  * `listen`指定监听的地址和端口, ip可省略

  * `server_name`: 指定域名

  * 匹配规则: 当请求到来时

    * 找出IP和端口一致的`server`
    
    > ip和端口都由`listen`给出
    
    * 再将请求的`Host`与`server_name`比较, 成功匹配则使用该规则, 否则使用默认`server`
    
    > 一般第一个`server`指令即为默认`server`, 可以通过`default_server`参数修改默认`server`, 该参数位于端口号后

* `location`: 匹配HTTP请求的路径, 即`path`

  * 语法

      ```
      Syntax:	location [ = | ~ | ~* | ^~ ] uri { ... }
              location @name { ... }
      Default:	—
      Context:	server, location
      ```

  * 匹配模式: 第一个参数定义匹配方式:
  
      >大致分为前缀和后缀匹配
  
      * 空: 前缀匹配
        
      * `~`: 使用正则进行后缀匹配, 大小写敏感
  
      * `~*`: 使用正则进行后缀匹配, 大小写不敏感
  
      * `^~` 前缀匹配, 同时不使用后缀匹配过程.
  
          > 即影响下面的匹配规则
  
      * `=`: 准确匹配, 同样也无后缀匹配过程
  
  * 匹配规则: 当`server`收到请求时
  
    * 先前缀匹配  选择并记住前缀最长的`location`
    * 再匹配后缀, 选中第一个成功匹配的后缀并**结束**, 使用该`location`的配置
    * 如果后缀匹配不成功, 则使用之前记住的最长前缀的`location`配置
  
  * 例子, 见文档[location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)
  
  > 注意, 前缀匹配的字符串以`/`结尾时, 请求会被cgi程序处理, 这些都是用来处理动态资源的, 如php. 如果请求匹配该字符串, 除了没有`/`, 则会被重定向到该前缀上来.

* `root`: 指定请求资源的根目录, 如

  ```
  location /i/ {
      root /data/w3;
  }
  ```

  `/i/top.gif`的请求将访问`/data/w3/i/top.gif`文件, 即文件路径=`root`+请求的`path`

* `proxy_pass`: 不仅可以访问本地的资源, 还可以访问其他URL提供的资源, 这就是反向代理...

  使用时只需将`root`替换成`proxy_pass`即可, 参数为URL(可以含路径), 最终资源的路径计算与`root`一致

# 其他

## 403 forbidden

403说明我们的访问受限, 很多种原因都可造成403, 这里只谈及我遇到的.

原因:

虽然我们是通过`root`用户运行的Nginx, 但是在它的配置文件中, 指定了工作进程使用`nginx`用户来运行. 因此不能访问`root`用户的文件.

解决方案:

1. 在配置文件中指定工作进程以`root`身份运行, 但不安全
2. 改变文件所属者为`nginx`, 给与目录访问权限即可. (推荐)

> 参考:[Nginx出现403 forbidden](https://blog.csdn.net/qq_35843543/article/details/81561240)

# 参考

* [所有指令](http://nginx.org/en/docs/http/ngx_http_core_module.html)











