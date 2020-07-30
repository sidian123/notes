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


## 其他

### 分页&排序

* 分页&排序

  > 注意, 页码从0开始

  ```java
  @Repository
  public interface RelationshipDao extends JpaRepository<Relationship,Long> {
      Page<Relationship> findAll(Pageable page);
  }
  ```

  ```java
  List<Relationship> relationship = relationshipDao.findBy(PageRequest.of(0,1,Sort.by(Sort.Direction.DESC,"id"))).getContent();
  ```

* 仅排序

  ```java
  age<Relationship> findAllByName(String name,Pageable page);
  ```

  ```java
  relationshipDao.findAll(Sort.by(Sort.Direction.DESC,"id"));
  ```

# Spring Data JPA

## Getting Start

### 介绍

Java Persistent API规范的一种实现, 让使用者仅通过操作实体对象便可实现对数据库的操作.

### 配置

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
  * *update*: update the schema; 但是实体类未声明的字段, 数据库中存在时, 并不会删除该字段哦. 若表不存在, 会自动创建. 
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

### 实体类创建

#### 定义

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

* 名字映射

  未显示给出名字时, 采用首字母小写, 驼峰转下划线的方案, 见`ImplicitNamingStrategyJpaCompliantImpl`
  
* `@Entity` 标注Jpa实体

