# 介绍

* 目的

  > Spring Data’s mission is to provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store. 

* Spring Data提供了一个抽象层, 允许底层存在不同的数据访问技术, 如 
  * relational and non-relational databases,
  *  map-reduce frameworks
  * cloud-based data services 
* Spring Data之下有很多子项目, 都是对Spring Data规范的一种实现.

# Spring Data Commons

Spring Data是抽象接口,类等等元数据定义的地方, 规定了Spring Data所有项目通用的使用方法. 该项目被其他所有子项目所依赖.

主要内容:

* 底层数据模型如何映射到对象模型, 由其他子项目规定和实现. Spring Data Commons规定了对象的创建, 属性填充等过程

* 如何使用Repository

  1. 声明继承` Repository `的接口, 类型形参传入实体类和ID类型

  2. 可自行依照规定补充数据方法.

     > 这些方法将在第3步, 由框架实现

  3. 配置, 去生成接口的代理实例对象

     ```javascript
     @EnableJpaRepositories
     class Config { … }
     ```

     > * 这里启动`Jap`,可自行更改, 格式: ` @Enable${store}Repositories `, 如` @EnableRedisRepositories `
     >
     > * 未指定要扫描的包时, 它会扫描当前类所在的包, 并生成实例
     >
     > * 貌似JPA中可不用这个注解, 接口有`@Repository`标注也会生效.

  4. 注入实例到其他对象中使用.

>参考[Spring Data Commons - Reference Documentation](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#project)

# Spring Data JPA

Java Persistent API规范的一种实现, 让使用者仅通过操作实体对象便可实现对数据库的操作.

> 参考https://www.jianshu.com/p/c14640b63653 

# Spring Data Redis

## 介绍

Spring Data Redis 将Redis与Spring集合, 并简化了Redis的使用. 它同时提供了低层次和高层次的抽象, 来与Redis交互.

特性:

*  Redis [implementation](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:support:cache-abstraction) for Spring 3.1 **cache abstraction**. 

  > 我比较感兴趣

*  [RedisTemplate](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:template) that provides a **high-level abstraction** for performing various Redis operations, exception translation and serialization support. 

  > 有点感兴趣

* Automatic implementation of `Repository` interfaces including support for custom query methods using `@EnableRedisRepositories`. 

  > 微微有点兴趣

*  Connection package as **low-level abstraction** across multiple Redis drivers([Lettuce](https://github.com/lettuce-io/lettuce-core) and [Jedis](https://github.com/xetorthio/jedis)). 

  > 默认使用Lettuce

*  [Exception](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:connectors) translation to Spring’s portable Data Access exception hierarchy for Redis driver exceptions. 

* [Pubsub](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#pubsub) support (such as a MessageListenerContainer for message-driven POJOs).

* [Redis Sentinel](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:sentinel) and [Redis Cluster](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#cluster) support.

* 

## Low-Level Connection

* 连接Redis

  * `RedisConnection` & ` RedisConnectionFactory `

  * 配置Lettuce连接器

    ```java
    @Configuration
    class AppConfig {
    
      @Bean
      public LettuceConnectionFactory redisConnectionFactory() {
    
        return new LettuceConnectionFactory(new RedisStandaloneConfiguration("server", 6379));
      }
    }
    ```

  * 配置Jedis连接器

    ```java
    @Configuration
    class RedisConfiguration {
    
      @Bean
      public JedisConnectionFactory redisConnectionFactory() {
    
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration("server", 6379);
        return new JedisConnectionFactory(config);
      }
    }
    ```

  * 主写, 从读配置

    ```java
    @Configuration
    class WriteToMasterReadFromReplicaConfiguration {
    
      @Bean
      public LettuceConnectionFactory redisConnectionFactory() {
    
        LettuceClientConfiguration clientConfig = LettuceClientConfiguration.builder()
          .readFrom(SLAVE_PREFERRED)
          .build();
    
        RedisStandaloneConfiguration serverConfig = new RedisStandaloneConfiguration("server", 6379);
    
        return new LettuceConnectionFactory(serverConfig, clientConfig);
      }
    }
    ```

* 哨兵模式支持

  ```java
  /**
   * Jedis
   */
  @Bean
  public RedisConnectionFactory jedisConnectionFactory() {
    RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
    .master("mymaster")
    .sentinel("127.0.0.1", 26379)
    .sentinel("127.0.0.1", 26380);
    return new JedisConnectionFactory(sentinelConfig);
  }
  
  /**
   * Lettuce
   */
  @Bean
  public RedisConnectionFactory lettuceConnectionFactory() {
    RedisSentinelConfiguration sentinelConfig = new RedisSentinelConfiguration()
    .master("mymaster")
    .sentinel("127.0.0.1", 26379)
    .sentinel("127.0.0.1", 26380);
    return new LettuceConnectionFactory(sentinelConfig);
  }
  ```

## High-Level Template

` RedisConnection `仅接收和返回二进制值 (`byte`数组) , 而`template`负责数据的序列化, 连接的事务管理和提供了不同Redis数据类型的操作接口.

| Interface               | Description                                                  |
| :---------------------- | :----------------------------------------------------------- |
| *Key Type Operations*   |                                                              |
| `GeoOperations`         | Redis geospatial operations, such as `GEOADD`, `GEORADIUS`,… |
| `HashOperations`        | Redis hash operations                                        |
| `HyperLogLogOperations` | Redis HyperLogLog operations, such as `PFADD`, `PFCOUNT`,…   |
| `ListOperations`        | Redis list operations                                        |
| `SetOperations`         | Redis set operations                                         |
| `ValueOperations`       | Redis string (or value) operations                           |
| `ZSetOperations`        | Redis zset (or sorted set) operations                        |
| *Key Bound Operations*  |                                                              |
| `BoundGeoOperations`    | Redis key bound geospatial operations                        |
| `BoundHashOperations`   | Redis hash key bound operations                              |
| `BoundKeyOperations`    | Redis key bound operations                                   |
| `BoundListOperations`   | Redis list key bound operations                              |
| `BoundSetOperations`    | Redis set key bound operations                               |
| `BoundValueOperations`  | Redis string (or value) key bound operations                 |
| `BoundZSetOperations`   | Redis zset (or sorted set) key bound operations              |







> 参考[Spring Data Redis](https://docs.spring.io/spring-data/redis/docs/2.2.0.RELEASE/reference/html/#introduction)







