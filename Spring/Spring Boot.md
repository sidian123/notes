# 一 介绍

通过Spring Boot，能够快速的创建独立的、生产级的、基于spring的应用。spring boot整合了spring平台和三方库，通starter依赖与自动配置功能，极大的减少了配置的需要，甚至不需要配置。spring boot与框架整合的过程中也提供了配置的接口，如通过`application.properties`或`application.yml`来配置整合的框架。如果还是不满意，那只能单独为该框架写配置文件了。

总之，spring boot整合了spring框架和三方库，提供自动配置，并将框架的常用配置集中在单独的配置文件中（相当于配置文件的一层简单的封装），达到了快速开发的目的。

# 二 入门

首先通过[Spring Initializr](https://start.spring.io/)初始化项目（在IDEA中操作比较合适），会看到所有的starter依赖，这是spring boot整合后形成的依赖，几乎涵盖了软件整个生命周期所需的所有依赖。选择Web。

将产生的目录结构中的`.mvn`,`HELP.md`,`mvnw`,`mvnw.cmd`等文件或目录删除。

## pom.xml

spring boot应用可以直接打包成jar，如有starter-web依赖，还会嵌入tomcat容器，可直接在命令行上启动web该程序，提供web服务。在pom.xml文件中，我们几乎不必设置版本号，由spring boot管理。等等之类的功能，可以在pom.xml文件中得到解释，默认提供的内容如下所示：

- `parent`元素指定要继承配置的父pom文件`spring-boot-starter-parent`，含有spring boot的默认配置，如：
  - 默认JDK1.8
  - maven默认构建、输出使用UTF-8
  - `spring-boot-dependencies`作为父pom文件，含有spring boot配置的所有依赖的版本号。通过该依赖，在自己的pom中，我们几乎不必设置版本号，当然也可以手动覆盖它。
- 属性`java.version`指定JDK版本，这是spring boot设置独有的方式。
- 已选中的依赖，如web，和默认提供的test依赖。此时这些依赖我们不用设置版本号。
- [Starters](<https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#using-boot-starter>)：starter是spring boot整合后，用于描述一系列相关依赖的集合。如已提供的web、test依赖，包含web开发所需的依赖，并且内嵌tomcat。
  - 命名方式： `spring-boot-starter-*`，`*`表示某一类应用。
  - 官方支持[starter](<https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#using-boot-starter>)；社区支持的[starters](<https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-starters/README.adoc>)
- 插件`spring-boot-maven-plugin`的功能：
  - 自动搜索含有`main`方法，标记为可执行的class
  - 将环境变量下的所有jars打包成一个可运行的jar
  - 内置依赖解析器，自动设置依赖版本号与spring boot依赖一致。可以自行覆盖
  - 使用mvn package打包时，spring boot会执行repackge的过程（一些开发需要的jar会被排除）但**repackage**之前生成的jar仍会被提供。

## 自动配置

spring boot会根据classpath中jar或你已配置的Bean，然后自动配置和生成相关的Bean到spring容器中。具体配置什么Bean，spring boot已经默认设置了，基本上你不必手动配置。如果你已经配置了Bean，spring boot则不会配置该bean，让你通过一点点努力而得到更多的控制权。

> 使用了习惯大于配置的原则，通常自动配置的都是自己想要的

如果一些自动配置的类不是自己想要的，可在注解上指定`exclude`属性。

## 运行入口

静态方法`SpringApplication.run`通过一个配置类，会创建一个应用**容器**，这是spring boot程序的入口。

`@SpringBootApplication`标注的类为配置类，实际上主要是三个注解的组合：

- `@Configuration`：标注该类是用于Bean定义的配置类
- `@EnableAutoConfiguration`：开启自动配置，详细见2.2。
- `@ComponentScan`：Bean扫描，会扫描该配置类所在的包和子包。

常见的组合是，`@SpringBootApplication`注解的配置类中定义主函数作为入口，并执行`SpringApplication.run`，如：

```java
@SpringBootApplication
public class SpringBootHelloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootHelloworldApplication.class, args);
    }
}
```

## 目录结构

目录结构和平常使用的一样，可分层或分模块来组织代码：

- 分层

  ![在这里插入图片描述](.Spring%20Boot/20190509110331698.png)

  ![在这里插入图片描述](.Spring%20Boot/20190509110331698.png)

- 分模块

  ![在这里插入图片描述](.Spring%20Boot/20190509110354644.png)

其中：

- 主类所在的包和子包都会被spring boot扫描。
- static：包含静态资源文件，在web项目中，则存放可访问的前端代码
- templates：也是存放静态资源文件，在使用视图模板的web项目中存入模板文件。
- application.properties：spring boot项目的配置文件，被spring boot所支持的库都可以在这里进行配置。

注意点：

- spring boot入口类一定要放在包下，否则会导致扫描所有jar包。而是放在项目的root package下。
- 想要引入其他配置文件，无论xml、java代码配置，使用`@Import`就行了，spring ioc的内容。

## 测试

默认已经提供了test依赖（使用Junit），和一个测试类：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootHelloworldApplicationTests {
    @Autowired
    HelloController controller;

    @Test
    public void contextLoads() {
        System.out.println(controller.index());
    }
}
```

- `@RunWith(SpringRunner.class)`：启动spring测试容器，因此可以使用注解注入Bean。
- `@SpringBootTest`：在基于spring测试容器之上，提供过多的特性。看不懂都是些啥子特性。
- `@Test`：标注要测试的方法，和Junit一样。

# 三 深入

## 热更新

### 介绍

大致三种方案:

* 原生的JVM热更新: 限制比较大, 只能修改方法体内的代码, 其他情况将失败.

  > spring boot程序（被打包成jar）只是一个普通的java程序，因此可以使用Jvm热更新功能. 

* Devtools: SpringBoot提供的方案, 采用双类加载器方案. ` base`类加载器默认加载第三方Jar包, `restart`类加载器默认加载当前项目代码. Devtools监视classpath下文件变动, 当文件改变时, 触发重新启动, Devtools将丢弃原有`restart`类加载器, 重新创建 ,达到更新代码的效果.

  * 优点: 由于只加载项目代码, 重启速度快

  * 缺点: 将造成用到序列化的三方库失效

    > 如, Spring Data Redis使用`JdkSerializationRedisSerializer`和`@Cacheable`进行反序列化时, 将报`ClassCastException`错误.
    >
    > 具体见[Springboot + Redis相同类进行转换出现ClassCastException异常](https://blog.csdn.net/qq_43024380/article/details/94621611) 和[springboot+devtools+shiro-redis整合出现ClassCastException异常](https://blog.csdn.net/feinifi/article/details/85237166)

* JRebel: 以颗粒度更小的方法替换整个类, 拥有更少的限制, 只是收费的.

### devtools

依赖引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

> - 在repackage的过程中，devtools默认不会被打包进来。
> - 当运行为一个完整的jar时，devtools的功能会自动被禁用。因此应该使用IDE运行，或者`mvn spring-boot:run`？

**devtools的功能**：

- 默认关闭缓存：如spring mvc中，防止你不能立马看到改变后的效果。

- 自动重启（restart）：程序运行时，当classpath下文件改变会造成项目重启，IDE能够很方便的触发重启。

  > 注意，静态资源（位于`/resources`, `/static`, `/public`, or `/templates`等等）的更改不会触发程序重启，但会触发重载（reload）了。
  
  > Eclipse中，文件保存便会触发重启; IDEA中，需要点击`Build->Build Project`，快捷键`Ctrl+F9`

## spring boot配置

spring boot整合了spring框架和三方库后，提供了自动配置的功能，它的默认配置通常是我们需要的。spring boot也整合了他们的配置，允许我们使用spring boot的配置文件来配置他们。

- spring boot配置的途径有很多，常用的配置方式有：properties文件、YAML文件、环境变量、命令行参数，这些配置最终会存入`Environment`。

  > 同名属性覆盖顺序参考：[Externalized Configuration](<https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-external-config>)

- 获取`Environment`中属性值的两种方式：

  - `@Value`：将属性绑定到类的字段上，配合SpEL表达式使用。
  - `@ConfigurationProperties`：将一组属性绑定到类上。
  - 这两个玩意，复杂的呀~~皮！

- 一般使用配置文件`application.properties`,`application.yml`

- 属性可用随机数，如：

  ```properties
  my.secret=${random.value}
  my.number=${random.int}
  my.bignumber=${random.long}
  my.uuid=${random.uuid}
  my.number.less.than.ten=${random.int(10)}
  my.number.in.range=${random.int[1024,65536]}
  ```

- 占位符：当属性被使用时，会在`Environment`中处理一下，因此属性中可使用占位符：

  ```properties
  app.name=MyApp
  app.description=${app.name} is a Spring Boot application
  ```

- [通用配置](<https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html>)：这使用spring boot依赖或配置三方库时需要的大部分配置属性。也能存在其他的属性配置，这取决于引入的依赖是否从spring boot环境中取出值来。

### 配置文件

`PropertySource`定义了属性之间的覆盖关系, 如下以优先级降低的顺序列出所有的配置文件:

1. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#using-boot-devtools-globalsettings) on your home directory (`~/.spring-boot-devtools.properties` when devtools is active).
2. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.0.9.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.
3. [`@SpringBootTest#properties`](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/api/org/springframework/boot/test/context/SpringBootTest.html) annotation attribute on your tests.
4. Command line arguments.
5. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
6. `ServletConfig` init parameters.
7. `ServletContext` init parameters.
8. JNDI attributes from `java:comp/env`.
9. Java System properties (`System.getProperties()`).
10. OS environment variables.
11. A `RandomValuePropertySource` that has properties only in `random.*`.
12. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-external-config-profile-specific-properties) outside of your packaged jar (`application-{profile}.properties` and YAML variants).
13. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-external-config-profile-specific-properties) packaged inside your jar (`application-{profile}.properties` and YAML variants).
14. Application properties outside of your packaged jar (`application.properties` and YAML variants).
15. Application properties packaged inside your jar (`application.properties` and YAML variants).
16. [`@PropertySource`](https://docs.spring.io/spring/docs/5.0.9.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes.
17. Default properties (specified by setting `SpringApplication.setDefaultProperties`).

> 如命令行提供属性值:
>
> ```shell
> java -jar app.jar --name="Spring"
> ```

### YAML与properties

书写配置属性的两种方式，`@Value`也能获取yaml文件的属性。貌似yaml属性值前需要一个空格，文件后缀yml。

- 简单属性：

  - yaml：

    ```yaml
    url: http://dev.example.com
    name: Developer Setup
    ```

  - properties:

    ```properties
    url= http://dev.example.com
    name= Developer Setup
    ```

- 对象属性：

  - yaml：

    ```yaml
    environments:
    	dev:
    		url: http://dev.example.com
    		name: Developer Setup
    	prod:
    		url: http://another.example.com
    		name: My Cool App
    ```

  - properties:

    ```properties
    environments.dev.url=http://dev.example.com
    environments.dev.name=Developer Setup
    environments.prod.url=http://another.example.com
    environments.prod.name=My Cool App
    ```

- 数组属性：

  - yaml:

    ```yaml
    my:
        servers:
            - dev.example.com
            - another.example.com
    ```

    > `@Value()`获取不到, 考虑使用`,`分割的形式, 如
  	>
    > ```yaml
    > my:
    >     servers: dev.example.com, another.example.com
    > ```
    >
  
  - properties:
  
    ```properties
    my.servers[0]=dev.example.com
    my.servers[1]=another.example.com
    ```

### 属性值取出

一个例子，获取上面的数组属性：

```java
@Component
@ConfigurationProperties(prefix="my")
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```

或者

```java
@Component
public class Hello {
    @Value("${my.servers[0]}")
    String server1;
    @Value("${my.servers[1]}")
    String server2;
}
```

-------

给`@Value`设置默认值

```java
//基本类型
@Value("${some.key:})"//字符串
private String stringWithBlankDefaultValue;
       
@Value("${some.key:true}")//布尔
private boolean booleanWithDefaultValue;

@Value("${some.key:42}")//数值
private int intWithDefaultValue;

//数组
@Value("${some.key:one,two,three}")//字符串数组
private String[] stringArrayWithDefaults;

@Value("${some.key:1,2,3}")//int数组
private int[] intArrayWithDefaults;
```

### Profile

* 关于配置文件

  * `application-{profile}.properties`允许有多个, 但仅其中一个生效, 由 ` spring.profiles.active`决定. 

  * 未指定时, 默认使用`application-default.properties`(如果存在的话)

* 关于Bean与`@Profile`

  * 有`@Profile`注解的Bean, 仅在对应的Profile激活时, 会被注入
  * 注解中可声明多个Profile, 激活的Profile匹配一个则注入
  * 关于`@Profile`与重载方法之间的关系, 没看懂



### 其他

#### 路径

* 环境变量下的路径: 需加上前缀`classpath:`
* `**`匹配多层目录
* `*`匹配文件名中任意字符

例子如`mybatis.mapper-locations: classpath:mapperxml/**/*.xml`

# 四 Logging

Spring Boot对[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)、[Log4J2](https://logging.apache.org/log4j/2.x/) 和 [Logback](http://logback.qos.ch/)提供了**支持与默认配置**. 当在classpath下发现日记jar包时，会**自动配置**这些jar包，并且对于Spring Boot支持的日记系统，都可以通过[应用属性](<https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html>)进行配置，**即logging属性是独立于日记系统的**。

在Spring Boot框架内部，使用Commons Logging做内部使用的**接口**，而**接口实现**不强制规定。但由于spring boot使用的日记是在`ApplicationContext`创建之前初始化的，因此只能通过系统属性来修改spring boot使用的日记系统。并且，`starter`系的jar包默认使用`Logback`作为日结实现使用。替换麻烦，但Spring Boot提供恰当的路由，**保证其他使用不同日记系统的依赖能够正常工作**。

因此，自己的项目使用其他框架时不必关心spring boot内部使用的框架，并且应用属性的配置是可以同时作用到所支持的日记系统的！

> 我见有些人通过排除掉`spring-boot-starter-logging`（目的是去除Logback）的日记包来强制spring boot框架本身使用其他日记系统。但应该没必要吧，并且文档都强调了这点，如：
>
> There are a lot of logging frameworks available for Java. Do not worry if the above list seems confusing. Generally, **you do not need to change your logging dependencies and the Spring Boot defaults work just fine**.

> 补充：spring boot对Logback的支持度和耦合度高得多！！！不爽

## 使用log4j2

添加jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
<!-- 即使Logback有路由的机制, 但是还是可能会冲突, 此时需要去除Logback. 如Web starter可能会导致冲突, 因此去掉. 如果不冲突, 那么可以不去掉 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 该Starter存在于Spring Boot最新版中, 请同时也排除 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

类中获取Logger

```java
public class Hello{
    static final Logger LOGGER= LogManager.getLogger();
}
```

> 注意，类要引入和log4j相关的包，别引入错了。否则如果用的是Common Logging的接口，那么会导致不如意的使用了Logback，如果将Logback的jar排除掉，那么最终使用的就是log4j2了（被桥接了）。

至于配置文件呢？见下面。

## 配置

### 默认配置

之前说了，spring boot提供了默认配置，统一所支持日记系统的默认行为，即：

- 相同的Log格式，并带有颜色的。
- 日记级别默认为`info`
- 输出到Console。
- 等等

### 通用配置

spring boot也提供了常用的应用属性配置，并且这些属性是日记系统独立的：

- `debug`：true则设置日记级别为debug

- `trace`：true则设置日记级别为trace

- `logging.level.*`：为具体的某个包设置日记级别，如：

  ```properties
  logging.level.root=WARN
  logging.level.org.springframework.web=DEBUG
  logging.level.org.hibernate=ERROR
  ```

- `logging.group.*`：聚集多个包为组，便于配置为同一个日记级别。如：

  ```properties
  logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
  logging.level.tomcat=TRACE
  ```

  spring boot默认提供的有：

  | Name | Loggers                                                      |
  | :--- | :----------------------------------------------------------- |
  | web  | `org.springframework.core.codec`, `org.springframework.http`, `org.springframework.web` |
  | sql  | `org.springframework.jdbc.core`, `org.hibernate.SQL`         |

- 文件相关的属性

  * `logging.file.name` 日志文件名, 默认`spring.log`

    > 也可含路径

  * `logging.file.path` 日志文件的路径, 默认工作目录

    > 路径也可含文件名
    >
    > 好像该属性的解析有问题

  > 默认不输出到文件, 当上述两个属性任意一个存在时, 则会输出到文件中.

  ---------

  * `logging.file.max-siz` 一个日志文件最大的大小, 默认`10MB`

    > 超出后, 该文件会被归档

  * `ogging.file.max-history` 日志文件最多存档的数量, 默认`7`

    > 超出临界值后, 旧日志将被删除

  * `logging.file.clean-history-on-start` 是否启动时清除所有日志, 默认`false`

  > 上述默认值来源于logback的默认值.

### 自定义配置

> 注意, 即使提供自定义配置, Spring Boot的配置也生效, 且优先级要高

文件相关的属性，仅支持Logback，但是我们可以通过自定义日记系统自己的配置文件来支持这些属性。

> 貌似最新版支持文件相关的属性了, 但是有bug

默认情况下, 根据使用的日记系统, 不同的文件会被加载, 如下所示. 可通过属性`logging.config`显式指定使用的日志文件

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

> 配置文件名最好加上`-spring`。因为标准日记名也能被日记系统识别，如果日记系统比spring 容器先初始化，那么spring boot就不能很好的控制日记系统初始化了。

为了支持更好的扩展，spring boot将一一些`Environment`中的属性转移到了系统属性中，并且上述日记系统都支持从系统属性中获得属性。

| Spring Environment                    | System Property                   | Comments                                                     |
| ------------------------------------- | --------------------------------- | ------------------------------------------------------------ |
| `logging.exception-conversion-word`   | `LOG_EXCEPTION_CONVERSION_WORD`   | The conversion word used when logging exceptions.            |
| `logging.file.clean-history-on-start` | `LOG_FILE_CLEAN_HISTORY_ON_START` | Whether to clean the archive log files on startup (if LOG_FILE enabled). (Only supported with the default Logback setup.) |
| `logging.file.name`                   | `LOG_FILE`                        | If defined, it is used in the default log configuration.     |
| `logging.file.max-size`               | `LOG_FILE_MAX_SIZE`               | Maximum log file size (if LOG_FILE enabled). (Only supported with the default Logback setup.) |
| `logging.file.max-history`            | `LOG_FILE_MAX_HISTORY`            | Maximum number of archive log files to keep (if LOG_FILE enabled). (Only supported with the default Logback setup.) |
| `logging.file.path`                   | `LOG_PATH`                        | If defined, it is used in the default log configuration.     |
| `logging.file.total-size-cap`         | `LOG_FILE_TOTAL_SIZE_CAP`         | Total size of log backups to be kept (if LOG_FILE enabled). (Only supported with the default Logback setup.) |
| `logging.pattern.console`             | `CONSOLE_LOG_PATTERN`             | The log pattern to use on the console (stdout). (Only supported with the default Logback setup.) |
| `logging.pattern.dateformat`          | `LOG_DATEFORMAT_PATTERN`          | Appender pattern for log date format. (Only supported with the default Logback setup.) |
| `logging.pattern.file`                | `FILE_LOG_PATTERN`                | The log pattern to use in a file (if `LOG_FILE` is enabled). (Only supported with the default Logback setup.) |
| `logging.pattern.level`               | `LOG_LEVEL_PATTERN`               | The format to use when rendering the log level (default `%5p`). (Only supported with the default Logback setup.) |
| `logging.pattern.rolling-file-name`   | `ROLLING_FILE_NAME_PATTERN`       | Pattern for rolled-over log file names (default `${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`). (Only supported with the default Logback setup.) |
| `PID`                                 | `PID`                             | The current process ID (discovered if possible and when not already defined as an OS environment variable). |

> 注意，若想在日志配置文件中的属性中使用占位符时, 请使用[Spring的语法](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-external-config-placeholders-in-properties) , 经测试, 该功能仅在Logback中有效

Log4j2在Spring中的默认配置如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
	<Properties>
		<Property name="LOG_EXCEPTION_CONVERSION_WORD">%xwEx</Property>
		<Property name="LOG_LEVEL_PATTERN">%5p</Property>
		<Property name="LOG_DATEFORMAT_PATTERN">yyyy-MM-dd HH:mm:ss.SSS</Property>
		<Property name="CONSOLE_LOG_PATTERN">%clr{%d{${LOG_DATEFORMAT_PATTERN}}}{faint} %clr{${LOG_LEVEL_PATTERN}} %clr{%pid}{magenta} %clr{---}{faint} %clr{[%15.15t]}{faint} %clr{%-40.40c{1.}}{cyan} %clr{:}{faint} %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
		<Property name="FILE_LOG_PATTERN">%d{${LOG_DATEFORMAT_PATTERN}} ${LOG_LEVEL_PATTERN} %pid --- [%t] %-40.40c{1.} : %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
	</Properties>
	<Appenders>
		<Console name="Console" target="SYSTEM_OUT" follow="true">
			<PatternLayout pattern="${sys:CONSOLE_LOG_PATTERN}" />
		</Console>
	</Appenders>
	<Loggers>
		<Logger name="org.apache.catalina.startup.DigesterFactory" level="error" />
		<Logger name="org.apache.catalina.util.LifecycleBase" level="error" />
		<Logger name="org.apache.coyote.http11.Http11NioProtocol" level="warn" />
		<logger name="org.apache.sshd.common.util.SecurityUtils" level="warn"/>
		<Logger name="org.apache.tomcat.util.net.NioSelectorPool" level="warn" />
		<Logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="error" />
		<Logger name="org.hibernate.validator.internal.util.Version" level="warn" />
		<logger name="org.springframework.boot.actuate.endpoint.jmx" level="warn"/>
		<Root level="info">
			<AppenderRef ref="Console" />
		</Root>
	</Loggers>
</Configuration>
```

> 详细参考：[Custom Log Configuration](<https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-custom-log-configuration>)

## 自定义配置例子

加入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
<!-- 即使Logback有路由的机制, 但是还是可能会冲突, 此时需要去除Logback. 如Web starter可能会导致冲突, 因此去掉. 如果不冲突, 那么可以不去掉 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 该Starter存在于Spring Boot最新版中, 请同时也排除 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

在其中内中修改Spring内部使用的日志实现

```java
@SpringBootApplication
public class ClientApplication {
    static{
    	//让SpringBoot内部使用log4j2
		System.setProperty("org.springframework.boot.logging.LoggingSystem","org.springframework.boot.logging.log4j2.Log4J2LoggingSystem");
    }

    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }

}
```

创建`log4j2-spring.xml`文件, 从上述默认配置中修改

> 下面列出的配置, 比较全, 使用时, 根据自己需求修改即可.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别与严重程度: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration中status属性用于设置log4j2自身内部日志输出的等级, 默认warn; monitorInterval配置重新扫描配置的时间间隔-->
<Configuration status="WARN" monitorInterval="30">
    <!-- 属性,在其他元素中使用 -->
    <Properties>
        <Property name="LOG_EXCEPTION_CONVERSION_WORD">%xwEx</Property> <!-- 和异常有关, 具体不知. SpringBoot未提供时,将使用这个配置 -->
        <Property name="LOG_LEVEL_PATTERN">%5p</Property> <!-- 日志级别格式 -->
        <Property name="LOG_DATEFORMAT_PATTERN">yyyy-MM-dd HH:mm:ss.SSS</Property> <!-- 日期格式 -->
        <!-- 控制台输出格式, 带有颜色. 该颜色由ANSI转义字符提供, 若重定向到文件且用less查看时需要加上选项-R -->
        <Property name="CONSOLE_LOG_PATTERN">%clr{%d{${LOG_DATEFORMAT_PATTERN}}}{faint} %clr{${LOG_LEVEL_PATTERN}} %clr{%pid}{magenta} %clr{---}{faint} %clr{[%15.15t]}{faint} %clr{%-40.40c{1.}}{cyan} %clr{:}{faint} %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
        <!-- 文件输出格式, 无颜色, 因为考虑到很多编辑器不支持查看 -->
        <Property name="FILE_LOG_PATTERN">%d{${LOG_DATEFORMAT_PATTERN}} ${LOG_LEVEL_PATTERN} %pid --- [%15.15t] %-40.40c{1.} : %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
        <Property name="LOG_HOME">logs</Property> <!-- 日志家目录 -->
    </Properties>

    <!--所有的Appender,Appender负责将LogEvent派送到目的地，可以有很多目的地，如console，文件、远程服务器或数据库等等.-->
    <!-- Appender中可配置过滤器ThresholdFilter,即日志级别.-->
    <appenders>
        <!--打印所有日志到控制台-->
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout pattern="${CONSOLE_LOG_PATTERN}"/>
        </Console>

        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，用于临时测试-->
        <!--<File name="log" fileName="${LOG_HOME}/test.log" append="false">-->
        <!--<PatternLayout pattern="${FILE_LOG_PATTERN}"/>-->
        <!--</File>-->

        <!--打印所有info及以上级别的日志到文件中. 每天日志都会按照filePattern归档, 且当info.log文件大小超过size时，该日志也会被归档-->
        <RollingFile name="RollingFileInfo" fileName="${LOG_HOME}/info.log"
                     filePattern="./logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="${FILE_LOG_PATTERN}"/>
            <!-- 归档策略 -->
            <Policies>
                <!-- 根据上述filePattern归档, 即每天归档一次 -->
                <TimeBasedTriggeringPolicy/>
                <!-- 日志超过size归档 -->
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy的max属性设置该Appender最多存在多少个归档, 默认7个.注意,该元素即使未声明,也默认被使用. -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
        <!-- 打印所有warn及以上级别的日志到文件中 -->
        <RollingFile name="RollingFileWarn" fileName="${LOG_HOME}/warn.log"
                     filePattern="./logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="${FILE_LOG_PATTERN}"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
        <!-- 打印所有error及以上级别的日志到文件中 -->
        <RollingFile name="RollingFileError" fileName="${LOG_HOME}/error.log"
                     filePattern="./logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
        <!-- 打印所有的SQL日志到文件中. -->
        <RollingFile name="SQL" fileName="${LOG_HOME}/sql.log"
                     filePattern="./logs/$${date:yyyy-MM}/sql-%d{yyyy-MM-dd}-%i.log">
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="${FILE_LOG_PATTERN}"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
        </RollingFile>
    </appenders>

    <!-- 所有的Logger, 只有关联了Logger和Appender, Appender才生效. Logger元素与代理中使用的Logger对象需要绑定才能使用, 这个绑定关系是由框架维护的 -->
    <!-- 日志由Logger对象传给Logger元素, 这个传递过程涉及日志级别过滤和Level Inheritance -->
    <!-- 日志由logger元素传给Appender元素, 这个传递过程涉及日志级别的再次过滤和Additivity -->
    <loggers>
        <!-- Spring Boot默认Logger配置 -->
        <Logger name="org.apache.catalina.startup.DigesterFactory" level="error"/>
        <Logger name="org.apache.catalina.util.LifecycleBase" level="error"/>
        <Logger name="org.apache.coyote.http11.Http11NioProtocol" level="warn"/>
        <logger name="org.apache.sshd.common.util.SecurityUtils" level="warn"/>
        <Logger name="org.apache.tomcat.util.net.NioSelectorPool" level="warn"/>
        <Logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="error"/>
        <Logger name="org.hibernate.validator.internal.util.Version" level="warn"/>
        <logger name="org.springframework.boot.actuate.endpoint.jmx" level="warn"/>

        <!--sql日志. additivity为false, 表示不关联到父logger的appender上去. -->
        <logger name="live.sidian.blog.dao" level="debug" additivity="false">
            <appender-ref ref="Console"/>
            <appender-ref ref="SQL"/>
        </logger>
        <!-- 其他日志 -->
        <root level="info">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
        </root>
    </loggers>

</Configuration>
```

> 参考[spring boot2 使用log4j2](https://www.cnblogs.com/shaozm/p/10169708.html)

# 五 Spring MVC

需要引入starter-web依赖，然后会自动配置spring mvc所有的组件，并且加入了嵌入式tomcat，可直接打包成jar并运行。

- `RestController`：`@ResponseBody`与`@Controller`的结合，声明为控制器且使用对象作为返回值（默认使用Jackson转化为json数据）。
- `@RequestMapping`：将方法映射到某个http请求。可使用它的变种，如`@GetMapping`,`@PostMapping`等等。

## 自动 && 自定义配置

自动配置

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
- Support for serving static resources, including support for WebJars 
- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
- Support for `HttpMessageConverters` 
- Automatic registration of `MessageCodesResolver` 
- Static `index.html` support.
- Custom `Favicon` support 
- Automatic use of a `ConfigurableWebBindingInitializer` bean 

-----

自定义

* 若想添加额外特性, 如拦截器, 格式化器等, 需自己实现并注入一个`WebMvcConfigurer`
* 若想提供与映射相关的组件, 需注入` WebMvcRegistrationsAdapter `
* 若想完全控制Spring MVC的配置, 需提供` @EnableWebMvc `注解的`@ Configuration `配置类.

## HttpMessageConverters

`HttpMessageConverter` 用于转化HTTP请求或响应。默认对象会被转化为JSON或XML（如果存在Jackson xml），至于什么时候转化为什么类型，见[spring mvc-Content Types](https://blog.csdn.net/jdbdh/article/details/83512464#6.2%20Content%20Types)。字符编码默认使用`UTF-8`

自定义`HttpMessageConverters` ，添加更多`HttpMessageConverter` ：

```java
@Configuration
public class MyConfiguration {

	@Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}

}
```

这应该是添加新的converter吧。。

## JSON Serializers and Deserializers

对象序列化或解序列化时，使用的是Jackson的serializers或Deserializers，如果对象本身或对象的一个字段很复杂时，需要自定义serializer或deserializer，并在Jackson中注册。

略。。

> 参考:
>
> - [Custom JSON Serializers and Deserializers](<https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#boot-features-json-components>)
> - [Right way to write JSON deserializer in Spring or extend it](https://stackoverflow.com/questions/11376304/right-way-to-write-json-deserializer-in-spring-or-extend-it)

## 静态内容

看不懂。。。。

- `spring.resources.static-locations`属性定义静态内容位置
- `spring.mvc.static-path-pattern`：静态资源匹配模式。静态内容位置中匹配成功的才作为静态资源
- 只有静态内容位置的根目录才能使用欢迎页面。
- 当`DispatcherServlet`未能处理http请求时，会交给tomcat容器的默认servlet处理

## Path Matching and Content Negotiation

- 默认禁止后缀匹配
- 主要通过请求的`Accept`字段来返回相应MIME类型的响应。

## CORS支持

通过[`@CrossOrigin`](https://docs.spring.io/spring/docs/5.1.6.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) 可以对方法或类进行局部配置，而通过注册`WebMvcConfigurer` 可以进行全局配置，如：

```java
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")
            .allowCredentials(true);
        }
    };
}
```

`CorsRegistry`的默认行为：

- Allow all origins.
- Allow "simple" methods `GET`, `HEAD` and `POST`.
- Allow all headers.
- Set max age to 1800 seconds (30 minutes).

它默认允许所有源访问，却不是通过`*`实现的，而是通过设置为http请求的`Origin`字段（请求源域名）实现的。因此不会与`allowCredentials`冲突。

# 六 mybatis

## 使用

maven中加入依赖：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.1</version>
</dependency>
```

> 没看错, 确实有版本号

该依赖也加入了spring boot自动配置的功能：

- 从spring容器中自动检测已存在的`DataSource`
- `SqlSessionFactoryBean`中传入`DataSource` ，生成`SqlSessionFactory` 
- 从`SqlSessionFactory`中获得`SqlSessionTemplate` 
- 自动扫描mapper，连接到`SqlSessionTemplate` 上，并注入到spring容器中.

mybatis会引入starter-jdbc，而starter-jdbc又引入数据源[HikariCP](https://github.com/brettwooldridge/HikariCP)，因此只需要再添加mysql依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

默认扫描含`@Mapper`注解的接口，如：

```java
@Mapper
public interface UserDao {
    @Select("select * " +
            "from user " +
            "where id=#{id}")
    public User selectById(int id);
}
```

也可以在配置类上用注解`@MapperScan`将某个包的接口全部注册，如`@MapperScan("com.example.demo2.dao")`

> 在自动装配mapper时，idea报错说容器中无mapper，可禁止检查：
>
> Settings - Editor - Inspections - Spring - Spring Core - Code - Autowiring for Bean Class - disable

## 配置

mybatis与spring boot整合后，也可以在spring boot的配置文件中配置mybatis。部分属性如下：

- `mapper-locations`：xml Mapper的位置, 如`classpath:mapper/**/*.xml`

- `type-aliases-package`：类型匿名所在位置
- `configuration`：传给`Configuration`Bean的属性配置，见 [MyBatis reference page](http://www.mybatis.org/mybatis-3/configuration.html#settings)

mybatis基本不用配置什么，基本配置数据源就够了，如：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=GMT%2B8&allowPublicKeyRetrieval=true
    username: root
    password: 123456
    
mybatis:
  type-aliases-package: com.example.demo2.entity
```

mybatis注册mapper接口时，也会检测同包下是否存在对应xml文件，如果需要，还需配置maven，见[maven之允许src目录下xml文件输出到target目录](<https://blog.csdn.net/jdbdh/article/details/89068289>)

> 参考：
>
> - [mybatis-spring-boot-autoconfigure](<http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/#>)
> - [Spring Boot(六)：如何优雅的使用 Mybatis](https://www.cnblogs.com/ityouknow/p/6037431.html)：好文

# 七 其他

## JSON

spring boot支持三种JSON映射库的集成：

- Gson
- Jackson(默认使用)
- JSON-B

starter依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
</dependency>
```

> 自动配置Jackson，将注入`ObjectMapper` Bean。
>
> `spring-boot-starter-web`中已默认引入了该包。

## 嵌入servlet容器支持

- 支持嵌入式tomcat、Jetty和Undertow。

### 基本配置

- `server.port`：监听端口号，默认8080
- `server.address`：绑定地址
- `server.servlet.context-path`：配置content-path，即访问url的前缀。

### SSL

```yml
server:
  port: 8089
  ssl:
    enabled: true # 是否启用ssl, 默认true
    key-store-type: PKCS12 # 证书类型
    key-store: classpath:client.sidian.live.pfx # 证书
    key-store-password: q6W8VCSp # 证书的密码
```

> 参考:
>
> * [HTTPS using Self-Signed Certificate in Spring Boot](https://www.baeldung.com/spring-boot-https-self-signed-certificate)
> * [安装PFX格式证书](https://help.aliyun.com/document_detail/98576.html?spm=5176.2020520154.0.0.461956a75vS9c0)

# 参考

- [Building an Application with Spring Boot](<https://spring.io/guides/gs/spring-boot/>)：入门文档
- [Spring Boot Reference Guide](<https://docs.spring.io/spring-boot/docs/current/reference/html/>)或者[Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/)：spring boot参考文档
- [Common application properties](<https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html>)：通用应用属性配置，就是spring boot的配置文件。
- [Spring Boot’s "starters"](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/htmlsingle/#using-boot-starter)