* `GeneratedValue` 主键自动生成

  `strategy`属性指定生成策略, 有四种取值:

  * `GenerationType.TABLE` 使用特定的数据库表来存主键序号

  * `GenerationType.SEQUENCE` 使用数据库自身提供*序列*机制生成主键

    > Oracle支持, MySQL不支持

  * `GenerationType.IDENTITY` 插入时自增方式

    > MySQL支持, Oracle不支持

  * `GenerationType.AUTO` 让持久化引擎 (如hibernate) 自己决定. 多半使用的`Table`方式

  > 参考[GenerationType四中类型](https://www.cnblogs.com/hongchengshise/p/10612301.html)

* `@Column` 标注列

* `@Temporal` 设置`Date`映射到数据库的类型, 如`Date`(默认), `TIME`, `TIMESTAMP`

* `@Index` 声明索引

  定义多个单列索引

  ```java
  @Entity
  @Table(name = "region",
         indexes = {@Index(name = "my_index_name",  columnList="iso_code", unique = true),
                    @Index(name = "my_index_name2", columnList="name",     unique = false)})
  public class Region{
  
      @Column(name = "iso_code", nullable = false)
      private String isoCode;
  
      @Column(name = "name", nullable = false)
      private String name;
  
  } 
  ```

  声明一个聚簇索引

  ```java
  @Entity
  @Table(name    = "company__activity", 
         indexes = {@Index(name = "i_company_activity", columnList = "activity_id,company_id")})
  public class CompanyActivity{
  	...
  }
  ```


#### jdbcType <=> javaType

| Hibernate type (org.hibernate.type) | JDBC type                                            | Java type                                         |
| ----------------------------------- | ---------------------------------------------------- | ------------------------------------------------- |
| StringType                          | VARCHAR                                              | String                                            |
| MaterializedClob                    | CLOB                                                 | String                                            |
| TextType                            | LONGVARCHAR                                          | String                                            |
| CharacterType                       | CHAR                                                 | char or Character                                 |
| BooleanType                         | BIT                                                  | boolean or Boolean                                |
| NumericBooleanType                  | INTEGER (e.g. 0 = false and 1 = true)                | boolean or Boolean                                |
| YesNoType                           | CHAR (e.g. ‘N’ or ‘n’ = false and ‘Y’ or ‘y’ = true) | boolean or Boolean                                |
| TrueFalseType                       | CHAR (e.g. ‘F’ or ‘f’ = false and ‘T’ or ‘t’ = true) | boolean or Boolean                                |
| ByteType                            | TINYINT                                              | byte or Byte                                      |
| ShortType                           | SMALLINT                                             | short or Short                                    |
| IntegerType                         | INTEGER                                              | int or Integer                                    |
| LongType                            | BIGINT                                               | long or Long                                      |
| FloatType                           | FLOAT                                                | float or Float                                    |
| DoubleType                          | DOUBLE                                               | double or Double                                  |
| BigIntegerType                      | NUMERIC                                              | BigInteger                                        |
| BigDecimalType                      | NUMERIC                                              | BigDecimal                                        |
| TimestampType                       | TIMESTAMP                                            | java.sql.Timestamp or java.util.Date              |
| TimeType                            | TIME                                                 | java.sql.Time                                     |
| DateType                            | DATE                                                 | java.sql.Date                                     |
| CalendarType                        | TIMESTAMP                                            | java.util.Calendar or java.util.GregorianCalendar |
| CalendarType                        | DATE                                                 | java.util.Calendar or java.util.GregorianCalendar |
| CurrencyType                        | VARCHAR                                              | java.util.Currency                                |
| LocaleType                          | VARCHAR                                              | java.util.Locale                                  |
| TimeZoneType                        | VARCHAR                                              | java.util.TimeZone                                |
| UrlType                             | VARCHAR                                              | java.net.URL                                      |
| ClassType                           | VARCHAR                                              | java.lang.Class                                   |
| BlobType                            | BLOB                                                 | java.sql.Blob                                     |
| ClobType                            | CLOB                                                 | java.sql.Clob                                     |
| BinaryType                          | VARBINARY                                            | byte[] or Byte[]                                  |
| BinaryType                          | BLOB                                                 | byte[] or Byte[]                                  |
| BinaryType                          | LONGVARBINARY                                        | byte[] or Byte[]                                  |
| BinaryType                          | LONGVARBINARY                                        | byte[] or Byte[]                                  |
| CharArrayType                       | VARCHAR                                              | char[] or Character[]                             |
| UUIDBinaryType                      | BINARY                                               | java.util.UUID                                    |
| UUIDBinaryType                      | CHAR or VARCHAR                                      | java.util.UUID                                    |
| UUIDBinaryType                      | PostgreSQL UUID                                      | java.util.UUID                                    |
| SerializableType                    | VARBINARY                                            | Serializable                                      |

> 参考[A beginner’s guide to Hibernate Types](https://vladmihalcea.com/a-beginners-guide-to-hibernate-types/)

### Dao类创建

```java
@Repository
public interface UserDao extends JpaRepository<User, Integer> {}
```

### 使用Demo

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

### 创建

| Keyword                | Sample                                                       | JPQL snippet                                                 |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `And`                  | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`, `Equals`         | `findByFirstname`,`findByFirstnameIs`,<br/>`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`        | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`          | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`                | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`               | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`, `Null`       | `findByAge(Is)Null`                                          | `… where x.age is null`                                      |
| `IsNotNull`, `NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`                 | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`              | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`         | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`           | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`           | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`              | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                   | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`                | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
| `True`                 | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`                | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`           | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

### 分页 & 排序

* 分页, 需要传入`pageable`类型的对象, 如 

  ```java
  symptomDao.findAll(PageRequest.of(0,10));
  ```

  > 注意, 分页从`0`开始

## 其他操作

### save

执行`save()`方法时, 若对象不存在, 则新增; 若存在, 则更新.

更新时, 为了防止一些字段被更新, 可以配置字段如下:

```java
@Column(updatable=false)
```

## 多表连接

> 关于文档中的Bidirectional Relationships, 我不是很理解. 但是我弄清楚了注解的使用, 和原理

### 介绍

表之间的关系有多种, 一一对应多种注解

* 一对一 `@OneToOne`
* 一对多 `@OneToMany`
* 多对多 `@ManyToMany`
* 多对一 `@ManyToOne`

每种注解使用后, 就产生一个外键, 即被注解的字段指向字段类型代表的表的主键. 外键名通常由字段名加上后缀`_id`得到. 外键名也可由

若不想生成外键, 则需要在应用层上提供关系的另一端的信息. 由注解的`mappedBy`属性提供.

**小结**: 即在一个实体中, 关系的提供有两种方式: 一, 是提供数据库的外键提供; 二, 提供应用层中注解的`mappedBy`属性提供

### 懒加载

多表连接默认全部加载, 上述注解的`fetch`属性可配置懒加载....

### 例子

部门表

```java
@Data
@Entity
public class Department {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "name")
    private String name;

    @Column(name = "description")
    private String description;
	// 部门与职工是一对多的关系, 关系由mappedBy提供
    @OneToMany(mappedBy = "department")
    private Set<Employee> employees;
}
```

职工表

```java
@Data
@Entity
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "name")
    private String name;

    @Column(name = "email")
    private String email;

    @Column(name = "address")
    private String address;
	// 职工与部门是多对一的关系, 关系由外键提供.
    @ManyToOne
    @JoinColumn(name="dep_id")
    private Department department;
}
```

## 自定义查询

* 介绍

  `@Query`中可声明`JPQL `语句(默认), 和原生SQL语句; JPQL的具体使用见下

* 实体类`Employee`

  ```java
  @Data
  @Entity
  public class Employee {
      private  Integer id;
      private String name;
      private Integer age;
  }
  ```

* 基本查询

  ```java
  @Query("select o from Employee o where id=(select max(id) from Employee t1)")
  Employee getEmployeeById();
  ```

  > `Employee`表示实体类的表明, 若实体类用`@Entity`配置了其他表名, 则JPQL中需要添加该表名.
  >
  > sql中可使用`<tableName>.entityField`的方式设置字段名.

* 参数传递

  数字表示位置, `1`表示第一个

  ```java
  @Query("select o from Employee o where o.name=?1 and o.age=?2")
  List<Employee> queryParams1(String name, Integer age);
  ```

  或者使用参数名

  ```java
  @Query("select o from Employee o where o.name=:name and o.age=:age")
  List<Employee> queryParams2(@Param("name") String name, @Param("age") Integer age);
  ```

  > `@Param`据说可忽略?

* like查询

  ```java
  @Query("select o from Employee o where o.name like %?1%")
  List<Employee> queryLike1(String name);
  ```

* 分页&排序

  ```java
  @Query( "SELECT * FROM USERS WHERE LASTNAME = ?1")
  Page<User> findByLastname(String lastname, Pageable pageable);
  ```

  ```java
  @Query("select u from User u where u.lastname like ?1%")
  List<User> findByAndSort(String lastname, Sort sort);
  ```

* 更新

  必须提供`@Modifying`注解

  ```java
  @Modifying
  @Query("update Employee o set o.age = :age where o.id = :id")
  void update(@Param("id") Integer id, @Param("age") Integer age);
  ```

* 删除

  必须提供`@Modifying`注解

  ```java
  @Modifying
  @Query("delete from Employee o where o.id = :id")
  void delete(@Param("id") Integer id);
  ```

* 支持SpEL, 以及提供`#entityName`表示实体代表的表名

  ```java
  @Query("select u from #{#entityName} u where u.lastname = ?1")
  List<User> findByLastname(String lastname);
  ```

  > 好处: 当实体表明改变时, 不用再修改JPQL了

