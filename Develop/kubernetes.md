* k8s对象
  
  * Pod
    
    * 一组容器的集合
    * 共享存储层和同一IP
    * k8s操作的最小调度单位
    
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

* k8s集群

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