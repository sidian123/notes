* 介绍

  ZooKeeper是一种分布式协调服务，用于管理大型主机

  * 提供的服务
    * **命名服务**  - 按名称标识集群中的节点。它类似于DNS，但仅对于节点。
    * **配置管理**  - 加入节点的最近的和最新的系统配置信息。
    * **集群管理**  - 实时地在集群和节点状态中加入/离开节点。
    * **选举算法**  - 选举一个节点作为协调目的的leader。
    * **锁定和同步服务**  - 在修改数据的同时锁定数据。此机制可帮助你在连接其他分布式应用程序（如Apache HBase）时进行自动故障恢复。
    * **高度可靠的数据注册表**  - 即使在一个或几个节点关闭时也可以获得数据。
  * 优点
    * **简单的分布式协调过程**
    * **同步**  - 服务器进程之间的相互排斥和协作。此过程有助于Apache HBase进行配置管理。
    * **有序的消息**
    * **序列化**  - 根据特定规则对数据进行编码。确保应用程序运行一致。这种方法可以在MapReduce中用来协调队列以执行运行的线程。
    * **可靠性**
    * **原子性**  - 数据转移完全成功或完全失败，但没有事务是部分的。

* 基础

  * 架构

    ![ZooKeeper的架构](.Zookeeper/201612291344222238.jpg)

    * Client

      zookeeper的客户端

    * Server

      Zookeeper集群中的一个节点

    * Ensemble

      Zookeeper集群

    * Leader

      Zookeeper集群启动时选举出来的节点. 负责向其他节点发送指令

    * Follower

      集群中跟随Leader指令的节点.

  * 层次命名结构

  

  --------------------

  * 介绍

    Zookeeper是一个分布式的, 开源的, 为分布式应用提供的协调服务. 

    Zookeeper提供了一系列构建分布式应用的*基本单元*, 这些应用功能有同步, 配置维护, 分组和命名.

    为了简化客户端程序的编程使用, Zookeeper提供了类似文件系统的树状数据模型.

  * 动机

    减低分布式应用在协调服务的困难度, 减少竞争条件和死锁引起的bug.

  * 设计目标

    * 简单

      

  

  

  

  

  

  

  * 参考
    * [Zookeeper Doc](https://zookeeper.apache.org/doc/current/index.html)
    * [Zookeeper 教程 W3Cschool](https://www.w3cschool.cn/zookeeper/)

  

  

  

  

  

  

  

  

  