* 概念

  * Project

    为项目分组的名字空间

  * Pod

    由一个或多个容器组成, 是OpenShit中, 被定义, 部署和管理的最小计算单元.

    相对于容器而已, 可简单的把Pod看作物理机.

  * Service

    Pod的集合, 提供IP和端口, **集群内**的请求将被以负载均衡的方式转发到pod上.

  * Route

    路由规则, 提供域名, 将**外部**请求转发到集群的容器中.

    > Route和Service都是和路由有关的, Route对外, Service对内

* 网络

  * Pod

    每个Pod被分配了自己内部的IP地址, 和完整的端口空间, 因此Pod中的多个容器间可共享存储和网络. 

  * Service

    * cluster ip 

      内部IP, 与其他Services处于同于局域网内, 让不同Service的Pod相互访问

    * external IP

      用于被外部访问的IP


* Route详解

  创建Route, 将为应用自动生成一个唯一的域名, 提供外部访问的入口 . 

  自动生成的域名, 是容器云的子域名. 可以自定义域名, 即创建Route时提供自定义域名. 之后在提供该域名的DNS服务器上创建CNAME记录, 指向容器云的入站路由器

* 参考
  
  * [openshift基本概念总结](https://blog.csdn.net/qq_23348071/article/details/86605686)