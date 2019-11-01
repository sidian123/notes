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

Spring Data Redis 将Redis与Spring集合, 屏蔽了底层Redis客户端API的使用, 提供了多种操作Redis的方式, 极大简化了Redis的使用.

Spring提供的使用方式有:

* 支持Spring 缓存抽象层的注解
* 提供高级的API访问入口, `RedisTemplate`
* 提供Spring Data风格的使用方式, `Repository`
* 提供低级的API接口, `RedisConnection`
* 提供了`Collection`和`Atomic`接口的实现, 可达到无感知使用Redis

> Spring Data Redis很是灵活, 使用哪种方式取决于使用者.

> 关于底层客户端`Lettuce`和`Jedis`, 默认使用`Lettuce`

## 序列化

为何数据需要序列化? 

尽管Redis的值有数据类型之分, 但Redis数据类型仅规定了**元素间**在内存中是如何分布的, 但是**元素本身**在Redis的内存中就是一个**`byte`数组**. Java的对象如何转化为Redis的`byte`数据交给了使用者, 这个过程就是序列化.

序列化包括键和值的序列化. Spring Data Redis提供了多种序列化器, 如下所示:

* `JdkSerializationRedisSerializer` 使用JDK原生的序列化器, 被序列化的类需要实现`Serializable`接口.

  > `RedisCache`和`RedisTemplate`默认使用这个

* `StringRedisSerializer` 仅用于操作字符串, 使用`utf-8`字符编码来序列化对象.

  > `StringRedisTemplate`默认使用这个

* ` Jackson2JsonRedisSerializer `或` GenericJackson2JsonRedisSerializer ` 使用Jackson和`utf-8`序列化对象为JSON格式
* ` OxmSerializer ` 序列化对象为XML格式

## 缓存注解

Spring Data Redis主要通过实现` cache `和` RedisCacheManager `接口, 来提供Spring缓存抽象层的实现.

使用方法见[Spring Core.md](Spring Core.md)

## Template

`RedisTemplate`是访问Redis的一个高级接口, 通过该接口获取其**操作视图**即可, 每个操作视图都对应Redis一种类型的所有操作.

> Spring Data Redis提供了个Trick, 能让`RedisTemplate`直接注入操作视图中

大部分操作都是字符串操作, 因此提供了`StringRedisTemplate`, 使用`StringRedisSerializer`作为序列化器.

所有的操作视图:

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



## Connection

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

> 通过`Connection`可直接获取底层的客户端, 如Jedis

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

### DefaultSerializer需要序列化的负载

让存储到Redis的类实现`Serializable `接口即可.

## 参考

* [Spring Data Redis](https://docs.spring.io/spring-data/redis/docs/2.2.0.RELEASE/reference/html/#introduction)
* [Spring Boot（八）集成Spring Cache 和 Redis](https://www.cnblogs.com/ashleyboy/p/9595584.html)
* [Cache Abstraction](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/integration.html#cache)



