# 微服务介绍

- **微服务:** 为服务化的核心就是将传统的一站式应用, 根据业务拆分成一个一个的服务, 彻底地去耦合, 每一个微服务提供单个业务功能的服务, 一个服务做一件事, 从技术角度看就是一种小而独立的处理过程, 类似进程概念, 能够自行单独启动或销毁, 拥有自己独立的数据库.

- **微服务和微服务架构**: 可以理解为规范与实现吧

- **优缺点**: 解耦, 但增加部署负担

- **微服务技术栈**

  | 微服务组件                                   | 实现技术                                                     |
  | -------------------------------------------- | ------------------------------------------------------------ |
  | 服务开发                                     | Springboot, Spring, SpringMVC                                |
  | 服务配置与管理                               | Netflix公司的Archaius, 阿里的Diamond等                       |
  | 服务注册与发现                               | Eureka, Consul, Zookeeper等                                  |
  | 服务调用                                     | Rest, RPC, gRPC                                              |
  | 负载均衡                                     | Ribbon, Nginx等                                              |
  | 服务接口调用<br />(客户端调用服务的简化工具) | Feign等                                                      |
  | 消息队列                                     | Kafka, RabbitMQ, ActiveMQ等                                  |
  | 服务配置中心管理                             | SpringCloudConfig, Chef等                                    |
  | 服务路由(API网关)                            | Zuul等                                                       |
  | 服务监控                                     | Zabbix, Nagios, Metrics, Spectator等                         |
  | 全链路追踪                                   | Zipkin, OpenStack, Kubernetes等                              |
  | 服务部署                                     | Docker, OpenStack, Kubernetes等                              |
  | 数据流操作开发包                             | SpringCloud Stream<br />(封装与Redis, Rabbit, Kafka等发送接受消息) |
  | 事件消息总线                                 | Spring Cloud Bus                                             |

- **微服务框架**: 除了Spring Cloud, 还有Motan(新浪), gRPC(google), Thrift(facebook), Dubbo(阿里)/DubboX(当当网). Spring Cloud和Dubbo提供的微服务功能比较全. 最初Dubbo比较火, 但由于它停止维护长达5年, 导致现在Spring Cloud用的比较多.

# Spring Cloud介绍

- SpringCloud是分布式微服务架构下的一站式解决方案, 是各个微服务架构组件实现技术的集合体, 俗称微服务全家桶.

  ![img](.Cloud/diagram-distributed-systems.svg)

  > 微服务那, 多张叠在一起表示集群, 我们要开发的就是这些微服务, 其他的就是对微服务框架各个组件的配置. 就像当初学SSM一样, 繁杂的配置.

- SpringBoot与SpringCloud

  - SpringBoot专注于快速方便的开发单个个体微服务
  - SpringCloud关注全局的微服务协调与治理框架, 它将SpringBoot开发的一个个单体微服务整合并管理起来.

- SpringCloud与Dubbo对比

  ![1560832922953](.Cloud/1560832922953.png)

## Eureka(服务注册与发现)

- 介绍: 提供**注册**和**发现**微服务的功能

- C-S架构, eureka服务端为服务注册中心, eureka客户端连接服务端并维持心跳连接.

  ![1561544083082](.Cloud/1561544083082.png)

  可以看到, eureka中有三大角色: Eureka Server, Eureka Client(Service Consumer, Service Provider)

- 自动保护机制: 某时刻某一个微服务不可用了, eureka不会立刻清理, 依旧会对该微服务的信息进行保存. 

  - 触发条件: 服务变更, 长时间不发送心跳

## Ribbon

- 介绍: 一套实现客户端负载均衡的工具

## Feign

以面向接口的方式简化rest接口的调用, 集成了Ribbon, 也提供了负载均衡的功能.

# [学习二](https://www.baeldung.com/spring-cloud-bootstrapping)

* Spring Cloud Config
  * 用于集中管理其他微服务的通用配置, 避免因集群而导致配置增多
  * 通常配置会被放入Git仓库中, 获得版本管理的功能
  * 分为服务端和客户端, 客户端需要指明服务器在哪
  * Spring Cloud程序初始化时会读入`bootstrap.properties`配置文件, 因此客户端关于服务器信息的配置的信息可以放入此处.
* Eureka
  * 每个服务都通过`ip/port`区分, eureka可以提供更方便定位到微服务的方式.
  * 分为server和client

* Zuul
  * 提供反向代理功能, 避免CORS的需要.

# [学习三](https://howtodoinjava.com/microservices/microservices-definition-principles-benefits/)

## 微服务介绍

* 每个微服务有都有自己的业务逻辑和数据库.
* 与单体应用相比, 微服务更倾向于将整个系统的业务逻辑拆分到更小的单元.
* 对于单个微服务的修改, 一般不会影响到其他的微服务.
* 微服务之间通常采用轻量级协议通信, 如HTTP,REST,JMS,AMQP等.
* 好处
  * 给于开发更好的扩张性, 如使用不同的语言和数据库来开发单个微服务.
  * 可方便的集群
  * 可无顾虑的尝试新技术
  * 更快的开发效率, 大型单体应用需要更周密的设计, 这种设计是十分昂贵的.

