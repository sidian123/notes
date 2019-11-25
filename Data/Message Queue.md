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







## Server

### 创建新账号

```shell
$ rabbitmqctl add_user YOUR_USERNAME YOUR_PASSWORD
$ rabbitmqctl set_user_tags YOUR_USERNAME administrator
$ rabbitmqctl set_permissions -p / YOUR_USERNAME ".*" ".*" ".*"
```

### 查看队列

* 列出所有队列

  ```shell
  sudo rabbitmqctl list_queues
  ```

* 列出所有未确认的消息

  ```shell
  sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
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





# 参考

* [一个用消息队列 的人，不知道为啥用 MQ，这就有点尴尬](https://blog.csdn.net/alinshen/article/details/80583214)

* [RabbitMQ Tutotial](https://www.rabbitmq.com/getstarted.html)

* [RabbitMQ Server Config](https://www.rabbitmq.com/configure.html)















