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

  请求时边写入数据库, 造成数据库压力山大

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

中小型软件公司建议用RabbitMQ, 其开源且社区比较活跃; 大型软件公司建议用RocketMQ或Kafka, 吞吐量比较大.

## 其他

* 如何保证高可用? 集群, 主从复制

# 二 RabbitMQ

## Demo

* 介绍: 简单的展示, 生产者发送消息, 消费者接收消息

  ![python-one.png](.Message%20Queue/python-one-1574665345043.png)

* 目的: 入门

* 引入Java客户端

  ```xml
  <dependency>
      <groupId>com.rabbitmq</groupId>
      <artifactId>amqp-client</artifactId>
      <version>5.7.3</version>
  </dependency>
  ```

  > 服务端需自行搭建并配置

* Server

  ```java
  public class Sender {
      private final static String QUEUE_NAME = "hello";
  
      public static void main(String[] argv) throws Exception {
          //set config
          ConnectionFactory factory = new ConnectionFactory();
          factory.setHost("sidian123.geely.com");
          factory.setUsername("sidian");
          factory.setPassword("123456");
          //connect to server
          try (Connection connection = factory.newConnection();
               //acquire channel, like acquire socket
               Channel channel = connection.createChannel()) {
               //get queue for send
              channel.queueDeclare(QUEUE_NAME, false, false, false, null);
              String message = "Hello World!";
              //send
              channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
              System.out.println(" [x] Sent '" + message + "'");
  
          }
      }
  }
  ```

* Client

  ```java
  public class Receiver {
      private final static String QUEUE_NAME = "hello";
  
      public static void main(String[] argv) throws Exception {
          //set config
          ConnectionFactory factory = new ConnectionFactory();
          factory.setHost("sidian123.geely.com");
          factory.setUsername("sidian");
          factory.setPassword("123456");
          //connect to server, but without try-width-resource to close
          Connection connection = factory.newConnection();
          //acquire channel, like socket
          Channel channel = connection.createChannel();
          //get queue to receive
          channel.queueDeclare(QUEUE_NAME, false, false, false, null);
          System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
          //start to receive
          channel.basicConsume(QUEUE_NAME, true, (consumerTag, delivery) -> {//callback of received
              String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
              System.out.println(" [x] Received '" + message + "'");
          }, consumerTag -> { });
      }
  
  }
  ```

  > 消费者不释放资源的原因: 消费者异步监听队列后, 将存在子线程, 主线程的main()方法结束了, 主线程也不会结束, 而是等待子线程退出. 因此不应该在main()方法中释放资源

## 进阶

### 消息分配

当一个队列有多个消费者时, RabbitMQ默认以轮询的方式分配消息

![img](.Message%20Queue/python-two.png)

> 缺点: 如果100个任务到来, 奇数个任务都很繁重, 导致C1一直忙, C2很多时间空间.

可配置, 让消费者空闲时就被分配一个

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

> `prefetchCount`指定一个消费者一次最多能分配到的个数.

------

> 分配过程猜测
>
> 分配后, 如果消费者处理不过来, 则消息仍留在队列中, 消费者

### 消息确认机制

该机制在**消费者**监听队列时默认开启

### 消息持久性

### 消息模型

* 除了, 生产者, 消费者, 和队列, 还存在 ***exchange*** 这一中间层, 添加灵活性

	![img](.Message%20Queue/exchanges.png)

* 实际生产者并不直接将消息存入队列, 而是交给Exchange, 由Exchange决定消息推入哪个队列, 或不推送.

* 种类:

    * `direct`

    * `topic`

    * `headers`

    * `fanout` : 广播的方式将消息推送到所有已知的队列中

    * 空名的Exchange(默认): 即`''`, 直接将消息推送到 r`outingKey` 标识的队列, 如果存在的话. 如

      > ```shell
      > channel.basicPublish("", "hello", null, message.getBytes());
      > ```
      >
      > 第一个为Exchange名, 第二个参数为`routingKey`

* 非默认Exchange与队列之间需要消费者显示绑定

    ```java
    channel.queueBind(queueName, "logs", "");
    ```

## 路由

### direct Exchange

* 队列绑定Exchange时, 指定` binding key `, 生产者生产的消息将被路由到`binding key`与消息`routing key`一致的队列上.

* 例子

  * 不同消息推送到不同队列

    ![img](.Message%20Queue/direct-exchange.png)

  * `routing key`为black的消息广播给所有队列, 其他忽略

    ![img](.Message%20Queue/direct-exchange-multiple.png)

  * 按照日志的严重级别推送到不同的队列中

    ![img](.Message%20Queue/python-four.png)

### topic Exchange

消息的`routing_key`与队列的`bind_key`**模式匹配**成功的路线将被选择.

`key`由一组word组成, 以`.`分隔, 存在两个特殊字符:

* `*`匹配一个word
* `#`匹配0到多个word

例子:

![img](.Message%20Queue/python-five.png)

> * `lazy.orange.elephant `的消息将同时发给两个队列
>
> * ` lazy.pink.rabbit `尽管与队列Q2匹配多次, 但只发一次









## Server

### 创建新账号

