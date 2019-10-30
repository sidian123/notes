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

* ...

## 配置

SpringBoot会自动配置并注入所有所需的Bean, 只需引入对应starter jar包即可.

可以通过Java配置方式配置细节设置, 但推荐通过SpringBoot配置文件来配置.

自动注入的其中有`RedisTemplate`, `StringRedisTemplate`

## 连接Redis

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

  > 运行会报错, 建议用Lettuce

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

## High-Level Template

### 介绍

` RedisConnection `仅接收和返回二进制值 (`byte`数组) , 而`template`负责数据的序列化, 连接的事务管理和提供了不同Redis数据类型的操作视图.

所有**操作视图**如下:

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

### 使用

可直接注入`RedisTemplate`, 或者注入上述操作视图.

> 注意, `RedisTemplate`使用的序列化器并不会将键值对转化为非字符串, 因此在`Redis-cli`中通过字符串获取不到值.

## 便捷类

实际上, 大部分存储在Redis中的键值对都是以字符串存储的, 因此Spring提供了两个便捷类: ` StringRedisConnection `和`  StringRedisTemplate ` 

它的键值都是字符串, 并且使用`utf-8`序列化键值对.

## Low-Level Connection

想获取对Redis全面的控制时, 可通过` RedisTemplate `或` StringRedisTemplate `获取连接, 如

```java
public void useCallback() {

  redisTemplate.execute(new RedisCallback<Object>() {
    public Object doInRedis(RedisConnection connection) throws DataAccessException {
      Long size = connection.dbSize();
      // Can cast to StringRedisConnection if using a StringRedisTemplate
      ((StringRedisConnection)connection).set("key", "value");
    }
   });
}
```

## 序列化

在框架看来, Redis仅存字节数组, 而Redis的数据结构是指数据的存储方式, 而不是单个元素的存储结构.

> 如字符串数组在Redis中以数组结构储存, 但是单个字符串怎么存呢? 以什么样的格式?

因此Spring提供了序列化器, 用于序列化单个对象. 以下是最为常用的

- `JdkSerializationRedisSerializer`, which is used by default for `RedisCache` and `RedisTemplate`.

  > 使用Java原生序列化工具

- the `StringRedisSerializer`.

  > 使用`utf-8`序列化字符串

> Spring 推荐最好将数据存储为Json格式, 怎么存? 不知道

## 缓存支持

[Support for the Spring Cache Abstraction](https://docs.spring.io/spring-data/redis/docs/2.2.0.RELEASE/reference/html/#redis:support:cache-abstraction)

## 踩坑

### 设置键值后, redis-cli中不能取出

因为`RedisTemplate`的序列化器`RedisSerializer`序列化后, 键值都不同了. 

可以考虑使用它的便捷类`StringRedisTemplate`, 它使用`utf-8`进行序列化, 因此redis-cli中键值都不变.

> 参考[Using RedisTemplate set a value but get Nil from Terminal Redis-CLI](https://stackoverflow.com/questions/34736265/using-redistemplate-set-a-value-but-get-nil-from-terminal-redis-cli)

### RedisTemplate与ListOperations类型不一致, 却能注入

这是因为`PropertyEditorSupport`会干涉Spring IOC容器字段注入的过程: 

```java
class ListOperationsEditor extends PropertyEditorSupport {
    ListOperationsEditor() {
    }

    public void setValue(Object value) {
        if(value instanceof RedisOperations) {
            super.setValue(((RedisOperations)value).opsForList());
        } else {
            throw new IllegalArgumentException("Editor supports only conversion of type " + RedisOperations.class);
        }
    }
}
```

> 参考
>
> * [Why a “RedisTemplate” can convert to a “ListOperations”](https://stackoverflow.com/questions/43006197/why-a-redistemplate-can-convert-to-a-listoperations)
>
> * [Custom PropertyEditors Spring Example](https://www.concretepage.com/spring/custom-propertyeditors-spring-example)

# 参考

* [Spring Data Redis](https://docs.spring.io/spring-data/redis/docs/2.2.0.RELEASE/reference/html/#introduction)
* [Spring Boot（八）集成Spring Cache 和 Redis](https://www.cnblogs.com/ashleyboy/p/9595584.html)
* [Cache Abstraction](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/integration.html#cache)



