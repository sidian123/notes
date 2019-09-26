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

## 静态内容映射

  























