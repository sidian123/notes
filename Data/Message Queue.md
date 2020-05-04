# 一 介绍

## MQ使用场景

主要用于系统间解耦, 异步, 削峰.

> 下面都假设传统模式通过Rest API访问, 中间件模式使用MQ队列访问

### 解耦

* 传统模式

  ![img](.Message%20Queue/o_jieou5.png)

  系统间相互调用, 耦合度太高, 牵一发而动全身

* 中间件模式

  ![img](.Message%20Queue/o_jieou6.png)

  通过中间件约定好一个协议, 生产者将消息存入队列, 消费者自行订阅, 业务改变时无需生产者修改, 到达解耦的目的

### 异步

* 传统模式

  ![img](.Message%20Queue/o_yibu2.png)

  系统间同步运行, 一次请求需要所有系统完成才结束, 每个系统完成的时间不一样, 导致部分系统闲置, 且响应速度慢.

* 中间件模式

  ![img](.Message%20Queue/o_yibu3.png)

  消费者以订阅的方式获取数据, 即自己空闲了即可去获取数据, 消费者不必等待所有其他系统的完成, 极大提高了资源利用率的同时, 也加快了生产者响应的速度.

### 削峰

* 传统模式

  ![img](.Message%20Queue/o_xuefeng1.png)

  请求后立即写入数据库, 造成数据库压力山大

* 中间件模式

  ![img](.Message%20Queue/o_xuefeng2.png)

  请求缓存到队列中, 系统以异步的方式写入数据, 不慌不忙, 给数据库减压

## MQ的缺点

* 降低整体系统可用性: 消息队列挂了, 系统就不可用了
* 增加系统复杂性: 加入MQ后, 需要考虑的东西增多, 如一致性问题, 保证消息不被重复消费, 保证消息可靠传输等.

## 消息队列实现对比

| 特性       | ActiveMQ                                                     | RabbitMQ                                                     | RocketMQ                 | kafka                                                        |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------ | ------------------------------------------------------------ |
| 开发语言   | java                                                         | erlang                                                       | java                     | scala                                                        |
| 单机吞吐量 | 万级                                                         | 万级                                                         | 10万级                   | 10万级                                                       |
| 时效性     | ms级                                                         | us级                                                         | ms级                     | ms级以内                                                     |
| 可用性     | 高(主从架构)                                                 | 高(主从架构)                                                 | 非常高(分布式架构)       | 非常高(分布式架构)                                           |
| 功能特性   | 成熟的产品，在很多公司得到应用；有较多的文档；各种协议支持较好 | 基于erlang开发，所以并发能力很强，性能极其好，延时很低;管理界面较丰富 | MQ功能比较完备，扩展性佳 | 只支持主要的MQ功能，像一些消息查询，消息回溯等功能没有提供，毕竟是为大数据准备的，在大数据领域应用广。 |

中小型软件公司建议用RabbitMQ, 其开源且社区比较活跃; 大型软件公司建议用RocketMQ或Kafka, 吞吐量比较大

## 其他

* 如何保证高可用? 集群, 主从复制

# 二 RabbitMQ

RabbitMQ是一个实现了AMQP（Advanced Message Queuing Protocol）高级消息队列协议的消息队列服务, 由Erlang语言编写的。 

是MQ的一种实现, 这里仅介绍原理, 不介绍RabbitMQ的Java客户端, 而在下一章节介绍它的封装库Spring AMQP.

## 组件模型

![image-20191202143124228](.Message%20Queue/image-20191202143124228.png)

* RabbitMQ Server 又称Broker(代理) 
  * 可以存在多个VHost, 每个VHost拥有完整的Rabbit消息模型, 如队列, 交换器和绑定, 还拥有自己的权限控制.

  * VHost的一些规则

    * 默认存在一个VHost, 以`/`标识
    * VHost间是隔离的, 无法通讯. 也即名字隔离
    * 创建用户时必须指定VHost

* 消息传递过程

  * 生产者将消息发送到VHost的Exchange中.

    > 消息中必须指定Routing Key信息

  * Exchange根据消息的Routing Key, Exchange与Queue的Bind Key, 和算法, 选择消息的下一路由, 发送给对应的Queue

    > 在使用过程中, 若不显示指定Exchange, 将发送给**默认Exchange**. 详细见2.2小节-Exchange路由

  * Queue可能会被多个消费者监听, 怎么分配消息, 由对应算法决定, 然后发送给消费者.

    > 详细见2.3小节-消息分配

## Exchange路由

**默认Exchange**是`direct`类型的, 且名字为空.

### fanout

