# 介绍

* 目的

  > Spring Data’s mission is to provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store. 

* Spring Data提供了一个抽象层, 允许底层存在不同的数据访问技术, 如 
  * relational and non-relational databases,
  *  map-reduce frameworks
  * cloud-based data services 
* Spring Data之下有很多子项目, 都是对Spring Data规范的一种实现.

# Spring Data Commons

## 入门

* 介绍

  Spring Data是抽象接口,类等等元数据定义的地方, 规定了Spring Data所有项目通用的使用方法. 该项目被其他所有子项目所依赖.

* 特点

  * 提供Repository和自定义对象映射抽象
  * 从Repository派生出的动态查询

* 内容

  底层数据模型如何映射到对象模型, 由其他子项目规定和实现. 

  Spring Data Commons规定了对象的创建, 属性填充等过程

* 如何使用Repository

  1. 声明继承` Repository `的接口, 类型形参传入实体类和ID类型

  2. 可自行依照规定补充数据方法, Spring Data子项目将自动实现这些方法.

     > 这些方法将在第3步, 由框架实现

  3. 启动功能, 如生成`@Repository`标注接口的代理实例对象

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
  
* 参考

  [Spring Data Commons - Reference Documentation](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#project)

## 深入

### Dao层定义

* Dao类标注

  使用Dao类继承一下接口即可, 不同接口提供了不同的功能, 或聚合了其他接口功能. 其他接口都派生于`Repository`接口.

  * `Repository`接口

    用于标注实体类, 获取实体和ID类型

  * `CrudRepository`接口

    提供CRUD功能.

  * `PagingAndSortingRepository`接口

    提供分页和排序功能

  * 与具体持久化相关的
  
  * `JpaRepository` 标注是JPA的Repository
    
  * `MongoRepository` 标注是Mongo的Repository
  
* dao接口定义

  * dao类必须继承`Repository`接口, 可以继承`Repository`的子接口来暴露更多的实体操作方法.  或者使用`@RepositoryDefinition`, 与dao类继承`Repository`接口是一样的效果.

* 自定义/裁剪Repository接口

  即暴露部分方法, 如`CrudRepository`的方法. 只要是底层实现支持的方法即可, 如JPA中`SimpleJpaRepository`的方法

  ```java
  @NoRepositoryBean
  interface MyBaseRepository<T, ID> extends Repository<T, ID> {
  
    Optional<T> findById(ID id);
  
    <S extends T> S save(S entity);
  }
  
  interface UserRepository extends MyBaseRepository<User, Long> {
    User findByEmailAddress(EmailAddress emailAddress);
  }
  ```

  自定义的接口以`@NoRepositoryBean`标注, 

* 多Spring Data Module

  当使用了多个Spring Data Module, 如JPA, Elasticsearch, 那么dao类使用哪个数据源呢? 有三种方法区分

  * dao类继承和具体模块相关的`Repository`接口, 如JPA的`JpaRepository`

  * 实体类被具体模块相关的注解, 如JPA的`@Entity`, MongoDB的`@Document`

    > 实体类若同时标注了多个模块的注解, 则需要具体的`Repository`接口来区分.

  * 约束模块扫描的包

    ```java
    @EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
    @EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
    class Configuration { … }
    ```

### 查询方法定义

* 查询策略

  由`Enable${store}Repositories`注解的`queryLookupStrategy`属性配置

  * `CREATE`

    从方法名中构建

  * `USE_DECLARED_QUERY`

    使用查询语句, 如SQL, 通常以注解来定义. 若未找到, 则抛出异常.

  * `CREATE_IF_NOT_FOUND` (默认)

    组合上述两种行为, 首先看是否有查询语句, 若无则从方法名中构建.

* 查询方法名构建

  * 动作指示

    方法名以`find…By`, `read…By`, `query…By`, `count…By`, and `get…By` 为前缀

  * 表达式

    方法名之后接表达式, 如字段名. 多个表达式以`Or`或`And`组合.

  Demo

  ```java
  interface PersonRepository extends Repository<Person, Long> {
  
    List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
  
    // Enables the distinct flag for the query
    List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
    List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
  
    // Enabling ignoring case for an individual property
    List<Person> findByLastnameIgnoreCase(String lastname);
    // Enabling ignoring case for all suitable properties
    List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
  
    // Enabling static ORDER BY for a query
    List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
    List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
  }
  ```

  



* 创建repository实例

  

# Spring Data JPA

## Getting Start

* 介绍

  Java Persistent API规范的一种实现, 让使用者仅通过操作实体对象便可实现对数据库的操作.

* 配置

  * Maven依赖

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    ```

  * 项目配置

    ```properties
    # 目标数据库名字, 默认会自动探测, 可为空
    # spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
    # 是否显示执行的SQL语句
    spring.jpa.show-sql=true
    # 对结构的操作, 默认为none
    spring.jpa.hibernate.ddl-auto=update
    ```

    `ddl-auto`可取值

    * *validate*: validate the schema, makes no changes to the database.
    * *update*: update the schema. 若表不存在, 会自动创建.
    * *create*: creates the schema, destroying previous data.
    * *create-drop*: drop the schema when the SessionFactory is closed explicitly, typically when the application is stopped.
    * *none*: does nothing with the schema, makes no changes to the database

* 启动Jpa功能

  在`@Configuration`类上添加`@EnableJpaRepositories`注解, 如

  ```java
  @EnableJpaRepositories
  @SpringBootApplication
  public class HelloSpringDataApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(HelloSpringDataApplication.class, args);
      }
  }
  ```

* 实体类创建

  ```java
  @Data
  @Builder
  @NoArgsConstructor
  @AllArgsConstructor
  @Entity
  public class User {
      @Id
      @GeneratedValue
      Integer id;
      String name;
  }
  ```

  > 使用的`javax.persistence`包的注解

* Dao类创建

  ```java
  @Repository
  public interface UserDao extends JpaRepository<User, Integer> {}
  ```
  
* 使用Demo

  ```java
  @SpringBootTest
  class HelloSpringDataApplicationTests {
      @Autowired
      UserDao userDao;
  
      @Test
      void contextLoads() {
          userDao.save(User.builder().name("张三").build());
      }
  }
  ```

## 查询方法



## 多表连接

## 自定义查询



## 其他

### 设计理念

[引言](https://stackoverflow.com/a/11881628):

> Perhaps I should elaborate on overall semantics of JPA. There are two main approaches to design of persistence APIs:
>
> - **insert/update approach**. When you need to modify the database you should call methods of persistence API explicitly: you call `insert` to insert an object, or `update` to save new state of the object to the database.
> - **Unit of Work approach**. In this case you have a set of objects *managed* by persistence library. All changes you make to these objects will be  flushed to the database automatically at the end of Unit of Work (i.e.  at the end of the current transaction in typical case). When you need to insert new record to the database, you make the corresponding object *managed*. *Managed* objects are identified by their primary keys, so that if you make an object with predefined primary key *managed*, it will be associated with the database record of the same id, and  state of this object will be propagated to that record automatically.

在Jpa中, 使用的第二种方案, JPa会监控持久化的实体, 若有修改, 在事务结束时会自动提交. 

例子:

```java
@Test
@Transactional // 必须要加上事务
// @Rollback(false) // 默认Test中事务回滚, 这里使用默认行为
void contextLoads() {
    User user = userDao.findById(1).orElse(new User());
    user.setName("杀杀杀钉钉"); // 事务结束后会同步到数据库中.
    //userDao.save(user); // 无需save
    User user1 = userDao.findById(1).orElse(new User()); // 查询的内容也是修改过的实体.
    System.out.println(user1);
}
```

输出:

```
User(id=1, name=杀杀杀钉钉)
```

那问题来了, 有人说, 修改一个数据, 需要查出来, 然后更新. 相当于查了两次, 影响性能. 那我就来反驳了... 首先查的次数远大于写, 这是毋庸置疑的. 写的次数很少, 那么这点性能消耗相对于高效的开发来说, 是可以接受的. 

多表查询呢? 目前我不清楚, 但是现在的趋势是单表查询. TODO

## 参考

* [SpringBoot 中 JPA 的使用](https://www.jianshu.com/p/c14640b63653)

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

## 配置

* Maven依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

* 配置

  ```properties
  #Server host
  spring.redis.host=localhost
  #password
  spring.redis.password=
  #Redis server port
  spring.redis.port=6379
  ```

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

### 超时

```
io.lettuce.core.RedisCommandTimeoutException: Command timed out after 1 minute(s)
```

发送Redis请求后, 服务端太久没有响应, 导致超时. 这里增大超时时间 (ms) :

```properties
spring.redis.timeout=500
```

> 有人采取切换底层实现到JRedis来解决, 在4.1.2 lettuce前是存在这个bug, 之后这个Bug已经被修复了. 因此还是采取增大超时时间的办法.

> 参考[Too many RedisCommandTimeoutException in lettuce](https://stackoverflow.com/questions/49811076/too-many-rediscommandtimeoutexception-in-lettuce)

## 参考

* [Spring Data Redis](https://docs.spring.io/spring-data/redis/docs/2.2.0.RELEASE/reference/html/#introduction)
* [Spring Boot（八）集成Spring Cache 和 Redis](https://www.cnblogs.com/ashleyboy/p/9595584.html)
* [Cache Abstraction](https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/integration.html#cache)



# Spring Data Neo4j

## Getting Start

* 介绍
  
* 支持Spring Data Common的Repository使用方式, 支持派生方法
  
* 配置

  ```properties
  spring.data.neo4j.username=neo4j
  spring.data.neo4j.password=123456
  # spring.data.neo4j.uri=neo4j://localhost:7687
  ```

  > 若neo4j运行在本地, 且端口7687, 那么uri配置可省略.

* 依赖引入

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-neo4j</artifactId>
  </dependency>
  ```

