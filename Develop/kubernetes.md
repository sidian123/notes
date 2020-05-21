# 介绍

## 功能

* 提供容器编排, 调度服务
*  服务发现和负载均衡
* 健康检查和自修复
* 提供硬件资源利用率

## 集群架构

k8s集群由一个主节点和多个工作节点组成

![img](.kubernetes/6534887-ad58ca339c403a4b.png)

* 主节点
  - **Kube-apiserver** 
  
    Exposes the API. 命令行工具`kubectl`也与之交互
  
  - **Etcd** 
  
    Key value stores all cluster data. (Can be run on the same server as a master node or on a dedicated cluster.)
  
  - **Kube-scheduler** 
  
    Schedules new pods on worker nodes.
  
  - **Kube-controller-manager** 
  
    Runs the controllers.
  
  - **Cloud-controller-manager** 
  
    Talks to cloud providers.
  
* 工作节点

  * **Kubelet**

    Agent that ensures containers in a pod are running.

  * **Kube-proxy** 

    Keeps network rules and perform forwarding. Service的映射功能就是kube-proxy提供的.

  * **Container runtime** 

    Runs containers.

## k8s对象

仅列出常用的

### Pod

* 一组容器的集合
* 共享存储层
* 共享同一IP和端口范围, 容器间可通过环路地址(localhost)相互访问
* k8s操作的最小调度单位

可以将Pod看作一个逻辑主机

### Deployment



### Service



# k8s对象

* Pod
  
* Service

  提供服务端负载均衡和服务发现功能

* Volume

* Namespace

* Deployment
  * 管理Pods的创建和扩展
  * 检查Pod的健康, 或重启Pod
  
* DaemenSet

* StatefulSet

* ReplicaSet

* Job

* k8s功能

  * 服务发现和负载均衡
  * 健康检查&自我修复
  * 自动滚动或回滚
  * ...


# k8s集群

![Components of Kubernetes](.kubernetes/components-of-kubernetes.png)

由一个Master节点和多个非Master节点组成

* 主节点(Master Node)

  负责维护k8s集群到你描述的状态, 组成如下

  * kube-apiserver

    Master的API接口, 可通过`kubectl` CLI与之交互.

  * etcd

    高可用的键值对缓存

  * kube-controller-manager

  * kube-scheduler 

    调度器

* 工作节点(Worker Node)

  负责运行Pod, 由两个进程组成

  * kubelet 

    负责与Master交互

  * kube-proxy

    网络代理

* 待学习

  [A Kubernetes Basics Tutorial](https://www.bmc.com/blogs/what-is-kubernetes/) 