* 原生SQL

  没有了解析过程, 不能以`.`的方式获取字段名, 和使用分页功能了. 但是参数获取的方式还是一样.

  ```java
  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);
  ```

> 参考:
>
> * [SpringData JPA @Query 注解实现自定义查询方法](https://www.jianshu.com/p/f93a62a7ec39)
>
> * [Using @Query](https://docs.spring.io/spring-data/jpa/docs/2.3.2.RELEASE/reference/html/#jpa.query-methods.at-query)

## 多数据源

> 一般情况下建议不要使用多数据源!!!!! 它不符合微服务理念, 且这里实现的多数据源方案, 指不定就那里会出问题

一般的, 实现多数据源有两种方案: 

1. 一个数据库对应一个数据源, 一个事务管理器, 一个JPA实体管理器.

2. 一个数据库对应一个数据源, 多个数据库共享同一个事务管理器和JPA实体管理器. 

但问题时是, Hibernate封装太深了, 第一个方案比较方便. 像Mybatis, 适合第二个方案.  但无论哪个方案, 都会对数据源交叉使用的事务造成影响.

这里介绍第一个方案, 要注意的是, 多数据源造成了大部分自动配置失效了.

1. 配置文件

   定义两个数据源, 和配置JPA. 注意, 请不要漏掉其中任何一个属性.

   ```yaml
   spring:
     jpa:
       show-sql: false
       hibernate:
         ddl-auto: validate
       database-platform: org.hibernate.dialect.MySQL57Dialect
   datasource:
     drug:
       url: jdbc:mysql://localhost:3306/db_drug?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&&useSSL=false&serverTimezone=CTT
       username: root
       password: jingyi@2020
     term:
       url: jdbc:mysql://localhost:3306/db_term?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&&useSSL=false&serverTimezone=CTT
       username: root
       password: jingyi@2020
   ```

2. 配置主数据源`drug`

   ```java
   import com.clinical.jingyi.term.collect_symptom.infrastructure.Constant;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.context.annotation.Primary;
   import org.springframework.core.env.Environment;
   import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
   import org.springframework.jdbc.datasource.DriverManagerDataSource;
   import org.springframework.orm.jpa.JpaTransactionManager;
   import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
   import org.springframework.transaction.PlatformTransactionManager;
   import org.springframework.transaction.annotation.EnableTransactionManagement;
   
   import javax.annotation.Resource;
   import javax.sql.DataSource;
   import java.util.Properties;
   
   /**
    * @author sidian
    * @date 2020/7/29 13:02
    */
   @Configuration
   @EnableTransactionManagement
   @EnableJpaRepositories(
           entityManagerFactoryRef = "drugEntityManagerFactory",
           transactionManagerRef = "drugTransactionManager",
           basePackages = {Constant.DAO_PACKAGE_NAME+"."+"drug"})
   public class DrugDatasourceConfiguration {
       @Resource
       Environment environment;
   
       @Bean
       @Primary
       @ConfigurationProperties(prefix = "datasource.drug")
       DataSource drugDatasource() {
           return new DriverManagerDataSource();
       }
   
       @Bean
       @Primary
       LocalContainerEntityManagerFactoryBean drugEntityManagerFactory(EntityManagerFactoryBuilder builder) {
           LocalContainerEntityManagerFactoryBean em = builder
                   .dataSource(drugDatasource())
                   .packages(Constant.DAO_PACKAGE_NAME+"."+"drug")
                   .build();
           // 将spring data jpa配置转化为hibernate配置. 其他的自行补充
           Properties properties = new Properties();
           properties.setProperty("hibernate.hbm2ddl.auto", environment.getProperty("spring.jpa.hibernate.ddl-auto"));
           properties.setProperty("hibernate.dialect",environment.getProperty("spring.jpa.database-platform"));
           properties.setProperty("hibernate.show_sql",environment.getProperty("spring.jpa.show-sql"));
           properties.setProperty("hibernate.physical_naming_strategy", "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy"); // 与spring data jpa名字映射一致
           em.setJpaProperties(properties);
           return em;
       }
   
       @Bean
       @Primary
       PlatformTransactionManager drugTransactionManager(EntityManagerFactoryBuilder builder) {
           JpaTransactionManager txManager = new JpaTransactionManager();
           txManager.setEntityManagerFactory(drugEntityManagerFactory(builder).getObject());
           return txManager;
       }
   }
   ```

3. 配置其他数据源`term` 

   不详细给出代码了, 只需将里面的`drug`替换为`term`, 以及去掉`@Primary`
   
4. 事务, 使用`@Transactional`时, 会使用主数据源的事务管理器, 因为有`@Primary`. 使用其他事务管理器, 需要显示指定事务管理器

   ```java
   @Transactional(transactionManager = "termTransactionManager")
   ```

> 参考如下, 参数链接的内容已经不适用了, 仅做参考
>
> * [Spring JPA – Multiple Databases](https://www.baeldung.com/spring-data-jpa-multiple-databases)
> * [springboot项目使用spring-data-jpa如何连接多数据源](https://www.imooc.com/article/43065)

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

### 自定义ID生成器

一般的, 插入一条记录时, 若数据库不存在, 则插入, 否则更新; 但是, 插入时, 无论实体是否设置了id, 插入时用的还是自动生成的.

下面实现一个ID生成器, 若实体设置了ID, 则插入该ID; 若为设置, 则使用`GenerationType.IDENTITY`策略

```java
/**
 *  自定义的主键生成策略，如果填写了主键id，如果数据库中没有这条记录，则新增指定id的记录；否则更新记录
 *
 *  如果不填写主键id，则利用数据库本身的自增策略指定id
 *
 * Created by @author yihui in 20:51 19/11/13.
 */
public class ManulInsertGenerator extends IdentityGenerator {

    @Override
    public Serializable generate(SharedSessionContractImplementor s, Object obj) throws HibernateException {
        // 获取实体id
        Serializable id = s.getEntityPersister(null, obj).getClassMetadata().getIdentifier(obj, s);

        if (id != null) { // id不为空
            // 使用该id
            return id;
        } else { // id为空
            // 使用IdentityGenerator自己的生成策略
            return super.generate(s, obj);
        }
    }
}
```

声明ID时, 同时配置使用的生成器

```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO, generator = "myid")
@GenericGenerator(name = "myid", strategy = "com.git.hui.boot.jpa.generator.ManulInsertGenerator")
@Column(name = "id")
private Integer id;
```

> `strategy = GenerationType.AUTO`可省略

> 来源: [SpringBoot系列教程JPA之指定id保存](https://juejin.im/post/5dd5400d6fb9a05a92108429)

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
  
  符合Spring Data Commons规范, 可自行扩充方法.
  
  > 注意的是, 与`Repository`提供的方法不同, 派生方法查询出来的实体, 遍历图的深度默认为1. 可通过`Depth`注解修改遍历升读.

## 参考

* [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/) 一个了解Neo4j的Demo案例
* [Spring Data Neo4j Reference Documentation](https://docs.spring.io/spring-data/neo4j/docs/5.3.1.RELEASE/reference/html/#reference)



# Spring Data Elasticsearch

> 转载至[Spring Data Elasticsearch基本使用](https://www.cnblogs.com/ifme/p/12005026.html)

## 介绍

* 无事务
* 弱类型映射, 即使Java实体映射到ES索引的类型是错误的, 也没关系....

## 配置

* 引入依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
  </dependency>
  ```

* 启用Elasticsearch

  ```java
  @EnableElasticsearchRepositories
  @SpringBootApplication
  public class HelloSpringDataApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(HelloSpringDataApplication.class, args);
      }
  }
  ```

* 配置文件

  ```yaml
  spring:
  	elasticsearch:
      	rest:
        		uris: http://localhost:9600
        		#username: root
        		#password: 123456
  ```

-----------------

上述使用的Rest API接口来访问Elasticsearch, 但对老版本ES不兼容. 下面提供TCP连接的方法.

* 降低Spring boot版本到`2.2.9.RELEASE`, 因为之后的版本不支持TCP连接了

  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.2.9.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
  </parent>
  ```

* 配置

  ```yaml
  spring:
    data:
      elasticsearch:
  #      repositories:
  #        enabled: true # 是否启用repository使用方式
        cluster-nodes: 119.3.200.75:9600 # es的tcp ip和端口
        cluster-name: gi-es-cluster # 集群名
  ```

  这里的`spring.data.elasticsearch.repositories.enabled`也可以使用上述用到的`@EnableElasticsearchRepositories`启动

## 实体类创建

> 参考[Mapping Annotation Overview](https://docs.spring.io/spring-data/elasticsearch/docs/4.0.2.RELEASE/reference/html/#elasticsearch.mapping.meta-model.annotations)

### Demo

```java
package com.example.elasticsearch.pojo;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

/**
 * @author john
 * @date 2019/12/8 - 13:47
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName = "item",type = "docs", shards = 1, replicas = 0)
public class Item {
    @Id
    private Long id;

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String title; //标题

    @Field(type = FieldType.Keyword)
    private String category;// 分类

    @Field(type = FieldType.Keyword)
    private String brand; // 品牌

    private Double price; // 价格

    @Field(index = false, type = FieldType.Keyword)
    private String images; // 图片地址
}
```

### 常用注解

Spring Data通过注解来声明字段的映射属性，有下面的三个注解：

- `@Document`

   作用在类，标记实体类为文档对象，一般有四个属性

  - indexName：对应索引库名称
  - type：~~对应在索引库中的类型~~
  - shards：分片数量，默认5
  - replicas：副本数量，默认1

- `@Id` 作用在成员变量，标记一个字段作为id主键

   > spring data里的`Id`注解

   > [spring data `id` vs. es `_id`](https://stackoverflow.com/questions/37277017/spring-data-elasticsearch-id-vs-id)
   >
   > spring data中被`@Id`注解的主键被设置值后, 在ES产生的文档中, `_id`与`_source.id`值一致. 若主键没有值, 那么ES产生的文档中, `_id`被ES自动生成, `_source.id`由于没有设置值, 因为为`null`.

- `@Field` 标注文档字段

   - `name` 字段名, 默认Java字段名
   
   - `type`：字段类型, 默认`FieldType.Auto`. 即每种类型都对应一种默认的ES类型, 其中, `String`类型字段会映射到`Text`, 同时有`keyword`的版本, 如
   
     ```
     "branch": {
         "type": "text",
         "fields": {
             "keyword": {
                 "type": "keyword",
                 "ignore_above": 256
             }
         }
     }
     ```
   
   - `format`: 字段为`Date`类型时, 必须使用该属性设置字段的格式化方式
   
   - `analyzer`：分词器名称, 中文环境下常用`ik_max_word`

### 其他

当保存实体时, 若字段不存在, 会自动创建. 若字段存在, 但类型不匹配, 不会去修改字段, 仅仅写入这类型不一致的值.

## Dao类创建

```java
@Repository
public interface ItemRepository extends ElasticsearchRepository<Item,Long> {}
```

> 注意, 修改操作没有Jpa那样的理念哦, 即数据修改后, 必须显式保存`save()`

## 测试创建索引

```java
package com.example.elasticsearch;

import com.example.elasticsearch.pojo.Item;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @author john
 * @date 2019/12/8 - 14:09
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class ElasticSearctTest {
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @Test
    public void testCreate() {
        // 创建索引，会根据Item类的@Document注解信息来创建
        elasticsearchTemplate.createIndex(Item.class);
        // 配置映射，会根据Item类中的id、Field等字段来自动完成映射
        elasticsearchTemplate.putMapping(Item.class);
    }
}
```

使用kubia查询
 ![img](.Spring%20Data/1580998-20191208143456081-415075024.png)

## 增删改操作

Spring Data 的强大之处，就在于你不用写任何DAO处理，自动根据方法名或类的信息进行CRUD操作。只要你定义一个接口，然后继承Repository提供的一些子接口，就能具备各种基本的CRUD功能。

编写 ItemRepository

```java
package com.example.elasticsearch.repository;

import com.example.elasticsearch.pojo.Item;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

/**
 * @author john
 * @date 2019/12/8 - 14:39
 */
public interface ItemRepository extends ElasticsearchRepository<Item,Long> {
}
```

### 增加

```java
    @Autowired
    private ItemRepository itemRepository;

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @Test
    public void testAdd() {
        Item item = new Item(1L, "小米手机7", " 手机",
                "小米", 3499.00, "http://image.leyou.com/13123.jpg");
        itemRepository.save(item);
    }
```

![img](.Spring%20Data/1580998-20191208144316714-48874555.png)

### 修改(id存在就是修改，否则就是插入)

```java
    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void testUpdate() {
        Item item = new Item(1L, "小米手机7777", " 手机",
                "小米", 9499.00, "http://image.leyou.com/13123.jpg");
        itemRepository.save(item);
    }
```

![img](.Spring%20Data/1580998-20191208144538408-2099147202.png)

### 批量新增

```java
    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void indexList() {
        List<Item> list = new ArrayList<>();
        list.add(new Item(2L, "坚果手机R1", " 手机", "锤子", 3699.00, "http://image.leyou.com/123.jpg"));
        list.add(new Item(3L, "华为META10", " 手机", "华为", 4499.00, "http://image.leyou.com/3.jpg"));
        // 接收对象集合，实现批量新增
        itemRepository.saveAll(list);
    }
```

![img](.Spring%20Data/1580998-20191208144734280-1624665433.png)

### 删除操作

```java
    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void testDelete() {
        itemRepository.deleteById(1L);
    }
```

![img](.Spring%20Data/1580998-20191208145056386-1545604299.png)

### 根据id查询

```java
    @Test
    public void testQuery(){
        Optional<Item> optional = itemRepository.findById(2L);
        System.out.println(optional.get());
    }
```

![img](.Spring%20Data/1580998-20191208151005101-1090727892.png)

### 查询全部，并按照价格降序排序

```java
     @Test
    public void testFind(){
        // 查询全部，并按照价格降序排序
        Iterable<Item> items = this.itemRepository.findAll(Sort.by(Sort.Direction.DESC, "price"));
        items.forEach(item-> System.out.println(item));
    }
```

![img](.Spring%20Data/1580998-20191208151142472-2111083555.png)

## 自定义方法

Spring Data 的另一个强大功能，是根据方法名称自动实现功能。
 比如：你的方法名叫做：findByTitle，那么它就知道你是根据title查询，然后自动帮你完成，无需写实现类。
 当然，方法名称要符合一定的约定：

| Keyword               | Sample                                     | Elasticsearch Query String                                   |
| --------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| `And`                 | `findByNameAndPrice`                       | `{"bool" : {"must" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}}` |
| `Or`                  | `findByNameOrPrice`                        | `{"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}}` |
| `Is`                  | `findByName`                               | `{"bool" : {"must" : {"field" : {"name" : "?"}}}}`           |
| `Not`                 | `findByNameNot`                            | `{"bool" : {"must_not" : {"field" : {"name" : "?"}}}}`       |
| `Between`             | `findByPriceBetween`                       | `{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : ?,"include_lower" : true,"include_upper" : true}}}}}` |
| `LessThanEqual`       | `findByPriceLessThan`                      | `{"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}}` |
| `GreaterThanEqual`    | `findByPriceGreaterThan`                   | `{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}}` |
| `Before`              | `findByPriceBefore`                        | `{"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}}` |
| `After`               | `findByPriceAfter`                         | `{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}}` |
| `Like`                | `findByNameLike`                           | `{"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}}` |
| `StartingWith`        | `findByNameStartingWith`                   | `{"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}}` |
| `EndingWith`          | `findByNameEndingWith`                     | `{"bool" : {"must" : {"field" : {"name" : {"query" : "*?","analyze_wildcard" : true}}}}}` |
| `Contains/Containing` | `findByNameContaining`                     | `{"bool" : {"must" : {"field" : {"name" : {"query" : "**?**","analyze_wildcard" : true}}}}}` |
| `In`                  | `findByNameIn(Collection<String>names)`    | `{"bool" : {"must" : {"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"name" : "?"}} ]}}}}` |
| `NotIn`               | `findByNameNotIn(Collection<String>names)` | `{"bool" : {"must_not" : {"bool" : {"should" : {"field" : {"name" : "?"}}}}}}` |
| `Near`                | `findByStoreNear`                          | `Not Supported Yet !`                                        |
| `True`                | `findByAvailableTrue`                      | `{"bool" : {"must" : {"field" : {"available" : true}}}}`     |
| `False`               | `findByAvailableFalse`                     | `{"bool" : {"must" : {"field" : {"available" : false}}}}`    |
| `OrderBy`             | `findByAvailableTrueOrderByNameDesc`       | `{"sort" : [{ "name" : {"order" : "desc"} }],"bool" : {"must" : {"field" : {"available" : true}}}}` |

例如，我们来按照价格区间查询，定义这样的一个方法：

```java
public interface ItemRepository extends ElasticsearchRepository<Item,Long> {

    /**
     * 根据价格区间查询
     * @param price1
     * @param price2
     * @return
     */
    List<Item> findByPriceBetween(double price1, double price2);
}
```

然后添加一些测试数据：

```java
@Test
public void indexList() {
    List<Item> list = new ArrayList<>();
    list.add(new Item(1L, "小米手机7", "手机", "小米", 3299.00, "http://image.leyou.com/13123.jpg"));
    list.add(new Item(2L, "坚果手机R1", "手机", "锤子", 3699.00, "http://image.leyou.com/13123.jpg"));
    list.add(new Item(3L, "华为META10", "手机", "华为", 4499.00, "http://image.leyou.com/13123.jpg"));
    list.add(new Item(4L, "小米Mix2S", "手机", "小米", 4299.00, "http://image.leyou.com/13123.jpg"));
    list.add(new Item(5L, "荣耀V10", "手机", "华为", 2799.00, "http://image.leyou.com/13123.jpg"));
    // 接收对象集合，实现批量新增
    itemRepository.saveAll(list);
}
```

不需要写实现类，然后我们直接去运行：

```java
@Test
public void queryByPriceBetween(){
    List<Item> list = this.itemRepository.findByPriceBetween(2000.00, 3500.00);
    for (Item item : list) {
        System.out.println("item = " + item);
    }
}
```

![img](.Spring%20Data/1580998-20191208151845872-2023675795.png)

虽然基本查询和自定义方法已经很强大了，但是如果是复杂查询（模糊、通配符、词条查询等）就显得力不从心了。此时，我们只能使用原生查询。

## 高级查询

### 基本查询

先看看基本玩法

```java
@Test
public void testBaseQuery(){
    // 词条查询
    MatchQueryBuilder queryBuilder = QueryBuilders.matchQuery("title", "小米");
    // 执行查询
    Iterable<Item> items = this.itemRepository.search(queryBuilder);
    items.forEach(System.out::println);
}
```

QueryBuilders提供了大量的静态方法，用于生成各种不同类型的查询对象，例如：词条、模糊、通配符等QueryBuilder对象。

结果：
 ![img](.Spring%20Data/1580998-20191208152945530-193467310.png)

elasticsearch提供很多可用的查询方式，但是不够灵活。如果想玩过滤或者聚合查询等就很难了。

### 自定义查询

先来看最基本的match query：

```java
@Test
public void testNativeQuery(){
    // 构建查询条件
    NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
    // 添加基本的分词查询
    queryBuilder.withQuery(QueryBuilders.matchQuery("title", "小米"));
    // 执行搜索，获取结果
    Page<Item> items = this.itemRepository.search(queryBuilder.build());
    // 打印总条数
    System.out.println(items.getTotalElements());
    // 打印总页数
    System.out.println(items.getTotalPages());
    items.forEach(System.out::println);
}
```

NativeSearchQueryBuilder：Spring提供的一个查询条件构建器，帮助构建json格式的请求体

`Page<item>`：默认是分页查询，因此返回的是一个分页的结果对象，包含属性：

- totalElements：总条数
- totalPages：总页数
- Iterator：迭代器，本身实现了Iterator接口，因此可直接迭代得到当前页的数据
- 其它属性：

![img](.Spring%20Data/1580998-20191208152719617-2122634873.png)

### 分页查询

利用`NativeSearchQueryBuilder`可以方便的实现分页：

```java
@Test
public void testNativeQuery2(){
    // 构建查询条件
    NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
    // 添加基本的分词查询
    queryBuilder.withQuery(QueryBuilders.termQuery("category", "手机"));

    // 初始化分页参数
    int page = 0;
    int size = 3;
    // 设置分页参数
    queryBuilder.withPageable(PageRequest.of(page, size));

    // 执行搜索，获取结果
    Page<Item> items = this.itemRepository.search(queryBuilder.build());
    // 打印总条数
    System.out.println(items.getTotalElements());
    // 打印总页数
    System.out.println(items.getTotalPages());
    // 每页大小
    System.out.println(items.getSize());
    // 当前页
    System.out.println(items.getNumber());
    items.forEach(System.out::println);
}
```

结果：

![img](.Spring%20Data/1580998-20191208153148705-81352934.png)

可以发现，**Elasticsearch中的分页是从第0页开始**。

### 排序

排序也通用通过`NativeSearchQueryBuilder`完成：

```java
@Test
public void testSort(){
    // 构建查询条件
    NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
    // 添加基本的分词查询
    queryBuilder.withQuery(QueryBuilders.termQuery("category", "手机"));

    // 排序
    queryBuilder.withSort(SortBuilders.fieldSort("price").order(SortOrder.DESC));

    // 执行搜索，获取结果
    Page<Item> items = this.itemRepository.search(queryBuilder.build());
    // 打印总条数
    System.out.println(items.getTotalElements());
    items.forEach(System.out::println);
}
```

结果：

![img](.Spring%20Data/1580998-20191208153254837-1204654654.png)

## 聚合

### 聚合为桶

桶就是分组，比如这里我们按照品牌brand进行分组：

```java
@Test
public void testAgg(){
    NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
    // 不查询任何结果
    queryBuilder.withSourceFilter(new FetchSourceFilter(new String[]{""}, null));
    // 1、添加一个新的聚合，聚合类型为terms，聚合名称为brands，聚合字段为brand
    queryBuilder.addAggregation(
        AggregationBuilders.terms("brands").field("brand"));
    // 2、查询,需要把结果强转为AggregatedPage类型
    AggregatedPage<Item> aggPage = (AggregatedPage<Item>) this.itemRepository.search(queryBuilder.build());
    // 3、解析
    // 3.1、从结果中取出名为brands的那个聚合，
    // 因为是利用String类型字段来进行的term聚合，所以结果要强转为StringTerm类型
    StringTerms agg = (StringTerms) aggPage.getAggregation("brands");
    // 3.2、获取桶
    List<StringTerms.Bucket> buckets = agg.getBuckets();
    // 3.3、遍历
    for (StringTerms.Bucket bucket : buckets) {
        // 3.4、获取桶中的key，即品牌名称
        System.out.println(bucket.getKeyAsString());
        // 3.5、获取桶中的文档数量
        System.out.println(bucket.getDocCount());
    }

}
```

显示的结果：

![img](.Spring%20Data/1580998-20191208153413642-1883478842.png)

### 嵌套聚合，求平均值

代码：

```java
@Test
public void testSubAgg(){
    NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
    // 不查询任何结果
    queryBuilder.withSourceFilter(new FetchSourceFilter(new String[]{""}, null));
    // 1、添加一个新的聚合，聚合类型为terms，聚合名称为brands，聚合字段为brand
    queryBuilder.addAggregation(
        AggregationBuilders.terms("brands").field("brand")
        .subAggregation(AggregationBuilders.avg("priceAvg").field("price")) // 在品牌聚合桶内进行嵌套聚合，求平均值
    );
    // 2、查询,需要把结果强转为AggregatedPage类型
    AggregatedPage<Item> aggPage = (AggregatedPage<Item>) this.itemRepository.search(queryBuilder.build());
    // 3、解析
    // 3.1、从结果中取出名为brands的那个聚合，
    // 因为是利用String类型字段来进行的term聚合，所以结果要强转为StringTerm类型
    StringTerms agg = (StringTerms) aggPage.getAggregation("brands");
    // 3.2、获取桶
    List<StringTerms.Bucket> buckets = agg.getBuckets();
    // 3.3、遍历
    for (StringTerms.Bucket bucket : buckets) {
        // 3.4、获取桶中的key，即品牌名称  3.5、获取桶中的文档数量
        System.out.println(bucket.getKeyAsString() + "，共" + bucket.getDocCount() + "台");

        // 3.6.获取子聚合结果：
        InternalAvg avg = (InternalAvg) bucket.getAggregations().asMap().get("priceAvg");
        System.out.println("平均售价：" + avg.getValue());
    }

}
```

结果：
 ![img](.Spring%20Data/1580998-20191208153539986-690533759.png)

  

# 参考

* [Spring Data Elasticsearch基本使用](https://www.cnblogs.com/ifme/p/12005026.html)