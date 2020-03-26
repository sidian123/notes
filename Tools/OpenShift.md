* 概念

  * Project

    为项目分组的名字空间

  * Pod

    容器和存储的集合, 提供运行环境

  * Service

    pod的集合, 提供IP和端口, **集群内**的请求将被以负载均衡的方式转发到pod上.

  * Route

    路由规则, 提供域名, 将**外部**请求转发到集群的容器中.

    > Route和Service都是和路由有关的, Route对外, Service对内

* 参考
  * [openshift基本概念总结](https://blog.csdn.net/qq_23348071/article/details/86605686)