* 实体定义

  ```java
  import org.neo4j.ogm.annotation.GeneratedValue;
  import org.neo4j.ogm.annotation.Id;
  import org.neo4j.ogm.annotation.NodeEntity;
  import org.neo4j.ogm.annotation.Relationship;
  
  @Data
  @Builder
  @NoArgsConstructor
  @AllArgsConstructor
  @NodeEntity
  public class Person {
  
  	@Id @GeneratedValue
  	private Long id;
  
  	private String name;
  
      @EqualsAndHashCode.Exclude
  	@Relationship(type = "TEAMMATE", direction = Relationship.UNDIRECTED)
  	public Set<Person> teammates;
  }
```
  
* `NodeEntity`注解
  
  定义Neo4j的实体类
  
* `Id`, `GeneratedValue`注解
  
  `Id`声明主键, 是一个唯一约束. 当保存实体时, 会触发`merge`语句, 存在实体则修改, 无则新增.
  
    `GeneratedValue`表示新增时主键自动产生
  
  * `Relationship`注解
  
    声明实体间的关系. 
  
    原则上, Neo4j的关系除了有类型, 方向外, 还支持属性. 但这里没有提供关系属性定义的支持.
  
    在Neo4j中, 关系不支持双向(或无向)的, 因此这里的`Relationship.UNDIRECTED`仅表示查询时无向(即任意方向匹配) , 新增时仍为`OUTGOING`(流出).
  
    > 最好不要用`UNDIRECTED`, 容易入坑
  
* Dao类定义

  ```java
  public interface PersonRepository extends CrudRepository<Person, Long> {
  	Person findByName(String name);
  }
  ```

  符合Spring Data Commons规范, 可自行扩充方法



## 参考

* [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/) 一个了解Neo4j的Demo案例
* 