Exchange以广播的方式将消息推送到所有与Exchange绑定的队列中

![img](.Message%20Queue/python-three-overall.png)

> X收到的消息, 将同时发送给两个队列
>
> Routing Key和Binding Key将被忽略

### direct

根据消息的Routing Key, 将之发送到与Binding Key对应的队列中.

例子如下

* 不同消息推送到不同队列

  ![img](.Message%20Queue/direct-exchange.png)

* `routing key`为black的消息广播给所有队列, 其他忽略

  ![img](.Message%20Queue/direct-exchange-multiple.png)

* 按照日志的严重级别推送到不同的队列中

  ![img](.Message%20Queue/python-four.png)

### topic

消息的`routing_key`与队列的`bind_key`**模式匹配**成功的路线将被选择.

`key`由一组word组成, 以`.`分隔, 存在两个特殊字符:

* `*`匹配一个word
* `#`匹配0到多个word

例子:

![img](.Message%20Queue/python-five.png)

> * `lazy.orange.elephant `的消息将同时发给两个队列
>
> * ` lazy.pink.rabbit `尽管与队列Q2匹配多次, 但只发一次

### headers

根据消息的头部信息进行路由, 具体不太了解....

## 消息分配

* 循环分配 ( Round-robin )

  当一个队列有多个消费者时, 消息将循环分配到每一个队列上

  ![img](.Message%20Queue/python-two.png)

  > 缺点: 如果100个任务到来, 奇数个任务都很繁重, 导致C1一直忙, C2很多时间空间.

  >  Java原生客户端中, 默认使用该方案

* 公平分配 ( Fair dispatch )

  即一个消费者同一时间内最多仅被分配一定数量的消息. 防止耗时的消息大部分集中在一个消费者上.

  > Spring AMQP中, 默认使用该方案, 一个消费者最多接收250个消息

* 在Spring AMQP中, 默认使用公平的分配方案, 即一个消费者同一时间内最多仅被分配一定数量的消息. 防止耗时的消息大部分集中在一个消费者上.

> 分配过程猜测
>
> 消息分配后, 并不会立即发送给消费者, 而是仍旧缓存在队列中, 等待消费者获取.

## 可靠性保证

即消息传递的过程中保证数据不被丢失

### 隐患

* 消费者端

  Java原生客户端中, 消费者一旦获取到消息后, 便发送确定给队列, 队列将删除该消息. 如果消息者抛出异常了, 消息将被丢失.

* Broker端

  Broker重启后, 未持久化的消息, Queue和Exchange将消息. 服务器故障, 断电也是同样的道理.

* 生产者端

  * 消息发送到不存在的Exchange, Queue中, Broker将静默忽视, 消息即丢失
  * TCP连接假死, 一致超时
  * 即使消息是持久性的, 也会丢失, 因为Broker收到消息后, 先存入缓存, 等待写入到磁盘中. 如果此时Broker故障, 消息可能将丢失

### 解决方案

* 消费者端确认

  使用手动确认的方案, 在自己的业务处理完后, 再发送确认

* Broker端持久化

  声明持久化的Exchange, Queue和消息

  > 持久化的消息在非持久化的队列中, 照样活不过Broker下一次重启

* 生产者端确认

  消息确保到达Broker后, Broker发送确认消息给生产者. 对于持久化的消息, Broker仅在消息被写入到磁盘后才发送确认消息

> 在RabbitMQ使用的AMQP 0-9-1协议中, 还提供了生产者/消费者**事务方案**来保证数据安全, 而仅提供了消费者的确认方案, 而生产者确认方案则作为AMQP的协议扩展.
>
> 并不建议使用事务, 性能销毁较严重.  
>
> 此外, 事务是同步方案, 生产者确认是异步方案.

# 三 Server配置

下面列出了很多命令, 但仍推荐使用Rabbit提供的后端网站来管理, 网站端口`15672`, 服务端口`5662`

## 安装

```shell
sudo apt-get install rabbitmq-server
```

## 用户管理

- 查看所有用户：`rabbitmqctl list_users`

- 添加用户

  ```shell
  sudo rabbitmqctl add_user sidian 123456 # 设置账号密码
  sudo rabbitmqctl set_user_tags sidian administrator # 设置用户表示
  sudo rabbitmqctl set_permissions -p / sidian ".*" ".*" ".*" # 设置权限
  ```

- 删除用户：`rabbitmqctl delete_user username`

- 修改密码：`rabbitmqctl change_password username newpassword`

## 查看信息

* 列出所有队列

  ```shell
  sudo rabbitmqctl list_queues
  ```

* 列出所有未确认的消息

  ```shell
  sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
  ```

