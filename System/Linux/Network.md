# 基础

![在这里插入图片描述](.Network/20190322161040387.png)
关于防火墙，这里在做补充。先看一幅图
![在这里插入图片描述](.Network/20190322161359967.png)
防火墙是内核提供的功能，可以通过`iptables`命令直接设置防火墙规则，也可以通过图中的两个守护进程提供的工具来设置，它们底层使用的命令都是`iptables`。目前发行版大都使用firewalld作为守护进程，使用`firewall-cmd`来配置规则，这里就介绍firewalld。

firewalld中，**zone**是一系列规则的集合（类似iptables命令中的chain），zone的target字段定义默认行为，如果**到来**（incoming）数据包没有匹配zone的规则，就使用默认行为。firewalld预定义了多种zone，默认网卡设备使用public。public zone中target为default，表示没有成功匹配的（incoming）数据包会被拒绝进入。

规则可以被定义为多种形式，如直接指定服务，如http，ftp，ssh；或者使用端口+协议，如80/tcp，123/udp。但是你可能会发现这些功能很有限，远没有`iptables`命令灵活，因此`firewall-cmd`提供`--direct`直接使用`iptables`命令的规则来配置防火墙。当不用直接使用iptables，这会造成firewalld服务内核数据不一致。

规则分为运行时规则和永久规则，其实就是保存为内存中副本的规则为运行时规则，保存文件中的规则为永久规则。只不过`firewall-cmd`一次只能保存一个地方，内存或文件。

网卡具体使用什么zone由NetworkManager确定，如果没有明确指定，会使用firewalld的默认zone，一般为public，但可以通过`public-cmd`修改。

# 命令

## ip

ip命令功能很强大，常用于查看、修改、设置网卡ip、路由表、设备等功能。配置不会被保存。

>代替了ifconfig、route、arp等老工具的使用

>ip [ OPTIONS ] OBJECT { COMMAND | help }
>
>* `OBJECT`：表示ip具体的某个子功能，常用的为`addr`（设置ip）、`route`（路由表）、`neighbour`（arp缓存）. 
>
> > 可使用简写.
>
>* `COMMAND`：每个object的动作都不一样，可以查询、修改、添加对应实体，但如果没有给出则默认查询
>
>* `help`：通过help可查询对应object具体用法