## Config Server

* 将client配置外置集中管理. 
* 也能够在修改配置后, 不刷新client, 需要加个注解, 略.

## Eureka(服务注册与发现)

当服务越来越多和服务开始集群后, 直接通过`host:port`定位某个服务变得很繁琐与复杂. 如

```java
String response = restTemplate.exchange("http://localhost:8098/getStudentDetailsForSchool/abcschool",
                                HttpMethod.GET, null, new ParameterizedTypeReference<String>() {}).getBody();
```

 使用Eureka后, 所有的微服务都在eureka server上注册, 其他微服务只需服务名来访问该服务即可, 如

```java
String response = restTemplate.exchange("http://student-service/getStudentDetailsForSchool/{schoolname}",
                                HttpMethod.GET, null, new ParameterizedTypeReference<String>() {}, schoolname).getBody();
```

> 其中微服务名到`host:port`的转换都是由eureka和rest template内部完成的.

如果`RestTemplate`Bean被`@LoadBalanced`注解, 那么仅通过一个服务名就可以达到负载均衡地访问到该服务.

## Hystrix(Circuit Breaker)

当被调用的微服务出现某种问题或直接停止时, 一般会导致调用者超时或抛出异常, 造成该生态系统不够稳定. 

Hystrix就是用来解决这个问题, 当一个服务出现问题时, 会将执行顺序转换到其他地方, 如fall back处理函数. 仅当检测到该服务正常了, 才恢复正确的执行顺序.

本质上就是在调用其他微服务处的函数上包裹了一层Wrapper函数, 一旦出现问题就执行其他已指定的方法.

## Zuul(API Gateway)

提供给客户端(如浏览器)一个统一的访问API接口. Zuul是一个**边缘**服务, **代理**请求, 转发给对应的服务.

Zuul也提供了过滤器, 过滤某些请求. 如

![img](.Cloud/Zull-filters.jpg)

## Ribbon(Load Balancer)

集群后, 需要一个负载均衡器将访问请求平摊在每个服务上, 达到高可用的目的. 

负载均衡分为服务端负载均衡和客户端负载均衡. 

* 服务端: 即在集群的服务前放置一个负载均衡器, 由它分发流量
* 客户端: 由请求方选择到底请求集群服务器中的哪一个

Ribbon属于客户端负载均衡器, 一般与eureka一起使用, 并且被eureka client依赖. 

具体实现: 都是`RestTemplate`通过服务名访问, 首先会去查询该服务对应的`host:port`, 如果是集群的, 会返回多个对应关系, ribbon则选择其中一个交给`RestTemplate`请求. Ribbon默认的算法是轮询.







# [学习四](https://cloud.spring.io/spring-cloud-static/Greenwich.SR2/single/spring-cloud.html)

## Spring Cloud提供的功能

* Distributed/versioned configuration
* Service registration and discovery
* Routing
* Service-to-service calls
* Load balancing
* Circuit Breakers
* Distributed messaging

## 基础

### Spring Cloud Content

* **介绍**: 提供有用的工具和服务, 扩展了`ApplicationContent`的功能. 如bootstrap context, encryption, refresh scope, and environment endpoints.

* **容器**: Spring Cloud中额外提供了一个`Bootstrap`容器作为应用的父容器, 配置文件以`bootstrap.[yml|properties]`给出. 共享同一个`Environment`.

  > 回顾下, 父容器的bean可以被子容器访问和隐藏.

* **属性源**: 即配置的来源. 默认设置中, 本地配置源由`bootstrap.[yml|properties]`提供, 远程配置源由Spring Cloud Server提供. 一般情况下, 远端配置不被覆盖(隐藏), 除非远端配置源允许它, 见[Overriding the Values of Remote Properties](https://cloud.spring.io/spring-cloud-static/Greenwich.SR2/single/spring-cloud.html#overriding-bootstrap-properties); 而本地配置源可以.

* **日记配置**: 日记配置放在`bootstrap.[yml | properties]`中才会作用于所有事件.

  > 不能使用自定义前缀

### Spring Cloud Commons

* **介绍**: 微服务由一系列基础设施和服务组成, Spring Cloud Commons则提供了关于基础设施的抽象层, 如服务发现,负载均衡,断路机制等

  > 简而言之, 即定义了基础设施服务的接口.

* **服务注册**: Commons提供了`ServiceRegistry`接口, 用于服务注册与注销. 实现类如`ZookeeperServiceRegistry`,`EurekaServiceRegistry`,`ConsulServiceRegistry`等.

  * 默认client运行服务时自动向server注册

* **负载均衡客户端**: 可选择的很多, 其中有`RestTemplate`. 该bean必须手动创建, 并添加`@LoadBalanced`注解, 如

  ```java
  @LoadBalanced
  @Bean
  RestTemplate restTemplate() {
      return new RestTemplate();
  }
  ```

  `RestTemplate`在ribbon存在时, 会被自动配置好.

## Spring Cloud Config

* **介绍**: 分为client端和server端, Spring Cloud Config可将client端的配置外置化, 集中存储在server端中. Server端默认使用git仓库为配置文件的存储仓库.



















