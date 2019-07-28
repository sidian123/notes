# 初识

* **介绍**

  Eureka提供服务注册与发现的功能, **主要为了**给负载均衡服务和其他中间层故障转移服务提供底层服务.

  **Eureka Server**提供该功能, **Eureka Client**与server交互. client一般含有内置的负载均衡器(Ribbon), 使用基础的轮询算法.

* 

