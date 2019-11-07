# 介绍

* **目的**: 使用注解, 减少重复性的模板代码

  > 如减少手动生成`setter`,`getter`方法

* **原理**: 在编译阶段为JavaBean生成相关字节码.

  > 通过某些API, 程序本身可以参与构建时期的注解处理

# 引入

只需要添加依赖, 

* Spring Boot项目中

    ```xml
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    ```

* 非Spring Boot

  ```xml
  <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.10</version>
      <scope>provided</scope>
  </dependency>
  ```
  
  > 编译时提供即可, 运行时不需要.

> 一般情况下, IDEA会报错, 因为未编译前注解未被解析, 代码未被生成. 因此需要下载Lombok插件, 让其修改IDEA的解析行为. 

# 使用

* `@Data`用于JavaBean

  * 自动生成`getter`,`setter`,`toString`,`hashCode`,`equals`等方法. 

      > 生成规则符合JavaBean规范, 注意:
      >
      > * 对于`boolean`类型属性, 如`foo`, 将生成`getter`方法`isFoo()`
      > * 如果已存在同名方法, 无论参数,返回值是否一致, 都不将自动生成该方法
      
  * 等同于`@Getter`,`@Setter`,`@RequiredArgsConstructor` ,`@ToString`,`@EqualsAndHashCode`

  * 防止某个属性生成Getter, Setter方法

      ```java
      @Getter(AccessLevel.NONE)
      @Setter(AccessLevel.NONE)
      private int mySecret;
      ```

* `@NonNull`生成检查参数的代码, 为`null`时将抛出`NullPointerException`异常. 可注解到参数或字段上, 在字段上时, 则检查其`getter`方法.

* 日记, 通过不同的日记注解, 可以生成对应的`log`字段. 下面列举常用的注解

  * `@Log4j2`生成

    ```java
    private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
    ```

  * `@Slf4j`生成

    ```java
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
    ```

  * ...

# 参考

* [入门](https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok)
* [官网](https://projectlombok.org/)

* [Lombok介绍、使用方法、工作原理和总结](http://www.yuanrengu.com/index.php/20180324.html)