用法参考：[Linux命令之ip](http://www.cnblogs.com/diantong/p/9511072.html)

## ss

用于查看sockets信息。

>代替了老工具netstat的使用

>```shell
>ss [options]
>```
>
>默认显示所有已建立连接（non-listening）的sockets，包括TCP/UNIX/UDP类型的socket。
>
>* `-l`:仅显示正在监听（listening）的sockets
>* `无`：仅显示已建立连接（non-listening）的sockets
>* `-a`：同时显示正在监听（listening）和已建立连接（non-listening）的sockets
>
>> 注意, 已建立连接和监听是两种状态, 一种是还未连接, 所以要监听; 一种是已经连接上了, 不用监听了
>
>----------------
>
>* `-t`：仅显示tcp类型的sockets
>* `-u`：仅显示udp类型的sockets
>* `-x`：仅实现unix类型的sockets
>* `无`：同时显示tcp、udp、unix类型的sockets
>
>----------------
>
>* `-n`：显示端口而不是服务名
>* `-p`：显示使用socket的进程

## nmtui

NetworkManager提供的，用于配置网络的字符界面工具。如配置网卡ip、掩码、网关、DNS服务器，并且配置会被保存下来。

## firewall-cmd

* 检测防火墙状态

  ```bash
  sudo firewall-cmd --state
  ```

* 重新加载防火墙规则，运行时添加但没有保存的规则会消失

  ```bash
  sudo firewall-cmd --reload
  ```

* 一般添加规则不会被永久保存，添加`--permanent``永久保存却不会立刻生效。有两种方法解决：

  ```bash
  # 设置运行时规则后再保存
  firewall-cmd <other options>
  firewall-cmd --runtime-to-permanent
  保存到文件中，然后重新加载所有规则
  firewall-cmd --permanent <other options>
  firewall-cmd --reload
  ```

* zone操作

  ```bash
  # 查看当前默认zone
  sudo firewall-cmd --get-default-zone
  # 改变默认zone
  sudo firewall-cmd --set-default-zone=internal
  # 查看网卡设备使用的zone。网卡只会在没有明确设置zone时才使用默认zone
  sudo firewall-cmd --get-active-zones
  # 得到一个zone的全部配置
  sudo firewall-cmd --zone=public --list-all
  # 得到全部zone的配置
  sudo firewall-cmd --list-all-zones
  ```

* 添加、删除服务或端口，并且端口还可以添加一个范围的端口，如`8000-8999/tcp`

  ```bash
  # 通过服务名添加、删除服务
  sudo firewall-cmd --zone=public --add-service=http --permanent
  sudo firewall-cmd --zone=public --remove-service=http --permanent
  # 通过端口+协议添加、删除服务
  sudo firewall-cmd --zone=public --add-port=12345/tcp --permanent
  sudo firewall-cmd --zone=public --remove-port=12345/tcp --permanent
  ```

* 等等等等

参考：[Introduction to FirewallD on CentOS](https://www.linode.com/docs/security/firewalls/introduction-to-firewalld-on-centos/)

# 网络进阶

## curl

>已重新学习curl，详细见[curl](https://blog.csdn.net/jdbdh/article/details/90911383)

curl用于与服务间传输数据，支持各种协议。但我只用http协议

>curl [options] [URL...]
>默认从URL下载数据到终端

* `-i`:输出包括http头字段
* `-I`：仅获得头部
* `-v`：输出更多的信息。`>`表示请求头字段，`<`表示响应头字段，`*`表示额外信息
* `-H`:添加请求头字段
* `-b`：添加cookie
* `-L`：如果响应要求重定向，则向新地址发起请求。
* `-o` <file>：指定文件保存
* `-O`：于当前目录保存文件，文件名为远程文件名。
* `-F, --form <name=content>`：对于http协议族，curl以`multipart/form-data`类型的`Content-Type`发送post数据。如果要发送文件，`content`需要以`@`开始，后接文件名。

例子：

```bash
//下载jdk，允许重定向，以网络文件名保存在本地
curl -L -b "oraclelicense=a" -O http://download.oracle.com/otn-pub/java/jdk/10.0.2+13/19aef61b38124481863b1413dce1855f/jdk-10.0.2_linux-x64_bin.tar.gz
//向微信接口发送图片
curl -F media=@test.jpg "https://api.weixin.qq.com/cgi-bin/media/upload?access_token=ACCESS_TOKEN&type=TYPE"
```

>同类工具还有wget，但curl更强大
>最全教程：[Using curl to automate HTTP jobs](https://curl.haxx.se/docs/httpscripting.html)

## Unix Domain Sockets

socket有很多中，我们常用的是network socket，linux也提供了unix socket，它表现起来像TCP或UDP，但它只用作内部进程交流的途径，因此更高校。

# 其他

## 实时查看网速

```shell
sar -n DEV 1 100
```

命令解读

* `sar`用于打印系统活动信息
* `-n DEV`指定打印网络信息
* `1`表示每秒统计一次
* `100` 表示统计100次

输出结束

```log
07:48:29 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
07:48:30 PM      eth0      2.00      2.00      0.12      1.21      0.00      0.00      0.00      0.00
07:48:30 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

分别是

* 网络接口名 IFACE
* 每秒收包率 rxpck
* 每秒发包率 txpck
* 每秒接收字节 rxkB
* 每秒发送字节 txKb
* ...
* 网络接口利用率 ifutil

## 修改主机名

```shell
hostname hk.sidian.live
```

> 直接修改`/etc/hostname`文件无效

## nc & telnet

`nc`用于简单的创建一个sockets并监听, 如

```shell
nc -l localhost 8080
```

> 监听本地8080端口

`telnet`简单的与监听端口的进程交互

```shell
telnet localhost 8080
```

两者都可以在里面发送消息, 只是`nc`是监听者, `telnet`是请求者罢了.

# 参考

* [Introduction to FirewallD on CentOS](https://www.linode.com/docs/security/firewalls/introduction-to-firewalld-on-centos/)
* [CHAPTER 5. USING FIREWALLS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls#sec-Introduction_to_firewalld)
* [Firewalld](https://fedoraproject.org/wiki/Firewalld?rd=FirewallD)
* [CentOS 7 中firewall-cmd命令](https://www.jianshu.com/p/411274f96492)
* [Computer network](https://en.wikipedia.org/wiki/Computer_network)
* [OSI model](https://en.wikipedia.org/wiki/OSI_model)
* [A Deep Dive into Iptables and Netfilter Architecture](https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture)
* [iptables查看、开放、删除端口、保存设置](https://www.centos.bz/2018/01/iptables%E6%9F%A5%E7%9C%8B%E3%80%81%E5%BC%80%E6%94%BE%E3%80%81%E5%88%A0%E9%99%A4%E7%AB%AF%E5%8F%A3%E3%80%81%E4%BF%9D%E5%AD%98%E8%AE%BE%E7%BD%AE/)
* [NetworkManager](https://en.wikipedia.org/wiki/NetworkManager)
* [What's the difference between Network Initscript and NetworkManager in RHEL 7?](https://access.redhat.com/solutions/783533)
* [Linux命令之ip](http://www.cnblogs.com/diantong/p/9511072.html)