```shell
$ rabbitmqctl add_user YOUR_USERNAME YOUR_PASSWORD
$ rabbitmqctl set_user_tags YOUR_USERNAME administrator
$ rabbitmqctl set_permissions -p / YOUR_USERNAME ".*" ".*" ".*"
```

### 查看

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

  















## 其他

### 登陆失败

默认账号密码都是 `guest`, 但只能本地登陆, 若远程登陆需重新创建账号密码, 如:

```shell
$ rabbitmqctl add_user YOUR_USERNAME YOUR_PASSWORD
$ rabbitmqctl set_user_tags YOUR_USERNAME administrator
$ rabbitmqctl set_permissions -p / YOUR_USERNAME ".*" ".*" ".*"
```

> 参考[Spring AMQP + RabbitMQ 3.3.5 ACCESS_REFUSED - Login was refused using authentication mechanism PLAIN](https://stackoverflow.com/questions/26811924/spring-amqp-rabbitmq-3-3-5-access-refused-login-was-refused-using-authentica)

# 三RabbitMQ学习2

## Ubuntu环境搭建

* 安装

  ```shell
  sudo apt-get install rabbitmq-server
  ```

* 用户管理

  - 查看所有用户：`rabbitmqctl list_users`
  - 添加用户：`rabbitmqctl add_user username password`
  - 删除用户：`rabbitmqctl delete_user username`
  - 修改密码：`rabbitmqctl change_password username newpassword`

* 开启网页

  * 开启` rabbitmq-plugins enable rabbitmq_management `
  * 重启服务`systemctl restart rabbitmq-server.service`
  * 登陆: 本地搭建的服务用`guest/guest`登陆, 远程的请添加新用户并添加了权限后再登陆

* 例子: 添加用户并赋予所有权限

  ```shell
  rabbitmqctl add_user test test
  rabbitmqctl set_user_tags test administrator
  rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
  ```

## 工作原理与使用

### 介绍

*  RabbitMQ是一个实现了AMQP（Advanced Message Queuing Protocol）高级消息队列协议的消息队列服务
* 由Erlang语言编写的。 

### 使用场景

商品秒杀时, 为了避免某个时间断大量的数据查询导致数据库宕机, 而是先将操作入队列, 一个一个执行.

### 重要概念

* 生产者：消息的创建者，负责创建和推送数据到消息服务器；
* 消费者：消息的接收方，用于处理数据和确认消息；

* 代理 ( broker )：就是RabbitMQ本身，用于扮演“快递”的角色，本身不生产消息，只是扮演“快递”的角色。

### 信道

每个生产者,消费者连接到RabbitMQ时, 会建立TCP连接, 并认证, 通过后, 便接着创建一条**AMQP信道Channel**. 数据的传输都是在该信道上完成的.

> 一个TCP连接可创建一个AMQP连接, 一个AMQP连接可创建多个信道
>
> ![img](.Message%20Queue/rabbit_channel.png)
>
> 信道连接代理与生产者(或消费者)

### RabbitMQ核心概念

* **ConnectionFactory（连接管理器）：**应用程序与Rabbit之间建立连接的管理器，程序代码中使用；

* **Channel（信道）：**消息推送使用的通道；

* **Exchange（交换器）：**用于接受、分配消息；

* **Queue（队列）**：用于存储生产者的消息；

* **RoutingKey（路由键）**：用于把生成者的数据分配到交换器上；

* **BindingKey（绑定键）**：用于把交换器的消息绑定到队列上；

![img](.Message%20Queue/rabbit-producer.gif)

### 数据可靠传输(防止丢数据)

#### 防止生产者丢数据

即生产者在发送给RabbitMQ时, RabbitMQ未收到, 即数据丢失

* 事务: 信道Channel提供了事务的功能, 即
  * ` channel.txSelect() `开启事务
  * ` channel.txRollback() `回滚事务
  * ` channel.txCommit() `提交事务
* Confirm模式, 见[Publisher Confirms](https://www.rabbitmq.com/tutorials/tutorial-seven-java.html)

> `Channel.basicPublish()`抛出的`IOException`不能作为确认的标志吗? 应该不能, 呵呵...

#### 防止消息队列丢数据

即, RabbitMQ重启或突然挂了时, 数据会丢失.

* 首先, 先保证队列是持久的, 即`Channel.queueDeclare()`的`durable`参数为`true`

* 现在保证生产者推送的消息是持久的, 如

  ```java
  channel.basicPublish("", "task_queue",
              MessageProperties.PERSISTENT_TEXT_PLAIN,
              message.getBytes());
  ```

  > 即设置`MessageProperties.PERSISTENT_TEXT_PLAIN`属性

#### 消费者丢数据



# 参考

* [一个用消息队列 的人，不知道为啥用 MQ，这就有点尴尬](https://blog.csdn.net/alinshen/article/details/80583214)
* [RabbitMQ Tutotial](https://www.rabbitmq.com/getstarted.html)
* [RabbitMQ Server Config](https://www.rabbitmq.com/configure.html)
* [RabbitMQ系列文章](https://www.cnblogs.com/vipstone/p/9350075.html)