* 查看所有Exchange

  ```shell
  sudo rabbitmqctl list_exchanges
  ```

* 查看Exchange与队列的绑定情况

  ```shell
  rabbitmqctl list_bindings
  ```

## VHost管理

* 创建

  ```shell
  rabbitmqctl add_vhost[vhost_name]
  ```

* 删除

  ```shell
  rabbitmqctl delete_vhost[vhost_name]
  ```

* 查看所有VHost

  ```shell
  rabbitmqctl list_vhosts
  ```


## Web管理网站开启

* 开启

   ```shell
   rabbitmq-plugins enable rabbitmq_management
   ```

* 重启服务

   ```shell
   systemctl restart rabbitmq-server.service
   ```

* 登陆

   本地搭建的服务用`guest/guest`登陆, 远程的请添加新用户并添加了权限后再登陆, 服务端口:`15672`

   > 默认配置中, `guest`账号只能在localhost中登陆

## Windows安装

1. 安装依赖[Erlang](https://www.erlang.org/downloads)

2. 安装[RabbitMQ](https://www.rabbitmq.com/download.html)

   > 它会默认注册并运行RabbitMQ服务

3. 安装可视化界面

   ```shell
   rabbitmq-plugins.bat enable rabbitmq_management
   ```

4. 其他命令

   1. 检测状态

      ```shell
      rabbitmqctl.bat status
      ```

   2. 其他命令

      ![image-20200413124225426](.Message%20Queue/image-20200413124225426.png)

> 参考[RabbitMQ的安装和配置化可视界面](https://www.cnblogs.com/wade-luffy/p/6003668.html) 		

# 四 Spring AMQP

该章节着重介绍Spring Boot中Spring AMQP的使用

## 介绍

* 介绍

  Spring-AMQP是消息队列协议AMQP的抽象概括, Spring-Rabbit是一个使用RabbitMQ的实现.

* JMP Vs. AMQP

  JMS是J2EE的一部分 , AMQP是一个协议, 对使用不同MQ的限制较少.

  > 参考[JMS vs AMQP](https://www.linkedin.com/pulse/jms-vs-amqp-eran-shaham)

* 功能
  
  * 提供`RabbitTemplate`进行消息的发送和接收
  * 提供队列的监听器, 用于异步接收消息
  * 提供`RabbitAdmin`自动声明Queue, Exchange和Binding
  
* SpringBoot的Starter依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
  ```

## 使用(入门)

### 配置

一般使用Spring Boot的默认配置即可, 也可修改配置

* 配置文件方式

  默认以用户`guest/guest`连接到`localhost:15672`的RabbitMQ服务上, 可使用以下属性修改

  * `spring.rabbitmq.host` 连接的主机
  * `spring.rabbitmq.username` 用户名
  * `spring.rabbitmq.password` 用户密码

* Java代码方式

  手动注入`CachingConnectionFactory`Bean即可

> 见[Integration properties](https://docs.spring.io/spring-boot/docs/2.2.1.RELEASE/reference/html/appendix-application-properties.html#integration-properties)

### 声明

接着声明所需要Queue, Exchange和Binding即可, 在Spring AMQP都存在对应的类.

声明方式有:

* 配置`CachingConnectionFactory`

* 使用`RabbitAdmin`声明

* ( 推荐 ) 注入Queue, Exchange和Binding的Bean, Spring Boot会自动配置这些实体

  >在最简情况下, 仅需声明`Queue`即可, 因为存在默认的Exchange, 且未执行Binding时, 默认会将Queue连接到默认Exchange上.

### 发送

发送只能靠`RabbitTemplate`了.

关于默认的序列化行为

* 如果是字符串, 则根据utf-8解码成字节码
* 如果是可序列化对象, 则使用原生的JDK序列化器序列化

### 接收

* 当`@RabbitListener`注解在Bean的方法上时

  该方法用于处理监听到的消息

* 当`@RabbitListener`注解在类上时

  在方法上使用`@RabbitHandler`注解来声明为消息处理器

* `@RabbitListener`的`queues`元素指定要监听的队列, 可以使用SpEL表达式, 如

  ```java
  @RabbitListener(queues = "#{autoDeleteQueue1.name}")
  public void receive1(String in) throws InterruptedException {
      receive(in, 1);
  }
  ```



## 可靠性保证

* 消费者确认

  * Spring AMQP默认在监听器执行完后才发送确认消息, 而非获取消息时.

  * 一般情况下, 当抛出异常时, 监听器将发送拒绝确认. 而消息将重回队列, 等待重新分发
  * 当抛出` AmqpRejectAndDontRequeueException `异常时, 消息则不会重回队列

* 消息持久化

  Spring AMQP默认开启了消息持久化, 但需要使用者保证存储消息的队列也是持久化的( durable ).

* 生产者确认

  默认未开启确认功能, 详细使用见4.4.3小节-生产者确认

## 使用(进阶)

这里暂不完成, 以后有空了再写

### 序列化器

### 自动重试

### 生产者确认

* Publisher Confirms vs. Publisher Returns

  * 默认行为

    生产者将消息发送到Broker的Exchange后, Exchange发现该消息是不可路由的(即没有对应的Queue), 将忽略. 

  * Publisher Confirms

    开启确认功能. Broker收到消息, 会发送确认, 不可路由消息也发确认.

    > 配置:
    >
    > ```
    > spring.rabbitmq.publisher-confirms=true
    > ```

  * Publisher Returns

    接着开启Publisher Returns后, 此时消息是强制的 ( mandatory ), 将在发送确认之前发送return
    
    > 配置:
    >
    > ```
    > spring.rabbitmq.publisher-returns=true
    > ```

* 使用( 仅介绍Publisher Confirms)

  1. 先开启确认功能

     ```proper
     spring.rabbitmq.publisher-confirms=true
     ```

  2. 程序开始前, 先初始化确认监听器, 如

     ```java
     this.rabbitTemplate.setConfirmCallback((correlation, ack, reason) -> {
     			if (correlation != null) {
     				System.out.println("Received " + (ack ? " ack " : " nack ") + "for correlation: " + correlation);
     			}
     		});
     ```

  3. 与正常发送消息一样, 但此时多传了个一个`CorrelationData `, 设置其`id`, 用于唯一标识消息. 当生产者收到确认时, 可通过该对象的`id`得知是哪个消息.

     ```java
     CorrelationData correlationData = new CorrelationData("Correlation for message 1");
     this.rabbitTemplate.convertAndSend("", QUEUE, "foo", correlationData);
     ```

> 我觉得也不必处理Publisher Returns这种情况, 只要程序初始化时配置好了, 是可以避免这种问题的.

> 上述给出了异步处理方案, Publisher Confirms也可以同步处理确认, 详细见链接

> 参考
>
> * [What is publisher Returns in Spring AMQP](https://stackoverflow.com/questions/50460230/what-is-publisher-returns-in-spring-amqp) 介绍了Returns与Confirms的区别
> * [Demo](https://github.com/spring-projects/spring-amqp-samples/blob/master/spring-rabbit-confirms-returns/src/main/java/org/springframework/amqp/samples/confirms/SpringRabbitConfirmsReturnsApplication.java) 生产者确认的Demo源代码

## 使用例子

见参考中的官方Demo源代码链接

# 其他

## 信道

每个生产者,消费者连接到RabbitMQ时, 会建立TCP连接, 并认证, 通过后, 便接着创建一条**AMQP信道Channel**. 数据的传输都是在该信道上完成的.

> 一个TCP连接可创建一个AMQP连接, 一个AMQP连接可创建多个信道
>
> ![img](.Message%20Queue/rabbit_channel.png)
>
> 信道连接代理与生产者(或消费者), 是处理消息的工作单元.

## 消息分配

一个队列多个消费者的分发算法

* 轮循

  消息进入队列时, 就已经采用轮循的算法分配好了消息, 等待消费者获取.

  > 分配的消息并不会被消费者一次性消费完

* 公平分发(默认)

  一个消费者默认最多仅分配250个消息, 由

  `AbstractMessageListenerContainer `的`DEFAULT_PREFETCH_COUNT `确定

  > 通过查看信道中消费者已分配但未处理的消息数目得知还可分配多少个消息.

# 参考

* 入门
  * [一个用消息队列 的人，不知道为啥用 MQ，这就有点尴尬](https://blog.csdn.net/alinshen/article/details/80583214)
  * [RabbitMQ Tutotial](https://www.rabbitmq.com/getstarted.html)
  * [RabbitMQ系列文章](https://www.cnblogs.com/vipstone/p/9350075.html)
* 进阶
  * [RabbitMQ Server Config](https://www.rabbitmq.com/configure.html)
  * [Messaging with Spring AMQP](https://www.baeldung.com/spring-amqp)
  * [Consumer Acknowledgements and Publisher Confirms](https://www.rabbitmq.com/confirms.html)

* 官方Demo的源代码

  * [spring-amqp-samples](https://github.com/spring-projects/spring-amqp-samples)   

  * [rabbitmq-tutorials](https://github.com/rabbitmq/rabbitmq-tutorials)











